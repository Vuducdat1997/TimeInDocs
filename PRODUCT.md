# TimeIn — Ý tưởng sản phẩm v2

Cập nhật: 2026-09-15. Đây là thiết kế đích theo yêu cầu mới, chưa phải toàn bộ tính năng đã chạy. Tiến độ thực tế xem [PLAN.md](PLAN.md).

## Mục tiêu

Một app Flutter cho iOS/Android, dùng một backend NestJS và PostgreSQL local. Sau đăng nhập, app tự đưa người dùng vào giao diện phù hợp với quyền tài khoản trong công ty đang chọn:

- Chủ công ty/cửa hàng: quản lý vận hành, nhân sự, lịch làm, giờ làm, phê duyệt, bảng công và checklist.
- Quản lý chi nhánh: giao diện quản lý tương tự, dữ liệu/thao tác giới hạn chi nhánh được giao.
- Nhân viên: chấm công, lịch sử công, lịch làm, đăng ký nghỉ, yêu cầu sửa công và công việc cần thực hiện.

Màn hiển thị danh sách quyền hiện nay được thay bằng trang chủ thực sự cho từng nhóm. Không cho người dùng tự chọn vai trò khi đăng nhập.

## Cấu trúc tổ chức — đã chốt

Người dùng đã chốt mô hình Công ty/cửa hàng → Chi nhánh → Nhân viên:

```text
Công ty / Cửa hàng (đơn vị kinh doanh, Organization)
├── Chi nhánh A
│   ├── Nhân viên được phân về chi nhánh
│   ├── Địa điểm chấm công
│   ├── Ca làm và lịch phân ca
│   └── Checklist công việc
└── Chi nhánh B
    └── Các nhóm dữ liệu tương tự
```

Ví dụ: Cửa hàng Cà phê A có chi nhánh Quận 1 và Quận 3. Một cửa hàng chỉ có một địa điểm vẫn được tổ chức với một chi nhánh chính. Địa điểm chấm công là tọa độ/bán kính trong chi nhánh, không phải một cấp chủ sở hữu mới.

Công ty và cửa hàng là hai cách gọi của cùng cấp Organization; không thêm tầng Store. Membership gắn tài khoản với đơn vị và quyền; nhân viên được phân về chi nhánh. Giữ schema tổ chức hiện có.

## Luồng đăng nhập và chọn giao diện

1. Đăng nhập email/mật khẩu.
2. Server trả các membership còn hoạt động cùng quyền của từng membership.
3. Chỉ có một đơn vị: vào thẳng giao diện theo vai trò; không bắt chọn công ty.
4. Có nhiều đơn vị: chọn đơn vị, sau đó vào giao diện đúng vai trò trong đơn vị đó.
5. Nhớ đơn vị chọn gần nhất trên thiết bị; mở lại vẫn xác minh quyền với server.
6. Không còn membership hoạt động: hiển thị hướng dẫn liên hệ quản lý, không mở dashboard.
7. Đổi đơn vị, thay quyền hoặc vô hiệu hóa: xóa dữ liệu hiển thị/cache của ngữ cảnh cũ và điều hướng lại. Backend kiểm tra quyền trên mỗi request.

Một tài khoản có thể là chủ tại công ty A và nhân viên tại B; vai trò không được dùng chung cho mọi công ty. Đổi giao diện là kết quả của quyền server, không phải thao tác nâng quyền của người dùng.

## Giao diện nhân viên

Thanh điều hướng đề xuất: **Chấm công · Lịch làm · Công việc · Yêu cầu · Cá nhân**.

| Màn hình | Nội dung và thao tác |
| --- | --- |
| Chấm công (trang chủ) | Ca hiện tại, chi nhánh, giờ vào/ra, nút chấm công chính, việc sắp đến hạn, liên kết lịch sử công |
| Xác nhận chấm công | Kiểm tra vị trí, xác nhận kết quả server, báo lỗi và thử lại |
| Lịch sử chấm công | Lọc ngày/tháng, giờ vào/ra, tổng giờ, đi muộn, thiếu công; chi tiết ca và gửi yêu cầu sửa |
| Lịch làm | Lịch tuần, ca hôm nay, giờ làm/nghỉ, chi nhánh; ca đêm ghi rõ ngày kết thúc |
| Công việc | Checklist được giao theo ca/ngày, tiến độ và hạn; đánh dấu từng mục, ghi chú |
| Yêu cầu | Đăng ký nghỉ, bổ sung/sửa công, xem chờ duyệt/đã duyệt/từ chối và lý do |
| Cá nhân | Hồ sơ, mã nhân viên, chi nhánh, đổi đơn vị khi có nhiều membership, đăng xuất |

