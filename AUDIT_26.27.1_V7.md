# MAX 26.27.1 V7 — Технический аудит мода

**Версия стока**: 26.27.1 (build 6802)
**Версия мода**: V7 (Privacy Mod by JohNick)
**Дата**: 2026-08-20
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **V7**: релиз поверх V6. Главное — прозрачность обоев **внутри открытого чата** вынесена в отдельный независимый тумблер «Прозрачный фон в чатах», который работает даже когда общая «Прозрачная тема» выключена (список остаётся тёмным, обои видны только внутри чата). Плюс фикс: пикатель вложений (камера/галерея) в альбомном режиме больше не разъезжается — принудительно лочится в портрет. Весь функционал V6 (обои на списке чатов, тема-switcher, независимый гейт «Прозрачная тема») сохранён без изменений.

---

## 🆕 26.27.1 V7 — что нового

### 💬 «Прозрачный фон в чатах» — независимый тумблер
До V7 обои внутри открытого чата показывались только когда включена общая «Прозрачная тема» (тогда прозрачны и список, и звонки, и настройки). В V7 добавлен отдельный флаг `AppFlags.transparentThemeChats:Z` (default `false`, тумблер «💬 Прозрачный фон в чатах» в категории «🎨 ВИД»), который делает прозрачным **только внутренность чата**, не трогая список/звонки/настройки.

Техническая сложность: список чатов и экран чата (`ChatScreen`) живут в **одном** Conductor-контейнере `0x7f090939` — статически покрасить его непрозрачным нельзя (гаснет и чат). Решение — динамическое управление фоном контейнера по жизненному циклу экрана чата:

- `AppFlags.applyRootTransparency(View, I)` в `fqe.onThemeChanged` красит корневой `fqe` и контейнер в зависимости от режима: main ON → `fqe` alpha `0x99`, контейнер прозрачный; **chat-only** (chats ON, main OFF) → `fqe` прозрачный, контейнер **непрозрачный** (список тёмный); сток → `fqe` непрозрачный.
- `ChatScreen.onAttach` → `AppFlags.chatContainerAttach(View)`: при режиме chat-only делает контейнер `0x7f090939` прозрачным (видны обои внутри чата), инкрементит счётчик вложенности `chatBgDepth`.
- `ChatScreen.onDetach` → `AppFlags.chatContainerDetach(View)`: декрементит счётчик, при возврате к списку красит контейнер обратно непрозрачным (`rootThemeColor`).

Оконная прозрачность (`FLAG_SHOW_WALLPAPER` + translucent theme) в `MainActivity.onCreate` теперь включается при **любом** из двух флагов (`transparentTheme || transparentThemeChats`) — OR-гейт в `applyTransparentIfEnabled` и `applyTransparentThemeEarly`.

Гейт обоев внутри чата (`ModsWallpaper.apply(ViewGroup)` и `t93.setBackground`) переведён на `transparentThemeChats`. Обои на списке чатов (`applyAsBackground`) остаются под `transparentTheme` — независимость сохранена.

Sentinel в собранном APK: `# :root_transparency_helper_v1` (хелперы в `AppFlags`), `# :chat_lifecycle_bg_v1` (2 хука в `ChatScreen`), `# :root_transparent_bg_v3` (`fqe.onThemeChanged`).

### 📷 Пикатель вложений в альбомном режиме — фикс layout
После восстановления auto-rotate (V5) пикатель «Фото и видео» (камера + галерея + Файл/Опрос/Место/Контакт) в горизонтальной ориентации разъезжался. В V7 при открытии пикателя ориентация принудительно лочится в PORTRAIT, при закрытии — восстанавливается USER.

- `ChatScreen.k2` (open-hook пикателя): `setRequestedOrientation(PORTRAIT = 0x1)` при каждом открытии.
- `jb.h:pswitch_31` (close-hook): восстановление `setRequestedOrientation(USER = 0x2)`.

Проверено на S25 (device-verify 2026-08-20): в альбомном режиме тап по скрепке → пикатель открывается корректно в портрете, сетка галереи и нижние вкладки рендерятся без разъезжания.

---

## 🆕 Унаследовано из 26.27.1 V6 (без изменений)

