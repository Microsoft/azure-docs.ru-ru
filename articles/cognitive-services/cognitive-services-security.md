---
title: Безопасность Cognitive Services Azure
titleSuffix: Azure Cognitive Services
description: Узнайте о различных вопросах безопасности, касающихся Cognitive Services использования.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: erhopf
ms.custom: devx-track-python, devx-track-js, devx-track-csharp
ms.openlocfilehash: 90fbc41edbfea0ea99d436906d3052e43bfb4fb8
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105043885"
---
# <a name="azure-cognitive-services-security"></a>Безопасность Cognitive Services Azure

Безопасность следует рассматривать как высший приоритет при разработке любых приложений. С SES приложений с поддержкой искусственного интеллекта безопасность еще важнее. В этой статье описываются различные аспекты безопасности Azure Cognitive Services, такие как использование безопасности транспортного уровня, проверка подлинности, безопасная настройка конфиденциальных данных и защищенное хранилище доступа к данным клиентов.

## <a name="transport-layer-security-tls"></a>Протокол TLS

Все конечные точки Cognitive Services, предоставляемые по протоколу HTTP, обеспечивают TLS 1,2. При использовании принудительного протокола безопасности потребители, пытающиеся вызвать конечную точку Cognitive Services, должны соблюдать следующие рекомендации:

* Клиентская операционная система должна поддерживать TLS 1,2
* В языке (и платформе), используемом для вызова HTTP, необходимо указать TLS 1,2 в качестве части запроса.
  * В зависимости от языка и платформы, указание протокола TLS выполняется явно или неявно.

Для пользователей .NET рассмотрите рекомендации по <a href="/dotnet/framework/network-programming/tls" target="_blank">обеспечению безопасности транспортного уровня </a>.

## <a name="authentication"></a>Аутентификация

При обсуждении проверки подлинности существует несколько распространенных заблуждений. Проверка подлинности и авторизация часто путают друг с другом. Удостоверение является также основным компонентом в системе безопасности. Удостоверение — это коллекция сведений об <a href="https://en.wikipedia.org/wiki/Principal_(computer_security)" target="_blank">участнике </a>. Поставщики удостоверений (IdP) предоставляют удостоверения для служб проверки подлинности. Проверка подлинности — это процедура проверки удостоверения пользователя. Авторизация — это спецификация прав доступа и привилегий на ресурсы для заданного удостоверения. Некоторые из Cognitive Services предложений включают управление доступом на основе ролей Azure (Azure RBAC). Azure RBAC можно использовать для упрощения некоторых формальностей, участвующих в управлении участниками вручную. Дополнительные сведения см. в статье [Управление доступом на основе ролей Azure для ресурсов Azure](../role-based-access-control/overview.md).

Дополнительные сведения о проверке подлинности с помощью ключей подписки, маркеров доступа и Azure Active Directory (AAD) см. <a href="/azure/cognitive-services/authentication" target="_blank">в статье Проверка подлинности запросов в Azure Cognitive Services</a>.

## <a name="environment-variables-and-application-configuration"></a>Переменные среды и конфигурация приложения

Переменные среды — это пары "имя-значение", хранящиеся в определенной среде. Более безопасной альтернативой использованию жестко зафиксированных значений для конфиденциальных данных является использование переменных среды. Жестко заданные значения являются небезопасными, и их следует избегать.

> [!CAUTION]
> **Не** используйте жестко заданные значения для конфиденциальных данных, так как это является основной уязвимостью безопасности.

> [!NOTE]
> Хотя переменные среды хранятся в виде обычного текста, они изолированы от среды. Если окружение скомпрометировано, это также относится к переменным среды.

### <a name="set-environment-variable"></a>Задать переменную среды

Чтобы задать переменные среды, используйте одну из следующих команд, где `ENVIRONMENT_VARIABLE_KEY` — это именованный ключ, а `value` — значение, хранящееся в переменной среды.

# <a name="command-line"></a>[Командная строка](#tab/command-line)

Создайте и назначьте сохраненную переменную среды с учетом значения.

```CMD
:: Assigns the env var to the value
setx ENVIRONMENT_VARIABLE_KEY="value"
```

В новом экземпляре **командной строки** считайте эту переменную среды.

```CMD
:: Prints the env var value
echo %ENVIRONMENT_VARIABLE_KEY%
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Создайте и назначьте сохраненную переменную среды с учетом значения.

```powershell
# Assigns the env var to the value
[System.Environment]::SetEnvironmentVariable('ENVIRONMENT_VARIABLE_KEY', 'value', 'User')
```

В новом экземпляре **Windows PowerShell** прочтите переменную среды.

```powershell
# Prints the env var value
[System.Environment]::GetEnvironmentVariable('ENVIRONMENT_VARIABLE_KEY')
```

# <a name="bash"></a>[Bash](#tab/bash)

Создайте и назначьте сохраненную переменную среды с учетом значения.

```Bash
# Assigns the env var to the value
echo export ENVIRONMENT_VARIABLE_KEY="value" >> /etc/environment && source /etc/environment
```

В новом экземпляре **Bash** прочтите переменную среды.

```Bash
# Prints the env var value
echo "${ENVIRONMENT_VARIABLE_KEY}"

