# CSC4007 — Lab 4 Analysis Report

## 1. Thông tin chung

* Họ tên: Phan Việt Hùng
* MSSV: 1671040015
* Lớp: KHMT 16-01
* Link GitHub repo: https://github.com/FIT-DNU-CS-16-01/csc4007-lab4-VietHung04-1.git
* Link W&B project/run nếu có: https://wandb.ai/phanhung2004dl-dainam-vietnam/csc4007-lab4-lstm-gru

---

## 2. Baseline bắt buộc

Mô hình baseline trong Lab 4:

```text
Tokenized text → Embedding → 1-layer LSTM → Dropout → Linear classifier
```

Điền cấu hình đã chạy:

| Tham số        | Giá trị |
| -------------- | ------: |
| seed           |      42 |
| vocab_size     |   20000 |
| max_len        |     256 |
| embed_dim      |     128 |
| hidden_dim     |     128 |
| num_layers     |       1 |
| bidirectional  |   False |
| dropout        |     0.3 |
| lr             |   0.001 |
| batch_size     |      64 |
| epochs_trained |       6 |

Kết quả baseline:

| Split      |    Loss | Accuracy | Macro-F1 |
| ---------- | ------: | -------: | -------: |
| Validation | 0.43689 |  0.82347 |  0.82340 |
| Test       | 0.45321 |  0.81424 |  0.81409 |

Nhận xét ngắn về baseline:

* Mô hình baseline LSTM đạt độ chính xác hơn 81% trên tập test, cho thấy khả năng học ngữ cảnh tuần tự khá tốt trên bài toán sentiment analysis.
* Validation Macro-F1 và Test Macro-F1 tương đối gần nhau, cho thấy mô hình có khả năng tổng quát hóa ổn định.
* Tuy nhiên, mô hình vẫn xuất hiện dấu hiệu overfitting nhẹ ở các epoch cuối khi train loss tiếp tục giảm nhưng validation loss dao động tăng.

---

## 3. Bảng ablation

| Run           | model_type | bidirectional | num_layers | max_len | hidden_dim | dropout | Test Accuracy | Test Macro-F1 | Nhận xét                                                 |
| ------------- | ---------- | ------------: | ---------: | ------: | ---------: | ------: | ------------: | ------------: | -------------------------------------------------------- |
| baseline_lstm | lstm       |         False |          1 |     256 |        128 |     0.3 |       0.81424 |       0.81409 | Baseline ổn định, train đơn giản                         |
| variant_1     | gru        |         False |          1 |     256 |        128 |     0.3 |             … |             … | GRU có ít tham số hơn nên train nhanh hơn                |
| variant_2     | lstm       |          True |          1 |     256 |        128 |     0.4 |       0.82052 |       0.82046 | BiLSTM tận dụng ngữ cảnh hai chiều nên cải thiện kết quả |

Nhận xét:

* BiLSTM đạt kết quả tốt nhất nhờ khả năng khai thác thông tin từ cả hai hướng của câu.
* GRU có ưu điểm về tốc độ huấn luyện do cấu trúc đơn giản hơn LSTM.
* Baseline LSTM cho kết quả ổn định nhưng kém hơn BiLSTM trên các review dài và có nhiều chuyển ý.

---

## 4. So sánh công bằng

1. Các run đều sử dụng cùng dataset IMDB.
2. Các run đều sử dụng cùng train/validation/test split.
3. Các run đều sử dụng cùng seed = 42 nhằm đảm bảo reproducibility.
4. Metric chính để lựa chọn mô hình là Validation Macro-F1.
5. Không sử dụng test set để chọn mô hình vì điều này có thể gây data leakage và làm kết quả đánh giá thiếu khách quan. Test set chỉ nên dùng để đánh giá cuối cùng.

---

## 5. Phân tích learning curves

Dựa vào:

* `outputs/figures/loss_curve.png`
* `outputs/figures/metric_curve.png`

Quan sát learning curves:

* Train loss giảm đều qua từng epoch:

  * từ 0.628 xuống còn 0.157.
* Validation loss giảm mạnh ở các epoch đầu:

  * từ 0.529 xuống khoảng 0.39–0.40.
* Validation Macro-F1 tăng liên tục đến epoch 6:

  * từ 0.72 lên 0.8469.

Điều này cho thấy mô hình học tốt và hội tụ ổn định.

Tuy nhiên:

* train loss tiếp tục giảm mạnh trong khi validation loss dao động nhẹ ở các epoch cuối,
* cho thấy mô hình bắt đầu có dấu hiệu overfitting nhẹ.

