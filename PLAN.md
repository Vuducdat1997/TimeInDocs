# TimeIn — Kế hoạch triển khai v2

Cập nhật: 2026-09-17. Thiết kế chi tiết: [Ý tưởng sản phẩm](PRODUCT.md). Lịch sử và bằng chứng kiểm tra trước thay đổi: [Kế hoạch v1](PLAN-v1-history.md).

## Mục tiêu đã chốt

- **App (Flutter):** Dành cho Nhân viên (chấm công, lịch làm, yêu cầu) và Quản lý (theo dõi nhanh, phê duyệt gấp).
- **Web Admin (React + TypeScript):** Dành cho Admin/Quản lý để vận hành hệ thống, quản lý nhân sự, sắp xếp lịch làm và xuất báo cáo.
- **Backend (NestJS):** Tập trung API phân quyền chặt chẽ theo mô hình **Công ty/cửa hàng → Chi nhánh → Nhân viên**.
- **Ưu tiên hiện tại:** Phần 4A đã hoàn thành local. Tiếp theo phần 5 — ca làm/phân lịch để phục vụ chấm công nhân viên (phần 6); nghiệm thu Web bổ sung vẫn được theo dõi riêng.

- Một app Flutter iOS/Android; backend NestJS + TypeScript, Prisma và PostgreSQL qua Docker Compose.
- Chạy local, ưu tiên iOS; Android tiếp tục hoãn. Chưa deploy public hoặc tích hợp tính lương.
- Tổ chức: **Công ty/cửa hàng → Chi nhánh → Nhân viên**. Công ty/cửa hàng cùng cấp Organization, không thêm Store; địa điểm chấm công thuộc chi nhánh.
- OWNER vào giao diện quản lý toàn đơn vị; MANAGER vào giao diện quản lý chi nhánh; EMPLOYEE vào giao diện chấm công cá nhân. Quyền lấy từ membership trên server.
- Nhân viên: chấm công, lịch sử công, lịch làm, đăng ký nghỉ/sửa công và thực hiện checklist.
- Chủ/quản lý: nhân sự, phân ca/giờ làm, phê duyệt, bảng công nhân viên và giao/theo dõi checklist.

## Cách quản lý tiến độ

`[x]` là đã hoàn thành và có bằng chứng kiểm tra; `[ ]` là còn việc. Sau mỗi phần cập nhật kết quả, giới hạn và bước tiếp theo. Phần 1–4 hoàn thành theo phạm vi v1, không có nghĩa hai giao diện v2 đã chạy. Giữ số phần cũ, thêm 4A và 8A để không làm sai các tham chiếu kiểm thử hiện có.

## Tổng quan

| Phần | Nội dung | Phụ thuộc | Trạng thái |
| --- | --- | --- | --- |
| 1 | Môi trường và bộ khung | — | Hoàn thành |
| 2 | Dữ liệu và nghiệp vụ nền | 1 | Hoàn thành |
| 3 | Đăng nhập và phân quyền | 2 | Hoàn thành |
| 4 | Quản lý Công ty, Chi nhánh, Nhân sự (App + Web) | 3 | Chức năng chính đã có; còn nghiệm thu Web bổ sung |
| 4A | Tách giao diện App theo tài khoản | 3, 4 | Hoàn thành local; iOS Simulator đã kiểm thử |
| 5 | Ca làm, giờ làm và lịch (App + Web) | 4 | Chưa bắt đầu |
| 6 | Chấm công GPS (App) | 4A, 5 | Chưa bắt đầu |
| 7 | Lịch sử công và Bảng công (App + Web) | 6 | Chưa bắt đầu |
| 8 | Đăng ký nghỉ, sửa công và phê duyệt | 7 | Chưa bắt đầu |
| 8A | Checklist công việc (App + Web) | 5, 4 | Chưa bắt đầu |
| 9 | Báo cáo, chốt kỳ và nghiệm thu | 7, 8, 8A | Chưa bắt đầu |