# Or use printenv:
# printenv ENVIRONMENT_VARIABLE_KEY
```

---

> [!TIP]
> После настройки переменной среды перезапустите интегрированную среду разработки, чтобы гарантировать доступность добавленных переменных среды.

### <a name="get-environment-variable"></a>Получить переменную среды

Чтобы получить переменную среды, ее необходимо прочитать в памяти. В зависимости от языка, который вы используете, рассмотрим приведенные ниже фрагменты кода. В этих фрагментах кода показано, как получить переменную среды, заданную `ENVIRONMENT_VARIABLE_KEY` и присвоить переменной с именем `value` .

# <a name="c"></a>[C#](#tab/csharp)

Дополнительные сведения см. в <a href="/dotnet/api/system.environment.getenvironmentvariable" target="_blank"> `Environment.GetEnvironmentVariable` </a>разделе.

```csharp
using static System.Environment;

class Program
{
    static void Main()
    {
        // Get the named env var, and assign it to the value variable
        var value =
            GetEnvironmentVariable(
                "ENVIRONMENT_VARIABLE_KEY");
    }
}
```

# <a name="c"></a>[C++](#tab/cpp)

Дополнительные сведения см. в <a href="/cpp/c-runtime-library/reference/getenv-wgetenv" target="_blank"> `getenv` </a>разделе.

```cpp
#include <stdlib.h>

int main()
{
    // Get the named env var, and assign it to the value variable
    auto value =
        getenv("ENVIRONMENT_VARIABLE_KEY");
}
```

# <a name="java"></a>[Java](#tab/java)

Дополнительные сведения см. в <a href="https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#getenv(java.lang.String)" target="_blank"> `System.getenv` </a>разделе.

```java
import java.lang.*;

public class Program {
   public static void main(String[] args) throws Exception {
    // Get the named env var, and assign it to the value variable
    String value =
        System.getenv(
            "ENVIRONMENT_VARIABLE_KEY")
   }
}
```

# <a name="nodejs"></a>[Node.js](#tab/node-js)

Дополнительные сведения см. в <a href="https://nodejs.org/api/process.html#process_process_env" target="_blank"> `process.env` </a>разделе.

```javascript
// Get the named env var, and assign it to the value variable
const value =
    process.env.ENVIRONMENT_VARIABLE_KEY;
```

# <a name="python"></a>[Python](#tab/python)

Дополнительные сведения см. в <a href="https://docs.python.org/2/library/os.html#os.environ" target="_blank"> `os.environ` </a>разделе.

```python
import os

# Get the named env var, and assign it to the value variable
value = os.environ['ENVIRONMENT_VARIABLE_KEY']
```

# <a name="objective-c"></a>[Objective-C](#tab/objective-c)

Дополнительные сведения см. в <a href="https://developer.apple.com/documentation/foundation/nsprocessinfo/1417911-environment?language=objc" target="_blank"> `environment` </a>разделе.

```objectivec
// Get the named env var, and assign it to the value variable
NSString* value =
    [[[NSProcessInfo processInfo]environment]objectForKey:@"ENVIRONMENT_VARIABLE_KEY"];
```

---

## <a name="customer-lockbox"></a>Защищенное хранилище клиента

[Защищенное хранилище для Microsoft Azure](../security/fundamentals/customer-lockbox-overview.md) предоставляет пользователям интерфейс для просмотра и утверждения или отклонения запросов на доступ к данным клиентов. Этот интерфейс позволяет специалисту Майкрософт получить доступ к данным клиента в рамках процессов поддержки. Сведения о инициировании, отслеживании и хранении запросов защищенное хранилище для последующего просмотра и аудита см. в разделе [защищенное хранилище](../security/fundamentals/customer-lockbox-overview.md). 

Защищенное хранилище доступна для этой службы:

* Переводчик

Для следующих служб инженеры Майкрософт не будут получать доступ к данным клиентов на уровне E0: 

* Распознавание речи
* Распознавание лиц
* Content Moderator
* Персонализатор

> [!IMPORTANT]
> Для **распознавателя форм** специалисты Майкрософт не будут получать доступ к данным клиентов в ресурсах, созданных после 10 июля 2020.

Чтобы запросить возможность использования номера SKU E0, заполните и отправьте эту [форму запроса](https://aka.ms/cogsvc-cmk). Для получения сведений о состоянии вашего запроса потребуется около 3-5 рабочих дней. В зависимости от спроса вы можете поместить в очередь и утвердить, как только пространство станет доступным. После утверждения для использования SKU E0 с LUIS необходимо создать новый ресурс на основе портал Azure и выбрать в качестве ценовой категории значение E0. Пользователи не смогут выполнить обновление с F0 на новый SKU E0.

Служба "речь" в настоящее время не поддерживает защищенное хранилище. Тем не менее данные клиентов можно хранить с помощью функции "BYOS", что позволяет использовать аналогичные элементы управления данными для защищенное хранилище. Помните, что данные службы речи остаются и обрабатываются в регионе, где был создан речевой ресурс. Это относится к любым неоставшимся данным и передаче данных. При использовании функций настройки, например Пользовательское распознавание речи и пользовательского голоса, все данные клиента передаются, сохраняются и обрабатываются в том же регионе, где находятся ресурсы BYOS (если используется) и служба речи.

> [!IMPORTANT]
> Корпорация Майкрософт **не** использует данные клиентов для улучшения своих речевых моделей. Кроме того, если ведение журнала конечных точек отключено и никакие настройки не используются, данные клиента не сохраняются. 

## <a name="next-steps"></a>Дальнейшие действия

* Изучите различные [Cognitive Services](./what-are-cognitive-services.md)
* Дополнительные сведения о [Cognitive Services виртуальных сетях](cognitive-services-virtual-networks.md)