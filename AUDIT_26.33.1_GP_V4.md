# MAX 26.33.1 GP V4 — AUDIT

**Дата:** 2026-10-02
**Версия:** 26.33.1 GP V4 (`MIRROR_VERSION_TAG = "26.33.1.4"`, `VERSION=26.33.1`, `BUILD_NUM=4`)
**Сборка:** `make_v1.py --strict --e2e --skip-anti-split --skip-baksmali` (incremental поверх V3, 02.10.2026, exit 0; 270 шагов, все 4 APK собраны и подписаны)
**Автор:** JohNick (Mods by JohNick)

---

## 1. Что нового в V4

Четвёртый выпуск на стоке MAX 26.33.1 — исправление ложной метки «удалено», подтверждённой репортом юзера:

- **2331-F27 — при antiDelete=ON обычные входящие/канальные сообщения сразу получают пометку «удалено» — ✅ FIXED.** Root cause: общий FCM parser `c2/w0f.smali::c(Lflf;JJLxye;)V` вызывает `AppFlags.addDeletedKept(J)` для ЛЮБОГО payload с `msgid` (патч `apply_v_antidel_offline_kept.py`, sentinel `:antidel_offline_kept_v3`). Метод generic и вызывается из нескольких FCM-путей до классификации типа удаления. Фикс: type-gate через уже прочитанный `v5` (payload type) и null-safe `Lvu8;->c(...)Z` — `addDeletedKept` выполняется ТОЛЬКО для точных `"MessageRemoved"` и `"ChatMessageRemoved"`; `"ChatMessageRemoved-channel"` исключён (источник ложных меток у канальных публикаций). Обычные входящие и канальные push больше не попадают в `deletedKept`. Подтверждено юзером на устройстве (smoke PASS), dex-дизассемблером установленного APK (гейт содержит только 2 типа, `addDeletedKept` — 1 вызов строго под гейтом).

Инфраструктурная доводка:
- `apply_v_bg_wake_alarm_bypass_autostart.py` — распознаёт существующий F12 direct-wake эквивалент (`:f12_alarm_direct_wake_v1`) и НЕ поднимает ложный anchor-FAIL на инкрементальных сборках (старый патч поглощён F12-фиксом по дизайну).
- Устаревшая документация в `apply_v_antidel_offline_kept.py` приведена к фактической точке врезки (общий `w0f.c()`, а не «специализированный remove-handler»).

## 2. Артефакты сборки

4 APK + 4 `.idsig` + `SHA256SUMS` в `RELEASE/26.33.1_GP_V4/`:

| Файл | Заметка |
|---|---|
| `MAX_26.33.1_arm64_V4.apk` | arm64, source |
| `MAX_26.33.1_arm64_V4_clone.apk` | arm64, clone |
| `MAX_26.33.1_arm7_V4.apk` | arm7, source |
| `MAX_26.33.1_arm7_V4_clone.apk` | arm7, clone |

## 3. Статические гейты (--strict)

| Гейт | Результат |
|---|---|
| undefined_method_gate | 0 findings — clean |
| table_registration_gate | 0 findings — clean |
| reg_collision_gate | 0 |
| r8_drift_gate | OK (V4 не менял R8-имена) |
| version_consistency_gate | OK (7 файлов, VERSION=26.33.1) |
| dex_refs_monitor | c1 58 205 (88.8% OK) / c2 62 644 (95.6% WARN) / c3 28 862 (44.0% OK) |
| partial_fail / soft_fail | 0 / 0 |
| APK sha256sums | сгенерированы |

## 4. Device smoke (Redmi HyperOS, serial WGJBBEY9SSMN8TIB)

- `install -r` V4 arm64 main (`ru.oneme.app`) + clone (`ru.oneme.ap2`) → Success (оба).
- Device-side SHA-256 совпал с `RELEASE/26.33.1_GP_V4/SHA256SUMS` (main `8eb9b1c0…`, clone `69a7ec96…`) — без stale-dex.
- Оба приложения запускаются (PID живы), `logcat -b crash` — пуст.
- **Независимая dex-верификация фикса F27:** установленный `base.apk` дизассемблирован `verify_dex_markers.py` → `classes2/w0f.smali`: гейт содержит только `"MessageRemoved"`/`"ChatMessageRemoved"`, `addDeletedKept(J)` — 1 вызов строго под гейтом, `"ChatMessageRemoved-channel"` в патче отсутствует (единственное вхождение — стоковый dispatcher, не в гейте).
- UI (uiautomator): список чатов и открытый чат БЕЗ маркеров «удалено»/`❌`; канал без ложных меток.
- **2331-F27 подтверждён юзером на устройстве (smoke PASS).**

## 5. Fan-out аудит патчей (HARD-RULE)

По решению пользователя для V4 fan-out LLM-аудит НЕ проводился (явная команда «fan audit не нужен»). Изменения в V4 минимальны и локальны: только `apply_v_antidel_offline_kept.py` (type-gate) и `apply_v_bg_wake_alarm_bypass_autostart.py` (идемпотентность на инкременталке). Оба покрыты статическими гейтами сборки `--strict` (BUILD_EXIT=0), sentinel_audit и независимой dex-верификацией установленного APK; smoke подтверждён юзером на устройстве.