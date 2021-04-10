---
title: Запрос ключа авторизации для ExpressRoute
description: Инструкции по запросу ключа авторизации для ExpressRoute.
ms.topic: include
ms.date: 03/15/2021
ms.openlocfilehash: 99d9fba33d64fca1d9c5b960041fbabe1f9060db
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105027017"
---
<!-- used in expressroute-global-reach-private-cloud.md and create-ipsec-tunnel.md -->

1. На портале Azure в разделе **Подключение** на вкладке **ExpressRoute** нажмите **+ Запросить ключ авторизации**. 

   :::image type="content" source="../media/expressroute-global-reach/start-request-authorization-key.png" alt-text="Снимок экрана, показывающий, как запросить ключ авторизации ExpressRoute." border="true" lightbox="../media/expressroute-global-reach/start-request-authorization-key.png":::

1. Укажите имя и выберите **Создать**. 
      
   Создание ключа может занять около 30 секунд. После создания новый ключ появится в списке ключей авторизации для частного облака.

   :::image type="content" source="../media/expressroute-global-reach/show-global-reach-auth-key.png" alt-text="Снимок экрана, показывающий ключ авторизации ExpressRoute Global Reach.":::
  
1. Скопируйте ключ авторизации и идентификатор ExpressRoute. Они понадобятся для установки пиринга.  

   > [!NOTE]
   > Ключ авторизации через некоторое время исчезнет, поэтому скопируйте его сразу же, когда он появится.