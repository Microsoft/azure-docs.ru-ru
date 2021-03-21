---
title: Использование SAP на виртуальных машинах Linux | Документация Майкрософт
description: Узнайте об использовании SAP на виртуальных машинах в Microsoft Azure.
services: virtual-machines-linux,virtual-network,storage
documentationcenter: saponazure
author: MSSedusch
manager: gwallace
editor: ''
tags: azure-service-management
keywords: ''
ms.assetid: f9cd93dc-71ad-48a4-8778-4e48aec484a6
ms.service: virtual-machines-sap
ms.topic: conceptual
ms.tgt_pltfrm: vm-linux
ms.workload: na
ms.date: 10/04/2016
ms.author: sedusch
ms.openlocfilehash: bf0ae8bdd800f5fa43df73eca312504f56fc009a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101675385"
---
# <a name="using-sap-on-linux-virtual-machines-in-azure"></a>Использование SAP в виртуальных машинах Linux в Azure
Облачные вычисления — это широко используемый термин, который приобретает все большее значение в отрасли ИТ, начиная от малых компаний и заканчивая крупными и транснациональными корпорациями. Microsoft Azure — это созданная корпорацией Майкрософт платформа облачных служб, предоставляющая широкий спектр новых возможностей. Теперь клиенты могут быстро подготавливать и отзывать приложения в формате облачных служб без технических или бюджетных ограничений. Вместо того, чтобы тратить время и средства на создание и поддержку аппаратной инфраструктуры, компании могут сконцентрироваться на приложениях, бизнес-процессах, а также связанных преимуществах для клиентов и пользователей.

Корпорация Майкрософт предоставляет виртуальные машины Microsoft Azure как комплексную платформу "инфраструктура как услуга" (IaaS). Виртуальные машины Azure (IaaS) поддерживают приложения на основе SAP NetWeaver. Приведенная ниже техническая документация описывает планирование и реализацию приложений на основе SAP NetWeaver в виртуальных машинах Windows в Azure. Приложения на основе SAP NetWeaver можно также реализовать на [виртуальных машинах Windows](./get-started.md?toc=/azure/virtual-machines/windows/classic/toc.json).

[!INCLUDE [virtual-machines-common-classic-sap-get-started](../../../../includes/virtual-machines-common-classic-sap-get-started.md)]

## <a name="sap-netweaver-on-azure-suse-linux-virtual-machines"></a>SAP NetWeaver на виртуальных машинах Azure под управлением SUSE Linux
Заголовок. Тестирование SAP NetWeaver на виртуальных машинах Microsoft Azure под управлением SUSE Linux

Содержание. На данный момент времени официальной поддержки SAP для запуска SAP NetWeaver на виртуальных машинах Azure под управлением Linux нет. Тем не менее клиентам может потребоваться выполнить тестирование или демонстрационный запуск SAP или систем обучения на виртуальных машинах Azure Linux при условии, что нет необходимости для обращения в службу поддержки SAP. Эта статья поможет выполнить настройку виртуальных машин Azure под управлением SUSE Linux для запуска SAP и предоставляет некоторые базовые советы, которые помогут избежать потенциальных подводных камней.

Последнее обновление: декабрь 2015 г.

[Эту статью можно найти здесь](./sap-deployment-checklist.md?toc=/azure/virtual-machines/linux/toc.json)
