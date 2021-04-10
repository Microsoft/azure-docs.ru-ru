---
title: Установка сторонних приложений в Azure HDInsight
description: Сведения об установке сторонних приложений Apache Hadoop в Azure HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/17/2019
ms.openlocfilehash: 46a22b78ade3d137f25b14e5cbdaff3264208228
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104868398"
---
# <a name="install-third-party-apache-hadoop-applications-on-azure-hdinsight"></a>Установка сторонних приложений Apache Hadoop в Azure HDInsight

Узнайте, как установить стороннее приложение [Apache Hadoop](https://hadoop.apache.org/) в Azure HDInsight. Инструкции по установке собственного приложения см. в статье [Установка пользовательских приложений HDInsight](hdinsight-apps-install-custom-applications.md).

Пользователи могут устанавливать приложения HDInsight в кластере HDInsight. Разработчиками этих приложений могут быть корпорация Майкрософт, независимые поставщики программного обеспечения или вы сами.  

Опубликованные приложения представлены в следующем списке:

|Приложение |Типы кластеров | Описание |
|---|---|---|
|[Платформа AtScale Intelligence](https://aws.amazon.com/marketplace/pp/AtScale-AtScale-Intelligence-Platform/B07BWWHH18) |Hadoop |AtScale превращает ваш кластер HDInsight в OLAP-сервер с возможностью горизонтального увеличения масштаба, позволяя запрашивать миллиарды строк данных в интерактивном режиме с помощью уже знакомых и полюбившихся инструментов бизнес-аналитики, которые уже имеются, — от Microsoft Excel, Power BI, Tableau Software до QlikView. |
|[CDAP для HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/cask.cdap-for-hdinsight) |HBase |CDAP — это первая единая платформа интеграции для больших данных, которая ускоряет окупаемость вложений в Hadoop и позволяет ИТ-отделам предоставлять данные самообслуживания. Расширяемое решение CDAP с открытым исходным кодом устраняет барьеры для инноваций. Требования 4 узла регионов, минимум D3 версии 2. |
|[Datameer](https://azuremarketplace.microsoft.com/marketplace/apps/datameer.datameer) |Hadoop |Datameer — это масштабируемая платформа самообслуживания для подготовки, изучения и управления данными для аналитики, которая ускоряет превращение сложных данных из нескольких источников в ценные сведения, готовые к использованию, быстрее и эффективнее предоставляя аналитические данные на корпоративном уровне. |
|[Dataiku DSS в HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/dataiku.dss-on-hdi) |Hadoop, Spark |Dataiku DSS представляет собой корпоративную платформу для обработки и анализа данных, которая позволяет специалистам по обработке и анализу данных совместно создавать и запускать новые продукты и службы данных, преобразуя необработанные данные в информативные прогнозы. |
|[Приложение WANdisco Fusion HDI](https://community.wandisco.com/s/article/Use-WANdisco-Fusion-for-parallel-operation-of-ADLS-Gen1-and-Gen2) |Hadoop, Spark, HBase, Storm, Kafka |Обеспечение согласованности данных в распределенной среде — задача по выполнению массовых операций с данными. WANdisco Fusion, программная платформа корпоративного класса, решает эту проблему, обеспечивая согласованность неструктурированных данных в любой среде. |
|[H2O SparklingWater для HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/h2o-ai.h2o-sparklingwater) |Spark |H2O Sparkling Water поддерживает следующие распределенные алгоритмы: GLM, Naïve Bayes, распределенный случайный лес, компьютер градиентного усиления, глубокие нейронные сети, глубокое обучение, метод K-средних, помощник по совместимости программ, обобщенные модели низкорангового представления, обнаружение аномалий, автокодировщики. |
|[Striim для интеграции данных с HDInsight в режиме реального времени](https://azuremarketplace.microsoft.com/marketplace/apps/striim.striimbyol) |Hadoop, HBase, Storm, Spark, Kafka |Striim — это комплексная платформа для интеграции и аналитики потоковой передачи данных, обеспечивающая непрерывный прием, обработку и анализ разрозненных потоков данных. |
|[Jumbune Enterprise-Accelerating BigData Analytics](https://azuremarketplace.microsoft.com/marketplace/apps/impetus-infotech-india-pvt-ltd.impetus_jumbune) |Hadoop, Spark |Jumbune помогает предприятиям в следующем. 1. Повышение производительности рабочей нагрузки Hive, Java, Scala на основе ядра Tez, MapReduce и Spark. 2. Упреждающий мониторинг кластера Hadoop. 3. Организация управления качеством данных в распределенной файловой системе. |
|[Kyligence Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/kyligence.kyligence) |Hadoop, HBase, Spark |Kyligence Enterprise на платформе Apache Kylin обеспечивает бизнес-аналитику на основе больших данных. Являясь подсистемой OLAP корпоративного уровня в Hadoop, Kyligence Enterprise позволяет выполнять бизнес-аналитику в Hadoop с использованием методологии бизнес-аналитики и хранилища данных, соответствующих отраслевым стандартам. |
|[Starburst Presto для Azure HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/starburstdatainc1579800938563.starburst-presto?tab=Overview) |Hadoop |Presto — это быстрая и масштабируемая распределенная подсистема запросов SQL. Разработанная для разделения хранилища и вычислений Presto идеально подходит для выполнения запросов к данным в Azure Data Lake Storage, хранилищах BLOB-объектов Azure, базах данных SQL и NoSQL и других источниках данных. |
|[StreamSets Data Collector для облака HDInsight](https://azuremarketplace.microsoft.com/marketplace/apps/streamsets.streamsets-data-collector-hdinsight) |Hadoop, HBase, Spark, Kafka |StreamSets Data Collector — это облегченная и в то же время эффективная подсистема, которая осуществляет потоковую передачу данных в режиме реального времени. Используйте Data Collector для маршрутизации и обработки данных в потоках данных. Предлагается 30-дневная пробная версия продукта. |
|[Trifacta Wrangler Enterprise](https://azuremarketplace.microsoft.com/marketplace/apps/trifacta.trifacta-db?tab=Overview) |Hadoop, Spark, HBase |Trifacta Wrangler Enterprise для HDInsight поддерживает первичную обработку данных уровня предприятия для любого масштаба данных. Стоимость запуска Trifacta в Azure представляет собой сочетание затрат на подписку Trifacta и затрат на инфраструктуру Azure для виртуальных машин. |
|[Unifi Data Platform](https://www.crunchbase.com/organization/unifi-software) |Hadoop, HBase, Storm, Spark |Unifi Data Platform — это набор интегрированных средств для самостоятельной обработки данных, предназначенных для расширения возможностей бизнес-пользователя при решении проблем с данными, приносящий дополнительный доход, снижающий затраты и уменьшающий сложность эксплуатации. |
|[Unraveldata APM](https://azuremarketplace.microsoft.com/marketplace/apps/unravel-data.unravel-app) |Spark |Приложение Unravel Data для кластера HDInsight Spark. |
|[Каталог данных на базе искусственного интеллекта Waterline](https://azuremarketplace.microsoft.com/marketplace/apps/waterline_data.waterline_data) |Spark |Waterline каталогизирует, упорядочивает и управляет данными, используя искусственный интеллект, чтобы автоматически отмечать данные бизнес-терминами. Каталог грамотного бизнеса Waterline —это чрезвычайно важный компонент успешного проведения самостоятельной аналитики, обеспечения соответствия требованиям и системы управления, а также инициатив в области управления ИТ. |

В этой статье используется портал Azure. Вы можете экспортировать шаблон Azure Resource Manager с портала или получить его копию у поставщиков, а затем развернуть его с помощью Azure PowerShell и классического интерфейса командной строки Azure.  Ознакомьтесь со статьей [Создание кластеров Apache Hadoop в HDInsight с помощью шаблонов Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md).

## <a name="prerequisites"></a>Предварительные требования
Если вы хотите установить приложение HDInsight в существующем кластере HDInsight, вы должны создать кластер HDInsight. Инструкции по созданию кластера см. в [этом разделе](hadoop/apache-hadoop-linux-tutorial-get-started.md). Вы также можете установить приложения HDInsight во время создания кластера HDInsight.

## <a name="install-applications-to-existing-clusters"></a>Установка приложений в имеющиеся кластеры
Здесь приведена процедура установки приложения HDInsight в имеющийся кластер HDInsight.

**Установка приложения HDInsight**

1. Войдите на [портал Azure](https://portal.azure.com).
2. В меню слева последовательно выберите **Все службы**  >  **Аналитика**  >  **Кластеры HDInsight**.
3. Выберите из списка кластер HDInsight.  Если у вас нет кластера, сначала его необходимо создать.  См. [этот раздел](hadoop/apache-hadoop-linux-tutorial-get-started.md).
4. В категории **Параметры** выберите **Приложения**. В основном окне отобразится список установленных приложений. 
   
    :::image type="content" source="./media/hdinsight-apps-install-applications/hdinsight-apps-portal-menu.png" alt-text="Меню портала для приложений HDInsight":::
5. В меню выберите **+Добавить**. Отобразится список доступных приложений.  Если параметр **+Добавить** недоступен для выбора, это означает, что приложения для этой версии кластера HDInsight отсутствуют.
   
    :::image type="content" source="./media/hdinsight-apps-install-applications/hdinsight-apps-list1.png" alt-text="Доступные приложения HDInsight":::
6. Выберите одно из доступных приложений и следуйте инструкциям, чтобы принять условия лицензионного соглашения.

Состояние установки приложения можно отслеживать в уведомлениях портала (щелкните значок колокольчика в верхней части портала). После установки приложение отобразится в списке "Установленные приложения".

## <a name="install-applications-during-cluster-creation"></a>Установка приложения во время создания кластера

Вы также можете установить приложения HDInsight во время создания кластера. Их установка осуществляется после создания и запуска кластера. Чтобы установить приложения во время создания кластера с помощью портала Azure, на вкладке **Конфигурация и цены** выберите **+ Добавить приложение**.

:::image type="content" source="./media/hdinsight-apps-install-applications/azure-portal-cluster-configuration-applications.png" alt-text="Приложения для конфигурации кластера на портале Azure":::

## <a name="list-installed-hdinsight-apps-and-properties"></a>Отображение списка установленных приложений HDInsight и их свойств
На портале содержится список установленных приложений HDInsight для кластера и их свойств.

**Список приложений HDInsight и их свойств**

1. Войдите на [портал Azure](https://portal.azure.com).
2. В меню слева последовательно выберите **Все службы**  >  **Аналитика**  >  **Кластеры HDInsight**.
3. Выберите из списка кластер HDInsight.
4. В категории **Параметры** выберите **Приложения**. В основном окне отобразится список установленных приложений. 
   
    :::image type="content" source="./media/hdinsight-apps-install-applications/hdinsight-apps-installed-apps-with-apps.png" alt-text="Установленные приложения HDInsight":::
5. Чтобы открыть свойства установленного приложения, щелкните его. В свойствах указано следующее:

    |Свойство | Описание |
    |---|---|
    |Имя приложения. |Имя приложения. |
    |Состояние |Состояние приложения. |
    |Веб-страница |URL-адрес веб-приложения, развернутого на граничном узле. В качестве учетных данных используются учетные данные пользователя HTTP, настроенные для кластера. |
    |Конечная точка SSH |К граничному узлу можно подключиться с помощью SSH. В качестве учетных данных SSH используются учетные данные пользователя SSH, настроенные для кластера. См. дополнительные сведения об [использовании SSH в HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md). |
    |Описание | Описание приложения. |

6. Чтобы удалить приложение, щелкните его правой кнопкой мыши и в контекстном меню выберите **Удалить**.

## <a name="connect-to-the-edge-node"></a>Подключение к граничному узлу
Для подключения к граничному узлу можно использовать протокол HTTP и SSH. Дополнительные сведения о конечной точке см. на [портале](#list-installed-hdinsight-apps-and-properties). См. дополнительные сведения об [использовании SSH в HDInsight](hdinsight-hadoop-linux-use-ssh-unix.md).

В качестве учетных данных конечной точки HTTP используютсяучетные данные пользователя HTTP, настроенные для кластера HDInsight. Что же касается конечной точки SSH, используются учетные данные пользователя SSH, настроенные для кластера HDInsight.

## <a name="troubleshoot"></a>Диагностика
См. раздел [Устранение неполадок, связанных с установкой](hdinsight-apps-install-custom-applications.md#troubleshoot-the-installation).

## <a name="next-steps"></a>Дальнейшие действия
* [Установка пользовательских приложений HDInsight](hdinsight-apps-install-custom-applications.md)— узнайте, как развернуть в HDInsight неопубликованное приложение HDInsight.
* [Публикация приложения HDInsight в Azure Marketplace](hdinsight-apps-publish-applications.md). Узнайте, как опубликовать пользовательские приложения HDInsight в Microsoft Azure Marketplace.
* [MSDN. Application](/rest/api/hdinsight/hdinsight-application) (Приложение). Узнайте, как определить приложения HDInsight.
* [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](hdinsight-hadoop-customize-cluster-linux.md)— узнайте, как использовать действие скрипта для установки дополнительных приложений.
* [Создание кластеров Apache Hadoop под управлением Linux в HDInsight с помощью шаблонов Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md) — узнайте, как вызывать шаблоны Resource Manager для создания кластеров HDInsight.
* [Использование пустых граничных узлов в HDInsight](hdinsight-apps-use-edge-node.md)— узнайте, как использовать пустой граничный узел для доступа к кластеру HDInsight, а также тестирования и размещения приложений HDInsight.