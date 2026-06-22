---
id: incoming-intel
name: Incoming Intelligence
description: "Process a folder of raw materials from external sources (forums, TG channels, courses, communities). Scans files, extracts knowledge, checks against existing experience, evaluates usefulness, runs adversarial critic, presents structured report. After approval - integrates into your skill system."
runtime: claude-only
group: meta-orchestration
triggers:
  - "process folder"
  - "analyze folder"
  - "incoming"
  - "intel"
  - "new materials"
---

# Incoming Intelligence - Process External Materials into Actionable Skills

7-step pipeline that turns a chaotic folder of raw materials from external sources into structured, integrated skills, approaches, and knowledge.

## The Problem

You attend webinars, follow Telegram channels, join forums, take courses. You collect PDFs, skill files, code snippets, guides - and dump them into a folder. Then what?

- You forget what's in there
- You don't know what's genuinely useful vs. noise
- You already have tools that cover half of it - but you don't check
- Good materials rot in a folder instead of becoming part of your workflow

This skill solves all four problems in one pass.

## How It Works

```
FOLDER WITH RAW MATERIALS
    |
    v
[1. SCAN]        Build manifest: file, type, size, language
    |
    v
[2. EXTRACT]     Parallel readers (haiku) for each file type
    |             MD, PDF, images, code, Office docs
    v
[3. CONTEXT]     Check YOUR existing skills/knowledge for overlap
    |             graphify, skill index, router registry, memory
    v
[4. EVALUATE]    Rate each artifact: useful? better than ours? safe?
    |             Categorize: NEW_SKILL | ENHANCE | APPROACH | REFERENCE | DISCARD
    v
[5. CRITIC]      Adversarial review - independent agent challenges recommendations
    |             Can flip TAKE to SKIP and vice versa
    v
[6. REPORT]      Structured report for human decision
    |             STOP - wait for approval
    v
[7. INTEGRATE]   Create skills, update existing, distribute files
```

## Requirements

- Claude Code with Agent tool (for parallel sub-agents)
- Python 3 with pdfminer (`pip install pdfminer.six`) for PDF extraction
- graphify (optional, for knowledge graph queries)
- skill-register.py (optional, for automated skill registration)

## Installation

```bash
# Copy SKILL.md to your skills directory
mkdir -p ~/.claude/skills/incoming-intel
cp SKILL.md ~/.claude/skills/incoming-intel/

# Register (if you have skill-register.py)
python ~/.claude/orchestration/skill-register.py ~/.claude/skills/incoming-intel
```

Or just feed the file to Claude Code:
```bash
cat SKILL.md | claude
```

## Usage

Point Claude Code at any folder:

```
"Process the folder ~/Downloads/webinar-materials"
"What's useful in this folder?"
"Analyze ~/Desktop/AI-course-files"
```

---

## Pipeline Details

### Step 1: SCAN - Build Manifest

Recursively scan the target folder. Build a manifest:

```
MANIFEST: [folder path]
Total files: N
-------------------------------------
ID | File               | Type      | Size | Lang
01 | README.md          | markdown  | 4KB  | en
02 | guide.pdf          | pdf       | 350KB| ru
03 | SKILL.md           | skill     | 22KB | en
04 | screenshot.png      | image    | 1.2MB| -
```

**File types and handlers:**

| Type | Extensions | Handler | Model |
|---|---|---|---|
| Markdown/Text | .md, .txt, .rtf | Direct Read | haiku |
| PDF | .pdf | pdfminer extract_text | haiku |
| Image | .png, .jpg, .jpeg, .webp | Claude Vision (Read) | haiku |
| Code/Skill | .py, .js, .ts, .sh, SKILL.md | Direct Read | haiku |
| Office | .docx, .xlsx, .pptx | python-docx/openpyxl | haiku |
| Archive | .zip, .tar.gz | Unpack first, then recurse | - |
| Config | .json, .yaml, .toml, .env | Direct Read | haiku |
| Other | * | Skip with note | - |

