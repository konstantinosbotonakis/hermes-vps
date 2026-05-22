# 09, Bonus, busyBee-cpu

This is an optional advanced guide.

Do not do this during your first Hermes setup.

First make the normal Hermes Agent setup work with Telegram, automations, and a hosted model provider.

Then come back to this if you want to experiment with reducing unnecessary model calls inside an agent harness.

## What busyBee-cpu is

`busyBee-cpu` is not a chat model.

It is a small CPU policy router for agent loops.

In a normal agent loop, the model may spend calls on obvious next steps, such as:

- Read the file before editing it.
- Run tests after a patch.
- Apply a patch that is already prepared.
- Stop and let the real model reason.

`busyBee-cpu` tries to handle those mechanical routing decisions locally on CPU.

The main actions are:

- `read_file`
- `run_tests`
- `apply_patch`
- `escalate`

When it returns `escalate`, the real LLM should take over.

That is the important point.

`busyBee-cpu` does not replace your main Hermes model. It only helps a compatible Hermes harness decide when a simple tool action can happen without waking the main model.

## When this is worth trying

Try this if:

- Your normal Hermes setup already works.
- You are comfortable using Git, Python, and command-line tools.
- You are experimenting with HermesAgent-20 or a compatible harness.
- You want to benchmark policy offload for agent workflows.
- You can safely patch and roll back a development checkout.

Do not try this if:

- You only want a Telegram assistant.
- You are still setting up Hermes for the first time.
- You expect it to improve answer quality.
- You are not comfortable applying a Git patch.
- You are running Hermes directly on an important production server.

For most users, this is a lab experiment, not a required VPS upgrade.

## What gains to expect

The upstream project reports these approximate results:

- CPU prediction latency around 16 to 32 ms, depending on the dataset.
- Model size around 2.3 to 2.6 MB.
- Peak training memory around 13 to 26 MB.
- 96.4 percent routing accuracy on 11,881 held-out SWE-bench rows.

Those numbers are about routing decisions, not final agent answer quality.

They mean the policy often picks the right next mechanical action. They do not mean it can reason like an LLM.

The upstream Hermes stress-test numbers vary across the project docs. Treat them as something to verify locally. Browser export is also a boundary case: the CPU adapter can prepare a structured export spec, but real browser login, navigation, and DOM work still belong to the Hermes browser controller.

## Requirements

Use this on a development VPS or local test machine.

You need:

- Ubuntu or another Linux environment.
- Python 3.10 or newer.
- Git.
- A working Hermes Agent setup.
- A compatible HermesAgent-20 checkout if you want to use the adapter patch.
- Node.js 18 or newer for the HermesAgent-20 harness.

If you followed the main Hermes VPS guide, the Hermes installer may already have installed several useful dependencies.

## Step 1, Clone busyBee-cpu

Connect to your VPS as the `hermes` user.

Then run:

```bash
cd ~
git clone https://github.com/DJLougen/busyBee-cpu.git
cd busyBee-cpu
```

## Step 2, Create the Python environment

Run:

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -U pip
pip install -e ".[dev]"
```

Check that the install works:

```bash
python -m pytest
```

The exact number of tests may change as the project changes. The important result is that the test run passes.

## Step 3, Train a policy model

Train a small policy model from the included examples:

```bash
bee-train \
  --train examples/train.jsonl \
  --eval examples/eval.jsonl \
  --model-out runs/policy.joblib
```

If you are experimenting seriously, also run cross-validation:

```bash
bee-train \
  --train examples/train.jsonl \
  --eval examples/eval.jsonl \
  --model-out runs/policy.joblib \
  --cv 3
```

## Step 4, Start the CPU policy server

If Hermes and busyBee-cpu are running on the same machine, keep the server local:

```bash
bee-serve \
  --model runs/policy.joblib \
  --host 127.0.0.1 \
  --port 8767 \
  --exposed-model busybee-cpu
