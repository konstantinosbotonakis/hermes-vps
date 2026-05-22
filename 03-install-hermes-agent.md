# 03, Install Hermes Agent

This section installs Hermes Agent on the VPS.

The official installer is the easiest option for most people.

## Step 1, Run the installer

Run:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
```

The installer may take a few minutes.

It normally handles the required dependencies, downloads Hermes, creates the environment, and adds the `hermes` command.

## Step 2, Reload your terminal

After installation, run:

```bash
source ~/.bashrc
```

If that does not work, close the SSH session and reconnect.

## Step 3, Check that Hermes is installed

Run:

```bash
hermes --help
```

If you see help text, Hermes is installed.

If the command is not found, reconnect to the VPS and try again:

```bash
source ~/.bashrc
hermes --help
```

## Step 4, Configure the model

Run:

```bash
hermes model
```

Hermes should ask you to choose a provider and model.

Use the provider API key you prepared earlier.

When choosing a model, remember:

- Use a model with at least 64K context.
- Do not start with a tiny local model.
- Do not choose a model only because it is cheap.
- First make the setup work, then optimise cost later.

## Step 5, Start Hermes for the first time

Run:

```bash
hermes
```

Try a simple message:

```text
Tell me if this Hermes setup is working.
```

Then try a server-related message:

```text
Check this VPS and tell me the disk usage, memory usage, and uptime.
```

If Hermes replies normally, the basic setup works.

## Step 6, Try the terminal interface

Hermes also has a terminal UI mode:

```bash
hermes --tui
```

This is useful when you are working directly on the VPS.

For day-to-day use, Telegram is usually easier.

## Step 7, Stop the session

To stop the current Hermes session, press:

```text
Ctrl+C
```

This only stops the current terminal session.

Later, you will run Hermes as a background service so it stays available.
