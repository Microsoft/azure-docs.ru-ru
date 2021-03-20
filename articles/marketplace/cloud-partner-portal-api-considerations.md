---
title: Рекомендации по API — Azure Marketplace
description: Управление версиями, обработка ошибок и авторизации проблем благодаря использованию интерфейсов API Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
author: mingshen-ms
ms.author: mingshen
ms.date: 07/14/2020
ms.openlocfilehash: 0920bec160874f27c8b1b6f2132951b57719b31c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88035286"
---
# <a name="api-considerations"></a>Важные аспекты API

<a name="api-versioning"></a>Управление версиями API
--------------

> [!NOTE]
> Портал Cloud Partner API интегрированы с и будут продолжать работать в центре партнеров. Переход содержит небольшие изменения. Ознакомьтесь с изменениями, приведенными в [справочнике по портал Cloud Partner API](./cloud-partner-portal-api-overview.md) , чтобы убедиться, что код продолжит работать после перехода в центр партнеров. API-интерфейсы CPP следует использовать только для существующих продуктов, которые уже были интегрированы до перехода в центр партнеров. новые продукты должны использовать API-интерфейсы отправки центра партнеров.

Несколько версий API может бить доступно в одно и тоже время. Клиентам необходимо указать, какую версию, они будут использовать, предоставляя параметр `api-version` как часть строки запроса.

   `GET https://cloudpartner.azure.com/api/offerTypes?api-version=2017-10-31`

Ошибка HTTP с кодом 400 — это ответ на запрос неизвестной или недопустимой версии API. Эта ошибка возвращает коллекцию известных версий API в текст ответа.

``` json
    {
        "error": { 
            "code":"InvalidAPIVersion",
            "message":"Invalid api version. Allowed values are [2016-08-01-preview]"
        }
    }
```            

<a name="errors"></a>ошибки
------

API отвечает на ошибки с соответствующими кодами состояния HTTP и, необязательно, с дополнительной информацией в ответе, сериализованном как JSON.
При получении ошибки, особенно класса 400, не повторяйте запрос не исправив основную проблему. Например в ответе примера выше, перед повторной отправкой запроса, исправьте параметр версии API.

<a name="authorization-header"></a>Заголовок авторизации
--------------------

Для всех интерфейсов API в этом руководстве необходимо передать заголовок авторизации вместе с маркером носителя, полученного из Azure Active Directory (Azure AD). Этот заголовок необходим для получения допустимого ответа, если он отсутствует, ошибка `401 Unauthorized` возвращается. 

``` HTTP
  GET https://cloudpartner.azure.com/api/offerTypes?api-version=2016-08-01-preview

    Accept: application/json 
    Authorization: Bearer <YOUR_TOKEN> 
```
