# PLAN-BE — Backend TimeIn (NestJS + Prisma)

Kế hoạch chi tiết cho repo [`TimeInBE`](https://github.com/Vuducdat1997/TimeInBE). Điều hướng chung và ma trận tiến độ: [PLAN.md](PLAN.md). Quy ước code của repo: `BE/AGENTS.md`.

Tài liệu này **chỉ quản lý phần BE**. Việc của App và Web nằm ở [PLAN-APP.md](PLAN-APP.md) và [PLAN-WEB.md](PLAN-WEB.md); phần BE chỉ cam kết hợp đồng API và trạng thái dữ liệu.

## Trạng thái: `[x]` xong và có bằng chứng chạy thật · `[ ]` còn việc

## Yêu cầu đầu vào của phần BE

BE chỉ bắt đầu một feature khi các đầu vào dưới đây đã chốt:

| Nguồn | Cần gì |
| --- | --- |
| `docs/PRODUCT.md` | Mô hình nghiệp vụ và ma trận quyền của feature |
| `BE/docs/DATA-RULES.md` | Quy tắc dữ liệu đã thống nhất (nghỉ, đi muộn, cửa sổ vào/ra, GPS) |
| `docs/PLAN-WEB.md`, `docs/PLAN-APP.md` | Kiểu dữ liệu và luồng màn hình mà client cần, để chốt contract trước khi code |
| Người quyết định | Các mục trong bảng **Quyết định còn mở** của `PLAN.md` liên quan tới feature |

Không viết API khi quy tắc nghiệp vụ còn mơ hồ — ghi lại quyết định trước, rồi mới code.

## Bàn giao của phần BE

| Đầu ra | Nơi nhận |
| --- | --- |
| Endpoint + Swagger tại `/docs` | `PLAN-APP.md`, `PLAN-WEB.md` |
| Migration đã áp dụng | Toàn dự án |
| Script verify (`db:verify`, `auth:verify`, `management:verify`) | Bằng chứng nghiệm thu |
| Cập nhật `BE/docs/*.md` | Tài liệu dự án |

---

## BE-1 — Môi trường và bộ khung

- [x] **BE-1.1** Khởi tạo NestJS + TypeScript trong `BE/`, giữ dữ liệu hiện có.
- [x] **BE-1.2** PostgreSQL 17 bằng Docker Compose, volume lưu dữ liệu, chỉ mở loopback.
- [x] **BE-1.3** Cấu hình Prisma và kết nối database.
- [x] **BE-1.4** `.env.example` đầy đủ (`DATABASE_URL`, `PORT`, `HOST`, `CORS_ORIGINS`); `.env` không vào Git.
- [x] **BE-1.5** API `/health` kiểm tra database, không lộ chi tiết kết nối (200 khi sẵn sàng, 503 khi không).
- [x] **BE-1.6** README hướng dẫn chạy, dừng và xử lý lỗi thường gặp.

**Đầu vào:** quyết định dùng PostgreSQL local, ưu tiên chạy trên máy không deploy public.
**Đầu ra:** `src/main.ts`, `src/database.ts`, `compose.yaml`, `prisma/`, `README.md`.
**Nghiệm thu:** `npm run db:up` → healthy; `npm run db:deploy` tạo được schema; `curl /health` trả `{"status":"ok","database":"ready"}`; `npm run build` sạch lỗi.

## BE-2 — Dữ liệu và nghiệp vụ nền

- [x] **BE-2.1** ERD: users, sessions, organizations, memberships, branches, work_locations.
- [x] **BE-2.2** Schema ca làm: shift_templates, shift_assignments, attendance_policies.
- [x] **BE-2.3** Schema chấm công: attendance_events, attendance_sessions.
- [x] **BE-2.4** Schema yêu cầu: leave_requests, correction_requests, timesheet_periods, audit_logs.
- [x] **BE-2.5** Khóa ngoại ghép, chỉ mục, ràng buộc chống trùng và phạm vi công ty.
- [x] **BE-2.6** Lưu UTC; ngày công tính theo múi giờ địa điểm.
- [x] **BE-2.7** Snapshot giờ/policy/địa điểm cho ca được phân; hỗ trợ ca qua đêm.
- [x] **BE-2.8** Tách dữ liệu chấm công gốc khỏi kết quả tính công và điều chỉnh.
- [x] **BE-2.9** Ghi quy tắc giờ nghỉ, đi muộn, về sớm, thiếu công và ca chồng lấn.
- [x] **BE-2.10** Migration + seed local chạy lại an toàn, không ghi đè dữ liệu người dùng.

**Đầu vào:** mô hình **Công ty/cửa hàng → Chi nhánh → Nhân viên** đã chốt trong `PRODUCT.md`.
**Đầu ra:** `prisma/schema.prisma`, 2 migration (`202609140001_initial`, `202609140002_auth_access`), `scripts/seed.cjs`.
**Nghiệm thu:** `npm run db:deploy` + `npm run db:seed` chạy hai lần không nhân bản; `npm run db:verify` đạt 12 tình huống (FK công ty, ca chồng lấn, kỳ/ngày, unique event, bất biến, yêu cầu điều chỉnh); CHECK/EXCLUDE/trigger hoạt động.

## BE-3 — Đăng nhập và phân quyền

- [x] **BE-3.1** Đăng nhập email/mật khẩu, mật khẩu băm scrypt với salt riêng.
- [x] **BE-3.2** Access token 15 phút, refresh tối đa 7 ngày, thu hồi phiên, đăng xuất.
- [x] **BE-3.3** `/auth/me` trả hồ sơ và membership ACTIVE, không trả password hash.
- [x] **BE-3.4** Quyền OWNER/MANAGER/EMPLOYEE theo membership, phạm vi chi nhánh.
- [x] **BE-3.5** `/organizations/:id/access` trả role, scope và permissions.
- [x] **BE-3.6** Chống tự nâng quyền và cách ly dữ liệu giữa hai công ty.
- [ ] **BE-3.7** API đổi mật khẩu cho chính người dùng: yêu cầu mật khẩu hiện tại, bắt buộc mật khẩu mới từ 12 ký tự, thu hồi các phiên khác sau khi đổi.

**Đầu vào:** quyết định token opaque (không JWT), giới hạn đăng nhập 15 yêu cầu/phút theo IP và email.
**Đầu ra:** `src/auth.ts`, `scripts/setup-demo-auth.cjs`, `scripts/verify-auth.cjs`, `BE/docs/AUTH.md`.
**Nghiệm thu:** `DEMO_PASSWORD=... npm run auth:verify` đạt: 401/403, chống role injection, khác công ty, giới hạn chi nhánh, membership inactive, access hết hạn, refresh đồng thời chỉ một request thành công, logout thu hồi cả hai token.

## BE-4 — Công ty, chi nhánh, nhân sự

- [x] **BE-4.1** Quản lý công ty (xem, đổi tên), chi nhánh và địa điểm chấm công.
- [x] **BE-4.2** Cấu hình tọa độ, bán kính và múi giờ địa điểm.
- [x] **BE-4.3** Thêm nhân viên, gán chi nhánh, đổi vai trò, vô hiệu hóa tài khoản.
- [x] **BE-4.4** Khóa hàng `Organization` trong transaction khi ghi, kiểm tra quyền lại, ghi audit cùng transaction.
- [x] **BE-4.5** Chặn tự đổi vai trò/vô hiệu hóa chính mình và chặn mất chủ công ty cuối cùng.
- [ ] **BE-4.6** Rà soát lại ràng buộc nghiệp vụ khi mở phần 5: ca của nhân viên bị vô hiệu hóa phải xử lý thế nào.
- [ ] **BE-4.7** Chức danh nhân viên (ví dụ "Nhân viên bán hàng"): bổ sung vào membership, cho OWNER/MANAGER đặt khi tạo/sửa, và trả về trong hồ sơ để tab Cá Nhân hiển thị. **Cần migration.**

**Đầu vào:** `BE/docs/MANAGEMENT.md`; yêu cầu form của App và Web (trường nào bắt buộc, giới hạn ký tự).
**Đầu ra:** endpoint `/organizations/:org`, `profile`, `branches`, `locations`, `employees`; `scripts/verify-management.cjs`; `scripts/cleanup-management-ui.cjs`.
**Nghiệm thu:** `npm run management:verify` đạt; EMPLOYEE nhận 403 ở mọi API quản lý; MANAGER chỉ tác động được nhân viên thường và địa điểm trong chi nhánh; mọi mutation sinh một dòng `AuditLog` có before/after; không ghi mật khẩu vào audit.

## BE-4A — Hợp đồng membership cho App

- [x] **BE-4A.1** Rà soát contract membership ACTIVE, vai trò và phạm vi chi nhánh để client chọn đúng ngữ cảnh.
- [x] **BE-4A.2** `assertEmployeeAssignment`: EMPLOYEE thuộc nhiều công ty hoạt động bị chặn (fail closed).

**Đầu vào:** yêu cầu từ `PLAN-APP.md` về việc App cần biết gì để chọn giao diện.
**Đầu ra:** `/auth/me` và `/organizations/:id/access` đủ dữ liệu; không cần API mới.
**Nghiệm thu:** `npm run verify:employee-assignment` đạt; EMPLOYEE nhiều company nhận 403 với thông báo yêu cầu liên hệ quản lý.

## BE-5 — Ca làm, giờ làm và lịch

- [ ] **BE-5.1** API mẫu ca: giờ bắt đầu/kết thúc (phút từ 00:00), cờ qua đêm, phút nghỉ, địa điểm, múi giờ.
- [ ] **BE-5.2** Phân ca theo nhân viên/ngày; kiểm tra cùng đơn vị/chi nhánh, trạng thái nhân viên, ca chồng lấn.
- [ ] **BE-5.3** Snapshot giờ/policy/địa điểm khi phân ca; sửa mẫu không đổi lịch sử.
- [ ] **BE-5.4** Chốt và thực thi quy tắc sửa/hủy ca đã có công.
- [ ] **BE-5.5** API lịch theo tuần cho quản lý (lọc chi nhánh/nhân viên) và lịch cá nhân cho nhân viên.

**Đầu vào:** quyết định còn mở về sửa/hủy ca đã chấm công; yêu cầu hiển thị lịch của `PLAN-WEB.md` và `PLAN-APP.md`.
**Đầu ra:** endpoint mẫu ca, phân ca, lịch tuần; migration mới nếu schema cần bổ sung.
**Nghiệm thu:** ca thường, ca qua đêm, ca chồng lấn bị từ chối rõ ràng; truy cập lịch ngoài phạm vi trả 403; sửa mẫu ca không làm đổi ca đã phân; `npm run db:verify` mở rộng thêm tình huống ca.

## BE-6 — Chấm công GPS

- [ ] **BE-6.1** Check-in/check-out theo ca dùng giờ server; kiểm tra thứ tự vào/ra, cửa sổ cho phép và quyền cá nhân.
- [ ] **BE-6.2** Kiểm tra khoảng cách tới địa điểm, độ mới và độ chính xác của vị trí.
- [ ] **BE-6.3** Chống ghi trùng khi retry hoặc request đồng thời bằng idempotency key.
- [ ] **BE-6.4** Tính `AttendanceSession` từ sự kiện gốc; giữ null khi thiếu đầu vào.
- [ ] **BE-6.5** Đăng ký mẫu khuôn mặt cho nhân viên; cập nhật và vô hiệu hóa mẫu cũ có audit. **Cần chốt phạm vi** — dữ liệu sinh trắc học cần phương án lưu trữ, thời hạn và quyền xóa riêng trước khi code.
- [ ] **BE-6.6** Xác thực khuôn mặt khi vào/ra ca: so khớp với mẫu đã đăng ký, lưu kết quả kèm điểm tin cậy, chống giả mạo cơ bản. **Cần chốt phạm vi.**

**Đầu vào:** quy tắc GPS trong `DATA-RULES.md` (bán kính 150 m, chính xác ≤ 100 m, vị trí không quá 60 giây) phải được rà soát và chốt lại trước khi code. BE-6.5/BE-6.6 cần thêm quyết định ở `PLAN.md` về phạm vi nhận diện khuôn mặt và lưu trữ dữ liệu sinh trắc học.
**Đầu ra:** endpoint chấm công; `AttendanceEvent` chỉ thêm, `AttendanceSession` tính lại được; bảng mẫu khuôn mặt và bảng kết quả xác thực.
**Nghiệm thu:** vào/ra end-to-end không sinh bản ghi trùng; retry cùng idempotency key trả kết quả cũ thay vì tạo mới; request đồng thời chỉ một thành công; bấm lặp bị chặn; ca đêm và thiếu giờ ra được ghi đúng; thời gian do điện thoại gửi không được tin. Với khuôn mặt: không lưu ảnh thô ngoài mẫu cần thiết, mọi lần đăng ký/cập nhật/xóa đều có audit, và từ chối xác thực không chặn nhân viên gửi yêu cầu bổ sung công.

## BE-7 — Lịch sử chấm công và bảng công

- [ ] **BE-7.1** Lịch sử theo khoảng ngày có phân trang; nhân viên chỉ thấy bản thân, quản lý theo phạm vi.
- [ ] **BE-7.2** Tính giờ làm, phút muộn/về sớm, thiếu vào/ra và tổng hợp tháng theo múi giờ.
- [ ] **BE-7.3** API bảng công cho quản lý, lọc nhân viên/chi nhánh/tháng, mở chi tiết từng ca.
- [ ] **BE-7.4** API thống kê cá nhân theo tháng cho tab Cá Nhân của App: số ca đã làm, tổng giờ, số ngày nghỉ.

**Đầu vào:** quyết định làm tròn giờ và quy đổi ngày công (mục còn mở của `PLAN.md`); công thức trong `DATA-RULES.md`.
**Đầu ra:** endpoint lịch sử và bảng công; một công thức tính công dùng chung cho App và Web.
**Nghiệm thu:** tổng tháng khớp tổng chi tiết từng ca; ca đêm và ca qua tháng tính đúng theo ngày địa phương bắt đầu ca; dữ liệu thiếu không bị coi là đủ công; hai client dùng cùng kết quả.

## BE-8 — Nghỉ, sửa công và phê duyệt

- [ ] **BE-8.1** Chốt đơn vị nghỉ (giờ/nửa ngày/ngày), tác động lên lịch và công, giới hạn gửi/hủy.
- [ ] **BE-8.2** API gửi, xem trạng thái, duyệt/từ chối; kiểm tra người duyệt, không tự duyệt, không xử lý lặp.
- [ ] **BE-8.3** Giữ sự kiện công gốc; lưu lý do, người thao tác, giá trị trước/sau; tính lại kết quả theo quyết định.
- [ ] **BE-8.4** Chuyển trạng thái bằng cập nhật có điều kiện để chống duyệt đồng thời.
- [ ] **BE-8.5** Đổi ca: API gửi, xem và duyệt yêu cầu đổi ca giữa hai nhân viên cùng chi nhánh; kiểm tra ca chồng lấn, trạng thái nhân viên nhận và ca đã chấm công. **Cần chốt phạm vi.**
- [ ] **BE-8.6** Quy tắc thời hạn gửi đơn nghỉ (ví dụ phải gửi trước 24 giờ) áp dụng ở server, trả lỗi rõ ràng để App hiển thị.

**Đầu vào:** quyết định còn mở về loại nghỉ và tác động công; quyết định về đổi ca (tự thỏa thuận hay quản lý duyệt); quy tắc trong `DATA-RULES.md`.
**Đầu ra:** endpoint nghỉ và sửa công; cập nhật `AttendanceSession` khi duyệt, không sửa `AttendanceEvent`.
**Nghiệm thu:** từ chối không đổi công; duyệt có lịch sử đầy đủ; duyệt đồng thời chỉ xử lý một lần; không tự duyệt; yêu cầu của OWNER khi chỉ có một OWNER giữ ở trạng thái chờ.

## BE-8A — Checklist công việc

- [ ] **BE-8A.1** Chốt quy tắc hoàn tất/mở lại checklist theo đề xuất trong `PRODUCT.md`.
- [ ] **BE-8A.2** Migration mẫu checklist, mục việc, lần giao, kết quả từng mục và lịch sử.
- [ ] **BE-8A.3** Tạo/sửa mẫu, giao một nhân viên theo ca/ngày và hạn; snapshot nội dung khi giao.
- [ ] **BE-8A.4** Hoàn thành từng mục bằng giờ server; tiến độ/quá hạn; hủy/mở lại có audit.
- [ ] **BE-8A.5** Cập nhật lặp không sinh kết quả trùng.

**Đầu vào:** chưa có schema — cần thiết kế trước; quyết định về khóa khi hoàn tất và hạn giao việc.
**Đầu ra:** migration checklist, bảng mới có `organizationId`, `branchId`, assignee membership và audit.
**Nghiệm thu:** sửa mẫu không đổi việc đã giao; hủy/mở lại bảo toàn lịch sử; cập nhật đồng thời không sinh kết quả trùng; cách ly quyền theo đơn vị/chi nhánh; hoàn thành checklist không tự chấm công hay đổi giờ công.

## BE-9 — Báo cáo và chốt kỳ

- [ ] **BE-9.1** API tổng quan quản lý theo ngày/chi nhánh từ dữ liệu thật (ca, công, yêu cầu, checklist).
- [ ] **BE-9.2** Xuất CSV bảng công theo quyền, đối chiếu tổng và chi tiết.
- [ ] **BE-9.3** Chốt/mở lại kỳ theo quyền chủ, có audit, chặn điều chỉnh kỳ đã chốt.
- [ ] **BE-9.4** Thử backup/restore local.
- [ ] **BE-9.5** Review phân quyền, dữ liệu nhạy cảm và lỗi nghiệp vụ còn mở.
- [ ] **BE-9.6** Thông báo trong app: tạo thông báo khi đơn được duyệt/từ chối, khi được giao checklist mới và khi sắp đến giờ vào ca; API danh sách, đếm chưa đọc và đánh dấu đã đọc. **Cần chốt phạm vi.**
- [ ] **BE-9.7** Bảng lương và thu nhập theo tháng cho nhân viên. **Cần chốt phạm vi** — hiện nằm ngoài phạm vi dự án; cần công thức lương và quyền xem trước khi code.

**Đầu vào:** các module nguồn (7, 8, 8A) đã có dữ liệu thật; quyết định về thời hạn lưu dữ liệu và quy trình mở lại kỳ; quyết định về phạm vi thông báo và bảng lương.
**Đầu ra:** endpoint tổng quan, endpoint xuất CSV, luồng chốt/mở kỳ.
**Nghiệm thu:** chỉ số tổng quan lấy từ API nguồn thật, không suy diễn; kỳ CLOSED chặn chấm công, sửa ca và phê duyệt điều chỉnh; mở lại cần OWNER và sinh audit; CSV khớp với bảng công trên màn hình.
