<div align="center">

# 💼 DevPortfolio

**A MERN-stack social network for developers — build a profile, list your experience, education and skills, pull in your GitHub repos, and share posts in a community feed.**

![Node](https://img.shields.io/badge/Node.js-Express_4-339933?logo=node.js&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-Mongoose_5-47A248?logo=mongodb&logoColor=white)
![React](https://img.shields.io/badge/React-17-61DAFB?logo=react&logoColor=black)
![Redux](https://img.shields.io/badge/Redux-Thunk-764ABC?logo=redux&logoColor=white)
![JWT](https://img.shields.io/badge/Auth-JWT-000000?logo=jsonwebtokens&logoColor=white)
![License](https://img.shields.io/badge/license-ISC-blue)

</div>

---

## Overview

DevPortfolio is a full-stack MERN application where developers can create an account, build out a rich professional profile, and interact with other developers through a post feed. It pairs an Express + MongoDB REST API with a React + Redux single-page frontend, using stateless JWT authentication.

**Highlights**

- 🔐 **Auth** — register and log in with JWT (token sent via the `x-auth-token` header), passwords hashed with bcrypt.
- 👤 **Profiles** — company, website, location, status, skills, bio, and social links, plus repeatable **experience** and **education** entries.
- 🐙 **GitHub integration** — fetches a user's latest public repositories via the GitHub API.
- 📝 **Posts feed** — create posts, like/unlike, and comment, with ownership checks on delete.
- 🌐 **Public directory** — browse all developer profiles and view any individual profile.
- 🖼️ **Gravatar** — avatars are auto-generated from the account email.

---

## Tech Stack

| Layer | Technology |
| --- | --- |
| **Frontend** | React 17 (Create React App), Redux + redux-thunk, React Router 5, Axios, Moment / react-moment |
| **Backend** | Node.js, Express 4 |
| **Database** | MongoDB with Mongoose 5 |
| **Auth** | JSON Web Tokens (`x-auth-token` header) + bcryptjs |
| **Validation** | express-validator |
| **Integrations** | GitHub REST API (repos), Gravatar (avatars) |
| **Tooling** | nodemon, concurrently |

---

## How It Works

```
React (CRA :3000)  ──axios──►  Express API (:5000)  ──Mongoose──►  MongoDB
      │  Redux store                  │
      │  (alert, auth, profile)       ├── JWT verified from x-auth-token header
      └── token in localStorage       └── GitHub API for repo data
```

The React dev server proxies API calls to the backend on port `5000` (see `client/package.json` → `proxy`). On the client, the JWT is stored in `localStorage` and attached to every request. In production, Express serves the built React app from `client/build`.

---

## Project Structure

```
portfolio/
├── server.js                 # Express entry — connects DB, mounts /api routes
├── config/
│   ├── db.js                 # Mongoose connection
│   └── default.json          # ⚠️ you create this (gitignored) — see Configuration
├── middleware/
│   └── auth.js               # Verifies the x-auth-token JWT
├── models/
│   ├── User.js               # name, email, password, avatar
│   ├── Profile.js            # profile + experience[] + education[] + social
│   └── Post.js               # text, likes[], comments[]
├── routes/api/
│   ├── users.js              # register
│   ├── auth.js               # login + load current user
│   ├── profile.js            # profile CRUD, experience, education, github repos
│   └── posts.js              # posts, likes, comments
│
└── client/                   # React + Redux frontend
    ├── public/
    └── src/
        ├── App.js            # Router + routes (public + PrivateRoute)
        ├── store.js          # Redux store
        ├── reducers/         # alert, register (auth), profile
        └── components/
            ├── actions/      # Redux thunks + action types
            ├── auth/         # Login, Register
            ├── layouts/      # Navbar, Landing, Alert, Spinner
            ├── dashboard/    # Dashboard, Experience, Education
            ├── profile-forms/# Create/Edit profile, Add experience/education
            ├── profile/      # Single profile view (about, github, etc.)
            ├── Profiles/      # Developer directory
            └── routing/      # PrivateRoute guard
```

---

## Getting Started

### Prerequisites

- Node.js and npm
- A MongoDB database (local `mongod` or a MongoDB Atlas cluster)
- *(Optional)* A GitHub OAuth app, to raise the GitHub API rate limit for repo fetching

### 1. Clone and install

```bash
git clone https://github.com/erysimum/portfolio.git
cd portfolio

npm install                 # backend dependencies
npm install --prefix client # frontend dependencies
```

### 2. Configure (see Configuration below) — create `config/default.json`.

### 3. Run in development

```bash
npm run dev
```

This uses `concurrently` to start both servers at once:

| Service | URL |
| --- | --- |
| React client | http://localhost:3000 |
| Express API | http://localhost:5000 |

---

## Configuration

This project uses the [`config`](https://www.npmjs.com/package/config) package. It reads `config/default.json` always, and merges `config/production.json` when `NODE_ENV=production`.

Create **`config/default.json`** locally (this file should be **gitignored and never committed**):

```json
{
  "mongoURI": "your-mongodb-connection-string",
  "jwtSecret": "a-long-random-secret",
  "githubClientId": "your-github-oauth-client-id",
  "githubSecret": "your-github-oauth-client-secret"
}
```

> 🔒 **Security:** never commit real credentials. Add `config/default.json` and `config/production.json` to `.gitignore`. If secrets were ever committed, rotate them and purge them from git history — removing them in a later commit is not enough, since they remain in the repo's history.

---

## API Reference

Base URL: `http://localhost:5000`. Protected routes require a valid JWT in the `x-auth-token` header.

### Users & Auth

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| `POST` | `/api/users` | Public | Register a new user → returns JWT |
| `POST` | `/api/auth` | Public | Log in → returns JWT |
| `GET` | `/api/auth` | Private | Get the current authenticated user |

### Profiles

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| `GET` | `/api/profile/me` | Private | Get the current user's profile |
| `POST` | `/api/profile` | Private | Create or update a profile |
| `GET` | `/api/profile` | Public | List all profiles |
| `GET` | `/api/profile/user/:user_id` | Public | Get a profile by user ID |
| `DELETE` | `/api/profile` | Private | Delete the profile **and** the user account |
| `PUT` | `/api/profile/experience` | Private | Add an experience entry |
| `DELETE` | `/api/profile/experience/:experienceid` | Private | Remove an experience entry |
| `PUT` | `/api/profile/education` | Private | Add an education entry |
| `DELETE` | `/api/profile/education/:educationid` | Private | Remove an education entry |
| `GET` | `/api/profile/github/:username` | Public | Fetch a user's latest GitHub repos |

### Posts

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| `POST` | `/api/posts` | Private | Create a post |
| `GET` | `/api/posts` | Private | Get all posts (newest first) |
| `GET` | `/api/posts/:id` | Private | Get a post by ID |
| `DELETE` | `/api/posts/:id` | Private | Delete own post |
| `PUT` | `/api/posts/like/:id` | Private | Like a post |
| `PUT` | `/api/posts/unlike/:id` | Private | Unlike a post |
| `POST` | `/api/posts/comment/:id` | Private | Comment on a post |
| `DELETE` | `/api/posts/comment/:id/:comment_id` | Private | Delete own comment |

---

## Available Scripts

Run from the project root:

| Script | Description |
| --- | --- |
| `npm run server` | Start the API only (nodemon) |
| `npm run client` | Start the React client only |
| `npm run dev` | Start API + client together (concurrently) |
| `npm start` | Start the API in production mode |

---

## Deployment

The backend is deployment-ready for a Node host. When `NODE_ENV=production`, Express serves the React build from `client/build`, so the whole app runs from a single service. A `heroku-postbuild` script is included that installs client dependencies and builds the frontend during deploy. Supply `mongoURI`, `jwtSecret`, and the GitHub keys through the host's environment / config — not a committed file.

---

## Roadmap / Possible Improvements

- [ ] Move the `request` dependency (deprecated) to `axios` or `node-fetch` for GitHub calls
- [ ] Re-enable the `current` flag on experience/education entries
- [ ] Add pagination to the posts feed and profile directory
- [ ] Add automated tests for the API routes
- [ ] Migrate auth token storage to httpOnly cookies

---

## License

ISC.
