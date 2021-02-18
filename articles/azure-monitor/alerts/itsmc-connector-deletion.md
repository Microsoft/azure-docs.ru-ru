---
title: Удаление соединителя ITSM и связанного с ним действия
description: В этой статье приводится объяснение того, как удалить соединитель ITSM и связанные с ним группы действий.
ms.subservice: logs
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 12/29/2020
ms.custom: references_regions
ms.openlocfilehash: 1fbaa48d104642984e604bc66a3aa914fbfed44c
ms.sourcegitcommit: e559daa1f7115d703bfa1b87da1cf267bf6ae9e8
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/17/2021
ms.locfileid: "100624000"
---
# <a name="deletion-of-unused-itsm-connectors"></a>Удаление неиспользуемых соединителей ITSM

Процесс удаления неиспользуемого соединителя содержит 2 фазы:

1. Удаление связанных действий. все действия, связанные с соединителем ITSM, должны быть удалены. Это необходимо сделать, чтобы не выполнять действия без соединителя, которые могут вызвать ошибки в подписке.

2. Удаление неиспользуемого соединителя ITSM.

## <a name="deletion-of-the-associated-actions"></a>Удаление связанных действий

1. Чтобы найти группу действий, перейдите на  ![ снимок экрана выбора монитора.](media/itsmc-connector-deletion/itsmc-monitor-selection.png)

2. Щелкните снимок экрана "оповещения"  ![ для выбора оповещений.](media/itsmc-connector-deletion/itsmc-alert-selection.png)
3. Щелкните снимок экрана "Управление действиями"  ![ для выбора действий "Управление действиями".](media/itsmc-connector-deletion/itsmc-actions-selection.png)
4. Выберите все соединители ITSM, подключенные к  ![ снимку экрана Cherwell в соединителях ITSM, подключенных к Cherwell.](media/itsmc-connector-deletion/itsmc-actions-screen.png)
5. Удаление  ![ снимка экрана группы действий для удаления группы действий.](media/itsmc-connector-deletion/itsmc-action-deletion.png)

## <a name="deletion-of-the-unused-itsm-connector"></a>Удаление неиспользуемого соединителя ITSM

1. Необходимо выполнить поиск и выбрать "ServiceDesk" LA на  ![ снимке экрана вверху панели поиска и выбрать "ServiceDesk" La.](media/itsmc-connector-deletion/itsmc-connector-selection.png)
2. Выберите "подключения ITSM" и щелкните снимок экрана соединителя Cherwell  ![ для соединителей CHERWELL ITSM.](media/itsmc-connector-deletion/itsmc-cherwell-connector.png)
3. Выберите "Удалить"  ![ снимок экрана удаления СОЕДИНИТЕЛЯ ITSM.](media/itsmc-connector-deletion/itsmc-connector-deletion.png)

## <a name="next-steps"></a>Дальнейшие шаги

* [Устранение неполадок с соединителем ITSM](./itsmc-resync-servicenow.md)
