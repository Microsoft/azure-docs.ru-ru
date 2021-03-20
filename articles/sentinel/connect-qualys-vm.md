---
title: Подключение данных управления уязвимостью Qualys к Azure Sentinel | Документация Майкрософт
description: Узнайте, как подключить данные управления уязвимостью Qualys к Azure Sentinel.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/17/2020
ms.author: yelevin
ms.openlocfilehash: 8e11c4182520e143007b46c8a7907b5e71bfb27d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100097591"
---
# <a name="connect-your-qualys-vm-to-azure-sentinel-with-azure-function"></a>Подключение виртуальной машины Qualys к Azure Sentinel с помощью функции Azure

> [!IMPORTANT]
> Соединитель данных виртуальной машины Qualys в Azure Sentinel в настоящее время находится в общедоступной предварительной версии.
> Эта функция предоставляется без соглашения об уровне обслуживания и не рекомендуется для рабочих нагрузок. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены. Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Соединитель управления уязвимостью Qualys (VM) позволяет легко подключать все журналы решений безопасности [виртуальной машины Qualys](https://www.qualys.com/apps/vulnerability-management/) с помощью Sentinel Azure, просматривать панели мониторинга, создавать пользовательские оповещения и улучшать исследование. Интеграция между виртуальной машиной Qualys и Azure Sentinel использует функции Azure для извлечения данных журнала с помощью REST API.

> [!NOTE]
> Данные будут храниться в географическом расположении рабочей области, на которой вы используете метку Azure.

## <a name="configure-and-connect-qualys-vm"></a>Настройка и подключение виртуальной машины Qualys

Функции Azure могут интегрировать и получать события и журналы непосредственно из виртуальной машины Qualys и пересылать их в Azure Sentinel.

1. На портале Sentinel Azure щелкните **соединители данных** и выберите **Qualys уязвимость Management Connector (соединитель управления уязвимостью** ).

1. Выберите **открыть страницу соединителя**.

1. Следуйте инструкциям на странице **управления уязвимостью Qualys** .

## <a name="find-your-data"></a>Поиск данных

После установки успешного подключения данные появляются в Log Analytics в таблице **QualysHostDetection_CL** .

## <a name="validate-connectivity"></a>Проверка подключения

После того, как журналы начнут отображаться в Log Analytics, может пройти до 20 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этом документе вы узнали, как подключить виртуальную машину Qualys к Sentinel Azure с помощью приложений функций Azure. Ознакомьтесь с дополнительными сведениями об Azure Sentinel в соответствующих статьях.

- Узнайте, как [отслеживать свои данные и потенциальные угрозы](quickstart-get-visibility.md).
- Узнайте, как приступить к [обнаружению угроз с помощью Azure Sentinel](tutorial-detect-threats-built-in.md).
- [Используйте книги](tutorial-monitor-your-data.md) для мониторинга данных.
