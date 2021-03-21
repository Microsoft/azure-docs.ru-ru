---
title: Отрисовка оболочки
description: Объясняется, как использовать результат визуализации оболочки.
author: jumeder
ms.author: jumeder
ms.date: 10/23/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 7af95cba807cea340438a7de30f096758d0369ad
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594169"
---
# <a name="shell-rendering"></a>Отрисовка оболочки

Состояние оболочки [компонента переопределения иерархического состояния](../../overview/features/override-hierarchical-state.md) является эффектом прозрачности. В отличие от [просмотра](../../overview/features/override-hierarchical-state.md) визуализации, видимым является только самый передний слой объектов, как и для непрозрачной визуализации. Кроме того, нормальный внешний вид объектов можно изменить при подготовке к просмотру в виде оболочек. Этот эффект предназначен для случаев, когда пользователь должен визуально отойти от несущественных частей, сохраняя при этом пространственное распознавание для всей сцены.

Можно настроить внешний вид объектов, визуализированных оболочкой, с помощью `ShellRenderingSettings` глобального состояния. Все объекты, использующие рендеринг оболочки, будут использовать один и тот же параметр. Для параметров объекта нет.

## <a name="shellrenderingsettings-parameters"></a>Параметры Шеллрендерингсеттингс

Класс `ShellRenderingSettings` содержит параметры, связанные со свойствами отрисовки глобальной оболочки:

| Параметр      | Type    | Описание                                             |
|----------------|---------|---------------------------------------------------------|
| `Desaturation` | FLOAT   | Степень насыщенности, применяемая к обычному окончательному цвету объекта в диапазоне 0 (без раснасыщения) до 1 (полное раснасыщение) |
| `Opacity`      | FLOAT   | Непрозрачность объектов, визуализированных оболочкой, в диапазоне 0 (невидимый) до 1 (полностью непрозрачный) |

См. также в следующей таблице приведены примеры эффектов "Параметры" при применении к целой сцене:

|                | 0 | 0,25 | 0,5 | 0,75 | 1.0 | 
|----------------|:-:|:----:|:---:|:----:|:---:|
| **Насыщению** | ![Раснасыщение — 0,0](./media/shell-desaturation-00.png) | ![Раснасыщение — 0,25](./media/shell-desaturation-025.png) | ![Раснасыщение — 0,5](./media/shell-desaturation-05.png) | ![Раснасыщение — 0,75](./media/shell-desaturation-075.png) | ![Раснасыщение — 1,0](./media/shell-desaturation-10.png) |
| **Непрозрачность**      | ![Непрозрачность — 0,0](./media/shell-opacity-00.png) | ![Непрозрачность — 0,25](./media/shell-opacity-025.png) | ![Непрозрачность — 0,5](./media/shell-opacity-05.png) | ![Непрозрачность — 0,75](./media/shell-opacity-075.png) | ![Непрозрачность — 1,0](./media/shell-opacity-10.png) |

Эффекты оболочки применяются к окончательному непрозрачному цвету, который будет подготовлен к просмотру в противном случае. Это включает [Переопределение иерархического состояния оттенок](../../overview/features/override-hierarchical-state.md).

## <a name="example"></a>Пример

В следующем коде показан пример использования `ShellRenderingSettings` состояния через API:

```cs
void SetShellSettings(RenderingSession session)
{
    ShellRenderingSettings shellRenderingSettings = session.Connection.ShellRenderingSettings;
    shellRenderingSettings.Desaturation = 0.5f;
    shellRenderingSettings.Opacity = 0.1f;
}
```

```cpp
void SetShellSettings(ApiHandle<RenderingSession> session)
{
    ApiHandle<ShellRenderingSettings> shellRenderingSettings = session->Connection()->GetShellRenderingSettings();
    shellRenderingSettings->SetDesaturation(0.5f);
    shellRenderingSettings->SetOpacity(0.1f);
}
```

## <a name="performance"></a>Производительность

Функция отрисовки оболочки содержит небольшие постоянные издержки при сравнении со стандартной непрозрачной отрисовкой. Это происходит значительно быстрее, чем использование прозрачных материалов в объектах или Просмотр [с помощью](../../overview/features/override-hierarchical-state.md) визуализации. Производительность может снизиться более строго, если только части сцены переключены на отрисовку оболочки. Это снижение может возникать из-за дополнительно обнаруженных объектов, нуждающихся в отрисовке. В этом случае производительность ведет себя аналогично функции « [Вырезать плоскости](../../overview/features/cut-planes.md) ».

## <a name="next-steps"></a>Дальнейшие действия

* [Компонент переопределения иерархического состояния](../../overview/features/override-hierarchical-state.md)