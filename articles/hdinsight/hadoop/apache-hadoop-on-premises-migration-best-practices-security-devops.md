---
title: 'Безопасность: Миграция локальных Apache Hadoop в Azure HDInsight'
description: Рекомендации по обеспечению безопасности и операциям DevOps в контексте миграции локальных кластеров Hadoop в Azure HDInsight.
ms.reviewer: ashishth
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/19/2019
ms.openlocfilehash: fa6a4a8686fe5a33a6f240a8e972a687e872732a
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98939735"
---
# <a name="migrate-on-premises-apache-hadoop-clusters-to-azure-hdinsight---security-and-devops-best-practices"></a>Миграция локальных кластеров Apache Hadoop в Azure HDInsight — рекомендации по обеспечению безопасности и операциям DevOps

В этой статье представлены рекомендации по обеспечению безопасности и операциям DevOps в системах Azure HDInsight. Это часть цикла, где приведены лучшие методики, применимые при перемещении локальных систем Apache Hadoop в Azure HDInsight.

## <a name="secure-and-govern-cluster-with-enterprise-security-package"></a>Защита кластера и управление им с помощью Корпоративного пакета безопасности

Корпоративный пакет безопасности (ESP) предоставляет поддержку аутентификации на основе Active Directory, многопользовательского режима и управления доступом на основе ролей. Если выбрать ESP, кластер HDInsight подключается к домену AD DS, а администратор предприятия может настроить управление доступом на основе ролей (RBAC) для обеспечения безопасности Apache Hive с помощью Apache Ranger. Кроме того, администратор может проводить аудит доступа сотрудников к данным, а также аудит любых изменений, внесенных в политики управления доступом.

Возможности ESP доступны для кластеров таких типов: Apache Hadoop, Apache Spark, Apache HBase, Apache Kafka и Interactive Query (Hive LLAP).

Чтобы развернуть кластер HDInsight, присоединенный к домену, выполните следующие действия:

- Разверните Azure Active Directory (AAD), передав доменное имя.
- Разверните доменные службы Azure Active Directory (AAD DS).
- Создайте необходимые виртуальную сеть и подсеть.
- Разверните виртуальную машину в виртуальной сети для управления AAD DS.
- Присоедините виртуальную машину к домену.
- Установите инструменты AD и DNS.
- Попросите администратора AAD DS создать подразделение.
- Включите защищенный протокол LDAP для AAD DS.
- Создайте учетную запись службы в Azure Active Directory с делегированными правами администратора на чтение и запись в подразделении. Затем эта учетная запись службы сможет присоединить компьютеры к домену и поместить субъекты компьютеров в подразделение. Кроме того, она может создать субъекты-службы в определенном подразделении, которое указывается во время создания кластера.

    > [!Note]
    > Учетная запись службы не должна быть учетной записью администратора домена AD.

- Разверните кластер HDInsight ESP, настроив следующие параметры:

    |Параметр |Описание |
    |---|---|
    |Доменное имя|доменное имя, связанное с Azure AD DS.|
    |Имя пользователя домена|учетная запись службы в управляемом домене Azure AD DS в контроллере домена, созданная в предыдущем разделе, например `hdiadmin@contoso.onmicrosoft.com`. Этот пользователь домена станет администратором этого кластера HDInsight.|
    |Пароль домена|пароль учетной записи службы.|
    |Подразделение|уникальное имя подразделения, которое необходимо использовать с кластером HDInsight, например `OU=HDInsightOU,DC=contoso,DC=onmicrosoft,DC=com`. Если это подразделение не существует, кластер HDInsight пытается создать подразделение с использованием привилегий учетной записи службы.|
    |URL-АДРЕС LDAPS|Например, `ldaps://contoso.onmicrosoft.com:636` .|
    |Доступ к группе пользователей|группы безопасности, пользователей которых нужно синхронизировать с кластером, например `HiveUsers`. Чтобы указать несколько групп пользователей, разделяйте их точкой с запятой (;). Группы должны существовать в каталоге перед созданием кластера ESP.|

Дополнительные сведения см. в следующих статьях:

