---
title: Настройка POST для служб мультимедиа Azure v3 REST API
description: В этой статье описано, как настроить Postman для вызова REST API Служб мультимедиа Azure (AMS).
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: b0a521ddeee6d93ce809e5e6e85eefa1f01a3f12
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98953977"
---
# <a name="configure-postman-for-media-services-v3-rest-api-calls"></a>Настройка POST для вызовов REST API служб мультимедиа v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

В этой статье описано, как настроить **Postman** для вызова REST API Служб мультимедиа Azure (AMS). Здесь также описывается, как импортировать файлы среды и коллекции в **Postman**. Коллекция содержит сгруппированные определения HTTP-запросов, которые вызывают REST API служб мультимедиа Azure (AMS). Файл среды содержит переменные, которые используются коллекцией.

Перед началом разработки ознакомьтесь [с разработкой с помощью API-интерфейсов служб мультимедиа v3](media-services-apis-overview.md).

## <a name="prerequisites"></a>Предварительные требования

- [Создание учетной записи Служб мультимедиа](./create-account-howto.md). Обязательно запомните имя группы ресурсов и имя учетной записи Служб мультимедиа. 
- Получите информацию, необходимую для [доступа к API-интерфейсам](./access-api-howto.md).
- Установите клиент REST [Postman](https://www.getpostman.com/) для выполнения REST API, как показано в некоторых руководствах по REST AMS. 

    Мы используем **Postman**, но подойдет любое средство REST. Другие варианты включают: **Visual Studio Code** с подключаемым модулем REST или **Telerik Fiddler**. 

> [!IMPORTANT]
> Проверьте [соглашения об именовании](media-services-apis-overview.md#naming-conventions).

## <a name="download-postman-files"></a>Скачивание файлов Postman

Клонируйте репозиторий GitHub, который содержит файлы коллекции и среды Postman.

 ```bash
 git clone https://github.com/Azure-Samples/media-services-v3-rest-postman.git
 ```

## <a name="configure-postman"></a>Настройка Postman

### <a name="configure-the-environment"></a>Настройка среды 

1. Откройте приложение **Postman**.
2. В правой части экрана выберите параметр **Управление средой**.

    ![Управление средой](./media/develop-with-postman/postman-import-env.png)
4. В диалоговом окне **Управление средой** нажмите кнопку **Импорт**.
2. Перейдите к файлу `Azure Media Service v3 Environment.postman_environment.json`, который вы скачали при клонировании `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`.
6. Будет добавлена среда **Azure Media Service v3 Environment**.

    > [!Note]
    > Задайте переменным доступа новые значения, полученные в разделе **Доступ к API Служб мультимедиа** выше.

7. Дважды щелкните выбранный файл и введите значения, которые вы получили, получив доступ к API.
8. Закройте диалоговое окно.
9. Выберите среду **Azure Media Service v3 Environment** в раскрывающемся списке.

    ![Выбор среды](./media/develop-with-postman/choose-env.png)
   
### <a name="configure-the-collection"></a>Настройка коллекции

1. Нажмите кнопку **импорта**, чтобы импортировать файл коллекции.
1. Перейдите к файлу `Media Services v3.postman_collection.json`, который вы скачали при клонировании `https://github.com/Azure-Samples/media-services-v3-rest-postman.git`.
3. Выберите файл **Media Services v3.postman_collection.json**.

    ![Импорт файла](./media/develop-with-postman/postman-import-collection.png)

## <a name="get-azure-ad-token"></a>Получение маркера Azure AD 

Прежде чем вы сможете работать с ресурсами AMS версии 3, вам нужно получить и задать маркер Azure AD для аутентификации субъекта-службы.

1. В левом окне приложения Postman выберите "Шаг 1. Получить маркер проверки подлинности AAD".
2. Затем выберите Get Azure AD Token for Service Principal Authentication (Получение маркера безопасности Azure AD для аутентификации субъекта-службы).
3. Нажмите кнопку **Отправить**.

    Это действие отправляет следующую операцию **POST**.

    ```
    https://login.microsoftonline.com/:tenantId/oauth2/token
    ```

4. Ответ на этот запрос возвращает маркер и присваивает его значение переменной среды AccessToken.  

    ![Получение маркера AAD](./media/develop-with-postman/postman-get-aad-auth-token.png)

## <a name="troubleshooting"></a>Устранение неполадок 

* Если приложение завершается с ошибкой "HTTP 504: время ожидания шлюза", убедитесь, что для переменной расположения не было явно задано значение, отличное от ожидаемого расположения учетной записи служб мультимедиа. 
* Если возникает ошибка "учетная запись не найдена", убедитесь, что для свойства Location в тексте сообщения JSON задано расположение учетной записи служб мультимедиа. 

## <a name="see-also"></a>См. также раздел

- [Создание фильтров с помощью REST API Служб мультимедиа](filters-dynamic-manifest-rest-howto.md)
- [REST API на основе Azure Resource Manager](https://github.com/Azure-Samples/media-services-v3-arm-templates)

## <a name="next-steps"></a>Дальнейшие действия

- [Потоковая передача файлов с помощью REST](stream-files-tutorial-with-rest.md)  
- [Руководство. Кодирование удаленного файла на основе URL-адреса и потоковой передачи видео с помощью REST](stream-files-tutorial-with-rest.md)
