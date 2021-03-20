---
title: Использование выделенного пула для выполнения задач задачи
description: Настройте выделенный пул вычислений (пул агентов) в реестре для выполнения задачи реестра контейнеров Azure.
ms.topic: article
ms.date: 10/12/2020
ms.custom: references_regions, devx-track-azurecli
ms.openlocfilehash: eeb9a71854f52da5c1a9f4befae93c377ad67b05
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98920313"
---
# <a name="run-an-acr-task-on-a-dedicated-agent-pool"></a>Выполнение задачи записи контроля доступа в выделенном пуле агентов

Настройте пул виртуальных машин под управлением Azure (*Пул агентов*), чтобы включить выполнение [задач реестра контейнеров Azure][acr-tasks] в выделенной среде вычислений. После настройки одного или нескольких пулов в реестре можно выбрать пул для запуска задачи вместо среды вычислений по умолчанию службы.

Пул агентов предоставляет следующие сведения.

- **Поддержка виртуальных сетей** . назначьте пул агентов виртуальной сети Azure, предоставив доступ к ресурсам в виртуальной сети, таким как реестр контейнеров, хранилище ключей или хранилище.
- **Масштабировать по мере необходимости** — увеличьте число экземпляров в пуле агентов для ресурсоемких задач или измените масштаб до нуля. Выставление счетов основано на распределении пула. Дополнительные сведения см. в разделе [цены](https://azure.microsoft.com/pricing/details/container-registry/).
- **Гибкие возможности** — Выбирайте различные [уровни пула](#pool-tiers) и параметры масштабирования в соответствии с потребностями рабочей нагрузки.
- **Управление Azure** . пулы задач исправлены и обслуживаются в Azure, что обеспечивает зарезервированное выделение без необходимости поддерживать отдельные виртуальные машины.

Эта возможность доступна на уровне **Премиум** службы реестра контейнеров. Сведения об уровнях и ограничениях служб реестра см. в статье [номера SKU реестра контейнеров Azure][acr-tiers].

> [!IMPORTANT]
> Сейчас эта функция доступна в предварительной версии, и применяются [некоторые ограничения](#preview-limitations). Предварительные версии предоставляются при условии, что вы принимаете [дополнительные условия использования][terms-of-use]. Некоторые аспекты этой функции могут быть изменены до выхода общедоступной версии.
>

## <a name="preview-limitations"></a>Ограничения предварительной версии

- В настоящее время Пулы агентов задач поддерживают узлы Linux. Узлы Windows в настоящее время не поддерживаются.
- Пулы агентов задач доступны в предварительной версии в следующих регионах: Западная часть США 2, Юго-Центральный регион США, восток США 2, восток США, Центральная часть США, государственных организаций США, государственных организаций США Техас и государственных организаций США Виргиния.
- Для каждого реестра по умолчанию квота общей виртуальных ЦП (Core) составляет 16 для всех пулов стандартных агентов, а для изолированных пулов агентов — 0. Откройте [запрос в службу поддержки][open-support-ticket] для дополнительного выделения.
- Сейчас невозможно отменить выполнение задачи в пуле агентов.

## <a name="prerequisites"></a>Предварительные условия

* Чтобы использовать Azure CLI действия, описанные в этой статье, требуется Azure CLI версии 2.3.1 или более поздней. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][azure-cli]. Кроме того, можно воспользоваться запуском в [Azure Cloud Shell](../cloud-shell/quickstart.md).
* Если у вас еще нет реестра контейнеров, [создайте его][create-reg-cli] (требуемый уровень Premium) в области просмотра.

## <a name="pool-tiers"></a>Уровни пула

Уровни пула агентов предоставляют следующие ресурсы для каждого экземпляра в пуле.

|Уровень    | Type  |  ЦП  |Память (ГБ)  |
|---------|---------|---------|---------|
|S1     |  standard    | 2       |    3     |
|S2     |  standard    | 4       |    8     |
|S3     |  standard    | 8       |   16     |
|I6     |  изолирован    | 64     |   216     |


## <a name="create-and-manage-a-task-agent-pool"></a>Создание пула агентов задач и управление им

### <a name="set-default-registry-optional"></a>Задать реестр по умолчанию (необязательно)

Чтобы упростить Azure CLI следующие команды, задайте реестр по умолчанию, выполнив команду [AZ configure][az-configure] :

```azurecli
az configure --defaults acr=<registryName>
```

В следующих примерах предполагается, что вы установили реестр по умолчанию. В противном случае передайте `--registry <registryName>` параметр в каждой `az acr` команде.

### <a name="create-agent-pool"></a>Создание пула агентов

Создайте пул агентов с помощью команды [AZ запись контроля доступа ажентпул Create][az-acr-agentpool-create] . В следующем примере создается пул уровня S2 (4 ЦП или экземпляр). По умолчанию пул содержит 1 экземпляр.

```azurecli
az acr agentpool create \
    --name myagentpool \
    --tier S2
```

> [!NOTE]
> Создание пула агентов и других операций управления пулом занимает несколько минут.

### <a name="scale-pool"></a>Пул масштабирования

Масштабирование размера пула с помощью команды [AZ контроля доступа ажентпул Update][az-acr-agentpool-update] . В следующем примере пул масштабируется до двух экземпляров. Можно масштабировать до 0 экземпляров.

```azurecli
az acr agentpool update \
    --name myagentpool \
    --count 2
```

## <a name="create-pool-in-a-virtual-network"></a>Создание пула в виртуальной сети

### <a name="add-firewall-rules"></a>Добавление правил брандмауэра

Для пулов агентов задач требуется доступ к следующим службам Azure. Следующие правила брандмауэра должны быть добавлены в существующие группы безопасности сети или определяемые пользователем маршруты.

| Направление | Протокол | Источник         | Исходный порт | Назначение          | Порт приемника | Используется    |
|-----------|----------|----------------|-------------|----------------------|-----------|---------|
| Исходящие  | TCP      | Виртуальная сеть | Любой         | AzureKeyVault        | 443       | По умолчанию |
| Исходящие  | TCP      | Виртуальная сеть | Любой         | Служба хранилища              | 443       | По умолчанию |
| Исходящие  | TCP      | Виртуальная сеть | Любой         | концентратор событий.             | 443       | По умолчанию |
| Исходящие  | TCP      | Виртуальная сеть | Любой         | AzureActiveDirectory | 443       | По умолчанию |
| Исходящие  | TCP      | Виртуальная сеть | Любой         | AzureMonitor         | 443       | По умолчанию |

> [!NOTE]
> Если задачам требуются дополнительные ресурсы из общедоступного Интернета, добавьте соответствующие правила. Например, дополнительные правила необходимы для запуска задачи сборки DOCKER, которая извлекает базовые образы из центра DOCKER, или восстанавливает пакет NuGet.

### <a name="create-pool-in-vnet"></a>Создание пула в виртуальной сети

В следующем примере создается пул агентов в подсети *mysubnet* сетевого *myvnet*:

```azurecli
# Get the subnet ID
subnetId=$(az network vnet subnet show \
        --resource-grop myresourcegroup \
        --vnet-name myvnet \
        --name mysubnetname \
        --query id --output tsv)

az acr agentpool create \
    --name myagentpool \
    --tier S2 \
    --subnet-id $subnetId
```

## <a name="run-task-on-agent-pool"></a>Выполнение задачи в пуле агентов

В следующих примерах показано, как указать Пул агентов при постановке задачи в очередь.

> [!NOTE]
> Чтобы использовать пул агентов в задаче контроля учетных записей, убедитесь, что в пуле содержится по крайней мере 1 экземпляр.
>

### <a name="quick-task"></a>Быстрая задача

Постановка в очередь быстрой задачи в пуле агентов с помощью команды [AZ запись контроля][az-acr-build] доступа и передачи `--agent-pool` параметра:

```azurecli
az acr build \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --file Dockerfile \
    https://github.com/Azure-Samples/acr-build-helloworld-node.git#main
```

### <a name="automatically-triggered-task"></a>Автоматически активируемая задача

Например, создайте запланированную задачу в пуле агентов с помощью команды [AZ запись контроля доступа Create][az-acr-task-create], передав `--agent-pool` параметр.

```azurecli
az acr task create \
    --name mytask \
    --agent-pool myagentpool \
    --image myimage:mytag \
    --schedule "0 21 * * *" \
    --file Dockerfile \
    --context https://github.com/Azure-Samples/acr-build-helloworld-node.git#main \
    --commit-trigger-enabled false
```

Чтобы проверить настройку задачи, выполните команду [AZ запись контроля][az-acr-task-run]доступа:

```azurecli
az acr task run \
    --name mytask
```

### <a name="query-pool-status"></a>Состояние пула запросов

Чтобы узнать число запусков, запланированных в пуле агентов, выполните команду [AZ запись контроля доступа ажентпул][az-acr-agentpool-show].

```azurecli
az acr agentpool show \
    --name myagentpool \
    --queue-count
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные примеры сборок и обслуживания образов контейнеров в облаке см. в статье Руководство по [задачам записи контроля](container-registry-tutorial-quick-task.md)доступа.



[acr-tasks]:           container-registry-tasks-overview.md
[acr-tiers]:           container-registry-skus.md
[azure-cli]:           /cli/azure/install-azure-cli
[open-support-ticket]: https://aka.ms/acr/support/create-ticket
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[az-configure]: /cli/azure#az-configure
[az-acr-agentpool-create]: /cli/azure/acr/agentpool#az-acr-agentpool-create
[az-acr-agentpool-update]: /cli/azure/acr/agentpool#az-acr-agentpool-update
[az-acr-agentpool-show]: /cli/azure/acr/agentpool#az-acr-agentpool-show
[az-acr-build]: /cli/azure/acr#az-acr-build
[az-acr-task-create]: /cli/azure/acr/task#az-acr-task-create
[az-acr-task-run]: /cli/azure/acr/task#az-acr-task-run
[create-reg-cli]: container-registry-get-started-azure-cli.md