### 🖼 Обои чата на списке чатов
`ModsWallpaper.applyAsBackground(View)V` ставит `BitmapDrawable` (`Gravity.FILL`) через `View.setBackground(...)` на root-View `ChatsTabWidget.onCreateView`. Не оверлеит содержимое — `RecyclerView`, табы папок и навбар функциональны. Гейт — `AppFlags.transparentTheme:Z`. Sentinel `# :cl_wallpaper_bg_v1`.

### 🎨 «Прозрачная тема» — тумблер-гейт обоев списка
Флаг `AppFlags.transparentTheme:Z` (default `false`): ON — обои списка видны, OFF — no-op (файл сохраняется). Sentinel `# :wp_gate_v1`.

### 🧹 Кнопки тем «Свет / Тёмная / AMOLED»
Применение темы через `ContextWrapper.getBaseContext()`-unwrap-loop до реальной `Activity` + `recreate()` — тема меняется на всех главных экранах. Sentinel `:ocv_theme_unwrap_loop`.

---

## 🆕 Унаследовано из 26.27.1 V5 (без изменений)

### 🔄 Альбомный/горизонтальный режим восстановлен
В `MainActivity.y(Ljava/lang/Boolean;)V` форсируется `const/4 v3, 0x1` → `setRequestedOrientation(USER)`. Приложение следует повороту сенсора.

### 📌 Лимит закреплённых чатов снят по умолчанию
`unlockPinLimit` захардкожен `true` в `AppFlags.<clinit>`; `pinLimit(I)I` возвращает `Integer.MAX_VALUE`. Тумблер `hidden`.

⚠️ Ограничение серверное: сервер может сбросить лишние пины при синхронизации.

---

## 🆕 Унаследовано из 26.27.1 V4 (без изменений)

### 🔒 PIN-защита чата: содержимое не видно за диалогом
`ChatPinManager.hideView(View)` → `View.INVISIBLE` (static `WeakReference`); `revealView()` только на верном PIN; `dropHidden()` + `finishActivity()` на неверном; `abortHidden()` в catch. Минимум 4 цифры при установке.

### 📌 Порядок закреплённых чатов сохраняется после перетаскивания
`PinDragCb.n()`: инверсия `if-eqz → if-nez`. Порядок — `AppFlags.putString("pinned_order", csv)`.

### 📶 Тумблер «Удерживать связь в фоне» (`bgWatchdog`)
Настройки → Звонки и система. По умолчанию ВКЛЮЧЁН. Escape-hatch для MIUI/HyperOS.

---

## 🆕 Унаследовано из более ранних версий (без изменений)
- 🔑 **Chat-PIN** (V2/V3): `locked_chats_csv` + `ChatPinReceiver` + long-press заголовка.
- 📌 **Drag ACTION_DOWN consume** (V3): `PinHandleTouch.onTouch` → `true`.
- 🔔 **Гудок дозвона** (V2): `stop()` перед `Connection.destroy()` в `qe1` + `h75.onCallAccepted()`.
- 📜 **Прокрутка чата под невидимкой** (V2): якоря `bta`/`Lv2f;->g:J`.
- 🔒 **Нейтрализация телеметрии** (V2): push-транспорт, aggregation, operator-fetch (IP-strip), DPS.
- 🛡 **Обход App Lock** (V1): `onStart`-гейт + `appLockGateActive:Z`.
- 🔐 **«О приложении» → настройки мода** (V1): long-press.
- ❌ **Визуальная метка удалённых сообщений** (26.26.0 V5): красный крестик + `setAlpha 0.5f`.
- 📜 **Форс-консьюм при открытии чата** (26.26.0 V5): гейт `smartAntiRead || antiRead`.

---

## 🛡 Профиль приватности

### Невидимка (`invisible`) — мастер-переключатель

| Дочерний флаг | Что делает |
|---|---|
| `antiTyping` | собеседник не видит набор текста |
| `antiRead` | собеседник не видит отметку «прочитано» |
| `smartAntiRead` | «прочитано» уходит только после вашего ответа (consume-on-use) |
| `antiPresence` | онлайн / last-seen скрыты |

### Антиудаление (`antiDelete`) — 4 слоя
1. **L3** — Central WS chat-event. 2. **L4** — Room ORM body-clear. 3. **Маркер** ❌ + время удаления. 4. **FCM офлайн-персист**.

