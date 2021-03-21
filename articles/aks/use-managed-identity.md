---
title: Использование управляемых удостоверений в службе Kubernetes Azure
description: Узнайте, как использовать управляемые удостоверения в службе Kubernetes Azure (AKS).
services: container-service
ms.topic: article
ms.date: 12/16/2020
ms.openlocfilehash: 3ace7f1c93ab3918f460d245a863db43d98f1db5
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102176099"
---
# <a name="use-managed-identities-in-azure-kubernetes-service"></a>Использование управляемых удостоверений в службе Kubernetes Azure

В настоящее время кластер Azure Kubernetes Service (AKS) (в частности, поставщик облачных служб Kubernetes) требует удостоверения для создания дополнительных ресурсов, таких как подсистемы балансировки нагрузки и управляемые диски в Azure. Это может быть *управляемое удостоверение* или субъект- *Служба*. При использовании [субъекта-службы](kubernetes-service-principal.md)необходимо предоставить один или AKS от вашего имени. Если вы используете управляемое удостоверение, оно будет создано автоматически AKS. Кластеры, использующие субъекты-службы, в конечном итоге достигают состояния, в котором необходимо обновить субъект-службу, чтобы обеспечить работу кластера. Управление субъектами-службами повышает сложность, поэтому вместо этого проще использовать управляемые удостоверения. Те же требования к разрешениям применяются и к субъектам-службам, и к управляемым удостоверениям.

