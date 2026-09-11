# FamWorks Builds

Публичные сборки (modpacks) для лаунчера FamWorks + хостинг кастомных модов, ассетов
персонажа и релизов лаунчера.

Лаунчер читает файлы напрямую через `raw.githubusercontent.com`. Достаточно закоммитить и
запушить изменения — лаунчер подтянет их по кнопке ↻.

## Структура

```
modpacks/
├── index.json                       # список сборок для сайдбара
├── <id>.json                        # полные данные каждой сборки
├── anim/
│   └── <id>-idle.animation.json     # idle-анимация персонажа (экспорт Blockbench)
└── scene/
    └── <id>.gltf                    # единая 3D-сцена персонажа (экспорт Blockbench)
```

## `index.json`

Краткий список сборок — что показывать в сайдбаре. Поля совпадают с шапкой `<id>.json`.

```json
{
  "modpacks": [
    {
      "id": "famrokia-main",
      "name": "Фамрокия",
      "description": "Приватный ванильный приключенческий сервер",
      "mc_version": "1.21.1",
      "loader": "fabric",
      "loader_version": "0.18.4",
      "updated_at": "2026-08-16T16:33:11.905Z"
    }
  ]
}
```

`loader`: `fabric` | `forge` | `neoforge` | `quilt` | `vanilla`.

## `<id>.json`

Полные данные сборки.

```json
{
  "id": "famrokia-main",
  "name": "Фамрокия",
  "description": "Краткое описание (сайдбар)",
  "long_description": "Полное описание для вкладки Обзор",
  "mc_version": "1.21.1",
  "loader": "fabric",
  "loader_version": "0.18.4",
  "fabric_api_version": "0.116.12+1.21.1",
  "updated_at": "2026-08-16T16:33:11.905Z",
  "changelog": [
    { "version": "1.21.1", "description": "Что изменилось" }
  ],
  "character": {
    "scene": "https://raw.githubusercontent.com/zqicev/famworks-builds/main/modpacks/scene/famrokia.gltf",
    "idle": "https://raw.githubusercontent.com/zqicev/famworks-builds/main/modpacks/anim/famrokia-idle.animation.json"
  },
  "mods": [ ... ],
  "resourcepacks": [ ... ],
  "shaders": [ ... ],
  "servers": [
    { "name": "Фамрокия", "ip": "play.example.com", "port": 25565 }
  ],
  "configs": [
    { "path": "config/sodium-options.json", "download_url": "https://.../cfg-....json", "sha512": "...", "overwrite": false }
  ]
}
```

`fabric_api_version` — лаунчер сам скачает Fabric API именно этой версии (в `mods` добавлять не нужно).

## Персонаж и сцена (`character`)

На вкладке «Обзор» лаунчер показывает анимированную 3D-модель игрока. Текстура модели —
это скин активного аккаунта (лицензионный тянется с Mojang, иначе Steve). Есть два режима:

- **`scene`** (рекомендуется) — один экспорт Blockbench (`.gltf`/`.glb`) с игроком, объектами
  (пчела/питомец/декор) и всеми анимациями в одном файле. Рендерится как единая сцена; текстура
  игрока в ней подменяется скином аккаунта. Файл кладём в `modpacks/scene/` и ссылаемся raw-URL.
- **`idle`** — только idle-анимация (`.animation.json`) для встроенной модели игрока, без
  собственной сцены. Файл кладём в `modpacks/anim/`.

Если задан `scene`, используется он (поля `idle*`/`objects` игнорируются).

| Поле | Описание |
|------|----------|
| `scene` | URL к `.gltf`/`.glb` единой сцены (игрок + объекты + анимации). Приоритетнее остального |
| `idle` | URL к `.animation.json` с idle-анимацией (когда нет `scene`); иначе встроенная idle |
| `idle_data` | инлайн-содержимое `.animation.json` (для локальных сборок; приоритетнее `idle`) |
| `idle_name` | имя анимации внутри файла, если их несколько; иначе берётся первая |
| `objects` | доп. glTF-объекты со своими анимациями: `[{ "url": "...", "scale": 1 }]` (устар. — используйте `scene`) |

Поле `character` целиком опционально — без него показывается модель со стандартной idle-анимацией.

## Моды, ресурспаки, шейдеры

`mods`, `resourcepacks`, `shaders` — массивы одной формы. Разница только в папке установки:

