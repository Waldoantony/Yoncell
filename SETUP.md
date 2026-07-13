# Yoncell — Admin Dashboard Setup

This gives you an admin page (`admin.html`) where you log in, add a phone with
a photo and price, and it shows up on `index.html` instantly — no GitHub
uploads, no code editing.

It runs on **Firebase** (free tier is plenty for this). One-time setup below,
then it's all point-and-click from your phone or computer.

---

## 1. Create your Firebase project

1. Go to https://console.firebase.google.com and click **Add project**.
2. Name it (e.g. "Yoncell"), skip Google Analytics (not needed), click **Create project**.

## 2. Register a web app

1. In your new project, click the **`</>`** (web) icon on the project overview page.
2. Give it a nickname (e.g. "Yoncell Web"), click **Register app**.
3. You'll see a `firebaseConfig` object like this:

```js
const firebaseConfig = {
  apiKey: "AIzaSy...",
  authDomain: "yoncell-xxxxx.firebaseapp.com",
  projectId: "yoncell-xxxxx",
  storageBucket: "yoncell-xxxxx.appspot.com",
  messagingSenderId: "...",
  appId: "..."
};
```

4. Copy those values into **`firebase-config.js`** (the file I made — just replace the placeholder text for each field). Both `index.html` and `admin.html` read from this one file.

## 3. Turn on Firestore (the database)

1. In the left sidebar: **Build → Firestore Database → Create database**.
2. Choose a region close to you (e.g. `us-east1`), start in **production mode**.
3. Once created, go to the **Rules** tab and paste this, then **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /products/{productId} {
      allow read: if true;           // anyone can see products (your storefront)
      allow write: if request.auth != null;  // only logged-in admin can add/edit/delete
    }
  }
}
```

## 4. Turn on Storage (for photos)

1. Left sidebar: **Build → Storage → Get started**. Accept the defaults.
2. Go to the **Rules** tab and paste this, then **Publish**:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /products/{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

## 5. Turn on Authentication (so only you can log into the admin page)

1. Left sidebar: **Build → Authentication → Get started**.
2. Click the **Sign-in method** tab → enable **Email/Password**.
3. Go to the **Users** tab → **Add user** → enter your own email + a password.
   This is the login you'll use on `admin.html`. There's no public signup —
   only accounts you add here can log in.

## 6. Upload the files

Upload these three files to your GitHub repo, next to your existing `index.html`:
- `index.html` (replaces your current one)
- `admin.html` (new — this is your dashboard)
- `firebase-config.js` (new — your keys from step 2)

Once live, go to `https://waldoantony.github.io/Yoncell/admin.html`, log in
with the account from step 5, and add your first phone.

---

## Day-to-day use

- Bookmark `admin.html` on your phone.
- Log in → fill the form → choose a photo → **Ajouter le produit**.
- It appears on the live site within a second or two, on every visitor's screen.
- Edit price or stock status any time with the ✎ button; delete with 🗑.
- Check "Mettre en Produits Populaires" to feature it in the homepage carousel.

## Notes

- Your existing Unsplash-linked products (Galaxy S24 Ultra, Pixel 8 Pro, etc.)
  are **not** automatically imported — the storefront now reads only from
  Firestore. Re-add them once through the admin page (with real photos if you
  have them) and they'll appear.
- Deleting a photo from the admin page doesn't delete the old file from
  Storage — it just detaches it from the product. If you re-upload a lot,
  you can periodically clean up unused files in Firebase Console → Storage.
- Everything is free at this scale (Firebase's free tier covers far more
  reads/writes/storage than a shop like this will use).
