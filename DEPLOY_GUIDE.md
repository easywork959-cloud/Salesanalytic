# 🚀 Firebase Hosting Deployment Guide

> **เป้าหมาย**: Deploy Sales Analytics App ขึ้น Firebase Hosting แบบ Lazy-Load
> **เวลาที่ใช้**: ครั้งแรก ~20 นาที, ครั้งต่อๆ ไป ~1 นาที
> **ค่าใช้จ่าย**: ฟรี (Spark plan พอ)

---

## 📦 ไฟล์ในโฟลเดอร์นี้

```
firebase-deploy/
├── index.html                       (150 KB - HTML app เปล่า ไม่มี demo)
├── demo_data_302.json               (23 MB - demo data แยกไฟล์)
├── firebase.json                    (Firebase config)
├── .firebaserc                      (Firebase project ID — แก้)
├── .github/workflows/
│   └── firebase-deploy.yml          (Auto-deploy on push to GitHub)
└── DEPLOY_GUIDE.md                  (ไฟล์นี้)
```

---

## 🎯 Workflow ที่จะได้

```
ผู้ใช้เปิด https://your-app.web.app
   ↓
โหลด index.html (150 KB) → เร็ว ~1 วินาที
   ↓
กดปุ่ม "🎬 Load Demo" → fetch demo_data_302.json (23 MB)
   ↓ (cached ใน browser 30 วัน)
ครั้งถัดไป กด Load Demo → instant (จาก cache)
```

---

## 🛠 Setup ขั้นตอนทีละ step

### Step 1: ติดตั้งของที่ต้องใช้ (ทำครั้งเดียว)

```bash
# 1.1 ติดตั้ง Node.js (ถ้ายังไม่มี)
# โหลดจาก https://nodejs.org (LTS version)
# Verify:
node --version    # ควรเห็น v18+ หรือ v20+
npm --version

# 1.2 ติดตั้ง Firebase CLI
npm install -g firebase-tools

# Verify:
firebase --version    # ควรเห็น 13+
```

### Step 2: Login Firebase

```bash
firebase login
```

จะเปิด browser → login ด้วย Google account → Allow → กลับมาที่ terminal เห็น `✔ Success!`

### Step 3: สร้าง Firebase Project

