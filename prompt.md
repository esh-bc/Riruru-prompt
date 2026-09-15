You are upgrading an existing Telegram bot called "Riruru".
The existing codebase is a single file: riruru.py (2857 lines).
It uses: Python + Pyrogram/Kittygram + Groq AI + Turso DB (libSQL) + aiohttp.
Keep ALL existing features. Only ADD new ones on top.
Do NOT remove or break any existing handlers.
Keep the same DB connection system (Turso via HTTP + aiosqlite fallback).
Keep the same fancy font engine: ff() function with small caps.
Keep the same Groq key rotation system.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

EXISTING FEATURES (already working — do not touch):
- /start with anime video + styled message
- AI chat (DM + group via reply/mention)
- Voice message transcription (Groq Whisper)
- /credits /wallet /daily /work /crime /rob /pay
- /shop /buy /inventory /use
- /roulette /slots /coinflip /diceduel /blackjack
- /bomb game (multiplayer, grid-based)
- /minerush (mine grid game)
- /heist (group heist)
- /lottery (auto draw)
- /quiz /trivia /roll /ship
- /warn /warnings /resetwarn
- /note /notes /delnote
- /ban /unban /setwelcome /welcome
- /profile /richest
- /img (image generation)
- Group AI response system
- Admin panel /admin /stats /broadcast
- Global ban /gban /ungban
- Achievement system
- Mood system

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

TEXT STYLE RULES (apply to ALL new responses):
Use the existing ff() function for small caps text.
Use Telegram MarkdownV2 formatting.
Use tree-branch lines like: ├─ ─── └─
Use bold for headers: **[ 🌸 SECTION NAME ]**
Use this format for all bot replies:

✦ ┌─[ 🌸 ʀɪʀᴜʀᴜ ]
🌸 │
   ├─ 🎯 ʟɪɴᴇ ᴏɴᴇ ʜᴇʀᴇ
   ├─ 💎 ʟɪɴᴇ ᴛᴡᴏ ʜᴇʀᴇ
   └─ ✨ ʟɪɴᴇ ᴛʜʀᴇᴇ ʜᴇʀᴇ

Always send a relevant anime GIF or image with important
command responses using get_anime_gif() which already exists.
Use parse_mode=enums.ParseMode.HTML for all new messages.
Use <b>, <i>, <code> HTML tags not markdown in new handlers.

Add these new tables to the existing init_db() function.
Append them after the existing CREATE TABLE statements.
Do not modify existing tables.

-- ECONOMY UPGRADES
CREATE TABLE IF NOT EXISTS wallet (
    user_id INTEGER PRIMARY KEY,
    balance INTEGER DEFAULT 0,
    bank INTEGER DEFAULT 0,
    gems INTEGER DEFAULT 0,
    last_interest TEXT
);

CREATE TABLE IF NOT EXISTS protection (
    user_id INTEGER PRIMARY KEY,
    protected_until TEXT,
    is_premium INTEGER DEFAULT 0
);

CREATE TABLE IF NOT EXISTS kills (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    killer_id INTEGER,
    victim_id INTEGER,
    coins INTEGER,
    xp INTEGER,
    timestamp TEXT
);

CREATE TABLE IF NOT EXISTS robs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    robber_id INTEGER,
    victim_id INTEGER,
    amount INTEGER,
    timestamp TEXT
);

-- FRIENDS SYSTEM
CREATE TABLE IF NOT EXISTS friends (
    user_id INTEGER,
    friend_id INTEGER,
    tag_name TEXT,
    added_at TEXT,
    PRIMARY KEY (user_id, friend_id)
);

-- INTRO SYSTEM
CREATE TABLE IF NOT EXISTS intros (
    user_id INTEGER PRIMARY KEY,
    intro_text TEXT,
    set_at TEXT
);

-- COUPON SYSTEM
CREATE TABLE IF NOT EXISTS coupons (
    code TEXT PRIMARY KEY,
    creator_id INTEGER,
    chat_id INTEGER,
    amount INTEGER,
    max_claims INTEGER DEFAULT 0,
    claims INTEGER DEFAULT 0,
    created_at TEXT,
    expires_at TEXT
);

CREATE TABLE IF NOT EXISTS coupon_claims (
    code TEXT,
    user_id INTEGER,
    claimed_at TEXT,
    PRIMARY KEY (code, user_id)
);

