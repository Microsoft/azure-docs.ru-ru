---
ms.openlocfilehash: 0837c0a23c837065dc2214f947912ee25e4f1d2d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103008010"
---
| Язык и платформа | Проект включен<br/>GitHub                                                                 | Пакет                                                                                | Появляться<br/>запущен                           | Выполнение входа пользователей                                            | Доступ к веб-API                                                 | Общедоступная версия (GA) *или*<br/>Общедоступная Предварительная версия<sup>1</sup> |
|----------------------|---------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------|:---------------------------------------------:|:--------------------------------------------------------:|:---------------------------------------------------------------:|:------------------------------------------------------------:|
| .NET                 | [MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)    | [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client/) | [Краткое руководство](../articles/active-directory/develop/quickstart-v2-netcore-daemon.md) | ![Библиотека не может запросить маркеры идентификатора для входа пользователя.][n] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| Java                 | [MSAL4J](https://github.com/AzureAD/microsoft-authentication-library-for-java)        | [msal4j](https://javadoc.io/doc/com.microsoft.azure/msal4j/latest/index.html)          | —                                             | ![Библиотека не может запросить маркеры идентификатора для входа пользователя.][n] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| Узел               | [MSAL Node](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) | [msal-node](https://www.npmjs.com/package/@azure/msal-node)  | [Краткое руководство](../articles/active-directory/develop/quickstart-v2-nodejs-console.md)  | ![Библиотека не может запросить маркеры идентификатора для входа пользователя.][n] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA  |
| Python               | [MSAL Python](https://github.com/AzureAD/microsoft-authentication-library-for-python) | [msal-Python](https://github.com/AzureAD/microsoft-authentication-library-for-python)  | [Краткое руководство](../articles/active-directory/develop/quickstart-v2-python-daemon.md)  | ![Библиотека не может запросить маркеры идентификатора для входа пользователя.][n] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
<!--
|PHP| [The PHP League oauth2-client](https://oauth2-client.thephpleague.com/usage/) | [League\OAuth2](https://oauth2-client.thephpleague.com/) | ![Green check mark.][n] | ![X indicating no.][n] | ![Green check mark.][y] | -- |
-->

<sup>1</sup> [Дополнительные условия использования для Microsoft Azure предварительных версий][preview-tos] применяются к библиотекам в *общедоступной предварительной версии*.

<!--Image references-->

[y]: ../articles/active-directory/develop/media/common/yes.png
[n]: ../articles/active-directory/develop/media/common/no.png

<!--Reference-style links -->

[preview-tos]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/