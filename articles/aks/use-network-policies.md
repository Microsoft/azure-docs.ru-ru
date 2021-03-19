---
title: Защита трафика групп pod с помощью политик сети
titleSuffix: Azure Kubernetes Service
description: Узнайте, как для защитить входящий и исходящий трафик групп pod с помощью политик сети Kubernetes в Службе Azure Kubernetes (AKS).
services: container-service
ms.topic: article
ms.date: 03/16/2021
ms.openlocfilehash: 17e14859ecdfe11872d5b0526d755d01bc1b034a
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104577858"
---
# <a name="secure-traffic-between-pods-using-network-policies-in-azure-kubernetes-service-aks"></a>Защита трафика между группами pod с использованием политик сети в Службе Azure Kubernetes (AKS)

При выполнении современных приложений на базе микрослужб в Kubernetes часто требуется управлять тем, какие компоненты могут взаимодействовать друг с другом. К передаче трафика между группами pod в кластере Службы Azure Kubernetes (AKS) следует применить принцип наименьших привилегий. Например, скорее всего трафик, передаваемый напрямую в серверные приложения, нужно будет блокировать. В Kubernetes доступны *политики сети*, которые позволяют определить правила для входящего и исходящего трафика, передаваемого между группами pod в кластере.

В этой статье показано, как установить подсистему политик сети и создать политики сети Kubernetes для управления потоком трафика между группами pod в AKS. Политику сети следует использовать только для узлов под управлением Linux и групп pod в AKS.

## <a name="before-you-begin"></a>Перед началом

Необходимо установить и настроить Azure CLI версии 2.0.61 или более поздней. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0][install-azure-cli].

