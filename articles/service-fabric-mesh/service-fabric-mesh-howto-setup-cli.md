---
title: Настройка интерфейса командной строки Azure Service Fabric для сетки
description: Интерфейс командной строки (CLI) вложенной службы "Сетка Service Fabric" требуется для развертывания и администрирования ресурсов локально в этой службе. Вот как его настроить.
author: georgewallace
ms.author: gwallace
ms.date: 11/28/2018
ms.topic: conceptual
ms.openlocfilehash: f8a9c26e65ef911ad85806c72c7946947379ab72
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "104613347"
---
# <a name="set-up-service-fabric-mesh-cli"></a>Настройка CLI для Сетки Service Fabric

> [!IMPORTANT]
> Поддержка предварительной версии Сетки Azure Service Fabric была прекращена. Новые развертывания больше не будут разрешены через API Сетки Service Fabric. Поддержка существующих развертываний будет продолжена до 28 апреля 2021 г. включительно.
> 
> Дополнительные сведения см. в статье [Прекращение поддержки предварительной версии Сетки Azure Service Fabric](https://azure.microsoft.com/updates/azure-service-fabric-mesh-preview-retirement/).

Интерфейс командной строки (CLI) вложенной службы "Сетка Service Fabric" требуется для развертывания и администрирования ресурсов локально в этой службе. Вот как его настроить.

Существует три вида интерфейса командной строки, который может использоваться. Они перечислены в таблице, приведенной ниже.

| Модуль CLI | Целевое окружение |  Описание | 
|---|---|---|
| az mesh | Служба "Сетка Azure Service Fabric" | Основной интерфейс командной строки для развертывания приложений и управления ресурсами в среде сетки Azure Service Fabric. 
| sfctl | Локальные кластеры | Служба Service Fabric CLI позволяет развертывать и тестировать ресурсы Service Fabric для локальных кластеров.  
| Интерфейс командной строки Maven | Локальные кластеры и Сетка Azure Service Fabric | Оболочка `az mesh` `sfctl` , которая позволяет разработчикам Java использовать привычный интерфейс командной строки для работы в локальной среде и разработке Azure.  

В режиме предварительной версии CLI для Сетки Azure Service Fabric записывается как расширение для Azure CLI. Вы можете установить его в Azure Cloud Shell или локально в Azure CLI. 

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Для работы с этой статьей требуется Azure CLI версии 2.0.67 или более поздней. Если вы используете Azure Cloud Shell, последняя версия уже установлена.

## <a name="install-the-azure-service-fabric-mesh-cli"></a>Установка интерфейса командной строки Сетки Azure Service Fabric

Если вы еще не сделали этого, установите модуль расширения CLI для сетки Service Fabric Azure с помощью следующей команды: 
 
```azurecli-interactive
az extension add --name mesh
```

Если он уже установлен, обновите существующий модуль CLI для сетки Service Fabric Azure с помощью следующей команды:

```azurecli-interactive
az extension update --name mesh
```

## <a name="install-the-service-fabric-cli-sfctl"></a>Установка интерфейса командной строки Service Fabric (sfctl) 

Следуйте инструкциям раздела [Настройка интерфейса командной строки Service Fabric](../service-fabric/service-fabric-cli.md). Модуль **sfctl** можно использовать для развертывания приложений на основе модели ресурсов для кластеров Service Fabric на локальном компьютере. 

## <a name="install-the-maven-cli"></a>Установка интерфейса командной строки Maven 

Чтобы использовать интерфейс командной строки Maven, необходимо установить на компьютере: 

* [Java](https://www.azul.com/downloads/zulu/)
* [Maven](https://maven.apache.org/download.cgi)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git);
* Интерфейс командной строки Сетка Azure (az mesh) для Сетки Azure Service Fabric 
* SFCTL (sfctl) для локальных кластеров 

Интерфейс командной строки Maven для Service Fabric все еще находится в предварительной версии. 

Чтобы использовать подключаемый модуль Maven в приложении Maven Java, добавьте следующий фрагмент кода в файл pom.xml:

```XML
<project>
  ...
  <build>
    ...
    <plugins>
      ...
      <plugin>
        <groupId>com.microsoft.azure</groupId>
          <artifactId>azure-sfmesh-maven-plugin</artifactId>
          <version>0.1.0</version>
          <configuration>
            ...
          </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

Дополнительные сведения см. в разделе [Руководство по интерфейсу командной строки Maven](service-fabric-mesh-reference-maven.md).

## <a name="next-steps"></a>Дальнейшие действия

Вы также можете настроить [среду разработки Windows](service-fabric-mesh-howto-setup-developer-environment-sdk.md).

Найдите ответы на [часто задаваемые вопросы и способы решения распространенных проблем](service-fabric-mesh-faq.md).

[azure-cli-install]: /cli/azure/install-azure-cli
