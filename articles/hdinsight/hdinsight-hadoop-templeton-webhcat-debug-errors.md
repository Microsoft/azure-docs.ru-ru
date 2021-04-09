---
title: Понимание и устранение ошибок WebHCat в HDInsight в Azure
description: Узнайте о наиболее распространенных ошибках, возвращаемых WebHCat в HDInsight, и о способах их устранения.
ms.service: hdinsight
ms.topic: troubleshooting
ms.custom: hdinsightactive
ms.date: 04/14/2020
ms.openlocfilehash: ce7a21a2b5401c68c6b58bc934fb20b0225ba8ad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "98945467"
---
# <a name="understand-and-resolve-errors-received-from-webhcat-on-hdinsight"></a>Понимание и устранение ошибок, полученных из WebHCat в HDInsight

Дополнительные сведения об ошибках, возникающих при использовании WebHCat в HDInsight, а также способы их устранения. WebHCat используется во внутренних процессах таких клиентских инструментов, как Azure PowerShell и средства Azure Data Lake для Visual Studio.

## <a name="what-is-webhcat"></a>Что такое WebHCat

[WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat) — это REST API для [HCatalog](https://cwiki.apache.org/confluence/display/Hive/HCatalog), уровень управления таблицами и хранилищем данных для Apache Hadoop. WebHCat активирован по умолчанию в кластерах HDInsight и используется различными средствами для отправки заданий, получения состояния задания и т. д. без входа в кластер.

## <a name="modifying-configuration"></a>Изменение конфигурации

Некоторые из ошибок, приведенных в этом документе, возникают из-за превышения установленного максимума. Если на этапе разрешения говорится, что можно изменить значение, используйте Apache Ambari (онлайн-версию или REST API) для изменения значения. Дополнительные сведения см. в разделе [Управление кластерами HDInsight с помощью веб-интерфейса Ambari](hdinsight-hadoop-manage-ambari.md).

### <a name="default-configuration"></a>Конфигурация по умолчанию

При превышении следующих значений по умолчанию может снизиться производительность WebHCat или могут возникнуть ошибки.

| Параметр | Действие | Значение по умолчанию |
| --- | --- | --- |
| [yarn.scheduler.capacity.maximum-applications][maximum-applications] |Максимальное количество заданий, которые могут быть активны одновременно (в состоянии ожидания или выполнения) |10 000 |
| [templeton.exec.max-procs][max-procs] |Максимальное число запросов, которые могут обрабатываться параллельно |20 |
| [mapreduce.jobhistory.max-age-ms][max-age-ms] |Число дней, в течение которых хранится журнал заданий. |7 дней |

## <a name="too-many-requests"></a>Слишком много запросов

**Код состояния HTTP**: 429

| Причина | Решение |
| --- | --- |
| Превышено максимальное число одновременных запросов, обслуживаемых WebHCat в минуту (по умолчанию 20) |Уменьшите рабочую нагрузку и убедитесь, что максимальное количество параллельных запросов не превышено, или увеличьте количество одновременных запросов путем изменения `templeton.exec.max-procs`. Для получения дополнительных сведений ознакомьтесь с разделом [Изменение конфигурации](#modifying-configuration). |

## <a name="server-unavailable"></a>Сервер недоступен

**Код состояния HTTP**: 503

| Причина | Решение |
| --- | --- |
| Обычно этот код состояния отображается во время обработки отказа между первичным и вторичным головным узлом кластера. |Подождите две минуты, а затем повторите операцию |

## <a name="bad-request-content-could-not-find-job"></a>Недопустимый запрос. Содержимое: не удалось найти задание

**Код состояния HTTP**: 400

| Причина | Решение |
| --- | --- |
| Сведения о задании были удалены утилитой очистки журнала заданий |Срок хранения по умолчанию для журнала заданий составляет 7 дней. Его можно изменить, внеся изменения в `mapreduce.jobhistory.max-age-ms`. Для получения дополнительных сведений ознакомьтесь с разделом [Изменение конфигурации](#modifying-configuration). |
| Задание был прекращено из-за отработки отказа |Повторить попытку отправки задания через две минуты. |
| Использован недопустимый идентификатор задания |Проверьте, правильно ли указан идентификатор задания |

## <a name="bad-gateway"></a>Недопустимый шлюз

**Код состояния HTTP**: 502

| Причина | Решение |
| --- | --- |
| Внутренний сбор мусора происходит в процессе WebHCat |Дождитесь завершения процесса сбора мусора или перезапустите службу WebHCat |
| Превышено время ожидания ответа от службы Resource Manager. Эта ошибка может произойти, когда число активных приложений становится выше заданного максимума (по умолчанию 10 000). |Дождитесь завершения выполняемых заданий или увеличьте ограничение на количество одновременно выполняемых заданий, изменив `yarn.scheduler.capacity.maximum-applications`. Дополнительные сведения см. в разделе [Изменение конфигурации](#modifying-configuration). |
| Выполнена попытка получить все задания вызовом [GET /Jobs](https://cwiki.apache.org/confluence/display/Hive/WebHCat+Reference+Jobs) при `Fields` со значением `*`. |Не пытайтесь получить сведения обо *всех* заданиях. Вместо этого используйте `jobid` для получения сведений только о тех заданиях, идентификатор которых больше определенного значения. Или не используйте `Fields` |
| Служба WebHCat не работает во время отработки отказа узла HeadNode |Подождите 2 минуты и повторите попытку |
| В настоящий момент более 500 заданий, отправленных через WebHCat, находятся в режиме ожидания |Дождитесь завершения текущих заданий, находящихся в режиме ожидания, перед отправкой новых заданий |

## <a name="next-steps"></a>Дальнейшие действия

[!INCLUDE [troubleshooting next steps](../../includes/hdinsight-troubleshooting-next-steps.md)]

[maximum-applications]: https://docs.cloudera.com/HDPDocuments/HDP2/HDP-2.1.3/bk_system-admin-guide/content/setting_application_limits.html
[max-procs]: https://cwiki.apache.org/confluence/display/Hive/WebHCat+Configure#WebHCatConfigure-WebHCatConfiguration
[max-age-ms]: https://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.0.6.0/ds_Hadoop/hadoop-mapreduce-client/hadoop-mapreduce-client-core/mapred-default.xml