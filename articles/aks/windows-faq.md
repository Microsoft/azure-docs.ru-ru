---
title: 'Пулы узлов Windows Server: вопросы и ответы'
titleSuffix: Azure Kubernetes Service
description: Ознакомьтесь с часто задаваемыми вопросами при запуске пулов узлов Windows Server и рабочих нагрузок приложений в службе Azure Kubernetes (AKS).
services: container-service
ms.topic: article
ms.date: 10/12/2020
ms.openlocfilehash: cc5a5ec2bbfb64a1e787277bf67579bad0543cd6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101739582"
---
# <a name="frequently-asked-questions-for-windows-server-node-pools-in-aks"></a>Часто задаваемые вопросы о пулах узлов Windows Server в AKS

В службе Azure Kubernetes Service (AKS) можно создать пул узлов, который будет работать под управлением Windows Server в качестве гостевой ОС на узлах. Эти узлы могут запускать собственные приложения контейнеров Windows, например созданные на платформа .NET Framework. Существуют различия в том, как операционная система Linux и ОС Windows предоставляет поддержку контейнеров. Некоторые распространенные компоненты Linux Kubernetes и Pod в настоящее время недоступны для пулов узлов Windows.

В этой статье описаны некоторые часто задаваемые вопросы и основные понятия ОС для узлов Windows Server в AKS.

## <a name="which-windows-operating-systems-are-supported"></a>Какие операционные системы Windows поддерживаются?

AKS использует Windows Server 2019 в качестве версии ОС узла и поддерживает только изоляцию процессов. Образы контейнеров, созданные с использованием других версий Windows Server, не поддерживаются. Дополнительные сведения см. в разделе [Совместимость версий контейнера Windows][windows-container-compat].

## <a name="is-kubernetes-different-on-windows-and-linux"></a>Отличается ли Kubernetes в Windows и Linux?

Поддержка пула узлов Window Server включает некоторые ограничения, которые входят в состав вышестоящего сервера Windows Server в Kubernetes проекте. Эти ограничения не относятся к AKS. Дополнительные сведения об этой вышестоящей поддержке Windows Server в Kubernetes см. в разделе [Поддерживаемые функции и ограничения][upstream-limitations] статьи [Введение в поддержку Windows в документе Kubernetes][intro-windows] в проекте Kubernetes.

Kubernetes является историческим, ориентированным на Linux. Многие примеры, используемые в вышестоящем веб-сайте [Kubernetes.IO][kubernetes] , предназначены для использования на узлах Linux. При создании развертываний, использующих контейнеры Windows Server, применяются следующие рекомендации на уровне операционной системы.

- **Identity** — Linux определяет пользователя с помощью целочисленного идентификатора пользователя (UID). Пользователь также имеет буквенно-цифровое имя пользователя для входа в систему, которое Linux преобразуется в UID пользователя. Аналогичным образом Linux определяет группу пользователей по идентификатору целочисленной группы (GID) и преобразует имя группы в соответствующий ему GID.
    - Windows Server использует более крупный двоичный идентификатор безопасности (SID), который хранится в базе данных диспетчера доступа безопасности Windows (SAM). Эта база данных не является общей для узла и контейнеров, а также между контейнерами.
- **Разрешения для файлов** — Windows Server использует список управления доступом на основе идентификаторов безопасности, а не битовую маску разрешений и UID + GID.
- **Пути к файлам** — соглашение в Windows Server заключается в использовании \ вместо/.
    - В спецификациях Pod, в которых подключены тома, правильно укажите путь для контейнеров Windows Server. Например, вместо точки подключения */МНТ/Волуме* в контейнере Linux укажите букву диска и расположение, например */к/Волуме* , чтобы подключить как диск *K:* .

## <a name="what-kind-of-disks-are-supported-for-windows"></a>Какие диски поддерживаются для Windows?

Диски Azure и файлы Azure являются поддерживаемыми типами томов. Они доступны как тома NTFS в контейнере Windows Server.

