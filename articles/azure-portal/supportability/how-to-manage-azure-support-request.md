---
title: Управление запросом на поддержку Azure
description: В этой статье описывается, как просматривать запросы в службу поддержки, отправлять сообщения, изменять уровень серьезности запроса, предоставлять диагностические сведения в службе поддержки Azure, повторно открывать закрытый запрос на поддержку и отправлять файлы.
tags: billing
ms.assetid: 86697fdf-3499-4cab-ab3f-10d40d3c1f70
ms.topic: how-to
ms.date: 12/14/2020
ms.openlocfilehash: d6c68dd341e0794a690b41b73ecc4be954db7359
ms.sourcegitcommit: 227b9a1c120cd01f7a39479f20f883e75d86f062
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 02/18/2021
ms.locfileid: "100653855"
---
# <a name="manage-an-azure-support-request"></a>Управление запросом на поддержку Azure

После [создания запроса на поддержку Azure](how-to-create-azure-support-request.md)вы можете управлять им в [портал Azure](https://portal.azure.com), как описано в этой статье. Вы также можете создавать запросы и управлять ими программно, используя запрос в службу [поддержки Azure REST API](/rest/api/support)или с помощью [Azure CLI](/cli/azure/azure-cli-support-request).

## <a name="view-support-requests"></a>Просмотр запросов на поддержку

Просмотрите сведения и состояние запросов на поддержку, перейдя по запросу **Справка и поддержка**  >   **все запросы на поддержку**.

:::image type="content" source="media/how-to-manage-azure-support-request/all-requests-lower.png" alt-text="Все запросы на поддержку":::

На этой странице можно выполнять поиск, фильтрацию и сортировку запросов на поддержку. Выберите запрос в службу поддержки, чтобы просмотреть подробные сведения, включая его серьезность и все сообщения, связанные с запросом.

## <a name="send-a-message"></a>Отправка сообщения

1. На странице **все запросы на поддержку** выберите запрос в службу поддержки.

1. На странице **запрос на поддержку** выберите **новое сообщение**.

1. Введите сообщение и нажмите кнопку **Отправить**.

## <a name="change-the-severity-level"></a>Изменение степени серьезности

> [!NOTE]
> Максимальный уровень серьезности зависит от [плана поддержки](https://azure.microsoft.com/support/plans).
>

1. На странице **все запросы на поддержку** выберите запрос в службу поддержки.

1. На странице **запроса в службу поддержки** выберите **изменить**.

    :::image type="content" source="media/how-to-manage-azure-support-request/change-severity.png" alt-text="Изменение важности запроса поддержки":::

1. В портал Azure показано одно из двух экранов в зависимости от того, назначен ли ваш запрос инженеру службы поддержки.

    - Если запрос не назначен, отобразится экран следующего вида. Выберите новый уровень серьезности, а затем нажмите кнопку **изменить**.

        :::image type="content" source="media/how-to-manage-azure-support-request/unassigned-can-change-severity.png" alt-text="Выберите новый уровень серьезности":::

    - Если ваш запрос был назначен, отобразится экран следующего вида. Нажмите кнопку **ОК**, а затем создайте [новое сообщение](#send-a-message) , чтобы запросить изменение на уровне серьезности.

        :::image type="content" source="media/how-to-manage-azure-support-request/assigned-cant-change-severity.png" alt-text="Не удается выбрать новый уровень серьезности":::

## <a name="share-diagnostic-information-with-azure-support"></a>Общий доступ к диагностическим сведениям с помощью службы поддержки Azure

При создании запроса на поддержку по умолчанию выбирается параметр **общий доступ к диагностическим сведениям** . Это позволяет службе поддержки Azure собирать [диагностические данные](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) из ресурсов Azure:

* Этот параметр нельзя снять после создания запроса.

* Если при создании запроса был снят параметр, его можно выбрать после создания запроса.

    1. На странице **все запросы на поддержку** выберите запрос в службу поддержки.
    
    1. На странице **запрос на поддержку** выберите **предоставить разрешение**, а затем нажмите кнопку **Да** и **ОК**.
    
        :::image type="content" source="media/how-to-manage-azure-support-request/grant-permission-manage.png" alt-text="Предоставление разрешений для диагностических сведений":::

## <a name="upload-files"></a>Отправка файлов

Можно использовать параметр отправки файлов для отправки диагностических файлов или других файлов, которые вы считаете релевантными для запроса в службу поддержки.

1. На странице **все запросы на поддержку** выберите запрос в службу поддержки.

1. На странице **запрос в службу поддержки** найдите файл и нажмите кнопку **Отправить**. Повторите эту процедуру, если имеется несколько файлов.

    :::image type="content" source="media/how-to-manage-azure-support-request/file-upload.png" alt-text="Отправка файла":::

### <a name="file-upload-guidelines"></a>Рекомендации по отправке файлов

При использовании параметра отправки файла следуйте приведенным ниже рекомендациям.

* Для защиты вашей конфиденциальности не добавляйте в загружаемые файлы никакие личные данные.
* Имя файла должно содержать не больше 110 знаков.
* Невозможно отправить больше одного файла.
* Размер файлов не может превышать 4 МБ.
* Все файлы должны иметь расширение имени файла, например *docx* или *xlsx*. В следующей таблице приведены расширения имен файлов, разрешенные для отправки.

| 0-9, A-C     | D-G   | H-M         | N-P   | R-T      | U-W        | X-Z     |
|-------------|-------|-------------|-------|----------|------------|---------|
| .7z         | .dat  | . Har        | .odx  | .rar     | .tdb       | .xlam   |
| .a          | .db   | .hwl        | .oft  | RDL     | .tdf       | .xlr    |
| .abc        | .dmp  | .ics        | .old  | .rdlc    | .text      | .xls    |
| .adm        | .do_  | .ini        | .one  | .re_     | .thmx      | .xlsb   |
| .aspx       | .doc  | .java       | .osd  | .reg     | .tif       | .xlsm   |
| .atf        | .docm | .jpg        | .out  | .remove  | .trc       | .xlsx   |
| .b          | .docx | .ldf        | .p1   | .ren     | .ttd       | .xlt    |
| .ba_        | .dotm | .letterhead | .pcap | .rename  | .tx_       | .xltx   |
| .bak        | .dotx | .lnk        | .pdb  | .rft     | .txt       | XML    |
| .bat        | .dtsx | .lo_        | .pdf  | .rpt     | .uccapilog | XMLA   |
| .blg        | .eds  | .log        | .piz  | .rte     | .uccplog   | .xps    |
| .ca_        | .emf  | .lpk        | .pmls | .rtf     | .udcx      | .xsd    |
| .cab        | .eml  | .manifest   | .png  | .run     | .vb_       | .xsn    |
| .cap        | .emz  | .master     | .potx | .saz     | .vbs_      | .xxx    |
| .catx       | .err  | .mdmp       | .ppt  | .sql     | .vcf       | .z_     |
| .cfg        | .etl  | .mof        | .pptm | .sqlplan | .vsd       | .z01    |
| .compressed | .evt  | .mp3        | .pptx | .stp     | .wdb       | .z02    |
| .config     | .evtx | .mpg        | .prn  | .svclog  | .wks       | .zi     |
| .cpk        | .ex   | .ms_        | .psf  | -        | .wma       | .zi_    |
| .cpp        | .ex_  | .msg        | .pst  | -        | .wmv       | .zip    |
| .cs         | .ex0  | .msi        | .pub  | -        | .wmz       | .zip_   |
| .csv        | .frd  | .mso        | -     | -        | .wps       | .zipp   |
| .cvr        | .gif  | .msu        | -     | -        | .wpt       | .zipped |
| -           | .guid | .nfo        | -     | -        | .wsdl      | .zippy  |
| -           | .gz   | -           | -     | -        | .wsp       | .zipx   |
| -           | -     | -           | -     | -        | .wtl       | .zit    |
| -           | -     | -           | -     | -        | -          | .zix    |
| -           | -     | -           | -     | -        | -          | .zzz    |

## <a name="close-a-support-request"></a>Закрыть запрос в службу поддержки

Если вам нужно закрыть запрос в службу поддержки, [отправьте сообщение](#send-a-message) с просьбой закрыть запрос.

## <a name="reopen-a-closed-request"></a>Повторное открытие закрытого запроса

Если необходимо повторно открыть закрытый запрос на поддержку, создайте [новое сообщение](#send-a-message), которое автоматически повторно откроет запрос.

## <a name="cancel-a-support-plan"></a>Отмена плана поддержки

Если вам нужно отменить план поддержки, см. статью [Отмена плана поддержки](../../cost-management-billing/manage/cancel-azure-subscription.md#cancel-a-support-plan).

## <a name="next-steps"></a>Дальнейшие действия

[Создание запроса на поддержку Azure](how-to-create-azure-support-request.md)

[REST API запросов в службу поддержки](/rest/api/support)
