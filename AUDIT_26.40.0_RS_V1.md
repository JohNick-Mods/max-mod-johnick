# AUDIT 26.40.0 RS V1

> Статус: **FINAL PASS** (13.11.2026). Все гейты `--strict`, независимая проверка 4 APK (`verify_apks.py`), Threat Scan и живой AVD Smoke (`8/8` + Max-Coverage `7/7`) пройдены.

## 1. Сводка
- Версия: `26.40.0 RS`, сборка `V1` (тег `v26.40.0.1`)
- Результат сборки (`make_v1.py --strict --e2e`, прогон `v1_cold8`, `run_id=0ff2ccf5145c`): **PASS** (`rc=0`, `overall_pass=True`, `fatal_markers=False`, время cold build `1080.359` сек)
- Итоговый вердикт: **PASS (READY)**
- Общее время цикла (S1–S6, `TIMINGS_26.40.0_RS_V1.json`): `67407.755` сек (`t0_epoch = 1794451722.5761063`)
- Открытые дефекты и закрытые инциденты: см. [BUGS.md](../../../BUGS.md), секция `26.40.0 RS` (`26400-A1..A10`, `26400-P2..P13`)

## 2. Статические гейты (`--strict`, `v1_cold8`)
| Гейт | Результат |
|---|---|
| `undefined_method_gate` | **PASS** (`0` находок) |
| `reg_collision_gate` | **PASS** (`0` новых; suppressed PENDING baseline: `6`) |
| `type_ref_gate` | **PASS** (WARN-only; suppressed baseline: `1`) |
| `table_registration_gate` | **PASS** (`0` companion-builder сайтов, проверка пропущена штатно) |
| `version_consistency_gate` | **PASS** (`VERSION='26.40.0'`, `0` несовпадений в 7 файлах) |
| `field_ref_gate` (`gates/known_false_positives.yaml`, `26400-P7`) | **PASS** (`0` новых находок вне baseline) |
| `resource_id_gate` | **PASS** (`0` неизвестных id в литералах мода) |
| `r8_drift_gate` (`26400-P9`) | **PASS** (STEP 350, пины синхронизированы) |
| `new_instance_init_gate` | **PASS** (`26177` smali-файлов, F16-класс: `0`) |
| `replace_selfcontained_gate` | **PASS** (`1000` изменённых файлов, `3190` методов, `0` находок) |
| `class_hierarchy_gate` | **PASS** (`26177` классов, `0` находок) |
| `skip_ledger_gate` | **PASS** (`13` пропусков, `0` вне `gates/skip_allowlist.yaml`) |
| `dex_budget_gate` / `dex_refs_monitor` (лимит `65535`) | **PASS** (`c1`: `59162` / 90.28% OK; `c2`: `63325` / 96.63% WARN; `c3`: `33635` / 51.32% OK) |
| `landing_check` (`26400-P5`) | **PASS** (все активные якоря найдены в `work/baksmali/`) |
| `prepackage_contract_gate` (`26400-P12`) | **PASS** (`DEX_PREPACKAGE_CONTRACTS_PASS`, `48/48` классов, `4/4` маркеров) |
| `audit_tumblers_hidden` | **PASS** (`0` подозрительных из `20` `hidden:true`, всего `109` тумблеров) |
| `audit_release_notes_tumblers` | **PASS** (`BAD=0`, `WARN=1` ложный: `hideSettingsBanners` `default:true`, `OK=3`) |

## 3. Fan-out аудит 14 групп
Отчёт: [AUDIT_FANOUT_14_GROUPS.md](../../AUDIT_FANOUT_14_GROUPS.md) — **PASS (ZERO CRITICAL REGRESSIONS)**: проанализированы `343` активных шага (`APPLY_STEPS=257`, `E2E_ONLY_STEPS=28`, `POST_E2E_STEPS=11`, `FINAL_PATCH_STEPS=47`), `281` кастомных DEX-сентинелов подтверждены в дереве, `0` блокирующих дефектов.

## 4. Threat Scan
Отчёт: [THREAT_SCAN.md](THREAT_SCAN/THREAT_SCAN.md) / `THREAT_SCAN.json`.
- Всего находок: **772** (`NEW=580`, `CHANGED=173`, `KNOWN-NEUTRALIZED=14`, `KNOWN-BENIGN=5`)
- Открытых **HIGH** угроз (`NEW`/`CHANGED`): **0**

## 5. Независимая проверка APK (`verify_apks.py` / `apksigner`)
Отчёт: `build-support/runs/release_26400_v1_20261112/s5_verify/verify_apks_report.json` — **RESULT: PASS** (`4/4` APK).