Trạng thái trang chủ phải rõ: chưa phân ca, chưa vào ca, đang làm, đã ra ca, thiếu công, không có dữ liệu, mất mạng và lỗi quyền. Màn trống không hiển thị giờ làm hoặc tiến độ giả. Lịch sử công là chức năng riêng có thể mở trực tiếp từ trang chủ, dù không chiếm một tab riêng.

## Giao diện chủ công ty/cửa hàng và quản lý

Thanh điều hướng đề xuất: **Tổng quan · Nhân sự · Lịch làm · Phê duyệt · Thêm**.

| Màn hình | Nội dung và thao tác |
| --- | --- |
| Tổng quan | Chọn ngày/chi nhánh; nhân sự theo lịch, đã vào ca, thiếu công; yêu cầu chờ duyệt; checklist chưa xong/quá hạn; lối tắt bảng công và checklist |
| Nhân sự | Danh sách, tìm kiếm/lọc, hồ sơ, thêm nhân viên, phân vai trò/chi nhánh, vô hiệu hóa theo quyền |
| Lịch làm | Lịch theo tuần/chi nhánh/nhân viên; tạo mẫu ca, thiết lập giờ bắt đầu/kết thúc/nghỉ, phân ca, đổi/hủy ca hợp lệ |
| Phê duyệt | Nghỉ phép và sửa công; xem dữ liệu gốc/đề xuất, duyệt/từ chối có lý do, không tự duyệt |
| Bảng công | Theo nhân viên/chi nhánh/tháng, giờ làm, đi muộn, thiếu công, chi tiết, xuất báo cáo, chốt kỳ |
| Checklist | Tạo mẫu việc, giao theo nhân viên/ca/ngày, theo dõi hoàn thành/chưa làm/quá hạn |
| Cài đặt đơn vị | Công ty/cửa hàng, chi nhánh, địa điểm chấm công, quy tắc giờ làm, tài khoản |
| Thêm | Điểm truy cập Bảng công, Checklist, Chi nhánh, Địa điểm, Cài đặt và Cá nhân |

Chủ xem tổng hợp toàn đơn vị hoặc lọc chi nhánh. Quản lý chỉ thấy chi nhánh được giao; bộ lọc không cho mở rộng quyền. V1 tiếp tục một chi nhánh/membership quản lý; phân quản lý nhiều chi nhánh chỉ bổ sung nếu được yêu cầu.

Các chỉ số dashboard chỉ lấy từ API đã có nghiệp vụ tương ứng. Khi module chưa triển khai, hiển thị mô tả/chưa sẵn sàng thay vì bịa số liệu hoặc gắn nút không có tác dụng.

## Ma trận quyền đích

| Chức năng | OWNER | MANAGER | EMPLOYEE |
| --- | --- | --- | --- |
| Giao diện mặc định | Quản lý toàn đơn vị | Quản lý chi nhánh | Chấm công |
| Nhân sự | Toàn đơn vị | Nhân viên thường trong chi nhánh | Hồ sơ bản thân |
| Tạo/sửa chi nhánh | Có | Không | Không |
| Địa điểm chấm công | Toàn đơn vị | Chi nhánh mình | Xem địa điểm liên quan ca |
| Lịch/mẫu ca | Toàn đơn vị | Chi nhánh mình | Xem lịch được phân |
| Chấm công/lịch sử cá nhân | Không hiển thị mặc định | Không hiển thị mặc định | Bản thân |
| Bảng công nhân sự | Toàn đơn vị | Chi nhánh mình | Chỉ công bản thân |
| Gửi nghỉ/sửa công | Theo quyền làm việc cá nhân nếu bổ sung sau | Theo quyền làm việc cá nhân nếu bổ sung sau | Bản thân |
| Phê duyệt | Trong đơn vị, không tự duyệt | Nhân viên thường chi nhánh mình, không tự duyệt | Không |
| Tạo/giao checklist | Toàn đơn vị | Chi nhánh mình | Không |
| Thực hiện checklist | Theo dõi quản lý | Theo dõi quản lý | Việc được giao |
| Chốt/mở kỳ công | Có, có lịch sử | Xem | Không |

Chủ hoặc quản lý muốn đồng thời chấm công là lựa chọn sản phẩm còn mở; không tự thêm nút chuyển sang chế độ nhân viên để vượt mô hình quyền. Hiện ưu tiên hai trải nghiệm theo yêu cầu.

## Checklist công việc — chức năng mới

MVP đề xuất:

