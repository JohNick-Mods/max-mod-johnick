# MAX 26.31.0 RS V3 — Технический аудит мода

**Версия стока**: 26.31.0 RS
**Версия мода**: V3 (Privacy Mod by JohNick)
**Дата**: 2026-09-16
**Совместимость**: `ru.oneme.app` (Android 8.0–16.0 / API 26–36) · arm64-v8a + armeabi-v7a · основной + клон (`ru.oneme.ap2`)

> **V3**: хотфикс-релиз поверх V1 (v26.31.0.1). Добавлено: фикс краша QR-сканера (A8), пустой список «Устройства» (F11), **фикс отправки видео (F12)**, синхронизация «Автообновления», 8 R8-drift-фиксов аудита, voice-download реактивирован. на базе стока 26.31.0 RS. Полный перенос (~202 патч-скрипта) с ребампом R8-маппинга. Устранено 7 крашей нового стока, 5 багов E2E-шифрования (в т.ч. критическое авто-включение без согласия), 1 краш звонков, 7 UI-регрессий, 10 функциональных дефектов. Гонка мутации E2E-сессии закрыта синхронизацией. Предыдущий публичный релиз — 26.29.1 V12 (2026-09-10).

---

## 🔧 Краши нового стока (A1–A7)

### 💥 Краш старта — тема приложения (A1)

**Проблема**: краш при первом запуске с `need Theme.AppCompat`. Ресурсный id темы приложения изменился между версиями стока (`0x7f120139 → 0x7f1200d3`). Хардкодный id в шаблоне указывал на отсутствующий ресурс.

**Фикс**: id берётся из `aapt2 dump` итогового собранного APK, не из шаблона. Id стока и id после пересборки не совпадают — добавлено правило в pipeline.

### 💥 Краш рендера Switch (A2)

**Проблема**: `abc_control_background_material not found` при первом экране с Switch-элементом. Флаг `--optimize` / `--target-densities` в `build_v1.py` вырезал AppCompat-ресурсы.

**Фикс**: убран `aapt2 optimize` на шаге сборки (`build_v1.py:2734`).

### 💥 Краш клона при старте — отсутствующая строка (A3)

**Проблема**: клон падал с `string/folder_all not found`. Клон собирался с собственным arsc без ресурсов основного пакета.

**Фикс**: клон явно берёт main-arsc (`build_v1.py:2515`) с последующим байтовым ренеймингом пакета.

### 💥 Краш экрана звонка и спонтанные краши — dangling res-refs (A4)

**Проблема**: 36 оборванных файловых ссылок в arsc: пути `res/drawable/` ↔ `res/drawable-v23/` не совпадали с реальным layout в APK после оптимизации плотностей. ART не мог разрешить drawable-ресурс при инфляции View.

**Фикс**: `_reconcile_dangling_res_files()` в `build_v1.py` выравнивает пути в arsc по реальному содержимому ZIP.

### 💥 Краш профиля — чужой viewType (A5)

**Проблема**: `NoSuchElementException key 8192` при открытии профиля. `Ll0a;` decoration применялась к ViewHolder с viewType `0x2000`, не предназначенного для неё.

**Фикс**: skip-guard `instance-of La43;` перед decoration. Device-verified.

### 💥 VerifyError в ChatScreen / ManualReadButton / ModFab (A6)

**Проблема**: `VerifyError` при загрузке класса. Два независимых корня: (1) R8-drift — `requireActivity()` изменил дескриптор возвращаемого типа `()Lar; → ()Lvs;`; (2) wide-pair clobber в `nw7.d` — патч использовал регистр, занятый lower-half широкой пары.

**Фикс**: обновлены дескрипторы якорей; смещён патч-регистр. Оба gate (`reg_collision_gate`, `type_ref_gate`) прошли PASS.

### 💥 Краш экрана Оформление — отсутствующий атрибут (A7)

**Проблема**: `Настройки → Оформление` падало с ошибкой `colorSurface attribute not found`. MaterialButton требует атрибут `colorSurface` из MaterialTheme; после ресборки тема не разрешала его.

**Фикс**: устранён тем же reconcile config-путей arsc (A4). Device-verified S25: экран Оформление открывается штатно — размер текста, темы Системная/Светлая/Тёмная, превью чатов, набор «Космос»; `logcat -b crash` чист.

---

## 🔐 E2E-шифрование: исправления

### ⛔ Авто-включение E2E без согласия пользователя (E1, E2)

**Проблема**: при получении входящего push или cap-токена функции `setChatE2E(true)` активировались без действия пользователя. В чате появлялись ложные служебные плашки авто-включения. Opt-in превращался в opt-out.

