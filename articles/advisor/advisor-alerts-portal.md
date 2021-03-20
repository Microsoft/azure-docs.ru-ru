---
title: Создание оповещений Azure Advisor для получения новых рекомендаций с помощью портал Azure
description: Создание оповещений Помощника по Azure для новой рекомендации
ms.topic: article
ms.date: 09/09/2019
ms.openlocfilehash: 3c51479821914ef34edcd13d8708344169f17aae
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100590111"
---
# <a name="create-azure-advisor-alerts-on-new-recommendations-using-the-azure-portal"></a>Создание оповещений помощника Azure для новых рекомендаций с помощью портал Azure 

В этой статье показано, как настроить оповещение о новых рекомендациях из помощника по Azure с помощью портал Azure. 

Каждый раз, когда Помощник по Azure обнаруживает новую рекомендацию для одного из ваших ресурсов, событие сохраняется в [журнале действий Azure](../azure-monitor/essentials/platform-logs-overview.md). Вы можете настроить оповещения для этих событий из Помощника по Azure с помощью интерфейса создания оповещений для конкретных рекомендаций. Вы можете выбрать подписку и при необходимости группу ресурсов, чтобы определить ресурсы, по которым требуется получать оповещения. 

Вы также можете определить типы рекомендаций с помощью следующих свойств.

* Категория
* Уровень влияния
* Тип рекомендации

Вы также можете настроить действие, которое будет выполняться при активации оповещения одним из следующих действий:  

* выбор существующей группы действий;
* создание новой группы действий.

Дополнительные сведения о группах действий см. в разделе [Создание групп действий и управление ими на портале Azure](../azure-monitor/alerts/action-groups.md).

> [!NOTE] 
> Сейчас оповещения Помощника доступны только для рекомендаций, связанных с высоким уровнем доступности, производительностью и расходами. Рекомендации, связанные с безопасностью, не поддерживаются. 

## <a name="create-alert-rule"></a>Создание правила генерации оповещений
1. На **портале** выберите **Помощник по Azure**.

    ![Помощник по Azure на портале](./media/advisor-alerts/create1.png)

2. В разделе **мониторинг** в меню слева выберите **оповещения**. 

    ![Оповещения в помощнике](./media/advisor-alerts/create2.png)

3. Выберите **создать оповещение Advisor**.

    ![Новое оповещение Advisor](./media/advisor-alerts/create3.png)

4. В разделе **область** выберите подписку и, при необходимости, группу ресурсов, для которой необходимо создать оповещение. 

    ![Область оповещений Advisor](./media/advisor-alerts/create4.png)

5. В разделе **условие** выберите метод, который нужно использовать для настройки оповещения. Если вы хотите получать оповещения для всех рекомендаций по определенной категории или уровню влияния, выберите **категорию и уровень влияния**. Если вы хотите создать оповещение для всех рекомендаций определенного типа, выберите **тип рекомендации**.

    ![Условие оповещения Azure Advisor](./media/advisor-alerts/create5.png)

6. В зависимости от выбранного параметра Настройка в можно будет указать критерии. Если вам нужны все рекомендации, просто оставьте оставшиеся поля пустыми. 

    ![Группа действий оповещений Advisor](./media/advisor-alerts/create6.png)

7. В разделе **группы действий** выберите **Добавить существующий** , чтобы использовать уже созданную группу действий, или выберите **создать** , чтобы настроить новую [группу действий](../azure-monitor/alerts/action-groups.md). 

    ![Оповещение Advisor добавить существующий](./media/advisor-alerts/create7.png)

8. В разделе сведения о предупреждении введите имя и краткое описание оповещения. Если вы хотите, чтобы оповещение было включено, оставьте параметр **включить правило при создании** выбора со значением **Да**. Затем выберите группу ресурсов, в которую нужно сохранить оповещение. Это не повлияет на область определения целевой платформы рекомендации. 

    :::image type="content" source="./media/advisor-alerts/create8.png" alt-text="Снимок экрана с разделом сведений о предупреждении.":::


