# Deploy guide (GitHub + Supabase + Render, all free, no card)

## Run locally (optional)

```powershell
cd "course management"
python -m venv ..\.venv
..\.venv\Scripts\pip install -r ..\requirements.txt
..\.venv\Scripts\python app.py
```
Open http://localhost:5000. With no `.env` file it uses a local SQLite file and a local `uploads/` folder.
To use Supabase from your laptop, copy `.env.example` to `course management\.env` and fill it in.

---

## A. Put the code on your GitHub

1. Go to https://github.com → **+** (top right) → **New repository**.
2. Name: `course-management-cloud`. Choose **Public** (or Private) → do NOT tick "Add README" → **Create repository**.
3. In PowerShell, inside the `Course` folder:
   ```powershell
   git remote set-url origin https://github.com/YOUR-USERNAME/course-management-cloud.git
   git push -u origin main
   ```
   (A browser window asks you to log in to GitHub the first time.)
4. Refresh the GitHub page. Check that there is **no `.env` file** in the list.

## B. Create the Supabase project (database + storage)

1. Go to https://supabase.com → **Start your project** → **Continue with GitHub**.
2. **New project**:
   - Name: `course-management`
   - Database Password: click **Generate a password**, but if it has symbols, type your own using only letters and numbers (for example `CloudCms2026Pass`). **Write it down.**
   - Region: **South Asia (Mumbai)** (`ap-south-1`)
   - Plan: Free → **Create new project**. Wait about 2 minutes.
3. **Storage bucket**: left menu **Storage** → **New bucket** → Name `uploads` → keep **Public bucket OFF** (private) → **Create**.
4. **Database URL**: click **Connect** (top of the page) → **Connection String** tab → under **Session pooler** copy the URI. It looks like:
   `postgresql://postgres.abcdxyz:[YOUR-PASSWORD]@aws-0-ap-south-1.pooler.supabase.com:5432/postgres`
   Replace `[YOUR-PASSWORD]` with your password (remove the brackets).
   Use the **Session pooler**, not "Direct connection". Render's free plan can't reach the direct one.
5. **API keys**: left menu **Project Settings** (gear) → **API** (or **API Keys**):
   - copy **Project URL** → this is `SUPABASE_URL`
   - copy the **service_role** key (click Reveal). This is `SUPABASE_SERVICE_KEY`. It's a secret: never put it in GitHub.

You don't need to create any tables. The app creates them, along with the demo users, the first time it starts.

## C. Deploy on Render

1. Go to https://render.com → **Get Started** → **GitHub** login (no card needed).
2. **New +** → **Web Service** → connect your GitHub → choose `course-management-cloud` → **Connect**.
3. Fill in:
   - Name: `course-management` (your URL becomes `https://course-management-xxxx.onrender.com`)
   - Region: **Singapore** (closest to Mumbai)
   - Branch: `main`
   - Root Directory: *(leave empty)*
   - Runtime: **Python 3**
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `gunicorn --chdir "course management" app:app --workers 1 --threads 4 --timeout 120`
   - Instance Type: **Free**
4. **Environment Variables** → **Add Environment Variable** (4 of them):

   | Key | Value |
   |---|---|
   | `SECRET_KEY` | click **Generate** |
   | `DATABASE_URL` | the Session pooler URI from B4 |
   | `SUPABASE_URL` | Project URL from B5 |
   | `SUPABASE_SERVICE_KEY` | service_role key from B5 |

5. **Deploy Web Service**. Watch the logs. When you see `Your service is live 🎉`, open the URL at the top.

## D. Test on the live URL

1. Log in as `admin@cms.edu` / `admin123` → open Courses, Users, Enrollments, Reports.
2. Log out → log in as `faculty@cms.edu` / `faculty123` → open a course → **upload a study material** (any small PDF).
3. Supabase → Storage → `uploads` → `materials`: your file should be there. **This shows file storage works.**
4. Supabase → **Table Editor** → `users`, `courses`: the data is there. **This shows the database works.**
5. Log in as `student@cms.edu` / `student123` → My Courses → open the course → **Download** the material. The link is a signed link that expires after 5 minutes.
6. In Render, click **Manual Deploy → Restart service**. Log in again: your data and file are still there.

Note: the free Render server sleeps after 15 minutes without visitors. The first visit after that takes about 30–60 seconds to wake it up.

## Troubleshooting

- `password authentication failed` → wrong password in `DATABASE_URL`, or you left the `[ ]` brackets in.
- `Network is unreachable` / IPv6 error → you used "Direct connection". Use the **Session pooler** URI.
- Upload error `400/403` → bucket name isn't exactly `uploads`, or you used the anon key instead of the service_role key.
