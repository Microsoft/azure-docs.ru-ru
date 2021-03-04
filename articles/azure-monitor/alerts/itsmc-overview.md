---
title: Обзор Соединитель управления ИТ-услугами
description: В этой статье представлен обзор Соединитель управления ИТ-услугами (ITSMC).
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 12/16/2020
ms.custom: references_regions
ms.openlocfilehash: 3f6131bc585b91676f29ed34dfedd49a5ca92201
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102041796"
---
# <a name="it-service-management-connector-overview"></a>Обзор Соединитель управления ИТ-услугами

:::image type="icon" source="media/itsmc-overview/itsmc-symbol.png":::

Соединитель управления ИТ-услугами (ITSMC) позволяет подключать Azure к поддерживаемому продукту или службе управления ИТ-службами (ITSM).

Службы Azure, такие как Azure Log Analytics и Azure Monitor, предоставляют средства для обнаружения, анализа и устранения неполадок в ресурсах Azure и не являющихся ресурсами Azure. Но рабочие элементы, связанные с проблемой, обычно находятся в ITSM продукте или службе. ITSMC обеспечивает двунаправленное подключение между инструментами Azure и ITSM, которые позволяют быстрее устранять проблемы.

## <a name="configuration-steps"></a>Этапы настройки

Соединитель управления ИТ-услугами поддерживает подключения к следующим инструментам ITSM:

-   ServiceNow
-   System Center Service Manager;
-   Provance;
-   Cherwell

   >[!NOTE]
> Начиная с 1-Oct-2020 Cherwell и Provance ITSM интеграция с оповещением Azure больше не будут включены для новых клиентов. Новые подключения ITSM не будут поддерживаться.
> Существующие подключения ITSM будут поддерживаться.

С помощью ITSMC можно:

-  Создание рабочих элементов в средстве ITSM на основе оповещений Azure (оповещения метрик, оповещения журнала действий и Log Analytics оповещения).
-  При необходимости синхронизировать данные инцидентов и запросов на изменение из инструмента ITSM с рабочей областью Azure Log Analytics.

Сведения о юридических условиях и политике конфиденциальности см. в [заявлении о конфиденциальности Майкрософт](https://go.microsoft.com/fwLink/?LinkID=522330&clcid=0x9).

Вы можете начать использовать ITSMC, выполнив следующие действия.

1. [Настройте среду ITSM, чтобы принимать оповещения из Azure.](./itsmc-connections.md)
1. [Настройка решения Azure ITSM](./itsmc-definition.md#add-it-service-management-connector)
1. [Настройте соединитель Azure ITSM для среды ITSM.](./itsmc-definition.md#create-an-itsm-connection)
1. [Настройте группу действий, чтобы использовать соединитель ITSM.](./itsmc-definition.md#define-a-template)

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок с соединителем ITSM](./itsmc-resync-servicenow.md)
