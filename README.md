# 🧨 OPERATION: BLACKOUT — Team Bomb‑Defusal Game

A mobile‑only, multi‑phone party game.

- **4 teams**, each team uses **6 phones** → **6 links per team**.
  - **1 × MAIN phone** → shows a dungeon door, a bomb and a live **5:00** countdown. When it hits `00:00` the bomb "explodes" and shows **FAKE ALARM — use the code on the other phones to find your next station.** A tab opens a **see‑through instructions panel** so the bomb/explosion stays visible behind the text.
  - **5 × WIRE phones** → each shows one wire (**White, Blue, Green, Yellow, Red**). Tap to "cut".
    - Correct order (**White → Blue → Green → Yellow → Red**) → **“TRY AGAIN”** (the bomb can never really be defused 😈).
    - Any wrong order → **⚠️ red warning triangle** and the phone is **locked for 20 seconds**.
    - After the blast, each wire phone reveals its **digit** in a grim, glitchy style:
      **White = 5 · Blue = 6 · Green = 1 · Yellow = 3 · Red = 9**.

All 6 phones on a team stay in sync through **Firebase Realtime Database**. The site itself is a single static `index.html` hosted for free on **GitHub Pages**. You only ever use the **web interface** of GitHub and Firebase — no installs, no command line.

---

## What's in this folder
| File | Purpose |
|------|---------|
| `index.html` | The entire game (main, wires, and organizer console in one file). **You paste your Firebase config into it.** |
| `database.rules.json` | The database security rules to paste into Firebase. |
| `README.md` | This guide. |

---

## STEP 1 — Create the Firebase project & database (web console)
1. Go to **https://console.firebase.google.com** and sign in.
2. **Add project** → give it a name (e.g. `blackout-game`) → you can **disable** Google Analytics → **Create project**.
3. In the left menu open **Build → Realtime Database → Create Database**.
4. Choose a location → select **Start in test mode** → **Enable**.
5. Open the **Rules** tab and replace everything with the contents of `database.rules.json`, then **Publish**:
   ```json
   {
     "rules": {
       "games": { ".read": true, ".write": true }
     }
   }
   ```
   *(Open rules are fine for a one‑off event. See “Locking it down” at the bottom.)*

## STEP 2 — Get your config & paste it into index.html
1. In Firebase, click the ⚙️ **Project settings** → scroll to **Your apps** → click the **`</>` (Web)** icon.
2. Register the app (any nickname, you don't need Hosting) → Firebase shows a **`firebaseConfig`** object. Copy it.
3. Open `index.html` in any text editor. Near the top of the `<script type="module">` block find:
   ```js
   const firebaseConfig = {
     apiKey: "PASTE_YOUR_API_KEY",
     ...
   };
   ```
4. Replace the whole object with **your** config. ✅ Make sure it includes the **`databaseURL`** line (it looks like `https://your-project-default-rtdb.firebaseio.com`). Save.

## STEP 3 — Put the site on GitHub Pages (web interface)
1. Go to **https://github.com** → **New repository** → name it e.g. `blackout-game` → **Public** → **Create repository**.
2. On the repo page click **Add file → Upload files** → drag in **`index.html`** (that single file is all Pages needs) → **Commit changes**.
3. Go to **Settings → Pages**.
4. Under **Build and deployment → Source** choose **Deploy from a branch**, pick branch **`main`** and folder **`/ (root)`** → **Save**.
5. Wait ~1 minute, then refresh. Pages shows your live URL, e.g.
   `https://YOURNAME.github.io/blackout-game/`

## STEP 4 — Generate the phone links (organizer console)
1. Open your live URL **with no extra text** — e.g. `https://YOURNAME.github.io/blackout-game/`.
2. You'll see the **organizer console**: for every team (1–4) it lists all 6 links **and a QR code** for each phone.
3. Assign phones and let each one **scan its QR** (or open its link). Link format:
   ```
   .../index.html?role=main&team=1     ← main phone, team 1
   .../index.html?role=white&team=1    ← white wire, team 1
   .../index.html?role=blue&team=1
   .../index.html?role=green&team=1
   .../index.html?role=yellow&team=1
   .../index.html?role=red&team=1
   ```
   Change `team=1` → `2`, `3`, `4` for the other teams.

## STEP 5 — Run it
1. Give each phone its link/QR. Wire phones show **“STANDBY”** until the game starts.
2. **Opening a team's `main` link starts that team's 5‑minute timer** — all 6 of that team's phones sync instantly.
3. Players tap wires trying to defuse; correct order says *“TRY AGAIN”*, wrong order locks them out 20 s.
4. At `00:00` the main phone detonates (**FAKE ALARM**) and each wire phone reveals its digit → the combined code sends teams to their next station.

---

## Handy tips
- **Reset a team:** on that team's **main** phone tap the small **⟳** button (top‑left) → confirm. It clears the timer and lets you start again.
- **Test quickly:** add `&mins=0.3` to a main link to make the timer ~18 seconds (e.g. `...?role=main&team=1&mins=0.3`). Remove it for the real 5:00.
- **Keep screens awake:** set each phone's auto‑lock to a few minutes, or keep them plugged in.
- **Sound & vibration** turn on after the first tap on each phone (mobile browsers block autoplay until you interact).

## Locking it down (optional, after the event)
Open rules let anyone read/write the `games` node. For a private event that's fine. To restrict later, tighten the Realtime Database **Rules** (e.g. require Firebase Anonymous Auth) and re‑publish.

## Troubleshooting
- **Phones don't sync / “CONFIG REQUIRED” screen** → your `firebaseConfig` in `index.html` still has `PASTE...` placeholders or is missing `databaseURL`. Fix Step 2 and re‑upload the file to GitHub.
- **QR codes blank on the console** → that device had no internet when loading (the QR library loads from a CDN). Reload with a connection.
- **Timer differs slightly between phones** → the app auto‑corrects using Firebase server time; give it a second after opening.
