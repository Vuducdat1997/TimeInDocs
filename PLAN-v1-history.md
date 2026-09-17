# TimeIn — Kế hoạch triển khai

Cập nhật: 2026-09-15

## Mục tiêu và phạm vi

Xây dựng app chấm công iOS/Android và backend, chạy hoàn toàn local.
Chưa triển khai public, thuê VPS hoặc tích hợp tính lương.

| Thành phần | Công nghệ / vị trí |
| --- | --- |
| App | Flutter, thư mục `App/` |
| Backend | NestJS + TypeScript, thư mục `BE/` |
| Database | PostgreSQL chạy bằng Docker Compose |
| Truy cập dữ liệu | Prisma |
| Tài liệu API | Swagger / OpenAPI |

## Cách quản lý tiến độ

- `[ ]`: chưa hoàn thành; `[x]`: đã hoàn thành và có bằng chứng kiểm tra.
- Trạng thái phần: Chưa bắt đầu / Đang làm / Bị chặn / Hoàn thành.
- Khi bắt đầu một phần, cập nhật trạng thái và công việc hiện tại.
- Sau mỗi phần, ghi kết quả kiểm tra, giới hạn chưa xác minh và bước tiếp theo.
- Không đánh dấu hoàn thành chỉ vì đã viết code; phải đạt điều kiện nghiệm thu.
- Khi thay đổi phạm vi hoặc quy tắc, cập nhật file này cùng code liên quan.

## Tổng quan

| Phần | Nội dung | Phụ thuộc | Trạng thái |
| --- | --- | --- | --- |
| 1 | Môi trường và bộ khung | — | Hoàn thành (iOS; Android hoãn) |
| 2 | Dữ liệu và nghiệp vụ | 1 (backend đã sẵn sàng) | Hoàn thành |
| 3 | Đăng nhập và phân quyền | 2 | Hoàn thành |
| 4 | Công ty, chi nhánh, nhân viên | 3 | Hoàn thành |
| 5 | Ca làm và lịch làm | 4 | Chưa bắt đầu |
| 6 | Chấm công GPS | 5 | Chưa bắt đầu |
| 7 | Bảng công | 6 | Chưa bắt đầu |
| 8 | Xin nghỉ và điều chỉnh công | 7 | Chưa bắt đầu |
| 9 | Báo cáo và hoàn thiện local | 8 | Chưa bắt đầu |

## Công việc hiện tại

- Phần 1 nghiệm thu trên iOS Simulator; Android vẫn hoãn theo yêu cầu.
- Phần 2 và 3 hoàn thành: schema/seed, đăng nhập, lưu phiên Keychain, chọn công ty và kiểm tra quyền backend.
- iOS build thành công khi dùng bản sao thư mục tạm: `App/scripts/ios-local.sh`.
- Test iOS thực tế đạt: đăng nhập quản lý → chọn công ty → hiển thị quyền → đọc lại Keychain bằng client mới → đăng xuất.
- Phần 4 hoàn thành: quản lý công ty/chi nhánh/địa điểm và nhân viên; đã kiểm thử end-to-end trên iOS.
- Bước tiếp theo: phần 5 — ca làm và lịch làm.
- Màn Trang chủ chấm công và nghiệp vụ ca/công vẫn chưa triển khai.
- Còn cảnh báo dependency Prisma CLI/deepmerge-ts đã ghi nhận; chưa xử lý trong thay đổi đăng nhập này.

## 1. Môi trường và bộ khung

