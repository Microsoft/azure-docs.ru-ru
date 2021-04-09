---
title: Руководство по настройке Zscaler One для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Zscaler One.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/27/2019
ms.author: jeedes
ms.openlocfilehash: f6725045064b74079e00ca5bbe1d560f3b19f3ff
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97937135"
---
# <a name="tutorial-configure-zscaler-one-for-automatic-user-provisioning"></a>Руководство по настройке Zscaler One для автоматической подготовки пользователей

В этом учебнике показаны шаги, которые необходимо выполнить в Zscaler One и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей или групп в Zscaler One.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на основе службы подготовки пользователей Azure AD. Сведения о том, что делает эта служба, как она работает, а также часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и их отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).


## <a name="prerequisites"></a>Предварительные требования

Сценарий, описанный в этом руководстве, предполагает, что у вас уже имеется:

* Клиент Azure AD.
* клиент Zscaler One;
* учетная запись пользователя в Zscaler One с разрешениями администратора.

> [!NOTE]
> Интеграция подготовки Azure AD зависит от API SCIM Zscaler One. Этот API доступен разработчикам Zscaler One для учетных записей с корпоративным пакетом.

## <a name="add-zscaler-one-from-the-azure-marketplace"></a>Добавление Zscaler One из Azure Marketplace

Перед настройкой Zscaler One для автоматической подготовки пользователей в Azure AD добавьте Zscaler One из Azure Marketplace в список управляемых приложений SaaS.

Чтобы добавить Zscaler One из Azure Marketplace, сделайте следующее:

