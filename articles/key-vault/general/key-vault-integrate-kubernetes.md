---
title: Интеграция Azure Key Vault с Kubernetes
description: В рамках этого учебника вы получите доступ к секретам Azure Key Vault с помощью драйвера интерфейса хранилища контейнера (CSI) хранилища секретов, а затем подключитесь к объектам pod Kubernetes.
author: msmbaldwin
ms.author: mbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 09/25/2020
ms.openlocfilehash: 6cf76e980fab4e5be3f8c2c6d72baff05ab03815
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/01/2021
ms.locfileid: "106108393"
---
# <a name="tutorial-configure-and-run-the-azure-key-vault-provider-for-the-secrets-store-csi-driver-on-kubernetes"></a>Руководство по настройке и запуску поставщика Azure Key Vault для драйвера CSI хранилища секретов в Kubernetes

> [!IMPORTANT]
> Драйвер CSI хранилища секретов — это проект с открытым исходным кодом, и он не поддерживается службой технической поддержки Azure. Все отзывы и сведения об ошибках, связанные с интеграцией Key Vault с драйвером CSI, отправляйте по [этой](https://github.com/kubernetes-sigs/secrets-store-csi-driver) ссылке в GitHub. Это средство предоставляется пользователям для самостоятельной установки на кластерах и сбора отзывов представителей сообщества.

Для работы с этим руководством вы получите доступ к секретам хранилища ключей Azure с помощью драйвера Container Storage Interface (CSI) хранилища секретов, а затем подключите секреты к группам pod Kubernetes в качестве тома.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * использование управляемых удостоверений;
> * Развертывание кластера Службы Azure Kubernetes (AKS) с помощью Azure CLI.
> * Установите Helm, драйвер CSI хранилища секретов и поставщик Azure Key Vault для драйвера CSI.
> * Создание хранилища ключей Azure и задание секретов.
> * Создание собственного объекта SecretProviderClass.
> * Развертывание pod с подключенными секретами из хранилища ключей.

## <a name="prerequisites"></a>Предварительные требования

* Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

* Перед началом работы с этим руководством установите [Azure CLI](/cli/azure/install-azure-cli-windows).

В этом руководстве предполагается, что Служба Azure Kubernetes выполняется на узлах Linux.

## <a name="use-managed-identities"></a>Использование управляемых удостоверений

На этой схеме показан поток интеграции AKS с Key Vault для управляемого удостоверения:

![Схема с демонстрацией потока интеграции AKS с Key Vault для управляемого удостоверения](../media/aks-key-vault-integration-flow.png)

## <a name="deploy-an-azure-kubernetes-service-aks-cluster-by-using-the-azure-cli"></a>Развертывание кластера Службы Azure Kubernetes (AKS) с помощью Azure CLI

Вам не нужно использовать Azure Cloud Shell. Командной строки (терминала) с установленным Azure CLI будет достаточно. 

Выполните действия в разделах "Создание группы ресурсов", "Создание кластера AKS" и "Подключение к кластеру" из [краткого руководства по развертыванию кластера Службы Azure Kubernetes с помощью Azure CLI](../../aks/kubernetes-walkthrough.md). 

> [!NOTE] 
> Если вы планируете использовать удостоверение pod, рекомендуемым сетевым подключаемым модулем является `azure`. Дополнительные сведения см. в [этом документе](https://azure.github.io/aad-pod-identity/docs/configure/aad_pod_identity_on_kubenet/). Создайте кластер Kubernetes, как показано в следующей команде:
>
> ```azurecli
> az aks create -n contosoAKSCluster -g contosoResourceGroup --kubernetes-version 1.16.9 --node-count 1 --enable-managed-identity
> ```

1. [Задайте переменную среды PATH](https://www.java.com/en/download/help/path.xml) для скачанного файла *kubectl.exe*.
2. Проверьте версию Kubernetes с помощью приведенной ниже команды, которая выводит версию клиента и сервера. Версия клиента — это установленный файл *kubectl.exe*, а версия сервера — это Служба Azure Kubernetes (AKS), в которой выполняется кластер.
    ```azurecli
    kubectl version
    ```
3. Убедитесь, что используется версия Kubernetes 1.16.0 или более поздняя. Убедитесь, что для кластеров Windows используется версия Kubernetes 1.18.0 или более поздняя. Приведенная ниже команда обновит кластер Kubernetes и пул узлов. Выполнение команды может занять несколько минут. В этом примере группа ресурсов — *contosoResourceGroup*, а кластер Kubernetes — *contosoAKSCluster*.
    ```azurecli
    az aks upgrade --kubernetes-version 1.16.9 --name contosoAKSCluster --resource-group contosoResourceGroup
    ```
4. Отобразите метаданные созданного кластера AKS с помощью приведенной ниже команды. Скопируйте значения **principalId**, **clientId**, **subscriptionId** и **nodeResourceGroup** для последующего использования. Если кластер AKS не был создан с включенными управляемыми удостоверениями, **principalId** и **clientId** будут иметь значение NULL. 

    ```azurecli
    az aks show --name contosoAKSCluster --resource-group contosoResourceGroup
    ```

    В выходных данных выделены оба параметра.
    
    ![Снимок экрана Azure CLI с выделенными значениями principalId и clientId](../media/kubernetes-key-vault-2.png) ![Снимок экрана Azure CLI с выделенными значениями subscriptionId и nodeResourceGroup](../media/kubernetes-key-vault-3.png)
    
## <a name="install-helm-and-the-secrets-store-csi-driver"></a>Установка драйвера CSI Helm и хранилища секретов.
> [!NOTE]
> Приведенная ниже процедура установки применима только к AKS в Linux. Дополнительные сведения об установке драйвера CSI хранилища секретов см. в статье [Поставщик Azure Key Vault для драйвера CSI хранилища секретов](https://azure.github.io/secrets-store-csi-driver-provider-azure/getting-started/installation/). 

Чтобы установить драйвер CSI хранилища секретов и поставщик Azure Key Vault, сначала нужно установить [Helm](https://helm.sh/docs/intro/install/).

Интерфейс драйвера [CSI хранилища секретов](https://azure.github.io/secrets-store-csi-driver-provider-azure/) позволяет получать секреты, хранящиеся в экземпляре Azure Key Vault, и использовать интерфейс драйвера для подключения содержимого этих секретов к объектам pod в Kubernetes.

1. Убедитесь, что для Helm установлена версия 3 или более поздняя версия.
    ```azurecli
    helm version
    ```
1. Установите драйвер CSI хранилища секретов и поставщик Azure Key Vault для драйвера:
    ```azurecli
    helm repo add csi-secrets-store-provider-azure https://raw.githubusercontent.com/Azure/secrets-store-csi-driver-provider-azure/master/charts

    helm install csi-secrets-store-provider-azure/csi-secrets-store-provider-azure --generate-name
    ```
> [!NOTE] 
> Если вы планируете использовать драйвер CSI хранилища секретов и поставщик Azure Key Vault на узлах Windows, включите этот драйвер и поставщик на узлах Windows, используя [значения конфигурации helm](https://github.com/Azure/secrets-store-csi-driver-provider-azure/tree/master/charts/csi-secrets-store-provider-azure#configuration).

## <a name="create-an-azure-key-vault-and-set-your-secrets"></a>Создание Azure Key Vault и задание секретов.

Чтобы создать собственное хранилище ключей и задать секреты, выполните инструкции из [краткого руководства по настройке и получении секрета из Azure Key Vault с помощью Azure CLI](../secrets/quick-create-cli.md).

> [!NOTE] 
> Вам не нужно использовать Azure Cloud Shell или создавать новую группу ресурсов. Вы можете использовать группу ресурсов, созданную ранее для кластера Kubernetes.

## <a name="create-your-own-secretproviderclass-object"></a>Создание собственного объекта SecretProviderClass

Используйте этот [шаблон](https://github.com/Azure/secrets-store-csi-driver-provider-azure/blob/master/examples/service-principal/v1alpha1_secretproviderclass_service_principal.yaml), чтобы создать собственный пользовательский объект SecretProviderClass с заданными поставщиком параметрами для драйвера CSI хранилища секретов. Этот объект предоставит доступ к удостоверениям хранилища ключей.

В файле YAML SecretProviderClass введите отсутствующие параметры. Требуются следующие параметры.

* **KeyVaultName:** имя хранилища ключей.
* **objects**: список всего содержимого секрета, которое вы хотите подключить.
    * **objectName:** имя содержимого секрета.
    * **objectType:** тип объекта (секрет, ключ, сертификат).
* **tenantID:** идентификатор клиента или идентификатор каталога хранилища ключей.

Документация по всем обязательным полям и поддерживаемым конфигурациям доступна здесь: [Ссылка](https://azure.github.io/secrets-store-csi-driver-provider-azure/getting-started/usage/#create-your-own-secretproviderclass-object)

Обновленный шаблон показан в следующем коде. Скачайте его как файл YAML и заполните обязательные поля. В этом примере хранилище ключей — **contosoKeyVault5**. Оно содержит два секрета: **secret1** и **secret2**.

> [!NOTE] 
> Если вы используете удостоверения pod, укажите для параметра **usePodIdentity** значение *true* и задайте **userAssignedIdentityID** в виде пары кавычек ( **""** ). Если вы используете управляемые удостоверения, укажите для параметра **useVMManagedIdentity** значение *true* и задайте **userAssignedIdentityID** как clientID, назначаемого пользователем удостоверения.

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1alpha1
kind: SecretProviderClass
metadata:
  name: azure-kvname
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"                                                 # [OPTIONAL] if not provided, will default to "false". Set to "true" if using pod identities.
    useVMManagedIdentity: "false"                                           # [OPTIONAL] if not provided, will default to "false". Set to "true" if using managed identities.
    userAssignedIdentityID: "<clientID of user-assigned managed identity"   # [OPTIONAL] If you're using managed identities, use the client id to specify which user-assigned managed identity to use. If the value is empty, it defaults to use the system-assigned identity on the VM
    keyvaultName: "contosoKeyVault5"                                        # [REQUIRED] the name of the key vault
                                                                            #     az keyvault show --name contosoKeyVault5
                                                                            #     the preceding command will display the key vault metadata, which includes the subscription ID, resource group name, key vault 
    cloudName: ""                                                           # [OPTIONAL] if not provided, Azure environment will default to AzurePublicCloud
    objects:  |
      array:
        - |
          objectName: secret1                                               # [REQUIRED] object name
                                                                            #     az keyvault secret list --vault-name "contosoKeyVault5"
                                                                            #     the above command will display a list of secret names from your key vault
          objectType: secret                                                # [REQUIRED] object types: secret, key, or cert
          objectVersion: ""                                                 # [OPTIONAL] object versions, default to latest if empty
        - |
          objectName: secret2
          objectType: secret
          objectVersion: ""
    tenantId: "tenantID"                                                    # [REQUIRED] the tenant ID of the key vault
```
Ниже приведен вывод консоли для **az keyvault show --name contosoKeyVault5** с соответствующими выделенными метаданными.

![Снимок экрана с выходными данными консоли для az keyvault show --name contosoKeyVault5](../media/kubernetes-key-vault-4.png)

## <a name="install-azure-active-directory-azure-ad-pod-identity"></a>Установка удостоверения pod для Azure Active Directory (Azure AD)

1. Назначьте определенные роли кластеру AKS, который вы создали. 

    Документация по всем необходимым назначениям ролей с использованием удостоверения pod для Azure Active Directory (Azure AD) доступна здесь: [Ссылка](https://azure.github.io/aad-pod-identity/docs/getting-started/role-assignment/)

    ```azurecli
    VAULT_RESOURCE_GROUP=contosoResourceGroup
    NODE_RESOURCE_GROUP=contosoResourceGroup
    
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$VAULT_RESOURCE_GROUP
    
    az role assignment create --role "Managed Identity Operator" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$NODE_RESOURCE_GROUP
    
    az role assignment create --role "Virtual Machine Contributor" --assignee $clientId --scope /subscriptions/<SUBID>/resourcegroups/$NODE_RESOURCE_GROUP
    ```

2. Установите удостоверение Azure Active Directory (Azure AD) в AKS.

    > [!NOTE] 
    > Если вы используете кластер AKS с сетевым подключаемым модулем kubenet, ознакомьтесь с этим [документом](https://azure.github.io/aad-pod-identity/docs/configure/aad_pod_identity_on_kubenet/), чтобы узнать, как развернуть удостоверение pod в кластере.

    ```azurecli
    helm repo add aad-pod-identity https://raw.githubusercontent.com/Azure/aad-pod-identity/master/charts

    helm install pod-identity aad-pod-identity/aad-pod-identity
    ```

3. Создайте управляемое удостоверение, назначаемое пользователем. В выходных данных скопируйте **clientId** для последующего использования.
    ```azurecli
    az identity create -g $resourceGroupName -n $identityName
    ```

4. Предоставьте удостоверению разрешения на получение секретов из хранилища ключей. Используйте **clientId** из управляемого удостоверения, назначаемого пользователем.
    ```azurecli
    az keyvault set-policy -n contosoKeyVault5 --secret-permissions get --spn $clientId
    az keyvault set-policy -n contosoKeyVault5 --key-permissions get --spn $clientId
    az keyvault set-policy -n contosoKeyVault5 --certificate-permissions get --spn $clientId
    ```

## <a name="deploy-your-pod-with-mounted-secrets-from-your-key-vault"></a>Разверните pod с подключенными секретами из хранилища ключей.

Чтобы настроить объект SecretProviderClass, выполните следующую команду:
```azurecli
kubectl apply -f secretProviderClass.yaml
```

### <a name="use-managed-identities"></a>Использование управляемых удостоверений

Если вы используете удостоверения pod, создайте *AzureIdentity* в кластере, который ссылается на созданное ранее удостоверение. Затем создайте *AzureIdentityBinding*, который будет ссылаться на созданную AzureIdentity. Заполните параметры в следующем шаблоне и сохраните его как *podIdentityAndBinding.yaml*.  

```yml
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentity
metadata:
    name: azureIdentityName                 # The name of your Azure identity
spec:
    type: 0                                 # Set type: 0 for managed identity
    resourceID: /subscriptions/<SUBID>/resourcegroups/<RESOURCEGROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<AZUREIDENTITYNAME>
    clientID: "<managed identity clientID>"   # The clientId of the User-assigned managed identity that you created earlier
---
apiVersion: aadpodidentity.k8s.io/v1
kind: AzureIdentityBinding
metadata:
    name: azure-pod-identity-binding
spec:
    azureIdentity: "azureIdentityName"      # The name of your Azure identity
    selector: azure-pod-identity-binding-selector
```
    
Выполните следующую команду для настройки привязки:

```azurecli
kubectl apply -f podIdentityAndBinding.yaml
```

Затем выполняется развертывание pod. Ниже приведен код файла развертывания YAML, который будет использовать привязку удостоверения pod из последнего шага. Сохраните этот файл как *podBindingDeployment.yaml*.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-secrets-store-inline
  labels:
    aadpodidbinding: azure-pod-identity-binding-selector # The selector defined in AzureIdentityBinding in the previous step
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets-store"
          readOnly: true
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: azure-kvname
```

Выполните следующую команду, чтобы развернуть шаблон:

```azurecli
kubectl apply -f podBindingDeployment.yaml
```

### <a name="check-the-pod-status-and-secret-content"></a>Проверка состояния pod и секретного содержимого 

Для просмотра развернутых объектов pod выполните следующую команду:
```azurecli
kubectl get pods
```

Чтобы проверить состояние объекта pod, используйте следующую команду:
```azurecli
kubectl describe pod/nginx-secrets-store-inline
```

![Снимок экрана выходных данных Azure CLI с отображением состояния "Выполняется" для pod и всех событий как "Обычный" ](../media/kubernetes-key-vault-6.png)

В окне выходных данных развернутый объект pod должен находиться в состоянии *Выполняется*. В нижней части раздела **События** все типы событий отображаются как *Обычный*.

Убедившись, что объект pod работает, вы можете проверить, есть ли у него секреты из хранилища ключей.

Чтобы отобразить все секреты, содержащиеся в объекте pod, выполните следующую команду:
```azurecli
kubectl exec nginx-secrets-store-inline -- ls /mnt/secrets-store/
```

Чтобы отобразить содержимое определенного секрета, выполните следующую команду:
```azurecli
kubectl exec nginx-secrets-store-inline -- cat /mnt/secrets-store/secret1
```

Убедитесь, что отображается содержимое секрета.

## <a name="resources"></a>Ресурсы
[Об Azure Key Vault](overview.md)
[Руководство разработчика по Azure Key Vault](developers-guide.md)
[Драйвер CSI для секретов](https://secrets-store-csi-driver.sigs.k8s.io/introduction.html)


Чтобы обеспечить возможность восстановления хранилища ключей, см. следующие материалы:
> [!div class="nextstepaction"]
> [Включить обратимое удаление](./key-vault-recovery.md)
