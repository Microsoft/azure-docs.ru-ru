---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 9c734ff03b1cf277c7e0967d8b76b1941434f414
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "86050287"
---
В этой процедуре будут выполнены следующие действия.

1. [Подготовка к запуску исполняемого файла программы обслуживания](#to-prepare-to-run-the-maintainer) .
2. [Подготовка базы данных содержимого и корзины к немедленному удалению потерянных больших двоичных объектов](#to-prepare-the-content-database-and-recycle-bin-to-immediately-delete-orphaned-blobs).
3. [Запуск Maintainer.exe](#to-run-the-maintainer).
4. [Восстановление базы данных содержимого и параметров корзины](#to-revert-the-content-database-and-recycle-bin-settings).

#### <a name="to-prepare-to-run-the-maintainer"></a>Подготовка к запуску программы обслуживания
1. На интерфейсном веб-сервере откройте консоль управления SharePoint 2013 от имени администратора.
2. Перейдите в папку *загрузочный диск*:\Program Files\Microsoft SQL Remote Blob Storage 10.50\Maintainer\..
3. Переименуйте **Microsoft.Data.SqlRemoteBlobs.Maintainer.exe.config** в **web.config**.
4. Используйте `aspnet_regiis -pdf connectionStrings` для расшифровки файла web.config.
5. В расшифрованном файле web.config в узле `connectionStrings` добавьте строку подключения для вашего экземпляра SQL Server и имя базы данных содержимого. См. следующий пример.
   
    `<add name="RBSMaintainerConnectionWSSContent" connectionString="Data Source=SHRPT13-SQL12\SHRPT13;Initial Catalog=WSS_Content;Integrated Security=True;Application Name=&quot;Remote Blob Storage Maintainer for WSS_Content&quot;" providerName="System.Data.SqlClient" />`
6. Используйте `aspnet_regiis –pef connectionStrings` для повторного шифрования файла web.config. 
7. Переименуйте web.config в Microsoft.Data.SqlRemoteBlobs.Maintainer.exe.config. 

#### <a name="to-prepare-the-content-database-and-recycle-bin-to-immediately-delete-orphaned-blobs"></a>Подготовка базы данных содержимого и корзины для немедленного удаления потерянных больших двоичных объектов
1. На SQL Server в SQL Management Studio выполните следующие запросы на обновление для целевой базы данных содержимого: 

    `use WSS_Content`

    `exec mssqlrbs.rbs_sp_set_config_value ‘garbage_collection_time_window’ , ’time 00:00:00’`

    `exec mssqlrbs.rbs_sp_set_config_value ‘delete_scan_period’ , ’time 00:00:00’`

2. На интерфейсном веб-сервере в разделе **Центр администрирования** измените **общие параметры веб-приложения** для необходимой базы данных содержимого, чтобы временно отключить корзину. Это действие также очистит корзину для всех связанных коллекций веб-сайтов. Для этого щелкните **центр администрирования**  ->  **Управление** приложениями  ->  **веб-приложения (Управление веб-приложениями)**  ->  **SharePoint-80**  ->  **Общие параметры приложения**. Для параметра **Состояние корзины задайте** значение **Выкл.**
   
    ![Общие параметры веб-приложения](./media/storsimple-sharepoint-adapter-garbage-collection/HCS_WebApplicationGeneralSettings-include.png)

#### <a name="to-run-the-maintainer"></a>Запуск программы обслуживания
* На интерфейсном веб-сервере в консоли управления SharePoint 2013 запустите программу обслуживания следующим образом:
  
    `Microsoft.Data.SqlRemoteBlobs.Maintainer.exe -ConnectionStringName RBSMaintainerConnectionWSSContent -Operation GarbageCollection -GarbageCollectionPhases rdo`
  
  > [!NOTE]
  > В данный момент для StorSimple поддерживается только операция `GarbageCollection`. Также обратите внимание, что параметры для Microsoft.Data.SqlRemoteBlobs.Maintainer.exe чувствительны к регистру. 
  > 
  > 

#### <a name="to-revert-the-content-database-and-recycle-bin-settings"></a>Восстановление базы данных содержимого и параметров корзины
1. На SQL Server в SQL Management Studio выполните следующие запросы на обновление для целевой базы данных содержимого:
   
      `use WSS_Content`
   
      `exec mssqlrbs.rbs_sp_set_config_value ‘garbage_collection_time_window’ , ‘days 30’`
   
      `exec mssqlrbs.rbs_sp_set_config_value ‘delete_scan_period’ , ’days 30’`
   
      `exec mssqlrbs.rbs_sp_set_config_value ‘orphan_scan_period’ , ’days 30’`
2. На интерфейсном веб-сервере в разделе **Центр администрирования** измените **общие параметры веб-приложения** для необходимой базы данных содержимого, чтобы снова включить корзину. Для этого щелкните **центр администрирования**  ->  **Управление** приложениями  ->  **веб-приложения (Управление веб-приложениями)**  ->  **SharePoint-80**  ->  **Общие параметры приложения**. Для параметра «Состояние корзины» задайте значение **ВКЛ**.

