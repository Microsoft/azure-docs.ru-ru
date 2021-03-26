---
title: Синхронизация пользователей Azure Active Directory с кластером HDInsight
description: Синхронизация пользователей, прошедших проверку подлинности, от Azure Active Directory к кластеру HDInsight.
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/21/2019
ms.openlocfilehash: 2a753a33e9ddf16cc277ab10c1f91049a6382066
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104871951"
---
# <a name="synchronize-azure-active-directory-users-to-an-hdinsight-cluster"></a>Синхронизация пользователей Azure Active Directory с кластером HDInsight

[Кластеры HDInsight с корпоративный пакет безопасности (ESP)](./domain-joined/hdinsight-security-overview.md) могут использовать строгую проверку подлинности для пользователей Azure Active Directory (Azure AD), а также политики *управления доступом на основе ролей Azure (Azure RBAC)* . При добавлении пользователей и групп в Azure AD можно синхронизировать пользователей, которым необходим доступ к кластеру.

## <a name="prerequisites"></a>Предварительные требования

Если вы еще этого не сделали, [создайте кластер HDInsight с Корпоративным пакетом безопасности](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).

## <a name="add-new-azure-ad-users"></a>Добавление новых пользователей Azure AD

Чтобы просмотреть узлы, откройте веб-интерфейс Ambari. Для каждого узла будут заданы новые параметры автоматического обновления.

