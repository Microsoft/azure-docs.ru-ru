---
title: Развертывание службы работоспособности — диспетчер развертывания Azure
description: В этой статье описывается, как развернуть службу в нескольких регионах с помощью диспетчера развертывания Azure. Кроме того, здесь приведены методики безопасного развертывания, которые позволяют проверить стабильность развертывания во все регионы, прежде чем выполнять этот процесс.
author: mumian
ms.topic: conceptual
ms.date: 09/21/2020
ms.author: jgao
ms.openlocfilehash: ae95269dbac3ef1561e19d4b7ea5dd383c1eed73
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99536973"
---
# <a name="introduce-health-integration-rollout-to-azure-deployment-manager-public-preview"></a>Внедрение интеграции работоспособности в Azure диспетчер развертывания (общедоступная Предварительная версия)

[Azure диспетчер развертывания](./deployment-manager-overview.md) позволяет выполнять промежуточное развертывания Azure Resource Manager ресурсов. Ресурсы разворачиваются по регионам в упорядоченном виде. Встроенная проверка работоспособности диспетчер развертывания Azure позволяет отслеживать развертывания и автоматически прекращает проблемные развертывания, позволяя устранять и уменьшать масштаб влияния. Эта функция может снизить недоступность службы, вызванную регрессией в обновлениях.

## <a name="health-monitoring-providers"></a>Поставщики мониторинга работоспособности

Чтобы максимально упростить интеграцию функций работоспособности, корпорация Майкрософт сотрудничает с некоторыми ведущими компаниями по мониторингу работоспособности служб, чтобы предоставить вам простое решение для копирования и вставки, позволяющее интегрировать проверки работоспособности с вашими развертываниями. Если вы еще не используете монитор работоспособности, это отличные решения, которые можно начать с:

| ![монитор Azure "монитор работоспособности диспетчера развертывания Azure"](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-azure-monitor.svg)| ![поставщик монитора работоспособности диспетчера развертывания Azure datadog](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-datadog.svg) | ![поставщик монитора работоспособности диспетчера развертывания Azure site24x7](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-site24x7.svg) | ![поставщик монитора работоспособности диспетчера развертывания Azure файл Wavefront](./media/deployment-manager-health-check/azure-deployment-manager-health-monitor-provider-wavefront.svg) |
|-----|-----|------|------|
|Azure Monitor, полнофункциональная платформа наблюдаемых стеков Майкрософт для гибридного мониторинга и аналитики облачной среды &. |Datadog — ведущая платформа мониторинга и аналитики для современных облачных сред. Узнайте [, как Datadog интегрируется с Azure диспетчер развертывания](https://www.datadoghq.com/azure-deployment-manager/).|Site24x7 — решение для мониторинга "все в одном частном" и "общедоступное облачное облако". Узнайте [, как Site24x7 интегрируется с Azure диспетчер развертывания](https://www.site24x7.com/azure/adm.html).| Файл Wavefront, платформа мониторинга и аналитики для сред приложений с несколькими облаками. Узнайте [, как файл Wavefront интегрируется с Azure диспетчер развертывания](https://go.wavefront.com/wavefront-adm/).|

## <a name="how-service-health-is-determined"></a>Как определяется работоспособность службы

[Поставщики мониторинга работоспособности](#health-monitoring-providers) предлагают несколько механизмов мониторинга служб и предупреждения о любых проблемах с работоспособностью служб. [Azure Monitor](../../azure-monitor/overview.md) является примером одного такого предложения. Azure Monitor можно использовать для создания предупреждений при превышении определенных пороговых значений. Например, память и загрузка ЦП выходят за пределы ожидаемых уровней при развертывании нового обновления для службы. При уведомлении можно предпринять корректирующие действия.

Эти поставщики работоспособности обычно предлагают интерфейсы API-интерфейсов RESTFUL, чтобы можно было проверять состояние мониторов службы программным способом. Интерфейсы API-интерфейса могут воздержаться в виде простого работоспособного или неработоспособного сигнала (определяемого кодом HTTP-ответа) и (или) с подробными сведениями о получаемых сигналах.

Новый `healthCheck` шаг в Azure диспетчер развертывания позволяет объявлять коды HTTP, указывающие на работоспособную службу. Для сложных результатов поиска неактивных запросов можно указать регулярные выражения, которые при совпадении указывают на работоспособный ответ.

Последовательность для настройки проверок работоспособности диспетчер развертывания Azure:

1. Создайте мониторы работоспособности с помощью выбранного поставщика службы работоспособности.
1. Создайте один или несколько `healthCheck` шагов в рамках развертывания диспетчер развертывания Azure. Заполните эти `healthCheck` шаги следующими сведениями:

    1. Универсальный код ресурса (URI) для REST API мониторов работоспособности (как определено поставщиком службы работоспособности).
    1. Сведения о проверке подлинности. В настоящее время поддерживается только проверка подлинности с помощью ключа API. Для Azure Monitor тип проверки подлинности должен быть установлен так, как `RolloutIdentity` назначенное пользователем управляемое удостоверение, используемое для развертывания Azure диспетчер развертывания, расширяется для Azure Monitor.
    1. [Коды состояния HTTP](https://www.wikipedia.org/wiki/List_of_HTTP_status_codes) или регулярные выражения, определяющие работоспособный ответ. Вы можете предоставить регулярные выражения, которые должны совпадать, чтобы ответ считался работоспособным, либо можно указать выражения, которые должны совпадать, чтобы ответ считался работоспособным. Поддерживаются оба метода.

    Следующий пример JSON является примером интеграции Azure Monitor с диспетчер развертывания Azure. В примере используется `RolloutIdentity` и устанавливается проверка работоспособности, когда развертывание продолжается, если нет предупреждений. Единственный поддерживаемый API Azure Monitor: [Alerts — получить все](/rest/api/monitor/alertsmanagement/alerts/getall).

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT1M",
          "maxElasticDuration": "PT1M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "RolloutIdentity"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "\"value\":\\[\\]"
                    ],
                    "matchQuantifier": "All"
                  }
                }
              }
            ]
          }
        }
      }
    }
    ```

    Ниже приведен пример JSON для всех других поставщиков мониторинга работоспособности:

    ```json
    {
      "type": "Microsoft.DeploymentManager/steps",
      "apiVersion": "2018-09-01-preview",
      "name": "healthCheckStep",
      "location": "[parameters('azureResourceLocation')]",
      "properties": {
        "stepType": "healthCheck",
        "attributes": {
          "waitDuration": "PT0M",
          "maxElasticDuration": "PT0M",
          "healthyStateDuration": "PT1M",
          "type": "REST",
          "properties": {
            "healthChecks": [
              {
                "name": "appHealth",
                "request": {
                  "method": "GET",
                  "uri": "[parameters('healthCheckUrl')]",
                  "authentication": {
                    "type": "ApiKey",
                    "name": "code",
                    "in": "Query",
                    "value": "[parameters('healthCheckAuthAPIKey')]"
                  }
                },
                "response": {
                  "successStatusCodes": [
                    "200"
                  ],
                  "regex": {
                    "matches": [
                      "Status: healthy",
                      "Status: warning"
                    ],
                    "matchQuantifier": "Any"
                  }
                }
              }
            ]
          }
        }
      }
    },
    ```

1. Вызовите `healthCheck` шаги в нужное время в ходе развертывания Azure диспетчер развертывания. В следующем примере `healthCheck` вызывается шаг в `postDeploymentSteps` `stepGroup2` .

    ```json
    "stepGroups": [
        {
            "name": "stepGroup1",
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup2",
            "dependsOnStepGroups": ["stepGroup1"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceWUS.name,  variables('serviceTopology').serviceWUS.serviceUnit1.name)]",
            "postDeploymentSteps": [
                {
                    "stepId": "[resourceId('Microsoft.DeploymentManager/steps/', 'healthCheckStep')]"
                }
            ]
        },
        {
            "name": "stepGroup3",
            "dependsOnStepGroups": ["stepGroup2"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit2.name)]",
            "postDeploymentSteps": []
        },
        {
            "name": "stepGroup4",
            "dependsOnStepGroups": ["stepGroup3"],
            "preDeploymentSteps": [],
            "deploymentTargetId": "[resourceId('Microsoft.DeploymentManager/serviceTopologies/services/serviceUnits', variables('serviceTopology').name, variables('serviceTopology').serviceEUS.name,  variables('serviceTopology').serviceEUS.serviceUnit1.name)]",
            "postDeploymentSteps": []
        }
    ]
    ```

Пример см. в разделе [учебник. Использование проверки работоспособности в диспетчер развертывания Azure](./deployment-manager-tutorial-health-check.md).

## <a name="phases-of-a-health-check"></a>Этапы проверки работоспособности

На этом этапе Azure диспетчер развертывания знает, как запрашивать сведения о работоспособности службы и на каких этапах развертывания. Однако диспетчер развертывания Azure также позволяет выполнять глубокую настройку времени этих проверок. `healthCheck`Шаг выполняется в трех последовательных этапах, все из которых имеют настраиваемую длительность:

1. Ожидание

    1. После завершения операции развертывания виртуальные машины могут перезагружаться, перенастраиваться на основе новых данных или даже запускаться в первый раз. Кроме того, для служб требуется некоторое время, чтобы службы начали выдавать сигналы о работоспособности для агрегирования поставщиком мониторинга работоспособности. Во время этого процесса забавная может не иметь смысла проверять работоспособность службы, так как обновление еще не достигло стабильного состояния. Действительно, служба может быть осЦиллатинг между работоспособными и неработоспособными состояниями при сопоставлении ресурсов.
    1. На этапе ожидания работоспособность службы не отслеживается. Это позволяет развернутым ресурсам внедрить время до начала процесса проверки работоспособности.

1. Elastic

    1. Поскольку во всех случаях невозможно понять, сколько времени будет выполняться до стабильной работы ресурсов, эластичный этап обеспечивает гибкий период времени между тем, что ресурсы потенциально нестабильны, и когда им требуется поддерживать работоспособное устойчивое состояние.
    1. Когда начнется эластичный этап, Azure диспетчер развертывания периодически опрашивает предоставленную конечную точку для работоспособности службы. Интервал опроса можно настроить.
    1. Если монитор работоспособности возвращается с сигналами, указывающими на то, что служба неработоспособна, эти сигналы игнорируются, этап эластичности продолжится, и опрос продолжится.
    1. Когда монитор работоспособности возвращает сигналы, указывающие, что служба работоспособна, этап эластичности завершается и начинается этап Хеалсистате.
    1. Таким же временем продолжительность, указанная для фазы эластичности, — это максимальное время, которое может быть затрачено на опрос работоспособности службы, прежде чем работоспособный ответ считается обязательным.

1. HealthyState

    1. На этапе Хеалсистате служба работоспособности постоянно опрашивается в том же интервале, что и эластичный этап.
    1. Служба должна поддерживать работоспособные сигналы от поставщика мониторинга работоспособности за весь указанный период.
    1. Если в любой момент обнаруживается неработоспособный ответ, Azure диспетчер развертывания останавливает весь процесс развертывания и возвращает ответ о неработоспособных сигналах службы.
    1. После завершения Хеалсистате длительности `healthCheck` завершается, а развертывание переходит к следующему шагу.

## <a name="next-steps"></a>Дальнейшие действия

В этой статье вы узнали, как интегрировать наблюдение за работоспособностью в Azure диспетчер развертывания. Перейдите к следующей статье, чтобы узнать, как выполнить развертывание с помощью диспетчера развертывания.

> [!div class="nextstepaction"]
> [Учебник. Использование проверки работоспособности в Azure диспетчер развертывания](./deployment-manager-tutorial-health-check.md)
