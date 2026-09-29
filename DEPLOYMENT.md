# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Ngô Thế Khanh |
| Mã học viên | 2A202602503 |
| Repo | https://github.com/khanhken159/K4-L3B-DAY12-Ng-Th-Khanh-2A202602503-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://k4-l3b-day12-ng-th-khanh-2a202602503-cloudservic-production.up.railway.app |
| Platform | Railway |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Railway tự cấp |
| `AGENT_API_KEY` | ✅ | đặt trong Variables của app; không ghi giá trị vào repo |
| `REDIS_URL` | ✅ | tham chiếu `REDIS_URL` từ dịch vụ Redis trên Railway |
| `RATE_LIMIT_PER_MINUTE` | Mặc định | 10 trong cấu hình app |
| `MONTHLY_BUDGET_USD` | Mặc định | 10.0 trong cấu hình app |
| `LOG_LEVEL` | Mặc định | INFO trong cấu hình app |

## Bonus — CI/CD GitHub Actions

Workflow `.github/workflows/ci.yml` chạy CP1–CP4 và build Docker image trên cả
push lên `main` lẫn pull request vào `main`. CP5 bị loại khỏi CI vì nó gọi
Railway thật; bài kiểm tra badge được chạy sau khi workflow đã hoàn tất để
tránh tự tham chiếu. Job deploy phụ thuộc cả job test và build, chỉ chạy với
push vào `main`.

Để bật job deploy:

1. Tạo Railway **Project Token** và lưu thành GitHub repository secret
   `RAILWAY_TOKEN` (Settings → Secrets and variables → Actions).
2. Thêm repository variable `RAILWAY_DEPLOY_ENABLED` với giá trị `true`.
3. Vì Railway service hiện cũng liên kết trực tiếp với GitHub, tắt
   **automatic deployments** của Railway để tránh deploy trùng. Sau đó job
   deploy trong workflow là đường deploy duy nhất.

Workflow gọi Railway CLI bằng token secret, triển khai đúng project/service,
rồi gọi `/health` để xác nhận service đã lên. Token không nằm trong repository.
Hướng dẫn chính thức: [Railway CLI](https://docs.railway.com/cli) và
[điều khiển GitHub autodeploys](https://docs.railway.com/deployments/github-autodeploys).

## Lệnh Kiểm Tra

Thay `<URL>` bằng Public URL ở trên:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i <URL>/health

# 2. Readiness — mong đợi 200 {"status":"ready","redis":true}
curl -i <URL>/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST <URL>/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST <URL>/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật — 2026-09-29

```
GET /health → 200 {"status":"ok","service":"day12-agent","version":"1.0.0"}
GET /ready  → 200 {"status":"ready","redis":true}
POST /ask không có API key → 401
pytest tests/test_cp5.py -v → 9 passed, 4 skipped (4 test local fallback không áp dụng)
```

Lần deploy đầu từng crash lúc khởi động do `NotImplementedError: TODO (CP4): cài đặt`
trong phần cài handler tắt tiến trình. CP4 đã được hoàn thiện, sau đó Railway
hiển thị deployment `ACTIVE` và service `Online`.

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl
- `screenshots/ready.png` — kết quả `/ready`, xác nhận Redis đã kết nối

---

## Nếu Dùng Phương Án Dự Phòng

Không đăng ký được tài khoản cloud? Vẫn nộp được bài, nhưng CP5 tối đa 60% điểm:

1. Đặt `LOCAL_FALLBACK=true` trong `.env`
2. Chạy `docker compose up -d` rồi kiểm tra `docker compose ps`
3. Chụp màn hình vào `screenshots/`
4. Chạy `pytest tests/test_cp5.py -v` — bộ test sẽ tự chuyển sang kiểm tra
   `http://localhost:8000`
5. Ghi rõ lý do không deploy được vào phần dưới đây:

```
Không dùng phương án dự phòng; service được triển khai trên Railway.
```
