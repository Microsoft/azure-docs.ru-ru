---
title: Краткое руководство. Создание маркеров доступа и управление ими
titleSuffix: An Azure Communication Services quickstart
description: Узнайте, как управлять удостоверениями и маркерами доступа с помощью пакета SDK для удостоверений Служб коммуникации Azure.
author: tomaschladek
manager: nmurav
services: azure-communication-services
ms.author: tchladek
ms.date: 03/10/2021
ms.topic: quickstart
ms.service: azure-communication-services
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: e356219d22ee558ce3de5a96d58f24b9e7902d8a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105726623"
---
# <a name="quickstart-create-and-manage-access-tokens"></a>Краткое руководство. Создание маркеров доступа и управление ими

Начните работу со Службами коммуникации Azure с помощью пакета SDK для удостоверений Служб коммуникации Azure. Она позволяет создавать удостоверения и управлять маркерами доступа. Удостоверение представляет сущность приложения в Службах коммуникации Azure (например, пользователь или устройство). Маркеры доступа позволяют выполнять проверку подлинности в пакетах SDK для чатов и вызовов непосредственно в Службах коммуникации Azure. Рекомендуем создавать маркеры доступа для службы на стороне сервера. Маркеры доступа используются для инициализации пакетов SDK Служб коммуникации на клиентских устройствах.

Все цены, указанные на изображениях в этом руководстве, приведены только для демонстрации.

::: zone pivot="programming-language-csharp"
[!INCLUDE [.NET](./includes/user-access-token-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/user-access-token-js.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/user-access-token-python.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/user-access-token-java.md)]
::: zone-end

В выходных данных приложения описывают каждое выполненное действие.
<!---cSpell:disable --->
```console
Azure Communication Services - Access Tokens Quickstart

Created an identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502

Issued an access token with 'voip' scope that expires at 30/03/21 08:09 09 AM:
<token signature here>

Created an identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-1ce9-31b4-54b7-a43a0d006a52

Issued an access token with 'voip' scope that expires at 30/03/21 08:09 09 AM:
<token signature here>

Successfully revoked all access tokens for identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502

Deleted the identity with ID: 8:acs:4ccc92c8-9815-4422-bddc-ceea181dc774_00000006-19e0-2727-80f5-8b3a0d003502
```
<!---cSpell:enable --->

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы хотите очистить и удалить подписку на Службы коммуникации, вы можете удалить ресурс или группу ресурсов. При этом удаляются все ресурсы, связанные с ней. См. сведения об [очистке ресурсов](./create-communication-resource.md#clean-up-resources).


## <a name="next-steps"></a>Next Steps

В этом кратком руководстве рассматривались следующие темы:

> [!div class="checklist"]
> * Управление идентификаторами
> * Выпуск маркеров доступа
> * Использование пакета SDK для удостоверений Служб коммуникации Azure


> [!div class="nextstepaction"]
> [Добавление функции голосового вызова в приложение](./voice-video-calling/getting-started-with-calling.md)

Полезные ссылки

 - [Сведения о проверке подлинности](../concepts/authentication.md)
 - [Добавление чата в приложение](./chat/get-started.md)
 - [Сведения об архитектуре клиент-сервер](../concepts/client-and-server-architecture.md)
