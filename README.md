# 🧨 OPERATION: BLACKOUT — Team Bomb‑Defusal Game (Firestore edition)

A mobile‑only, multi‑phone party game synced through your existing **Firestore** database (project **“Hostage Game”**) and hosted free on **GitHub Pages**. You only ever use the **web interfaces** of GitHub and Firebase — no installs, no command line.

> **Do I need a new Realtime Database?** ❌ No. Firestore and Realtime Database are different products, but you don't need either a new project *or* a Realtime Database. This build talks to **Firestore directly**, so it uses your existing “Hostage Game” database as‑is.

## The flow
1. **Opening screen** shows **4 team buttons** (2 per row). Team **names come from Firestore** (your `teams` collection).
2. Tapping a team asks for that team's **passcode** — also stored in Firestore.
3. Correct passcode → that team's **6 links + QR codes** (1 main phone + 5 wire phones).
4. Each team's game (timer + cut‑progress) is **completely separate** — one team starting does **not** affect another team's timer.
5. A private **Owner** screen (`?role=owner`) lets only the organizer **reset a single team or reset all**.

### Game mechanics
- **Main phone:** dungeon door + bomb + live **5:00** timer → at `00:00` detonates → **FAKE ALARM — use the code on the other phones to find your next station.** A tab opens a **see‑through instructions panel**.
- **5 wire phones** (White, Blue, Green, Yellow, Red): tap to “cut.”
  - Correct order (**White → Blue → Green → Yellow → Red**) → **“TRY AGAIN.”**
  - Wrong order → **⚠️ red warning triangle**, phone locked **20 s**.
  - After the blast each wire reveals a glitchy digit: **White 5 · Blue 6 · Green 1 · Yellow 3 · Red 9.**

---

## What's in this folder
| File | Purpose |
|------|---------|
| `index.html` | The whole game (team select, passcode, console, main, wires, owner). **Paste your Firebase config into it.** |
| `firestore.rules` | Security rules to paste into the **Firestore → Rules** tab. |
| `README.md` | This guide. |

---

## STEP 1 — Turn on Firestore (if it isn't already)
1. Go to **https://console.firebase.google.com** → open **Hostage Game**.
2. Left menu **Build → Firestore Database**. If it says *Create database*, click it → **Start in test mode** → pick a location → **Enable**. (If Firestore already exists, skip.)

## STEP 2 — Add your teams + passcodes (in Firestore)
Firestore stores data as **collections → documents → fields**. Create a collection called **`teams`**; each document is one team.
1. In **Firestore Database → Data**, click **Start collection** → Collection ID: **`teams`** → **Next**.
2. Add the first team document (use **Auto‑ID** or type an ID like `t1`), with these fields:
   | Field | Type | Example |
   |-------|------|---------|
   | `name` | string | `Red Dragons` |
   | `passcode` | string | `1111` |
   | `order` *(optional)* | number | `1` |
3. **Save**, then **Add document** three more times for your other teams (`Blue Sharks / 2222`, etc.).
4. *(Optional owner lock)* Create a collection **`config`** with a document ID **`owner`** and a field **`passcode`** (string, e.g. `9999`). If you skip this, the owner screen opens without a passcode.

> Field flexibility: the passcode field may be named `passcode`, `password`, `accessCode`, `code`, or `pin`; the name field may be `name`, `teamName`, or `title`. So if your Hostage Game DB already stores teams with, say, an `accessCode`, it'll just work.
>
> Different collection name? If your teams live in a collection that isn't called `teams`, open `index.html` and change `const TEAMS_COLLECTION = "teams";` to your collection's name.

## STEP 3 — Publish the security rules
Open **Firestore Database → Rules**, paste the contents of `firestore.rules`, and **Publish**:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /teams/{doc}  { allow read: if true;  allow write: if false; }
    match /config/{doc} { allow read: if true;  allow write: if false; }
    match /games/{doc}  { allow read, write: if true; }
    match /_clock/{doc} { allow read, write: if true; }
  }
}
```
*(Open `games`/`_clock` is fine for a private one‑off event — see the security note at the bottom.)*

## STEP 4 — Paste your config into index.html
1. Firebase ⚙️ **Project settings → Your apps → `</>` (Web)** → copy the **`firebaseConfig`** object (Firestore doesn't need `databaseURL`).
2. In `index.html`, replace the `const firebaseConfig = {…}` block near the top of the `<script>` with yours. Save.

## STEP 5 — Host on GitHub Pages
1. **https://github.com → New repository** (Public) → **Create**.
2. **Add file → Upload files** → drag in **`index.html`** → **Commit changes**.
3. **Settings → Pages** → Source = **Deploy from a branch** → `main` / `/(root)` → **Save**.
4. Wait ~1 min → your live URL appears, e.g. `https://YOURNAME.github.io/blackout-game/`.

## STEP 6 — Run it
1. **Players:** open the live URL plain → **4 team buttons** → tap team → enter passcode → get the team's **6 links + QR codes**. Hand each phone its link/QR.
2. **Opening a team's MAIN link starts that team's 5:00 timer** — only that team is affected.
3. At `00:00` → **FAKE ALARM** + each wire phone reveals its digit (**W5 · B6 · G1 · Y3 · R9**).
4. **You (owner):** open `https://YOURNAME.github.io/blackout-game/index.html?role=owner` → enter owner passcode (if set) → **Reset** any single team or **RESET ALL TEAMS**.

---

## Link reference (auto‑generated for you)
The team console builds these + QR codes automatically. For reference, with a team document id `t1`:
```
.../index.html?role=main&team=t1
.../index.html?role=white&team=t1
.../index.html?role=blue&team=t1
.../index.html?role=green&team=t1
.../index.html?role=yellow&team=t1
.../index.html?role=red&team=t1
.../index.html?role=owner              ← organizer only (no team)
```

## How the separate timers work
- Each team's live state is one Firestore document at **`games/<teamId>`** holding `started`, `dur`, and `progress`.
- The main phone creates it (with a **server timestamp**) the first time it opens; all 6 phones subscribe to that one document, so their countdown and cut‑sequence stay in lock‑step — and totally independent from any other team's document.

## Handy tips
- **Quick test:** add `&mins=0.3` to a main link (`...?role=main&team=t1&mins=0.3`) for an ~18‑second timer. Remove it for the real 5:00.
- **Keep screens awake:** set phones' auto‑lock to a few minutes or keep them plugged in.
- **Sound & vibration** turn on after the first tap on each phone (mobile browsers block autoplay until you interact).

## Troubleshooting
- **“CONFIG REQUIRED”** → your `firebaseConfig` still has `PASTE…` placeholders. Fix Step 4 and re‑upload.
- **“No teams found”** → make sure the collection is named `teams` (or update `TEAMS_COLLECTION`) and each doc has `name` + `passcode`; confirm the Rules allow read (Step 3).
- **Wrong passcode always** → the value in the team doc doesn't match what you typed (compared as text; e.g. `007` must be typed `007`).
- **Timer slightly off between phones** → the app estimates server time on load; give it a second after opening.
- **Owner reset fails** → make sure `games` has `allow read, write: if true;` in the rules.

## Security note
These rules make `teams`/`config` publicly **readable** (so the app can show names and check passcodes on the phone) and `games`/`_clock` publicly writable. That's fine for a private event. To harden afterward, move passcode checks behind Firebase Auth / Cloud Functions and tighten the rules.
