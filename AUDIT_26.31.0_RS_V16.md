# MAX 26.31.0 RS V16 — AUDIT

**Дата**: 2026-09-25
**Версия**: 26.31.0 RS V16 (`MIRROR_VERSION_TAG = "26.31.0.16"`)
**Сборка**: `make_v1.py --e2e --strict`
**Автор**: JohNick (Mods by JohNick)

---

## 1. Что нового в V16

- **UpdateChecker**: сортировка тегов релизов по убыванию, `group(2)`-фикс HTML-fallback (парсинг имени файла), fallback `mode=14` + `downloadAndInstall`, идемпотентность upgrade-веток (commit `0740f92`).
- **r8 hot-override и drift-скан**: `gen_r8_map`, `r8_hot_override`, `fingerprint_resolver`, `r8_mapping` — актуализация R8-имён под текущий сток.
- Сохранён весь функционал V15 (gap-анализ угроз: T1 call-analytics, T3 VPN-метки, T6+T7 вендорные метрики, T8 Tracer, T5 PMS recovery-url).

## 2. Артефакты сборки

4 APK + 4 `.idsig` + `SHA256SUMS` в `RELEASE/26.31.0_RS_V16/`:

| Файл | Заметка |
|---|---|
| `MAX_26.31.0_RS_arm64_V16.apk` | arm64, source |
| `MAX_26.31.0_RS_arm64_V16_clone.apk` | arm64, clone |
| `MAX_26.31.0_RS_arm7_V16.apk` | arm7, source |
| `MAX_26.31.0_RS_arm7_V16_clone.apk` | arm7, clone |

APK V16 отличаются от V15 (SHA-256 arm64: `FCB62011E…` vs `D6EA5669…`).

## 3. Статические гейты сборки (`--strict`)

| Гейт | Результат | Примечание |
|---|---|---|
| `undefined_method_gate` | 0 находок | чисто |
| `table_registration_gate` | 0 находок | чисто |
| `reg_collision_gate` | 0 реальных | 4 записи PENDING — после ручной сверки с baksmali и fan-out все ложные (R8-drift-пути устарели) |
| `type_ref_gate` | 0 реальных | 7 HIGH + 2 MEDIUM — все ложные позитивы: Kotlin-интерфейсы с унаследованными методами (`Ljeh->getValue` ≤5 stоковых callsite), классы из других dex (`Lek9->d/Lek9->H` — 10+ стоковых callsite, `Lzx4->a0`, `Lsl9->a`), framework `org.json` |

## 4. Fan-out аудит всех активных `apply_v_*.py` (13 групп)

Проведён read-only аудит 13 тематических групп (~270 скриптов из `APPLY_STEPS` + `E2E_ONLY_STEPS` + `CALL_E2E_ONLY_STEPS`): sentinel'ы приземлены, якоря живы, defined-but-forgotten не обнаружен.

**Итог: реальных активных багов 0.** Все вердикты — PASS либо SKIP-штатно (закомментирован/вычеркнут/DEFERRED в `make_v1.py`).

## 5. Проверка маркеров в собранном APK (DEX)

По дизассемблированному кэшу `work/dexcache` (arm64 V16):

| Маркер | Статус |
|---|---|
| `gateUnread` | FOUND (AppFlags:4082 + 3 callsite) |
| `getAccentColor` | FOUND (ModFab + DialogsUtils + 9+ callsite) |
| `rrRecord` | FOUND (AppFlags:8354 + n55:2847) |
| `amoledBg` | FOUND (20+ callsite) |
| `filterSettingsListCopy` | FOUND (AppFlags:3558 + xhg:2277) |

Патчи физически доехали до DEX.

## 6. Device-верификация (мини smoke, Redmi `21091116AG`)

- `install -r` main + clone: `Success`.
- Запуск main (`ru.oneme.app`) и clone (`ru.oneme.ap2`): процессы живы, logcat чист (нет FATAL / VerifyError / краша).
- Активное окно после пробуждения: clone `one.me.android.IconAliasOrange`.

**Ограничение**: мини-smoke (установка + запуск + отсутствие крашей). Полный функциональный smoke (ручные тумблеры фич, E2E-сценарий) в рамках этого аудита не проводился.

## 7. Известные deferred-долги (не блокируют V16)

| Область | Статус |
|---|---|
| call-E2E | Active шаги несут guard-skip/R8-drift; фактическое приземление на текущем стоке не доказано; `CALL_LOCK_UI=false` — намеренно (безопасный режим) |
| `apply_v_antidel_isread_fix.py` | DEFERRED: якоря 26.29.1 (`Lgja→Lgva`, `Lzwe->b→Lajf->a`, `Lsia→Lsua`) |
| `apply_v_ghost_presence.py` | DEFERRED: verify-токены протухли под 26.31.0 |
| `apply_v_e2e_silent_send.py` | Инертен по дизайну (stub, sendSilentText нейтрализован) до V1.1 |
| Chat export, Share logs | Вычеркнуты из пайплайна (внешние фичи не входят в V16) |

## 8. Косметика (не баги, задокументировано)

- Устаревшие имена в комментариях/docstring части скриптов (пути c2→c3, имена `wr0/ta` → `es3/b03` и т.п.) — код корректен, текст протух.
- `verify_dex_markers.py` не запускается в данном окружении (зависание вывода, кэш-HIT корректно читается только вручную) — заменено прямым grep по `work/dexcache`.

## 9. Вердикт

Сборка и состав V16 — корректны. Активных багов пайплайна и гейтов не обнаружено. Публикация — по отдельной явной команде пользователя (предусловие: gh-авторизация и полный smoke по чеклисту).