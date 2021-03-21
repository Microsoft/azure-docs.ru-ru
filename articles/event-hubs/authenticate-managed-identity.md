---
title: Проверка подлинности управляемого удостоверения с помощью Azure Active Directory
description: В этой статье содержатся сведения о проверке подлинности управляемого удостоверения с Azure Active Directory для доступа к ресурсам концентраторов событий Azure.
ms.topic: conceptual
ms.date: 01/25/2021
ms.custom: devx-track-csharp
ms.openlocfilehash: 2070cfd94b39a08afb86ffd3579f1116faac72d5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98805290"
---
# <a name="authenticate-a-managed-identity-with-azure-active-directory-to-access-event-hubs-resources"></a>Проверка подлинности управляемого удостоверения с Azure Active Directory для доступа к ресурсам концентраторов событий
Концентраторы событий Azure поддерживают проверку подлинности Azure Active Directory (Azure AD) с помощью [управляемых удостоверений для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md). Управляемые удостоверения для ресурсов Azure могут авторизовать доступ к ресурсам концентраторов событий с помощью учетных данных Azure AD из приложений, работающих на виртуальных машинах Azure, приложений-функций, масштабируемых наборов виртуальных машин и других служб. Используя управляемые удостоверения для ресурсов Azure вместе с проверкой подлинности Azure AD, можно избежать хранения учетных данных в приложениях, выполняемых в облаке.

В этой статье показано, как авторизовать доступ к концентратору событий с помощью управляемого удостоверения на виртуальной машине Azure.

## <a name="enable-managed-identities-on-a-vm"></a>Включение управляемых удостоверений на виртуальной машине
Прежде чем вы сможете использовать управляемые удостоверения для ресурсов Azure для авторизации ресурсов концентраторов событий из виртуальной машины, сначала необходимо включить управляемые удостоверения для ресурсов Azure на виртуальной машине. Сведения о включении управляемых удостоверений для ресурсов Azure см. в одной из следующих статей.