1. В [портал Azure](https://portal.azure.com)перейдите к каталогу Azure AD, связанному с кластером ESP.

2. Выберите **All users** (Все пользователи) в левом меню, а затем щелкните **New user** (Новый пользователь).

    :::image type="content" source="./media/hdinsight-sync-aad-users-to-cluster/users-and-groups-new.png" alt-text="портал Azure пользователей и групп":::

3. Заполните форму нового пользователя. Выберите группы, созданные для назначения разрешений на основе кластеров. В этом примере создайте группу с именем HiveUsers, которой можно назначить новых пользователей. [Примеры инструкций](./domain-joined/apache-domain-joined-configure-using-azure-adds.md) для создания кластера ESP включают в себя добавление двух групп: `HiveUsers` и `AAD DC Administrators`.

    :::image type="content" source="./media/hdinsight-sync-aad-users-to-cluster/hdinsight-new-user-form.png" alt-text="портал Azure область пользователя &quot;Выбор групп&quot;":::

4. Нажмите кнопку **создания**.

## <a name="use-the-apache-ambari-rest-api-to-synchronize-users"></a>Синхронизация пользователей с помощью REST API Apache Ambari

Группы пользователей, указанные во время создания кластера, уже синхронизированы. Синхронизация пользователей выполняется автоматически один раз в час. Чтобы немедленно синхронизировать пользователей или группы, которые не указаны при создании кластера, используйте REST API Ambari.

Следующий метод использует POST с REST API Ambari. Дополнительные сведения см. в статье [Управление кластерами HDInsight с помощью REST API Ambari](hdinsight-hadoop-manage-ambari-rest-api.md).

1. С помощью команды [ssh command](hdinsight-hadoop-linux-use-ssh-unix.md) подключитесь к кластеру. Измените приведенную ниже команду, заменив `CLUSTERNAME` именем своего кластера, а затем введите команду:

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. После аутентификации введите следующую команду:

    ```bash
    curl -u admin:PASSWORD -sS -H "X-Requested-By: ambari" \
    -X POST -d '{"Event": {"specs": [{"principal_type": "groups", "sync_type": "existing"}]}}' \
    "https://CLUSTERNAME.azurehdinsight.net/api/v1/ldap_sync_events"
    ```

    Теперь ответ должен выглядеть так:

    ```json
    {
      "resources" : [
        {
          "href" : "http://<ACTIVE-HEADNODE-NAME>.<YOUR DOMAIN>.com:8080/api/v1/ldap_sync_events/1",
          "Event" : {
            "id" : 1
          }
        }
      ]
    }
    ```

1. Чтобы просмотреть состояние синхронизации, выполните новую команду `curl`:

    ```bash
    curl -u admin:PASSWORD https://CLUSTERNAME.azurehdinsight.net/api/v1/ldap_sync_events/1
    ```

    Теперь ответ должен выглядеть так:

    ```json
    {
      "href" : "http://<ACTIVE-HEADNODE-NAME>.YOURDOMAIN.com:8080/api/v1/ldap_sync_events/1",
      "Event" : {
        "id" : 1,
        "specs" : [
          {
            "sync_type" : "existing",
            "principal_type" : "groups"
          }
        ],
        "status" : "COMPLETE",
        "status_detail" : "Completed LDAP sync.",
        "summary" : {
          "groups" : {
            "created" : 0,
            "removed" : 0,
            "updated" : 0
          },
          "memberships" : {
            "created" : 1,
            "removed" : 0
          },
          "users" : {
            "created" : 1,
            "removed" : 0,
            "skipped" : 0,
            "updated" : 0
          }
        },
        "sync_time" : {
          "end" : 1497994072182,
          "start" : 1497994071100
        }
      }
    }
    ```

1. В результате отобразится состояние " **завершено**", был создан один пользователь, и ему было назначено членство. В этом примере пользователь назначен для группы HiveUsers, синхронизированной по протоколу LDAP, так как он был добавлен в ту же группу в Azure AD.

    > [!NOTE]  
    > Предыдущий метод синхронизирует только группы Azure AD, указанные в свойстве **доступа к группе пользователей** параметров домена во время создания кластера. Дополнительные сведения см. в статье [Настройка среды с присоединенной к домену песочницей HDInsight](./domain-joined/apache-domain-joined-configure-using-azure-adds.md).

## <a name="verify-the-newly-added-azure-ad-user"></a>Проверка только что добавленного пользователя Azure AD

Откройте [веб-интерфейс Apache Ambari](hdinsight-hadoop-manage-ambari.md) и убедитесь, что новый пользователь Azure AD добавлен. Получите доступ к веб-ИНТЕРФЕЙСу Ambari, перейдя по адресу **`https://CLUSTERNAME.azurehdinsight.net`** . Укажите имя администратора и пароль кластера.

1. На панели мониторинга Ambari выберите **Manage Ambari** (Управление Ambari) в меню **admin**.

    :::image type="content" source="./media/hdinsight-sync-aad-users-to-cluster/manage-apache-ambari.png" alt-text="Панель мониторинга Apache Ambari управление Ambari":::

2. Выберите **Пользователи** в группе меню **User + Group Management** (Управление пользователями и группами) в левой части страницы.

    :::image type="content" source="./media/hdinsight-sync-aad-users-to-cluster/hdinsight-users-menu-item.png" alt-text="Меню пользователей и групп HDInsight":::

3. Новый пользователь должен быть указан в таблице "Пользователи". Для типа необходимо задать значение `LDAP`, а не `Local`.

    :::image type="content" source="./media/hdinsight-sync-aad-users-to-cluster/hdinsight-users-page.png" alt-text="Обзор страницы пользователей AAD HDInsight":::

## <a name="log-in-to-ambari-as-the-new-user"></a>Вход в Ambari в качестве нового пользователя

Когда новый пользователь (или любой другой пользователь домена) входит в Ambari, он использует полное имя пользователя и учетные данные домена Azure AD.  В Ambari показан псевдоним пользователя, являющийся отображаемым именем пользователя в Azure AD.
В новом примере имя пользователя — `hiveuser3@contoso.com`. В Ambari этот пользователь отображается как `hiveuser3`, но он входит в Ambari как `hiveuser3@contoso.com`.

## <a name="see-also"></a>См. также раздел

* [Настройка политик Apache Hive в кластере HDInsight с ESP](./domain-joined/apache-domain-joined-run-hive.md)
* [Управление кластерами HDInsight с помощью корпоративного пакета безопасности](./domain-joined/apache-domain-joined-manage.md)
* [Предоставление пользователям доступа к представлениям Apache Ambari](hdinsight-authorize-users-to-ambari.md)