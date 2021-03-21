---
title: Шаблоны Azure Resource Manager
description: Azure Resource Manager шаблоны для использования Azure Site Recoveryных функций.
services: site-recovery
author: rishjai
manager: gaggupta
ms.service: site-recovery
ms.topic: article
ms.date: 02/04/2021
ms.author: rishjai
ms.openlocfilehash: efd5c1ab3e23348373be76c79d1b9689bc5f4941
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101720534"
---
# <a name="azure-resource-manager-templates-for-azure-site-recovery"></a>Шаблоны Azure Resource Manager для Azure Site Recovery

В следующей таблице приведены ссылки на шаблоны Azure Resource Manager для использования Azure Site Recoveryных функций.

| Шаблон | Описание |
|---|---|
|**Из Azure в Azure** | |
| [создание хранилища служб восстановления](./quickstart-create-vault-template.md);| Создайте хранилище служб восстановления, Хранилище можно использовать для Azure Backup и Azure Site Recovery. |
| [Включение репликации для виртуальных машин Azure](https://aka.ms/asr-arm-enable-replication) | Включите репликацию для виртуальных машин Azure, используя существующее хранилище и настраиваемые целевые параметры.|
| [Активация отработки отказа и повторного включения защиты](https://aka.ms/asr-arm-failover-reprotect) | Активируйте операцию отработки отказа и повторного включения защиты для набора виртуальных машин Azure. |
| [Выполнение сквозного процесса аварийного восстановления для виртуальных машин Azure](https://aka.ms/asr-arm-e2e-flow) | Запустите комплексный поток аварийного восстановления (включите репликацию + отработку отказа и повторно включите защиту + восстановление размещения и повторное включение защиты) для виртуальных машин Azure, которые также называются потоком 540 °.|
|   |   |