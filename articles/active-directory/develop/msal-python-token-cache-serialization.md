---
title: Сериализация кэша настраиваемых маркеров (MSAL Python) | Службы
titleSuffix: Microsoft identity platform
description: Сведения о сериализации кэша маркеров для MSAL для Python
services: active-directory
author: rayluo
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 11/13/2019
ms.author: rayluo
ms.reviewer: nacanuma
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 7aaf59377949101ba8dd5c9454b89229e925e859
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87846196"
---
# <a name="custom-token-cache-serialization-in-msal-for-python"></a>Сериализация кэша пользовательской лексемы в MSAL для Python

В MSAL Python кэш маркеров в памяти, который сохраняется в течение сеанса приложения, предоставляется по умолчанию при создании экземпляра [клиентаппликатион](https://msal-python.readthedocs.io/en/latest/#confidentialclientapplication).

Сериализация кэша маркеров, чтобы различные сеансы приложения могли получить к нему доступ, не предоставляется "из этого поля". Это обусловлено тем, что MSAL Python можно использовать в типах приложений, у которых нет доступа к файловой системе, таким как веб-приложения. Чтобы создать устойчивый кэш маркеров в приложении Python MSAL, необходимо указать сериализацию кэша пользовательской лексемы.

Стратегии сериализации кэша маркеров зависят от того, пишется ли общедоступное клиентское приложение (Настольный) или конфиденциальное клиентское приложение (веб-приложение, веб-API или приложение управляющей программы).

## <a name="token-cache-for-a-public-client-application"></a>Кэш маркеров для общедоступного клиентского приложения

Общедоступные клиентские приложения выполняются на устройстве пользователя и управляют маркерами для одного пользователя. В этом случае можно сериализовать весь кэш в файл. Не забывайте предоставлять блокировку файлов, если приложение или другое приложение может обращаться к кэшу параллельно. Простой пример сериализации кэша токена в файл без блокировки см. в примере в справочной документации по классу [сериализаблетокенкаче](https://msal-python.readthedocs.io/en/latest/#msal.SerializableTokenCache) .

## <a name="token-cache-for-a-web-app-confidential-client-application"></a>Кэш маркеров для веб-приложения (конфиденциальное клиентское приложение)

Для веб-приложений или веб-API можно использовать сеанс или кэш Redis или базу данных для хранения кэша маркеров. Должен существовать один кэш маркеров на пользователя (на учетную запись), чтобы обеспечить сериализацию кэша маркеров на учетную запись.

## <a name="next-steps"></a>Дальнейшие действия

Пример использования кэша маркеров для веб-приложения Windows или Linux или веб-API см. в статье [MS-Identity-Python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp/blob/master/app.py#L64-L72) . Этот пример предназначен для веб-приложения, которое вызывает API Microsoft Graph.
