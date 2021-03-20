---
title: Проверка подлинности выделенного пула SQL (ранее — хранилище данных SQL)
description: Узнайте, как выполнить аутентификацию в выделенном пуле SQL (ранее — в хранилище данных SQL) в Azure синапсе Analytics с помощью Azure Active Directory (Azure AD) или проверки подлинности SQL Server.
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/02/2019
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019
tag: azure-synapse
ms.openlocfilehash: 80bc9f6fc6af94ba2a5ade77cc1d53b3fc29f1ea
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98685350"
---
# <a name="authenticate-to-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Проверка подлинности в выделенном пуле SQL (ранее — хранилище данных SQL) в Azure синапсе Analytics

Узнайте, как выполнить аутентификацию в выделенном пуле SQL (ранее — в хранилище данных SQL) в Azure синапсе с помощью Azure Active Directory (Azure AD) или SQL Server проверки подлинности.

Для подключения к выделенному пулу SQL (ранее — ХРАНИЛИЩу данных SQL) необходимо передать учетные данные безопасности для проверки подлинности. После установки подключения некоторые его параметры настраиваются при установке сеанса запроса.  

Дополнительные сведения о безопасности и о том, как включить подключения к выделенному пулу SQL (ранее — SQL DW), см. [в документации по защите базы данных](sql-data-warehouse-overview-manage-security.md).

## <a name="sql-authentication"></a>Проверка подлинности SQL

Для подключения к выделенному пулу SQL (ранее — хранилище данных SQL) необходимо предоставить следующие сведения:

* Полное имя сервера
* Тип проверки подлинности SQL
* Имя пользователя
* Пароль
* База данных по умолчанию (необязательно)

По умолчанию устанавливается подключение к базе данных *master*, а не к пользовательской базе данных. Для подключения к пользовательской базе данных можно выполнить одно из следующих действий:

* Укажите базу данных по умолчанию при регистрации сервера в обозревателе объектов SQL Server в SSDT, SSMS или в строке подключения приложения. Например, добавьте параметр InitialCatalog для подключения ODBC.
* Выделите пользовательскую базу данных перед созданием сеанса в SSDT.

> [!NOTE]
> Использование инструкции Transact-SQL **USE MyDatabase;** для изменения базы данных, к которой осуществляется подключение, не поддерживается. Инструкции по подключению к пулу SQL с помощью Visual Studio и SSDT см. в [этой статье](sql-data-warehouse-query-visual-studio.md).

## <a name="azure-active-directory-authentication"></a>Проверка подлинности Azure Active Directory

Проверка подлинности [Azure Active Directory](../../active-directory/fundamentals/active-directory-whatis.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) — это механизм подключения к пулу SQL с помощью удостоверений в Azure Active Directory (Azure AD). С помощью аутентификации Azure Active Directory можно централизованно управлять удостоверениями пользователей базы данных и другими службами Майкрософт. Централизованное управление ИДЕНТИФИКАТОРами предоставляет единое место для управления выделенными пользователями пула SQL (ранее — SQL DW) и упрощает управление разрешениями.

### <a name="benefits"></a>Преимущества

Преимущества Azure Active Directory:

* наличие альтернативы аутентификации SQL Server;
* возможность остановить увеличение количества пользователей на серверах;
* изменение паролей в одном расположении;
* управление разрешениями базы данных с помощью внешних групп Azure AD;
* возможность исключить хранение паролей с помощью встроенной проверки подлинности Windows и других видов аутентификации, поддерживаемых Azure Active Directory;
* для проверки подлинности удостоверений на уровне базы данных используются данные пользователей автономной базы данных;
* поддержка аутентификации на основе маркеров для приложений, подключающихся к пулу SQL;
* поддержка Многофакторной идентификации с помощью универсальной аутентификации Active Directory для различных инструментов, включая [SQL Server Management Studio](../../azure-sql/database/authentication-mfa-ssms-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) и [SQL Server Data Tools](/sql/ssdt/azure-active-directory?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

> [!NOTE]
> Azure Active Directory является сравнительно новым компонентом и имеет некоторые ограничения. Чтобы убедиться, что Azure Active Directory подходит для вашей среды, ознакомьтесь с разделом [Функции и ограничения Azure AD](../../azure-sql/database/authentication-aad-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json#azure-ad-features-and-limitations), в частности с его подразделом "Дополнительные замечания".

### <a name="configuration-steps"></a>Этапы настройки

Следуйте приведенным ниже инструкциям, чтобы настроить аутентификацию Azure Active Directory.

1. Создание и заполнение каталога Azure Active Directory
2. Необязательное действие: Свяжите или измените каталог Active Directory, который сейчас связан с вашей подпиской Azure.
3. Создание администратора Azure Active Directory для хранилища данных Azure Synapse
4. Настройка клиентских компьютеров
5. Создание пользователей автономной базы данных в базе данных, сопоставленной с удостоверениями Azure AD
6. Подключение к пулу SQL с помощью удостоверений Azure AD

Сейчас пользователи Azure Active Directory не отображаются в обозревателе объектов SSDT. Сведения о пользователях можно просмотреть в файле [sys.database_principals](/sql/relational-databases/system-catalog-views/sys-database-principals-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

### <a name="find-the-details"></a>Поиск подробных сведений

* Процедуры настройки и использования аутентификации Azure Active Directory практически идентичны для базы данных SQL Azure и Synapse SQL в Azure Synapse. Выполните действия, описанные в разделе [Подключение к Базе данных SQL или пулу SQL c использованием проверки подлинности Azure Active Directory](../../azure-sql/database/authentication-aad-overview.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).
* Создайте пользовательские роли базы данных и назначьте их пользователям. Затем предоставьте ролям управляемые разрешения. Дополнительные сведения см. в разделе [Приступая к работе с разрешениями Database Engine](/sql/relational-databases/security/authentication-access/getting-started-with-database-engine-permissions?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="next-steps"></a>Дальнейшие действия

Чтобы приступить к отправке запросов с помощью Visual Studio и других приложений, см. статью [Connect to Azure Synapse Analytics with Visual Studio and SSDT](sql-data-warehouse-query-visual-studio.md) (Подключение к Azure Synapse Analytics с помощью Visual Studio и SSDT).
