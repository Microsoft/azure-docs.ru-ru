---
title: Учебник по созданию кластера Azure Red Hat OpenShift 4
description: Узнайте, как создать кластер Microsoft Azure Red Hat OpenShift с помощью Azure CLI.
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: azure-redhat-openshift
ms.date: 10/26/2020
ms.openlocfilehash: dda4fc6a80bbe07977f8d2a5ffcbea895a4e1fe6
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771845"
---
# <a name="tutorial-create-an-azure-red-hat-openshift-4-cluster"></a>Руководство по созданию кластера Azure Red Hat OpenShift 4

В этом учебнике (часть 1 из 3) вы подготовите среду для создания кластера Azure Red Hat OpenShift с OpenShift 4 и создадите кластер. Вы узнаете, как:
> [!div class="checklist"]
> * настроить необходимые компоненты; 
> * создать необходимые виртуальную сеть и подсеть;
> * Развертывание кластера

## <a name="before-you-begin"></a>Перед началом

Если решено установить и использовать CLI локально, то для работы с этим учебником понадобится Azure CLI 2.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

Для создания и запуска кластера OpenShift в Azure Red Hat OpenShift требуется не менее 40 ядер. Стандартная квота ресурсов Azure для новой подписки Azure не соответствует этому требованию. Чтобы запросить увеличение ограничений ресурсов, см. статью об [ увеличении ограничений для различных серий виртуальных машин](../azure-portal/supportability/per-vm-quota-requests.md).

Секрет для извлечения ARO не изменяет цену лицензии RH OpenShift для ARO.

### <a name="verify-your-permissions"></a>Проверка разрешений

При работе с этим учебником будет создана группа ресурсов, в которой будет содержаться виртуальная сеть для кластера. Необходимо иметь разрешения "Участник" и "Администратор доступа пользователей" или разрешения "Владелец" либо непосредственно для виртуальной сети, либо для группы ресурсов или подписки, содержащей ее.

Кроме того, для создания приложения и субъекта-службы от вашего имени для кластера потребуются разрешения Azure Active Directory.

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
    
1. Зарегистрируйте поставщик ресурсов `Microsoft.Authorization`.

    ```azurecli-interactive
    az provider register -n Microsoft.Authorization --wait
    ```

### <a name="get-a-red-hat-pull-secret-optional"></a>Получение секрета для извлечения Red Hat (необязательно)

Секрет для извлечения Red Hat позволяет вашему кластеру получать доступ к реестрам контейнера Red Hat и дополнительному содержимому. Этот шаг необязателен, но мы рекомендуем его выполнить.

1. [Перейдите на портал диспетчера кластеров Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) и выполните вход.

   Вам потребуется войти в учетную запись Red Hat или создать учетную запись Red Hat с помощью рабочего адреса электронной почты и принять условия.

1. Щелкните **Download pull secret** (Скачать секрет для извлечения) и скачайте секрет для извлечения, который будет использоваться с кластером ARO.

    Сохраните файл `pull-secret.txt` в безопасном месте. Этот файл будет использоваться при каждом создании кластера, если вам потребуется создать кластер, включающий примеры или операторы для партнеров Red Hat или сертифицированных партнеров.

    При выполнении команды `az aro create` вы можете ссылаться на секрет для извлечения, используя параметр `--pull-secret @pull-secret.txt`. Выполните `az aro create` из каталога, в котором был сохранен файл `pull-secret.txt`. В противном случае измените `@pull-secret.txt` на `@/path/to/my/pull-secret.txt`.

    Если вы копируете свой секрет для извлечения или ссылаетесь на него в других скриптах, он должен быть отформатирован в виде допустимой строки JSON.

### <a name="prepare-a-custom-domain-for-your-cluster-optional"></a>Подготовка личного домена для кластера (дополнительно)

При выполнении команды `az aro create` можно указать личный домен для кластера с использованием параметра `--domain foo.example.com`.

Если вы предоставляете личный домен для кластера, учитывайте следующие моменты:

* После создания кластера необходимо создать две записи DNS типа A на DNS-сервере для указанного значения`--domain`:
    * запись **api**, указывающую на IP-адрес сервера;
    * запись **\*.apps**, указывающую на IP-адрес входящего трафика.
    * Извлеките эти значения, выполнив следующую команду после создания кластера: `az aro show -n -g --query '{api:apiserverProfile.ip, ingress:ingressProfiles[0].ip}'`.

* Консоль OpenShift будет доступна по URL-адресу, такому как `https://console-openshift-console.apps.example.com`, вместо встроенного домена `https://console-openshift-console.apps.<random>.<location>.aroapp.io`.

