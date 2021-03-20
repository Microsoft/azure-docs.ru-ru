---
title: Устранение неполадок при развертывании функций службы автоматизации Azure
description: В этой статье рассказывается, как устранять неполадки, которые могут возникать при развертывании функций службы автоматизации Azure.
services: automation
ms.date: 02/11/2021
ms.topic: troubleshooting
ms.openlocfilehash: 43bc367be9ad9bb32f33f94df774acb3e808182a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100651125"
---
# <a name="troubleshoot-feature-deployment-issues"></a>Устранение неполадок с развертыванием компонентов

При развертывании на виртуальных машинах функций "Управление обновлениями" или "Отслеживание изменений и инвентаризация" для службы автоматизации Azure могут появляться сообщения об ошибках. В этой статье описываются возможные ошибки и пути их устранения.

## <a name="known-issues"></a>Известные проблемы

### <a name="scenario-renaming-a-registered-node-requires-unregister-or-register-again"></a><a name="node-rename"></a>Сценарий. Для переименования зарегистрированного узла нужно отменить и повторно создать регистрацию

#### <a name="issue"></a>Проблема

Для узла, который зарегистрирован в службе автоматизации Azure, изменяется имя компьютера в операционной системе. Отчеты с этого узла отображаются с прежним именем.

#### <a name="cause"></a>Причина

При переименовании зарегистрированных узлов не обновляется имя узла в службе автоматизации Azure.

#### <a name="resolution"></a>Решение

Отмените регистрацию этого узла в службе State Configuration службы автоматизации Azure, а затем заново зарегистрируйте его. Все отчеты, опубликованные в службе до повторной регистрации, станут недоступны.

### <a name="scenario-re-signing-certificates-via-https-proxy-isnt-supported"></a><a name="resigning-cert"></a>Сценарий. Повторное подписывание сертификатов через прокси-сервер HTTPS не поддерживается

#### <a name="issue"></a>Проблема

При подключении через прокси-сервер, который терминирует трафик HTTPS и повторно шифрует его с использованием нового сертификата, служба не разрешает подключение.

#### <a name="cause"></a>Причина

Служба автоматизации Azure не поддерживает повторное подписывание сертификатов, которые используются для шифрования трафика.

#### <a name="resolution"></a>Решение

В настоящее время решение для этой проблемы отсутствует.

## <a name="general-errors"></a>Общие ошибки

### <a name="scenario-feature-deployment-fails-with-the-message-the-solution-cannot-be-enabled"></a><a name="missing-write-permissions"></a>Сценарий. Развертывание функции завершается сбоем с сообщением "Решение невозможно включить".

#### <a name="issue"></a>Проблема

При попытке включить функцию на виртуальной машине вы получаете одно из следующих сообщений.

```error
The solution cannot be enabled due to missing permissions for the virtual machine or deployments
```

```error
The solution cannot be enabled on this VM because the permission to read the workspace is missing
```

#### <a name="cause"></a>Причина

Такая ошибка возникает из-за неправильных или отсутствующих разрешений для виртуальной машины, рабочей области или пользователя.

#### <a name="resolution"></a>Решение

