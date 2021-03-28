---
title: Роль "Читатели каталогов" в Azure Active Directory для Azure SQL
description: Сведения о роли читателя каталога в Azure AD для SQL Azure.
ms.service: sql-db-mi
ms.subservice: security
ms.custom: azure-synapse
ms.topic: conceptual
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 08/14/2020
ms.openlocfilehash: d764c6b6cff6a0ba23d659d4fda63e21aac9b155
ms.sourcegitcommit: c8b50a8aa8d9596ee3d4f3905bde94c984fc8aa2
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/28/2021
ms.locfileid: "105644183"
---
# <a name="directory-readers-role-in-azure-active-directory-for-azure-sql"></a>Роль "Читатели каталогов" в Azure Active Directory для Azure SQL

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

> [!NOTE]
> Эта функция в этой статье доступна в **общедоступной предварительной версии**.

Azure Active Directory (Azure AD) предоставила [Использование облачных групп для управления назначениями ролей в Azure Active Directory (Предварительная версия)](../../active-directory/roles/groups-concept.md). Это позволяет назначать роли Azure AD группам.

При включении [управляемого удостоверения](../../active-directory/managed-identities-azure-resources/overview.md#managed-identity-types) для базы данных SQL Azure, azure SQL управляемый экземпляр или Azure синапсе Analytics роль " [**читатели каталога**](../../active-directory/roles/permissions-reference.md#directory-readers) Azure AD" должна быть назначена удостоверению, чтобы разрешить доступ на чтение к [API Graph Azure AD](/graph/migrate-azure-ad-graph-planning-checklist). Управляемое удостоверение базы данных SQL и Azure синапсе называется удостоверением сервера. Управляемое удостоверение SQL Управляемый экземпляр называется удостоверением управляемого экземпляра и автоматически назначается при создании экземпляра. Дополнительные сведения о назначении удостоверения сервера для базы данных SQL или Azure синапсе см. в статье [Включение субъектов-служб для создания пользователей Azure AD](authentication-aad-service-principal.md#enable-service-principals-to-create-azure-ad-users).

Роль " **читатели каталога** " необходима для:

- Создание имен входа Azure AD для SQL Управляемый экземпляр
- Олицетворение пользователей Azure AD в Azure SQL
- Миграция SQL Server пользователей, использующих проверку подлинности Windows, в SQL Управляемый экземпляр с проверкой подлинности Azure AD (с помощью команды [ALTER USER (Transact-SQL)](/sql/t-sql/statements/alter-user-transact-sql?view=azuresqldb-mi-current&preserve-view=true#d-map-the-user-in-the-database-to-an-azure-ad-login-after-migration) )
- Изменение администратора Azure AD для Управляемый экземпляр SQL
- Разрешение [субъектов-служб (приложений)](authentication-aad-service-principal.md) создавать пользователей Azure AD в Azure SQL

## <a name="assigning-the-directory-readers-role"></a>Назначение роли читателей каталога

Чтобы назначить роль " [**читатели каталога**](../../active-directory/roles/permissions-reference.md#directory-readers) " удостоверению, требуется пользователь с разрешениями администратора [глобального администратора](../../active-directory/roles/permissions-reference.md#global-administrator) или [привилегированной роли](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) . Пользователи, которые часто управляют или развертывают базу данных SQL, SQL Управляемый экземпляр или Azure синапсе, могут не иметь доступа к этим ролям с высоким уровнем привилегий. Это часто может привести к осложнениям для пользователей, которые создают незапланированные ресурсы SQL Azure или нуждаются в помощи членов ролей с высоким уровнем привилегий, которые часто недоступны в крупных организациях.

Чтобы [настроить администратора Azure AD для Управляемого экземпляра SQL](authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance), удостоверению управляемого экземпляра необходимо назначить роль **Читатели каталогов**. 

Для Базы данных SQL или Azure Synapse при настройке администратора Azure AD для логического сервера назначение роли **Читатели каталогов** удостоверению сервера не требуется. Однако для создания объекта Azure AD в Базе данных SQL или Azure Synapse от имени приложения Azure AD требуется роль **Читатели каталогов**. Если роль удостоверению логического сервера SQL не назначена, создание пользователей Azure AD в Azure SQL завершится ошибкой. Дополнительные сведения см. в статье [Субъект-служба Azure Active Directory с SQL Azure](authentication-aad-service-principal.md).

## <a name="granting-the-directory-readers-role-to-an-azure-ad-group"></a>Предоставление роли "читатели каталога" группе Azure AD

Сейчас в **общедоступной предварительной версии** вы можете создать группу Azure AD с правами [глобального администратора](../../active-directory/roles/permissions-reference.md#global-administrator) или [администратора привилегированных ролей](../../active-directory/roles/permissions-reference.md#privileged-role-administrator) и назначить ей разрешение [**чтение каталога**](../../active-directory/roles/permissions-reference.md#directory-readers) . Это позволит получить доступ к API Graph Azure AD для членов этой группы. Кроме того, пользователи Azure AD, являющиеся владельцами этой группы, могут назначать новые члены для этой группы, включая удостоверения логических серверов Azure SQL.

Этому решению по-прежнему требуется пользователь с высоким уровнем прав (глобальный администратор или администратор привилегированных ролей) для создания группы и назначения пользователей в качестве однократного действия, но владельцы групп Azure AD смогут назначать дополнительные участники. Это избавляет от необходимости использования пользователя с высоким уровнем привилегий в будущем для настройки всех баз данных SQL, управляемых экземпляров SQL или серверов Azure синапсе в своем клиенте Azure AD.

## <a name="next-steps"></a>Дальнейшие шаги

> [!div class="nextstepaction"]
> [Руководство по назначению роли "Читатели каталогов" группе Azure AD и управлению назначениями ролей](authentication-aad-directory-readers-role-tutorial.md)