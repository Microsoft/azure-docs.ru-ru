---
title: Руководство. Использование Azure Key Vault в веб-приложении Azure (.NET)
description: Узнайте, как настроить считывание секрета из хранилища ключей для веб-приложения Azure в приложении ASP.NET Core.
services: key-vault
author: msmbaldwin
manager: rajvijan
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 05/06/2020
ms.author: mbaldwin
ms.custom: devx-track-csharp, devx-track-azurecli
ms.openlocfilehash: 2960726cf687908e8e4aed9333fce490dd7ff006
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98788743"
---
# <a name="tutorial-use-a-managed-identity-to-connect-key-vault-to-an-azure-web-app-in-net"></a>Руководство. Использование управляемого удостоверения для подключения Key Vault к веб-приложению Azure (.NET)

[Azure Key Vault](./overview.md) предоставляет способ хранения учетных данных и других секретов с высоким уровнем безопасности. Но для получения этих секретов в коде нужно проходить проверку подлинности в Key Vault. [Управляемые удостоверения для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md) помогают решить эту задачу, предоставляя службам Azure автоматически управляемое удостоверение в Azure Active Directory (Azure AD). Вы можете использовать это удостоверение для аутентификации в любой службе, которая поддерживает аутентификацию Azure AD, включая Key Vault, не отображая учетные данные в коде.

С помощью этого учебника вы создадите и развернете веб-приложение Azure в [Службе приложений Azure](../../app-service/overview.md). Управляемое удостоверение позволит вам выполнить аутентификацию веб-приложения Azure с помощью хранилища ключей Azure. Кроме того, вы воспользуетесь [клиентской библиотекой секретов Azure Key Vault для .NET](/dotnet/api/overview/azure/key-vault) и [Azure CLI](/cli/azure/get-started-with-azure-cli). Те же базовые принципы применяются и для других языков разработки, а также при использовании Azure PowerShell и портала Azure.

Дополнительные сведения о веб-приложениях Службы приложений Azure и развертывании, представленном в этом учебнике, см. в следующих статьях:
- [Обзор Службы приложений Azure](../../app-service/overview.md)
- [Создание веб-приложения ASP.NET Core в Службе приложений Azure](../../app-service/quickstart-dotnetcore.md)
- [Развертывание локального репозитория Git в Службе приложений Azure](../../app-service/deploy-local-git.md)

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником необходимы указанные ниже компоненты.

