---
title: Пометка виртуальной машины с помощью портал Azure
description: Сведения о добавлении тегов виртуальной машины с помощью портал Azure.
ms.topic: how-to
ms.workload: infrastructure-services
author: cynthn
ms.service: virtual-machines
ms.date: 11/11/2020
ms.author: cynthn
ms.openlocfilehash: 9c220814ae1f714e9eac0c0c11a50d9cf68a621d
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98897414"
---
# <a name="tagging-a-vm-using-the-portal"></a>Добавление тегов для виртуальной машины с помощью портала

В этой статье описывается, как добавить теги в виртуальную машину с помощью портала. Теги — это определяемые пользователем пары "ключ-значение", которые можно помещать непосредственно в ресурс или группу ресурсов. В настоящее время Azure поддерживает до 50 тегов на ресурс и группу ресурсов. Теги можно добавлять к ресурсу во время его создания или к уже существующему ресурсу.


1. Перейдите к виртуальной машине на портале.
1. В разделе **основные компоненты** выберите **щелкните здесь, чтобы добавить теги**.

    :::image type="content" source="media/tag/azure-portal-tag.png" alt-text="Снимок экрана с разделом основных компонентов на странице виртуальной машины.":::

1. Добавьте значение для параметра **имя** и **значение** и нажмите кнопку **сохранить**.

    :::image type="content" source="media/tag/key-value.png" alt-text="Снимок экрана страницы для добавления пары &quot;ключ-значение&quot; в качестве тега.":::

### <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о добавлении тегов для ресурсов Azure см. в статьях [Общие сведения об Azure Resource Manager](../azure-resource-manager/management/overview.md) и [Использование тегов для организации ресурсов в Azure](../azure-resource-manager/management/tag-resources.md).
- Чтобы узнать, как теги могут помочь в управлении использованием ресурсов Azure, см. статью [понимание счета Azure](../cost-management-billing/understand/review-individual-bill.md).
