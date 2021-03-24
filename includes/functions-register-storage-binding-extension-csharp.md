---
ms.openlocfilehash: 1dd9e377a88d8b35cc5ad43f3e8dd1c48dc35d7c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "78201035"
---
::: zone pivot="programming-language-csharp"  
## <a name="register-binding-extensions"></a>Регистрация расширений привязки

За исключением триггеров HTTP и таймера, привязки реализованы в виде пакетов расширений. Выполните следующую команду [dotnet add package](/dotnet/core/tools/dotnet-add-package) в окне терминала, чтобы добавить пакет расширений службы хранилища в свой проект.

```bash
dotnet add package Microsoft.Azure.WebJobs.Extensions.Storage --version 3.0.4
```

Теперь вы можете добавить выходную привязку службы хранилища к проекту.  
::: zone-end  