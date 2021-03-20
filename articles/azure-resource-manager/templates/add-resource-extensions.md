---
title: Настройка после развертывания с помощью расширений
description: Узнайте, как использовать расширения шаблона (шаблоны ARM) Azure Resource Manager для предоставления конфигураций после развертывания.
author: mumian
ms.topic: conceptual
ms.date: 12/14/2018
ms.author: jgao
ms.openlocfilehash: 23d5a9aaa546542218a6f839ab415184ff309928
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97934328"
---
# <a name="provide-post-deployment-configurations-by-using-extensions"></a>Предоставление конфигураций после развертывания с помощью расширений

Расширения шаблонов Azure Resource Manager (шаблоны ARM) — это небольшие приложения, которые предоставляют задачи настройки и автоматизации, выполняемые после развертывания, в ресурсах Azure. Самые популярные — расширения виртуальных машин. Ознакомьтесь со статьей [Обзор расширений и компонентов виртуальной машины под управлением Windows](../../virtual-machines/extensions/features-windows.md) или [Обзор расширений и компонентов виртуальных машин под управлением Linux](../../virtual-machines/extensions/features-linux.md).

## <a name="extensions"></a>Расширения

Ниже приведены имеющиеся расширения:

- [Microsoft. COMPUTE/virtualMachines/Extensions](/azure/templates/microsoft.compute/2018-10-01/virtualmachines/extensions)
- [Microsoft.Compute virtualMachineScaleSets/extensions](/azure/templates/microsoft.compute/2018-10-01/virtualmachinescalesets/extensions)
- [Microsoft.HDInsight clusters/extensions](/azure/templates/microsoft.hdinsight/2018-06-01-preview/clusters)
- [Microsoft.Sql servers/databases/extensions](/azure/templates/microsoft.sql/2014-04-01/servers/databases/extensions)
- [Microsoft.Web/sites/siteextensions](/azure/templates/microsoft.web/2016-08-01/sites/siteextensions)

Чтобы узнать доступные расширения, перейдите по [ссылке на шаблон](/azure/templates/). В **фильтре по названию** введите **расширение**.

Подробнее об использовании этих расширений см. в следующих статьях:

- [Руководство. развертывание расширений виртуальной машины с помощью шаблонов ARM](template-tutorial-deploy-vm-extensions.md).
- [Руководство по Импорт BACPAC-файлов SQL с помощью шаблонов ARM](template-tutorial-deploy-sql-extensions-bacpac.md)

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Руководство по развертыванию расширений виртуальных машин с помощью шаблонов Resource Manager](template-tutorial-deploy-vm-extensions.md)
