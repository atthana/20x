# 20x : คู่มือทำความเข้าใจโปรเจคฉบับสมบูรณ์ (ภาษาไทย)

> เอกสารนี้เขียนขึ้นสำหรับคนที่เพิ่ง clone โปรเจคมาครั้งแรก ยังไม่รู้อะไรเลย
> โดยอธิบายเปรียบเทียบกับโลกของ Backend (Django) เพื่อให้เข้าใจง่ายขึ้น

---

## สารบัญ

1. [20x คืออะไร และมีไว้ทำไม](#1-20x-คืออะไร-และมีไว้ทำไม)
2. [Tech Stack ทั้งหมด](#2-tech-stack-ทั้งหมด)
3. [สถาปัตยกรรม Electron (เทียบกับ Django)](#3-สถาปัตยกรรม-electron-เทียบกับ-django)
4. [โครงสร้างโฟลเดอร์](#4-โครงสร้างโฟลเดอร์)
5. [Database Schema (SQLite)](#5-database-schema-sqlite)
6. [วงจรชีวิตของ Task (Task Lifecycle)](#6-วงจรชีวิตของ-task-task-lifecycle)
7. [ระบบ AI Agent Orchestration (หัวใจของโปรเจค)](#7-ระบบ-ai-agent-orchestration-หัวใจของโปรเจค)
8. [Triage Agent : ระบบคัดแยกงานอัตโนมัติ](#8-triage-agent--ระบบคัดแยกงานอัตโนมัติ)
9. [ระบบ Skills และ Auto-Learning](#9-ระบบ-skills-และ-auto-learning)
10. [Git Worktree Management](#10-git-worktree-management)
11. [Integrations กับเครื่องมือภายนอก](#11-integrations-กับเครื่องมือภายนอก)
12. [IPC Architecture : Frontend คุยกับ Backend ยังไง](#12-ipc-architecture--frontend-คุยกับ-backend-ยังไง)
13. [Heartbeat Monitoring](#13-heartbeat-monitoring)
14. [ส่วนประกอบอื่นๆ ที่น่าสนใจ](#14-ส่วนประกอบอื่นๆ-ที่น่าสนใจ)
15. [วิธีติดตั้งและรันโปรเจค](#15-วิธีติดตั้งและรันโปรเจค)
16. [Use Case แบบ End-to-End](#16-use-case-แบบ-end-to-end)
17. [แนะนำเส้นทางการอ่านโค้ดสำหรับมือใหม่](#17-แนะนำเส้นทางการอ่านโค้ดสำหรับมือใหม่)

---

## 1. 20x คืออะไร และมีไว้ทำไม

**20x** คือ **Desktop Application** (ทำงานบน macOS, Windows, Linux) ที่พัฒนาโดยบริษัท **Peakflo**
สโลแกนของมันคือ:

> "One app. All your tasks. Powered by AI agents."
> (แอปเดียว รวมทุกงานของคุณ ขับเคลื่อนด้วย AI agents)

### ปัญหาที่มันแก้

ปกติเวลาเราใช้ AI ช่วยทำงาน เราต้อง copy-paste ข้อมูลไปมาระหว่างแท็บต่างๆ เช่น
copy ticket จาก Jira ไปวางใน ChatGPT แล้ว copy คำตอบกลับมาวางใน editor ซึ่งเสียเวลามาก

20x กลับด้านความคิดนี้: **"ให้งานวิ่งไปหา AI แทนที่เราจะเอางานไปป้อน AI"**

### หลักการทำงานโดยย่อ

1. **งานไหลเข้ามา** จากระบบที่เราใช้อยู่แล้ว เช่น Linear, YouTrack, HubSpot, GitHub Issues, Notion, GitLab, Peakflo หรือสร้างงานเองในแอป
2. **Triage Agent** (AI คัดแยกงาน) วิเคราะห์งานแต่ละชิ้นแล้วกำหนด priority, เลือก coding agent ที่เหมาะ, เลือก skills และ git repos ที่เกี่ยวข้องให้อัตโนมัติ
3. **AI Agent ลงมือทำงาน** เช่น อ่าน context, เขียนโค้ดใน git worktree แยกของตัวเอง, เปิด PR โดยเราเห็นการทำงานแบบ real-time
4. **HITL (Human-in-the-loop)** ก่อนทำอะไรที่เสี่ยง (เช่นรันคำสั่ง shell อันตราย) agent จะหยุดถามมนุษย์ก่อน
5. **Feedback Loop** หลังงานเสร็จ เราให้คะแนน + คอมเมนต์ แล้วระบบจะเอา feedback ไปปรับปรุง "Skills" (คลังความรู้) ให้ agent ฉลาดขึ้นเรื่อยๆ

### จุดขายสำคัญ

- **Local-first** : ข้อมูลทุกอย่างอยู่ในเครื่องเรา (SQLite) ไม่มี cloud, ไม่มี subscription
- **Multi-agent** : เลือกใช้ AI ได้ 3 ค่าย คือ Claude Code (Anthropic), OpenCode (open-source), Codex (OpenAI)
- **License MIT** : เป็น open source

---

## 2. Tech Stack ทั้งหมด

| ชั้น (Layer) | เทคโนโลยี | เทียบกับโลก Django |
|---|---|---|
| App Shell | **Electron 34** | เหมือนตัว runtime ที่ห่อทั้งแอป (Chromium + Node.js) |
| Build Tool | **electron-vite** + Vite 6 | เหมือน build pipeline (collectstatic + bundler) |
| Frontend | **React 19** + TypeScript | เหมือน Django Templates แต่เป็น SPA เต็มตัว |
| Styling | **Tailwind CSS 4** + cva | utility-first CSS |
| UI Components | **Radix UI** primitives | component library แบบ headless |
| State Management | **Zustand 5** | เหมือน cache/session state ฝั่ง client (มี store ~25 ตัว) |
| Database | **SQLite (better-sqlite3, WAL mode)** | แทน PostgreSQL ใน Django แต่เป็น embedded DB เขียน SQL ตรงๆ ไม่มี ORM |
| Agent SDKs | `@anthropic-ai/claude-agent-sdk`, `@opencode-ai/sdk`, `@zed-industries/codex-acp` | เหมือน 3rd-party API clients |
| MCP | `@modelcontextprotocol/sdk` | โปรโตคอลมาตรฐานให้ AI เรียกใช้ tools |
| Terminal | **node-pty + xterm.js** | จำลอง terminal ในแอป |
| Auth (Enterprise) | **Supabase** (`@supabase/supabase-js`) | ใช้แค่ login enterprise ไม่ได้เก็บข้อมูลงาน |
| Testing | **Vitest + happy-dom** | เหมือน pytest + test client |
| Auto Update | **electron-updater** | เช็ค release ใหม่จาก GitHub Releases |
| Package Manager | **pnpm 9** | ห้ามใช้ npm |
| Video/Image Gen | **Remotion** | ใช้ render ภาพ preview ของ UI (สำหรับ docs/marketing) |
| CI/CD | **GitHub Actions** | build + release อัตโนมัติเมื่อ push tag |

**เวอร์ชันปัจจุบัน:** 0.0.104 (ดูได้จาก `package.json`)

---

## 3. สถาปัตยกรรม Electron (เทียบกับ Django)

Electron app หนึ่งตัวประกอบด้วย **2 process หลัก** ที่แยกกันเด็ดขาด ตรงนี้สำคัญมากสำหรับคนมาจาก backend:

```
┌─────────────────────────────────────────────────────────────┐
│  Renderer Process (React UI)        ← เหมือน "Frontend/SPA"  │
│  - ไม่มีสิทธิ์เข้าถึง Node.js, ไฟล์, database โดยตรง           │
│  - ทำได้แค่แสดงผล + เรียก API ผ่าน bridge                     │
├─────────────────────────────────────────────────────────────┤
│  Preload Bridge (src/preload)       ← เหมือน "API Gateway"   │
│  - ประกาศว่า renderer เรียกอะไรได้บ้าง (whitelist)            │
│  - expose เป็น window.electronAPI                            │
├─────────────────────────────────────────────────────────────┤
│  Main Process (Node.js)             ← เหมือน "Django Backend"│
│  - เข้าถึง SQLite, file system, spawn process ได้             │
│  - รัน agent orchestration, integrations, HTTP servers       │
└─────────────────────────────────────────────────────────────┘
```

### เทียบ Django ตรงๆ

| Django | 20x (Electron) |
|---|---|
| `views.py` + DRF ViewSets | `src/main/ipc-handlers.ts` (รับ request จาก UI) |
| `models.py` + ORM | `src/main/database.ts` (เขียน SQL ตรงๆ ผ่าน better-sqlite3) |
| `migrations/` | ฟังก์ชัน `runMigrations()` ใน `database.ts` + `SCHEMA_VERSION` |
| Celery workers | `agent-manager.ts`, `heartbeat-scheduler.ts`, `recurrence-scheduler.ts` |
| Celery Beat (cron) | `recurrence-scheduler.ts` (ใช้ cron-parser) |
| `urls.py` | ชื่อ IPC channel เช่น `db:getTasks`, `agentSession:start` |
| DRF Serializers | TypeScript interfaces ใน `src/shared/` และ `src/renderer/src/types/` |
| Django REST API | IPC (Inter-Process Communication) ผ่าน preload bridge |
| `settings.py` | ตาราง `settings` ใน SQLite (key-value) |
| Django Apps (3rd party) | Plugins ใน `src/main/plugins/` (linear, hubspot, notion ฯลฯ) |

### Data Flow หลักของแอป

```
React UI → Zustand Store → IPC Client → Preload Bridge → Main Process → SQLite
```

**ความปลอดภัย:** เปิด `contextIsolation: true` และ `nodeIntegration: false`
แปลว่าโค้ด React **ไม่มีทาง** แตะไฟล์หรือ database ได้เอง ต้องผ่าน bridge เท่านั้น
(เหมือน frontend เรียก REST API เท่านั้น แตะ DB ตรงๆ ไม่ได้)

---

## 4. โครงสร้างโฟลเดอร์

```
20x/
├── src/
│   ├── main/                  ← "Backend" ทั้งหมด (Node.js)
│   │   ├── index.ts           ← Entry point: สร้างหน้าต่าง, init ทุก manager
│   │   ├── database.ts        ← Schema + Migration + CRUD ทุกตาราง (ไฟล์ใหญ่สุด ~100KB)
│   │   ├── agent-manager.ts   ← หัวใจ: จัดการ AI agent sessions (~188KB)
│   │   ├── ipc-handlers.ts    ← รวม "endpoint" ทั้งหมดที่ UI เรียกได้ (~81KB)
│   │   ├── heartbeat-scheduler.ts    ← เช็คสุขภาพงานทุก 60 วินาที
│   │   ├── recurrence-scheduler.ts   ← สร้างงานซ้ำตาม cron
│   │   ├── worktree-manager.ts       ← จัดการ git worktree แยกต่องาน
│   │   ├── task-api-server.ts        ← HTTP API ภายใน ให้ agent เรียกกลับมา
│   │   ├── mobile-api-server.ts      ← HTTP + WebSocket สำหรับ mobile client
│   │   ├── secret-broker.ts          ← แจกจ่าย secrets แบบเข้ารหัสให้ agent
│   │   ├── enterprise-auth.ts        ← Login enterprise ผ่าน Supabase
│   │   ├── enterprise-sync.ts        ← Sync skills/MCP กับ Workflo (องค์กร)
│   │   ├── claude-plugin-manager.ts  ← จัดการ Claude Code plugins/marketplace
│   │   ├── auto-updater.ts           ← อัปเดตแอปอัตโนมัติ
│   │   ├── adapters/          ← ตัวเชื่อม AI 3 ค่าย
│   │   │   ├── coding-agent-adapter.ts   ← Interface กลาง
│   │   │   ├── claude-code-adapter.ts    ← Claude Code (Anthropic)
│   │   │   ├── opencode-adapter.ts       ← OpenCode
│   │   │   └── acp-adapter.ts            ← Codex (OpenAI ผ่าน Zed ACP)
│   │   ├── plugins/           ← Integrations (เหมือน Django apps)
│   │   │   ├── registry.ts, linear-plugin.ts, hubspot-plugin.ts,
│   │   │   ├── notion-plugin.ts, youtrack-plugin.ts,
│   │   │   ├── github-issues-plugin.ts, peakflo-plugin.ts
│   │   ├── oauth/             ← OAuth flows ของแต่ละ provider
│   │   ├── mcp-servers/       ← MCP server ในตัว (task-management tools)
│   │   └── agent-installer/   ← ตรวจ + ติดตั้ง CLI tools (git, gh, node ฯลฯ)
│   │
│   ├── preload/index.ts       ← Bridge: expose window.electronAPI ให้ UI
│   │
│   ├── renderer/src/          ← "Frontend" (React 19)
│   │   ├── components/
│   │   │   ├── layout/        ← AppLayout, Sidebar
│   │   │   ├── dashboard/     ← Dashboard + Kanban board
│   │   │   ├── tasks/         ← TaskList, TaskDetailView, TaskForm, FeedbackDialog
│   │   │   ├── agents/        ← Transcript panel, Approval banner, Agent form
│   │   │   ├── canvas/        ← Infinite canvas (panels: task/terminal/browser)
│   │   │   ├── skills/        ← จัดการ Skills
│   │   │   ├── settings/      ← Settings ทุกแท็บ
│   │   │   └── ui/            ← ปุ่ม, Dialog, Input (Radix + Tailwind)
│   │   ├── stores/            ← Zustand stores ~25 ตัว (task, agent, ui, canvas...)
│   │   ├── hooks/             ← use-agent-auto-start (ตัวจัดคิว auto-run) ฯลฯ
│   │   └── lib/ipc-client.ts  ← Wrapper เรียก electronAPI แบบ type-safe
│   │
│   ├── mobile/                ← Web app แยกสำหรับมือถือ (build ด้วย vite.mobile.config.ts)
│   └── shared/constants.ts    ← ค่าคงที่ใช้ร่วม เช่น TaskStatus enum
│
├── docs/                      ← เอกสารเชิงลึก 13 ไฟล์ (อ่านเพิ่มได้)
├── scripts/                   ← build, release, notarize (macOS), rebuild native modules
├── test/                      ← setup ของ Vitest (ไฟล์ test กระจายอยู่ข้างโค้ด)
├── remotion/                  ← สคริปต์ render ภาพ preview UI
├── .github/workflows/         ← CI: verify.yml, release.yml, auto-tag.yml
├── package.json               ← dependencies + config ของ electron-builder
├── AGENTS.md                  ← สเปคสถาปัตยกรรม multi-agent (เอกสารออกแบบ)
├── WINDOWS_PORT_LOG.md        ← บันทึกการ port จาก macOS ไป Windows
└── README.md                  ← ภาพรวมโปรเจค
```

---

## 5. Database Schema (SQLite)

ทุกอย่างอยู่ในไฟล์ SQLite ไฟล์เดียวในเครื่องผู้ใช้ (โหมด WAL เพื่อ performance)
Schema กับ migration อยู่ใน `src/main/database.ts` ทั้งหมด

> **เทียบ Django:** ที่นี่ไม่มี ORM และไม่มีไฟล์ migration แยก
> ใช้วิธี: ฟังก์ชัน `createTables()` สำหรับติดตั้งใหม่ + `runMigrations()` ที่เช็ค `SCHEMA_VERSION`
> แล้วค่อยๆ `ALTER TABLE` (มีเอกสาร `docs/database-migrations.md` สอนวิธีเพิ่มคอลัมน์)

### ตารางทั้งหมด 11 ตาราง

| ตาราง | หน้าที่ |
|---|---|
| **tasks** | งานทั้งหมด: title, description, status, priority, labels, due_date, attachments, repos, agent_id, skill_ids, session_id, snoozed_until, ฟิลด์ recurring (cron), ฟิลด์ heartbeat, parent_task_id (subtask), feedback_rating/comment |
| **agents** | ค่าคอนฟิก agent แต่ละตัว: ชื่อ, coding_agent (claude-code/opencode/codex), model, system_prompt, mcp_servers, skill_ids, permission_mode |
| **mcp_servers** | รายการ MCP servers: local (command+args) หรือ remote (url+headers), source (user/enterprise/plugin) |
| **task_sources** | การเชื่อมต่อแหล่งงานภายนอก (Linear, Notion ฯลฯ): plugin_id, list_tool, update_tool, last_synced_at |
| **skills** | คลังความรู้ของ agent: name, content (markdown), version, confidence (0-1), uses, tags, soft delete |
| **heartbeat_logs** | ประวัติการเช็ค heartbeat: status, summary, session_id |
| **oauth_tokens** | OAuth tokens **เข้ารหัสด้วย Electron safeStorage** (เก็บเป็น BLOB) |
| **secrets** | API keys ต่างๆ เข้ารหัส ผูกกับ env_var_name เช่น `OPENAI_API_KEY` |
| **marketplace_sources** | แหล่ง marketplace ของ Claude plugins |
| **installed_plugins** | plugins ที่ติดตั้งแล้ว + manifest |
| **settings** | key-value settings ของแอป (เหมือน django-constance) |

### จุดที่น่าสนใจสำหรับ Backend Dev

- **ความปลอดภัย:** token/secret ทุกตัวเข้ารหัสด้วย `safeStorage` ของ Electron (ผูกกับ Keychain ของ OS) ก่อนเก็บลง DB
- **SQL Injection:** ใช้ parameterized queries ทั้งหมด
- **Soft delete:** ตาราง skills ใช้ flag `is_deleted` ไม่ลบจริง (เก็บประวัติไว้)
- **Subtasks:** ใช้ self-reference `parent_task_id` + `ON DELETE CASCADE` + `sort_order` สำหรับลากเรียง

---

## 6. วงจรชีวิตของ Task (Task Lifecycle)

Task มี 6 สถานะ (ประกาศใน `src/shared/constants.ts`):

```
not_started → triaging → not_started → agent_working → ready_for_review → completed ⇄ agent_learning
```

| สถานะ | ความหมาย |
|---|---|
| `not_started` | สร้างใหม่ รอถูกหยิบไปทำ |
| `triaging` | Triage agent กำลังวิเคราะห์เพื่อมอบหมายงาน |
| `agent_working` | AI agent กำลังทำงานอยู่ (session active) |
| `ready_for_review` | Agent ทำเสร็จแล้ว รอมนุษย์ตรวจ |
| `agent_learning` | กำลังเรียนรู้จาก feedback (สกัดเป็น skill) |
| `completed` | เสร็จสมบูรณ์ |

### Auto-Run

มีปุ่ม Play ที่ sidebar เมื่อเปิด ระบบ scheduler (`use-agent-auto-start.ts`) จะ:

1. หา task ที่ `status=not_started` + มี `agent_id` + ไม่ถูก snooze + ไม่มี session ค้าง
2. จัดกลุ่มตาม agent แล้วเรียงตาม priority (critical > high > medium > low)
3. แต่ละ agent รันพร้อมกันได้ตาม `max_parallel_sessions` (default: 1) ที่เหลือเข้าคิว
4. agent ว่างเมื่อไหร่ ดึงงานถัดไปจากคิวทันที
5. มี periodic check ทุก 60 วินาทีกันงานค้าง

> **เทียบ Django:** นี่คือ Celery worker + task queue ฉบับย่อส่วน ที่รันใน process เดียว

---

## 7. ระบบ AI Agent Orchestration (หัวใจของโปรเจค)

ไฟล์หลักคือ `src/main/agent-manager.ts` (ไฟล์ใหญ่ที่สุดในโปรเจค ~188KB)

### Adapter Pattern : รองรับ AI 3 ค่าย

ทุก agent backend implement interface เดียวกัน (`coding-agent-adapter.ts`):

```
CodingAgentAdapter (interface กลาง)
├── claude-code-adapter.ts  → @anthropic-ai/claude-agent-sdk (Claude ของ Anthropic)
├── opencode-adapter.ts     → @opencode-ai/sdk (open-source, spawn server แยก)
└── acp-adapter.ts          → @zed-industries/codex-acp (Codex ของ OpenAI)
```

methods หลัก: `createSession()`, `resumeSession()`, `streamSession()`, `abortSession()`, `approve()`, `stop()`

> **เทียบ Django:** เหมือนเราเขียน abstract class `PaymentGateway` แล้วมี implementation
> `StripeGateway`, `OmiseGateway`, `2C2PGateway` สลับกันได้โดย business logic ไม่ต้องรู้

### Session Lifecycle 6 ขั้นตอน

```
1. START      → เตรียม workspace (git worktree) → เขียนไฟล์ skills ลง workspace
              → สร้าง session ผ่าน adapter → ส่ง prompt เริ่มต้น

2. STREAMING  → Polling Coordinator กลาง poll ทุก 2 วินาที (timer เดียวสำหรับทุก session)
              → ดึง messages ใหม่ → dedup → ส่งให้ UI แสดงผ่าน IPC 'agent:output'
              → มี watchdog: session ค้าง 5 นาทีจะ abort อัตโนมัติ

3. HITL       → agent ขอ approve การกระทำเสี่ยง → UI แสดง banner
              → ผู้ใช้กด Approve/Reject (+ ข้อความได้) → ส่งกลับให้ agent

4. COMPLETE   → agent idle + ไม่มีข้อความใหม่ครบ grace period
              → เปลี่ยน status เป็น ready_for_review → แจ้ง UI

5. LEARNING   → ผู้ใช้ให้ดาว + คอมเมนต์ → ส่ง feedback กลับเข้า session เดิม
              → agent ปรับปรุง/สร้างไฟล์ SKILL.md → sync กลับเข้า DB

6. ABORT/STOP → ผู้ใช้สั่งหยุด → ส่งสัญญาณ abort → cleanup
```

### Use Case: ทำไมต้องมี "Polling Coordinator กลาง"?

**ปัญหา:** ถ้ามี 10 sessions และแต่ละ session ตั้ง `setInterval` ของตัวเอง จะมี timer 10 ตัว
ยิง DB writes พร้อมกันจน event loop ของ Node.js สำลัก (UI ค้าง)

**ทางแก้:** มี timer **ตัวเดียว** วนทุก 2 วินาที แล้วไล่ poll ทุก session ตามลำดับ
พร้อมจำกัดขนาด dedup structures (เช่น seenMessageIds) ไว้ที่ 5,000 รายการกัน memory บวม

> **เทียบ Django:** เหมือนการรวม cron jobs ยิบย่อยหลายตัวให้เป็น Celery Beat ตัวเดียว
> ที่ dispatch งานอย่างมีระเบียบ แทนที่จะปล่อยให้ N processes แย่งเขียน DB กันเอง

### HITL (Human-in-the-Loop) ละเอียดๆ

1. Agent จะทำสิ่งเสี่ยง เช่น `rm -rf dist/` หรือเขียนทับไฟล์
2. SDK ฝั่ง agent ยิง approval event มาที่ main process
3. Main process ส่ง IPC `agent:approval` ไปที่ UI
4. UI แสดง banner: "Agent wants to run `rm -rf dist/` [Approve] [Reject]"
5. การตัดสินใจของเราถูกส่งกลับผ่าน `agentSession:approve`
6. Agent ทำต่อหรือยกเลิกตามคำตอบ

---

## 8. Triage Agent : ระบบคัดแยกงานอัตโนมัติ

นี่คือฟีเจอร์เด่นของ 20x : เมื่องานใหม่เข้ามาโดย **ยังไม่มีใครกำหนดว่า agent ตัวไหนควรทำ**
ระบบจะส่ง "AI คัดแยกงาน" ไปวิเคราะห์ก่อน

### Flow การ Triage

```
งานใหม่ (ไม่มี agent_id, status=not_started)
    ↓ auto-run ตรวจพบ
status → 'triaging' แล้วปลุก default agent ขึ้นมา
    ↓ agent ใช้ MCP tools:
    → find_similar_tasks   (ค้นงานเก่าที่คล้ายกัน ด้วย SQL LIKE)
    → list_agents          (มี agent อะไรให้เลือกบ้าง)
    → list_skills          (มี skill อะไรบ้าง)
    → list_repos           (มี repo อะไรบ้าง)
    ↓ ตัดสินใจแล้วเรียก
    → update_task(agent_id, skill_ids, repos, priority, labels) ครั้งเดียว
    ↓
status กลับเป็น 'not_started' (แต่คราวนี้มี agent_id แล้ว)
    ↓
auto-run หยิบไปรันตามปกติ
```

### Use Case จริง

> ทีมคุณมี agent 2 ตัว: "Backend Agent" (ถนัด Django/API) กับ "Frontend Agent" (ถนัด React)
> มี ticket ใหม่จาก Linear เข้ามาว่า *"แก้ bug: API /orders ตอบ 500 เมื่อส่ง date format ผิด"*
>
> Triage agent จะค้นงานเก่าด้วยคีย์เวิร์ด "API", "500", "orders" เจอว่างานลักษณะนี้เคยถูก
> Backend Agent ทำสำเร็จมาแล้ว 5 ครั้ง จึงมอบหมายให้ Backend Agent
> พร้อมแนบ skill "django-error-handling" และ repo "company/backend-api"
> ตั้ง priority เป็น high แล้วถอยออกไป ให้ auto-run เริ่มงานจริงต่อ

### Safety Guards ของ Triage

- API `/update_task` จะ**บล็อกการเปลี่ยน status** ขณะอยู่ในสถานะ triaging (กัน agent มือลั่น)
- จำกัด retry ที่ 2 ครั้ง ถ้ายังหา agent ไม่ได้ จะ toast บอกให้มนุษย์เลือกเอง
- มีปุ่ม "Triage" (ไอคอนประกาย) ให้กดสั่งเองได้ในหน้า task detail

---

## 9. ระบบ Skills และ Auto-Learning

**Skill** = คลังความรู้/คำสั่งสำเร็จรูปที่ agent โหลดไปใช้ตอนทำงาน เก็บเป็น Markdown

### รูปแบบไฟล์ SKILL.md

```markdown
---
name: django-migration-safety
description: แนวทางการเขียน migration อย่างปลอดภัยใน production
---

## ขั้นตอน
1. ห้าม drop column ทันที ให้ deprecate ก่อน
2. ...
```

### กลไกการทำงาน

1. ตอนเริ่ม session ระบบ serialize skills จาก DB ไปเขียนเป็นไฟล์ใน workspace:
   `workspaces/<taskId>/.agents/skills/<skill-name>/SKILL.md`
2. Agent อ่านไฟล์เหล่านี้เป็น context ประกอบการทำงาน
3. ลำดับการเลือก skill: ระดับ task (`task.skill_ids`) > ระดับ agent (`agent.config.skill_ids`) > ไม่กำหนด (โหลดทั้งหมด)

### Auto-Learning Loop (ฟีเจอร์เด็ดสุด)

```
งานเสร็จ → ผู้ใช้ให้ดาว 1-5 + คอมเมนต์
    ↓
learnFromSession() ส่ง feedback เข้า session เดิม (status → agent_learning)
    ↓
Agent ทบทวนงานตัวเอง แล้วแก้ไข/สร้างไฟล์ SKILL.md ใน workspace
    ↓
syncSkillsFromWorkspace() สแกน .agents/skills/ กลับเข้า DB
    → match ตามชื่อ, version +1 อัตโนมัติเมื่อเนื้อหาเปลี่ยน
    → อัปเดต confidence (0-1) และตัวนับ uses
```

### Use Case จริง

> Agent ทำงาน "เพิ่ม endpoint ใหม่" แล้วลืมเขียน test คุณให้ 3 ดาวพร้อมคอมเมนต์
> "โค้ดโอเค แต่ลืม unit test ทุกครั้งเลย"
>
> ระบบส่ง feedback นี้กลับเข้า session agent จะอัปเดต skill "api-development" เพิ่มหัวข้อ
> "ต้องเขียน unit test ครอบ endpoint ใหม่เสมอ" → ครั้งถัดไป agent ทุกตัวที่โหลด skill นี้
> จะไม่ลืม test อีก นี่คือการที่ระบบ "ฉลาดขึ้นเอง" จาก feedback ของเรา

### Enterprise 2-way Sync

ถ้าองค์กรใช้ Peakflo Workflo skills จะ sync 2 ทาง ผ่านฟิลด์ `enterprise_skill_id`
มี drift detection (`uses_at_last_sync`) เพื่อรู้ว่าฝั่งไหนใหม่กว่า

---

## 10. Git Worktree Management

ไฟล์: `src/main/worktree-manager.ts`

### ปัญหาที่แก้

ถ้า agent 3 ตัวทำงาน 3 tasks ใน repo เดียวกันพร้อมกัน แล้วทุกตัวแก้ไฟล์ใน clone เดียวกัน = ชนกันพินาศ

### วิธีแก้: Git Worktree

```
~/<userData>/
├── repos/                      ← bare clone (โหลดครั้งเดียว ใช้ร่วมกัน)
│   └── <org>/<repo>.git
└── workspaces/                 ← worktree แยกต่อ task
    ├── <taskId-1>/<repo>/      ← branch: task/<taskId-1>
    └── <taskId-2>/<repo>/      ← branch: task/<taskId-2>
```

- ใช้ `gh repo clone --bare` (GitHub) หรือ `glab repo clone --bare` (GitLab)
- แต่ละ task ได้ branch `task/<taskId>` และโฟลเดอร์ของตัวเอง โดยแชร์ git objects กัน (ประหยัด disk)
- จบงานแล้ว `git worktree remove --force` ทิ้งได้สะอาดๆ

> **เทียบ Django:** เหมือนการที่แต่ละ test case ได้ database transaction แยกของตัวเอง
> ทำอะไรก็ไม่กระทบคนอื่น rollback ง่าย

---

## 11. Integrations กับเครื่องมือภายนอก

ทุก integration เป็น **plugin** อยู่ใน `src/main/plugins/` (มี `registry.ts` เป็นตัวกลาง โหลดทุก plugin)

| Integration | วิธี Auth | วิธี Sync | หมายเหตุ |
|---|---|---|---|
| **Linear** | OAuth | GraphQL API, polling | ดึง issues, อัปเดต status, โพสต์ comment |
| **HubSpot** | OAuth | REST API | sync CRM tickets/deals |
| **Notion** | OAuth | REST API v1 | sync database pages รองรับ properties ครบ + attachments |
| **YouTrack** | OAuth (custom) | REST API | รองรับ custom fields |
| **GitHub Issues** | ผ่าน `gh` CLI | REST API v3 | ดึง issues จาก repos |
| **GitLab** | ผ่าน `glab` CLI | REST API | repo discovery (task sync ยังอยู่ระหว่างพัฒนา) |
| **Peakflo Workflo** | Supabase JWT | REST API | sync 2 ทาง ทั้ง tasks และ skills (enterprise) |

### Pattern กลางของ Plugin

1. `listTasks()` ดึงงานจากระบบภายนอก
2. สร้าง local task โดยเก็บ `external_id` + `source_id` ไว้ map กลับ
3. Polling เป็นรอบ (ประมาณทุก 60 วินาที)
4. เมื่อ task ในแอปเปลี่ยน status จะยิง update กลับไปที่ต้นทาง

OAuth tokens ถูกเก็บเข้ารหัสในตาราง `oauth_tokens` (Electron safeStorage)
ส่วน OAuth callback ใช้ custom protocol ของ Electron

---

## 12. IPC Architecture : Frontend คุยกับ Backend ยังไง

นี่คือส่วนที่ backend dev ควรเข้าใจที่สุด เพราะมันคือ "REST API ภายในแอป"

### รูปแบบที่ 1: Request-Response (เหมือน REST API)

```
React component
  → เรียก window.electronAPI.db.getTasks()        (จาก preload bridge)
  → preload เรียก ipcRenderer.invoke('db:getTasks')
  → main process: ipcMain.handle('db:getTasks', ...) ทำงานกับ SQLite
  → return Promise กลับไปหา React
```

> **เทียบ Django:** `ipcMain.handle('db:getTasks')` = `path('tasks/', TaskListView.as_view())`
> แค่เปลี่ยนจาก HTTP เป็น IPC ภายใน process

### รูปแบบที่ 2: Event Streaming (เหมือน WebSocket/SSE)

```
main process: mainWindow.webContents.send('agent:output', data)   ← ยิงฝั่งเดียว
renderer: ipcRenderer.on('agent:output', callback)                ← subscribe
```

ใช้กับข้อมูลถี่ๆ เช่น output ของ agent แบบ real-time, approval requests, task refresh

### ตั้งชื่อ Channel แบบ `namespace:action`

`db:getTasks`, `agentSession:start`, `agentSession:approve`, `mcp:testConnection`,
`worktree:setup`, `terminal:spawn`, `oauth:callback` ฯลฯ

### HTTP Servers ภายใน (พิเศษ!)

นอกจาก IPC แล้ว main process ยังเปิด HTTP servers จริงๆ อีก 3 ตัว:

1. **task-api-server.ts** : ให้ **AI agent เรียกกลับเข้ามา** (ผ่าน env var `TASK_API_URL`)
   มี endpoints `/update_task`, `/get_task`, `/find_similar_tasks` ฯลฯ
   (เพราะ agent เป็น subprocess แยก ใช้ IPC ไม่ได้ ต้องใช้ HTTP)
2. **mobile-api-server.ts** : ให้มือถือ/เบราว์เซอร์เชื่อมเข้ามาดูงาน + WebSocket แจ้ง update
3. **secret-broker.ts** : แจก secrets ให้ agent ผ่าน per-session token
   (agent ไม่เคยเห็น API key ตรงๆ จนกว่าจะถามผ่าน broker)

---

## 13. Heartbeat Monitoring

ไฟล์: `src/main/heartbeat-scheduler.ts`

### มันคืออะไร

หลังจาก agent ทำงานเสร็จ (เช่น เปิด PR แล้ว) งานมัน "ยังไม่จบจริง" เพราะต้องรอ CI, รอ review
Heartbeat คือระบบที่คอยตามเช็คความคืบหน้าให้อัตโนมัติ

### กลไก

1. Agent เขียนไฟล์ `heartbeat.md` (checklist สิ่งที่ต้องตามเช็ค) ไว้ใน workspace
2. ผู้ใช้เปิด heartbeat บน task → ตั้ง `heartbeat_next_check_at`
3. Scheduler ตื่นทุก 60 วินาที หา task ที่ถึงเวลาเช็ค
4. spawn agent session เบาๆ ไปอ่าน `heartbeat.md` แล้วทำตาม checklist เช่น เช็คว่า CI เขียวไหม มี comment ใหม่ใน PR ไหม
5. บันทึกผลลง `heartbeat_logs` แจ้งเตือนผู้ใช้เมื่อ "ต้องการความสนใจ"

### Use Case จริง

> Agent เปิด PR #123 ไว้เมื่อเช้า คุณเปิด heartbeat แล้วไปทำอย่างอื่น
> บ่ายสอง CI ใน PR ล้ม heartbeat ตรวจเจอ ส่ง notification เด้งบนเครื่อง:
> "PR #123 ของ task X มี CI failure" คุณกดเข้าไปสั่ง agent แก้ต่อได้ทันที
> ไม่ต้องคอยนั่งเฝ้า GitHub เอง

---

## 14. ส่วนประกอบอื่นๆ ที่น่าสนใจ

### Infinite Canvas
`src/renderer/src/components/canvas/InfiniteCanvas.tsx` : พื้นที่ทำงาน 2D เลื่อน/ซูมได้
วาง panel ได้หลายชนิด: Task panel, Transcript panel (ดู agent ทำงานสด), **Terminal panel**
(terminal จริงในแอป ใช้ xterm.js + node-pty), Browser panel มี minimap และเส้นเชื่อมระหว่าง panel

### Recurring Tasks
`recurrence-scheduler.ts` : สร้างงานซ้ำตาม **cron expression** (แบบใหม่) หรือ JSON pattern (legacy)
ตัว template มี `is_recurring=true` แล้ว scheduler จะสร้าง instance ใหม่เมื่อ `next_occurrence_at` ถึงเวลา

### Task Snoozing
เลื่อนงานด้วย `snoozed_until` งานจะหายจากสายตา auto-run แล้วโผล่กลับมาเองเมื่อถึงเวลา

### Subtasks
แตกงานย่อยด้วย `parent_task_id` ลากจัดลำดับได้ (`sort_order` + dnd-kit)

### MCP Server Management
เพิ่ม MCP servers ได้เอง (local stdio หรือ remote HTTP) แล้ว agent ทุกตัวเรียกใช้ tools เหล่านั้นได้
มี Orchestrator panel ไว้ทดสอบยิง tool ตรงๆ

### Claude Plugin Marketplace
`claude-plugin-manager.ts` : ติดตั้ง Claude Code plugins จาก marketplace (GitHub repos)
ดึง skills และ MCP servers จาก plugin bundle มาใช้ในแอปได้

### Enterprise Mode (Supabase)
ตอบคำถามว่า "ทำไมมี `@supabase/supabase-js` ทั้งที่บอก local-first?"
คำตอบ: Supabase ใช้แค่ **login องค์กร** (OAuth + JWT) เพื่อ sync skills/MCP/task sources
กับ Peakflo Workflo เท่านั้น **ข้อมูลงานยังอยู่ใน SQLite ในเครื่อง 100%**

### Mobile Build
`vite.mobile.config.ts` build แอป React อีกตัวจาก `src/mobile/` (ออกที่ `out/mobile`)
เสิร์ฟผ่าน mobile-api-server ให้เปิดดูงานจากมือถือในวง LAN ได้

### Remotion
ใช้ render ภาพ preview ของ UI components (เช่นภาพ subtask UI) เป็น PNG สำหรับเอกสาร/มาร์เก็ตติ้ง

### Auto Updater + Crash Logger
เช็คเวอร์ชันใหม่จาก GitHub Releases ตอนเปิดแอป / เก็บ crash log ไว้ที่โฟลเดอร์ logs ของแอป

### WINDOWS_PORT_LOG.md
บันทึกบทเรียนตอน port จาก macOS ไป Windows 13+ ประเด็น เช่น ปัญหา titlebar, EPIPE,
การ unpack ASAR สำหรับ MCP servers, PowerShell wrapper สำหรับ secret broker, NSIS installer

---

## 15. วิธีติดตั้งและรันโปรเจค

### สิ่งที่ต้องมีก่อน

- **Node.js** (แนะนำ LTS) และ **pnpm 9** (สำคัญ: โปรเจคบังคับ pnpm ห้าม npm)
- **Git** และแนะนำ **gh CLI** (GitHub) / **glab CLI** (GitLab) สำหรับฟีเจอร์ worktree

### คำสั่ง

```bash
# 1. ติดตั้ง dependencies (จะ rebuild native modules ให้อัตโนมัติผ่าน postinstall)
pnpm install

# 2. รันโหมดพัฒนา (hot reload)
pnpm dev

# 3. รัน tests (ต้องรันผ่าน electron เพราะใช้ native modules)
pnpm test:run

# 4. ตรวจ type + lint
pnpm typecheck
pnpm lint

# 5. Build สำหรับแจกจ่าย
pnpm build:mac      # macOS (dmg + zip)
pnpm build:win      # Windows (NSIS installer)
pnpm build:linux    # Linux (AppImage)

# โหมด mobile web (แยกต่างหาก)
pnpm dev:mobile     # dev server port 5174
```

### หมายเหตุเรื่อง native modules

`better-sqlite3` และ `node-pty` เป็น native modules (C++) ต้อง compile ให้ตรงกับ Electron
สคริปต์ `scripts/rebuild-native.mjs` จัดการให้ตอน `pnpm install` ถ้า build พังให้ดูสคริปต์นี้ก่อน

### การใช้งานครั้งแรกในแอป

1. เปิดแอป จะเจอ **Onboarding Wizard** ตรวจว่ามี CLI tools ครบไหม (git, gh, node, agent CLIs) ถ้าไม่ครบติดตั้งให้ได้
2. ไปที่ **Settings** ตั้งค่า agent (เลือกค่าย AI + ใส่ API key หรือ auth)
3. เชื่อม integrations ที่ใช้ (Linear, Notion ฯลฯ) ผ่าน OAuth
4. สร้างหรือ sync tasks เข้ามา แล้วกดปุ่ม Play (auto-run) หรือสั่งรันทีละงาน

---

## 16. Use Case แบบ End-to-End

ลองดูภาพเต็มๆ หนึ่งรอบ ว่างานชิ้นหนึ่งเดินทางผ่านระบบยังไง:

```
[1] PM สร้าง issue ใน Linear: "เพิ่ม rate limiting ให้ API /login"
        ↓ (linear-plugin polling ทุก ~60 วิ)
[2] 20x ดึง issue เข้ามาเป็น task ใหม่ (status: not_started, ไม่มี agent_id)
        ↓ (auto-run เปิดอยู่)
[3] Triage Agent ตื่น (status: triaging)
    → ค้นงานเก่า เจอว่างาน security/API เคยให้ "Backend Agent" ทำ
    → มอบหมาย Backend Agent + skill "api-security" + repo "company/auth-service"
    → ตั้ง priority: high → status กลับเป็น not_started
        ↓
[4] Auto-run หยิบงาน → status: agent_working
    → worktree-manager สร้าง branch task/<id> ใน workspace แยก
    → เขียนไฟล์ SKILL.md ลง workspace
    → Claude Code agent เริ่มทำงาน เราดู transcript สดๆ ใน UI
        ↓
[5] Agent จะรัน "pip install slowapi" → HITL banner เด้งถามเรา → เรากด Approve
        ↓
[6] Agent เขียนโค้ด + test เสร็จ เปิด PR → status: ready_for_review
        ↓
[7] เราเปิด heartbeat → ระบบเช็ค CI/comments ใน PR ให้ทุกช่วงเวลา
    → CI ผ่าน, reviewer คอมเมนต์เล็กน้อย → agent แก้ตาม
        ↓
[8] เรา review พอใจ กด "Complete Task" → status: completed
    → ให้ 4 ดาว + คอมเมนต์ "ดี แต่ rate limit ควร config ได้ผ่าน env"
        ↓
[9] status: agent_learning → agent อัปเดต skill "api-security"
    เพิ่มข้อ "rate limit values ต้องอ่านจาก env เสมอ" → sync เข้า DB (version +1)
        ↓
[10] สถานะ "completed" sync กลับไปปิด issue ใน Linear อัตโนมัติ
     ครั้งหน้างานแบบนี้มา agent จะทำได้ดีขึ้นตั้งแต่ครั้งแรก
```

---

## 17. แนะนำเส้นทางการอ่านโค้ดสำหรับมือใหม่

เรียงตามลำดับที่แนะนำ:

1. **`README.md`** : ภาพรวม features
2. **`docs/task-lifecycle.md`** : เข้าใจ state machine ของงาน (สำคัญสุด)
3. **`src/shared/constants.ts`** : enum และค่าคงที่ทั้งหมด (ไฟล์เล็ก อ่านง่าย)
4. **`src/main/database.ts`** : ดู schema จะเข้าใจ data model ทั้งแอป (อ่านเฉพาะ `createTables()`)
5. **`src/main/ipc-handlers.ts`** : เหมือนอ่าน `urls.py` รู้ว่าแอปทำอะไรได้บ้าง
6. **`src/preload/index.ts`** : ดูว่า UI เรียกอะไรได้บ้าง
7. **`src/main/adapters/coding-agent-adapter.ts`** : interface กลางของ agent ก่อนไปอ่าน adapter จริง
8. **`src/main/agent-manager.ts`** : หัวใจของระบบ (ใหญ่มาก ค่อยๆ อ่านทีละ flow)
9. **`src/renderer/src/stores/`** : ดู state ฝั่ง UI
10. **`docs/` ที่เหลือ** : เจาะเรื่องที่สนใจ (skills.md, taskSources.md, enterprise-sync ฯลฯ)

### เอกสารใน docs/ มีอะไรบ้าง

| ไฟล์ | เนื้อหา |
|---|---|
| `task-lifecycle.md` | สถานะงาน + auto-run + triage (ต้องอ่าน) |
| `skills.md` | ระบบ skills + learning loop |
| `database-migrations.md` | วิธีเพิ่มคอลัมน์/แก้ schema อย่างถูกต้อง |
| `taskSources.md` | การเชื่อม task sources + OAuth lifecycle |
| `opencode.md`, `opencode-sdk.md`, `opencode-server.md` | การทำงานกับ OpenCode |
| `enterprise-sync-implementation.md`, `enterprise-tab-spec.md` | ระบบ enterprise + Workflo sync |
| `RECURRING_TASKS.md` | งานซ้ำ + cron format |
| `mobile-api-spec.md` | สเปค REST API + WebSocket ของ mobile |
| `LINEAR_OAUTH_SETUP.md` | ตั้งค่า OAuth app ของ Linear |
| `macos-signing-notarization.md` | sign + notarize แอปบน macOS |

---

*เอกสารนี้สร้างจากการสแกนโค้ดจริง ณ version 0.0.104 (มิถุนายน 2026)*
*ไฟล์คู่กัน: `PROJECT_GUIDE_TH.html` (เวอร์ชันอ่านง่ายในเบราว์เซอร์)*
