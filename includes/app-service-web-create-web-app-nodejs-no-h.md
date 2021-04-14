---
title: Включить файл
description: включить файл
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 02/02/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: c00aa8d83c3ef69e4804822b5527c6faa64d4c8d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "102244541"
---
Создайте [веб-приложение](../articles/app-service/overview.md#app-service-on-linux) в плане службы приложений `myAppServicePlan`. 

В Cloud Shell можно использовать команду [`az webapp create`](/cli/azure/webapp). В следующем примере замените `<app-name>`глобальным уникальным именем приложения (допустимые символы: `a-z`, `0-9` и `-`). Для среды выполнения установлено значение `NODE|6.9`. Список всех поддерживаемых сред выполнения можно получить с помощью команды [`az webapp list-runtimes`](/cli/azure/webapp#az-webapp-list-runtimes). 

```azurecli-interactive
# Bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "NODE|6.9" --deployment-local-git
# PowerShell
az --% webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "NODE|6.9" --deployment-local-git
```

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
  "deploymentLocalGitUrl": "https://&lt;username&gt;@&lt;app-name&gt;.scm.azurewebsites.net/&lt;app-name&gt;.git",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

Вы создали пустое веб-приложение с включенным развертыванием Git.

> [!NOTE]
> URL-адрес удаленного репозитория Git отображается в свойстве `deploymentLocalGitUrl` в формате `https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git`. Сохраните этот URL-адрес для дальнейшего использования.
>
