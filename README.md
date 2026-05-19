# Dashboard_Amazon

Next.js 14 frontend cho Amazon Niche & ASIN Dashboard. Hai trang chính:

- `/` — **Overview & ASINs**: KPIs, filter, sort, niche browser + ASIN images
- `/growth` — **Growth Analysis**: segments, scatter, histogram, top growers, compare 2–6 niches

Data lấy từ FastAPI server riêng. Đầy đủ hướng dẫn ở `NEXT_STEPS.md` và `DEPLOY_VERCEL.md`.

## Stack

- Next.js 14 App Router + TypeScript
- Tailwind CSS (dark theme custom)
- Recharts cho charts
- Server Components fetch FastAPI, ISR cache 5 phút
- API key giữ server-side (env, không có `NEXT_PUBLIC_`)

## Quick start

```bash
npm install
cp .env.local.example .env.local
# Điền FASTAPI_URL, FASTAPI_KEY trong .env.local
npm run dev    # http://localhost:3000
```

Build production:
```bash
npm run build && npm start
```

## Deploy

Xem `NEXT_STEPS.md` cho luồng đầy đủ (npm install → push GitHub → deploy Vercel → update CORS).
Xem `DEPLOY_VERCEL.md` cho chi tiết Vercel config, custom domain, self-host Docker.

## Cấu trúc

```
app/
├── layout.tsx, page.tsx, loading.tsx, error.tsx, not-found.tsx
├── globals.css
├── growth/page.tsx, growth/loading.tsx
└── api/snapshot/route.ts        Proxy endpoint
components/
├── Shell.tsx                    Header, Nav, Container, Card
├── DashboardClient.tsx          UI cho /
└── GrowthClient.tsx             UI cho /growth
lib/
├── types.ts                     Niche / Asin / Snapshot types
├── api.ts                       fetchSnapshot()
└── format.ts                    Formatters + growth segment logic
```

## Scripts

```bash
npm run dev        # Dev server
npm run build      # Production build
npm start          # Run prod build
npm run lint       # ESLint
npm run typecheck  # tsc --noEmit
```

## Environment variables

| Name | Required | Mô tả |
|------|----------|------|
| `FASTAPI_URL` | có | URL public của FastAPI server |
| `FASTAPI_KEY` | có | Trùng với `API_KEY` trong `server/.env` |
| `SNAPSHOT_REVALIDATE` | không | Cache TTL giây (default 300) |
