---
title: Справочник. Известные проблемы и устранение неполадок
titleSuffix: Azure Data Science Virtual  Machine
description: Здесь можно ознакомиться с перечнем известных проблем, со способами обхода и устранения неполадок на виртуальных машинах Azure для обработки и анализа данных
services: machine-learning
ms.service: data-science-vm
author: gvashishtha
ms.author: gopalv
ms.topic: reference
ms.date: 10/10/2019
ms.openlocfilehash: bcd75347f91109967ac48427ca0b8855c11b7d9b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "101656862"
---
# <a name="known-issues-and-troubleshooting-the-azure-data-science-virtual-machine"></a>Известные проблемы и устранение неполадок на виртуальных машинах Azure для обработки и анализа данных

Эта статья поможет вам находить и устранять ошибки или сбои, которые могут возникать при использовании виртуальных машин Azure для обработки и анализа данных.

## <a name="python-package-installation-issues"></a>Проблемы с установкой пакета Python

### <a name="installing-packages-with-pip-breaks-dependencies-on-linux"></a>Установка пакетов с помощью "pip" нарушает зависимости в ОС Linux

При установке пакетов используйте `sudo pip install` вместо `pip install`.

## <a name="disk-encryption-issues"></a>Проблемы с шифрованием дисков

### <a name="disk-encryption-fails-on-the-ubuntu-dsvm"></a>Сбой шифрования диска в Ubuntu DSVM

Шифрование дисков Azure (ADE) в настоящее время не поддерживается в Ubuntu DSVM. В качестве обходного решения этой проблемы рекомендуется настроить [Шифрование на стороне сервера для управляемых дисков Azure](../../virtual-machines/disk-encryption.md).

## <a name="tool-appears-disabled"></a>Инструмент отображается как отключенный

### <a name="hyper-v-does-not-work-on-the-windows-dsvm"></a>Hyper-V не работает на Windows DSVM

То, что Hyper-V изначально не работает на платформе Windows, — это нормально. Для повышения производительности некоторые службы были отключены. Чтобы включить Hyper-V:

1. Откройте панель поиска на виртуальной машине Windows для обработки и анализа данных
1. Введите в поисковое поле: "Services".
1. Задайте для всех служб Hyper-V значение "Вручную"
1. Задайте для параметра "Управление виртуальными машинами Hyper-V" значение "Автоматически".

Итоговое окно должно выглядеть следующим образом:

   ![Включение Hyper-V](./media/workaround/hyperv-enable-dsvm.png)