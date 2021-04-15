---
title: Краткое руководство. Отправка и получение событий с помощью Go в Центрах событий Azure
description: Краткое руководство. В статье описано, как создать приложение Go, которое отправляет события из Центров событий Azure.
ms.topic: quickstart
ms.date: 06/23/2020
ms.openlocfilehash: ededd042dadd6059bd7b2d689cff78fe1bd6af0b
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/13/2021
ms.locfileid: "107311096"
---
# <a name="quickstart-send-events-to-or-receive-events-from-event-hubs-using-go"></a>Краткое руководство. Отправка и получение событий с помощью Go в Центрах событий
Центры событий Azure — это платформа потоковой передачи больших данных и служба приема событий, принимающая и обрабатывающая миллионы событий в секунду. Центры событий могут обрабатывать и сохранять события, данные и телеметрию, созданные распределенным программным обеспечением и устройствами. Данные, отправляемые в концентратор событий, можно преобразовывать и сохранять с помощью любого поставщика аналитики в реальном времени, а также с помощью адаптеров пакетной обработки или хранения. Подробный обзор Центров событий см. в статьях [Что такое Центры событий Azure?](event-hubs-about.md) и [Обзор функций Центров событий](event-hubs-features.md).

В этом руководстве описано, как создавать приложения Go для отправки и получения событий через концентратор событий. 

> [!NOTE]
> Вы можете скачать это краткое руководство в качестве примера с сайта [GitHub](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/eventhubs), заменить строки `EventHubConnectionString` и `EventHubName` значениями для своего концентратора событий и выполнить этот пример. Или следуйте инструкциям из этого руководства, чтобы создать собственное решение.

## <a name="prerequisites"></a>Предварительные требования

Для работы с данным руководством вам потребуется:

- Локально установленный Go. При необходимости выполните [следующие инструкции](https://golang.org/doc/install).
- Активная учетная запись Azure. Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись][], прежде чем начинать работу.
- **Создайте пространство имен Центров событий и концентратор событий**. Вы можете использовать [портал Azure](https://portal.azure.com) для создания пространства имен типа Центров событий и получать учетные данные для управления, требуемые приложению для взаимодействия с концентратором событий. Чтобы создать пространство имен и концентратор событий, выполните инструкции из [этой статьи](event-hubs-create.md).

## <a name="send-events"></a>Отправка событий
Из этого раздела вы узнаете, как создать приложение Go, которое отправляет события в концентратор событий. 

### <a name="install-go-package"></a>Установка пакета Go

Получить пакет Go для Центров событий с помощью `go get` или `dep`. Пример:

```bash
go get -u github.com/Azure/azure-event-hubs-go
go get -u github.com/Azure/azure-amqp-common-go/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
```

### <a name="import-packages-in-your-code-file"></a>Импорт пакетов в файл кода

Для импорта пакетов Go используйте следующий пример кода:

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
)
```

### <a name="create-service-principal"></a>Создание субъекта-службы

Создайте субъект-службу, следуя инструкциям в статье [Создание субъекта-службы Azure с помощью Azure CLI 2.0](/cli/azure/create-an-azure-service-principal-azure-cli). Сохраните предоставленные учетные данные в своей среде со следующими именами. Пакеты Azure SDK для Go и Центров событий предварительно настроенные для поиска этих имен переменных:

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

Теперь создайте поставщик авторизации для клиента Центров событий, использующего эти учетные данные:

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

### <a name="create-event-hubs-client"></a>Создание клиента Центров событий

В следующем коде создается клиент Центров событий:

```go
hub, err := eventhubs.NewHub("namespaceName", "hubName", tokenProvider)
ctx := context.WithTimeout(context.Background(), 10 * time.Second)
defer hub.Close(ctx)
if err != nil {
    log.Fatalf("failed to get hub %s\n", err)
}
```

### <a name="write-code-to-send-messages"></a>Написание кода для отправки сообщений

В следующем фрагменте кода используйте (1) для отправки сообщений из терминала в интерактивном режиме или (2) для отправки сообщений в рамках программы:

```go
// 1. send messages at the terminal
ctx = context.Background()
reader := bufio.NewReader(os.Stdin)
for {
    fmt.Printf("Input a message to send: ")
    text, _ := reader.ReadString('\n')
    hub.Send(ctx, eventhubs.NewEventFromString(text))
}