* Подписка Azure. [Создайте ее бесплатно.](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Пакет SDK для .NET Core 3.1 или более поздней версии](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* [Git.](https://www.git-scm.com/downloads)
* [Интерфейс командной строки Azure](/cli/azure/install-azure-cli) или [Azure PowerShell](/powershell/azure/).
* [Azure Key Vault.](./overview.md) Хранилище ключей можно создать с помощью [портала Azure](quick-create-portal.md), [Azure CLI](quick-create-cli.md) или [Azure PowerShell](quick-create-powershell.md).
* [Секрет](../secrets/about-secrets.md) Key Vault. Вы можете создать секрет с помощью [портала Azure](../secrets/quick-create-portal.md), [PowerShell](../secrets/quick-create-powershell.md) или [Azure CLI](../secrets/quick-create-cli.md).

Если ваше веб-приложение уже развернуто в Службе приложений Azure, вы можете перейти к разделам [Настройка доступа веб-приложения к хранилищу ключей](#create-and-assign-a-managed-identity) и [Изменение кода веб-приложения](#modify-the-app-to-access-your-key-vault).

## <a name="create-a-net-core-app"></a>Создание приложения .NET Core
На этом шаге вы настроите локальный проект .NET Core.

В окне терминала на локальном компьютере создайте каталог с именем `akvwebapp` и перейдите в него.

```bash
mkdir akvwebapp
cd akvwebapp
```

Теперь создайте приложение .NET Core с помощью команды [dotnet new web](/dotnet/core/tools/dotnet-new).

```bash
dotnet new web
```

Запустите приложение локально и ознакомьтесь с тем, как оно будет выглядеть после развертывания в Azure.

```bash
dotnet run
```

В веб-браузере перейдите к приложению по адресу `http://localhost:5000`.

На странице отобразится сообщение "Hello World!" из примера приложения.

Дополнительные сведения о создании веб-приложений для Azure см. в статье [Создание веб-приложения ASP.NET Core в Службе приложений Azure](../../app-service/quickstart-dotnetcore.md).

## <a name="deploy-the-app-to-azure"></a>Развертывание приложения в Azure

На этом шаге вы развернете приложение .NET Core в Службе приложений с помощью локального репозитория Git. Дополнительные сведения о создании и развертывании приложений см. в статье [Создание веб-приложения ASP.NET Core в Azure](../../app-service/quickstart-dotnetcore.md)

### <a name="configure-the-local-git-deployment"></a>Настройка локального развертывания Git

В окне терминала нажмите клавиши **CTRL+C**, чтобы закрыть веб-сервер.  Инициализируйте репозиторий Git для проекта .NET Core.

```bash
git init
git add .
git commit -m "first commit"
```

Для развертывания веб-приложения Azure можно использовать FTP и локальный репозиторий Git, работая как *пользователь развертывания*. Настроив пользователя развертывания, вы сможете использовать его для всех последующих развертываний в Azure. Имя пользователя и пароль для учетной записи развертывания отличаются от учетных данных подписки Azure. 

Чтобы настроить пользователя для развертывания, выполните команду [az webapp deployment user set](/cli/azure/webapp/deployment/user?#az-webapp-deployment-user-set). Выберите имя пользователя и пароль согласно следующим рекомендациям: 

- Имя пользователя должно быть уникальным в пределах Azure. Для отправки обновлений из локального репозитория Git имя не может содержать символ @. 
- Пароль должен содержать не менее восьми символов и включать два из трех следующих элементов: буквы, цифры и символы. 

```azurecli-interactive
az webapp deployment user set --user-name "<username>" --password "<password>"
```

В выходных данных JSON пароль отображается как `null`. Если появляется сообщение об ошибке `'Conflict'. Details: 409`, измените имя пользователя. Если появляется сообщение об ошибке `'Bad Request'. Details: 400`, используйте более надежный пароль. 

Запишите имя пользователя и пароль, чтобы в будущем использовать их для развертывания веб-приложений.

### <a name="create-a-resource-group"></a>Создание группы ресурсов

Группой ресурсов называется логический контейнер, в котором развертываются и управляются ресурсы Azure. Создайте группу ресурсов для размещения хранилища ключей и веб-приложения, выполнив команду [az group create](/cli/azure/group?#az-group-create).

```azurecli-interactive
az group create --name "myResourceGroup" -l "EastUS"
```

### <a name="create-an-app-service-plan"></a>Создание плана службы приложений

Создайте [план службы приложений](../../app-service/overview-hosting-plans.md), выполнив команду [az appservice plan create](/cli/azure/appservice/plan) в Azure CLI. В следующем примере создается план службы приложений с именем `myAppServicePlan` и ценовой категорией `FREE`.

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku FREE
```

После создания плана службы приложений Azure CLI отображает следующие сведения.

<pre>
{ 
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "West Europe",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "app",
  "location": "West Europe",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
} 
</pre>

См. сведения о том, как [управлять планом службы приложений в Azure](../../app-service/app-service-plan-manage.md).

### <a name="create-a-web-app"></a>Создание веб-приложения

Создайте [веб-приложение Azure](../../app-service/overview.md) в плане службы приложений `myAppServicePlan`. 

> [!Important]
> Как и хранилище ключей, веб-приложение Azure должно иметь уникальное имя. В следующих примерах замените `<your-webapp-name>` реальным именем веб-приложения.


```azurecli-interactive
az webapp create --resource-group "myResourceGroup" --plan "myAppServicePlan" --name "<your-webapp-name>" --deployment-local-git
```

При создании веб-приложения в Azure CLI отображаются следующие выходные данные.

<pre>
Local git is configured with url of 'https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;ayour-webapp-name&gt;.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "clientCertExclusionPaths": null,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;your-webapp-name&gt;.azurewebsites.net",
  "deploymentLocalGitUrl": "https://&lt;username&gt;@&lt;your-webapp-name&gt;.scm.azurewebsites.net/&lt;your-webapp-name&gt;.git",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>


URL-адрес удаленного репозитория Git отображается в свойстве `deploymentLocalGitUrl` и имеет формат `https://<username>@<your-webapp-name>.scm.azurewebsites.net/<your-webapp-name>.git`. Сохраните этот URL-адрес. Он понадобится вам позднее.

Войдите в приложение с помощью следующей команды. Замените `<your-webapp-name>` реальным именем приложения.

```bash
https://<your-webapp-name>.azurewebsites.net
```

Вы увидите веб-страницу по умолчанию для созданного веб-приложения Azure.

### <a name="deploy-your-local-app"></a>Развертывание локального приложения

Вернитесь к окну терминала (в локальном расположении) и добавьте удаленное приложение Azure в локальный репозиторий Git. В следующей команде замените `<deploymentLocalGitUrl-from-create-step>` URL-адресом удаленного репозитория Git (см. раздел [Создание веб-приложения](#create-a-web-app)).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Отправьте код в удаленный репозиторий Azure, чтобы развернуть приложение, с помощью следующей команды. Когда диспетчер учетных данных Git отобразит запрос на ввод учетных данных, введите значения, созданные на шаге [Настройка локального развертывания Git](#configure-the-local-git-deployment).

```bash
git push azure main
```

Выполнение этой команды может занять несколько минут. После завершения отобразятся следующие сведения.
<pre>
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 285 bytes | 95.00 KiB/s, done.
Total 3 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Deploy Async
remote: Updating branch 'main'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'd6b54472f7'.
remote: Repository path is /home/site/repository
remote: Running oryx build...
remote: Build orchestrated by Microsoft Oryx, https://github.com/Microsoft/Oryx
remote: You can report issues at https://github.com/Microsoft/Oryx/issues
remote:
remote: Oryx Version      : 0.2.20200114.13, Commit: 204922f30f8e8d41f5241b8c218425ef89106d1d, ReleaseTagName: 20200114.13
remote: Build Operation ID: |imoMY2y77/s=.40ca2a87_
remote: Repository Commit : d6b54472f7e8e9fd885ffafaa64522e74cf370e1
.
.
.
remote: Deployment successful.
remote: Deployment Logs : 'https://&lt;your-webapp-name&gt;.scm.azurewebsites.net/newui/jsonviewer?view_url=/api/deployments/d6b54472f7e8e9fd885ffafaa64522e74cf370e1/log'
To https://&lt;your-webapp-name&gt;.scm.azurewebsites.net:443/&lt;your-webapp-name&gt;.git
   d87e6ca..d6b5447  main -> main
</pre>

Откройте или обновите в браузере страницу развернутого приложения.

```bash
http://<your-webapp-name>.azurewebsites.net
```

На странице отобразится сообщение "Hello World!" которое вы уже видели на странице `http://localhost:5000`.

Дополнительные сведения о развертывании веб-приложения с помощью Git см. в статье [Локальное развертывание Git в Службе приложений Azure](../../app-service/deploy-local-git.md).
 
## <a name="configure-the-web-app-to-connect-to-key-vault"></a>Настройка подключения к Key Vault в веб-приложении

В этом разделе показано, как настроить веб-доступ к Key Vault и изменить код приложения, чтобы оно получало секрет из Key Vault.

### <a name="create-and-assign-a-managed-identity"></a>Создание и назначение управляемого удостоверения

При работе с этим руководством мы будем использовать [управляемое удостоверение](../../active-directory/managed-identities-azure-resources/overview.md) для проверки подлинности в Key Vault. Управляемое удостоверение автоматически управляет учетными данными приложения.

В Azure CLI выполните команду [az webapp-identity assign](/cli/azure/webapp/identity?#az-webapp-identity-assign), чтобы создать удостоверение для этого приложения.

```azurecli-interactive
az webapp identity assign --name "<your-webapp-name>" --resource-group "myResourceGroup"
```

Эта команда вернет следующий фрагмент JSON.

```json
{
  "principalId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "tenantId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "type": "SystemAssigned"
}
```

Чтобы предоставить веб-приложению разрешение на выполнение операций **get** и **list** в хранилище ключей, передайте значение `principalId` в команду Azure CLI [az keyvault set-policy](/cli/azure/keyvault?#az-keyvault-set-policy).

```azurecli-interactive
az keyvault set-policy --name "<your-keyvault-name>" --object-id "<principalId>" --secret-permissions get list
```

Политики доступа также можно назначать с помощью [портала Azure](./assign-access-policy-portal.md) или [PowerShell](./assign-access-policy-powershell.md).

### <a name="modify-the-app-to-access-your-key-vault"></a>Изменение приложения для доступа к хранилищу ключей

При роботе с этим руководством вы будете использовать [клиентскую библиотеку секретов Azure Key Vault](/dotnet/api/overview/azure/security.keyvault.secrets-readme). Кроме того, можно использовать [клиентскую библиотеку сертификатов Azure Key Vault](/dotnet/api/overview/azure/security.keyvault.certificates-readme) или [клиентскую библиотеку ключей Azure Key Vault](/dotnet/api/overview/azure/security.keyvault.keys-readme).

#### <a name="install-the-packages"></a>Установка пакетов

В окне терминала установите клиентскую библиотеку секретов Azure Key Vault для .NET и клиентскую библиотеку удостоверений Azure.

```console
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

#### <a name="update-the-code"></a>Обновление кода

Найдите и откройте файл Startup.cs в проекте akvwebapp. 

Добавьте следующие строки в заголовок:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Core;
```

Добавьте следующие строки перед вызовом `app.UseEndpoints`, включив в URI значение `vaultUri` для вашего хранилища ключей. Этот код использует [DefaultAzureCredential()](/dotnet/api/azure.identity.defaultazurecredential) для проверки подлинности в Key Vault, для которой используется токен из управляемого удостоверения. Дополнительные сведения о проверке подлинности в Key Vault см. в [руководстве для разработчиков](./developers-guide.md#authenticate-to-key-vault-in-code). В этом коде также реализована экспоненциальная задержка для повторных попыток на случай регулирования запросов к Key Vault. Дополнительные сведения об ограничениях транзакций в Key Vault см. в статье [Руководство по регулированию Azure Key Vault](./overview-throttling.md)

```csharp
SecretClientOptions options = new SecretClientOptions()
    {
        Retry =
        {
            Delay= TimeSpan.FromSeconds(2),
            MaxDelay = TimeSpan.FromSeconds(16),
            MaxRetries = 5,
            Mode = RetryMode.Exponential
         }
    };
var client = new SecretClient(new Uri("https://<your-unique-key-vault-name>.vault.azure.net/"), new DefaultAzureCredential(),options);

KeyVaultSecret secret = client.GetSecret("<mySecret>");

string secretValue = secret.Value;
```

Измените строку `await context.Response.WriteAsync("Hello World!");` следующим образом.

```csharp
await context.Response.WriteAsync(secretValue);
```

Не забудьте сохранить внесенные изменения, прежде чем переходить к следующему шагу.

#### <a name="redeploy-your-web-app"></a>Повторное развертывание веб-приложения

Теперь, когда вы изменили код, его можно повторно развернуть в Azure с помощью следующих команд Git.

```bash
git add .
git commit -m "Updated web app to access my key vault"
git push azure main
```

## <a name="go-to-your-completed-web-app"></a>Переход на страницу готового веб-приложения

```bash
http://<your-webapp-name>.azurewebsites.net
```

Вместо текста "Hello World", который мы видели раньше, отобразится значение секрета.

## <a name="next-steps"></a>Дальнейшие действия

- [Руководство. Использование Azure Key Vault с виртуальной машиной в .NET](./tutorial-net-virtual-machine.md)
- Дополнительные сведения см. в статье [Использование управляемых удостоверений для ресурсов Azure](../../active-directory/managed-identities-azure-resources/overview.md).
- Изучите [руководство для разработчиков](./developers-guide.md)
- [Безопасный доступ к хранилищу ключей](./secure-your-key-vault.md)