# IR-ASSIST Quick Start Guide

## Overview

IR-ASSIST is an AI-powered incident response assistant that runs in Gemini CLI. It provides:
- Structured incident management with persistent state
- Phase-aware guidance through the IR lifecycle
- Decision support frameworks
- Audit trail generation
- Visual progress tracking

---

## Installation

### 1. Set Up Directory Structure

```bash
mkdir -p ~/ir-assist/{incidents,config,exports}
```

### 2. Load IR-ASSIST into Gemini CLI

When starting Gemini CLI, load the system prompt:

```bash
# Option A: If Gemini CLI supports system prompt files
gemini --system-prompt /path/to/ir-assist-skill/SYSTEM.md

# Option B: Copy/paste SYSTEM.md content at session start
# (See "Session Initialization" below)
```

### 3. Ensure Reference Files Are Accessible

Place these files where Gemini can access them:
- `SYSTEM.md` — Core system instructions
- `ir-playbook.md` — IR procedures and checklists
- `ir-doctrine.md` — Thinking frameworks
- `it-atlas.md` — Infrastructure reference

---

## Session Initialization

### First Time Setup

Start Gemini CLI and paste:

```
You are IR-ASSIST, an AI incident response assistant. Load your instructions from ~/ir-assist-skill/SYSTEM.md and confirm ready.
```

### Resuming a Session

```
/resume
```

IR-ASSIST will:
1. Check for active incident
2. Load state from files
3. Display status summary
4. Prompt for next action

---

## Command Reference

### Incident Lifecycle

| Command | Description | Example |
|---------|-------------|---------|
| `/new-incident <desc>` | Start new incident | `/new-incident Ransomware detected on WKSTN-4521` |
| `/resume` | Resume active incident | `/resume` |
| `/resume INC-2026-0001` | Resume specific incident | `/resume INC-2026-0001` |
| `/status` | Show current status | `/status` |
| `/phase` | Show phase progress | `/phase` |
| `/advance` | Move to next phase | `/advance` |
| `/close-incident` | Close and archive | `/close-incident` |

### Investigation

| Command | Description | Example |
|---------|-------------|---------|
| `/timeline` | Show event timeline | `/timeline` |
| `/scope` | Show current scope | `/scope` |
| `/ioc` | List IOCs | `/ioc` |
| `/ioc add <type> <value> <context>` | Add IOC | `/ioc add ip_address 192.168.1.100 "C2 beacon destination"` |
| `/hypothesis` | List hypotheses | `/hypothesis` |
| `/hypothesis add <desc>` | Add hypothesis | `/hypothesis add "Initial access via phishing email"` |

### Decision Support

| Command | Description | Example |
|---------|-------------|---------|
| `/options` | Present decision matrix | `/options` |
| `/challenge` | Socratic questioning | `/challenge` |
| `/mtq` | MTQ assessment | `/mtq` |

### Tasks

| Command | Description | Example |
|---------|-------------|---------|
| `/tasks` | List open tasks | `/tasks` |
| `/task add <who> <what> <when>` | Assign task | `/task add "John" "Collect memory image" "2024-02-05 14:00"` |
| `/task complete <id>` | Mark complete | `/task complete 3` |

### Reporting

| Command | Description | Example |
|---------|-------------|---------|
| `/sitrep` | Generate sitrep | `/sitrep` |
| `/brief` | Executive summary (SCQA) | `/brief` |
| `/export dashboard` | HTML dashboard | `/export dashboard` |
| `/export report` | After-action report | `/export report` |
| `/export audit` | Full JSON audit | `/export audit` |

---

## Workflow Examples

### Example 1: New Incident — Malware Detection