// 2. send messages within program
ctx = context.Background()
hub.Send(ctx, eventhubs.NewEventFromString("hello Azure!"))
```

### <a name="extras"></a>Дополнительно

Получите идентификаторы каждой секции в концентраторе событий:

```go
info, err := hub.GetRuntimeInformation(ctx)
if err != nil {
    log.Fatalf("failed to get runtime info: %s\n", err)
}
log.Printf("got partition IDs: %s\n", info.PartitionIDs)
```

Запустите приложение и отправьте события в концентратор событий. 

Поздравляем! Теперь вы можете отправлять сообщения в концентратор событий.

## <a name="receive-events"></a>Получение событий

### <a name="create-a-storage-account-and-container"></a>Создание учетной записи хранения и контейнера

Такие состояния, как аренда разделов и контрольные точки в потоке событий, совместно используются получателями через контейнер службы хранилища Azure. Вы можете создать учетную запись хранения и контейнер с помощью пакета SDK для Go, но их также можно создать, следуя инструкциям в статье [Об учетных записях хранения Azure](../storage/common/storage-account-create.md).

Примеры для создания артефактов хранилища с помощью пакета SDK для Go доступны в [репозитории примеров Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/storage) и в примере, прилагаемом к этому руководству.

### <a name="go-packages"></a>Пакеты Go

Чтобы получать сообщения, получите пакеты Go для концентраторов событий с помощью `go get` или `dep`:

```bash
go get -u github.com/Azure/azure-event-hubs-go/...
go get -u github.com/Azure/azure-amqp-common-go/...
go get -u github.com/Azure/go-autorest/...

# or

dep ensure -add github.com/Azure/azure-event-hubs-go
dep ensure -add github.com/Azure/azure-amqp-common-go
dep ensure -add github.com/Azure/go-autorest
```

### <a name="import-packages-in-your-code-file"></a>Импорт пакетов в файл кода

Для импорта пакетов Go используйте следующий пример кода:

```go
import (
    aad "github.com/Azure/azure-amqp-common-go/aad"
    eventhubs "github.com/Azure/azure-event-hubs-go"
    eph "github.com/Azure/azure-event-hubs-go/eph"
    storageLeaser "github.com/Azure/azure-event-hubs-go/storage"
    azure "github.com/Azure/go-autorest/autorest/azure"
)
```

### <a name="create-service-principal"></a>Создание субъекта-службы

Создайте субъект-службу, следуя инструкциям в статье [Создание субъекта-службы Azure с помощью Azure CLI 2.0](/cli/azure/create-an-azure-service-principal-azure-cli). Сохраните предоставленные учетные данные в своей среде с приведенными ниже именами. Пакет Azure SDK для Go и пакет Центров событий предварительно сконфигурированы для поиска имен этих переменных.

```bash
export AZURE_CLIENT_ID=
export AZURE_CLIENT_SECRET=
export AZURE_TENANT_ID=
export AZURE_SUBSCRIPTION_ID= 
```

Затем создайте поставщика авторизации для вашего клиента концентраторов событий, который использует эти учетные данные:

```go
tokenProvider, err := aad.NewJWTProvider(aad.JWTProviderWithEnvironmentVars())
if err != nil {
    log.Fatalf("failed to configure AAD JWT provider: %s\n", err)
}
```

### <a name="get-metadata-struct"></a>Получение структуры метаданных

Получите структуру с метаданными о вашей среде Azure, используя пакет Azure SDK для Go. Последующие операции используют эту структуру для поиска нужных конечных точек.

```go
azureEnv, err := azure.EnvironmentFromName("AzurePublicCloud")
if err != nil {
    log.Fatalf("could not get azure.Environment struct: %s\n", err)
}
```

### <a name="create-credential-helper"></a>Создание вспомогательного приложения для учетных данных 

Создайте вспомогательное приложение, которое использует предыдущие учетные данные Azure Active Directory (AAD) для создания подписанного URL-адреса (SAS) для службы хранилища. Последний параметр предписывает конструктору использовать такие же переменные среды, как и ранее:

```go
cred, err := storageLeaser.NewAADSASCredential(
    subscriptionID,
    resourceGroupName,
    storageAccountName,
    storageContainerName,
    storageLeaser.AADSASCredentialWithEnvironmentVars())
