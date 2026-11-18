# Технический аудит MAX Mod 26.40.1 RS V2

- **Версия мода:** `26.40.1 RS V2` (тег `v26.40.1.2`)
- **Базовый APK:** `MAX 26.40.1 RS` (сборка `6928`)
- **Дата:** `2026-11-18`
- **Статус:** `FINAL PASS (READY)`
- **Run ID:** `f48cd23b9399` (`1288.0s`, `397` шагов, `make_v1.py --strict --e2e --skip-anti-split`, status=success)
- **Итоговый вердикт:** `PASS (READY)`

## 1. Назначение и границы отчёта
Данный отчёт фиксирует результаты статических гейтов сборки, дедуплицированного fan-out аудита патчей на базе `26.40.1 RS` (`6928`), Threat Scan, независимой проверки подписей и упаковки 4 APK, а также живого прогона на эмуляторе Android (`emulator-5554`).
В `26.40.1 RS V2` полностью сохранена кодовая база `26.40.1 RS V1` и добавлены целевые исправления: восстановление отправки исходящих сообщений/медиа/тихого транспорта при любом состоянии `e2eEncryption`, откат на предыдущие релизы через встроенный интерфейс и тихое фоновое уведомление (`IMPORTANCE_MIN`, без повторного появления после смахивания).
Физическая приёмка на реальном телефоне и запись живого аудиопотока активного WebRTC-звонка с собеседником для этой сборки не проводились.

## 2. Основные изменения 26.40.1 RS V2
1. **Исходящая отправка сообщений и тихий транспорт (`Outbox26401`):** устранено зависание исходящих сообщений, фото и голосовых в личных чатах, «Избранном», группах и на клоне (`ru.oneme.ap2`) за счёт рефлексивного моста в `c3/one/me/mods/outbox/Outbox26401` без модификации замороженного `lib/classes4.dex` (`ff3442d3...`) и без статических E2E-ссылок. Корректно разрешаются R8-классы сборки `6928` и привязывается очередь тихой отправки.
2. **Откат на предыдущие сборки (`UpdateRollback`):** в окне обновлений («О приложении → Обновления») и в меню «Починить → ⏪ Откатиться на предыдущую сборку» добавлен выбор прошлых релизов с резервным HTML/Codeberg-источником при API 403/429, диалогом подтверждения и заметками о совместимости данных и базы `Room v103`.
3. **Тихое фоновое уведомление (`BgNotifGuard`):** системное уведомление фоновой службы связи изолировано в канал `IMPORTANCE_MIN`, не помечается как несбрасываемое и не пересоздаётся повторно после смахивания пользователем при активной службе.
4. **База 26.40.1 RS V1 сохранена без потерь:** `B18`, `F1`/`F2`/`F3`/`F4`/`F5`, `F24`–`F37`, `AccountSwitchGuard`, `A3` (сброс настроек мода), `P1`/`P20`, `callRecording`, `UID`, `ModOfflineHelp`, `ZIP_STORED` рингтон/гудок.

## 3. Проверка APK, контрактов и безопасности
- Детерминированный fan-out аудит активных `apply_v_*.py` (`fanout_audit_26401.py`) + `sentinel_audit.py`: **REVIEWED — реальных багов активного пайплайна НЕТ** (единственный FATAL `apply_v_e2e_attach_upload_a` и 2 DRIFT `push2_throttle`/`v22_settings` — документированные false positives / exemptions).
- Threat Scan (`threat_scan.py`): всего `526` находок, открытых **HIGH** (`NEW`/`CHANGED`): **0**.
- Независимая проверка 4 APK (`verify_apk.py --full-apk` + `apksigner_google.jar verify`): **PASS (4/4)** (`apksigner`: `4/4 Verifies`, `classes4.dex` SHA-256 `ff3442d3d1567375647472b7973327ca2d1a9b0a2acd7e8968953deb02ef6b3a` не изменён).
- Скрытые тумблеры (`audit_tumblers_hidden.py --fail-loud`) и релиз-ноты (`audit_release_notes_tumblers.py --fail-loud`): **BAD=0**.
- Универсальные гейты (`universal_zero_escape_gate.py --strict`, `reg_collision_gate.py`, `undefined_method_gate.py`, `r8_drift_gate.py`, `landing_check.py`): **PASS**.
- Автотесты (`pytest`): `test_f6_outgoing_fix_26401` (5/5 до финальной сборки; после сборки подтверждена идемпотентность на пропатченном дереве), `test_update_rollback_26401` (4/4), `test_bg_notif_quiet_26401` (6/6), `test_e2e_pending_queue` (3/3), `test_v3_f9_f10_f11_e2e_fixes` (3/3), `test_feature12_mod_feedback` (7 passed, 1 skipped).

