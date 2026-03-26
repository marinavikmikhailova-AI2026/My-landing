CLAUDE.md
This file helps Claude (and other AI assistants) work effectively on this project.
Project Overview
JD Generator — a single-file web app for HR teams that turns 5 form fields into
ready-to-use job description texts for three platforms simultaneously: HH.ru, Telegram,
and an internal company portal.
File: `My_SPA_first_project_MM.htm`  
Stack: Pure HTML + CSS + vanilla JS, no dependencies, no build step, no server.  
Language: Russian UI throughout.
---
Architecture
The entire application is one self-contained `.htm` file.
Do not split it into separate files unless explicitly asked.
```
My_SPA_first_project_MM.htm
├── <style>        CSS with CSS custom properties (design tokens)
├── <body>         Header + form card + result container + footer
└── <script>       All JS: generation logic, validation, tabs, clipboard, reset
```
User flow
```
Fill 5 fields → click "Сгенерировать JD →"
       ↓
validate() checks required fields (role, grade, tasks, requirements)
       ↓
1200ms simulated progress bar (setTimeout)
       ↓
generateHH() + generateTelegram() + generatePortal() run in parallel
       ↓
renderResult() injects result card with 3 tabs into #result-container
       ↓
User switches tabs → switchTab() swaps text with fade
User copies → copyText() writes to clipboard
User resets → resetForm() clears everything
```
JS functions
Function	Purpose
`parseItems(text)`	Splits text by `\n` or `,` into a clean array of strings
`formatBullets(text, prefix)`	Converts multi-line or comma-separated text into a bullet list
`generateHH(role, grade, team, tasks, requirements)`	Returns HH.ru-formatted JD string
`generateTelegram(...)`	Returns Telegram post with emoji structure
`generatePortal(...)`	Returns formal internal portal format
`validate()`	Adds/removes `.error` class on required fields; returns boolean
`handleGenerate()`	Orchestrates validation → progress → generation → render
`renderResult(versions)`	Injects result card HTML into `#result-container`
`switchTab(tab)`	Fades out/in result text, updates active tab class
`copyText()`	`navigator.clipboard` with `execCommand` fallback
`showCopied(btn)`	Temporarily changes button label, reverts after 2s
`resetForm()`	Clears all fields, removes errors, destroys result card
Global state (two variables)
```js
let currentTab = 'hh';          // active tab key: 'hh' | 'tg' | 'portal'
let currentVersions = {};       // { hh: string, tg: string, portal: string }
```
---
Design System
All colours and tokens are defined as CSS custom properties on `:root`.
Never hardcode colour values — always reference a variable.
```css
--mint: #3ECFB2        /* primary accent, focus rings, copy button */
--sky: #6EB5FF         /* Telegram tab accent */
--lavender: #A78BFA    /* Portal tab accent */
--orange: #FF8C42      /* generate button, footer dot, error states */
--text: #1A1A2E        /* body text */
--secondary: #6B7280   /* labels, placeholders, secondary text */
--bg: #F7FAFA          /* page background, result text background */
--white: #FFFFFF       /* cards, inputs */
--border: #E5E7EB      /* input borders, header border */
--mint-light: #EDFAF6  /* HH tab active bg, copy button hover */
--sky-light: #EBF4FF   /* Telegram tab active bg */
--lavender-light: #F3EFFE /* Portal tab active bg */
--error: #FF8C42       /* validation error (same as --orange) */
```
Font: `Plus Jakarta Sans` (Google Fonts), weights 400/500/600/700/800.
Component patterns
Cards — `background: white`, `border-radius: 16px`, `box-shadow: 0 4px 24px rgba(0,0,0,0.07)`, `padding: 28px`
Inputs/selects/textareas — `border: 1.5px solid var(--border)`, focus state adds `var(--mint)` border + `rgba(62,207,178,0.12)` ring
Validation error — adds `.error` class to `.field` wrapper; child `.field-error` becomes visible; input border turns `var(--error)`
Tab active classes — `active-hh`, `active-tg`, `active-portal` (each has its own colour pair)
Result card entry — `fadeSlideUp` keyframe animation (opacity 0→1, translateY 20px→0, 0.4s)
---
JD Template Structure
Each generator function produces a plain text string (no HTML, no markdown).
`white-space: pre-wrap` on `.result-text` handles line breaks.
HH.ru template
```
{role} | {grade} | Финтех

О роли:
<intro paragraph>

Чем предстоит заниматься:
• item
• item

Что мы ожидаем:
• item

Что мы предлагаем:
• 4 fixed benefit lines

Откликайся — и давай обсудим детали.
```
Telegram template
```
🚀 Открыта вакансия: {role} ({grade})
🏢 Команда: {team}
📌 Задачи: • ...
✅ Ожидаем: • ...
💼 company blurb
👉 CTA
```
Internal portal template
```
ОТКРЫТА ВНУТРЕННЯЯ ВАКАНСИЯ
Должность / Уровень / Подразделение / Статус

ОПИСАНИЕ РОЛИ: ...
ОСНОВНЫЕ ФУНКЦИИ: — item
ТРЕБОВАНИЯ: — item
ДЛЯ ОТКЛИКА: HR contact
```
Bullet prefixes by format: HH → `•`, Telegram → `•`, Portal → `—`
---
Form Fields
ID	Label	Required	Element
`role`	Название роли	✅	`<input type="text">`
`grade`	Грейд	✅	`<select>` — Intern / Junior / Middle / Senior / Lead / Head / Director
`team`	Команда или отдел	❌	`<input type="text">` — fallback: "нашей команде" / "наша команда" / "не указано"
`tasks`	Ключевые задачи	✅	`<textarea>`
`requirements`	Требования к кандидату	✅	`<textarea>`
---
Working with This Project
The golden rule
Every change must be delivered as a complete, ready-to-use HTML file.
Never output diffs, snippets, or partial edits — always the full file from
`<!DOCTYPE html>` to `</html>`.
How to make changes
User describes what they want in plain language
Claude reads the current file in full
Claude outputs the entire updated file
User saves it, replacing the old version
When adding a new output format
Add a generator function `generateXxx(role, grade, team, tasks, requirements)`
Add the key to the `versions` object in `handleGenerate()`
Add a `<button class="tab" id="tab-xxx" onclick="switchTab('xxx')">` in `renderResult()`
Add an `active-xxx` CSS class with its own colour pair from the design system
Add the new tab key to the `tabClasses` map in `switchTab()`
When adding a new form field
Add a `.field` div with matching `id="field-{name}"` wrapper (for validation)
Add the input/select/textarea with `id="{name}"`
If required: add to the `required` array in `validate()`
Pass the new value into generator functions
Section comments to maintain
Keep these comments as landmarks inside `<script>` when the file grows:
```js
// --- HELPERS ---
// --- GENERATORS ---
// --- VALIDATION ---
// --- UI / RENDER ---
// --- CLIPBOARD ---
// --- RESET ---
```
---
Known Limitations & Improvement Ideas
Item	Notes
Hardcoded company copy	"финтех-компания с масштабом 1400+ человек" and benefit bullets are static strings inside generator functions. Parameterise them when the tool needs to serve multiple companies.
No AI involved	Generation is pure string templating — no API call. Adding Claude API would allow free-form, context-aware JD writing rather than fixed templates.
`grade` `<select>` resets incorrectly	`resetForm()` sets `grade.value = ''` which works only if `<option value="">` exists — it does, so this is fine. Preserve that empty-value option.
Clipboard fallback	`execCommand('copy')` is deprecated; the fallback exists for old iOS Safari. Keep it until Safari Clipboard API support is universal.
No persistence	Generated JDs are lost on refresh. If save/history is needed, use `localStorage` with a key like `jd_generator_history`.
Mobile layout	`max-width: 680px` centered layout works well on mobile. Do not widen beyond 720px without testing on 375px viewport.
---
What to Avoid
Do not introduce `npm`, Node.js, a bundler, or any external dependencies
Do not use a JS framework (React, Vue, etc.) — plain JS only
Do not split into multiple files
Do not change CSS variable names — they are referenced throughout
Do not add `console.log` statements to production output
Do not remove the `execCommand` clipboard fallback
Do not hardcode colours outside of `:root`
---
Don't — 5 правил с объяснением
1. Не трогай `<option value="">Выбери грейд</option>` в селекте
Что пойдёт не так: `resetForm()` сбрасывает поле через `grade.value = ''`.
Если убрать пустую опцию или изменить её `value`, после сброса селект застрянет
на первом реальном значении ("Intern") — визуально форма выглядит заполненной,
но пользователь этого не заметит и отправит незаполненный JD.
Пример ситуации: Просишь "добавь подсказку в дропдаун" — Claude меняет первую
опцию на `<option value="hint" disabled>Выбери грейд</option>`. После сброса
`grade.value = ''` больше не работает: селект показывает "Intern" вместо подсказки.
---
2. Не меняй ключи `'hh'`, `'tg'`, `'portal'` в объекте `currentVersions`
Что пойдёт не так: Эти строки используются в пяти местах одновременно:
`versions` в `handleGenerate()`, `switchTab()`, `tabClasses`, атрибуты `id` кнопок
(`tab-hh`, `tab-tg`, `tab-portal`) и `onclick`. Переименование ключа в одном месте
без синхронного обновления всех пяти сломает переключение вкладок — активная вкладка
перестанет подсвечиваться, а текст не будет меняться.
Пример ситуации: Просишь переименовать вкладку "Портал" в "HR-система".
Claude меняет `portal` → `hr` в `generatePortal()` и `versions`, но забывает
обновить `tabClasses` и `id="tab-portal"`. Вкладка кликается, но стиль не применяется
и текст остаётся от предыдущей вкладки.
---
3. Не добавляй HTML-теги или markdown внутрь строк генераторов
Что пойдёт не так: `.result-text` использует `white-space: pre-wrap` и
`textContent` (не `innerHTML`). HTML-теги внутри строки будут видны пользователю
дословно — `<b>Senior</b>` в тексте вместо жирного "Senior". Markdown-символы
(`**`, `##`) тоже не рендерятся и выглядят как мусор при копировании в HH или Telegram.
Пример ситуации: Просишь "выдели название роли жирным". Claude добавляет
`\`${role}``в`generateHH()`. В интерфейсе и в скопированном тексте пользователь видит `Product Manager` — именно так это и вставится в HH.ru.
---
4. Не удаляй двойной `requestAnimationFrame` в `handleGenerate()`
Что пойдёт не так: Прогресс-бар анимируется через CSS transition (`width 1.2s ease`).
Двойной `requestAnimationFrame` нужен, чтобы браузер успел применить `display: block`
к враперу до старта анимации. Без него браузер объединяет оба изменения в один
paint — прогресс-бар появляется уже заполненным, без анимации.
Пример ситуации: Просишь "почисти код, убери лишние обёртки". Claude
упрощает до одного `requestAnimationFrame` или убирает его совсем.
Прогресс-бар мгновенно перескакивает в 100% без плавного заполнения.
---
5. Не разноси логику валидации за пределы функции `validate()`
Что пойдёт не так: Сейчас `validate()` — единственное место, которое
управляет классом `.error` и вешает `{ once: true }` обработчики для
авто-снятия ошибки при вводе. Если добавить дополнительные проверки прямо
в `handleGenerate()` или в отдельный обработчик `oninput`, состояние ошибок
начнёт расходиться: одна проверка добавляет `.error`, другая — не снимает,
поле остаётся красным даже после исправления.
Пример ситуации: Просишь "покажи ошибку сразу при потере фокуса, не только
при клике". Claude добавляет `onblur` на каждый input с inline-логикой добавления
`.error`. Теперь два механизма конкурируют: `validate()` добавляет ошибку,
`{ once: true }` снимает её, `onblur` добавляет снова — поле мигает.
---
What Claude Should Know
1. `team` — единственное необязательное поле, у каждого генератора свой фоллбэк
Если `team` пуст, каждая функция использует своё значение по умолчанию:
`generateHH` → `"нашей команде"`, `generateTelegram` → `"наша команда"`,
`generatePortal` → `"не указано"`. Это разные строки — они согласованы с тоном
каждого формата. При добавлении нового генератора нужно явно определить свой фоллбэк,
не копируя чужой.
2. Результат рендерится через `innerHTML`, а не существующую разметку
`renderResult()` полностью пересоздаёт карточку результата через
`container.innerHTML = \`...``. Это значит: кнопки внутри результата (`.btn-copy`, `.btn-reset`, вкладки) не существуют в DOM до первой генерации. Нельзя навешивать на них обработчики в `<script>`на уровне инициализации — только через`onclick` прямо в шаблонной строке, как сейчас.
3. Анимация появления результата завязана на классе, а не на JS
Карточка результата появляется плавно благодаря CSS-анимации `fadeSlideUp`,
которая объявлена на классе `.result-card`. Она срабатывает автоматически при
добавлении элемента в DOM. Не нужно добавлять JS-анимацию поверх — это создаст
двойной эффект. Если нужно изменить появление, менять только `@keyframes fadeSlideUp`
и параметры анимации в `.result-card { animation: ... }`.