> [!TIP]
> Если вы использовали функцию политики сети на этапе предварительной версии, рекомендуется [создать новый кластер](#create-an-aks-cluster-and-enable-network-policy).
> 
> Если вы хотите продолжить использовать существующие тестовые кластеры, которые использовали политику сети на этапе предварительной версии, обновите кластер до новой версии Kubernetes для установки последнего общедоступного выпуска, а затем разверните приведенный ниже манифест YAML, чтобы устранить сбой сервера метрик и панели мониторинга Kubernetes. Это исправление требуется только для кластеров, в которых использовалась подсистема политик сети Calico.
>
> Для обеспечения безопасности рекомендуется [проверить содержимое этого манифеста YAML][calico-aks-cleanup], чтобы понять, что развертывается в кластере AKS.
>
> `kubectl delete -f https://raw.githubusercontent.com/Azure/aks-engine/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml`

## <a name="overview-of-network-policy"></a>Общие сведения о политике сети

По умолчанию все группы pod в кластере AKS могут отправлять и получать трафик без ограничений. Для повышения безопасности вы можете определить правила, управляющие потоком трафика. Например, серверные приложения часто предоставляются только обязательным интерфейсным службам. Или компоненты базы данных доступны только для уровней приложений, которые к ним подключаются.

Политика сети — это спецификация Kubernetes, определяющая политики доступа для обмена данными между группами pod. С помощью политик сети можно определить упорядоченный набор правил отправки и получения трафика и применить их к коллекции pod, соответствующей одному или нескольким селекторам меток.

Правила политики сети определяются как манифесты YAML. Политики сети могут быть включены в более глобальный манифест, который помимо политики сети создает развертывание или службу.

### <a name="network-policy-options-in-aks"></a>Параметры политики сети в AKS

Azure предоставляет два способа реализации политики сети. При создании кластера AKS можно выбрать параметр политики сети. После создания кластера изменить параметр политики невозможно.

* Собственная реализация Azure, которая называется *политиками сети Azure*.
* *Политики сети Calico* — решение с открытым кодом для организации сетей и обеспечения сетевой безопасности, созданное группой разработчиков [Tigera][tigera].

В обеих реализациях для применения указанных политик используется *IPTables* Linux. Политики преобразовываются в наборы разрешенных и запрещенных пар IP-адресов. Затем эти пары программируются как правила фильтрации IPTables.

### <a name="differences-between-azure-and-calico-policies-and-their-capabilities"></a>Различия между политиками Azure и Калико и их возможностями

| Функция                               | Azure                      | Calico                      |
|------------------------------------------|----------------------------|-----------------------------|
| Поддерживаемые платформы                      | Linux                      | Linux, Windows Server 2019 (Предварительная версия)  |
| Поддерживаемые сетевые подключения             | Azure CNI                  | Azure CNI (Windows Server 2019 и Linux) и кубенет (Linux)  |
| Соответствие спецификации Kubernetes | Поддерживаются все типы политик. |  Поддерживаются все типы политик. |
| Дополнительные функции                      | None                       | Расширенная модель политики, состоящая из политики глобальной сети, набора глобальных сетей и конечной точки узла. Дополнительные сведения об использовании интерфейса командной строки `calicoctl` для управления этими расширенными функциями см. в [справочных материалах по calicoctl][calicoctl]. |
| Поддержка                                  | Поддержку осуществляет группа поддержки и разработки Azure. | Поддержку осуществляет сообщество Calico. Сведения о дополнительной платной поддержке см. на странице [вариантов поддержки проекта Calico][calico-support]. |
| Logging                                  | Правила, добавленные или удаленные в IPTables, регистрируются на каждом узле в разделе */var/log/azure-npm.log*. | Дополнительные сведения см. в разделе [Calico component logs][calico-logs] (Журналы компонентов Calico). |

## <a name="create-an-aks-cluster-and-enable-network-policy"></a>Создание кластера AKS и включение политики сети

Чтобы посмотреть, как действуют политики сети, давайте создадим и развернем политику, определяющую поток трафика следующим образом:

* запрет любого трафика к pod;
* разрешение трафика на основе меток pod;
* разрешение трафика на основе пространства имен.

Сначала создадим кластер AKS, который поддерживает политику сети.

> [!IMPORTANT]
>
> Функцию политики сети можно включить только при создании кластера. Вы не можете включить политику сети в существующем кластере AKS.

Чтобы использовать политику сети Azure, необходимо использовать [подключаемый модуль Azure CNI][azure-cni], а также определить собственную виртуальную сеть и подсети. Более подробные сведения о том, как планировать необходимые диапазоны подсетей, см. в разделе [Настройка сети Azure CNI в Службе Azure Kubernetes (AKS)][use-advanced-networking]. Политику сети Calico можно использовать с тем же подключаемым модулем Azure CNI или с подключаемым модулем Kubenet CNI.

Ниже приведен пример сценария, который выполняет следующее:

* Создает виртуальную сеть и подсеть.
* Создает субъект-службу Azure Active Directory (Azure AD) для кластера AKS.
* Назначает субъекту-службе кластера AKS разрешения *Участник* для виртуальной сети.
* Создает кластер AKS в определенной виртуальной сети и включает политику сети.
    * При этом используется политика _сети Azure_. Чтобы вместо этого использовать политику сети Calico, используйте параметр `--network-policy calico`. Примечание. Calico можно использовать с `--network-plugin azure` или `--network-plugin kubenet`.

Учтите, для разрешений вместо субъекта-службы можно использовать управляемое удостоверение. Дополнительные сведения см. в статье о том, [как использовать управляемые удостоверения](use-managed-identity.md).

Укажите собственный безопасный пароль *SP_PASSWORD*. Можно заменить переменные *RESOURCE_GROUP_NAME* и *CLUSTER_NAME*.

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup-NP
CLUSTER_NAME=myAKSCluster
LOCATION=canadaeast

# Create a resource group
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

# Create a virtual network and subnet
az network vnet create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16

# Create a service principal and read in the application ID
SP=$(az ad sp create-for-rbac --output json)
SP_ID=$(echo $SP | jq -r .appId)
SP_PASSWORD=$(echo $SP | jq -r .password)

# Wait 15 seconds to make sure that service principal has propagated
echo "Waiting for service principal to propagate..."
sleep 15

# Get the virtual network resource ID
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP_NAME --name myVnet --query id -o tsv)

# Assign the service principal Contributor permissions to the virtual network resource
az role assignment create --assignee $SP_ID --scope $VNET_ID --role Contributor

# Get the virtual network subnet resource ID
SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP_NAME --vnet-name myVnet --name myAKSSubnet --query id -o tsv)
```

### <a name="create-an-aks-cluster-for-azure-network-policies"></a>Создание кластера AKS для политик сети Azure

Создайте кластер AKS и укажите виртуальную сеть, сведения о субъекте-службе и *Azure* для сетевого подключаемого модуля и политики сети.

```azurecli
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 1 \
    --generate-ssh-keys \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal $SP_ID \
    --client-secret $SP_PASSWORD \
    --network-plugin azure \
    --network-policy azure
