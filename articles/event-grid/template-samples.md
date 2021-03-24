---
title: Примеры шаблонов Azure Resource Manager для службы "Сетка событий" | Документация Майкрософт
description: В этой статье представлен список примеров шаблонов Azure Resource Manager для службы "Сетка событий Azure" на GitHub.
ms.topic: sample
ms.date: 07/07/2020
ms.openlocfilehash: 910012adf2dc930e6f1a26f1a7fc41f5ed0580c9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86119061"
---
# <a name="azure-resource-manager-templates-for-event-grid"></a>Шаблоны Azure Resource Manager для службы "Сетка событий"

Синтаксис и свойства JSON, используемые в шаблоне, см. в статье о [типах ресурсов Microsoft.EventGrid](/azure/templates/microsoft.eventgrid/allversions). В следующей таблице представлены ссылки на шаблоны Azure Resource Manager для службы "Сетка событий".

## <a name="event-grid-subscriptions"></a>Подписки на службу "Сетка событий"
- [Настраиваемый раздел и подписка с конечной точкой веб-перехватчика](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid) — развертывает настраиваемый раздел службы "Сетка событий". Создает подписку на этот пользовательский раздел, который использует конечную точку веб-перехватчика. 
- [Подписка на настраиваемый раздел с конечной точкой концентратора событий](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-event-hubs-handler) — создает подписку службы "Сетка событий" на настраиваемый раздел. Подписка использует концентратор событий для конечной точки. 
- [Подписка Azure или подписка на группу ресурсов](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-resource-events-to-webhook) — создает подписку на события для группы ресурсов или подписку Azure. Группа ресурсов, указанная как целевой объект во время развертывания, является источником событий. Подписка использует веб-перехватчик для конечной точки. 
- [Учетная запись Хранилища BLOB-объектов и подписка](https://github.com/Azure/azure-quickstart-templates/tree/master/101-event-grid-subscription-and-storage) — развертывает учетную запись службы Хранилища BLOB-объектов Azure и создает подписку на события для этой учетной записи хранения. 

## <a name="next-steps"></a>Дальнейшие действия
См. следующие примеры:

- [Примеры для PowerShell](powershell-samples.md)
- [Примеры интерфейса командной строки](cli-samples.md)
