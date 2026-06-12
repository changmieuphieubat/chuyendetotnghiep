# ÔN THI CUỐI KỲ — MA004 Cấu trúc rời rạc

> **GV:** Lê Hoàng Tuấn | **Thi:** viết tay, 90 phút, tập trung (~13–14/06/2026)  
> **Được mang:** tài liệu giấy không giới hạn + máy tính bỏ túi  
> **Cấm:** điện thoại, iPad, laptop, AI  
> **Phạm vi:** đại số Boole, hàm Bool, đồ thị, cây (buổi 5–10) — **KHÔNG** hỏi lại giữa kỳ

---

## Cấu trúc đề thi (3 câu)

| Câu | Điểm | Nội dung |
|-----|------|----------|
| **1** | ~4đ | Hàm Bool: DNF (1a) + công thức tối tiểu (1b) + sơ đồ mạch (1c) |
| **2** | ~1đ | Vẽ đồ thị minh họa + lý thuyết “có tồn tại không” |
| **3** | ~5đ | Đồ thị trọng số: Euler (a), Hamilton (b), Dijkstra (c), Kruskal (d) |

**Chiến lược thi:** làm phần chắc trước; Câu 1 + Câu 3 chiếm ~9đ.

---

# PHẦN A — CÂU 1: HÀM BOOL (~4đ)

## A.1. Ký hiệu & quy ước

| Viết | Nghĩa |
|------|-------|
| **x̄, ȳ, …** | NOT (phủ định) |
| **·** | AND (nhân) |
| **+** | OR (cộng) |
| Thứ tự biến | **x y z t** (bit trái → phải) |

**Karnaugh 4 biến:** hàng = **xy**, cột = **zt**, thứ tự Gray: **00, 01, 11, 10** (cột 11 và 10 kề nhau — “cuộn tờ bìa”).

Chuỗi nhị phân `abcd` → hàng **ab**, cột **cd**.

---

## A.2. Hai dạng đề Câu 1

| Dạng | Đề | Bước đầu |
|------|-----|----------|
| **Công thức** | Đề 1, 3: `f = x·ȳ·z + …` | Khai triển từng thành phần → minterm → tô Kar |
| **Tập nghịch ảnh** | Đề 2, 4, 5: `f⁻¹(0)=…` hoặc `f⁻¹(1)=…` | Tô thẳng Kar (không khai triển) |

| Ký hiệu | Nghĩa | Trên Karnaugh |
|---------|--------|---------------|
| **f⁻¹(1)** | f = 1 tại các tổ hợp đó | **Tô ■** |
| **f⁻¹(0)** | f = 0 tại các tổ hợp đó | **Để trống □**, tô phần còn lại |

> **Cảnh báo:** tô ngược f⁻¹(0)/f⁻¹(1) → mất hết điểm phần Karnaugh.

---

## A.3. Câu 1a — DNF (dạng chính tắc tuyển)

**Quy trình:**
```
1. Vẽ Karnaugh 4×4, tô đúng các ô f = 1
2. DNF = tổng các minterm (mỗi minterm đủ 4 biến, có gạch nếu bit = 0)
3. Ghi kèm mã nhị phân (vd. 1010) nếu cần
```

**Khai triển từ công thức:** thành phần thiếu biến → nhân `(biến + biến̄)` rồi gộp bỏ trùng.

**Ví dụ đọc nhầm hay gặp:** thành phần 6 Đề 3 là **x·z·t̄** (x, z, t gạch), không phải x·z̄·t.

---

## A.4. Câu 1b — Công thức tối tiểu

Mỗi hàm có **2–3 công thức tối tiểu** — phải tìm **đủ**.

### Cách 1: Gom Karnaugh (Đề 3, 4 — thầy hay dùng)

```
Bước 1: Gom tế bào LỚN trước (8 ô → 4 ô → 2 ô → 1 ô)
Bước 2: Ghi T1, T2, T3… trên bảng
Bước 3: Phủ hết ô ■ (ô chỉ thuộc 1 tế bào → PI bắt buộc)
Bước 4: Ô còn trống → chọn T4 hoặc T5 (rẽ nhánh → nhiều công thức)
Bước 5: Viết 2–3 công thức, mỗi công thức ít thành phần nhất
```

