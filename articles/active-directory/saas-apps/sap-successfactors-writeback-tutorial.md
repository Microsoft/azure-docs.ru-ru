---
title: Руководство по настройке обратной записи SAP SuccessFactors в Azure Active Directory | Документация Майкрософт
description: Узнайте, как настроить обратную запись атрибута из Azure AD в SAP SuccessFactors.
services: active-directory
author: cmmdesai
manager: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.topic: tutorial
ms.workload: identity
ms.date: 10/14/2020
ms.author: chmutali
ms.openlocfilehash: 3260787dec4ae26cd6ef7cc3bd562f39db8e3655
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "99526981"
---
# <a name="tutorial-configure-attribute-write-back-from-azure-ad-to-sap-successfactors"></a>Руководство по настройке обратной записи данных из Azure AD в SAP SuccessFactors
Цель этого учебника — продемонстрировать действия, которые необходимо выполнить для обратной записи атрибутов из Azure AD в SAP SuccessFactors Employee Central. 

## <a name="overview"></a>Обзор

Для записи конкретных атрибутов из Azure Active Directory в SAP SuccessFactors Employee Central можно настроить приложение для обратной записи в SAP SuccessFactors. Приложение подготовки обратной записи в SuccessFactors поддерживает назначение значений следующим атрибутам Employee Central:

* рабочий адрес электронной почты;
* Имя пользователя
* номер рабочего телефона (включая код страны, код города, номер и добавочный номер);
* основной флаг номера рабочего телефона;
* номер мобильного телефона (включая код страны, код города, номер);
* основной флаг мобильного телефона; 
* пользовательские custom01–custom15;
* атрибут loginMethod.

> [!NOTE]
> Это приложение не зависит от приложений интеграции SuccessFactors для подготовки входящих пользователей. Вы можете настроить его независимо от приложения подготовки для записи [из SuccessFactors в локальную службу AD](sap-successfactors-inbound-provisioning-tutorial.md) или [из SuccessFactors в Azure AD](sap-successfactors-inbound-provisioning-cloud-only-tutorial.md).

### <a name="who-is-this-user-provisioning-solution-best-suited-for"></a>Кому это решение подготовки пользователей подходит лучше всего?

Это решение подготовки пользователей для обратной записи в SuccessFactors идеально подходит для:

* организаций, использующих Microsoft 365 и желающих выполнять обратную запись достоверных атрибутов, которыми управляет ИТ-служба (например, адреса электронной почты, телефона, имени пользователя) в SuccessFactors Employee Central.

## <a name="configuring-successfactors-for-the-integration"></a>Настройка SuccessFactors для интеграции

Для всех соединителей подготовки SuccessFactors требуются учетные данные SuccessFactors с соответствующими разрешениями для вызова API-интерфейсов OData Employee Central. В этом разделе описана процедура создания учетной записи службы в SuccessFactors и предоставления соответствующих разрешений. 

