---
title: Общий доступ к ресурсам независимо от источника (CORS) в Azure Cosmos DB
description: В этой статье описано, как настроить общий доступ к ресурсам независимо от источника (CORS) в Azure Cosmos DB с помощью портала Azure и шаблонов Azure Resource Manager.
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/11/2019
ms.author: dech
ms.openlocfilehash: eba49ff45ba9ab1f5cfaa1d75973d656ac32ca6a
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93339909"
---
# <a name="configure-cross-origin-resource-sharing-cors"></a>Настройка общего доступа к ресурсам независимо от источника (CORS)
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Общий доступ к ресурсам независимо от источника (CORS) — функция HTTP, которая позволяет веб-приложению, работающему в одном домене, обращаться к ресурсам другого домена. Веб-браузеры реализуют ограничение безопасности, известное как политика одного источника, которая не позволяет веб-странице вызывать API-интерфейсы в другом домене. Однако CORS обеспечивает безопасный способ, позволяющий исходному домену вызывать интерфейсы API в другом домене. API ядра (SQL) в Azure Cosmos DB теперь поддерживает общий доступ к ресурсам между источниками (CORS) с помощью заголовка "allowedOrigins". При включении поддержки CORS для учетной записи Azure Cosmos только прошедшие проверку подлинности запросы оцениваются, чтобы определить, разрешены ли они в соответствии с правилами, которые вы указали.

Вы можете настроить общий доступ независимо от источника (CORS) на портале Azure или на основе шаблона Azure Resource Manager. Для учетных записей Cosmos, использующих API ядра (SQL), Azure Cosmos DB поддерживает библиотеку JavaScript, которая работает как в Node.js, так и в среде на основе браузера. Теперь для этой библиотеки доступны преимущества поддержки CORS при использовании режима шлюза. Для использования этой функции не нужно ничего настраивать на стороне клиента. С поддержкой CORS ресурсы из браузера могут напрямую обращаться к Azure Cosmos DB через [библиотеку JavaScript](https://www.npmjs.com/package/@azure/cosmos) или непосредственно из [REST API](/rest/api/cosmos-db/) для простых операций.

> [!NOTE]
> Поддержка CORS применима и поддерживается для API Azure Cosmos DB Core (SQL). Он неприменим к Azure Cosmos DBным API-интерфейсам для Cassandra, Gremlin или MongoDB, так как эти протоколы не используют HTTP для обмена данными между клиентом и сервером.

## <a name="enable-cors-support-from-azure-portal"></a>Включение поддержки CORS на портале Azure

Чтобы включить общий доступ к ресурсам независимо от источника с помощью портала Azure, следуйте инструкциям ниже:

1. Перейдите к своей учетной записи Azure Cosmos DB. Откройте колонку **CORS**.

2. Укажите через запятую список источников, которые могут совершать вызовы независимо от источника в учетную запись Cosmos DB. Например `https://www.mydomain.com`, `https://mydomain.com`, `https://api.mydomain.com`. Вы также можете использовать подстановочный знак "\*", чтобы разрешить использование всех источников, и выбрать **Отправить**. 

   > [!NOTE]
   > В настоящее время в имени домена нельзя использовать подстановочные знаки. Например, формат `https://*.mydomain.net` еще не поддерживается. 

   :::image type="content" source="./media/how-to-configure-cross-origin-resource-sharing/enable-cross-origin-resource-sharing-using-azure-portal.png" alt-text="Включение общего доступа к ресурсам независимо от источника с помощью портала Azure":::

## <a name="enable-cors-support-from-resource-manager-template"></a>Включение поддержки CORS из шаблона Resource Manager

Чтобы включить CORS с помощью шаблона Resource Manager, добавьте раздел cors со свойством allowedOrigins в любой имеющийся шаблон. Приведенный ниже код JSON является примером шаблона, который создает учетную запись Azure Cosmos с включенной поддержкой CORS.

```json
{
  "type": "Microsoft.DocumentDB/databaseAccounts",
  "name": "[variables('accountName')]",
  "apiVersion": "2019-08-01",
  "location": "[parameters('location')]",
  "kind": "GlobalDocumentDB",
  "properties": {
    "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
    "locations": "[variables('locations')]",
    "databaseAccountOfferType": "Standard",
    "cors": [
      {
        "allowedOrigins": "*"
      }
    ]
  }
}
```

## <a name="using-the-azure-cosmos-db-javascript-library-from-a-browser"></a>Использование библиотеки Azure Cosmos DB JavaScript в браузере

Сейчас в библиотеке Azure Cosmos DB JavaScript доступна только библиотека версии CommonJS, которая поставляется в составе пакета. Чтобы использовать эту библиотеку из браузера, необходимо использовать такое средство, как Rollup или Webpack для создания Библиотеки, совместимой с браузером. Для некоторых библиотек Node.js нужны макеты браузера. Ниже приведен пример файла конфигурации webpack, который содержит необходимые настройки макетов.

```javascript
const path = require("path");

module.exports = {
  entry: "./src/index.ts",
  devtool: "inline-source-map",
  node: {
    net: "mock",
    tls: "mock"
  },
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, "dist")
  }
};
```
 
Вот [пример кода](https://github.com/christopheranderson/cosmos-browser-sample), в котором используется TypeScript и Webpack с библиотекой пакета SDK Azure Cosmos DB для JavaScript для создания приложения Todo, которое отправляет обновления в режиме реального времени при создании элементов.
Рекомендуется не использовать первичный ключ для взаимодействия с Azure Cosmos DB из браузера. Вместо этого используйте маркеры ресурсов для обмена данными. Дополнительные сведения о маркерах ресурсов см. в разделе [Защита доступа к Azure Cosmos DB](secure-access-to-data.md#resource-tokens).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о других способах защиты вашей учетной записи Azure Cosmos см. в следующих статьях:

* [Настройка брандмауэра IP-адресов для учетной записи Azure Cosmos DB](how-to-configure-firewall.md)

* [Настройка доступа на основе подсети и виртуальной сети для учетной записи Azure Cosmos DB](how-to-configure-vnet-service-endpoint.md)
