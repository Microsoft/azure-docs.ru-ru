---
title: Перемещение данных в хранилище BLOB-объектов Azure и из него — процесс обработки и анализа данных группы
description: Перемещение данных в хранилище BLOB-объектов Azure и из него с помощью Обозреватель службы хранилища Azure, AzCopy, Python и SSIS.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 5148084fa22266b1352046c7d8737b9804c5f4d0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "93311856"
---
# <a name="move-data-to-and-from-azure-blob-storage"></a>Перемещение данных в хранилище BLOB-объектов Azure и из него

Процесс обработки и анализа данных группы требует приема или загрузки данных в различные среды хранения для обработки или анализа наиболее подходящим способом на каждом этапе процесса.

## <a name="different-technologies-for-moving-data"></a>Технологии перемещения данных

В следующих статьях описано, как перемещать данные из хранилища BLOB-объектов Azure и в него с помощью разных технологий.

* [Обозреватель хранилищ Azure](move-data-to-azure-blob-using-azure-storage-explorer.md)
* [AzCopy](../../storage/common/storage-use-azcopy-v10.md)
* [Python](../../storage/blobs/storage-quickstart-blobs-python.md)
* [MSSQL Integration Services](move-data-to-azure-blob-using-ssis.md)

Выбор метода зависит от сценария. Статья [Сценарии для расширенной аналитики в Машинном обучении Azure](plan-sample-scenarios.md) поможет определить ресурсы, необходимые для различных рабочих процессов обработки и анализа данных в рамках расширенного аналитического процесса.

> [!NOTE]
> Полное описание базовых принципов использования хранилища BLOB-объектов Azure см. в статьях [Приступая к работе с хранилищем BLOB-объектов Azure с помощью .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md) и [Основные понятия службы BLOB-объектов](/rest/api/storageservices/Blob-Service-Concepts).
> 
> 

## <a name="using-azure-data-factory"></a>Использование Фабрики данных Azure

В качестве альтернативы можно использовать [фабрику данных Azure](https://azure.microsoft.com/services/data-factory/) для выполнения следующих действий: 

* создание и планирование конвейера, который скачивает данные из хранилища BLOB-объектов Azure; 
* передача данных в опубликованную веб-службу Машинного обучения Azure; 
* получение результатов прогнозной аналитики; 
* отправка результатов в хранилище. 

Дополнительные сведения см. в статье [Создание прогнозных конвейеров с помощью действий Машинного обучения Azure](../../data-factory/transform-data-using-machine-learning.md).

## <a name="prerequisites"></a>Предварительные условия
Для выполнения инструкций из этой статьи у вас должна быть подписка Azure, учетная запись хранения и соответствующий ключ к хранилищу данных для этой учетной записи. Перед отправкой и загрузкой данных необходимо указать имя учетной записи хранения Azure и ключ учетной записи.

* Сведения о настройке подписки Azure см. в [статье Бесплатная пробная версия на один месяц](https://azure.microsoft.com/pricing/free-trial/).
* Инструкции по созданию учетной записи хранения и сведениям об учетных записях и ключах [см. в](../../storage/common/storage-account-create.md)этой статье.