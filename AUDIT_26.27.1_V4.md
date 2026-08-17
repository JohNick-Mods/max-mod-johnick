# MAX 26.27.1 V4 — Технический аудит мода

**Версия стока**: 26.27.1 (build 6802)
**Версия мода**: V4 (Privacy Mod by JohNick)
**Дата**: 2026-08-17
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **V4**: точечный релиз поверх V3. Закрыта дыра приватности в PIN-защите чатов (содержимое было видно за диалогом), починен persist порядка закреплённых чатов после перетаскивания, добавлен тумблер `bgWatchdog` против бага «Подключение…» на MIUI/HyperOS. Весь функционал V3 сохранён.

---

## 🆕 26.27.1 V4 — что исправлено

### 🔒 PIN-защита чата: содержимое больше не видно за диалогом
До V4 при открытии защищённого чата над ним появлялся PIN-диалог, но переписка, история звонков и вложения полностью читались за полупрозрачным окном — блокировался только ввод. Реальная дыра приватности.

Фикс: перед показом PIN-диалога вызывается `ChatPinManager.hideView(View)` — root-View чата переводится в `View.INVISIBLE` (0x4), ссылка сохраняется в static `WeakReference` (без утечки Activity). Возврат видимости `revealView()` вызывается **только на пути верного PIN** в `ChatPinManager$OkClick.onClick`. Путь неверного PIN — `dropHidden()` (обнуление ссылки без показа) + `finishActivity()` — чат не «мигает» перед закрытием. Аварийный путь `abortHidden()` в catch-блоке гейта: если сбой произошёл ПОСЛЕ скрытия, чат закрывается (не показывается без PIN); если ДО — no-op (чат работает как в стоке).

Дополнительно: при первичной установке PIN добавлена проверка минимум 4 цифр — раньше случайное нажатие OK с пустым полем молча сохраняло хэш пустой строки как PIN.

### 📌 Порядок закреплённых чатов сохраняется после перетаскивания
До V4 закреплённый чат можно было перетащить, но `PinDragCb.n()` вызывался сотни раз за drag и ни разу не доходил до persist — после переключения списка чаты возвращались на прежние места.

Причина: во второй проверке типа ViewHolder стояло `if-eqz v1, :pd_ok_a` вместо `if-nez v1, :pd_ok_a`. При корректном ViewHolder (`instance-of` → true, v1=1) прыжок не срабатывал, и метод падал в bad-ветку с `return v0`. То есть валидный VH всегда уходил в «отклонить». Ошибка была в первой ревизии метода и пережила несколько правок, потому что искали R8-дрейф VH-класса, а не логику ветвлений.

Фикс: инверсия `if-eqz → if-nez`. Порядок закреплённых теперь корректно сохраняется в локальном хранилище (`AppFlags.putString("pinned_order", csv)`) и восстанавливается при переоткрытии.

### 📶 Тумблер «Удерживать связь в фоне» (`bgWatchdog`)
Настройки → Звонки и система → **«Удерживать связь в фоне»**. По умолчанию ВКЛЮЧЁН (текущее поведение мода).

Симптом (жалобы пользователей на MIUI/HyperOS): в шапке чата вечно висит статус «Подключение…», хотя сообщения приходят. Гипотеза: MIUI агрессивно убивает foreground-службу связи, watchdog мода её тут же поднимает (AlarmManager + WorkManager 15 мин + restart на `onTaskRemoved` + `NetworkChangeCallback`), соединение циклически передёргивается и не успевает дойти до Connected. Без MIUI-устройства воспроизвести нельзя — тумблер даёт пользователю escape hatch.

Реализация: два гейта в `Lone/me/util/KeepAliveManager;` (единственный файл-цель):
- `ensureServiceRunning(Context)V` — воронка ВСЕХ watchdog-путей (KeepAliveWorker, NetworkChangeCallback, WsFallbackHelper) → `sget-boolean bgWatchdog:Z` + `if-eqz :throttled`.
- `enable(Context)V` — не планировать WorkManager/AlarmManager при OFF → `sget-boolean bgWatchdog:Z` + `if-eqz :catch_alm_en`.

`disable()`/cancel НЕ гейтится — отмена задач должна работать всегда, иначе при выключении тумблера ранее запланированные задачи остались бы висеть. При выключении работа связи возвращается к стоковой (уведомления могут приходить с задержкой).

---

## 🆕 Унаследовано из 26.27.1 V3 (без изменений)

### 🔑 Chat-PIN — тумблер виден в настройках
Флаг `hidden` снят в V3, тумблер «🔒 PIN на отдельные чаты» рендерится в категории Приватность.

### 📌 Drag: `PinHandleTouch.onTouch` потребляет ACTION_DOWN
Возврат `true` после `pinStartDrag` — жест не перехватывается родительским `ViewPager2`/`RecyclerView` (V3-фикс входа в drag; V4-фикс — persist результата drag, см. выше).

