# Báo cáo 

Đặng Hữu Nghĩa - 2A202601005
## Bộ siêu tham số đã chọn

Mô hình được chọn là `RandomForestClassifier` với các siêu tham số:

```yaml
n_estimators: 500
max_depth: 20
min_samples_split: 2
```

Trong Bước 1, em đã chạy nhiều thí nghiệm với các giá trị khác nhau cho `n_estimators`, `max_depth` và `min_samples_split`, sau đó so sánh kết quả trên MLflow bằng hai chỉ số `accuracy` và `f1_score`. Bộ tham số trên được chọn vì cho kết quả tốt hơn các cấu hình nhỏ hơn như `n_estimators: 100`, `max_depth: 5`. Trên tập `train_phase1.csv`, cấu hình này đạt khoảng `accuracy = 0.6780` và `f1_score = 0.6766`. Sau khi bổ sung thêm dữ liệu ở Bước 3, cùng cấu hình này cho kết quả cao hơn và vượt ngưỡng eval gate `0.70`, nên phù hợp hơn cho pipeline tự động.

Lý do chọn:

- `n_estimators: 500` giúp mô hình ổn định hơn so với số lượng cây nhỏ.
- `max_depth: 20` cho phép cây học được các quan hệ phức tạp hơn trong dữ liệu Wine Quality, nhưng vẫn giới hạn độ sâu để giảm overfitting.
- `min_samples_split: 2` giữ khả năng tách nút linh hoạt khi dữ liệu có đủ mẫu.

## Khó khăn và cách giải quyết

Khó khăn đầu tiên là chuyển hướng dẫn từ GCP sang AWS. Ban đầu `serve.py` và workflow còn theo GCS, nên em đã đổi sang S3: thay `google-cloud-storage` bằng `boto3`, đổi biến môi trường thành `S3_BUCKET`, và upload/tải model tại đường dẫn `models/latest/model.pkl`.

Khó khăn tiếp theo là cấu hình EC2. Em gặp lỗi không truy cập được endpoint từ máy local qua port `8000`. Cách xử lý là kiểm tra service bằng `sudo systemctl status mlops-serve`, xem log bằng `sudo journalctl -u mlops-serve -n 80 --no-pager`, test nội bộ bằng `curl http://localhost:8000/health`, và mở inbound rule `Custom TCP 8000` trong Security Group.

Khi chạy GitHub Actions deploy, em gặp lỗi SSH private key không hợp lệ (`ssh.ParsePrivateKey: ssh: no key found`). Nguyên nhân là secret `VM_SSH_KEY` chưa đúng nội dung private key. Em đã copy lại toàn bộ nội dung file `.pem` bằng PowerShell và cập nhật lại GitHub Secret.

Ngoài ra, khi test trên Windows PowerShell, lệnh `curl` với biến `$env:VM_IP` bị parse sai trước dấu `:`. Em đã sửa bằng cách dùng `"http://${env:VM_IP}:8000/health"` hoặc dùng trực tiếp IP trong URL.

Sau khi hoàn thành, pipeline có thể huấn luyện mô hình, upload model lên S3, restart service trên EC2 và phục vụ dự đoán qua hai endpoint `/health` và `/predict`.
