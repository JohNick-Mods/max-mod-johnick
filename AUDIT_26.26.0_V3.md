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
| `MAX_26.26.0_arm64_V3.apk` | `41729594cf8fbc578d3d9d836d5a6b7a53b647933d168b3b38df78280475ce8d` |
| `MAX_26.26.0_arm64_V3_clone.apk` | `d58e0e0318fcc98c1a2a35196d06c9282a59e488fb2bd058b68aeb7c7d305f5f` |
| `MAX_26.26.0_arm7_V3.apk` | `554fd694abbab4ac16414b414439840e5a8a964897c0d50c4dd20197d058ef39` |
| `MAX_26.26.0_arm7_V3_clone.apk` | `7c6f1faa0644b84932df731485ddb03e89e5be4f8d0a70199e121cc83e4f3e2f` |

Подпись: v2+v3 APK Signature Scheme. Cert SHA-1 идентичен V1/V2 — install -r без потери данных.

---

*Аудит выполнен статическим анализом smali (baksmali + независимый агент-верификатор). Динамическое тестирование: smoke на S25 Ultra + HA218XJZ.*
