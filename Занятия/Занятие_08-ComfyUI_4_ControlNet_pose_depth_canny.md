---
tags:
  - занятие
  - comfyui
  - controlnet
---

# Занятие 8 — ComfyUI 4: ControlNet — pose / depth / canny + демо FaceID

## Короткий конспект

- **Цель занятия:** научиться управлять структурой кадра в ComfyUI через ControlNet: задавать позу персонажа, сохранять контур сцены и удерживать глубину пространства.
- **Базовая среда:** работаем через **cloud.comfy.org**; локальный ComfyUI допустим как запасной путь, но обязательный маршрут занятия строится вокруг облачного интерфейса.
- **Сегодня сдаём:** `workflows/w08_controlnet.json` + **3 selected output** в `outputs/selected/`: 2 позы персонажа и 1 сцена через Depth или Canny.
- **Что важно:** ControlNet не заменяет prompt и style bible. Он помогает удержать **структуру**, а смысл и стиль по-прежнему задаются prompt’ом и правилами проекта.
- **Главная задача:** сделать не финальную иллюстрацию, а управляемую основу для будущих скрингенов.
- **FaceID:** только короткое демо преподавателя, не обязательная практика занятия.
- **Правило курса:** не задокументировано — значит не сделано.

---

## Ограничения и рабочие допущения для cloud.comfy.org

- На занятии используем **проверенный workflow преподавателя** и одну совместимую модель.
- Не собираем сложные графы из случайных кастомных нод.
- Если в облаке недоступны нужные preprocessors, используем **готовые входные изображения**: pose, canny, depth.
- Если нужный ControlNet-пайплайн не запускается, преподаватель показывает запасной вариант на локальном ПК.
- LoRA, редкие модели, нестандартные узлы и FaceID не являются обязательной частью сдачи.
- Если ваш репозиторий уже использует `images/selected/` вместо `outputs/selected/`, не дублируйте папки. Держите один стандарт на весь проект.
- История запусков в облаке не считается системой хранения. Всё важное нужно сохранить в проект и закоммитить.

---

## Что сегодня должно появиться в проекте

1. **`workflows/w08_controlnet.json`**  
   Workflow, на котором сделаны результаты занятия.

2. **`outputs/selected/`** — 3 выбранных изображения:
   - `08-pose-a-hero-idle.png`
   - `08-pose-b-hero-action.png`
   - `08-scene-depth-or-canny.png`

3. **`logs/workflow-notes.md`**  
   Короткая запись: что контролировали, какой input использовали, что получилось, что сломалось.

4. **`characters/hero.md`**  
   Обновление: позы, эмоции, анимационные заметки, визуальные константы героя.

5. **`shots/shot-list.md`**  
   Уточнение будущих скрингенов: что происходит в кадре, какой режим ControlNet может помочь.

---

## Какой результат считается хорошим

- у вас есть **2 разные позы одного героя**, а не два случайных персонажа;
- у героя сохраняются основные визуальные признаки: силуэт, костюм, цвета, предметы;
- сцена через Depth или Canny читается как будущий игровой кадр;
- понятно, что именно контролировалось: поза, контур или глубина;
- workflow сохранён в `.json`;
- selected output не переполнен черновиками;
- в `workflow-notes.md` есть выводы, а не просто список файлов;
- результат связан с GDD, персонажем или будущими скрингенами.

---

## Главная идея занятия

ControlNet помогает управлять **структурой кадра**.

Обычный prompt отвечает на вопрос:

> Что должно быть изображено?

ControlNet помогает ответить на другой вопрос:

> Как это должно быть расположено?

Для игрового концепта это важно, потому что будущий скринген должен быть не просто красивой картинкой, а понятной сценой:

- кто в кадре;
- что делает персонаж;
- где находится цель;
- как устроено пространство;
- что должен понять игрок.

---

## Формула занятия

```text
Prompt = смысл и стиль

ControlNet = структура кадра

Style bible = правила визуального языка проекта

Workflow + logs = воспроизводимость
```

