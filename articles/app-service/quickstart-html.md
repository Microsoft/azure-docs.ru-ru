---
title: Краткое руководство. Создание статического веб-приложения HTML
description: Быстрое развертывание первого приложения Hello World на HTML в Службе приложений Azure. Развертывание выполняется с помощью системы Git, которая является одним из многих способов развертывания в Службе приложений.
author: msangapu-msft
ms.assetid: 60495cc5-6963-4bf0-8174-52786d226c26
ms.topic: quickstart
ms.date: 08/23/2019
ms.author: msangapu
ms.custom: mvc, cli-validate, seodec18
adobe-target: true
adobe-target-activity: DocsExp–386541–A/B–Enhanced-Readability-Quickstarts–2.19.2021
adobe-target-experience: Experience B
adobe-target-content: ./quickstart-html-uiex
ms.openlocfilehash: 0a33889277c9f6cc60f5e81aff520e27828a1d06
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379627"
---
# <a name="create-a-static-html-web-app-in-azure"></a>Создание статического веб-приложения HTML в Azure

[Служба приложений Azure](overview.md) — это служба веб-размещения с самостоятельной установкой исправлений и высоким уровнем масштабируемости. В этом кратком руководстве объясняется, как развернуть простейший сайт HTML+CSS в Служба приложений Azure. Действия в этом руководстве выполняются в [Cloud Shell](../cloud-shell/overview.md), но эти же команды можно выполнить локально в [Azure CLI](/cli/azure/install-azure-cli).

![Домашняя страница в примере приложения](media/quickstart-html/hello-world-in-browser-az.png)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="download-the-sample"></a>Скачивание примера приложения

В Cloud Shell создайте каталог quickstart и перейдите в него.

```bash
mkdir quickstart

cd $HOME/quickstart
```

Затем выполните следующую команду, чтобы клонировать репозиторий с примером приложения в локальный каталог quickstart.

```bash
git clone https://github.com/Azure-Samples/html-docs-hello-world.git
```

## <a name="create-a-web-app"></a>Создание веб-приложения

Перейдите в каталог, в котором содержится пример кода, и выполните команду `az webapp up`. В следующем примере замените <app_name> уникальным именем приложения. Статическое содержимое обозначается флагом `--html`.

```bash
cd html-docs-hello-world

az webapp up --location westeurope --name <app_name> --html
```

Команда `az webapp up` выполняет следующие действия:

- создание группы ресурсов по умолчанию;

- создание плана службы приложений по умолчанию;

- создание приложения с указанным именем.

- [развертывание ZIP-файлов](./deploy-zip.md) для веб-приложения из текущего рабочего каталога.

Выполнение этой команды может занять несколько минут. При выполнении эта команда выводит приблизительно следующие сведения:

```output
{
  "app_url": "https://&lt;app_name&gt;.azurewebsites.net",
  "location": "westeurope",
  "name": "&lt;app_name&gt;",
  "os": "Windows",
  "resourcegroup": "appsvc_rg_Windows_westeurope",
  "serverfarm": "appsvc_asp_Windows_westeurope",
  "sku": "FREE",
  "src_path": "/home/&lt;username&gt;/quickstart/html-docs-hello-world ",
  &lt; JSON data removed for brevity. &gt;
}
```

Запишите значение `resourceGroup`. Оно потребуется при выполнении задач в разделе об [очистке ресурсов](#clean-up-resources).

## <a name="browse-to-the-app"></a>Переход в приложение

В браузере перейдите по URL-адресу приложения: `http://<app_name>.azurewebsites.net`.

Страница выполняется как веб-приложение службы приложений Azure.

![Домашняя страница примера приложения](media/quickstart-html/hello-world-in-browser-az.png)

**Поздравляем!** Вы развернули свое первое HTML-приложение в службе приложений.

## <a name="update-and-redeploy-the-app"></a>Обновление и повторное развертывание приложения

В Cloud Shell введите `nano index.html`, чтобы открыть текстовый редактор Nano. В теге заголовка `<h1>` измените запись "Azure App Service - Sample Static HTML Site" (Служба приложений Azure — пример статического HTML-сайта) на "Azure App Service" (Служба приложений Azure), как показано ниже.

![Файл index.html в редакторе Nano](media/quickstart-html/nano-index-html.png)

Сохраните изменения и выйдите из редактора Nano. Выполните команду `^O`, чтобы сохранить файл, и `^X` — чтобы выйти.

Теперь повторно разверните приложение с помощью той же команды `az webapp up`.

```bash
az webapp up --location westeurope --name <app_name> --html
```

После завершения развертывания перейдите в окно браузера, открытое на шаге **перехода в приложение**, и обновите страницу.

![Обновленная домашняя страница примера приложения](media/quickstart-html/hello-azure-in-browser-az.png)

## <a name="manage-your-new-azure-app"></a>Управление новым приложением Azure

Чтобы управлять созданным веб-приложением, на [портале Azure](https://portal.azure.com) найдите в поиске и выберите **Службы приложений**. 

![Выбор службы приложений на портале Azure](./media/quickstart-html/portal0.png)

На странице **Службы приложений** выберите имя приложения Azure.

![Переход к приложению Azure на портале](./media/quickstart-html/portal1.png)

Отобразится страница обзора вашего веб-приложения. Вы можете выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление.

![Колонка службы приложений на портале Azure](./media/quickstart-html/portal2.png)

В меню слева доступно несколько страниц для настройки приложения.

## <a name="clean-up-resources"></a>Очистка ресурсов

На предыдущем шаге вы создали ресурсы Azure в группе ресурсов. Если эти ресурсы вам не понадобятся в будущем, вы можете удалить группу ресурсов, выполнив приведенную ниже команду в Cloud Shell. Помните, что имя группы ресурсов автоматически создано на этапе [создания веб-приложения](#create-a-web-app).

```bash
az group delete --name appsvc_rg_Windows_westeurope
```

Ее выполнение может занять до минуты.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сопоставление пользовательского домена](app-service-web-tutorial-custom-domain.md)
