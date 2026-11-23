# Технический аудит MAX Mod 26.40.1 RS V5 (Pre-Release Draft)

- **Версия мода:** `26.40.1 RS V5` (тег `v26.40.1.5`)
- **Базовый APK:** `MAX 26.40.1 RS` (сборка `6928`)
- **Дата:** `2026-11-23`
- **Статус:** `STAGE A (r9) — OFFLINE GATES PASS ([PENDING WAVE D AVD SMOKE])`
- **Run ID:** `247fb56194a4` (`417` шагов, `status="success"`, `partial_fail=[]`, `soft_fail=[]`, commit `5bc1f3c`)
- **Итоговый вердикт:** `[PENDING WAVE D AVD SMOKE]`

## 1. Назначение и границы отчёта
Данный предрелизный черновик фиксирует дельту изменений между `26.40.1 RS V4` (`83c342e127331f55519162b0373a85f472de18af`) и релиз-кандидатом `V5` (`codex/e2e-v5-release-rc`), а также результаты предрелизных проверок документации.

В состав `26.40.1 RS V5` входят:
- **1:1 E2E аудио- и видеозвонки (`C3`/`C2`) + E2E-вложения 5 типов (`PHOTO`, `VIDEO`, `FILE`, `AUDIO`, `VIDEO_MESSAGE`) + `26401-B8`:**
  - Автоматическое включение E2E звонков и вложений при активном `🔒 Приват → E2E Шифрование` (`default: false` по B18) и E2E-режиме личного чата;
  - `e2eAttach` (`E2E Вложения (5 типов медиа)`) в `tumblers.yaml`: `hidden: false, default: true` с однократной миграцией `e2e_attach_v5_default_migrated` при обновлении с V1–V4;
  - `26401-B8` (`classes4.dex`): привязка отображаемой версии E2E собеседника к свежему одноразовому подтверждению расшифровки Signal;
  - `E2E6` (`apply_v_e2e6_exact_owner.py`, `apply_v_e2e6_lifecycle.py`, `apply_v_e2e6_video_gate.py`, `apply_v_e2e6_ui_lock.py`, `CallPolicyProvider6`, `CallControlCoordinator6`, `V6MediaHooks`): двухфазное владение сессией звонка, pre-addTrack gate камеры в `c3/s9e.smali` и продакшн-мост вложений 5 типов.
- **UX и системные улучшения (R23/R24):** история изменений сообщений (`MessageEditHistory`, `💬 Чаты`, `default: true`, пункт `🕓 История изменений (N)` в `MessageContextMenuBottomSheet` с хранением ревизий в `no_backup/mod_edit_history_v1.db` и пропуском E2E-шифротекста), воспроизведение голосовых без датчика приближения (`disableVoiceProximity`, `default: false`), сохранение фоновой музыки при записи (`keepMusicOnRecord`, `default: false`), скрытие кнопки «Сферум» (`hideSferumButton`, `default: false`), экспорт/импорт настроек через SAF (`SettingsBackup`, `60` правил, `96` пунктов каталога, `119` тумблеров), переключатель секунд во времени пузырьков сообщений (`messageTimeSeconds`), быстрое прочтение чатов долгим тапом по вкладке «Чаты» (`F27`, прямое выполнение без диалога подтверждения), прозрачная верхняя панель списка чатов (`apply_v_transparent_top_bar.py`), переход к первому доступному сообщению (`F3`/`26400-F27`), обновлённая справка `ModOfflineHelp` (`F26`) и каталог отката `UpdateRollback` (`F25`).
- **Исправления стабильности и синхронизации:** исправление зомби-таймера записи звонков (`26401-F19`), усиление защиты не-«Все» вкладок папок при холодном `localArchive` (`26401-F18`), защита от противоречивых пакетов синхронизации папок (`26401-F22`), привязка статуса истории при включённом антиудалении (`26401-F19` ON-mode), восстановление фоновых запросов истории (`F21`), а также вся база V3/V2.

**Границы проверки:**
- Финальная Wave D сборка 4 APK (`run_id=247fb56194a4`, commit `5bc1f3c`) и все `10/10` офлайн-гейтов пройдены (`publishable=True`).
- Живой AVD smoke (`run_wave_d_full_smoke.py` + `universal_avd_prerelease_gate.py`): `[PENDING WAVE D AVD SMOKE]`.

## 2. Основные изменения относительно V4 (`83c342e`)
1. **E2E V5 (`B8`, `B2–B7`, `C2–C3`):**
   - `26401-B8` (`lib/v6_auth/`): `53 passed` на host-регрессии `test_v6_*.py` по `classes4.dex`;
   - `C3`/`C2` (`lib/v6/`, `CallPolicyProvider6`, `CallControlCoordinator6`, `apply_v_e2e6_exact_owner.py`, `apply_v_e2e6_video_gate.py`) и `B2–B7` (`lib/v6_media/`, `V6MediaHooks`): продакшн-активация E2E звонков и 5 типов вложений, видимый тумблер `e2eAttach` (`default: true`) и однократная миграция `e2e_attach_v5_default_migrated`.
