# Перехід команди з Trello на GitHub Projects

> **Це загальна порада, а не документ конкретного проєкту.**
> Тут немає назв компаній, репозиторіїв чи реальних задач: усі приклади вигадані й показують підхід.
> Зараз ця порада написана для Android-команди, яка веде задачі в Trello, а код тримає в GitHub.
> Той самий підхід підходить і для iOS, вебу чи бекенду.

## Коротко

Команда переходить з Trello на GitHub-борду з тими самими статусами, полями й автоматизаціями, що й в інших команд.
Баги QA заводить у репозиторії тієї команди, чий це код, тому PR сам закриває свій баг.
Done означає «QA перевірив на білді», а не «змерджено».

## Чому варто переходити

Коли задачі живуть у Trello, а код у GitHub, одна й та сама робота легко розходиться по кількох місцях.
Типова картина:

| Де | Що там | Стан |
|---|---|---|
| Trello | Картка «Fix crash when opening profile» | Done |
| Репозиторій Android | PR із виправленням | Змерджено |
| Репозиторій Android | Issue про цей креш | Досі відкрита |
| Спільна QA-борда | Баг від QA про той самий креш | Статус Bug |

Розробник зробив роботу, але QA і тімліди цього не бачать.
У Trello немає зв'язку з PR, тому все доводиться синхронізувати вручну.

На GitHub це одна issue: PR із `Closes #N` сам переводить її по борді, а QA закриває її після перевірки на білді.

## Що змінюється

Замість 4 списків Trello буде 6 статусів, з яких нові лише Ready та Internal testing.

| Trello | GitHub-борда | Що означає | Хто рухає |
|---|---|---|---|
| To-do | Backlog | Зафіксовано, не заплановано | Автоматично при створенні issue |
| To-do | Ready | Є критерії приймання й дизайн, можна брати | Тімлід |
| Doing | In Progress | Хтось пише код, є виконавець | Розробник |
| On Review | In Review | PR відкрито, чекає рев'ю | Автоматично, коли PR прив'язаний |
| — | Internal testing | Змерджено і є в білді Google Play Internal testing | Автоматично після мерджу, versionCode у полі Build |
| Done | Done | QA перевірив на білді | QA |

Для iOS п'ятий статус називається In TestFlight, а для вебу й бекенду — In Staging.

**Поля замість кольорових лейблів:**

- `Priority`: Urgent · High · Medium · Low
- `Effort`: High · Medium · Low
- `Work type`: Bug · Feature · Chore · Refactor · Spike
- `Area`: область застосунку (Auth, Networking, Profile, Notifications…)
- `Blocked by`: посилання на задачу-блокер і короткий текст
- `Build`: versionCode білда, у який увійшла зміна
- `QA result`: Passed · Failed · Not needed

Blocked — це не статус.
Заблокована задача лишається у своїй колонці, а причину пишуть у `Blocked by`.

## Автоматизації GitHub Projects

Людина руками робить три речі: пише issue, відкриває PR із `Closes #N` і ставить `QA result`.
Решту роблять вбудовані автоматизації борди:

| Автоматизація | Що робить |
|---|---|
| Auto-add to project | Кожна нова issue з репозиторію потрапляє на борду |
| Auto-add sub-issues | Підзадачі додаються разом із батьківською |
| Item added to project | Ставить статус Backlog |
| Pull request linked to issue | Прив'язаний PR переводить задачу в In Review |
| Pull request merged | Мердж переводить задачу в п'ятий статус |
| Item closed | Закрита issue переходить у Done |
| Auto-close issue | Задача в Done закриває issue |

Без `Closes #N` у PR переходи не спрацюють, і задачу доведеться тягнути руками.

## Як вести задачі

### Розробник

1. **Задача з'являється сама.** Нова issue в репозиторії потрапляє на борду в Backlog.
2. **Береш задачу тільки з Ready.** Ставиш себе в Assignees і переносиш її в In Progress.
3. **Гілка:** `<тип>/<номер>-<коротко>`, наприклад `fix/42-profile-crash`.
4. **Коміт:** дієслово в наказовому способі, наприклад `Fix crash when opening profile on Android 9`.
5. **PR:** перший рядок опису — `Closes #42`, далі розділ «How to test» для QA. Задача сама перейде в In Review.
6. **Після мерджу** задача переходить в Internal testing, а той, хто завантажує білд, пише versionCode у `Build`.
7. **Якщо QA повернув задачу** (`QA result = Failed`), вона знову йде в In Progress до того самого розробника.

### QA

1. Баг Android-застосунку заводиш як issue в репозиторії Android за шаблоном Bug, без префіксів на кшталт `[AndroidTest]`.
2. Помилку API (500, неправильна відповідь, посилання з листа) заводиш у репозиторії бекенду.
3. Перед тим як створити баг, пошукай дублі; якщо такий уже є, додай коментар із versionCode.
4. У шаблоні вкажи versionCode, середовище (Dev / Prod), пристрій і версію Android, кроки, очікуваний і фактичний результат, скріншот, серйозність.
5. Перевіряєш на Internal testing: Passed → Done, Failed → коментар із versionCode і повернення в In Progress.

### Серйозність і пріоритет

Серйозність (наскільки все погано) ставить QA, а Priority (коли це робити) — тімлід.

| Серйозність | Коли | Приклад | Типовий Priority |
|---|---|---|---|
| Blocker | Креш, втрата даних, не вдається увійти чи заплатити | Застосунок вилітає після тапу на профіль | Urgent |
| Major | Функція не працює, але є обхідний шлях | Не можна змінити дату народження | High |
| Minor | Валідація, верстка, текст | У полі телефону немає плейсхолдера | Medium / Low |

