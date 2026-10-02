# MAX Mod by JohNick

[![Latest Release](https://img.shields.io/github/v/release/JohNick-Mods/max-mod-johnick?label=Последняя%20версия&color=blue)](https://github.com/JohNick-Mods/max-mod-johnick/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/JohNick-Mods/max-mod-johnick/total?label=Скачиваний&color=green)](https://github.com/JohNick-Mods/max-mod-johnick/releases)
[![Issues](https://img.shields.io/github/issues/JohNick-Mods/max-mod-johnick?label=Issues)](https://github.com/JohNick-Mods/max-mod-johnick/issues)
[![Discussions](https://img.shields.io/github/discussions/JohNick-Mods/max-mod-johnick?label=Обсуждения)](https://github.com/JohNick-Mods/max-mod-johnick/discussions)

Модифицированная сборка мессенджера **MAX** (`ru.oneme.app`) с упором на приватность, нейтрализацию модулей телеметрии и пользовательские UX-функции.

> Проект создан исключительно в **учебных и исследовательских целях** в области информационной безопасности. Все правки выполнены статическим перепатчем APK и подписаны собственным ключом разработчика. Демонстрирует техническую возможность анализа и нейтрализации модулей телеметрии — **не предназначен для нанесения ущерба правообладателю**.

📦 [**Скачать последнюю сборку →**](../../releases/latest)

---

## Актуальная версия — **26.33.1 GP V4**

Четвёртая сборка на стоке MAX 26.33.1: исправлена подтверждённая юзером ложная пометка «удалено» у обычных входящих/канальных сообщений при включённом анти-удалении (F27). Изменения минимальны, покрыты статическими гейтами сборки и подтверждены на устройстве.

**Что нового в V4 (2026-10-02):**
- 🗑️ **Исправлена ложная метка «удалено»** — общий FCM-обработчик больше не заносит любой `msgid` в список защищённых удалённых; `addDeletedKept` выполняется только для настоящих уведомлений об удалении (`MessageRemoved` / `ChatMessageRemoved`). Канальные публикации больше не помечаются ошибочно (F27, подтверждено на устройстве + dex-верификация).

Полный список изменений — [RELEASE_NOTES_v26.33.1.4.md](./RELEASE_NOTES_v26.33.1.4.md).

**Базовый APК:** MAX 26.33.1 (новый сток). Обновление с 26.33.1 V1/V2 / 26.31.0 RS / 26.32.1 GP — `install -r`, данные сохраняются.

---