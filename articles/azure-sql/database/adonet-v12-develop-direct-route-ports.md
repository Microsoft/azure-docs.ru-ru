---
title: Порты за пределами 1433
description: Взаимодействие с базой данных при клиентских подключениях из ADO.NET к базе данных SQL Azure может происходить напрямую, без прокси-сервера, если база данных не использует порт 1433.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1, devx-track-dotnet
ms.devlang: ''
ms.topic: reference
author: stevestein
ms.author: sstein
ms.reviewer: genemi
ms.date: 06/11/2020
ms.openlocfilehash: 0d009522ea0d0986233983f8725549b618ffb537
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91444872"
---
# <a name="ports-beyond-1433-for-adonet-45"></a>Порты для ADO.NET 4.5, отличные от порта 1433
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

В этом разделе описывается поведение подключения к Базе данных SQL Azure клиентов, использующих ADO.NET 4.5 или более поздней версии.

> [!IMPORTANT]
> Сведения об архитектуре подключения см. в статье [Azure SQL Database Connectivity Architecture](connectivity-architecture.md) (Архитектура подключения Базы данных SQL Azure).
>

## <a name="outside-vs-inside"></a>Снаружи или внутри

В случае подключения к Базе данных SQL Azure сначала нужно узнать, запускается ли клиентская программа *за пределами* или *в пределах* границ облака Azure. В подразделах рассматриваются два типичных сценария.

### <a name="outside-client-runs-on-your-desktop-computer"></a>*Внешняя программа.* Клиент работает на настольном компьютере

Порт 1433 — единственный порт, который должен быть открыт на компьютере, где размещено ваше клиентское приложение базы данных SQL.

### <a name="inside-client-runs-on-azure"></a>*Внутренняя программа.* Клиент работает в Azure

Когда клиент работает внутри границы облака Azure, он использует то, что мы можем вызвать *прямой маршрут* для взаимодействия с базой данных SQL. После установления подключения дальнейшее взаимодействие между клиентом и базой данных не включает в себя шлюз Базы данных SQL Azure.

Последовательность выглядит так:

1. ADO.NET 4.5 (или более поздней версии) инициирует краткое взаимодействие с облаком Azure и получает динамически указанный номер порта.

   * Динамически идентифицируемый номер порта находится в диапазоне 11000-11999.
2. Затем ADO.NET подключается к базе данных SQL напрямую, без промежуточного слоя.
3. Запросы отправляются непосредственно в базу данных, а результаты возвращаются клиенту.

Убедитесь, что диапазоны портов 11000-11999 на клиентском компьютере Azure доступны для взаимодействия клиента ADO.NET 4,5 с базой данных SQL.

* В частности, порты в этом диапазоне должны оставаться свободными от других исходящих ошибок.
* На виртуальной машине Azure параметрами порта управляет **брандмауэр Windows в режиме повышенной безопасности** .
  
  * С помощью [пользовательского интерфейса брандмауэра](/sql/sql-server/install/configure-the-windows-firewall-to-allow-sql-server-access) можно добавить правило, для которого указывается протокол **TCP**, а также диапазон портов, используя следующий синтаксис: **11000–11999**.

## <a name="version-clarifications"></a>Уточнение версии

В этом разделе описываются псевдонимы, относящиеся к версиям продуктов. В нем также перечисляются некоторые пары версий продуктов.

### <a name="adonet"></a>ADO.NET

* ADO.NET 4.0 поддерживает протокол TDS 7.3, но не 7.4.
* ADO.NET 4.5 и более поздних версий поддерживает протокол TDS 7.4.

### <a name="odbc"></a>ODBC

* Microsoft SQL Server ODBC 11 или более поздней версии

### <a name="jdbc"></a>JDBC

* Microsoft SQL Server JDBC 4.2 или более поздней версии (JDBC 4.0 поддерживает TDS 7.4, но не выполняет перенаправление)

## <a name="related-links"></a>Связанные ссылки

* 20 июля 2015 г. был выпущен ADO.NET 4.6. Объявление в блоге группы разработчиков для .NET доступно [здесь](https://devblogs.microsoft.com/dotnet/announcing-net-framework-4-6/).
* 15 августа 2012 г. был выпущен ADO.NET 4.5. Объявление в блоге группы разработчиков для .NET доступно [здесь](https://devblogs.microsoft.com/dotnet/announcing-the-release-of-net-framework-4-5-rtm-product-and-source-code/).
  * Запись блога об ADO.NET 4.5.1 доступна [здесь](https://devblogs.microsoft.com/dotnet/announcing-the-net-framework-4-5-1-preview/).

* Microsoft ODBC Driver 17 для SQL Server https://aka.ms/downloadmsodbcsql

* Подключение к Базе данных SQL Azure версии 12 с помощью перенаправления https://techcommunity.microsoft.com/t5/DataCAT/Connect-to-Azure-SQL-Database-V12-via-Redirection/ba-p/305362

* [Список версий протокола TDS](https://www.freetds.org/)
* [Общие сведения о разработке базы данных SQL](develop-overview.md)
* [Брандмауэр базы данных SQL Azure](firewall-configure.md)
