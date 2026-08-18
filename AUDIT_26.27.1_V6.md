# MAX 26.27.1 V6 — Технический аудит мода

**Версия стока**: 26.27.1 (build 6802)
**Версия мода**: V6 (Privacy Mod by JohNick)
**Дата**: 2026-08-18
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **V6**: релиз поверх V5. Впервые в публикуемой версии — обои чата на списке чатов (`ChatsTabWidget` root) через `View.setBackground(BitmapDrawable)`. Переработаны тумблеры темы: «Прозрачная тема» отделена от применения обоев и стала независимым переключателем-гейтом. Весь функционал V5 (auto-rotate USER, `pinLimit` ∞, PIN-приватность, drag-persist, `bgWatchdog`) сохранён без изменений.

---

## 🆕 26.27.1 V6 — что нового

### 🖼 Обои чата — теперь и на списке чатов
До V6 выбранные обои (`ModsWallpaper.file`) применялись только внутри открытого чата (`ChatScreen`, root `ViewGroup`). Список чатов, вкладки «Все / Новые / Личные / Каналы» показывались на стандартном фоне темы.

В V6 добавлен второй hook: `ModsWallpaper.applyAsBackground(View)V` — читает файл обоев, декодирует `Bitmap`, оборачивает в `BitmapDrawable` (`Gravity.FILL = 0x77`) и ставит через `View.setBackground(...)` на root-View `ChatsTabWidget.onCreateView(...)`. Реализация в отличие от `addView` не оверлеит содержимое — фон рисуется в `View.draw` до `dispatchDraw`, поэтому `RecyclerView` списка чатов, табы папок и нижний навбар остаются полностью функциональными.

Два инжект-пойнта: перед `return-object v5` (ветка `FrameLayout`, портрет) и перед `return-object p1` (ветка `Lte4`, master-detail планшеты). Оба гейтятся на `AppFlags.transparentTheme:Z` — при выключенном тумблере `applyAsBackground` возвращается из первой инструкции (`sget-boolean` + `if-nez` + `return-void`), фон списка не меняется.

Sentinel в собранном APK: `# :cl_wallpaper_bg_v1` (2 инжекта в `ChatsTabWidget`), `# :wp_gate_v1` (2 гейта в `ModsWallpaper`).

### 🎨 «Прозрачная тема» — теперь независимый тумблер-гейт
Раньше подпункт «Прозрачная тема» технически включал/выключал ту же логику что и «Выбрать обои» — путал и оставлял «мёртвые» состояния (файл обоев есть, но не рисуется). В V6 роль тумблера чёткая: **выключатель обоев без удаления файла**.

- Флаг: `AppFlags.transparentTheme:Z`, default `false`, тумблер в категории «🎨 ВИД».
- ON: `ModsWallpaper.apply()` (внутри чата) и `applyAsBackground()` (root списка) выполняются как обычно.
- OFF: обе функции no-op (`return-void` из первой инструкции). Файл обоев остаётся на диске — можно временно вернуть стандартный фон темы, не выбирая обои заново.
- Не меняет системные `windowIsTranslucent`/`windowShowWallpaper` (эти флаги статичны в манифесте и не редактируются в рантайме).

### 🧹 Переработаны кнопки тем «Свет / Тёмная / AMOLED»
Три вкладки в категории «🎨 ВИД» — независимые пресеты. Ветвление применения темы теперь идёт через `ContextWrapper.getBaseContext()`-unwrap-loop: `AlertDialog.getContext()` возвращает `ContextThemeWrapper`, `instance-of Activity` на нём всегда `0` — до V6 `recreate()` не вызывался, тема применялась только к диалогу настроек, не к главным экранам. Фикс: unwrap-цикл до реальной `Activity`, затем `recreate()` — тема применяется на списке чатов и внутри чата сразу.

---

## 🆕 Унаследовано из 26.27.1 V5 (без изменений)

### 🔄 Альбомный/горизонтальный режим восстановлен
В `MainActivity.y(Ljava/lang/Boolean;)V` перед `if-eqz v3, :cond_89` форсируется `const/4 v3, 0x1` — переход всегда идёт на `const/4 v1, 0x2` → `setRequestedOrientation(USER)`. Приложение снова следует повороту сенсора.

### 📌 Лимит закреплённых чатов снят по умолчанию
Флаг `unlockPinLimit` захардкожен в `AppFlags.<clinit>` как `true`. Метод `pinLimit(I)I` при `unlockPinLimit=true` возвращает `Integer.MAX_VALUE`. Тумблер `hidden: true` (в UI нет), лимит снят «из коробки».

⚠️ Ограничение серверное: сервер может сбросить лишние пины при синхронизации между устройствами.

---

## 🆕 Унаследовано из 26.27.1 V4 (без изменений)

### 🔒 PIN-защита чата: содержимое больше не видно за диалогом
Перед показом PIN-диалога `ChatPinManager.hideView(View)` переводит root-View чата в `View.INVISIBLE`, ссылка в static `WeakReference`. `revealView()` вызывается **только на пути верного PIN**. Неверный PIN — `dropHidden()` + `finishActivity()` без «мигания» чата. `abortHidden()` в catch-блоке. При первичной установке PIN — проверка минимум 4 цифр.

