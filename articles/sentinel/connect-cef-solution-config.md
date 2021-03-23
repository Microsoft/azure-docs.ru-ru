---
title: Настройка решения безопасности для подключения данных CEF к предварительной версии Azure Sentinel | Документация Майкрософт
description: Узнайте, как настроить решение безопасности для подключения данных CEF к Azure Sentinel.
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
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 60ce503d4a89f245f28d5034924cb8c89c926b3f
ms.sourcegitcommit: 2c1b93301174fccea00798df08e08872f53f669c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/22/2021
ms.locfileid: "104771300"
---
# <a name="step-2-configure-your-security-solution-to-send-cef-messages"></a>Шаг 2. Настройка решения безопасности для отправки сообщений CEF

На этом шаге будут выполнены необходимые изменения в конфигурации решения по обеспечению безопасности для отправки журналов агенту CEF.

## <a name="configure-a-solution-with-a-connector"></a>Настройка решения с помощью соединителя

Если в решении для обеспечения безопасности уже есть существующий соединитель, используйте инструкции, относящиеся к этому соединителю, как показано ниже.

- [Vectra AI Detect](connect-ai-vectra-detect.md)
- [Akamai Security Events](connect-akamai-security-events.md)
- [Aruba ClearPass](connect-aruba-clearpass.md)
- [Broadcom Symantec DLP](connect-broadcom-symantec-dlp.md)
- [Check Point](connect-checkpoint.md)
- [Cisco ASA](connect-cisco.md)
- [Citrix WAF](connect-citrix-waf.md)
- [CyberArk Enterprise Password Vault](connect-cyberark.md)
- [ExtraHop Reveal(x)](connect-extrahop.md)
- [F5 ASM](connect-f5.md)
- [Продукты Forcepoint](connect-forcepoint-casb-ngfw.md)
- [Fortinet](connect-fortinet.md)
- [Illusive Networks AMS](connect-illusive-attack-management-system.md)
- [Imperva WAF Gateway](connect-imperva-waf-gateway.md)
- [One Identity Safeguard](connect-one-identity.md)
- [Palo Alto Networks](connect-paloalto.md)
- [Thycotic Secret Server](connect-thycotic-secret-server.md)
- [Trend Micro Deep Security](connect-trend-micro.md)
- [Trend Micro TippingPoint](connect-trend-micro-tippingpoint.md)
- [Платформа сетевого судебного Вирекса](connect-wirex-systems.md)
- [Zscaler](connect-zscaler.md)
## <a name="configure-any-other-solution"></a>Настройка любого другого решения

Если соединитель не существует для конкретного решения безопасности, используйте следующие общие инструкции для перенаправления журналов в агент CEF.

1. Инструкции по настройке решения для отправки CEF сообщений см. в статье о конфигурации. Если решение отсутствует в списке, на устройстве необходимо задать эти значения, чтобы устройство отправляло необходимые журналы в нужном формате в агент syslog Sentinel Azure на основе агента Log Analytics. Вы можете изменить эти параметры на своем устройстве при условии, что они также будут изменены в управляющей программе syslog в агенте Sentinel Azure.
    - Протокол = TCP
    - Порт = 514
    - Format = CEF
    - IP-адрес. не забудьте отправить сообщения CEF по IP-адресу виртуальной машины, выделенной для этой цели.

   Это решение поддерживает syslog RFC 3164 или RFC 5424.

1. Чтобы найти события CEF в Log Analytics, введите `CommonSecurityLog` в окне запроса.

1. Перейдите к ШАГУ 3. [Проверка подключения](connect-cef-verify.md).

> [!NOTE]
> **Изменение источника поля TimeGenerated**
>
> - По умолчанию агент Log Analytics заполняет поле *timegenerated* в схеме временем получения агентом события из управляющей программы syslog. В результате время, когда событие было создано в исходной системе, не записывается в Azure Sentinel.
>
> - Тем не менее, можно выполнить следующую команду, которая загрузит и запустит `TimeGenerated.py` скрипт. Этот сценарий настраивает агент Log Analytics для заполнения поля *timegenerated* исходным временем события в исходной системе, а не временем получения агентом.
>
>    ```bash
>    wget -O TimeGenerated.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/TimeGenerated.py && python TimeGenerated.py {ws_id}
>    ```

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить устройства CEF к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Сведения о [сопоставлении полей CEF и CommonSecurityLog](cef-name-mapping.md).
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](./tutorial-detect-threats-built-in.md).