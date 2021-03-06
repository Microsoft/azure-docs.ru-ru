---
title: Настройка контроля доступа для рабочей области синапсе
description: Из этой статьи вы узнаете, как управлять доступом к рабочей области синапсе с помощью ролей Azure, ролей синапсе, разрешений SQL и разрешений Git.
services: synapse-analytics
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: security
ms.date: 12/03/2020
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: 97f9d0e0037090a8c058eb6e2393451d975e79c6
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103472257"
---
# <a name="how-to-set-up-access-control-for-your-synapse-workspace"></a>Настройка контроля доступа для рабочей области синапсе 

Из этой статьи вы узнаете, как управлять доступом к рабочей области синапсе с помощью ролей Azure, ролей синапсе, разрешений SQL и разрешений Git.   

В этом разделе вы настроите рабочую область и настроите базовую систему контроля доступа, подходящую для многих проектов синапсе.  Затем в нем описываются более сложные параметры для более точного управления, если это необходимо.  

Управление доступом синапсе можно упростить с помощью групп безопасности, которые согласованы с ролями и персонажами в Организации.  Для управления доступом необходимо только добавлять пользователей в группы безопасности и удалять их из них.

Прежде чем приступить к работе с этим пошаговым руководством, прочитайте [Обзор управления доступом синапсе](./synapse-workspace-access-control-overview.md) , чтобы ознакомиться с механизмами управления доступом, используемыми синапсе.   

## <a name="access-control-mechanisms"></a>Механизмы управления доступом

> [!NOTE]
> Этот подход состоит в создании нескольких групп безопасности и назначении ролей этим группам. После настройки групп необходимо управлять членством в группах безопасности, чтобы управлять доступом к рабочей области.

Чтобы защитить рабочую область синапсе, необходимо выполнить шаблон настройки следующих элементов:

- **Группы безопасности** для группировки пользователей с одинаковыми требованиями к доступу.
- **Роли Azure** позволяют контролировать, кто может создавать и администрировать пулы SQL, пулы Apache Spark и среды выполнения интеграции, а также получать доступ к ADLS 2-го поколенияному хранилищу.
- **Роли синапсе** для контроля доступа к артефактам опубликованного кода, использования ресурсов Apache Spark вычислений и сред выполнения интеграции 
- **Разрешения SQL** для управления доступом администраторов и плоскости данных к пулам SQL. 
- **Разрешения Git** для управления доступом к артефактам кода в системе управления версиями при настройке поддержки Git для рабочей области 
 
## <a name="steps-to-secure-a-synapse-workspace"></a>Действия по обеспечению безопасности рабочей области Synapse

В этой документации для упрощения инструкций используются стандартные имена. Замените их именами по своему усмотрению.

|Параметр | Стандартное имя | Описание |
| :------ | :-------------- | :---------- |
| **Рабочая область Synapse** | `workspace1` |  Имя, которое будет иметь рабочая область Synapse. |
| **Учетная запись ADLSGEN2** | `storage1` | Учетная запись ADLS для использования с рабочей областью. |
| **Контейнер** | `container1` | Контейнер в STG1, который будет использоваться по умолчанию рабочей областью. |
| **Клиент Active Directory** | `contoso` | Имя клиента Active Directory.|
||||

## <a name="step-1-set-up-security-groups"></a>Шаг 1. Настройка групп безопасности

>[!Note] 
>Во время предварительной версии рекомендуется создавать группы безопасности, сопоставленные с ролями администратора синапсе **СИНАПСЕ SQL** и **синапсе Apache Spark** .  С появлением новых ролей и областей с более детализированными Синапсеами RBAC мы рекомендуем использовать эти новые возможности для управления доступом к рабочей области.  Эти новые роли и области обеспечивают большую гибкость в настройке и узнают, что разработчики часто используют сочетание SQL и Spark при создании аналитических приложений, и им может потребоваться предоставить доступ к конкретным ресурсам, а не ко всей рабочей области. Дополнительные [сведения](./synapse-workspace-synapse-rbac.md) о синапсе RBAC.

