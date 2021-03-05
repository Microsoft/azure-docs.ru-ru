---
title: Подключение RDP к AKS узлам Windows Server
titleSuffix: Azure Kubernetes Service
description: Узнайте, как создать подключение по протоколу RDP с узлами Windows Server в кластере Azure Kubernetes Service (AKS) для решения задач по устранению неполадок и обслуживанию.
services: container-service
ms.topic: article
ms.date: 06/04/2019
ms.openlocfilehash: 4cfac73c66969148927897de7ed6da0c56aa276a
ms.sourcegitcommit: 24a12d4692c4a4c97f6e31a5fbda971695c4cd68
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102173823"
---
# <a name="connect-with-rdp-to-azure-kubernetes-service-aks-cluster-windows-server-nodes-for-maintenance-or-troubleshooting"></a>Подключение по протоколу RDP к узлам Windows Server в кластере Kubernetes Service (AKS) для обслуживания или устранения неполадок

На протяжении жизненного цикла кластера Azure Kubernetes Service (AKS) может потребоваться доступ к узлу AKS Windows Server. Этот доступ используется для обслуживания, сбора журналов или других операций по устранению неполадок. Доступ к узлам AKS Windows Server можно получить с помощью протокола удаленного рабочего стола. Кроме того, если вы хотите использовать SSH для доступа к узлам AKS Windows Server и у вас есть доступ к одной и той же пары ключей, которая использовалась при создании кластера, вы можете выполнить действия, описанные в [разделе Подключение SSH к узлу кластера Azure Kubernetes Service (AKS)][ssh-steps]. В целях безопасности узлы Службы Azure Kubernetes (AKS) недоступны через Интернет.

В этой статье показано, как создать подключение RDP с узлом AKS, используя их частные IP-адреса.

## <a name="before-you-begin"></a>Перед началом

В этой статье предполагается, что у вас уже есть кластер AKS с узлом Windows Server. Если вам нужен кластер AKS, см. статью о [создании кластера AKS с контейнером Windows с помощью Azure CLI][aks-windows-cli]. Необходимо имя пользователя и пароль администратора Windows для узла Windows Server, для которого требуется устранить неполадки. Если вы не знакомы с ними, их можно сбросить, выполнив [сброс службы удаленных рабочих столов или пароль администратора на виртуальной машине Windows ](../virtual-machines/troubleshooting/reset-rdp.md). Вам также потребуется клиент RDP, например [Удаленный рабочий стол (Майкрософт)][rdp-mac].

Также требуется Azure CLI версии 2.0.61 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

## <a name="deploy-a-virtual-machine-to-the-same-subnet-as-your-cluster"></a>Развертывание виртуальной машины в той же подсети, что и кластер

На узлах Windows Server кластера AKS нет доступных извне IP-адресов. Чтобы создать подключение по протоколу RDP, можно развернуть виртуальную машину с общедоступным IP-адресом в той же подсети, что и узлы Windows Server.

В следующем примере создается виртуальная машина с именем *myVM* в группе ресурсов *myResourceGroup* .

Сначала получите подсеть, используемую пулом узлов Windows Server. Чтобы получить идентификатор подсети, необходимо имя подсети. Чтобы получить имя подсети, требуется имя виртуальной сети. Получите имя виртуальной сети, выполнив запрос к кластеру для получения списка сетей. Чтобы запросить кластер, необходимо его имя. Все эти действия можно получить, выполнив следующую команду в Azure Cloud Shell:

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
VNET_NAME=$(az network vnet list -g $CLUSTER_RG --query [0].name -o tsv)
SUBNET_NAME=$(az network vnet subnet list -g $CLUSTER_RG --vnet-name $VNET_NAME --query [0].name -o tsv)
SUBNET_ID=$(az network vnet subnet show -g $CLUSTER_RG --vnet-name $VNET_NAME --name $SUBNET_NAME --query id -o tsv)
```

Теперь, когда у вас есть SUBNET_ID, выполните следующую команду в том же окне Azure Cloud Shell, чтобы создать виртуальную машину:

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image win2019datacenter \
    --admin-username azureuser \
    --admin-password myP@ssw0rd12 \
    --subnet $SUBNET_ID \
    --query publicIpAddress -o tsv
```

В следующем примере выходных данных показано, что виртуальная машина успешно создана, и отобразится общедоступный IP-адрес виртуальной машины.

```console
13.62.204.18
```

Запишите общедоступный IP-адрес виртуальной машины. Этот адрес будет использоваться на следующем шаге.

## <a name="allow-access-to-the-virtual-machine"></a>Разрешить доступ к виртуальной машине

Подсети пула узлов AKS по умолчанию защищаются с помощью группы безопасности сети (групп безопасности сети). Чтобы получить доступ к виртуальной машине, необходимо включить доступ в NSG.

