---
title: Настройка промежуточной среды в Azure Spring Cloud | Документация Майкрософт
description: Узнайте, как использовать развертывание по методу Blue-Green с помощью Azure Spring Cloud
author: MikeDodaro
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 01/14/2021
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 39420afbd2b1dd8a4e239b814c73e17bb258663c
ms.sourcegitcommit: f3ec73fb5f8de72fe483995bd4bbad9b74a9cc9f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/04/2021
ms.locfileid: "102035209"
---
# <a name="set-up-a-staging-environment-in-azure-spring-cloud"></a>Настройка промежуточной среды в облаке Azure весны

**Эта статья применима к:** ✔️ Java

В этой статье объясняется, как настроить промежуточное развертывание с помощью шаблона развертывания "синий-зеленый" в Azure Веснного облака. Сине-зеленое развертывание — это шаблон непрерывной поставки Azure DevOps, который использует существующую (синюю) версию в режиме реального времени при развертывании новой (зеленого). В этой статье показано, как разместить промежуточное развертывание в рабочей среде без изменения рабочего развертывания.

## <a name="prerequisites"></a>Предварительные требования

* Облачный экземпляр Azure весны в **ценовой** категории " *стандартный* ".
* Azure CLI [облачное расширение Azure весны](/cli/azure/azure-cli-extensions-overview)

В этой статье используется приложение, построенное на основе пружинного инициализатора. Если вы хотите использовать другое приложение в этом примере, необходимо внести простое изменение в общедоступную часть приложения, чтобы отличать промежуточное развертывание от рабочей среды.

