---
title: Примеры для разработчиков, использующие Каталог данных Azure
description: В этой статье рассматриваются доступные примеры разработчиков, использующие REST API каталога данных.
ms.service: data-catalog
author: JasonWHowell
ms.author: jasonh
ms.topic: conceptual
ms.date: 08/01/2019
ms.openlocfilehash: 15b48bc41e230ca5b9003675e2caab25741bcbfd
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104674773"
---
# <a name="azure-data-catalog-developer-samples"></a>Примеры для разработчиков, использующие Каталог данных Azure

[!INCLUDE [Azure Purview redirect](../../includes/data-catalog-use-purview.md)]

Приступите к разработке приложений каталога данных Azure с помощью REST API каталога данных. REST API каталога данных — это API на основе REST, который предоставляет программный доступ к ресурсам каталога данных для регистрации, аннотирования и поиска ресурсов-контейнеров данных программными средствами.

## <a name="samples-available-on-githubcom"></a>Примеры, доступные в GitHub.com

* [Начало работы с Каталогом данных Azure](https://github.com/Azure-Samples/data-catalog-dotnet-get-started/)
  
   В примере для начала работы показано, как пройти проверку подлинности в Azure AD для регистрации, поиска и удаления ресурса данных с помощью REST API каталога данных.
   
* [Приступая к работе с каталогом данных Azure с помощью субъекта-службы](https://github.com/Azure-Samples/data-catalog-dotnet-service-principal-get-started/)

   В этом примере показано, как зарегистрировать, найти и удалить ресурс данных с помощью REST API каталога данных. В этом примере используется проверка подлинности субъекта-службы.

* [Средство импорта и экспорта для каталога данных Azure](https://github.com/Azure-Samples/data-catalog-dotnet-import-export/)

   В этом примере показано, как использовать каталог данных REST API для выборки ресурсов из каталога данных Azure и их сериализации в файл. Он также показывает, как взять набор ресурсов-контейнеров, сериализованный в формат JSON, и отправить его в каталог. Он поддерживает экспорт подмножества каталога с помощью поискового запроса.

* [Полное регистрация и Добавление аннотаций в каталог данных Azure](https://github.com/Azure-Samples/data-catalog-dotnet-excel-register-data-assets/)
  
   В этом примере показано, как выполнить массовый регистрацию ресурсов данных из книги Excel с помощью REST API и Open XML.
  
* [Групповое импорт терминов глоссария в каталог данных Azure](https://github.com/Azure-Samples/data-catalog-bulk-import-glossary/)

   В этом примере показано, как импортировать термины глоссария из CSV-файлов в глоссарий ADC.

* [Групповое импорт связей в каталог данных Azure](https://github.com/Azure-Samples/data-catalog-bulk-import-relationship/)

   В этом примере показано, как программным способом импортировать сведения о связях из CSV-файла в каталог данных.

* [Публикация отношений в каталоге данных Azure](https://github.com/Azure-Samples/data-catalog-dotnet-publish-relationships/)

   В этом примере показано, как можно программно публиковать сведения о связях в каталоге данных.
   
## <a name="next-steps"></a>Дальнейшие действия
[Справочник по REST API каталога данных Azure](/rest/api/datacatalog/)
