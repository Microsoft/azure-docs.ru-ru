---
title: Устранение неполадок SQL Insights (Предварительная версия)
description: Устранение неполадок SQL Insights в Azure Monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/04/2021
ms.openlocfilehash: 85a3505dd347b96036c28c85c089afa04e3e3bd5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104610109"
---
# <a name="troubleshooting-sql-insights-preview"></a>Устранение неполадок SQL Insights (Предварительная версия)
Чтобы устранить проблемы с сбором данных в SQL Insights, проверьте состояние компьютера мониторинга на вкладке **Управление профилями** . Оно будет иметь одно из следующих состояний:

- Идет 
- Не собираются 
- Сбор с ошибками 
 
Щелкните **состояние** детализации, чтобы просмотреть журналы и дополнительные сведения, которые могут помочь в устранении проблемы. 

:::image type="content" source="media/sql-insights-enable/monitoring-machine-status.png" alt-text="Наблюдение за состоянием компьютера":::

## <a name="not-collecting-state"></a>Не собирает состояние 
Компьютер мониторинга находится в состоянии *не собираются* , если в *инсигхтсметрикс* для SQL нет данных за последние 10 минут. 

SQL Insights использует следующий запрос для получения этих сведений:

```
InsightsMetrics 
    | extend Tags = todynamic(Tags) 
    | extend SqlInstance = tostring(Tags.sql_instance) 
    | where TimeGenerated > ago(10m) and isnotempty(SqlInstance) and Namespace == 'sqlserver_server_properties' and Name == 'uptime' 
```

Проверьте, есть ли журналы из Telegraf, которые помогут определить причину проблемы. Если имеются записи журнала, можно щелкнуть *не собирать* и проверить журналы и сведения об устранении неполадок для распространенных проблем. 


Если журналы отсутствуют, необходимо проверить журналы на виртуальной машине мониторинга для следующих служб, установленных двумя расширениями виртуальной машины:

- Microsoft. Azure. Monitor. Азуремониторлинуксажент 
  - Служба: мдсд 
- Microsoft. Azure. Monitor. рабочей нагрузки. Рабочая нагрузка. Влилинуксекстенсион 
  - Служба: вли 
  - Служба: MS-Telegraf 
  - Служба: TD-Agent-bit-вли 
  - Журнал расширений для проверки сбоев при установке:/Вар/лог/Азуре/Микрософт.Азуре.монитор.ворклоадс.ворклоад.влилинуксекстенсион/влилогс.лог 



### <a name="wli-service-logs"></a>журналы службы вли 

Журналы службы: `/var/log/wli.log`

Чтобы просмотреть последние журналы, выполните следующие действия. `tail -n 100 -f /var/log/wli.log`
 

Если вы видите следующий журнал ошибок, это указывает на проблему со службой **мдсд** .

```
2021-01-27T06:09:28Z [Error] Failed to get config data. Error message: dial unix /var/run/mdsd/default_fluent.socket: connect: no such file or directory 
```


### <a name="telegraf-service-logs"></a>Журналы службы Telegraf 

Журналы службы: `/var/log/ms-telegraf/telegraf.log`

Чтобы просмотреть последние журналы `tail -n 100 -f /var/log/ms-telegraf/telegraf.log` ошибок и предупреждений, выполните следующие действия. `tail -n 1000 /var/log/ms-telegraf/telegraf.log | grep "E\!\|W!"`

 Конфигурация, используемая Telegraf, создается службой вли и помещается в: `/etc/ms-telegraf/telegraf.d/wli`
 
Если при создании неверной конфигурации служба MS-Telegraf может не запуститься, проверьте, запущена ли служба MS-Telegraf с помощью команды: `service ms-telegraf status`

Чтобы просмотреть сообщения об ошибках из службы Telegraf, запустите ее вручную с помощью следующей команды: 

```
/usr/bin/ms-telegraf --config /etc/ms-telegraf/telegraf.conf --config-directory /etc/ms-telegraf/telegraf.d/wli --test 
```

### <a name="mdsd-service-logs"></a>журналы службы мдсд 

