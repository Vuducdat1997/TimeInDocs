# PLAN-APP — Ứng dụng Flutter TimeIn

Kế hoạch chi tiết cho repo [`TimeInApp`](https://github.com/Vuducdat1997/TimeInApp). Điều hướng chung và ma trận tiến độ: [PLAN.md](PLAN.md). Quy ước code của repo: `App/AGENTS.md`.

Tài liệu này **chỉ quản lý phần App**. Tài liệu này không quyết định nghiệp vụ hay quyền — mọi quyền do BE cấp; App chỉ khai báo hợp đồng API mình cần.

## Thiết kế tham chiếu

Bộ 8 màn hình nhân viên do chủ dự án cung cấp ngày 2026-09-17:

| # | Màn hình | Feature | Trạng thái |
| --- | --- | --- | --- |
| 1 | Trang Chủ | APP-9.5, APP-9.6 | Đã dựng giao diện |
| 2 | Lịch Làm Việc | APP-5.1 → APP-5.6 | Chưa |
| 3 | Chấm Công (khuôn mặt) | APP-6.1 → APP-6.9 | Chưa |
| 4 | Checklist Công Việc | APP-8A.1, APP-8A.5, APP-8A.6 | Chưa |
| 5 | Cá Nhân | APP-9.7, APP-9.8, APP-9.9 | Chưa |
| 6 | Đơn Xin Phép & Đổi Ca | APP-8.4 → APP-8.7 | Chưa |
| 7 | Chấm Công Thành Công | APP-6.7 | Chưa |
| 8 | Chấm Công Thất Bại | APP-6.7 | Chưa |

Thiết kế này **rộng hơn plan gốc**: có ba nội dung chưa từng nằm trong phạm vi dự án — chấm công bằng khuôn mặt, đổi ca và bảng lương/thu nhập — cộng với thay đổi thanh tab của nhân viên. Các feature đó đã được ghi vào plan và **đánh dấu "cần chốt phạm vi"**; không feature nào trong số đó được coi là đã chốt cho tới khi có quyết định ở `PLAN.md`.

Nguồn thiết kế đã lưu trong repo: `docs/design/trang-chu/` gồm `DESIGN.md` (token màu, chữ, khoảng cách), `code.html` (bản dựng tham chiếu) và `screen.png`.

## Trạng thái: `[x]` xong và có bằng chứng chạy thật · `[ ]` còn việc

## Yêu cầu đầu vào của phần App

Mỗi feature App chỉ bắt đầu khi đầu vào tương ứng đã sẵn sàng:

| Đầu vào | Từ đâu | Dùng cho |
| --- | --- | --- |
| Endpoint + kiểu dữ liệu trả về | `PLAN-BE.md` | Mọi feature gọi API |
| Mã lỗi và ngữ nghĩa (401 vs 403) | `BE/docs/AUTH.md` | Điều hướng khi mất phiên/mất quyền |
| Quy tắc nghiệp vụ hiển thị (cửa sổ vào ca, ngưỡng muộn) | `BE/docs/DATA-RULES.md` | Hiển thị đúng, không tự đoán |
| Thiết kế màn hình | `docs/PRODUCT.md` | Tab, luồng, trạng thái |
| Quyết định còn mở | `PLAN.md` | Không code khi nghiệp vụ chưa chốt |

**Nguyên tắc bắt buộc:** module chưa có API thì hiển thị trạng thái "chưa sẵn sàng" bằng `UnavailablePage`/`UnavailableCard`. Không hiển thị số liệu giả, không gắn nút không có tác dụng.

## Bàn giao của phần App

| Đầu ra | Nơi nhận |
| --- | --- |
| Bộ màn hình theo vai trò | Người dùng cuối |
| Danh sách yêu cầu dữ liệu thực tế đã dùng | Đối chiếu với `PLAN-BE.md` |
| Widget test + integration test | Bằng chứng nghiệm thu |
| Số lượng test thực tế | `App/README.md`, `docs/PLAN.md` |

---

## APP-1 — Bộ khung

- [x] **APP-1.1** Khởi tạo Flutter trong `App/`, kết nối được API `/health`.
- [x] **APP-1.2** Màn hình hiển thị trạng thái API/database, có trạng thái lỗi và thử lại.
- [x] **APP-1.3** Cấu hình API URL cho iOS Simulator (`127.0.0.1`), Android Emulator (`10.0.2.2`) và điện thoại thật (`--dart-define=API_BASE_URL`).
- [x] **APP-1.4** Theme dùng chung; màu `ink`, `blue`, `muted` định nghĩa một nơi.
- [x] **APP-1.5** Đã lưu bộ thiết kế màn nhân viên vào `docs/design/trang-chu/` (`DESIGN.md`, `code.html`, `screen.png`).
- [x] **APP-1.6** Logo thương hiệu: logo gốc ở `assets/branding/logo.png`; `scripts/make-icons.sh` làm phẳng nền trong suốt và sinh 19 icon iOS + 5 icon Android; hiển thị ở màn đăng nhập. Chạy lại script khi logo đổi, không cần thêm dependency.

**Đầu vào:** `PLAN-BE.md` BE-1.5 (`/health`) và BE-1.4 (biến môi trường); ảnh thiết kế và logo do chủ dự án cung cấp.
**Đầu ra:** `lib/main.dart`, `lib/auth/login_page.dart` (nơi định nghĩa màu cũ), `lib/theme/app_colors.dart` (bảng màu theo thiết kế), `assets/branding/`, `scripts/make-icons.sh`, `docs/design/trang-chu/`.
**Nghiệm thu:** chạy trên iOS Simulator gọi được `/health`; `flutter analyze` sạch lỗi; đổi `API_BASE_URL` bằng `--dart-define` hoạt động trên máy thật; thiết kế mở được từ link trong plan; icon hiển thị đúng trên màn hình chính của máy ảo.

## APP-3 — Đăng nhập và phiên

- [x] **APP-3.1** Màn đăng nhập: validation email/mật khẩu, hiện/ẩn mật khẩu, loading, lỗi giữ nguyên email đã nhập.
- [x] **APP-3.2** Lưu phiên trong Keychain, khôi phục phiên khi mở lại; mất mạng thì giữ dữ liệu và cho thử lại.
- [x] **APP-3.3** Tự gia hạn access token khi gặp 401; gộp nhiều request đồng thời vào một lần refresh.
- [x] **APP-3.4** Đăng xuất thu hồi phiên trên server; không báo thành công nếu không liên lạc được server.
- [x] **APP-3.5** Màn "Quên mật khẩu" hướng dẫn liên hệ quản lý (chưa có đặt lại qua email).

**Đầu vào:** `PLAN-BE.md` BE-3.1 → BE-3.3; mã lỗi 401 từ `BE/docs/AUTH.md`.
**Đầu ra:** `lib/auth/auth_client.dart`, `lib/auth/login_page.dart`, `test/widget_test.dart`, `integration_test/auth_flow_test.dart`.
**Nghiệm thu:** đăng nhập/đăng xuất end-to-end trên Simulator; Keychain đọc lại được bằng client mới; sai mật khẩu hiện đúng thông báo và giữ email; refresh đồng thời chỉ tạo một phiên; logout khi mất mạng báo lỗi chứ không báo thành công.

## APP-4 — Quản lý công ty, chi nhánh, nhân sự

- [x] **APP-4.1** Danh sách nhân viên: tìm kiếm, lọc trạng thái, phân trang 25 bản ghi.
- [x] **APP-4.2** Chi tiết và biểu mẫu nhân viên: thêm mới, sửa mã/chi nhánh/vai trò, vô hiệu hóa.
- [x] **APP-4.3** Quản lý chi nhánh và địa điểm chấm công (tọa độ, bán kính, múi giờ, trạng thái).
- [x] **APP-4.4** Hồ sơ cá nhân cho nhân viên và thông tin công ty.
- [x] **APP-4.5** Xác nhận trước khi thay đổi quyền hoặc vô hiệu hóa; chặn tự sửa chính mình.
- [ ] **APP-4.6** Nghiệm thu lại toàn bộ đường dẫn trực tiếp sau khi thêm màn mới ở phần 5.

**Đầu vào:** `PLAN-BE.md` BE-4.1 → BE-4.5; giới hạn ký tự của `BE/docs/MANAGEMENT.md`.
**Đầu ra:** `lib/management/management_page.dart`, `test/management_test.dart`, `integration_test/management_flow_test.dart`.
**Nghiệm thu:** OWNER sửa được toàn công ty; MANAGER chỉ thấy nút sửa nhân viên thường và địa điểm chi nhánh mình; EMPLOYEE bị chặn; form chặn dữ liệu sai trước khi gọi API; `flutter analyze` sạch.

## APP-4A — Giao diện theo tài khoản

- [x] **APP-4A.1** Một membership thì vào thẳng; nhiều thì chọn đơn vị; EMPLOYEE không chọn/đổi công ty.
- [x] **APP-4A.2** Nhớ đơn vị gần nhất theo ID tài khoản trong Keychain, luôn xác minh lại với server.
- [x] **APP-4A.3** `EmployeeShell`: **Chấm công · Lịch làm · Công việc · Yêu cầu · Cá nhân**.
- [x] **APP-4A.4** `ManagementShell`: **Tổng quan · Nhân sự · Lịch làm · Phê duyệt · Thêm**.
- [x] **APP-4A.5** Xác minh lại quyền khi mở app, quay lại từ nền và khi mở/lưu màn dữ liệu; bỏ màn/form cũ khi quyền đổi.
- [x] **APP-4A.6** Đổi quyền/vô hiệu hóa/hết phiên: xóa ngữ cảnh cũ và điều hướng lại đúng; 401 về đăng nhập, 403 mở lại màn chọn công ty.
- [x] **APP-4A.7** Module chưa có API hiển thị trạng thái chưa sẵn sàng, không dùng số liệu giả.
- [x] **APP-4A.8** Chống race bằng biến `generation` cho mọi thao tác bất đồng bộ.
- [ ] **APP-4A.9** Đăng ký deep link tới module quản lý (chưa làm; hiện kiểm tra vai trò bằng callback).
- [x] **APP-4A.10** Thanh điều hướng nhân viên theo thiết kế 2026-09-17: **Trang Chủ · Lịch · Chụp ảnh (nút giữa) · Checklist · Cá Nhân**. Tab **Yêu cầu** đã bỏ; lịch sử công và đơn từ mở từ tab Cá Nhân (APP-8.7). Giữ khoá `tab-0`…`tab-4` nên test cũ vẫn dùng được.

**Đầu vào:** `PLAN-BE.md` BE-4A.1, BE-4A.2; ma trận quyền trong `docs/PRODUCT.md`; thiết kế trong `docs/design/trang-chu/`.
**Đầu ra:** `lib/workspace/session_gate.dart`, `lib/workspace/workspace_shell.dart`, `test/workspace_test.dart`, `integration_test/workspace_flow_test.dart`.
**Nghiệm thu:** ba vai trò vào đúng shell; một/nhiều/không có membership xử lý đúng; dữ liệu công ty cũ bị xóa khi đổi đơn vị; route quản lý không mở được từ callback của nhân viên; 5 tab dùng được ở màn 320px và chữ lớn. Nếu đổi thanh tab: đơn từ vẫn tới được từ ít nhất một lối vào, và test workspace hiện có được cập nhật. **Bằng chứng 2026-09-17:** `flutter analyze` sạch, 21 widget test đạt, `workspace_flow_test.dart` và `management_flow_test.dart` đạt trên iPhone 17 Pro/iOS 26.

## APP-5 — Lịch làm

- [ ] **APP-5.1** Nhân viên: lịch tuần, ca hôm nay, giờ làm/nghỉ và địa điểm; ca qua đêm ghi rõ ngày kết thúc.
- [ ] **APP-5.2** Quản lý: lịch tuần lọc theo chi nhánh và nhân viên.
- [ ] **APP-5.3** Quản lý: tạo/sửa mẫu ca, phân ca, đổi/hủy ca trong phạm vi quyền.
- [ ] **APP-5.4** Xử lý trạng thái chưa phân ca, lỗi mạng và lỗi quyền trên màn lịch.
- [ ] **APP-5.5** Lịch tháng: chấm ngày có ca, ngày nghỉ và ngày chưa xếp; có chú thích màu; tổng giờ theo ngày đã chọn.
- [ ] **APP-5.6** Chọn một ngày trong tháng để xem danh sách ca của ngày đó; lối vào "Xem lịch tháng" từ tab Lịch.

**Đầu vào:** `PLAN-BE.md` BE-5.1 → BE-5.5; quyết định về sửa/hủy ca đã có công (mục còn mở của `PLAN.md`).
**Đầu ra:** tab **Lịch** của cả hai shell, gồm lịch tuần và lịch tháng; widget test cho ca qua đêm, màn rỗng và chú thích lịch.
**Nghiệm thu:** nhân viên thấy đúng ca của mình, đúng giờ và ngày; quản lý thấy đúng phạm vi chi nhánh; ca không hợp lệ hiển thị lỗi rõ ràng từ BE; lịch tuần đổi tuần không cần tải lại toàn app; lịch tháng đánh dấu đúng ba trạng thái ngày và khớp với lịch tuần cùng kỳ.

## APP-6 — Chấm công GPS

- [ ] **APP-6.1** Trang chủ ca hiện tại: trạng thái (chưa phân ca, chưa vào, đang làm, đã ra, thiếu công) và nút vào/ra.
- [ ] **APP-6.2** Xin quyền và lấy vị trí khi người dùng thao tác; không lấy vị trí khi chưa cần.
- [ ] **APP-6.3** Xử lý từ chối quyền, ngoài phạm vi, GPS kém, mất mạng và lỗi server; chỉ báo thành công khi server xác nhận.
- [ ] **APP-6.4** Quản lý: xem trạng thái vào/ra của nhân viên theo phạm vi từ dữ liệu thật.
- [ ] **APP-6.5** Chống bấm lặp: khóa nút trong lúc chờ và dùng idempotency key khi retry.
- [ ] **APP-6.6** Xác thực khuôn mặt trong màn chấm công: camera trước, khung hướng dẫn khuôn mặt, hiển thị trạng thái "đã xác thực" và "vị trí hợp lệ" trước khi cho gửi. **Cần chốt phạm vi.**
- [ ] **APP-6.7** Màn hình kết quả sau chấm công: thành công (kèm ca và giờ đã ghi) và thất bại (kèm lý do GPS/khuôn mặt, nút **Thử lại** và **Đã hiểu**).
- [ ] **APP-6.8** Đăng ký và cập nhật khuôn mặt từ tab Cá Nhân, kèm hướng dẫn chụp và xác nhận trước khi lưu. **Cần chốt phạm vi.**
- [ ] **APP-6.9** Nhắc nhở trước giờ vào ca; phụ thuộc quyết định về phạm vi thông báo trong `PLAN.md`.

**Đầu vào:** `PLAN-BE.md` BE-6.1 → BE-6.6; quy tắc cửa sổ vào/ra và GPS trong `BE/docs/DATA-RULES.md` phải chốt trước; quyết định về phạm vi nhận diện khuôn mặt và quyền camera trong `Info.plist`.
**Đầu ra:** trang **Chấm Công** thật (thay `UnavailableCard`) gồm camera, trạng thái xác thực, nút chấm công; hai màn kết quả; màn đăng ký khuôn mặt; quyền camera và vị trí trong `Info.plist`; widget test cho các nhánh lỗi.
**Nghiệm thu:** vào/ra end-to-end trên Simulator với vị trí kiểm soát; không sinh bản ghi trùng khi bấm lặp hoặc retry; mọi nhánh lỗi có thông báo hướng dẫn cụ thể; màn kết quả chỉ báo thành công khi server đã xác nhận; từ chối quyền camera hoặc vị trí không làm app treo và vẫn có đường thoát; GPS và camera trên thiết bị thật ghi riêng là chưa xác minh.

## APP-7 — Lịch sử và bảng công

- [ ] **APP-7.1** Nhân viên: lịch sử theo ngày/tháng, chi tiết ca, tổng giờ, đi muộn, thiếu công.
- [ ] **APP-7.2** Liên kết gửi sửa công từ chi tiết ca (bật khi phần 8 sẵn sàng).
- [ ] **APP-7.3** Quản lý: bảng công lọc nhân viên/chi nhánh/tháng, mở chi tiết từng ca.

**Đầu vào:** `PLAN-BE.md` BE-7.1 → BE-7.3; quyết định làm tròn giờ và quy đổi ngày công.
**Đầu ra:** màn **Lịch sử chấm công** mở từ tab Chấm công; màn **Bảng công** trong shell quản lý.
**Nghiệm thu:** tổng trên màn hình khớp tổng chi tiết; ca đêm và ca qua tháng hiển thị đúng; dữ liệu thiếu hiển thị là thiếu, không tự coi là đủ công; hai giao diện dùng cùng kết quả từ BE.

## APP-8 — Yêu cầu và phê duyệt

- [ ] **APP-8.1** Tab **Yêu cầu** cho nhân viên: đăng ký nghỉ, gửi sửa công, xem trạng thái và lý do từ chối.
- [ ] **APP-8.2** Tab **Phê duyệt** cho quản lý: lọc loại/trạng thái, xem dữ liệu gốc và đề xuất, xác nhận trước khi duyệt.
- [ ] **APP-8.3** Không hiển thị nút duyệt cho yêu cầu của chính mình.
- [ ] **APP-8.4** Form **Đơn Xin Phép**: loại phép (có lương / không lương), từ ngày–đến ngày kèm giờ, số ngày nghỉ, ca nằm trong khoảng nghỉ, lý do, và người duyệt dự kiến.
- [ ] **APP-8.5** Hiển thị quy tắc thời hạn gửi đơn (ví dụ phải gửi trước 24 giờ) ngay trên form, và chặn gửi khi vi phạm kèm lý do rõ ràng.
- [ ] **APP-8.6** **Đổi ca**: chọn ca của mình, chọn nhân viên nhận, gửi yêu cầu và theo dõi trạng thái. **Cần chốt phạm vi.**
- [ ] **APP-8.7** Màn **Đơn của tôi**: lọc theo loại (nghỉ / đổi ca) và trạng thái; mở từ Trang Chủ và Cá Nhân thay cho tab **Yêu cầu** đã bỏ.

**Đầu vào:** `PLAN-BE.md` BE-8.1 → BE-8.6; quyết định về loại nghỉ, tác động công và đổi ca.
**Đầu ra:** form đơn xin phép, màn đổi ca, màn Đơn của tôi (thay `UnavailablePage`); widget test cho không-tự-duyệt và cho chặn gửi sai thời hạn.
**Nghiệm thu:** số ngày nghỉ tính đúng theo khoảng thời gian đã chọn; ca bị ảnh hưởng trong khoảng nghỉ hiển thị đúng; gửi đơn vi phạm thời hạn bị chặn ngay trên form chứ không chỉ bị BE từ chối; nhân viên theo dõi được yêu cầu của mình; quản lý duyệt trong phạm vi; từ chối hiển thị lý do; thao tác lặp không tạo hai lần xử lý.

## APP-8A — Checklist công việc

- [ ] **APP-8A.1** Tab **Công việc**: danh sách việc được giao, tiến độ, hạn, đánh dấu từng mục và ghi chú.
- [ ] **APP-8A.2** Trang chủ nhân viên hiển thị việc sắp đến hạn.
- [ ] **APP-8A.3** Quản lý: tạo mẫu, giao việc, theo dõi tiến độ và việc quá hạn.
- [ ] **APP-8A.4** Sau khi hoàn tất toàn bộ, không cho tự bỏ đánh dấu; yêu cầu quản lý mở lại.
- [ ] **APP-8A.5** Checklist nhóm **theo ca trong ngày** (ví dụ Ca Sáng / Ca Chiều / Ca Tối) với tiến độ tổng, số việc đã xong, số việc còn lại và phần trăm hoàn thành.
- [ ] **APP-8A.6** Bộ lọc theo ngày và theo ca trên màn checklist.

**Đầu vào:** `PLAN-BE.md` BE-8A.1 → BE-8A.5 (chưa có schema); checklist phải gắn được với ca, không chỉ với ngày.
**Đầu ra:** tab **Checklist**, màn giao việc trong shell quản lý.
**Nghiệm thu:** quản lý giao → nhân viên thực hiện → quản lý thấy tiến độ thật; việc hiển thị đúng nhóm ca; tiến độ tổng khớp tổng số mục đã đánh dấu; hoàn thành checklist không làm thay đổi giờ công; cập nhật lặp không sinh kết quả trùng.

## APP-9 — Tổng quan và hoàn thiện

- [ ] **APP-9.1** Tab **Tổng quan** cho quản lý: nhân sự theo lịch, đã vào ca, thiếu công, yêu cầu chờ duyệt, checklist quá hạn — từ API nguồn thật.
- [ ] **APP-9.2** Hoàn thiện trạng thái tải/rỗng/lỗi/mất phiên và điều hướng trên cả hai shell.
- [ ] **APP-9.3** Nghiệm thu trên iOS Simulator; ghi riêng GPS và iPhone vật lý chưa xác minh.
- [ ] **APP-9.4** Review lại toàn bộ: màn 320px, chữ lớn, xoay màn hình, mất mạng giữa luồng.
- [x] **APP-9.5** **Trang Chủ** nhân viên: danh tính theo thiết kế, ngày giờ, băng ca có trạng thái suy ra từ giờ thật, danh sách công việc hôm nay kèm bộ đếm tiến độ và đánh dấu được, dải GPS. Xem ghi chú triển khai bên dưới.
- [ ] **APP-9.6** Chuông thông báo trên Trang Chủ: số chưa đọc, danh sách thông báo, đánh dấu đã đọc. **Cần chốt phạm vi.**
- [ ] **APP-9.7** Tab **Cá Nhân** phần thông tin: ảnh, tên, chức danh, mã nhân viên, vai trò, trạng thái; thống kê tháng gồm số ca và tổng giờ.
- [ ] **APP-9.8** Tab **Cá Nhân** các lối vào: đăng ký khuôn mặt, cài đặt nhắc nhở, đổi đơn vị làm việc, đổi mật khẩu, đăng xuất và số phiên bản app. *Danh sách mục cần chốt lại với chủ dự án vì ảnh thiết kế có chữ nhỏ.*
- [ ] **APP-9.9** **Bảng lương và thu nhập** theo tháng. **Cần chốt phạm vi** — hiện nằm ngoài phạm vi dự án.

**Đầu vào:** tất cả module nguồn (5, 6, 7, 8, 8A) đã có API thật; `PLAN-BE.md` BE-7.4 (thống kê cá nhân), BE-9.6 (thông báo), BE-9.7 (bảng lương), BE-3.7 (đổi mật khẩu), BE-4.7 (chức danh).
**Đầu ra:** Trang Chủ thật thay `UnavailableCard` hiện tại; tab Cá Nhân đầy đủ; báo cáo nghiệm thu theo từng shell; cập nhật `App/README.md`.
**Nghiệm thu:** chủ tạo nhân viên → phân ca → nhân viên chấm công → gửi yêu cầu → quản lý duyệt → xem bảng công, toàn bộ trên app thật, không dùng dữ liệu giả; Trang Chủ không hiển thị số liệu nào khi module nguồn chưa có API; thống kê tháng khớp dữ liệu bảng công cùng kỳ; mọi lối vào trong Cá Nhân đều có tác dụng thật, không có nút chết. Android vẫn hoãn.

### Ghi chú triển khai — Trang Chủ, 2026-09-17

Đã code xong giao diện Trang Chủ (`lib/home/home_page.dart`, `lib/home/home_data.dart`, `lib/theme/app_colors.dart`). Vì BE chưa có API cho ca làm, checklist và GPS, phần dữ liệu được tách làm hai loại rõ ràng:

| Phần trên màn hình | Nguồn | Ghi chú |
| --- | --- | --- |
| Tên nhân viên, công ty, chi nhánh | **API thật** (`/auth/me` + `/organizations/:id/profile`) | Không có ảnh đại diện trong schema nên hiển thị chữ cái đầu |
| Ca làm việc hôm nay | Dữ liệu mẫu (chờ BE-5) | Trạng thái ca suy ra từ giờ thật, không gán cứng |
| Công việc hôm nay | Dữ liệu mẫu (chờ BE-8A) | Bộ đếm chạy đúng theo thao tác đánh dấu |
| Định vị GPS | Dữ liệu mẫu (chờ BE-6) | **Chưa xin quyền vị trí trên thiết bị** — chưa thêm dependency `geolocator` |

Trong lúc còn dữ liệu mẫu, màn hình hiển thị nhãn **"DỮ LIỆU MẪU"** ở đầu trang, đúng nguyên tắc không hiển thị số liệu giả như thật. Khi nối API: thay `HomeData.sample` bằng lời gọi API và đặt `HomeData.isDemo = false` để nhãn tự mất.

**Chưa làm ở màn này:** nút chuông thông báo mới chỉ hiển thị, chưa mở được (chờ BE-9.6); chưa có deep link; đồng hồ trên màn cập nhật theo lần vẽ chứ chưa tự nhích mỗi phút (tránh `Timer.periodic` làm widget test không kết thúc).

**Bằng chứng:** `flutter analyze` sạch lỗi; 22 widget test đạt, gồm test mới "Home shows demo notice and checklist counter follows ticks" và test cũ "All five tabs stay usable at 320px and large text" vẫn đạt sau khi đổi thanh tab.

**Kiểm tra trên máy ảo 2026-09-17:** app tự khôi phục phiên từ Keychain, vào thẳng Trang Chủ sau khi tắt và mở lại, không hỏi lại mật khẩu (đúng như thiết kế lưu phiên). Ba lỗi lệch thiết kế đã phát hiện qua ảnh chụp và sửa xong: nhãn trạng thái ca bị kéo giãn hết chiều ngang (thiếu `mainAxisSize.min`), tên quản lý bị cắt cụt (`Spacer` tranh chỗ với `Flexible`), và thanh xanh trên thẻ ca hiển thị cả ở ca đã kết thúc (giờ chỉ dành cho ca đang chạy). Icon logo đã lên màn hình chính của máy ảo.
