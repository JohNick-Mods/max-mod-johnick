# AUDIT 26.40.0 RS V4

> Статус: **FINAL PASS (READY)** (14.11.2026). Все гейты `--strict`, независимая проверка 4 APK (`verify_apk.py` + `apksigner`), Threat Scan, пост-сборочные проверки, unit-тесты V4 и живой AVD QA на `emulator-5554` (ограничения по живому аудиозвонку и физ. устройству зафиксированы в разделе 5) пройдены.

## 1. Сводка
- Версия: `26.40.0 RS`, сборка `V4` (тег `v26.40.0.4`)
- Результат сборки (`make_v1.py --strict --e2e --skip-anti-split`, `run_id=e58f03dae4a5`): **PASS** (`rc=0`, время сборки `1073.086` сек, `375` шагов, `partial_fail: 0`, `soft_fail: 0`)
- Лимиты DEX (`dex_refs_monitor`): `c2/classes2.dex` `63 330 / 65 535` (`96.6%`); `classes4.dex` (`Lone/me/e2e/`): `736 420` байт (`75` вызовов smali -> Java)
- Итоговый вердикт: **PASS (READY)**
- Изменения и закрытые дефекты V4: `26400-F13` + регрессия главного экрана V3, `26400-F12`, `26400-F14`, `CALLREC_v1` («Запись аудиозвонков»)

## 2. Нововведения и исправленные дефекты V4
1. **`26400-F13` и регрессия главного экрана / вкладок / превью (`experiments/local_archive/`):** исправлена логика границы пагинации и повторного прикрепления (`ArchiveServerLoaderState.attachBoundary`, `ArchiveChatMenu`). Устранены бесконечный спиннер при прокрутке списка чатов ниже ~10 элементов, пустые вкладки папок («Новые», «Личные», «Каналы») и зависшие превью чатов на главном экране при `localArchive=true` и `localArchive=false`, сохранён полный цикл архивации и возврата из архива.
2. **`26400-F12` (PIN по долгому нажатию на заголовок чата):** добавлены классы `ChatPinIdentity`, `ChatPinTitleLongClickHandler`, `ChatPinManager` (`apply_v_chat_pin.py`, переключатель `chatPin` — «🔒 PIN на отдельные чаты» в «Приватность → Приватность чата»). Исправлено разрешение идентификатора чата и обработка долгого нажатия на заголовок чата для защиты/снятия PIN-кода.
3. **`26400-F14` (зависание исходящих на `⏱` в групповых чатах):** обновлены `E2ESilentTransport` (корректная проверка `server == 0` для `CHAT`/`GROUP_CHAT`/`CHANNEL`/`COMMENTS`, допускающая отрицательные `server` ID) и `E2EOutgoingDispatch` (`chat == policy.server && chat != 0 && user == 0`). Устранено зависание исходящих сообщений на значке часов в группах.
4. **Запись аудиозвонков (`CALLREC_v1`):** реализованы модули `lib/src/one/me/callrec/` (`CallRecorder`, `CallAudioPipeline` — SPSC ring buffer 48 kHz mono -> AAC-LC 64 kbps `.m4a` в `Music/MAX Calls/`, `CallRecordingsHelper`), патчер `apply_v_call_recording.py` (12 хуков) и переключатели в `tumblers.yaml` («⚙️ Система → Звонки»): `callRecording` («Запись аудиозвонков»), `callRecordingAuto` («Автозапись всех звонков») и кнопка просмотра «Записи звонков» (`callRecordingsAction`).

## 3. Проверка APK, контрактов и безопасности
- Threat Scan (`threat_scan.py`): всего `450` находок (`NEW=432`, `CHANGED=13`, `KNOWN-NEUTRALIZED=4`, `KNOWN-BENIGN=1`; `MEDIUM=435`, `LOW=15`), открытых **HIGH** (`NEW`/`CHANGED`): `0`.
- Независимая проверка 4 APK (`verify_apk.py` + `apksigner_google.jar verify`): **PASS (4/4)** (`settings contract: APK DEX instructions OK`, `DEX: 4 sequential entries`, `48/48 defined classes OK`, `4/4 string-pool markers OK`; `apksigner`: `4/4 Verifies`).
- Скрытые тумблеры (`audit_tumblers_hidden.py --fail-loud`): **0** подозрительных из `20` `hidden:true` (всего тумблеров `113`).
- Пост-сборочный аудит (`@audit-1`, `@audit-6`): **CONFIRMED**.
- Целевой набор unit/contract автотестов V4 (`pytest` по `test_v4_call_recording_core.py`, `test_v4_call_recording_hooks.py`, `test_v4_chat_pin_longpress.py`, `test_v4_f14_group_send.py`, `test_v4_local_archive.py`): **24 passed in 9.59s** (полный сборочный прогон V4 — **27 passed in 16.07s**).

