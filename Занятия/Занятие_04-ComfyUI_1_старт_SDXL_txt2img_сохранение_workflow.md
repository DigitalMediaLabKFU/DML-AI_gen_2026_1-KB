---
tags:
  - занятие
презентация: https://docs.google.com/presentation/d/1rm8NI8W-T4oDItbuMTOPLOI0U5xQ2H5p-rdkZZjsCYs/edit?usp=sharing
---

## Короткий конспект

- **Цель занятия:** собрать первый воспроизводимый `txt2img` workflow в ComfyUI и получить 4 осмысленных варианта изображения под свой проект игры.
- **Базовая среда:** работаем в облачном ComfyUI; локальная установка — дополнительный, а не обязательный путь.
- **Сегодня сдаём:** `workflows/w04_sdxl_txt2img.json` + 4 selected output в `outputs/selected/`.
- **Что важно:** фиксируем `seed`, основные параметры и сам workflow; не меняем всё сразу.
- **Базовая модель занятия:** SDXL как простой и наглядный старт для первого workflow.
- **Правило курса:** не задокументировано — значит не сделано.

## Что сегодня должно появиться в проекте

1. **`workflows/w04_sdxl_txt2img.json`** — базовый рабочий граф `txt2img`.
2. **`outputs/selected/`** — 4 выбранных изображения, которые поддерживают концепт игры.
3. **`logs/prompt-plan.md`** — цель генерации, positive/negative prompt, критерии отбора.
4. **`logs/workflow-notes.md`** — checkpoint, seed, параметры, краткие выводы по результатам.
5. **`GDD.md`** — короткое обновление раздела про визуальную гипотезу / приоритетный ассет.

## Какой результат считается хорошим

- workflow открывается без сломанных узлов;
- у вас зафиксирован `seed`;
- преподаватель понимает, какой checkpoint и какие параметры использованы;
- 4 selected output не случайные, а связаны с концептом проекта;
- по заметкам видно, что вы понимаете, что именно пробовали и почему выбрали эти результаты.

## Словарь терминов занятия

- **ComfyUI** — node-based среда, где генерация собирается как граф из узлов.
- **Workflow** — сохранённый граф ComfyUI в `.json`.
- **Checkpoint** — файл модели, который задаёт базовые визуальные возможности генерации.
- **SDXL** — модель семейства Stable Diffusion XL; на этом занятии используется как базовый стартовый вариант.
- **txt2img** — генерация изображения по текстовому описанию.
- **Positive prompt** — что должно быть в изображении.
- **Negative prompt** — чего в изображении быть не должно.
- **Seed** — число, которое помогает повторить генерацию.
- **Steps** — количество шагов генерации.
- **CFG** — сила следования prompt’у.
- **Sampler / Scheduler** — способ, по которому идёт генерация.
- **Latent size** — базовый размер изображения до декодирования.
- **VAE Decode** — этап преобразования latent-представления в обычное изображение.
- **Selected outputs** — вручную отобранные результаты, которые идут в проект и на проверку.
- **Воспроизводимость** — возможность повторить результат по тем же параметрам и workflow.

## Мини-FAQ

1. **Зачем сохранять workflow, если картинка уже получилась?**
   Потому что в курсе проверяется не только результат, но и процесс. Картинка без workflow — слабый артефакт.

2. **Можно ли менять много параметров сразу?**
   Не стоит. На первом занятии важно менять 1–2 параметра, а остальное держать фиксированным.

3. **Нужен ли negative prompt?**
   Да. Даже в простом workflow он помогает убрать часть типичных ошибок и мусора.

4. **Можно ли использовать локальный ComfyUI вместо облака?**
   Можно как личную альтернативу, но результат всё равно должен быть оформлен по правилам курса: workflow, selected outputs, логи.

5. **Что важнее: “красивая картинка” или воспроизводимость?**
   На этом занятии — воспроизводимость. Красивая картинка без понятного процесса не решает учебную задачу.

6. **Сколько изображений нужно сдать?**
   В проверку идут 4 selected output. Всё остальное — черновые запуски.

7. **Что делать, если облако тормозит или не даёт быстро генерировать?**
   Не спамить очередь. Сначала оформить prompt-plan и workflow-notes, затем сделать минимально достаточный набор запусков.

8. **Что делать, если результат “не похож на проект”?**
   Вернуться к формулировке задачи, уточнить prompt и критерии отбора, а не хаотично менять все настройки подряд.

## Минимальный workflow этого занятия

Базовый граф состоит из следующих частей:

1. **Load Checkpoint**
2. **CLIP Text Encode (positive)**
3. **CLIP Text Encode (negative)**
4. **Empty Latent Image**
5. **KSampler**
6. **VAE Decode**
7. **Save Image**

Этого достаточно, чтобы:
- выбрать модель;
- задать запрос;
- зафиксировать параметры;
- получить результат;
- сохранить workflow и изображения.

## Как работать на занятии

### Шаг 1. Выберите одну понятную задачу
Подходят задачи вида:
- концепт главного героя;
- игровая локация;
- ключевой проп / предмет;
- mood-shot под ваш проект.

Не подходят задачи вида:
- “сделать весь стиль игры сразу”; 
- “сделать вообще всё красиво”; 
- “просто попробовать ComfyUI”.

### Шаг 2. Зафиксируйте цель в `logs/prompt-plan.md`
Минимум, который стоит записать:
- что именно вы генерируете;
- 3–5 обязательных признаков;
- 3–5 ограничений;
- по каким критериям будете отбирать результат.

