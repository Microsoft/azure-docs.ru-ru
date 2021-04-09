---
title: Руководство по интеграции единого входа Azure Active Directory с Oracle PeopleSoft — Protected by F5 BIG-IP APM | Документация Майкрософт
description: Узнайте, как настроить единый вход между Azure Active Directory и Oracle PeopleSoft — Protected by F5 BIG-IP APM.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 09/14/2020
ms.author: jeedes
ms.openlocfilehash: 3b7c8e024ac8361c08cc41195531a114bb12fcb4
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92522297"
---
# <a name="tutorial-azure-active-directory-single-sign-on-sso-integration-with-oracle-peoplesoft---protected-by-f5-big-ip-apm"></a>Руководство по интеграции единого входа Azure Active Directory с Oracle PeopleSoft — Protected by F5 BIG-IP APM

В этом учебнике описано, как интегрировать Oracle PeopleSoft — Protected by F5 BIG-IP APM с Azure Active Directory (Azure AD). Интеграция Oracle PeopleSoft — Protected by F5 BIG-IP APM с Azure AD обеспечивает следующие возможности:

* Управление доступом к Oracle PeopleSoft — Protected by F5 BIG-IP APM с помощью Azure AD.
* Автоматический вход пользователей в Oracle PeopleSoft — Protected by F5 BIG-IP APM с помощью учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.
* В этом учебнике рассматриваются инструкции по Oracle PeopleSoft ELM.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

1. Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
1. Подписка Oracle PeopleSoft — Protected by F5 BIG-IP APM с поддержкой единого входа.

1. Для развертывания совместного решения требуется следующая лицензия:

    1. Пакет F5 BIG-IP® Best (или) 
    2. отдельная лицензия F5 BIG-IP Access Policy Manager™ (APM). 
    3. Дополнительная лицензия F5 BIG-IP Access Policy Manager™ (APM) на имеющийся BIG-IP F5 BIG-IP® Local Traffic Manager™ (LTM).
    4. В дополнение к указанной выше лицензии система F5 также может быть лицензирована с помощью: 
        * подписки на фильтрацию URL-адресов для использования базы данных категории URL-адресов; 
        * подписки на F5 IP Intelligence для обнаружения и блокирования известных злоумышленников и вредоносного трафика; 
        * аппаратного сетевого модуля безопасности (HSM) для защиты цифровых ключей и управления ими с целью обеспечения надежной проверки подлинности.
