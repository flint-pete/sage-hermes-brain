# Sage + Hermes Brain

A transplantable knowledge bundle ("brain") for students working on **Sage /
Waggle** edge-computing projects with
**[Hermes Agent](https://hermes-agent.nousresearch.com/docs/)**.

It packages the accumulated, hard-won knowledge from real Sage plugin development —
camera/audio capture, pywaggle, node identity & GPS, ECR/deploy pipelines, BirdNET
and other ML plugins, job scheduling, on-node debugging — as a Hermes **skill** you
load into your *own* agent, plus the current pywaggle2 design docs for context.

> This bundle contains **no credentials and no personal data**. Anywhere a real
> secret would go, you'll see a placeholder like `CAMERA_USER` / `CAMERA_PASSWORD`
> / `CAMERA_IP:PORT` / `<YOUR_SAGE_TOKEN_FILE>`. Get the real values from your
> instructor or the node owner — never hard-code a credential into a skill or repo.

## What's inside

```
skills/sage-waggle/     The Sage/Waggle skill: SKILL.md + ~90 reference files.
                        This is the "brain" — load it and the agent knows how Sage
                        plugins actually work (and the pitfalls we already hit).
                        Covers camera/audio capture, pywaggle2, node identity/GPS,
                        ECR + native-node deploy, ML plugins (YOLO/BioCLIP/BirdNET),
                        the cache-mediated detect→classify cascade, job scheduling,
                        data viz, and on-node debugging.
docs/                   Design docs for context / reading:
  pywaggle2-design.md     pywaggle2 library redesign (camera acquisition, node
                          identity + GPS, sentinels/None normalization, 2-tier GPS).
  local-cache-design.md   Shared /local-cache design (library primitive + WES mgr).
  project-status.txt      Where the project stands.
  Infra-problems-to-fix.md  Running list of infra issues to file/fix.
MEMORY.starter.md       Optional, sanitized domain facts you may seed into your
                        own Hermes memory (~/.hermes/memories/MEMORY.md).
```

## Prerequisites

- Hermes Agent installed and working: <https://hermes-agent.nousresearch.com/docs/>
  ```bash
  curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
  hermes setup        # pick a model/provider, add your own API key
  ```
- Your own model/provider API key (this bundle ships none — you use yours).

## Sage access setup (do this before your first plugin task)

The skill knows *how* Sage works, but you need your own access to actually touch
nodes and data. Set these up once:

1. **Sage portal account.** Sign in at <https://portal.sagecontinuum.org> (Globus /
   institutional login). This is your identity for nodes, data, and the app
   registry (ECR).
2. **Portal access token** (for downloading protected data). Generate one at
   <https://portal.sagecontinuum.org/account/access>. Your **username** is your
   portal username (not your GitHub name). Keep the token in a file you control —
   e.g. `~/.sage/token.txt` — and NEVER commit it. (The skill refers to this as
   `<YOUR_SAGE_TOKEN_FILE>`.)
3. **Node SSH access** (only if your class works directly on nodes). Access is
   granted per-node by the node owner / your instructor — ask them for the exact
   `ssh` route and any credentials. Node LAN addresses (like `NODE_CONTROL_PLANE_IP`) and
   camera creds come from them, never hard-coded here.
4. **Wire up the Sage MCP server** so your agent can query nodes, sensors, images,
   and docs directly. Read-only tools need no token:
   ```bash
   printf 'n\nY\n' | hermes mcp add sage --url "https://mcp.sagecontinuum.org/mcp"
   hermes mcp list          # confirm 'sage' is connected
   # then start a NEW hermes session so the tools load
   ```
   (Job-submission tools need your portal Bearer token; read-only data/query tools
   do not. See `skills/sage-waggle/references/mcp-tools.md`.)

Docs for all of the above: <https://sagecontinuum.org> and the Sage docs linked
from the portal.

## Install the skill (two ways)

### A) Skill tap (recommended — stays updatable)

Point Hermes at this repo as a skill source, then install:

```bash
hermes skills tap add flint-pete/sage-hermes-brain
hermes skills install sage-waggle
hermes skills list          # confirm it's there
```

Later, pull updates with:
```bash
hermes skills check && hermes skills update
```

### B) Direct copy (quick, no tap)

```bash
git clone https://github.com/flint-pete/sage-hermes-brain.git
cp -r sage-hermes-brain/skills/sage-waggle ~/.hermes/skills/
# in a Hermes session:  /reload-skills
```

## Verify it loaded (smoke test)

After installing, confirm the skill and (optionally) the Sage MCP are wired up:

```bash
hermes skills list | grep sage-waggle        # skill is installed
hermes mcp list                              # 'sage' shows connected (if you added it)
```

Then start a session and ask a question only the skill would know:

```bash
hermes -s sage-waggle
```
> Ask: **"Using the sage-waggle skill, how do I chain a detector and a classifier
> on a Sage node without any cross-plugin calls?"**

If the agent explains the cache-mediated detect→classify cascade — a detector
crops each detection into a new `/local-cache` stream as a v2 frame, and the
classifier consumes those crops via a single `--input` path (raw stream vs. crop
stream), all frame-anchored with provenance — the skill is loaded and working. If
it gives a generic answer, run `/reload-skills` (or `/skill sage-waggle`) and retry.

## Your first task (guided walkthrough)

A gentle on-ramp — each step builds on the last. Run these as prompts inside
`hermes -s sage-waggle`:

1. **Orient.** *"Give me a 5-bullet overview of what a Sage/Waggle plugin is and
   the lifecycle from code to running on a node."* — sanity-check your mental model.
2. **Explore live data (needs the Sage MCP).** *"List a few available Sage nodes
   and show the latest temperature readings from one of them."* — confirms your MCP
   access works end to end.
3. **Read a real design.** *"Summarize `docs/pywaggle2-design.md` — specifically
   how a plugin should get its node's VSN and GPS location."* — grounds you in the
   current thinking (point the agent at the docs/ folder if needed).
4. **Build something small.** *"Help me scaffold a minimal plugin that captures one
   camera snapshot and prints its size — using placeholder camera credentials I'll
   fill in from my instructor."* — your first hands-on artifact.
5. **Learn the pitfalls before you hit them.** *"What are the top 5 mistakes people
   make deploying Sage plugins, from the sage-waggle skill?"* — cheap insurance.

From here, pick a real class assignment and let the skill guide the specifics; its
~90 reference files cover cameras, audio/BirdNET, ML plugins, scheduling, ECR/deploy,
and on-node debugging.

## Use it (reference)

Load the skill into any session:

```bash
hermes -s sage-waggle
# or, inside a running session:
/skill sage-waggle
```

Then ask it Sage things — e.g. "help me write a plugin that captures a snapshot
from a Reolink camera and runs BirdNET," or "how do I deploy a plugin to a node
when ECR builds are broken?" The skill's references carry the specifics.

## (Optional) Seed the starter memory

`MEMORY.starter.md` holds a few generally-true, non-sensitive Sage facts. If you
want them always in context, append them to your own memory file:

```bash
cat MEMORY.starter.md >> ~/.hermes/memories/MEMORY.md
```

Keep it short — Hermes memory has a small char budget and is injected every turn.

## The design docs

`docs/` are plain Markdown — read them, or drop them into your project workspace so
your agent can reference them. `pywaggle2-design.md` is the current design for the
next-generation Waggle Python library and is the best single overview of node
identity, GPS resolution, and camera acquisition as we currently understand them.

---

*Everything here is knowledge, not secrets. Bring your own keys, your own node
access, and your own credentials.*
