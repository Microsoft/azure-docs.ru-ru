---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 09/26/2019
ms.author: glenga
ms.openlocfilehash: 4ace70abe0112e0fe27d177c02bcb697746c92cc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78262556"
---
### <a name="set-the-storage-account-connection"></a>Установка подключения к учетной записи хранилища

Откройте файл local.settings.json и скопируйте значение `AzureWebJobsStorage`, которое является строкой подключения к учетной записи хранилища. В качестве значения переменной среды `AZURE_STORAGE_CONNECTION_STRING` установите значение строки подключения, использовав эту команду Bash:

```bash
AZURE_STORAGE_CONNECTION_STRING="<STORAGE_CONNECTION_STRING>"
```

Когда значение строки подключения будет установлено вместо переменной среды `AZURE_STORAGE_CONNECTION_STRING`, вы можете получить доступ к учетной записи хранения без необходимости проходить проверку подлинности каждый раз.