Создайте следующие группы безопасности для рабочей области:

- **`workspace1_SynapseAdministrators`**, для пользователей, которым необходим полный контроль над рабочей областью.  Добавьте себя в эту группу безопасности, по крайней мере, изначально.
- **`workspace1_SynapseContributors`**, для разработчиков, которым требуется разрабатывать, отлаживать и публиковать код в службе.   
- **`workspace1_SynapseComputeOperators`**, для пользователей, которым требуется управлять и отслеживать пулы Apache Spark и среды выполнения интеграции.
- **`workspace1_SynapseCredentialUsers`**, для пользователей, которым требуется выполнять отладку и запуск конвейеров оркестрации с использованием учетных данных MSI рабочей области (управляемое удостоверение службы) и отмены выполнения конвейера.   

В ближайшее время в области рабочей области вы назначите роли синапсе для этих групп.  

Также создайте эту группу безопасности: 
- **`workspace1_SQLAdmins`**, группа для пользователей, которым нужен центр администрирования SQL Active Directory в пулах SQL в рабочей области. 

`workspace1_SQLAdmins`Группа будет использоваться при настройке разрешений SQL в ПУЛАХ SQL при их создании. 

Для базовой настройки достаточно этих пяти групп. Позже можно добавить группы безопасности для управления пользователями, которым необходим более специализированный доступ, или предоставления пользователям доступа только к конкретным ресурсам.

> [!NOTE]
>- В [этой статье](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md) описано, как создать группу безопасности.
>- В [этой статье ](../../active-directory/fundamentals/active-directory-groups-membership-azure-portal.md) описано, как добавить группу безопасности из другой группы безопасности.

>[!Tip]
>Отдельные пользователи синапсе могут использовать Azure Active Directory в портал Azure для просмотра их членства в группах, чтобы определить, какие роли они были предоставлены.

## <a name="step-2-prepare-your-adls-gen2-storage-account"></a>Шаг 2. Подготовка учетной записи хранения ADLS 2-го поколения

Рабочая область синапсе использует контейнер хранилища по умолчанию для:
  - хранения резервных файлов данных для таблиц Spark;
  - журналов выполнения для заданий Spark.
  - Управление библиотеками, выбранными для установки

Найдите следующие сведения о хранилище:

- Учетная запись ADLS 2-го поколения, используемая для рабочей области. Этот документ вызывает его `storage1` . `storage1` считается первичной учетной записью хранения для рабочей области.
- Контейнер в `workspace1` , который по умолчанию будет использоваться рабочей областью синапсе. Этот документ вызывает его `container1` . 

- Используя портал Azure, назначьте следующие роли Azure `container1` группам безопасности. 

  - Назначение роли **участника данных BLOB-объекта хранилища**`workspace1_SynapseAdmins` 
  - Назначение роли **участника данных BLOB-объекта хранилища**`workspace1_SynapseContributors`
  - Назначение роли **участника данных BLOB-объекта хранилища**`workspace1_SynapseComputeOperators`

## <a name="step-3-create-and-configure-your-synapse-workspace"></a>Шаг 3. Создание и настройка рабочей области Synapse

На портале Azure создайте рабочую область Synapse:

- Выберите свою подписку.
- Выберите или создайте группу ресурсов, для которой у вас есть роль **владельца** Azure.
- Имя рабочей области `workspace1`
- Выберите `storage1` учетную запись хранения
- Выберите `container1` контейнер, который используется в качестве файловой системы.
- Откройте WS1 в Synapse Studio.
- Перейдите к разделу **Управление**  >  **доступом** и назначьте роли синапсе в *области действия рабочей области* для групп безопасности следующим образом:
  - Назначьте роль **администратора синапсе**`workspace1_SynapseAdministrators` 
  - Назначьте роль **участника синапсе**`workspace1_SynapseContributors` 
  - Назначьте роль **оператора вычислений синапсе**`workspace1_SynapseComputeOperators`

