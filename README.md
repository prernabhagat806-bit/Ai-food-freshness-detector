# AI Food Freshness Detector

Log in, take or upload a photo of food, and get an AI verdict: **Fresh**, **Needs Attention** or **Likely Spoiled**.
Photos that are not real food get a clear **Not Food** answer, and photos the AI cannot judge reliably get **Can't Tell**,
instead of made-up results.

Stack: React + TypeScript + Vite + Tailwind (frontend) and an Express server (accounts, AI proxy, API keys stay private).

## Run in VS Code

1. Install [Node.js 20+](https://nodejs.org).
2. Open this folder in VS Code and open the terminal (Ctrl + `).
3. `npm install`
4. Create the env file: `copy .env.example .env` (Windows) or `cp .env.example .env`
5. Open `.env` and fill in `AI_API_KEY` (free Gemini key: https://aistudio.google.com/apikey).
   Optional but recommended: `GOOGLE_VISION_API_KEY` (second food check, see below).
6. `npm run dev`, then open http://localhost:5173
7. Click **Sign up**, create an account, and you land on your profile.

Restart `npm run dev` after every change to `.env`.

## How the food check works (why it is stricter now)

1. **Identify** – the AI names the main subject and must say whether it is *real, edible food* (toys, drawings,
   pictures of food on a screen, plants, objects are rejected). It needs at least 80% confidence.
2. **Second opinion (optional)** – if `GOOGLE_VISION_API_KEY` is set, Google Cloud Vision labels the photo and must
   also see food. If the two disagree, the app refuses instead of guessing.
3. **Freshness** – only for confirmed food. If the condition cannot be judged from the photo (sealed packs, blur,
   poor light) or confidence is below 60%, the result is **Can't Tell**.
4. Temperature is 0, so the same photo gives the same answer.

No image AI can be 100% accurate, and a photo can never show bacteria, smell or storage history.
The result is a visual estimate, not a food-safety guarantee.

## Accounts

- Passwords are hashed with bcrypt (never stored in plain text).
- Login uses an HttpOnly, SameSite cookie, so page scripts cannot read the session.
- Login/sign-up and scanning are rate limited. Scanning requires being logged in.
- Accounts are stored in `data/db.json` (created automatically, git-ignored). Scan history is stored in the
  browser, separately for each account.
- Camera: the browser asks for permission when you tap the camera button. It only works on `http://localhost`
  or `https://` pages.

## Production build

```bash
npm run build
npm start        # site + API on http://localhost:8787
```
When served over https, set `COOKIE_SECURE=true` and a long random `JWT_SECRET` in `.env`.

## Project structure

```
server/index.ts      Express app, /api/analyze
server/auth.ts       sign-up, login, logout, profile, session middleware
server/analyze.ts    food check -> freshness pipeline
server/vision.ts     optional Google Vision second check
server/db.ts         file-based user store
src/pages/           Login, Signup, Home, Scan, History, Tips, Profile
src/context/         AuthContext
src/lib/             api, image helpers, history, verdict styles, types
```
