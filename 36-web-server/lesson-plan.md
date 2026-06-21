# Конспект уроков — BAS WebServer

> Подробный пошаговый план создания скрипта: что делать в каждом из 12 уроков, с кодом и командами. Дополняет видеоуроки.

[← К модулю](README.md) · [Техническое задание](technical-specification.md)

---

## Урок №1: Подготовка инструментов

1. Установить BAS свежей или предыдущей версии **28.3.0** — https://bablosoft.com/shop/BrowserAutomationStudio#download
2. Установить **Postman** для тестирования запросов — https://www.postman.com/downloads/
3. Установить **Node.js** версии, равной версии BAS — **22.11.0** — https://nodejs.org/en/blog/release/v22.11.0
4. Создать новый XML-проект в BAS.
5. Активировать Node.js 22.11.0 в настройках BAS.
6. Добавить NPM-модуль **Express** для создания веб-сервера.
7. Перезагрузить проект.

## Урок №2: Подготовка ресурсов

Добавляем ресурсы:

1. «Потоки»
2. «Ссылка на Форму» (сразу заполнить)
3. «Пост Запрос на Код» (сразу заполнить)
4. «Пост Запрос на Изображение» (сразу заполнить)
5. «Пост Запрос Отправки» (сразу заполнить)
6. «Google Client ID»
7. «Google Client Secret»
8. «Google Auth Code» (на этом этапе не заполняем)
9. «Отпечатки ApiKey»
10. «Тип Прокси»
11. «Список Прокси»
12. «Список Сайтов для Куков»
13. «Веб-Сервер Порт»
14. «Веб-Сервер ApiKey»
15. «Лимит Сайтов для Нагула»

Дополнительно разбираем: что такое потоки; ссылки на форму и запросы; как получить Google Client ID и Secret.

**Инструкция по получению Google-токенов:**

1. Заходим на https://console.cloud.google.com/
2. Создаём новый проект или выбираем старый.
3. API & Services → Library.
4. Найти **Google Drive API** → Enable.
5. API & Services → Credentials.
6. Create Credentials → OAuth Client ID.
7. Application Type: **Desktop App**.
8. Create.
9. Сохраняем Client ID и Client Secret.
10. В разделе Test Users (Audience → Test Users) добавляем свою почту.

**Продолжение:**

- Для чего и где берётся ключ отпечатков — https://fp.bablosoft.com/
- Список прокси: формат для обычных и мобильных прокси.
- Список сайтов для куков — https://publicwww.com/websites/fbevents.js/ — где собрать и как сформировать.

## Урок №3: Развертывание Express веб-сервера в BAS

1. Создаём функцию `LaunchServer`.
2. Добавляем переменные для порта и API-ключа.
3. Вызываем кубик Node.js и пишем код.

**Общий код для асинхронной работы:**

```js
await (new Promise((resolve, reject) => {
    /* Код веб-сервера */
}));
```

**Инициализация Express:**

```js
var express = require('express');
var app = express();
```

**Функция проверки Bearer-токена:**

```js
function checkAuth(req, res, next) {
    const authHeader = req.headers['authorization'];

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'Нет токена или неверный формат' });
    }

    const token = authHeader.split(' ')[1]; // Извлекаем токен

    if (token !== [[APIKEY]]) { // Здесь можно сделать проверку через JWT
        return res.status(403).json({ error: 'Недействительный токен' });
    }

    next(); // Передаём управление дальше
}
```

**Поддержка JSON и form-data:**

```js
app.use(express.json());                          // принимать JSON
app.use(express.urlencoded({ extended: true }));  // form-data
```

**Обработка POST-запросов:**

```js
app.post('/task', checkAuth, function (req, res) {
    [[REQ_BODY]] = JSON.stringify(req.body);
    res.send('<h1>Запрос принят от сервера!</h1>');
});
```

**Запуск сервера на порту:**

```js
var server = app.listen([[PORT]], function () {
    console.log('Example app listening http://127.0.0.1:' + [[PORT]]);
});
```

## Урок №4: Настройка архитектуры работы

1. Пояснение архитектуры: асинхронная функция на приём запросов + ресурс с задачами для обработки в многопотоке.
2. Создаём функцию `OnApplicationStart`.
3. Создаём ресурс «Tasks» с типом **Ожидать Появления**.
4. Асинхронно вызываем функцию веб-сервера внутри `OnApplicationStart`.
5. После запроса от другого сервера добавляем элемент в ресурс: `BAS_API('RInsert("Tasks",VAR_REQ_BODY,false)');`
6. Выставляем интервал выполнения Node.js-кубика 999999999 сек.
7. Включаем игнорирование ошибок на Node.js-сервере.
8. Создаём бесконечный цикл `true` и переносим кубики сервера внутрь — для перезапуска при непредвиденном закрытии по таймауту.
9. Вызываем лог `{{Tasks}}`.
10. Запускаем скрипт в реальном времени, шлём запросы через Postman для проверки.

