---
title: Руководство по настройке Zscaler Three для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Zscaler Three.
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
ms.openlocfilehash: c16c02a870edb1a777b63da6fea57fc02136d643
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97936606"
---
# <a name="tutorial-configure-zscaler-three-for-automatic-user-provisioning"></a>Руководство по настройке Zscaler Three для автоматической подготовки пользователей

Узнайте, как настроить Azure Active Directory (Azure AD) для автоматической подготовки и отзыва учетных записей пользователей и (или) групп в Zscaler Three.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на основе службы подготовки пользователей Azure AD. Подробные сведения о том, какие функции выполняет эта служба и каков принцип ее работы, а также ответы на часто задаваемые вопросы см. в статье об [автоматизации подготовки и отзыва пользователей в приложениях SaaS с помощью Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством требуется следующее:

* Клиент Azure AD.
* Клиент Zscaler Three.
* Учетная запись пользователя в Zscaler Three с разрешениями администратора.

> [!NOTE]
> Интеграция подготовки Azure AD зависит от API Zscaler ZSCloud SCIM, доступного для корпоративных учетных записей.

## <a name="adding-zscaler-three-from-the-gallery"></a>Добавление Zscaler Three из коллекции

Перед настройкой Zscaler Three для автоматической подготовки пользователей в Azure AD необходимо добавить Zscaler Three из коллекции приложений Azure AD в список управляемых приложений SaaS.

На [портале Azure](https://portal.azure.com) в области слева щелкните **Azure Active Directory**.

![Выберите Azure Active Directory.](common/select-azuread.png)

Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

![корпоративные приложения.](common/enterprise-applications.png)

Чтобы добавить новое приложение, выберите **Новое приложение** в верхней части окна.

![Выбор элемента "Новое приложение"](common/add-new-app.png)

В поле поиска введите **Zscaler Three**. Выберите **Zscaler Three** в результатах поиска и щелкните **Добавить**.

![Список результатов](common/search-new-app.png)

## <a name="assign-users-to-zscaler-three"></a>Назначение пользователей в Zscaler Three

Пользователям Azure AD нужно предоставить доступ к выбранным приложениям, чтобы они могли использовать их. В контексте автоматической подготовки синхронизируются только те пользователи и группы, которые были назначены приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к Zscaler Three. После этого можно назначить этих пользователей и группы в Zscaler Three, следуя инструкциям из статьи о [назначении пользователей или групп корпоративному приложению](../manage-apps/assign-user-or-group-access-portal.md).

### <a name="important-tips-for-assigning-users-to-zscaler-three"></a>Важные рекомендации по назначению пользователей в Zscaler Three

* Рекомендуется сначала назначить одного пользователя Azure AD в Zscaler Three, чтобы проверить настройки автоматической подготовки пользователей. Позже можно назначить других пользователей и группы.

* При назначении пользователя в Zscaler Three необходимо выбрать действительную роль для конкретного приложения (если она доступна) в диалоговом окне назначения. Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-automatic-user-provisioning"></a>Настройка автоматической подготовки пользователей

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в Zscaler Three на основе их назначений в Azure AD.

> [!TIP]
> Для Zscaler Three можно также включить единый вход на основе SAML. Для этого выполните инструкции из статьи о [настройке единого входа в Zscaler Three](zscaler-three-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки, хотя обе функции дополняют друг друга.

> [!NOTE]
> При подготовке или отмене подготовки пользователей и групп рекомендуется периодически перезапускать подготовку, чтобы членство в группах правильно обновлялось. При перезапуске служба повторно оценит все группы и обновит членство. 

1. На [портале Azure](https://portal.azure.com) выберите **Корпоративные приложения** > **Все приложения** > **Zscaler Three**.

    ![корпоративные приложения.](common/enterprise-applications.png)

2. В списке приложений выберите **Zscaler Three**.

    ![Список приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Подготовка Zscaler Three](./media/zscaler-three-provisioning-tutorial/provisioning-tab.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Автоматически**.

    ![Установка режима подготовки к работе](./media/zscaler-three-provisioning-tutorial/provisioning-credentials.png)

5. В разделе **Учетные данные администратора** введите **URL-адрес клиента** и **секретный токен** учетной записи Zscaler Three, как описано в следующем шаге.

6. Чтобы получить **URL-адрес клиента** и **секретный токен**, на портале Zscaler Three перейдите в меню **Administration** (Администрирование)  > **Authentication Settings** (Настройки аутентификации) и выберите **SAML** в разделе **Authentication Type** (Тип аутентификации).

    ![Настройки аутентификации в Zscaler Three](./media/zscaler-three-provisioning-tutorial/secret-token-1.png)

    Выберите **Configure SAML** (Настроить SAML), чтобы открыть окно **настроек SAML**.

    ![Окно настроек SAML](./media/zscaler-three-provisioning-tutorial/secret-token-2.png)

    Выберите **Enable SCIM-Based Provisioning** (Включить подготовку на основе SCIM), скопируйте **базовый URL-адрес** и **токен носителя** и сохраните настройки. На портале Azure вставьте **базовый URL-адрес** в поле **URL-адрес клиента**, а **токен носителя** — в поле **Секретный токен**.

7. После ввода значений в полях **URL-адрес клиента** и **Секретный токен** выберите **Проверить подключение**, чтобы убедиться в том, что Azure AD может подключаться к Zscaler Three. Если установить подключение не удалось, убедитесь, что у учетной записи Zscaler Three есть разрешения администратора, и повторите попытку.

    ![Проверка подключения](./media/zscaler-three-provisioning-tutorial/test-connection.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки. Выберите **Отправить уведомление по электронной почте при сбое**.

    ![Настройка уведомлений, отправляемых по электронной почте](./media/zscaler-three-provisioning-tutorial/notification.png)

9. Щелкните **Сохранить**.

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Zscaler Three** (Синхронизировать пользователей Azure Active Directory с Zscaler Three).

    ![Синхронизация пользователей Azure AD](./media/zscaler-three-provisioning-tutorial/user-mappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы в Azure AD и Zscaler Three. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Zscaler Three для операций обновления. Чтобы зафиксировать изменения, щелкните **Сохранить**.

    ![Снимок экрана: раздел "Сопоставление атрибутов" с семью сопоставляемыми атрибутами](./media/zscaler-three-provisioning-tutorial/user-attribute-mappings.png)

12. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to Zscaler Three** (Синхронизировать группы Azure Active Directory с Zscaler Three).

    ![Синхронизация групп Azure AD](./media/zscaler-three-provisioning-tutorial/group-mappings.png)

13. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизированы в Azure AD и Zscaler Three. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в Zscaler Three для операций обновления. Чтобы зафиксировать изменения, щелкните **Сохранить**.

    ![Снимок экрана: раздел "Сопоставление атрибутов" с тремя сопоставляемыми атрибутами](./media/zscaler-three-provisioning-tutorial/group-attribute-mappings.png)

14. Чтобы настроить фильтры области, ознакомьтесь с инструкциями в статье [Подготовка приложений на основе атрибутов с использованием фильтров области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Чтобы включить службу подготовки Azure AD для Zscaler Three, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![ProvisioningStatus](./media/zscaler-three-provisioning-tutorial/provisioning-status.png)

16. Определите пользователей или группы для подготовки в Zscaler Three, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Значения области](./media/zscaler-three-provisioning-tutorial/scoping.png)

17. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Нажатие кнопки "Сохранить"](./media/zscaler-three-provisioning-tutorial/save-provisioning.png)

Эта операция запускает начальную синхронизацию всех пользователей и групп, указанных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если запущена служба подготовки Azure AD, синхронизация выполняется примерно каждые 40 минут. Вы можете отслеживать ход выполнения в разделе **Сведения о синхронизации**. Также доступны ссылки для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Zscaler Three.

Сведения о журналах подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

<!--Image references-->
[1]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-01.png
[2]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-02.png
[3]: ./media/zscaler-three-provisioning-tutorial/tutorial-general-03.png