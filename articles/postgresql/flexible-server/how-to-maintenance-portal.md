---
title: База данных Azure для PostgreSQL — гибкий сервер (предварительная версия) — плановое обслуживание — портал Azure
description: Узнайте, как настроить параметры планового обслуживания для базы данных Azure для PostgreSQL — гибкий сервер на портале Azure.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: be6040b8b84a4b86746d62bd2f1c07f0ffea0a3b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "91336298"
---
# <a name="manage-scheduled-maintenance-settings-for-azure-database-for-postgresql--flexible-server"></a>Управление настройками планового обслуживания для базы данных Azure для PostgreSQL — гибкий сервер
 
Вы можете указать варианты обслуживания для каждого гибкого сервера в подписке Azure. Параметры включают расписание обслуживания и настройки уведомлений для предстоящих и завершенных событий обслуживания.

> [!IMPORTANT]
> База данных Azure для PostgreSQL. Предварительная версия гибкого сервера.

## <a name="prerequisites"></a>Предварительные требования
Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:
- [База данных Azure для PostgreSQL — гибкий сервер](quickstart-create-server-portal.md)
 
## <a name="specify-maintenance-schedule-options"></a>Укажите параметры графика обслуживания
 
1. На странице сервер PostgreSQL под заголовком **Параметры** выберите **Обслуживание**, чтобы открыть параметры запланированного обслуживания.
2. Расписание по умолчанию (управляемое системой) — это случайный день недели, а 60-минутное окно для обслуживания начинается с 23:00 до 7:00 по местному времени сервера. Для изменения этого расписания выберите **Настраиваемое расписание**. Затем вы можете выбрать предпочтительный день недели и 60-минутное окно для времени начала обслуживания.
 
## <a name="notifications-about-scheduled-maintenance-events"></a>Уведомления о плановых мероприятиях по техническому обслуживанию
 
Вы можете использовать Azure Service Health для [просмотра уведомлений](../../service-health/service-notifications.md) о предстоящем и выполненном плановом обслуживании на гибком сервере. Вы также можете [настроить](../../service-health/resource-health-alert-monitor-guide.md) оповещения в Azure Service Health, чтобы получать уведомления о событиях обслуживания.
 
## <a name="next-steps"></a>Дальнейшие действия  
 
* Узнайте о [плановом обслуживании в базе данных Azure для PostgreSQL — гибкий сервер](concepts-maintenance.md)
* Узнайте про [Работоспособность службы Azure](../../service-health/overview.md)
