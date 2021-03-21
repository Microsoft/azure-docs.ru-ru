---
title: Настройка Postman для вызовов REST API служб мультимедиа Azure
description: В этой статье описывается, как настроить POST для вызовов служб мультимедиа REST API.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/10/2021
ms.author: inhenkel
ms.openlocfilehash: ef92e772085b1b89388c3f85fb3fdb91df0f6a75
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103012213"
---
# <a name="configure-postman-for-media-services-v2-rest-api-calls"></a>Настройка POST для вызовов служб мультимедиа v2 REST API вызовы

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> В Cлужбы мультимедиа версии 2 больше не добавляются новые компоненты или функциональные возможности. <br/>Ознакомьтесь с новейшей версией Служб мультимедиа — [версией 3](../latest/index.yml). Также изучите руководство по [миграции из версии 2 в версию 3](../latest/migrate-v-2-v-3-migration-introduction.md).

В этом руководстве описано, как настроить **Postman** для вызова REST API служб мультимедиа Azure (AMS). В руководстве также показано, как импортировать файлы среды и коллекции в **Postman**. Коллекция содержит сгруппированные определения HTTP-запросов, которые вызывают REST API служб мультимедиа Azure (AMS). Файл среды содержит переменные, которые используются коллекцией.

Среда и коллекция описаны в руководствах по выполнению различных задач с помощью REST API служб мультимедиа Azure.

## <a name="prerequisites"></a>Предварительные условия

- Установите клиент REST [Postman](https://www.getpostman.com/) для выполнения REST API, как показано в некоторых руководствах по REST AMS. 

    Мы используем **Postman**, но подойдет любое средство REST. Другие варианты включают: **Visual Studio Code** с подключаемым модулем REST или **Telerik Fiddler**. 

## <a name="configure-the-environment"></a>Настройка среды 

1. Создайте JSON-файл, содержащий переменные среды, используемые в руководствах по AMS. Назовите файл (например, **AzureMediaServices.postman_environment.json**). Откройте файл и вставьте код, который определяет среду Postman из [этого примера кода](postman-environment.md). 
2. Откройте **Postman**.
3. В правой части экрана выберите параметр **Управление средой**.

    ![На снимке экрана показан выбранный параметр "Управление средой".](./media/media-services-rest-upload-files/postman-create-env.png)
4. В диалоговом окне **Управление средой** нажмите кнопку **Импорт**.
5. Найдите и выберите файл **AzureMediaServices.postman_environment.json**.
6. Будет добавлена среда **AzureMedia**.
7. Закройте диалоговое окно.
8. Выберите среду **AzureMedia**.

    ![На снимке экрана показана выбранная среда AzureMedia.](./media/media-services-rest-upload-files/postman-choose-env.png)

## <a name="configure-the-collection"></a>Настройка коллекции

1. Создайте JSON-файл, содержащий коллекцию **Postman** со всеми операциями, которые необходимы для передачи файла в службы мультимедиа. Назовите файл (например, **AzureMediaServicesOperations.postman_collection.json**). Откройте файл и вставьте код, который определяет коллекцию **Postman** из этого [примера кода](postman-collection.md).
2. Нажмите кнопку **импорта**, чтобы импортировать файл коллекции.
3. Выберите файл **AzureMediaServicesOperations.postman_collection.json**.

    ![На снимке экрана показано диалоговое окно Импорт с выбранными файлами выбор файлов.](./media/media-services-rest-upload-files/postman-import-collection.png)

## <a name="next-steps"></a>Дальнейшие действия

См. руководство по [передаче файлов](media-services-rest-upload-files.md).  
