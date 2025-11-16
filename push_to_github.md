# Hướng dẫn Push lên GitHub

## Các bước thực hiện:

1. Mở Git Bash hoặc Command Prompt/Terminal trong thư mục `github`

2. Khởi tạo git repository (nếu chưa có):
```bash
git init
```

3. Thêm remote repository:
```bash
git remote add origin https://github.com/changmieuphieubat/chuyendetotnghiep.git
```

4. Thêm tất cả các file:
```bash
git add .
```

5. Commit:
```bash
git commit -m "Initial commit: Add plan and documentation"
```

6. Push lên GitHub:
```bash
git branch -M main
git push -u origin main
```

## Lưu ý:
- Nếu repository đã có file, có thể cần pull trước:
```bash
git pull origin main --allow-unrelated-histories
```

- Nếu gặp lỗi authentication, cần setup GitHub credentials hoặc sử dụng Personal Access Token

