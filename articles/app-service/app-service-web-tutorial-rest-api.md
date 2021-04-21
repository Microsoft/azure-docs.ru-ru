---
title: Руководство по размещению REST API с CORS
description: Узнайте, как служба приложений Azure помогает размещать API-интерфейсы RESTful с поддержкой CORS. В Службе приложений могут размещаться интерфейсные веб-приложения и API-интерфейсы серверной части.
ms.assetid: a820e400-06af-4852-8627-12b3db4a8e70
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/28/2020
ms.custom: devx-track-csharp, mvc, devcenter, seo-javascript-september2019, seo-javascript-october2019, seodec18
ms.openlocfilehash: 6d2f4d8fc7a2a7011a2417467f7131c4cfb26edc
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107788229"
---
# <a name="tutorial-host-a-restful-api-with-cors-in-azure-app-service"></a>Руководство по Размещение API-интерфейсов RESTful с поддержкой CORS в службе приложений Azure

[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости. Кроме того, служба приложений включает встроенную поддержку [общего доступа к ресурсам независимо от источника (CORS)](https://wikipedia.org/wiki/Cross-Origin_Resource_Sharing) для API-интерфейсов RESTful. В этом руководстве рассматривается развертывание приложения API ASP.NET Core в службу приложений с поддержкой CORS. Вы настроите приложение с помощью программ командной строки и развернете его с помощью Git. 

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создавать ресурсы службы приложений с помощью Azure CLI;
> * развертывать API-интерфейс RESTful в Azure с помощью Git;
> * включать в службе приложений поддержку CORS.

Шаги, описанные в этом руководстве, можно выполнить для macOS, Linux и Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

* <a href="https://git-scm.com/" target="_blank">установите Git</a>;
 * <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">установите пакет SDK для .NET Core 3.1 с последними обновлениями</a>.

## <a name="create-local-aspnet-core-app"></a>Создание локального приложения ASP.NET Core

На этом шаге вы настроите локальный проект ASP.NET Core. Служба приложений поддерживает единый рабочий процесс для API-интерфейсов, написанных на других языках.

### <a name="clone-the-sample-application"></a>Клонирование примера приложения

Откройте окно терминала и c помощью команды `cd` перейдите в рабочий каталог.  

Выполните команду ниже, чтобы клонировать репозиторий с примером. 

```bash
git clone https://github.com/Azure-Samples/dotnet-core-api
```

Этот репозиторий содержит приложение, созданное во время работы с предыдущим руководством: [Использование страниц справки по веб-API ASP.NET Core со Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger?tabs=visual-studio). В этом руководстве генератор Swagger используется для обслуживания [пользовательского интерфейса Swagger](https://swagger.io/swagger-ui/) и конечной точки JSON Swagger.

### <a name="run-the-application"></a>Выполнение приложения

Выполните указанные ниже команды, чтобы установить необходимые пакеты, перенести базы данных и запустить приложение.

```bash
cd dotnet-core-api
dotnet restore
dotnet run
```

Перейдите в расположение `http://localhost:5000/swagger` в браузере, чтобы поэкспериментировать с пользовательским интерфейсом Swagger.

![Выполняющийся локально API ASP.NET Core](./media/app-service-web-tutorial-rest-api/azure-app-service-local-swagger-ui.png)

Перейдите в расположение `http://localhost:5000/api/todo`, чтобы просмотреть список элементов списка задач в формате JSON.

Перейдите в расположение `http://localhost:5000`, чтобы поэкспериментировать с приложением браузера. Позже вы перейдете из приложения браузера к удаленному API в службе приложений, чтобы проверить функции CORS. Код для приложения браузера находится в каталоге _wwwroot_ репозитория.

Чтобы остановить ASP.NET Core в любое время, нажмите клавиши `Ctrl+C` в окне терминала.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-app-to-azure"></a>Развертывание приложения в Azure

На этом шаге вы развернете приложение .NET Core, подключенное к базе данных SQL, в службе приложений.

### <a name="configure-local-git-deployment"></a>Настройка локального развертывания Git

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-a-resource-group"></a>Создание группы ресурсов

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)]

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-no-h.md)]

### <a name="create-a-web-app"></a>Создание веб-приложения

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-dotnetcore-win-no-h.md)] 

### <a name="push-to-azure-from-git"></a>Публикация в Azure из Git

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Enumerating objects: 83, done.
Counting objects: 100% (83/83), done.
Delta compression using up to 8 threads
Compressing objects: 100% (78/78), done.
Writing objects: 100% (83/83), 22.15 KiB | 3.69 MiB/s, done.
Total 83 (delta 26), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '509236e13d'.
remote: Generating deployment script.
remote: Project file path: .\TodoApi.csproj
remote: Generating deployment script for ASP.NET MSBuild16 App
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling ASP.NET Core Web Application deployment with MSBuild16.
remote: .
remote: .
remote: .
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Triggering recycle (preview mode disabled).
remote: Deployment successful.
To https://&lt;app_name&gt;.scm.azurewebsites.net/&lt;app_name&gt;.git
 * [new branch]      master -> master