**Quy tắc đọc tế bào:** biến **cố định** trong khối → giữ; biến **đổi** → bỏ.

**Khối 4 ô hàng 10+11, cột 11+10:** minterm 1010, 1011, 1110, 1111 — đều x=1, z=1 → **x·z** (4 ô, không phải 3).

### Cách 2: Quine–McCluskey (Đề 1, 2)

```
Bảng 1: liệt kê minterm, nhóm theo số bit 1
Bảng 2: ghép cặp khác đúng 1 bit → tế bào lớn (đánh dấu *)
Bảng 3: bảng phủ — cột chỉ có 1 dấu X → PI bắt buộc
→ 2 công thức (chọn X cho cột còn lại)
```

---

## A.5. Câu 1c — Sơ đồ mạch

Từ **1 công thức tối tiểu** ở 1b:

```
f = [TP1] + [TP2] + [TP3] + …
      │       │       │
     AND1   AND2   AND3  …  →  OR  →  f
```

| Trong công thức | Trên mạch |
|----------------|-----------|
| **·** | Cổng **AND** |
| **+** | Cổng **OR** |
| Gạch (x̄, ȳ, t̄) | Cổng **NOT** trước AND |

**Đếm cổng mẫu** `f = x·z + ȳ·t̄ + x̄·z + x·y·t`: 3 NOT + 4 AND + 1 OR.

**Lưu ý:** vẽ hình cổng; ghi x, y, z, t, f; rẽ nhánh dây (không vẽ lại NOT).

---

## A.6. Bảng tra nhanh 5 đề ôn — Câu 1

| Đề | Dạng | Số ô ■ | Cách 1b | Số công thức tối tiểu |
|----|------|--------|---------|------------------------|
| 1 | Công thức | 8 | QM | 2 |
| 2 | f⁻¹(0), 5 ô trống | 11 | QM | 2 |
| 3 | Công thức | 11 | Karnaugh | 2 |
| 4 | f⁻¹(1), 9 ô | 9 | Karnaugh | 3 |
| 5 | f⁻¹(0), 6 ô trống | 10 | Karnaugh | 2 |

---

# PHẦN B — CÂU 2: VẼ ĐỒ THỊ (~1đ)

## B.1. Khái niệm nền

| Thuật ngữ | Định nghĩa |
|-----------|------------|
| **Đơn đồ thị** | Không vòng, không cạnh bội |
| **Đa đồ thị** | Có thể có vòng và/hoặc cạnh bội |
| **Vòng (loop)** | Cạnh từ đỉnh về chính nó; **bậc +2** |
| **Cạnh bội** | 2+ cạnh nối cùng 1 cặp đỉnh |
| **Kₙ đầy đủ** | n đỉnh, n(n−1)/2 cạnh, mỗi đỉnh bậc n−1 |
| **Liên thông yếu** (có hướng) | Bỏ mũi tên → liên thông |
| **Liên thông mạnh** (có hướng) | Mọi cặp đỉnh đi được **có hướng** hai chiều |

## B.2. Bổ đề bắt tay

```
Σ deg(v) = 2 × |E|  →  tổng bậc luôn CHẴN
Số đỉnh bậc lẻ = số CHẴN
```

**“Có tồn tại không?”** — tính tổng bậc trước khi vẽ.  
Ví dụ: 9 người × 5 = 45 (lẻ) → **không tồn tại** đơn đồ thị.

## B.3. Ba dạng Câu 2 trong đề ôn

### Dạng 1 — Vẽ theo tính chất (Đề 1, 2, 4, 5)

| Yêu cầu | Cách làm nhanh |
|---------|----------------|
| Có hướng, đầy đủ, liên thông **yếu** | **K₄** có hướng (4 đỉnh, 12 cạnh) |
| Có hướng, đầy đủ, liên thông **mạnh** | **K₅/K₆** có hướng + chu trình có hướng |
| Đơn, không đầy đủ, Euler **và** Hamilton | **Ngũ giác** C₅: mọi đỉnh bậc 2 |
| Euler có, Hamilton **không** | Mọi bậc chẵn + xóa 1 đỉnh cầu → tách 2 mảnh |
| Đa đồ thị có vòng, không đầy đủ | Chu trình + loop tại 1 đỉnh |