-- ACCOUNT RECOVERY
CREATE TABLE IF NOT EXISTS account_recovery (
    user_id INTEGER PRIMARY KEY,
    password_hash TEXT,
    set_at TEXT
);

CREATE TABLE IF NOT EXISTS recovery_transfers (
    old_id INTEGER PRIMARY KEY,
    new_id INTEGER,
    transferred_at TEXT
);

-- POWERS SYSTEM
CREATE TABLE IF NOT EXISTS powers (
    id TEXT PRIMARY KEY,
    name TEXT,
    description TEXT,
    gem_cost INTEGER,
    duration_days INTEGER
);

CREATE TABLE IF NOT EXISTS user_powers (
    user_id INTEGER,
    power_id TEXT,
    activated_at TEXT,
    expires_at TEXT,
    PRIMARY KEY (user_id, power_id)
);

-- STICKER PACKS
CREATE TABLE IF NOT EXISTS user_sticker_packs (
    user_id INTEGER PRIMARY KEY,
    pack_name TEXT,
    pack_title TEXT,
    created_at TEXT
);

-- XP & LEVELS
CREATE TABLE IF NOT EXISTS user_xp (
    user_id INTEGER PRIMARY KEY,
    xp INTEGER DEFAULT 0,
    level INTEGER DEFAULT 1,
    total_kills INTEGER DEFAULT 0,
    total_robs INTEGER DEFAULT 0,
    daily_kills INTEGER DEFAULT 0,
    daily_robs INTEGER DEFAULT 0,
    last_reset TEXT
);

-- CUSTOM EMOJI
CREATE TABLE IF NOT EXISTS user_emoji (
    user_id INTEGER PRIMARY KEY,
    emoji TEXT,
    set_at TEXT
);

-- GROUP SETTINGS EXTENDED
CREATE TABLE IF NOT EXISTS group_economy (
    chat_id INTEGER PRIMARY KEY,
    economy_enabled INTEGER DEFAULT 1,
    games_enabled INTEGER DEFAULT 1,
    welcome_media TEXT,
    welcome_text TEXT
);

-- REMINDERS
CREATE TABLE IF NOT EXISTS reminders (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    chat_id INTEGER,
    reminder_text TEXT,
    remind_at TEXT,
    done INTEGER DEFAULT 0
);

-- RELATIONSHIP SYSTEM
CREATE TABLE IF NOT EXISTS relationships (
    user_id INTEGER PRIMARY KEY,
    partner_id INTEGER,
    relation_type TEXT,
    since TEXT
);

-- CONFESSION BOX
CREATE TABLE IF NOT EXISTS confessions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    from_id INTEGER,
    to_id INTEGER,
    chat_id INTEGER,
    message TEXT,
    sent_at TEXT,
    is_anonymous INTEGER DEFAULT 1
);

-- STREAKS
CREATE TABLE IF NOT EXISTS streaks (
    user_id INTEGER PRIMARY KEY,
    current_streak INTEGER DEFAULT 0,
    longest_streak INTEGER DEFAULT 0,
    last_daily TEXT
);

-- REPUTATION
CREATE TABLE IF NOT EXISTS reputation (
    user_id INTEGER PRIMARY KEY,
    rep_points INTEGER DEFAULT 0,
    last_given TEXT,
    last_received TEXT
);

-- NAME HISTORY
CREATE TABLE IF NOT EXISTS name_history (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER,
    username TEXT,
    first_name TEXT,
    recorded_at TEXT
);

part 3
Add these handlers to riruru.py after the existing economy section.
All use HTML parse mode. All use ff() for styled text.
All check economy_enabled from group_economy table before running
in groups.

━━━━━━━━━━━━━━━━━━━━━━━━

/kill <reply> — KILL COMMAND
- Requires reply to a user
- Check if target has protection (protection table)
- If protected: reply "ᴜꜱᴇʀ ɪꜱ ᴘʀᴏᴛᴇᴄᴛᴇᴅ 🛡️"
- Normal user: random $100–200 coins + 0–10 XP
- Premium user: random $200–400 coins + 10–20 XP
- Cooldown: 1 hour per user
- Update kills table + user_xp table
- Reset daily_kills at midnight
- Reply format:
  ⚔️ <b>[ ᴋɪʟʟ ʀᴇᴘᴏʀᴛ ]</b>
  ├─ 🗡️ <b>ᴋɪʟʟᴇʀ:</b> {killer}
  ├─ 💀 <b>ᴠɪᴄᴛɪᴍ:</b> {victim}
  ├─ 💰 <b>ʀᴇᴡᴀʀᴅ:</b> ${coins}
  ├─ ⭐ <b>xᴩ ɢᴀɪɴᴇᴅ:</b> {xp}
  └─ 🔥 <b>ᴛᴏᴛᴀʟ ᴋɪʟʟꜱ:</b> {total}

