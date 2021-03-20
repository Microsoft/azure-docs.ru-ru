---
title: Создание пулов ресурсов с помощью сопоставления подписок Azure
titleSuffix: Azure VMware Solution by CloudSimple
description: Описание создания пулов ресурсов для частного облака с помощью сопоставления подписок Azure
author: Ajayan1008
ms.author: v-hborys
ms.date: 08/14/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 7be4a4c601d3f33972c1e52596ef623116dcadd4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97897082"
---
# <a name="create-resource-pools-for-your-private-cloud-with-azure-subscription-mapping"></a>Создание пулов ресурсов для частного облака с помощью сопоставления подписок Azure
Сопоставление подписок Azure позволяет создавать пулы ресурсов для частного облака из доступных пулов ресурсов vSphere. На портале Клаудсимпле вы можете просматривать подписку Azure для частных облаков и управлять ею.

> [!NOTE]
> Сопоставление пула ресурсов также сопоставляет все дочерние пулы ресурсов. Родительский пул ресурсов не может быть сопоставлен, если уже сопоставлены все дочерние пулы ресурсов.

1. [Доступ к порталу клаудсимпле](access-cloudsimple-portal.md).
2. Откройте страницу **ресурсы** и выберите **сопоставление подписок Azure**.  
3. Щелкните **изменить сопоставление подписок Azure**.  
4. Чтобы отобразить доступные пулы ресурсов, выберите их слева и щелкните стрелку вправо. 
5. Чтобы удалить сопоставления, выберите их справа и щелкните стрелку влево. 

    ![Подписки Azure](media/resources-azure-mapping.png)

6. Нажмите кнопку **ОК**.
