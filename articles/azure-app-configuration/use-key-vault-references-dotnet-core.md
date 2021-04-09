---
title: Учебник по использованию ссылок Key Vault службы "Конфигурация приложений Azure" в приложении ASP.NET Core | Документация Майкрософт
description: В этом учебнике описано, как использовать ссылки Key Vault службы "Конфигурация приложений Azure" из приложения ASP.NET Core.
services: azure-app-configuration
documentationcenter: ''
author: AlexandraKemperMS
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 04/08/2020
ms.author: alkemper
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 37bc7fbcd366455668d5316e45ffbf79127a49f3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99981244"
---
# <a name="tutorial-use-key-vault-references-in-an-aspnet-core-app"></a>Руководство по использованию ссылок Key Vault в приложении ASP.NET Core

В этом учебнике вы узнаете, как использовать службу "Конфигурация приложений Azure" с Azure Key Vault. Конфигурация приложений Azure и Key Vault — это дополняющие службы, которые используются параллельно в большинстве развертываний приложений.

Служба "Конфигурация приложений Azure" упрощает их использование, позволяя создавать ключи, ссылающиеся на значения, хранящиеся в Key Vault. Когда служба "Конфигурация приложений Azure" создает такие ключи, она сохраняет URI значений Key Vault, а не сами значения.

С помощью поставщика клиента "Конфигурация приложений Azure" ваше приложение получает ссылки на Key Vault так же, как и любые другие ключи, хранящиеся в службе "Конфигурация приложений Azure". В этом случае значения, хранящиеся в службе "Конфигурация приложений Azure", являются URI, которые ссылаются на значения в Key Vault. Это не значения или учетные данные Key Vault. Так как поставщик клиента распознает ключи как ссылки на Key Vault, для получения их значений он использует Key Vault.

Приложение отвечает за правильную проверку подлинности как в службе "Конфигурация приложений", так и в Key Vault. Две службы не обмениваются данными напрямую.

В этом учебнике показано, как можно реализовать ссылки на Key Vault в коде. При этом в качестве основы используется код веб-приложения, представленный в кратких руководствах. Прежде чем продолжить, ознакомьтесь со статьей [Краткое руководство. Создание приложения ASP.NET Core с помощью службы "Конфигурация приложений Azure"](./quickstart-aspnet-core-app.md).

