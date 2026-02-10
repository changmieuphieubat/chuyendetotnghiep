# Hướng Dẫn Tích Hợp OAuth 2.0 PAR + PKCE

## Thông Tin Tài Liệu

| Hạng mục | Chi tiết |
|----------|----------|
| **Phiên bản** | 2.0 |
| **Ngày cập nhật** | 10/02/2026 |
| **Mục đích** | Hướng dẫn đối tác tích hợp lấy Access Token qua luồng PAR + PKCE |
| **Tác giả** | OCB - Open Banking Team |
| **Phân loại** | Tài liệu kỹ thuật dành cho đối tác |

### Lịch Sử Phiên Bản

| Phiên bản | Ngày | Nội dung thay đổi |
|-----------|------|-------------------|
| 1.0 | 19/01/2026 | Bản nháp đầu tiên |
| 2.0 | 10/02/2026 | Chuẩn hoá tài liệu cho đối tác; bổ sung hướng dẫn tạo PKCE; bổ sung Token Revocation |

---

## Mục Lục

1. [Tổng Quan](#tổng-quan)
2. [Thông Tin Môi Trường](#thông-tin-môi-trường)
3. [Điều Kiện Tiên Quyết](#điều-kiện-tiên-quyết)
4. [Sơ Đồ Luồng PAR + PKCE](#sơ-đồ-luồng-par--pkce)
5. [Hướng Dẫn Tạo PKCE Code Verifier & Code Challenge](#hướng-dẫn-tạo-pkce-code-verifier--code-challenge)
6. [Hướng Dẫn Từng Bước](#hướng-dẫn-từng-bước)
   - [Bước 1: PAR Request](#bước-1-par-request-yêu-cầu-ủy-quyền-đẩy)
   - [Bước 2: Authorization Request](#bước-2-authorization-request-xác-thực-người-dùng)
   - [Bước 3: Token Exchange](#bước-3-token-exchange-lấy-access-token)
   - [Bước 4: Token Revocation](#bước-4-token-revocation-thu-hồi-token)
7. [Xử Lý Lỗi](#xử-lý-lỗi)
8. [Hỗ Trợ Kỹ Thuật](#hỗ-trợ-kỹ-thuật)

---

## Tổng Quan

### OAuth 2.0 PAR + PKCE là gì?

**PAR (Pushed Authorization Request)** kết hợp với **PKCE (Proof Key for Code Exchange)** là luồng xác thực nâng cao theo tiêu chuẩn OAuth 2.0, được thiết kế để bảo mật tối đa quá trình trao đổi mã ủy quyền.

- **PAR (RFC 9126)**: Cho phép client gửi trước các tham số ủy quyền đến server qua kênh back-channel an toàn, nhận về một `request_uri` đại diện cho yêu cầu. Điều này giúp tránh lộ tham số trên URL và giảm nguy cơ bị giả mạo.
- **PKCE (RFC 7636)**: Sử dụng cặp `code_verifier` / `code_challenge` để chứng minh rằng client thực hiện Token Exchange chính là client đã khởi tạo Authorization Request, ngăn chặn tấn công chặn mã ủy quyền (authorization code interception).

### Lợi Ích

| Lợi ích | Mô tả |
|---------|-------|
| **Bảo mật cao** | Tham số ủy quyền được gửi qua back-channel, kết hợp PKCE ngăn chặn tấn công chặn mã |
| **Không cần Client Secret trên front-channel** | Phù hợp cho ứng dụng mobile, SPA và các ứng dụng công khai |
| **Xác thực QR Code** | Người dùng xác thực qua OMNI 4 eBanking App, không cần nhập mật khẩu trên web |
| **Tuân thủ tiêu chuẩn** | RFC 9126 (PAR), RFC 7636 (PKCE), RFC 6749 (OAuth 2.0) |

### Tóm Tắt Luồng

```
1. Client tạo code_verifier & code_challenge (PKCE)
2. Client gửi PAR Request → nhận request_uri
3. Chuyển hướng người dùng đến trang Authorization → hiển thị QR Code
4. Người dùng quét QR bằng OMNI 4 App → xác thực → nhận authorization code
5. Client đổi authorization code + code_verifier → nhận Access Token
```

### Kiến Trúc Hệ Thống

| Thành phần | Vai trò |
|------------|---------|
| **Client / Partner App** | Ứng dụng của đối tác thực hiện tích hợp |
| **OCB Open API Gateway** | Cổng API của OCB, xử lý routing, validation và rate limiting |
| **Keycloak Open API** | Identity Provider chính cho Open API |
| **Keycloak OMNI** | Identity Provider cho hệ thống OMNI, xử lý xác thực QR Code |
| **OMNI 4 eBanking App** | Ứng dụng mobile banking của khách hàng OCB |

---

## Thông Tin Môi Trường

### Base URL

| Môi trường | API Gateway Base URL | Authorization URL | Trạng thái |
|------------|---------------------|-------------------|------------|
| **Sandbox** | `<Sẽ cập nhật>` | `<Sẽ cập nhật>` | Sẽ cung cấp |
| **SIT** | `https://apic-sit-gw-gateway-apic-ocb-sit.apps.apic-dev.ocb.vn/ocb-sit/ocbsit/ocb-oauth-provider` | `https://open-banking-auth-sit.ocb.vn/realms/open-api/protocol/openid-connect/auth` | Nội bộ |
| **Production** | `<Sẽ cập nhật>` | `<Sẽ cập nhật>` | Sẽ cung cấp |

### API Endpoints

| API | Method | Path |
|-----|--------|------|
| PAR Request | `POST` | `{base_url}/oauth2/v2/par/request` |
| Authorization | `GET` | `{authorization_url}?client_id=...&request_uri=...` |
| Token Exchange | `POST` | `{base_url}/oauth2/v2/access-token` |
| Token Revocation | `POST` | `{base_url}/oauth2/v2/revoke` |

---

## Điều Kiện Tiên Quyết

Trước khi bắt đầu tích hợp, đối tác cần chuẩn bị các thông tin sau:

### Thông tin do OCB cung cấp

| Thông tin | Mô tả | Ví dụ |
|-----------|-------|-------|
| `client_id` | Định danh client được OCB cấp | `your-client-id` |
| `client_secret` | Bí mật client được OCB cấp | `your-client-secret` |
| `partner_id` | Mã đối tác, sử dụng cùng giá trị với `client_id` | `your-client-id` |
| `partner_name` | Tên đối tác | `TenDoiTac` |
| `scope` | Danh sách quyền được phép sử dụng | `VIRTUAL_ACCOUNT` |
| `partner_scope` | Scope đặc thù của đối tác, sử dụng cùng giá trị với `scope` | `VIRTUAL_ACCOUNT` |

### Thông tin đối tác tự chuẩn bị

| Thông tin | Mô tả | Yêu cầu |
|-----------|-------|---------|
| `redirect_uri` | URL callback nhận authorization code | Phải được đăng ký trước với OCB; sử dụng HTTPS |
| `code_verifier` | Chuỗi ngẫu nhiên dùng cho PKCE | Tạo mới cho mỗi phiên xác thực (xem mục hướng dẫn bên dưới) |
| `code_challenge` | Hash SHA-256 của code_verifier | Tạo từ code_verifier (xem mục hướng dẫn bên dưới) |
| `state` | Chuỗi ngẫu nhiên chống tấn công CSRF | Tạo mới cho mỗi phiên xác thực |
| `nonce` | Chuỗi ngẫu nhiên chống tấn công replay | Tạo mới cho mỗi phiên xác thực |

### Xác Thực API (Basic Authentication)

Tất cả các API request (trừ Authorization Request) yêu cầu header `Authorization` dạng Basic:

```
Authorization: Basic base64({client_id}:{client_secret})
```

**Ví dụ:**

```
client_id:     your-client-id
client_secret: your-client-secret
Base64 encode: eW91ci1jbGllbnQtaWQ6eW91ci1jbGllbnQtc2VjcmV0
→ Authorization: Basic eW91ci1jbGllbnQtaWQ6eW91ci1jbGllbnQtc2VjcmV0
```

> **Lưu ý bảo mật:** Không lưu trữ `client_secret` trong mã nguồn front-end, mobile app hoặc bất kỳ nơi nào có thể bị truy cập công khai.

---

## Sơ Đồ Luồng PAR + PKCE

```mermaid
sequenceDiagram
    participant Client as Client / Partner App
    participant Browser as Trình Duyệt
    participant Gateway as OCB Open API Gateway
    participant KCOpenAPI as Keycloak Open API
    participant KCOMNI as Keycloak OMNI<br/>(QR Code Page)
    participant MobileApp as OMNI 4 eBanking App
    participant Customer as Khách Hàng

    Note over Client: 1. Tạo code_verifier & code_challenge (PKCE)

    Client->>Gateway: 2. PAR Request<br/>(POST /oauth2/v2/par/request)<br/>+ code_challenge, scope, redirect_uri, partner_id
    Gateway->>KCOpenAPI: Chuyển tiếp PAR Request
    KCOpenAPI-->>Gateway: request_uri (expires_in: 60s)
    Gateway-->>Client: request_uri (expires_in: 60s)

    Note over Client: Lưu code_verifier an toàn

    Client->>Browser: 3. Chuyển hướng đến Authorization URL<br/>với request_uri
    Browser->>Gateway: Authorization Request<br/>(GET /auth?client_id=...&request_uri=...)
    Gateway->>KCOpenAPI: Chuyển tiếp Authorization Request
    KCOpenAPI->>KCOMNI: Ủy quyền xác thực (IdP Broker)
    KCOMNI->>KCOMNI: Tạo QR Code Session
    KCOMNI->>Browser: Hiển thị trang QR Code

    Note over Browser,KCOMNI: Trang web hiển thị QR Code<br/>chờ khách hàng quét

    Customer->>MobileApp: 4. Mở OMNI 4 eBanking App
    MobileApp->>KCOMNI: Quét QR Code (gửi Session ID)
    KCOMNI->>MobileApp: Yêu cầu xác thực

    Customer->>MobileApp: 5. Xác thực trên App<br/>(Username / Password / Biometric)
    MobileApp->>KCOMNI: Gửi thông tin xác thực
    KCOMNI->>KCOMNI: Xác thực người dùng

    alt Xác thực thành công
        KCOMNI->>Browser: Cập nhật trạng thái QR Page
        KCOMNI->>KCOpenAPI: Trả kết quả xác thực + User Info
        KCOpenAPI->>KCOpenAPI: Tạo Authorization Code
        KCOpenAPI->>Gateway: Authorization Code
        Gateway->>Browser: Redirect về redirect_uri<br/>với authorization code + state
        Browser->>Client: 6. Callback với authorization code + state

        Client->>Gateway: 7. Token Request<br/>(POST /oauth2/v2/access-token)<br/>+ authorization code + code_verifier
        Gateway->>KCOpenAPI: Chuyển tiếp Token Request
        KCOpenAPI->>KCOpenAPI: Xác minh code_verifier ↔ code_challenge
        KCOpenAPI-->>Gateway: Access Token + Refresh Token + ID Token
        Gateway-->>Client: Access Token + Refresh Token + ID Token

        Note over Client: Lưu trữ tokens an toàn

        Client->>Gateway: 8. Gọi Protected API với Access Token
        Gateway->>Gateway: Validate Access Token
        Gateway-->>Client: Dữ liệu phản hồi

    else Xác thực thất bại
        KCOMNI->>Browser: Hiển thị lỗi trên QR Page
        KCOMNI->>KCOpenAPI: Thông báo lỗi
        KCOpenAPI->>Gateway: Error Response
        Gateway->>Browser: Redirect với error
        Browser->>Client: Callback với error
    end
```

---

## Hướng Dẫn Tạo PKCE Code Verifier & Code Challenge

Trước khi bắt đầu luồng PAR + PKCE, client cần tạo cặp `code_verifier` và `code_challenge`. Cặp giá trị này **phải được tạo mới cho mỗi phiên xác thực**.

### Quy tắc

| Thành phần | Yêu cầu |
|------------|---------|
| `code_verifier` | Chuỗi ngẫu nhiên, độ dài 43–128 ký tự, chỉ chứa `[A-Z]`, `[a-z]`, `[0-9]`, `-`, `.`, `_`, `~` |
| `code_challenge` | `BASE64URL(SHA256(code_verifier))` (không padding `=`) |
| `code_challenge_method` | Luôn là `S256` |

### Ví dụ triển khai

#### JavaScript / Node.js

```javascript
const crypto = require('crypto');

// 1. Tạo code_verifier (chuỗi ngẫu nhiên 64 ký tự)
function generateCodeVerifier() {
  return crypto.randomBytes(48).toString('base64url'); // 64 ký tự
}

// 2. Tạo code_challenge từ code_verifier
function generateCodeChallenge(codeVerifier) {
  return crypto
    .createHash('sha256')
    .update(codeVerifier)
    .digest('base64url'); // base64url không có padding '='
}

// Sử dụng
const codeVerifier = generateCodeVerifier();
const codeChallenge = generateCodeChallenge(codeVerifier);

console.log('code_verifier:', codeVerifier);
console.log('code_challenge:', codeChallenge);
```

#### Python

```python
import os
import hashlib
import base64

# 1. Tạo code_verifier
code_verifier = base64.urlsafe_b64encode(os.urandom(48)).rstrip(b'=').decode('ascii')

# 2. Tạo code_challenge
code_challenge = (
    base64.urlsafe_b64encode(
        hashlib.sha256(code_verifier.encode('ascii')).digest()
    )
    .rstrip(b'=')
    .decode('ascii')
)

print('code_verifier:', code_verifier)
print('code_challenge:', code_challenge)
```

#### Java

```java
import java.security.MessageDigest;
import java.security.SecureRandom;
import java.util.Base64;

// 1. Tạo code_verifier
SecureRandom random = new SecureRandom();
byte[] bytes = new byte[48];
random.nextBytes(bytes);
String codeVerifier = Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);

// 2. Tạo code_challenge
MessageDigest digest = MessageDigest.getInstance("SHA-256");
byte[] hash = digest.digest(codeVerifier.getBytes("ASCII"));
String codeChallenge = Base64.getUrlEncoder().withoutPadding().encodeToString(hash);
```

> **Quan trọng:** `code_verifier` phải được lưu trữ an toàn phía server hoặc trong secure storage và **không được gửi đi** cho đến Bước 3 (Token Exchange).

---

## Hướng Dẫn Từng Bước

### Bước 1: PAR Request (Yêu Cầu Ủy Quyền Đẩy)

#### Mục đích

Gửi các tham số ủy quyền đến OCB Open API Gateway qua kênh back-channel an toàn và nhận về `request_uri` đại diện cho yêu cầu ủy quyền.

#### Endpoint

```
POST {base_url}/oauth2/v2/par/request
```

#### Request

**Headers:**

```http
Content-Type: application/x-www-form-urlencoded
Authorization: Basic {base64(client_id:client_secret)}
```

**Body Parameters:**

| Tham số | Bắt buộc | Mô tả | Giá trị mẫu |
|---------|----------|-------|-------------|
| `response_type` | Có | Loại phản hồi, cố định | `code` |
| `redirect_uri` | Có | URI callback đã đăng ký với OCB | `https://partner.example.com/callback` |
| `scope` | Có | Quyền truy cập được yêu cầu | `VIRTUAL_ACCOUNT` |
| `state` | Khuyến nghị | Chuỗi ngẫu nhiên chống CSRF, client tự tạo | `a1b2c3d4e5` |
| `nonce` | Khuyến nghị | Chuỗi ngẫu nhiên chống replay, client tự tạo | `n1o2n3c4e5` |
| `code_challenge` | Có | `BASE64URL(SHA256(code_verifier))` | `9W_kwIn8pZpCXSbIOfkdVaY...` |
| `code_challenge_method` | Có | Phương thức hash, cố định | `S256` |
| `partner_name` | Có | Tên đối tác (do OCB cung cấp) | `TenDoiTac` |
| `partner_scope` | Có | Giá trị trùng với `scope` | `VIRTUAL_ACCOUNT` |
| `partner_id` | Có | Giá trị trùng với `client_id` | `your-client-id` |

**Ví dụ cURL:**

```bash
curl --location 'POST {base_url}/oauth2/v2/par/request' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Authorization: Basic {base64_credentials}' \
--data-urlencode 'response_type=code' \
--data-urlencode 'redirect_uri=https://partner.example.com/callback' \
--data-urlencode 'scope=VIRTUAL_ACCOUNT' \
--data-urlencode 'state=a1b2c3d4e5' \
--data-urlencode 'nonce=n1o2n3c4e5' \
--data-urlencode 'code_challenge={code_challenge}' \
--data-urlencode 'code_challenge_method=S256' \
--data-urlencode 'partner_name={partner_name}' \
--data-urlencode 'partner_scope={scope}' \
--data-urlencode 'partner_id={client_id}'
```

#### Response

**Thành công (HTTP 200):**

```json
{
    "request_uri": "urn:ietf:params:oauth:request_uri:6E1A2B3C4D5E6F7G8H9I0J",
    "expires_in": 60
}
```

| Trường | Kiểu | Mô tả |
|--------|------|-------|
| `request_uri` | string | URI tham chiếu cho yêu cầu ủy quyền |
| `expires_in` | number | Thời gian hiệu lực tính bằng giây (60s) |

> **Lưu ý:**
> - `request_uri` hết hạn sau **60 giây** — cần tiến hành Bước 2 ngay lập tức.
> - Lưu `code_verifier` an toàn, sẽ cần ở Bước 3.

---

### Bước 2: Authorization Request (Xác Thực Người Dùng)

#### Mục đích

Chuyển hướng người dùng (khách hàng OCB) đến trang xác thực để quét QR Code bằng OMNI 4 eBanking App.

#### Endpoint

```
GET {authorization_url}?client_id={client_id}&request_uri={request_uri}
```

**Tham số URL:**

| Tham số | Bắt buộc | Mô tả |
|---------|----------|-------|
| `client_id` | Có | Định danh client (do OCB cung cấp) |
| `request_uri` | Có | Giá trị `request_uri` nhận được từ Bước 1 |

**Ví dụ URL:**

```
{authorization_url}?client_id={client_id}&request_uri=urn:ietf:params:oauth:request_uri:6E1A2B3C4D5E6F7G8H9I0J
```

#### Luồng Xác Thực Người Dùng

```
┌─────────────────────────────────────────────────────────────────┐
│  1. Client chuyển hướng người dùng đến Authorization URL        │
│                         ↓                                       │
│  2. Trình duyệt hiển thị trang QR Code (do Keycloak OMNI tạo)  │
│                         ↓                                       │
│  3. Khách hàng mở OMNI 4 eBanking App → quét QR Code           │
│                         ↓                                       │
│  4. Khách hàng xác thực trên App (mật khẩu / vân tay / FaceID) │
│                         ↓                                       │
│  5. Trang QR Code tự động cập nhật → chuyển hướng về            │
│     redirect_uri kèm authorization code + state                 │
└─────────────────────────────────────────────────────────────────┘
```

#### Callback Response

Sau khi xác thực thành công, trình duyệt sẽ chuyển hướng về `redirect_uri`:

```
https://partner.example.com/callback?code={authorization_code}&state=a1b2c3d4e5
```

| Tham số | Mô tả |
|---------|-------|
| `code` | Mã ủy quyền (authorization code), sử dụng một lần duy nhất |
| `state` | Giá trị `state` đã gửi ở Bước 1, **phải xác minh trùng khớp** |

> **Lưu ý:**
> - **Bắt buộc** xác minh `state` trùng khớp với giá trị đã gửi ở Bước 1 để ngăn tấn công CSRF.
> - Authorization code chỉ dùng **một lần** và hết hạn nhanh (thường 60–300 giây).
> - Cần tiến hành Bước 3 ngay lập tức sau khi nhận code.
> - QR Code session có thời gian hết hạn (thường 2–5 phút). Nếu hết hạn, cần bắt đầu lại từ Bước 1.

---

### Bước 3: Token Exchange (Lấy Access Token)

#### Mục đích

Đổi authorization code kết hợp `code_verifier` để lấy Access Token, Refresh Token và ID Token.

#### Endpoint

```
POST {base_url}/oauth2/v2/access-token
```

#### Request

**Headers:**

```http
Content-Type: application/x-www-form-urlencoded
Authorization: Basic {base64(client_id:client_secret)}
```

**Body Parameters:**

| Tham số | Bắt buộc | Mô tả | Giá trị mẫu |
|---------|----------|-------|-------------|
| `grant_type` | Có | Loại grant, cố định | `authorization_code` |
| `code` | Có | Authorization code từ Bước 2 | `6f75ceea-3e92-4db0-92e6-...` |
| `redirect_uri` | Có | Phải trùng khớp với `redirect_uri` ở Bước 1 | `https://partner.example.com/callback` |
| `code_verifier` | Có | Giá trị `code_verifier` gốc đã tạo trước Bước 1 | `fW7f721Pf8rzb79UFVFV...` |

**Ví dụ cURL:**

```bash
curl --location 'POST {base_url}/oauth2/v2/access-token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Authorization: Basic {base64_credentials}' \
--data-urlencode 'grant_type=authorization_code' \
--data-urlencode 'code={authorization_code}' \
--data-urlencode 'redirect_uri=https://partner.example.com/callback' \
--data-urlencode 'code_verifier={code_verifier}'
```

#### Response

**Thành công (HTTP 200):**

```json
{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 3600,
    "refresh_expires_in": 86400,
    "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "token_type": "Bearer",
    "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
    "not-before-policy": 0,
    "session_state": "224198a0-42a6-4869-ad90-9b92518c1526",
    "scope": "openid VIRTUAL_ACCOUNT profile email"
}
```

| Trường | Kiểu | Mô tả |
|--------|------|-------|
| `access_token` | string | JWT token để gọi Protected API |
| `expires_in` | number | Thời gian hiệu lực của access token (giây). Mặc định: 3600 (1 giờ) |
| `refresh_token` | string | Token dùng để làm mới access token khi hết hạn |
| `refresh_expires_in` | number | Thời gian hiệu lực của refresh token (giây). Mặc định: 86400 (24 giờ) |
| `token_type` | string | Loại token, luôn là `Bearer` |
| `id_token` | string | JWT chứa thông tin định danh người dùng (OpenID Connect) |
| `scope` | string | Danh sách scope đã được cấp |

#### Sử dụng Access Token để gọi API

Sau khi có access token, đối tác gửi kèm trong header khi gọi Protected API:

```http
GET /api/v1/resource HTTP/1.1
Host: {api_host}
Authorization: Bearer {access_token}
```

> **Lưu ý bảo mật:**
> - Lưu trữ token an toàn (mã hóa, secure storage, keychain).
> - **Không** lưu token trong logs, URL, hoặc local storage trên trình duyệt.
> - Chủ động sử dụng `refresh_token` để lấy access token mới **trước khi** access token hết hạn.
> - Server sẽ xác minh `code_verifier` khớp với `code_challenge` từ Bước 1. Nếu không khớp, request sẽ bị từ chối.

---

### Bước 4: Token Revocation (Thu Hồi Token)

#### Mục đích

Thu hồi (vô hiệu hoá) access token hoặc refresh token khi người dùng đăng xuất hoặc khi phát hiện token bị lộ.

#### Endpoint

```
POST {base_url}/oauth2/v2/revoke
```

#### Request

**Headers:**

```http
Content-Type: application/x-www-form-urlencoded
Authorization: Basic {base64(client_id:client_secret)}
```

**Body Parameters:**

| Tham số | Bắt buộc | Mô tả | Giá trị mẫu |
|---------|----------|-------|-------------|
| `token` | Có | Access token hoặc refresh token cần thu hồi | `eyJhbGciOiJSUzI1NiIs...` |
| `token_type_hint` | Không | Gợi ý loại token để server xử lý nhanh hơn | `access_token` hoặc `refresh_token` |

**Ví dụ cURL:**

```bash
curl --location 'POST {base_url}/oauth2/v2/revoke' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--header 'Authorization: Basic {base64_credentials}' \
--data-urlencode 'token={token_to_revoke}' \
--data-urlencode 'token_type_hint=access_token'
```

#### Response

**Thành công (HTTP 200):**

```
HTTP/1.1 200 OK
Content-Length: 0
```

Server trả về HTTP 200 với body rỗng khi thu hồi thành công.

> **Lưu ý:**
> - Sau khi thu hồi, token **không thể** sử dụng để gọi API nữa.
> - Thu hồi `refresh_token` sẽ đồng thời vô hiệu hoá tất cả access token liên quan.
> - Server có thể trả về 200 OK ngay cả khi token không tồn tại hoặc đã hết hạn (theo RFC 7009).
> - **Khuyến nghị** gọi API này khi người dùng đăng xuất.

#### Kịch Bản Sử Dụng

| Kịch bản | Hành động |
|----------|-----------|
| Người dùng đăng xuất | Thu hồi cả access token và refresh token |
| Phát hiện token bị lộ | Thu hồi ngay lập tức refresh token (sẽ vô hiệu hoá luôn access token) |
| Cấp lại quyền truy cập | Thu hồi token cũ, thực hiện lại luồng PAR + PKCE từ Bước 1 |

---

## Xử Lý Lỗi

### Bảng Mã Lỗi

| Mã lỗi | HTTP Status | Mô tả | Cách khắc phục |
|---------|-------------|-------|----------------|
| `invalid_request` | 400 | Thiếu tham số bắt buộc hoặc tham số không hợp lệ | Kiểm tra lại tất cả tham số bắt buộc |
| `invalid_client` | 401 | Thông tin xác thực client không đúng | Xác minh `client_id` và `client_secret` |
| `invalid_grant` | 400 | Authorization code không hợp lệ, đã hết hạn, hoặc đã sử dụng | Thực hiện lại luồng từ Bước 1 |
| `invalid_grant` | 400 | `code_verifier` không khớp với `code_challenge` | Kiểm tra lại logic tạo PKCE (xem mục hướng dẫn) |
| `invalid_scope` | 400 | Scope yêu cầu không được phép | Chỉ sử dụng scope đã được OCB cấp |
| `access_denied` | 403 | Người dùng từ chối cấp quyền | Người dùng cần đồng ý cấp quyền trên OMNI 4 App |
| `server_error` | 500 | Lỗi hệ thống phía server | Thử lại sau hoặc liên hệ OCB hỗ trợ |

### Ví Dụ Phản Hồi Lỗi

```json
{
    "error": "invalid_grant",
    "error_description": "Code verifier does not match code challenge"
}
```

### Khuyến Nghị Xử Lý Lỗi

1. **Luôn kiểm tra HTTP status code** trước khi xử lý response body.
2. **Log đầy đủ thông tin lỗi** (`error`, `error_description`) để hỗ trợ debug.
3. **Implement retry logic** cho lỗi `server_error` (HTTP 500) với exponential backoff.
4. **Không retry** cho các lỗi client (HTTP 4xx) — cần sửa request trước khi gửi lại.
5. Với lỗi `invalid_grant`, **bắt đầu lại toàn bộ luồng** từ Bước 1.

---

## Hỗ Trợ Kỹ Thuật

Nếu gặp vấn đề trong quá trình tích hợp, vui lòng liên hệ:

| Kênh hỗ trợ | Thông tin |
|-------------|-----------|
| **Email** | `<Sẽ cập nhật>` |
| **Hotline** | `<Sẽ cập nhật>` |

Khi liên hệ hỗ trợ, vui lòng cung cấp:
- Môi trường đang sử dụng (Sandbox / Production)
- Mã đối tác (`partner_id`)
- Mã lỗi và mô tả lỗi (`error`, `error_description`)
- Thời gian xảy ra lỗi
- Request ID (nếu có)

---

> **Tuyên bố miễn trừ:** Tài liệu này là tài sản của Ngân hàng TMCP Phương Đông (OCB). Nghiêm cấm sao chép, phân phối cho bên thứ ba khi chưa được sự đồng ý bằng văn bản của OCB.
