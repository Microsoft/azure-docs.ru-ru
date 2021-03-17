---
title: Детализация материалов, освещения и эффектов
description: Изменение материалов модели и освещения. Добавление дополнительных эффектов, таких как структурирование и секущие плоскости.
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 400f5de5326ce7700b627999ade312484055a4e7
ms.sourcegitcommit: 87a6587e1a0e242c2cfbbc51103e19ec47b49910
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/16/2021
ms.locfileid: "103574447"
---
# <a name="tutorial-refining-materials-lighting-and-effects"></a>Руководство по Детализация материалов, освещения и эффектов

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Выделение и структурирование моделей и их компонентов
> * Применение различных материалов к моделям
> * Создание срезов моделей с помощью секущих плоскостей
> * Добавление простых анимаций для удаленно преобразованных для просмотра объектов

## <a name="prerequisites"></a>Предварительные требования

* Перед изучением этого учебника ознакомьтесь с [учебником по манипулированию моделями](..\manipulate-models\manipulate-models.md).

## <a name="highlighting-and-outlining"></a>Выделение и структурирование

Предоставление визуальной обратной связи — это важная часть взаимодействия с пользователем в любом приложении. Удаленная отрисовка Azure предоставляет механизмы визуальной обратной связи с помощью [иерархических переопределений состояний](../../../overview/features/override-hierarchical-state.md). Иерархические переопределения состояния реализуются с помощью компонентов, присоединенных к локальным экземплярам моделей. Процесс создания этих локальных экземпляров описан в разделе [Синхронизация удаленного графа объектов в иерархии Unity](../manipulate-models/manipulate-models.md#synchronizing-the-remote-object-graph-into-the-unity-hierarchy).

Сначала мы создадим оболочку вокруг компонента [**HierarchicalStateOverrideComponent**](/dotnet/api/microsoft.azure.remoterendering.hierarchicalstateoverridecomponent). **HierarchicalStateOverrideComponent** — это локальный скрипт, который управляет переопределениями удаленной сущности. [**Учебные ресурсы**](../custom-models/custom-models.md#import-assets-used-by-this-tutorial) включают в себя абстрактный базовый класс с именем **BaseEntityOverrideController**, который мы расширим для создания оболочки.

1. Создайте скрипт с именем **EntityOverrideController** и замените все его содержимое следующим кодом:

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    public class EntityOverrideController : BaseEntityOverrideController
    {
        public override event Action<HierarchicalStates> FeatureOverrideChange;

        private ARRHierarchicalStateOverrideComponent localOverride;
        public override ARRHierarchicalStateOverrideComponent LocalOverride
        {
            get
            {
                if (localOverride == null)
                {
                    localOverride = gameObject.GetComponent<ARRHierarchicalStateOverrideComponent>();
                    if (localOverride == null)
                    {
                        localOverride = gameObject.AddComponent<ARRHierarchicalStateOverrideComponent>();
                    }

                    var remoteStateOverride = TargetEntity.Entity.FindComponentOfType<HierarchicalStateOverrideComponent>();

                    if (remoteStateOverride == null)
                    {
                        // if there is no HierarchicalStateOverrideComponent on the remote side yet, create one
                        localOverride.Create(RemoteManagerUnity.CurrentSession);
                    }
                    else
                    {
                        // otherwise, bind our local stateOverride component to the remote component
                        localOverride.Bind(remoteStateOverride);

                    }
                }
                return localOverride;
            }
        }

        private RemoteEntitySyncObject targetEntity;
        public override RemoteEntitySyncObject TargetEntity
        {
            get
            {
                if (targetEntity == null)
                    targetEntity = gameObject.GetComponent<RemoteEntitySyncObject>();
                return targetEntity;
            }
        }

        private HierarchicalEnableState ToggleState(HierarchicalStates feature)
        {
            HierarchicalEnableState setToState = HierarchicalEnableState.InheritFromParent;
            switch (LocalOverride.RemoteComponent.GetState(feature))
            {
                case HierarchicalEnableState.ForceOff:
                case HierarchicalEnableState.InheritFromParent:
                    setToState = HierarchicalEnableState.ForceOn;
                    break;
                case HierarchicalEnableState.ForceOn:
                    setToState = HierarchicalEnableState.InheritFromParent;
                    break;
            }

            return SetState(feature, setToState);
        }

        private HierarchicalEnableState SetState(HierarchicalStates feature, HierarchicalEnableState enableState)
        {
            if (GetState(feature) != enableState) //if this is actually different from the current state, act on it
            {
                LocalOverride.RemoteComponent.SetState(feature, enableState);
                FeatureOverrideChange?.Invoke(feature);
            }

            return enableState;
        }

        public override HierarchicalEnableState GetState(HierarchicalStates feature) => LocalOverride.RemoteComponent.GetState(feature);

        public override void ToggleHidden() => ToggleState(HierarchicalStates.Hidden);

        public override void ToggleSelect() => ToggleState(HierarchicalStates.Selected);

        public override void ToggleSeeThrough() => ToggleState(HierarchicalStates.SeeThrough);

        public override void ToggleTint(Color tintColor = default)
        {
            if (tintColor != default) LocalOverride.RemoteComponent.TintColor = tintColor.toRemote();
            ToggleState(HierarchicalStates.UseTintColor);
        }

        public override void ToggleDisabledCollision() => ToggleState(HierarchicalStates.DisableCollision);

        public override void RemoveOverride()
        {
            var remoteStateOverride = TargetEntity.Entity.FindComponentOfType<HierarchicalStateOverrideComponent>();
            if (remoteStateOverride != null)
            {
                remoteStateOverride.Destroy();
            }

            if (localOverride == null)
                localOverride = gameObject.GetComponent<ARRHierarchicalStateOverrideComponent>();

            if (localOverride != null)
            {
                Destroy(localOverride);
            }
        }
    }
    ```

Основным заданием переопределения **LocalOverride** является создание связи между ним самим и `RemoteComponent`. **LocalOverride** позволит задать для локального компонента флаги состояния, привязанные к удаленной сущности. Переопределения и их состояния описаны на странице [Иерархические переопределения состояния](../../../overview/features/override-hierarchical-state.md). 

Эта реализация просто по очереди переключает состояния одно за другим. Однако вполне можно объединить несколько переопределений отдельных сущностей и создать сочетания на разных уровнях иерархии. Например, сочетание `Selected` и `SeeThrough` для одного компонента придаст ему не только структуру, но и прозрачность. Если же установить для переопределения корневой сущности `Hidden` значение `ForceOn` и при этом задать для переопределения дочерней сущности `Hidden` значение `ForceOff`, в результате будет скрыто все, кроме дочерней сущности.

Чтобы применить состояния к сущностям, можно изменить созданный ранее скрипт **RemoteEntityHelper**.

1. Измените класс **RemoteEntityHelper**, чтобы реализовать абстрактный класс **BaseRemoteEntityHelper**. Это изменение позволит использовать контроллер представления, предоставленный в составе **учебных ресурсов**. После изменения результат будет выглядеть так:

    ```cs
    public class RemoteEntityHelper : BaseRemoteEntityHelper
    ```

2. Переопределите абстрактные методы, используя следующий код:

    ```cs
    public override BaseEntityOverrideController EnsureOverrideComponent(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var overrideComponent = entityGameObject.GetComponent<EntityOverrideController>();
        if (overrideComponent == null)
            overrideComponent = entityGameObject.AddComponent<EntityOverrideController>();
        return overrideComponent;
    }

    public override HierarchicalEnableState GetState(Entity entity, HierarchicalStates feature)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        return overrideComponent.GetState(feature);
    }

    public override void ToggleHidden(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleHidden();
    }

    public override void ToggleSelect(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleSelect();
    }

    public override void ToggleSeeThrough(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleSeeThrough();
    }

    public Color TintColor = new Color(0.0f, 1.0f, 0.0f, 0.1f);
    public override void ToggleTint(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleTint(TintColor);
    }

    public override void ToggleDisableCollision(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleHidden();
    }

    public override void RemoveOverrides(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var overrideComponent = entityGameObject.GetComponent<EntityOverrideController>();
        if (overrideComponent != null)
        {
            overrideComponent.RemoveOverride();
            Destroy(overrideComponent);
        }
    }
    ```

Этот код гарантирует добавление компонента **EntityOverrideController** в целевую сущность с последующим вызовом одного из переключающих методов. При необходимости эти вспомогательные методы можно вызвать в игровом объекте **TestModel**, добавив **RemoteEntityHelper** в качестве обратного вызова для события `OnRemoteEntityClicked` компонента **RemoteRayCastPointerHandler**.

![Обратные вызовы указателя](./media/pointer-event-callbacks.png)

Теперь, когда эти скрипты добавлены в модель, после подключения к среде выполнения у контроллера представления **AppMenu** должны быть включены дополнительные интерфейсы для взаимодействия со скриптом **EntityOverrideController**. Проверьте, есть ли в меню **Model Tools** (Средства работы с моделью) разблокированные контроллеры представления.

На этом этапе компоненты игрового объекта **TestModel** должны выглядеть примерно так:

![Тестовая модель с дополнительными скриптами](./media/test-model-updated.png)

Ниже приведен пример распределенных переопределений для одной сущности. Мы использовали `Select` и `Tint`, чтобы обеспечить структуру и выделение цветом.

![Выбор окраски тестовой модели](./media/select-tint-test-model.png)

## <a name="cut-planes"></a>Вырезание плоскостей

[Секущие плоскости](../../../overview/features/cut-planes.md) — это функция, которую можно добавить в любую удаленную сущность. Чаще всего создаваемая удаленная сущность не связана с какими-либо данными сетки для размещения такого компонента, как секущая плоскость. Положение и ориентация секущей плоскости определяются положением и ориентацией удаленной сущности, к которой она присоединена.

Мы создадим скрипт, который автоматически создает удаленную сущность, добавляет компонент "Секущая плоскость" и синхронизирует преобразование локального объекта с сущностью секущей плоскости. Затем мы сможем воспользоваться **CutPlaneViewController**, чтобы заключить секущую плоскость в оболочку-интерфейс, что позволит нам манипулировать ею.

1. Создайте скрипт с именем **RemoteCutPlane** и замените его код приведенным ниже.

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    public class RemoteCutPlane : BaseRemoteCutPlane
    {
        public Color SliceColor = new Color(0.5f, 0f, 0f, .5f);
        public float FadeLength = 0.01f;
        public Axis SliceNormal = Axis.NegativeY;

        public bool AutomaticallyCreate = true;

        private CutPlaneComponent remoteCutPlaneComponent;
        private bool cutPlaneReady = false;

        public override bool CutPlaneReady 
        { 
            get => cutPlaneReady;
            set 
            { 
                cutPlaneReady = value;
                CutPlaneReadyChanged?.Invoke(cutPlaneReady);
            }
        }

        public override event Action<bool> CutPlaneReadyChanged;

        public UnityBoolEvent OnCutPlaneReadyChanged = new UnityBoolEvent();

        public void Start()
        {
            // Hook up the event to the Unity event
            CutPlaneReadyChanged += (ready) => OnCutPlaneReadyChanged?.Invoke(ready);

            RemoteRenderingCoordinator.CoordinatorStateChange += RemoteRenderingCoordinator_CoordinatorStateChange;
            RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        private void RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    if (AutomaticallyCreate)
                        CreateCutPlane();
                    break;
                default:
                    DestroyCutPlane();
                    break;
            }
        }

        public override void CreateCutPlane()
        {
            //Implement me
        }

        public override void DestroyCutPlane()
        {
            //Implement me
        }
    }
    ```

    Этот код расширяет класс **BaseRemoteCutPlane**, который входит в состав **учебных ресурсов**. Аналогично удаленно преобразованной для просмотра модели этот скрипт присоединяется к удаленному координатору и ожидает от него передачи данных об изменениях `RemoteRenderingState`. Когда координатор достигнет состояния `RuntimeConnected`, он попытается автоматически подключиться (если он должен это делать). Мы будем также отслеживать переменную `CutPlaneComponent`. Это компонент службы "Удаленная отрисовка Azure", который синхронизируется с секущей плоскостью в удаленном сеансе. Давайте посмотрим, что нужно сделать, чтобы создать секущую плоскость.

2. Замените метод `CreateCutPlane()` приведенной ниже готовой версией.

    ```cs
    public override void CreateCutPlane()
    {
        if (remoteCutPlaneComponent != null)
            return; //Nothing to do!

        //Create a root object for the cut plane
        var cutEntity = RemoteRenderingCoordinator.CurrentSession.Connection.CreateEntity();

        //Bind the remote entity to this game object
        cutEntity.BindToUnityGameObject(this.gameObject);

        //Sync the transform of this object so we can move the cut plane
        var syncComponent = this.gameObject.GetComponent<RemoteEntitySyncObject>();
        syncComponent.SyncEveryFrame = true;

        //Add a cut plane to the entity
        remoteCutPlaneComponent = RemoteRenderingCoordinator.CurrentSession.Connection.CreateComponent(ObjectType.CutPlaneComponent, cutEntity) as CutPlaneComponent;

        //Configure the cut plane
        remoteCutPlaneComponent.Normal = SliceNormal;
        remoteCutPlaneComponent.FadeColor = SliceColor.toRemote();
        remoteCutPlaneComponent.FadeLength = FadeLength;
        CutPlaneReady = true;
    }
    ```

    Здесь мы создаем удаленную сущность и привязываем ее к локальному игровому объекту. Мы обеспечим синхронизацию преобразования удаленной сущности с локальным преобразованием, установив для `SyncEveryFrame` значение `true`. Затем мы используем вызов `CreateComponent`, чтобы добавить `CutPlaneComponent` к удаленному объекту. Наконец, мы настроим секущую плоскость с помощью параметров, определяемых в верхней части класса MonoBehaviour. Давайте посмотрим, что требуется для очистки секущей плоскости, реализовав метод `DestroyCutPlane()`.

3. Замените метод `DestroyCutPlane()` приведенной ниже готовой версией.

    ```cs
    public override void DestroyCutPlane()
    {
        if (remoteCutPlaneComponent == null)
            return; //Nothing to do!

        remoteCutPlaneComponent.Owner.Destroy();
        remoteCutPlaneComponent = null;
        CutPlaneReady = false;
    }
    ```

Так как удаленный объект довольно прост и мы очищаем только удаленную часть (сохранив локальный объект), достаточно просто вызвать `Destroy` для удаленного объекта и удалить указывающую на него ссылку.

**AppMenu** включает в себя контроллер представления, который автоматически присоединяется к секущей плоскости и позволяет взаимодействовать с ней. **AppMenu** или другой контроллер представления использовать не обязательно, но они упрощают работу. Теперь протестируем секущую плоскость и ее контроллер представления.

1. Создайте пустой игровой объект в сцене и назовите его **CutPlane**.
1. Добавьте компонент **RemoteCutPlane** в игровой объект **CutPlane**.

   ![Конфигурация компонента "Секущая плоскость"](./media/cut-plane-config.png)

1. Нажмите кнопку воспроизведения в редакторе Unity, чтобы загрузить модель и подключиться к удаленному сеансу.
1. Используя имитацию руки MRTK, возьмите и поверните объект CutPlane (удерживайте клавишу CTRL для поворота) для перемещения его в сцене. Посмотрите, как она создает срез игрового объекта **TestModel**, отображая внутренние компоненты.

![Пример секущей плоскости](./media/cut-plane-example-engine.png)

## <a name="configuring-the-remote-lighting"></a>Настройка удаленного освещения

Сеанс удаленной отрисовки поддерживает полный спектр [параметров освещения](../../../overview/features/lights.md). Мы создадим скрипты для [текстуры неба](../../../overview/features/sky.md) и простую карту для двух типов освещения Unity, которые будут использоваться при удаленной отрисовке.

### <a name="sky-texture"></a>Текстура неба

При изменении текстуры неба можно выбрать одну из нескольких встроенных кубических карт. Они загружаются в сеанс и применяются к текстуре неба. Можно также [загрузить собственные текстуры](../../../concepts/textures.md) и использовать их в качестве источника освещения с неба.

Мы создадим скрипт **RemoteSky** со списком встроенных доступных кубических карт в виде параметров загрузки. После этого мы предоставим пользователю возможность выбрать и загрузить один из вариантов.

1. Создайте скрипт с именем **RemoteSky** и замените все его содержимое следующим кодом:

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using System;
    using System.Collections.Generic;
    using UnityEngine;

    public class RemoteSky : BaseRemoteSky
    {
        public override Dictionary<string, LoadTextureFromSasOptions> AvailableCubemaps => builtInTextures;

        private bool canSetSky;
        public override bool CanSetSky
        {
            get => canSetSky;
            set
            {
                canSetSky = value;
                CanSetSkyChanged?.Invoke(canSetSky);
            }
        }

        private string currentSky = "DefaultSky";
        public override string CurrentSky
        {
            get => currentSky;
            protected set
            {
                currentSky = value;
                SkyChanged?.Invoke(value);
            }
        }

        private Dictionary<string, LoadTextureFromSasOptions> builtInTextures = new Dictionary<string, LoadTextureFromSasOptions>()
        {
            {"Autoshop",new LoadTextureFromSasOptions("builtin://Autoshop", TextureType.CubeMap)},
            {"BoilerRoom",new LoadTextureFromSasOptions("builtin://BoilerRoom", TextureType.CubeMap)},
            {"ColorfulStudio",new LoadTextureFromSasOptions("builtin://ColorfulStudio", TextureType.CubeMap)},
            {"Hangar",new LoadTextureFromSasOptions("builtin://Hangar", TextureType.CubeMap)},
            {"IndustrialPipeAndValve",new LoadTextureFromSasOptions("builtin://IndustrialPipeAndValve", TextureType.CubeMap)},
            {"Lebombo",new LoadTextureFromSasOptions("builtin://Lebombo", TextureType.CubeMap)},
            {"SataraNight",new LoadTextureFromSasOptions("builtin://SataraNight", TextureType.CubeMap)},
            {"SunnyVondelpark",new LoadTextureFromSasOptions("builtin://SunnyVondelpark", TextureType.CubeMap)},
            {"Syferfontein",new LoadTextureFromSasOptions("builtin://Syferfontein", TextureType.CubeMap)},
            {"TearsOfSteelBridge",new LoadTextureFromSasOptions("builtin://TearsOfSteelBridge", TextureType.CubeMap)},
            {"VeniceSunset",new LoadTextureFromSasOptions("builtin://VeniceSunset", TextureType.CubeMap)},
            {"WhippleCreekRegionalPark",new LoadTextureFromSasOptions("builtin://WhippleCreekRegionalPark", TextureType.CubeMap)},
            {"WinterRiver",new LoadTextureFromSasOptions("builtin://WinterRiver", TextureType.CubeMap)},
            {"DefaultSky",new LoadTextureFromSasOptions("builtin://DefaultSky", TextureType.CubeMap)}
        };

        public UnityBoolEvent OnCanSetSkyChanged;
        public override event Action<bool> CanSetSkyChanged;

        public UnityStringEvent OnSkyChanged;
        public override event Action<string> SkyChanged;

        public void Start()
        {
            // Hook up the event to the Unity event
            CanSetSkyChanged += (canSet) => OnCanSetSkyChanged?.Invoke(canSet);
            SkyChanged += (key) => OnSkyChanged?.Invoke(key);

            RemoteRenderingCoordinator.CoordinatorStateChange += ApplyStateToView;
            ApplyStateToView(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        private void ApplyStateToView(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    CanSetSky = true;
                    break;
                default:
                    CanSetSky = false;
                    break;
            }
        }

        public override async void SetSky(string skyKey)
        {
            if (!CanSetSky)
            {
                Debug.Log("Unable to set sky right now");
                return;
            }

            if (AvailableCubemaps.ContainsKey(skyKey))
            {
                Debug.Log("Setting sky to " + skyKey);
                //Load the texture into the session
                var texture = await RemoteRenderingCoordinator.CurrentSession.Connection.LoadTextureFromSasAsync(AvailableCubemaps[skyKey]);

                //Apply the texture to the SkyReflectionSettings
                RemoteRenderingCoordinator.CurrentSession.Connection.SkyReflectionSettings.SkyReflectionTexture = texture;
                SkyChanged?.Invoke(skyKey);
            }
            else
            {
                Debug.Log("Invalid sky key");
            }
        }
    }
    ```

    Наиболее важной частью этого кода являются всего несколько строк.

    ```cs
    //Load the texture into the session
    var texture = await RemoteRenderingCoordinator.CurrentSession.Connection.LoadTextureFromSasAsync(AvailableCubemaps[skyKey]);

    //Apply the texture to the SkyReflectionSettings
    RemoteRenderingCoordinator.CurrentSession.Connection.SkyReflectionSettings.SkyReflectionTexture = texture;
    ```

    Здесь мы получаем ссылку на необходимую текстуру, загрузив ее в сеанс из встроенного Хранилища BLOB-объектов. Остается только назначить эту текстуру свойству `SkyReflectionTexture` сеанса, чтобы ее применить.

1. Создайте пустой игровой объект в сцене и назовите его **SkyLight**.

1. Добавьте скрипт **RemoteSky** в игровой объект **SkyLight**.

    Переключение между источниками освещения с неба можно выполнить путем вызова `SetSky` с одним из строковых ключей, определенных в `AvailableCubemaps`. Контроллер представления, встроенный в **AppMenu**, автоматически создает кнопки и привязывает их события к вызовам `SetSky` с соответствующими ключами.
1. Нажмите кнопку воспроизведения в редакторе Unity и авторизуйте подключение.
1. После подключения локальной среды выполнения к удаленному сеансу последовательно выберите **AppMenu —> Session Tools —> Remote Sky** (AppMenu —> Средства для работы с сеансом —> Удаленное небо), чтобы изучить различные параметры неба и узнать, как они влияют на **TestModel**.

### <a name="scene-lights"></a>Источники освещения сцены

Удаленные источники освещения сцены могут быть точечными, конусными и направленными. Как и созданная ранее секущая плоскость, эти источники освещения сцены являются удаленными сущностями с прикрепленными к ним компонентами. Важно учитывать, что при освещении удаленной сцены предпринимается попытка сопоставить их с освещением в локальной сцене. Эта стратегия не всегда возможна, так как многие приложения Unity для HoloLens 2 не используют физическую отрисовку для локально преобразованных для просмотра объектов. Однако мы можем в определенной мере имитировать упрощенное освещение, используемое в Unity по умолчанию.

1. Создайте скрипт с именем **RemoteLight** и замените его код приведенным ниже.

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    [RequireComponent(typeof(Light))]
    public class RemoteLight : BaseRemoteLight
    {
        public bool AutomaticallyCreate = true;

        private bool lightReady = false;
        public override bool LightReady 
        {
            get => lightReady;
            set
            {
                lightReady = value;
                LightReadyChanged?.Invoke(lightReady);
            }
        }

        private ObjectType remoteLightType = ObjectType.Invalid;
        public override ObjectType RemoteLightType => remoteLightType;

        public UnityBoolEvent OnLightReadyChanged;

        public override event Action<bool> LightReadyChanged;

        private Light localLight; //Unity Light

        private Entity lightEntity;
        private LightComponentBase remoteLightComponent; //Remote Rendering Light

        private void Awake()
        {
            localLight = GetComponent<Light>();
            switch (localLight.type)
            {
                case LightType.Directional:
                    remoteLightType = ObjectType.DirectionalLightComponent;
                    break;
                case LightType.Point:
                    remoteLightType = ObjectType.PointLightComponent;
                    break;
                case LightType.Spot:
                case LightType.Area:
                    //Not supported in tutorial
                case LightType.Disc:
                    // No direct analog in remote rendering
                    remoteLightType = ObjectType.Invalid;
                    break;
            }
        }

        public void Start()
        {
            // Hook up the event to the Unity event
            LightReadyChanged += (ready) => OnLightReadyChanged?.Invoke(ready);

            RemoteRenderingCoordinator.CoordinatorStateChange += RemoteRenderingCoordinator_CoordinatorStateChange;
            RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        public void OnDestroy()
        {
            lightEntity?.Destroy();
        }

        private void RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    if (AutomaticallyCreate)
                        CreateLight();
                    break;
                default:
                    DestroyLight();
                    break;
            }
        }

        public override void CreateLight()
        {
            if (remoteLightComponent != null)
                return; //Nothing to do!

            //Create a root object for the light
            if(lightEntity == null)
                lightEntity = RemoteRenderingCoordinator.CurrentSession.Connection.CreateEntity();

            //Bind the remote entity to this game object
            lightEntity.BindToUnityGameObject(this.gameObject);

            //Sync the transform of this object so we can move the light
            var syncComponent = this.gameObject.GetComponent<RemoteEntitySyncObject>();
            syncComponent.SyncEveryFrame = true;

            //Add a light to the entity
            switch (RemoteLightType)
            {
                case ObjectType.DirectionalLightComponent:
                    var remoteDirectional = RemoteRenderingCoordinator.CurrentSession.Connection.CreateComponent(ObjectType.DirectionalLightComponent, lightEntity) as DirectionalLightComponent;
                    //No additional properties
                    remoteLightComponent = remoteDirectional;
                    break;

                case ObjectType.PointLightComponent:
                    var remotePoint = RemoteRenderingCoordinator.CurrentSession.Connection.CreateComponent(ObjectType.PointLightComponent, lightEntity) as PointLightComponent;
                    remotePoint.Radius = 0;
                    remotePoint.Length = localLight.range;
                    //remotePoint.AttenuationCutoff = //No direct analog in Unity legacy lights
                    //remotePoint.ProjectedCubeMap = //No direct analog in Unity legacy lights

                    remoteLightComponent = remotePoint;
                    break;
                default:
                    LightReady = false;
                    return;
            }

            // Set the common values for all light types
            UpdateRemoteLightSettings();

            LightReady = true;
        }

        public override void UpdateRemoteLightSettings()
        {
            remoteLightComponent.Color = localLight.color.toRemote();
            remoteLightComponent.Intensity = localLight.intensity;
        }

        public override void DestroyLight()
        {
            if (remoteLightComponent == null)
                return; //Nothing to do!

            remoteLightComponent.Destroy();
            remoteLightComponent = null;
            LightReady = false;
        }

        [ContextMenu("Sync Remote Light Configuration")]
        public override void RecreateLight()
        {
            DestroyLight();
            CreateLight();
        }

        public override void SetIntensity(float intensity)
        {
            localLight.intensity = Mathf.Clamp(intensity, 0, 1);
            UpdateRemoteLightSettings();
        }

        public override void SetColor(Color color)
        {
            localLight.color = color;
            UpdateRemoteLightSettings();
        }
    }
    ```

    Этот скрипт создает удаленные источники освещения различных типов в зависимости от типа локального источника освещения Unity, к которому он присоединен. Удаленный источник освещения будет точной копией локального с точки зрения расположения, поворота, цвета и интенсивности. Там, где это возможно, удаленный источник света также задает дополнительную конфигурацию. Это соответствие не идеально, так как для источников света Unity применяется не PBR.

1. Найдите в сцене игровой объект **DirectionalLight**. Если вы удалили из сцены стандартный объект **DirectionalLight**, в верхней строке меню выберите *GameObject —> Light —> DirectionalLight*(Игровой объект —> Освещение —> DirectionalLight), чтобы создать источник освещения в сцене.

1. Выберите игровой объект **DirectionalLight** и с помощью кнопки **Add Component** (Добавить компонент) добавьте скрипт **RemoteLight**.

1. Поскольку этот скрипт реализует базовый класс `BaseRemoteLight`, для взаимодействия с удаленным источником освещения можно использовать предоставленный контроллер представления **AppMenu**. Последовательно выберите **AppMenu —> Session Tools —> Directional Light** (AppMenu —> Средства для работы с сеансом —> Направленный свет).

    > [!NOTE]
    > Для простоты пользовательский интерфейс в **AppMenu** ограничен одним направленным источником освещения. Однако при этом можно и даже рекомендуется добавить точечные источники освещения и присоединить к ним скрипт **RemoteLight**. Эти дополнительные источники можно изменить, отредактировав в редакторе свойства освещения Unity. Локальные изменения необходимо синхронизировать с удаленным освещением вручную, используя контекстное меню **RemoteLight** в инспекторе.
    >
    > ![Синхронизация с удаленным освещением вручную](./media/sync-remote-light.png)

1. Нажмите кнопку воспроизведения в редакторе Unity и авторизуйте подключение.

1. После подключения среды выполнения к удаленному сеансу разместите и наведите камеру (используйте клавиши W, A, S и D и перемещение мыши при нажатой правой кнопке), чтобы отобразить в представлении контроллер представления направленного освещения. 
1. Чтобы изменить свойства источника света, используйте контроллер представления удаленного освещения. Используя имитацию руки MRTK, возьмите и поверните направленный источник освещения (удерживайте клавишу CTRL для поворота), чтобы увидеть воздействие на освещение сцены.

    ![Направленное освещение](./media/directional-light-test.png)

## <a name="editing-materials"></a>Изменение материалов

Удаленно преобразованные для просмотра [материалы](../../../concepts/materials.md) можно изменить для предоставления дополнительных визуальных эффектов, точной настройки визуальных элементов преобразованных для просмотра моделей или предоставления дополнительной обратной связи пользователям. Существует множество способов и причин для изменения материала. Здесь мы покажем, как изменить цвет альбедо материала, а также шероховатость и металлические свойства материала PBR.

> [!NOTE]
> Если возможность или эффект можно реализовать с помощью **HierarchicalStateOverrideComponent**, зачастую вместо изменения материала лучше использовать именно этот компонент.

Мы создадим скрипт, который принимает целевую сущность и настраивает несколько объектов `OverrideMaterialProperty` для изменения свойств ее материала. Начнем с получения компонента [**MeshComponent**](../../../concepts/meshes.md#meshcomponent) целевой сущности, содержащего список материалов, используемых в сетке. Для простоты будем просто использовать первый найденный материал. Такая упрощенная стратегия может легко завершаться сбоем. Это зависит от того, как было создано содержимое. Поэтому для выбора соответствующего материала, вероятно, потребуется более сложный подход.

Из материала мы можем получить доступ к общим значениям, например альбедо. Сначала материалы необходимо привести к соответствующему типу, `PbrMaterial` или `ColorMaterial`, чтобы получить их значения, применяемые в методе **GetMaterialColor**. После получения ссылки на нужный материал достаточно просто задать значения, и служба "Удаленная отрисовка Azure" позаботится о синхронизации между свойствами локального и удаленного материалов.

1. Создайте скрипт с именем **EntityMaterialController** и замените все его содержимое следующим кодом:

    ```cs
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using System.Linq;
    using UnityEngine;
    // to prevent namespace conflicts
    using ARRMaterial = Microsoft.Azure.RemoteRendering.Material;

    public class EntityMaterialController : BaseEntityMaterialController
    {
        public override bool RevertOnEntityChange { get; set; } = true;

        public override OverrideMaterialProperty<Color> ColorOverride { get; set; }
        public override OverrideMaterialProperty<float> RoughnessOverride { get; set; }
        public override OverrideMaterialProperty<float> MetalnessOverride { get; set; }

        private Entity targetEntity;
        public override Entity TargetEntity
        {
            get => targetEntity;
            set
            {
                if (targetEntity != value)
                {
                    if (targetEntity != null && RevertOnEntityChange)
                    {
                        Revert();
                    }

                    targetEntity = value;
                    ConfigureTargetEntity();
                    TargetEntityChanged?.Invoke(value);
                }
            }
        }

        private ARRMaterial targetMaterial;
        private ARRMeshComponent meshComponent;

        public override event Action<Entity> TargetEntityChanged;
        public UnityRemoteEntityEvent OnTargetEntityChanged;

        public void Start()
        {
            // Forward events to Unity events
            TargetEntityChanged += (entity) => OnTargetEntityChanged?.Invoke(entity);

            // If there happens to be a remote RayCaster on this object, assume we should listen for events from it
            if (GetComponent<BaseRemoteRayCastPointerHandler>() != null)
                GetComponent<BaseRemoteRayCastPointerHandler>().RemoteEntityClicked += (entity) => TargetEntity = entity;
        }

        protected override void ConfigureTargetEntity()
        {
            //Get the Unity object, to get the sync object, to get the mesh component, to get the material.
            var targetEntityGameObject = TargetEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);

            var localSyncObject = targetEntityGameObject.GetComponent<RemoteEntitySyncObject>();
            meshComponent = targetEntityGameObject.GetComponent<ARRMeshComponent>();
            if (meshComponent == null)
            {
                var mesh = localSyncObject.Entity.FindComponentOfType<MeshComponent>();
                if (mesh != null)
                {
                    targetEntityGameObject.BindArrComponent<ARRMeshComponent>(mesh);
                    meshComponent = targetEntityGameObject.GetComponent<ARRMeshComponent>();
                }
            }

            meshComponent.enabled = true;

            targetMaterial = meshComponent.RemoteComponent.Mesh.Materials.FirstOrDefault();
            if (targetMaterial == default)
            {
                return;
            }

            ColorOverride = new OverrideMaterialProperty<Color>(
                GetMaterialColor(targetMaterial), //The original value
                targetMaterial, //The target material
                ApplyMaterialColor); //The action to take to apply the override

            //If the material is a PBR material, we can override some additional values
            if (targetMaterial.MaterialSubType == MaterialType.Pbr)
            {
                var firstPBRMaterial = (PbrMaterial)targetMaterial;

                RoughnessOverride = new OverrideMaterialProperty<float>(
                    firstPBRMaterial.Roughness, //The original value
                    targetMaterial, //The target material
                    ApplyRoughnessValue); //The action to take to apply the override

                MetalnessOverride = new OverrideMaterialProperty<float>(
                    firstPBRMaterial.Metalness, //The original value
                    targetMaterial, //The target material
                    ApplyMetalnessValue); //The action to take to apply the override
            }
            else //otherwise, ensure the overrides are cleared out from any previous entity
            {
                RoughnessOverride = null;
                MetalnessOverride = null;
            }
        }

        public override void Revert()
        {
            if (ColorOverride != null)
                ColorOverride.OverrideActive = false;

            if (RoughnessOverride != null)
                RoughnessOverride.OverrideActive = false;

            if (MetalnessOverride != null)
                MetalnessOverride.OverrideActive = false;
        }

        private Color GetMaterialColor(ARRMaterial material)
        {
            if (material == null)
                return default;

            if (material.MaterialSubType == MaterialType.Color)
                return ((ColorMaterial)material).AlbedoColor.toUnity();
            else
                return ((PbrMaterial)material).AlbedoColor.toUnity();
        }

        private void ApplyMaterialColor(ARRMaterial material, Color color)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Color)
                ((ColorMaterial)material).AlbedoColor = color.toRemoteColor4();
            else
                ((PbrMaterial)material).AlbedoColor = color.toRemoteColor4();
        }

        private void ApplyRoughnessValue(ARRMaterial material, float value)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Pbr) //Only PBR has Roughness
                ((PbrMaterial)material).Roughness = value;
        }

        private void ApplyMetalnessValue(ARRMaterial material, float value)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Pbr) //Only PBR has Metalness
                ((PbrMaterial)material).Metalness = value;
        }
    }
    ```

Тип `OverrideMaterialProperty` должен быть достаточно гибким, чтобы при необходимости можно было изменить несколько других значений материала. Тип `OverrideMaterialProperty` отслеживает состояние переопределения, сохраняет старое и новое значение и использует делегат, чтобы задать переопределение. В качестве примера рассмотрим `ColorOverride`.

```cs
ColorOverride = new OverrideMaterialProperty<Color>(
    GetMaterialColor(targetMaterial), //The original value
    targetMaterial, //The target material
    ApplyMaterialColor); //The action to take to apply the override
```

Этот компонент создает свойство `OverrideMaterialProperty`, в котором переопределение заключает в оболочку тип `Color`. Мы указываем текущий или исходный цвет во время создания переопределения. Мы также предоставляем для него материал Удаленной отрисовки Azure, с которым будут выполняться действия. Наконец, предоставляется делегат, который будет применять переопределение. Делегат — это метод, который принимает материал Удаленной отрисовки Azure и тип, заключаемый в оболочку переопределением. Этот метод играет важнейшую роль для понимания того, как Удаленная отрисовка Azure корректирует значения материала.

Чтобы этот подход сработал, `ColorOverride` использует метод `ApplyMaterialColor`.

```cs
private void ApplyMaterialColor(ARRMaterial material, Color color)
{
    if (material.MaterialSubType == MaterialType.Color)
        ((ColorMaterial)material).AlbedoColor = color.toRemoteColor4();
    else
        ((PbrMaterial)material).AlbedoColor = color.toRemoteColor4();
}
```

Этот код принимает материал и цвет. Он проверяет, к какому типу принадлежит материал, а затем приводит его к нужному типу для применения цвета.

`RoughnessOverride` и `MetalnessOverride` работают аналогично, используя методы `ApplyRoughnessValue` и `ApplyMetalnessValue` для выполнения своей работы.

Теперь выполним тестирование контроллера материалов.

1. Добавьте скрипт **EntityMaterialController** в игровой объект **TestModel**.
1. Нажмите кнопку воспроизведения в Unity, чтобы запустить сцену и подключиться к службе "Удаленная отрисовка Azure".
1. После подключения среды выполнения к удаленному сеансу и загрузки модели последовательно выберите **AppMenu —> Model Tools —> Edit Material** (AppMenu —> Средства работы с моделью —> Изменить материал).
1. Выберите сущность из модели, используя имитацию руки, чтобы щелкнуть **TestModel**.
1. Убедитесь, что контроллер представления материалов обновлен до целевой сущности. Для этого выберите **AppMenu —> Model Tools —> Edit Material** (AppMenu —> Средства работы с моделью —> Изменить материал).
1. Используйте контроллер представления материалов для настройки материала целевой сущности.

Так как мы изменяем только первый материал в сетке, его изменение может быть незаметным. Используйте иерархическое переопределение **SeeThrough**, чтобы определить, находится ли изменяемый вами материал в сетке.

![Пример изменения материала](./media/material-edit-example.png)

## <a name="next-steps"></a>Дальнейшие действия

Поздравляем! Теперь вы реализовали все основные функциональные возможности Удаленной отрисовки Azure. В следующей главе вы узнаете, как защищать Удаленную отрисовку Azure и Хранилище BLOB-объектов. В ней будут описаны первые шаги процесса выпуска коммерческого приложения, использующего Удаленную отрисовку Azure.

> [!div class="nextstepaction"]
> [Дальнейшие действия. Защита службы "Удаленная отрисовка Azure" и хранилища моделей](../security/security.md)