---
title: Учебник по ротации ресурсов с одним набором учетных данных аутентификации, хранимых в Azure Key Vault
description: В этом учебнике описывается, как автоматизировать смену секретов для ресурсов, которые используют один набор учетных данных для аутентификации.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: rotation
ms.service: key-vault
ms.subservice: secrets
ms.topic: tutorial
ms.date: 01/26/2020
ms.author: mbaldwin
ms.custom: devx-track-csharp
ms.openlocfilehash: 526c3d2d85a3f2877f82b3b764f395c51f7c05c0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99805235"
---
# <a name="automate-the-rotation-of-a-secret-for-resources-that-use-one-set-of-authentication-credentials"></a>Автоматизация смены секретов для ресурсов с одним набором учетных данных для аутентификации

Лучшим способом проверки подлинности в службах Azure является использование [управляемого удостоверения](../general/authentication.md), но иногда его использование невозможно. В этих случаях используют ключи доступа или секреты. Время от времени необходимо сменять эти ключи доступа или секреты.

В этом учебнике описано, как автоматизировать периодическую смену секретов для баз данных и служб, которые используют один набор учетных данных для аутентификации. В частности, в этом учебнике выполняется смена паролей SQL Server, хранящихся в Azure Key Vault. Для ротации используется функция, запускаемая уведомлением Сетки событий Azure:


:::image type="content" source="../media/rotate-1.png" alt-text="Схема решения ротации":::

1. За 30 дней до истечения срока действия секрета Key Vault публикует в Сетке событий событие "Срок действия скоро истекает".
1. Сетка событий проверяет подписки на события и с помощью HTTP-запроса методом POST вызывает конечную точку приложения-функции, подписанную на это событие.
1. Приложение-функция получает сведения о секрете, генерирует случайный пароль и создает другую версию секрета с новым паролем в хранилище ключей.
1. Приложение-функция обновляет SQL Server, указав новый пароль.

> [!NOTE]
> Между шагами 3 и 4 может быть задержка. В течение этого времени секрет в Key Vault не сможет пройти проверку подлинности на SQL Server. В случае сбоя на любом из шагов Сетка событий повторит попытку в течение двух часов.

## <a name="prerequisites"></a>Предварительные требования

