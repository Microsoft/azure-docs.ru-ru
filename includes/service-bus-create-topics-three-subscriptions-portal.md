---
title: включить файл
description: включить файл
services: service-bus-messaging
author: spelluru
ms.service: service-bus-messaging
ms.topic: include
ms.date: 02/20/2019
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: ace42278269ff6af31902dbecead81329815af12
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "67185523"
---
## <a name="create-a-topic-using-the-azure-portal"></a>Создание раздела с помощью портала Azure
1. На странице **Пространство имен служебной шины** выберите пункт **Разделы** в левом меню.
2. На панели инструментов выберите **+ Раздел**. 
4. Введите **имя** раздела. Для других параметров оставьте значения по умолчанию.
5. Нажмите кнопку **Создать**.

    ![Создание раздела](./media/service-bus-create-topics-subscriptions-portal/create-topic.png)

## <a name="create-subscriptions-to-the-topic"></a>Создание подписок на раздел
1. Выберите **раздел**, который был создан в предыдущем разделе. 
    
    ![Выбор раздела](./media/service-bus-create-topics-subscriptions-portal/select-topic.png)
2. На странице **Раздел служебной шины** выберите пункт **Подписки** в левом меню, а затем — **+ Подписка** на панели инструментов. 
    
    ![Кнопка добавления подписки](./media/service-bus-create-topics-subscriptions-portal/add-subscription-button.png)
3. На странице **Создание подписки** введите **S1** в качестве **имени** подписки и нажмите кнопку **Создать**. 

    ![Страница создания подписки](./media/service-bus-create-topics-subscriptions-portal/create-subscription-page.png)
4. Повторите предыдущий шаг дважды, создав подписки с именами **S2** и **S3**.