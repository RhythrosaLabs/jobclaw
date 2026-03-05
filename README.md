# JobClaw — Autonomous AI Job Hunting Agent

**Find it. Research it. Apply. On autopilot.**

**JobClaw** is an autonomous AI job hunting agent built on top of [OpenClaw](https://openclaw.ai). It handles the entire job search lifecycle end-to-end: building your candidate profile, scraping 15+ job boards, researching companies, generating tailored resumes and cover letters, and automatically filling and submitting applications — all through a conversational interface or fully on autopilot.

> **Not just a job board aggregator.** JobClaw is an active agent that works on your behalf — searching, matching, writing, and applying while you focus on the interviews.

---

## What it does

### 1. Profile Building

Interactive interview that extracts your experience, skills, preferences, and career goals into a structured candidate profile. Or paste your existing resume and it extracts everything automatically.

### 2. Multi-Source Job Search

Scrapes **15+ job boards and career pages** simultaneously, including:

- LinkedIn, Indeed, Glassdoor
- Greenhouse, Lever, Workday
- Y Combinator (Work at a Startup), Built In, AngelList
- Direct company career pages for your target companies
- Configurable refresh interval (default: every 60 minutes)

### 3. Intelligent Job Matching

Every listing is scored against your profile using both heuristic rules and LLM-powered semantic analysis — so you only see roles that actually fit.

### 4. Deep Company Research

Before you apply, JobClaw investigates:

- Company culture, size, funding stage, and tech stack
- Glassdoor reviews and hiring velocity
- Recent news, leadership changes, and competitive position
- Key contacts and hiring managers

### 5. Document Generation

Generates **ATS-optimized, company-tailored** resumes and cover letters for each application. Tone-matched to the company's voice. Formatted as PDF or DOCX.

### 6. Automated Application Submission

Detects application form types (Greenhouse, Lever, Workday, custom), fills every field, uploads documents, answers screening questions, and submits — with or without your confirmation.

### 7. Tracking & Follow-Up

Tracks every application's status and drafts strategic follow-up emails at the right time.

---

## Quick start

**Requires Node ≥ 22.**

```bash
npm install -g openclaw@latest

openclaw onboard --install-daemon
```

Then enable the job-hunter plugin in `~/.openclaw/openclaw.json`:

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-6",
  },
  plugins: {
    "job-hunter": {
      autoApply: false,
      targetRolesPerDay: 20,
      searchSources: ["linkedin", "indeed", "greenhouse", "lever", "ycombinator"],
      locationPreferences: {
        remote: true,
        hybrid: true,
        onsite: false,
      },
      salaryRange: {
        min: 120000,
        currency: "USD",
      },
    },
  },
}
```

Then just talk to it:

```
"Start my job search for senior backend engineer roles"
"Build my profile from my resume at ~/resume.pdf"
"Apply to that Stripe role we found yesterday"
"Show me my application status"
"Run the full autonomous pipeline"
```

---

## Autonomous pipeline

Run the full pipeline hands-free:

```bash
openclaw agent --message "Run the job hunter pipeline for senior engineer roles at Series B+ startups, remote only, $150k+"
```

JobClaw will:

1. Build/load your profile
2. Search all configured sources
3. Score and filter matches
4. Research the top companies
5. Generate tailored documents
6. Submit applications (if `autoApply: true`)
7. Report back with a summary

---

## Available tools

| Tool                               | Description                              |
| ---------------------------------- | ---------------------------------------- |
| `job_hunter_build_profile`         | Interactive profile building interview   |
| `job_hunter_profile_respond`       | Continue a profile interview session     |
| `job_hunter_search`                | Search across all configured job boards  |
| `job_hunter_search_company`        | Search a specific company's openings     |
| `job_hunter_research_company`      | Deep company research and briefing       |
| `job_hunter_company_briefing`      | Strategic pre-application briefing       |
| `job_hunter_generate_resume`       | ATS-optimized tailored resume (PDF/DOCX) |
| `job_hunter_generate_cover_letter` | Company-tailored cover letter            |
| `job_hunter_apply`                 | Full application pipeline for one job    |
| `job_hunter_list_applications`     | List all tracked applications            |
| `job_hunter_run_pipeline`          | Run the full autonomous pipeline         |
| `job_hunter_status`                | Pipeline status and metrics              |

---

## Configuration reference

```json5
{
  plugins: {
    "job-hunter": {
      // Storage paths (default: ~/.openclaw/job-hunter/)
      profilePath: "~/.openclaw/job-hunter/profile",
      documentsPath: "~/.openclaw/job-hunter/documents",
      applicationsPath: "~/.openclaw/job-hunter/applications",

      // Search behavior
      searchSources: [
        "linkedin",
        "indeed",
        "glassdoor",
        "greenhouse",
        "lever",
        "workday",
        "builtin",
        "angellist",
        "ycombinator",
      ],
      maxConcurrentSearches: 5,
      searchRefreshIntervalMinutes: 60,
      targetCompanies: ["Stripe", "Notion", "Linear"],
      excludeCompanies: ["CompanyToSkip"],
      industryKeywords: ["fintech", "developer tools"],

      // Location
      locationPreferences: {
        remote: true,
        hybrid: true,
        onsite: false,
        cities: ["San Francisco", "New York"],
        countries: ["US"],
      },

      // Salary
      salaryRange: { min: 120000, max: 250000, currency: "USD" },

      // Automation
      autoApply: false, // set true to submit without confirmation
      headlessBrowser: false, // true = run browser silently
      targetRolesPerDay: 20,

      // Models
      defaultProvider: "anthropic",
      defaultModel: "claude-opus-4-6",
    },
  },
}
```

---

## How it works

```
You (chat / CLI / WhatsApp / Telegram / etc.)
         │
         ▼
