---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 03/26/2020
ms.author: larryfr
ms.openlocfilehash: 493c674fa161bf33436e560fdcbb9196410db931
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102209480"
---
> [!TIP]
> После входа в систему появится список подписок, связанных с учетной записью Azure. В сведениях о подписке с `isDefault: true` указана текущая активная подписка для команд Azure CLI. Эта подписка должна быть той же, которая содержит рабочую область Машинного обучения Azure. Идентификатор подписки можно найти на [портале Azure](https://portal.azure.com), перейдя на страницу обзора рабочей области. Вы также можете использовать пакет SDK для получения идентификатора подписки из объекта рабочей области. Например, `Workspace.from_config().subscription_id`.
> 
> Чтобы выбрать другую подписку, укажите имя или идентификатор этой подписки с помощью команды `az account set -s <subscription name or ID>`. См. дополнительные сведения о [выборе нужной подписки при использовании нескольких подписок Azure](/cli/azure/manage-azure-subscriptions-azure-cli).