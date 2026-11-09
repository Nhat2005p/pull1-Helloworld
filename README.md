# Pull 1: Init Project NestJS + Hello World API + Setup i18n + Setup Swagger

> **Bài thực hành Pull 1 - Module NestJS Tutorial (NodeJS Course - Sun Asterisk)**
>
> **Mentor / Trainer phụ trách:** Phạm Thanh Bình (HCM_Trainer)
> **Tài liệu tham khảo:**
> - [NestJS Official Documentation](https://docs.nestjs.com/)
> - [Sun* Training Guideline - Node.js Tutorial](https://github.com/framgia/Training-Guideline/blob/master/Nodejs/tutorial.md)
> - [nestjs-i18n Documentation](https://nestjs-i18n.com/)
> - [@nestjs/swagger Documentation](https://docs.nestjs.com/openapi/introduction)

---

## 📌 Nội Dung Mục Tiêu Của Pull 1
Theo yêu cầu phân chia Pull Request từ Trainer:
- [x] **1. Setup được project:** Khởi tạo cấu trúc dự án NestJS v10 chuẩn mực, cấu hình TypeScript, environment `.env`, `nest-cli.json` đồng bộ assets.
- [x] **2. Setup Swagger:** Tích hợp Swagger OpenAPI UI trực quan tại route `/api/docs`, khai báo BearerAuth (`JWT-auth`) chuẩn bị cho các Pull tiếp theo.
- [x] **3. Setup i18n:** Cấu hình thư viện `nestjs-i18n` hỗ trợ đa ngôn ngữ (tiếng Anh `en`, tiếng Việt `vi`, tiếng Nhật `ja`) qua query param, header `x-custom-lang` hoặc `Accept-Language`.
- [x] **4. 1 API Hello World:** Triển khai endpoint `GET /api/v1/hello` đa ngôn ngữ, tự động phản hồi theo ngôn ngữ yêu cầu.
- [x] **5. Kiểm thử tự động:** Viết đầy đủ Unit Tests (`npm test`: 7/7 tests pass) và E2E Tests (`npm run test:e2e`: 3/3 tests pass).

---

## 🛠️ Cấu Trúc Mã Nguồn Dự Án
```text
pull1-Helloworld/
├── src/
│   ├── common/
│   │   ├── decorators/
│   │   │   └── custom.decorators.ts       # Custom Decorators: @Roles() và @User()
│   │   ├── filters/
│   │   │   └── http-exception.filter.ts   # Global Exception Filter chuẩn hóa lỗi JSON
│   │   ├── guards/
│   │   │   └── roles.guard.ts             # Roles Guard bảo vệ endpoint
│   │   ├── interceptors/
│   │   │   ├── logging.interceptor.ts     # Đo thời gian xử lý request bằng RxJS
│   │   │   └── transform.interceptor.ts   # Bọc phản hồi dạng { success, data, ... }
│   │   ├── middleware/
│   │   │   └── logger.middleware.ts       # Middleware HTTP request logger
│   │   └── pipes/
│   │       └── parse-positive-int.pipe.ts # Custom Pipe validate ID số nguyên dương (> 0)
│   ├── i18n/                              # Thư mục từ điển đa ngôn ngữ i18n
│   │   ├── en/
│   │   │   └── app.json                   # Bản dịch tiếng Anh (mặc định)
│   │   ├── ja/
│   │   │   └── app.json                   # Bản dịch tiếng Nhật
│   │   └── vi/
│   │       └── app.json                   # Bản dịch tiếng Việt
│   ├── items/                             # Feature Module minh họa CRUD & Architecture
│   │   ├── dto/
│   │   │   ├── create-item.dto.ts
│   │   │   └── update-item.dto.ts
│   │   ├── items.controller.spec.ts       # Unit tests cho ItemsController
│   │   ├── items.controller.ts            # Controller với Swagger annotations
│   │   ├── items.module.ts
│   │   └── items.service.ts
│   ├── app.controller.spec.ts             # Unit tests cho Hello World & i18n
│   ├── app.controller.ts                  # Controller tiếp nhận Hello World API
│   ├── app.module.ts                      # Root module cấu hình I18nModule & ConfigModule
│   ├── app.service.ts                     # Service xử lý dịch i18n qua I18nService
│   └── main.ts                            # Bootstrap NestJS & cấu hình SwaggerModule
├── test/
│   ├── app.e2e-spec.ts                    # E2E Test kiểm tra API Hello World & i18n
│   └── jest-e2e.json                      # Cấu hình Jest E2E
├── .env.example                           # Biến môi trường mẫu
├── .gitignore                             # Loại bỏ node_modules, dist, cache khỏi Git
├── nest-cli.json                          # Cấu hình tự động sao chép assets i18n
├── package.json                           # Quản lý dependencies & npm scripts
├── README.md                              # Báo cáo chi tiết Pull 1
└── tsconfig.json                          # Cấu hình TypeScript Decorators
```

---

## 🌟 Chi Tiết Triển Khai Kỹ Thuật

### 1. Cấu hình Đa ngôn ngữ (`nestjs-i18n`)
- Trong `src/app.module.ts`, `I18nModule` được thiết lập với 3 bộ phân giải (Resolvers):
  1. `QueryResolver(['lang', 'l'])`: Bắt ngôn ngữ qua URL param (ví dụ: `?lang=vi`).
  2. `HeaderResolver(['x-custom-lang'])`: Bắt ngôn ngữ qua HTTP header tùy biến.
  3. `AcceptLanguageResolver`: Bắt ngôn ngữ theo header chuẩn của trình duyệt.
- `nest-cli.json` được bổ sung cấu hình:
  ```json
  "compilerOptions": {
    "deleteOutDir": true,
    "assets": ["i18n/**/*"],
    "watchAssets": true
  }
  ```
  Giúp toàn bộ thư mục `src/i18n` được tự động đồng bộ sang `dist/i18n` khi build.

### 2. Cấu hình Swagger OpenAPI (`/api/docs`)
- Thiết lập trong `src/main.ts` với `DocumentBuilder`:
  - Khai báo Bearer Auth `JWT-auth`.
  - Phân nhóm Tags: `Hello World` và `Items`.
  - Endpoint truy cập tài liệu UI: `http://localhost:3000/api/docs`.

### 3. API Hello World Đa Ngôn Ngữ (`GET /api/v1/hello`)
- Endpoint: `GET /api/v1/hello`
- Nhận diện ngôn ngữ linh hoạt:
  ```bash
  # Mặc định tiếng Anh:
  curl -X GET http://localhost:3000/api/v1/hello

  # Tiếng Việt:
  curl -X GET "http://localhost:3000/api/v1/hello?lang=vi"

  # Tiếng Nhật:
  curl -X GET "http://localhost:3000/api/v1/hello?lang=ja"
  ```
- Định dạng dữ liệu phản hồi:
  ```json
  {
    "success": true,
    "statusCode": 200,
    "timestamp": "2026-11-09T08:00:00.000Z",
    "data": {
      "message": "Xin chào thế giới!",
      "welcome": "Chào mừng bạn đến với khóa học NestJS Tutorial!",
      "description": "Tính năng đa ngôn ngữ (i18n) và Swagger đã được thiết lập thành công.",
      "currentLanguage": "vi",
      "timestamp": "2026-11-09T08:00:00.000Z"
    }
  }
  ```

---

## 🚀 Hướng Dẫn Cài Đặt & Khởi Chạy

### Bước 1: Cài đặt thư viện (nếu chưa có)
```bash
npm install
```

### Bước 2: Tạo file biến môi trường
```bash
cp .env.example .env
```

### Bước 3: Biên dịch dự án
```bash
npm run build
```

### Bước 4: Khởi chạy Server
```bash
# Chế độ phát triển (Watch mode tự động reload khi sửa code):
npm run start:dev

# Chế độ thông thường:
npm start
```
- 📄 **Swagger UI:** [http://localhost:3000/api/docs](http://localhost:3000/api/docs)
- 🌍 **Hello World API:** [http://localhost:3000/api/v1/hello?lang=vi](http://localhost:3000/api/v1/hello?lang=vi)

---

## 🧪 Kết Quả Kiểm Thử (Unit Tests & E2E Tests)

### 1. Chạy Unit Tests
```bash
npm test
```
```text
PASS src/app.controller.spec.ts
PASS src/items/items.controller.spec.ts

Test Suites: 2 passed, 2 total
Tests:       7 passed, 7 total
Snapshots:   0 total
```

### 2. Chạy E2E Tests
```bash
npm run test:e2e
```
```text
PASS test/app.e2e-spec.ts
  AppController (e2e)
    √ / (GET) - Overview (2601 ms)
    √ /hello (GET) - i18n English default (55 ms)
    √ /hello?lang=vi (GET) - i18n Vietnamese (40 ms)

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   0 total
```

---

## 🗺️ Kế Hoạch Cho Pull 2 Tiếp Theo
Sau khi Pull 1 được hoàn thành và review, Pull 2 sẽ thực hiện:
1. Thiết lập **Migration Database thủ công** với TypeORM:
   - Tạo migration, apply migration, revert, reset.
2. Xây dựng Module Authentication:
   - `POST /api/v1/auth/register`: Đăng ký tài khoản, băm mật khẩu với bcrypt.
   - `POST /api/v1/auth/login`: Xác thực và cấp phát JWT Token.
   - `POST /api/v1/auth/logout`: Sử dụng **Redis để lưu blacklist token** ngăn chặn tái sử dụng token.
   - `GET /api/v1/auth/me`: Lấy thông tin Current User qua Bearer Token.

---
*Thực hiện bởi [Nhat2005p](https://github.com/Nhat2005p) - Thực tập sinh NodeJS tại Sun Asterisk.*
