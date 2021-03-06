---
title: Адреса управления
description: Найдите адреса управления, используемые для управления Средой службы приложений. Включите их в таблицу маршрутов, чтобы избежать проблем с асимметричной маршрутизацией.
author: ccompy
ms.assetid: a7738a24-89ef-43d3-bff1-77f43d5a3952
ms.topic: article
ms.date: 03/22/2021
ms.author: ccompy
ms.custom: seodec18, references_regions
ms.openlocfilehash: 25bdfb7a0301af472baa89d3d9e73aacf8cff139
ms.sourcegitcommit: ac035293291c3d2962cee270b33fca3628432fac
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "104954621"
---
# <a name="app-service-environment-management-addresses"></a>Адреса управления среды службы приложений

Среда службы приложений Azure (ASE) — это реализация Службы приложений Azure с одним клиентом в виртуальной сети Azure.  Хотя ASE выполняется в виртуальной сети, она должна быть доступна для нескольких выделенных IP-адресов, которые используются Службой приложений Azure для управления службой.  При использовании ASE трафик управления проходит через контролируемую пользователем сеть. Если этот трафик блокируется или неправильно маршрутизируется, работа ASE приостанавливается. Подробные сведения о зависимостях сетей ASE см. в статье [Рекомендации по работе с сетями в Среде службы приложений][networking]. Для получения общей информации об ASE можно начать со статьи [Общие сведения о средах службы приложений][intro].

Каждая среда ASE имеет общедоступный виртуальный IP-адрес, на который поступает трафик управления. Входящий трафик управления с этих адресов на поступает на порты 454 и 455 для общедоступного виртуального IP-адреса ASE. В этом документе перечислены адреса источников службы приложений, передающих трафик управления в среду ASE. Эти адреса указаны также в теге службы протокола IP с именем AppServiceManagement.

Указанные ниже адреса можно настроить в таблице маршрутов, чтобы избежать возникновения проблем с асимметричной маршрутизацией для трафика управления. Маршруты применяются к трафику на уровне IP-адреса и не имеют сведений о направлении этого трафика или о том, что этот трафик является частью ответного сообщения TCP. Если адрес ответа на TCP-запрос отличается от адреса, на который он был отправлен, возникает проблема с асимметричной маршрутизацией. Чтобы избежать проблем с асимметричной маршрутизацией для трафика управления ASE, необходимо обеспечить возврат ответов с того адреса, на который был отправлен запрос. Дополнительные сведения о настройке работы ASE в среде, где исходящий трафик передается локально, см. в статье [Настройка принудительного туннелирования в среде службы приложений][forcedtunnel].

## <a name="list-of-management-addresses"></a>Список адресов управления ##

| Регион | Адреса |
|--------|-----------|
| Все общедоступные регионы | 13.66.140.0, 13.67.8.128, 13.69.64.128, 13.69.227.128, 13.70.73.128, 13.71.170.64, 13.71.194.129, 13.75.34.192, 13.75.127.117, 13.77.50.128, 13.78.109.0, 13.89.171.0, 13.94.141.115, 13.94.143.126, 13.94.149.179, 20.36.106.128, 20.36.114.64, 20.37.74.128, 23.96.195.3, 23.102.135.246, 23.102.188.65, 40.69.106.128, 40.70.146.128, 40.71.13.64, 40.74.100.64, 40.78.194.128, 40.79.130.64, 40.79.178.128, 40.83.120.64, 40.83.121.56, 40.83.125.161, 40.112.242.192, 51.107.58.192, 51.107.154.192, 51.116.58.192, 51.116.155.0, 51.120.99.0, 51.120.219.0, 51.140.146.64, 51.140.210.128, 52.151.25.45, 52.162.106.192, 52.165.152.214, 52.165.153.122, 52.165.154.193, 52.165.158.140, 52.174.22.21, 52.178.177.147, 52.178.184.149, 52.178.190.65, 52.178.195.197, 52.187.56.50, 52.187.59.251, 52.187.63.19 52.231.146.128, 65.52.172.237, 65.52.250.128, 70.37.57.58, 104.44.129.141, 104.44.129.243, 104.44.129.255, 104.44.134.255, 104.208.54.11, 104.211.81.64, 104.211.146.128, 157.55.208.185, 191.233.50.128, 191.233.203.64, 191.236.154.88 |
| Azure для государственных организаций | 23.97.29.209, 13.72.53.37, 13.72.180.105, 52.181.183.11, 52.227.80.100, 52.182.93.40, 52.244.79.34, 52.238.74.16 |
| Azure для Китая | 42.159.4.236, 42.159.80.125 |

## <a name="configuring-a-network-security-group"></a>Настройка группы безопасности сети

