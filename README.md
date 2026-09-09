# TaskFlow — setup guide

This app is a single static file (`index.html`) that talks directly to
**Firebase Firestore** from the browser using Firebase's JS SDK. That's why
there's no PHP anymore — GitHub Pages only serves static files, and Firestore's
client SDK doesn't need a backend to work.

## What's new

- **Member field** on every task (pick from a fixed list you set in the code).
- **Backdate toggle** when adding a task — flip it on to log a task as if it
  was added on an earlier date/time; leave it off and it uses "now".
- **Two new statuses**: `QA Pass` and `Issue Found`, alongside Pending,
  In Progress, On Hold and Completed. Hold, QA Pass and Issue Found all
  require a short reason when you apply them.
- **Summary tab** — pick a member (or "All members") and a date range, and
  see their work grouped by day, e.g.:
  ```
  Kamal
  Yesterday
    Task 1 — QA Pass  (reason)
    Task 2 — Issue Found  (reason)
  Today
    Task 3 — Pending
    Task 2 — In Progress
  ```
  Each line is one status-history entry, so a task that changed status on two
  different days shows up under both days.
- Everything now saves to Firestore in real time instead of the browser's
  local storage, so the same task list is shared across devices/people.

## 1. Create a Firebase project

1. Go to <https://console.firebase.google.com> and create a project (free
   Spark plan is enough).
2. In the left sidebar: **Build → Firestore Database → Create database**.
   Choose a region close to your team and start in **test mode** for now
   (see the security note below).
3. In the left sidebar: **Project settings → General**, scroll to
   "Your apps", click the **Web (`</>`)** icon, and register an app
   (no need for Firebase Hosting).
4. Firebase will show you a `firebaseConfig` object. Copy it.

## 2. Paste your config into index.html

Open `index.html` and find this block near the top of the `<script type="module">`:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

Replace the placeholder values with the ones Firebase gave you.

Just below it, set your team's names:

```js
const MEMBERS = ["Kamal", "Nimal", "Sunil", "Priya", "Ayesha"];
```

## 3. Security rules (important — you chose "open, no login")

Since the app has no login, anyone with the Firestore rules open can read
and write the `tasks` collection. For a small internal team sharing a link
that's usually an acceptable trade-off for the convenience, but it does mean
**anyone with your Firebase config values could also write to your database**
directly (not just through this app). To keep it simple but scoped to just
this collection, use rules like:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /tasks/{taskId} {
      allow read, write: if true;
    }
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```

Set this under **Firestore Database → Rules** in the Firebase console.
If you ever want real access control, the next step up is turning on
Firebase Authentication and changing `if true` to `if request.auth != null`
— ask me if you'd like that added later.

## 4. Host on GitHub Pages

1. Create a new GitHub repo (or use an existing one) and push `index.html`
   to it.
2. In the repo: **Settings → Pages → Build and deployment → Source**,
   choose **Deploy from a branch**, pick your branch (e.g. `main`) and
   folder `/ (root)`, then **Save**.
3. GitHub will give you a URL like `https://yourname.github.io/reponame/`
   — that's your live app.

## Notes

- Firestore's free tier (Spark plan) is generous for a small team's daily
  task list — you're unlikely to hit any limits.
- Because there's no login, don't put anything sensitive in task notes.
- If tasks don't load, open the browser console (F12) — connection or rules
  errors will show up there, and the app also shows a "Can't connect to
  Firebase" message on screen.
