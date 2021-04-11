---
title: Руководство по Интеграция единого входа Azure Active Directory с Kemp LoadMaster Azure AD integration | Документация Майкрософт
description: Сведения о том, как настроить единый вход Azure Active Directory для Kemp LoadMaster Azure AD integration.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 02/04/2021
ms.author: jeedes
ms.openlocfilehash: 91f6db79b7d18dc8b34ba1712d74a92000d63528
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104953686"
---
# <a name="tutorial-azure-active-directory-sso-integration-with-kemp-loadmaster-azure-ad-integration"></a>Руководство по Интеграция единого входа Azure Active Directory с Kemp LoadMaster Azure AD integration

В этом руководстве объясняется, как интегрировать приложение Kemp LoadMaster Azure AD integration с Azure Active Directory (Azure AD). Интеграция Kemp LoadMaster Azure AD integration с Azure AD обеспечивает указанные ниже возможности:

* Управление доступом к Kemp LoadMaster Azure AD integration в Azure AD.
* Автоматический вход пользователей в Kemp LoadMaster Azure AD integration с использованием учетных записей Azure AD.
* Централизованное управление учетными записями через портал Azure.

## <a name="prerequisites"></a>Предварительные требования

Чтобы приступить к работе, потребуется следующее.

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Kemp LoadMaster Azure AD integration с поддержкой единого входа.

## <a name="scenario-description"></a>Описание сценария

В рамках этого руководства вы настроите и проверите единый вход Azure AD в тестовой среде.

* Kemp LoadMaster Azure AD integration поддерживает единый вход, инициируемый **поставщиком удостоверений**.

## <a name="add-kemp-loadmaster-azure-ad-integration-from-the-gallery"></a>Добавление приложения Kemp LoadMaster Azure AD integration из коллекции

Чтобы настроить интеграцию Kemp LoadMaster Azure AD integration с Azure AD, необходимо добавить приложение Kemp LoadMaster Azure AD integration из коллекции в список управляемых приложений SaaS.

1. Войдите на портал Azure с помощью личной учетной записи Майкрософт либо рабочей или учебной учетной записи.
1. В области навигации слева выберите службу **Azure Active Directory**.
1. Перейдите в колонку **Корпоративные приложения** и выберите **Все приложения**.
1. Чтобы добавить новое приложение, выберите **Новое приложение**.
1. В разделе **Добавление из коллекции** в поле поиска введите **Kemp LoadMaster Azure AD integration**.
1. Выберите **Kemp LoadMaster Azure AD integration** в области результатов, а затем добавьте это приложение. Подождите несколько секунд, пока приложение не будет добавлено в ваш клиент.

## <a name="configure-and-test-azure-ad-sso-for-kemp-loadmaster-azure-ad-integration"></a>Настройка и проверка единого входа Azure AD для Kemp LoadMaster Azure AD integration

Настройте и проверьте единый вход Azure AD в Kemp LoadMaster Azure AD integration, используя данные тестового пользователя **B.Simon**. Чтобы обеспечить работу единого входа, необходимо установить связь между пользователем Azure AD и соответствующим пользователем в Kemp LoadMaster Azure AD integration.

Чтобы настроить и проверить единый вход Azure AD в Kemp LoadMaster Azure AD integration, выполните следующие действия:

