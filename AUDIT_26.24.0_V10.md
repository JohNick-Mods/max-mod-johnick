# MAX 26.24.0 RS V10 — Технический аудит мода

**Версия стока**: 26.24.0 (источник — RuStore, RS variant)
**Версия мода**: V10 (Privacy Mod by JohNick)
**Дата**: 2026-07-29
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **26.24.0 RS V10** — двойной хотфикс поверх V8. V9: оптимизация размера APK (−17.6% через max-DEFLATE). V10: исправление непрерывного гудка дозвона — `stop()` перенесён из преждевременного `fc1.b()` в `d25.onCallAccepted()`, реализация атомарным `ToneGenerator`. **Безопасность и приватность не менялись** — все security-инварианты идентичны V1–V8. Весь функционал V8 сохранён.

---

## 🆕 Что нового в 26.24.0 RS V9–V10

### 1. Оптимизация размера APK (V9)

Уровень сжатия при репаке APK поднят с 6 до 9 (максимальный DEFLATE). Результат: **−17.6%** от размера (arm64: с ~40 МБ до ~33 МБ). На совместимость и производительность не влияет — сжатие применяется только к DEFLATE-записям ZIP, `resources.arsc` остаётся STORED. Дополнительно: путь к `aapt2.exe` теперь берётся напрямую из `tools/`, без поиска в PATH.

### 2. Исправление непрерывного гудка дозвона (`#USER-2 V29`)

**Симптом (V8 и ранее)**: гудок на исходящем звонке обрывался досрочно — телефон переставал «гудеть» в момент ответа собеседника, но вместо нормального перехода в разговор было ~2 секунды тишины.

**Корневая причина**: `RingbackTonePlayer.stop()` вызывался в `fc1.b()` (markActive) — точке, где Android Telecom фиксирует «звонок принят». Однако Android Telecom закрывает audio session с задержкой ~2 сек после markActive, всё это время WebRTC транслировал тишину поверх уже остановленного ToneGenerator.

**Исправление V29**:

| | Было (V15–V28) | Стало (V29) |
|---|---|---|
| `stop()` при ответе | `fc1.b()` (markActive) | `d25.onCallAccepted()` — точное событие ответа |
| `stop()` при сбросе | `fc1.a(I)` (destroy) | `fc1.a(I)` (без изменений) |
| Реализация плеера | Цепочка fallback V15/V18/V25/V27/V28 | Атомарный `ToneGenerator(0, 80).startTone(0x17, -1)` |
| Sentinel | `:ringback_v28_tonegen_supringtone` | `:ringback_v29_fix` |

**Детали реализации**: `ToneGenerator(STREAM_VOICE_CALL=0, volume=80)` + `startTone(TONE_SUP_RINGTONE=0x17=23, duration=-1)` — стандартный supervisory ringback tone, бесконечный цикл до `stopTone()`. Атомарный подход: убраны все ветки fallback (Samsung Ringtone, MediaPlayer, STREAM_RING) — на всех устройствах единый путь через ToneGenerator. Класс `RingbackTonePlayer.smali` полностью переписан; `apply_v_ringback_v15/v18/v27.py` помечены DEPRECATED в pipeline.

Второй callsite-патч добавлен в `d25.smali` (onCallAccepted): `invoke-static {}, Lone/me/util/RingbackTonePlayer;->stop()V` с sentinel `:ringback_v29_accepted`.

---

## 📦 Из V8 — сохранено без изменений

### Дедупликация уведомлений при Антиудалении (`#PUSH-2`)

**Шаг A — расширение dedup-гейта**: ring-buffer 500 msgId активен при `antiRead OR antiDelete` (V7: только antiRead). Sentinel `:push2_dedup_gate_v10` в `AppFlags.smali`.

**Шаг B — дроссель `ensureServiceRunning`**: throttle 30 сек при частых реконнектах. Всё тело `try/catch Throwable` — при ошибке throttle пропускается. Sentinel `:push2_throttle_v1` в `KeepAliveManager.smali`.

### Цвет уведомления = цвет иконки (`#ICON-1`)

Тумблер **`notifMatchIcon`** (default ON). Акцент уведомления в шторке совпадает с цветом активной иконки:

| Цвет иконки | ARGB акцента |
|---|---|
| Синий (0) | стоковый (passthrough) |
| Красный (1) | `0xFFEB3B39` |
| Зелёный (2) | `0xFF4CAF50` |
| Фиолетовый (3) | `0xFF9C27B0` |
| Чёрный (4) | `0xFF424242` |
| Оранжевый (5) | `0xFFFF9800` |

Sentinel `:icon_notif_color_v1` в `nhh.smali`, `:notif_accent_color_v1` в `AppFlags.smali`.

### Больше закреплённых чатов (до 10) (`#PIN-1`)

Тумблер **`unlockPinLimit`** (default OFF). Хелпер `AppFlags.pinLimit(I)I` → `10` вместо серверного значения. Три call-site: `:pin1_callsite_wr0_v1`, `:pin1_callsite_ta_v1`, `:pin1_callsite_kp2_v1`.

---

## ⌚ Из V7 и ранее — сохранено

