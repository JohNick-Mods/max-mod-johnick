# MAX 26.24.0 RS V7 — Технический аудит мода

**Версия стока**: 26.24.0 (источник — RuStore, RS variant)
**Версия мода**: V7 (Privacy Mod by JohNick)
**Дата**: 2026-07-28
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **26.24.0 RS V7** — релиз поверх V4. Добавлен гудок (ringback tone) на исходящих звонках: Samsung-устройства воспроизводят системный файл `/system/media/audio/ui/Dialer_new.ogg` через `MediaPlayer` с `AudioAttributes.USAGE_ALARM` (обходит беззвучный режим / DND); прочие устройства — `ToneGenerator(STREAM_VOICE_CALL, TONE_SUP_RINGTONE=19)` (стандартный supervisory ringback). **Безопасность и приватность не менялись** — все security-инварианты идентичны V1–V4. Весь UI-функционал V4 сохранён.

---

## 📞 Что нового в 26.24.0 RS V7

### 1. Гудок (ringback tone) на исходящих звонках (`#RINGBACK-V7`)

В V4 исходящий звонок не воспроизводил стандартный гудок ожидания ответа — абонент слышал только тишину, пока не ответит собеседник. В V7 добавлен класс `one.me.util.RingbackTonePlayer`, внедрённый в lifecycle исходящего звонка:

**Samsung-устройства** (определяются через `Build.MANUFACTURER.equalsIgnoreCase("samsung")`):
- Воспроизводит `/system/media/audio/ui/Dialer_new.ogg` — стандартный системный звук набора Samsung, присутствующий во всех сборках One UI.
- Реализация: `Ringtone` (MediaPlayer-based) + `AudioAttributes.USAGE_ALARM` — сигнал проходит через alarm-поток, обходит беззвучный режим и DND, что соответствует поведению стокового Samsung Dialer.
- `RingtoneManager.getRingtone(context, Uri.parse("file:///system/media/audio/ui/Dialer_new.ogg"))`, `setLooping(true)`, `play()` в `start()`.

**Прочие устройства** (AOSP/Custom ROM):
- `ToneGenerator(AudioManager.STREAM_VOICE_CALL, 100)`, `startTone(ToneGenerator.TONE_SUP_RINGTONE /* 19 */, -1)` — стандартный Android-тон supervisory ringback.
- Воспроизводится на voice-call stream, интегрируется в текущий аудио-маршрут звонка.

**Жизненный цикл**:
- `start()` — вызывается при инициализации исходящего звонка (до ответа).
- `markActive()` — вызывается при ответе собеседника: `stop()` + освобождение ресурсов.
- `destroy()` — вызывается при разрыве звонка: `stop()` + `release()`.
- Оба пути (ответ / завершение без ответа) корректно останавливают воспроизведение — утечек `MediaPlayer`/`ToneGenerator` нет.

> На не-Samsung устройствах без файла `/system/media/audio/ui/Dialer_new.ogg` используется ветка `ToneGenerator` автоматически, файл не запрашивается.

---

## ⌚ Из V4 — сохранено без изменений

- **📐 Bounded ScrollView в диалогах мода на часах** (`#WATCH-3`). Высота ScrollView ограничена 55% `heightPixels`. Кнопки не уезжают за экран на watchface 466×466.
- **📋 Точечное скрытие `topIndicatorView` (id `0x7f090908`) на часах** (`#WATCH-5`). `Handler.postDelayed(600ms)` + гейт `widthPixels<500`. Только `findViewById + setVisibility`, скролл не затрагивается.
- **🎯 UpdateChecker выбирает ассет по ABI** (`#ABI-UC`). `Build.SUPPORTED_ABIS[0]` → `"arm7"` / `"arm64"`. Двойной фильтр (ABI + isClone) даёт ровно 1 ассет.
- **🔘 Три кнопки в диалоге обновления на часах** (`#WATCH-4`). «Нет / Да / Скачать последнюю» в один ряд.
- **🤝 WearOS wearable-метаданные в манифесте** (`#WATCH-6`). `uses-feature=watch (not required)`, `standalone=true`, `notificationBridgeMode=NO_BRIDGING`. Прямой AXML-редактор, без apktool.

