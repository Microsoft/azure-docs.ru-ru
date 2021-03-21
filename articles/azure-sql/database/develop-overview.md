---
title: Обзор разработки приложений
description: Сведения о доступных библиотеках подключения и рекомендации для приложений, подключающихся к базе данных SQL.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.topic: conceptual
author: stevestein
ms.author: sstein
ms.reviewer: genemi
ms.date: 11/14/2019
ms.custom: sqldbrb=2
ms.openlocfilehash: 7ea3456dbd1d0942cba48c97d70982ae088d73f9
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92782983"
---
# <a name="application-development-overview---sql-database--sql-managed-instance"></a>Общие сведения о разработке приложений — База данных SQL & SQL Управляемый экземпляр

[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

В этой статье рассматриваются основные моменты, которые необходимо учитывать при написании кода для подключения к базе данных в Azure. Эта статья относится к базе данных SQL Azure и Управляемый экземпляр Azure SQL.

## <a name="language-and-platform"></a>Язык и платформа

Для подключения и отправки запросов к базе данных SQL Azure подходят различные [языки и платформы](connect-query-content-reference-guide.md). Вы можете найти [примеры приложений](https://azure.microsoft.com/resources/samples/?service=sql-database&sort=0) , которые можно использовать для подключения к базе данных.

Вы можете использовать инструменты с открытым кодом, такие как [cheetah](https://github.com/wunderlist/cheetah), [sql-cli](https://www.npmjs.com/package/sql-cli) и [VS Code](https://code.visualstudio.com/). Кроме того, база данных SQL Azure поддерживает инструменты Майкрософт, например [Visual Studio](https://www.visualstudio.com/downloads/) и [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms). Кроме того, портал Azure, PowerShell и интерфейсы REST API дополнительно упростят вам работу.

## <a name="authentication"></a>Аутентификация

Доступ к Базе данных SQL Azure защищен с помощью механизмов входа в систему и брандмауэров. База данных SQL Azure поддерживает как SQL Server, так [Azure Active Directory пользователей проверки подлинности](authentication-aad-overview.md) и имена входа. Azure Active Directory имена входа доступны только в Управляемый экземпляр SQL. 

Ознакомьтесь с дополнительными сведения об [управлении именами для входа и доступом к базам данных](logins-create-manage.md).

## <a name="connections"></a>Соединения

В логике подключения к клиенту задайте для времени ожидания по умолчанию 30 секунд. Установленных изначально 15 секунд недостаточно, если подключение зависит от Интернета.

Если вы используете [пул подключений](/dotnet/framework/data/adonet/sql-server-connection-pooling), не забудьте закрыть экземпляр подключения, который ваша программа не использует активно и который не предполагается использовать повторно.

Избегайте длительных транзакций, потому что при любом сбое инфраструктуры или подключения может произойти откат транзакции. Если возможно, разделите транзакцию на несколько меньших транзакций и используйте [пакетную обработку, чтобы повысить производительность](../performance-improve-use-batching.md).

## <a name="resiliency"></a>Устойчивость

База данных SQL Azure — это облачная служба, где в базовой инфраструктуре или при обмене данными между облачными объектами могут происходить временные ошибки. Хотя База данных SQL Azure устойчива к транзитивным сбоям инфраструктуры, они могут повлиять на ваше подключение. Ваш код должен предусматривать возможность [повторного вызова](troubleshoot-common-connectivity-issues.md), если при подключении к Базе данных SQL возникает временная ошибка. Рекомендуется, чтобы логика повторных попыток использовала логику отхода, чтобы не перегружать службу с несколькими клиентами одновременно. Логика повторных попыток зависит от [сообщений об ошибках для клиентских программ Базы данных SQL](troubleshoot-common-errors-issues.md).

Дополнительные сведения о подготовке к запланированным событиям обслуживания в базе данных SQL Azure см. [в статье Планирование событий обслуживания Azure в базе данных SQL Azure](planned-maintenance.md).

## <a name="network-considerations"></a>Рекомендации по сети

- На компьютере с вашей клиентской программой убедитесь, что брандмауэр разрешает исходящие TCP-соединения через порт 1433.  Дополнительные сведения: [Настройка брандмауэра Базы данных SQL Azure](firewall-configure.md).
- Если клиентская программа подключается к базе данных SQL, а клиент работает на виртуальной машине Azure, необходимо открыть на ней определенные диапазоны портов. Дополнительные сведения: [порты свыше 1433 для ADO.NET 4,5 и базы данных SQL](adonet-v12-develop-direct-route-ports.md).
- Клиентские подключения к базе данных SQL Azure иногда обходят прокси-сервер и взаимодействуют непосредственно с базой данных. Порты, отличные от 1433, становятся важными. Дополнительные сведения см. в статьях [Архитектура подключений к базе данных SQL Azure](connectivity-architecture.md) и [Порты для ADO.NET 4.5, отличные от порта 1433](adonet-v12-develop-direct-route-ports.md).
- Сведения о настройке сети для экземпляра SQL Управляемый экземпляр см. в разделе [Сетевая конфигурация для sql управляемый экземпляр](../managed-instance/how-to-content-reference-guide.md#network-configuration).

## <a name="next-steps"></a>Дальнейшие действия

Изучите все возможности [базы данных SQL](sql-database-paas-overview.md) и [управляемый экземпляр SQL](../managed-instance/sql-managed-instance-paas-overview.md).

Чтобы приступить к работе, см. руководства по [базам данных SQL Azure](quickstart-content-reference-guide.md) и [управляемым экземплярам SQL Azure](../managed-instance/quickstart-content-reference-guide.md).