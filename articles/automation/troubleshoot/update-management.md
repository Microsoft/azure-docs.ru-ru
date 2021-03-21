---
title: Устранение неполадок с Управлением обновлениями службы автоматизации Azure
description: Эта статья содержит сведения об устранении неполадок с Управлением обновлениями службы автоматизации Azure.
services: automation
ms.subservice: update-management
ms.date: 01/13/2021
ms.topic: troubleshooting
ms.openlocfilehash: c16b032502401b633532ab0fcf9518aa85a1b8d6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100579738"
---
# <a name="troubleshoot-update-management-issues"></a>Устранение неполадок с Управлением обновлениями

В этой статье обсуждаются проблемы, с которыми можно столкнуться при развертывании решения "Управление обновлениями" на компьютерах. Агент устранения неполадок определяет первоначальную проблему для агента гибридной рабочей роли Runbook. Дополнительные сведения о средстве устранения неполадок см. в статьях [Устранение неполадок с агентом обновления Windows](update-agent-issues.md) и [Устранение неполадок с агентом обновления Linux](update-agent-issues-linux.md). Сведения о других проблемах при развертывании компонентов см. в статье [Устранение неполадок с развертыванием компонентов](onboarding.md).

>[!NOTE]
>Если при развертывании Управление обновлениями на компьютере Windows возникают проблемы, откройте Просмотр событий Windows и проверьте журнал событий **Operations Manager** в разделе **журналы приложений и служб** на локальном компьютере. Найдите события с идентификатором 4502 и сведения о событии, содержащие `Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent`.

## <a name="scenario-linux-updates-shown-as-pending-and-those-installed-vary"></a><a name="updates-linux-installed-different"></a>Сценарий: обновления Linux отображаются как ожидающие, а установленные — разные

### <a name="issue"></a>Проблема

Для компьютера Linux Управление обновлениями отображает конкретные обновления, доступные в разделе **Безопасность** классификации и **другие**. Но если на компьютере выполняется расписание обновления, например для установки только тех обновлений, которые соответствуют классификации **безопасности** , установленные обновления отличаются от или подмножества обновлений, приведенных ранее в соответствии с этой классификацией.

### <a name="cause"></a>Причина

