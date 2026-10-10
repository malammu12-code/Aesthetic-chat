# InterConnect Chat App

A simple, pastel-themed chat application with email/password authentication, friend requests, real-time messaging, and photo sharing.

---

## Overview

InterConnect is a lightweight social chat app. It supports:

- User signup and login with email and password
- A discover page to find other users and send friend requests
- A friends page to accept/decline requests and view current friends
- A real-time 1-on-1 discussion page
- Image sharing inside conversations

The project is intentionally kept simple and easy to read, with a soft pastel UI (purple/pink/blue gradients, rounded pill inputs, and glass-card panels).

---

## Project Structure

### React + Lovable Cloud (default in this repo)

The main application in this repository is built with:

- React 18 + TypeScript
- Vite 5 + Tailwind CSS v3
- Lovable Cloud (backend, auth, database, storage)
- Supabase client libraries under the hood

Key files:

- `src/pages/Auth.tsx` — Login and signup page
- `src/pages/AppHome.tsx` — Tabs for Chats, Friends, and Discover
- `src/pages/Chat.tsx` — Real-time 1-on-1 conversation with image upload
- `src/hooks/useAuth.tsx` — Authentication context
- `src/index.css` — Pastel design tokens, gradients, and glass-card components
- `src/integrations/supabase/` — Auto-generated Supabase client and types

### Vanilla HTML/CSS/JS + Node.js + Socket.IO (standalone version)

The same app can also be built without React or a managed backend. The vanilla version uses:

- Static HTML files for each page
- Plain CSS for the same pastel UI
- Vanilla JavaScript for frontend logic
- Node.js + Express + Socket.IO for the server
- SQLite for the database
- Cookie-based sessions shared with WebSocket connections
- Local file storage for uploaded images

This version is useful for learning, self-hosting, or keeping everything in one codebase without external services.

---

## Vanilla Version File Map

```
interconnect-node/
├── server.js          # Express + Socket.IO + SQLite backend
├── public/
│   ├── index.html     # Login / Signup
│   ├── app.html       # Chats / Friends / Discover tabs
│   ├── chat.html      # Real-time discussion page
│   ├── styles.css     # Pastel design system
│   └── app.js         # Shared frontend helpers (optional)
├── uploads/           # Image uploads (auto-created)
├── chat.db            # SQLite database (auto-created)
└── README.md          # Run instructions
```

### Backend responsibilities

`server.js` handles:

- Static file serving
- JSON API endpoints for auth, profiles, friends, and messages
- SQLite database creation and queries
- Socket.IO rooms for real-time messaging
- Image upload endpoint
- Cookie-based session middleware so WebSocket connections know the current user

### API endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| POST | `/api/auth/signup` | Create an account |
| POST | `/api/auth/login` | Log in and start a session |
| POST | `/api/auth/logout` | End the session |
| GET | `/api/me` | Get current user profile |
| GET | `/api/users` | List all user profiles (for discover) |
| GET | `/api/friends` | List friends and pending requests |
| POST | `/api/friends/request` | Send a friend request |
| POST | `/api/friends/accept/:id` | Accept a friend request |
| POST | `/api/friends/decline/:id` | Decline / remove a friend request |
| GET | `/api/messages/:friendId` | Fetch chat history |
| POST | `/api/upload` | Upload an image |

### Socket.IO events

| Event | Direction | Purpose |
|-------|-----------|---------|
| `message` | client → server | Send a new message |
| `message` | server → client | Receive a new message in real time |
| `join` | client → server | Join a room for a friend conversation |

### Database schema (SQLite)

```sql
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  display_name TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE friendships (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  requester_id INTEGER NOT NULL,
  addressee_id INTEGER NOT NULL,
  status TEXT NOT NULL CHECK(status IN ('pending', 'accepted')),
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(requester_id, addressee_id)
);

CREATE TABLE messages (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  sender_id INTEGER NOT NULL,
  recipient_id INTEGER NOT NULL,
  content TEXT,
  image_url TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### Security notes for the vanilla version

- Passwords must be hashed (use `bcrypt`)
- Add server-side validation for all inputs
- Rate-limit login and signup endpoints
- Validate the session cookie on every API and WebSocket request
- Serve uploaded images only to authenticated users who are part of the conversation
- Use HTTPS in production
- Store the session secret in an environment variable, never in the code

---

## Running the Vanilla Version

```bash
# 1. Extract the standalone project
unzip interconnect-node.zip
cd interconnect-node

# 2. Install dependencies
npm install

# 3. Start the server
npm start

# 4. Open the app
http://localhost:3000
```

The SQLite database (`chat.db`) and `uploads/` folder are created automatically on first run.

---

## Design Tokens

The UI uses a consistent pastel palette.

CSS variables (from `src/index.css`):

```css
:root {
  --background: 280 40% 97%;
  --foreground: 350 35% 25%;
  --primary: 220 70% 70%;
  --secondary: 280 50% 90%;
  --accent: 10 80% 80%;
  --destructive: 0 75% 65%;
  --border: 280 30% 88%;
  --radius: 1.5rem;

  --gradient-bg: linear-gradient(160deg, hsl(200 60% 85%) 0%, hsl(280 50% 88%) 50%, hsl(330 60% 88%) 100%);
  --gradient-button: linear-gradient(90deg, hsl(10 80% 80%) 0%, hsl(280 50% 80%) 50%, hsl(220 70% 75%) 100%);
  --gradient-button-warm: linear-gradient(90deg, hsl(0 75% 80%) 0%, hsl(30 90% 78%) 100%);
  --gradient-text: linear-gradient(90deg, hsl(330 70% 75%), hsl(280 60% 70%), hsl(220 70% 70%));
  --gradient-card: linear-gradient(180deg, hsl(0 0% 100% / 0.8) 0%, hsl(0 0% 100% / 0.5) 100%);

  --shadow-soft: 0 8px 24px -8px hsl(280 40% 60% / 0.25);
  --shadow-pill: 0 4px 16px -4px hsl(280 40% 60% / 0.2);
}
```

Reusable classes:

- `.glass-card` — frosted-glass panel with rounded corners
- `.pill-input` — rounded search/input fields
- `.pill-button` — primary gradient button
- `.pill-button-warm` — warm gradient button
- `.gradient-text` — text with the brand gradient

---

## Feature Comparison

| Feature | React + Lovable Cloud | Vanilla + Node/Socket.IO |
|--------|------------------------|---------------------------|
| Frontend | React 18 + TypeScript | Plain HTML/CSS/JS |
| Backend | Lovable Cloud (managed) | Node.js + Express |
| Real-time | Supabase Realtime | Socket.IO |
| Database | Postgres (managed) | SQLite |
| Auth | Managed email/password | Cookie sessions + bcrypt |
| Image storage | Managed storage bucket | Local `uploads/` folder |
| Hosting | Lovable / Vercel | Any Node host (Render, Railway, Fly.io) |
| Boilerplate | Lower | Higher |
| Maintenance | Platform handles backend | You maintain everything |

---

## Next Steps / Ideas

- Group chats
- Message reactions
- Push notifications
- Online/typing indicators
- Message read receipts
- Voice messages
- Dark mode toggle
- Mobile PWA install

---

## License

This project is for educational and personal use. Customize freely.
