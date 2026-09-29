# MAX Mod by JohNick

[![Latest Release](https://img.shields.io/github/v/release/JohNick-Mods/max-mod-johnick?label=Последняя%20версия&color=blue)](https://github.com/JohNick-Mods/max-mod-johnick/releases/latest)
[![Downloads](https://img.shields.io/github/downloads/JohNick-Mods/max-mod-johnick/total?label=Скачиваний&color=green)](https://github.com/JohNick-Mods/max-mod-johnick/releases)
[![Issues](https://img.shields.io/github/issues/JohNick-Mods/max-mod-johnick?label=Issues)](https://github.com/JohNick-Mods/max-mod-johnick/issues)
[![Discussions](https://img.shields.io/github/discussions/JohNick-Mods/max-mod-johnick?label=Обсуждения)](https://github.com/JohNick-Mods/max-mod-johnick/discussions)

Модифицированная сборка мессенджера **MAX** (`ru.oneme.app`) с упором на приватность, нейтрализацию модулей телеметрии и пользовательские UX-функции.

> Проект создан исключительно в **учебных и исследовательских целях** в области информационной безопасности. Все правки выполнены статическим перепатчем APK и подписаны собственным ключом разработчика. Демонстрирует техническую возможность анализа и нейтрализации модулей телеметрии — **не предназначен для нанесения ущерба правообладателю**.

📦 [**Скачать последнюю сборку →**](../../releases/latest)

---

## Актуальная версия — **26.33.1 GP V2**

Вторая сборка на стоке MAX 26.33.1: закрыты пост-релизные баги F23 (краш красного текста) и F12 (фоновые уведомления), исправлена чистая пересборка пайплайна (reorder/hide-phone/sharelogs/dex-rebalance), пройден полный fan-out-аудит 8 параллельными агентами.

**Что нового в V2 (2026-09-29):**
- 🖍️ **Краш красного цвета текста (F23) исправлен** — VerifyError CodeSpan устранён, цветовая вставка E2E-текста переведена на span-сохраняющий путь.
- ⏰ **Фоновые уведомления (F12)**: исправлен BackgroundWakeBootReceiver, сторожевой alarm + прямой старт BackgroundListenService.
- 📋 **Панель вкладок**: fix чистой сборки — reorder и hide Digital ID совместимы, скрытие «Цифрового ID» работает в обоих порядках вкладок.
- 🔧 **Аудит пайплайна**: 8 групп устранены тихие no-op, частичные применения, переезды классов между dex-файлами.

Полный список изменений — [RELEASE_NOTES_v26.33.1.2.md](./RELEASE_NOTES_v26.33.1.2.md).

**Базовый APК:** MAX 26.33.1 (новый сток). Обновление с 26.33.1 V1 / 26.31.0 RS / 26.32.1 GP — `install -r`, данные сохраняются.

---