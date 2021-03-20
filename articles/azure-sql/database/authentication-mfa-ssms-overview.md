---
title: Использование многофакторной проверки подлинности Azure Active Directory
description: База данных SQL Azure, Azure SQL Управляемый экземпляр и Azure синапсе Analytics поддерживают подключения из SQL Server Management Studio (SSMS) с использованием Active Directory универсальной аутентификации.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
ms.custom: seoapril2019, has-adal-ref, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 09/28/2020
tags: azure-synapse
ms.openlocfilehash: 9afad44bcf67478a81e75c17d0ff8ffc6d8c65aa
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94841138"
---
# <a name="using-multi-factor-azure-active-directory-authentication"></a>Использование многофакторной проверки подлинности Azure Active Directory
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

База данных SQL Azure, Azure SQL Управляемый экземпляр и Azure синапсе Analytics поддерживают подключения из [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) с помощью *Azure Active Directory-Universal с* проверкой подлинности mfa. В этой статье обсуждаются различия между различными вариантами проверки подлинности, а также ограничения, связанные с использованием универсальной проверки подлинности в Azure Active Directory (Azure AD) для Azure SQL.

**Скачивание последней версии SSMS.** Скачайте последнюю версию SSMS на клиентский компьютер, воспользовавшись страницей [Скачивание SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms). 

[!INCLUDE[ssms-connect-azure-ad](../includes/ssms-connect-azure-ad.md)]


Для всех функций, описанных в этой статье, используйте по крайней мере версию 17.2 за июль 2017 года. Диалоговое окно "Последнее подключение" должно выглядеть следующим образом:

  ![Снимок экрана: диалоговое окно подключения к серверу в SQL Server Management Studio, в котором отображаются параметры типа сервера, имени сервера и проверки подлинности.](./media/authentication-mfa-ssms-overview/1mfa-universal-connect.png)

## <a name="authentication-options"></a>Параметры проверки подлинности

Существует две неинтерактивные модели проверки подлинности для Azure AD, которые можно использовать во многих разных приложениях (ADO.NET, ЖДКБ, ODC и т. д.). При использовании этих двух методов никогда не отображаются всплывающие диалоговые окна:

- `Azure Active Directory - Password`
- `Azure Active Directory - Integrated`

Интерактивный метод, который также поддерживает многофакторную идентификацию Azure AD (MFA),: 

- `Azure Active Directory - Universal with MFA`

Azure AD MFA помогает защитить доступ к данным и приложениям, а также удовлетворить потребность пользователей в простом процессе входа. Она обеспечивает надежную аутентификацию с использованием ряда простых вариантов подтверждения: телефонного звонка, текстового сообщения, смарт-карты с ПИН-кодом или уведомления в мобильном приложении. Пользователи сами выбирают предпочтительный способ подтверждения. Интерактивная MFA с использованием Azure AD может привести к появлению всплывающего диалогового окна для реализации проверки.

Описание многофакторной идентификации Azure AD [см. в](../../active-directory/authentication/concept-mfa-howitworks.md)этой статье.
Инструкции по настройке см. в разделе [Настройка многофакторной проверки подлинности в Базе данных SQL Azure для SQL Server Management Studio](authentication-mfa-ssms-configure.md).

### <a name="azure-ad-domain-name-or-tenant-id-parameter"></a>Параметр доменного имени или идентификатора клиента Azure AD

Начиная с [SSMS версии 17](/sql/ssms/download-sql-server-management-studio-ssms)пользователи, импортированные в текущую службу Azure AD из других каталогов Azure в качестве гостевых пользователей, могут указать доменное имя Azure AD или идентификатор клиента при подключении. К гостевым пользователям относятся пользователи, приглашенные из других каталогов Azure AD, учетные записи Майкрософт, такие как outlook.com, hotmail.com и live.com, или другие учетные записи, например, gmail.com. Эти сведения обеспечивают `Azure Active Directory - Universal with MFA` проверку подлинности для идентификации правильного центра проверки подлинности. Этот параметр также является обязательным для поддержки учетных записей Майкрософт (MSA), таких как outlook.com, hotmail.com или live.com, и сторонних учетных записей. 

