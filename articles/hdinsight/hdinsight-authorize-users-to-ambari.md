---
title: Предоставление пользователям доступа к представлениям Ambari в Azure HDInsight
description: Как управлять разрешениями пользователей и групп Ambari для кластеров HDInsight с включенным ESP.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/27/2019
ms.openlocfilehash: 0b6d4dacf43d80f371a2a4d286af1972198c3be3
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98942707"
---
# <a name="authorize-users-for-apache-ambari-views"></a>Предоставление пользователям доступа к представлениям Apache Ambari

[Кластеры HDInsight с поддержкой Корпоративного пакета безопасности](./domain-joined/hdinsight-security-overview.md) обеспечивают возможности корпоративного уровня, включая аутентификацию на основе Azure Active Directory. Вы можете [синхронизировать новых пользователей](hdinsight-sync-aad-users-to-cluster.md), добавленных в группы Azure Active Directory, которым был предоставлен доступ к кластеру, что позволит определенным пользователям выполнять конкретные действия. Работа с пользователями, группами и разрешениями в [Apache Ambari](https://ambari.apache.org/) поддерживается как для кластера ESP HDInsight, так и для стандартного кластера HDInsight.

Active Directory пользователи могут входить на узлы кластера, используя их учетные данные домена. Они также могут использовать учетные данные домена для проверки подлинности взаимодействия в кластере с другими утвержденными конечными точками, такими как [оттенок](https://gethue.com/), Ambariные представления, ODBC, JDBC, PowerShell и API-интерфейсы RESTful.

> [!WARNING]  
> Не изменяйте пароль модуля наблюдения Ambari (hdinsightwatchdog) в кластере HDInsight под управлением Linux. Это не позволит выполнять действия сценария или операции масштабирования в кластере.

Выполните [эти инструкции](./domain-joined/apache-domain-joined-configure-using-azure-adds.md), чтобы подготовить новый кластер ESP, если у вас его еще нет.

## <a name="access-the-ambari-management-page"></a>Переход на страницу управления Ambari

Чтобы открыть **страницу управления Ambari** в [пользовательском веб-интерфейсе Apache Ambari](hdinsight-hadoop-manage-ambari.md), перейдите по адресу `https://CLUSTERNAME.azurehdinsight.net`. Введите имя пользователя и пароль администратора кластера, определенные при создании кластера. Затем на панели мониторинга Ambari выберите **Manage Ambari** (Управление Ambari) в меню **admin** (Администрирование).

![Управление панелью Apache Ambari](./media/hdinsight-authorize-users-to-ambari/manage-apache-ambari.png)

## <a name="add-users"></a>Добавление пользователей

### <a name="add-users-through-the-portal"></a>Добавление пользователей на портале

1. На странице Управление выберите **Пользователи**.

    ![Пользователи страницы управления Apache Ambari](./media/hdinsight-authorize-users-to-ambari/apache-ambari-management-page-users.png)

1. Выберите **+ создать локального пользователя**.

1. Укажите **имя пользователя** и **пароль**. Нажмите кнопку **сохранить**.

### <a name="add-users-through-powershell"></a>Добавление пользователей с помощью PowerShell

Измените приведенные ниже переменные, заменив `CLUSTERNAME` , `NEWUSER` и `PASSWORD` соответствующими значениями.

```powershell
# Set-ExecutionPolicy Unrestricted

# Begin user input; update values
$clusterName="CLUSTERNAME"
$user="NEWUSER"
$userpass='PASSWORD'
# End user input

$adminCredentials = Get-Credential -UserName "admin" -Message "Enter admin password"

$clusterName = $clusterName.ToLower()
$createUserUrl="https://$($clusterName).azurehdinsight.net/api/v1/users"

$createUserBody=@{
    "Users/user_name" = "$user"
    "Users/password" = "$userpass"
    "Users/active" = "$true"
    "Users/admin" = "$false"
} | ConvertTo-Json

# Create user
$statusCode =
Invoke-WebRequest `
    -Uri $createUserUrl `
    -Credential $adminCredentials `
    -Method POST `
    -Headers @{"X-Requested-By" = "ambari"} `
    -Body $createUserBody | Select-Object -Expand StatusCode

if ($statusCode -eq 201) {
    Write-Output "User is created: $user"
}
else
{
    Write-Output 'User is not created'
    Exit
}

$grantPrivilegeUrl="https://$($clusterName).azurehdinsight.net/api/v1/clusters/$($clusterName)/privileges"

$grantPrivilegeBody=@{
    "PrivilegeInfo" = @{
        "permission_name" = "CLUSTER.USER"
        "principal_name" = "$user"
        "principal_type" = "USER"
    }
} | ConvertTo-Json

# Grant privileges
$statusCode =
Invoke-WebRequest `
    -Uri $grantPrivilegeUrl `
    -Credential $adminCredentials `
    -Method POST `
    -Headers @{"X-Requested-By" = "ambari"} `
    -Body $grantPrivilegeBody | Select-Object -Expand StatusCode

if ($statusCode -eq 201) {
    Write-Output 'Privilege is granted'
}
else
{
    Write-Output 'Privilege is not granted'
    Exit
}

Write-Host "Pausing for 100 seconds"
Start-Sleep -s 100

$userCredentials = "$($user):$($userpass)"
$encodedUserCredentials = [System.Convert]::ToBase64String([System.Text.Encoding]::ASCII.GetBytes($userCredentials))
$zookeeperUrlHeaders = @{ Authorization = "Basic $encodedUserCredentials" }
$getZookeeperurl="https://$($clusterName).azurehdinsight.net/api/v1/clusters/$($clusterName)/services/ZOOKEEPER/components/ZOOKEEPER_SERVER"

# Perform query with new user
$zookeeperHosts =
Invoke-WebRequest `
    -Uri $getZookeeperurl `
    -Method Get `
    -Headers $zookeeperUrlHeaders

Write-Output $zookeeperHosts
```

### <a name="add-users-through-curl"></a>Добавление пользователей через фигуру

Измените приведенные ниже переменные, заменив `CLUSTERNAME` ,, `ADMINPASSWORD` `NEWUSER` и `USERPASSWORD` соответствующими значениями. Скрипт предназначен для выполнения с bash. Для командной строки Windows потребуются небольшие изменения.

```bash
export clusterName="CLUSTERNAME"
export adminPassword='ADMINPASSWORD'
export user="NEWUSER"
export userPassword='USERPASSWORD'

# create user
curl -k -u admin:$adminPassword -H "X-Requested-By: ambari" -X POST \
-d "{\"Users/user_name\":\"$user\",\"Users/password\":\"$userPassword\",\"Users/active\":\"true\",\"Users/admin\":\"false\"}" \
https://$clusterName.azurehdinsight.net/api/v1/users

echo "user created: $user"

# grant permissions
curl -k -u admin:$adminPassword -H "X-Requested-By: ambari" -X POST \
-d '[{"PrivilegeInfo":{"permission_name":"CLUSTER.USER","principal_name":"'$user'","principal_type":"USER"}}]' \
https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/privileges

echo "Privilege is granted"

echo "Pausing for 100 seconds"
sleep 10s

# perform query using new user account
curl -k -u $user:$userPassword -H "X-Requested-By: ambari" \
-X GET "https://$clusterName.azurehdinsight.net/api/v1/clusters/$clusterName/services/ZOOKEEPER/components/ZOOKEEPER_SERVER"
```

## <a name="grant-permissions-to-apache-hive-views"></a>Предоставление разрешений для представлений Apache Hive

В Ambari доступны экземпляры представлений для [Apache Hive](https://hive.apache.org/) и [Apache TEZ](https://tez.apache.org/), среди прочих. Чтобы предоставить доступ к одному или нескольким экземплярам представлений Hive, перейдите на **страницу управления Ambari**.

1. На странице управления щелкните ссылку **Views** (Представления) под заголовком меню **Views** (Представления) в левой части экрана.

    ![Ссылки представления представлений Apache Ambari](./media/hdinsight-authorize-users-to-ambari/apache-ambari-views-link.png)

2. На странице "Views" (Представления) разверните строку **HIVE**. При добавлении службы Hive в кластер создается представление по умолчанию Hive. При необходимости можно также создать дополнительные экземпляры представлений Hive. Выберите представление Hive.

    ![Представления HDInsight — представление Apache Hive](./media/hdinsight-authorize-users-to-ambari/views-apache-hive-view.png)

3. Прокрутите страницу "Views" (Представления) вниз. В разделе *Permissions* (Разрешения) доступно два параметра для предоставления разрешений на представление пользователям домена:

**Grant permission to these users** ![Grant permission to these users](./media/hdinsight-authorize-users-to-ambari/hdi-add-user-to-view.png) (Предоставить разрешение данным пользователям)

**Grant permission to these groups** ![Grant permission to these groups](./media/hdinsight-authorize-users-to-ambari/add-group-to-view-permission.png) (Предоставить разрешение данным группам)

1. Чтобы добавить пользователя, нажмите кнопку **Add User** (Добавить пользователя).

   * Начните вводить имя пользователя, и вы увидите раскрывающийся список ранее определенных имен.

     ![Автоматическое завершение пользователем Apache Ambari](./media/hdinsight-authorize-users-to-ambari/ambari-user-autocomplete.png)

   * Выберите имя пользователя из списка или завершите его ввод. Чтобы добавить нового пользователя с этим именем, нажмите кнопку **New** (Создать).

   * Чтобы сохранить изменения, щелкните **синий флажок**.

     ![Разрешения пользователя для Apache Ambari Grant](./media/hdinsight-authorize-users-to-ambari/user-entered-permissions.png)

1. Чтобы добавить группу, нажмите кнопку **Add Group** (Добавить группу).

   * Начните вводить имя группы. Процедура выбора существующего имени группы или добавления новой группы выполняется так же, как и добавление пользователей.
   * Чтобы сохранить изменения, щелкните **синий флажок**.

     ![Разрешения на предоставление доступа Apache Ambari](./media/hdinsight-authorize-users-to-ambari/ambari-group-entered.png)

Непосредственное добавление пользователей для представления удобно, когда пользователю нужно назначить разрешения для использования этого представления, но вы не хотите добавлять его в группу, имеющую дополнительные разрешения. Чтобы уменьшить объем административных операций, может оказаться проще назначать разрешения группам.

## <a name="grant-permissions-to-apache-tez-views"></a>Предоставление разрешений для представлений Apache TEZ

С помощью экземпляров представлений [Apache TEZ](https://tez.apache.org/) пользователи могут отслеживать и отлаживать все задания Tez, отправляемые запросами [Apache Hive](https://hive.apache.org/) и сценариями [Apache Pig](https://pig.apache.org/). Доступен один экземпляр представления по умолчанию Tez, создаваемый при подготовке кластера.

Чтобы назначить пользователей и группы для экземпляра представления Tez, разверните строку **TEZ** на странице "Views" (Представления), как было описано выше.

![Представления HDInsight — представление Apache Tez](./media/hdinsight-authorize-users-to-ambari/views-apache-tez-view.png)

Чтобы добавить пользователей или группы, повторите шаги 3–5 из предыдущего раздела.

## <a name="assign-users-to-roles"></a>Назначение пользователей ролям

Существуют пять ролей безопасности для пользователей и групп, которые приведены ниже в порядке убывания уровня разрешений на доступ:

* администратор кластера;
* оператор кластера;
* Администратор служб
* оператор службы;
* пользователь кластера.

Для управления ролями перейдите на **страницу управления Ambari**, а затем щелкните ссылку **Roles** (Роли) в группе меню *Clusters* (Кластеры) в левой части экрана.

![Ссылки меню ролей Apache Ambari](./media/hdinsight-authorize-users-to-ambari/cluster-roles-menu-link.png)

Чтобы просмотреть список разрешений, предоставляемых каждой роли, щелкните синий вопросительный знак рядом с заголовком таблицы **Roles** (Роли) на странице "Roles" (Роли).

![Меню "роли" Apache Ambari. Связывание разрешений](./media/hdinsight-authorize-users-to-ambari/roles-menu-permissions.png "Меню "роли" Apache Ambari. Связывание разрешений")

На этой странице размещены два разных представления, которые можно использовать для управления ролями пользователей и групп: "Block" (Блок) и "List" (Список).

### <a name="block-view"></a>Представление "Block" (Блок)

Представление "Block" (Блок) отображает каждую роль в отдельной строке, а также содержит параметры **Assign roles to these users** (Назначить роли этим пользователям) и **Assign roles to these groups** (Назначить роли этим группам), как описано выше.

![Представление блокировки ролей Apache Ambari](./media/hdinsight-authorize-users-to-ambari/ambari-roles-block-view.png)

### <a name="list-view"></a>Представление списка

Представление "List" (Список) предоставляет возможности быстрого редактирования в двух категориях: "Users" (Пользователи) и "Groups" (Группы).

* В категории "Users" (Пользователи) представления "List" (Список) отображается список всех пользователей, в котором из раскрывающегося списка можно выбрать роль для каждого пользователя.

    ![Представление списка ролей Apache Ambari — пользователи](./media/hdinsight-authorize-users-to-ambari/roles-list-view-users.png)

* Категория "Groups" (Группы) представления "List" (Список) содержит все группы, а также роли, назначенные каждой группе. В нашем примере список групп синхронизирован с группами Azure AD, указанными в свойстве **Access user group** (Группа доступа пользователей) параметров домена кластера. См. [Создание кластера HDInsight с корпоративным пакетом безопасности](./domain-joined/apache-domain-joined-configure-using-azure-adds.md#create-an-hdinsight-cluster-with-esp).

    ![Представление списка ролей Apache Ambari — группы](./media/hdinsight-authorize-users-to-ambari/roles-list-view-groups.png)

    На приведенном выше рисунке группе hiveusers назначена роль *Cluster User* (Пользователь кластера). Это роль только для чтения, которая позволяет пользователям в группе просматривать, но не изменять конфигурации службы и метрики кластера.

## <a name="log-in-to-ambari-as-a-view-only-user"></a>Вход в Ambari от имени пользователя с доступом только к представлениям

Мы назначили пользователю домена Azure AD hiveuser1 разрешения для представлений Hive и Tez. После запуска пользовательского веб-интерфейса Ambari и ввода учетных данных домена пользователя (имя пользователя Azure AD в формате электронного адреса и пароль) выполняется перенаправление на страницу Ambari Views. Здесь пользователь сможет выбрать любое доступное представление. Пользователь не может посетить какую-либо другую часть сайта, включая страницы панели мониторинга, служб, узлов, предупреждений или администрирования.

![Пользователь Apache Ambari с только представлениями](./media/hdinsight-authorize-users-to-ambari/ambari-user-views-only.png)

## <a name="log-in-to-ambari-as-a-cluster-user"></a>Вход в Ambari от имени пользователя кластера

Мы назначили пользователю домена Azure AD hiveuser2 роль *Cluster User* (Пользователь кластера). Эта роль предоставляет доступ к панели мониторинга и всем пунктам меню. Пользователю кластера доступно меньше параметров, чем администратору. Например, пользователь hiveuser2 может просмотреть конфигурации для каждой из служб, но не может их изменить.

![Отображение панели мониторинга Apache Ambari](./media/hdinsight-authorize-users-to-ambari/user-cluster-user-role.png)

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка политик Apache Hive в кластере HDInsight с ESP](./domain-joined/apache-domain-joined-run-hive.md)
* [Управление кластерами в HDInsight с ESP](./domain-joined/apache-domain-joined-manage.md)
* [Использование представления Apache Hive с Apache Hadoop в HDInsight](hadoop/apache-hadoop-use-hive-ambari-view.md)
* [Синхронизация пользователей Azure Active Directory с кластером HDInsight](hdinsight-sync-aad-users-to-cluster.md)
* [Управление кластерами HDInsight с помощью Apache Ambari REST API](./hdinsight-hadoop-manage-ambari-rest-api.md)