1. На [портале Azure](https://portal.azure.com) в области навигации слева выберите **Azure Active Directory**.

    ![Значок Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в верхней части диалогового окна выберите **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Zscaler One** и выберите **Zscaler One** на панели результатов. Нажмите **Добавить**, чтобы добавить приложение.

    ![Zscaler One в списке результатов](common/search-new-app.png)

## <a name="assign-users-to-zscaler-one"></a>Назначение пользователей в Zscaler One

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и группы, которые были назначены приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей решите, какие пользователи или группы в Azure AD должны иметь доступ к Zscaler One. Чтобы назначить этих пользователей или группы для Zscaler One, следуйте инструкциям в статье [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-zscaler-one"></a>Важные рекомендации по назначению пользователей в Zscaler One

* Мы рекомендуем назначить одного пользователя Azure AD в Zscaler One, чтобы проверить настройки автоматической подготовки пользователей. Дополнительных пользователей или группы можно будет назначить позже.

* При назначении пользователя в Zscaler One в диалоговом окне назначения выберите действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configure-automatic-user-provisioning-to-zscaler-one"></a>Настройка автоматической подготовки пользователей в Zscaler One

В этом разделе описывается процедура настройки службы подготовки Azure AD. Используйте ее для создания, обновления и отключения пользователей или групп в Zscaler One в соответствии с назначениями пользователей или групп в Azure AD.

> [!TIP]
> Для Zscaler One также можно включить единый вход на основе SAML. Выполните инструкции из статьи о [настройке единого входа в Zscaler One](zscaler-One-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

> [!NOTE]
> При подготовке или отмене подготовки пользователей и групп рекомендуется периодически перезапускать подготовку, чтобы членство в группах правильно обновлялось. При перезапуске служба повторно оценит все группы и обновит членство.  

### <a name="configure-automatic-user-provisioning-for-zscaler-one-in-azure-ad"></a>Настройка автоматической подготовки пользователей для Zscaler One в Azure AD

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения** > **Все приложения** > **Zscaler One**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Zscaler One**.

    ![Ссылка на Zscaler One в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Подготовка Zscaler One](./media/zscaler-one-provisioning-tutorial/provisioning-tab.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Режим подготовки Zscaler One](./media/zscaler-one-provisioning-tutorial/provisioning-credentials.png)

5. В разделе **Учетные данные администратора** заполните поля **URL-адрес клиента** и **Секретный токен** параметрами учетной записи Zscaler One, как описано на шаге 6.

6. Чтобы получить URL-адрес клиента и секретный токен, на пользовательском интерфейсе портала Zscaler One перейдите в меню **Administration (Администрирование)**  > **Authentication Settings (Параметры проверки подлинности)** . В разделе **Authentication Type** (Тип проверки подлинности) выберите **SAML**.

    ![Параметры проверки подлинности в Zscaler One](./media/zscaler-one-provisioning-tutorial/secret-token-1.png)

    a. Выберите **Configure SAML** (Настроить SAML), чтобы открыть параметры **настроек SAML**.

    ![Окно настроек SAML в Zscaler One](./media/zscaler-one-provisioning-tutorial/secret-token-2.png)

    b. Выберите **Enable SCIM-Based Provisioning** (Включить подготовку на основе SCIM), чтобы получить параметры для **Base URL** (Базовый URL-адрес) и **Bearer Token** (Токен носителя). После этого сохраните параметры. Скопируйте параметр **Base URL** (Базовый URL-адрес) в поле **URL-адрес клиента** на портале Azure. Скопируйте параметр **Bearer Token** (Токен носителя) в поле **Секретный токен** на портале Azure.

7. После заполнения полей, показанных на шаге 5, выберите **Проверить подключение**, чтобы убедиться, что Azure AD может подключаться к Zscaler One. Если установить подключение не удалось, убедитесь, что у учетной записи Zscaler One есть разрешения администратора, и повторите попытку.

    ![Проверка подключения к Zscaler One](./media/zscaler-one-provisioning-tutorial/test-connection.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление Zscaler One](./media/zscaler-one-provisioning-tutorial/notification.png)

9. Нажмите кнопку **Сохранить**.

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Zscaler One** (Синхронизировать пользователей Azure Active Directory в Zscaler One).

    ![Синхронизация пользователей Zscaler One](./media/zscaler-one-provisioning-tutorial/user-mappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователя, синхронизированные в Azure AD и Zscaler One. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Zscaler One для операций обновления. Нажмите **Сохранить**, чтобы сохранить все изменения.

    ![Сопоставляемые атрибуты пользователей Zscaler One](./media/zscaler-one-provisioning-tutorial/user-attribute-mappings.png)

12. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to Zscaler One** (Синхронизировать группы Azure Active Directory в Zscaler One).

    ![Синхронизация групп Zscaler One](./media/zscaler-one-provisioning-tutorial/group-mappings.png)

13. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, синхронизированные из Azure AD в Zscaler One. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в Zscaler One для операций обновления. Нажмите **Сохранить**, чтобы сохранить все изменения.

    ![Сопоставляемые атрибуты групп Zscaler One](./media/zscaler-one-provisioning-tutorial/group-attribute-mappings.png)

14. Чтобы настроить фильтры области, ознакомьтесь с инструкциями в статье [Подготовка приложений на основе атрибутов с использованием фильтров области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Чтобы включить службу подготовки Azure AD для Zscaler One, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки Zscaler One](./media/zscaler-one-provisioning-tutorial/provisioning-status.png)

16. Определите пользователей или группы, которые требуется подготовить для Zscaler One. В разделе **Параметры** выберите нужные значения в разделе **Область**.

    ![Область Zscaler One](./media/zscaler-one-provisioning-tutorial/scoping.png)

17. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Кнопка "Сохранить" в Zscaler One](./media/zscaler-one-provisioning-tutorial/save-provisioning.png)

После этого начнется начальная синхронизация пользователей или групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Они происходят примерно каждые 40 минут, пока выполняется служба подготовки Azure AD. 

В разделе **Сведения о синхронизации** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчета о подготовке. В отчете представлены сведения обо всех действиях, выполненных службой подготовки Azure AD в отношении Zscaler One.

Сведения о журналах подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Managing user account provisioning for enterprise apps in the Azure portal](../app-provisioning/configure-automatic-user-provisioning-portal.md) (Управление подготовкой учетных записей пользователей для корпоративных приложений на портале Azure)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zscaler-one-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-one-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-one-provisioning-tutorial/tutorial-general-03.png