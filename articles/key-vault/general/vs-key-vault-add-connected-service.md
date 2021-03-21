---
title: Добавление поддержки Key Vault в проект ASP.NET с помощью Visual Studio — Azure Key Vault | Документация Майкрософт
description: Это руководство поможет вам узнать, как добавить поддержку Key Vault в веб-приложение ASP.NET или ASP.NET Core.
services: key-vault
author: ghogen
manager: jillfra
ms.service: key-vault
ms.custom: vs-azure, devx-track-csharp
ms.topic: how-to
ms.date: 08/07/2019
ms.author: ghogen
ms.openlocfilehash: 9c62534acdbfbff7fd4e718bad1f07a92c641626
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92792401"
---
# <a name="add-key-vault-to-your-web-application-by-using-visual-studio-connected-services"></a>Добавление хранилища ключей в веб-приложение с помощью функции "Подключенные службы" в Visual Studio

В этом руководстве вы узнаете, как легко добавить все, что нужно, чтобы приступить к использованию Azure Key Vault для управления секретами веб-проектов в Visual Studio при работе с проектом ASP.NET Core или проектом ASP.NET любого типа. С помощью функции Подключенные службы в Visual Studio можно автоматически добавить в Visual Studio все пакеты NuGet и параметры конфигурации, необходимые для подключения к Key Vault в Azure.