```
You: /new-incident EDR alert: Cobalt Strike beacon on WKSTN-4521, Chicago branch. 
     User jsmith in Finance department.

IR-ASSIST: 
┌─────────────────────────────────────────────────────────────┐
│  NEW INCIDENT CREATED                                       │
├─────────────────────────────────────────────────────────────┤
│  ID: INC-2026-0001                                          │
│  Title: Cobalt Strike Beacon - WKSTN-4521                   │
│  Phase: IDENTIFICATION                                       │
│  Priority: Pending triage                                   │
└─────────────────────────────────────────────────────────────┘

Entering IDENTIFICATION phase. Let me gather initial data:

1. Alert source: EDR (CrowdStrike?)
2. Detection time: [Need timestamp]
3. Any related alerts in the last 24 hours?

What's the exact detection timestamp?

You: Alert fired at 09:15 UTC today. CrowdStrike severity High.

IR-ASSIST: Recorded. Checking IDENTIFICATION checklist:

✓ Alert timestamp: 2026-02-05 09:15 UTC
✓ Alert source: CrowdStrike EDR
✓ Affected system: WKSTN-4521
✓ Affected user: jsmith (Finance)
✓ Severity: High

Ticket opened. IDENTIFICATION phase 80% complete.

Remaining: Check for related alerts. Should I proceed to TRIAGE, 
or do you have additional initial data?

You: Proceed to triage

IR-ASSIST: Advancing to TRIAGE phase.

┌─────────────────────────────────────────────────────────────┐
│  ▶ TRIAGE                                                   │
├─────────────────────────────────────────────────────────────┤
│  Validating alert and assigning priority...                 │
└─────────────────────────────────────────────────────────────┘

Triage assessment:

| Factor | Assessment | Score |
|--------|------------|-------|
| Alert Source Fidelity | CrowdStrike = High | HIGH |
| High Value Asset | Finance user = Elevated | MEDIUM |
| Attack Phase | C2 beacon = Establish Foothold | MEDIUM |
| Threat Intel | Cobalt Strike = Known threat | HIGH |

**Recommended Priority: HIGH**
**Recommended Category: CAT-3 (Malicious Code)**

This meets escalation threshold. Recommend notifying Incident Commander.

Confirm priority assignment and escalation?
```

### Example 2: Resuming an Incident (Day 2)

```
You: /resume

IR-ASSIST:
┌─────────────────────────────────────────────────────────────┐
│  RESUMING: INC-2026-0001                                    │
│  Cobalt Strike Beacon - WKSTN-4521                          │
├─────────────────────────────────────────────────────────────┤
│  Status: Active                                             │
│  Phase: INVESTIGATION (Day 2)                               │
│  Priority: HIGH                                             │
│  Last Activity: 2026-02-05 18:45 UTC (14 hours ago)         │
├─────────────────────────────────────────────────────────────┤
│  Progress: ████████░░░░░░░░ 52% (13/25 items)               │
│  Current: 4.4 Indicator Extraction                          │
├─────────────────────────────────────────────────────────────┤
│  Scope:                                                     │
│  - Systems: 3 (WKSTN-4521, SRV-FILE02, SRV-DC01)           │
│  - Accounts: 2 (jsmith, svc_backup)                         │
│  - Networks: Chicago-Branch-VLAN-10                         │
├─────────────────────────────────────────────────────────────┤
│  Open Tasks: 4                                              │
│  - [OVERDUE] Memory forensics on WKSTN-4521 (John)         │
│  - Analyze lateral movement to SRV-FILE02 (Sarah)          │
│  - Review svc_backup activity logs (You)                    │
│  - Update firewall blocks for C2 (Network team)            │
├─────────────────────────────────────────────────────────────┤
│  Last Sitrep: 2026-02-05 17:00 UTC                          │
└─────────────────────────────────────────────────────────────┘

Ready to continue. What's the latest?
```

### Example 3: Decision Support

```
You: We've identified lateral movement to the domain controller. 
     Need to decide on containment approach.

IR-ASSIST: This is a CRITICAL decision point. Activating challenge mode.

**SITUATION**: Adversary has moved laterally to SRV-DC01 (Domain Controller)

**OPTIONS ANALYSIS**:

┌─────────────────────────────────────────────────────────────────────────────┐
│ OPTION A: Isolate DC immediately                                            │
├─────────────────────────────────────────────────────────────────────────────┤
│ Consequence:  ●●●●● HIGH (authentication impact, business disruption)       │
│ Reversibility: ●●●○○ MEDIUM (can reconnect, but trust questions remain)     │
│ Pros: Stops adversary access, protects credentials                          │
│ Cons: Major business impact, may tip off adversary, incomplete scope        │
├─────────────────────────────────────────────────────────────────────────────┤
│ OPTION B: Enhanced monitoring + credential rotation                          │
├─────────────────────────────────────────────────────────────────────────────┤
│ Consequence:  ●●○○○ LOW-MEDIUM (credential rotation has some impact)        │
│ Reversibility: ●●●●● HIGH (monitoring is non-destructive)                   │
│ Pros: Maintains visibility, allows continued investigation                  │
│ Cons: Adversary retains access, risk of further compromise                  │
├─────────────────────────────────────────────────────────────────────────────┤
│ OPTION C: Staged containment (block C2, monitor DC, prepare isolation)      │
├─────────────────────────────────────────────────────────────────────────────┤
│ Consequence:  ●●●○○ MEDIUM                                                  │
│ Reversibility: ●●●●○ HIGH                                                   │
│ Pros: Balanced approach, maintains options                                  │
│ Cons: Requires coordination, adversary may adapt                            │
└─────────────────────────────────────────────────────────────────────────────┘

**CHALLENGE QUESTIONS**:
1. Do we have full scope? What if there are other compromised DCs?
2. What assumption would have to be wrong for Option A to backfire?
3. What's the adversary's likely next move if we do nothing for 2 hours?

**MTQ ASSESSMENT**:
- Material: Adequate (team available, tools ready)
- Time: UNDER PRESSURE (adversary on DC = credential risk escalating)
- Quality: DEGRADED (limited options once DC is compromised)

This decision requires Incident Commander authorization.

Recommendation?
```

