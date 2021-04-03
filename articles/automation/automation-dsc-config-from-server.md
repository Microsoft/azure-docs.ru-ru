---
title: Создание конфигураций на основе существующих серверов для конфигурации состояния службы автоматизации Azure
description: В этой статье рассказывается, как создать конфигурации на основе существующих серверов для конфигурации состояния службы автоматизации Azure.
keywords: dsc,powershell,конфигурация,установка
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 8f376fb383e50a39f0f12d45cf9b5ae47ad6fcbb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "86186509"
---
# <a name="create-configurations-from-existing-servers"></a>Создание конфигураций на основе существующих серверов

> Область применения: Windows PowerShell 5.1

Создание конфигураций на основе существующих серверов может оказаться сложной задачей.
Возможно, потребуются не *все параметры*, а только те, которые вас интересуют.
Даже в этом случае необходимо знать, в каком порядке должны быть применены параметры, чтобы конфигурация была успешно применена.

> [!NOTE]
> В этой статье используется решение, которое поддерживается сообществом ПО с открытым исходным кодом.
> Поддержка этого решения предоставляется только на форуме GitHub и не имеет отношения к корпорации Майкрософт.

## <a name="community-project-reversedsc"></a>Проект сообщества: ReverseDSC

Для работы с этой областью, запускающей SharePoint, было создано решение, поддерживаемое сообществом, с именем [ReverseDSC](https://github.com/microsoft/reversedsc).

В основе решения лежит [ресурс SharePointDSC](https://github.com/powershell/sharepointdsc). Решение расширяет его возможности для оркестрации [сбора информации](https://github.com/Microsoft/sharepointDSC.reverse#how-to-use) от существующих серверов SharePoint.
В последней версии имеется несколько [режимов извлечения](https://github.com/Microsoft/SharePointDSC.Reverse/wiki/Extraction-Modes), которые позволяют определить, какой уровень информации следует включить.

Результатом использования решения является создание [данных конфигурации](https://github.com/Microsoft/sharepointDSC.reverse#configuration-data) для использования с сценариями конфигурации SharePointDSC.

После создания файлов данных их можно применить в [сценариях конфигурации DSC](/powershell/scripting/dsc/overview/overview) для создания MOF-файлов и [передачи MOF-файлов в службу автоматизации Azure](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
Затем зарегистрируйте серверы [локально](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) или [в Azure](./automation-dsc-onboarding.md#enable-azure-vms) для получения конфигураций.

Чтобы испытать ReverseDSC в работе, откройте [коллекцию PowerShell](https://www.powershellgallery.com/packages/ReverseDSC/) и скачайте решение или щелкните ссылку "Сайт проекта" для перехода к [документации](https://github.com/Microsoft/sharepointDSC.reverse).

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о DSC для PowerShell см. в статье [Общие сведения о Desired State Configuration Windows PowerShell](/powershell/scripting/dsc/overview/overview).
- Изучите [ресурсы](/powershell/scripting/dsc/resources/resources), предоставляемые для PowerShell DSC.
- Дополнительные сведения о конфигурации Local Configuration Manager см. в статье [Настройка локального диспетчера конфигураций](/powershell/scripting/dsc/managing-nodes/metaconfig).
