# Quy tắc AI - SU_HDG

File này là luật làm việc cho AI khi sửa plugin SketchUp `SU_HDG`. Đọc file này trước khi sửa code, trước `AI_HANDOFF.md`, trước README.

## Phạm vi sản phẩm

- `SU_HDG` hiện có năm chức năng người dùng: `Siêu hàn cạnh`, `Phá đường cong`, `Làm mịn cạnh`, `Trim`, và `Bo góc`.
- Command ids: `hdg_weld_edges`, `hdg_explode_curves`, `hdg_smooth_edges`, `hdg_trim_solid`, `hdg_round_corner`.
- Tooltip hàn: `Nối các cạnh được kết nối thành 1 đường cong`.
- Tooltip rã: `Phá tất cả các đường cong đã chọn`.
- Tooltip làm mịn: `Làm mịn các cạnh đã chọn theo số segments`.
- Tooltip Trim: `Cắt target solid theo modifier solid, hỗ trợ chọn sẵn nhiều target`.
- Tooltip Bo góc: `Bo góc tấm ván theo bán kính nhập từ góc đang hover`.
- Không thêm lại các tool cũ như vẽ, dựng hình, view, purge, material, tag nếu chủ repo chưa yêu cầu rõ.
- Không gọi, không copy source, không copy icon từ TIG, CadFather, JHS, PTL, ABF, Toolbar Editor hay plugin bên thứ ba nào.

## Vai trò repository

- Repo source private: `BLACK-BOT25/SRC_PLUGIN_HDG`.
  - Lưu Ruby source, icon, script build, README, AI handoff, file `dist/SU_HDG.rbz`.
  - Repo này phải giữ private trừ khi chủ repo nói rõ là muốn đổi.
- Repo release public: `BLACK-BOT25/SRC_PLUGIN_HDG_RELEASE`.
  - Chỉ lưu artifact public cho auto update.
  - File hợp lệ: `SU_HDG.rbz`, `update-manifest.json`, `README.md`, `AGENTS.md`.
  - Tuyệt đối không đưa source Ruby, ghi chú riêng, token, key, secret vào repo release.

## Thứ tự đọc trước khi sửa

Khi bắt đầu task mới, đọc các file liên quan thay vì đoán:

- `AGENTS.md`
- `AI_HANDOFF.md`
- `README.md`
- `su_hdg.rb`
- `su_hdg/commands/catalog.rb`
- `su_hdg/config.rb`
- `su_hdg/updater.rb` nếu task liên quan auto update
- Release `update-manifest.json` nếu task liên quan phát hành

Nếu thấy file đang bị sửa bởi người dùng, không revert. Đọc diff và làm tiếp trên thay đổi đó.

## Quy tắc code

- Giữ Ruby rõ ràng, tên hàm biến có nghĩa, tách helper khi logic dài hoặc lặp lại.
- Nếu một file Ruby phình to quá khoảng 300 dòng, cân nhắc tách module nhỏ thay vì nhét thêm logic.
- Text hiện cho người dùng nên dùng tiếng Việt có dấu trong code/README khi có thể.
- File text nên là UTF-8, không BOM.
- Không hardcode token, cookie, key GitHub, đường dẫn máy cá nhân vào code runtime.
- Không thêm dependency ngoài nếu SketchUp Ruby API hoặc standard library đã đủ.

## Quy tắc an toàn model