## <a name="can-i-run-windows-only-clusters-in-aks"></a>Можно ли запускать кластеры только для Windows в AKS?

Главные узлы (плоскость управления) в кластере AKS размещаются в AKS службе, поэтому вы не будете предоставлять доступ к операционной системе узлов, на которых размещаются главные компоненты. Все кластеры AKS создаются с использованием пула первого узла по умолчанию, который основан на Linux. Этот пул узлов содержит системные службы, необходимые для работы кластера. Рекомендуется запускать по крайней мере два узла в пуле первого узла, чтобы обеспечить надежность кластера и возможность выполнять кластерные операции. Первый пул узлов на основе Linux нельзя удалить, пока не будет удален сам кластер AKS.

## <a name="how-do-i-patch-my-windows-nodes"></a>Как поставить заплаты для моих узлов Windows?

Чтобы получить последние исправления для узлов Windows, можно либо [Обновить пул узлов][nodepool-upgrade] , либо [обновить образ узла][upgrade-node-image]. На узлах в AKS не включены обновления Windows. AKS выпускает новые образы пула узлов, как только будут доступны исправления, и ответственность за обновление пулов узлов, чтобы они оставались актуальными для исправлений и исправлений. Это также верно для используемой версии Kubernetes. [Заметки о выпуске AKS][aks-release-notes] указывают, когда доступны новые версии. Дополнительные сведения об обновлении всего пула узлов Windows Server см. в разделе [Обновление пула узлов в AKS][nodepool-upgrade]. Если вы заинтересованы только в обновлении образа узла, см. раздел [Обновление образа узла AKS][upgrade-node-image].

> [!NOTE]
> Обновленный образ Windows Server будет использоваться только в том случае, если обновление кластера (обновление плоскости управления) было выполнено до обновления пула узлов.
>

## <a name="what-network-plug-ins-are-supported"></a>Какие сетевые подключаемые модули поддерживаются?

Кластеры AKS с пулами узлов Windows должны использовать модель сети Azure CNI (Advanced). Кубенет (базовая) сеть не поддерживается. Дополнительные сведения о различиях в сетевых моделях см. в разделе [Основные понятия сети для приложений в AKS][azure-network-models]. Для использования модели сети Azure CNI требуется дополнительное планирование и рекомендации по управлению IP-адресами. Дополнительные сведения о планировании и реализации Azure CNI см. в статье [Настройка сети CNI для Azure в AKS][configure-azure-cni].

Узлы Windows в кластерах AKS также имеют [прямой возврат сервера (DSR)][dsr] , включенный по умолчанию при включенном Калико.

## <a name="is-preserving-the-client-source-ip-supported"></a>Поддерживается ли сохранение исходного IP-адреса клиента?

В настоящее время [Сохранение IP-адресов источника клиента][client-source-ip] не поддерживается для узлов Windows.

## <a name="can-i-change-the-max--of-pods-per-node"></a>Можно ли изменить максимальное число модулей Pod на узел?

Да. Сведения о возможных последствиях и доступных параметрах см. в разделе [Максимальное число модулей][maximum-number-of-pods]Pod.

## <a name="why-am-i-seeing-an-error-when-i-try-to-create-a-new-windows-agent-pool"></a>Почему при попытке создать новый пул агента Windows появляется сообщение об ошибке?

Если вы создали кластер до февраля 2020 и не выполнили никаких операций обновления кластера, кластер по-прежнему использует старый образ Windows. Возможно, вы обнаружили ошибку, похожую на:

"Не найден следующий список образов, на которые ссылается шаблон развертывания: Издатель: MicrosoftWindowsServer, предложение: WindowsServer, SKU: 2019-Datacenter-Core-смаллдиск-2004, версия: Последняя. https://docs.microsoft.com/azure/virtual-machines/windows/cli-ps-findimageИнструкции по поиску доступных образов см. в разделе.

Чтобы исправить эту ошибку, сделайте следующее:

1. Обновите [плоскость управления кластером][upgrade-cluster-cp] , чтобы обновить предложение образа и издателя.
1. Создайте новые Пулы агентов Windows.
1. Перемещение модулей Windows Pod из существующих пулов агентов Windows в новые Пулы агентов Windows.
1. Удалите старые Пулы агентов Windows.

## <a name="how-do-i-rotate-the-service-principal-for-my-windows-node-pool"></a>Разделы справки поворачивать субъект-службу для пула узлов Windows?

Пулы узлов Windows не поддерживают смену субъекта-службы. Чтобы обновить субъект-службу, создайте пул узлов Windows и перенесите модули из старого пула в новый. После завершения этого действия удалите пул старых узлов.

Вместо этого используйте управляемые удостоверения, которые, по сути, являются оболочками для субъектов-служб. Дополнительные сведения см. в статье [Использование управляемых удостоверений в службе Kubernetes Azure][managed-identity].

## <a name="how-many-node-pools-can-i-create"></a>Сколько пулов узлов можно создать?

Кластер AKS не может содержать больше 10 пулов узлов. В этих пулах узлов может быть не более 1000 узлов. [Ограничения пула узлов][nodepool-limitations].

## <a name="what-can-i-name-my-windows-node-pools"></a>Как можно присвоить имя пулам узлов Windows?

Длина имени должна быть не более 6 (шести) символов. Это текущее ограничение AKS.

## <a name="are-all-features-supported-with-windows-nodes"></a>Поддерживаются ли все функции узлов Windows?

Кубенет в настоящее время не поддерживается для узлов Windows.

## <a name="can-i-run-ingress-controllers-on-windows-nodes"></a>Можно ли запускать контроллеры входящего трафика на узлах Windows?

Да, входной контроллер, который поддерживает контейнеры Windows Server, может работать на узлах Windows в AKS.

## <a name="can-i-use-azure-dev-spaces-with-windows-nodes"></a>Можно ли использовать Azure Dev Spaces с узлами Windows?

Azure Dev Spaces в настоящее время доступно только для пулов узлов на основе Linux.

## <a name="can-my-windows-server-containers-use-gmsa"></a>Могут ли мои контейнеры Windows Server использовать gMSA?

Поддержка групповых управляемых учетных записей служб (gMSA) в настоящее время недоступна в AKS.

## <a name="can-i-use-azure-monitor-for-containers-with-windows-nodes-and-containers"></a>Можно ли использовать Azure Monitor для контейнеров с узлами и контейнерами Windows?

Да, вы можете, однако, Azure Monitor в общедоступной предварительной версии для сбора журналов (stdout, stderr) и метрик из контейнеров Windows. Вы также можете присоединиться к динамическому потоку журналов stdout из контейнера Windows.

## <a name="are-there-any-limitations-on-the-number-of-services-on-a-cluster-with-windows-nodes"></a>Существуют ли ограничения на количество служб в кластере с узлами Windows?

Кластер с узлами Windows может иметь около 500 служб, прежде чем будет обнаружено нехватка портов.

## <a name="can-i-use-azure-hybrid-benefit-with-windows-nodes"></a>Можно ли использовать Преимущество гибридного использования Azure с узлами Windows?

Да. Преимущество гибридного использования Azure для Windows Server сокращает эксплуатационные расходы, позволяя перенести локальную лицензию Windows Server на AKS узлы Windows.

Преимущество гибридного использования Azure можно использовать в целом кластере AKS или на отдельных узлах. Для отдельных узлов необходимо перейти к [группе ресурсов узла][resource-groups] и применить преимущество гибридного использования Azure к узлам напрямую. Дополнительные сведения о применении Преимущество гибридного использования Azure к отдельным узлам см. в разделе [преимущество гибридного использования Azure для Windows Server][hybrid-vms]. 

Чтобы использовать Преимущество гибридного использования Azure в новом кластере AKS, используйте `--enable-ahub` аргумент.

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --load-balancer-sku Standard \
    --windows-admin-password 'Password1234$' \
    --windows-admin-username azure \
    --network-plugin azure
    --enable-ahub
