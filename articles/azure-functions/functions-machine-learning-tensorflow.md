---
title: Использование Python и TensorFlow для машинного обучения в Azure
description: Классификация изображения на основе его содержимого с помощью модели машинного обучения с использованием Python, TensorFlow и Функций Azure.
author: anthonychu
ms.topic: tutorial
ms.date: 01/15/2020
ms.author: antchu
ms.custom: mvc, devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: ca9ce27583168dfee1a597fce559afad38a3a8c7
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95994610"
---
# <a name="tutorial-apply-machine-learning-models-in-azure-functions-with-python-and-tensorflow"></a>Руководство по Применение моделей машинного обучения в Функциях Azure с использованием Python и TensorFlow

В этой статье вы узнаете, как классифицировать изображение на основе его содержимого с помощью модели машинного обучения, используя Python, TensorFlow и Функции Azure. Так как вся работа выполняется локально и никакие ресурсы Azure не создаются в облаке, вы не понесете никакие затраты на прохождение этого учебника.

> [!div class="checklist"]
> * Инициализация локальной среды разработки Функций Azure в Python.
> * Импорт настраиваемой модели машинного обучения TensorFlow в приложение-функцию.
> * Создание бессерверного API HTTP для классификации изображения как изображения с собакой или котом.
> * Использование API из веб-приложения.

## <a name="prerequisites"></a>Предварительные требования 

