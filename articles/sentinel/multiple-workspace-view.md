---
title: Работа с инцидентами Sentinel Azure во многих рабочих областях одновременно | Документация Майкрософт
description: Просмотр инцидентов в нескольких рабочих областях одновременно в Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/20/2020
ms.author: yelevin
ms.openlocfilehash: 448998328ff15b74b0aa0b17e2435a7ff55c54a5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "83124193"
---
# <a name="work-with-incidents-in-many-workspaces-at-once"></a>Работа с инцидентами во многих рабочих областях одновременно 

 Чтобы воспользоваться всеми преимуществами возможностей Azure Sentinel, корпорация Майкрософт рекомендует использовать среду с одной рабочей областью. Однако в некоторых случаях требуется наличие нескольких рабочих областей, в некоторых случаях — от [управляемого поставщика службы безопасности (МССП)](./multiple-tenants-service-providers.md) и его клиентов — в нескольких клиентах. **Представление с несколькими рабочими областями** позволяет видеть и работать с инцидентами безопасности в нескольких рабочих областях одновременно, даже в разных клиентах, что позволяет поддерживать полную видимость и контроль скорости реагирования на безопасность вашей организации.

## <a name="entering-multiple-workspace-view"></a>Переход к представлению нескольких рабочих областей

При открытии Sentinel Azure вы увидите список всех рабочих областей, к которым у вас есть права доступа во всех выбранных клиентах и подписках. Слева от имени рабочей области находится флажок. Щелкнув имя одной рабочей области, вы перейдете в эту рабочую область. Чтобы выбрать несколько рабочих областей, щелкните все соответствующие флажки, а затем нажмите кнопку **представление нескольких рабочих областей** в верхней части страницы.

> [!IMPORTANT]
> В настоящее время представление "несколько рабочих областей" поддерживает не более 10 одновременно отображаемых рабочих областей. 
> 
> Если вы проверите более 10 рабочих областей, появится предупреждающее сообщение.

Обратите внимание, что в списке рабочих областей можно увидеть каталог, подписку, расположение и группу ресурсов, связанные с каждой рабочей областью. Каталог соответствует клиенту.

   ![Выбор нескольких рабочих областей](./media/multiple-workspace-view/workspaces.png)

## <a name="working-with-incidents"></a>Работа с инцидентами

В **представлении с несколькими рабочими областями** теперь доступен только экран **инциденты** . Он выглядит и работает в большинстве случаев, таких как обычный экран **инцидентов** . Однако существует несколько важных отличий.

   ![Просмотр инцидентов в нескольких рабочих областях](./media/multiple-workspace-view/incidents.png)

- Счетчики в верхней части страницы — *открытые* инциденты, *новые инциденты*, *выполняется* и т. д. отображают числа для всех выбранных рабочих областей в совокупности.

- Вы увидите инциденты из всех выбранных рабочих областей и каталогов (клиентов) в одном едином списке. Вы можете отфильтровать список по рабочей области и каталогу в дополнение к фильтрам на экране обычного **инцидента** .

- Необходимо иметь разрешения на чтение и запись для всех рабочих областей, из которых вы выбрали инциденты. Если в некоторых рабочих областях есть только разрешения на чтение, то при выборе инцидентов в этих рабочих областях отображаются предупреждающие сообщения. Вы не сможете изменить эти инциденты или другие, которые вы выбрали вместе с ними (даже если у вас есть разрешения для других пользователей).

- Если выбрать один инцидент и щелкнуть **Просмотреть все подробные сведения** или **исследовать**, то будет находиться в контексте данных рабочей области инцидента и других.

## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как просматривать инциденты в нескольких рабочих областях Azure Sentinel и работать с ними одновременно. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).

