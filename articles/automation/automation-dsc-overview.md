---
title: Обзор службы State Configuration службы автоматизации Azure
description: Эта статья содержит общие сведения о службе State Configuration службы автоматизации Azure.
keywords: PowerShell DSC, настройка требуемого состояния, PowerShell DSC для Azure
services: automation
ms.service: automation
ms.subservice: dsc
author: mgoedtel
ms.author: magoedte
ms.date: 01/26/2021
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 752d7f86941967c218b3a57fa163698b9f502057
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98897026"
---
# <a name="azure-automation-state-configuration-overview"></a>Обзор службы State Configuration службы автоматизации Azure

State Configuration службы автоматизации Azure — это служба управления конфигурацией Azure, которая позволяет создавать и компилировать [конфигурации](/powershell/scripting/dsc/configurations/configurations) PowerShell Desired State Configuration (DSC) и управлять ими для узлов в любом облаке или локальном центре обработки данных. Служба также импортирует [ресурсы DSC](/powershell/scripting/dsc/resources/resources) и назначает конфигурации целевым узлам, и все это в облаке. Вы можете получить доступ к службе State Configuration службы автоматизации Azure на портале Azure, выбрав **State Configuration (DSC)** в разделе **Управление конфигурацией**.

Службу State Configuration службы автоматизации Azure можно использовать для управления разными компьютерами:

- Виртуальные машины Azure
- Виртуальные машины Azure (классические).
- Физические или виртуальные машины под управлением Windows, расположенные локально или в облачной службе, отличной от Azure (в том числе экземплярах AWS EC2)
- Физические или виртуальные машины под управлением Linux, расположенные локально, в Azure или облачной службе, отличной от Azure.

Если вы не готовы управлять конфигурацией компьютера из облака, служба State Configuration службы автоматизации Azure может также использоваться как конечная точка только для отчетности. Эта функция позволяет задавать (отправлять) конфигурации через DSC и просматривать сведения об отчетах в службе автоматизации Azure.

