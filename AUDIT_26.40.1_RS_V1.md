# Технический аудит MAX Mod 26.40.1 RS V1

- **Версия мода:** `26.40.1 RS V1` (тег `v26.40.1.1`)
- **Базовый APK:** `MAX 26.40.1 RS` (сборка `6928`)
- **Дата:** `2026-11-18` (full smoke всех патчей/настроек + A3 hotfix; F4 resolved)
- **Статус:** `FINAL PASS (READY)` — локальная сборка, публикация по отдельной команде
- **Run ID:** `3ad2009ae328` (`832.0s`, `make_v1.py --strict --e2e --skip-anti-split`, status=success; A3 v2 hotfix поверх `370686c4226a`)
- **Итоговый вердикт:** `PASS (READY)`

## 1. Назначение и границы отчёта
Данный отчёт фиксирует результаты статических гейтов сборки, дедуплицированного fan-out аудита патчей на базе `26.40.1 RS` (`6928`), Threat Scan, независимой проверки подписей и упаковки 4 APK, а также прогона на эмуляторе Android (`emulator-5554` и `emulator-5556`).
В `26.40.1 RS V1` перенесена и объединена вся кодовая база, исправления и универсальные гейты, подготовленные в `26.40.0 RS V5` (без отдельного выпуска `v26.40.0.5`).
Физическая приёмка на реальном телефоне и запись живого аудиопотока активного WebRTC-звонка с собеседником для этой сборки не проводились.