## Công việc hiện tại và bước tiếp theo

**Hiện tại: chức năng chính phần 4 đã có; phần 4A đã hoàn thành local và kiểm thử iOS Simulator.** Các mục nghiệp vụ chưa có API hiển thị “Chưa sẵn sàng”, chưa ghi nhận chấm công thật.

Thứ tự công việc tiếp theo:

1. Phần 5: BE mẫu ca/phân ca, Web xếp lịch, App nhân viên xem ca hôm nay và lịch làm.
2. Phần 6: BE và App check-in/check-out GPS, sau đó phần 7 lịch sử và bảng công.
3. Phần 8, 8A, 9: nghỉ/sửa công/phê duyệt, checklist, báo cáo và chốt kỳ.
4. Nghiệm thu Web còn mở: tài khoản nhiều công ty, màn hình nhỏ và thiết bị LAN khác. Push/deploy ghi riêng, không đồng nghĩa hoàn thành tính năng.

App đã có EmployeeShell và ManagementShell, chọn/nhớ công ty, xác minh lại quyền khi mở/khôi phục/quay lại từ nền, và các màn nhân sự/chi nhánh/địa điểm/hồ sơ hiện có. Schema ca/công/nghỉ đã có nhưng API và giao diện nghiệp vụ phần 5–8 chưa triển khai; checklist chưa có schema.

Bằng chứng v1: kiểm tra dữ liệu 12 tình huống, auth và management API đạt; backend build, Flutter analyze, 4 widget tests và integration trên iPhone 17 Pro Simulator/iOS 26 đạt. Chưa kiểm tra Android/iPhone vật lý. Dùng `App/scripts/ios-local.sh` để tránh lỗi metadata codesign trong Documents. Cảnh báo dependency Prisma CLI/deepmerge-ts còn trong backlog, cần đánh giá trước mở rộng schema.

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
- [x] App v1: màn quyền và chức năng theo vai trò, xử lý hết phiên và đăng xuất. Hai trang chủ riêng triển khai ở 4A.
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

### Web Admin — kết quả sửa ngày 2026-09-16

- [x] Sửa file địa điểm lỗi cú pháp; chuẩn hóa dependency, lockfile và thêm TypeScript vào build.
- [x] Proxy `/api` local; URL BE cấu hình bằng env; CORS allowlist tùy chọn cho Web gọi BE trực tiếp.
- [x] Đăng nhập, bảo vệ route, khôi phục/gia hạn phiên, đăng xuất thu hồi token và xóa cache.
- [x] Chọn công ty từ membership quản lý; phân biệt OWNER/MANAGER/EMPLOYEE, cache theo công ty/vai trò/chi nhánh.
- [x] Nhân sự: tìm kiếm, phân trang, lọc trạng thái, thêm/sửa trong phạm vi quyền; hiển thị lỗi BE.
- [x] Chi nhánh: OWNER thêm/sửa, MANAGER chỉ xem; địa điểm: thêm/sửa tọa độ, bán kính, múi giờ và trạng thái.
- [x] Cài đặt công ty: OWNER đổi tên, MANAGER chỉ xem. Có loading/error/empty và modal dùng bàn phím.
- [x] Build Web/BE đạt; 8 kiểm thử phiên Web đạt; auth:verify và management:verify với PostgreSQL local đạt.
- [x] Trình duyệt local đăng nhập OWNER, đọc nhân sự/địa điểm từ BE, lưu địa điểm và đăng xuất thành công; MANAGER chỉ có nút sửa nhân viên thường, không có nút thêm/sửa chi nhánh; EMPLOYEE bị chặn khỏi giao diện quản trị. Đăng xuất trở về trang đăng nhập.
- [ ] Nghiệm thu tài khoản có nhiều công ty, các kích thước màn hình và thiết bị LAN khác.
- [ ] Deploy public: cần BE HTTPS và cấu hình VITE_API_BASE_URL/CORS; chưa đẩy code lượt này lên GitHub/Vercel.
- [ ] Admin toàn hệ thống theo yêu cầu quản lý tài khoản/dữ liệu: chưa triển khai. BE hiện chỉ có OWNER/MANAGER/EMPLOYEE; cần chốt quyền và phạm vi dữ liệu riêng trước khi xây, sau ưu tiên nhân sự/chấm công.

