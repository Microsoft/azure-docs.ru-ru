---
title: Платформы, поддерживаемые Центром безопасности Azure | Документация Майкрософт
description: Этот документ содержит список платформ, работу с которыми поддерживает Центр безопасности Azure.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: overview
ms.date: 03/31/2020
ms.author: memildin
ms.openlocfilehash: 88dc0760f320a99b0cbc99b7637dc34dd11dfecc
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "103465452"
---
# <a name="supported-platforms"></a>Поддерживаемые платформы 

На этой странице указаны платформы и среды, поддерживаемые Центром безопасности Azure.

## <a name="combinations-of-environments"></a>Сочетания сред <a name="vm-server"></a>

Центр безопасности Azure поддерживает работу с виртуальными машинами и серверами в различных гибридных средах:

* только Azure;
* Azure и локальная среда;
* Azure и другие облака;
* Azure, другие облака и локальная среда.

Для среды Azure, активированной в подписке Azure, Центр безопасности Azure автоматически определяет ресурсы IaaS, развернутые в подписке.

## <a name="supported-operating-systems"></a>Поддерживаемые операционные системы

Работа Центра безопасности зависит от [агента Log Analytics](../azure-monitor/agents/agents-overview.md#log-analytics-agent). Убедитесь, что компьютеры работают под управлением одной из операционных систем, поддерживаемых этим агентом, как описано на следующих страницах:

* [Поддерживаемые операционные системы: Windows](../azure-monitor/agents/agents-overview.md#supported-operating-systems)
* [Поддерживаемые операционные системы: Linux](../azure-monitor/agents/agents-overview.md#supported-operating-systems)

Кроме того, убедитесь, что агент Log Analytics [правильно настроен для отправки данных в Центр безопасности](security-center-enable-data-collection.md#manual-agent).

Дополнительные сведения об отдельных функциях Центра безопасности, доступных в Windows и Linux, см. в статье [Поддерживаемые функции для виртуальных машин](security-center-services.md).

> [!NOTE]
> Хотя инструмент Azure Defender предназначен для защиты серверов, большинство возможностей **Azure Defender для серверов** поддерживаются на компьютерах Windows 10. Одна из неподдерживаемых в настоящее время функций — это [интегрированное решение EDR Центра безопасности: Microsoft Defender для конечных точек](security-center-wdatp.md).

## <a name="managed-virtual-machine-services"></a>Виртуальные машины в управляемых службах <a name="virtual-machine"></a>

В клиентской подписке виртуальные машины также создаются как часть некоторых управляемых Azure служб, таких как Azure Kubernetes (AKS), Azure Databricks и т. д. Центр безопасности находит такие виртуальные машины, и на них можно установить и настроить агент Log Analytics, если доступна поддерживаемая ОС.

## <a name="cloud-services"></a>Облачные службы <a name="cloud-services"></a>

Виртуальные машины, запущенные в облачной службе, также поддерживаются. Мониторинг выполняется только для облачных служб и рабочих ролей, запущенных в слотах рабочей среды. Дополнительные сведения см. в статье [Общие сведения об облачных службах Azure](../cloud-services/cloud-services-choose-me.md).

Кроме того, обеспечивается защита виртуальных машин, находящихся в Azure Stack Hub. Дополнительные сведения об интеграции Центра безопасности с Azure Stack Hub см. в разделе [Подключение виртуальных машин Azure Stack Hub к Центру безопасности](quickstart-onboard-machines.md?pivots=azure-portal#onboard-your-azure-stack-hub-vms). 

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте, как [в Центре безопасности собираются данные с помощью агента Log Analytics](security-center-enable-data-collection.md).
- Узнайте, как [Центр безопасности управляет данными и защищает их](security-center-data-security.md).
- Узнайте, как [спланировать работу в Центре безопасности Azure, и получите рекомендации по переходу к его использованию](security-center-planning-and-operations-guide.md).