Подробные сведения об изменениях, вносимых функцией "Подключенные службы" в проект для поддержки Key Vault, можно получить из раздела [Что произошло с моим проектом ASP.NET при добавлении подключенной службы Key Vault в Visual Studio?](#how-your-aspnet-framework-project-is-modified) или [Что произошло с моим проектом ASP.NET Core при добавлении подключенной службы Key Vault в Visual Studio?](#how-your-aspnet-core-project-is-modified)

## <a name="prerequisites"></a>Предварительные условия

- **Подписка Azure**. Если у вас нет подписки, зарегистрируйтесь, чтобы получить [бесплатную учетную запись](https://azure.microsoft.com/pricing/free-trial/).
- **Visual Studio 2019 версии 16,3** или более поздней, [скачайте ее сейчас](https://aka.ms/vsdownload?utm_source=mscom&utm_campaign=msdocs).


## <a name="add-key-vault-support-to-your-project"></a>Добавление поддержки Key Vault в проект

Прежде чем начать, убедитесь, что вы вошли в Visual Studio. Войдите, используя ту же учетную запись, которая используется для подписки Azure. Затем откройте веб-проект ASP.NET 4.7.1 или более поздней версии или ASP.NET Core 2,0 и выполните следующие действия.

1. В **Обозреватель решений** щелкните правой кнопкой мыши проект, для которого требуется добавить поддержку Key Vault, и выберите команду **Добавить**  >  **подключенную службу**  >  **Добавить**.
   Откроется страница "Подключенная служба" с перечнем служб, которые можно добавить в проект.
1. В меню доступных служб выберите **Azure Key Vault** и нажмите кнопку **Далее**.

   ![Выберите "Azure Key Vault"](../media/vs-key-vault-add-connected-service/key-vault-connected-service.png)

1. Выберите подписку, которую хотите использовать, а затем выберите существующую Key Vault и нажмите кнопку **Готово**. 

   ![Выберите свою подписку.](../media/vs-key-vault-add-connected-service/key-vault-connected-service-select-vault.png)

Теперь подключение к Key Vault установлено, и вы можете получить доступ к секретам в коде. Дальнейшие действия отличаются в зависимости от того, используется ASP.NET 4.7.1 или ASP.NET Core.

## <a name="access-your-secrets-in-code-aspnet-core"></a>Доступ к секретам в коде (ASP.NET Core)

1. Откройте один из файлов подкачки, например *index. cshtml. CS* , и напишите следующий код:
   1. Включите ссылку на `Microsoft.Extensions.Configuration` с помощью директивы using:

       ```csharp
       using Microsoft.Extensions.Configuration;
       ```

   1. Добавьте переменную конфигурации.

      ```csharp
      private static IConfiguration _configuration;
      ```

   1. Добавьте этот конструктор или замените существующий конструктор следующим:

       ```csharp
       public IndexModel(IConfiguration configuration)
       {
           _configuration = configuration;
       }
       ```

   1. Внесите изменения в метод `OnGet`. Обновите значение заполнителя, показанное здесь, на имя секрета, созданного в приведенных выше командах.

       ```csharp
       public void OnGet()
       {
           ViewData["Message"] = "My key val = " + _configuration["<YourSecretNameStoredInKeyVault>"];
       }
       ```

   1. Чтобы подтвердить значение во время выполнения, добавьте код для вывода в `ViewData["Message"]` *CSHTML* -файл, чтобы отобразить секрет в сообщении.

      ```cshtml
          <p>@ViewData["Message"]</p>
      ```

Вы можете запустить приложение локально, чтобы убедиться, что секрет успешно получен из Key Vault.

## <a name="access-your-secrets-aspnet"></a>Доступ к своим секретам (ASP.NET)
Можно настроить конфигурацию таким образом, чтобы web.config файл был пустым значением в `appSettings` элементе, который заменяется значением true во время выполнения. Затем можно получить доступ через `ConfigurationManager.AppSettings` структуру данных.

1. В обозреватель решений щелкните правой кнопкой мыши проект и выберите Управление пакетами NuGet. На вкладке Обзор найдите и установите [Microsoft.Configuration.Configуратионбуилдерс. Azure.](https://www.nuget.org/packages/Microsoft.Configuration.ConfigurationBuilders.Azure/)
 
1. Откройте файл web.config и напишите следующий код:
    1. Добавьте `configSections` и `configBuilders` :
        ```xml
         <configSections>
            <section
                name="configBuilders"
                type="System.Configuration.ConfigurationBuildersSection, System.Configuration, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a"
                restartOnExternalChanges="false"
                requirePermission="false" />
         </configSections>
         <configBuilders>
            <builders>
            <add
                    name="AzureKeyVault"
                    vaultName="vaultname"
                    type="Microsoft.Configuration.ConfigurationBuilders.AzureKeyVaultConfigBuilder, Microsoft.Configuration.ConfigurationBuilders.Azure, Version=1.0.0.0, Culture=neutral"
                    vaultUri="https://vaultname.vault.azure.net" />
            </builders>
         </configBuilders>
        ```
    1. Найдите тег appSettings, добавьте атрибут `configBuilders="AzureKeyVault"` и добавьте строку:
        ```xml
         <add key="<secretNameInYourKeyVault>" value="dummy"/>
        ```

1. Измените `About` метод в *HomeController. CS*, чтобы отобразить значение для подтверждения.

   ```csharp
   public ActionResult About()
   {
       ViewBag.Message = "Key vault value = " + ConfigurationManager.AppSettings["<secretNameInYourKeyVault>"];
   }
   ```
1. Запустите приложение локально в отладчике, перейдите на вкладку **о программе** и убедитесь, что отображается значение из Key Vault.

## <a name="troubleshooting"></a>Устранение неполадок

Если Key Vault работает на учетная запись Майкрософт, отличном от того, который вы выполнили в Visual Studio (например, Key Vault работает в вашей рабочей учетной записи, но Visual Studio использует вашу личную учетную запись), в файле Program. CS появляется сообщение об ошибке: Visual Studio не удается получить доступ к Key Vault. Чтобы устранить эту проблему:

1. Перейдите к [портал Azure](https://portal.azure.com) и откройте Key Vault.

1. Выберите **политики доступа**, затем **Добавить политику доступа** и выберите учетную запись, с которой вы вошли в качестве участника.

1. В Visual Studio выберите   >  **Параметры учетной записи** файла.
Выберите **Добавить учетную запись** в разделе **все учетная запись** . Войдите, используя учетную запись, выбранную в качестве участника политики доступа.

1. Выберите **Сервис**  >  **Параметры** и найдите **проверку подлинности службы Azure**. Затем выберите учетную запись, которую вы только что добавили в Visual Studio.

Теперь при отладке приложения Visual Studio подключается к учетной записи, в которой находится Key Vault.

## <a name="how-your-aspnet-core-project-is-modified"></a>Изменение проекта ASP.NET Core

В этом разделе указаны точные изменения, внесенные в проект ASP.NET при добавлении подключенной службы Key Vault с помощью Visual Studio.

### <a name="added-references-for-aspnet-core"></a>Добавлены ссылки на ASP.NET Core

Влияет на ссылки .NET на файл проекта и на ссылки на пакет NuGet.

| Тип | Справочник |
| --- | --- |
| NuGet | Microsoft.AspNetCore.AzureKeyVault.HostingStartup |

### <a name="added-files-for-aspnet-core"></a>Добавлены файлы для ASP.NET Core

- `ConnectedService.json` Добавлены сведения о поставщике подключенной службы, версии и ссылке на документацию.

### <a name="project-file-changes-for-aspnet-core"></a>Изменения в файле проекта для ASP.NET Core

- Добавлены Подключенные службы ItemGroup и `ConnectedServices.json` File.

### <a name="launchsettingsjson-changes-for-aspnet-core"></a>launchsettings.jsоб изменениях для ASP.NET Core

- В профиль IIS Express и профиль, который соответствует имени вашего веб-проекта, добавлены следующие записи переменных среды.

    ```json
      "environmentVariables": {
        "ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONENABLED": "true",
        "ASPNETCORE_HOSTINGSTARTUP__KEYVAULT__CONFIGURATIONVAULT": "<your keyvault URL>"
      }
    ```

### <a name="changes-on-azure-for-aspnet-core"></a>Изменения в Azure для ASP.NET Core

- Создана группа ресурсов или использована существующая.
- В указанной группе ресурсов создано хранилище ключей.

## <a name="how-your-aspnet-framework-project-is-modified"></a>Изменение проекта ASP.NET Framework

В этом разделе указаны точные изменения, внесенные в проект ASP.NET при добавлении подключенной службы Key Vault с помощью Visual Studio.

### <a name="added-references-for-aspnet-framework"></a>Добавлены ссылки для ASP.NET Framework

Затрагивает ссылки на файл проекта .NET и `packages.config` (ссылки NuGet).

| Тип | Справочник |
| --- | --- |
| .NET; NuGet | Azure. Identity |
| .NET; NuGet | Azure. Security. KeyVault. Keys |
| .NET; NuGet | Azure. Security. KeyVault. Секреты |

> [!IMPORTANT] 
> По умолчанию Azure. Identity 1.1.1 установлен, что не поддерживает учетные данные Visual Studio. Ссылку на пакет можно обновить вручную до версии 1.2 и использовать учетные данные Visual Studio.

### <a name="added-files-for-aspnet-framework"></a>Добавлены файлы для ASP.NET Framework

- `ConnectedService.json` Добавлены сведения о поставщике подключенной службы, версии и ссылке на документацию.

### <a name="project-file-changes-for-aspnet-framework"></a>Изменения в файле проекта для ASP.NET Framework

- Добавлены элемент ItemGroup "Подключенные службы" и файл ConnectedServices.json.
- Ссылки на сборки .NET, описанные в разделе [Добавленные ссылки](#added-references-for-aspnet-framework).

## <a name="next-steps"></a>Дальнейшие действия

Если вы пропустили этот учебник, ваши разрешения Key Vault настроены для работы с вашей собственной подпиской Azure, но это может быть нежелательно для рабочего сценария. Вы можете создать управляемое удостоверение для управления доступом к Key Vault для вашего приложения. См. статью [Проверка подлинности в Key Vault](./authentication.md) и [назначение политики доступа Key Vault](./assign-access-policy-portal.md).

Дополнительные сведения о Key Vault разработке см. в разделе [руководств разработчика Key Vault](developers-guide.md).