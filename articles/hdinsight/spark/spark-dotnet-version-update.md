---
title: Обновление .NET для Apache Spark до версии v 1.0 в HDI
description: Узнайте об обновлении .NET для Apache Spark версии до 1,0 в HDI и о том, как это влияет на существующий код и кластеры.
author: Niharikadutta
ms.author: nidutta
ms.service: hdinsight
ms.topic: how-to
ms.date: 01/05/2021
ms.openlocfilehash: a1602f29a6d0066ec3c99e990532411621652c47
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98788135"
---
# <a name="updating-net-for-apache-spark-to-version-v10--in-hdinsight"></a>Обновление .NET для Apache Spark до версии v 1.0 в HDInsight

В этом документе рассказывается о первой основной версии [.NET для Apache Spark](https://github.com/dotnet/spark)и о том, как она может повлиять на текущие рабочие конвейеры в кластерах HDInsight.

## <a name="about-net-for-apache-spark-version-100"></a>Сведения о .NET для Apache Spark версии 1.0.0

Это первый [старший официальный выпуск](https://github.com/dotnet/spark/releases/tag/v1.0.0) .net для Apache Spark и обеспечивает полноту работы API-интерфейса данных для Spark 2.4. x, а также для Spark 3.0. x вместе с другими функциями. Полный список всех функций, улучшений и исправлений ошибок см. в официальном [заметках о выпуске версии 1.0.0](https://github.com/dotnet/spark/blob/master/docs/release-notes/1.0.0/release-1.0.0.md).
Следует отметить еще одно важное замечание **: Эта версия несовместима** с предыдущими версиями `Microsoft.Spark` и `Microsoft.Spark.Worker` . Ознакомьтесь с [руководством по миграции](https://github.com/dotnet/spark/blob/master/docs/migration-guide.md#upgrading-from-microsoftspark-0x-to-10) , если вы планируете обновить .net для Apache Sparkного приложения, чтобы оно было совместимо с v 1.0.0.

## <a name="using-net-for-apache-spark-v10-in-hdinsight"></a>Использование .NET для Apache Spark v 1.0 в HDInsight

Пока не будут затронуты текущие кластеры HDI (т. е. они будут иметь ту же версию, что и раньше), только что созданные кластеры HDI будут содержать последнюю версию платформы .NET 1.0.0 для Apache Spark. Это означает, что:

- **У вас есть более старый кластер HDi**: Если вы хотите обновить приложение Spark .NET до версии 1.0.0 (рекомендуется), вам потребуется обновить `Microsoft.Spark.Worker` версию в кластере HDi. Дополнительные сведения см. в [разделе изменение версий .NET для Apache Spark в кластере HDi](#changing-net-for-apache-spark-version-on-hdinsight).
Если вы не хотите обновлять текущую версию .NET для Apache Spark в приложении, дальнейшие действия не требуются.  

- **У вас есть новый кластер HDi**. Если вы хотите обновить приложение Spark .NET до версии 1.0.0 (рекомендуется), не требуется никаких действий для изменения рабочей роли в HDi, однако вам придется обратиться к [руководству по миграции](https://github.com/dotnet/spark/blob/master/docs/migration-guide.md#upgrading-from-microsoftspark-0x-to-10) , чтобы понять, какие действия необходимо выполнить для обновления кода и конвейеров.
Если вы не хотите изменять текущую версию .NET для Apache Spark в приложении, вам потребуется изменить версию в кластере HDI с v 1.0 (по умолчанию в новых кластерах) на любую используемую версию. Дополнительные сведения см. в [разделе изменение версий .NET для Apache Spark в кластере HDi](spark-dotnet-version-update.md#changing-net-for-apache-spark-version-on-hdinsight).  

## <a name="changing-net-for-apache-spark-version-on-hdinsight"></a>Изменение версии .NET для Apache Spark в HDInsight

### <a name="deploy-microsoftsparkworker"></a>Развертывание Microsoft.Spark.Worker

`Microsoft.Spark.Worker` — это серверный компонент, который находится на отдельных рабочих узлах кластера Spark. Если вы хотите выполнить определяемую пользователем функцию (UDF) C#, Spark необходимо понять, как запустить среду CLR .NET для выполнения этой функции. `Microsoft.Spark.Worker` предоставляет коллекцию классов для Spark, которые обеспечивают эту функциональность. Выберите версию рабочей роли в зависимости от версии .NET для Apache Spark, которую требуется развернуть в кластере HDI.

1. Скачайте выпуск Microsoft. Spark. Worker Linux для конкретной версии. Например, если нужно `.NET for Apache Spark v1.0.0` , скачайте [Microsoft. Spark. Worker. netcoreapp 3.1. Linux-x64-1.0.0. tar. gz](https://github.com/dotnet/spark/releases/tag/v1.0.0).  

2. Скачайте скрипт [Install-Worker.sh](https://github.com/dotnet/spark/blob/master/deployment/install-worker.sh) для установки исполняемых файлов рабочих процессов, скачанных на шаге 1, на все рабочие узлы кластера HDi.  

3. Отправьте упомянутые выше файлы в учетную запись хранения Azure, к которой имеет доступ ваш кластер. Дополнительные сведения см. в [статье о развертывании .NET для Apache Spark HDi](/dotnet/spark/tutorials/hdinsight-deployment#upload-files-to-azure) .

4. Запустите `install-worker.sh` сценарий на всех рабочих узлах кластера с помощью действий сценария. Дополнительные сведения см. в [статье о развертывании .NET для Apache Spark HDi](/dotnet/spark/tutorials/hdinsight-deployment#run-the-hdinsight-script-action) .

### <a name="update-your-application-to-use-specific-version"></a>Обновление приложения для использования определенной версии

Вы можете обновить .NET для Apache Spark приложения, чтобы использовать определенную версию, выбрав требуемую версию [пакета NuGet Microsoft. Spark](https://www.nuget.org/packages/Microsoft.Spark/) в проекте. Обязательно ознакомьтесь с заметками о выпуске конкретной версии и [руководством по миграции](https://github.com/dotnet/spark/blob/master/docs/migration-guide.md#upgrading-from-microsoftspark-0x-to-10) , как упоминалось выше, при выборе обновления приложения до версии 1.0.0.

## <a name="faqs"></a>Часто задаваемые вопросы

### <a name="will-my-existing-hdi-cluster-with-version--100-start-failing-with-the-new-release"></a>Будет ли мой существующий кластер HDI с версией < 1.0.0 начать работать с новым выпуском?

Существующие кластеры HDI будут иметь ту же самую предыдущую версию для .NET для Apache Spark и имеющегося приложения (с предыдущей версией Spark .NET) не будут затронуты.

## <a name="next-steps"></a>Дальнейшие действия

[Развертывание .NET для Apache Spark приложения в HDInsight](/dotnet/spark/tutorials/hdinsight-deployment)