- Mẫu checklist gồm tên, mô tả và các mục có thứ tự; ví dụ mở cửa, kiểm tra thiết bị, vệ sinh, bàn giao cuối ca.
- Quản lý giao một checklist cho một nhân viên tại một chi nhánh, gắn ca hoặc ngày và hạn cụ thể.
- Mỗi lần giao lưu bản chụp nội dung mẫu. Sửa mẫu không thay đổi việc đã giao/lịch sử.
- Nhân viên chỉ đánh dấu công việc của mình; mỗi mục lưu người thực hiện và giờ server.
- Tiến độ checklist tính từ số mục hoàn thành; quá hạn là trạng thái suy ra từ hạn và phần việc còn thiếu.
- Không cho nhân viên bỏ đánh dấu sau khi đã hoàn tất toàn checklist; yêu cầu quản lý mở lại, có lý do/audit. Quy tắc này là mặc định đề xuất cần chốt trước phần 8A.
- Tạo/giao/cập nhật/hủy/mở lại cần kiểm tra quyền công ty, chi nhánh, người nhận và trạng thái tài khoản.
- Cập nhật lặp lại không tạo lần hoàn thành trùng. Hủy checklist giữ lịch sử.
- Hoàn thành checklist không tự chấm công hoặc thay đổi giờ công; thiếu checklist không tự trừ công/lương.
- Ảnh minh chứng, việc theo nhóm, tự lặp theo lịch và quy trình duyệt checklist nhiều cấp để sau MVP.

## Ảnh hưởng đến dữ liệu và code

| Phần | Hiện trạng | Thay đổi cần làm |
| --- | --- | --- |
| Tài khoản/phiên | Đã có đăng nhập, refresh, thu hồi, Keychain | Tận dụng; thêm khôi phục đơn vị gần nhất và điều hướng theo quyền |
| Tổ chức | Organization → Branch → WorkLocation, Membership | Giữ schema; thống nhất cách gọi Công ty/cửa hàng → Chi nhánh → Nhân viên |
| Flutter | Đang dùng chung màn quyền và các trang quản lý | Tách EmployeeShell và ManagementShell, route/tab/cache theo ngữ cảnh |
| Nhân sự | CRUD và giới hạn quyền đã có | Đưa vào giao diện quản lý; kiểm tra lại toàn bộ đường dẫn trực tiếp |
| Lịch/công/nghỉ | Schema có; API và giao diện còn thiếu | Triển khai song song trải nghiệm nhân viên và quản lý ở phần 5–8 |
| Checklist | Chưa có | Thiết kế ChecklistTemplate, ChecklistTemplateItem, ChecklistAssignment, ChecklistItemResult và lịch sử |
| Dashboard | Chưa có thống kê nghiệp vụ | Bổ sung API tổng hợp có phạm vi rõ, sau khi module nguồn có dữ liệu |

Bảng checklist cần organizationId, branchId, assignee membership, ca/ngày, hạn, bản chụp nội dung và audit. Tham chiếu ghép theo tổ chức; không tin ID nhân viên/chi nhánh do client gửi. API đích gồm tổng quan nhân viên/quản lý, lịch cá nhân/lịch nhân sự, lịch sử công cá nhân/bảng công quản lý và checklist theo người được giao. Tên endpoint cụ thể được chốt khi viết contract từng phần.

## Giới hạn giữ nguyên

Flutter + NestJS + PostgreSQL/Prisma, chạy local và ưu tiên iOS. Chưa deploy public, tính lương, nhận diện khuôn mặt hoặc SMS OTP. Thiết kế v2 không xóa dữ liệu hay tự thay đổi quyền các tài khoản đã có. Báo cáo và test đã đạt của v1 vẫn là bằng chứng cho v1, chưa chứng minh hai giao diện mới hoạt động.

## Quy tắc cấp tài khoản và công ty của nhân viên — cập nhật 2026-09-17

- Chỉ OWNER/MANAGER được tạo tài khoản nhân viên trong phạm vi quản lý; không có tự đăng ký.
- Công ty và chi nhánh do cấp quản lý gán khi tạo tài khoản. Nhân viên vào thẳng đơn vị được phân công, không có chọn/đổi công ty và không dùng lựa chọn công ty lưu trên thiết bị.
- OWNER/MANAGER chỉ chọn trong các membership quản lý được cấp. Quy tắc nhiều công ty trước đây không áp dụng cho EMPLOYEE.
- Nếu dữ liệu cũ gán EMPLOYEE vào nhiều công ty đang hoạt động, App yêu cầu liên hệ quản lý; BE chặn truy cập nghiệp vụ cho tới khi phân công được điều chỉnh. Không tự chọn công ty đầu tiên.
- BE đã chặn nhân viên tạo tài khoản hoặc tự sửa membership. MANAGER chỉ tạo/sửa nhân viên thường trong chi nhánh; OWNER quản lý trong công ty. Chuyển công ty giữa các đơn vị chưa có luồng quản trị riêng, cần thiết kế quyền nhận/chuyển; không cho nhân viên tự thực hiện.
