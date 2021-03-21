---
title: Перенос управления API Azure в разных регионах
description: Узнайте, как перенести экземпляр управления API из одного региона в другой.
services: api-management
documentationcenter: ''
author: miaojiang
manager: erikre
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 08/26/2019
ms.author: apimpm
ms.openlocfilehash: 0eed2328aca78402c5f4691bb9b3d07d4f36472e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86250232"
---
# <a name="how-to-migrate-azure-api-management-across-regions"></a>Перенос управления API Azure в разных регионах
Чтобы перенести экземпляры службы управления API из одного региона Azure в другой, можно использовать функцию [резервного копирования и восстановления](api-management-howto-disaster-recovery-backup-restore.md) . Необходимо выбрать ту же ценовую категорию управления API в исходном и целевом регионах. 

> [!NOTE]
> Резервное копирование и восстановление не будут работать при миграции между разными типами облака. Для этого необходимо экспортировать ресурс в [качестве шаблона](../azure-resource-manager/management/manage-resource-groups-portal.md#export-resource-groups-to-templates). Затем адаптируйте экспортированный шаблон к целевому региону Azure и повторно создайте ресурс. 

## <a name="option-1-use-a-different-api-management-instance-name"></a>Вариант 1. Использование другого имени экземпляра управления API

1. В целевом регионе создайте новый экземпляр управления API с той же ценовой категорией, что и исходный экземпляр управления API. Новый экземпляр должен иметь другое имя. 
1. Резервное копирование существующего экземпляра службы управления API в учетную запись хранения.
1. Восстановите резервную копию, созданную на шаге 2, в новый экземпляр управления API, созданный в новом регионе на шаге 1.
1. Если у вас есть пользовательский домен, указывающий на экземпляр управления API исходного региона, обновите запись CNAME пользовательского домена, чтобы она указывала на новый экземпляр управления API. 


## <a name="option-2-use-the-same-api-management-instance-name"></a>Вариант 2. использование того же имени экземпляра управления API

> [!NOTE]
> Этот параметр приведет к простою во время миграции.

1. Создайте резервную копию экземпляра службы управления API в исходном регионе в учетной записи хранения.
1. Удалите управление API в исходном регионе. 
1. Создайте новый экземпляр управления API в целевом регионе с тем же именем, что и в исходном регионе.
1. Восстановите резервную копию, созданную на шаге 1, в новый экземпляр управления API в целевом регионе.  


## <a name="next-steps"></a><a name="next-steps"> </a>Дальнейшие действия
* Дополнительные сведения о функции резервного копирования и восстановления см. [в разделе Реализация аварийного восстановления](api-management-howto-disaster-recovery-backup-restore.md).
* Сведения о переносе ресурсов Azure см. в [статье Руководство по миграции в различные регионы Azure](https://github.com/Azure/Azure-Migration-Guidance).
* [Оптимизируйте и сохраняйте расходы на облачные](../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)технологии.
