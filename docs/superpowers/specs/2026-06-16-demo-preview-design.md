# Demo Preview Design

## Context

The current project is a single-page landing/pitch for "Где болит?" with a small embedded survey demo. The knowledge base positions the product as an AI-assisted medical navigator: it helps users reduce uncertainty, understand urgency, choose a likely next specialist, prepare for a visit, and keep a history of cases. It does not diagnose, replace a doctor, or promise to bypass primary care.

This design adds a fuller product preview that opens from the landing page through a "Попробовать демо" button. The preview must use only artificial data and exist only to demonstrate the product idea.

## Goals

- Let a viewer use a believable web-app preview from a desktop computer.
- Preserve the current landing page and visual language.
- Demonstrate the core value loop: login, history, case card, new symptom intake, route result, follow-up state.
- Show that the product reduces anxiety and organizational load without sounding like a doctor.
- Make every clinic, doctor, document, price, slot, lab, case, and user detail clearly synthetic.

## Non-Goals

- No real authentication.
- No backend, database, API, file upload, payment, or clinic integration.
- No medical diagnosis, treatment recommendation, or claim that the app replaces a doctor.
- No real clinic availability, real laboratory prices, or real personal medical data.
- No redesign of the whole landing page beyond adding an entry point and preview mode.

## Recommended Approach

Use a fullscreen in-page demo app launched from the existing landing page. The landing page remains the pitch, while the demo behaves like a product surface:

1. User clicks "Попробовать демо".
2. The preview switches to a login screen.
3. User clicks a fake "Войти через Яндекс" button.
4. The app opens a desktop web cabinet with history and a selected case.
5. User can inspect synthetic cases or start a new intake.
6. After the intake, the app shows a route and can save the new case as "В прогрессе".

This keeps the experience coherent and avoids sending the user to a separate artifact while still making the preview feel like a usable app.

## UX Principles

- Calm and practical: the app should feel like a navigator, not a hospital system or a medical chatbot.
- Explainable: every recommendation should include a short "why this step matters".
- Safe by default: red flags should increase urgency and point the user toward timely care.
- Neutral: DocMed, Medsi, labs, doctors, and prices are demo entities, not endorsements.
- Desktop-first: the preview is designed for PC demonstrations, with responsive fallbacks where reasonable.
- Synthetic transparency: visible demo badges and copy should clarify that all data is artificial.

## Main Screens

### Landing Entry

Add or wire a "Попробовать демо" button in the current landing page style. It opens the demo mode without replacing the landing permanently. The user should be able to return to the landing from the demo.

### Login Screen

The login screen uses the same visual tone as the landing page: light background, blue accent, calm typography, restrained panels.

Content:

- Brand: "Где болит?"
- Description: "Демо медицинского навигатора"
- Visible note: "Все данные в этом превью искусственные"
- Fake auth button: "Войти через Яндекс"
- Secondary note: "Авторизация не выполняется, это демонстрационный режим"

Clicking the auth button immediately enters the app.

### App Shell

The authenticated preview is a fullscreen desktop web app.

Layout:

- Top bar with brand, demo badge, synthetic user profile "Аня, 24", and "Выйти из демо".
- Left sidebar with case history, filters, and "Новое обращение".
- Main content area with either the selected case card, the intake wizard, or the generated route result.

The visual style should be denser than the landing page, because this is a working interface. Use the current palette and typography, but reduce marketing-scale headings.

### Case History

Filters:

- Все
- В прогрессе
- Завершены

Each history item shows:

- Case title
- Short symptom summary
- Status
- Date
- Urgency label
- Next step or result

Selecting an item opens its detailed case card.

### Case Card

Each case card is the central product artifact. It should show the route rather than a diagnosis.

Sections:

- Summary: symptom, status, urgency, date.
- Route: suggested next step and explanation.
- Analysis recommendations where relevant, including synthetic price/options.
- Doctor recommendations where relevant, including synthetic clinic, specialty, slot, and rationale.
- Documents where relevant, including synthetic uploaded discharge notes.
- Follow-up/result: what changed after the user acted.
- Safety note: "Это не диагноз. Медицинские решения принимает врач."

## Synthetic Demo Cases

### Case 1: Mild Symptoms, Completed

Title: "Першение в горле и слабость"

State:

- Status: "Завершено"
- Urgency: low, no red flags
- Route: observe for 48 hours, rest, fluids, temperature monitoring
- Follow-up: repeated mini-survey after two days, user reports symptoms passed
- Outcome: "Визит не потребовался"

Purpose: show that the product does not send every user to a doctor.

### Case 2: Abdominal Pain, Completed After Route

Title: "Тянущая боль в животе после еды"

State:

- Status: "Завершено"
- Urgency: planned visit unless symptoms worsen
- Suggested route: gastroenterologist, with possible therapist start if symptoms become mixed
- Analysis examples: CBC, biochemistry, abdominal ultrasound, stool test
- Synthetic lab recommendations: show cheapest or most convenient options with clearly fake prices
- Synthetic doctors:
  - "Елена Морозова", gastroenterologist, DocMed, fictional
  - "Илья Романов", therapist-gastroenterology focus, Medsi, fictional