Убедитесь, что вы правильно настроили [разрешения на развертывание функций](../automation-role-based-access-control.md#feature-setup-permissions), и повторите попытку развертывания. Если появится сообщение об ошибке `The solution cannot be enabled on this VM because the permission to read the workspace is missing` , см. следующие [сведения по устранению неполадок](update-management.md#failed-to-enable-error).

### <a name="scenario-feature-deployment-fails-with-the-message-failed-to-configure-automation-account-for-diagnostic-logging"></a><a name="diagnostic-logging"></a>Сценарий. Развертывание функции завершается сбоем с ошибкой "Не удалось указать учетной записи службы автоматизации вести журнал диагностики".

#### <a name="issue"></a>Проблема

При попытке включить функцию на виртуальной машине вы получаете следующее сообщение.

```error
Failed to configure automation account for diagnostic logging
```

#### <a name="cause"></a>Причина

Такая ошибка может быть вызвана тем, что ценовая категория не соответствует модели выставления счетов для подписки. Дополнительные сведения см. в статье [Мониторинг использования и ожидаемых затрат в Azure Monitor](../../azure-monitor//usage-estimated-costs.md).

#### <a name="resolution"></a>Решение

Создайте рабочую область Log Analytics вручную и повторите развертывание функции, чтобы выбрать только что созданную рабочую область.

### <a name="scenario-computergroupqueryformaterror"></a><a name="computer-group-query-format-error"></a>Сценарий. ComputerGroupQueryFormatError

#### <a name="issue"></a>Проблема

Этот код ошибки означает, что сохраненный запрос на поиск группы компьютеров, который используется для выбора этой функции, имеет неправильный формат. 

#### <a name="cause"></a>Причина

Возможно, вы изменили запрос или система внесла автоматическое изменение.

#### <a name="resolution"></a>Решение

Вы можете удалить запрос для этой функции и снова включить ее, и запрос будет создан заново. Текущий запрос можно найти в разделе **Сохраненные поисковые запросы** для рабочей области. Этот запрос имеет имя **MicrosoftDefaultComputerGroup** и категорию, имя которой совпадает с именем связанной функции. Если включены сразу несколько функций, запрос **MicrosoftDefaultComputerGroup** отображается в области **Сохраненные условия поиска** несколько раз.

### <a name="scenario-policyviolation"></a><a name="policy-violation"></a>Сценарий. PolicyViolation

#### <a name="issue"></a>Проблема

Этот код ошибки означает, что не удалось выполнить развертывание из-за нарушения одной или нескольких политик.

#### <a name="cause"></a>Причина 

Политика блокирует завершение операции.

#### <a name="resolution"></a>Решение

Чтобы успешно развернуть функцию, придется изменить указанную политику. Поскольку политики могут быть очень разными, требуемые изменения тоже будут разными в зависимости от конкретной нарушенной политики. Например, если эта политика определена для группы ресурсов и запрещает изменять содержимое некоторых автономных ресурсов, вы можете применить одно из следующих исправлений:

* Полностью удалите политику.
* Попробуйте включить функцию в другой группе ресурсов.
* Переместите политику в конкретный ресурс (например, в учетную запись службы автоматизации).
* Пересмотрите набор ресурсов, к которым политика должна запрещать доступ.

Проверьте уведомления в верхнем правом углу портала Azure или перейдите к группе ресурсов, которая содержит выбранную учетную запись службы автоматизации, и щелкните **Развертывания** в разделе **Параметры**, чтобы изучить проблемное развертывание. Дополнительные сведения о службе "Политика Azure" см. в [обзоре политики Azure](../../governance/policy/overview.md?toc=%2fazure%2fautomation%2ftoc.json).

### <a name="scenario-errors-trying-to-unlink-a-workspace"></a><a name="unlink"></a>Сценарий. Ошибки при попытке удалить связь с рабочей областью

#### <a name="issue"></a>Проблема

При попытке удалить связь с рабочей областью появляется следующее сообщение об ошибке:

```error
The link cannot be updated or deleted because it is linked to Update Management and/or ChangeTracking Solutions.
```

#### <a name="cause"></a>Причина

Такая ошибка возникает, если в рабочей области Log Analytics имеются активные функции с зависимостью от учетной записи службы автоматизации и связанной рабочей области Log Analytics.

### <a name="resolution"></a>Решение

Удалите из рабочей области ресурсы для следующих функций, если они используются.

* Управление обновлениями
* Отслеживание изменений и инвентаризация
* Запуск и остановка виртуальных машин в нерабочее время

После удаления ресурсов для функций вы сможете удалить связь с рабочей областью. При этом важно удалить из рабочей области и учетной записи службы автоматизации все существующие артефакты для этих функций.

* Для функции "Управление обновлениями" удалите компонент **Развертывания обновлений (расписания)** из учетной записи службы автоматизации.
* Для функции "Запуск и остановка виртуальных машин в нерабочее время" удалите все блокировки компонентов функции из учетной записи службы автоматизации с помощью раздела **Параметры** > **Блокировки**. Дополнительные сведения см. в разделе [Удаление функции](../automation-solution-vm-management-remove.md).

## <a name="log-analytics-for-windows-extension-failures"></a><a name="mma-extension-failures"></a>Сбой расширения Log Analytics для Windows

[!INCLUDE [log-analytics-agent-note](../../../includes/log-analytics-agent-note.md)] 

Установка расширения Log Analytics для Windows может завершиться сбоем по ряду причин. В следующем разделе описаны возможные проблемы с развертыванием функций, которые могут приводить к сбоям во время развертывания расширения агента Log Analytics для Windows.

>[!NOTE]
>Агент Log Analytics для Windows — это текущее название компонента Microsoft Monitoring Agent (MMA) в службе автоматизации Azure.

### <a name="scenario-an-exception-occurred-during-a-webclient-request"></a><a name="webclient-exception"></a>Сценарий. An exception occurred during a WebClient request (Исключение во время запроса WebClient)

Расширению Log Analytics для Windows на виртуальной машине не удается установить связь с внешними ресурсами, что приводит к ошибке развертывания.

#### <a name="issue"></a>Проблема

Ниже приведены примеры возвращаемых сообщений об ошибках.

```error
Please verify the VM has a running VM agent, and can establish outbound connections to Azure storage.
```

```error
'Manifest download error from https://<endpoint>/<endpointId>/Microsoft.EnterpriseCloud.Monitoring_MicrosoftMonitoringAgent_australiaeast_manifest.xml. Error: UnknownError. An exception occurred during a WebClient request.
```

#### <a name="cause"></a>Причина

Возможные причины возникновения этой ошибки перечислены ниже.

* Настроенный прокси-сервер виртуальной машины разрешает только определенные порты.
* Настройки брандмауэра, которые блокируют доступ к запрашиваемым адресам и портам.

#### <a name="resolution"></a>Решение

Убедитесь в том, что у вас открыты порты и адреса, которые необходимы для сеанса связи. Список портов и адресов можно узнать в разделе [Настройка сети](../automation-hybrid-runbook-worker.md#network-planning).

### <a name="scenario-install-failed-because-of-transient-environment-issues"></a><a name="transient-environment-issue"></a>Сценарий. Сбой установки из-за временных проблем со средой

Развертывание расширения Log Analytics для Windows завершилось сбоем, так как другой процесс установки или действие блокируют установку.

#### <a name="issue"></a>Проблема

Ниже приведены примеры возвращаемых сообщений об ошибках.

```error
The Microsoft Monitoring Agent failed to install on this machine. Please try to uninstall and reinstall the extension. If the issue persists, please contact support.
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1618'
```

```error
'Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.2) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.2\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 1601'
```

#### <a name="cause"></a>Причина

Возможные причины возникновения этой ошибки перечислены ниже.

* Запущен другой процесс установки.
* Во время развертывания шаблона была инициирована перезагрузка системы.

#### <a name="resolution"></a>Решение

Эта ошибка по своей природе является временной. Чтобы установить расширение, необходимо повторить развертывание.

### <a name="scenario-installation-timeout"></a><a name="installation-timeout"></a>Сценарий. Installation timeout (Превышено время ожидания при установке)

Установка расширения Log Analytics для Windows завершается сбоем из-за превышения времени ожидания.

#### <a name="issue"></a>Проблема

Ниже приведен пример сообщения об ошибке, которое может отображаться на экране.

```error
Install failed for plugin (name: Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent, version 1.0.11081.4) with exception Command C:\Packages\Plugins\Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent\1.0.11081.4\MMAExtensionInstall.exe of Microsoft.EnterpriseCloud.Monitoring.MicrosoftMonitoringAgent has exited with Exit code: 15614
```

#### <a name="cause"></a>Причина

Ошибки такого типа возникают из-за высокой нагрузки на виртуальную машину во время установки.

### <a name="resolution"></a>Решение

Попытайтесь установить расширение агента Log Analytics для Windows в период низкой загруженности виртуальной машины.

## <a name="next-steps"></a>Дальнейшие действия

Если вы не нашли здесь свою проблему или рекомендации не позволили ее решить, попробуйте использовать один из следующих каналов для получения дополнительной поддержки.

* Получите ответы специалистов Azure на [форумах Azure](https://azure.microsoft.com/support/forums/).
* Подпишитесь на [@AzureSupport](https://twitter.com/azuresupport) — официальный канал Microsoft Azure для работы с клиентами. Служба поддержки Azure взаимодействует с сообществом Azure, предоставляя ответы, поддержку и советы экспертов.
* Отправьте запрос в службу поддержки Azure Перейдите на [сайт поддержки Azure](https://azure.microsoft.com/support/options/) и выберите **Получить поддержку**.