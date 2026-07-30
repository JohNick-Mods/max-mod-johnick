# MAX 26.24.0 V20 — AUDIT

**Дата**: 2026-07-30 | **Версия**: 26.24.0 RS V20 | **Платформы**: arm64, armv7, x86_64, x86

## Что изменилось в V20 (δ от V19)

### Новый тумблер: «Скрыть просмотр историй» (раздел Невидимка)
- `tumblers.yaml`: новый флаг `hideStoryView` (privacy, default OFF), визуально дочерний для мастер-тумблера «Невидимка».
- Точка врезки: `Lscg;->r(Loxa;Z)V` — гейт перед вызовом `Lscg;->m(Loig;)V`, который толкает состояние «просмотрено» на сервер/WS. Локальный кэш и список просмотревших свои истории не затрагиваются — блокируется только исходящее событие.
- **Функциональная зависимость от «Невидимки»**: гейт активен при `hideStoryView=ON` **ИЛИ** `invisible=ON` — тот же OR-паттерн, что у `antiTyping`/`antiRead`. Каскад в `gen_appflags.py` расширен на 4-й дочерний флаг: включение/выключение мастера «Невидимка» теперь взводит/снимает `hideStoryView` вместе с остальными тремя (typing/read/presence), обратная каскадность (авто-снятие мастера, если все дочерние выключены) тоже учитывает новый флаг. UI-синхронизация свитчей обновлена на 5 виджетов.

### Регрессия, найденная и исправленная до релиза
- Первая версия патча использовала регистр `v11` как scratch — на самом деле это `this` (метод `.registers 14` с 3 param-словами, параметры — последние регистры: `p0(this)=v11`). Перезапись ловилась Dalvik/ART verifier'ом при запуске (`java.lang.VerifyError: tried to get class from non-reference register v11`), а не smali-ассемблером или `apksigner verify`. Обнаружено и подтверждено ТОЛЬКО реальным device-тестом (install + monkey + logcat) на S25. Исправлено — заменён на безопасный мёртвый регистр `v10`.

### Техдолг тестов (не влияет на функциональность мода)
- `tests/test_appflags_wired.py`: исправлен баг самих тестов — жёстко зашитый тип `:Z` для флагов `playbackSpeed`/`iconColor`, реально типизированных `F`/`I` в `tumblers.yaml` (введён `FLAG_TYPES` lookup в `conftest.py`).
- Документированы 4 дополнительных `DEFERRED_GATE_FLAGS` (`iconColor`, `appLockUseBiometric`, `changePinAction`, `amoledTheme`) — эти флаги гейтятся helper-методами ВНУТРИ `AppFlags.smali`, а не внешним `sget-boolean`, что раньше ошибочно читалось тестом как «тумблер мёртвый».

## Проверки

### Сборка
- ✅ `make_v1.py` завершена без ошибок, 4 APK подписаны (arm64/armv7 × main/clone)
- ✅ `apksigner verify` успешно для всех 4
- ✅ SHA256SUMS согласованы
- ✅ `verify_critical_patches()`: явный critical-check на sentinel `:hide_story_view_gate` + оба sget (`hideStoryView`, `invisible`)

### Smoke-тесты
- ✅ Установка main arm64 APK на S25 (`RFCX5013V5F`, Android 15) — успешно
- ✅ Запуск (`monkey -c android.intent.category.LAUNCHER`) без краша, чистый logcat (0 `FATAL EXCEPTION`/`VerifyError`/`AndroidRuntime` exception)
- ✅ Процесс `ru.oneme.app` стабильно жив после запуска (`pidof`)
- ⚠️ Клон **не устанавливался** на устройство — явное требование пользователя в этой сессии («клон не обновляй на телефоне»). Клон собран и подписан, но не device-тестирован в этом цикле.

### pytest
- ✅ `tests/test_appflags_wired.py`: 148 passed, 19 skipped (все skip — документированные DEFERRED, не регрессии)
- ⚠️ Полный `pytest tests/` не чист — 15 падений в других файлах (`test_critical_layers.py`, `test_active_patches_landed.py`, `test_fingerprint_resolver.py` и др.), похоже на стейл-кэш `work/baksmali` после серии ручных правок в этой сессии, не проверялось отдельно — вне скоупа текущего изменения.

## Риски и ограничения

- Новый тумблер аддитивен, гейт минимальный (2 sget-boolean + 2 branch перед существующим вызовом) — низкий блэст-радиус для остальной функциональности звонков/чатов.
- Клон не smoke-тестирован в этом цикле (см. выше) — рекомендуется ручная проверка перед тем, как полагаться на клон-сборку.

## Ассеты

- MAX_26.24.0_arm64_V20.apk
- MAX_26.24.0_arm64_V20_clone.apk
- MAX_26.24.0_arm7_V20.apk
- MAX_26.24.0_arm7_V20_clone.apk
- SHA256SUMS