Hướng dẫn local và deploy: [WebAdmin/README.md](https://github.com/Vuducdat1997/TimeInWebAdmin/blob/main/README.md). Lịch làm, bảng công, phê duyệt và checklist chưa có trên Web; không tính vào phần quản lý nhân sự đã triển khai.

## 4A. Tách giao diện theo tài khoản — hoàn thành local

- [x] Chốt Công ty/cửa hàng → Chi nhánh → Nhân viên; giữ Organization/Branch/Membership hiện có.
- [x] BE: rà soát contract membership đang hoạt động, vai trò và phạm vi chi nhánh để App chọn đúng ngữ cảnh.
- [x] App: một membership thì vào thẳng; nhiều membership thì chọn đơn vị. Lưu đơn vị gần nhất và xác minh lại với server khi mở app.
- [x] App: EmployeeShell với tab **Chấm công · Lịch làm · Công việc · Yêu cầu · Cá nhân**; lịch sử mở từ Chấm công.
- [x] App: ManagementShell với tab **Tổng quan · Nhân sự · Lịch làm · Phê duyệt · Thêm**; Bảng công và Checklist trong Thêm và lối tắt Tổng quan.
- [x] Đưa màn nhân sự/chi nhánh/địa điểm hiện có vào giao diện quản lý, hồ sơ vào đúng giao diện.
- [x] OWNER xem toàn đơn vị; MANAGER chỉ chi nhánh được giao. Backend chặn cả request trực tiếp ngoài quyền.
- [x] Đổi đơn vị/quyền, vô hiệu hóa hoặc hết phiên: xóa dữ liệu ngữ cảnh cũ, xác minh lại và điều hướng phù hợp.
- [x] Module chưa có API hiển thị trạng thái chưa sẵn sàng; không dùng số liệu giả hoặc nút không có tác dụng.
- [x] Kiểm thử ba vai trò, một/nhiều/không có membership, đổi đơn vị, thu hồi quyền và các đường dẫn trực tiếp.

**Nghiệm thu:** tài khoản chủ vào giao diện quản lý, nhân viên vào giao diện chấm công; chỉ chọn công ty khi cần; không rò dữ liệu giữa đơn vị. Các tính năng chưa xây được ghi rõ, chưa nghiệm thu chấm công thực tế ở bước này.

**Kết quả 2026-09-17:** `flutter analyze` sạch lỗi; 19 widget tests đạt (15 test workspace mới, 4 test cũ). `workspace_flow_test.dart` trên iPhone 17 Pro/iOS 26 đạt cả EMPLOYEE/MANAGER/OWNER: đăng nhập UI, tự vào đúng shell, hồ sơ/nhân sự từ BE, nhớ đơn vị và khôi phục Keychain, đăng xuất. `management_flow_test.dart` đạt tạo chi nhánh/địa điểm/nhân viên và vô hiệu hóa nhân viên qua giao diện mới. Dữ liệu test gắn mã S4A260917A được dọn bằng script có giới hạn local; giữ audit.

**Giới hạn kiểm tra:** nhiều/không có membership, đổi công ty/vai trò, thu hồi quyền, hết phiên, lỗi mạng, chặn callback route quản lý và màn 320px/chữ lớn kiểm tra bằng widget tests. Chưa đăng ký deep link quản lý; chưa thử Android/iPhone vật lý. Không cần migration hay API mới ở phần 4A: `/auth/me` và `/organizations/:id/access` hiện có đủ contract. BE tiếp tục quyết định quyền mọi request. App xác minh lại khi quay về foreground/làm mới và khi mở/lưu màn dữ liệu; không có push thông báo đổi quyền tức thời.

## 5. Ca làm, giờ làm và lịch làm

- [ ] BE: API mẫu ca với giờ bắt đầu/kết thúc, nghỉ, địa điểm và múi giờ; áp dụng policy đã thống nhất.
- [ ] BE: phân ca theo nhân viên/ngày, kiểm tra cùng đơn vị/chi nhánh, trạng thái nhân viên và ca chồng lấn.
- [ ] BE: snapshot giờ/policy/địa điểm khi phân ca; sửa mẫu không đổi lịch sử. Chốt quy tắc sửa/hủy ca đã có công.
- [ ] Quản lý: tạo/sửa mẫu ca, phân ca, đổi/hủy ca hợp lệ; lịch tuần lọc chi nhánh/nhân viên.
- [ ] Nhân viên: lịch tuần, ca hôm nay, giờ làm/nghỉ và địa điểm; ca qua đêm ghi cả ngày kết thúc.
- [ ] Kiểm thử ca thường, qua đêm, chồng lấn, đổi mẫu và truy cập lịch trái quyền.

**Nghiệm thu:** chủ/quản lý phân ca trong phạm vi; nhân viên thấy đúng ca của mình, giờ và ngày; ca không hợp lệ bị từ chối rõ ràng.

## 6. Chấm công GPS

- [ ] BE: check-in/check-out theo ca, giờ server; kiểm tra thứ tự vào/ra, cửa sổ cho phép và quyền cá nhân.
- [ ] BE: kiểm tra khoảng cách, độ mới và độ chính xác vị trí; chống ghi trùng khi retry/request đồng thời.
- [ ] Nhân viên: trang chủ ca hiện tại, trạng thái và nút vào/ra; xin quyền/lấy vị trí khi thao tác.
- [ ] App: xử lý chưa có ca, từ chối quyền, ngoài phạm vi, GPS kém, mất mạng và lỗi server; chỉ báo thành công khi server xác nhận.
- [ ] Quản lý: xem trạng thái vào/ra của nhân viên theo phạm vi từ dữ liệu thật.
- [ ] Kiểm thử vị trí kiểm soát, bấm lặp, request đồng thời, thiếu giờ ra và ca đêm; GPS thiết bị thật ghi riêng khi có.

**Nghiệm thu:** nhân viên vào/ra end-to-end, không trùng bản ghi; quản lý xem đúng phạm vi. GPS không bảo đảm chống giả mạo tuyệt đối.

## 7. Lịch sử chấm công và bảng công nhân viên

- [ ] BE: lịch sử theo khoảng ngày có phân trang; nhân viên chỉ bản thân, quản lý theo phạm vi.
- [ ] BE: tính giờ làm/nghỉ, phút muộn/về sớm, thiếu vào/ra và tổng hợp tháng theo múi giờ.
- [ ] Nhân viên: lịch sử ngày/tháng, chi tiết ca và liên kết gửi sửa công khi phần 8 sẵn sàng.
- [ ] Quản lý: bảng công lọc nhân viên/chi nhánh/tháng, mở chi tiết từng ca.
- [ ] Kiểm thử đối chiếu tổng với chi tiết, ca đêm/qua tháng và dữ liệu thiếu; không tự coi thiếu công là đủ công.

**Nghiệm thu:** hai giao diện dùng cùng kết quả tính công, đúng phạm vi; chỉ quy đổi ngày công sau khi chốt công thức.

## 8. Đăng ký nghỉ, sửa công và phê duyệt

- [ ] Chốt đơn vị nghỉ theo giờ/nửa ngày/ngày, cách tác động lịch/công và giới hạn gửi/hủy yêu cầu.
- [ ] BE: gửi, xem trạng thái, duyệt/từ chối; kiểm tra người duyệt, không tự duyệt và không xử lý lặp.
- [ ] BE: giữ sự kiện công gốc, lưu lý do/người thao tác/giá trị trước sau; tính lại kết quả theo quyết định duyệt.
- [ ] Nhân viên: tab Yêu cầu gồm đăng ký nghỉ, sửa công, lịch sử và lý do từ chối.
- [ ] Quản lý: tab Phê duyệt, lọc loại/trạng thái, chi tiết dữ liệu gốc/đề xuất và xác nhận thao tác.
- [ ] Kiểm thử duyệt/từ chối, trái quyền, duyệt đồng thời và kết quả công sau duyệt.

**Nghiệm thu:** nhân viên theo dõi được yêu cầu; quản lý duyệt đúng phạm vi; từ chối không đổi công, duyệt có lịch sử đầy đủ.

## 8A. Checklist công việc

- [ ] Chốt quy tắc hoàn tất/mở lại checklist theo đề xuất trong PRODUCT.md.
- [ ] BE: migration mẫu checklist/mục việc, lần giao, kết quả từng mục và lịch sử; ràng buộc đơn vị/chi nhánh/người nhận.
- [ ] BE: tạo/sửa mẫu, giao một nhân viên theo ca/ngày và hạn; snapshot nội dung khi giao.
- [ ] BE: hoàn thành từng mục bằng giờ server, tiến độ/quá hạn, hủy/mở lại có audit; cập nhật lặp không sinh kết quả trùng.
- [ ] Quản lý: tạo mẫu, giao việc, xem tiến độ/lọc chi nhánh, nhân viên, ngày và việc quá hạn.
- [ ] Nhân viên: tab Công việc, xem việc được giao, đánh dấu và ghi chú; trang chủ có việc sắp đến hạn.
- [ ] Kiểm thử cách ly quyền, sửa mẫu không đổi việc cũ, cập nhật đồng thời và bảo toàn lịch sử khi hủy/mở lại.
- [ ] Kiểm thử hoàn thành checklist không tự chấm công hoặc thay đổi giờ công.

**Nghiệm thu:** quản lý giao → nhân viên thực hiện → quản lý thấy tiến độ thật. Ảnh minh chứng, việc nhóm và lặp tự động để sau MVP.

## 9. Tổng quan, báo cáo và hoàn thiện local

- [ ] BE/App: tổng quan quản lý theo ngày/chi nhánh với ca, công, yêu cầu và checklist từ API nguồn thật; chủ xem toàn đơn vị, quản lý trong chi nhánh.
- [ ] Xuất CSV bảng công theo quyền, đối chiếu tổng và chi tiết.
- [ ] Chốt/mở lại kỳ theo quyền chủ, audit và chặn điều chỉnh kỳ đã chốt; kiểm thử cả luồng phê duyệt.
- [ ] Hoàn thiện tải/rỗng/lỗi/mất phiên và điều hướng trên cả hai giao diện.
- [ ] Nghiệm thu iOS Simulator; ghi riêng GPS/iPhone vật lý chưa xác minh. Android hoãn, chỉ mở lại kiểm tra khi triển khai nền tảng đó.
- [ ] Thử backup/restore local; cập nhật README, tài khoản mẫu và lệnh kiểm tra.
- [ ] Review phân quyền, dữ liệu nhạy cảm và lỗi nghiệp vụ còn mở.

**Nghiệm thu:** chạy từ README; chủ tạo nhân viên → phân ca/giao checklist → nhân viên xem lịch/chấm công/làm việc/gửi yêu cầu → quản lý duyệt/xem bảng công → xuất báo cáo/chốt kỳ. Không dùng dữ liệu giả để chứng minh nghiệp vụ.

## Quyết định còn mở khi đến phần liên quan

- Phần 5–6: rà soát mặc định đã ghi trong DATA-RULES.md về nghỉ, đi muộn, cửa sổ vào/ra và GPS trước viết API; ghi thay đổi nếu có.
- Phần 7: làm tròn giờ, quy đổi ngày công nếu cần.
- Phần 8: loại nghỉ và tác động công; chủ/quản lý có cần thêm quyền làm việc cá nhân không (hiện không hiển thị mặc định).
- Phần 8A: khóa khi hoàn tất, quản lý mở lại có lý do; hạn và cách giao checklist.
- Phần 9: thời hạn lưu dữ liệu và quy trình chốt/mở lại kỳ.

## Ngoài phạm vi hiện tại

Trong giai đoạn local này chưa triển khai deploy BE public/phát hành store, tính lương, SMS OTP, nhận diện khuôn mặt, QR động, chấp nhận công offline tự động, microservices, Redis và push qua dịch vụ ngoài. Web quản trị thuộc phạm vi dự án. Web đã có chọn công ty từ membership hiện hữu, nhưng chưa nghiệm thu đầy đủ luồng nhiều công ty; mỗi MANAGER vẫn gắn một chi nhánh. Onboarding công ty mới, mời email đã có và cấp nhiều chi nhánh cho cùng một quản lý chưa triển khai. Admin toàn hệ thống là yêu cầu còn mở, tách khỏi quyền OWNER.

## Nhật ký v2

| Ngày | Thay đổi | Kiểm tra / bước tiếp theo |
| --- | --- | --- |
| 2026-09-17 | Hoàn thành 4A: hai shell Flutter, chọn/nhớ công ty theo tài khoản, xác minh lại quyền và tái sử dụng màn quản lý | Analyze, 19 widget tests và integration ba vai trò + CRUD quản lý trên iOS Simulator đạt. Tiếp theo phần 5 |
| 2026-09-16 | Hoàn thiện chức năng chính Web nhân sự/chi nhánh/địa điểm/cài đặt; sửa build, kết nối BE và phiên đăng nhập; đồng bộ lại phạm vi và thứ tự plan | Web/BE build đạt, 8 test phiên Web và kiểm thử auth/management BE đạt; kiểm tra UI ba vai trò. Còn nghiệm thu nhiều công ty, responsive và thiết bị LAN khác |
| 2026-09-15 | Chốt cấu trúc Công ty/cửa hàng → Chi nhánh → Nhân viên; thiết kế hai giao diện, thêm 4A và 8A | Chỉ cập nhật tài liệu; bắt đầu code phần 4A khi tiếp tục triển khai |

Chi tiết kết quả phần 1–4 và các vấn đề đã xử lý xem [lịch sử v1](PLAN-v1-history.md). Kết quả kiểm thử v1 không thay cho nghiệm thu các tính năng v2.

## Quy tắc cấp tài khoản và công ty của nhân viên — cập nhật 2026-09-17

- Chỉ OWNER/MANAGER được tạo tài khoản nhân viên trong phạm vi quản lý; không có tự đăng ký.
- Công ty và chi nhánh do cấp quản lý gán khi tạo tài khoản. Nhân viên vào thẳng đơn vị được phân công, không có chọn/đổi công ty và không dùng lựa chọn công ty lưu trên thiết bị.
- OWNER/MANAGER chỉ chọn trong các membership quản lý được cấp. Quy tắc nhiều công ty trước đây không áp dụng cho EMPLOYEE.
- Nếu dữ liệu cũ gán EMPLOYEE vào nhiều công ty đang hoạt động, App yêu cầu liên hệ quản lý; BE chặn truy cập nghiệp vụ cho tới khi phân công được điều chỉnh. Không tự chọn công ty đầu tiên.
- BE đã chặn nhân viên tạo tài khoản hoặc tự sửa membership. MANAGER chỉ tạo/sửa nhân viên thường trong chi nhánh; OWNER quản lý trong công ty. Chuyển công ty giữa các đơn vị chưa có luồng quản trị riêng, cần thiết kế quyền nhận/chuyển; không cho nhân viên tự thực hiện.