Проверьте [текущие ограничения](../agents/azure-monitor-agent-overview.md#current-limitations) для агента Azure Monitor. 


Журналы службы:  
- `/var/log/mdsd.err`
- `/var/log/mdsd.warn`
- `/var/log/mdsd.info`

Чтобы просмотреть последние ошибки, выполните следующие действия. `tail -n 100 -f /var/log/mdsd.err`

 Если необходимо обратиться в службу поддержки, собирайте следующие сведения: 

- Вход в систему `/var/log/azure/Microsoft.Azure.Monitor.AzureMonitorLinuxAgent/` 
- Войти `/var/log/waagent.log` 
- Вход в систему `/var/log/mdsd*`
- Файлы в `/etc/mdsd.d/`
- File `/etc/default/mdsd`

### <a name="invalid-monitoring-virtual-machine-configuration"></a>Недопустимая конфигурация виртуальной машины мониторинга

Одна из причин невозможности *сбора* данных — при наличии недействительной конфигурации виртуальной машины мониторинга.  Ниже приведена конфигурация по умолчанию.

```json
{
    "version": 1,
    "secrets": {
        "telegrafPassword": {
            "keyvault": "https://mykeyvault.vault.azure.net/",
            "name": "sqlPassword"
        }
    },
    "parameters": {
        "sqlAzureConnections": [
            "Server=mysqlserver.database.windows.net;Port=1433;Database=mydatabase;User Id=telegraf;Password=$telegrafPassword;"
        ],
        "sqlVmConnections": [
        ],
        "sqlManagedInstanceConnections": [
        ]
    }
}
```

Эта конфигурация задает токены замены, которые будут использоваться в конфигурации профиля виртуальной машины мониторинга. Он также позволяет ссылаться на секреты из Azure Key Vault, поэтому вы не храните секретные значения в какой-либо конфигурации, что настоятельно рекомендуется.

#### <a name="secrets"></a>Секреты
Секреты — это токены, значения которых извлекаются во время выполнения из Azure Key Vault. Секрет определяется парой Key Vault ссылок и имени секрета. Это позволяет Azure Monitor получить динамическое значение секрета и использовать его в качестве подчиненных ссылок на конфигурацию.

В конфигурации можно определить столько секретов, сколько необходимо, включая секреты, хранящиеся в отдельных хранилищах ключей.

```json
   "secrets": {
        "<secret-token-name-1>": {
            "keyvault": "<key-vault-uri>",
            "name": "<key-vault-secret-name>"
        },
        "<secret-token-name-2>": {
            "keyvault": "<key-vault-uri-2>",
            "name": "<key-vault-secret-name-2>"
        }
    }
```

Разрешения на доступ к Key Vault предоставляются Управляемое удостоверение службы на виртуальной машине мониторинга. Azure Monitor предполагало, что Key Vault предоставить как минимум секретные разрешения GET для виртуальной машины. Его можно включить с помощью шаблона портал Azure, PowerShell, интерфейса командной строки или диспетчер ресурсов.

#### <a name="parameters"></a>Параметры
Параметры — это токены, на которые можно ссылаться в конфигурации профиля с помощью шаблонов JSON. Параметры имеют имя и значение. Значения могут быть любого типа JSON, включая объекты и массивы. На параметр в конфигурации профиля указывает ссылка с использованием его имени в этом соглашении `.Parameters.<name>` .

Параметры могут ссылаться на секреты в Key Vault с использованием того же соглашения. Например, `sqlAzureConnections` ссылается на секрет, `telegrafPassword` используя соглашение `$telegrafPassword` .

Во время выполнения все параметры и секреты будут разрешены и объединены с конфигурацией профиля для создания реальной конфигурации, которая будет использоваться на компьютере.

> [!NOTE]
> Имена параметров `sqlAzureConnections` , `sqlVmConnections` и `sqlManagedInstanceConnections` являются обязательными в конфигурации, даже если нет строк подключения, которые будут предоставляться для некоторых из них.


## <a name="collecting-with-errors-state"></a>Сбор с состоянием ошибок
Компьютер мониторинга будет находиться в состоянии *сбора с ошибками* , если существует по крайней мере один журнал *инсигхтсметрикс* , но в таблице *операций* есть ошибки.

SQL Insights использует следующие запросы для получения этих сведений:

```
InsightsMetrics 
    | extend Tags = todynamic(Tags) 
    | extend SqlInstance = tostring(Tags.sql_instance) 
    | where TimeGenerated > ago(240m) and isnotempty(SqlInstance) and Namespace == 'sqlserver_server_properties' and Name == 'uptime' 
```

```
Operation 
 | where OperationCategory == "WorkloadInsights" 
 | summarize Errors = countif(OperationStatus == 'Error') 
```

В общих случаях мы предоставляем базу знаний по устранению неполадок в нашем представлении журналов: 

:::image type="content" source="media/sql-insights-enable/troubleshooting-logs-view.png" alt-text="Просмотр журналов устранения неполадок.":::



## <a name="next-steps"></a>Дальнейшие действия

- Получите сведения о [включении SQL Insights](sql-insights-enable.md).
