# Working in this repo

This repo (`pc-123z/vs-code`) holds the Home Assistant config (`homeassistant/config/automations.yaml`, etc.) for the user's house. It is a **git checkout only** — editing it here does not touch the live, running Home Assistant instance.

## Live vs. cloud sessions

- A Claude Code session running in an ephemeral cloud container (e.g. started from claude.ai/code web) has **no network path to the user's home server**. It can read/edit files and push commits, nothing more. There is no docker, ssh, or other live connection available from that kind of session.
- The user has Claude Desktop / Claude Code running locally on their own machines, which **does** have access to the box actually running Home Assistant (referred to as "docker2"). Only a session on one of those local instances can apply config changes to the live system, restart/reload automations, or call live HA services.
- **If you are a cloud/container session and the user asks you to apply, deploy, enable, or otherwise act on the *live* config** (not just edit files and commit): say so immediately, before doing anything else — don't wait to be asked "are you connected to docker2?" Tell them this session can only edit and push, and that they should use a Claude Code session running on their local machine (Claude Desktop) for anything that needs to touch the running system.
- Do not assume pushing a commit to GitHub gets picked up by the live instance automatically — the user has said this is "useless" for their workflow. Applying changes to the running config is a manual/local step outside what a cloud session can trigger.
