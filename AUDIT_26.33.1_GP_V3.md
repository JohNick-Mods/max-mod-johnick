# MAX 26.33.1 GP V3 — AUDIT

**Дата:** 2026-10-01
**Версия:** 26.33.1 GP V3 (`MIRROR_VERSION_TAG = "26.33.1.3"`, `VERSION=26.33.1`, `BUILD_NUM=3`)
**Сборка:** `make_v1.py --strict --e2e` (полный clean-build БЕЗ --skip-baksmali, 01.10.2026, exit 0; все 272 шага применены с pristine-дерева)
**Автор:** JohNick (Mods by JohNick)

---

## 1. Что нового в V3

Третий выпуск на стоке MAX 26.33.1 — исправления багов второго выпуска, подтверждённых репортами юзера:

- **2331-A10 — краш «Цифровой ID» (доступ через Госуслуги) — ✅ FIXED.** Краш проявлялся после переустановки/обновления «с нуля». Корень: `VerifyError` в `LinkInterceptorActivity` — патч `:e2e_lic_finish_v1` записывал `move-result v9`, но при `.registers 11` **v9 == p1 = параметр Bundle onCreate** — запись boolean затирала Bundle, сток ниже использует p1 как reference → VerifyError. Исправлено: `move-result v9` → `move-result v5` (мёртв в точке вставки). Подтверждено юзером на устройстве («работает, закрывай баг»).
- **2331-C2 — гудок продолжается после ответа абонента (Tecno/Android Go) — 🔧 FIXED.** Второй путь события «абонент ответил»: `c2/vk5.smali::F()` (CallEngine onCallAccepted) не имел stop-хука — на Tecno/Android Go сток доставляет ответ именно через него. Добавлен `apply_v_ringback_v29_vk5.py` (stop() после лога, sentinel `:ringback_v29_accepted_vk5`). Ждёт device-verify на Tecno.
- **2331-F26 — свои удалённые сообщения возвращаются после перезапуска — ✅ FIXED.** Глубокий рекон: реальный путь удаления — ЛОКАЛЬНЫЕ UI use cases `k3b.a`/`fr2.a` → `gqg` → `e3b.p` (AP-MARK метил свои ids в `deletedKept`) → `kcb.h` гейт блокировал UPDATE DELETED → статус не писался в БД. Фикс v3.1 (по источнику): свои локальные ids помечаются в `AppFlags.f26OwnDeleteIds`, AP-MARK их не метит, `kcb.h` реально пишет DELETED. Анти-удаление чужих WS-удалений и stories сохранено. Подтверждено юзером на устройстве.
- **2331-U6 — полноэкранные уведомления (ColorOS/OxygenOS/realmeUI) — 🔧 FIXED.** Пункт «Полноэкранные уведомления» в «Устранении проблем» на SDK>=34 использует стоковый хелпер `La49->d(Context,Z)` → экран `MANAGE_APP_USE_FULL_SCREEN_INTENT` (работает и на ColorOS/OxygenOS/realmeUI); legacy — `APP_NOTIFICATION_SETTINGS`; catch — `openAppDetails`. Ждёт device-verify.

Инфраструктурная доводка:
- F26-рекон выявил и задокументировал реальные пути удаления (k3b/fr2→gqg, av4/v9c/w9c, te4-stories) — `F26_RECON.md`; старый нерабочий фикс `apply_v_f26_own_delete_bypass.py` заменён на `apply_v_f26_own_delete_local.py`.
- После первого прогона V3 обнаружен и исправлен краш при запуске: VerifyError в `e3b.p` — `move-result v8` ломал wide-параметр p1:p2 (v7:v8 = J chatId); заменён на `v5` (свободен в цикле). Полная пересборка чистая.

## 2. Артефакты сборки

4 APK + 4 `.idsig` + `SHA256SUMS` в `RELEASE/26.33.1_GP_V3/`:

| Файл | Заметка |
|---|---|
| `MAX_26.33.1_arm64_V3.apk` | arm64, source |
| `MAX_26.33.1_arm64_V3_clone.apk` | arm64, clone |
| `MAX_26.33.1_arm7_V3.apk` | arm7, source |
| `MAX_26.33.1_arm7_V3_clone.apk` | arm7, clone |

## 3. Статические гейты (--strict)

| Гейт | Результат |
|---|---|
| undefined_method_gate | 0 |
| table_registration_gate | 0 findings / 0 diagnostics |
| reg_collision_gate | 0 |
| r8_drift_gate | 75 OK / 0 DRIFT / MISSING=0 |
| new_instance_init_gate | 0 F16-класс (raw 7667 — все super-call/covariance) |
| dex_refs_monitor | c1 56 019 (85.5% OK) / c2 62 644 (95.6% WARN) / c3 31 457 (48.0% OK) |
| partial_fail / soft_fail | 0 / 0 |
| APK sha256sums | сгенерированы |

## 4. Device smoke (Redmi HyperOS, serial WGJBBEY9SSMN8TIB)

- `install -r` arm64 main (`ru.oneme.app`) → Success.
- Device-side SHA-256 main APK совпал с `RELEASE/26.33.1_GP_V3/SHA256SUMS` (`780fbfea…`) — без stale-dex.
- Приложение запущено (процесс жив), `logcat -b crash` — пусто (после фикса VerifyError).
- F26 device-verify юзером: удаление своего сообщения + перезапуск → сообщение не возвращается; анти-удаление чужих — работает.

## 5. Fan-out аудит патчей (HARD-RULE)

Аудит всех 303 `apply_v_*.py` перед публикацией 26.33.1.3:

- **Статический sentinel_audit.py — ВСЕ 303 скрипта:** OK (WARN только для audit-only/generator-скриптов без TARGET/SENTINEL — штатно).
- **Статические гейты сборки `--strict` (BUILD_EXIT=0):** undefined_method_gate, table_registration_gate, reg_collision_gate, r8_drift_gate (0 DRIFT), new_instance_init_gate, dex_refs_monitor — ловят wired-but-undefined, R8-name-reuse, регистровые коллизии, DEX-лимиты на каждой сборке.
- **LLM-аудит тематических групп** (проверка якорей/sentinel'ов в итоговом дереве, мёртвые вызовы, silent-skip): покрыты группы 1–2 (по 2 параллельных агента, ограничение среды на число активных агентов); остальные группы покрыты статическими гейтами выше.
- Найденные реальные проблемы устранены до публикации (или задокументированы как отключённые/неактивные скрипты).

**Примечание по V3:** единственное наблюдение LLM-аудита — `apply_v_accent_switch.py` (MED, косметика): заголовки секций настроек перекрашиваются скриптом, но `AppFlagsView.smali` регенерируется каждой сборкой; на чистом дереве 5 заголовков секций остались стоковыми (табы и Switch перекрашены). Скрипт self-heal при повторном прогоне; визуальный нюанс, не влияет на работу. Зафиксировано в BUGS.md (косметика, P-класс).