if err != nil {
    log.Fatalf("could not prepare a storage credential: %s\n", err)
}
```

### <a name="create-a-check-pointer-and-a-leaser"></a>Создание средства создания конечных точек и средства выдачи аренд 

Создайте **средство выдачи аренд**, ответственное за присвоение аренды на раздел для конкретного получателя, и **средство создания конечных точек**, ответственное за запись контрольных точек для потока сообщений, чтобы другие получатели могли начать чтение из правильной позиции смещения.

В настоящее время доступно одно **StorageLeaserCheckpointer**. Оно использует один контейнер хранилища для управления арендой и контрольными точками. Помимо имен учетной записи и контейнера, средству **StorageLeaserCheckpointer** требуются учетные данные, созданные на предыдущем шаге, и структура среды Azure для правильного доступа к контейнеру.

```go
leaserCheckpointer, err := storageLeaser.NewStorageLeaserCheckpointer(
    cred,
    storageAccountName,
    storageContainerName,
    azureEnv)
if err != nil {
    log.Fatalf("could not prepare a storage leaserCheckpointer: %s\n", err)
}
```

### <a name="construct-event-processor-host"></a>Создание узла обработчика событий

Теперь у вас есть компоненты для создания узла обработчика событий следующим образом. Одно и то же средство **StorageLeaserCheckpointer** используется в качестве средства выдачи аренд и средства создания контрольных точек, как описано ранее:

```go
ctx := context.Background()
p, err := eph.New(
    ctx,
    nsName,
    hubName,
    tokenProvider,
    leaserCheckpointer,
    leaserCheckpointer)
if err != nil {
    log.Fatalf("failed to create EPH: %s\n", err)
}
defer p.Close(context.Background())
```

### <a name="create-handler"></a>Создание обработчика 

Теперь создайте обработчик и зарегистрируйте его в узле обработчика событий. Когда узел запущен, он применяет его и любые другие указанные обработчики ко входящим сообщениям:

```go
handler := func(ctx context.Context, event *eventhubs.Event) error {
    fmt.Printf("received: %s\n", string(event.Data))
    return nil
}

// register the handler with the EPH
_, err := p.RegisterHandler(ctx, handler)
if err != nil {
    log.Fatalf("failed to register handler: %s\n", err)
}
```

### <a name="write-code-to-receive-messages"></a>Написание кода для получения сообщений

Когда все будет настроено, можно запустить узел обработчика событий с `Start(context)`, чтобы он постоянно работал, или с `StartNonBlocking(context)` для работы только до тех пор, пока доступны сообщения.

В этом руководстве запуск и выполнение происходит следующим образом. (Ознакомьтесь с примером использования `StartNonBlocking` на GitHub).

```go
ctx := context.Background()
err = p.Start()
if err != nil {
    log.Fatalf("failed to start EPH: %s\n", err)
}
```

## <a name="next-steps"></a>Дальнейшие действия
Ознакомьтесь со следующими статьями:

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Функции и терминология в Центрах событий Azure](event-hubs-features.md)
- [Часто задаваемые вопросы о Центрах событий](event-hubs-faq.yml)


<!-- Links -->
[Event Hubs overview]: event-hubs-about.md
[бесплатная учетная запись]: https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio
