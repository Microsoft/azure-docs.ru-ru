---
title: Обзор Azure Data Lake Analytics
description: Data Lake Analytics позволяет управлять компанией с помощью аналитических сведений, получаемых из любых объемов данных в облаке.
author: saveenr
ms.author: saveenr
ms.reviewer: jasonwhowell
ms.service: data-lake-analytics
ms.topic: overview
ms.date: 06/23/2017
ms.openlocfilehash: f2916b45c04aac3e36e8dfb82a6bb9b332f55286
ms.sourcegitcommit: f6193c2c6ce3b4db379c3f474fdbb40c6585553b
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/08/2021
ms.locfileid: "102449158"
---
# <a name="what-is-azure-data-lake-analytics"></a>Что такое аналитика озера данных Azure

Azure Data Lake Analytics — это служба обработки заданий аналитики по запросу, позволяющая упростить большие данные. Вместо развертывания и настройки оборудования вы пишете запросы для преобразования данных и получения ценных выводов. Служба аналитики способна мгновенно выполнять задания любого масштаба, для чего нужно указать необходимый объем ресурсов. Вы оплачиваете только время выполнения задания, что приводит к значительной экономии средств. 

## <a name="azure-data-lake-analytics-recent-update-information"></a>Сведения о недавнем обновлении Azure Data Lake Analytics

Служба Azure Data Lake Analytics периодически обновляется с определенными целями. Мы продолжаем предоставлять поддержку для этой службы, выпуская обновления компонентов, предварительные бета-версии компонентов и т. д. 

- Актуальные общие сведения см. в статье [Новые возможности Data Lake Analytics](data-lake-analytics-whats-new.md).
- Сведения о каждом обновлении см. в [заметках о выпуске Azure Data Lake Analytics](https://github.com/Azure/AzureDataLake/tree/master/docs/Release_Notes).

## <a name="dynamic-scaling"></a>Динамическое масштабирование
  
Служба Data Lake Analytics динамически подготавливает ресурсы и позволяет анализировать терабайты и даже петабайты данных. При этом вы платите только за используемые вычислительные ресурсы. При увеличении или уменьшении объема хранимых данных или используемых вычислительных ресурсов не нужно переписывать код. 

## <a name="develop-faster-debug-and-optimize-smarter-using-familiar-tools"></a>Быстрая разработка, эффективная отладка и оптимизация с помощью знакомых средств
  
Data Lake Analytics тесно интегрируется с Visual Studio. Вы можете использовать уже знакомые средства для выполнения, отладки и оптимизации кода. Визуализация заданий U-SQL позволяет видеть в масштабе, как выполняется код и, следовательно, легко выявлять узкие места и оптимизировать затраты.

## <a name="u-sql-simple-and-familiar-powerful-and-extensible"></a>U-SQL — простой, знакомый, мощный и расширяемый язык
  
Аналитика озера данных включает U-SQL, язык запросов, который объединяет знакомую, простую декларативную природу SQL с выразительностью C#. В языке U-SQL используется распределенная среда выполнения, лежащая в основе внутреннего озера данных Майкрософт размером в несколько эксабайт. Разработчики SQL и .NET теперь могут обрабатывать и анализировать все свои данные, опираясь на те навыки, которые у них уже есть.

## <a name="integrates-seamlessly-with-your-it-investments"></a>Простая интеграция с имеющимися ИТ-решениями
  
Служба Data Lake Analytics может использовать существующие ИТ-решения в области идентификации, управления и безопасности. Этот подход упрощает работу с данными и позволяет расширить возможности существующих приложений по обработке данных. Data Lake Analytics интегрирована с Active Directory для управления пользователями и разрешениями. Эта служба поставляется со встроенными средствами мониторинга и аудита.

## <a name="affordable-and-cost-effective"></a>Доступное и экономичное решение

Аналитика озера данных — это экономичное решение для выполнения рабочих нагрузок с большими данными. Вы оплачиваете каждое задание отдельно и только тогда, когда обрабатываются данные. Не требуется какого-либо оборудования, лицензий или соглашений о поддержке служб. Система автоматически масштабируется, когда задание запускается и завершается. Это означает, что вы не платите больше, чем используете. [Узнайте больше об управлении затратами и экономии средств](https://aka.ms/adlasavemoney).

## <a name="works-with-all-your-azure-data"></a>Работает со всеми данными Azure
  
Data Lake Analytics работает со службой Azure Data Lake Storage, обеспечивающей максимальную производительность, пропускную способность и параллелизацию, а также с большими двоичными объектами службы хранилища Azure, службой "База данных SQL Azure" и Azure Synapse Analytics.

## <a name="in-region-data-residency"></a>Место расположения данных в регионе
  
Служба Data Lake Analytics не перемещает и не хранит данные клиентов за пределами региона, в котором она развернута.


## <a name="next-steps"></a>Дальнейшие действия

* Сведения о последнем обновлении Azure Data Lake Analytics см. в статье [Новые возможности Azure Data Lake Analytics](data-lake-analytics-whats-new.md).
* Начало работы с Data Lake Analytics с помощью [портала Azure](data-lake-analytics-get-started-portal.md) | [Azure PowerShell](data-lake-analytics-get-started-powershell.md) | [CLI](data-lake-analytics-get-started-cli.md)
* Управление Data Lake Analytics с использованием [портала Azure](data-lake-analytics-manage-use-portal.md) | [Azure PowerShell](data-lake-analytics-manage-use-powershell.md) | [CLI](data-lake-analytics-manage-use-cli.md) | [Azure .NET SDK](data-lake-analytics-manage-use-dotnet-sdk.md) | [Node.js](data-lake-analytics-manage-use-nodejs.md)
* [Управление затратами и экономия средств с помощью Data Lake Analytics](https://1drv.ms/f/s!AvdZLquGMt47h213Hg3rhl-Tym1c)