```

In another terminal, check it:

```bash
curl http://127.0.0.1:8767/health
```

You should see JSON showing that the server is OK and exposing the `busybee-cpu` model.

Do not expose this service publicly unless you know exactly why. If Docker needs to reach it from another container, bind carefully and firewall the port.

## Step 5, Optional, run it in the background

For a quick test, foreground mode is better because you can see errors.

If you want to background it temporarily:

```bash
nohup bee-serve \
  --model runs/policy.joblib \
  --host 127.0.0.1 \
  --port 8767 \
  --exposed-model busybee-cpu \
  > busybee-server.log 2>&1 &
```

Check it again:

```bash
curl http://127.0.0.1:8767/health
```

## Step 6, Patch a compatible HermesAgent-20 checkout

This step is only for a compatible HermesAgent-20 development checkout.

It is not part of the normal Hermes VPS install.

From your HermesAgent-20 checkout, run:

```bash
cd /path/to/HermesAgent-20
git apply /home/hermes/busyBee-cpu/integrations/hermesagent20/busybee-cpu-adapter.patch
```

If the patch does not apply cleanly, stop and inspect the conflict. Do not force it into an unrelated Hermes version.

If the project has drifted but the patch is still close, you can try:

```bash
git apply --3way /home/hermes/busyBee-cpu/integrations/hermesagent20/busybee-cpu-adapter.patch
```

Then install and build the harness:

```bash
npm install
npm run build:benchlocal
```

## Step 7, Run a HermesAgent-20 smoke test

Set the base URL:

```bash
export BASE_URL=http://127.0.0.1:8767/v1
```

Run a small smoke test:

```bash
npm run dev:run -- \
  --scenario HA-05 \
  --scenario HA-06 \
  --scenario HA-13 \
  --scenario HA-18 \
  --scenario HA-20 \
  --provider busybee-cpu \
  --model busybee-cpu \
  --provider-model busybee-cpu \
  --label busyBee-cpu \
  --base-url "$BASE_URL" \
  --auth-mode none \
  --json \
  --build-image
```

If that works, run the full benchmark:

```bash
npm run dev:run -- \
  --all \
  --provider busybee-cpu \
  --model busybee-cpu \
  --provider-model busybee-cpu \
  --label busyBee-cpu \
  --base-url "$BASE_URL" \
  --auth-mode none \
  --json \
  --build-image
```

Read the result carefully.

Do not assume the upstream result will exactly match your server, your Hermes checkout, or your local patch state.

## Step 8, Run the busyBee benchmark

From the `busyBee-cpu` directory:

```bash
cd ~/busyBee-cpu
. .venv/bin/activate
python scripts/benchmark.py
```

This measures training time, memory use, model size, prediction latency, throughput, and resolver latency.

Use your own result as the source of truth for your VPS.

## Step 9, Roll back

To remove the HermesAgent-20 patch:

```bash
cd /path/to/HermesAgent-20
git apply -R /home/hermes/busyBee-cpu/integrations/hermesagent20/busybee-cpu-adapter.patch
```

If that does not apply because files changed after patching, inspect the Git diff and restore only the files changed by the adapter patch.

To stop the policy server:

```bash
pkill -f "busybee_cpu.server"
```

If you started it in a foreground terminal, press:

```text
Ctrl+C
```

## Practical recommendation

For a normal Hermes VPS, skip this.

For a development harness, it is worth testing if your goal is to reduce unnecessary LLM calls on mechanical agent turns.

Keep your main hosted model configured. Keep approval and safety gates enabled. Treat busyBee-cpu as a local routing helper, not as the brain of the agent.

## Sources

- busyBee-cpu: https://github.com/DJLougen/busyBee-cpu
- Hermes integration notes: https://raw.githubusercontent.com/DJLougen/busyBee-cpu/main/docs/hermes-agent.md
- busyBee-cpu setup guide: https://raw.githubusercontent.com/DJLougen/busyBee-cpu/main/docs/HERMES_HARNESS_SETUP.md
- Evaluation report: https://raw.githubusercontent.com/DJLougen/busyBee-cpu/main/reports/honest_evaluation.md
- Hermes Agent installation: https://hermes-agent.nousresearch.com/docs/getting-started/installation
