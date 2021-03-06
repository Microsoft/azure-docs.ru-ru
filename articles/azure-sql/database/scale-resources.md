---
title: Масштабирование ресурсов
description: В этой статье объясняется, как масштабировать базу данных в базе данных SQL Azure и Управляемый экземпляр SQL путем добавления или удаления выделенных ресурсов.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: wiassaf, sstein
ms.date: 06/25/2019
ms.openlocfilehash: ca1a2edec70b13f111ffd89278aa39d1ddea7f67
ms.sourcegitcommit: bb330af42e70e8419996d3cba4acff49d398b399
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105035648"
---
# <a name="dynamically-scale-database-resources-with-minimal-downtime"></a>Динамическое масштабирование ресурсов базы данных с минимальным временем простоя
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

База данных SQL Azure и SQL Управляемый экземпляр позволяют динамически добавлять ресурсы в базу данных с минимальным [временем простоя](https://azure.microsoft.com/support/legal/sla/sql-database); Тем не менее в течение короткого промежутка времени, которое можно уменьшить с помощью логики повторных попыток, существует переключение между периодами, в которых подключение к базе данных теряется.

## <a name="overview"></a>Обзор

Когда потребность в вашем приложении растет с нескольких устройств и клиентов до миллионов, база данных SQL Azure и SQL Управляемый экземпляр масштабироваться на лету с минимальным временем простоя. Масштабируемость является одной из важнейших характеристик платформы как услуги (PaaS), которая позволяет динамически добавлять в службу дополнительные ресурсы при необходимости. База данных SQL Azure позволяет легко изменять выделенные для баз данных ресурсы (загрузку ЦП, объем памяти, пропускную способность операций ввода-вывода и объем хранения).

При этом вы можете избежать проблем с производительностью, которые возникают при более интенсивном использовании приложения и которые нельзя устранить с помощью индексации или методов переопределения запросов. Благодаря возможности добавления ресурсов вы можете оперативно реагировать на ситуации, когда база данных достигает текущих ограничений ресурсов и ей требуются дополнительные ресурсы для обработки входящих рабочих нагрузок. База данных SQL Azure также позволяет уменьшать объем ресурсов, когда они не нужны, и, таким образом, снижать расходы.

Вам не нужно беспокоиться о приобретении оборудования и изменении базовой инфраструктуры. Масштабирование базы данных можно легко выполнить с помощью портал Azure ползунком.

![Масштабирование производительности базы данных](./media/scale-resources/scale-performance.svg)

База данных SQL Azure предлагает [модель приобретения на основе DTU](service-tiers-dtu.md) и [модель приобретения на основе виртуальное ядро](service-tiers-vcore.md), а Azure SQL управляемый экземпляр предлагает только [модель приобретения на основе виртуальное ядро](service-tiers-vcore.md). 

- [Модель приобретения на основе DTU](service-tiers-dtu.md) предлагает сочетание ресурсов вычислений, памяти и ввода-вывода на трех уровнях служб для поддержки упрощенных рабочих нагрузок баз данных: Basic, Standard и Premium. Для каждого уровня производительности на уровнях обслуживания предусмотрено отдельное сочетание этих ресурсов, к которым можно добавить ресурсы хранилища.
- [Модель приобретения на основе виртуальное ядро](service-tiers-vcore.md) позволяет выбрать количество виртуальных ядер, объем или объем памяти, а также объем и скорость хранения. Эта модель приобретения предлагает три уровня служб: общего назначения, критически важный для бизнеса и масштабирование.

Вы можете создать первое приложение на основе небольшой отдельной базы данных на уровне служб "Базовый", "Стандартный" или "Общего назначения" по оптимальной ежемесячной цене, а затем в любое время изменить уровень служб на "Премиум" или "Критически важный для бизнеса" вручную или программным способом в соответствии с требованиями вашего решения. Вы можете настроить производительность без простоя для приложения и работы клиентов. Динамическая масштабируемость позволяет базе данных прозрачно реагировать на быстро меняющиеся требования к ресурсам. Кроме того, таким образом вы можете платить только за необходимые ресурсы, и только когда они вам нужны.

> [!NOTE]
> Динамическое масштабирование отличается от автомасштабирования. Автомасштабирование происходит, когда служба автоматически масштабируется на основе критериев, а динамическая масштабируемость позволяет вручную масштабироваться с минимальным временем простоя.

Отдельные базы данных в Базе данных SQL Azure поддерживают динамическое масштабирование вручную, но не автомасштабирование. Для более *автоматического* взаимодействия рекомендуется использовать эластичные пулы, которые позволяют базам данных совместно использовать ресурсы в пуле в зависимости от потребностей конкретной базы данных.
Однако существуют сценарии, которые могут помочь автоматизировать масштабируемость отдельной базы данных в базе данных SQL Azure. С ними можно ознакомится в статье [Мониторинг и масштабирование отдельной базы данных SQL с помощью PowerShell](scripts/monitor-and-scale-database-powershell.md).

Можно изменять [уровни служб DTU](service-tiers-dtu.md) или [характеристики виртуального ядра](resource-limits-vcore-single-databases.md) в любое время с минимальным простоем приложения (обычно в среднем не более четырех секунд). Для многих организаций и приложений достаточно иметь возможность создавать и уменьшать или увеличивать производительность баз данных по запросу, особенно если закономерности использования базы данных относительно хорошо прогнозируются. Но если закономерности использования непредсказуемы, это может усложнить управление расходами и бизнес-моделью. В этом сценарии вы используете эластичный пул с определенным числом единиц eDTU, распределенных между несколькими базами данных в пуле.

![Общие сведения о базе данных SQL: единицы DTU отдельных баз данных по категориям и уровням](./media/scale-resources/single_db_dtus.png)

База данных SQL Azure предлагает возможность динамического масштабирования баз данных.

- Для [отдельной базы данных](single-database-scale.md) можно воспользоваться моделями на основе [DTU](resource-limits-dtu-single-databases.md) или [виртуальных ядер](resource-limits-vcore-single-databases.md), чтобы определить максимальное количество ресурсов, которые будут назначены каждой базе данных.
- [Эластичные пулы](elastic-pool-scale.md) позволяют определять максимальный предел использования ресурсов для каждой группы баз данных в пуле.

Управляемый экземпляр Azure SQL позволяет масштабировать также: 

- [SQL управляемый экземпляр](../managed-instance/sql-managed-instance-paas-overview.md) использует режим [виртуальных ядер](../managed-instance/sql-managed-instance-paas-overview.md#vcore-based-purchasing-model) и позволяет определить максимальное количество ядер ЦП и максимальный объем хранилища, выделенный для вашего экземпляра. Все базы данных в управляемом экземпляре будут совместно использовать ресурсы, выделенные экземпляру.

Запуск действия увеличения или уменьшения масштаба в любой из разновидностей приведет к перезапуску процесса ядра СУБД и перемещению его на другую виртуальную машину, если это необходимо. Процесс перемещения процесса ядра СУБД на новую виртуальную машину — это **оперативный процесс** , в котором можно продолжать использовать имеющуюся службу базы данных SQL Azure во время выполнения процесса. После того как целевое ядро СУБД полностью инициализировано и готово к обработке запросов, подключения будут [переключаться с источника на целевое ядро СУБД](single-database-scale.md#impact).

> [!NOTE]
> Не рекомендуется масштабировать управляемый экземпляр, если выполняется длительная транзакция, например импорт данных, задания обработки данных, перестроение индекса и т. д., или если имеется активное соединение с экземпляром. Чтобы не допустить увеличения времени выполнения масштабирования, чем обычно, необходимо масштабировать экземпляр по завершении всех длительных операций.

> [!NOTE]
> При завершении процесса увеличения или уменьшения масштаба можно избежать короткого разрыва соединения. Если вы реализовали [логику повторных попыток для стандартных временных ошибок](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors), вы не увидите отработку отказа.

## <a name="alternative-scale-methods"></a>Альтернативные методы масштабирования

Масштабирование ресурсов — самый простой и эффективный способ повысить производительность базы данных, не изменяя код базы данных или приложения. В некоторых случаях даже самые высокие уровни служб, размеры вычислений и оптимизации производительности могут не обеспечивать успешное и экономичное выполнение рабочей нагрузки. В этом случае вы можете масштабировать базу данных с помощью следующих дополнительных параметров:

- Функция [горизонтального масштабирования для чтения](read-scale-out.md) — это доступная для чтения реплика данных, которая позволяет выполнять ресурсоемкие запросы только для чтения, например отчеты. Реплика только для чтения будет работать с рабочей нагрузкой только для чтения, не влияя на использование ресурсов в базе данных-источнике.
- [Сегментирование базы данных](elastic-scale-introduction.md) — это набор методов, который позволяет разделять данные по нескольким базам данных и масштабировать их независимо друг от друга.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о повышении производительности базы данных путем изменения кода базы данных см. в статье [Поиск и применение рекомендаций по производительности](database-advisor-find-recommendations-portal.md).
- Сведения об оптимизации базы данных с помощью встроенной системы аналитики см. в статье [Автоматическая настройка базы данных SQL Azure](automatic-tuning-overview.md).
- Сведения о горизонтальном масштабировании в базе данных SQL Azure см. в статье [Использование реплик только для чтения для балансировки нагрузки рабочих нагрузок запросов только для чтения](read-scale-out.md).
- Сведения о сегментировании базы данных см. в статье [Развертывание с помощью Базы данных SQL Azure](elastic-scale-introduction.md).
