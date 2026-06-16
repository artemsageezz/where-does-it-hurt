# Demo Preview Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a fullscreen synthetic web-app preview launched from the existing landing page through "Попробовать демо".

**Architecture:** Keep the current bundled landing page intact and inject one self-contained demo module into the HTML template stored in `index.html` under `script[type="__bundler/template"]`. The module adds demo entry buttons to the rendered landing page, owns all in-memory demo state, and renders the login/app/wizard UI in a fullscreen overlay.

**Tech Stack:** Static HTML, inline CSS, vanilla JavaScript, existing bundled fonts/styles, no backend, no package install.

---

## File Structure

- Modify: `index.html`
  - Add a self-contained `<script id="where-does-it-hurt-demo-preview">...</script>` inside the bundled template string, immediately before the template's closing `</body>`.
  - The script creates its own `<style>` tag, entry buttons, overlay root, synthetic data, in-memory state, render functions, and event handlers.
  - The script must not make network calls, read files, use real auth, or persist data.
- No new production files.
- Existing docs remain:
  - Design spec: `docs/superpowers/specs/2026-06-16-demo-preview-design.md`
  - This plan: `docs/superpowers/plans/2026-06-16-demo-preview.md`

## Implementation Notes

The current `index.html` is a generated bundle. The actual rendered page lives as an escaped JSON string in `script[type="__bundler/template"]` around line 179. Do not edit the initial loader script or manifest. Insert the demo module into the template content so the bundler recreates and executes it after the document is unpacked.

Recommended edit method during execution:

1. Use a small local script or editor operation to parse the JSON string from `script[type="__bundler/template"]`.
2. Insert the demo module before `</body></html>` inside the parsed template string.
3. Serialize the template string back as JSON so escaping stays valid.
4. Keep the manifest unchanged.

The inserted script should have a stable marker:

```html
<script id="where-does-it-hurt-demo-preview">
```

This marker makes verification and future edits straightforward.

---

### Task 1: Confirm Bundle Insertion Point

**Files:**
- Read: `index.html`
- Modify: none

- [ ] **Step 1: Verify the template script exists**

Run:

```bash
rg -n "__bundler/template|</body></html>|where-does-it-hurt-demo-preview" index.html
```

Expected:

- `__bundler/template` is present.
- `where-does-it-hurt-demo-preview` is not present before implementation.

- [ ] **Step 2: Verify the template JSON can be parsed**

Run:

```bash
python3 - <<'PY'
from pathlib import Path
import json, re
text = Path("index.html").read_text()
m = re.search(r'<script type="__bundler/template">\s*(.*?)\s*</script>', text, re.S)
assert m, "template script not found"
template = json.loads(m.group(1))
assert "</body></html>" in template, "template closing body/html not found"
assert "Где&nbsp;болит" in template, "landing content not found in template"
print("template ok", len(template))
PY
```

Expected:

```text
template ok <number>
```

- [ ] **Step 3: Commit checkpoint**

No commit is needed for this read-only task.

---

### Task 2: Add Demo Module Skeleton

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write the failing marker check**

Run:

```bash
rg -n "where-does-it-hurt-demo-preview|Попробовать демо" index.html
```

Expected: command exits with no matches, unless another worker already implemented this task.

- [ ] **Step 2: Insert the module skeleton**

Add this script to the parsed bundled template immediately before `</body></html>`:

```html
<script id="where-does-it-hurt-demo-preview">
(function () {
  const DEMO_ROOT_ID = "wdih-demo-root";
  const STYLE_ID = "wdih-demo-style";

  const state = {
    mode: "landing",
    view: "login",
    selectedCaseId: "case-throat",
    filter: "all",
    wizardStep: 0,
    wizardAnswers: {},
    generatedRoute: null,
    cases: []
  };

  function init() {
    state.cases = typeof structuredClone === "function" ? structuredClone(DEMO_CASES) : JSON.parse(JSON.stringify(DEMO_CASES));
    injectStyles();
    injectLandingButtons();
    ensureRoot();
  }

  function ensureRoot() {
    if (document.getElementById(DEMO_ROOT_ID)) return;
    const root = document.createElement("div");
    root.id = DEMO_ROOT_ID;
    document.body.appendChild(root);
  }

  function openDemo() {
    state.mode = "demo";
    state.view = "login";
    render();
  }

  function closeDemo() {
    state.mode = "landing";
    state.view = "login";
    state.selectedCaseId = "case-throat";
    state.filter = "all";
    state.wizardStep = 0;
    state.wizardAnswers = {};
    state.generatedRoute = null;
    state.cases = typeof structuredClone === "function" ? structuredClone(DEMO_CASES) : JSON.parse(JSON.stringify(DEMO_CASES));
    render();
  }

  function login() {
    state.view = "case";
    render();
  }

  function render() {
    const root = document.getElementById(DEMO_ROOT_ID);
    if (!root) return;
    if (state.mode !== "demo") {
      root.innerHTML = "";
      return;
    }
    root.innerHTML = state.view === "login" ? renderLogin() : renderApp();
  }

  const DEMO_CASES = [];

  window.whereDoesItHurtDemo = { open: openDemo, close: closeDemo };

  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", init);
  } else {
    init();
  }
})();
</script>
```

Use this Python insertion command, replacing `MODULE_HTML` with the skeleton above:

```bash
python3 - <<'PY'
from pathlib import Path
import json, re
path = Path("index.html")
text = path.read_text()
module = r'''<script id="where-does-it-hurt-demo-preview">
(function () {
  const DEMO_ROOT_ID = "wdih-demo-root";
  const STYLE_ID = "wdih-demo-style";

  const state = {
    mode: "landing",
    view: "login",
    selectedCaseId: "case-throat",
    filter: "all",
    wizardStep: 0,
    wizardAnswers: {},
    generatedRoute: null,
    cases: []
  };

  function init() {
    state.cases = typeof structuredClone === "function" ? structuredClone(DEMO_CASES) : JSON.parse(JSON.stringify(DEMO_CASES));
    injectStyles();
    injectLandingButtons();
    ensureRoot();
  }

  function ensureRoot() {
    if (document.getElementById(DEMO_ROOT_ID)) return;
    const root = document.createElement("div");
    root.id = DEMO_ROOT_ID;
    document.body.appendChild(root);
  }

  function injectStyles() {}
  function injectLandingButtons() {}
  function renderLogin() { return ""; }
  function renderApp() { return ""; }

  function openDemo() {
    state.mode = "demo";
    state.view = "login";
    render();
  }

  function closeDemo() {
    state.mode = "landing";
    state.view = "login";
    state.selectedCaseId = "case-throat";
    state.filter = "all";
    state.wizardStep = 0;
    state.wizardAnswers = {};
    state.generatedRoute = null;
    state.cases = typeof structuredClone === "function" ? structuredClone(DEMO_CASES) : JSON.parse(JSON.stringify(DEMO_CASES));
    render();
  }

  function login() {
    state.view = "case";
    render();
  }

  function render() {
    const root = document.getElementById(DEMO_ROOT_ID);
    if (!root) return;
    if (state.mode !== "demo") {
      root.innerHTML = "";
      return;
    }
    root.innerHTML = state.view === "login" ? renderLogin() : renderApp();
  }

  const DEMO_CASES = [];

  window.whereDoesItHurtDemo = { open: openDemo, close: closeDemo };

  if (document.readyState === "loading") {
    document.addEventListener("DOMContentLoaded", init);
  } else {
    init();
  }
})();
</script>'''
m = re.search(r'(<script type="__bundler/template">\s*)(.*?)(\s*</script>)', text, re.S)
assert m, "template script not found"
template = json.loads(m.group(2))
assert 'where-does-it-hurt-demo-preview' not in template, "demo module already inserted"
assert "</body></html>" in template, "closing body/html not found"
template = template.replace("</body></html>", module + "\n</body></html>", 1)
text = text[:m.start(2)] + json.dumps(template, ensure_ascii=False) + text[m.end(2):]
path.write_text(text)
PY
```

- [ ] **Step 3: Verify marker and JSON parse**

Run:

```bash
rg -n "where-does-it-hurt-demo-preview" index.html
python3 - <<'PY'
from pathlib import Path
import json, re
text = Path("index.html").read_text()
m = re.search(r'<script type="__bundler/template">\s*(.*?)\s*</script>', text, re.S)
template = json.loads(m.group(1))
assert 'id="where-does-it-hurt-demo-preview"' in template
print("demo skeleton inserted")
PY
```

Expected:

```text
demo skeleton inserted
```

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Add demo preview module skeleton"
```

---

### Task 3: Add Styling And Landing Entry Buttons

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write failing checks**

Run:

```bash
rg -n "wdih-demo-login|wdih-demo-launch|Демо-режим" index.html
```

Expected: no matches for these UI classes/text before this task.

- [ ] **Step 2: Implement `injectStyles()`**

Replace the empty `injectStyles()` in the inserted module with:

```js
  function injectStyles() {
    if (document.getElementById(STYLE_ID)) return;
    const style = document.createElement("style");
    style.id = STYLE_ID;
    style.textContent = `
      .wdih-demo-overlay {
        position: fixed;
        inset: 0;
        z-index: 10000;
        background: oklch(98.5% 0.005 230);
        color: oklch(23% 0.02 250);
        font-family: 'Manrope', system-ui, sans-serif;
        overflow: auto;
      }
      .wdih-demo-shell {
        min-height: 100vh;
        display: grid;
        grid-template-rows: 72px 1fr;
      }
      .wdih-demo-topbar {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 18px;
        padding: 0 28px;
        border-bottom: 1px solid oklch(89% 0.008 235);
        background: oklch(98.5% 0.005 230 / 0.94);
        backdrop-filter: blur(14px);
        position: sticky;
        top: 0;
        z-index: 2;
      }
      .wdih-demo-brand {
        font-family: 'Newsreader', serif;
        font-size: 23px;
        font-weight: 500;
        letter-spacing: -0.01em;
      }
      .wdih-demo-badge {
        border-radius: 999px;
        padding: 6px 10px;
        background: oklch(94% 0.04 245);
        color: oklch(42% 0.1 245);
        font-size: 12px;
        font-weight: 800;
      }
      .wdih-demo-button {
        border: 0;
        border-radius: 999px;
        padding: 11px 18px;
        background: oklch(55% 0.13 245);
        color: #fff;
        font: 800 13px/1 'Manrope', system-ui, sans-serif;
        cursor: pointer;
        box-shadow: 0 12px 26px oklch(55% 0.13 245 / 0.18);
      }
      .wdih-demo-button.secondary {
        background: #fff;
        color: oklch(40% 0.1 245);
        border: 1px solid oklch(86% 0.012 235);
        box-shadow: none;
      }
      .wdih-demo-launch {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        text-decoration: none;
        white-space: nowrap;
      }
      .wdih-demo-login {
        min-height: 100vh;
        display: grid;
        place-items: center;
        padding: 28px;
        background:
          linear-gradient(135deg, oklch(98.5% 0.005 230), oklch(94% 0.04 245));
      }
      .wdih-demo-login-panel {
        width: min(520px, 100%);
        background: #fff;
        border: 1px solid oklch(89% 0.008 235);
        border-radius: 20px;
        padding: 34px;
        box-shadow: 0 24px 70px oklch(23% 0.02 250 / 0.12);
      }
      .wdih-demo-layout {
        display: grid;
        grid-template-columns: 360px minmax(0, 1fr);
        min-height: calc(100vh - 72px);
      }
      .wdih-demo-sidebar {
        border-right: 1px solid oklch(89% 0.008 235);
        background: oklch(96.5% 0.006 235);
        padding: 22px;
      }
      .wdih-demo-main {
        padding: 26px;
        overflow: auto;
      }
      .wdih-demo-card {
        background: #fff;
        border: 1px solid oklch(89% 0.008 235);
        border-radius: 18px;
        padding: 22px;
        box-shadow: 0 12px 36px oklch(23% 0.02 250 / 0.06);
      }
      .wdih-demo-muted { color: oklch(52% 0.015 245); }
      .wdih-demo-grid { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 14px; }
      .wdih-demo-stack { display: flex; flex-direction: column; gap: 14px; }
      .wdih-demo-case-row {
        width: 100%;
        text-align: left;
        border: 1px solid oklch(89% 0.008 235);
        border-radius: 14px;
        background: #fff;
        padding: 14px;
        cursor: pointer;
      }
      .wdih-demo-case-row.active {
        border-color: oklch(55% 0.13 245);
        box-shadow: 0 0 0 3px oklch(55% 0.13 245 / 0.10);
      }
      .wdih-demo-option {
        border: 1px solid oklch(89% 0.008 235);
        border-radius: 14px;
        background: oklch(98% 0.004 235);
        padding: 13px 14px;
        cursor: pointer;
        text-align: left;
        font: 700 14px/1.35 'Manrope', system-ui, sans-serif;
      }
      .wdih-demo-option.active {
        background: oklch(94% 0.04 245);
        border-color: oklch(55% 0.13 245);
        color: oklch(38% 0.1 245);
      }
      .wdih-demo-input {
        width: 100%;
        min-height: 104px;
        border: 1px solid oklch(86% 0.012 235);
        border-radius: 14px;
        padding: 13px 14px;
        font: 500 14px/1.5 'Manrope', system-ui, sans-serif;
        resize: vertical;
      }
      .wdih-demo-pill {
        display: inline-flex;
        align-items: center;
        gap: 6px;
        border-radius: 999px;
        padding: 5px 9px;
        font-size: 12px;
        font-weight: 800;
        background: oklch(94% 0.04 245);
        color: oklch(42% 0.1 245);
      }
      .wdih-demo-warning {
        background: oklch(95% 0.05 60);
        border: 1px solid oklch(82% 0.1 60);
        color: oklch(42% 0.12 45);
      }
      @media (max-width: 860px) {
        .wdih-demo-layout { grid-template-columns: 1fr; }
        .wdih-demo-sidebar { border-right: 0; border-bottom: 1px solid oklch(89% 0.008 235); }
        .wdih-demo-grid { grid-template-columns: 1fr; }
      }
    `;
    document.head.appendChild(style);
  }