## <a name="step-4-grant-the-workspace-msi-access-to-the-default-storage-container"></a>Шаг 4. предоставление MSI рабочей области доступа к контейнеру хранилища по умолчанию

Для выполнения конвейеров и выполнения системных задач синапсе требует, чтобы управляемое удостоверение службы рабочей области (MSI) требовало доступа к `container1` в учетной записи ADLS 2-го поколения по умолчанию.

- Откройте портал Azure.
- Найдите учетную запись хранения, `storage1` а затем `container1`
- С помощью **управления доступом (IAM)** убедитесь, что роль **участника данных BLOB-объекта хранилища** назначена MSI рабочей области.
  - Если он не назначен, назначьте его.
  - Имя MSI совпадает с именем рабочей области. В этой статье это будет `workspace1` .

## <a name="step-5-grant-synapse-administrators-the-azure-contributor-role-on-the-workspace"></a>Шаг 5. Предоставление администраторам синапсе роли участника Azure в рабочей области 

Чтобы создать пулы SQL, пулы Apache Spark и среды выполнения интеграции, пользователи должны иметь по крайней мере роль участника Azure в рабочей области. Роль участника также позволяет этим пользователям управлять ресурсами, включая приостановку и масштабирование. Если вы используете портал Azure или синапсе Studio для создания пулов SQL, Apache Spark пулов и сред выполнения интеграции, вам потребуется роль участника Azure на уровне группы ресурсов. 

- Откройте портал Azure.
- Выберите рабочую область, `workspace1`
- Назначьте роль **участника Azure** `workspace1` для `workspace1_SynapseAdministrators` . 

## <a name="step-6-assign-sql-active-directory-admin-role"></a>Шаг 6. Назначение роли администратора Active Directory SQL

Создатель рабочей области автоматически настраивается как администратор SQL Active Directory для рабочей области.  Эта роль может предоставить только один пользователь или группа. На этом шаге вы назначаете администратору SQL Active Directory в рабочей области `workspace1_SQLAdmins` группе безопасности.  Назначение этой роли дает этой группе широкие права администратора для доступа ко всем пулам и базам данных SQL в рабочей области.   

- Откройте портал Azure.
- Перейдите на страницу `workspace1`.
- В разделе **Параметры** выберите **Администратор Active Directory SQL** .
- Щелкните **задать администратора** и выберите **`workspace1_SQLAdmins`**

>[!Note]
>Шаг 6 является необязательным.  Вы можете предоставить `workspace1_SQLAdmins` группе более низкий приоритет роли. Для назначения `db_owner` или других РОЛЕЙ SQL необходимо выполнять сценарии в каждой базе данных SQL. 

## <a name="step-7-grant-access-to-sql-pools"></a>Шаг 7. предоставление доступа к пулам SQL

По умолчанию всем пользователям, которым назначена роль администратора синапсе, также назначается роль SQL для независимого от `db_owner` сервера пула SQL, встроенного и всех его баз данных.

Доступ к пулам SQL для других пользователей и для MSI-файла рабочей области контролируется с помощью разрешений SQL.  Назначение разрешений SQL требует, чтобы скрипты SQL выполнялись в каждой базе данных SQL после создания.  Существует три варианта, требующие выполнения следующих сценариев:
1. Предоставление другим пользователям доступа к бессерверному пулу SQL, встроенным службам и базам данных
2. Предоставление любому пользователю доступа к выделенным базам данных пула
3. Предоставление доступа MSI рабочей области к базе данных пула SQL, чтобы обеспечить успешное выполнение конвейеров, требующих доступа к пулу SQL.

Ниже приведены примеры сценариев SQL.

