---
title: Настройка группы доступности Always On с помощью решения DH2i DxEnterprise, работающего на Виртуальных машинах Azure под управлением Linux
description: Использование DH2i DxEnterprise в качестве диспетчера кластеров для обеспечения высокого уровня доступности, используя группу доступности на Виртуальных машинах SQL Server на Linux
ms.date: 03/04/2021
ms.service: virtual-machines-sql
ms.topic: tutorial
author: amvin87
ms.author: amitkh
ms.reviewer: vanto
ms.openlocfilehash: 56002aaa977b94b0fabee4f17343f483706eb77d
ms.sourcegitcommit: 56b0c7923d67f96da21653b4bb37d943c36a81d6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/06/2021
ms.locfileid: "106449432"
---
# <a name="tutorial---setup-a-three-node-always-on-availability-group-with-dh2i-dxenterprise-running-on-linux-based-azure-virtual-machines"></a>Руководство. Настройка группы доступности Always On с тремя узлами с помощью решения DH2i DxEnterprise, работающего на Виртуальных машинах Azure под управлением Linux

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этом руководстве содержатся сведения о настройке группы доступности Always On SQL Server с помощью решения DH2i DxEnterprise, запущенного на Виртуальных машинах Azure под управлением Linux. 

Дополнительные сведения о DxEnterprise см. [здесь](https://dh2i.com/dxenterprise-availability-groups/).

> [!NOTE]
> Корпорация Майкрософт поддерживает перемещение данных, группы доступности и компоненты SQL Server. Обратитесь к DH2i, чтобы получить поддержку касательно документации о кластере DH2i DxEnterprise и способов управления кластерами и кворумом.
 

Данное руководство состоит из следующих шагов.

> [!div class="checklist"]
> * Установка SQL Server на всех виртуальных машинах Azure, которые будут входить в группу доступности.
> * Установка DxEnterprise на всех виртуальных машинах и настройка кластера DxEnterprise.
> * Создание виртуальных узлов для обеспечения поддержки отработки отказа и высокой доступности и последующее добавление группы доступности и базы данных в эту группу доступности.
> * Создание внутреннего Azure Load Balancer для прослушивателя группы доступности (необязательно).
> * Выполнение ручного или автоматического перехода на другой ресурс.

Проходя это руководство, мы настроим кластер DxEnterprise с помощью [пользовательского интерфейса клиента DxAdmin](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-dxadmin-client-ui-quick-start-guide/). При необходимости кластер также можно настроить с помощью интерфейса командной строки [DxCLI](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-software-dxcli-guide/). В этом примере мы использовали четыре виртуальные машины. Три из них работают под управлением Ubuntu 18.04 и являются частью кластера з тремя узлами. Четвертая виртуальная машина работает под управлением Windows 10 и использует для настройки кластера и управления им средство DxAdmin.

## <a name="prerequisites"></a>Предварительные условия

- Создайте четыре виртуальные машины в Azure. Следуйте указаниям, приведенным в статье [Краткое руководство. Создание виртуальной машины Linux на портале Azure](../../../virtual-machines/linux/quick-create-portal.md), чтобы создать виртуальные машины на базе Linux. Аналогичным образом, чтобы создать виртуальную машину на базе Windows, следуйте указаниям, приведенным в статье [Краткое руководство. Создание виртуальной машины Windows на портале Azure](../../../virtual-machines/windows/quick-create-portal.md).
- Установите .NET 3.1 на всех виртуальных машинах под управлением Linux, которые будут входить в кластер. Следуйте приведенным [здесь](/dotnet/core/install/linux) инструкциям в зависимости от выбранной операционной системы Linux.
- Потребуется действующая лицензия DxEnterprise с включенными функциями управления группами доступности. Дополнительные сведения о том, как получить бесплатную пробную версию DxEnterprise, см. [здесь](https://dh2i.com/trial/).

## <a name="install-sql-server-on-all-the-azure-vms-that-will-be-part-of-the-availability-group"></a>Установка SQL Server на всех виртуальных машинах Azure, которые будут входить в группу доступности

Проходя это руководство, мы настроим кластер с тремя узлами под управлением Linux, на котором будет выполняться группа доступности. Следуйте документации по [установке SQL Server на Linux](/sql/linux/sql-server-linux-overview#install) в зависимости от выбранной вами платформы Linux. Кроме того, для работы с этим руководством рекомендуется установить [средства SQL Server](/sql/linux/sql-server-linux-setup-tools).
 
> [!NOTE]
> Убедитесь, что выбранная вами ОС Linux является общим дистрибутивом, поддерживаемым как [DH2i DxEnterprise (см. раздел "Минимальные требования к системе")](https://dh2i.com/wp-content/uploads/DxEnterprise-v20-Admin-Guide.pdf), так и [Microsoft SQL Server](/sql/linux/sql-server-linux-release-notes-2019#supported-platforms).
>
> В этом примере используется Ubuntu 18.04, то есть дистрибутив, поддерживаемый как DH2i DxEnterprise, так и Microsoft SQL Server.

В рамках этого руководства мы не будем устанавливать SQL Server на виртуальную машину Windows, поскольку этот узел не будет частью кластера и используется только для управления кластером с помощью DxAdmin.

После выполнения этого действия на всех трех виртуальных машинах под управлением Linux, которые будут использоваться в группе доступности, должны быть установлены SQL Server и [средства SQL Server](/sql/linux/sql-server-linux-setup-tools) (необязательно).
 
## <a name="install-dxenterprise-on-all-the-vms-and-configure-the-cluster"></a>Установка DxEnterprise на всех виртуальных машинах и настройка кластера

На этом шаге мы установим DH2i DxEnterprise для Linux на трех виртуальных машинах Linux. В следующей таблице описывается роль каждого сервера в кластере.

| Число виртуальных машин | Роль DH2i DxEnterprise | Роль реплики группы доступности Microsoft SQL Server |
|--|--|--|
| 1 | Узел кластера — под управлением Linux | Основной |
| 1 | Узел кластера — под управлением Linux | Вторичная — синхронная фиксация |
| 1 | Узел кластера — под управлением Linux | Вторичная — синхронная фиксация |
| 1 | Клиент DxAdmin | Н/Д |


Чтобы установить DxEnterprise на трех узлах под управлением Linux, следуйте документации по DH2i DxEnterprise для выбранной вами операционной системы Linux. Установите DxEnterprise с помощью любого из перечисленных ниже методов.

- **Ubuntu**
    - [Краткое руководство по началу работы с установкой репозитория](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-ubuntu-installation-quick-start-guide/)
    - [Краткое руководство по началу работы с расширением](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-azure-extension-quick-start-guide/)
    - [Краткое руководство по началу работы с образами из Marketplace](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-azure-marketplace-image-for-linux-quick-start-guide/)
- **RHEL**
    - [Краткое руководство по началу работы с установкой репозитория](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-rhel-centos-installation-quick-start-guide/)
    - [Краткое руководство по началу работы с расширением](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-azure-extension-quick-start-guide/)
    - [Краткое руководство по началу работы с образами из Marketplace](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-azure-marketplace-image-for-linux-quick-start-guide/)

Чтобы установить только клиентское средство DxAdmin на виртуальной машине Windows, следуйте указаниям статьи [Краткое руководство по началу работы с пользовательским интерфейсом DxAdmin](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-dxadmin-client-ui-quick-start-guide/).

После выполнения этого действия у вас должны быть кластер DxEnterprise на виртуальных машинах Linux и клиент DxAdmin на клиентском компьютере Windows. 

> [!NOTE]
> Вы также можете создать кластер с тремя узлами, добавив один из узлов в *режиме только для конфигурации*, как это описано [здесь](/sql/database-engine/availability-groups/windows/availability-modes-always-on-availability-groups#SupportedAvModes), чтобы включить автоматический переход на другой ресурс. 

## <a name="create-the-virtual-hosts-to-provide-failover-support-and-high-availability"></a>Создание виртуальных узлов для обеспечения поддержки отработки отказа и высокой доступности

На этом шаге мы собираемся создать виртуальный узел, группу доступности, а затем добавить базу данных, используя для этого пользовательский интерфейс DxAdmin.   

> [!NOTE]
> На этом этапе понадобится перезапустить экземпляры SQL Server, чтобы включить группы доступности Always On. 

Подключитесь к клиентскому компьютеру Windows, на котором запущено средство DxAdmin, чтобы подключиться к кластеру, созданному ранее. Выполните действия, описанные в статье об использовании [групп доступности MSSQL с DxAdmin](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-mssql-availability-groups-with-dxadmin-quick-start-guide/), чтобы включить группы доступности Always On и создать виртуальный узел и группу доступности. 

> [!TIP]
> Прежде чем добавлять базы данных, убедитесь, что база данных создана и архивируется на основном экземпляре SQL Server.  

## <a name="create-the-internal-azure-load-balancer-for-listener-optional"></a>Создание внутреннего Azure Load Balancer для прослушивателя (необязательно)

На этом необязательном шаге можно создать и настроить Azure Load Balancer, содержащий IP-адреса для прослушивателей группы доступности. Дополнительные сведения об Azure Load Balancer см. в [этой статье](../../../load-balancer/load-balancer-overview.md). Чтобы настроить Azure Load Balancer и прослушиватель группы доступности с помощью DxAdmin, следуйте указаниям статьи [Краткое руководство по началу работы с Azure Load Balancer](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-0-software-azure-load-balancer-quick-start-guide/).

После выполнения этого шага у вас должен быть прослушиватель группы доступности, сопоставленный с внутренним Azure Load Balancer.

## <a name="test-manual-or-automatic-failover"></a>Тестирование ручного или автоматического перехода на другой ресурс

Для тестирования автоматического перехода на другой ресурс можно перейти к первичной реплике и остановить ее (выключить виртуальную машину на портале Azure). Это приведет к репликации из-за внезапной недоступности основного узла. Ожидаемое поведение:
- Диспетчер кластеров повышает уровень одной из вторичных реплик в группе доступности до первичной.
- Нерабочая первичная реплика автоматически присоединяет кластер после резервного копирования. Диспетчер кластеров повышает ее уровень до вторичной реплики.

 
Можно также выполнить переход на другой ресурс вручную, следуя приведенным ниже инструкциям.

1. Подключитесь к кластеру через DxAdmin   
1. Разверните виртуальный узел для группы доступности.
1. Щелкните правой кнопкой мыши целевой узел / вторичную реплику и выберите **Start Hosting on Member** (Начать размещение на элементе), чтобы инициировать отработку отказа. 

Сведения о дополнительных операциях в DxEnterprise см. в [руководстве администратора по DxEnterprise](https://dh2i.com/wp-content/uploads/DxEnterprise-v20-Admin-Guide.pdf), а также в [руководстве по DxEnterprise DxCLI](https://dh2i.com/docs/20-0/dxenterprise/dh2i-dxenterprise-20-software-dxcli-guide/).

## <a name="next-steps"></a>Следующие шаги

- Узнайте больше о [группах доступности на Linux](/sql/linux/sql-server-linux-availability-group-overview).
- [Краткое руководство. Создание виртуальной машины Linux на портале Azure](../../../virtual-machines/linux/quick-create-portal.md)
- [Краткое руководство. Создание виртуальной машины Windows на портале Azure](../../../virtual-machines/windows/quick-create-portal.md)
- [Поддерживаемые платформы для SQL Server 2019 на Linux](/sql/linux/sql-server-linux-release-notes-2019#supported-platforms)