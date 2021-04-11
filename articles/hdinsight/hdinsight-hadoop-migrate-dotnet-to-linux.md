---
title: Использование .NET с Hadoop MapReduce в HDInsight на основе Linux в Azure
description: Узнайте, как использовать приложения .NET для потоковой передачи MapReduce в HDInsight под управлением Linux.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 02/27/2018
ms.openlocfilehash: 9e626d08e7fd315ca70765477c5dae56030521c6
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104866409"
---
# <a name="migrate-net-solutions-for-windows-based-hdinsight-to-linux-based-hdinsight"></a>Перенос решений .NET из HDInsight под управлением Windows в HDInsight под управлением Linux

В кластерах HDInsight под управлением Linux для запуска приложений .NET используется [Mono (https://mono-project.com)](https://mono-project.com). Mono позволяет использовать компоненты .NET, такие как приложения MapReduce, с HDInsight под управлением Linux. В этом документе вы узнаете, как перенести решения .NET, созданные для кластеров HDInsight под управлением Windows, на кластеры HDInsight под управлением Linux для работы с Mono.

## <a name="mono-compatibility-with-net"></a>Совместимость Mono с .NET

Mono версии 4.2.1 входит в состав HDInsight версии 3.6. Дополнительные сведения о версии Mono, которая входит в состав HDInsight, см. в разделе [Что представляют собой различные компоненты Hadoop, доступные в HDInsight?](hdinsight-component-versioning.md)

См. дополнительные сведения о [совместимости Mono и .NET (https://www.mono-project.com/docs/about-mono/compatibility/)](https://www.mono-project.com/docs/about-mono/compatibility/).

> [!IMPORTANT]  
> Платформа SCP.NET совместима с Mono. Дополнительные сведения об использовании SCP.NET с Mono см. в статье [Разработка топологий для Apache Storm в HDInsight на C# с помощью средств Hadoop для Visual Studio](storm/apache-storm-develop-csharp-visual-studio-topology.md).

## <a name="automated-portability-analysis"></a>Автоматический анализ переносимости

С помощью анализатора [.NET Portability Analyzer](https://marketplace.visualstudio.com/items?itemName=ConnieYau.NETPortabilityAnalyzer) можно создать отчет о несовместимости приложения и Mono. Следуйте приведенным ниже инструкциям по настройке анализатора для проверки приложения на совместимость с Mono.

1. Установите [.NET Portability Analyzer](https://marketplace.visualstudio.com/items?itemName=ConnieYau.NETPortabilityAnalyzer). Во время установки выберите используемую версию Visual Studio.

2. В Visual Studio 2015 выберите __Анализ__ > __Portability Analyzer Settings__ (Параметры анализатора переносимости) и убедитесь, что в разделе __Mono__ установлен флажок __4.5__.

    :::image type="content" source="./media/hdinsight-hadoop-migrate-dotnet-to-linux/portability-analyzer-settings.png" alt-text="Установленный флажок &quot;4.5&quot; для Mono в разделе параметров анализатора":::

    Нажмите кнопку __ОК__ , чтобы сохранить настройки.

3. Выберите __Анализ__ > __Analyze Assembly Portability__ (Анализ переносимости сборки). Выберите сборку, содержащую решение, а затем щелкните __Открыть__, чтобы начать анализ.

4. После завершения анализа выберите __Анализ__ > __View analysis reports__ (Просмотр отчетов об анализе). В разделе __Portability Analysis Results__ (Результаты анализа переносимости) щелкните __Открыть отчет__, чтобы открыть отчет.

    :::image type="content" source="./media/hdinsight-hadoop-migrate-dotnet-to-linux/portability-analyzer-results.png" alt-text="Диалоговое окно результатов анализатора переносимости":::

> [!IMPORTANT]  
> Анализатор не может выявить все проблемы с решением. Например, путь к файлу `c:\temp\file.txt` считается нормальным, если Mono работает под управлением Windows. Тот же путь не является допустимым для платформы Linux.

## <a name="manual-portability-analysis"></a>Ручной анализ переносимости

Вручную выполните аудит кода с помощью информации в документе о [переносимости приложений (https://www.mono-project.com/docs/getting-started/application-portability/)](https://www.mono-project.com/docs/getting-started/application-portability/).

## <a name="modify-and-build"></a>Изменение и выполнение сборки

Можно продолжать использовать Visual Studio для создания решений .NET для HDInsight. Однако необходимо обеспечить настройку проекта для использования .NET Framework 4.5.

## <a name="deploy-and-test"></a>Развертывание и тестирование

После изменения решения с помощью рекомендаций из результатов .NET Portability Analyzer или ручного анализа необходимо протестировать его работу с HDInsight. Тестирование решения в кластере HDInsight под управлением Linux может выявить малозаметные проблемы, которые необходимо устранить. Мы рекомендуем включить дополнительное ведение журнала в приложении во время тестирования.

Дополнительные сведения о доступе к журналам см. в следующих документах:

* [Доступ к журналам приложений Apache Hadoop YARN в HDInsight под управлением Linux](hdinsight-hadoop-access-yarn-app-logs-linux.md)

## <a name="next-steps"></a>Дальнейшие действия

* [Использование языка C# для потоковой передачи MapReduce в Hadoop в HDInsight](hadoop/apache-hadoop-dotnet-csharp-mapreduce-streaming.md)

* [Использование определяемых пользователем функций C# при потоковой передаче Apache Hive и Apache Pig в Apache Hadoop в HDInsight](hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

* [Разработка топологий для Apache Storm на C# с помощью средств Data Lake для Visual Studio](storm/apache-storm-develop-csharp-visual-studio-topology.md)
