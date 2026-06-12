# Tóm tắt kết quả TCR Feature Engineering v2

## Giải thích thuật ngữ

| Thuật ngữ | Nghĩa |
|---|---|
| MCC | Matthews Correlation Coefficient: chỉ số phân loại cân bằng cả 4 ô confusion matrix, khoảng [-1, 1], 0 = không tốt hơn ngẫu nhiên |
| F1 | Harmonic mean của Precision và Recall, khoảng [0, 1] |
| AUC | Area Under the ROC Curve: xác suất model xếp đúng thứ tự một cặp positive/negative |
| SE | Standard Error = SD / sqrt(n), đo độ không chắc chắn của trung bình |
| Cross-project | Mỗi project chỉ xuất hiện trong train hoặc test, không cả hai (GroupKFold theo App) |
| Random-split | Shuffle toàn bộ dữ liệu rồi chia fold (StratifiedKFold); cùng project có thể vừa train vừa test |
| Ablation | Thử nghiệm thêm/bớt từng nhóm features để đo đóng góp |
| LOO | Leave-One-Out: bỏ một nhóm ra khỏi X_full, đo MCC giảm bao nhiêu |
| MI | Mutual Information: đo mức độ phụ thuộc thống kê giữa feature và label |
| Combined rank | Xếp hạng features tổng hợp từ RF impurity, XGB gain và MI |

---

**Dữ liệu:** PRE_CK, 4.227 mẫu (52,1% positive), 120 features (37 gốc + 83 mới)
**Seed:** 20260612 · Folds: 5 · Metric chính: MCC

---

## RQ1 - So sánh tập features (RandomForest)

### Cross-project (kết quả chính)

| Tập features | n | MCC (tb) | SD | F1 (tb) | SD |
|---|---|---|---|---|---|
| X_original | 37 | -0,060 | 0,054 | 0,546 | 0,049 |
| X_new_only | 83 | +0,023 | 0,058 | 0,594 | 0,037 |
| X_full | 120 | +0,017 | 0,048 | 0,581 | 0,039 |
| **X_selected** | **20** | **+0,072** | 0,098 | **0,616** | 0,026 |

> X_original có MCC âm (kém hơn ngẫu nhiên). X_selected đạt MCC cao nhất nhưng SD = 0,098 là lớn, cho thấy kết quả không ổn định qua các fold; khoảng tin cậy rộng, cần thận trọng khi diễn giải.

Wilcoxon (X_full > X_original, cross-project, RF): delta-MCC = +0,078, p = 0,063 (n=5, chỉ mang tính chỉ báo)

### Random-split (cận trên, không phải kết quả chính)

| Tập features | MCC | F1 | AUC |
|---|---|---|---|
| X_original | 0,266 | 0,666 | 0,703 |
| X_new_only | 0,316 | 0,686 | 0,728 |
| X_full | 0,321 | 0,688 | 0,732 |
| **X_selected** | **0,328** | **0,692** | 0,728 |

> Xu hướng tăng rõ và nhất quán từ X_original lên X_selected. MCC 0,33 là mức khả quan trong random-split, nhưng khoảng cách lớn so với cross-project (0,07) cho thấy random-split bị ảnh hưởng bởi project leakage giữa train/test.

### MCC cross-project theo classifier

| | DT | LR | RF | SVM | XGB |
|---|---|---|---|---|---|
| X_original | -0,015 | +0,052 | -0,060 | +0,021 | -0,010 |
| X_new_only | +0,018 | -0,009 | +0,023 | -0,002 | +0,027 |
| X_full | +0,005 | +0,015 | +0,017 | +0,018 | +0,066 |
| X_selected | +0,044 | +0,072 | +0,072 | +0,067 | **+0,070** |

> X_selected là tập duy nhất có MCC dương trên tất cả 5 classifier. X_original âm ở RF, DT, XGB. X_new_only và X_full không nhất quán (LR và SVM có giá trị âm hoặc sát 0).

---

## RQ2 - Đóng góp của từng nhóm features (RF, cross-project)

### Category-add (X_original + một nhóm)

| Nhóm thêm | delta-MCC | delta-F1 |
|---|---|---|
| AST distribution | +0,097 | +0,042 |
| Dependency | +0,076 | +0,030 |
| Test semantic ext | +0,071 | +0,022 |
| Lexical dist | +0,047 | +0,024 |
| CK extended | +0,042 | +0,009 |
| CK method dist | +0,041 | +0,020 |
| CK novel | +0,040 | +0,010 |
| Lexical | +0,032 | +0,014 |
| Test semantic | +0,021 | +0,005 |
| AST structural | -0,009 | +0,002 |

