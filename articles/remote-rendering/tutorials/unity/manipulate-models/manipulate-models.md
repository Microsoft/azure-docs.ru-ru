---
title: Манипулирование моделями
description: Удаленное манипулирование преобразованными для просмотра моделями путем перемещения, вращения, масштабирования и выполнения других действий
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 301d6eb0867604a6e780eb1d059eef0c153f246f
ms.sourcegitcommit: 87a6587e1a0e242c2cfbbc51103e19ec47b49910
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103574515"
---
# <a name="tutorial-manipulating-models"></a>Руководство по Манипулирование моделями

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Добавление границ для визуального элемента и манипуляций вокруг удаленно преобразованных для просмотра моделей
> * Перемещение, вращение и масштабирование
> * Рейкастинг с помощью пространственных запросов
> * Добавление простых анимаций для удаленно преобразованных для просмотра объектов

## <a name="prerequisites"></a>Предварительные требования

* Перед изучением этого учебника ознакомьтесь с [учебником по интерфейсам и пользовательским моделям](../custom-models/custom-models.md).

## <a name="query-remote-object-bounds-and-apply-to-local-bounds"></a>Запрос границ удаленных объектов и их применение к локальным границам

Для взаимодействия с удаленными объектами сначала необходимо обеспечить локальное представление для взаимодействия. [Границы объектов](../../../concepts/object-bounds.md) удобно использовать для быстрого манипулирования удаленным объектом. Удаленные границы можно запрашивать из службы "Удаленная отрисовка Azure", используя для ссылки локальную сущность. Границы запрашиваются после загрузки модели в удаленный сеанс.