- Chức năng hàn cạnh chỉ xử lý `Sketchup::Edge` trong `model.active_entities` và selection hiện tại.
- Cần chọn cạnh trước khi bấm icon.
- Nếu chưa chọn cạnh nào, bấm icon không có hiệu lực và không chuyển sang tool click cạnh.
- Khi đã chọn nhiều cạnh, chức năng phải hàn selection hiện tại.
- Edge đã thuộc `curve` vẫn phải xử lý được bằng cách tách curve cũ và dựng lại curve mới.
- Không xóa cạnh gốc thủ công khi không cần; ưu tiên tạo curve qua group tạm rồi explode để SketchUp merge geometry, giảm rủi ro phá mặt.
- Chuỗi có nhánh phức tạp nên được tách tại điểm nhánh thay vì đoán thành một chuỗi duy nhất.
- Chức năng `Phá đường cong` chỉ rã curve/edge thuộc curve trong selection hiện tại.
- `Phá đường cong` dùng SketchUp Ruby API `explode_curve`, không gọi CadFather/JHS/plugin ngoài.
- Chức năng `Làm mịn cạnh` chỉ xử lý cạnh/curve trong selection hiện tại, hỏi `Segments:` trước khi chạy, và không chuyển sang tool click cạnh.
- `Làm mịn cạnh` phải xử lý được cạnh/curve đang nằm trên mặt. Tạo curve/circle/arc mới trước, xóa cạnh cũ sau, rồi gọi `find_faces` trên cạnh mới khi có thể.
- `Làm mịn cạnh` chỉ nhận diện cạnh bo cong. Cạnh thẳng rời hoặc curve thẳng/collinear phải được bỏ qua.
- `Trim` chỉ xử lý `Sketchup::Group` hoặc `Sketchup::ComponentInstance` là solid hợp lệ.
- `Trim` chạy bằng tool kiểu target trước, modifier sau: có thể chọn sẵn nhiều target trước khi bấm icon; nếu chưa chọn sẵn thì click vật 1 là target bị cắt, click vật 2 là modifier/vật cắt; click modifier xong thì tự cắt, không mở dialog hỏi lại, và vẫn ở trong tool để trim tiếp bằng modifier khác.
- `Trim` phải hiển thị số thứ tự chọn `1`/`2` tại vị trí con trỏ chuột, không đặt số ở tâm/bounds của đối tượng.
- `Trim` phải giữ modifier/vật cắt, thay từng target bị cắt bằng kết quả trim, và khôi phục name/tag/material/attributes của từng target cho kết quả. Không để kết quả mang tên `Difference` nếu có thể tránh.
- `Trim` không được để modifier và target đổi tên/màu cho nhau. Các mặt/cạnh của kết quả bị cắt nếu bị SketchUp gán màu từ modifier thì phải trả về màu của target hoặc để trống như ban đầu.
- `Bo góc` chạy bằng tool hover/click, ưu tiên cạnh/mặt thuộc context đang mở.
- `Bo góc` cũng được phép bắt cạnh nằm trong group/component qua `PickHelper`, nhưng phải sửa đúng `entities` chứa cạnh đó và vẽ preview bằng transformation của pick path.
- `Bo góc` phải chọn cặp cạnh/mặt có khoảng bo hợp lý nhất quanh đỉnh hover, tránh lấy nhầm cạnh độ dày làm giới hạn bán kính.
- `Bo góc` khi rê chuột vào cạnh gần góc phải hiện preview giống kết quả sau khi cắt, trong suốt theo màu mặt/tấm gốc và viền preview màu cam dễ thấy.
- `Bo góc` khi apply phải tạo mặt cutter kín ở góc bo rồi xóa đúng face nhỏ chứa góc; không chỉ vẽ arc/line lên mặt rồi hy vọng SketchUp tự cắt.
- `Bo góc` khi apply cần rebuild lại face lớn bằng loop đã thay đỉnh góc bằng cung bo nếu SketchUp không tự chia face, rồi tự thoát tool để bỏ preview sau khi cắt.
- `Bo góc` khi click phải hỏi `Bán kính bo góc:`, tạo cung bo theo bán kính nhập, tự giảm về bán kính tối đa hợp lệ nếu số nhập quá lớn, và dùng số segments tự tính để cạnh cắt mịn.
- `Bo góc` phải rebuild mặt bằng chính cạnh thuộc face, không tạo curve/cạnh rời nằm chồng lên mặt vì sẽ làm rã ra thấy nét không liền mặt.
- `Bo góc` phải rebuild cả hai mặt cạnh thẳng ở góc, không chỉ rebuild mặt trên/dưới; phần khối góc cũ phải bị cắt thật.
- Cung bo sau khi tạo phải được hàn thành curve và các cạnh segment nội bộ trên mặt cong phải soft/smooth/hidden để nhìn liền như mặt bo thực tế.
- Auto hàn cung bo phải dùng lại lõi `Siêu hàn cạnh` nếu có thể, không chỉ tự tạo curve trùng kiểu fallback.
- `Bo góc` phải cho nhập bán kính bằng VCB rồi Enter; nếu mở ô nhập riêng thì Enter phải xác nhận như nút OK.
- `Bo góc` phải tự tăng segments cao để cung sau khi cắt mịn.
- Không rebuild bằng cách `add_item` lại vào toolbar cũ tên `SU_HDG`; SketchUp không có API clear/remove toolbar item, nên phải dựng toolbar runtime tên riêng khi cần rebuild.
- Toolbar runtime được phép dùng suffix ẩn để tránh nhân item, nhưng title người dùng nhìn thấy chỉ nên là `SU_HDG`, không kèm version.
- Toolbar command phải có trạng thái active/checked để người dùng biết icon nào vừa bấm hoặc tool nào đang chạy.
- Icon thường không được có nền SVG phủ kín vì sẽ che hover/checked native của SketchUp; nếu cần trạng thái đang chạy thì dùng icon `_active.svg` nền xanh rõ.
- Không gọi `UI::Toolbar.new(name)` chỉ để hide toolbar cũ; nếu toolbar đó chưa tồn tại trong session, SketchUp có thể tạo toolbar rỗng.
- Khi có lỗi hoặc dữ liệu không hợp lệ, báo tóm tắt rõ, không im lặng phá model.

