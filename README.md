# 🛡️ NoDMBot

**NoDMBot** is a private Telegram userbot for protecting your private messages from unwanted DMs.

It automatically deletes private messages from unknown users, logs requests into a Telegram topic, forwards media and long texts to a separate topic, caches known Telegram users, and gives the admin quick commands to allow, remove, block, unblock, inspect, or temporarily change a user's access.

NoDMBot is maintained as a **closed-source private project** and is intended only for trusted private use.

---

## 🔒 Project Availability

NoDMBot is a **closed-source private project**.

The official deployment method is now based on a **private Docker image**. The source code is not published directly in this repository unless the developer explicitly provides a review artifact or private access.

The project must not be publicly leaked, mirrored, re-uploaded, resold, redistributed, or published as a public source repository.

If someone needs to use the bot, they must contact the original developer privately.

---

## 🐳 Official Docker Deployment

NoDMBot can be deployed using the official Docker image:

```text
mahdibmrf/nodmbot:stable
```

Versioned image example:

```text
mahdibmrf/nodmbot:v1.0
```

Docker Hub page:

```md
[Official Docker Image](https://hub.docker.com/r/mahdibmrf/nodmbot)
```

The image runs `main.py` directly and uses a read-only application directory:

```text
/app/main.py
/app/requirements.txt
```

Runtime data is stored in:

```text
/data
```

This keeps the application files separate from database/runtime files.

---

## 🧱 Docker Runtime Layout

Inside the official image:

| Path | Purpose |
|---|---|
| `/app/main.py` | Main NoDMBot source file |
| `/app/requirements.txt` | Python dependencies |
| `/data/lists.db` | Runtime SQLite database |
| `/data` | Writable runtime directory |

The `/app` directory is read-only inside the container. The `/data` directory is writable so SQLite and temporary backup/restore files can work correctly.

---

## 🚀 Run With Docker Locally

Create a `.env` file with the required environment variables, then run:

```bash
docker run --rm --env-file .env -p 5000:5000 -v nodmbot_data:/data mahdibmrf/nodmbot:stable
```

Run in background:

```bash
docker run -d --name nodmbot --env-file .env -p 5000:5000 -v nodmbot_data:/data mahdibmrf/nodmbot:stable
```

View logs:

```bash
docker logs -f nodmbot
```

Stop:

```bash
docker stop nodmbot
```

Remove the stopped container:

```bash
docker rm nodmbot
```

The named volume keeps the local database persistent:

```text
nodmbot_data:/data
```

---

## ☁️ Render Deployment With Docker

In Render:

```text
New +
→ Web Service
→ Deploy an existing image from a registry
```

Image URL:

```text
mahdibmrf/nodmbot:stable
```

or:

```text
mahdibmrf/nodmbot:v1.0
```

If the Docker image is private, add a Docker Hub credential using a **read-only Docker Hub token**.

Required Render environment variables are listed below in the Environment Variables section.

### Render Persistent Disk Note

NoDMBot stores its SQLite database in:

```text
/data/lists.db
```

Render free services do not include persistent disks. Without a persistent disk, the bot can still run, but the database may be reset after restart, redeploy, rebuild, or platform migration.

If persistent disk is available, use:

```text
Mount Path: /data
```

If persistent disk is not available, use `.backup` regularly and `.restore` when needed.

---

## 🔎 Source Review Method

NoDMBot can optionally provide a **source-view workflow** for transparency.

A GitHub Actions workflow can:

1. Log in to Docker Hub using a read-only token.
2. Pull the official Docker image.
3. Create a temporary container.
4. Extract files from `/app`.
5. Upload them as a GitHub Actions artifact.

The source-view artifact usually contains:

```text
main.py
requirements.txt
```

This method lets a trusted reviewer inspect the files extracted from the official Docker image without installing Docker Desktop locally.

Review access does not grant permission to redistribute, republish, repackage, resell, or remove attribution.

---

## ⚠️ Important Usage Notice

NoDMBot is intended to be used as a personal DM protection tool.

It must not be used for:

- Spam
- Mass messaging
- Scraping users
- Harassment
- Automated abuse
- Public deployment for unknown users
- Bypassing Telegram limits
- Any activity that violates Telegram rules

Using the project publicly or distributing it widely may cause Telegram accounts to receive limits or restrictions.

---

## ✨ Main Features

