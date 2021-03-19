---
title: Непрерывная интеграция и развертывание для выделенного пула SQL
description: DevOps в базе данных корпоративного класса для выделенного пула SQL в Azure синапсе Analytics со встроенной поддержкой непрерывной интеграции и развертывания с помощью Azure Pipelines.
services: synapse-analytics
author: gaursa
manager: craigg
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: sql-dw
ms.date: 02/04/2020
ms.author: gaursa
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: d43039c98e991cd23a8e5c4fb866a8e3dcab6fc2
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104585644"
---
# <a name="continuous-integration-and-deployment-for-dedicated-sql-pool-in-azure-synapse-analytics"></a>Непрерывная интеграция и развертывание для выделенного пула SQL в Azure синапсе Analytics

В этом простом учебнике объясняется, как интегрировать проект базы данных SQL Server Data Tools (SSDT) с Azure DevOps и настроить непрерывную интеграцию и развертывание с помощью Azure Pipelines. Это руководство является вторым шагом в создании конвейера непрерывной интеграции и развертывания для хранения данных.

## <a name="before-you-begin"></a>Перед началом

- Изучите [учебник по интеграции системы управления версиями](sql-data-warehouse-source-control-integration.md).

- Настройка и подключение к Azure DevOps

## <a name="continuous-integration-with-visual-studio-build"></a>Непрерывная интеграция с использованием сборки Visual Studio

1. Перейдите к Azure Pipelines и создайте конвейер сборки.

      ![Новый конвейер](./media/sql-data-warehouse-continuous-integration-and-deployment/1-new-build-pipeline.png "Новый конвейер")

2. Выберите репозиторий исходного кода (Azure Repos Git) и шаблон классического приложения .NET.

      ![Установка конвейера](./media/sql-data-warehouse-continuous-integration-and-deployment/2-pipeline-setup.png "Установка конвейера")

3. Измените файл YAML, указав соответствующий пул агента. Файл YAML должен выглядеть примерно так:

      ![YAML](./media/sql-data-warehouse-continuous-integration-and-deployment/3-yaml-file.png "YAML")

На этом этапе у вас есть простая среда, в которой любой возврат в основную ветвь репозитория системы управления версиями должен автоматически активировать успешное построение проекта базы данных в Visual Studio. Убедитесь, что автоматизация работает в конце, внеся изменения в проект локальной базы данных и отметив это изменение в основной ветви.

## <a name="continuous-deployment-with-the-azure-synapse-analytics-or-database-deployment-task"></a>Непрерывное развертывание с помощью задачи развертывания Azure синапсе Analytics (или базы данных)

1. Добавьте новую задачу с помощью [задачи развертывания Базы данных SQL Azure](/azure/devops/pipelines/targets/azure-sqldb) и заполните необходимые поля, чтобы подключиться к целевому хранилищу данных. При запуске этой задачи DACPAC, созданный в результате предыдущего процесса сборки, развертывается в целевом хранилище данных. Вы также можете использовать [задачу развертывания Azure синапсе Analytics](https://marketplace.visualstudio.com/items?itemName=ms-sql-dw.SQLDWDeployment).

      ![Задача развертывания](./media/sql-data-warehouse-continuous-integration-and-deployment/4-deployment-task.png "Задача развертывания")

2. Если вы используете собственный агент, убедитесь, что для переменной среды задано использование правильного SqlPackage.exe для Azure синапсе Analytics. Путь должен выглядеть следующим образом:

      ![Переменная среды](./media/sql-data-warehouse-continuous-integration-and-deployment/5-environment-variable-preview.png "Переменная среды")

   C:\Program Files (x86)\Microsoft Visual Studio\2019\Preview\Common7\IDE\Extensions\Microsoft\SQLDB\DAC\150  

   Запустите и проверьте свой конвейер. Вы можете вносить изменения локально и возвращать изменения в системе управления версиями, в результате чего должна выполняться автоматическая сборка и развертывание.

## <a name="next-steps"></a>Дальнейшие действия

- Изучение [архитектуры выделенного пула SQL (прежнее название — хранилище данных SQL)](massively-parallel-processing-mpp-architecture.md)
- Быстрое [Создание выделенного пула SQL (прежнее название — хранилище данных SQL)](create-data-warehouse-portal.md)
- [Отправка примера данных](./load-data-from-azure-blob-storage-using-copy.md)
- Просмотрите [видео](sql-data-warehouse-videos.md).