- [x] Kiểm tra code hiện có và hướng dẫn dự án trước khi chỉnh sửa.
- [x] Kiểm tra Flutter, Dart, Node.js, Docker, Xcode, Android SDK và thiết bị giả lập.
- [x] Ghi rõ công cụ còn thiếu và cách cài đặt.
- [x] Khởi tạo Flutter trong `App/` và NestJS trong `BE/`, giữ dữ liệu hiện có.
- [x] Cấu hình PostgreSQL bằng Docker Compose với volume lưu dữ liệu (đã chạy và healthy).
- [x] Cấu hình Prisma và kết nối database.
- [x] Tạo `.env.example`, loại trừ secret và file sinh tự động khỏi Git.
- [x] Kiểm tra/khởi tạo Git nếu chưa có.
- [x] Tạo API `/health` kiểm tra database, không lộ thông tin nhạy cảm.
- [x] Tạo màn hình Flutter hiển thị trạng thái API/database.
- [x] Cấu hình API URL cho iOS Simulator, Android Emulator và điện thoại thật.
- [x] Viết README hướng dẫn chạy, dừng và xử lý lỗi thường gặp.

**Nghiệm thu:** Flutter gọi được API và backend kết nối được PostgreSQL. Ghi rõ nền tảng đã chạy thực tế; phần chưa kiểm tra phải được nêu riêng.

## 2. Dữ liệu và quy tắc nghiệp vụ

- [x] Thiết kế ERD: users, sessions, organizations, memberships, branches, work_locations.
- [x] Thiết kế shift_templates, shift_assignments, attendance_events, attendance_sessions.
- [x] Thiết kế leave_requests, correction_requests, timesheet_periods và audit_logs.
- [x] Xác định khóa ngoại, chỉ mục, ràng buộc chống trùng và phạm vi công ty.
- [x] Lưu thời gian UTC; tính ngày công theo múi giờ địa điểm.
- [x] Lưu thời điểm bắt đầu/kết thúc cụ thể cho ca được phân, hỗ trợ ca qua đêm.
- [x] Tách dữ liệu chấm công gốc khỏi kết quả tính công và điều chỉnh.
- [x] Ghi quy tắc giờ nghỉ, đi muộn, về sớm, thiếu công và ca chồng lấn.
- [x] Tạo migration và seed dữ liệu local có thể chạy lại an toàn.

**Nghiệm thu:** database tạo được từ migration; seed có công ty, chi nhánh, quản lý, nhân viên và ca mẫu. Các giả định nghiệp vụ được ghi rõ.

## 3. Đăng nhập và phân quyền

- [x] BE: đăng nhập email/mật khẩu với mật khẩu được băm an toàn.
- [x] BE: access token, refresh token, thu hồi phiên và đăng xuất.
- [x] BE: API thông tin tài khoản hiện tại.
- [x] BE: quyền chủ công ty, quản lý, nhân viên và phạm vi chi nhánh.
- [x] App: màn hình đăng nhập, lưu phiên an toàn và khôi phục phiên.
- [x] App: điều hướng theo vai trò, xử lý hết phiên và đăng xuất.
- [x] Kiểm thử sai mật khẩu, token hết hạn/thu hồi và truy cập trái quyền.
- [x] Kiểm thử cách ly dữ liệu giữa hai công ty.

**Nghiệm thu:** đăng nhập/đăng xuất hoạt động end-to-end; quyền được kiểm tra tại backend.

## 4. Công ty, chi nhánh và nhân viên

- [x] BE: quản lý công ty, chi nhánh và địa điểm chấm công.
- [x] BE: cấu hình tọa độ, bán kính và múi giờ địa điểm.
- [x] BE: thêm nhân viên, phân quyền, gán chi nhánh và vô hiệu hóa tài khoản.
- [x] App: danh sách, chi tiết và biểu mẫu nhân viên cho quản lý.
- [x] App: quản lý chi nhánh và địa điểm.
- [x] App: hồ sơ cá nhân cho nhân viên.
- [x] Kiểm thử phạm vi quản lý và chặn tài khoản bị vô hiệu hóa, kể cả phiên có sẵn.

**Nghiệm thu:** quản lý tạo được nhân viên, gán địa điểm; nhân viên chỉ truy cập dữ liệu được phép.

## 5. Ca làm và lịch làm

