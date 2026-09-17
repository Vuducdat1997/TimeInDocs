# TimeIn — chạy local

Bốn kho mã nguồn:

- [TimeInApp](https://github.com/Vuducdat1997/TimeInApp) — `App/`: Flutter iOS/Android.
- [TimeInBE](https://github.com/Vuducdat1997/TimeInBE) — `BE/`: NestJS + TypeScript, Prisma, PostgreSQL 17 qua Docker Compose.
- [WebAdmin](https://github.com/Vuducdat1997/WebAdmin) — `WebAdmin/`: React + TypeScript + Vite.
- [TimeInDocs](https://github.com/Vuducdat1997/TimeInDocs) — tài liệu: `README.md` này, `PLAN.md`, `PRODUCT.md`, `PLAN-v1-history.md`.

Đã có schema, seed, API đăng nhập và kiểm tra phân quyền; nghiệp vụ chấm công ở các bước sau.

## Thiết kế và tiến độ

Ý tưởng đã cập nhật: [PRODUCT.md](PRODUCT.md). Mô hình đã chốt: **Công ty/cửa hàng → Chi nhánh → Nhân viên**. Chủ/quản lý dùng giao diện quản lý; nhân viên dùng giao diện chấm công, lịch làm, yêu cầu và checklist.

Phần 1–4 đã có theo v1. Phần **4A** đã triển khai giao diện riêng theo vai trò trên Flutter: nhân viên vào khu vực cá nhân, chủ/quản lý vào khu vực quản lý. Ca làm, chấm công, phê duyệt và checklist chưa có API; các mục này hiển thị “Chưa sẵn sàng”. Bước tiếp theo là **5 — ca làm và phân lịch**, xem [PLAN.md](PLAN.md).

## Chuẩn bị

Máy đã có Flutter 3.38.4, Dart 3.10.3, Node 25.9.0, npm 11.12.1, Xcode 26 và CocoaPods 1.16.2.

Docker Desktop đã cài và engine 29.8.0 hoạt động. PostgreSQL đã healthy; `/health` trả HTTP 200 với database `ready`. Khi khởi động lại máy, mở Docker Desktop và chờ engine sẵn sàng.

Android SDK chưa có. Nếu muốn chạy Android: cài Android Studio, hoàn tất Setup Wizard với SDK và Emulator, sau đó chạy `flutter doctor --android-licenses` và `flutter doctor -v`. Yêu cầu cài Android Studio trong phiên này đã bị từ chối nên chưa cài.

## Chạy backend

Từ thư mục dự án:

```sh
cd BE
npm ci
# Chỉ thực hiện dòng copy nếu chưa có .env:
cp .env.example .env
npm run db:up
npm run db:generate
npm run db:deploy
npm run db:seed
npm run dev
```

Mật khẩu database mẫu chỉ dành cho local. Database chỉ mở tại loopback; API nghe `0.0.0.0` để điện thoại trong LAN truy cập.

- Health: http://127.0.0.1:3000/health
- Swagger: http://127.0.0.1:3000/docs
- HTTP 200: `{"status":"ok","database":"ready"}`.
- HTTP 503: database chưa sẵn sàng, không trả chi tiết kết nối.
- Build: `npm run build`; chạy bản build: `npm start`.
- Khi thay đổi model: `npm run db:migrate -- --name ten_thay_doi`.
- Dừng API: Ctrl+C. Dừng database: `npm run db:down`, volume vẫn được giữ.

## Chạy Flutter

```sh
cd App
flutter pub get
flutter devices
flutter run -d <device-id>
```

Đăng nhập bằng tài khoản demo trong mục Đăng nhập và phân quyền bên dưới.

| Thiết bị | API |
| --- | --- |
| iOS Simulator | Mặc định `http://127.0.0.1:3000` |
| Android Emulator | Mặc định `http://10.0.2.2:3000` |
| Điện thoại thật | Truyền IP LAN máy tính như bên dưới |

```sh
flutter run -d <device-id> --dart-define=API_BASE_URL=http://192.168.1.10:3000
```

Thay IP mẫu bằng IP máy tính; hai thiết bị cần kết nối mạng nội bộ. iPhone thật cần signing trong Xcode và quyền mạng local. HTTP local chỉ được cấu hình cho debug iOS/Android.

## Kiểm tra

Trong `BE`: `npm run db:generate`, `npm run build`, `curl -i http://127.0.0.1:3000/health`.

Trong `App`: `flutter analyze`, `flutter test`, `flutter build ios --simulator --debug`.

## Xử lý lỗi

- Docker daemon chưa chạy: mở Docker Desktop và chờ engine.
- Health 503: kiểm tra `docker compose ps`, `docker compose logs db` trong `BE` và `.env`.
- App không gọi được API: kiểm tra backend, API URL và firewall; điện thoại thật không dùng localhost máy tính.
- Cổng 5432/3000 bị chiếm: đổi cổng và cấu hình kết nối tương ứng.
- iOS báo `resource fork, Finder information`: metadata trên build/framework gây lỗi codesign; không tắt signing của bản release để xử lý.

Tiến độ, kiểm tra đã thực hiện và phần còn chặn: xem `PLAN.md`.

## Dữ liệu và nghiệp vụ

Xem [quy tắc và ERD](https://github.com/Vuducdat1997/TimeInBE/blob/main/docs/DATA-RULES.md). Trong `BE/`:

```sh
npm run db:deploy
npm run db:generate
npm run db:seed
npm run db:verify
```

Seed có thể chạy lại, không ghi đè dữ liệu sẵn có. Chỉ dùng local. Ba tài khoản demo TimeIn đã có luồng khởi tạo mật khẩu; xem tài liệu AUTH bên dưới. Kiểm tra dữ liệu dùng transaction rollback và không tạo công thật.

## Đăng nhập và phân quyền

Xem [luồng đăng nhập, tài khoản demo và API](https://github.com/Vuducdat1997/TimeInBE/blob/main/docs/AUTH.md).

Backend đã chạy thì mở iOS từ thư mục `App/`:

```sh
./scripts/ios-local.sh run -d 113FC3A8-1564-4FFC-80C7-2C404A8BD14A
```

Script tạo bản sao làm việc trong thư mục tạm để tránh lỗi metadata Finder khi codesign framework trong Documents. Code gốc vẫn ở `App/`. Chạy lại script để đưa thay đổi mới từ code gốc vào bản sao; hot reload của phiên này theo dõi bản sao, không tự đồng bộ code gốc. Không tắt codesigning của release.

Tài khoản thử: `manager.timein_demo@example.test`, mật khẩu mẫu local `TimeInLocal!2026`. Có thêm `owner.timein_demo@example.test` và `employee.timein_demo@example.test` cùng mật khẩu mẫu đã khởi tạo. Đây là dữ liệu local, không dùng cho môi trường public.

Màn Quên mật khẩu hiện hướng dẫn liên hệ quản lý; gửi email đặt lại mật khẩu chưa triển khai.

## Quản lý công ty và nhân viên (bước 4)

Tài liệu quyền, API và kiểm thử: [MANAGEMENT.md](https://github.com/Vuducdat1997/TimeInBE/blob/main/docs/MANAGEMENT.md). Một membership sẽ vào thẳng giao diện phù hợp; nhiều membership thì chọn công ty hoặc khôi phục đơn vị gần nhất sau khi xác minh với BE. Chủ/quản lý mở tab **Nhân sự** hoặc **Tổng quan/Thêm** để quản lý chi nhánh, địa điểm và công ty.

- OWNER thêm/sửa toàn công ty; MANAGER chỉ nhân viên thường và địa điểm ở chi nhánh được gán.
- Hồ sơ cá nhân luôn lấy đúng membership đang hoạt động.
- Nhân viên mới cần email chưa tồn tại và mật khẩu ban đầu ít nhất 12 ký tự.
- Dữ liệu form được kiểm tra ở App và API; không có API xóa nhân viên, dùng vô hiệu hóa.

Trong `BE/`: `DEMO_PASSWORD='TimeInLocal!2026' npm run management:verify`.


## Giao diện theo tài khoản (bước 4A)

- Nhân viên: **Chấm công · Lịch làm · Công việc · Yêu cầu · Cá nhân**. Hồ sơ cá nhân gọi API thật; lịch sử công mở từ Chấm công. Các module chưa triển khai không hiển thị số liệu hoặc nút ghi công giả.
- Chủ/quản lý: **Tổng quan · Nhân sự · Lịch làm · Phê duyệt · Thêm**. Tab Nhân sự dùng màn quản lý đã có; Tổng quan/Thêm mở chi nhánh, địa điểm, công ty, hồ sơ và các module đang chờ triển khai.
- Công ty gần nhất được lưu trong Keychain theo ID tài khoản; lựa chọn này không cấp quyền. App xác minh membership và `/access` khi khôi phục phiên, chọn đơn vị và trở lại từ nền. Màn quản lý kiểm tra quyền khi tải/lưu dữ liệu; BE kiểm tra quyền mọi request.
- Khi làm mới hoặc xác minh lại quyền, App bỏ màn/form cũ và tải lại ngữ cảnh; mất mạng hiển thị lỗi và Thử lại, hết phiên về đăng nhập. Chỉ hiện Đổi công ty khi có nhiều membership.
- App chưa đăng ký deep link tới module quản lý. Luồng mở module nội bộ kiểm tra vai trò; nhân viên không mở được các route quản lý bằng callback.

Kiểm tra trong `App/`:

```sh
flutter analyze
flutter test
./scripts/ios-local.sh test integration_test/workspace_flow_test.dart -d 113FC3A8-1564-4FFC-80C7-2C404A8BD14A --dart-define=DEMO_PASSWORD=TimeInLocal!2026
```

Kiểm thử workspace trên iOS dùng BE/database local đang chạy, kiểm tra cả ba vai trò, đọc dữ liệu thật, khôi phục Keychain và đăng xuất. Nhiều/không có membership, đổi vai trò, mất quyền, lỗi mạng, chặn route quản lý và màn 320px được kiểm tra bằng widget test; chưa nghiệm thu trên Android hoặc iPhone vật lý.

## Quy tắc cấp tài khoản và công ty của nhân viên — cập nhật 2026-09-17

- Chỉ OWNER/MANAGER được tạo tài khoản nhân viên trong phạm vi quản lý; không có tự đăng ký.
- Công ty và chi nhánh do cấp quản lý gán khi tạo tài khoản. Nhân viên vào thẳng đơn vị được phân công, không có chọn/đổi công ty và không dùng lựa chọn công ty lưu trên thiết bị.
- OWNER/MANAGER chỉ chọn trong các membership quản lý được cấp. Quy tắc nhiều công ty trước đây không áp dụng cho EMPLOYEE.
- Nếu dữ liệu cũ gán EMPLOYEE vào nhiều công ty đang hoạt động, App yêu cầu liên hệ quản lý; BE chặn truy cập nghiệp vụ cho tới khi phân công được điều chỉnh. Không tự chọn công ty đầu tiên.
- BE đã chặn nhân viên tạo tài khoản hoặc tự sửa membership. MANAGER chỉ tạo/sửa nhân viên thường trong chi nhánh; OWNER quản lý trong công ty. Chuyển công ty giữa các đơn vị chưa có luồng quản trị riêng, cần thiết kế quyền nhận/chuyển; không cho nhân viên tự thực hiện.
