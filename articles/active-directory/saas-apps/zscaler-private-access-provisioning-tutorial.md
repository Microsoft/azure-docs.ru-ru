---
title: Руководство по настройке автоматической подготовки пользователей в Zscaler Private Access (ZPA) с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о том, как настроить в Azure Active Directory автоматическую подготовку и отзыв учетных записей пользователей для Zscaler Private Access (ZPA).
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 10/07/2019
ms.author: Zhchia
ms.openlocfilehash: 14708ddcc5c0e06ee58f5e9db5945c4e9f1a1d08
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97937150"
---
# <a name="tutorial-configure-zscaler-private-access-zpa-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Zscaler Private Access (ZPA)

В этом руководстве описаны шаги, которые нужно выполнить в Zscaler Private Access (ZPA) и Azure Active Directory (Azure AD), чтобы настроить автоматическую подготовку и отзыв пользователей и групп в Azure AD для Zscaler Private Access (ZPA).

> [!NOTE]
> В этом руководстве рассматривается соединитель, созданный на базе службы подготовки пользователей Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md).
>
> Сейчас этот соединитель предоставляется в общедоступной предварительной версии. Дополнительные сведения об общих условиях использования продуктов в предварительной версии см. в документе [Дополнительные условия использования Предварительных версий Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* клиент Azure AD;
* [клиент Zscaler Private Access (ZPA);](https://www.zscaler.com/pricing-and-plans#contact-us)
* учетная запись пользователя Zscaler Private Access (ZPA) с разрешениями администратора.

## <a name="assigning-users-to-zscaler-private-access-zpa"></a>Назначение пользователей для Zscaler Private Access (ZPA)

В Azure Active Directory для определения того, какие пользователи должны получать доступ к выбранным приложениям, используется концепция, называемая *назначением*. В контексте автоматической подготовки синхронизируются только те пользователи и (или) группы, которые были назначены конкретному приложению в Azure AD.

Перед настройкой и включением автоматической подготовки пользователей нужно решить, какие пользователи или группы в Azure AD должны иметь доступ к Zscaler Private Access (ZPA). Когда этот вопрос будет решен, этих пользователей и (или) группы можно будет назначить приложению Zscaler Private Access (ZPA), следуя приведенным ниже инструкциям.
* [Назначение корпоративному приложению пользователя или группы](../manage-apps/assign-user-or-group-access-portal.md)

## <a name="important-tips-for-assigning-users-to-zscaler-private-access-zpa"></a>Важные замечания о назначении пользователей для Zscaler Private Access (ZPA)

* Чтобы проверить конфигурацию автоматической подготовки пользователей, рекомендуем назначить одного пользователя Azure AD для Zscaler Private Access (ZPA). Дополнительные пользователи и/или группы можно назначить позднее.

* При назначении пользователя для Zscaler Private Access (ZPA) в диалоговом окне назначения нужно выбрать действительную роль для конкретного приложения (при наличии). Пользователи с ролью **Доступ по умолчанию** исключаются из подготовки.

## <a name="set-up-zscaler-private-access-zpa-for-provisioning"></a>Настройка подготовки в Zscaler Private Access (ZPA)

1. Войдите в [консоль администрирования Zscaler Private Access (ZPA)](https://admin.private.zscaler.com/). Перейдите к разделу **Administration > IdP Configuration** (Администрирование > Конфигурация поставщика удостоверений).

    ![Консоль администрирования Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/idpconfig.png)

2.  Убедитесь, что для **единого входа** настроен поставщик удостоверений. Если поставщик удостоверений не настроен, добавьте его, щелкнув значок "плюс" в правом верхнем углу экрана.

    ![Zscaler Private Access (ZPA) — добавление SCIM](media/zscaler-private-access-provisioning-tutorial/plusicon.png)

3. Выполните инструкции мастера настройки **Add IdP Configuration** (Добавление поставщиков удостоверений), чтобы добавить поставщик удостоверений. В поле **Single sign-on** (Единый вход) оставьте значение **User** (Пользователь). Укажите значение **Name** (Имя) и выберите домены из раскрывающегося списка **Domains**. Щелкните **Next** (Далее), чтобы перейти к следующему окну.

    ![Zscaler Private Access (ZPA) — добавление поставщика удостоверений](media/zscaler-private-access-provisioning-tutorial/addidp.png)

4. Скачайте **Service Provider Certificate** (Сертификат поставщика услуг). Щелкните **Next** (Далее), чтобы перейти к следующему окну.

    ![Zscaler Private Access (ZPA) — сертификат поставщика услуг](media/zscaler-private-access-provisioning-tutorial/spcertificate.png)

5. В следующем окне отправьте скачанный ранее **сертификат поставщика услуг**.

    ![Zscaler Private Access (ZPA) — отправка сертификата](media/zscaler-private-access-provisioning-tutorial/uploadfile.png)

6.  Прокрутите страницу вниз, чтобы ввести значения **Single sign-On URL** (URL-адрес единого входа) и **IdP Entity ID** (Идентификатор сущности поставщика удостоверений).

    ![Zscaler Private Access (ZPA) — идентификатор поставщика удостоверений](media/zscaler-private-access-provisioning-tutorial/idpid.png)

7.  Прокрутите страницу вниз до раздела **Enable SCIM Sync** (Включить синхронизацию SCIM). Нажмите кнопку **Generate New Token** (Сгенерировать новый токен). Скопируйте значение **Bearer Token** (Токен носителя). Его нужно будет ввести в поле "Секретный токен" на вкладке "Подготовка" для приложения Zscaler Private Access (ZPA) на портале Azure.

    ![Zscaler Private Access (ZPA) — создание токена](media/zscaler-private-access-provisioning-tutorial/token.png)

8.  Чтобы найти значение **Tenant URL** (URL-адрес клиента), перейдите к разделу **Administration > IdP Configuration** (Администрирование > Конфигурация поставщика удостоверений). Щелкните имя недавно добавленной конфигурации поставщика удостоверений, которая отображается в списке на этой странице.

    ![Zscaler Private Access (ZPA) — имя поставщика удостоверений](media/zscaler-private-access-provisioning-tutorial/idpname.png)

9.  Прокрутите страницу вниз до элемента **SCIM Service Provider Endpoint** (Конечная точка SCIM поставщика услуг), который находится в самом низу страницы. Скопируйте значение **SCIM Service Provider Endpoint** (Конечная точка SCIM поставщика услуг). Его нужно будет ввести в поле "URL-адрес клиента" на вкладке "Подготовка" для приложения Zscaler Private Access (ZPA) на портале Azure.

    ![Zscaler Private Access (ZPA) — URL-адрес SCIM](media/zscaler-private-access-provisioning-tutorial/tenanturl.png)


## <a name="add-zscaler-private-access-zpa-from-the-gallery"></a>Добавление Zscaler Private Access (ZPA) из коллекции

Прежде чем настроить в Zscaler Private Access (ZPA) автоматическую подготовку пользователей в Azure AD, нужно добавить Zscaler Private Access (ZPA) из коллекции приложений Azure AD в список управляемых приложений SaaS.

**Чтобы добавить Zscaler Private Access (ZPA) из коллекции приложений Azure AD, выполните следующие действия.**

1. На **[портале Azure](https://portal.azure.com)** в области навигации слева выберите элемент **Azure Active Directory**.

    ![Кнопка Azure Active Directory](common/select-azuread.png)

2. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

3. Чтобы добавить новое приложение, в области сверху нажмите кнопку **Новое приложение**.

    ![Кнопка "Создать приложение"](common/add-new-app.png)

4. В поле поиска введите **Zscaler Private Access (ZPA)** , выберите **Zscaler Private Access (ZPA)** в области результатов, а затем нажмите кнопку **Добавить**, чтобы добавить приложение.

    ![Zscaler Private Access (ZPA) в списке результатов](common/search-new-app.png)

## <a name="configuring-automatic-user-provisioning-to-zscaler-private-access-zpa"></a>Настройка автоматической подготовки пользователей в Zscaler Private Access (ZPA) 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в Zscaler Private Access (ZPA) на основе их назначений в Azure AD.

> [!TIP]
> Вы также можете настроить для Zscaler Private Access (ZPA) единый вход на основе SAML, используя инструкции из [руководства по единому входу для Zscaler Private Access (ZPA)](./zscalerprivateaccess-tutorial.md). Единый вход можно настроить независимо от автоматической подготовки пользователей, хотя эти две возможности хорошо дополняют друг друга.

> [!NOTE]
> При подготовке или отмене подготовки пользователей и групп рекомендуется периодически перезапускать подготовку, чтобы членство в группах правильно обновлялось. При перезапуске служба повторно оценит все группы и обновит членство.  

> [!NOTE]
> Дополнительные сведения о конечной точке SCIM для Zscaler Private Access см. [здесь](https://www.zscaler.com/partners/microsoft).

### <a name="to-configure-automatic-user-provisioning-for-zscaler-private-access-zpa-in-azure-ad"></a>Для настройки автоматической подготовки пользователей в Azure AD для Zscaler Private Access (ZPA) сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. Из списка приложений выберите **Zscaler Private Access (ZPA)** .

    ![Ссылка на Zscaler Private Access (ZPA) в списке "Приложения"](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите в поле **URL-адрес клиента** полученное ранее значение **SCIM Service Provider Endpoint** (Конечная точка SCIM поставщика услуг). Введите полученное ранее значение **Bearer Token** (Токен носителя) в поле **Секретный токен**. Щелкните **Проверить подключение** и убедитесь, что Azure AD может подключиться к Zscaler Private Access (ZPA). Если установить подключение не удалось, убедитесь, что у учетной записи Zscaler Private Access (ZPA) есть разрешения администратора, и повторите попытку.

    ![URL-адрес клиента + токен](common/provisioning-testconnection-tenanturltoken.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Send an email notification when a failure occurs** (Отправить уведомление по электронной почте при сбое).

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Выберите команду **Сохранить**.

8. В разделе **Сопоставления** выберите **Синхронизировать пользователей Azure Active Directory с Zscaler Private Access (ZPA)** .

    ![Сопоставления пользователей Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/usermappings.png)

9. В разделе **Сопоставления атрибутов** просмотрите атрибуты пользователей, которые синхронизируются из Azure AD в Zscaler Private Access (ZPA). Атрибуты, выбранные как свойства **Сопоставление**, используются для сопоставления учетных записей пользователей в Zscaler Private Access (ZPA) при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты пользователей Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/userattributes.png)

10. В разделе **Сопоставления** выберите **Синхронизировать группы Azure Active Directory с Zscaler Private Access (ZPA)** .

    ![Сопоставления групп Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/groupmappings.png)

11. В разделе **Сопоставления атрибутов** просмотрите атрибуты групп, которые синхронизируются из Azure AD в Zscaler Private Access (ZPA). Атрибуты, выбранные как свойства **Сопоставление**, используются для сопоставления групп в Zscaler Private Access (ZPA) при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    ![Атрибуты групп Zscaler Private Access (ZPA)](media/zscaler-private-access-provisioning-tutorial/groupattributes.png)

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Zscaler Private Access (ZPA), измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Укажите пользователей или группы для подготовки в Zscaler Private Access (ZPA), выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется начальная синхронизация пользователей и (или) групп, определенных в поле **Область** раздела **Параметры**. Начальная синхронизация занимает больше времени, чем последующие операции синхронизации. Если служба запущена, они выполняются примерно каждые 40 минут. В разделе **Текущее состояние** можно отслеживать ход выполнения и переходить по ссылкам для просмотра отчетов о подготовке, в которых зафиксированы все действия, выполняемые службой подготовки Azure AD с приложением Zscaler Private Access (ZPA).

Дополнительные сведения о чтении журналов подготовки Azure AD см. в руководстве по [отчетам об автоматической подготовке учетных записей](../app-provisioning/check-status-user-account-provisioning.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)