| APK | Пакет | Размер | Схема подписи | `classes4.dex` (`Lone/me/e2e/`) |
|---|---|---|---|---|
| `MAX_26.40.0_arm64_V1.apk` | `ru.oneme.app` | `34 719 899` байт (`33.11 MB`) | `v2=true, v3=true, v1=false` | `692 520` байт (OK) |
| `MAX_26.40.0_arm64_V1_clone.apk` | `ru.oneme.ap2` | `34 719 899` байт (`33.11 MB`) | `v2=true, v3=true, v1=false` | `692 520` байт (OK) |
| `MAX_26.40.0_arm7_V1.apk` | `ru.oneme.app` | `33 421 495` байт (`31.87 MB`) | `v2=true, v3=true, v1=false` | `692 520` байт (OK) |
| `MAX_26.40.0_arm7_V1_clone.apk` | `ru.oneme.ap2` | `33 421 495` байт (`31.87 MB`) | `v2=true, v3=true, v1=false` | `692 520` байт (OK) |

## 6. SHA256 (`SHA256SUMS`)
```
dde3a34081789418f11984ae893e807bb97ac5a5c71b3d1a31171810032dae29  MAX_26.40.0_arm64_V1.apk
49932902a43edcfeda9d2fffc3d268253e3957410fe1444ecf5a9a43dce340a8  MAX_26.40.0_arm64_V1_clone.apk
4ec89a0f44e6e9cbae642c5a904d8ef5e9742931fc2167ba875a87c258ab3528  MAX_26.40.0_arm7_V1.apk
2a4e575511cda43e7260e4dda8b70dbd8b42f013bc4cd7c4246a9ba0df604209  MAX_26.40.0_arm7_V1_clone.apk
```

## 7. Живой AVD Smoke (`emulator-5556` / `maxmod_api34_clean_20261101`)

### 7.1. Основной 8-стадийный прогон (`run_avd_smoke_26400_v1.py`): **PASS (8/8)**
| Стадия | Описание | Результат |
|---|---|---|
| S1 (`u21_about_badge_and_version`) | Экран «О приложении»: `MAX 26.40.0`, `Сборка мода V1`, `about_counter_badge_present=false` (`0` узлов) | **PASS** |
| S2 (`u10_u11_mod_settings`) | Настройки мода: поиск, избранное, изменённые, справка, возврат в корень (`crash=false`) | **PASS** |
| S3 (`a24_developer_menu`) | DevMenu → Feature Toggles → `LockedTlsRow` (`locked_tls_present=true`, `pid=3592` сохранён, `no_when_crash=true`) | **PASS** |
| S4 (`f122_u12_modem_profile`) | Профиль «Модем»: `f122_whitelist_present=true`, `u12_e2e_present=true`, `crash=false` | **PASS** |
| S5 (`f127_antidelete_regression`) | Переключение `antiDelete` off/on + восстановление, `viewport=14`, `mmg_rjb_cancellation_exception=false` | **PASS** |
| S6 (`f126_send_message_regression`) | Реальная отправка токена `avd_26400_e954e35076d1a833`: `before=0 -> after=1`, `sent=true`, `stuck_on_clock=false` | **PASS** |
| S7 (`clone_app_check`) | Клон `ru.oneme.ap2`: запуск, авторизованный UI, вход в настройки мода, `crash=false` | **PASS** |
| S8 (`max_extra_parity_ui`) | `VendorAutostart` (`vendor_autostart_dialog=true`) и `ChatKeywordFilter` (`chat_keyword_filter_dialog=true`) | **PASS** |

### 7.2. Расширенный Max-Coverage прогон (`run_avd_max_coverage_26400.py`): **PASS (7/7)**
- Stage 0 (`install_and_crosscheck`): `adb install --no-incremental -r` + совпадение SHA-256 на устройстве (`2/2`).
- Stage 1 (`art_verification`): `cmd package compile -m speed -f` (`Success`, `0` значимых `VerifyError`) на `ru.oneme.app` и `ru.oneme.ap2`.
- Stage 2 (`categories_crawl_and_new_toggles`): все 5 категорий (`🔒 Приват`, `📁 Папки`, `💬 Чаты`, `⚙️ Система`, `🎨 Вид`), `hideDockBadge=true`, `crashes=false`.
- Stage 3 (`help_catalog_update_and_feedback`): `AboutCards` (`["Настройки мода", "Обновления", "Меню разработчика"]`) + диалог `UpdateChecker`, `crash=false`.
- Stage 4 (`navigation_tabs_and_bottom_bar`): `["Чаты", "Контакты", "Звонки", "Настройки", "Чаты"]`, `crash=false`.
- Stage 5 (`chat_rich_features_and_codespan`): `scrolled_history=true`, `codespan_error_found=false`, `crash=false`.
- Stage 6 (`clone_app_deep_verification`): `both_pids_alive=true`, `mod_settings_crawled=true`, `crash=false`.

Логи `logcat -b crash`: **чисто (0 крашей)** во всех стадиях S1–S8 и S0–S6 Max-Coverage.

## 8. Изменения релиза
Черновик примечаний к релизу: `RELEASE/RELEASE_NOTES_v26.40.0.1.draft.md`. Новые классы: `VendorAutostart`, `ChatKeywordFilter`, `WatchdogAlarm`. Тумблеры: `hideDockBadge`, `hideMicStatus`, `hideSettingsBanners`; действия: `hideChatKeywordsAction`, `vendorAutostartAction`.
