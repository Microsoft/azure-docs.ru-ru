---
title: Использование управляемых удостоверений для получения доступа к службе "Конфигурация приложений"
titleSuffix: Azure App Configuration
description: Проверка подлинности в конфигурации приложений Azure с помощью управляемых удостоверений
author: AlexandraKemperMS
ms.author: alkemper
ms.service: azure-app-configuration
ms.custom: devx-track-csharp, fasttrack-edit
ms.topic: conceptual
ms.date: 2/25/2020
ms.openlocfilehash: 386a0e27c0f73f5bcd42397ed515f7561d5097fd
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104955063"
---
# <a name="use-managed-identities-to-access-app-configuration"></a>Использование управляемых удостоверений для получения доступа к службе "Конфигурация приложений"

Azure Active Directory [управляемые удостоверения](../active-directory/managed-identities-azure-resources/overview.md) упрощают управление секретами для облачного приложения. С помощью управляемого удостоверения ваш код может использовать субъект-службу, созданный для службы Azure, в которой она выполняется. Вы используете управляемое удостоверение, а не отдельные учетные данные, хранящиеся в Azure Key Vault, или локальную строку подключения.

В конфигурации приложений Azure, а также в клиентских библиотеках .NET Core, платформа .NET Framework и Java есть встроенная поддержка идентификации. Хотя это и не обязательно, управляемое удостоверение устраняет необходимость в маркере доступа, содержащем секреты. Код может получить доступ к хранилищу конфигураций приложений, используя только конечную точку службы. Этот URL-адрес можно внедрить в код напрямую без предоставления секрета.

В этой статье показано, как можно воспользоваться преимуществами управляемого удостоверения для доступа к конфигурации приложения. При этом в качестве основы используется код веб-приложения, представленный в кратких руководствах. Прежде чем продолжить, сначала  [Создайте приложение ASP.NET Core с конфигурацией приложения](./quickstart-aspnet-core-app.md) .

> [!NOTE]
> В этой статье в качестве примера используется служба приложений Azure, но та же концепция применяется к любой другой службе Azure, которая поддерживает управляемое удостоверение, например [службу Azure Kubernetes](../aks/use-azure-ad-pod-identity.md), [виртуальную машину Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)и [экземпляры контейнеров Azure](../container-instances/container-instances-managed-identity.md). Если Рабочая нагрузка размещается в одной из этих служб, вы также можете воспользоваться поддержкой управляемого удостоверения службы.

В этой статье также показано, как можно использовать управляемое удостоверение в сочетании со ссылками на Key Vault конфигурации приложения. С одним управляемым удостоверением можно легко получить доступ к секретам из Key Vault и значений конфигурации из конфигурации приложения. Если вы хотите изучить эту возможность, то сначала [используйте Key Vault ссылки с ASP.NET Core](./use-key-vault-references-dotnet-core.md) .

