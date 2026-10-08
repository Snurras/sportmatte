# Joars Sportmatte

A Swedish math game for kids (6–8 years). Six levels from "5 + 5" up to "33 × 12", a Help button that splits hard problems into steps, badges, and a timed Sportmatte mode where the Rymdtroll steals 5 seconds when you answer wrong. Kids log in with a username and password, so their badges and records follow them between computers, and there is a shared scoreboard per level.

The whole game is one file: `public/index.html`.

## Running without Firebase

Leave `FIREBASE_CONFIG = null` in `public/index.html`. The game then skips login and saves progress in the browser on that computer only. Open the file directly or serve the `public` folder.

## Setting up Firebase (separate from your other project)

Use a **new Firebase project** for Sportmatte. Each Firebase project has its own users, database, rules and hosting, so nothing touches your existing project.

1. **Create the project**
   - Go to https://console.firebase.google.com and click **Add project**.
   - Name it `sportmatte` (if the ID is taken, Firebase suggests e.g. `sportmatte-1a2b3`; note the ID).
   - Google Analytics is not needed.

2. **Turn on login**
   - **Build → Authentication → Get started → Sign-in method → Email/Password → Enable** (leave "Email link" off).
   - Kids never see an email address. The game turns the username `Elias` into `elias@sportmatte.app` behind the scenes.

3. **Create the database**
   - **Build → Firestore Database → Create database**.
   - Choose a location in Europe, for example `eur3 (europe-west)`, and start in **production mode**.

4. **Register the web app and paste the config**
   - **Project settings (gear icon) → General → Your apps → Web (`</>`)**. Name it `sportmatte`; you don't need Firebase Hosting setup from this screen.
   - Copy the `firebaseConfig` object it shows and paste it into `public/index.html`:
     ```js
     const FIREBASE_CONFIG = {
       apiKey: "…",
       authDomain: "sportmatte.firebaseapp.com",
       projectId: "sportmatte",
       storageBucket: "…",
       messagingSenderId: "…",
       appId: "…"
     };
     ```
   - This config is not a secret; access is controlled by `firestore.rules`.

5. **Deploy rules and hosting from this repo**
   ```bash
   npm install -g firebase-tools
   firebase login

   firebase deploy --only firestore:rules,hosting
   ```
   The game is then live at `https://<project-id>.web.app`.

6. **Optional: deploy automatically on every push**
   ```bash
   firebase init hosting:github
   ```
   Pick this repo, answer "no" to a build script, and "yes" to deploying on merge to `main`. It creates a GitHub Action and the needed secret for you.

### Alternative: a second database inside your existing project

Firestore allows several named databases in one project. That keeps the data apart, but the **login users would be shared** with your other project. A separate project is cleaner, which is why the steps above use one.

## What is stored

| Path | What | Who can read |
| --- | --- | --- |
| `users/{uid}` | Username, badges, stars, personal records | Only that child |
| `boards/{level}/best/{uid}` | Username and best score on that level | Everyone logged in |

Usernames appear on the shared scoreboard, so let kids pick a nickname rather than their full name.

## Forgotten passwords

Kids have no real email, so Firebase's "reset password" email can't reach them. Write the passwords down somewhere safe. If one is lost, you can set a new password with the Firebase Admin SDK, or delete the user under **Authentication → Users** and let the child create the account again (their badges start over).

## Demo mode

The small "Demoläge för vuxna" link at the bottom of the home page unlocks all levels and shows "Svara rätt" / "Svara fel" buttons in the game. Nothing played in demo mode is saved. The setting is remembered per browser.
