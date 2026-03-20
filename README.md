# Project Planner

**Project Planner** is a modern, collaborative platform for project management and tracking. It bridges the gap between technical and non-technical stakeholders, providing a unified source of truth for project progress — from Git commits to business milestones.

## Key Features

- **Unified Progress Tracking** — High-level overview or granular task details, in one place.
- **Git Workflow Integration** — Sync project milestones and tasks with branches, commits, and pull requests.
- **Multi-Role Experience**
  - **Technical Teams**: Integrated with developer workflows and CI/CD status.
  - **Non-Technical Stakeholders**: Visual milestones and progress reports without needing to understand code.
- **Collaborative Planning** — Real-time updates, shared roadmaps, and threaded task discussions.
- **Automated Notifications** — Email alerts for reviews, milestones, deployments, and task assignments.

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 16 (App Router) |
| Language | TypeScript |
| Styling | Tailwind CSS v4 |
| Package Manager | pnpm |

## Project Structure

```
project-planner/
├── app/                  # Next.js App Router (pages, layouts, API routes)
│   ├── layout.tsx        # Root layout
│   └── page.tsx          # Home page
├── docs/                 # Project documentation
│   ├── features.md       # Feature specifications
│   └── data-flow.md      # Data flow architecture
└── public/               # Static assets
```

## Getting Started

### Prerequisites

- Node.js 18.x or later
- pnpm 9.x or later

### Installation

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd project-planner
   ```

2. **Install dependencies**:
   ```bash
   pnpm install
   ```

3. **Configure environment variables** (copy and fill in):
   ```bash
   cp .env.example .env.local
   ```

4. **Run the development server**:
   ```bash
   pnpm dev
   ```

5. Open [http://localhost:3000](http://localhost:3000) in your browser.

## Documentation

- [Features](docs/features.md) — Detailed breakdown of all planned features.
- [Data Flow](docs/data-flow.md) — How data moves through the application.

## Email Integrations

Project Planner includes a built-in notification system that sends automated emails to keep your team informed about what matters most — without needing to check the app constantly.

### Notification Types

| Trigger | Recipients | Description |
|---|---|---|
| **Pending Project** | Project owner, assigned members | Sent when a project has had no activity for a configurable period (default: 3 days) |
| **Review Requested** | Reviewer(s) | Triggered when a PR or task is marked as "Needs Review" |
| **Review Completed** | Task author | Sent when a reviewer approves or requests changes |
| **Milestone Approaching** | All project members | Reminder sent 3 days and 1 day before a milestone deadline |
| **Milestone Overdue** | Project owner, managers | Sent the day a milestone passes without being marked complete |
| **Task Assigned** | Assignee | Triggered when a task is assigned or reassigned to a team member |
| **Deployment Status** | Project owner | Sent on CI/CD pipeline success or failure tied to a project task |

### Configuration

Email notifications can be configured per-user in **Settings → Notifications**, or at the project level by a project owner. Supported providers:

- **SMTP** — bring your own mail server
- **Resend** — recommended for production (`RESEND_API_KEY`)
- **SendGrid** — via `SENDGRID_API_KEY`

Set the following environment variables to enable email:

```env
EMAIL_PROVIDER=resend          # resend | sendgrid | smtp
RESEND_API_KEY=re_xxxxxxxxxxxx
EMAIL_FROM=noreply@yourdomain.com
```

### Email Design

All emails follow a consistent, minimal layout:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│   [ Project Planner Logo ]                          │
│                                                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│   Hi Sarah,                                         │
│                                                     │
│   ⚠️  You have a pending review                     │
│                                                     │
│   Project:   Mobile App Redesign                    │
│   Task:      Update onboarding flow (#42)           │
│   Requested: 2 days ago by James K.                 │
│   Status:    Awaiting your review                   │
│                                                     │
│          [ View Task & Review ]                     │
│                                                     │
├─────────────────────────────────────────────────────┤
│   You're receiving this because you are a           │
│   reviewer on this project.                         │
│   Manage preferences · Unsubscribe                  │
└─────────────────────────────────────────────────────┘
```

**Design principles:**
- Single, clear call-to-action button per email
- Context block (project, task, who triggered it, timestamp) above the CTA
- Muted footer with unsubscribe and notification preferences link
- Plain-text fallback included for all HTML emails
- Consistent branding: white background, dark heading, accent color for the CTA button

### Digest Mode

Instead of per-event emails, users can opt into a **Daily Digest** — a single email summarising all activity from the past 24 hours across their projects. Sent at a configurable time (default: 8:00 AM in the user's timezone).

---

## Contributing

Contributions are welcome. Please read the [Contributing Guidelines](CONTRIBUTING.md) before submitting pull requests.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