| APK | Пакет | Размер | `apksigner` | `classes4.dex` (`Lone/me/e2e/`) |
|---|---|---|---|---|
| `MAX_26.40.0_arm64_V4.apk` | `ru.oneme.app` | `34 740 379` байт | `Verifies` (`OK`) | `736 420` байт (`OK`) |
| `MAX_26.40.0_arm64_V4_clone.apk` | `ru.oneme.ap2` | `34 740 379` байт | `Verifies` (`OK`) | `736 420` байт (`OK`) |
| `MAX_26.40.0_arm7_V4.apk` | `ru.oneme.app` | `33 446 071` байт | `Verifies` (`OK`) | `736 420` байт (`OK`) |
| `MAX_26.40.0_arm7_V4_clone.apk` | `ru.oneme.ap2` | `33 446 071` байт | `Verifies` (`OK`) | `736 420` байт (`OK`) |

## 4. SHA256 (`SHA256SUMS.txt`)
```
5dfb0d53410faabcc3ff54ac80b79960b48233ba2ee582dbe46bd2e1950cfcea  MAX_26.40.0_arm64_V4.apk
d8d9f560556b3ea5ef517793b133eacb2e015e22f391c255f3c558a94c59c8af  MAX_26.40.0_arm64_V4_clone.apk
3cbdc6960930e6a439e991e13cec647309210fd0401bce5a71bffd9ab2b32884  MAX_26.40.0_arm7_V4.apk
1487c74e6bd45a729bbdc1ffe386eaca83c7c0fdc0fa7abc05333e64bc94b2b2  MAX_26.40.0_arm7_V4_clone.apk
```

## 5. Живой AVD QA V4 (`emulator-5554` / `maxmod_api34_clean_20261101`)
- Установка поверх (`-r`) и сверка SHA-256 `base.apk`: `ru.oneme.app` (`5dfb0d53410f...`) и клон `ru.oneme.ap2` (`d8d9f560556b...`) — **PASS (2/2)**.
- ART-компиляция `speed` (`0 VerifyError`) и чистый старт клона `ru.oneme.ap2` (`0` записей в crash-логе) — **PASS**.
- `26400-F13` и регрессия главного экрана (`localArchive=true` и `localArchive=false`): подтверждены отсутствие спиннера на главном экране (`pb=0`), прокрутка ниже 10-го чата (`pb1=0, pb2=0`), заполненные вкладки «Новые», «Личные», «Каналы» (`pb=0`), открытие экрана «Архив» (`pb=0`), архивация группы «Жопец» (исчезновение из `All`, отображение в «Архиве») и возврат обратно в `All` («Архив пуст», `pb=0`) — **PASS**.
- `26400-F14`: в группе «Жопец» (`e2e=true`) отправлены контрольные сообщения при `localArchive=true` (`v4_qa_send_0147`, статус `Sent`, обновление превью `You:⁠​⁠v4_qa_send_0147`) и при `localArchive=false` (`v4_qa_off_0152`, статус `Sent`, обновление превью в `All` при `pb=0`) без зависания на `⏱` — **PASS**.
- `26400-F12`: проверено долгое нажатие на заголовок чата «Жопец» (`ChatPin: long-press fired`, `ChatPinIdentity.resolveLocal` -> `"41"`), включение «🔒 PIN на отдельные чаты» (`chatPin`), установка PIN (`1234`), скрытие/разблокировка экрана чата по PIN, снятие защиты повторным долгим нажатием на заголовок и возврат флага в `false` — **PASS**.
- `CALLREC_v1`: в разделе «⚙️ Система» проверено переключение тумблеров `Запись аудиозвонков` (`callRecording`) и `Автозапись всех звонков` (`callRecordingAuto`) (`false -> true -> false`), а также открытие окна «Записи звонков (Music/MAX Calls)» (`callRecordingsAction`). **Ограничение:** живая запись аудиопотока во время активного WebRTC-звонка с реальным собеседником на AVD не проводилась (функционал опирается на JVM unit-тесты DSP/ABI, проверку 12 хуков в `work/baksmali` и AVD UI-приёмку настроек/списка записей).
- Финальный `adb logcat -b crash -d`: пусто (`0` крашей). Физическая проверка на реальном телефоне не проводилась.