- Учетная запись Azure с активной подпиской. [Создайте учетную запись бесплатно](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- [Python версии 3.7.4](https://www.python.org/downloads/release/python-374/). (Python 3.7.4 и Python 3.6.x совместимы с Функциями Azure. Python 3.8 и более поздние версии пока не поддерживаются.)
- [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools).
- Редактор кода, например [Visual Studio Code](https://code.visualstudio.com/).

### <a name="prerequisite-check"></a>Проверка предварительных условий

1. В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 2.7.1846 или более поздняя.
1. Выполните команду `python --version` (в ОС Linux и MacOS) или `py --version` (в ОС Windows), чтобы убедиться, что для Python возвращается версия 3.7 или более поздняя.

## <a name="clone-the-tutorial-repository"></a>Клонирование репозитория руководства

1. В окне терминала или командном окне клонируйте следующий репозиторий, используя Git.

    ```
    git clone https://github.com/Azure-Samples/functions-python-tensorflow-tutorial.git
    ```

1. Перейдите в папку и изучите ее содержимое.

    ```
    cd functions-python-tensorflow-tutorial
    ```

    - *start* — это папка для работы с этим учебником;
    - *end* — справочная папка с итоговым результатом проекта и полной его реализацией;
    - *resources* — папка с моделью машинного обучения и вспомогательными библиотеками;
    - *frontend* — папка с веб-сайтом, который вызывает приложение-функцию.
    
## <a name="create-and-activate-a-python-virtual-environment"></a>Создание и активация виртуальной среды Python

Перейдите к папке *start* и выполните следующие команды для создания и активации виртуальной среды с именем `.venv`. Обязательно используйте версию Python 3.7, которую поддерживают Функции Azure.


# <a name="bash"></a>[bash](#tab/bash)

```bash
cd start
```

```bash
python -m venv .venv
```

```bash
source .venv/bin/activate
```

Если пакет venv не установлен Python для вашего дистрибутива Linux, выполните следующую команду:

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
cd start
```

```powershell
py -3.7 -m venv .venv
```

```powershell
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd start
```

```cmd
py -3.7 -m venv .venv
```

```cmd
.venv\scripts\activate
```

---

Все последующие команды будут выполняться в этой активированной виртуальной среде. (Чтобы выйти из виртуальной среды, выполните команду `deactivate`.)


## <a name="create-a-local-functions-project"></a>Создание локального проекта службы "Функции"

В Функциях Azure проект функций представляет собой контейнер для одной или нескольких отдельных функций, каждая из которых реагирует на конкретный триггер. Все функции в проекте совместно используют те же локальные конфигурации и конфигурации размещения. В этом разделе вы создадите проект функции, содержащий одну шаблонную функцию с именем `classify`, которая предоставляет конечную точку HTTP. Мы добавим более конкретный код для нее в одном из следующих разделов.

1. В папке *start* инициализируйте приложение-функцию Python с помощью Azure Functions Core Tools.

    ```
    func init --worker-runtime python
    ```

    После инициализации папка *start* содержит различные файлы проекта, включая файлы конфигурации [local.settings.json](functions-run-local.md#local-settings-file) и [host.json](functions-host-json.md). Файл *local.settings.json* может содержать секреты, скачанные из Azure, поэтому файл по умолчанию исключен из системы управления версиями в *GITIGNORE*-файле.

    > [!TIP]
    > Так как проект функций привязан к определенной среде выполнения, все функции в проекте должны быть написаны на одном языке.

1. Добавьте функцию в проект с помощью следующей команды, где аргумент `--name` — уникальное имя функции, а аргумент `--template` указывает триггер функции. С помощью `func new` создайте вложенную папку с именем функции, которая содержит файл кода на выбранном для проекта языке и файл конфигурации *function.json*.

    ```
    func new --name classify --template "HTTP trigger"
    ```

    Эта команда создает папку *classify*, которая соответствует имени функции. В этой папке содержатся два файла: *\_\_init\_\_.py*, содержащий код функции, и *function.json*, который описывает триггер функции и его входные и выходные привязки. Дополнительные сведения о содержимом этих файлов см. в разделе [об изучении содержимого файла](./create-first-function-cli-python.md#optional-examine-the-file-contents) краткого руководства по программированию на Python.


## <a name="run-the-function-locally"></a>Локальное выполнение функции

1. Активируйте функцию, запустив локальное хост-приложение среды выполнения Функций Azure в папке *start*:

    ```
    func start
    ```
    
1. Когда в выходных данных отобразится конечная точка `classify`, перейдите по URL-адресу ```http://localhost:7071/api/classify?name=Azure```. В выходных данных должно отобразиться сообщение "Hello Azure!".

1. Чтобы остановить хост, используйте клавиши **CTRL**-**C**.


## <a name="import-the-tensorflow-model-and-add-helper-code"></a>Импорт модели TensorFlow и добавление вспомогательного кода

Чтобы изменить функцию `classify` для классификации изображения на основе его содержимого, используйте предварительно созданную модель TensorFlow, которая была обучена и экспортирована из службы "Пользовательское визуальное распознавание Azure". Модель, которая содержится в папке *resources* в примере, клонированном ранее, классифицирует изображение, определяя, кто на нем: кошка или собака. Затем вы добавите в проект вспомогательный код и зависимости.

Чтобы создать собственную модель с помощью службы "Пользовательское визуальное распознавание" уровня "Бесплатный", следуйте инструкциям из [репозитория примера проекта](https://github.com/Azure-Samples/functions-python-tensorflow-tutorial/blob/master/train-custom-vision-model.md).

> [!TIP]
> Если вы хотите разместить модель TensorFlow независимо от приложения-функции, вы можете подключить общую папку, содержащую модель, к приложению-функции Linux. Подробные сведения см. в статье [Подключение общей папки к приложению-функции Python с помощью Azure CLI](./scripts/functions-cli-mount-files-storage-linux.md).

1. В папке *start* выполните следующую команду, чтобы скопировать файлы модели в папку *classify*. Не забудьте добавить `\*` в эту команду. 

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    cp ../resources/model/* classify
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    copy ..\resources\model\* classify
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    copy ..\resources\model\* classify
    ```
    
    ---
    
1. Убедитесь, что файлы *model.pb* и *labels.txt* находятся в папке *classify*. В противном случае убедитесь, что вы выполнили команду в папке *start*.

1. В папке *start* выполните следующую команду, чтобы скопировать файл с вспомогательным кодом в папку *classify*:

    # <a name="bash"></a>[bash](#tab/bash)
    
    ```bash
    cp ../resources/predict.py classify
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    copy ..\resources\predict.py classify
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    copy ..\resources\predict.py classify
    ```
    
    ---

1. Убедитесь, что файл *predict.py* находится в папке *classify*.

1. Откройте в текстовом редакторе файл *start/requirements.txt* и добавьте в него следующие зависимости, требуемые вспомогательному коду.

    ```txt
    tensorflow==1.14
    Pillow
    requests
    ```
    
1. Сохраните файл *requirements.txt*.

1. Установите зависимости, выполнив следующую команду в папке *start*: Установка может занять несколько минут. В это время можно перейти к изменению функции в следующем разделе.

    ```
    pip install --no-cache-dir -r requirements.txt
    ```
    
    В ОС Windows может возникнуть ошибка "Could not install packages due to an EnvironmentError: [Errno 2] No such file or directory:" (Не удалось установить пакеты из-за ошибки среды: [среда № 2]. Не существует такого файла или папки:), после которой указан длинный путь к файлу, например *sharded_mutable_dense_hashtable.cpython-37.pyc*. Как правило, эта ошибка возникает из-за слишком длинного пути к папке. В этом случае задайте для раздела реестра `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem@LongPathsEnabled` значение `1`, чтобы включить длинные пути. Можно также проверить, где установлен интерпретатор Python. Если у этого расположения длинный путь, попробуйте переустановить интерпретатор в папке с более коротким путем.

> [!TIP]
> При вызове *predict.py* для создания первого прогноза функция `_initialize` загружает модель TensorFlow с диска и кэширует ее в глобальных переменных. Благодаря этому кэшированию последующие прогнозы ускоряются. Дополнительные сведения об использовании глобальных переменных см. в [справочнике разработчика Python по Функциям Azure](functions-reference-python.md#global-variables).

## <a name="update-the-function-to-run-predictions"></a>Обновление функции для выполнения прогнозов

1. Чтобы импортировать стандартную библиотеку JSON и вспомогательные функции *predict*, откройте файл *classify/\_\_init\_\_.py* в текстовом редакторе и добавьте следующие строки после существующих инструкций `import`.

    :::code language="python" source="~/functions-python-tensorflow-tutorial/end/classify/__init__.py" range="1-6" highlight="5-6":::

1. Замените все содержимое функции `main` следующим кодом:

    :::code language="python" source="~/functions-python-tensorflow-tutorial/end/classify/__init__.py" range="8-19":::

    Эта функция получает URL-адрес изображения в параметре строки запроса с именем `img`. Затем она вызывает `predict_image_from_url` из вспомогательной библиотеки, чтобы скачать и классифицировать изображение с помощью модели TensorFlow. Затем функция возвращает ответ HTTP с результатами. 

    > [!IMPORTANT]
    > Так как конечная точка HTTP вызывается веб-страницей, размещенной в другом домене, ответ содержит заголовок `Access-Control-Allow-Origin` в соответствии с требованиями к общему доступу к ресурсам независимо от источника (CORS), предъявляемыми браузером.
    >
    > В рабочем приложении измените `*` на источник веб-страницы для повышения безопасности.

1. Сохраните изменения, а затем, предполагая, что зависимости установлены, снова запустите локальный узел функции с помощью `func start`. Не забудьте запустить узел в папке *start* с активированной виртуальной средой. В противном случае узел запустится, но при вызове функции будут отображаться ошибки.

    ```
    func start
    ```

1. В браузере откройте следующий URL-адрес для вызова функции с URL-адресом изображения кота и убедитесь, что возвращенный JSON классифицирует изображение как изображение с котом.

    ```
    http://localhost:7071/api/classify?img=https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat1.png
    ```
    
1. Оставьте узел запущенным, так как он используется на следующем шаге. 

### <a name="run-the-local-web-app-front-end-to-test-the-function"></a>Запуск интерфейса локального веб-приложения для проверки функции

Чтобы проверить вызов конечной точки функции из другого веб-приложения, можно воспользоваться простым приложением, расположенным в папке *frontend* репозитория.

1. Откройте новое окно терминала или командное окно и активируйте виртуальную среду (как описано выше в разделе [Создание и активация виртуальной среды Python](#create-and-activate-a-python-virtual-environment)).

1. Перейдите в папку *frontend* в репозитории.

1. Запустите HTTP-сервер с Python.

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash 
    python -m http.server
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    py -m http.server
    ```

    # <a name="cmd"></a>[Cmd](#tab/cmd)

    ```cmd
    py -m http.server
    ```

1. В браузере перейдите по адресу `localhost:8000`, введите один из следующих URL-адресов фотографий в текстовое поле или используйте URL-адрес любого общедоступного изображения.

    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat1.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/cat2.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/dog1.png`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-tensorflow-tutorial/master/resources/assets/samples/dog2.png`
    
1. Нажмите кнопку **Отправить**, чтобы вызвать конечную точку функции для классификации изображения.

    ![Снимок экрана готового проекта](media/functions-machine-learning-tensorflow/functions-machine-learning-tensorflow-screenshot.png)

    Если при попытке отправить URL-адрес изображения браузер выдает сообщение об ошибке, проверьте терминал, в котором выполняется приложение-функция. Если отображается сообщение об ошибке наподобие "No module found 'PIL'" (Не найден модуль PIL), возможно, вы запустили приложение-функцию в папке *start* без предварительной активации созданной ранее виртуальной среды. Если ошибки по-прежнему отображаются, снова запустите `pip install -r requirements.txt` с активированной виртуальной средой и найдите ошибки.

> [!NOTE]
> Модель всегда классифицирует содержимое изображения как "Кошка" или "Собака", независимо от того, есть ли на изображении одно из этих животных. По умолчанию изображение классифицируется как "Собака". Например, изображения тигров и пантер классифицируются как "Кошка", а изображения слонов, морковок или самолетов — как "Собака".

## <a name="clean-up-resources"></a>Очистка ресурсов

Все инструкции из этого учебника выполняются локально на вашем компьютере, поэтому нет необходимости в очистке ресурсов или служб Azure.

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как создать и настроить конечную точку API HTTP с помощью Функций Azure для классификации изображений с использованием модели TensorFlow. Вы также научились вызывать API из веб-приложения. С помощью описанных в этом руководстве методов можно создавать API любой сложности, работая при этом в бессерверной вычислительной модели, которую предоставляют Функции Azure.

> [!div class="nextstepaction"]
> [Руководство по развертыванию функции в Функциях Azure с помощью Azure CLI](./functions-run-local.md#publish)

См. также

- [Руководство. Создание и развертывание бессерверных функций Azure на языке Python с помощью Visual Studio Code](https://code.visualstudio.com/docs/python/tutorial-azure-functions).
- [Azure Functions Python Developer Guide](./functions-reference-python.md) (Справочник по Функциям Azure для разработчика Python)
- [Подключение общей папки к приложению-функции Python с помощью Azure CLI](./scripts/functions-cli-mount-files-storage-linux.md)
