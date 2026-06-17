# Cấu trúc paper

**Tiêu đề:** *Test Code Feature Engineering for Cross-Project Refactoring Prediction: An Empirical Study*

---

## Research Questions và Contributions

### RQ1. Does feature engineering improve cross-project test refactoring prediction?

**Trả lời bằng:** So sánh X_original, X_new_only, X_full trên cross-project protocol (MCC, F1, AUC).

**Lưu ý:** "Dataset Ra thay vì Rb" là điều kiện tiên quyết của thí nghiệm, không phải câu trả lời cho RQ1. Nên trình bày trong phần methodology, không phải findings của RQ1.

### RQ2. Which feature categories contribute most to prediction performance?

**Trả lời bằng:** Category-add, leave-one-out, feature importance (combined rank RF + XGB + MI).

### RQ3. Can a compact feature subset preserve prediction performance?

**Trả lời bằng:** Global ablation sweep K=5..120, X_selected (20 features) so với X_full (120 features).

---

## Contributions

**C1: Pre-refactoring dataset (methodological contribution)**

Dataset được re-extract tại Ra (trước refactoring), thay vì Rb như trong Martins et al. Đây là điều kiện cần để tránh temporal leakage trong bài toán prediction. C1 không gắn với RQ nào mà là tiền đề cho toàn bộ thí nghiệm.

**C2: Feature engineering (37 → 120 features)**

Gắn với RQ1. Mở rộng từ 37 features gốc lên 120 bằng 83 features mới thuộc 10 nhóm, có thể nhóm lại thành 5 perspectives để trình bày trong paper:

- AST structural và distribution
- Lexical
- Dependency
- Test semantic
- Extended CK metrics

Lưu ý: cần nhất quán giữa "5 perspectives" và "10 feature families" trong toàn bộ paper. Một cách xử lý: nói "ten feature families organized into five perspectives."

**C3: Feature group contribution analysis**

Gắn với RQ2. Bao gồm cả category-add lẫn leave-one-out. C4 ban đầu ("identifying informative vs redundant features") thực chất là cách diễn giải kết quả của C3, không phải contribution độc lập. Nên gộp C3 và C4 thành một.

**C4: Compact feature set (20 features)**

Gắn với RQ3. X_selected giảm 83% feature space trong khi duy trì hiệu suất cross-project.

---

## Cấu trúc contribution đề xuất

| Contribution | Nội dung | Gắn với |
|---|---|---|
| C1 | Pre-refactoring dataset tại Ra | Methodology (tiền đề) |
| C2 | 83 features mới, 5 perspectives | RQ1 |
| C3 | Category-add + LOO + importance | RQ2 |
| C4 | Compact subset 20 features | RQ3 |
