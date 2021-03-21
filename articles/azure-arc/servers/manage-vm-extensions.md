---
title: Управление расширениями виртуальных машин с помощью серверов с поддержкой дуги Azure
description: Серверы с поддержкой Arc Azure могут управлять развертыванием расширений виртуальных машин, которые обеспечивают настройку и задачи автоматизации после развертывания с помощью виртуальных машин, не относящихся к Azure.
ms.date: 03/01/2021
ms.topic: conceptual
ms.openlocfilehash: 039c52ccbee03636da0f5acc0fc5844be9b646f5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101687912"
---
# <a name="virtual-machine-extension-management-with-azure-arc-enabled-servers"></a>Управление расширениями виртуальных машин с помощью серверов с поддержкой Azure Arc

Расширения виртуальных машин — это небольшие приложения, которые обеспечивают настройку и задачи автоматизации, выполняемые после развертывания на виртуальных машинах Azure. Например, если для виртуальной машины требуется установка программного обеспечения, антивирусная защита или запуск скрипта, можно использовать расширение виртуальной машины.

Серверы с поддержкой Arc Azure позволяют развертывать расширения виртуальных машин Azure на виртуальных машинах Windows и Linux без Azure, упрощая управление гибридным компьютером с помощью жизненного цикла. Для управления расширениями виртуальной машины можно использовать следующие методы на гибридных компьютерах или серверах, управляемых с помощью серверов с поддержкой ARC:

- [портал Azure](manage-vm-extensions-portal.md).
- [Интерфейс командной строки Azure](manage-vm-extensions-cli.md)
- [Azure PowerShell](manage-vm-extensions-powershell.md)
- [Шаблоны диспетчер ресурсов](manage-vm-extensions-template.md) Azure

## <a name="key-benefits"></a>Основные преимущества

Поддержка расширения виртуальной машины на серверах с поддержкой дуги Azure обеспечивает следующие основные преимущества.

- Собирайте данные журналов для анализа с помощью [журналов в Azure Monitor](../../azure-monitor/logs/data-platform-logs.md) , включив расширение виртуальной машины агента log Analytics. Это полезно для выполнения сложного анализа данных из различных типов источников.

- С помощью [Azure Monitor для виртуальных машин](../../azure-monitor/vm/vminsights-overview.md)анализирует производительность виртуальных машин Windows и Linux и отслеживает их процессы и зависимости от других ресурсов и внешних процессов. Это достигается благодаря включению как агента Log Analytics, так и расширений виртуальной машины агента зависимостей.

- Скачайте и выполните скрипты на гибридных подключенных компьютерах с помощью расширения пользовательских сценариев. Это расширение можно использовать для настройки после развертывания, установки программного обеспечения и других задач настройки или управления.

- Автоматическое обновление сертификатов, хранящихся в [Azure Key Vault](../../key-vault/general/overview.md).

## <a name="availability"></a>Доступность