> AST distribution, Dependency và Test semantic ext là 3 nhóm mang lại delta-MCC lớn nhất khi thêm vào X_original. AST structural là nhóm duy nhất có delta-MCC âm, gợi ý nhóm này thêm nhiễu hơn signal trong cross-project setting.

### Leave-one-out (bỏ một nhóm khỏi X_full)

MCC càng thấp = nhóm đó càng quan trọng

| Nhóm bị bỏ | MCC còn lại | Delta vs X_full |
|---|---|---|
| CK extended | 0,011 | -0,006 |
| Dependency | 0,014 | -0,003 |
| AST structural | 0,031 | +0,014 |
| AST distribution | 0,032 | +0,015 |
| Lexical | 0,036 | +0,019 |
| Lexical dist | 0,040 | +0,023 |
| Test semantic | 0,040 | +0,023 |
| Test semantic ext | 0,044 | +0,027 |
| CK novel | 0,046 | +0,029 |
| CK method dist | 0,064 | +0,047 |

> Kết quả LOO mâu thuẫn nhẹ với category-add: CK extended quan trọng nhất trong LOO nhưng đứng thứ 5 trong category-add; AST distribution quan trọng nhất trong category-add nhưng chỉ đứng thứ 4 trong LOO. Điều này cho thấy tầm quan trọng của các nhóm phụ thuộc vào context (thêm vào từ nền gốc vs. bỏ ra khỏi full set).

---

## RQ3 - Mức độ quan trọng của features (combined rank)

Top 15 (12/15 là features mới):

| Rank | Feature | Nhóm |
|---|---|---|
| 1 | lex_mean_comment_length | lexical_dist |
| 2 | ast_depth_std | ast_distribution |
| 3 | ast_node_type_entropy | ast_structural |
| 4 | dep_annotation_type_diversity | dependency |
| 5 | test_fluent_assertion_chain_mean | test_semantic_ext |
| 6 | ast_branching_std | ast_distribution |
| 7 | ast_statement_ratio | ast_distribution |
| 8 | ast_expression_ratio | ast_distribution |
| 9 | lex_variable_identifier_entropy | lexical_dist |
| 10 | LOC | original |
| 11 | test_mock_call_ratio | test_semantic |
| 12 | codeChurnMax | original |
| 13 | lex_comment_density | lexical |
| 14 | ast_annotation_ratio | ast_distribution |
| 15 | ck_method_cbo_mean | ck_method_dist |

> Nhóm ast_distribution chiếm 5/15 vị trí (rank 2, 6, 7, 8, 14) và lexical_dist chiếm 2 vị trí đầu-sau (rank 1, 9). Features gốc còn lại trong top-15 chỉ có LOC (rank 10) và codeChurnMax (rank 12), cho thấy size và churn vẫn giữ giá trị nhưng không thống trị.

---

## RQ4 - Tập features nhỏ gọn

K_selected = 20 (từ 120 features, giảm 83%)

Quy tắc chọn: K nhỏ nhất thoả mean_MCC(K) + SE(K) >= mean_MCC(X_full), đo trên random-split RF.

| K | MCC (random-split) | SE |
|---|---|---|
| 10 | 0,293 | 0,010 |
| **20** | **0,327** | 0,010 |
| 35 | 0,302 | 0,008 |
| 75 | 0,326 | 0,012 |
| 120 | 0,330 | 0,011 |

> MCC không tăng đơn điệu khi tăng K: K=35 (0,302) thấp hơn K=20 (0,327), cho thấy các features thứ yếu thêm nhiễu. K=20 là điểm cân bằng giữa compactness và hiệu suất trong random-split; hành vi cross-project có thể khác.

20 features được chọn: ast_depth_std, ast_node_type_entropy, lex_mean_comment_length, dep_annotation_type_diversity, test_fluent_assertion_chain_mean, ast_branching_std, ast_statement_ratio, ast_expression_ratio, lex_variable_identifier_entropy, LOC, test_mock_call_ratio, codeChurnMax, lex_comment_density, ast_annotation_ratio, ck_method_cbo_mean, ast_mean_branching, ast_mean_call_chain, addLinesMax, removedLinesMax, AsD

---

## Nhận xét kết quả

### MCC cross-project thấp: có phù hợp với literature không?

Có. Kết quả MCC cross-project gần 0 đã được ghi nhận trong các bài toán tương tự:

- **Pontillo et al. (Empirical Software Engineering, 2024)** "Machine learning-based test smell detection" (Valeria Pontillo, Dario Amoroso d'Aragona, Fabiano Pecorelli, Dario Di Nucci, Filomena Ferrucci, Fabio Palomba): cross-project MCC của các cấu hình tốt nhất nằm trong khoảng -0,01 đến 0,30 (theo search snippet; chưa verify trực tiếp từ full text published). Bài toán khác (test smell, không phải refactoring prediction), dataset khác, nên không phải benchmark trực tiếp. [Link](https://link.springer.com/article/10.1007/s10664-023-10436-2)
- **Hoai, Van & Binh (Springer, 2026)** "Enhancing Cross-Project Test Smell Detection via Test Code Feature Engineering and Ensemble Learning": cho thấy feature engineering cải thiện F1 cross-project so với baseline; abstract tuyên bố cải thiện đáng kể so với các nghiên cứu cross-project trước. [Link](https://link.springer.com/chapter/10.1007/978-3-032-21631-1_43)
- **Zimmermann et al. (FSE/ESEC 2009)** về cross-project defect prediction: chỉ 21/622 cặp project (khoảng 3,4%) cho kết quả cross-project chấp nhận được. Paper chỉ ra nhiều yếu tố (khác biệt dataset, domain, process) chứ không quy toàn bộ về một nguyên nhân duy nhất. [Link](https://doi.org/10.1145/1595696.1595713)

**Kết luận:** MCC cross-project gần 0 đã được ghi nhận trong cross-project test-smell detection (Pontillo et al.) và cross-project defect prediction (Zimmermann et al.). Kết quả nghiên cứu này (MCC tối đa 0,07, RF) nằm trong khoảng cross-project của Pontillo et al., dù hai nghiên cứu khác task và dataset.

**Khoảng cách cross-project vs random-split** (MCC 0,07 vs 0,33) cho thấy random-split lạc quan hơn group-split đáng kể; khoảng cách này có thể đến từ nhiều yếu tố (distribution shift, project identity leakage, độ khó fold) và chưa thể cô lập được nguyên nhân cụ thể.

### Điểm mạnh của kết quả

1. **X_selected đạt MCC cao hơn X_original trên cả 5 classifier** (về mặt mô tả): là xu hướng nhất quán. X_new_only và X_full không nhất quán trên tất cả classifier. Chưa đủ bằng chứng thống kê để kết luận (Wilcoxon p=0,063, n=5).
2. **X_selected (20) đạt hiệu suất tương tự X_full (120) về mặt mô tả**: MCC chênh 0,005 ở random-split. "Tương đương" theo nghĩa thống kê cần equivalence test; kết quả hiện tại chỉ cho thấy descriptively comparable.
3. AST distribution và Dependency là nhóm có đóng góp mô tả rõ nhất khi thêm vào X_original (delta-MCC +0,097 và +0,076).
4. CK extended và Dependency quan trọng nhất trong LOO (MCC giảm nhiều nhất khi bỏ ra).

---

### Điểm yếu

- **MCC cross-project gần 0:** absolute performance thấp -> Cần framing rõ: đây là feature contribution study, không phải claim high-performance predictor.
- **Chỉ 1 dataset (PRE_CK):** không thể claim generalizability. Rank C có thể chấp nhận nhưng sẽ là limitation rõ ràng.
- **p = 0,063, n = 5:** chưa đủ bằng chứng thống kê. Với n=5 fold, Wilcoxon rất yếu. Cần thừa nhận là exploratory.
- **X_new_only và X_full không cải thiện nhất quán** trên tất cả classifier: chỉ X_selected mới nhất quán. Nếu claim "new features improve performance" sẽ bị phản bác.

---

## Chi tiết metrics theo classifier

**In đậm** = cao nhất trong hàng. "=" = Dummy không dùng features nên kết quả như nhau với mọi tập. Số là trung bình 5 fold.

### Cross-project: MCC

| Classifier | X_original | X_full | X_selected | X_sel > X_orig? | X_sel > X_full? |
|---|---|---|---|---|---|
| Dummy | -0,024 | -0,024 | -0,024 | = | = |
| DecisionTree | -0,015 | +0,005 | **+0,044** | Yes | Yes |
| LogisticRegression | +0,052 | +0,015 | **+0,072** | Yes | Yes |
| RandomForest | -0,060 | +0,017 | **+0,072** | Yes | Yes |
| SVM_RBF | +0,021 | +0,018 | **+0,067** | Yes | Yes |
| XGBoost | -0,010 | +0,066 | **+0,070** | Yes | Yes |

> X_selected có MCC cao nhất trên cả 5 classifier thực sự (cross-project). X_original RF và DT còn kém cả Dummy.

### Cross-project: F1

| Classifier | X_original | X_full | X_selected | X_sel > X_orig? | X_sel > X_full? |
|---|---|---|---|---|---|
| Dummy | 0,506 | 0,506 | 0,506 | = | = |
| DecisionTree | 0,507 | 0,536 | **0,556** | Yes | Yes |
| LogisticRegression | 0,562 | 0,533 | **0,584** | Yes | Yes |
| RandomForest | 0,546 | 0,581 | **0,616** | Yes | Yes |
| SVM_RBF | 0,599 | 0,574 | **0,608** | Yes | Yes |
| XGBoost | 0,541 | 0,576 | **0,577** | Yes | Yes |

> X_original DT (0,507) gần sát Dummy (0,506) về F1. X_selected cải thiện rõ nhất ở RF (+0,070 so với X_original).

### Cross-project: AUC

| Classifier | X_original | X_full | X_selected | X_sel > X_orig? | X_sel > X_full? |
|---|---|---|---|---|---|
| Dummy | 0,488 | 0,488 | 0,488 | = | = |
| DecisionTree | 0,493 | 0,502 | **0,522** | Yes | Yes |
| LogisticRegression | 0,542 | 0,506 | **0,558** | Yes | Yes |
| RandomForest | 0,467 | 0,517 | **0,556** | Yes | Yes |
| SVM_RBF | 0,526 | 0,516 | **0,545** | Yes | Yes |
| XGBoost | 0,492 | **0,540** | 0,551 | Yes | Yes |

> X_original RF (0,467) thấp hơn cả Dummy (0,488) về AUC. X_selected đưa RF lên 0,556.

### Random-split: MCC

| Classifier | X_original | X_full | X_selected | X_sel > X_orig? | X_sel > X_full? |
|---|---|---|---|---|---|
| Dummy | 0,008 | 0,008 | 0,008 | = | = |
| DecisionTree | 0,225 | **0,268** | 0,234 | Yes | No |
| LogisticRegression | 0,163 | **0,226** | 0,163 | No | No |
| RandomForest | 0,266 | 0,321 | **0,328** | Yes | Yes |
| SVM_RBF | 0,199 | **0,301** | 0,236 | Yes | No |
| XGBoost | 0,246 | **0,325** | 0,299 | Yes | No |

> Random-split: X_selected chỉ vượt X_full ở RF (0,328 vs 0,321). 4/5 classifier X_full tốt hơn. Feature selection được tối ưu trên RF x random-split nên X_selected phù hợp nhất với RF.

### Random-split: F1

| Classifier | X_original | X_full | X_selected | X_sel > X_orig? | X_sel > X_full? |
|---|---|---|---|---|---|
| Dummy | 0,522 | 0,522 | 0,522 | = | = |
| DecisionTree | 0,622 | **0,646** | 0,631 | Yes | No |
| LogisticRegression | 0,629 | **0,650** | 0,628 | No | No |
| RandomForest | 0,666 | 0,688 | **0,692** | Yes | Yes |
| SVM_RBF | 0,669 | **0,682** | 0,670 | Yes | No |
| XGBoost | 0,646 | **0,683** | 0,670 | Yes | No |

### Random-split: AUC

| Classifier | X_original | X_full | X_selected | X_sel > X_orig? | X_sel > X_full? |
|---|---|---|---|---|---|
| Dummy | 0,504 | 0,504 | 0,504 | = | = |
| DecisionTree | 0,613 | **0,634** | 0,617 | Yes | No |
| LogisticRegression | 0,613 | **0,660** | 0,626 | Yes | No |
| RandomForest | 0,703 | **0,732** | 0,728 | Yes | No |
| SVM_RBF | 0,626 | **0,710** | 0,675 | Yes | No |
| XGBoost | 0,686 | **0,733** | 0,717 | Yes | No |

> Random-split AUC: X_full tốt hơn X_selected ở cả 5 classifier. X_selected vẫn tốt hơn X_original trên tất cả.

### Tom tat: X_selected co cai thien khong? (khong tinh Dummy)

| Protocol | Metric | X_sel > X_orig (5 clf) | X_sel > X_full (5 clf) |
|---|---|---|---|
| Cross-project | MCC | 5/5 | 5/5 |
| Cross-project | F1 | 5/5 | 5/5 |
| Cross-project | AUC | 5/5 | 5/5 |
| Random-split | MCC | 4/5 | 1/5 |
| Random-split | F1 | 4/5 | 1/5 |
| Random-split | AUC | 5/5 | 0/5 |

> X_selected cải thiện nhất quán so với X_original trong cross-project (15/15). So với X_full, X_selected tốt hơn trong cross-project (15/15) nhưng kém hơn trong random-split (2/15). Nguyên nhân: feature selection được thực hiện trên RF x random-split, nên X_selected chỉ tối ưu cho RF, không cho các classifier khác.
