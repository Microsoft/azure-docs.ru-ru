---
title: Синхронизация конфигурации сети для службы приложений Azure
titleSuffix: Azure SQL Managed Instance
description: В этой статье описывается, как синхронизировать конфигурацию сети для плана размещения службы приложений Azure с Управляемый экземпляр Azure SQL.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova
ms.date: 12/13/2018
ms.openlocfilehash: e102aaace15d065d02e44fa06655827068020959
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91620228"
---
# <a name="sync-networking-configuration-for-azure-app-service-hosting-plan-with-azure-sql-managed-instance"></a>Синхронизация конфигурации сети для плана размещения службы приложений Azure с помощью Управляемый экземпляр Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Возможно, это связано с тем, что несмотря на то, что [приложение интегрировано с виртуальной сетью Azure](../../app-service/web-sites-integrate-with-vnet.md), невозможно установить соединение с SQL управляемый экземпляр. При обновлении или синхронизации конфигурация сети для плана обслуживания может решить эту проблему. 

## <a name="sync-network-configuration"></a>Синхронизация конфигурации сети 

Для этого выполните следующие действия:  

1. Перейдите к плану службы веб-приложений.

   ![Снимок экрана плана службы приложений](./media/azure-app-sync-network-configuration/app-service-plan.png)

2. Выберите **сети** , а затем **щелкните здесь для управления**.

   ![Снимок экрана: Управление планом службы](./media/azure-app-sync-network-configuration/manage-plan.png)

3. Выберите **виртуальную сеть** и щелкните **Синхронизировать сеть**.

   ![Снимок экрана: сеть синхронизации](./media/azure-app-sync-network-configuration/sync.png)

4. Дождитесь завершения синхронизации.
  
   ![Снимок экрана: Синхронизация выполнена](./media/azure-app-sync-network-configuration/sync-done.png)

Теперь вы можете попытаться восстановить подключение к Управляемый экземпляр SQL.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о настройке виртуальной сети для Управляемый экземпляр SQL см. в статье [Архитектура виртуальной сети sql управляемый экземпляр](connectivity-architecture-overview.md) и [Настройка существующей виртуальной сети](vnet-existing-add-subnet.md).
