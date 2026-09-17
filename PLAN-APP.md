# PLAN-APP — Ứng dụng Flutter TimeIn

Kế hoạch chi tiết cho repo [`TimeInApp`](https://github.com/Vuducdat1997/TimeInApp). Điều hướng chung và ma trận tiến độ: [PLAN.md](PLAN.md). Quy ước code của repo: `App/AGENTS.md`.

Tài liệu này **chỉ quản lý phần App**. Tài liệu này không quyết định nghiệp vụ hay quyền — mọi quyền do BE cấp; App chỉ khai báo hợp đồng API mình cần.

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

**Đầu vào:** `PLAN-BE.md` BE-1.5 (`/health`) và BE-1.4 (biến môi trường).
**Đầu ra:** `lib/main.dart`, `lib/auth/login_page.dart` (nơi định nghĩa màu), `analysis_options.yaml`.
**Nghiệm thu:** chạy trên iOS Simulator gọi được `/health`; `flutter analyze` sạch lỗi; đổi `API_BASE_URL` bằng `--dart-define` hoạt động trên máy thật.

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

**Đầu vào:** `PLAN-BE.md` BE-4A.1, BE-4A.2; ma trận quyền trong `docs/PRODUCT.md`.
**Đầu ra:** `lib/workspace/session_gate.dart`, `lib/workspace/workspace_shell.dart`, `test/workspace_test.dart`, `integration_test/workspace_flow_test.dart`.
**Nghiệm thu:** ba vai trò vào đúng shell; một/nhiều/không có membership xử lý đúng; dữ liệu công ty cũ bị xóa khi đổi đơn vị; route quản lý không mở được từ callback của nhân viên; 5 tab dùng được ở màn 320px và chữ lớn. **Bằng chứng 2026-09-17:** `flutter analyze` sạch, 21 widget test đạt, `workspace_flow_test.dart` và `management_flow_test.dart` đạt trên iPhone 17 Pro/iOS 26.

## APP-5 — Lịch làm

- [ ] **APP-5.1** Nhân viên: lịch tuần, ca hôm nay, giờ làm/nghỉ và địa điểm; ca qua đêm ghi rõ ngày kết thúc.
- [ ] **APP-5.2** Quản lý: lịch tuần lọc theo chi nhánh và nhân viên.
- [ ] **APP-5.3** Quản lý: tạo/sửa mẫu ca, phân ca, đổi/hủy ca trong phạm vi quyền.
- [ ] **APP-5.4** Xử lý trạng thái chưa phân ca, lỗi mạng và lỗi quyền trên màn lịch.

**Đầu vào:** `PLAN-BE.md` BE-5.1 → BE-5.5; quyết định về sửa/hủy ca đã có công (mục còn mở của `PLAN.md`).
**Đầu ra:** tab **Lịch làm** của cả hai shell; widget test cho ca qua đêm và màn rỗng.
**Nghiệm thu:** nhân viên thấy đúng ca của mình, đúng giờ và ngày; quản lý thấy đúng phạm vi chi nhánh; ca không hợp lệ hiển thị lỗi rõ ràng từ BE; lịch tuần đổi tuần không cần tải lại toàn app.

## APP-6 — Chấm công GPS

- [ ] **APP-6.1** Trang chủ ca hiện tại: trạng thái (chưa phân ca, chưa vào, đang làm, đã ra, thiếu công) và nút vào/ra.
- [ ] **APP-6.2** Xin quyền và lấy vị trí khi người dùng thao tác; không lấy vị trí khi chưa cần.
- [ ] **APP-6.3** Xử lý từ chối quyền, ngoài phạm vi, GPS kém, mất mạng và lỗi server; chỉ báo thành công khi server xác nhận.
- [ ] **APP-6.4** Quản lý: xem trạng thái vào/ra của nhân viên theo phạm vi từ dữ liệu thật.
- [ ] **APP-6.5** Chống bấm lặp: khóa nút trong lúc chờ và dùng idempotency key khi retry.

**Đầu vào:** `PLAN-BE.md` BE-6.1 → BE-6.4; quy tắc cửa sổ vào/ra và GPS trong `BE/docs/DATA-RULES.md` phải chốt trước.
**Đầu ra:** trang chủ **Chấm công** thật (thay `UnavailableCard`); quyền vị trí trong `Info.plist`; widget test cho các nhánh lỗi.
**Nghiệm thu:** vào/ra end-to-end trên Simulator với vị trí kiểm soát; không sinh bản ghi trùng khi bấm lặp hoặc retry; mọi nhánh lỗi có thông báo hướng dẫn cụ thể; GPS thiết bị thật ghi riêng là chưa xác minh.

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

**Đầu vào:** `PLAN-BE.md` BE-8.1 → BE-8.4; quyết định về loại nghỉ và tác động công.
**Đầu ra:** hai tab thay thế `UnavailablePage`; widget test cho không-tự-duyệt.
**Nghiệm thu:** nhân viên theo dõi được yêu cầu của mình; quản lý duyệt trong phạm vi; từ chối hiển thị lý do; thao tác lặp không tạo hai lần xử lý.

## APP-8A — Checklist công việc

- [ ] **APP-8A.1** Tab **Công việc**: danh sách việc được giao, tiến độ, hạn, đánh dấu từng mục và ghi chú.
- [ ] **APP-8A.2** Trang chủ nhân viên hiển thị việc sắp đến hạn.
- [ ] **APP-8A.3** Quản lý: tạo mẫu, giao việc, theo dõi tiến độ và việc quá hạn.
- [ ] **APP-8A.4** Sau khi hoàn tất toàn bộ, không cho tự bỏ đánh dấu; yêu cầu quản lý mở lại.

**Đầu vào:** `PLAN-BE.md` BE-8A.1 → BE-8A.5 (chưa có schema).
**Đầu ra:** tab **Công việc**, màn giao việc trong shell quản lý.
**Nghiệm thu:** quản lý giao → nhân viên thực hiện → quản lý thấy tiến độ thật; hoàn thành checklist không làm thay đổi giờ công; cập nhật lặp không sinh kết quả trùng.

## APP-9 — Tổng quan và hoàn thiện

- [ ] **APP-9.1** Tab **Tổng quan** cho quản lý: nhân sự theo lịch, đã vào ca, thiếu công, yêu cầu chờ duyệt, checklist quá hạn — từ API nguồn thật.
- [ ] **APP-9.2** Hoàn thiện trạng thái tải/rỗng/lỗi/mất phiên và điều hướng trên cả hai shell.
- [ ] **APP-9.3** Nghiệm thu trên iOS Simulator; ghi riêng GPS và iPhone vật lý chưa xác minh.
- [ ] **APP-9.4** Review lại toàn bộ: màn 320px, chữ lớn, xoay màn hình, mất mạng giữa luồng.

**Đầu vào:** tất cả module nguồn (5, 6, 7, 8, 8A) đã có API thật.
**Đầu ra:** báo cáo nghiệm thu theo từng shell; cập nhật `App/README.md`.
**Nghiệm thu:** chủ tạo nhân viên → phân ca → nhân viên chấm công → gửi yêu cầu → quản lý duyệt → xem bảng công, toàn bộ trên app thật, không dùng dữ liệu giả. Android vẫn hoãn.
