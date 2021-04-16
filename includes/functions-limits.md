---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 05/04/2020
ms.author: glenga
ms.openlocfilehash: 91f72117fdbcdbeda1d906a9760243e66404920c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105104574"
---
| Ресурс |[План потребления](../articles/azure-functions/consumption-plan.md)|[План категории "Премиум"](../articles/azure-functions/functions-premium-plan.md)|[План ценовой категории "Выделенный"](../articles/azure-functions/dedicated-plan.md)|[ASE](../articles/app-service/environment/intro.md)| [Kubernetes](../articles/aks/quotas-skus-regions.md) |
| --- | --- | --- | --- | --- | --- |
|[Время ожидания](../articles/azure-functions/functions-scale.md#timeout) по умолчанию (в мин) |5 | 30 |30<sup>1</sup> | 30 | 30 |
|Максимальное [время ожидания](../articles/azure-functions/functions-scale.md#timeout) (в мин) |10 | не ограничено<sup>7</sup> | не ограничено<sup>2</sup> | unbounded | unbounded |
| Максимальное количество исходящих подключений (на экземпляр) | 600 активных (всего 1200) | unbounded | unbounded | unbounded | unbounded |
| Максимальный размер запроса (МБ)<sup>3</sup> | 100 | 100 | 100 | 100 | Зависит от кластера |
| Максимальная длина строки запроса<sup>3</sup> | 4096 | 4096 | 4096 | 4096 | Зависит от кластера |
| Максимальная длина URL-адреса запроса<sup>3</sup> | 8192 | 8192 | 8192 | 8192 | Зависит от кластера |
|[ACU](../articles/virtual-machines/acu.md) на экземпляр | 100 | 210–840 | 100–840 | 210–250<sup>8</sup> | [Цены на AKS](https://azure.microsoft.com/pricing/details/container-service/) |
| Максимальный объем памяти (ГБ на экземпляр) | 1.5 | 3,5–14 | 1,75–14 | 3,5–14 | Поддерживается любой узел |
| Максимальное число экземпляров | 200 | 100<sup>9</sup> | зависит от SKU<sup>10</sup> | 100<sup>10</sup> | Зависит от кластера |   
| Количество приложений-функций на план |100 |100 |не ограничено<sup>4</sup> | unbounded | unbounded |
| [Планы службы приложений](../articles/app-service/overview-hosting-plans.md) | 100 на [регион](https://azure.microsoft.com/global-infrastructure/regions/) |100 шт. на одну группу ресурсов |100 шт. на одну группу ресурсов | - | - |
| Хранилище<sup>5</sup> |5 ТБ |250 ГБ |50–1000 ГБ | 1 TБ | Недоступно |
| Количество личных доменов на одно приложение</a> |500<sup>6</sup> |500 |500 | 500 | Недоступно |
| личных доменов [Поддержка SSL](../articles/app-service/configure-ssl-bindings.md) |включено неограниченное количество SSL-соединений на основе SNI | включено неограниченное количество SSL-соединений на основе SNI и 1 SSL-соединение на основе IP |включено неограниченное количество SSL-соединений на основе SNI и 1 SSL-соединение на основе IP | включено неограниченное количество SSL-соединений на основе SNI и 1 SSL-соединение на основе IP | Недоступно |

<sup>1</sup> По умолчанию время ожидания для среды выполнения Функций версии 1.x в плане службы приложений не ограничено.  
<sup>2</sup> Требует, чтобы для плана службы приложений был задан параметр [Всегда включено](../articles/azure-functions/dedicated-plan.md#always-on). Оплата по стандартным [тарифам](https://azure.microsoft.com/pricing/details/app-service/).  
<sup>3</sup> Такие ограничения [заданы на уровне узла](https://github.com/Azure/azure-functions-host/blob/dev/src/WebJobs.Script.WebHost/web.config).  
<sup>4</sup> Фактическое количество приложений-функций, которые можно разместить, определяется интенсивностью работы приложений, размером экземпляров виртуальных машин и степенью использования соответствующих ресурсов.  
<sup>5</sup> Максимальный размер хранилища — это общий размер содержимого во временном хранилище для всех приложений в одном плане службы приложений. В плане потребления в качестве временного хранилище используется служба "Файлы Azure".  
<sup>6</sup> Если ваше приложение-функция размещается в [плане потребления](../articles/azure-functions/consumption-plan.md), поддерживается только параметр CNAME. Для приложений-функций в [плане категории "Премиум"](../articles/azure-functions/functions-premium-plan.md) или в [плане службы приложений](../articles/azure-functions/dedicated-plan.md) вы можете сопоставить личный домен с помощью записи CNAME или A.  
<sup>7</sup> Гарантированное время до 60 мин.  
<sup>8</sup> Рабочие роли — это роли в которых размещаются приложения клиента. Они доступны в трех фиксированных размерах: один виртуальный ЦП/3,5 ГБ ОЗУ; два виртуальных ЦП/7 ГБ ОЗУ; четыре виртуальных ЦП/14 ГБ ОЗУ.   
<sup>9</sup> В Linux в плане "Премиум" сейчас действует ограничение в 20 экземпляров.  
<sup>10</sup> Дополнительные сведения см. в разделе [Ограничения Службы приложений](../articles/azure-resource-manager/management/azure-subscription-service-limits.md#app-service-limits).