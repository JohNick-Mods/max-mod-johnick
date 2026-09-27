# MAX 26.33.1 GP V1 — AUDIT

**Дата**: 2026-09-27
**Версия**: 26.33.1 GP V1 (`MIRROR_VERSION_TAG = "26.33.1.1"`, `VERSION=26.33.1`, `BUILD_NUM=1`)
**Сборка**: `make_v1.py --strict --e2e` (пересборка V1 rev3, 27.09.2026, exit 0)
**Автор**: JohNick (Mods by JohNick)

---

## 1. Что нового в V1

Первый публичный релиз на стоке 26.33.1 — полный порт функционала из 26.32.1/26.31.0 GP/RS с исправлением R8-drift (152/313 apply-скриптов, fan-out 18 агентов → 0 FAIL) плюс багфиксы порта:

- **2331-F8 — тап по цитате (reply/quote) джампит к исходному сообщению**: новые `apply_v_quote_jump_gate_a.py` (гейт по `Lcag->a`, sentinel `:quote_jump_gate_a_v1`) + `apply_v_quote_jump_g.py`. Device-verified Redmi 27.09.
- **2331-F11 — кнопка «прочитать всё» (✓✓) реально помечает прочитанным**: `apply_v_manual_read_button.py` V3 — активная отправка read-mark из onClick: захват `chatServerId` в `saf.c` → `AppFlags.lastMarkedChatServerId` → ClickListener (`readMarkSender` → `g53.J(serverId)` → `saf.a(Lj23)` → bypass `consumeForceReadOnce` → `sendReadMark`). Device-verified Redmi 27.09 (пользователь подтвердил дважды).
- **2331-C1 — гудок дозвона на Redmi (MTK/HyperOS)**: `RingbackTonePlayer.start()` стрим 0→`STREAM_MUSIC` (3), volume 80→100 (`apply_v_ringback_v29.py`). Device-verified Redmi 27.09 («громкость ок»).
- **2331-U4 — прозрачный фон**: закреплённые чип строки чата и плашка закреплённого сообщения больше не рисуются непрозрачным цветом темы (`apply_v_chats_transparent_flag.py`, sentinel'ы `:n33_pin_chip_transparent_v1` / `:pin_bars_transparent_v1`).
- **2331-A1/A2/A3/A4/A5/A6, 2331-B2/B3/B5/B6/B7/B8, 2331-F1/F3/F4/F5/F6/F7/F9/F10, 2331-U1/U2/U3, 2331-P1–P6** — закрыты при порте (см. BUGS.md секция 26.33.1 GP, все device-verified на Redmi 26-27.09).
- Сохранён весь функционал 26.31.0/26.32.1: E2E-шифрование (Signal Protocol), anti-удаление с live-маркером, умная античиталка, hide-чаты/каналы/папки, AMOLED, обои чата, watch-mode, телеметрия-нейтрализация (P0-P1 + T1/T3/T6/T7/T8), VPN-глушилки, update-checker с fallback, Dev-меню.

## 2. Артефакты сборки

4 APK + 4 `.idsig` + `SHA256SUMS` в `RELEASE/26.33.1_GP_V1/`:

| Файл | Заметка |
|---|---|
| `MAX_26.33.1_arm64_V1.apk` | arm64, source |
| `MAX_26.33.1_arm64_V1_clone.apk` | arm64, clone |
| `MAX_26.33.1_arm7_V1.apk` | arm7, source |
| `MAX_26.33.1_arm7_V1_clone.apk` | arm7, clone |

SHA-256 — см. `RELEASE/26.33.1_GP_V1/SHA256SUMS`. `build_report.json` — полный отчёт сборки.

## 3. Статические гейты сборки (`--strict`, build_report.json 27.09)

| Гейт | Результат | Примечание |
|---|---|---|
| `undefined_method_gate` | 0 находок | чисто |
| `table_registration_gate` | 0 находок | чисто |
| `reg_collision_gate` | 0 находок | чисто |
| `type_ref_gate` | keys 25 (ожидаемый список) | проверен |
| `r8_drift_gate` | `feature_loss_scripts: []` | чисто |
| `partial_fail` / `soft_fail` | `[]` | чисто |
| `audit_tumblers_hidden --fail-loud` | 0 подозрительных из 19 hidden | 7 READ/BOTH задокументированы alt-UI-контролами («управляется») |
| `threat_scan --prev 26.32.1 --cur 26.33.1` | 34 находки, NEW/CHANGED HIGH: 0 | отчёт `RELEASE/26.33.1_GP_V1/THREAT_SCAN.{md,json}` |

## 4. Аудит порта и пайплайна

- **R8-drift порт**: fan-out 18 sonnet-агентов по 313 apply-скриптам → раунды → **0 FAIL** (урок `lessons_port_26331_r8drift_20260926.md`). Новые гейты: `field_ref_gate.py`, `table_registration_gate.py`.
- **Аудит BUGS.md секции 26.33.1**: все 10 device-багов (2331-F1/B2/B3/B4/B5/B6/U1/A5/F5/F6) закрыты device-verify на Redmi 26-27.09; не-блокеры (D1-D6, F2) задокументированы DEFERRED.
- Инжекты кнопки read-mark приземлены: `:mrb_capture_chat_v1` (saf.smali:275), `:mrb_lastmarked_v1` (AppFlags.smali:6142), `:mrb_active_send_v1` (ManualReadClickListener.smali:77-112); `g53.J(J)Lj23` — public final (сверено).

## 5. Проверка маркеров в собранном дереве (baksmali)

| Маркер | Статус |
|---|---|
| `:quote_jump_gate_a_v1` | FOUND (c3/ojb.smali, гейт по `Lcag->a`) |
| `:mrb_active_send_v1` | FOUND (ManualReadClickListener) |
| `:mrb_capture_chat_v1` | FOUND (saf.smali:275-281) |
| `:mrb_lastmarked_v1` | FOUND (AppFlags.smali:6142) |
| `:n33_pin_chip_transparent_v1` / `:pin_bars_transparent_v1` | FOUND (n33.smali:424 / PinBarsWidget:1439) |
| `RingbackTonePlayer STREAM_MUSIC(3)` | FOUND (RingbackTonePlayer.start) |

## 6. Device-верификация (Redmi `WGJBBEY9SSMN8TIB`, MTK/HyperOS, 27.09.2026)

- `install -r` main (`ru.oneme.app`) + clone (`ru.oneme.ap2`): `Success` (обоих).
- Запуск обоих пакетов: процессы живы, `logcat -b crash` пуст, нет FATAL/VerifyError.
- Device-verify пользователя (27.09): гудок дозвона слышен («громкость ок»); кнопка «прочитать всё» помечает прочитанным у собеседника (подтверждено дважды); quote-jump работает; прочие 10 багов секции 26.33.1 — подтверждены ранее.

**Ограничение**: полный функциональный smoke по всем тумблерам фич в рамках этого аудита не проводился (см. чеклист `/patch-smoke`).

## 7. Известные deferred-долги (не блокируют V1)

| Область | Статус |
|---|---|
| call-E2E цепочка (keyderive/hook_spike/addpart_clear/callhist/video_toggle/w22_indicator) | DEFERRED: SDK звонков обфусцирован, guard-skip; тумблер скрыт, `CALL_LOCK_UI=false` — не блокер |
| 2331-F2 (chatPin title long-click) | DEFERRED: мёртв с 26.31.0 (be3 не toolbar) |
| 2331-D6 (fail-open R8 name-reuse: filterInviteLinks/cacheContacts/cacheChatList/filterFolders/exportCurrentChatNow/filterSettingsList/FolderTabModel) | DEFERRED: класс не падает, фичи деградируют, TODO re-resolve |
| `apply_v_scroll_cold_bottom_v13` | NO-OP известен, патч E2E_ONLY (U2 закрыт через postForceBottom H/A0→D/n0) |

## 8. Косметика (не баги, задокументировано)

- `Loh5;->n` — логгер-обёртка заглушён модом (методы void): `sendReadMark` в logcat не виден даже при успешной отправке; верификация read-mark — по поведению собеседника (см. `lessons_26331_manual_read_button_20260927.md`).
- `audit_tumblers_hidden.py` доработан: задокументированные alt-UI-контролы («управляется»/«hidden-ok») в комментариях tumblers.yaml больше не считаются подозрительными.

## 9. Вердикт

Сборка и состав 26.33.1 GP V1 — корректны: `--strict` exit 0, все статические гейты чисты, R8-drift-аудит порта 0 FAIL, device-verify ключевых фикс-багов на Redmi подтверждён пользователем. Публикация — по отдельной явной команде пользователя (предусловие: gh-авторизация проверена, ассеты сверены с последним релизом v26.31.0.16).