```

Чтобы использовать Преимущество гибридного использования Azure в существующем кластере AKS, обновите кластер с помощью `--enable-ahub` аргумента.

```azurecli
az aks update \
    --resource-group myResourceGroup
    --name myAKSCluster
    --enable-ahub
```

Чтобы проверить, заданы ли Преимущество гибридного использования Azure в кластере, используйте следующую команду:

```azurecli
az vmss show --name myAKSCluster --resource-group MC_CLUSTERNAME
```

Если в кластере включен Преимущество гибридного использования Azure, выходные данные `az vmss show` будут выглядеть следующим образом:

```console
"platformFaultDomainCount": 1,
  "provisioningState": "Succeeded",
  "proximityPlacementGroup": null,
  "resourceGroup": "MC_CLUSTERNAME"
```

## <a name="can-i-use-the-kubernetes-web-dashboard-with-windows-containers"></a>Можно ли использовать веб-панель мониторинга Kubernetes с контейнерами Windows?

Да, вы можете использовать [веб-панель мониторинга Kubernetes][kubernetes-dashboard] для доступа к информации о контейнерах Windows, но в настоящее время вы не сможете запустить *kubectl Exec* в работающем контейнере Windows непосредственно из веб-панели мониторинга Kubernetes. Дополнительные сведения о подключении к работающему контейнеру Windows см. в [статье подключение с помощью RDP к Azure Kubernetes Service (AKS) узлы кластера Windows Server для обслуживания или устранения неполадок][windows-rdp].

## <a name="what-if-i-need-a-feature-thats-not-supported"></a>Что делать, если мне нужна неподдерживаемая функция?

Мы работаем над тем, чтобы приложить все необходимые компоненты Windows в AKS, но если у вас возникают пробелы, проект [AKS-Engine][aks-engine] с открытым кодом предоставляет простой и полностью настраиваемый способ запуска Kubernetes в Azure, включая поддержку Windows. Обязательно ознакомьтесь с нашим планом функций, поступающих в [AKSную схему][aks-roadmap].

## <a name="next-steps"></a>Дальнейшие действия

Чтобы приступить к работе с контейнерами Windows Server в AKS, [Создайте пул узлов под управлением Windows Server в AKS][windows-node-cli].

<!-- LINKS - external -->
[kubernetes]: https://kubernetes.io
[aks-engine]: https://github.com/azure/aks-engine
[upstream-limitations]: https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations
[intro-windows]: https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/
[aks-roadmap]: https://github.com/Azure/AKS/projects/1
[aks-release-notes]: https://github.com/Azure/AKS/releases

<!-- LINKS - internal -->
[azure-network-models]: concepts-network.md#azure-virtual-networks
[configure-azure-cni]: configure-azure-cni.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[windows-node-cli]: windows-container-cli.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[upgrade-cluster]: upgrade-cluster.md
[upgrade-cluster-cp]: use-multiple-node-pools.md#upgrade-a-cluster-control-plane-with-multiple-node-pools
[azure-outbound-traffic]: ../load-balancer/load-balancer-outbound-connections.md#defaultsnat
[nodepool-limitations]: use-multiple-node-pools.md#limitations
[windows-container-compat]: /virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-2019%2Cwindows-10-1909
[maximum-number-of-pods]: configure-azure-cni.md#maximum-pods-per-node
[azure-monitor]: ../azure-monitor/containers/container-insights-overview.md#what-does-azure-monitor-for-containers-provide
[client-source-ip]: concepts-network.md#ingress-controllers
[kubernetes-dashboard]: kubernetes-dashboard.md
[windows-rdp]: rdp.md
[upgrade-node-image]: node-image-upgrade.md
[managed-identity]: use-managed-identity.md
[hybrid-vms]: ../virtual-machines/windows/hybrid-use-benefit-licensing.md
[resource-groups]: faq.md#why-are-two-resource-groups-created-with-aks
[dsr]: ../load-balancer/load-balancer-multivip-overview.md#rule-type-2-backend-port-reuse-by-using-floating-ip