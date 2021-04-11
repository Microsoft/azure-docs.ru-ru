---
title: Просмотр модели удаленной отрисовки
description: Этот учебник посвящен базовой возможности службы "Удаленная отрисовка Azure", а точнее — просмотру модели, удаленно преобразованной для просмотра с помощью Azure
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: d8784bc4744e2d4beb6a72fdc0df0fd0b32346f9
ms.sourcegitcommit: 73d80a95e28618f5dfd719647ff37a8ab157a668
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/26/2021
ms.locfileid: "105605014"
---
# <a name="tutorial-viewing-a-remotely-rendered-model"></a>Руководство по Просмотр модели удаленной отрисовки

В этом руководстве описано следующее:

> [!div class="checklist"]
>
> * Подготовка экземпляра Удаленной отрисовки Azure
> * Создание и остановка сеанса отрисовки
> * Повторное использование имеющегося сеанса отрисовки
> * Подключение к сеансам и отключение от них
> * Загрузка моделей в сеанс отрисовки

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим руководством необходимо следующее:

* Подписка на Azure с оплатой по мере использования ([создать учетную запись](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/))
* Windows SDK 10.0.18362.0 [(скачать)](https://developer.microsoft.com/windows/downloads/windows-10-sdk).
* Последняя версия Visual Studio 2019 [(скачать)](https://visualstudio.microsoft.com/vs/older-downloads/).
* GIT [(скачать)](https://git-scm.com/downloads).
* Unity (поддерживаемые версии см. в статье [Системные требования](../../../overview/system-requirements.md#unity)).
* Средний уровень знания Unity и языка C# (например, умение создавать скрипты и объекты, использовать префабы, настраивать события Unity и т. д.).

## <a name="provision-an-azure-remote-rendering-arr-instance"></a>Подготовка экземпляра Удаленной отрисовки Azure

Чтобы получить доступ к службе "Удаленная отрисовка Azure", сначала необходимо [создать учетную запись](../../../how-tos/create-an-account.md#create-an-account).

## <a name="create-a-new-unity-project"></a>Создание проекта Unity

> [!TIP]
> [Репозиторий примеров Удаленной отрисовки Azure](https://github.com/Azure/azure-remote-rendering) содержит проект, в котором выполнены инструкции всех учебников. Его можно использовать для справки. Готовый проект Unity можно найти в папке *Unity\Tutorial-Complete*.

Создайте проект в Unity Hub.
В этом примере предполагается, что проект создается в папке с именем **RemoteRendering**.

:::image type="content" source="./media/unity-new-project.PNG" alt-text="Создание проекта Unity":::

## <a name="include-the-azure-remote-rendering-package"></a>Включение пакета Удаленной отрисовки Azure

[Следуйте инструкциям](../../../how-tos/unity/install-remote-rendering-unity-package.md) по добавлению пакета Удаленной отрисовки Azure к проекту Unity.


## <a name="configure-the-camera"></a>Настройка камеры

1. Выберите узел **Main Camera** (Основная камера).

1. Откройте контекстное меню, щелкнув правой кнопкой мыши компонент *Transform* и выбрав параметр **Reset** (Сброс).

    ![Сброс преобразования камеры](./media/camera-reset-transform.png)

1. Задайте свойству **Clear flags** (Очистить флаги) значение *Solid Color* (Сплошной цвет).

1. Для параметра **Background** (Фон) задайте значение *Black* (Черный, #000000) с полностью прозрачным (0) альфа-каналом (A).

    ![Цветовой круг](./media/color-wheel-black.png)

1. Задайте для свойства **Clipping Planes** (Плоскости отсечения) такие значения: *Near (Ближний) = 0,3* и *Far (Дальний) = 20*. Это означает, что при отрисовке будет обрезаться геометрический объект, расположенный ближе чем 30 см или дальше 20 м.

    ![Свойства камеры Unity](./media/camera-properties.png)

## <a name="adjust-the-project-settings"></a>Настройка параметров проекта

1. Выберите *Edit > Project Settings* (Изменить > Параметры проекта).
1. Выберите в меню со списком слева пункт **Quality** (Качество).
1. Измените настройки **Default Quality Level** (Уровень качества по умолчанию) для всех платформ на значение *Low* (Низкий). Этот параметр обеспечивает более эффективную отрисовку локального содержимого и не влияет на качество содержимого, преобразованного для просмотра удаленно.

    ![Изменение параметров качества в проекте](./media/settings-quality.png)

1. Выберите в меню со списком слева пункт **Graphics** (Графика).
1. Измените параметр **Scriptable Rendering Pipeline** (Конвейер отрисовки с поддержкой скриптов), задав значение *HybridRenderingPipeline*.\
    ![Снимок экрана, указывающий, где можно изменить параметр Scriptable Rendering Pipeline (Конвейер отрисовки с поддержкой скриптов) на HybridRenderingPipeline.](./media/settings-graphics-render-pipeline.png)\
    Иногда список доступных типов конвейеров в пользовательском интерфейсе не заполняется данными из пакетов. В этом случае необходимо перетащить ресурс *HybridRenderingPipeline* на поле вручную.\
    ![Изменение параметров графики проекта](./media/hybrid-rendering-pipeline.png)

    > [!NOTE]
    > Если не удается перетащить ресурс *HybridRenderingPipeline* в поле Render Pipeline Asset (Преобразовать для просмотра ресурс конвейера), например потому, что это поле не существует, убедитесь, что конфигурация пакета содержит пакет `com.unity.render-pipelines.universal`.

1. Выберите в меню со списком слева пункт **Player** (Проигрыватель).
1. Перейдите на вкладку с **параметрами универсальной платформы Windows**, обозначенную значком Windows.
1. Измените **XR Settings** (Параметры XR), чтобы настроить поддержку Windows Mixed Reality, как показано ниже.
    1. Установите флажок **Virtual Reality Supported** (Поддержка виртуальной реальности).
    1. Нажмите кнопку "+" и добавьте **Windows Mixed Reality**.
    1. Задайте для параметра **Depth Format** (Формат глубины) значение *16-Bit Depth* (Глубина 16 бит).
    1. Убедитесь, что флажок **Depth Buffer Sharing** (Общий доступ к буферу глубины) установлен.
    1. Задайте для параметра **Stereo Rendering Mode** (Режим стерео отрисовки) значение *Single Pass Instanced* (Однопроходный экземпляр).

    ![Параметры проигрывателя](./media/xr-player-settings.png)

1. В этом же окне над **XR Settings** (Параметры XR) разверните меню **Publishing Settings** (Параметры публикации).
1. Прокрутите содержимое вниз до **Capabilities** (Возможности) и выберите:
    * **InternetClient**;
    * **InternetClientServer**;
    * **SpatialPerception**;
    * **PrivateNetworkClientServer** (*необязательно*). Выберите этот параметр, если вы хотите подключить к устройству удаленный отладчик Unity.

1. В разделе **Supported Device Families** (Поддерживаемые семейства устройств) установите флажки **Holographic** (Голографические) и **Desktop** (Компьютер).
1. Закройте или закрепите панель **Project Settings** (Параметры проекта).
1. Откройте *File —> Build Settings* (Файл —> Параметры сборки).
1. Выберите **Universal Windows Platform** (Универсальная платформа Windows).
1. Настройте параметры так, как указанно ниже.
1. Нажмите кнопку **Switch Platform** (Сменить платформу).
![Параметры сборки](./media/build-settings.png)
1. После смены платформы в Unity закройте панель сборки.

## <a name="validate-project-setup"></a>Проверка настроек проекта

Выполните следующие действия, чтобы проверить правильность параметров проекта:

1. Выберите запись **ValidateProject** в меню **RemoteRendering** на панели инструментов редактора Unity.
1. Проверьте, нет ли в окне **ValidateProject** сообщений об ошибках, и исправьте параметры проекта в случае необходимости.

    ![Проверка проекта редактора Unity](./media/remote-render-unity-validation.png)

## <a name="create-a-script-to-coordinate-azure-remote-rendering-connection-and-state"></a>Создание скрипта для координирования подключения и состояния службы "Удаленная отрисовка Azure"

Отображение удаленно преобразованных для просмотра моделей состоит из четырех основных этапов, продемонстрированных на блок-схеме ниже. Все этапы должны выполняться по порядку. Следующий шаг — создать скрипт, который будет управлять состоянием приложения и выполнять все необходимые этапы.

![Стек Удаленной отрисовки Azure 0](./media/remote-render-stack-0.png)

1. В области *Project* (Проект) в разделе **Assets** (Ресурсы) создайте новую папку с именем *RemoteRenderingCore*. Затем внутри *RemoteRenderingCore* создайте другую папку с именем *Scripts*.

1. Создайте [скрипт C#](https://docs.unity3d.com/Manual/CreatingAndUsingScripts.html) с именем **RemoteRenderingCoordinator**.
Проект должен выглядеть так:

    ![Иерархия проекта](./media/project-structure.png)

    Этот скрипт координатора будет отслеживать состояние удаленной отрисовки и управлять им. Обратите внимание, что часть данного кода используется для поддержания состояния, предоставления функциональных возможностей другим компонентам, активации событий и хранения данных приложения, которые не связаны *непосредственно* с Удаленной отрисовкой Azure. Используйте приведенный ниже код в качестве отправной точки. Реализацию конкретного кода Удаленной отрисовки Azure мы рассмотрим далее в этом учебнике.

1. Откройте скрипт **RemoteRenderingCoordinator** в редакторе кода и замените все его содержимое следующим кодом:

```cs
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License. See LICENSE in the project root for license information.

using Microsoft.Azure.RemoteRendering;
using Microsoft.Azure.RemoteRendering.Unity;
using System;
using System.Linq;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;

#if UNITY_WSA
using UnityEngine.XR.WSA;
#endif

/// <summary>
/// Remote Rendering Coordinator is the controller for all Remote Rendering operations.
/// </summary>

// Require the GameObject with a RemoteRenderingCoordinator to also have the ARRServiceUnity component
[RequireComponent(typeof(ARRServiceUnity))]
public class RemoteRenderingCoordinator : MonoBehaviour
{
    public enum RemoteRenderingState
    {
        NotSet,
        NotInitialized,
        NotAuthorized,
        NoSession,
        ConnectingToExistingRemoteSession,
        ConnectingToNewRemoteSession,
        RemoteSessionReady,
        ConnectingToRuntime,
        RuntimeConnected
    }

    public static RemoteRenderingCoordinator instance;

    // AccountDomain must be '<region>.mixedreality.azure.com' - if no '<region>' is specified, connections will fail
    // The list of regions is available at https://docs.microsoft.com/azure/remote-rendering/reference/regions
    [SerializeField]
    private string accountDomain = "westus2.mixedreality.azure.com";
    public string AccountDomain
    {
        get => accountDomain.Trim();
        set => accountDomain = value;
    }

    [Header("Development Account Credentials")]
    [SerializeField]
    private string accountId = "<enter your account id here>";
    public string AccountId {
        get => accountId.Trim();
        set => accountId = value;
    }

    [SerializeField]
    private string accountAuthenticationDomain = "<enter your account authentication domain here>";
    public string AccountAuthenticationDomain
    {
        get => accountAuthenticationDomain.Trim();
        set => accountAuthenticationDomain = value;
    }   

    [SerializeField]
    private string accountKey = "<enter your account key here>";
    public string AccountKey {
        get => accountKey.Trim();
        set => accountKey = value;
    }

    // These settings are important. All three should be set as low as possible, while maintaining a good user experience
    // See the documentation around session management and the technical differences in session VM size
    [Header("New Session Defaults")]

    public RenderingSessionVmSize renderingSessionVmSize = RenderingSessionVmSize.Standard;
    public uint maxLeaseHours = 0;
    public uint maxLeaseMinutes = 20;

    [Header("Other Configuration")]

    [Tooltip("If you have a known active SessionID, you can fill it in here before connecting")]
    [SerializeField]
    private string sessionIDOverride;
    public string SessionIDOverride {
        get => sessionIDOverride.Trim();
        set => sessionIDOverride = value;
    }

    // When Automatic Mode is true, the coordinator will attempt to automatically proceed through the process of connecting and loading a model
    public bool automaticMode = true;

    public event Action RequestingAuthorization;
    public UnityEvent OnRequestingAuthorization = new UnityEvent();

    public event Action AuthorizedChanged;
    public UnityEvent OnAuthorizationChanged = new UnityEvent();
    private bool authorized;
    public bool Authorized
    {
        get => authorized;
        set
        {
            if (value == true) //This is a one-way value, once we're authorized it lasts until the app is shutdown.
            {
                authorized = value;
                AuthorizedChanged?.Invoke();
            }
        }
    }

    public delegate Task<SessionConfiguration> AccountInfoGetter();

    public static AccountInfoGetter ARRCredentialGetter
    {
        private get;
        set;
    }

    private RemoteRenderingState currentCoordinatorState = RemoteRenderingState.NotSet;
    public RemoteRenderingState CurrentCoordinatorState
    {
        get => currentCoordinatorState;
        private set
        {
            if (currentCoordinatorState != value)
            {
                currentCoordinatorState = value;
                Debug.Log($"State changed to: {currentCoordinatorState}");
                CoordinatorStateChange?.Invoke(currentCoordinatorState);
            }
        }
    }

    public static event Action<RemoteRenderingState> CoordinatorStateChange;

    public static RenderingSession CurrentSession => instance?.ARRSessionService?.CurrentActiveSession;

    private ARRServiceUnity arrSessionService;

    private ARRServiceUnity ARRSessionService
    {
        get
        {
            if (arrSessionService == null)
                arrSessionService = GetComponent<ARRServiceUnity>();
            return arrSessionService;
        }
    }

    private async Task<SessionConfiguration> GetDevelopmentCredentials()
    {
        Debug.LogWarning("Using development credentials! Not recommended for production.");
        return await Task.FromResult(new SessionConfiguration(AccountAuthenticationDomain, AccountDomain, AccountId, AccountKey));
    }

    /// <summary>
    /// Keep the last used SessionID, when launching, connect to this session if its available
    /// </summary>
    private string LastUsedSessionID
    {
        get
        {
            if (!string.IsNullOrEmpty(SessionIDOverride))
                return SessionIDOverride;

            if (PlayerPrefs.HasKey("LastUsedSessionID"))
                return PlayerPrefs.GetString("LastUsedSessionID");
            else
                return null;
        }
        set
        {
            PlayerPrefs.SetString("LastUsedSessionID", value);
        }
    }

    public void Awake()
    {
        //Forward events to Unity events
        RequestingAuthorization += () => OnRequestingAuthorization?.Invoke();
        AuthorizedChanged += () => OnAuthorizationChanged?.Invoke();

        //Attach to event
        AuthorizedChanged += RemoteRenderingCoordinator_AuthorizedChanged;

        if (instance == null)
            instance = this;
        else
            Destroy(this);

        CoordinatorStateChange += AutomaticMode;

        CurrentCoordinatorState = RemoteRenderingState.NotInitialized;
    }

    private void RemoteRenderingCoordinator_AuthorizedChanged()
    {
        if (CurrentCoordinatorState != RemoteRenderingState.NotAuthorized)
            return; //This isn't valid from any other state than NotAuthorized


        //We just became authorized to connect to Azure
        InitializeSessionService();
    }

    /// <summary>
    /// Automatic mode attempts to automatically progress through the connection and loading steps. Doesn't handle error states.
    /// </summary>
    /// <param name="currentState">The current state</param>
    private async void AutomaticMode(RemoteRenderingState currentState)
    {
        if (!automaticMode)
            return;

        //Add a small delay for visual effect
        await Task.Delay(1500);
        switch (currentState)
        {
            case RemoteRenderingState.NotInitialized:
                InitializeARR();
                break;
            case RemoteRenderingState.NotAuthorized:
                RequestAuthorization();
                break;
            case RemoteRenderingState.NoSession:
                JoinRemoteSession();
                break;
            case RemoteRenderingState.RemoteSessionReady:
                ConnectRuntimeToRemoteSession();
                break;
        }
    }

    /// <summary>
    /// Initializes ARR, associating the main camera
    /// Note: This must be called on the main Unity thread
    /// </summary>
    public void InitializeARR()
    {
        //Implement me
    }

    /// <summary>
    /// Create a new remote session manager
    /// If the ARRCredentialGetter is set, use it as it, otherwise use the development credentials 
    /// </summary>
    public async void InitializeSessionService()
    {
        //Implement me
    }

    /// <summary>
    /// Trigger the event for checking authorization, respond to this event by prompting the user for authentication
    /// If authorized, set Authorized = true
    /// </summary>
    public void RequestAuthorization()
    {
        RequestingAuthorization?.Invoke();
    }

    public void BypassAuthorization()
    {
        Authorized = true;
    }

    /// <summary>
    /// Attempts to join an existing session or start a new session
    /// </summary>
    public async void JoinRemoteSession()
    {
        //Implement me
    }

    public void StopRemoteSession()
    {
        //Implement me
    }

    private async Task<bool> IsSessionAvailable(string sessionID)
    {
        bool sessionAvailable = false;
        try
        {
            RenderingSessionPropertiesArrayResult result = await ARRSessionService.Client.GetCurrentRenderingSessionsAsync();
            if (result.ErrorCode == Result.Success)
            {
                RenderingSessionProperties[] properties = result.SessionProperties;
                if (properties != null)
                {
                    sessionAvailable = properties.Any(x => x.Id == sessionID && (x.Status == RenderingSessionStatus.Ready || x.Status == RenderingSessionStatus.Starting));
                }
            }
            else
            {
                Debug.LogError($"Failed to get current rendering sessions. Error: {result.Context.ErrorMessage}");
            }
        }
        catch (RRException ex)
        {
            Debug.LogError($"Failed to get current rendering sessions. Error: {ex.Message}");
        }
        return sessionAvailable;
    }

    /// <summary>
    /// Connects the local runtime to the current active session, if there's a session available
    /// </summary>
    public void ConnectRuntimeToRemoteSession()
    {
        //Implement me
    }

    public void DisconnectRuntimeFromRemoteSession()
    {
        //Implement me
    }

    /// <summary>
    /// The session must have its runtime pump updated.
    /// The Connection.Update() will push messages to the server, receive messages, and update the frame-buffer with the remotely rendered content.
    /// </summary>
    private void LateUpdate()
    {
        ARRSessionService?.CurrentActiveSession?.Connection?.Update();
    }

    /// <summary>
    /// Loads a model into the remote session for rendering
    /// </summary>
    /// <param name="modelPath">The model's path</param>
    /// <param name="progress">A call back method that accepts a float progress value [0->1]</param>
    /// <param name="parent">The parent Transform for this remote entity</param>
    /// <returns>An awaitable Remote Rendering Entity</returns>
    public async Task<Entity> LoadModel(string modelPath, Transform parent = null, Action<float> progress = null)
    {
        //Implement me
        return null;
    }

    private async void OnRemoteSessionStatusChanged(ARRServiceUnity caller, RenderingSession session)
    {
        var properties = await session.GetPropertiesAsync();

        switch (properties.SessionProperties.Status)
        {
            case RenderingSessionStatus.Error:
            case RenderingSessionStatus.Expired:
            case RenderingSessionStatus.Stopped:
            case RenderingSessionStatus.Unknown:
                CurrentCoordinatorState = RemoteRenderingState.NoSession;
                break;
            case RenderingSessionStatus.Starting:
                CurrentCoordinatorState = RemoteRenderingState.ConnectingToNewRemoteSession;
                break;
            case RenderingSessionStatus.Ready:
                CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
                break;
        }
    }

    private void OnLocalRuntimeStatusChanged(ConnectionStatus status, Result error)
    {
        switch (status)
        {
            case ConnectionStatus.Connected:
                CurrentCoordinatorState = RemoteRenderingState.RuntimeConnected;
                break;
            case ConnectionStatus.Connecting:
                CurrentCoordinatorState = RemoteRenderingState.ConnectingToRuntime;
                break;
            case ConnectionStatus.Disconnected:
                CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
                break;
        }
    }
}
```

## <a name="create-the-azure-remote-rendering-gameobject"></a>Создание игрового объекта Удаленной отрисовки Azure

Координатор удаленной отрисовки и его обязательный скрипт (*ARRServiceUnity*) реализуют классы MonoBehaviour, которые должны быть присоединены к игровому объекту в сцене. Скрипт *ARRServiceUnity* предоставляется службой "Удаленная отрисовка Azure", реализуя многие из ее функциональных возможностей для подключения к удаленным сеансам и управления ими.

1. Создайте игровой объект в сцене, нажав клавиши CTRL+SHIFT+N или выбрав *GameObject —> Create Empty* (Игровой объект —> Создать пустой), и назовите его **RemoteRenderingCoordinator**.
1. Добавьте скрипт *RemoteRenderingCoordinator* в игровой объект **RemoteRenderingCoordinator**.\
![Добавление компонента RemoteRenderingCoordinator](./media/add-coordinator-script.png)
1. Убедитесь, что скрипт *ARRServiceUnity*, который отображается в инспекторе как *служба*, автоматически добавлен в игровой объект. Интересно, почему так должно быть? Это следствие размещения `[RequireComponent(typeof(ARRServiceUnity))]` в верхней части скрипта **RemoteRenderingCoordinator**.
1. Добавьте в скрипт координатора учетные данные Удаленной отрисовки Azure, домен аутентификации учетной записи и домен учетной записи.\
![Добавление учетных данных](./media/configure-coordinator-script.png)

## <a name="initialize-azure-remote-rendering"></a>Инициализация Удаленной отрисовки Azure

Теперь, когда у нас есть платформа для координатора, мы реализуем каждый из четырех этапов, начиная с **инициализации Удаленной отрисовки**.

![Стек Удаленной отрисовки Azure 1](./media/remote-render-stack-1.png)

При **инициализации** служба "Удаленная отрисовка Azure" получает сведения о том, какой объект "Камера" следует использовать для отрисовки, и переводит конечный автомат в состояние **NotAuthorized**. Это означает, что он инициализирован, но еще не авторизован для подключения к сеансу. Поскольку запуск сеанса Удаленной отрисовки Azure сопряжен с затратами, нам необходимо убедиться, что пользователь хочет продолжить работу.

При переходе в состояние **NotAuthorized** вызывается метод **CheckAuthorization**, который инициирует событие **RequestingAuthorization** и определяет, какие учетные данные нужно использовать (**AccountInfo** определяется в верхней части класса и использует учетные данные, определенные с помощью инспектора Unity на предыдущем шаге).

   > [!NOTE]
   > Удаленная отрисовка Azure не поддерживает повторную компиляцию во время выполнения. Изменение и сохранение скрипта в режиме воспроизведения может привести к зависанию Unity, что потребует принудительного завершения работы Unity через диспетчер задач. Перед редактированием скриптов всегда проверяйте, остановлен ли режим воспроизведения.

1. Замените содержимое **InitializeARR** и **InitializeSessionService** готовым кодом, приведенным ниже.

 ```cs
/// <summary>
/// Initializes ARR, associating the main camera
/// Note: This must be called on the main Unity thread
/// </summary>
public void InitializeARR()
{
    RemoteManagerUnity.InitializeManager(new RemoteUnityClientInit(Camera.main));

    CurrentCoordinatorState = RemoteRenderingState.NotAuthorized;
}

/// <summary>
/// Create a new remote session manager
/// If the ARRCredentialGetter is set, use it as it, otherwise use the development credentials 
/// </summary>
public async void InitializeSessionService()
{
    if (ARRCredentialGetter == null)
        ARRCredentialGetter = GetDevelopmentCredentials;

    var accountInfo = await ARRCredentialGetter.Invoke();

    ARRSessionService.OnSessionStatusChanged += OnRemoteSessionStatusChanged;

    ARRSessionService.Initialize(accountInfo);

    CurrentCoordinatorState = RemoteRenderingState.NoSession;
}
```

Для перехода из состояния **NotAuthorized** в состояние **NoSession** мы обычно предоставляем пользователю модальное диалоговое окно, чтобы он мог сделать выбор (описание этого процесса приведено в другой главе). Сейчас мы автоматически обойдем проверку авторизации, вызвав **ByPassAuthentication** как только будет активировано событие **RequestingAuthorization**.

1. Выберите игровой объект **RemoteRenderingCoordinator** и найдите событие Unity **OnRequestingAuthorization**, доступное в инспекторе компонента **RemoteRenderingCoordinator**.

1. Добавьте новое событие, нажав кнопку "+" в правом нижнем углу.
1. Перетащите компонент на его собственное событие, чтобы он ссылался на себя.\
![Обход проверки подлинности](./media/bypass-authorization-add-event.png)\
1. В раскрывающемся списке выберите **RemoteRenderingCoordinator —> BypassAuthorization**.\
![Снимок экрана, на котором показан выбранный параметр RemoteRenderingCoordinator.BypassAuthorization.](./media/bypass-authorization-event.png)

## <a name="create-or-join-a-remote-session"></a>Создание удаленного сеанса или присоединение к нему

Второй этап — создание сеанса Удаленной отрисовки или присоединение к нему (дополнительные сведения см. в статье [Сеансы Удаленной отрисовки](../../../concepts/sessions.md)).

![Стек Удаленной отрисовки Azure 2](./media/remote-render-stack-2.png)

Удаленный сеанс — это место, где модели преобразовываются для просмотра. Метод **JoinRemoteSession( )** попытается подключиться к существующему сеансу, который отслеживается с помощью свойства **LastUsedSessionID**, или при наличии назначенного активного идентификатора сеанса для **SessionIDOverride**. Интерфейс **SessionIDOverride** предназначен только для отладки. Его следует использовать только в случае, если известно, что сеанс существует и необходимо подключиться к нему явно.

Если сеансы недоступны, будет создан новый сеанс. Создание сеанса, однако, требует много времени. Поэтому пытаться создавать сеансы следует только в случае необходимости. По возможности необходимо использовать их повторно (дополнительные сведения об управлении сеансами см. в [статье о готовности к коммерческому использованию в разделе, посвященном созданию пула сеансов, планированию и рекомендациям](../commercial-ready/commercial-ready.md#fast-startup-time-strategies).

> [!TIP]
> **StopRemoteSession()** завершит активный сеанс. Во избежание ненужных расходов следует всегда останавливать сеансы, если они больше не нужны.

Теперь конечный автомат перейдет в состояние **ConnectingToNewRemoteSession** или **ConnectingToExistingRemoteSession** в зависимости от доступных сеансов. При открытии существующего или создании нового сеанса будет активировано событие **ARRSessionService.OnSessionStatusChanged**, выполняющее метод **OnRemoteSessionStatusChanged**. В идеале это приведет к тому, что конечный автомат перейдет в состояние **RemoteSessionReady**.

1. Чтобы присоединиться к новому сеансу, измените код, заменив методы **JoinRemoteSession( )** и **StopRemoteSession( )** приведенными ниже готовыми примерами.

```cs
/// <summary>
/// Attempts to join an existing session or start a new session
/// </summary>
public async void JoinRemoteSession()
{
    //If there's a session available that previously belonged to us, and it's ready, use it. Otherwise start a new session.
    RenderingSessionProperties joinResult;
    if (await IsSessionAvailable(LastUsedSessionID))
    {
        CurrentCoordinatorState = RemoteRenderingState.ConnectingToExistingRemoteSession;
        joinResult = await ARRSessionService.OpenSession(LastUsedSessionID);
    }
    else
    {
        CurrentCoordinatorState = RemoteRenderingState.ConnectingToNewRemoteSession;
        joinResult = await ARRSessionService.StartSession(new RenderingSessionCreationOptions(renderingSessionVmSize, (int)maxLeaseHours, (int)maxLeaseMinutes));
    }

    if (joinResult.Status == RenderingSessionStatus.Ready || joinResult.Status == RenderingSessionStatus.Starting)
    {
        LastUsedSessionID = joinResult.Id;
    }
    else
    {
        //The session should be ready or starting, if it's not, something went wrong
        await ARRSessionService.StopSession();
        if(LastUsedSessionID == SessionIDOverride)
            SessionIDOverride = "";
        CurrentCoordinatorState = RemoteRenderingState.NoSession;
    }
}

public void StopRemoteSession()
{
    if (ARRSessionService.CurrentActiveSession != null)
    {
        ARRSessionService.CurrentActiveSession.StopAsync();
    }
}
```

Если вы хотите сэкономить время, повторно использовав сеансы, отключите параметр **Auto-Stop Session** (Автоматическая остановка сеанса) в компоненте *ARRServiceUnity*. Помните, что при этом сеансы продолжают выполняться, даже если к ним ничто не подключено. Сеанс может выполняться до тех пор, пока сервер не завершит его по истечении времени *MaxLeaseTime*. Значение *MaxLeaseTime* можно изменить в координаторе Удаленной отрисовки в разделе *New Session Defaults* (Значения по умолчанию для нового сеанса). С другой стороны, если при отключении будет завершаться каждый сеанс, вам всякий раз придется ждать, пока будет запущен новый сеанс, а этот процесс может быть довольно длительным.

> [!NOTE]
> Остановка сеанса произойдет мгновенно, и ее нельзя будет отменить. После остановки необходимо создать сеанс с теми же затратами на запуск.

## <a name="connect-the-local-runtime-to-the-remote-session"></a>Подключение локальной среды выполнения к удаленному сеансу

Теперь приложению необходимо подключить свою локальную среду выполнения к удаленному сеансу.

![Стек Удаленной отрисовки Azure 3](./media/remote-render-stack-3.png)

Приложение также должно ожидать передачи данных о событиях, связанных с подключением между средой выполнения и текущим сеансом. Эти изменения состояния обрабатываются в **OnLocalRuntimeStatusChanged**. Этот код сменит текущее состояние на **ConnectingToRuntime**. После подключения в **OnLocalRuntimeStatusChanged** состояние сменится на **RuntimeConnected**. Подключение к среде выполнения — это последнее состояние, с которым имеет дело координатор. Это означает, что для приложения выполнены все общие настройки и оно готово к началу работы в конкретном сеансе (загрузке моделей и преобразованию их для просмотра).

 1. Замените методы **ConnectRuntimeToRemoteSession( )** и **DisconnectRuntimeFromRemoteSession( )** приведенными ниже готовыми версиями.
 1. Обратите особое внимание на метод Unity под названием **LateUpdate** и на тот факт, что он обновляет текущий активный сеанс. Это позволяет текущему сеансу отправлять и получать сообщения и обновлять буфер кадров кадрами, полученными из удаленного сеанса. Очень важно обеспечить правильную работу Удаленной отрисовки Azure.

```cs
/// <summary>
/// Connects the local runtime to the current active session, if there's a session available
/// </summary>
public void ConnectRuntimeToRemoteSession()
{
    if (ARRSessionService == null || ARRSessionService.CurrentActiveSession == null)
    {
        Debug.LogError("Not ready to connect runtime");
        return;
    }

    //Connect the local runtime to the currently connected session
    //This session is set when connecting to a new or existing session

    ARRSessionService.CurrentActiveSession.ConnectionStatusChanged += OnLocalRuntimeStatusChanged;
    ARRSessionService.CurrentActiveSession.ConnectAsync(new RendererInitOptions());
    CurrentCoordinatorState = RemoteRenderingState.ConnectingToRuntime;
}

public void DisconnectRuntimeFromRemoteSession()
{
    if (ARRSessionService == null || ARRSessionService.CurrentActiveSession == null || ARRSessionService.CurrentActiveSession.ConnectionStatus != ConnectionStatus.Connected)
    {
        Debug.LogError("Runtime not connected!");
        return;
    }

    ARRSessionService.CurrentActiveSession.Disconnect();
    ARRSessionService.CurrentActiveSession.ConnectionStatusChanged -= OnLocalRuntimeStatusChanged;
    CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
}

/// <summary>
/// The session must have its runtime pump updated.
/// The Connection.Update() will push messages to the server, receive messages, and update the frame-buffer with the remotely rendered content.
/// </summary>
private void LateUpdate()
{
    ARRSessionService?.CurrentActiveSession?.Connection?.Update();
}
```

> [!NOTE]
> Подключение локальной среды выполнения к удаленному сеансу зависит от метода **Update**, вызываемого для текущего активного сеанса. Если вы обнаружите, что приложение никогда не переходит в состояние **ConnectingToRuntime**, убедитесь, что вы регулярно вызываете **Update** для активного сеанса.

## <a name="load-a-model"></a>Загрузка модели

Подготовив необходимые базовые элементы, вы можете загрузить модель в удаленный сеанс и начать получать кадры.

![Схема потока процесса подготовки к загрузке и просмотру модели.](./media/remote-render-stack-4.png)

Метод **LoadModel** должен принимать путь к модели, обработчик выполнения и родительское преобразование. Эти аргументы будут использоваться для загрузки модели в удаленный сеанс, предоставления пользователю актуальных сведений о ходе загрузки и ориентации удаленно преобразованной для просмотра модели на основе родительского преобразования.

1. Полностью замените метод **LoadModel** следующим кодом:

    ```cs
    /// <summary>
    /// Loads a model into the remote session for rendering
    /// </summary>
    /// <param name="modelName">The model's path</param>
    /// <param name="parent">The parent Transform for this remote entity</param>
    /// <param name="progress">A call back method that accepts a float progress value [0->1]</param>
    /// <returns>An awaitable Remote Rendering Entity</returns>
    public async Task<Entity> LoadModel(string modelPath, Transform parent = null, Action<float> progress = null)
    {
        //Create a root object to parent a loaded model to
        var modelEntity = ARRSessionService.CurrentActiveSession.Connection.CreateEntity();

        //Get the game object representation of this entity
        var modelGameObject = modelEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);

        //Ensure the entity will sync its transform with the server
        var sync = modelGameObject.GetComponent<RemoteEntitySyncObject>();
        sync.SyncEveryFrame = true;

        //Parent the new object under the defined parent
        if (parent != null)
        {
            modelGameObject.transform.SetParent(parent, false);
            modelGameObject.name = parent.name + "_Entity";
        }

    #if UNITY_WSA
        //Anchor the model in the world, prefer anchoring parent if there is one
        if (parent != null)
        {
            parent.gameObject.AddComponent<WorldAnchor>();
        }
        else
        {
            modelGameObject.AddComponent<WorldAnchor>();
        }
    #endif

        //Load a model that will be parented to the entity
        var loadModelParams = new LoadModelFromSasOptions(modelPath, modelEntity);
        var loadModelAsync = ARRSessionService.CurrentActiveSession.Connection.LoadModelFromSasAsync(loadModelParams, progress);
        var result = await loadModelAsync;
        return modelEntity;
    }
    ```

Приведенный выше код выполняет следующие действия:

1. Создает [удаленную сущность](../../../concepts/entities.md).
1. Создайте локальный игровой объект для представления удаленной сущности.
1. Настраивает локальный игровой объект для синхронизации своего состояния (т. е. преобразования) с каждым кадром удаленной сущности.
1. Задает имя и добавляет [**WorldAnchor**](https://docs.unity3d.com/550/Documentation/ScriptReference/VR.WSA.WorldAnchor.html), чтобы способствовать стабилизации.
1. Загружает данные модели из Хранилища BLOB-объектов в удаленную сущность.
1. Возвращает родительскую сущность для ссылки на нее в дальнейшем.

## <a name="view-the-test-model"></a>Просмотр тестовой модели

Теперь у нас есть весь код, необходимый для просмотра удаленно преобразованной для просмотра модели — все четыре необходимых этапа удаленной отрисовки завершены. Осталось добавить немного кода для запуска процесса загрузки модели.

![Стек Удаленной отрисовки Azure 4](./media/remote-render-stack-5.png)

1. Добавьте следующий код в класс **RemoteRenderingCoordinator**. Его можно поместить под методом **LoadModel**.

    ```cs
    private bool loadingTestModel = false;
    [ContextMenu("Load Test Model")]
    public async void LoadTestModel()
    {
        if(CurrentCoordinatorState != RemoteRenderingState.RuntimeConnected)
        {
            Debug.LogError("Please wait for the runtime to connect before loading the test model. Try again later.");
            return;
        }
        if(loadingTestModel)
        {
            Debug.Log("Test model already loading or loaded!");
            return;
        }
        loadingTestModel = true;
    
        // Create a parent object to use for positioning
        GameObject testParent = new GameObject("TestModelParent");
        testParent.transform.position = new Vector3(0f, 0f, 3f);
    
        // The 'built in engine path' is a special path that references a test model built into Azure Remote Rendering.
        await LoadModel("builtin://Engine", testParent.transform, (progressValue) => Debug.Log($"Loading Test Model progress: {Math.Round(progressValue * 100, 2)}%"));
    }
    ```
    
    Этот код создает игровой объект, который будет служить родительским для тестовой модели. Затем вызовите метод **LoadModel** для загрузки модели "builtin://Engine". Это встроенный ресурс службы "Удаленная отрисовка Azure", предназначенный для тестирования отрисовки.

1. Сохраните код.
1. Нажмите кнопку воспроизведения в редакторе Unity, чтобы начать процесс подключения к Удаленной отрисовке Azure и создания нового сеанса.
1. Вы мало что увидите в игровом представлении, но в консоли будет отображаться изменение состояния приложения. Скорее всего, оно перейдет в состояние `ConnectingToNewRemoteSession` и, возможно, будет оставаться в нем до пяти минут.
1. Выберите игровой объект **RemoteRenderingCoordinator**, чтобы просмотреть в инспекторе вложенные в него скрипты. Следите за обновлением компонента **Service** в процессе его инициализации и подключения.
1. Дождитесь перехода в состояние **RuntimeConnected**, следя за выходными данными в консоли.
1. После подключения среды выполнения щелкните в инспекторе правой кнопкой мыши **RemoteRenderingCoordinator**, чтобы открыть контекстное меню. Затем выберите в нем пункт **Load Test Model** (Загрузить тестовую модель), добавленный частью `[ContextMenu("Load Test Model")]` приведенного выше кода.

    ![Загрузка из контекстного меню](./media/load-test-model.png)

1. Просмотрите в консоли выходные данные **ProgressHandler**, переданные в метод **LoadModel**.
1. Просмотрите модель, удаленно преобразованную для просмотра.

> [!NOTE]
> Удаленная модель никогда не отображается в представлении сцены. Она видна только в игровом представлении. Это связано с тем, что Удаленная отрисовка Azure удаленно преобразовывает кадры для просмотра (точнее, для перспективы камеры игрового представления) и не распознает камеру редактора (используемую для отрисовки представления сцены).

## <a name="next-steps"></a>Дальнейшие действия

![Модель загружена](./media/test-model-rendered.png)

Поздравляем! Вы создали базовое приложение, с помощью которого можно просматривать удаленно преобразованные для просмотра модели с помощью Удаленной отрисовки Azure. В следующем учебнике мы расскажем о том, как интегрировать MRTK и импортировать собственные модели.

> [!div class="nextstepaction"]
> [Дальнейшие действия. Интерфейсы и пользовательские модели](../custom-models/custom-models.md)