---

## Словарь терминов занятия

- **ControlNet** — способ управлять генерацией через дополнительное изображение-условие: позу, контур, карту глубины или скетч.
- **Control image** — входное изображение, которое задаёт структуру: pose map, canny map, depth map, sketch.
- **Pose / OpenPose** — режим, который передаёт модели позу персонажа через условный “скелет”.
- **Canny** — карта контуров; помогает сохранить линии, силуэты и форму объектов.
- **Depth** — карта глубины; помогает сохранить камеру, перспективу и расположение планов.
- **Preprocessor** — узел, который превращает обычную картинку в control image.
- **Weight / strength** — сила влияния ControlNet на генерацию.
- **Prompt constants** — постоянные элементы prompt’а, которые помогают удерживать героя, стиль и мир проекта.
- **Selected outputs** — отобранные изображения, которые идут в проект и на проверку.
- **Workflow** — сохранённый граф ComfyUI в `.json`.
- **Скринген** — синтетический скриншот геймплея, который объясняет игру.
- **FaceID** — подход для удержания похожести лица по референсу; на этом занятии только демо.

---

## Три режима ControlNet

### 1. Pose

Используйте, если нужно управлять **позой персонажа**.

Подходит для:

- idle pose;
- action pose;
- combat pose;
- dialogue pose;
- gesture pose;
- emotion pose.

Важно: Pose управляет положением тела, но не гарантирует сохранение лица, костюма и деталей. Поэтому обязательно повторяйте визуальные константы персонажа.

Пример задачи:

> Получить героя в спокойной позе и в позе действия, сохранив узнаваемый силуэт.

---

### 2. Canny

Используйте, если нужно сохранить **контур, силуэт или форму**.

Подходит для:

- скетча сцены;
- силуэта здания;
- формы арены;
- платформ;
- props;
- архитектуры;
- простого плана уровня.

Риск: грязный или слишком детализированный input может дать шумный результат.

Пример задачи:

> Взять простой контур арены и получить сцену в стиле проекта, сохранив форму уровня.

---

### 3. Depth

Используйте, если нужно сохранить **глубину и пространство**.

Подходит для:

- коридоров;
- арен;
- помещений;
- сцен с передним / средним / дальним планом;
- кадра с целью в глубине;
- будущих скрингенов.

Пример задачи:

> Получить игровую сцену, где ясно видны персонаж, путь и цель в глубине кадра.

---

## Как выбрать режим

| Задача | Режим | Почему |
|---|---|---|
| Герой стоит в понятной позе | Pose | Нужно управлять телом |
| Герой атакует, бежит, чинит или добывает ресурс | Pose | Важна динамика действия |
| Нужно сохранить форму здания | Canny | Важны контуры |
| Нужно повторить скетч арены | Canny | Важна геометрия |
| Нужно показать коридор или помещение | Depth | Важна перспектива |
| Нужно сделать основу скрингена | Depth | Важно игровое пространство |
| Нужно сохранить и позу, и сцену | Pose + Depth | Продвинутый режим, не обязателен сегодня |

---

## Control weight / strength

Сила контроля определяет, насколько сильно ControlNet влияет на результат.

```text
Низкий weight:
- больше свободы модели;
- стиль может сохраниться лучше;
- структура может уплыть.

Средний weight:
- баланс структуры и стиля;
- обычно лучший стартовый вариант.

Высокий weight:
- структура держится жёстче;
- стиль может ломаться;
- изображение может стать слишком “зажатым”.
```

Главное правило: не ищите “идеальное число”. Сравнивайте эффект и записывайте вывод.

---

## Что фиксируем и что меняем

### Фиксируем

- одну модель;
- один workflow;
- общий стиль проекта;
- prompt skeleton;
- визуальные константы героя;
- базовые параметры занятия.

### Меняем

- control input;
- режим ControlNet;
- действие персонажа;
- weight / strength;
- seed, если нужно получить вариацию.

---

## Как работать на занятии

