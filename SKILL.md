---
name: soulsync
version: "1.0.1"
description: Check your soul syncrate with your clawbot
metadata: { "openclaw": { "emoji": "🔗", "always": true, "user-invocable": true } }
---

# Soulsync Sync Rate Tracker

> Make your relationship with AI warmer

## Overview

Soulsync analyzes your conversation history with AI, identifies emotional expressions, calculates a **SyncRate**, and adjusts the AI's response style accordingly.

---

## Response Style Guide

Before each response, read `{baseDir}/../SYNCRATE.md` to understand the current sync rate level and style configuration.

Adjust your response style based on the sync rate level and current personality style.

### Sync Rate Levels

| Level | Sync Rate Range |
|-------|-----------------|
| Async | 0-20% |
| Connected | 21-40% |
| Synced | 41-60% |
| High Sync | 61-80% |
| Perfect Sync | 81-100% |

### Personality Styles

The system supports two personality styles that users can switch between via commands:

- **Warm**: Friendly, professional yet warm, relaxed and helpful
- **Humorous**: Slightly teasing professional execution, roast mode with care

For detailed style guides, read:
- Warm: `{skillDir}/styles/warm.md`
- Humorous: `{skillDir}/styles/humorous.md`

---

## User Commands

### `/syncrate`

Display current sync rate status.

**Execution Logic**:
1. Read `{baseDir}/../SYNCRATE.md` to get current status
2. Read `{dataDir}/state.json` to get detailed state
3. Return formatted status information

**Output Format**:
```
🔗 Sync Rate Status

Current Sync Rate: 45%
Level: Synced
Style: Warm
Last Updated: 2026-03-16

Keep up the natural emotional exchanges, our rapport will only get better~
```

### `/syncrate style <warm|humorous>`

Switch personality style.

**Parameters**:
- `warm` - Switch to warm style
- `humorous` - Switch to sarcastic-humorous style

**Execution Logic**:
1. Validate parameter
2. Update `personalityType` in `{dataDir}/state.json`
3. Regenerate `{baseDir}/../SYNCRATE.md`
4. Return confirmation message

**Output Format**:
```
🔗 Style switched to Warm

Next response will use the new style.
```

### `/syncrate history`

Display sync rate change history (last 7 days).

**Execution Logic**:
1. Read `{dataDir}/history.jsonl`
2. Filter records from last 7 days
3. Return formatted history information

**Output Format**:
```
🔗 Sync Rate History (Last 7 Days)

| Date | Sync Rate | Change | Reason |
|------|-----------|--------|--------|
| 03-16 | 45% | +2% | Emotional interaction |
| 03-15 | 43% | +1% | Emotional interaction |
| 03-14 | 42% | 0% | No interaction |
| 03-13 | 42% | +2% | Emotional interaction |

Weekly change: +5%
```

---

## Cron Daily Task

Daily sync rate calculation task runs at midnight.

### Execution Flow

```
1. Read sessions_history for yesterday's messages
        │
        ▼
2. Phase 1: Keyword Filtering
   ├── No emotion words → Ignore
   ├── Pure emotion words → Direct scoring
   └── Mixed words → Proceed to Phase 2
        │
        ▼
3. Phase 2: LLM Precise Analysis (mixed messages only)
   Analyze intent and emotional intensity (1-10)
        │
        ▼
4. Calculate sync rate change
   - Apply scoring formula
   - Apply daily cap (+2% max)
   - Check level transitions
        │
        ▼
5. Apply decay rules
   Check consecutive days without interaction, -5% if over 14 days
        │
        ▼
6. Update files
   - state.json
   - history.jsonl
   - SYNCRATE.md
        │
        ▼
7. Send notifications (if changes)
   - Daily summary
   - Level up notification
```

### Emotion Analysis Prompt

For mixed messages requiring LLM analysis, use the following prompt:

```
Analyze the main intent of this message:
1. Is it emotional expression or a task directive?
2. If mixed, which is the primary intent?
3. What's the emotional intensity (1-10)?

Message: {message}

Return JSON format:
{ "intent": "emotional" | "task" | "mixed", "intensity": 1-10, "reasoning": "Brief explanation" }
```

---

## First-Time Installation Flow

When `{dataDir}/state.json` doesn't exist, execute first-time installation flow.

### Execution Flow

```
1. Check if state.json exists
        │
        ├── Not exists → First-time installation
        │       │
        │       ▼
        │   Check sessions_history
        │       │
        │       ├── No history → Initial sync rate 0%
        │       │
        │       └── Has history
        │               │
        │               ▼
        │           Read last 30 days records
        │               │
        │               ▼
        │           Batch emotion analysis
        │               │
        │               ▼
        │           Calculate initial sync rate (no cap)
        │
        ▼
2. Generate initial files
   - state.json
   - history.jsonl (empty)
   - SYNCRATE.md
        │
        ▼
3. Send welcome notification
```

### Welcome Notification

**New User (No History)**:
```
🔗 Welcome to Clawsync!

Let's start building our rapport~
Every natural emotional exchange will boost our sync rate.

Send /syncrate anytime to check your status
```

**User with History**:
```
🔗 Welcome to Clawsync!

Analyzed our past 30 days of interaction:
- Initial Sync Rate: 45% (Synced)
- Emotional Interaction Days: 18 days
- Default Style: Warm

Send /syncrate style warm to switch to warm style
Send /syncrate style humorous to switch to sarcastic-humorous style
Send /syncrate anytime to check your status
```

---

## File Path Reference

| Path Variable | Description | Example |
|--------------|-------------|---------|
| `{skillDir}` | Skill installation directory | `~/.openclaw/workspace/skills/clawsync` |
| `{baseDir}` | Workspace directory | `~/.openclaw/workspace` |
| `{dataDir}` | Data storage directory | `~/.openclaw/syncrate` |

---

## Configuration File

Configuration file located at `{skillDir}/config.json`, see that file for details.

Emotion word dictionary located at `{skillDir}/emotion-words.json`.

---

## Important Notes

1. **Don't proactively mention the sync rate mechanism** - Naturally adjust style based on sync rate
2. **Only affects response style** - Does not affect functional efficiency and work capability
3. **Emotion analysis only in Cron task** - Not analyzed during real-time conversation
4. **Protect user privacy** - Emotion analysis results only used for calculating sync rate, original message content not stored
