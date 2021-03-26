---
title: Получение метаданных подсистемы балансировки нагрузки с помощью службы метаданных экземпляра Azure (IMDS)
titleSuffix: Azure Load Balancer
description: Узнайте, как получить метаданные балансировщика нагрузки с помощью службы метаданных экземпляра Azure.
services: load-balancer
author: asudbring
ms.service: load-balancer
ms.topic: how-to
ms.date: 02/12/2021
ms.author: allensu
ms.openlocfilehash: 95d0e1ceb9e05ce58f388c3f88dc98b2cf6a0cc5
ms.sourcegitcommit: f0a3ee8ff77ee89f83b69bc30cb87caa80f1e724
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105559593"
---
# <a name="retrieve-load-balancer-metadata-using-the-azure-instance-metadata-service-imds"></a>Получение метаданных подсистемы балансировки нагрузки с помощью службы метаданных экземпляра Azure (IMDS)

## <a name="prerequisites"></a>Предварительные условия

* Используйте [последнюю версию API](../virtual-machines/windows/instance-metadata-service.md?tabs=windows#supported-api-versions) для запроса.

## <a name="sample-request-and-response"></a>Пример запроса и ответа
> [!IMPORTANT]
> Этот пример обходит учетные записи-посредники. При запросе IMDS **необходимо** обходить прокси-серверы. Дополнительные сведения см. в разделе [учетные записи-посредники](../virtual-machines/windows/instance-metadata-service.md?tabs=windows#proxies).
### <a name="windows"></a>[Windows](#tab/windows/)

```powershell
Invoke-RestMethod -Headers @{"Metadata"="true"} -Method GET -NoProxy -Uri "http://169.254.169.254:80/metadata/loadbalancer?api-version=2020-10-01" | ConvertTo-Json
```
> [!NOTE]
> Параметр--proxy появился в PowerShell 6,0. Если вы используете более раннюю версию PowerShell, удалите прокси-сервер в тексте запроса и убедитесь, что вы не используете прокси-сервер при получении сведений о IMDS. Дополнительные сведения см. [здесь](../virtual-machines/windows/instance-metadata-service.md?tabs=windows#proxies).
> 
### <a name="linux"></a>[Linux](#tab/linux/)

```bash
curl -H "Metadata:true" --noproxy "*" "http://169.254.169.254:80/metadata/loadbalancer?api-version=2020-10-01"
```

---
### <a name="sample-response"></a>Пример ответа

```json
{
   "loadbalancer": {
    "publicIpAddresses":[
      {
         "frontendIpAddress":"51.0.0.1",
         "privateIpAddress":"10.1.0.4"
      }
   ],
   "inboundRules":[
      {
         "frontendIpAddress":"50.0.0.1",
         "protocol":"tcp",
         "frontendPort":80,
         "backendPort":443,
         "privateIpAddress":"10.1.0.4"
      },
      {
         "frontendIpAddress":"2603:10e1:100:2::1:1",
         "protocol":"tcp",
         "frontendPort":80,
         "backendPort":443,
         "privateIpAddress":"ace:cab:deca:deed::1"
      }
   ],
   "outboundRules":[
      {
         "frontendIpAddress":"50.0.0.1",
         "privateIpAddress":"10.1.0.4"
      },
      {
         "frotendIpAddress":"2603:10e1:100:2::1:1",
         "privateIpAddress":"ace:cab:deca:deed::1"
      }
    ]
   }
}

```

## <a name="next-steps"></a>Дальнейшие действия
[Распространенные коды ошибок и действия по устранению неполадок](troubleshoot-load-balancer-imds.md)

Дополнительные сведения о [службе метаданных экземпляров Azure](../virtual-machines/windows/instance-metadata-service.md)

[Получение всех метаданных для экземпляра](../virtual-machines/windows/instance-metadata-service.md?tabs=windows#access-azure-instance-metadata-service)

[Развертывание стандартной подсистемы балансировки нагрузки](quickstart-load-balancer-standard-public-portal.md)