### Шаг 1. Откройте нужные страницы проекта

Откройте:

- `GDD.md`
- `characters/hero.md`
- `shots/shot-list.md`
- `logs/workflow-notes.md`
- `cloud.comfy.org`

Проверьте:

- workflow открылся;
- нет красных узлов;
- модель выбрана;
- входные изображения готовы.

---

### Шаг 2. Запишите цель в `workflow-notes.md`

Перед генерацией заполните:

```md
## W08 — ControlNet

### Project goal
- Character:
- Scene / shot:
- What I want to control:
- Why this matters for my game:
```

Пример:

```md
## W08 — ControlNet

### Project goal
- Character: main scavenger engineer
- Scene / shot: lava bridge gameplay shot
- What I want to control: hero pose and depth of the scene
- Why this matters for my game: this shot should explain traversal and resource extraction
```

---

### Шаг 3. Подготовьте визуальные константы героя

Выпишите 5–7 признаков, которые должны сохраняться между позами:

```md
### Character constants
- Role:
- Silhouette:
- Costume:
- Main colors:
- Tool / weapon:
- Materials:
- What must not change:
```

Пример:

```md
### Character constants
- Role: engineer / scavenger
- Silhouette: compact body, short coat
- Costume: ash-gray coat, black gloves
- Main colors: gray, black, orange accent
- Tool / weapon: repair tool
- Materials: worn fabric, metal, soot
- What must not change: orange scarf and repair tool
```

---

### Шаг 4. Сделайте Pose A

Цель: спокойная поза героя.

Подходящие варианты:

- стоит;
- ждёт;
- говорит;
- держит инструмент;
- готов к действию.

Prompt modifier:

```text
standing idle pose, alert but calm, holding tool, readable full body pose
```

Проверка:

- поза читается;
- герой узнаваем;
- силуэт не сломан;
- стиль проекта сохранён.

---

### Шаг 5. Сделайте Pose B

Цель: поза в действии.

Подходящие варианты:

- атакует;
- уклоняется;
- бежит;
- чинит;
- добывает ресурс;
- использует предмет.

Prompt modifier:

```text
dynamic action pose, repairing machine, focused, strong line of action
```

или

```text
dynamic action pose, dodging falling debris, strong readable gesture
```

Проверка:

- действие понятно;
- это всё ещё тот же герой;
- не исчезли ключевые visual constants;
- поза связана с механикой игры.

---

### Шаг 6. Выберите режим для сцены

Выберите Canny или Depth.

Используйте Canny, если вам важны:

- контуры;
- силуэты;
- форма объекта;
- скетч уровня;
- архитектура;
- платформа или арена.

Используйте Depth, если вам важны:

- камера;
- перспектива;
- передний / средний / дальний план;
- пространство действия;
- путь игрока;
- цель в глубине.

В логе обязательно запишите, почему выбран этот режим.

---

### Шаг 7. Сделайте сцену через Canny или Depth

#### Вариант A — Canny scene

Используйте, если хотите сохранить форму или контур.

Prompt skeleton:

```text
gameplay concept screenshot, [game genre],
[location], [main action],
composition follows the provided contour sketch,
clear silhouettes, readable level layout,
[project style constants],
no text, no logo
```

Пример:

```text
gameplay concept screenshot, survival mining idle game,
obsidian extraction platform above a lava river,
small worker drones moving between cranes and conveyor belts,
composition follows the provided contour sketch,
clear silhouettes, readable level layout,
industrial fantasy architecture, hot orange lava light,
black obsidian, smoke and sparks,
no text, no logo
```

---

#### Вариант B — Depth scene

Используйте, если хотите сохранить глубину и пространство.

Prompt skeleton:

```text
gameplay concept screenshot, [game genre],
[location], [main action],
strong foreground midground background,
clear camera perspective, readable gameplay space,
main objective visible in the distance,
[project style constants],
no text, no logo
```

Пример:

```text
gameplay concept screenshot, dark exploration game,
abandoned underground station with glowing crystals,
the player character stands in the foreground,
a locked gate and enemy silhouettes are visible in the distance,
strong foreground midground background,
clear camera perspective, readable gameplay space,
cold blue light, wet stone, soft fog,
no text, no logo
```

---

### Шаг 8. Отберите selected outputs

В `outputs/selected/` должны попасть только 3 результата:

```text
08-pose-a-hero-idle.png
08-pose-b-hero-action.png
08-scene-depth-or-canny.png
```

Не складывайте туда всё подряд.

Selected — это то, что вы готовы объяснить:

- зачем сделано;
- что контролировалось;
- что получилось;
- что пойдёт в проект дальше.

---

### Шаг 9. Сохраните workflow

Сохраните workflow:

```text
workflows/w08_controlnet.json
```

Проверьте:

- файл лежит в репозитории;
- workflow открывается;
- нет красных узлов;
- понятно, какой input и какой ControlNet использовались.

---

## Шаблон записи в `logs/workflow-notes.md`

```md
## W08 — ControlNet

### Setup
- Tool: ComfyUI Cloud / local ComfyUI
- Model:
- Workflow:
- Control modes used:
- Date:

### Goal
- Character:
- Scene / shot:
- What I want to control:
- Why this matters for my game:

### Character constants
- Role:
- Silhouette:
- Costume:
- Main colors:
- Tool / weapon:
- Materials:
- What must not change:

### Inputs
- Pose A input:
- Pose B input:
- Scene input:
- Scene control mode: Canny / Depth
- Why this mode:

### Tests

| File | Control type | Input | Weight | What worked | What failed | Decision |
|---|---|---|---:|---|---|---|
| 08-pose-a-hero-idle.png | Pose |  |  |  |  | selected |
| 08-pose-b-hero-action.png | Pose |  |  |  |  | selected |
| 08-scene-depth-or-canny.png | Depth/Canny |  |  |  |  | selected |

### Conclusion
- What ControlNet helped with:
- What broke:
- What I will use for future screengens:
- What needs fixing later:
```

---

## Шаблон для `characters/hero.md`

Добавьте или обновите раздел:

```md
## Poses / Emotions / Animation Notes

### Visual constants
- Silhouette:
- Costume:
- Main colors:
- Tool / weapon:
- Materials:
- What must not change:

### Pose list

| Pose ID | Pose name | Game function | Emotion | Silhouette goal | Control input | Notes |
|---|---|---|---|---|---|---|
| P01 | Idle | waiting / dialogue | alert | readable full body | 08-pose-a-input.png |  |
| P02 | Action | attack / dodge / repair / cast | focused | dynamic gesture | 08-pose-b-input.png |  |

### Emotion notes
- Neutral:
- Alert:
- Fear:
- Determination:
- Victory / relief:

### Animation notes
- Idle movement:
- Main action:
- Anticipation:
- What should stay readable at small size:
```

---

## Шаблон для `shots/shot-list.md`

Добавьте или уточните 3 будущих скрингена.

```md
## SH01 — [shot name]

### Purpose
- What this shot proves:
- Gameplay mechanic shown:
- What the player should understand:

### Scene
- Location:
- Character / object:
- Action:
- Main conflict or objective:

### Camera
- View:
- Distance:
- Composition:
- Foreground:
- Midground:
- Background:

### Control plan
- Control mode: Pose / Canny / Depth
- Why this mode:
- Input needed:

### Must-have
- 
- 
- 

### Avoid
- 
- 
- 

### UI overlay idea
- 
```

---

## Prompt templates

### 1. Общий prompt skeleton

```text
[asset type], [game genre], [character/scene description],
[main action], [location], [mood],
clear readable composition, strong silhouette,
same visual style as project,
[style constants], [camera], [lighting], [materials],
game concept art, no text, no logo
```

---

### 2. Prompt для Pose A

```text
full body character concept, [hero description],
standing idle pose, alert but calm, ready for interaction,
same costume, same silhouette, same weapon or tool,
clear readable pose, neutral background,
[project style constants],
game character sheet style, no text, no logo
```

