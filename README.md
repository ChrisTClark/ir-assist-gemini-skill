# IR-ASSIST Skill Package

**Version:** 1.0  
**Target Platform:** Gemini CLI (Gemini 2.5 Pro)  
**Purpose:** AI-powered incident response assistant for banking cybersecurity operations

---

## Overview

IR-ASSIST is a comprehensive incident response system designed to run as a conversational AI assistant within Gemini CLI. It provides:

- **Structured IR Lifecycle Management** — Phase-aware guidance from detection to closure
- **Persistent State Tracking** — File-based incident state that survives session restarts
- **Decision Support Frameworks** — 11 thinking models for high-stakes decisions
- **Infrastructure Context** — Bank technology atlas for investigation guidance
- **Audit Trail Generation** — JSON-based logging for compliance and review
- **Visual Progress Tracking** — Phase indicators, checklists, and dashboards

### Core Philosophy

> **"Too fast for humans, too consequential for machines."**

IR-ASSIST handles velocity (information processing, playbook execution, documentation) while preserving human authority over consequential decisions.

---

## Package Contents

```
ir-assist-skill/
├── README.md                 # This file
├── QUICKSTART.md             # User guide with commands and examples
├── SYSTEM.md                 # Core system prompt for Gemini
├── ir-playbook.md            # IR procedures, phases, checklists
├── ir-doctrine.md            # 11 thinking frameworks
├── it-atlas.md               # Bank infrastructure reference
├── templates/
│   ├── incident-state-schema.json
│   ├── timeline-schema.json
│   ├── iocs-schema.json
│   ├── hypotheses-schema.json
│   └── tasks-schema.json
└── examples/
    └── incident-state-example.json
```

### File Descriptions

| File | Purpose |
|------|---------|
| `SYSTEM.md` | Load this as Gemini's system prompt. Contains identity, commands, behavioral rules. |
| `ir-playbook.md` | Complete IR procedures with decision trees and checklists for each phase. |
| `ir-doctrine.md` | Thinking frameworks: MTQ, SCQA, Consequence×Reversibility, Socratic challenge, etc. |
| `it-atlas.md` | Reference architecture for typical large bank infrastructure. Customize for your environment. |
| `QUICKSTART.md` | User guide with command reference, workflow examples, troubleshooting. |
| `templates/*.json` | JSON schemas for structured data (incident state, timeline, IOCs, etc.) |
| `examples/` | Sample files showing expected formats |

---

## Installation

### 1. Create Working Directory

```bash
mkdir -p ~/ir-assist/{incidents,config,exports}
```

### 2. Copy Skill Files

Place the skill package files where Gemini CLI can access them:

```bash
cp -r ir-assist-skill ~/ir-assist-skill
```

### 3. Initialize Gemini Session

When starting Gemini CLI:

```
You are IR-ASSIST. Load your system instructions from ~/ir-assist-skill/SYSTEM.md.
Your reference documents are:
- ~/ir-assist-skill/ir-playbook.md (IR procedures)
- ~/ir-assist-skill/ir-doctrine.md (thinking frameworks)
- ~/ir-assist-skill/it-atlas.md (infrastructure reference)

Confirm ready.
```

---

## Quick Command Reference

### Incident Management
| Command | Action |
|---------|--------|
| `/new-incident <desc>` | Create new incident |
| `/resume` | Load active incident |
| `/status` | Show current state |
| `/close-incident` | Close and archive |

### Investigation
| Command | Action |
|---------|--------|
| `/phase` | Show phase progress |
| `/timeline` | Display event timeline |
| `/ioc` | List indicators |
| `/hypothesis` | Track hypotheses |

### Decision Support
| Command | Action |
|---------|--------|
| `/options` | Decision matrix |
| `/challenge` | Socratic questioning |
| `/mtq` | Material/Time/Quality assessment |

### Reporting
| Command | Action |
|---------|--------|
| `/sitrep` | Situation report |
| `/brief` | Executive summary (SCQA) |
| `/export` | Generate outputs |

See `QUICKSTART.md` for complete command reference and workflow examples.

---

## Customization

### Incident Categories

Edit `ir-playbook.md` Section "Incident Categories" to add bank-specific categories:

```markdown
| CAT 7 | ATM Compromise | Physical or logical ATM attack |
| CAT 8 | Wire Fraud | Unauthorized wire transfer attempt |
| CAT 9 | Core Banking Breach | Unauthorized access to core systems |
```

### Roles and Responsibilities

Edit `ir-playbook.md` Section "Roles and Responsibilities" with your team structure.

### Infrastructure Reference

Edit `it-atlas.md` to reflect your actual:
- Network architecture
- Security tools
- Critical assets
- Contact information

### Escalation Thresholds

Edit `ir-playbook.md` Section "Priority Matrix" and escalation rules.

---

## Session Persistence

IR-ASSIST maintains state across sessions using the file system:

```
~/ir-assist/
├── incidents/
│   └── INC-YYYY-NNNN/
│       ├── incident-state.json    # Master state
│       ├── timeline.json          # Event chronology
│       ├── iocs.json              # Indicators
│       ├── hypotheses.json        # Hypothesis tracking
│       ├── tasks.json             # Task assignments
│       ├── sitreps/               # Situation reports
│       └── notes.md               # Freeform notes
├── config/
│   └── active-incident.txt        # Points to current incident
└── exports/                       # Generated reports
```

To resume an incident after closing Gemini:
1. Start new Gemini session
2. Load system prompt
3. Run `/resume`

---

## Thinking Frameworks

IR-ASSIST incorporates 11 decision support frameworks from `ir-doctrine.md`:

1. **Positive vs. Negative Control** — AI handles playbooks, humans handle judgment
2. **Cognitive Limits Are Structural** — Actively compensate for human biases
3. **Process Over Outcomes** — Evaluate decisions, not just results
4. **Consequence × Reversibility** — Classify decisions for appropriate automation
5. **Risk vs. Uncertainty** — Acknowledge when probabilities are unknown
6. **The Metaphor Trap** — Surface competing mental models
7. **Chess Phases** — Opening/Middlegame/Endgame require different approaches
8. **MTQ Assessment** — Material/Time/Quality situational analysis
9. **Answer-First (Minto)** — Lead with conclusions
10. **Socratic Challenge** — Probe assumptions at decision points
11. **Shared Meaning** — Capture who/what/when commitments

---

## Outputs

IR-ASSIST can generate:

| Output | Format | Command |
|--------|--------|---------|
| Situation Report | Markdown | `/sitrep` |
| Executive Brief | Markdown (SCQA) | `/brief` |
| Timeline | JSON/Markdown | `/timeline` |
| IOC List | JSON | `/ioc` |
| Task List | JSON/Markdown | `/tasks` |
| Full Audit Trail | JSON | `/export audit` |
| HTML Dashboard | HTML | `/export dashboard` |
| After-Action Report | Markdown | `/export report` |

---

## Best Practices

1. **Always start with `/new-incident` or `/resume`** — Establishes context
2. **Use commands for consistent output** — `/sitrep` beats "give me a sitrep"
3. **Verify state saves** — After key updates, confirm persistence
4. **Check exit criteria before advancing phases** — Don't skip steps
5. **Generate sitreps regularly** — Start of shift, end of shift, major developments
6. **Use `/challenge` for critical decisions** — Forces consideration of alternatives
7. **Export for handoffs** — `/export audit` + `/sitrep` for analyst rotation

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Gemini drifts off-topic | "Focus. Return to INC-XXXX, phase Y." |
| Gemini gets verbose | "Be concise. Key facts and next action only." |
| Gemini hallucinates | "Stop. What do we actually know from evidence?" |
| State not persisting | "Save current state to incident-state.json" |
| Context lost | "Read incident-state.json and resume from that state" |
| Wrong phase behavior | "Switch to [INVESTIGATE/TRIAGE/etc.] mode" |

See `QUICKSTART.md` for detailed troubleshooting guidance.

---

## Security Considerations

- **No real incident data in examples** — All examples use fictitious data
- **Customize before production use** — Replace placeholders with real contacts/procedures
- **Protect incident files** — `~/ir-assist/incidents/` contains sensitive investigation data
- **Review exports before sharing** — Ensure appropriate classification

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | February 2026 | Initial release |

---

## License

Internal use. Customize for your organization.

---

*IR-ASSIST — Ready to respond.*
