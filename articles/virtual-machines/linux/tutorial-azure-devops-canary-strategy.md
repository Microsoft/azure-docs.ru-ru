---
title: Руководство. Настройка развертывания canary для Виртуальных машин Linux
description: Из этого руководства вы узнаете, как настроить конвейер непрерывного развертывания (CD). Этот конвейер обновляет группу виртуальных машин Linux в Azure с использованием стратегии канареечного развертывания.
author: moala
tags: azure-devops-pipelines
ms.assetid: ''
ms.service: virtual-machines
ms.collection: linux
ms.topic: tutorial
ms.tgt_pltfrm: azure-pipelines
ms.workload: infrastructure
ms.date: 4/10/2020
ms.author: moala
ms.custom: devops
ms.openlocfilehash: bbfe6571cf075b2ce4930eea91bfd1e239470c5a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "102552513"
---
# <a name="tutorial---configure-the-canary-deployment-strategy-for-azure-linux-virtual-machines"></a>Руководство. Настройка стратегии канареечного развертывания для виртуальных машин Linux в Azure

## <a name="infrastructure-as-a-service-iaas---configure-cicd"></a>Инфраструктура как услуга (IaaS). Настройка CI/CD

Azure Pipelines предоставляет полнофункциональный набор инструментов для автоматизации CI/CD при развертывании на виртуальных машинах. Вы можете настроить на портале Azure конвейер непрерывной поставки для виртуальной машины Azure.

В этой статье описывается, как настроить конвейер CI/CD, который использует стратегию канареечного развертывания для нескольких компьютеров. Также портал Azure поддерживает другие стратегии, например [последовательного](./tutorial-devops-azure-pipelines-classic.md) и [сине-зеленого](./tutorial-azure-devops-blue-green-strategy.md) развертывания.

### <a name="configure-cicd-on-virtual-machines"></a>Настройка CI/CD для виртуальных машин

Вы можете добавить целевые виртуальные машины в [группу развертывания](/azure/devops/pipelines/release/deployment-groups). Это позволит организовать для них многокомпьютерное обновление. После развертывания на компьютерах просмотрите **журнал развертывания** в группе развертывания. Это представление позволяет выполнять трассировку действий от виртуальной машины до конвейера и далее до фиксации.

### <a name="canary-deployments"></a>Канареечные развертывания

Канареечное развертывание снижает риск, так как развертывание изменений выполняется медленно для небольшого подмножества пользователей. Когда вы будете уверены в новой версии, ее можно выпустить на другие серверы в инфраструктуре, предоставив большему количеству пользователей.

Используя вариант непрерывной поставки, вы можете настроить канареечное развертывание на виртуальных машинах с помощью портала Azure. Ниже представлены пошаговые инструкции.

1. Войдите на портал Azure и перейдите к нужной виртуальной машине.
1. На крайней левой панели параметров виртуальной машины выберите **Непрерывная поставка**. Затем выберите **Настроить**.

   ![Панель непрерывной поставки с кнопкой настройки](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure.png)

1. На панели конфигурации щелкните **Организация Azure DevOps**, чтобы выбрать существующую учетную запись или создать новую. Затем выберите проект, в котором вы хотите настроить конвейер.  

   ![Панель непрерывной поставки](media/tutorial-devops-azure-pipelines-classic/azure-devops-rolling.png)

1. Группа развертывания — это логический набор целевых компьютеров для развертывания, который представляет определенную физическую среду. Это может быть среда разработки, тестирования или пользовательского приемочного тестирования, а также рабочая среда. Вы можете создать новую группу развертывания или выбрать существующую.
1. Выберите конвейер сборки, который публикует пакет для развертывания на виртуальной машине. Опубликованный пакет должен содержать скрипт развертывания deploy.ps1 или deploy.sh в папке deployscripts, расположенной в корневой папке пакета. Конвейер запускает этот скрипт развертывания.
1. В списке **Стратегия развертывания** выберите вариант **Canary** (Канареечное).
1. Добавьте тег "canary" к тем виртуальным машинам, для которых будет применяться канареечное развертывание. Добавьте тег "prod" к тем виртуальным машинам, для которых развертывание будет выполняться после успешного завершения канареечного развертывания. Эти теги помогут правильно выбрать целевые виртуальные машины с определенной ролью.

   ![Панель непрерывной поставки с канареечным развертыванием, выбранным в качестве стратегии развертывания](media/tutorial-devops-azure-pipelines-classic/azure-devops-configure-canary.png)

1. Нажмите кнопку **ОК**, чтобы настроить конвейер непрерывной поставки для развертывания на виртуальной машине.

   ![Конвейер канареечного развертывания](media/tutorial-devops-azure-pipelines-classic/azure-devops-canary-pipeline.png)

1. Отобразятся сведения о развертывании для виртуальной машины. Вы можете щелкнуть здесь ссылку, чтобы перейти к конвейеру выпуска в Azure DevOps. На странице конвейера выпуска щелкните **Изменить**, чтобы просмотреть конфигурацию конвейера. Этот конвейер состоит из трех этапов.

   1. Первым является этап группы развертывания. Приложения развертываются только на тех виртуальных машинах, которые имеют тег canary.
   1. На втором этапе конвейер приостанавливает работу и ожидает возобновления вручную.
   1. Это продолжение этапа группы развертывания. Теперь обновление развертывается на виртуальных машинах с тегом prod.

      ![Панель "Группа развертывания" для задачи канареечного развертывания](media/tutorial-devops-azure-pipelines-classic/azure-devops-canary-task.png)

1. Прежде чем возобновить выполнение конвейера, убедитесь, что хотя бы одна виртуальная машина имеет тег prod. На третьем этапе конвейера приложения будут развернуты только на тех виртуальных машинах, которые имеют тег prod.

1. Задача выполнения скрипта развертывания по умолчанию выполняет скрипт развертывания deploy.ps1 или deploy.sh. Этот скрипт размещается в подпапке deployscripts корневой папки опубликованного пакета. Убедитесь, что выбранный конвейер сборки публикует развертывание в корневой папке пакета.

   ![Панель "Артефакты", где отображается файл deploy.sh в папке deployscripts](media/tutorial-deployment-strategy/package.png)

## <a name="other-deployment-strategies"></a>Другие стратегии развертывания
- [Настройка стратегии последовательного развертывания](./tutorial-devops-azure-pipelines-classic.md)
- [Настройка стратегии развертывания по схеме "синий-зеленый"](./tutorial-azure-devops-blue-green-strategy.md)

## <a name="azure-devops-projects"></a>Azure DevOps Projects

Вам будет легко начать работу с Azure. Функция Azure DevOps Projects позволяет запустить приложение в любой службе Azure, выполнив всего три шага:

- выбор языка приложения;
- выбор среды выполнения;
- выбор службы Azure.

[Подробнее](https://azure.microsoft.com/features/devops-projects/).

## <a name="additional-resources"></a>Дополнительные ресурсы

- [Развертывание на виртуальных машинах Azure с помощью Azure DevOps Projects](../../devops-project/azure-devops-project-vms.md)
- [Реализация непрерывного развертывания приложения в масштабируемом наборе виртуальных машин Azure](/azure/devops/pipelines/apps/cd/azure/deploy-azure-scaleset)