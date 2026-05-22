# 04, Connect Telegram

Telegram is the easiest way to use Hermes from your phone.

You message the bot. Hermes replies.

For most non-technical users, this is better than logging into the VPS every time.

## Step 1, Create a Telegram bot

Open Telegram and search for:

```text
@BotFather
```

Start a chat with BotFather.

Send:

```text
/newbot
```

BotFather will ask for a display name.

Example:

```text
My Hermes Assistant
```

Then it will ask for a username.

The username must end in `bot`.

Example:

```text
my_hermes_helper_bot
```

BotFather will give you a bot token.

It looks like a long string.

Copy it somewhere safe.

Do not publish it.

Anyone with this token may be able to control your bot.

## Step 2, Find your Telegram user ID

Hermes should only accept messages from allowed users.

Telegram usernames are not enough. You need your numeric user ID.

In Telegram, search for one of these:

```text
@userinfobot
```

or:

```text
@get_id_bot
```

Start the bot and it will show your numeric Telegram user ID.

Copy the number.

Example:

```text
123456789
```

## Step 3, Start Hermes gateway setup

On your VPS, run:

```bash
hermes gateway setup
```

Choose Telegram when asked.

Paste:

- Your Telegram bot token.
- Your numeric Telegram user ID.

Save the configuration.

## Step 4, Test Telegram manually

Start the gateway in the foreground:

```bash
hermes gateway
```

Now open Telegram and send your bot:

```text
Hello. Are you working?
```

If Hermes replies, Telegram is connected.

Stop the foreground gateway with:

```text
Ctrl+C
```

## Step 5, Install Hermes as a background service

On Linux, install the gateway service:

```bash
sudo hermes gateway install --system
```

Then start it:

```bash
hermes gateway start
```

Check it:

```bash
hermes gateway status
```

The gateway should now keep running in the background.

If the VPS reboots, the service should start again.

## Step 6, Send a useful test message

In Telegram, send:

```text
Check the VPS disk usage, memory usage, and uptime. Keep the answer short.
```

If you get a sensible answer, the useful part is working.

## Step 7, Check logs if Telegram does not reply

Run:

```bash
hermes gateway status
hermes logs
```

Common causes:

- Wrong bot token.
- Wrong Telegram user ID.
- Gateway not running.
- Model provider not configured.
- API key missing or invalid.
