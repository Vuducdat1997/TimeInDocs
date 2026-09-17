# PLAN-WEB — Web quản trị TimeIn (React + TypeScript)

Kế hoạch chi tiết cho repo [`WebAdmin`](https://github.com/Vuducdat1997/WebAdmin). Điều hướng chung và ma trận tiến độ: [PLAN.md](PLAN.md). Hướng dẫn chạy và deploy: `WebAdmin/README.md`.

Tài liệu này **chỉ quản lý phần Web**. Web là giao diện vận hành trên máy tính cho quản lý và chủ đơn vị; Web **không** có chức năng chấm công cá nhân vì đó là việc của App.

## Trạng thái: `[x]` xong và có bằng chứng chạy thật · `[ ]` còn việc

## Yêu cầu đầu vào của phần Web

| Đầu vào | Từ đâu | Dùng cho |
| --- | --- | --- |
| Endpoint + kiểu dữ liệu | `PLAN-BE.md` | Mọi màn dữ liệu |
| Ngữ nghĩa 401/403 | `BE/docs/AUTH.md` | Gia hạn phiên và điều hướng khi mất quyền |
| Ma trận quyền | `docs/PRODUCT.md` | Ẩn/hiện nút theo vai trò |
| Kiểu hiển thị và đơn vị (phút, giờ, ngày) | `BE/docs/DATA-RULES.md` | Không tự quy đổi ở client |

**Nguyên tắc bắt buộc:** chỉ số trên dashboard lấy từ API đã có nghiệp vụ; module chưa triển khai hiển thị mô tả "chưa triển khai" thay vì số liệu giả hoặc nút không có tác dụng.

## Bàn giao của phần Web

| Đầu ra | Nơi nhận |
| --- | --- |
| Màn hình vận hành cho OWNER/MANAGER | Người dùng cuối |
| Bộ test phiên (`tests/session.mjs`) | Bằng chứng nghiệm thu |
| Cấu hình `vercel.json`, `VITE_API_BASE_URL` | Triển khai |

---

## WEB-1 — Bộ khung và kết nối BE

- [x] **WEB-1.1** Khởi tạo React + TypeScript + Vite + Tailwind; chuẩn hóa dependency và lockfile.
- [x] **WEB-1.2** Thêm bước kiểm tra TypeScript vào build; sửa file địa điểm lỗi cú pháp.
- [x] **WEB-1.3** Proxy `/api` cho môi trường local; URL BE cấu hình bằng `VITE_API_BASE_URL`.
- [x] **WEB-1.4** CORS allowlist tùy chọn cho trường hợp Web gọi BE trực tiếp.
- [x] **WEB-1.5** Cấu hình `vercel.json` (cài devDependencies, build vào `dist`, hỗ trợ route SPA).

**Đầu vào:** `PLAN-BE.md` BE-1.x; quyết định BE chạy local tại `127.0.0.1:3000`.
**Đầu ra:** `vite.config.ts`, `tsconfig.json`, `vercel.json`, `package.json`.
**Nghiệm thu:** `npm run build` sạch lỗi TypeScript; proxy `/api` gọi được BE local; `vercel.json` mở trực tiếp được các đường dẫn SPA.

## WEB-3 — Đăng nhập và phiên

- [x] **WEB-3.1** Màn đăng nhập, bảo vệ route bằng `ProtectedRoute`.
- [x] **WEB-3.2** Khôi phục và gia hạn phiên; gộp nhiều request 401 vào một lần refresh; xóa cache khi mất phiên.
- [x] **WEB-3.3** Đăng xuất thu hồi token trên BE và xóa phiên trên trình duyệt.
- [x] **WEB-3.4** Xử lý 403 bằng sự kiện `timein:access` để tải lại ngữ cảnh quyền.
- [x] **WEB-3.5** Phiên lưu trong `sessionStorage`, phân biệt theo thế hệ phiên để tránh ghi đè token cũ.

**Đầu vào:** `PLAN-BE.md` BE-3.1 → BE-3.3.
**Đầu ra:** `src/api/client.ts`, `src/auth/Auth.tsx`, `src/pages/LoginPage.tsx`, `tests/session.mjs`.
**Nghiệm thu:** 8 kiểm thử phiên đạt: refresh đồng thời, phản hồi 401 muộn, token hết hạn, lỗi mạng, refresh sau đăng xuất, quyền 403, sai mật khẩu, đăng xuất BE. Đăng nhập/đăng xuất thủ công trên trình duyệt local thành công.

## WEB-4 — Nhân sự, chi nhánh, địa điểm, cài đặt

- [x] **WEB-4.1** Chọn công ty từ membership quản lý; phân biệt OWNER/MANAGER/EMPLOYEE; cache theo công ty/vai trò/chi nhánh.
- [x] **WEB-4.2** Nhân sự: tìm kiếm, phân trang, lọc trạng thái, thêm/sửa trong phạm vi quyền, hiển thị lỗi BE.
- [x] **WEB-4.3** Chi nhánh: OWNER thêm/sửa, MANAGER chỉ xem.
- [x] **WEB-4.4** Địa điểm: thêm/sửa tọa độ, bán kính, múi giờ và trạng thái.
- [x] **WEB-4.5** Cài đặt công ty: OWNER đổi tên, MANAGER chỉ xem.
- [x] **WEB-4.6** Có đủ trạng thái loading/error/empty; modal dùng được bằng bàn phím.
- [ ] **WEB-4.7** Nghiệm thu tài khoản có nhiều công ty (luồng chọn và đổi công ty đầy đủ).
- [ ] **WEB-4.8** Nghiệm thu responsive ở nhiều kích thước màn hình.
- [ ] **WEB-4.9** Nghiệm thu truy cập từ thiết bị khác trong LAN.
- [ ] **WEB-4.10** Deploy public: cần BE HTTPS và cấu hình `VITE_API_BASE_URL`/`CORS_ORIGINS`; code đã ở GitHub, chưa deploy BE public hoặc Vercel.
- [ ] **WEB-4.11** Admin toàn hệ thống: chưa triển khai. Cần chốt quyền và phạm vi dữ liệu riêng trước khi xây, sau ưu tiên nhân sự/chấm công.

**Đầu vào:** `PLAN-BE.md` BE-4.1 → BE-4.5; `BE/docs/MANAGEMENT.md`.
**Đầu ra:** `src/pages/EmployeePage.tsx`, `BranchPage.tsx`, `LocationPage.tsx`, `SettingsPage.tsx`, `src/components/EmployeeModal.tsx`.
**Nghiệm thu:** OWNER đọc và lưu được nhân sự/địa điểm từ BE thật; MANAGER chỉ có nút sửa nhân viên thường, không có nút thêm/sửa chi nhánh; EMPLOYEE bị chặn khỏi giao diện quản trị; lỗi BE hiển thị nguyên văn cho người dùng. **Bằng chứng 2026-09-16:** Web/BE build đạt, 8 test phiên đạt, kiểm tra UI ba vai trò trên trình duyệt local.

## WEB-5 — Lịch làm và mẫu ca

- [ ] **WEB-5.1** Màn mẫu ca: danh sách, tạo/sửa giờ bắt đầu/kết thúc, cờ qua đêm, phút nghỉ, địa điểm.
- [ ] **WEB-5.2** Màn phân ca theo tuần, lọc chi nhánh/nhân viên; kéo/thêm ca cho nhân viên.
- [ ] **WEB-5.3** Đổi/hủy ca trong phạm vi quyền; hiển thị rõ ca đã có công hay chưa.
- [ ] **WEB-5.4** Cảnh báo xung đột ca trùng giờ ngay trên lịch.

**Đầu vào:** `PLAN-BE.md` BE-5.1 → BE-5.5; quyết định về sửa/hủy ca đã có công.
**Đầu ra:** các trang lịch làm và mẫu ca trong `src/pages/`.
**Nghiệm thu:** tạo mẫu ca → phân ca → lịch tuần hiển thị đúng; MANAGER chỉ thao tác trong chi nhánh mình; ca chồng lấn bị BE từ chối và hiển thị lỗi rõ ràng; sửa mẫu không đổi ca đã phân.

## WEB-7 — Bảng công

- [ ] **WEB-7.1** Bảng công theo nhân viên/chi nhánh/tháng: giờ làm, đi muộn, về sớm, thiếu công.
- [ ] **WEB-7.2** Mở chi tiết từng ca từ bảng công.
- [ ] **WEB-7.3** Đối chiếu tổng tháng với tổng chi tiết hiển thị trên cùng màn.

**Đầu vào:** `PLAN-BE.md` BE-7.1 → BE-7.3.
**Đầu ra:** trang bảng công trong shell quản lý.
**Nghiệm thu:** số liệu khớp API BE; lọc theo chi nhánh không mở rộng quá phạm vi quyền của MANAGER; dữ liệu thiếu hiển thị là thiếu.

## WEB-8 — Phê duyệt

- [ ] **WEB-8.1** Danh sách yêu cầu nghỉ và sửa công; lọc theo loại và trạng thái.
- [ ] **WEB-8.2** Xem dữ liệu gốc và đề xuất cạnh nhau trước khi quyết định.
- [ ] **WEB-8.3** Duyệt/từ chối kèm lý do; xác nhận trước khi gửi; không có nút duyệt cho yêu cầu của chính mình.

**Đầu vào:** `PLAN-BE.md` BE-8.1 → BE-8.4.
**Đầu ra:** trang phê duyệt.
**Nghiệm thu:** duyệt đúng phạm vi; từ chối không đổi công; duyệt đồng thời chỉ xử lý một lần và hiển thị lỗi phù hợp cho người thao tác sau.

## WEB-8A — Checklist công việc

- [ ] **WEB-8A.1** Tạo và sửa mẫu checklist gồm các mục có thứ tự.
- [ ] **WEB-8A.2** Giao việc cho một nhân viên theo ca/ngày kèm hạn.
- [ ] **WEB-8A.3** Bảng theo dõi tiến độ, lọc chi nhánh/nhân viên/ngày, làm nổi bật việc quá hạn.
- [ ] **WEB-8A.4** Hủy và mở lại checklist có lý do.

**Đầu vào:** `PLAN-BE.md` BE-8A.1 → BE-8A.5 (chưa có schema).
**Đầu ra:** các trang mẫu checklist, giao việc và theo dõi.
**Nghiệm thu:** quản lý giao việc → nhân viên thực hiện trên App → Web thấy tiến độ thật; sửa mẫu không đổi việc đã giao; hủy/mở lại giữ lịch sử.

## WEB-9 — Báo cáo và chốt kỳ

- [ ] **WEB-9.1** Dashboard theo ngày/chi nhánh: ca, công, yêu cầu chờ duyệt, checklist quá hạn — từ API nguồn thật.
- [ ] **WEB-9.2** Xuất CSV bảng công theo quyền; đối chiếu tổng và chi tiết trước khi xuất.
- [ ] **WEB-9.3** Chốt kỳ và mở lại kỳ (chỉ OWNER), có xác nhận và hiển thị audit.
- [ ] **WEB-9.4** Hoàn thiện trạng thái tải/rỗng/lỗi/mất quyền trên mọi trang.
- [ ] **WEB-9.5** Review lại phân quyền trên từng nút và từng route.

**Đầu vào:** `PLAN-BE.md` BE-9.1 → BE-9.3; module 7, 8, 8A đã có dữ liệu thật.
**Đầu ra:** dashboard, chức năng xuất CSV, luồng chốt/mở kỳ.
**Nghiệm thu:** chỉ số dashboard lấy từ API nguồn thật; CSV khớp bảng công trên màn hình; kỳ đã chốt chặn thao tác chỉnh sửa và thông báo rõ; mở lại kỳ sinh audit.