```

- [ ] **Step 3: Implement `injectLandingButtons()`**

Replace the empty `injectLandingButtons()` with:

```js
  function injectLandingButtons() {
    if (document.querySelector("[data-wdih-demo-launch]")) return;

    const navInner = document.querySelector("nav > div");
    if (navInner) {
      const navButton = document.createElement("button");
      navButton.type = "button";
      navButton.dataset.wdihDemoLaunch = "nav";
      navButton.className = "wdih-demo-button wdih-demo-launch";
      navButton.textContent = "Попробовать демо";
      navButton.addEventListener("click", openDemo);
      navInner.appendChild(navButton);
    }

    const hero = document.querySelector("#top div");
    if (hero) {
      const heroButton = document.createElement("button");
      heroButton.type = "button";
      heroButton.dataset.wdihDemoLaunch = "hero";
      heroButton.className = "wdih-demo-button wdih-demo-launch";
      heroButton.style.marginTop = "22px";
      heroButton.textContent = "Попробовать демо";
      heroButton.addEventListener("click", openDemo);
      hero.appendChild(heroButton);
    }
  }
```

- [ ] **Step 4: Verify markers**

Run:

```bash
rg -n "wdih-demo-login|wdih-demo-launch|Попробовать демо|Демо-режим" index.html
python3 - <<'PY'
from pathlib import Path
import json, re
text = Path("index.html").read_text()
m = re.search(r'<script type="__bundler/template">\s*(.*?)\s*</script>', text, re.S)
template = json.loads(m.group(1))
for needle in ["wdih-demo-login", "wdih-demo-launch", "Попробовать демо"]:
    assert needle in template, needle
print("styles and launchers present")
PY
```

Expected:

```text
styles and launchers present
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add demo preview launch styling"
```

---

### Task 4: Add Synthetic Demo Data

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write failing data checks**

Run:

```bash
rg -n "case-throat|case-abdomen|case-headache|Елена Морозова|Илья Романов|ЛабСити|МедЛаб" index.html
```

Expected: no matches before this task.

- [ ] **Step 2: Replace `const DEMO_CASES = [];` with full synthetic cases**

Use this data block:

```js
  const DEMO_CASES = [
    {
      id: "case-throat",
      title: "Першение в горле и слабость",
      summary: "Легкие симптомы без красных флагов",
      date: "12 июня",
      status: "Завершено",
      statusType: "done",
      urgency: "Низкая срочность",
      urgencyType: "low",
      nextStep: "Симптомы прошли после наблюдения",
      route: [
        "48 часов наблюдения за температурой и самочувствием",
        "Питьевой режим, отдых, без самостоятельного приема антибиотиков",
        "Повторный мини-опрос через два дня"
      ],
      rationale: "По ответам не было красных флагов: высокой температуры, одышки, сильной боли или быстрого ухудшения.",
      followUp: "Через два дня Аня отметила, что температура не поднималась, першение прошло, слабость ушла.",
      outcome: "Обращение закрыто, визит не потребовался.",
      analyses: [],
      doctors: [],
      documents: []
    },
    {
      id: "case-abdomen",
      title: "Тянущая боль в животе после еды",
      summary: "Подготовка к гастроэнтерологу",
      date: "4 июня",
      status: "Завершено",
      statusType: "done",
      urgency: "Плановый визит",
      urgencyType: "medium",
      nextStep: "Прием и выписки суммаризованы",
      route: [
        "Начать с гастроэнтеролога в плановом порядке",
        "Подготовить результаты базовых анализов для обсуждения с врачом",
        "Ускорить обращение при резкой боли, температуре или крови"
      ],
      rationale: "Симптом связан с приемом пищи и повторяется несколько дней, но в демо-ответах нет признаков немедленной срочности.",
      followUp: "Пользователь загрузил демо-выписки из DocMed и Medsi. Сервис выделил назначения, дату приема и список действий до визита.",
      outcome: "План сохранен: прием 7 июня в 18:30, анализы сдать до приема, взять результаты УЗИ и список лекарств.",
      analyses: [
        { name: "Общий анализ крови", why: "Обсудить признаки воспаления и анемии", recommended: "ЛабСити", price: "520 ₽", alternative: "МедЛаб, 640 ₽" },
        { name: "Биохимия крови", why: "Подготовить базовые показатели для врача", recommended: "МедЛаб", price: "1 450 ₽", alternative: "ЛабСити, 1 690 ₽" },
        { name: "УЗИ брюшной полости", why: "Врач может использовать как часть маршрута", recommended: "DocMed Демо", price: "2 300 ₽", alternative: "Medsi Демо, 2 700 ₽" },
        { name: "Копрограмма", why: "Может помочь врачу уточнить дальнейшие шаги", recommended: "ЛабСити", price: "710 ₽", alternative: "МедЛаб, 780 ₽" }
      ],
      doctors: [
        { name: "Елена Морозова", clinic: "DocMed Демо", specialty: "Гастроэнтеролог", slot: "7 июня, 18:30", reason: "Есть опыт с жалобами после еды и ближайший вечерний слот" },
        { name: "Илья Романов", clinic: "Medsi Демо", specialty: "Терапевт с фокусом на гастроэнтерологию", slot: "8 июня, 10:00", reason: "Подойдет, если хочется начать с общего маршрута" }
      ],
      documents: [
        { title: "Выписка DocMed Демо", summary: "Назначен плановый прием гастроэнтеролога, рекомендовано принести результаты ОАК, биохимии и УЗИ." },
        { title: "Памятка Medsi Демо", summary: "До приема вести короткий дневник питания и отметить, после каких продуктов возникает боль." }
      ]
    },
    {
      id: "case-headache",
      title: "Головные боли после работы за компьютером",
      summary: "Ожидается выбор специалиста",
      date: "Сегодня",
      status: "В прогрессе",
      statusType: "active",
      urgency: "Нужно уточнить контекст",
      urgencyType: "medium",
      nextStep: "Выбрать слот или продолжить опрос",
      route: [
        "Проверить красные флаги: резкая необычная боль, слабость, нарушение речи или зрения",
        "Измерять давление утром и вечером два дня",
        "Рассмотреть невролога и офтальмолога, если симптомы повторяются"
      ],
      rationale: "Симптом появляется после нагрузки на зрение, но маршрут зависит от давления, зрения и характера боли.",
      followUp: "Пользователь еще не выбрал специалиста.",
      outcome: "Кейс открыт: сервис ждет выбор слота или дополнительные ответы.",
      analyses: [
        { name: "Контроль артериального давления", why: "Помогает врачу увидеть связь боли с давлением", recommended: "Домашние измерения", price: "0 ₽", alternative: "Аптека рядом, демо-измерение 120 ₽" },
        { name: "Консультация офтальмолога", why: "Актуально при нагрузке от экрана", recommended: "Medsi Демо", price: "2 100 ₽", alternative: "DocMed Демо, 2 400 ₽" }
      ],
      doctors: [
        { name: "Мария Лебедева", clinic: "Medsi Демо", specialty: "Невролог", slot: "Сегодня, 19:10", reason: "Ближайший вечерний слот по направлению головной боли" },
        { name: "Артем Савин", clinic: "DocMed Демо", specialty: "Офтальмолог", slot: "Завтра, 12:30", reason: "Подходит для проверки зрительного фактора" }
      ],
      documents: []
    }
  ];
