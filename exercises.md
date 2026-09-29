# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Lương Sỹ Khánh  Mã học viên: 2A202602715

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu quên set API key khi deploy mà có mặc định `"changeme"`, app vẫn chạy bình
> thường nhưng ai cũng đoán được khóa. Fail fast làm app dừng ngay khi khởi động
> nên lỗi được phát hiện lúc deploy, trước khi bị lộ.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> ```
> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T03:28:28.665953+00:00", "user_id": "sv-khanh", "tokens_in": 45, "tokens_out": 47, "cost_usd": 3.495e-05}
> ```
>
> Lọc và thống kê theo từng trường (ví dụ tổng chi phí của một user), và truy lại
> sự cố theo thời gian và user.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ... MB |
| Multi-stage | ... MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> *Câu trả lời của bạn*

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Chỉ hai layer `COPY app/` và `COPY utils/` chạy lại (khoảng 0.2 giây), các layer
> tạo venv và `pip install` đều dùng cache. Nếu `COPY . .` đứng trước `pip install`
> thì sửa code làm mất cache từ bước đó, mỗi lần build lại phải cài lại toàn bộ
> thư viện.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Lỗ hổng cho kẻ tấn công chạy lệnh trong container. Nếu là root, họ có toàn quyền
> trong container và dễ thoát ra host với quyền root. `USER` khiến lệnh đó chỉ chạy
> với quyền user thường, nên thiệt hại bị giới hạn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> 20 request: gửi 10 ở cuối phút trước và 10 ngay đầu phút sau, khi bộ đếm vừa
> reset. Sliding window vẫn tính 10 request trước nên chặn được.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request, cost guard giới hạn số tiền. Gọi chậm nhưng câu
> hỏi dài, tiêu hết ngân sách thì cost guard chặn. Gửi dồn nhiều request trong một
> phút khi ngân sách còn nhiều thì rate limit chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối thì cả 3 container cùng báo lỗi, load balancer không còn chỗ
> gửi traffic, rồi orchestrator restart cả 3. Khi Redis có lại thì app vẫn đang
> khởi động, nên downtime kéo dài hơn. Tách riêng thì chỉ `/ready` báo lỗi,
> container không bị restart.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Gọi lần lượt vào 3 instance, `history_length` tăng đều 0, 2, 4, 6 vì cả 3 dùng
> chung Redis. Nếu lưu trong dict, mỗi instance có lịch sử riêng nên con số nhảy
> lung tung tùy request rơi vào instance nào, ví dụ 0, 0, 0, 2.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Mình không tạo được token deploy trên Railway (lỗi "Not Authorized"). Có vẻ
> tài khoản dùng thử bị giới hạn quyền, nên mình chuyển sang Render
> và deploy qua Deploy Hook.