### Step 2: EXTRACT - Parallel Reading

Launch parallel forks (model: haiku) to read files grouped by type.

**Each extractor returns:**
```
FILE: [filename]
TYPE: [markdown/pdf/image/code/office]
LANGUAGE: [ru/en]
SUMMARY: [2-3 sentences]
KEY_FINDINGS:
  - [finding 1]
  - [finding 2]
CATEGORY: [skill | tool | approach | reference | tutorial | config | unknown]
TOPICS: [comma-separated]
QUALITY: [1-5]
ACTIONABLE: [yes/no]
```

**Extraction rules:**
- Read files FULLY - not just the first 100 lines
- PDF via pdfminer with UTF-8 encoding fix
- Images via Claude Vision
- SKILL files - pay attention to frontmatter, triggers, pipeline
- Code - evaluate quality, safety, dependencies
- .env files - NEVER output key values

### Step 3: CONTEXT CHECK - Check Existing Experience

The key step. Check ALL your existing work for overlaps:

```python
# Pseudo-code
for topic in extracted_topics:
    # 1. Knowledge graph (if available)
    graphify_query(topic)
    
    # 2. Existing skills
    grep(topic, "~/.claude/skills/INDEX.md")
    
    # 3. Router/orchestration
    grep(topic, "~/.claude/orchestration/ROUTER_REGISTRY.md")
    
    # 4. Project memory
    grep(topic, "~/.claude/projects/*/memory/MEMORY.md")
    
    # 5. Knowledge base (if available)
    grep(topic, "<your-knowledge-base>/**/*.md")
```

**Result - overlap map:**
```
TOPIC: video montage
  EXISTING SKILL: video-montage (created 2024-01-15)
  OVERLAP: 90% - full pipeline already exists
  VERDICT: skip or minor enhance

TOPIC: prompt engineering
  EXISTING SKILL: senior-prompt-engineer
  OVERLAP: 70% - basics covered, but step-back prompting missing
  VERDICT: possible enhance
```

### Step 4: EVALUATE - Rate Each Artifact

For each artifact (not file - semantic unit):

| Criterion | Question | Scale |
|---|---|---|
| Usefulness | Does this solve a real problem in our work? | 1-5 |
| Quality | Is it better than our current solution? | 1-5 |
| Safety | No hardcoded secrets, suspicious code, dangerous deps? | safe/review/danger |
| Freshness | Are the APIs, libraries, approaches current? | current/outdated |
| Compatibility | Works on our OS? Compatible with our stack? | yes/partial/no |

**Categories:**

| Category | When | Action |
|---|---|---|
| `NEW_SKILL` | No equivalent exists, usefulness >= 4 | Create skill in ~/.claude/skills/ |
| `ENHANCE` | Similar skill exists but incoming adds value | Update existing SKILL.md |
| `APPROACH` | Not a skill, but a workflow/process change | Update CLAUDE.md or memory |
| `VAULT_FILE` | Useful info for a specific project or knowledge area | Distribute to project folder |
| `REFERENCE` | Useful as reference, not tied to a project | Save in knowledge base |
| `DISCARD` | Not applicable, outdated, duplicate, or unsafe | Don't take, explain why |

### Step 5: CRITIC - Adversarial Review

**MANDATORY. Do not skip.**

Launch a separate fork (model: sonnet) that does NOT see the evaluator's reasoning. Critic receives:
- Original files from the folder
- Evaluator's recommendations (category + brief justification)
- List of existing skills

The critic checks each recommendation:

```
CRITIC REVIEW:
---------------------
[1] video-montage -> NEW_SKILL
    Evaluator says: full pipeline, we don't have it
    Critic says: AGREE - no equivalent exists, ffmpeg commands verified, safe.
    But: font paths need OS adaptation.
    Verdict: TAKE with adaptation

[2] prompt-engineering.pdf -> REFERENCE
    Evaluator says: basic level, we have it
    Critic says: DISAGREE - step-back prompting worth adding to our skill.
    Verdict: ENHANCE existing prompt skill

[3] remote-control.md -> DISCARD
    Evaluator says: macOS only
    Critic says: AGREE - fully macOS, useless for us.
    Verdict: DISCARD
```