**Euler ⟺ mọi đỉnh bậc chẵn** (vô hướng).  
**Hamilton không có:** xóa k đỉnh → số thành phần > k.

### Dạng 2 — Dãy bậc (Đề 3)

**Đề:** 6 đỉnh, bậc **2, 2, 3, 4, 4, 5**, liên thông.

```
Tổng bậc = 20 → 10 cạnh
F bậc 5 → nối F với A,B,C,D,E (đơn đồ thị)
Nối thêm 5 cạnh: C-D, C-E, D-E, D-B, C-A
```

| Ý | Vòng | Cạnh bội | Sửa từ (a) |
|---|------|----------|------------|
| **a** đơn | ✗ | ✗ | Xây như trên |
| **b** không vòng | ✗ | ✓ | 2 cạnh song song C—D |
| **c** không cạnh bội | ✓ | ✗ | Vòng tại F + F nối 3 đỉnh |
| **d** cả hai | ✓ | ✓ | Vòng F + cạnh bội C—D |

### Dạng 3 — Lý thuyết (Đề 1b)

Trả lời ngắn + lập luận (bổ đề bắt tay).

---

# PHẦN C — CÂU 3: ĐỒ THỊ TRỌNG SỐ (~5đ)

## C.1. Euler (3a) — duyệt CẠNH

| Câu hỏi | Điều kiện (vô hướng) |
|---------|----------------------|
| **Chu trình Euler (E_c)?** | Mọi đỉnh **bậc chẵn** |
| **Đường đi Euler (E_p)?** | **Đúng 2** đỉnh bậc **lẻ** |
| **Không có?** | **> 2** đỉnh bậc lẻ |

**Cách làm bài (bắt buộc):**
```
1. Ghi bậc từng đỉnh
2. Kết luận có/không + lý do
3. Nếu có → ghi tên: E_c = A→B→C→…→A
4. Kiểm tra: số cạnh trong chu trình = số cạnh đồ thị
```

**Có hướng:** chu trình Euler ⟺ deg⁺(v) = deg⁻(v) mọi v.

---

## C.2. Hamilton (3b) — duyệt ĐỈNH

**Không có điều kiện đơn giản như Euler** → vẽ thử hoặc dùng định lý.

| Công cụ | Nội dung |
|---------|----------|
| **Kₙ** | Luôn có chu trình Hamilton |
| **Dirac** | n đỉnh, mọi bậc ≥ n/2 → có H_c |
| **Ore** | Mọi cặp không kề: deg(u)+deg(v) ≥ n |
| **Không có H_c** | Xóa k đỉnh → số thành phần > k |

**Cách làm:** thử tìm → ghi `H_c = …` hoặc `H_p = …`.  
Nếu H_p đi hết đỉnh và điểm cuối nối điểm đầu → nâng lên H_c.

> Tìm không ra ≠ không có — cần lý do nếu kết luận không có.

---

## C.3. Dijkstra (3c) — đường đi ngắn nhất

**Mục tiêu:** từ đỉnh nguồn (đề cho) → dist ngắn nhất đến **mọi** đỉnh.

**Quy trình:**
```
L = {nguồn}, dist[nguồn] = 0, còn lại = ∞
Lặp:
  Chọn u ∉ L có dist nhỏ nhất → đưa u vào L
  Với mỗi v kề u: dist[v] = min(dist[v], dist[u] + w(u,v))  ← CỘNG DỒN
Ghi ô: (dist, đỉnh_trước)
```

**Bắt buộc 3 phần:** **bảng + hình + kết luận** (từ X đến Y: đường …, độ dài …).  
**Không có bảng → không chấm.**

| | Dijkstra | Prim (MST) |
|---|----------|------------|
| Mục tiêu | Đường ngắn nhất | Cây bao trùm |
| Cộng dồn dist | **Có** | **Không** |

---

## C.4. Kruskal (3d) — cây bao trùm

**Đọc kỹ đề:** chỉ hỏi **T1 (tối tiểu)** HOẶC **T2 (tối đa)** — làm sai loại = 0đ.

