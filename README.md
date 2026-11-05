# Kit Ledger

Camera kit register, rentals log and payback tracker for Jono Le Grice.

- Live app: https://jonolegrice.github.io/kit-ledger/
- Hosting: GitHub Pages (this repo, `main` branch, root folder)
- Data: Firebase project `kit-ledger` (Firestore + email sign-in). Each user's data lives under `users/{uid}`; see `firestore.rules`.
- No kit data is stored in this repo.

Files: `index.html` (the whole app), `config.js` (Firebase project settings, not secret), `sw.js` (offline support), `manifest.webmanifest` + icons (install as an app).
