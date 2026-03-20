# Data Flow

This document describes how data moves through the Project Planner application — from user actions in the browser to external integrations like Git providers and email services.

---

## Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                            CLIENTS                                   │
│                                                                      │
│   Browser (React UI)          Git Provider          CI/CD Pipeline   │
│        │                      (Webhook)              (Webhook)       │
└────────┼──────────────────────────┼──────────────────────┼──────────┘
         │                          │                      │
         ▼                          ▼                      ▼
┌──────────────────────────────────────────────────────────────────────┐
│                      NEXT.JS APP (App Router)                        │
│                                                                      │
│   Server Components / Client Components                              │
│   API Routes  (/app/api/*)                                           │
│   Server Actions                                                     │
└──────────────────────────────────┬───────────────────────────────────┘
                                   │
         ┌─────────────────────────┼──────────────────────┐
         ▼                         ▼                      ▼
┌─────────────────┐    ┌─────────────────────┐   ┌──────────────────┐
│    Database     │    │   Email Provider    │   │  Git Provider    │
│  (Projects,     │    │  (Resend/SendGrid/  │   │  API (GitHub,    │
│  Tasks, Users,  │    │      SMTP)          │   │  GitLab, etc.)   │
│  Milestones)    │    └─────────────────────┘   └──────────────────┘
└─────────────────┘
```

---

## 1. User Interaction Flow

The primary flow for a user creating or updating project data.

```
User Action (e.g. create task)
        │
        ▼
React Client Component
  └── form submit / button click
        │
        ▼
Next.js Server Action  OR  API Route (/app/api/tasks)
  └── validate input
  └── apply business logic (e.g. status change, assignee resolution)
        │
        ├──► Write to Database
        │         └── task record created/updated
        │
        └──► Trigger Notification Check
                  └── (see Notification Flow below)
```

---

## 2. Git Integration Flow

Connects external Git provider events to internal task state.

```
Git Provider (GitHub / GitLab / Bitbucket)
  └── Event: push, PR opened, PR merged, PR review requested
        │
        ▼
Webhook  →  POST /app/api/webhooks/git
  └── verify webhook signature
  └── parse event payload
        │
        ├── Extract task ID from branch name or commit message
        │     e.g.  feat/task-42-onboarding  or  "fix #42 ..."
        │
        ├──► Update Task in Database
        │         └── status, linked PR URL, commit SHA
        │
        └──► Trigger Notification Check
                  └── (see Notification Flow below)
```

---

## 3. CI/CD Pipeline Flow

Surfaces deployment status inside the planner.

```
CI/CD Pipeline (GitHub Actions / GitLab CI / etc.)
  └── Job completes: success or failure
        │
        ▼
Webhook  →  POST /app/api/webhooks/ci
  └── verify payload
  └── map pipeline run to project task via branch name
        │
        ├──► Update Task deployment status in Database
        │         └── "In Staging" | "Live" | "Failed"
        │
        └──► Send Deployment Status Email to Project Owner
```

---

## 4. Notification Flow

Centralised logic that decides when and to whom emails are sent.

```
Trigger Sources:
  - User Action (task assigned, review requested)
  - Git Webhook (PR status change)
  - CI/CD Webhook (deployment result)
  - Scheduled Job (milestone reminders, inactivity checks)
        │
        ▼
Notification Service  (/app/api/notifications  or  server action)
  └── look up recipients and their preferences from Database
  └── check if user has opted out / uses digest mode
        │
        ├── [Per-Event Mode]
        │     └──► Email Provider API (Resend / SendGrid / SMTP)
        │               └── sends email immediately
        │
        └── [Digest Mode]
              └──► Queue notification in Database
                        └── Scheduled Job (daily at user's preferred time)
                                └──► Email Provider API
                                          └── sends digest email
```

---

## 5. Scheduled Jobs

Background tasks that run on a timer, not triggered by user actions.

| Job | Trigger | Action |
|---|---|---|
| Inactivity Check | Every 6 hours | Finds projects with no activity > 3 days, queues "Pending Project" email |
| Milestone Reminder | Daily | Finds milestones due in 3 days or 1 day, sends reminders |
| Overdue Milestone | Daily | Finds milestones past due date, notifies owner and managers |
| Digest Sender | Per-user timezone (default 8:00 AM) | Bundles queued notifications into a single digest email |

---

## 6. Data Read Flow (Page Render)

How data reaches the UI for read operations.

```
User navigates to a page (e.g. /projects/42)
        │
        ▼
Next.js Server Component
  └── fetch project data from Database (server-side, no API round-trip)
  └── fetch linked Git PR status from Git Provider API (if stale)
        │
        ▼
Render HTML on the server → stream to browser
        │
        ▼
Client Components hydrate
  └── subscribe to real-time updates (polling or WebSocket)
        │
        ▼
Subsequent updates pushed to client as tasks/PRs change
```

---

## 7. Role-Based Data Visibility

Data returned from the API is filtered based on the requesting user's role.

```
API Request  +  Auth Token (session/JWT)
        │
        ▼
Auth Middleware
  └── verify identity
  └── resolve role: Owner | Manager | Developer | Stakeholder
        │
        ▼
Data Layer
  ├── [Developer / Owner / Manager]
  │     └── Full task detail, Git links, CI status, technical comments
  │
  └── [Stakeholder]
        └── Milestone progress, high-level status, business-facing reports
            (Git commit detail and code links are excluded)
```

---

## Key Data Entities

| Entity | Description | Relationships |
|---|---|---|
| **User** | Team member with a role | belongs to many Projects |
| **Project** | Top-level container | has many Milestones, Tasks, Members |
| **Milestone** | Time-boxed goal | belongs to Project, has many Tasks |
| **Task** | Unit of work | belongs to Milestone, linked to Git branch/PR |
| **Notification** | Pending or sent alert | belongs to User, references Task or Milestone |
| **GitLink** | Branch / PR / commit association | belongs to Task |
| **DeploymentStatus** | CI/CD result | belongs to Task |
