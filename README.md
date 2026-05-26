# SU_HDG

`SU_HDG` là plugin SketchUp cá nhân, hiện có năm chức năng: **Siêu hàn cạnh**, **Phá đường cong**, **Làm mịn cạnh**, **Trim**, và **Bo góc**.

Các chức năng này xử lý `Sketchup::Curve` từ selection hiện tại, tương tự nhóm lệnh weld/explode curve, nhưng code độc lập và không gọi TIG/CadFather/JHS hay plugin bên thứ ba.

## Cấu Trúc

- `su_hdg.rb`: loader extension ở root của RBZ.
- `su_hdg/main.rb`: điểm vào chính, đăng ký menu và toolbar.
- `su_hdg/commands/catalog.rb`: đăng ký command `hdg_weld_edges`, `hdg_explode_curves`, `hdg_smooth_edges`, `hdg_trim_solid`, và `hdg_round_corner`.
- `su_hdg/commands/weld_edges.rb`: hàn cạnh đã chọn thành curve.
- `su_hdg/commands/explode_curves.rb`: phá curve đã chọn thành edge rời.
- `su_hdg/commands/smooth_edges.rb`: làm mịn chuỗi cạnh đã chọn theo số segments.
- `su_hdg/commands/trim_solid.rb`: trim một solid bằng một solid khác và khôi phục thông tin đối tượng.
- `su_hdg/commands/trim_solid_tool.rb`: tool target trước, modifier sau cho Trim.
- `su_hdg/commands/round_corner.rb`: tính và áp dụng bo góc theo bán kính.
- `su_hdg/commands/round_corner_tool.rb`: tool hover/click cho Bo góc.
- `su_hdg/config.rb`: default toolbar có `hdg_weld_edges`, `hdg_explode_curves`, `hdg_smooth_edges`, `hdg_trim_solid`, và `hdg_round_corner`.
- `su_hdg/toolbar.rb`: dựng `UI::Toolbar` và `UI::Command`.
- `su_hdg/updater.rb`: kiểm tra, tải và cài bản cập nhật từ manifest JSON.
- `su_hdg/ui/config_dialog.rb`: màn hình cấu hình toolbar.
- `su_hdg/icons/`: icon riêng của plugin.
- `scripts/build_rbz.ps1`: đóng gói RBZ.
- `dist/SU_HDG.rbz`: file cài đặt sau khi build.

## Chức Năng

Toolbar mặc định có năm nút:

- `Siêu hàn cạnh`: nối các cạnh được kết nối thành 1 đường cong.
- `Phá đường cong`: phá tất cả các đường cong đã chọn.
- `Làm mịn cạnh`: hỏi `Segments:` rồi làm mịn các cạnh đã chọn.
- `Trim`: click target solid bị cắt rồi click modifier solid để cắt, hỗ trợ chọn sẵn nhiều target và trim lặp.
- `Bo góc`: rê chuột vào cạnh gần góc cần bo để xem preview, click nhập bán kính rồi cắt bo góc.

Quy tắc hàn cạnh:

- Cần chọn cạnh trước khi bấm icon.
- Nếu chưa chọn cạnh nào, bấm icon sẽ không có hiệu lực và không chuyển sang tool click cạnh.
- Khi đã chọn nhiều cạnh, bấm nút sẽ hàn đúng selection hiện tại.
- Edge đã thuộc curve vẫn được xử lý: plugin tách curve cũ và dựng lại curve mới từ các đoạn đã chọn.
- Không xóa cạnh gốc thủ công; plugin tạo curve qua group tạm rồi explode để SketchUp merge lại geometry, giảm rủi ro phá mặt.
- Tự tách nhiều chuỗi cạnh rời nhau.
- Hàn open chain, closed loop, và tách chuỗi tại điểm nhánh.
- Sau khi hàn, chọn các edge thuộc curve vừa tạo.

Quy tắc phá đường cong:

- Cần chọn curve hoặc edge đang thuộc curve trước khi bấm icon.
- Nếu chọn edge rời không thuộc curve, plugin bỏ qua edge đó.
- Với mỗi curve hợp lệ, plugin gọi `explode_curve` để rã curve thành các cạnh rời.
- Sau khi phá, chọn lại các edge vừa rã.

Quy tắc làm mịn cạnh:

- Cần chọn cạnh hoặc curve trước khi bấm icon.
- Khi bấm nút, plugin hỏi `Segments:`; số này là số đoạn mục tiêu cho mỗi chuỗi cạnh.
- Plugin tự tách nhiều chuỗi cạnh rời nhau và làm mịn từng chuỗi.
- Plugin chỉ nhận diện cạnh bo cong; cạnh thẳng rời hoặc curve thẳng/collinear sẽ bị bỏ qua.
- Circle/arc của SketchUp được dựng lại bằng `add_circle`/`add_arc` theo đúng số `Segments`.
- Cạnh đang nằm trên mặt vẫn được xử lý; plugin tạo cạnh mới trước, xóa cạnh cũ sau, rồi gọi `find_faces` để SketchUp tự vá mặt khi có thể.
- Sau khi làm mịn, chọn lại các edge mới.

