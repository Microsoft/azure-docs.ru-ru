---
title: Журналы аудита аттестации Azure
description: Описание полных журналов аудита, собираемых для аттестации Azure
services: attestation
author: msmbaldwin
ms.service: attestation
ms.topic: reference
ms.date: 11/23/2020
ms.author: mbaldwin
ms.openlocfilehash: 1fa4a458a4e3e1df1d84c343a32e3a41a4a25e75
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95759027"
---
# <a name="audit-logs-for-azure-attestation"></a>Журналы аудита для аттестации Azure

Журналы аудита — это безопасные, неизменяемые записи с метками времени дискретных событий, произошедших с течением времени. Эти журналы записывают важные события, которые могут повлиять на функциональность экземпляра аттестации.

Аттестация Azure управляет экземплярами аттестации и связанными с ними политиками. Действия, связанные с управлением экземплярами и изменениями политики, подлежат аудиту и занесению в журнал.

Эта статья содержит сведения о регистрируемых событиях, собираемых данных и расположении этих журналов.

## <a name="about-audit-logs"></a>Сведения о журналах аудита

Аттестация Azure использует код для создания журналов аудита для событий, влияющих на способ выполнения аттестации. Это обычно сводится к тому, как или когда в экземпляр аттестации вносятся изменения политики, а также некоторые действия администратора.

### <a name="auditable-events"></a>События, поддерживающие аудит
Ниже приведены некоторые журналы аудита, которые мы соберем:

|     Событие или API                              |     Описание события                                                                         |
|--------------------------------------------|-----------------------------------------------------------------------------------------------|
|     создание экземпляра                        |     Создает новый экземпляр службы аттестации. |
|     Уничтожить экземпляр                       |     Уничтожает экземпляр службы аттестации. |
|     Добавление сертификата политики                 |     Добавление сертификата к текущему набору сертификатов управления политиками. |
|     Удаление сертификата политики              |     Удаление сертификата из текущего набора сертификатов управления политиками. |
|     Задать текущую политику                     |     Задает политику аттестации для заданного типа TEE. |
|     Сбросить политику аттестации               |     Сбрасывает политику аттестации для заданного типа TEE. |
|     Подготовка к обновлению политики               |     Подготовка к обновлению политики аттестации для заданного типа TEE. |
|     Восстановление клиентов после аварии       |     Повторно запечатывает все клиенты аттестации на этом экземпляре службы аттестации. Это может выполнить только администраторы службы аттестации. |

### <a name="collected--information"></a>Собранные сведения
Для каждого из этих событий аттестация Azure собирает следующие сведения:

- Имя операции
- Операция выполнена успешно
- Вызывающий объект операции, который может быть одним из следующих:
    - Имя участника-пользователя Azure AD
    - Идентификатор объекта
    - Certificate
    - Идентификатор клиента Azure AD
- Целевой объект операции, который может иметь одно из следующих значений:
    - Среда
    - Регион службы
    - Роль службы
    - Экземпляр роли службы
    - Идентификатор ресурса
    - Область ресурсов

### <a name="sample-audit-log"></a>Пример журнала аудита

Журналы аудита предоставляются в формате JSON. Ниже приведен пример того, как может выглядеть журнал аудита.

```json
{"operationName":"SetCurrentPolicy","resultType":"Success","resultDescription":null,"auditEventCategory":["ApplicationManagement"],"nCloud":null,"requestId":null,"callerIpAddress":null,"callerDisplayName":null,"callerIdentities":[{"callerIdentityType":"ObjectID","callerIdentity":"<some object ID>"},{"callerIdentityType":"TenantId","callerIdentity":"<some tenant ID>"}],"targetResources":[{"targetResourceType":"Environment","targetResourceName":"PublicCloud"},{"targetResourceType":"ServiceRegion","targetResourceName":"EastUS2"},{"targetResourceType":"ServiceRole","targetResourceName":"AttestationRpType"},{"targetResourceType":"ServiceRoleInstance","targetResourceName":"<some service role instance>"},{"targetResourceType":"ResourceId","targetResourceName":"/subscriptions/<some subscription ID>/resourceGroups/<some resource group name>/providers/Microsoft.Attestation/attestationProviders/<some instance name>"},{"targetResourceType":"ResourceRegion","targetResourceName":"EastUS2"}],"ifxAuditFormat":"Json","env_ver":"2.1","env_name":"#Ifx.AuditSchema","env_time":"2020-11-23T18:23:29.9427158Z","env_epoch":"MKZ6G","env_seqNum":1277,"env_popSample":0.0,"env_iKey":null,"env_flags":257,"env_cv":"##00000000-0000-0000-0000-000000000000_00000000-0000-0000-0000-000000000000_00000000-0000-0000-0000-000000000000","env_os":null,"env_osVer":null,"env_appId":null,"env_appVer":null,"env_cloud_ver":"1.0","env_cloud_name":null,"env_cloud_role":null,"env_cloud_roleVer":null,"env_cloud_roleInstance":null,"env_cloud_environment":null,"env_cloud_location":null,"env_cloud_deploymentUnit":null}
```

## <a name="access-audit-logs"></a>Доступ к журналам аудита

Эти журналы хранятся в Azure и недоступны напрямую. Если вам нужно получить доступ к этим журналам, отправьте запрос в службу поддержки. Дополнительные сведения см. в статье [Обращение в службу поддержки Майкрософт](https://azure.microsoft.com/support/options/). 

После того как обращение в службу поддержки будет завершено, корпорация Майкрософт загрузит и предоставит вам доступ к этим журналам.