- 🛡️ Private DM protection
- 📩 Log topic for deleted DM requests
- 📎 Forward topic for media, attachments, and long texts
- ✅ Whitelist system
- 🚫 Blacklist system
- ⏳ Temporary access actions
- 👥 Automatic contacts scanner
- 🔁 Contact auto-sync control
- 🧠 Known users cache system
- 🔐 Access hash cache support
- 👤 Clickable user profile links
- 🧪 Recent DM attempts tracking
- 🔎 `.who` command to inspect cached users
- 🆔 `.id` command to show chat/user ID information
- 👥 Multi-user whitelist and blacklist commands
- ⏳ Multi-user temporary actions
- 💾 SQLite database storage
- 🔐 Encrypted `.db.enc` backup and restore
- 🧬 Argon2id + AES-256-GCM backup encryption
- 🧹 Database cleanup and maintenance commands
- ⚙️ Runtime config and stats commands
- 📊 Current RAM / CPU / disk monitoring
- 🔐 Protected admin, owner, and Telegram service IDs
- 🧵 Telegram topics support
- 🌐 Flask keep-alive web server for hosting platforms
- 🐳 Docker image deployment support
- 🔎 Optional source-view artifact workflow

---

## 🛡️ DM Protection

NoDMBot listens only to **private incoming messages**.

If a user is not whitelisted, not blocked, not a contact, not a bot, and not protected by default, the bot will:

1. Cache the sender entity when possible.
2. Log the attempt.
3. Forward media or long text when needed.
4. Delete the original DM.
5. Send or update a log message in the logs topic.

The bot ignores:

- `ADMIN_ID`
- Protected `OWNER_ID`
- Telegram service IDs
- Contacts loaded by the contact scanner
- Bot accounts
- Group and channel messages

---

## 📩 Log System

Deleted DM requests are logged into the configured **Logs Topic**.

Example log:

```text
📩 New Request:
👤 From: User Name
🆔 ID: 123456789
💬 Msg: Hello

✅ .ok 123456789
🚫 .rem 123456789
⛔️ .block 123456789
✅ .unblock 123456789
```

When the same user sends multiple messages, the bot updates the previous log message instead of spamming the log topic.

If a log becomes too long, the bot starts a new log message.

---

## 👤 Clickable User Links

NoDMBot tries to make user names clickable whenever possible.

The bot uses:

1. `https://t.me/username` if the user has a username.
2. `tg://user?id=user_id` if no username is available.
3. Cached user data if the user was seen before.

This is used in:

- New request logs
- `.tried`
- `.list`
- `.blist`
- `.templist`
- `.who`

Note:

```text
tg://user?id=... links may not always open a full profile if Telegram does not expose the user entity to the account.
```

User cache improves reliability, but it does not bypass Telegram privacy restrictions.

---

## 🧠 User Cache System

NoDMBot stores known Telegram user information in the database.

Cached data includes:

```text
user_id
access_hash
username
first_name
last_name
updated_at
```

The cache is used to:

- Improve clickable user links
- Resolve users without usernames
- Store access hashes when available
- Improve `.who`, `.list`, `.blist`, `.tried`, and `.templist`

The bot caches users when:

- An unknown user sends a private message
- Contacts are refreshed
- Known dialogs are scanned
- The bot resolves a user by ID or username

---

## 📎 Media, Attachments, and Long Text Forwarding

If a non-whitelisted user sends media or a long text, NoDMBot forwards it to the configured **Forward Topic** and adds a clickable link in the log.

Supported media labels:

```text
📷 Photo
🎥 Video
🎙️ Voice
📄 File
🎭 Sticker
📼 Gif
📄 Long Text
```

Example log line:

```text
💬 Msg: [🎭 Sticker](https://t.me/c/group_id/2/message_id)
```

Forwarded message links use this format:

```text
https://t.me/c/group_id_without_-100/topic_id/message_id
```

---

## ✅ Whitelist

Whitelisted users are allowed to message normally.

Commands:

```text
.ok user_id        Allow a user
.rem user_id       Remove a user from whitelist
.list             Show whitelist
.clearwl          Clear whitelist table
```

Inside a private chat, you can use:

```text
.ok
.rem
```

The bot will use the current private chat ID automatically.

You can also target multiple users:

```text
.ok 111 222 333
.rem 111 222 333
```

---

## 🚫 Blacklist

Blacklisted users are deleted silently without sending new request logs.

Commands:

```text
.block user_id     Block a user
.unblock user_id   Remove user from blacklist
.blist             Show blacklist
.clearbl           Clear blacklist table
```

Inside a private chat, you can use:

```text
.block
.unblock
```

The bot will use the current private chat ID automatically.

You can also target multiple users:

```text
.block 111 222 333
.unblock 111 222 333
```

Protected users cannot be blocked.

---

## 🔁 Contact Auto-Sync Control

NoDMBot automatically loads Telegram contacts and adds them to the whitelist unless contact auto-sync is disabled for a specific user.

Commands:

```text
.dsynlist          Show users with contact auto-sync disabled
.sync user_id      Enable contact auto-sync for user
.unsync user_id    Disable contact auto-sync for user
```

This is useful when a contact should not be automatically re-added to the whitelist.

---

## ⏳ Temporary Actions

NoDMBot supports temporary whitelist and blacklist actions.

Commands:

```text
.tempok user_id duration        Allow user temporarily
.temprem user_id duration       Remove user from whitelist temporarily
.tempblock user_id duration     Block user temporarily
.tempunblock user_id duration   Unblock user temporarily
.templist                       Show active temporary actions
.cleartemp                      Clear all temporary actions and restore users
```

Example:

```text
.tempok 123456789 1h30m
.tempblock 123456789 2d5h
```

Multiple users with same duration:

```text
.tempok 111 222 333 1h
.tempblock 111 222 333 2d
```

Multiple users with different durations:

```text
.tempok 111 10m 222 2h 333 1d
.tempblock 111 30m 222 1h
```

Inside a private chat:

```text
.tempok 1h
.temprem 1h
.tempblock 1h
.tempunblock 1h
```

Supported duration units:

```text
s   seconds
m   minutes
h   hours
d   days
w   weeks
mo  months, treated as 30 days
y   years, treated as 365 days
```

Protected users cannot receive temporary actions.

---

## 🧪 Attempts Tracking

Command:

```text
.tried
```

It shows recent DM attempts from the last 24 hours, including:

- Clickable user name
- User ID
- Number of messages
- Last message preview
- Last attempt time

Old attempts are automatically cleaned after 7 days.

---

## 🔎 User Inspection

Command:

```text
.who user_id
.who username
```

The command shows cached information about a user:

```text
👤 User Info:

👤 Name: User Name
🆔 ID: 123456789
🔗 Username: @username
🧠 Cached: YES
🔐 Access Hash: YES
✅ Whitelisted: YES/NO
🚫 Blacklisted: YES/NO
📇 Contact: YES/NO
🔁 Contact Sync: ON/OFF/N/A
⏳ Temp Action: NO / action + remaining time
```

---

## 🆔 ID Command

Command:

```text
.id
```

Shows current chat and user information:

```text
🆔 ID Info:

👤 Me: Admin Name
👤 My ID: 123456789
💬 Chat Name: Chat Name
💬 Chat Type: Private / Group / Channel
📍 Chat ID: 123456789
```

---

## ⚙️ Status, Config, and Stats

Commands:

```text
.status     Show current protection status
.stats      Show bot statistics
.config     Show configuration and current system usage
.id         Show current user/chat ID information
.who        Show cached user information
.help       Show help menu
.on         Enable DM protection
.off        Disable DM protection
.restart    Restart the userbot safely
```

`.config` shows runtime information such as protection status, topic IDs, database status, current RAM, CPU usage, disk usage, uptime, Python version, and platform.

`.stats` shows whitelist, blacklist, disabled contact sync, temporary actions, attempts, pending alerts, cached users, backup encryption readiness, settings count, contacts loaded, database existence, database size, and uptime.

---

## 🔐 Encrypted Backup and Restore

NoDMBot stores data in SQLite:

```text
lists.db
```

In Docker, the database is stored in:

```text
/data/lists.db
```

Commands:

```text
.backup
.backupinfo
.encryption
.restore
```

### Encryption Method

Backups use:

```text
Argon2id + AES-256-GCM
```

Backup format:

```text
NDBENC2
```

Backup files are encrypted as:

```text
.db.enc
```

Required secrets:

```text
BACKUP_PASSWORD
BACKUP_PEPPER
```

Minimum lengths:

```text
BACKUP_PASSWORD >= 20 characters
BACKUP_PEPPER >= 32 characters
```

### `.backup`

Creates an encrypted database backup and sends a `.db.enc` file.

