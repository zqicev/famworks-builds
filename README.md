# FamWorks Builds

Публичные сборки (modpacks) для лаунчера FamWorks + хостинг кастомных модов и релизов лаунчера.

Лаунчер читает файлы сборок напрямую через `raw.githubusercontent.com`.

## Структура

```
modpacks/
├── index.json          # список всех сборок (что показывать в сайдбаре)
└── <id>.json           # полные данные каждой сборки
```

## `index.json`

```json
{
  "modpacks": [
    {
      "id": "famworks-main",
      "name": "FamWorks",
      "description": "Краткое описание",
      "mc_version": "1.21.1",
      "loader": "fabric",
      "loader_version": "0.16.5",
      "updated_at": "2025-06-01T00:00:00Z"
    }
  ]
}
```

## `<id>.json`

```json
{
  "id": "famworks-main",
  "name": "FamWorks",
  "description": "Краткое описание",
  "long_description": "Полное описание для вкладки Обзор",
  "mc_version": "1.21.1",
  "loader": "fabric",
  "loader_version": "0.16.5",
  "fabric_api_version": "0.110.0+1.21.1",
  "updated_at": "2025-06-01T00:00:00Z",
  "changelog": [
    { "version": "1.21.1", "description": "Что изменилось" }
  ],
  "mods": [ ... ]
}
```

### Поля мода

| Поле | Описание |
|------|----------|
| `id` | уникальный id мода внутри сборки |
| `name` | отображаемое имя |
| `modrinth_id` | **project id** на Modrinth (лаунчер сам найдёт нужную версию) |
| `download_url` | прямая ссылка на `.jar` (альтернатива `modrinth_id`, для кастомных модов) |
| `sha512` | контрольная сумма (hex) — проверяется после загрузки. Обязательна для кастомных jar |
| `filename` | имя файла как он ляжет в `mods/` |
| `category` | категория для UI (Оптимизация, Графика, Карта, Кастом…) |
| `size_mb` | размер в МБ для отображения |
| `required` | `true` — обязательный (нельзя выключить/удалить), `false` — опциональный |

Мод задаётся **либо** через `modrinth_id`, **либо** через `download_url`.

## Кастомные `.jar` (не из Modrinth)

Свои моды храним как **ассеты в GitHub Releases этого репозитория** — это не раздувает git-историю.

1. **Создай релиз** для модов (можно переиспользовать), напр. тег `mods`:
   - Releases → Draft a new release → тег `mods` → Publish
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

При обновлении мода — залей новый `.jar`, обнови `download_url`, `filename` и `sha512`, подними `updated_at` сборки. Лаунчер скачает новый файл (старое имя останется — при необходимости почисти вручную).

## Как добавить/обновить сборку

1. Создай/измени `modpacks/<id>.json`
2. Добавь краткую запись в `modpacks/index.json`
3. Обнови `updated_at` — лаунчер покажет бейдж «ОБНОВЛЕНО»
4. Закоммить и запушь — лаунчер подтянет изменения по кнопке ↻

## Релизы лаунчера

Сюда же `npm run release` (из репозитория лаунчера) публикует установщик + `latest.yml` + `.blockmap` для автообновления.
