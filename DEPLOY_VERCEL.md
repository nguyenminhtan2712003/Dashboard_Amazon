# Deploy hướng dẫn: Next.js dashboard lên Vercel

Tài liệu này hướng dẫn cụ thể từng bước. Bạn cần:
- Tài khoản GitHub (free)
- Tài khoản Vercel (free, ký bằng GitHub)
- FastAPI server đã chạy và có public URL (xem `server/README.md`)

## Tổng quan

```
[Vercel: Next.js dashboard]  ──HTTPS+API key──→  [Server riêng của bạn: FastAPI + Postgres]
```

Vercel chỉ host Next.js. FastAPI vẫn ở server của bạn (DigitalOcean, Render, AWS, VPS, etc.).

---

## Bước 1 — Đảm bảo FastAPI có public URL

Vercel cần truy cập được FastAPI từ Internet. Mấy phương án:

| Vị trí FastAPI | Cách public |
|---|---|
| VPS / cloud VM | Open port 8000 hoặc đặt sau Nginx + HTTPS với Let's Encrypt |
| Render / Railway / Fly.io | Tự động có URL https |
| Local (chỉ test) | Dùng `ngrok http 8000` để có URL tạm |

**Quan trọng**: trong `server/.env`, set `CORS_ORIGINS=https://<your-vercel-domain>.vercel.app`. Khi chưa có domain, để `*` để dev, đổi sau khi deploy xong.

Test public:
```bash
curl -H "X-API-Key: <key>" https://api.your-domain.com/api/meta
# phải trả về JSON, không lỗi
```

---

## Bước 2 — Push code Next.js lên GitHub

### Cách A — Repo riêng cho `nextjs-app/`

```bash
cd nextjs-app
git init
git add .
git commit -m "init: amazon niche dashboard"
git branch -M main

# Tạo repo trên GitHub (vd: amazon-niche-dashboard), rồi:
git remote add origin https://github.com/<your-username>/amazon-niche-dashboard.git
git push -u origin main
```

### Cách B — Push cả folder `dashboard/` (giữ chung với server)

```bash
cd /path/to/dashboard
git init
git add .
git commit -m "init: full project"
git remote add origin https://github.com/<your-username>/dashboard.git
git push -u origin main
```

Với cách B, sau này deploy Vercel bạn sẽ set **Root Directory** = `nextjs-app`.

> Vercel free tier yêu cầu repo public hoặc bạn dùng GitHub Pro/Team. Hobby plan đã support private repo.

---

## Bước 3 — Tạo project trên Vercel

1. Mở https://vercel.com/new
2. Đăng nhập bằng GitHub → cho phép Vercel access repo
3. Chọn repo bạn vừa push → click **Import**
4. Trang config:
   - **Project Name**: `amazon-niche-dashboard` (hoặc tên bạn thích)
   - **Framework Preset**: Vercel tự detect Next.js ✓
   - **Root Directory**: nếu code nằm trong sub-folder thì click "Edit" và đặt `nextjs-app`. Nếu là repo riêng thì để mặc định.
   - **Build Command**: để mặc định (`next build`)
   - **Output Directory**: để mặc định
   - **Install Command**: để mặc định (`npm install`)

---

## Bước 4 — Set environment variables

Trước khi click Deploy, mở phần **Environment Variables**:

| Name | Value | Environment |
|---|---|---|
| `FASTAPI_URL` | `https://api.your-domain.com` | Production, Preview, Development |
| `FASTAPI_KEY` | Giá trị `API_KEY` trong `server/.env` | Production, Preview, Development |
| `SNAPSHOT_REVALIDATE` | `300` (5 phút) hoặc số khác | Production, Preview, Development |

Tick cả 3 environment cho mỗi biến. Không cần `NEXT_PUBLIC_` prefix — biến này phải stay server-side.

---

## Bước 5 — Deploy

Click **Deploy**. Đợi khoảng 1-2 phút. Khi thành công bạn sẽ thấy:

```
https://amazon-niche-dashboard-xxxx.vercel.app
```