### `.backupinfo`

Reply to a backup file with `.backupinfo` to inspect its encrypted backup structure.

### `.encryption`

Shows whether backup encryption is ready and whether the password and pepper are set.

### `.restore`

Reply to an encrypted `.db.enc` backup file with:

```text
.restore
```

The restore system verifies:

- Backup format header
- Decryption
- SQLite integrity
- Required tables
- Required timed action columns

Then it replaces the current database, reinitializes tables, clears stale alerts, and refreshes contacts.

---

## 🧹 Database Management

Commands:

```text
.cleardb     Clear database content
.clearwl     Clear whitelist table
.clearbl     Clear blacklist table
.cleartemp   Clear temporary actions and restore users
```

`.cleardb` clears whitelist, blacklist, settings, attempts, timed actions, last alerts, user cache, and disabled contact sync entries. Then it re-adds protected/default whitelist entries and re-enables protection.

`.clearwl` clears whitelist and re-adds admin, protected owner, Telegram service IDs, and loaded contacts that do not have contact sync disabled.

`.clearbl` clears blacklist only.

---

## 🧼 Automatic Cleanup

NoDMBot has a background database cleanup loop.

It removes:

```text
attempts older than 7 days
last_alerts older than 2 days
```

It does not delete whitelist, blacklist, settings, timed actions, user cache, or contact sync settings.

---

## 🗂️ Database Tables

Current SQLite tables:

```text
whitelist
blacklist
settings
attempts
timed_actions
last_alerts
user_cache
contact_sync_disabled
```

Table usage:

| Table | Purpose |
|---|---|
| `whitelist` | Allowed users |
| `blacklist` | Blocked users |
| `settings` | Persistent settings, such as protection state |
| `attempts` | Recent DM attempts |
| `timed_actions` | Temporary actions and restore state |
| `last_alerts` | Last log message IDs for log editing |
| `user_cache` | Cached user entities and access hashes |
| `contact_sync_disabled` | Users excluded from automatic contact sync |

SQLite uses WAL mode, busy timeout, and normal synchronous mode for improved stability.

---

## 🧵 Telegram Topic Setup

Create one Telegram supergroup and enable **Topics**.

Recommended setup:

```text
Topic 1: Logs
Topic 2: Forwarded
```

Environment variables:

```env
LOG_GROUP_ID=-100xxxxxxxxxx
LOG_TOPIC_ID=1
FORW_TOPIC_ID=2
```

---

## 📦 Requirements

`requirements.txt`:

```txt
telethon
flask
gunicorn
cryptography
argon2-cffi
```

Note: Docker runs `python main.py` directly. `gunicorn` may exist in requirements for compatibility, but the bot should not be started with gunicorn because Telethon and Flask are started from `main.py`.

---

## ⚙️ Environment Variables

Required environment variables:

```env
API_ID=your_api_id
API_HASH=your_api_hash
STRING_SESSION=your_string_session
ADMIN_ID=your_telegram_user_id
LOG_GROUP_ID=-100xxxxxxxxxx
LOG_TOPIC_ID=1
FORW_TOPIC_ID=2
BACKUP_PASSWORD=your_long_backup_password_20_chars_min
BACKUP_PEPPER=your_long_backup_pepper_32_chars_min
PORT=5000
```

Variable explanation:

| Variable | Description |
|---|---|
| `API_ID` | Telegram API ID |
| `API_HASH` | Telegram API Hash |
| `STRING_SESSION` | Telethon string session |
| `ADMIN_ID` | Main admin user ID |
| `LOG_GROUP_ID` | Telegram supergroup ID with topics enabled |
| `LOG_TOPIC_ID` | Topic ID for logs |
| `FORW_TOPIC_ID` | Topic ID for forwarded media and long text |
| `BACKUP_PASSWORD` | Backup encryption password, minimum 20 characters |
| `BACKUP_PEPPER` | Backup encryption pepper, minimum 32 characters |
| `PORT` | Flask web server port |

Keep `STRING_SESSION`, `API_HASH`, `BACKUP_PASSWORD`, `BACKUP_PEPPER`, and all environment variables private.

Never upload `.env` to GitHub or any public platform.

---

## 🧪 Local Python Run

For private development only:

```bash
python main.py
```

The Flask server starts and the Telethon userbot connects using your string session.

---

## 🧠 Commands List

### Status and Control