- [ ] BE: mẫu ca gồm giờ bắt đầu, kết thúc và quy tắc giờ nghỉ.
- [ ] BE: phân ca theo nhân viên/ngày và kiểm tra chồng lấn.
- [ ] BE: thay đổi mẫu ca không làm sai ca lịch sử.
- [ ] App: tạo ca, phân ca và xem chi tiết.
- [ ] App: lịch tuần và ca hôm nay của nhân viên.
- [ ] Kiểm thử ca thường, ca qua đêm và ca bị trùng.

**Nghiệm thu:** ca được phân hiển thị đúng thời gian, ngày và nhân viên; ca không hợp lệ bị từ chối rõ ràng.

## 6. Chấm công GPS

- [ ] BE: API check-in/check-out, kiểm tra ca và trạng thái vào/ra hợp lệ.
- [ ] BE: kiểm tra khoảng cách, độ mới và độ chính xác của vị trí.
- [ ] BE: dùng giờ máy chủ cho chấm công trực tuyến.
- [ ] BE: chống ghi trùng khi retry hoặc có request đồng thời bằng transaction và ràng buộc dữ liệu.
- [ ] App: xin quyền và chỉ lấy vị trí khi chấm công.
- [ ] App: hiển thị ca, nút vào/ra và kết quả server xác nhận.
- [ ] App: xử lý từ chối quyền, ngoài phạm vi, GPS kém và mất mạng.
- [ ] Kiểm thử vị trí bằng dữ liệu kiểm soát; xác minh GPS trên thiết bị thật khi có.
- [ ] Kiểm thử bấm nhiều lần, gửi lại request và thiếu giờ ra.

**Nghiệm thu:** vào/ra end-to-end; không có bản ghi trùng; không báo thành công khi server chưa xác nhận. GPS là tín hiệu kiểm tra, không bảo đảm chống giả mạo tuyệt đối.

## 7. Bảng công

- [ ] BE: truy vấn lịch sử theo nhân viên và khoảng ngày, có phân trang.
- [ ] BE: tính giờ làm, giờ nghỉ, phút đi muộn và về sớm.
- [ ] BE: đánh dấu thiếu giờ vào/ra; không tự coi là đủ công.
- [ ] BE: tổng hợp theo tháng và múi giờ địa điểm.
- [ ] App: bảng công cá nhân và chi tiết từng ca.
- [ ] App: quản lý lọc theo nhân viên, chi nhánh và tháng.
- [ ] Kiểm thử đối chiếu kết quả với dữ liệu mẫu, gồm ca qua đêm và qua tháng.

**Nghiệm thu:** tổng giờ khớp chi tiết và bộ dữ liệu kiểm thử. Chỉ quy đổi ngày công sau khi thống nhất công thức.

## 8. Xin nghỉ và điều chỉnh công

- [ ] BE: gửi yêu cầu nghỉ và yêu cầu điều chỉnh công.
- [ ] BE: duyệt/từ chối, kiểm tra người duyệt và trạng thái yêu cầu.
- [ ] BE: lưu lý do, người thao tác, giá trị trước/sau; giữ sự kiện công gốc.
- [ ] BE: tính lại công sau phê duyệt và chống xử lý một yêu cầu hai lần.
- [ ] App: biểu mẫu yêu cầu và lịch sử trạng thái.
- [ ] App: danh sách chờ duyệt và thao tác phê duyệt cho quản lý.
- [ ] Kiểm thử yêu cầu được duyệt, bị từ chối và phê duyệt trái quyền.

**Nghiệm thu:** yêu cầu được duyệt cập nhật công đúng; yêu cầu bị từ chối không thay đổi công; có lịch sử đầy đủ.

## 9. Báo cáo và hoàn thiện local