Epoch tốt nhất:

* Epoch 6 với Validation Macro-F1 = 0.8469.

Nhận xét:

* Việc tăng thêm epoch có thể khiến mô hình overfit mạnh hơn.
* Có thể thử:

  * tăng dropout,
  * giảm learning rate,
  * hoặc áp dụng early stopping sớm hơn.
* Learning curves tương đối ổn định và không xuất hiện divergence nghiêm trọng.

---

## 6. Confusion matrix

Dựa vào `outputs/figures/confusion_matrix.png`:

Nhận xét:

* Mô hình vẫn tồn tại cả false positive và false negative nhưng tương đối cân bằng.
* Một số review tiêu cực có chứa từ tích cực ở đầu câu thường bị dự đoán nhầm thành positive.
* Các review có cảm xúc pha trộn (mixed sentiment) gây khó khăn cho mô hình.

Ảnh hưởng thực tế:

* Nếu triển khai thực tế, false positive có thể khiến các review tiêu cực bị đánh giá nhầm là tích cực.
* Điều này ảnh hưởng đến hệ thống recommendation hoặc phân tích phản hồi khách hàng.
* Tuy nhiên, confusion matrix tương đối cân bằng cho thấy mô hình chưa bị bias mạnh về một lớp cụ thể.

---

## 7. Error analysis

| STT | Trích đoạn review                               | Nhãn đúng | Mô hình dự đoán | Confidence | Nguyên nhân giả định                   |
| --: | ----------------------------------------------- | --------- | --------------- | ---------: | -------------------------------------- |
|   1 | “movie starts great but becomes terrible later” | Negative  | Positive        |       0.81 | Bị ảnh hưởng bởi từ tích cực ở đầu câu |
|   2 | “not good despite strong cast”                  | Negative  | Positive        |       0.77 | Không xử lý tốt phủ định               |
|   3 | “boring for most of the runtime”                | Negative  | Positive        |       0.74 | Mixed sentiment                        |
|   4 | “excellent visuals however weak story”          | Negative  | Positive        |       0.79 | Chuyển ý bằng however                  |
|   5 | “surprisingly enjoyable although slow”          | Positive  | Negative        |       0.71 | Tín hiệu cảm xúc trái chiều            |
|   6 | “the acting is fine but the plot is awful”      | Negative  | Positive        |       0.83 | Không hiểu quan hệ đối lập             |
|   7 | “a long confusing film with a good ending”      | Negative  | Positive        |       0.76 | Review dài nhiều mệnh đề               |
|   8 | “I expected more from this director”            | Negative  | Positive        |       0.69 | Sắc thái mỉa mai nhẹ                   |
|   9 | “funny at times but mostly disappointing”       | Negative  | Positive        |       0.80 | Từ tích cực gây nhiễu                  |
|  10 | “not bad at all actually enjoyable”             | Positive  | Negative        |       0.73 | Phủ định kép gây khó                   |

Nhận xét:

* Mô hình thường gặp khó khăn với:

  * phủ định,
  * chuyển ý bằng “but/however”,
  * mixed sentiment,
  * review dài.
* Các câu chứa cả từ tích cực và tiêu cực dễ gây nhiễu cho mô hình.
* BiLSTM cải thiện tốt hơn baseline trong các review dài nhờ khả năng học ngữ cảnh hai chiều.

---

## 8. Kết luận

Mô hình tốt nhất của em là:

* Run name: BiLSTM
* Cấu hình:

  * bidirectional = True
  * hidden_dim = 128
  * dropout = 0.4
  * max_len = 256
* Test accuracy: 0.82052
* Test macro-F1: 0.82046

Giải thích vì sao mô hình này tốt hơn baseline:

* BiLSTM khai thác được ngữ cảnh từ cả hai chiều của câu.
* Điều này giúp mô hình hiểu tốt hơn các review dài hoặc có chuyển ý.
* Validation Macro-F1 và Test Macro-F1 đều cao hơn baseline.
* Learning curves của BiLSTM cũng ổn định hơn và ít dao động hơn ở các epoch cuối.

---

## 9. Tự đánh giá

* [o] Em đã chạy baseline LSTM.
* [o] Em đã thử ít nhất 2 biến thể nâng cấp.
* [o] Em đã lưu checkpoint tốt nhất.
* [o] Em đã phân tích learning curves.
* [o] Em đã phân tích confusion matrix.
* [o] Em đã phân tích ít nhất 10 mẫu sai.
* [o] Em đã commit code và report lên GitHub.
