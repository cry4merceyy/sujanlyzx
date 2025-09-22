# Discord Ticket Bot (JavaScript)

Lightweight, production-ready Discord **ticketing** system written in JavaScript/Node.js.
Built and maintained by **Sujan (aka *lyzx*)**.

> Create, manage, and archive support tickets using slash commands with proper permissions, transcripts, and customizable workflows.

---

## ✨ Features

* **Slash commands**: `/ticket open`, `/ticket close`, `/ticket claim`, `/ticket add`, `/ticket remove`, `/ticket transcript`
* **Role-based access** for staff & admins
* **Category auto-routing** (e.g., Billing, Appeals, Tech)
* **Private ticket channels** with synced overwrites
* **Transcripts** (HTML or text)
* **Configurable limits** (one ticket per user, cooldowns, etc.)
* **Persistent storage** (JSON/SQLite — pick your adapter)
* **Docker-ready**, one-click deploy to common hosts (Railway/Render)
* **Type-safe dev hints** via JSDoc

---

## 🧱 Tech Stack

* Node.js (LTS 18+ recommended)
* discord.js v14+
* dotenv
* (Optional) sqlite3 / better-sqlite3 or lowdb
* pino (logging)

---

## 🚀 Quick Start

### 1) Create your Discord Application / Bot

1. Go to the **Discord Developer Portal** → Applications → **New Application**
2. Add a **Bot** user, copy the **Bot Token**
3. Enable **Privileged Gateway Intents** you need:

   * Server Members Intent (if you add/remove users to ticket channels)
   * Message Content Intent (only if you read message content—usually not required)
4. **OAuth2 URL**: scopes `bot applications.commands`
   Recommended base permissions:

   * `Manage Channels`, `Manage Roles`, `Read Messages/View Channels`, `Send Messages`, `Manage Messages`, `Attach Files`, `Read Message History`, `Add Reactions`

> ⚠️ Never commit your token. Keep it in `.env`.