> [!NOTE]
> Управление виртуальными машинами Azure с помощью службы State Configuration службы автоматизации Azure включается без дополнительной платы, если установлена версия расширения Desired State Configuration для виртуальной машины Azure позднее 2.70. См. дополнительные сведения на [**странице с ценами на службу автоматизации**](https://azure.microsoft.com/pricing/details/automation/).

## <a name="why-use-azure-automation-state-configuration"></a>Преимущества службы "Настройка состояния службы автоматизации Azure"

Служба State Configuration службы автоматизации Azure предоставляет ряд преимуществ по сравнению с использованием DSC за пределами Azure. Она обеспечивает быструю и простую масштабируемость тысяч компьютеров из безопасного центрального расположения. Вы можете легко переносить компьютеры в облачную среду, присваивать им декларативные конфигурации, а также просматривать отчеты, отражающие соответствие каждого компьютера требуемому состоянию.

Служба State Configuration службы автоматизации Azure связана с DSC так же, как модули runbook со скриптами PowerShell. Другими словами, служба автоматизации Azure помогает управлять сценариями PowerShell так же, как и конфигурациями DSC.

### <a name="built-in-pull-server"></a>Встроенный опрашивающий сервер

Служба State Configuration службы автоматизации Azure предоставляет опрашивающий сервер DSC аналогично [компоненту Windows DSC-Service](/powershell/scripting/dsc/pull-server/pullserver). Целевые узлы могут автоматически получать конфигурации, переходить в требуемое состояние и сообщать о соответствии. Встроенный опрашивающий сервер в службе автоматизации Azure избавляет от необходимости устанавливать и обслуживать собственный опрашивающий сервер. Служба автоматизации Azure может работать с виртуальными и физическими машинами под управлением Windows или Linux, размещенными в облаке или локально.

### <a name="management-of-all-your-dsc-artifacts"></a>Управление всеми артефактами DSC

Служба State Configuration службы автоматизации Azure обеспечивает тот же слой управления для [Desired State Configuration PowerShell](/powershell/scripting/dsc/overview/overview), который служба автоматизации Azure предлагает для скриптов PowerShell. Вы можете управлять всеми конфигурациями, ресурсами и целевыми узлами DSC с помощью портала Azure или PowerShell.

![Снимок экрана со страницей службы автоматизации Azure](./media/automation-dsc-overview/azure-automation-blade.png)

### <a name="import-of-reporting-data-into-azure-monitor-logs"></a>Импорт данных отчетов в журналы Azure Monitor

Узлы, управление которыми осуществляется с помощью "Настройка состояния службы автоматизации Azure", отправляют подробные отчеты с данными о состоянии на встроенный опрашивающий сервер. В службе "Настройка состояния службы автоматизации Azure" можно настроить отправку этих данных в рабочую область Log Analytics. См. раздел [Пересылка данных отчетов службы State Configuration службы автоматизации Azure в журналах Azure Monitor](automation-dsc-diagnostics.md).

## <a name="prerequisites"></a>Предварительные условия

При использовании службы State Configuration службы автоматизации Azure учитывайте требования, описанные в этом разделе.

### <a name="operating-system-requirements"></a>Требования к операционной системе

Для узлов под управлением Windows поддерживаются следующие версии:

- Windows Server 2019
- Windows Server 2016
- Windows Server 2012 R2;
- Windows Server 2012
- Windows Server 2008 R2 с пакетом обновления 1 (SP1)
- Windows 10
- Windows 8.1
- Windows 7

>[!NOTE]
>Номер SKU автономного продукта [Microsoft Hyper-V Server](/windows-server/virtualization/hyper-v/hyper-v-server-2016) не содержит реализацию DSC. Поэтому управлять им с помощью PowerShell DSC или службы State Configuration службы автоматизации Azure невозможно.

Для узлов под управлением Linux расширение DSC для Linux поддерживает все дистрибутивы Linux, перечисленные в [документации по DSC PowerShell](/powershell/scripting/dsc/getting-started/lnxgettingstarted).

### <a name="dsc-requirements"></a>Требования DSC

Для всех узлов Windows, работающих в Azure, [WMF 5.1](/powershell/scripting/wmf/setup/install-configure) устанавливается при включении компьютеров. Для узлов, работающих под управлением Windows Server 2012 и Windows 7, включен [WinRM](/powershell/scripting/dsc/troubleshooting/troubleshooting#winrm-dependency).

Для всех узлов Linux, работающих в Azure, [PowerShell DSC для Linux](https://github.com/Microsoft/PowerShell-DSC-for-Linux) устанавливается при включении компьютеров.

### <a name="configuration-of-private-networks"></a><a name="network-planning"></a>Настройка частных сетей

Дополнительные сведения о портах, URL-адресах и других данных о сети, необходимых для узлов в частной сети, см. в [конфигурации сети службы автоматизации Azure](automation-network-configuration.md#hybrid-runbook-worker-and-state-configuration) .

#### <a name="proxy-support"></a>Поддержка прокси-сервера

Поддержка прокси-сервера для агента DSC доступна в Windows версии 1809 и более поздних версиях. Этот параметр включается путем установки значений свойств `ProxyURL` и `ProxyCredential` в [скрипте метаконфигурации](automation-dsc-onboarding.md#generate-dsc-metaconfigurations), используемом для регистрации узлов.

>[!NOTE]
>В службе State Configuration службы автоматизации Azure не предусмотрена поддержка прокси-сервера DSC для предыдущих версий Windows.

Для узлов Linux агент DSC поддерживает прокси-сервер и использует переменную `http_proxy` для определения URL-адреса. Дополнительные сведения о поддержке прокси-сервера см. в разделе [Создание метаконфигураций DSC](automation-dsc-onboarding.md#generate-dsc-metaconfigurations).

#### <a name="dns-records-per-region"></a>Записи DNS для каждого региона

При определении исключений рекомендуется использовать адреса, перечисленные в таблице [записей DNS для каждого региона](how-to/automation-region-dns-records.md) .

## <a name="next-steps"></a>Дальнейшие действия

- Чтобы приступить к работе, см. статью [Начало работы со службой State Configuration службы автоматизации Azure](automation-dsc-getting-started.md).
- Сведения о том, как включить узлы, см. в статье [Включение службы State Configuration службы автоматизации Azure](automation-dsc-onboarding.md).
- Сведения о компилировании конфигураций DSC, которые затем можно назначить целевым узлам, см. в статье [Компилирование конфигураций DSC в службе State Configuration службы автоматизации Azure](automation-dsc-compile.md).
- Пример использования службы State Configuration в службе автоматизации Azure в конвейере непрерывного развертывания см. в статье [Настройка непрерывного развертывания с помощью Chocolatey](automation-dsc-cd-chocolatey.md).
- Сведения о ценах см. на странице [с ценами на использование State Configuration службы автоматизации Azure](https://azure.microsoft.com/pricing/details/automation/).
- Справочник по командлетам PowerShell см. в документации по [Az.Automation](/powershell/module/az.automation).