Вы можете выполнять шаги в этом учебнике с помощью любого редактора кода. [Visual Studio Code](https://code.visualstudio.com/) является отличным вариантом, который доступен на платформах Windows, macOS и Linux.

Вы узнаете, как выполнять следующие задачи:

> [!div class="checklist"]
> * предоставление доступа к службе "Конфигурация приложений" с помощью управляемого удостоверения;
> * настройка приложения на использование управляемого удостоверения при подключении к службе "Конфигурация приложений".
> * При необходимости настройте приложение для использования управляемого удостоверения при подключении к Key Vault через конфигурацию приложения Key Vault ссылке.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством требуется:

* [пакет SDK для .NET Core](https://www.microsoft.com/net/download/windows);
* [Azure Cloud Shell настроены](../cloud-shell/quickstart.md).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="add-a-managed-identity"></a>Добавление управляемого удостоверения

Чтобы настроить управляемое удостоверение на портале, сначала необходимо создать приложение, а затем включить эту функцию.

1. Создайте экземпляр служб приложений в [портал Azure](https://portal.azure.com) , как обычно. Перейдите к нему на портале.

1. Прокрутите вниз до группы **параметров** в левой панели и выберите **Удостоверение**.

1. На вкладке **Назначено системой** для параметра **Состояние** установите значение **Вкл.** и выберите **Сохранить**.

1. Ответьте **Да** при появлении запроса на включение управляемого удостоверения, назначенного системой.

    ![Настройка управляемого удостоверения в службе приложений](./media/set-managed-identity-app-service.png)

## <a name="grant-access-to-app-configuration"></a>Предоставление доступа к службе "Конфигурация приложений"

1. В [портал Azure](https://portal.azure.com)выберите **все ресурсы** и выберите хранилище конфигураций приложений, которое вы создали в кратком руководстве.

1. Выберите **Управление доступом (IAM)** .

1. На вкладке **Проверить доступ** щелкните **Добавить** в пользовательском интерфейсе карточки **Добавить назначение роли**.

1. В разделе **роль** выберите **модуль чтения данных конфигурации приложения**. В поле **Назначение доступа к** задайте значение **Служба приложений** (в разделе **Управляемое удостоверение, назначаемое системой**).

1. В поле **Подписка** выберите подписку Azure. Выберите ресурс Службы приложений для своего приложения.

1. Щелкните **Сохранить**.

    ![Добавление управляемого удостоверения](./media/add-managed-identity.png)

1. Необязательно. Если вы хотите предоставить доступ к Key Vault, следуйте указаниям [Key Vault](../key-vault/general/assign-access-policy-portal.md)в этой инструкции.

## <a name="use-a-managed-identity"></a>Использование управляемого удостоверения

1. Добавьте ссылку на пакет *Azure. Identity* :

    ```bash
    dotnet add package Azure.Identity
    ```

1. Найдите конечную точку в хранилище конфигураций приложения. Этот URL-адрес указан на вкладке " **ключи доступа** " для магазина в портал Azure.

1. Откройте *appsettings.json* и добавьте следующий скрипт. Замените *\<service_endpoint>* , включая квадратные скобки, URL-адресом хранилища конфигураций приложения.

    ```json
    "AppConfig": {
        "Endpoint": "<service_endpoint>"
    }
    ```

1. Откройте *Program. CS* и добавьте ссылку на `Azure.Identity` `Microsoft.Azure.Services.AppAuthentication` пространства имен и.

    ```csharp-interactive
    using Azure.Identity;
    ```

1. Если вы хотите получить доступ только к значениям, хранящимся непосредственно в конфигурации приложения, обновите `CreateWebHostBuilder` метод, заменив `config.AddAzureAppConfiguration()` метод (он находится в `Microsoft.Azure.AppConfiguration.AspNetCore` пакете).

    > [!IMPORTANT]
    > `CreateHostBuilder` заменяет `CreateWebHostBuilder` в .NET Core 3.0.  Выберите правильный синтаксис в зависимости от среды.

    ### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
               .ConfigureAppConfiguration((hostingContext, config) =>
               {
                   var settings = config.Build();
                   config.AddAzureAppConfiguration(options =>
                       options.Connect(new Uri(settings["AppConfig:Endpoint"]), new ManagedIdentityCredential()));
               })
               .UseStartup<Startup>();
    ```

    ### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                        options.Connect(new Uri(settings["AppConfig:Endpoint"]), new ManagedIdentityCredential()));
                });
            })
            .UseStartup<Startup>());
    ```
    ---

    > [!NOTE]
    > Если вы хотите использовать **управляемое пользователем удостоверение**, обязательно укажите ClientID при создании [манажедидентитикредентиал](/dotnet/api/azure.identity.managedidentitycredential).
    >```
    >config.AddAzureAppConfiguration(options =>
    >   options.Connect(new Uri(settings["AppConfig:Endpoint"]), new ManagedIdentityCredential(<your_clientId>)));
    >```
    >Как описано в разделах [управляемые удостоверения для ресурсов Azure: вопросы и ответы](../active-directory/managed-identities-azure-resources/known-issues.md#what-identity-will-imds-default-to-if-dont-specify-the-identity-in-the-request), можно определить, какое управляемое удостоверение используется по умолчанию. В этом случае библиотека удостоверений Azure позволяет указать необходимое удостоверение, чтобы избежать проблем среды выполнения превысил допустимое в будущем (например, если Добавлено новое управляемое удостоверение, назначенное пользователем, или если включено управляемое системой удостоверение). Поэтому необходимо указать clientId, даже если определено только одно назначенное пользователем управляемое удостоверение, а управляемое удостоверение не назначено системой.


1. Чтобы использовать значения конфигурации приложения и ссылки Key Vault, обновите *программу. CS* , как показано ниже. Этот код вызывает `SetCredential` как часть `ConfigureKeyVault` , чтобы сообщить поставщику конфигурации, какие учетные данные следует использовать при проверке подлинности в Key Vault.

    ### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
               .ConfigureAppConfiguration((hostingContext, config) =>
               {
                   var settings = config.Build();
                   var credentials = new ManagedIdentityCredential();

                   config.AddAzureAppConfiguration(options =>
                   {
                       options.Connect(new Uri(settings["AppConfig:Endpoint"]), credentials)
                              .ConfigureKeyVault(kv =>
                              {
                                 kv.SetCredential(credentials);
                              });
                   });
               })
               .UseStartup<Startup>();
    ```

    ### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    var credentials = new ManagedIdentityCredential();

                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(new Uri(settings["AppConfig:Endpoint"]), credentials)
                               .ConfigureKeyVault(kv =>
                               {
                                   kv.SetCredential(credentials);
                               });
                    });
                });
            })
            .UseStartup<Startup>());
    ```
    ---

    Теперь доступ к Key Vault ссылкам можно получить так же, как и к любому другому ключу конфигурации приложения. Поставщик конфигурации будет использовать `ManagedIdentityCredential` для проверки подлинности в Key Vault и получения значения.

    > [!NOTE]
    > `ManagedIdentityCredential`Работает только в средах Azure служб, поддерживающих аутентификацию управляемого удостоверения. Он не работает в локальной среде. Используйте [`DefaultAzureCredential`](/dotnet/api/azure.identity.defaultazurecredential) , чтобы код работал как в локальных средах, так и в среде Azure, так как он будет возвращаться к некоторым вариантам проверки подлинности, включая управляемое удостоверение.
    > 
    > Если вы хотите использовать **управляемое удостоверение пользователя асигнед** с `DefaultAzureCredential` при развертывании в Azure, [Укажите ClientID](/dotnet/api/overview/azure/identity-readme#specifying-a-user-assigned-managed-identity-with-the-defaultazurecredential).

[!INCLUDE [Prepare repository](../../includes/app-service-deploy-prepare-repo.md)]

## <a name="deploy-from-local-git"></a>Развертывание из локального репозитория Git

Самый простой способ включить локальное развертывание Git для приложения с сервером сборки KUDU — использовать [Azure Cloud Shell](https://shell.azure.com).

### <a name="configure-a-deployment-user"></a>Настойка пользователя развертывания

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="enable-local-git-with-kudu"></a>Включить локальный Git с помощью Kudu
Если у вас нет локального репозитория Git для приложения, необходимо его инициализировать. Чтобы инициализировать локальный репозиторий Git, выполните следующие команды в каталоге проекта приложения:

```cmd
git init
git add .
git commit -m "Initial version"
```

Чтобы включить локальное развертывание Git для приложения с помощью сервера сборки KUDU, выполните команду [`az webapp deployment source config-local-git`](/cli/azure/webapp/deployment/#az-webapp-deployment-source-config-local-git) в Cloud Shell.

```azurecli-interactive
az webapp deployment source config-local-git --name <app_name> --resource-group <group_name>
```

Эта команда дает примерно следующий результат:

```json
{
  "url": "https://<username>@<app_name>.scm.azurewebsites.net/<app_name>.git"
}
```

### <a name="deploy-your-project"></a>Развертывание проекта

В _окне терминала на локальном_ компьютере добавьте удаленное хранилище Azure в локальный репозиторий Git. Замените _\<url>_ URL-адресом удаленного Git, полученным на странице [Включение локального репозитория Git с KUDU](#enable-local-git-with-kudu).

```bash
git remote add azure <url>
```

Отправьте код в удаленное приложение Azure, чтобы развернуть приложение. Когда появится запрос на ввод пароля, введите пароль, созданный в разделе [Настройка пользователя развертывания](#configure-a-deployment-user). Не используйте пароль, который вы используете для входа на портал Azure.

```bash
git push azure main
```

В выходных данных могут отображаться автоматизированные операции среды выполнения, например MSBuild для ASP.NET, `npm install` для Node.js и `pip install` для Python.

### <a name="browse-to-the-azure-web-app"></a>Переход к веб-приложению Azure

Перейдите к своему веб-приложению с помощью браузера, чтобы убедиться, что содержимое развернуто.

```bash
http://<app_name>.azurewebsites.net
```

## <a name="use-managed-identity-in-other-languages"></a>Использование управляемого удостоверения с другими языками

Поставщики службы "Конфигурация приложений" для платформы .NET Framework и Java Spring также имеют встроенную поддержку управляемых удостоверений. При настройке одного из этих поставщиков можно использовать конечную точку URL-адреса вашего хранилища вместо полной строки подключения.

Например, можно обновить консольное приложение платформа .NET Framework, созданное в кратком руководстве, чтобы указать следующие параметры в файле *App.config* :

```xml
    <configSections>
        <section name="configBuilders" type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a" restartOnExternalChanges="false" requirePermission="false" />
    </configSections>

    <configBuilders>
        <builders>
            <add name="MyConfigStore" mode="Greedy" endpoint="${Endpoint}" type="Microsoft.Configuration.ConfigurationBuilders.AzureAppConfigurationBuilder, Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration" />
            <add name="Environment" mode="Greedy" type="Microsoft.Configuration.ConfigurationBuilders.EnvironmentConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Environment" />
        </builders>
    </configBuilders>

    <appSettings configBuilders="Environment,MyConfigStore">
        <add key="AppName" value="Console App Demo" />
        <add key="Endpoint" value ="Set via an environment variable - for example, dev, test, staging, or production endpoint." />
    </appSettings>
```

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия
В этом руководстве вы добавили управляемое удостоверение Azure для упрощения доступа к конфигурации приложения и улучшения управления учетными данными для приложения. Чтобы узнать больше об использовании службы "Конфигурация приложений", перейдите к примерам скриптов Azure CLI.

> [!div class="nextstepaction"]
> [Примеры интерфейса командной строки](./cli-samples.md)