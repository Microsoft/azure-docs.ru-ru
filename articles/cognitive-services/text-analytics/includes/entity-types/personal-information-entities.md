---
title: Именованные сущности для персональных данных
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 03/15/2021
ms.author: aahi
ms.openlocfilehash: 19586c09cca9a0dc74ba9ee4ef9da459964f9b7e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104599330"
---
> [!NOTE]
> Чтобы определить защищенную информацию о работоспособности (фи), используйте `domain=phi` параметр и версию модели `2020-04-01` или более поздней версии.
>
> Пример: `https://<your-custom-subdomain>.cognitiveservices.azure.com/text/analytics/v3.1-preview.3/entities/recognition/pii?domain=phi&model-version=2021-01-15`
 
При отправке запросов в конечную точку возвращаются следующие категории сущностей `/v3.1-preview.3/entities/recognition/pii` .


| Категория   |  Описание                          |
|------------|-------------|
| [Person](#category-person)      |  Имена людей.  |
| [персонтипе](#category-persontype) | Типы заданий или роли, удерживаемые человеком. |
| [Номер телефона](#category-phonenumber) |Номера телефонов (только для телефонных номеров США и ЕС). |
| [Организация](#category-organization) |  Компании, группы, государственные органы и другие организации.  |
| [Адрес](#category-address) | Полные адреса электронной почты.  |
| [Электронная почта](#category-email) | Адреса электронной почты.   |
| [URL-адрес](#category-url) | URL-адреса веб-сайтов.  |
| [СМ](#category-ip) | Сетевые IP-адреса.  |
| [DateTime](#category-datetime) | Даты и время суток. | 
| [Количество](#category-quantity) | Числа и числовые величины.  |
| [Сведения об Azure](#azure-information) | Идентифицируемые сведения об Azure, например сведения о проверке подлинности.  |
| [Идентификация](#identification) | Финансовая и региональная идентификация.  |

### <a name="category-person"></a>Категория: Person

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Человек

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Имена людей. 

        Чтобы получить эту категорию сущностей, добавьте `Person` к `pii-categories` параметру. `Person` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`   
      
   :::column-end:::
:::row-end:::

### <a name="category-persontype"></a>Категория: Персонтипе

Эта категория содержит следующую сущность:


:::row:::
    :::column span="":::
        **Сущность**

        персонтипе

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Типы заданий или роли, удерживаемые человеком.

        Чтобы получить эту категорию сущностей, добавьте `PersonType` к `pii-categories` параметру. `PersonType` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::

### <a name="category-phonenumber"></a>Категория: PhoneNumber

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        PhoneNumber

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Номера телефонов (только для телефонных номеров США и ЕС). Также возвращается с `domain=phi` .

        Чтобы получить эту категорию сущностей, добавьте `PhoneNumber` к `pii-categories` параметру. `PhoneNumber` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt` `pt-br`
      
   :::column-end:::

:::row-end:::


### <a name="category-organization"></a>Категория: Организация

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Организация

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Компании, «неправительственные группы», «музыкальные зоны», «спорт», «государственные органы» и «общественные организации». Национальные и религионсы не включаются в этот тип сущности.

        Чтобы получить эту категорию сущностей, добавьте `Organization` к `pii-categories` параметру. `Organization` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::

:::row-end:::

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Медицина    

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Медицинские компании и группы.

        Чтобы получить эту категорию сущностей, добавьте `OrganizationMedical` к `pii-categories` параметру. `OrganizationMedical` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::
      **Поддерживаемые языки документов**

      `en`   
      
   :::column-end:::

:::row-end:::
:::row:::
    :::column span="":::

        Обмен на фондовой бирже

    :::column-end:::
    :::column span="2":::

        Группы обмена фондовой биржи. 

        Чтобы получить эту категорию сущностей, добавьте `OrganizationStockExchange` к `pii-categories` параметру. `OrganizationStockExchange` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::

      `en`   
      
   :::column-end:::

:::row-end:::
:::row:::
    :::column span="":::

        Спорт

    :::column-end:::
    :::column span="2":::

        Организации, связанные с спортивными делами.

        Чтобы получить эту категорию сущностей, добавьте `OrganizationSports` к `pii-categories` параметру. `OrganizationSports` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::

      `en`   
      
   :::column-end:::

:::row-end:::


### <a name="category-address"></a>Категория: адрес

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        Адрес

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Полный почтовый адрес.

        Чтобы получить эту категорию сущностей, добавьте `Address` к `pii-categories` параметру. `Address` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::

:::row-end:::

### <a name="category-email"></a>Категория: Электронная почта

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        электронная почта;

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Адреса электронной почты.
      
        Чтобы получить эту категорию сущностей, добавьте `Email` к `pii-categories` параметру. `Email` При обнаружении будет возвращено в ответе API.

    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::
:::row-end:::


### <a name="category-url"></a>Категория: URL-адрес

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        URL-адрес

    :::column-end:::
    :::column span="2":::
        **Сведения**

        URL-адреса веб-сайтов. 

        Чтобы получить эту категорию сущностей, добавьте `URL` к `pii-categories` параметру. `URL` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::

:::row-end:::

### <a name="category-ip"></a>Категория: IP-адрес

Эта категория содержит следующую сущность:

:::row:::
    :::column span="":::
        **Сущность**

        IP-адрес

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Сетевые IP-адреса. 

        Чтобы получить эту категорию сущностей, добавьте `IP` к `pii-categories` параметру. `IP` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::

    :::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::
:::row-end:::

### <a name="category-datetime"></a>Категория: DateTime

Эта категория содержит следующие сущности:

:::row:::
    :::column span="":::
        **Сущность**

        Дата/время

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Даты и время суток. 

        Чтобы получить эту категорию сущностей, добавьте `DateTime` к `pii-categories` параметру. `DateTime` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
:::column span="":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
   :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        Date

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Календарные даты.

        Чтобы получить эту категорию сущностей, добавьте `Date` к `pii-categories` параметру. `Date` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**
      
      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`   
      
    :::column-end:::
:::row-end:::

### <a name="category-quantity"></a>Категория: количество

Эта категория содержит следующие сущности:

:::row:::
    :::column span="":::
        **Сущность**

        Количество

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Числа и числовые величины.

        Чтобы получить эту категорию сущностей, добавьте `Quantity` к `pii-categories` параметру. `Quantity` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="2":::
      **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`
      
    :::column-end:::
:::row-end:::

#### <a name="subcategories"></a>Подкатегории

Сущность в этой категории может иметь следующие подкатегории.

:::row:::
    :::column span="":::
        **Подкатегория сущности**

        возраст;

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Устаревают.

        Чтобы получить эту категорию сущностей, добавьте `Age` к `pii-categories` параметру. `Age` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="2":::
        **Поддерживаемые языки документов**

      `en`, `es`, `fr`, `de`, `it`, `zh-hans`, `ja`, `ko`, `pt-pt`, `pt-br`  
      
   :::column-end:::
:::row-end:::

### <a name="azure-information"></a>Сведения об Azure

Эти категории сущностей включают в себя идентифицируемые сведения об Azure, включая сведения о проверке подлинности и строки подключения. Не возвращается с `domain=phi` параметром.

:::row:::
    :::column span="":::
        **Сущность**

        ключ проверки подлинности Azure DocumentDB; 

    :::column-end:::
    :::column span="2":::
        **Сведения**

        Ключ авторизации для сервера Azure Cosmos DB.   

        Чтобы получить эту категорию сущностей, добавьте `AzureDocumentDBAuthKey` к `pii-categories` параметру. `AzureDocumentDBAuthKey` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::
      **Поддерживаемые языки документов**

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Строка подключения к базе данных Azure IAAS и строка подключения SQL Azure.
        

    :::column-end:::
    :::column span="2":::

        Строка подключения для базы данных инфраструктуры Azure в виде службы (IaaS) и строки подключения SQL.

        Чтобы получить эту категорию сущностей, добавьте `AzureIAASDatabaseConnectionAndSQLString` к `pii-categories` параметру. `AzureIAASDatabaseConnectionAndSQLString` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        строка подключения Azure IoT;  

    :::column-end:::
    :::column span="2":::

        Строка подключения для Azure IoT. 
      
        Чтобы получить эту категорию сущностей, добавьте `AzureIoTConnectionString` к `pii-categories` параметру. `AzureIoTConnectionString` При обнаружении будет возвращено в ответе API.

    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        пароль для публикации Azure;  

    :::column-end:::
    :::column span="2":::

        Пароль для параметров публикации Azure.

        Чтобы получить эту категорию сущностей, добавьте `AzurePublishSettingPassword` к `pii-categories` параметру. `AzurePublishSettingPassword` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        строка подключения кэша Redis для Azure; 

    :::column-end:::
    :::column span="2":::

        Строка подключения для кэша Redis.

        Чтобы получить эту категорию сущностей, добавьте `AzureRedisCacheString` к `pii-categories` параметру. `AzureRedisCacheString` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Azure SAS; 

    :::column-end:::
    :::column span="2":::

        Строка подключения для программного обеспечения Azure как услуги (SaaS).

        Чтобы получить эту категорию сущностей, добавьте `AzureSAS` к `pii-categories` параметру. `AzureSAS` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        строка подключения Служебной шины Azure;

    :::column-end:::
    :::column span="2":::

        Строка подключения для служебной шины Azure.

        Чтобы получить эту категорию сущностей, добавьте `AzureServiceBusString` к `pii-categories` параметру. `AzureServiceBusString` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        Ключ учетной записи хранения Azure 

    :::column-end:::
    :::column span="2":::

        Ключ учетной записи хранения Azure. 

        Чтобы получить эту категорию сущностей, добавьте `AzureStorageAccountKey` к `pii-categories` параметру. `AzureStorageAccountKey` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        ключ учетной записи хранилища Azure (универсальный).

    :::column-end:::
    :::column span="2":::

        Универсальный ключ учетной записи для учетной записи хранения Azure.

        Чтобы получить эту категорию сущностей, добавьте `AzureStorageAccountGeneric` к `pii-categories` параметру. `AzureStorageAccountGeneric` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::
:::row:::
    :::column span="":::

        строка подключения SQL Server; 

    :::column-end:::
    :::column span="2":::

        Строка подключения для компьютера, на котором работает SQL Server.

        Чтобы получить эту категорию сущностей, добавьте `SQLServerConnectionString` к `pii-categories` параметру. `SQLServerConnectionString` При обнаружении будет возвращено в ответе API.
      
    :::column-end:::
    :::column span="":::

      `en` 

    :::column-end:::
:::row-end:::

### <a name="identification"></a>Идентификация

[!INCLUDE [supported identification entities](./identification-entities.md)]
