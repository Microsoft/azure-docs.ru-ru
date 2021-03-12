---
ms.openlocfilehash: 15e809e1fa361f758a3df7e5b56d4fb80400f656
ms.sourcegitcommit: 225e4b45844e845bc41d5c043587a61e6b6ce5ae
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/11/2021
ms.locfileid: "103007904"
---
| Платформа          | Проект включен<br/>GitHub                                                                          | Пакет                                                                               | Появляться<br/>запущен                    | Выполнение входа пользователей                                         | Доступ к веб-API                                                 | Общедоступная версия (GA) *или*<br/>Общедоступная Предварительная версия<sup>1</sup> |
|-------------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|:--------------------------------------:|:-----------------------------------------------------:|:---------------------------------------------------------------:|:------------------------------------------------------------:|
| Android (Java)    | [MSAL Android](https://github.com/AzureAD/microsoft-authentication-library-for-android)        | [MSAL](https://mvnrepository.com/artifact/com.microsoft.identity.client/msal)         | [Краткое руководство](../articles/active-directory/develop/quickstart-v2-android.md) | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| Android (Котлин)  | [MSAL Android](https://github.com/AzureAD/microsoft-authentication-library-for-android)        | [MSAL](https://mvnrepository.com/artifact/com.microsoft.identity.client/msal)         | —                                      | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| iOS (SWIFT/obj-C) | [MSAL для iOS и MacOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc) | [MSAL](https://cocoapods.org/pods/MSAL)                                               | [Руководство](../articles/active-directory/develop/tutorial-v2-ios.md)         | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| Xamarin (.NET)    | [MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)             | [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client) | —                                      | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
<!--
| React Native |[React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth/blob/main/docs/config-examples/azure-active-directory.md) | [react-native-app-auth](https://www.npmjs.com/package/react-native-app-auth) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
-->

<sup>1</sup> [Дополнительные условия использования для Microsoft Azure предварительных версий][preview-tos] применяются к библиотекам в *общедоступной предварительной версии*.

<!--Image references-->

[y]: ../articles/active-directory/develop/media/common/yes.png
[n]: ../articles/active-directory/develop/media/common/no.png

<!--Reference-style links -->

[preview-tos]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/