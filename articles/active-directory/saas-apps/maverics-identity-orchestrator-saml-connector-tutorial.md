---
title: Руководство по интеграции единого входа Azure Active Directory с Maverics Identity Orchestrator SAML Connector | Документация Майкрософт
description: Сведения о настройке единого входа между Azure Active Directory и Maverics Identity Orchestrator SAML Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/17/2021
ms.author: jeedes
ms.openlocfilehash: 19f6b0601afe9ad84f02c93d7f6e1ae3a71a06a4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104585100"
---
# <a name="integrate-azure-ad-single-sign-on-with-maverics-identity-orchestrator-saml-connector"></a>интеграции единого входа Azure AD с Maverics Identity Orchestrator SAML Connector

Решение Maverics Identity Orchestrator от Strata предоставляет простой способ интеграции локальных приложений с Azure Active Directory (Azure AD) для проверки подлинности и управления доступом. Maverics Orchestrator поддерживает модернизацию проверки подлинности и авторизации приложений, в которых в настоящее время применяются заголовки, файлы cookie и другие защищаемые методы проверки подлинности. Экземпляры Maverics Orchestrator можно развертывать локально или в облаке. 

В этом учебнике по гибридному доступу показано, как перенести локальное веб-приложение, защищенное устаревшим продуктом управления веб-доступом, чтобы использовать Azure AD для проверки подлинности и контроля доступа. Ниже перечислены основные шаги.

1. Настройка Maverics Orchestrator
1. Использовать прокси-сервер для приложения
1. Регистрация корпоративного приложения в Azure AD
1. Проверка подлинности с помощью Azure и авторизация доступа к приложению
1. Добавление заголовков для простого доступа к приложениям
1. Работа с несколькими приложениями

## <a name="prerequisites"></a>Обязательные условия

