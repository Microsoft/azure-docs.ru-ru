---
title: Создание частного кластера Azure Red Hat OpenShift 4
description: Узнайте, как создать частный кластер Azure Red Hat OpenShift с OpenShift 4
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 03/12/2020
author: sakthi-vetrivel
ms.author: suvetriv
keywords: aro, openshift, az aro, red hat, cli
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 5a6eafa6b2380ee61c0de2e329a1751ec7d59c1f
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102216077"
---
# <a name="create-an-azure-red-hat-openshift-4-private-cluster"></a>Создание частного кластера Azure Red Hat OpenShift 4

В этой статье объясняется, как подготовить среду для создания частных кластеров Azure Red Hat OpenShift с OpenShift 4. Вы узнаете, как:

> [!div class="checklist"]
> * настраивать необходимые компоненты и создавать необходимую виртуальную сеть и подсети;
> * Развертывание кластера с частной конечной точкой сервера API и закрытым контроллером входящего трафика

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="before-you-begin"></a>Перед началом

### <a name="register-the-resource-providers"></a>Регистрация поставщиков ресурсов

1. Если у вас несколько подписок Azure, выберите нужный идентификатор подписки:

    ```azurecli-interactive
    az account set --subscription <SUBSCRIPTION ID>
    ```

1. Зарегистрируйте поставщик ресурсов `Microsoft.RedHatOpenShift`.

    ```azurecli-interactive
    az provider register -n Microsoft.RedHatOpenShift --wait
    ```

1. Зарегистрируйте поставщик ресурсов `Microsoft.Compute`.

    ```azurecli-interactive
    az provider register -n Microsoft.Compute --wait
    ```

1. Зарегистрируйте поставщик ресурсов `Microsoft.Storage`.

    ```azurecli-interactive
    az provider register -n Microsoft.Storage --wait
    ```

### <a name="get-a-red-hat-pull-secret-optional"></a>Получение секрета для извлечения Red Hat (необязательно)

Секрет для извлечения Red Hat позволяет вашему кластеру получать доступ к реестрам контейнера Red Hat и дополнительному содержимому. Этот шаг необязателен, но мы рекомендуем его выполнить.

1. **[Перейдите на портал диспетчера кластеров Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) и выполните вход.**

   Вам потребуется войти в учетную запись Red Hat или создать учетную запись Red Hat с помощью рабочего адреса электронной почты и принять условия.

2. **Щелкните "Download pull secret" (Скачать секрет для извлечения).**

Сохраненный файл `pull-secret.txt` должен находиться в безопасном месте — он будет использоваться при каждом создании кластера.

При выполнении команды `az aro create` вы можете ссылаться на секрет для извлечения, используя параметр `--pull-secret @pull-secret.txt`. Выполните `az aro create` из каталога, в котором был сохранен файл `pull-secret.txt`. В противном случае измените `@pull-secret.txt` на `@<path-to-my-pull-secret-file`.

Если вы копируете свой секрет для извлечения или ссылаетесь на него в других скриптах, он должен быть отформатирован в виде допустимой строки JSON.

### <a name="create-a-virtual-network-containing-two-empty-subnets"></a>Создание виртуальной сети, содержащей две пустые подсети

Далее вы создадите виртуальную сеть, содержащую две пустые подсети.

1. **Задайте следующие переменные.**

   ```console
   LOCATION=eastus                 # the location of your cluster
   RESOURCEGROUP="v4-$LOCATION"    # the name of the resource group where you want to create your cluster
   CLUSTER=aro-cluster             # the name of your cluster
   ```

1. **Создайте группу ресурсов**.

    Группа ресурсов Azure — это логическая группа, в которой развертываются и управляются ресурсы Azure. Во время создания группы ресурсов вам будет предложено указать расположение. В этом расположении сохраняются метаданные группы ресурсов, а также выполняется их работа в Azure, если во время создания ресурса вы не указали другой регион. Создайте группу ресурсов с помощью команды [az group create][az-group-create].

    ```azurecli-interactive
    az group create --name $RESOURCEGROUP --location $LOCATION
    ```

    В следующем примере выходных данных показано, что группа ресурсов успешно создана:

    ```json
    {
    "id": "/subscriptions/<guid>/resourceGroups/aro-rg",
    "location": "eastus",
    "managedBy": null,
    "name": "aro-rg",
    "properties": {
        "provisioningState": "Succeeded"
    },
    "tags": null
    }
    ```

2. **Создайте виртуальную сеть.**

    Для кластеров Azure Red Hat OpenShift с OpenShift 4 требуется виртуальная сеть с двумя пустыми подсетями для главного и рабочего узлов.

    Создайте виртуальную сеть в той же группе ресурсов, которая была создана ранее.

    ```azurecli-interactive
    az network vnet create \
    --resource-group $RESOURCEGROUP \
    --name aro-vnet \
    --address-prefixes 10.0.0.0/22
    ```

    В следующем примере выходных данных показано, что виртуальная сеть успешно создана:

    ```json
    {
    "newVNet": {
        "addressSpace": {
        "addressPrefixes": [
            "10.0.0.0/22"
        ]
        },
        "id": "/subscriptions/<guid>/resourceGroups/aro-rg/providers/Microsoft.Network/virtualNetworks/aro-vnet",
        "location": "eastus",
        "name": "aro-vnet",
        "provisioningState": "Succeeded",
        "resourceGroup": "aro-rg",
        "type": "Microsoft.Network/virtualNetworks"
    }
    }
    ```

