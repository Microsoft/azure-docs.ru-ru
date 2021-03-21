---
title: Установка пространственных привязок Azure для приложения C++/WinRT HoloLens
description: Настройка проекта/WinRT HoloLens C++ для использования пространственных привязок Azure
author: msftradford
manager: MehranAzimi-msft
services: azure-spatial-anchors
ms.author: parkerra
ms.date: 11/20/2020
ms.topic: how-to
ms.service: azure-spatial-anchors
ms.openlocfilehash: 69134ef87d90fe69de2d9e4e9222e90f65edc785
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "95506999"
---
# <a name="configuring-azure-spatial-anchors-in-a-cwinrt-hololens-project"></a>Настройка пространственных привязок Azure в проекте C++/WinRT HoloLens

## <a name="requirements"></a>Требования

* [Visual Studio 2019](https://www.visualstudio.com/downloads/) устанавливается вместе с рабочей нагрузкой **универсальная платформа Windows Development** и **Windows 10 SDK (10.0.18362.0 или более поздней версии)** .

## <a name="configuring-a-project"></a>Настройка проекта

Пространственные привязки Azure для HoloLens и C++/WinRT распределяются с помощью пакета NuGet [Microsoft. Azure. спатиаланчорс. WinRT](https://www.nuget.org/packages/Microsoft.Azure.SpatialAnchors.WinRT/) .

Следуйте приведенным [здесь](/nuget/consume-packages/install-use-packages-visual-studio) инструкциям, чтобы использовать диспетчер пакетов NuGet Visual Studio для установки пакета NuGet [Microsoft. Azure. спатиаланчорс. WinRT](https://www.nuget.org/packages/Microsoft.Azure.SpatialAnchors.WinRT/) в проект.

## <a name="next-steps"></a>Дальнейшие действия

> [!div class="nextstepaction"]
> [Как создавать и размещать привязки в C++/WinRT](./create-locate-anchors-cpp-winrt.md)