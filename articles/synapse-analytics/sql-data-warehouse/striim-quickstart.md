---
title: Быстрый запуск Стриим
description: Быстро Начните работу с Стриим и Azure синапсе Analytics.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 10/12/2018
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: d6b5fa1a42201479c5d426a5bbb9188d0cb8e03d
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89485980"
---
# <a name="striim-azure-synapse-analytics-marketplace-offering-install-guide"></a>Стриим Azure синапсе Analytics Marketplace, предлагающая рекомендации по установке

В этом кратком руководстве предполагается, что у вас уже есть предварительно существующий экземпляр Azure синапсе Analytics.

Выполните поиск Стриим в Azure Marketplace и выберите параметр Стриим для интеграции данных в Azure синапсе Analytics (промежуточная). 

![Установка Striim][install]

Настройте виртуальную машину Striim, указав заданные свойства, и запишите имя кластера Striim, пароль и пароль администратора.

![Настройка Striim][configure]

После развертывания щелкните основной узел \<VM Name> на портале Azure, щелкните "Подключить" и скопируйте имя для входа, используя локальную учетную запись виртуальной машины. 

![Подключение Стриим к Azure синапсе Analytics][connect]

Скачайте файл sqljdbc42.jar по ссылке <https://www.microsoft.com/en-us/download/details.aspx?id=54671> на локальный компьютер. 

Откройте окно командной строки и перейдите в каталог, куда вы скачали JAR-файл JDBC. С помощью SCP передайте JAR-файл на виртуальную машину Striim, получив ее адрес и пароль на портале Azure.

![Копирование JAR-файла на виртуальную машину][copy-jar]

Откройте еще одно окно командной строки или используйте служебную программу ssh, чтобы подключиться по протоколу SSH к кластеру Striim.

![Установка SSH-подключения к кластеру][ssh]

Выполните следующие команды, чтобы переместить JAR-файл JDBC в каталог lib Striim, после чего запустите и остановите сервер.

   1. sudo su
   2. cd /tmp
   3. mv sqljdbc42.jar /opt/striim/lib
   4. systemctl stop striim-node
   5. systemctl stop striim-dbms
   6. systemctl start striim-dbms
   7. systemctl start striim-node

![Запуск кластера Striim][start-striim]

Теперь откройте свой браузер и перейдите по адресу \<DNS Name>:9080

![Переход на экран входа][navigate]

Войдите с помощью имени пользователя и пароля, которые вы настроили на портале Azure. Выберите предпочитаемый мастер, чтобы приступить к работе, или перейдите на страницу приложений, чтобы начать использовать пользовательский интерфейс для перетаскивания.

![Вход с помощью учетных данных сервера][login]



[install]: ./media/striim-quickstart/install.png
[configure]: ./media/striim-quickstart/configure.png
[connect]:./media/striim-quickstart/connect.png
[copy-jar]:./media/striim-quickstart/copy-jar.png
[ssh]:./media/striim-quickstart/ssh.png
[start-striim]:./media/striim-quickstart/start-striim.png
[navigate]:./media/striim-quickstart/navigate.png
[login]:./media/striim-quickstart/login.png