- [ ] Xuất bảng công CSV, kiểm tra dữ liệu và quyền tải báo cáo.
- [ ] Chốt kỳ công, chặn điều chỉnh kỳ đã chốt.
- [ ] Mở lại kỳ theo quyền, lưu audit và kiểm thử quy trình.
- [ ] Hoàn thiện trạng thái tải, rỗng, lỗi và mất phiên trên app.
- [ ] Kiểm tra toàn bộ luồng trên iOS và Android; ghi rõ simulator/emulator/thiết bị thật.
- [ ] Thiết lập và thử backup/restore database local.
- [ ] Hoàn thiện README, tài khoản mẫu và các lệnh kiểm tra.
- [ ] Review lỗi nghiệp vụ, phân quyền và dữ liệu nhạy cảm.

**Nghiệm thu:** chạy được từ README và thực hiện luồng tạo nhân viên → phân ca → chấm công → điều chỉnh → xuất báo cáo.

## Các quyết định cần chốt khi đến phần liên quan

- Phần 2/5: giờ nghỉ có trừ cố định không; ngưỡng đi muộn; cửa sổ được phép vào/ra ca.
- Phần 4/6: bán kính địa điểm, độ chính xác và tuổi tối đa của dữ liệu GPS.
- Phần 7: cách làm tròn giờ và quy đổi ngày công nếu cần.
- Phần 8: nghỉ theo giờ/nửa ngày/cả ngày; ai duyệt yêu cầu của quản lý.
- Phần 9: ai được chốt/mở kỳ công và thời hạn lưu dữ liệu.

Các mục này chưa cản trở việc dựng bộ khung. Ghi quyết định cụ thể trước khi triển khai logic tương ứng.

## Ngoài phạm vi hiện tại

- Deploy public, domain, VPS và phát hành store.
- Tính lương, SMS OTP, nhận diện khuôn mặt và QR động.
- Chấm công offline tự động được chấp nhận.
- Web quản trị, microservices và Redis.
- Push notification qua dịch vụ bên ngoài; sẽ đánh giá ở giai đoạn sau.

## Nhật ký triển khai

| Ngày | Phần | Thay đổi / kiểm tra | Bước tiếp theo |
| --- | --- | --- | --- |
| 2026-09-14 | Kế hoạch | Tạo checklist; chưa khởi tạo hoặc kiểm thử code | Kiểm tra môi trường ở phần 1 |

### Kết quả phần 1 (đang cập nhật)

- Flutter doctor: Flutter/Dart, Xcode, CocoaPods đạt; thiếu Android SDK.
- Flutter analyze và widget test đạt.
- Prisma Client generate và TypeScript backend build đạt.
- NestJS đã nâng lên 12 để xử lý cảnh báo dependency runtime. Còn 3 cảnh báo high trong chuỗi Prisma CLI/deepmerge-ts; cần xử lý trước khi mở rộng schema.
- Docker Desktop tải được nhưng cài thất bại ở bước sudo tạo symlink; cần người dùng nhập mật khẩu trong Terminal của họ.
- Build iOS vẫn bị lỗi metadata codesign sau lần thử lại; chưa chạy được UI trên Simulator.
- Chưa có migration nghiệp vụ (phần 2), đã xác nhận kết nối database thật qua health HTTP 200.

- Backend dev đã khởi động: Swagger trả HTTP 200; health trả HTTP 503 đúng khi chưa có PostgreSQL.
- Người dùng chốt test iOS trước, không cần cài Android Studio ở giai đoạn này.

- Docker đã được người dùng cài; engine 29.8.0 hoạt động.
- PostgreSQL 17 (`be-db-1`) healthy; API `/health` trả HTTP 200 với database `ready`.
- Bước tiếp theo: xử lý build iOS và xác minh màn hình App → API → database trên Simulator.

### Kết quả phần 2

