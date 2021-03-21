---
title: Настройка конфигурации узла для пулов узлов Azure Kubernetes Service (AKS) (Предварительная версия)
description: Узнайте, как настроить конфигурацию в узлах кластера Azure Kubernetes Service (AKS) и пулах узлов.
ms.service: container-service
ms.topic: article
ms.date: 12/03/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: 7b39242a7d7208b33a070e86088b25e9414ead04
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "101714635"
---
# <a name="customize-node-configuration-for-azure-kubernetes-service-aks-node-pools-preview"></a>Настройка конфигурации узла для пулов узлов Azure Kubernetes Service (AKS) (Предварительная версия)

Настройка конфигурации узла позволяет настраивать или настраивать параметры операционной системы (ОС) или параметры kubelet в соответствии с потребностями рабочих нагрузок. При создании кластера AKS или добавлении пула узлов в кластер можно настроить подмножество часто используемых параметров ОС и kubelet. Чтобы настроить параметры за пределами этого подмножества, [используйте набор управляющих программ для настройки необходимых конфигураций без поддержки разрыва AKS для узлов](support-policies.md#shared-responsibility).

## <a name="register-the-customnodeconfigpreview-preview-feature"></a>Регистрация `CustomNodeConfigPreview` функции предварительной версии

Чтобы создать кластер AKS или пул узлов, который может настраивать параметры kubelet или параметры операционной системы, необходимо включить `CustomNodeConfigPreview` флаг компонента в подписке.

Зарегистрируйте `CustomNodeConfigPreview` флаг компонента с помощью команды [AZ Feature Register][az-feature-register] , как показано в следующем примере:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "CustomNodeConfigPreview"
```

Через несколько минут отобразится состояние *Registered* (Зарегистрировано). Проверьте состояние регистрации с помощью команды [AZ Feature List][az-feature-list] .

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/CustomNodeConfigPreview')].{Name:name,State:properties.state}"
```

Когда все будет готово, обновите регистрацию поставщика ресурсов *Microsoft. ContainerService* с помощью команды [AZ Provider Register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="install-aks-preview-cli-extension"></a>Установка расширения интерфейса командной строки предварительной версии AKS

Чтобы создать кластер AKS или пул узлов, который может настраивать параметры kubelet или параметры операционной системы, требуется последняя версия расширения *AKS-preview* Azure CLI. Установите расширение Azure CLI *AKS-Preview* с помощью команды [AZ Extension Add][az-extension-add] . Или установите все доступные обновления с помощью команды [AZ Extension Update][az-extension-update] .

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
``` 

## <a name="use-custom-node-configuration"></a>Использовать конфигурацию настраиваемого узла

### <a name="kubelet-custom-configuration"></a>Настраиваемая конфигурация Kubelet

Поддерживаемые параметры Kubelet и допустимые значения перечислены ниже.

| Параметр | Допустимые значения и интервал | По умолчанию | Описание |
| --------- | ----------------------- | ------- | ----------- |
| `cpuManagerPolicy` | нет, статический | нет | Статическая политика позволяет контейнерам в [гарантированно гарантировать](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/) использование ЦЕЛОЧИСЛЕННЫХ ресурсов ЦП для доступа к эксклюзивным ЦП узла. |
| `cpuCfsQuota` | true, false | true |  Включение и отключение использования квоты CFS ЦП для контейнеров, задающих ограничения ЦП. | 
| `cpuCfsQuotaPeriod` | Интервал в миллисекундах (МС) | `100ms` | Задает значение периода квоты CFS ЦП. | 
| `imageGcHighThreshold` | 0-100 | 85 | Процент использования дискового пространства, по истечении которого всегда выполняется сборка мусора образа. Минимальное использование **диска,** запускающее сборку мусора. Для отключения сборки мусора образа установите значение 100. | 
| `imageGcLowThreshold` | 0-100, не выше `imageGcHighThreshold` | 80 | Процент использования диска, до которого сборка мусора образа никогда не выполняется. Минимальный объем использования дискового пространства, который **может** активировать сбор мусора. |
| `topologyManagerPolicy` | нет, оптимальный, ограниченный, одиночный NUMA-node | нет | Оптимизация выравнивания узлов NUMA см. [здесь](https://kubernetes.io/docs/tasks/administer-cluster/topology-manager/). Только kubernetes v 18E +. |
| `allowedUnsafeSysctls` | `kernel.shm*`, `kernel.msg*`, `kernel.sem`, `fs.mqueue.*`, `net.*` | Нет | Разрешенный список ненадежных сисктлс или ненадежных шаблонов сисктл. | 

### <a name="linux-os-custom-configuration"></a>Настраиваемая конфигурация ОС Linux

Ниже перечислены поддерживаемые параметры ОС и принятые значения.

#### <a name="file-handle-limits"></a>Ограничения для маркеров файлов

Когда вы обслуживаете много трафика, обычно трафик, который вы обслуживаете, поступает из большого числа локальных файлов. Вы можете настроить следующие параметры ядра и встроенные ограничения, чтобы вы могли управлять большим объемом системной памяти.

| Параметр | Допустимые значения и интервал | По умолчанию | Описание |
| ------- | ----------------------- | ------- | ----------- |
| `fs.file-max` | 8192-12000500 | 709620 | Максимальное количество дескрипторов файлов, выделяемых ядром Linux. увеличение этого значения позволяет увеличить максимально допустимое число открытых файлов. |
| `fs.inotify.max_user_watches` | 781250-2097152 | 1048576 | Максимальное количество отслеживаемых файлов, разрешенное системой. Каждый *Контрольный* размер составляет примерно 90 байт на 32-разрядном ядре и примерно 160 байт на 64-разрядном ядре. | 
| `fs.aio-max-nr` | 65536-6553500 | 65536 | AIO-nr показывает текущее системное количество асинхронных запросов на ввод-вывод. AIO-Max-Nr позволяет изменить максимальное значение AIO-Nr, которое может увеличиваться до. |
| `fs.nr_open` | 8192-20000500 | 1048576 | Максимальное число дескрипторов файлов, которые может выделить процесс. |


#### <a name="socket-and-network-tuning"></a>Настройка сокетов и сети

Для узлов агентов, которые должны работать с очень большим количеством одновременных сеансов, можно использовать подмножество параметров TCP и сети ниже, которые можно настроить для каждого пула узлов. 

| Параметр | Допустимые значения и интервал | По умолчанию | Описание |
| ------- | ----------------------- | ------- | ----------- |
| `net.core.somaxconn` | 4096-3240000 | 16384 | Максимальное количество запросов на соединение, которые могут быть поставлены в очередь для любого заданного прослушивающего сокета. Верхний предел значения параметра невыполненной работы, переданного функции [Listen (2)](http://man7.org/linux/man-pages/man2/listen.2.html) . Если аргумент невыполненной работы больше `somaxconn` , то он усекается без уведомления до этого предела.
| `net.core.netdev_max_backlog` | 1000-3240000 | 1000 | Максимальное количество пакетов, поставленное в очередь на стороне входа, когда интерфейс получает пакеты быстрее, чем ядро может их обработать. |
| `net.core.rmem_max` | 212992-134217728 | 212992 | Максимальный размер буфера сокета приема в байтах. |
| `net.core.wmem_max` | 212992-134217728 | 212992 | Максимальный размер буфера сокета отправки в байтах. | 
| `net.core.optmem_max` | 20480-4194304 | 20480 | Максимальный размер вспомогательного буфера (буфер памяти для параметра), допустимый для каждого сокета. Память параметра сокета используется в нескольких случаях для хранения дополнительных структур, относящихся к использованию сокета. | 
| `net.ipv4.tcp_max_syn_backlog` | 128-3240000 | 16384 | Максимальное количество запросов на подключение в очереди, которые по-прежнему не получили подтверждение от подключающегося клиента. Если это число превышено, ядро начнет удалять запросы. |
| `net.ipv4.tcp_max_tw_buckets` | 8000-1440000 | 32768 | Максимальное число `timewait` сокетов, удерживаемых системой одновременно. Если это число превышено, сокет времени ожидания уничтожается немедленно и выводится предупреждение. |
| `net.ipv4.tcp_fin_timeout` | 5 - 120 | 60 | Продолжительность времени, в течение которого потерянная (больше не указана ни одна из приложений) связь остается в состоянии FIN_WAIT_2, прежде чем оно будет прервано на локальном конце. |
| `net.ipv4.tcp_keepalive_time` | 30 - 432000 | 7200 | Частота отправки сообщений по протоколу TCP `keepalive` при `keepalive` включении. |
| `net.ipv4.tcp_keepalive_probes` | 1 - 15 | 9 | Сколько `keepalive` проверок ОТПРАВЛЯЕТ TCP, пока не решите, что соединение разорвано. |
| `net.ipv4.tcp_keepalive_intvl` | 1 - 75 | 75 | Частота отправки проб. Умножение на `tcp_keepalive_probes` это увеличивает время на закрытие соединения, которое не отвечает, после запуска проб. | 
| `net.ipv4.tcp_tw_reuse` | 0 или 1 | 0 | Разрешить повторно использовать `TIME-WAIT` сокеты для новых подключений, если точка зрения "протокол" является надежной. | 
| `net.ipv4.ip_local_port_range` | Первый: 1024-60999 и последний: 32768-65000] | Первый: 32768 и последний: 60999 | Диапазон локальных портов, используемый трафиком TCP и UDP для выбора локального порта. Состоит из двух чисел: первое число — это первый локальный порт, разрешенный для трафика TCP и UDP на узле агента, второй — последний номер локального порта. | 
| `net.ipv4.neigh.default.gc_thresh1`|  128-80000 | 4096 | Минимальное число записей, которые могут находиться в кэше ARP. Сборка мусора не будет активирована, если число записей ниже этого параметра. | 
| `net.ipv4.neigh.default.gc_thresh2`|  512-90000 | 8192 | Программное максимальное число записей, которые могут находиться в кэше ARP. Этот параметр является наиболее важным, так как сборка мусора для протокола ARP будет активирована примерно через 5 секунд после достижения этого мягкого максимума. |
| `net.ipv4.neigh.default.gc_thresh3`|  1024-100000 | 16384 | Жесткое максимальное число записей в кэше ARP. |
| `net.netfilter.nf_conntrack_max` | 131072-589824 | 131072 | `nf_conntrack` — это модуль, отслеживающий записи подключения для NAT в Linux. `nf_conntrack`Модуль использует хэш-таблицу для записи *установленной записи соединения* протокола TCP. `nf_conntrack_max` Максимальное количество узлов в хэш-таблице, то есть максимальное число соединений, поддерживаемое `nf_conntrack` модулем, или размер таблицы отслеживания соединений. | 
| `net.netfilter.nf_conntrack_buckets` | 65536-147456 | 65536 | `nf_conntrack` — это модуль, отслеживающий записи подключения для NAT в Linux. `nf_conntrack`Модуль использует хэш-таблицу для записи *установленной записи соединения* протокола TCP. `nf_conntrack_buckets` Размер хэш-таблицы. | 

#### <a name="worker-limits"></a>Ограничения рабочих ролей

Как и в случае с пределами дескрипторов файлов, количество рабочих процессов или потоков, которые может создать процесс, ограничено параметром ядра и ограничениями пользователя. Предельное число пользователей в AKS не ограничено. 

| Параметр | Допустимые значения и интервал | По умолчанию | Описание |
| ------- | ----------------------- | ------- | ----------- |
| `kernel.threads-max` | 20 - 513785 | 55601 | Процессы могут выполнять обработку рабочих потоков. Максимальное число создаваемых потоков задается параметром ядра `kernel.threads-max` . | 

#### <a name="virtual-memory"></a>Виртуальная память

Приведенные ниже параметры можно использовать для настройки работы подсистемы виртуальной памяти (ВМ) ядра Linux и `writeout` измененных данных на диск.

| Параметр | Допустимые значения и интервал | По умолчанию | Описание |
| ------- | ----------------------- | ------- | ----------- |
| `vm.max_map_count` |  65530-262144 | 65530 | Этот файл содержит максимальное число областей отображения памяти, которое может быть у процесса. Области карт памяти используются как побочные эффекты вызова `malloc` , непосредственно с помощью `mmap` , `mprotect` и `madvise` , а также при загрузке общих библиотек. | 
| `vm.vfs_cache_pressure` | 1 - 500 | 100 | Это процентное значение управляет обтенденциями ядра, чтобы освободить память, которая используется для кэширования объектов Directory и иноде. |
| `vm.swappiness` | 0–100 | 60 | Этот элемент управления используется для определения того, насколько агрессивно ядро будет менять страницы памяти. Более высокие значения увеличивают интенсивность, а низкие значения уменьшают объем подкачки. Значение 0 указывает ядру не инициировать переключение до тех пор, пока объем свободных и файловых страниц не превысит максимальный размер воды в зоне. | 
| `swapFileSizeMB` | 1 МБ — размер [временного диска](../virtual-machines/managed-disks-overview.md#temporary-disk) (/dev/sdb) | Нет | Свапфилесиземб указывает размер файла подкачки в МБ, который будет создан на узлах агента из этого пула узлов. | 
| `transparentHugePageEnabled` | `always`, `madvise`, `never` | `always` | [Прозрачный хужепажес](https://www.kernel.org/doc/html/latest/admin-guide/mm/transhuge.html#admin-guide-transhuge) — это функция ядра Linux, предназначенная для повышения производительности за счет более эффективного использования аппаратного обеспечения сопоставления памяти процессора. Если этот параметр включен, ядро пытается выделить память по `hugepages` мере возможности, и любой процесс Linux получит 2 – МБ страниц, если для `mmap` региона установлено 2 МБ естественное соответствие. В некоторых случаях `hugepages` , когда включена поддержка всей системы, приложения могут в конечном итоге выделить больше ресурсов памяти. Приложение может быть `mmap` крупным регионом, но оно затрагивает только 1 байт. в этом случае можно выделить страницу размером 2 МБ, а не 4-килобайтную страницу, что не является хорошей причиной. В этом сценарии объясняется, что можно отключить в `hugepages` масштабе всей системы или только в `MADV_HUGEPAGE madvise` регионах. | 
| `transparentHugePageDefrag` | `always`, `defer`, `defer+madvise`, `madvise`, `never` | `madvise` | Это значение определяет, должно ли ядро активно использовать сжатие памяти для повышения `hugepages` доступности. | 

> [!IMPORTANT]
> Для простоты поиска и удобочитаемости параметры ОС отображаются в этом документе по имени, но их следует добавить в файл конфигурации JSON или API AKS, используя [соглашение о капитализации camelCase](/dotnet/standard/design-guidelines/capitalization-conventions).

Создайте `kubeletconfig.json` файл со следующим содержимым:

```json
{
 "cpuManagerPolicy": "static",
 "cpuCfsQuota": true,
 "cpuCfsQuotaPeriod": "200ms",
 "imageGcHighThreshold": 90,
 "imageGcLowThreshold": 70,
 "topologyManagerPolicy": "best-effort",
 "allowedUnsafeSysctls": [
  "kernel.msg*",
  "net.*"
],
 "failSwapOn": false
}
```
Создайте `linuxosconfig.json` файл со следующим содержимым:

```json
{
 "transparentHugePageEnabled": "madvise",
 "transparentHugePageDefrag": "defer+madvise",
 "swapFileSizeMB": 1500,
 "sysctls": {
  "netCoreSomaxconn": 163849,
  "netIpv4TcpTwReuse": true,
  "netIpv4IpLocalPortRange": "32000 60000"
 }
}
```

Создайте новый кластер, указав конфигурации kubelet и ОС, используя JSON-файлы, созданные на предыдущем шаге. 

> [!NOTE]
> При создании кластера можно указать конфигурацию kubelet, конфигурацию ОС или и то, и другое. Если при создании кластера указать конфигурацию, то эта конфигурация будет применена только к узлам в начальном пуле узлов. Все параметры, не настроенные в JSON, будут хранить значение по умолчанию.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --kubelet-config ./kubeletconfig.json --linux-os-config ./linuxosconfig.json
```

Добавьте новый пул узлов, указав параметры Kubelet, используя созданный JSON-файл.

> [!NOTE]
> При добавлении пула узлов в существующий кластер можно указать конфигурацию kubelet, конфигурацию ОС или и то, и другое. Если при добавлении пула узлов указана конфигурация, то эта конфигурация будет применена только к узлам в новом пуле узлов. Все параметры, не настроенные в JSON, будут хранить значение по умолчанию.

```azurecli
az aks nodepool add --name mynodepool1 --cluster-name myAKSCluster --resource-group myResourceGroup --kubelet-config ./kubeletconfig.json
```

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте [, как настроить кластер AKS](cluster-configuration.md).
- Узнайте [, как обновить образы узлов](node-image-upgrade.md) в кластере.
- Сведения о том, как обновить кластер до последней версии Kubernetes, см. в статье [Обновление кластера Службы Azure Kubernetes (AKS)](upgrade-cluster.md).
- В списке [вопросов и ответов об AKS](faq.md) собраны самые популярные рекомендации.

<!-- LINKS - internal -->
[aks-faq]: faq.md
[aks-faq-node-resource-group]: faq.md#can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group
[aks-multiple-node-pools]: use-multiple-node-pools.md
[aks-scale-apps]: tutorial-kubernetes-scale.md
[aks-support-policies]: support-policies.md
[aks-upgrade]: upgrade-cluster.md
[aks-view-master-logs]: ./view-control-plane-logs.md#enable-resource-logs
[autoscaler-profile-properties]: #using-the-autoscaler-profile
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-update]: /cli/azure/aks#az-aks-update
[az-aks-scale]: /cli/azure/aks#az-aks-scale
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[upgrade-cluster]: upgrade-cluster.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[max-surge]: upgrade-cluster.md#customize-node-surge-upgrade


<!-- LINKS - external -->
[az-aks-update-preview]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview
[az-aks-nodepool-update]: https://github.com/Azure/azure-cli-extensions/tree/master/src/aks-preview#enable-cluster-auto-scaler-for-a-node-pool
[autoscaler-scaledown]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-types-of-pods-can-prevent-ca-from-removing-a-node
[autoscaler-parameters]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#what-are-the-parameters-to-ca
[kubernetes-faq]: https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/FAQ.md#ca-doesnt-work-but-it-used-to-work-yesterday-why