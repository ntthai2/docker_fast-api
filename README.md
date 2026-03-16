# AI Service

REST API phân tích cảm xúc văn bản xây dựng bằng **FastAPI**, model **TextBlob**, database **PostgreSQL**, triển khai bằng **Docker Compose**.

---

## Tính năng

- Phân tích cảm xúc văn bản (POSITIVE / NEGATIVE) kèm confidence score
- Lưu lịch sử kết quả dự đoán vào PostgreSQL
- Tra cứu kết quả theo ID
- Health check endpoint kiểm tra trạng thái service và database
- Validate input tự động bằng Pydantic (HTTP 422 khi sai)
- CORS configurable qua biến môi trường

---

## Yêu cầu

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (bao gồm Docker Compose v2)

---

## Cấu trúc project

```
ai-service/
├── app/
│   ├── main.py              # Khởi tạo FastAPI app, middleware, health check
│   ├── routers/
│   │   └── api.py           # Định nghĩa endpoints /predict, /result/{id}
│   ├── models/
│   │   ├── db.py            # ORM model (bảng predictions)
│   │   └── schemas.py       # Pydantic schemas cho request / response
│   └── services/
│       ├── core.py          # TextBlob sentiment analysis
│       └── database.py      # SQLAlchemy engine và session
├── .env                     # Biến môi trường (không commit lên git)
├── .dockerignore
├── Dockerfile               # Multi-stage build, non-root user
├── docker-compose.yml       # Orchestration: db + api với healthcheck
└── requirements.txt
```

---

## Cài đặt và chạy

### 1. Tạo file `.env`

```bash
cp .env.example .env
```

### 2. Khởi động

```bash
docker compose up
```

### 3. Dừng service

```bash
docker compose down
```

---

## API Endpoints

Base URL: `http://localhost:8000`

### `GET /health`

Kiểm tra trạng thái service và kết nối database.

**Response**
```json
{
  "status": "ok",
  "database": "connected"
}
```

---

### `POST /predict`

Phân tích cảm xúc một đoạn văn bản bằng TextBlob. Score là giá trị tuyệt đối của polarity (0.0 – 1.0).

**Request body**
```json
{
  "text": "I love this product!"
}
```

| Field | Type   | Ràng buộc      |
|-------|--------|----------------|
| text  | string | bắt buộc, tối đa 500 ký tự |

**Response**
```json
{
  "id": 1,
  "label": "POSITIVE",
  "score": 0.9998
}
```

---

### `GET /result/{prediction_id}`

Lấy lại kết quả dự đoán đã lưu theo ID.

**Response**
```json
{
  "id": 1,
  "input_text": "I love this product!",
  "label": "POSITIVE",
  "score": 0.9998
}
```

Trả về `404` nếu không tìm thấy ID.

---

## Biến môi trường

| Biến               | Mô tả                                         | Ví dụ                                              |
|--------------------|-----------------------------------------------|----------------------------------------------------|
| `POSTGRES_USER`    | Username PostgreSQL                           | `aiuser`                                           |
| `POSTGRES_PASSWORD`| Password PostgreSQL                           | `aipassword`                                       |
| `POSTGRES_DB`      | Tên database                                  | `aidb`                                             |
| `DATABASE_URL`     | Connection string SQLAlchemy                  | `postgresql://aiuser:aipassword@db:5432/aidb`      |
| `ALLOWED_ORIGINS`  | Danh sách CORS origins, phân cách bằng dấu phẩy | `http://localhost:3000,https://yourdomain.com`  |

---

## Model AI

- **Library:** [TextBlob](https://textblob.readthedocs.io/)
- **Task:** Sentiment Analysis (POSITIVE / NEGATIVE)
- **Phương pháp:** Tính `polarity` từ `TextBlob.sentiment`; dương → `POSITIVE`, không dương → `NEGATIVE`; score = `abs(polarity)` ∈ [0, 1]

---

## Stack công nghệ

| Thành phần    | Công nghệ                              |
|---------------|----------------------------------------|
| API framework | FastAPI 0.135                          |
| ASGI server   | Uvicorn 0.41                           |
| AI model      | TextBlob 0.18                          |
| ORM           | SQLAlchemy 2.0                         |
| Database      | PostgreSQL 16                          |
| Container     | Docker (python:3.11-slim, multi-stage) |
| Orchestration | Docker Compose v2                      |
