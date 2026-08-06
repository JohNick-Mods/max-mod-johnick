# MAX 26.25.0 V2 — AUDIT

**Дата**: 2026-08-06 | **Версия**: 26.25.0 RS V2 | **Платформы**: arm64-v8a, armeabi-v7a

## Что изменилось в 26.25.0 V2 (δ от V1)

### Исправление уведомлений при античиталке (#NOTIF)
- `notifShouldSuppress` вызывался из двух независимых путей (FCM + показ), при этом имел side-effect: первый вызов (FCM) ставил флаг «suppressed», второй вызов (mfb.a — реальный показ) видел «уже подавлено» → тишина.
- Фикс: peek/commit split. Новый метод `notifPeekSuppressed(J)Z` (только `containsKey`, без `put`). FCM-гейт переведён на него (sentinel v1→v2), commit-запись только в `mfb.a`. Патч в `gen_appflags.py`.

### Расширение антиудаления (#ANTIDEL)
- V1 охватывал только `c1/tha` (1 сайт, live-delete через L3). Reload/scroll шёл через display-pager `c3/rha` и `c3/aha` — там DELETED не фильтровалось.
- Фикс: новая THA-форма с lookahead-защитой от V5 `delivery_status` false-positive. TARGETS расширен: `c1/tha`(1) + `c3/rha`(4) + `c3/aha`(1) = 6 сайтов read-layer. Патч в `apply_v22_antidel_persist.py`.

### Расписание родительского контроля (#SCHED)
- `checkParentalTimeGate` проверял `parental_master`-тумблер → если родительский контроль выключен, расписание тоже не срабатывало.
- Фикс: убран `parental_master`-гейт из `checkParentalTimeGate`. Расписание работает автономно.

### Скрытие каналов — полный охват (#CHANNELS)
- V1: каналы скрывались только из основного списка чатов.
- V2: скрытие распространено на все поверхности:
  - Глобальный поиск — gate в `hh3.invokeSuspend` (`apply_v_search_hide_channels.py`, sentinel `:search_hide_channels_v1`).
  - Заголовок «Каналы» в поиске — gate по названию в `hh3.invokeSuspend` (`apply_v_search_hide_channels_title.py`).
  - Строки результатов поиска — type-gate в `Lq10;->b()` (`apply_v_search_hide_channels_rows.py`).
  - Чип «Каналы» в панели категорий — chokepoint `Lgv4.<init>()` + runtime fix `gv4`/`nc5` (`apply_v_search_hide_channels_chip.py`, `_chip2.py`, `fix_gv4_channels_chip_runtime.py`, `fix_nc5_channels_chip_gate.py`).
  - Серверная карточка «Каналы» в search-showcase — gate в `gg3` (`apply_v_search_showcase_hide_channels.py`).
  - Папки / вкладка «Новые» — `Lk1a` filterChatList (`apply_v_filter_chatlist_k1a.py`).
  - Nuclear path: удаление CHANNEL из `Lgy6->e` в `reload()` (`fix_channels_nuclear.py`, sentinel `:channels_nuclear_v1`).
  - Синхронизация тумблеров: `parental_hideChannels` ↔ `hideChannels` читают/пишут единый ключ.

### Фикс скролла при невидимке+античиталке
- При накопленных непрочитанных и активной невидимке/античиталке сервер шлёт `event.g = first_unread_msgId (> 0)`. V1-гейт `g == -1` пропускал это → чат открывался на первом сообщении.
- Фикс: при `invisible=ON` всегда scroll вниз без проверки `event.g` (`apply_v_smart_antiread_scroll_v2.py`, sentinel `:sar_invis_v2`).

### Сервисные чаты не скрывались (#SERVICECHATS)
- В V1 `apply_v_hide_service_chats.py` выполнялся ДО `apply_v22_settings.py`, который перезаписывал `AppFlags.smali` из `smali_inject_v22` и стирал все изменения.
- Фикс: `apply_v_hide_service_chats.py`, `apply_v_hide_archive.py`, `apply_v_filter_chatlist_k1a.py` перенесены ПОСЛЕ `apply_v22_settings.py` в `make_v1.py`.

### Секунды в метках времени сообщений
- Патч `apply_v_message_time_seconds.py`: `w59.y()` форкнут с `HH:mm → HH:mm:ss`. Sentinel `:msg_time_seconds_v1`.

### Исправление noForward (родительский контроль)
- Старый якорь в `c0a.smali` был ошибочным (метод загружает участников чата, а не обрабатывает пересылку).
- Фикс: патч переключён на `odd.smali` — реальный роутер навигации. При включённом `parental_noForward` блокируется открытие `ForwardPickerScreen` (`goto :goto_3a6`). Sentinel `:parental_fwd_v3_odd`.

