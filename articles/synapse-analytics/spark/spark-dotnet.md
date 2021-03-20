---
title: Использование .NET для Apache Spark
description: Сведения об использовании .NET и Apache Spark для пакетной обработки, потоковой передачи в реальном времени, машинного обучения и создания ad-hoc-запросов в записных книжках Azure Synapse Analytics.
author: luisquintanilla
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 05/01/2020
ms.author: luquinta
ms.reviewer: jrasnick
ms.openlocfilehash: 895d766cc1e70e6a001904770d6f3ef12b6945cd
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98018667"
---
# <a name="use-net-for-apache-spark-with-azure-synapse-analytics"></a>Использование .NET для Apache Spark с помощью Azure Synapse Analytics

[.NET для Apache Spark](https://dot.net/spark) предоставляет бесплатную поддержку .NET для платформы Spark с [открытым кодом](https://github.com/dotnet/spark)и кросс-платформенный. 

Он предоставляет привязки .NET для Spark, которые позволяют получать доступ к API Spark через C# и F #. С помощью .NET для Apache Spark можно также написать и выполнить определяемые пользователем функции для Spark, написанного на .NET. Интерфейсы API .NET для Spark предоставляют доступ ко всем аспектам Spark DataFrames для анализа данных, включая Spark SQL, Delta Lake и структурированную потоковую передачу.

Вы можете анализировать данные в .NET для Apache Spark с использованием определений пакетных заданий Spark или интерактивных записных книжек Azure Synapse Analytics. Из этой статьи вы узнаете, как работать с .NET для Apache Spark в Azure Synapse Analytics с помощью обоих методов.

## <a name="submit-batch-jobs-using-the-spark-job-definition"></a>Отправка пакетных заданий с помощью определения задания Spark

Ознакомьтесь с учебником, который описывает применение Azure Synapse Analytics для [создания определений заданий Apache Spark для пулов Synapse Spark](apache-spark-job-definitions.md). Если вы не упакованы в приложение для отправки в Azure синапсе, выполните следующие действия.

1. Выполните следующие команды, чтобы опубликовать приложение. Не забудьте заменить *mySparkApp* реальным значением пути к приложению.

   **В Windows:**

   ```dotnetcli
   cd mySparkApp
   dotnet publish -c Release -f netcoreapp3.1 -r win-x64
   ```
   
   **В Linux:**

   ```dotnetcli
   cd mySparkApp
   dotnet publish -c Release -f netcoreapp3.1 -r ubuntu.16.04-x64
   ```

2. Заархивировать содержимое папки публикации, например, `publish.zip` созданной в результате выполнения шага 1. Все сборки должны находиться на первом уровне ZIP-файла. Не должны использоваться промежуточные уровни папки. Это означает, что при распаковке `publish.zip` все сборки извлекаются в текущий рабочий каталог.

    **В Windows:**

    Используйте программу извлечения, например [7-Zip](https://www.7-zip.org/) или [WinZip](https://www.winzip.com/), чтобы извлечь файл в каталог bin со всеми опубликованными двоичными файлами.

    **В Linux:**

    Откройте оболочку Bash и компакт-диск в каталоге bin со всеми опубликованными двоичными файлами и выполните следующую команду.

    ```bash
    zip -r publish.zip
    ```

## <a name="net-for-apache-spark-in-azure-synapse-analytics-notebooks"></a>.NET для Apache Spark в записных книжках Azure Synapse Analytics 

Записные книжки предоставляют отличный способ создания прототипов конвейеров и сценариев .NET для Apache Spark. Вы можете быстро и эффективно начать работу с данными, анализировать их, фильтровать, отображать и визуализировать. 

Специалисты по инжинирингу, обработке и анализу данных, бизнес-аналитики и специалисты по машинному обучению могут совместно работать над общим интерактивным документом. Вы немедленно получите результаты исследования данных и сможете визуализировать их прямо в той же записной книжке.

### <a name="how-to-use-net-for-apache-spark-notebooks"></a>Использование .NET для записных книжек Apache Spark

При создании записной книжки вы можете выбрать ядро языка, на котором будете выражать бизнес-логику. Поддержка ядра доступна для нескольких языков, включая C#.

Чтобы использовать .NET для Apache Spark в записной книжке Azure синапсе Analytics, выберите **.NET Spark (C#)** в качестве ядра и подключите записную книжку к существующему бессерверному Apache Spark пулу.

Записная книжка .NET Spark основана на [интерактивном](https://github.com/dotnet/interactive) интерфейсе .NET и предоставляет интерактивные возможности C# с возможностью использования .NET для Spark из Box с `spark` уже предопределенной переменной сеанса Spark.

### <a name="install-nuget-packages-in-notebooks"></a>Установка пакетов NuGet в записные книжки

Вы можете установить выбранные вами пакеты NuGet в записную книжку, используя `#r nuget` команду Magic перед именем пакета NuGet. На следующей схеме показан пример.

![Снимок экрана, показывающий использование #r для установки пакета NuGet записной книжки .NET для Spark](./media/apache-spark-development-using-notebooks/synapse-spark-dotnet-notebook-nuget.png)

Дополнительные сведения о работе с пакетами NuGet в записных книжках см. в [интерактивной документации по .NET](https://github.com/dotnet/interactive/blob/main/docs/nuget-overview.md).

### <a name="net-for-apache-spark-c-kernel-features"></a>.NET для функций ядра C# в Apache Spark

При использовании .NET для Apache Spark в записной книжке Azure Synapse Analytics доступны следующие возможности.

* Декларативный HTML. Создавайте выходные данные из ячеек с использованием синтаксиса HTML, в том числе заголовки, маркированные списки и даже изображения.
* Простые инструкции C# (назначения, печать в консоль, создание исключений и т. д.).
* Многострочные блоки кода C# (операторы условий, циклы foreach, определения классов и т. д.).
* Доступ к стандартной библиотеке C# (System, LINQ, Enumerables и т. д.).
* Поддержка функций языка C# 8.0.
* `spark` как предварительно определенная переменная для предоставления доступа к сеансу Apache Spark.
* Поддержка [определяемых пользователем функций .NET, которые могут выполняться в Apache Spark](/dotnet/spark/how-to-guides/udf-guide). Мы рекомендуем [писать и вызывать определяемые пользователем функции в .NET для Apache Spark интерактивных сред](/dotnet/spark/how-to-guides/dotnet-interactive-udf-issue) , чтобы узнать, как использовать UDF в .net для Apache Spark интерактивного взаимодействия.
* Поддержка визуализации выходных данных из заданий Spark с использованием нескольких типов диаграмм (график, линейчатая диаграмма или гистограмма) и макетов (одиночный, наложенный и т. п.) благодаря поддержке библиотеки `XPlot.Plotly`.
* Возможность включения пакетов NuGet в записную книжку C#.

## <a name="next-steps"></a>Дальнейшие действия

* [Документация по .NET для Apache Spark](/dotnet/spark/)
* [Интерактивные руководства .NET для Apache Spark](/dotnet/spark/how-to-guides/dotnet-interactive-udf-issue)
* [Azure Synapse Analytics](https://azure.microsoft.com/services/synapse-analytics/)
* [Интерактивные возможности .NET](https://devblogs.microsoft.com/dotnet/creating-interactive-net-documentation/)
