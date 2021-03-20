---
title: включить файл
description: включить файл
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 08/24/2020
ms.openlocfilehash: aa8aea66c5481454f0d7d4d118934f5fbf34a911
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89147600"
---
**По умолчанию управляемым удостоверением** является управляемое системой удостоверение или первое назначенное пользователю управляемое удостоверение.

Во время выполнения существует два приложения с удостоверением:

1. Система использует удостоверение для настройки подключений к хранилищу, реестра контейнеров и хранилищ данных пользователя.

    * В этом случае система будет использовать удостоверение, управляемое по умолчанию.

1. Пользователь применяет удостоверение для доступа к ресурсам из кода для отправленного выполнения

    * В этом случае укажите *client_id* , соответствующие управляемому удостоверению, которое вы хотите использовать для получения учетных данных.
    * Кроме того, можно получить идентификатор клиента идентификатора, назначенного пользователем, с помощью переменной среды *DEFAULT_IDENTITY_CLIENT_ID* .

    Например, чтобы получить маркер для хранилища данных с удостоверением, управляемым по умолчанию:

    ```python
    client_id = os.environ.get('DEFAULT_IDENTITY_CLIENT_ID')
    credential = ManagedIdentityCredential(client_id=client_id)
    token = credential.get_token('https://storage.azure.com/')
    ```