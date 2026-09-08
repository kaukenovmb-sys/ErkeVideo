# ErkeVideo

ErkeVideo — аккаунтсыз қауіпсіз видео бөлісуге арналған толық-stack жоба.

## Stack

- FastAPI + SQLAlchemy
- PostgreSQL
- FFmpeg / FFprobe
- Vanilla JS + CSS (жеңіл frontend)
- Docker Compose
- Argon2 admin password hashing
- SlowAPI rate limiting

## 1. Windows: Docker арқылы ең оңай іске қосу

Қажет:

- Docker Desktop
- Git (қаласаңыз)

Project папкасының ішінде:

```powershell
Copy-Item .env.example .env
```

`.env` ішіне ұзақ кездейсоқ `SECRET_KEY` және Argon2 admin hash қойыңыз.

Hash жасау:

```powershell
py -m pip install "pwdlib[argon2]"
py scripts/hash_password.py
```

Шыққан мәнді `.env` ішіндегі `ADMIN_PASSWORD_HASH=` жолына қойыңыз.

Сосын:

```powershell
docker compose up --build
```

Браузер:

- Site: http://localhost:8000
- Admin: http://localhost:8000/admin
- API docs: http://localhost:8000/api/docs
- Health: http://localhost:8000/health

## 2. Docker қолданбай development

### PostgreSQL

PostgreSQL іске қосып, database/user жасаңыз немесе локалды PostgreSQL connection string қолданыңыз.

### Backend

```powershell
cd backend
py -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

FFmpeg және FFprobe Windows PATH ішінде болуы керек.

`.env` файлын project root-та жасаңыз.

```powershell
uvicorn app.main:app --reload --port 8000
```

### Frontend

Frontend backend арқылы да беріледі, сондықтан бөлек dev server міндетті емес.

Егер Vite қолданғыңыз келсе, бұл нұсқа әдейі dependency-light болуы үшін vanilla JS ретінде қалдырылды.

## 3. Upload limit

`.env`:

```text
MAX_VIDEO_SIZE_MB=500
```

Мәнді өзгертіп backend-ті қайта іске қосыңыз.

## 4. Storage

Development/бір серверлік deployment үшін:

```text
uploads/
  videos/
  thumbnails/
```

Docker кезінде named volume қолданылады.

Production-та бұл storage қабатын S3-compatible object storage-ға ауыстыруға болады. Database тек storage path-ты сақтайды.

## 5. Production ескертулері

Production алдында:

- HTTPS қосыңыз.
- `SECRET_KEY` кездейсоқ әрі құпия болсын.
- PostgreSQL password-ты ауыстырыңыз.
- Reverse proxy (Nginx/Caddy) қолданыңыз.
- Object storage және backup стратегиясын қосыңыз.
- Upload quota және abuse monitoring-ті күшейтіңіз.
- FFmpeg-ті sandbox/container policy арқылы оқшаулаңыз.
- Privacy Policy мен Terms-ті нақты hosting юрисдикцияңызға бейімдеңіз.
- User-uploaded content moderation workflow-ын заң талаптарына сәйкестендіріңіз.

## Архитектура

`backend/app/routes_public.py` — public API, upload, search, views, reports.

`backend/app/routes_admin.py` — admin auth, dashboard, moderation.

`backend/app/services.py` — upload validation, random storage names, FFmpeg processing.

`backend/app/security.py` — signed admin session + CSRF.

`frontend/assets/app.js` — SPA-like lightweight frontend.

## Қауіпсіздік

Файл атауы storage path ретінде қолданылмайды. MIME allowlist, size limit, rate limits, signed admin cookie, CSRF, secure headers және SQLAlchemy parameterized queries қолданылады.

Бұл жоба тек заңды, қауіпсіз контентке арналған. Adult/explicit контентке арналған арнайы функционал жоқ.
