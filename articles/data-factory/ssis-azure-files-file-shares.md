---
title: Открытие и сохранение файлов с помощью пакетов SSIS, развернутых в Azure
description: Узнайте, как открывать и сохранять файлы в локальной среде и в Azure при переносе пакетов SSIS, использующих локальные файловые системы, в службы SSIS в Azure.
ms.date: 06/27/2018
ms.topic: conceptual
ms.prod: sql
ms.technology: integration-services
author: swinarko
ms.author: sawinark
ms.reviewer: maghan
ms.openlocfilehash: 97ed9b6f3f21b6d9e220c1f4e17d4ba6b0d8b485
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98555514"
---
# <a name="open-and-save-files-on-premises-and-in-azure-with-ssis-packages-deployed-in-azure"></a>Открытие и сохранение файлов в локальной среде и в Azure с помощью пакетов SSIS, развернутых в Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

В этой статье описывается, как открывать и сохранять файлы в локальной среде и в Azure при переносе пакетов SSIS, использующих локальные файловые системы, в службы SSIS в Azure.

## <a name="save-temporary-files"></a>Сохранение временных файлов

Если при выполнении отдельного пакета необходимо хранить и обрабатывать временные файлы, для пакетов можно использовать текущую рабочую папку `.` или временную папку `%TEMP%` на узлах Azure-SSIS Integration Runtime.

## <a name="use-on-premises-file-shares"></a>Использование локальных общих папок

Чтобы и далее применять **локальные общие папки** при переносе пакетов, использующих локальные файловые системы, в службы SSIS в Azure, выполните указанные ниже действия.

1. Перенесите файлы из локальных файловых систем в локальные общие папки.

2. Присоедините локальные общие папки к виртуальной сети Azure.

3. Присоедините среду Azure-SSIS IR к той же виртуальной сети. Дополнительные сведения см. в разделе [Присоединение среды выполнения интеграции Azure SSIS к виртуальной сети](./join-azure-ssis-integration-runtime-virtual-network.md).

4. Подключите среду Azure-SSIS IR к локальным общим папкам в той же виртуальной сети, настроив учетные данные для доступа с проверкой подлинности Windows. Дополнительные сведения см. в статье [Подключение к данным и общим папкам с помощью проверки подлинности Windows](ssis-azure-connect-with-windows-auth.md).

5. Измените локальные пути к файлам в пакетах на UNC-пути, указывающие на локальные общие папки. Например, измените `C:\abc.txt` на `\\<on-prem-server-name>\<share-name>\abc.txt`.

## <a name="use-azure-file-shares"></a>Использование общих папок Azure

Чтобы применять службу **Файлы Azure** при переносе пакетов, использующих локальные файловые системы, в службы SSIS в Azure, выполните указанные ниже действия.

1. Перенесите файлы из локальных файловых систем в службы файлов Azure. Дополнительные сведения см. на странице [Файлы Azure](https://azure.microsoft.com/services/storage/files/).

2. Подключите среду Azure-SSIS IR к службе файлов Azure, настроив учетные данные для доступа с проверкой подлинности Windows. Дополнительные сведения см. в статье [Подключение к данным и общим папкам с помощью проверки подлинности Windows](ssis-azure-connect-with-windows-auth.md).

3. Измените локальные пути к файлам в пакетах на UNC-пути, указывающие на службу файлов Azure. Например, измените `C:\abc.txt` на `\\<storage-account-name>.file.core.windows.net\<share-name>\abc.txt`.

## <a name="next-steps"></a>Дальнейшие действия

- Развертывание пакетов. Дополнительные сведения см. в статье [Развертывание проекта служб SSIS с помощью SQL Server Management Studio (SSMS)](/sql/integration-services/ssis-quickstart-deploy-ssms).
- Запуск пакетов. Дополнительные сведения см. в статье [Выполнение пакета служб SSIS с помощью SQL Server Management Studio (SSMS)](/sql/integration-services/ssis-quickstart-run-ssms).
- Планирование выполнения пакетов. Дополнительные сведения см. [в разделе Планирование пакетов служб SSIS в Azure](/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).