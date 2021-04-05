---
title: Руководство по настройке SAP Analytics Cloud для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить автоматическую подготовку и отзыв учетных записей пользователей из Azure AD в SAP Analytics Cloud.
services: active-directory
documentationcenter: ''
author: Zhchia
writer: Zhchia
manager: beatrizd
ms.assetid: 27d12989-efa8-4254-a4ad-8cb6bf09d839
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 08/13/2020
ms.author: Zhchia
ms.openlocfilehash: 31e5393cb5de627ebf8832e43302583d6eacbf59
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96181500"
---
# <a name="tutorial-configure-sap-analytics-cloud-for-automatic-user-provisioning"></a>Руководство по настройке SAP Analytics Cloud для автоматической подготовки пользователей

В этом учебнике описаны действия, которые необходимо выполнить в SAP Analytics Cloud и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически осуществляет и отменяет подготовку пользователей и групп для [SAP Analytics Cloud](https://www.sapanalytics.cloud/) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в SAP Analytics Cloud
> * Удаление пользователей в SAP Analytics Cloud, если им больше не нужен доступ
> * Синхронизация атрибутов пользователей между Azure AD и SAP Analytics Cloud.
> * [Единый вход](sapboc-tutorial.md) в SAP Analytics Cloud (рекомендуется)

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* Клиент SAP Analytics Cloud
* Учетная запись пользователя в консоли администрирования "Подготовка удостоверений" SAP с разрешениями администратора. Убедитесь, что у вас есть доступ к системным прокси-серверам в консоли администрирования "Подготовка удостоверений". Если вы не видите элемент **Proxy Systems** (Прокси-системы), создайте инцидент для компонента **BC-IAM-IP**, чтобы запросить доступ к этому элементу.
* Клиент OAuth с учетными данными клиента для предоставления авторизации в SAP Analytics Cloud. Чтобы узнать, как это сделать, ознакомьтесь со следующими материалами: [Управление клиентами OAuth и доверенными поставщиками удостоверений](https://help.sap.com/viewer/00f68c2e08b941f081002fd3691d86a7/release/en-US/4f43b54398fc4acaa5efa32badfe3df6.html)

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки

1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и SAP Analytics Cloud](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-sap-analytics-cloud-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка SAP Analytics Cloud для поддержки подготовки с помощью Azure AD

1. Войдите в [консоль администрирования подготовки удостоверений SAP](https://ips-xlnk9v890j.dispatcher.us1.hana.ondemand.com/) с учетной записью администратора, а затем выберите **Proxy Systems** (Прокси-системы).

   ![Прокси-системы SAP](./media/sap-analytics-cloud-provisioning-tutorial/sap-proxy-systems.png.png)

2. Выберите **Свойства**.

   ![Свойства прокси-систем SAP](./media/sap-analytics-cloud-provisioning-tutorial/sap-proxy-systems-properties.png)

3. Скопируйте **URL-адрес** и добавьте `/api/v1/scim` к URL-адресу. Сохраните его для последующего использования в поле **URL-адрес клиента**.

   ![URL-адрес свойств прокси-систем SAP](./media/sap-analytics-cloud-provisioning-tutorial/sap-proxy-systems-details-url.png)

4. Используйте [POSTMAN](https://www.postman.com/), чтобы отправить HTTP-запрос POST по адресу: `<Token URL>?grant_type=client_credentials`, где `Token URL` — это URL-адрес в поле **OAuth2TokenServiceURL**. Этот шаг необходим для создания маркера доступа, который будет использоваться в поле секретного токена при настройке автоматической подготовки.

   ![OAuth свойств IP прокси-систем SAP](./media/sap-analytics-cloud-provisioning-tutorial/sap-proxy-systems-details-oauth.png)

5. В Postman используйте параметр **Обычная проверка подлинности** и задайте идентификатор клиента OAuth в качестве пользователя и секрет в качестве пароля. Этот вызов возвращает маркер доступа. Сохраните эти данные для дальнейшего использования в поле **Секретный токен**.

   ![Запрос POST в Postman](./media/sap-analytics-cloud-provisioning-tutorial/postman-post-request.png)

## <a name="step-3-add-sap-analytics-cloud-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление SAP Analytics Cloud из коллекции приложений Azure AD

Добавление SAP Analytics Cloud из коллекции приложений Azure AD, чтобы начать управление подготовкой в SAP Analytics Cloud. Если вы ранее настроили SAP Analytics Cloud для единого входа, можете использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для SAP Analytics Cloud необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-sap-analytics-cloud"></a>Шаг 5. Настройка автоматической подготовки пользователей в SAP Analytics Cloud 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-sap-analytics-cloud-in-azure-ad"></a>Чтобы настроить автоматическую подготовку учетных записей пользователей для SAP Analytics Cloud в Azure AD, сделайте следующее.

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **SAP Analytics Cloud**.

    ![Ссылка на SAP Analytics Cloud в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите в поле **URL-адрес арендатора** значение URL-адреса, полученное ранее. Введите значение маркера доступа, полученное ранее из поля **Secret Token** (Секретный токен). Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к InVision. Если установить подключение не удалось, убедитесь, что у учетной записи SAP Analytics Cloud есть разрешения администратора, и повторите попытку.

    ![Снимок экрана: диалоговое окно учетных данных администратора, в котором можно ввести URL-адрес клиента и секретный токен.](./media/sap-analytics-cloud-provisioning-tutorial/provisioning.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Подготовка пользователей Azure Active Directory**.

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в SAP Analytics Cloud. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в SAP Analytics Cloud при операциях обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), сначала убедитесь, что API SAP Analytics Cloud поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

   |attribute|Тип|Поддерживается для фильтрации|
   |---|---|---|
   |userName|Строка|&check;|
   |name.givenName|Строка|
   |name.familyName|Строка|
   |active|Логическое|
   |emails[type eq "work"].value|Строка|
   |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Строка|

10. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

11. Чтобы включить службу подготовки Azure AD для SAP Analytics Cloud, в разделе **Параметры** измените значение параметра **Состояние подготовки** на **Включено**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

12. Определите пользователей и (или) группы для подготовки в SAP Analytics Cloud, выбрав нужные значения в поле **Область** в разделе **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

13. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

1. Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
2. Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
3. Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)