# MAX Mod by JohNick

[![Latest Release](https://img.shields.io/github/v/release/JohNick-Mods/max-mod-johnick?label=Последняя%20версия&color=blue)](https://github.com/JohNick-Mods/max-mod-johnick/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/JohNick-Mods/max-mod-johnick/total?label=Скачиваний&color=green)](https://github.com/JohNick-Mods/max-mod-johnick/releases)
[![Issues](https://img.shields.io/github/issues/JohNick-Mods/max-mod-johnick?label=Issues)](https://github.com/JohNick-Mods/max-mod-johnick/issues)
[![Discussions](https://img.shields.io/github/discussions/JohNick-Mods/max-mod-johnick?label=Обсуждения)](https://github.com/JohNick-Mods/max-mod-johnick/discussions)

Модифицированная сборка мессенджера **MAX** (`ru.oneme.app`) с упором на приватность, нейтрализацию модулей телеметрии и пользовательские UX-функции.

> Проект создан исключительно в **учебных и исследовательских целях** в области информационной безопасности. Все правки выполнены статическим перепатчем APK и подписаны собственным ключом разработчика. Демонстрирует техническую возможность анализа и нейтрализации модулей телеметрии — **не предназначен для нанесения ущерба правообладателю**.

📦 [**Скачать последнюю сборку →**](../../releases/latest)

---

## Актуальная версия — **26.33.1 GP V3**

Третья сборка на стоке MAX 26.33.1: исправлены подтверждённые юзером баги — краш «Цифрового ID» (A10), возвращение удалённых сообщений (F26), гудок дозвона на Android Go (C2), полноэкранные уведомления на ColorOS/OxygenOS (U6). Пройден полный fan-out-аудит всех 303 патчей.

**Что нового в V3 (2026-10-01):**
- 💳 **«Цифровой ID» (Госуслуги) больше не крашит** — VerifyError в LinkInterceptorActivity исправлен (A10, подтверждено на устройстве).
- 🗑️ **Удалённые сообщения не возвращаются после перезапуска** — свои удаления реально пишутся в БД; анти-удаление чужих работает (F26, подтверждено на устройстве).
- 📞 **Гудок дозвона останавливается после ответа на MTK/Android Go** — добавлен второй стоп-хук CallEngine (C2).
- 🔔 **«Полноэкранные уведомления» открываются корректно на ColorOS/OxygenOS/realmeUI** (U6).

Полный список изменений — [RELEASE_NOTES_v26.33.1.3.md](./RELEASE_NOTES_v26.33.1.3.md).

**Базовый APК:** MAX 26.33.1 (новый сток). Обновление с 26.33.1 V1/V2 / 26.31.0 RS / 26.32.1 GP — `install -r`, данные сохраняются.

---