Функциональность расширения виртуальной машины доступна только в списке [поддерживаемых регионов](overview.md#supported-regions). Убедитесь, что ваш компьютер подключен в одном из этих регионов.

## <a name="extensions"></a>Расширения

В этом выпуске поддерживаются следующие расширения виртуальных машин на компьютерах Windows и Linux.

Сведения о пакете агента подключенного компьютера Azure и сведения о компоненте агента расширения см. в разделе [Обзор агентов](agent-overview.md#agent-component-details).

### <a name="windows-extensions"></a>Расширения Windows

|Расширение |Издатель |Type |Дополнительные сведения |
|----------|----------|-----|-----------------------|
|Средство проверки уязвимостей, интегрированное с защитником Azure |Qualys |Виндовсажент. Азуресекуритицентер |[Решение для оценки уязвимости в интегрированной службе "защитник Azure" для Azure и гибридных компьютеров](../../security-center/deploy-vulnerability-assessment-vm.md)|
|Расширение пользовательских сценариев |Microsoft.Compute; | CustomScriptExtension |[Расширение пользовательских сценариев Windows](../../virtual-machines/extensions/custom-script-windows.md)|
|Агент Log Analytics |Microsoft.EnterpriseCloud.Monitoring |MicrosoftMonitoringAgent |[Расширение виртуальной машины Log Analytics для Windows](../../virtual-machines/extensions/oms-windows.md)|
|Azure Monitor для виртуальных машин (аналитика) |Microsoft.Azure.Monitoring.DependencyAgent |депенденциажентвиндовс | [Расширение виртуальной машины агента зависимостей для Windows](../../virtual-machines/extensions/agent-dependency-windows.md)|
|Azure Key Vault синхронизации сертификатов | Microsoft. Azure. key. Vault |KeyVaultForWindows | [Расширение виртуальной машины Key Vault для Windows](../../virtual-machines/extensions/key-vault-windows.md) |
|Агент Azure Monitor |Microsoft. Azure. Monitor |азуремониторвиндовсажент |[Установка агента Azure Monitor (Предварительная версия)](../../azure-monitor/agents/azure-monitor-agent-install.md) |

### <a name="linux-extensions"></a>Расширения Linux

|Расширение |Издатель |Type |Дополнительные сведения |
|----------|----------|-----|-----------------------|
|Средство проверки уязвимостей, интегрированное с защитником Azure |Qualys |Линуксажент. Азуресекуритицентер |[Решение для оценки уязвимости в интегрированной службе "защитник Azure" для Azure и гибридных компьютеров](../../security-center/deploy-vulnerability-assessment-vm.md)|
|Расширение пользовательских сценариев |Microsoft. Azure. Extensions |CustomScript |[Расширение пользовательских сценариев Linux версии 2](../../virtual-machines/extensions/custom-script-linux.md) |
|Агент Log Analytics |Microsoft.EnterpriseCloud.Monitoring |OmsAgentForLinux |[Расширение виртуальной машины Log Analytics для Linux](../../virtual-machines/extensions/oms-linux.md) |
|Azure Monitor для виртуальных машин (аналитика) |Microsoft.Azure.Monitoring.DependencyAgent |DependencyAgentLinux |[Расширение виртуальной машины агента зависимостей для Linux](../../virtual-machines/extensions/agent-dependency-linux.md) |
|Azure Key Vault синхронизации сертификатов | Microsoft. Azure. key. Vault |KeyVaultForLinux | [Расширение виртуальной машины Key Vault для Linux](../../virtual-machines/extensions/key-vault-linux.md) |
|Агент Azure Monitor |Microsoft. Azure. Monitor |азуремониторлинуксажент |[Установка агента Azure Monitor (Предварительная версия)](../../azure-monitor/agents/azure-monitor-agent-install.md) |

## <a name="prerequisites"></a>Предварительные условия

Эта функция зависит от следующих поставщиков ресурсов Azure в вашей подписке:

- **Microsoft.HybridCompute**
- **Microsoft.GuestConfiguration**

Если они еще не зарегистрированы, выполните действия, описанные в разделе [Регистрация поставщиков ресурсов Azure](agent-overview.md#register-azure-resource-providers).

Обязательно ознакомьтесь с документацией по каждому расширению виртуальной машины, упомянутому в предыдущей таблице, чтобы понять, есть ли требования к сети или к системе. Это позволит избежать проблем с подключением к службе или компоненту Azure, которые используют это расширение виртуальной машины.

### <a name="log-analytics-vm-extension"></a>Расширение виртуальной машины Log Analytics

Для расширения виртуальной машины агента Log Analytics для Linux требуется, чтобы на целевом компьютере был установлен Python 2. x.

### <a name="azure-key-vault-vm-extension-preview"></a>Расширение виртуальной машины Azure Key Vault (Предварительная версия)

Расширение Key Vault VM (Предварительная версия) не поддерживает следующие операционные системы Linux:

- CentOS Linux 7 (x64);
- Red Hat Enterprise Linux (RHEL) 7 (x64)
- Amazon Linux 2 (x64)

Развертывание расширения Key Vault VM (Предварительная версия) поддерживается только с помощью:

- Azure CLI
- Azure PowerShell
- Шаблон Azure Resource Manager

Перед развертыванием расширения необходимо выполнить следующие действия.

1. [Создайте хранилище и сертификат](../../key-vault/certificates/quick-create-portal.md) (самозаверяющий или импортированный).

2. Предоставьте серверу с поддержкой Arc Azure доступ к секрету сертификата. Если вы используете [предварительную версию RBAC](../../key-vault/general/rbac-guide.md), найдите имя ресурса Arc Azure и назначьте его роли **пользователя Key Vault секреты (Предварительная версия)** . Если вы используете [политику доступа Key Vault](../../key-vault/general/assign-access-policy-portal.md), назначьте ей разрешения на **Получение** секрета для назначенного системе удостоверения ресурса дуги Azure.

### <a name="connected-machine-agent"></a>Агент подключенного компьютера

Убедитесь, что компьютер соответствует [поддерживаемым версиям](agent-overview.md#supported-operating-systems) операционной системы Windows и Linux для агента подключенного компьютера Azure.

Минимальная версия агента подключенного компьютера, поддерживаемая с этой функцией в Windows и Linux, — это выпуск 1,0.

Сведения об обновлении компьютера до версии, необходимой для агента, см. в разделе [Upgrade Agent](manage-agent.md#upgrading-agent).

## <a name="next-steps"></a>Дальнейшие действия

Расширения виртуальных машин можно развертывать, администрировать и удалять с помощью [Azure CLI](manage-vm-extensions-cli.md), [Azure PowerShell](manage-vm-extensions-powershell.md), из [портал Azure](manage-vm-extensions-portal.md)или [шаблонов Azure Resource Manager](manage-vm-extensions-template.md).
