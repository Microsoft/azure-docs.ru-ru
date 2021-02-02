---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 01/25/2021
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 0e5edcfad636c76107adc7e72b50ea61249aa00a
ms.sourcegitcommit: fc8ce6ff76e64486d5acd7be24faf819f0a7be1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/26/2021
ms.locfileid: "98807172"
---
|Имя |Описание |Политики |Версия |
|---|---|---|---|
|[Развертывание. Настройка необходимых компонентов для включения Azure Monitor и агентов безопасности Azure на виртуальных машинах](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitoring_Prerequisites.json) |Настройка компьютеров для автоматической установки Azure Monitor и агентов безопасности Azure. Центр безопасности собирает события, поступающие от агентов, и предоставляет на их основе оповещения системы безопасности и специализированные задачи для усиления защиты (рекомендации). Создайте группу ресурсов и рабочую область Log Analytics в том же регионе, где находится компьютер с записями аудита. Эта политика применяется только к виртуальным машинам в нескольких регионах. |5 |1.0.0 (предварительная версия) |
|[Включение Azure Monitor для масштабируемых наборов виртуальных машин](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitor_VMSS.json) |Включите Azure Monitor для масштабируемых наборов виртуальных машин в указанной области (группа управления, подписка или группа ресурсов). В качестве параметра принимается рабочая область Log Analytics. Примечание. Если параметру upgradePolicy масштабируемого набора присвоено значение Manual, необходимо применить расширение для всех виртуальных машин в наборе, вызвав их обновление. В интерфейсе командной строки для этого потребовалось бы ввести команду az vmss update-instances. |6 |1.0.1 |
|[Включение Azure Monitor для виртуальных машин](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Monitoring/AzureMonitor_VM.json) |Включение Azure Monitor для виртуальных машин (ВМ) в указанной области (группа управления, подписка или группа ресурсов). В качестве параметра принимается рабочая область Log Analytics. |10 |2.0.0 |