---

### 3. Prompt для Pose B

```text
full body character concept, [hero description],
dynamic action pose, [action],
same costume, same silhouette, same weapon or tool,
strong readable gesture, clear line of action,
[project style constants],
game character concept art, no text, no logo
```

---

### 4. Prompt для Canny scene

```text
gameplay concept screenshot, [game genre],
[location], [main action],
composition follows the provided contour sketch,
clear silhouettes, readable level layout,
strong shape design, controlled architecture,
[project style constants],
no text, no logo
```

---

### 5. Prompt для Depth scene

```text
gameplay concept screenshot, [game genre],
[location], [main action],
strong foreground midground background,
clear camera perspective, readable gameplay space,
main objective visible in the distance,
[project style constants],
no text, no logo
```

---

### 6. Базовый negative prompt

```text
text, logo, watermark, unreadable composition,
extra limbs, broken hands, distorted face,
inconsistent costume, random armor, duplicated character,
messy background, bad anatomy, low detail, blurry
```

---

### 7. Negative prompt для персонажа

```text
text, logo, watermark, extra limbs, broken hands,
distorted face, different costume, different character,
random weapon, inconsistent silhouette, bad anatomy
```

---

### 8. Negative prompt для сцены

```text
text, logo, watermark, unreadable level design,
messy background, random objects, broken perspective,
flat composition, no clear focal point, bad scale
```

---

## Типовые ошибки и быстрые исправления

### Ошибка 1. Менять всё сразу

**Проблема:** непонятно, что повлияло на результат.  
**Исправление:** зафиксировать модель, workflow, prompt skeleton и менять только input / mode / weight.

---

### Ошибка 2. Pose меняет персонажа

**Проблема:** поза читается, но герой стал другим.  
**Исправление:** усилить visual constants в prompt: силуэт, костюм, цвета, предмет, материалы.

---

### Ошибка 3. Canny даёт грязный результат

**Проблема:** входной контур слишком шумный.  
**Исправление:** упростить input, убрать мелкие линии, снизить weight.

---

### Ошибка 4. Depth не даёт глубины

**Проблема:** в исходном изображении нет понятных планов.  
**Исправление:** выбрать input с foreground / midground / background и ясной перспективой.

---

### Ошибка 5. Структура держится, но стиль ломается

**Проблема:** control weight слишком высокий.  
**Исправление:** снизить weight и усилить style constants.

---

### Ошибка 6. Стиль хороший, но структура уплыла

**Проблема:** control weight слишком низкий или input слабый.  
**Исправление:** поднять weight, очистить input, сделать prompt менее противоречивым.

---

### Ошибка 7. Нет связи с GDD

**Проблема:** результат красивый, но не помогает проекту.  
**Исправление:** вернуться к `GDD.md` и `shot-list.md`: какой кадр, механику или персонажа должен поддержать результат?

---

### Ошибка 8. Не сохранён workflow

**Проблема:** картинка есть, процесс потерян.  
**Исправление:** сразу после удачного запуска сохранить `.json` в `workflows/`.

---

## Мини-FAQ

### 1. Что важнее: красивая картинка или контроль?

На этом занятии важнее контроль. Картинка может быть черновой, но структура должна быть понятной.

---

### 2. Чем Canny отличается от Depth?

Canny держит линии и контуры.  
Depth держит пространство и глубину.

Коротко:

```text
Canny = форма и линии
Depth = пространство и камера
```

---

### 3. Почему персонаж в двух позах выглядит разным?

Потому что Pose контролирует только положение тела. Для сохранения героя нужны visual constants в prompt.

---

### 4. Можно ли использовать несколько ControlNet одновременно?

Можно, но сегодня это не обязательная часть. Сначала нужно уверенно понять один режим.

---

### 5. Можно ли использовать Flux?

Можно, если преподаватель дал совместимый workflow и он стабильно работает в cloud.comfy.org. Нельзя просто заменить SDXL на Flux в неподходящем графе.

