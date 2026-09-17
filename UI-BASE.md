# UI-BASE — nền giao diện dùng chung của App

**Đọc file này trước khi tạo hoặc sửa màn hình trong `App/`.** Mục đích: mọi màn hình dùng chung một nền, không mỗi lần lại dựng thanh điều hướng, bảng màu, thẻ hay trạng thái rỗng mới.

Thiết kế gốc: `docs/design/trang-chu/` (`DESIGN.md` cho token, `screen.png` cho hình).

## Base nằm ở đâu

| File | Cung cấp |
| --- | --- |
| `App/lib/theme/app_colors.dart` | Bảng màu, bán kính, viền và bóng của thẻ |
| `App/lib/theme/app_nav_bar.dart` | `TimeInAppBar` (thanh điều hướng), `TimeInAction`, `initialsOf` |
| `App/lib/workspace/workspace_shell.dart` | `UnavailablePage`, `UnavailableCard`, hai shell và thanh tab |
| `App/lib/home/home_page.dart` | Màn mẫu đầy đủ: dùng đúng base, có dữ liệu mẫu, có nhãn |

## Quy tắc bắt buộc

1. **Không tạo `AppBar` mới.** Dùng `TimeInAppBar`. Màn đẩy thì truyền `onBack`, màn gốc trong shell thì không.
2. **Không viết màu trực tiếp.** Dùng `AppColors`; không dùng `Color(0x...)` trong màn hình.
3. **Ảnh đại diện người dùng dùng `initialsOf(name)`.** BE chưa có trường ảnh nên không hiển thị ảnh thật.
4. **Không dựng màn "chưa sẵn sàng" mới.** Dùng `UnavailablePage` (màn riêng) hoặc `UnavailableCard` (khối trong màn).
5. **Mọi màn phải có đủ bốn trạng thái:** đang tải, lỗi kèm nút thử lại, rỗng, có dữ liệu.
6. **Không nút chết.** Chức năng chưa có API thì hiển thị không bấm được và có tooltip nói rõ, hoặc ẩn hẳn.
7. **Không dùng `Timer.periodic` hay animation lặp** trong widget có test — `pumpAndSettle` sẽ không bao giờ kết thúc và test treo.
8. **Thông báo lỗi bọc trong `Semantics(liveRegion: true)`.**
9. **Không thêm dependency mới** khi chưa được xác nhận.

## `TimeInAppBar`

Bố cục theo thiết kế: logo thương hiệu (kèm nút quay lại nếu là màn đẩy), tiêu đề đậm, rồi chuông và ảnh đại diện ở bên phải. Thao tác phụ nằm trong menu của ảnh đại diện, không nhồi icon lên thanh.

| Tham số | Kiểu | Ghi chú |
| --- | --- | --- |
| `title` | `String` | Bắt buộc |
| `onBack` | `VoidCallback?` | Có giá trị thì hiện nút quay lại (tooltip "Quay lại") |
| `initials` | `String?` | Chữ trên ảnh đại diện; truyền `initialsOf(tên)` |
| `account` | `List<TimeInAction>` | Menu tài khoản; rỗng thì ảnh đại diện không bấm được |
| `extraActions` | `List<Widget>` | Nút phụ hiện thẳng trên thanh |
| `onBell` | `VoidCallback?` | `null` thì chuông chỉ hiển thị, chưa bấm được |

`TimeInAction(label:, icon:, onSelected:)` — `onSelected: null` thì mục bị vô hiệu hoá, dùng khi đang đăng xuất.

```dart
Scaffold(
  appBar: TimeInAppBar(
    title: 'Bảng công',
    onBack: () => Navigator.of(context).pop(),
    initials: initialsOf(member['user']['displayName']),
    account: [
      TimeInAction(
        label: 'Đăng xuất',
        icon: Icons.logout,
        onSelected: data.onLogout,
      ),
    ],
  ),
  body: ...,
);
```

## Bảng màu

| Tên | Dùng cho |
| --- | --- |
| `AppColors.primary` `#006E2F` | Chữ nhấn, icon đang chọn |
| `AppColors.primaryContainer` `#22C55E` | Nút hành động chính, chấm trạng thái đang chạy |
| `AppColors.secondaryContainer` `#FD761A` | Việc sắp bắt đầu, cảnh báo nhẹ |
| `AppColors.surface` `#F8F9FF` | Nền canvas |
| `AppColors.surfaceContainerLowest` `#FFFFFF` | Thẻ |
| `AppColors.surfaceContainerLow` → `Highest` | Nền phụ, từ nhạt tới đậm |
| `AppColors.onSurface` `#0B1C30` | Chữ chính |
| `AppColors.onSurfaceVariant` `#3D4A3D` | Chữ phụ, nhãn |
| `AppColors.error` `#BA1A1A` | Lỗi |

Thẻ dùng chung: `AppColors.cardRadius` (16), `AppColors.cardBorder`, `AppColors.cardShadow`.

## Checklist trước khi xong một màn mới

- [ ] Dùng `TimeInAppBar`, không tự dựng `AppBar`.
- [ ] Không có màu viết trực tiếp; chỉ dùng `AppColors`.
- [ ] Đủ bốn trạng thái: tải, lỗi + thử lại, rỗng, có dữ liệu.
- [ ] Chức năng chưa có API: dùng `UnavailablePage`/`UnavailableCard`, có nhãn dữ liệu mẫu nếu đang dùng dữ liệu mẫu.
- [ ] Không có nút không có tác dụng.
- [ ] Chạy được ở màn 320px và khi người dùng bật cỡ chữ lớn.
- [ ] `flutter analyze` sạch, `flutter test` đạt.
- [ ] Cập nhật mục Nghiệm thu của feature tương ứng trong `PLAN-APP.md`.

## Khi phải sửa base

Sửa ở đúng file trong bảng **Base nằm ở đâu**, không sửa bản sao ở màn hình. Sau khi sửa, chạy `flutter test` — các test workspace phụ thuộc vào khoá `tab-0`…`tab-4` và tooltip **"Quay lại"**, nên đổi tên khoá hay tooltip phải cập nhật test tương ứng.
