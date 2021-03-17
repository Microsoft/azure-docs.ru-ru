---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 12/28/2020
ms.author: tamram
ms.openlocfilehash: a9d7f4f77d91abc88ea348e71a3d9c471b26a273
ms.sourcegitcommit: 87a6587e1a0e242c2cfbbc51103e19ec47b49910
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103622257"
---
Эмулятор поддерживает одну фиксированную учетную запись и известный ключ проверки подлинности для аутентификации с общим ключом. Эта учетная запись и ключ являются единственными учетными данными общего ключа, разрешенными для использования с эмулятором. Их можно сформулировать следующим образом.

```
Account name: devstoreaccount1
Account key: Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==
```

> [!NOTE]
> Ключ проверки подлинности, поддерживаемый эмулятором, предназначен только для тестирования функциональности кода проверки подлинности клиента. Его использование не гарантирует защиту. Вы не можете использовать рабочую учетную запись хранения и ключ в эмуляторе. Не следует использовать учетную запись для разработки с рабочими данными.
>
> Эмулятор поддерживает подключение только по протоколу HTTP. Тем не менее для доступа к ресурсам в рабочей учетной записи хранения Azure рекомендуется использовать протокол HTTPS.
>

#### <a name="connect-to-the-emulator-account-using-the-shortcut"></a>Подключение к учетной записи эмулятора с помощью ярлыка

Самый простой способ подключиться к эмулятору из приложения — настроить строку подключения в файле конфигурации приложения, который ссылается на ярлык `UseDevelopmentStorage=true` . Сочетание клавиш эквивалентно полной строке подключения для эмулятора, которая указывает имя учетной записи, ключ учетной записи и конечные точки эмулятора для каждой из служб хранилища Azure:

```
DefaultEndpointsProtocol=http;AccountName=devstoreaccount1;
AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;
BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;
TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;
```

В следующем фрагменте кода .NET показано, как можно использовать сочетание клавиш из метода, принимающего строку подключения. Например, конструктор [блобконтаинерклиент (строка, строка)](/dotnet/api/azure.storage.blobs.blobcontainerclient.-ctor#Azure_Storage_Blobs_BlobContainerClient__ctor_System_String_System_String_) принимает строку подключения.

```csharp
BlobContainerClient blobContainerClient = new BlobContainerClient("UseDevelopmentStorage=true", "sample-container");
blobContainerClient.CreateIfNotExists();
```

Перед вызовом кода во фрагменте убедитесь, что эмулятор работает.
