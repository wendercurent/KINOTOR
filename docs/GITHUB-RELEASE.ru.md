# Публикация Kinotor на GitHub без исходного кода

Пошаговая инструкция для мейнтейнера: сборка установщика локально, VirusTotal, создание Release и обновление публичного репозитория `wendercurent/KinoTor`.

> **Приватная разработка** ведётся в полном репозитории (папка `Kinoflow` на диске).  
> **Публичный репозиторий** содержит только README, LICENSE, CHANGELOG, docs и артефакты в Releases — без `src/`, `renderer/`, `package.json` и прочего кода.

---

## Стратегия: два варианта

### Вариант A — «Документация + Releases» (рекомендуется)

Публичный репозиторий `KinoTor`:

```
KinoTor/                    ← публичный GitHub
├── README.md
├── README.en.md
├── LICENSE
├── CHANGELOG.md
├── RELEASE_NOTES.template.md   (опционально, для копирования)
├── docs/
│   ├── screenshots/
│   ├── GITHUB-RELEASE.ru.md
│   └── PUBLIC-REPO-FILES.ru.md
└── .gitignore                  (минимальный)
```

- Сборка **только локально** из приватного/полного клона.
- Установщики загружаются в **GitHub Releases** вручную или через `gh release upload`.
- **Не включать** `.github/workflows/ci.yml` — он требует исходников в репозитории.

### Вариант B — «Только Releases»

Репозиторий с одним `README.md` + `LICENSE`; всё остальное — в Releases. Минимум файлов, но хуже SEO и нет CHANGELOG в git.

**Для Kinotor выбран вариант A.**

---

## Подготовка публичной копии репозитория

### Автоматически (скрипт)

Из **полного** репозитория разработки:

```bash
node scripts/prepare-public-repo.mjs
```

Скрипт создаёт папку `../KinoTor-public/` (или путь из переменной `KINOTOR_PUBLIC_REPO_DIR`) с разрешёнными файлами.

### Вручную

См. полный список в [PUBLIC-REPO-FILES.ru.md](PUBLIC-REPO-FILES.ru.md).

```bash
# Пример: новый пустой репозиторий
mkdir ../KinoTor-public && cd ../KinoTor-public
git init
# скопировать файлы из списка «Включать»
git add .
git commit -m "Initial public release repo (docs only)"
git remote add origin git@github.com:wendercurent/KinoTor.git
git push -u origin main
```

---

## Цикл релиза (каждая версия)

### Шаг 1. Preflight в репозитории разработки

```bash
cd E:/develop/Reyohoho/konosite/Kinoflow
npm install
npm run release:preflight:full
```

Команда выполняет:

1. `npm run verify` — тесты (`tsc` + `node --test`)
2. `npm run build:win` — NSIS-установщик
3. `npm run build:win:portable` — портативный `.exe`

**Артефакты** (после успешной сборки):

| Файл | Путь |
|------|------|
| Установщик NSIS | `release/Kinotor Setup <version>.exe` |
| Портативная сборка | `release/Kinotor <version>.exe` |
| Распакованное приложение | `release/win-unpacked/` (для smoke-теста, не загружать в Release) |

Если portable-папка заблокирована:

```bash
npx electron-builder --win portable --publish=never --config.directories.output=release-portable
```

Дополнительный чеклист: `RELEASE_CHECKLIST.md` (только в приватном репо).

### Шаг 2. Smoke-тест

1. Установить `Kinotor Setup … .exe` на чистой VM или тестовой машине.
2. Первый запуск, каталог, торрент, воспроизведение.
3. Проверить обновление с предыдущей версии (если применимо).

### Шаг 3. VirusTotal

1. Открыть https://www.virustotal.com/
2. Загрузить **`Kinotor Setup <version>.exe`** (основной артефакт для пользователей).
3. Дождаться отчёта. При 0–2 ложных срабатываниях из 70+ — норма для Electron.
4. Скопировать ссылку вида:  
   `https://www.virustotal.com/gui/file/<sha256>`
5. **Обновить плейсхолдер** в:
   - `README.md` (раздел «Проверка на VirusTotal»)
   - `README.en.md` (если публикуете)
   - `RELEASE_NOTES` для этого релиза
6. Закоммитить обновление README в **публичный** репозиторий **до** или **вместе** с тегом.

Опционально: загрузить portable `.exe` отдельным отчётом VT и приложить вторую ссылку в Release notes.

### Шаг 4. Версия и CHANGELOG

1. В **приватном** репо: версия в `package.json` = `X.Y.Z`.
2. Добавить секцию в `CHANGELOG.md`.
3. Скопировать актуальный `CHANGELOG.md` в публичный репозиторий.

### Шаг 5. Синхронизация публичного репозитория

