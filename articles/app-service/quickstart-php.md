---
title: Краткое руководство. Создание веб-приложения PHP
description: Быстрое развертывание первого приложения Hello World на PHP в Службе приложений Azure. Развертывание выполняется с помощью системы Git, которая является одним из многих способов развертывания в Службе приложений.
ms.assetid: 6feac128-c728-4491-8b79-962da9a40788
ms.topic: quickstart
ms.date: 08/01/2020
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: c04e1b33233abebde746fd7fb1ef8d761dba7e4e
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107788211"
---
# <a name="create-a-php-web-app-in-azure-app-service"></a>Создание веб-приложения PHP в Службе приложений Azure

::: zone pivot="platform-windows"  
[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости.  Из этого краткого руководства вы узнаете, как развернуть приложение PHP в Службе приложений Azure на платформе Windows.
::: zone-end  

::: zone pivot="platform-linux"
[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости.  Из этого краткого руководства вы узнаете, как развернуть приложение PHP в службе приложений Azure на платформе Linux.
::: zone-end  

Создайте веб-приложение с помощью [Azure CLI](/cli/azure/get-started-with-azure-cli) в Cloud Shell и разверните пример кода PHP в веб-приложении с помощью Git.

![Пример приложения, выполняющегося в Azure](media/quickstart-php/hello-world-in-browser.png)

Выполните приведенные здесь инструкции с помощью компьютера Mac, Windows или Linux. После установки необходимых компонентов для выполнения этих шагов потребуется около пяти минут.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим кратким руководством сделайте следующее:

* <a href="https://git-scm.com/" target="_blank">установите Git</a>;
* <a href="https://php.net/manual/install.php" target="_blank">установите PHP</a>.

## <a name="download-the-sample-locally"></a>Скачать пример на локальный компьютер

Выполните следующие команды в окне терминала. Так вы клонируете пример приложения на локальный компьютер и перейдете в каталог, содержащий пример кода. 

```bash
git clone https://github.com/Azure-Samples/php-docs-hello-world
cd php-docs-hello-world
```

## <a name="run-the-app-locally"></a>Локальный запуск приложения

Запустите приложение локально, чтобы увидеть, как оно будет выглядеть после развертывания в Azure. Откройте окно терминала и выполните команду `php`, чтобы запустить встроенный веб-сервер PHP.

```bash
php -S localhost:8080
```

Откройте веб-браузер и перейдите к примеру приложения по адресу `http://localhost:8080`.

На странице отобразится сообщение **Hello World!** из примера приложения.

![Пример приложения, выполняющегося локально](media/quickstart-php/localhost-hello-world-in-browser.png)

В окне терминала нажмите клавиши **CTRL+C**, чтобы выйти из веб-сервера.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user.md)]

::: zone pivot="platform-windows"  
[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group.md)]
::: zone-end  

::: zone pivot="platform-linux"
[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-linux.md)]
::: zone-end

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan.md)]

## <a name="create-a-web-app"></a>Создание веб-приложения

