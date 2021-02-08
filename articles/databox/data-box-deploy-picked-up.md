---
title: Учебник по отправке Azure Data Box обратно | Документация Майкрософт
description: В этом руководстве содержатся сведения о возвращении Azure Data Box, а также о подготовке к отправке, доставке Data Box, проверке передачи данных и удалении данных из Data Box.
services: databox
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: tutorial
ms.date: 02/02/2020
ms.author: alkohli
ms.localizationpriority: high
ms.openlocfilehash: 228f837a8826612bbbadf2ca8c5ef339ab248397
ms.sourcegitcommit: ea822acf5b7141d26a3776d7ed59630bf7ac9532
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/03/2021
ms.locfileid: "99524998"
---
::: zone target="docs"

# <a name="tutorial-return-azure-data-box-and-verify-data-upload-to-azure"></a>Руководство по Отправка Azure Data Box и проверка передачи данных в Azure

::: zone-end

::: zone target="chromeless"

## <a name="return-data-box-and-verify-data-upload-to-azure"></a>Отправка Data Box и проверка передачи данных в Azure

::: zone-end

::: zone target="docs"

В этом руководстве объясняется, как отправить Azure Data Box и проверить передачу данных в Azure.

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Предварительные требования
> * Подготовка к отправке
> * Отправка Data Box в Майкрософт
> * Проверка передачи данных в Azure
> * Стирание данных из Data Box

## <a name="prerequisites"></a>Предварительные требования

Перед тем как начать, убедитесь в следующем:

* Вы ознакомились со статьей [Руководство. Копирование данных в Azure Data Box через SMB](data-box-deploy-copy-data.md).
* Задания копирования завершены, и на странице **Подключение и копирование** ошибок нет. **Подготовка к отправке** недоступна, если задания копирования не завершены или если на странице **Подключение и копирование** есть ошибки.

## <a name="prepare-to-ship"></a>Подготовка к отправке

[!INCLUDE [data-box-prepare-to-ship](../../includes/data-box-prepare-to-ship.md)]

::: zone-end

::: zone target="chromeless"

После копирования данных устройство подготавливается и отправляется. Когда устройство поступает в центр обработки данных Azure, данные автоматически передаются в Azure.

## <a name="prepare-to-ship"></a>Подготовка к отправке

Прежде чем приступать к отправке, убедитесь, что задания копирования завершены.

1. Перейдите к странице **Подготовка к отправке** в локальном пользовательском веб-интерфейсе и начните подготовку к отправке.
2. Отключите устройство от локального веб-интерфейса. Отсоедините кабели от устройства.

Дальнейшие действия зависят от того, в каком регионе вы возвращаете устройство.

::: zone-end

::: zone target="docs"

## <a name="ship-data-box-back"></a>Отправка Data Box

Убедитесь, что копирование данных на устройство завершено и **подготовка к отправке** прошла успешно. Процедура отличается в зависимости от региона, в который вы отправляете устройство.

::: zone-end

## <a name="us-canada-europe"></a>[США, Канада, Европа](#tab/in-us-canada-europe)

Если вы возвращаете устройство в США или Канаде, выполните следующие действия:

1. Убедитесь, что устройство отключено и кабели отсоединены.
2. Смотайте и аккуратно разместите шнур питания, входящий в комплект устройства, на задней панели устройства.
3. Убедитесь, что транспортная этикетка отображается на дисплее E-ink и запланируйте отправку со своим курьером. Если метка повреждена, утеряна или не отображается на дисплее E-ink, обратитесь в службу поддержки Майкрософт. На портале Azure выберите **Обзор > Скачать транспортную этикетку**, если это посоветует специалист службы поддержки. Скачайте транспортную этикетку и прикрепите ее на устройство. 
4. Запланируйте визит курьера UPS для отправки посылки, если возвращаете устройство. Чтобы запланировать отправку с курьером:

    * Позвоните в местное отделение UPS (бесплатный номер конкретной страны/региона).
    * При вызове укажите обратный номер для отслеживания доставки, как показано на дисплее E-ink или на напечатанной этикетке. Если вы не укажете номер для отслеживания, служба доставки UPS потребует дополнительную плату во время приема посылки.
    * Если при планировании приема посылки возникнут какие-либо проблемы или с вас потребуют дополнительную плату, обратитесь к сотрудникам отдела эксплуатации Azure Data Box. Отправьте сообщение электронной почты с запросом кода на адрес [adbops@microsoft.com](mailto:adbops@microsoft.com).

    Вместо планирования отправки с курьером вы также можете оставить Data Box в ближайшем отделении этой курьерской службы.
