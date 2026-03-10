# PI Confidence Vote 🗳️

Real-time PI Planning confidence voting tool for agile teams.

**v1.0.0 — March 2026**

## Features

- Host creates a session with ART name + team names
- Shareable link / Session ID for participants
- Each participant votes 1–5 independently for each team AND for ART level
- Results update live via Firebase Realtime Database
- Anonymous voting — scores shown without names
- Color-coded averages (red → yellow → green)

## Tech Stack

- Vanilla HTML/CSS/JS (no build step)
- Firebase Realtime Database (real-time sync)
- Hosted on Vercel

## Deploy

### 1. Firebase setup

- Create project at [console.firebase.google.com](https://console.firebase.google.com)
- Enable **Realtime Database** (europe-west1)
- Set rules:
```json
{
  "rules": {
    "sessions": {
      ".read": true,
      ".write": true
    }
  }
}
```

### 2. GitHub

```bash
git init
git add .
git commit -m "feat: initial release v1.0.0"
git remote add origin https://github.com/YOUR_USERNAME/pi-confidence-vote.git
git push -u origin main
```

### 3. Vercel

- Go to [vercel.com](https://vercel.com) → New Project
- Import the GitHub repo
- No build settings needed — deploy as static
- Done ✅

## Usage

1. Open the app URL
2. **Host**: Enter name → Configure ART + teams → Create Session
3. Share the generated link with participants
4. **Participants**: Open link → Enter name → Vote on each card
5. **Host**: Watch live results update in real time
