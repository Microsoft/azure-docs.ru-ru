---
title: Распространенные вопросы по сети Azure Service Fabric
description: Изучите ответы на распространенные вопросы о службе "Сетка Azure Service Fabric".
ms.author: pepogors
ms.date: 4/23/2019
ms.topic: troubleshooting
ms.openlocfilehash: 8e53ab0ae4cc463bea8a6a8cb6d339f94fdcac6d
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "99626042"
---
# <a name="commonly-asked-service-fabric-mesh-questions"></a>Распространенные вопросы о службе "Сетка Service Fabric"

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Сетка Azure Service Fabric — это полностью управляемая служба, которая позволяет разработчикам развертывать приложения для микрослужб без управления виртуальными машинами, хранилищем или сетями. Эта статья содержит ответы на часто задаваемые вопросы.

## <a name="how-do-i-report-an-issue-or-ask-a-question"></a>Как сообщить о проблеме или задать вопрос?

Можно задать вопросы, получить ответы от инженеров Майкрософт и сообщить о проблемах в [репозитории GitHub service-fabric-mesh-preview](https://aka.ms/sfmeshissues).

## <a name="quota-and-cost"></a>Квоты и стоимость

### <a name="what-is-the-cost-of-participating-in-the-preview"></a>Какова стоимость использования предварительной версии?

В настоящее время не взимается плата за развертывание приложений или контейнеров в предварительной версии сетки. Ознакомьтесь с обновлениями в, которые могут быть включены для выставления счетов. Однако мы рекомендуем удалить развертываемые ресурсы, не покидая их, пока вы не протестируете их.

### <a name="is-there-a-quota-limit-of-the-number-of-cores-and-ram"></a>Существует ли предел квоты числа ядер и объема ОЗУ?

Да. Ниже приведены соответствующие квоты.

- Число приложений: 5
- Количество ядер на приложение: 12
- Общий объем ОЗУ на приложение: 48 ГБ
- Конечные точки сети и входящего трафика: 5
- Тома Azure, которые можно подключить: 10
- Число реплик службы: 3
- Наибольший контейнер, который вы можете развернуть, может использовать 4 ядра и 16 ГБ ОЗУ.
- Вы можете выделять для контейнеров частичные ядра с шагом в 0,5 ядра, но не более 6 ядер.

### <a name="how-long-can-i-leave-my-application-deployed"></a>Сколько времени приложение может находиться в развернутом состоянии?

На данный момент продолжительность времени существования приложения ограничена двумя днями. Это сделано для эффективного использования свободных ядер, выделенных для предварительного просмотра. Таким образом, вы сможете выполнять развертывание на протяжении не более чем 48 часов, после чего оно будет отключено системой.

Если приложение было отключено, вы можете проверить, было ли это сделано системой, с помощью команды `az mesh app show` в Azure CLI. Выполните команду, чтобы увидеть, вернется ли результат `"status": "Failed", "statusDetails": "Stopped resource due to max lifetime policies for an application during preview. Delete the resource to continue."`. 

Пример: 

```azurecli
az mesh app show --resource-group myResourceGroup --name helloWorldApp
```

```output
{
  "debugParams": null,
  "description": "Service Fabric Mesh HelloWorld Application!",
  "diagnostics": null,
  "healthState": "Ok",
  "id": "/subscriptions/1134234-b756-4979-84re-09d671c0c345/resourcegroups/myResourceGroup/providers/Microsoft.ServiceFabricMesh/applications/helloWorldApp",
  "location": "eastus",
  "name": "helloWorldApp",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroup",
  "serviceNames": [
    "helloWorldService"
  ],
  "services": null,
  "status": "Failed",
  "statusDetails": "Stopped resource due to max lifetime policies for an application during preview. Delete the resource to continue.",
  "tags": {},
  "type": "Microsoft.ServiceFabricMesh/applications",
  "unhealthyEvaluation": null
}
```

Чтобы удалить группу ресурсов, используйте команду `az group delete <nameOfResourceGroup>`.

## <a name="deployments"></a>Развертывания

### <a name="what-container-images-are-supported"></a>Какие образы контейнеров поддерживаются?

При разработке на компьютере с обновлением Windows Fall Creators (версия 1709) можно использовать только образы Docker для Windows версии 1709.

При разработке на компьютере с обновлением Windows 10 за апрель 2018 г. (версия 1803) можно использовать образы Docker как версии 1709, так и версии 1803.

Ниже приведены образы ОС контейнера, которые можно использовать для развертывания служб.
- Windows: windowsservercore и nanoserver
    - Windows Server 1709
    - Windows Server 1803
    - Windows Server 1809
    - Windows Server 2019 LTSC
- Linux
    - Известные ограничения отсутствуют

> [!NOTE]
> Инструментарий Visual Studio для сетки пока не поддерживает развертывание в контейнерах Windows Server 2019 и 1809.

### <a name="what-types-of-applications-can-i-deploy"></a>Какие типы приложений можно развернуть? 

Можно развернуть все, что выполняется в контейнерах, которые соответствуют ограничениям, накладываемым на ресурс приложения (Дополнительные сведения о квотах см. выше). Если мы обнаружите, что вы используете сетку для выполнения недопустимых рабочих нагрузок или злоумышленниками систему (т. е. интеллектуальный анализ данных), мы оставляем за собой право завершить развертывание и списка блокировки свою подписку на работу службы. Если у вас возникнут вопросы по выполнению определенной рабочей нагрузки, обратитесь к нам. 

## <a name="developer-experience-issues"></a>Проблемы, с которыми может столкнуться разработчик

### <a name="dns-resolution-from-a-container-doesnt-work"></a>Разрешение DNS из контейнера не работает

Исходящие запросы DNS из контейнера в службу DNS Service Fabric могут завершиться ошибкой при определенных обстоятельствах. Эта проблема рассматривается. Чтобы устранить эту проблему, сделайте следующее.

- Используйте обновление Windows Fall Creators (версии 1709) или выше как базовый образ контейнера.
- Если имя службы не работает, попробуйте использовать полное имя: ServiceName. ApplicationName.
- В файле Docker своей службы добавьте `EXPOSE <port>`, где port означает порт, через который вы предоставляете свою службу. Пример:

```Dockerfile
EXPOSE 80
```

### <a name="dns-does-not-work-the-same-as-it-does-for-service-fabric-development-clusters-and-in-mesh"></a>DNS работает не так, как в кластере разработки Service Fabric и службе "Сетка"

Вам может понадобиться по-разному указывать ссылки на службы в локальном кластере разработки и службе "Сетка Azure".

В своем локальном кластере разработки используйте `{serviceName}.{applicationName}`. В службе "Сетка Azure Service Fabric" используйте `{servicename}`. 

Сейчас служба "Сетка Azure" не поддерживает разрешение DNS между приложениями.

Другие известные проблемы DNS, связанные с запуском кластера Service Fabric Development в Windows 10, см. в разделе [Отладка контейнеров Windows](../service-fabric/service-fabric-how-to-debug-windows-containers.md) и [известных проблем с DNS](../service-fabric/service-fabric-dnsservice.md#known-issues).

### <a name="networking"></a>Сеть

Преобразование сетевых адресов (NAT) сети ServiceFabric может исчезнуть при запуске приложения на локальном компьютере. Чтобы определить, случилось ли это, выполните в командной строке команду

`docker network ls` и обратите внимание, есть ли в списке `servicefabric_nat`.  Если нет, выполните следующую команду: `docker network create -d=nat --subnet 10.128.0.0/24 --gateway 10.128.0.1 servicefabric_nat`

Это позволит устранить проблему, даже если приложение уже развернуто локально и не работает.

### <a name="issues-running-multiple-apps"></a>Проблемы с запуском нескольких приложений

Вы можете столкнуться с такой проблемой, как недоступность ЦП, и ограничениями, установленными для всех приложений. Чтобы устранить эту проблему, сделайте следующее.
- Создайте кластер из пяти узлов.
- Сократите использование ЦП в службах развернутого приложения. Например, в файле service.yaml своей службы измените `cpu: 1.0` на `cpu: 0.5`.

В кластере с одним узлом нельзя развернуть несколько приложений. Чтобы устранить эту проблему, сделайте следующее.
- Используйте кластер из пяти узлов при развертывании нескольких приложений в локальный кластер.
- Удалите приложения, которые вы в данный момент не тестируете.

### <a name="vs-tooling-has-limited-support-for-windows-containers"></a>Инструментарий VS имеет ограниченную поддержку для контейнеров Windows

Инструментарий Visual Studio поддерживает развертывание только контейнеров Windows с базовой версией ОС Windows Server 1709 и 1803 уже сегодня. 

## <a name="feature-gaps-and-other-known-issues"></a>Пробелы в функциях и другие известные проблемы

### <a name="after-deploying-my-application-the-network-resource-associated-with-it-does-not-have-an-ip-address"></a>После развертывания приложения сетевой ресурс, связанный с ним, не имеет IP-адреса

Имеется известная проблема, когда IP-адрес становится доступным не сразу. Проверьте состояние сетевого ресурса через несколько минут, чтобы просмотреть связанный IP-адрес.

### <a name="my-application-fails-to-access-the-right-networkvolume-resource"></a>Мое приложение не может получить доступ к нужному сетевому ресурсу или ресурсу тома

В модели приложения используйте полный идентификатор ресурса для сетей и томов, чтобы иметь к ним доступ. Ниже приведен образец из примера краткого руководства:

```json
"networkRefs": [
    {
    "name":  "[resourceId('Microsoft.ServiceFabric/networks', 'SbzVotingNetwork')]" 
    }
]
```

### <a name="when-i-scale-out-all-of-my-containers-are-affected-including-running-ones"></a>При развертывании затрагиваются все контейнеры, включая запущенные

Над устранением этой ошибки работают.

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о Service Fabric сети см. в [обзоре](service-fabric-mesh-overview.md).
