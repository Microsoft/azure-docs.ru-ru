---
title: Создание, Настройка кластеров Корпоративный пакет безопасности в Azure
description: Узнайте, как создавать и настраивать кластеры Корпоративный пакет безопасности в Azure HDInsight.
services: hdinsight
ms.service: hdinsight
ms.topic: how-to
ms.date: 12/10/2019
ms.openlocfilehash: 3e6a5742d2dc8079c36f2d9f39a827f8db99cd67
ms.sourcegitcommit: 42e4f986ccd4090581a059969b74c461b70bcac0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104867174"
---
# <a name="create-and-configure-enterprise-security-package-clusters-in-azure-hdinsight"></a>Создание и настройка кластеров Корпоративный пакет безопасности в Azure HDInsight

Корпоративный пакет безопасности (ESP) для Azure HDInsight предоставляет доступ к аутентификации на основе Active Directory, многопользовательской поддержке и управлению доступом на основе ролей для кластеров Apache Hadoop в Azure. Кластеры HDInsight ESP позволяют организациям безопасно обрабатывать конфиденциальные данные в соответствии с ограниченными корпоративными политиками безопасности.

В этом руководство показано, как создать кластер Azure HDInsight с поддержкой ESP. Здесь также показано, как создать виртуальную машину IaaS Windows, на которой включены Active Directory и служба доменных имен (DNS). Используйте это руководством для настройки необходимых ресурсов, чтобы разрешить локальным пользователям входить в кластер HDInsight с поддержкой ESP.

Созданный вами сервер будет использоваться в качестве замены для *реальной* локальной среды. Вы будете использовать его для настройки и настройки. Позже вы повторите действия в своей среде.

Это также поможет вам создать гибридную среду удостоверений с помощью синхронизации хэшей паролей с Azure Active Directory (Azure AD). В этом руководством [используется ESP в HDInsight](apache-domain-joined-architecture.md).

Перед использованием этого процесса в собственной среде выполните следующие действия.

* Настройте Active Directory и DNS.
* Включите Azure AD.
* Синхронизация локальных учетных записей пользователей с Azure AD.

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0002.png" alt-text="Схема архитектуры Azure AD" border="false":::

## <a name="create-an-on-premises-environment"></a>Создание локальной среды

В этом разделе вы будете использовать шаблон развертывания быстрого запуска Azure для создания новых виртуальных машин, настройки DNS и добавления нового Active Directory леса.

