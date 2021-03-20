---
title: Упорядочение данных в большом масштабе для State Configuration службы автоматизации Azure
description: Эта статья содержит сведения об упорядочение данных в большом масштабе для State Configuration службы автоматизации Azure.
keywords: dsc,powershell,конфигурация,установка
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: bc55a4c5ab20cac041a00a0f924b207eb256ae8b
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86186526"
---
# <a name="configure-data-at-scale-for-azure-automation-state-configuration"></a>Упорядочение данных в большом масштабе для State Configuration службы автоматизации Azure

> Область применения: Windows PowerShell 5.1

Управление сотнями или тысячами серверов бывает очень сложной задачей.
Клиенты сообщают нам, что самым сложным аспектом обычно становится управление [данными конфигураций](/powershell/scripting/dsc/configurations/configdata).
Информацию нужно упорядочивать по таким логическим конструкциям, как расположение, тип и среда.

> [!NOTE]
> В этой статье рассматривается решение, которое поддерживается сообществом разработчиков ПО с открытым кодом.
> Поддержка этого решения предоставляется только на форуме GitHub и не имеет отношения к корпорации Майкрософт.

## <a name="community-project-datum"></a>Проект сообщества: Datum

Для решения этой задачи было создано поддерживаемое сообществом решение с именем [Datum](https://github.com/gaelcolas/Datum).
Решение Datum основано на замечательных идеях, заимствованных из других платформ управления конфигурацией, и реализует тот же подход для PowerShell DSC.
Сведения [организованы в текстовые файлы](https://github.com/gaelcolas/Datum#3-intended-usage) по логическим направлениям.
Пример будет следующим:

- параметры, которые нужно применять глобально;
- параметры, которые нужно применять ко всем серверам в расположении;
- параметры, которые нужно применять ко всем серверам баз данных;
- параметры отдельных серверов.

Все эти сведения можно упорядочить в удобном для вас формате файлов (JSON, YAML или PSD1).
Предоставляются командлеты, которые создают файлы конфигурации путем [объединения сведений](https://github.com/gaelcolas/Datum#datum-tree) из каждого файла в единое представление для сервера или роли сервера.

После создания файлов данных их можно применить в [скриптах конфигурации DSC](/powershell/scripting/dsc/configurations/write-compile-apply-configuration) для создания MOF-файлов и [передачи MOF-файлов в службу автоматизации Azure](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
Затем зарегистрируйте серверы [локально](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) или [в Azure](./automation-dsc-onboarding.md#enable-azure-vms) для получения конфигураций.

Чтобы испытать Datum в работе, откройте [коллекцию PowerShell](https://www.powershellgallery.com/packages/datum/) и скачайте решение или щелкните ссылку "Project Site" (Сайт проекта) для перехода к [документации](https://github.com/gaelcolas/Datum#2-getting-started--concepts).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о DSC для PowerShell см. в статье [Общие сведения о Desired State Configuration Windows PowerShell](/powershell/scripting/dsc/overview/overview).
- Изучите [ресурсы](/powershell/scripting/dsc/resources/resources), предоставляемые для PowerShell DSC.
- Дополнительные сведения о конфигурации Local Configuration Manager см. в статье [Настройка локального диспетчера конфигураций](/powershell/scripting/dsc/managing-nodes/metaconfig).
