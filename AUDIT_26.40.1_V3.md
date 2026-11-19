# Технический аудит MAX Mod 26.40.1 RS V3

- **Версия мода:** `26.40.1 RS V3` (тег `v26.40.1.3`)
- **Базовый APK:** `MAX 26.40.1 RS` (сборка `6928`)
- **Дата:** `2026-11-19`
- **Статус:** `FINAL PASS (READY)`
- **Run ID:** `be9a74fd207c` (`2472.8s`, `407` шагов, `make_v1.py --strict --e2e --call-e2e --media-e2e --skip-anti-split`, status=success, commit `dfb7efa`)
- **Итоговый вердикт:** `PASS (READY)`

## 1. Назначение и границы отчёта
Данный отчёт фиксирует результаты статических гейтов сборки, полного fan-out аудита патчей (`505` перечисленных скриптов, `350` уникальных активных, `REAL_ACTIVE_BUG = 0`), Threat Scan, независимой проверки подписей и упаковки 4 APK, изолированных ART/JNI стресс-проб и живого прогона на эмуляторе Android (`emulator-5554`, `maxmod_api34_clean_20261101`, API 34, `arm64-v8a` через `libndk_translation`).

В `26.40.1 RS V3` объединены:
- **1:1 E2E аудио- и видеозвонки** с переключением аудио↔видео (`classes5.dex` + `libv6call.so` для `arm64-v8a` и `armeabi-v7a`, выравнивание `0x4000` под 16 KB страницы);
- **Полное E2E-шифрование вложений всех 5 типов**: `PHOTO`, `AUDIO` (голосовые), `VIDEO`, `FILE`, `VIDEO_MESSAGE` (видеокружки);
- **вся база и исправления `26.40.1 RS V2`**: `Outbox26401` (`26401-F6` / `26401-B1`), `UpdateRollback` (откат на прошлые релизы + резервный HTML/Codeberg-источник), `BgNotifGuard` (`IMPORTANCE_MIN` тихое фоновое уведомление), а также база V1 (`B18`, `F1`–`F5`, `F24`–`F37`, `AccountSwitchGuard`, `A3`, `callRecording`, `UID`, `ModOfflineHelp`, `ZIP_STORED` рингтон/гудок).

**Границы проверки (`NOT_RUN`):** живой обмен E2E-медиапотоком звонка между двумя физическими ARM64-устройствами (ограничение одного эмулятора `holdActiveCallForNewCall`), физическая установка на Redmi/S25 и исполнение 32-битного `arm7` в рантайме в этом цикле не проводились.

## 2. Основные изменения 26.40.1 RS V3
1. **1:1 E2E аудио/видеозвонки и переключение аудио↔видео:**
   - Нативный модуль `libv6call.so` (`239 344` байт для `arm64-v8a`, `0x4000` 16 KB page-aligned) + `classes5.dex` (`316 168` байт, `186` классов, `1 716 / 65 535` refs).
   - Настройка `E2E Звонки` задаётся индивидуально для контакта в профиле личного чата (`Сквозное шифрование` → `OFF` / `PREFERRED` / `REQUIRED`). Совместимое legacy-поле `e2eCalls` в `tumblers.yaml` имеет статус `hidden: true` и не отображается в UI.
   - В режиме `REQUIRED` действует жёсткий fail-closed отказ (`DROP_STUB` с 0-го кадра без установленного ключа).
   - Постоянный `DROP` на демонстрацию экрана в E2E-звонке (`D4`) и fail-closed завершение при групповом звонке / звонке по ссылке (`D5`).
   - Устранён дедлок завершения сессии в JNI (`26401-C2`, lock-free teardown: `500` раундов ART/JNI, `0` дедлоков, `0` крашей).
2. **E2E-вложения 5 типов:**
   - Сквозное шифрование и расшифровка для `PHOTO`, `AUDIO` (голосовые), `VIDEO`, `FILE` и `VIDEO_MESSAGE` (видеокружки).
   - Зачистка метаданных на уровне wire-протокола (`26400-B3`), блокировка понижения шифротекста до открытого текста (`26400-B20`), очистка временных файлов при отмене (`26400-B21`).
   - Исправлены 4 ошибки верификации ART (`26401-A4`): `c3/hge.smali`, `c1/jd0.smali`, `c3/v9l.smali` (переведены на самодостаточные замены метода с сохранением `this`) и `UploadFileAttachWorker.smali` (переключён на свободные регистры `v14..v18` с `invoke-static/range`). Повторная проверка всех 23 пропатченных классов в D8 и ART на API 34: `0` `VerifyError`.
3. **Исправления `26.40.1 RS V2` полностью в составе V3:**
   - `Outbox26401` (`26401-F6` / `26401-B1`): стабильная отправка текста, фото и голосовых в обычных чатах, «Избранном» и на клоне, в том числе при включённом глобальном E2E в нешифрованных чатах.
   - `UpdateRollback`: откат на предыдущие сборки из окна обновлений и меню «Починить», резервный источник при 403/429.
   - `BgNotifGuard`: тихий канал `IMPORTANCE_MIN`, без повторного появления после смахивания.

