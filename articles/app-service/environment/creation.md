---
title: Создание Среда службы приложений
description: Узнайте, как создать Среда службы приложений.
author: ccompy
ms.assetid: 7690d846-8da3-4692-8647-0bf5adfd862a
ms.topic: article
ms.date: 11/16/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 021c85037ea105ff7d8ff642e3a9f28ed3ebe991
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100594104"
---
# <a name="create-an-app-service-environment"></a>Создание Среда службы приложений

> [!NOTE]
> Эта статья относится к Среда службы приложений v3 (Предварительная версия)
> 

[Среда службы приложений (ASE)][Intro] — это одно развертывание клиента службы приложений, которое внедряется в виртуальную сеть Azure (VNet).  ASEv3 поддерживает предоставление доступа к приложениям только по частному адресу в виртуальной сети. При создании ASEv3 на этапе предварительной версии эти ресурсы добавляются в подписку.

- Среда службы приложений
- Частная конечная точка

Для развертывания ASE потребуется использовать две подсети.  В одной подсети будет храниться частная конечная точка.  Эту подсеть можно использовать для других целей, например виртуальных машин.  Другая подсеть используется для исходящих вызовов, сделанных из ASE.  Эту подсеть нельзя использовать ни для чего еще, ни для ASE. 

## <a name="before-you-create-your-ase"></a>Перед созданием среды службы приложений

После создания ASE изменить:

- Расположение
- Подписка
- Группа ресурсов
- Используемая виртуальная сеть Azure
- Используемые подсети
- Размер подсети
- Имя ASE

Исходящая подсеть должна быть достаточно большой, чтобы вместить максимальный размер, который будет масштабировать ASE. Выберите достаточно большую подсеть для поддержки максимальных потребностей в масштабе, так как ее нельзя изменить после создания. Рекомендуемый размер — a/24 с 256 адресами.

После создания ASE можно добавить в него приложения. Если в ASEv3 нет планов службы приложений, плата взимается так, как если бы в ASE существовал один экземпляр плана службы приложений I1v2.  

ASEv3 предлагается только в области выбора. В качестве предварительной версии будут добавлены дополнительные регионы, а затем — общедоступные. 

## <a name="creating-an-ase-in-the-portal"></a>Создание ASE на портале

1. Чтобы создать ASEv3, выполните поиск **Среда службы приложений (Предварительная версия)** в Marketplace.  
2. Основные сведения. Выберите подписку, выберите или создайте группу ресурсов и введите имя ASE.  Имя ASE также будет использоваться для суффикса домена ASE.  Если имя ASE — *contoso* , суффикс домена будет *contoso.appserviceenvironment.NET*.  Это имя будет автоматически задано в частной зоне Azure DNS, используемой виртуальной сетью, в которой развернута ASE. 

    ![Вкладка "создание основ" Среда службы приложений](./media/creation/creation-basics.png)

3. Размещение: выберите параметры ОС, развертывание группы узлов. Параметр ОС указывает операционную систему, которая изначально используется для приложений в этой ASE. После создания ASE можно добавить приложения другой операционной системы. Развертывание группы узлов используется для выбора выделенного оборудования. С помощью ASEv3 можно выбрать параметр включено, а затем наземный на выделенном оборудовании. Вы платите за весь выделенный узел с помощью создания ASE, а затем уменьшаете цену на экземпляры плана службы приложений. 

    ![Среда службы приложений размещения элементов](./media/creation/creation-hosting.png)

4. Сеть: выберите или создайте виртуальную сеть, выберите или создайте входящую подсеть, выберите или создайте исходящую подсеть. Любая подсеть, используемая для исходящего трафика, должна быть пустой и делегирована в Microsoft. Web/hostingEnvironments. Если вы создаете подсеть на портале, подсеть будет автоматически делегирована.

    ![Среда службы приложений выбора сети](./media/creation/creation-networking.png)

5. Проверка и создание: Проверьте правильность конфигурации и нажмите кнопку Создать. Создание ASE займет около часа. 

    ![Среда службы приложений просмотра и создания](./media/creation/creation-review.png)

После завершения создания ASE можно выбрать его в качестве расположения при создании приложений. Дополнительные сведения о создании приложений в новом ASE см. в статье [использование среда службы приложений][UsingASE]

## <a name="os-preference"></a>Настройка ОС
В ASE вы можете использовать приложения Windows, приложения Linux или и то, и другое. В ASEv2 начальная настройка, выбранная во время создания, добавляет инфраструктуру высокой доступности для этой ОС во время создания ASE. Чтобы добавить приложения другой ОС, просто сделайте приложения обычными и выберите нужную ОС. В ASEv3 это не повлияет на поведение серверной части аппреЦиативели.  

## <a name="dedicated-hosts"></a>Выделенные узлы
Как правило, ASE разворачивается на виртуальных машинах, подготовленных в гипервизоре с несколькими клиентами. Если необходимо выполнить развертывание на выделенных системах, включая оборудование, можно подготавливать ASE на выделенных узлах. В первоначальной предварительной версии ASEv3 выделенные узлы поступают в пару. Каждый выделенный узел находится в отдельной зоне доступности, если это разрешено регионом. Для выделенных развертываний ASE на основе узлов цена отличается от обычной. За выделенный узел взимается плата, а затем еще одна плата за каждый экземпляр плана службы приложений.  

<!--Links-->
[Intro]: ./overview.md
[MakeASE]: ./creation.md
[ASENetwork]: ./networking.md
[UsingASE]: ./using.md
[UDRs]: ../../virtual-network/virtual-networks-udr-overview.md
[NSGs]: ../../virtual-network/network-security-groups-overview.md
[Pricing]: https://azure.microsoft.com/pricing/details/app-service/
[ARMOverview]: ../../azure-resource-manager/management/overview.md
[ConfigureSSL]: ../configure-ssl-certificate.md
[Kudu]: https://azure.microsoft.com/resources/videos/super-secret-kudu-debug-console-for-azure-web-sites/
[AppDeploy]: ../deploy-local-git.md
[ASEWAF]: app-service-app-service-environment-web-application-firewall.md
[AppGW]: ../../web-application-firewall/ag/ag-overview.md
[logalerts]: ../../azure-monitor/alerts/alerts-log.md