ไปที่ [console.firebase.google.com](https://console.firebase.google.com)

1. กด **➕ Add project**
2. **Project name**: เช่น `sales-analytics-302`
   - Project ID จะถูกตั้งให้: `sales-analytics-302` (จดไว้)
   - ถ้าซ้ำ Firebase จะเพิ่ม suffix เช่น `sales-analytics-302-abc12`
3. **Google Analytics**: ⛔ **Disable** (ไม่จำเป็น ทำให้ setup ซับซ้อนขึ้น)
4. กด **Create project** → รอ ~30 วินาที
5. กด **Continue** เข้าหน้า Dashboard

### Step 4: ปรับ Project ID ในไฟล์ config

แก้ `.firebaserc` ในโฟลเดอร์นี้:

```json
{
  "projects": {
    "default": "sales-analytics-302"
  }
}
```

แทนที่ `REPLACE-WITH-YOUR-FIREBASE-PROJECT-ID` ด้วย Project ID ที่ Firebase สร้างให้

### Step 5: Deploy ครั้งแรก

```bash
# เข้าโฟลเดอร์
cd path/to/firebase-deploy

# Deploy
firebase deploy --only hosting
```

จะเห็น output แบบนี้:
```
=== Deploying to 'sales-analytics-302'...

i  deploying hosting
i  hosting[sales-analytics-302]: beginning deploy...
i  hosting[sales-analytics-302]: found 2 files in .
✔  hosting[sales-analytics-302]: file upload complete
i  hosting[sales-analytics-302]: finalizing version...
✔  hosting[sales-analytics-302]: version finalized
i  hosting[sales-analytics-302]: releasing new version...
✔  hosting[sales-analytics-302]: release complete

✔  Deploy complete!

Project Console: https://console.firebase.google.com/project/sales-analytics-302/overview
Hosting URL: https://sales-analytics-302.web.app
```

🎉 **เปิด `Hosting URL`** → ใช้ได้เลย!

---

## ✅ ทดสอบหลัง Deploy

1. เปิด URL ที่ Firebase ให้ — ควรโหลด ~1 วินาที (ไฟล์ 150 KB)
2. กด **🎬 Load Demo (DC 302)**
3. ดู modal "Downloading demo data..." (ครั้งแรก โหลด 23 MB ~10-30 วินาที)
4. หลังโหลดเสร็จ → เห็น Overview tab พร้อมข้อมูล
5. **F12 → Network tab** → ตรวจว่า `demo_data_302.json` มี status 200
6. Refresh หน้า → กด Load Demo อีกครั้ง → เร็วมาก (เพราะ cache)

---

## 🔄 Deploy ครั้งต่อๆ ไป (ง่ายมาก)

```bash
cd path/to/firebase-deploy
firebase deploy --only hosting
```

แค่นี้ — ไม่ต้อง login ซ้ำ ไม่ต้อง setup ซ้ำ

---

## 🌐 (Optional) Custom Domain

ใช้ `sales.yourcompany.com` แทน `sales-analytics-302.web.app`:

1. Firebase Console → **Hosting** → กด **Add custom domain**
2. ใส่ domain เช่น `sales.yourcompany.com`
3. Firebase จะให้ **2 DNS records** (A records)
4. ไปที่ DNS provider (Cloudflare/GoDaddy/etc.) → add A records
5. รอ propagation 24-48 ชั่วโมง (ปกติเร็วกว่านั้น)
6. Firebase auto-issue SSL ให้ ✓

---

## 🤖 (Optional) Auto-Deploy via GitHub

ถ้าอยากให้ **push code → auto deploy** ทันที:

### Step A: Setup GitHub repo

```bash
cd path/to/firebase-deploy
git init
git add .
git commit -m "Initial commit"

# สร้าง repo บน github.com → ชื่อเช่น sales-analytics-deploy
git remote add origin https://github.com/USER/sales-analytics-deploy.git
git push -u origin main
```

### Step B: ตั้ง Firebase service account

```bash
firebase init hosting:github
```

จะถาม:
- GitHub repo? → `USER/sales-analytics-deploy`
- Auto-deploy on PR? → Yes
- Auto-deploy on main? → Yes

Firebase จะสร้าง secret `FIREBASE_SERVICE_ACCOUNT_*` ใน GitHub repo ให้อัตโนมัติ

### Step C: แก้ workflow ให้ใช้ project ID จริง

แก้ `.github/workflows/firebase-deploy.yml`:
```yaml
projectId: sales-analytics-302    # ← ใส่ project ID ของคุณ
```

### Step D: ทดสอบ

```bash
# แก้อะไรซักอย่างใน index.html
git commit -am "Update"
git push
```

ไป **GitHub → Actions tab** → เห็น workflow running → ภายใน 1-2 นาที deploy เสร็จ → ตรวจ Hosting URL

---

## 📊 Monitoring & Quota

### ดู usage ที่ Firebase Console

- **Hosting** → **Usage** tab
- ดู bandwidth ของเดือนนี้

### Free Tier Limits (Spark Plan)

| Resource | Limit |
|---|---|
| Storage | 10 GB |
| Bandwidth | 360 MB / day = ~10 GB / month |
| SSL | Unlimited |
| Custom domain | Unlimited |

### ประมาณการใช้งาน

```
File sizes:
- index.html:           150 KB (cached, โหลดบ่อย)
- demo_data_302.json:   23 MB (cached 30 days, โหลดน้อย)

ใช้กับ team 30 คน, เปิดเฉลี่ย 5 ครั้ง/วัน/คน:
- 30 × 5 = 150 page loads/วัน
- 150 × 150 KB = 22 MB/วัน → ไกลจาก limit

ถ้า 50% โหลด demo (cache 30 วัน):
- ครั้งแรกของแต่ละคน: 30 × 23 MB = 690 MB
- หลังจากนั้น: ~0 (cached)
- เฉลี่ย: ~30 MB/วัน → ภายใน limit สบาย
```

---

## ⚠️ Troubleshooting

### Error: "Firebase Hosting: file upload failed"

**สาเหตุ**: ไฟล์ใหญ่เกิน หรือ network ขาด

**แก้**:
```bash
# ลอง deploy ใหม่
firebase deploy --only hosting --debug
```

### Demo load ไม่ขึ้น (CORS error)

**สาเหตุ**: Browser block CORS

**แก้**: ตรวจ `firebase.json` ว่ามี:
```json
"headers": [{
  "source": "**/demo_data_*.json",
  "headers": [{ "key": "Access-Control-Allow-Origin", "value": "*" }]
}]
```

### "Permission denied" deploy

**สาเหตุ**: ไม่ได้ login Firebase

**แก้**:
```bash
firebase logout
firebase login
```

### Cache ไม่ refresh

**สาเหตุ**: Browser cache HTML เก่า

**แก้**: 
- Hard refresh: `Ctrl+Shift+R` (Win) หรือ `Cmd+Shift+R` (Mac)
- หรือ disable cache ใน DevTools

---

## 🔒 Security Considerations

### ⚠️ ข้อจำกัดสำคัญ

URL **เป็น public** — ใครเดา URL ได้ก็เปิดได้
- แต่ ข้อมูลเก็บใน **browser ของแต่ละคน** (IndexedDB) → ไม่ leak ระหว่าง user

### ถ้าต้องการ Private

ต้อง upgrade เป็น **Level 3: Firebase Hosting + Auth**:
- เพิ่ม Firebase Authentication (Google login เท่านั้น)
- เพิ่ม redirect rule: ถ้าไม่ login → ไปหน้า login
- ใช้ Firestore เก็บข้อมูลแทน IndexedDB (sync ข้ามอุปกรณ์)
- ค่าใช้จ่ายเริ่มที่ ~$25/เดือน (Blaze plan)

ดู `compact.md` Section 12.2 สำหรับรายละเอียด

---

## 🎁 Tips & Best Practices

### 1. ตั้ง URL ง่ายๆ ที่ Firebase Console

- ค่า default: `sales-analytics-302.web.app` (ยาวเกิน)
- ทำ alias: ที่ **Hosting** → Add site → ตั้งชื่อสั้น เช่น `mp-sales.web.app`

### 2. Preview Channel ก่อน Production

```bash
# Deploy ไปที่ preview URL (ไม่กระทบ production)
firebase hosting:channel:deploy preview --expires 7d

# จะได้ URL ชั่วคราว เช่น https://sales-analytics-302--preview-abc123.web.app
```

### 3. Rollback ถ้า deploy ผิด

Firebase Console → Hosting → Release history → กด **Rollback** → ใน 1 click กลับ version เก่าได้

### 4. Monitor errors

- Firebase Console → **Hosting** → ไม่มี error log (เป็น static site)
- ถ้าต้องการดู error: ใช้ browser console (F12) ของ user
- หรือเพิ่ม Sentry / LogRocket (third-party)

### 5. Backup demo data

```bash
# Keep demo_data_302.json in source control
git add demo_data_302.json
git commit -m "Add demo data backup"
```

แต่ถ้าไฟล์ > 100 MB ใช้ Git LFS:
```bash
git lfs track "*.json"
```

---

## 📝 Checklist ก่อน Deploy Production

- [ ] แก้ `.firebaserc` ใส่ project ID จริง
- [ ] ทดสอบ Load Demo ใน local browser (open `index.html` ด้วย local server เช่น `python -m http.server 8000`)
- [ ] ตรวจ `index.html` ไม่มี hard-coded credentials หรือ API key
- [ ] ตรวจ `firebase.json` headers ถูกต้อง
- [ ] ตั้ง Firebase quota alerts (Console → Settings → Usage and billing → Budget alerts)
- [ ] เตรียม Custom domain (ถ้าใช้)
- [ ] เตรียม backup `demo_data_302.json` (กรณีไฟล์เสีย)

---

## 🆘 ถ้าติดปัญหา

1. ดู `firebase --help` หรือ `firebase deploy --help`
2. Firebase Documentation: [https://firebase.google.com/docs/hosting](https://firebase.google.com/docs/hosting)
3. GitHub Actions issue: ดู `Actions` tab ใน repo
4. ถ้าต้องการ Level 3 (Auth + Firestore) ดู `compact.md` Section 12.2

---

**Status**: Ready for production deployment
**Estimated total time**: 20 นาทีครั้งแรก, 1 นาทีครั้งต่อๆ ไป
**Cost**: $0/เดือน (within Spark free tier limits)