Mở URL → dashboard load với data từ FastAPI của bạn. ✓

---

## Bước 6 — Update CORS trên FastAPI

Quay lại `server/.env`:

```
CORS_ORIGINS=https://amazon-niche-dashboard-xxxx.vercel.app
```

Restart FastAPI:
```bash
docker compose restart api   # nếu dùng Docker
# hoặc
systemctl restart your-fastapi-service
```

Lý do: bây giờ chỉ domain Vercel của bạn được phép gọi API, chặn các domain lạ.

---

## Tùy chọn: Custom domain

1. Trên Vercel project → **Settings** → **Domains**
2. Add domain (vd: `niche.your-domain.com`)
3. Update DNS theo hướng dẫn (CNAME tới `cname.vercel-dns.com`)
4. Vercel auto-issue SSL cert
5. Update `CORS_ORIGINS` trong `server/.env` cho domain mới

---

## Auto-redeploy khi push code

Khi đã link Vercel ↔ GitHub:
- Push lên branch `main` → Vercel auto-build và deploy lên Production
- Push lên branch khác → Vercel build và tạo Preview URL (cho từng PR)

```bash
# Sửa code...
git add .
git commit -m "fix: thêm filter brand"
git push
# Vercel tự deploy trong ~1 phút
```

---

## Troubleshooting

| Lỗi | Nguyên nhân & fix |
|---|---|
| `502 Bad Gateway` trên Vercel | FastAPI không reachable. Test bằng `curl` từ máy khác xem URL public ok không. |
| `FastAPI returned 401` | `FASTAPI_KEY` trên Vercel khác với `API_KEY` trong server/.env |
| `FASTAPI_URL must be set` | Quên thêm env var, hoặc thêm xong nhưng chưa **Redeploy** (Vercel cần redeploy để pickup env mới) |
| CORS error trong browser | Khi gọi /api routes trên cùng origin thì không có CORS issue. Nếu vẫn gặp → kiểm tra `CORS_ORIGINS` trên FastAPI. |
| Data cũ sau update | Vercel cache 5 phút theo `SNAPSHOT_REVALIDATE`. Để force refresh ngay, vào Vercel project → **Deployments** → **... menu** → **Redeploy**. |
| Build fail "Module not found" | Chạy `npm install` local check trước. Đảm bảo `package.json` được commit. |

---

## Cập nhật data sau này

Không cần redeploy Next.js. Chỉ cần:
1. Import CSV mới vào Postgres (`docker compose exec api python import_csv.py ...`)
2. Đợi `SNAPSHOT_REVALIDATE` giây hoặc redeploy Next.js để pickup ngay
3. Refresh trang dashboard

---

## Tự host (không dùng Vercel)

Nếu không muốn dùng Vercel, có 2 option:

### Option A — Node trực tiếp trên server

```bash
cd nextjs-app
npm install
npm run build
PORT=3000 npm start
```

Đặt sau Nginx reverse proxy + HTTPS. Set env vars trong systemd service hoặc `.env.local`.

### Option B — Docker

Tạo `nextjs-app/Dockerfile`:

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package*.json ./
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 3000
CMD ["npm", "start"]
```

Run:
```bash
docker build -t niche-dashboard .
docker run -p 3000:3000 \
  -e FASTAPI_URL=https://api.your-domain.com \
  -e FASTAPI_KEY=xxx \
  niche-dashboard
```

---

## Checklist trước khi go-live

- [ ] FastAPI có public HTTPS URL
- [ ] `API_KEY` server dài, ngẫu nhiên (`openssl rand -hex 32`)
- [ ] Vercel env vars đã set đầy đủ (FASTAPI_URL, FASTAPI_KEY, SNAPSHOT_REVALIDATE)
- [ ] `CORS_ORIGINS` trên FastAPI giới hạn vào domain Vercel (không để `*`)
- [ ] Test dashboard load đúng data trên Vercel URL
- [ ] (Optional) Custom domain + SSL
- [ ] (Optional) Rate limit ở Nginx hoặc Cloudflare đặt trước FastAPI
