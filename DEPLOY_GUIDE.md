# JPhone Admin Panel – Hosting Guide (Vercel, mobile se bhi)

## Pehle Firebase Console (project jphone-8e336)
1. Authentication > Sign-in method > **Google** ko Enable karein.
2. Project Settings > Your apps > **Add app > Web (</>)**. Wahan se config values milengi.
3. Firestore Database > Rules: `firestore.rules` ka content paste karke Publish karein
   (ye Android app wale rules hi hain – isse app ka kuch nahi bigarta).
4. Firestore mein collection `admins` > document ID = **aapka Firebase UID** (Authentication > Users se copy karein)
   fields: `role` = super_admin, `active` = true, `email` = aapki email.
   (Rules mein `beyondhorizons121@gmail.com` pehle se admin hai, us account se bina doc ke bhi login ho jata hai.)

## Vercel par deploy
1. Panel ka zip extract karke GitHub repo mein upload karein (`.env` upload na karein).
2. vercel.com > Add New > Project > repo import.
3. Framework: **Vite**, Build: `npm run build`, Output: `dist`.
4. Environment Variables:
   VITE_FIREBASE_API_KEY, VITE_FIREBASE_AUTH_DOMAIN, VITE_FIREBASE_PROJECT_ID (jphone-8e336),
   VITE_FIREBASE_STORAGE_BUCKET, VITE_FIREBASE_MESSAGING_SENDER_ID, VITE_FIREBASE_APP_ID
5. Deploy. Phir Firebase > Authentication > Settings > **Authorized domains** mein `xxxx.vercel.app` add karein.

## Android app ke saath mapping
| Panel | Firestore collection |
|---|---|
| Users | users (status, isPremium, premiumPlanId, premiumExpiryDate, adminNotes) |
| Plans | premium_plans |
| Premium grant | users + subscriptions/{uid} |
| Notifications | admin_notifications (status QUEUED) |
| Settings / Ads / Update | app_config/general, app_config/advertising, app_config/update_announcement |
| Audit logs | admin_audit_logs |
| Secret keys (bypass key, AppLovin) | admin_reports/panel_private_settings (sirf admin) |

## Zaroor jaanein
- Notifications abhi sirf **queue** hoti hain. Asli FCM push bhejne ke liye Cloud Function chahiye jo
  `admin_notifications` ko parh kar topic par send kare. Wo is zip mein nahi hai.
- Android app abhi `app_config/update_announcement` ko Firestore se nahi parhti; panel likhta hai, app side baad mein jodna hoga.
- Users ki suspension reason Firestore rules ki wajah se user doc mein save nahi hoti, sirf audit log mein jati hai.

## Free hosting (cPanel / FTP) par upload
1. GitHub repo mein Settings > Secrets and variables > Actions: 6 secrets banayein
   (VITE_FIREBASE_API_KEY, _AUTH_DOMAIN, _PROJECT_ID, _STORAGE_BUCKET, _MESSAGING_SENDER_ID, _APP_ID).
2. Actions > **Build Admin Panel** > Run workflow. Khatam hone par artifact `admin-panel-dist` download karein.
3. Hosting File Manager > `public_html` (ya htdocs) mein zip upload karke **Extract** karein.
   `index.html` seedha public_html mein honi chahiye.
4. Firebase > Authentication > Settings > Authorized domains mein apna hosting domain add karein.