### Шаг 3. Соберите или откройте базовый workflow
На этом занятии не нужно усложнять граф. Цель — научиться уверенно работать с минимальным `txt2img` пайплайном.

### Шаг 4. Зафиксируйте параметры
Обязательно запишите:
- checkpoint;
- seed;
- steps;
- CFG;
- sampler / scheduler;
- latent size.

### Шаг 5. Сгенерируйте 4 варианта
Держите базовую логику такой:
- одна задача;
- один базовый workflow;
- небольшой набор вариаций;
- осмысленный отбор.

### Шаг 6. Сохраните артефакты
После генерации у вас должны быть:
- JSON workflow;
- 4 selected output;
- короткие notes о том, что получилось и что хотите пробовать дальше.

## Типовые ошибки

- слишком общий prompt;
- отсутствие negative prompt;
- потерянный `seed`;
- изменение слишком большого числа параметров сразу;
- несохранённый workflow;
- результаты не связаны с проектом;
- в проекте лежат десятки файлов, но нет осмысленного selected-набора.

## Если хотите поставить ComfyUI локально (опционально)

Локальная установка не обязательна для занятия, но может быть полезна как запасной путь.

### Рекомендуемая логика

- **Самый простой путь:** Desktop-версия ComfyUI.
- **Для Windows:** можно использовать portable-сборку.
- **Для продвинутых пользователей:** manual install через Python-окружение.

### Что нужно для минимального локального старта

1. Установить ComfyUI удобным для вас способом.
2. Положить SDXL checkpoint в папку `models/checkpoints`.
3. Если используется отдельный VAE — положить его в `models/vae`.
4. Запустить ComfyUI.
5. Проверить, что открывается интерфейс и виден checkpoint.
6. Собрать простой `txt2img` workflow и сохранить его.

### Что проверять после установки

- интерфейс открывается;
- модель доступна в списке;
- workflow сохраняется в `.json`;
- изображение сохраняется без ошибок;
- после перезапуска workflow открывается повторно.

### Когда локальная установка не нужна

Если локальный запуск требует долгой отладки, лучше не тратить на это учебное время и работать в облачном варианте.

## Примеры prompt’ов для SDXL

Ниже — стартовые примеры. Их не нужно копировать вслепую: подставляйте особенности своего проекта.

### Базовый negative prompt

`low quality, blurry, text, watermark, logo, deformed anatomy, extra limbs, bad hands, cropped, duplicate elements`

### 1. Локация — sci-fi лаборатория

`abandoned underground research lab, sci-fi game environment concept art, cold industrial materials, broken panels, cables, emergency lights, atmospheric fog, cinematic lighting, wide shot, strong focal point, detailed environment`

### 2. Локация — фэнтезийное святилище

`forest shrine, fantasy game environment concept art, mossy stones, lanterns, carved wood, morning mist, soft directional light, quiet mystical atmosphere, medium wide shot, detailed scene`

### 3. Персонаж — главный герой

`playable sci-fi protagonist, full body character concept art, practical combat outfit, modular armor, utility belt, readable silhouette, neutral pose, studio lighting, realistic materials, detailed costume design`

### 4. Персонаж — NPC механик

`female mechanic NPC, dieselpunk game character concept, worn overalls, tool harness, gloves, workshop background, warm industrial lighting, half body portrait, detailed costume, grounded design`

### 5. Проп — артефакт

`ancient crystal artifact, prop concept art for video game, floating shards, metallic frame, glowing core, dark neutral background, centered composition, dramatic rim light, realistic detail`

### 6. Проп — интерфейсный терминал

`sci-fi control terminal, prop concept art, rugged industrial casing, glowing interface, functional buttons, worn materials, three quarter view, isolated presentation, high detail`

### 7. Mood-shot проекта

`key visual for an indie sci-fi adventure game, lone explorer in front of a massive alien structure, dramatic sky, atmospheric perspective, cinematic composition, strong silhouette, detailed concept art`

## Шаблон prompt’а для своей задачи

Используйте такой каркас:

`[что это], [жанр / тип игры], [главные визуальные признаки], [материалы / фактуры], [свет], [ракурс], [фон / окружение], [уровень детализации]`

Пример:

`ritual chamber, dark fantasy game environment concept art, cracked stone floor, red banners, candle light, centered composition, underground temple background, high detail`

## Чек-лист самопроверки перед сдачей

- [ ] У меня есть `workflows/w04_sdxl_txt2img.json`
- [ ] У меня сохранены 4 selected output
- [ ] Я записал(а) `seed`
- [ ] Я знаю, какой checkpoint использовал(а)
- [ ] У меня есть краткий `prompt-plan`
- [ ] У меня есть `workflow-notes` с выводом
- [ ] Результаты связаны с моим проектом игры

## Домашнее задание (2–4 часа)

1. Подготовить **2–3 версии prompt’а** для своего проекта:
   - персонаж;
   - локация;
   - проп или mood-shot.
2. В `logs/workflow-notes.md` записать:
   - какие параметры вы использовали;
   - что менялось между вариантами;
   - какой результат лучше поддерживает концепт игры и почему.
3. При необходимости сделать дополнительные генерации и обновить 4 selected output.
4. В `GDD.md` кратко обновить визуальную гипотезу проекта:
   - какой стиль вы пробуете;
   - какой тип ассета сейчас приоритетен;
   - что уже получилось удачно, а что ещё требует доработки.

## Главное правило занятия

Не пытайтесь сразу сделать “финальный арт”.

Цель этого занятия — собрать первый **рабочий и воспроизводимый** pipeline, который:
- понятен вам;
- понятен преподавателю;
- поддерживает проект;
- может быть улучшен на следующих занятиях.
