---
title: Включение прозрачное шифрование данных для Stretch Database
description: Включение прозрачного шифрования данных (TDE) для SQL Server Stretch Database в Azure
services: sql-server-stretch-database
documentationcenter: ''
ms.assetid: a44ed8f5-b416-4c41-9b1e-b7271f10bdc3
ms.service: sql-server-stretch-database
ms.workload: data-management
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 06/14/2016
author: blazem-msft
ms.author: blazem
ms.reviewer: jroth
manager: jroth
ms.custom: seo-lt-2019
ms.openlocfilehash: b016987162cc8202b7ad28d4dd8e5ab2953469d1
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95024251"
---
# <a name="enable-transparent-data-encryption-tde-for-stretch-database-on-azure"></a>Включение прозрачного шифрования данных (TDE) для Stretch Database в Azure
> [!div class="op_single_selector"]
> * [Портал Azure](sql-server-stretch-database-encryption-tde.md)
> * [TSQL](sql-server-stretch-database-tde-tsql.md)
>
>

Прозрачное шифрование данных (TDE) помогает защититься от угрозы вредоносных атак за счет шифрования и расшифровки базы данных, связанных резервных копий и файлов журналов транзакций при хранении в реальном времени, не внося изменения в само приложение.

При использовании TDE хранилище всей базы данных шифруется с помощью симметричного ключа, который называется ключом шифрования базы данных. Ключ шифрования базы данных защищается встроенным сертификатом сервера. Каждый сервер Azure обладает уникальным встроенным сертификатом. Майкрософт автоматически меняет эти сертификаты каждые 90 дней. Общее описание TDE см. в статье [Прозрачное шифрование данных (TDE)].

## <a name="enabling-encryption"></a>Включение шифрования
Чтобы включить прозрачное шифрование для базы данных Azure, где хранятся данные, перенесенные из Базы данных SQL Server Stretch, выполните следующее.

1. Откройте базу данных в [портал Azure](https://portal.azure.com)
2. В колонке базы данных нажмите кнопку **Параметры** .
3. Выберите параметр **прозрачное шифрование данных** ![ на снимке экрана портал Azure, в котором отображается колонка параметры. В разделе Общие выделяется прозрачное шифрование данных.][1]
4. Выберите параметр **вкл** ., а затем нажмите кнопку **сохранить** 
    ![ снимок экрана портал Azure, чтобы отобразить колонку прозрачное шифрование данных. Шифрование данных включено, и кнопка Сохранить выделена.][2]

## <a name="disabling-encryption"></a>Отключение шифрования
Чтобы отключить прозрачное шифрование для базы данных Azure, где хранятся данные, перенесенные из Базы данных SQL Server Stretch, выполните следующее.

1. Откройте базу данных в [портал Azure](https://portal.azure.com)
2. В колонке базы данных нажмите кнопку **Параметры** .
3. Выберите параметр **Прозрачное шифрование данных** . 
4. Выберите параметр **Отключить**, а затем — **Сохранить**.

<!--Anchors-->
[Прозрачное шифрование данных (TDE)]: /sql/relational-databases/security/encryption/transparent-data-encryption


<!--Image references-->
[1]: ./media/sql-server-stretch-database-encryption-tde/stretchtde1.png
[2]: ./media/sql-server-stretch-database-encryption-tde/stretchtde2.png


<!--Link references-->