В Cloud Shell создайте веб-приложение в рамках плана службы приложений `myAppServicePlan` с помощью команды [`az webapp create`](/cli/azure/webapp#az_webapp_create). 

В следующем примере замените `<app-name>`глобальным уникальным именем приложения (допустимые символы: `a-z`, `0-9` и `-`). Для среды выполнения установлено значение `PHP|7.4`. Список всех поддерживаемых сред выполнения можно получить с помощью команды [`az webapp list-runtimes`](/cli/azure/webapp#az_webapp_list_runtimes). 

```azurecli-interactive
# Bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.4" --deployment-local-git
# PowerShell
az --% webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.4" --deployment-local-git
```

> [!NOTE]
> С помощью символа `(--%)`, появившегося в PowerShell 3.0, можно указать, что входные данные не следует интерпретировать как команды или выражения.
>

Когда веб-приложение будет создано, в Azure CLI отобразится примерно следующее:

<pre>
Local git is configured with url of 'https://&lt;username&gt;@&lt;app-name&gt;.scm.azurewebsites.net/&lt;app-name&gt;.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;app-name&gt;.azurewebsites.net",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

Вы создали пустое веб-приложение с включенным развертыванием Git.

> [!NOTE]
> URL-адрес удаленного репозитория Git отображается в свойстве `deploymentLocalGitUrl` в формате `https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git`. Сохраните этот URL-адрес для дальнейшего использования.
>

Перейдите к только что созданному веб-приложению. Замените _&lt;app-name>_ уникальным именем приложения, созданного на предыдущем шаге.

```bash
http://<app-name>.azurewebsites.net
```

Новое веб-приложение должно выглядеть так:

![Пустая страница веб-приложения](media/quickstart-php/app-service-web-service-created.png)

[!INCLUDE [Push to Azure](../../includes/app-service-web-git-push-to-azure.md)] 

<pre>
Counting objects: 2, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (2/2), 352 bytes | 0 bytes/s, done.
Total 2 (delta 1), reused 0 (delta 0)
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id '25f18051e9'.
remote: Generating deployment script.
remote: Running deployment command...
remote: Handling Basic Web Site deployment.
remote: Kudu sync from: '/home/site/repository' to: '/home/site/wwwroot'
remote: Copying file: '.gitignore'
remote: Copying file: 'LICENSE'
remote: Copying file: 'README.md'
remote: Copying file: 'index.php'
remote: Ignoring: .git
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Deployment successful.
To https://&lt;app-name&gt;.scm.azurewebsites.net/&lt;app-name&gt;.git
   cc39b1e..25f1805  main -> main
</pre>

## <a name="browse-to-the-app"></a>Переход в приложение

Перейдите в развертываемое приложение с помощью веб-браузера.

```
http://<app-name>.azurewebsites.net
```

Пример кода PHP выполняется в веб-приложении службы приложений Azure.

![Пример приложения, выполняющегося в Azure](media/quickstart-php/hello-world-in-browser.png)

**Поздравляем!** Вы развернули свое первое приложение PHP в службе приложений.

## <a name="update-locally-and-redeploy-the-code"></a>Обновление на локальном компьютере и повторное развертывание кода

В локальном текстовом редакторе в приложении PHP откройте файл `index.php` и внесите небольшое изменение в текстовой строке рядом с `echo`:

```php
echo "Hello Azure!";
```

В окне терминала на локальном компьютере зафиксируйте изменения в Git, а затем отправьте изменение кода в Azure.

```bash
git commit -am "updated output"
git push azure main
```

После развертывания вернитесь к окну браузера, открытому на шаге **перехода в приложение**, и обновите страницу.

![Обновленный пример приложения, выполняющегося в Azure](media/quickstart-php/hello-azure-in-browser.png)

## <a name="manage-your-new-azure-app"></a>Управление новым приложением Azure

1. Перейдите на <a href="https://portal.azure.com" target="_blank">портал Azure</a> для управления созданным веб-приложением. Найдите в поиске и выберите **Службы приложений**.

    ![Поиск Служб приложений, портал Azure, создание веб-приложения PHP](media/quickstart-php/navigate-to-app-services-in-the-azure-portal.png)

2. Выберите имя приложения Azure.

    ![Переход к приложению Azure на портале](./media/quickstart-php/php-docs-hello-world-app-service-list.png)

    Отобразится страница **обзора** вашего веб-приложения. Вы можете выполнять базовые задачи управления: **обзор**, **остановку**, **перезагрузку** и **удаление**.

    ![Страница службы приложений на портале Azure](media/quickstart-php/php-docs-hello-world-app-service-detail.png)

    Меню веб-приложение предоставляет различные параметры для настройки приложения. 

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Использование PHP и MySQL](tutorial-php-mysql-app.md)

> [!div class="nextstepaction"]
> [Настройка приложения PHP](configure-language-php.md)
