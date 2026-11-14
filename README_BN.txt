HOMELAND LOGISTICS SERVICE COMPANY
INTEGRATED RELEASE CANDIDATE — V8 STAFF PORTAL + V4 CUSTOMER TRACKING
======================================================================

এই প্যাকেজে কী আছে
-----------------
- index.html: V8 Admin/Driver cloud portal; V4 Customer Tracking link যুক্ত।
- public-tracking.html: গ্রাহকের জন্য English/Arabic public tracking page।
- config.js: Supabase Project URL ও Publishable Key-এর public template।
- sql/01...04: V1 core schema থেকে V4 public tracking পর্যন্ত SQL।
- sql/03_VERIFY_V3.sql ও sql/04_VERIFY_V4.sql: read-only verification query।
- supabase/functions/admin-invite-driver/index.ts: Admin-only driver invitation Edge Function।
- supabase/config.toml: Edge Function JWT verification configuration।

এটি কী নয়
----------
এটি deploy-ready integration starter, কিন্তু আপনার Supabase project-এ SQL চালানো,
Auth user তৈরি, Storage bucket তৈরি, Edge Function deploy বা Hosting publish করা হয়নি।
অতএব এটি এখনো live production system নয়। Live database, email, Storage, public RPC,
HTTPS domain ও বাস্তব Android device-এ পরীক্ষা বাকি।

গুরুত্বপূর্ণ: সঠিক Supabase project বেছে নিন
-------------------------------------------
শুধু HOMELAND LOGISTICS-এর জন্য নির্ধারিত একটি Supabase project ব্যবহার করুন।
BAMS বা অন্য কোনো project-এ এই SQL চালাবেন না। Project URL/Project Ref মিলিয়ে নিন।

SQL চালানোর নিয়ম
----------------
Supabase Dashboard > SQL Editor ব্যবহার করলে নিচের ক্রম অনুসরণ করুন:

নতুন, খালি HOMELAND LOGISTICS project হলে:
  1. sql/01_SCHEMA_V1.sql
  2. sql/02_MIGRATION_V2_DRIVER_RLS.sql
  3. sql/03_MIGRATION_V3_PROOF_OF_DELIVERY.sql
  4. sql/04_MIGRATION_V4_PUBLIC_TRACKING.sql

যদি একই project-এ V1, V2 ও V3 আগে সফলভাবে চালানো হয়ে থাকে:
  - শুধু sql/04_MIGRATION_V4_PUBLIC_TRACKING.sql চালান।

যদি V1/V2/V3-এর কোনটি অসম্পূর্ণ, error-সহ চালানো, বা অন্য version-এর schema থাকে:
  - থামুন। আগে existing table, function ও policy যাচাই করুন। এই starter SQL
    পুরোনো schema-কে স্বয়ংক্রিয়ভাবে merge/repair করার নিশ্চয়তা দেয় না।

একই migration SQL Editor ও Supabase CLI—দুই পদ্ধতিতে চালাবেন না। এই প্যাকেজে
manual SQL Editor-এর জন্য সাজানো SQL দেওয়া হয়েছে। প্রতিটি ধাপ সফল না হলে পরেরটি
চালাবেন না। আগে database backup/export রাখুন।

V3 ও V4 যাচাই
-------------
V3-এর পর sql/03_VERIFY_V3.sql চালান।
V4-এর পর sql/04_VERIFY_V4.sql চালান।
V3-তে hls-delivery-proof bucket PRIVATE থাকতে হবে। V4-তে public RPC এবং anon execute
permission দেখা উচিত। V4 sample tracking ID-তে HLS-YYMMDD- এর পর 32টি hex character
থাকবে। Verification query-তে পুরোনো sequential ID lookup শূন্য row ফেরত দেবে।

প্রথম Admin account
-------------------
V1 সফল হওয়ার পর Supabase Authentication > Users-এ একটি company admin Auth user তৈরি
করুন। তার User UID নিয়ে SQL Editor-এ (শুধু একবার, যদি profile আগে না থাকে) চালান:

insert into public.hls_profiles (user_id, full_name, app_role, driver_id)
values ('AUTH-USER-UUID-HERE', 'Company Administrator', 'admin', null);

' AUTH-USER-UUID-HERE ' জায়গায় বাস্তব Auth UID বসাবেন; অতিরিক্ত space দেবেন না।
আগে থেকেই profile থাকলে duplicate insert করবেন না—আগের row যাচাই করুন।

Website config ও Hosting
------------------------
1. config.js খুলুন।
2. url-এ আপনার HOMELAND Supabase Project URL দিন।
3. publishableKey-তে Supabase Publishable Key (অথবা legacy anon key) দিন।
4. sb_secret_ বা service_role key কখনো config.js, HTML, browser, GitHub বা public repo-তে
   রাখবেন না। Browser-এ Publishable/anon key থাকা স্বাভাবিক; নিরাপত্তা RLS-এর উপর নির্ভর করে।
5. index.html, public-tracking.html ও config.js একই website root/folder-এ রাখুন।
6. HTTPS static hosting-এ publish করুন। GitHub repository root-এ এই তিনটি ফাইল রাখলে
   root index.html হবে Staff/Admin portal; public-tracking.html হবে Customer portal।

