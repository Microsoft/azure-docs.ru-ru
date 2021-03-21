---
title: Руководство по Настройка автоматической подготовки пользователей в Symantec Web Security Service (WSS) с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Symantec Web Security Service (WSS).
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
ms.openlocfilehash: d7e0db1b0bc1e7aef68ee06f3bdd5e5e0f83b73e
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94354722"
---
# <a name="tutorial-configure-symantec-web-security-service-wss-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Symantec Web Security Service (WSS)

В этом руководстве описаны действия, которые нужно выполнить в Symantec Web Security Service (WSS) и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку, а также отзыв пользователей и групп в AAD для Symantec Web Security Service (WSS).

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [арендатор Symantec Web Security Service (WSS)](https://www.websecurity.symantec.com/buy-renew?inid=brmenu_nav_brhome);
* учетная запись пользователя с правами администратора в Symantec Web Security Service (WSS).

## <a name="assigning-users-to-symantec-web-security-service-wss"></a>Назначение пользователей для Symantec Web Security Service (WSS)

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Symantec Web Security Service (WSS). Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Symantec Web Security Service (WSS) по приведенным ниже инструкциям.
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

##  <a name="important-tips-for-assigning-users-to-symantec-web-security-service-wss"></a>Важные замечания о назначении пользователей для Symantec Web Security Service (WSS)

* Рекомендуется назначить одного пользователя Azure AD для Symantec Web Security Service (WSS), чтобы проверить конфигурацию автоматической подготовки пользователей. Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для Symantec Web Security Service (WSS) в диалоговом окне назначения необходимо выбрать действительную роль для конкретного приложения (если это доступно). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="setup-symantec-web-security-service-wss-for-provisioning"></a>Настройка подготовки в Symantec Web Security Service (WSS)

Прежде, чем настраивать автоматическую подготовку пользователей в AAD для Symantec Web Security Service (WSS), необходимо включить подготовку SCIM для Symantec Web Security Service (WSS).

1. Войдите в [консоль администрирования Symantec Web Security Service](https://portal.threatpulse.com/login.jsp). Перейдите к разделу **Solutions (Решения)**  > **Service (Служба)** .

    ![Symantec Web Security Service (WSS)](media/symantec-web-security-service/service.png)

2. Последовательно выберите элементы **Account Maintenance (Обслуживание учетных записей)**  > **Integrations (Интеграции)**  > **New Integration (Новая интеграция)** .

    ![Symantec Web Security Service (WSS)](media/symantec-web-security-service/acount.png)

3.  Выберите элемент **Third-Party Users & Groups Sync** (Синхронизация сторонних пользователей и групп). 

    ![Снимок экрана с элементом Third-Party Users & Groups Sync (Синхронизация сторонних пользователей и групп)](media/symantec-web-security-service/third-party-users.png)

4.  Скопируйте значения **SCIM URL** (URL-адрес SCIM) и **Token** (Маркер). Эти значения вы введете в поля **URL-адрес клиента** и **Секретный маркер** соответственно на вкладке "Подготовка" для приложения Symantec Web Security Service (WSS) на портале Azure.

    ![Снимок экрана для диалогового окна New Integration (Новая интеграция), где отмечены текстовые поля SCIM URL (URL-адрес SCIM) и Token (Маркер).](media/symantec-web-security-service/scim.png)

## <a name="add-symantec-web-security-service-wss-from-the-gallery"></a>Добавление Symantec Web Security Service (WSS) из коллекции

Чтобы настроить в Symantec Web Security Service (WSS) автоматическую подготовку пользователей в Azure AD, необходимо добавить Symantec Web Security Service (WSS) из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Symantec Web Security Service (WSS) из коллекции приложений AAD, выполните следующие действия:**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Symantec Web Security Service** и выберите элемент **Symantec Web Security Service** на панели результатов. Затем нажмите кнопку **Добавить**, чтобы добавить это приложение.

    ![Служба Symantec Web Security Service (WSS) в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-symantec-web-security-service-wss"></a>Настройка автоматической подготовки пользователей в Symantec Web Security Service (WSS)

В этом разделе объясняется, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и групп в Symantec Web Security Service (WSS) на основе их назначений в Azure AD.

> [!TIP]
> Кроме того, вы можете настроить для Symantec Web Security Service (WSS) единый вход на основе SAML, используя инструкции из [руководства по единому входу для Symantec Web Security Service (WSS)](symantec-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

### <a name="to-configure-automatic-user-provisioning-for-symantec-web-security-service-wss-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в AAD для Symantec Web Security Service (WSS) сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Symantec Web Security Service**.

    ![Ссылка на Symantec Web Security Service (WSS) в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе "Учетные данные администратора" введите полученные ранее значения **SCIM URL** (URL-адрес SCIM) и **Token** (Маркер) в поля **URL-адрес клиента** и **Секретный токен** соответственно. Щелкните элемент **Проверить подключение** и убедитесь, что Azure AD может подключиться к Symantec Web Security Service. Если установить подключение не удалось, убедитесь, что у учетной записи Symantec Web Security Service (WSS) есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите параметр **Синхронизировать пользователей Azure Active Directory с Symantec Web Security Service (WSS)** .

    ![Снимок экрана для раздела "Сопоставления", где выделен параметр "Синхронизировать пользователей Azure Active Directory с Symantec Web Security Service (WSS)".](media/symantec-web-security-service/usermapping.png)

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Symantec Web Security Service (WSS). Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления учетных записей пользователей в Symantec Web Security Service (WSS) при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Снимок экрана для раздела "Сопоставление атрибутов", где отмечены 16 свойств.](media/symantec-web-security-service/userattribute.png)

10. В разделе **Сопоставления** выберите параметр **Синхронизировать группы Azure Active Directory с Symantec Web Security Service (WSS)** .

    ![Снимок экрана для раздела "Сопоставления", где выделен параметр "Синхронизировать группы Azure Active Directory с Symantec Web Security Service (WSS)".](media/symantec-web-security-service/groupmapping.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Symantec Web Security Service (WSS). Атрибуты, выбранные как свойства с меткой **Сопоставление**, используются для сопоставления групп в Symantec Web Security Service (WSS) при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Снимок экрана для раздела "Сопоставление атрибутов", где отмечены три свойства.](media/symantec-web-security-service/groupattribute.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Symantec Web Security Service (WSS), измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей или группы для подготовки в Symantec Web Security Service (WSS), выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Дополнительные сведения о том, сколько продлится подготовка пользователей и (или) групп, см. в разделе [Сколько времени занимает подготовка пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md#how-long-will-it-take-to-provision-users).

В разделе **Текущее состояние** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Symantec Web Security Service (WSS). Дополнительные сведения см. в статье [Проверка состояния подготовки пользователей](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md). Сведения о чтении журналов подготовки Azure AD см. в руководстве [по отчетам об автоматической подготовке учетных записей пользователей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
