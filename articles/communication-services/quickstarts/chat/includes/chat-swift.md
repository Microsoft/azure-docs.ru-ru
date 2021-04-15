---
title: включить файл
description: включить файл
services: azure-communication-services
author: mikben
manager: mikben
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/10/2021
ms.topic: include
ms.custom: include file
ms.author: mikben
ms.openlocfilehash: 30dcda4d7bb95ac59add104452415a0ddfc3c016
ms.sourcegitcommit: d23602c57d797fb89a470288fcf94c63546b1314
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106178537"
---
[!INCLUDE [Public Preview Notice](../../../includes/public-preview-include-chat.md)]

## <a name="prerequisites"></a>Предварительные требования
Перед началом работы нужно сделать следующее:

- Создайте учетную запись Azure с активной подпиской. Дополнительные сведения см. на странице [Создайте бесплатную учетную запись Azure уже сегодня](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Установите [Xcode](https://developer.apple.com/xcode/) и [Cocoapods](https://cocoapods.org/). Мы будем использовать Xcode для создания приложения iOS для работы с кратким руководством, а Cocoapods — для установки зависимостей.
- Создайте ресурс Служб коммуникации Azure. Дополнительные сведения см. в статье [Краткое руководство по созданию ресурсов Служб коммуникации и управлению ими](../../create-communication-resource.md). Для работы с этим кратким руководством необходимо записать **конечную точку**.
- Создайте **двух** пользователей ACS и выдайте им маркер доступа пользователя (подробнее о [маркере доступа пользователя](../../access-tokens.md)). Обязательно задайте для области значение **chat** и **запишите строку маркера, а также строку userId**. При работе с этим кратким руководством мы создадим поток с первым участником, а затем добавим в поток второго участника.

## <a name="setting-up"></a>Настройка

### <a name="create-a-new-ios-application"></a>Создание нового приложения iOS

Запустите Xcode и щелкните `Create a new Xcode project` (Создать новый проект Xcode).

В следующем окне выберите платформу `iOS` и шаблон `App`.

При выборе параметров введите имя проекта `ChatQuickstart`. Выберите интерфейс `Storyboard`, жизненный цикл `UIKit App Delegate` и язык `Swift`.

Нажмите кнопку Next (Далее) и выберите каталог, в котором нужно создать проект.

### <a name="install-the-libraries"></a>Установка библиотек

Мы будем использовать Cocoapods для установки необходимых зависимостей Служб коммуникации.

В командной строке перейдите в корневой каталог проекта iOS `ChatQuickstart`.

Создайте Podfile: `pod init`.

Откройте Podfile и добавьте следующие зависимости в целевой объект `ChatQuickstart`:
```
pod 'AzureCommunication', '~> 1.0.0-beta.9'
pod 'AzureCommunicationChat', '~> 1.0.0-beta.9'
```

Установите зависимости. Кроме того, будет создана рабочая область Xcode: `pod install`.

**После выполнения установки Pod повторно откройте проект в Xcode, выбрав только что созданный `.xcworkspace`.**

### <a name="setup-the-placeholders"></a>Настройка заполнителей

Откройте файл рабочей области `ChatQuickstart.xcworkspace` в Xcode, затем откройте `ViewController.swift`.

В рамках этого краткого руководства мы добавим наш код в класс `viewController` и просмотрим выходные данные в консоли Xcode. В этом кратком руководстве не рассматривается создание пользовательского интерфейса в iOS. 

Импортируйте библиотеки `AzureCommunication` и `AzureCommunicatonChat` в начало файла `viewController.swift`:

```
import AzureCommunication
import AzureCommunicationChat
```

Скопируйте следующий код в метод `viewDidLoad()` класса `ViewController`:

```
override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.

        let semaphore = DispatchSemaphore(value: 0)
        DispatchQueue.global(qos: .background).async {
            do {
                // <CREATE A CHAT CLIENT>
                
                // <CREATE A CHAT THREAD>
                
                // <CREATE A CHAT THREAD CLIENT>
                
                // <SEND A MESSAGE>
                
                // <ADD A USER>
                
                // <LIST USERS>
                
                // <REMOVE A USER>
            } catch {
                print("Quickstart failed: \(error.localizedDescription)")
            }
        }
    }
```

В демонстрационных целях мы будем использовать семафор для синхронизации нашего кода. В следующих шагах мы заменим заполнители примером кода с помощью библиотеки чата Служб коммуникации Azure.


### <a name="create-a-chat-client"></a>Создание клиента чата

Замените комментарий `<CREATE A CHAT CLIENT>` следующим фрагментом кода:

```
let endpoint = "<ACS_RESOURCE_ENDPOINT>"
    let credential =
    try CommunicationTokenCredential(
        token: "<ACCESS_TOKEN>"
    )
    let options = AzureCommunicationChatClientOptions()

    let chatClient = try ChatClient(
        endpoint: endpoint,
        credential: credential,
        withOptions: options
    )
```

Замените `<ACS_RESOURCE_ENDPOINT>` конечной точкой вашего ресурса ACS.
Замените `<ACCESS_TOKEN>` допустимым маркером доступа ACS.

В этом кратком руководстве не рассматривается создание уровня служб для управления маркерами для приложения чата, хотя это и рекомендуется. Дополнительные сведения см. в этой документации об [архитектуре чатов](../../../concepts/chat/concepts.md)

Дополнительные сведения о маркерах доступа пользователей см. [здесь](../../access-tokens.md).

## <a name="object-model"></a>Объектная модель 
Следующие классы и интерфейсы реализуют некоторые основные функции пакета SDK для чата Служб коммуникации для JavaScript.

| Имя                                   | Описание                                                                                                                                                                           |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ChatClient | Этот класс требуется для реализации всех функций чата. Его можно создать с помощью сведений о подписке и использовать для создания, получения и удаления потоков. |
| ChatThreadClient | Этот класс требуется для реализации всех функций потоков чата. Вы получаете экземпляр с помощью ChatClient и используете его для отправки, получения, обновления и удаления сообщений, добавления, удаления и получения пользователей, отправки уведомлений о вводе и уведомления о прочтении и подписки на чат. |

## <a name="start-a-chat-thread"></a>Запуск потока чата

Теперь мы воспользуемся `ChatClient`, чтобы создать новый поток с исходным пользователем.

Замените комментарий `<CREATE A CHAT THREAD>` следующим фрагментом кода:

```
let request = CreateThreadRequest(
    topic: "Quickstart",
    participants: [
        Participant(
            id: CommunicationUserIdentifier("<USER_ID>"),
            displayName: "Jack"
        )
    ]
)

var threadId: String?
chatClient.create(thread: request) { result, _ in
    switch result {
    case let .success(result):
        threadId = result.thread?.id

    case .failure:
        fatalError("Failed to create thread.")
    }
    semaphore.signal()
}
semaphore.wait()
```

Замените `<USER_ID>` допустимым идентификатором пользователя Служб коммуникации.

Мы используем здесь семафор, чтобы дождаться завершающего обработчика, прежде чем продолжить. Мы будем использовать `threadId` из ответа, возвращаемого завершающему обработчику в последующих шагах.

## <a name="get-a-chat-thread-client"></a>Получение клиента потока чата

Теперь, когда мы создали поток чата, мы получим `ChatThreadClient` для выполнения операций в потоке.

Замените комментарий `<CREATE A CHAT THREAD CLIENT>` следующим фрагментом кода:

```
let chatThreadClient = try chatClient.createClient(forThread: threadId!)
```

## <a name="send-a-message-to-a-chat-thread"></a>Отправка сообщения в поток чата

Замените комментарий `<SEND A MESSAGE>` следующим фрагментом кода:

```
let message = SendChatMessageRequest(
    content: "Hello!",
    senderDisplayName: "Jack"
)

chatThreadClient.send(message: message) { result, _ in
    switch result {
    case let .success(result):
        print("Message sent, message id: \(result.id)")
    case .failure:
        print("Failed to send message")
    }
    semaphore.signal()
}
semaphore.wait()
```

Сначала мы создадим интерфейс `SendChatMessageRequest`, который включает содержимое и отображаемые имена отправителей (также может включать время общего доступа к журналу). Ответ, возвращенный завершающему обработчику, содержит идентификатор отправленного сообщения.

## <a name="add-a-user-as-a-participant-to-the-chat-thread"></a>Добавление пользователя в качестве участника в поток чата

Замените комментарий `<ADD A USER>` следующим фрагментом кода:

```
let user = Participant(
    id: CommunicationUserIdentifier("<USER_ID>"),
    displayName: "Jane"
)

chatThreadClient.add(participants: [user]) { result, _ in
    switch result {
    case let .success(result):
        (result.errors != nil) ? print("Added participant") : print("Error adding participant")
    case .failure:
        print("Failed to list participants")
    }
    semaphore.signal()
}
semaphore.wait()
```

Замените `<USER_ID>` идентификатором пользователя ACS, которого нужно добавить.

При добавлении участника в поток возвращенный ответ может содержать ошибки. Эти ошибки вызваны сбоем при добавлении определенного участника.

## <a name="list-users-in-a-thread"></a>Вывод списка пользователей в потоке

Замените комментарий `<LIST USERS>` следующим кодом:

```
chatThreadClient.listParticipants { result, _ in
    switch result {
    case let .success(participants):
        var iterator = participants.syncIterator
        while let participant = iterator.next() {
            let user = participant.id as! CommunicationUserIdentifier
            print(user.identifier)
        }
    case .failure:
        print("Failed to list participants")
    }
    semaphore.signal()
}
semaphore.wait()
```


## <a name="remove-user-from-a-chat-thread"></a>Удаление пользователя из потока чата

Замените комментарий `<REMOVE A USER>` следующим кодом:

```
chatThreadClient
    .remove(
        participant: CommunicationUserIdentifier("<USER_ID>")
    ) { result, _ in
        switch result {
        case .success:
            print("Removed user from the thread.")
        case .failure:
            print("Failed to remove user from the thread.")
        }
    }
```

Замените `<USER ID>` идентификатором пользователя Служб коммуникации удаляемого участника.

## <a name="run-the-code"></a>Выполнение кода

В Xcode нажмите кнопку Run (Выполнить), чтобы выполнить сборку и запуск проекта. В консоли можно просмотреть выходные данные кода и средства ведения журнала из ChatClient.

