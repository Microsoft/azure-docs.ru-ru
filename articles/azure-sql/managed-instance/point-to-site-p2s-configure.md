---
title: Настройка подключения "точка — сеть" с помощью SSMS
titleSuffix: Azure SQL Managed Instance
description: Сведения об установке подключения "точка — сеть" к Управляемому экземпляру SQL Azure с помощью SQL Server Management Studio (SSMS) с локального клиентского компьютера.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, jovanpop
ms.date: 03/13/2019
ms.openlocfilehash: 3baf2d7ed7c326895ae40948fc2d0a4cc03021f9
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "92788372"
---
# <a name="quickstart-configure-a-point-to-site-connection-to-azure-sql-managed-instance-from-on-premises"></a>Краткое руководство. Настройка подключения "точка — сеть" к Управляемому экземпляру SQL Azure из локальной среды
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

В этом кратком руководстве показано, как установить подключение "точка — сеть" к Управляемому экземпляру SQL Azure с помощью [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) (SSMS) с локального клиентского компьютера. Дополнительные сведения см. в статье [Сведения о VPN-подключениях "точка — сеть"](../../vpn-gateway/point-to-site-about.md).

## <a name="prerequisites"></a>Предварительные требования

В этом кратком руководстве:

- Как отправная точка используются ресурсы, созданные при работе с руководством по [созданию управляемого экземпляра](instance-create-quickstart.md).
- На локальном клиентском компьютере требуется PowerShell 5.1 и Azure PowerShell 1.4.0 или более поздней версии. При необходимости ознакомьтесь с инструкциями по [установке модуля PowerShell для Azure](/powershell/azure/install-az-ps#install-the-azure-powershell-module).
- Требуется последняя версия [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) на локальном клиентском компьютере.

## <a name="attach-a-vpn-gateway-to-a-virtual-network"></a>Подключение VPN-шлюза к виртуальной сети

1. Откройте PowerShell на локальном клиентском компьютере.

2. Скопируйте этот сценарий PowerShell. Этот скрипт позволяет подключить VPN-шлюз к виртуальной сети Управляемого экземпляра SQL, созданной при работе с кратким руководством [Создание управляемого экземпляра](instance-create-quickstart.md). Этот скрипт использует модуль Az Azure PowerShell и отвечает за следующие действия для узлов, работающих под управлением Windows или Linux:

   - создание и установка сертификатов на клиентском компьютере;
   - вычисление будущего диапазона IP-адресов подсети VPN-шлюза;
   - создание подсети шлюза;
   - развертывание шаблона Azure Resource Manager для подключения VPN-шлюза к подсети VPN.

     ```powershell
     $scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/attach-vpn-gateway'

     $parameters = @{
       subscriptionId = '<subscriptionId>'
       resourceGroupName = '<resourceGroupName>'
       virtualNetworkName = '<virtualNetworkName>'
       certificateNamePrefix  = '<certificateNamePrefix>'
       }

     Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/attachVPNGateway.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters, $scriptUrlBase
     ```

3. Вставьте сценарий в окне PowerShell и укажите необходимые параметры. Значения для `<subscriptionId>`, `<resourceGroup>` и `<virtualNetworkName>` должны соответствовать тем, которые используются в кратком руководстве по [созданию управляемого экземпляра](instance-create-quickstart.md). В качестве значения для `<certificateNamePrefix>` можно указать строку на ваш выбор.

4. Выполните следующий сценарий PowerShell.

> [!IMPORTANT]
> Не выполняйте никаких действий, пока не завершится выполнение скрипта PowerShell.

## <a name="create-a-vpn-connection"></a>Создание VPN-подключения

1. Войдите на [портал Azure](https://portal.azure.com/).
2. Откройте группу ресурсов, в которой вы создали шлюз виртуальной сети, а затем откройте ресурс шлюза виртуальной сети.
3. Щелкните **Конфигурация "точка — сеть"**, а затем — **Скачивание VPN-клиента**.

    ![Скачивание VPN-клиента](./media/point-to-site-p2s-configure/download-vpn-client.png)  
4. На локальном клиентском компьютере извлеките файлы из ZIP-файла и откройте папку с извлеченными файлами.
5. Откройте папку **WindowsAmd64** и файл **VpnClientSetupAmd64.exe**.
6. Если появится сообщение **Windows protected your PC** (Система Windows защитила ваш компьютер), щелкните **Дополнительно**, а затем **Выполнить в любом случае**.

    ![Установка VPN-клиента](./media/point-to-site-p2s-configure/vpn-client-defender.png)
7. Если откроется диалоговое окно Контроль учетных записей пользователей, нажмите кнопку **Да**, чтобы продолжить.
8. В диалоговом окне с указанием на вашу виртуальную сеть выберите вариант **Да**, чтобы установить VPN-клиент для нее.

## <a name="connect-to-the-vpn-connection"></a>Установка соединения с VPN-подключением

1. На локальном клиентском компьютере последовательно выберите элементы **Сеть и Интернет** и **VPN**. Затем выберите виртуальную сеть Управляемого экземпляра SQL для подключения к этой виртуальной сети. На следующем рисунке такая виртуальная сеть называется **MyNewVNet**.

    ![VPN-подключение](./media/point-to-site-p2s-configure/vpn-connection.png)  
2. Выберите **Подключиться**.
3. В диалоговом окне выберите **Подключиться**.

    ![Снимок экрана, на котором выделена кнопка "Подключиться".](./media/point-to-site-p2s-configure/vpn-connection2.png)  
4. При появлении сообщения о том, что диспетчеру подключений требуется более высокий уровень привилегий, чтобы обновить таблицу маршрутизации, выберите элемент **Продолжить**.
5. Чтобы продолжить, выберите **Да** в диалоговом окне контроля учетных записей.

   Вы установили VPN-подключение к виртуальной сети Управляемого экземпляра SQL.

    ![Снимок экрана: выделено сообщение "Подключено", которое отображается после установки подключения.](./media/point-to-site-p2s-configure/vpn-connection-succeeded.png)  

## <a name="connect-with-ssms"></a>Подключение к SSMS

1. На локальном клиентском компьютере откройте SQL Server Management Studio.
2. В диалоговом окне **Подключение к серверу** в поле **Имя сервера** введите полное **имя узла** для управляемого экземпляра.
3. Выберите **Проверка подлинности SQL Server**, укажите имя пользователя и пароль, а затем нажмите кнопку **Подключиться**.

    ![Подключение SSMS](./media/point-to-site-p2s-configure/ssms-connect.png)  

После подключения можно будет просматривать системные и пользовательские базы данных в узле "Базы данных". Также можно просматривать различные объекты в узлах безопасности, серверных объектов, репликации, управления, агента SQL Server и XEvent Profiler.

## <a name="next-steps"></a>Дальнейшие действия

- Краткое руководство по подключению с виртуальной машины Azure см. в статье о [настройке подключения "точка — сеть"](point-to-site-p2s-configure.md).
- Обзор вариантов подключения для приложений см. в статье [Подключение приложений к Управляемому экземпляру SQL](connect-application-instance.md).
- Чтобы восстановить существующую базу данных SQL Server из локальной среды в управляемом экземпляре, можно использовать [Azure Database Migration Service для миграции](../../dms/tutorial-sql-server-to-managed-instance.md) или [команду T-SQL RESTORE](restore-sample-database-quickstart.md) для восстановления из файла резервной копии базы данных.