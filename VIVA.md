# VIVA – Course Management System on the Cloud

**Live URL:** https://YOUR-APP.onrender.com  *(fill in after deploy)*
**Code:** https://github.com/YOUR-USERNAME/course-management-cloud

Demo logins: `admin@cms.edu / admin123`, `faculty@cms.edu / faculty123`, `student@cms.edu / student123` (also `kavitha@cms.edu`, `vignesh@cms.edu`, same password).

## 1. Where is the database stored?
In **Supabase PostgreSQL**, in the **Mumbai (ap-south-1)** region. Supabase runs it on Amazon Web Services servers in Mumbai.
The app reads the connection string from the `DATABASE_URL` environment variable. Tables (users, courses, enrollments, assignments, submissions, exams, results, materials, announcements) are created automatically on first start.

## 2. Where are the files stored?
In a **private Supabase Storage bucket** called `uploads`, with folders `materials/`, `assignments/` and `submissions/`.
Render's free server deletes local files whenever it restarts, so uploads can't be kept on the server.
To download a file, the app checks that the user may see it and then gives a **signed link that expires after 5 minutes**.

## 3. IaaS / PaaS / SaaS in my project
| Model | In my project |
|---|---|
| **IaaS** (Infrastructure as a Service: raw servers, storage, network) | **AWS** data centres (Mumbai) underneath Supabase. I don't manage it directly. |
| **PaaS** (Platform as a Service: I give the code, the platform runs it) | **Render** runs my Flask app (it builds from GitHub and runs Gunicorn). **Supabase** gives me a ready database and storage. |
| **SaaS** (Software as a Service: ready software used through a browser) | **My Course Management System itself**: students and faculty just open the URL. **GitHub** is also SaaS (code hosting). |

## 4. Why I chose these platforms
- **Free with no credit card**: Render, Supabase and GitHub all have free plans.
- **Render**: deploys straight from GitHub, gives free HTTPS, and supports Python/Gunicorn without any server setup.
- **Supabase**: real PostgreSQL (a proper production database, unlike SQLite) plus file storage in one place, with a **Mumbai** region close to our users.
- **GitHub**: version control, and Render redeploys automatically on every push.

## 5. How the data is secure
- **HTTPS**: Render gives the site a TLS certificate, so passwords and data are encrypted in transit.
- **Passwords are hashed** (Werkzeug `generate_password_hash`), never stored as plain text.
- **Secrets are not in the code**: `SECRET_KEY`, `DATABASE_URL` and the Supabase key are environment variables. `.env` is listed in `.gitignore`.
- **Private bucket**: files aren't public. Only logged-in users with access get a short-lived (5-minute) signed link.
- **Role-based access**: admin, faculty and student pages check the user's role.
- **Supabase** encrypts stored data at rest and keeps daily backups (on paid plans; free has limited backups).
- *Known limitation (existing design, not changed):* the public Register page lets anyone choose the "Admin" role. In real use, admin signup should be removed.

## 6. Cost: free cloud vs buying a college server
| | Cloud (this project) | College server (approx.) |
|---|---|---|
| Upfront | ₹0 | ₹1.5 – 3 lakh (server + UPS + rack) |
| Monthly | ₹0 (free tiers) | Electricity, internet with static IP, AC: around ₹3,000–8,000 |
| Staff | None. Platform handles OS, patches and HTTPS | A system admin is needed |
| Setup time | About 30 minutes | Weeks (purchase, install, configure) |
| If it grows | Paid plans: Render ~$7/month, Supabase Pro ~$25/month | Buy more hardware |

**Free tier limits:** Render free sleeps after 15 minutes with no visitors (first load then takes ~30–60 s) and has 512 MB RAM. Supabase free gives 500 MB database and 1 GB file storage, and **pauses the project after 7 days with no activity** (un-pause it in the dashboard before the viva).

## 7. Results: live response time
Measured with `curl` (built into Windows), no extra code. Run in PowerShell:
```powershell
1..5 | ForEach-Object { curl.exe -o NUL -s -w "%{http_code}  %{time_total}s`n" https://YOUR-APP.onrender.com/login }
```

| Request | Time |
|---|---|
| 1st request (server was asleep, cold start) | _fill in_ s |
| Next requests (average of 4) | _fill in_ s |

*Observation:* the first request is slow because the free Render server wakes up from sleep. After that, pages load in well under a second.