2. **Новые и обновлённые UX-функции:**
   - `v5_features/message_edit_history/apply_v_message_edit_history.py`: история изменений обычных сообщений (`messageEditHistory`, `default: true`), просмотр и копирование ревизий из контекстного меню (`🕓 История изменений (N)`), пропуск E2E-шифротекста.
   - `apply_v_v5_audio_and_sferum_tweaks.py`: `disableVoiceProximity` (`default: false`), `keepMusicOnRecord` (`default: false`), `hideSferumButton` (`default: false`).
   - `v5_features/story_mark/` (`StoryMarkGate`, `StoryMarkUi` + `apply_v_hide_story_view.py` + `UserStoriesScreen`): кнопка **«👁 Прочитать»** / пункт «Отметить просмотренной» при активных `hideStoryView` или `invisible` для ручной отправки отметки просмотра конкретной истории.
   - `v5_features/settings_transfer/apply_settings_backup.py`: экспорт и импорт настроек мода через Android SAF (`SettingsBackupUi`, `60` правил, `96` пунктов каталога, `119` тумблеров, `ApplicationContext` `app_flags`).
   - `v5_features/message_time_seconds_toggle/apply_message_time_seconds_toggle.py`: тумблер секунд во времени пузырьков сообщений (`AppFlags->messageTimeSeconds:Z`, по умолчанию ON).
   - `v5_features/chats_read_all_hold/apply_chats_read_all_hold.py`: долгое нажатие на вкладку «Чаты» сразу отмечает прочитанными все доступные чаты аккаунта (исключая скрытые каналы).
   - `apply_v_transparent_top_bar.py`: прозрачная шапка списка чатов (`chats_list_appbar`) при включённых `transparentTheme` и `transparentBottomBar`.
   - `v5_features/f3/apply_v_f3_first_message.py`: переход к первому доступному сообщению в чате с `ReadFence` + `ReadQuiescence`.
   - `v5_features/f24/apply_v_only_chats_policy_refresh.py` + `ModOfflineHelp`: terminal publication guard в `AppFlags` и обновлённая справка без внутренних имён классов.
3. **Стабильность и синхронизация:**
   - `apply_v_call_recording.py` + `CallRecorder` / `CallRecordingsHelper` (`26401-F19`): исправлен «зомби-таймер» записи аудиозвонков (добавлены хуки локального завершения `c2/c16.smali :: d0(Lrh5;)V` и `VoIpCallService.onDestroy()V`, переход на монотонный `SystemClock.elapsedRealtime()`, авто-очистка сиротских записей и неблокирующая остановка).
   - `apply_v_u3_radio_wire_fix.py` (`26401-F18` hardening): защита не-«Все» вкладок папок при холодном старте с `localArchive == ON` и авто-восстановление при `archive07 == null`.
   - `apply_v_f22_folder_sync_guard.py` (`26401-F22`), `apply_v_f19_status_bind.py` (`26401-F19` ON-mode), `apply_internal_avd_f21_background.py`, `apply_v_update_rollback.py`.

## 3. Проверка APK, контрактов и безопасности
- **Сборка `run_id=247fb56194a4`:** `417` шагов, `status="success"`, `partial_fail=[]`, `soft_fail=[]`.
- **Скрытые тумблеры (`audit_tumblers_hidden.py --fail-loud`):** `0` подозрительных из `19` `hidden:true` (`119` тумблеров всего, `EXIT=0`).
- **Релиз-ноты и документация (`audit_release_notes_tumblers.py --fail-loud`):** `BAD=0` (`EXIT=0`).
- **`verify_apk.py`:** `4/4 PASS` (`57/57` классов, `4/4` маркеров, `5` DEX, `packaging: OK`).
- **`apksigner`:** `4/4 Verifies` (`v2 + v3`, SHA-256 сертификата `fcf5078923e383b92ca6c60d02e9000c907460af369492ebc39fe17dab7fe4c5`).
- **`universal_zero_escape_gate.py`:** `PASS` (`26 351` классов, `0` находок).
- **`dex_refs_monitor.py`:** все DEX `< 65 535` (`c1=56,954`, `c2=63,300`, `c3=36,965`, `c4=5,483`, `c5=2,881`).
- **`reg_collision_gate.py`:** `COLLISION: 0`, `CLASS-3: 0`.
- **Threat Scan (`threat_scan.py`):** `2 229` находок всего, `0` непротриаженных `NEW/CHANGED HIGH`.

| APK | Пакет | Размер | `apksigner` | Состав E2E |
|---|---|---|---|---|
| `MAX_26.40.1_arm64_V5.apk` | `ru.oneme.app` | `45 639 062` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`arm64-v8a`) |
| `MAX_26.40.1_arm64_V5_clone.apk` | `ru.oneme.ap2` | `45 639 062` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`arm64-v8a`) |
| `MAX_26.40.1_arm7_V5.apk` | `ru.oneme.app` | `44 324 276` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`armeabi-v7a`) |
| `MAX_26.40.1_arm7_V5_clone.apk` | `ru.oneme.ap2` | `44 324 276` байт | `Verifies` (`OK`) | `classes4.dex` + `classes5.dex` + `libv6call.so` (`armeabi-v7a`) |

## 4. SHA256 (`SHA256SUMS`)
```
bdb1ec5b6949c201714da026ca6b432bc7dd8fa1101141eea197e60715eee186  MAX_26.40.1_arm64_V5.apk
44ee16ea649f9fdb83ec206ffc0200c63bdbbc7c57431a03958eeef1cec5760f  MAX_26.40.1_arm64_V5_clone.apk
b461f9cbfff30571cf62a03eefd946ba9ce2e3a05ae6b6164f909480a6d309b7  MAX_26.40.1_arm7_V5.apk
e158ab7961c674de190c089a94bb6081634e55712c288570395f158b16a8a1fc  MAX_26.40.1_arm7_V5_clone.apk
```

## 5. Живой AVD / Device QA (`[PENDING WAVE D AVD SMOKE]`)
- **Установка, AOT и запуск (main + clone):** `[PENDING WAVE D AVD SMOKE]`
- **Wave D full smoke (`run_wave_d_full_smoke.py` + `universal_avd_prerelease_gate.py`):** `[PENDING WAVE D AVD SMOKE]`