---

## 🐞 Из V3 — сохранено без изменений

- **📎 Отправка фото и файлов** (пере-отрезолвленная R8-ссылка `Lts5;→Lew5;`).
- **🖍 Читаемость диалогов обновления** (`DialogsUtils.tintTextViews` — фон `#2D2D2D`, текст + рамки CompoundButton белым).

## 🆕 Из V2 — UI-рефактор диалогов (сохранён)

- **🔵 Закруглённые диалоги** (16dp) для всех диалогов мода.
- **🎨 Выбор цвета иконки — нативные RadioButton** (6 цветов, prefs `app_flags`/`iconColor`).
- **🧩 Класс `DialogsUtils`** — единый тема-детект + закруглённый показ + тонирование текста.
- **📐 `AppFlagsView` переписан (−63%)** — хелперы `addSwitchRow`/`showDialog`.
- **🌗 Тема диалогов — по системному night-mode** (R8-независимо).

---

## 🔧 База V1 (порт под сток 26.24.0)

- **Порт стока 26.24.0** из RuStore, **R8-rebump 47 классов**.
- **`filterSettingsList`** под новую sealed-архитектуру (`Lxdf;`/`Lwaf;` + `Lq6h;`).
- **`filterChatList` / `filterPageList`** мигрированы (`Ln13;→Lc33;`, `Les6;→Ltv6;`).
- **`startOnNew`** — chokepoint `c1/wi3.smali`. **`hide_phone` Gate2** (`b94→db4→tc7→cgf`).
- **`manual_update_check` + long-press дев-меню**. **`resources.arsc`** — icon-switcher stable-ids.

---

## 🐞 Сохранено из V1 / 26.23.2 V1

- 🗑 **Секунды и иконка корзины на удалённых** (`ЧЧ:ММ:СС ДД.ММ` + 🗑).
- 🔢 **Счётчик непрочитанных** не накручивается на холодном старте.
- 🎨 **«Цвет иконки» — 6 цветов** (нативный RadioGroup).
- 🕹 **«Невидимка» — единый мастер-тумблер** (`antiRead`/`antiTyping`/`antiPresence`).
- 🛡 **Антиудаление** — 4 слоя + офлайн-персист, нет push об удалённых, ⏱→✓.
- ⌚ **Watch-mode** (< 500 px), 🌑 **AMOLED-тема** (авто при тёмной).
- 📲 **Push у клона стабилен**, 🔎 **проверка/автопроверка обновлений**, 🎛 **дев-меню MAX**, 📱 **SMS-регистрация**.

---

## 🛡 Профиль приватности

### Невидимка (`invisible`) — мастер-переключатель

| Дочерний флаг | Что делает |
|---|---|
| Антипечатанье (`antiTyping`) | собеседник не видит набор текста |
| Античиталка (`antiRead`) | собеседник не видит отметку «прочитано» |
| Умная античиталка (`smartAntiRead`) | «прочитано» уходит только ПОСЛЕ вашего ответа |
| Антиприсутствие (`antiPresence`) | онлайн / last-seen скрыты (независимый тумблер) |

**Взаимоисключение**: `antiRead` и `smartAntiRead` не могут быть включены одновременно.

### Анти-удаление (`antiDelete`) — 4 слоя + офлайн-персист
1. **L3 — Central WS chat-event**. 2. **L4 — Room ORM body-clear**. 3. **V3 persist**. 4. **Маркер 🗑 + время**. 5. **Офлайн-удаления через FCM** восстанавливаются на старте (с `isRead=1`).

---

## 🔒 Security-инварианты (идентичны V1/V2/V3/V4 — V7 их не касался)

### 🌐 DNS-блокировка телеметрии (P0)
```
trace-flow.ru
sdk-api.apptracer.ru
tracker-api.vk-analytics.ru
tracker.my.com
top-fwz1.mail.ru
```

### 🔕 Приваси-стабы (P0/P1)
- **myTracker (VK)** — нейтрализован (`return-void`).
- **Tracer-инициализаторы (7×)** — застаблены.
- **Firebase Installations** — `getComponents() → emptyList()`.
- **Фоновая выгрузка контактов** — нейтрализована.
- **NFC-HCE** — APDU-ответ `SW 6F00`.
- **TrustManager accept-all** — заменён на `throw`.

### 📱 SMS-регистрация — работает
WS-redirect от сервера MAX (шардинг auth-flow по номеру) **не блокируется** — SMS-код доходит для новых номеров штатно.

### ✍️ Подпись
- **apksigner**, схемы **v2 + v3** (без v1). Cert **SHA-1 `d6a7d757…1298b`** — идентичен для всех сборок 26.x → `install -r` поверх любой предыдущей версии мода без потери данных.

### 📦 Нативные библиотеки
Удаляется **только `libtracernative.so`**. Остальные `.so` — оригинальные из стока 26.24.0.

---

## ⚠ Честно: ограничения в 26.24.0 RS V7

- **Ringback (Samsung)**: требует файл `/system/media/audio/ui/Dialer_new.ogg`. На Samsung One UI он всегда присутствует. На кастомных прошивках с Samsung-брендингом — возможно отсутствие; в этом случае `MediaPlayer` вернёт ошибку и гудок не зазвучит (silent-fail, звонок продолжается штатно).
- **Ringback (AOSP)**: `ToneGenerator.TONE_SUP_RINGTONE` — звучание зависит от прошивки.
- **AMOLED — покрытие ~95%**: список чатов, окно чата и bottom-sheets могут остаться тёмно-серыми.
- **Диалоги мода — тёмная схема**: фон фиксированно тёмный (для гарантированного контраста).
- **Закруглённые диалоги — best-effort**: у отдельных системных диалогов углы могут остаться прямыми.
- **Watch-фичи активны только на устройствах с `widthPixels < 500`** (гейт runtime). На телефонах инертны.
- **Тумблер `improveNotifications` скрыт из UI**.
- **«Цифровой ID» и «Госуслуги баннер»** — один `hideDigitalId`.
- **Требуется one-time `pm clear ru.oneme.ap2`** после установки APK клона.
- **Минимальная версия Android — 8.0** (API 26).

---

## ✅ Smoke-тест (2026-07-28)

- Устройство: Samsung S25 (`RFCX5013V5F`), Android 15, arm64.
- Основной (`ru.oneme.app`) + клон (`ru.oneme.ap2`) — **0 FATAL, 0 VerifyError, 0 ClassCast** в логах приложения.
- `install -r` поверх V4 → user-data сохранена, чистый старт, оба процесса живы.
- Все sentinel'ы V4 присутствуют в собранных DEX; wearable meta подтверждена через `aapt2 dump badging`.
- **Ringback подтверждён (logcat)**: sentinel'ы `:ringback_v18_callsvcimpl_start`, `:ringback_v15_start`, `:ringback_v15_active`, `:ringback_v15_destroy` подтверждены в DEX; при исходящем звонке logcat фиксирует `start()` при инициализации и `stop()` при ответе / завершении (`markActive()` / `destroy()`).

---

## 🔒 SHA-256 (V7)

```
4b03978dfff94313764b1822b2ede80ac8e456adfe24f925e9cec4f06a9ca3d1  MAX_26.24.0_arm64_V7.apk
0d74b47418f1303948294e2e9d6b78bc1bd81f60966a65c50eccf514a9e09286  MAX_26.24.0_arm64_V7_clone.apk
e9bb2d756e34740a701edd30d78c97155184b7d742ce5d14e49015f3340f342c  MAX_26.24.0_arm7_V7.apk
460f09805dcd34fbcdbd732d22d88da7c2572984a744c8f705f5e46aab069dec  MAX_26.24.0_arm7_V7_clone.apk
```

---

## 📜 Лицензия

Только для личного использования, в учебных и исследовательских целях. Мод предоставляется «as is», подписан debug-ключом, не публикуется в Google Play. Правообладатель оригинального MAX — **ООО «МАХ» (VK)**; мод не аффилирован, не претендует на права на код / знаки / иконки. Использование может нарушать Условия сервиса — **на свой риск**.