- [Общие сведения об обеспечении безопасности Apache Hadoop с помощью Корпоративного пакета безопасности](../domain-joined/hdinsight-security-overview.md)
- [Корпоративный пакет безопасности для HDInsight](../domain-joined/apache-domain-joined-architecture.md)
- [Настройка присоединенных к домену кластеров HDInsight с помощью доменных служб Azure Active Directory](../domain-joined/apache-domain-joined-configure-using-azure-adds.md)
- [Синхронизация пользователей Azure Active Directory с кластером HDInsight](../hdinsight-sync-aad-users-to-cluster.md)
- [Настройка политик Apache Hive в HDInsight с Корпоративным пакетом безопасности](../domain-joined/apache-domain-joined-run-hive.md)
- [Запуск Apache Oozie в присоединенных к домену кластерах HDInsight Hadoop](../domain-joined/hdinsight-use-oozie-domain-joined-clusters.md)

## <a name="implement-end-to-end-enterprise-security"></a>Реализация комплексной безопасности предприятия

Обеспечить комплексную безопасность предприятия можно с помощью следующих элементов управления:

**Частный и защищенный конвейер данных (безопасность на уровне периметра)**
    - Безопасности на уровне периметра можно достичь с помощью виртуальных сетей Azure, групп безопасности сети и службы шлюза.

**Аутентификация и авторизация для доступа к данным**
    - Создайте присоединенный к домену кластер HDInsight с помощью доменных служб Azure Active Directory (Корпоративного пакета безопасности).
    - Используйте Ambari, чтобы предоставить доступ к ресурсам кластера для пользователей AD на основе ролей.
    - Настройте политики контроля доступа для Hive на уровне таблицы, столбца и строк с помощью Apache Ranger.
    - Только администраторы могут получить доступ к кластеру по протоколу SSH.

**Аудит**
    - Просмотрите все случаи получения доступа к ресурсам и данным кластера HDInsight и создайте о них отчет.
    - Просмотрите все изменения в политиках управления доступом и создайте о них отчет.

**Шифрование**
    - Прозрачное шифрование на стороне сервера с использованием ключей, управляемых корпорацией Майкрософт, или ключей, управляемых клиентом.
    - При транзитном шифровании с помощью шифрования Client-Side, HTTPS и TLS.

Дополнительные сведения см. в следующих статьях:

- [Обзор виртуальных сетей Azure](../../virtual-network/virtual-networks-overview.md)
- [Группы безопасности](../../virtual-network/network-security-groups-overview.md)
- [Пиринг между виртуальными сетями Azure](../../virtual-network/virtual-network-peering-overview.md)
- [Руководство по безопасности службы хранилища Azure](../../storage/blobs/security-recommendations.md)
- [Шифрование службы хранилища Azure для неактивных данных](../../storage/common/storage-service-encryption.md)

## <a name="use-monitoring--alerting"></a>Использование мониторинга и оповещений

Дополнительные сведения см. в следующей статье:

[Обзор Azure Monitor](../../azure-monitor/overview.md)

## <a name="upgrade-clusters"></a>Установка новых версий кластеров

Регулярно обновляйте кластеры до последней версии HDInsight, чтобы воспользоваться новейшими возможностями. Обновить кластер до последней версии можно, выполнив следующие шаги:

1. Создайте кластер HDInsight для тестирования, используя последнюю доступную версию HDInsight.
1. Протестируйте новый кластер, чтобы убедиться, что задания и рабочие нагрузки работают должным образом.
1. Измените задания, приложения или рабочие нагрузки соответствующим образом.
1. Создайте резервную копию всех временных данных, хранящихся локально на узлах кластера.
1. Удалите существующий кластер.
1. Создайте кластер последней версии HDInsight в той же подсети виртуальной сети, используя те же данные и хранилище метаданных по умолчанию, что и в предыдущем кластере.
1. Импортируйте все временные данные из резервной копии.
1. Запустите задания и продолжите обработку с помощью нового кластера.

Дополнительные сведения см. в статье [Обновление кластера HDInsight до новой версии](../hdinsight-upgrade-cluster.md).

## <a name="patch-cluster-operating-systems"></a>Исправление операционных систем кластера

Дополнительные сведения см. в статье [исправление ОС для HDInsight](../hdinsight-os-patching.md).

## <a name="post-migration"></a>Действия после миграции

1. **Исправление приложений**. Внесите необходимые изменения в заданиях, процессах и сценариях в итеративном режиме.
2. **Выполнение тестов**. Выполните функциональные тесты и тесты производительности в итеративном режиме.
3. **Оптимизация**. Устраните любые проблемы с производительностью на основе вышеуказанных результатов тестов, а затем выполните повторное тестирование для подтверждения повышения производительности.

## <a name="next-steps"></a>Дальнейшие действия

[Сведения об HDInsight 4.0](./apache-hadoop-introduction.md).