### 2) Clone & Install

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
npm install
```

### 3) Configure Environment

Create a `.env` file:

```env
DISCORD_TOKEN=your-bot-token-here
DISCORD_CLIENT_ID=your-application-id
GUILD_ID=optional-single-guild-id-for-dev
# Storage choices (pick one)
DB_PROVIDER=file               # file | sqlite
DB_FILE=./data/tickets.json    # used if DB_PROVIDER=file
SQLITE_FILE=./data/tickets.db  # used if DB_PROVIDER=sqlite
# Bot behavior
STAFF_ROLE_ID=123456789012345678
TICKET_CATEGORY_ID=123456789012345678        # default category for new tickets
MAX_OPEN_TICKETS_PER_USER=1
TICKET_COOLDOWN_SECONDS=120
TRANSCRIPTS_DIR=./transcripts
```

### 4) Register Slash Commands

(Do this whenever you add/rename commands.)

```bash
npm run deploy:commands
```

### 5) Run the Bot

```bash
npm run dev       # nodemon
# or
npm start         # node .
```

---

## 📦 Suggested `package.json` Scripts

```json
{
  "scripts": {
    "start": "node .",
    "dev": "nodemon --watch src --ext js,json --exec node .",
    "deploy:commands": "node scripts/deploy-commands.js",
    "lint": "eslint .",
    "format": "prettier -w ."
  }
}
```

---

## 🗂️ Project Structure (recommended)

```
.
├─ src/
│  ├─ index.js                 # entrypoint (login, client, handlers)
│  ├─ commands/
│  │  └─ ticket/
│  │     ├─ open.js
│  │     ├─ close.js
│  │     ├─ claim.js
│  │     ├─ add.js
│  │     ├─ remove.js
│  │     └─ transcript.js
│  ├─ lib/
│  │  ├─ tickets.js            # create/lookup helpers
│  │  ├─ storage.js            # file/sqlite adapters
│  │  └─ perms.js              # permission checks
│  ├─ config.js
│  └─ logger.js
├─ scripts/
│  └─ deploy-commands.js
├─ data/
│  ├─ tickets.json
│  └─ tickets.db
├─ transcripts/
├─ .env.example
└─ README.md
```

---

## 🔧 Configuration Notes

* **STAFF\_ROLE\_ID**: Members with this role can view/respond to tickets by default.
* **TICKET\_CATEGORY\_ID**: New ticket channels are created under this category.
  You can implement **routing** by topic (e.g., `/ticket open topic:billing`) to different category IDs.
* **Permissions**: The bot needs permission to **create channels** and manage overwrites.
* **Limits/Cooldowns**: Prevent spam by capping open tickets and adding cooldowns.

---

## 🧑‍💻 Commands (default)

* `/ticket open [topic] [details]`
  Creates a private channel only visible to the user + staff.
* `/ticket close [reason]`
  Locks the ticket, optionally posts a transcript and archives it.
* `/ticket claim`
  Marks current ticket as being handled by a staff member.
* `/ticket add @member` / `/ticket remove @member`
  Grants/revokes access to the ticket channel.
* `/ticket transcript [public:boolean]`
  Generates transcript (HTML or text) and stores it in `/transcripts`.

> You can add `/ticket rename`, `/ticket move`, `/ticket note` as needed.

---

## 🗄️ Storage

Pick one:

* **File** (default): simple JSON at `DB_FILE`
* **SQLite**: set `DB_PROVIDER=sqlite` and provide `SQLITE_FILE`
  Useful for concurrency and larger servers.

Abstraction lives in `src/lib/storage.js` — swap providers without touching commands.

---

## 🐳 Docker (optional)

**Dockerfile**

```dockerfile
FROM node:20-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
CMD ["node", "."]
```

**Build & Run**

```bash
docker build -t discord-ticket-bot .
docker run --env-file .env -v $(pwd)/data:/app/data -v $(pwd)/transcripts:/app/transcripts discord-ticket-bot
```

---

## ☁️ Deploy Tips

* **Railway/Render/Fly.io**: add `.env` variables in dashboard, set start command to `node .`
* **PM2** (VPS):

  ```bash
  npm i -g pm2
  pm2 start . --name "ticket-bot"
  pm2 save
  ```

---

## 🔒 Security & Privacy

* Never log raw tokens or PII to public logs.
* Keep transcripts private by default; only share on explicit request or with staff approval.
* Rotate tokens if leaked. Re-invite the bot with least-privilege permissions.
* Respect Discord’s **Developer Terms** and **Rate Limits**.

---

## 🧭 Troubleshooting

* **Slash commands not showing** → You likely didn’t run `npm run deploy:commands` or you’re in the wrong guild.
* **Bot online but no channels created** → Missing `Manage Channels` permission, wrong `TICKET_CATEGORY_ID`, or role hierarchy.
* **“Missing Access”** → The bot’s role is below the target role/category in **Server Settings → Roles**.
* **Transcripts empty** → Ensure the bot has `Read Message History` and you’re not blocking content via channel overwrites.

---

## 🤝 Contributing

1. Fork the repo & create a feature branch
2. Follow the existing code style (`eslint`, `prettier`)
3. Add/update tests (if present) & docs
4. Open a PR with a clear description

---

## 📜 License

**MIT License** — see `LICENSE` file.
**Attribution required** in your README or bot `/about` command:

> “Built on the **Discord Ticket Bot** by **Sujan (*lyzx*)**.”

If you redistribute a modified version publicly, keep this credit (fair use/credit clause).

---

## 🙌 Credits & Fair Use

* Original concept & implementation: **Sujan (lyzx)**
* Libraries: **discord.js**, dotenv, sqlite/lowdb, pino
* You’re free to **use, modify, and self-host** under MIT, provided you keep the **credit** above.
* Please **do not sell** this code as a closed-source product without highlighting the original credit and MIT terms.

---

## 📎 Example `.env.example`

```env
DISCORD_TOKEN=
DISCORD_CLIENT_ID=
GUILD_ID=
STAFF_ROLE_ID=
TICKET_CATEGORY_ID=
MAX_OPEN_TICKETS_PER_USER=1
TICKET_COOLDOWN_SECONDS=120
DB_PROVIDER=file
DB_FILE=./data/tickets.json
SQLITE_FILE=./data/tickets.db
TRANSCRIPTS_DIR=./transcripts
```

---

## 🧪 Health Check

* Use `/ticket open` in a test guild
* Verify the ticket channel is created under your category
* Confirm visibility: user + staff + bot only
* Run `/ticket close` → transcript is saved to `./transcripts`

---

## 📣 Support

Open an issue with:

* Node.js version
* discord.js version
* Logs (sanitized)
* Repro steps
* What you expected vs. what happened

---

### README badge  



```md
![Node](https://img.shields.io/badge/node-%3E%3D18-brightgreen)
![discord.js](https://img.shields.io/badge/discord.js-v14-blue)
![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
```

---