### Очистка ложных патчей каналов
- `apply_v_remove_false_search_channel_patches.py`: удалены ложные вхождения в `yp7`/`dy6`/`ml0`, которые применялись в пустоту (smali-файлы не участвуют в pipeline каналов).

### PUSH2-throttle fix
- Убран невалидный nested try/catch (`.catch Ljava/lang/Throwable;`) из throttle-блока `apply_v_push2_throttle.py`. Все операции в блоке pure (нет Throwable), nested catch вызывал потенциальный verifier-reject.

## Что изменилось в 26.25.0 V1 (δ от 26.24.0 V21)

### Переработанное меню настроек мода
- `AppFlagsView.smali` полностью перегенерирован через `gen_appflags.py`. Добавлена двухуровневая иерархия: первый уровень — меню категорий (`showCategoryMenu`), второй — диалог группы (`showCat<N>`).
- Три режима отображения, переключаются тумблером `settingsUiMode` (I, default=1): 0 — Табы, 1 — Вертикальный список, 2 — Единый плоский список. Режим персистируется в SharedPreferences `app_flags`.
- FAB-кнопка ⚙ (`showModFab:Z`, default=true): плавающая кнопка в правом нижнем углу списка чатов. Патч в `apply_v_mod_fab.py` (sentinel `:appflags_fab_v1`).
- Системная кнопка «Назад» в под-диалогах возвращает в меню категорий (`BackToCategoriesListener` implements `DialogInterface$OnCancelListener`). Патч в `apply_v_backkey_nav.py` (sentinel `:backkey_nav_v1`).

### PIN на настройки мода (`settingsPin`)
- Новый тумблер `settingsPin:Z` (default=false). При включении — диалог ввода PIN при каждом открытии настроек мода. PIN хранится как SHA-256 в SharedPreferences `settings_pin_hash`.
- Вспомогательные методы в `AppFlags.smali`: `isSettingsPinSet()Z`, `isSettingsUnlocked()Z`, `unlockSettingsPin()V`, `lockSettingsPin()V`.

### Родительский контроль (новая категория `parental`)
- Мастер-тумблер `parental_master:Z` + 6 ограничений: `parental_noOutgoingCalls`, `parental_noNewChats`, `parental_noForward`, `parental_hideChannels`, `parental_hidePayments`, `parental_hideVKPromo`.
- `parental_timeLock:Z` — блокировка по расписанию. Диапазон: `parental_time_from:I` (default 1320=22:00) и `parental_time_to:I` (default 420=07:00). Поддерживает переход через полночь.

### Скрыть реакции (`hideReactions`)
- Тумблер `hideReactions:Z` (default=false). Патч в `apply_v_hide_reactions.py` (sentinel `:hide_reactions_v1`).

### Новые фильтры чатов
- `hideBotChats:Z`, `hidePayments:Z`, `hideVKPromo:Z`, `hideSferumChat:Z`, `hideAssistant:Z`, `hideArchive:Z`.

### Управление вкладками навбара
- `hideTabChats:Z`, `hideTabCalls:Z`, `hideTabContacts:Z`.

### Анонимная статистика (`sendStats`)
- Тумблер `sendStats:Z` (default=true). Анонимный ping: версия мода + страна. Endpoint: Cloudflare Worker.

### R8-rebump 26.24.0 → 26.25.0
- Все 166+ apply-скриптов перерезолвлены под сток 26.25.0. Обновлён `r8_mapping.py`.

## Проверки

### Сборка
- ✅ `make_v1.py --skip-anti-split --skip-baksmali` завершена без ошибок, 4 APK подписаны
- ✅ `apksigner verify` успешно для всех 4
- ✅ SHA256SUMS согласованы
- ✅ `verify_critical_patches()`: все критические sentinel присутствуют в DEX

### Smoke-тесты
- ✅ S25 (`RFCX5013V5F`) — `install -r` main + clone arm64, 0 FATAL/VerifyError
- ✅ HA218XJZ (Lenovo планшет) — `install -r` main + clone arm64, 0 FATAL/VerifyError
- ✅ Уведомления приходят при активной античиталке
- ✅ Антиудаление держит сообщения при scroll/reload
- ✅ Расписание блокирует вход независимо от мастер-тумблера
- ✅ «Только чаты»: каналы не видны в списке, поиске, папках и вкладке «Новые»
- ✅ FAB ⚙ отображается, открывает диалог настроек
- ✅ Системная ← Назад возвращает в меню категорий

## Ассеты

- MAX_26.25.0_arm64_V2.apk
- MAX_26.25.0_arm64_V2_clone.apk
- MAX_26.25.0_arm7_V2.apk
- MAX_26.25.0_arm7_V2_clone.apk
- SHA256SUMS
- DIGITAL_ID.md
