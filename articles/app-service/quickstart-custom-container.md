---
title: Краткое руководство. Запуск пользовательского контейнера в Службе приложений
description: Начните работу с контейнерами в Службе приложений Azure, развернув первый пользовательский контейнер.
author: msangapu-msft
ms.author: msangapu
ms.date: 10/21/2019
ms.topic: quickstart
ms.custom: devx-track-csharp
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: 360da015f012822593dbb6390cb7df0017ba85b1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96745083"
---
# <a name="run-a-custom-container-in-azure"></a>Запуск пользовательского контейнера в Azure

::: zone pivot="container-windows"
[Служба приложений Azure](overview.md) предоставляет предопределенные стеки приложений на платформе Windows, например ASP.NET или Node.js, выполняющиеся в IIS. Предварительно настроенный контейнер Windows блокирует в операционной системе возможность административного доступа, установки программного обеспечения, изменений в глобальном кэше сборок и т. д. Дополнительные сведения см. в статье [Функциональные возможности операционной системы для службы приложений Azure](operating-system-functionality.md). Если приложению требуется более высокий уровень доступа, чем предусмотрено в предварительно настроенной среде, можно развернуть пользовательский контейнер Windows.

В этом кратком руководстве показано, как развернуть приложение ASP.NET в образе Windows на сайте [Docker Hub](https://hub.docker.com/) из Visual Studio. Приложение можно запустить в пользовательском контейнере в Службе приложений Azure.

> [!NOTE]
> Контейнеры Windows доступны только для Файлов Azure и в настоящее время не поддерживают BLOB-объекты Azure.


## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством сделайте следующее:

- <a href="https://hub.docker.com/" target="_blank">зарегистрируйте учетную запись центра Docker</a>.
- <a href="https://docs.docker.com/docker-for-windows/install/" target="_blank">Установите Docker для ОС Windows</a>.
- <a href="/virtualization/windowscontainers/quick-start/quick-start-windows-10" target="_blank">Переключите Docker для запуска контейнеров Windows</a>.
- <a href="https://www.visualstudio.com/downloads/" target="_blank">Установите Visual Studio 2019</a>, а также следующие рабочие нагрузки: **ASP.NET и веб-разработка** и **разработка Azure**. Если у вас уже установлена версия Visual Studio 2019, сделайте следующее.

    - Установите последние обновления для Visual Studio, выбрав **Справка** > **Проверить обновления**.
    - Добавьте рабочие нагрузки в Visual Studio, выбрав **Инструменты** > **Получить средства и компоненты**.

## <a name="create-an-aspnet-web-app"></a>Создание веб-приложения ASP.NET

Создайте веб-приложение ASP.NET, сделав следующее:

1. Откройте Visual Studio и выберите **Создать проект**.

1. В окне **Создание нового проекта** найдите и выберите **Веб-приложение ASP.NET (.NET Framework)** для C#, а затем нажмите кнопку **Далее**.

1. В окне **Настроить новый проект** присвойте приложению имя _myfirstazurewebapp_ и щелкните **Создать**.

   ![Настройка проекта веб-приложения](./media/quickstart-custom-container/configure-web-app-project-container.png)

1. Вы можете развернуть любой тип веб-приложения ASP.NET в Azure. В рамках этого краткого руководства выберите шаблон **MVC**.

1. Выберите **Поддержка Docker** и обязательно выберите **Без проверки подлинности** в параметрах проверки подлинности. Нажмите кнопку **создания**.

   ![Создание веб-приложения ASP.NET](./media/quickstart-custom-container/select-mvc-template-for-container.png)

1. Если файл _Dockerfile_ не открылся автоматически, откройте его в **обозревателе решений**.

1. Вам потребуется [поддерживаемый родительский образ](configure-custom-container.md#supported-parent-images). Измените родительский образ, заменив строку `FROM` приведенным ниже кодом. Затем сохраните файл.

   ```dockerfile
   FROM mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019
   ```

1. В меню Visual Studio выберите **Отладка** > **Запустить без отладки**, чтобы запустить приложение локально.

   ![Локальный запуск приложения](./media/quickstart-custom-container/local-web-app.png)

## <a name="publish-to-docker-hub"></a>Публикация на сайте Docker Hub

1. Щелкните правой кнопкой мыши проект **myfirstazurewebapp** в **обозревателе решений** и выберите **Опубликовать**.

1. Выберите **Служба приложений** и щелкните **Опубликовать**.

1. В разделе **Выберите целевой объект публикации** выберите **Реестр контейнеров** и **Docker Hub**, а затем нажмите кнопку **Опубликовать**.

   ![Публикация с помощью страницы обзора проекта](./media/quickstart-custom-container/publish-to-docker-vs2019.png)

1. Укажите данные для входа учетной записи Docker Hub и нажмите кнопку **Сохранить**.

   Дождитесь завершения развертывания. Теперь на странице **Публикация** отображается имя репозитория для последующего использования.

   ![Снимок экрана: выделено имя репозитория.](./media/quickstart-custom-container/published-docker-repository-vs2019.png)

1. Скопируйте имя этого репозитория для последующего использования.

## <a name="create-a-windows-container-app"></a>Создание приложения-контейнера Windows

1. Войдите на [портал Azure]( https://portal.azure.com).

1. Выберите **Создать ресурс** в верхнем левом углу окна портала Azure.

1. В поле поиска над списком ресурсов Azure Marketplace найдите и выберите **Веб-приложение для контейнеров**, а затем нажмите кнопку **Создать**.

1. В окне **Создание веб-приложения** выберите подписку и **группу ресурсов**. При необходимости можно создать новую группу ресурсов.

1. Укажите имя приложения, например *win-container-demo*, а для параметра **Операционная система** выберите значение **Windows**. По завершении выберите **Next: Docker** (Далее: Docker), чтобы продолжить.

   ![Создание Веб-приложения для контейнеров](media/quickstart-custom-container/create-web-app-continer.png)

1. В поле **Источник образа** выберите **Docker Hub**, а для параметра **Образ и тег** введите имя репозитория, скопированное в разделе [Публикация на сайте Docker Hub](#publish-to-docker-hub).

   ![Настройка Веб-приложения для контейнеров](media/quickstart-custom-container/configure-web-app-continer.png)

    Если у вас есть пользовательский образ для веб-приложения в другом расположении, например [реестре контейнеров Azure](../container-registry/index.yml) или любом другом частном репозитории, его можно настроить здесь.

1. Выберите **Просмотр и создание**, а затем нажмите кнопку **Создать** и подождите, пока Azure создаст необходимые ресурсы.

## <a name="browse-to-the-container-app"></a>Переход к контейнеру приложения

По завершении операции Azure отображается окно уведомления.

![Развертывание выполнено](media/quickstart-custom-container/portal-create-finished.png)

1. Щелкните **Перейти к ресурсу**.

1. В обзоре этого ресурса перейдите по ссылке рядом с **URL-адресом**.

В браузере откроется следующая страница:

![Запуск приложения-контейнера Windows](media/quickstart-custom-container/app-starting.png)

Подождите несколько минут и повторите попытку, пока не отобразится домашняя страница ASP.NET по умолчанию.

![Приложение-контейнер Windows работает](media/quickstart-custom-container/app-running-vs.png)

**Поздравляем!** Вы запустили свой первый пользовательский контейнер Windows в службе приложений Azure.

## <a name="see-container-start-up-logs"></a>Просмотр журналов запуска контейнера

Загрузка контейнера Windows может занять некоторое время. Чтобы просмотреть ход выполнения, перейдите по следующему URL-адресу, заменив *\<app_name>* именем приложения.
```
https://<app_name>.scm.azurewebsites.net/api/logstream
```

Потоковые журналы выглядят следующим образом:

```
2018-07-27T12:03:11  Welcome, you are now connected to log-streaming service.
27/07/2018 12:04:10.978 INFO - Site: win-container-demo - Start container succeeded. Container: facbf6cb214de86e58557a6d073396f640bbe2fdec88f8368695c8d1331fc94b
27/07/2018 12:04:16.767 INFO - Site: win-container-demo - Container start complete
27/07/2018 12:05:05.017 INFO - Site: win-container-demo - Container start complete
27/07/2018 12:05:05.020 INFO - Site: win-container-demo - Container started successfully
```

## <a name="update-locally-and-redeploy"></a>Обновление на локальном компьютере и повторное развертывание

1. В Visual Studio в **обозревателе решений** откройте **Представления** > **Главная страница** > **Index.cshtml**.

1. Найдите тег HTML `<div class="jumbotron">` в верхней области и замените его следующим кодом:

   ```html
   <div class="jumbotron">
       <h1>ASP.NET in Azure!</h1>
       <p class="lead">This is a simple app that we've built that demonstrates how to deploy a .NET app to Azure App Service.</p>
   </div>
   ```

1. Чтобы выполнить повторное развертывание в Azure, щелкните правой кнопкой мыши проект **myfirstazurewebapp** в **обозревателе решений** и выберите **Опубликовать**.

1. На странице публикации выберите **Опубликовать** и дождитесь завершения публикации.

1. Чтобы указать службе приложений извлечь новый образ из Docker Hub, перезапустите приложение. На странице приложения на портале щелкните **Перезапустить** > **Да**.

   ![Перезапуск веб-приложения в Azure](./media/quickstart-custom-container/portal-restart-app.png)

Еще раз [перейдите к контейнерному приложению](#browse-to-the-container-app). После обновления веб-страницы сначала должна появиться страница "Starting up" (Запуск) приложения, а через несколько минут должна отобразиться обновленная веб-страница.

![Обновленное веб-приложение в Azure](./media/quickstart-custom-container/azure-web-app-updated.png)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Перенос в контейнер Windows в Azure](tutorial-custom-container.md)

Также ознакомьтесь с другими ресурсами:

> [!div class="nextstepaction"]
> [Настройка пользовательского контейнера](configure-custom-container.md)

::: zone-end  

::: zone pivot="container-linux"
Служба приложений в Linux предоставляет предопределенные стеки приложений на платформе Linux с поддержкой определенных языков (например, .NET, PHP, Node.js и т. д.). Вы также можете использовать пользовательский образ Docker для запуска веб-приложения в стеке приложений, который еще не определен в Azure. В этом кратком руководстве показано, как развернуть образ из [Реестра контейнеров Azure](../container-registry/index.yml) (ACR) в Службе приложений.

## <a name="prerequisites"></a>Предварительные требования

* [учетная запись Azure](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-docker-extension&mktingSource=vscode-tutorial-docker-extension);
* [Docker](https://www.docker.com/community-edition)
* [Visual Studio Code](https://code.visualstudio.com/)
* [Расширение Службы приложений Azure для VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice). Это расширение можно использовать для создания и развертывания веб-приложений Linux на платформе Azure в рамках модели "платформа как служба" (PaaS), а также для управления ими.
* [Расширение Docker для VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker). С помощью этого расширения можно упростить управление локальными образами и командами Docker, а также развернуть созданные образы приложений в Azure.

## <a name="create-an-image"></a>Создание образа

Для выполнения заданий этого краткого руководства вам потребуется подходящий образ веб-приложения, хранящийся в [Реестре контейнеров Azure](../container-registry/index.yml). Следуйте инструкциям в статье [Краткое руководство. Создание частного реестра контейнеров с помощью портала Azure](../container-registry/container-registry-get-started-portal.md), но вместо образа `hello-world` используйте `mcr.microsoft.com/azuredocs/go`. Для справки см. [пример Dockerfile из репозитория примеров Azure](https://github.com/Azure-Samples/go-docs-hello-world).

> [!IMPORTANT]
> Не забудьте установить для параметра **Администратор** значение **Включить** при создании реестра контейнеров. Его также можно установить из раздела **Ключи доступа** на странице реестра на портале Azure. Этот параметр необходим для доступа к Службе приложений.

## <a name="sign-in"></a>Войти

Затем запустите VS Code и войдите в учетную запись Azure с помощью расширения Службы приложений. Для этого выберите логотип Azure на панели действий, перейдите в обозреватель **Службы приложений**, а затем выберите **Войти в Azure** и следуйте инструкциям.

![Вход в Azure](./media/quickstart-docker/sign-in.png)

## <a name="check-prerequisites"></a>Проверка предварительных требований

Теперь можно проверить, правильно ли установлены и настроены все необходимые компоненты.

В VS Code должна отображаться следующая информация: адрес электронной почты Azure в строке состояния и подписка в обозревателе **Службы приложений**.

Затем убедитесь, что у вас установлен и работает Docker. Следующая команда отобразит версию Docker, если она выполняется.

```bash
docker --version
```

Наконец, убедитесь, что Реестр контейнеров Azure подключен. Для этого выберите логотип Docker на панели действий, а затем перейдите к разделу **Реестры**.

![Снимок экрана: раздел "Реестры" с развернутым пунктом Azure и файлом с расширением "io"](./media/quickstart-docker/registries.png)

## <a name="deploy-the-image-to-azure-app-service"></a>Развертывание образа в Службе приложений Azure

Теперь, когда все настроено, вы можете развернуть образ в [Службе приложений Azure](https://azure.microsoft.com/services/app-service/) непосредственно из обозревателя расширений Docker.

Найдите образ в узле **Реестры** в обозревателе **DOCKER** и разверните его, чтобы отобразить его теги. Щелкните тег правой кнопкой мыши и выберите пункт **Deploy Image to Azure App Service** (Развернуть образ в Службе приложений Azure).

Следуйте инструкциям на экране, чтобы выбрать подписку, глобально уникальное имя приложения, группу ресурсов и план Службы приложений. Выберите значение **B1 Basic** (B1 Базовый) для ценовой категории и регион.

После развертывания приложение будет доступно по адресу `http://<app name>.azurewebsites.net`.

**Группа ресурсов** — это именованная коллекция всех ресурсов вашего приложения в Azure. Например, группа ресурсов может содержать ссылку на веб-сайт, базу данных и функцию Azure.

**План Службы приложений** определяет физические ресурсы, которые будут использоваться для размещения веб-сайта. В этом кратком руководстве используется план размещения **Базовый** в инфраструктуре **Linux**. Это означает, что сайт будет размещаться на компьютере Linux вместе с другими веб-сайтами. Если начать работу с планом **Базовый**, можно использовать портал Azure для увеличения масштаба, чтобы ваш сайт был единственным на компьютере.

## <a name="browse-the-website"></a>Обзор веб-сайта

Панель **Выходные данные** откроется во время развертывания, чтобы указать состояние операции. После завершения операции найдите приложение, созданное в обозревателе **Службы приложений**, щелкните его правой кнопкой мыши, а затем выберите **Обзор веб-сайта**, чтобы открыть сайт в браузере.

> [!div class="nextstepaction"]
> [У меня есть проблема](https://www.research.net/r/PWZWZ52?tutorial=quickstart-docker&step=deploy-app)

## <a name="next-steps"></a>Дальнейшие действия

Поздравляем, вы успешно завершили работу с этим руководством!

Теперь ознакомьтесь с другими расширениями Azure.

* [База данных Cosmos](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Функции Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Средства интерфейса командной строки Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Средства Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

Или получите их, установив пакет расширений [инструментов Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).

Также ознакомьтесь с другими ресурсами:

> [!div class="nextstepaction"]
> [Настройка пользовательского контейнера](configure-custom-container.md)

::: zone-end
