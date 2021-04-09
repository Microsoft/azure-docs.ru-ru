---
title: Руководство по настройке Oracle Fusion ERP для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Oracle Fusion ERP.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/26/2019
ms.author: Zhchia
ms.openlocfilehash: da6e1a8ba31f8f4991bde4803191598a015a68b3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94358445"
---
# <a name="tutorial-configure-oracle-fusion-erp-for-automatic-user-provisioning"></a>Руководство по настройке Oracle Fusion ERP для автоматической подготовке пользователей

В этом руководстве описаны шаги, которые нужно выполнить в Oracle Fusion ERP и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и (или) групп в Azure AD для Oracle Fusion ERP.

> [!NOTE]
>  В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель доступен в режиме предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Oracle Fusion ERP](https://www.oracle.com/applications/erp/);
* учетная запись пользователя Oracle Fusion ERP с разрешениями администратора.

## <a name="assign-users-to-oracle-fusion-erp"></a>Назначение пользователей для Oracle Fusion ERP 
В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется понятие "назначение". В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к Oracle Fusion ERP. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Oracle Fusion ERP по приведенным ниже инструкциям.
 
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md) 

 ## <a name="important-tips-for-assigning-users-to-oracle-fusion-erp"></a>Важные замечания о назначении пользователей для Oracle Fusion ERP 

 * Рекомендуется назначить одного пользователя Azure AD для Oracle Fusion ERP, чтобы проверить конфигурацию автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для Oracle Fusion ERP в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью Доступ по умолчанию исключаются из подготовки.

## <a name="set-up-oracle-fusion-erp-for-provisioning"></a>Настройка подготовки в Oracle Fusion ERP

Прежде чем настраивать автоматическую подготовку пользователей в Azure AD для Oracle Fusion ERP, необходимо включить подготовку SCIM для Oracle Fusion ERP.

1. Войдите в [консоль администрирования Oracle Fusion ERP](https://cloud.oracle.com/sign-in).

2. Щелкните элемент Navigator (Навигация) в левом верхнем углу. В разделе **Tools** (Средства) выберите **Security Console** (Консоль безопасности).

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/login.png" alt-text="Снимок экрана со страницей Navigator (Навигация) в консоли администрирования Oracle Fusion ERP. На ней выделены элементы Tools (Средства) и Security console (Консоль безопасности)." border="false":::

3. Перейдите в раздел **Users** (Пользователи).
    
    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/user.png" alt-text="Снимок экрана: панель в консоли администрирования Oracle Fusion ERP. На ней выделен элемент Users (Пользователи)." border="false":::

4. Сохраните имя пользователя и пароль для учетной записи администратора, с которой вы будете входить в консоль администрирования Oracle Fusion ERP. Эти значения нужно ввести в поля **Имя администратора** и **Пароль** на вкладке "Подготовка" для приложения Oracle Fusion ERP на портале Azure.

## <a name="add-oracle-fusion-erp-from-the-gallery"></a>Добавление Oracle Fusion ERP из коллекции

Чтобы настроить автоматическую подготовку пользователей для Oracle Fusion ERP с помощью Azure AD, нужно добавить Oracle Fusion ERP из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Oracle Fusion ERP из коллекции приложений Azure AD, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Oracle Fusion ERP** и выберите **Oracle Fusion ERP** на панели результатов.

    ![Oracle Fusion ERP в списке результатов](common/search-new-app.png)

 ## <a name="configure-automatic-user-provisioning-to-oracle-fusion-erp"></a>Настройка автоматической подготовки пользователей в Oracle Fusion ERP 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Oracle Fusion ERP на основе их назначений в Azure AD.

> [!TIP]
> Вы также можете настроить для Oracle Fusion ERP единый вход на основе SAML, используя инструкции из [руководства по единому входу для Oracle Fusion ERP](oracle-fusion-erp-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

> [!NOTE]
> Дополнительные сведения о конечной точке SCIM для Oracle Fusion ERP см. в статье [REST API for Common Features in Oracle Applications Cloud](https://docs.oracle.com/en/cloud/saas/applications-common/18b/farca/index.html) (REST API для основных функций облака приложений Oracle).

### <a name="to-configure-automatic-user-provisioning-for-fuze-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для Fuze выполните следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Oracle Fusion ERP**.

    ![Ссылка на Oracle Fusion ERP в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://ejlv.fa.em2.oraclecloud.com/hcmRestApi/scim/` в поле **URL-адрес клиента**. Введите полученные ранее имя администратора и пароль в поля **Имя администратора** и **Пароль**. Щелкните **Проверить подключение**, чтобы проверить связь между Azure AD и Oracle Fusion ERP. 

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/admin.png" alt-text="Снимок экрана: раздел &quot;Учетные данные администратора&quot;. В нем отображаются кнопка &quot;Проверить подключение&quot; и поля &quot;URL-адрес клиента&quot;, &quot;Имя администратора&quot; и &quot;Пароль&quot;." border="false":::

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Oracle Fusion ERP** (Синхронизировать пользователей Azure Active Directory с Oracle Fusion ERP).

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/user-mapping.png" alt-text="Снимок экрана: раздел &quot;Сопоставления&quot;. Под элементом &quot;Имя&quot; выделен элемент для синхронизации пользователей Azure Active Directory с Oracle Fusion ERP" border="false":::.

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Oracle Fusion ERP. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в Oracle Fusion ERP при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    :::image type="content" source="media/oracle-fusion-erp-provisioning-tutorial/user-attribute.png" alt-text="Снимок экрана: страница &quot;Сопоставление атрибутов&quot;. В таблице перечислены атрибуты Azure Active Directory и Oracle Fusion ERP с указанием приоритета сопоставления." border="false":::

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to Oracle Fusion ERP** (Синхронизировать группы Azure Active Directory с Oracle Fusion ERP).

    ![Сопоставления групп для Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/groupmappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Oracle Fusion ERP. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления групп в Oracle Fusion ERP при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты групп для Oracle Fusion ERP](media/oracle-fusion-erp-provisioning-tutorial/groupattributes.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Oracle Fusion ERP, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей и (или) группы для подготовки в Oracle Fusion ERP, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

    После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Oracle Fusion ERP.

    Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

* Oracle Fusion ERP поддерживает только базовую проверку подлинности для конечной точки SCIM.
* Oracle Fusion ERP не поддерживает подготовку групп.
* Роли в Oracle Fusion ERP сопоставляются с группами в Azure AD. Чтобы назначить пользователям Oracle Fusion ERP роли из Azure AD, вам придется назначить пользователей в соответствующие группы Azure AD, имена которых совпадают с ролями в Oracle Fusion ERP.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
