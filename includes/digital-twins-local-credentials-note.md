---
author: baanders
description: включить файл для Дефаултазурекредентиал в примерах цифровых двойников Azure — Примечание
ms.service: digital-twins
ms.topic: include
ms.date: 10/22/2020
ms.author: baanders
ms.openlocfilehash: 8aa3cbb2a037e49a694192c9852eeae0215c089c
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102211794"
---
>[!NOTE]
> В этом примере используется [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential?preserve-view=true&view=azure-dotnet) (часть библиотеки `Azure.Identity`) для аутентификации пользователей с помощью экземпляра Azure Digital Twins, запускаемого на локальном компьютере. При таком типе проверки подлинности пример будет искать учетные данные Azure в локальной среде, такие как имя входа из локального [Azure CLI](/cli/azure/install-azure-cli) или в Visual Studio/Visual Studio Code.
>
> Дополнительные сведения об использовании `DefaultAzureCredential` и других вариантах проверки подлинности см. [*в разделе Практические руководства. Написание кода проверки подлинности приложения*](../articles/digital-twins/how-to-authenticate-client.md).