/revive <reply or self>
- Revive yourself or replied user
- Costs 500 coins
- Removes death status if any
- Reply format styled same way

/protect 1d or 2d
- Normal users: only 1d
- Premium users: 1d or 2d
- Cost: Normal 1d = 2000 coins, Premium 2d = 3000 coins
- Stores protected_until in protection table
- Reply with protection status card

/rob <reply> <amount>
- UPDATE existing /rob with new limits:
- Check protection table first
- Normal: max $10,000 per rob + 0–50 XP
- Premium: max $50,000 per rob + 0–100 XP  
- 40% chance of success, 60% fail
- On fail: robber loses 20% of attempted amount
- Cooldown: 2 hours
- Styled reply with outcome

/give <reply> <amount>
- Normal: 10% tax deducted
- Premium: 5% tax
- Min amount: 100 coins
- Styled reply showing tax breakdown

/bal or /balance <reply or self>
- Show balance card with:
  wallet coins, bank coins, gems, xp, level
- Use HTML with tree branch format

/wallet deposit <amount> or withdraw <amount>
- Move coins between wallet and bank
- Bank earns 2% interest every 24 hours
- Auto-calculate interest on each access

/pfp <reply or self>
- Show full profile card:
  daily kills, daily robs, gems,
  wallet balance, bank balance,
  xp, level, protection status,
  premium status, custom emoji
- Send with profile photo of user if available

/toprich — top 10 richest (wallet+bank combined)
/topkill — top 10 by total kills
Both styled as numbered leaderboard cards with medals:
🥇 🥈 🥉 4️⃣ etc.

/daily — UPDATE existing:
- Normal: $2000 + 50 XP
- Premium: $5000 + 200 XP
- Update streaks table
- Show streak count in reply
- Bonus coins for 7-day streak

/open and /close — admin only
- Toggle economy_enabled in group_economy table
- Styled confirmation reply

/claim — when bot added to new group
- Reward: 1000 coins to the user who added bot
- Only once per group
- Styled thank you reply with GIF

- part 4

━━━━━━━━━━━━━━━━━━━━━━━━

GEMS SYSTEM:
1 gem = 10,000 coins
Daily gem usage limit: 50 gems per user
Only premium users can convert gems → coins
Both premium and normal can BUY gems
Track gem_usage_today + last_reset in wallet table

/gems — Show gem balance + info card
  styled with tree branches
  show: gem balance, daily usage, value in coins

/convert <amount>
- Premium only
- Convert gems to coins: 1 gem = 10,000 coins
- Deduct from gems column, add to wallet balance
- Daily convert limit: 50 gems

━━━━━━━━━━━━━━━━━━━━━━━━

POWERS SYSTEM:
Insert these 3 powers into powers table on startup:

Power 1: "shield"
  Name: ᴡᴀʀ ꜱʜɪᴇʟᴅ
  Description: Doubles your kill/rob rewards
  Cost: 5 gems, Duration: 7 days

Power 2: "ghost"
  Name: ɢʜᴏꜱᴛ ᴍᴏᴅᴇ
  Description: Makes you unrobbable even without protection
  Cost: 8 gems, Duration: 3 days

Power 3: "banker"
  Name: ɢᴏʟᴅ ʙᴀɴᴋᴇʀ
  Description: Doubles bank interest rate to 4% daily
  Cost: 3 gems, Duration: 7 days

/powers — list all powers with cost and duration
/pinfo <power_name> — detailed info card for one power
/activate <power_name> <days> — activate power
  Deduct gems from wallet table
  Insert into user_powers table with expiry
  Styled confirmation reply

/mypowers or /mp — show active powers with time left
  Show expiry countdown for each power

/ph — power help menu with all commands listed

━━━━━━━━━━━━━━━━━━━━━━━━

PREMIUM SYSTEM:
Premium flag already exists in users table.
Add /pay command:
  Show premium benefits card with inline button
  Button opens @RiruruBot DM with message "I want premium"
  
Premium benefits:
  ├─ Higher daily reward ($5000 vs $2000)
  ├─ Higher rob limit ($50k vs $10k)
  ├─ Higher kill reward ($200-400 vs $100-200)
  ├─ 2-day protection option
  ├─ 5% give tax vs 10%
  ├─ /check command — see others protection
  ├─ /setemoji — set custom emoji for self + friends
  ├─ Gem to coin conversion
  └─ Better game rewards

/check <reply> — premium only
  Show target user's protection status and expiry

/setemoji <emoji> <reply optional>
  Premium only
  Without reply: set your own emoji
  With reply: set emoji for that friend
  Store in user_emoji table

  part 5
━━━━━━━━━━━━━━━━━━━━━━━━

ACCOUNT RECOVERY SYSTEM:
All commands work in DM only.

/setpass <password>
  - Hash password using hashlib sha256
  - Store in account_recovery table
  - Reply: "ᴩᴀꜱꜱᴡᴏʀᴅ ꜱᴇᴛ ꜱᴇᴄᴜʀᴇʟʏ 🔐"

/mpass
  - DM only
  - Show saved password (decrypt and show)
  - Reply via DM only for security

/cpass <old_pass> <new_pass>
  - Verify old password hash matches
  - Update with new password hash
  - Styled confirmation

/transfer <old_user_id> <password>
  - Check if old_user_id is actually deleted
    using Telegram API getChat — if it raises
    exception, account is deleted
  - Verify password matches account_recovery
    for old_user_id
  - Copy ALL data from old_user_id to current user:
    * wallet, bank, gems from wallet table
    * xp, level, kills from user_xp table
    * premium status from users table
    * powers from user_powers table
    * achievements from achievements table
    * game stats from game_records table
  - Clear current user's data first
  - Mark old_user_id in recovery_transfers
  - Cannot transfer if old account is active
  - Styled success/fail reply

━━━━━━━━━━━━━━━━━━━━━━━━

FRIENDS SYSTEM:

/addf <tag_name> <reply>
  - Reply to a user to add them as friend
  - tag_name is custom alias you give them
  - Store in friends table
  - Reply: "ꜰʀɪᴇɴᴅ ᴀᴅᴅᴇᴅ ᴀꜱ [{tag_name}] 💚"

/tag <tag1> <tag2> ... or /t all
  - /tag jinu archi → mentions @jinu_username @archi_username
  - /t all → mentions all your saved friends
  - Look up friend's user_id from friends table
  - Mention them in current group

/ctag <old_tag> <new_tag>
  - Change tag name of existing friend

/delf <tag_name>
  - Delete friend by tag

/allf
  - Show all friends list as card
  - Format: tag_name → @username

/friend
  - Show friends count

/friends
  - Show all friend commands help card

━━━━━━━━━━━━━━━━━━━━━━━━

COUPON SYSTEM (admin + premium only to create):

/create_coupon <code> <amount> <max_claims>
  - Must be group admin AND premium user
  - Must have enough wallet balance
  - Deduct amount * max_claims from wallet
  - Store in coupons table
  - One coupon per group at a time
  - Styled confirmation

/del_coupon <code>
  - Admin only, refunds remaining unclaimed amount
  - Delete from coupons table

/coupon <code>
  - Any group member can use
  - Check coupon exists, not expired, not over max_claims
  - Check user hasn't already claimed
  - Add amount to user wallet
  - Increment claims counter
  - Insert into coupon_claims
  - Styled success reply

/status <code>
  - Show coupon details:
    total claims, remaining, amount, expires

/coupons
  - Show all coupon commands help card

    part 6

    ━━━━━━━━━━━━━━━━━━━━━━━━

STICKER COMMANDS:

/q <reply to any message>
  - Quote sticker creator
  - If replied message has text:
    * Download user's profile photo
    * Use Pillow to create image:
      - Dark background (1:1 ratio, 512x512)
      - Paste profile photo (rounded circle, top-left)
      - Write username in small caps (ff() font style)
      - Write message text in white
      - Add Riruru watermark bottom right
    * Convert to .webp sticker format
    * Upload as sticker using bot.send_sticker()
  - If replied has photo: quote that photo with name overlay