Когда выполняется оценка обновлений ОС для компьютера Linux, откройте файлы на [языке уязвимости и оценки](https://oval.mitre.org/) (овалы), предоставляемые поставщиком дистрибутив для Linux, с помощью Управление обновлениями для классификации. Классификация выполняется для обновлений Linux в качестве **безопасности** или **других**, в зависимости от файлов овала, в которых говорится об обновлениях, устраняющих проблемы безопасности или уязвимости. Но при запуске расписания обновления оно выполняется на компьютере Linux с помощью соответствующего диспетчера пакетов, например YUM, APT или ZYPPER для установки. Диспетчер пакетов для дистрибутив Linux может иметь другой механизм классификации обновлений, где результаты могут отличаться от тех, которые были получены из ОВАЛА-файлов Управление обновлениями.

### <a name="resolution"></a>Решение

Можно вручную проверить компьютер Linux, применимые обновления и их классификацию для диспетчера пакетов дистрибутив. Чтобы узнать, какие обновления классифицируются как **Безопасность** диспетчером пакетов, выполните следующие команды.

Для YUM Следующая команда возвращает ненулевой список обновлений, отнесенных к **безопасности** с помощью Red Hat. Обратите внимание, что в случае с CentOS всегда возвращает пустой список, и классификация безопасности не выполняется.

```bash
sudo yum -q --security check-update
```

Для ZYPPER Следующая команда возвращает ненулевой список обновлений, отнесенных к **безопасности** SUSE.

```bash
sudo LANG=en_US.UTF8 zypper --non-interactive patch --category security --dry-run
```

Для APT Следующая команда возвращает ненулевой список обновлений, отнесенных к **безопасности** на основе канонического для Ubuntu Linux дистрибутивов.

```bash
sudo grep security /etc/apt/sources.list > /tmp/oms-update-security.list LANG=en_US.UTF8 sudo apt-get -s dist-upgrade -oDir::Etc::Sourcelist=/tmp/oms-update-security.list
```

В этом списке выполните команду, `grep ^Inst` чтобы получить все ожидающие обновления для системы безопасности.

## <a name="scenario-you-receive-the-error-failed-to-enable-the-update-solution"></a><a name="failed-to-enable-error"></a>Сценарий: отображается сообщение об ошибке Failed to enable the Update solution (Не удалось включить решение по обновлению)

### <a name="issue"></a>Проблема

При попытке включить решение "Управление обновлениями" в учетной записи службы автоматизации возникает следующая ошибка:

```error
Error details: Failed to enable the Update solution
```

### <a name="cause"></a>Причина

Эта ошибка может возникать по следующим причинам.

* Требования к сетевому брандмауэру для агента Log Analytics могут быть настроены неправильно. Такая ситуация может привести к сбою агента при разрешении URL-адресов DNS.

* Целевая настройка Управления обновлениями неправильно настроена, и компьютер не получает обновлений должным образом.

* Вы также можете заметить, что на компьютере в разделе **Соответствие** отображается состояние `Non-compliant`. В то же время **Аналитика компьютеров** сообщает, что агент имеет состояние `Disconnected`.

### <a name="resolution"></a>Решение

* Запустите средство устранения неполадок для [Windows](update-agent-issues.md#troubleshoot-offline) или [Linux](update-agent-issues-linux.md#troubleshoot-offline) в зависимости от операционной системы.

* Дополнительные сведения о том, какие адреса и порты должны быть разрешены, чтобы Управление обновлениями работало, см. в разделе [о конфигурации сети](../automation-hybrid-runbook-worker.md#network-planning).  

* Проверьте наличие проблем с конфигурацией области. [Конфигурация области](../update-management/scope-configuration.md) определяет, какие компьютеры настроены для Управления обновлениями. Если компьютер отображается в рабочей области, но не в Управление обновлениями, необходимо задать конфигурацию области для целевых компьютеров. Дополнительные сведения о конфигурации области см. в разделе [Включение компьютеров в рабочей области](../update-management/enable-from-automation-account.md#enable-machines-in-the-workspace).

* Удалите конфигурацию рабочей роли, выполнив действия из разделов [Удаление гибридной рабочей роли Runbook с локального компьютера Windows](../automation-windows-hrw-install.md#remove-windows-hybrid-runbook-worker) или [Удаление гибридной рабочей роли Runbook с локального компьютера Linux](../automation-linux-hrw-install.md#remove-linux-hybrid-runbook-worker).

## <a name="scenario-superseded-update-indicated-as-missing-in-update-management"></a>Сценарий: заменяемое обновление отображается как отсутствующее в Управлении обновлениями

### <a name="issue"></a>Проблема

Старые обновления отображаются для учетной записи службы автоматизации как отсутствующие, даже если они были заменены. Заменяемое обновление не надо устанавливать, так как доступно более позднее обновление, устраняющее ту же уязвимость. Управление обновлениями игнорирует заменяемое обновление и делает его неприменимым. Вместо него используется заменяющее обновление. Сведения о связанных проблемах см. в разделе о [замененном обновлении](/windows/deployment/update/windows-update-troubleshooting#the-update-is-not-applicable-to-your-computer).

### <a name="cause"></a>Причина

Замененные обновления не отклоняются в Windows Server Update Services (WSUS), поэтому их можно считать неприменимыми.

### <a name="resolution"></a>Решение

Если заменяемое обновление преобразуется в 100% неприменимо, следует изменить состояние утверждения этого обновления на `Declined` в службах WSUS. Чтобы изменить состояние утверждения всех обновлений, выполните следующие действия:

1. Выберите **Управление обновлениями** в учетной записи службы автоматизации, чтобы просмотреть состояние компьютера. Ознакомьтесь с разделом [Просмотр оценок обновления](../update-management/view-update-assessments.md).

2. Проверьте заменяемое обновление, чтобы убедиться в том, что оно на 100 % неприменимо.

3. На сервере WSUS, на котором выявляются компьютеры, [отклоните обновление](/windows-server/administration/windows-server-update-services/manage/updates-operations#declining-updates).

4. Выберите **Компьютеры** и в столбце **Соответствие** выполните повторное сканирование на соответствие. См. раздел [Управление обновлениями для виртуальных машин](../update-management/manage-updates-for-vm.md).

5. Повторите предыдущие шаги для других заменяемых обновлений.

6. Для Windows Server Update Services (WSUS) Очистите все заменяемые обновления, чтобы обновить инфраструктуру с помощью [мастера очистки сервера](/windows-server/administration/windows-server-update-services/manage/the-server-cleanup-wizard)WSUS.

7. Повторяйте эту процедуру регулярно, чтобы устранить проблему с отображением и максимально сократить объем дискового пространства, используемого для управления обновлениями.

## <a name="scenario-machines-dont-show-up-in-the-portal-under-update-management"></a><a name="nologs"></a>Сценарий. Компьютеры не отображаются на портале в разделе Управления обновлениями

### <a name="issue"></a>Проблема

У ваших компьютеров возникли указанные ниже симптомы:

* На компьютере отображается `Not configured` в представлении Управления обновлениями виртуальной машины.

* Ваши компьютеры отсутствуют в представлении Управления обновлениями учетной записи службы автоматизации Azure.

* У вас есть компьютеры, которые имеют состояние `Not assessed` в разделе **Соответствие**. Однако в журналах Azure Monitor отображаются данные пульса для гибридной рабочей роли Runbook, но не для Управления обновлениями.

### <a name="cause"></a>Причина

Эта проблема может быть вызвана проблемами с локальной конфигурацией или неправильно настроенной конфигурацией области. Возможные причины:

* Возможно, потребуется повторно зарегистрировать и установить гибридную рабочую роль Runbook.

* Возможно, вы определили квоту в рабочей области, которая была достигнута, что препятствует последующему хранению данных.

### <a name="resolution"></a>Решение

1. Запустите средство устранения неполадок для [Windows](update-agent-issues.md#troubleshoot-offline) или [Linux](update-agent-issues-linux.md#troubleshoot-offline) в зависимости от операционной системы.

2. Убедитесь, что компьютер отправляет отчеты в правильную рабочую область. Инструкции по проверке этого аспекта см. в разделе [Проверка подключения агента к Azure Monitor](../../azure-monitor/agents/agent-windows.md#verify-agent-connectivity-to-azure-monitor). Также убедитесь, что эта рабочая область связана с учетной записью службы автоматизации Azure. Для подтверждения перейдите на страницу учетной записи службы автоматизации и в разделе **Связанные ресурсы** выберите **Связанная рабочая область**.

3. Убедитесь, что компьютеры отображаются в рабочей области Log Analytics, связанной с вашей учетной записью службы автоматизации. Выполните следующий запрос в рабочей области Log Analytics.

   ```kusto
   Heartbeat
   | summarize by Computer, Solutions
   ```

    Если компьютер не отображается в результатах запроса, он давно не синхронизировался. Вероятно, существует ошибка локальной конфигурации и необходимо [переустановить агент](../../azure-monitor/vm/quick-collect-windows-computer.md#install-the-agent-for-windows).

    Если компьютер указан в результатах запроса, проверьте свойство **Solutions** , которое присутствует в списке **обновлений** . Он проверяет, зарегистрирован ли он в Управление обновлениями. Если это не так, проверьте наличие проблем с конфигурацией области. [Конфигурация области](../update-management/scope-configuration.md) определяет, какие компьютеры настроены для Управления обновлениями. Чтобы настроить конфигурацию области для целевого компьютера, см. раздел [Включение компьютеров в рабочей области](../update-management/enable-from-automation-account.md#enable-machines-in-the-workspace).

4. В рабочей области выполните этот запрос.

   ```kusto
   Operation
   | where OperationCategory == 'Data Collection Status'
   | sort by TimeGenerated desc
   ```

   Если получен такой результат: `Data collection stopped due to daily limit of free data reached. Ingestion status = OverQuota`, значит достигнута квота, определенная в рабочей области, которая не позволяет сохранить данные. В рабочей области перейдите к параметру **Управление объемом данных** в разделе **Использование и ожидаемые затраты** и измените или удалите квоту.

5. Если проблема по прежнему не решена, следуйте инструкциям статьи [Развертывание гибридной рабочей роли Runbook для Windows](../automation-windows-hrw-install.md), чтобы переустановить гибридную рабочую роль для Windows. Для Linux следуйте инструкциям из статьи [Развертывание гибридной рабочей роли Runbook для Linux](../automation-linux-hrw-install.md).

## <a name="scenario-unable-to-register-automation-resource-provider-for-subscriptions"></a><a name="rp-register"></a>Сценарий. Не удается оформить подписки для поставщика ресурсов службы автоматизации

### <a name="issue"></a>Проблема

При работе с развертываниями компонентов в учетной записи службы автоматизации возникает следующая ошибка:

```error
Error details: Unable to register Automation Resource Provider for subscriptions
```

### <a name="cause"></a>Причина

Поставщик ресурсов службы автоматизации не зарегистрирован в подписке.

### <a name="resolution"></a>Решение

Чтобы зарегистрировать поставщик ресурсов службы автоматизации, выполните указанные ниже действия на портале Azure.

1. В нижней части портала в списке служб Azure выберите **Все службы**, а затем в общей группе служб выберите **Подписки**.

2. Выберите свою подписку.

3. В разделе **Параметры** выберите **Поставщики ресурсов**.

4. В списке поставщиков ресурсов убедитесь в том, что поставщик ресурсов Microsoft.Automation зарегистрирован.

5. Если он отсутствует в списке, зарегистрируйте поставщик Microsoft.Automation, выполнив действия из статьи [Устранение ошибок регистрации поставщика ресурсов](../../azure-resource-manager/templates/error-register-resource-provider.md).

## <a name="scenario-scheduled-update-with-a-dynamic-schedule-missed-some-machines"></a><a name="scheduled-update-missed-machines"></a>Сценарий. В запланированном обновлении с динамическим расписанием отсутствуют некоторые компьютеры

### <a name="issue"></a>Проблема

Компьютеры, включенные в предварительный просмотр обновления, не отображаются в списке компьютеров, исправленных во время запланированного запуска.

### <a name="cause"></a>Причина

Эта проблема может возникать по любой из следующих причин:

* Подписки, определенные в области в динамическом запросе, не настроены для зарегистрированного поставщика ресурсов службы автоматизации.

* Компьютеры не были доступны или не имеют подходящих тегов при выполнении расписания.

### <a name="resolution"></a>Решение

#### <a name="subscriptions-not-configured-for-registered-automation-resource-provider"></a>Подписки, не настроенные для зарегистрированного поставщика ресурсов службы автоматизации

Если ваша подписка не настроена для поставщика ресурсов службы автоматизации, вы не сможете запрашивать или получать сведения на компьютерах в этой подписке. Чтобы проверить регистрацию подписки, выполните следующие действия.

1. На [портале Azure](../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal) перейдите к списку служб Azure.

2. Щелкните **Все службы**, а затем в общей группе служб выберите **Подписки**.

3. Найдите подписку, определенную в области для развертывания.

4. В разделе **Параметры** выберите **Поставщики ресурсов**.

5. Убедитесь в том, что поставщик ресурсов Microsoft.Automation зарегистрирован.

6. Если он отсутствует в списке, зарегистрируйте поставщик Microsoft.Automation, выполнив действия из статьи [Устранение ошибок регистрации поставщика ресурсов](../../azure-resource-manager/templates/error-register-resource-provider.md).

#### <a name="machines-not-available-or-not-tagged-correctly-when-schedule-executed"></a>Компьютеры недоступны или неправильно помечены тегами при выполнении расписания

Выполните перечисленные ниже действия, если ваша подписка настроена для поставщика ресурсов службы автоматизации, но при выполнении расписания обновлений с указанными [динамическими группами](../update-management/configure-groups.md) пропущены некоторые компьютеры.

1. На портале Azure откройте учетную запись службы автоматизации и выберите **Управление обновлениями**.

2. Проверьте [журнал Управления обновлениями](../update-management/deploy-updates.md#view-results-of-a-completed-update-deployment), чтобы определить точное время, когда было запущено развертывание обновления.

3. Чтобы [найти изменения на компьютерах](../../governance/resource-graph/how-to/get-resource-changes.md#find-detected-change-events-and-view-change-details), которые (как вы подозреваете) были пропущены Управлением обновлениями, используйте Azure Resource Graph (ARG).

4. Ищите изменения за продолжительный период, например за один день до запуска развертывания обновления.

5. Проверьте результаты поиска на наличие системных изменений, таких как изменения удалений или обновлений, на компьютерах за этот период. Эти изменения могут изменить состояние или теги компьютера, чтобы он не был выбран в списке компьютеров при развертывании обновлений.

6. При необходимости измените параметры компьютеров и ресурсов, чтобы устранить проблемы с тегами или состоянием компьютера.

7. Повторно запустите расписание обновлений, чтобы убедиться, что развертывание с указанными динамическими группами включает все компьютеры.

## <a name="scenario-expected-machines-dont-appear-in-preview-for-dynamic-group"></a><a name="machines-not-in-preview"></a>Сценарий. Ожидаемые компьютеры не отображаются в предварительном просмотре для динамической группы

### <a name="issue"></a>Проблема

Виртуальные машины для выбранных областей динамической группы не отображаются в списке предварительного просмотра на портале Azure. В этом списке находятся все компьютеры, полученные через запрос ARG для выбранных областей. Области фильтруются для компьютеров, на которых установлены гибридные рабочие роли Runbook, к которым у вас есть право доступа.

### <a name="cause"></a>Причина

Возможные причины этой проблемы:

* У вас нет нужного уровня доступа к выбранным областям.
* Запрос ARG не извлекает ожидаемые компьютеры.
* Гибридная рабочая роль Runbook не установлена на компьютерах.

### <a name="resolution"></a>Решение 

#### <a name="incorrect-access-on-selected-scopes"></a>Неверный доступ к выбранным областям

На портале Azure отображаются только те компьютеры, к которым у вас есть доступ на запись в заданной области. Если у вас нет правильного доступа к области, см. раздел [учебник. предоставление пользователю доступа к ресурсам Azure с помощью портал Azure](../../role-based-access-control/quickstart-assign-role-user-portal.md).

#### <a name="arg-query-doesnt-return-expected-machines"></a>Запрос ARG не извлекает ожидаемые компьютеры

Выполните следующие действия, чтобы выяснить, правильно ли работают запросы.

1. Выполните запрос ARG, чтобы его формат соответствовал показанному ниже в колонке обозревателя Resource Graph на портале Azure. Этот запрос имитирует фильтры, выбранные при создании динамической группы в Управлении обновлениями. Ознакомьтесь со статьей [Использование динамических групп с Управлением обновлениями](../update-management/configure-groups.md).

    ```kusto
    where (subscriptionId in~ ("<subscriptionId1>", "<subscriptionId2>") and type =~ "microsoft.compute/virtualmachines" and properties.storageProfile.osDisk.osType == "<Windows/Linux>" and resourceGroup in~ ("<resourceGroupName1>","<resourceGroupName2>") and location in~ ("<location1>","<location2>") )
    | project id, location, name, tags = todynamic(tolower(tostring(tags)))
    | where  (tags[tolower("<tagKey1>")] =~ "<tagValue1>" and tags[tolower("<tagKey2>")] =~ "<tagValue2>") // use this if "All" option selected for tags
    | where  (tags[tolower("<tagKey1>")] =~ "<tagValue1>" or tags[tolower("<tagKey2>")] =~ "<tagValue2>") // use this if "Any" option selected for tags
    | project id, location, name, tags
    ```

   Например:

    ```kusto
    where (subscriptionId in~ ("20780d0a-b422-4213-979b-6c919c91ace1", "af52d412-a347-4bc6-8cb7-4780fbb00490") and type =~ "microsoft.compute/virtualmachines" and properties.storageProfile.osDisk.osType == "Windows" and resourceGroup in~ ("testRG","withinvnet-2020-01-06-10-global-resources-southindia") and location in~ ("australiacentral","australiacentral2","brazilsouth") )
    | project id, location, name, tags = todynamic(tolower(tostring(tags)))
    | where  (tags[tolower("ms-resource-usage")] =~ "azure-cloud-shell" and tags[tolower("temp")] =~ "temp")
    | project id, location, name, tags
    ```

2. Проверьте, перечислены ли запрашиваемые компьютеры в результатах запроса.

3. Если компьютеры отсутствуют в списке, вероятно, возникли проблемы с фильтром, выбранным в динамической группе. Внесите необходимые изменения в конфигурацию группы.

#### <a name="hybrid-runbook-worker-not-installed-on-machines"></a>Гибридная рабочая роль Runbook не установлена на компьютерах

Компьютеры отображаются в результатах запроса ARG, но по-прежнему не отображаются в предварительном просмотре динамической группы. В этом случае компьютеры могут быть не назначенными в качестве гибридных рабочих ролей и, таким образом, не могут запускать задания службы автоматизации Azure и Управления обновлениями. Чтобы убедиться, что компьютеры, которые вы ожидаете увидеть, настроены как гибридные рабочие роли Runbook, выполните следующие действия:

1. На портале Azure перейдите к учетной записи службы автоматизации компьютера, который отображается неправильно.

2. В разделе **Автоматизация процессов** выберите **Группы гибридных рабочих ролей**.

3. Выберите вкладку **Системные группы гибридных рабочих ролей**.

4. Проверьте наличие гибридной рабочей роли для этого компьютера.

5. Если компьютер не настроен в качестве гибридной рабочей роли, внесите изменения, используя инструкции по [автоматизации ресурсов в центре обработки данных или облаке с помощью гибридной рабочей роли Runbook](../automation-hybrid-runbook-worker.md).

6. Присоедините компьютер к группе гибридных рабочих ролей Runbook.

7. Повторите описанные выше действия для всех компьютеров, которые не отображались в предварительном просмотре.

## <a name="scenario-update-management-components-enabled-while-vm-continues-to-show-as-being-configured"></a><a name="components-enabled-not-working"></a>Сценарий. Компоненты Управления обновлениями включены, тогда как виртуальная машина продолжает отображаться во время настройки

### <a name="issue"></a>Проблема

Приведенное ниже сообщение на виртуальной машине не исчезает в течение 15 минут после начала развертывания:

```error
The components for the 'Update Management' solution have been enabled, and now this virtual machine is being configured. Please be patient, as this can sometimes take up to 15 minutes.
```

### <a name="cause"></a>Причина

Эта ошибка может возникать по следующим причинам.

* Обмен данными в учетной записи службы автоматизации заблокирован.

* Существует повторяющееся имя компьютера с разными идентификаторами исходного компьютера. Такая ситуация возникает, если виртуальная машина с определенным именем компьютера создается в разных группах ресурсов и отправляет отчеты в одну и ту же рабочую область Logistics Agent в подписке.

* Развертываемый образ виртуальной машины может быть взят с клонированного компьютера, который не был подготовлен с помощью программы подготовки системы (sysprep) с установленным агентом Log Analytics для Windows.

### <a name="resolution"></a>Решение

Чтобы определить точную проблему с виртуальной машиной, выполните следующий запрос в рабочей области Log Analytics, связанной с вашей учетной записью службы автоматизации.

```
Update
| where Computer contains "fillInMachineName"
| project TimeGenerated, Computer, SourceComputerId, Title, UpdateState 
```

#### <a name="communication-with-automation-account-blocked"></a>Обмен данными с учетной записью службы автоматизации заблокирован

Дополнительные сведения о том, какие адреса и порты должны быть разрешены, чтобы Управление обновлениями работало, см. в разделе [Планирование сети](../update-management/overview.md#ports).

#### <a name="duplicate-computer-name"></a>Повторяющееся имя компьютера

Переименуйте виртуальные машины, чтобы обеспечить уникальность имен в среде.

#### <a name="deployed-image-from-cloned-machine"></a>Развернутый образ с клонированного компьютера

Если вы используете клонированный образ, разные имена компьютеров имеют одинаковый идентификатор исходного компьютера. В данном случае:

1. В рабочей области Log Analytics удалите виртуальную машину из сохраненных поисковых запросов в конфигурации области `MicrosoftDefaultScopeConfig-Updates`, если она отображается. Сохраненные поисковые запросы хранятся на вкладке **Общие** рабочей области.

2. Выполните следующий командлет.

    ```azurepowershell-interactive
    Remove-Item -Path "HKLM:\software\microsoft\hybridrunbookworker" -Recurse -Force
    ```

3. Выполните `Restart-Service HealthService` для перезапуска службы работоспособности. Эта операция повторно создаст ключ, а также сгенерирует идентификатор UUID.

4. Если такой подход не работает, сначала запустите Sysprep на образе, а затем установите агент Log Analytics для Windows.

## <a name="scenario-you-receive-a-linked-subscription-error-when-you-create-an-update-deployment-for-machines-in-another-azure-tenant"></a><a name="multi-tenant"></a>Сценарий. Вы получаете ошибку связанной подписки при создании развертывания обновлений для компьютеров в другом клиенте Azure

### <a name="issue"></a>Проблема

Вы получаете следующую ошибку при попытке создания развертывания обновлений для компьютеров в другом клиенте Azure:

```error
The client has permission to perform action 'Microsoft.Compute/virtualMachines/write' on scope '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resourceGroupName/providers/Microsoft.Automation/automationAccounts/automationAccountName/softwareUpdateConfigurations/updateDeploymentName', however the current tenant '00000000-0000-0000-0000-000000000000' is not authorized to access linked subscription '00000000-0000-0000-0000-000000000000'.
```

### <a name="cause"></a>Причина

Эта ошибка возникает при создании развертывания обновлений с виртуальными машинами Azure, расположенными в другом клиенте.

### <a name="resolution"></a>Решение

Используйте обходной путь, чтобы запланировать обновление этих элементов. Чтобы создать расписание, можно использовать командлет [New-AzAutomationSchedule](/powershell/module/az.automation/new-azautomationschedule) с параметром `ForUpdateConfiguration`. Затем примените командлет [New-AzAutomationSoftwareUpdateConfiguration](/powershell/module/Az.Automation/New-AzAutomationSoftwareUpdateConfiguration), передав ему параметр `NonAzureComputer` со списком компьютеров в другом клиенте. В приведенном ниже примере показано, как это сделать.

```azurepowershell-interactive
$nonAzurecomputers = @("server-01", "server-02")

$startTime = ([DateTime]::Now).AddMinutes(10)

$s = New-AzAutomationSchedule -ResourceGroupName mygroup -AutomationAccountName myaccount -Name myupdateconfig -Description test-OneTime -OneTime -StartTime $startTime -ForUpdateConfiguration

New-AzAutomationSoftwareUpdateConfiguration  -ResourceGroupName $rg -AutomationAccountName $aa -Schedule $s -Windows -AzureVMResourceId $azureVMIdsW -NonAzureComputer $nonAzurecomputers -Duration (New-TimeSpan -Hours 2) -IncludedUpdateClassification Security,UpdateRollup -ExcludedKbNumber KB01,KB02 -IncludedKbNumber KB100
```

## <a name="scenario-unexplained-reboots"></a><a name="node-reboots"></a>Сценарий. Необъяснимые перезагрузки

### <a name="issue"></a>Проблема

Несмотря на то, что для параметра **управления перезагрузкой** установлено значение **Никогда не перезагружать**, компьютеры по-прежнему перезагружаются после установки обновлений.

### <a name="cause"></a>Причина

В Центр обновления Windows могли внести изменения, используя некоторые разделы реестра, с помощью которых можно изменить поведение при перезагрузке.

### <a name="resolution"></a>Решение

Проверьте разделы реестра, перечисленные в разделе [Настройка автоматического обновления путем редактирования реестра](/windows/deployment/update/waas-wu-settings#configuring-automatic-updates-by-editing-the-registry) и [Разделы реестра, используемые для управления перезапуском](/windows/deployment/update/waas-restart#registry-keys-used-to-manage-restart), чтобы убедиться, что компьютеры настроены правильно.

## <a name="scenario-machine-shows-failed-to-start-in-an-update-deployment"></a><a name="failed-to-start"></a>Сценарий. На компьютере отображается сообщение "Не удалось запустить" при развертывании обновлений

### <a name="issue"></a>Проблема

На компьютере отображается состояние `Failed to start`. При просмотре подробной информации о компьютере наблюдается следующая ошибка:

```error
Failed to start the runbook. Check the parameters passed. RunbookName Patch-MicrosoftOMSComputer. Exception You have requested to create a runbook job on a hybrid worker group that does not exist.
```

### <a name="cause"></a>Причина

Ошибка может быть вызвана одной из следующих причин:

* Компьютер больше не существует.
* Компьютер отключен и недоступен.
* У компьютера неполадки с сетевым подключением, поэтому гибридная рабочая роль на компьютере недоступна.
* Произошло обновление агента Log Analytics, который изменил идентификатор исходного компьютера.
* Выполнение обновления было отрегулировано, если достигнуто ограничение в 200 параллельных заданий в учетной записи службы автоматизации. Каждое развертывание считается заданием, а каждый компьютер в развертывании обновлений учитывается как задание. Любое другое задание службы автоматизации или развертывание обновлений, выполняющееся в данный момент в вашей учетной записи службы автоматизации, учитывает ограничение на число параллельных заданий.

### <a name="resolution"></a>Решение

При возможности используйте [динамические группы](../update-management/configure-groups.md) для развертываний обновлений. Кроме того, можно выполнить следующие действия.

1. Убедитесь, что компьютер или сервер соответствуют [требованиям](../update-management/overview.md#system-requirements).
2. Проверьте подключение к гибридной рабочей роли Runbook с помощью средства устранения неполадок агента гибридной рабочей роли Runbook. Для дополнительных сведений о средстве устранения неполадок см. [Общие сведения о результатах проверки агента в службе "Управление обновлениями"](update-agent-issues.md).

## <a name="scenario-updates-are-installed-without-a-deployment"></a><a name="updates-nodeployment"></a>Сценарий. Обновления устанавливаются без развертывания

### <a name="issue"></a>Проблема

При регистрации компьютера Windows в Управлении обновлениями отобразятся обновления, установленные без развертывания.

### <a name="cause"></a>Причина

Обновления устанавливаются в Windows автоматически, как только они становятся доступными. Такое поведение может вызвать путаницу, если вы не планировали развертывание обновления на компьютере.

### <a name="resolution"></a>Решение

Раздел реестра `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU` по умолчанию имеет значение 4: `auto download and install`.

Для клиентов Управления обновлениями рекомендуется задать для этого раздела значение 3: `auto download but do not auto install`.

Ознакомьтесь с дополнительными сведениями о [настройке автоматических обновлений](/windows/deployment/update/waas-wu-settings#configure-automatic-updates).

## <a name="scenario-machine-is-already-registered-to-a-different-account"></a><a name="machine-already-registered"></a>Сценарий. Компьютер уже зарегистрирован в другой учетной записи

### <a name="issue"></a>Проблема

Отображается следующее сообщение об ошибке:

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception System.InvalidOperationException: {"Message":"Machine is already registered to a different account."}
```

### <a name="cause"></a>Причина

Компьютер уже развернут в другой рабочей области для Управления обновлениями.

### <a name="resolution"></a>Решение

1. Выполните действия из раздела [Компьютеры не отображаются на портале в разделе Управления обновлениями](#nologs), чтобы компьютер отчитывался соответствующей рабочей области.
2. Выполните очистку артефактов на компьютере путем [удаления группы гибридных модулей Runbook](../automation-windows-hrw-install.md#remove-a-hybrid-worker-group), а затем повторите попытку.

## <a name="scenario-machine-cant-communicate-with-the-service"></a><a name="machine-unable-to-communicate"></a>Сценарий. Компьютеру не удается связаться со службой

### <a name="issue"></a>Проблема

Отображается одно из следующих сообщений об ошибке.

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception System.Net.Http.HttpRequestException: An error occurred while sending the request. ---> System.Net.WebException: The underlying connection was closed: An unexpected error occurred on a receive. ---> System.ComponentModel.Win32Exception: The client and server can't communicate, because they do not possess a common algorithm
```

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception Newtonsoft.Json.JsonReaderException: Error parsing positive infinity value.
```

```error
The certificate presented by the service <wsid>.oms.opinsights.azure.com was not issued by a certificate authority used for Microsoft services. Contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication.
```

```error
Access is denied. (Exception form HRESULT: 0x80070005(E_ACCESSDENIED))
```

### <a name="cause"></a>Причина

Возможно, прокси-сервер, шлюз или брандмауэр блокируют сетевую связь.

### <a name="resolution"></a>Решение

Проверьте сеть и убедитесь, что необходимые порты и адреса разрешены. Раздел [Требования сети](../automation-hybrid-runbook-worker.md#network-planning) содержит информацию о том, как создать список портов и адресов, необходимых для Управления обновлениями и гибридных рабочих ролей Runbook.

## <a name="scenario-unable-to-create-self-signed-certificate"></a><a name="unable-to-create-selfsigned-cert"></a>Сценарий. Невозможно создать самозаверяющий сертификат

### <a name="issue"></a>Проблема

Отображается одно из следующих сообщений об ошибке.

```error
Unable to Register Machine for Patch Management, Registration Failed with Exception AgentService.HybridRegistration. PowerShell.Certificates.CertificateCreationException: Failed to create a self-signed certificate. ---> System.UnauthorizedAccessException: Access is denied.
```

### <a name="cause"></a>Причина

Гибридной рабочей роли Runbook не удалось создать самозаверяющий сертификат.

### <a name="resolution"></a>Решение

Убедитесь, что системная учетная запись имеет доступ для чтения к папке **C:\ProgramData\Microsoft\Crypto\RSA** и повторите попытку.

## <a name="scenario-the-scheduled-update-failed-with-a-maintenancewindowexceeded-error"></a><a name="mw-exceeded"></a>Сценарий. Запланированное обновление завершилось с ошибкой MaintenanceWindowExceeded

### <a name="issue"></a>Проблема

Период обслуживания по умолчанию для обновлений — 120 минут. Период обслуживания можно увеличить до 6 часов или 360 минут.

### <a name="resolution"></a>Решение

Чтобы понять, почему это произошло во время выполнения обновления после его успешного запуска, [Проверьте выходные данные задания](../update-management/deploy-updates.md#view-results-of-a-completed-update-deployment) на затронутом компьютере в ходе выполнения. Просмотрите конкретные сообщения об ошибках ваших компьютеров и примите меры.  

Измените все неудачные запланированные развертывания обновлений и увеличьте период обслуживания.

Дополнительные сведения о периодах обслуживания см. в разделе об [установке обновлений](../update-management/deploy-updates.md#schedule-an-update-deployment).

## <a name="scenario-machine-shows-as-not-assessed-and-shows-an-hresult-exception"></a><a name="hresult"></a>Сценарий. Компьютер отображается как "Без оценки" и показывает исключение HRESULT

### <a name="issue"></a>Проблема

* У вас есть компьютеры, которые отображаются как `Not assessed` в разделе **Совместимость**, а ниже появляется сообщение об ошибке.
* На портале отобразится код ошибки HRESULT.

### <a name="cause"></a>Причина

Агент обновления (агент Центра обновления Windows для Windows; диспетчер пакетов для дистрибутива Linux) настроен неправильно. Управление обновлениями зависит от агента обновления на компьютере для предоставления необходимых обновлений, состояния исправления и результатов развернутых исправлений. Без этой информации Управление обновлениями не сможет должным образом предоставлять отчеты об исправлениях, которые требуется установить или которые уже установлены.

### <a name="resolution"></a>Решение

Попробуйте выполнить обновления на компьютере локально. Если эта операция завершается неудачно, это обычно означает ошибку конфигурации агента обновления.

Эта проблема часто возникает из-за неправильной конфигурации сети или брандмауэра. Выполните следующие проверки, чтобы устранить проблему.

* Для Linux проверьте соответствующую документацию, чтобы убедиться, что вы можете связаться с конечной точкой сети репозитория пакетов.

* Для Windows проверьте конфигурацию агента, как указано в статье об [устранении проблем с обновлением Windows](/windows/deployment/update/windows-update-troubleshooting#updates-arent-downloading-from-the-intranet-endpoint-wsussccm).

  * Если компьютеры настроены для Центра обновления Windows, убедитесь, что вы можете обратиться к конечным точкам, описанным в разделе [Проблемы с HTTP- и прокси-сервером](/windows/deployment/update/windows-update-troubleshooting#issues-related-to-httpproxy).
  * Если компьютеры настроены для службы Windows Server Update Services (WSUS), убедитесь, что вы можете подключиться к серверу WSUS, настроенному с помощью [раздела реестра WUServer](/windows/deployment/update/waas-wu-settings).

Если отображается значение HRESULT, чтобы просмотреть целиком сообщение об исключении, дважды щелкните исключение, выделенное красным. Ознакомьтесь со следующей таблицей потенциальных решений или рекомендуемых действий.

|Исключение  |Решение или действие  |
|---------|---------|
|`Exception from HRESULT: 0x……C`     | Найдите соответствующий код ошибки в [списке кодов ошибок обновления Windows](https://support.microsoft.com/help/938205/windows-update-error-code-list), чтобы найти подробности о причине создания исключения.        |
|`0x8024402C`</br>`0x8024401C`</br>`0x8024402F`      | Проблемы с сетевым подключением. Убедитесь, что компьютер имеет сетевое подключение для Управления обновлениями. Список обязательных портов и адресов см. в разделе [Планирование сети](../update-management/overview.md#ports).        |
|`0x8024001E`| Операция обновления не завершена, так как служба или система завершали работу.|
|`0x8024002E`| Служба Центра обновления Windows отключена.|
|`0x8024402C`     | Если вы используете сервер WSUS, убедитесь, что значения реестра `WUServer` и `WUStatusServer` в разделе реестра `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate` указывают на правильный сервер WSUS.        |
|`0x80072EE2`|Проблема с сетевым подключением или с обращением к настроенному серверу WSUS. Проверьте параметры WSUS и убедитесь, что служба доступна из клиента.|
|`The service cannot be started, either because it is disabled or because it has no enabled devices associated with it. (Exception from HRESULT: 0x80070422)`     | Убедитесь, что служба Центра обновлений Windows (wuauserv) работает и не отключена.        |
|`0x80070005`| Ошибка отказа в доступе может быть вызвана любой из следующих причин:<br> зараженный компьютер;<br> параметры Центра обновления Windows неправильно настроены;<br> ошибка разрешения файла в папке %WinDir%\SoftwareDistribution;<br> недостаточно дискового пространства на системном диске (C:).
|Другое универсальное исключение     | Выполните поиск возможных решений в Интернете и обратитесь к локальной службе ИТ-поддержки.         |

Просмотр файла **%Windir%\Windowsupdate.log** также может помочь определить возможные причины. Дополнительные сведения о просмотре журнала см. в статье о [чтении файла Windowsupdate.log](https://support.microsoft.com/help/902093/how-to-read-the-windowsupdate-log-file).

ВЫ можете также скачать и запустить [средство устранения неполадок Центра обновления Windows](https://support.microsoft.com/help/4027322/windows-update-troubleshooter) для проверки возможных проблем с этой службой на компьютере.

> [!NOTE]
> В документации по [средству устранения неполадок Центра обновления Windows](https://support.microsoft.com/help/4027322/windows-update-troubleshooter) указано, что оно предназначено для ОС Windows, но также работает в Windows Server.

## <a name="scenario-update-run-returns-failed-status-linux"></a>Сценарий: Запуск обновления возвращает состояние "Сбой" (Linux)

### <a name="issue"></a>Проблема

Запуск обновления начинается, но возникают ошибки во время выполнения.

### <a name="cause"></a>Причина

Возможные причины:

* диспетчер пакетов неработоспособен;
* агент обновления (WUA для Windows, диспетчера пакетов дистрибутива для Linux) неправильно настроен;
* определенные пакеты конфликтуют с установкой исправлений облака;
* компьютер недоступен;
* обновления имели зависимости, которые не были устранены.

### <a name="resolution"></a>Решение

Если сбои происходят во время запуска обновления после того, как он успешно запущен, [проверьте выполнение задания](../update-management/deploy-updates.md#view-results-of-a-completed-update-deployment) на затронутом компьютере в ходе выполнения. Просмотрите конкретные сообщения об ошибках ваших компьютеров и примите меры. Для успешного обновления развертываний управлению обновлениями необходима работоспособность диспетчера пакетов.

Если конкретные исправления, пакеты или обновления отображаются непосредственно перед сбоем задания, можно попытаться [исключить](../update-management/deploy-updates.md#schedule-an-update-deployment) эти элементы из следующего развертывания обновлений. Сведения о сборе информации журнала из Центра обновления Windows см. в статье [Файлы журнала Центра обновления Windows](/windows/deployment/update/windows-update-logs).

Если не удается исправить проблему, создайте копию файла **/var/opt/microsoft/omsagent/run/automationworker/omsupdatemgmt.log** и сохраните его для устранения неполадок перед началом следующего развертывания обновления.

## <a name="patches-arent-installed"></a>Исправления не установлены

### <a name="machines-dont-install-updates"></a>Обновления не устанавливаются на компьютеры

Попробуйте выполнить обновление непосредственно на компьютере. Если не удается обновить программное обеспечение на компьютере, см. [список возможных ошибок в руководстве по устранению неполадок](#hresult).

Если обновления выполняются локально, попробуйте удалить и переустановить агент на компьютере, следуя инструкциям из статьи [Удаление виртуальной машины для Управления обновлениями](../update-management/remove-vms.md).

### <a name="i-know-updates-are-available-but-they-dont-show-as-available-on-my-machines"></a>Я знаю, что обновления доступны, но установить их на компьютерах не предлагается

Это часто происходит, если компьютеры настроены для получения обновлений из WSUS или Microsoft Endpoint Configuration Manager, но WSUS или Configuration Manager не утвердили обновления.

Проверить, настроены ли компьютеры на получение обновлений из WSUS или SCCM можно с помощью перекрестной ссылки между разделом реестра `UseWUServer` и разделами реестра, упомянутыми в статье о [настройке автоматического обновления путем редактирования реестра](https://support.microsoft.com/help/328010/how-to-configure-automatic-updates-by-using-group-policy-or-registry-s).

Если обновления не утверждены в WSUS, они не устанавливаются. Вы можете проверить неутвержденные обновления в Log Analytics, выполнив следующий запрос.

  ```kusto
  Update | where UpdateState == "Needed" and ApprovalSource == "WSUS" and Approved == "False" | summarize max(TimeGenerated) by Computer, KBID, Title
  ```

### <a name="updates-show-as-installed-but-i-cant-find-them-on-my-machine"></a>Обновления отображаются как установленные, но найти их на компьютере не удается

Обновления часто заменяются другими обновлениями. Дополнительные сведения см. в описании причины [Обновление заменено](/windows/deployment/update/windows-update-troubleshooting#the-update-is-not-applicable-to-your-computer) в руководстве по устранению неполадок Центра обновлений Windows.

### <a name="installing-updates-by-classification-on-linux"></a>Установка обновлений с учетом классификации на платформе Linux

При развертывании обновлений на Linux с учетом классификации (критические обновления и обновления системы безопасности) следует учитывать некоторые важные ограничения, особенно для CentOS. Эти ограничения описаны на [странице с общими сведениями об Управлении обновлениями](../update-management/overview.md#linux).

### <a name="kb2267602-is-consistently-missing"></a>Статья базы знаний № 2267602 постоянно отсутствует

Статья базы знаний № 2267602 касается [обновления определений Защитника Windows](https://www.microsoft.com/wdsi/definitions). Обновляется ежедневно.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не нашли свою проблему или рекомендации не позволили ее решить, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы специалистов Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами.
* Отправьте запрос в службу поддержки Azure Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и щелкните **Получить поддержку**.
