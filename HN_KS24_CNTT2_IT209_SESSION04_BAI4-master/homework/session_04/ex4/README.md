# Báo Cáo Bài Tập 4: Quản lý `.gitignore` và Sửa Lịch Sử (Amend)

**Sinh viên:** ngotrungchien  
**Email:** ngotrungchien232@gmail.com  
**Môn học:** IT209 - DevOps  
**Session:** 04 - Bài 4

---

## Mục Tiêu

- Cấu hình `.gitignore` để bỏ qua file nhạy cảm (`credentials.txt`) và file rác hệ thống.
- Gỡ bỏ file đã commit khỏi cache theo dõi của Git **mà không xóa file vật lý** trên ổ đĩa.
- Sửa thông điệp commit gần nhất bằng tùy chọn `--amend`.

---

## Bối Cảnh Bài Toán

Học viên đã **vô tình commit** file `credentials.txt` (chứa thông tin nhạy cảm: mật khẩu DB, API key, AWS secret key) lên Git. Nhiệm vụ cần thực hiện:

1. Gỡ file `credentials.txt` khỏi sự theo dõi của Git (giữ lại file vật lý).
2. Tạo `.gitignore` để Git tự động bỏ qua file này trong tương lai.
3. Sửa message commit cuối bằng `git commit --amend`.

---

## Các Bước Thực Hiện

### Bước 1: Tạo file và commit nhầm credentials.txt

```bash
# Tạo file credentials.txt chứa thông tin nhạy cảm
echo "DB_PASSWORD=SuperSecretPassword123!" > credentials.txt
echo "API_KEY=sk-prod-xxxxxxxxxxxx" >> credentials.txt

# Vô tình add và commit cả file nhạy cảm
git add .
git commit -m "add project files"
```

> ⚠️ **Lỗi xảy ra:** File `credentials.txt` đã bị commit vào Git history.

---

### Bước 2: Gỡ `credentials.txt` khỏi cache Git (không xóa file)

Sử dụng lệnh `git rm --cached` để **chỉ xóa file khỏi index (staging area) của Git**, giữ nguyên file trên ổ đĩa cứng:

```bash
git rm --cached credentials.txt
```

**Giải thích lệnh:**
| Tham số | Ý nghĩa |
|---------|---------|
| `git rm` | Lệnh xóa file trong Git |
| `--cached` | Chỉ xóa khỏi index/cache của Git, **KHÔNG xóa file vật lý** |
| `credentials.txt` | File cần gỡ khỏi tracking |

**Kết quả:** File `credentials.txt` vẫn tồn tại trên ổ đĩa nhưng Git sẽ không theo dõi nữa.

---

### Bước 3: Tạo file `.gitignore`

Tạo file `.gitignore` với nội dung bỏ qua `credentials.txt` và các file nhạy cảm khác:

```bash
# Tạo .gitignore
cat > .gitignore << 'EOF'
# File bao mat
credentials.txt
*.env
.env
*.pem
*.key

# Build Java
target/
build/
*.class
*.jar

# IDE
.idea/
*.iml
.vscode/

# OS
.DS_Store
Thumbs.db
EOF
```

```bash
# Add .gitignore vào staging
git add .gitignore
```

---

### Bước 4: Commit thay đổi

```bash
git commit -m "add gitignore and remove credentials from tracking"
```

---

### Bước 5: Sửa message commit bằng `--amend`

Sử dụng `git commit --amend` để cập nhật thông điệp commit **gần nhất** thành rõ ràng và chuyên nghiệp hơn:

```bash
git commit --amend -m "fix: remove sensitive credentials.txt from Git tracking and add .gitignore"
```

**Giải thích lệnh:**
| Tham số | Ý nghĩa |
|---------|---------|
| `git commit` | Lệnh tạo commit |
| `--amend` | Sửa đổi commit **gần nhất** thay vì tạo commit mới |
| `-m "..."` | Thông điệp commit mới |

> ⚠️ **Lưu ý quan trọng:** `git commit --amend` sẽ **thay thế** commit cuối bằng commit mới (hash thay đổi). Chỉ dùng khi commit **chưa được push** lên remote, hoặc phải dùng `git push --force` nếu đã push.

---

## Kết Quả Kiểm Tra

### Kiểm tra trạng thái file (`git status`)

```
$ git status
On branch master
nothing to commit, working tree clean
```

✅ `credentials.txt` **không còn** xuất hiện trong danh sách theo dõi của Git.  
✅ File `credentials.txt` **vẫn tồn tại** trên ổ đĩa cứng.  
✅ Working tree sạch, không còn file nào ở trạng thái Staged hay Modified.

---

### Kiểm tra lịch sử commit (`git log -n 1`)

```
$ git log -n 1

commit 0666741e270add2919e02ceeeeba73665d4954e1
Author: ngotrungchien <ngotrungchien232@gmail.com>
Date:   Sat Oct 3 19:28:47 2026 +0700

    fix: remove sensitive credentials.txt from Git tracking and add .gitignore
```

✅ Commit gần nhất hiển thị **thông điệp đã được sửa đổi** thành công qua `--amend`.  
✅ Author đúng: `ngotrungchien <ngotrungchien232@gmail.com>`

---

### Toàn bộ lịch sử commit (`git log --oneline`)

```
$ git log --oneline

0666741 fix: remove sensitive credentials.txt from Git tracking and add .gitignore
561d3ee add project files
```

---

## Tổng Kết

### So sánh `git rm` vs `git rm --cached`

| Lệnh | Xóa khỏi Git index | Xóa file vật lý |
|------|---------------------|-----------------|
| `git rm <file>` | ✅ Có | ✅ Có |
| `git rm --cached <file>` | ✅ Có | ❌ Không |

→ Dùng `git rm --cached` khi muốn **giữ file trên máy** nhưng **không để Git theo dõi**.

### Khi nào dùng `git commit --amend`?

- ✅ Sửa typo trong message commit vừa tạo.
- ✅ Thêm file bị quên vào commit cuối.
- ✅ Chỉnh sửa nội dung commit trước khi push.
- ❌ **Không nên dùng** cho commit đã push lên remote (shared branch).

### Quy trình bảo mật tốt nhất

```
1. Luôn tạo .gitignore TRƯỚC khi git add
2. Kiểm tra git status trước khi commit
3. Không commit file chứa password, key, token
4. Sử dụng biến môi trường hoặc secret manager
```

---

## Cấu Trúc File Nộp Bài

```
homework/session_04/ex4/
├── .gitignore     # File cấu hình bỏ qua file nhạy cảm
└── README.md      # Báo cáo giải thích chi tiết (file này)
```