Чтобы предоставить доступ к выделенной базе данных пула SQL, эти сценарии могут выполнять создатель рабочей области или любой член `workspace1_SQLAdmins` группы.  

Чтобы предоставить доступ к неиспользуемому для сервера пулу SQL, сценарии могут запускаться любым членом `workspace1_SQLAdmins` группы или  `workspace1_SynapseAdministrators` группы. 

> [!TIP]
> Приведенные ниже действия необходимо выполнить для **каждого** пула SQL, чтобы предоставить пользователю доступ ко всем базам данных SQL, за исключением [разрешения уровня рабочей области](#workspace-scoped-permission) "область", где можно назначить пользователю роль sysadmin на уровне рабочей области.

### <a name="step-71-serverless-sql-pool-built-in"></a>Шаг 7,1. бессерверный пул SQL, встроенный

В этом разделе приведены примеры сценариев, показывающих, как предоставить пользователю разрешение на доступ к определенной базе данных или ко всем базам данных в бессерверном пуле SQL Server, встроенном.

> [!NOTE]
> В примерах сценария замените *Alias* псевдонимом пользователя или группы, которым предоставляется доступ, и укажите *домен* с используемым доменом организации.

#### <a name="database-scoped-permission"></a>Разрешение уровня базы данных

Чтобы предоставить пользователю доступ к **одной** базе данных SQL, не обращающейся к серверу, выполните действия, описанные в этом примере.

1. Создайте имя для входа:

    ```sql
    use master
    go
    CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
    go
    ```

2. Создайте пользователя:

    ```sql
    use yourdb -- Use your database name
    go
    CREATE USER alias FROM LOGIN [alias@domain.com];
    ```

3. Добавьте пользователя к членам указанной роли:

    ```sql
    use yourdb -- Use your database name
    go
    alter role db_owner Add member alias -- Type USER name from step 2
    ```

#### <a name="workspace-scoped-permission"></a>Разрешение области действия рабочей области

Чтобы предоставить полный доступ ко **всем** бессерверным пулам SQL в рабочей области, используйте сценарий в этом примере:

```sql
use master
go
CREATE LOGIN [alias@domain.com] FROM EXTERNAL PROVIDER;
ALTER SERVER ROLE sysadmin ADD MEMBER [alias@domain.com];
```

### <a name="step-72-dedicated-sql-pools"></a>Шаг 7,2. выделенные пулы SQL

Чтобы предоставить доступ к **одной** выделенной базе данных пула SQL, выполните следующие действия в РЕДАКТОРЕ скриптов SQL синапсе:

1. Создайте пользователя в базе данных, выполнив следующую команду в целевой базе данных, выбранную с помощью раскрывающегося списка *Подключение к* :

    ```sql
    --Create user in the database
    CREATE USER [<alias@domain.com>] FROM EXTERNAL PROVIDER;
    ```

2. Предоставьте пользователю роль для доступа к базе данных:

    ```sql
    --Grant role to the user in the database
    EXEC sp_addrolemember 'db_owner', '<alias@domain.com>';
    ```

> [!IMPORTANT]
> *db_datareader* и *db_datawriter* могут работать для разрешений на чтение и запись, если предоставление разрешения *db_owner* не требуется.
> Чтобы пользователь Spark прочитал и записывал непосредственно из Spark в пул SQL или из него, требуется *db_owner* разрешение.

После создания пользователей выполните запросы, чтобы убедиться, что серверный пул SQL может запрашивать учетную запись хранения.

### <a name="step-73-sql-access-control-for-synapse-pipeline-runs"></a>Шаг 7,3. Управление доступом SQL для выполнения конвейера синапсе

### <a name="workspace-managed-identity"></a>Управляемое удостоверение рабочей области

> [!IMPORTANT]
> Для успешного выполнения конвейеров, включающих наборы данных или действия, которые ссылаются на пул SQL, удостоверению рабочей области необходимо предоставить доступ к пулу SQL.

Выполните следующие команды в каждом пуле SQL, чтобы разрешить управляемой рабочей областью удостоверение системы для запуска конвейеров в базах данных пула SQL:  

>[!note]
>В сценариях ниже для выделенной базы данных пула SQL DatabaseName совпадает с именем пула.  Для базы данных в бессерверном пуле SQL, databasename — это имя базы данных.

```sql
--Create a SQL user for the workspace MSI in database
CREATE USER [<workspacename>] FROM EXTERNAL PROVIDER;

--Granting permission to the identity
GRANT CONTROL ON DATABASE::<databasename> TO <workspacename>;
```

Это разрешение можно удалить, выполнив следующий скрипт в том же пуле SQL:

```sql
--Revoke permission granted to the workspace MSI
REVOKE CONTROL ON DATABASE::<databasename> TO <workspacename>;

--Delete the workspace MSI user in the database
DROP USER [<workspacename>];
```

## <a name="step-8-add-users-to-security-groups"></a>Шаг 8. Добавление пользователей в группы безопасности

Начальная конфигурация для системы контроля доступа завершена.

Для управления доступом можно добавлять пользователей в настроенные группы безопасности и удалять их.  Хотя вы можете вручную назначить пользователей ролям синапсе, в противном случае они не будут согласованно настраивать их разрешения. Вместо этого только добавьте пользователей в группы безопасности или удалите их из них.

## <a name="step-9-network-security"></a>Шаг 9. Сетевая безопасность

В качестве последнего шага для защиты рабочей области следует защитить сетевой доступ, используя:
- [Брандмауэр рабочей области](./synapse-workspace-ip-firewall.md)
- [Управляемая виртуальная сеть](./synapse-workspace-managed-vnet.md) 
- [Частные конечные точки](./synapse-workspace-managed-private-endpoints.md)
- [Приватный канал](../../azure-sql/database/private-endpoint-overview.md)

## <a name="step-10-completion"></a>Шаг 10. Завершение

Теперь ваша рабочая область полностью настроена и защищена.

## <a name="supporting-more-advanced-scenarios"></a>Поддержка более сложных сценариев

Это пошаговое руководством посвящено настройке базовой системы контроля доступа. Вы можете реализовать более сложные сценарии, создав дополнительные группы безопасности и назначив эти группы более детализированными ролями в более конкретных областях. Рассмотрим следующие случаи.

**Включите поддержку Git** для рабочей области для более сложных сценариев разработки, включая непрерывную интеграцию и компакт-диск.  В режиме Git разрешения Git определяют, может ли пользователь зафиксировать изменения в своей рабочей ветви.  Публикация в службе выполняется только из ветви совместной работы.  Рассмотрите возможность создания группы безопасности для разработчиков, которым требуется разрабатывать и отлаживать обновления в рабочей ветви, но не нужно публиковать изменения в интерактивной службе.

**Ограничьте доступ разработчика** к определенным ресурсам.  Создавайте дополнительные группы безопасности с более детальной детализацией для разработчиков, которым необходим доступ только к конкретным ресурсам.  Назначьте этим группам соответствующие роли синапсе, областью действия которых являются определенные пулы Spark, среды выполнения интеграции или учетные данные.

**Ограничьте операторы от доступа к артефактам кода**.  Создайте группы безопасности для операторов, которым требуется отслеживать операционное состояние Синапсеных ресурсов и просматривать журналы, но не требуется доступ к коду или публикация обновлений в службе. Назначьте этим группам роль оператор вычислений, ограниченную конкретными пулами Spark и средами выполнения интеграции.  

## <a name="next-steps"></a>Следующие шаги

Узнайте, [как управлять назначениями РОЛЕЙ RBAC синапсе](./how-to-manage-synapse-rbac-role-assignments.md) создание [рабочей области синапсе](../quickstart-create-workspace.md)
