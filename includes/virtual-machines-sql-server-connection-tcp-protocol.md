---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: f7af6962c3343cd9d3e35e96d88650aa6a42c6b3
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95560102"
---
1. При подключении к виртуальной машине с помощью удаленного рабочего стола найдите **диспетчер конфигурации**:

    ![Откройте SSCM](./media/virtual-machines-sql-server-connection-tcp-protocol/sql-server-configuration-manager.png)

1. В диспетчере конфигурации SQL Server в области консоли разверните раздел **Сетевая конфигурация SQL Server**.

1. В области консоли щелкните **протоколы для MSSQLSERVER** (имя экземпляра по умолчанию). В области сведений щелкните правой кнопкой мыши **TCP** и выберите пункт **включить** , если он еще не включен.

    ![Включение TCP](./media/virtual-machines-sql-server-connection-tcp-protocol/enable-tcp.png)

1. В области консоли выберите **Службы SQL Server**. В области сведений щелкните правой кнопкой мыши **SQL Server (*имя экземпляра*)** (экземпляр по умолчанию — **SQL Server (MSSQLSERVER)**) и выберите **Перезагрузить**, чтобы остановить и перезапустить экземпляр SQL Server.

    ![Перезапустите ядро СУБД](./media/virtual-machines-sql-server-connection-tcp-protocol/restart-sql-server.png)

1. Закройте диспетчер конфигурации SQL Server.

Дополнительные сведения о включении протоколов для компонента Ядро СУБД SQL Server см. в статье [Включение или отключение сетевого протокола сервера](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol).