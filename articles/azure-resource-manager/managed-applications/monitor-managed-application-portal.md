---
title: Использование портал Azure для мониторинга управляемого приложения
description: Показано, как использовать портал Azure для мониторинга доступности и оповещений управляемого приложения.
author: tfitzmac
ms.topic: conceptual
ms.date: 10/04/2018
ms.author: tomfitz
ms.openlocfilehash: a02062edd1a940bcc6588ab53457e0af91fedd9a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100578268"
---
# <a name="monitor-a-deployed-instance-of-a-managed-application"></a>Мониторинг развернутого экземпляра управляемого приложения

После развертывания управляемого приложения в вашей подписке Azure, можно проверить состояние приложения. В этой статье показаны параметры проверки состояния на портале Azure. Можно отслеживать доступность ресурсов в управляемом приложении. Можно также настроить и просмотреть оповещения.

## <a name="view-resource-health"></a>Просмотр состояния работоспособности ресурса.

1. Выберите экземпляр управляемого приложения.

   ![Выбор управляемого приложения](./media/monitor-managed-application-portal/select-managed-application.png)

1. Выберите элемент **Работоспособность ресурсов**.

   ![Выбор работоспособности ресурсов](./media/monitor-managed-application-portal/select-resource-health.png)

1. В управляемом приложении можно отслеживать доступность ресурсов.

   ![Просмотр состояния работоспособности ресурса.](./media/monitor-managed-application-portal/view-health.png)

## <a name="view-alerts"></a>Просмотр оповещений

1. Выберите **Оповещения**.

   ![Выбор оповещений](./media/monitor-managed-application-portal/select-alerts.png)

1. Если правила генерации оповещений настроены, появится информация о созданных оповещениях.

   ![Просмотр оповещений](./media/monitor-managed-application-portal/view-alerts.png)

1. Чтобы добавить правила генерации оповещений, выберите **+Новое правило генерации оповещений**.

   ![Создание оповещения](./media/monitor-managed-application-portal/create-new-alert.png)

В управляемом приложении, для его экземпляра или ресурсов можно создать оповещения. Сведения о создании оповещений см. в статье [Обзор оповещений в Microsoft Azure](../../azure-monitor/alerts/alerts-overview.md).

## <a name="next-steps"></a>Дальнейшие действия

* См. дополнительные сведения о [примерах проектов для управляемых приложений Azure](sample-projects.md).
* Сведения о том, как развернуть экземпляр управляемого приложения, см. в статье [Развертывание приложения из каталога служб с помощью портала Azure](deploy-service-catalog-quickstart.md).