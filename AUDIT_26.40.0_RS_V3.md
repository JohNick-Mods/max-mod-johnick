# AUDIT 26.40.0 RS V3

> Статус: **FINAL PASS (READY)** (13.11.2026). Все гейты `--strict`, независимая проверка 4 APK (`verify_apk.py` + `apksigner`), Threat Scan, пост-сборочный фан-аут аудит (10/10 групп, `0` открытых реальных багов) и живой AVD Smoke V3 пройдены.

## 1. Сводка
- Версия: `26.40.0 RS`, сборка `V3` (тег `v26.40.0.3`)
- Результат сборки (`make_v1.py --strict --e2e --skip-anti-split`, `run_id=587ed65fe9d0`): **PASS** (`rc=0`, время сборки `1057.638` сек, `374` шагов, `partial_fail: 0`, `soft_fail: 0`)
- Итоговый вердикт: **PASS (READY)**
- Изменения и закрытые дефекты V3: `noKeyboardLearning` (новая функция), `26400-F8`, `26400-U2`, `26400-F9`, `26400-F10`, `26400-F11`

## 2. Нововведения и исправленные дефекты V3
1. **`noKeyboardLearning` (`IME_FLAG_NO_PERSONALIZED_LEARNING`):** добавлен переключатель в `tumblers.yaml` («Приватность → Данные и профиль», `default: false`) и патчер `apply_v_ime_incognito.py` (`c2/mhd.smali`, `c1/ew.smali`, `c3/ipb.smali`), добавляющий `0x1000000` в `EditorInfo.imeOptions`.
2. **`26400-F8` (вкладка «Цифровой ID» и транскрибация голосовых):** исправлена инверсия перехода в `:vpn_pms_gate` (`apply_v_vpn_pms_gate.py`, `c1/xse.smali`), тип `Integer.valueOf(0)` для `show-vpn-call-bottomsheet`, а также добавлен однократный сброс `settings.prefs` при обновлении с V2 с сохранением `version` (`:vpn_pms_rehash`) для повторной загрузки PMS.
3. **`26400-U2` (дата регистрации в профиле):** исправлено отображение `RegistrationRow` на экране профиля (`apply_v_profile_registration_date.py` — `setTextColor` и привязка путей презентера профиля).
4. **`26400-F9` (отправка plaintext-комментариев):** исправлен `apply_v_e2e_outgoing_heh.py` (`c1/rn4.smali` `CommentSendApiTask` — корректный маршрут через `E2EPendingMessages.processOutgoing` / `E2EOutgoingDispatch.notePlaintext`).
5. **`26400-F10`, `26400-F11` (зависание исходящих на `⏱` и отправка в группах):** гидрация метаданных чата (`E2ESilentTransport.hydrateTaskChat`), корректное различение групповых чатов (`chatType`) и устранение head-of-line блокировки очереди `E2EPendingMessages.defer`.

## 3. Проверка APK, контрактов и безопасности
- Threat Scan: всего `450` находок (`MEDIUM=435`, `LOW=15`, `CHANGED=13`, `KNOWN-BENIGN=1`, `KNOWN-NEUTRALIZED=4`, `NEW=432`), открытых **HIGH** (`NEW`/`CHANGED`): `0`.
- Независимая проверка 4 APK (`verify_apk.py` + `apksigner verify --verbose`): **PASS (4/4)** (`48/48` классов, `4/4` маркеров пула строк, DEX 4 последовательных секции, settings contract OK).
- Скрытые тумблеры (`audit_tumblers_hidden.py --fail-loud`): **0** подозрительных из `20` `hidden:true` (всего `110`).
- Пост-сборочный фан-аут аудит (`345/345` apply-шагов, 10 групп): `0` открытых реальных багов (единственное расхождение по `apply_v_watch_hide_compact_banner.py` / `c1/cqd.smali` подтверждено как штатный откат широкого патча шагом `STEP 307` `native_batches/watch_v1/apply.py` с переносом точечной защиты в `CallHistoryScreen` и `c2/dv1.smali`).
- Целевой набор автотестов V3 (`pytest`: `test_v3_f9_f10_f11_e2e_fixes.py`, `test_e2e_pending_queue.py`, `test_ime_incognito_contract.py`, `test_registration_date_contract.py`): **28 passed**.

