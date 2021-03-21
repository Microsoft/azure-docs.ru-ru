---
title: Интеграция Log Analytics с Power BI и Excel
description: Как отправить результаты из Log Analytics в Power BI
ms.topic: conceptual
author: roygalMS
ms.author: roygal
ms.date: 11/03/2020
ms.openlocfilehash: 894b87bfa17a59db20b424199a4021ad75b182fc
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102041065"
---
# <a name="log-analytics-integration-with-power-bi"></a>Интеграция Log Analytics с Power BI

В этой статье рассматриваются способы передачи данных из Log Analytics в Microsoft Power BI для создания более визуально привлекательных отчетов и панелей мониторинга. 

## <a name="background"></a>Историческая справка 

Журналы Azure Monitor — это платформа, предоставляющая комплексное решение для приема журналов. [Azure Monitor log Analytics](../platform/data-platform.md#) — это интерфейс для запроса этих журналов. Дополнительные сведения о всей платформе Azure Monitor данных, включая Log Analytics, см. в разделе [Azure Monitor Data Platform](../data-platform.md). 

Microsoft Power BI — это платформа визуализации данных Майкрософт. Дополнительные сведения о том, как приступить к работе, см. в разделе [Домашняя страница Power BI](https://powerbi.microsoft.com/). 


В целом можно использовать бесплатные функции Power BI для интеграции и создания визуально привлекательных отчетов и панелей мониторинга.

Для более сложных функций может потребоваться приобрести учетную запись Power BI Pro или Premium. Эти функции включают в себя следующие. 
 - Совместное использование работы 
 - запланированные обновления
 - Приложения Power BI 
 - потоки данных и добавочное обновление 

Дополнительные сведения см. в [статье Подробнее о Power BI ценах и функциях](https://powerbi.microsoft.com/pricing/) . 

## <a name="integrating-queries"></a>Интеграция запросов  

Power BI использует [язык запросов M](/powerquery-m/power-query-m-language-specification/) в качестве основного языка запросов. 

Log Analytics запросы можно экспортировать в M и использовать непосредственно в Power BI. После успешного выполнения запроса выберите **Экспорт в Power BI (M Query)** на кнопке **Экспорт** в верхней панели действий log Analytics пользовательского интерфейса.


:::image type="content" source="media/log-powerbi/export-query2.png" alt-text="Нераскрывающееся меню параметров экспорта Log Analytics запроса" border="true":::

Log Analytics создает txt-файл, содержащий код M, который можно использовать непосредственно в Power BI.

## <a name="connecting-your-logs-to-a-dataset"></a>Подключение журналов к набору данных 

Набор данных Power BI — это источник, готовый для создания отчетов и визуализации. Чтобы подключить Log Analyticsный запрос к набору данных, скопируйте код M, экспортированный из Log Analytics, в пустой запрос в Power BI. 

Дополнительные сведения см. в разделе [Основные сведения о Power BI наборах данных](/power-bi/service-datasets-understand/). 

## <a name="collect-data-with-power-bi-dataflows"></a>Собирайте данные с Power BI потоками данных 

Power BI потоки данных также позволяют получать и хранить данные. Дополнительные сведения см. в разделе [Power BI DataFlow](/power-bi/service-dataflows-overview).

Поток данных — это тип "облачного ETL", предназначенный для помощи в собрании и подготовке данных. Набор данных — это «модель», предназначенная для подключения различных сущностей и моделирования их потребностей.

## <a name="incremental-refresh"></a>Добавочное обновление 

Как Power BI наборы данных, так и Power BIные потоки имеют параметр добавочного обновления. Power BIные потоки данных и Power BIные наборы поддерживают эту функцию, но для ее использования требуется Power BI Premium.  


Добавочное обновление выполняет небольшие запросы и обновляет меньшие объемы данных на один запуск вместо того, чтобы повторно принимать все данные и при выполнении запроса. Вы можете сохранять большие объемы данных, но добавлять новые данные при каждом выполнении запроса. Такое поведение идеально подходит для более длительного выполнения отчетов.

Power BI добавочного обновления полагается на существование *даты и времени* в результирующем наборе. Перед настройкой добавочного обновления убедитесь, что в наборе результатов запроса Log Analytics содержится по крайней мере одно значение *даты и времени* . 

Дополнительные сведения и о настройке добавочного обновления см. в разделе [Power BI наборы данных и добавочное обновление](/power-bi/service-premium-incremental-refresh) , [Power BI потоках и последовательных обновлениях](/power-bi/service-dataflows-incremental-refresh).

## <a name="reports-and-dashboards"></a>Отчеты и панели мониторинга

После отправки данных в Power BI можно по-прежнему использовать Power BI для создания отчетов и панелей мониторинга.

Дополнительные сведения см. [в этом руководство по созданию первой модели Power BI и отчета](/learn/modules/build-your-first-power-bi-report/).  

## <a name="excel-integration"></a>интеграция с Excel

Вы можете использовать ту же интеграцию M, которая используется в Power BI для интеграции с электронной таблицей Excel. Дополнительные сведения см. в этом [руководство по интеграции с Excel](https://support.microsoft.com/office/import-data-from-external-data-sources-power-query-be4330b3-5356-486c-a168-b68e9e616f5a) и вставке запроса M, экспортированного из log Analytics.

Дополнительные сведения можно найти в статьях [интеграция log Analytics и Excel](log-excel.md) .

## <a name="next-steps"></a>Дальнейшие действия

Приступая к работе с [log Analytics запросами](./log-query-overview.md).