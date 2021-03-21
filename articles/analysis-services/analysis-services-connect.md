---
title: Подключение к Azure Analysis Services | Документация Майкрософт
description: Узнайте, как подключиться к серверу служб Analysis Services в Azure и получить от него данные.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 12/01/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: 4abe1e9c6f9d7b62792936f816b9c46a937be41a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96499430"
---
# <a name="connecting-to-servers"></a>Подключение к серверам

В этой статье описывается подключение к серверу с помощью приложений моделирования данных и управления, таких как SQL Server Management Studio (SSMS) или Visual Studio с Analysis Services проектами, а также с клиентскими приложениями для создания отчетов, такими как Microsoft Excel, Power BI Desktop или пользовательские приложения. Для подключений к службам Azure Analysis Services используется протокол HTTPS.

## <a name="client-libraries"></a>Клиентские библиотеки

[Получение последних версий клиентских библиотек](/analysis-services/client-libraries?view=azure-analysis-services-current&preserve-view=true)

Все подключения к серверу независимо от типа требуют обновленных клиентских библиотек AMO, ADOMD.NET и OLEDB для взаимодействия с сервером Analysis Services. Для SSMS, Visual Studio, Excel 2016 и более поздних версий и Power BI последние клиентские библиотеки устанавливаются или обновляются в ежемесячных выпусках. Однако в некоторых случаях последние версии могут отсутствовать в приложении. Например, если политики откладывают обновления или Microsoft 365 обновления находятся на отложенном канале.

> [!NOTE]
> Клиентские библиотеки не могут подключаться к Azure Analysis Services через прокси-серверы, для которых требуется имя пользователя и пароль. 

## <a name="server-name"></a>Имя сервера

При создании сервера служб Analysis Services в Azure вам нужно указать уникальное имя сервера и регион, в котором он будет создан. При указании имени сервера в подключении используется следующая схема именования:

```
<protocol>://<region>/<servername>
```
 Здесь protocol — это строка **asazure**, region — код URI региона, в котором был создан сервер (например, westus.asazure.windows.net), а servername — это имя сервера, уникальное в пределах указанного региона.

### <a name="get-the-server-name"></a>Получение имени сервера

На **портале Azure** выберите сервер, а затем щелкните **Обзор** > **Имя сервера** и скопируйте имя сервера. Если к этому серверу подключаются и другие пользователи в организации, вы можете сообщить им это имя сервера. Указывая имя сервера, необходимо использовать полный путь.

![Получение имени сервера в Azure](./media/analysis-services-deploy/aas-deploy-get-server-name.png)

> [!NOTE]
> Протокол для региона "Восточная часть США 2" — **aspaaseastus2**.

## <a name="connection-string"></a>Строка подключения

При подключении к службам Azure Analysis Services с помощью модели табличного объекта используйте следующие форматы строки подключения:

###### <a name="integrated-azure-active-directory-authentication"></a>Встроенная проверка подлинности Azure Active Directory

Служба встроенной проверки подлинности извлекает данные из кэша учетных данных Azure Active Directory, если он доступен. В противном случае отобразится окно входа в Azure.

```
"Provider=MSOLAP;Data Source=<Azure AS instance name>;"
```


###### <a name="azure-active-directory-authentication-with-username-and-password"></a>Проверка подлинности Azure Active Directory с использованием имени пользователя и пароля

```
"Provider=MSOLAP;Data Source=<Azure AS instance name>;User ID=<user name>;Password=<password>;Persist Security Info=True; Impersonation Level=Impersonate;";
```

###### <a name="windows-authentication-integrated-security"></a>Проверка подлинности Windows (встроенный механизм безопасности)

Используйте учетную запись Windows, с которой выполняется текущий процесс.

```
"Provider=MSOLAP;Data Source=<Azure AS instance name>; Integrated Security=SSPI;Persist Security Info=True;"
```

## <a name="connect-using-an-odc-file"></a>Подключение с помощью ODC-файла

Пользователи предыдущих версий Excel могут подключаться к серверу Azure Analysis Services с помощью файла подключения к данным Office (ODC). Дополнительные сведения см. в разделе [Создание файла подключения к данным Office (ODC-файла)](analysis-services-odc.md).

## <a name="connect-as-a-linked-server-from-sql-server"></a>Подключиться как связанный сервер из SQL Server

SQL Server может подключаться к ресурсу Azure Analysis Services как к [связанному серверу](/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine) , указав MSOLAP в качестве поставщика источника данных. Перед настройкой соединения с связанным сервером обязательно установите последнюю версию [клиентской библиотеки MSOLAP](/analysis-services/client-libraries?view=azure-analysis-services-current&preserve-view=true) (поставщика). 

Для соединений связанного сервера с Azure Analysis Services необходимо создать экземпляр поставщика MSOLAP вне процесса SQL Server. При настройке параметров связанных серверов убедитесь, что **не выбран** параметр **Разрешить необрабатываемые процессы** .

Если выбран параметр **Разрешить необработку** и экземпляр поставщика создается в SQL Server процессе, возвращается следующая ошибка:

```
OLE DB provider "MSOLAP" for linked server "(null)" returned message "The following system error occurred: ".

OLE DB provider "MSOLAP" for linked server "(null)" returned message "The connection failed because user credentials are needed and Sign-In UI is not allowed.".

Msg 7303, Level 16, State 1, Line 2
Cannot initialize the data source object of OLE DB provider "MSOLAP" for linked server "(null)".
```



## <a name="next-steps"></a>Дальнейшие действия

[Подключение к Excel](analysis-services-connect-excel.md)    
[Подключение с помощью Power BI](analysis-services-connect-pbi.md)   
[Управление службами Analysis Services](analysis-services-manage.md)