```text
.status
.stats
.config
.tried
.restart
.on
.off
.id
.who user_id
.who username
.help
.slist
```

### Whitelist

```text
.ok user_id
.ok user1 user2 user3
.ok
.rem user_id
.rem user1 user2 user3
.rem
.list
.list n<number>
.list b<number>
.clearwl
```

### Contact Sync

```text
.dsynlist
.dsynlist n<number>
.dsynlist b<number>
.sync user_id
.unsync user_id
```

### Blacklist

```text
.block user_id
.block user1 user2 user3
.block
.unblock user_id
.unblock user1 user2 user3
.unblock
.blist
.blist n<number>
.blist b<number>
.clearbl
```

### Temporary Actions

```text
.tempok user_id duration
.tempok user1 user2 user3 duration
.tempok user1 duration1 user2 duration2

.temprem user_id duration
.temprem user1 user2 user3 duration
.temprem user1 duration1 user2 duration2

.tempblock user_id duration
.tempblock user1 user2 user3 duration
.tempblock user1 duration1 user2 duration2

.tempunblock user_id duration
.tempunblock user1 user2 user3 duration
.tempunblock user1 duration1 user2 duration2

.templist
.cleartemp
```

### Backup and Database

```text
.backup
.backupinfo
.encryption
.restore
.cleardb
```

---

## ⚠️ Notes

- Management commands work only for `ADMIN_ID`.
- `.status` is an outgoing command and edits your own message.
- `.id` works for the admin and shows current chat/user information.
- `.who` shows cached information only if the bot has seen or cached the user before.
- `.list`, `.blist`, `.dsynlist`, `.tried`, and `.templist` support item and batch modes.
- `.slist` stops the current list display.
- SQLite reading is fast; Telegram entity resolution may add delay.
- `tg://user?id=...` links may not always open a full profile if Telegram does not expose the user entity.
- User cache improves reliability but does not bypass Telegram privacy restrictions.
- Docker image names and tags are lowercase by Docker convention.
- `latest`, `stable`, and `v1.0` are Docker tags; they are controlled by the developer.

---

## 🔒 Security Notes

- Keep your string session private.
- Do not share your API hash.
- Do not share your `.env` file.
- Do not publish Docker tokens publicly.
- Use read-only Docker tokens only when needed.
- Use a private Telegram group for logs and forwarded media.
- Do not redistribute or resell the project.
- The project contains a protected developer/owner ID.
- Removing, modifying, bypassing, or abusing protected developer/owner logic is not allowed.
- The bot is intended for personal/private use only.
- Docker image access does not grant ownership or redistribution rights.
- Source-view artifacts are for review only and remain subject to the license.

---

## 📌 Project Status

Current version: **NoDMBot Private Docker Edition**

Stable features:

- DM protection
- Logs topic
- Forwarded media topic
- Whitelist and blacklist
- Contact auto-sync control
- Multi-user whitelist and blacklist actions
- Temporary actions
- Multi-user temporary actions
- Contacts scanner
- Known users cache
- Access hash cache
- Clickable profile links
- `.who` user inspection
- `.id` chat/user ID info
- Attempts tracking
- Encrypted backup and restore
- Backup info and encryption status commands
- Config and stats commands
- Current RAM and CPU monitoring
- SQLite connection cleanup
- Persistent protection status
- Docker deployment support
- Optional source-view workflow support

---

## 🛡️ NoDMBot

Your DM, your rules.

Private protection. Private deployment.

---

## 📝 Private Terms of Use

- This project is closed source.
- This project is private and must not be published publicly.
- The official deployment method is the private Docker image.
- Docker image access does not grant permission to redistribute or republish the image.
- Source-view access is for review only.
- Removing, modifying, or bypassing protected developer/owner logic is not allowed.
- Sharing, leaking, re-uploading, mirroring, or redistributing the source code or extracted image contents is not allowed.
- Selling or reselling this project without permission is not allowed.
- Giving access to untrusted users is not allowed.
- The bot must only be used for personal DM protection.
- The bot must not be used for spam, scraping, harassment, mass messaging, or abuse.
- By using this bot, you agree to these terms.

---

## 👤 Owner

Original developer: **Mahdi Boumaaraf**

Telegram: [Mahdi Boumaaraf](https://t.me/XYF_R)

---

## 📄 License

This project uses a custom private license.

The source code and Docker image are not publicly licensed for redistribution.

See the [LICENSE](LICENSE) file for details.
