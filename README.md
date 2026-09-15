# The Trade Post — deploy guide

A static site (plain HTML/CSS/JS, no build step) for a local skill-barter board: post offers and needs, match on skills, message, and video-call — all backed by a free Firebase database so it works for real, for anyone who visits the live link.

## 1. Create a free Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) and sign in with any Google account.
2. Click **Add project**, give it a name (e.g. "trade-post"), and finish the wizard — you can skip Google Analytics.
3. In the left sidebar: **Build → Firestore Database → Create database**. Pick a region close to you, and choose **Start in test mode**.
4. Once it's created, open the **Rules** tab and replace the contents with:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /kv/{docId} {
         allow read, write: if true;
       }
     }
   }
   ```

   Click **Publish**.

   **Heads up:** this makes the database fully open — anyone who has your site's URL can read or write any data in it (this is what lets two strangers' browsers talk to each other with no server in between). That's fine for testing with friends or a small pilot group, but don't put sensitive information into listings or messages, and tighten these rules before any wider public launch.

5. Back in the project overview page, click the **`</>`** (web) icon to register a new web app. Name it anything, and leave "Firebase Hosting" unchecked. Firebase will show you a `firebaseConfig` object with real values — keep that page open, you'll need it next.

## 2. Add your config to the site

Open `index.html` and find this block near the top of the `<script>` tag:

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

Replace it with the real values from step 1.5, then save the file. (If you skip this, the site will show a setup reminder instead of loading.)

## 3. Put it on GitHub

1. Create a new repository at [github.com/new](https://github.com/new) (public or private both work fine).
2. Add `index.html`, `README.md`, and `netlify.toml` to it. Easiest way: on the repo's page, click **Add file → Upload files** and drag them in. Or, from the command line:

   ```
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
   git push -u origin main
   ```

## 4. Deploy on Netlify

1. Go to [app.netlify.com](https://app.netlify.com) and sign in (using your GitHub account is easiest).
2. Click **Add new site → Import an existing project → GitHub**, and pick your repository.
3. Leave the build command blank; the publish directory should be `.` (the included `netlify.toml` sets this automatically).
4. Click **Deploy**. Netlify gives you a live URL in a minute or two (something like `random-name-123.netlify.app`, which you can rename in site settings).

## 5. Test it

Open the Netlify URL in two separate browser windows — or send it to a friend on a different device — sign in as two different people, fill in some skills on each profile, and try posting a listing, checking Matches, messaging, and starting a video call.

## Notes and limitations

- **Not real Google sign-in.** People just type a name and email; there's no OAuth or password. Adding real "Sign in with Google" would need a proper backend and OAuth client registration — a bigger step up from this prototype.
- **Video calls have no relay server.** They use free WebRTC with only a public STUN server, no paid TURN server. Calls should connect fine on normal home or mobile networks, but may fail to connect if one person is behind a strict corporate or school firewall.
- **The database is wide open** (per the rules above) so that two browsers can find each other with no backend server. Good for a private pilot with people you trust; revisit before a public launch — Firebase's docs on [securing Firestore](https://firebase.google.com/docs/firestore/security/get-started) are the right next stop when you're ready.