4. После того как курьер примет и проверит Data Box, состояние заказа на портале обновится до **Принято курьером**. Также отображается идентификатор отслеживания.

::: zone target="chromeless"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## <a name="erasure-of-data-from-data-box"></a>Стирание данных из Data Box

Когда передача данных в Azure завершится, Data Box удалит данные с дисков согласно рекомендациям [NIST SP 800-88 в редакции 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

## <a name="australia"></a>[Австралия](#tab/in-australia)

В центрах обработки данных Azure в Австралии предусмотрена процедура предварительного уведомления о посылке. Об отправке посылки необходимо заранее предупреждать. Выполните следующие действия для отправки в Австралии.

1. Сохраните оригинальную коробку, в которой было доставлено устройство, для обратной отправки.
2. Убедитесь, что копирование данных на устройство завершено и **Подготовка к отправке** прошла успешно.
3. Выключите устройство и отсоедините кабели.
4. Смотайте и аккуратно разместите шнур питания, входящий в комплект устройства, на задней панели устройства.
5. Закажите отправку через Интернет по ссылке [DHL](https://mydhl.express.dhl/au/en/schedule-pickup.html#/schedule-pickup#label-reference).

::: zone target="chromeless"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## <a name="erasure-of-data-from-data-box"></a>Стирание данных из Data Box

Когда передача данных в Azure завершится, Data Box удалит данные с дисков согласно рекомендациям [NIST SP 800-88 в редакции 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

## <a name="japan"></a>[Япония](#tab/in-japan)

1. Сохраните оригинальную коробку, в которой было доставлено устройство, для обратной отправки.
2. Выключите устройство и отсоедините кабели.
3. Смотайте и аккуратно разместите шнур питания, входящий в комплект устройства, на задней панели устройства.
4. Укажите название и адрес своей компании в качестве сведений об отправителе в квитанции.
5. Отправьте сообщение электронной почты в компанию Quantium Solutions, используя указанный ниже шаблон.

    * Если квитанция при доставке наложенным платежом Japan Post отсутствует, сообщите об этом в письме. В компании Quantium Solutions распорядятся, чтобы служба Japan Post организовала отправку посылки и оформила квитанцию.
    * Если у вас несколько заказов, отправьте отдельное электронное письмо для каждого из них.

    ```
    To: Customerservice.JP@quantiumsolutions.com
    Subject: Pickup request for Azure Data Box｜Job name： 
    Body:
    - Japan Post Yu-Pack tracking number (reference number)：
    - Requested pickup date：mmdd (Select a requested time slot from below).
    a. 08：00-13：00 
    b. 13：00-15：00 
    c. 15：00-17：00 
    d. 17：00-19：00 
    ```

6. Компания Quantium Solutions отправит подтверждение по электронной почте после оформления отправки. В подтверждении по электронной почте также содержатся сведения о квитанции при доставке наложенным платежом.

При необходимости вы можете обратиться в службу поддержки Quantium Solutions (на японском языке), используя следующие сведения:

* Электронная почта: Customerservice.JP@quantiumsolutions.com 
* Телефон: 03-5755-0150 

::: zone target="chromeless"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## <a name="erasure-of-data-from-data-box"></a>Стирание данных из Data Box
 
Когда передача данных в Azure завершится, Data Box удалит данные с дисков согласно рекомендациям [NIST SP 800-88 в редакции 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

## <a name="singapore"></a>[Сингапур](#tab/in-singapore)

1. Сохраните оригинальную коробку, в которой было доставлено устройство, для обратной отправки.
2. Запишите номер для отслеживания (отображается в виде справочного номера на странице **Подготовка к отправке** локального пользовательского веб-интерфейса Data Box). Этот номер будет доступен после успешного завершения шага **Подготовка к отправке**. Скачайте этикетку отгрузки на этой странице и прикрепите ее к упаковочной коробке.
3. Выключите устройство и отсоедините кабели.
4. Смотайте и аккуратно разместите шнур питания, входящий в комплект устройства, на задней панели устройства. 
5. Отправьте сообщение электронной почты в службу поддержки SingPost, используя следующий шаблон с номером отслеживания.

    ```
    To: kadcustcare@singpost.com
    Subject: Microsoft Azure Pickup - OrderName 
    Body: 
        1. Requestor name  
        2. Requestor contact number
        3. Requestor collection address
        4. Preferred collection date
    ```

   > [!NOTE]
   > Для запросов на резервирование, полученных в рабочий день:
   > * До 15:00 отправка будет выполняться в следующий рабочий день в диапазоне от 9 до 13:00.
   > * После 15:00 отправка будет выполняться в следующий рабочий день в диапазоне от 9 до 18:00.  

::: zone target="chromeless"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## <a name="erasure-of-data-from-data-box"></a>Стирание данных из Data Box

Когда передача данных в Azure завершится, Data Box удалит данные с дисков согласно рекомендациям [NIST SP 800-88 в редакции 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

## <a name="south-africa"></a>[ЮАР](#tab/in-sa)

1. Для обратной отправки упакуйте устройство в оригинальную упаковку.
2. Смотайте и аккуратно разместите шнур питания, входящий в комплект устройства, на задней панели устройства.
3. Запишите номер отслеживания (отображается в виде справочного номера на странице **Подготовка к отправке** локального пользовательского веб-интерфейса Data Box). Этот номер будет доступен после успешного выполнения подготовки к отправке. Скачайте этикетку отгрузки на этой странице и прикрепите ее к упаковочной коробке.
4. Запросите код возврата у специалистов Azure Data Box. Этот код требуется для доставки посылки обратно в центр обработки данных. Отправьте сообщение электронной почты с запросом кода на адрес [adbops@microsoft.com](mailto:adbops@microsoft.com). Разборчиво запишите этот код на этикетке отгрузки рядом с адресом возврата.
5. Закажите отправку с помощью DHL, выбрав один из следующих вариантов:
 
   * Закажите отправку через Интернет на сайте [DHL Express (ЮАР), выбрав **Schedule a Pickup** (Запланировать вывоз)](https://mydhl.express.dhl/za/en/schedule-pickup.html#/schedule-pickup#label-reference).
   * Отправьте сообщение электронной почты на [Priority.Support@dhl.com](mailto:Priority.Support@dhl.com), используя следующий шаблон:

     ```output
     To: Priority.Support@dhl.com
     Subject: Pickup request for Microsoft Azure
     Body: Need pick up for the below shipment
       *  DHL tracking number: (reference number/waybill number)
       *  Requested pickup date: yyyy/mm/dd;time:HH MM
       *  Shipper contact: (company name)
       *  Contact person: 
       *  Phone number: 
       *  Full physical address: 
       *  Item to be collected: Azure Dt
     ```

    * Вы также можете отправить посылку в ближайшей точке обслуживания DHL.

6. Если возникнут проблемы, отправьте на адрес [Priority.Support@dhl.com](mailto:Priority.Support@dhl.com) сообщение электронной почты с их описанием, указав номер накладной в строке "Тема:". Вы можете также позвонить по телефону +27(0)119213902.

::: zone target="chromeless"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## <a name="erasure-of-data-from-data-box"></a>Стирание данных из Data Box

Когда передача данных в Azure завершится, Data Box удалит данные с дисков согласно рекомендациям [NIST SP 800-88 в редакции 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

## <a name="hong-kong"></a>[Специальный административный регион Гонконг](#tab/in-hk)

1. Для обратной отправки упакуйте устройство в оригинальную упаковку.
2. Смотайте и аккуратно разместите шнур питания, входящий в комплект устройства, на задней панели устройства.
3. Позвоните на горячую линию **Quantium Solutions** по телефону **(852) 2318 1213** в рабочее время (9:00–18:00 с понедельника по пятницу).  
4. Укажите отправку в Microsoft Azure, идентификационный номер и номер отслеживания (выше штрих-кода) на этикетке отгрузки, чтобы организовать отправку.
5. Вы получите устное подтверждение планирования отправки. Если курьер не явился за посылкой, обратитесь на горячую линию Quantium Solutions, чтобы организовать доставку иным способом.
6. При заказе отправки с помощью Quantium Solutions предоставьте подтверждение отделу эксплуатации [Microsoft Data Box (Азия)](mailto:adbo@microsoft.com), используя следующий шаблон:

    ```output
    To: adbo@microsoft.com
    Subject: Microsoft Data Box Job: [order name] has completed copy
    Body:
    We have confirmed the pickup details with Quantium Solutions.

       * Requestor name:
       * Requestor contact number:
       * Pickup Date:  
       * Pickup time:
    ```

Если возникнут проблемы, отправьте на адрес отдела эксплуатации Data Box Operations в Азии ([adbo@microsoft.com](mailto:adbo@microsoft.com)) сообщение электронной почты с их описанием, указав имя задания в строке "Тема:".

::: zone target="chromeless"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## <a name="erasure-of-data-from-data-box"></a>Стирание данных из Data Box
 
Когда передача данных в Azure завершится, Data Box удалит данные с дисков согласно рекомендациям [NIST SP 800-88 в редакции 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

<!--## [In Korea](#tab/in-korea) 

1. Retain the original box used to ship the device for return shipment.
2. Note down the tracking number (shown as reference number on the **Prepare to Ship** page of the Data Box local web UI). The tracking number is available after the **Prepare to ship** step successfully completes. Download the shipping label from this page and paste on the packing box. 
3. Power off the device and remove the cables.
4. Spool and securely place the power cord that was provided with the device in the back of the device. 

Request pickup  
If consignment note is present:  

1. Call Quantium Solutions International hotline at 070-8231-1418 during office hours (10 AM to 5 PM, Monday to Friday). Quote Microsoft Azure pickup and the service request number to arrange for a collection.
2. If the hotline is busy, email microsoft@rocketparcel.com, with the email subject Microsoft Azure Pickup and the service request number as reference.  
3. If the courier does not arrive for collection, call Quantium Solutions International hotline for alternate arrangements.  
4. You will receive an email confirmation for the pickup schedule.  

Exception process
If the consignment note is not present:
1. Call Quantium Solutions International hotline at 070-8231-1418 during office hours (10 AM to 5 PM, Monday to Friday). Quote Microsoft Azure pickup and the service request number. Specify that you need a new consignment note to arrange for a collection. Provide sender (customer), receiver information (Azure datacenter), and reference number (service request number).
2. If the hotline is busy, email microsoft@rocketparcel.com, with the email subject Microsoft Azure Pickup and the service request number as reference.
3. If the courier does not arrive for collection, call Quantium Solutions International hotline for alternate arrangements.
4. You get a verbal confirmation if request is made via telephone.  
::: zone target="chromeless"

## Verify data upload to Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## Erasure of data from Data Box
 
Once the upload to Azure is complete, the Data Box erases the data on its disks as per the [NIST SP 800-88 Revision 1 guidelines](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

::: zone target="docs"

[!INCLUDE [data-box-verify-upload-return](../../includes/data-box-verify-upload-return.md)]

::: zone-end
-->

## <a name="self-managed"></a>[Cамостоятельное управление](#tab/in-selfmanaged)

Если вы используете Data Box в государственной организации США, в Японии, Сингапуре, Корее, Индии, Южной Африке, Соединенном Королевстве, Западной Европе или Австралии и выбрали при создании заказа самостоятельное управление отгрузкой, следуйте этим инструкциям.

1. Запишите код авторизации, который отобразится на странице **Подготовка к отправке** локального пользовательского веб-интерфейса Data Box после успешного выполнения этого шага.
2. Выключите устройство и отсоедините кабели. Смотайте и аккуратно разместите шнур питания, входящий в комплект устройства, на задней панели устройства.
3. Когда будете готовы вернуть устройство, отправьте сообщение электронной почты в отдел эксплуатации Azure Data Box, используя указанный ниже шаблон.

    ```
    To: adbops@microsoft.com
    Subject: Request for Azure Data Box drop-off for order: 'orderName'
    Body:
        1. Order name  
        2. Authorization code available after Prepare to Ship has completed [Yes/No]  
        3. Contact name of the person dropping off. You will need to display a Government approved ID during the drop off.
    ```

::: zone target="chromeless"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload](../../includes/data-box-verify-upload.md)]

## <a name="erasure-of-data-from-data-box"></a>Стирание данных из Data Box
 
Когда передача данных в Azure завершится, Data Box удалит данные с дисков согласно рекомендациям [NIST SP 800-88 в редакции 1](https://csrc.nist.gov/News/2014/Released-SP-800-88-Revision-1,-Guidelines-for-Medi).

::: zone-end

---

::: zone target="docs"

## <a name="verify-data-upload-to-azure"></a>Проверка передачи данных в Azure

[!INCLUDE [data-box-verify-upload-return](../../includes/data-box-verify-upload-return.md)]

::: zone-end