| Массив | Куда ставится |
|--------|---------------|
| `mods` | `mods/` |
| `resourcepacks` | `resourcepacks/` |
| `shaders` | `shaderpacks/` |

### Поля элемента

| Поле | Описание |
|------|----------|
| `id` | уникальный id внутри сборки |
| `name` | отображаемое имя |
| `modrinth_id` | **project id** на Modrinth (лаунчер сам найдёт совместимую версию) |
| `modrinth_version_number` | конкретная версия Modrinth (иначе берётся последняя совместимая) |
| `curseforge_id` | project id на CurseForge (альтернатива Modrinth) |
| `download_url` | прямая ссылка на файл (для кастомных модов, не из Modrinth/CF) |
| `filename` | имя файла, как оно ляжет в папку |
| `version` | версия (для отображения) |
| `category` | категория для UI (Оптимизация, Графика, Карта, Кастом…) |
| `size_mb` | размер в МБ для отображения |
| `required` | `true` — обязательный (нельзя выключить/удалить), `false` — опциональный |
| `sha512` | контрольная сумма hex (Modrinth / `Get-FileHash`). Обязательна для кастомных `download_url` |
| `sha1` | контрольная сумма hex (CurseForge). Проверяется, если нет `sha512` |

Источник задаётся **одним из**: `modrinth_id`, `curseforge_id` или `download_url`.

## Серверы (`servers`)

Добавляются в список мультиплеера игрока (`servers.dat`) при запуске. Серверы, которые игрок
добавил сам, сохраняются.

| Поле | Описание |
|------|----------|
| `name` | отображаемое имя сервера |
| `ip` | хост (без порта) |
| `port` | порт (по умолчанию 25565) |

## Конфиги (`configs`)

Файлы, которые кладутся в папку игры по указанному пути.

| Поле | Описание |
|------|----------|
| `path` | путь относительно папки игры: `config/mod.json`, `options.txt`, любой вложенный |
| `download_url` | ссылка на файл (обычно в Releases этого репозитория) |
| `sha512` | контрольная сумма (hex) |
| `overwrite` | `true` — всегда перезаписывать; `false`/нет — положить только если файла нет (юзер может менять) |
| `extract` | `true` — `download_url` это zip-архив: распаковать в корень сборки (структура папок сохраняется), архив удалить |

## Кастомные `.jar` (не из Modrinth/CurseForge)

Свои моды храним как **ассеты в GitHub Releases этого репозитория** — это не раздувает git-историю.

1. **Создай релиз** для модов (можно переиспользовать), напр. тег `mods`:
   Releases → Draft a new release → тег `mods` → Publish.
2. **Залей `.jar`** в этот релиз (перетащи в Assets).
3. **Возьми ссылку** на ассет — вид:
   `https://github.com/zqicev/famworks-builds/releases/download/mods/famworks-core-1.0.jar`
4. **Посчитай sha512** файла:
   - PowerShell: `Get-FileHash -Algorithm SHA512 .\famworks-core-1.0.jar`
   - bash: `sha512sum famworks-core-1.0.jar`
5. **Добавь мод** в `<id>.json`:
   ```json
   {
     "id": "famworks-core",
     "name": "FamWorks Core",
     "filename": "famworks-core-1.0.jar",
     "download_url": "https://github.com/zqicev/famworks-builds/releases/download/mods/famworks-core-1.0.jar",
     "sha512": "9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08...",
     "version": "1.0",
     "category": "Кастом",
     "size_mb": 2.4,
     "required": true
   }
   ```

При обновлении мода — залей новый `.jar`, обнови `download_url`, `filename` и `sha512`, подними
`updated_at` сборки. Лаунчер скачает новый файл (старое имя останется — при необходимости почисти вручную).

## Как добавить/обновить сборку

1. Создай/измени `modpacks/<id>.json`.
2. Добавь/обнови краткую запись в `modpacks/index.json`.
3. Ассеты персонажа (если есть) положи в `modpacks/scene/` и/или `modpacks/anim/` и сошлись на них raw-URL в `character`.
4. Обнови `updated_at` — лаунчер покажет бейдж «ОБНОВЛЕНО».
5. Закоммить и запушь — лаунчер подтянет изменения по кнопке ↻.

## Релизы лаунчера

Сюда же `npm run release` (из репозитория лаунчера) публикует установщик + `latest.yml` +
`.blockmap` для автообновления.
