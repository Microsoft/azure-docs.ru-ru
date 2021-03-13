---
title: включить файл
description: включить файл
services: machine-learning
author: sdgilley
ms.service: machine-learning
ms.author: sgilley
manager: cgronlund
ms.custom: include file
ms.topic: include
ms.date: 03/05/2020
ms.openlocfilehash: 0d7622217d192a100fd809c32c9e85970cf61fd7
ms.sourcegitcommit: 15d27661c1c03bf84d3974a675c7bd11a0e086e6
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/09/2021
ms.locfileid: "102511077"
---
1. Чтобы установить пакет SDK службы "Машинное обучение Azure" для Python, выполните инструкции из [этой статьи](/python/api/overview/azure/ml/install).

1. Создайте [рабочую область Машинного обучения Azure](../articles/machine-learning/how-to-manage-workspace.md).

1. Создайте [файл конфигурации](../articles/machine-learning/how-to-configure-environment.md#workspace) (**aml_config/config.json**).

1. Клонируйте [репозиторий GitHub](https://aka.ms/aml-notebooks).

    ```bash
    git clone https://github.com/Azure/MachineLearningNotebooks.git
    ```

1. Запустите сервер записной книжки из клонированного каталога.

    ```bash
    jupyter notebook
    ```