---
title: Миграция к Bridge to Kubernetes
services: azure-dev-spaces
ms.date: 10/21/2020
ms.topic: conceptual
description: Описание процесса миграции с Azure Dev Spaces на мост в Kubernetes
keywords: Azure Dev Spaces, пространства разработки, Docker, Kubernetes, Azure, AKS, служба Kubernetes Azure, контейнеры, мост — Kubernetes
ms.openlocfilehash: d48814df30c17f9b51d8642efa0960a26bbd24f4
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "94888527"
---
# <a name="migrating-to-bridge-to-kubernetes"></a>Миграция к Bridge to Kubernetes

> [!IMPORTANT]
> Azure Dev Spaces будет прекращена 31 октября 2023 г. Клиенты должны перейти на использование Bridge в Kubernetes, средство разработчика клиента.
>
> Цель Azure Dev Spaces была замедление пользователей в разработке на Kubernetes. Значительный компромисс в подходе Azure Dev Spaces был помещать дополнительные затраты на пользователей для понимания конфигураций DOCKER и Kubernetes, а также концепций развертывания Kubernetes. Со временем также стало ясно, что подход Azure Dev Spaces не уменьшает скорость разработки внутреннего цикла на Kubernetes. Bridge to Kubernetes эффективно уменьшает скорость разработки внутреннего цикла и позволяет избежать ненужных затрат на пользователей.
>
> Основная миссия остается неизменной: Создайте лучшие возможности для разработки, тестирования и отладки кода микрослужбы в контексте более крупного приложения.

Мост в Kubernetes предоставляет более светлую альтернативу многим сценариям разработки, которые работают с Azure Dev Spaces. Bridge to Kubernetes — это интерфейс на стороне клиента, использующий расширения в [Visual Studio][vs]   и [Visual Studio Code][vsc].  

Bridge to Kubernetes помогает в разработке, позволяя установленному приложению Kubernetes включать службу, работающую на локальной рабочей станции разработки. В отличие от пространств разработки, мост в Kubernetes сокращает сложность внутреннего цикла за счет пошагового создания конфигураций DOCKER и Kubernetes, позволяя разработчикам сосредоточиться исключительно на бизнес-логике своего кода микрослужб.

Bridge to Kubernetes поможет вам выполнить итерацию по коду, выполняемому на компьютере разработчика, используя зависимости и существующую конфигурацию из среды Kubernetes. В отличие от этого, Azure Dev Spaces развертывает микрослужбы в среде Kubernetes, прежде чем можно будет удаленно отлаживать службу и выполнять итерацию по коду.

Эта статья содержит сравнение Azure Dev Spaces и моста с Kubernetes, а также инструкции по переходу с Azure Dev Spaces на мост в Kubernetes.

## <a name="development-approaches"></a>Подходы к разработке

![Подходы к разработке](media/migrate-to-btk/development-approaches.svg)

Azure Dev Spaces помогли разработчикам Kubernetes работать с кодом, работающим непосредственно в своем кластере AKS, избегая необходимости в локальной среде, которая не похожа на развернутую среду. Этот подход позволил улучшить определенные аспекты разработки, но в нем также появились предварительные требования для обучения и поддержки дополнительных концепций, таких как DOCKER, Kubernetes и Helm, прежде чем можно будет использовать Azure Dev Spaces.

Мост в Kubernetes позволяет разработчикам работать непосредственно на компьютерах разработки, взаимодействуя с остальными частями своего кластера. Этот подход использует преимущества и скорость выполнения кода непосредственно на компьютерах разработки, одновременно совместно используя зависимости и среду, предоставляемые их кластером. Этот подход также использует преимущества точности и масштабирования, которые поступают от выполнения в Kubernetes.

## <a name="feature-comparison"></a>Сравнение возможностей

Azure Dev Spaces и Bridge в Kubernetes имеют аналогичные функции, они также отличаются в нескольких областях:

| Требование  | Azure Dev Spaces  | Bridge to Kubernetes  |
|---------------|-------------------|--------------------------------|
| Служба Azure Kubernetes | В 15 регионах Azure | Любой регион службы AKS    |
| **Безопасность** |
| Требуется доступ к системе безопасности в кластере  | Участник кластера AKS  | Kubernetes RBAC — обновление развертывания   |
| Требуется безопасность на компьютере разработчика  | Н/Д  | Локальный администратор/sudo   |
| **Удобство использования** |
| Независимо от артефактов Kubernetes и DOCKER  | Нет  | Да   |
| Автоматический откат изменений, после отладки  | Нет  | Да   |
| **Поддерживаемые средства клиента** |
| Работает с Visual Studio 2019  | Да  | Да   |
| Работает с Visual Studio Code  | Да  | Да   |
| Работает с интерфейсом командной строки  | Да  | Нет   |
| **Совместимость операционных систем** |
| Работает в Windows 10  | Да  | Да  |
| Работает в Linux  | Да  | Да  |
| Работает в macOS  | Да  | Да  |
| **Capabilities** |
| Изоляция разработчиков или коллективная разработка  | Да  | Да  |
| Выборочная перезапись переменных среды  | Нет  | Да  |
| Создание диаграммы Dockerfile и Helm  | Да  | Нет  |
| Постоянное развертывание кода в Kubernetes  | Да  | Нет  |
| Удаленная отладка в Kubernetes Pod  | Да  | Нет  |
| Локальная отладка, подключенная к Kubernetes  | Нет  | Да  |
| Одновременное Отладка нескольких служб на одной рабочей станции  | Да  | Да  |

