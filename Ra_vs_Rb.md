# Ra vs Rb: Đo features ở thời điểm nào?

## Hiểu đúng về `[Ra, Rb]`

`[Ra, Rb]` **không phải là hai lựa chọn label**. Nó là *cửa sổ thời gian* (từ release `Ra` đến release `Rb`) dùng để **định nghĩa** label.

Cái thực sự được lựa chọn là: **đo features tại thời điểm nào, Ra hay Rb**. Label thì giống hệt nhau trong cả hai trường hợp.

---

## Label luôn cố định

`isRefactored = 1 (True)` nghĩa là: *"file này từng xuất hiện trong ít nhất một refactoring action ở đâu đó trong khoảng `[Ra, Rb]`"*.

Đây là một sự thật đã xảy ra trong lịch sử git. RefactoringMiner xác định nó, và nó **không đổi** dù ta đo features ở thời điểm nào.

---

## Cái thay đổi là snapshot của features

### Đo tại Ra (cách ta đang làm)

- Chụp file ở trạng thái **trước khi** refactoring xảy ra.
- Features là **nguyên nhân (cause)**, đứng *trước* label về mặt thời gian.
- Model học: *"một test file trông như thế này (coupling cao, nhiều test smell, churn nhiều...) thì có khả năng sẽ bị refactor trong tương lai gần hay không?"*
- Đây đúng là bài toán **prediction**, khớp với trực giác "file có cần refactor hay không".

### Đo tại Rb, cách Martins gốc làm

- Chụp file ở trạng thái **sau khi** refactoring đã được áp dụng.
- Features là **kết quả (effect)** của chính việc refactoring, đứng *sau* label.
- Refactoring vốn làm giảm coupling, tách method, dọn smell... nên features tại Rb **tự động tương quan** với label vì chúng là hệ quả của nó.
- Model học *"code sau khi đã refactor trông ra sao"*, tức là **detection / hậu nghiệm** ("đã refactor hay chưa").

> **Kết luận:** tại Ra features là **cause**, tại Rb features là **effect**. Muốn *dự báo* thì phải học từ cause.

---

## Điểm yếu của từng cách

### Cách Rb, sai về phương pháp

- **Data leakage:** metric đẹp một cách giả tạo vì model đang "đọc đáp án" từ trạng thái sau.
- **Không deploy được:** ở thời điểm thực tế cần quyết định (bây giờ), ta không có snapshot tương lai (Rb) để đo. Không thể đo trạng thái tương lai của file để quyết định có refactor nó hôm nay hay không.

Đây chính là lý do pipeline phải re-extract toàn bộ features tại Ra.

### Cách Ra, đúng phương pháp nhưng "khó"

Các điểm yếu này thuộc loại *khó* chứ không phải *sai*:

- **Nhiễu do cửa sổ label dài.** Label gán cho cả khoảng `[Ra, Rb]` (có thể là cả một release cycle). Một file có thể bị refactor ở *cuối* cửa sổ vì lý do mới nảy sinh *sau* Ra (thêm feature mới, fix bug mới...). Snapshot tại Ra không thể chứa nguyên nhân chưa tồn tại, nên một phần nhãn dương về bản chất không thể dự báo được từ features Ra.
- **Quyết định refactor không thuần code.** Phụ thuộc nhiều vào yếu tố con người/tổ chức (ưu tiên team, deadline, ai đang sửa file đó) mà features code không nắm bắt được, nên kể cả model hoàn hảo vẫn có sai số không loại bỏ được.
- **Vấn đề thông thường khác:** class imbalance; label chỉ phản ánh những loại refactoring mà RefactoringMiner phát hiện được.

---

## Bảng tóm tắt

| | Đo tại **Ra** | Đo tại **Rb** |
|---|---|---|
| Vị trí features so với label | Trước (cause) | Sau (effect) |
| Bài toán | Prediction (dự báo) | Detection (hậu nghiệm) |
| Câu hỏi model học | "Sẽ bị refactor không?" | "Đã refactor chưa?" |
| Leakage | Không | Có |
| Deploy được | Có | Không |

`[Ra, Rb]` là **cửa sổ định nghĩa label** dùng chung. Ra/Rb là **lựa chọn thời điểm đo features**. Bắt buộc chọn Ra để features đứng *trước* label (tránh leakage), -> không phải vì hai cách cho ra hai label khác nhau.
