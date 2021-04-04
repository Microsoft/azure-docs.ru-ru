---
title: Руководство по Настройка Cornerstone OnDemand для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Cornerstone OnDemand.
services: active-directory
author: zhchia
writer: zhchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: 59c599167089d222324ed880c18e68d763f5e468
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94358462"
---
# <a name="tutorial-configure-cornerstone-ondemand-for-automatic-user-provisioning"></a>Руководство по Настройка Cornerstone OnDemand для автоматической подготовки пользователей

В этом руководстве показаны шаги, которые необходимо выполнить в Cornerstone OnDemand и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей или групп в Cornerstone OnDemand.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на основе службы подготовки пользователей Azure AD. Сведения о том, что делает эта служба, как она работает, а также часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и их отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Предварительные требования

Сценарий, описанный в этом руководстве, предполагает, что у вас уже имеется:

* Клиент Azure AD.
* Клиент Cornerstone OnDemand.
* Учетная запись пользователя в Cornerstone OnDemand с разрешениями администратора.

> [!NOTE]
> Интеграция подготовки Azure AD зависит от [веб-службы Cornerstone OnDemand](https://www.cornerstoneondemand.com/). Эта служба доступна для команд Cornerstone OnDemand.

## <a name="add-cornerstone-ondemand-from-the-azure-marketplace"></a>Добавление Cornerstone OnDemand из Azure Marketplace

Перед настройкой Cornerstone OnDemand для автоматической подготовки пользователей в Azure AD необходимо добавить Cornerstone OnDemand из Marketplace в список управляемых приложений SaaS.

Чтобы добавить Cornerstone OnDemand из Marketplace, выполните указанные ниже действия.

1. На [портале Azure](https://portal.azure.com) в области навигации слева выберите **Azure Active Directory**.

    ![Значок Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна выберите **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Cornerstone OnDemand** и выберите **Cornerstone OnDemand** на панели результатов. Нажмите **Добавить**, чтобы добавить приложение.

    ![Cornerstone OnDemand в списке результатов](common/search-new-app.png)

## <a name="assign-users-to-cornerstone-ondemand"></a>Назначение пользователей для Cornerstone OnDemand

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и группы, которые были назначены приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей решите, какие пользователи и (или) группы в Azure AD должны иметь доступ к Cornerstone OnDemand. Чтобы назначить этих пользователей или группы для Cornerstone OnDemand, следуйте инструкциям в разделе[Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-cornerstone-ondemand"></a>Важные рекомендации по назначению пользователей в Cornerstone OnDemand

* Рекомендуется назначить одного пользователя Azure AD для Cornerstone OnDemand для тестирования конфигурации автоматической подготовки пользователей. Дополнительных пользователей или группы можно будет назначить позже.

* При назначении пользователя в Cornerstone OnDemand в диалоговом окне назначения выберите действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configure-automatic-user-provisioning-to-cornerstone-ondemand"></a>Настройка автоматической подготовки пользователей для Cornerstone OnDemand

В этом разделе описывается процедура настройки службы подготовки Azure AD. Используйте ее для создания, обновления и отключения пользователей или групп в Cornerstone OnDemand в соответствии с назначениями пользователей или групп в Azure AD.

Чтобы настроить автоматическую подготовку пользователей для Cornerstone OnDemand в Azure AD, выполните указанные ниже действия.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения** > **Все приложения** > **Cornerstone OnDemand**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Cornerstone OnDemand**.

    ![Ссылка на Cornerstone OnDemand в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Подготовка Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningTab.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Автоматически**.

    ![Режим подготовки Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningCredentials.png)

5. В разделе **Учетные данные администратора** введите имя пользователя администратора, пароль администратора и домен своей учетной записи Cornerstone OnDemand.

    * В поле **Имя администратора** укажите домен или имя пользователя учетной записи администратора в клиенте Cornerstone OnDemand. Пример: contoso\admin.

    * В поле **Пароль администратора** введите пароль, соответствующий имени пользователя администратора.

    * В поле **Домен** введите URL-адрес веб-службы клиента Cornerstone OnDemand. Например, если служба расположена по адресу `https://ws-[corpname].csod.com/feed30/clientdataservice.asmx`, а доменом Contoso является `https://ws-contoso.csod.com/feed30/clientdataservice.asmx`. Дополнительные сведения о том, как получить URL-адрес веб-службы, приведены [в этом PDF-документе](https://help.csod.com/help/csod_0/Content/Resources/Documents/WebServices/CSOD_Web_Services_-_User-OU_Technical_Specification_v20160222.pdf).

6. После заполнения полей, показанных на шаге 5, выберите **Проверить подключение**, чтобы убедиться, что Azure AD может подключаться к Cornerstone OnDemand. Если установить подключение не удалось, убедитесь, что у учетной записи Cornerstone OnDemand есть разрешения администратора и попробуйте снова.

    ![Проверка подключения к Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/TestConnection.png)

7. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Уведомление по электронной почте в Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/EmailNotification.png)

8. Щелкните **Сохранить**.

9. В разделе **сопоставления** выберите **Synchronize Azure Active Directory Users to Cornerstone OnDemand** (Синхронизировать пользователей Azure Active Directory с Cornerstone OnDemand).

    ![Синхронизация Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserMapping.png)

10. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Cornerstone OnDemand. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Cornerstone OnDemand для операций обновления. Нажмите **Сохранить**, чтобы сохранить все изменения.

    ![Сопоставления атрибутов Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/UserMappingAttributes.png)

11. Чтобы настроить фильтры области, ознакомьтесь с инструкциями в статье [Подготовка приложений на основе атрибутов с использованием фильтров области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

12. Чтобы включить службу подготовки Azure AD для Cornerstone OnDemand, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/ProvisioningStatus.png)

13. Определите пользователей или группы, которые требуется подготовить для Cornerstone OnDemand. В разделе **Параметры** выберите нужные значения в разделе **Область**.

    ![Область Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/SyncScope.png)

14. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Кнопка "Сохранить" в Cornerstone OnDemand](./media/cornerstone-ondemand-provisioning-tutorial/Save.png)

После этого начнется начальная синхронизация пользователей или групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Они происходят примерно каждые 40 минут, пока выполняется служба подготовки Azure AD. 

В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчета о подготовке. В отчете представлены сведения обо всех действиях, выполненных службой подготовки Azure AD в отношении Cornerstone OnDemand.

Сведения о журналах подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя

Атрибут **Position** в Cornerstone OnDemand ожидает значение, соответствующее ролям на портале Cornerstone OnDemand. Список допустимых значений **Position** можно получить, выбрав **Edit User Record (Изменить запись пользователя) > Organization Structure (Структура организации) > Position (Позиция)** на портале Cornerstone OnDemand.

![Подготовка Cornerstone OnDemand — редактирование записи пользователя](./media/cornerstone-ondemand-provisioning-tutorial/UserEdit.png)

![Подготовка Cornerstone OnDemand — позиция](./media/cornerstone-ondemand-provisioning-tutorial/UserPosition.png)

![Подготовка Cornerstone OnDemand — список позиций](./media/cornerstone-ondemand-provisioning-tutorial/PostionId.png)

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Managing user account provisioning for enterprise apps in the Azure portal](../app-provisioning/configure-automatic-user-provisioning-portal.md) (Управление подготовкой учетных записей пользователей для корпоративных приложений на портале Azure)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_01.png
[2]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_02.png
[3]: ./media/cornerstone-ondemand-provisioning-tutorial/tutorial_general_03.png
