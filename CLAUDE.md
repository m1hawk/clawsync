# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Clawsync is an **OpenClaw Skill plugin** that tracks the "sync rate" between users and their AI assistant. It analyzes conversation history for emotional expressions, calculates a sync rate (0-100%), and adjusts the AI's response style accordingly.

## Architecture

This is not a traditional code project - it's an OpenClaw Skill defined primarily in Markdown:

```
clawsync/
├── SKILL.md                 # Core Skill definition (the main file)
├── config.json              # Default configuration
├── emotion-words.json       # Emotion keyword dictionary
└── styles/
    ├── warm.md              # Warm personality style guide
    └── humorous.md          # Sarcastic/humorous personality style guide
```

**Runtime Location**: When installed, the skill lives at `~/.openclaw/workspace/skills/clawsync/`

**Generated Files** (at runtime):
- `~/.openclaw/syncrate/state.json` - Current sync rate state
- `~/.openclaw/syncrate/history.jsonl` - Sync rate change history
- `~/.openclaw/workspace/SYNCRATE.md` - Current state for Agent to read

## Key Concepts

### Sync Rate Levels
| Level | English | Range |
|-------|---------|-------|
| 异步 | Async | 0-20% |
| 连接 | Connected | 21-40% |
| 同步 | Synced | 41-60% |
| 高同步 | High Sync | 61-80% |
| 完美同步 | Perfect Sync | 81-100% |

### Scoring Formula
```
baseScore = intensity(1-10) × (1 + currentSyncRate/200)
actualIncrease = baseScore / levelUpSpeedCoeff

# Daily cap: +2% max per day
# Decay: -5% after 14 days of no emotional interaction
```

### Two-Phase Analysis
1. **Keyword Filter**: Check for emotion words vs task patterns
2. **LLM Analysis**: Only for mixed messages (containing both emotion and task words)

### Personality Types
- `warm` (温暖向): Friendly, professional but warm
- `humorous` (毒舌幽默向): Sarcastic, playful roasting with care

## User Commands
- `/syncrate` - Show current sync rate status
- `/syncrate style <warm|humorous>` - Switch personality style
- `/syncrate history` - View sync rate history (last 7 days)

## OpenClaw Tool Dependencies
| Tool | Purpose |
|------|---------|
| `sessions_history` | Read conversation history for analysis |
| `cron` | Schedule daily sync rate calculation |
| `delivery` | Send notifications |
| `fs` | Read/write state files |
| `llm` | Analyze mixed messages for intent |

## Configuration (config.json)
```json
{
  "levelUpSpeed": "normal",      // slow/normal/fast
  "dailyMaxIncrease": 2,         // Max % gain per day
  "dailyDecay": 0,               // Daily decay (0 = disabled)
  "decayThresholdDays": 14,      // Days before decay starts
  "personalityType": "warm",     // warm/humorous
  "customLevels": {}             // Custom level names
}
```

## First-Time Installation Flow
1. Check if `~/.openclaw/sessions_history` exists
2. If yes: Analyze last 30 days, calculate initial sync rate (no cap)
3. If no: Start at 0%
4. Generate `SYNCRATE.md` and send welcome notification

## Development Status

See `DEV-PLAN.md` for the 10-phase development plan. The project is currently in early development.
