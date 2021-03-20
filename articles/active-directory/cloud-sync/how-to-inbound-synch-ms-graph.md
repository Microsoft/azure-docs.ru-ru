---
title: Как программно настроить синхронизацию в облаке с помощью MS API Graph
description: В этом разделе описано, как включить входящую синхронизацию, используя только API Graph
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 12/04/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6c84636ea86b3b640aef365c1c5d8e634b9a1f48
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99593168"
---
# <a name="how-to-programmatically-configure-cloud-sync-using-ms-graph-api"></a>Как программно настроить синхронизацию в облаке с помощью MS API Graph

В следующем документе описано, как реплицировать профиль синхронизации с нуля с помощью только API-интерфейсов Мсграф.  
Структура того, как это сделать, состоит из следующих шагов.  Их можно сформулировать следующим образом.

- [Базовая настройка](#basic-setup)
- [Создание субъектов-служб](#create-service-principals)
- [Создание задания синхронизации](#create-sync-job)
- [Обновление целевого домена](#update-targeted-domain)
- [Включить хэши паролей синхронизации](#enable-sync-password-hashes-on-configuration-blade)
- [Случайные удаления](#accidental-deletes)
- [Запуск задания синхронизации](#start-sync-job)
- [Состояние проверки](#review-status)

Используйте эти [модуль Microsoft Azure Active Directory для Windows PowerShell](/powershell/module/msonline/) команды, чтобы включить синхронизацию для рабочего клиента — это предварительная версия для возможности вызова веб-службы администрирования для этого клиента.

## <a name="basic-setup"></a>Базовая настройка

### <a name="enable-tenant-flags"></a>Включить флаги клиента

 ```PowerShell
 Connect-MsolService ('-AzureEnvironment <AzureEnvironmnet>')
 Set-MsolDirSyncEnabled -EnableDirSync $true
 ```
Первая из этих двух команд требует Azure Active Directory учетных данных. Эти командлеты неявно указывают клиента и включают его для синхронизации.

## <a name="create-service-principals"></a>Создание субъектов-служб
Далее необходимо создать [приложение AD2AAD или субъект-службу](/graph/api/applicationtemplate-instantiate?view=graph-rest-beta&tabs=http) .

Необходимо использовать идентификатор приложения 1a4721b3-e57f-4451-ae87-ef078703ec94. DisplayName — это URL-адрес домена AD, если он используется на портале (например, contoso.com), но может называться что-то другое.

 ```
 POST https://graph.microsoft.com/beta/applicationTemplates/1a4721b3-e57f-4451-ae87-ef078703ec94/instantiate
 Content-type: application/json
 {
    displayName: [your app name here]
 }
 ```


## <a name="create-sync-job"></a>Создание задания синхронизации
Выходные данные приведенной выше команды будут возвращать идентификатор объекта созданного субъекта-службы. В этом примере objectId имеет значение 614ac0e9-a59b-481f-bd8f-79a73d167e1c.  Используйте Microsoft Graph, чтобы добавить Синчронизатионжоб к этому субъекту-службе.  

Документацию по созданию задания синхронизации можно найти [здесь](/graph/api/synchronization-synchronizationjob-post?tabs=http&view=graph-rest-beta).

Если вы не зарегистрировали идентификатор, указанный выше, можно найти субъект-службу, выполнив следующий вызов MS Graph. Для этого потребуется разрешение Directory. Read. ALL.
 
 `GET https://graph.microsoft.com/beta/servicePrincipals `

Затем найдите имя приложения в выходных данных.

Выполните следующие две команды, чтобы создать два задания: одно для подготовки пользователей или групп, а другое для синхронизации хэша паролей. Это тот же запрос дважды, но с разными идентификаторами шаблонов.


Вызовите следующие два запроса:

 ```
 POST https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs
 Content-type: application/json
 {
 "templateId":"AD2AADProvisioning"
 } 
 ```

 ```
 POST https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs
 Content-type: application/json
 {
 "templateId":"AD2AADPasswordHash"
 }
 ```

Если вы хотите создать оба метода, потребуется два вызова.

Пример возвращаемого значения (для подготовки):

 ```
HTTP 201/Created
{
    "@odata.context": "https://graph.microsoft.com/beta/$metadata#servicePrincipals('614ac0e9-a59b-481f-bd8f-79a73d167e1c')/synchronization/jobs/$entity",
    "id": "AD2AADProvisioning.fc96887f36da47508c935c28a0c0b6da",
    "templateId": "ADDCInPassthrough",
    "schedule": {
        "expiration": null,
        "interval": "PT40M",
        "state": "Disabled"
    },
    "status": {
        "countSuccessiveCompleteFailures": 0,
        "escrowsPruned": false,
        "code": "Paused",
        "lastExecution": null,
        "lastSuccessfulExecution": null,
        "lastSuccessfulExecutionWithExports": null,
        "quarantine": null,
        "steadyStateFirstAchievedTime": "0001-01-01T00:00:00Z",
        "steadyStateLastAchievedTime": "0001-01-01T00:00:00Z",
        "troubleshootingUrl": null,
        "progress": [],
        "synchronizedEntryCountByType": []
    }
}
```

## <a name="update-targeted-domain"></a>Обновление целевого домена
Для этого клиента идентификатор объекта и идентификатор приложения субъекта-службы имеют следующий вид:

ObjectId: 8895955e-2e6c-4d79-8943-4d72ca36878f AppId: 00000014-0000-0000-C000-000000000000 DisplayName: testApp

Необходимо обновить домен, для которого предназначена эта конфигурация, поэтому обновите секреты для этого домена.

Убедитесь, что используемое доменное имя совпадает с URL-адресом, заданным для локального контроллера домена.

 ```
 PUT – https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/secrets
 ```
 Добавьте следующую пару "ключ-значение" в массиве значений ниже, исходя из того, что вы пытаетесь сделать:
 - Включить флаги клиента PHS и Sync {ключ: "AppKey", значение: "{" Аппкэйсценарио ":" AD2AADPasswordHash "}"}
 
 - Включить только флаг клиента синхронизации (не включать PHS) {ключ: "AppKey", значение: "{" Аппкэйсценарио ":" AD2AADProvisioning "}"}
 ```
 Request body –
 {
    "value": [
               {
                 "key": "Domain",
                 "value": "{\"domain\":\"ad2aadTest.com\"}"
               }
             ]
  }
```

Ожидаемый ответ:... HTTP 204/нет содержимого

Здесь выделенное значение "домен" — это имя локального домена Active Directory, из которого должны быть подготовлены записи для Azure Active Directory.

## <a name="enable-sync-password-hashes-on-configuration-blade"></a>Включить хэширование паролей синхронизации в колонке конфигурации

 В этом разделе рассматривается Включение синхронизации хэшей паролей для конкретной конфигурации. Это отличается от секрета AppKey, который включает флаг функции на уровне клиента. Это относится только к одному домену или конфигурации. Вам потребуется задать для ключа приложения PHS один, чтобы он работал в конце.

1. Захватить схему (предупреждение это довольно большое) 
 ```
 GET –https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ [AD2AADProvisioningJobId]/schema
 ```
2. Выполните сопоставление атрибута Кредентиалдата:
 ``` 
 {
 "defaultValue": null,
 "exportMissingReferences": false,
 "flowBehavior": "FlowWhenChanged",
 "flowType": "Always",
 "matchingPriority": 0,
 "targetAttributeName": "CredentialData",
 "source": {
 "expression": "[PasswordHash]",
 "name": "PasswordHash",
 "type": "Attribute",
 "parameters": []
 }
 ```
3. Найдите следующие сопоставления объектов со следующими именами в схеме.
 - Подготавливайте пользователей Active Directory
 - Инициализация Active Directory Инеторгперсонс

 Сопоставления объектов находятся в схеме. Синчронизатионрулес [0]. Обжектмаппингс (для теперь можно предположить, что существует только 1 правило синхронизации).

4. Выполните сопоставление Кредентиалдата из шага (2) и вставьте его в сопоставления объектов на шаге (3)

 Сопоставление объектов выглядит примерно так:
 ```
 {
 "enabled": true,
 "flowTypes": "Add,Update,Delete",
 "name": "Provision Active Directory users",
 "sourceObjectName": "user",
 "targetObjectName": "User",
 "attributeMappings": [
 ...
 } 
 ```
 Скопируйте или вставьте сопоставление из шага **создания заданий AD2AADProvisioning и AD2AADPasswordHash** выше в массив аттрибутемаппингс. 

 Порядок элементов в этом массиве не важен (серверная часть будет сортироваться за вас). Будьте внимательны при добавлении этого сопоставления атрибута, если такое имя уже существует в массиве (например, если в Аттрибутемаппингс уже есть элемент, имеющий Таржетаттрибутенаме Кредентиалдата), могут возникать ошибки конфликта, а также можно объединять существующие и новые сопоставления (обычно нежелательный результат). Серверная часть не выполняет дедупликацию. 

 Не забудьте сделать это как для пользователей, так и для Инеторгперсонс

5. Сохранение созданной схемы 
 ```
 PUT –
 https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ [AD2AADProvisioningJobId]/schema
```

 Добавьте схему в текст запроса. 

## <a name="accidental-deletes"></a>Случайные удаления
В этом разделе рассматривается программное включение, отключение и использование [случайных удалений](how-to-accidental-deletes.md) .


### <a name="enabling-and-setting-the-threshold"></a>Включение и установка порогового значения
Существует два параметра для каждого задания, которые можно использовать.

 - Делетесрешолденаблед — включает предотвращение случайного удаления для задания, если задано значение "true". По умолчанию имеет значение true.
 - Делетесрешолдвалуе — определяет максимальное число удалений, которые будут разрешены при каждом выполнении задания, если включена защита от случайного удаления. По умолчанию задано значение 500.  Таким образом, если задано значение 500, максимальное количество удалений будет составлять 499 при каждом выполнении.

Параметры порога удаления являются частью компонента `SyncNotificationSettings` и могут быть изменены с помощью Graph. 

Нам нужно обновить Синкнотификатионсеттингс, для которого предназначена эта конфигурация, поэтому обновите секреты.

 ```
 PUT – https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/secrets
 ```

 Добавьте следующую пару "ключ-значение" в массиве значений ниже, исходя из того, что вы пытаетесь сделать:

```
 Request body -
 {
   "value":[
             {
               "key":"SyncNotificationSettings",
               "value": "{\"Enabled\":true,\"Recipients\":\"foobar@xyz.com\",\"DeleteThresholdEnabled\":true,\"DeleteThresholdValue\":50}"
              }
            ]
  }


```

Параметр "включено" в приведенном выше примере предназначен для включения и отключения уведомлений по электронной почте, когда задание помещено в карантин.


Сейчас мы не поддерживаем запросы на исправление для секретов, поэтому для сохранения других значений необходимо добавить все значения в текст запроса размещения (как в примере выше).

Существующие значения для всех секретов можно получить с помощью команды. 

```
GET https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/secrets 
```

### <a name="allowing-deletes"></a>Разрешение операций удаления
Чтобы разрешить удаленное выполнение операций удаления после того, как задание перейдет в карантин, необходимо выполнить перезагрузку, указав в качестве области только "Форцеделетес". 

```
Request:
POST https://graph.microsoft.com/beta/servicePrincipals/{id}/synchronization/jobs/{jobId}/restart
```

```
Request Body:
{
  "criteria": {"resetScope": "ForceDeletes"}
}
```






## <a name="start-sync-job"></a>Запуск задания синхронизации
Задание можно получить снова с помощью следующей команды:

 `GET https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ ` 

Документацию по получению заданий можно найти [здесь](/graph/api/synchronization-synchronizationjob-list?tabs=http&view=graph-rest-beta). 
 
Чтобы запустить задание, выдайте этот запрос, используя objectId субъекта-службы, созданного на первом шаге, и идентификатор задания, полученный из запроса, создавшего задание.

Документацию по запуску задания можно найти [здесь](/graph/api/synchronization-synchronizationjob-start?tabs=http&view=graph-rest-beta). 

 ```
 POST  https://graph.microsoft.com/beta/servicePrincipals/8895955e-2e6c-4d79-8943-4d72ca36878f/synchronization/jobs/AD2AADProvisioning.fc96887f36da47508c935c28a0c0b6da/start
 ```

Ожидаемый ответ:... HTTP 204/нет содержимого.

Другие команды для управления заданием описаны [здесь](/graph/api/resources/synchronization-synchronizationjob?view=graph-rest-beta).
 
Чтобы перезапустить задание, в одном из них будет использоваться...

 ```
 POST  https://graph.microsoft.com/beta/servicePrincipals/8895955e-2e6c-4d79-8943-4d72ca36878f/synchronization/jobs/AD2AADProvisioning.fc96887f36da47508c935c28a0c0b6da/restart
 {
   "criteria": {
       "resetScope": "Full"
   }
 }
 ```

## <a name="review-status"></a>Состояние проверки
Получение состояния задания с помощью...

 ```
 GET https://graph.microsoft.com/beta/servicePrincipals/[SERVICE_PRINCIPAL_ID]/synchronization/jobs/ 
 ```

Просмотрите раздел "Status" возвращаемого объекта, чтобы получить соответствующие сведения.

## <a name="next-steps"></a>Дальнейшие действия 

- [Что представляет собой облачная синхронизация Azure AD Connect?](what-is-cloud-sync.md)
- [Преобразования](how-to-transformation.md)
- [API синхронизации Azure AD](/graph/api/resources/synchronization-overview?view=graph-rest-beta)