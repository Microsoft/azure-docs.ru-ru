---
title: Условный доступ
description: Узнайте, как настроить условный доступ для базы данных SQL Azure, Управляемый экземпляр Azure SQL и Azure синапсе Analytics.
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.topic: how-to
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.custom: sqldbrb=1
ms.date: 04/28/2020
tag: azure-synpase
ms.openlocfilehash: c18d235977f1256a10e813fa8e02aa3590366fe1
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97936419"
---
# <a name="conditional-access-with-azure-sql-database-and-azure-synapse-analytics"></a>Условный доступ с помощью базы данных SQL Azure и Azure синапсе Analytics

[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

[База данных SQL Azure](sql-database-paas-overview.md), [Azure SQL управляемый экземпляр](../managed-instance/sql-managed-instance-paas-overview.md)и [Azure синапсе Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) поддерживают условный доступ Майкрософт.

Ниже описано, как настроить базу данных SQL Azure, SQL Управляемый экземпляр или Azure синапсе для применения политики условного доступа.  

## <a name="prerequisites"></a>Предварительные условия

- Для поддержки проверки подлинности Azure Active Directory (Azure AD) необходимо настроить базу данных SQL Azure, Управляемый экземпляр Azure SQL или выделенный пул SQL в Azure синапсе. Дополнительные сведения см. в статье [Настройка проверки подлинности Azure Active Directory с помощью базы данных SQL или Azure синапсе и управление ею](authentication-aad-configure.md).  
- Если включена многофакторная проверка подлинности, необходимо подключиться с помощью поддерживаемого средства, например последней версии SQL Server Management Studio (SSMS). Дополнительные сведения см. в разделе [Настройка многофакторной проверки подлинности в Базе данных SQL Azure для SQL Server Management Studio](authentication-mfa-ssms-configure.md).  

## <a name="configure-conditional-access"></a>Настройка условного доступа

> [!NOTE]
> В приведенном ниже примере используется база данных SQL Azure, но следует выбрать соответствующий продукт, для которого требуется настроить условный доступ.

1. Войдите в портал Azure, выберите **Azure Active Directory**, а затем выберите **Условный доступ**. Дополнительные сведения см. в статье [Техническая информация об условном доступе в Azure Active Directory](../../active-directory/conditional-access/concept-conditional-access-conditions.md).  
   ![Колонка условного доступа](./media/conditional-access-configure/conditional-access-blade.png)

2. В колонке **политик условного доступа** щелкните **Создать политику**, укажите имя, а затем щелкните **Configure rules** (Настройка правил).  
3. В разделе **назначения** выберите **Пользователи и группы**, установите флажок **выбрать пользователей и группы**, а затем выберите пользователя или группу для условного доступа. Щелкните **Выбрать** , а затем **Готово** ,чтобы подтвердить выбор.  
   ![выбор пользователей и групп](./media/conditional-access-configure/select-users-and-groups.png)  

4. Выберите **Облачные приложения** и щелкните **Выбрать приложения**. Вы увидите все приложения, доступные для условного доступа. Выберите **База данных SQL Azure**, в нижней области щелкните **Выбрать**, а затем щелкните **Готово**.  
   ![выбор базы данных SQL](./media/conditional-access-configure/select-sql-database.png)  
   Если вы не можете найти **базу данных SQL Azure** , указанную на следующем третьем снимке экрана, выполните следующие действия.
   - Подключитесь к базе данных в базе данных SQL Azure с помощью среды SSMS с учетной записью администратора Azure AD.  
   - Выполните процедуру `CREATE USER [user@yourtenant.com] FROM EXTERNAL PROVIDER`.  
   - Войдите в Azure AD и убедитесь, что база данных SQL Azure, SQL Управляемый экземпляр или Azure синапсе указаны в приложениях в вашем экземпляре Azure AD.  

5. Выберите **Элементы управления доступом**, выберите **Предоставить**, а затем просмотрите политику, которую вы хотите применить. Для этого примера мы выберем **Требовать многофакторную проверку подлинности**.  
   ![выбор предоставления доступа](./media/conditional-access-configure/grant-access.png)  

## <a name="summary"></a>Итоги

Выбранное приложение (база данных SQL Azure) с помощью Azure AD Premium теперь обеспечивает выбранную политику условного доступа, **необходимую многофакторную проверку подлинности.**

Вопросы о базе данных SQL Azure и Azure синапсе в отношении многофакторной идентификации см. по адресу <MFAforSQLDB@microsoft.com> .  

## <a name="next-steps"></a>Дальнейшие действия  

Руководство см. в статье [Защита базы данных в базе данных SQL](secure-database-tutorial.md).