Все гостевые пользователи, которые хотят пройти проверку подлинности с помощью универсальной проверки подлинности, должны ввести свое доменное имя или идентификатор клиента Azure AD. Этот параметр представляет текущее доменное имя или идентификатор клиента Azure AD, с которым связан логический сервер SQL Azure. Например, если логический сервер SQL связан с доменом Azure AD `contosotest.onmicrosoft.com` , где пользователь `joe@contosodev.onmicrosoft.com` размещен в качестве импортированного пользователя из домена Azure AD `contosodev.onmicrosoft.com` , доменное имя, необходимое для проверки подлинности этого пользователя, — `contosotest.onmicrosoft.com` . Если пользователь является собственным пользователем Azure AD, связанным с логическим сервером SQL, и не является учетной записью MSA, имя домена или идентификатор клиента не требуются. Чтобы ввести параметр (начиная с SSMS версии 17,2):


1. Откройте подключение в среде SSMS. Введите имя сервера и выберите **Azure Active Directory-Universal с** проверкой подлинности mfa. Добавьте **имя пользователя** , с которым хотите войти.
1. Выберите поле **Параметры** и перейдите на вкладку **Свойства подключения** . В диалоговом окне **Подключение к базе данных** заполните диалоговое окно для своей базы данных. Установите флажок **Доменное имя AD или идентификатор клиента** и укажите центр аутентификации, введя доменное имя (**contosotest.onmicrosoft.com**) или GUID идентификатора клиента. 

   ![Снимок экрана: вкладка "Свойства подключения", в которой выделяются параметры подключения к базе данных, доменному имени AD или ИДЕНТИФИКАТОРу клиента.](./media/authentication-mfa-ssms-overview/mfa-tenant-ssms.png)

Если вы используете SSMS 18. x или более поздней версии, имя домена AD или идентификатор клиента больше не требуется для гостевых пользователей, поскольку 18. x или более поздней версии автоматически распознает его.

   ![Снимок экрана: вкладка "Свойства подключения" диалогового окна "соединение с сервером" в s м S. в поле "подключение к базе данных" выбрано значение "MyDatabase".](./media/authentication-mfa-ssms-overview/mfa-no-tenant-ssms.png)

### <a name="azure-ad-business-to-business-support"></a>Поддержка Azure AD B2B

> [!IMPORTANT]
> Поддержка гостевых пользователей для подключения к базе данных SQL Azure, SQL Управляемый экземпляр и Azure синапсе без необходимости быть частью группы в настоящее время доступна в **общедоступной предварительной версии**. Дополнительные сведения см. в статье [Создание гостевых пользователей Azure AD и назначение их в качестве администратора Azure AD](authentication-aad-guest-users.md).

Пользователи Azure AD, которые поддерживаются в сценариях Azure AD B2B в качестве гостевых пользователей (см. раздел [что такое служба совместной работы Azure B2B](../../active-directory/external-identities/what-is-b2b.md)), могут подключаться к базе данных SQL и Azure синапсе только как часть членов группы, созданной в связанном Azure AD, и сопоставлять вручную с помощью инструкции [CREATE USER (Transact-SQL)](/sql/t-sql/statements/create-user-transact-sql) в данной базе данных. Например, если `steve@gmail.com` приглашен в Azure AD `contosotest` (с доменом Azure AD `contosotest.onmicrosoft.com` ), Группа Azure AD, например, `usergroup` должна быть создана в Azure AD, содержащей `steve@gmail.com` член. Затем эту группу необходимо создать для конкретной базы данных (например, `MyDatabase` ) с помощью администратора SQL Azure AD или Azure AD dbo, выполнив инструкцию Transact-SQL `CREATE USER [usergroup] FROM EXTERNAL PROVIDER` . 

После создания пользователя базы данных пользователь `steve@gmail.com` может войти в систему `MyDatabase` с помощью параметра проверки подлинности SSMS `Azure Active Directory – Universal with MFA` . По умолчанию компонент `usergroup` имеет только разрешение CONNECT. Любой дополнительный доступ к данным должен быть [предоставлен](/sql/t-sql/statements/grant-transact-sql) пользователю с достаточными правами. 