## <a name="kubernetes-inner-loop-development"></a>Разработка внутреннего цикла Kubernetes

Крупнейшим различием между Azure Dev Spaces и мостом в Kubernetes является то, где выполняется код. Azure Dev Spaces помогает разрабатывать и отлаживать код микрослужбы, но требует запуска этого кода в кластере. Bridge to Kubernetes позволяет разрабатывать и отлаживать код микрослужбы непосредственно на компьютере разработчика, а также в контексте приложения большего размера, работающего в Kubernetes. Bridge to Kubernetes расширяет периметр кластера Kubernetes и позволяет локальным процессам наследовать конфигурацию от Kubernetes.

![Разработка внутреннего цикла](media/migrate-to-btk/btk-graphic-non-isolated.gif)

При использовании моста для Kubernetes устанавливается сетевое подключение между компьютером разработчика и кластером.В течение времени существования этого подключения в кластер добавляется прокси-сервер, который перенаправляет запросы к службе на компьютер разработчика. При отключении развертывание приложения вернется к использованию исходной версии развертывания, работающей в кластере. Этот подход отличается от того, как работает Azure Dev Spaces, в котором выполняется синхронизация кода с кластером, а затем выполняется сборка и запуск. Azure Dev Spaces также не выполняет откат кода.

Мост в Kubernetes обеспечивает гибкость работы с приложениями, работающими в Kubernetes, независимо от способа их развертывания. При использовании CI/CD для сборки и запуска приложения независимо от того, используются ли установленные средства или пользовательские сценарии, вы по-прежнему можете использовать мост для Kubernetes разработки и отладки кода.

> [!TIP]
>  [Расширение Microsoft Kubernetes][kubernetes-extension] позволяет быстро разрабатывать манифесты Kubernetes с помощью технологии IntelliSense и помогает формировать диаграммы с использованием шаблонов Helm.  

### <a name="transition-to-bridge-to-kubernetes-from-azure-dev-spaces"></a>Переход на мост в Kubernetes из Azure Dev Spaces