## 🆕 Унаследовано из 26.27.1 V2 (без изменений)
- 🔔 Гудок дозвона: `stop()` перед `Connection.destroy()` в `qe1` + в `h75.onCallAccepted()`.
- 📜 Прокрутка чата под невидимкой/античиталкой: якоря `bta`/`Lv2f;->g:J` актуализированы.
- 🔒 Нейтрализация телеметрии: VK Push (`szj`), латентный DPS-пинг (`cs5`).
- 🔑 Chat-PIN: long-press заголовка чата → toggle защиты (Toast-подтверждение).
- 🛠 Инфраструктура: `work/sentinel_audit.py` — детектор молчаливо-непримененных патчей.

## 🆕 Унаследовано из 26.27.1 V1
- 🔐 «О приложении» → настройки мода (long-press).
- 🔑 Chat-PIN — вход по PIN (ChatPinReceiver в манифесте).
- 🛡 Обход App Lock через переоткрытие устранён (`onStart`-гейт).

## 🆕 Унаследовано из 26.26.0 V5
- ❌ Визуальная метка удалённых сообщений (красный крестик + setAlpha 0.5f).
- 📜 Форс-консьюм при открытии чата (гейт `smartAntiRead || antiRead`).

---

## 🛡 Профиль приватности

### Невидимка (`invisible`) — мастер-переключатель

| Дочерний флаг | Что делает |
|---|---|
| `antiTyping` | собеседник не видит набор текста |
| `antiRead` | собеседник не видит отметку «прочитано» |
| `smartAntiRead` | «прочитано» уходит только после вашего ответа (consume-on-use) |
| `antiPresence` | онлайн / last-seen скрыты |

### Антиудаление (`antiDelete`) — 4 слоя
1. **L3** — Central WS chat-event. 2. **L4** — Room ORM body-clear. 3. **Маркер** ❌ + время удаления. 4. **FCM офлайн-персист**.

### Нейтрализация телеметрии — чокпоинты
`p0_telem_push` (VK Push), `p0_telem_onelog`, `p0_telem_operator` (IP-strip), `p0_telem_dps`.

---

## 🔍 Верификация (V4)

Независимая адверсариальная кросс-верификация — декомпиляция **фактически собранного** `MAX_26.27.1_arm64_V4.apk` (baksmali, 3 DEX), проверка по байткоду:

| Проверка | Результат |
|---|---|
| `ChatPinManager` содержит `hideView`, `revealView`, `dropHidden`, `abortHidden` + поле `hiddenRef:WeakReference` | ✅ подтверждено |
| `hideView`: `const/4 v0, 0x4` (INVISIBLE) + `setVisibility(I)V` + `sput-object hiddenRef` | ✅ подтверждено |
| `ChatScreen.onViewCreated`: `hideView` вызван СТРОГО раньше `promptForOpen` и только после `isPinned == true` | ✅ подтверждено |
| `OkClick.onClick`: `revealView` только на верном PIN; ветка неверного PIN — `dropHidden` + `finishActivity` без `revealView` | ✅ подтверждено |
| Первичная установка PIN: `String.length()` + `if-ge 0x4`; при < 4 → `dropHidden` + `finishActivity` без `setStoredHash` | ✅ подтверждено |
| `PinDragCb.n()`: вторая проверка ViewHolder использует `if-nez v1, :pd_ok_a` (не `if-eqz`) | ✅ подтверждено |
| `KeepAliveManager.ensureServiceRunning` начинается с `sget-boolean AppFlags->bgWatchdog:Z` + `if-eqz` → return-void | ✅ подтверждено |
| `KeepAliveManager.enable` содержит `sget-boolean AppFlags->bgWatchdog:Z` + `if-eqz` | ✅ подтверждено |
| Регистровый аудит 147 apply-скриптов по 8 классам коллизий | ✅ 0 COLLISION |
| `apksigner verify` (все 4 APK) | ✅ v2+v3, exact ABI, no tracer, distinct SHA |
| Динамический smoke на устройстве (S25) | ✅ main+clone: cold start clean, PIN-фикс и `bgWatchdog` подтверждены пользователем |

---

## 📦 Артефакты (V4)

| Файл | SHA-256 |
|---|---|
| `MAX_26.27.1_arm64_V4.apk` | `99b1d21f93b8f6b0b43bb1bc6ed815614883e17bbd4029953302a7ae75c5dfc1` |
| `MAX_26.27.1_arm64_V4_clone.apk` | `049617bd8df6e1f15a2528afb01b8eaa5c9ec61ba2cdc51a2de6d5f525348714` |
| `MAX_26.27.1_arm7_V4.apk` | `6de1eaf71acda9eca35ae75044e1de66856ee22d54e2be7d2c213d95638e1c95` |
| `MAX_26.27.1_arm7_V4_clone.apk` | `08f2121049168d5e56dc0a9c46074c824bafcb10aab039412af95fffd4680a33` |

Подпись: v2+v3 APK Signature Scheme. Cert SHA-1 идентичен всей линейке 26.x — install -r без потери данных.

---

*Аудит выполнен статическим анализом smali (baksmali + независимый агент-верификатор) с декомпиляцией фактически собранных APK и подтверждён динамическим smoke-тестом на устройстве S25. Тумблер `bgWatchdog` — escape-hatch для MIUI/HyperOS, гипотеза без прямого воспроизведения на подконтрольном устройстве.*