Quy tắc Trim:

- Bấm icon để vào tool Trim; nếu đang chọn sẵn nhiều solid thì các solid đó là target bị cắt.
- Nếu chưa chọn sẵn, click vật 1 là `target`/vật bị cắt; click vật 2 là `modifier`/vật cắt. Click modifier xong plugin tự cắt ngay, không hỏi thêm.
- Tool hiển thị nhãn `1`/`2` tại vị trí con trỏ chuột để tránh nhầm thứ tự.
- Plugin giữ modifier, thay từng target bằng kết quả trim, rồi chọn lại các result để có thể trim tiếp bằng modifier khác.
- Kết quả khôi phục name, tag, material, shadow flags và attributes từ target bị cắt.
- Mặt/cạnh của kết quả không lấy màu của modifier; nếu SketchUp gán nhầm material từ modifier thì plugin đổi lại theo target hoặc trả cạnh về trống như ban đầu.
- Kết quả không để tên mặc định `Difference`; nếu vật gốc chưa có tên definition thì dùng `SU_HDG Trim`.

Quy tắc Bo góc:

- Bấm icon để vào tool Bo góc. Tool có thể bắt cạnh trong context đang mở hoặc trong group/component đang hover.
- Rê chuột vào cạnh gần góc cần bo; tool quét các cặp cạnh quanh đỉnh để tránh lấy nhầm cạnh độ dày, rồi hiển thị preview trong suốt theo màu mặt gốc với viền cam.
- Gõ bán kính rồi Enter trực tiếp, hoặc click vào góc/cạnh preview để mở ô nhập `Bán kính bo góc:`; Enter trong ô nhập sẽ xác nhận như OK.
- Tool có 3 mode: `Bo tròn`, `Vát góc`, `Khoét tròn`. Bấm Tab để xoay mode hoặc 1/2/3 để chọn nhanh; hướng dẫn được vẽ bên trái viewport khi tool đang chạy.
- Plugin tạo cung bo theo bán kính nhập, tạo `Sketchup::ArcCurve` bán kính trước rồi rebuild mặt lớn và hai mặt cạnh thẳng bằng loop bo/tangent để cắt thật khối góc, không tạo curve rời nằm chồng lên mặt, giữ Entity Info có `Radius`, giữ viền cung trên/dưới nhìn thấy, và chỉ ẩn/làm mịn segment nội bộ của mặt cong.
- Nếu phát hiện cạnh độ dày của tấm ván, plugin tạo cung ở mặt sau và các mặt nối để bo xuyên tấm ván; nếu không, plugin chỉ bo trên mặt đang hover.

## Cài Vào SketchUp 2025

1. Mở SketchUp 2025.
2. Vào `Window > Extension Manager`.
3. Chọn `Install Extension`.
4. Chọn file `dist\SU_HDG.rbz`.
5. Sau khi cài, vào `Extensions > SU_HDG > Show Toolbar`.

## Build RBZ

Mở PowerShell tại thư mục source và chạy:

```powershell
.\scripts\build_rbz.ps1
```

File cài đặt sẽ nằm tại:

```text
dist\SU_HDG.rbz
```

## Auto Update

Auto update đọc manifest public ở repo release. Manifest hiện dùng dạng:

```json
{
  "id": "SU_HDG",
  "version": "1.5.32",
  "rbz_url": "https://raw.githubusercontent.com/BLACK-BOT25/SRC_PLUGIN_HDG_RELEASE/refs/heads/main/SU_HDG.rbz",
  "notes": "Auto update có sẵn manifest public mặc định để user mới tự check khi mở SketchUp."
}
```

## Lưu Ý Toolbar

SketchUp Ruby API không có lệnh xóa item khỏi `UI::Toolbar` cũ. Plugin dựng toolbar runtime mới khi Save/Reset cấu hình và chỉ hide các toolbar object đang giữ trong session hiện tại. Không tạo toolbar theo tên chỉ để hide, vì SketchUp có thể sinh toolbar rỗng nếu tên đó chưa tồn tại trong session. Toolbar runtime dùng suffix ẩn để tránh nhân item, nhưng tiêu đề nhìn thấy chỉ là `SU_HDG`. Icon thường để nền trong suốt để không che hover/checked native của SketchUp; icon tool trả `MF_CHECKED` và swap sang icon `_active.svg` nền xanh khi đang chạy.

URL manifest để nhập trong SketchUp:

```text
https://raw.githubusercontent.com/BLACK-BOT25/SRC_PLUGIN_HDG_RELEASE/refs/heads/main/update-manifest.json
```

Trong SketchUp, vào `Extensions > SU_HDG > Update Settings`, nhập Manifest URL, để `Tự check khi mở SketchUp` là `yes`, và để `Tự cài update` là `yes`.

## Repo

- Source private: `BLACK-BOT25/SRC_PLUGIN_HDG`
- Release public: `BLACK-BOT25/SRC_PLUGIN_HDG_RELEASE`

Repo release chỉ chứa `SU_HDG.rbz` và `update-manifest.json`; không đưa source Ruby lên repo release.