**Фикс**: все вызовы `setChatE2E(true)` из push/cap-путей удалены. E2E активируется исключительно через UI (тумблер «Начать защищённый чат»). При первом запуске V1 выполняется одноразовый `resetAllE2EStrict()`, очищающий остаточные флаги предыдущих версий (`e2e_strict_/e2e_chat_flag_/e2e_chat_optin_/se_`).

### 🔁 Responder не пересобирал сессию при bundle-only reset (E4)

**Проблема**: когда инициатор сбрасывал E2E-сессию и отправлял новый bundle без REKEY-токена, сторона-ответчик не пересоздавала сессию, продолжая шифровать старой. Тип шифртекста оставался `type=2` (старая сессия).

**Фикс**: `rebuildAsResponderNewBundle` в `handlePeerBundle` — явная пересборка сессии при получении нового bundle. Флаг `e2e_applied_bundle_<peer>` предотвращает зацикливание. Device-verified S25↔Redmi: полный round-trip, `decrypt OK inType=2` / `decrypt=OK` на обоих устройствах, `decrypt failed=0`.

### 📤 Открытый текст при редактировании E2E-сообщения (E5)

**Проблема**: при редактировании ранее отправленного E2E-сообщения отредактированный текст уходил на сервер в открытом виде, минуя слой шифрования.

**Фикс**: `apply_v_e2e_block_edit.py` v4 — гейт в `c3/gk3.smali::g0`. chatId извлекается только через логируемый тег `gk3.p="gk3-<chatId>"` (функция `strictActiveForChatTag`). Device-verified S25.

### ⚙️ PEERVER_TOO_OLD — peer держал устаревший cap без TTL (E3)

**Проблема**: peer мог хранить устаревший cap-токен без TTL-сброса, что блокировало поднятие E2E-сессии с ошибкой `PEERVER_TOO_OLD`.

**Фикс**: `advertiseVersionForPeer(..., force=true)` без троттлинга на путях REKEY/no-session.

---

## 🔒 E2E: гонка мутации сессии (B2)

**Проблема**: `handlePeerBundle` — единственный метод, мутирующий E2E-сессию — не был синхронизирован. Неатомарная последовательность `hasSession → deleteSession → processPreKeyBundle` могла пересекаться с `decrypt()` из параллельного push-потока.

**Фикс**: `static synchronized void handlePeerBundle(...)` — единый монитор `E2ECrypto.class`, совпадает с монитором `encrypt/decrypt/processPreKeyBundle`. Дедлок исключён (реентрантный монитор, инверсии lock-order нет). Device-verified S25↔Redmi round-trip: ANR=0, краши мода=0. Код в `classes4.dex`.

---

## 🛎️ UX: плашка при неудачной расшифровке (B1)

**Проблема**: при потере E2E-сессии первое входящее сообщение падало с `decrypt fail` и терялось безвозвратно — без какого-либо уведомления в UI.

**Фикс**: перед молчаливым `return null` при decrypt-fail (`E2ECrypto.java:521`) вызывается `recordServiceEventForPeer` — в ленте чата появляется плашка «Сообщение не расшифровано, попросите собеседника переслать». Механизм тот же, что у reset-notify плашки (`2627-E2`), чей рендер device-verified. Код в `classes4.dex` (строки `e2e_enc_fail` / `recordServiceEventForPeer` подтверждены в dex).

---

## 📞 Гудок дозвона (C1)

**Проблема**: после перехода `s32.C` на Flow-паттерн наблюдателя вызов `stop()` обрывал тон через ≈0.3 с при исходящем звонке.

**Фикс**: гейт на `cc2.d != null` в `apply_v_ringback_v29.py` — `stop()` вызывается только при активном объекте тона. Device-verified.

---

## 🎛️ Функциональность: скрытие и фильтры (F1–F6, F8)

### Чаты-фильтры (F1)

R8-drift: тип `filterChatList` изменился с `Lc73` на `Lbe3`. Чаты «Коды подтверждения» и «MAX» снова корректно скрываются.

### Вкладка «Всё» и папка «Новое» (F2, F3)

`filterPageList` обновлён под новый тип `Lb67 → Lpe7`. ViewPager2-вызов `w1() → x1()` / `viewpager2->h(IZ)V` исправлен. Кнопка открытия папки «Новое» работает.

### Скрытие пунктов настроек (F4, F8)

**Корневая причина**: инверсия логики в self-gen методе `filterSettingsListCopy` — было `if-nez v4, :fslc_add`, что при v4=1 (флаг «скрыть») добавляло элемент, а не пропускало. Метод возвращал список скрываемых вместо видимых.

**Фикс**: убрана метка `:fslc_add`, логика `if-nez v4, :fslc_loop` + inline add. Один фикс закрыл «Войти в Сферум», «MAX для бизнеса», «Пригласить друзей». Device-verified S25 (тройной A/B: флаги ON → элемент скрыт; OFF → появился; ON → снова скрыт). Список настроек не пустой (~14 видимых пунктов).