### Нейтрализация телеметрии
Чокпоинты в push-транспорте, aggregation-логах, operator-fetch (IP-strip), DPS-пинге.

---

## 🔍 Верификация (V7)

Независимая адверсариальная кросс-верификация — статический анализ smali по 3 DEX собранного `MAX_26.27.1_arm64_V7.apk` (baksmali) + device-verify на S25:

| Проверка | Результат |
|---|---|
| `AppFlags` содержит поля `transparentThemeChats:Z`, `rootThemeColor:I`, `chatBgDepth:I` + хелперы `applyRootTransparency` / `chatContainerAttach` / `chatContainerDetach` / `setContainerBg` / `artSetScreenBg` (sentinel `# :root_transparency_helper_v1`) | ✅ подтверждено |
| `ChatScreen.onAttach`/`onDetach`: `invoke-static ...chatContainerAttach/Detach` сразу после `invoke-super` (sentinel `# :chat_lifecycle_bg_v1`, 2 хука) | ✅ подтверждено |
| `fqe.onThemeChanged`: `invoke-static ...applyRootTransparency(Landroid/view/View;I)V` вместо прямого `setBackgroundColor` (sentinel `# :root_transparent_bg_v3`) | ✅ подтверждено |
| Оконная прозрачность: OR-гейт `transparentTheme \|\| transparentThemeChats` в `applyTransparentIfEnabled` и `applyTransparentThemeEarly` | ✅ подтверждено |
| Гейт обоев чата (`ModsWallpaper.apply` + `t93.setBackground`) на `transparentThemeChats`; гейт списка (`applyAsBackground`) на `transparentTheme` — независимость | ✅ подтверждено |
| Пикатель вложений: `ChatScreen.k2` lock PORTRAIT (`0x1`) при открытии, `jb.h:pswitch_31` restore USER (`0x2`) при закрытии | ✅ подтверждено |
| Device-verify S25: 4 комбинации флагов (off/off, off/on, on/on, on/off) — список тёмный при chat-only, обои внутри чата видны, возврат к списку тёмный | ✅ PASS |
| Device-verify S25: пикатель вложений в альбомном режиме открывается в портрете без разъезжания layout | ✅ PASS |
| `MainActivity.y()`: `const/4 v3, 0x1` перед `if-eqz v3` (auto-rotate USER) | ✅ подтверждено (V5) |
| `AppFlags.pinLimit()I`: при `unlockPinLimit=true` (дефолт) → `const v0, 0x7fffffff` | ✅ подтверждено (V5) |
| `ChatPinManager`: `hideView` / `revealView` / `dropHidden` / `abortHidden` + `hiddenRef:WeakReference` | ✅ подтверждено (V4) |
| Регистровый аудит apply-скриптов по классам коллизий | ✅ 0 COLLISION |
| `apksigner verify` (все 4 APK) | ✅ v2+v3, exact ABI, distinct SHA |

---

## 📦 Артефакты (V7)

| Файл | SHA-256 |
|---|---|
| `MAX_26.27.1_arm64_V7.apk` | `c4eb4dd5162c09debe715df73823a973aec0a99c63e5eac95ca356bf8c0ba7af` |
| `MAX_26.27.1_arm64_V7_clone.apk` | `079fef109f8bcf74f52d16cf2784f5dce443fc534deda2e49ed5e8e3c6e777df` |
| `MAX_26.27.1_arm7_V7.apk` | `29e55d3e12503b285dff29f8531202673245d02b627cfe710fd36870d11d1e4e` |
| `MAX_26.27.1_arm7_V7_clone.apk` | `741ea1cebae242b700358e89d6617af4bf6f77212a3871f2f0436f77853ff8f0` |

Подпись: v2+v3 APK Signature Scheme. Cert SHA-1 идентичен всей линейке 26.x — `install -r` без потери данных.

---

*Аудит выполнен статическим анализом smali (baksmali + независимый агент-верификатор) по декомпиляции фактически собранных APK. Динамический smoke — на S25 (RFCX5013V5F) через adb: независимая прозрачность чатов (4 комбинации флагов) и пикатель вложений в альбомном режиме.*
