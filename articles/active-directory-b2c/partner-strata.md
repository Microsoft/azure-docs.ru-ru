---
title: Руководство по настройке Azure Active Directory B2C с помощью Strata
titleSuffix: Azure AD B2C
description: Узнайте, как интегрировать проверку подлинности Azure AD B2C с Вхоиам для проверки пользователей
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/25/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 69355b58e36f435b5b5dbe94af72d55d9806b156
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100557176"
---
# <a name="tutorial-for-extending-azure-ad-b2c-to-protect-on-premises-applications-using-strata"></a>Руководство по расширению Azure AD B2C для защиты локальных приложений с помощью Strata

В этом примере руководства вы узнаете, как интегрировать Azure Active Directory (AD) B2C с Strata [Маверикс Identity Orchestrator](https://www.strata.io/maverics-identity-orchestrator/).
Маверикс Identity Orchestrator расширяет Azure AD B2C для защиты локальных приложений. Он подключается к любой системе идентификации, прозрачно переносит пользователей и учетные данные, синхронизирует политики и конфигурации и абстрагирует проверку подлинности и управление сеансами. Используя Strata предприятия, можно быстро переходить от устаревших к Azure AD B2C без перезаписи приложений. Это решение имеет следующие преимущества:

- **Единый Sign-On клиента (SSO) в локальные гибридные приложения**. Azure AD B2C поддерживает единый вход пользователей с помощью Маверикс Identity Orchestrator. Пользователи входят с учетными записями, размещенными в Azure AD B2C или в поставщике удостоверений социальных сетей (IdP). Маверикс расширяет единый вход для приложений, которые были исторически защищены устаревшими системами идентификации, такими как Symantec SiteMinder.

- **Расширение единого входа на основе стандартов в приложения без их перезаписи**: используйте Azure AD B2C для управления доступом пользователей и включения единого входа с соединителями SAML или OIDC для Маверикс Identity Orchestrator.

- **Простая настройка**. Azure AD B2C предоставляет простой пошаговый пользовательский интерфейс для подключения соединителей SAML Маверикс Identity ORCHESTRATOR или OIDC к Azure AD B2C.

## <a name="prerequisites"></a>Предварительные условия

Чтобы приступить к работе, вам потребуется:

- Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).

- [Azure AD B2C клиент](./tutorial-create-tenant.md) , связанный с подпиской Azure.

- Экземпляр [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) для хранения секретов, используемых Маверикс Identity Orchestrator. Он используется для подключения к Azure AD B2C или другим поставщикам атрибутов, например к каталогу или базе данных LDAP.

