---
title: Настройка данных на основе STIG для State Configuration службы автоматизации Azure
description: В этой статье рассказывается, как настроить данные на основе Стиг для конфигурации состояния службы автоматизации Azure.
keywords: dsc,powershell,конфигурация,установка
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: dab0cd7f7d660808b4ed7a91318baad55f80928c
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87015142"
---
# <a name="configure-data-based-on-security-technical-information-guide-stig"></a>Настройка данных на основе технического характера по безопасности (Стиг)

> Область применения: Windows PowerShell 5.1

Создание содержимого конфигурации в первый раз может быть непростой задачей.
Во многих случаях цель состоит в том, чтобы автоматизировать настройку серверов по некоторым базовым показателям в соответствии с отраслевыми рекомендациями.

> [!NOTE]
> В этой статье описывается решение, которое поддерживается сообществом ПО с открытым исходным кодом.
> Поддержка этого решения предоставляется только на форуме GitHub, но не средствами корпорации Майкрософт.

## <a name="community-project-powerstig"></a>Проект сообщества: PowerSTIG

Проект сообщества с именем [PowerSTIG](https://github.com/microsoft/powerstig) предназначен для решения этой проблемы путем создания содержимого DSC на основе [общедоступной информации](https://public.cyber.mil/stigs/) STIG (Security Technical Implementation Guide — Руководство по технической реализации систем безопасности).

Настройка по базовым показателям сложнее, чем может показаться.
Многим организациям приходится [документировать исключения](https://github.com/microsoft/powerstig#powerstigdata) из существующих правилам и управлять этими данными в большом масштабе.
PowerSTIG решает эту проблему, предоставляя [составные ресурсы](https://github.com/microsoft/powerstig#powerstigdsc) для каждого из аспектов конфигурации, вместо того чтобы собирать весь набор параметров в одном большом файле.

Созданные конфигурации можно применить в [скриптах конфигурации DSC](/powershell/scripting/dsc/configurations/configurations), чтобы создать MOF-файлы и [передать их в службу автоматизации Azure](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
Затем вы сможете зарегистрировать серверы [локально](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) или [в Azure](./automation-dsc-onboarding.md#enable-azure-vms) для получения этих конфигураций.

Чтобы поработать с PowerSTIG, откройте [коллекцию PowerShell](https://www.powershellgallery.com) и скачайте решение или щелкните ссылку "Сайт проекта" для просмотра [документации](https://github.com/microsoft/powerstig).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о DSC для PowerShell см. в статье [Общие сведения о Desired State Configuration Windows PowerShell](/powershell/scripting/dsc/overview/overview).
- Изучите [ресурсы](/powershell/scripting/dsc/resources/resources), предоставляемые для PowerShell DSC.
- Дополнительные сведения о конфигурации Local Configuration Manager см. в статье [Настройка локального диспетчера конфигураций](/powershell/scripting/dsc/managing-nodes/metaconfig).
