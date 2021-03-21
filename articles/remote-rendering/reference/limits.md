---
title: Ограничения
description: Ограничения для кода функций пакета SDK
author: erscorms
ms.author: erscor
ms.date: 02/11/2020
ms.topic: reference
ms.openlocfilehash: f62a07ef7109c669c6abc90d4e1b4a2ee83ece23
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100530252"
---
# <a name="limitations"></a>Ограничения

Ряд функций имеет ограничения по размеру, количеству или другим параметрам.

## <a name="azure-frontend"></a>Интерфейсный сервер Azure

К интерфейсному API (C++ и C#) применяются следующие ограничения.
* Всего экземпляров [ремотерендерингклиент](/dotnet/api/microsoft.azure.remoterendering.remoterenderingclient) на процесс: 16.
* Всего экземпляров [рендерингсессион](/dotnet/api/microsoft.azure.remoterendering.renderingsession) на [ремотерендерингклиент](/dotnet/api/microsoft.azure.remoterendering.remoterenderingclient): 16.

## <a name="objects"></a>Объекты

* Общее количество допустимых объектов одного типа ([сущность](../concepts/entities.md), [кутпланекомпонент](../overview/features/cut-planes.md)и т. д.): 16 777 215.
* Общее допустимое количество активных сечений: 8.

## <a name="geometry"></a>Геометрия

* **Анимация:** Анимация ограничивается анимацией отдельных преобразований [игровых объектов](../concepts/entities.md). Анимации скелетообразных с обложками или анимацией вершин не поддерживаются. Дорожки анимации из исходного файла ресурса не сохраняются. Вместо этого анимация преобразования объектов должна управляться кодом клиента.
* **Пользовательские шейдеры:** Создание пользовательских шейдеров не поддерживается. Можно использовать только встроенные [цветовые материалы](../overview/features/color-materials.md) или [материалы PBR](../overview/features/pbr-materials.md) .
* **Максимальное число различных материалов** в ресурсе: 65 535. Дополнительные сведения об автоматическом уменьшении количества материалов см. в разделе «Удаление [дубликатов материалов](../how-tos/conversion/configure-model-conversion.md#material-de-duplication) ».
* **Максимальная размерность одной текстуры**: 16 384 x 16 384. Размер больших исходных текстур будет уменьшен в процессе преобразования.

### <a name="overall-number-of-polygons"></a>Общее число многоугольников

Допустимое количество многоугольников для всех загруженных моделей зависит от размера виртуальной машины, обрабатывающей [REST API управления сеансами](../how-tos/session-rest-api.md).

| Размер сервера | Максимальное число многоугольников |
|:--------|:------------------|
|standard| 20 млн |
|Премиум| без ограничений |

Подробные сведения об этом ограничении см. в главе « [Размер сервера](../reference/vm-sizes.md) ».

## <a name="platform-limitations"></a>Ограничения платформы

**Windows 10 Desktop**

* Win32/x64 является единственной поддерживаемой платформой Win32. Win32/x86 не поддерживаются.

**HoloLens 2**

* Визуализация [данных с камеры](/windows/mixed-reality/mixed-reality-capture-for-developers#render-from-the-pv-camera-opt-in) не поддерживается.