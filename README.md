# Лабораторная: оболочка приложения, экран и PWA

# Вы форкаете прошлую `mobdev-ts-entity-lab` работу и работаете в ней

[MiniPaint](https://viliusle.github.io/miniPaint/)

В первой лабораторной вы сделали сущность, коллекцию в Strapi и приложение, которое показывает, создаёт и удаляет записи. Приложение уже собирается в APK и запускается на телефоне. В рамках этой работы вы будете настраивать свое приложение.

- иконка приложения в лаунчере;
- splash-экран при запуске;
- название и описание;
- размер экрана, viewport, статус-бар и его цвет;
- PWA функционал

Всё делается для **вашей** сущности. Примеры ниже — на «играх», но иконки, цвета, тексты и скриншоты должны быть про вашу тему.

## 0. Что дано и что нужно сделать

Дано:

- React + Vite приложение с вашей сущностью;
- Capacitor и папка `android/` (из предыдущей лабы);

_Не забудьте, что если вы сделали коммит после того, как собрали проект для Pages - у вас указан baseURL в проекте, который будет ломать Vite сборщик в Codespace и приложение на телефоне_

## 1. Подготовка

### 1.1. Пакеты

В корне проекта:

```bash
npm i @capacitor/splash-screen
npm i -D @capacitor/assets
npx cap sync
```

- `@capacitor/splash-screen` — управляет splash-экраном.
- `@capacitor/assets` — генерирует иконки и splash из исходников. Работает без Android Studio.
- `SystemBars` для системных полос уже входит в `@capacitor/core` — отдельный пакет не нужен.

Если `android/` ещё нет — значит у вас не попала настройка проекта `android` с прошлой пары в коммит. Вернитесь и соберите `apk`, а потом продолжите с шага 1.1.

### 1.2. Исходники ассетов

Создайте в корне папку `assets/`. В ней — исходники **в большом размере**, отрисуйте исходники через MiniPaint (ссылка выше).

Рекомендуемый вариант (полный контроль):

```
assets/
├── icon-only.png          # 1024×1024, обычная иконка
├── icon-foreground.png    # 1024×1024, прозрачный фон, логотип с отступами
├── icon-background.png    # 1024×1024, фон adaptive-иконки
├── splash.png             # 2732×2732, светлая тема
└── splash-dark.png        # 2732×2732, тёмная тема
```

Простой вариант (одна картинка):

```
assets/
├── logo.png
└── logo-dark.png
```

_Будет получено меньше баллов, если реализовать не все ассеты_

Требования к исходникам:

- Только PNG. (По-хорошему должен быть SVG, но это сложнее реализовать с нашим софтом)
- `icon-only` и слои adaptive — не меньше 1024×1024.
- splash — не меньше 2732×2732, иначе на больших экранах будет мыло.
- В `icon-foreground` логотип должен занимать примерно центральные 60–70% холста: Android обрежет края.
- В splash не должно быть текста и мелких деталей: он показывается 0,5–2 секунды.

### 1.3. Генерация

Для полного варианта:

```bash
npx @capacitor/assets generate --android
```

Что появится в `android/app/src/main/res/`:

| Что                  | Куда                                                                              |
| -------------------- | --------------------------------------------------------------------------------- |
| Обычные иконки       | `mipmap-mdpi/…/ic_launcher.png` и остальные плотности                             |
| Круглые иконки       | `mipmap-*/ic_launcher_round.png`                                                  |
| Adaptive-иконка      | `mipmap-anydpi-v26/ic_launcher.xml`                                               |
| Слои adaptive-иконки | `mipmap-*/ic_launcher_foreground.png`, `ic_launcher_background.png`               |
| Splash               | `drawable/splash.png`, `drawable-land-*/splash.png`, `drawable-port-*/splash.png` |
| Тёмный splash        | `drawable-night*/splash.png`                                                      |

## 2. Иконка

### 2.1. Как Android рисует иконку

На Android одновременно есть несколько видов иконок:

| Вид                               | Когда используется                             |
| --------------------------------- | ---------------------------------------------- |
| Обычная PNG (`ic_launcher`)       | старые версии и некоторые лаунчеры             |
| Adaptive icon (`ic_launcher.xml`) | Android 8+                                     |
| Круглая (`ic_launcher_round`)     | лаунчеры, которые просят круг                  |
| Monochrome / themed icon          | Android 13+, если включены тематические иконки |

Adaptive-иконка — это не одна картинка, а два слоя:

- `background` — фон (цвет или картинка);
- `foreground` — логотип.

Система сама режет их под форму лаунчера: круг, квадрат со скруглением, «капля» и т.д. Поэтому важно оставлять отступы.

Размеры: холст **108×108 dp**, видимая область примерно **72×72 dp**, безопасный круг — около **66 dp** по диаметру. Если логотип залезает в края, на круглом лаунчере он обрежется.

### 2.2. Adaptive-иконка

После генерации откройте:

```
android/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml
```

Там должно быть примерно так:

```xml
<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@mipmap/ic_launcher_background" />
    <foreground android:drawable="@mipmap/ic_launcher_foreground" />
</adaptive-icon>
```

Если вы используете простой вариант с `logo.png`, инструмент создаст эти файлы сам. Руками их пишут только при тонкой настройке.

### 2.3. Монохромная иконка (Android 13+)

Тематические иконки — это когда система перекрашивает иконку под обои. Для них нужен третий слой — `monochrome`.

`@capacitor/assets` его **не генерирует**, его нужно добавить самому.

Слой `monochrome` — это обычный drawable, поэтому делаем его из PNG: без вектора, `pathData` и трассировки.

1. Подготовьте PNG-силуэт логотипа:
   - возьмите `icon-foreground.png` или `icon-only.png`;
   - уберите фон и оставьте одну форму: в miniPaint переведите изображение в чёрно-белое (`Desaturate` → `Levels`/`Threshold`), выделите фон волшебной палочкой и удалите его;
   - снаружи должна быть прозрачность, внутри — сплошная заливка. Цвет не важен: система сама перекрасит слой в цвет темы, важен только альфа-канал.

2. Сохраните файл размером 432×432 px (это 108 dp на xxxhdpi) в:

   ```
   android/app/src/main/res/drawable-xxxhdpi/ic_launcher_monochrome.png
   ```

   Логотип держите в центральном круге примерно 264 px (66 dp): Android обрежет края по маске иконки.

3. Добавьте слой в `ic_launcher.xml`:

   ```xml
   <adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
       <background android:drawable="@mipmap/ic_launcher_background" />
       <foreground android:drawable="@mipmap/ic_launcher_foreground" />
       <monochrome android:drawable="@drawable/ic_launcher_monochrome" />
   </adaptive-icon>
   ```

4. Повторите то же для `ic_launcher_round.xml`, если он есть.

5. Пересоберите APK и проверьте пока в Codespace

Важно: повторный запуск `@capacitor/assets` перезапишет `ic_launcher.xml` и удалит `<monochrome>`. PNG-файл останется, а строку в XML нужно добавить заново. Поэтому слой делают **после** генерации.

На Android 12 и ниже тег `<monochrome>` игнорируется, поэтому его обычно добавляют в тот же `mipmap-anydpi-v26/ic_launcher.xml`. Показывают themed icons не все лаунчеры — отсутствие эффекта на скриншоте не всегда ошибка.

### 2.4. Проверка иконки

Соберите APK:

```bash
(cd android && ./gradlew assembleDebug)
```

Посмотрите метаданные:

```bash
$ANDROID_HOME/build-tools/35.0.0/aapt2 dump badging \
  android/app/build/outputs/apk/debug/app-debug.apk | head -40
```

В выводе должны быть:

- `package: name='...'` — ваш `applicationId`;
- `application-label:'...'` — имя под иконкой;
- `application-icon-…` — пути к иконкам;
- `sdkVersion` и `targetSdkVersion`.

Проверьте, что в APK лежат нужные файлы:

```bash
unzip -l android/app/build/outputs/apk/debug/app-debug.apk | grep -E 'mipmap|drawable.*splash'
```

Если иконка не обновилась, это не баг сборки, а кэш лаунчера. Помогает: переустановить приложение, перезагрузить телефон, очистить кэш лаунчера.

## 3. Splash-экран

### 3.1. Как это работает на Android 12+

Начиная с Android 12 система показывает **свой** splash-экран при запуске приложения:

- фон берётся из темы;
- иконка — иконка приложения (или то, что указано в теме);
- отключить системный splash нельзя;
- после него показывается WebView с вашим приложением.

Capacitor использует библиотеку `androidx.core:core-splashscreen`, чтобы то же поведение было и на Android 11 и ниже.

Что это значит на практике:

- «сделать splash без статус-бара» на Android 12+ нельзя — системный splash рисует система;
- `backgroundColor`, `androidScaleType`, `showSpinner`, `splashFullScreen`, `splashImmersive` из конфига **не работают при старте** на Android 12+;
- эти параметры работают для программного `SplashScreen.show()` и на старых версиях.

Поэтому на Android 12+ внешний вид запуска задаётся **темой и ресурсами**, а не только конфигом плагина.

### 3.2. Настройка плагина

В `capacitor.config.ts`:

```ts
/// <reference types="@capacitor/splash-screen" />

import type { CapacitorConfig } from '@capacitor/cli'

const config: CapacitorConfig = {
    appId: 'ru.example.games',
    appName: 'Мои игры',
    webDir: 'dist',
    backgroundColor: '#f5efe3',
    plugins: {
        SplashScreen: {
            launchShowDuration: 1200,
            launchAutoHide: true,
            launchFadeOutDuration: 300,
            backgroundColor: '#f5efe3',
            androidScaleType: 'CENTER_CROP',
            showSpinner: false,
        },
    },
}

export default config
```

| Параметр                | Что делает                            |
| ----------------------- | ------------------------------------- |
| `launchShowDuration`    | сколько миллисекунд показывать splash |
| `launchAutoHide`        | скрывать автоматически                |
| `launchFadeOutDuration` | длительность исчезновения (Android)   |
| `backgroundColor`       | фон вокруг картинки                   |
| `androidScaleType`      | как вписать картинку                  |
| `showSpinner`           | показывать ли спиннер                 |

### 3.3. Тема, цвета и тёмный splash

Откройте:

```
android/app/src/main/res/values/styles.xml
```

Там есть тема запуска:

```xml
<style name="AppTheme.NoActionBarLaunch" parent="Theme.SplashScreen">
    <item name="android:background">@drawable/splash</item>
</style>
```

Добавьте цвет фона:

`values/colors.xml`:

```xml
<resources>
    <color name="splashBackground">#F5EFE3</color>
</resources>
```

`values/styles.xml`:

```xml
<style name="AppTheme.NoActionBarLaunch" parent="Theme.SplashScreen">
    <item name="android:background">@drawable/splash</item>
    <item name="windowSplashScreenBackground">@color/splashBackground</item>
</style>
```

`windowSplashScreenBackground` — это фон системного splash на Android 12+.

Для тёмной темы создайте `values-night/colors.xml`:

```xml
<resources>
    <color name="splashBackground">#211D18</color>
</resources>
```

Тёмная картинка splash генерируется из `splash-dark.png` и попадает в `drawable-night*/`. Если её нет, на тёмной теме останется светлый фон.

## 4. Имя и описание

### 4.1. Имя под иконкой

Имя приложения в ОС берётся не из `capacitor.config.ts`, а из ресурсов Android:

```
android/app/src/main/res/values/strings.xml
```

```xml
<resources>
    <string name="app_name">Мои игры</string>
    <string name="title_activity_main">Мои игры</string>
</resources>
```

- `app_name` — подпись под иконкой и в настройках.
- `title_activity_main` — заголовок Activity, может использоваться в «недавних».

`appName` в `capacitor.config.ts` используется при создании проекта (`cap add android`). Если поменять его позже, Android сам не переименуется — правьте `strings.xml`.

Правила:

- имя должно быть коротким: длинное обрезается в лаунчере;
- без эмодзи и спецсимволов;
- имя и иконка должны читаться вместе.

### 4.2. Локализация имени

Если хотите русское и английское имя, создайте:

```
android/app/src/main/res/values-ru/strings.xml
```

```xml
<resources>
    <string name="app_name">Мои игры</string>
    <string name="title_activity_main">Мои игры</string>
</resources>
```

а в `values/strings.xml` — английское. Язык системы выберет нужный файл.

### 4.3. Описание для PWA

В `public/manifest.webmanifest` есть поле `description`. Оно не показывается в лаунчере, но используется при установке, в некоторых браузерах и в поиске:

```json
{
    "name": "Мои игры",
    "short_name": "Игры",
    "description": "Каталог игр: жанр, платформа, год выпуска и рейтинг"
}
```

`short_name` — то, что видно под иконкой на домашнем экране. Держите его короче 12 символов.

Дублируйте описание в `index.html`:

```html
<meta
    name="description"
    content="Каталог игр: жанр, платформа, год выпуска и рейтинг" />
```

### 4.4. Описание для магазина

Даже если вы не публикуете приложение, описание — часть «релиза». В Google Play есть два поля:

- **короткое описание** — до 80 символов, одна строка;

## 5. Экран: базовые настройки

Разные версии Android ведут себя по-разному:

| Android | Системные полосы |
| --- | --- |
| 14 и ниже | статус-бар можно красить и скрывать, edge-to-edge не обязателен |
| 15 | edge-to-edge включается принудительно, но остаётся флаг `windowOptOutEdgeToEdgeEnforcement` |
| 16 | этот флаг убран: приложение всегда рисует под системными полосами |

Подгонять каждую версию отдельно не нужно. В Capacitor 8 есть встроенный `SystemBars`: он сам добавляет отступы под системные полосы, поэтому не нужны ни safe-area в CSS, ни JS-код. Достаточно конфигов.

### 5.1. Viewport

В `index.html`:

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

- `width=device-width` — ширина страницы равна ширине экрана, а не 980 px;
- `initial-scale=1` — без автоматического зума.

`viewport-fit=cover` для базовой настройки не нужен: `SystemBars` сам оставит контент между полосами.

### 5.2. Системные полосы

В `capacitor.config.ts`:

```ts
const config: CapacitorConfig = {
    // ...
    plugins: {
        SystemBars: {
            insetsHandling: 'native',
            style: 'LIGHT',
        },
    },
}
```

- `insetsHandling: 'native'` — Capacitor сам добавляет отступы под статус-бар и навигацию. Работает и на Android 15/16 с принудительным edge-to-edge.
- `style: 'LIGHT'` — тёмные иконки для светлого фона; `'DARK'` — светлые иконки для тёмного фона.

Названия стилей относятся к фону, а не к цвету иконок: `'LIGHT'` — светлый фон, `'DARK'` — тёмный. Для светлой темы приложения берите `'LIGHT'`.

`SystemBars` уже входит в `@capacitor/core`, отдельный пакет не нужен. Цвет полосы не задаётся: на Android 15+ она прозрачная, а цвет под ней берётся из фона приложения (`backgroundColor` и фон темы).

### 5.3. Ориентация и клавиатура

В `AndroidManifest.xml` у Activity:

```xml
<activity
    android:name=".MainActivity"
    android:screenOrientation="portrait"
    android:windowSoftInputMode="adjustResize"
    ... >
```

- `screenOrientation` — `portrait`, `landscape`, `sensorPortrait`, `sensorLandscape` или `unspecified`;
- `windowSoftInputMode="adjustResize"` — клавиатура не закрывает поля формы.

Дополнительный плагин и код не нужны.

## 6. PWA: что это и чем отличается

### 6.1. Определение

**PWA (Progressive Web App)** — это обычный сайт, который:

- имеет `manifest.webmanifest` с именем, иконками и цветами;
- обслуживается по HTTPS;
- обычно имеет service worker (для офлайна и кэша);
- может быть установлен на домашний экран и открываться в отдельном окне без адресной строки.

PWA — это не новый язык и не фреймворк. Это набор соглашений поверх веб-технологий.

### 6.2. Сравнение

|                 | Обычная веб-страница    | PWA                               | Capacitor-приложение     |
| --------------- | ----------------------- | --------------------------------- | ------------------------ |
| Где живёт       | вкладка браузера        | вкладка или отдельное окно        | установленное приложение |
| Установка       | нет                     | «Установить» / «На главный экран» | APK или AAB              |
| Иконка          | favicon во вкладке      | из `manifest.webmanifest`         | из `res/mipmap-*`        |
| Splash          | нет                     | частично, через манифест и тему   | `drawable/splash`        |
| Офлайн          | только с service worker | service worker                    | JS-бандл уже внутри APK  |
| Доступ к железу | браузерные API          | браузерные API                    | плагины Capacitor        |
| Обновление      | F5                      | при открытии                      | новый APK или магазин    |
| Магазин         | нет                     | обычно нет                        | Google Play              |
| Манифест        | нет                     | `manifest.webmanifest`            | `AndroidManifest.xml`    |
| Размер          | килобайты               | килобайты                         | десятки мегабайт         |

### 6.3. Два разных манифеста

Это главное, что нужно понять:

- `AndroidManifest.xml` — описывает **нативное** приложение: Activity, permissions, иконку, тему. Его читает Android.
- `manifest.webmanifest` — описывает **сайт**: имя, иконки, `start_url`, `display`. Его читает браузер.

Capacitor **не читает** `manifest.webmanifest`. Внутри APK работает WebView, а иконку и splash берут из Android-ресурсов. Поэтому PWA-манифест нужен для браузерной версии, а не для APK.

При этом брендинг общий: одни и те же исходники иконок можно использовать и для Android, и для PWA.

### 6.4. Зачем делать PWA, если есть APK

- Пользователь может открыть приложение по ссылке без установки из магазина.
- На десктопе оно ставится как отдельное окно.
- Это быстрее, чем объяснять, где скачать APK.
- Для лабораторной это способ увидеть разницу между «сайт», «PWA» и «нативное приложение».

### 6.5. Условия установки

Chrome устанавливает PWA, если:

- сайт открыт по HTTPS (или на `localhost`);
- есть манифест с `name`/`short_name`, `start_url`, `display` и иконками;
- есть иконки 192×192 и 512×512;
- пользователь провёл на сайте немного времени (Chrome может не показывать кнопку сразу).

## 7. Настройка PWA

### 7.1. Манифест

Создайте `public/manifest.webmanifest`:

```json
{
    "name": "Мои игры",
    "short_name": "Игры",
    "description": "Каталог игр: жанр, платформа, год выпуска и рейтинг",
    "lang": "ru",
    "dir": "ltr",
    "start_url": "./",
    "scope": "./",
    "display": "standalone",
    "orientation": "portrait",
    "background_color": "#f5efe3",
    "theme_color": "#2f6b4f",
    "icons": [
        {
            "src": "pwa-192x192.png",
            "sizes": "192x192",
            "type": "image/png",
            "purpose": "any"
        },
        {
            "src": "pwa-512x512.png",
            "sizes": "512x512",
            "type": "image/png",
            "purpose": "any"
        },
        {
            "src": "maskable-icon-512x512.png",
            "sizes": "512x512",
            "type": "image/png",
            "purpose": "maskable"
        }
    ]
}
```

Поля:

| Поле               | Зачем                                              |
| ------------------ | -------------------------------------------------- |
| `name`             | полное имя при установке                           |
| `short_name`       | подпись под иконкой                                |
| `description`      | описание для установки и поиска                    |
| `start_url`        | с какой страницы открывать                         |
| `scope`            | какие страницы считаются приложением               |
| `display`          | `standalone` — со статус-баром, `fullscreen` — без |
| `orientation`      | предпочтительная ориентация                        |
| `theme_color`      | цвет панели браузера и системной полосы            |
| `background_color` | фон при запуске                                    |
| `icons`            | иконки, включая `maskable`                         |

Обратите внимание: `display: "standalone"` оставляет статус-бар, `display: "fullscreen"` убирает и его. Это PWA-аналог «со статусом / без статуса».

### 7.2. Иконки для PWA

Иконки для PWA лежат в `public/`. Всё, что там находится, Vite копирует в `dist/` как есть. Не путайте эту папку с `dist/assets/`, куда Vite складывает собранный JS и CSS.

Самый предсказуемый способ — `@vite-pwa/assets-generator`. Положите исходник в `public/logo.png` (можно взять `assets/icon-only.png`):

```bash
npm i -D @vite-pwa/assets-generator
npx pwa-assets-generator --preset minimal-2023 public/logo.png
```

Он сделает из одного логотипа:

```
public/
├── favicon.ico
├── pwa-64x64.png
├── pwa-192x192.png
├── pwa-512x512.png
├── maskable-icon-512x512.png
└── apple-touch-icon-180x180.png
```

`logo.png` — исходник не меньше 512×512, лучше 1024×1024.

### 7.3. Подключение манифеста

В `index.html`:

```html
<link rel="manifest" href="./manifest.webmanifest" />
<meta name="theme-color" content="#2f6b4f" />
<link rel="apple-touch-icon" href="./apple-touch-icon-180x180.png" />
<meta name="apple-mobile-web-app-capable" content="yes" />
<meta name="apple-mobile-web-app-status-bar-style" content="default" />
```

`apple-*` теги нужны для iOS: Safari не всё берёт из манифеста.

## 8. Соберите приложение

Соберите APK по инструкции из прошлых работ (_Не забудьте, что у вас уже инициализирован `android` в `capacitor`, вам надо просто напросто собрать приложение через `vite` и синхронизировать его с Android пакетом_)

# Как сдавать

- Создайте форк репозитория в вашей организации с названием-этого-репозитория-вашафамилия
- Используя ветку wip сделайте задание
- Соберите APK файл, поместите его в корень
- Зафиксируйте изменения в вашем репозитории
- Когда документ будет готов - создайте пул реквест из ветки wip (вашей) на ветку main (тоже вашу) и укажите меня (ktkv419) как reviewer

Не мержите сами коммит, это сделаю я после проверки задания