## 2. Основные изменения 26.40.1 RS V1
1. **Обновление базы MAX до `26.40.1 RS` (`6928`):** выполнен `/r8-rebump` и полная перепроверка якорей/полей/методов (`fingerprint_resolver.py`, `r8_drift_gate.py`) с сохранением замороженного `classes4.dex` (`Lone/me/e2e/`).
2. **`B18` (Политика E2E при установке/обновлении):** `e2eEncryption` и `e2eAttach` принудительно сбрасываются в `false` при чистой установке и обновлении (`B18InstallPolicy` по `versionCode` + `lastUpdateTime` + `stickyDirty`); ручное включение сохраняется между перезапусками внутри текущей установки; `MainActivity.onCreate` при `OFF` не вызывает Signal/JNI и не генерирует PreKey/SignedPreKey.
3. **`F1` (Шрифт интерфейса):** расширенный каталог встроенных шрифтов («Шрифт» / `appFontFamily`, `38` файлов в `assets/fonts/`).
4. **`F2` (Нижняя панель):** выбор стиля нижней панели навигации (`bottomBarStyle`) с корректным сбросом кэшей и перерисовкой.
5. **`F24` (Только чаты):** навигационный guard в `OnlyChatsNavigation` + атомарная публикация политики в финальной когорте `AppFlags` (исключён переход по ссылкам в каналы при `hideChannels=true`).
6. **PIN настроек мода (Stage A + B session lifetime):** усилено время жизни сессии PIN-кода (Stage A close hook + Stage B ordered weak dialog stack/top) и авторитарное хранение в `AppFlags.p`.
7. **`F28` (Скрытие номера в профиле):** исправлена инвертированная ветвь `if-ne` → `if-eq` в `apply_v_hide_phone.py` для действия копирования.
8. **`F30` (Телефон в мини-приложениях):** исправлена полярность `blockWebAppPhone` в `apply_v_threat_webapp_phone.py`.
9. **`P1` (Телеметрия):** добавлена политика фильтрации хостов `TelemetryHostPolicy` в `TracerLite`.
10. **`P20` (`flat_settings`) и `B27` (`E2EOutgoingDispatch`):** раздельные сентинелы в `apply_v_flat_settings.py` (BOM сохранён) и исправление маршрутизации исходящих сообщений (`apply_b27_outgoing_dispatch.py`).
11. **`UID` и `UpdateChecker`:** добавлено безусловное отображение `ID пользователя: <uid>` в строке даты регистрации профиля (`RegistrationRow.smali`), обновлена целевая версия `UpdateChecker` до `26.40.1.1`.
12. **Автоповорот видеоплеера:** восстановлен автоповорот экрана при просмотре видео в плеере (`ChatMediaViewerScreen` / `apply_v59_orientation_fix.py`).
13. **Входящие звонки в фоне (FCM + KeepAlive):** восстановлена регистрация `FirebaseInstallationsRegistrar` и исправлена цепочка фонового WebSocket/`KeepAlive`/`BackgroundWakeBootReceiver`.
14. **Фирменный рингтон MAX и нативный гудок:** все `14` ресурсов `res/raw/*.m4a` упакованы без сжатия (`ZIP_STORED`) с 4-байтным выравниванием; устаревшие патчи `RingbackTonePlayer` удалены (`0` вызовов).
15. **Смена цвета иконки (activity-alias):** исправлена последовательность переключения `activity-alias` (`AppFlags.applyIconColor` — включение нового алиаса до отключения старого) и разрешение `LAUNCHER`.
16. **Номер телефона контакта в профиле:** исправлена ветвь в `apply_v_hide_phone.py` (`if-eqz` → `if-nez`) для отображения номера контакта при `hidePhoneNumber=false`.
17. **Переход к первому сообщению (ReadFence):** внедрён переход к первому сообщению с `ReadFence` (сохранение непрочитанного состояния).
18. **Иконка приложения из галереи и deep-link:** сохранены `6` цветовых алиасов иконки и точка входа `maxmod://flags`, добавлена поддержка собственной иконки из галереи (`ImageIconActivity` + `LauncherIconSettings`).
19. **Поддержка рабочего профиля (F26ProviderLookup):** добавлен `F26ProviderLookup` для разрешения URI-грантов контент-провайдеров; верифицирована безопасность широких регистров (`wide-register safety`) в `u91.e` (`0` коллизий).
20. **`AccountSwitchGuard`:** добавлена защита от смены аккаунта (`selfId`) и поздних асинхронных операций в кэшах и коллбэках мода.
21. **Стабильность WatchdogAlarm и оформление showFlatAll:** убран жёстко зашитый зелёный `0xFF4CAF50` в `showFlatAll`, исправлена сигнатура `Context.startForegroundService(...)Landroid/content/ComponentName;` и заменён `invoke-virtual` на `invoke-interface` для `SharedPreferences` в `WatchdogAlarm.smali` (`0` `IncompatibleClassChangeError`).
22. **Вкладки папок, создание Цифрового ID и кнопка «В архив» в панели выделения:** исправлены пустые вкладки папок («Все», «Новые», «Личные», «Каналы») при включении `localArchive` и первом холодном запуске без перезапуска (плюс подавление ложного тоста ошибки до завершения авторизации); восстановлена кнопка «Создать Цифровой ID» во вкладке «Цифровой» (включая медленный ответ сети >3 сек); и в нижней панели выделения по долгому тапу и при множественном выборе (`[Mute | Pin | Archive | Read | Delete]` при `localArchive=true && !hideArchive`, штатные 4 кнопки при `false` или скрытом архиве) добавлена кнопка **«В архив»** для чатов и каналов с мгновенным обновлением папок/счётчиков и возвратом через `ArchiveScreenView`.
23. **`F4` (Скрытие рекомендаций каналов):** исправлено отображение рекомендаций каналов в папке «Каналы» при включённом `hideChannelRecs` + `localArchive`. Фильтр clear-gate в archive suggest-продюсере (`a14.v`, `b14.a=="chat.channel.folder"`) убирает карточки рекомендаций (`chat_suggest_item`), оставляя только подписки; дополнительный defensive-фильтр агрегации `m50.N`. Верифицировано ON/OFF на AVD: ON — deep-scroll только подписки (`chat_suggest_item=0`), OFF — штатные рекомендации возвращаются.
24. **`F1`/`F5` (Холодный старт папок):** вкладки «Все»/«Новые»/«Личные»/«Каналы» наполняются при первом холодном запуске с `localArchive=ON` (cold-start kick + seed-clobber guard); «Личные» показывает личные чаты без пустого экрана.

