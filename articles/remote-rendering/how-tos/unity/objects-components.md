---
title: Игровые объекты и компоненты Unity
description: Описывает методы Unity, которые используются для работы с сущностями и компонентами удаленной отрисовки.
author: jakrams
ms.author: jakras
ms.date: 02/28/2020
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 15822c357db63db81e6c1efda2467279a98d7c34
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99594152"
---
# <a name="interact-with-unity-game-objects-and-components"></a>Взаимодействие с объектами и компонентами игр Unity

Служба удаленной подготовки Azure (ARR) оптимизирована для большого числа объектов (см. раздел [ограничения](../../reference/limits.md)). Хотя на узле можно управлять большими и сложными иерархиями, их репликация в Unity на устройствах с низким энергопотреблением неосуществима.

Поэтому при загрузке модели на узле Удаленная визуализация Azure отражает сведения о структуре модели на клиентском устройстве (что приведет к сетевому трафику), но не реплицирует объекты и компоненты в Unity. Вместо этого он предполагает, что нужно запросить необходимые объекты и компоненты игры Unity вручную, чтобы вы могли ограничить издержки на фактически требуемые. Таким образом вы получите больший контроль над производительностью на стороне клиента.

Следовательно, интеграция средств удаленной подготовки Azure с Unity обеспечивает дополнительные функции для репликации структуры удаленной подготовки к просмотру по требованию.

## <a name="load-a-model-in-unity"></a>Загрузка модели в Unity

При загрузке модели вы получаете ссылку на корневой объект загруженной модели. Эта ссылка не является объектом игры Unity, но ее можно преобразовать в одну с помощью метода расширения `Entity.GetOrCreateGameObject()` . Эта функция принимает аргумент типа `UnityCreationMode` . При передаче `CreateUnityComponents` вновь созданный объект игры Unity будет дополнительно заполнен компонентами прокси-сервера для всех компонентов удаленной отрисовки, существующих на узле. Рекомендуется, однако, предпочесть `DoNotCreateUnityComponents` , чтобы избежать минимальных затрат.

### <a name="load-model-with-unity-coroutines"></a>Загрузка модели с соподпрограммами Unity

```cs
IEnumerator LoadModelWithCoroutine(RenderingSession session)
{
    float currentProgress = 0.0f;
    var task = session.Connection.LoadModelFromSasAsync(new LoadModelFromSasOptions("builtin://Engine"),
        (float progress) =>
        {
            currentProgress = progress;
        });

    while (!task.IsCompleted && !task.IsFaulted)
    {
        int percentage = (int)(currentProgress * 100.0f);
        yield return null;
    }

    if (!task.IsFaulted)
    {
        var gameObject = task.Result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
    }
}
```

### <a name="load-model-with-await-pattern"></a>Загрузить модель с шаблоном ожидания

```cs
async void LoadModelWithAwait(RenderingSession session)
{
    var result = await session.Connection.LoadModelFromSasAsync(new LoadModelFromSasOptions("builtin://Engine"), null);
    var gameObject = result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
}
```

В приведенных выше примерах кода использовался путь загрузки модели через SAS, так как Встроенная модель загружена. Обращение к модели через контейнеры больших двоичных объектов (с помощью `LoadModelAsync` и `LoadModelOptions` ) работает полностью.

## <a name="remoteentitysyncobject"></a>ремотинтитисинкобжект

Создание игрового объекта Unity неявно добавляет `RemoteEntitySyncObject` компонент к игровому объекту. Этот компонент используется для синхронизации преобразования сущностей с сервером. По умолчанию `RemoteEntitySyncObject` пользователю необходимо явно вызвать метод `SyncToRemote()` , чтобы синхронизировать локальное состояние Unity с сервером. При включении `SyncEveryFrame` объект будет автоматически синхронизирован.

Объекты с помощью `RemoteEntitySyncObject` могут иметь экземпляры удаленных дочерних элементов и отображаться в редакторе Unity с помощью **:::no-loc text="Show children":::** кнопки.

![ремотинтитисинкобжект](media/remote-entity-sync-object.png)

## <a name="wrapper-components"></a>Компоненты-оболочки

[Компоненты](../../concepts/components.md) , подключенные к сущностям удаленной отрисовки, предоставляются Unity через прокси `MonoBehavior` s. Эти прокси-серверы представляют удаленный компонент в Unity и перенаправляют все изменения на узел.

Чтобы создать компоненты удаленной подготовки к просмотру прокси-сервера, используйте метод расширения `GetOrCreateArrComponent` :

```cs
var cutplane = gameObject.GetOrCreateArrComponent<ARRCutPlaneComponent>(RemoteManagerUnity.CurrentSession);
```

## <a name="coupled-lifetimes"></a>Пару жизненных циклов

Время существования удаленной [сущности](../../concepts/entities.md) и игрового объекта Unity связывается, пока они привязаны через `RemoteEntitySyncObject` . При вызове `UnityEngine.Object.Destroy(...)` с таким игровым объектом Удаленная сущность также будет удалена.

Чтобы уничтожить объект игры Unity, не затрагивая удаленную сущность, необходимо сначала вызвать метод `Unbind()` в `RemoteEntitySyncObject` .

Это справедливо и для всех прокси-компонентов. Чтобы уничтожить только представление на стороне клиента, сначала необходимо вызвать `Unbind()` прокси-компонент:

```cs
var cutplane = gameObject.GetComponent<ARRCutPlaneComponent>();
if (cutplane != null)
{
    cutplane.Unbind();
    UnityEngine.Object.Destroy(cutplane);
}
```

## <a name="next-steps"></a>Дальнейшие действия

* [Настройка Удаленной отрисовки для Unity](unity-setup.md)
* [Руководство. Работа с удаленными сущностями в Unity](../../tutorials/unity/manipulate-models/manipulate-models.md)