```

- [ ] **Step 3: Add route support data**

Insert this block after `DEMO_CASES`:

```js
  const ROUTES_BY_AREA = {
    abdomen: {
      title: "Боль или дискомфорт в животе",
      specialist: "Гастроэнтеролог",
      firstStep: "Плановый визит к гастроэнтерологу",
      explanation: "По описанию полезно начать с врача, который занимается пищеварением, и подготовить базовые результаты для обсуждения.",
      analyses: DEMO_CASES.find(item => item.id === "case-abdomen").analyses,
      doctors: DEMO_CASES.find(item => item.id === "case-abdomen").doctors
    },
    head: {
      title: "Головная боль",
      specialist: "Невролог / офтальмолог",
      firstStep: "Уточнить красные флаги и выбрать профиль по контексту",
      explanation: "Если боль связана с экраном и нагрузкой, врачу пригодятся данные давления и проверка зрения.",
      analyses: DEMO_CASES.find(item => item.id === "case-headache").analyses,
      doctors: DEMO_CASES.find(item => item.id === "case-headache").doctors
    },
    throat: {
      title: "Горло и симптомы простуды",
      specialist: "Терапевт при ухудшении",
      firstStep: "Наблюдение 24-48 часов без красных флагов",
      explanation: "При легких симптомах без ухудшения маршрут может начаться с наблюдения, чтобы не делать лишний визит.",
      analyses: [],
      doctors: [
        { name: "Ольга Крылова", clinic: "DocMed Демо", specialty: "Терапевт", slot: "Завтра, 09:40", reason: "Подойдет, если температура поднимется или симптомы усилятся" }
      ]
    },
    back: {
      title: "Спина или суставы",
      specialist: "Невролог / ортопед",
      firstStep: "Планово выбрать специалиста по характеру боли",
      explanation: "Маршрут зависит от травмы, онемения, ограничения движения и длительности симптомов.",
      analyses: [
        { name: "Общий анализ крови", why: "Обсудить признаки воспаления", recommended: "ЛабСити", price: "520 ₽", alternative: "МедЛаб, 640 ₽" },
        { name: "Витамин D", why: "Иногда обсуждается при мышечно-суставных жалобах", recommended: "МедЛаб", price: "1 090 ₽", alternative: "ЛабСити, 1 240 ₽" }
      ],
      doctors: [
        { name: "Денис Орлов", clinic: "Medsi Демо", specialty: "Ортопед", slot: "Пятница, 17:20", reason: "Подходит при боли после нагрузки или травмы" },
        { name: "Наталья Соколова", clinic: "DocMed Демо", specialty: "Невролог", slot: "Суббота, 11:15", reason: "Подходит при онемении или отдающей боли" }
      ]
    },
    skin: {
      title: "Кожа",
      specialist: "Дерматолог",
      firstStep: "Плановый прием дерматолога",
      explanation: "Для кожных симптомов полезно показать врачу динамику и не маскировать проявления перед приемом.",
      analyses: [
        { name: "Общий анализ крови", why: "Может понадобиться врачу для общей картины", recommended: "ЛабСити", price: "520 ₽", alternative: "МедЛаб, 640 ₽" },
        { name: "Соскоб по назначению врача", why: "Лучше согласовать после осмотра", recommended: "DocMed Демо", price: "900 ₽", alternative: "Medsi Демо, 1 100 ₽" }
      ],
      doctors: [
        { name: "Софья Белова", clinic: "DocMed Демо", specialty: "Дерматолог", slot: "Четверг, 16:45", reason: "Есть ближайший слот по кожным жалобам" }
      ]
    },
    other: {
      title: "Неочевидный симптом",
      specialist: "Терапевт",
      firstStep: "Начать с терапевта и уточнить направление",
      explanation: "Если симптом сложно отнести к одной области, терапевт помогает собрать маршрут без лишних переключений.",
      analyses: [
        { name: "Общий анализ крови", why: "Базовый контекст для первичного визита", recommended: "ЛабСити", price: "520 ₽", alternative: "МедЛаб, 640 ₽" }
      ],
      doctors: [
        { name: "Анна Ковалева", clinic: "Medsi Демо", specialty: "Терапевт", slot: "Завтра, 18:00", reason: "Вечерний слот для первичной маршрутизации" }
      ]
    }
  };
```

- [ ] **Step 4: Verify data is present and parseable**

Run:

```bash
rg -n "case-throat|case-abdomen|case-headache|Елена Морозова|Илья Романов|ЛабСити|МедЛаб|ROUTES_BY_AREA" index.html
python3 - <<'PY'
from pathlib import Path
import json, re
text = Path("index.html").read_text()
m = re.search(r'<script type="__bundler/template">\s*(.*?)\s*</script>', text, re.S)
template = json.loads(m.group(1))
for needle in ["case-throat", "case-abdomen", "case-headache", "Елена Морозова", "ROUTES_BY_AREA"]:
    assert needle in template, needle
