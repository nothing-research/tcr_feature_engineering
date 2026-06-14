# Quy trình tạo dataset

Dataset hiện tại (`features_pre_full_original.csv`) gồm 4.280 mẫu từ 59 Java project, với 120 features đo tại commit Ra (trước refactoring).

---

## Nguồn label

Label `isRefactored` được lấy từ **Martins et al.** (artifact công bố trên figshare). Martins et al. đã dùng **RefactoringMiner** để quét lịch sử git của 59 Java project, xác định các cặp release liên tiếp `(Ra, Rb)`, và gán nhãn từng test file: `isRefactored = 1` nếu file xuất hiện trong ít nhất một refactoring action trong cửa sổ `[Ra, Rb]`, ngược lại là 0.

Dataset gốc của Martins đo features tại Rb (trạng thái sau refactoring). Vì bài toán là prediction nên cần đo features tại Ra (trước refactoring). Pipeline trong project này thực hiện re-extract toàn bộ features tại Ra, giữ nguyên label từ Martins.

---

## Các bước xây dựng dataset

### Bước 1: Clone repo và resolve commit Ra

Clone từng repo về local. Với mỗi mẫu `(App, TestFilePath, Ra_tag, Rb_tag)` từ Martins, resolve tag Ra thành commit hash cụ thể bằng git. Loại bỏ các mẫu mà test file chưa tồn tại tại commit Ra (khoảng 3% bị loại, tương ứng những file được thêm vào sau Ra).

### Bước 2: Extract CK metrics

Checkout toàn bộ source tree tại commit Ra, chạy **CK tool (v0.7.0)** để đo class-level metrics trên toàn project: coupling (CBO, fanin, fanout), cohesion (LCOM*, TCC), inheritance (DIT, NOC), kích thước (LOC, NOM, WMC, RFC). CK chạy trên toàn project để đảm bảo context đầy đủ, không chỉ riêng test file.

### Bước 3: Extract AST, Lexical, Dependency features

Đọc nội dung test file tại commit Ra, parse bằng **tree-sitter-java**. Từ AST thu được:

- **AST features:** độ sâu, phân nhánh, entropy loại node, độ dài chuỗi method call
- **Lexical features:** entropy và độ dài identifier, comment density, string literal vocabulary
- **Dependency features:** số import, tỉ lệ static/wildcard/external, package diversity, annotation diversity

### Bước 4: Extract Test Semantic features và Test Smell counts

Từ cùng AST của test file, phát hiện các pattern kiểm thử bằng **tree-sitter-java**: tỉ lệ Mockito API calls (mock/stub/verify), annotation counts, fluent assertion chains, framework API diversity. Song song đó, chạy **tsDetect** để đếm 21 loại test smell (Eager Test, Mystery Guest, Assertion Roulette, v.v.).

### Bước 5: Extract Process metrics

Dùng git log trên lịch sử commit trước Ra để tính các chỉ số lịch sử phát triển của test file: số lần bị sửa, tổng dòng thêm/xóa, code churn, số tác giả, tuổi file.

### Bước 6: Merge và filter

Gộp tất cả feature tables theo key `(App, TestFilePath, SHA)`. Loại bỏ các cột duplicate về mặt thống kê (correlation gần 1) và các cột zero-variance đã được preregister trong `ablation_sets.json`. Kết quả là 120 features (37 original + 83 new) dùng trong experiments.

---

## Lưu ý

- Features được đo tại Ra, không phải Rb, để tránh data leakage.
- NaN từ CK (LCOM*, TCC không tính được khi thiếu methods/fields) được giữ nguyên, không impute trước khi split.
- SHA256 của dataset được lưu vào `DATASET_SHA256.txt` và không thay đổi sau khi experiments bắt đầu.
