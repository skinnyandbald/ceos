# CompanyOS Integration

How to use CEOS alongside [CompanyOS](https://github.com/bradfeld/companyos) for teams that want both EOS-specific tools and broader company operations support.

## What is CompanyOS?

CompanyOS is a Claude Code skills package for company operations — the day-to-day workflows of running a company. It handles things like communications, scheduling, feedback triage, content creation, and operational processes. CompanyOS is general-purpose and adapts to how your company works.

## What is CEOS?

CEOS (Claude + EOS) is a Claude Code skills package specifically for the [Entrepreneurial Operating System](https://www.eosworldwide.com/). It provides 5 skills that implement EOS tools: V/TO, Rocks, Scorecard, L10 Meetings, and IDS.

## Complementary Boundaries

CEOS and CompanyOS solve different problems. They overlap in spirit (both help you run a business) but not in implementation.

| Scope | CEOS | CompanyOS |
|-------|------|-----------|
| **Strategic planning** | V/TO, 10-Year Target, 3-Year Picture, 1-Year Plan | Company policies, decision logs |
| **Quarterly priorities** | Rocks (EOS-structured, binary scoring) | Project tracking (flexible, ongoing) |
| **Weekly meetings** | L10 (strict 7-section, 90-minute format) | General meeting facilitation |
| **Metrics** | Scorecard (5-15 weekly measurables, trend analysis) | Custom dashboards, reporting |
| **Problem solving** | IDS (Identify, Discuss, Solve with 5 Whys) | General issue tracking |
| **Communications** | Not covered | Email, announcements, updates |
| **Scheduling** | Not covered | Calendar, meeting prep |
| **Content** | Not covered | Blog posts, marketing, social |
| **Operations** | Not covered | Contacts, feedback, processes |

**The key distinction:** CEOS is opinionated (it follows the EOS framework exactly). CompanyOS is flexible (it adapts to your workflows).

## When to Use Which

| Situation | Use |
|-----------|-----|
| Setting quarterly priorities with measurable outcomes | CEOS (`ceos-rocks`) |
| Tracking a project that doesn't fit the Rock format | CompanyOS |
| Running a weekly leadership meeting with the EOS agenda | CEOS (`ceos-l10`) |
| Preparing for a board meeting or investor update | CompanyOS |
| Logging weekly business metrics | CEOS (`ceos-scorecard`) |
| Sending a company-wide announcement | CompanyOS |
| Solving a team issue with structured root-cause analysis | CEOS (`ceos-ids`) |
| Triaging customer feedback | CompanyOS |
| Reviewing the company's strategic direction | CEOS (`ceos-vto`) |
| Managing day-to-day operational tasks | CompanyOS |

## Data Flow

CEOS and CompanyOS maintain separate data stores. There is no automatic synchronization between them.

```
~/Code/ceos/data/           ← CEOS data (EOS-specific)
├── vision.md
├── rocks/
├── scorecard/
├── issues/
└── meetings/

~/Code/companyos/           ← CompanyOS data (company operations)
├── contacts/
├── decisions/
├── policies/
└── ...
```

This separation is intentional:
- **CEOS data follows the EOS format** — strict structure, specific enums, quarterly cadence
- **CompanyOS data follows your company's needs** — flexible structure, custom workflows
- **No coupling means no breakage** — updating one doesn't affect the other

## Skill Discovery

Both packages install skills via symlinks into `~/.claude/skills/`:

```
~/.claude/skills/
├── ceos-vto/        ← symlink to ~/Code/ceos/skills/ceos-vto/
├── ceos-rocks/      ← symlink to ~/Code/ceos/skills/ceos-rocks/
├── ceos-scorecard/  ← symlink to ~/Code/ceos/skills/ceos-scorecard/
├── ceos-l10/        ← symlink to ~/Code/ceos/skills/ceos-l10/
├── ceos-ids/        ← symlink to ~/Code/ceos/skills/ceos-ids/
├── co-ops/          ← symlink to ~/Code/companyos/skills/co-ops/
├── co-comms/        ← symlink to ~/Code/companyos/skills/co-comms/
└── ...              ← other CompanyOS skills
```

Claude Code discovers both sets of skills automatically. The `ceos-` and `co-` naming prefixes prevent collisions.

**Installation:**

```bash
# Install CEOS skills
cd ~/Code/ceos && ./setup.sh

# Install CompanyOS skills (per CompanyOS setup docs)
cd ~/Code/companyos && ./setup.sh
```

Both can be installed and uninstalled independently.

## Example Workflows

### L10 Meeting Creates To-Dos, CompanyOS Tracks Follow-Through

```
1. Run L10 meeting (CEOS: ceos-l10)
   → IDS session creates To-Dos with owners and due dates

2. To-Dos are recorded in the L10 meeting notes
   → data/meetings/l10/2026-02-13.md

3. Next week's L10 reviews To-Do completion
   → 90%+ completion rate target

4. For To-Dos that need broader coordination:
   → Use CompanyOS to send updates, schedule follow-ups
```

### Strategic Rock Drives Operational Projects

```
1. Set a Rock: "Launch Partner Program" (CEOS: ceos-rocks)
   → data/rocks/2026-Q1/rock-003-launch-partner-program.md

2. Rock owner uses CompanyOS for execution:
   → Draft partner outreach emails (co-comms)
   → Schedule partner meetings (calendar skills)
   → Track partner feedback (feedback skills)

3. Weekly L10 checks Rock status (CEOS: ceos-l10)
   → On track / Off track — reported in 5 minutes
```

### Company Policy Change Triggers V/TO Update

```
1. CompanyOS tracks a policy decision
   → "Expanding to European market"

2. This affects strategic direction:
   → Update 3-Year Picture in V/TO (CEOS: ceos-vto)
   → Check Rock alignment with new strategy

3. If misalignment found:
   → Create Issue for next L10 (CEOS: ceos-ids)
   → "Rocks don't reflect Europe expansion"
```

## FAQ

**Can I use CEOS without CompanyOS?**
Yes. CEOS is fully standalone. It only needs Claude Code and git.

**Can I use CompanyOS without CEOS?**
Yes. CompanyOS works independently of any EOS framework.

**What if I want to sync data between them?**
Build a custom skill or script that reads from one and writes to the other. The markdown + YAML format makes parsing straightforward. See [data-format-spec.md](data-format-spec.md) for the CEOS data format.

**Do they share any code?**
No. They are separate repositories with separate skills. The only shared dependency is Claude Code itself.
