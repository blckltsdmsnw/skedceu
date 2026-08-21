# SkedCEU — Product & Design Plan

Everything planned and decided for SkedCEU as of **2026-08-21**. This is the living feature plan; the README covers the SIA2 Lab 4 architecture deliverable.

> **Design canvas (interactive mockups):** https://claude.ai/code/artifact/f337db59-5eea-401d-a561-925319ebc356
> Local design working files: `Documents\SIA PROJECT\design-workfiles\`

---

## Vision

Scan or import your Certificate of Matriculation once, and your whole semester lives on your phone: schedule, widgets, reminders, curriculum map, tuition deadlines — and the ability to share your schedule with the people who matter.

## Design direction — "CEU Pink" (chosen)

- Background `#fbf9fa` · Text `#1c1720` · Accent `#d6247a` · Muted `#a89aa2` · Card border `#f0e4eb` · Pink tint `#fbe7f1`
- Font: **Sora** (fallback system-ui) · Cards radius 14–18px · 390×844 reference frame
- Per-subject timetable colors: SysAd pink `#b0327a`, IAS 1 blue `#3b5f8a`, Research 2 purple `#6b5fa8`, Game Dev green `#3d7a52`, SIA 2 amber `#8a6a3b`, Networking teal `#2d7a6e`
- Rejected directions (kept as reference on canvas page 2): Midnight (dark), Pastel Planner (cream)

## Information architecture

**Tab bar: Schedule · Curriculum · Tuition · Settings** — four tabs, no center FAB.

- Scanning/importing a COM is a once-per-semester action, so it is NOT a tab. It lives in **onboarding** (first launch) and **Settings → Semesters → add new via COM import**.
- Premium is NOT a tab and NOT a headline feature — it exists only as the **SkedCEU Plus** row in Settings.

## Screens (canvas page 1, in flow order)

1. **Import COM** — three entry paths (see Feature 1) + privacy note
2. **Review & Confirm** — parsed subjects, editable, warnings, unit total, save button
3. **Today** — next-class card, tuition banner, day timeline **Grid | List** toggle, Share chip
4. **Week** — full timetable grid (days × hours, colored blocks), **Grid | List** toggle, holiday banner, legend
5. **Curriculum** — prerequisite flowchart + subject list with badges
6. **Tuition** — payment plan timeline + reminders
7. **Settings** — profile, notifications, widgets, appearance, semesters, SkedCEU Plus, privacy, sign out
8. **Share via QR** — QR card, scan-a-friend, share-as-image, find-time-together
9. **Subject Details** — tap any class anywhere to open it: COM-sourced schedule (editable) + user-added professor, meeting link, notes, color, per-subject reminder

---

## Features

### 1. COM import (three paths, in priority order)
- **Import from CARES (primary):** the COM is an HTML page from the CARES student portal (Ledgea School ERP) — students can save/share that page and the app parses it **deterministically** (field IDs + enrollment table). No OCR errors.
- **Scan a photo (fallback):** OCR (Google Cloud Vision) for students who only have a printout.
- **Manual entry (last resort).**
- Always followed by the **Review & Confirm** screen — extracted schedule is shown for correction before saving. OCR/parsing mistakes never silently corrupt a schedule.
- COM images/files are **deleted after parsing**; only the structured schedule is kept.

### 2. Parsing rules (learned from a real COM)
- Subjects come as **lecture + lab pairs** (e.g., PRCO123 / PRCO123L) — group under one subject with two meeting blocks.
- Schedule codes: day letters with **multi-day tokens** (`MT 1700-1800` = Mon+Tue), 24h ranges, `S` = Saturday.
- **Merged sections** (`BSCS4A/BSIT4A`) and cross-year sections (`BSIT3A`) are labeled.
- COM warnings like `[NON-CURRIC]` are captured and surfaced (see Feature 6).
- Also extractable: units per subject, total units, school year/semester, payment plan and due dates.

### 3. Schedule views
- **Today:** next-class hero card (subject, time, room, countdown badge) + single-day hour timeline (grid) or row list — **Grid | List toggle**.
- **Week:** visual timetable grid — days Mon–Sat across, hours 7a–7p down, color-coded blocks per subject, legend below; **Grid | List toggle** (list = grouped day sections).
- Home screen + lock screen **widgets**; schedule cached on-device so everything works offline.
- **Semester archive:** each new sem is a new import; old schedules kept.

### 4. Holiday & CEU calendar sync
- Backend periodically pulls the **Google Calendar public Philippine Holidays feed** (REST) into `academic_events`.
- **CEU-specific dates** (class suspensions, university events) maintained as admin data in the same table.
- Holidays render as a **"no classes" banner** replacing that day's class rows/blocks (e.g., Aug 31 — National Heroes Day).

### 5. Reminders & meeting links
- Push notification before each class **with the room number** (FCM/APNs).
- A subject can carry an **online-meeting link** (Google Meet etc.); synchronous classes show a **Join** chip on the schedule row/block and in the reminder notification.