## <a name="configure-recommendation-alerts-to-use-a-webhook"></a>Настройка оповещений об рекомендациях для использования веб-перехватчика
В этом разделе показано, как настроить оповещения помощника Azure для отправки данных рекомендаций через веб-перехватчики в существующие системы. 

Вы можете настроить оповещения, которые будут уведомлены при наличии новой рекомендации Advisor для одного из ресурсов. Эти оповещения могут уведомлять вас по электронной почте или текстовому сообщению, но их также можно использовать для интеграции с существующими системами через веб-перехватчик. 


### <a name="using-the-advisor-recommendation-alert-payload"></a>Использование полезных данных оповещений Advisor
Если вы хотите интегрировать оповещения Advisor в свои системы с помощью веб-перехватчика, необходимо проанализировать полезные данные JSON, отправленные из уведомления. 

При настройке группы действий для этого предупреждения выбирается, если вы хотите использовать общую схему предупреждений. Если выбрать общую схему оповещений, полезная нагрузка будет выглядеть следующим образом: 

```json
{  
   "schemaId":"azureMonitorCommonAlertSchema",
   "data":{  
      "essentials":{  
         "alertId":"/subscriptions/<subid>/providers/Microsoft.AlertsManagement/alerts/<alerted>",
         "alertRule":"Webhhook-test",
         "severity":"Sev4",
         "signalType":"Activity Log",
         "monitorCondition":"Fired",
         "monitoringService":"Activity Log - Recommendation",
         "alertTargetIDs":[  
            "/subscriptions/<subid>/resourcegroups/<resource group name>/providers/microsoft.dbformariadb/servers/<resource name>"
         ],
         "originAlertId":"001d8b40-5d41-4310-afd7-d65c9d4428ed",
         "firedDateTime":"2019-07-17T23:00:57.3858656Z",
         "description":"A new recommendation is available.",
         "essentialsVersion":"1.0",
         "alertContextVersion":"1.0"
      },
      "alertContext":{  
         "channels":"Operation",
         "claims":"{\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress\":\"Microsoft.Advisor\"}",
         "caller":"Microsoft.Advisor",
         "correlationId":"8554b847-2a72-48ef-9776-600aca3c3aab",
         "eventSource":"Recommendation",
         "eventTimestamp":"2019-07-17T22:28:54.1566942+00:00",
         "httpRequest":"{\"clientIpAddress\":\"0.0.0.0\"}",
         "eventDataId":"001d8b40-5d41-4310-afd7-d65c9d4428ed",
         "level":"Informational",
         "operationName":"Microsoft.Advisor/recommendations/available/action",
         "properties":{  
            "recommendationSchemaVersion":"1.0",
            "recommendationCategory":"Performance",
            "recommendationImpact":"Medium",
            "recommendationName":"Increase the MariaDB server vCores",
            "recommendationResourceLink":"https://portal.azure.com/#blade/Microsoft_Azure_Expert/RecommendationListBlade/source/ActivityLog/recommendationTypeId/a5f888e3-8cf4-4491-b2ba-b120e14eb7ce/resourceId/%2Fsubscriptions%<subscription id>%2FresourceGroups%2<resource group name>%2Fproviders%2FMicrosoft.DBforMariaDB%2Fservers%2F<resource name>",
            "recommendationType":"a5f888e3-8cf4-4491-b2ba-b120e14eb7ce"
         },
         "status":"Active",
         "subStatus":"",
         "submissionTimestamp":"2019-07-17T22:28:54.1566942+00:00"
      }
   }
}
  ```

Если общая схема не используется, полезная нагрузка выглядит следующим образом: 