```bash
node scripts/prepare-public-repo.mjs
cd ../KinoTor-public
git add -A
git commit -m "docs: release vX.Y.Z — VirusTotal link, changelog"
git push origin main
```

### Шаг 6. Создать GitHub Release

**Через веб-интерфейс:**

1. https://github.com/wendercurent/KinoTor/releases/new
2. Tag: `vX.Y.Z` (должен совпадать с `package.json`, префикс `v`)
3. Title: `Kinotor vX.Y.Z`
4. Описание: из `RELEASE_NOTES.template.md` (заполненный)
5. Прикрепить файлы:
   - `Kinotor Setup X.Y.Z.exe`
   - `Kinotor X.Y.Z.exe`
6. Отметить **Latest release** для стабильной beta-ветки.

**Через GitHub CLI:**

```bash
# из папки Kinoflow (полный репо), где лежат артефакты
gh release create vX.Y.Z \
  "release/Kinotor Setup X.Y.Z.exe" \
  "release/Kinotor X.Y.Z.exe" \
  --repo wendercurent/KinoTor \
  --title "Kinotor vX.Y.Z" \
  --notes-file RELEASE_NOTES_FILLED.md
```

`electron-updater` в приложении ищет релизы на `wendercurent/KinoTor` (см. `package.json` → `build.publish`).

### Шаг 7. После публикации

- [ ] Проверить https://github.com/wendercurent/KinoTor/releases/latest
- [ ] Установка с GitHub на тестовой машине
- [ ] В приложении: **Настройки → Проверить обновления**
- [ ] Обновить ссылку на сайте-компаньоне (если используется)
- [ ] Пост в [@kinotorapp](https://t.me/kinotorapp)

---

## Почему нет CI workflow в публичном репо

Workflow `.github/workflows/ci.yml` в полном репозитории выполняет `npm ci` и `npm run build` — для этого нужны исходники. В публичном репозитории **без кода** такой workflow бессмысленен и раскрывает отсутствие исходников при failed build.

**Альтернативы автоматизации (только в приватном репо):**

| Подход | Описание |
|--------|----------|
| Private repo + Actions | Сборка в приватном форке, `gh release upload` в публичный `KinoTor` с PAT |
| Self-hosted runner | Runner на машине мейнтейнера, секреты не в публичном git |
| Ручная загрузка | Текущий рекомендуемый путь для публичной beta |

Не коммитьте в публичный репозиторий: `package.json`, `src/`, workflow с `npm ci`, приватные ключи, `licensing/private-key.pem`.

---

## Именование: Kinoflow vs Kinotor

| Контекст | Имя |
|----------|-----|
| Папка проекта на диске | `Kinoflow` |
| Продукт, установщик, GitHub | **Kinotor** |
| npm / appId | `kinotor`, `ru.kinotor` |
| GitHub repo | `wendercurent/KinoTor` |

В пользовательской документации всегда **Kinotor**.

---

## Быстрая шпаргалка команд

### Полный цикл одной цепочкой

```bash
# 1. Сборка установщика (приватный репо Kinoflow)
npm run release:preflight:full

# 2. VirusTotal — загрузить Kinotor Setup X.Y.Z.exe, вставить ссылку в README.md

# 3. Скриншоты — положить в docs/screenshots/*.png

# 4. Подготовить публичный репо (README, docs, декоративные папки)
npm run publish:public

# 5a. Только push документации на GitHub
npm run publish:github:push

# 5b. Или всё сразу: push + GitHub Release с .exe из release/
npm run publish:github:all 0.1.4
```

### Отдельные команды

```bash
npm run build:win              # только NSIS
npm run build:win:portable     # только portable
npm run publish:public         # ../KinoTor-public/ без git
npm run publish:github         # то же + подсказки
npm run publish:github:push    # + git commit + push
npm run publish:github:release 0.1.4   # + gh release create

# Вручную (если без скрипта):
gh release create v0.1.4 "release/Kinotor Setup 0.1.4.exe" "release/Kinotor 0.1.4.exe" \
  --repo wendercurent/KinoTor --title "Kinotor v0.1.4" --notes-file RELEASE_NOTES.template.md
```

### Первый push (один раз)

```bash
cd ../KinoTor-public
git remote add origin git@github.com:wendercurent/KinoTor.git
git push -u origin main
```

Требуется: `git`, `gh auth login` (для Release).

---

## Скриншоты для README

1. Запустить собранное приложение.
2. Сохранить PNG в `docs/screenshots/`:
   - `home.png` — главная
   - `detail.png` — карточка фильма
   - `anime.png` — аниме
   - `watch-party.png` — совместный просмотр
   - `settings.png` — настройки
3. Скопировать папку `docs/screenshots/` в публичный репозиторий.
4. Пути в `README.md` уже указывают на `docs/screenshots/*.png`.

Рекомендуемый размер: ширина 1280–1920 px, PNG или WebP.