1. Система F5 BIG-IP подготавливается с модулями APM (LTM является необязательным). 
1. Хотя это необязательно, настоятельно рекомендуется развернуть системы F5 в [группе устройств для синхронизации и отработки отказа](https://techdocs.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/big-ip-device-service-clustering-administration-14-1-0.html), которая включает в себя активную резервную пару с плавающим IP-адресом для обеспечения высокой доступности. Дальнейшая избыточность интерфейса реализуется с помощью протокола LACP. LACP управляет подключенными физическими интерфейсами как одним виртуальным интерфейсом (статистической группой) и обнаруживает все сбои интерфейса в группе.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Oracle PeopleSoft — Protected by F5 BIG-IP APM поддерживает вход, инициированный **поставщиком услуг или поставщиком удостоверений**.

## <a name="adding-oracle-peoplesoft---protected-by-f5-big-ip-apm-from-the-gallery"></a>Добавление Oracle PeopleSoft — Protected by F5 BIG-IP APM из коллекции

Чтобы настроить интеграцию Oracle PeopleSoft — Protected by F5 BIG-IP APM с Azure AD, необходимо добавить Oracle PeopleSoft — Protected by F5 BIG-IP APM из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** введите **Oracle PeopleSoft — Protected by F5 BIG-IP APM** в поле поиска.
1. Выберите **Oracle PeopleSoft — Protected by F5 BIG-IP APM** в области результатов и добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.


## <a name="configure-and-test-azure-ad-sso-for-oracle-peoplesoft---protected-by-f5-big-ip-apm"></a>Настройка и проверка единого входа Azure AD для Oracle PeopleSoft — Protected by F5 BIG-IP APM

Настройте и проверьте единый вход Azure AD в Oracle PeopleSoft — Protected by F5 BIG-IP APM с помощью тестового пользователя **B. Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Oracle PeopleSoft — Protected by F5 BIG-IP APM.

Чтобы настроить и проверить единый вход Azure AD в Oracle PeopleSoft — Protected by F5 BIG-IP APM, выполните действия из следующих стандартных блоков:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Oracle PeopleSoft — Protected by F5 BIG-IP APM](#configure-oracle-peoplesoft-protected-by-f5-big-ip-apm-sso)** необходима для настройки параметров единого входа на стороне приложения.
    1. **[Создание тестового пользователя в Oracle PeopleSoft — Protected by F5 BIG-IP APM](#create-oracle-peoplesoft-protected-by-f5-big-ip-apm-test-user)** требуется для того, чтобы в Oracle PeopleSoft — Protected by F5 BIG-IP APM существовал пользователь B. Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Oracle PeopleSoft — Protected by F5 BIG-IP APM** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок "Изменить" (значок пера), чтобы открыть диалоговое окно **Базовая конфигурация SAML** и изменить параметры.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. Если вы хотите настроить приложение в режиме, инициируемом **поставщиком удостоверений**, в разделе **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор** введите URL-адрес в формате `https://<FQDN>.peoplesoft.f5.com`.

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес в формате `https://<FQDN>.peoplesoft.f5.com/saml/sp/profile/post/acs`.

    c. В текстовом поле **URL-адрес выхода** введите URL-адрес в следующем формате: `https://<FQDN>.peoplesoft.f5.com/saml/sp/profile/redirect/slr`

1. Чтобы настроить приложение для работы в режиме, инициируемом **поставщиком услуг**, щелкните **Задать дополнительные URL-адреса** и выполните следующие действия.

    В текстовом поле **URL-адрес входа** введите URL-адрес в формате `https://<FQDN>.peoplesoft.f5.com/`.

    > [!NOTE]
    >Эти значения приведены для примера. Замените эти значения фактическим идентификатором, URL-адресом ответа, URL-адресом для входа и URL-адресом для выхода. Чтобы получить значение, обратитесь в [группу поддержки клиентов Oracle PeopleSoft — Protected by F5 BIG-IP APM](https://support.f5.com). Можно также посмотреть шаблоны в разделе **Базовая конфигурация SAML** на портале Azure.

1. Приложение Oracle PeopleSoft — Protected by F5 BIG-IP APM ожидает проверочные утверждения SAML в определенном формате, который требует добавить сопоставления настраиваемых атрибутов в вашу конфигурацию атрибутов токена SAML. На следующем снимке экрана показан список атрибутов по умолчанию.

    ![Изображение](common/default-attributes.png)

1. В дополнение к описанному выше приложение Oracle PeopleSoft — Protected by F5 BIG-IP APM ожидает в ответе SAML несколько дополнительных атрибутов, которые показаны ниже. Эти атрибуты также заранее заполнены, но вы можете изменить их в соответствии со своими требованиями.
    
    | Имя |  Исходный атрибут|
    | ------------------ | --------- |
    | EMPLID | user.employeeid |

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** скачайте файл **XML-файл метаданных федерации** и **Сертификат (Base64)** и сохраните их на компьютере.

    ![Ссылка для скачивания сертификата](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/both-certificate.png)

### <a name="create-an-azure-ad-test-user"></a>Создание тестового пользователя Azure AD

В этом разделе описано, как на портале Azure создать тестового пользователя с именем B.Simon.

1. На портале Azure в области слева выберите **Azure Active Directory**, **Пользователи**, а затем — **Все пользователи**.
1. В верхней части экрана выберите **Новый пользователь**.
1. В разделе **Свойства пользователя** выполните следующие действия.
   1. В поле **Имя** введите `B.Simon`.  
   1. В поле **Имя пользователя** введите username@companydomain.extension. Например, `B.Simon@contoso.com`.
   1. Установите флажок **Показать пароль** и запишите значение, которое отображается в поле **Пароль**.
   1. Нажмите кнопку **Создать**.

### <a name="assign-the-azure-ad-test-user"></a>Назначение тестового пользователя Azure AD

В этом разделе описано, как предоставить пользователю B. Simon доступ к Oracle PeopleSoft — Protected by F5 BIG-IP APM, чтобы он мог использовать единый вход Azure.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Oracle PeopleSoft — Protected by F5 BIG-IP APM**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-oracle-peoplesoft-protected-by-f5-big-ip-apm-sso"></a>Настройка единого входа для Oracle PeopleSoft — Protected by F5 BIG-IP APM

### <a name="f5-saml-sp-configuration"></a>Конфигурации поставщика служб SAML в F5

Импортируйте в F5 сертификат метаданных, который будет использоваться позже в процессе настройки. Выберите **System > Certificate Management > Traffic Certificate Management > SSL Certificate List** (Система > Управление сертификатами > Управление сертификатами трафика > Список SSL-сертификатов). В правом верхнем углу нажмите кнопку **Import** (Импортировать).

![Конфигурации поставщика служб SAML в F5](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sp-configuration.png)

#### <a name="setup-the-saml-idp-connector"></a>Настройка соединителя поставщика удостоверений SAML 

1. Выберите **Access > Federation > SAML: Service Provider > External Idp Connectors** (Доступ > Федерация > SAML: поставщик служб > Внешние соединители поставщика удостоверений) и щелкните **Create > From Metadata** (Создать > Из метаданных).

    ![Соединитель поставщика удостоверений SAML в F5](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/saml-idp-connector.png)

1. На следующей странице нажмите кнопку **Browse** (Обзор), чтобы отправить XML-файл.

1. В текстовом поле **Identity Provider Name** (Имя поставщика удостоверений) введите действительное имя, а затем нажмите кнопку **OK**.

    ![Новый соединитель поставщика удостоверений SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/new-saml-idp.png)

1. Выполните необходимые действия на вкладке **Security Settings** (Параметры безопасности), а затем нажмите кнопку **OK**.

    ![Изменение соединителя поставщика удостоверений SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/edit-saml-idp.png)

#### <a name="setup-the-saml-sp"></a>Настройка поставщика служб SAML

1. Выберите **Access > Federation > SAML Service Provider > Local SP Services** (Доступ > Федерация > Поставщик служб SAML > Локальные службы поставщика удостоверений) и щелкните **Create** (Создать). Введите следующие сведения и нажмите кнопку **ОК**.

    * Имя: `<Name>`
    * Идентификатор сущности: `https://<FQDN>.peoplesoft.f5.com`
    * Параметры имени поставщика служб.
        * Схема: `https`
        * Host: `<FQDN>.peoplesoft.f5.com`
        * Описание: `<Description>`

    ![Создание служб поставщика удостоверений SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/new-saml-sp-service.png)

1. Выберите конфигурацию поставщика служб, PeopleSoftAppSSO, и щелкните **Bind/UnBind IdP Connectors** (Привязать/отменить привязку соединителей IdP).
Щелкните **Add New Row** (Добавить новую строку) и выберите **External IdP connector** (Внешний соединитель IdP), созданный на предыдущем шаге, щелкните **Update** (Обновить), а затем нажмите кнопку **ОК**.

    ![Создание служб поставщика удостоверений SAML](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/edit-saml-idp-use-sp.png)

## <a name="configuring-application"></a>Настройка приложения

### <a name="create-a-new-pool"></a>Создание пула
1. Выберите **Local Traffic > Pools > Pool List** (Локальный трафик > Пулы > Список пулов), щелкните **Create** (Создать), введите следующие сведения и нажмите кнопку **Finished** (Завершено).

    * Имя: `<Name>`
    * Описание: `<Description>`
    * Мониторы работоспособности: `http`
    * Адрес: `<Address>`
    * Порт службы: `<Service Port>`

    ![Создание пула](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/create-pool.png)


### <a name="create-a-new-client-ssl-profile"></a>Создание профиля SSL клиента

Выберите **Local Traffic > Profiles > SSL > Client > +** (Локальный трафик > Профили > SSL > Клиент > +), введите следующие сведения и нажмите кнопку **Finished** (Завершено).

* Имя: `<Name>`
* Сертификат: `<Certificate>`
* Ключ: `<Key>`.

![Создание профиля SSL клиента](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/client-ssl-profile.png)

### <a name="create-a-new-virtual-server"></a>Создание виртуального сервера

1. Выберите **Local Traffic > Virtual Servers > Virtual Server List > +** (Локальный трафик > Виртуальные серверы > Список виртуальных серверов > +), введите следующие сведения и нажмите кнопку **Finished** (Завершено).
    * Имя: `<Name>`
    * Адрес назначения или маска: `<Address>`
    * Порт службы: порт 443 HTTPS.
    * Профиль HTTP (клиент): HTTP

    ![Создание виртуального сервера](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/virtual-server-list.png)

1. Заполните следующие значения на указанной ниже странице:

    * Профиль SSL (клиент): `<SSL Profile>`
    * Преобразование сетевых адресов: Автоматическое сопоставление
    * Профиль доступа: `<Access Profile>`
    * Пул по умолчанию: `<Pool>`


    ![Создание пула виртуального сервера PeopleSoft](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/virtual-server-people-soft.png)

## <a name="setting-up-peoplesoft-application-to-support-f5-big-ip-apm-as-the-single-sign-on-solution"></a>Настройка приложения PeopleSoft для поддержки F5 Big-IP APM в качестве решения единого входа

>[!Note]
> Справочник: https://docs.oracle.com/cd/E12530_01/oam.1014/e10356/people.htm

1. Войдите в PeopleSoft Console `https://<FQDN>.peoplesoft.f5.com/:8000/psp/ps/?cmd=start`, используя учетные данные администратора (например, PS/PS)

    ![Портал самообслуживания Service Manager](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-console.png)

1. В приложении PeopleSoft создайте **OAMPSFT** в качестве нового профиля пользователя и свяжите роль безопасности низкого уровня, например **PeopleSoft User** (Пользователь PeopleSoft).
Выберите **Peopletools > Security > User Profiles > User Profiles** (Peopletools > Безопасность > Профили пользователей > Профили пользователей), чтобы создать профиль пользователя, например **OAMPSFT**, и добавьте **пользователя Peoplesoft**.

    ![Пользователь Peoplesoft](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/user-profile.png)

1. Получите доступ к веб-профилю и введите **OAMPSFT** в качестве **идентификатора пользователя** для общего доступа.

    ![Профили пользователей](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/web-profile-configuration.png)

1. В **конструкторе приложений PeopleTools** откройте запись **FUNCLIB_LDAP**.

    ![Конфигурация веб-профиля](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/funclib.png)

1. Замените заголовок пользователя на **PS_SSO_UID** для функции **OAMSSO_AUTHENTICATION**.
В функции **getWWWAuthConfig()** замените значение, назначенное **&defaultUserId**, на **OAMPSFT**, которое мы определили в веб-профиле. Сохраните определение записи.

    ![OAMSSO_AUTHENTICATION](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/record.png)

1. Откройте страницу **Signon PeopleCode** (Выберите PeopleTools > Security > Security Objects > Signon PeopleCode) (PeopleTools > Безопасность > Объекты безопасности > Signon PeopleCode) и активируйте функцию **OAMSSO_AUTHENTICATION** — Signon PeopleCode для единого входа Oracle Access Manager.

    ![OAMSSO_AUTHENTICATION ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sign-on-people-soft.png)

## <a name="setting-up-f5-big-ip-apm-to-populate-ps_sso_uid-http-header-with-the-peoplesoft-user-id"></a>Настройка F5 Big-IP APM для заполнения заголовка HTTP PS_SSO_UID идентификатором пользователя PeopleSoft

### <a name="configuring-per-request-policy"></a>Настройка политики для каждого запроса
1. Выберите **Access > Profile/Policies > Per-Request Policies** (Доступ > Профиль/политики > Политики для каждого запроса), щелкните **Create** (Создать), введите следующие сведения и нажмите кнопку **Finished** (Завершено).

    * Имя: `<Name>`
    * Тип профиля: All
    * Языки: `<Language>`

    ![Настройка политики для каждого запроса ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/per-request.png)

1. Щелкните **Edit** (Изменить), чтобы изменить политику для каждого запроса `<Name>`. ![Изменение политики для каждого запроса PeopleSoftSSO](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-sso.png)

    `Header Name: <Header Name>`   
    `Header Value: <Header Value>`

### <a name="assign-per-request-policy-to-the-virtual-server"></a>Назначение виртуальному серверу политики для каждого запроса

Выберите **Local Traffic > Virtual Servers > Virtual Server List > PeopleSoftApp** (Локальный трафик > Виртуальные серверы > Список виртуальных серверов > PeopleSoftApp). Укажите `<Name>` в качестве политики для каждого запроса.

![PeopleSoftSSO в качестве политики для каждого запроса ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/people-soft-sso-1.png)

## <a name="setting-up-f5-big-ip-apm-to-support-single-logout-from-peoplesoft-application"></a>Настройка F5 Big-IP APM для поддержки единого выхода из приложения PeopleSoft

Чтобы добавить поддержку единого выхода для всех пользователей PeopleSoft, выполните следующие действия.

1. Определите правильный URL-адрес выхода для портала PeopleSoft.
    * Чтобы определить адрес, используемый приложением PeopleSoft для завершения сеанса пользователя, необходимо открыть портал в веб-браузере и включить средства отладки браузера, как показано в следующем примере:

        ![URL-адрес выхода для портала PeopleSoft ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/sign-out.png)

    * Найдите элемент с идентификатором `PT_LOGOUT_MENU` и сохраните путь URL-адреса с параметрами запроса. В нашем примере мы получили следующее значение: `/psp/ps/?cmd=logout`

1. Создайте LTM iRule для перенаправления пользователя на URL-адрес выхода APM: `/my.logout.php3`.

    * Выберите **Local Traffic > iRule** (Локальный трафик > iRule), щелкните **Create** (Создать), введите следующие сведения и нажмите кнопку **Finished** (Завершено).

        Имя: `<Name>`  
        Определение:  
                    _when HTTP_REQUEST { switch -glob -- [HTTP::uri] { `/psp/ps/?cmd=logout` { HTTP::redirect `/my.logout.php3` } } }_

1. Назначьте созданный iRule виртуальному серверу.

    * Выберите **Local Traffic > Virtual Servers > Virtual Server List > PeopleSoftApp > Resources** (Локальный трафик > Виртуальные серверы > Список виртуальных серверов > PeopleSoftApp > Ресурсы). Нажмите кнопку **Manage…** (Управление…).   

    * Укажите `<Name>` в качестве включенного iRule и нажмите кнопку **Finished** (Завершено).

        ![_iRule_PeopleSoftApp ](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/irule-people-soft.png)

    * Введите в текстовое поле **Name** (Имя) значение `<Name>`. 

        ![_iRule_PeopleSoftApp завершено](./media/oracle-peoplesoft-protected-by-f5-big-ip-apm-tutorial/common-irule.png)


### <a name="create-oracle-peoplesoft-protected-by-f5-big-ip-apm-test-user"></a>Создание тестового пользователя в Oracle PeopleSoft — Protected by F5 BIG-IP APM

В этом разделе описано, как создать пользователя B. Simon в Oracle PeopleSoft — Protected by F5 BIG-IP APM. Обратитесь в [группу поддержки Oracle PeopleSoft — Protected by F5 BIG-IP APM](https://support.f5.com), чтобы добавить пользователей на платформу Oracle PeopleSoft — Protected by F5 BIG-IP APM. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов. 

#### <a name="sp-initiated"></a>Инициация поставщиком услуг:

* Выберите **Тестировать приложение** на портале Azure. Вы будете перенаправлены по URL-адресу для входа в Oracle PeopleSoft — Protected by F5 BIG-IP APM, где можно инициировать поток входа.  

* Перейдите по URL-адресу для входа в Oracle PeopleSoft — Protected by F5 BIG-IP APM и инициируйте поток входа.

#### <a name="idp-initiated"></a>Вход, инициированный поставщиком удостоверений

* Выберите **Тестировать приложение** на портале Azure, и вы автоматически войдете в Oracle PeopleSoft — Protected by F5 BIG-IP APM, для которого настроен единый вход. 

Вы можете также использовать Панель доступа корпорации Майкрософт для тестирования приложения в любом режиме. Щелкнув плитку Oracle PeopleSoft — Protected by F5 BIG-IP APM на Панели доступа, вы будете перенаправлены на страницу входа приложения, если настроено для использования в режиме поставщика услуг, или автоматически войдете в приложение Oracle PeopleSoft — Protected by F5 BIG-IP APM, для которого настроен единый вход, если настроено для использования в режиме поставщика удостоверений. См. дополнительные сведения о [панели доступа](../user-help/my-apps-portal-end-user-access.md)


## <a name="next-steps"></a>Дальнейшие действия

После настройки Oracle PeopleSoft — Protected by F5 BIG-IP APM вы можете применить управление сеансами, которое защищает от хищения конфиденциальных данных вашей организации и несанкционированного доступа к ним в реальном времени. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).