* [Создание или определение учетной записи пользователя API в SuccessFactors](#createidentify-api-user-account-in-successfactors)
* [Создание роли разрешений для API](#create-an-api-permissions-role)
* [Создание группы разрешений для пользователя API](#create-a-permission-group-for-the-api-user)
* [Предоставление роли разрешений группе разрешений](#grant-permission-role-to-the-permission-group)

### <a name="createidentify-api-user-account-in-successfactors"></a>Создание или определение учетной записи пользователя API в SuccessFactors
Совместно с группой администраторов или партнером по внедрению SuccessFactors создайте или укажите в SuccessFactors учетную запись пользователя, которая будет использоваться для вызова API-интерфейсов OData. При настройке приложений подготовки в Azure AD потребуются имя пользователя и пароль этой учетной записи. 

### <a name="create-an-api-permissions-role"></a>Создание роли разрешений для API

1. Войдите в SAP SuccessFactors с учетной записью пользователя, имеющего доступ к центру администрирования.
1. Введите в строке поиска *Manage Permission Roles* (Управление ролями разрешений), а затем среди результатов поиска выберите **Manage Permission Roles** (Управление ролями разрешений).

   ![Управление ролями разрешений](./media/sap-successfactors-inbound-provisioning/manage-permission-roles.png)

1. В списке ролей разрешений щелкните **Create New** (Создать).

   > [!div class="mx-imgBorder"]
   > ![Создание роли разрешения](./media/sap-successfactors-inbound-provisioning/create-new-permission-role-1.png)

1. Заполните поля **Role Name** (Имя роли) и **Description** (Описание) для новой роли разрешений. Имя и описание должны указывать на то, что роль предназначена для разрешений на использование API.

   > [!div class="mx-imgBorder"]
   > ![Сведения о роли разрешений](./media/sap-successfactors-inbound-provisioning/permission-role-detail.png)

1. В разделе Permission settings (Параметры разрешений) щелкните **Permission...** (Разрешение...), затем прокрутите список разрешений вниз и щелкните **Manage Integration Tools** (Управление инструментами интеграции). Установите флажок **Allow Admin to Access to OData API through Basic Authentication** (Разрешить администратору доступ к API-интерфейсу OData с использованием обычной проверки подлинности).

   > [!div class="mx-imgBorder"]
   > ![Раздел Manage integration tools (Управление инструментами интеграции)](./media/sap-successfactors-inbound-provisioning/manage-integration-tools.png)

1. Прокрутите вниз в том же поле и выберите **Employee Central API** (API-интерфейс Employee Central). Добавьте разрешения на чтение и изменение с помощью API-интерфейса OData, как показано ниже. Параметр изменения следует выбирать, если вы планируете использовать ту же учетную запись и для сценария обратной записи в SuccessFactors. 

   > [!div class="mx-imgBorder"]
   > ![Разрешения на чтение и запись](./media/sap-successfactors-inbound-provisioning/odata-read-write-perm.png)

1. Нажмите кнопку **Готово**. Щелкните **Сохранить изменения**.

### <a name="create-a-permission-group-for-the-api-user"></a>Создание группы разрешений для пользователя API

1. В центре администрирования SuccessFactors в строке поиска введите *Manage Permission Groups* (Управление группами разрешений), а затем среди результатов поиска выберите **Manage Permission Groups** (Управление группами разрешений).

   > [!div class="mx-imgBorder"]
   > ![Пункт Manage permission groups (Управление группами разрешений)](./media/sap-successfactors-inbound-provisioning/manage-permission-groups.png)

1. В окне Manage permission groups (Управление группами разрешений) щелкните **Create New** (Создать).

   > [!div class="mx-imgBorder"]
   > ![Добавление новой группы](./media/sap-successfactors-inbound-provisioning/create-new-group.png)

1. Добавьте имя новой группы в поле Group Name (Имя группы). Это имя должно указывать на то, что группа предназначена для пользователей API.

   > [!div class="mx-imgBorder"]
   > ![Имя группы разрешений](./media/sap-successfactors-inbound-provisioning/permission-group-name.png)

1. Добавьте членов в группу. Например, в раскрывающемся меню People Pool (Пул пользователей) можно выбрать **Username** (Имя пользователя), а затем ввести имя пользователя учетной записи API, которая будет использоваться для интеграции. 

   > [!div class="mx-imgBorder"]
   > ![Добавление членов группы](./media/sap-successfactors-inbound-provisioning/add-group-members.png)

1. Щелкните **Done** (Готово), чтобы завершить создание группы разрешений.

### <a name="grant-permission-role-to-the-permission-group"></a>Предоставление роли разрешений группе разрешений

1. В центре администрирования SuccessFactors в строке поиска введите *Manage Permission Roles* (Управление ролями разрешений), а затем среди результатов поиска выберите **Manage Permission Roles** (Управление ролями разрешений).
1. В разделе **Permission Role List** (Список ролей разрешений) выберите роль, созданную для разрешений на использование API.
1. В разделе **Grant this role to...** (Предоставить эту роль...) нажмите кнопку **Add...** (Добавить...).
1. В раскрывающемся меню выберите **Permission Group...** (Группа разрешений...), а затем щелкните **Select...** (Выбрать...), чтобы открыть окно Groups (Группы) для поиска и выбора созданной ранее группы. 

   > [!div class="mx-imgBorder"]
   > ![Добавление группы разрешений](./media/sap-successfactors-inbound-provisioning/add-permission-group.png)

1. Просмотрите роль разрешений, предоставленную группе разрешений. 
   > [!div class="mx-imgBorder"]
   > ![Сведения о роли и группе разрешений](./media/sap-successfactors-inbound-provisioning/permission-role-group.png)

1. Щелкните **Сохранить изменения**.

## <a name="preparing-for-successfactors-writeback"></a>Подготовка к обратной записи в SAP SuccessFactors

Приложение подготовки обратной записи в SuccessFactors использует определенные значения *кода* для настройки адреса электронной почты и номеров телефонов в Employee Central. Эти значения *кода* задаются как постоянные значения в таблице сопоставления атрибутов и отличаются для каждого экземпляра SuccessFactors. В этом разделе описана процедура записи этих значений *кода*.

   > [!NOTE]
   > Для выполнения действий, описанных в этом разделе, потребуется помощь администратора SuccessFactors. 

### <a name="identify-email-and-phone-number-picklist-names"></a>Выявление имен в поле выбора адреса электронной почты и номера телефона 

В SAP SuccessFactors *поле выбора* — это настраиваемый набор вариантов, предлагаемых на выбор пользователю. С помощью полей выбора представляются различные типы адресов электронной почты и номеров телефонов (например, рабочий, личный, другой). На этом шаге мы определим поля выбора, настроенные в клиенте SuccessFactors для хранения значений адресов электронной почты и номеров телефонов. 
 
1. В центре администрирования SuccessFactors найдите *Manage business configuration* (Управление бизнес-конфигурацией). 

   > [!div class="mx-imgBorder"]
   > ![Поиск Manage business configuration (Управление бизнес-конфигурацией)](./media/sap-successfactors-inbound-provisioning/manage-business-config.png)

1. В разделе **HRIS Elements** (Элементы HRIS) выберите **emailInfo** и щелкните *Details* (Сведения) для поля **email-type**.

   > [!div class="mx-imgBorder"]
   > ![Получение сведений об адресе электронной почты](./media/sap-successfactors-inbound-provisioning/get-email-info.png)

1. На странице сведений о поле **email-type** указано имя связанного с ним поля выбора. Запишите это имя. По умолчанию это **ecEmailType**. Однако в вашем клиенте оно может отличаться. 

   > [!div class="mx-imgBorder"]
   > ![Определение поля выбора типа адреса электронной почты](./media/sap-successfactors-inbound-provisioning/identify-email-picklist.png)

1. В разделе **HRIS Elements** (Элементы HRIS) выберите **phoneInfo** и щелкните *Details* (Сведения) для поля **phone-type**.

   > [!div class="mx-imgBorder"]
   > ![Получение сведений о телефоне](./media/sap-successfactors-inbound-provisioning/get-phone-info.png)

1. На странице сведений о поле **phone-type** указано имя связанного с ним поля выбора. Запишите это имя. По умолчанию это **ecPhoneType**. Однако в вашем клиенте оно может отличаться. 

   > [!div class="mx-imgBorder"]
   > ![Определение поля выбора телефона](./media/sap-successfactors-inbound-provisioning/identify-phone-picklist.png)

### <a name="retrieve-constant-value-for-emailtype"></a>Получение постоянного значения для emailType

1. В центре администрирования SuccessFactors найдите и откройте раздел *Picklist Center* (Центр полей выбора). 
1. Используйте имя поля выбора типа адреса электронной почты, полученное при выполнении инструкций предыдущего раздела (например, ecEmailType), чтобы найти это поле выбора. 

   > [!div class="mx-imgBorder"]
   > ![Поиск поля выбора типа адреса электронной почты](./media/sap-successfactors-inbound-provisioning/find-email-type-picklist.png)

1. Откройте активное поле выбора типа адреса электронной почты. 

   > [!div class="mx-imgBorder"]
   > ![Открытие поля выбора типа адреса электронной почты](./media/sap-successfactors-inbound-provisioning/open-active-email-type-picklist.png)

1. На странице выбора типа адреса электронной почты выберите тип *Business* (Рабочий).

   > [!div class="mx-imgBorder"]
   > ![Выбор типа адреса электронной почты Business (Рабочий)](./media/sap-successfactors-inbound-provisioning/select-business-email-type.png)

1. Запишите значение **Option ID** (Идентификатор варианта), связанное с адресом электронной почты *Business* (Рабочий). Это код, который будет использоваться с *emailType* в таблице сопоставления атрибутов.

   > [!div class="mx-imgBorder"]
   > ![Получение кода типа адреса электронной почты](./media/sap-successfactors-inbound-provisioning/get-email-type-code.png)

   > [!NOTE]
   > В скопированном значении удалите символ запятой. Например, если параметр **Option ID** (Идентификатор варианта) имеет значение *8,448*, задайте в качестве значения *emailType* в Azure AD постоянное число *8448* (без символа запятой). 

### <a name="retrieve-constant-value-for-phonetype"></a>Получение постоянного значения для phoneType

1. В центре администрирования SuccessFactors найдите и откройте раздел *Picklist Center* (Центр полей выбора). 
1. Используйте имя поля выбора типа телефона, полученное при выполнении инструкций предыдущего раздела, чтобы найти это поле выбора. 

   > [!div class="mx-imgBorder"]
   > ![Поиск поля выбора типа телефона](./media/sap-successfactors-inbound-provisioning/find-phone-type-picklist.png)

1. Откройте активное поле выбора типа телефона. 

   > [!div class="mx-imgBorder"]
   > ![Открытие поля выбора типа телефона](./media/sap-successfactors-inbound-provisioning/open-active-phone-type-picklist.png)

1. На странице поля выбора типа телефона просмотрите различные типы телефонов, перечисленные в разделе **Picklist Values** (Значения поля выбора).

   > [!div class="mx-imgBorder"]
   > ![Проверка типов телефонов](./media/sap-successfactors-inbound-provisioning/review-phone-types.png)

1. Запишите значение **Option ID** (Идентификатор варианта), связанное с номером телефона *Business* (Рабочий). Это код, который будет использоваться с *businessPhoneType* в таблице сопоставления атрибутов.

   > [!div class="mx-imgBorder"]
   > ![Получение кода рабочего телефона](./media/sap-successfactors-inbound-provisioning/get-business-phone-code.png)

1. Запишите значение **Option ID** (Идентификатор варианта), связанное с номером телефона *Cell* (Мобильный). Это код, который будет использоваться с *cellPhoneType* в таблице сопоставления атрибутов.

   > [!div class="mx-imgBorder"]
   > ![Получение кода мобильного телефона](./media/sap-successfactors-inbound-provisioning/get-cell-phone-code.png)

   > [!NOTE]
   > В скопированном значении удалите символ запятой. Например, если параметр **Option ID** (Идентификатор варианта) имеет значение *10,606*, задайте в качестве значения *cellPhoneType* в Azure AD постоянное число *10606* (без символа запятой). 


## <a name="configuring-successfactors-writeback-app"></a>Настройка приложения обратной записи в SuccessFactors

В этом разделе описывается последовательность выполнения перечисленных ниже действий. 

* [Добавление приложения соединителя подготовки и настройка возможности подключения к SuccessFactors](#part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors)
* [Настройка сопоставлений атрибутов](#part-2-configure-attribute-mappings)
* [Включение и запуск подготовки пользователей](#enable-and-launch-user-provisioning)

### <a name="part-1-add-the-provisioning-connector-app-and-configure-connectivity-to-successfactors"></a>Часть 1. Добавление приложения соединителя подготовки и настройка возможности подключения к SuccessFactors

**Чтобы настроить обратную запись в SuccessFactors, выполните следующие действия.**

1. Перейдите на страницу <https://portal.azure.com>.

2. В области навигации слева выберите **Azure Active Directory**.

3. Щелкните **Корпоративные приложения**, а затем — **Все приложения**.

4. Выберите **Добавить приложение** и выберите категорию **Все**.

5. Выполните поиск по условию **Обратная запись в SuccessFactors** и добавьте это приложение из коллекции.

6. После добавления приложения и отображения экрана сведений о приложений выберите **Подготовка**.

7. Для параметра **Режим** **подготовки к работе** выберите значение **Авто**.

8. В разделе **Учетные данные администратора** заполните поля следующим образом.

   * **Имя пользователя администратора**. Введите имя пользователя учетной записи API SuccessFactors, добавив к нему идентификатор компании. Формат: **имя_пользователя\@идентификатор_компании**.

   * **Пароль администратора**. Введите пароль учетной записи пользователя API SuccessFactors. 

   * **URL-адрес клиента**. Введите имя конечной точки служб API-интерфейса OData для SuccessFactors. Вводить следует только имя узла сервера без http или https. Это значение должно выглядеть следующим образом: `api4.successfactors.com`.

   * **Адрес электронной почты для уведомлений —** введите адрес электронной почты и установите флажок "send email if failure occurs" (Отправлять по электронной почте в случае сбоя).
    > [!NOTE]
    > Служба подготовки Azure AD отправляет уведомление по электронной почте, если задание подготовки переходит в состояние [Карантин](../app-provisioning/application-provisioning-quarantine-status.md).

   * Нажмите кнопку **Проверить подключение**. Если проверка подключения выполнена успешно, нажмите кнопку **Сохранить** в верхней части. В случае неудачи дополнительно проверьте правильность учетных данных и URL-адреса SuccessFactors.
    >[!div class="mx-imgBorder"]
    >![Портал Azure](./media/sap-successfactors-inbound-provisioning/sfwb-provisioning-creds.png)

   * После успешного сохранения учетных данных в разделе **Сопоставления** отобразится сопоставление по умолчанию. Если сопоставления атрибутов не отображаются, обновите страницу.  

### <a name="part-2-configure-attribute-mappings"></a>Часть 2. Настройка сопоставлений атрибутов

В этом разделе описана настройка потоков данных пользователя из SuccessFactors в Active Directory.

1. На вкладке "Подготовка" в разделе **Сопоставления** щелкните **Provision Azure Active Directory Users** (Подготовка пользователей Azure Active Directory).

1. В поле **Область исходного объекта** можно выбрать, какие наборы пользователей в Azure AD должны учитываться при обратной записи, определив набор фильтров на основе атрибутов. Область по умолчанию — "все пользователи в Azure AD". 
   > [!TIP]
   > При первой настройке приложения подготовки необходимо проверить сопоставления атрибутов и выражения, чтобы получить желаемый результат. Корпорация Майкрософт рекомендует использовать фильтры области действия в разделе **Область исходного объекта** для проверки сопоставлений с несколькими тестовыми пользователями из Azure AD. После проверки работоспособности сопоставлений можно либо удалить фильтр, либо постепенно расширять его, добавляя больше пользователей.

1. Поле **Действия с целевыми объектами** поддерживает только операцию **Обновление**.

1. В таблице сопоставления в разделе **Сопоставления атрибутов** можно сопоставить с SuccessFactors перечисленные ниже атрибуты Azure Active Directory. В следующей таблице приведены инструкции по сопоставлению атрибутов обратной записи. 

   | \# | Атрибут Azure AD | Атрибут SuccessFactors | Комментарии |
   |--|--|--|--|
   | 1 | employeeId | personIdExternal | По умолчанию этот атрибут является сопоставляемым идентификатором. Вместо employeeId можно использовать любой другой атрибут Azure AD, который может хранить значение, равное personIdExternal в SuccessFactors.    |
   | 2 | mail | email | Сопоставление источника атрибута адреса электронной почты. В целях тестирования userPrincipalName можно сопоставить с электронной почтой. |
   | 3 | 8448 | emailType | Это постоянное значение является идентификатором SuccessFactors, связанным с рабочим адресом электронной почтой. Замените это значение соответствующим значением для своей среды SuccessFactors. Инструкции по его настройке см. в разделе [Получение постоянного значения для emailType](#retrieve-constant-value-for-emailtype). |
   | 4 | true | emailIsPrimary | Используйте этот атрибут, чтобы задать рабочий адрес электронной почты в качестве основного в SuccessFactors. Если рабочий адрес электронной почты не является основным, установите для этого флага значение false. |
   | 5 | userPrincipalName | [custom01–custom15] | С помощью функции **Добавить новое сопоставление** можно дополнительно записать userPrincipalName или любой атрибут Azure AD в настраиваемый атрибут, доступный в объекте пользователя SuccessFactors.  |
   | 6 | On Prem SamAccountName | username | С помощью функции **Добавить новое сопоставление** можно при необходимости сопоставить локальный атрибут samAccountName с атрибутом username в SuccessFactors. Для синхронизации samAccountName с Azure AD используйте инструкции из статьи [Синхронизация Azure AD Connect: расширения каталогов](../hybrid/how-to-connect-sync-feature-directory-extensions.md). Атрибут отобразится в раскрывающемся списке исходных атрибутов в таком виде: *extension_yourTenantGUID_samAccountName*. |
   | 7 | Единый вход | loginMethod | Если клиент SuccessFactors настроен для [частично единого входа](https://apps.support.sap.com/sap/support/knowledge/en/2320766), с помощью функции "Добавить новое сопоставление" можно при необходимости задать для loginMethod постоянное значение "SSO" или "PWD". |
   | 8 | TelephoneNumber | businessPhoneNumber | Используйте это сопоставление для передачи *telephoneNumber* из Azure AD в атрибут рабочего номера телефона в SuccessFactors. |
   | 9 | 10605 | businessPhoneType | Это постоянное значение является идентификатором SuccessFactors, связанным с рабочим номером телефона. Замените это значение соответствующим значением для своей среды SuccessFactors. Инструкции по его настройке см. в разделе [Получение постоянного значения для phoneType](#retrieve-constant-value-for-phonetype). |
   | 10 | true | businessPhoneIsPrimary | Этот атрибут позволяет задать основной флаг для номера рабочего телефона. Допустимыми являются значения true и false. |
   | 11 | mobile | cellPhoneNumber | Используйте это сопоставление для передачи *telephoneNumber* из Azure AD в атрибут рабочего номера телефона в SuccessFactors. |
   | 12 | 10606 | cellPhoneType | Это постоянное значение является идентификатором SuccessFactors, связанным с мобильным номером телефона. Замените это значение соответствующим значением для своей среды SuccessFactors. Инструкции по его настройке см. в разделе [Получение постоянного значения для phoneType](#retrieve-constant-value-for-phonetype). |
   | 13 | false | cellPhoneIsPrimary | Этот атрибут позволяет задать основной флаг для номера мобильного телефона. Допустимыми являются значения true и false. |
 
1. Просмотрите и проверьте сопоставления атрибутов. 
 
    >[!div class="mx-imgBorder"]
    >![Сопоставление атрибутов обратной записи](./media/sap-successfactors-inbound-provisioning/writeback-attribute-mapping.png)

1. Нажмите кнопку **Сохранить**, чтобы сохранить сопоставление. Далее следует обновить выражения API пути JSON, чтобы использовать коды phoneType в экземпляре SuccessFactors. 
1. Выберите **Показать расширенные параметры**. 

    >[!div class="mx-imgBorder"]
    >![Отображение дополнительных параметров](./media/sap-successfactors-inbound-provisioning/show-advanced-options.png)

1. Щелкните **Изменить список атрибутов для SuccessFactors**. 

   > [!NOTE] 
   > Если параметр **Изменить список атрибутов для SuccessFactors** не отображается на портале Azure, используйте для доступа к странице URL-адрес *https://portal.azure.com/?Microsoft_AAD_IAM_forceSchemaEditorEnabled=true* . 

1. В этом представлении в столбце **Выражение API** отображаются используемые соединителем выражения пути JSON. 
1. Обновите выражения пути JSON для рабочего и мобильного телефона, чтобы в них использовалось значение идентификатора (*businessPhoneType* и *cellPhoneType*), соответствующее вашей среде. 

    >[!div class="mx-imgBorder"]
    >![Изменение пути JSON для телефона](./media/sap-successfactors-inbound-provisioning/phone-json-path-change.png)

1. Нажмите кнопку **Сохранить**, чтобы сохранить сопоставление.

## <a name="enable-and-launch-user-provisioning"></a>Включение и запуск подготовки пользователей

После завершения настройки приложения подготовки SuccessFactors можно включить службу подготовки на портале Azure.

> [!TIP]
> По умолчанию при включении службы подготовки она инициализирует операции подготовки для всех пользователей в области. При наличии ошибок в сопоставлении или проблем с данными задание подготовки может завершиться ошибкой и перейти в состояние карантина. Чтобы избежать этого, рекомендуется настроить фильтр **Область исходного объекта** и протестировать сопоставления атрибутов с несколькими тестовыми пользователями перед запуском полной синхронизации всех пользователей. После проверки работоспособности сопоставлений и получения желаемых результатов можно либо удалить фильтр, либо постепенно расширять его, добавляя больше пользователей.

1. На вкладке **Подготовка** установите для параметра **Состояние подготовки** значение **Вкл**.

1. Выберите **Область**. Вы можете выбрать один из следующих параметров: 
   * **Синхронизация всех пользователей и групп**. Выберите этот параметр, если вы планируете выполнять обратную запись сопоставленных атрибутов всех пользователей из Azure AD в SuccessFactors в соответствии с правилами области, определенными в поле **Сопоставления** -> **Область исходного объекта**. 
   * **Синхронизация только назначенных пользователей и групп**. Выберите этот параметр, если вы планируете выполнять обратную запись сопоставленных атрибутов только для пользователей, назначенных этому приложению с помощью параметра меню **Приложение** -> **Управление** -> **Пользователи и группы**. На этих пользователей также распространяются правила области, определенные в поле **Сопоставления** -> **Область исходного объекта**.

   > [!div class="mx-imgBorder"]
   > ![Выбор области для приложения Writeback](./media/sap-successfactors-inbound-provisioning/select-writeback-scope.png)

   > [!NOTE]
   > Приложение подготовки обратной записи в SuccessFactors не поддерживает "назначение групп". Поддерживается только "назначение пользователей". 

1. Выберите команду **Сохранить**.

1. Будет запущена начальная синхронизация, продолжительность которой может варьироваться в зависимости от количества пользователей в клиенте Azure AD и определенной для операции области. Ход выполнения цикла синхронизации можно проверить на индикаторе выполнения для отслеживания. 

1. В любой момент можно проверить вкладку **Журналы подготовки** на портале Azure, чтобы узнать, какие действия выполнила служба подготовки. В журналах подготовки перечислены все отдельные события синхронизации, выполняемые службой подготовки. 

1. После завершения первичной синхронизации на вкладке **Подготовка** будет создан сводный отчет аудита, как показано ниже.

   > [!div class="mx-imgBorder"]
   > ![Индикатор выполнения подготовки](./media/sap-successfactors-inbound-provisioning/prov-progress-bar-stats.png)

## <a name="supported-scenarios-known-issues-and-limitations"></a>Поддерживаемые сценарии, известные проблемы и ограничения

См. раздел [Сценарии обратной записи](../app-provisioning/sap-successfactors-integration-reference.md#writeback-scenarios) в справочном руководстве по интеграции с SAP SuccessFactors. 

## <a name="next-steps"></a>Дальнейшие шаги

* [Изучите справочник по интеграции Azure AD и SAP SuccessFactors](../app-provisioning/sap-successfactors-integration-reference.md)
* [Сведения о просмотре журналов и получении отчетов о действиях по подготовке](../app-provisioning/check-status-user-account-provisioning.md)
* [Узнайте, как настроить единый вход между SuccessFactors и Azure Active Directory](successfactors-tutorial.md)
* [Узнайте, как интегрировать другие приложения SaaS с Azure Active Directory](tutorial-list.md).
* [Узнайте, как экспортировать и импортировать конфигурации подготовки.](../app-provisioning/export-import-provisioning-configuration.md)