---
title: Руководство по Настройка автоматической подготовки пользователей в Meta Networks Connector с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Meta Networks Connector.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/01/2019
ms.author: Zhchia
ms.openlocfilehash: b6a8f192cd26639431cc9fcb6b43e1bc5e8e2843
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96353635"
---
# <a name="tutorial-configure-meta-networks-connector-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Meta Networks Connector

В этом руководстве описаны шаги, которые нужно выполнить в Meta Networks Connector и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку, а также отзыв пользователей и групп в AAD для Meta Networks Connector.

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Meta Networks Connector](https://www.metanetworks.com/);
* учетная запись пользователя Meta Networks Connector с разрешениями администратора.

## <a name="assigning-users-to-meta-networks-connector"></a>Назначение пользователей для Meta Networks Connector

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Meta Networks Connector. Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Meta Networks Connector по приведенным ниже инструкциям.
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-meta-networks-connector"></a>Важные замечания о назначении пользователей для Meta Networks Connector

* Рекомендуется назначить одного пользователя Azure AD для Meta Networks Connector, чтобы проверить конфигурацию автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для Meta Networks Connector в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если это доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="setup-meta-networks-connector-for-provisioning"></a>Настройка подготовки в Meta Networks Connector

1. Войдите в [консоль администрирования Meta Networks Connector](https://login.metanetworks.com/login/), указав название организации. Откройте раздел **Administration (Администрирование) > API Keys (Ключи API)** .

    ![Консоль администрирования Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/apikey.png)

2.  Щелкните значок "плюс" в верхней правой части экрана, чтобы создать новый **ключ API**.

    ![Meta Networks Connector — значок "плюс"](media/meta-networks-connector-provisioning-tutorial/plusicon.png)

3.  Задайте значения **API Key Name** (Имя ключа API) и **API Key Description** (Описание ключа API).

    :::image type="content" source="media/meta-networks-connector-provisioning-tutorial/keyname.png" alt-text="Снимок экрана для консоли администрирования Meta Networks Connector, где выделены значения API Key Name (Имя ключа API) и API Key Description (Описание ключа API) для AAD и ключа API." border="false":::

4.  Включите права **Write** (Запись) для элементов **Groups** (Группы) и **Users** (Пользователи).

    ![Meta Networks Connector — права](media/meta-networks-connector-provisioning-tutorial/privileges.png)

5.  Щелкните **Добавить**. Скопируйте значение **SECRET** (Секрет) и сохраните его, так как больше вы не сможете открыть его для просмотра. Его нужно будет ввести в поле "Секретный токен" на вкладке "Подготовка" для приложения Meta Networks Connector на портале Azure.

    :::image type="content" source="media/meta-networks-connector-provisioning-tutorial/token.png" alt-text="Снимок экрана: окно с сообщением о добавлении ключа API. Выделено поле Secret (Секрет), в котором содержится не поддающееся определению значение." border="false":::

6.  Добавьте поставщик удостоверений, открыв раздел **Administration (Администрирование) > Settings (Параметры) > IdP (Поставщик удостоверений) > Create New (Создать новый)** .

    ![Meta Networks Connector — добавление поставщика удостоверений](media/meta-networks-connector-provisioning-tutorial/newidp.png)

7.  На странице **IdP Configuration** (Конфигурация поставщика удостоверений) вы можете присвоить этой конфигурации имя (**Name**) и выбрать значок (**Icon**).

    ![Meta Networks Connector — имя поставщика удостоверений](media/meta-networks-connector-provisioning-tutorial/idpname.png)

    ![Meta Networks Connector — значок поставщика удостоверений](media/meta-networks-connector-provisioning-tutorial/icon.png)

8.  В разделе **Configure SCIM** (Настройка SCIM) выберите имя ключа API, который вы создали на предыдущих шагах. Щелкните **Save**(Сохранить).

    ![Meta Networks Connector — Configure SCIM (Настройка SCIM)](media/meta-networks-connector-provisioning-tutorial/configure.png)

9.  Откройте вкладку **Administration (Администрирование) > Settings (Параметры) > IdP (Поставщик удостоверений)** . Щелкните имя конфигурации поставщика удостоверений, которую вы создали на предыдущих шагах, чтобы отобразилось значение **IdP ID** (Идентификатор поставщика удостоверений). Этот **идентификатор** добавляется в конец значения **URL-адреса клиента** при заполнении поля **URL-адрес клиента** на вкладке "Подготовка" для приложения Meta Networks Connector на портале Azure.

    ![Meta Networks Connector — идентификатор поставщика удостоверений](media/meta-networks-connector-provisioning-tutorial/idpid.png)

## <a name="add-meta-networks-connector-from-the-gallery"></a>Добавление Meta Networks Connector из коллекции

Прежде чем настроить в Meta Networks Connector автоматическую подготовку пользователей в Azure AD, необходимо добавить Meta Networks Connector из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Meta Networks Connector из коллекции приложений AAD, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Meta Networks Connector** и выберите элемент **Meta Networks Connector** на панели результатов. Затем нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Meta Networks Connector в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-meta-networks-connector"></a>Настройке автоматической подготовки пользователей в Meta Networks Connector 

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Meta Networks Connector на основе их назначений в Azure AD.

> [!TIP]
> Для Meta Networks Connector также можно включить единый вход на основе SAML. Для этого следуйте инструкциям, приведенным в [руководстве по единому входу в Meta Networks Connector](./metanetworksconnector-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-meta-networks-connector-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в AAD для Meta Networks Connector сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Meta Networks Connector**.

    ![Ссылка на Meta Networks Connector в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите значение `https://api.metanetworks.com/v1/scim/<IdP ID>` в поле **URL-адрес клиента**. Введите полученное ранее значение **токена проверки подлинности SCIM** в поле **Секретный токен**. Щелкните элемент **Проверить подключение** и убедитесь, что Azure AD может подключиться к Meta Networks Connector. Если установить подключение не удалось, убедитесь, что у учетной записи Meta Networks Connector есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите параметр **Синхронизировать пользователей Azure Active Directory с Meta Networks Connector**.

    ![Сопоставления пользователей Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Meta Networks Connector. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в Meta Networks Connector при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователя Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/userattributes.png)

10. В разделе **Сопоставления** выберите параметр **Синхронизировать группы Azure Active Directory с Meta Networks Connector**.

    ![Сопоставления групп Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/groupmappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Meta Networks Connector. Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления групп в Meta Networks Connector при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты групп Meta Networks Connector](media/meta-networks-connector-provisioning-tutorial/groupattributes.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Meta Networks Connector, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей или группы для подготовки в Meta Networks Connector, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Текущее состояние** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Meta Networks Connector.

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)