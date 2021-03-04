---
title: Данные, синхронизированные из продукта ITSM в LA рабочую область
description: В этой статье представлен обзор данных, синхронизированных из продукта ITSM в LA рабочую область.
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 12/29/2020
ms.custom: references_regions
ms.openlocfilehash: 83994c1b6e150342a777a079d79d6a594d30c3ff
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102041609"
---
# <a name="data-synced-from-your-itsm-product"></a>Данные, синхронизированные из продукта ITSM

Инциденты и запросы на изменение синхронизируются из средства ITSM в рабочую область Log Analytics в зависимости от конфигурации подключения (с помощью поля "Синхронизация данных"):
* [ServiceNow](./itsmc-connections-servicenow.md)
* [System Center Service Manager](./itsmc-connections-scsm.md);
* [Cherwell](./itsmc-connections-cherwell.md)
* [Provance](./itsmc-connections-provance.md);

## <a name="synced-data"></a>Синхронизированные данные

В этом разделе показаны некоторые примеры данных, собранных ITSMC.

Поля в **ServiceDesk_CL** зависят от типа рабочего элемента, который импортируется в log Analytics. Ниже приведен список полей для двух типов рабочих элементов:

**Рабочий элемент:** **Инциденты**  
ServiceDeskWorkItemType_s="Incident"

**Fields**

- ServiceDeskConnectionName;
- "Идентификатор службы поддержки";
- Состояние
- "Срочность";
- Влияние
- Приоритет
- Escalation (Эскалация);
- Автор
- "Кем разрешено";
- Closed By (Кем закрыто);
- Источник
- Кому назначено
- Категория
- Title
- Описание
- "Дата создания";
- Closed Date (Дата закрытия);
- Resolved Date (Дата разрешения);
- "Дата последнего изменения";
- Компьютер

**Рабочий элемент:** **Запросы на изменение**

ServiceDeskWorkItemType_s="ChangeRequest"

**Fields**
- ServiceDeskConnectionName;
- "Идентификатор службы поддержки";
- Автор
- Closed By (Кем закрыто);
- Источник
- Кому назначено
- Title
- Тип
- Категория
- Состояние
- Escalation (Эскалация);
- Conflict Status (Состояние конфликта);
- "Срочность";
- Приоритет
- Риск
- Влияние
- Кому назначено
- "Дата создания";
- Closed Date (Дата закрытия);
- "Дата последнего изменения";
- Requested Date (Запрошенная дата);
- Planned Start Date (Планируемая дата начала);
- Planned End Date (Планируемая дата окончания);
- Work Start Date (Дата начала работы);
- Work End Date (Дата окончания работы);
- Описание
- Компьютер

## <a name="servicenow-example"></a>Пример ServiceNow 
### <a name="output-data-for-a-servicenow-incident"></a>Выходные данные инцидента ServiceNow

| Поле Log Analytics | Поле ServiceNow |
|:--- |:--- |
| ServiceDeskId_s| Number |
| IncidentState_s | Состояние |
| Urgency_s |"Срочность"; |
| Impact_s |Влияние|
| Priority_s | Приоритет |
| CreatedBy_s | Opened by (Кем открыто) |
| ResolvedBy_s | "Кем разрешено"|
| ClosedBy_s  | Closed By (Кем закрыто) |
| Source_s| Contact type (Тип контакта) |
| AssignedTo_s | Кому назначено  |
| Category_s | Категория |
| Title_s|  Краткое описание |
| Description_s|  Примечания |
| CreatedDate_t|  Opened (Открыто) |
| ClosedDate_t| closed|
| ResolvedDate_t|"Разрешено"|
| Компьютер  | Элемент конфигурации |

### <a name="output-data-for-a-servicenow-change-request"></a>Выходные данные запроса на изменение ServiceNow

| Log Analytics | Поле ServiceNow |
|:--- |:--- |
| ServiceDeskId_s| Number |
| CreatedBy_s | "Кем запрошено" |
| ClosedBy_s | Closed By (Кем закрыто) |
| AssignedTo_s | Кому назначено  |
| Title_s|  Краткое описание |
| Type_s|  Тип |
| Category_s|  Категория |
| CRState_s|  Состояние|
| Urgency_s|  "Срочность"; |
| Priority_s| Приоритет|
| Risk_s| Риск|
| Impact_s| Влияние|
| RequestedDate_t  | Requested by date |
| ClosedDate_t | Closed Date (Дата закрытия) |
| PlannedStartDate_t  | Planned Start Date (Планируемая дата начала) |
| PlannedEndDate_t  | Planned End Date (Планируемая дата окончания) |
| WorkStartDate_t  | Actual start date (Фактическая дата начала) |
| WorkEndDate_t | Actual end date (Фактическая дата окончания)|
| Description_s | Описание |
| Компьютер  | Элемент конфигурации |

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок с соединителем ITSM](./itsmc-resync-servicenow.md)
