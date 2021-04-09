---
title: Параметры подключений Azure Synapse
description: В статье описывается настройка параметров подключений в Azure Synapse Analytics.
author: RonyMSFT
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: security
ms.date: 03/15/2021
ms.author: ronytho
ms.reviewer: jrasnick
ms.openlocfilehash: e0d8a8e3320b49b6fbe3e8ab66c0b4569fac9afd
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104587939"
---
# <a name="azure-synapse-analytics-connectivity-settings"></a>Параметры подключений Azure Synapse Analytics

В этой статье описаны параметры подключений в Azure Synapse Analytics, а также приведены сведения по их настройке.


## <a name="connection-policy"></a>Политика подключения
Для политики подключения Synapse SQL в Azure Synapse Analytics задано значение *По умолчанию*. Вы не можете изменить его в Azure Synapse Analytics. Дополнительные сведения о том, как это влияет на подключения к Synapse SQL в Azure Synapse Analytics, см. [здесь](../../azure-sql/database/connectivity-architecture.md#connection-policy). 

## <a name="minimal-tls-version"></a>Минимальная версия TLS
Synapse SQL в Azure Synapse Analytics позволяет создавать подключения с использованием всех версий TLS. Задать минимальную версию TLS для Synapse SQL в Azure Synapse Analytics нельзя.

## <a name="next-steps"></a>Дальнейшие действия

Создание [рабочей области Azure Synapse](./synapse-workspace-ip-firewall.md).