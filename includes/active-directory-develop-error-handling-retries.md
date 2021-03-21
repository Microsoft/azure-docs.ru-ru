---
author: mmacy
ms.author: marsma
ms.date: 11/25/2020
ms.service: active-directory
ms.subservice: develop
ms.topic: include
ms.openlocfilehash: d0377e3e918a8431c35c1f5b15a1c4b54eeae3b4
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98760639"
---
## <a name="retrying-after-errors-and-exceptions"></a>Повторные попытки после ошибок и исключений

При вызове MSAL требуется реализовать собственные политики повтора. MSAL выполняет HTTP-вызовы к службе Azure AD, и иногда могут возникать сбои. Например, сеть может быть остановлена или сервер перегружен.  

### <a name="http-429"></a>HTTP 429

Когда сервер токенов службы (STS) перегружен из-за слишком большого количества запросов, он возвращает ошибку HTTP 429 с указанием времени ожидания до следующей попытки в поле ответа `Retry-After`.