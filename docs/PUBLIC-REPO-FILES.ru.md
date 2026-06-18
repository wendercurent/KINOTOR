# Файлы публичного репозитория (без исходного кода)

Что **включать** и **исключать** при публикации `wendercurent/KinoTor` на GitHub.

Используйте скрипт `scripts/prepare-public-repo.mjs` — он копирует только разрешённый набор.

---

## Включать в публичный git

Скрипт `npm run publish:public` копирует всё ниже + создаёт **декоративную структуру** (папки с README «исходники закрыты»).

| Путь | Назначение |
|------|------------|
| `README.md` | Главная страница (SEO / ASO) |
| `README.en.md` | Английская версия |
| `LICENSE` | Условия распространения бинарников |
| `CHANGELOG.md` | История версий |
| `SECURITY.md` | Политика безопасности |
| `RELEASE_NOTES.template.md` | Шаблон Release |
| `docs/user-guide/` | Руководство, FAQ |
| `docs/legal/` | Privacy, terms |
| `docs/screenshots/*` | Скриншоты |
| `docs/GITHUB-RELEASE.ru.md` | Инструкция мейнтейнера |
| `docs/PUBLIC-REPO-FILES.ru.md` | Этот файл |
| `.github/ISSUE_TEMPLATE/` | Шаблоны Issues |
| `.github/FUNDING.yml` | Ссылка на Telegram |
| `assets/branding/`, `assets/icons/` | Плейсхолдеры под брендинг |
| `src/README.md`, `renderer/README.md`, … | Заглушки «код не публикуется» |
| `.gitignore` | Минимальный |

### Минимальный `.gitignore` для публичного репо

```gitignore
.DS_Store
Thumbs.db
*.log
.env
.env.*
```

---

## НЕ включать в публичный git

### Исходный код и сборка

| Путь | Причина |
|------|---------|
| `src/` | Исходники TypeScript |
| `renderer/` | UI (HTML/CSS/JS) |
| `dist/` | Скомпилированный main process |
| `package.json`, `package-lock.json` | Зависимости и скрипты сборки |
| `tsconfig.json` | Конфиг компилятора |
| `node_modules/` | Зависимости |
| `scripts/` | Скрипты лицензий и утилиты |
| `tools/` | Прокси, realm-admin, wrangler |
| `website/` | Сайт-компаньон |
| `prebuilts/` | TorrServer и плееры (внутри установщика) |
| `licensing/private-key.pem` | **Секрет** |
| `licensing/public-key.pem` | Часть сборки, не нужна в docs-репо |

### Артефакты сборки (только Releases, не git)

| Путь | Куда |
|------|------|
| `release/` | GitHub Release assets |
| `release-portable/` | GitHub Release assets |
| `*.exe`, `*.blockmap` | Releases |
| `latest.yml` | Генерируется builder для auto-update; прикреплять к Release |

### Внутренняя документация разработки

| Путь | Причина |
|------|---------|
| `docs/PROJECT-STATE.ru.md` | Внутренний обзор архитектуры |
| `docs/SECURITY-AUDIT.ru.md` | Аудит безопасности |
| `docs/COMMENTS-FEASIBILITY.ru.md` | Внутренние заметки |
| `KINOTOR_PUBLISHING_GUIDE.md` | Дублирует GITHUB-RELEASE с dev-деталями |
| `RELEASE_CHECKLIST.md` | Внутренний QA |
| `LICENSE_SETUP.md` | Настройка ключей |
| `ROADMAP.md`, `WEBSITE_*.md`, `KINOFLOW_*.md` | Планы разработки |
| `TELEGRAM_PROMO_PACK.md` | Маркетинг |
| `.cursorrules` | Настройки IDE |

### CI/CD и секреты

| Путь | Причина |
|------|---------|
| `.github/workflows/ci.yml` | Требует полного исходного кода |
| `.github/workflows/release.yml` | Не создавать в публичном репо без private fork |
| `.env`, `*.pem`, `*.key`, `*.p12` | Секреты |
| `.tmp_*` | Временные файлы отладки |
| `release/builder-debug.yml` | Артефакт electron-builder |

### Прочее

| Путь | Причина |
|------|---------|
| `prebuilts/windows/player/portable_config/cache/` | Кэш шейдеров MPV |
| `gear.svg`, `popular.svg` в корне | Дубликаты иконок |
| Любые API-ключи в markdown | Утечка секретов |

---

## GitHub Releases (не git)

Загружать **вручную** на каждый тег `vX.Y.Z`:

1. `Kinotor Setup X.Y.Z.exe` — основной установщик
2. `Kinotor X.Y.Z.exe` — portable (опционально, но рекомендуется)
3. `latest.yml` — для `electron-updater` (если генерируется в `release/`)
4. `*.exe.blockmap` — для дельта-обновлений (если используется)

Не загружать: `win-unpacked/`, исходники, `node_modules`, debug-логи.

---

## Два репозитория: схема

```
┌─────────────────────────────┐     локальная сборка      ┌──────────────────────────┐
│  Kinoflow (приватный/local) │  ──────────────────────►  │  release/*.exe           │
│  полный код, npm, CI        │                           └───────────┬──────────────┘
└─────────────────────────────┘                                       │
         │                                                             │ gh release upload
         │ prepare-public-repo.mjs                                     ▼
         │ (только docs)                               ┌──────────────────────────┐
         └──────────────────────────────────────────►  │  wendercurent/KinoTor    │
                                                        │  публичный: README, etc. │
                                                        └──────────────────────────┘
```

---

## Проверка перед первым push

```bash
# В папке публичного репо не должно быть:
git ls-files | rg -i "src/|renderer/|package\.json|\.ts$|preload|torrserver\.ts"

# Должны быть:
git ls-files README.md LICENSE CHANGELOG.md
```

---

## Обновление при новой версии

1. Собрать в приватном репо → VT → Release assets.
2. Обновить `CHANGELOG.md`, VirusTotal URL в `README.md`.
3. `node scripts/prepare-public-repo.mjs` → commit → push в `KinoTor`.
4. Создать GitHub Release с `.exe` файлами.

Подробности: [GITHUB-RELEASE.ru.md](GITHUB-RELEASE.ru.md).
