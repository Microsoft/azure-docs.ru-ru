---
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: include
ms.date: 03/04/2021
ms.author: erhopf
ms.openlocfilehash: f88ca39a378e997bb72300188166192e3383f6f1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102620184"
---
## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/cognitive-services).
* Ресурс "Иммерсивное средство чтения", настроенный для проверки подлинности Azure Active Directory. Инструкции по настройке см. [здесь](../../how-to-create-immersive-reader.md).  Вам потребуются некоторые значения, созданные здесь при настройке свойств среды. Сохраните результаты своего сеанса в текстовом файле для использования в будущем.
* [Git](https://git-scm.com/).
* [Пакет SDK иммерсивного средства чтения.](https://github.com/microsoft/immersive-reader-sdk)
* [Python](https://www.python.org/downloads/) и [pip](https://docs.python.org/3/installing/index.html) Начиная с Python 3.4, pip включается по умолчанию с двоичными установщиками Python.
* [Flask.](https://flask.palletsprojects.com/en/1.0.x/)
* [Jinja.](http://jinja.pocoo.org/docs/2.10/)
* [virtualenv](https://virtualenv.pypa.io/en/latest/) и [virtualenvwrapper-win для Windows](https://pypi.org/project/virtualenvwrapper-win/) или [virtualenvwrapper для OSX](https://virtualenvwrapper.readthedocs.io/en/latest/).
* [Модуль запросов.](https://pypi.org/project/requests/2.7.0/)
* Интегрированная среда разработки, такая как [Visual Studio Code](https://code.visualstudio.com/).

## <a name="configure-authentication-credentials"></a>Определение учетных данных для проверки подлинности

Создайте новый файл с расширением **.env** и вставьте в него следующие имена и значения. Укажите значения, заданные при создании ресурса "Иммерсивное средство чтения".

```text
TENANT_ID={YOUR_TENANT_ID}
CLIENT_ID={YOUR_CLIENT_ID}
CLIENT_SECRET={YOUR_CLIENT_SECRET}
SUBDOMAIN={YOUR_SUBDOMAIN}
```

Не фиксируйте этот файл в системе управления версиями, так как он содержит секреты, не предназначенные для публикации.

Защитите конечную точку API **getimmersivereadertoken** какой-либо формой проверки подлинности, например [OAuth](https://oauth.net/2/). Проверка подлинности не позволяет неавторизованным пользователям получать маркеры для использования в службе иммерсивного средства чтения и управления выставлением счетов. Эти действия не рассматриваются в этом руководстве.

## <a name="create-a-python-web-app-on-windows"></a>Создание веб-приложения Python в Windows

Создайте веб-приложение Python с помощью `flask` в Windows.

Установите [Git](https://git-scm.com/).

После установки Git откройте командную строку и клонируйте репозиторий пакета SDK для иммерсивного средства чтения в любую папку на компьютере.

```cmd
git clone https://github.com/microsoft/immersive-reader-sdk.git
```

установите [Python](https://www.python.org/downloads/);

Установите флажок **Add Python to PATH** (Добавить Python в PATH).

![Диалоговое окно установки Python в Windows, шаг 1](../../media/pythoninstallone.jpg)

Добавьте **дополнительные функции**, установив нужные флажки, а затем щелкните **Далее**.

![Диалоговое окно установки Python в Windows, шаг 2](../../media/pythoninstalltwo.jpg)

Выберите **Custom installation** (Настраиваемая установка) и задайте корневую папу для установки, например `C:\Python37-32\`. Щелкните **Установить**.

![Диалоговое окно установки Python в Windows, шаг 3](../../media/pythoninstallthree.jpg)

После завершения установки Python откройте командную строку и командой `cd` перейдите в папку со скриптами Python.

```cmd
cd C:\Python37-32\Scripts
```

Установите Flask.

```cmd
pip install flask
```

Установите Jinja2. Это полнофункциональный механизм шаблонов для Python.

```cmd
pip install jinja2
```

Установите virtualenv. Это средство для создания изолированных сред Python.

```cmd
pip install virtualenv
```

Установите virtualenvwrapper-win. virtualenvwrapper предназначен для упрощения работы с virtualenv.

```cmd
pip install virtualenvwrapper-win
```

Установите модуль "requests". Requests (Запросы) содержит лицензированную для Apache2 библиотеку HTTP на языке Python.

```cmd
pip install requests
```

Установите модуль python-dotenv. Этот модуль считывает пару "ключ-значение" из ENV-файла и сохраняет их в переменную среды.

```cmd
pip install python-dotenv
```

Создайте виртуальную среду.

```cmd
mkvirtualenv advanced-python
```

Командой `cd` перейдите в корневую папку примера проекта.

```cmd
cd C:\immersive-reader-sdk\js\samples\advanced-python
```

Подключите пример проекта к этой среде. Это действие сопоставляет новую виртуальную среду с корневой папкой примера проекта.

```cmd
setprojectdir .
```

Включите виртуальную среду.

```cmd
activate
```

Теперь проект должен стать активным, и в командной строке отобразится примерно следующее: `(advanced-python) C:\immersive-reader-sdk\js\samples\advanced-python>`.

Отключите среду.

```cmd
deactivate
```

Префикс `(advanced-python)` исчезнет, так как среда отключена.

Чтобы повторно включить среду, выполните команду `workon advanced-python` из корневой папки примера проекта.

```cmd
workon advanced-python
```

### <a name="start-the-immersive-reader-with-sample-content"></a>Запуск иммерсивного средства чтения с примером содержимого

При включенной среде запустите пример проекта, введя команду `flask run` в корневой папке проекта.

```cmd
flask run
```

Откройте браузер и перейдите по адресу http://localhost:5000.

## <a name="create-a-python-web-app-on-osx"></a>Создание веб-приложения Python в OSX

Создайте веб-приложение Python с помощью `flask` в OSX.

Установите [Git](https://git-scm.com/).

После установки Git откройте терминал и клонируйте репозиторий пакета SDK для иммерсивного средства чтения в любую папку на компьютере.

```bash
git clone https://github.com/microsoft/immersive-reader-sdk.git
```

установите [Python](https://www.python.org/downloads/);

Корневая папка Python, например `Python37-32`, должна находиться в папке Applications.

После завершения установки Python откройте окно терминала и командой `cd` перейдите в папку со скриптами Python.

```bash
cd immersive-reader-sdk/js/samples/advanced-python
```

установить pip.

```bash
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```

Выполните следующий фрагмент кода, чтобы установить pip для активного пользователя, чтобы избежать проблем с разрешениями.

```bash
python get-pip.py --user
```

```bash
sudo nano /etc/paths
```

- В ответ на запрос введите свой пароль.
- Добавьте расположение установки pip в переменную пути.
- Перейдите в нижнюю часть файла и введите путь, который нужно добавить в конец списка, например `PATH=$PATH:/usr/local/bin`.
- Чтобы выйти, нажмите клавиши **CTRL+X**.
- Введите **Y** для сохранения изменений буфера.
- Вот и все! Чтобы проверить установку, в новом окне терминала введите `echo $PATH`.

Установите Flask.

```bash
pip install flask --user
```

Установите Jinja2. Это полнофункциональный механизм шаблонов для Python.

```bash
pip install Jinja2 --user
```

Установите virtualenv. Это средство для создания изолированных сред Python.

```bash
pip install virtualenv --user
```

Установите virtualenvwrapper. virtualenvwrapper предназначен для упрощения работы с virtualenv.

```bash
pip install virtualenvwrapper --user
```

Установите модуль "requests". Requests (Запросы) содержит лицензированную для Apache2 библиотеку HTTP на языке Python.

```bash
pip install requests --user
```

Установите модуль python-dotenv. Этот модуль считывает пару "ключ-значение" из ENV-файла и сохраняет их в переменную среды.

```bash
pip install python-dotenv --user
```

Выберите папку, в которой вы хотите разместить виртуальные среды, и выполните эту команду:

```bash
mkdir ~/.virtualenvs
```

С помощью `cd` перейдите в папку примера приложения Python с SDK для иммерсивного средства чтения.

```bash
cd immersive-reader-sdk/js/samples/advanced-python
```

Создайте виртуальную среду.

```bash
mkvirtualenv -p /usr/local/bin/python3 advanced-python
```

Подключите пример проекта к этой среде. Это действие сопоставляет новую виртуальную среду с корневой папкой примера проекта.

```bash
setprojectdir .
```

Включите виртуальную среду.

```bash
activate
```

Теперь проект должен стать активным, и в командной строке отобразится примерно следующее: `(advanced-python) /immersive-reader-sdk/js/samples/advanced-python>`.

Отключите среду.

```bash
deactivate
```

Префикс `(advanced-python)` исчезнет, так как среда отключена.

Чтобы повторно включить среду, выполните команду `workon advanced-python` из корневой папки примера проекта.

```bash
workon advanced-python
```

## <a name="start-the-immersive-reader-with-sample-content"></a>Запуск иммерсивного средства чтения с примером содержимого

При включенной среде запустите пример проекта, введя команду `flask run` в корневой папке проекта.

```bash
flask run
```

Откройте браузер и перейдите по адресу http://localhost:5000.

## <a name="next-steps"></a>Дальнейшие действия

* Ознакомьтесь со статьей о [пакете SDK для иммерсивного средства чтения](https://github.com/microsoft/immersive-reader-sdk) и [справкой по этому пакету](../../reference.md).
* Просмотрите примеры кода на сайте [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/).