1. Перейдите к шаблону быстрого развертывания, чтобы [создать виртуальную машину Azure с новым Active Directory лесом](https://azure.microsoft.com/resources/templates/active-directory-new-domain/).

1. Выберите **развертывание в Azure**.
1. Войдите в подписку Azure.
1. На странице **Создание виртуальной машины Azure с помощью нового леса AD** укажите следующие сведения.

    |Свойство | Значение |
    |---|---|
    |Подписка|Выберите подписку, в которой требуется развернуть ресурсы.|
    |Группа ресурсов|Выберите **создать** и введите имя. `OnPremADVRG`|
    |Расположение|Выберите расположение.|
    |Имя входа администратора|`HDIFabrikamAdmin`|
    |Пароль администратора|Введите пароль.|
    |Имя домена|`HDIFabrikam.com`|
    |Префикс DNS|`hdifabrikam`|

    Оставьте остальные значения по умолчанию.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-azure-vm-ad-forest.png" alt-text="Шаблон для создания виртуальной машины Azure с новым лесом Azure AD" border="true":::

1. Просмотрите **условия**, а затем установите флажок **я принимаю указанные выше условия**.
1. Выберите **приобрести** и отслеживайте развертывание и дождитесь его завершения. Выполнение развертывания займет около 30 минут.

## <a name="configure-users-and-groups-for-cluster-access"></a>Настройка пользователей и групп для доступа к кластеру

В этом разделе вы создадите пользователей, которые будут иметь доступ к кластеру HDInsight, в конце этого руководством.

1. Подключитесь к контроллеру домена с помощью удаленный рабочий стол.
    1. В портал Azure перейдите в раздел **группы ресурсов**  >  **онпремадврг**  >  **адвм**  >  **подключить**.
    1. В раскрывающемся списке **IP-адрес** выберите общедоступный IP-адрес.
    1. Выберите **скачать RDP-файл**, а затем откройте файл.
    1. Используйте `HDIFabrikam\HDIFabrikamAdmin` в качестве имени пользователя.
    1. Введите пароль, выбранный для учетной записи администратора.
    1. Щелкните **ОК**.

1. На панели мониторинга **Диспетчер сервера** контроллер домена перейдите в **меню Инструменты**  >  **Active Directory пользователи и компьютеры**.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/server-manager-active-directory-screen.png" alt-text="На панели мониторинга диспетчер сервера откройте управление Active Directory" border="true":::

1. Создайте два новых пользователя: **хдиадмин** и **HDIUser**. Эти два пользователя будут входить в кластеры HDInsight.

    1. На странице **Active Directory пользователи и компьютеры** щелкните правой кнопкой мыши `HDIFabrikam.com` и перейдите к пункту **Новый**  >  **пользователь**.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-active-directory-user.png" alt-text="Создание нового Active Directory пользователя" border="true":::

    1. На странице **новый объект-пользователь** введите имя `HDIUser` и **имя входа пользователя**.  Другие поля будут заполнены автоматически. Выберите **Далее**.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0020.png" alt-text="Создание первого объекта пользователя &quot;Администратор&quot;" border="true":::

    1. Во всплывающем окне введите пароль для новой учетной записи. Выберите **срок действия пароля не ограничен**, а затем во всплывающем сообщении нажмите **кнопку ОК** .
    1. Нажмите кнопку **Далее**, а затем **Готово** , чтобы создать новую учетную запись.
    1. Повторите описанные выше действия, чтобы создать пользователя `HDIAdmin` .

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0024.png" alt-text="Создание второго объекта пользователя &quot;Администратор&quot;" border="true":::

1. Создайте глобальную группу безопасности.

    1. В **Active Directory пользователи и компьютеры** щелкните правой кнопкой мыши `HDIFabrikam.com` , а затем выберите пункт **создать**  >  **группу**.

    1. Введите `HDIUserGroup` в текстовое поле **имя группы** .

    1. Щелкните **ОК**.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-active-directory-group.png" alt-text="Создание новой группы Active Directory" border="true":::

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0028.png" alt-text="Создать новый объект" border="true":::

1. Добавьте элементы в **хдиусерграуп**.

    1. Щелкните правой кнопкой мыши **HDIUser** и выберите **Добавить в группу...**.
    1. В текстовом поле **Введите имена объектов для выбора** введите `HDIUserGroup` . Затем нажмите кнопку **ОК**, а затем снова **кнопку ОК** во всплывающем окне.
    1. Повторите предыдущие шаги для учетной записи **хдиадмин** .

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/active-directory-add-users-to-group.png" alt-text="Добавление члена HDIUser в группу Хдиусерграуп" border="true":::

Теперь вы создали среду Active Directory. Вы добавили двух пользователей и группу пользователей, которые имеют доступ к кластеру HDInsight.

Пользователи будут синхронизированы с Azure AD.

### <a name="create-an-azure-ad-directory"></a>Создание каталога Azure AD

1. Войдите на портал Azure.
1. Выберите **создать ресурс** и введите `directory` . Выберите **Azure Active Directory**  >  **создать**.
1. В разделе **название организации** введите `HDIFabrikam` .
1. В поле **имя исходного домена** введите `HDIFabrikamoutlook` .
1. Нажмите кнопку **создания**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-new-directory.png" alt-text="Создание каталога Azure AD" border="true":::

### <a name="create-a-custom-domain"></a>Создание пользовательского домена

1. В новом **Azure Active Directory** в разделе **Управление** выберите **пользовательские доменные имена**.
1. Щелкните **+ Добавить личный домен**.
1. В поле **имя личного домена** введите `HDIFabrikam.com` , а затем выберите **Добавить домен**.
1. Затем завершите [Добавление данных DNS в регистратор домена](../../active-directory/fundamentals/add-custom-domain.md#add-your-dns-information-to-the-domain-registrar).

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-custom-domain.png" alt-text="Создание пользовательского домена" border="true":::

### <a name="create-a-group"></a>Создание группы

1. В новом **Azure Active Directory** в разделе **Управление** выберите **группы**.
1. Выберите **+ создать группу**.
1. В текстовом поле **имя группы** введите `AAD DC Administrators` .
1. Нажмите кнопку **создания**.

## <a name="configure-your-azure-ad-tenant"></a>Настройка клиента Azure AD

Теперь вы настроите клиент Azure AD, чтобы вы могли синхронизировать пользователей и группы из локального экземпляра Active Directory с облаком.

Создайте Active Directory администратора клиента.

1. Войдите в портал Azure и выберите свой клиент Azure AD **хдифабрикам**.

1. Выберите **Управление**  >  **Пользователи**  >  **Новый пользователь**.

1. Введите следующие сведения для нового пользователя:

   **Удостоверение**

   |Свойство |Описание |
   |---|---|
   |Имя пользователя|Введите `fabrikamazureadmin` в текстовое поле. В раскрывающемся списке доменное имя выберите `hdifabrikam.com`|
   |Имя| Введите `fabrikamazureadmin`.|

   **Пароль**
   1. Выберите параметр **Разрешить создание пароля**.
   1. Введите безопасный пароль по своему усмотрению.

   **Группы и роли**
   1. Выберите **0 групп**.
   1. Выберите **Администраторы контроллера домена AAD**, а затем **выберите**.

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/azure-ad-add-group-member.png" alt-text="Диалоговое окно &quot;группы Azure AD&quot;" border="true":::

   1. Выберите **пользователь**.
   1. Выберите **глобальный администратор**, **а затем щелкните**.

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/azure-ad-add-role-member.png" alt-text="Диалоговое окно &quot;роль Azure AD&quot;" border="true":::

1. Нажмите кнопку **создания**.

1. Затем поменяйте вход нового пользователя в портал Azure там, где ему будет предложено изменить пароль. Это необходимо сделать перед настройкой Microsoft Azure Active Directory Connect.

## <a name="sync-on-premises-users-to-azure-ad"></a>Синхронизация локальных пользователей с Azure AD

### <a name="configure-microsoft-azure-active-directory-connect"></a>Настройка Microsoft Azure Active Directory Connect

1. На контроллере домена Скачайте [Microsoft Azure Active Directory Connect](https://www.microsoft.com/download/details.aspx?id=47594).

1. Откройте загруженный исполняемый файл и примите условия лицензии. Выберите **Continue** (Продолжить).

1. Выберите **использовать Экспресс параметры**.

1. На странице **Подключение к Azure AD** введите имя пользователя и пароль глобального администратора для Azure AD. Используйте имя пользователя `fabrikamazureadmin@hdifabrikam.com` , созданное при настройке клиента Active Directory. Выберите **Далее**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0058.png" alt-text="Подключение к Azure A D" border="true":::

1. На странице **Подключение к домен Active Directory службам** введите имя пользователя и пароль для учетной записи администратора предприятия. Используйте имя пользователя `HDIFabrikam\HDIFabrikamAdmin` и пароль, созданные ранее. Выберите **Далее**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0060.png" alt-text="Подключитесь к странице D D." border="true":::

1. На странице **Настройка входа в Azure AD** нажмите кнопку **Далее**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0062.png" alt-text="Страница настройки входа в Azure AD" border="true":::

1. На странице **все готово для настройки** нажмите кнопку **установить**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0064.png" alt-text="Страница &quot;готово к настройке&quot;" border="true":::

1. На странице **Конфигурация завершена** выберите **Выйти**.
   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0078.png" alt-text="Страница &quot;Настройка завершена&quot;" border="true":::

1. После завершения синхронизации убедитесь, что пользователи, созданные в каталоге IaaS, синхронизированы с Azure AD.
   1. Войдите на портал Azure.
   1. Выберите **Azure Active Directory**  >  **хдифабрикам**  >  **пользователей**.

### <a name="create-a-user-assigned-managed-identity"></a>Создание управляемого удостоверения, назначаемого пользователем

Создайте управляемое пользователем удостоверение, которое можно использовать для настройки доменных служб Azure AD (AD DS Azure). Дополнительные сведения см. [в разделе Создание, перечисление, удаление или назначение роли назначенному пользователем управляемому удостоверению с помощью портал Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).

1. Войдите на портал Azure.
1. Выберите **создать ресурс** и введите `managed identity` . Выберите **Создание управляемого удостоверения, назначенное пользователем**  >  .
1. В качестве **имени ресурса** введите `HDIFabrikamManagedIdentity` .
1. Выберите свою подписку.
1. В разделе **Группа ресурсов** выберите **создать** и введите `HDIFabrikam-CentralUS` .
1. В разделе **Расположение** выберите **Центральная американская**.
1. Нажмите кнопку **создания**.

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0082.png" alt-text="Создание нового управляемого удостоверения, назначенного пользователем" border="true":::

### <a name="enable-azure-ad-ds"></a>Включение Azure AD DS

Чтобы включить AD DS Azure, выполните следующие действия. Дополнительные сведения см. [в разделе Включение Azure AD DS с помощью портал Azure](../../active-directory-domain-services/tutorial-create-instance.md).

1. Создайте виртуальную сеть для размещения AD DS Azure. Выполните следующий код PowerShell.

    ```powershell
    # Sign in to your Azure subscription
    $sub = Get-AzSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Connect-AzAccount
    }

    # If you have multiple subscriptions, set the one to use
    # Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"
    
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS' -Location 'Central US' -Name 'HDIFabrikam-AADDSVNET' -AddressPrefix 10.1.0.0/16
    $subnetConfig = Add-AzVirtualNetworkSubnetConfig -Name 'AADDS-subnet' -AddressPrefix 10.1.0.0/24 -VirtualNetwork $virtualNetwork
    $virtualNetwork | Set-AzVirtualNetwork
    ```

1. Войдите на портал Azure.
1. Выберите **создать ресурс**, введите `Domain services` и выберите **доменные службы Azure AD**  >  **создать**.
1. На странице **Основные сведения** :
   1. В разделе **имя каталога** выберите созданный каталог Azure AD: **хдифабрикам**.
   1. В качестве **имени домена DNS** введите *HDIFabrikam.com*.
   1. Выберите свою подписку.
   1. Укажите группу ресурсов **хдифабрикам-CentralUS**. В качестве **расположения** выберите **Central US**.

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0084.png" alt-text="Основные сведения об Azure AD DS" border="true":::

1. На странице **сеть** выберите сеть (**хдифабрикам-vnet**) и подсеть (**AADDS-Subnet**), созданную с помощью сценария PowerShell. Или выберите **создать** , чтобы создать виртуальную сеть прямо сейчас.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0086.png" alt-text="Шаг создания виртуальной сети" border="true":::

1. На странице **Группа администраторов** должно появиться уведомление о том, что группа с именем **Администраторы контроллера домена AAD** уже создана для администрирования этой группы. Вы можете изменить членство в этой группе, если хотите, но в данном случае ее не нужно изменять. Щелкните **ОК**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0088.png" alt-text="Просмотр группы администраторов Azure AD" border="true":::

1. На странице **Синхронизация** Включите полную синхронизацию, выбрав **все**  >  **ОК**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0090.png" alt-text="Включение синхронизации AD DS Azure" border="true":::

1. На странице **Сводка** проверьте сведения о AD DS Azure и нажмите кнопку **ОК**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0092.png" alt-text="Включение доменных служб Azure AD" border="true":::

После включения AD DS Azure локальный DNS-сервер будет работать на виртуальных машинах Azure AD.

### <a name="configure-your-azure-ad-ds-virtual-network"></a>Настройка виртуальной сети Azure AD DS

Выполните следующие действия, чтобы настроить виртуальную сеть Azure AD DS (**хдифабрикам-ааддсвнет**) для использования пользовательских DNS-серверов.

1. Нахождение IP-адресов пользовательских DNS-серверов.
   1. Выберите `HDIFabrikam.com` ресурс Azure AD DS.
   1. В разделе **Управление** выберите **Свойства**.
   1. Найдите IP-адреса в разделе **IP-адрес в виртуальной сети**.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0096.png" alt-text="Поиск пользовательских IP-адресов DNS для Azure AD DS" border="true":::

1. Настройте **хдифабрикам-ааддсвнет** для использования НАСТРАИВАЕМЫХ IP-адресов 10.0.0.4 и 10.0.0.5.

   1. В разделе **Параметры** выберите **DNS-серверы**.
   1. Выберите **Пользовательский**.
   1. В текстовом поле введите первый IP-адрес (*10.0.0.4*).
   1. Щелкните **Сохранить**.
   1. Повторите шаги, чтобы добавить другой IP-адрес (*10.0.0.5*).

В нашем сценарии мы настроили Azure AD DS для использования IP-адресов 10.0.0.4 и 10.0.0.5, задав тот же IP-адрес в виртуальной сети Azure AD DS.

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0098.png" alt-text="Страница &quot;пользовательские DNS-серверы&quot;" border="true":::

## <a name="securing-ldap-traffic"></a>Защита трафика LDAP

Протокол LDAP используется для чтения и записи в Azure Active Directory. Трафик LDAP можно сделать конфиденциальным и безопасным с помощью технологии SSL (SSL) или протокола TLS. Вы можете включить LDAP через SSL (LDAPs), установив правильно отформатированный сертификат.

Дополнительные сведения о защищенном протоколе LDAP см. в статье [Настройка LDAPS для управляемого домена AD DS Azure](../../active-directory-domain-services/tutorial-configure-ldaps.md).

В этом разделе вы создадите самозаверяющий сертификат, Скачайте сертификат и настроите LDAPs для управляемого домена **хдифабрикам** Azure AD DS.

Следующий скрипт создает сертификат для **хдифабрикам**. Сертификат сохраняется в пути *LocalMachine* .

```powershell
$lifetime = Get-Date
New-SelfSignedCertificate -Subject hdifabrikam.com `
-NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `
-Type SSLServerAuthentication -DnsName *.hdifabrikam.com, hdifabrikam.com
```

> [!NOTE]  
> \#Для формирования запроса на сертификат TLS/SSL можно использовать любую служебную программу или приложение, которое создает допустимый запрос PKCS 10 для шифрования с открытым ключом.

Убедитесь, что сертификат установлен в **личном** хранилище компьютера:

1. Запустите консоль управления (MMC).
1. Добавьте оснастку " **Сертификаты** ", которая управляет сертификатами на локальном компьютере.
1. Разверните **Certificates (Local Computer)** (Сертификаты на локальном компьютере)  > **Личные** > **Сертификаты**. В **личном** хранилище должен присутствовать новый сертификат. Этот сертификат выдается полному имени узла.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0102.png" alt-text="Проверка создания локального сертификата" border="true":::

1. В области справа щелкните созданный сертификат правой кнопкой мыши. Наведите указатель на пункт **все задачи**, а затем выберите **Экспорт**.

1. На странице **Экспорт закрытого ключа** выберите **Да, экспортировать закрытый ключ**. На компьютере, на котором будет импортирован ключ, требуется закрытый ключ для чтения зашифрованных сообщений.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0103.png" alt-text="Страница «Экспорт закрытого ключа» мастера экспорта сертификатов" border="true":::

1. На странице **Формат экспортируемого файла** оставьте параметры по умолчанию, а затем нажмите кнопку **Далее**.
1. На странице **пароль** введите пароль для закрытого ключа. В качестве **шифрования** выберите **TripleDES-SHA1**. Выберите **Далее**.
1. На странице **файл для экспорта** введите путь и имя экспортированного файла сертификата, а затем нажмите кнопку **Далее**. Имя файла должно иметь расширение PFX. Этот файл настраивается в портал Azure для установления безопасного подключения.
1. Включите LDAPs для управляемого домена Azure AD DS.
   1. В портал Azure выберите домен `HDIFabrikam.com` .
   1. В разделе **Управление** выберите **защищенный протокол LDAP**.
   1. На странице **защищенный протокол LDAP** в разделе **защищенный протокол LDAP** выберите **включить**.
   1. Найдите PFX-файл сертификата, экспортированный на компьютере.
   1. Введите пароль сертификата.

   :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0113.png" alt-text="Включение защищенного протокола LDAP" border="true":::

1. Теперь, когда вы включили LDAPs, убедитесь, что он доступен, включив порт 636.
   1. В группе ресурсов **хдифабрикам-CentralUS** выберите группу безопасности сети **AADDS-HDIFabrikam.com-NSG**.
   1. В разделе **Параметры** выберите **правила безопасности для входящего трафика**  >  **Добавить**.
   1. На странице **Добавление правила безопасности для входящего трафика** введите следующие свойства и выберите **Добавить**.

      | Свойство | Значение |
      |---|---|
      | Источник | Любой |
      | Диапазоны исходных портов | * |
      | Назначение | Любой |
      | Диапазон портов назначения | 636 |
      | Протокол | Любой |
      | Действие | Allow |
      | Приоритет | \<Desired number> |
      | Имя | Port_LDAP_636 |

      :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/add-inbound-security-rule.png" alt-text="Диалоговое окно «Добавление правила безопасности для входящего трафика»" border="true":::

**Хдифабрикамманажедидентити** — назначенное пользователем управляемое удостоверение. Для роли участника доменных служб HDInsight включено управляемое удостоверение, которое позволит этому удостоверению читать, создавать, изменять и удалять операции доменных служб.

:::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0117.png" alt-text="Создание управляемого удостоверения, назначаемого пользователем" border="true":::

## <a name="create-an-esp-enabled-hdinsight-cluster"></a>Создание кластера HDInsight с поддержкой ESP

Для этого шага требуются следующие компоненты:

1. Создайте новую группу ресурсов *хдифабрикам-WestUS* в расположении " **Западная часть США**".
1. Создайте виртуальную сеть, в которой будет размещен кластер HDInsight с поддержкой ESP.

    ```powershell
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS' -Location 'West US' -Name 'HDIFabrikam-HDIVNet' -AddressPrefix 10.1.0.0/16
    $subnetConfig = Add-AzVirtualNetworkSubnetConfig -Name 'SparkSubnet' -AddressPrefix 10.1.0.0/24 -VirtualNetwork $virtualNetwork
    $virtualNetwork | Set-AzVirtualNetwork
    ```

1. Создайте одноранговую связь между виртуальной сетью, в которой размещается Azure AD DS ( `HDIFabrikam-AADDSVNET` ), и виртуальной сетью, в которой будет размещен кластер HDInsight с поддержкой ESP ( `HDIFabrikam-HDIVNet` ). Используйте следующий код PowerShell для пиринга между двумя виртуальными сетями.

    ```powershell
    Add-AzVirtualNetworkPeering -Name 'HDIVNet-AADDSVNet' -RemoteVirtualNetworkId (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS').Id -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS')

    Add-AzVirtualNetworkPeering -Name 'AADDSVNet-HDIVNet' -RemoteVirtualNetworkId (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS').Id -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS')
    ```

1. Создайте новую учетную запись Azure Data Lake Storage 2-го поколения с именем **Hdigen2store**. Настройте учетную запись с управляемым пользователем удостоверением **хдифабрикамманажедидентити**. Дополнительные сведения см. в статье [использование Azure Data Lake Storage 2-го поколения с кластерами Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md).

1. Настройте пользовательский DNS-сервер в виртуальной сети **хдифабрикам-ааддсвнет** .
    1. Перейдите в раздел портал Azure > **группы ресурсов**  >  **онпремадврг**  >  DNS-серверы **хдифабрикам ааддсвнет**  >  .
    1. Выберите **Custom (пользовательские** ) и введите *10.0.0.4* и *10.0.0.5*.
    1. Щелкните **Сохранить**.

        :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0123.png" alt-text="Сохранение пользовательских параметров DNS для виртуальной сети" border="true":::

1. Создайте новый кластер HDInsight Spark с поддержкой ESP.
    1. Выберите **Пользовательский (размер, параметры, приложения)**.
    1. Введите сведения для **основ** (раздел 1). Убедитесь, что **тип кластера** — **Spark 2,3 (HDi 3,6)**. Убедитесь, что **Группа ресурсов** — **хдифабрикам-CentralUS**.

    1. В разделе " **безопасность и сеть** " (раздел 2) введите следующие сведения:
        * В разделе **Корпоративный пакет безопасности** выберите **включено**.
        * Выберите **пользователь администратора кластера** и выберите учетную запись **хдиадмин** , созданную в качестве локального пользователя с правами администратора. Нажмите кнопку **Выбрать**.
        * Выберите **Группа доступа к кластеру**  >  **хдиусерграуп**. Любой пользователь, добавляемый в эту группу в будущем, сможет получить доступ к кластерам HDInsight.

            :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0129.jpg" alt-text="Выберите группу доступа к кластеру Хдиусерграуп" border="true":::

    1. Выполните другие действия в конфигурации кластера и проверьте сведения в **сводке по кластеру**. Нажмите кнопку **создания**.

1. Войдите в пользовательский интерфейс Ambari для созданного кластера в `https://CLUSTERNAME.azurehdinsight.net` . Используйте имя пользователя `hdiadmin@hdifabrikam.com` и пароль администратора.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0135.jpg" alt-text="Окно входа в пользовательский интерфейс Apache Ambari" border="true":::

1. На панели мониторинга кластера выберите **роли**.
1. На странице **роли** в разделе **назначение ролей**, рядом с ролью **Администратор кластера** введите группу *хдиусерграуп*.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0137.jpg" alt-text="Назначение роли администратора кластера хдиусерграуп" border="true":::

1. Откройте клиент Secure Shell (SSH) и войдите в кластер. Используйте **hdiuser** , созданный в локальном экземпляре Active Directory.

    :::image type="content" source="./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0139.jpg" alt-text="Вход в кластер с помощью SSH-клиента" border="true":::

Если вы можете войти с помощью этой учетной записи, вы правильно настроили кластер ESP для синхронизации с локальным экземпляром Active Directory.

## <a name="next-steps"></a>Дальнейшие действия

Ознакомьтесь с [введением в Apache Hadoop безопасность с помощью ESP](hdinsight-security-overview.md).