- Migration `202609140001_initial` đã áp dụng vào PostgreSQL local; `prisma migrate status` xác nhận đồng bộ.
- Schema gồm tài khoản/phiên, tổ chức/nhân sự, địa điểm, policy version, ca, sự kiện/kết quả công, yêu cầu và audit.
- Có khóa ngoại ghép theo công ty, CHECK, EXCLUDE chống ca trùng và trigger bảo vệ dữ liệu gốc.
- Seed chạy hai lần thành công: hai công ty, sáu người dùng, bốn ca phân gồm ca thường/qua đêm; chưa có mật khẩu hoặc công giả.
- `npm run db:verify`: 12 kiểm tra transaction rollback và kiểm tra snapshot ca đêm đạt.
- `prisma validate`, generate và TypeScript build đạt.
- Logic quyền API, tính giờ công, phê duyệt và khóa kỳ chưa triển khai; tiếp tục theo phần 3/6/7/8/9. Không đánh dấu các phần đó hoàn thành từ schema.

### Kết quả phần 3 — 2026-09-15

- Màn Flutter: đăng nhập, hiện/ẩn mật khẩu, lỗi/validation/loading, chọn công ty, màn quyền theo vai trò và danh sách nhân viên cho quản lý.
- API: login, refresh, me, logout, organization access và employees theo phạm vi. Password scrypt; opaque access 15 phút, refresh tối đa 7 ngày, DB chỉ lưu hash token.
- Migration `202609140002_auth_access` đã áp dụng; ba tài khoản demo local đã khởi tạo mật khẩu (xem [AUTH.md](https://github.com/Vuducdat1997/TimeInBE/blob/main/docs/AUTH.md)).
- `npm run auth:verify` đạt: sai mật khẩu, thiếu token, chống role injection, khác công ty, giới hạn chi nhánh, inactive membership, hết hạn, refresh đồng thời và thu hồi logout.
- Backend build đạt. Flutter analyze và 2 widget tests đạt (validation/error và màn hình nhỏ).
- Integration test trên iPhone 17 Pro Simulator / iOS 26 đạt, gồm lưu/đọc phiên thật qua Keychain và logout.
- Chưa có đặt lại mật khẩu/email hoặc API thay đổi vai trò; chưa kiểm tra Android và iPhone vật lý.
- Lỗi codesign trong Documents được xử lý bằng script tạo bản sao tạm, không tắt ký framework.

### Kết quả phần 4 — 2026-09-15

- BE có API xem/đổi tên công ty, hồ sơ cá nhân, tạo/sửa chi nhánh và địa điểm, nhân viên phân trang/tìm kiếm/lọc/chi tiết/tạo/sửa/vô hiệu hóa.
- OWNER quản lý toàn công ty; MANAGER chỉ nhân viên thường và địa điểm trong chi nhánh; EMPLOYEE chỉ hồ sơ và thông tin được phép.
- Mutation khóa hàng công ty, kiểm tra quyền lại và ghi audit cùng transaction; chặn tự đổi quyền/vô hiệu hóa và mất chủ công ty cuối cùng.
- Email cũ không được ghi đè/tự gán sang công ty; password mới băm scrypt; không ghi password vào audit.
- Flutter có màn danh sách, chi tiết, biểu mẫu, phân trang, lỗi/loading, xác nhận thay đổi quyền/trạng thái và làm mới thông tin sau chỉnh sửa.
- Backend build, kiểm thử auth hiện có và `management:verify` đều đạt.
- Flutter analyze và 4 widget tests đạt; integration test iPhone 17 Pro / iOS 26 đạt trên bản cuối: tạo chi nhánh → địa điểm → nhân viên → sửa/vô hiệu hóa.
- Fixture test đã dọn, audit bất biến được giữ. Không thay đổi dữ liệu seed hiện có.
- Quyết định/giới hạn: tên/email danh tính chỉ nhập khi tạo; địa điểm không chuyển chi nhánh; chưa có mời tài khoản email đã tồn tại, onboarding công ty mới, reset mật khẩu hoặc bản đồ. Xem [MANAGEMENT.md](https://github.com/Vuducdat1997/TimeInBE/blob/main/docs/MANAGEMENT.md).
