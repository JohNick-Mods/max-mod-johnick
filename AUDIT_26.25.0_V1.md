# MAX 26.25.0 V1 — AUDIT

**Дата**: 2026-08-04 | **Версия**: 26.25.0 RS V1 | **Платформы**: arm64-v8a, armeabi-v7a

## Что изменилось в 26.25.0 V1 (δ от 26.24.0 V21)

### Переработанное меню настроек мода
- `AppFlagsView.smali` полностью перегенерирован через `gen_appflags.py`. Добавлена двухуровневая иерархия: первый уровень — меню категорий (`showCategoryMenu`), второй — диалог группы (`showCat<N>`).
- Три режима отображения, переключаются тумблером `settingsUiMode` (I, default=1): 0 — Табы, 1 — Вертикальный список, 2 — Единый плоский список. Режим персистируется в SharedPreferences `app_flags`.
- FAB-кнопка ⚙ (`showModFab:Z`, default=true): плавающая кнопка в правом нижнем углу списка чатов. Патч в `apply_v_appflags_fab.py` (sentinel `:appflags_fab_v1`).
- Системная кнопка «Назад» в под-диалогах возвращает в меню категорий (`BackToCategoriesListener` implements `DialogInterface$OnCancelListener`). Патч в `apply_v_backkey_nav.py` (sentinel `:backkey_nav_v1`). Root-диалог «Настройки MAX» listener не получает — защита от бесконечного цикла при `showVerticalList`.
- `SettingsExitRefreshListener.run()`: убран вызов `Activity.recreate()` (вызывал перезапуск активити при нажатии Back; тема применяется на лету — recreate не нужен). Sentinel `:back_exit_fix_v1`.

### PIN на настройки мода (`settingsPin`)
- Новый тумблер `settingsPin:Z` (default=false). При включении — диалог ввода PIN при каждом открытии настроек мода. PIN хранится как SHA-256 в SharedPreferences `settings_pin_hash`.
- Вспомогательные методы в `AppFlags.smali`: `isSettingsPinSet()Z`, `isSettingsUnlocked()Z`, `unlockSettingsPin()V`, `lockSettingsPin()V`. Добавлены через `apply_v_settings_pin.py` (sentinel `:settings_pin_v1`).
- `changeSettingsPinAction:Z` — тумблер смены PIN настроек.

### Родительский контроль (новая категория `parental`)
- Мастер-тумблер `parental_master:Z` + 6 ограничений: `parental_noOutgoingCalls`, `parental_noNewChats`, `parental_noForward`, `parental_hideChannels`, `parental_hidePayments`, `parental_hideVKPromo`.
- `parental_timeLock:Z` — блокировка по расписанию. Диапазон хранится как `parental_time_from:I` (HH*60+MM, default 1320=22:00) и `parental_time_to:I` (default 420=07:00). Диапазон поддерживает переход через полночь. Патч в `apply_v_parental_time_lock.py` (sentinel `:parental_time_lock_v1`).
- OR-гейты: `parental_hideChannels` работает совместно с `hideChannels`, `parental_hidePayments` — с `hidePayments`, `parental_hideVKPromo` — с `hideVKPromo`.

### Скрыть реакции (`hideReactions`)
- Тумблер `hideReactions:Z` (default=false). Полностью скрывает emoji-реакции под сообщениями. Патч в `apply_v_hide_reactions.py` (sentinel `:hide_reactions_v1`).

### Новые фильтры чатов
- `hideBotChats:Z` — скрыть чаты типа BOT.
- `hidePayments:Z` — скрыть платёжные чаты.
- `hideVKPromo:Z` — скрыть VK-промо чаты.
- `hideSferumChat:Z` — скрыть чат Сферум.
- `hideAssistant:Z` — скрыть чат Ассистент.
- `hideArchive:Z` — скрыть pinned-заголовок «Архив» из списка чатов.
- Патчи в `apply_v_hide_chat_filters.py` (sentinel `:hide_chat_filters_v1`).

### Управление вкладками навбара
- `hideTabChats:Z`, `hideTabCalls:Z`, `hideTabContacts:Z` — скрыть отдельные вкладки нижней навигации.
- Патч в `apply_v_hide_tabs.py` (sentinel `:hide_tabs_v1`).

### Анонимная статистика (`sendStats`)
- Тумблер `sendStats:Z` (default=true). При запуске отправляет анонимный ping: версия мода + страна (системная локаль). Без идентификаторов и содержимого переписки. Endpoint: Cloudflare Worker (telemetry infra).

### Антиудаление — WS live-путь (`apply_v_antidel_trash_prefix_ws.py`)
- Фикс P2: маркер 🗑️ теперь добавляется немедленно при получении события удаления по live-WS, без перезагрузки чата. Инжект в `c3/m8l.smali::e(Luv3;)Lkv3;` (WS-DTO mapper). Sentinel `:ad_trash_ws_v1`.

### R8-rebump 26.24.0 → 26.25.0
- Закрыт drift обфусцированных классов. Все 166+ apply-скриптов перерезолвлены под сток 26.25.0. Обновлён `r8_mapping.py` (LATEST sentinel).

### Исправления сборочного пайплайна
- `gen_appflags.py`: исправлен `clinit_sput` — bool-флаги с `default:true` (showModFab, amoledTheme) теперь корректно инициализируются при первом запуске без reload (sentinel `:clinit_sput_defaults_v1`).
- `apply_v_backkey_nav.py`: FIND_TAIL переключён на уникальный WeakRef-блок `showDialog` — устранено ложное совпадение с `showResetConfirm`.

## Проверки

### Сборка
- ✅ `make_v1.py --skip-anti-split --skip-baksmali` завершена без ошибок, 4 APK подписаны
- ✅ `apksigner verify` успешно для всех 4
- ✅ SHA256SUMS согласованы
- ✅ `verify_critical_patches()`: все критические sentinel присутствуют в DEX

### Smoke-тесты
- ✅ Установка main arm64 APK на S25 (`RFCX5013V5F`) — `install -r` поверх V1, без потери данных
- ✅ Запуск без краша, чистый logcat (0 `FATAL EXCEPTION` / `VerifyError`)
- ✅ FAB ⚙ отображается, открывает диалог настроек
- ✅ Системная кнопка ← Назад в главном меню настроек закрывает диалог (не зацикливается)
- ✅ Системная кнопка ← Назад в под-диалоге (категория) возвращает в меню категорий
- ✅ PIN на настройки мода — устанавливается, запрашивается при входе, сброс не ломает диалог
- ✅ Меню категорий отображается корректно во всех трёх режимах
- ✅ Кнопки темы 🌞 / 🌙 / 🖤 работают в разделе «Внешний вид»
- ✅ Тумблер «Скрыть реакции» убирает emoji-реакции
- ✅ Установка clone arm64 APK — Success

## Ассеты

- MAX_26.25.0_arm64_V1.apk
- MAX_26.25.0_arm64_V1_clone.apk
- MAX_26.25.0_arm7_V1.apk
- MAX_26.25.0_arm7_V1_clone.apk
- SHA256SUMS
- DIGITAL_ID.md
