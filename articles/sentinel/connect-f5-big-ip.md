---
title: Подключение данных больших IP-адресов F5 к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить данные больших IP-адресов F5 к Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: d068223f-395e-46d6-bb94-7ca1afd3503c
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 17c31bbe54326962c26ab53f702cbd28d1e36c6d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100092797"
---
# <a name="connect-your-f5-big-ip-appliance"></a>Подключение устройства F5 с большим IP-адресом 

Соединитель F5 BIG-IP позволяет легко подключать все журналы больших IP-адресов F5 с помощью Azure Sentinel, просматривать книги, создавать пользовательские оповещения и улучшать исследование. Это позволяет получить более подробные сведения о сети организации и улучшить возможности обеспечения безопасности. Интеграция между F5 BIG-IP и Sentinel Azure использует REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-f5-big-ip"></a>Настройка и подключение F5 BIG-IP 

F5 BIG-IP может интегрировать и экспортировать журналы непосредственно в Azure Sentinel.

1. На портале Sentinel Azure щелкните **соединители данных** и выберите **F5 BIG-IP** , а затем **откройте страницу соединителя**. 
1. Чтобы подключиться к серверу F5 BIG-IP, необходимо опубликовать объявление JSON в конечной точке API системы. Инструкции по выполнению этой задачи см. в статье [Интеграция команды F5 BIG-IP с Sentinel](https://devcentral.f5.com/s/articles/Integrating-the-F5-BIGIP-with-Azure-Sentinel).
8. На странице соединителя F5 BIG-IP скопируйте идентификатор рабочей области и первичный ключ и вставьте их, как описано в разделе [потоковая передача данных в Azure log Analytics](https://devcentral.f5.com/s/articles/Integrating-the-F5-BIGIP-with-Azure-Sentinel#streaming-data-to-azure-log-analytics).
1. После выполнения инструкций F5 BIG-IP на странице соединителя Sentinel Azure отображаются подключенные типы данных.
1. Чтобы использовать соответствующую схему в Log Analytics для событий больших IP-адресов F5, выполните поиск **F5Telemetry_LTM_CL**, **F5Telemetry_system_CL** и **F5Telemetry_ASM_CL**.


## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут. 



## <a name="next-steps"></a>Дальнейшие действия
В этом документе вы узнали, как подключить метку F5 BIG-IP к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.


