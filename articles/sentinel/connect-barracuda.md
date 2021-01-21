---
title: Подключение данных Barracuda к Azure Sentinel | Документация Майкрософт
description: Узнайте, как использовать соединитель Barracuda Web Application Firewall (WAF) для подключения журналов Barracuda к Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 3b33b4aa-7286-4d79-b461-8e1812edc2e1
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/30/2019
ms.author: yelevin
ms.openlocfilehash: 1131c8e79d9df2883dcf7e975af9732c21318a49
ms.sourcegitcommit: a0c1d0d0906585f5fdb2aaabe6f202acf2e22cfc
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 01/21/2021
ms.locfileid: "98621170"
---
# <a name="connect-your-barracuda-waf-appliance"></a>Подключение устройства Barracuda WAF 

Соединитель брандмауэра веб-приложения Barracuda (WAF) позволяет легко подключать журналы Barracuda с помощью Sentinel Azure, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Это позволяет получить более подробные сведения о сети организации и улучшить возможности обеспечения безопасности. Azure Sentinel использует преимущества собственной интеграции между **Barracuda** и агентом log Analytics для обеспечения беспрепятственной интеграции. 

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-barracuda-waf"></a>Настройка и подключение Barracuda WAF
Брандмауэр веб-приложения Barracuda может интегрировать и экспортировать журналы непосредственно в Azure Sentinel с помощью агента Log Analytics.
1. Перейдите в [Barracuda WAF Configuration Flow](https://campus.barracuda.com/product/webapplicationfirewall/doc/73696965/configure-the-barracuda-web-application-firewall-to-integrate-with-the-oms-server-and-export-logs/)и следуйте инструкциям по настройке подключения, используя следующие параметры:
    - **Идентификатор рабочей области**: СКОПИРУЙТЕ значение идентификатора рабочей области со страницы соединителя Azure Sentinel Barracuda.
    - **Первичный ключ**. Скопируйте значение первичного ключа со страницы соединителя Azure Sentinel Barracuda.
1. Чтобы использовать соответствующую схему в Log Analytics для событий Barracuda, выполните поиск **CommonSecurityLog** и **barracuda_CL**.


## <a name="validate-connectivity"></a>Проверка подключения

Если журналы начнут появляться в Log Analytics, это может занять до 20 минут. 



## <a name="next-steps"></a>Следующие шаги
В этом документе вы узнали, как подключить устройства Barracuda к Azure Sentinel. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.
- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.