### Кнопка «Прочитать всё» (F5)

Device-verified S25: logcat подтвердил 3 срабатывания `MRDL onClick → toast «Все сообщения отмечены прочитанными, MAX»`. Восстановлена chatId-цепочка `l2() → G1:Llye → a:Ljeh → getValue → Luz2;->a:J`.

### Невидимка не палит «в сети» (F6)

Device-verified Redmi↔S25 (невакуумный A/B): S25 с invisible ON — Redmi стабильно видит «1 ч назад», не «В сети», даже при активном S25. Baseline (invisible OFF) за 30 с показал «В сети» — тест невакуумный. Гейт `d19.a()` держит все 5 callsite'ов.

---

## 🖼️ UI: визуальные регрессии (U1–U7)

### Шрифт диалогов мода (U1)

`AlertDialog` под темой `OneMe.Theme.Transparent` не имел `textAppearance` → текст отображался микроскопическим. Фикс: `msgView()` / `getDialogTheme()` с явным атрибутом.

### Многострочность в диалогах и настройках (U2)

Тема `OneMe.Theme.Transparent` навязывает `singleLine=true` + `ellipsize`. Длинные тексты E2E-диалогов и описаний тумблеров обрезались в «…». Фикс: `setSingleLine(false) + setMaxLines(...) + setEllipsize(null)` на всех затронутых View. Device-verified S25.

### Имя приложения «icon» (U3)

Label id сдвинулся (`0x7f110802 → 0x7f1108a4`). Имя на иконке и в менеджере приложений отображалось как «icon». Исправлено.

### Цвет иконки (U4)

Двойной stable-ID drift в адаптивной иконке (binary AXML). Исправлены 18 файлов в `templates/patched`. Device-verified S25 (OneUI). Примечание: MIUI маскирует фон адаптивной иконки — эффект виден только на OneUI.

### About-карточки (U5)

R8-drift dev-роутера: `Ll→Lz` / `Lefb→Llrb` / `Li85→Llf5` / `Lxc9→Lmn9`. Все три About-карточки открываются корректно.

### Постскролл в чате (U6)

R8-drift якоря `G/z0` нарушил автоскролл к новым сообщениям. Исправлено. Device-verified Redmi: logcat «POST_FORCE stable, done».

### Многострочность E2E-плашки в ленте (U7)

ViewHolder `Lbz4` (itemView = `android.widget.TextView`) рендерил E2E-плашку в одну строку под той же темой `OneMe.Theme.Transparent`. Фикс: `apply_v_e2e_pill_multiline.py` — после `setMovementMethod(p2)` вставка `setSingleLine(false) + setMaxLines + setEllipsize(null)` (регистр v0 мёртв в точке вставки, `.registers` не трогается). Device-verified S25+Redmi.

---


## 🔍 Исправления V2 (26.31.0.2) и V3 (26.31.0.3)

### 💥 Краш «Устройства → Войти по QR-коду» (A8) — FIXED V2

`QrScannerWidget.v1` падал VerifyError: носитель `getView()` `Lus4` (26.29.1) отсутствует в 26.31.0. Фикс: `Lus4 → Loz4` в `apply_v_e2e_qr_scanner_hook.py` (как в стоковом теле v1). Device-verified S25: QR-скан работает.

### 📱 Пустой список «Устройства» (F11) — FIXED V2

`apply_v_hide_settings_oye` мис-таргетился: в 26.31.0 `c3/lkg.smali` — VM экрана «Устройства» (devices), НЕ настроек. Инжект вызывал `filterSettingsList` на СПИСКЕ УСТРОЙСТВ → вычищал элементы. Фикс: скрипт исключён из APPLY_STEPS (hide-настроек работает через `xhg.r()`/`filterSettingsListCopy`, F8). Device-verified S25: полный список (текущая + 3×WEB + 2×ANDROID).

### 🎬 Не отправляются видео (F12) — FIXED V3

**Симптом**: фото уходят, видео — нет (Redmi WGJBBEY9SSMN8TIB).

**Корень**: `apply_v_e2e_attach_upload_b.py` в `UploadFileAttachWorker.getForegroundInfo()` (метод `j()` = `getForegroundInfoAsync`) вставил блок с ПРОТУХШИМИ R8-именами 26.29.1:
- `Lcb9;->b:Landroidx/work/WorkerParameters;` — в 26.31.0 `cb9` теперь lambda (поля `b` нет; базовый класс воркера = `sl9`)
- `Landroidx/work/WorkerParameters;->b:Lw35;` — реально `b:Lta5;` (InputData)
- `Lw35;->c(Ljava/lang/String;J)J` — метода нет