1. Если вы используете Visual Studio, обновите интегрированную среду разработки Visual Studio до версии 16,7 или более поздней и установите мост Kubernetes Extension из [Visual Studio Marketplace][vs-marketplace]. Если вы используете Visual Studio Code, установите [мост в расширение Kubernetes][vsc-marketplace].
1. Отключите контроллер Azure Dev Spaces с помощью портал Azure или [интерфейса командной строки Azure dev Spaces][azds-delete].
1. Используйте [Azure Cloud Shell](https://shell.azure.com). На компьютерах Mac, Linux или Windows с установленной Bash откройте командную строку Bash Shell. Убедитесь, что в среде командной строки доступны следующие средства: Azure CLI, Docker, kubectl, Dock, tar и гунзип.
1. Создайте реестр контейнеров или используйте существующий. Вы можете создать реестр контейнеров в Azure с помощью [реестра контейнеров Azure](https://azure.microsoft.com/services/container-registry/) или [DOCKER Hub](https://hub.docker.com/). При использовании Azure Cloud Shell доступен только реестр контейнеров Azure для размещения образов DOCKER.
1. Запустите скрипт миграции, чтобы преобразовать ресурсы Azure Dev Spaces в мосты в ресурсы Kubernetes. Сценарий создает новый образ, совместимый с Bridge to Kubernetes, передает его в указанный реестр, а затем использует [Helm](https://helm.sh) для обновления кластера с помощью образа. Необходимо указать группу ресурсов, имя кластера AKS и реестр контейнеров. Существуют другие параметры командной строки, как показано ниже.

   ```azure-cli
   curl -sL https://aka.ms/migrate-tool | bash -s -- -g ResourceGroupName -n AKSName -h ContainerRegistryName -r PathOfTheProject -y
   ```

   Скрипт поддерживает следующие флаги:

   ```cmd  
    -g Name of resource group of AKS Cluster [required]
    -n Name of AKS Cluster [required]
    -h Container registry name. Examples: ACR, Docker [required]
    -k Kubernetes namespace to deploy resources (uses 'default' otherwise)
    -r Path to root of the project that needs to be migrated (default = current working directory)
    -t Image name & tag in format 'name:tag' (default is 'projectName:stable')
    -i Enable a public endpoint to access your service over internet. (default is false)
    -c Docker build context path. (default = project root path passed to '-r' option)
    -y Doesn't prompt for non-tty terminals
    -d Helm Debug switch
   ```

1. Вручную перенесите все настройки, такие как параметры переменных среды, в *аздс. YAML* в файл Values. *yml* проекта.
1. используемых Удалите `azds.yaml` файл из проекта.
1. Настройте мост для Kubernetes в развернутом приложении. Дополнительные сведения об использовании моста для Kubernetes в Visual Studio см. в статье [Использование моста для Kubernetes в Visual Studio][use-btk-vs]. VS Code см. [в разделе Использование моста с Kubernetes в VS Code][use-btk-vsc].
1. Начните отладку с помощью созданного моста, чтобы Kubernetes профиль отладки/запуска.
1. Вы можете запустить сценарий еще раз, чтобы повторно развернуть его в кластере.

## <a name="team-development-in-a-shared-cluster"></a>Коллективная разработка в общем кластере

Вы также можете использовать маршрутизацию для конкретного разработчика с помощью Bridge to Kubernetes. В сценарии Azure Dev Spaces коллективной разработки используются несколько пространств имен Kubernetes для изоляции службы от остальной части приложения с помощью концепции родительских и дочерних пространств имен. Bridge to Kubernetes предлагает такую же возможность, но с улучшенными характеристиками производительности и в рамках одного пространства имен приложения.

Как мосту, так и Azure Dev Spacesу требуется, чтобы заголовки HTTP присутствовали и распространялись в рамках всего приложения. Если вы уже настроили приложение для работы с распространением заголовка для Azure Dev Spaces, необходимо обновить заголовок. Чтобы перевести мост в Kubernetes из Azure Dev Spaces, обновите настроенный заголовок с *аздс-Route-AS* на *Kubernetes-Route-AS*.

## <a name="evaluate-bridge-to-kubernetes"></a>Оценка моста до Kubernetes

Если вы хотите поэкспериментировать с Bridge до Kubernetes перед отключением Azure Dev Spaces в кластере, проще всего использовать новый кластер. Если вы попытаетесь использовать Azure Dev Spaces и мост для одновременного Kubernetes в одном кластере, при использовании функций маршрутизации в обоих случаях будут возникать проблемы.

### <a name="use-visual-studio-to-evaluate-bridge-to-kubernetes"></a>Оценка моста до Kubernetes с помощью Visual Studio

1. Обновите интегрированную среду разработки Visual Studio до версии 16,7 или более поздней и установите мост Kubernetes Extension из [Visual Studio Marketplace][vs-marketplace].
1. Создайте новый кластер AKS и разверните приложение. Можно также использовать [пример приложения][btk-sample-app].
1. Настройте мост для Kubernetes в развернутом приложении. Дополнительные сведения об использовании моста для Kubernetes в Visual Studio см. в статье [Использование моста для Kubernetes][use-btk-vs].
1. Начните отладку в Visual Studio, используя только что созданный мост для Kubernetes отладки профиля.

### <a name="use-visual-studio-code-to-evaluate-bridge-to-kubernetes"></a>Использование Visual Studio Code для вычисления моста в Kubernetes

1. Установите [мост в расширение Kubernetes][vsc-marketplace].
1. Создайте новый кластер AKS и разверните приложение. Можно также использовать [пример приложения][btk-sample-app].
1. Настройте мост для Kubernetes в развернутом приложении. Дополнительные сведения об использовании моста для Kubernetes в Visual Studio Code см. в разделе [Использование моста с Kubernetes][use-btk-vsc].
1. Начните отладку в Visual Studio, используя только что созданный мост для Kubernetes запуска профиля.

## <a name="next-steps"></a>Дальнейшие действия

Узнайте больше о принципах работы моста с Kubernetes.

> [!div class="nextstepaction"]
> [Как работает Bridge to Kubernetes][how-it-works-bridge-to-kubernetes]


[azds-delete]: how-to/install-dev-spaces.md#remove-azure-dev-spaces-using-the-cli
[kubernetes-extension]: https://marketplace.visualstudio.com/items?itemName=ms-kubernetes-tools.vscode-kubernetes-tools
[btk-sample-app]: /visualstudio/containers/bridge-to-kubernetes#install-the-sample-application
[how-it-works-bridge-to-kubernetes]: /visualstudio/containers/overview-bridge-to-kubernetes
[use-btk-vs]: /visualstudio/containers/bridge-to-kubernetes#connect-to-your-cluster-and-debug-a-service
[use-btk-vsc]: https://code.visualstudio.com/docs/containers/bridge-to-kubernetes
[vs]: https://visualstudio.microsoft.com/
[vsc-marketplace]: https://marketplace.visualstudio.com/items?itemName=mindaro.mindaro
[vs-marketplace]: https://marketplace.visualstudio.com/items?itemName=ms-azuretools.mindaro
[vsc]: https://code.visualstudio.com/
