---
title: Руководство по Создание приложения Flask для перевода, синтеза и анализа текста — Переводчик
titleSuffix: Azure Cognitive Services
description: В этом руководстве описано, как создать веб-приложение на основе Flask для перевода текста, анализа тональности и синтеза речи на основе переведенного текста.
services: cognitive-services
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: tutorial
ms.date: 03/04/2021
ms.author: lajanuar
ms.custom: devx-track-python, devx-track-js
ms.openlocfilehash: 6ec951e57b40ae1440f541c02b26e7788b3cf151
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105043739"
---
# <a name="tutorial-build-a-flask-app-with-azure-cognitive-services"></a>Руководство по Создание приложения Flask в Azure Cognitive Services

В этом руководстве описано, как создать веб-приложение Flask, которое использует Azure Cognitive Services для перевода текста, анализа тональности и синтеза речи на основе переведенного текста. Здесь описаны код Python и маршруты Flask, которые потребуются нашему приложению, а также приведены сведения о создании кода HTML и JavaScript, которые собирают приложение воедино. Если вы столкнетесь с проблемами, сообщите нам об этом с помощью расположенной ниже кнопки обратной связи.

Темы, рассматриваемые в этом руководстве:

> [!div class="checklist"]
> * получение ключей подписки Azure;
> * настройка среды разработки и установка зависимостей;
> * создание приложения Flask;
> * Перевод текста с помощью Переводчика
> * применение Анализа текста для определения положительной или отрицательной тональности введенного текста и перевода;
> * применение служб речи для преобразования переведенного текст в синтезированную речь;
> * локальное выполнение приложения Flask.

> [!TIP]
> Если вы хотите пропустить этот этап и сразу получить готовый код, весь пример с инструкциями по сборке доступен в репозитории [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Flask-App-Tutorial).

## <a name="what-is-flask"></a>Что такое Flask?

Flask — это микроплатформа для создания веб-приложений. Это означает, что Flask предоставляет все средства, библиотеки и технологии, требуемые для создания веб-приложения. Это веб-приложение может содержать несколько веб-страниц, блог, вики-сайт или что-то более существенное, например приложение календаря или коммерческий веб-сайт.

Для тех, кто хочет узнать больше после работы с этим руководством, мы предлагаем несколько полезных ссылок:

* [Документация по Flask](http://flask.pocoo.org/)
* [Flask for Dummies - A Beginner's Guide to Flask](https://codeburst.io/flask-for-dummies-a-beginners-guide-to-flask-part-uno-53aec6afc5b1) (Flask для чайников — руководство по Flask для начинающих)

## <a name="prerequisites"></a>Предварительные требования

Давайте перечислим программное обеспечение и ключи подписки, которые вам потребуются в этом руководстве.

* [Python 3.6 или более поздней версии](https://www.python.org/downloads/).
* [Средства Git](https://git-scm.com/downloads).
* Среда разработки или редактор кода, например [Visual Studio Code](https://code.visualstudio.com/) или [Atom](https://atom.io/).  
* Браузер [Chrome](https://www.google.com/chrome/browser/) или [Firefox](https://www.mozilla.org/firefox).
* Ключ подписки **Переводчик** (скорее всего, вы можете использовать **глобальное** расположение).
* Ключ подписки для службы **Анализ текста** в регионе **Западная часть США**.
* Ключ подписки для службы **Речь** в регионе **Западная часть США**.

## <a name="create-an-account-and-subscribe-to-resources"></a>Создание учетной записи и подписки на ресурсы

Как упоминалось ранее, в этом руководстве вам потребуются три ключа подписки. Это означает, что в учетной записи Azure следует создать ресурсы для следующих служб:
* API перевода
* Анализ текста
* Службы "Речь"

В руководстве по [созданию учетной записи Cognitive Services на портале Azure](../cognitive-services-apis-create-account.md) приведены пошаговые инструкции по созданию ресурсов.

> [!IMPORTANT]
> Для этого руководства следует создавать ресурсы в регионе "Западная часть США". Если вы выберете другой регион, вам нужно будет вручную изменить базовый URL-адрес в каждом из файлов Python.

## <a name="set-up-your-dev-environment"></a>Настройка среды разработки

Перед созданием веб-приложения Flask необходимо создать рабочий каталог для проекта и установить несколько пакетов Python.

### <a name="create-a-working-directory"></a>Создание рабочего каталога

1. Откройте командную строку (в Windows) или терминал (в Mac OS и Linux). Затем создайте рабочий каталог и все нужные вложенные папки для проекта:  

   ```
   mkdir -p flask-cog-services/static/scripts && mkdir flask-cog-services/templates
   ```
2. Перейдите в рабочий каталог проекта:  

   ```
   cd flask-cog-services
   ```

### <a name="create-and-activate-your-virtual-environment-with-virtualenv"></a>Создание и активация виртуального окружения с помощью `virtualenv`

Давайте создадим виртуальное окружение для приложения Flask с помощью `virtualenv`. Виртуальное окружение предоставляет вам чистую среду для работы.

1. Выполните в рабочем каталоге следующую команду, чтобы создать виртуальное окружение **для macOS или Linux**:
   ```
   virtualenv venv --python=python3
   ```
   Мы явным образом объявили, что виртуальное окружение будет использовать Python 3. Это гарантирует, что пользователи с несколькими установками Python будут использовать правильную версию.

   **Командная строка или Bash в Windows:**
   ```
   virtualenv venv
   ```
   Для простоты мы присвоении виртуальному окружению имя venv.

2. Команды для активации виртуального окружения будут разными в зависимости от платформы или оболочки:   

   | Платформа | Оболочка | Get-Help |
   |----------|-------|---------|
   | Mac OS и Linux | bash/zsh | `source venv/bin/activate` |
   | Windows | bash | `source venv/Scripts/activate` |
   | | Командная строка | `venv\Scripts\activate.bat` |
   | | PowerShell | `venv\Scripts\Activate.ps1` |

   После выполнения этой команды в командной строке или сеансе терминала должен появиться префикс `venv`.

3. Вы можете в любой момент отключить этот режим, введя в командной строке или терминале команду `deactivate`.

> [!NOTE]
> Python содержит подробную документацию по созданию виртуальных окружений и управления ими — [virtualenv](https://virtualenv.pypa.io/en/latest/).

### <a name="install-requests"></a>Установка Requests

Модуль Requests широко применяется для отправки запросов HTTP 1.1. Вам не придется вручную добавлять строки запроса в URL-адреса или кодировать данные POST для использования в форме.

1. Чтобы установить модуль Requests, выполните следующую команду:

   ```
   pip install requests
   ```

> [!NOTE]
> Если вам нужны дополнительные сведения, воспользуйтесь статьей [Requests: HTTP for Humans](https://2.python-requests.org/en/master/) (Модуль Requests: HTTP с человеческим лицом).

### <a name="install-and-configure-flask"></a>Установка и настройка Flask

Теперь нам нужно установить Flask. Flask обрабатывает маршрутизацию для веб-приложения и позволяет выполнять вызовы между серверами, скрывая ключи подписки от пользователя.

1. Чтобы установить Flask, выполните следующую команду:
   ```
   pip install Flask
   ```
   Давайте убедимся, что платформа Flask успешно установлена. Выполните команду:
   ```
   flask --version
   ```
   В терминал будет показан номер версии. Любой другой ответ означает, что что-то пошло не так.

2. Чтобы запустить приложение Flask, можно применить команду flask или параметр -m в Python с указанием Flask. Но до этого вам нужно сообщить терминалу, с каким приложением нужно работать, экспортировав переменную среды `FLASK_APP`:

   **Mac OS и Linux**:
   ```
   export FLASK_APP=app.py
   ```

   **Windows**:
   ```
   set FLASK_APP=app.py
   ```

## <a name="create-your-flask-app"></a>Создание приложения Flask

В этом разделе описано, как создать приложение Flask с базовыми функциями, которое возвращает HTML-файл при обращении пользователя к корневому каталогу приложения. Не тратьте пока время на подробный разбор кода — мы позднее вернемся к этому файлу и изменим его.

### <a name="what-is-a-flask-route"></a>Что такое маршрут Flask?

Давайте немного поговорим о концепции [маршрутов](http://flask.pocoo.org/docs/1.0/api/#flask.Flask.route). Маршрутизация используется для привязки URL-адресов к определенным функциям. Flask использует декораторы маршрутов для регистрации функций по конкретным URL-адресам. Например, при входе пользователя в корневой каталог веб-приложения (`/`) отображается `index.html`.  

```python
@app.route('/')
def index():
    return render_template('index.html')
```

Давайте рассмотрим еще один пример, чтобы закрепить понимание.

```python
@app.route('/about')
def about():
    return render_template('about.html')
```

Этот код нужен для того, чтобы при переходе пользователя на страницу `http://your-web-app.com/about` отображался файл `about.html`.

В этих примерах показано, как отобразить нужные HTML-страницы для пользователя. Но с помощью маршрутов можно также вызывать API по нажатию кнопки или выполнять любое количество действий без необходимости покидать главную страницу. Вы сможете увидеть это в действии, создав маршруты для перевода, анализа тональности и синтеза речи.

### <a name="get-started"></a>Начало работы

1. Откройте проект в интегрированной среде разработки и создайте файл с именем `app.py` в корневом каталоге рабочей папки. Скопируйте следующий код в файл `app.py` и сохраните его:

   ```python
   from flask import Flask, render_template, url_for, jsonify, request

   app = Flask(__name__)
   app.config['JSON_AS_ASCII'] = False

   @app.route('/')
   def index():
       return render_template('index.html')
   ```

   Этот блок кода сообщает приложению, что нужно отображать `index.html` каждый раз, когда пользователь переходит в корневой каталог веб-приложения (`/`).

2. Теперь давайте создадим интерфейсную часть для веб-приложения. Создайте файл с именем `index.html` в каталоге `templates`. Затем скопируйте следующий код в `templates/index.html`:

   ```html
   <!doctype html>
   <html lang="en">
     <head>
       <!-- Required metadata tags -->
       <meta charset="utf-8">
       <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
       <meta name="description" content="Translate and analyze text with Azure Cognitive Services.">
       <!-- Bootstrap CSS -->
       <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">
       <title>Translate and analyze text with Azure Cognitive Services</title>
     </head>
     <body>
       <div class="container">
         <h1>Translate, synthesize, and analyze text with Azure</h1>
         <p>This simple web app uses Azure for text translation, text-to-speech conversion, and sentiment analysis of input text and translations. Learn more about <a href="https://docs.microsoft.com/azure/cognitive-services/">Azure Cognitive Services</a>.
        </p>
        <!-- HTML provided in the following sections goes here. -->

        <!-- End -->
       </div>

       <!-- Required Javascript for this tutorial -->
       <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN" crossorigin="anonymous"></script>
       <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q" crossorigin="anonymous"></script>
       <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl" crossorigin="anonymous"></script>
       <script type = "text/javascript" src ="static/scripts/main.js"></script>
     </body>
   </html>
   ```

3. Давайте протестируем приложение Flask. В окне терминала выполните следующую команду:

   ```
   flask run
   ```

4. Откройте браузер и перейдите по предоставленному URL-адресу. Вы увидите созданное одностраничное приложение. Нажмите клавиши **CTRL+C**, чтобы завершить работу приложения.

## <a name="translate-text"></a>Перевод текста

Итак, вы получили некоторое представление о работе приложения Flask. Теперь мы сделаем следующее:

* напишем на Python код, который вызывает Переводчика и возвращает ответ;
* создадим маршрут Flask для вызова этого кода Python;
* добавим в HTML область для ввода и перевода текста, элемент управления для выбора языка и кнопку получения перевода;
* напишем код JavaScript, который позволяет пользователям взаимодействовать с приложением Flask из HTML.

### <a name="call-the-translator"></a>Вызов переводчика

Прежде всего вам нужно написать функцию для вызова Переводчика. Эта функция будет принимать два аргумента: `text_input` и `language_output`. Эта функция вызывается всякий раз, когда пользователь нажимает в приложении кнопку перевода. Текстовая область с HTML-страницы отправляется в параметре `text_input`, а значение выбранного языка — `language_output`.

1. Для начала создайте файл с именем `translate.py` в корневом каталоге рабочей папки.
2. Теперь добавьте в `translate.py` следующий код. Эта функция принимает два аргумента: `text_input` и `language_output`.
   ```python
   import os, requests, uuid, json

   # Don't forget to replace with your Cog Services subscription key!
   # If you prefer to use environment variables, see Extra Credit for more info.
   subscription_key = 'YOUR_TRANSLATOR_TEXT_SUBSCRIPTION_KEY'
   location = 'YOUR_TRANSLATOR_RESOURCE_LOCATION'
   # Don't forget to replace with your Cog Services location!
   # Our Flask route will supply two arguments: text_input and language_output.
   # When the translate text button is pressed in our Flask app, the Ajax request
   # will grab these values from our web app, and use them in the request.
   # See main.js for Ajax calls.
   def get_translation(text_input, language_output):
       base_url = 'https://api.cognitive.microsofttranslator.com'
       path = '/translate?api-version=3.0'
       params = '&to=' + language_output
       constructed_url = base_url + path + params

       headers = {
           'Ocp-Apim-Subscription-Key': subscription_key,
           'Ocp-Apim-Subscription-Region': location,
           'Content-type': 'application/json',
           'X-ClientTraceId': str(uuid.uuid4())
       }

       # You can pass more than one object in body.
       body = [{
           'text' : text_input
       }]
       response = requests.post(constructed_url, headers=headers, json=body)
       return response.json()
   ```
3. Добавьте ключ подписки Переводчика и сохраните файл.

### <a name="add-a-route-to-apppy"></a>Добавление маршрута в `app.py`

Теперь вам нужно создать в приложении Flask маршрут, который вызывает `translate.py`. Этот маршрут будет вызываться каждый раз, когда пользователь нажимает в приложении кнопку перевода.

Маршрут для этого приложения будет принимать запросы `POST`. Это связано с тем, что функция ожидает текст и целевой язык для перевода.

Flask предоставляет вспомогательные функции, которые упрощают синтаксический анализ запросов и управление ими. В предоставленном коде `get_json()` возвращает данные из запроса `POST` в формате JSON. Затем с помощью `data['text']` и `data['to']` исходный текст и целевой язык передаются в функцию `get_translation()`, которая доступна из `translate.py`. Последний шаг — получение ответа в формате JSON, который и нужно отобразить в веб-приложении.

В следующих разделах описано, как повторить эту процедуру при создании маршрутов для анализа тональности и синтеза речи.

1. Откройте `app.py` и найдите инструкцию import в верхней части `app.py`, затем добавьте рядом следующую строку:

   ```python
   import translate
   ```
   Теперь приложение Flask может использовать метод, доступный через `translate.py`.

2. Скопируйте этот код в конец `app.py` и сохраните файл:

   ```python
   @app.route('/translate-text', methods=['POST'])
   def translate_text():
       data = request.get_json()
       text_input = data['text']
       translation_output = data['to']
       response = translate.get_translation(text_input, translation_output)
       return jsonify(response)
   ```

### <a name="update-indexhtml"></a>Обновление `index.html`

Теперь, когда в приложении Flask есть функция для преобразования текста и маршрут для ее вызова, создайте код HTML для этого приложения. Представленный ниже код HTML выполняет несколько задач:

* предоставляет текстовую область, в которую пользователи будут вводить текст для перевода;
* добавляет элемент управления для выбора языка;
* добавляет элементы HTML для отображения определяемого исходного языка и оценок достоверности, полученных во время перевода;
* предоставляет текстовую область с доступом только для чтения, в которой отображаются результаты перевода;
* содержит прототипы кода для анализа тональности и синтеза речи, которые вы добавите в этот файл позднее при работе с руководством.

Давайте обновим `index.html`.

1. Откройте `index.html` и найдите такие комментарии в коде:
   ```html
   <!-- HTML provided in the following sections goes here. -->

   <!-- End -->
   ```

2. Замените комментарии в коде этим блоком HTML:
   ```html
   <div class="row">
     <div class="col">
       <form>
         <!-- Enter text to translate. -->
         <div class="form-group">
           <label for="text-to-translate"><strong>Enter the text you'd like to translate:</strong></label>
           <textarea class="form-control" id="text-to-translate" rows="5"></textarea>
         </div>
         <!-- Select output language. -->
         <div class="form-group">
           <label for="select-language"><strong>Translate to:</strong></label>
           <select class="form-control" id="select-language">
             <option value="ar">Arabic</option>
             <option value="ca">Catalan</option>
             <option value="zh-Hans">Chinese (Simplified)</option>
             <option value="zh-Hant">Chinese (Traditional)</option>
             <option value="hr">Croatian</option>
             <option value="en">English</option>
             <option value="fr">French</option>
             <option value="de">German</option>
             <option value="el">Greek</option>
             <option value="he">Hebrew</option>
             <option value="hi">Hindi</option>
             <option value="it">Italian</option>
             <option value="ja">Japanese</option>
             <option value="ko">Korean</option>
             <option value="pt">Portuguese</option>
             <option value="ru">Russian</option>
             <option value="es">Spanish</option>
             <option value="th">Thai</option>
             <option value="tr">Turkish</option>
             <option value="vi">Vietnamese</option>
           </select>
         </div>
         <button type="submit" class="btn btn-primary mb-2" id="translate">Translate text</button></br>
         <div id="detected-language" style="display: none">
           <strong>Detected language:</strong> <span id="detected-language-result"></span><br />
           <strong>Detection confidence:</strong> <span id="confidence"></span><br /><br />
         </div>

         <!-- Start sentiment code-->

         <!-- End sentiment code -->

       </form>
     </div>
     <div class="col">
       <!-- Translated text returned by the Translate API is rendered here. -->
       <form>
         <div class="form-group" id="translator-text-response">
           <label for="translation-result"><strong>Translated text:</strong></label>
           <textarea readonly class="form-control" id="translation-result" rows="5"></textarea>
         </div>

         <!-- Start voice font selection code -->

         <!-- End voice font selection code -->

       </form>

       <!-- Add Speech Synthesis button and audio element -->

       <!-- End Speech Synthesis button -->

     </div>
   </div>
   ```

На следующем этапе мы напишем код JavaScript. Это связующее звено между кодом HTML и маршрутом Flask.

### <a name="create-mainjs"></a>Создайте `main.js`  

Файл `main.js` связывает код HTML с маршрутом Flask. Приложение будет использовать XMLHttpRequest, Ajax и jQuery для отображения содержимого, а также выполнять запросы `POST` к маршрутам Flask.

В приведенном ниже коде содержимое HTML используется для создания запроса к маршруту Flask. Этот код назначает переменным значения из текстовой области и элемента управления для выбора языка, а затем передает их в запросе к `translate-text`.

Затем он выполняет итерацию в ответе и обновляет код HTML, размещая там перевод, определенный язык и оценку достоверности.

1. Создайте в интегрированной среде разработки файл с именем `main.js` в каталоге `static/scripts`.
2. Скопируйте следующий код в `static/scripts/main.js`:
   ```javascript
   //Initiate jQuery on load.
   $(function() {
     //Translate text with flask route
     $("#translate").on("click", function(e) {
       e.preventDefault();
       var translateVal = document.getElementById("text-to-translate").value;
       var languageVal = document.getElementById("select-language").value;
       var translateRequest = { 'text': translateVal, 'to': languageVal }

       if (translateVal !== "") {
         $.ajax({
           url: '/translate-text',
           method: 'POST',
           headers: {
               'Content-Type':'application/json'
           },
           dataType: 'json',
           data: JSON.stringify(translateRequest),
           success: function(data) {
             for (var i = 0; i < data.length; i++) {
               document.getElementById("translation-result").textContent = data[i].translations[0].text;
               document.getElementById("detected-language-result").textContent = data[i].detectedLanguage.language;
               if (document.getElementById("detected-language-result").textContent !== ""){
                 document.getElementById("detected-language").style.display = "block";
               }
               document.getElementById("confidence").textContent = data[i].detectedLanguage.score;
             }
           }
         });
       };
     });
     // In the following sections, you'll add code for sentiment analysis and
     // speech synthesis here.
   })
   ```

### <a name="test-translation"></a>Тестирование перевода

Теперь мы протестируем, как приложение выполняет перевод.

```
flask run
```

Откройте указанный адрес сервера. Введите текст в область ввода, выберите язык и нажмите кнопку перевода. Вы получите перевод текста. В противном случае проверьте добавленный ключ подписки.

> [!TIP]
> Если внесенные изменения не отображаются или приложение не работает ожидаемым образом, попробуйте очистить кэш или открыть окно в приватном (анонимном) режиме.

Нажмите клавиши **CTRL+C**, чтобы завершить работу приложения и перейти к следующему разделу.

## <a name="analyze-sentiment"></a>Анализ тональности

[API Анализа текста](../text-analytics/overview.md) позволяет выполнять анализ тональности, извлекать из текста ключевые фразы и определять исходный язык. В этом приложении мы применим анализ тональности для предоставленного текста и определим, является ли он положительным, негативным или нейтральным. API возвращает числовую оценку в диапазоне от 0 до 1. Оценки, близкие к 1, указывают на позитивную тональность, а оценки, близкие к 0, — на негативную.

В этом разделе мы сделаем следующее:

* напишем код Python, который вызывает API Анализа текста для анализа тональности и возвращает ответ;
* создадим маршрут Flask для вызова этого кода Python;
* обновим код HTML, добавив область для оценки тональности и кнопку запуска анализа;
* напишем код JavaScript, который позволяет пользователям взаимодействовать с приложением Flask из HTML.

### <a name="call-the-text-analytics-api"></a>Вызов API "Анализ текста"

Сейчас мы создадим функцию для вызова API Анализа текста. Эта функция будет принимать четыре аргумента: `input_text`, `input_language`, `output_text` и `output_language`. Эта функция вызывается всякий раз, когда пользователь нажимает в приложении кнопку анализа тональности. Данные, предоставленные пользователем в текстовой области и средстве выбора языка, а также определенный язык и полученный перевод предоставляются с каждым запросом. Объект ответа содержит тональность высказываний, определенную для исходного текста и перевода. В следующих разделах описано, как создать код JavaScript, который будет анализировать ответ и применять его в приложении. А пока давайте перейдем к вызову API Анализа текста.

1. Создайте файл с именем `sentiment.py` в корневом каталоге рабочей папки.
2. Теперь добавьте в `sentiment.py` следующий код.
   ```python
   import os, requests, uuid, json

   # Don't forget to replace with your Cog Services subscription key!
   subscription_key = 'YOUR_TEXT_ANALYTICS_SUBSCRIPTION_KEY'
   endpoint = "YOUR_TEXT_ANALYTICS_ENDPOINT" 
   # Our Flask route will supply four arguments: input_text, input_language,
   # output_text, output_language.
   # When the run sentiment analysis button is pressed in our Flask app,
   # the Ajax request will grab these values from our web app, and use them
   # in the request. See main.js for Ajax calls.

   def get_sentiment(input_text, input_language):
       path = '/text/analytics/v3.0/sentiment'
       constructed_url = endpoint + path

       headers = {
           'Ocp-Apim-Subscription-Key': subscription_key,
           'Content-type': 'application/json',
           'X-ClientTraceId': str(uuid.uuid4())
       }

       # You can pass more than one object in body.
       body = {
           'documents': [
               {
                   'language': input_language,
                   'id': '1',
                   'text': input_text
               },
           ]
       }
       response = requests.post(constructed_url, headers=headers, json=body)
       return response.json()
   ```
3. Добавьте ключ подписки Анализа текста и сохраните файл.

### <a name="add-a-route-to-apppy"></a>Добавление маршрута в `app.py`

Создайте в приложении Flask маршрут, который вызывает `sentiment.py`. Этот маршрут будет вызываться каждый раз, когда пользователь нажимает в приложении кнопку анализа тональности. Как и маршрут для перевода, этот маршрут будет принимать запросы `POST`, так как наша функция ожидает аргументы.

1. Откройте `app.py`, найдите инструкцию import в верхней части `app.py` и измените ее следующим образом:

   ```python
   import translate, sentiment
   ```
   Теперь приложение Flask может использовать метод, доступный через `sentiment.py`.

2. Скопируйте этот код в конец `app.py` и сохраните файл:
   ```python
   @app.route('/sentiment-analysis', methods=['POST'])
   def sentiment_analysis():
       data = request.get_json()
       input_text = data['inputText']
       input_lang = data['inputLanguage']
       response = sentiment.get_sentiment(input_text, input_lang)
       return jsonify(response)
   ```

### <a name="update-indexhtml"></a>Обновление `index.html`

Теперь, когда в приложении Flask есть функция анализа тональности и маршрут для ее вызова, переходите к написанию кода HTML приложения. Представленный ниже код HTML выполняет несколько задач:

* добавляет в приложение кнопку для анализа тональности;
* добавляет элемент с объяснением оценки тональности;
* добавляет элемент для отображения оценки тональности.

1. Откройте `index.html` и найдите такие комментарии в коде:
   ```html
   <!-- Start sentiment code-->

   <!-- End sentiment code -->
   ```

2. Замените комментарии в коде этим блоком HTML:
   ```html
   <button type="submit" class="btn btn-primary mb-2" id="sentiment-analysis">Run sentiment analysis</button></br>
   <div id="sentiment" style="display: none">
      <p>Sentiment can be labeled as "positive", "negative", "neutral", or "mixed". </p>
      <strong>Sentiment label for input:</strong> <span id="input-sentiment"></span><br />
   </div>
   ```

### <a name="update-mainjs"></a>Обновление `main.js`

В приведенном ниже коде содержимое HTML используется для создания запроса к маршруту Flask. Этот код назначает переменным значения из текстовой области и средства выбора языка, а затем передает их в запросе к маршруту `sentiment-analysis`.

Затем он выполняет итерацию в ответе и обновляет код HTML, размещая там оценки тональности.

1. Создайте в интегрированной среде разработки файл с именем `main.js` в каталоге `static`.

2. Скопируйте следующий код в `static/scripts/main.js`:
   ```javascript
   //Run sentiment analysis on input and translation.
   $("#sentiment-analysis").on("click", function(e) {
     e.preventDefault();
     var inputText = document.getElementById("text-to-translate").value;
     var inputLanguage = document.getElementById("detected-language-result").innerHTML;
     var outputText = document.getElementById("translation-result").value;
     var outputLanguage = document.getElementById("select-language").value;

     var sentimentRequest = { "inputText": inputText, "inputLanguage": inputLanguage};

     if (inputText !== "") {
       $.ajax({
         url: "/sentiment-analysis",
         method: "POST",
         headers: {
             "Content-Type":"application/json"
         },
         dataType: "json",
         data: JSON.stringify(sentimentRequest),
         success: function(data) {
           for (var i = 0; i < data.documents.length; i++) {
             if (typeof data.documents[i] !== "undefined"){
               if (data.documents[i].id === "1") {
                 document.getElementById("input-sentiment").textContent = data.documents[i].sentiment;
               }
             }
           }
           for (var i = 0; i < data.errors.length; i++) {
             if (typeof data.errors[i] !== "undefined"){
               if (data.errors[i].id === "1") {
                 document.getElementById("input-sentiment").textContent = data.errors[i].message;
               }
             }
           }
           if (document.getElementById("input-sentiment").textContent !== ''){
             document.getElementById("sentiment").style.display = "block";
           }
         }
       });
     }
   });
   // In the next section, you'll add code for speech synthesis here.
   ```

### <a name="test-sentiment-analysis"></a>Тестирование анализа тональности

Давайте протестируем, как приложение выполняет анализ тональности.

```
flask run
```

Откройте указанный адрес сервера. Введите текст в область ввода, выберите язык и нажмите кнопку перевода. Вы получите перевод текста. Теперь нажмите кнопку, которая запускает анализ тональности. Вы увидите две оценки. В противном случае проверьте добавленный ключ подписки.

> [!TIP]
> Если внесенные изменения не отображаются или приложение не работает ожидаемым образом, попробуйте очистить кэш или открыть окно в приватном (анонимном) режиме.

Нажмите клавиши **CTRL+C**, чтобы завершить работу приложения и перейти к следующему разделу.

## <a name="convert-text-to-speech"></a>Преобразование текста в речь

[API преобразования текста в речь](../speech-service/text-to-speech.md) позволяет приложению преобразовать текст в синтезированную речь. Эта служба поддерживает стандартные, нейронные и пользовательские голоса. В нашем примере приложения применяется лишь часть доступных голосов. Подробнее см. в полном списке [поддерживаемых языков](../speech-service/language-support.md#text-to-speech).

В этом разделе мы сделаем следующее:

* напишем код Python, который преобразовывает текст в речь с помощью API преобразования текста в речь;
* создадим маршрут Flask для вызова этого кода Python;
* обновим код HTML, добавив кнопку для преобразования текста в речь и элемент для воспроизведения звука;
* напишем код JavaScript, который позволяет пользователям взаимодействовать с приложением Flask.

### <a name="call-the-text-to-speech-api"></a>Вызов API преобразования текста в речь

Давайте напишем функцию, которая преобразует текст в речь. Эта функция будет принимать два аргумента: `input_text` и `voice_font`. Эта функция вызывается всякий раз, когда пользователь нажимает в приложении кнопку преобразования текста в речь. `input_text` содержит полученный перевод, возвращенный функцией перевода текста, а `voice_font` — это значение из средства выбора голоса в HTML.

1. Создайте файл с именем `synthesize.py` в корневом каталоге рабочей папки.

2. Теперь добавьте в `synthesize.py` следующий код.
   ```Python
   import os, requests, time
   from xml.etree import ElementTree

   class TextToSpeech(object):
       def __init__(self, input_text, voice_font):
           subscription_key = 'YOUR_SPEECH_SERVICES_SUBSCRIPTION_KEY'
           self.subscription_key = subscription_key
           self.input_text = input_text
           self.voice_font = voice_font
           self.timestr = time.strftime('%Y%m%d-%H%M')
           self.access_token = None

       # This function performs the token exchange.
       def get_token(self):
           fetch_token_url = 'https://westus.api.cognitive.microsoft.com/sts/v1.0/issueToken'
           headers = {
               'Ocp-Apim-Subscription-Key': self.subscription_key
           }
           response = requests.post(fetch_token_url, headers=headers)
           self.access_token = str(response.text)

       # This function calls the TTS endpoint with the access token.
       def save_audio(self):
           base_url = 'https://westus.tts.speech.microsoft.com/'
           path = 'cognitiveservices/v1'
           constructed_url = base_url + path
           headers = {
               'Authorization': 'Bearer ' + self.access_token,
               'Content-Type': 'application/ssml+xml',
               'X-Microsoft-OutputFormat': 'riff-24khz-16bit-mono-pcm',
               'User-Agent': 'YOUR_RESOURCE_NAME',
           }
           # Build the SSML request with ElementTree
           xml_body = ElementTree.Element('speak', version='1.0')
           xml_body.set('{http://www.w3.org/XML/1998/namespace}lang', 'en-us')
           voice = ElementTree.SubElement(xml_body, 'voice')
           voice.set('{http://www.w3.org/XML/1998/namespace}lang', 'en-US')
           voice.set('name', 'Microsoft Server Speech Text to Speech Voice {}'.format(self.voice_font))
           voice.text = self.input_text
           # The body must be encoded as UTF-8 to handle non-ascii characters.
           body = ElementTree.tostring(xml_body, encoding="utf-8")

           #Send the request
           response = requests.post(constructed_url, headers=headers, data=body)

           # Write the response as a wav file for playback. The file is located
           # in the same directory where this sample is run.
           return response.content
   ```
3. Добавьте ключ подписки службы распознавания речи и сохраните его.

### <a name="add-a-route-to-apppy"></a>Добавление маршрута в `app.py`

Создайте в приложении Flask маршрут, который вызывает `synthesize.py`. Этот маршрут будет вызываться каждый раз, когда пользователь нажимает в приложении кнопку преобразования текста в речь. Как и маршруты для преобразования и анализа тональности, этот маршрут будет принимать запросы `POST`, так как наша функция ожидает два аргумента: синтезируемый текст и голос для его воспроизведения.

1. Откройте `app.py`, найдите инструкцию import в верхней части `app.py` и измените ее следующим образом:

   ```python
   import translate, sentiment, synthesize
   ```
   Теперь приложение Flask может использовать метод, доступный через `synthesize.py`.

2. Скопируйте этот код в конец `app.py` и сохраните файл:

   ```Python
   @app.route('/text-to-speech', methods=['POST'])
   def text_to_speech():
       data = request.get_json()
       text_input = data['text']
       voice_font = data['voice']
       tts = synthesize.TextToSpeech(text_input, voice_font)
       tts.get_token()
       audio_response = tts.save_audio()
       return audio_response
   ```

### <a name="update-indexhtml"></a>Обновление `index.html`

Теперь, когда в приложении Flask есть функция преобразования текста в речь и маршрут для ее вызова, напишите код HTML для приложения. Представленный ниже код HTML выполняет несколько задач:

* предоставляет средство выбора голоса из раскрывающегося списка;
* добавляет кнопку для преобразования текста в речь;
* добавляет элемент для воспроизведения синтезированной речи.

1. Откройте `index.html` и найдите такие комментарии в коде:
   ```html
   <!-- Start voice font selection code -->

   <!-- End voice font selection code -->
   ```

2. Замените комментарии в коде этим блоком HTML:
   ```html
   <div class="form-group">
     <label for="select-voice"><strong>Select voice font:</strong></label>
     <select class="form-control" id="select-voice">
       <option value="(ar-SA, Naayf)">Arabic | Male | Naayf</option>
       <option value="(ca-ES, HerenaRUS)">Catalan | Female | HerenaRUS</option>
       <option value="(zh-CN, HuihuiRUS)">Chinese (Mainland) | Female | HuihuiRUS</option>
       <option value="(zh-CN, Kangkang, Apollo)">Chinese (Mainland) | Male | Kangkang, Apollo</option>
       <option value="(zh-HK, Tracy, Apollo)">Chinese (Hong Kong)| Female | Tracy, Apollo</option>
       <option value="(zh-HK, Danny, Apollo)">Chinese (Hong Kong) | Male | Danny, Apollo</option>
       <option value="(zh-TW, Yating, Apollo)">Chinese (Taiwan)| Female | Yating, Apollo</option>
       <option value="(zh-TW, Zhiwei, Apollo)">Chinese (Taiwan) | Male | Zhiwei, Apollo</option>
       <option value="(hr-HR, Matej)">Croatian | Male | Matej</option>
       <option value="(en-US, AriaRUS)">English (US) | Female | AriaRUS</option>
       <option value="(en-US, Guy24kRUS)">English (US) | Male | Guy24kRUS</option>
       <option value="(en-IE, Sean)">English (IE) | Male | Sean</option>
       <option value="(fr-FR, Julie, Apollo)">French | Female | Julie, Apollo</option>
       <option value="(fr-FR, HortenseRUS)">French | Female | Julie, HortenseRUS</option>
       <option value="(fr-FR, Paul, Apollo)">French | Male | Paul, Apollo</option>
       <option value="(de-DE, Hedda)">German | Female | Hedda</option>
       <option value="(de-DE, HeddaRUS)">German | Female | HeddaRUS</option>
       <option value="(de-DE, Stefan, Apollo)">German | Male | Apollo</option>
       <option value="(el-GR, Stefanos)">Greek | Male | Stefanos</option>
       <option value="(he-IL, Asaf)">Hebrew (Isreal) | Male | Asaf</option>
       <option value="(hi-IN, Kalpana, Apollo)">Hindi | Female | Kalpana, Apollo</option>
       <option value="(hi-IN, Hemant)">Hindi | Male | Hemant</option>
       <option value="(it-IT, LuciaRUS)">Italian | Female | LuciaRUS</option>
       <option value="(it-IT, Cosimo, Apollo)">Italian | Male | Cosimo, Apollo</option>
       <option value="(ja-JP, Ichiro, Apollo)">Japanese | Male | Ichiro</option>
       <option value="(ja-JP, HarukaRUS)">Japanese | Female | HarukaRUS</option>
       <option value="(ko-KR, HeamiRUS)">Korean | Female | Heami</option>
       <option value="(pt-BR, HeloisaRUS)">Portuguese (Brazil) | Female | HeloisaRUS</option>
       <option value="(pt-BR, Daniel, Apollo)">Portuguese (Brazil) | Male | Daniel, Apollo</option>
       <option value="(pt-PT, HeliaRUS)">Portuguese (Portugal) | Female | HeliaRUS</option>
       <option value="(ru-RU, Irina, Apollo)">Russian | Female | Irina, Apollo</option>
       <option value="(ru-RU, Pavel, Apollo)">Russian | Male | Pavel, Apollo</option>
       <option value="(ru-RU, EkaterinaRUS)">Russian | Female | EkaterinaRUS</option>
       <option value="(es-ES, Laura, Apollo)">Spanish | Female | Laura, Apollo</option>
       <option value="(es-ES, HelenaRUS)">Spanish | Female | HelenaRUS</option>
       <option value="(es-ES, Pablo, Apollo)">Spanish | Male | Pablo, Apollo</option>
       <option value="(th-TH, Pattara)">Thai | Male | Pattara</option>
       <option value="(tr-TR, SedaRUS)">Turkish | Female | SedaRUS</option>
       <option value="(vi-VN, An)">Vietnamese | Male | An</option>
     </select>
   </div>
   ```

3. Найдите такие комментарии в коде:
   ```html
   <!-- Add Speech Synthesis button and audio element -->

   <!-- End Speech Synthesis button -->
   ```

4. Замените комментарии в коде этим блоком HTML:

```html
<button type="submit" class="btn btn-primary mb-2" id="text-to-speech">Convert text-to-speech</button>
<div id="audio-playback">
  <audio id="audio" controls>
    <source id="audio-source" type="audio/mpeg" />
  </audio>
</div>
```

5. Сохраните изменения.

### <a name="update-mainjs"></a>Обновление `main.js`

В приведенном ниже коде содержимое HTML используется для создания запроса к маршруту Flask. Этот код назначает переменным значения перевода и выбранного голоса, а затем передает их в запросе к маршруту `text-to-speech`.

Затем он выполняет итерацию в ответе и обновляет код HTML, размещая там оценки тональности.

1. Создайте в интегрированной среде разработки файл с именем `main.js` в каталоге `static`.
2. Скопируйте следующий код в `static/scripts/main.js`:
   ```javascript
   // Convert text-to-speech
   $("#text-to-speech").on("click", function(e) {
     e.preventDefault();
     var ttsInput = document.getElementById("translation-result").value;
     var ttsVoice = document.getElementById("select-voice").value;
     var ttsRequest = { 'text': ttsInput, 'voice': ttsVoice }

     var xhr = new XMLHttpRequest();
     xhr.open("post", "/text-to-speech", true);
     xhr.setRequestHeader("Content-Type", "application/json");
     xhr.responseType = "blob";
     xhr.onload = function(evt){
       if (xhr.status === 200) {
         audioBlob = new Blob([xhr.response], {type: "audio/mpeg"});
         audioURL = URL.createObjectURL(audioBlob);
         if (audioURL.length > 5){
           var audio = document.getElementById("audio");
           var source = document.getElementById("audio-source");
           source.src = audioURL;
           audio.load();
           audio.play();
         }else{
           console.log("An error occurred getting and playing the audio.")
         }
       }
     }
     xhr.send(JSON.stringify(ttsRequest));
   });
   // Code for automatic language selection goes here.
   ```
3. Осталось совсем немного. Теперь вам осталось добавить в `main.js` код для автоматического выбора голоса с учетом языка, выбранного для перевода. Добавьте следующий блок кода в `main.js`:
   ```javascript
   // Automatic voice font selection based on translation output.
   $('select[id="select-language"]').change(function(e) {
     if ($(this).val() == "ar"){
       document.getElementById("select-voice").value = "(ar-SA, Naayf)";
     }
     if ($(this).val() == "ca"){
       document.getElementById("select-voice").value = "(ca-ES, HerenaRUS)";
     }
     if ($(this).val() == "zh-Hans"){
       document.getElementById("select-voice").value = "(zh-HK, Tracy, Apollo)";
     }
     if ($(this).val() == "zh-Hant"){
       document.getElementById("select-voice").value = "(zh-HK, Tracy, Apollo)";
     }
     if ($(this).val() == "hr"){
       document.getElementById("select-voice").value = "(hr-HR, Matej)";
     }
     if ($(this).val() == "en"){
       document.getElementById("select-voice").value = "(en-US, Jessa24kRUS)";
     }
     if ($(this).val() == "fr"){
       document.getElementById("select-voice").value = "(fr-FR, HortenseRUS)";
     }
     if ($(this).val() == "de"){
       document.getElementById("select-voice").value = "(de-DE, HeddaRUS)";
     }
     if ($(this).val() == "el"){
       document.getElementById("select-voice").value = "(el-GR, Stefanos)";
     }
     if ($(this).val() == "he"){
       document.getElementById("select-voice").value = "(he-IL, Asaf)";
     }
     if ($(this).val() == "hi"){
       document.getElementById("select-voice").value = "(hi-IN, Kalpana, Apollo)";
     }
     if ($(this).val() == "it"){
       document.getElementById("select-voice").value = "(it-IT, LuciaRUS)";
     }
     if ($(this).val() == "ja"){
       document.getElementById("select-voice").value = "(ja-JP, HarukaRUS)";
     }
     if ($(this).val() == "ko"){
       document.getElementById("select-voice").value = "(ko-KR, HeamiRUS)";
     }
     if ($(this).val() == "pt"){
       document.getElementById("select-voice").value = "(pt-BR, HeloisaRUS)";
     }
     if ($(this).val() == "ru"){
       document.getElementById("select-voice").value = "(ru-RU, EkaterinaRUS)";
     }
     if ($(this).val() == "es"){
       document.getElementById("select-voice").value = "(es-ES, HelenaRUS)";
     }
     if ($(this).val() == "th"){
       document.getElementById("select-voice").value = "(th-TH, Pattara)";
     }
     if ($(this).val() == "tr"){
       document.getElementById("select-voice").value = "(tr-TR, SedaRUS)";
     }
     if ($(this).val() == "vi"){
       document.getElementById("select-voice").value = "(vi-VN, An)";
     }
   });
   ```

### <a name="test-your-app"></a>Тестирование приложения

Давайте протестируем, как приложение выполняет синтез речи.

```
flask run
```

Откройте указанный адрес сервера. Введите текст в область ввода, выберите язык и нажмите кнопку перевода. Вы получите перевод текста. Теперь выберите голос и нажмите кнопку, которая запускает преобразование текста в речь. Вы услышите перевод, озвученный функцией синтезированной речи. В противном случае проверьте добавленный ключ подписки.

> [!TIP]
> Если внесенные изменения не отображаются или приложение не работает ожидаемым образом, попробуйте очистить кэш или открыть окно в приватном (анонимном) режиме.

Итак, все готово. У вас есть работающее приложение, которое выполняет переводы, анализирует тональность и синтезирует речь. Нажмите клавиши **CTRL+C**, чтобы завершить работу приложения. Обязательно ознакомьтесь с другими службами [Azure Cognitive Services](../index.yml).

## <a name="get-the-source-code"></a>Получение исходного кода

Исходный код этого проекта доступен на [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-Flask-App-Tutorial).

## <a name="next-steps"></a>Дальнейшие действия

* [Справочник по Переводчику](./reference/v3-0-reference.md)
* [Справочник по API анализа текста](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics.V2.0/operations/56f30ceeeda5650db055a3c7)
* [Справочник по API преобразования текста в речь](../speech-service/rest-text-to-speech.md)