## Урок №5: Создание функции эмуляции активности

1. Создаём функцию `EmulateActivity` с параметром `wait_time` (в секундах).
2. Создаём список «Точность действий»: low, medium, high.
3. Создаём список «Интенсивность действий»: low, medium, high.
4. Случайный элемент для «Точность действий».
5. Случайный элемент для «Интенсивность действий».
6. Кубик «Начать эмуляцию бездействия» с полученными параметрами.

## Урок №6: Создание функции прогрева профиля куками

1. Создаём функцию `CookiesWarming` с параметром `sites_limit`.
2. Функция «Ресурс В Список» для подготовки списка сайтов.
3. Цикл `While true`.
4. Проверка наличия элементов: `[[COOKIES_SITES]].length > 0`.
5. Случайный элемент из сайтов с удалением.
6. Загружаем сайт кубиком «Загрузить».
7. Действие «случайное число».
8. Вызываем `EmulateActivity` со случайным числом.
9. Если условие не выполняется — `break`.

## Урок №7: Работа с профилями (профили, отпечатки, прокси)

1. Создаём функцию `LoadProfile` с параметрами `proxy`, `proxy_type`.
2. Кубик «Настройки браузера» с параметрами и ключом фингерпринта:

```text
--disk-cache-size=104857600
--disable-gpu-program-cache
--disable-gpu-shader-disk-cache
--ignore-certificate-errors
--enable-compute-pressure-rate-obfuscation-mitigation
--bounce-tracking-mitigations
--heavy-ad-privacy-mitigations
--disable-accelerated-2d-canvas
--enable-gpu-rasterization
```

3. Кубик «Получить отпечаток браузера» с настройками и ключом фингерпринта.
4. Кубик «Применить отпечаток» с ключом фингерпринта.
5. Логика: обычные или мобильные прокси.
6. Обычные → кубик «Прокси».
7. Мобильные → парсинг формата, смена по ссылке, вызов кубика «Прокси».
8. Вызываем `CookiesWarming` со случайным параметром из ресурса «Лимит Сайтов для Нагула».
9. В `Main` вызываем `LoadProfile` со случайными значениями из «Список Прокси» и «Тип Прокси».

## Урок №8: Запрос кода и изображения с другого сервера

**Функция `GetCode`** (параметр `url`, результат `CODE`):
1. Post-запрос с url `{{Пост Запрос на Код}}`.
2. Добавляем содержание ответа.
3. Извлекаем код из ключа `code` («Получить значение JSON»).
4. `return` полученного значения.

**Функция `GetImageLink`** (параметр `url`, результат `IMAGE_LINK`):
1. Post-запрос с url `{{Пост Запрос на Изображение}}`.
2. Добавляем содержание ответа.
3. Извлекаем ссылку из ключа `img`.
4. `return` полученного значения.

**Функция `DownloadImage`** (параметр `url`, результат `IMAGE_PATH`):
1. Извлекаем `FILE_ID` из ссылки («Получить подстроку между» `d/` и `/view`).
2. Получаем `INSTALLATION_PATH`.
3. Кубик «Случайная строка» для имени файла.
4. Кубик «Скачать» (HTTP-запросы): `https://drive.google.com/uc?export=download&id=[[FILE_ID]]`, путь сохранения `[[INSTALLATION_PATH]]/images/[[RANDOM_STRING]].png`.
5. `return` → `[[INSTALLATION_PATH]]/images/[[RANDOM_STRING]].png`.

## Урок №9: Открытие сайта с формой, эмуляция и заполнение

Функция `LoadFormAndFill` (параметры `url`, `data`, результат `SCREENSHOT_BASE64`):

1. Извлекаем `id`, `name`, `phone` из строки («Получить значение JSON»).
2. Загрузить `{{Ссылка на Форму}}`.
3. Ввод текста в поле **Имя** → `[[NAME]]`.
4. Ввод текста в поле **Телефон** → `[[PHONE]]`.
5. Вызываем `GetCode` → `CODE`.
6. Ввод текста в поле **Код верификации** → `[[CODE]]`.
7. Вызываем `GetImageLink` → `IMAGE_LINK`.
8. Вызываем `DownloadImage(IMAGE_LINK)` → `IMAGE_PATH`.
9. Кубик «Диалог Открыть Файл» с `IMAGE_PATH` для загрузки изображения в форму.
10. «Двигать Мышь И Кликнуть На Элемент» — выбор файла.
11. Ждать появления элемента после загрузки (крестик `CSS > .clear-button`).
12. «Двигать Мышь И Кликнуть На Элемент» — кнопка **Submit**.
13. Ждать появления `CSS > #submitted-form` (подтверждение отправки).
14. «Получить разрешение и положение курсора» → ширина/высота браузера.
15. «Браузер — Скриншот» с `[[BROWSER_WIDTH]]`, `[[BROWSER_HEIGHT]]`.
16. `return RENDER_RESULT`.
17. В `Main` вызываем `LoadFormAndFill(data = {{Tasks}}, url = {{Ссылка на Форму}})`.

