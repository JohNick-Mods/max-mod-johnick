# MAX 26.26.0 V3 — Технический аудит мода

**Версия стока**: 26.26.0  
**Версия мода**: V3 (Privacy Mod by JohNick)  
**Дата**: 2026-08-10  
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **V3**: убрана иконка 🗑️ на удалённых сообщениях (по запросу). Антиудаление сохраняет текст и время удаления.

---

## 🐞 Исправлено в V3

### Убрана иконка 🗑️ на удалённых сообщениях

- `apply_v_antidel_trash_prefix.py` (DB-path, sentinel `:ad_trash_v1`) и `apply_v_antidel_trash_prefix_ws.py` (WS live-path, sentinel `:ad_trash_ws_v1`) исключены из `make_v1.py` APPLY_STEPS.
- Антиудаление продолжает сохранять текст удалённых сообщений. Маркер времени удаления (ЧЧ:ММ:СС ДД.ММ) остаётся. Эмодзи-префикс 🗑️ в тексте не добавляется.
- Верификация: sentinels `:ad_trash_v1` и `:ad_trash_ws_v1` отсутствуют в smali после пересборки.

---

## 🐞 Исправлено в V2

### Кнопки настроек оставались тёмными на светлой теме

- Фон диалога читал системную `Configuration.uiMode`, цвета кнопок — флаг мода `AppFlags.amoledTheme`. При несовпадении — некорректный UI.
- Фикс: новый метод `isDarkThemeModa(Context)Z` в `DialogsUtils` как единственная точка чтения темы. `getDialogTheme()` и `roundAndTint()` переведены на `isDarkThemeModa`. `getCategoryBg/TextColor`: светлая → `#E0E0E0`/`#212121`, тёмная/AMOLED → `#444444`/`#FFFFFF`. Охват: 39 вызовов во всех 3 режимах.
- `apply_v_light_theme.py`: поиск исправлен на `re.IGNORECASE`, guard при 0 заменах.

### Автосинк темы мода с системной (one-time)

- `syncNightModeWithSystem(Context)V` в `DialogsUtils`, вызывается из `MainActivity.onCreate()` один раз. Флаг `modThemeSyncedV2` в SharedPreferences. Ручной выбор темы не перезаписывается.

---

## 🔍 Верификация (V3)

| Проверка | Результат |
|---|---|
| sentinel `:ad_trash_v1` в smali | **0 вхождений** |
| sentinel `:ad_trash_ws_v1` в smali | **0 вхождений** |
| `apksigner verify` (все 4 APK) | ✅ v2+v3 |
| exact ABI (arm64/arm7), no tracer | ✅ |
| distinct SHA arm64 ≠ arm7 | ✅ |

## 🔍 Верификация (V2)

| Проверка | Результат |
|---|---|
| Хардкод `-0xCFCFD0` во всех 3 dex | **0 вхождений** |
| `getCategoryBg` / `getCategoryTextColor` | **39 / 39** — симметрично |
| `isDarkThemeModa` читает только `AppFlags->amoledTheme:Z` | ✅ |
| `getDialogTheme` + `roundAndTint` → `isDarkThemeModa` | ✅ |
| Вызовов `isNightMode` (за пределами объявления) | **0** |
| Register-collision (AppFlagsView) | **CLEAN** |

**Smoke test V3:** S25 Ultra (RFCX5013V5F) + HA218XJZ — `install -r`, 0 FATAL/VerifyError. Удалённые сообщения сохраняются без 🗑️.

---

## 🛡 Профиль приватности (без изменений с V1)

### Невидимка (`invisible`) — мастер-переключатель

| Дочерний флаг | Что делает |
|---|---|
| `antiTyping` | собеседник не видит набор текста |
| `antiRead` | собеседник не видит отметку «прочитано» |
| `smartAntiRead` | «прочитано» уходит только после вашего ответа (consume-on-use) |
| `antiPresence` | онлайн / last-seen скрыты |

### Антиудаление (`antiDelete`) — 4 слоя

1. **L3** — Central WS chat-event (live-канал).
2. **L4** — Room ORM body-clear (локальный SQL-сброс).
3. **Маркер** «· удалено» + время удаления (ЧЧ:ММ:СС ДД.ММ). Emoji 🗑️ не добавляется (убран в V3).
4. **FCM офлайн-персист** — удаления при убитом приложении восстанавливаются при следующем старте.

### Нейтрализация телеметрии — 4 чокпоинта (V1)

`p0_telem_push`, `p0_telem_onelog`, `p0_telem_operator` (IP-strip), `p0_telem_dps`.

---

## 📦 Артефакты (V3)

| Файл | SHA-256 |
|---|---|
| `MAX_26.26.0_arm64_V3.apk` | `3d88f2c0ae8bbec6fce9489a191b6fd56fd8518a6b95909716f9a4dfe47ea401` |
| `MAX_26.26.0_arm64_V3_clone.apk` | `99c7c145f9e839f7592f45052abd91c3e9e70af517777ad4ae07854ddd687cb6` |
| `MAX_26.26.0_arm7_V3.apk` | `c4ee2363fdf9c6e537402f5375f42a317b50919ee5da7a0d7bb487422015b1ca` |
| `MAX_26.26.0_arm7_V3_clone.apk` | `9972ac72d5b2cdbdcace9ac5089350d14770f2a26521bf79b426a9f07024587a` |

Подпись: v2+v3 APK Signature Scheme. Cert SHA-1 идентичен V1/V2 — install -r без потери данных.

---

*Аудит выполнен статическим анализом smali (baksmali + независимый агент-верификатор). Динамическое тестирование: smoke на S25 Ultra + HA218XJZ.*