print("synthetic data present")
PY
```

Expected:

```text
synthetic data present
```

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "Add synthetic demo preview data"
```

---

### Task 5: Render Login And App Shell

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write failing UI checks**

Run:

```bash
rg -n "Войти через Яндекс|Авторизация не выполняется|Аня, 24|Новое обращение" index.html
```

Expected: no matches before this task, except "Новое обращение" if another worker has already added it.

- [ ] **Step 2: Add helpers and login renderer**

Insert these functions before `function render()`:

```js
  function escapeHtml(value) {
    return String(value ?? "")
      .replaceAll("&", "&amp;")
      .replaceAll("<", "&lt;")
      .replaceAll(">", "&gt;")
      .replaceAll('"', "&quot;");
  }

  function statusClass(caseItem) {
    return caseItem.urgencyType === "high" ? "wdih-demo-pill wdih-demo-warning" : "wdih-demo-pill";
  }

  function renderLogin() {
    return `
      <div class="wdih-demo-overlay wdih-demo-login">
        <div class="wdih-demo-login-panel">
          <div class="wdih-demo-brand">Где болит<span style="color:oklch(55% 0.13 245);">?</span></div>
          <div style="margin-top:18px;" class="wdih-demo-badge">Демо-режим</div>
          <h1 style="font-size:34px;line-height:1.08;letter-spacing:-0.03em;margin:18px 0 12px;">Превью медицинского навигатора</h1>
          <p class="wdih-demo-muted" style="font-size:16px;margin:0 0 20px;">Все данные в этом превью искусственные: обращения, врачи, клиники, цены, слоты и документы.</p>
          <button class="wdih-demo-button" data-demo-action="login" type="button" style="width:100%;height:48px;">Войти через Яндекс</button>
          <button class="wdih-demo-button secondary" data-demo-action="close" type="button" style="width:100%;height:44px;margin-top:10px;">Вернуться на лендинг</button>
          <p class="wdih-demo-muted" style="font-size:12px;margin:16px 0 0;">Авторизация не выполняется, это демонстрационный режим.</p>
        </div>
      </div>
    `;
  }
```

- [ ] **Step 3: Add shell renderer**

Replace `function renderApp() { return ""; }` with:

```js
  function renderApp() {
    return `
      <div class="wdih-demo-overlay">
        <div class="wdih-demo-shell">
          <header class="wdih-demo-topbar">
            <div style="display:flex;align-items:center;gap:14px;">
              <div class="wdih-demo-brand">Где болит<span style="color:oklch(55% 0.13 245);">?</span></div>
              <span class="wdih-demo-badge">Демо-режим</span>
            </div>
            <div style="display:flex;align-items:center;gap:12px;">
              <span class="wdih-demo-muted" style="font-size:13px;">Аня, 24 · синтетический профиль</span>
              <button class="wdih-demo-button secondary" data-demo-action="close" type="button">Выйти из демо</button>
            </div>
          </header>
          <div class="wdih-demo-layout">
            <aside class="wdih-demo-sidebar">
              ${renderSidebar()}
            </aside>
            <main class="wdih-demo-main">
              ${state.view === "wizard" ? renderWizard() : state.view === "result" ? renderRouteResult() : renderSelectedCase()}
            </main>
          </div>
        </div>
      </div>
    `;
  }
```

- [ ] **Step 4: Add initial sidebar and main renderers**

Insert before `renderApp()`:

```js
  function filteredCases() {
    if (state.filter === "active") return state.cases.filter(item => item.statusType === "active");
    if (state.filter === "done") return state.cases.filter(item => item.statusType === "done");
    return state.cases;
  }

  function renderSidebar() {
    const filters = [
      ["all", "Все"],
      ["active", "В прогрессе"],
      ["done", "Завершены"]
    ];
    const cases = filteredCases();
    return `
      <div class="wdih-demo-stack">
        <button class="wdih-demo-button" data-demo-action="start-wizard" type="button">Новое обращение</button>
        <div style="display:flex;gap:8px;flex-wrap:wrap;">
          ${filters.map(([key, label]) => `
            <button class="wdih-demo-button secondary" data-demo-action="filter" data-filter="${key}" type="button" style="${state.filter === key ? "border-color:oklch(55% 0.13 245);" : ""}">${label}</button>
          `).join("")}
        </div>
        <div class="wdih-demo-stack">
          ${cases.length ? cases.map(item => `
            <button class="wdih-demo-case-row ${item.id === state.selectedCaseId ? "active" : ""}" data-demo-action="select-case" data-case-id="${escapeHtml(item.id)}" type="button">
              <div style="font-weight:800;">${escapeHtml(item.title)}</div>
              <div class="wdih-demo-muted" style="font-size:13px;margin-top:3px;">${escapeHtml(item.summary)}</div>
              <div style="display:flex;justify-content:space-between;gap:8px;margin-top:10px;">
                <span class="${statusClass(item)}">${escapeHtml(item.status)}</span>
                <span class="wdih-demo-muted" style="font-size:12px;">${escapeHtml(item.date)}</span>
              </div>
            </button>
          `).join("") : `<div class="wdih-demo-card wdih-demo-muted">В этом фильтре пока нет обращений.</div>`}
        </div>
      </div>
    `;
  }

  function renderSelectedCase() {
    const item = state.cases.find(caseItem => caseItem.id === state.selectedCaseId) || state.cases[0];
    if (!item) return `<div class="wdih-demo-card">Выберите обращение.</div>`;
    return `<div class="wdih-demo-card"><h2 style="margin:0;">${escapeHtml(item.title)}</h2><p>${escapeHtml(item.summary)}</p></div>`;
  }

  function renderWizard() {
    return `<div class="wdih-demo-card"><h2 style="margin:0;">Новое обращение</h2><p class="wdih-demo-muted">Опрос появится на следующем шаге реализации.</p></div>`;
  }

  function renderRouteResult() {
    return `<div class="wdih-demo-card"><h2 style="margin:0;">Маршрут</h2></div>`;
  }
```

- [ ] **Step 5: Add root event delegation**

Insert after `ensureRoot()`:

```js
  document.addEventListener("click", function (event) {
    const target = event.target.closest("[data-demo-action]");
    if (!target) return;
    const action = target.dataset.demoAction;
    if (action === "login") login();
    if (action === "close") closeDemo();
    if (action === "start-wizard") {
      state.view = "wizard";
      state.wizardStep = 0;
      state.wizardAnswers = {};
      state.generatedRoute = null;
      render();
    }
    if (action === "select-case") {
      state.selectedCaseId = target.dataset.caseId;
      state.view = "case";
      render();
    }
    if (action === "filter") {
      state.filter = target.dataset.filter;
      render();
    }
  });
```

