---
author: baanders
description: включить файл для Дефаултазурекредентиал в примерах цифровых двойников Azure — Примечание
ms.service: digital-twins
ms.topic: include
ms.date: 10/22/2020
ms.author: baanders
ms.openlocfilehash: bec2681c33108417aab1a33932c4f5f4d2ed730f
ms.sourcegitcommit: 6386854467e74d0745c281cc53621af3bb201920
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102473745"
---
>[!NOTE]
> В этом примере используется [DefaultAzureCredential](/dotnet/api/azure.identity.defaultazurecredential) (часть библиотеки `Azure.Identity`) для аутентификации пользователей с помощью экземпляра Azure Digital Twins, запускаемого на локальном компьютере. При таком типе проверки подлинности пример будет искать учетные данные Azure в локальной среде, такие как имя входа из локального [Azure CLI](/cli/azure/install-azure-cli) или в Visual Studio/Visual Studio Code.
>
> Дополнительные сведения об использовании `DefaultAzureCredential` и других вариантах проверки подлинности см. [*в разделе Практические руководства. Написание кода проверки подлинности приложения*](../articles/digital-twins/how-to-authenticate-client.md).