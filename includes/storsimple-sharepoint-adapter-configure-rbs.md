---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 45b2ac73ffcb4b777572f9ad62d8aae0e8277e1c
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95557506"
---
> [!NOTE]
> При внесении изменений в адаптер StorSimple для настройки SharePoint RBS вы должны войти в систему с учетной записью, входящей в группу «Администраторы домена». Кроме того, необходимо получить доступ к странице «Конфигурация» из браузера, работающего на том же узле центра администрирования.
> 
> 

#### <a name="to-configure-rbs"></a>Настройка RBS
1. Откройте страницу центра администрирования SharePoint и перейдите к странице **Параметры системы**. 
2. Щелкните в разделе **Azure StorSimple** команду **Configure StorSimple Adapter** (Настроить адаптер StorSimple).
   
    ![Настройка адаптера StorSimple](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS1-include.png) 
3. Выполните следующие действия на странице **Configure StorSimple Adapter** (Настройка адаптера StorSimple):
   
   1. Убедитесь, что установлен флажок **Enable editing path** (Включить изменение пути).
   2. В текстовом поле введите UNC-путь хранилища больших двоичных объектов.
      
      > [!NOTE]
      > Том хранилища больших двоичных объектов должен быть размещен на томе iSCSI, настроенном на устройстве StorSimple.

   3. Нажмите кнопку **Включить** под каждой из баз данных контента, которую требуется настроить для удаленного хранилища.
      
      > [!NOTE]
      > Хранилище больших двоичных объектов должно быть общим и доступным всем интерфейсным веб-серверам (WFE), а учетная запись пользователя, настроенная на ферме серверов SharePoint, должна иметь доступ к общей папке.
      
      ![Включение поставщика RBS](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS2-include.png)
      
      При включении или отключении RBS также появится следующее сообщение.
      
      ![Настройка включения и выключения адаптера StorSimple](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_ConfigureStorSimpleAdapterEnableDisableMessage-include.png)

   4. Нажмите кнопку **Обновить**, чтобы применить конфигурацию. При нажатии кнопки **Обновить** состояние конфигурации RBS будет обновлено на всех интерфейсных веб-серверах, и вся ферма получит поддержку RBS. Появится указанное ниже сообщение.
      
      ![Сообщение о конфигурации адаптера](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS3-include.png)
      
      > [!NOTE]
      > Если вы настраиваете RBS для фермы SharePoint с очень большим количеством баз данных (более 200), время ожидания веб-страницы центра администрирования SharePoint может истекает. Если это происходит, обновите страницу. Это не влияет на процесс настройки.

4. Проверьте конфигурацию.
   
   1. Войдите на веб-сайт центра администрирования SharePoint и перейдите на страницу **Configure StorSimple Adapter** (Настройка адаптера StorSimple).
   2. Проверьте сведения о конфигурации, чтобы убедиться в том, что они соответствуют введенным параметрам. 
5. Проверьте правильность работы RBS.
   
   1. Отправьте документ в SharePoint. 
   2. Перейдите по UNC-пути, который вы настроили. Убедитесь, что структура каталога RBS создана и содержит отправленный объект.
6. (Необязательно) Можно использовать командлет PowerShell `Migrate()` для Microsoft RBS, включенный в SharePoint, чтобы перенести существующее содержимое больших двоичных объектов на устройство StorSimple. Дополнительные сведения см. в статье [Перенос контента в удаленное хранилище больших двоичных объектов или из него в SharePoint 2013][6] или [Migrate content into or out of RBS (SharePoint Foundation 2010) (Перенос содержимого в RBS или из него (SharePoint Foundation 2010))7][7].
7. (Необязательно.) При тестовой установке можно следующим образом убедиться, что большие двоичные объекты перемещены из базы данных содержимого. 
   
   1. Запустите SQL Management Studio
   2. Выполните запрос ListBlobsInDB_2010.sql или ListBlobsInDB_2013.sql, как показано ниже.
      
      ```
      **ListBlobsInDB_2013.sql**
      
        USE WSS_Content
        GO
      
        SELECT DocStreams.DocId,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               DocStreams.RbsId,
               TimeLastModified
      
        FROM DocStreams
      
             INNER JOIN AllDocs ON DocStreams.DocId = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      
      **ListBlobsInDB_2010.sql**
      
        USE WSS_Content
        GO
      
        SELECT AllDocStreams.Id,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               RbsId,
               TimeLastModified
        FROM AllDocStreams
      
             INNER JOIN AllDocs ON AllDocStreams.Id = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      ```
      
      Если RBS настроена правильно, значение NULL должно отображаться в столбце SizeOfContentInDB любого объекта, отправленного и успешно перемещенного с помощью RBS.
8. (Необязательно.) После настройки RBS и перемещения содержимого больших двоичных объектов на устройство StorSimple можно переместить базу данных содержимого на устройство. Если необходимо переместить базу данных содержимого, рекомендуется настроить хранилище базы данных содержимого на устройстве как первичный том. Используйте рекомендации по работе с SQL Server для перемещения базы данных содержимого на устройство StorSimple. 
   
   > [!NOTE]
   > Перемещение базы данных содержимого на устройство поддерживается только для устройств серии StorSimple 8000 (функция не поддерживается для серий 5000 и 7000).
   
   Если вы храните большие двоичные объекты и базы данных содержимого в разных томах на устройстве StorSimple, рекомендуется настроить их в том же контейнере томов. Это гарантирует, что они будут подвергаться резервному копированию вместе.
   
   > [!WARNING]
   > Если RBS не включена, перемещать базу данных содержимого на устройство StorSimple не рекомендуется. Эта конфигурация не тестировалась.
   
9. Перейдите к следующему шагу: [Настройка сборки мусора](#configure-garbage-collection).

[6]: /SharePoint/administration/migrate-content-into-or-out-of-rbs
[7]: /previous-versions/office/sharepoint-foundation-2010/ff628255(v=office.14)