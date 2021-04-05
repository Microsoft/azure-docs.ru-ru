---
title: Развертывание модели PyTorch в качестве приложения "Функции Azure"
description: Используйте предварительно обученную глубокую нейронную сеть ResNet 18 из PyTorch с Функциями Azure, чтобы назначить изображению одну из 1000 меток ImageNet.
author: gvashishtha
ms.topic: tutorial
ms.date: 02/28/2020
ms.author: gopalv
ms.custom: devx-track-python, devx-track-azurepowershell
ms.openlocfilehash: 8891c29e5d8d06df6292d06ec06e5e57fb9880e7
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93422847"
---
# <a name="tutorial-deploy-a-pre-trained-image-classification-model-to-azure-functions-with-pytorch"></a>Руководство. Развертывание предварительно обученной модели классификации изображений в Функциях Azure с помощью PyTorch

В этой статье вы узнаете, как использовать Python, PyTorch и Функции Azure для загрузки предварительно обученной модели классификации изображений на основе его содержимого. Так как вся работа выполняется локально и никакие ресурсы Azure не создаются в облаке, вы не понесете никакие затраты на прохождение этого учебника.

> [!div class="checklist"]
> * Инициализация локальной среды разработки Функций Azure в Python.
> * Импорт обученной модели машинного обучения PyTorch в приложение-функцию.
> * Создание бессерверного API HTTP для классификации как одного из 1000 [классов](https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a) ImageNet.
> * Использование API из веб-приложения.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) бесплатно.
- [Python 3.7.4 или более поздней версии](https://www.python.org/downloads/release/python-374/). (Python 3.8.x и Python 3.6.x также проверяются с помощью Функций Azure.)
- [Azure Functions Core Tools](functions-run-local.md#install-the-azure-functions-core-tools).
- Редактор кода, например [Visual Studio Code](https://code.visualstudio.com/).

### <a name="prerequisite-check"></a>Проверка предварительных условий

1. В окне терминала или командном окне запустите `func --version`, чтобы убедиться, что используется версия Azure Functions Core Tools 2.7.1846 или более поздняя.
1. Выполните команду `python --version` (в ОС Linux и MacOS) или `py --version` (в ОС Windows), чтобы убедиться, что для Python возвращается версия 3.7 или более поздняя.

## <a name="clone-the-tutorial-repository"></a>Клонирование репозитория руководства

1. В окне терминала или командном окне клонируйте следующий репозиторий, используя Git.

    ```
    git clone https://github.com/Azure-Samples/functions-python-pytorch-tutorial.git
    ```

1. Перейдите в папку и изучите ее содержимое.

    ```
    cd functions-python-pytorch-tutorial
    ```

    - *start* — это папка для работы с этим учебником;
    - *end* — справочная папка с итоговым результатом проекта и полной его реализацией;
    - *resources* — папка с моделью машинного обучения и вспомогательными библиотеками;
    - *frontend* — папка с веб-сайтом, который вызывает приложение-функцию.

## <a name="create-and-activate-a-python-virtual-environment"></a>Создание и активация виртуальной среды Python

Перейдите к папке *start* и выполните следующие команды для создания и активации виртуальной среды с именем `.venv`.


# <a name="bash"></a>[bash](#tab/bash)

```bash
cd start
python -m venv .venv
source .venv/bin/activate
```

Если пакет venv не установлен Python для вашего дистрибутива Linux, выполните следующую команду:

```bash
sudo apt-get install python3-venv
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
cd start
py -m venv .venv
.venv\scripts\activate
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
cd start
py -m venv .venv
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


## <a name="import-the-pytorch-model-and-add-helper-code"></a>Импорт модели PyTorch и добавление вспомогательного кода

Чтобы изменить функцию `classify` для классификации изображений на основе его содержимого, используйте предварительно обученную модель [ResNet](https://arxiv.org/abs/1512.03385). Предварительно обученная модель, которая поступает от [PyTorch](https://pytorch.org/hub/pytorch_vision_resnet/), относит изображение к 1 из 1000 [классов ImageNet](https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a). Затем вы добавите в проект вспомогательный код и зависимости.

1. В папке *start* выполните следующую команду, чтобы скопировать код прогнозирования и метки в папку *classify*.

    # <a name="bash"></a>[bash](#tab/bash)

    ```bash
    cp ../resources/predict.py classify
    cp ../resources/labels.txt classify
    ```

    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    copy ..\resources\predict.py classify
    copy ..\resources\labels.txt classify
    ```

    # <a name="cmd"></a>[Cmd](#tab/cmd)

    ```cmd
    copy ..\resources\predict.py classify
    copy ..\resources\labels.txt classify
    ```

    ---

1. Убедитесь, что файлы *predict.py* и *labels.txt* находятся в папке *classify*. В противном случае убедитесь, что вы выполнили команду в папке *start*.

1. Откройте в текстовом редакторе файл *start/requirements.txt* и добавьте в него требуемые вспомогательному коду зависимости, которые должны выглядеть следующим образом:

    ```txt
    azure-functions
    requests
    -f https://download.pytorch.org/whl/torch_stable.html
    torch==1.5.0+cpu
    torchvision==0.6.0+cpu
    ```

1. Сохраните *requirements.txt*, а затем выполните следующую команду из папки *start*, чтобы установить зависимости.


    ```
    pip install --no-cache-dir -r requirements.txt
    ```

Установка может занять несколько минут. В это время можно перейти к изменению функции в следующем разделе.
> [!TIP]
> >В ОС Windows может возникнуть ошибка "Could not install packages due to an EnvironmentError: [Errno 2] No such file or directory:" (Не удалось установить пакеты из-за ошибки среды: [среда № 2]. Не существует такого файла или папки:), после которой указан длинный путь к файлу, например *sharded_mutable_dense_hashtable.cpython-37.pyc*. Как правило, эта ошибка возникает из-за слишком длинного пути к папке. В этом случае задайте для раздела реестра `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem@LongPathsEnabled` значение `1`, чтобы включить длинные пути. Можно также проверить, где установлен интерпретатор Python. Если у этого расположения длинный путь, попробуйте переустановить интерпретатор в папке с более коротким путем.

## <a name="update-the-function-to-run-predictions"></a>Обновление функции для выполнения прогнозов

1. Чтобы импортировать стандартную библиотеку JSON и вспомогательные функции *predict*, откройте файл *classify/\_\_init\_\_.py* в текстовом редакторе и добавьте следующие строки после существующих инструкций `import`.

    :::code language="python" source="~/functions-pytorch/end/classify/__init__.py" range="1-6" highlight="5-6":::

1. Замените все содержимое функции `main` следующим кодом:

    :::code language="python" source="~/functions-pytorch/end/classify/__init__.py" range="8-19":::

    Эта функция получает URL-адрес изображения в параметре строки запроса с именем `img`. Затем она вызывает `predict_image_from_url` из вспомогательной библиотеки, чтобы скачать и классифицировать изображение с помощью модели PyTorch. Затем функция возвращает ответ HTTP с результатами.

    > [!IMPORTANT]
    > Так как конечная точка HTTP вызывается веб-страницей, размещенной в другом домене, ответ содержит заголовок `Access-Control-Allow-Origin` в соответствии с требованиями к общему доступу к ресурсам независимо от источника (CORS), предъявляемыми браузером.
    >
    > В рабочем приложении измените `*` на источник веб-страницы для повышения безопасности.

1. Сохраните изменения, а затем, предполагая, что зависимости установлены, снова запустите локальный узел функции с помощью `func start`. Не забудьте запустить узел в папке *start* с активированной виртуальной средой. В противном случае узел запустится, но при вызове функции будут отображаться ошибки.

    ```
    func start
    ```

1. В браузере откройте следующий URL-адрес для вызова функции с URL-адресом изображения бернского зенненхунда и убедитесь, что возвращенный JSON классифицирует изображение как изображение с бернским зенненхундом.

    ```
    http://localhost:7071/api/classify?img=https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/Bernese-Mountain-Dog-Temperament-long.jpg
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

    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/Bernese-Mountain-Dog-Temperament-long.jpg`
    - `https://github.com/Azure-Samples/functions-python-pytorch-tutorial/blob/master/resources/assets/bald-eagle.jpg?raw=true`
    - `https://raw.githubusercontent.com/Azure-Samples/functions-python-pytorch-tutorial/master/resources/assets/penguin.jpg`

1. Нажмите кнопку **Отправить**, чтобы вызвать конечную точку функции для классификации изображения.

    ![Снимок экрана готового проекта](media/machine-learning-pytorch/screenshot.png)

    Если при попытке отправить URL-адрес изображения браузер выдает сообщение об ошибке, проверьте терминал, в котором выполняется приложение-функция. Если отображается сообщение об ошибке наподобие "No module found 'PIL'" (Не найден модуль PIL), возможно, вы запустили приложение-функцию в папке *start* без предварительной активации созданной ранее виртуальной среды. Если ошибки по-прежнему отображаются, снова запустите `pip install -r requirements.txt` с активированной виртуальной средой и найдите ошибки.

## <a name="clean-up-resources"></a>Очистка ресурсов

Все инструкции из этого учебника выполняются локально на вашем компьютере, поэтому нет необходимости в очистке ресурсов или служб Azure.

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как создать и настроить конечную точку API HTTP с помощью Функций Azure для классификации изображений с использованием модели PyTorch. Вы также научились вызывать API из веб-приложения. С помощью описанных в этом руководстве методов можно создавать API любой сложности, работая при этом в бессерверной вычислительной модели, которую предоставляют Функции Azure.

См. также

- [Руководство. Создание и развертывание бессерверных функций Azure на языке Python с помощью Visual Studio Code](https://code.visualstudio.com/docs/python/tutorial-azure-functions).
- [Azure Functions Python Developer Guide](./functions-reference-python.md) (Справочник по Функциям Azure для разработчика Python)


> [!div class="nextstepaction"]
> [Руководство по развертыванию функции в Функциях Azure с помощью Azure CLI](./functions-run-local.md#publish)
