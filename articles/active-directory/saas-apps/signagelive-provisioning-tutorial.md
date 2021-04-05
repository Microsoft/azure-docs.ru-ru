---
title: Руководство по настройке Signagelive для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке Azure Active Directory для автоматической подготовки и отзыва учетных записей пользователей в Signagelive.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 07/23/2019
ms.author: Zhchia
ms.openlocfilehash: 10ad06041e8136b5661b1b1ff487cd4d3b0f5153
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "94358411"
---
# <a name="tutorial-configure-signagelive--for-automatic-user-provisioning"></a>Руководство по настройке Signagelive для автоматической подготовки пользователей

В этом учебнике описаны шаги, которые нужно выполнить в Signagelive и Azure Active Directory (Azure AD), чтобы настроить Azure AD для автоматической подготовки и отзыва пользователей и (или) групп в Signagelive.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* Клиент Azure AD.
* [клиент Signagelive](https://signagelive.com/pricing/);
* учетная запись пользователя в Signagelive с разрешениями администратора.

## <a name="assigning-users-to-signagelive"></a>Назначение пользователей в Signagelive   

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи и (или) группы в Azure AD должны иметь доступ к Signagelive. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Signagelive, следуя приведенным ниже указаниям.
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-signagelive"></a>Важные советы по назначению пользователей в Signagelive   

* Рекомендуется назначить одного пользователя Azure AD в Signagelive для тестирования конфигурации автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя в Signagelive в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="setup-signagelive--for-provisioning"></a>Настройка Signagelive для подготовки

Перед настройкой Signagelive для автоматической подготовки пользователей с помощью Azure AD необходимо включить подготовку SCIM в Signagelive.

  Обратитесь к команде [Signagelive](mailto:development@signagelive.com), чтобы получить секретный токен, необходимый для настройки подготовки SCIM.

## <a name="add-signagelive-from-the-gallery"></a>Добавление Signagelive из коллекции

Чтобы настроить Signagelive для автоматической подготовки пользователей в Azure AD, необходимо добавить Signagelive из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Signagelive из коллекции приложений Azure AD, сделайте следующее:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Signagelive**, выберите **Signagelive** на панели результатов и нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Signagelive в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-signagelive"></a>Настройка автоматической подготовки пользователей в Signagelive    

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Signagelive на основе их назначений в Azure AD.

> [!TIP]
>  Для Signagelive также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [учебнике по единому входу в Signagelive](Signagelive-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-signagelive--in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Signagelive, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Signagelive**.

    ![Ссылка на Signagelive в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе "Учетные данные администратора" введите значение ` https://samlapi.signagelive.com/scim/v2` в поле **URL-адрес клиента**. В поле **Секретный токен** введите **токен носителя**, предоставленный командой технической разработки. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Signagelive. Если установить подключение не удалось, убедитесь, что у учетной записи Signagelive есть разрешения администратора, и повторите попытку.
    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Signagelive** (Синхронизировать пользователей Azure Active Directory с Signagelive).

    ![Снимок экрана: раздел "Сопоставления" с выделенным параметром Synchronize Azure Active Directory Users to Signagelive (Синхронизировать пользователей Azure Active Directory с Signagelive).](media/signagelive-provisioning-tutorial/usermapping.png)

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Signagelive. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Signagelive для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Снимок экрана: раздел "Сопоставление атрибутов" с семью сопоставляемыми атрибутами](media/signagelive-provisioning-tutorial/userattribute.png)

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to Signagelive** (Синхронизировать группы Azure Active Directory с Signagelive).

    ![Снимок экрана: раздел "Сопоставления" с выделенным параметром Synchronize Azure Active Directory Groups to Signagelive (Синхронизировать группы Azure Active Directory с Signagelive).](media/signagelive-provisioning-tutorial/groupmapping.png)

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты групп, которые синхронизированы из Azure AD в Signagelive. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей групп в Signagelive для операций обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Снимок экрана: раздел "Сопоставление атрибутов" с тремя сопоставляемыми атрибутами](media/signagelive-provisioning-tutorial/groupattribute.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Signagelive, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в Signagelive, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Дополнительные сведения о том, сколько продлится подготовка пользователей и (или) групп, см. в разделе [Сколько времени занимает подготовка пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users). 

В разделе **Текущее состояние** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Signagelive. Дополнительные сведения см. в статье [Проверка состояния подготовки пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Сведения о чтении журналов подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