## Урок №10: Загрузка скриншота на Google Drive

Сначала получаем авторизационный код Google в браузере (действителен 10 минут) и вставляем в ресурс «Google Auth Code»:

```text
https://accounts.google.com/o/oauth2/auth?client_id=YOUR_CLIENT_ID&redirect_uri=urn:ietf:wg:oauth:2.0:oob&response_type=code&scope=https://www.googleapis.com/auth/drive.file&access_type=offline
```

**Функция `GetAccessToken`** (без параметров, результат `ACCESS_TOKEN`):

1. Получаем `INSTALLATION_PATH`.
2. «Информация О Файле/Папке» `[[INSTALLATION_PATH]]/google_tokens.txt`.
3. Если `[[FILEINFO_EXISTS]]` → «Читать Файл» в `SAVED_CONTENT`.
4. Иначе POST-запрос `https://oauth2.googleapis.com/token` с параметрами:
   `client_id={{Google Client ID}}, client_secret={{Google Client Secret}}, code={{Google Auth Code}}, grant_type=authorization_code, redirect_uri=urn:ietf:wg:oauth:2.0:oob`
5. Проверяем `SAVED_STATUS`: если 200 → содержание ответа и «Запись В Файл» `google_tokens.txt`; иначе «Прервать скрипт: неверный код или токены».
6. «Получить значение JSON» `refresh_token`.
7. POST `https://oauth2.googleapis.com/token`:
   `client_id={{Google Client ID}}, client_secret={{Google Client Secret}}, refresh_token=[[REFRESH_TOKEN]], grant_type=refresh_token`
8. Если `SAVED_STATUS == 200` → `access_token` из JSON → `return [[ACCESS_TOKEN]]`; иначе «Прервать скрипт: refresh-токен истёк».
9. Вызываем `GetAccessToken` в `OnApplicationStart`.

**Функция `UploadScreenToDisk`** (параметр `screen_base64`):

1. Вызываем `GetAccessToken`.
2. POST `https://www.googleapis.com/upload/drive/v3/files?uploadType=media`, заголовок `Authorization: Bearer [[ACCESS_TOKEN]]`, тело `base64:[[SCREEN_BASE64]]`.
3. Если `SAVED_STATUS == 200`:
   - `id` из JSON → `FILE_ID`.
   - POST `https://www.googleapis.com/drive/v3/files/[[FILE_ID]]/permissions`, заголовок `Authorization: Bearer [[ACCESS_TOKEN]]`, тело `{"role": "reader", "type": "anyone"}`, тип `application/json`.
   - GET `https://www.googleapis.com/drive/v3/files/[[FILE_ID]]?fields=webViewLink`, заголовок `Authorization: Bearer [[ACCESS_TOKEN]]`.
   - `webViewLink` из JSON → `SCREENSHOT_DRIVE_LINK` → `return`.
   - Иначе «Прервать Скрипт: проблема с Google».
4. Вызываем `UploadScreenToDisk([[SCREENSHOT_BASE64]])` в `Main`.

## Урок №11: Отправка ссылки с логом + финальная проверка + логирование

1. Функция `SendScreenAndLogSuccess` (параметры `log_text`, `screen_link`).
2. Создать объект JSON с ключами `screen_link`, `log_text`.
3. POST-запрос отправки с телом `NEW_OBJECT` (JSON), тип `application/json`.
4. Содержание ответа → `SAVED_CONTENT`, вывести в лог.
5. В `Main` вызываем `SendScreenAndLogSuccess(log_text = "Задача №[[ID]] выполнена", screen_link = [[SCREENSHOT_DRIVE_LINK]])`.
6. Запускаем в режим запуска и отлаживаем.

## Урок №12: Компиляция и перенос на виртуальный сервер

1. Компиляция скрипта BAS.
2. Установка на виртуальный сервер.
3. Запуск на виртуальном сервере.
4. Настройки брандмауэра:
   - Панель управления → Система и безопасность → Брандмауэр Защитника Windows.
   - Дополнительные параметры → Правила для входящих подключений → Создать правило.
   - «Для порта» → Далее.
   - TCP или UDP (по протоколу).
   - Нужный порт (например, 3000) или диапазон.
   - «Разрешить подключение».
   - Указать профили сетей (Домены / Частные / Общедоступные).
   - Имя правила → сохранить.
5. Запуск скрипта на приём запросов и тест.