## Quy tắc debug

- Không đoán nguyên nhân nếu chưa kiểm tra code, config, manifest, archive hoặc log liên quan.
- Khi sửa bug, ghi rõ root cause trong commit message hoặc final response nếu cần.
- Nếu không thể chạy SketchUp manual test trong môi trường hiện tại, phải nói rõ phần nào chưa test.

## Build và kiểm tra

Trước khi commit thay đổi code plugin:

1. Chạy build:

```powershell
.\scripts\build_rbz.ps1
```

2. Kiểm tra archive có root đúng:

```powershell
tar -tf dist\SU_HDG.rbz
```

3. Xác nhận:
   - `su_hdg/commands/catalog.rb` chỉ đăng ký các command đang được yêu cầu.
   - `su_hdg/config.rb` default toolbar khớp danh sách command hiện hành.
   - `su_hdg/icons/` chỉ có icon cần dùng.

Với thay đổi tài liệu thuần túy, không cần rebuild RBZ nhưng phải kiểm tra diff trước commit.

## Workflow release auto update

Khi tạo bản phát hành mới:

1. Sửa source trong repo private.
2. Tăng `VERSION` trong `su_hdg.rb`.
3. Build `dist\SU_HDG.rbz`.
4. Commit và push repo source private.
5. Copy `dist\SU_HDG.rbz` sang repo release public thành `SU_HDG.rbz`.
6. Sửa `update-manifest.json`:
   - `version` phải khớp `VERSION`.
   - `rbz_url` dùng raw URL public.
   - `notes` ngắn gọn, tiếng Việt.
7. Commit và push repo release public.
8. Nếu có thể, verify raw manifest qua URL GitHub.

## Git

- Không dùng `git reset --hard`, `git checkout --`, xóa recursive hoặc lệnh phá hủy nếu chủ repo chưa yêu cầu rõ.
- Trước commit, xem `git status -sb` và diff liên quan.
- Commit riêng source và release. Không gom source Ruby vào repo release.
- Không revert thay đổi của người dùng.

## Trả lời cho chủ repo

- Trả lời tiếng Việt, ngắn gọn, nói rõ đã sửa file nào và đã test gì.
- Nếu đã commit/push, nói rõ commit nào ở repo source và repo release.
- Nếu còn test thủ công trên SketchUp chưa làm được, nói thẳng.
