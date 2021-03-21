---
title: Настройка TLS — портал Azure — база данных Azure для MariaDB
description: Узнайте, как настроить конфигурацию TLS с помощью портал Azure для базы данных Azure для MariaDB.
author: mksuni
ms.author: sumuth
ms.service: mariadb
ms.topic: how-to
ms.date: 06/02/2020
ms.openlocfilehash: 55a664ceb3524a0c545e037dcba7a2af90034ad4
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98664772"
---
# <a name="configuring-tls-settings-in-azure-database-for-mariadb-using-azure-portal"></a>Настройка параметров TLS в базе данных Azure для MariaDB с помощью портал Azure

В этой статье описывается, как настроить сервер базы данных Azure для MariaDB для принудительного применения минимальной версии TLS для подключений, чтобы пропускать все соединения с более ранней версией TLS по сравнению с настроенной минимальной версией TLS, тем самым улучшая сетевую безопасность.

Вы можете применить версию TLS для подключения к своей базе данных Azure для MariaDB, задав минимальную версию TLS для своего сервера базы данных. Например, если задать для параметра минимальной настройки TLS значение TLS 1,0, сервер разрешит подключения клиентов, использующих TLS 1,0, 1,1 и 1.2. Кроме того, если задать для этого параметра значение 1,2, то разрешаются подключения только от клиентов, использующих TLS 1.2 +, а все соединения с TLS 1,0 и TLS 1,1 будут отклонены.

## <a name="prerequisites"></a>Предварительные условия

Вот что вам нужно, чтобы выполнить инструкции, приведенные в этом руководстве:

* [База данных Azure для MariaDB](quickstart-create-mariaDB-server-database-using-azure-portal.md)

## <a name="set-tls-configurations-for-azure-database-for-mariadb"></a>Настройка конфигурации TLS для базы данных Azure для MariaDB

Выполните следующие действия, чтобы задать минимальную версию TLS сервера MariaDB:

1. В [портал Azure](https://portal.azure.com/)выберите существующий сервер базы данных Azure для MariaDB.

1. На странице сервер MariaDB в разделе **Параметры** щелкните **Безопасность подключения** , чтобы открыть страницу Настройка безопасности подключения.

1. В **минимальной версии TLS** выберите **1,2** , чтобы запретить подключения с использованием протокола tls версии ниже TLS 1,2 для сервера MariaDB.

    ![Конфигурация TLS базы данных Azure для MariaDB](./media/howto-tls-configurations/tls-configurations.png)

1. **Сохраните** изменения.

1. Уведомление подтверждает успешное включение параметра безопасности подключения.

    ![Настройка Azure Database для MariaDB TLS выполнена](./media/howto-tls-configurations/tls-configurations-success.png)

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о [создании оповещений о метриках](howto-alert-metric.md)