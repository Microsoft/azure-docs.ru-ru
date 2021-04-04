---
title: Локальная отладка кода Azure Data Lake Analytics
description: Узнайте, как выполнять отладку заданий U-SQL на локальной рабочей станции с помощью Средств Azure Data Lake для Visual Studio.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 07/03/2018
ms.openlocfilehash: 1c7218deac9efba6df6c1284f2578a744e768284
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92221032"
---
# <a name="debug-azure-data-lake-analytics-code-locally"></a>Локальная отладка кода Azure Data Lake Analytics

Вы можете использовать Средства Azure Data Lake для Visual Studio для выполнения и отладки кода Azure Data Lake Analytics не только в службе Azure Data Lake Analytics, но и на локальной рабочей станции.

Узнайте, [как выполнить скрипт U-SQL на локальном компьютере](data-lake-analytics-data-lake-tools-local-run.md).

## <a name="debug-scripts-and-c-assemblies-locally"></a>Локальная отладка скриптов и сборок C#

Сборки C# можно отлаживать, не отправляя и не регистрируя их в службе Azure Data Lake Analytics. Точки останова можно устанавливать как в файле кода программной части, так и в указанном в ссылке проекте C#.

### <a name="debug-local-code-in-a-code-behind-file"></a>Отладка локального кода в файле кода программной части

1. Установите точки останова в файле кода программной части.
2. Нажмите клавишу **F5** для запуска локальной отладки скрипта.

> [!NOTE]
   > Описанные далее действия работают только в Visual Studio 2015. В более ранних версиях Visual Studio **PDB-файлы**, возможно, потребуется добавить вручную.  
   >
   >

### <a name="debug-local-code-in-a-referenced-c-project"></a>Отладка локального кода в указанном в ссылке проекте C#

1. Создайте проект сборки C# и скомпилируйте его, чтобы получить выходной **DLL-файл**.
2. Зарегистрируйте **DLL-файл** с помощью инструкции U-SQL:

   ```sql
   CREATE ASSEMBLY assemblyname FROM @"..\..\path\to\output\.dll";
   ```
   
3. Установите точки останова в коде C#.
4. Нажмите клавишу **F5**, чтобы запустить отладку скрипта, ссылаясь на локальный **DLL-файл** C#.


## <a name="next-steps"></a>Дальнейшие действия

- Пример более сложного запроса см. в статье [Анализ журналов веб-сайта с помощью Azure Data Lake Analytics](data-lake-analytics-analyze-weblogs.md).
- Для просмотра сведений о заданиях см. статью [Использование браузера и представления для заданий Azure Data Lake Analytics](data-lake-analytics-data-lake-tools-view-jobs.md).
- Дополнительные сведения см. в статье [Использование представления выполнения вершин в инструментах Data Lake для Visual Studio](data-lake-analytics-data-lake-tools-use-vertex-execution-view.md).