* Подписка Azure AD. Если у вас нет подписки, вы можете получить [бесплатную учетную запись](https://azure.microsoft.com/free/).
* Подписка Mavericks Identity Orchestrator SAML Connector с поддержкой единого входа. Чтобы получить программное обеспечение Mavericks, свяжитесь с [отделом продаж Strata](mailto:sales@strata.io).
* По крайней мере одно приложение, использующее проверку подлинности на основе заголовка. Примеры приведены для приложения под названием Sonar, которое размещено по адресу https://app.sonarsystems.com, и приложения под названием Connectulum, которое размещено по адресу https://app.connectulum.com.
* Компьютер с Linux для размещения Maverics Orchestrator
  * ОС: RHEL 7.7 или более поздней версии, CentOS 7+
  * Диск: не менее 10 ГБ
  * Память: не менее 4 ГБ
  * Порты: 22 (SSH/SCP), 443, 7474
  * Доступ с правами root для задач установки и администрирования
  * Сетевой исходящий трафик, поступающий с сервера, на котором размещено решение Maverics Identity Orchestrator, в защищенное приложение

## <a name="step-1-set-up-the-maverics-orchestrator"></a>Шаг 1. Настройка Maverics Orchestrator

### <a name="install-maverics"></a>Установка Maverics

1. Получите новейшую версию RPM-пакета Maverics. Скопируйте пакет в систему, в которой вы хотите установить программное обеспечение Maverics.

1. Установите пакет Maverics, указав свое имя файла вместо `maverics.rpm`.

   `sudo rpm -Uvf maverics.rpm`

   После установки Maverics будет работать как служба под `systemd`. Чтобы убедиться, что служба запущена, выполните указанную ниже команду.

   `sudo systemctl status maverics`

1. Чтобы перезапустить Orchestrator и отслеживать журналы, можно воспользоваться следующей командой:

   `sudo service maverics restart; sudo journalctl --identifier=maverics -f`

После установки Maverics в каталоге `/etc/maverics` создается файл по умолчанию `maverics.yaml`. Прежде чем вы отредактируете конфигурацию, включив в нее `appgateways` и `connectors`, ваш файл конфигурации будет выглядеть следующим образом:

```yaml
# © Strata Identity Inc. 2020. All Rights Reserved. Patents Pending.

version: 0.1
listenAddress: ":7474"
```

### <a name="configure-dns"></a>Настройка DNS

Служба DNS избавляет от необходимости запоминать IP-адрес сервера Orchestrator.

Измените файл hosts для компьютера (портативного компьютера), где установлен браузер, используя гипотетический IP-адрес Orchestrator 12.34.56.78. В операционных системах на основе Linux этот файл находится в папке `/etc/hosts`. В Windows он находится в папке `C:\windows\system32\drivers\etc`.

```
12.34.56.78 sonar.maverics.com
12.34.56.78 connectulum.maverics.com
```

Чтобы убедиться в правильности настройки DNS, можно отправить запрос в конечную точку состояния Orchestrator. В браузере отправьте запрос http://sonar.maverics.com:7474/status.

### <a name="configure-tls"></a>Настройка TLS

Для обеспечения безопасности крайне важно обеспечить взаимодействие с Orchestrator по защищенным каналам. Для этого можно добавить пару "сертификат — ключ" в раздел `tls`.

Чтобы создать самозаверяющий сертификат и ключ для сервера Orchestrator, запустите выполнение следующей команды в каталоге `/etc/maverics`:

`openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out maverics.crt -keyout maverics.key`

> [!NOTE]
> Чтобы избежать предупреждений в браузере, для рабочих сред, скорее всего, потребуется использовать сертификат, подписанный известным центром сертификации. Если вам нужен доверенный центр сертификации, для этих целей прекрасно подойдет бесплатный ресурс [Let's Encrypt](https://letsencrypt.org/).

Теперь воспользуйтесь только что созданным сертификатом и ключом для Orchestrator. В итоге файл конфигурации должен содержать следующий код:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key
```

Чтобы убедиться в правильной настройке протокола TLS, перезапустите службу Maverics и выполните запрос к конечной точке состояния. В браузере отправьте запрос https://sonar.maverics.com/status.

## <a name="step-2-proxy-an-application"></a>Шаг 2. Использование прокси-сервера для приложения

Следующий шаг — настроить базовый прокси-сервер в Orchestrator с помощью `appgateways`. Это позволит обеспечить необходимое подключение Orchestrator к защищенному приложению.

В итоге файл конфигурации должен содержать следующий код:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

appgateways:
  - name: sonar
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com
```

Чтобы убедиться в правильной работе прокси-сервера, перезапустите службу Maverics и выполните запрос к приложению через прокси-сервер Maverics. В браузере отправьте запрос https://sonar.maverics.com. При необходимости можно выполнить запрос к конкретным ресурсам приложения, например `https://sonar.maverics.com/RESOURCE`, где `RESOURCE` — это допустимый ресурс защищенного вышестоящего приложения.

## <a name="step-3-register-an-enterprise-application-in-azure-ad"></a>Шаг 3. Регистрация корпоративного приложения в Azure AD

Теперь создайте в Azure AD новое корпоративное приложение, которое будет использоваться для проверки подлинности конечных пользователей.

> [!NOTE]
> При использовании таких функций Azure AD, как условный доступ, важно создать свое корпоративное приложение для каждого локального приложения. Это обеспечит условный доступ, оценку риска, назначение разрешений и т. д. отдельно для каждого приложения. Как правило, корпоративное приложение в Azure AD сопоставляется с соединителем Azure в Maverics.

Для регистрации корпоративного приложения в Azure AD выполните следующие действия:

1. В клиенте Azure AD перейдите в колонку **Корпоративные приложения** и выберите **Новое приложение**. В коллекции Azure AD найдите приложение **Maverics Identity Orchestrator SAML Connector** и выберите его.

1. На панели **Properties** (Свойства) страницы Maverics Identity Orchestrator SAML Connector установите для параметра **User assignment required?** (Требуется ли назначение пользователя?) значение **No** (Нет), чтобы с приложением могли работать все пользователи в каталоге.

1. На панели **Overview** (Обзор) страницы Maverics Identity Orchestrator SAML Connector Overview выберите **Set up single sign-on** (Настройка единого входа), а затем — **SAML**.

1. На панели **SAML-based sign on** (Вход на основе SAML) страницы Maverics Identity Orchestrator SAML Connector измените **базовую конфигурацию SAML**, нажав кнопку **редактирования** (значок карандаша).

   ![Снимок экрана: кнопка редактирования базовой конфигурации SAML.](common/edit-urls.png)

1. В поле **Идентификатор сущности** введите `https://sonar.maverics.com`. Идентификатор сущности должен быть уникальным для приложений в клиенте и может иметь произвольное значение. Это значение будет использоваться при определении поля `samlEntityID` для соединителя Azure в следующем разделе.

1. В поле **URL-адрес ответа** введите `https://sonar.maverics.com/acs`. Это значение будет использоваться при определении поля `samlConsumerServiceURL` для соединителя Azure в следующем разделе.

1. В поле **URL-адрес для входа** введите `https://sonar.maverics.com/`. Это поле не будет использоваться Maverics, но оно требуется в Azure AD, чтобы пользователи могли получить доступ к приложению через портал "Мои приложения" Azure AD.

1. Щелкните **Сохранить**.

1. В разделе **Сертификат подписи SAML** нажмите кнопку **Копировать**, чтобы скопировать значение **URL-адрес метаданных федерации приложений** и сохранить его на компьютере.

   ![Снимок экрана: "Сертификат подписи SAML" с кнопкой "Копировать".](common/copy-metadataurl.png)

## <a name="step-4-authenticate-via-azure-and-authorize-access-to-the-application"></a>Шаг 4. Проверка подлинности с помощью Azure и авторизация доступа к приложению

Затем разместите только что созданное корпоративное приложение для использования, настроив соединитель Azure в Maverics. Эта конфигурация `connectors`, связанная с блоком `idps`, позволяет Orchestrator проверять подлинность пользователей.

В итоге файл конфигурации должен содержать следующий код. Обязательно замените `METADATA_URL` значением URL-адреса метаданных федерации приложений из предыдущего шага.

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

idps:
  - name: azureSonarApp

appgateways:
  - name: sonar
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureSonarApp.authenticated}}", "true"]

