---
title: Получение данных о соответствии политике
description: Соответствие определяется оценками и действиями Политики Azure. Узнайте о том, как получить подробные сведения о соответствии для ресурсов Azure.
ms.date: 03/16/2021
ms.topic: how-to
ms.openlocfilehash: cdd23d685750fb8a5d3803f4b6030e7e67bbddce
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104598547"
---
# <a name="get-compliance-data-of-azure-resources"></a>Получение данных о соответствии для ресурсов Azure

Одним из наибольших преимуществ Политики Azure является анализ и контроль, который эта служба выполняет над ресурсами в подписке или [группе управления](../../management-groups/overview.md) подписками. Этот контроль может быть реализован разными способами, такими как предотвращение создания ресурсов в неправильном месте, принудительное использование общих и согласованных тегов, а также аудит существующих ресурсов на предмет соответствующих конфигураций и параметров. Во всех случаях Политика Azure создает данные, которые позволяют определить состояние соответствия среды.

Существует несколько способов доступа к информации о соответствии, которая создается при назначении политик и инициатив:

- с помощью [портала Azure](#portal);
- с помощью скриптов [командной строки](#command-line).

Прежде чем переходить к способам создания отчетов о соответствии, давайте рассмотрим, когда обновляются данные о соответствии, а также с какой частотой и какие события запускают цикл оценки.

> [!WARNING]
> Если состояние соответствия указывается как **не зарегистрировано**, убедитесь, что поставщик ресурсов **Microsoft. полициинсигхтс** зарегистрирован и что пользователь имеет соответствующие разрешения на управление доступом на основе РОЛЕЙ Azure (Azure RBAC), как описано в разделе [разрешения RBAC в политике](../overview.md#azure-rbac-permissions-in-azure-policy)Azure.

## <a name="evaluation-triggers"></a>Триггеры оценки

Результаты завершенного цикла оценки отражаются в поставщике ресурсов `Microsoft.PolicyInsights` с помощью операций `PolicyStates` и `PolicyEvents`. Дополнительные сведения об операциях REST API Azure Policy Insights см. [здесь](/rest/api/policy/).

Оценка назначенных политик и инициатив выполняется в результате разных событий:

- Определенной области назначается новая политика или инициатива. Применение назначения к заданной области занимает около 30 минут. После применения цикла оценки начинается для ресурсов в этой области по отношению к вновь назначенной политике или инициативе и в зависимости от эффектов, используемых политикой или инициативой, ресурсы помечены как соответствующие, несоответствующие или исключенные. Оценка масштабной политики или инициативы относительно большой области ресурсов может занять много времени. Поэтому сведений об ожидаемом времени завершения цикла оценки нет. После этого обновленные результаты оценки соответствия становятся доступными на портале и в пакетах SDK.

- Политика или инициатива, уже назначенная определенной области, является обновленной. Цикл и время оценки в этом сценарии аналогичны новому назначению области.

- Ресурс разворачивается или обновляется в области с назначением с помощью Azure Resource Manager, REST API или поддерживаемого пакета SDK. В этом сценарии событие действия политики (присоединение, аудит, отказ, развертывание) и данные о состоянии соответствия отдельного ресурса становятся доступными на портале и в пакетах SDK примерно через 15 минут. Это событие не вызывает оценку других ресурсов.

- [Исключение политики](../concepts/exemption-structure.md) создается, обновляется или удаляется. В этом сценарии соответствующее назначение оценивается для определенной области исключения.

- Стандартный цикл оценки соответствия. Каждые 24 часа назначения автоматически повторно оцениваются. Оценка масштабной политики или инициативы для большой области ресурсов может занять много времени. Поэтому сведений об ожидаемом времени завершения цикла оценки нет. После этого обновленные результаты оценки соответствия становятся доступными на портале и в пакетах SDK.

- Управляемый ресурс обновляет поставщик ресурсов [гостевой конфигурации](../concepts/guest-configuration.md), добавляя сведения о соответствии.

- Проверка по запросу.

### <a name="on-demand-evaluation-scan"></a>Проверка оценки по запросу

Оценочное сканирование подписки или группы ресурсов можно начать с Azure CLI, Azure PowerShell, вызова REST API или с помощью [действия "Проверка соответствия политики Azure](https://github.com/marketplace/actions/azure-policy-compliance-scan)" в GitHub.
Эта проверка является асинхронным процессом.

#### <a name="on-demand-evaluation-scan---github-action"></a>Проверка по требованию — действие GitHub

Используйте [действие проверки соответствия политики Azure](https://github.com/marketplace/actions/azure-policy-compliance-scan) для активации проверки по требованию в [рабочем процессе GitHub](https://docs.github.com/actions/configuring-and-managing-workflows/configuring-a-workflow#about-workflows) на одном или нескольких ресурсах, группах ресурсов или подписках, а также на основе рабочего процесса, основанного на состоянии соответствия ресурсов. Вы также можете настроить выполнение рабочего процесса в запланированное время, чтобы получить актуальное состояние соответствия в удобное время. При необходимости это действие GitHub может формировать отчет о состоянии соответствия проверенных ресурсов для дальнейшего анализа или архивирования.

Следующий пример выполняет проверку соответствия для одной подписки. 

```yaml
on:
  schedule:    
    - cron:  '0 8 * * *'  # runs every morning 8am
jobs:
  assess-policy-compliance:    
    runs-on: ubuntu-latest
    steps:         
    - name: Login to Azure
      uses: azure/login@v1
      with:
        creds: ${{secrets.AZURE_CREDENTIALS}} 

    
    - name: Check for resource compliance
      uses: azure/policy-compliance-scan@v0
      with:
        scopes: |
          /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Дополнительные сведения и примеры рабочих процессов см. в разделе [действие GitHub для репозитория проверки соответствия политики Azure](https://github.com/Azure/policy-compliance-scan).

#### <a name="on-demand-evaluation-scan---azure-cli"></a>Сканирование оценки по требованию — Azure CLI

Проверка соответствия запускается с помощью команды [AZ Policy State-Scan](/cli/azure/policy/state#az_policy_state_trigger_scan) .

По умолчанию `az policy state trigger-scan` запускает оценку для всех ресурсов в текущей подписке. Чтобы начать оценку определенной группы ресурсов, используйте параметр **Resource-Group** . Следующий пример запускает проверку соответствия в текущей подписке для группы ресурсов _MyRG_:

```azurecli-interactive
az policy state trigger-scan --resource-group "MyRG"
```

Можно не дожидаться завершения асинхронного процесса, прежде чем продолжить работу с параметром **без ожидания** .

#### <a name="on-demand-evaluation-scan---azure-powershell"></a>Проверка оценки по запросу (Azure PowerShell)

Проверка соответствия запускается командлетом [Start-AzPolicyComplianceScan](/powershell/module/az.policyinsights/start-azpolicycompliancescan).

По умолчанию `Start-AzPolicyComplianceScan` запускает оценку для всех ресурсов в текущей подписке. Чтобы выполнить оценку для определенной группы ресурсов, укажите параметр **ResourceGroupName**. Следующий пример запускает проверку соответствия в текущей подписке для группы ресурсов _MyRG_:

```azurepowershell-interactive
Start-AzPolicyComplianceScan -ResourceGroupName 'MyRG'
```

Можно сделать так, чтобы PowerShell ожидал завершения асинхронного вызова перед выдачей выходных данных или выполнял его в фоновом режиме в качестве [задания](/powershell/module/microsoft.powershell.core/about/about_jobs). Чтобы PowerShell выполнял проверку соответствия в фоновом режиме, укажите параметр **AsJob** и задайте значение для объекта, например `$job` в следующем примере:

```azurepowershell-interactive
$job = Start-AzPolicyComplianceScan -AsJob
```

Чтобы проверить состояние этого задания, получите свойства объекта `$job`. Задание имеет тип `Microsoft.Azure.Commands.Common.AzureLongRunningJob`. Используйте `Get-Member` для объекта `$job`, чтобы просмотреть доступные свойства и методы.

Пока выполняется проверка соответствия, объект `$job` возвращает следующие результаты:

```azurepowershell-interactive
$job

Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
2      Long Running O… AzureLongRunni… Running       True            localhost            Start-AzPolicyCompliance…
```

После завершения проверки соответствия свойство **State** получает значение _Completed_ (Завершено).

#### <a name="on-demand-evaluation-scan---rest"></a>Проверка оценки по запросу (REST)

Так как процесс является асинхронным, запускающая его конечная точка REST возвращает ответ, не дожидаясь его завершения. Вместо этого она предоставляет URI для получения состояния выполняемой оценки.

В каждом универсальном коде ресурса (URI) REST API есть переменные, которые необходимо заменить собственными значениями:

- `{YourRG}` — замените это значение именем своей группы ресурсов.
- `{subscriptionId}` — замените это значение идентификатором своей подписки.

Сканирование поддерживает оценку ресурсов в подписке или группе ресурсов. Запустите сканирование для области через команду **POST** REST API, используя следующие структуры универсального кода ресурса (URI):

- Подписка

  ```http
  POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/triggerEvaluation?api-version=2019-10-01
  ```

- Группа ресурсов

  ```http
  POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{YourRG}/providers/Microsoft.PolicyInsights/policyStates/latest/triggerEvaluation?api-version=2019-10-01
  ```

Этот вызов возвращает состояние **202 — принято**. В заголовок ответа включается свойство **Location** в следующем формате:

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/asyncOperationResults/{ResourceContainerGUID}?api-version=2019-10-01
```

`{ResourceContainerGUID}` создается для запрошенной области статически. Если эта область уже выполняет проверку по требованию, новое сканирование не запускается. Вместо этого новому запросу для проверки состояния предоставляется тот же URI, который указан в свойстве **Location** (`{ResourceContainerGUID}`). Команда **GET** REST API, отправленная на URI, указанный в свойстве **Location**, возвращает состояние **202 — принято**, пока продолжается выполнение оценки. Когда оценочное сканирование завершается, возвращается состояние **200 — ОК**. Вот пример текста в формате JSON, который передается в ответе после завершения проверки:

```json
{
    "status": "Succeeded"
}
```

#### <a name="on-demand-evaluation-scan---visual-studio-code"></a>Сканирование оценки по требованию — Visual Studio Code

Расширение политики Azure для Visual Studio Code может выполнять пробное сканирование определенного ресурса. Эта проверка является синхронным процессом, в отличие от методов Azure PowerShell и RESTFUL.
Дополнительные сведения и инструкции см. в разделе [Оценка по запросу с помощью расширения VS Code](./extension-for-vscode.md#on-demand-evaluation-scan).

## <a name="how-compliance-works"></a>Как работает соответствие

В назначении ресурс не **соответствует требованиям** , если он не соответствует правилам политики или инициативы и не _исключен_. В следующей таблице показано, как действуют разные политики в сочетании с оценкой условий для определения итогового состояния соответствия.

| Состояние ресурса | Действие | Вычисление политики | Состояние соответствия |
| --- | --- | --- | --- |
| Новая или обновленная | Audit, Modify, AuditIfNotExist | True | Не соответствует |
| Новая или обновленная | Audit, Modify, AuditIfNotExist | False | Соответствует |
| Exists | Deny, Audit, Append, Modify, DeployIfNotExist, AuditIfNotExist | True | Не соответствует |
| Exists | Deny, Audit, Append, Modify, DeployIfNotExist, AuditIfNotExist | False | Соответствует |

> [!NOTE]
> Для установки состояния несоответствия эффекты DeployIfNotExist и AuditIfNotExist требуют, чтобы оператор IF возвращал значение TRUE, а условие существования возвращало FALSE. Когда установлено значение TRUE, условие IF запускает оценку условия существования для связанных ресурсов.

Например, предположим, что у вас есть группа ресурсов ContosoRG с некоторыми учетными записями хранения (выделены красным цветом), которые доступны в общедоступных сетях.

:::image type="complex" source="../media/getting-compliance-data/resource-group01.png" alt-text="Схема учетных записей хранения, доступных для общедоступных сетей в группе ресурсов Contoso R G." border="false":::
   На схеме показаны изображения для пяти учетных записей хранения в группе ресурсов Contoso R G.  Учетные записи хранения один и три являются синими, а учетные записи хранения 2, четыре и пять — красным.
:::image-end:::

В этом примере необходимо опасаться угроз безопасности. Теперь, когда вы создали назначение политики, оно оценивается для всех включаемых и неисключенных учетных записей хранения в группе ресурсов ContosoRG. Оно проверяет три несоответствующие учетные записи хранения, соответственно меняя их состояние на **Не соответствует**.

:::image type="complex" source="../media/getting-compliance-data/resource-group03.png" alt-text="Схема соответствия учетной записи хранения в группе ресурсов Contoso R G." border="false":::
   На схеме показаны изображения для пяти учетных записей хранения в группе ресурсов Contoso R G. В учетных записях хранения один и три теперь помечены зеленой галочкой, а у учетных записей хранения 2, 4 и 5 теперь есть красные предупреждения под ними.
:::image-end:::

Кроме **соответствующих** и **не соответствующих** политике, политики и ресурсы имеют четыре других состояния:

- **Исключение**: ресурс находится в области назначения, но имеет [определенное исключение](../concepts/exemption-structure.md).
- **Конфликт**: существует два или более определений политик с конфликтующими правилами. Например, два определения добавляют один и тот же тег с разными значениями.
- **Не запущен**. Цикл оценки не запущен для политики или ресурса.
- **Не зарегистрирован**. Поставщик ресурсов Политики Azure не зарегистрирован или выполнившая вход учетная запись не имеет разрешения на чтение данных о соответствии.

Для определения совпадения ресурса в политике Azure используются поля **тип**, **имя** **или тип в** определении. Если ресурс соответствует, он считается применимым и имеет состояние " **соответствует**", " **не соответствует**" или " **исключено**". Если либо **тип**, **имя**, либо **тип** являются единственным свойством в определении, все включенные и Неисключенные ресурсы считаются применимыми и оцениваются.

Процент соответствия определяется путем разделения **соответствующих** и **исключаемых** ресурсов по _общему количеству ресурсов_. _Общее число ресурсов_ определяется как сумма **соответствующих**, **несоответствующих**, **исключаемых** и **конфликтующих** ресурсов. Общие номера соответствия — это сумма отдельных ресурсов, **соответствующих** или **исключаемых** на сумму всех различных ресурсов. На приведенном ниже рисунке показано 20 различных ресурсов, которые являются применимыми, и только один является **несоответствующим**.
Общее соответствие ресурсов составляет 95 % (19 из 20).

:::image type="content" source="../media/getting-compliance-data/simple-compliance.png" alt-text="Снимок экрана со сведениями о соответствии политик на странице соответствия требованиям." border="false":::

> [!NOTE]
> Соответствие нормативным требованиям в политике Azure является предварительной версией функции. Свойства соответствия между пакетом SDK и страницами на портале отличаются для включенных инициатив. Дополнительные сведения см. в разделе [соответствие нормативным требованиям](../concepts/regulatory-compliance.md) .

## <a name="portal"></a>Портал

На портале Azure можно ознакомиться с процессом визуализации и оценки состояния соответствия в вашей среде. На странице **Политика** параметр **Обзор** предоставляет сведения о соответствии для доступных областей в рамках соответствия для политик и инициатив. В дополнение к состоянию соответствия и числу на каждое назначение, в нем содержится диаграмма, отображающая соответствие за последние семь дней. На странице **Соответствие** содержится большая часть этой информации (за исключением диаграммы), а также предоставляются дополнительные возможности фильтрации и сортировки.

:::image type="content" source="../media/getting-compliance-data/compliance-page.png" alt-text="Снимок экрана со страницей соответствия, параметрами фильтрации и подробными сведениями." border="false":::

Так как политика или инициатива может быть назначена разным областям, в таблице предусмотрена область для каждого назначения и тип определения, назначенный этой области. Здесь также отображается число несоответствующих ресурсов и политик для каждого назначения. Выбор политики или инициативы в таблице более подробно рассматривает соответствие требованиям для данного назначения.

:::image type="content" source="../media/getting-compliance-data/compliance-details.png" alt-text="Снимок экрана со страницей сведений о соответствии, включая количество и сведения о соответствии ресурсов." border="false":::

В списке ресурсов на вкладке **Соответствие ресурсов** отображается состояние оценки существующих ресурсов для текущего назначения. На вкладке по умолчанию установлено значение **Не соответствует**, но можно выполнить фильтрацию.
События (Добавление, аудит, отклонение, развертывание, изменение), активируемые запросом на создание ресурса, отображаются на вкладке **события** .

> [!NOTE]
> В политике ядра AKS отображается ресурс группы ресурсов.

:::image type="content" source="../media/getting-compliance-data/compliance-events.png" alt-text="Снимок экрана вкладки &quot;события&quot; на странице сведений о соответствии." border="false":::

<a name="component-compliance"></a> Для ресурсов [режима поставщика ресурсов](../concepts/definition-structure.md#resource-provider-modes) на вкладке **соответствие ресурсов** выберите ресурс или щелкните правой кнопкой мыши строку и выберите **Просмотреть сведения о соответствии** , чтобы открыть сведения о соответствии компонентов. На этой странице также представлены вкладки для просмотра назначенных этому ресурсу политик, событий, событий компонентов и журнала изменений.

:::image type="content" source="../media/getting-compliance-data/compliance-components.png" alt-text="Снимок экрана с вкладкой соответствия компонентов и сведениями о соответствии для назначения режима поставщика ресурсов." border="false":::

Вернитесь на страницу соответствия ресурса, щелкните правой кнопкой мыши в строке события, о котором нужно собрать подробные сведения, и выберите **Показать журналы действий**. Откроется страница журнала действий с предварительно отфильтрованными результатами поиска, где отображаются подробные сведения о назначениях и событиях. В журнале действий содержатся дополнительный контекст и сведения об этих событиях.

:::image type="content" source="../media/getting-compliance-data/compliance-activitylog.png" alt-text="Снимок экрана журнала действий для действий и оценок в политике Azure." border="false":::

### <a name="understand-non-compliance"></a>Понимание причин несоответствия

Если ресурс определен как **не соответствующий требованиям**, причины могут быть разными. Чтобы определить причину **несоответствия** ресурса или найти соответствующее изменение, воспользуйтесь [этими инструкциями](./determine-non-compliance.md).

## <a name="command-line"></a>Командная строка

Те же сведения, которые доступны на портале, можно получить с помощью REST API (включая [ARMClient](https://github.com/projectkudu/ARMClient)), Azure PowerShell и Azure CLI. Полные сведения о REST API см. в справочнике по [политикам Azure](/rest/api/policy/) . На страницах справки по REST API есть зеленая кнопка "Попробовать" для каждой операции, которую можно опробовать непосредственно в браузере.

Используйте ARMClient или аналогичное средство для управления аутентификацией в Azure для примеров с REST API.

### <a name="summarize-results"></a>Суммирование результатов

С помощью REST API формирование сводных данных можно выполнить по контейнеру, определению или назначению. Ниже приведен пример суммирования на уровне подписки с использованием функции [Summarize For Subscription](/rest/api/policy/policystates/summarizeforsubscription) (Суммировать для подписки) в Azure Policy Insight:

```http
POST https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/summarize?api-version=2019-10-01
```

В выходных данных будет представлена сводка по подписке. В следующем примере вывода сводные данные о соответствии содержатся в разделах **value.results.nonCompliantResources** и **value.results.nonCompliantPolicies**. Этот запрос предоставляет дополнительные сведения, включая каждое назначение, входящее в число несоответствующих, а также сведения об определении для каждого назначения. Каждый объект политики в иерархии предоставляет идентификатор **queryResultsUri**, который позволяет получить дополнительные сведения на этом уровне.

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary/$entity",
        "results": {
            "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant'",
            "nonCompliantResources": 15,
            "nonCompliantPolicies": 1
        },
        "policyAssignments": [{
            "policyAssignmentId": "/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77",
            "policySetDefinitionId": "",
            "results": {
                "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77'",
                "nonCompliantResources": 15,
                "nonCompliantPolicies": 1
            },
            "policyDefinitions": [{
                "policyDefinitionReferenceId": "",
                "policyDefinitionId": "/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
                "effect": "deny",
                "results": {
                    "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'",
                    "nonCompliantResources": 15
                }
            }]
        }]
    }]
}
```

### <a name="query-for-resources"></a>Запрос ресурсов

В примере выше значение **value.policyAssignments.policyDefinitions.results.queryResultsUri** содержало пример Uri для всех несоответствующих ресурсов для заданного определения политики. При просмотре значения **$Filter** ComplianceState равно (EQ) в значение "не соответствует", для определения политики задается полициассигнментид, а затем полицидефинитионид. PolicyAssignmentId включается в фильтр, так как PolicyDefinitionId может существовать в нескольких назначениях политик или инициатив с разными областями. Указав как PolicyAssignmentId, так и PolicyDefinitionId, мы можем явно определить ожидаемые результаты. Ранее мы использовали значение **latest** для параметра PolicyStates, что автоматически делает временное окно **from** и **to** равным последним 24 часам.

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2018-05-18 04:28:22Z&$to=2018-05-19 04:28:22Z&$filter=ComplianceState eq 'NonCompliant' and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/resourcegroups/rg-tags/providers/microsoft.authorization/policyassignments/37ce239ae4304622914f0c77' and PolicyDefinitionId eq '/providers/microsoft.authorization/policydefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62'
```

Приведенный ниже пример усечен к одному несоответствующему ресурсу для краткости. Подробный ответ имеет несколько фрагментов данных о ресурсе, политике или инициативе и назначении. Здесь вы также можете увидеть, какие параметры назначения были переданы в определение политики.

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest",
    "@odata.count": 15,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
        "timestamp": "2018-05-19T04:41:09Z",
        "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Compute/virtualMachines/linux",
        "policyAssignmentId": "/subscriptions/{subscriptionId}/resourceGroups/rg-tags/providers/Microsoft.Authorization/policyAssignments/37ce239ae4304622914f0c77",
        "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "effectiveParameters": "",
        "ComplianceState": "NonCompliant",
        "subscriptionId": "{subscriptionId}",
        "resourceType": "/Microsoft.Compute/virtualMachines",
        "resourceLocation": "westus2",
        "resourceGroup": "RG-Tags",
        "resourceTags": "tbd",
        "policyAssignmentName": "37ce239ae4304622914f0c77",
        "policyAssignmentOwner": "tbd",
        "policyAssignmentParameters": "{\"tagName\":{\"value\":\"costCenter\"},\"tagValue\":{\"value\":\"Contoso-Test\"}}",
        "policyAssignmentScope": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags",
        "policyDefinitionName": "1e30110a-5ceb-460c-a204-c1c3969c6d62",
        "policyDefinitionAction": "deny",
        "policyDefinitionCategory": "tbd",
        "policySetDefinitionId": "",
        "policySetDefinitionName": "",
        "policySetDefinitionOwner": "",
        "policySetDefinitionCategory": "",
        "policySetDefinitionParameters": "",
        "managementGroupIds": "",
        "policyDefinitionReferenceId": ""
    }]
}
```

### <a name="view-events"></a>Просмотр событий

После создания или обновления ресурса создается результат оценки политики. Такие результаты называются _событиями политики_. Чтобы просмотреть последние события политики, связанные с подпиской, используйте следующий URI.

```http
https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/default/queryResults?api-version=2019-10-01
```

Результаты должны выглядеть примерно так:

```json
{
    "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default",
    "@odata.count": 1,
    "value": [{
        "@odata.id": null,
        "@odata.context": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyEvents/$metadata#default/$entity",
        "NumAuditEvents": 16
    }]
}
```

Дополнительные сведения о запросах событий политик см. в справочной статье по [событиям Политики Azure](/rest/api/policy/policyevents).

### <a name="azure-cli"></a>Azure CLI

Группа команд [Azure CLI](/cli/azure/what-is-azure-cli) для политики Azure охватывает большинство операций, доступных в оставшейся или Azure PowerShell. Полный список доступных команд см. в разделе [Azure CLI — общие сведения о политике Azure](/cli/azure/policy).

Пример Получение сводки состояния для приоритетной назначенной политики с наибольшим числом несоответствующих ресурсов.

```azurecli-interactive
az policy state summarize --top 1
```

Верхняя часть ответа выглядит, как в следующем примере:

```json
{
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#summary/$entity",
    "odataid": null,
    "policyAssignments": [{
            "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
            "policyDefinitions": [{
                "effect": "audit",
                "policyDefinitionGroupNames": [
                    ""
                ],
                "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
                "policyDefinitionReferenceId": "",
                "results": {
                    "nonCompliantPolicies": null,
                    "nonCompliantResources": 398,
                    "policyDetails": [{
                        "complianceState": "noncompliant",
                        "count": 1
                    }],
                    "policyGroupDetails": [{
                        "complianceState": "noncompliant",
                        "count": 1
                    }],
                    "queryResultsUri": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/latest/queryResults?api-version=2019-10-01&$from=2020-07-14 14:01:22Z&$to=2020-07-15 14:01:22Z and PolicyAssignmentId eq '/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8' and PolicyDefinitionId eq '/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a'",
                    "resourceDetails": [{
                            "complianceState": "noncompliant",
                            "count": 398
                        },
                        {
                            "complianceState": "compliant",
                            "count": 4
                        }
                    ]
                }
            }],
    ...
```

Пример Получение записи состояния для последнего оцененного ресурса (по умолчанию — по меткам времени в порядке убывания).

```azurecli-interactive
az policy state list --top 1
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "Compliant",
    "effectiveParameters": "",
    "isCompliant": true,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/securitycenterbuiltin",
    "policyAssignmentName": "SecurityCenterBuiltIn",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "auditifnotexists",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/providers/microsoft.authorization/policydefinitions/aa633080-8b72-40c4-a2d7-d00c03e80bed",
    "policyDefinitionName": "aa633080-8b72-40c4-a2d7-d00c03e80bed",
    "policyDefinitionReferenceId": "identityenablemfaforownerpermissionsmonitoring",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "security center",
    "policySetDefinitionId": "/providers/Microsoft.Authorization/policySetDefinitions/1f3afdf9-d0c9-4c3d-847f-89da613e70a8",
    "policySetDefinitionName": "1f3afdf9-d0c9-4c3d-847f-89da613e70a8",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "",
    "resourceId": "/subscriptions/{subscriptionId}",
    "resourceLocation": "",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Resources/subscriptions",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.903433+00:00"
  }
]
```

Пример Получение сведений для всех несоответствующих ресурсов виртуальной сети.

```azurecli-interactive
az policy state list --filter "ResourceType eq 'Microsoft.Network/virtualNetworks'"
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "NonCompliant",
    "effectiveParameters": "",
    "isCompliant": false,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
    "policyAssignmentName": "e0704696df5e4c3c81c873e8",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "audit",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionName": "2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionReferenceId": "",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "",
    "policySetDefinitionId": "",
    "policySetDefinitionName": "",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "RG-Tags",
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Microsoft.Network/virtualNetworks/RG-Tags-vnet",
    "resourceLocation": "westus2",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Network/virtualNetworks",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.901911+00:00"
  }
]
```

Пример Получение событий, связанных с несоответствующими ресурсами виртуальной сети и произошедших после указанной даты.

```azurecli-interactive
az policy state list --filter "ResourceType eq 'Microsoft.Network/virtualNetworks'" --from '2020-07-14T00:00:00Z'
```

```json
[
  {
    "complianceReasonCode": "",
    "complianceState": "NonCompliant",
    "effectiveParameters": "",
    "isCompliant": false,
    "managementGroupIds": "{managementgroupId}",
    "odatacontext": "https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.PolicyInsights/policyStates/$metadata#latest/$entity",
    "odataid": null,
    "policyAssignmentId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policyassignments/e0704696df5e4c3c81c873e8",
    "policyAssignmentName": "e0704696df5e4c3c81c873e8",
    "policyAssignmentOwner": "tbd",
    "policyAssignmentParameters": "",
    "policyAssignmentScope": "/subscriptions/{subscriptionId}",
    "policyAssignmentVersion": "",
    "policyDefinitionAction": "audit",
    "policyDefinitionCategory": "tbd",
    "policyDefinitionGroupNames": [
      ""
    ],
    "policyDefinitionId": "/subscriptions/{subscriptionId}/providers/microsoft.authorization/policydefinitions/2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionName": "2e3197b6-1f5b-4b01-920c-b2f0a7e9b18a",
    "policyDefinitionReferenceId": "",
    "policyDefinitionVersion": "",
    "policyEvaluationDetails": null,
    "policySetDefinitionCategory": "",
    "policySetDefinitionId": "",
    "policySetDefinitionName": "",
    "policySetDefinitionOwner": "",
    "policySetDefinitionParameters": "",
    "policySetDefinitionVersion": "",
    "resourceGroup": "RG-Tags",
    "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Microsoft.Network/virtualNetworks/RG-Tags-vnet",
    "resourceLocation": "westus2",
    "resourceTags": "tbd",
    "resourceType": "Microsoft.Network/virtualNetworks",
    "subscriptionId": "{subscriptionId}",
    "timestamp": "2020-07-15T08:37:07.901911+00:00"
  }
]
```

### <a name="azure-powershell"></a>Azure PowerShell

Модуль Azure PowerShell для Политики Azure доступен в коллекции PowerShell как [Az.PolicyInsights](https://www.powershellgallery.com/packages/Az.PolicyInsights). Используя PowerShellGet, вы можете установить модуль с помощью `Install-Module -Name Az.PolicyInsights` (установите последнюю версию [Azure PowerShell](/powershell/azure/install-az-ps)):

```azurepowershell-interactive
# Install from PowerShell Gallery via PowerShellGet
Install-Module -Name Az.PolicyInsights

# Import the downloaded module
Import-Module Az.PolicyInsights

# Login with Connect-AzAccount if not using Cloud Shell
Connect-AzAccount
```

Модуль содержит следующие командлеты.

- `Get-AzPolicyStateSummary`
- `Get-AzPolicyState`
- `Get-AzPolicyEvent`
- `Get-AzPolicyRemediation`
- `Remove-AzPolicyRemediation`
- `Start-AzPolicyRemediation`
- `Stop-AzPolicyRemediation`

Пример Получение сводки состояния для приоритетной назначенной политики с наибольшим числом несоответствующих ресурсов.

```azurepowershell-interactive
PS> Get-AzPolicyStateSummary -Top 1

NonCompliantResources : 15
NonCompliantPolicies  : 1
PolicyAssignments     : {/subscriptions/{subscriptionId}/resourcegroups/RG-Tags/providers/micros
                        oft.authorization/policyassignments/37ce239ae4304622914f0c77}
```

Пример Получение записи состояния для последнего оцененного ресурса (по умолчанию — по меткам времени в порядке убывания).

```azurepowershell-interactive
PS> Get-AzPolicyState -Top 1

Timestamp                  : 5/22/2018 3:47:34 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/networkInterfaces/linux316
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/networkInterfaces
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

Пример Получение сведений для всех несоответствующих ресурсов виртуальной сети.

```azurepowershell-interactive
PS> Get-AzPolicyState -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'"

Timestamp                  : 5/22/2018 4:02:20 PM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : westus2
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
```

Пример. получение событий, связанных с несоответствующими ресурсами виртуальной сети, которые произошли после определенной даты, преобразование в объект CSV и экспорт в файл.

```azurepowershell-interactive
$policyEvents = Get-AzPolicyEvent -Filter "ResourceType eq '/Microsoft.Network/virtualNetworks'" -From '2020-09-19'
$policyEvents | ConvertTo-Csv | Out-File 'C:\temp\policyEvents.csv'
```

Выходные данные `$policyEvents` объекта выглядят как следующие выходные данные:

```output
Timestamp                  : 9/19/2020 5:18:53 AM
ResourceId                 : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Network/virtualNetworks/RG-Tags-vnet
PolicyAssignmentId         : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags/providers/Mi
                             crosoft.Authorization/policyAssignments/37ce239ae4304622914f0c77
PolicyDefinitionId         : /providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62
ComplianceState            : NonCompliant
SubscriptionId             : {subscriptionId}
ResourceType               : /Microsoft.Network/virtualNetworks
ResourceLocation           : eastus
ResourceGroup              : RG-Tags
ResourceTags               : tbd
PolicyAssignmentName       : 37ce239ae4304622914f0c77
PolicyAssignmentOwner      : tbd
PolicyAssignmentParameters : {"tagName":{"value":"costCenter"},"tagValue":{"value":"Contoso-Test"}}
PolicyAssignmentScope      : /subscriptions/{subscriptionId}/resourceGroups/RG-Tags
PolicyDefinitionName       : 1e30110a-5ceb-460c-a204-c1c3969c6d62
PolicyDefinitionAction     : deny
PolicyDefinitionCategory   : tbd
TenantId                   : {tenantId}
PrincipalOid               : {principalOid}
```

Поле **PrincipalOid** может использоваться для получения определенного пользователя с помощью командлета Azure PowerShell `Get-AzADUser`. Замените **{principalOid}** ответом, полученным в предыдущем примере.

```azurepowershell-interactive
PS> (Get-AzADUser -ObjectId {principalOid}).DisplayName
Trent Baker
```

## <a name="azure-monitor-logs"></a>Журналы Azure Monitor

Если у вас есть [Рабочая область log Analytics](../../../azure-monitor/logs/log-query-overview.md) с `AzureActivity` [аналитика журнала действий решением](../../../azure-monitor/essentials/activity-log.md) , привязанным к вашей подписке, можно также просмотреть результаты несоответствия на основе оценки новых и обновленных ресурсов с помощью простых запросов Kusto и `AzureActivity` таблицы. С учетом подробных сведений о несоответствии в журналах Azure Monitor вы также можете настроить оповещения для отслеживания несоответствия.

:::image type="content" source="../media/getting-compliance-data/compliance-loganalytics.png" alt-text="Снимок экрана Azure Monitor журналов, демонстрирующих действия политики Azure в таблице AzureActivity." border="false":::

## <a name="next-steps"></a>Дальнейшие действия

- Рассмотрите [примеры для службы "Политика Azure"](../samples/index.md).
- Изучите статью о [структуре определения Политики Azure](../concepts/definition-structure.md).
- Изучите [сведения о действии политик](../concepts/effects.md).
- Узнайте о [программном создании политик](programmatically-create.md).
- Узнайте, как [исправлять несоответствующие ресурсы](remediate-resources.md).
- Дополнительные сведения о группе управления см. в статье [Упорядочивание ресурсов с помощью групп управления Azure](../../management-groups/overview.md).