WorkManager для expedited-задачи вызывает `getForegroundInfo` ДО `doWork` → `NoSuchFieldError` → задача тихо отменяется → видео не отправляется. Фото идут инлайн-аплоадом, минуя этот воркер.

**Дополнительно**: `apply_v_e2e_attach_upload_a.py` (E2E-шифрование вложений) в 26.31.0 — GUARD SKIP (`c2/iii.smali` отсутствует, `Laqi;->valueOf` исчез) → шифрование вложений не работает → патч B был бессмыслен и вреден.

**Фикс**: `apply_v_e2e_attach_upload_b.py` исключён из E2E_ONLY_STEPS; инжект откачен к стоковому `p()Lhza`-фрагменту. **Device-verified Redmi 16.09.2026: видео отправляется.**

### 🔄 Синхронизация «Автообновления» (новая фича)

Стоковый switch «Автообновление» (Настройки → О приложении) = зеркало мод-флага «Автоматически проверять при запуске» (`AppFlags.autoUpdateCheck`, UpdateChecker на GitHub). `apply_v_sync_autoupdate.py`: `n0.k(JZ)V` пишет только `autoUpdateCheck` (стоковый `app.selfservice.update` остаётся false — оригинал не перезатрёт мод), `h0.r()` рисует checked из мод-флага. Sentinel `:auto_update_sync_v1`.

### 🛠 R8-drift фиксы аудита (V3, 8 скриптов)

| Скрипт | Дрейф 26.29.1 → 26.31.0 |
|--------|--------------------------|
| `allow_any_filetype` | c3/dnd → **c4/g4e** (был NO-OP → активен: обход клиентского блэклиста расширений) |
| `dps_boot_gate` | DpsInitProvider c3 → **c2** |
| `noip_telemetry` | `:cond_1c5`→`:cond_1c1`, `Lko9`→`Liz9` |
| `p0_telem_dps` | фикс `path is None` (раньше крэш) |
| `pinned_drag` | `Luie`→`Le4f`, `Lc20`→`Lh40`, `Lm93`→`Lbe3` |
| `search_hide_channels_rows` | `Lsf3`→`Lkk3`, `Ldq7`→`Ljz7` (иначе VerifyError/мёртвый фильтр) |
| `settings_filter_fix` | перенесён после apply_v22_settings (snapshot стирал фикс F8/F4) |
| `voice_download` | реактивирован (якорь c3/s63.smali существует, sentinel приземлён) |

Также: `apply_v_manual_update_check` выключен (NO-OP на 26.31.0), `apply_v_filter_chatlist_k1a` выключен (мёртвая фича), `r8_drift_gate.py` научился динамическому резолву TARGET (rebalance_dex: c3/v4b → c1/v4b).

## ✅ Верификация

- **Сборка**: `make_v1.py --strict` — все 4 APK прошли gate (`exact_abi`, `no_tracer`, v2+v3 подпись `apksigner`, distinct arm64/armeabi-v7a SHA256 ассетов).
- **Статические gate**: `reg_collision_gate.py`, `type_ref_gate.py`, `undefined_method_gate.py` — PASS (0 CRITICAL).
- **Fan-out аудит**: все ~202 `apply_v_*.py` пройдены 13 параллельными агентами по тематическим группам (краши/E2E/call-e2e/antidel/scroll/ringback/watch-mode/hide-filter/antiread/telemetry/UI/security/misc) — багов в собранном APK не найдено; 3 rebuild-hazard в apply-скриптах обнаружены и исправлены до релиза. Процедура соответствует HARD-RULE «fan-out аудит перед публикацией».
- **Device-verify Samsung Galaxy S25 (RFCX5013V5F, Android 15)**:
  - Экран Оформление (A7): открывается без краша, `logcat -b crash` чист.
  - Скрытие пунктов настроек F4/F8: тройной A/B, список ~14 пунктов корректен.
  - E2E-блокировка редактирования (E5): device-verified.
  - Кнопка «Прочитать всё» F5: 3× `MRDL onClick` в logcat.
  - Невидимка F6: невакуумный A/B с Redmi — «В сети» не показывается при invisible ON.
  - Постскролл U6: device-verified Redmi, `POST_FORCE stable, done`.
  - Многострочность плашки U2, U7: device-verified S25+Redmi.
- **E2E round-trip S25↔Redmi (B2, E4)**: обмен в обе стороны, `decrypt OK`, `decrypt failed=0`, ANR=0, крашей мода нет.
- **Ringback C1**: device-verified.

---

*Аудит подготовлен по результатам fan-out аудита и трекера BUGS.md (секция 26.31.0 RS), сессии 2026-09-14/16. V3 = V1 + V2 + R8-drift аудит.*