connectors:
  - name: azureSonarApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://sonar.maverics.com/acs
    samlEntityID: https://sonar.maverics.com
```

Чтобы убедиться в правильной работе проверки подлинности, перезапустите службу Maverics и выполните запрос к ресурсу приложения через прокси-сервер Maverics. Перед доступом к ресурсу должно выполняться перенаправление в Azure для проверки подлинности.

## <a name="step-5-add-headers-for-seamless-application-access"></a>Шаг 5. Добавление заголовков для простого доступа к приложениям

На данный момент заголовки еще не отправляются в вышестоящее приложение. Давайте добавим `headers` в запрос, передаваемый через прокси-сервер Maverics, чтобы вышестоящее приложение могло опознать пользователя.

В итоге файл конфигурации должен содержать следующий код:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

idps:
  - name: azureSonarApp

appgateways:
  - name: sonar
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureSonarApp.authenticated}}", "true"]

    headers:
      email: azureSonarApp.name
      firstname: azureSonarApp.givenname
      lastname: azureSonarApp.surname

connectors:
  - name: azureSonarApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://sonar.maverics.com/acs
    samlEntityID: https://sonar.maverics.com
```

Чтобы убедиться в правильной работе проверки подлинности, выполните запрос к ресурсу приложения через прокси-сервер Maverics. Теперь защищенное приложение должно получать заголовки в запросе. 

Вы можете изменить ключи заголовков, если для приложения требуются другие заголовки. В заголовках можно использовать все утверждения, возвращенные из Azure AD в составе потока SAML. Например, можно включить другой заголовок `secondary_email: azureSonarApp.email`, где `azureSonarApp` — имя соединителя, а `email` — утверждение, возвращенное из Azure AD. 

## <a name="step-6-work-with-multiple-applications"></a>Шаг 6. Работа с несколькими приложениями

Теперь рассмотрим, что требуется для использования прокси-сервера для нескольких приложений на разных узлах. Чтобы это было возможно, настройте другой Шлюз приложений, другое корпоративное приложение в Azure AD и другой соединитель.

В итоге файл конфигурации должен содержать следующий код:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

idps:
  - name: azureSonarApp
  - name: azureConnectulumApp

appgateways:
  - name: sonar
    host: sonar.maverics.com
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureSonarApp.authenticated}}", "true"]

    headers:
      email: azureSonarApp.name
      firstname: azureSonarApp.givenname
      lastname: azureSonarApp.surname

  - name: connectulum
    host: connectulum.maverics.com
    location: /
    # Replace https://app.connectulum.com with the address of your protected application
    upstream: https://app.connectulum.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureConnectulumApp.authenticated}}", "true"]

    headers:
      email: azureConnectulumApp.name
      firstname: azureConnectulumApp.givenname
      lastname: azureConnectulumApp.surname

connectors:
  - name: azureSonarApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://sonar.maverics.com/acs
    samlEntityID: https://sonar.maverics.com

  - name: azureConnectulumApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://connectulum.maverics.com/acs
    samlEntityID: https://connectulum.maverics.com
```

Возможно, вы заметили, что код добавляет поле `host` в определения Шлюза приложений. Поле `host` позволяет Maverics Orchestrator определять вышестоящий узел, на который прокси-сервер должен передать тот или иной трафик.

Чтобы убедиться, что добавленный только что Шлюз приложений работает должным образом, выполните запрос к https://connectulum.maverics.com.

## <a name="advanced-scenarios"></a>Сложные сценарии

### <a name="identity-migration"></a>Миграция удостоверений

Не можете работать с устаревшим средством управления веб-доступом, но при этом нет возможности перенести пользователей без массовых сбросов паролей? Maverics Orchestrator поддерживает миграцию идентификаторов с помощью `migrationgateways`.

### <a name="web-server-gateways"></a>Шлюзы веб-сервера

Не хотите перенастраивать сеть и использовать прокси-сервер для направления трафика через Maverics Orchestrator? Не проблема. Maverics Orchestrator можно связать со шлюзами веб-серверов (модулями), чтобы создать такое же решение, но без прокси-сервера.

## <a name="wrap-up"></a>Создание оболочки

На данный момент вы установили Maverics Orchestrator, создали и настроили корпоративное приложение в Azure AD, а также настроили Orchestrator, чтобы использовать прокси-сервер для защищенного приложения, требуя при этом выполнения проверки подлинности и применения политики. Чтобы узнать больше о том, как можно применить Maverics Orchestrator для вариантов использования с управлением распределенными удостоверениями, [обратитесь в Strata](mailto:sales@strata.io).

## <a name="next-steps"></a>Дальнейшие действия

- [Что такое доступ к приложениям и единый вход с помощью Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
- [Что представляет собой условный доступ в Azure Active Directory?](../conditional-access/overview.md)
