---
title: Краткое руководство. Создание статического веб-приложения HTML
description: Быстрое развертывание первого приложения Hello World на HTML в Службе приложений Azure. Развертывание выполняется с помощью системы Git, которая является одним из многих способов развертывания в Службе приложений.
author: msangapu-msft
ms.assetid: 60495cc5-6963-4bf0-8174-52786d226c26
ms.topic: quickstart
ms.date: 08/23/2019
ms.author: msangapu
ms.custom: mvc, cli-validate, seodec18
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: 603d8e642cd2e88beec6ae34094a2c6c43d179ee
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107768957"
---
# <a name="create-a-static-html-web-app-in-azure"></a>Создание статического веб-приложения HTML в Azure

В этом кратком руководстве объясняется, как развернуть базовый сайт HTML+CSS в <abbr title="Служба на основе HTTP для размещения веб-приложений, REST API и мобильных внутренних приложений.">Служба приложений Azure</abbr>. Действия в этом руководстве выполняются в [Cloud Shell](../cloud-shell/overview.md), но эти же команды можно выполнить локально в [Azure CLI](/cli/azure/install-azure-cli).

## <a name="1-prepare-your-environment"></a>1. Подготовка среды

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

В [Cloud Shell](../cloud-shell/overview.md) создайте каталог quickstart и перейдите в него.

```bash
mkdir quickstart

cd $HOME/quickstart
```

Затем выполните следующую команду, чтобы клонировать репозиторий с примером приложения в локальный каталог quickstart.

```bash
git clone https://github.com/Azure-Samples/html-docs-hello-world.git
```
<hr/>

## <a name="2-create-a-web-app"></a>2. Создание веб-приложения

Перейдите в каталог, в котором содержится пример кода, и выполните команду `az webapp up`. **Замените** `<app-name>` на глобально уникальное имя.

```bash
cd html-docs-hello-world

az webapp up --location westeurope --name <app_name> --html
```

<details>
<summary>Устранение неполадок</summary>
<ul>
<li>Если команда <code>az</code> не распознана, проверьте, установили ли вы Azure CLI согласно сведениям, указанным в разделе <a href="#1-prepare-your-environment">Подготовка среды</a>.</li>
<li>Замените <code>&lt;app-name&gt;</code> именем, уникальным для всех регионов Azure (<em>допустимыми символами являются <code>a-z</code>, <code>0-9</code>и <code>-</code></em>). Рекомендуется использовать сочетание названия компании и идентификатора приложения.</li>
<li>Аргумент <code>--sku F1</code> создает веб-приложение в ценовой категории "Бесплатный". Этот аргумент можно опустить, чтобы использовать более быструю ценовую категорию "Премиум" с почасовой оплатой.</li>
<li>Аргумент <code>--html</code> указывает на то, что все содержимое папок должно рассматриваться как статическое содержимое, а также на то, что необходимо отключить автоматизацию сборки.</li>
<li>При необходимости вы можете использовать аргумент <code>--location &lt;location-name&gt;</code>, где <code>&lt;location-name&gt;</code> является доступным регионом Azure. Список допустимых регионов для учетной записи Azure можно получить, выполнив команду <a href="/cli/azure/appservice#az_appservice_list_locations"><code>az account list-locations</code></a>.</li>
</ul>
</details>

Выполнение этой команды может занять несколько минут. 

<details>
<summary>Что делает <code>az webapp up</code>?</summary>
<p>Команда <code>az webapp up</code> выполняет следующие действия:</p>
<ul>
<li>создание группы ресурсов по умолчанию;</li>
<li>создание плана Службы приложений по умолчанию;</li>
<li><a href="/cli/azure/webapp#az_webapp_create">создание приложения Службы приложений</a> с указанным именем;</li>
<li><a href="/azure/app-service/deploy-zip">развертывание ZIP-файлов</a> для приложения из текущего рабочего каталога.</li>
<li>При выполнении оно предоставляет сообщения о создании ресурсов, ведении журналов и развертывании из ZIP-файлов.</li>
</ul>

По окончании эта команда выводит приблизительно следующие сведения:

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

</details>

Значение `resourceGroup` понадобится позже для [очистки ресурсов](#6-clean-up-resources).

<hr/>

## <a name="3-browse-to-the-app"></a>3. Переход в приложение

В браузере перейдите по URL-адресу приложения: `http://<app_name>.azurewebsites.net`.

Страница выполняется как веб-приложение службы приложений Azure.

![Домашняя страница примера приложения](media/quickstart-html/hello-world-in-browser-az.png)

<hr/>

## <a name="4-update-and-redeploy-the-app"></a>4. Обновление и повторное развертывание приложения

В Cloud Shell **введите** `nano index.html`, чтобы открыть текстовый редактор Nano. 

В теге заголовка `<h1>` измените запись Azure App Service - Sample Static HTML Site (Служба приложений Azure — пример статического HTML-сайта) на Azure App Service (Служба приложений Azure).

![Файл index.html в редакторе Nano](media/quickstart-html/nano-index-html.png)

**Сохраните** изменения с помощью команды `^O`.

**Выйдите** из редактора Nano, выполнив команду `^X`.

Повторно разверните приложение с помощью команды `az webapp up`.

```bash
az webapp up --html
```

Перейдите в окно браузера, открытое на шаге **перехода в приложение**.

**Обновите** страницу.

![Обновленная домашняя страница примера приложения](media/quickstart-html/hello-azure-in-browser-az.png)

<hr/>

## <a name="5-manage-your-new-azure-app"></a>5. Управление новым приложением Azure

**Перейдите** на [портал Azure](https://portal.azure.com). 

**Найдите** и **выберите** **Службы приложений**.

![Выбор службы приложений на портале Azure](./media/quickstart-html/portal0.png)

**Выберите** имя приложения Azure.

![Переход к приложению Azure на портале](./media/quickstart-html/portal1.png)

Отобразится страница обзора вашего веб-приложения. Вы можете выполнять базовые задачи управления: обзор, завершение, запуск, перезагрузку и удаление.

![Колонка службы приложений на портале Azure](./media/quickstart-html/portal2.png)

В меню слева доступно несколько страниц для настройки приложения.

<hr/>

## <a name="6-clean-up-resources"></a>6. Очистка ресурсов

На предыдущем шаге вы создали ресурсы Azure в группе ресурсов. Если эти ресурсы вам не понадобятся в будущем, вы можете удалить группу ресурсов, выполнив приведенную ниже команду в Cloud Shell. Помните, что имя группы ресурсов автоматически создано на этапе [создания веб-приложения](#2-create-a-web-app).

```bash
az group delete --name appsvc_rg_Windows_westeurope
```

Ее выполнение может занять до минуты.

<hr/>

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Сопоставление пользовательского домена](app-service-web-tutorial-custom-domain-uiex.md)