### 📌 Порядок закреплённых чатов сохраняется после перетаскивания
`PinDragCb.n()`: инверсия `if-eqz → if-nez` во второй проверке типа ViewHolder. Порядок сохраняется в `AppFlags.putString("pinned_order", csv)`.

### 📶 Тумблер «Удерживать связь в фоне» (`bgWatchdog`)
Настройки → Звонки и система → **«Удерживать связь в фоне»**. По умолчанию ВКЛЮЧЁН. Escape-hatch для MIUI/HyperOS.

---

## 🆕 Унаследовано из более ранних версий (без изменений)
- 🔑 **Chat-PIN** (V2/V3): защита отдельных чатов через `AppFlags.putString("locked_chats_csv")` + `ChatPinReceiver` (ADB broadcast) + long-press заголовка.
- 📌 **Drag ACTION_DOWN consume** (V3): `PinHandleTouch.onTouch` возвращает `true`.
- 🔔 **Гудок дозвона** (V2): `stop()` перед `Connection.destroy()` в `qe1` + `h75.onCallAccepted()`.
- 📜 **Прокрутка чата под невидимкой** (V2): якоря `bta`/`Lv2f;->g:J`.
- 🔒 **Нейтрализация телеметрии** (V2): чокпоинты в push-транспорте, aggregation, operator-fetch (IP-strip), DPS.
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

## 🔍 Верификация (V6)

Независимая адверсариальная кросс-верификация — статический анализ smali по 3 DEX собранного `MAX_26.27.1_arm64_V6.apk` (baksmali):

| Проверка | Результат |
|---|---|
| `ModsWallpaper` содержит методы `apply(Landroid/view/ViewGroup;)V` и `applyAsBackground(Landroid/view/View;)V` | ✅ подтверждено |
| Гейт `AppFlags->transparentTheme:Z` + `if-nez` + `return-void` в первой инструкции обоих методов (sentinel `# :wp_gate_v1` — 2 вхождения) | ✅ подтверждено |
| `ChatsTabWidget.onCreateView`: два `invoke-static Lone/me/mods/ModsWallpaper;->applyAsBackground(...)V` перед `return-object v5` (FrameLayout) и `return-object p1` (Lte4 master-detail) (sentinel `# :cl_wallpaper_bg_v1`) | ✅ подтверждено |
| Тема-switcher: unwrap-loop `ContextWrapper.getBaseContext()` до реальной `Activity` перед `recreate()` (sentinel `:ocv_theme_unwrap_loop`) | ✅ подтверждено |
| `MainActivity.y()`: `const/4 v3, 0x1` форсируется перед `if-eqz v3, :cond_89` (auto-rotate USER) | ✅ подтверждено (V5) |
| `AppFlags.pinLimit()I`: при `unlockPinLimit:Z = true` (дефолт) возвращает `const v0, 0x7fffffff` | ✅ подтверждено (V5) |
| `ChatPinManager`: `hideView` / `revealView` / `dropHidden` / `abortHidden` + `hiddenRef:WeakReference` | ✅ подтверждено (V4) |
| `PinDragCb.n()`: вторая проверка ViewHolder — `if-nez v1, :pd_ok_a` | ✅ подтверждено (V4) |
| `KeepAliveManager.ensureServiceRunning`/`.enable`: `sget-boolean AppFlags->bgWatchdog:Z` гейт | ✅ подтверждено (V4) |
| Регистровый аудит apply-скриптов по 8 классам коллизий | ✅ 0 COLLISION |
| `apksigner verify` (все 4 APK) | ✅ v2+v3, exact ABI, no tracer, distinct SHA |
| Динамический smoke на HA218XJZ (Android-MCP): UI-тумблер, навигация Чаты→ПРИВАТ→ВИД, темы Свет/Тёмная/AMOLED, «Прозрачная тема» переключается | ✅ PASS |

---

## 📦 Артефакты (V6)

| Файл | SHA-256 |
|---|---|
| `MAX_26.27.1_arm64_V6.apk` | `c21c30190640e9eb4e01d7bbf22442712cb8667edf6689dd2ea4396802995028` |
| `MAX_26.27.1_arm64_V6_clone.apk` | `ad6c498e2c81ac2fc8d6bd6e7c49748989bab911f12ebaef39c31a9fc7695458` |
| `MAX_26.27.1_arm7_V6.apk` | `8443bec78f9b3bb40cb1a70f5ed42338a0a500372976a92260d65a781d9b98f2` |
| `MAX_26.27.1_arm7_V6_clone.apk` | `0d27a4021f99c9fa336f7bd55d983ee3058b5fb5b93468e7774b07af7a483c48` |

Подпись: v2+v3 APK Signature Scheme. Cert SHA-1 идентичен всей линейке 26.x — `install -r` без потери данных.

---

*Аудит выполнен статическим анализом smali (baksmali + независимый агент-верификатор) по декомпиляции фактически собранных APK. Динамический smoke — на планшете HA218XJZ через Android-MCP: UI-тумблер, навигация, темы, «Прозрачная тема».*