### Чого не робити

- Не ставити Bug як статус: Bug — це `Work type`, а задача стоїть у своєму реальному статусі.
- Не створювати draft-картки прямо на борді: їх не видно в репозиторії, і PR їх не закриє.
- Не заводити баг однієї команди в репозиторії іншої, бо тоді PR його не закриє.
- Не переводити задачу в Done без перевірки на білді та без `QA result`.

## Шаблони в репозиторії

Шаблони форм лежать у `.github/ISSUE_TEMPLATE/`, а шаблон PR — у `.github/pull_request_template.md`.
Форми починають працювати на кнопці New issue, щойно потрапляють у гілку за замовчуванням.

<details>
<summary><code>.github/ISSUE_TEMPLATE/bug.yml</code></summary>

```yaml
name: Bug
description: Something in the Android app does not work as expected.
title: "[Bug] "
labels: ["bug"]
body:
  - type: markdown
    attributes:
      value: |
        Before you file, search open and closed issues for the same problem.
        If it already exists, add a comment with your versionCode instead of a new issue.
        Name the issue after what is broken and where, for example "Crash after tapping My profile", not "Profile bug".

  - type: input
    id: build
    attributes:
      label: Build
      description: App version and versionCode from Google Play Internal testing.
      placeholder: 1.4.0 (212)
    validations:
      required: true

  - type: dropdown
    id: environment
    attributes:
      label: Environment
      options:
        - Dev
        - Prod
    validations:
      required: true

  - type: input
    id: device
    attributes:
      label: Device and Android version
      placeholder: Pixel 7, Android 14
    validations:
      required: true

  - type: textarea
    id: steps
    attributes:
      label: Steps to reproduce
      placeholder: |
        1. Sign in as test account …
        2. Open …
        3. Tap …
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: Expected
    validations:
      required: true

  - type: textarea
    id: actual
    attributes:
      label: Actual
    validations:
      required: true

  - type: textarea
    id: attachments
    attributes:
      label: Screenshot or video
      description: Drag files here.
        Attach a crash log if the app crashed.

  - type: dropdown
    id: severity
    attributes:
      label: Severity
      options:
        - Blocker — crash, data loss, cannot sign in or pay
        - Major — a feature does not work, but there is a workaround
        - Minor — validation, layout or copy
    validations:
      required: true

  - type: input
    id: related
    attributes:
      label: Related feature or epic
      placeholder: "#42"
```

</details>

<details>
<summary><code>.github/ISSUE_TEMPLATE/feature.yml</code></summary>

```yaml
name: Feature
description: New behaviour or a change to existing behaviour in the Android app.
title: "[Feature] "
labels: ["enhancement"]
body:
  - type: textarea
    id: goal
    attributes:
      label: What and why
    validations:
      required: true

  - type: textarea
    id: acceptance
    attributes:
      label: Acceptance criteria
      description: The task leaves Backlog only when these are written.
        QA checks exactly these on the build.
      placeholder: |
        - [ ] …
    validations:
      required: true

  - type: input
    id: design
    attributes:
      label: Design
      description: Link to the design, or write "not needed".
    validations:
      required: true

  - type: textarea
    id: backend
    attributes:
      label: Backend dependency
      description: Endpoints this needs and the backend issue that delivers them, if any.

  - type: input
    id: epic
    attributes:
      label: Epic
```

</details>

<details>
<summary><code>.github/pull_request_template.md</code></summary>

```markdown
Closes #

## What changed

<!-- One or two sentences on what this pull request does and why. -->

## How to test

- **Screen:**
- **Steps:**
  1.
- **Test account:**
- **Environment:** Dev / Prod

## Checklist

- [ ] `Closes #N` above points at the issue, so the board moves it automatically
- [ ] Branch is named `<type>/<id>-<short>`
- [ ] Screenshots attached for UI changes
```

</details>

## План перенесення

Перенесення займає близько тижня й не зупиняє роботу.

- [ ] Домовитися з командою, що означають кольорові лейбли Trello, і розкласти їх по `Work type`, `Priority` або `Area`.
- [ ] Визначити, хто на борді розробник, тімлід і QA.
- [ ] Узгодити, що означав Done у Trello: «змерджено» чи «перевірено на білді».
- [ ] Перевірити, чи були в Trello Power-Ups, Custom Fields або правила автоматизації, щоб не втратити дані.
- [ ] Створити GitHub-борду з тими самими статусами, полями й автоматизаціями, що в інших команд.
- [ ] Додати в репозиторій шаблони Bug, Feature і PR.
- [ ] Перенести картки в issues: To-do → Backlog або Ready, Doing → In Progress, On Review → In Review, Done → Done або Internal testing.
- [ ] Перенести баги команди з чужих репозиторіїв через Transfer issue і закрити дублі.
- [ ] Перевести Trello-борду в режим тільки для читання.
- [ ] Провести 15-хвилинну зустріч і разом пройтися по новій борді.

## Як розповісти команді

Почніть із її власного болю, а не з теорії: покажіть одну задачу, яка зараз живе в трьох місцях.
Скажіть, що змін небагато: колонки майже ті самі, а ручної роботи стає менше.
Попросіть відповісти на кілька питань, щоб команда брала участь у рішенні, а не отримала його згори.
Дайте конкретний план і строк.

---

Можна вільно копіювати й адаптувати під свою команду.