*Управляемые удостоверения* по сути являются оболочкой для субъектов-служб и упрощают их управление. Поворот учетных данных для MI происходит автоматически каждые 46 дней в соответствии с Azure Active Directory по умолчанию. AKS использует управляемые системой и назначенные пользователем типы удостоверений. В настоящее время эти удостоверения неизменяемы. Дополнительные сведения см. в статье об [управляемых удостоверениях для ресурсов Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="before-you-begin"></a>Перед началом

Необходимо установить следующий ресурс:

- Azure CLI версии 2.15.1 или более поздней.

## <a name="limitations"></a>Ограничения

* Клиенты, перемещая и миграция управляемых удостоверений, поддерживают кластеры.
* Если в кластере `aad-pod-identity` включены Node-Managed, модули iptables Identity (NMI) могут изменить узлы, чтобы перехватить вызовы к конечной точке метаданных экземпляра Azure. Такая конфигурация означает, что любой запрос, сделанный в конечной точке метаданных, перехватывается функцией NMI, даже если Pod не используется `aad-pod-identity` . Азуреподидентитексцептион CRD можно настроить так, чтобы сообщать `aad-pod-identity` о том, что любые запросы к конечной точке метаданных, созданные из Pod, совпадающие с метками, определенными в CRD, должны быть прокси-серверами без обработки в NMI. Системные модули с `kubernetes.azure.com/managedby: aks` меткой в пространстве имен _KUBE-System_ должны быть исключены в `aad-pod-identity` с помощью настройки азуреподидентитексцептион CRD. Дополнительные сведения см. [в разделе Отключение AAD-Pod-Identity для конкретного Pod или приложения](https://azure.github.io/aad-pod-identity/docs/configure/application_exception).
  Чтобы настроить исключение, установите [YAML-исключение MIC](https://github.com/Azure/aad-pod-identity/blob/master/deploy/infra/mic-exception.yaml).

## <a name="summary-of-managed-identities"></a>Сводка по управляемым удостоверениям

AKS использует несколько управляемых удостоверений для встроенных служб и надстроек.

| Идентификация                       | Имя    | Вариант использования | Разрешения по умолчанию | Приведите свое удостоверение
|----------------------------|-----------|----------|
| Уровень управления | не видно | Используется компонентами плоскости управления AKS для управления ресурсами кластера, включая подсистемы балансировки нагрузки и управляемые общедоступные IP-адреса AKS, а также операции автомасштабирования кластера. | Роль участника для группы ресурсов узла | Поддерживается
| kubelet | Имя кластера AKS — ажентпул | Проверка подлинности с помощью реестра контейнеров Azure (запись контроля доступа) | НД (для kubernetes v 1,15 доллара +) | Сейчас не поддерживается
| Дополнительно | азуренпм | Удостоверение не требуется | Н/Д | Нет
| Дополнительно | Мониторинг сети Азурекни | Удостоверение не требуется | Н/Д | Нет
| Дополнительно | Политика Azure (привратник) | Удостоверение не требуется | Н/Д | Нет
| Дополнительно | Политика Azure | Удостоверение не требуется | Н/Д | Нет
| Дополнительно | Calico | Удостоверение не требуется | Н/Д | Нет
| Дополнительно | Панель мониторинга | Удостоверение не требуется | Н/Д | Нет
| Дополнительно | хттпаппликатионраутинг | Управляет необходимыми сетевыми ресурсами | Роль читателя для группы ресурсов Node, роль участника для зоны DNS | Нет
| Дополнительно | Шлюз приложений входящего трафика | Управляет необходимыми сетевыми ресурсами| Роль участника для группы ресурсов узла | Нет
| Дополнительно | omsagent | Используется для отправки метрик AKS в Azure Monitor | Роль издателя метрик мониторинга | Нет
| Дополнительно | Virtual-Node (АЦиконнектор) | Управляет необходимыми сетевыми ресурсами для службы "экземпляры контейнеров Azure" (ACI) | Роль участника для группы ресурсов узла | Нет
| Проект OSS | AAD-Pod-Identity | Позволяет приложениям безопасно получать доступ к облачным ресурсам с помощью Azure Active Directory (AAD) | Н/Д | Действия по предоставлению разрешения в https://github.com/Azure/aad-pod-identity#role-assignment .

## <a name="create-an-aks-cluster-with-managed-identities"></a>Создание кластера AKS с управляемыми удостоверениями

Теперь можно создать кластер AKS с управляемыми удостоверениями с помощью следующих команд интерфейса командной строки.

Сначала создайте группу ресурсов Azure:

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Затем создайте кластер AKS:

```azurecli-interactive
az aks create -g myResourceGroup -n myManagedCluster --enable-managed-identity
```

Успешное создание кластера с помощью управляемых удостоверений содержит следующие сведения о профиле субъекта-службы:

```output
"servicePrincipalProfile": {
    "clientId": "msi"
  }
```

Используйте следующую команду, чтобы запросить ObjectID управляемого удостоверения плоскости управления:

```azurecli-interactive
az aks show -g myResourceGroup -n myManagedCluster --query "identity"
```

Результат должен выглядеть следующим образом.

```output
{
  "principalId": "<object_id>",   
  "tenantId": "<tenant_id>",      
  "type": "SystemAssigned"                                 
}
```

После создания кластера можно развернуть рабочие нагрузки приложений в новом кластере и взаимодействовать с ним так же, как и с кластерами AKS на основе субъектов-служб.

> [!NOTE]
> Для создания и использования собственной виртуальной сети, статического IP-адреса или подключенного диска Azure, где ресурсы выходят за пределы группы ресурсов рабочего узла, используйте PrincipalID управляемого удостоверения, назначенного системой кластера, для выполнения назначения ролей. Дополнительные сведения о назначении ролей см. в статье [Делегирование доступа к другим ресурсам Azure](kubernetes-service-principal.md#delegate-access-to-other-azure-resources).
>
> Предоставление разрешений на управляемое удостоверение кластера, используемое поставщиком облачных служб Azure, может занять 60 минут для заполнения.

Наконец, получите учетные данные для доступа к кластеру:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```
## <a name="update-an-aks-cluster-to-managed-identities-preview"></a>Обновление кластера AKS до управляемых удостоверений (Предварительная версия)

Теперь вы можете обновить кластер AKS, который сейчас работает с субъектами-службами, для работы с управляемыми удостоверениями с помощью следующих команд интерфейса командной строки.

Сначала зарегистрируйте флаг функции для назначенного системой удостоверения:

```azurecli-interactive
az feature register --namespace Microsoft.ContainerService -n MigrateToMSIClusterPreview
```

Обновите назначенное системой удостоверение:

```azurecli-interactive
az aks update -g <RGName> -n <AKSName> --enable-managed-identity
```

Зарегистрируйте флаг функции для назначенного пользователю удостоверения:

```azurecli-interactive
az feature register --namespace Microsoft.ContainerService -n UserAssignedIdentityPreview
```

Обновите назначенное пользователем удостоверение:

```azurecli-interactive
az aks update -g <RGName> -n <AKSName> --enable-managed-identity --assign-identity <UserAssignedIdentityResourceID> 
```
> [!NOTE]
> После того как назначенные системой или назначенные пользователем удостоверения были обновлены до управляемого удостоверения, выполните `az aks nodepool upgrade --node-image-only` на узлах, чтобы завершить обновление управляемого удостоверения.

## <a name="bring-your-own-control-plane-mi"></a>Приведите собственную плоскость управления
Настраиваемое удостоверение плоскости управления позволяет предоставлять доступ к существующему удостоверению до создания кластера. Эта функция позволяет выполнять такие сценарии, как использование настраиваемой виртуальной сети или Аутбаундтипе UDR с предварительно созданным управляемым удостоверением.

Необходимо установить Azure CLI версии 2.15.1 или более поздней.

### <a name="limitations"></a>Ограничения
* В настоящее время Azure для государственных организаций не поддерживается.
* В настоящее время в Azure Китая 21Vianet не поддерживается.

Если у вас еще нет управляемого удостоверения, вы можете создать его, например, с помощью команды [AZ Identity CLI][az-identity-create].

```azurecli-interactive
az identity create --name myIdentity --resource-group myResourceGroup
```
Результат должен выглядеть следующим образом.

```output
{                                                                                                                                                                                 
  "clientId": "<client-id>",
  "clientSecretUrl": "<clientSecretUrl>",
  "id": "/subscriptions/<subscriptionid>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity", 
  "location": "westus2",
  "name": "myIdentity",
  "principalId": "<principalId>",
  "resourceGroup": "myResourceGroup",                       
  "tags": {},
  "tenantId": "<tenant-id>>",
  "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

Если управляемое удостоверение является частью подписки, можно выполнить запрос с помощью [команды AZ Identity CLI][az-identity-list] .  

```azurecli-interactive
az identity list --query "[].{Name:name, Id:id, Location:location}" -o table
```

Теперь для создания кластера с существующим удостоверением можно использовать следующую команду:

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myManagedCluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 \
    --enable-managed-identity \
    --assign-identity <identity-id> \
```

Успешное создание кластера с помощью управляемых удостоверений содержит следующие сведения о профиле Userassignedidentities класса:

```output
 "identity": {
   "principalId": null,
   "tenantId": null,
   "type": "UserAssigned",
   "userAssignedIdentities": {
     "/subscriptions/<subscriptionid>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity": {
       "clientId": "<client-id>",
       "principalId": "<principal-id>"
     }
   }
 },
```

## <a name="next-steps"></a>Дальнейшие действия
* Используйте [шаблоны Azure Resource Manager (ARM) ][aks-arm-template] для создания кластеров с поддержкой управляемого удостоверения.

<!-- LINKS - external -->
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters
[az-identity-create]: /cli/azure/identity#az-identity-create
[az-identity-list]: /cli/azure/identity#az-identity-list
