# MAX 26.33.1 GP V2 — AUDIT

**Дата:** 2026-09-29
**Версия:** 26.33.1 GP V2 (`MIRROR_VERSION_TAG = "26.33.1.2"`, `VERSION=26.33.1`, `BUILD_NUM=2`)
**Сборка:** `make_v1.py --strict --e2e --skip-anti-split` (полный clean-build БЕЗ --skip-baksmali, 29.09.2026, exit 0; все 254 шага применены с pristine-дерева)
**Автор:** JohNick (Mods by JohNick)

---

## 1. Что нового в V2

Вторая сборка на стоке MAX 26.33.1 — пост-релизная доводка V1:

- **2331-F23 — краш композера при красном цвете текста (E2E)**: исправлен VerifyError класса `CodeSpan` (тип стиля возвращён на 9 = CODE), цветовая вставка E2E-текста переведена на span-сохраняющий путь `E2ECrypto.transformedDisplay()` (перенос spans через `SpannableStringBuilder`), пересобран `lib/classes4.dex`. CodeSpan.a()Ly45; сохранён (обязательный copy-метод интерфейса, предотвращает VerifyError).
- **2331-F12 — фоновые уведомления**: исправлен `BackgroundWakeBootReceiver` (.registers 5, catch-handler move-exception, success goto), сторожевой alarm + прямой старт `BackgroundListenService`; живой/background путь проверен (startForegroundCount=1, крэшей нет).
- **Clean-build конвейер исправлен** (первый полный прогон без `--skip-baksmali`): P12 reorder FIND против pristine, P13 hide-phone `c1/a28`→`c3` (dex-rebalance), P14 hide-digital-id sentinel-конфликт с reorder, P15 hide-phone `wn3` c1→c3, P16 r8_drift_gate `_resolve()` распознавание. Всё зафиксировано в BUGS.md.

## 2. Артефакты сборки

4 APK + 4 `.idsig` + `SHA256SUMS` в `RELEASE/26.33.1_GP_V2/`:

| Файл | Заметка |
|---|---|
| `MAX_26.33.1_arm64_V2.apk` | arm64, source |
| `MAX_26.33.1_arm64_V2_clone.apk` | arm64, clone |
| `MAX_26.33.1_arm7_V2.apk` | arm7, source |
| `MAX_26.33.1_arm7_V2_clone.apk` | arm7, clone |

## 3. Статические гейты (--strict)

| Гейт | Результат |
|---|---|
| undefined_method_gate | 0 |
| table_registration_gate | 0 findings / 0 diagnostics |
| reg_collision_gate | 0 |
| r8_drift_gate | 81 OK / 0 DRIFT / 1 DEAD→SKIP (dynamic resolve) / 148 SKIP / 21 WARN; feature-loss списки — штатные отключённые/деферированные скрипты |
| partial_fail / soft_fail | 0 / 0 |
| APK sha256sums | сгенерированы |

## 4. Device smoke (Redmi HyperOS, serial WGJBBEY9SSMN8TIB)

- `install -r` обоих arm64 APK (main `ru.oneme.app` + clone `ru.oneme.ap2`) → Success.
- Device-side SHA-256 main APK совпал с `RELEASE/26.33.1_GP_V2/SHA256SUMS` (`90c4c760…`).
- Оба пакета запущены (launcher `one.me.android.IconAliasBlue` / `IconAliasOrange`), процессы живы (PIDs 9734 / 9975).
- `logcat -b crash` — пусто.

## 5. Fan-out аудит

8 параллельных групп (разные модели) проверили все `apply_v_*.py` до публикации: **0 активных FAIL** после исключений. Устранены перед релизом:

- `apply_v_call_e2e_callhist.py` — снят с активного pipeline (P11, тихий no-op на отсутствующем c2/z40).
- `apply_v_p0_telem_onelog.py` — A3-резолвер c3→c1 + fail-closed (больше не лжёт о полном A2+A3).
- `sentinel_audit.py` — учёт динамических TARGET_2331/CURRENT/NEW селекторов.
- Легаси CP1251→UTF-8: `apply_v_contacts_dao_hook.py`, `apply_v_smart_unread_inchat.py` (все 300 apply-скриптов проходят py_compile).

## 6. Известные ограничения

- E2E-шифрование аудио в звонках деактивировано на этом стоке (обфускация call-SDK) — шифрование сообщений работает.
- F12: cold-start после принудительного `am kill` не доказывается на HyperOS (`process is bad`) — ограничение стенда, не кода.
- F23: ручной тест красного текста запланирован, статус OPEN в BUGS.md до подтверждения.

---
