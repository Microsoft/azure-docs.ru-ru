---
author: mmacy
ms.author: marsma
ms.date: 11/25/2020
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.openlocfilehash: e54301e01d25fb075325ef34522daa5f1b691dd5
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98760700"
---
## <a name="conditional-access-and-claims-challenges"></a>Условный доступ и запросы утверждений

Если вы получаете токены автоматически, приложение может получать ошибки, когда для используемого API применяются [запросы утверждений условного доступа](../articles/active-directory/develop/v2-conditional-access-dev-guide.md), например политики MFA.

Для обработки этой ошибки следует получить токен в интерактивном режиме с помощью MSAL. При этом пользователю предлагается получить возможность удовлетворить требуемую политику условного доступа.

В некоторых случаях при вызове API, требующих условного доступа, вы можете получать запрос утверждений прямо в полученной от API ошибке. Например, если политика условного доступа требует использовать управляемое устройство (Intune), вы получите ошибку [AADSTS53000: Your device is required to be managed to access this resource](../articles/active-directory/develop/reference-aadsts-error-codes.md) (Требуется управляемое устройство для доступа к этому ресурсу) или что-то подобное. В этом случае вы можете передать утверждения в запросе на получение токена, чтобы пользователь получил приглашение выполнить требования политики.
