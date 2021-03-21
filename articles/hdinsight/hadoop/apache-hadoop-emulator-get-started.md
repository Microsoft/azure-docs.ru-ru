---
title: Узнайте, как использовать песочницу Apache Hadoop, эмулятор — Azure HDInsight.
description: 'Чтобы начать ознакомление с экосистемой Hadoop, настройте на виртуальной машине Azure песочницу Hadoop от Hortonworks. '
keywords: эмулятор hadoop,песочница hadoop
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: how-to
ms.date: 05/29/2019
ms.openlocfilehash: eb286adfd7267a78fcf80bcf5ad34f8f1cc9f493
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98946613"
---
# <a name="get-started-with-an-apache-hadoop-sandbox-an-emulator-on-a-virtual-machine"></a>Начало работы с Apache Hadoop песочницой, эмулятором на виртуальной машине

Узнайте, как настроить на виртуальной машине песочницу Apache Hadoop от Hortonworks, чтобы ознакомиться с экосистемой Hadoop. Песочница представляет собой локальную среду разработки для ознакомления с Hadoop, распределенной файловой системой Hadoop (HDFS) и отправкой заданий. Если вы знакомы с Hadoop, вы можете начать использовать Hadoop в Azure, создав кластер HDInsight. Дополнительные сведения см. в статье [Руководство по Hadoop. Начало работы с Hadoop в HDInsight на платформе Linux](apache-hadoop-linux-tutorial-get-started.md).

## <a name="prerequisites"></a>Предварительные условия

* [VirtualBox Oracle](https://www.virtualbox.org/). Скачайте и установите приложение [отсюда](https://www.virtualbox.org/wiki/Downloads).

## <a name="download-and-install-the-virtual-machine"></a>Скачивание и установка виртуальной машины

1. Перейдите к [файлам для загрузки Cloudera](https://www.cloudera.com/downloads/hortonworks-sandbox/hdp.html).

1. Щелкните **VIRTUALBOX** в разделе **Выбор типа установки** , чтобы загрузить последнюю версию ПЕСОЧНИЦЫ Hortonworks на виртуальной машине. Войдите в систему или заполните форму «товарный интерес».

1. Чтобы начать загрузку, нажмите кнопку **песочницы HDP (последняя версия)** .

Инструкции по настройке "песочницы" см. в статье [развертывание и установка песочницы](https://hortonworks.com/tutorial/sandbox-deployment-and-install-guide/section/1/).

Чтобы скачать более раннюю версию песочницы HDP, ознакомьтесь со ссылками в **более ранних версиях**.

## <a name="start-the-virtual-machine"></a>Запуск виртуальной машины

1. Откройте Oracle VirtualBox на виртуальной машине.
1. В меню **Файл** щелкните **Import Appliance** (Импорт устройства), а затем укажите образ песочницы Hortonworks.
1. Выберете песочницу Hortonworks, щелкните **Start** (Запустить) > **Normal Start** (Обычный запуск). После завершения процесса загрузки в виртуальной машине отобразятся инструкции для входа.

    ![нормальное начало диспетчера VirtualBox](./media/apache-hadoop-emulator-get-started/virtualbox-normal-start.png)

1. Откройте веб-браузер и перейдите по отображаемому URL-адресу (обычно `http://127.0.0.1:8888` ).

## <a name="set-sandbox-passwords"></a>Задание паролей для песочницы

1. На **начальном шаге** на странице песочницы Hortonworks выберите **View Advanced Options** (Просмотр дополнительных параметров). Используйте сведения на этой странице, чтобы войти в песочницу с помощью SSH. Введите указанные имя и пароль.

   > [!NOTE]
   > Если клиент SSH не установлен, можно использовать веб-протокол SSH, предоставленный виртуальной машиной по адресу **http://localhost:4200/** .

    При первом подключении с помощью SSH вам будет предложено изменить пароль для учетной записи root. Введите новый пароль, используемый при входе с помощью SSH.

2. После входа в систему введите следующую команду:

    ```bash
    ambari-admin-password-reset
    ```

    При появлении запроса укажите пароль для учетной записи администратора Ambari. Он используется при получении доступа к веб-интерфейсу Ambari.

## <a name="use-hive-commands"></a>Использование команд Hive

1. Используя подключение SSH к песочнице, запустите оболочку Hive с помощью следующей команды:

    ```bash
    hive
    ```

2. После запуска оболочки просмотрите таблицы, представленные в песочнице, используя следующую команду:

    ```hiveql
    show tables;
    ```

3. Используйте следующую команду, чтобы извлечь 10 строк из таблицы `sample_07` :

    ```hiveql
    select * from sample_07 limit 10;
    ```

## <a name="next-steps"></a>Дальнейшие действия

* [Использование средств Azure Data Lake для Visual Studio с песочницей Hortonworks](./apache-hadoop-visual-studio-tools-get-started.md)

* [Learning the ropes of the Hortonworks Sandbox](https://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/)

* [Hadoop tutorial - Getting started with HDP](https://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/) (Руководство по началу работы с Hadoop)