3. **Добавьте пустую подсеть для главных узлов.**

    ```azurecli-interactive
    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name master-subnet \
    --address-prefixes 10.0.0.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

4. **Добавьте пустую подсеть для рабочих узлов.**

    ```azurecli-interactive
    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name worker-subnet \
    --address-prefixes 10.0.2.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
    ```

5. **[Отключите политики частной конечной точки подсети](../private-link/disable-private-link-service-network-policy.md) в главной подсети.** Это необходимо для подключения к кластеру и управления им.

    ```azurecli-interactive
    az network vnet subnet update \
    --name master-subnet \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --disable-private-link-service-network-policies true
    ```

## <a name="create-the-cluster"></a>Создайте кластер.

Чтобы создать кластер, выполните команду ниже. При необходимости вы можете [передать секрет для извлечения Red Hat](#get-a-red-hat-pull-secret-optional), который позволяет кластеру получать доступ к реестрам контейнеров Red Hat, а также к дополнительному содержимому.

>[!NOTE]
> При копировании и вставке команд, а также использовании одного из необязательных параметров убедитесь, что исходные хэштеги и комментарии в конце удалены. Кроме того, добавьте к аргументу в предыдущей строке команды обратную косую черту в конце.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  --apiserver-visibility Private \
  --ingress-visibility Private
  # --domain foo.example.com # [OPTIONAL] custom domain
  # --pull-secret @pull-secret.txt # [OPTIONAL]
```

После выполнения команды `az aro create` создание кластера обычно занимает около 35 минут.

>[!IMPORTANT]
> Если вы решили указать личный домен, например **foo.example.com**, консоль OpenShift будет доступна по URL-адресу, например `https://console-openshift-console.apps.foo.example.com`, вместо встроенного адреса `https://console-openshift-console.apps.<random>.<location>.aroapp.io` домена.
>
> По умолчанию OpenShift использует самозаверяющие сертификаты для всех маршрутов, созданных в `*.apps.<random>.<location>.aroapp.io`.  Если вы решили использовать пользовательскую службу DNS, то после подключения к кластеру нужно выполнить действия, описанные в документации по OpenShift, чтобы [настроить пользовательский ЦС для контроллера входящего трафика](https://docs.openshift.com/container-platform/4.3/authentication/certificates/replacing-default-ingress-certificate.html) и [пользовательский ЦС для сервера API](https://docs.openshift.com/container-platform/4.3/authentication/certificates/api-server.html).

## <a name="connect-to-the-private-cluster"></a>Подключение к частному кластеру

Вы можете войти в кластер как пользователь `kubeadmin`.  Выполните следующую команду, чтобы найти пароль для пользователя `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

В следующем примере вывода показано, что пароль будет находиться в `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

URL-адрес консоли кластера можно найти, выполнив следующую команду, которая будет выглядеть вот так: `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`

```azurecli-interactive
 az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
```

>[!IMPORTANT]
> Чтобы подключиться к частному кластеру Azure Red Hat OpenShift, необходимо выполнить следующий шаг с узла, который находится либо в созданной виртуальной сети, либо в виртуальной сети с [пиринговым](../virtual-network/virtual-network-peering-overview.md) подключением к виртуальной сети, в которой был развернут кластер.

Перейдите по URL-адресу консоли в браузере и войдите с помощью учетных данных `kubeadmin`.

![Снимок экрана, на котором показан экран входа в Azure Red Hat OpenShift.](media/aro4-login.png)

## <a name="install-the-openshift-cli"></a>Установка интерфейса командной строки OpenShift

После входа в веб-консоль OpenShift щелкните **?** в правом верхнем углу, а затем выберите **Программы командной строки**. Скачайте выпуск, подходящий вашему компьютеру.

![На рисунке показан экран входа Azure Red Hat OpenShift](media/aro4-download-cli.png)

Вы также можете скачать последнюю версию интерфейса командной строки, подходящую для компьютера, по адресу <https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/>.

## <a name="connect-using-the-openshift-cli"></a>Подключение с помощью интерфейса командной строки OpenShift

Получите адрес сервера API.

```azurecli-interactive
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```

>[!IMPORTANT]
> Чтобы подключиться к частному кластеру Azure Red Hat OpenShift, необходимо выполнить следующий шаг с узла, который находится либо в созданной виртуальной сети, либо в виртуальной сети с [пиринговым](../virtual-network/virtual-network-peering-overview.md) подключением к виртуальной сети, в которой был развернут кластер.

Войдите на сервер API кластера OpenShift с помощью следующей команды. Замените **\<kubeadmin password>** паролем, который вы только что получили.

```azurecli-interactive
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

## <a name="next-steps"></a>Дальнейшие действия

Из этой статьи вы узнали, как развернуть кластер Azure Red Hat OpenShift с OpenShift 4. Вы ознакомились с выполнением следующих задач:

> [!div class="checklist"]
> * настраивать необходимые компоненты и создавать необходимую виртуальную сеть и подсети;
> * Развертывание кластера
> * Подключение к кластеру от имени пользователя `kubeadmin`

Перейдите к следующей статье, чтобы узнать, как настроить проверку подлинности в кластере с помощью Azure Active Directory.


* [Настройка проверки подлинности в Azure Active Directory с помощью командной строки](configure-azure-ad-cli.md)


* [Настройка проверки подлинности в Azure Active Directory с портала Azure и веб-консоли OpenShift](configure-azure-ad-cli.md)