| APK | Пакет | Размер | `apksigner` | `classes4.dex` (`Lone/me/e2e/`) |
|---|---|---|---|---|
| `MAX_26.40.1_arm64_V2.apk` | `ru.oneme.app` | `45 200 660` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |
| `MAX_26.40.1_arm64_V2_clone.apk` | `ru.oneme.ap2` | `45 200 660` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |
| `MAX_26.40.1_arm7_V2.apk` | `ru.oneme.app` | `43 902 256` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |
| `MAX_26.40.1_arm7_V2_clone.apk` | `ru.oneme.ap2` | `43 902 256` байт | `Verifies` (`OK`) | `736 420` байт (frozen V4) |

## 4. SHA256 (`SHA256SUMS`)
```
a892a1d82259003c7bc24e628e5d9e16cba96543e3fc0696d0bce1793560d2d3  MAX_26.40.1_arm64_V2.apk
630f622b171ed72032b23322d7f66c85bcc3cf58690b3bbe5345d0b4849d5265  MAX_26.40.1_arm64_V2_clone.apk
2c1d93451c8ca23957f44e1c6783878535499b09b3aa53194154ac0a3f1fe2f5  MAX_26.40.1_arm7_V2.apk
b145d4364e9b19ac27bc3866d52d4ed1b401b919e783a4f173a2ab5a493dc81a  MAX_26.40.1_arm7_V2_clone.apk
```

## 5. Живой AVD QA 26.40.1 RS V2 (`emulator-5554`, main + clone)
- **Установка и запуск:** `adb install -r` main (`ru.oneme.app`) + clone (`ru.oneme.ap2`), SHA-256 установленных `base.apk` = EXACT match; `IconAliasOrange` активен на клоне; `0 VerifyError`, `logcat -b crash` чистый.
- **Исходящие сообщения (`e2eEncryption=OFF` и `ON`):**
  - «Модем» (main): текст, фото и голосовое (2 сек) — мгновенно `"Sent"` (`0` `"Not sent"`, `0` ⏱, `0` `TaskMonitorWorker FAILURE`).
  - «Избранное» (main): текст — `"Sent"`.
  - Клон (`ru.oneme.ap2`): текст — `"Sent"`; подтверждена реальная доставка сообщений между main и clone.
  - Переключение `e2eEncryption=ON` → нешифрованный чат «Модем» → `"Sent"` → возврат `OFF` + сброс настроек («Изменённые» = «Ничего не найдено»).
- **Звонки:** исходящие аудио- и видеозвонок на «Модем» (main) — экран «Calling…», элементы управления, корректное завершение, `0` крашей/VE.
- **Откат на предыдущую сборку:**
  - UI в «О приложении → Обновления → Откатиться на предыдущую» и «Настройки мода → Починить → ⏪ Откатиться на предыдущую сборку» → список тегов (`v26.40.1.1`, `v26.40.0.4`…) → диалог подтверждения с описанием и предупреждением о совместимости `Room v103`.
  - Реальный `adb install -r` (без `-d`) V1 поверх V2 → `Success`, запуск без крашей, переписка сохранена → `adb install -r` V2 поверх V1 → `Success`, SHA-256 совпадает, запуск чистый.
- **Тихое фоновое уведомление и финальный дельта-смоук (`f48cd23b9399`):**
  - `adb install -r` main (`a892a1d8...`) + clone (`630f622b...`) — EXACT match `base.apk` SHA-256, `0` VerifyError / `0` крашей.
  - `dumpsys notification --noredact`: канал `ru.oneme.app.bg_keepalive` («Фоновая служба связи», `mOriginalImp=1`, `mShowBadge=false`, `mSound=null`, `mVibrationEnabled=false`), `flags=0x161` (`FLAG_ONGOING_EVENT` и `FLAG_NO_CLEAR` сняты), `pri=-2`, `vis=SECRET`, кнопка «Скрыть навсегда».
  - `verify_f31_f35_avd.check_f128_watchdog_alarm_avd`: `PASS` (`broadcast_ok=True`, `alarm_rearmed=True`, `bg_listen_service_seen=True`, `no_such_method_error=False`); повторный `WATCHDOG_ALARM` при активном `BackgroundListenService` не пересоздаёт FGS-уведомление и не вызывает `ForegroundServiceDidNotStartInTimeException`.
  - Контрольная отправка `v2final_smoke` в «Модем» (main) — `"Sent"`.
- **Ограничения:** физическая приёмка на реальном телефоне и запись живого аудиопотока активного WebRTC-звонка с собеседником не проводились.
