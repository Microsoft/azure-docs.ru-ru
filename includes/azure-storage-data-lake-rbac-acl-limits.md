---
services: storage
author: normesta
ms.service: storage
ms.topic: include
ms.date: 09/29/2020
ms.author: normesta
ms.custom: include file
ms.openlocfilehash: 97b47b48a183951002d04cd58b08d5e2b9051eda
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96017694"
---
| Механизм | Область |Ограничения | Поддерживаемый уровень разрешений |
|---|---|---|---|
| Azure RBAC | Учетные записи хранения, контейнеры. <br>Назначения ролей перекрестных ресурсов Azure на уровне подписки или группы ресурсов. | 2000 назначения ролей Azure в подписке | Роли Azure (встроенные или пользовательские) |
| СПИСКОМ| Каталог, файл |32. записи ACL (фактически 28 записей ACL) для каждого файла и каталога. Списки ACL для доступа и по умолчанию имеют собственный предел записей ACL 32. |Разрешение ACL|
