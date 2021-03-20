---
title: Подключение защитника Майкрософт для данных конечной точки к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить Microsoft Defender для конечных точек (ранее — Microsoft Defender ATP) к Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/16/2020
ms.author: yelevin
ms.openlocfilehash: 0db4e0fe0472c75f1eae392980ae697f53007244
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98623372"
---
# <a name="connect-alerts-from-microsoft-defender-for-endpoint-formerly-microsoft-defender-atp"></a>Подключение оповещений от защитника Майкрософт к конечной точке (прежнее название — ATP в защитнике Майкрософт)

> [!IMPORTANT]
>
> - **Защитник Майкрософт для конечной точки** ранее назывался **Microsoft Defender Advanced Threat protection** или **мдатп**.
>
>     Вы можете увидеть, что старое имя по-прежнему используется в продукте (включая его соединитель данных в Azure Sentinel) в течение определенного периода времени.

[Защитник Майкрософт для соединителя конечной точки](/windows/security/threat-protection/microsoft-defender-atp/microsoft-defender-advanced-threat-protection) позволяет передавать оповещения от защитника Майкрософт для конечной точки в Azure Sentinel. Это позволит вам более детально анализировать события безопасности в Организации и создавать модули PlayBook для эффективного и немедленного реагирования.

> [!NOTE]
>
> Чтобы принять новые журналы необработанных данных от защитника Майкрософт для [расширенного](/windows/security/threat-protection/microsoft-defender-atp/advanced-hunting-overview)обнаружения конечной точки, используйте новый соединитель для Microsoft 365ного защитника (ранее — Microsoft Threat Protection, [см. документацию](./connect-microsoft-365-defender.md)).

## <a name="prerequisites"></a>Предварительные условия

- Необходимо иметь действительную лицензию для защитника Майкрософт для конечной точки, как описано в разделе [Настройка защитника Майкрософт для развертывания конечных точек](/windows/security/threat-protection/microsoft-defender-atp/licensing). 

- Вы должны быть глобальным администратором или администратором безопасности для клиента Sentinel Azure.

## <a name="connect-to-microsoft-defender-for-endpoint"></a>Подключение к защитнику Майкрософт для конечной точки

Если защитник Майкрософт для конечной точки развернут и принимает данные, оповещения можно легко передавать в Azure Sentinel.

1. В поле Метка Azure выберите **соединители данных**, выберите **защитник Майкрософт для конечной точки** (может по-прежнему называться *Microsoft Защитник Advanced Threat protection*) из коллекции и щелкните **открыть страницу соединителя**.

1. Нажмите кнопку **Подключить**. 

1. Чтобы запросить у защитника Майкрософт оповещения о конечных точках в **журналах**, введите **секуритялерт** в окне запроса и добавьте фильтр, где **имя поставщика** — **мдатп**.

## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить защитник Майкрософт для конечной точки к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](./tutorial-detect-threats-built-in.md).