| APK | Пакет | Размер | Схема подписи | `classes4.dex` (`Lone/me/e2e/`) |
|---|---|---|---|---|
| `MAX_26.40.0_arm64_V3.apk` | `ru.oneme.app` | `34 719 899` байт | `v2=true, v3=true, v1=false` | `694 748` байт (`OK`) |
| `MAX_26.40.0_arm64_V3_clone.apk` | `ru.oneme.ap2` | `34 719 899` байт | `v2=true, v3=true, v1=false` | `694 748` байт (`OK`) |
| `MAX_26.40.0_arm7_V3.apk` | `ru.oneme.app` | `33 425 591` байт | `v2=true, v3=true, v1=false` | `694 748` байт (`OK`) |
| `MAX_26.40.0_arm7_V3_clone.apk` | `ru.oneme.ap2` | `33 425 591` байт | `v2=true, v3=true, v1=false` | `694 748` байт (`OK`) |

## 4. SHA256 (`SHA256SUMS.txt`)
```
e826698fbaddbb5ac236a04a79abce34a80f2037c22869d2a728be795ad2f655  MAX_26.40.0_arm64_V3.apk
885d1a8d78efc99ec9424aab6e16d5e2b63c393257089e573092b586a2b0451e  MAX_26.40.0_arm64_V3_clone.apk
afadf8436d56e8e64c37c9731a639699a42ac77837f2841efea5c71bd6fa14ac  MAX_26.40.0_arm7_V3.apk
71c33c547eb326cecf6b1cd2167387932df13b41423e482017b6fef054fc31e4  MAX_26.40.0_arm7_V3_clone.apk
```

## 5. Живой AVD Smoke V3 (`emulator-5556` / `maxmod_api34_clean_20261101`)
- Установка (`--no-incremental -r`, без `uninstall`/`pm clear`) и сверка SHA-256 `base.apk`: `ru.oneme.app` (`e826698fbadd...`) и `ru.oneme.ap2` (`885d1a8d78ef...`) — **PASS (2/2)**
- `cmd package compile --reset` + `compile -m speed -f` (`ru.oneme.app` + `ru.oneme.ap2`): **PASS (`Success`, `0 VerifyError`)**
- `noKeyboardLearning`: тумблер проверен в «Приватность → Данные и профиль»; при включении `dumpsys input_method` подтвердил `imeOptions=0x51000006` (`IME_FLAG_NO_PERSONALIZED_LEARNING` `0x1000000`), после проверки возвращён в `false` — **PASS**
- `26400-F8`: в `app_file_prefs/settings.prefs` подтверждены `mod_v3_pms_rehash=true`, `digitalid-botid`, `enable-audio-messages-transcription=true`, `enable-video-messages-transcription=true`, `show-vpn-chat-bottomsheet=false`, `show-vpn-call-bottomsheet=0`; на нижней панели отображается вкладка `'Digital ID'` — **PASS**
- `26400-U2`: в профиле контакта «Модем» отображена строка `'Дата регистрации: May 30, 2026'`, `0` предупреждений `MaxModRegDate` — **PASS**
- `26400-F10`: отправка `'v3_smoke'` в чат «Модем» завершена со статусом `'Your message. 🔒 v3_smoke'` (без зависания на `⏱`) — **PASS**
- `26400-F9`, `26400-F11`: проверены байткод/smali-контрактными автотестами (`28 passed`) и ART-компиляцией `speed` (`0 VerifyError`); отдельные интерактивные сценарии комментария в канале и группового чата в живом AVD-прогоне не воспроизводились.
- Персистентность `autoUpdateCheck=true` в `app_flags.prefs`, чистый запуск `ru.oneme.app` и `ru.oneme.ap2`, `logcat -b crash`: **0 строк (чисто)**.