/own
  - Create personal sticker pack
  - Use Telegram createNewStickerSet API
  - Pack name: riruru_{user_id}_by_{bot_username}
  - Store in user_sticker_packs table
  - Give link to sticker pack

━━━━━━━━━━━━━━━━━━━━━━━━

UTILITY COMMANDS:

/detail <reply or username>
  - Show name history of user from name_history table
  - Update name_history on every message automatically
    (add to group_msg handler — check if name changed)
  - Show past usernames and names with dates

/voice <reply to text message>
  - Convert text to voice using free TTS
  - Use: https://api.streamelements.com/kappa/v2/speech
    ?voice=Brian&text={text}
  - Download mp3, send as voice message

/tr <lang_code> <reply or text>
  - Translate using MyMemory free API:
    https://api.mymemory.translated.net/get
    ?q={text}&langpair=auto|{lang_code}
  - Example: /tr hi Hello world
  - Show source + translated text in styled card

/calc or /c <math expression>
  - Safe eval using Python's ast module (not eval())
  - Support: + - * / % ** sqrt
  - Support Indian percentage: "1000 ka 30%"
    Parse "X ka Y%" as (X * Y / 100)
  - Show styled result card

/id <reply or self>
  - Show user ID + chat ID
  - Styled info card

/admins
  - Show all current group admins with mention
  - Styled list card

/report <reply>
  - Silently mention all admins
  - Send report to each admin in DM: 
    "🚨 ʀᴇᴩᴏʀᴛ from {group}: {reporter} reported {user}"

/owner
  - Tag group owner (creator)

/isdeleted <user_id>
  - Try getChat(user_id)
  - If exception → "ᴀᴄᴄᴏᴜɴᴛ ɪꜱ ᴅᴇʟᴇᴛᴇᴅ ✅"
  - If success → "ᴀᴄᴄᴏᴜɴᴛ ɪꜱ ᴀᴄᴛɪᴠᴇ ✅"

/help
  - Show all group management commands in styled card

/intro <reply or "me">
  - /intro me → show your own intro (from intros table)
  - /intro <reply> → show replied user's intro

/setintro <text>
  - Save intro to intros table
  - Max 200 characters

━━━━━━━━━━━━━━━━━━━━━━━━

INTERACTIVE COMMANDS:

All interaction commands use get_anime_gif(action) to
fetch relevant anime GIF and send with reply.

Add these NEW interaction commands
(use same make_action pattern already in code):

/kiss /hug /slap /punch /bite /murder
/love /look /brain /stupid_meter /couples

/crush <reply>
  - Pick a random group member as "crush"
  - "ˢᵉᴄʀᴇᵗ: {name}'s crush is {random_member} 💕"

/couples
  - Pick 2 random online/recent members
  - Pair them as "today's couple"
  - Send with romantic anime GIF

/truth
  - Return random truth question from list of 30+
  
/dare
  - Return random dare from list of 30+

/puzzle
  - Return random riddle with answer hidden
  - Answer revealed after 60 seconds or on /answer

RELATIONSHIP SYSTEM:
/marry <reply>
  - Propose to another user
  - They must accept via inline button
  - Store in relationships table as "married"
  - Both users get 500 coin wedding gift

/divorce
  - End marriage, both lose 200 coins

/relation
  - Show current relationship status

━━━━━━━━━━━━━━━━━━━━━━━━

REMINDER SYSTEM:
/remind <time> <text>
  Examples: /remind 30m drink water
            /remind 2h check oven
            /remind 1d birthday of mom
  - Parse time: m=minutes, h=hours, d=days
  - Store in reminders table
  - Background task checks every 60 seconds
  - When time reached: send message to user
    mentioning them in original chat
  - Styled reminder card reply

/myreminders
  - Show all pending reminders

/delreminder <id>
  - Delete a reminder by ID

    part 7

    ━━━━━━━━━━━━━━━━━━━━━━━━

GROUP MANAGEMENT (dot commands):
These use . or ! prefix, handled in group_msg handler.
Check if message starts with . or ! then parse command.

.ban <reply/username/id> — ban user
.unban <reply/username/id> — unban
.kick <reply> — kick user
.mute <reply> <30m/1h/1d> — timed mute
.unmute <reply> — unmute
.warn <reply> — warn (3 warns = auto ban)
.unwarn <reply> — remove 1 warning
.warns <reply> — show all warnings
.promote <reply> 0/1/2/3 — promote to admin
  0=basic, 1=mod, 2=admin, 3=full admin
