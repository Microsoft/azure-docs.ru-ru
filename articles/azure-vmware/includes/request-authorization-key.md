---
title: Запрос ключа авторизации для ExpressRoute
description: Инструкции по запросу ключа авторизации для ExpressRoute.
ms.topic: include
ms.date: 03/15/2021
ms.openlocfilehash: 54a610c8b0f3f3fe9d3ebe39291bba7767007839
ms.sourcegitcommit: 4bda786435578ec7d6d94c72ca8642ce47ac628a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103491858"
---
<!-- used in expressroute-global-reach-private-cloud.md and create-ipsec-tunnel.md -->

1. На портале Azure в разделе **Подключение** на вкладке **ExpressRoute** нажмите **+ Запросить ключ авторизации**. 

   :::image type="content" source="../media/expressroute-global-reach/start-request-authorization-key.png" alt-text="Снимок экрана, показывающий, как запросить ключ авторизации ExpressRoute." border="true" lightbox="../media/expressroute-global-reach/start-request-authorization-key.png":::

1. Укажите имя и выберите **Создать**. 
      
   Создание ключа может занять около 30 секунд. После создания новый ключ появится в списке ключей авторизации для частного облака.

   :::image type="content" source="../media/expressroute-global-reach/show-global-reach-auth-key.png" alt-text="Снимок экрана, показывающий ключ авторизации ExpressRoute Global Reach.":::
  
1. Запишите ключ авторизации и идентификатор ExpressRoute. Они понадобятся для установки пиринга.  

   > [!NOTE]
   > Ключ авторизации через некоторое время исчезнет, поэтому скопируйте его сразу же, когда он появится.