┌─────────────────────────────────┐
│         OpenClaw Gateway         │
│      (WebSocket control plane)   │
└──────────────┬──────────────────┘
               │
         ┌─────▼──────┐
         │  Job Hunter │
         │   Plugin    │
         └─────┬───────┘
               │
    ┌──────────┼──────────────┐
    │          │              │
    ▼          ▼              ▼
Job Boards  Company       Document
Scraper     Researcher    Generator
(15+ sites) (LLM+search)  (PDF/DOCX)
    │                         │
    └──────────┬──────────────┘
               ▼
        Application
         Submitter
      (Browser automation)
```

JobClaw runs as an OpenClaw plugin. OpenClaw's gateway manages sessions, scheduling (cron), browser automation, and multi-channel delivery — so JobClaw can message you on WhatsApp, Telegram, Slack, Discord, or any other connected channel with updates and questions.

---

## Requirements

- **Node ≥ 22**
- **OpenClaw** installed and running (`openclaw onboard`)
- **Browser automation** enabled (for application submission) — OpenClaw manages a dedicated Chrome/Chromium instance
- An LLM API key (OpenAI, Anthropic, or any supported provider)

---

## Development

```bash
git clone https://github.com/RhythrosaLabs/jobclaw.git
cd jobclaw
pnpm install
pnpm build

# Run the gateway with job-hunter loaded
pnpm gateway:dev
```

The job-hunter extension lives in `extensions/job-hunter/`. The skill prompt is in `skills/job-hunter/SKILL.md`.

---

## Built on OpenClaw

JobClaw is the job hunting extension for [OpenClaw](https://openclaw.ai) — an open-source personal AI assistant platform. OpenClaw provides the WebSocket gateway, browser automation, multi-channel messaging, cron scheduling, and plugin SDK that JobClaw builds on.

[OpenClaw docs](https://docs.openclaw.ai) · [OpenClaw repo](https://github.com/openclaw/openclaw) · [Discord](https://discord.gg/clawd)

---

## License

MIT — see [LICENSE](LICENSE).
