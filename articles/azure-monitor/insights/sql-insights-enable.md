---
title: Включить SQL Insights
description: Включение аналитики SQL в Azure Monitor
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/15/2021
ms.openlocfilehash: e8dd887d151eb553131048f232940555dbef324b
ms.sourcegitcommit: a8ff4f9f69332eef9c75093fd56a9aae2fe65122
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105025039"
---
# <a name="enable-sql-insights-preview"></a>Включить SQL Insights (Предварительная версия)
В этой статье описывается, как включить [SQL Insights](sql-insights-overview.md) для мониторинга развертываний SQL. Мониторинг выполняется из виртуальной машины Azure, которая устанавливает подключение к развертываниям SQL и использует динамические административные представления (DMV) для сбора данных мониторинга. Вы можете управлять сбором наборов данных и частотой сбора с помощью профиля мониторинга.

## <a name="create-log-analytics-workspace"></a>Создание рабочей области Log Analytics
SQL Insights хранит свои данные в одной или нескольких [рабочих областях log Analytics](../logs/data-platform-logs.md#log-analytics-workspaces).  Перед включением SQL Insights необходимо либо [создать рабочую область](../logs/quick-create-workspace.md) , либо выбрать существующую. Одну рабочую область можно использовать с несколькими профилями мониторинга, но Рабочая область и профили должны находиться в одном регионе Azure. Чтобы включить функции и получить доступ к ним в SQL Insights, необходимо иметь [роль участника log Analytics](../logs/manage-access.md) в рабочей области. 

## <a name="create-monitoring-user"></a>Создание пользователя мониторинга 
Необходимо иметь пользователя на развертываниях SQL, которые требуется отслеживать. Следуйте приведенным ниже процедурам для различных типов развертываний SQL.

### <a name="azure-sql-database"></a>База данных SQL Azure
Откройте базу данных SQL Azure с помощью [SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md) или [редактора запросов (Предварительная версия)](../../azure-sql/database/connect-query-portal.md) в портал Azure.

Выполните следующий скрипт, чтобы создать пользователя с необходимыми разрешениями. Замените *User* именем пользователя и *мистронгпассворд* паролем.

```
CREATE USER [user] WITH PASSWORD = N'mystrongpassword'; 
GO 
GRANT VIEW DATABASE STATE TO [user]; 
GO 
```

:::image type="content" source="media/sql-insights-enable/telegraf-user-database-script.png" alt-text="Создание пользовательского скрипта Telegraf." lightbox="media/sql-insights-enable/telegraf-user-database-script.png":::

Убедитесь, что пользователь создан.

:::image type="content" source="media/sql-insights-enable/telegraf-user-database-verify.png" alt-text="Проверьте пользовательский скрипт Telegraf." lightbox="media/sql-insights-enable/telegraf-user-database-verify.png":::

### <a name="azure-sql-managed-instance"></a>Управляемый экземпляр SQL Azure
Войдите в Управляемый экземпляр Azure SQL и используйте [SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md) или аналогичное средство для выполнения следующего скрипта, чтобы создать пользователя мониторинга с необходимыми разрешениями. Замените *User* именем пользователя и *мистронгпассворд* паролем.

 
```
USE master; 
GO 
CREATE LOGIN [user] WITH PASSWORD = N'mystrongpassword'; 
GO 
GRANT VIEW SERVER STATE TO [user]; 
GO 
GRANT VIEW ANY DEFINITION TO [user]; 
GO 
```

### <a name="sql-server"></a>SQL Server
Войдите на виртуальную машину Azure под управлением SQL Server и используйте [SQL Server Management Studio](../../azure-sql/database/connect-query-ssms.md) или аналогичное средство, чтобы выполнить следующий скрипт для создания пользователя мониторинга с необходимыми разрешениями. Замените *User* именем пользователя и *мистронгпассворд* паролем.

 
```
USE master; 
GO 
CREATE LOGIN [user] WITH PASSWORD = N'mystrongpassword'; 
GO 
GRANT VIEW SERVER STATE TO [user]; 
GO 
GRANT VIEW ANY DEFINITION TO [user]; 
GO
```

## <a name="create-azure-virtual-machine"></a>Создание виртуальной машины Azure 
Вам потребуется создать одну или несколько виртуальных машин Azure, которые будут использоваться для сбора данных для мониторинга SQL.  

> [!NOTE]
>  [Профили мониторинга](#create-sql-monitoring-profile) указывают, какие данные будут собираются из различных типов SQL, которые вы хотите отслеживать. С каждой виртуальной машиной мониторинга может быть связан только один профиль мониторинга. Если требуется несколько профилей мониторинга, необходимо создать виртуальную машину для каждой из них.

### <a name="azure-virtual-machine-requirements"></a>Требования к виртуальной машине Azure
Виртуальные машины Azure имеют следующие требования.

- Операционная система: Ubuntu 18,04 
- Рекомендуемые размеры виртуальных машин Azure: Standard_B2s (2 ЦП, 4 памяти гиб) 
- Поддерживаемые регионы: любой [регион, поддерживаемый агентом Azure Monitor](../agents/azure-monitor-agent-overview.md#supported-regions)

> [!NOTE]
> Размер виртуальной машины Standard_B2s (2 ЦП, 4 гиб Memory) будет поддерживать до 100 строк подключения. Не следует выделять более 100 подключений к одной виртуальной машине.

В зависимости от параметров сети ресурсов SQL виртуальные машины могут быть помещены в ту же виртуальную сеть, что и ресурсы SQL, чтобы они могли выполнять сетевые подключения для получения данных мониторинга.  

## <a name="configure-network-settings"></a>Настройка параметров сети
Каждый тип SQL предлагает методы для виртуальной машины мониторинга для безопасного доступа к SQL.  В следующих разделах рассматриваются варианты, основанные на типе SQL.

### <a name="azure-sql-databases"></a>Базы данных SQL Azure  

SQL Insights поддерживает доступ к базе данных SQL Azure через общедоступную конечную точку и из виртуальной сети.

Для доступа через общедоступную конечную точку необходимо добавить правило на странице **Параметры брандмауэра** и [Параметры IP-брандмауэра](https://docs.microsoft.com/azure/azure-sql/database/network-access-controls-overview#ip-firewall-rules) .  Для указания доступа из виртуальной сети можно задать [правила брандмауэра виртуальной сети](https://docs.microsoft.com/azure/azure-sql/database/network-access-controls-overview#virtual-network-firewall-rules) и задать [теги службы, необходимые для агента Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/agents/azure-monitor-agent-overview#networking).  В [этой статье](https://docs.microsoft.com/azure/azure-sql/database/network-access-controls-overview#ip-vs-virtual-network-firewall-rules) описываются различия между этими двумя типами правил брандмауэра.

:::image type="content" source="media/sql-insights-enable/set-server-firewall.png" alt-text="Настройка брандмауэра для сервера" lightbox="media/sql-insights-enable/set-server-firewall.png":::

:::image type="content" source="media/sql-insights-enable/firewall-settings.png" alt-text="Параметры брандмауэра." lightbox="media/sql-insights-enable/firewall-settings.png":::


### <a name="azure-sql-managed-instances"></a>Управляемые экземпляры SQL Azure 

Если виртуальная машина мониторинга будет находиться в той же виртуальной сети, что и ваши ресурсы SQL MI, см. раздел [подключение в той же виртуальной сети](https://docs.microsoft.com/azure/azure-sql/managed-instance/connect-application-instance#connect-inside-the-same-vnet). Если виртуальная машина мониторинга будет находиться в виртуальной сети, отличной от ресурсов SQL MI, то см. раздел [подключение в другой виртуальной сети](https://docs.microsoft.com/azure/azure-sql/managed-instance/connect-application-instance#connect-inside-a-different-vnet).


### <a name="azure-virtual-machine-and-azure-sql-virtual-machine"></a>Виртуальная машина Azure и виртуальная машина SQL Azure  
Если виртуальная машина мониторинга находится в той же виртуальной сети, что и ресурсы виртуальной машины SQL, см. сведения [в разделе Подключение к SQL Server в пределах виртуального окружения](https://docs.microsoft.com/azure/azure-sql/virtual-machines/windows/ways-to-connect-to-sql#connect-to-sql-server-within-a-virtual-network). Если виртуальная машина мониторинга будет находиться в сети, отличной от виртуальной машины SQL, см. раздел  [Подключение к SQL Server через Интернет](https://docs.microsoft.com/azure/azure-sql/virtual-machines/windows/ways-to-connect-to-sql#connect-to-sql-server-over-the-internet).

## <a name="store-monitoring-password-in-key-vault"></a>Хранение пароля мониторинга в Key Vault
Пароли для подключения пользователей SQL следует хранить в Key Vault вместо того, чтобы вводить их непосредственно в строки подключения профиля мониторинга.

При настройке профиля для мониторинга SQL необходимо иметь одно из следующих разрешений на ресурс Key Vault, который предполагается использовать:

- Microsoft.Authorization/roleAssignments/write. 
- Разрешения Microsoft. Authorization/roleAssignments/DELETE, такие как администратор доступа пользователей или владелец 

Новая политика доступа будет создана автоматически в ходе создания профиля SQL Monitoring, использующего указанный Key Vault. Используйте параметр *Разрешить доступ из всех сетей* для Key Vault параметров сети.


## <a name="create-sql-monitoring-profile"></a>Создание профиля мониторинга SQL
Откройте SQL Insights, выбрав **SQL (Предварительная версия)** в разделе **Insights** в меню **Azure Monitor** в портал Azure. Выберите команду **Создать новый профиль**. 

:::image type="content" source="media/sql-insights-enable/create-new-profile.png" alt-text="Создать новый профиль." lightbox="media/sql-insights-enable/create-new-profile.png":::

В профиле будут храниться сведения, которые необходимо получить из систем SQL.  Он имеет определенные параметры для: 

- База данных SQL Azure 
- Управляемые экземпляры SQL Azure 
- SQL Server, выполняемые на виртуальных машинах  

Например, можно создать один профиль с именем *SQL Production* , а другой — *промежуточное размещение SQL* с различными настройками частоты сбора данных, сбором данных и рабочей областью, в которую будут отправлены данные. 

Профиль хранится в качестве ресурса [правила сбора данных](../agents/data-collection-rule-overview.md) в выбранном подписке и группе ресурсов. Для каждого профиля необходимо следующее:

- Название Невозможно изменить после создания.
- Расположение. Это регион Azure.
- Log Analytics рабочей области для хранения данных мониторинга.
- Параметры коллекции для частоты и типа данных мониторинга SQL для сбора.

> [!NOTE]
> Расположение профиля должно находиться в том же расположении, что и Log Analytics рабочей области, в которую планируется отправить данные мониторинга.


:::image type="content" source="media/sql-insights-enable/profile-details.png" alt-text="Сведения о профиле." lightbox="media/sql-insights-enable/profile-details.png":::

После введения сведений о профиле мониторинга щелкните **создать профиль мониторинга** . Развертывание профиля может занять до минуты.  Если вы не видите новый профиль, указанный в поле со списком **профиль мониторинга** , нажмите кнопку обновить, и после завершения развертывания оно должно отобразиться.  Выбрав новый профиль, перейдите на вкладку **Управление профилем** , чтобы добавить компьютер мониторинга, который будет связан с профилем.

### <a name="add-monitoring-machine"></a>Добавление компьютера мониторинга
Выберите **Добавить компьютер мониторинга** , чтобы открыть панель контекста, чтобы выбрать виртуальную машину для настройки мониторинга экземпляров SQL и предоставления строк подключения.

Выберите подписку и имя виртуальной машины мониторинга. Если вы используете Key Vault для хранения пароля для пользователя мониторинга, выберите ресурсы Key Vault с этими секретами и введите URL-адрес и имя секрета для использования в строках подключения. Дополнительные сведения об определении строки подключения для различных развертываний SQL см. в следующем разделе.


:::image type="content" source="media/sql-insights-enable/add-monitoring-machine.png" alt-text="Добавление компьютера мониторинга." lightbox="media/sql-insights-enable/add-monitoring-machine.png":::


### <a name="add-connection-strings"></a>Добавление строк подключения 
Строка подключения указывает имя пользователя, которое SQL Insights должно использовать при входе в SQL для запуска динамических административных представлений. Если вы используете Key Vault для хранения пароля для пользователя мониторинга, укажите URL-адрес и имя секрета для использования. 

Строка подключения будет отличаться для каждого типа ресурсов SQL:

#### <a name="azure-sql-databases"></a>Базы данных SQL Azure 
Введите строку подключения в формате:

```
sqlAzureConnections": [ 
   "Server=mysqlserver.database.windows.net;Port=1433;Database=mydatabase;User Id=$username;Password=$password;" 
}
```

Получите сведения из пункта меню **строки подключения** для базы данных.

:::image type="content" source="media/sql-insights-enable/connection-string-sql-database.png" alt-text="Строка подключения к базе данных SQL" lightbox="media/sql-insights-enable/connection-string-sql-database.png":::

Для мониторинга доступной для чтения вторичной реплики включите значение ключа `ApplicationIntent=ReadOnly` в строку подключения.


#### <a name="azure-virtual-machines-running-sql-server"></a>Виртуальные машины Azure с SQL Server 
Введите строку подключения в формате:

```
"sqlVmConnections": [ 
   "Server=MyServerIPAddress;Port=1433;User Id=$username;Password=$password;" 
] 
```

Если виртуальная машина мониторинга находится в той же виртуальной сети, используйте частный IP-адрес сервера.  В противном случае используйте общедоступный IP-адрес. Если вы используете виртуальную машину SQL Azure, вы можете узнать, какой порт будет использоваться на странице " **Безопасность** " для ресурса.

:::image type="content" source="media/sql-insights-enable/sql-vm-security.png" alt-text="Безопасность виртуальных машин SQL" lightbox="media/sql-insights-enable/sql-vm-security.png":::

Для мониторинга доступной для чтения вторичной реплики включите значение ключа `ApplicationIntent=ReadOnly` в строку подключения.


### <a name="azure-sql-managed-instances"></a>Управляемые экземпляры SQL Azure 
Введите строку подключения в формате:

```
"sqlManagedInstanceConnections": [ 
      "Server= mysqlserver.database.windows.net;Port=1433;User Id=$username;Password=$password;", 
    ] 
```
Получите сведения из пункта меню **строки подключения** для управляемого экземпляра.


:::image type="content" source="media/sql-insights-enable/connection-string-sql-managed-instance.png" alt-text="Строка подключения SQL Управляемый экземпляр" lightbox="media/sql-insights-enable/connection-string-sql-managed-instance.png":::

Для мониторинга доступной для чтения вторичной реплики включите значение ключа `ApplicationIntent=ReadOnly` в строку подключения.



## <a name="monitoring-profile-created"></a>Профиль мониторинга создан 

Выберите **добавить мониторинг виртуальная машина** , чтобы настроить виртуальную машину для получения данных из ресурсов SQL. Не возвращайтесь на вкладку **Обзор** .  Через несколько минут столбец Status должен измениться на Reading (сбор). Вы увидите данные для ресурсов SQL, которые вы выбрали для отслеживания.

Если данные не отображаются, см. раздел [Устранение неполадок SQL Insights](sql-insights-troubleshoot.md) для выявления проблемы. 

:::image type="content" source="media/sql-insights-enable/profile-created.png" alt-text="Профиль создан" lightbox="media/sql-insights-enable/profile-created.png":::

> [!NOTE]
> Если вам нужно обновить профиль мониторинга или строки подключения на виртуальных машинах мониторинга, это можно сделать с помощью вкладки **профиль управления** SQL Insights.  После сохранения обновлений изменения будут применены примерно через 5 минут.

## <a name="next-steps"></a>Следующие шаги

- Если SQL Insights не работает должным образом после включения, см. раздел [Устранение неполадок SQL Insights](sql-insights-troubleshoot.md) .
