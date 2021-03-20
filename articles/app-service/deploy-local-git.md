---
title: развертывание из локального репозитория Git
description: Узнайте, как включить локальное развертывание репозитория Git в службе приложений Azure. Один из самых простых способов развертывания кода с локального компьютера.
ms.assetid: ac50a623-c4b8-4dfd-96b2-a09420770063
ms.topic: article
ms.date: 02/16/2021
ms.reviewer: dariac
ms.custom: seodec18, devx-track-azurecli
ms.openlocfilehash: 5dd6183bf88c167adb2f084c319cd90b94351dfb
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100560491"
---
# <a name="local-git-deployment-to-azure-app-service"></a>Развертывание локального репозитория Git в Службе приложений Azure

В этом пошаговом руководство показано, как развернуть приложение в [службе приложений Azure](overview.md) из репозитория Git на локальном компьютере.

## <a name="prerequisites"></a>Предварительные условия

Выполните следующие шаги для изучения данного руководства.

- [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]
  
- [Установка Git](https://www.git-scm.com/downloads).

- Создать локальный репозиторий Git с кодом, который вы хотите развернуть. Чтобы скачать пример репозитория, выполните следующую команду в локальном окне терминала:
  
  ```bash
  git clone https://github.com/Azure-Samples/nodejs-docs-hello-world.git
  ```

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="configure-a-deployment-user"></a>Настойка пользователя развертывания

См. раздел [Настройка учетных данных развертывания для службы приложений Azure](deploy-configure-credentials.md). Можно использовать либо учетные данные области пользователя, либо учетные данные области приложения.

## <a name="create-a-git-enabled-app"></a>Создание приложения с поддержкой Git

Если у вас уже есть приложение службы приложений и вы хотите настроить для него локальное развертывание Git, см. раздел [Настройка существующего приложения](#configure-an-existing-app) .

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Запустите [`az webapp create`](/cli/azure/webapp#az_webapp_create) с `--deployment-local-git` параметром. Пример:

```azurecli-interactive
az webapp create --resource-group <group-name> --plan <plan-name> --name <app-name> --runtime "<runtime-flag>" --deployment-local-git
```

Выходные данные содержат URL-адрес следующего вида: `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git` . Используйте этот URL-адрес для развертывания приложения на следующем шаге.

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Выполните команду [New-азвебапп](/powershell/module/az.websites/new-azwebapp) из корня репозитория Git. Пример:

```azurepowershell-interactive
New-AzWebApp -Name <app-name>
```

При запуске этого командлета из каталога, который является репозиторием Git, он автоматически создает удаленный репозиторий Git для вашего приложения службы приложений с именем `azure` .

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

На портале сначала необходимо создать приложение, а затем настроить его развертывание. См. раздел [Настройка существующего приложения](#configure-an-existing-app).

-----

## <a name="configure-an-existing-app"></a>Настройка существующего приложения

Если вы еще не создали приложение, см. статью [Создание приложения с поддержкой Git](#create-a-git-enabled-app) .

# <a name="azure-cli"></a>[Azure CLI](#tab/cli)

Выполните [`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/source#az-webapp-deployment-source-config-local-git) . Пример:

```azurecli-interactive
az webapp deployment source config-local-git --name <app-name> --resource-group <group-name>
```

Выходные данные содержат URL-адрес следующего вида: `https://<deployment-username>@<app-name>.scm.azurewebsites.net/<app-name>.git` . Используйте этот URL-адрес для развертывания приложения на следующем шаге.

> [!TIP]
> Этот URL-адрес содержит имя пользователя для развертывания области пользователя. При желании вместо этого можно [использовать учетные данные области приложения](deploy-configure-credentials.md#appscope) . 

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/powershell)

Задайте `scmType` приложение, выполнив командлет [Set-азресаурце](/powershell/module/az.resources/set-azresource) .

```powershell-interactive
$PropertiesObject = @{
    scmType = "LocalGit";
}

Set-AzResource -PropertyObject $PropertiesObject -ResourceGroupName <group-name> `
-ResourceType Microsoft.Web/sites/config -ResourceName <app-name>/web `
-ApiVersion 2015-08-01 -Force
```

# <a name="azure-portal"></a>[Портал Azure](#tab/portal)

1. В [портал Azure](https://portal.azure.com)перейдите на страницу управления приложением.

1. В меню слева выберите параметры **центра развертывания**  >  . Выберите **локальный git** в **источнике**, а затем нажмите кнопку **сохранить**.

    ![Показано, как включить локальное развертывание Git для службы приложений в портал Azure](./media/deploy-local-git/enable-portal.png)

1. В разделе Local Git скопируйте **git clone URI** для последующего использования. Этот URI не содержит никаких учетных данных.

-----

## <a name="deploy-the-web-app"></a>Развертывание веб-приложения

1. В локальном окне терминала измените каталог на корень репозитория Git и добавьте удаленный Git с помощью URL-адреса, полученного из приложения. Если выбранный метод не предоставляет URL-адрес, используйте `https://<app-name>.scm.azurewebsites.net/<app-name>.git` с именем приложения в `<app-name>` .
   
   ```bash
   git remote add azure <url>
   ```

    > [!NOTE]
    > Если вы [создали приложение с поддержкой Git в PowerShell с помощью New-азвебапп](#create-a-git-enabled-app), удаленный объект уже создан.
   
1. Отправка в Azure Remote с помощью `git push azure master` . 
   
1. В окне **диспетчера учетных данных Git** введите [учетные данные области пользователя или приложения](#configure-a-deployment-user), а не учетные данные для входа в Azure.

    Если удаленный URL-адрес Git уже содержит имя пользователя и пароль, вам не будет предложено. 
   
1. Просмотрите выходные данные. `npm install`Для Node.js и для Python можно увидеть автоматизацию среды выполнения, например MSBuild для ASP.NET `pip install` . 
   
1. Перейдите к приложению в портал Azure, чтобы убедиться, что содержимое развернуто.

## <a name="troubleshoot-deployment"></a>Устранение неполадок с развертыванием

При использовании Git для публикации в приложении службы приложений в Azure могут отобразиться следующие распространенные сообщения об ошибках:

|Сообщение|Причина|Решение
---|---|---|
|`Unable to access '[siteURL]': Failed to connect to [scmAddress]`|Приложение не работает.|запуск приложения на портале Azure. Развертывание Git недоступно, если веб-приложение остановлено.|
|`Couldn't resolve host 'hostname'`|Неверные сведения об адресе удаленного ресурса "Azure".|используйте команду `git remote -v`, чтобы вывести список всех удаленных репозиториев с соответствующими URL-адресами. Проверьте правильность URL-адреса удаленного репозитория "azure". При необходимости удалите и повторно создайте этот удаленный репозиторий, используя правильный URL-адрес.|
|`No refs in common and none specified; doing nothing. Perhaps you should specify a branch such as 'main'.`|Вы не указали ветвь во время `git push` или не установили `push.default` значение в `.gitconfig` .|Выполните команду `git push` еще раз, указав главную ветвь: `git push azure main` .|
|`src refspec [branchname] does not match any.`|Предпринята попытка принудительной отправки в удаленную ветвь Azure, отличную от Main.|Выполните команду `git push` еще раз, указав главную ветвь: `git push azure main` .|
|`RPC failed; result=22, HTTP code = 5xx.`|эта ошибка может возникнуть при попытке отправить большой репозиторий Git по протоколу HTTPS.|Измените конфигурацию Git на локальном компьютере, чтобы `postBuffer` увеличить ее размер. Например, `git config --global http.postBuffer 524288000`.|
|`Error - Changes committed to remote repository but your web app not updated.`|Вы развернули приложение Node.js с _package.jsв_ файле, который указывает дополнительные необходимые модули.|Проверьте `npm ERR!` сообщения об ошибках до этой ошибки для получения дополнительных сведений о сбое. Ниже приведены известные причины этой ошибки и соответствующие `npm ERR!` сообщения.<br /><br />**Неправильный формат package.jsфайла**: `npm ERR! Couldn't read dependencies.`<br /><br />**Собственный модуль не имеет двоичного распространения для Windows**:<br />`npm ERR! \cmd "/c" "node-gyp rebuild"\ failed with 1` <br />или диспетчер конфигурации служб <br />`npm ERR! [modulename@version] preinstall: \make || gmake\ `|

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Сервер сборки службы приложений (документация по проекту KUDU)](https://github.com/projectkudu/kudu/wiki)
- [Непрерывное развертывание в Службе приложений Azure](deploy-continuous-deployment.md)
- [Пример. Создание веб-приложения и развертывание кода из локального репозитория Git (Azure CLI)](./scripts/cli-deploy-local-git.md?toc=%2fcli%2fazure%2ftoc.json)
- [Пример. Создание веб-приложения и развертывание кода из локального репозитория Git (PowerShell)](./scripts/powershell-deploy-local-git.md?toc=%2fpowershell%2fmodule%2ftoc.json)
