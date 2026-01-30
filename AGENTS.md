# Project Instructions (Cursor Agent)

## Language
- Always reply **in English** (documentation, comments, chat responses), unless an exact quote or code snippet requires otherwise.

## Source of Truth (check before making changes)
- Development guidelines: `CLAUDE.md`
- Add-on runtime / update logic: `moltbot_gateway/run.sh`
- Add-on metadata / options / schema: `moltbot_gateway/config.json`
- HA repository metadata: `repository.json`
- Docs: `README.md`, `INSTALLATION.md`, `CONFIGURATION.md`, `TROUBLESHOOTING.md`

## Working Principles
- **No secrets**: Never commit real API keys/tokens/passwords or put them in examples.
- **Conventions**: Use Conventional Commits (see `CLAUDE.md`).
- **Safety**: No destructive Git actions (force push, hard reset, history rewrite) unless explicitly requested.
- **Add-on stability**: Don’t “simplify” or remove the update/rollback/snapshot mechanics in `run.sh`. Changes must be targeted and clearly justified.

## Changes to Add-on Options / Docs
- If `moltbot_gateway/config.json` (options/schema) changes, update the docs to match.
- Always use placeholders in examples (e.g., `sk-ant-...`, `YOUR-HA-IP`).