>[!TIP]
> Azure Cloud Shell — это бесплатная интерактивная оболочка, которую можно использовать для выполнения инструкций, описанных в этой статье.  Он содержит общие, предварительно установленные инструменты Azure, включая последние версии Git, JDK, Maven и Azure CLI. Если вы вошли в подписку Azure, запустите [Azure Cloud Shell](https://shell.azure.com).  Дополнительные сведения см. в разделе [обзор Azure Cloud Shell](../cloud-shell/overview.md).

Чтобы настроить развертывания со синим зеленым цветом в Azure Веснного облака, следуйте инструкциям в следующих разделах.

## <a name="install-the-azure-cli-extension"></a>Установка расширения Azure CLI

Установите расширение Azure Spring Cloud для Azure CLI с помощью следующей команды:

```azurecli
az extension add --name spring-cloud
```
## <a name="prepare-app-and-deployments"></a>Подготовка приложения и развертываний
Чтобы выполнить сборку приложения, выполните следующие действия.
1. Создайте код для примера приложения с помощью инициализатора пружины с [этой конфигурацией](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.4.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=hellospring&name=hellospring&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.hellospring&dependencies=web,cloud-eureka,actuator,cloud-starter-sleuth,cloud-starter-zipkin,cloud-config-client).

2. Скачайте код.
3. Добавьте в папку следующий исходный файл HelloController. `\src\main\java\com\example\hellospring\` Java.
```java
package com.example.hellospring; 
import org.springframework.web.bind.annotation.RestController; 
import org.springframework.web.bind.annotation.RequestMapping; 

@RestController 

public class HelloController { 

@RequestMapping("/") 

  public String index() { 

      return "Greetings from Azure Spring Cloud!"; 
  } 

} 
```
4. Создайте JAR-файл:
```azurecli
mvn clean packge -DskipTests
```
5. Создайте приложение в облачном экземпляре Azure весны:
```azurecli
az spring-cloud app create -n demo -g <resourceGroup> -s <Azure Spring Cloud instance> --assign-endpoint
```
6. Развертывание приложения в Azure Веснного облака:
```azurecli
az spring-cloud app deploy -n demo -g <resourceGroup> -s <Azure Spring Cloud instance> --jar-path target\hellospring-0.0.1-SNAPSHOT.jar
```
7. Измените код для промежуточного развертывания:
```java
package com.example.hellospring; 
import org.springframework.web.bind.annotation.RestController; 
import org.springframework.web.bind.annotation.RequestMapping; 

@RestController 

public class HelloController { 

@RequestMapping("/") 

  public String index() { 

      return "Greetings from Azure Spring Cloud! THIS IS THE GREEN DEPLOYMENT"; 
  } 

} 
```
8. Перестройте JAR-файл:
```azurecli
mvn clean packge -DskipTests
```
9. Создайте зеленое развертывание: 
```azurecli
az spring-cloud app deployment create -n green --app demo -g <resourceGroup> -s <Azure Spring Cloud instance> --jar-path target\hellospring-0.0.1-SNAPSHOT.jar 
```

## <a name="view-apps-and-deployments"></a>Просмотр приложений и развертываний

Просмотрите развернутые приложения, выполнив следующие процедуры.

1. Перейдите к облачному экземпляру Azure весны в портал Azure.

1. В левой области навигации откройте колонку "приложения", чтобы просмотреть приложения для своего экземпляра службы.

    [![Приложения — панель мониторинга](media/spring-cloud-blue-green-staging/app-dashboard.png)](media/spring-cloud-blue-green-staging/app-dashboard.png)

1. Можно щелкнуть приложение и просмотреть сведения.

    [![Общие сведения о приложениях](media/spring-cloud-blue-green-staging/app-overview.png)](media/spring-cloud-blue-green-staging/app-overview.png)

1. Откройте **развертывания** , чтобы просмотреть все развертывания приложения. В сетке отображаются как рабочие, так и промежуточные развертывания.

    [![Панель мониторинга приложений и развертываний](media/spring-cloud-blue-green-staging/deployments-dashboard.png)](media/spring-cloud-blue-green-staging/deployments-dashboard.png)

1. Щелкните URL-адрес, чтобы открыть развернутое приложение.
    ![URL-адрес развернут](media/spring-cloud-blue-green-staging/running-blue-app.png)
1. Щелкните **Рабочий** в столбце **состояние** , чтобы просмотреть приложение по умолчанию.
    ![Работает по умолчанию](media/spring-cloud-blue-green-staging/running-default-app.png)
1. Щелкните **промежуточное хранение** в столбце **состояние** , чтобы просмотреть промежуточное приложение.
    ![Промежуточное выполнение](media/spring-cloud-blue-green-staging/running-staging-app.png)

>[!TIP]
> * Убедитесь, что конечная точка теста заканчивается косой чертой (/), чтобы убедиться, что CSS-файл загружен правильно.  
> * Если в браузере для просмотра страницы требуется ввести учетные данные, используйте параметр [Декодировать в URL-адресе](https://www.urldecoder.org/) для декодирования тестовой конечной точки. Декодирование URL-адреса возвращает URL-адрес в формате "https:// \<username> : \<password> @ \<cluster-name> . Test.azureapps.IO/Gateway/Green".  Используйте эту форму для доступа к конечной точке.

>[!NOTE]    
> Параметры сервера конфигурации применяются как к промежуточной, так и рабочей среде. Например, если задать путь контекста ( `server.servlet.context-path` ) для шлюза приложений на сервере конфигурации в качестве *сомепас*, то путь к зеленому развертыванию изменится на "https:// \<username> : \<password> @ \<cluster-name> . Test.azureapps.IO/Gateway/Green/somepath/...".
 
 Если на этом этапе посетить общедоступный шлюз приложений, вы увидите старую страницу без новых изменений.

## <a name="set-the-green-deployment-as-the-production-environment"></a>Установка зеленого развертывания в качестве рабочей среды

1. После проверки изменений в промежуточной среде ее можно отправить в рабочую среду. На странице  / **развертывания** приложений выберите приложение, которое в настоящее время находится в `Production` .

1. Щелкните многоточие после **состояния регистрации** зеленого развертывания и настройте промежуточную сборку в рабочей среде. 

   [![Настройка промежуточного развертывания в рабочей среде](media/spring-cloud-blue-green-staging/set-staging-deployment.png)](media/spring-cloud-blue-green-staging/set-staging-deployment.png)

1. Теперь URL-адрес приложения должен отображать изменения.

   ![Промежуточное хранение сейчас в развертывании](media/spring-cloud-blue-green-staging/new-production-deployment.png)

>[!NOTE]
> После настройки зеленого развертывания в качестве рабочей среды, предыдущее развертывание станет промежуточным развертыванием.

## <a name="modify-the-staging-deployment"></a>Изменение промежуточного развертывания

Если вы не удовлетворены изменениями, вы можете изменить код приложения, создать новый пакет JAR и передать его в зеленое развертывание с помощью Azure CLI.

```azurecli
az spring-cloud app deploy  -g <resource-group-name> -s <service-instance-name> -n gateway -d green --jar-path gateway.jar
```

## <a name="delete-the-staging-deployment"></a>Удаление промежуточного развертывания

Чтобы удалить промежуточное развертывание из порта Azure, перейдите на страницу промежуточного развертывания, а затем нажмите кнопку **Удалить** .

Кроме того, можно удалить промежуточное развертывание из Azure CLI, выполнив следующую команду:

```azurecli
az spring-cloud app deployment delete -n <staging-deployment-name> -g <resource-group-name> -s <service-instance-name> --app gateway
```

## <a name="next-steps"></a>Дальнейшие действия

* [CI/CD для Azure Веснного облака](https://review.docs.microsoft.com/azure/spring-cloud/spring-cloud-howto-cicd?branch=pr-en-us-142929&pivots=programming-language-java)