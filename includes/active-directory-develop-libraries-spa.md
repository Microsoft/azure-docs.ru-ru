---
ms.openlocfilehash: a393cf978318c39081805cae7e38c3386ff156f5
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103010713"
---
| Язык и платформа | Проект включен<br/>GitHub                                                                                                    | Пакет                                                                      | Появляться<br/>запущен                             | Выполнение входа пользователей                                         | Доступ к веб-API                                                 | Общедоступная версия (GA) *или*<br/>Общедоступная Предварительная версия<sup>1</sup> |
|----------------------|--------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|:-----------------------------------------------:|:-----------------------------------------------------:|:---------------------------------------------------------------:|:------------------------------------------------------------:|
| Angular              | [MSAL, угловой](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)<sup>2</sup> v2         | [@azure/msal-angular](https://www.npmjs.com/package/@azure/msal-angular)     | —                                               | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | Общедоступная предварительная версия                                               |
| Angular              | [MSAL, угловой](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/msal-angular-v1/lib/msal-angular)<sup>3</sup> | [@azure/msal-angular](https://www.npmjs.com/package/@azure/msal-angular)     |[Руководство](../articles/active-directory/develop/tutorial-v2-angular.md)| ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| AngularJS            | [MSAL AngularJS](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angularjs)<sup>3</sup>         | [@azure/msal-angularjs](https://www.npmjs.com/package/@azure/msal-angularjs) | —                                               | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | Общедоступная предварительная версия                                               |
| JavaScript           | [MSAL.js v2](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)<sup>2</sup>              | [@azure/msal-browser](https://www.npmjs.com/package/@azure/msal-browser)     | [Руководство](../articles/active-directory/develop/tutorial-v2-javascript-auth-code.md) | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
|JavaScript|[MSAL.js 1,0](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core)<sup>3</sup> | [@azure/msal-core](https://www.npmjs.com/package/@azure/msal-core)    | [Руководство](../articles/active-directory/develop/tutorial-v2-javascript-spa.md)| ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| React                | [MSAL отреагировать](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-react)<sup>2</sup>                 | [@azure/msal-react](https://www.npmjs.com/package/@azure/msal-react)         | —                                               | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | Общедоступная предварительная версия                                               |
<!--
| Vue | [Vue MSAL]( https://github.com/mvertopoulos/vue-msal) | [vue-msal]( https://www.npmjs.com/package/vue-msal) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
-->

<sup>1</sup> [Дополнительные условия использования для Microsoft Azure предварительных версий][preview-tos] применяются к библиотекам в *общедоступной предварительной версии*.

<sup>2</sup> [поток кода проверки подлинности][auth-code-flow] только с пкке (рекомендуется). 

<sup>3</sup> [неявный поток предоставления разрешений][implicit-flow] .

<!--Image references-->

[y]: ../articles/active-directory/develop/media/common/yes.png
[n]: ../articles/active-directory/develop/media/common/no.png

<!--Reference-style links -->
[AAD-App-Model-V2-Overview]: v2-overview.md
[Microsoft-SDL]: https://www.microsoft.com/securityengineering/sdl/
[preview-tos]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[auth-code-flow]: ../articles/active-directory/develop/v2-oauth2-auth-code-flow.md
[implicit-flow]: ../articles/active-directory/develop/v2-oauth2-implicit-grant-flow.md