Границы модели определяются рамкой, охватывающей всю модель, наподобие рамочного коллайдера [**BoxCollider**](https://docs.unity3d.com/Manual/class-BoxCollider.html) в Unity, центр и размер которой определяется координатами по осям x, y и z. На самом деле для представления границ удаленной модели мы будем использовать **BoxCollider** в Unity.

1. Создайте скрипт в том же каталоге, что и **RemoteRenderedModel**, и назовите его **RemoteBounds**.
1. Замените содержимое скрипта приведенным ниже кодом.

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    [RequireComponent(typeof(BaseRemoteRenderedModel))]
    public class RemoteBounds : BaseRemoteBounds
    {
        //Remote bounds works with a specific remotely rendered model
        private BaseRemoteRenderedModel targetModel = null;

        private RemoteBoundsState currentBoundsState = RemoteBoundsState.NotReady;

        public override RemoteBoundsState CurrentBoundsState
        {
            get => currentBoundsState;
            protected set
            {
                if (currentBoundsState != value)
                {
                    currentBoundsState = value;
                    BoundsStateChange?.Invoke(value);
                }
            }
        }

        public override event Action<RemoteBoundsState> BoundsStateChange;

        public void Awake()
        {
            BoundsStateChange += HandleUnityEvents;
            targetModel = GetComponent<BaseRemoteRenderedModel>();

            targetModel.ModelStateChange += TargetModel_OnModelStateChange;
            TargetModel_OnModelStateChange(targetModel.CurrentModelState);
        }

        private void TargetModel_OnModelStateChange(ModelState state)
        {
            switch (state)
            {
                case ModelState.Loaded:
                    QueryBounds();
                    break;
                default:
                    BoundsBoxCollider.enabled = false;
                    CurrentBoundsState = RemoteBoundsState.NotReady;
                    break;
            }
        }

        // Create an async query using the model entity
        async private void QueryBounds()
        {
            //Implement me
        }
    }
    ```

    > [!NOTE]
    > Если в Visual Studio отображается сообщение об ошибке наподобие *Возможность "X" недоступна в C# 6. Используйте версию языка 7.0 или выше*, ошибку такого рода можно просто проигнорировать. Она связана с созданием решения и проекта Unity.

    Этот скрипт должен быть добавлен в тот же игровой объект, что и скрипт, реализующий **BaseRemoteRenderedModel**. В данном случае — в **RemoteRenderedModel**. Как и предыдущие скрипты, этот исходный код будет обрабатывать все изменения состояния, события и данные, связанные с удаленными границами.

    Для реализации осталось применить только один метод: **QueryBounds**. **QueryBounds** получает границы, асинхронно, принимает результат запроса и применяет его к локальному экземпляру **BoxCollider**.

    Метод **QueryBounds** очень простой: он отправляет запрос в сеанс удаленной отрисовки и ожидает результат.

1. Замените метод **QueryBounds** следующим готовым методом:

    ```cs
    // Create a query using the model entity
    async private void QueryBounds()
    {
        var remoteBounds = targetModel.ModelEntity.QueryLocalBoundsAsync();
        CurrentBoundsState = RemoteBoundsState.Updating;
        await remoteBounds;

        if (remoteBounds.IsCompleted)
        {
            var newBounds = remoteBounds.Result.toUnity();
            BoundsBoxCollider.center = newBounds.center;
            BoundsBoxCollider.size = newBounds.size;
            BoundsBoxCollider.enabled = true;
            CurrentBoundsState = RemoteBoundsState.Ready;
        }
        else
        {
            CurrentBoundsState = RemoteBoundsState.Error;
        }
    }
    ```

    Мы проверим запрос, чтобы убедиться в успешности его выполнения. Если это так, преобразуйте и примените возвращенные границы в формат, который может принимать **BoxCollider**.

Теперь, когда скрипт **RemoteBounds** добавлен к тому же игровому объекту, что и **RemoteRenderedModel**, в случае необходимости будет добавляться **BoxCollider**. Когда модель достигнет состояния `Loaded`, границы будут автоматически запрашиваться и применяться к **BoxCollider**.

1. Используя созданный ранее игровой объект **TestModel**, добавьте компонент **RemoteBounds**.
1. Убедитесь, что скрипт добавлен.

     ![Добавление компонента RemoteBounds](./media/remote-bounds-script.png)

1. Повторный запуск приложения Вскоре после загрузки модели отобразятся границы для удаленного объекта. Вы увидите примерно следующие значения:

     ![Снимок экрана: пример границ для удаленного объекта.](./media/updated-bounds.png)

Теперь у нас есть локальный коллайдер **BoxCollider**, настроенный по точным границам объекта Unity. Границы обеспечивают визуализацию и взаимодействие с использованием тех же стратегий, которые применяются для объекта, преобразовываемого для просмотра локально. Например, с помощью скриптов, которые изменяют преобразование, физику и многое другое.

## <a name="move-rotate-and-scale"></a>Перемещение, вращение и масштабирование  

Перемещение, вращение и масштабирование удаленно преобразовываемых для просмотра объектов выполняется так же, как для любого другого объекта Unity. **RemoteRenderingCoordinator** вызывает `Update` в методе `LateUpdate` для текущего активного сеанса. Одна из операций, выполняемых `Update`, — синхронизация преобразований сущностей локальной модели с их удаленными аналогами. Чтобы переместить, повернуть или масштабировать удаленно преобразованную для просмотра модель, достаточно переместить, повернуть или масштабировать преобразование игрового объекта, представляющего удаленную модель. В данном случае мы собираемся изменить преобразование родительского игрового объекта с прикрепленным к нему скриптом **RemoteRenderedModel**.

В этом учебнике для взаимодействия с объектом используется MRTK. Реализация конкретных возможностей для перемещения, вращения и масштабирования объекта с помощью MRTK по большей части выходит за рамки данного учебника. В **AppMenu** в меню **Model Tools** (Средства работы с моделью) предварительно настроен контроллер представления модели.

1. Убедитесь, что созданный ранее игровой объект **TestModel** находится в сцене.
1. Убедитесь, что префаб **AppMenu** находится в сцене.
1. В Unity нажмите кнопку Play (Воспроизвести), чтобы воспроизвести сцену и открыть меню **Model Tools** (Средства работы с моделью) в **AppMenu**.
![Контроллер представления](./media/model-with-view-controller.png)

Префаб **AppMenu** содержит подменю **Model Tools** (Средства работы с моделью), которое реализует контроллер представления для привязки к модели. Если игровой объект содержит компонент **RemoteBounds**, контроллер представления добавит компонент [**BoundingBox**](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_BoundingBox.html). Это компонент MRTK, который отрисовывает ограничивающий прямоугольник вокруг объекта с **BoxCollider**. [**ObjectManipulator**](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.5.1/api/Microsoft.MixedReality.Toolkit.UI.ObjectManipulator.html) отвечает за взаимодействие с помощью руки. Вместе эти скрипты позволяют перемещать, вращать и масштабировать удаленно подготовленную для просмотра модель.

1. Переместите указатель мыши на игровую панель и щелкните внутри нее, чтобы переместить на нее фокус.
1. Используя [имитацию руки MRTK](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/InputSimulation/InputSimulationService.html#hand-simulation), нажмите и удерживайте клавишу SHIFT слева.
1. Обведите имитацию руки, чтобы ее луч указывал на тестовую модель.

    ![Направленный луч руки](./media/handray-engine.png)

1. Удерживая левую кнопку мыши, перетащите модель, чтобы ее переместить.

Удаленно преобразованное для просмотра содержимое будет перемещаться вместе с ограничивающим прямоугольником. Вы можете заметить некоторую задержку или запаздывание между ограничивающим прямоугольником и удаленным содержимым. Эта задержка будет зависеть от задержки в Интернете и пропускной способности.

## <a name="ray-cast-and-spatial-queries-of-remote-models"></a>Рейкастинг и пространственные запросы удаленных моделей

BoxCollider вокруг модели подходит для взаимодействия с моделью целиком, но не обеспечивает достаточную детализацию для работы с ее отдельными частями. Чтобы решить эту проблему, можно использовать [удаленный рейкастинг](../../../overview/features/spatial-queries.md#ray-casts). Удаленный рейкастинг — это API, предоставляемый Удаленной отрисовкой Azure для бросания лучей на удаленную сцену и локального возврата результатов их пересечения с объектами. Этот метод можно использовать для выбора дочерних сущностей большой модели или получения сведений о результате пересечения, таких как расположение, нормаль к поверхности и расстояние.

В тестовой модели имеется ряд вложенных сущностей, которые можно запрашивать и выбирать. На данный момент в результате выбора в консоль Unity выводится имя выбранной сущности. Сведения о выделении выбранной сущности см. в главе о [материалах, освещении и эффектах](../materials-lighting-effects/materials-lighting-effects.md#highlighting-and-outlining).

Сначала создадим статическую оболочку для запросов удаленного рейкастинга. Этот скрипт будет принимать расположение и направление в пространстве Unity, преобразовывать его в типы данных, принимаемые удаленным рейкастингом, и возвращать результаты. Скрипт будет использовать API-интерфейс `RayCastQueryAsync`.

1. Создайте скрипт с именем **RemoteRayCaster** и замените все его содержимое следующим кодом:

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System.Linq;
    using System.Threading.Tasks;
    using UnityEngine;

    /// <summary>
    /// Wraps the Azure Remote Rendering RayCast queries to easily send requests using Unity data types
    /// </summary>
    public class RemoteRayCaster
    {
        public static double maxDistance = 30.0;

        public static async Task<RayCastHit[]> RemoteRayCast(Vector3 origin, Vector3 dir, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            if(RemoteRenderingCoordinator.instance.CurrentCoordinatorState == RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected)
            {
                var rayCast = new RayCast(origin.toRemotePos(), dir.toRemoteDir(), maxDistance, hitPolicy);
                var result = await RemoteRenderingCoordinator.CurrentSession.Connection.RayCastQueryAsync(rayCast);
                return result.Hits;
            }
            else
            {
                return new RayCastHit[0];
            }
        }

        public static async Task<Entity[]> RemoteRayCastEntities(Vector3 origin, Vector3 dir, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            var hits = await RemoteRayCast(origin, dir, hitPolicy);
            return hits.Select(hit => hit.HitEntity).Where(entity => entity != null).ToArray();
        }
    }
    ```

    > [!NOTE]
    > В Unity существует класс с именем [**RaycastHit**](https://docs.unity3d.com/ScriptReference/RaycastHit.html), а в службе "Удаленная отрисовка Azure" — класс [**RayCastHit**](/dotnet/api/microsoft.azure.remoterendering.raycasthit). Во избежание ошибок компиляции обратите внимание на важное различие: разный регистр буквы **C**.

    **RemoteRayCaster** предоставляет общую точку доступа для удаленного рейкастинга в текущий сеанс. Уточним, что далее мы будем реализовывать обработчик указателя MRTK. Скрипт реализует интерфейс `IMixedRealityPointerHandler`, который будет сообщать MRTK, что этот скрипт ожидает передачи данных о событиях [указателя Смешанной реальности](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/Input/Pointers.html).

1. Создайте скрипт с именем **RemoteRayCastPointerHandler** и замените все его содержимое следующим кодом:

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.MixedReality.Toolkit.Input;
    using System;
    using System.Linq;
    using System.Threading.Tasks;
    using UnityEngine;

    public class RemoteRayCastPointerHandler : BaseRemoteRayCastPointerHandler, IMixedRealityPointerHandler
    {
        public UnityRemoteEntityEvent OnRemoteEntityClicked = new UnityRemoteEntityEvent();

        public override event Action<Entity> RemoteEntityClicked;

        public void Awake()
        {
            // Forward events to Unity events
            RemoteEntityClicked += (entity) => OnRemoteEntityClicked?.Invoke(entity);
        }

        public async void OnPointerClicked(MixedRealityPointerEventData eventData)
        {
            if (RemoteEntityClicked != null) //Ensure someone is listening before we do the work
            {
                var firstHit = await PointerDataToRemoteRayCast(eventData.Pointer);
                if (firstHit.success)
                    RemoteEntityClicked.Invoke(firstHit.hit.HitEntity);
            }
        }

        public void OnPointerDown(MixedRealityPointerEventData eventData) { }

        public void OnPointerDragged(MixedRealityPointerEventData eventData) { }

        public void OnPointerUp(MixedRealityPointerEventData eventData) { }

        private async Task<(bool success, RayCastHit hit)> PointerDataToRemoteRayCast(IMixedRealityPointer pointer, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            RayCastHit hit;
            var result = pointer.Result;
            if (result != null)
            {
                var endPoint = result.Details.Point;
                var direction = pointer.Rays[pointer.Result.RayStepIndex].Direction;
                Debug.DrawRay(endPoint, direction, Color.green, 0);
                hit = (await RemoteRayCaster.RemoteRayCast(endPoint, direction, hitPolicy)).FirstOrDefault();
            }
            else
            {
                hit = new RayCastHit();
            }
            return (hit.HitEntity != null, hit);
        }
    }
    ```

Метод `OnPointerClicked` скрипта **RemoteRayCastPointerHandler** вызывается MRTK при "щелчках" указателем по коллайдеру, например нашему коллайдеру BoxCollider. После этого вызывается `PointerDataToRemoteRayCast` для преобразования полученного для указателя результата в точку и направление. Затем эта точка и направление используются для бросания удаленного луча в удаленном сеансе.

![Обновление границ](./media/raycast-local-remote.png)

Отправка запросов на рейкастинг по щелчку является эффективной стратегией запросов удаленных объектов. Однако такое взаимодействие с пользователем не идеально, так как при этом курсор попадает на BoxCollider, а не на саму модель.

Можно также создать указатель MRTK, который будет чаще бросать лучи в удаленном сеансе. Хотя это более сложный подход, взаимодействие с пользователем улучшится. Эта стратегия выходит за рамки данного учебника, но пример такого подхода можно увидеть в демонстрационном приложении, которое находится в [репозитории примеров службы "Удаленная отрисовка Azure"](https://github.com/Azure/azure-remote-rendering/tree/master/Unity/Showcase).

При успешном выполнении рейкастинга в **RemoteRayCastPointerHandler** пересекающая объекты сущность `Entity` испускается при наступлении события `OnRemoteEntityClicked` в Unity. Чтобы отреагировать на это событие, мы создадим вспомогательный скрипт, который принимает сущность `Entity` и выполняет над ней действие. Для начала создадим скрипт, выводящий имя `Entity` в журнал отладки.

1. Создайте скрипт с именем **RemoteEntityHelper** и замените его содержимое следующим:

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.
    
    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using UnityEngine;
    
    public class RemoteEntityHelper : MonoBehaviour
    {
        public void EntityToDebugLog(Entity entity)
        {
            Debug.Log(entity.Name);
        }
    }
    ```

2. В ранее созданном игровом объекте **TestModel** добавьте компоненты **RemoteRayCastPointerHandler** и **RemoteEntityHelper**.
1. Назначьте метод `EntityToDebugLog` событию `OnRemoteEntityClicked`. Если тип выходных данных события совпадает с типом входных данных метода, можно использовать динамическую привязку события Unity, которая будет автоматически передавать значение события методу.
    1. Создайте новое поле обратного вызова. ![Добавление обратного вызова](./media/add-callback-remote-entity-clicked.png)
    1. Перетащите компонент **Remote Entity Helper** (Вспомогательный объект удаленной сущности) в поле Object (Объект), чтобы сослаться на родительский игровой объект. ![Назначение объекта](./media/assign-object.png)
    1. Назначьте `EntityToDebugLog` в качестве обратного вызова. ![Назначение обратного вызова](./media/remote-entity-event.png)
1. Нажмите кнопку воспроизведения в редакторе Unity, чтобы начать отрисовку сцены, подключитесь к удаленному сеансу и загрузите тестовую модель.
1. Используя имитацию руки MRTK, нажмите и удерживайте клавишу SHIFT слева.
1. Обведите имитацию руки, чтобы ее луч указывал на тестовую модель.
1. Щелкните и удерживайте кнопку мыши, чтобы имитировать касание, инициирующее событие `OnPointerClicked`.
1. Следите за консолью Unity, в которой должно отобразиться сообщение журнала с именем выбранной дочерней сущности. Пример: ![Пример дочерней сущности](./media/child-entity-example.png)

## <a name="synchronizing-the-remote-object-graph-into-the-unity-hierarchy"></a>Синхронизация удаленного графа объектов в иерархии Unity

До этого момента мы видели только один локальный игровой объект, представляющий всю модель. Такая схема хорошо подходит для отрисовки всей модели и манипулирования ею как единым целым. Однако, если нужно применить эффекты к конкретным вложенным сущностям или манипулировать ими, необходимо создать локальные игровые объекты для представления этих сущностей. Для начала мы можем изучить этот подход вручную на тестовой модели.

1. Запустите сцену и загрузите тестовую модель.
1. Разверните дочерний игровой объект **TestModel** в иерархии Unity и выберите игровой объект **TestModel_Entity**.
1. В области Inspector (Инспектор) нажмите кнопку *Show Children* (Показывать дочерние объекты).
![Кнопка Show children (Показывать дочерние объекты)](./media/show-remote-children.png)
1. Продолжайте развертывать дочерние объекты в иерархии и нажимать кнопку *Show Children* (Показывать дочерние объекты), пока не отобразится большой список дочерних объектов.
![Все дочерние объекты](./media/test-model-children.png)

Теперь иерархия будет заполнена списком десятков сущностей. При выборе одного из них в инспекторе будут отображаться компоненты `Transform` и `RemoteEntitySyncObject`. По умолчанию автоматическая синхронизация каждого кадра для каждой сущности отключена, поэтому локальные изменения в компоненте `Transform` не синхронизируются с сервером. Вы можете установить флажок *Sync Every Frame* (Синхронизировать каждый кадр), а затем переместить, масштабировать или повернуть преобразование в представлении сцены. Преобразованная для просмотра модель не будет отображаться в представлении сцены. Чтобы увидеть визуальное изменение положения и вращение модели, воспользуйтесь игровым представлением.

Этот же процесс можно осуществить программно, и это первый шаг к изменению конкретных удаленных сущностей.

1. Измените скрипт **RemoteEntityHelper**, чтобы он также содержал следующий метод:

    ```cs
    public void MakeSyncedGameObject(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var sync = entityGameObject.GetComponent<RemoteEntitySyncObject>();
        sync.SyncEveryFrame = true;
    }
    ```

1. Добавьте дополнительный обратный вызов в событие `OnRemoteEntityClicked` скрипта **RemoteRayCastPointerHandler**, установив для него значение `MakeSyncedGameObject`.
![Дополнительный обратный вызов](./media/additional-callback.png)
1. Используя имитацию руки MRTK, нажмите и удерживайте клавишу SHIFT слева.
1. Обведите имитацию руки, чтобы ее луч указывал на тестовую модель.
1. Щелкните и удерживайте кнопку мыши, чтобы имитировать касание, инициирующее событие `OnPointerClicked`.
1. Проверьте и разверните иерархию, чтобы увидеть новый дочерний объект, представляющий сущность, которую щелкнул пользователь.
![Представление игрового объекта](./media/gameobject-representing-entity.png)
1. После тестирования удалите обратный вызов для `MakeSyncedGameObject`, так как мы внедрим его позже в составе других эффектов.

> [!NOTE]
> Синхронизация каждого кадра необходима только в случае, если необходимо синхронизировать данные преобразования. Синхронизация преобразований сопряжена с определенными накладными расходами, поэтому прибегать к ней следует только в крайних случаях.

Создание локального экземпляра и обеспечение его автоматической синхронизации — это первый шаг к манипулированию вложенными сущностями. Для вложенных сущностей можно использовать те же методы, которые мы применяли для манипулирования моделью в целом. Например, после создания синхронизированного локального экземпляра сущности можно запрашивать его границы. Кроме того, можно добавить обработчики манипуляций, чтобы обеспечить его перемещение с помощью лучей руки пользователя.

## <a name="next-steps"></a>Дальнейшие действия

Теперь вы можете манипулировать удаленно преобразованными для просмотра моделями и взаимодействовать с ними. В следующем учебнике мы обсудим изменение материалов и освещения, а также применение эффектов к удаленно преобразованным для просмотра моделям.

> [!div class="nextstepaction"]
> [Дальнейшие действия. Детализация материалов, освещения и эффектов](../materials-lighting-effects/materials-lighting-effects.md)