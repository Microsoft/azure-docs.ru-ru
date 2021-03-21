---
title: Создание & запроса Azure Data Lake Analytics портал Azure
description: Создание учетной записи Azure Data Lake Analytics и отправка задания U-SQL с помощью портала Azure.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: conceptual
ms.date: 03/21/2017
ms.openlocfilehash: 6e51c6848476147073626119689407ebc02c036d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220964"
---
# <a name="get-started-with-azure-data-lake-analytics-using-the-azure-portal"></a>Начало работы с Azure Data Lake Analytics с помощью портала Azure
[!INCLUDE [get-started-selector](../../includes/data-lake-analytics-selector-get-started.md)]

Из этой статьи вы узнаете, как с помощью портала Azure создавать учетные записи Azure Data Lake Analytics, определять задания в [U-SQL](data-lake-analytics-u-sql-get-started.md) и отправлять их в службу Data Lake Analytics.

## <a name="prerequisites"></a>Предварительные условия

Перед началом работы с этим руководством необходимо иметь **подписку Azure**. См. страницу [бесплатной пробной версии Azure](https://azure.microsoft.com/pricing/free-trial/).

## <a name="create-a-data-lake-analytics-account"></a>Создание учетной записи аналитики озера данных

Теперь мы одновременно создадим учетные записи Data Lake Analytics и Azure Data Lake Storage 1-го поколения.  Этот простой шаг занимает около минуты.

1. Выполните вход на [портал Azure](https://portal.azure.com).
2. Щелкните **создать ресурс**  >   **данные и аналитика**  >  **Data Lake Analytics**.
3. Выберите значения для следующих параметров:
   * **Имя**: имя учетной записи Data Lake Analytics (разрешены только строчные буквы и цифры).
   * **Подписка**: выберите подписку Azure, которая используется для учетной записи аналитики.
   * **Группа ресурсов**. Обычно приложения состоят из множества компонентов, например веб-приложения, базы данных, сервера базы данных, хранилища и служб сторонних поставщиков.
   * **Расположение**. выберите центр обработки данных Azure для учетной записи аналитики озера данных.
   * **Azure Data Lake Storage 1-го поколения.** Следуйте инструкциям для создания учетной записи Data Lake Storage 1-го поколения или выберите имеющуюся. 
4. При необходимости выберите ценовую категорию для учетной записи Data Lake Analytics.
5. Нажмите кнопку **Создать**. 


## <a name="your-first-u-sql-script"></a>Первый скрипт U-SQL

Ниже приводится очень простой скрипт U-SQL. Он определяет небольшой набор данных (в рамках скрипта) и записывает его в Azure Data Lake Storage 1-го поколения по умолчанию как файл с именем `/data.csv`.

```usql
@a  = 
    SELECT * FROM 
        (VALUES
            ("Contoso", 1500.0),
            ("Woodgrove", 2700.0)
        ) AS 
              D( customer, amount );
OUTPUT @a
    TO "/data.csv"
    USING Outputters.Csv();
```

## <a name="submit-a-u-sql-job"></a>Отправка задания U-SQL

1. В учетной записи Data Lake Analytics выберите **Новое задание**.
2. Вставьте в текст скрипт U-SQL, приведенный выше. Присвойте заданию имя. 
3. Нажмите кнопку **Отправить**, чтобы запустить задание.   
4. Отслеживайте **состояние** задания и подождите, пока оно изменится на **Успешно**.
5. Перейдите на вкладку **данные** и выберите вкладку **выходные** данные. Выберите выходной файл с именем `data.csv` и просмотрите выходные данные.

## <a name="see-also"></a>См. также раздел

* Чтобы приступить к разработке приложений U-SQL, ознакомьтесь со статьей [Разработка скриптов U-SQL с помощью средств озера данных для Visual Studio](data-lake-analytics-data-lake-tools-get-started.md).
* Для знакомства с U-SQL см. статью о [начале работы с языком U-SQL для Azure Data Lake Analytics](data-lake-analytics-u-sql-get-started.md).
* Задачи управления описываются в руководстве по [управлению Azure Data Lake Analytics с помощью портала Azure](data-lake-analytics-manage-use-portal.md).
