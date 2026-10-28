# Firebase gallery setup for the Kalarav / GMC Palakkad site

## Included files
- `index.html`: existing public site, now loads published photos from Firestore/Storage when Firebase is configured. Until then, its original placeholder gallery remains.
- `admin.html`: admin sign-in, upload, publish/unpublish, and delete UI.
- `firestore.rules`: public read of published records; writes limited to listed admin UIDs.
- `storage.rules`: public image reads; uploads/updates/deletes limited to listed admin UIDs and image type/size restrictions.

## Important security note
The UID allowlist in the HTML is only a user-interface check. The Firebase Security Rules are the enforcement boundary. Replace `REPLACE_WITH_ADMIN_FIREBASE_AUTH_UID` in BOTH rule files and in `admin.html` with each admin's Firebase Authentication UID. Do not deploy rules with the placeholder still present.

## Setup
1. Create a Firebase project at https://console.firebase.google.com/.
2. Add a Web App and copy its Firebase config.
3. In Firebase Console → Authentication → Sign-in method, enable Email/Password.
4. In Authentication → Users, add each admin account manually. Do not add a public sign-up page.
5. Copy each admin's UID from Authentication → Users. Replace the UID placeholder in `admin.html`, `firestore.rules`, and `storage.rules`.
6. Create a Firestore database.
7. Enable Cloud Storage and choose the bucket. Firebase Storage for new projects may require the Blaze billing plan; review current pricing and set budget alerts before enabling billing.
8. Paste `firestore.rules` into Firestore → Rules and publish. Paste `storage.rules` into Storage → Rules and publish.
9. Replace the `firebaseConfig` placeholder in BOTH `index.html` and `admin.html` with the same config values from Project settings → General → Your apps.
10. Test locally using a local web server (ES modules generally do not work correctly when opening files directly with `file://`). For example, run `python -m http.server 8000` in this folder and open `http://localhost:8000/`.
11. Deploy the site and both HTML files to Firebase Hosting. Example:
   - Install Node.js, then `npm install -g firebase-tools`
   - `firebase login`
   - `firebase init hosting` (select this Firebase project and the directory containing these files)
   - `firebase deploy --only hosting`
12. Open `/admin.html`, sign in as an allowlisted admin, upload a test photo, publish it, then open the public gallery. Verify a non-admin account cannot upload, publish, or delete.

## Notes
- Firebase web config values are not server secrets; Security Rules must be correctly configured.
- Never put a service-account JSON key or Admin SDK private key in either HTML file.
- If you add another admin, update the UID allowlist in the admin UI and both rule files.
- Firestore queries require the `published == true` filter and `createdAt` ordering used in `index.html`; if Firebase asks to create a composite index, follow the link in the console. A simple single-field index is normally sufficient for this query.
- This starter implementation stores the image in Storage and its path/metadata in Firestore. Public reads of images are enabled for the `winner-photos` folder because those photos are intended to be public. Do not upload private or sensitive images to that folder.