* Подписка Azure — [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Azure Key Vault
* SQL Server

Если у вас нет существующего Key Vault и SQL Server, можно использовать предоставленную ниже ссылку на развертывание.

[![Изображение с кнопкой "Развернуть в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmain%2FARM-Templates%2FInitial-Setup%2Fazuredeploy.json)

1. Для параметра **Группа ресурсов** выберите **Создать**. Укажите имя группы. В нашем примере используется имя **akvrotation**.
1. В разделе **Sql Admin Login** (Имя входа администратора SQL) введите имя для входа администратора SQL. 
1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **Создать**

:::image type="content" source="../media/rotate-2.png" alt-text="Создайте группу ресурсов":::.

Теперь у вас есть Key Vault и экземпляр SQL Server. Вы можете проверить эту установку в Azure CLI, выполнив следующую команду:

```azurecli
az resource list -o table -g akvrotation
```

Результат будет выглядеть примерно следующим образом:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv           akvrotation      eastus      Microsoft.KeyVault/vaults
akvrotation-sql          akvrotation      eastus      Microsoft.Sql/servers
akvrotation-sql/master   akvrotation      eastus      Microsoft.Sql/servers/databases
akvrotation-sql2         akvrotation      eastus      Microsoft.Sql/servers
akvrotation-sql2/master  akvrotation      eastus      Microsoft.Sql/servers/databases
```

## <a name="create-and-deploy-sql-server-password-rotation-function"></a>Создание и развертывание функции смены пароля SQL Server
> [!IMPORTANT]
> Для показанного ниже шаблона требуется, чтобы Key Vault, экземпляр SQL Server и функция Функций Azure находились в одной группе ресурсов.

Теперь создайте приложение-функцию с управляемым системой удостоверением и другими обязательными компонентами, а затем разверните функции смены пароля SQL Server.

Для приложения-функции требуются следующие компоненты:
- План службы приложений Azure.
- Приложение-функция с функциями смены пароля SQL с триггером события и триггером HTTP. 
- Учетная запись хранения, которая нужна для управления триггерами приложения-функции.
- Политика доступа для получения доступа к секретам в Key Vault с помощью удостоверения приложения-функции.
- Подписка на событие Сетки событий **SecretNearExpiry**.

1. Щелкните ссылку для развертывания шаблона в Azure: 

   [![Изображение с кнопкой "Развернуть в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp%2Fmain%2FARM-Templates%2FFunction%2Fazuredeploy.json)

1. В списке **Группа ресурсов** выберите **avkrotation**.
1. В поле **Sql Server Name** (Имя SQL Server) введите имя SQL Server и пароль для смены.
1. В поле **Имя Key Vault** введите имя хранилища ключей.
1. В поле **Имя приложения-функции** введите имя приложения-функции.
1. В поле **Secret Name** (Имя секрета) введите имя секрета, где будет храниться пароль.
1. В поле **URL-адрес репозитория** введите расположение кода для функции на сайте GitHub ( **https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp.git** ).
1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **Создать**.

:::image type="content" source="../media/rotate-3.png" alt-text="Выбор параметра Review+create (Просмотр и создание)":::
  

Выполнив описанные выше действия, вы создадите учетную запись хранения, ферму серверов и приложение-функцию. Вы можете проверить эту установку в Azure CLI, выполнив следующую команду:

```azurecli
az resource list -o table -g akvrotation
```

Результат будет выглядеть примерно следующим образом:

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv           akvrotation       eastus      Microsoft.KeyVault/vaults
akvrotation-sql          akvrotation       eastus      Microsoft.Sql/servers
akvrotation-sql/master   akvrotation       eastus      Microsoft.Sql/servers/databases
cfogyydrufs5wazfunctions akvrotation       eastus      Microsoft.Storage/storageAccounts
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/serverFarms
akvrotation-fnapp        akvrotation       eastus      Microsoft.Web/sites
akvrotation-fnapp        akvrotation       eastus      Microsoft.insights/components
```

Дополнительные сведения о создании приложения-функции и использовании управляемого удостоверения для доступа к Key Vault см. в статьях [Создание приложения-функции на портале Azure](../../azure-functions/functions-create-function-app-portal.md), [Использование управляемых удостоверений в Службе приложений и Функциях Azure](../../app-service/overview-managed-identity.md) и [Назначение политики доступа к Key Vault с помощью портала Azure](../general/assign-access-policy-portal.md).

### <a name="rotation-function"></a>Функция ротации
Развернутая на предыдущем шаге функция использует событие, чтобы активировать ротацию секрета путем обновления Key Vault и базы данных SQL. 

#### <a name="function-trigger-event"></a>Событие, вызывающее функцию

Приведенная ниже функция считывает данные события и выполняет логику ротации.

```csharp
public static class SimpleRotationEventHandler
{
   [FunctionName("AKVSQLRotation")]
   public static void Run([EventGridTrigger]EventGridEvent eventGridEvent, ILogger log)
   {
      log.LogInformation("C# Event trigger function processed a request.");
      var secretName = eventGridEvent.Subject;
      var secretVersion = Regex.Match(eventGridEvent.Data.ToString(), "Version\":\"([a-z0-9]*)").Groups[1].ToString();
      var keyVaultName = Regex.Match(eventGridEvent.Topic, ".vaults.(.*)").Groups[1].ToString();
      log.LogInformation($"Key Vault Name: {keyVaultName}");
      log.LogInformation($"Secret Name: {secretName}");
      log.LogInformation($"Secret Version: {secretVersion}");

      SecretRotator.RotateSecret(log, secretName, keyVaultName);
   }
}
```

#### <a name="secret-rotation-logic"></a>Логика ротации секретов
Этот метод ротации считывает сведения о базе данных из секрета, создает другую версию секрета и обновляет базу данных, указав новый секрет.

```csharp
    public class SecretRotator
    {
        private const string CredentialIdTag = "CredentialId";
        private const string ProviderAddressTag = "ProviderAddress";
        private const string ValidityPeriodDaysTag = "ValidityPeriodDays";

        public static void RotateSecret(ILogger log, string secretName, string keyVaultName)
        {
            //Retrieve Current Secret
            var kvUri = "https://" + keyVaultName + ".vault.azure.net";
            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential());
            KeyVaultSecret secret = client.GetSecret(secretName);
            log.LogInformation("Secret Info Retrieved");

            //Retrieve Secret Info
            var credentialId = secret.Properties.Tags.ContainsKey(CredentialIdTag) ? secret.Properties.Tags[CredentialIdTag] : "";
            var providerAddress = secret.Properties.Tags.ContainsKey(ProviderAddressTag) ? secret.Properties.Tags[ProviderAddressTag] : "";
            var validityPeriodDays = secret.Properties.Tags.ContainsKey(ValidityPeriodDaysTag) ? secret.Properties.Tags[ValidityPeriodDaysTag] : "";
            log.LogInformation($"Provider Address: {providerAddress}");
            log.LogInformation($"Credential Id: {credentialId}");

            //Check Service Provider connection
            CheckServiceConnection(secret);
            log.LogInformation("Service  Connection Validated");
            
            //Create new password
            var randomPassword = CreateRandomPassword();
            log.LogInformation("New Password Generated");

            //Add secret version with new password to Key Vault
            CreateNewSecretVersion(client, secret, randomPassword);
            log.LogInformation("New Secret Version Generated");

            //Update Service Provider with new password
            UpdateServicePassword(secret, randomPassword);
            log.LogInformation("Password Changed");
            log.LogInformation($"Secret Rotated Successfully");
        }
}
```
Полный пример кода можно найти на сайте [GitHub](https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp).

## <a name="add-the-secret-to-key-vault"></a>Добавление секрета в хранилище ключей
Настройте политику доступа, предоставив пользователям разрешение на *управление секретами*.

```azurecli
az keyvault set-policy --upn <email-address-of-user> --name akvrotation-kv --secret-permissions set delete get list
```

Создайте секрет с тегами, содержащими идентификатор ресурса SQL Server, имя пользователя SQL Server и срок действия секрета в днях. Укажите имя секрета, начальный пароль из базы данных SQL (в нашем примере — Simple123) и дату окончания срока действия, которая назначена на завтра.

```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddThh:mm:ssZ")
az keyvault secret set --name sqlPassword --vault-name akvrotation-kv --value "Simple123" --tags "CredentialId=sqlAdmin" "ProviderAddress=<sql-database-resource-id>" "ValidityPeriodDays=90" --expires $tomorrowDate
```

При создании секрета с коротким сроком действия через 15 минут публикуется событие `SecretNearExpiry`, которое вызывает функцию для смены секрета.

## <a name="test-and-verify"></a>Тестирование и проверка

Чтобы убедиться в ротации секрета, перейдите в раздел **Key Vault** > **Секреты**:

:::image type="content" source="../media/rotate-8.png" alt-text="Снимок экрана: как выбрать [хранилище ключей] > &quot;Секреты&quot;.":::

Откройте секрет **sqlPassword** и просмотрите исходную и измененную версию.

:::image type="content" source="../media/rotate-9.png" alt-text="Переход к секретам":::

### <a name="create-a-web-app"></a>Создание веб-приложения

Чтобы проверить учетные данные SQL, создайте веб-приложение. Это веб-приложение получит секрет из Key Vault, извлечет из секрета сведения о базе данных и учетные данные SQL, а также выполнит проверку подключения к SQL.

Для веб-приложения требуются следующие компоненты:
- веб-приложение с управляемым системой удостоверением;
- политика доступа для получения доступа к секретам в хранилище ключей с использованием управляемого удостоверения веб-приложения.

1. Щелкните ссылку для развертывания шаблона в Azure: 

   [![Изображение с кнопкой "Развернуть в Azure".](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2FKeyVault-Rotation-SQLPassword-Csharp-WebApp%2Fmain%2FARM-Templates%2FWeb-App%2Fazuredeploy.json)

1. Выберите группу ресурсов **akvrotation**.
1. В поле **Sql Server Name** (Имя SQL Server) введите имя SQL Server и пароль для смены.
1. В поле **Имя Key Vault** введите имя хранилища ключей.
1. В поле **Secret Name** (Имя секрета) введите имя секрета, где будет храниться пароль.
1. В поле **Repo Url** (URL-адрес репозитория) введите расположение кода для веб-приложения на сайте GitHub ( **https://github.com/Azure-Samples/KeyVault-Rotation-SQLPassword-Csharp-WebApp.git** ).
1. Выберите **Review + create** (Просмотреть и создать).
1. Нажмите кнопку **создания**.


### <a name="open-the-web-app"></a>Открытие веб-приложения

Выберите URL-адрес развернутого приложения.
 
'https://akvrotation-app.azurewebsites.net/'

Когда приложение откроется в браузере, вы увидите **созданное секретное значение** и **подключенную базу данных** со значением *true*.

## <a name="learn-more"></a>Дополнительные сведения

- Руководство по [смене учетных данных для ресурсов, которые используют два набора учетных данных](tutorial-rotation-dual.md)
- Общие сведения. [Мониторинг Key Vault с помощью Сетки событий Azure](../general/event-grid-overview.md)
- Руководство. [Получение сообщения электронной почты при изменении секрета в хранилище ключей](../general/event-grid-logicapps.md)
- [Схема событий Сетки событий Azure для Azure Key Vault](../../event-grid/event-schema-key-vault.md)
