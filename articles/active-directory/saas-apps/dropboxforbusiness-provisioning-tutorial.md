---
title: Руководство по настройке автоматической подготовки пользователей в Dropbox for Business с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Dropbox for Business.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 05/20/2019
ms.author: jeedes
ms.openlocfilehash: 04d17e17ef11696efd52f04ea83639f2a9b81fea
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96938772"
---
# <a name="tutorial-configure-dropbox-for-business-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Dropbox for Business

В этом руководстве описаны шаги, которые нужно выполнить в Dropbox for Business и Azure Active Directory (Azure AD), чтобы в Azure AD настроить автоматическую подготовку и отзыв пользователей и (или) групп для Dropbox for Business.

> [!IMPORTANT]
> С 01.04.2021 г. корпорация Майкрософт и Dropbox начнут выводить из эксплуатации старую интеграцию с Dropbox. Чтобы избежать перебоев в работе служб, мы рекомендуем перейти на новую интеграцию Dropbox, которая поддерживает группы. Чтобы перейти на новую интеграцию Dropbox, добавьте и настройте в клиенте Azure AD новый экземпляр Dropbox для подготовки, выполнив приведенные ниже действия. После настройки новой интеграции Dropbox отключите подготовку для старой интеграции Dropbox, чтобы избежать конфликтов при подготовке. Более подробные инструкции по переходу на новую интеграцию с Dropbox см. в статье [Update to the newest Dropbox for Business application using Azure AD](https://help.dropbox.com/installs-integrations/third-party/update-dropbox-azure-ad-connector) (Обновление до новейшего приложения Dropbox for Business с помощью Azure AD).

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Dropbox for Business](https://www.dropbox.com/business/pricing);
* учетная запись пользователя в Dropbox for Business с разрешениями администратора.

## <a name="add-dropbox-for-business-from-the-gallery"></a>Добавление Dropbox for Business из коллекции

Перед настройкой Dropbox for Business для автоматической подготовки пользователей в Azure AD необходимо добавить Dropbox for Business из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Dropbox for Business из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Dropbox for Business**, выберите **Dropbox for Business** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Dropbox for Business в списке результатов](common/search-new-app.png)

## <a name="assigning-users-to-dropbox-for-business"></a>Назначение пользователей в Dropbox for Business

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы из Azure AD должны иметь доступ к Dropbox for Business. Решив этот вопрос, назначьте выбранных пользователей и (или) группы приложению Dropbox for Business, следуя инструкциям ниже.

* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

### <a name="important-tips-for-assigning-users-to-dropbox-for-business"></a>Важные рекомендации по назначению пользователей для приложения Dropbox for Business

* Рекомендуется назначить одного пользователя Azure AD в Dropbox for Business для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Dropbox for Business необходимо выбрать в диалоговом окне назначения действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="configuring-automatic-user-provisioning-to-dropbox-for-business"></a>Настройка автоматической подготовки пользователей в Dropbox for Business 

В этом разделе описывается, как настроить в службе подготовки Azure AD создание, обновление и отключение пользователей и (или) групп в Dropbox for Business на основе назначений пользователей и (или) групп в Azure AD.

> [!TIP]
> Для Dropbox for Business также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [руководстве по единому входу для Dropbox for Business](dropboxforbusiness-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-dropbox-for-business-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Dropbox for Business, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Dropbox for Business**.

    ![Ссылка на Dropbox for Business в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** щелкните **Авторизовать**. Откроется диалоговое окно входа в Dropbox for Business в новом окне браузера.

    ![Подготовка ](common/provisioning-oauth.png)

6. В диалоговом окне **Вход в Dropbox for Business для установки связи с Azure AD** войдите в клиент Dropbox for Business и подтвердите личность.

    ![Вход в Dropbox for Business](media/dropboxforbusiness-provisioning-tutorial/dropbox01.png)

7. Завершив шаги 5 и 6, щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к приложению Dropbox for Business. Если установить подключение не удалось, убедитесь, что у учетной записи Dropbox for Business есть разрешения администратора, и повторите попытку.

    ![Токен](common/provisioning-testconnection-oauth.png)

8. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

9. Выберите команду **Сохранить**.

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Dropbox** (Синхронизировать пользователей Azure Active Directory с Dropbox).

    ![Сопоставления пользователей Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-user-mapping.png)

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты пользователей, синхронизированные из Azure AD в Dropbox. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Dropbox при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователей в Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-user-attributes.png)

12. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to Dropbox** (Синхронизировать группы Azure Active Directory с Dropbox).

    ![Сопоставления групп в Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-group-mapping.png)

13. В разделе **Сопоставление атрибутов** просмотрите атрибуты группы, которые синхронизированы из Azure AD в Dropbox. Атрибуты, выбранные как свойства в разделе **Сопоставления**, используются для сопоставления групп в Dropbox при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты группы Dropbox](media/dropboxforbusiness-provisioning-tutorial/dropbox-group-attributes.png)

14. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

15. Чтобы включить службу подготовки Azure AD для Dropbox, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

16. Определите пользователей и (или) группы для подготовки в Dropbox, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

17. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Сведения о синхронизации** можно отслеживать ход выполнения, а по дополнительным ссылкам просмотреть отчет обо всех действиях подготовки, выполненных службой подготовки Azure AD в отношении Dropbox.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="connector-limitations"></a>Ограничения соединителя
 
* Dropbox не поддерживает приостановку приглашенных пользователей. В случае приостановки приглашенный пользователь удаляется.

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)

