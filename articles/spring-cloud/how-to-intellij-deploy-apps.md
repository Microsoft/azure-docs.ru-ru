---
title: Руководство по Развертывание приложений Azure Spring Cloud с помощью IntelliJ
description: Сведения о развертывании приложений в Azure Spring Cloud с помощью IntelliJ.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 03/26/2020
ms.custom: devx-track-java
ms.openlocfilehash: 64ff778f9735c690f4622402ef2e3124185fa3ea
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104877128"
---
# <a name="use-intellij-to-deploy-azure-spring-cloud-applications"></a>Развертывание приложений Azure Spring Cloud с помощью IntelliJ

**Эта статья применима к:** ✔️ Java

Подключаемый модуль IntelliJ для Azure Spring Cloud поддерживает развертывание приложений из IntelliJ IDEA.  

Перед выполнением этого примера вы можете ознакомиться с [базовым кратким руководством](spring-cloud-quickstart.md).

## <a name="prerequisites"></a>Предварительные требования
* [IntelliJ IDEA, выпуск Community или Ultimate, версия 2020.1 или 2020.2](https://www.jetbrains.com/idea/download/#section=windows)

## <a name="install-the-plug-in"></a>Установка подключаемого модуля
Вы можете добавить Azure Toolkit for IntelliJ IDEA 3.43.0 из пользовательского интерфейса **подключаемых модулей** в IntelliJ.

1. Запустите IntelliJ.  Если вы уже открыли проект, закройте его, чтобы отобразилось диалоговое окно приветствия. Щелкните ссылку **Configure** (Настроить) в нижнем правом углу, а затем щелкните **Plugins** (Подключаемые модули), чтобы открыть диалоговое окно настройки подключаемого модуля, и выберите **Install Plugins from disk** (Установить подключаемые модули с диска).

    ![Выбор пункта меню настроек](media/spring-cloud-intellij-howto/configure-plugin-1.png)

1. Найдите Azure Toolkit for IntelliJ.  Щелкните **Install**(Установить).

    ![Установка подключаемого модуля](media/spring-cloud-intellij-howto/install-plugin.png)

1. Щелкните **Restart IDE** (Перезагрузить IDE).

## <a name="tutorial-procedures"></a>Процедуры для руководства
Следующие процедуры позволяют развернуть приложение Hello World с помощью IntelliJ IDEA:

* открытие проекта gs-spring-boot;
* развертывание в Azure Spring Cloud;
* отображение журналов потоковой передачи.

## <a name="open-gs-spring-boot-project"></a>Открытие проекта gs-spring-boot

1. Скачайте и распакуйте репозиторий с исходным кодом для этого руководства или клонируйте его с помощью Git: git clone https://github.com/spring-guides/gs-spring-boot.git. 
1. Перейдите в папку gs-spring-boot\complete.
1. Откройте **диалоговое окно приветствия** IntelliJ и выберите **Import Project** (Импорт проекта), чтобы открыть мастер импорта.
1. Выберите папку `gs-spring-boot\complete`.

    ![Импорт проекта](media/spring-cloud-intellij-howto/import-project-1.png)

## <a name="deploy-to-azure-spring-cloud"></a>Развертывание в Azure Spring Cloud
Чтобы выполнить развертывание в Azure, необходимо войти в учетную запись Azure и выбрать подписку.  Дополнительные сведения см. в статье [Установка и вход](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

1. Щелкните правой кнопкой мыши проект в обозревателе проектов IntelliJ и выберите **Azure** -> **Deploy to Azure Spring Cloud** (Развернуть в Azure Spring Cloud).

    ![Развертывание в Azure (шаг 1)](media/spring-cloud-intellij-howto/deploy-to-azure-1.png)

1. Подтвердите имя для приложения, указанное в поле **Name** (Имя). **Имя** здесь обозначает конфигурацию, а не имя приложения. Обычно пользователям его не нужно изменять.
1. Подтвердите идентификатор, полученный из проекта, в поле **Artifact** (Артефакт).
1. Щелкните **App:** (Приложение) и **Create app...** (Создать приложение).

    ![Развертывание в Azure (шаг 2)](media/spring-cloud-intellij-howto/deploy-to-azure-2.png)

1. Введите имя приложения в поле **App name** (Имя приложения), затем щелкните **OK** (ОК).

    ![Развертывание в Azure, подтверждение](media/spring-cloud-intellij-howto/deploy-to-azure-2a.png)

1. Запустите развертывание, нажав кнопку **Run** (Выполнить). 

    ![Развертывание в Azure (шаг 3)](media/spring-cloud-intellij-howto/deploy-to-azure-3.png)

1. Подключаемый модуль запустит команду `mvn package` в проекте, затем создаст новое приложение и развернет JAR-файл, созданный командой `package`.

1. Если URL-адрес приложения не отображается в окне вывода, получите его на портале Azure. Перейдите из группы ресурсов к своему экземпляру Azure Spring Cloud.  Теперь щелкните **Приложения**.  Приложение появится в списке.

    ![Получение тестового URL-адреса](media/spring-cloud-intellij-howto/get-test-url.png)

1. В веб-браузере перейдите по URL-адресу.

    ![Переход в браузере (шаг 2)](media/spring-cloud-intellij-howto/navigate-in-browser-2.png)

## <a name="show-streaming-logs"></a>Отображение журналов потоковой передачи
Чтобы получить журналы, сделайте следующее:
1. Выберите **Azure Explorer** и **Spring Cloud**.
1. Щелкните правой кнопкой мыши запущенное приложение.
1. Выберите **Streaming Logs** (Журналы потоковой передачи) из раскрывающегося списка.

    ![Выбор журналов потоковой передачи](media/spring-cloud-intellij-howto/streaming-logs.png)

1. Выберите экземпляр.

    ![Выбор экземпляра](media/spring-cloud-intellij-howto/select-instance.png)

1. Журнал потоковой передачи отобразится в окне выходных данных.

    ![Вывод журнала потоковой передачи](media/spring-cloud-intellij-howto/streaming-log-output.png)

## <a name="next-steps"></a>Дальнейшие действия
* [Подготовка приложения Spring для Azure Spring Cloud](how-to-prepare-app-deployment.md)
* [Дополнительные сведения об Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/)
