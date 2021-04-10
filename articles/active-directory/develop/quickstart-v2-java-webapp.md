---
title: Краткое руководство. Добавление возможности входа с помощью учетной записи Майкрософт в веб-приложение Java | Azure
titleSuffix: Microsoft identity platform
description: Из этого краткого руководства вы узнаете, как добавить вход с учетной записью Майкрософт в веб-приложение Java с использованием OpenID Connect.
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 10/09/2019
ms.author: sagonzal
ms.custom: aaddev, scenarios:getting-started, languages:Java, devx-track-java
ms.openlocfilehash: a7337175241834cef862b4af07c7bcf7c8b845d0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "100103776"
---
# <a name="quickstart-add-sign-in-with-microsoft-to-a-java-web-app"></a>Краткое руководство. Добавление возможности входа в веб-приложение Java с помощью учетной записи Майкрософт

При работе с этим кратким руководством вы скачаете и выполните пример кода. Такой пример кода демонстрирует, как в веб-приложении Java реализовать вход пользователей и вызов Microsoft Graph. Входить в приложение могут пользователи из любой организации Azure Active Directory (Azure AD).

 Чтобы получить общие сведения см. [схему, демонстрирующую принцип работы примера](#how-the-sample-works).

## <a name="prerequisites"></a>Предварительные требования

Для запуска этого примера вам потребуются следующие компоненты:

- [Пакет SDK для Java](https://openjdk.java.net/) 8 или более поздней версии. 
- [Maven](https://maven.apache.org/).

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-app"></a>Регистрация и скачивание приложения, используемого в этом кратком руководстве
> Есть два варианта запуска приложения, используемого в этом кратком руководстве: оперативно (вариант 1) и вручную (вариант 2).
>
> ### <a name="option-1-register-and-automatically-configure-your-app-and-then-download-the-code-sample"></a>Вариант 1. Регистрация и автоматическая настройка вашего приложения, а затем скачивание примера кода
>
> 1. Откройте страницу <a href="https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/JavaQuickstartPage/sourceType/docs" target="_blank">регистрации приложений</a> на портале Azure и приступите к работе.
> 1. Введите имя приложения и нажмите кнопку **Зарегистрировать**.
> 1. Следуйте инструкциям на экране, чтобы скачать автоматически настроенный код приложения.
>
> ### <a name="option-2-register-and-manually-configure-your-application-and-code-sample"></a>Вариант 2. Регистрация и настройка приложения и примера кода вручную
>
> #### <a name="step-1-register-your-application"></a>Шаг 1. Регистрация приложения
>
> Чтобы зарегистрировать приложение и добавить в него сведения о регистрации вручную, сделайте следующее:
>
> 1. Войдите на <a href="https://portal.azure.com/" target="_blank">портал Azure</a>.
> 1. Если у вас есть доступ к нескольким арендаторам, в верхнем меню используйте фильтр **Каталог и подписка** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false":::, чтобы выбрать арендатор, в котором следует зарегистрировать приложение.
> 1. Найдите и выберите **Azure Active Directory**.
> 1. В разделе **Управление** выберите **Регистрация приложений**.
> 1. Выберите **Новая регистрация**.
> 1. Введите **имя** приложения, например **java-webapp**. Это имя отображается для пользователей вашего приложения. Его можно изменить позже.
> 1. Выберите **Зарегистрировать**.
> 1. На странице **Обзор** запишите идентификаторы **приложения (клиента)** и **каталога (арендатора)** . Эти значения потребуются позже.
> 1. В разделе **Управление** выберите **Проверка подлинности**.
> 1. Выберите **Добавить платформу** > **Веб**.
> 1. В разделе **URI перенаправления** введите `https://localhost:8443/msal4jsample/secure/aad`.
> 1. Нажмите кнопку **Настроить**.
> 1. В разделе **Веб** в поле **URI-коды перенаправления** введите `https://localhost:8443/msal4jsample/graph/me` в качестве второго URI перенаправления.
> 1. В разделе **Управление** выберите **Сертификаты и секреты**. В разделе **Секреты клиента** выберите **Новый секрет клиента**.
> 1. Введите описание ключа (например, *Секрет приложения*), оставьте срок действия по умолчанию и выберите элемент **Добавить**.
> 1. Запишите **значение** секрета клиента. Он понадобится вам позднее.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Шаг 1. Настройка приложения на портале Azure
>
> Для использования примера кода, приведенного в этом кратком руководстве, сделайте следующее:
>
> 1. Добавьте URL-адреса ответа `https://localhost:8443/msal4jsample/secure/aad` и `https://localhost:8443/msal4jsample/graph/me`.
> 1. Создайте секрет клиента.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Внести эти изменения для меня]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Уже настроено](media/quickstart-v2-aspnet-webapp/green-check.png). Ваше приложение настроено с помощью этих атрибутов.

#### <a name="step-2-download-the-code-sample"></a>Шаг 2. Скачивание примера кода
> [!div renderon="docs"]
> [Скачивание примера кода](https://github.com/Azure-Samples/ms-identity-java-webapp/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> Скачайте проект и извлеките ZIP-файл в папку рядом с корнем диска. Например, в *C:\Azure-Samples*.
>
> Чтобы использовать протокол HTTPS с localhost, укажите свойства `server.ssl.key`. Чтобы создать самозаверяющий сертификат, используйте служебную программу keytool (входит в JRE).
>
> Ниже приведен пример:
>  ```
>   keytool -genkeypair -alias testCert -keyalg RSA -storetype PKCS12 -keystore keystore.p12 -storepass password
>
>   server.ssl.key-store-type=PKCS12
>   server.ssl.key-store=classpath:keystore.p12
>   server.ssl.key-store-password=password
>   server.ssl.key-alias=testCert
>   ```
>   Поместите созданный файл хранилища ключей в папку *resources*.

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Скачивание примера кода](https://github.com/Azure-Samples/ms-identity-java-webapp/archive/master.zip)

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
> #### <a name="step-3-configure-the-code-sample"></a>Шаг 3. Настройка примера кода
> 1. Извлеките ZIP-файл в локальную папку.
> 1. *Необязательный элемент.* Если вы используете интегрированную среду разработки, откройте пример в этой среде.
> 1. Откройте файл *application.properties*. Его можно найти в папке *src/main/resources/* . Замените значения в полях `aad.clientId`, `aad.authority` и `aad.secretKey` значениями идентификатора приложения, идентификатора арендатора и секрета клиента соответственно. Это будет выглядеть примерно так:
>
>    ```file
>    aad.clientId=Enter_the_Application_Id_here
>    aad.authority=https://login.microsoftonline.com/Enter_the_Tenant_Info_Here/
>    aad.secretKey=Enter_the_Client_Secret_Here
>    aad.redirectUriSignin=https://localhost:8443/msal4jsample/secure/aad
>    aad.redirectUriGraph=https://localhost:8443/msal4jsample/graph/me
>    aad.msGraphEndpointHost="https://graph.microsoft.com/"
>    ```
>    В предыдущем коде:
>
>    - `Enter_the_Application_Id_here` — идентификатор регистрируемого приложения.
>    - `Enter_the_Client_Secret_Here` — это **секрет клиента**, созданный вами в разделе **сертификатов и секретов** для зарегистрированного приложения.
>    - `Enter_the_Tenant_Info_Here` — это значение **идентификатора каталога (арендатора)** приложения, которое вы зарегистрировали.
> 1. Чтобы использовать протокол HTTPS с localhost, укажите свойства `server.ssl.key`. Чтобы создать самозаверяющий сертификат, используйте служебную программу keytool (входит в JRE).
>
>    Ниже приведен пример:
>
>     ```
>      keytool -genkeypair -alias testCert -keyalg RSA -storetype PKCS12 -keystore keystore.p12 -storepass password
>
>      server.ssl.key-store-type=PKCS12
>      server.ssl.key-store=classpath:keystore.p12
>      server.ssl.key-store-password=password
>      server.ssl.key-alias=testCert
>      ```
>   1. Поместите созданный файл хранилища ключей в папку *resources*.


> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-run-the-code-sample"></a>Шаг 3. Запуск примера кода
> [!div renderon="docs"]
> #### <a name="step-4-run-the-code-sample"></a>Шаг 4. Запуск примера кода

Чтобы запустить проект, выполните одно из следующих действий:

- Запустите его непосредственно из IDE, используя встроенный сервер Spring Boot. 
- Упакуйте проект в WAR-файл с помощью [Maven](https://maven.apache.org/plugins/maven-war-plugin/usage.html), а затем разверните в решении J2EE для контейнеров, например в [Apache Tomcat](http://tomcat.apache.org/).

##### <a name="running-the-project-from-an-ide"></a>Запуск проекта из IDE

Чтобы запустить веб-приложение из IDE, щелкните элемент "Запустить", а затем перейдите на домашнюю страницу проекта. В этом примере стандартная домашняя страница имеет следующий URL-адрес: https://localhost:8443.

1. На основной странице нажмите кнопку **Вход** для перенаправления пользователей в Azure Active Directory и запрашивания у них учетных данных.

1. После проверки подлинности пользователи перенаправляются по адресу `https://localhost:8443/msal4jsample/secure/aad`. После входа на странице появятся сведения об учетной записи пользователя. В примере пользовательского интерфейса есть следующие кнопки:
    - **Выход**: пользователь выходит из приложения и перенаправляется на домашнюю страницу.
    - **Показать сведения о пользователе**: получение маркера Microsoft Graph и вызов Microsoft Graph с помощью запроса, который содержит этот маркер. Этот запрос возвращает сведения о пользователе, выполнившем вход в систему.

##### <a name="running-the-project-from-tomcat"></a>Запуск проекта из Tomcat

Если вы хотите развернуть пример веб-приложения в Tomcat, внесите несколько изменений в исходный код.

1. Откройте файл *ms-identity-java-webapp/pom.xml*.
    - В разделе `<name>msal-web-sample</name>` добавьте `<packaging>war</packaging>`.

2. Откройте файл *ms-identity-java-webapp/src/main/java/com.microsoft.azure.msalwebsample/MsalWebSampleApplication*.

    - Удалите весь исходный код, заменив его следующим кодом:

      ```Java
       package com.microsoft.azure.msalwebsample;

       import org.springframework.boot.SpringApplication;
       import org.springframework.boot.autoconfigure.SpringBootApplication;
       import org.springframework.boot.builder.SpringApplicationBuilder;
       import org.springframework.boot.web.servlet.support.SpringBootServletInitializer;

       @SpringBootApplication
       public class MsalWebSampleApplication extends SpringBootServletInitializer {

        public static void main(String[] args) {
         SpringApplication.run(MsalWebSampleApplication.class, args);
        }

        @Override
        protected SpringApplicationBuilder configure(SpringApplicationBuilder builder) {
         return builder.sources(MsalWebSampleApplication.class);
        }
       }
      ```

3.   HTTP-порт Tomcat по умолчанию — 8080, но вам требуется HTTPS-подключение через порт 8443. Чтобы настроить этот параметр:
        - Перейдите к файлу *tomcat/conf/server.xml*.
        - Найдите тег `<connector>` и замените существующий соединитель следующим соединителем:

          ```xml
          <Connector
                   protocol="org.apache.coyote.http11.Http11NioProtocol"
                   port="8443" maxThreads="200"
                   scheme="https" secure="true" SSLEnabled="true"
                   keystoreFile="C:/Path/To/Keystore/File/keystore.p12" keystorePass="KeystorePassword"
                   clientAuth="false" sslProtocol="TLS"/>
          ```

4. Откройте окно командной строки и Перейдите к корневой папке этого примера (где находится файл pom.xml) и запустите `mvn package` для сборки проекта.
    - С помощью этой команды в каталоге */targets* будет создан файл *msal-web-sample-0.1.0.war*.
    - Переименуйте этот файл в *msal4jsample.war*.
    - Разверните этот WAR-файл с помощью Tomcat или любого другого решения J2EE для контейнеров.
        - Чтобы развернуть файл msal4jsample.war, скопируйте его в каталог */webapps/* в своей установке Tomcat, а затем запустите сервер Tomcat.

5. После развертывания файла перейдите по адресу https://localhost:8443/msal4jsample в браузере.


> [!IMPORTANT]
> В рамках этого краткого руководства приложение использует секрет клиента для собственной идентификации в качестве конфиденциального клиента. Так как секрет клиента добавляется в качестве обычного текста в файлы проекта, из соображений безопасности рекомендуем применять сертификат вместо секрета клиента, прежде чем использовать приложение в рабочей среде. Дополнительные сведения о том, как использовать сертификат, см. в статье [Учетные данные сертификата для аутентификации приложения](./active-directory-certificate-credentials.md).

## <a name="more-information"></a>Дополнительные сведения

### <a name="how-the-sample-works"></a>Как работает этот пример
![Схема: работа примера приложения, созданного при работе с этим кратким руководством.](media/quickstart-v2-java-webapp/java-quickstart.svg)

### <a name="get-msal"></a>Получение MSAL

MSAL для Java (MSAL4J) — это библиотека Java, используемая для выполнения входа пользователей и запрашивания маркеров, которые нужны для доступа к API, защищенному платформой удостоверений Майкрософт.

Добавьте библиотеку MSAL4J в приложение с помощью Maven или Gradle для управления зависимостями, внеся следующие изменения в файл pom.xml (Maven) или build.gradle (Gradle) приложения.

В файле pom.xml:

```XML
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>msal4j</artifactId>
    <version>1.0.0</version>
</dependency>
```

В файле build.gradle:

```$xslt
compile group: 'com.microsoft.azure', name: 'msal4j', version: '1.0.0'
```

### <a name="initialize-msal"></a>Инициализация MSAL

Добавьте ссылку на MSAL для Java, добавив следующий код в начало файла, в котором будет использоваться MSAL4J:

```Java
import com.microsoft.aad.msal4j.*;
```

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Дальнейшие действия

Тема создания веб-приложений, в которых вход пользователей выполняется с помощью платформы удостоверений Майкрософт, подробно рассматривается в нашей серии сценариев:

> [!div class="nextstepaction"]
> [Scenario: Веб-приложение, выполняющее вход пользователей](scenario-web-app-sign-user-overview.md?tabs=java)