### 6. Curriculum & prerequisites
- Per-program curriculum stored as **admin-maintained data** (new program = data rows, not code).
- **Prerequisite flowchart** per subject: passed prerequisites (green ✓) → currently enrolled (pink) → **future subjects unlocked if you pass** (dashed, "if you pass"). Reads top-to-bottom like a flowchart.
- **NON-CURRIC warning badges** on subjects outside the student's curriculum (mirrors the COM's own flag).
- **Units/workload summary:** per-subject units + semester total (e.g., "18 units this sem").

### 7. Tuition timeline
- Installment plan (e.g., Plan D) rendered as a **timeline**: Paid ✓ → Due now → Reminder set → Upcoming, with due dates and amounts.
- Configurable reminders (e.g., 3 days before each due date).
- **Privacy: amounts and due dates are stored on-device only — never on the server.** (Mockups use fake amounts deliberately.)

### 8. Schedule sharing
- **QR share (live):** shares the **whole semester's recurring weekly schedule** — class times only, never personal info (no student number, address, etc.). The share **expires after 7 days** and is **revocable anytime** in Settings. Architecture: share token encoded in the QR, resolved via the REST API.
- **Find time together:** when two students add each other, the app highlights the hours **both are free** — the core value for couples/friends/group-mates.
- **Share as image (snapshot):** export today's or the week's schedule as an image for any chat (Messenger etc.) — one-off, static, no app or account needed by the recipient.
- Rationale for whole-sem scope: a class schedule is a weekly pattern; expiry controls access duration, not content; free-time overlap requires both full weekly patterns.

### 9. Subject details & editing
- The COM provides only code, name, section, units, times, and rooms — **professor names and meeting links are NOT on the COM**, so they are user-added.
- **Tapping any class row/block** (Today, Week, Curriculum) opens the **Subject Detail screen**:
  - *From your COM* (editable): meeting times and room — with an edit pencil, since parsing can need correction mid-sem.
  - *Details you add*: professor name, online-meeting link (powers the Join button and reminder deep-link), personal notes, subject color, per-subject reminder toggle/offset.
- Future idea (not committed): per-section crowd-sourcing of professor names/links between students who share a section.

### 10. Monetization — SkedCEU Plus
- App is **free**; all core features (import, schedule, reminders, curriculum, tuition, sharing) are free.
- One-time payment unlocks **cosmetics only**: more widget styles, custom backgrounds/themes.
- Surfaced only as a quiet row in Settings. Purchases via platform stores; confirmation via **webhook**; no card data touches the backend.

---

## Data model additions (beyond the Lab 4 README)

| Model | Purpose |
|---|---|
| `AcademicEvent` | PH holidays + CEU dates affecting classes (already in README) |
| `MeetingLink` (or field on `ScheduleEntry`) | Online-meeting URL per subject (already in README) |
| `ShareToken` | QR share: owner, scope (semester), expiry (7 days), revoked flag |
| `PaymentPlanEntry` | On-device only: plan label, due date, amount, status, reminder offset |

## Integrations summary

1. REST → Google Cloud Vision (OCR, on-demand)
2. REST → Google Calendar PH Holidays feed (periodic pull)
3. Webhook ← Google Play / App Store billing (purchase/refund)
4. FCM / APNs (class + tuition reminders)
5. REST (own API) — share-token resolution for QR shares

## Privacy rules (non-negotiable)

- COM contains PII (name, student number, address, balances) — **deleted after parsing**; only structured schedule kept.
- Tuition amounts: **on-device only**.
- QR share: **class times only**, expiring, revocable.
- The real COM file used for design reference stays local (`Downloads\CEUMANILA.html`) — never committed or published.

## v2 Roadmap (approved 2026-08-21 — out of scope for the Lab 4 v1)

1. **Export to Google Calendar / .ics** — one-tap export of the semester schedule as an .ics file (rooms in the location field), importable into Google/Samsung/Apple Calendar. Cheap (an .ics is plain text) and adds another clean integration to the architecture.
2. **Absence tracker** — a "mark absent" action on each class; the app knows each subject's units and meeting hours, so it computes the allowed-absence cap and warns "2 absences left in Networking 3." Quiet, high-value anxiety relief.
3. **Semester progress** — "Week 7 of 18" in the schedule header plus a days-until-finals countdown (also as a widget style). Tiny effort, strong emotional texture.

**Considered and rejected:** grade tracker / GWA calculator (user decision — out of the app's scope).
**Undecided (explained, awaiting decision):** custom events & exam mode (one-off entries so midterm/finals schedules and org meetings appear correctly); room finder (room-code → building/floor hint on the subject detail screen).

## Pending work

- [ ] User re-exports `docs/architecture.drawio` → `docs/architecture.png` (diagram gained the holidays-feed box)
- [ ] Fold features 1–3 and 6–8 of this plan into the README (holiday sync + meeting links already committed in `cd27d0b`)
- [ ] Regenerate `Olores_SIA2_Lab4.pdf` with updated README content + new PNG
- [ ] User uploads PDF to Canvas (due **Sat Aug 29, 2026 11:59 PM**, 3 attempts)
- [ ] (Future) Replace the invented "PRCO141 Network Management" unlock example with real BSIT curriculum data