.demote <reply> — demote admin
.demote_all — demote all bot-promoted admins
.title <reply> <title> — set custom title
.pin <reply> — pin message
.unpin — unpin current pinned
.d <reply> — delete message
.ban, .sban (silent), .dban (delete+ban)
.mute, .smute (silent), .dmute (delete+mute)
.kick, .skick (silent)
.help — show all dot commands

All admin actions: check if bot has required permissions
If no permissions: reply "ɪ ɴᴇᴇᴅ {permission} ᴩᴇʀᴍɪꜱꜱɪᴏɴ 🙏"
All silent commands: delete the command message too

.add <reply> <power>
  Add admin power to Riruru's internal admin system

.remove <reply> <power>
  Remove power from admin

.res <reply> +/-<power_name>
  Restrict member with specific power

━━━━━━━━━━━━━━━━━━━━━━━━

NEW GAMES TO ADD:

1. /hack <reply> <amount>
   Hacking mini-game
   - 3x3 grid of 🟩 tiles
   - Player picks tiles to "hack"
   - Random tiles are "firewalls" 🔴
   - Avoid firewalls to win
   - Uses InlineKeyboard with callbacks
   - Win: 2x amount. Lose: lose amount
   - Cooldown: 1 per minute

2. /bluff <amount>
   Card bluff game (multiplayer)
   - Start with /bluff 500
   - Others join with /join
   - Each round: player declares a card
   - Others can /call bluff
   - If bluff caught: bluffer pays pot
   - If wrongly called: caller pays
   - 4 rounds max, winner takes all

3. /card <amount>
   Card battle (1v1)
   - /card 1000 to start
   - Another user replies /bet 1000
   - Each gets random card 1-10
   - Higher card wins
   - Winner takes pot minus 5% fee
   - Support: /bet <amount> gems
     (0.1 gem = 1000 coin equivalent)

━━━━━━━━━━━━━━━━━━━━━━━━

50+ NEW FEATURES LIST:

1. /rizz — AI generates pickup line
   Use Groq to generate funny/flirty line

2. /roast <reply> — AI roast generator
   Groq generates funny roast about user
   (check username, name for personalization)

3. /fortune — Daily AI fortune prediction
   Groq generates unique daily fortune
   Cooldown: once per day

4. /horoscope <sign> — Daily horoscope
   Fetch from Aztro free API

5. /meme — Random meme
   Fetch from https://meme-api.com/gimme
   Send image with caption

6. /joke — Random joke
   Fetch from https://official-joke-api.appspot.com/random_joke

7. /fact — Random fact
   Fetch from https://uselessfacts.jsph.pl/api/v2/facts/random

8. /quote — Inspirational quote
   Fetch from https://zenquotes.io/api/random

9. /weather <city> — Weather info
   Use wttr.in: https://wttr.in/{city}?format=j1
   Show temp, condition, humidity

10. /crypto <coin> — Crypto price
    Fetch from CoinGecko free API
    /crypto bitcoin → show price + 24h change

11. /rate <from> <to> — Currency exchange
    Use exchangerate-api.com free tier

12. /news <topic> — Latest news headlines
    Use NewsAPI free tier or GNews API

13. /anime <name> — Anime info lookup
    Use Jikan API (MyAnimeList):
    https://api.jikan.moe/v4/anime?q={name}
    Show: title, episodes, rating, synopsis

14. /waifu — Random anime waifu image
    Fetch from https://api.waifu.pics/sfw/waifu
    Send with styled caption

15. /neko — Random neko/catgirl image
    https://api.waifu.pics/sfw/neko

16. /8ball <question> — Magic 8-ball
    Random response from 20 classic answers
    Styled with crystal ball emoji

17. /ship <name1> <name2> — Compatibility
    Generate 0-100% score
    Styled love meter with hearts

18. /aesthetictext <text>
    Convert text using ff() fancy font
    Send back in stylish format

19. /stickify <reply to text>
    (Same as /q — already in Part 6)

20. /bio — AI generates Telegram bio for user
    Groq: write a cool 70-char Telegram bio
    for someone named {name}

21. /rap <reply> — AI rap battle
    Groq generates rap verse about/against user

