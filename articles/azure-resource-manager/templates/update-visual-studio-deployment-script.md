---
title: Обновление скрипта развертывания шаблона Visual Studio для использования команды AZ PowerShell
description: Обновление скрипта развертывания шаблона Visual Studio с AzureRM на AZ PowerShell
author: cweining
ms.topic: conceptual
ms.date: 01/31/2020
ms.author: cweining
ms.openlocfilehash: 357e0289f3237ed32b0801280316225ba5530282
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "76963875"
---
# <a name="update-visual-studio-template-deployment-script-to-use-az-powershell-module"></a>Обновите скрипт развертывания шаблона Visual Studio, чтобы использовать команду AZ PowerShell Module

Visual Studio 16,4 поддерживает использование модуля AZ PowerShell в скрипте развертывания шаблона. Однако Visual Studio не устанавливает этот модуль автоматически. Чтобы использовать модуль AZ, необходимо выполнить четыре шага:

1. [Удаление модуля AzureRM](/powershell/azure/uninstall-az-ps#uninstall-the-azurerm-module)
1. [Установка AZ Module](/powershell/azure/install-az-ps)
1. Обновление Visual Studio до версии 16.4
1. (Обновите скрипт развертывания в проекте.)

## <a name="update-visual-studio-to-164"></a>Обновление Visual Studio до версии 16.4

Обновите установку Visual Studio до версии 16,4 или более поздней. Во время обновления убедитесь, что компонент Azure PowerShell не установлен. Так как вы установили модуль AZ через коллекцию, вам не нужно переустанавливать модуль AzureRM.

Если вы уже обновили до 16,4 и компонент Azure PowerShell был установлен, его можно удалить, запустив Visual Studio Installer. Не выбирайте компонент Azure PowerShell в рабочей нагрузке Azure или на странице отдельные компоненты.

## <a name="update-the-deployment-script-in-your-project"></a>Обновление скрипта развертывания в проекте

Замените все вхождения строки "AzureRm" на "Az" в скрипте развертывания. Чтобы увидеть изменения, обратитесь к исправлениям в этом [gist](https://gist.github.com/cweining/d2da2479418ea403499c4306dcf4f619). Дополнительные сведения об обновлении скриптов для модуля AZ см. в разделе [миграция Azure PowerShell из AzureRM в AZ](/powershell/azure/migrate-from-azurerm-to-az).

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения об использовании проекта Visual Studio см. в статье [Создание и развертывание проектов групп ресурсов Azure с помощью Visual Studio](create-visual-studio-deployment-project.md).
