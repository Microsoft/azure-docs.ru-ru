---
title: Очистка изолированного кластера
description: Из этого учебника вы узнаете, как удалить ресурсы AWS и Azure в изолированном кластере Service Fabric.
ms.topic: tutorial
ms.date: 07/22/2019
ms.custom: mvc
ms.openlocfilehash: 0d46e9068a311594f779411c3ccee2b408febb3f
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91842892"
---
# <a name="tutorial-clean-up-your-standalone-cluster"></a>Руководство по очистке изолированного кластера

Изолированные кластеры Service Fabric позволяют выбрать собственную среду для размещения Service Fabric. Из этой серии учебников вы узнаете, как создать изолированный кластер, размещенный на AWS или Azure, и развернуть в нем приложение.

Это руководство представляет собой четвертую часть цикла. В этой части учебника показано, как удалить ресурсы AWS или Azure, созданные для размещения кластера Service Fabric.

Из этой статьи вы узнаете следующее.

> [!div class="checklist"]
> * Удаление кластера Service Fabric
> * Удаление ресурсов AWS или Azure

## <a name="remove-a-service-fabric-cluster"></a>Удаление кластера Service Fabric

1. Подключитесь по протоколу RDP к виртуальной машине, использованной для установки Service Fabric.
2. Откройте PowerShell.
3. Перейдите в каталог извлеченной папки из второго руководства.
4. Чтобы удалить кластер Service Fabric, выполните следующую команду:

  ```powershell
  .\RemoveServiceFabricCluster.ps1 -ClusterConfigFilePath .\ClusterConfig.Unsecure.MultiMachine.json
  ```

5. При появлении запроса введите `Y`. Если операция выполнена успешно, выходные данные будут выглядеть так, как показано ниже (будут указаны ваши IP-адреса).

  ```powershell
  Best Practices Analyzer completed successfully.
  Removing configuration from machine 172.31.21.141
  Removing configuration from machine 172.31.27.1
  Removing configuration from machine 172.31.20.163
  Configuration removed from machine 172.31.21.141
  Configuration removed from machine 172.31.27.1
  Configuration removed from machine 172.31.20.163
  The cluster is successfully removed.
  ```

## <a name="delete-aws-resources"></a>Удаление ресурсов AWS

1. Войдите в учетную запись AWS.
2. Перейдите к консоли EC2.
3. Выберите три узла, созданные в первой части этой серии руководств.
4. Последовательно выберите **Действия** > **Состояние экземпляра** > **Завершить**.

## <a name="delete-azure-resources"></a>Удаление ресурсов Azure

1. Войдите на портал Azure.
2. Перейдите в раздел **Виртуальные машины**.
3. Установите флажки для трех узлов, созданных в первой части этой серии учебников.
4. Выберите **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого учебника вы узнали, как удалять ресурсы, созданные на предыдущих шагах.

> [!div class="checklist"]
> * Очистка ресурсов

> [!div class="nextstepaction"]
> [В начало](service-fabric-tutorial-standalone-create-infrastructure.md)
