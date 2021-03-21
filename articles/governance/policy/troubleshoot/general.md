---
title: Устранение распространенных ошибок
description: Узнайте, как устранять неполадки при создании определений политик, различных пакетах SDK и надстройки для Kubernetes.
ms.date: 01/26/2021
ms.topic: troubleshooting
ms.openlocfilehash: 6e0e4067f07266bae9c87fd4443d27314cc28c0b
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100592609"
---
# <a name="troubleshoot-errors-with-using-azure-policy"></a>Устранение ошибок при использовании политики Azure

При создании определений политик, работе с пакетами SDK или настройке [политики Azure для](../concepts/policy-for-kubernetes.md) надстройки Kubernetes можно столкнуться с ошибками. В этой статье описываются различные общие ошибки, которые могут возникнуть, и предлагаются способы их устранения.

## <a name="find-error-details"></a>Найти сведения об ошибке

Расположение сведений об ошибке зависит от того, с какими аспектами политики Azure вы работаете.

- Если вы работаете с пользовательской политикой, перейдите к портал Azure, чтобы получить linting отзыв о схеме, или просмотрите полученные [данные о соответствии](../how-to/get-compliance-data.md) , чтобы увидеть, как были оценены ресурсы.
- Если вы работаете с любым из различных пакетов SDK, пакет SDK предоставляет сведения о том, почему функция завершилась ошибкой.
- Если вы работаете с надстройкой для Kubernetes, начните с [ведения журнала](../concepts/policy-for-kubernetes.md#logging) в кластере.

## <a name="general-errors"></a>Общие ошибки

### <a name="scenario-alias-not-found"></a>Сценарий: псевдоним не найден

#### <a name="issue"></a>Проблема

В определении политики используется недопустимый или несуществующий псевдоним. Политика Azure использует [псевдонимы](../concepts/definition-structure.md#aliases) для соотнесения со свойствами Azure Resource Manager.

#### <a name="cause"></a>Причина

В определении политики используется недопустимый или несуществующий псевдоним.

#### <a name="resolution"></a>Решение

Сначала убедитесь, что свойство диспетчер ресурсов имеет псевдоним. Чтобы найти доступные псевдонимы, перейдите к [расширению политики Azure для Visual Studio Code](../how-to/extension-for-vscode.md) или пакета SDK.
Если псевдоним для свойства диспетчер ресурсов не существует, создайте запрос в службу поддержки.

### <a name="scenario-evaluation-details-arent-up-to-date"></a>Сценарий: сведения об оценке не актуальны

#### <a name="issue"></a>Проблема

Ресурс находится в состоянии " _не запущено_ " или сведения о соответствии не являются актуальными.

#### <a name="cause"></a>Причина

Применение новой политики или назначения инициативы займет около 30 минут. Новые или обновленные ресурсы в области действия существующего назначения становятся доступными примерно через 15 минут. Стандартная проверка соответствия выполняется каждые 24 часа. Дополнительные сведения см. в разделе [триггеры оценки](../how-to/get-compliance-data.md#evaluation-triggers).

#### <a name="resolution"></a>Решение

Сначала дождитесь необходимого времени для завершения оценки, и результаты обеспечения соответствия будут доступны в портал Azure или пакете SDK. Чтобы запустить новую проверку для оценки с Azure PowerShell или REST API, см. статью [Проверка оценки по требованию](../how-to/get-compliance-data.md#on-demand-evaluation-scan).

### <a name="scenario-compliance-isnt-as-expected"></a>Сценарий: соответствие не так, как ожидалось

#### <a name="issue"></a>Проблема

Ресурс не входит в состояние оценки, _соответствующее_ или _не соответствующее требованиям_ , которое ожидается для ресурса.

#### <a name="cause"></a>Причина

Ресурс не находится в нужной области для назначения политики, или определение политики не работает должным образом.

#### <a name="resolution"></a>Решение

Чтобы устранить неполадки определения политики, выполните следующие действия.

1. Сначала дождитесь необходимого времени для завершения оценки, и результаты обеспечения соответствия будут доступны в портал Azure или пакете SDK. 

1. Чтобы запустить новую проверку для оценки с Azure PowerShell или REST API, см. статью [Проверка оценки по требованию](../how-to/get-compliance-data.md#on-demand-evaluation-scan).
1. Убедитесь, что параметры назначения и область назначения заданы правильно.
1. Проверьте [режим определения политики](../concepts/definition-structure.md#mode):
   - Режим должен быть `all` для всех типов ресурсов.
   - Режим должен иметь значение, `indexed` Если определение политики проверяет наличие тегов или расположения.
1. Убедитесь, что область действия ресурса не исключена или [не](../concepts/exemption-structure.md) [исключается](../concepts/assignment-structure.md#excluded-scopes) .
1. Если соответствие для назначения политики показывает `0/0` ресурсы, ресурсы не были определены как применимые в области назначения. Проверьте определение политики и область назначения.
1. Для несоответствующего ресурса, который должен быть соответствующим, см. раздел [Определение причин несоответствия](../how-to/determine-non-compliance.md). Сравнение определения с вычисленным значением свойства указывает, почему ресурс был несоответствующим.
   - Если **целевое значение** неверно, измените определение политики.
   - Если **Текущее значение** не подходит, проверьте полезные данные ресурса с помощью `resources.azure.com` .
1. Другие распространенные проблемы и решения см. в разделе [Устранение неполадок: принудительное применение не так, как ожидалось](#scenario-enforcement-not-as-expected).

Если у вас по-прежнему возникли проблемы с дубликатом и настраиваемым определением встроенной политики или пользовательским определением, создайте запрос в службу поддержки в разделе **Создание политики** для правильной маршрутизации проблемы.

### <a name="scenario-enforcement-not-as-expected"></a>Сценарий: принудительное применение не так, как ожидалось

#### <a name="issue"></a>Проблема

Ресурс, для которого требуется действовать политика Azure, не обрабатывается, и в [журнале действий Azure](../../../azure-monitor/essentials/platform-logs-overview.md)нет записи.

#### <a name="cause"></a>Причина

Назначение политики настроено для параметра [**Енфорцементмоде**](../concepts/assignment-structure.md#enforcement-mode) _Disabled (отключено_).
Хотя **енфорцементмоде** отключен, этот результат политики не применяется, и в журнале действий нет записи.

#### <a name="resolution"></a>Решение

Устраните неполадки применения назначения политики, выполнив следующие действия.

1. Сначала дождитесь необходимого времени для завершения оценки, и результаты обеспечения соответствия будут доступны в портал Azure или пакете SDK. 

1. Чтобы запустить новую проверку для оценки с Azure PowerShell или REST API, см. статью [Проверка оценки по требованию](../how-to/get-compliance-data.md#on-demand-evaluation-scan).
1. Убедитесь, что параметры назначения и область назначения заданы правильно и что **Енфорцементмоде** _включен_.
1. Проверьте [режим определения политики](../concepts/definition-structure.md#mode):
   - Режим должен быть `all` для всех типов ресурсов.
   - Режим должен иметь значение, `indexed` Если определение политики проверяет наличие тегов или расположения.
1. Убедитесь, что область действия ресурса не исключена или [не](../concepts/exemption-structure.md) [исключается](../concepts/assignment-structure.md#excluded-scopes) .
1. Убедитесь, что полезные данные ресурса соответствуют логике политики. Это можно сделать, [записывая трассировку HTTP Archive (HAR)](../../../azure-portal/capture-browser-trace.md) или просмотрев свойства шаблона Azure Resource Manager (шаблон ARM).
1. Другие распространенные проблемы и решения см. в разделе [Устранение неполадок: соответствие не так, как ожидалось](#scenario-compliance-isnt-as-expected).

Если у вас по-прежнему возникли проблемы с дубликатом и настраиваемым определением встроенной политики или пользовательским определением, создайте запрос в службу поддержки в разделе **Создание политики** для правильной маршрутизации проблемы.

### <a name="scenario-denied-by-azure-policy"></a>Сценарий: отклонено политикой Azure

#### <a name="issue"></a>Проблема

Для создания или обновления ресурса отказано.

#### <a name="cause"></a>Причина

Назначение политики области нового или обновленного ресурса соответствует критериям определения политики с применением [запрета](../concepts/effects.md#deny) . Создание или обновление ресурсов, соответствующих этим определениям, запрещено.

#### <a name="resolution"></a>Решение

Сообщение об ошибке из назначения «запретить политику» включает в себя определение политики и идентификаторы назначения политики. Если информация об ошибке в сообщении отсутствует, она также доступна в [журнале действий](../../../azure-monitor/essentials/activity-log.md#view-the-activity-log). Используйте эти сведения для получения дополнительных сведений об ограничениях ресурсов и настройке свойств ресурсов в запросе в соответствии с допустимыми значениями.

## <a name="template-errors"></a>Ошибки шаблона

### <a name="scenario-policy-supported-functions-processed-by-template"></a>Сценарий: Поддерживаемые функцией политики функции, обрабатываемые шаблоном

#### <a name="issue"></a>Проблема

Политика Azure поддерживает ряд функций и функций шаблонов ARM, доступных только в определении политики. Диспетчер ресурсов обрабатывает эти функции как часть развертывания, а не как часть определения политики.

#### <a name="cause"></a>Причина

Использование поддерживаемых функций, таких как `parameter()` или `resourceGroup()` , приводит к обработке результата функции во время развертывания, вместо того чтобы разрешить функции определения политики и обработчика политики Azure обрабатываться.

#### <a name="resolution"></a>Решение

Чтобы передать функцию через как часть определения политики, заэкранирование всей строки с `[` таким образом, чтобы свойство было похоже на `[[resourceGroup().tags.myTag]` . Escape-символ заставляет диспетчер ресурсов считать значение строкой при обработке шаблона. Затем политика Azure помещает функцию в определение политики, что позволяет использовать ее динамически, как и ожидалось. Дополнительные сведения см. [в разделе синтаксис и выражения в шаблонах Azure Resource Manager](../../../azure-resource-manager/templates/template-expressions.md).

## <a name="add-on-for-kubernetes-installation-errors"></a>Надстройка для ошибок установки Kubernetes

### <a name="scenario-installation-by-using-a-helm-chart-fails-because-of-a-password-error"></a>Сценарий: Установка с помощью диаграммы Helm завершается сбоем из-за ошибки в пароле

#### <a name="issue"></a>Проблема

`helm install azure-policy-addon`Команда завершается ошибкой и возвращает одну из следующих ошибок:

- `!: event not found`
- `Error: failed parsing --set data: key "<key>" has no value (cannot end with ,)`

#### <a name="cause"></a>Причина

Созданный пароль содержит запятую ( `,` ), на которой Helm диаграмма.

#### <a name="resolution"></a>Решение

При запуске используйте `helm install azure-policy-addon` escape-символ ( `,` ) в значении пароля с обратной косой чертой ( `\` ).

### <a name="scenario-installation-by-using-a-helm-chart-fails-because-the-name-already-exists"></a>Сценарий: Установка с использованием диаграммы Helm завершается сбоем, так как имя уже существует

#### <a name="issue"></a>Проблема

`helm install azure-policy-addon`Команда завершается ошибкой и возвращает следующую ошибку:

- `Error: cannot re-use a name that is still in use`

#### <a name="cause"></a>Причина

Диаграмма Helm с именем `azure-policy-addon` уже установлена или частично установлена.

#### <a name="resolution"></a>Решение

Следуйте инструкциям, чтобы [удалить политику Azure для надстройки Kubernetes](../concepts/policy-for-kubernetes.md#remove-the-add-on), а затем выполните команду повторно `helm install azure-policy-addon` .

### <a name="scenario-azure-virtual-machine-user-assigned-identities-are-replaced-by-system-assigned-managed-identities"></a>Сценарий: назначенные пользователем удостоверения виртуальной машины Azure заменяются управляемыми идентификаторами, назначенными системой

#### <a name="issue"></a>Проблема

После назначения инициатив политики гостевой конфигурации для аудита параметров внутри компьютера назначенные пользователю управляемые удостоверения, назначенные компьютеру, больше не будут назначены. Назначается только управляемое системой удостоверение.

#### <a name="cause"></a>Причина

Определения политик, которые ранее использовались в определениях гостевой конфигурации DeployIfNotExists, гарантируют, что назначенное системой удостоверение назначено компьютеру, но они также удаляют назначенные пользователем назначения удостоверений.

#### <a name="resolution"></a>Решение

Определения, которые ранее вызывали эту неполадку, отображаются как _\[ устаревшие \]_ и заменяются определениями политик, которые управляют необходимыми компонентами без удаления назначенных пользователем управляемых удостоверений. Требуется выполнить шаг вручную. Удалите все существующие назначения политик, помеченные как _\[ устаревшие \]_, и замените их обновленными инициативами и определениями политик, имена которых совпадают с именами исходных файлов.

Подробное описание см. в записи блога [важные изменения, выпущенные для политик аудита настройки гостя](https://techcommunity.microsoft.com/t5/azure-governance-and-management/important-change-released-for-guest-configuration-audit-policies/ba-p/1655316).

## <a name="add-on-for-kubernetes-general-errors"></a>Общая ошибка надстройки для Kubernetes

### <a name="scenario-the-add-on-is-unable-to-reach-the-azure-policy-service-endpoint-because-of-egress-restrictions"></a>Сценарий: надстройке не удалось связаться с конечной точкой службы политики Azure из-за ограничений исходящего трафика

#### <a name="issue"></a>Проблема

Надстройка не может связаться с конечной точкой службы политики Azure и возвращает одну из следующих ошибок:

- `failed to fetch token, service not reachable`
- `Error getting file "Get https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/template.yaml: dial tcp 151.101.228.133.443: connect: connection refused`

#### <a name="cause"></a>Причина

Эта проблема возникает при блокировании исходящего трафика кластера.

#### <a name="resolution"></a>Решение

Убедитесь, что домены и порты, указанные в следующих статьях, открыты:

- [Требуемые правила исходящей сети и полные доменные имена (FQDN) для кластеров AKS](../../../aks/limit-egress-traffic.md#required-outbound-network-rules-and-fqdns-for-aks-clusters)
- [Установка надстройки политики Azure для Kubernetes с поддержкой Arc Azure (Предварительная версия)](../concepts/policy-for-kubernetes.md#install-azure-policy-add-on-for-azure-arc-enabled-kubernetes)

### <a name="scenario-the-add-on-is-unable-to-reach-the-azure-policy-service-endpoint-because-of-the-aad-pod-identity-configuration"></a>Сценарий: надстройка не может связаться с конечной точкой службы политики Azure из-за конфигурации AAD – Pod-Identity.

#### <a name="issue"></a>Проблема

Надстройка не может связаться с конечной точкой службы политики Azure и возвращает одну из следующих ошибок:

- `azure.BearerAuthorizer#WithAuthorization: Failed to refresh the Token for request to https://gov-prod-policy-data.trafficmanager.net/checkDataPolicyCompliance?api-version=2019-01-01-preview: StatusCode=404`
- `adal: Refresh request failed. Status Code = '404'. Response body: getting assigned identities for pod kube-system/azure-policy-8c785548f-r882p in CREATED state failed after 16 attempts, retry duration [5]s, error: <nil>`

#### <a name="cause"></a>Причина

Эта ошибка возникает, если в кластере установлена _надстройка Add-Pod-Identity_ , а _KUBE-System_ не исключены в _AAD-Pod-Identity_.

Модули управления " _AAD-Pod-Identity_ " с узлом компонента идентификации (NMI) изменяют узлы "Iptables" для перехвата вызовов к конечной точке метаданных экземпляра Azure. Такая настройка означает, что любой запрос, отправленный в конечную точку метаданных, перехватывается функцией NMI, даже если в Pod не используется _AAD-Pod-Identity_.
_Азуреподидентитексцептион_ КУСТОМРЕСАУРЦЕДЕФИНИТИОН (CRD) можно настроить для информирования _AAD-Pod_ о том, что любые запросы к конечной точке метаданных, которые берутся из Pod, соответствующей МЕТКАМ, указанным в CRD, должны быть прокси без какой-либо обработки в NMI.

#### <a name="resolution"></a>Решение

Исключите системные модули, имеющие `kubernetes.azure.com/managedby: aks` метку в пространстве имен _KUBE-System_ в _AAD-Pod-Identity_ , настроив _азуреподидентитексцептион_ CRD.

Дополнительные сведения см. [в разделе Отключение удостоверения pod Azure Active Directory (Azure AD) для конкретного Pod или приложения](https://azure.github.io/aad-pod-identity/docs/configure/application_exception).

Чтобы настроить исключение, выполните приведенный ниже пример.

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: mic-exception
  namespace: default
spec:
  podLabels:
    app: mic
    component: mic
---
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: aks-addon-exception
  namespace: kube-system
spec:
  podLabels:
    kubernetes.azure.com/managedby: aks
```

### <a name="scenario-the-resource-provider-isnt-registered"></a>Сценарий: поставщик ресурсов не зарегистрирован

#### <a name="issue"></a>Проблема

Надстройка может получить доступ к конечной точке службы политики Azure, но в журналах надстроек отображается одна из следующих ошибок:

- `The resource provider 'Microsoft.PolicyInsights' is not registered in subscription '{subId}'. See
  https://aka.ms/policy-register-subscription for how to register subscriptions.`

- `policyinsightsdataplane.BaseClient#CheckDataPolicyCompliance: Failure responding to request:
  StatusCode=500 -- Original Error: autorest/azure: Service returned an error. Status=500
  Code="InternalServerError" Message="Encountered an internal server error.`

#### <a name="cause"></a>Причина

Поставщик ресурсов Microsoft. Полициинсигхтс не зарегистрирован. Он должен быть зарегистрирован для надстройки, чтобы получать определения политик и возвращать данные о соответствии.

#### <a name="resolution"></a>Решение

Зарегистрируйте поставщик ресурсов Microsoft. Полициинсигхтс в подписке кластера. Инструкции см. [в разделе Регистрация поставщика ресурсов](../../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).

### <a name="scenario-the-subscription-is-disabled"></a>Сценарий: Подписка отключена.

#### <a name="issue"></a>Проблема

Надстройка может получить доступ к конечной точке службы политики Azure, но отображается следующая ошибка:

`The subscription '{subId}' has been disabled for azure data-plane policy. Please contact support.`

#### <a name="cause"></a>Причина

Эта ошибка означает, что подписка была определена как проблематичная, а флаг функции `Microsoft.PolicyInsights/DataPlaneBlocked` был добавлен для блокировки подписки.

#### <a name="resolution"></a>Решение

Чтобы исследовать и устранить эту проблему, [обратитесь](mailto:azuredg@microsoft.com)к специализированной группе.

## <a name="next-steps"></a>Дальнейшие действия

Если проблема не указана в этой статье или ее невозможно устранить, получите поддержку, посетив один из следующих каналов:

- Получите ответы от экспертов по [Microsoft Q&а](/answers/topics/azure-policy.html).
- Обратитесь за помощью в Твиттер — [@AzureSupport](https://twitter.com/azuresupport). Этот официальный Microsoft Azure ресурс в Twitter поможет улучшить взаимодействие с клиентами, подключив сообщество Azure к правильным ответам, поддержке и экспертам.
- Если вам по-прежнему нужна помощь, перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Отправить запрос в службу поддержки**.
