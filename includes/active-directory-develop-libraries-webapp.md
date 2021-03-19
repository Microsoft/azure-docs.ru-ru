---
ms.openlocfilehash: 124290321815d22cad74d235205401b9380ff395
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "104578381"
---
| Язык и платформа | Проект включен<br/>GitHub                                                                                     | Пакет                                                                                                    | Появляться<br/>запущен                               | Выполнение входа пользователей                                            | Доступ к веб-API                                                    | Общедоступная версия (GA) *или*<br/>Общедоступная Предварительная версия<sup>1</sup> |
|----------------------|-----------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|:-------------------------------------------------:|:--------------------------------------------------------:|:------------------------------------------------------------------:|:------------------------------------------------------------:|
| .NET                 | [MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)                        | [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client)                      | —                                                 | ![Библиотека не может запросить маркеры идентификатора для входа пользователя.][n] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y]    | GA                                                           |
| ASP.NET Core         | [Безопасность ASP.NET](/aspnet/core/security/)                                                                | [Microsoft.AspNetCore.Authentication](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication/) | —                                                 | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y]    | ![Библиотека не может запросить маркеры доступа для защищенных веб-API.][n] | GA                                                           |
| ASP.NET Core         | [Microsoft. Identity. Web](https://github.com/AzureAD/microsoft-identity-web)                               | [Microsoft. Identity. Web](https://www.nuget.org/packages/Microsoft.Identity.Web)                            | —                                                 | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y]    | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y]    | GA                                                           |
| Java                 | [MSAL4J](https://github.com/AzureAD/microsoft-authentication-library-for-java)                            | [msal4j](https://search.maven.org/artifact/com.microsoft.azure/msal4j)                                     |[Краткое руководство](../articles/active-directory/develop/quickstart-v2-java-webapp.md) | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y]    | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y]    | GA                                                           |
| Node.js              | [MSAL Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) | [msal-node](https://www.npmjs.com/package/@azure/msal-node)                                                | [Краткое руководство](../articles/active-directory/develop/quickstart-v2-nodejs-webapp-msal.md) | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y]    | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y]    | GA                                               |
| Node.js              | [Azure AD Passport](https://github.com/AzureAD/passport-azure-ad)                                         | [Passport-Azure-AD](https://www.npmjs.com/package/passport-azure-ad)                                       | [Краткое руководство](../articles/active-directory/develop/quickstart-v2-nodejs-webapp.md)      | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y]    | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| Python               | [MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python)                     | [msal](https://pypi.org/project/msal)                                                                      | [Краткое руководство](../articles/active-directory/develop/quickstart-v2-python-webapp.md)      | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y]    | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y]    | GA                                                           |
<!--
| Java | [ScribeJava](https://github.com/scribejava/scribejava) | [ScribeJava 3.2.0](https://github.com/scribejava/scribejava/releases/tag/scribejava-3.2.0) | ![X indicating no.][n] | ![X indicating no.][n] | ![Green check mark.][y] | -- |
| Java | [Gluu oxAuth](https://github.com/GluuFederation/oxAuth) | [oxAuth 3.0.2](https://github.com/GluuFederation/oxAuth/releases/tag/3.0.2) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
| Node.js | [openid-client](https://github.com/panva/node-openid-client/) | [openid-client 2.4.5](https://github.com/panva/node-openid-client/releases/tag/v2.4.5) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
| PHP | [PHP League oauth2-client](https://github.com/thephpleague/oauth2-client) | [oauth2-client 1.4.2](https://github.com/thephpleague/oauth2-client/releases/tag/1.4.2) | ![X indicating no.][n] | ![X indicating no.][n] | ![Green check mark.][y] | -- |
| Ruby | [OmniAuth](https://github.com/omniauth/omniauth) | [omniauth 1.3.1](https://github.com/omniauth/omniauth/releases/tag/v1.3.1)<br/>[omniauth-oauth2 1.4.0](https://github.com/intridea/omniauth-oauth2) | ![X indicating no.][n] | ![X indicating no.][n] | ![Green check mark.][y] | -- |
-->

<sup>1</sup> [Дополнительные условия использования для Microsoft Azure предварительных версий][preview-tos] применяются к библиотекам в *общедоступной предварительной версии*.

<!--Image references-->

[y]: ../articles/active-directory/develop/media/common/yes.png
[n]: ../articles/active-directory/develop/media/common/no.png

<!--Reference-style links -->

[preview-tos]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/