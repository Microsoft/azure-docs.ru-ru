---
ms.openlocfilehash: 60023f9b97a2ce68cf5689c800080454768ba40d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "103007977"
---
| Язык и платформа | Проект включен<br/>GitHub                                                                                     | Пакет                                                                               | Появляться<br/>запущен                        | Выполнение входа пользователей                                         | Доступ к веб-API                                                 | Общедоступная версия (GA) *или*<br/>Общедоступная Предварительная версия<sup>1</sup> |
|----------------------|-----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------|:------------------------------------------:|:-----------------------------------------------------:|:---------------------------------------------------------------:|:------------------------------------------------------------:|
| Electron             | [MSAL Node.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-node) | [@azure/msal-node](https://www.npmjs.com/package/@azure/msal-node)                    | —                                          | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | Общедоступная предварительная версия                                               |
| Java                 | [MSAL4J](https://github.com/AzureAD/microsoft-authentication-library-for-java)                            | [msal4j](https://mvnrepository.com/artifact/com.microsoft.azure/msal4j)               | —                                          | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| macOS (SWIFT/obj-C)  | [MSAL для iOS и MacOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc)            | [MSAL](https://cocoapods.org/pods/MSAL)                                               | [Руководство](../articles/active-directory/develop/tutorial-v2-ios.md)             | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| UWP                  | [MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)                        | [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client) | [Руководство](../articles/active-directory/develop/tutorial-v2-windows-uwp.md)     | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
| WPF                  | [MSAL.NET](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet)                        | [Microsoft.Identity.Client](https://www.nuget.org/packages/Microsoft.Identity.Client) | [Руководство](../articles/active-directory/develop/tutorial-v2-windows-desktop.md) | ![Библиотека может запросить маркеры идентификатора для входа пользователя.][y] | ![Библиотека может запрашивать маркеры доступа для защищенных веб-API.][y] | GA                                                           |
<!--
| Java | Scribe | [Scribe Java](https://mvnrepository.com/artifact/org.scribe/scribe) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
| React Native | [React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth/blob/main/docs/config-examples/azure-active-directory.md) | [react-native-app-auth](https://www.npmjs.com/package/react-native-app-auth) | ![X indicating no.][n] | ![Green check mark.][y] | ![Green check mark.][y] | -- |
-->

<sup>1</sup> [Дополнительные условия использования для Microsoft Azure предварительных версий][preview-tos] применяются к библиотекам в *общедоступной предварительной версии*.

<!--Image references-->

[y]: ../articles/active-directory/develop/media/common/yes.png
[n]: ../articles/active-directory/develop/media/common/no.png


<!--Reference-style links -->

[preview-tos]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/