- [ ] **Step 6: Verify login/shell text exists**

Run:

```bash
rg -n "Войти через Яндекс|Авторизация не выполняется|Аня, 24|Новое обращение|renderSidebar|renderSelectedCase" index.html
python3 - <<'PY'
from pathlib import Path
import json, re
text = Path("index.html").read_text()
m = re.search(r'<script type="__bundler/template">\s*(.*?)\s*</script>', text, re.S)
template = json.loads(m.group(1))
for needle in ["Войти через Яндекс", "Аня, 24", "Новое обращение", "renderSidebar"]:
    assert needle in template, needle
print("login and shell present")
PY
```

Expected:

```text
login and shell present
```

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "Render demo preview login and shell"
```

---

### Task 6: Render Full Case Cards

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write failing content checks**

Run:

```bash
rg -n "Рекомендованный маршрут|Анализы и подготовка|Демо-документы|Это не диагноз" index.html
```

Expected: no matches before this task, except "Это не диагноз" may already exist in FAQ.

- [ ] **Step 2: Replace `renderSelectedCase()`**

Use:

```js
  function renderList(title, items) {
    if (!items.length) return "";
    return `
      <section class="wdih-demo-card">
        <h3 style="margin:0 0 12px;">${escapeHtml(title)}</h3>
        <div class="wdih-demo-stack">
          ${items.map(item => `<div style="display:flex;gap:10px;"><span style="color:oklch(55% 0.13 245);font-weight:900;">•</span><span>${escapeHtml(item)}</span></div>`).join("")}
        </div>
      </section>
    `;
  }

  function renderAnalyses(analyses) {
    if (!analyses.length) return "";
    return `
      <section class="wdih-demo-card">
        <h3 style="margin:0 0 12px;">Анализы и подготовка</h3>
        <div class="wdih-demo-grid">
          ${analyses.map(item => `
            <div style="border:1px solid oklch(89% 0.008 235);border-radius:14px;padding:14px;">
              <div style="font-weight:800;">${escapeHtml(item.name)}</div>
              <div class="wdih-demo-muted" style="font-size:13px;margin-top:4px;">${escapeHtml(item.why)}</div>
              <div style="margin-top:10px;font-weight:800;color:oklch(42% 0.1 245);">${escapeHtml(item.recommended)} · ${escapeHtml(item.price)}</div>
              <div class="wdih-demo-muted" style="font-size:12px;">Альтернатива: ${escapeHtml(item.alternative)}</div>
            </div>
          `).join("")}
        </div>
      </section>
    `;
  }

  function renderDoctors(doctors) {
    if (!doctors.length) return "";
    return `
      <section class="wdih-demo-card">
        <h3 style="margin:0 0 12px;">Специалисты по направлению</h3>
        <div class="wdih-demo-grid">
          ${doctors.map(item => `
            <div style="border:1px solid oklch(89% 0.008 235);border-radius:14px;padding:14px;">
              <div style="font-weight:900;">${escapeHtml(item.name)}</div>
              <div class="wdih-demo-muted" style="font-size:13px;">${escapeHtml(item.specialty)} · ${escapeHtml(item.clinic)}</div>
              <div style="margin-top:10px;font-weight:800;">${escapeHtml(item.slot)}</div>
              <div class="wdih-demo-muted" style="font-size:13px;margin-top:4px;">${escapeHtml(item.reason)}</div>
            </div>
          `).join("")}
        </div>
      </section>
    `;
  }

  function renderDocuments(documents) {
    if (!documents.length) return "";
    return `
      <section class="wdih-demo-card">
        <h3 style="margin:0 0 12px;">Демо-документы и суммаризация</h3>
        <div class="wdih-demo-stack">
          ${documents.map(item => `
            <div style="border:1px solid oklch(89% 0.008 235);border-radius:14px;padding:14px;">
              <div style="font-weight:800;">${escapeHtml(item.title)}</div>
              <div class="wdih-demo-muted" style="font-size:13px;margin-top:4px;">${escapeHtml(item.summary)}</div>
            </div>
          `).join("")}
        </div>
      </section>
    `;
  }

  function renderSelectedCase() {
    const item = state.cases.find(caseItem => caseItem.id === state.selectedCaseId) || state.cases[0];
    if (!item) return `<div class="wdih-demo-card">Выберите обращение.</div>`;
    return `
      <div class="wdih-demo-stack">
        <section class="wdih-demo-card">
          <div style="display:flex;justify-content:space-between;gap:16px;align-items:flex-start;flex-wrap:wrap;">
            <div>
              <div class="wdih-demo-muted" style="font-size:13px;">${escapeHtml(item.date)} · ${escapeHtml(item.summary)}</div>
              <h2 style="font-size:30px;line-height:1.12;letter-spacing:-0.03em;margin:6px 0 10px;">${escapeHtml(item.title)}</h2>
              <div style="display:flex;gap:8px;flex-wrap:wrap;">
                <span class="${statusClass(item)}">${escapeHtml(item.status)}</span>
                <span class="${item.urgencyType === "high" ? "wdih-demo-pill wdih-demo-warning" : "wdih-demo-pill"}">${escapeHtml(item.urgency)}</span>
              </div>
            </div>
            <button class="wdih-demo-button secondary" data-demo-action="start-wizard" type="button">Новое обращение</button>
          </div>
        </section>
        ${renderList("Рекомендованный маршрут", item.route)}
        <section class="wdih-demo-card">
          <h3 style="margin:0 0 8px;">Почему такой шаг</h3>
          <p style="margin:0;" class="wdih-demo-muted">${escapeHtml(item.rationale)}</p>
        </section>
        ${renderAnalyses(item.analyses)}
        ${renderDoctors(item.doctors)}
        ${renderDocuments(item.documents)}
        <section class="wdih-demo-card">
          <h3 style="margin:0 0 8px;">Результат сопровождения</h3>
          <p style="margin:0 0 8px;" class="wdih-demo-muted">${escapeHtml(item.followUp)}</p>
          <strong>${escapeHtml(item.outcome)}</strong>
        </section>
        <section class="wdih-demo-card" style="background:oklch(96.5% 0.006 235);">
          <strong>Это не диагноз.</strong>
          <span class="wdih-demo-muted"> Сервис помогает подготовиться к обращению, а медицинские решения принимает врач.</span>
        </section>
      </div>
    `;
  }
