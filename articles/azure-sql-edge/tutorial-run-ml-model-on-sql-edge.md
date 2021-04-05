---
title: Развертывание модели ML в SQL Azure для пограничных вычислений с помощью ONNX
description: В части 3 (из 3) руководства по SQL Azure для пограничных вычислений, посвященного прогнозированию примеси железной руды, запустите модели машинного обучения ONNX на SQL Azure для пограничных вычислений.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 9e5bb037b88b7c370e31d05c2d20fc6f558a8b39
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "93422201"
---
# <a name="deploy-ml-model-on-azure-sql-edge-using-onnx"></a>Развертывание модели ML в SQL Azure для пограничных вычислений с помощью ONNX 

В части 3 ( из 3) этого руководства по прогнозированию примеси железной руды в SQL Azure для пограничных вычислений вы:

1. Используете Azure Data Studio для подключения к Базе данных SQL в экземпляре SQL Azure для пограничных вычислений.
2. Спрогнозируете примесь железной руды с помощью ONNX в SQL Azure для пограничных вычислений.

## <a name="key-components"></a>Основные компоненты

1. В решении используется значение по умолчанию в 500 миллисекунд между всеми сообщениями, отправляемыми в концентратор для пограничных вычислений. Это можно изменить в файле **Program.cs** 
   ```json
   TimeSpan messageDelay = configuration.GetValue("MessageDelay", TimeSpan.FromMilliseconds(500));
   ```
2. В решении создано сообщение с указанными ниже атрибутами. Добавьте или удалите атрибуты в соответствии с требованиями. 
```json
{
    timestamp 
    cur_Iron_Feed
    cur_Silica_Feed 
    cur_Starch_Flow 
    cur_Amina_Flow 
    cur_Ore_Pulp_pH
    cur_Flotation_Column_01_Air_Flow
    cur_Flotation_Column_02_Air_Flow
    cur_Flotation_Column_03_Air_Flow
    cur_Flotation_Column_04_Air_Flow
    cur_Flotation_Column_01_Level
    cur_Flotation_Column_02_Level
    cur_Flotation_Column_03_Level
    cur_Flotation_Column_04_Level
    cur_Iron_Concentrate
}
```

## <a name="connect-to-the-sql-database-in-the-azure-sql-edge-instance-to-train-deploy-and-test-the-ml-model"></a>Установите подключение к базе данных SQL в экземпляре SQL Azure для пограничных вычислений, чтобы обучить, развернуть и протестировать модель ML

1. Откройте Azure Data Studio.

2. На вкладке **Приветствие** запустите новое подключение со следующими сведениями:

   |_Поле_|_Значение_|
   |-------|-------|
   |Тип соединений| Microsoft SQL Server|
   |Сервер|Общедоступный IP-адрес, упомянутый в виртуальной машине, созданной для этой демо-версии|
   |Имя пользователя|sa|
   |Пароль|Надежный пароль, который использовался при создании экземпляра SQL Azure для пограничных вычислений.|
   |База данных|По умолчанию|
   |Группа серверов|По умолчанию|
   |Имя (необязательно)|Укажите необязательное имя|

3. Добавьте новый отчет, щелкнув **Подключить**

4. В разделе **Файл** откройте файл **/DeploymentScripts/MiningProcess_ONNX.jpynb** из папки, в которой были клонированы файлы проекта на компьютере.

5. Установите ядро для Python 3.


## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании моделей ONNX в SQL Azure для пограничных вычислений см. в статье [Машинное обучение и искусственный интеллект с ONNX в SQL Azure для пограничных вычислений](onnx-overview.md).