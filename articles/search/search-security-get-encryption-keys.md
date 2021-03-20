---
title: Получение сведений о ключе шифрования
titleSuffix: Azure Cognitive Search
description: Получите имя и версию ключа шифрования, используемые в индексе или сопоставлении синонимов, чтобы можно было управлять ключом в Azure Key Vault.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/01/2020
ms.openlocfilehash: 37ff94608e9756142f70a4f3c64d0a6f7eeea685
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "88932905"
---
# <a name="get-customer-managed-key-information-from-indexes-and-synonym-maps"></a>Получение сведений о ключах, управляемых клиентом, из индексов и карт синонимов

В Когнитивный поиск Azure ключи шифрования, управляемые клиентом, создаются, хранятся и управляются в Azure Key Vault. Если необходимо определить, зашифрован ли объект или использовалось имя или версия ключа, используйте REST API или пакет SDK для получения свойства **encryptionKey** из определения индекса или схемы синонимов. 

Рекомендуется [включить ведение журнала](../key-vault/general/logging.md) на Key Vault, чтобы можно было отслеживать использование ключа.

## <a name="get-the-admin-api-key"></a>Получение ключа API администратора

Чтобы получить определения объектов из службы поиска, необходимо пройти проверку подлинности с правами администратора. Самый простой способ получить ключ API администратора — через портал.

1. Войдите в [портал Azure](https://portal.azure.com/) и откройте страницу Обзор службы поиска.

1. В левой части щелкните **ключи** и скопируйте API администратора. Для получения индекса и поиска синонимов требуется ключ администратора.

Для выполнения остальных шагов переключитесь на PowerShell и REST API. На портале не отображаются сопоставления синонимов и свойства ключей шифрования индексов.

## <a name="use-powershell-and-rest"></a>Использование PowerShell и других компонентов

Выполните следующие команды, чтобы настроить переменные и получить определения объектов.

```powershell
<# Connect to Azure #>
$Connect-AzAccount

<# Provide the admin API key used for search service authentication  #>
$headers = @{
'api-key' = '<YOUR-ADMIN-API-KEY>'
'Content-Type' = 'application/json'
'Accept' = 'application/json' }

<# List all existing synonym maps #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/synonyms?api-version=2020-06-30&$select=name'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# List all existing indexes #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes?api-version=2020-06-30&$select=name'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# Return a specific synonym map definition. The encryptionKey property is at the end #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/synonyms/<YOUR-SYNONYM-MAP-NAME>?api-version=2020-06-30'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json

<# Return a specific index definition. The encryptionKey property is at the end #>
$uri= 'https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/<YOUR-INDEX-NAME>?api-version=2020-06-30'
Invoke-RestMethod -Uri $uri -Headers $headers | ConvertTo-Json
```

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда известно, какой ключ и версия шифрования используются, можно управлять ключом в Azure Key Vault или проверять другие параметры конфигурации.

+ [Краткое руководство. Настройка и получение секрета из Azure Key Vault с помощью PowerShell](../key-vault/secrets/quick-create-powershell.md)

+ [Настройка ключей, управляемых клиентом, для шифрования данных в Azure Когнитивный поиск](search-security-manage-encryption-keys.md)