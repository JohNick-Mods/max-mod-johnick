# AUDIT 26.40.0 RS V2

> Статус: **FINAL PASS** (13.11.2026). Все гейты `--strict`, независимая проверка 4 APK (`verify_apks.py`), Threat Scan и живой AVD Smoke V2 пройдены.

## 1. Сводка
- Версия: `26.40.0 RS`, сборка `V2` (тег `v26.40.0.2`)
- Результат сборки (`make_v1.py --strict --e2e --skip-anti-split`, `run_id=e7cc3ca838c3`): **PASS** (`rc=0`, время сборки `1117.7` сек)
- Итоговый вердикт: **PASS (READY)**
- Закрытые баги релиза V2: `26400-A11`, `26400-F6`, `26400-F7` (см. [BUGS.md](../../../BUGS.md))

## 2. Исправленные дефекты V2
1. **`26400-A11` (`localArchive` краш при старте):** исправлен DI-ключ кэша (`0x98 -> 0x97`) и R8-сигнатура `Lisd;-><init>(ILl1k;...)` в `experiments/local_archive/` и `native_batches/f95_f97_v1/`.
2. **`26400-F6` (вкладка «Сферум» не скрывается):** устранена префиксная коллизия сентинела в `apply_v_folder_filter_fg3.py` (`# :folder_filter_fg3_fanout`), добавлена строгая проверка обеих веток `c1/ta4.smali` (`:pswitch_4dd` и `:pswitch_4ee`).
3. **`26400-F7` (`autoUpdateCheck` при установке):** добавлен `apply_v_autoupdate_persist.py` в `POST_E2E_STEPS` — при отсутствии ключа в `app_flags.prefs` (`!prefs.contains("autoUpdateCheck")`) записывается `true`, ручное отключение пользователем сохраняется.

## 3. Threat Scan и проверка APK
- Threat Scan: всего `449`, открытых **HIGH** (`NEW`/`CHANGED`): `0`.
- Независимая проверка 4 APK (`verify_apks.py`): **PASS**.

| APK | Пакет | Размер | Схема подписи | `classes4.dex` (`Lone/me/e2e/`) |
|---|---|---|---|---|
| `MAX_26.40.0_arm64_V2.apk` | `ru.oneme.app` | `34 719 899` байт | `v2=true, v3=true, v1=false` | `OK` |
| `MAX_26.40.0_arm64_V2_clone.apk` | `ru.oneme.ap2` | `34 719 899` байт | `v2=true, v3=true, v1=false` | `OK` |
| `MAX_26.40.0_arm7_V2.apk` | `ru.oneme.app` | `33 421 495` байт | `v2=true, v3=true, v1=false` | `OK` |
| `MAX_26.40.0_arm7_V2_clone.apk` | `ru.oneme.ap2` | `33 421 495` байт | `v2=true, v3=true, v1=false` | `OK` |

## 4. SHA256 (`SHA256SUMS`)
```
beb46a3d6e39b05f4b3a193d5904b5fe00e863cef66537ad118db279bcf49b57  MAX_26.40.0_arm64_V2.apk
016016f5dcc768a70f9d7a0921c6316e5736a22b6044bd009eaed3ab55ce4446  MAX_26.40.0_arm64_V2_clone.apk
b67b2dbcb699385c5262f5cb17485c6cee3fac97ed2ce193b4bea5153153e422  MAX_26.40.0_arm7_V2.apk
ec64e7f2d2f64b06f4f9360617e3a2f5d816acdcaa33641d5b19d0046011c886  MAX_26.40.0_arm7_V2_clone.apk
```

## 5. Живой AVD Smoke V2 (`emulator-5556` / `maxmod_api34_clean_20261101`)
- Результат прогона (`verify_v2_fixes_avd.py`): **ALL PASS (6/6)**
- Установка (`--no-incremental -r`, без `uninstall`/`pm clear`) + SHA-256 `base.apk`: `ru.oneme.app` (`beb46a3d...`) и `ru.oneme.ap2` (`016016f5...`) — **PASS (2/2)**
- `cmd package compile --reset` + `compile -m speed -f` (`ru.oneme.app` + `ru.oneme.ap2`): **PASS (`Success`, `0 VerifyError`)**
- `26400-F7` (`autoUpdateCheck`): автозапись `autoUpdateCheck=True` после удаления ключа (`pid=5700`) и сохранение ручного `False` при рестарте (`pid=5898`) — **PASS**
- `26400-A11` (`localArchive=true`): холодный старт (`pid=6074`) + открытие экрана Архива (`['Архив', 'Архив пуст', 'ЗАКРЫТЬ']`), `0` крашей — **PASS**
- `26400-F6` (`hideSferum=true` + `hideJunkFolders=true`): холодный старт (`pid=6307`), `uiautomator dump` подтвердил видимые вкладки `['All', 'Новые', 'Личные', 'Каналы']` без «Сферум» — **PASS**
- Финальный чистый запуск (`ru.oneme.app` `pid=6474`, `ru.oneme.ap2` `pid=6595`), `logcat -b crash`: **пусто (0 крашей)**
