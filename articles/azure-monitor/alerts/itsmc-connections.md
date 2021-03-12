---
title: Соединитель управления ИТ-услугами в Azure Monitor
description: В этой статье описывается, как подключить продукты и службы ITSM с помощью соединителя управления ИТ-службами (ITSM) в Azure Monitor, чтобы централизованно отслеживать рабочие элементы ITSM и управлять ими.
ms.topic: conceptual
author: nolavime
ms.author: v-jysur
ms.date: 05/12/2020
ms.openlocfilehash: 40e737a1ec5fb34cd22a08925143a100d36cdc6b
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103009323"
---
# <a name="connect-itsm-productsservices-with-it-service-management-connector"></a>Подключение продуктов и служб ITSM с помощью соединителя управления ИТ-службами
В этой статье описывается, как настроить в Log Analytics связь между продуктами или службами ITSM и соединителем управления ИТ-службами (ITSM), чтобы централизованно управлять рабочими элементами ITSM. Дополнительные сведения об ITSMC см. в [этом обзоре](./itsmc-overview.md).

Поддерживаются следующие службы и продукты ITSM. Выберите продукт, чтобы просмотреть подробные сведения о том, как его подключить к ITSMC.

- [ServiceNow](./itsmc-connections-servicenow.md)
- [System Center Service Manager](./itsmc-connections-scsm.md);
- [Cherwell](./itsmc-connections-cherwell.md)
- [Provance](./itsmc-connections-provance.md);

> [!NOTE]
> Мы предлагаем нашим клиентам Cherwell и Provance использовать [действие веб-перехватчика](./action-groups.md#webhook) для Cherwell и Provance конечной точки в качестве другого решения интеграции.

## <a name="ip-ranges-for-itsm-partners-connections"></a>Диапазоны IP-адресов для подключений партнеров ITSM
Чтобы получить список IP-адресов ITSM, чтобы разрешить подключения ITSM от партнеров ITSM Tools, мы рекомендуем использовать для перечисления всего общедоступного диапазона IP-адреса региона Azure, в котором находится рабочая область LogAnalytics. [подробные сведения](https://www.microsoft.com/en-us/download/details.aspx?id=56519) Для регионов ЕУС/ВЕУ/EUS2/WUS2/Юго-Центральный регион США клиент может вывести только тег сети ActionGroup.

## <a name="next-steps"></a>Дальнейшие действия

* [Устранение неполадок с соединителем ITSM](./itsmc-resync-servicenow.md)
