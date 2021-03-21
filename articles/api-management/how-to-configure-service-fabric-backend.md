---
title: Настройка серверной части Service Fabric в службе управления API Azure | Документация Майкрософт
description: Как создать серверную часть службы Service Fabric в службе управления API Azure с помощью портал Azure
services: api-management
documentationcenter: ''
author: dlepow
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 01/29/2021
ms.author: apimpm
ms.openlocfilehash: f6474dbd02c501612b951ddae490385a5d843fbf
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99500634"
---
# <a name="set-up-a-service-fabric-backend-in-api-management-using-the-azure-portal"></a>Настройка Service Fabric серверной части в службе управления API с помощью портал Azure

В этой статье показано, как настроить службу [Service Fabric](../service-fabric/service-fabric-api-management-overview.md) в качестве настраиваемой серверной части API с помощью портал Azure. В демонстрационных целях показано, как настроить базовую надежную службу ASP.NET Core без отслеживания состояния в качестве Service Fabric серверной части.

Дополнительные сведения см. в разделе задние [концы в управлении API](backends.md).

## <a name="prerequisites"></a>Предварительные условия

Предварительные требования для настройки примера службы в кластере Service Fabric под Windows в качестве пользовательского серверного сервера:

* **Среда разработки Windows** . Установите [Visual Studio 2019](https://www.visualstudio.com) и среду разработки **Azure**, **ASP.NET и веб-разработку**, а также рабочие нагрузки **.NET Core для кросс-платформенной разработки** . Теперь настройте [среду разработки .NET](../service-fabric/service-fabric-get-started.md).

* **Кластер Service Fabric** . см. [руководство по развертыванию Service Fabric кластера под управлением Windows в виртуальной сети Azure](../service-fabric/service-fabric-tutorial-create-vnet-and-windows-cluster.md). Можно создать кластер с существующим сертификатом X. 509 или в целях тестирования создать самозаверяющий сертификат. Кластер создается в виртуальной сети.

* **Пример Service Fabric приложения** . Создайте приложение веб-API и разверните его в кластере Service Fabric, как описано в статье [Интеграция управления API с Service Fabric в Azure](../service-fabric/service-fabric-tutorial-deploy-api-management.md).

    Эти действия создают базовую надежную службу ASP.NET Core без отслеживания состояния с помощью шаблона проекта веб-API по умолчанию. Позже вы предоставляете конечную точку HTTP для этой службы через службу управления API Azure.

    Запишите имя приложения, например `fabric:/myApplication/myService` . 

* **Экземпляр управления API** — существующий или новый экземпляр управления API на уровне **Premium** или  **Developer** и в том же регионе, что и кластер Service Fabric. Если требуется, [Создайте экземпляр управления API](get-started-create-service-instance.md).

* **Виртуальная сеть** . Добавьте экземпляр управления API в виртуальную сеть, созданную для кластера Service Fabric. Для управления API требуется выделенная подсеть в виртуальной сети.

  Инструкции по включению подключения к виртуальной сети для экземпляра службы управления API см. [в статье Использование управления API Azure с виртуальными сетями](api-management-using-with-vnet.md).

## <a name="create-backend---portal"></a>Создание серверной части — портал

### <a name="add-service-fabric-cluster-certificate-to-api-management"></a>Добавление Service Fabric сертификата кластера в Управление API

Сертификат кластера Service Fabric хранится и управляется в хранилище ключей Azure, связанном с кластером. Добавьте этот сертификат в экземпляр управления API в качестве сертификата клиента.

Действия по добавлению сертификата в экземпляр службы управления API см. [в статье Защита серверных служб с помощью проверки подлинности с использованием сертификата клиента в службе управления API Azure](api-management-howto-mutual-certificates.md). 

> [!NOTE]   
> Мы рекомендуем добавить сертификат в Управление API, обратившись к сертификату хранилища ключей. 

### <a name="add-service-fabric-backend"></a>Добавление Service Fabric серверной части

1. Перейдите к экземпляру Управления API на [портале Azure](https://portal.azure.com).
1. В разделе **интерфейсы API** выберите **элементы "конец**" + "  >  **Добавить**".
1. Введите имя серверной части и необязательное описание
1. В списке **тип** выберите **Service Fabric**.
1. В поле **URL-адрес среды выполнения** введите имя серверной службы Service Fabric, на которую служба управления API будет пересылать запросы. Например, `fabric:/myApplication/myService`. 
1. В **разделе Максимальное число повторных попыток разрешения секций** введите число от 0 до 10.
1. Введите конечную точку управления кластера Service Fabric. Эта конечная точка является URL-адресом кластера в порте `19080` , например `https://mysfcluster.eastus.cloudapp.azure.com:19080` .
1. В поле **сертификат клиента** выберите сертификат кластера Service Fabric, добавленный в экземпляр управления API в предыдущем разделе.
1. В поле **метод авторизации конечной точки управления** введите отпечаток или имя X509 сертификата, используемого службой управления кластерами Service Fabric для обмена данными TLS.
1. Включите параметр **проверить цепочку сертификатов** и **проверить параметры имени сертификата** .
1. В поле **учетные данные для авторизации** укажите учетные данные (при необходимости) для доступа к настроенной серверной службе в Service Fabric. Для примера приложения, используемого в этом сценарии, учетные данные авторизации не требуются.
1. Нажмите кнопку **создания**.

:::image type="content" source="media/backends/create-service-fabric-backend.png" alt-text="Создание серверной части Service Fabric":::

## <a name="use-the-backend"></a>Использование серверной части

Чтобы использовать настраиваемую серверную часть, сослаться на нее с помощью [`set-backend-service`](api-management-transformation-policies.md#SetBackendService) политики. Эта политика преобразует базовый URL-адрес внутренней службы внутреннего сервера для входящего запроса API в указанную серверную часть, в данном случае Service Fabric серверной части. 

`set-backend-service`Политика может быть полезна с существующим API для преобразования входящего запроса на другую серверную часть, чем указанная в параметрах API. В демонстрационных целях в этой статье мы создадим API тестирования и настроили политику для направления запросов API на серверную часть Service Fabric. 

### <a name="create-api"></a>Создание API

Чтобы создать пустой API, выполните действия, описанные в статье [Добавление API вручную](add-api-manually.md) .

* В параметрах API оставьте поле **URL-адрес** пустой.
* Добавьте **суффикс URL-адреса API**, например *структуру*.

  :::image type="content" source="media/backends/create-blank-api.png" alt-text="Создание пустого API":::

### <a name="add-get-operation-to-the-api"></a>Добавление операции GET в API

Как показано в [развертывании серверной службы Service Fabric](../service-fabric/service-fabric-tutorial-deploy-api-management.md#deploy-a-service-fabric-back-end-service), пример службы ASP.NET Core, развернутой в кластере Service Fabric, поддерживает одну операцию HTTP GET по URL-адресу `/api/values` .

Ответ по умолчанию для этого пути — это массив JSON из двух строк:

```json
["value1", "value2"]
```

Чтобы протестировать интеграцию управления API с кластером, добавьте соответствующую операцию GET в API по пути `/api/values` :

1. Выберите API, созданный на предыдущем шаге.
1. Щелкните **+ Add Operation** (+ Добавить операцию).
1. В окне **внешнего** интерфейса введите следующие значения и нажмите кнопку **сохранить**.

     | Параметр             | Значение                             | 
    |---------------------|-----------------------------------|
    | **Отображаемое имя**    | *Тестирование серверной части*                       |  
    | **URL-адрес** | GET                               | 
    | **URL-адрес**             | `/api/values`                           | 
    
    :::image type="content" source="media/backends/configure-get-operation.png" alt-text="Добавление операции GET в API":::

### <a name="configure-set-backend-policy"></a>Настройка `set-backend` политики

Добавьте [`set-backend-service`](api-management-transformation-policies.md#SetBackendService) политику в API тестирования.

1. На вкладке **конструктор** в разделе **обработка входящих** сообщений выберите значок редактор кода ( **</>** ). 
1. Поместить курсор внутрь элемента **&lt; входящего &gt;**
1. Добавьте следующую инструкцию политики. В `backend-id` замените имя серверной части Service Fabric.

   `sf-resolve-condition`— Это условие повтора, если раздел кластера не разрешается. Число повторных попыток, заданных при настройке серверной части.

    ```xml
    <set-backend-service backend-id="mysfbackend" sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")"  />
    ```
1. Щелкните **Сохранить**.

    :::image type="content" source="media/backends/set-backend-service.png" alt-text="Настройка политики Set-серверной службы":::

### <a name="test-backend-api"></a>API тестирования серверной части

1. На вкладке **Test (тест** ) выберите операцию **Get** , созданную в предыдущем разделе.
1. Нажмите кнопку **Отправить**.

При правильной настройке ответ HTTP показывает код успешного выполнения HTTP и отображает JSON, возвращенный из серверной Service Fabric службы.

:::image type="content" source="media/backends/test-backend-service.png" alt-text="Тестирование серверной Service Fabric":::

## <a name="next-steps"></a>Дальнейшие действия

* Узнайте, как [настроить политики](api-management-advanced-policies.md) для перенаправления запросов в серверную часть
* Кроме того, с помощью шаблонов управления API [REST API](/rest/api/apimanagement/2020-06-01-preview/backend), [Azure PowerShell](/powershell/module/az.apimanagement/new-azapimanagementbackend)или [Azure Resource Manager](../service-fabric/service-fabric-tutorial-deploy-api-management.md) можно настроить незавершенные работы.

