---
title: Справочник по интеграции Azure Active Directory и Workday
description: Технический подробный обзор подготовки с учетом часового часа
services: active-directory
author: cmmdesai
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 02/09/2021
ms.author: chmutali
ms.openlocfilehash: 2b1a43ee6b13d32c0eaed92538cf9c25405e061b
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100104337"
---
# <a name="how-azure-active-directory-provisioning-integrates-with-workday"></a>Как Azure Active Directory подготовка интегрируется с Workday

[Azure Active Directory служба подготовки пользователей](../app-provisioning/user-provisioning.md) интегрируется с [Workday HCM](https://www.workday.com) для управления жизненным циклом удостоверений пользователей. Azure Active Directory предлагает три предварительно созданных интеграции: 

* [Подготовка рабочего дня к локальной Active Directory подготовки пользователей](../saas-apps/workday-inbound-tutorial.md)
* [Рабочий день для Azure Active Directory подготовки пользователей](../saas-apps/workday-inbound-cloud-only-tutorial.md)
* [Обратная запись Workday](../saas-apps/workday-writeback-tutorial.md)

В этой статье объясняется, как работает интеграция и как можно настроить поведение подготовки для различных сценариев HR. 

## <a name="establishing-connectivity"></a>Установление подключения 

### <a name="restricting-workday-api-access-to-azure-ad-endpoints"></a>Ограниченный доступ к конечным точкам Azure AD через API Workday
Служба подготовки Azure AD использует обычную проверку подлинности для подключения к конечным точкам API веб-служб Workday.  

Чтобы обеспечить безопасность подключения между службой подготовки Azure AD и Workday, можно ограничить доступ таким образом, чтобы назначенный пользователь системы интеграции мог получить доступ к API Workday только из разрешенных диапазонов IP-адресов Azure AD. Обратитесь к администратору Workday, чтобы выполнить следующую настройку в клиенте Workday. 

1. Скачайте [последние диапазоны IP-адресов](https://www.microsoft.com/download/details.aspx?id=56519) для общедоступного облака Azure. 
1. Откройте файл и выполните поиск тега **AzureActiveDirectory** 

   >[!div class="mx-imgBorder"] 
   >![Диапазон IP-адресов Azure AD](media/sap-successfactors-integration-reference/azure-active-directory-ip-range.png)

1. Скопируйте все диапазоны IP-адресов, перечисленные в элементе *аддресспрефиксес* , и используйте диапазон для создания списка IP-адресов.
1. Войдите на портал администрирования Workday. 
1. Чтобы создать новый диапазон IP-адресов для центров обработки данных Azure, можно обратиться к задаче **Ведение диапазонов IP** -адресов. Укажите диапазоны IP-адресов (в нотации CIDR) в виде списка с разделителями-запятыми.  
1. Чтобы создать политику проверки подлинности, необходимо получить доступ к задаче **Управление политиками проверки подлинности** . В политике аутентификации используйте список разрешить проверку подлинности, чтобы указать диапазон IP-адресов Azure AD и группу безопасности, которым будет разрешен доступ из этого диапазона IP-адресов. Сохраните изменения. 
1. Чтобы подтвердить изменения, необходимо получить доступ к задаче **Активация изменений политики незавершенной проверки подлинности** .

### <a name="limiting-access-to-worker-data-in-workday-using-constrained-security-groups"></a>Ограничение доступа к данным рабочих ролей в Workday с помощью ограниченных групп безопасности

По умолчанию действия по [настройке пользователя системы интеграции Workday](../saas-apps/workday-inbound-tutorial.md#configure-integration-system-user-in-workday) предоставляют доступ для получения всех пользователей в клиенте workday. В некоторых сценариях интеграции может потребоваться ограничить доступ, чтобы пользователи, принадлежащие только к определенным организациям, возвращались вызовом API Get_Workers и обрабатывались соединителем Azure AD Workday. 

Это требование можно выполнить, работая с администратором Workday и настроив группы безопасности системы с ограниченной интеграцией. Дополнительные сведения о том, как это сделать, см. в [этой статье сообщества workday](https://community.workday.com/forums/customer-questions/620393) (*для доступа к этой статье требуются учетные данные для входа в сообщество workday*).

Эта стратегия ограничения доступа с помощью ограниченных ИССГ (групп безопасности системы интеграции) особенно полезна в следующих сценариях: 
* **Сценарий поэтапного развертывания**. у вас есть большой клиент Workday и планируется выполнить поэтапный выпуск workday в автоматическую подготовку Azure AD. В этом сценарии вместо того, чтобы исключать пользователей, которые не находятся в области текущего этапа с фильтрами области Azure AD, рекомендуется настроить ограниченный ИССГ, чтобы только рабочие процессы в области были видны в Azure AD.
* **Сценарий с несколькими заданиями подготовки**: у вас есть большой клиент workday и несколько доменов AD, каждый из которых поддерживает разные подразделения, подразделения и компании. Для поддержки этой топологии вы хотите выполнить несколько заданий подготовки к работе в Azure AD с каждой задачей, подготовкой определенного набора рабочих ролей. В этом сценарии вместо использования фильтров области Azure AD для исключения рабочих данных рекомендуется настроить ограниченные ИССГ, чтобы только соответствующие рабочие данные были видимы в Azure AD. 

### <a name="workday-test-connection-query"></a>Запрос на тестирование подключения Workday

Чтобы проверить подключение к Workday, Azure AD отправляет следующий запрос на веб-службы *Get_Workers* Workday. 

```XML
<!-- Test connection query tries to retrieve one record from the first page -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to the test connection event -->
<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Date_Range_Data>
        <p1:Updated_From>2021-01-19T02:28:50.1491022Z</p1:Updated_From>
        <p1:Updated_Through>2021-01-19T02:28:50.1491022Z</p1:Updated_Through>
      </p1:Transaction_Date_Range_Data>
    </p1:Transaction_Log_Criteria_Data>
    <p1:Exclude_Employees>true</p1:Exclude_Employees>
    <p1:Exclude_Contingent_Workers>true</p1:Exclude_Contingent_Workers>
    <p1:Exclude_Inactive_Workers>true</p1:Exclude_Inactive_Workers>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:28:50.1491022Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:28:50.1491022Z</p1:As_Of_Entry_DateTime>
    <p1:Page>1</p1:Page>
    <p1:Count>1</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
  </p1:Response_Group>
</Get_Workers_Request>
```

## <a name="how-full-sync-works"></a>Как работает полная синхронизация

**Полная синхронизация** в контексте подготовки, управляемой workday, относится к процессу выборки всех удостоверений из workday и определения правил подготовки, применяемых к каждому рабочему объекту. Полная синхронизация происходит при первом включении подготовки, а также при *перезапуске подготовки* из портал Azure или с помощью API Graph. 

Azure AD отправляет следующий запрос на веб-службы *Get_Workers* Workday для получения данных о рабочих процессах. Запрос ищет журнал транзакций Workday для всех действующих записей о работниках, начиная с времени, соответствующего полному выполнению синхронизации. 

```XML
<!-- Workday full sync query -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to full sync run -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Type_References>
        <p1:Transaction_Type_Reference>
          <p1:ID p1:type="Business_Process_Type">Hire Employee</p1:ID>
        </p1:Transaction_Type_Reference>
        <p1:Transaction_Type_Reference>
          <p1:ID p1:type="Business_Process_Type">Contract Contingent Worker</p1:ID>
        </p1:Transaction_Type_Reference>
      </p1:Transaction_Type_References>
    </p1:Transaction_Log_Criteria_Data>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:29:16.0094202Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:29:16.0094202Z</p1:As_Of_Entry_DateTime>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Transaction_Log_Data>1</p1:Include_Transaction_Log_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```
Узел *Response_Group* используется, чтобы указать, какие рабочие атрибуты получать из Workday. Описание каждого флага в узле *Response_Group* см. в [документации по API-интерфейсу Workday Get_Workers](https://community.workday.com/sites/default/files/file-hosting/productionapi/Human_Resources/v35.2/Get_Workers.html#Worker_Response_GroupType). 

Некоторые значения флагов, указанные в узле *Response_Group* , рассчитываются на основе атрибутов, настроенных в приложении подготовки Azure AD для Workday. Дополнительные сведения о критериях, используемых для задания значений флагов, см. в разделе, посвященном *поддерживаемым сущностям* . 

*Get_Workersный* ответ из Workday для указанного выше запроса включает число записей рабочей роли и число страниц.

```XML
  <wd:Response_Results>
    <wd:Total_Results>509</wd:Total_Results>
    <wd:Total_Pages>17</wd:Total_Pages>
    <wd:Page_Results>30</wd:Page_Results>
    <wd:Page>1</wd:Page>
  </wd:Response_Results>
```
Чтобы получить следующую страницу результирующего набора, следующий запрос *Get_Workers* указывает номер страницы в качестве параметра в *Response_Filter*.

```XML
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:29:16.0094202Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:29:16.0094202Z</p1:As_Of_Entry_DateTime>
    <p1:Page>2</p1:Page>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
```
Служба подготовки Azure AD обрабатывает каждую страницу и перебирает все действующие рабочие роли во время полной синхронизации. Для каждой записи работника, импортированной из Workday:
* [Выражение XPath](workday-attribute-reference.md) применяется для получения значений атрибутов из Workday.
* Применяются сопоставление атрибутов и правила сопоставления. 
* Служба определяет, какая операция должна выполняться в целевом объекте (Azure AD или AD). 

После завершения обработки она сохраняет метку времени, связанную с началом полной синхронизации, в качестве водяного знака. Этот водяной знак выступает в качестве отправной точки для цикла добавочной синхронизации. 

## <a name="how-incremental-sync-works"></a>Как работает добавочная синхронизация

После полной синхронизации Служба подготовки Azure AD обслуживает `LastExecutionTimestamp` и использует ее для создания разностных запросов для получения добавочных изменений. Во время добавочной синхронизации Azure AD отправляет в Workday следующие типы запросов: 

* [Запрос обновлений вручную](#query-for-manual-updates)
* [Запрос на актуальные обновления и прекращения действия](#query-for-effective-dated-updates-and-terminations)
* [Запрос для сотрудников, датированных будущим будущее](#query-for-future-dated-hires)

### <a name="query-for-manual-updates"></a>Запрос обновлений вручную

Следующие *Get_Workers* запрашивают запросы на обновления вручную, произошедшие между последним выполнением и текущим временем выполнения. 

```xml
<!-- Workday incremental sync query for manual updates -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to last execution and current execution time -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Date_Range_Data>
        <p1:Updated_From>2021-01-19T02:29:16.0094202Z</p1:Updated_From>
        <p1:Updated_Through>2021-01-19T02:49:06.290136Z</p1:Updated_Through>
      </p1:Transaction_Date_Range_Data>
    </p1:Transaction_Log_Criteria_Data>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:49:06.290136Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:49:06.290136Z</p1:As_Of_Entry_DateTime>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```

### <a name="query-for-effective-dated-updates-and-terminations"></a>Запрос на актуальные обновления и прекращения действия

Следующие *Get_Workers* запрашивают запросы о последних обновлениях, произошедших между последним выполнением и текущим временем выполнения. 

```xml
<!-- Workday incremental sync query for effective-dated updates -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below with the UTC time corresponding to last execution and current execution time -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_Criteria>
    <p1:Transaction_Log_Criteria_Data>
      <p1:Transaction_Date_Range_Data>
        <p1:Effective_From>2021-01-19T02:29:16.0094202Z</p1:Effective_From>
        <p1:Effective_Through>2021-01-19T02:49:06.290136Z</p1:Effective_Through>
      </p1:Transaction_Date_Range_Data>
    </p1:Transaction_Log_Criteria_Data>
  </p1:Request_Criteria>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-01-19T02:49:06.290136Z</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-01-19T02:49:06.290136Z</p1:As_Of_Entry_DateTime>
    <p1:Page>1</p1:Page>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```

### <a name="query-for-future-dated-hires"></a>Запрос для сотрудников, датированных будущим будущее

Если любой из приведенных выше запросов возвращает сведения о найме в будущем, следующий запрос *Get_Workers* используется для получения сведений о новом найме в будущем. Атрибут *WID* нового приема на работу используется для выполнения уточняющего запроса, а для даты вступления в силу устанавливается дата и время найма. 

```xml
<!-- Workday incremental sync query to get new hire data effective as on hire date/first day of work -->
<!-- Replace version with Workday Web Services version present in your connection URL -->
<!-- Replace timestamps below hire date/first day of work -->
<!-- Count specifies the number of records to return in each page -->
<!-- Response_Group flags derived from provisioning attribute mapping -->

<Get_Workers_Request p1:version="v21.1" xmlns:p1="urn:com.workday/bsvc" xmlns="urn:com.workday/bsvc">
  <p1:Request_References>
    <p1:Worker_Reference>
      <p1:ID p1:type="WID">7bf6322f1ea101fd0b4433077f09cb04</p1:ID>
    </p1:Worker_Reference>
  </p1:Request_References>
  <p1:Response_Filter>
    <p1:As_Of_Effective_Date>2021-02-01T08:00:00+00:00</p1:As_Of_Effective_Date>
    <p1:As_Of_Entry_DateTime>2021-02-01T08:00:00+00:00</p1:As_Of_Entry_DateTime>
    <p1:Count>30</p1:Count>
  </p1:Response_Filter>
  <p1:Response_Group>
    <p1:Include_Reference>1</p1:Include_Reference>
    <p1:Include_Personal_Information>1</p1:Include_Personal_Information>
    <p1:Include_Employment_Information>1</p1:Include_Employment_Information>
    <p1:Include_Organizations>1</p1:Include_Organizations>
    <p1:Exclude_Organization_Support_Role_Data>1</p1:Exclude_Organization_Support_Role_Data>
    <p1:Exclude_Location_Hierarchies>1</p1:Exclude_Location_Hierarchies>
    <p1:Exclude_Cost_Center_Hierarchies>1</p1:Exclude_Cost_Center_Hierarchies>
    <p1:Exclude_Company_Hierarchies>1</p1:Exclude_Company_Hierarchies>
    <p1:Exclude_Matrix_Organizations>1</p1:Exclude_Matrix_Organizations>
    <p1:Exclude_Pay_Groups>1</p1:Exclude_Pay_Groups>
    <p1:Exclude_Regions>1</p1:Exclude_Regions>
    <p1:Exclude_Region_Hierarchies>1</p1:Exclude_Region_Hierarchies>
    <p1:Exclude_Funds>1</p1:Exclude_Funds>
    <p1:Exclude_Fund_Hierarchies>1</p1:Exclude_Fund_Hierarchies>
    <p1:Exclude_Grants>1</p1:Exclude_Grants>
    <p1:Exclude_Grant_Hierarchies>1</p1:Exclude_Grant_Hierarchies>
    <p1:Exclude_Business_Units>1</p1:Exclude_Business_Units>
    <p1:Exclude_Business_Unit_Hierarchies>1</p1:Exclude_Business_Unit_Hierarchies>
    <p1:Exclude_Programs>1</p1:Exclude_Programs>
    <p1:Exclude_Program_Hierarchies>1</p1:Exclude_Program_Hierarchies>
    <p1:Exclude_Gifts>1</p1:Exclude_Gifts>
    <p1:Exclude_Gift_Hierarchies>1</p1:Exclude_Gift_Hierarchies>
    <p1:Include_Management_Chain_Data>1</p1:Include_Management_Chain_Data>
    <p1:Include_Additional_Jobs>1</p1:Include_Additional_Jobs>
  </p1:Response_Group>
</Get_Workers_Request>
```

## <a name="retrieving-worker-data-attributes"></a>Получение атрибутов рабочих данных

*Get_Workers* API может возвращать различные наборы данных, связанные с рабочими процессами. В зависимости от [выражений API XPath](workday-attribute-reference.md) , настроенных в схеме подготовки, служба подготовки Azure AD определяет, какие наборы данных необходимо получить из Workday. Соответственно, флаги *Response_Group* задаются в запросе *Get_Workers* . 

В таблице ниже приводятся рекомендации по конфигурации сопоставления, используемой для получения определенного набора данных. 

| \# | Сущность Workday                       | Включено по умолчанию | Шаблон XPATH для указания в сопоставлении для выборки сущностей, отличных от по умолчанию             |
|----|--------------------------------------|---------------------|-------------------------------------------------------------------------------|
| 1  | Персональные данные                        | Да                 | WD: Worker \_ Data/WD: персональные \_ данные                                             |
| 2  | Данные о занятости                      | Да                 | WD: Worker \_ Data/WD: данные о занятости \_                                           |
| 3  | Дополнительные данные задания                  | Да                 | WD: Worker \_ Data/WD: данные о занятости \_ /WD: \_ Задание данных задания рабочей роли \_ \[ @wd:Primary \_ = 0\]|
| 4  | Данные Организации                    | Да                 | WD: Worker \_ Data/WD: \_ данные Организации                                         |
| 5  | Данные цепочки управления                | Да                 | WD: Worker \_ Data/WD: \_ данные цепочки управления \_                                    |
| 6  | Организация, ответственная за             | Да                 | Управляющая                                                                 |
| 7  | Company                              | Да                 | ВО                                                                     |
| 8  | Бизнес-единица                        | Нет                  | "бизнес- \_ единица"                                                              |
| 9  | Иерархия бизнес-единицы              | Нет                  | " \_ \_ Иерархия филиалов"                                                   |
| 10 | Иерархия Организации                    | Нет                  | " \_ Иерархия компании"                                                          |
| 11 | Cost Center                          | Нет                  | " \_ центр затрат"                                                                |
| 12 | Иерархия центра затрат                | Нет                  | " \_ \_ Иерархия центров затрат"                                                     |
| 13 | Финансировать                                 | Нет                  | ФИНАНСИРОВАТЬ                                                                        |
| 14 | Иерархия фонда                       | Нет                  | " \_ Иерархия фонда"                                                             |
| 15 | Сальдо                                 | Нет                  | САЛЬДО                                                                        |
| 16 | Подарочная иерархия                       | Нет                  | "ПОДАРОЧная \_ Иерархия"                                                             |
| 17 | Предоставить                                | Нет                  | ИМ                                                                       |
| 18 | Предоставить иерархию                      | Нет                  | "предоставить \_ иерархию"                                                            |
| 19 | Иерархия бизнес-сайтов              | Нет                  | " \_ \_ Иерархия сайтов предприятия"                                                   |
| 20 | Матричная организация                  | Нет                  | ТАБЛИЦУ                                                                      |
| 21 | Группа оплаты                            | Нет                  | " \_ Группа оплаты"                                                                  |
| 22 | Programs                             | Нет                  | ПРИЛОЖЕНИЯМИ                                                                    |
| 23 | Иерархия программы                    | Нет                  | " \_ Иерархия программ"                                                          |
| 24 | Регион                               | Нет                  | " \_ Иерархия регионов"                                                           |
| 25 | Иерархия расположений                   | Нет                  | " \_ Иерархия расположения"                                                         |
| 26 | Данные подготовки учетных записей            | Нет                  | WD: Worker \_ Data/WD: \_ данные подготовки учетной записи \_                                |
| 27 | Данные фоновой проверки                | Нет                  | WD: \_ данные Worker/WD: данные фоновой \_ проверки \_                                    |
| 28 | Данные о допустимости льготы             | Нет                  | WD: Worker \_ Data/WD: \_ данные о допустимости льготы \_                                 |
| 29 | Данные о регистрации преимуществ              | Нет                  | WD: Worker \_ Data/WD: \_ данные регистрации преимуществ \_                                  |
| 30 | Данные карьеры                          | Нет                  | WD: данные рабочих ролей \_ и WD: данные о карьере \_                                               |
| 31 | Данные компенсации                    | Нет                  | WD: Worker \_ Data/WD: \_ данные компенсации                                         |
| 32 | Данные налоговой инспекции по рабочему процессу | Нет                  | WD: Worker \_ Data/WD: \_ \_ \_ \_ \_ данные типа формы налогового органа для производной рабочей роли \_       |
| 33 | Данные элемента разработки                | Нет                  | WD: данные рабочих ролей \_ /WD: \_ данные элемента разработки \_                                    |
| 34 | Данные контрактов сотрудников              | Нет                  | WD: Worker \_ Data/WD: \_ данные контрактов \_ сотрудников                                  |
| 35 | Данные проверки сотрудника                 | Нет                  | WD: Worker \_ Data/WD: \_ данные проверки сотрудника \_                                     |
| 36 | Отзывы о полученных данных               | Нет                  | WD: Worker \_ Data/WD: Отзывы \_ полученных \_ данных                                   |
| 37 | Данные цели рабочей роли                     | Нет                  | WD: Worker \_ Data/WD: \_ данные цели рабочей роли \_                                         |
| 38 | Данные фото                           | Нет                  | WD: Worker \_ Data/WD: \_ Данные фотографий                                                |
| 39 | Квалификационные данные                   | Нет                  | WD: Worker \_ Data/WD: квалификационные \_ данные                                        |
| 40 | Данные связанных лиц                 | Нет                  | WD: данные рабочих ролей \_ /WD: \_ данные связанных лиц \_                                     |
| 41 | Данные роли                            | Нет                  | WD: Worker \_ Data/WD: \_ данные роли                                                 |
| 42 | Данные о навыках                           | Нет                  | WD: Worker \_ Data/WD: данные о навыках \_                                                |
| 43 | Данные профиля последующего выполнения              | Нет                  | WD: Worker \_ Data/WD: \_ данные профиля последующего выполнения \_                                  |
| 44 | Данные оценки неспециалистов               | Нет                  | WD: Data Workers \_ /WD: \_ \_ данные оценки недоступности                                   |
| 45 | Данные учетной записи пользователя                    | Нет                  | WD: Worker \_ Data/WD: \_ данные учетной записи пользователя \_                                        |
| 46 | Данные рабочего документа                 | Нет                  | WD: Worker \_ Data/WD: \_ данные документа рабочей роли \_                                     |

>[!NOTE]
>Каждая сущность Workday, указанная в таблице, защищена **политикой безопасности домена** в Workday. Если вы не можете получить атрибут, связанный с сущностью, после установки правильного XPATH, обратитесь к администратору Workday, чтобы убедиться, что для пользователя системы интеграции, связанного с приложением подготовки, настроена соответствующая политика безопасности домена. Например, чтобы получить *данные о навыках*, необходимо *получить* доступ к *данным рабочей роли домена Workday: навыки и опыт работы*. 

Ниже приведены некоторые примеры того, как можно расширить интеграцию с Workday для удовлетворения конкретных требований. 

**Пример 1**

Предположим, вы хотите получить из Workday следующие наборы данных и использовать их в правилах подготовки:

* место возникновения затрат;
* Иерархия центра затрат
* Группа оплаты

Указанные выше наборы данных не включаются по умолчанию. Для получения этих наборов данных:
1. Войдите в портал Azure и откройте приложение подготовки пользователей Workday в AD или Azure AD. 
1. В колонке подготовка измените сопоставления и откройте список атрибутов Workday из раздела дополнительно. 
1. Добавьте следующие определения атрибутов и пометьте их как "обязательные". Эти атрибуты не будут сопоставляться ни с одним атрибутом в AD или Azure AD. Они просто служат сигналами к соединителю для получения сведений о центре затрат, иерархии центра затрат и сведениях о группах оплаты. 

     > [!div class="mx-tdCol2BreakAll"]
     >| Имя атрибута | Выражение API XPATH |
     >|---|---|
     >| костцентерхиерарчифлаг  |  WD: Worker/WD: Worker_Data/WD: Organization_Data/WD: Worker_Organization_Data [WD: Organization_Data/WD: Organization_Type_Reference/WD: ID [ @wd:type = ' Organization_Type_ID '] = ' COST_CENTER_HIERARCHY ']/wd:Organization_Reference/@wd:Descriptor |
     >| костцентерфлаг  |  WD: Worker/WD: Worker_Data/WD: Organization_Data/WD: Worker_Organization_Data [WD: Organization_Data/WD: Organization_Type_Reference/WD: ID [ @wd:type = ' Organization_Type_ID '] = ' COST_CENTER ']/WD: Organization_Data/WD: Organization_Code/Text () |
     >| пайграупфлаг  |  WD: Worker/WD: Worker_Data/WD: Organization_Data/WD: Worker_Organization_Data [WD: Organization_Data/WD: Organization_Type_Reference/WD: ID [ @wd:type = ' Organization_Type_ID '] = ' PAY_GROUP ']/WD: Organization_Data/WD: Organization_Reference_ID/Text () |

1. После того как центр затрат и набор данных группы оплаты будут доступны в *Get_Workers* ответе, можно использовать приведенные ниже значения XPath, чтобы получить имя центра затрат, код центра затрат и группу оплаты. 

     > [!div class="mx-tdCol2BreakAll"]
     >| Имя атрибута | Выражение API XPATH |
     >|---|---|
     >| костцентернаме  | WD: Worker/WD: Worker_Data/WD: Organization_Data/WD: Worker_Organization_Data/WD: Organization_Data [ wd:Organization_Type_Reference/@wd:Descriptor = ' центр затрат ']/WD: Organization_Name/Text () |
     >| костцентеркоде | WD: Worker/WD: Worker_Data/WD: Organization_Data/WD: Worker_Organization_Data/WD: Organization_Data [ wd:Organization_Type_Reference/@wd:Descriptor = ' центр затрат ']/WD: Organization_Code/Text () |
     >| пайграуп | WD: Worker/WD: Worker_Data/WD: Organization_Data/WD: Worker_Organization_Data/WD: Organization_Data [ wd:Organization_Type_Reference/@wd:Descriptor = "Группа оплаты"]/WD: Organization_Name/Text () |

**Пример 2**

Предположим, вы хотите получить сертификаты, связанные с пользователем. Эти сведения доступны в составе набора *квалификационных данных* . Чтобы получить этот набор данных как часть ответа *Get_Workers* , используйте следующий XPath: 

`wd:Worker/wd:Worker_Data/wd:Qualification_Data/wd:Certification/wd:Certification_Data/wd:Issuer/text()`

**Пример 3**

Предположим, вы хотите получить *группы подготовки* , назначенные рабочей роли. Эти сведения доступны как часть набора *данных подготовки учетных записей* . Чтобы получить этот набор данных как часть ответа *Get_Workers* , используйте следующий XPath: 

`wd:Worker/wd:Worker_Data/wd:Account_Provisioning_Data/wd:Provisioning_Group_Assignment_Data[wd:Status='Assigned']/wd:Provisioning_Group/text()`

## <a name="handling-different-hr-scenarios"></a>Обработка различных сценариев HR

### <a name="retrieving-international-job-assignments-and-secondary-job-details"></a>Получение международных назначений заданий и сведений о вторичном задании

По умолчанию соединитель Workday извлекает атрибуты, связанные с основным заданием работника. Соединитель также поддерживает получение *дополнительных данных о заданиях* , связанных с назначениями международных заданий или вторичными заданиями. 

Чтобы получить атрибуты, связанные с назначениями международного задания, выполните следующие действия. 

1. Задайте для URL-адреса подключения Workday использование API веб-службы Workday версии 30,0 или более поздней. Соответствующим образом задайте [правильные значения XPath](workday-attribute-reference.md#xpath-values-for-workday-web-services-wws-api-v30) в приложении подготовки Workday. 
1. `@wd:Primary_Job=0` `Worker_Job_Data` Для получения правильного атрибута используйте селектор на узле. 
   * **Пример 1.** Чтобы получить `SecondaryBusinessTitle` Использование XPath `wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Worker_Job_Data[@wd:Primary_Job=0]/wd:Position_Data/wd:Business_Title/text()`
   * **Пример 2:** Чтобы получить `SecondaryBusinessLocation` Использование XPath `wd:Worker/wd:Worker_Data/wd:Employment_Data/wd:Worker_Job_Data[@wd:Primary_Job=0]/wd:Position_Data/wd:Business_Site_Summary_Data/wd:Location_Reference/@wd:Descriptor`

 

## <a name="next-steps"></a>Дальнейшие действия

* [Узнайте, как настроить Workday для Active Directory подготовки](../saas-apps/workday-inbound-tutorial.md)
* [Узнайте, как настроить обратную запись в Workday](../saas-apps/workday-writeback-tutorial.md)
* [Дополнительные сведения о поддерживаемых атрибутах Workday для входящей подготовки](workday-attribute-reference.md)

