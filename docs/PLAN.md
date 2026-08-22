# SkedCEU — Product & Design Plan

Everything planned and decided for SkedCEU as of **2026-08-22**. This is the living feature plan; the README covers the SIA2 Lab 4 architecture deliverable.

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
- **COM storage (decided 2026-08-22):** after parsing, the COM file/image is kept **on-device only, encrypted, never uploaded** — the server only ever sees the structured schedule. Viewable anytime via **Settings → My COM** (Feature 12). Privacy promise wording: *"Your COM never leaves your phone."*
- The **import screen carries a "Not sure how? See the guide →" link** to the illustrated import FAQ (Feature 15) under the Import-from-CARES card.

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
- **Semester progress** (moved from v2 roadmap, 2026-08-22): a "Week 5 of 18" chip in the Today header, plus a days-until-finals countdown (also as a widget style).
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

### 11. Accounts & login (added 2026-08-22)
- Sign-up with **email verification** (register → verification email → confirm).
- **Biometric unlock** (fingerprint / Face) after first login — Flutter `local_auth`.
- Open sub-questions: restrict registration to `@*.ceu.edu.ph` addresses? Is an account required up-front, or optional until account-backed features (QR share, Plus) are used?

### 12. My COM viewer (added 2026-08-22 — decision: Option B)
- **Settings → My COM** shows the **original uploaded file exactly as issued** (the saved CARES page/PDF or the scanned photo) in a pinch-to-zoom document viewer — not a re-rendered app version, so it can never differ from the real thing.
- Stored **on-device only, encrypted at rest, never uploaded** — see Feature 1. Deleting a semester deletes its COM file.

### 13. Full course curriculum view (added 2026-08-22)
- Curriculum tab gains **"View full curriculum"**: the entire program (all years/semesters), color-coded with the student's own status — passed ✓ / enrolled (pink) / future (dashed) — plus a units-progress chip (e.g., "112 / 148 units done").
- Also a **"View official curriculum (PDF)" row**: opens the registrar-issued curriculum document untouched, for when the student needs the university's own wording. Admin-maintained per program, stored alongside the curriculum data rows so the two can't drift apart.
- Same admin-maintained curriculum data as Feature 6, wider lens. Also the anchor content for browsing mode (Feature 14).

### 14. First-run experience: onboarding + enrollment-aware empty state (added 2026-08-22)
- **Onboarding carousel** on first launch: 3–4 swipeable slides (import once → whole schedule; widgets & reminders; curriculum tracking) ending on the Import COM screen.
- **The app works without an import — "browsing mode":** full course curriculum (Feature 13), FAQs, and onboarding content are available; Schedule and Tuition tabs show the enrollment-aware empty state instead of dead screens.
- **Enrollment-aware empty state:** with no semester imported, the app asks *"Enrolled already?"* → Import your COM; otherwise shows enrollment-period info (e.g., *"Enrollment is open until …"*) sourced from the CEU academic-calendar data (`AcademicEvent` — same admin-maintained table as Feature 4; needs per-term upkeep).

### 15. Help & import guide FAQ (added 2026-08-22)
- **Settings → Help / FAQs**, headlined by an **illustrated step-by-step guide with screenshots**: log into CARES → save/share the COM page → upload it in SkedCEU.
- Linked directly from the Import COM screen ("Not sure how? See the guide →").

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

- COM contains PII (name, student number, address, balances) — **stored on-device only, encrypted, never uploaded** (updated 2026-08-22; was "deleted after parsing"). The server only ever receives the structured schedule. Promise wording: *"Your COM never leaves your phone."*
- Tuition amounts: **on-device only**.
- QR share: **class times only**, expiring, revocable.
- The real COM file used for design reference stays local (`Downloads\CEUMANILA.html`) — never committed or published.

## v2 Roadmap (approved 2026-08-21 — out of scope for the Lab 4 v1)

1. **Export to Google Calendar / .ics** — one-tap export of the semester schedule as an .ics file (rooms in the location field), importable into Google/Samsung/Apple Calendar. Cheap (an .ics is plain text) and adds another clean integration to the architecture.
2. **Absence tracker** — a "mark absent" action on each class; the app knows each subject's units and meeting hours, so it computes the allowed-absence cap and warns "2 absences left in Networking 3." Quiet, high-value anxiety relief.
3. ~~**Semester progress**~~ — **promoted into the current feature set 2026-08-22** (see Feature 3): "Week 5 of 18" chip in the schedule header plus a days-until-finals countdown.
4. **Custom events** (approved 2026-08-22) — manually add one-off or recurring entries (org meetings, consultations, appointments) that appear in Today/Week views, grid, reminders, and widgets alongside classes.
5. **Exam mode** (approved 2026-08-22) — enter a midterms/finals exam schedule for a date range; during that window the exam timetable replaces/overlays the regular classes, then the normal schedule returns automatically.

**Considered and rejected:** grade tracker / GWA calculator (user decision — out of the app's scope).
**Undecided (explained, awaiting decision):** room finder (room-code → building/floor hint on the subject detail screen; data-maintenance burden is the concern — lightweight alternative: free-text location note per subject).

## Pending work

- [x] Diagram PNG re-exported and committed (2026-08-21)
- [x] README synced with the full design (portal import, timetable views, tagline, PLAN.md link)
- [x] `Olores_SIA2_Lab4.pdf` regenerated with final content and fresh screenshots (2026-08-22)
- [ ] User uploads PDF to Canvas (due **Sat Aug 29, 2026 11:59 PM**, 3 attempts)
- [x] Decide on custom events / exam mode → **approved into v2** (2026-08-22)
- [ ] Decide on room finder (still open)
- [ ] Decide Feature 11 sub-questions: CEU-email-only registration? account required vs optional?
- [x] Design pass for the 2026-08-22 features (canvas artboards): onboarding carousel, enrollment-aware empty state, Settings → My COM (original-document viewer), full-curriculum view (+ official-PDF row), import-guide link on the Import screen, Week-of-semester chip on Today
- [ ] Source the real registrar curriculum PDF for the official-curriculum row (per program)
- [ ] (Future) Replace the invented "PRCO141 Network Management" unlock example with real BSIT curriculum data
