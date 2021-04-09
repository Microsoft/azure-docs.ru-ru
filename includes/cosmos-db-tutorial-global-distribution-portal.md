---
title: Глобальное распределение Azure Cosmos DB
description: Сведения о глобальной репликации данных с помощью Azure DB Cosmos на портале Azure
services: cosmos-db
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.topic: include
ms.date: 12/26/2018
ms.custom: include file
ms.openlocfilehash: 58788d6194454c8bd40730c9c350aa901924ba3d
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96028052"
---
## <a name="add-global-database-regions-using-the-azure-portal"></a><a id="addregion"></a>Добавление регионов глобальной базы данных с помощью портала Azure
Служба Azure Cosmos DB доступна во всех [регионах Azure][azureregions] по всему миру. После выбора уровня согласованности по умолчанию для учетной записи базы данных вы можете связать один или несколько регионов (в зависимости от выбранного уровня согласованности по умолчанию и потребностей глобального распространения).

1. На левой панели на [портале Azure](https://portal.azure.com/) щелкните **Azure Cosmos DB**.
2. На странице **Azure Cosmos DB** выберите учетную запись базы данных, которую нужно изменить.
3. На странице учетной записи в меню щелкните **Глобальная репликация данных**.
4. На странице **Глобальная репликация данных** выберите регионы для добавления или удаления, щелкнув их на карте, и нажмите кнопку **Сохранить**. Добавление регионов оплачивается. Дополнительные сведения см. на [странице цен](https://azure.microsoft.com/pricing/details/cosmos-db/) или в руководстве по [глобальному распространению данных с помощью Azure Cosmos DB](../articles/cosmos-db/distribute-data-globally.md).
   
    ![Можно щелкать регионы на карте, чтобы добавить или удалить их.][1]
    
Когда вы добавите второй регион, на портале на странице **Глобальная репликация данных** активируется параметр **Ручная отработка отказа**. Этот параметр можно использовать для тестирования отработки отказа или изменения основного региона записи. После добавления третьего региона на той же странице активируется параметр **Приоритеты при отработке отказа**. Он позволяет изменить порядок отработки отказов для операций чтения.  

### <a name="selecting-global-database-regions"></a>Выбор регионов глобальной базы данных
Существуют два распространенных сценария настройки нескольких регионов:

1. Обеспечение низкой задержки при обращении пользователей к данным по всему миру не зависимо от расположения.
2. Добавление региональной устойчивости для непрерывности бизнес-процессов и аварийного восстановления (BCDR)

Чтобы обеспечить низкую задержку для пользователей, мы советуем развернуть приложение и Azure Cosmos DB в регионы, соответствующие расположению пользователей приложения.

Для BCDR мы рекомендуем добавлять регионы исходя из пар регионов, описанных в статье [Непрерывность бизнес-процессов и аварийное восстановление в службах BizTalk: пары регионов Azure][bcdr].

<!--

## <a id="selectwriteregion"></a>Select the write region

While all regions associated with your Cosmos DB database account can serve reads (both, single item as well as multi-item paginated reads) and queries, only one region can actively receive the write (insert, upsert, replace, delete) requests. To set the active write region, do the following  


1. In the **Azure Cosmos DB** blade, select the database account to modify.
2. In the account blade, click **Replicate data globally** from the menu.
3. In the **Replicate data globally** blade, click **Manual Failover** from the top bar.
    ![Change the write region under Azure Cosmos DB Account > Replicate data globally > Manual Failover][2]
4. Select a read region to become the new write region, click the checkbox to confirm triggering a failover, and click OK
    ![Change the write region by selecting a new region in list under Azure Cosmos DB Account > Replicate data globally > Manual Failover][3]

--->

<!--Image references-->
[1]: ./media/cosmos-db-tutorial-global-distribution-portal/azure-cosmos-db-add-region.png
[2]: ./media/cosmos-db-tutorial-global-distribution-portal/azure-cosmos-db-manual-failover-1.png
[3]: ./media/cosmos-db-tutorial-global-distribution-portal/azure-cosmos-db-manual-failover-2.png

<!--Reference style links - using these makes the source content way more readable than using inline links-->
[bcdr]: ../articles/best-practices-availability-paired-regions.md
[consistency]: ../articles/cosmos-db/consistency-levels.md
[azureregions]: https://azure.microsoft.com/regions/#services
[offers]: https://azure.microsoft.com/pricing/details/cosmos-db/