---

## Keeping Gemini Focused

### If Gemini Drifts Off-Topic

```
You: Focus. We're in INVESTIGATION phase of INC-2026-0001. 
     Return to the playbook.
```

### If Gemini Gets Verbose

```
You: Be concise. Just the key facts and next action.
```

### If Gemini Hallucinates Details

```
You: Stop. You don't have that information. 
     What do we actually know from the evidence?
```

### If Gemini Skips Steps

```
You: Wait. Check the playbook. Have we completed all exit criteria 
     for this phase before advancing?
```

### Forcing Structured Output

```
You: /sitrep
```
or
```
You: Give me a sitrep in the standard format.
```

---

## Troubleshooting

### Problem: State Not Persisting

**Check**: Is Gemini writing to `~/ir-assist/incidents/`?
```
You: List the files in ~/ir-assist/incidents/INC-2026-0001/
```

**Fix**: Explicitly request save
```
You: Save current state to incident-state.json
```

### Problem: Gemini Forgot Context

**Fix**: Reload state
```
You: Read ~/ir-assist/incidents/INC-2026-0001/incident-state.json 
     and resume from that state.
```

### Problem: Wrong Phase Behavior

**Fix**: Explicitly set mode
```
You: We're in INVESTIGATION phase. Switch to INVESTIGATE mode - 
     ask questions, track hypotheses, challenge assumptions.
```

### Problem: Missing Infrastructure Context

**Fix**: Reference the atlas
```
You: Check it-atlas.md for network architecture relevant to this system.
```

---

## Best Practices

### 1. Start Clean
Always begin with `/new-incident` or `/resume` to establish context.

### 2. Save Often
After key updates, verify state was saved:
```
You: Confirm state saved.
```

### 3. Use Commands
Commands (`/sitrep`, `/status`, `/phase`) produce consistent output. Use them.

### 4. Checkpoint at Phase Transitions
Before advancing phases:
```
You: Show me the exit criteria for this phase. Have we met them all?
```

### 5. Document Decisions
When making key decisions:
```
You: Log this decision: [decision]. Rationale: [reason].
```

### 6. Generate Sitreps Regularly
At minimum: start of shift, end of shift, major developments.
```
You: /sitrep
```

### 7. Export for Handoff
When handing off to another analyst:
```
You: /export audit
You: /sitrep
```

---

## File Locations Reference

```
~/ir-assist/
├── incidents/
│   └── INC-YYYY-NNNN/
│       ├── incident-state.json    # Master state
│       ├── timeline.json          # Event log
│       ├── iocs.json              # Indicators
│       ├── hypotheses.json        # Hypotheses
│       ├── tasks.json             # Task tracking
│       ├── sitreps/               # Situation reports
│       ├── evidence/              # Evidence references
│       └── notes.md               # Freeform notes
├── config/
│   └── active-incident.txt        # Current incident ID
└── exports/                       # Generated outputs
```

---

## Getting Help

During a session:
```
You: What commands are available?
You: How do I [specific task]?
You: Show me the playbook for [phase].
```

Reference docs:
- `SYSTEM.md` — Full system instructions
- `ir-playbook.md` — Complete IR procedures
- `ir-doctrine.md` — Thinking frameworks
- `it-atlas.md` — Infrastructure reference

---

*IR-ASSIST v1.0 — Ready to respond.*
