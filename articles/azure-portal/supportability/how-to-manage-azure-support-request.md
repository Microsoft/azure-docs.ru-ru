---
title: Управление запросом на поддержку Azure
description: В этой статье описывается, как просматривать запросы в службу поддержки, отправлять сообщения, изменять уровень серьезности запроса, предоставлять диагностические сведения в службе поддержки Azure, повторно открывать закрытый запрос на поддержку и отправлять файлы.
tags: billing
ms.assetid: 86697fdf-3499-4cab-ab3f-10d40d3c1f70
ms.topic: how-to
ms.date: 12/14/2020
ms.openlocfilehash: 4d0c03e0035f6b71a23891ac1691f5421c1bdb76
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102502524"
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

| 0-9, A-C    | D-G   | H-N         | O-Q   | R-T      | U-W        | X-Z     |
|-------------|-------|-------------|-------|----------|------------|---------|
| .7z         | .dat  | . Har        | .odx  | .rar     | .uccapilog | .xlam   |
| .a          | .db   | .hwl        | .oft  | RDL     | .uccplog   | .xlr    |
| .abc        | .dmp  | .ics        | .old  | .rdlc    | .udcx      | .xls    |
| .adm        | .do_  | .ini        | .one  | .re_     | .vb_       | .xlsb   |
| .aspx       | .doc  | .java       | .osd  | .remove  | .vbs_      | .xlsm   |
| .atf        | .docm | .jpg        | .out  | .ren     | .vcf       | .xlsx   |
| .b          | .docx | .ldf        | .p1   | .rename  | .vsd       | .xlt    |
| .ba_        | .dotm | .letterhead | .pcap | .rft     | .wdb       | .xltx   |
| .bak        | .dotx | .lo_        | .pdb  | .rpt     | .wks       | XML    |
| .blg        | .dtsx | .log        | .pdf  | .rte     | .wma       | XMLA   |
| .ca_        | .eds  | .lpk        | .piz  | .rtf     | .wmv       | .xps    |
| .cab        | .emf  | .manifest   | .pmls | .run     | .wmz       | .xsd    |
| .cap        | .eml  | .master     | .png  | .saz     | .wps       | .xsn    |
| .catx       | .emz  | .mdmp       | .potx | .sql     | .wpt       | .xxx    |
| .cfg        | .err  | .mof        | .ppt  | .sqlplan | .wsdl      | .z_     |
| .compressed | .etl  | .mp3        | .pptm | .stp     | .wsp       | .z01    |
| .config     | .evt  | .mpg        | .pptx | .svclog  | .wtl       | .z02    |
| .cpk        | .evtx | .ms_        | .prn  | .tdb     | -          | .zi     |
| .cpp        | .ex   | .msg        | .psf  | .tdf     | -          | .zi_    |
| .cs         | .ex_  | .mso        | .pst  | .text    | -          | .zip    |
| .csv        | .ex0  | .msu        | .pub  | .thmx    | -          | .zip_   |
| .cvr        | .frd  | .nfo        | -     | .tif     | -          | .zipp   |
| -           | .gif  | -           | -     | .trc     | -          | .zipped |
| -           | .guid | -           | -     | .ttd     | -          | .zippy  |
| -           | .gz   | -           | -     | .tx_     | -          | .zipx   |
| -           | -     | -           | -     | .txt     | -          | .zit    |
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