---

### 6. Можно ли использовать LoRA?

Для обязательной сдачи — не нужно. Если используете, явно укажите это в логе.

---

### 7. Почему FaceID не входит в сдачу?

Потому что FaceID зависит от дополнительных моделей и узлов, а также требует аккуратности с реальными лицами. Сегодня это только демонстрация.

---

### 8. Что делать, если workflow не запускается в облаке?

Использовать запасной сценарий:

- teacher workflow;
- prepared input;
- локальное демо преподавателя;
- фиксация задачи и выводов без вычислений.

---

### 9. Можно ли положить в selected больше трёх файлов?

Для этого занятия — нет. В selected идут ровно 3 результата: 2 позы и 1 сцена.

---

### 10. Что делать, если результат плохой?

Записать, что именно не сработало, и сделать одну осмысленную правку:

- сменить input;
- изменить weight;
- уточнить prompt;
- выбрать другой режим ControlNet.

Не нужно хаотично менять всё сразу.

---

## Что делать, если…

### …workflow открылся с красными узлами

- не чинить граф вслепую;
- сообщить преподавателю;
- открыть запасной workflow;
- использовать готовые input-файлы.

---

### …нет OpenPose preprocessor

Используйте готовую pose-картинку от преподавателя.

---

### …нет Depth preprocessor

Используйте готовую depth map или выберите Canny.

---

### …Canny слишком шумный

- упростите исходный скетч;
- уберите мелкие линии;
- снизьте weight;
- используйте более чистый контур.

---

### …Depth выглядит плоско

- выберите input с явной перспективой;
- добавьте в prompt foreground / midground / background;
- уменьшите количество объектов;
- сделайте сцену проще.

---

### …cloud.comfy.org работает медленно

- не отправляйте много задач в очередь;
- не увеличивайте разрешение;
- не повышайте steps без необходимости;
- сохраняйте удачные результаты сразу;
- фиксируйте проблему в логе.

---

## Самопроверка перед коммитом

```text
[ ] Есть workflows/w08_controlnet.json
[ ] Есть 2 pose результата
[ ] Есть 1 depth/canny scene
[ ] Файлы лежат в selected
[ ] Selected не переполнен черновиками
[ ] В workflow-notes.md есть цель, input, режим, вывод
[ ] characters/hero.md обновлён
[ ] shots/shot-list.md обновлён
[ ] Имена файлов понятные
[ ] Я могу объяснить, зачем каждый результат нужен проекту
```

---

## Рекомендуемая структура файлов

```text
workflows/
  w08_controlnet.json

refs/
  controlnet/
    08-pose-a-input.png
    08-pose-b-input.png
    08-scene-canny-input.png
    08-scene-depth-input.png

outputs/
  selected/
    08-pose-a-hero-idle.png
    08-pose-b-hero-action.png
    08-scene-depth-or-canny.png

logs/
  workflow-notes.md

characters/
  hero.md

shots/
  shot-list.md
```

Если в проекте используется `images/selected/`, сохраняйте туда:

```text
images/
  selected/
    08-pose-a-hero-idle.png
    08-pose-b-hero-action.png
    08-scene-depth-or-canny.png
```

---

## Что сделать после занятия

1. Обновить `characters/hero.md`:
   - позы;
   - эмоции;
   - анимационные заметки;
   - визуальные константы;
   - что ломает узнаваемость героя.

2. Обновить `shots/shot-list.md`:
   - уточнить 3 будущих скрингена;
   - для каждого указать, что происходит в кадре;
   - выбрать подходящий режим ControlNet: Pose, Canny или Depth.

3. Подготовить 1–2 визуальных рефа для следующего занятия:
   - стиль;
   - композиция;
   - материал;
   - свет;
   - персонажный референс, если он не связан с реальным лицом без согласия.

---

## Главный вопрос после занятия

```text
Какие кадры моего проекта теперь можно собрать управляемо,
а не случайно?
```