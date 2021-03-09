---
title: Написание кода проверки подлинности приложения
titleSuffix: Azure Digital Twins
description: См. статью как написать код проверки подлинности в клиентском приложении.
author: baanders
ms.author: baanders
ms.date: 10/7/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 1b7a846ee92da001ea2ac3ddd02efa9a870f72c6
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102501912"
---
# <a name="write-client-app-authentication-code"></a>Запись кода проверки подлинности клиентского приложения

После [настройки экземпляра и проверки подлинности Azure Digital двойников](how-to-set-up-instance-portal.md)можно создать клиентское приложение, которое будет использоваться для взаимодействия с экземпляром. После настройки проекта начального клиента необходимо **написать код в этом клиентском приложении для проверки подлинности** в экземпляре Azure Digital двойников.

Azure Digital двойников выполняет проверку подлинности с помощью [маркеров безопасности Azure AD на основе OAUTH 2,0](../active-directory/develop/security-tokens.md#json-web-tokens-and-claims). Чтобы проверить подлинность пакета SDK, необходимо получить маркер носителя с нужными разрешениями для Azure Digital двойников и передать его вместе с вызовами API. 

В этой статье описывается, как получить учетные данные с помощью `Azure.Identity` клиентской библиотеки. Хотя в этой статье показаны примеры кода для [пакета SDK для .NET (C#)](/dotnet/api/overview/azure/digitaltwins/client), можно использовать версию независимо от того, `Azure.Identity` какой пакет SDK вы используете (Дополнительные сведения см. в разделе с [*инструкциями по использованию интерфейсов API цифровых двойников и пакетов SDK*](how-to-use-apis-sdks.md)для Azure Digital двойников).

## <a name="prerequisites"></a>Предварительные требования

Сначала выполните действия по настройке, описанные в разделе [*инструкции. Настройка экземпляра и аутентификации*](how-to-set-up-instance-portal.md). Это обеспечит наличие экземпляра цифрового двойников Azure и наличие у пользователя разрешений на доступ. После этого программа установки будет готова к написанию кода клиентского приложения.

Для продолжения Вам потребуется проект клиентского приложения, в котором будет написан код. Если вы еще не настроили проект клиентского приложения, создайте базовый проект на выбранном вами языке для использования с этим руководством.

## <a name="common-authentication-methods-with-azureidentity"></a>Общие методы проверки подлинности с помощью Azure. Identity

`Azure.Identity` — это клиентская библиотека, которая предоставляет несколько методов получения учетных данных, которые можно использовать для получения маркера носителя и проверки подлинности с помощью пакета SDK. Хотя в этой статье приведены примеры в C#, можно просматривать `Azure.Identity` несколько языков, включая...

* [.NET (C#)](/dotnet/api/azure.identity)
* [Java](/java/api/overview/azure/identity-readme)
* [JavaScript](/javascript/api/overview/azure/identity-readme)
* [Python](/python/api/overview/azure/identity-readme)

Три общих метода получения учетных данных в `Azure.Identity` :

* [Дефаултазурекредентиал](/dotnet/api/azure.identity.defaultazurecredential) предоставляет `TokenCredential` поток проверки подлинности по умолчанию для приложений, которые будут развернуты в Azure. это **рекомендуемый вариант для локальной разработки**. Также можно включить другие два метода, которые рекомендуются в этой статье. Он является оболочкой `ManagedIdentityCredential` и может иметь доступ к `InteractiveBrowserCredential` переменной конфигурации.
* [Манажедидентитикредентиал](/dotnet/api/azure.identity.managedidentitycredential) работает отлично в тех случаях, когда вам нужны [управляемые удостоверения (MSI)](../active-directory/managed-identities-azure-resources/overview.md), и это хороший кандидат на работу с функциями Azure и развертывание в службах Azure.
* [Интерактивебровсеркредентиал](/dotnet/api/azure.identity.interactivebrowsercredential) предназначен для интерактивных приложений и может использоваться для создания клиента SDK с проверкой подлинности

В следующем примере показано, как использовать каждый из них с пакетом SDK для .NET (C#).

## <a name="authentication-examples-net-c-sdk"></a>Примеры проверки подлинности: пакет SDK для .NET (C#)

В этом разделе показан пример в C# для использования предоставленного пакета SDK для .NET для написания кода проверки подлинности.

Сначала включите пакет SDK `Azure.DigitalTwins.Core` и `Azure.Identity` пакет в проект. В зависимости от выбранных вами средств можно включить пакеты с помощью диспетчера пакетов Visual Studio или `dotnet` средства командной строки. 

Кроме того, в код проекта необходимо добавить следующие операторы using:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="Azure_Digital_Twins_dependencies":::

Затем добавьте код для получения учетных данных с помощью одного из методов в `Azure.Identity` .

### <a name="defaultazurecredential-method"></a>Метод Дефаултазурекредентиал

[Дефаултазурекредентиал](/dotnet/api/azure.identity.defaultazurecredential) предоставляет `TokenCredential` поток проверки подлинности по умолчанию для приложений, которые будут развернуты в Azure. это **рекомендуемый вариант для локальной разработки**.

Чтобы использовать учетные данные Azure по умолчанию, вам потребуется URL-адрес экземпляра Azure Digital двойников ([инструкции по поиску](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)).

Ниже приведен пример кода для добавления в `DefaultAzureCredential` проект.

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="DefaultAzureCredential_full":::

#### <a name="set-up-local-azure-credentials"></a>Настройка локальных учетных данных Azure

[!INCLUDE [Azure Digital Twins: local credentials prereq (inner)](../../includes/digital-twins-local-credentials-inner.md)]

### <a name="managedidentitycredential-method"></a>Метод Манажедидентитикредентиал

Метод [манажедидентитикредентиал](/dotnet/api/azure.identity.managedidentitycredential) работает отлично в тех случаях, когда вам требуются [управляемые удостоверения (MSI)](../active-directory/managed-identities-azure-resources/overview.md), например при работе с функциями Azure.

Это означает, что можно использовать `ManagedIdentityCredential` в том же проекте, что `DefaultAzureCredential` и или `InteractiveBrowserCredential` , для проверки подлинности другой части проекта.

Чтобы использовать учетные данные Azure по умолчанию, вам потребуется URL-адрес экземпляра Azure Digital двойников ([инструкции по поиску](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values)).

В функции Azure можно использовать учетные данные управляемого удостоверения следующим образом:

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="ManagedIdentityCredential":::

### <a name="interactivebrowsercredential-method"></a>Метод Интерактивебровсеркредентиал

Метод [интерактивебровсеркредентиал](/dotnet/api/azure.identity.interactivebrowsercredential) предназначен для интерактивных приложений и позволяет открыть веб-браузер для проверки подлинности. Это можно использовать вместо `DefaultAzureCredential` в тех случаях, когда требуется Интерактивная проверка подлинности.

Чтобы использовать интерактивные учетные данные браузера, потребуется **Регистрация приложения** , имеющего разрешения на доступ к API-интерфейсам цифрового двойников Azure. Инструкции по настройке регистрации приложения см. в разделе [*инструкции. Создание регистрации приложения*](how-to-create-app-registration.md). После настройки регистрации приложения вам потребуется...
* *идентификатор приложения (клиента)* регистрации приложения ([инструкции для поиска](how-to-create-app-registration.md#collect-client-id-and-tenant-id))
* *идентификатор каталога (клиента)* регистрации приложения ([инструкции для поиска](how-to-create-app-registration.md#collect-client-id-and-tenant-id))
* URL-адрес экземпляра Azure Digital двойников ([инструкции для поиска](how-to-set-up-instance-portal.md#verify-success-and-collect-important-values))

Ниже приведен пример кода для создания клиента пакета SDK с проверкой подлинности с помощью `InteractiveBrowserCredential` .

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/authentication.cs" id="InteractiveBrowserCredential":::

>[!NOTE]
> Хотя идентификатор клиента, идентификатор клиента и URL-адрес экземпляра можно разместить непосредственно в коде, как показано выше, рекомендуется заставить код получать эти значения из файла конфигурации или переменной среды.

#### <a name="other-notes-about-authenticating-azure-functions"></a>Другие примечания о проверке подлинности функций Azure

См. [*инструкции по настройке функции Azure для обработки данных*](how-to-create-azure-function.md) для более полного примера, в котором объясняются некоторые важные варианты настройки в контексте функций.

Кроме того, чтобы использовать проверку подлинности в функции, не забудьте:
* [Включение управляемого удостоверения](../app-service/overview-managed-identity.md?tabs=dotnet)
* Используйте [переменные среды](/sandbox/functions-recipes/environment-variables?tabs=csharp) соответствующим образом
* Назначьте разрешения приложению "функции", которое позволяет ему получить доступ к интерфейсам API цифровых двойников. Дополнительные сведения о процессах функций Azure см. [*в разделе Практические руководства. Настройка функции Azure для обработки данных*](how-to-create-azure-function.md).

## <a name="other-credential-methods"></a>Другие методы учетных данных

Если выделенные выше сценарии проверки подлинности не охватывают потребности приложения, можно изучить другие типы проверки подлинности, предлагаемые на [**платформе Microsoft Identity**](../active-directory/develop/v2-overview.md#getting-started). Документация по этой платформе охватывает дополнительные сценарии проверки подлинности, упорядоченные по типам приложений.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о работе системы безопасности в Azure Digital двойников.
* [*Основные понятия: безопасность решений для цифровых двойников Azure*](concepts-security.md)

Кроме того, теперь, когда проверка подлинности настроена, переходите к созданию моделей и управлению ими в экземпляре.
* [*Руководство. Управление моделями ДТДЛ*](how-to-manage-model.md)