```
T1: sắp cạnh tăng dần trọng số
T2: sắp cạnh giảm dần trọng số
Lần lượt thêm cạnh nếu KHÔNG tạo chu trình
Dừng khi có n−1 cạnh
```

**Bảng 3 cột:** Trọng số | Cạnh | Quyết định (Chọn / Dừng)

**Kết quả:** vẽ cây trên hình + **tính tổng trọng số**.

---

# PHẦN D — CHECKLIST NỘP BÀI

## Câu 1

| Ý | Kiểm tra |
|---|----------|
| 1a | Kar 4×4, Gray code, đúng f⁻¹(0)/f⁻¹(1), DNF đủ minterm |
| 1b | Gom T1…Tn hoặc QM Bảng 1–3; **2–3 công thức** tối tiểu |
| 1c | Hình cổng NOT/AND/OR; ghi biến; từ 1 công thức 1b |

## Câu 2

| Ý | Kiểm tra |
|---|----------|
| | Đủ số đỉnh; đơn/đa đúng; có hướng → **mũi tên** |
| | Ghi bậc từng đỉnh (dãy bậc) |
| | Euler/Hamilton → ghi tên chu trình (nếu đề yêu cầu) |

## Câu 3

| Ý | Kiểm tra |
|---|----------|
| 3a | Bậc → kết luận → tên E_c/E_p |
| 3b | Tên H_c/H_p (hoặc lý do không có) |
| 3c | **Bảng Dijkstra + hình + kết luận từng đỉnh** |
| 3d | Bảng Kruskal + hình cây + **tổng trọng số**; đúng T1/T2 |

---

# PHẦN E — LỖI HAY MẤT ĐIỂM

| Lỗi | Hậu quả |
|-----|---------|
| Tô ngược Karnaugh (f⁻¹(0) vs f⁻¹(1)) | Mất hết điểm Kar |
| Thiếu công thức tối tiểu (chỉ tìm 1/2) | Trừ nặng 1b |
| Dijkstra thiếu bảng | **Không chấm** 3c |
| Nhầm Prim ↔ Dijkstra | Sai thuật toán |
| Kruskal: làm T1 khi đề hỏi T2 | 0đ ý d |
| Thiếu tổng trọng số cây | Trừ điểm 3d |
| Đồ thị có hướng thiếu mũi tên | Trừ Câu 2 |
| Sơ đồ mạch chỉ viết chữ, không vẽ cổng | Trừ 1c |
| Nhầm cạnh bội với “song song” hình học | Sai Câu 2 |
| Đọc nhầm bit xyzt (vd. 1010: z=1 không phải z=0) | Sai Kar |

---

# PHẦN F — TÀI LIỆU THAM KHẢO TRONG REPO

| File | Nội dung |
|------|----------|
| `DeOnTap/DE_1.png` … `DE_5.png` | 5 đề ôn |
| `Dap an de on tap CK mon MA004/DAP AN DE ONTAP 1.pdf` … `5.pdf` | Đáp án thầy |
| `De3_Cau1_HuongDan.md` | Hướng dẫn chi tiết Câu 1 Đề 3 |
| `TongHop_CauTrucRoiRac_thayLeHoangTuan.md` | Tổng hợp 10 buổi học (transcript) |

---

# PHẦN G — CÔNG THỨC & ĐỊNH LÝ TÓM TẮT

## Đại số Boole
- 1+1=1; không có x²; x·x = x; x+x̄ = 1
- De Morgan: (x·y)̄ = x̄+ȳ; (x+y)̄ = x̄·ȳ

## Đồ thị
- Σdeg = 2|E|; vòng: +2 bậc; Kₙ: n(n−1)/2 cạnh
- Euler (VH): E_c ⟺ mọi bậc chẵn; E_p ⟺ 2 bậc lẻ
- Cây: n đỉnh → n−1 cạnh; liên thông + không chu trình

## Thuật toán
- **Dijkstra:** dist mới = min(dist cũ, dist[u]+w) — cộng dồn
- **Kruskal:** greedy theo trọng số, tránh chu trình

---

*Biên soạn phục vụ ôn thi cuối kỳ MA004 — theo đề ôn & phương pháp thầy Lê Hoàng Tuấn.*
