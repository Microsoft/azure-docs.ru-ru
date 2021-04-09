---
title: Краткое руководство. Создание веб-приложения Node.js
description: Быстрое развертывание первого приложения Hello World на Node.js в Службе приложений Azure.
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 08/01/2020
ms.custom: mvc, devcenter, seodec18
zone_pivot_groups: app-service-platform-windows-linux
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 7cd9add699d9bd4a3eff9cdcf1e65af0d754b70a
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "101747792"
---
# <a name="create-a-nodejs-web-app-in-azure"></a>Создание веб-приложений Node.js в Azure

Начало работы с <abbr title="Служба на основе HTTP для размещения веб-приложений, REST API и мобильных внутренних приложений.">Служба приложений Azure</abbr> путем локального создания приложения Node.js/Express с помощью Visual Studio Code и его последующего развертывания в облаке Azure. Поскольку используется <abbr title="в Службе приложений Azure, базовый уровень, в котором на одной виртуальной машине выполняются все приложения, в том числе приложения других клиентов. Этот уровень предназначен только для разработки и тестирования.">уровень "Бесплатный"</abbr>, для прохождения этого краткого руководства не требуются какие-либо затраты.

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

- Учетная запись Azure с активной <abbr title="Подписка Azure — это логический контейнер, используемый для подготовки ресурсов в Azure. Она содержит сведения обо всех ресурсах, таких как виртуальные машины, базы данных и т. д.">Подписка</abbr>. [Создайте учетную запись](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension) бесплатно.
- <a href="https://git-scm.com/" target="_blank">установите Git</a>;
- [Node.js и NPM](https://nodejs.org). Выполнив команду `node --version`, убедитесь, что платформа Node.js установлена.
- [Visual Studio Code](https://code.visualstudio.com/).
- [Расширение Службы приложений Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) для Visual Studio Code.

[Сообщите о проблеме](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&prepare-your-environment)




<br>
<hr/>

## <a name="2-clone-and-run-a-local-nodejs-application"></a>2. Клонирование и запуск локального приложения Node.js

1. На локальном компьютере откройте терминал и выполните клонирование примера репозитория:

    ```bash
    git clone https://github.com/Azure-Samples/nodejs-docs-hello-world
    ```

1. Перейдите в папку нового приложения:

    ```bash
    cd nodejs-docs-hello-world
    ```

1. Установка зависимостей:

    ```bash
    npm install
    ```

1. Запустите приложение, чтобы проверить его локально:

    ```bash
    npm start
    ```
    
1. Откройте веб-браузер и перейдите по адресу `http://localhost:1337`. В браузере должно отобразиться сообщение "Hello World!".

1. Нажмите <kbd>Ctrl</kbd> + <kbd>C</kbd> в терминале, чтобы остановить работу сервера.

[Сообщите о проблеме](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&prepare-your-environment)


<br>
<hr/>




<!-- VS Code extension works differently for Windows/Linus - Step 3 -->

::: zone pivot="platform-windows"  

[!INCLUDE [Windows](./includes/quickstart-nodejs-uiex-windows.md)]


::: zone-end

::: zone pivot="platform-linux"  

[!INCLUDE [Windows](./includes/quickstart-nodejs-uiex-linux.md)]

::: zone-end


## <a name="4-viewing-logs-from-visual-studio-code"></a>4. Просмотр журналов из Visual Studio Code

Просмотрите <abbr title="Все вызовы `console.log` в этом приложении отображаются в окне выходных данных в Visual Studio Code.">log</abbr> запущенного приложения Службы приложений.

1. Найдите приложение в обозревателе **Служба приложений Azure**, щелкните имя приложения правой кнопкой мыши и выберите **Начать потоковую передачу журналов**.

1. Откроется окно вывода Visual Studio Code.

    ![Просмотр журналов потоковой передачи](./media/quickstart-nodejs/view-logs.png)

    :::image type="content" source="./media/quickstart-nodejs/enable-restart.png" alt-text="Снимок экрана запроса Visual Studio Code на включение ведения журнала и перезапуск веб-приложения с выбранной кнопкой &quot;Да&quot;.":::

1. Через несколько секунд появится сообщение о том, что вы подключены к службе потоковой передачи журналов. 
1. Обновите страницу несколько раз, чтобы отобразить больше действий.

    <pre class="is-monospace is-size-small has-padding-medium has-background-tertiary has-text-tertiary-invert">
    2020-09-20 20:37:39.574 INFO  - Initiating warmup request to container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node
    2020-09-20 20:37:55.011 INFO  - Waiting for response to warmup request for container msdocs-vscode-node_2_00ac292a. Elapsed time = 15.4373071 sec
    2020-09-20 20:38:08.233 INFO  - Container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node initialized successfully and is ready to serve requests.
    2020-09-20T20:38:21  Startup Request, url: /Default.cshtml, method: GET, type: request, pid: 61,1,7, SCM_SKIP_SSL_VALIDATION: 0, SCM_BIN_PATH: /opt/Kudu/bin, ScmType: None
    </pre>

<br>

[Сообщите о проблеме](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&prepare-your-environment)

<br>
<hr/>

## <a name="5-clean-up-resources"></a>5. Очистка ресурсов

Найдите приложение в обозревателе **Служба приложений Azure**, щелкните имя приложения правой кнопкой мыши и выберите **Удалить**. 

:::image type="content" source="./media/quickstart-nodejs/delete-resource-ieux.png" alt-text="приложение в обозревателе &quot;Служба приложений Azure&quot;, щелкните имя приложения правой кнопкой мыши и выберите &quot;Удалить&quot;":::

## <a name="next-steps"></a>Дальнейшие действия

Поздравляем, вы успешно завершили работу с этим руководством! Вы можете развернуть изменения в это приложение, повторив тот же процесс и выбрав существующее приложение вместо создания нового.

Теперь ознакомьтесь с другими расширениями Azure.

* [База данных Cosmos](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Функции Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Инструменты Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Средства интерфейса командной строки Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Средства Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

Вы можете установить их все сразу в составе [пакета расширений для узла Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).