> [!NOTE]
> Для SSMS 17. x, используя `steve@gmail.com` в качестве гостевого пользователя, необходимо проверить поле **доменное имя AD или идентификатор клиента** и добавить доменное имя AD `contosotest.onmicrosoft.com` в диалоговом окне **Свойства соединения** . Параметр **доменное имя AD или идентификатор клиента** поддерживается только для **Azure Active Directory-Universal с** проверкой подлинности mfa. В противном случае флажок будет серым.

## <a name="universal-authentication-limitations"></a>Ограничения универсальной проверки подлинности

- SSMS и SqlPackage.exe — единственные инструменты, в настоящее время поддерживающее MFA с помощью универсальной аутентификации Active Directory.
- SSMS версии 17,2 поддерживает одновременный доступ нескольких пользователей с помощью универсальной проверки подлинности с MFA. Для SSMS версии 17,0 и 17,1 средство ограничит имя входа для экземпляра SSMS с помощью универсальной проверки подлинности для одной учетной записи Azure Active Directory. Чтобы войти в качестве другой учетной записи Azure AD, необходимо использовать другой экземпляр SSMS. Это ограничение ограничено Active Directory универсальной аутентификации. Вы можете войти на другой сервер, используя `Azure Active Directory - Password` проверку подлинности, `Azure Active Directory - Integrated` проверку подлинности или `SQL Server Authentication` .
- SSMS поддерживает универсальную аутентификацию Active Directory для отображения обозревателя объектов, редактора запросов и хранилища запросов.
- SSMS версии 17.2 поддерживает мастер DacFx для экспорта, извлечения и развертывания данных базы данных. После того, как пользователь пройдет универсальную аутентификацию в диалоговом окне начальной аутентификации, мастер DacFx функционирует так же, как и для всех других методов аутентификации.
- Конструктор таблиц SSMS не поддерживает универсальную аутентификацию.
- Для универсальной аутентификации Active Directory не требуется дополнительное программное обеспечение, за исключением того, что необходимо использовать поддерживаемую версию SSMS.  
- См. следующую ссылку для последней версии Библиотека проверки подлинности Active Directory (ADAL) для универсальной проверки подлинности: [Microsoft. IdentityModel. Clients. ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/).  

## <a name="next-steps"></a>Дальнейшие действия

- Инструкции по настройке см. в разделе [Настройка многофакторной проверки подлинности в Базе данных SQL Azure для SQL Server Management Studio](authentication-mfa-ssms-configure.md).
- Предоставьте другим пользователям доступ к своей базе данных: [Проверка подлинности и авторизация в базе данных SQL: предоставление доступа](logins-create-manage.md).  
- Убедитесь, что другие пользователи могут подключаться через брандмауэр: [Настройка правила брандмауэра на уровне сервера с помощью портал Azure](firewall-configure.md)  
- [Настройка проверки подлинности Azure Active Directory и управление ею с помощью базы данных SQL или Azure синапсе](authentication-aad-configure.md)
- [Создание гостевых пользователей Azure AD и предоставление им прав администратора Azure AD](authentication-aad-guest-users.md) 
- [Платформа приложения уровня данных Microsoft® SQL Server® (17.0.0 GA)](https://www.microsoft.com/download/details.aspx?id=55088)  
- [SQLPackage.exe](/sql/tools/sqlpackage)  
- [Импорт BACPAC-файла в новую базу данных](database-import.md)  
- [Экспорт базы данных в BACPAC-файл](database-export.md)  
- Интерфейс C# [IUniversalAuthProvider](/dotnet/api/microsoft.sqlserver.dac.iuniversalauthprovider)  
- При использовании **Azure Active Directory-Universal с** аутентификацией MFA доступна трассировка ADAL, начиная с [SSMS 17,3](/sql/ssms/download-sql-server-management-studio-ssms). Трассировка ADAL по умолчанию отключена. Включите ее в меню **Средства**, **Параметры** в разделе **Службы Azure**, **Облако Azure**, **Уровень трассировки для окна вывода ADAL** после включения **Вывод** в меню **Представление**. Эти трассировки доступны в окне вывода при выборе **параметра Azure Active Directory**.