> [!NOTE]
> Группы безопасности сети управляются службой AKS. Любые изменения, вносимые в NSG, будут перезаписаны на плоскости управления в любое время.
>

Сначала получите группу ресурсов и NSG имя NSG, чтобы добавить правило:

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NSG_NAME=$(az network nsg list -g $CLUSTER_RG --query [].name -o tsv)
```

Затем создайте правило NSG:

```azurecli-interactive
az network nsg rule create --name tempRDPAccess --resource-group $CLUSTER_RG --nsg-name $NSG_NAME --priority 100 --destination-port-range 3389 --protocol Tcp --description "Temporary RDP access to Windows nodes"
```

## <a name="get-the-node-address"></a>Получение адреса узла

Управлять кластером Kubernetes можно c помощью [kubectl][kubectl], клиента командной строки Kubernetes. Если вы используете Azure Cloud Shell, `kubectl` уже установлен. Чтобы установить `kubectl` локально, используйте команду [az aks install-cli][az-aks-install-cli]:
    
```azurecli-interactive
az aks install-cli
```

Чтобы настроить `kubectl` на подключение к кластеру Kubernetes, выполните команду [az aks get-credentials][az-aks-get-credentials]. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Выведите список внутренних IP-адресов узлов Windows Server с помощью команды [kubectl Get][kubectl-get] :

```console
kubectl get nodes -o wide
```

В выходных данных примера ниже показаны внутренние IP-адреса всех узлов в кластере, включая узлы Windows Server.

```console
$ kubectl get nodes -o wide
NAME                                STATUS   ROLES   AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                    KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-42485177-vmss000000   Ready    agent   18h   v1.12.7   10.240.0.4    <none>        Ubuntu 16.04.6 LTS          4.15.0-1040-azure   docker://3.0.4
aksnpwin000000                      Ready    agent   13h   v1.12.7   10.240.0.67   <none>        Windows Server Datacenter   10.0.17763.437
```

Запишите внутренний IP-адрес узла Windows Server, который вы хотите устранить. Этот адрес будет использоваться на следующем шаге.

## <a name="connect-to-the-virtual-machine-and-node"></a>Подключение к виртуальной машине и узлу

Подключитесь к общедоступному IP-адресу виртуальной машины, созданной ранее с помощью клиента RDP, например [Удаленный рабочий стол (Майкрософт)][rdp-mac].

![Изображение подключения к виртуальной машине с помощью клиента RDP](media/rdp/vm-rdp.png)

После подключения к виртуальной машине подключитесь к *внутреннему IP-адресу* узла Windows Server, на котором вы хотите устранить неполадки, используя клиент RDP из виртуальной машины.

![Изображение подключения к узлу Windows Server с помощью клиента RDP](media/rdp/node-rdp.png)

Теперь вы подключены к узлу Windows Server.

![Изображение окна CMD в узле Windows Server](media/rdp/node-session.png)

Теперь можно выполнять любые команды устранения неполадок в окне *cmd* . Поскольку узлы Windows Server используют Windows Server Core, при подключении к узлу Windows Server по протоколу удаленного рабочего стола не нужно использовать полный графический интерфейс или другие средства графического пользовательского интерфейса.

## <a name="remove-rdp-access"></a>Удаление доступа по протоколу RDP

После этого Завершите подключение RDP к узлу Windows Server и завершите сеанс RDP на виртуальной машине. После выхода из обоих сеансов RDP Удалите виртуальную машину с помощью команды [AZ VM Delete][az-vm-delete] :

```azurecli-interactive
az vm delete --resource-group myResourceGroup --name myVM
```

И правило NSG:

```azurecli-interactive
CLUSTER_RG=$(az aks show -g myResourceGroup -n myAKSCluster --query nodeResourceGroup -o tsv)
NSG_NAME=$(az network nsg list -g $CLUSTER_RG --query [].name -o tsv)
```

```azurecli-interactive
az network nsg rule delete --resource-group $CLUSTER_RG --nsg-name $NSG_NAME --name tempRDPAccess
```

## <a name="next-steps"></a>Дальнейшие действия

Если требуются дополнительные данные по устранению неполадок, можно [Просмотреть журналы главного узла Kubernetes][view-master-logs] или [Azure Monitor][azure-monitor-containers].

<!-- EXTERNAL LINKS -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[rdp-mac]: https://aka.ms/rdmac

<!-- INTERNAL LINKS -->
[aks-windows-cli]: windows-container-cli.md
[az-aks-install-cli]: /cli/azure/aks#az-aks-install-cli
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-vm-delete]: /cli/azure/vm#az-vm-delete
[azure-monitor-containers]: ../azure-monitor/containers/container-insights-overview.md
[install-azure-cli]: /cli/azure/install-azure-cli
[ssh-steps]: ssh.md
[view-master-logs]: view-master-logs.md