## 3. Проверка APK, контрактов и безопасности
- **Полный fan-out аудит патчей (`fanout_audit_26401_v3_unified.md`):** `505` перечисленных скриптов, `350` уникальных активных скриптов выполнены (`350 / 350`, `0` `SOFT_FAIL`, `0` `PARTIAL_FAIL`, `0` Tracebacks). `REAL_ACTIVE_BUG = 0`. `4` ожидаемых `GRACEFUL_SKIP_SCRIPTS` (`apply_v22_settings.py`, `apply_v_clone_deviceid.py`, `apply_v_clone_deviceid_local.py`, `apply_v_devmenu_version.py`).
- **Целостность DEX (все 5 DEX в `MAX_26.40.1_arm64_V3.apk`):** `27 327` классов суммарно (`classes.dex` — `8 972`, `classes2.dex` — `11 104`, `classes3.dex` — `6 225`, `classes4.dex` — `840`, `classes5.dex` — `186`), `0` дубликатов, `0` недостающих `method_ids` в пространстве имён мода (`10 416` проверено), `r8.jar` D8 `exit=0`.
- **Threat Scan (`threat_scan.py`, V2 `arm64_V2` → V3 `arm64_V3`):** **0 NEW/CHANGED HIGH** (`1` MEDIUM — новая библиотека `lib\arm64-v8a\libv6call.so`; `2` LOW — отсутствие `nm/objdump` и эвристика `PmsProperty-registry`).
- **Независимая проверка 4 APK:** `verify_apk.py --full-apk --call-e2e --media-e2e` — **`4/4 PASS`** (`5` DEX, `56/56` классов, `4/4` маркеров); `apk_packaging_escape_gate.py` — **`4/4 PASS`**; `call_e2e6_landing_gate.py --strict` — **`4/4 PASS`** (`95` E2E-ссылок разрешены); `apksigner_google.jar verify` — **`4/4 Verifies`** (`v2 + v3`, SHA-1 сертификата `d6a7d757fa383af131263eb585d4e2be4da1298b`).
- **Скрытые тумблеры (`audit_tumblers_hidden.py --fail-loud`) и релиз-ноты (`audit_release_notes_tumblers.py --fail-loud`):** **`0` ошибок** (`0` подозрительных из `20` `hidden:true`).
- **Автотесты (`pytest`) и ART/JNI-пробы:**
  - `51 passed` (pre-build 8 suites с `MAX_REQUIRE_AVD_ART_VERIFY=1`); `3 passed` (`test_v6_unified_plumbing.py`); `52 passed` на post-build 9-suite прогоне (2 теста V2 ожидают чистое дерево до применения V2-патчей).
  - `AVD_G3_G4_PROBE`: `82/82 PASS` (`realJni=true`).
  - `C2_STRESS_PROBE`: `500` раундов, `0` дедлоков, `0` крашей, `270 146` аудио- и `317 968` видео-операций JNI, `1 500` проверок искажённых кадров, `8 000` проверок состояния.
  - `W5OffAndR3cProbe`: `148/148 PASS` на Host-JVM и `148/148 PASS` в ART `app_process64` по всем 5 типам вложений.

| APK | Пакет | Размер | `apksigner` | Состав E2E |
|---|---|---|---|---|
| `MAX_26.40.1_arm64_V3.apk` | `ru.oneme.app` | `45 454 742` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`arm64-v8a`) |
| `MAX_26.40.1_arm64_V3_clone.apk` | `ru.oneme.ap2` | `45 454 742` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`arm64-v8a`) |
| `MAX_26.40.1_arm7_V3.apk` | `ru.oneme.app` | `44 144 052` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`armeabi-v7a`) |
| `MAX_26.40.1_arm7_V3_clone.apk` | `ru.oneme.ap2` | `44 144 052` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`armeabi-v7a`) |

## 4. SHA256 (`SHA256SUMS`)
```
b0759aaad96551e061952ba1e1bb3b0f0a42468f62714df5367f775d6b946f3d  MAX_26.40.1_arm64_V3.apk
3f05a486fb44beb7ac2482e023980e4aba947489f0e00f0d43ad3a1f6893ac85  MAX_26.40.1_arm64_V3_clone.apk
2f67597328556b7f45863231d717e1f4bef569fdcc4530a288a490dbaff54b6d  MAX_26.40.1_arm7_V3.apk
f029c84db2c7314eefe77a1da3d1675a730569b5e1f6750d3015d8426a6db163  MAX_26.40.1_arm7_V3_clone.apk
```

## 5. Живой AVD QA 26.40.1 RS V3 (`emulator-5554`, main + clone)
- **Установка, AOT и запуск:** `adb install -r` main (`ru.oneme.app`, `b0759aaa...`) + clone (`ru.oneme.ap2`, `3f05a486...`), SHA-256 установленных `base.apk` совпадает; `dex2oat speed` завершён успешно (`0` `VerifyError`, `0` крашей), `tcpdump` DNS P0 (`0` утечек).
- **Регрессия `26401-A4`:** поиск контакта `Модем`, открытие чата и меню вложений — `0` `VerifyError`, процесс стабилен.
- **Pillar 2 (UI и E2E-профиль):** `6/6` этапов `PASS` (вкладки, чаты, dev-menu/запись звонков, 7 строк E2E в профиле контакта «Модем» включая выбор `E2E Звонки`, IPC и passthrough `B1` на обоих пакетах, клон).
- **Исправления V2 (`Outbox26401`, `UpdateRollback`, `BgNotifGuard`):** `4/4` этапов `PASS` (текст, фото и голосовое отправлены с `0` `Not sent`, возврат `e2eEncryption=False`, список отката до `v26.40.1.2` отрисован, канал `IMPORTANCE_MIN` подтверждён).
- **Pillar 3 (звонки и камера):** `3/3` этапов `PASS` (аудиозвонок с mute и завершением, видеозвонок с `Frames produced = 94` и переключением камеры, входящий push клон→основное).
- **`universal_avd_prerelease_gate.py`:** `11/11` проверок `PASS`, квитанция `AVD_PRERELEASE_GATE_RECEIPT.json` (`2026-11-19T00:47:16Z`).
- **Непроверенные зоны (`NOT_RUN`):** живой медиапоток E2E-звонка между двумя реальными ARM64-устройствами, установка на физические Redmi/S25, рантайм `arm7` (32-bit).
