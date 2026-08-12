# MAX 26.26.0 V4 — Технический аудит мода

**Версия стока**: 26.26.0  
**Версия мода**: V4 (Privacy Mod by JohNick)  
**Дата**: 2026-08-12  
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **V4**: ручной порядок закреплённых чатов (drag за pin-иконку, безусловно); авто-выход из любых групп; фиксы (SMS-регистрация, исчезающие чаты из-за hideChannels-гейта, умная античиталка без «Невидимки», сужение фильтров «MAX для бизнеса»).
> Изменения V4 — **UX-only**: слой нейтрализации телеметрии и приватности идентичен V3 (см. разделы ниже без изменений).

---

## 🆕 V4 — новые UX-фичи

### 📌 Ручной порядок закреплённых чатов
- Реализация: `apply_v_pinned_sort.py` (Comparator по CSV-порядку в `AppFlags.pinned_order`) + `apply_v_pinned_drag.py` (subclass `Lhk8` — ItemTouchHelper.Callback UP|DOWN, drag-handle OnTouchListener на pin-иконке `Lvs2;->w`).
- Инъекция в `r10.b` после `move-result-object p1` (sentinel `:pinned_sort_v1`).
- Данные хранятся локально в `SharedPref "app_flags"`. Ничего не отправляется на сервер.
- Фича безусловная (нет тумблера).

### 🚪 Авто-выход из групп для любых чатов
- Тумблер `autoLeaveGroupInvites` (по умолчанию OFF). Теперь без исключений по контактам.
- Реализация: enqueue в `ome.i` + drain/leave в `ou2.W`.

---

## 🐞 V4 — исправления

- **SMS-Retriever инициализация** — вынесена из-под гейта `blockMobileIdVerify` → SMS с кодом подтверждения приходят стабильно.
- **filterChatList OR-гейт** — `hideChannels` убран из общего OR-гейта и из 6 title-веток (живёт отдельно в `apply_v_hide_channels.py` через `ChatType==CHANNEL`).
- **Сужены substring-фильтры сервис-чатов**: `подтвержд`→`коды подтвержд`, `интересное`→`интересное для вас`, `бизнес`→`max для бизнеса` (lowercase-контекст).
- **filterPageList `hideBusinessPromo`** — с `contains "бизнес"` (v5, оригинал) на `toLowerCase()`+`contains "max для бизнеса"` (v7). Аналогично в `filterSettingsList`.
- **smartAntiRead scroll-fix v3** — добавлена ветка `smartAntiRead=ON, invisible=OFF` в `c2/ara.smali :sar_noninvis_gate`.

---

## 🐞 Исправлено в V3 (унаследовано)

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
| `MAX_26.26.0_arm64_V3.apk` | `298cd36aecab1f5ec7cb0108e6499a43b39aeaa3d87fed0eb6a8e974ff16d957` |
| `MAX_26.26.0_arm64_V3_clone.apk` | `0e48bc47900b54031612221a20c7142cef24bed41082af1468ddee9f46aa88c1` |
| `MAX_26.26.0_arm7_V3.apk` | `664d5824c005b72e4d73317eb2241372732d6ffbeff593dfc2069358af26fba5` |
| `MAX_26.26.0_arm7_V3_clone.apk` | `bd5919fd7eea78cb150d54631cc358b69e06ff8cc4cd1b5c937ee8a145acd6e1` |

Подпись: v2+v3 APK Signature Scheme. Cert SHA-1 идентичен V1/V2 — install -r без потери данных.

---

*Аудит выполнен статическим анализом smali (baksmali + независимый агент-верификатор). Динамическое тестирование: smoke на S25 Ultra + HA218XJZ.*
