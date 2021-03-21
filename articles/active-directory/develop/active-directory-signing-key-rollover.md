---
title: Смена ключей подписывания на платформе Microsoft Identity
description: В этой статье рассматриваются рекомендации по смене ключей подписывания для Azure Active Directory
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 8/11/2020
ms.author: ryanwi
ms.reviewer: paulgarn, hirsin
ms.custom: aaddev
ms.openlocfilehash: ce4917f968ef1664a1d41f4eaff162df116bda4f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102035090"
---
# <a name="signing-key-rollover-in-the-microsoft-identity-platform"></a>Смена ключей подписывания на платформе Microsoft Identity
В этой статье рассказывается о том, что необходимо знать об открытых ключах, используемых платформой Microsoft Identity для подписи маркеров безопасности. Важно отметить, что эти ключи переносятся на периодический основе, и в экстренной ситуации можно сразу же выполнить откат. Все приложения, использующие платформу Microsoft Identity, должны быть способны программно обрабатывать процесс смены ключей. Здесь вы узнаете принцип действия ключей, а также то, как оценить степень влияния их смены на приложение и как обновить приложение или настроить периодический запуск процесса смены ключей в ручном режиме, чтобы приложение могло обрабатывать этот процесс.

## <a name="overview-of-signing-keys-in-the-microsoft-identity-platform"></a>Общие сведения о ключах подписывания на платформе Microsoft Identity
Платформа Microsoft Identity использует шифрование с открытым ключом, основанное на отраслевых стандартах, для установления отношений доверия между собой и приложениями, которые его используют. На практике это работает следующим образом: платформа Microsoft Identity использует ключ подписывания, состоящий из пары открытого и закрытого ключей. Когда пользователь входит в приложение, использующее платформу идентификации Майкрософт для проверки подлинности, платформа Microsoft Identity создает маркер безопасности, содержащий сведения о пользователе. Этот маркер подписывается платформой удостоверений Майкрософт с помощью закрытого ключа перед отправкой обратно в приложение. Чтобы убедиться, что маркер является допустимым и исходит от платформы Microsoft Identity, приложение должно проверить подпись маркера с помощью открытых ключей, предоставляемых платформой Microsoft Identity, которая содержится в документе [OpenID Connect Connect Discovery](https://openid.net/specs/openid-connect-discovery-1_0.html) или в [документе метаданных федерации](../azuread-dev/azure-ad-federation-metadata.md)SAML/WS-подач.

В целях безопасности ключ подписывания платформы Microsoft Identity выполняется периодически и, в случае аварии, может быть выполнен немедленно. Нет установленного или гарантированного времени между этим ключом. все приложения, которые интегрируются с платформой Microsoft Identity, должны быть готовы к обработке события смены ключа, независимо от того, насколько часто это может произойти. Если такая логика не предусмотрена и приложение попытается использовать ключ с истекшим сроком действия, чтобы проверить подпись токена, произойдет сбой запроса на вход.  Рекомендуется проверять наличие обновлений каждые 24 часа, при этом с регулированием (каждые пять минут в большинстве случаев) немедленно обновляет ключ документа, если обнаружен маркер с неизвестным идентификатором ключа. 

В документе метаданных федерации и документе обнаружения OpenID Connect всегда существует несколько допустимых ключей. Приложение должно быть подготовлено к использованию всех ключей, указанных в документе, так как один ключ может быть выполнен в ближайшее время, другой может быть его заменой и т. д.  Число существующих ключей может меняться со временем на основе внутренней архитектуры платформы Microsoft Identity, так как мы поддерживаем новые платформы, новые облака или новые протоколы проверки подлинности. Ни порядок ключей в ответе JSON, ни порядок, в котором они не были предоставлены, следует рассматривать как меанинфул для вашего приложения. 

Приложения, поддерживающие только один ключ подписывания или требующие ручного обновления ключей подписывания, по своей природе являются менее безопасными и надежными.  Они должны быть обновлены для использования [стандартных библиотек](reference-v2-libraries.md) , чтобы гарантировать, что они всегда используют актуальные ключи подписывания, помимо других рекомендаций. 

## <a name="how-to-assess-if-your-application-will-be-affected-and-what-to-do-about-it"></a>Как оценить степень влияния смены ключей на приложение
Обработка смены ключей в приложении зависит от нескольких переменных, например от типа приложения или используемых протокола идентификации и библиотеки. В приведенных ниже разделах рассматривается, влияет ли смена ключей на самые распространенные типы приложений, а также содержатся рекомендации по обновлению приложения для поддержки автоматической смены ключей и обновлении ключей вручную.

* [Собственные клиентские приложения, осуществляющие доступ к ресурсам](#nativeclient)
* [Веб-приложения и интерфейсы API, осуществляющие доступ к ресурсам](#webclient)
* [Веб-приложения и интерфейсы API, защищающие ресурсы и созданные с помощью служб приложений Azure](#appservices)
* [Веб-приложения и интерфейсы API, защищающие ресурсы с помощью .NET OWIN OpenID Connect Connect, WS-Fed или Виндовсазуреактиведиректорибеарераусентикатион по промежуточного слоя](#owin)
* [Веб-приложения и интерфейсы API, защищающие ресурсы с использованием ПО промежуточного слоя для аутентификации на основе токена носителя JWT или OpenID Connect .NET Core](#owincore)
* [Веб-приложения и интерфейсы API, защищающие ресурсы с помощью модуля Node.js passport-azure-ad](#passport)
* [Веб-приложения и интерфейсы API, защищающие ресурсы и созданные с помощью Visual Studio 2015 или более поздней версии](#vs2015)
* [Веб-приложения, защищающие ресурсы и созданные с помощью Visual Studio 2013](#vs2013)
* Защищающие ресурсы веб-API, созданные в Visual Studio 2013
* [Веб-приложения, защищающие ресурсы и созданные с помощью Visual Studio 2012](#vs2012)
* [Защищающие ресурсы веб-приложения, созданные с помощью Visual Studio 2008, 2010 или Windows Identity Foundation](#vs2010)
* [Веб-приложения и интерфейсы API, защищающие ресурсы с помощью любых других библиотек или вручную реализующих Поддерживаемые протоколы](#other)

Это руководство **не** применимо к:

* приложениям, добавленным из коллекции приложений Azure AD (включая пользовательские), так как для них предусмотрены отдельные правила в отношении ключей подписывания [Дополнительные сведения.](../manage-apps/manage-certificates-for-federated-single-sign-on.md)
* локальным приложениям, опубликованным через прокси приложения, так как им не требуются ключи подписывания.

### <a name="native-client-applications-accessing-resources"></a><a name="nativeclient"></a>Собственные клиентские приложения, осуществляющие доступ к ресурсам
Как правило, приложения, только осуществляющие доступ к ресурсам Microsoft Graph, KeyVault, API Outlook и другие API-интерфейсы Майкрософт) обычно получают только маркер и передают его владельцу ресурса. Так как они не защищают ресурсы, они не проверяют токен, а вместе с тем и его подпись.

Собственные клиентские приложения, будь то классические или мобильные, попадают в эту категорию. Поэтому смена ключей не влияет на этот тип приложений.

### <a name="web-applications--apis-accessing-resources"></a><a name="webclient"></a>Веб-приложения и интерфейсы API, осуществляющие доступ к ресурсам
Как правило, приложения, только осуществляющие доступ к ресурсам Microsoft Graph, KeyVault, API Outlook и другие API-интерфейсы Майкрософт) обычно получают только маркер и передают его владельцу ресурса. Так как они не защищают ресурсы, они не проверяют токен, а вместе с тем и его подпись.

Веб-приложения и веб-API, использующие поток только для приложений (учетные данные клиента или клиентский сертификат) для запроса маркеров, попадают в эту категорию и поэтому не затрагиваются сменой.

### <a name="web-applications--apis-protecting-resources-and-built-using-azure-app-services"></a><a name="appservices"></a>Защищающие ресурсы веб-приложения и интерфейсы API, созданные с помощью служб приложений Azure
В функциях проверки подлинности и авторизации (EasyAuth) служб приложений Azure уже предусмотрена логика для обработки автоматической смены ключей.

### <a name="web-applications--apis-protecting-resources-using-net-owin-openid-connect-ws-fed-or-windowsazureactivedirectorybearerauthentication-middleware"></a><a name="owin"></a>Веб-приложения и интерфейсы API, защищающие ресурсы с использованием ПО промежуточного слоя для аутентификации Microsoft Azure Active Directory, .NET OWIN OpenID Connect или WS-Fed
Если в приложении используется ПО промежуточного слоя для аутентификации Microsoft Azure Active Directory, .NET OWIN OpenID Connect или WS-Fed, в нем уже предусмотрена логика для обработки автоматической смены ключей.

Вы можете убедиться, что приложение использует любое из следующих фрагментов кода в файлах Startup. cs или Startup. auth. CS приложения.

```csharp
app.UseOpenIdConnectAuthentication(
    new OpenIdConnectAuthenticationOptions
    {
        // ...
    });
```

```csharp
app.UseWsFederationAuthentication(
    new WsFederationAuthenticationOptions
    {
        // ...
    });
```

```csharp
app.UseWindowsAzureActiveDirectoryBearerAuthentication(
    new WindowsAzureActiveDirectoryBearerAuthenticationOptions
    {
        // ...
    });
```

### <a name="web-applications--apis-protecting-resources-using-net-core-openid-connect-or--jwtbearerauthentication-middleware"></a><a name="owincore"></a>Веб-приложения и интерфейсы API, защищающие ресурсы с использованием ПО промежуточного слоя для аутентификации на основе токена носителя JWT или OpenID Connect .NET Core
Если в приложении используется ПО промежуточного слоя для аутентификации на основе токена носителя JWT или .NET Core OWIN OpenID Connect, в нем уже предусмотрена логика для обработки автоматической смены ключей.

Чтобы проверить это, нужно поискать один из следующих фрагментов кода в файле Startup.cs или Startup.Auth.cs приложения.

```
app.UseOpenIdConnectAuthentication(
     new OpenIdConnectAuthenticationOptions
     {
         // ...
     });
```
```
app.UseJwtBearerAuthentication(
    new JwtBearerAuthenticationOptions
    {
     // ...
     });
```

### <a name="web-applications--apis-protecting-resources-using-nodejs-passport-azure-ad-module"></a><a name="passport"></a>Веб-приложения и интерфейсы API, защищающие ресурсы с помощью модуля Node.js passport-azure-ad
Если в приложении используется модуль Node.js passport-ad, в нем уже предусмотрена логика для обработки автоматической смены ключей.

Это можно проверить, выполнив поиск следующего фрагмента кода в файле app.js приложения.

```
var OIDCStrategy = require('passport-azure-ad').OIDCStrategy;

passport.use(new OIDCStrategy({
    //...
));
```

### <a name="web-applications--apis-protecting-resources-and-created-with-visual-studio-2015-or-later"></a><a name="vs2015"></a>Веб-приложения и интерфейсы API, защищающие ресурсы и созданные с помощью Visual Studio 2015 или более поздней версии
Если приложение было создано с помощью шаблона веб-приложения в Visual Studio 2015 или более поздней версии и вы выбрали пункт **рабочие или учебные учетные записи** в меню **изменить проверку подлинности** , у него уже есть необходимая логика для автоматической обработки ключей смены ключа. Эта логика, внедренная в ПО промежуточного слоя OWIN OpenID Connect, извлекает ключи из документа обнаружения OpenID Connect, кэширует их и периодически обновляет.

Если проверка подлинности добавлена в решение вручную, в приложении может отсутствовать необходимая логика смены ключей. Вам потребуется написать его самостоятельно или выполнить действия, описанные в разделе [веб-приложения и интерфейсы API, используя любые другие библиотеки или вручную реализовать любой из поддерживаемых протоколов](#other).

### <a name="web-applications-protecting-resources-and-created-with-visual-studio-2013"></a><a name="vs2013"></a>Защищающие ресурсы веб-приложения, созданные в Visual Studio 2013
Если приложение создано в Visual Studio 2013 с использованием шаблона веб-приложения и в меню **Изменить способ проверки подлинности** выбран пункт **Учетные записи организации**, в приложении уже есть необходимая логика для обработки автоматической смены ключей. Логика хранит уникальный идентификатор организации и сведения о ключе подписывания в двух таблицах базы данных, связанных с проектом. Строку подключения для базы данных можно найти в файле Web.config проекта.

Если проверка подлинности добавлена в решение вручную, в приложении может отсутствовать необходимая логика смены ключей. Ее понадобится написать самостоятельно или выполнить действия, описанные в разделе [Веб-приложения и интерфейсы API, использующие другие библиотеки или ручное внедрение поддерживаемых протоколов](#other).

Далее представлены шаги, которые помогут проверить правильность работы логики приложения.

1. Откройте в Visual Studio 2013 решение, а затем перейдите на вкладку **Обозреватель серверов** в правом окне.
2. Разверните **Подключения данных**, **DefaultConnection**, а затем **Таблицы**. Найдите таблицу **IssuingAuthorityKeys**, щелкните ее правой кнопкой мыши и выберите команду **Показать таблицу данных**.
3. В таблице **IssuingAuthorityKeys** будет по крайней мере одна строка, соответствующая значению отпечатка для ключа. Удалите все строки в таблице.
4. Щелкните правой кнопкой мыши таблицу **Клиенты**, а затем выберите команду **Показать таблицу данных**.
5. В таблице **Клиенты** будет по крайней мере одна строка, соответствующая уникальному идентификатору клиента каталога. Удалите все строки в таблице. Если не удалить строки в таблицах **Tenants** и **IssuingAuthorityKeys**, то при выполнении произойдет ошибка.
6. Выполните сборку и запуск приложения. Войдя в учетную запись, можно остановить работу приложения.
7. Вернитесь в **Обозреватель серверов** и найдите значения в таблицах **IssuingAuthorityKeys** и **Клиенты**. Как можно заметить, в них автоматически добавлены соответствующие сведения из документа метаданных федерации.

### <a name="web-apis-protecting-resources-and-created-with-visual-studio-2013"></a><a name="vs2013"></a>Защищающие ресурсы веб-интерфейсы, созданные в Visual Studio 2013
Если вы создали приложение с веб-интерфейсами API в Visual Studio 2013 с помощью шаблона веб-интерфейса API, а затем выбрали пункт **Учетные записи в организации** меню **Изменить аутентификацию**, ваше приложение уже обладает необходимой логикой.

Если вы настроили проверку подлинности вручную, следуйте приведенным ниже инструкциям, чтобы узнать, как настроить веб-API для автоматического обновления своих основных сведений.

В следующем фрагменте кода показано, как получить последние ключи из документа метаданных федерации, а затем использовать [обработчик маркеров JWT](/previous-versions/dotnet/framework/security/json-web-token-handler) для проверки маркера. В фрагменте кода предполагается, что вы будете использовать собственный механизм кэширования для сохранения ключа, чтобы проверить будущие токены от платформы Microsoft Identity, будь то база данных, файл конфигурации или другое место.

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.IdentityModel.Tokens;
using System.Configuration;
using System.Security.Cryptography.X509Certificates;
using System.Xml;
using System.IdentityModel.Metadata;
using System.ServiceModel.Security;
using System.Threading;

namespace JWTValidation
{
    public class JWTValidator
    {
        private string MetadataAddress = "[Your Federation Metadata document address goes here]";

        // Validates the JWT Token that's part of the Authorization header in an HTTP request.
        public void ValidateJwtToken(string token)
        {
            JwtSecurityTokenHandler tokenHandler = new JwtSecurityTokenHandler()
            {
                // Do not disable for production code
                CertificateValidator = X509CertificateValidator.None
            };

            TokenValidationParameters validationParams = new TokenValidationParameters()
            {
                AllowedAudience = "[Your App ID URI goes here, as registered in the Azure Portal]",
                ValidIssuer = "[The issuer for the token goes here, such as https://sts.windows.net/68b98905-130e-4d7c-b6e1-a158a9ed8449/]",
                SigningTokens = GetSigningCertificates(MetadataAddress)

                // Cache the signing tokens by your desired mechanism
            };

            Thread.CurrentPrincipal = tokenHandler.ValidateToken(token, validationParams);
        }

        // Returns a list of certificates from the specified metadata document.
        public List<X509SecurityToken> GetSigningCertificates(string metadataAddress)
        {
            List<X509SecurityToken> tokens = new List<X509SecurityToken>();

            if (metadataAddress == null)
            {
                throw new ArgumentNullException(metadataAddress);
            }

            using (XmlReader metadataReader = XmlReader.Create(metadataAddress))
            {
                MetadataSerializer serializer = new MetadataSerializer()
                {
                    // Do not disable for production code
                    CertificateValidationMode = X509CertificateValidationMode.None
                };

                EntityDescriptor metadata = serializer.ReadMetadata(metadataReader) as EntityDescriptor;

                if (metadata != null)
                {
                    SecurityTokenServiceDescriptor stsd = metadata.RoleDescriptors.OfType<SecurityTokenServiceDescriptor>().First();

                    if (stsd != null)
                    {
                        IEnumerable<X509RawDataKeyIdentifierClause> x509DataClauses = stsd.Keys.Where(key => key.KeyInfo != null && (key.Use == KeyType.Signing || key.Use == KeyType.Unspecified)).
                                                             Select(key => key.KeyInfo.OfType<X509RawDataKeyIdentifierClause>().First());

                        tokens.AddRange(x509DataClauses.Select(token => new X509SecurityToken(new X509Certificate2(token.GetX509RawData()))));
                    }
                    else
                    {
                        throw new InvalidOperationException("There is no RoleDescriptor of type SecurityTokenServiceType in the metadata");
                    }
                }
                else
                {
                    throw new Exception("Invalid Federation Metadata document");
                }
            }
            return tokens;
        }
    }
}
```

### <a name="web-applications-protecting-resources-and-created-with-visual-studio-2012"></a><a name="vs2012"></a>Защищающие ресурсы веб-приложения, созданные в Visual Studio 2012
Если приложение создано в Visual Studio 2012, вероятно, оно настроено с помощью средства Identity and Access Tool. Кроме того, возможно, используется [проверка реестра имен поставщиков](/previous-versions/dotnet/framework/security/validating-issuer-name-registry). VINR отвечает за хранение сведений о доверенных поставщиках удостоверений (платформа Microsoft Identity) и ключах, используемых для проверки маркеров, выданных ими. VINR также упрощает автоматическое обновление сведений о ключах, хранящихся в файле Web.config. Для этого скачивается последний документ метаданных федерации, связанный с каталогом, проверяется,не устарела ли конфигурация по сравнению с этим документом, и приложение обновляется для использования нового ключа при необходимости.

Если приложение создано с использованием любого из примеров кода или документации с пошаговыми инструкциями, предоставленных корпорацией Майкрософт, логика смены ключей уже добавлена в проект. Вы заметите, что код ниже уже есть в проекте. Если в приложении нет этой логики, выполните следующие шаги, чтобы добавить ее и убедиться в правильности ее работы.

1. Добавьте в **обозревателе решений** ссылку на сборку **System.IdentityModel** для соответствующего проекта.
2. Откройте файл **Global.asax.cs** и добавьте следующее, используя директивы:
   ```
   using System.Configuration;
   using System.IdentityModel.Tokens;
   ```
3. Добавьте в файл **Global.asax.cs** следующий метод:
   ```
   protected void RefreshValidationSettings()
   {
    string configPath = AppDomain.CurrentDomain.BaseDirectory + "\\" + "Web.config";
    string metadataAddress =
                  ConfigurationManager.AppSettings["ida:FederationMetadataLocation"];
    ValidatingIssuerNameRegistry.WriteToConfig(metadataAddress, configPath);
   }
   ```
4. Вызовите метод **RefreshValidationSettings()** в методе **Application_Start()** в файле **Global.asax.cs**, как показано далее:
   ```
   protected void Application_Start()
   {
    AreaRegistration.RegisterAllAreas();
    ...
    RefreshValidationSettings();
   }
   ```

После выполнения этих шагов файл Web.config приложения обновится с использованием последних сведений документа метаданных федерации, в том числе последних ключей. Это обновление будет выполняться при каждом перезапуске пула приложений в IIS. По умолчанию в IIS настроен перезапуск приложений через каждые 29 часов.

Выполните следующие шаги, чтобы убедиться, что логика смены ключей работает должным образом.

1. Убедившись, что приложение использует код, приведенный выше, откройте файл **Web.config**, перейдите к блоку **\<issuerNameRegistry>** и найдите следующие несколько строк.
   ```
   <issuerNameRegistry type="System.IdentityModel.Tokens.ValidatingIssuerNameRegistry, System.IdentityModel.Tokens.ValidatingIssuerNameRegistry">
        <authority name="https://sts.windows.net/ec4187af-07da-4f01-b18f-64c2f5abecea/">
          <keys>
            <add thumbprint="3A38FA984E8560F19AADC9F86FE9594BB6AD049B" />
          </keys>
   ```
2. В **\<add thumbprint="">** параметре измените значение отпечатка, заменив любой символ другим. Сохраните файл **Web.config**.
3. Выполните сборку приложения и запустите его. Если у вас получилось войти, приложение успешно обновляет ключ, скачивая необходимые данные из документа метаданных федерации каталога. Если у вас возникают проблемы при входе, убедитесь, что изменения в приложении верны, прочитав [добавление Sign-On в веб-приложение с помощью платформы удостоверений Майкрософт](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect) , или скачайте и проверите следующий пример кода: [Многоклиентское облачное приложение для Azure Active Directory](https://code.msdn.microsoft.com/multi-tenant-cloud-8015b84b).

### <a name="web-applications-protecting-resources-and-created-with-visual-studio-2008-or-2010-and-windows-identity-foundation-wif-v10-for-net-35"></a><a name="vs2010"></a>Защищающие ресурсы веб-приложения, созданные с помощью Visual Studio 2008 или 2010 и Windows Identity Foundation v1.0 для .NET 3.5
Если приложение создано на платформе Windows Identity Foundation v1.0, в нем не предусмотрен механизм автоматического обновления конфигурации для использования нового ключа.

* *проще* всего в пакете SDK для Windows Identity Foundation, с помощью которого можно получить последний документ метаданных и обновить конфигурацию.
* Обновите платформу приложения до версии .NET 4.5, которая содержит новейшую версию Windows Identity Foundation, расположенную в пространстве имен System. Затем можно использовать [проверку реестра имен поставщиков](/previous-versions/dotnet/framework/security/validating-issuer-name-registry) , чтобы автоматически обновить конфигурацию приложения.
* Запустите процесс смены ключей в ручном режиме согласно инструкциям, приведенным в конце этого руководства.

Ниже приведены инструкции по использованию FedUtil для обновления конфигурации.

1. Убедитесь, что на компьютере для разработки для Visual Studio 2008 или 2010 установлен пакет SDK для Windows Identity Foundation v1.0. Если этот пакет не установлен, его можно [скачать отсюда](https://www.microsoft.com/download/details.aspx?id=17331).
2. Откройте в Visual Studio решение, щелкните правой кнопкой мыши соответствующий проект и выберите пункт **Update federation metadata** (Обновление метаданных федерации). Если этот параметр недоступен, средство FedUtil и/или пакет SDK для Windows Identity Foundation v1.0 не установлен.
3. Выберите в информационном сообщении пункт **Обновить**, чтобы начать обновление метаданных федерации. При наличии доступа к среде сервера, где размещено приложение, можно использовать [планировщик автоматического обновления метаданных](/previous-versions/windows-identity-foundation/ee517272(v=msdn.10))средства FedUtil.
4. Чтобы завершить процесс обновления, нажмите кнопку **Готово**.

### <a name="web-applications--apis-protecting-resources-using-any-other-libraries-or-manually-implementing-any-of-the-supported-protocols"></a><a name="other"></a>Веб-приложения и интерфейсы API, защищающие ресурсы с помощью других библиотек или ручного внедрения поддерживаемых протоколов
Если используется другая библиотека или вручную внедрены поддерживаемые протоколы, их необходимо проверить, чтобы убедиться, что ключ извлекается из документа обнаружения OpenID Connect или документа метаданных федерации. Для этого необходимо проверить в коде приложения или коде библиотеки наличие вызовов документа обнаружения OpenID Connect или документа метаданных федерации.

Если ключ хранится в другом месте или жестко закодирован в приложении, можно вручную извлечь и обновить его, выполнив смену ключа вручную согласно инструкциям, приведенным в конце этого руководства. **Настоятельно рекомендуется, чтобы приложение поддерживало автоматическую смену ключей** с помощью любого из подходов, описанных в этой статье, чтобы избежать будущих сбоев и накладных расходов, если платформа Microsoft Identity повышает частоту смены ключей или имеет аварийную смену ресурсов.

## <a name="how-to-test-your-application-to-determine-if-it-will-be-affected"></a>Тестирование приложения для оценки степени влияния
Чтобы проверить, поддерживается ли автоматическая смена ключей в приложении, необходимо скачать скрипты и выполнить инструкции, описанные [в этом репозитории GitHub](https://github.com/AzureAD/azure-activedirectory-powershell-tokenkey)

## <a name="how-to-perform-a-manual-rollover-if-your-application-does-not-support-automatic-rollover"></a>Как сменить ключи вручную, если приложение не поддерживает автоматическую смену ключей
Если приложение **не поддерживает** автоматическую смену ключей, необходимо будет установить процесс, который периодически отслеживает ключи подписывания платформы Microsoft Identity и выполняет смену вручную соответствующим образом. [Этот репозиторий GitHub](https://github.com/AzureAD/azure-activedirectory-powershell-tokenkey) содержит скрипты и инструкции о том, как это сделать.
