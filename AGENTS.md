# Quy tắc AI - SU_HDG_RELEASE

Repo này là repo public chỉ để plugin `SU_HDG` tự cập nhật. Đọc file này trước khi commit vào repo release.

## Phạm vi repo

Repo release chỉ được chứa artifact public:

- `SU_HDG.rbz`
- `update-manifest.json`
- `README.md`
- `AGENTS.md`

Không commit các thứ sau vào repo này:

- Ruby source: `*.rb`, thư mục `su_hdg/`, script build, icon source nếu không cần public.
- Ghi chú riêng nội bộ.
- Token GitHub, API key, cookie, secret.
- File tạm, log, archive debug, backup source.

## Nguồn đúng

- Source gốc nằm ở repo private: `BLACK-BOT25/SRC_PLUGIN_HDG`.
- Build RBZ từ repo source bằng:

```powershell
.\scripts\build_rbz.ps1
```

- Sau đó copy `dist\SU_HDG.rbz` sang repo này thành `SU_HDG.rbz`.

## Manifest

`update-manifest.json` phải có:

- `id`: `SU_HDG`
- `version`: khớp `VERSION` trong `su_hdg.rb` của repo source.
- `rbz_url`: raw URL public của file `SU_HDG.rbz`.
- `notes`: ghi chú ngắn gọn, tiếng Việt.

Raw URL đúng:

```text
https://raw.githubusercontent.com/BLACK-BOT25/SRC_PLUGIN_HDG_RELEASE/main/SU_HDG.rbz
```

Manifest URL dùng trong SketchUp:

```text
https://raw.githubusercontent.com/BLACK-BOT25/SRC_PLUGIN_HDG_RELEASE/main/update-manifest.json
```

## Workflow release

1. Đảm bảo source private đã được commit và push trước.
2. Copy RBZ mới từ repo source sang repo này.
3. Cập nhật `update-manifest.json`.
4. Kiểm tra không có source Ruby trong diff.
5. Commit và push repo release.
6. Nếu có thể, verify raw manifest bằng GitHub raw URL.

## Git safety

- Không dùng `git reset --hard`, `git checkout --` hay xóa file hàng loạt nếu chủ repo chưa yêu cầu rõ.
- Trước commit, chạy `git status -sb` và xem diff.
- Nếu thấy source Ruby xuất hiện trong diff, dừng lại và gỡ bỏ khỏi release repo.