```

Создание кластера занимает несколько минут. Когда кластер будет готов, выполните команду [az aks get-credentials][az-aks-get-credentials], чтобы настроить `kubectl` для подключения к кластеру Kubernetes. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

### <a name="create-an-aks-cluster-for-calico-network-policies"></a>Создание кластера AKS для политик сети Калико

Создайте кластер AKS и укажите виртуальную сеть, сведения о субъекте-службе, *Azure* для сетевого подключаемого модуля и *Калико* для политики сети. Использование *Калико* в качестве политики сети позволяет Калико сети в пулах узлов Linux и Windows.

Если вы планируете добавить в кластер пулы узлов Windows, включите `windows-admin-username` Параметры и в `windows-admin-password` соответствии с [требованиями к паролю Windows Server][windows-server-password]. Чтобы использовать Калико с пулами узлов Windows, необходимо также зарегистрировать `Microsoft.ContainerService/EnableAKSWindowsCalico` .

Зарегистрируйте `EnableAKSWindowsCalico` флаг функции с помощью команды [AZ Feature Register][az-feature-register] , как показано в следующем примере:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "EnableAKSWindowsCalico"
```

 Состояние регистрации можно проверить с помощью команды [az feature list][az-feature-list].

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableAKSWindowsCalico')].{Name:name,State:properties.state}"
```

Когда все будет готово, обновите регистрацию поставщика ресурсов *Microsoft. ContainerService* с помощью команды [AZ Provider Register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

> [!IMPORTANT]
> В настоящее время использование политик Калико сети с узлами Windows доступно в новых кластерах с помощью Kubernetes версии 1,20 или более поздней с Калико 3.17.2 и требует использования Azure CNI Networking. Узлы Windows в кластерах AKS с включенной функцией Калико также обладают [прямым возвратом сервера (DSR)][dsr] по умолчанию.
>
> Для кластеров, использующих только пулы узлов Linux под управлением Kubernetes 1,20 с более ранними версиями Калико, версия Калико будет автоматически обновлена до 3.17.2.

Политики сети Калико с узлами Windows в настоящее время находятся на этапе предварительной версии.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Создайте имя пользователя для использования в качестве учетных данных администратора для контейнеров Windows Server в кластере. Следующие команды запрашивают имя пользователя и задают его WINDOWS_USERNAME для использования в более поздней команде (Помните, что команды в этой статье введены в оболочку BASH).

```azurecli-interactive
echo "Please enter the username to use as administrator credentials for Windows Server containers on your cluster: " && read WINDOWS_USERNAME
```

```azurecli
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 1 \
    --generate-ssh-keys \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal $SP_ID \
    --client-secret $SP_PASSWORD \
    --windows-admin-username $WINDOWS_USERNAME \
    --vm-set-type VirtualMachineScaleSets \
    --kubernetes-version 1.20.2 \
    --network-plugin azure \
    --network-policy calico
```

Создание кластера занимает несколько минут. По умолчанию кластер создается только с пулом узлов Linux. Если вы хотите использовать пулы узлов Windows, можно добавить их. Пример:

```azurecli
az aks nodepool add \
    --resource-group $RESOURCE_GROUP_NAME \
    --cluster-name $CLUSTER_NAME \
    --os-type Windows \
    --name npwin \
    --node-count 1
```

Когда кластер будет готов, выполните команду [az aks get-credentials][az-aks-get-credentials], чтобы настроить `kubectl` для подключения к кластеру Kubernetes. Эта команда скачивает учетные данные и настраивает интерфейс командной строки Kubernetes для их использования.

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

## <a name="deny-all-inbound-traffic-to-a-pod"></a>Запрещение всего входящего трафика pod

Прежде чем определить правила, разрешающие определенный сетевой трафик, создайте политику сети, чтобы запретить весь трафик. Эта политика дает отправную точку для начала создания разрешенных только для требуемого трафика. Кроме того, вы увидите, как трафик блокируется при применении политики сети.

Давайте сначала создадим для среды с примером приложения и правилами трафика пространство имен *development* для запуска примеров pod.

```console
kubectl create namespace development
kubectl label namespace/development purpose=development
```

Теперь создайте пример серверной группы pod, в которой выполняется NGINX. Эту серверную группу pod можно использовать для имитации серверного веб-приложения. Создайте этот pod в пространстве имен *development* и откройте порт *80* для обслуживания веб-трафика. Добавьте к pod метку с помощью *app=webapp,role=backend*. Так мы сможем указать его в политике сети в приведенном ниже разделе.

```console
kubectl run backend --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --labels app=webapp,role=backend --namespace development --expose --port 80
```

Чтобы проверить, можно ли связаться с веб-страницей NGINX по умолчанию, создайте другую группу pod и подключите сеанс терминала.

```console
kubectl run --rm -it --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 network-policy --namespace development
```

В командной строке оболочки введите `wget`, чтобы подтвердить, что вы можете получить доступ к веб-странице NGINX по умолчанию.

```console
wget -qO- http://backend
```

В следующем примере выходных данных показано, что возвращена веб-страница NGINX по умолчанию.

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Выйдите из подключенного сеанса терминала. Тестовая группа pod автоматически удаляется.

```console
exit
```

### <a name="create-and-apply-a-network-policy"></a>Создание и применение политики сети

Теперь, когда вы убедились, что базовая веб-страница NGINX в примере серверной группы pod доступна, создайте политику сети, чтобы запретить весь трафик. Создайте файл `backend-policy.yaml` и вставьте в него следующий манифест YAML. Этот манифест использует *podSelector* для подключения политики к группам pod с меткой *app:webapp,role:backend* — как у вашего примера pod NGINX. Правила в разделе *ingress* не определены, поэтому запрещен весь входящий трафик к pod.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress: []
```

