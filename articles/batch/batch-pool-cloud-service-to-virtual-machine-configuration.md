---
title: Перенос конфигурации пула пакетной службы из облачных служб на виртуальные машины
description: Узнайте, как обновить конфигурацию пула до последней и рекомендуемой конфигурации.
ms.topic: how-to
ms.date: 03/11/2021
ms.openlocfilehash: a176c4df1737a340a546b4ab7926447cd821350d
ms.sourcegitcommit: 5f32f03eeb892bf0d023b23bd709e642d1812696
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/12/2021
ms.locfileid: "103200561"
---
# <a name="migrate-batch-pool-configuration-from-cloud-services-to-virtual-machine"></a>Перенос конфигурации пула пакетной службы из облачных служб в виртуальную машину

В настоящее время пулы пакетной службы можно создавать с помощью [virtualMachineConfiguration](/rest/api/batchservice/pool/add#virtualmachineconfiguration) или [cloudServiceConfiguration](/rest/api/batchservice/pool/add#cloudserviceconfiguration). Рекомендуется использовать только конфигурацию виртуальной машины, так как эта конфигурация поддерживает все возможности пакета.

Пулы конфигураций облачных служб не поддерживают некоторые из текущих функций пакетной службы и не поддерживают новые функции. Вы не сможете создавать новые пулы "CloudServiceConfiguration" или добавлять новые узлы в существующие пулы [после 29 февраля 2024 г](https://azure.microsoft.com/updates/azure-batch-cloudserviceconfiguration-pools-will-be-retired-on-29-february-2024/).

Если в ваших решениях пакетной службы в настоящее время используются пулы "cloudServiceConfiguration", рекомендуется как можно скорее изменить на "virtualMachineConfiguration". Это позволит вам воспользоваться всеми возможностями пакетной службы, такими как расширенный [Выбор серии виртуальных машин](batch-pool-vm-sizes.md), виртуальных машин Linux, [контейнеров](batch-docker-container-workloads.md), [Azure Resource Manager виртуальных сетей](batch-virtual-network.md)и [шифрования дисков узла](disk-encryption.md).

## <a name="create-a-pool-using-virtual-machine-configuration"></a>Создание пула с помощью конфигурации виртуальной машины

Невозможно переключить существующий активный пул, который использует "cloudServiceConfiguration", для использования "virtualMachineConfiguration". Вместо этого необходимо создать новые пулы. После создания пулов "virtualMachineConfiguration" и репликации всех заданий и задач можно удалить старый "Клаудсервицеконфигуратион'пулс, который больше не используется.

Все API пакетной службы, программы командной строки, портал Azure и пользовательский интерфейс Batch Explorer позволяют создавать пулы с помощью "virtualMachineConfiguration".

Пошаговое руководство по созданию пулов, использующих "virtualMachineConfiguration", см. в руководстве по [.NET](tutorial-parallel-dotnet.md) или в [учебнике по Python](tutorial-parallel-python.md).

## <a name="pool-configuration-differences"></a>Различия в конфигурации пула

Ниже приведены некоторые основные различия между двумя конфигурациями.

- узлы пула "cloudServiceConfiguration" используют только ОС Windows. пулы "virtualMachineConfiguration" могут использовать ОС Linux или Windows.
- По сравнению с пулами "cloudServiceConfiguration", пулы "virtualMachineConfiguration" имеют более широкий набор возможностей, таких как поддержка контейнеров, диски данных и шифрование дисков.
- Время запуска и удаления пулов и узлов может немного отличаться между пулами "cloudServiceConfiguration" и пулами "virtualMachineConfiguration".
- узлы пула "virtualMachineConfiguration" используют управляемые диски ОС. [Тип управляемого диска](../virtual-machines/disks-types.md) , используемый для каждого узла, зависит от размера виртуальной машины, выбранной для пула. Если для пула указан размер виртуальной машины, например "Standard_D2s_v3", то используется SSD уровня "Премиум". Если указан размер виртуальной машины "не-s", например "Standard_D2_v3", используется стандартный жесткий диск.

   > [!IMPORTANT]
   > Как и в случае с виртуальными машинами и масштабируемыми наборами виртуальных машин, управляемый диск операционной системы, используемый для каждого узла, требует затрат, что является дополнительной стоимостью виртуальных машин. Для узлов "cloudServiceConfiguration" нет затрат на диск ОС, так как диск ОС создается на узлах локального SSD.

## <a name="azure-data-factory-custom-activity-pools"></a>Пулы настраиваемых действий фабрики данных Azure

Пулы пакетной службы Azure можно использовать для запуска настраиваемых действий фабрики данных. Все пулы "cloudServiceConfiguration", используемые для выполнения настраиваемых действий, необходимо удалить, а также создать новые пулы "virtualMachineConfiguration".

При создании новых пулов для запуска настраиваемых действий фабрики данных следуйте приведенным ниже рекомендациям.

- Приостановите все конвейеры перед созданием новых пулов и удалением старых, чтобы гарантировать, что выполнение не будет прервано.
- Для предотвращения изменений конфигурации связанной службы можно использовать один и тот же идентификатор пула.
- Возобновить конвейеры при создании новых пулов.

Дополнительные сведения об использовании пакетной службы Azure для выполнения настраиваемых действий фабрики данных см. в статье [связанная Пакетная служба Azure](../data-factory/compute-linked-services.md#azure-batch-linked-service) и [Настраиваемые действия в конвейере фабрики данных](../data-factory/transform-data-using-dotnet-custom-activity.md) .

## <a name="next-steps"></a>Дальнейшие действия

- Дополнительные сведения о [конфигурациях пулов](nodes-and-pools.md#configurations).
- Дополнительные сведения о рекомендациях по работе с [пулами](best-practices.md#pools).
- Дополнительные сведения о [добавлении пула](/rest/api/batchservice/pool/add) и [virtualMachineConfiguration](/rest/api/batchservice/pool/add#virtualmachineconfiguration)см. в справочнике по REST API.