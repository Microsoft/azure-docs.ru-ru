---
title: Краткое руководство. Создание веб-приложения Node.js
description: Быстрое развертывание первого приложения Hello World на Node.js в Службе приложений Azure.
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 08/01/2020
ms.custom: mvc, devcenter, seodec18
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-nodejs-uiex
ms.openlocfilehash: 6c6f0543dcfbecd16ba4176272f928ffd0eb54de
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101735120"
---
# <a name="create-a-nodejs-web-app-in-azure"></a>Создание веб-приложений Node.js в Azure

::: zone pivot="platform-windows"  

Чтобы начать работу со Службой приложений Azure, создайте локально приложение Node.js/Express с помощью Visual Studio Code, а затем разверните его в облаке. Так как вы используете бесплатный уровень Службы приложений, для прохождения этого краткого руководства никакие затраты не требуются.

## <a name="prerequisites"></a>Предварительные требования

- Учетная запись Azure с активной подпиской. [Создайте учетную запись](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension) бесплатно.
- <a href="https://git-scm.com/" target="_blank">установите Git</a>;
- [Node.js и NPM](https://nodejs.org). Выполнив команду `node --version`, убедитесь, что платформа Node.js установлена.
- [Visual Studio Code](https://code.visualstudio.com/).
- [Расширение Службы приложений Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) для Visual Studio Code.

## <a name="clone-and-run-a-local-nodejs-application"></a>Клонирование и запуск локального приложения Node.js

1. На локальном компьютере откройте терминал и выполните клонирование примера репозитория:

    ```bash
    git clone https://github.com/Azure-Samples/nodejs-docs-hello-world
    ```

1. Перейдите в папку нового приложения:

    ```bash
    cd nodejs-docs-hello-world
    ```

1. Запустите приложение, чтобы проверить его локально:

    ```bash
    npm start
    ```
    
1. Откройте веб-браузер и перейдите по адресу `http://localhost:1337`. В браузере должно отобразиться сообщение "Hello World!".

1. Нажмите **Ctrl**+**C** в терминале, чтобы остановить работу сервера.

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=create-app)

## <a name="deploy-the-app-to-azure"></a>Развертывание приложения в Azure

В этом разделе вы развернете приложение Node.js в Azure с помощью VS Code и расширения Службы приложений Azure.

1. В окне терминала убедитесь, что вы находитесь в папке *nodejs-docs-hello-world*, а затем запустите Visual Studio Code с помощью следующей команды:

    ```bash
    code .
    ```

1. На панели действий VS Code выберите эмблему Azure, чтобы отобразить обозреватель **СЛУЖБА ПРИЛОЖЕНИЙ AZURE**. Выберите **Войти в Azure...** и следуйте инструкциям. (Если произойдут ошибки, см. раздел [Устранение неполадок со входом в Azure](#troubleshooting-azure-sign-in) ниже.) После входа в обозревателе должно отобразиться имя вашей подписки Azure.

    ![Вход в Azure](media/quickstart-nodejs/sign-in.png)

1. В обозревателе **СЛУЖБА ПРИЛОЖЕНИЙ AZURE** в VS Code щелкните значок с направленной вверх синей стрелкой, чтобы развернуть приложение в Azure. (Ту же команду можно также вызвать из **палитры команд** (**Ctrl**+**Shift**+**P**). Для этого следует ввести "развернуть в веб-приложении" и выбрать пункт **Служба приложений Azure: развертывание в веб-приложении**).

    :::image type="content" source="media/quickstart-nodejs/deploy.png" alt-text="Снимок экрана Службы приложений Azure в VS Code с выбранным синим значком со стрелкой.":::
        
1. Выберите папку *nodejs-docs-hello-world*.

1. Выберите вариант создания на основе операционной системы, на которую требуется развернуть:

    - Linux: Щелкните **Создание нового веб-приложения**
    - Windows: Щелкните **Создание нового веб-приложения... Дополнительно**

1. Введите глобально уникальное имя веб-приложения и нажмите клавишу **ВВОД**. Имя должно быть уникальным во всех системах Azure и состоять только из буквенно-цифровых символов ("A-Z", "a-z" и "0-9") и дефисов ("-").

1. Если вы намерены использовать Linux, выберите версию Node.js при появлении соответствующего запроса. Рекомендуется использовать версию **LTS**.

1. Если вы намерены использовать Windows, следуйте дополнительным запросам:
    1. Выберите **Создать группу ресурсов**, а затем введите имя для группы ресурсов, например `AppServiceQS-rg`.
    1. Выберите **Windows** в поле операционной системы.
    1. Выберите **Создать новый план службы приложений**, введите имя плана (например, `AppServiceQS-plan`), а затем выберите **F1 Free** для ценовой категории.
    1. Выберите **Пропустить** при появлении запроса о Application Insights.
    1. Выберите регион, расположенный рядом с вами или рядом с ресурсами, к которым вы хотите получить доступ.

1. После ответа на все запросы, в VS Code отображаются ресурсы Azure, созданные для вашего приложения.

    При развертывании в Linux, нажмите **Да**, когда появится запрос на обновление конфигурации, чтобы выполнить `npm install` на целевом сервере Linux.

    ![Запрос на обновление конфигурации на целевом сервере Linux](media/quickstart-nodejs/server-build.png)

1. Выберите **Да** при появлении запроса **Always deploy the workspace "nodejs-docs-hello-world" to (app name)"** (Всегда развертывайте рабочую область "nodejs-docs-hello-world" в (имя приложения)). Если выбрать **Да**, VS Code будет в дальнейшем автоматически выполнять развертывания в то же веб-приложение Службы приложений.

1. При развертывании в Linux выберите **Обзор веб-сайта** в командной строке, чтобы просмотреть новое развернутое веб-приложение после завершения развертывания. В браузере должно отобразиться сообщение "Hello World!".

1. При развертывании в Windows необходимо сначала задать номер версии Node.js для веб-приложения:

    1. В VS Code разверните узел для новой службы приложений, щелкните правой кнопкой мыши **Параметры приложения** и выберите **Добавить новый параметр...** :

        ![Команда добавления параметра приложения](media/quickstart-nodejs/add-setting.png)

    1. Введите `WEBSITE_NODE_DEFAULT_VERSION` для ключа параметра.
    1. Введите `10.15.2` для значения параметра.
    1. Щелкните правой кнопкой мыши узел службы приложений и выберите **Перезапустить**.

        ![Команда перезапуска службы приложений](media/quickstart-nodejs/restart.png)

    1. Еще раз щелкните правой кнопкой мыши узел для службы приложений и выберите **Обзор веб-сайта**.

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=deploy-app)

### <a name="troubleshooting-azure-sign-in"></a>Устранение неполадок со входом в Azure

Если при входе в Azure отображается сообщение об ошибке **"Не удается найти подписку с именем [идентификатор подписки]"** , возможно, вы находитесь за прокси-сервером и не можете связаться с API Azure. Укажите в переменных среды `HTTP_PROXY` и `HTTPS_PROXY` параметры прокси-сервера, используя команду терминала `export`.

```bash
export HTTPS_PROXY=https://username:password@proxy:8080
export HTTP_PROXY=http://username:password@proxy:8080
```

Если настройка переменных среды не устранит проблему, свяжитесь с нами, нажав кнопку **I ran into an issue** (У меня есть проблема) выше.

### <a name="update-the-app"></a>Обновление приложения

Вы можете развернуть изменения в это приложение. Для этого внесите изменения в VS Code, сохраните файлы, а затем повторите тот же процесс, но выберите существующее приложение вместо создания нового.

## <a name="viewing-logs"></a>Просмотр журналов

Выходные данные журнала (вызовы `console.log`) из приложения непосредственно в окне вывода VS Code.

1. В обозревателе **СЛУЖБЫ ПРИЛОЖЕНИЙ AZURE** щелкните узел приложения правой кнопкой мыши и выберите **Start Streaming Logs** (Начать потоковую передачу журналов).

    ![Запуск потоковой передачи журналов](media/quickstart-nodejs/view-logs.png)

1. В запросе подтвердите намерение включить ведение журнала и перезапустить приложение. После перезапуска приложения откроется окно выходных данных VS Code, подключенное к потоку журнала. 

    :::image type="content" source="media/quickstart-nodejs/enable-restart.png" alt-text="Снимок экрана с запросом Visual Studio Code на включение ведения журнала и перезапуск приложения с выбранной кнопкой &quot;Да&quot;.":::

1. Через несколько секунд в окне вывода появится сообщение о том, что вы подключены к службе потоковой передачи журналов. Обновив страницу в браузере, можно создать дополнительные действия вывода.

    <pre>
    Connecting to log stream...
    2020-03-04T19:29:44  Welcome, you are now connected to log-streaming service. The default timeout is 2 hours.
    Change the timeout with the App Setting SCM_LOGSTREAM_TIMEOUT (in seconds).
    </pre>

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=tailing-logs)

## <a name="next-steps"></a>Дальнейшие действия

Поздравляем, вы успешно завершили работу с этим руководством!

> [!div class="nextstepaction"]
> [Руководство. Приложение Node.js с MongoDB](tutorial-nodejs-mongodb-app.md)

> [!div class="nextstepaction"]
> [Настройка приложения Node.js](configure-language-nodejs.md)

Ознакомьтесь с другими расширениями Azure.

* [База данных Cosmos](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Функции Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Инструменты Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Средства интерфейса командной строки Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Средства Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

Вы можете установить их все сразу в составе [пакета расширений для узла Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).
::: zone-end

::: zone pivot="platform-linux"  
## <a name="prerequisites"></a>Предварительные требования

Если у вас нет учетной записи Azure [, зарегистрируйтесь сегодня](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension) и получите бесплатную учетную запись с кредитами Azure на сумму $200, которые позволят проверить любое сочетание служб.

Вам нужно установить [Visual Studio Code](https://code.visualstudio.com/), а также [Node.js и npm](https://nodejs.org/en/download), диспетчер пакетов для Node.js.

Кроме того, потребуется [расширение Службы приложений Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice), которое позволяет создавать, администрировать и развертывать веб-приложения Linux в Azure в режиме PaaS (платформа как услуга).

### <a name="sign-in"></a>Вход

После установки расширения войдите в учетную запись Azure. На панели действий выберите эмблему Azure, чтобы отобразить обозреватель **Служба приложений Azure**. Выберите **Войти в Azure...** и следуйте инструкциям.

![Вход в Azure](./media/quickstart-nodejs/sign-in.png)

### <a name="troubleshooting"></a>Устранение неполадок

Если отображается сообщение об ошибке **"Не удается найти подписку с именем [идентификатор подписки]"** , возможно вы находитесь за прокси-сервером и не можете связаться с API Azure. Укажите в переменных среды `HTTP_PROXY` и `HTTPS_PROXY` параметры прокси-сервера, используя команду терминала `export`.

```sh
export HTTPS_PROXY=https://username:password@proxy:8080
export HTTP_PROXY=http://username:password@proxy:8080
```

Если настройка переменных среды не устранит проблему, свяжитесь с нами, нажав кнопку **I ran into an issue** (У меня есть проблема) ниже.

### <a name="prerequisite-check"></a>Проверка предварительных условий

Прежде чем продолжить, проверьте наличие и правильность настройки всех обязательных компонентов.

В VS Code должна отображаться следующая информация: адрес электронной почты Azure в строке состояния и подписка в обозревателе **Службы приложений Azure**.

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=getting-started)

## <a name="create-your-nodejs-application"></a>Создание приложения Node.js

Теперь создайте приложение Node. js, которое можно развернуть в облаке. В этом кратком руководстве используется генератор приложений для быстрого создания шаблона приложения с помощью терминала.

> [!TIP]
> Если вы уже завершили работу с [учебником по Node.js](https://code.visualstudio.com/docs/nodejs/nodejs-tutorial), можно сразу перейти к разделу о [развертывании в Azure](#deploy-to-azure).

### <a name="scaffold-a-new-application-with-the-express-generator"></a>Подготовка нового приложения с помощью генератора Express

[Express](https://www.expressjs.com) — это популярная платформа для создания и запуска приложений Node. js. Вы можете создать новое приложение Express с помощью средства [Генератор Express](https://expressjs.com/en/starter/generator.html). Генератор Express поставляется как модуль npm, и его можно непосредственно запустить (без установки) с помощью программы командной строки npm `npx`.

```bash
npx express-generator myExpressApp --view pug --git
```

Параметры `--view pug --git` сообщают генератору, что он должен использовать обработчик шаблонов [pug](https://pugjs.org/api/getting-started.html) (ранее известный под именем `jade`) и создать файл с именем `.gitignore`.

Чтобы установить все зависимости приложения, перейдите в новую папку и выполните команду `npm install`.

```bash
cd myExpressApp
npm install
```

### <a name="run-the-application"></a>Выполнение приложения

Теперь убедитесь, что приложение успешно выполняется. Запустите приложение из терминала с помощью команды `npm start`, чтобы начать работу сервера.

```bash
npm start
```

Теперь откройте в браузере страницу `http://localhost:3000`, которая должна выглядеть примерно так:

![Выполнение приложения Express](./media/quickstart-nodejs/express.png)

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=create-app)

## <a name="deploy-to-azure"></a>Развертывание в Azure

В этом разделе вы развернете приложение Node.js с помощью VS Code и расширения Службы приложений Azure. В этом кратком руководстве используется самая простая модель развертывания: приложение в сжатом виде развертывается как веб-приложение Azure на платформе Linux.

### <a name="deploy-using-azure-app-service"></a>Развертывание с помощью Службы приложений Azure

Сначала откройте папку приложения в VS Code.

```bash
code .
```

В обозревателе **Служба приложений Azure** щелкните значок с направленной вверх синей стрелкой, чтобы развернуть приложение в Azure.

:::image type="content" source="./media/quickstart-nodejs/deploy.png" alt-text="Снимок экрана меню Службы приложений Azure в Visual Studio Code с выбранной синей стрелкой развертывания.":::

> [!TIP]
> Также развертывание можно выполнить из **Палитры команд** (Ctrl+Shift+P), введя строку "Deploy to Web App" (развернуть в веб-приложение) и выполнив команду Службы приложений Azure **Azure App Service: Deploy to Web App** (Служба приложений Azure. Развертывание в виде веб-приложения).

1. Выберите каталог `myExpressApp`, который у вас уже открыт.

1. Выберите **Create new Web App** (Создать веб-приложение), которое по умолчанию развертывается в Службе приложений в Linux.

1. Введите глобально уникальное имя веб-приложения и нажмите клавишу ВВОД. В имени приложения допускаются символы 'a-z', '0-9' и '-'.

1. Выберите **версию Node.js**. Мы рекомендуем использовать LTS.

    В канале уведомлений отображаются ресурсы Azure, которые создаются для вашего приложения.

1. Щелкните **Да**, когда появится запрос на обновление конфигурации, чтобы выполнить `npm install` на целевом сервере. Затем приложение развертывается.

    :::image type="content" source="./media/quickstart-nodejs/server-build.png" alt-text="Снимок экрана с запросом на обновление конфигурации на целевом сервере с выбранной кнопкой &quot;Да&quot;.":::

1. При запуске развертывания вам будет предложено обновить рабочую область, чтобы последующие развертывания автоматически нацеливались на то же веб-приложение Службы приложений. Выберите **Да**, чтобы все изменения правильно применялись к приложению.

    :::image type="content" source="./media/quickstart-nodejs/save-configuration.png" alt-text="Снимок экрана с запросом на обновление рабочей области с выбранной кнопкой &quot;Да&quot;.":::

> [!TIP]
> Убедитесь, что приложение прослушивает порт, указанный в переменной среды PORT: `process.env.PORT`.

### <a name="browse-the-app-in-azure"></a>Просмотр приложения в Azure

По завершении развертывания щелкните **Обзор веб-сайта** в диалоговом окне с предложением просмотреть только что развернутое веб-приложение.

### <a name="troubleshooting"></a>Устранение неполадок

Если отображается сообщение об ошибке **Вы не имеете разрешения на просмотр этого каталога пли страницы.** , значит приложение не может нормально запуститься. Перейдите к следующему разделу и проверьте выходные данные журнала, чтобы найти и исправить ошибку. Если устранить ее не удается, свяжитесь с нами, нажав кнопку **I ran into an issue** (У меня есть проблема) ниже. Мы всегда рады помочь.

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=deploy-app)

### <a name="update-the-app"></a>Обновление приложения

Вы можете развернуть изменения в это приложение, повторив тот же процесс и выбрав существующее приложение вместо создания нового.

## <a name="viewing-logs"></a>Просмотр журналов

В этом разделе вы узнаете, как просматривать журналы (или последние записи журнала) работающего приложения Службы приложений. Все вызовы `console.log` в этом приложении отображаются в окне выходных данных в Visual Studio Code.

Найдите приложение в обозревателе **Службы приложений Azure**, щелкните его правой кнопкой мыши и выберите действие **Просмотреть журналы потоковой передачи**.

Откроется окно выходных данных VS Code, подключенное к потоку журнала.

![Просмотр журналов потоковой передачи](./media/quickstart-nodejs/view-logs.png)

:::image type="content" source="./media/quickstart-nodejs/enable-restart.png" alt-text="Снимок экрана запроса Visual Studio Code на включение ведения журнала и перезапуск веб-приложения с выбранной кнопкой &quot;Да&quot;.":::

Через несколько секунд появится сообщение о том, что вы подключены к службе потоковой передачи журналов. Обновите страницу несколько раз, чтобы отобразить больше действий.

<pre>
2019-09-20 20:37:39.574 INFO  - Initiating warmup request to container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node
2019-09-20 20:37:55.011 INFO  - Waiting for response to warmup request for container msdocs-vscode-node_2_00ac292a. Elapsed time = 15.4373071 sec
2019-09-20 20:38:08.233 INFO  - Container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node initialized successfully and is ready to serve requests.
2019-09-20T20:38:21  Startup Request, url: /Default.cshtml, method: GET, type: request, pid: 61,1,7, SCM_SKIP_SSL_VALIDATION: 0, SCM_BIN_PATH: /opt/Kudu/bin, ScmType: None
</pre>

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=tailing-logs)

## <a name="next-steps"></a>Дальнейшие действия

Поздравляем, вы успешно завершили работу с этим руководством!

Теперь ознакомьтесь с другими расширениями Azure.

* [База данных Cosmos](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Функции Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Инструменты Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Средства интерфейса командной строки Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Средства Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

Вы можете установить их все сразу в составе [пакета расширений для узла Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).


::: zone-end