```

- [ ] **Step 3: Verify full case markers**

Run:

```bash
rg -n "Рекомендованный маршрут|Анализы и подготовка|Демо-документы|Результат сопровождения|Это не диагноз" index.html
```

Expected: all terms are present.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "Render detailed synthetic case cards"
```

---

### Task 7: Implement Intake Wizard And Route Result

**Files:**
- Modify: `index.html`

- [ ] **Step 1: Write failing wizard checks**

Run:

```bash
rg -n "Где беспокоит|Опишите своими словами|Красные флаги|Сохранить маршрут|generatedRoute" index.html
```

Expected: no matches for the Russian wizard labels before this task.

- [ ] **Step 2: Add wizard definitions**

Insert after `ROUTES_BY_AREA`:

```js
  const WIZARD_STEPS = [
    {
      key: "area",
      title: "Где беспокоит?",
      help: "Выберите ближайшую область. Это не диагноз, а старт маршрута.",
      options: [
        ["abdomen", "Живот / пищеварение"],
        ["head", "Голова"],
        ["throat", "Горло / простуда"],
        ["back", "Спина / суставы"],
        ["skin", "Кожа"],
        ["other", "Другое / не понимаю"]
      ]
    },
    { key: "description", title: "Опишите своими словами", help: "Можно бытовым языком: где, как ощущается, что меняется.", textarea: true },
    { key: "timing", title: "Как давно это началось?", help: "Длительность влияет на срочность.", options: [["today", "Сегодня"], ["days", "2-3 дня"], ["week", "Больше недели"], ["months", "Периодически месяцами"]] },
    { key: "impact", title: "Насколько мешает?", help: "Оценим организационный маршрут и срочность.", options: [["mild", "Слабо"], ["noticeable", "Заметно"], ["work", "Мешает работе или учебе"], ["severe", "Сильно / трудно терпеть"]] },
    { key: "redFlags", title: "Есть красные флаги?", help: "Если есть опасные признаки, маршрут становится срочнее.", options: [["none", "Нет"], ["fever", "Температура"], ["sharp", "Резкая сильная боль"], ["blood", "Кровь"], ["worse", "Быстро ухудшается"]] },
    { key: "context", title: "Что еще важно?", help: "Выберите то, что может пригодиться врачу.", options: [["stress", "Стресс или недосып"], ["meds", "Недавно принимались лекарства"], ["food", "Новая еда или поездка"], ["known", "Есть хроническое состояние"], ["nothing", "Ничего из этого"]] },
    { key: "constraints", title: "Когда и как удобно?", help: "Демо подберет искусственные слоты и цены.", options: [["evening", "Вечером после работы"], ["morning", "Утром"], ["weekend", "В выходные"], ["cheap", "Где дешевле"], ["dms", "В рамках ДМС"]] }
  ];
```

- [ ] **Step 3: Replace `renderWizard()` and add route generation**

Use:

```js
  function currentWizardStep() {
    return WIZARD_STEPS[state.wizardStep];
  }

  function canContinueWizard() {
    const step = currentWizardStep();
    const value = state.wizardAnswers[step.key];
    return typeof value === "string" && value.trim().length > 0;
  }

  function generateRoute() {
    const area = state.wizardAnswers.area || "other";
    const base = ROUTES_BY_AREA[area] || ROUTES_BY_AREA.other;
    const red = state.wizardAnswers.redFlags;
    const highUrgency = red && red !== "none";
    return {
      title: base.title,
      urgency: highUrgency ? "Повышенная срочность" : "Плановый маршрут",
      urgencyType: highUrgency ? "high" : "medium",
      firstStep: highUrgency ? "Не откладывать обращение за медицинской помощью" : base.firstStep,
      specialist: base.specialist,
      explanation: highUrgency
        ? "В ответах отмечен красный флаг. Демо не оптимизирует запись в такой ситуации, а подсвечивает необходимость своевременной помощи."
        : base.explanation,
      analyses: highUrgency ? [] : base.analyses,
      doctors: base.doctors
    };
  }

  function renderWizard() {
    const step = currentWizardStep();
    const value = state.wizardAnswers[step.key] || "";
    return `
      <div class="wdih-demo-card">
        <div class="wdih-demo-muted" style="font-size:13px;">Шаг ${state.wizardStep + 1} из ${WIZARD_STEPS.length}</div>
        <h2 style="font-size:30px;line-height:1.12;letter-spacing:-0.03em;margin:6px 0 8px;">${escapeHtml(step.title)}</h2>
        <p class="wdih-demo-muted" style="margin:0 0 20px;">${escapeHtml(step.help)}</p>
        ${step.textarea ? `
          <textarea class="wdih-demo-input" data-demo-input="${escapeHtml(step.key)}" aria-label="Описание симптомов">${escapeHtml(value)}</textarea>
        ` : `
          <div class="wdih-demo-grid">
            ${step.options.map(([key, label]) => `
              <button class="wdih-demo-option ${value === key ? "active" : ""}" data-demo-action="wizard-pick" data-key="${escapeHtml(step.key)}" data-value="${escapeHtml(key)}" type="button">${escapeHtml(label)}</button>
            `).join("")}
          </div>
        `}
        <div style="display:flex;justify-content:space-between;gap:12px;margin-top:22px;">
          <button class="wdih-demo-button secondary" data-demo-action="wizard-back" type="button">${state.wizardStep === 0 ? "К истории" : "Назад"}</button>
          <button class="wdih-demo-button" data-demo-action="wizard-next" type="button" ${canContinueWizard() ? "" : "disabled"} style="${canContinueWizard() ? "" : "opacity:.45;cursor:not-allowed;"}">${state.wizardStep === WIZARD_STEPS.length - 1 ? "Показать маршрут" : "Далее"}</button>
        </div>
      </div>
    `;
  }

  function renderRouteResult() {
    const route = state.generatedRoute || generateRoute();
    return `
      <div class="wdih-demo-stack">
        <section class="wdih-demo-card">
          <div class="${route.urgencyType === "high" ? "wdih-demo-pill wdih-demo-warning" : "wdih-demo-pill"}">${escapeHtml(route.urgency)}</div>
          <h2 style="font-size:30px;line-height:1.12;letter-spacing:-0.03em;margin:12px 0 8px;">${escapeHtml(route.title)}</h2>
          <p class="wdih-demo-muted" style="margin:0;">${escapeHtml(route.explanation)}</p>
        </section>
        ${renderList("Следующий шаг", [route.firstStep, "Направление: " + route.specialist])}
        ${renderAnalyses(route.analyses)}
        ${renderDoctors(route.doctors)}
        <section class="wdih-demo-card" style="display:flex;justify-content:space-between;gap:12px;align-items:center;flex-wrap:wrap;">
          <div>
            <strong>Сохранить маршрут в историю?</strong>
            <div class="wdih-demo-muted" style="font-size:13px;">Он появится как открытое демо-обращение в прогрессе.</div>
          </div>
          <button class="wdih-demo-button" data-demo-action="save-route" type="button">Сохранить маршрут</button>
        </section>
        <section class="wdih-demo-card" style="background:oklch(96.5% 0.006 235);">
          <strong>Это не диагноз.</strong>
          <span class="wdih-demo-muted"> Сервис помогает подготовиться к обращению, а медицинские решения принимает врач.</span>
        </section>
      </div>
    `;
  }
```

