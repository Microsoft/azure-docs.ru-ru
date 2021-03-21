---
title: Параметры безопасности для Hive в Azure HDInsight
description: Параметры безопасности для Hive в кластерах Standard и ESP.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: conceptual
ms.date: 10/02/2020
ms.openlocfilehash: 1189a320d0dc700756c9f7664d0a6303be5dab51
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98940888"
---
# <a name="security-options-for-hive-in-azure-hdinsight"></a>Параметры безопасности для Hive в Azure HDInsight

В этом документе описаны рекомендуемые параметры безопасности для Hive в HDInsight. Эти параметры можно настроить с помощью Ambari.

!["Параметры безопасности для Hive"](./media/hdinsight-security-options-for-hive/security-options-hive.png "Параметры безопасности для Hive")

## <a name="hiveserver2-authentication"></a>Проверка подлинности HiveServer2

Для стандартных кластеров рекомендуемым параметром для проверки подлинности HiveServer2 является значение по умолчанию None. Чтобы включить проверку подлинности, рекомендуется выполнить обновление до кластера [ESP](../domain-joined/hdinsight-security-overview.md) (корпоративный пакет безопасности). 

Для кластеров ESP проверка подлинности [Kerberos](https://web.mit.edu/Kerberos/) включена по умолчанию. Подключаемые модули проверки подлинности (PAM) и пользовательские схемы проверки подлинности не поддерживаются.

## <a name="hiveserver2-authorization"></a>Авторизация HiveServer2

Для стандартных кластеров значение по умолчанию — None. [Склстдаус (авторизация на основе стандартов SQL)](https://cwiki.apache.org/confluence/display/Hive/SQL+Standard+based+hive+authorization) можно включить. Авторизация через [Apache Ranger](https://ranger.apache.org/) не поддерживается для стандартных кластеров. Мы рекомендуем выполнить обновление до кластера ESP для Ranger авторизации. 

Для кластеров ESP Авторизация через Ranger включена по умолчанию. 


## <a name="ssl-encryption-for-hiveserver2"></a>SSL-шифрование для HiveServer2

Включение Hiveserver2 SSL не рекомендуется для обоих кластеров Standard или ESP. Вместо этого на шлюзе включается протокол SSL. [Шифрование при передаче](../domain-joined/encryption-in-transit.md) можно включить для шифрования обмена данными между узлами кластера по [протоколу IPsec](https://en.wikipedia.org/wiki/IPsec).


## <a name="next-steps"></a>Дальнейшие действия
* [Общие сведения о проверке подлинности HiveServer2](https://cwiki.apache.org/confluence/display/Hive/Setting+up+HiveServer2#SettingUpHiveServer2-Authentication/SecurityConfiguration)
* [Обзор авторизации HiveServer2](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Authorization)
* [Включение авторизации Hive на основе стандартов SQL](https://community.cloudera.com/t5/Community-Articles/Getting-started-with-SQLStdAuth/ta-p/244263)
* [Apache Ranger с Hive](../domain-joined/apache-domain-joined-run-hive.md)