При использовании групп безопасности сети не нужно беспокоиться об отдельных адресах или обслуживании собственной конфигурации. Существует тег службы протокола IP с именем AppServiceManagement, который всегда содержит актуальный список адресов. Чтобы применить этот тег службы протокола IP в группе безопасности сети, перейдите на портал, откройте пользовательский интерфейс групп безопасности сети и выберите правила безопасности для входящего трафика. Если у вас есть готовое правило для входящего трафика управления, измените его. Если эта группа безопасности сети не была создана с использованием ASE или если она совсем новая, выберите **Добавить**. В раскрывающемся списке источника выберите **Тег службы**.  В разделе тега службы источника выберите **AppServiceManagement**. Укажите для диапазонов портов источника значение \*, для назначения — **Любые**, для диапазонов портов назначения — **454–455**, для протокола — **TCP**, а для действия — **Разрешить**. При создании правила необходимо задать приоритет. 

![создание группы безопасности сети с тегом службы][1]

## <a name="configuring-a-route-table"></a>Настройка таблицы маршрутов

Адреса управления можно поместить в таблицу маршрутов с типом следующего прыжка "Интернет", чтобы обеспечить возможность возвращения всего входящего трафика управления по тому же пути. Эти маршруты необходимы при настройке принудительного туннелирования. Создать таблицу маршрутов можно с помощью портала, PowerShell или Azure CLI.  Ниже приведены команды, позволяющие создать таблицу маршрутов с помощью Azure CLI из командной строки PowerShell. 

```azurepowershell-interactive
$rg = "resource group name"
$rt = "route table name"
$location = "azure location"
$managementAddresses = "13.66.140.0", "13.67.8.128", "13.69.64.128", "13.69.227.128", "13.70.73.128", "13.71.170.64", "13.71.194.129", "13.75.34.192", "13.75.127.117", "13.77.50.128", "13.78.109.0", "13.89.171.0", "13.94.141.115", "13.94.143.126", "13.94.149.179", "20.36.106.128", "20.36.114.64", "20.37.74.128", "23.96.195.3", "23.102.135.246", "23.102.188.65", "40.69.106.128", "40.70.146.128", "40.71.13.64", "40.74.100.64", "40.78.194.128", "40.79.130.64", "40.79.178.128", "40.83.120.64", "40.83.121.56", "40.83.125.161", "40.112.242.192", "51.107.58.192", "51.107.154.192", "51.116.58.192", "51.116.155.0", "51.120.99.0", "51.120.219.0", "51.140.146.64", "51.140.210.128", "52.151.25.45", "52.162.106.192", "52.165.152.214", "52.165.153.122", "52.165.154.193", "52.165.158.140", "52.174.22.21", "52.178.177.147", "52.178.184.149", "52.178.190.65", "52.178.195.197", "52.187.56.50", "52.187.59.251", "52.187.63.19", "52.187.63.37", "52.224.105.172", "52.225.177.153", "52.231.18.64", "52.231.146.128", "65.52.172.237", "65.52.250.128", "70.37.57.58", "104.44.129.141", "104.44.129.243", "104.44.129.255", "104.44.134.255", "104.208.54.11", "104.211.81.64", "104.211.146.128", "157.55.208.185", "191.233.50.128", "191.233.203.64", "191.236.154.88"

az network route-table create --name $rt --resource-group $rg --location $location
foreach ($ip in $managementAddresses) {
    az network route-table route create -g $rg --route-table-name $rt -n $ip --next-hop-type Internet --address-prefix ($ip + "/32")
}
```

Создав свою таблицу маршрутов, необходимо задать ее для подсети ASE.  

## <a name="get-your-management-addresses-from-api"></a>Получение адресов управления из API ##

С помощью вызова API системы можно получить список адресов управления, которые соответствуют ASE.

```http
get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01
```

API возвращает документ JSON, который включает все адреса входящего трафика для среды ASE. Список адресов включает адреса управления, виртуальный IP-адрес, используемый ASE и диапазон адресов подсети ASE.  

Для вызова API с помощью [armclient](https://github.com/projectkudu/ARMClient) используйте приведенные ниже команды, но укажите свои идентификатор подписки, группу ресурсов и имя среды ASE.  

```azurepowershell-interactive
armclient login
armclient get /subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Web/hostingEnvironments/<ASE Name>/inboundnetworkdependenciesendpoints?api-version=2016-09-01
```

<!--IMAGES-->
[1]: ./media/management-addresses/managementaddr-nsg.png

<!-- LINKS -->
[networking]: ./network-info.md
[intro]: ./intro.md
[forcedtunnel]: ./forced-tunnel-support.md
