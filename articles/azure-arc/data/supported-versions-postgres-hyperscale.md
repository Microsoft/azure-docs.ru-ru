---
title: Поддерживаемые версии Postgres с поддержкой дуги PostgreSQL в Azure
description: Поддерживаемые версии Postgres с поддержкой дуги PostgreSQL в Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 0f3f9e7f4566800bdea6871cae1c5bafb3b37b95
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "90939188"
---
# <a name="supported-versions-of-postgres-with-azure-arc-enabled-postgresql-hyperscale"></a>Поддерживаемые версии Postgres с Гипермасштабированием PostgreSQL с поддержкой Azure Arc

В этой статье объясняется, какие версии postgres доступны в службе "Дуга Azure" с поддержкой PostgreSQL Scale.
Список поддерживаемых версий со временем развивается. Сегодня поддерживаются следующие основные версии:
- Postgres 12 (по умолчанию)
- Postgres 11

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="how-to-chose-between-versions"></a>Как выбрать версию?
Рекомендуется рассмотреть, для каких версий приложений были разработаны приложения, а также каковы возможности каждой из этих версий. Дополнительные сведения см. в статье о каждой версии на официальном веб-сайте postgres:
- [Postgres 12 (по умолчанию)](https://www.postgresql.org/docs/12/index.html)
- [Postgres 11](https://www.postgresql.org/docs/11/index.html)

## <a name="how-to-create-a-particular-version-in-azure-arc-enabled-postgresql-hyperscale"></a>Как создать определенную версию в службе "Дуга Azure" с поддержкой PostgreSQLного масштабирования?
Во время создания можно указать, какую версию нужно создать, передав параметр _--Engine-Version_ . Если не указать сведения о версии, по умолчанию будет создана группа серверов postgres версии 12.

## <a name="how-do-be-notified-when-other-versions-are-available"></a>Как получать уведомления о доступности других версий?
Вернитесь к этой статье и прочтите эту статью. Он будет обновлен соответствующим образом. Вы также можете вывести список типов определений настраиваемых ресурсов (CRD) в контроллере данных ARC в кластере Kubernetes.
Выполните следующую команду.
```console
kubectl get crds
```

Он возвратит выходные данные следующего вида:
```console
NAME                                            CREATED AT
datacontrollers.arcdata.microsoft.com           2020-08-31T20:15:16Z
postgresql-11s.arcdata.microsoft.com            2020-08-31T20:15:16Z
postgresql-12s.arcdata.microsoft.com            2020-08-31T20:15:16Z
sqlmanagedinstances.sql.arcdata.microsoft.com   2020-08-31T20:15:16Z
```

В этом примере выходные данные указывают на наличие двух типов CRD, связанных с Postgres:
- postgresql-12s.arcdata.microsoft.com — это CRD для postgres 12
- postgresql-11s.arcdata.microsoft.com — это CRD для postgres 11

Это КРДС, а не группы серверов. Присутствие CRD не означает, что вы создали или не создали группу серверов этой версии.
CRD указывает, какой тип ресурсов можно создать.

## <a name="next-steps"></a>Дальнейшие действия
- [Ознакомьтесь со сведениями о создании PostgreSQL в Azure ARC с поддержкой геомасштабирования.](create-postgresql-hyperscale-server-group.md)
- [Узнайте, как получить список групп серверов PostgreSQL с поддержкой дуги Azure, созданных в контроллере данных ARC.](list-server-groups-postgres-hyperscale.md)
