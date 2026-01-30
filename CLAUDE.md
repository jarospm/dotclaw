# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Documentation repo for running OpenClaw (personal AI assistant) on a VPS. No code — just markdown guides and references.

## Architecture

- **guides/** — First-time setup walkthroughs (server-setup, openclaw-setup)
- **reference/** — Day-to-day operations (commands, file structure)
- **local/** — Instance-specific config (gitignored) — IPs, credentials, bot settings

## Key Distinction

- **guides/** = one-time installation steps
- **reference/** = ongoing operations for existing setup
- Keep them separate — don't mix setup instructions into reference docs

## Sensitive Data

All credentials, IPs, and instance-specific config go in `local/` which is gitignored. Public files use placeholders like `<your-server>`, `<SERVER_IP>`, `TOKEN_1`.
