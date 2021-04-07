---
title: Краткое руководство. Просмотр групп и участников в Azure AD
description: Инструкции по поиску и просмотру групп вашей организации и назначенных в них членов.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: quickstart
ms.date: 09/24/2018
ms.author: ajburnle
ms.custom: it-pro, seodec18, seo-update-azuread-jan
ms.reviewer: krbain
ms.collection: M365-identity-device-management
ms.openlocfilehash: ddedd753eb98a8656006c9dddc412e44330e1a21
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95996735"
---
<!--As a brand-new Azure AD administrator, I need to view my organization’s groups along with the assigned members, so I can manage permissions to apps and services for people in my organization-->

# <a name="quickstart-view-your-organizations-groups-and-members-in-azure-active-directory"></a>Краткое руководство. Просмотр групп и членов вашей организации в Azure Active Directory
Вы можете просмотреть существующие в вашей организации группы и членов этих групп с помощью портала Azure. Группы используются для управления пользователями (членами), которым нужны одинаковые уровни доступа и разрешения в приложениях и службах с ограниченным доступом.

В этом кратком руководстве вы просмотрите все существующие в организации группы и назначенных членов.

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись Azure](https://azure.microsoft.com/free/), прежде чем начинать работу. 

## <a name="prerequisites"></a>Предварительные требования
Для этого потребуются следующие компоненты.

- Создайте клиента Azure Active Directory. Дополнительные сведения см. в статье [Quickstart: Access Azure Active Directory to create a new tenant](active-directory-access-create-new-tenant.md) (Краткое руководство. Использование Azure Active Directory для создания нового клиента).

## <a name="sign-in-to-the-azure-portal"></a>Вход на портал Azure
Для входа на [портал Azure](https://portal.azure.com/) используйте учетную запись глобального администратора вашего каталога.

## <a name="create-a-new-group"></a>Создание группы 
Создайте новую группу с именем _MDM policy - West_. Дополнительные сведения о создании группы см. в статье [Создание группы и добавление в нее пользователей в Azure Active Directory](active-directory-groups-create-azure-portal.md).

1. Выберите **Azure Active Directory**, затем **Группы** и щелкните **Новая группа**.

2. Заполните страницу **Группа**:
    
    - **Тип группы**: выберите **Безопасность**.
    
    - **Имя группы:** введите _MDM policy - West_.
    
    - **Тип членства:** выберите **Назначенные**.

3. Нажмите кнопку **создания**.

## <a name="create-a-new-user"></a>Создание пользователя
Создайте нового пользователя с именем _Alain Charon_. В качестве члена группы можно добавить только существующих пользователей. Сначала перейдите на вкладку "Имена личных доменов", чтобы получить проверенное доменное имя, в котором нужно создать пользователей. Дополнительные сведения о создании пользователей см. в [этой статье](add-users-azure-active-directory.md).

1. Выберите **Azure Active Directory**, затем **Пользователи** и щелкните **Новый пользователь**.

2. Заполните данные на странице **Пользователь**.

    - **Имя:** введите _Alain Charon_.

    - **Имя пользователя:** введите *alain\@contoso.com*.

3. Скопируйте автоматически созданный пароль, который подставляется в поле **Пароль**, а затем щелкните **Создать**.

## <a name="add-a-group-member"></a>Добавление члена группы
Теперь у вас есть группа и пользователь для нее. Добавьте _Alain Charon_ в число членов группы _MDM policy - West_. Дополнительные сведения о добавлении членов группы, см. в статье [Управление участниками групп в клиенте Azure Active Directory](active-directory-groups-members-azure-portal.md).

1. Выберите **Azure Active Directory** > **Группы**.

2. На странице **Группы — Все группы** найдите и выберите группу **MDM policy - West**.

3. На странице **Обзор политики управления мобильными устройствами — Запад** выберите **Члены** из области **Управление**.

4. Выберите **Добавление членов**, а затем найдите и выберите **Alain Charon**.

5. Щелкните **Выбрать**.

## <a name="view-all-groups"></a>Просмотр всех групп
Все группы своей организации можно просмотреть на странице **Группы — Все группы** портала Azure.

- Выберите **Azure Active Directory** > **Группы**.

    Откроется страница **Группы — Все группы** со списком активных групп.

    ![Страница "Группы — Все группы", где отображаются все существующие группы](media/active-directory-groups-view-azure-portal/groups-all-groups-blade-with-all-groups.png)

## <a name="search-for-the-group"></a>Поиск группы
На странице **Группы — Все группы** найдите группу **MDM policy - West**.

1. На странице **Группы — Все группы** введите строку _MDM_ в поле **Поиск**.

    Под полем **Поиск** отобразятся найденные результаты, включая группу _MDM policy - West_.

    ![Страница "Группы — Все группы" с заполненным полем поиска](media/active-directory-groups-view-azure-portal/search-for-specific-group.png)

3. Выберите группу **MDM policy - West**.

4. Просмотрите сведения о группе на странице **Обзор MDM policy - West**, в том числе проверьте число членов этой группы.

    ![Страница "Обзор MDM policy - West" с информацией о членах](media/active-directory-groups-view-azure-portal/group-overview-blade.png)

## <a name="view-group-members"></a>Просмотр членов группы
Итак, вы нашли нужную группу и можете просмотреть назначенных в нее членов.

- Выберите **Члены** в области **Управление** и просмотрите полный список членов, назначенных этой группе, включая _Alain Charon_.

    ![Список членов, назначенных в группу MDM policy - West](media/active-directory-groups-view-azure-portal/groups-all-members.png)

## <a name="clean-up-resources"></a>Очистка ресурсов
Эта группа используется в нескольких руководствах, которые собраны в разделе документации **Практические руководства**. Но если вы не намерены ее использовать, удалите ее вместе с назначенными членами с помощью следующей процедуры.

1. На странице **Группы — Все группы** найдите группу **MDM policy - West**.

2.  Выберите группу **MDM policy - West**.

    Откроется страница **Обзор MDM policy - West**.

3. Выберите команду **Удалить**.

    Это действие удаляет группы и всех связанных с ней членов.

    ![Страница "Обзор MDM policy - West" с выделенной ссылкой "Удалить"](media/active-directory-groups-view-azure-portal/group-overview-blade-delete.png)

    >[!Important]
    >Но удаляется не сам пользователь Ален Charon, а его членство в удаленной группе.

## <a name="next-steps"></a>Дальнейшие действия
Перейдите к следующей статье, которая описывает процесс привязки подписки к каталогу Azure AD.

> [!div class="nextstepaction"]
> [Привязка подписки Azure](active-directory-how-subscriptions-associated-directory.md)
