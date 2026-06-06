# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## Project

Arcade Vault — an online gaming platform where users play and compete for points. Uses **Spec Driven Design** via the `/spec` and `/spec-impl` skills.

Specs live in `specs/` (01–09 implemented). New features always start with a spec.

## Stack

- **Next.js 16.2.6** with App Router (`app/` directory) — read `node_modules/next/dist/docs/` before writing Next.js code; APIs differ from training data
- **React 19.2.4**
- **Tailwind CSS v4** (PostCSS plugin via `@tailwindcss/postcss`)
- **TypeScript**
- **Supabase** (`@supabase/ssr` + `@supabase/supabase-js`) — project ref `ttgbmvotgdldwnjqltoe`
- **Resend** — email for the contact form (`app/api/contact/route.ts`)

No test runner is configured yet.

## Skills

Always use `/frontend-design` for UI design.

| Skill        | Source                                                      | Purpose                                 |
| ------------ | ----------------------------------------------------------- | --------------------------------------- |
| `/spec`      | `.claude/skills/spec` (from `Klerith/fernando-skills`)      | Write a new spec section-by-section     |
| `/spec-impl` | `.claude/skills/spec-impl` (from `Klerith/fernando-skills`) | Implement an approved spec step-by-step |
| `/add-game`  | `.claude/skills/add-game` (local)                           | Generate a spec for a new canvas game   |

## Hooks

`PostToolUse` on Write/Edit/MultiEdit → `.claude/hooks/format-and-lint.sh` — runs Prettier then ESLint --fix on the saved file automatically.

## MCP

Supabase MCP (`mcp__supabase__*`) is connected. Use it for SQL migrations, type generation, and logs. Enabled via `.mcp.json` + `.claude/settings.local.json`.

## Architecture

Uses Next.js **App Router** exclusively — no `pages/` directory.

### Entry points

- `app/layout.tsx` — root layout with Geist font variables and global CSS
- `app/globals.css` — Tailwind base styles
- `app/page.tsx` — home route (`/`)

### Routes implemented

| Route               | File                           | Notes                             |
| ------------------- | ------------------------------ | --------------------------------- |
| `/`                 | `app/page.tsx`                 | Landing                           |
| `/about`            | `app/about/page.tsx`           | About + contact form              |
| `/auth`             | `app/auth/page.tsx`            | Auth page                         |
| `/games`            | `app/games/page.tsx`           | Game grid (dynamic from Supabase) |
| `/games/[id]`       | `app/games/[id]/page.tsx`      | Game detail                       |
| `/games/[id]/play`  | `app/games/[id]/play/page.tsx` | Play wrapper                      |
| `/hall-of-fame`     | `app/hall-of-fame/page.tsx`    | Global leaderboard tabs           |
| `POST /api/contact` | `app/api/contact/route.ts`     | Resend email handler              |

### Key directories

- `components/` — shared UI; `Nav.tsx` + `components/games/` (one file per game)
- `lib/supabase/` — `client.ts`, `server.ts`, `types.ts`
- `app/data/` — static fallback data (`games.ts`, `scores.ts`)
- `app/context/UserContext.tsx` — React context for auth user
- `specs/` — spec documents (01-09 done)

### Supabase data model

```
GameRow:  id(string) | title | short | long | cat(ARCADE|PUZZLE|SHOOTER) | cover | color(cyan|magenta|yellow|green) | created_at
ScoreRow: id(string) | game_id | player_name | score(number) | user_id(string|null) | created_at
```

Tables `games` and `scores` exist in the remote project. **Do not recreate them.**

### Canvas game pattern

Each game is a React component in `components/games/<Name>Game.tsx` that accepts `{ paused, onScore, onGameOver }` props and manages a `<canvas>` via `useEffect`. Follow `AsteroidsGame.tsx` as the reference. Clean up keyboard listeners in the `useEffect` return.

New routes go under `app/` as folders with `page.tsx`. Server Components are the default; mark client components with `"use client"` only when needed.