- [ ] **Step 4: Extend event delegation**

Inside the existing document click handler, add these branches:

```js
    if (action === "wizard-pick") {
      state.wizardAnswers[target.dataset.key] = target.dataset.value;
      render();
    }
    if (action === "wizard-back") {
      if (state.wizardStep === 0) {
        state.view = "case";
      } else {
        state.wizardStep -= 1;
      }
      render();
    }
    if (action === "wizard-next") {
      if (!canContinueWizard()) return;
      if (state.wizardStep < WIZARD_STEPS.length - 1) {
        state.wizardStep += 1;
      } else {
        state.generatedRoute = generateRoute();
        state.view = "result";
      }
      render();
    }
    if (action === "save-route") {
      const route = state.generatedRoute || generateRoute();
      const newCase = {
        id: "case-new-" + Date.now(),
        title: route.title,
        summary: "Создано из демо-опроса",
        date: "Только что",
        status: "В прогрессе",
        statusType: "active",
        urgency: route.urgency,
        urgencyType: route.urgencyType,
        nextStep: route.firstStep,
        route: [route.firstStep, "Направление: " + route.specialist],
        rationale: route.explanation,
        followUp: "Пользователь сохранил маршрут и может вернуться к нему позже.",
        outcome: "Ожидается выбор слота или следующий шаг.",
        analyses: route.analyses,
        doctors: route.doctors,
        documents: []
      };
      state.cases = [newCase, ...state.cases];
      state.selectedCaseId = newCase.id;
      state.filter = "all";
      state.view = "case";
      render();
    }
```

Add this input handler after the click handler:

```js
  document.addEventListener("input", function (event) {
    const target = event.target.closest("[data-demo-input]");
    if (!target) return;
    state.wizardAnswers[target.dataset.demoInput] = target.value;
    const next = document.querySelector('[data-demo-action="wizard-next"]');
    if (next) {
      const enabled = target.value.trim().length > 0;
      next.disabled = !enabled;
      next.style.opacity = enabled ? "" : ".45";
      next.style.cursor = enabled ? "" : "not-allowed";
    }
  });
```

- [ ] **Step 5: Verify wizard markers**

Run:

```bash
rg -n "Где беспокоит|Опишите своими словами|Красные флаги|Сохранить маршрут|generateRoute|save-route" index.html
```

Expected: all terms are present.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "Implement demo intake wizard"
```

---

### Task 8: Manual Browser Verification And Polish

**Files:**
- Modify: `index.html` only if verification reveals layout or interaction issues.

- [ ] **Step 1: Start a simple local server**

Run:

```bash
python3 -m http.server 8000
```

Expected:

```text
Serving HTTP on :: port 8000
```

- [ ] **Step 2: Open the page**

Open:

```text
http://localhost:8000/index.html
```

Expected:

- Landing page renders.
- "Попробовать демо" appears in the nav and hero area.

- [ ] **Step 3: Verify demo login**

Actions:

1. Click "Попробовать демо".
2. Confirm the login screen opens.
3. Confirm it says all preview data is artificial.
4. Click "Войти через Яндекс".

Expected:

- No network auth occurs.
- App shell opens with "Аня, 24 · синтетический профиль".

- [ ] **Step 4: Verify predefined cases**

Actions:

1. Open "Першение в горле и слабость".
2. Open "Тянущая боль в животе после еды".
3. Open "Головные боли после работы за компьютером".

Expected:

- First case shows observation and symptoms resolved.
- Second case shows analyses, DocMed/Medsi demo doctors, synthetic documents, and summarized plan.
- Third case shows "В прогрессе" and next action.
- Each detailed case includes "Это не диагноз".

- [ ] **Step 5: Verify wizard**

Actions:

1. Click "Новое обращение".
2. Complete all seven steps.
3. Use at least one path with redFlags = "Нет".
4. Click "Показать маршрут".
5. Click "Сохранить маршрут".

Expected:

- Result includes urgency, next step, specialist, analyses/doctors where appropriate.
- Saved case appears at the top of history as "В прогрессе".

- [ ] **Step 6: Verify urgent path**

Actions:

1. Start another new case.
2. Choose a red flag such as "Резкая сильная боль".
3. Complete the wizard.

Expected:

- Result says "Повышенная срочность".
- Copy emphasizes timely medical help rather than cheap lab or scheduling optimization.

- [ ] **Step 7: Check text safety**

Run:

```bash
rg -n "у вас |лечение|точно|ИИ-врач|пропустить врача" index.html
```

Expected:

- No newly added unsafe medical copy.
- Existing FAQ/design text can contain "диагноз" only in boundary statements such as "Это не диагноз" or "Диагноз ставит врач".

- [ ] **Step 8: Check responsive layout**

Test browser widths:

- 1440 px desktop
- 1024 px narrower desktop
- 768 px tablet fallback

Expected:

- No incoherent overlaps.
- Sidebar stacks above content below 860 px.
- Buttons and long labels fit within containers.

- [ ] **Step 9: Commit polish fixes**

If any fixes were needed:

```bash
git add index.html
git commit -m "Polish demo preview interactions"
```

If no fixes were needed, do not create an empty commit.

---

## Self-Review Checklist

- Spec coverage:
  - Landing entry: Task 3.
  - Fake Yandex login: Task 5.
  - Fullscreen desktop app: Tasks 3 and 5.
  - Three synthetic cases: Task 4.
  - Detailed case cards: Task 6.
  - New intake wizard: Task 7.
  - Route result and save to history: Task 7.
  - Synthetic-data transparency: Tasks 4, 5, 6, 8.
  - No persistence/backend/auth: Tasks 2, 5, 8.
  - Verification: Task 8.
- Red-flag scan: no incomplete sections, future-work notes, deferred-work language, or unspecified validation steps.
- Type consistency:
  - Case IDs use `case-throat`, `case-abdomen`, `case-headache`.
  - `state.view` values are `login`, `case`, `wizard`, `result`.
  - `statusType` values are `done` and `active`.
  - `urgencyType` values are `low`, `medium`, and `high`.