- Uploaded synthetic documents: appointment note and test summary
- AI summary: what was prescribed, when the appointment is, what to do before it, and what questions to ask

Purpose: show the full preparation and follow-up value loop.

### Case 3: Open Case, In Progress

Title: "Головные боли после работы за компьютером"

State:

- Status: "В прогрессе"
- Current route: check red flags, track blood pressure, consider neurologist and ophthalmologist
- Next action: choose appointment slot or continue context collection
- No final outcome yet

Purpose: make history feel alive and show ongoing case support.

### Case 4: New Case Created From Intake

This case is generated when the user completes the intake wizard and saves the route. It appears in history as "В прогрессе".

## Intake Wizard

The new case flow should be deeper than the landing demo and use practical questions.

Steps:

1. Area of concern: stomach/digestion, head, throat/cold symptoms, back/joints, skin, other.
2. Free-text description in everyday language.
3. Timing: today, 2-3 days, more than a week, recurring for months.
4. Intensity and impact: mild, noticeable, interferes with work/study, severe.
5. Red flags: strong sharp pain, fever, fainting, chest pain, blood, rapid worsening, none.
6. Context: known conditions, recent medicines, recent travel/food changes, stress/sleep factors.
7. Constraints: DMS/private/self-pay, budget sensitivity, preferred area, available times.

The wizard should allow moving forward and back. It should not require real personal data.

## Route Result

After the wizard, the app shows a result with:

- Urgency level.
- Suggested first step.
- Specialist direction.
- Short explanation.
- Analysis list if appropriate.
- Synthetic lab comparison: cheapest and convenient options.
- Synthetic doctors/slots matching the specialty and availability preferences.
- A "save route" action that adds the case to history.

For red flags, the result should emphasize timely medical help rather than scheduling optimization.

## Data Model

All data is static JavaScript state inside the page.

Core entities:

- `demoUser`: synthetic profile.
- `cases`: array of synthetic case objects.
- `labs`: synthetic lab options and prices.
- `doctors`: synthetic doctor options with clinic, specialty, slots, and rationale.
- `documents`: synthetic uploaded note summaries.
- `wizardState`: current intake answers.

No data is persisted beyond the current page session. Use in-memory state only; after reload the demo returns to the predefined synthetic state.

## Components

- `DemoLauncher`: opens demo mode from the landing page.
- `DemoLogin`: fake Yandex login screen.
- `DemoAppShell`: top bar, sidebar, main panel.
- `CaseList`: filters and history cards.
- `CaseCard`: detailed route and status card.
- `IntakeWizard`: multi-step new case form.
- `RouteResult`: generated route, analyses, doctors, and save action.

The implementation should follow the existing single-file demo style unless the implementation plan finds a cleaner local pattern that can be introduced without broad refactoring.

## Copy And Safety Rules

Use:

- "маршрут"
- "следующий шаг"
- "подготовка к визиту"
- "можно обсудить с врачом"
- "по описанию полезно начать с..."
- "если симптомы усиливаются, не откладывайте обращение"

Avoid:

- "диагноз"
- "у вас ..."
- "лечение"
- "точно"
- "можно пропустить врача"
- "ИИ-врач"

Every detailed result should include a concise boundary statement: "Это не диагноз. Сервис помогает подготовиться к обращению, а медицинские решения принимает врач."

## Error And Empty States

Because this is a static demo, error handling is mostly interaction polish:

- Disabled next button until required wizard choice is made.
- Friendly empty state if a filter has no cases.
- Clear state if no case is selected.
- Fake upload state can show already-loaded demo documents rather than accepting real files.
- Fake auth cannot fail.

## Verification Plan

Manual verification is enough for this preview:

1. Open the page.
2. Click "Попробовать демо".
3. Confirm login screen shows synthetic-data notice.
4. Click "Войти через Яндекс".
5. Open each of the three predefined cases.
6. Check that each case has a detailed card and no diagnostic wording.
7. Start a new case, complete all wizard steps, and view route result.
8. Choose a doctor/slot or save route.
9. Confirm the new case appears in history as "В прогрессе".
10. Check the layout on desktop width and a narrower viewport for obvious overlap.

## Acceptance Criteria

- The current landing page remains accessible.
- "Попробовать демо" opens the preview.
- Fake Yandex login works without network calls.
- The app has three predefined synthetic cases matching the scenarios above.
- Each case has a detailed route card.
- At least one case shows completed observation with symptoms resolved.
- At least one case shows analyses, synthetic clinic/doctor recommendations, synthetic uploaded documents, and a summarized plan.
- At least one case remains in progress.
- The new intake has multiple steps and produces a route result.
- Saving the result adds a new in-progress case to the history.
- All demo data is visibly synthetic.
- The UI matches the existing style closely enough to feel like the same product.
- The app avoids diagnosis/treatment claims and keeps the doctor boundary clear.