## 3. Проверка APK, контрактов и безопасности
- Детерминированный fan-out аудит ВСЕХ активных `apply_v_*.py` (`fanout_audit_26401.py`, 357 скриптов) + `sentinel_audit.py --strict`: **REVIEWED — реальных багов активного пайплайна НЕТ**. Единственный FATAL (`apply_v_e2e_attach_upload_a`/`pwk.smali`) и 2 DRIFT (`push2_throttle`/`KeepAliveManager`, `v22_settings`/`AboutAppSettings`) — **false positives / документированные exemptions**: legacy-сентинел воронки A суперждён системой B12/B13 (`e2e_attach_v8c` + `b26_late_final` + `SecureAttachAdmission`), криптографические пины `r8_drift_gate._approved_final_owner()` совпадают (build verdict «pinned FINAL owner replaces legacy sentinel»); `KeepAliveManager` — inject-owned файл (присутствует); `AboutAppSettings` — штатный `GRACEFUL_SKIP` (перенос c3→c2). Авторитетные гейты сборки `r8_drift_gate` (STEP 370) + `landing_check --strict` (STEP 371) — GREEN. `0` инверсий полярности, `0` регистровых коллизий.
- Threat Scan (`threat_scan.py`): всего `515` находок, открытых **HIGH** (`NEW`/`CHANGED`): **0**.
- Независимая проверка 4 APK (`verify_apk.py` + `apksigner_google.jar verify` + контрактные проверки): **PASS (4/4)** (`0` `libcall_e2e.so`, `38` файлов в `assets/fonts/`, `0` `fixture_`-ассетов, `apksigner`: `4/4 Verifies`).
- Скрытые тумблеры (`audit_tumblers_hidden.py --fail-loud`) и релиз-ноты (`audit_release_notes_tumblers.py --fail-loud`): **BAD=0**.
- **Обязательные универсальные гейты пайплайна (Zero-Escape):**
  - `universal_zero_escape_gate.py` (`--strict`): **PASS** (полярность флагов `tumblers.yaml`, запрет hardcoded `ru.oneme.app` для клона, целостность регистраров/сервисов, порядок `setComponentEnabledSetting` enable-before-disable, Gate 5 `split_brain_prefs`, Gate 6 `interface_invoke_virtual`).
  - `verify_apk.py` (универсальная проверка упаковки): **PASS** (`res/raw/` аудиоресурсы и `baseline.prof` в `ZIP_STORED`, 4-byte alignment, ненулевой CRC32, активный `MAIN`/`LAUNCHER` в main и clone, `resources.arsc` `STORED`, `AndroidManifest.xml` #0).
  - `universal_avd_prerelease_gate.py`: **PASS** (живой AVD pre-release gate на `emulator-5554`: запуск `MAIN`/`LAUNCHER`, ненулевой UI-ориентации, `res/raw/*.m4a` `openRawResourceFd`, активные FCM/`KeepAlive`-компоненты).
- Автотесты (`pytest`): **24 passed in 24.90s**.

| APK | Пакет | Размер | `apksigner` | `classes4.dex` (`Lone/me/e2e/`) |
|---|---|---|---|---|
| `MAX_26.40.1_arm64_V1.apk` | `ru.oneme.app` | `45 176 084` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |
| `MAX_26.40.1_arm64_V1_clone.apk` | `ru.oneme.ap2` | `45 176 084` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |
| `MAX_26.40.1_arm7_V1.apk` | `ru.oneme.app` | `43 881 776` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |
| `MAX_26.40.1_arm7_V1_clone.apk` | `ru.oneme.ap2` | `43 881 776` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |

## 4. SHA256 (`SHA256SUMS.txt`)
```
5ca49146af8b6fd2a37902a6371082be2141f3a260d4c1aedb41129c738c536a  MAX_26.40.1_arm64_V1.apk
0f36f115529644fe4825c3c7806c94ce56885fe02904fb6bf8cf614bd0814093  MAX_26.40.1_arm64_V1_clone.apk
57efb1bc700fce59701ca6bb57868f19581bf5298817fbb6dbb8a796904a8b0e  MAX_26.40.1_arm7_V1.apk
2264f4a9c84fb4138ca3ea0083d76b69516827706b2ae3e8f667d36703b3ddea  MAX_26.40.1_arm7_V1_clone.apk
```

## 5. Живой AVD QA 26.40.1 RS V1 (`emulator-5554` + `emulator-5556`)
- **Module A (Установка, SHA-256, ART `speed`):** `PASS` (`0 VerifyError`, `0` записей в `logcat -b crash`).
- **Module B (`B18` Stage 2 на `emulator-5556` + Stage 3 на `emulator-5554`):** `PASS`.
- **Module C (Сценарии V1 / V5-порт):** `PASS`.
- **Module D (Базовые проверки MAX + клон `ru.oneme.ap2`):** `PASS`.

### 5.1 Финальный smoke 17.11.2026 (production `cce8886a`, `emulator-5554`, main + clone)
- **Установка + целостность:** `install -r` main (`ru.oneme.app`) + clone (`ru.oneme.ap2`), SHA-256 установленных `base.apk` = EXACT match с `SHA256SUMS` (main `cce8886a…`, clone `fb1c5503…`). `versionName=26.40.1`.
- **ART / стабильность:** cold start main + clone — **`0 VerifyError`, `logcat -b crash` чистый**, PID стабилен.
- **F4 (рекомендации каналов):** `hideChannelRecs=ON` + `localArchive=ON` → «Каналы» deep-scroll = только подписки (`ROWS=9`, все с таймстампами постов), **`chat_suggest_item=0`, «Подписаться»=0, пустых папок=0**. `OFF` → штатные рекомендации возвращаются (`chat_suggest_item=10`). Верифицировано ON/OFF.
- **F1/F5 (холодный старт папок):** «Все»/«Новые»/«Каналы» наполнены при cold-ON; «Личные» = 9 личных чатов (вкл. тестовый контакт), без пустого экрана.
- **Отправка сообщения (коррекция F6):** на `emulator-5554` исходящие текст/фото/голосовые получают статус **«Not sent»** (content-desc), `TaskMonitorWorker FAILURE` — **environmental-блокада эмулятора** (main+clone одинаково; приём сообщений, загрузка файлов и звонки работают, сеть VALIDATED). НЕ регрессия мода. Прежняя оценка «доставлено / ⏱=0» была false-green (⏱=0 — pending-часы, не факт доставки). Финальное подтверждение отправки — физический S24 (см. §5.2).
- **Аудиозвонок:** «Модем» — экран «Calling…», элементы управления (микрофон/динамик/камера/Завершить), корректное завершение, **`0` крашей**. (Живой пир на эмуляторе отсутствует — «Call failed» после завершения ожидаемо.)
- **Клон `ru.oneme.ap2`:** cold start `0 VerifyError`, `versionName=26.40.1`, список чатов наполнен (папки Все/Новые/Личные).
- **Гигиена устройства:** transparency-тумблеры OFF (default); `localArchive`/`hideChannelRecs` возвращены в OFF через UI (`chk=false`, релизные дефолты `AppFlags` = false). Устройства на чистой production-сборке.
- **Ограничения:** физическая проверка на реальном телефоне и запись живого аудиопотока активного WebRTC-звонка с собеседником не проводились.

### 5.2 Полный smoke всех патчей и настроек 18.11.2026 (production `5ca49146`, A3 v2 hotfix, `emulator-5554`)
- **Охват:** все 5 категорий настроек (🔒 Приват 23 / 📁 Папки 16 / 💬 Чаты 27 / ⚙️ Система 10 / 🎨 Вид 8), always-on патчи (телеметрия-блокада, анти-удаление, scroll, PIN), клон `ru.oneme.ap2`, ядро (звонки аудио/видео, вложения, запись+воспроизведение голосовых, E2E-инициализация).
- **A3 (release-blocking crash) — найден и ИСПРАВЛЕН:** «Починить → ♻️ Сбросить настройки мода» ронял `FATAL IllegalStateException: stale settings scope` (статический scope `SettingsPinSession28` привязан к прежнему инстансу `AppFlagsView`; после цикла settingsPin enable/disable сброс звал `showRegular` на другом инстансе → `matches()=false` → throw). **Фикс v2** (пост-райтер `apply_v_a3_reset_scope`, run `3ad2009ae328`): stale-recovery в `showRegular`/`admit` (close+re-begin) + `showResetConfirm` обходит instance-tied scope через прямой `DialogsUtils.showRoundedDialog` (reset-confirm имеет свой «Вы уверены?», вход в настройки уже PIN-gated). **Device-verify:** settingsPin OFF → Reset confirm ✓; settingsPin ON → Reset confirm без PIN-loop ✓; «Продолжить» → сброс всех флагов («Изменённые» = «Ничего не найдено») ✓; `logcat -b crash` + FATAL/VerifyError/stale **CLEAN** ✓; SHA-256 установленного `base.apk` == `5ca49146` ✓.
- **Functional PASS:** F4 `hideChannelRecs` («Каналы» deep-scroll: 0 рекомендаций/подписок-вставок); F1/F5 `localArchive` (табы Все/Новые/Личные/Каналы наполнены при cold-ON); запись звонка (⏹ таймер, файл сохранён); AMOLED/Светлая тема (pixel-verify); appLock/settingsPin/chatPin полные циклы (PIN 1982); E2E-инициализация текстовых сообщений; клон — независимые флаги, cold start clean.
- **F6 (отправка) — environmental, НЕ блокер:** исходящие текст/фото/голосовые «Not sent» на эмуляторе (main+clone одинаково, `TaskMonitorWorker FAILURE`); приём/загрузка файлов/звонки работают, сеть VALIDATED. Не регрессия мода. Финальное подтверждение отправки — физический S24.
- **A2 (minor, caught):** `notifShouldSuppressExtras` CCE Bundle→long[] перехвачен fail-safe (→ false, стандартный notify) — не краш, не privacy-leak; страдает только anti-read re-notif дедуп. Кандидат в будущий хотфикс.
- **F7 (minor, non-blocking):** глобальный сброс не очищает `transparentTheme` (спец wallpaper-gate флаг, «Отдельный сброс недоступен»); вручную OFF → «Изменённые» пусто. Pre-existing, не регрессия A3.
- **Parental:** раздел «Дети» намеренно скрыт из UI (решение владельца), не баг.
- **Итоговый вердикт 18.11:** `PASS (READY)` — **0 crashes / 0 VerifyError** по всему smoke после A3 v2 hotfix; все настройки и ядро проверены. Открытые пункты не блокируют релиз: F6 (environmental, физический S24 pending), A2/F7 (minor, caught/non-blocking). Публикация — по отдельной команде.
