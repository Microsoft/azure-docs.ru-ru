---
title: Перенос виртуальных машин в диспетчер ресурсов с помощью Azure CLI
description: В этой статье рассматриваются поддерживаемые платформой перенос ресурсов из классической модели в Azure Resource Manager с помощью Azure CLI.
author: tanmaygore
manager: vashan
ms.service: virtual-machines
ms.subservice: classic-to-arm-migration
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.custom: devx-track-azurecli
ms.openlocfilehash: 671b27f927c91397d2aacd98cb7b500d8197d1c5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101669341"
---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-azure-cli"></a>Перенос ресурсов IaaS из классического развертывания в развертывание с помощью Azure Resource Manager с использованием Azure CLI

> [!IMPORTANT]
> Сегодня около 90% виртуальных машин IaaS используют [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). По состоянию на 28 февраля 2020, классические виртуальные машины являются устаревшими и будут полностью сняты с 1 марта 2023. [Узнайте больше]( https://aka.ms/classicvmretirement) об этой нерекомендуемости и [о том, как она влияет на вас](classic-vm-deprecation.md#how-does-this-affect-me).

Ниже последовательно описано, как использовать команды интерфейса командной строки Azure (CLI) для переноса ресурсов IaaS из классической модели развертывания в модель развертывания с помощью Azure Resource Manager. Для выполнения инструкций в этой статье требуется [классический Azure CLI](/cli/azure/install-classic-cli). Так как Azure CLI применим только для ресурсов Azure Resource Manager, его нельзя использовать для этой миграции.

> [!NOTE]
> Все операции, описанные здесь, являются идемпотентными. Если вы столкнетесь с какой-либо проблемой, не связанной с неподдерживаемой функцией или ошибкой конфигурации, мы рекомендуем повторить подготовку, прервать или зафиксировать текущую операцию. Платформа повторит попытку.
> 
> 

<br>
Ниже приведена блок-схема с последовательностью действий во время переноса.

![Screenshot that shows the migration steps](./media/migration-classic-resource-manager/migration-flow.png)

## <a name="step-1-prepare-for-migration"></a>Шаг 1. Подготовка к переносу
Ниже приведено несколько рекомендаций для оценки переноса ресурсов IaaS из классической модели в модель Resource Manager.

* Прочитайте [список неподдерживаемых конфигураций и компонентов](migration-classic-resource-manager-overview.md). Если у вас есть виртуальные машины, которые используют неподдерживаемые конфигурации или компоненты, мы рекомендуем отложить перенос до того момента, пока не будет заявлено об их поддержке. Также вы можете удалить такую функцию или вынести ее за пределы конфигурации, чтобы выполнить перенос.
* Если у вас есть текущие автоматизированные сценарии, которые развертывают инфраструктуру и приложения, попробуйте создать аналогичную программу установки для миграции с помощью этих сценариев. Вы можете также настроить примеры среды с помощью портала Azure.

> [!IMPORTANT]
> В настоящее время не поддерживается перенос шлюзов приложений из классической модели в модель Resource Manager. Чтобы перенести классическую виртуальную сеть со шлюзом приложений, удалите этот шлюз перед выполнением операции подготовки для перемещения сети. После завершения переноса повторно подключите шлюз в Azure Resource Manager. 
>
>Шлюзы ExpressRoute, подключенные к каналам ExpressRoute в другой подписке, перенести автоматически невозможно. В таких случаях удалите шлюз ExpressRoute, перенесите виртуальную сеть и создайте шлюз заново. Дополнительные сведения см. в статье [Перенос каналов ExpressRoute и связанных виртуальных сетей из классической модели развертывания на модель Resource Manager](../expressroute/expressroute-migration-classic-resource-manager.md).
> 
> 

## <a name="step-2-set-your-subscription-and-register-the-provider"></a>Шаг 2. Настройка подписки и регистрация поставщика
Для сценариев миграции следует настроить среду для классической модели и модели Resource Manager. [Установите интерфейс командной строки Azure](/cli/azure/install-classic-cli) (Azure CLI) и [выберите подписку](/cli/azure/authenticate-azure-cli).

Выполните вход со своей учетной записью.

```azurecli
azure login
```

Выберите подписку Azure с помощью следующей команды.

```azurecli
azure account set "<azure-subscription-name>"
```

> [!NOTE]
> Регистрация — однократное действие, но, прежде чем выполнять миграцию, вам нужно зарегистрироваться. Если вы не зарегистрируетесь, отобразится такое сообщение об ошибке: 
> 
> *Неправильный запрос: Подписка не зарегистрирована для миграции.* 
> 
> 

Зарегистрируйтесь в поставщике ресурсов миграции с помощью такой команды: Обратите внимание, что в некоторых случаях эта команда истекает. Однако регистрация будет успешной.

```azurecli
azure provider register Microsoft.ClassicInfrastructureMigrate
```

Подождите пять минут для завершения регистрации. Состояние утверждения регистрации можно проверить, выполнив следующую команду. Убедитесь, что RegistrationState имеет значение `Registered` , прежде чем продолжить.

```azurecli
azure provider show Microsoft.ClassicInfrastructureMigrate
```

Теперь переключите интерфейс командной строки в режим `asm`.

```azurecli
azure config mode asm
```

## <a name="step-3-make-sure-you-have-enough-azure-resource-manager-virtual-machine-vcpus-in-the-azure-region-of-your-current-deployment-or-vnet"></a>Шаг 3. Проверка наличия достаточного числа виртуальных ЦП для виртуальных машин Azure, развертываемых с помощью Resource Manager, в регионе Azure, в котором находится текущее развертывание или виртуальная сеть
Для этого шага необходимо будет переключиться в режим `arm`. Для этого воспользуйтесь следующей командой.

```azurecli
azure config mode arm
```

Чтобы проверить текущее количество виртуальных ЦП в Azure Resource Manager, используйте приведенную ниже команду CLI. Чтобы узнать больше о квотах на виртуальные ЦП, см. соответствующий раздел статьи [Подписка Azure, границы, квоты и ограничения службы](../azure-resource-manager/management/azure-subscription-service-limits.md#managing-limits).

```azurecli
azure vm list-usage -l "<Your VNET or Deployment's Azure region"
```

После завершения проверки на этом шаге можно переключиться обратно в режим `asm` .

```azurecli
azure config mode asm
```

## <a name="step-4-option-1---migrate-virtual-machines-in-a-cloud-service"></a>Шаг 4. Вариант 1: миграция виртуальных машин в облачной службе
Получите список облачных служб, выполнив следующую команду, а затем выберите облачную службу для переноса. Обратите внимание: если виртуальные машины в облачной службе размещены в виртуальной сети или им назначены веб-роли или рабочие роли, вы получите сообщение об ошибке.

```azurecli
azure service list
```

Выполните следующую команду, чтобы получить имя развертывания для облачной службы из подробных выходных данных. В большинстве случаев имя развертывания совпадает с именем облачной службы.

```azurecli
azure service show <serviceName> -vv
```

Во-первых, проверьте возможность переноса облачной службы с помощью следующей команды.

```shell
azure service deployment validate-migration <serviceName> <deploymentName> new "" "" ""
```

Подготовьте к переносу виртуальные машины в облачной службе. Возможно два варианта.

Если вы хотите перенести виртуальные машины в виртуальную сеть, созданную платформой, используйте следующую команду.

```azurecli
azure service deployment prepare-migration <serviceName> <deploymentName> new "" "" ""
```

Если вы хотите перенести их в существующую виртуальную сеть в модели развертывания с помощью Resource Manager, используйте следующую команду.

```azurecli
azure service deployment prepare-migration <serviceName> <deploymentName> existing <destinationVNETResourceGroupName> <subnetName> <vnetName>
```

Когда операция подготовки успешно завершится, вы сможете просмотреть подробные выходные данные о состоянии миграции виртуальных машин, чтобы убедиться, что все они находятся в состоянии `Prepared` .

```azurecli
azure vm show <vmName> -vv
```

Проверьте конфигурацию для подготовленных ресурсов с помощью интерфейса командной строки или портала Azure. Если вы не готовы к миграции и хотите вернуть предыдущее состояние, используйте следующую команду.

```azurecli
azure service deployment abort-migration <serviceName> <deploymentName>
```

Если подготовленная конфигурация вас устраивает, можете продолжать процесс. Выполните следующую команду, чтобы зафиксировать ресурсы.

```azurecli
azure service deployment commit-migration <serviceName> <deploymentName>
```

## <a name="step-4-option-2----migrate-virtual-machines-in-a-virtual-network"></a>Шаг 4. Вариант 2: миграция виртуальных машин в виртуальной сети
Выберите виртуальную сеть, в которую будете переносить ресурсы. Обратите внимание: если в виртуальной сети есть виртуальные машины, веб-роли или рабочие роли с неподдерживаемыми конфигурациями, вы получите сообщение об ошибке проверки.

Выполните следующую команду, чтобы получить все виртуальные сети в подписке.

```azurecli
azure network vnet list
```

Результат должен выглядеть следующим образом.

![Снимок экрана командной строки с выделенным полным именем виртуальной сети.](./media/virtual-machines-linux-cli-migration-classic-resource-manager/vnet.png)

В приведенном выше примере **virtualNetworkName** представляет собой полное имя **Group classicubuntu16 classicubuntu16**.

Во-первых, проверьте возможность переноса виртуальной сети с помощью следующей команды.

```shell
azure network vnet validate-migration <virtualNetworkName>
```

Подготовьте выбранную виртуальную сеть к переносу, используя следующую команду.

```azurecli
azure network vnet prepare-migration <virtualNetworkName>
```

Проверьте конфигурацию для подготовленных виртуальных машин с помощью интерфейса командной строки или портала Azure. Если вы не готовы к миграции и хотите вернуть предыдущее состояние, используйте следующую команду.

```azurecli
azure network vnet abort-migration <virtualNetworkName>
```

Если подготовленная конфигурация вас устраивает, можете продолжать процесс. Выполните следующую команду, чтобы зафиксировать ресурсы.

```azurecli
azure network vnet commit-migration <virtualNetworkName>
```

## <a name="step-5-migrate-a-storage-account"></a>Шаг 5. Перенос учетной записи хранения
После переноса виртуальных машин рекомендуется перенести учетную запись хранения.

Подготовьте учетную запись хранения к переносу, используя следующую команду:

```azurecli
azure storage account prepare-migration <storageAccountName>
```

Проверьте конфигурацию для подготовленной учетной записи с помощью интерфейса командной строки или портала Azure. Если вы не готовы к миграции и хотите вернуть предыдущее состояние, используйте следующую команду.

```azurecli
azure storage account abort-migration <storageAccountName>
```

Если подготовленная конфигурация вас устраивает, можете продолжать процесс. Выполните следующую команду, чтобы зафиксировать ресурсы.

```azurecli
azure storage account commit-migration <storageAccountName>
```

## <a name="next-steps"></a>Дальнейшие действия

* [Обзор поддерживаемого платформой переноса ресурсов IaaS из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-overview.md)
* [Техническое руководство по поддерживаемому платформой переносу из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-deep-dive.md)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md) (Планирование переноса ресурсов IaaS из классической модели в модель Azure Resource Manager)
* [Перенос ресурсов IaaS из классической модели в модель Azure Resource Manager с помощью Azure PowerShell](migration-classic-resource-manager-ps.md)
* [Community tools to migrate IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-community-tools.md) (Инструменты сообщества для перемещения ресурсов IaaS из классической модели в модель Azure Resource Manager)
* [Распространенные ошибки миграции](migration-classic-resource-manager-errors.md)
* [Часто задаваемые вопросы о миграции из классической модели в модель Azure Resource Manager](migration-classic-resource-manager-faq.md)
