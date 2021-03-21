---
title: Подготовка технических ресурсов контейнера Azure
description: Статья содержит технические ресурсы и рекомендации, которые помогут настроить предложение по контейнерам в Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: keferna
ms.author: keferna
ms.date: 11/30/2020
ms.openlocfilehash: 014bcd6fc519c267cdf17e9e98b850425c25ead6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96459321"
---
# <a name="prepare-your-azure-container-technical-assets"></a>Подготовка технических ресурсов контейнера Azure

Эта статья содержит технические ресурсы и рекомендации, которые помогут настроить предложение по контейнерам в Azure Marketplace.

## <a name="before-you-begin"></a>Перед началом

Ознакомьтесь с [документацией по службе "Экземпляры контейнеров Azure"](../container-instances/index.yml). Там вы найдете краткие инструкции, руководства и примеры для начала работы.

## <a name="fundamental-technical-knowledge"></a>Основные технические знания

Проектирование, сборка и тестирование используемых ресурсов занимают время и требуют технических знаний не только платформы Azure, но и технологий, используемых для создания предложения.

Ваша команда разработчиков должна не только хорошо разбираться во всем, что касается вашего решения, но и обладать знаниями о технологиях Майкрософт. В частности, вашим специалистам потребуется следующее:

- базовое представление о [службах Azure](https://azure.microsoft.com/services/);
- умение [разработать приложения Azure](https://azure.microsoft.com/solutions/architecture/);
- практические знания о [Виртуальных машинах Azure](https://azure.microsoft.com/services/virtual-machines/), [Службе хранилища Azure](https://azure.microsoft.com/services/?filter=storage) и [Сетях Azure](https://azure.microsoft.com/services/?filter=networking);
- опыт работы с [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/);
- практическое умение использовать [JSON](https://www.json.org/).

## <a name="suggested-tools"></a>Рекомендуемые инструменты

Для управления образами контейнеров рекомендуем использовать следующие среды сценариев:

- [Azure PowerShell](/powershell/azure/);
- [Azure CLI](/cli/azure/).

Кроме того, рекомендуем добавить в среду разработки следующие инструменты:

- [Обозреватель службы хранилища Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows)
- [Visual Studio Code](https://code.visualstudio.com/)
  - Расширение: [Средства Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)
  - Расширение: [Beautify](https://marketplace.visualstudio.com/items?itemName=HookyQR.beautify)
  - Расширение: [Prettify JSON](https://marketplace.visualstudio.com/items?itemName=mohsen1.prettify-json)

Просмотрите доступные средства на странице [средств для разработчиков Azure](https://azure.microsoft.com/). Если вы используете Visual Studio, просмотрите доступные средства в [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

## <a name="create-the-container-image"></a>Создание образа контейнера

Вы не можете развернуть образ в службе "экземпляры контейнеров Azure" из локального реестра.

- Если у вас уже есть рабочий контейнер в локальном реестре, создайте реестр Azure и отправьте образ контейнера в реестр контейнеров Azure. Дополнительные сведения см. в разделе [учебник. Создание и развертывание образов контейнеров в облаке с помощью задач реестра контейнеров Azure](../container-registry/container-registry-tutorial-quick-task.md).

- Если у вас еще нет образа контейнера и вам нужно контейнеризовать существующее приложение или создать новое приложение на основе контейнера, клонировать исходный код приложения из GitHub, создать образ контейнера из источника приложения и протестировать образ в локальной среде DOCKER. Дополнительные сведения см. в разделе [учебник. Создание образа контейнера для развертывания в службе "экземпляры контейнеров Azure"](../container-instances/container-instances-tutorial-prepare-app.md).

## <a name="next-steps"></a>Дальнейшие действия

- [Создание предложения контейнера](create-azure-container-offer.md)