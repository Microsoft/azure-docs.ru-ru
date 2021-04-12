---
title: Руководство по настройке Oracle Cloud Infrastructure Console для автоматической подготовки пользователей с помощью Azure Active Directory | Документация Майкрософт
description: Сведения о настройке автоматической подготовки и отзыва учетных записей пользователей из Azure AD в Oracle Cloud Infrastructure Console.
services: active-directory
author: zchia
writer: zchia
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 01/16/2020
ms.author: Zhchia
ms.openlocfilehash: 94de0ca0a5393c891e567e558cbbadd0ca1f453b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97832086"
---
# <a name="tutorial-configure-oracle-cloud-infrastructure-console-for-automatic-user-provisioning"></a>Руководство по настройке автоматической подготовки пользователей в Oracle Cloud Infrastructure Console

В этом учебнике описаны действия, которые необходимо выполнить в Oracle Cloud Infrastructure Console и Azure Active Directory (Azure AD) для настройки автоматической подготовки пользователей. После настройки Azure AD автоматически подготавливает и отзывает пользователей и группы в [Oracle Cloud Infrastructure Console](https://www.oracle.com/cloud/free/?source=:ow:o:p:nav:0916BCButton&intcmp=:ow:o:p:nav:0916BCButton) с помощью службы подготовки Azure AD. Подробные сведения о том, что делает эта служба, как она работает, и часто задаваемые вопросы см. в статье [Автоматическая подготовка пользователей и ее отзыв для приложений SaaS в Azure Active Directory](../app-provisioning/user-provisioning.md). 


## <a name="capabilities-supported"></a>Поддерживаемые возможности
> [!div class="checklist"]
> * Создание пользователей в Oracle Cloud Infrastructure Console.
> * Удаление пользователей из Oracle Cloud Infrastructure Console, когда доступ им больше не нужен.
> * Синхронизация атрибутов пользователей между Azure AD и Oracle Cloud Infrastructure Console.
> * Подготовка групп и членство в группе в Oracle Cloud Infrastructure Console.
> * [Единый вход](./oracle-cloud-tutorial.md) в Oracle Cloud Infrastructure Console (рекомендуется)

## <a name="prerequisites"></a>Предварительные требования

В сценарии, описанном в этом руководстве, предполагается, что у вас уже имеется:

* [Клиент Azure AD.](../develop/quickstart-create-new-tenant.md) 
* Учетная запись пользователя в Azure AD с [разрешением](../roles/permissions-reference.md) на настройку подготовки (например, администратор приложений, администратор облачных приложений, владелец приложения или глобальный администратор). 
* [Арендатор](https://www.oracle.com/cloud/sign-in.html?intcmp=OcomFreeTier&source=:ow:o:p:nav:0916BCButton) Oracle Cloud Infrastructure Console.
* Учетная запись пользователя в Oracle Cloud Infrastructure Console с разрешениями администратора.

## <a name="step-1-plan-your-provisioning-deployment"></a>Шаг 1. Планирование развертывания для подготовки
1. Узнайте, [как работает служба подготовки](../app-provisioning/user-provisioning.md).
2. Определите, кто будет находиться в [области подготовки](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).
3. Определите, какие данные следует [сопоставлять между Azure AD и Oracle Cloud Infrastructure Console](../app-provisioning/customize-application-attributes.md). 

## <a name="step-2-configure-oracle-cloud-infrastructure-console-to-support-provisioning-with-azure-ad"></a>Шаг 2. Настройка в Oracle Cloud Infrastructure Console поддержки подготовки с помощью Azure AD

1. Войдите на портал администрирования Oracle Cloud Infrastructure Console. В верхнем правом углу экрана выберите **Удостоверение > Федерация**.

    ![Администратор Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/identity.png)

2. Щелкните URL-адрес, отображаемый на странице рядом с Oracle Identity Cloud Service Console.

    ![URL-адрес Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/url.png)

3. Щелкните **Добавить поставщик удостоверений**, чтобы создать поставщик удостоверений. Сохраните идентификатор поставщика удостоверений для использования как часть URL-адреса клиента. Щелкните значок плюса рядом со вкладкой **Приложения**, чтобы создать клиент OAuth и AppRole администратора домена предоставления удостоверений IDCS.

    ![Значок Oracle Cloud](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/add.png)

4. Настройте приложение, выполнив действия, указанные на приведенных ниже снимках экрана. Завершив настройку, щелкните **Сохранить**.

    ![Конфигурация Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/configuration.png)

    ![Политика выдачи токенов Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/token-issuance.png)

5. На вкладке конфигураций приложения разверните параметр **Общие сведения**, чтобы извлечь идентификатор и секрет клиента.

    ![Создание токена Oracle](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/general-information.png)

6. Чтобы сгенерировать секретный токен, закодируйте идентификатор клиента и секрет клиента в Base64-формат **client ID:Client Secret**. Сохраните секретный токен. Это значение нужно будет ввести в поле **Секретный токен** на вкладке подготовки для приложения Oracle Cloud Infrastructure Console на портале Azure.

## <a name="step-3-add-oracle-cloud-infrastructure-console-from-the-azure-ad-application-gallery"></a>Шаг 3. Добавление Oracle Cloud Infrastructure Console из коллекции приложений Azure AD

Добавьте Oracle Cloud Infrastructure Console из коллекции приложений Azure AD, чтобы начать управление подготовкой в Oracle Cloud Infrastructure Console. Если вы ранее настроили единый вход в e Cloud Infrastructure Console, можно использовать то же приложение. Однако при первоначальном тестировании интеграции рекомендуется создать отдельное приложение. Дополнительные сведения о добавлении приложения из коллекции см. [здесь](../manage-apps/add-application-portal.md). 

## <a name="step-4-define-who-will-be-in-scope-for-provisioning"></a>Шаг 4. Определение пользователей для включения в область подготовки 

Служба подготовки Azure AD позволяет определить пользователей, которые будут подготовлены, на основе назначения приложению и (или) атрибутов пользователя или группы. Если вы решили определить пользователей на основе назначения, выполните следующие [действия](../manage-apps/assign-user-or-group-access-portal.md), чтобы назначить пользователей и группы приложению. Если вы решили указать, кто именно будет подготовлен, на основе одних только атрибутов пользователя или группы, можете использовать фильтр задания области, как описано [здесь](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 

* При назначении пользователей и групп для Oracle Cloud Infrastructure Console необходимо выбрать роль, отличную от роли **Доступ по умолчанию**. Пользователи с ролью "Доступ по умолчанию" исключаются из подготовки и будут помечены в журналах подготовки как не назначенные явно. Кроме того, если эта роль является единственной, доступной в приложении, можно [изменить манифест приложения](../develop/howto-add-app-roles-in-azure-ad-apps.md), чтобы добавить дополнительные роли. 

* Начните с малого. Протестируйте небольшой набор пользователей и групп, прежде чем выполнять развертывание для всех. Если в область подготовки включены назначенные пользователи и группы, проверьте этот механизм, назначив приложению одного или двух пользователей либо одну или две группы. Если в область включены все пользователи и группы, можно указать [фильтр области на основе атрибутов](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md). 


## <a name="step-5-configure-automatic-user-provisioning-to-oracle-cloud-infrastructure-console"></a>Шаг 5. Настройка автоматической подготовки пользователей в Oracle Cloud Infrastructure Console 

В этом разделе описывается, как настроить службу подготовки Azure AD для создания, обновления и отключения пользователей и (или) групп в TestApp на основе их назначений в Azure AD.

### <a name="to-configure-automatic-user-provisioning-for-oracle-cloud-infrastructure-console-in-azure-ad"></a>Чтобы настроить автоматическую подготовку пользователей в Oracle Cloud Infrastructure Console, сделайте следующее:

1. Войдите на [портал Azure](https://portal.azure.com). Выберите **Корпоративные приложения**, а затем **Все приложения**.

    ![Колонка "Корпоративные приложения"](common/enterprise-applications.png)

2. В списке приложений выберите **Oracle Cloud Infrastructure Console**.

    ![Ссылка на Oracle Cloud Infrastructure Console в списке приложений](common/all-applications.png)

3. Выберите вкладку **Подготовка**.

    ![Снимок экрана: раздел "Управление" с выделенным параметром "Подготовка".](common/provisioning.png)

4. Для параметра **Режим подготовки к работе** выберите значение **Automatic** (Автоматически).

    ![Снимок экрана: раскрывающийся список "Режим подготовки" с выделенным параметром "Автоматически".](common/provisioning-automatic.png)

5. В разделе **Учетные данные администратора** введите **URL-адрес клиента** в формате `https://<IdP ID>.identity.oraclecloud.com/admin/v1`. Например, `https://idcs-0bfd023ff2xx4a98a760fa2c31k92b1d.identity.oraclecloud.com/admin/v1`. Введите значение секретного токена, полученное ранее на шаге **Секретный токен**. Щелкните **Проверить подключение**, чтобы убедиться, что Azure AD может подключиться к Oracle Cloud Infrastructure Console. Если установить подключение не удалось, убедитесь, что у учетной записи Oracle Cloud Infrastructure Console есть разрешения администратора, и повторите попытку.

    ![Снимок экрана: диалоговое окно учетных данных администратора, в котором можно ввести URL-адрес клиента и секретный токен.](./media/oracle-cloud-infratstructure-console-provisioning-tutorial/provisioning.png)

6. В поле **Почтовое уведомление** введите адрес электронной почты пользователя или группы, которые должны получать уведомления об ошибках подготовки, а также установите флажок **Отправить уведомление по электронной почте при сбое**.

    ![Почтовое уведомление](common/provisioning-notification-email.png)

7. Щелкните **Сохранить**.

8. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Users to Oracle Cloud Infrastructure Console** (Синхронизировать пользователей Azure Active Directory с Oracle Cloud Infrastructure Console).

9. В разделе **Сопоставление атрибутов** просмотрите пользовательские атрибуты, которые синхронизированы из Azure AD в Oracle Cloud Infrastructure Console. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления учетных записей пользователей в Oracle Cloud Infrastructure Console для операций обновления. Если вы решили изменить [целевой атрибут сопоставления](../app-provisioning/customize-application-attributes.md), потребуется убедиться, что API Oracle Cloud Infrastructure Console поддерживает фильтрацию пользователей по этому атрибуту. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    |attribute|Тип|
    |---|---|
    |displayName|Строка|
    |userName|Строка|
    |active|Логическое|
    |title|Строка|
    |emails[type eq "work"].value|Строка|
    |preferredLanguage|Строка|
    |name.givenName|Строка|
    |name.familyName|Строка|
    |addresses[type eq "work"].formatted|Строка|
    |addresses[type eq "work"].locality|Строка|
    |addresses[type eq "work"].region|Строка|
    |addresses[type eq "work"].postalCode|Строка|
    |addresses[type eq "work"].country|Строка|
    |addresses[type eq "work"].streetAddress|Строка|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|Строка|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|Строка|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:costCenter|Строка|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:division|Строка|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager|Справочник|
    |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:organization|Строка|
    |urn:ietf:params:scim:schemas:oracle:idcs:extension:user:User:bypassNotification|логический|
    |urn:ietf:params:scim:schemas:oracle:idcs:extension:user:User:isFederatedUser|логический|

10. В разделе **Сопоставления** выберите **Synchronize Azure Active Directory Groups to Oracle Cloud Infrastructure Console** (Синхронизировать группы Azure Active Directory с Oracle Cloud Infrastructure Console).

11. В разделе **Сопоставление атрибутов** просмотрите атрибуты группы, которые синхронизированы из Azure AD в Oracle Cloud Infrastructure Console. Атрибуты, выбранные как свойства **сопоставления**, используются для сопоставления групп в Oracle Cloud Infrastructure Console при операциях обновления. Нажмите кнопку **Сохранить**, чтобы зафиксировать все изменения.

    | attribute | Тип |
    |--|--|
    | displayName | Строка |
    | externalId | Строка |
    | members | Справочник |

12. Чтобы настроить фильтры области, ознакомьтесь со следующими инструкциями, предоставленными в [руководстве по фильтрам области](../app-provisioning/define-conditional-rules-for-provisioning-user-accounts.md).

13. Чтобы включить службу подготовки Azure AD для Oracle Cloud Infrastructure Console, измените значение параметра **Состояние подготовки** на **Включено** в разделе **Параметры**.

    ![Состояние подготовки "Включено"](common/provisioning-toggle-on.png)

14. Определите пользователей и (или) группы для подготовки в Oracle Cloud Infrastructure Console, выбрав нужные значения в поле **Область** раздела **Параметры**.

    ![Область действия подготовки](common/provisioning-scope.png)

15. Когда будете готовы выполнить подготовку, нажмите кнопку **Сохранить**.

    ![Сохранение конфигурации подготовки](common/provisioning-configuration-save.png)

После этого начнется цикл начальной синхронизации всех пользователей и групп, определенных в поле **Область** в разделе **Параметры**. Начальный цикл занимает больше времени, чем последующие циклы. Пока служба подготовки Azure AD запущена, они выполняются примерно каждые 40 минут. 

## <a name="step-6-monitor-your-deployment"></a>Шаг 6. Мониторинг развертывания
После настройки подготовки используйте следующие ресурсы для мониторинга развертывания.

* Используйте [журналы подготовки](../reports-monitoring/concept-provisioning-logs.md), чтобы определить, какие пользователи были подготовлены успешно или неудачно.
* Используйте [индикатор выполнения](../app-provisioning/application-provisioning-when-will-provisioning-finish-specific-user.md), чтобы узнать состояние цикла подготовки и приблизительное время до его завершения.
* Если конфигурация подготовки, вероятно, находится в неработоспособном состоянии, приложение перейдет в карантин. Дополнительные сведения о режимах карантина см. [здесь](../app-provisioning/application-provisioning-quarantine-status.md).

## <a name="additional-resources"></a>Дополнительные ресурсы

* [Управление подготовкой учетных записей пользователей для корпоративных приложений](../app-provisioning/configure-automatic-user-provisioning-portal.md)
* [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