1. **[Настройка единого входа Azure AD](#configure-azure-ad-sso)** необходима, чтобы пользователи могли использовать эту функцию.
    1. **[Создание тестового пользователя Azure AD](#create-an-azure-ad-test-user)** требуется для проверки работы единого входа Azure AD с помощью пользователя B.Simon.
    1. **[Назначение тестового пользователя Azure AD](#assign-the-azure-ad-test-user)** необходимо, чтобы позволить пользователю B.Simon использовать единый вход Azure AD.
1. **[Настройка единого входа в Kemp LoadMaster Azure AD integration](#configure-kemp-loadmaster-azure-ad-integration-sso)** необходима, чтобы настроить параметры единого входа на стороне приложения.

1. **[Публикация веб-сервера](#publishing-web-server)** :
    1. **[создание виртуальной службы](#create-a-virtual-service)** ;
    1. **[сертификаты и безопасность](#certificates-and-security)** ;
    1. **[профиль SAML для Kemp LoadMaster Azure AD integration](#kemp-loadmaster-azure-ad-integration-saml-profile)** ;
    1. **[проверка изменений](#verify-the-changes)** .
1. **[Настройка аутентификации на основе Kerberos](#configuring-kerberos-based-authentication)** :
    1. **[создание учетной записи делегирования Kerberos для Kemp LoadMaster Azure AD integration](#create-a-kerberos-delegation-account-for-kemp-loadmaster-azure-ad-integration)** ;
    1. **[KCD для Kemp LoadMaster Azure AD integration (учетные записи делегирования Kerberos)](#kemp-loadmaster-azure-ad-integration-kcd-kerberos-delegation-accounts)** ;
    1. **[ESP для Kemp LoadMaster Azure AD integration](#kemp-loadmaster-azure-ad-integration-esp)** ;
    1. **[создание тестового пользователя Kemp LoadMaster Azure AD integration](#create-kemp-loadmaster-azure-ad-integration-test-user)** требуется для того, чтобы в Kemp LoadMaster Azure AD integration существовал пользователь B.Simon, связанный с одноименным пользователем в Azure AD.
1. **[Проверка единого входа](#test-sso)** позволяет убедиться в правильности конфигурации.

## <a name="configure-azure-ad-sso"></a>Настройка единого входа Azure AD

Выполните следующие действия, чтобы включить единый вход Azure AD на портале Azure.

1. На портале Azure на странице интеграции с приложением **Kemp LoadMaster Azure AD integration** найдите раздел **Управление** и выберите **Единый вход**.
1. На странице **Выбрать метод единого входа** выберите **SAML**.
1. На странице **Настройка единого входа с помощью SAML** щелкните значок карандаша, чтобы открыть диалоговое окно **Базовая конфигурация SAML** для изменения параметров.

   ![Изменение базовой конфигурации SAML](common/edit-urls.png)

1. На странице **Базовая конфигурация SAML** введите значения следующих полей.

    а. В текстовом поле **Идентификатор (сущности)** введите URL-адрес следующим образом: `https://<KEMP-CUSTOMER-DOMAIN>.com/`

    b. В текстовом поле **URL-адрес ответа** введите URL-адрес `https://<KEMP-CUSTOMER-DOMAIN>.com/`

    > [!NOTE]
    > Эти значения приведены для примера. Измените их на фактические значения идентификатора и URL-адреса ответа. Чтобы получить эти значения, обратитесь к [группе поддержки клиентов Kemp LoadMaster Azure AD integration](mailto:support@kemp.ax). Можно также посмотреть шаблоны в разделе "Базовая конфигурация SAML" на портале Azure.

1. На странице **Настройка единого входа с помощью SAML** в разделе **Сертификат подписи SAML** найдите элемент **Сертификат (Base64)** и **XML метаданных федерации** и нажмите кнопку **Скачать**, чтобы скачать сертификат и XML-файлы метаданных сохранить их на компьютере.

    ![Ссылка для скачивания сертификата](./media/kemp-tutorial/certificate-base-64.png)

1. Требуемые URL-адреса можно скопировать из раздела **настройки Kemp LoadMaster Azure AD integration**.

    ![Копирование URL-адресов настройки](common/copy-configuration-urls.png)

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

В этом разделе объясняется, как включить единый вход Azure для пользователя B.Simon, предоставив этому пользователю доступ к Kemp LoadMaster Azure AD integration.

1. На портале Azure выберите **Корпоративные приложения**, а затем — **Все приложения**.
1. В списке приложений выберите **Kemp LoadMaster Azure AD integration**.
1. На странице "Обзор" приложения найдите раздел **Управление** и выберите **Пользователи и группы**.
1. Выберите **Добавить пользователя**, а в диалоговом окне **Добавление назначения** выберите **Пользователи и группы**.
1. В диалоговом окне **Пользователи и группы** выберите **B.Simon** в списке пользователей, а затем в нижней части экрана нажмите кнопку **Выбрать**.
1. Если пользователям необходимо назначить роль, вы можете выбрать ее из раскрывающегося списка **Выберите роль**. Если для этого приложения не настроена ни одна роль, будет выбрана роль "Доступ по умолчанию".
1. В диалоговом окне **Добавление назначения** нажмите кнопку **Назначить**.

## <a name="configure-kemp-loadmaster-azure-ad-integration-sso"></a>Настройка единого входа для Kemp LoadMaster Azure AD integration

## <a name="publishing-web-server"></a>Публикация веб-сервера 

### <a name="create-a-virtual-service"></a>Создание виртуальной службы 

1. Перейдите к веб-интерфейсу Kemp LoadMaster Azure AD integration и выберите элементы Virtual Services (Виртуальные службы) > Add New (Добавить новую).

1. Щелкните команду Add New (Добавить новую).

1. Укажите параметры для виртуальной службы:

    ![Снимок экрана: страница Please Specify the Parameters for the Virtual Service (Укажите параметры для виртуальной службы) с примерами значений в полях.](./media/kemp-tutorial/kemp-1.png)

    а. Virtual Address (Виртуальный адрес);
    
    b. Порт
    
    c. Service Name (Optional) (Имя службы, необязательно);
    
    d. Протокол 

1. Перейдите к разделу Real Servers (Реальные серверы).

1. Щелкните Add New (Добавить новый).

1. Укажите параметры для реального сервера.
    
    ![Снимок экрана: страница Please Specify the Parameters for the Real Server (Укажите параметры для реального сервера) с примерами значений в полях.](./media/kemp-tutorial/kemp-2.png)

    а. Выберите Allow Remote Addresses (Разрешить удаленные адреса).
    
    b. Введите адрес реального сервера в поле Real Server Address.
    
    c. Порт
    
    d. Forwarding method (Метод переадресации).
    
    д) Вес
    
    е) Connection Limit (Ограничение числа подключений).
    
    ж. Щелкните Add This Real Server (Добавить этот реальный сервер).

## <a name="certificates-and-security"></a>Сертификаты и безопасность
 
### <a name="import-certificate-on-kemp-loadmaster-azure-ad-integration"></a>Импорт сертификата для Kemp LoadMaster Azure AD integration 
 
1. Перейдите на веб-портал Kemp LoadMaster Azure AD integration и выберите элементы Certificates & Security (Сертификаты и безопасность) > SSL Certificates (Сертификаты SSL).

1. В разделе Manage Certificates (Управление сертификатами) выберите элемент Certificate Configuration (Настройка сертификата).

1. Щелкните команду Import Certificate (Импортировать сертификат).

1. Укажите имя файла с сертификатом. Этот файл также включает закрытый ключ. Если закрытый ключ отсутствует в файле, укажите файл с закрытым ключом. Сертификат может иметь формат PEM или PFX (IIS).

1. Щелкните команду Choose file (Выбрать файл) и выберите файл сертификата.

1. Щелкните элемент Key File (Файл ключа) (необязательно).

1. Щелкните Save (Сохранить).

### <a name="ssl-acceleration"></a>Ускорение SSL
 
1. Перейдите к веб-интерфейсу Kemp Load Master и выберите элементы Virtual Services (Виртуальные службы) > View/Modify Services (Просмотр или изменение служб).

1. Щелкните команду Modify (Изменить) в разделе Operation (Функционирование).

1. Щелкните элемент SSL Properties (Свойства SSL). Протокол SSL работает на уровне 7.
    
    ![Снимок экрана: раздел SSL Properties (Свойства SSL) с выбранным параметром SSL Acceleration - Enabled (Ускорение SSL — Включено) и выбранным примером сертификата.](./media/kemp-tutorial/kemp-3.png)
    
    а. Установите флажок Enabled (Включено) в разделе SSL Acceleration (Ускорение SSL).
    
    b. В разделе Available Certificates (Доступные сертификаты) выберите импортированный сертификат и щелкните символ `>`.
    
    c. Когда нужный SSL-сертификат появится в списке Assigned Certificates (Назначенные сертификаты), щелкните команду **Set Certificates** (Задать сертификаты).

    > [!NOTE]
    > Убедитесь, что выбрана команда **Set Certificates** (Задать сертификаты).

## <a name="kemp-loadmaster-azure-ad-integration-saml-profile"></a>Профиль SAML для Kemp LoadMaster Azure AD integration
 
### <a name="import-idp-certificate"></a>Импорт сертификата поставщика удостоверений

Перейдите к веб-консоли Kemp LoadMaster Azure AD integration. 

1. Щелкните элемент Intermediate Certificates (Промежуточные сертификаты) в разделе Certificates and Authority (Сертификаты и центры сертификации).

    ![Снимок экрана: раздел Currently installed Intermediate Certificates (Установленные на данный момент промежуточные сертификаты) с выбранным примером сертификата.](./media/kemp-tutorial/kemp-6.png)

    а. Щелкните команду Choose file (Выбрать файл) в разделе Add a new Intermediate Certificate (Добавить новый промежуточный сертификат).
    
    b. Перейдите к файлу сертификата, скачанному ранее из корпоративного приложения Azure AD.
    
    c. Щелкните команду Open (Открыть).
    
    d. Укажите имя в поле Certificate Name (Имя сертификата).
    
    д) Щелкните команду Add Certificate (Добавить сертификат).

### <a name="create-authentication-policy"></a>Создание политики проверки подлинности 
 
Выберите элемент Manage SSO (Управление единым входом) в разделе Virtual Services (Виртуальные службы).

   ![Снимок экрана: страница Manage SSO (Управление единым входом).](./media/kemp-tutorial/kemp-7.png)
   
   а. Щелкните команду Add (Добавить) в разделе Add new Client Side Configuration (Добавление новой конфигурации на стороне клиента) (после присвоения ей имени).

   b. Выберите элемент SAML в разделе Authentication Protocol (Протокол аутентификации).

   c. Выберите элемент MetaData File (Файл метаданных) в разделе IdP Provisioning (Подготовка поставщика удостоверений). 

   d. Щелкните команду Choose File (Выбрать файл).

   д) Перейдите к файлу XML, который вы скачали ранее с портала Azure.

   е) Щелкните команду Open (Открыть) и выберите элемент Import IdP MetaData file (Импортировать файл метаданных поставщика удостоверений).

   ж. Выберите промежуточный сертификат для параметра IdP Certificate (Сертификат поставщика удостоверений).

   h. Задайте значение параметра SP Entity ID (Идентификатор сущности поставщика службы), которое должно соответствовать удостоверению, созданному на портале Azure. 

   i. Щелкните команду Set SP Entity ID (Задать идентификатор сущности поставщика службы).

### <a name="set-authentication"></a>Настройка аутентификации  
 
Перейдите к веб-консоли Kemp LoadMaster Azure AD integration.

1. Щелкните элемент Virtual Services (Виртуальные службы).

1. Щелкните команду View/Modify Services (Просмотреть или изменить службы).

1. Щелкните команду Modify (Изменить) и перейдите к разделу ESP Options (Параметры ESP).
    
    ![Снимок экрана: страница View/Modify Services (Просмотр и изменение служб) с развернутыми разделами ESP Options (Параметры ESP) и Real Servers (Реальные серверы).](./media/kemp-tutorial/kemp-8.png)

    а. Установите флажок Enable ESP (Включить ESP).
    
    b. Выберите элемент SAML для Client Authentication Mode (Режим аутентификации клиента).
    
    c. Выберите ранее созданный домен аутентификации на стороне клиента в поле SSO Domain (Домен единого входа).
    
    d. Введите имя узла в поле Allowed Virtual Hosts (Разрешенные виртуальные узлы) и щелкните команду Set Allowed Virtual Hosts (Задать разрешенные виртуальные узлы).
    
    д) Введите /* в поле Allowed Virtual Directories (Разрешенные виртуальные каталоги) (на основе требований к доступу) и щелкните команду Set Allowed Directories (Задать разрешенные каталоги).

### <a name="verify-the-changes"></a>Проверка изменений 
 
Перейдите по URL-адресу приложения. 

Вместо страницы, которая ранее отображалась без аутентификации, должна отобразиться страница входа в арендатор. 

![Снимок экрана: страница входа с указанием арендатора](./media/kemp-tutorial/kemp-9.png)

## <a name="configuring-kerberos-based-authentication"></a>Настройка аутентификации на основе Kerberos 
 
### <a name="create-a-kerberos-delegation-account-for-kemp-loadmaster-azure-ad-integration"></a>Создание учетной записи делегирования Kerberos для Kemp LoadMaster Azure AD integration 

1. Создайте учетную запись пользователя (AppDelegation в нашем примере).
    
    ![Снимок экрана: окно kcd user Properties (Свойства пользователя KCD) с выбранной вкладкой Account (Учетная запись).](./media/kemp-tutorial/kemp-10.png)


    а. Перейдите на вкладку Attribute Editor (Редактор атрибутов).

    b. Перейдите к элементу servicePrincipalName. 

    c. Выберите элемент servicePrincipalName и щелкните команду Edit (Изменить).

    d. Введите http/kcduser в поле Value (Значение), чтобы добавить поле, и щелкните команду Add (Добавить). 

    д) Щелкните команду Apply (Применить) и нажмите кнопку OK. Окно должно закрыться, прежде чем вы снова откроете его (для перехода к новой вкладке Delegation (Делегирование)). 

1. Снова откройте окно свойств пользователя с уже доступной вкладкой Delegation (Делегирование). 

1. Откройте вкладку Delegation (Делегирование).

    ![Снимок экрана: окно kcd user Properties (Свойства пользователя KCD) с выбранной вкладкой Delegation (Делегирование).](./media/kemp-tutorial/kemp-11.png)

    а. Выберите параметр Trust this user for delegation to specified services only (Доверять этому пользователю делегирование только определенных служб).

    b. Выберите параметр Use any authentication protocol (Использовать любой протокол аутентификации).

    c. Добавьте реальные серверы и добавьте http в качестве типа службы. 
    
    d. Установите флажок Expanded (Расширенные). 

    д) Вы увидите все серверы с именем узла и полным доменным именем.
    
    е) Нажмите кнопку OK.

> [!NOTE]
> Задайте имя поставщика службы для приложения и (или) веб-сайта как применимое, чтобы получить доступ к приложению при заданном идентификаторе пула приложений. Чтобы получить доступ к приложению IIS с помощью полного доменного имени, перейдите к командной строке реального сервера и введите SetSpn с нужными параметрами, например Setspn –S HTTP/sescoindc.sunehes.co.in suneshes\kdcuser. 

### <a name="kemp-loadmaster-azure-ad-integration-kcd-kerberos-delegation-accounts"></a>KCD для Kemp LoadMaster Azure AD integration (учетные записи делегирования Kerberos) 

Перейдите к веб-консоли Kemp LoadMaster Azure AD integration и выберите элементы Virtual Services (Виртуальные службы) > Manage SSO (Управление единым входом).

![Снимок экрана: страница Manage SSO - Manage Domain (Управление единым входом — Управление доменом).](./media/kemp-tutorial/kemp-12.png)

а. Перейдите к разделу Server Side Single Sign On Configurations (Конфигурации единого входа на стороне сервера).

b. Введите имя в поле Add new Server-Side Configuration (Добавить новую конфигурацию на стороне сервера) и щелкните команду Add (Добавить).

c. Выберите значение Kerberos Constrained Delegation (Ограниченное делегирование Kerberos) для параметра Authentication Protocol (Протокол аутентификации).

d. Введите доменное имя в поле Kerberos Realm (Область Kerberos).

д) Щелкните команду Set Kerberos realm (Задать область Kerberos).

е) Введите IP-адрес контроллера домена в поле Kerberos Key Distribution Center (Центр дистрибуции ключей Kerberos).

ж. Щелкните команду Set Kerberos KDC (Задать центр дистрибуции ключей Kerberos).

h. Введите имя пользователя KCD в поле Kerberos Trusted User Name (Доверенное имя пользователя Kerberos).

i. Щелкните команду Set KDC trusted user name (Задать доверенное имя пользователя центра дистрибуции ключей).

j. Введите пароль в поле Kerberos Trusted User Password (Доверенный пароль пользователя Kerberos).

k. Щелкните команду Set KCD trusted user password (Задать доверенный пароль пользователя KCD).

### <a name="kemp-loadmaster-azure-ad-integration-esp"></a>ESP для Kemp LoadMaster Azure AD integration 

Выберите элементы Virtual Services (Виртуальные службы) > View/Modify Services (Просмотреть или изменить службы).

![Веб-сервер Kemp LoadMaster Azure AD integration](./media/kemp-tutorial/kemp-13.png)

а. Щелкните команду Modify (Изменить) рядом с полем Nick Name (Псевдоним) виртуальной службы.
    
b. Щелкните элемент ESP Options (Параметры ESP).
    
c. В разделе Server Authentication Mode (Режим аутентификации сервера) выберите KCD.
        
d. В разделе Server-Side configuration (Конфигурация на стороне сервера) выберите созданный ранее профиль для стороны сервера.

### <a name="create-kemp-loadmaster-azure-ad-integration-test-user"></a>Создание тестового пользователя для Kemp LoadMaster Azure AD integration

В этом разделе объясняется, как создать пользователя B.Simon в приложении Kemp LoadMaster Azure AD integration. Обратитесь в [службу технической поддержки приложения Kemp LoadMaster Azure AD integration](mailto:support@kemp.ax) для добавления пользователей на платформу этого приложения. Перед использованием единого входа необходимо создать и активировать пользователей.

## <a name="test-sso"></a>Проверка единого входа 

В этом разделе описано, как проверить конфигурацию единого входа Azure AD с помощью указанных ниже способов.

* На портале Azure щелкните "Тестировать приложение", и вы автоматически войдете в приложение Kemp LoadMaster Azure AD integration, для которого настроили единый вход.

* Вы можете использовать портал "Мои приложения" корпорации Майкрософт. Щелкнув плитку Kemp LoadMaster Azure AD integration на портале "Мои приложения", вы автоматически войдете в приложение Kemp LoadMaster Azure AD integration, для которого настроили единый вход. Дополнительные сведения о портале "Мои приложения" см. в [этой статье](../user-help/my-apps-portal-end-user-access.md).

## <a name="next-steps"></a>Дальнейшие действия

После настройки Kemp LoadMaster Azure AD integration вы можете применить управление сеансами, которое в реальном времени защищает конфиденциальные данные вашей организации от кражи и несанкционированного доступа. Управление сеансом является расширением функции условного доступа. [Узнайте, как применять управление сеансами с помощью Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-any-app).