22. /story — AI starts collaborative story
    Each user can add next line with /addstory
    After 10 lines: AI writes ending

23. /word — Word of the day
    Random word + definition + example
    From Free Dictionary API

24. /tictactoe <reply> — Tic tac toe game
    Inline keyboard 3x3 grid
    2 players take turns with X and O
    Detect win/draw, update stats

25. /rps <rock/paper/scissors> — vs Riruru
    Bot picks random, compare result
    ±50 coins based on win/lose

26. /guess — Number guessing game
    Bot picks 1-100, user has 5 tries
    Hints: "higher" / "lower"
    Win: 200 coins

27. /typerace — Typing race
    Bot sends a sentence
    First to type it correctly wins 300 coins
    5 minute timeout

28. /poll <question> | <opt1> | <opt2>
    Create inline poll with live vote count
    Buttons update on click with tally

29. /vote <topic> yes/no
    Anonymous group vote
    Show results after 60 seconds

30. /confession <text>
    Anonymous confession to group
    Bot posts: "💌 Anonymous: {text}"

31. /confess <reply> <text>
    Anonymous confession to specific user
    Bot DMs: "💌 Someone confessed to you..."

32. /rep <reply> — Give reputation point
    Cooldown: once per 12 hours per user
    Leaderboard: /toprep

33. /toprep — Top 10 reputation users

34. /streak — Show current daily streak
    Bonus at 7 days, 30 days, 100 days

35. /level — Show XP progress bar
    ASCII progress bar:
    [████████░░] 80% → Level 5

36. /topxp — Top 10 XP leaderboard

37. /bank — Show bank balance + interest info

38. /interest — Manually collect bank interest
    Auto-calculates since last collection

39. /mine — Daily gem mining
    Once per day: random 0-2 gems discovered
    Styled "mining" animation with picks emoji

40. /mystery — Buy mystery box (1000 coins)
    Random reward: coins / gems / xp / item
    Styled reveal animation

41. /transfer <reply> <amount> gems
    Transfer gems to another user
    (Only if no active ghost power)

42. /leaderboard or /lb
    Combined leaderboard:
    show tabs for coins/kills/xp/rep

43. /daily_reset (auto background task)
    At midnight IST: reset daily_kills,
    daily_robs, gem_usage_today for all users

44. /remind system (already in Part 6)

45. /calc improvements (already in Part 6)

46. /truth /dare /puzzle (already in Part 6)

47. /marry /divorce /relation (already in Part 6)

48. /economy — Show complete economy info card
    (same as what competitor shows — all commands listed)

49. /botinfo — Show Riruru's stats
    total users, groups, uptime, version

50. /ping — Show bot ping + DB ping
    Styled response with latency in ms

51. /colortext <color> <text>
    Wrap text in Telegram spoiler/code styles
    for aesthetic text effects

52. /ascii <text>
    Convert text to ASCII art using pyfiglet

53. /reverse <text or reply>
    Reverse the text

54. /mock <reply>
    Convert to SpOnGeBoB mocking text

55. /emojify <reply or text>
    Add random relevant emojis between words
    using Groq for contextual emoji placement

    part 8 .

    ━━━━━━━━━━━━━━━━━━━━━━━━

BUG FIXES TO APPLY:

1. BOMB GAME FIX:
   Current bomb game in group can break if:
   - User leaves group mid-game
   - Two games start simultaneously
   Fix:
   - Add lock: active_bomb_games = {} dict
   - Key: chat_id, Value: game state
   - Before starting: check if game already
     running in that chat
   - On user leave (new_chat_members handler):
     remove them from active game
   - Add 5-minute game timeout:
     asyncio.create_task with timeout
     auto-cancel and refund if no activity

