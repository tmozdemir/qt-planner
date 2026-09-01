# Meet up — setup guide

A one-time setup, then it's just `index.html` on GitHub Pages from here on.

## 1. Firebase project

1. Create a project at console.firebase.google.com (free "Spark" plan is fine
   to start — Functions calls at this volume will stay in the free tier,
   though Functions itself requires the "Blaze" pay-as-you-go plan to be
   *enabled*, even if you never get billed).
2. Enable **Firestore Database** and **Authentication** (no sign-in
   providers need to be turned on — custom tokens don't require one).
3. Project settings → your web app → copy the config object into
   `CONFIG.firebase` in `index.html`.
4. Deploy the security rules: `firebase deploy --only firestore:rules`
   (after `firebase init firestore` if you haven't already).

## 2. Passcode

Pick a passcode, then set its hash as a Functions secret so the raw
passcode never lives in your code:

```
echo -n "your-passcode" | shasum -a 256
firebase functions:secrets:set PASSCODE_HASH
# paste the hash when prompted
```

Update `functions/index.js` to read the secret (see Firebase's secrets docs
for wiring `PASSCODE_HASH` into the function's `secrets: [...]` option).

## 3. Google Calendar

1. In Google Cloud Console, create an **OAuth 2.0 Client ID** of type
   "Web application".
2. Under Authorized JavaScript origins, add your GitHub Pages URL
   (e.g. `https://your-username.github.io`).
3. Enable the **Google Calendar API** for the project.
4. Copy the client ID into `CONFIG.googleClientId` in `index.html`.
5. Repeat sign-in for each Google account you want connected — each one
   is a separate "Connect Google" click, since OAuth is per-account.

## 4. iCloud

Nothing to set up in advance — each person generates their own
app-specific password at appleid.apple.com when they first click
"Connect iCloud" in the page. It's scoped and revocable, never your
main Apple ID password.

The password itself is never stored — only the Apple ID email is
remembered locally, so the page can prompt "Sync iCloud" for that
account later. Each sync means re-entering the password. This trades
a bit of convenience for not keeping a plain-text credential sitting
in the browser; worth revisiting if it gets annoying in practice.

## 5. Deploy the functions

```
cd functions
npm install
firebase deploy --only functions
```

The deploy output lists a separate URL for each function (2nd-gen
Cloud Functions don't share one base URL) — copy `passcodeLogin`'s URL
into `CONFIG.functions.passcodeLoginUrl` and `icloudEvents`'s URL into
`CONFIG.functions.icloudEventsUrl` in `index.html`. Also update
`ALLOWED_ORIGIN` in `functions/index.js` to your GitHub Pages URL, then
redeploy.

## 6. Publish

Push `index.html` to your GitHub Pages repo as usual. Use an
unguessable page path (not `/schedule.html`) and it's already tagged
`noindex` so it won't turn up in search results — the passcode is the
real gate, those two are just extra quiet.

## Adjusting things later

- **Bucket hours** (morning/afternoon/night) — edit `CONFIG.buckets`.
- **How far ahead it looks** — `CONFIG.monthsAhead`.
- **Retagging a recurring event** — delete its entry from
  `people/{you}/rules` in the Firestore console and it'll ask again
  next sync.
