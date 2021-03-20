---
title: Защиты службы Azure Data Lake Analytics, используемой для нескольких пользователей
description: Сведения о настройке нескольких пользователей для запуска заданий в службе Azure Data Lake Analytics.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 05/30/2018
ms.openlocfilehash: 52bdd80dba1d5de150f403199fbd5c14d188a410
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "92220148"
---
# <a name="configure-user-access-to-job-information-to-job-information-in-azure-data-lake-analytics"></a>Настройка доступа пользователей к сведениям о задании в Azure Data Lake Analytics 

В Azure Data Lake Analytics (ADLA) для запуска заданий можно использовать несколько учетных записей пользователей или субъектов-служб. 

Для того, чтобы те же пользователи смогли увидеть подробные сведения о задании, у них должна быть возможность считывать содержимое папок заданий. Папки заданий размещаются в каталоге `/system/`. 

Если необходимые разрешения не настроены, пользователь может увидеть следующую ошибку: `Graph data not available - You don't have permissions to access the graph data.` 

## <a name="configure-user-access-to-job-information"></a>Настройка доступа пользователей к сведениям о задании

Чтобы настроить списки управления доступом в папках, можно использовать **мастер добавления пользователей**. Дополнительные сведения см. в разделе [Добавление нового пользователя](data-lake-analytics-manage-use-portal.md#add-a-new-user).

Если вам требуется более детализированный контроль или необходимо добавить разрешения в сценарий, тогда защитите папки, как показано ниже.

1. Предоставьте разрешения на **выполнение** (через список ACL для доступа) в корневой папке:
   - /
   
2. Предоставьте разрешения на **выполнение** и **чтение** (через список ACL для доступа и ACL по умолчанию) в папках, в которых содержатся папки задания. Например, для конкретного задания, которое было запущено 25 мая 2018 года, к этим папкам можно получить доступ с помощью:
   - /system
   - /system/jobservice
   - /system/jobservice/jobs
   - /system/jobservice/jobs/Usql
   - /system/jobservice/jobs/Usql/2018
   - /system/jobservice/jobs/Usql/2018/05
   - /system/jobservice/jobs/Usql/2018/05/25
   - /system/jobservice/jobs/Usql/2018/05/25/11
   - /system/jobservice/jobs/Usql/2018/05/25/11/01
   - /system/jobservice/jobs/Usql/2018/05/25/11/01/b074bd7a-1448-d879-9d75-f562b101bd3d

## <a name="next-steps"></a>Дальнейшие действия
[Добавление нового пользователя](data-lake-analytics-manage-use-portal.md#add-a-new-user)