Перейдите по адресу [https://shell.azure.com](https://shell.azure.com), чтобы открыть Azure Cloud Shell в браузере.

Примените политику сети с помощью команды [kubectl apply][kubectl-apply] и укажите имя манифеста YAML.

```console
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-network-policy"></a>Проверка политики сети

Давайте узнаем, сможете ли вы использовать веб-страницу NGINX в серверной группе pod. Создайте еще один тестовый pod и подключите сеанс терминала.

```console
kubectl run --rm -it --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 network-policy --namespace development
```

В командной строке оболочки введите `wget`, чтобы узнать, можете ли вы получить доступ к веб-странице NGINX по умолчанию. На этот раз установите значение времени ожидания в *2* секунды. Теперь политика сети блокирует весь входящий трафик, поэтому загрузить страницу не удается, как показано в следующем примере.

```console
wget -qO- --timeout=2 http://backend
```

```output
wget: download timed out
```

Выйдите из подключенного сеанса терминала. Тестовая группа pod автоматически удаляется.

```console
exit
```

## <a name="allow-inbound-traffic-based-on-a-pod-label"></a>Разрешение входящего трафика на основе метки pod

В предыдущем разделе был запланирован запуск внутренней группы pod NGINX и создана политика сети, которая запрещает весь трафик. Давайте создадим интерфейсную группу pod и обновим политику сети, чтобы разрешить трафик от интерфейсных групп pod.

Обновите политику сети, чтобы разрешить трафик из групп pod с метками *app:webapp,role:frontend* в любом пространстве имен. Измените предыдущий файл *backend-policy.yaml* и добавьте правила входящего трафика *matchLabels*, чтобы манифест выглядел как в следующем примере.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

> [!NOTE]
> Эта политика сети использует элементы *namespaceSelector* и *podSelector* для правила входящего трафика. Важно использовать синтаксис YAML, чтобы правила входящего трафика были аддитивными. В этом примере оба элемента должны совпадать для применения правила входящего трафика. Версии Kubernetes, предшествующие *1.12*, могут применять некорректную интерпретацию этих элементов и, естественно, ограничивать трафик. Дополнительные сведения об этой реакции на событие см. в статье [Behavior of to and from selectors][policy-rules] (Поведение входных и исходных селекторов).

Примените обновленную политику сети с помощью команды [kubectl apply][kubectl-apply] и укажите имя манифеста YAML.

```console
kubectl apply -f backend-policy.yaml
```

Запланируйте запуск pod с метками *app=webapp,role=frontend* и подключите сеанс терминала.

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace development
```

В командной строке оболочки введите `wget`, чтобы узнать, можете ли вы получить доступ к веб-странице NGINX по умолчанию.

```console
wget -qO- http://backend
```

Так как правило входящего трафика разрешает трафик из групп pod, имеющих метки *app:webapp,role:frontend*, трафик из серверной группы pod разрешен. В следующем примере выходных данных показано, что возвращена веб-страница NGINX по умолчанию.

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Выйдите из подключенного сеанса терминала. Группа pod автоматически удаляется.

```console
exit
```

### <a name="test-a-pod-without-a-matching-label"></a>Тестирование pod без соответствующей метки

Политика сети разрешает трафик из групп pod с метками *app:webapp,role:frontend*, но должна блокировать весь остальной трафик. Давайте проверим, сможет ли другая группа pod без этих меток получить доступ к серверной группе pod NGINX. Создайте еще один тестовый pod и подключите сеанс терминала.

```console
kubectl run --rm -it --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 network-policy --namespace development
```

В командной строке оболочки введите `wget`, чтобы узнать, можете ли вы получить доступ к веб-странице NGINX по умолчанию. Политика сети блокирует входящий трафик, поэтому загрузить страницу не удается, как показано в следующем примере.

```console
wget -qO- --timeout=2 http://backend
```

```output
wget: download timed out
```

Выйдите из подключенного сеанса терминала. Тестовая группа pod автоматически удаляется.

```console
exit
```

## <a name="allow-traffic-only-from-within-a-defined-namespace"></a>Разрешение трафика, передаваемого в пределах заданного пространства имен

В предыдущих примерах вы создали политику сети, которая запрещает весь трафик, а затем изменили ее, чтобы разрешить трафик из групп pod с определенной меткой. Нередко требуется разрешить только трафик, передаваемый в пределах указанного пространства имен. Если предыдущие примеры предназначались для трафика в пространстве имен *development*, создайте политику сети, которая блокирует трафик из другого пространства имен, например *production*, не пропуская его к группам pod.

Сначала создайте пространство имен для имитации рабочего пространства имен.

```console
kubectl create namespace production
kubectl label namespace/production purpose=production
```

Запланируйте запуск тестового pod в пространстве имен *production*, которое помечено как *app=webapp,role=frontend*. Подключите сеанс терминала.

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace production
```

В командной строке оболочки введите `wget`, чтобы подтвердить, что вы можете получить доступ к веб-странице NGINX по умолчанию.

```console
wget -qO- http://backend.development
```

Так как метки для pod совпадают с метками, разрешенными в политике сети, трафик не блокируется. Политика сети проверяет не пространства имен, а метки pod. В следующем примере выходных данных показано, что возвращена веб-страница NGINX по умолчанию.

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Выйдите из подключенного сеанса терминала. Тестовая группа pod автоматически удаляется.

```console
exit
```

### <a name="update-the-network-policy"></a>Обновление политики сети

Давайте обновим раздел *namespaceSelector* правила входящего трафика, чтобы разрешить только трафик из пространства имен *development*. Измените файл манифеста *backend-policy.yaml*, как показано в следующем примере.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: development
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

В более сложных примерах может быть определено несколько правил входящего трафика, например, использующих *namespaceSelector*, а затем — *podSelector*.

Примените обновленную политику сети с помощью команды [kubectl apply][kubectl-apply] и укажите имя манифеста YAML.

```console
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-updated-network-policy"></a>Тестирование обновленной политики сети

Запланируйте запуск еще одной группы pod в пространстве имен *production* и подключите сеанс терминала.

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace production
```

В командной строке оболочки введите `wget`, чтобы увидеть, что теперь политика сети запрещает трафик.

```console
wget -qO- --timeout=2 http://backend.development
```

```output
wget: download timed out
```

Выйдите из тестового pod.

```console
exit
```

Вы запретили трафик пространства имен *production*. Теперь запланируйте запуск тестовой группы pod в пространстве имен *development* и подключите сеанс терминала.

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace development
```

В командной строке оболочки введите `wget`, чтобы увидеть, что теперь политика сети разрешает трафик.

```console
wget -qO- http://backend
```

Так как запуск группы pod запланирован в пространстве имен, разрешенном в политике сети, трафик не блокируется. В следующем примере выходных данных показана возвращенная веб-страница NGINX по умолчанию.

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Выйдите из подключенного сеанса терминала. Тестовая группа pod автоматически удаляется.

```console
exit
```

## <a name="clean-up-resources"></a>Очистка ресурсов

В этой статье мы создали два пространства имен и применили политику сети. Чтобы очистить эти ресурсы, используйте команду [kubectl delete][kubectl-delete] и укажите имена ресурсов.

```console
kubectl delete namespace production
kubectl delete namespace development
```

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о сетевых ресурсах см. в разделе [Основные понятия сети в Службе Azure Kubernetes (AKS)][concepts-network].

Чтобы узнать больше об использовании политик, ознакомьтесь с разделом [Network Policies][kubernetes-network-policies] (Политики сети).

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubernetes-network-policies]: https://kubernetes.io/docs/concepts/services-networking/network-policies/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[policy-rules]: https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors
[aks-github]: https://github.com/azure/aks/issues
[tigera]: https://www.tigera.io/
[calicoctl]: https://docs.projectcalico.org/reference/calicoctl/
[calico-support]: https://www.tigera.io/tigera-products/calico/
[calico-logs]: https://docs.projectcalico.org/maintenance/troubleshoot/component-logs
[calico-aks-cleanup]: https://github.com/Azure/aks-engine/blob/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[use-advanced-networking]: configure-azure-cni.md
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[concepts-network]: concepts-network.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[windows-server-password]: /windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[dsr]: ../load-balancer/load-balancer-multivip-overview.md#rule-type-2-backend-port-reuse-by-using-floating-ip