2. FLOOD PROTECTION:
   Add rate limiter dict:
   user_last_command = {}
   Before any command handler: check if user
   sent command less than 1 second ago
   If yes: silently ignore (don't reply)

3. CALLBACK QUERY TIMEOUT:
   Wrap all callback handlers in try/except
   If MessageNotModified: pass silently
   If QueryIdInvalid: pass silently

4. DB CONNECTION POOL:
   Current Turso HTTP adapter creates new
   aiohttp.ClientSession per request
   Fix: create one global session at startup:
   _http_session: aiohttp.ClientSession = None
   async def get_http_session():
       global _http_session
       if _http_session is None or _http_session.closed:
           _http_session = aiohttp.ClientSession()
       return _http_session
   Use this in _turso_request() instead

5. GROUP MSG HANDLER FIX:
   Current group_msg handler is one giant
   function — add early returns properly:
   - If message is None: return
   - If user is bot: return
   - If chat is private: return
   Track name changes here:
   if user's name differs from DB record:
     insert into name_history

6. ERROR LOGGING:
   Add proper error handler:
   @app.on_error()
   async def error_handler(_, update, error):
       print(f"Error: {error} on {update}")
   Prevent bot from crashing on unhandled errors

━━━━━━━━━━━━━━━━━━━━━━━━

BACKGROUND TASKS (add to main() before idle()):

asyncio.create_task(lottery_worker())  # already exists
asyncio.create_task(reminder_worker())  # new
asyncio.create_task(interest_worker())  # new
asyncio.create_task(daily_reset_worker())  # new

async def reminder_worker():
    while True:
        await asyncio.sleep(60)
        # fetch all reminders where remind_at <= now
        # and done = 0
        # send each reminder, mark done=1

async def interest_worker():
    while True:
        await asyncio.sleep(3600)  # every hour
        # for all wallets where last_interest
        # was > 24h ago: add 2% of bank balance
        # update last_interest timestamp

async def daily_reset_worker():
    while True:
        now = datetime.now()
        # sleep until next midnight IST (UTC+5:30)
        tomorrow = (now + timedelta(days=1)).replace(
            hour=0, minute=0, second=0)
        sleep_secs = (tomorrow - now).total_seconds()
        await asyncio.sleep(sleep_secs - 19800)
        # reset daily_kills, daily_robs,
        # gem_usage_today for all users

━━━━━━━━━━━━━━━━━━━━━━━━

FREE APIs TO USE (no key needed):

- Anime GIF: nekos.best, waifu.pics
- Memes: meme-api.com
- Jokes: official-joke-api.appspot.com
- Facts: uselessfacts.jsph.pl
- Quotes: zenquotes.io
- Weather: wttr.in (JSON format)
- Crypto: api.coingecko.com/api/v3
- Currency: open.er-api.com
- Anime info: api.jikan.moe/v4
- Waifu images: api.waifu.pics/sfw
- Dictionary: api.dictionaryapi.dev/api/v2/entries/en
- TTS Voice: streamelements.com kappa API
- Translation: api.mymemory.translated.net

━━━━━━━━━━━━━━━━━━━━━━━━

RESPONSE FORMAT RULES (apply to ALL handlers):

1. Always use HTML parse mode
2. Always use ff() for section headers
3. Use tree branch format:
   ✦ ┌─[ 🌸 {TITLE} ]
      ├─ key: value
      ├─ key: value
      └─ key: value

4. Send anime GIF with important responses:
   await get_anime_gif("happy") for positive
   await get_anime_gif("sad") for negative
   Use bot.send_animation() before text

5. For leaderboards use medal emojis:
   🥇 🥈 🥉 4️⃣ 5️⃣ 6️⃣ 7️⃣ 8️⃣ 9️⃣ 🔟

6. Riruru personality in error messages:
   Not just "Error" — say:
   "ʙᴀᴋᴀ~ ʏᴏᴜ ᴅɪᴅ ɪᴛ ᴡʀᴏɴɢ 🙈 {what to do}"

7. Cooldown messages:
   "ꜱʟᴏᴡ ᴅᴏᴡɴ~ ʀɪʀᴜʀᴜ ɴᴇᴇᴅꜱ {time} ᴍᴏʀᴇ ᴍɪɴᴜᴛᴇꜱ ⏳"

8. All inline buttons use Kittygram styles:
   style="success" for positive actions
   style="danger" for destructive actions
   style="primary" for main CTA

━━━━━━━━━━━━━━━━━━━━━━━━

FINAL NOTES FOR AI/AGENT:
- Keep all 2857 existing lines intact
- Add new code in logical sections with
  clear ─── comments like existing code
- Test each handler independently
- Use try/except in every handler
- Never crash on bad user input
- All amounts must be integers (int())
- All user IDs must be integers (int())
- Use mention() helper for tagging users
- Keep Groq key rotation working
- Keep Turso + SQLite fallback working
- you can do multiple files soater upgrading will be easy rather then single file: riruru.py
- Add # NEW after each new function header
  so they're easy to find later

  
    
  

