# MAX 26.26.0 V2 — Технический аудит мода

**Версия стока**: 26.26.0  
**Версия мода**: V2 (Privacy Mod by JohNick)  
**Дата**: 2026-08-10  
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **V2 hotfix**: адаптивные цвета UI настроек для светлой темы — все кнопки и фон диалога теперь следуют теме мода (единый источник истины), а не системной теме. Добавлен автосинк темы мода с системной при первом запуске.

---

## 🐞 Исправлено в V2

### Кнопки настроек оставались тёмными на светлой теме

**Корень проблемы — две независимые точки управления темой:**

1. Фон диалога настроек читал системную `Configuration.uiMode` через `DialogsUtils.isNightMode()`.
2. Цвета кнопок категорий (`AppFlagsView`) читали флаг мода `AppFlags.amoledTheme`.

При «светлой» системной теме + включённом тёмном моде: фон диалога — белый (системная тема), кнопки — тёмные `#303030` (флаг мода) → белый фон с тёмными кнопками.

**Дополнительная причина — регистрозависимый поиск (silent no-op):**  
`apply_v_light_theme.py` искал константу `-0xcfcfd0` (нижний регистр), а `gen_appflags.py` генерирует `-0xCFCFD0` (верхний регистр). `str.replace` регистрозависим → 13 замен никогда не выполнялись, скрипт при этом возвращал «OK» без ошибки.

**Фиксы:**

- `isDarkThemeModa(Context)Z` — новый статический метод в `DialogsUtils`, единственная точка чтения темы: `sget-boolean AppFlags->amoledTheme:Z`. Возвращает флаг мода напрямую, без обращения к системной конфигурации.
- `getDialogTheme()` и `roundAndTint()` в `DialogsUtils` перенаправлены с `isNightMode` на `isDarkThemeModa` → фон диалога и тексты следуют флагу мода.
- `apply_v_light_theme.py`: поиск `-0xCFCFD0` заменён на `re.IGNORECASE` + guard: при 0 замен сборка падает с `ERROR` вместо молчаливого возврата.
- `getCategoryBg(Context)I` / `getCategoryTextColor(Context)I`: светлая тема → `#E0E0E0` / `#212121`; тёмная/AMOLED → `#444444` / `#FFFFFF`.
- Охват: **39 вызовов** `getCategoryBg` и `getCategoryTextColor` (было 26). Добавлены кнопки «Выбрать обои», «Сбросить обои», переключатели ☀️/🌙/🖤, инфо-кнопки, «🔧 Починить» — во всех 3 режимах (Категории / Вертикальный список / Плоский список).
- Brown 800 (`-0xa2bfc9`): 0 вхождений в APK.

### Автосинк темы мода с системной (one-time)

Новый метод `syncNightModeWithSystem(Context)V` в `DialogsUtils`, вызывается из `MainActivity.onCreate()` один раз.

**Логика:**
1. Читает `SharedPreferences("user.prefs")` — флаг `modThemeSyncedV2`.
2. Если флаг уже `true` — выход (выбор пользователя неприкосновенен).
3. Читает `Configuration.uiMode & 0x30` — `0x20` = `UI_MODE_NIGHT_YES` (тёмная система).
4. Пишет `AppFlags.set("amoledTheme", isSysDark)` + `AppFlags.set("amoledPure", false)`.
5. Ставит `modThemeSyncedV2 = true` → при следующих запусках блок пропускается.

Весь блок обёрнут в `try/catch Throwable` — при ошибке (null SharedPreferences, отсутствие ресурсов) синк молча пропускается, мод работает в штатном режиме.

**Регистровый аудит `syncNightModeWithSystem`:** `.registers 6`, P=1 (static, 1 аргумент). Параметры: p0 = v5. Scratch: v0–v4. Коллизий нет.

---

## 🔍 Результаты независимой верификации (агент)

| Проверка | Результат |
|---|---|
| Хардкод `-0xCFCFD0` во всех 3 dex | **0 вхождений** |
| Хардкод `-0xa2bfc9` (Brown 800) | **0 вхождений** |
| `getCategoryBg` / `getCategoryTextColor` | **39 / 39** — симметрично |
| `isDarkThemeModa` читает только `AppFlags->amoledTheme:Z` | ✅ |
| `getDialogTheme` + `roundAndTint` → `isDarkThemeModa` | ✅ |
| Вызовов `isNightMode` (за пределами объявления) | **0** |
| Register-collision (10 методов AppFlagsView) | **CLEAN** |

**Smoke test:** S25 Ultra (RFCX5013V5F, SM-S9210, Android 15), install -r поверх V1 — данные сохранены. Светлая тема — все кнопки светлые. Тёмная / AMOLED — без изменений.

---

## 🛡 Профиль приватности (без изменений с V1)

### Невидимка (`invisible`) — мастер-переключатель

| Дочерний флаг | Что делает |
|---|---|
| `antiTyping` | собеседник не видит набор текста |
| `antiRead` | собеседник не видит отметку «прочитано» |
| `smartAntiRead` | «прочитано» уходит только после вашего ответа |
| `antiPresence` | онлайн / last-seen скрыты (сервер видит offline) |

### Антиудаление (`antiDelete`) — 4 слоя

1. **L3** — Central WS chat-event (live-канал).
2. **L4** — Room ORM body-clear (локальный SQL-сброс).
3. **Маркер** «· удалено» + время удаления.
4. **FCM офлайн-персист** — удаления, пришедшие при убитом приложении, восстанавливаются при следующем старте.

### Нейтрализация телеметрии — 4 новых чокпоинта (V1)

`p0_telem_push` (push-трекеры), `p0_telem_onelog` (one-log агрегатор), `p0_telem_operator` (оператор-фингерпринт / IP-strip), `p0_telem_dps` (DPS-трекер).

---

## 📋 Переключатели — UI настроек

Все по умолчанию **OFF** — до первого включения мод ведёт себя идентично стоку.

**Оформление:** `redCode`, `chatWallpaper`, `amoledDividersBlack`, `wallpaperGreenmax`.  
**Навигация:** `troubleshoot`, `submenu_back`, `submenu_back_onkey`, `back_cycle_guard`.  
**FAB:** `showModFab` (drag-and-drop, persist позиции).  
**Приватность:** `antiDelete`, `invisible` (+4 дочерних), `blockMobileIdVerify`, `hidePhoneNumber`, `hideDigitalId`.  
**Папки:** `hideAll`, `hideSferum`, `hideFavorites`, `hideBusinessPromo`, `hideInviteFriends`, `reorderTabs`, `startOnNew`.  
**Сервисные чаты:** `hideVerifyCodes`, `hideMaxOfficial`, `hideInteresting`, `hideUsefulNotifs`, `hideMaxBusiness`, `hidePromoWidgets`, `hideInviteLinks`, `hideChannels`.  
**Система:** `autoStart`, `backCameraForNote`, `debugMenu`, `gateOpcode5`.

---

## 📦 Артефакты

| Файл | SHA-256 |
|---|---|
| `MAX_26.26.0_arm64_V2.apk` | см. SHA256SUMS |
| `MAX_26.26.0_arm64_V2_clone.apk` | см. SHA256SUMS |
| `MAX_26.26.0_arm7_V2.apk` | см. SHA256SUMS |
| `MAX_26.26.0_arm7_V2_clone.apk` | см. SHA256SUMS |

Подпись: v2+v3 APK Signature Scheme. Cert идентичен V1 — install -r без потери данных.

---

*Аудит выполнен статическим анализом smali (baksmali + независимый агент-верификатор). Динамическое тестирование: smoke на реальном устройстве S25 Ultra.*
