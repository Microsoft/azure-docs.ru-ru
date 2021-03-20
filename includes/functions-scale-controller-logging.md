---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 06/15/2020
ms.author: glenga
ms.openlocfilehash: 906413d0a6702e6146779f79d628b5cebf383af1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92165777"
---
| | |
|--|--|
|**`<DESTINATION>`**| Место назначения, в которое отправляются журналы. Допустимые значения: `AppInsights` и `Blob`.<br/>При использовании `AppInsights` Убедитесь, что [в приложении-функции включен Application Insights](../articles/azure-functions/configure-monitoring.md#enable-application-insights-integration).<br/>Если для назначения задано значение `Blob` , то журналы создаются в контейнере больших двоичных объектов с именем `azure-functions-scale-controller` в учетной записи хранения по умолчанию, заданной в `AzureWebJobsStorage` параметре приложения. |
|**`<VERBOSITY>`** | Задает уровень ведения журнала. Поддерживаются значения `None`, `Warning` и `Verbose`.<br/>Если задано значение `Verbose` , контроллер масштабирования регистрирует причину каждого изменения в количестве рабочих ролей, а также сведения о триггерах, которые учитываются в этих решениях. Подробные журналы содержат предупреждения триггеров и хэши, используемые триггерами до и после запуска контроллера масштабирования. |

> [!TIP]
> Помните, что при включении ведения журнала контроллера масштабирования это влияет на [потенциальные расходы на мониторинг приложения функции](../articles/azure-functions/functions-monitoring.md#application-insights-pricing-and-limits). Рекомендуется включить ведение журнала до тех пор, пока не будет собрано достаточно данных, чтобы понять, как работает контроллер масштабирования, а затем отключить его.