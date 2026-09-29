# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng mẫu dưới mỗi câu bằng câu trả lời của bạn.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Ngô Thế Khanh  |  Mã học viên: 2A202602503

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Ví dụ em push code lên Railway nhưng quên đặt `AGENT_API_KEY`. Nếu app có mặc định `changeme`, container vẫn khởi động và người ngoài có thể đoán khóa đó để gọi `/ask`; em chỉ nhận ra sau khi có request lạ hoặc phát sinh chi phí. Không có mặc định thì `Settings` báo thiếu key ngay lúc khởi động, deployment fail rõ ràng trước khi service nhận traffic.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Log thực tế thu được sau khi gọi `POST /ask` ở local:
> `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T05:16:15.327167+00:00", "user_id": "sv-bao-cao", "tokens_in": 6, "tokens_out": 44, "cost_usd": 2.73e-05}`
> Em có thể lọc log theo `event`/`user_id` để tìm request, và cộng `tokens_in`, `tokens_out`, `cost_usd` theo thời gian để theo dõi sử dụng/chi phí. Câu `print("đã trả lời xong")` không có các trường đó để lọc hoặc tổng hợp.

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
| 1 stage (bản đầu) | 1,731.7 MB (1.73 GB) |
| Multi-stage | 275.0 MB (275 MB) |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Em build Dockerfile 1 stage gốc (`python:3.11`, cài requirements trong cùng image) thành `day12-agent:single-original`: `docker images` báo 1.73 GB (1,731.7 MB theo `docker image inspect`). Bản multi-stage `day12-agent:cp2-test` là 275 MB (275.0 MB). Bản sau nhỏ hơn 1,456.7 MB, khoảng 84.1%. Chênh lệch chủ yếu do base image đầy đủ và pip cache của bản cũ; bản mới dùng base `slim`, cài dependency ở builder rồi chỉ chép dependency cần chạy sang runtime, đồng thời không giữ pip cache.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa `app/main.py`, hai layer `COPY requirements.txt` và `RUN pip install` trong stage `builder` không đổi nên Docker dùng lại cache. Ở stage runtime, layer `COPY --from=builder /install /usr/local` cũng có thể dùng lại; `COPY . .` thay đổi nên layer source và `RUN useradd ... chown ...` phía sau phải chạy lại. Nếu `COPY . .` đặt trước `RUN pip install`, mọi sửa source làm layer COPY đổi và pip phải cài lại toàn bộ dependency.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗ hổng cho phép chạy lệnh từ xa sẽ khiến mã khai thác chạy với quyền của process Python. Nếu process chạy root thì kẻ tấn công có quyền cao trong container; nếu còn có cấu hình nguy hiểm như privileged mode, Docker socket hoặc mount thư mục host, họ có thể lợi dụng để ảnh hưởng host. `USER appuser` làm process chỉ có quyền thường, giảm quyền truy cập file/process và giảm mức độ thiệt hại. Nó giảm rủi ro, không thay thế hoàn toàn cách ly container/kernel.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Với fixed window, em có thể gửi 10 request lúc 12:00:59 rồi thêm 10 request lúc 12:01:00–12:01:01, vì bộ đếm vừa reset sang phút mới. Như vậy đạt 20 request trong khoảng 2 giây dù giới hạn là 10/phút. Sliding window luôn đếm 60 giây gần nhất nên không cho phép vượt kiểu này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit chặn theo số request trong 60 giây; cost guard chặn theo tổng USD của user trong tháng. Nếu tổng chi phí các request trước đã vượt budget nhưng em vẫn còn quota trong phút này, rate limit cho qua còn cost guard trả 402. Ngược lại, khi em gửi quá 10 request trong một phút nhưng tổng tiền tháng vẫn thấp hơn budget, rate limit trả 429 còn cost guard chưa cần chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối → endpoint gộp trả 503 → liveness probe đánh dấu cả 3 container unhealthy và nền tảng restart chúng → Redis vẫn mất nên các container mới tiếp tục fail probe, tạo vòng lặp restart và làm cụm không phục vụ được trong 30 giây đó. Nếu tách endpoint, `/health` vẫn 200 để không restart process, còn `/ready` 503 để load balancer tạm ngừng gửi traffic.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis, mỗi lần gọi `/ask` ghi hai message dùng chung cho mọi instance; các giá trị `history_length` thường lần lượt là 0, 2, 4, ... Nếu dùng dict riêng trong RAM, mỗi container có lịch sử riêng: request đầu tiên vào container nào cũng có thể trả `0`, các giá trị tăng theo từng container và có thể quay về `0` khi load balancer chuyển sang container khác.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lần deploy Railway đầu tiên crash lúc startup với `NotImplementedError: TODO (CP4): cài đặt` trong phần đăng ký signal handler của lifecycle. Em mở Deploy Logs, đọc traceback tới `lifecycle.install()` rồi đối chiếu với TODO và chạy lại test CP4. Sau khi cài handler `SIGTERM`/`SIGINT` và hoàn thiện CP4, em deploy lại; Railway báo deployment `ACTIVE`, service `Online`, `/health` và `/ready` trả 200.
