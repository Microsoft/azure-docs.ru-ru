---
title: Создание кластера Linux Service Fabric в Azure
description: Узнайте, как развернуть кластер Service Fabric на платформе Linux в существующей виртуальной сети с помощью Azure CLI.
ms.topic: conceptual
ms.date: 02/14/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 77cc49c1b79e5c24e78a67a69493aa0b0059d565
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98791077"
---
# <a name="deploy-a-linux-service-fabric-cluster-into-an-azure-virtual-network"></a>Развертывание кластера Service Fabric на платформе Linux в виртуальной сети Azure

Из этой статьи вы узнаете как развернуть кластер Service Fabric на платформе Linux в [виртуальную сеть Azure](../virtual-network/virtual-networks-overview.md) с помощью Azure CLI и шаблона. После окончания этого учебника у вас будет кластер в облаке, в который можно разворачивать приложения. Создание кластера Windows с помощью PowerShell описывается в разделе [Развертывание безопасного кластера Service Fabric на платформе Windows в виртуальной сети Azure](service-fabric-tutorial-create-vnet-and-windows-cluster.md).

## <a name="prerequisites"></a>Предварительные требования

Перед началом работы

* Если у вас еще нет подписки Azure, создайте [бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Установка [Service Fabric CLI](service-fabric-cli.md)
* Установите [Azure CLI](/cli/azure/install-azure-cli).
* Чтобы узнать об основных понятиях кластеров, прочитайте статью [Общие сведения о кластерах Service Fabric в Azure](service-fabric-azure-clusters-overview.md).
* [Спланируйте и подготовьте](service-fabric-cluster-azure-deployment-preparation.md) развертывание рабочего кластера.

Ниже приведены процедуры для создания кластера Service Fabric с семью узлами. Чтобы рассчитать затраты, связанные с выполнением кластера Service Fabric в Azure, используйте [калькулятор цен Azure](https://azure.microsoft.com/pricing/calculator/).

## <a name="download-and-explore-the-template"></a>Скачивание и изучение шаблона

Скачайте следующие файлы шаблона Resource Manager:

Для Ubuntu 16,04 LTS:

* [AzureDeploy.jsна][template]
* [AzureDeploy.Parameters.jsна][parameters]

Для Ubuntu 18,04 LTS:

* [AzureDeploy.jsна][template2]
* [AzureDeploy.Parameters.jsна][parameters2]

Для Ubuntu 18,04 LTS разница между двумя шаблонами 
* для атрибута **вмимажеску** устанавливается значение "18,04-LTS".
* для каждого **typeHandlerVersion** узла устанавливается значение 1,1.
* Ресурс Microsoft. ServiceFabric/Clusters
   - для **apiVersion** задано значение "2019-03-01" или выше
   - Свойство **образ виртуальной машины** , для которого задано значение "Ubuntu18_04"

Этот шаблон позволяет развернуть безопасный кластер семи виртуальных машин и трех типов узлов в виртуальную сеть.  Другие примеры шаблонов можно найти на сайте [GitHub](https://github.com/Azure-Samples/service-fabric-cluster-templates). [AzureDeploy.jsдля][template] развертывает числовые ресурсы, включая следующие.

### <a name="service-fabric-cluster"></a>Кластер Service Fabric

В ресурсе **Microsoft.ServiceFabric/clusters** развернут кластер Linux со следующими характеристиками:

* три типа узлов;
* пять узлов на первичном типе узла (можно настроить в параметрах шаблона), по одном узлу на каждый тип узла;
* ОС: (Ubuntu 16,04 LTS/Ubuntu 18,04 LTS) (можно настроить в параметрах шаблона)
* защищенный сертификат (можно настроить в параметрах шаблона);
* [Служба DNS](service-fabric-dnsservice.md) включена
* [Уровень устойчивости](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) бронзовой (настраиваемый в параметрах шаблона)
* [Уровень надежности](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) серебристого (можно настроить в параметрах шаблона)
* конечная точка подключения клиента: 19000 (можно настроить в параметрах шаблона);
* конечная точка HTTP-шлюза: 19080 (можно настроить в параметрах шаблона).

### <a name="azure-load-balancer"></a>Azure Load Balancer

В ресурсе **Microsoft.Network/loadBalancers** настроена подсистема балансировки нагрузки, а также указаны пробы и правила для следующих портов:

* конечная точка подключения клиента: 19000;
* конечная точка HTTP-шлюза: 19080;
* порт приложения: 80;
* порт приложения: 443;

### <a name="virtual-network-and-subnet"></a>Виртуальная сеть и подсеть

В параметрах шаблона объявляются имена виртуальной сети и подсети.  Адресные пространства виртуальной сети и подсети также объявляются в параметрах шаблона и настраиваются в ресурсе **Microsoft.Network/virtualNetworks**:

* адресное пространство виртуальной сети: 10.0.0.0/16;
* адресное пространство подсети Service Fabric: 10.0.2.0/24.

Если нужны другие порты приложений, нужно настроить ресурс Microsoft.Network/loadBalancers, чтобы разрешить входящий трафик.

## <a name="set-template-parameters"></a>Установка параметров шаблона

В файле **AzureDeploy. parameters** объявляются многие значения, используемые для развертывания кластера и связанных с ним ресурсов. Далее представлены некоторые параметры, которые может понадобиться изменить для развертывания:

|Параметр|Пример значения|Примечания|
|---|---||
|adminUserName|vmadmin| Имя администратора кластера виртуальных машин. |
|adminPassword|Password#1234| Пароль администратора для кластера виртуальных машин.|
|clusterName|mysfcluster123| Имя кластера. |
|location|southcentralus| Расположение кластера. |
|certificateThumbprint|| <p>Если создается самозаверяющий сертификат или указывается файл сертификата, значение должно быть пустым.</p><p>Если необходимо использовать имеющийся сертификат, который вы ранее передали в хранилище ключей, заполните значение отпечатка SHA1 сертификата. Например, 6190390162C988701DB5676EB81083EA608DCCF3. </p>|
|certificateUrlValue|| <p>Если создается самозаверяющий сертификат или указывается файл сертификата, значение должно быть пустым.</p><p>Если необходимо использовать имеющийся сертификат, который вы ранее передали в хранилище ключей, укажите URL-адрес сертификата. Например https:\//mykeyvault.vault.azure.net:443/secrets/mycertificate/02bea722c9ef4009a76c5052bcbf8346.</p>|
|sourceVaultValue||<p>Если создается самозаверяющий сертификат или указывается файл сертификата, значение должно быть пустым.</p><p>Если необходимо использовать имеющийся сертификат, который вы ранее передали в хранилище ключей, укажите сведения об исходном хранилище. Например, /subscriptions/333cc2c84-12fa-5778-bd71-c71c07bf873f/resourceGroups/MyTestRG/providers/Microsoft.KeyVault/vaults/MYKEYVAULT.</p>|

<a id="createvaultandcert" name="createvaultandcert_anchor"></a>

## <a name="deploy-the-virtual-network-and-cluster"></a>Развертывание кластера и виртуальной сети

Настройте топологию сети и разверните кластер Service Fabric. Шаблон Resource Manager **AzureDeploy.json** создает виртуальную сеть и подсеть для Service Fabric. Шаблон также развертывает кластер с включенным сертификатом безопасности.  Для рабочих кластеров в качестве сертификата нужно использовать сертификат из центра сертификации (ЦС). Самозаверяющий сертификат можно использовать для защиты тестовых кластеров.

Шаблон в этой статье развертывает кластер, использующий отпечаток сертификата для идентификации сертификата кластера.  Два сертификата не могут иметь один и тот же отпечаток. Это затрудняет управление сертификатами. Переключение развернутого кластера с использования отпечатков сертификата на использование общих имен сертификатов упрощает управление им.  Чтобы узнать, как обновить кластер, чтобы использовать общие имена сертификатов для управления сертификатами, прочтите статью [изменение кластера на общее имя сертификата](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

### <a name="create-a-cluster-using-an-existing-certificate"></a>Создание кластера с помощью имеющегося сертификата

В приведенном ниже сценарии используется команда [az sf cluster create](/cli/azure/sf/cluster) и шаблон, которые позволяют развернуть новый кластер с помощью имеющегося сертификата. Эта команда также создает хранилище ключей в Azure и отправляет сертификат.

```azurecli
ResourceGroupName="sflinuxclustergroup"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates\MyCertificate.pem"

# sign in to your Azure account and select your subscription
az login
az account set --subscription <guid>

# Create a new resource group for your deployment and give it a name and a location.
az group create --name $ResourceGroupName --location $Location

# Create the Service Fabric cluster.
az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --certificate-password $Password --certificate-file $CertPath \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName  \
   --template-file AzureDeploy.json --parameter-file AzureDeploy.Parameters.json
```

### <a name="create-a-cluster-using-a-new-self-signed-certificate"></a>Создание кластера с помощью нового самозаверяющего сертификата

В приведенном ниже сценарии используется команда [az sf cluster create](/cli/azure/sf/cluster) и шаблон, которые позволяют развернуть в Azure новый кластер. Кроме того, при помощи этой команды создается хранилище ключей в Azure, в него добавляется новый самозаверяющий сертификат и локально скачивается файл сертификата.

```azurecli
ResourceGroupName="sflinuxclustergroup"
ClusterName="sflinuxcluster"
Location="southcentralus"
Password="q6D7nN%6ck@6"
VaultName="linuxclusterkeyvault"
VaultGroupName="linuxclusterkeyvaultgroup"
CertPath="C:\MyCertificates"

az sf cluster create --resource-group $ResourceGroupName --location $Location \
   --cluster-name $ClusterName --template-file C:\temp\cluster\AzureDeploy.json \
   --parameter-file C:\temp\cluster\AzureDeploy.Parameters.json --certificate-password $Password \
   --certificate-output-folder $CertPath --certificate-subject-name $ClusterName.$Location.cloudapp.azure.com \
   --vault-name $VaultName --vault-resource-group $ResourceGroupName
```

## <a name="connect-to-the-secure-cluster"></a>Подключение к безопасному кластеру

Подключитесь к кластеру, выполнив в командной строке Service Fabric команду `sfctl cluster select` и указав свой ключ.  Примечание. Используйте параметр **--no-verify** только для самозаверяющего сертификата.

```console
sfctl cluster select --endpoint https://aztestcluster.southcentralus.cloudapp.azure.com:19080 \
--pem ./aztestcluster201709151446.pem --no-verify
```

Выполните команду `sfctl cluster health`, чтобы проверить подключение к кластеру и убедиться, что кластер работает.

```console
sfctl cluster health
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не собираетесь немедленно приступить к следующей статье, то можете [удалить кластер](./service-fabric-tutorial-delete-cluster.md), чтобы за него не взималась плата.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте, как [масштабировать кластер](service-fabric-tutorial-scale-cluster.md).

Шаблон в этой статье развертывает кластер, использующий отпечаток сертификата для идентификации сертификата кластера.  Два сертификата не могут иметь один и тот же отпечаток. Это затрудняет управление сертификатами. Переключение развернутого кластера с использования отпечатков сертификата на использование общих имен сертификатов упрощает управление им.  Чтобы узнать, как обновить кластер, чтобы использовать общие имена сертификатов для управления сертификатами, прочтите статью [изменение кластера на общее имя сертификата](service-fabric-cluster-change-cert-thumbprint-to-cn.md).

[template]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-3-NodeTypes-Secure/AzureDeploy.json
[parameters]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-3-NodeTypes-Secure/AzureDeploy.Parameters.json
[template2]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-1804-3-NodeTypes-Secure/AzureDeploy.json
[parameters2]:https://github.com/Azure-Samples/service-fabric-cluster-templates/blob/master/7-VM-Ubuntu-1804-3-NodeTypes-Secure/AzureDeploy.Parameters.json
