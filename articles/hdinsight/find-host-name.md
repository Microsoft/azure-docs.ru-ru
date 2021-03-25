---
title: Как получить имена узлов кластера Azure HDInsight
description: Узнайте, как получить имена узлов и полное доменное имя узлов кластера Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
author: yanancai
ms.author: yanacai
ms.date: 03/23/2021
ms.openlocfilehash: 676b4ccd52e8a6f1f378756a255ad55b74f18ebe
ms.sourcegitcommit: bed20f85722deec33050e0d8881e465f94c79ac2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/25/2021
ms.locfileid: "105111490"
---
# <a name="find-the-host-names-of-cluster-nodes"></a>Поиск имен узлов в узлах кластера

Кластер HDInsight создается с общедоступным DNS-clustername.azurehdinsight.net. При подключении к отдельным узлам по протоколу SSH или настройке подключения к узлам кластера с помощью одной и той же настраиваемой виртуальной сети необходимо использовать имя узла или полные доменные имена (FQDN) узлов кластера.

Из этой статьи вы узнаете, как получить имена узлов кластера. Его можно получить вручную с помощью веб-интерфейса Ambari или автоматически с помощью Ambari REST API.

> [!WARNING]
> Используйте следующие Рекомендуемые подходы для выборки имен узлов кластера. Номера в имени узла не гарантировано в последовательности, и HDInsight может изменить формат имени узла для согласования с виртуальными машинами с обновлением выпуска. Не следует зависеть от любого соглашения об именовании, существующего в настоящее время. 
>

Имена узлов можно получить с помощью пользовательского интерфейса Ambari или Ambari REST API. 

## <a name="get-the-host-names-from-ambari-web-ui"></a>Получение имен узлов из веб-интерфейса Ambari
Вы можете использовать веб-интерфейс Ambari для получения имен узлов при использовании SSH для узла. Представление "веб-интерфейс пользователя Ambari" доступно в кластере HDInsight по адресу `https://CLUSTERNAME.azurehdinsight.net/#/main/hosts` , где `CLUSTERNAME` — это имя кластера.

![Get-Host-Names-in-Ambari-UI](.\media\find-host-name\find-host-name-in-ambari-ui.png)

## <a name="get-the-host-names-from-ambari-rest-api"></a>Получение имен узлов из Ambari REST API
При создании скриптов автоматизации можно использовать REST API Ambari для получения имен узлов перед подключением к узлам. Номера в имени узла не гарантированы в последовательностях, и HDInsight может изменить формат имени узла для согласования с виртуальными машинами с обновлением выпуска. Не следует зависеть от любого соглашения об именовании, существующего в настоящее время. 

Ниже приведены некоторые примеры получения полного доменного имени для узлов в кластере. Дополнительные сведения о REST API Ambari см. [в статье Управление кластерами HDInsight с помощью Apache Ambari REST API](.\hdinsight-hadoop-manage-ambari-rest-api.md)

В следующем примере используется [JQ](https://stedolan.github.io/jq/) или [ConvertFrom-JSON](/powershell/module/microsoft.powershell.utility/convertfrom-json) для синтаксического анализа документа ответа JSON и отображаются только имена узлов.

```bash
export password=''
export clusterName=''
curl -u admin:$password -sS -G "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" \
| jq -r '.items[].Hosts.host_name'
```  

```powershell
$clusterName=''
$creds = Get-Credential -UserName "admin" -Message "Enter the HDInsight login"
$resp = Invoke-WebRequest -Uri "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/hosts" `
    -Credential $creds -UseBasicParsing
$respObj = ConvertFrom-Json $resp.Content
$respObj.items.Hosts.host_name
```