</pre>

### <a name="browse-to-the-azure-app"></a>Переход к приложению Azure

Перейдите в расположение `http://<app_name>.azurewebsites.net/swagger` в браузере и ознакомьтесь с пользовательским интерфейсом Swagger.

![API ASP.NET Core, выполняющийся в службе приложений Azure](./media/app-service-web-tutorial-rest-api/azure-app-service-browse-app.png)

Перейдите в `http://<app_name>.azurewebsites.net/swagger/v1/swagger.json`, чтобы просмотреть файл _swagger.json_ для развернутого API.

Перейдите в `http://<app_name>.azurewebsites.net/api/todo`, чтобы просмотреть развернутый API в режиме работы.

## <a name="add-cors-functionality"></a>Добавление функциональных возможностей CORS

Далее вы включите встроенную поддержку CORS в службе приложений для API.

### <a name="test-cors-in-sample-app"></a>Тестирование CORS в примере приложения

В локальном репозитории откройте _wwwroot/index.html_.

В строке 51 задайте для переменной `apiEndpoint` URL-адрес развернутого API (`http://<app_name>.azurewebsites.net`). Замените _\<appname>_ именем своего приложения в Службе приложений.

В локальном окне терминала снова запустите пример приложения.

```bash
dotnet run
```

Перейдите в приложение браузера по адресу `http://localhost:5000`. Откройте окно инструментов разработчика в браузере (`Ctrl`+`Shift`+`i` в Chrome для Windows) и просмотрите вкладку **Консоль**. Вы должны увидеть сообщение об ошибке: `No 'Access-Control-Allow-Origin' header is present on the requested resource`.

![Ошибка CORS в клиенте браузера](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-error.png)

Из-за несоответствия домена между приложением браузера (`http://localhost:5000`) и удаленным ресурсом (`http://<app_name>.azurewebsites.net`), а также того факта, что ваш API в службе приложений не отправляет заголовок `Access-Control-Allow-Origin`, браузер ограничил загрузку междоменного содержимого в приложении браузера.

В рабочей среде для приложения браузера будет использоваться общедоступный URL-адрес вместо URL-адреса локального узла, но способ включения CORS для этих адресов одинаков.

### <a name="enable-cors"></a>Включение CORS 

В Cloud Shell включите CORS для URL-адреса своего клиента с помощью команды [`az webapp cors add`](/cli/azure/webapp/cors#az_webapp_cors_add). Замените заполнитель _&lt;app-name>_ .

```azurecli-interactive
az webapp cors add --resource-group myResourceGroup --name <app-name> --allowed-origins 'http://localhost:5000'
```

Можно задать несколько URL-адресов клиента в `properties.cors.allowedOrigins` (`"['URL1','URL2',...]"`). Также можно включить CORS для всех URL-адресов клиента с помощью `"['*']"`.

> [!NOTE]
> Если для вашего приложения требуется отправка учетных данных, например cookie или маркеров проверки подлинности, для браузера может понадобиться заголовок `ACCESS-CONTROL-ALLOW-CREDENTIALS` в ответе. Чтобы активировать этот заголовок в службе приложений, в конфигурации CORS установите значение параметра `properties.cors.supportCredentials` равным `true`. Его нельзя активировать, если параметр `allowedOrigins` содержит `'*'`.

### <a name="test-cors-again"></a>Повторное тестирование CORS

Обновите приложение браузера, перейдя по адресу `http://localhost:5000`. Сообщение об ошибке в окне **консоли** исчезнет, вы сможете просмотреть данные из развернутого API и выполнить с ними операции. Теперь ваш удаленный API поддерживает CORS для приложения браузера, запущенного в локальной среде. 

![Успешное тестирование CORS в клиенте браузера](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-success.png)

Поздравляем! Вы запустили API в службе приложений Azure с поддержкой CORS.

## <a name="app-service-cors-vs-your-cors"></a>Сравнение CORS в службе приложений и вашей среде

Вы можете использовать собственные служебные программы CORS вместо программ в службе приложений для большей гибкости. Например, вы можете указать разные разрешенные источники для разных маршрутов или методов. Так как CORS в службе приложений позволяет указать один набор принятых источников для всех маршрутов и методов API, вы можете использовать собственный код CORS. Подробные сведения об этом см. в статье о [включении запросов независимо от источника (CORS)](/aspnet/core/security/cors).

> [!NOTE]
> Не используйте совместно CORS службы приложений и собственный код CORS. В противном случае ваш код CORS не принесет никаких результатов, так как CORS службы приложений имеет приоритет.
>
>

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Дальнейшие действия

Вы научились выполнять следующие задачи:

> [!div class="checklist"]
> * создавать ресурсы службы приложений с помощью Azure CLI;
> * развертывать API-интерфейс RESTful в Azure с помощью Git;
> * включать в службе приложений поддержку CORS.

Перейдите к следующему руководству, чтобы узнать, как выполнять аутентификацию и авторизацию пользователей.

> [!div class="nextstepaction"]
> [Руководство. Сквозная аутентификация и авторизация пользователей](tutorial-auth-aad.md)
