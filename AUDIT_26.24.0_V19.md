# MAX 26.24.0 V19 — AUDIT

**Дата**: 2026-07-30 | **Версия**: 26.24.0 RS V19 | **Платформы**: arm64, armv7, x86_64, x86

## Что изменилось в V19 (δ от V18)

### Компилирование и сборка
- Версия bump: V18 → V19 (`build_config.py`, `MIRROR_VERSION_TAG = "26.24.0.19"`)
- Шаблонизация иконок: **Clone-aware icon-switcher**
  - `gen_template.py`: добавлена парадигма `VARIANTS` (main/clone), каждый со своим enabled alias (Blue для main, Orange для clone)
  - `build_v1.py`: выбор variant-ей по флагу `clone` (lines ~52, ~1116)
  - Результат: main-версия показывает синюю иконку (как было), clone-версия показывает оранжевую иконку (багфикс регрессии V18)

### Функциональность (без изменений от V18)
- SMS-фикс: CallsSdkInitializer anti-tamper bypass с genuine DEX/SO-константами (V18)
- UpdateChecker, всё остальное — стабильно

## Проверки

### Сборка
- ✅ `make_v1.py` завершена без ошибок, 4 APK подписаны (arm64/armv7 × main/clone)
- ✅ `apksigner verify` успешно для всех 4
- ✅ SHA256SUMS согласованы

### Smoke-тесты
- ✅ Установка на S25 (arm64-v8a) основной версии — успешно
- ✅ Запуск без краша (logcat clean)
- ✅ Установка main+clone на Lenovo tablet (HA218XJZ arm64) — успешно, обе версии работают

### Иконки (clone-aware fix)
- ✅ Байтовое сравнение: `templates/patched/main/AndroidManifest.xml` vs `templates/patched/clone/AndroidManifest.xml` — differ (char 36917)
- ✅ `resources.arsc` идентичны (expected)
- ✅ aapt2 dump xmltree подтверждает:
  - main: IconAliasBlue enabled=true, все остальные false
  - clone: IconAliasOrange enabled=true, все остальные false
- ✅ adb cmd package resolve-activity на real device:
  - main (ru.oneme.app): IconAliasBlue (свежая установка)
  - clone (ru.oneme.ap2): IconAliasOrange (свежая установка)
- ✅ Independent adversarial verification: подтверждено субагентом (code pattern + binary inspection)

## Риски и ограничения

Нет. V19 — локальное улучшение поверх V18 (SMS-фикс + иконка-свитчер). Откатываться не нужно.

## Ассеты

- MAX_26.24.0_arm64_V19.apk
- MAX_26.24.0_arm64_V19_clone.apk
- MAX_26.24.0_arm7_V19.apk
- MAX_26.24.0_arm7_V19_clone.apk
- SHA256SUMS