- Экземпляр [Маверикс Identity Orchestrator](https://www.strata.io/maverics-identity-orchestrator/) , установленный и выполняемый на виртуальной машине Azure или на выбранном локальном сервере. Сведения о том, как получить программное обеспечение и получить доступ к документации по установке и настройке, см. в [Strata](https://www.strata.io/contact/)

- Локальное приложение, которое будет перенесено из устаревшей системы удостоверений в Azure AD B2C.

## <a name="scenario-description"></a>Описание сценария

Интеграция Маверикс Strata включает следующие компоненты:

- **Azure AD B2C**: сервер авторизации, отвечающий за проверку учетных данных пользователя. Пользователи, прошедшие проверку подлинности, могут получить доступ к локальным приложениям с помощью локальной учетной записи, хранящейся в каталоге Azure AD B2C.

- **Внешний социальные или корпоративные IDP**: может быть любым поставщиком OpenID Connect Connect, Facebook, Google или GitHub. См. сведения об использовании [внешних поставщиков удостоверений](./technical-overview.md#external-identity-providers) с Azure AD B2C.  

- **Strata Маверикс Identity Orchestrator**. служба, которая управляет входом пользователя и прозрачно передает удостоверение приложениям через заголовки HTTP.

Реализация показана на следующей схеме архитектуры.

![На рисунке показана архитектура интеграции Azure AD B2C с Strata Маверикс для обеспечения доступа к гибридным приложениям.](./media/partner-strata/strata-architecture-diagram.png)

| Шаги | Описание |
|:-------|:---------------|
| 1. | Пользователь выполняет запрос на доступ к локальному размещенному приложению. Маверикс Identity Orchestrator передает запрос, сделанный пользователем, в приложение.|
| 2. | Orchestrator проверяет состояние проверки подлинности пользователя. Если он не получает маркер сеанса или указанный токен сеанса является недопустимым, он отправляет пользователю Azure AD B2C для проверки подлинности.|
| 3. | Azure AD B2C отправляет запрос на проверку подлинности в настроенный социальных IdP.|
| 4. | IdP выставит проблемы с учетными данными пользователя. В зависимости от IdP пользователю может потребоваться выполнить многофакторную проверку подлинности (MFA).|
| 5. | IdP отправляет ответ проверки подлинности обратно в Azure AD B2C. При необходимости пользователь может создать локальную учетную запись в каталоге Azure AD B2C на этом шаге.|
| 6. | Azure AD B2C отправляет запрос пользователя в конечную точку, указанную во время регистрации приложения Orchestrator в клиенте Azure AD B2C.|
| 7. | Orchestrator оценивает политики доступа и вычисляет значения атрибутов, которые будут включаться в заголовки HTTP, перенаправляемые в приложение. На этом шаге Orchestrator может обратиться к дополнительным поставщикам атрибутов, чтобы получить сведения, необходимые для правильного задания значений заголовка. Orchestrator задает значения заголовков и отправляет запрос в приложение.|
| 8. | Теперь пользователь прошел проверку подлинности и имеет доступ к приложению.|

## <a name="get-maverics-identity-orchestrator"></a>Получение Маверикс Identity Orchestrator
Чтобы получить программное обеспечение, которое будет использоваться для интеграции устаревшего локального приложения с Azure AD B2C, обратитесь в [Strata](https://www.strata.io/contact/). После получения программного обеспечения выполните следующие действия, чтобы определить необходимые компоненты Orchestrator и выполнить необходимые действия по установке и настройке.

## <a name="configure-your-azure-ad-b2c-tenant"></a>Настройка клиента Azure AD B2C

1. **Регистрация приложения**

   а. [Зарегистрируйте Orchestrator как приложение](./tutorial-register-applications.md?tabs=app-reg-ga) в Azure AD B2C клиенте.
   >[!Note]
   >Вам потребуется имя клиента и идентификатор, идентификатор клиента, секрет клиента, настроенные утверждения и URI перенаправления позже при настройке экземпляра Orchestrator.

   b. Предоставьте разрешения Microsoft MS API Graph для приложений. Приложению понадобятся следующие разрешения: `offline_access` , `openid` .

   c. Добавьте URI перенаправления для приложения. Этот URI будет соответствовать `oauthRedirectURL` параметру конфигурации соединителя Azure AD B2C Orchestrator, например `https://example.com/oidc-endpoint` .

2. **Создание потока пользователя**. Создайте пользовательский поток [регистрации и входа](./tutorial-create-user-flows.md).

3. **Добавление IDP**: выберите для входа пользователя с локальной учетной записью или социальным или корпоративным [IDP](./add-identity-provider.md).

4. **Определение атрибутов пользователя**: определите атрибуты, которые будут собираться во время регистрации.

5. **Укажите утверждения приложения**: укажите атрибуты, которые будут возвращены приложению через экземпляр Orchestrator. Orchestrator использует атрибуты из утверждений, возвращенных Azure AD B2C, и может получать дополнительные атрибуты из других подключенных систем идентификации, таких как каталоги LDAP и базы данных. Эти атрибуты задаются в заголовках HTTP и отправляются в вышестоящее локальное приложение.

## <a name="configure-maverics-identity-orchestrator"></a>Настройка Маверикс Identity Orchestrator

В следующих разделах мы рассмотрим шаги, необходимые для настройки экземпляра Orchestrator. Для получения дополнительной поддержки и документации обратитесь в [Strata](https://www.strata.io/contact/).

### <a name="maverics-identity-orchestrator-server-requirements"></a>Требования к серверу Orchestrator Identity Маверикс

Вы можете запустить экземпляр Orchestrator на любом сервере, будь то локальная или Общедоступная облачная инфраструктура, например Azure, AWS или обеспечить.

- ОС: РЕХЛ 7,7 или более поздней версии, CentOS 7 +

- Диск: 10 ГБ (маленький)

- Память: 16 ГБ

- Порты: 22 (SSH/SCP), 443, 80

- Корневой доступ для задач установки и администрирования

- Маверикс Identity Orchestrator запускается от имени пользователя `maverics` в `systemd`

- Сетевой исходящий трафик с сервера, на котором размещена Маверикс Identity Orchestrator, с возможностью доступа к клиенту Azure AD.

### <a name="install-maverics-identity-orchestrator"></a>Установка Maverics Identity Orchestrator

1. Получите последнюю версию пакета RPM Maverics. Поместите пакет в систему, в которой вы хотите установить Маверикс. Если вы копируете файл на удаленный узел, [точка подключения службы](https://www.ssh.com/ssh/scp/) является полезным инструментом.

2. Чтобы установить пакет Маверикс, выполните следующую команду, заменив имя вашего файла вместо `maverics.rpm` .

   `sudo rpm -Uvf maverics.rpm`

   По умолчанию Maverics устанавливается в каталог `/usr/local/bin`.

3. После установки Maverics будет работать как служба под `systemd`.  Чтобы проверить, работает ли служба Маверикс, выполните следующую команду:

   `sudo service maverics status`

  Если установка Orchestrator прошла успешно, вы увидите примерно следующее сообщение:

```
Redirecting to /bin/systemctl status maverics.service
  maverics.service - Maverics
  Loaded: loaded (/etc/systemd/system/maverics.service; enabled; vendor preset: disabled)
  Active: active (running) since Thu 2020-08-13 16:48:01 UTC; 24h ago
  Main PID: 330772 (maverics)
  Tasks: 5 (limit: 11389)
  Memory: 14.0M
  CGroup: /system.slice/maverics.service
          └─330772 /usr/local/bin/maverics --config /etc/maverics/maverics.yaml
  ```

4. Если не удается запустить службу Маверикс, выполните следующую команду, чтобы исследовать проблему:

   `journalctl --unit=maverics.service --reverse`

   Самая последняя запись журнала появится в начале выходных данных.

После установки Maverics в каталоге `/etc/maverics` создается файл по умолчанию `maverics.yaml`.

Настройте Orchestrator для защиты приложения. Интегрируйтесь с Azure AD B2C, храните и извлеките секреты из [Azure Key Vault](https://azure.microsoft.com/services/key-vault/?OCID=AID2100131_SEM_bf7bdd52c7b91367064882c1ce4d83a9:G:s&ef_id=bf7bdd52c7b91367064882c1ce4d83a9:G:s&msclkid=bf7bdd52c7b91367064882c1ce4d83a9). Определите расположение, из которого Orchestrator должен считывать конфигурацию.

### <a name="supply-configuration-using-environment-variables"></a>Предоставление конфигурации с помощью переменных среды

Предоставьте конфигурацию для экземпляров Orchestrator через переменные среды.

`MAVERICS_CONFIG`

Эта переменная среды сообщает экземпляру Orchestrator, какие файлы конфигурации YAML использовать, и где их можно найти во время запуска или перезапуска. Задайте переменную среды в `/etc/maverics/maverics.env` .

### <a name="create-the-orchestrators-tls-configuration"></a>Создание конфигурации TLS Orchestrator

`tls`В поле в `maverics.yaml` объявляется конфигурация безопасности транспортного уровня, которую будет использовать экземпляр Orchestrator. Соединители могут использовать объекты TLS и сервер Orchestrator.

Ключ `maverics` зарезервирован для сервера Orchestrator. Все остальные ключи доступны и могут использоваться для внедрения объекта TLS в данный соединитель.

```yaml
tls:
  maverics:
    certFile: /etc/maverics/maverics.cert
    keyFile: /etc/maverics/maverics.key
```

### <a name="configure-the-azure-ad-b2c-connector"></a>Настройка соединителя Azure AD B2C

Оркестрации используют соединители для интеграции с проверкой подлинности и поставщиками атрибутов. В этом случае шлюз приложений для оркестрации использует соединитель Azure AD B2C в качестве поставщика проверки подлинности и атрибута. Azure AD B2C использует социальные IdP для проверки подлинности, а затем выступает в качестве поставщика атрибутов для Orchestrator, передавая атрибуты в заявках, заданных в заголовках HTTP.  

Конфигурация этого соединителя соответствует приложению, зарегистрированному в клиенте Azure AD B2C.

1. Скопируйте идентификатор клиента, секрет и URI перенаправления из конфигурации приложения в клиенте.

2. Присвойте соединителю имя, как показано ниже `azureADB2C` , и задайте для соединителя значение `type` `azure` . Запишите имя соединителя, так как это значение используется в других параметрах конфигурации, приведенных ниже.

3. Для этой интеграции `authType` необходимо установить значение `oidc` .

4. Задайте идентификатор клиента, скопированный на шаге 1, в качестве значения `oauthClientID` параметра.

5. Задайте секрет клиента, скопированный на шаге 1 в качестве значения `oauthClientSecret` параметра.

6. Задайте URI перенаправления, скопированный на шаге 1 в качестве значения `oauthRedirectURL` параметра.

7. Соединитель Azure AD B2C OIDC использует хорошо известную конечную точку OIDC для обнаружения метаданных, включая URL-адреса и ключи подписывания. Задайте значение `oidcWellKnownURL` для конечной точки клиента.

```yaml
connectors:
  name: azureADB2C
  type: azure
  oidcWellKnownURL: https://<tenant name>.b2clogin.com/<tenant name>.onmicrosoft.com/B2C_1_login/v2.0/.well-known/openid-configuration
  oauthRedirectURL: https://example.com/oidc-endpoint
  oauthClientID: <azureADB2CClientID>
  oauthClientSecret: <azureADB2CClientSecret>
  authType: oidc
```

### <a name="define-azure-ad-b2c-as-your-authentication-provider"></a>Определение Azure AD B2C в качестве поставщика проверки подлинности

Поставщик проверки подлинности определяет, как выполнять проверку подлинности для пользователя, который не представил допустимый сеанс в рамках запроса ресурса приложения. Конфигурация в клиенте Azure AD B2C определяет способ запроса учетных данных пользователя и применение дополнительных политик проверки подлинности. Например, чтобы запросить второй фактор для завершения процесса проверки подлинности и решить, какие утверждения должны возвращаться в шлюз приложений Orchestrator после выполнения проверки подлинности.

Значение для параметра `authProvider` должно соответствовать `name` значению соединителя.

```yaml
authProvider: azureADB2C
```

### <a name="protect-your-on-premises-app-with-an-orchestrator-app-gateway"></a>Защита локального приложения с помощью шлюза приложений Orchestrator

Конфигурация шлюза приложений Orchestrator объявляет, как Azure AD B2C должен защищать приложение и как пользователи должны иметь доступ к приложению.

1. Создайте имя для шлюза приложений. Для приложения можно использовать понятное или полное имя узла в качестве идентификатора.

2. Задайте `location`. В примере здесь используется корневой каталог приложения `/` , но может быть любым URL-адресом приложения.

3. Определите защищенное приложение в `upstream` с помощью соглашения о порте узла: `https://example.com:8080` .

4. Задайте значения для ошибок и неавторизованных страниц.

5. Определите имена HTTP-заголовков и значения атрибутов, которые должны быть предоставлены приложению для установления проверки подлинности и управления доступом к приложению. Имена заголовков являются произвольными и обычно соответствуют конфигурации приложения. Значения атрибутов задаются соединителем, предоставляющим их. В приведенном ниже примере значения, возвращаемые из Azure AD B2C, имеют префикс с именем соединителя, `azureADB2C` где суффиксом является имя атрибута, содержащего требуемое значение, например `given_name` .

6. Задайте политики для оценки и принудительного применения. Определены три действия: `allowUnauthenticated` , `allowAnyAuthenticated` и `allowIfAny` . Каждое действие связано с `resource` и оценивается политика для этого `resource` .

>[!NOTE]
>`headers`И, и `policies` используют расширения служб JavaScript или GoLang для реализации произвольной логики, значительно расширяющей возможности по умолчанию.

```yaml
appgateways:
  - name: Sonar
    location: /
    upstream: https://example.com:8080
    errorPage: https://example.com:8080/sonar/error
    unauthorizedPage: https://example.com:8080/sonar/accessdenied

    headers:
      SM_USER: azureADB2C.sub
      firstname: azureADB2C.given_name
      lastname: azureADB2C.family_name

    policies:
      - resource: ~ \.(jpg|png|ico|svg)
        allowUnauthenticated: true
      - resource: /
        allowAnyAuthenticated: true
      - resource: /sonar/daily_deals
        allowIfAny:
          azureADB2C.customAttribute: Rewards Member
```

### <a name="use-azure-key-vault-as-your-secrets-provider"></a>Использование Azure Key Vault в качестве поставщика секретов

Важно защитить секреты, используемые Orchestrator для подключения к Azure AD B2C и любой другой системе идентификации. Однако Маверикс будет по умолчанию загружать секреты в виде обычного текста из `maverics.yaml` , но в этом руководстве вы будете использовать Azure Key Vault в качестве поставщика секретов.

Следуйте инструкциям, чтобы [создать новый Key Vault](../key-vault/secrets/quick-create-portal.md) , который будет использоваться экземпляром Orchestrator в качестве поставщика секретов. Добавьте секреты в хранилище и запишите их в `SECRET NAME` каждый секретный код. Например, `AzureADB2CClientSecret`.

Чтобы объявить значение как секрет в файле конфигурации `maverics.yaml`, заключите секрет в угловые скобки:

```yaml
connectors:
  - name: AzureADB2C
    type: azure
    oauthClientID: <AzureADB2CClientID>
    oauthClientSecret: <AzureADB2CClientSecret>
```

Значение, указанное в угловых скобках, должно соответствовать значению, `SECRET NAME` заданному для секрета в Azure Key Vault.

Чтобы загрузить секреты из Azure Key Vault, задайте переменную среды `MAVERICS_SECRET_PROVIDER` в файле `/etc/maverics/maverics.env` с учетными данными, найденными в azure-credentials.jsфайле, используя следующий шаблон:

`MAVERICS_SECRET_PROVIDER='azurekeyvault://<KEYVAULT NAME>.vault.azure.net?clientID=<APPID>&clientSecret=<PASSWORD>&tenantID=<TENANT>'`

### <a name="put-everything-together"></a>Итоговое объединение

Ниже показано, как конфигурация Orchestrator будет отображаться после завершения описанных выше конфигураций.

```yaml
version: 0.4.2
listenAddress: ":443"
tls:
  maverics:
    certFile: certs/maverics.crt
    keyFile: certs/maverics.key

authProvider: azureADB2C

connectors:
  - name: azureADB2C
    type: azure
    oidcWellKnownURL: https://<tenant name>.b2clogin.com/<tenant name>.onmicrosoft.com/B2C_1_login/v2.0/.well-known/openid-configuration
    oauthRedirectURL: https://example.com/oidc-endpoint
    oauthClientID: <azureADB2CClientID>
    oauthClientSecret: <azureADB2CClientSecret>
    authType: oidc

appgateways:
  - name: Sonar
    location: /
    upstream: http://example.com:8080
    errorPage: http://example.com:8080/sonar/accessdenied
    unauthorizedPage: http://example.com:8080/sonar/accessdenied

    headers:
      SM_USER: azureADB2C.sub
      firstname: azureADB2C.given_name
      lastname: azureADB2C.family_name

    policies:
      - resource: ~ \.(jpg|png|ico|svg)
        allowUnauthenticated: true
      - resource: /
        allowAnyAuthenticated: true
      - resource: /sonar/daily_deals
        allowIfAny:
          azureADB2C.customAttribute: Rewards Member
```

## <a name="test-the-flow"></a>Проверка потока

1. Перейдите к URL-адресу локального приложения, `https://example.com/sonar/dashboard` .

2. Orchestrator следует перенаправить на страницу, настроенную в потоке пользователя.

3. Выберите IdP из списка на странице.

4. Когда вы перейдете на IdP, укажите учетные данные в соответствии с запросом, включая токен MFA, если это требуется для этого IdP.

5. После успешной проверки подлинности необходимо перенаправить на Azure AD B2C, который перенаправляет запрос приложения в URI перенаправления Orchestrator.

6. Orchestrator оценивает политики, вычисляет заголовки и отправляет пользователя в вышестоящее приложение.  

7. Должно отобразиться запрошенное приложение.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения см. в следующих статьях:

- [Пользовательские политики в Azure AD B2C](./custom-policy-overview.md)

- [Приступая к работе с пользовательскими политиками в Azure AD B2C](./custom-policy-get-started.md?tabs=applications)
