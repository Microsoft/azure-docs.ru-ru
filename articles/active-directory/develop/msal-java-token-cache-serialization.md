---
title: Сериализация кэша пользовательской лексемы (MSAL4j)
titleSuffix: Microsoft identity platform
description: Сведения о сериализации кэша маркеров для MSAL для Java
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 11/07/2019
ms.author: sagonzal
ms.reviewer: nacanuma
ms.custom: aaddev, devx-track-java
ms.openlocfilehash: e85cfb8c03111b889a9f95509f337d415c517163
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87312373"
---
# <a name="custom-token-cache-serialization-in-msal-for-java"></a>Сериализация пользовательского кэша маркеров в MSAL для Java

Чтобы сохранить кэш маркеров между экземплярами приложения, необходимо настроить сериализацию. Классы и интерфейсы Java, участвующие в сериализации кэша маркеров, следующие:

- [Итокенкаче](https://static.javadoc.io/com.microsoft.azure/msal4j/0.5.0-preview/com/microsoft/aad/msal4j/ITokenCache.html): интерфейс, представляющий кэш маркеров безопасности.
- [Итокенкачеакцессаспект](https://static.javadoc.io/com.microsoft.azure/msal4j/0.5.0-preview/com/microsoft/aad/msal4j/ITokenCacheAccessAspect.html): интерфейс, представляющий операцию выполнения кода до и после доступа. Вы бы @Override *Бефорекачеакцесс* и *афтеркачеакцесс* с логикой, отвечающей за сериализацию и десериализацию кэша.
- [Итокенкачеконтекст](https://static.javadoc.io/com.microsoft.azure/msal4j/0.5.0-preview/com/microsoft/aad/msal4j/ITokenCacheAccessContext.html): интерфейс, представляющий контекст, в котором осуществляется доступ к кэшу маркера. 

Ниже приведена упрощенная реализация пользовательской сериализации сериализации и десериализации кэша маркеров. Не копируйте и не вставляйте его в рабочую среду.

```Java
static class TokenPersistence implements ITokenCacheAccessAspect {
String data;

TokenPersistence(String data) {
        this.data = data;
}

@Override
public void beforeCacheAccess(ITokenCacheAccessContext iTokenCacheAccessContext) {
        iTokenCacheAccessContext.tokenCache().deserialize(data);
}

@Override
public void afterCacheAccess(ITokenCacheAccessContext iTokenCacheAccessContext) {
        data = iTokenCacheAccessContext.tokenCache().serialize();
}
```

```Java
// Loads cache from file
String dataToInitCache = readResource(this.getClass(), "/cache_data/serialized_cache.json");

ITokenCacheAccessAspect persistenceAspect = new TokenPersistence(dataToInitCache);

// By setting *TokenPersistence* on the PublicClientApplication, MSAL will call *beforeCacheAccess()* before accessing the cache and *afterCacheAccess()* after accessing the cache. 
PublicClientApplication app = 
PublicClientApplication.builder("my_client_id").setTokenCacheAccessAspect(persistenceAspect).build();
```

## <a name="learn-more"></a>Дополнительные сведения

Узнайте [, как получать и удалять учетные записи из кэша маркеров с помощью MSAL для Java](msal-java-get-remove-accounts-token-cache.md).