* По умолчанию OpenShift использует самозаверяющие сертификаты для всех маршрутов, созданных в личных доменах `*.apps.example.com`.  Если вы решили использовать пользовательскую службу DNS после подключения к кластеру, необходимо выполнить действия, описанные в документации по OpenShift, чтобы [настроить пользовательский ЦС для контроллера входящего трафика](https://docs.openshift.com/container-platform/4.6/security/certificates/replacing-default-ingress-certificate.html) и [пользовательский ЦС для сервера API](https://docs.openshift.com/container-platform/4.6/security/certificates/api-server.html).

### <a name="create-a-virtual-network-containing-two-empty-subnets"></a>Создание виртуальной сети, содержащей две пустые подсети

Далее вы создадите виртуальную сеть, содержащую две пустые подсети. Если вы хотите использовать существующую виртуальную сеть, которая соответствует вашим требованиям, этот шаг можно пропустить.

1. **Задайте указанные ниже переменные в среде оболочки, где будут выполняться команды `az`.**

   ```console
   LOCATION=eastus                 # the location of your cluster
   RESOURCEGROUP=aro-rg            # the name of the resource group where you want to create your cluster
   CLUSTER=cluster                 # the name of your cluster
   ```

2. **Создайте группу ресурсов.**

   Группа ресурсов Azure — это логическая группа, в которой развертываются и управляются ресурсы Azure. Во время создания группы ресурсов вам будет предложено указать расположение. В этом расположении сохраняются метаданные группы ресурсов, а также выполняется их работа в Azure, если во время создания ресурса не указан другой регион. Создайте группу ресурсов с помощью команды [az group create](/cli/azure/group#az_group_create).
    
   > [!NOTE] 
   > Платформа Azure Red Hat OpenShift доступна не во всех регионах, где поддерживается создание группы ресурсов Azure. Список регионов, где поддерживается Azure Red Hat OpenShift, см. в разделе [Available regions](https://azure.microsoft.com/en-gb/global-infrastructure/services/?products=openshift) (Доступные регионы).

   ```azurecli-interactive
   az group create \
     --name $RESOURCEGROUP \
     --location $LOCATION
   ```

   В следующем примере выходных данных показано, что группа ресурсов успешно создана:

   ```json
   {
     "id": "/subscriptions/<guid>/resourceGroups/aro-rg",
     "location": "eastus",
     "name": "aro-rg",
     "properties": {
       "provisioningState": "Succeeded"
     },
     "type": "Microsoft.Resources/resourceGroups"
   }
   ```

2. **Создайте виртуальную сеть.**

   Для кластеров Azure Red Hat OpenShift с OpenShift 4 требуется виртуальная сеть с двумя пустыми подсетями для главного и рабочего узлов. Для этого вы можете создать новую виртуальную сеть или использовать существующую.

   Создайте виртуальную сеть в той же группе ресурсов, которая была создана ранее:

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
       "dhcpOptions": {
         "dnsServers": []
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

5. **[Отключите политики частной конечной точки подсети](../private-link/disable-private-link-service-network-policy.md) в главной подсети.** Это необходимо службе для подключения к кластеру и управления им.

   ```azurecli-interactive
   az network vnet subnet update \
     --name master-subnet \
     --resource-group $RESOURCEGROUP \
     --vnet-name aro-vnet \
     --disable-private-link-service-network-policies true
   ```

## <a name="create-the-cluster"></a>Создайте кластер.

Чтобы создать кластер, выполните команду ниже. Если вы решили использовать один из указанных ниже параметров, измените команду соответствующим образом:
* При необходимости вы можете [передать секрет для извлечения Red Hat](#get-a-red-hat-pull-secret-optional), который позволяет кластеру получать доступ к реестрам контейнеров Red Hat, а также к дополнительному содержимому. Добавьте в команду аргумент `--pull-secret @pull-secret.txt`.
* При необходимости можно [использовать личный домен](#prepare-a-custom-domain-for-your-cluster-optional). Добавьте в команду аргумент `--domain foo.example.com`, заменив `foo.example.com` собственным личным доменом.

> [!NOTE]
> Если вы добавляете в команду необязательные аргументы, обязательно закройте аргумент в предыдущей строке команды конечной обратной косой чертой.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet
```

После выполнения команды `az aro create` создание кластера обычно занимает около 35 минут.

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:
> [!div class="checklist"]
> * настраивать необходимые компоненты, а также создавать требуемую виртуальную сеть и подсети;
> * Развертывание кластера

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Подключение к кластеру Azure Red Hat OpenShift](tutorial-connect-cluster.md)
