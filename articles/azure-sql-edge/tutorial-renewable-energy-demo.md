---
title: Развертывание Azure SQL Edge на генераторах ветроэлектростанции Contoso
description: В этом руководстве вы будете использовать Azure SQL Edge для обнаружения попутных потоков на ветроустановках ветроэлектростанции Contoso.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 12/18/2020
ms.openlocfilehash: e966d756744210dc8f6739b96501dd91f0d8d1b5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "97723479"
---
# <a name="using-azure-sql-edge-to-build-smarter-renewable-resources"></a>Использование Azure SQL Edge для создания интеллектуальных ресурсов возобновляемых ресурсов

Этот демонстрационный ролик Azure SQL Edge создан для ветроэлектростанции Contoso Renewable Energy, на которой используется SQL DB Edge для обработки данных генератора. 

Из этого демонстрационного видео вы узнаете, как обработать предупреждение, созданное из-за того, что на устройстве обнаружена атмосферная турбулентность. Вы научитесь обучать модель и развернете ее на SQL DB Edge, которая будет корректировать обнаруженный попутный поток и, в конечном итоге, оптимизировать выходную мощность.

Azure SQL Edge — демонстрационное видео для Contoso Renewable Energy на канале Channel 9:
> [!VIDEO https://channel9.msdn.com/Shows/Data-Exposed/Azure-SQL-Edge-Demo-Renewable-Energy/player]

## <a name="setting-up-the-demo-on-your-local-computer"></a>Настройка демонстрации на локальном компьютере
Для копирования всех файлов из демонстрации на локальный компьютер будет использоваться Git. 

1. Установите Git [отсюда](https://git-scm.com/download).
2. Откройте командную строку и перейдите в папку, в которую должен быть загружен репозиторий. 
3. Выполните команду https://github.com/microsoft/sql-server-samples.git.
4. Перейдите в папку **sql-server-samples\samples\demos\azure-sql-edge-demos\Wind Turbine Demo** в месте клонирования репозитория.
5. Следуйте инструкциям в файле README.md, чтобы настроить демонстрационную среду и запустить демонстрацию.