Deploy-এর আগে আগের index.html ও config.js backup করুন। পুরোনো app ফাইল overwrite করার
আগে এই package-এর patched index.html-এ Customer Tracking link আছে কি না দেখুন।
GitHub Pages/Netlify/Vercel-এর free-tier, repo visibility, build, bandwidth, domain ও
usage limits সময়ের সঙ্গে বদলাতে পারে; deploy-এর দিন provider dashboard-এ যাচাই করুন।

Supabase Auth URL settings
--------------------------
Supabase Authentication > URL Configuration-এ deployed HTTPS site URL-কে Site URL
হিসেবে দিন এবং invite/recovery redirect-এর জন্য deployed app URL allow-list করুন।
Edge Function-এর HLS_APP_URL-ও একই deployed app URL হবে।

Driver invitation function
---------------------------
কম্পিউটারে Supabase CLI ব্যবহার করে package root-এ:

  supabase login
  supabase link --project-ref YOUR_PROJECT_REF
  supabase functions deploy admin-invite-driver
  supabase secrets set HLS_APP_URL=https://YOUR-DOMAIN.example/

Supabase project-এর server-side default secret variables-এ SUPABASE_URL,
SUPABASE_ANON_KEY এবং SUPABASE_SERVICE_ROLE_KEY উপলব্ধ থাকতে হবে; নতুন key management
ব্যবহার করলে SUPABASE_PUBLISHABLE_KEYS / SUPABASE_SECRET_KEYS-ও সমর্থিত। Secret key
শুধু Edge Function runtime-এ থাকবে। Function deploy ও invitation email পরীক্ষা না করা
পর্যন্ত বাস্তব driver invite পাঠাবেন না। Production email delivery-র জন্য custom SMTP
প্রয়োজন হতে পারে; Supabase built-in SMTP-কে bulk production mail service ধরে নেবেন না।

Customer Public Tracking
------------------------
- নতুন V4 shipment tracking ID: HLS-YYMMDD- + 32 hex character suffix।
- Public portal শুধু Tracking ID, Status, Destination City ও timestamps দেখায়।
- Sender/Recipient নাম, phone, street address, COD, driver details, internal note বা
  Proof of Delivery public portal-এ প্রকাশ করা হয় না।
- V4-এর আগের sequential ID staff/admin portal-এ থাকবে, কিন্তু public portal-এ lookup
  হবে না। নতুন V4 ID তৈরি হলে public lookup কাজ করবে।
- Random ID সহজে enumerate করা কঠিন করে; এটি request flooding/DoS ঠেকায় না। বড় public
  traffic-এর আগে WAF বা rate-limited Edge Function যোগ করে পরীক্ষা করুন।

Fictional data দিয়ে end-to-end test
----------------------------------
1. Admin account দিয়ে index.html খুলে login করুন।
2. একটি কাল্পনিক shipment তৈরি করুন; বাস্তব customer/driver data দেবেন না।
3. নতুন tracking ID কপি করুন এবং 32-hex suffix আছে কি না দেখুন।
4. Driver account-এ কেবল assigned shipment দেখা যাচ্ছে কি না যাচাই করুন।
5. Status flow পরীক্ষা করুন: Booked > Picked Up > In Transit > Out for Delivery।
6. Driver Proof of Delivery-তে কাল্পনিক recipient name ও test signature দিন; photo optional।
7. Delivered হওয়ার পর Admin shipment details-এ POD দেখতে পারে কি না যাচাই করুন।
8. public-tracking.html-এ নতুন tracking ID দিয়ে status দেখুন।
9. Arabic toggle-এ RTL এবং English toggle-এ LTR যাচাই করুন।
10. Public page-এ customer phone/name/address, COD, driver বা POD link প্রকাশ হচ্ছে না
    নিশ্চিত করুন। অন্য account দিয়ে অন্য driver's shipment পড়ার চেষ্টা করে RLS পরীক্ষা করুন।

সীমাবদ্ধতা ও Saudi deployment সতর্কতা
------------------------------------
- Live GPS tracking, SMS/WhatsApp Business API, payment gateway, COD settlement,
  customer accounts, offline sync, automatic backups, invoices ও analytics BI নেই।
- COD amount শুধু shipment record-এ থাকে; টাকা সংগ্রহ/settlement হয় না।
- POD signature screen acknowledgement; certified digital signature নয়।
- Supabase project region/data residency, Saudi PDPL, cross-border transfer, privacy
  notice, retention/deletion, breach handling ও recipient evidence storage-এর বিষয়ে
  Saudi legal/privacy review না হওয়া পর্যন্ত বাস্তব customer data/POD photo/signature
  upload করবেন না।
- 30–100 driver rollout-এর আগে RLS, Storage policies, backup/restore, SMTP, monitoring,
  rate limiting, quota/cost, Android camera/touchscreen এবং multi-device test করুন।
- Free tier-এর quotas/limits/backup support পরিবর্তিত হতে পারে। Production-এর আগে
  Supabase ও hosting provider-এর বর্তমান pricing/limits যাচাই করুন।

