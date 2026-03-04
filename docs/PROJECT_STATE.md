B2B CRM projesine devam ediyoruz. Aşağıda PROJECT_STATE var.

1. Supabase Project Name: b2b-crm
2. Region: Paris
3. Auth Settings: Email confirm açık mı kapalı mı?

Supabase Project Url: [https://yqwbzmnbhhcargbskzvc.supabase.co](https://yqwbzmnbhhcargbskzvc.supabase.co/)

Anon Key: 

**Publishable Key: sb_publishable_t3o4soEi7iRkO0O4F9S3Hw_U8As3M1x**

Direct connection: postgresql://postgres:[YOUR-PASSWORD]@db.yqwbzmnbhhcargbskzvc.supabase.co:5432/postgres

Host: [db.yqwbzmnbhhcargbskzvc.supabase.co](http://db.yqwbzmnbhhcargbskzvc.supabase.co/)

Port: 5432

database: postgres

User: postgres

# **🟠 PROJECT_STATE.md**

B2B CRM — Current Project Snapshot

# **1️⃣ Project Vision**

Bu proje:

- Internal single-company B2B CRM
- Satıcı performans takibi
- Min price approval sistemi
- Gerçek maliyet bazlı marj hesaplama
- Müşteri kaçışını engelleme
- 10 yıllık veri biriktirme hedefi

Ana hedef:

# Operasyonu disipline etmek ve ithalat kararlarını veriyle vermek.

# **2️⃣ Current Phase**

Phase: MVP

Current Sprint: [Sprint X – Başlık]

Next Target: [Örn: Sale Engine Approval]

# **3️⃣ Tech Stack**

Frontend: Next.js (App Router)

Language: TypeScript

Backend: Supabase (PostgreSQL)

Auth: Supabase Auth

RLS: Enabled

Deployment: Vercel (planned / active)

Environment: Windows

# **4️⃣ Database Status**

Schema Version: v1

RLS: Enabled

Tables Implemented:

- profiles
- customers
- products
- sales
- inquiries
- tasks
- exchange_rates
- activity_logs

Snapshot Logic: Designed / Partially implemented / Not implemented

Approval Engine: Designed / Partially implemented / Not implemented

# **5️⃣ What Is Working?**

Örnek format:

✔ Auth working

✔ Profile auto creation

✔ Customers table

✔ RLS isolation (sales sees only own customers)

✔ Customers list page

# **6️⃣ What Is NOT Built Yet?**

Örnek:

✘ Sales engine

✘ Min price approval flow

✘ Stock deduction logic

✘ Dashboard

✘ Cross-sell logic

# **7️⃣ Current Blocker (If Any)**

[Buraya varsa teknik problem yazılır]

# **8️⃣ Data Integrity Rules (Important)**

- Approved sale immutable
- Snapshot fields immutable
- Logs append-only
- Stock cannot go negative
- Sales cannot see other sales’ customers

# **9️⃣ Architecture Decisions Locked**

Buraya değişmemesi gereken kararlar yazılır:

✔ Single tenant

✔ USD base pricing

✔ TRY sales

✔ Exchange rate snapshot

✔ No FIFO in MVP

# **🔟 Immediate Next Action**

Örn:

- Build sale creation endpoint
- Implement approval status transition
