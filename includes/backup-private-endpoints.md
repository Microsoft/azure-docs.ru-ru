---
author: v-amallick
ms.service: backup
ms.topic: include
ms.date: 03/12/2020
ms.author: v-amallick
ms.openlocfilehash: d20ed4d39921f8000f77f947c4372bd8b10ca642
ms.sourcegitcommit: af6eba1485e6fd99eed39e507896472fa930df4d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/04/2021
ms.locfileid: "106294122"
---
Теперь вы можете использовать [частные конечные точки](../articles/private-link/private-endpoint-overview.md) для безопасного резервного копирования данных с серверов внутри виртуальной сети в хранилище Служб восстановления. Частная конечная точка использует IP-адрес из адресного пространства виртуальной сети для вашего хранилища. Сетевой трафик между ресурсами в виртуальной сети и хранилищем проходит по виртуальной сети и частный канал в магистральную сеть Майкрософт. Это устраняет уязвимость в общедоступном Интернете. Частные конечные точки можно использовать для резервного копирования и восстановления баз данных SQL и SAP HANA, которые работают в виртуальных машинах Azure. Их также можно использовать для локальных серверов с помощью агента MARS.

Для резервного копирования виртуальных машин Azure не требуется подключение к Интернету, поэтому, чтобы разрешить сетевую изоляцию, частные конечные точки не требуются.

Узнайте больше о частных конечных точках для Azure Backup [здесь](../articles/backup/private-endpoints.md).