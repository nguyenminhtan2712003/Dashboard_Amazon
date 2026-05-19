# Next Steps — chạy local + push GitHub + deploy Vercel

## 1. Cài deps trên máy local (cần Node.js 18+)

```bash
cd C:\Users\PC\OneDrive - Canawan Global\Desktop\dashboard\Dashboard_Amazon
npm install
```

Đợi vài phút. Sau khi xong sẽ có `node_modules/` và `package-lock.json` đầy đủ.

## 2. Tạo file `.env.local` để dev local

```bash
cp .env.local.example .env.local
```

Mở `.env.local`, điền:

```
FASTAPI_URL=https://api.your-domain.com   # hoặc http://localhost:8000 nếu chạy FastAPI local
FASTAPI_KEY=<API_KEY trùng với server/.env>
SNAPSHOT_REVALIDATE=300
```

## 3. Test dev server

```bash
npm run dev
```

Mở http://localhost:3000 → dashboard load. Kiểm tra cả `/` và `/growth`.

Nếu trang load lỗi → mở console hoặc xem `app/error.tsx` cho hint troubleshoot.

## 4. Kiểm tra TypeScript + Lint

```bash
npm run typecheck   # phải pass không có lỗi
npm run lint        # warnings ok, errors không nên có
npm run build       # build production thử trước khi push
```

## 5. Commit + push lên GitHub

Folder này đã có `.git` rồi. Nếu repo GitHub đã setup remote, đẩy lên:

```bash
git status                                          # xem files thay đổi
git add .
git commit -m "feat: implement Next.js dashboard (Overview + Growth Analysis)"
git push origin main
```

Nếu chưa có remote, tạo repo trên github.com rồi:

```bash
git remote add origin https://github.com/<your-username>/Dashboard_Amazon.git
git branch -M main
git push -u origin main
```

> `.gitignore` đã loại `node_modules`, `.next`, `.env.local` để không upload thừa.

## 6. Deploy lên Vercel

1. Mở https://vercel.com/new
2. Login bằng GitHub, **Import** repo `Dashboard_Amazon`
3. Page settings:
   - **Framework Preset**: Vercel tự detect Next.js
   - **Root Directory**: để trống (vì repo root chính là Next.js project)
4. Phần **Environment Variables**, add 3 biến (tick cả 3 environment Production/Preview/Development):
   - `FASTAPI_URL` = URL public của FastAPI server
   - `FASTAPI_KEY` = giá trị `API_KEY` của FastAPI
   - `SNAPSHOT_REVALIDATE` = `300`
5. Click **Deploy** → đợi ~1–2 phút → có URL `https://dashboard-amazon-xxxx.vercel.app`

## 7. Update CORS trên FastAPI

Vào server, sửa `.env`:

```
CORS_ORIGINS=https://dashboard-amazon-xxxx.vercel.app
```

Restart FastAPI (`docker compose restart api`).

## 8. Auto-deploy

Từ giờ mỗi lần `git push origin main`, Vercel tự build + deploy. Mỗi PR có Preview URL riêng.

```bash
# Workflow update sau này
git add .
git commit -m "fix: ..."
git push
# Vercel tự deploy ~1 phút
```

## Troubleshooting

| Vấn đề | Giải pháp |
|---|---|
| `npm install` báo `EJSONPARSE` | Mở `package.json` xem có thiếu `}` cuối file không. File hiện tại đúng JSON 656 bytes. |
| `npm run typecheck` báo `Cannot find module 'next'` | Chưa chạy `npm install`. Chạy lại. |
| Vercel build fail `Module not found` | Đảm bảo `package-lock.json` đã commit lên GitHub |
| Dashboard ra blank trên Vercel | Chưa set env vars hoặc set xong chưa Redeploy. Vào Vercel project → Deployments → ... menu → Redeploy |
| Error "FastAPI returned 401" | `FASTAPI_KEY` trên Vercel != `API_KEY` trên server |
| Error "502 Bad Gateway" | FastAPI không reachable từ Internet. Test bằng `curl -H "X-API-Key: $KEY" $URL/api/meta` từ máy khác |

Chi tiết đầy đủ ở `DEPLOY_VERCEL.md`.
