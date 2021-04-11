---
title: Подключение учетной записи Azure Purview 
description: Подключение учетной записи Azure Purview к рабочей области Synapse.
services: synapse-analytics
author: julieMSFT
ms.service: synapse-analytics
ms.topic: quickstart
ms.date: 12/16/2020
ms.author: jrasnick
ms.reviewer: jrasnick
ms.openlocfilehash: 73ea2f43591d1c7ca2d42591e14b4ff04eb7e496
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "105628634"
---
# <a name="quickstartconnect-an-azure-purview-account-to-a-synapse-workspace"></a>Краткое руководство. Подключение учетной записи Azure Purview к рабочей области Synapse 


В этом кратком руководстве показано, как зарегистрировать учетную запись Azure Purview в рабочей области Synapse. Это подключение позволяет обнаруживать ресурсы Azure Purview и взаимодействовать с ними с помощью возможностей Synapse. 

В Synapse можно выполнять следующие задачи: 
- использовать поле поиска в верхней части окна для поиска ресурсов Purview на основе ключевых слов; 
- изучать данные на основе метаданных, происхождения и заметок; 
- подключать эти данные к рабочей области с помощью связанных служб или наборов данных интеграции; 
- анализировать эти наборы данных с помощью Apache Spark для Synapse, Synapse SQL и средства "Поток данных". 

## <a name="prerequisites"></a>Предварительные требования 
- [учетная запись Azure Purview;](../../purview/create-catalog-portal.md) 
- [рабочая область Synapse.](../quickstart-create-workspace.md) 

## <a name="signin-toa-synapse-workspace"></a>Вход в рабочую область Synapse 

Перейдите на сайт  [https://web.azuresynapse.net](https://web.azuresynapse.net) и войдите в рабочую область. 

## <a name="permissions-for-connecting-an-azure-purview-account"></a>Разрешения для подключения учетной записи Azure Purview 

- Чтобы подключить учетную запись Azure Purview к рабочей области Synapse, в этой рабочей области у вас должна быть роль **Участник**, назначенная в системе IAM на портале Azure, и права на доступ к этой учетной записи Azure Purview. Дополнительные сведения см. в статье о [разрешениях Azure Purview](../../purview/catalog-permissions.md).

## <a name="connect-an-azure-purview-account"></a>Подключение учетной записи Azure Purview  

- В рабочей области Synapse последовательно выберите элементы **Управление** -> **Azure Purview**. Выберите команду **Connect to a Purview account** (Подключиться к учетной записи Purview). 
- Вы можете выбрать вариант **From Azure subscription** (Из подписки Azure) или **Ввести вручную**. **В подписке Azure** можно выбрать учетную запись, к которой у вас есть доступ. 
- После подключения на вкладке **Azure Purview account** (Учетная запись Azure Purview) должно отобразится имя учетной записи Purview. 
- Для поиска данных можно использовать панель поиска, которая отображается вверху по центру рабочей области Synapse. 

## <a name="nextsteps"></a>Дальнейшие действия 

[Регистрация и проверка ресурсов Azure Synapse в Azure Purview](../../purview/register-scan-azure-synapse-analytics.md)

[Обнаружение, подключение и просмотр данных в Synapse с помощью Azure Purview](how-to-discover-connect-analyze-azure-purview.md)   
