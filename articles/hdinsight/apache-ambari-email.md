---
title: Руководство по настройке уведомлений по электронной почте Apache Ambari в Azure HDInsight
description: В этой статье описывается, как использовать SendGrid с Apache Ambari для уведомлений по электронной почте.
ms.service: hdinsight
ms.topic: tutorial
ms.date: 03/10/2020
ms.openlocfilehash: 5b344c0c4b1db9159d0223c861e5d371cb225f5a
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/30/2021
ms.locfileid: "104867208"
---
# <a name="tutorial-configure-apache-ambari-email-notifications-in-azure-hdinsight"></a>Руководство по настройке уведомлений по электронной почте Apache Ambari в Azure HDInsight

В этом руководстве вы узнаете как настраивать уведомления по электронной почте Apache Ambari с помощью SendGrid. [Apache Ambari](./hdinsight-hadoop-manage-ambari.md) упрощает управление кластером HDInsight и его мониторинг за счет удобного пользовательского веб-интерфейса и интерфейса REST API. Ambari предоставляется с кластерами HDInsight и используется для мониторинга кластера и внесения изменений в его конфигурацию. [SendGrid](https://sendgrid.com/solutions/) — это бесплатная облачная служба электронной почты, которая обеспечивает надежную доставку транзакционных писем, предоставляет возможности масштабирования и аналитики в режиме реального времени, а также предоставляет гибкие интерфейсы API, которые упрощают пользовательскую интеграцию. Клиенты Azure могут разблокировать 25 000 бесплатных сообщений электронной почты каждый месяц.

В этом руководстве описано следующее:

> [!div class="checklist"]
> * Получение имени пользователя Sendgrid
> * Настройка уведомлений по электронной почте Apache Ambari

## <a name="prerequisites"></a>Предварительные требования

* Учетная запись электронной почты SendGrid. Инструкции см. в статье [Отправка электронной почты с помощью SendGrid в Azure](../sendgrid-dotnet-how-to-send-email.md).

* Кластер HDInsight. См. [Создание кластеров под управлением Linux в HDInsight с помощью портала Azure](./hdinsight-hadoop-create-linux-clusters-portal.md).

## <a name="obtain-sendgrid-username"></a>Получение имени пользователя SendGrid

1. На [портале Azure](https://portal.azure.com) перейдите к ресурсу SendGrid.

1. На странице "Обзор" выберите **Управление**, чтобы открыть веб-страницу SendGrid для своей учетной записи.

    :::image type="content" source="./media/apache-ambari-email/azure-portal-sendgrid-manage.png" alt-text="Общие сведения о SendGrid на портале Azure":::

1. В меню слева перейдите к имени учетной записи, а затем выберите **Сведения об учетной записи**.

    :::image type="content" source="./media/apache-ambari-email/sendgrid-dashboard-navigation.png" alt-text="Навигация по панели мониторинга SendGrid":::

1. На странице **Сведения об учетной записи** запишите **Имя пользователя**.

    :::image type="content" source="./media/apache-ambari-email/sendgrid-account-details.png" alt-text="Сведения об учетной записи SendGrid":::

## <a name="configure-ambari-e-mail-notification"></a>Настройка уведомлений Ambari по электронной почте

1. В веб-браузере перейдите на страницу `https://CLUSTERNAME.azurehdinsight.net/#/main/alerts`, где `CLUSTERNAME` — это имя вашего кластера.

1. В раскрывающемся списке **Действия** выберите **Управление уведомлениями**.

1. В окне **Manage Alert Notifications** (Управление уведомлениями об оповещениях) выберите значок **+** .

    :::image type="content" source="./media/apache-ambari-email/azure-portal-create-notification.png" alt-text="Снимок экрана: диалоговое окно Manage Alert Notifications (Управление уведомлениями об оповещениях)":::

1. В диалоговом окне **Create Alert Notification** (Создание уведомления об оповещениях) укажите следующие сведения:

    |Свойство |Описание |
    |---|---|
    |Имя|Указание имени для уведомления.|
    |Группы|Настройте расписание желаемым образом.|
    |Severity|Настройте расписание желаемым образом.|
    |Описание|Необязательный параметр.|
    |Метод|Оставьте **EMAIL**.|
    |Электронное письмо (кому)|Укажите адреса электронной почты для получения уведомлений, разделите их запятыми.|
    |SMTP-сервер|`smtp.sendgrid.net`|
    |Порт SMTP|25 или 587 (для незашифрованных или TLS-соединений).|
    |Электронное письмо от|Укажите адрес электронной почты. Адрес не обязательно должен быть подлинным.|
    |Используйте проверку подлинности|Установите этот флажок.|
    |Имя пользователя|Укажите имя пользователя SendGrid.|
    |Пароль|Укажите пароль, который вы использовали при создании ресурса SendGrid в Azure.|
    |Подтверждение паролем|Введите пароль еще раз.|
    |Запустите TLS|Установите этот флажок|

    :::image type="content" source="./media/apache-ambari-email/ambari-create-alert-notification.png" alt-text="Снимок экрана: диалоговое окно Create Alert Notification (Создание уведомления об оповещениях)":::

    Щелкните **Сохранить**. Вы вернетесь в окно **Manage Alert Notifications** (Управление уведомлениями об оповещениях).

1. В окне **Manage Alert Notifications** (Управление уведомлениями об оповещениях) выберите **Закрыть**.

## <a name="next-steps"></a>Дальнейшие действия

В этом руководстве вы узнали как настраивать уведомления по электронной почте Apache Ambari с помощью SendGrid. Дополнительные сведения о работе Apache Ambari.

* [Управление кластерами HDInsight с помощью веб-интерфейса Apache Ambari](./hdinsight-hadoop-manage-ambari.md)

* [Создание уведомления об оповещении](https://docs.cloudera.com/HDPDocuments/Ambari-latest/managing-and-monitoring-ambari/content/amb_create_an_alert_notification.html)