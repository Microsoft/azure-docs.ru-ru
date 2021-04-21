---
title: Развертывание Azure Monitor для решений SAP на портале Azure
description: Развертывание Azure Monitor для решений SAP на портале Azure
author: sameeksha91
ms.author: sakhare
ms.topic: how-to
ms.service: virtual-machines-sap
ms.date: 08/17/2020
ms.openlocfilehash: 02c0801aa0425db96a1e6f71f248c795e81b5ddf
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 04/07/2021
ms.locfileid: "106554065"
---
# <a name="deploy-azure-monitor-for-sap-solutions-with-azure-portal"></a>Развертывание Azure Monitor для решений SAP на портале Azure

Azure Monitor для ресурсов решений SAP можно создать с помощью [портала Azure](https://azure.microsoft.com/features/azure-portal). Этот метод предоставляет пользовательский интерфейс на основе браузера для развертывания Azure Monitor для решений SAP и настройки поставщиков.

## <a name="sign-in-to-azure-portal"></a>Вход на портал Azure

Войдите на портал Azure по адресу https://portal.azure.com.

## <a name="create-monitoring-resource"></a>Создание ресурса мониторинга

1. Выберите **Azure Monitor для решений SAP** в **Azure Marketplace**.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-1.png" alt-text="На рисунке показано, как выбрать Azure Monitor для решений SAP в Azure Marketplace." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-1.png":::

2. На вкладке **Основные сведения** введите требуемые значения. Если применимо, можно использовать существующую рабочую область Log Analytics.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-2.png" alt-text="Отображение параметров конфигурации портала Azure." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-2.png":::

3. При выборе виртуальной сети убедитесь, что системы, которые вы хотите отслеживать, доступны в пределах этой виртуальной сети. 

   > [!IMPORTANT]
   > Выбор параметра **Общ. доступ** для совместного использования данных корпорацией Майкрософт позволяет нашим группам поддержки предоставлять дополнительную помощь.

## <a name="configure-providers"></a>Настройка поставщиков

### <a name="sap-hana-provider"></a>Поставщик SAP HANA 

1. Перейдите на вкладку **Поставщик**, чтобы добавить поставщиков, которые нужно настроить. Вы можете вписать нескольких поставщиков один за другим или добавить их после развертывания ресурса мониторинга. 

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-3.png" alt-text="Отображение вкладки &quot;Поставщик&quot; для добавления дополнительных поставщиков в Azure Monitor для решений SAP." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-3.png":::

2. Выберите **Добавить поставщика** и в раскрывающемся списке выберите **SAP HANA**. 

   > [!IMPORTANT]
   > Убедитесь, что поставщик SAP HANA настроен на узел "master" SAP HANA.

3. Введите частный IP-адрес для сервера HANA.

4. Введите имя базы данных, которую нужно создать. Вы можете выбрать любое название, но мы рекомендуем использовать **SYSTEMDB**, так как оно обеспечивает более широкий спектр областей наблюдения. 

5. Введите номер порта SQL, связанный с базой данных HANA. Номер порта должен быть в формате **[3]**  +  **[instance #]**  +  **[13]** . Например, 30013. 

6. Введите имя пользователя базы данных, которое вы хотите использовать. Убедитесь, что пользователь базы данных имеет назначенные роли **мониторинга** и **чтения каталога**. 

7. После выбора щелкните **Добавить поставщика**. Продолжайте добавлять дополнительных поставщиков по мере необходимости или выберите **Проверить и создать**, чтобы завершить развертывание.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-4.png" alt-text="Изображение параметров конфигурации при добавлении сведений о поставщике." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-4.png":::

### <a name="high-availability-cluster-pacemaker-provider"></a>Поставщик кластера высокой доступности (Pacemaker)

1. В раскрывающемся списке выберите **кластер высокой доступности (Pacemaker)** . 

   > [!IMPORTANT]
   > Чтобы настроить поставщик кластера высокой доступности (Pacemaker), убедитесь, что поставщик ha_cluster_provider установлен на каждом узле. Дополнительные сведения см. в разделе [Средство экспорта кластеров высокой доступности](https://github.com/ClusterLabs/ha_cluster_exporter#installation)

2. Введите конечную точку Prometheus в формате http://IP:9664/metrics. 
 
3. Введите идентификатор системы (SID), имя узла и название кластера.

4. После выбора щелкните **Добавить поставщика**. Продолжайте добавлять дополнительных поставщиков по мере необходимости или выберите **Проверить и создать**, чтобы завершить развертывание.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-5.png" alt-text="На рисунке показаны параметры, относящиеся к поставщику кластера высокой доступности Pacemaker." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-5.png":::


### <a name="os-linux-provider"></a>Поставщик ОС (Linux) 

1. Выберите ОС (Linux) из раскрывающегося списка 

>[!IMPORTANT]
> Чтобы настроить поставщика ОС (Linux), убедитесь, что на каждом узле (BareMetal или VM) установлена последняя версия Node_Exporter, которую вы хотите отслеживать. Используйте эту [ссылку] (https://prometheus.io/download/#node_exporter), чтобы найти последнюю версию. Дополнительные сведения см. в разделе  [Node_Exporter](https://github.com/prometheus/node_exporter).

2. Введите имя, которое будет идентификатором экземпляра BareMetal.
3. Введите конечную точку экспорта узла в формате http://IP:9100/metrics.

>[!IMPORTANT]
> Используйте частный IP-адрес узла Linux. Убедитесь, что ресурсы узла и AMS находятся в одной виртуальной сети. 

>[!Note]
> Порт брандмауэра "9100" должен быть открыт на узле Linux.
>Если используется firewall-cmd: firewall-cmd--permanent --add-port=9100/tcp firewall-cmd --reload, если используется ufw: ufw allow 9100/tcp ufw reload

>[!Tip]
> Если узел Linux является виртуальной машиной Azure, убедитесь, что все применимые группы безопасности сети разрешают входящий трафик через порт 9100 из "VirtualNetwork" в качестве источника.
 
5. После выбора щелкните  **Добавить поставщика**. Продолжайте добавлять дополнительных поставщиков по мере необходимости или выберите  **Проверить и создать** , чтобы завершить развертывание. 


### <a name="microsoft-sql-server-provider"></a>Поставщик Microsoft SQL Server

1. Перед добавлением поставщика Microsoft SQL Server необходимо выполнить следующий скрипт в SQL Server Management Studio, чтобы создать пользователя с соответствующими разрешениями, необходимыми для настройки поставщика.

   ```sql
   USE [<Database to monitor>]
   DROP USER [AMS]
   GO
   USE [master]
   DROP USER [AMS]
   DROP LOGIN [AMS]
   GO
   CREATE LOGIN [AMS] WITH PASSWORD=N'<password>', DEFAULT_DATABASE=[<Database to monitor>], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
   CREATE USER AMS FOR LOGIN AMS
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GRANT CONNECT TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW ANY DEFINITION TO AMS
   GRANT EXEC ON xp_readerrorlog TO AMS
   GO
   USE [<Database to monitor>]
   CREATE USER [AMS] FOR LOGIN [AMS]
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GO
   ``` 

2. Выберите **Добавить поставщика** и в раскрывающемся списке выберите **Microsoft SQL Server**. 

3. Заполните поля, используя сведения, связанные с Microsoft SQL Server. 

4. После выбора щелкните **Добавить поставщика**. Продолжайте добавлять дополнительных поставщиков по мере необходимости или выберите **Проверить и создать**, чтобы завершить развертывание.

     :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-6.png" alt-text="На рисунке показаны сведения о добавлении поставщика Microsoft SQL Server." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-6.png":::

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об [Azure Monitor для решений SAP](azure-monitor-overview.md)