- **🔒 PIN-блокировка запуска** (`appLock`, `appLockUseBiometric`, `changePinAction`).
- **⏩ Скорость воспроизведения** (`playbackSpeed` 0.5×–2×).
- **🔔 Дедупликация уведомлений при antiRead** (ring-buffer 500 msgId).
- **⌚ Watch-улучшения** (WATCH-11/13/14: nag-баннеры, Stories, кнопка voice).
- **🗑️ Префикс удалённых сообщений**, **🎵 Рингтон MAX на One UI 7**.

---

## 🔧 База V1–V4 (сохранено)

- Антиудаление (4 слоя + офлайн-персист), невидимка, AMOLED, watch-mode, скрытие сервисных чатов.
- 6 цветов иконки (runtime alias-свитч), push клона стабилен, проверка/автопроверка обновлений, дев-меню MAX, SMS-регистрация.

---

## 🛡 Профиль приватности

### Невидимка (`invisible`) — мастер-переключатель

| Дочерний флаг | Что делает |
|---|---|
| Антипечатанье (`antiTyping`) | собеседник не видит набор текста |
| Античиталка (`antiRead`) | собеседник не видит отметку «прочитано» |
| Умная античиталка (`smartAntiRead`) | «прочитано» уходит только ПОСЛЕ вашего ответа |
| Антиприсутствие (`antiPresence`) | онлайн / last-seen скрыты |

### Анти-удаление (`antiDelete`) — 4 слоя + офлайн-персист
1. **L3 — Central WS chat-event**. 2. **L4 — Room ORM body-clear**. 3. **V3 persist**. 4. **Маркер 🗑 + время**. 5. **Офлайн-удаления через FCM** восстанавливаются на старте.

---

## 🔒 Security-инварианты (идентичны V1–V8 — V9/V10 их не касались)

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
WS-redirect от сервера MAX **не блокируется** — SMS-код доходит штатно.

### ✍️ Подпись
- **apksigner**, схемы **v2 + v3** (без v1). Cert **SHA-1 `d6a7d757…1298b`** — идентичен для всех сборок 26.x → `install -r` поверх любой предыдущей версии мода без потери данных.

### 📦 Нативные библиотеки
Удаляется **только `libtracernative.so`**. Остальные `.so` — оригинальные из стока 26.24.0.

---

## ⚠ Честно: ограничения в 26.24.0 RS V10

- **#PIN-1 — серверный сброс**: сервер MAX может сбросить пины сверх 2 при синхронизации. Тумблер default=OFF.
- **#ICON-1 — share-меню и «О приложении»**: Samsung Sharesheet и экран App Info берут иконку из `applicationInfo` пакета (OEM-ограничение OneUI), а не из активного alias. Домашний экран и шторка уведомлений — корректны.
- **Ringback**: ToneGenerator маршрутизируется через STREAM_VOICE_CALL; на устройствах без активной VoIP-сессии в момент DIALING гудок может быть тихим.
- **AMOLED — покрытие ~95%**: отдельные bottom-sheets могут остаться тёмно-серыми.
- **Диалоги мода — тёмная схема** (фиксированный тёмный фон).
- **Watch-фичи** активны только при `widthPixels < 500`.
- **Тумблер `improveNotifications` скрыт из UI**.
- **Требуется one-time `pm clear ru.oneme.ap2`** после первой установки клона.
- **Минимальная версия Android — 8.0** (API 26).

---

## ✅ Smoke-тест (2026-07-29)

- Устройство: Samsung S25 (`RFCX5013V5F`), Android 15, arm64.
- Основной (`ru.oneme.app`) + клон (`ru.oneme.ap2`) — **0 FATAL, 0 VerifyError** в логах.
- `install -r` поверх V8 → user-data сохранена, чистый старт.
- Sentinel `:ringback_v29_fix` и `:ringback_v29_accepted` присутствуют в DEX.
- **#USER-2 V29**: гудок непрерывен до ответа собеседника, останавливается точно при ответе.
- Все фичи V8 (PUSH-2 / ICON-1 / PIN-1) работают без регрессий.
- update-check ✅, clone ✅, AMOLED ✅, dev-menu ✅.

---

## 🔒 SHA-256 (V10)

```
6f2584309d214deb7c2171bc97209cfe44a9506e9e21a82e9c4d10b760f13082  MAX_26.24.0_arm64_V10.apk
774d69f04734c31e38081694720aa69b2c16bf5d24018ed52f037bacada56930  MAX_26.24.0_arm64_V10_clone.apk
f3132e2c54bcf747db591157784fe33888bfaf757b4f104908ce0b6afd157720  MAX_26.24.0_arm7_V10.apk
8488dab36240e97f587767a8c3496fb36bcddf6328db0b5caf03eb1070b7b2bd  MAX_26.24.0_arm7_V10_clone.apk
```

---

## 📜 Лицензия

Только для личного использования, в учебных и исследовательских целях. Мод предоставляется «as is», подписан debug-ключом, не публикуется в Google Play. Правообладатель оригинального MAX — **ООО «МАХ» (VK)**; мод не аффилирован, не претендует на права на код / знаки / иконки. Использование может нарушать Условия сервиса — **на свой риск**.
