# TimeIn — Kế hoạch triển khai v2

Cập nhật: 2026-09-17. Đây là **trang điều hướng**. Kế hoạch chi tiết nằm ở plan riêng của từng phần:

| Plan | Phạm vi | Repo |
| --- | --- | --- |
| [PLAN-BE.md](PLAN-BE.md) | Backend: API, schema, nghiệp vụ, phân quyền | [TimeInBE](https://github.com/Vuducdat1997/TimeInBE) |
| [PLAN-APP.md](PLAN-APP.md) | App Flutter cho nhân viên và quản lý | [TimeInApp](https://github.com/Vuducdat1997/TimeInApp) |
| [PLAN-WEB.md](PLAN-WEB.md) | Web quản trị cho chủ đơn vị và quản lý | [WebAdmin](https://github.com/Vuducdat1997/WebAdmin) |

Thiết kế đích: [PRODUCT.md](PRODUCT.md). Lịch sử và bằng chứng kiểm tra trước thay đổi: [PLAN-v1-history.md](PLAN-v1-history.md).

## Mục tiêu đã chốt

- **App (Flutter):** dành cho nhân viên (chấm công, lịch làm, yêu cầu, checklist) và quản lý (theo dõi nhanh, phê duyệt).
- **Web quản trị (React + TypeScript):** dành cho chủ đơn vị và quản lý để vận hành, quản lý nhân sự, xếp lịch và xuất báo cáo.
- **Backend (NestJS):** API tập trung, phân quyền chặt theo mô hình **Công ty/cửa hàng → Chi nhánh → Nhân viên**.
- Một app Flutter iOS/Android; backend NestJS + TypeScript, Prisma và PostgreSQL qua Docker Compose.
- Chạy local, ưu tiên iOS; Android tiếp tục hoãn. Chưa deploy public hoặc tích hợp tính lương.
- **Ưu tiên hiện tại:** phần 4A đã hoàn thành local. Tiếp theo phần 5 — ca làm và phân lịch, để mở đường cho chấm công ở phần 6.

## Cách quản lý tiến độ

- `[x]` là đã hoàn thành **và có bằng chứng chạy thật**; `[ ]` là còn việc.
- Mỗi plan con **chỉ quản lý phần mình**: `PLAN-BE.md` không mô tả màn hình, `PLAN-APP.md` không quyết định nghiệp vụ hay quyền.
- Mỗi feature có mã riêng `BE-x.y`, `APP-x.y`, `WEB-x.y`. Số giữ theo phần 1–9 của bản kế hoạch cũ để không phá vỡ các tham chiếu đang có trong tài liệu và test.
- Mỗi feature bắt buộc có ba mục:
  - **Đầu vào** — cần gì từ plan khác, hoặc quyết định nào phải chốt trước khi code.
  - **Đầu ra** — sản phẩm cụ thể: endpoint, file, màn hình, migration, script.
  - **Nghiệm thu** — tiêu chí kiểm chứng được, kèm lệnh chạy khi có.
- Feature chỉ được đánh `[x]` khi phần **Nghiệm thu** đã chạy thật, không suy đoán.
- Số liệu trong tài liệu phải lấy từ lần chạy thật.

## Ma trận tổng quan

| Phần | Nội dung | BE | App | Web |
| --- | --- | --- | --- | --- |
| 1 | Môi trường và bộ khung | Hoàn thành | Hoàn thành | Hoàn thành |
| 2 | Dữ liệu và nghiệp vụ nền | Hoàn thành | — | — |
| 3 | Đăng nhập và phân quyền | Hoàn thành | Hoàn thành | Hoàn thành |
| 4 | Công ty, chi nhánh, nhân sự | Hoàn thành | Hoàn thành | Chức năng chính đã có; còn nghiệm thu |
| 4A | Giao diện App theo tài khoản | Hoàn thành (contract) | Hoàn thành local | — |
| 5 | Ca làm, giờ làm và lịch | Chưa bắt đầu | Chưa bắt đầu | Chưa bắt đầu |
| 6 | Chấm công GPS | Chưa bắt đầu | Chưa bắt đầu | Không thuộc phạm vi |
| 7 | Lịch sử công và bảng công | Chưa bắt đầu | Chưa bắt đầu | Chưa bắt đầu |
| 8 | Nghỉ, sửa công và phê duyệt | Chưa bắt đầu | Chưa bắt đầu | Chưa bắt đầu |
| 8A | Checklist công việc | Chưa bắt đầu | Chưa bắt đầu | Chưa bắt đầu |
| 9 | Báo cáo, chốt kỳ và nghiệm thu | Chưa bắt đầu | Chưa bắt đầu | Chưa bắt đầu |

## Hợp đồng liên phần

App và Web không tự quyết định nghiệp vụ hay quyền. Mọi giao tiếp đi qua BE, và BE là bên duy nhất xác minh quyền từ membership trong database.

| Hợp đồng | Bên cung cấp | Bên dùng | Trạng thái |
| --- | --- | --- | --- |
| `/auth/login`, `/auth/refresh`, `/auth/me`, `/auth/logout` | BE-3.1 → BE-3.3 | App, Web | Đã có |
| `/organizations/:id/access` | BE-3.5 | App, Web | Đã có |
| `/organizations/:id` (profile, branches, locations, employees) | BE-4.1 → BE-4.5 | App, Web | Đã có |
| Contract membership cho chọn giao diện | BE-4A.1 | App | Đã có |
| Chặn EMPLOYEE nhiều công ty | BE-4A.2 | App, Web | Đã có |
| Mẫu ca, phân ca, lịch tuần | BE-5.1 → BE-5.5 | App, Web | Chưa có |
| Chấm công vào/ra | BE-6.1 → BE-6.4 | App | Chưa có |
| Mẫu khuôn mặt và xác thực khi chấm công | BE-6.5, BE-6.6 | App | Chưa có — **cần chốt phạm vi** |
| Lịch sử và bảng công | BE-7.1 → BE-7.3 | App, Web | Chưa có |
| Thống kê cá nhân theo tháng (số ca, số giờ) | BE-7.4 | App | Chưa có |
| Nghỉ, sửa công và phê duyệt | BE-8.1 → BE-8.4 | App, Web | Chưa có |
| Đổi ca giữa hai nhân viên | BE-8.5 | App, Web | Chưa có — **cần chốt phạm vi** |
| Checklist công việc | BE-8A.2 → BE-8A.5 | App, Web | Chưa có |
| Tổng quan, xuất CSV, chốt kỳ | BE-9.1 → BE-9.3 | App, Web | Chưa có |
| Thông báo trong app | BE-9.6 | App | Chưa có — **cần chốt phạm vi** |
| Bảng lương và thu nhập | BE-9.7 | App | Chưa có — **cần chốt phạm vi** |

Khi một hợp đồng thay đổi, cập nhật đồng thời feature tương ứng ở cả ba plan.

## Công việc hiện tại và bước tiếp theo

**Hiện tại:** chức năng chính phần 4 đã có trên cả ba phần; phần 4A đã hoàn thành local và kiểm thử trên iOS Simulator. Các mục nghiệp vụ chưa có API hiển thị "Chưa sẵn sàng", chưa ghi nhận chấm công thật.

Thứ tự công việc tiếp theo:

1. Phần 5: BE mẫu ca và phân ca → Web xếp lịch → App nhân viên xem ca hôm nay và lịch làm.
2. Phần 6: BE và App check-in/check-out GPS; sau đó phần 7 lịch sử và bảng công.
3. Phần 8, 8A, 9: nghỉ/sửa công/phê duyệt, checklist, báo cáo và chốt kỳ.
4. Nghiệm thu Web còn mở (WEB-4.7 → WEB-4.9): tài khoản nhiều công ty, màn hình nhỏ và thiết bị LAN khác. Push/deploy ghi riêng, không đồng nghĩa hoàn thành tính năng.

**Bằng chứng v1:** kiểm tra dữ liệu 12 tình huống, auth và management API đạt; backend build, Flutter analyze, widget test và integration trên iPhone 17 Pro Simulator/iOS 26 đạt. Chưa kiểm tra Android/iPhone vật lý. Cảnh báo dependency Prisma CLI/deepmerge-ts còn trong backlog, cần đánh giá trước khi mở rộng schema.

## Nhật ký v2

| Ngày | Thay đổi | Kiểm tra / bước tiếp theo |
| --- | --- | --- |
| 2026-09-17 | Thêm logo thương hiệu: sinh 19 icon iOS + 5 Android bằng script Swift + `sips` (không thêm dependency), hiển thị logo ở màn đăng nhập. Sửa ba lỗi lệch thiết kế ở Trang Chủ | `flutter analyze` sạch, 22 test đạt. Icon lên màn hình chính; nhãn ca, tên quản lý và thanh xanh đã đúng thiết kế. Xác nhận lưu phiên đăng nhập hoạt động |
| 2026-09-17 | Dựng màn **Trang Chủ** nhân viên theo thiết kế: danh tính, băng ca, công việc hôm nay, dải GPS; đổi thanh tab nhân viên sang Trang Chủ · Lịch · Chụp ảnh · Checklist · Cá Nhân. Lưu thiết kế vào `docs/design/trang-chu/` | `flutter analyze` sạch, 22 widget test đạt. Danh tính lấy API thật; ca/công việc/GPS còn là dữ liệu mẫu có nhãn, chờ BE-5/BE-8A/BE-6. Chưa chụp được màn trên máy ảo vì cần đăng nhập |
| 2026-09-17 | Bổ sung feature từ thiết kế 8 màn hình nhân viên: lịch tháng, chấm công bằng khuôn mặt, màn kết quả chấm công, checklist theo ca, form đơn xin phép, đổi ca, trang chủ có thông báo, tab Cá Nhân có thống kê tháng, bảng lương | Ba nội dung vượt phạm vi (khuôn mặt, bảng lương, đổi ca) được ghi kèm nhãn **cần chốt phạm vi**; đã cập nhật bảng Hợp đồng liên phần và mục Quyết định còn mở |
| 2026-09-17 | Tách kế hoạch thành trang điều hướng và ba plan riêng `PLAN-BE.md`, `PLAN-APP.md`, `PLAN-WEB.md`; mỗi feature có Đầu vào / Đầu ra / Nghiệm thu | Bổ sung `AGENTS.md` cho App và BE. Tiếp theo phần 5 |
| 2026-09-17 | Tách mã nguồn thành 4 repo: `TimeInApp`, `TimeInBE`, `TimeInDocs`, `WebAdmin`; chuyển `PLAN.md` và `README.md` vào `docs/`, đổi link chéo sang URL đầy đủ | Đã kiểm tra không đẩy `.env`, `node_modules` hay thư mục build; repo gốc rỗng đã xoá |
| 2026-09-17 | Hoàn thành 4A: hai shell Flutter, chọn/nhớ công ty theo tài khoản, xác minh lại quyền và tái sử dụng màn quản lý | Analyze, 21 widget test và integration ba vai trò + CRUD quản lý trên iOS Simulator đạt. Tiếp theo phần 5 |
| 2026-09-16 | Hoàn thiện chức năng chính Web nhân sự/chi nhánh/địa điểm/cài đặt; sửa build, kết nối BE và phiên đăng nhập; đồng bộ lại phạm vi và thứ tự plan | Web/BE build đạt, 8 test phiên Web và kiểm thử auth/management BE đạt; kiểm tra UI ba vai trò. Còn nghiệm thu nhiều công ty, responsive và thiết bị LAN khác |
| 2026-09-15 | Chốt cấu trúc Công ty/cửa hàng → Chi nhánh → Nhân viên; thiết kế hai giao diện, thêm 4A và 8A | Chỉ cập nhật tài liệu; bắt đầu code phần 4A khi tiếp tục triển khai |

Chi tiết kết quả phần 1–4 và các vấn đề đã xử lý xem [lịch sử v1](PLAN-v1-history.md). Kết quả kiểm thử v1 không thay cho nghiệm thu các tính năng v2.

## Quyết định còn mở

Phải chốt trước khi bắt đầu feature tương ứng; ghi lại quyết định vào plan con rồi mới code.

- **Phần 5–6:** rà soát và chốt các mặc định trong `BE/docs/DATA-RULES.md` về giờ nghỉ, đi muộn, cửa sổ vào/ra và GPS.
- **Phần 5:** quy tắc sửa/hủy ca đã có công.
- **Phần 6 — chấm công bằng khuôn mặt (mới, từ thiết kế 2026-09-17):** có đưa vào phạm vi không; nếu có thì dùng thư viện/thuật toán nào, chạy trên thiết bị hay gửi lên server, lưu mẫu khuôn mặt ở đâu, thời hạn lưu và quyền xóa. Dữ liệu sinh trắc học cần quyết định riêng trước khi viết code.
- **Phần 6 — thanh tab nhân viên:** đã chốt và triển khai 2026-09-17 theo thiết kế (Trang Chủ · Lịch · Chụp ảnh · Checklist · Cá Nhân). Đơn từ mở từ tab Cá Nhân theo `APP-8.7`; nếu muốn thêm lối vào ngay Trang Chủ thì bổ sung sau, không phá cấu trúc.
- **Phần 8 — đổi ca (mới):** cho phép nhân viên tự thỏa thuận đổi ca với nhau hay chỉ gửi đề xuất để quản lý duyệt; ca đã chấm công có được đổi không.
- **Phần 9 — bảng lương và thu nhập (mới):** hiện nằm ngoài phạm vi. Nếu đưa vào thì cần chốt công thức lương, quyền xem và có gắn với chốt kỳ công không.
- **Phần 9 — thông báo trong app (mới):** nguồn thông báo (đổi ca, duyệt đơn, nhắc vào ca), có cần push qua dịch vụ ngoài hay chỉ hiển thị trong app.
- **Phần 7:** làm tròn giờ và có quy đổi ngày công hay không.
- **Phần 8:** loại nghỉ và tác động lên công; chủ/quản lý có cần thêm quyền làm việc cá nhân.
- **Phần 8A:** khóa khi hoàn tất, quyền mở lại, cách đặt hạn giao việc.
- **Phần 9:** thời hạn lưu dữ liệu và quy trình chốt/mở lại kỳ.

## Ngoài phạm vi hiện tại

Trong giai đoạn local này chưa triển khai: deploy BE public và phát hành store, tính lương, SMS OTP, nhận diện khuôn mặt, QR động, chấp nhận công offline tự động, microservices, Redis và push qua dịch vụ ngoài.

**Thay đổi phạm vi đang chờ quyết định:** thiết kế màn hình nhân viên ngày 2026-09-17 có ba nội dung nằm trong danh sách trên — **chấm công bằng khuôn mặt**, **bảng lương và thu nhập**, và **đổi ca**. Các feature tương ứng đã được ghi vào plan (`BE-6.5`, `BE-6.6`, `BE-9.7`, `BE-8.5`, `APP-6.6` → `APP-6.8`, `APP-9.9`) nhưng đánh dấu *cần chốt phạm vi*. Chưa nội dung nào được coi là đã chốt; xem mục **Quyết định còn mở**.

Android chưa kiểm thử. Onboarding công ty mới, mời email đã có và cấp nhiều chi nhánh cho cùng một quản lý chưa triển khai. Admin toàn hệ thống là yêu cầu còn mở, tách khỏi quyền OWNER — theo dõi ở `PLAN-WEB.md` WEB-4.11.

## Quy tắc cấp tài khoản và công ty của nhân viên — cập nhật 2026-09-17

- Chỉ OWNER/MANAGER được tạo tài khoản nhân viên trong phạm vi quản lý; không có tự đăng ký.
- Công ty và chi nhánh do cấp quản lý gán khi tạo tài khoản. Nhân viên vào thẳng đơn vị được phân công, không có chọn/đổi công ty và không dùng lựa chọn công ty lưu trên thiết bị.
- OWNER/MANAGER chỉ chọn trong các membership quản lý được cấp. Quy tắc nhiều công ty trước đây không áp dụng cho EMPLOYEE.
- Nếu dữ liệu cũ gán EMPLOYEE vào nhiều công ty đang hoạt động, App yêu cầu liên hệ quản lý; BE chặn truy cập nghiệp vụ cho tới khi phân công được điều chỉnh. Không tự chọn công ty đầu tiên.
- BE đã chặn nhân viên tạo tài khoản hoặc tự sửa membership. MANAGER chỉ tạo/sửa nhân viên thường trong chi nhánh; OWNER quản lý trong công ty. Chuyển công ty giữa các đơn vị chưa có luồng quản trị riêng, cần thiết kế quyền nhận/chuyển; không cho nhân viên tự thực hiện.
