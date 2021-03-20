---
title: Доступ к репозиторию GitHub, мониторинг и управление им
description: Отслеживайте события GitHub и управляйте своим репозиторием GitHub, создавая автоматические рабочие процессы с Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: conceptual
ms.date: 03/02/2018
tags: connectors
ms.openlocfilehash: b67d2e860e8c69f6f2f43441144ef2f60ed3a104
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95999555"
---
# <a name="monitor-and-manage-your-github-repo-by-using-azure-logic-apps"></a>Отслеживайте свой репозиторий GitHub и управляйте им с помощью Azure Logic Apps

GitHub — это веб-служба, где размещен репозиторий Git. Она обеспечивает все возможности распределенного управления версиями и исходным кодом (SCM) в Git, а также другие функции.

Чтобы приступить к работе с соединителем GitHub, [сначала создайте приложение логики](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="create-a-connection-to-github"></a>Создание подключения к GitHub

Чтобы использовать в приложении логики соединитель GitHub, сначала создайте *подключение*, а затем введите сведения для следующих свойств: 

| Свойство | Обязательно | Описание | 
| -------- | -------- | ----------- | 
| Токен | Да | Укажите свои учетные данные GitHub. |

Создав подключение, вы можете выполнять действия и ожидать передачи данных от триггеров, описанных в этой статье.

> [!INCLUDE [Steps to create a connection to GitHub](../../includes/connectors-create-api-github.md)]
> 

## <a name="connector-reference"></a>Справочник по соединителям

Технические сведения о триггерах, действиях и ограничениях, которые описаны в описании OpenAPI (ранее — Swagger) соединителя, см. на [странице справочника по соединителю](/connectors/github/).

## <a name="next-steps"></a>Дальнейшие действия

* См. дополнительные сведения о других [соединителях Logic Apps](../connectors/apis-list.md).