```json
{  
   "schemaId":"Microsoft.Insights/activityLogs",
   "data":{  
      "status":"Activated",
      "context":{  
         "activityLog":{  
            "channels":"Operation",
            "claims":"{\"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress\":\"Microsoft.Advisor\"}",
            "caller":"Microsoft.Advisor",
            "correlationId":"3ea7320f-c002-4062-adb8-96d3bd92a5f4",
            "description":"A new recommendation is available.",
            "eventSource":"Recommendation",
            "eventTimestamp":"2019-07-17T20:36:39.3966926+00:00",
            "httpRequest":"{\"clientIpAddress\":\"0.0.0.0\"}",
            "eventDataId":"a12b8e59-0b1d-4003-bfdc-3d8152922e59",
            "level":"Informational",
            "operationName":"Microsoft.Advisor/recommendations/available/action",
            "properties":{  
               "recommendationSchemaVersion":"1.0",
               "recommendationCategory":"Performance",
               "recommendationImpact":"Medium",
               "recommendationName":"Increase the MariaDB server vCores",
               "recommendationResourceLink":"https://portal.azure.com/#blade/Microsoft_Azure_Expert/RecommendationListBlade/source/ActivityLog/recommendationTypeId/a5f888e3-8cf4-4491-b2ba-b120e14eb7ce/resourceId/%2Fsubscriptions%2F<subscription id>%2FresourceGroups%2F<resource group name>%2Fproviders%2FMicrosoft.DBforMariaDB%2Fservers%2F<resource name>",
               "recommendationType":"a5f888e3-8cf4-4491-b2ba-b120e14eb7ce"
            },
            "resourceId":"/subscriptions/<subscription id>/resourcegroups/<resource group name>/providers/microsoft.dbformariadb/servers/<resource name>",
            "resourceGroupName":"<resource group name>",
            "resourceProviderName":"MICROSOFT.DBFORMARIADB",
            "status":"Active",
            "subStatus":"",
            "subscriptionId":"<subscription id>",
            "submissionTimestamp":"2019-07-17T20:36:39.3966926+00:00",
            "resourceType":"MICROSOFT.DBFORMARIADB/SERVERS"
         }
      },
      "properties":{  
 
      }
   }
}
```

В любой из схем можно задать события рекомендации помощника, выполнив поиск **eventSource** , `Recommendation` а **operationName**  — `Microsoft.Advisor/recommendations/available/action` .

Ниже приведены некоторые другие важные поля, которые можно использовать. 

* *алерттаржетидс* (в общей схеме) или *resourceId* (устаревшая схема)
* *рекоммендатионтипе*
* *рекоммендатионнаме*
* *рекоммендатионкатегори*
* *рекоммендатионимпакт*
* *рекоммендатионресаурцелинк*


## <a name="manage-your-alerts"></a>Управление оповещениями 

С помощью помощника Azure вы можете изменять, удалять или отключать и включать оповещения о рекомендациях. 

1. На **портале** выберите **Помощник по Azure**.

    :::image type="content" source="./media/advisor-alerts/create1.png" alt-text="Снимок экрана: меню &quot;портал Azure&quot;, в котором выбран помощник по Azure.":::

2. В разделе **мониторинг** в меню слева выберите **оповещения**.

    :::image type="content" source="./media/advisor-alerts/create2.png" alt-text="Снимок экрана: меню портал Azure, в котором отображаются выбранные оповещения.":::

3. Чтобы изменить оповещение, щелкните имя оповещения, чтобы открыть оповещение и изменить поля, которые нужно изменить.

4. Чтобы удалить, включить или отключить оповещение, щелкните многоточие в конце строки, а затем выберите действие, которое хотите выполнить.
 

## <a name="next-steps"></a>Дальнейшие действия
- Изучите [обзор оповещений журнала действий](../azure-monitor/alerts/alerts-overview.md) и узнайте, как получать оповещения.
- Дополнительные сведения о группах действий см. в статье [Создание групп действий и управление ими на портале Azure](../azure-monitor/alerts/action-groups.md).