**Critic rules:**
- Can flip TAKE to SKIP and vice versa
- Checks: not a duplicate? safe? worth the complexity?
- Has the right to say "evaluator overestimates / underestimates"
- On disagreement - both positions go to report for human decision

### Step 6: REPORT - Present to Human

```
========================================
INCOMING INTEL: [folder name]
Source: [where materials came from]
Files: N | Artifacts: M
Useful: X | Skip: Y
========================================

TAKE (NEW_SKILL):
-----------------
1. [name]
   What: brief description
   Why: how it helps our work
   Critic: [agree/disagree + why]
   Steps: create skill -> register -> update router

ENHANCE:
--------
2. [name]
   What: what we're adding
   Where: which skill/approach to enhance
   Critic: [agree/disagree + why]

APPROACH CHANGE:
----------------
3. [name]
   What: which workflow changes
   Before: current state
   After: new state
   Critic: [agree/disagree + why]

DISTRIBUTE (VAULT_FILE):
------------------------
4. [file] -> projects/AI-Exoskelet/
   Reason: relates to channel automation
   Project status: active development

SKIP (DISCARD):
---------------
5. [name] - [reason in 1 line]

========================================
AWAITING DECISION
Approve all? / Selective? / Reject?
========================================
```

**After report - STOP.** Wait for human decision. Do not integrate automatically.

### Step 7: INTEGRATE - After Approval

Only after explicit approval:

1. **NEW_SKILL:** Create folder in `~/.claude/skills/[name]/`, write SKILL.md, copy scripts, register
2. **ENHANCE:** Edit existing SKILL.md, add new sections
3. **APPROACH:** Update CLAUDE.md or memory
4. **VAULT_FILE:** Move file to target folder in knowledge base, update navigation
5. **REFERENCE:** Save in knowledge base under appropriate section
6. Update router registry (if new skills)
7. Update knowledge graph (if available)
8. Save memory about decisions made
9. Clean up source folder - processed files to archive or delete

**Integration checklist:**
- [ ] Skills registered (INDEX + REGISTRY)
- [ ] Router updated
- [ ] Memory saved
- [ ] Source files: decision made (move / delete)
- [ ] Knowledge graph updated (if applicable)

---

## Vault Distribution Mode (Optional)

If you maintain a structured knowledge base (Obsidian vault, docs folder, etc.), the skill can also distribute non-skill files to appropriate project folders.

Activate by pointing at your inbox/incoming folder. The skill adds Step 3.5 (PROJECT MAPPING) that:
1. Reads your project registry to know what's active
2. Checks each project's status to determine relevance
3. Maps files to project folders, knowledge sections, or skill directory
4. Reports distribution plan alongside skill recommendations

Configure by setting your vault structure in the SKILL.md or providing it at runtime.

---

## Token Optimization

| Stage | Model | Why |
|---|---|---|
| Extract (file reading) | haiku subagents | Bulk reading, cheap |
| Context Check | sonnet fork | Needs access to your files |
| Evaluate | sonnet fork | Decision making |
| Critic | sonnet fork (adversarial) | Independent review |
| Report + Integrate | main session (sonnet) | Final assembly |

---

## Rules

1. **Read files FULLY** - not just first 100 lines
2. **PDF via pdfminer** - with UTF-8 encoding fix for non-Latin scripts
3. **Images via Vision** - describe contents
4. **.env files** - NEVER output key values
5. **Code** - check for hardcoded secrets before including
6. **Critic is mandatory** - don't skip even for "obvious" decisions
7. **Human in the loop** - human approves BEFORE integration
8. **All skills in one place** - `~/.claude/skills/`, don't scatter
9. **Adapt to OS** - font paths, shell commands, line endings
10. **Don't duplicate** - if a skill on this topic exists, enhance it, don't create a new one