- [Портал Azure](../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [Azure CLI](../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Шаблон Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [Клиентские библиотеки Azure Resource Manager](../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)

## <a name="grant-permissions-to-a-managed-identity-in-azure-ad"></a>Предоставление разрешений управляемому удостоверению в Azure AD
Чтобы авторизовать запрос к службе концентраторов событий из управляемого удостоверения в приложении, сначала настройте параметры управления доступом на основе ролей Azure (Azure RBAC) для этого управляемого удостоверения. Концентраторы событий Azure определяют роли Azure, которые охватывают разрешения для отправки и чтения из концентраторов событий. Когда роль Azure назначается управляемому удостоверению, управляемому удостоверению предоставляется доступ к данным концентраторов событий в соответствующей области.

Дополнительные сведения о назначении ролей Azure см. [в статье аутентификация с помощью Azure Active Directory для доступа к ресурсам концентраторов событий](authorize-access-azure-active-directory.md).

## <a name="use-event-hubs-with-managed-identities"></a>Использование Центров событий с управляемыми удостоверениями для ресурсов Azure
Чтобы использовать концентраторы событий с управляемыми удостоверениями, необходимо назначить этой роли и соответствующей области идентификатору. Процедура в этом разделе использует простое приложение, которое выполняется под управляемым удостоверением и обращается к ресурсам концентраторов событий.

Здесь мы используем пример веб-приложения, размещенного в [службе приложений Azure](https://azure.microsoft.com/services/app-service/). Пошаговые инструкции по созданию веб-приложения см. [в статье создание ASP.NET Core веб-приложения в Azure](../app-service/quickstart-dotnetcore.md) .

После создания приложения выполните следующие действия. 

1. Перейдите в раздел **Параметры** и выберите **удостоверение**. 
1. Выберите **состояние** **включено**. 
1. Нажмите кнопку **Сохранить**, чтобы сохранить параметры. 

    :::image type="content" source="./media/authenticate-managed-identity/identity-web-app.png" alt-text="Управляемое удостоверение для веб-приложения":::
4. Выберите **Да** в информационном сообщении. 

    После включения этого параметра в Azure Active Directory (Azure AD) создается новое удостоверение службы, которое настраивается в узле службы приложений.

    Теперь назначьте это удостоверение службы роли в требуемой области в ресурсах концентраторов событий.

### <a name="to-assign-azure-roles-using-the-azure-portal"></a>Назначение ролей Azure с помощью портал Azure
Чтобы назначить роль ресурсам концентраторов событий, перейдите к этому ресурсу в портал Azure. Отобразить параметры управления доступом (IAM) для ресурса и выполнить следующие инструкции для управления назначениями ролей:

> [!NOTE]
> Следующие шаги назначают роль удостоверения службы для пространств имен концентраторов событий. Вы можете выполнить те же действия, чтобы назначить роль для любого ресурса концентраторов событий. 

1. В портал Azure перейдите к пространству имен концентраторов событий и просмотрите **Общие сведения** о пространстве имен. 
1. Выберите **Управление доступом (IAM)** в меню слева, чтобы отобразить параметры управления доступом для концентратора событий.
1.  Выберите вкладку **Назначения ролей**, чтобы просмотреть список назначений ролей.
3.  Нажмите кнопку **Добавить**, а затем выберите * * добавить назначение ролей * * *.
4.  На странице **Добавление назначения ролей** выполните следующие действия.
    1. В поле **роль** выберите роль концентраторов событий, которую требуется назначить. В этом примере это **владелец данных концентраторов событий Azure**.
    1. В поле **назначение доступа к** выберите **служба приложений** в разделе **назначенное системой управляемое удостоверение**. 
    1. Выберите **подписку** , в которой было создано управляемое удостоверение для веб-приложения.
    1. Выберите **управляемое удостоверение** для созданного веб-приложения. Имя по умолчанию для удостоверения совпадает с именем веб-приложения. 
    1. Затем нажмите кнопку **Сохранить**. 
    
        ![Страница добавления назначения ролей](./media/authenticate-managed-identity/add-role-assignment-page.png)

    После назначения роли веб-приложение будет иметь доступ к ресурсам концентраторов событий в определенной области. 

    > [!NOTE]
    > Список служб, поддерживающих управляемые удостоверения, см. в статье [службы, поддерживающие управляемые удостоверения для ресурсов Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md).

### <a name="test-the-web-application"></a>Тестирование веб-приложения
1. Создайте пространство имен Центров событий и концентратор событий. 
2. Разверните веб-приложение в Azure. Ссылки на веб-приложение на сайте GitHub см. в следующем разделе с вкладками. 
3. Убедитесь, что Сендрецеиве. aspx задан в качестве документа по умолчанию для веб-приложения. 
3. Включите **удостоверение** для веб-приложения. 
4. Назначьте это удостоверение роли **владельца данных концентраторов событий** на уровне пространства имен или концентратора событий. 
5. Запустите веб-приложение, введите имя пространства имен и имя концентратора событий, сообщение и нажмите кнопку **Отправить**. Чтобы получить событие, выберите **получить**. 

#### <a name="azuremessagingeventhubs-latest"></a>[Azure. Messaging. EventHubs (последняя версия)](#tab/latest)
Теперь вы можете запустить веб-приложение и указать в браузере страницу с примером страницы ASPX. Вы можете найти пример веб-приложения, которое отправляет и получает данные из ресурсов концентраторов событий в [репозитории GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp).

Установите последний пакет из [NuGet](https://www.nuget.org/packages/Azure.Messaging.EventHubs/)и приступите к отправке событий в концентраторы событий с помощью **евенсубпродуцерклиент** и получения событий с помощью **евенсубконсумерклиент**. 

> [!NOTE]
> Пример на языке Java, в котором для публикации событий в концентраторе событий используется управляемое удостоверение, см. [в разделе Публикация событий с помощью примера Azure Identity на сайте GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs).

```csharp
protected async void btnSend_Click(object sender, EventArgs e)
{
    await using (EventHubProducerClient producerClient = new EventHubProducerClient(txtNamespace.Text, txtEventHub.Text, new DefaultAzureCredential()))
    {
        // create a batch
        using (EventDataBatch eventBatch = await producerClient.CreateBatchAsync())
        {

            // add events to the batch. only one in this case. 
            eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes(txtData.Text)));

            // send the batch to the event hub
            await producerClient.SendAsync(eventBatch);
        }

        txtOutput.Text = $"{DateTime.Now} - SENT{Environment.NewLine}{txtOutput.Text}";
    }
}
protected async void btnReceive_Click(object sender, EventArgs e)
{
    await using (var consumerClient = new EventHubConsumerClient(EventHubConsumerClient.DefaultConsumerGroupName, $"{txtNamespace.Text}.servicebus.windows.net", txtEventHub.Text, new DefaultAzureCredential()))
    {
        int eventsRead = 0;
        try
        {
            using CancellationTokenSource cancellationSource = new CancellationTokenSource();
            cancellationSource.CancelAfter(TimeSpan.FromSeconds(5));

            await foreach (PartitionEvent partitionEvent in consumerClient.ReadEventsAsync(cancellationSource.Token))
            {
                txtOutput.Text = $"Event Read: { Encoding.UTF8.GetString(partitionEvent.Data.Body.ToArray()) }{ Environment.NewLine}" + txtOutput.Text;
                eventsRead++;
            }
        }
        catch (TaskCanceledException ex)
        {
            txtOutput.Text = $"Number of events read: {eventsRead}{ Environment.NewLine}" + txtOutput.Text;
        }
    }
}
```

#### <a name="microsoftazureeventhubs-legacy"></a>[Microsoft.Azure.EventHubs (устаревшая версия)](#tab/old)
Теперь вы можете запустить веб-приложение и указать в браузере страницу с примером страницы ASPX. Вы можете найти пример веб-приложения, которое отправляет и получает данные из ресурсов концентраторов событий в [репозитории GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac/ManagedIdentityWebApp).

Установите последний пакет из [NuGet](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/)и начните отправлять и получать данные из концентраторов событий с помощью EventHubClient, как показано в следующем коде: 

```csharp
var ehClient = EventHubClient.CreateWithManagedIdentity(new Uri($"sb://{EventHubNamespace}/"), EventHubName);
```
---

## <a name="event-hubs-for-kafka"></a>Центры событий для Kafka
Apache Kafka приложения можно использовать для отправки и получения сообщений из концентраторов событий Azure с помощью управляемого удостоверения OAuth. См. Следующий пример на сайте GitHub: [концентраторы событий для Kafka — отправка и получение сообщений с помощью управляемого удостоверения OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/managedidentity).

## <a name="samples"></a>Примеры
- Примеры **Azure. Messaging. EventHubs**
    - [.NET](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/ManagedIdentityWebApp)
    - [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs/src/samples/java/com/azure/messaging/eventhubs)
- [Примеры Microsoft. Azure. EventHubs](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/Rbac). 
    
    В этих примерах используется библиотека **Microsoft.Azure.EventHubs** старой версии, но вы можете легко обновить ее до библиотеки **Azure.Messaging.EventHubs** последней версии. Чтобы начать использовать в примере библиотеку новой версии вместо старой, ознакомьтесь с руководством по [переходу с Microsoft.Azure.EventHubs на Azure.Messaging.EventHubs](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md).
    Этот пример обновлен для использования последней библиотеки **Azure. Messaging. EventHubs** .
- [Концентраторы событий для Kafka — отправка и получение сообщений с помощью управляемого удостоверения OAuth](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth/java/managedidentity)


## <a name="next-steps"></a>Дальнейшие действия
- Сведения об управляемых удостоверениях для ресурсов Azure см. в следующей статье: [что такое управляемые удостоверения для ресурсов Azure?](../active-directory/managed-identities-azure-resources/overview.md)
- См. следующие статьи:
    - [Проверка подлинности запросов к концентраторам событий Azure из приложения с помощью Azure Active Directory](authenticate-application.md)
    - [Проверка подлинности запросов к концентраторам событий Azure с помощью подписанных URL](authenticate-shared-access-signature.md)
    - [Авторизация доступа к ресурсам концентраторов событий с помощью Azure Active Directory](authorize-access-azure-active-directory.md)
    - [Авторизация доступа к ресурсам концентраторов событий с помощью подписанных URL](authorize-access-shared-access-signature.md)