Вы можете выполнять шаги в этом учебнике с помощью любого редактора кода. Например, [Visual Studio Code](https://code.visualstudio.com/) — это кроссплатформенный редактор кода, доступный для операционных систем Windows, macOS и Linux.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * создание ключа службы "Конфигурация приложений Azure", который ссылается на значение, хранящееся в Key Vault;
> * доступ к значению этого ключа из веб-приложения ASP.NET Core.

## <a name="prerequisites"></a>Предварительные требования

Перед работой с этим учебником установите [пакет SDK для .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="create-a-vault"></a>Создание хранилища

1. Выберите параметр **Создать ресурс** в верхнем левом углу окна портала Azure.

    ![Снимок экрана: вариант "Создать ресурс" на портале Azure.](./media/quickstarts/search-services.png)
1. В поле поиска введите **Key Vault**.
1. В списке результатов выберите **Key Vault** слева.
1. В разделе **Хранилища ключей** выберите **Добавить**.
1. Справа в разделе **Создать Key Vault** введите приведенные ниже сведения.
    - Выберите **Подписка**, чтобы выбрать подписку.
    - В разделе **Группа ресурсов** выберите **Создать** и введите имя группы ресурсов.
    - В поле **Имя Key Vault** укажите уникальное имя. Для работы с этим учебником введите **Contoso-vault2**.
    - В раскрывающемся списке **Регион** выберите расположение.
1. Для других свойств раздела **создания Key Vault** оставьте значения по умолчанию.
1. Нажмите кнопку **создания**.

На этом этапе доступ к этому новому хранилищу может получать только учетная запись Azure.

![Снимок экрана: ваше хранилище ключей.](./media/quickstarts/vault-properties.png)

## <a name="add-a-secret-to-key-vault"></a>Добавление секрета в Key Vault

Чтобы добавить секрет в хранилище, вам просто нужно выполнить несколько дополнительных шагов. В этом случае мы добавим сообщение, которое можно использовать для проверки возвращения Key Vault. Сообщение называется **Message**, и в нем сохраняется значение "Hello from Key Vault".

1. На странице свойств Key Vault выберите **Секреты**.
1. Выберите **Создать или импортировать**.
1. В области **Создание секрета** выберите следующие значения:
    - **Параметры отправки.** Введите **Вручную**.
    - **Name** (Имя). Введите **Message**.
    - **Value** (Значение): Введите **Hello from Key Vault**.
1. Для других свойств раздела **Создание секрета** оставьте значения по умолчанию.
1. Нажмите кнопку **создания**.

## <a name="add-a-key-vault-reference-to-app-configuration"></a>Добавление ссылки на Key Vault в службу "Конфигурация приложений Azure"

1. Войдите на [портал Azure](https://portal.azure.com). Щелкните **Все ресурсы** и выберите экземпляр хранилища "Конфигурация приложений Azure", который вы создали по инструкциям из краткого руководства.

1. Выберите **Обозреватель конфигураций**.

1. Выберите **+ Создать** > **Ссылка на хранилище ключей** и укажите следующие значения:
    - **Ключ**: Выберите **TestApp:Settings:KeyVaultMessage**.
    - **Метка** — оставьте это значение пустым.
    - **Подписка**, **Группа ресурсов** и **Хранилище ключей**: введите значения, соответствующие значениям хранилища ключей, созданного в предыдущем разделе.
    - **Секрет**: выберите секрет с именем **Message**, созданный в предыдущем разделе.

## <a name="connect-to-key-vault"></a>Подключение к Key Vault

1. В этом учебнике вы используете субъект-службу для проверки подлинности в KeyVault. Создайте субъект-службу с помощью команды [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) Azure CLI:

    ```azurecli
    az ad sp create-for-rbac -n "http://mySP" --sdk-auth
    ```

    Эта операция возвращает ряд пар "ключ — значение".

    ```console
    {
    "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
    "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
    "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
    "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
    "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
    "resourceManagerEndpointUrl": "https://management.azure.com/",
    "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
    "galleryEndpointUrl": "https://gallery.azure.com/",
    "managementEndpointUrl": "https://management.core.windows.net/"
    }
    ```

1. Выполните следующую команду, чтобы предоставить субъекту-службе доступ к хранилищу ключей:

    ```cmd
    az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --secret-permissions delete get list set --key-permissions create decrypt delete encrypt get list unwrapKey wrapKey
    ```

1. Добавьте переменные среды для хранения значений *clientId*, *clientSecret* и *tenantId*.

    #### <a name="windows-command-prompt"></a>[Командная строка Windows](#tab/cmd)

    ```cmd
    setx AZURE_CLIENT_ID <clientId-of-your-service-principal>
    setx AZURE_CLIENT_SECRET <clientSecret-of-your-service-principal>
    setx AZURE_TENANT_ID <tenantId-of-your-service-principal>
    ```

    #### <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```PowerShell
    $Env:AZURE_CLIENT_ID = <clientId-of-your-service-principal>
    $Env:AZURE_CLIENT_SECRET = <clientSecret-of-your-service-principal>
    $Env:AZURE_TENANT_ID = <tenantId-of-your-service-principal>
    ```

    #### <a name="bash"></a>[Bash](#tab/bash)

    ```bash
    export AZURE_CLIENT_ID = <clientId-of-your-service-principal>
    export AZURE_CLIENT_SECRET = <clientSecret-of-your-service-principal>
    export AZURE_TENANT_ID = <tenantId-of-your-service-principal>
    ```

    ---

    > [!NOTE]
    > Эти учетные данные Key Vault используются только в вашем приложении. Приложение выполняет проверку подлинности непосредственно в Key Vault с использованием этих учетных данных. Они никогда не передаются в службу настройки приложений.

1. Перезапустите терминал, чтобы загрузить эти новые переменные среды.

## <a name="update-your-code-to-use-a-key-vault-reference"></a>Обновление кода для использования ссылки Key Vault

1. Добавьте ссылку на необходимые пакеты NuGet, выполнив следующую команду.

    ```dotnetcli
    dotnet add package Azure.Identity
    ```

1. Откройте файл *Program.cs* и добавьте ссылки на приведенные ниже необходимые пакеты.

    ```csharp
    using Azure.Identity;
    ```

1. Обновите метод `CreateWebHostBuilder`, чтобы использовать службу "Конфигурация приложений", путем вызова метода `config.AddAzureAppConfiguration`. Добавьте параметр `ConfigureKeyVault` и передайте правильные учетные данные в Key Vault.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                            .ConfigureKeyVault(kv =>
                            {
                                kv.SetCredential(new DefaultAzureCredential());
                            });
                });
            })
            .UseStartup<Startup>();
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                            .ConfigureKeyVault(kv =>
                            {
                                kv.SetCredential(new DefaultAzureCredential());
                            });
                });
            })
            .UseStartup<Startup>());
    ```

1. При инициализации подключения к службе "Конфигурация приложений" устанавливается подключение к Key Vault путем вызова метода `ConfigureKeyVault`. После инициализации можно получить доступ к значениям ссылок на Key Vault так же, как и к значениям обычных ключей "Конфигурация приложений Azure".

    Чтобы увидеть этот процесс в действии, откройте файл *Index.cshtml* в папке **Представления** > **Главная страница**. Замените содержимое файла приведенным ниже кодом.

    ```html
    @using Microsoft.Extensions.Configuration
    @inject IConfiguration Configuration

    <style>
        body {
            background-color: @Configuration["TestApp:Settings:BackgroundColor"]
        }
        h1 {
            color: @Configuration["TestApp:Settings:FontColor"];
            font-size: @Configuration["TestApp:Settings:FontSize"]px;
        }
    </style>

    <h1>@Configuration["TestApp:Settings:Message"]
        and @Configuration["TestApp:Settings:KeyVaultMessage"]</h1>
    ```

    Вы получаете доступ к значению ссылки на Key Vault **TestApp:Settings:KeyVaultMessage** так же, как и к значению конфигурации **TestApp:Settings:Message**.

## <a name="build-and-run-the-app-locally"></a>Создание и запуск приложения локально

1. Чтобы создать приложение с помощью .NET Core CLI, выполните следующую команду в оболочке командной строки:

    ```dotnetcli
    dotnet build
    ```

1. Когда создание завершится, запустите веб-приложение локально с помощью следующей команды:

    ```dotnetcli
    dotnet run
    ```

1. Откройте окно браузера и перейдите по адресу `http://localhost:5000`, который является URL-адресом по умолчанию для веб-приложения, размещенного локально.

    ![Краткое руководство по запуску приложения, размещенного локально](./media/key-vault-reference-launch-local.png)

## <a name="clean-up-resources"></a>Очистка ресурсов

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Дальнейшие действия

В этом учебнике вы создали ключ службы "Конфигурация приложений Azure", который ссылается на значение, хранящееся в Key Vault. Чтобы узнать, как добавить удостоверение службы, управляемое Azure, которое упрощает доступ к службе "Конфигурация приложения Azure" и Key Vault, перейдите к следующему учебнику.

> [!div class="nextstepaction"]
> [Руководство по интеграции с управляемыми удостоверениями Azure](./howto-integrate-azure-managed-service-identity.md)
