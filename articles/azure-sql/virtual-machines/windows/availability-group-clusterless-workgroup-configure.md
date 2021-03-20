---
title: Настройка независимой от домена группы доступности рабочей группы
description: Узнайте, как настроить Active Directory независимую от домена группу доступности Always On на SQL Server виртуальной машине в Azure.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
editor: ''
tags: azure-service-management
ms.assetid: 53981f7e-8370-4979-b26a-93a5988d905f
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 01/29/2020
ms.author: mathoma
ms.openlocfilehash: 0f194101720481f71434709c467d0e3130a0f1f9
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "97359461"
---
# <a name="configure-a-workgroup-availability-group"></a>Настройка группы доступности рабочей группы 
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

В этой статье объясняется, как создать независимый от домена Active Directory кластер с группой доступности Always On, который также называется кластером рабочей группы. Здесь рассматриваются действия по подготовке и настройке рабочей группы и группы доступности, а также упоминаются пошаговые инструкции, приведенные в других статьях (например, о создании кластера или развертывании группы доступности). 


## <a name="prerequisites"></a>Предварительные требования

Чтобы настроить группу доступности рабочей группы, вам потребуется следующее:
- По крайней мере две виртуальные машины Windows Server 2016 (или более поздней версии) с SQL Server 2016 (или более поздней версии), развернутые в одной группе доступности или разных зонах доступности с использованием статических IP-адресов. 
- Локальная сеть с не менее чем 4 свободными IP-адресами в подсети. 
- Учетная запись на каждом компьютере в группе администраторов, у которой также есть права системного администратора в SQL Server. 
- Следующие открытые порты: TCP 1433, TCP 5022, TCP 59999. 

В этой статье используются следующие значения параметров, однако при необходимости их можно изменить: 

| **имя**; | **Параметр** |
| :------ | :---------------------------------- |
| **Узел1**   | AGNode1 (10.0.0.4) |
| **Узел2**   | AGNode2 (10.0.0.5) |
| **Имя кластера** | AGWGAG (10.0.0.6) |
| **Средство прослушивания** | AGListener (10.0.0.7) | 
| **DNS-суффикс** | ag.wgcluster.example.com | 
| **Имя рабочей группы** | AGWorkgroup | 
| &nbsp; | &nbsp; |

## <a name="set-a-dns-suffix"></a>Задание DNS-суффикса 

На этом этапе настраивается DNS-суффикс для обоих серверов. Например, `ag.wgcluster.example.com`. Это позволяет использовать имя объекта, к которому нужно подключиться, в качестве полного адреса в сети, например `AGNode1.ag.wgcluster.example.com`. 

Чтобы настроить DNS-суффикс, выполните следующие действия:

1. На первом узле и откройте диспетчер сервера RDP. 
1. Выберите **Локальный сервер**, затем в разделе **Имя компьютера** выберите имя виртуальной машины. 
1. Выберите **Изменить...** в разделе **Чтобы переименовать этот компьютер...** . 
1. Дайте рабочей группе понятное название, например `AGWORKGROUP`: 

   ![Изменение имени рабочей группы](./media/availability-group-clusterless-workgroup-configure/1-change-workgroup-name.png)

1. Нажмите **Еще...** , чтобы открыть диалоговое окно **DNS-суффикс и NetBIOS-имя компьютера**. 
1. Введите имя DNS-суффикса в разделе **Основной DNS-суффикс этого компьютера**, например `ag.wgcluster.example.com`, а затем нажмите **ОК**: 

   ![Снимок экрана — это диалоговое окно с суффиксом D и именем компьютера NetBIOS, в котором можно ввести значение.](./media/availability-group-clusterless-workgroup-configure/2-add-dns-suffix.png)

1. Убедитесь, что **полное имя компьютера** теперь включает DNS-суффикс, а затем нажмите **ОК**, чтобы сохранить изменения: 

   ![На снимке экрана показано, где можно просмотреть полное имя компьютера.](./media/availability-group-clusterless-workgroup-configure/3-confirm-full-computer-name.png)

1. Перезагрузите сервер, когда будет предложено. 
1. Повторите эти действия на всех других узлах, которые будут использоваться в группы доступности. 

## <a name="edit-a-host-file"></a>Изменение файла узла

Поскольку Active Directory не используется, проверка подлинности подключений Windows невозможна. В связи с этим нужно задать доверенные узлы, отредактировав файл hosts в текстовом редакторе. 

Чтобы отредактировать файл hosts, выполните следующие действия:

1. RDP в виртуальной машине. 
1. В **проводнике** откройте папку `c:\windows\system32\drivers\etc`. 
1. Щелкните правой кнопкой мыши файл **hosts** и откройте его в **Блокноте** (или любом другом текстовом редакторе).
1. В конце файла добавьте запись для каждого узла, группы доступности и прослушивателя в виде `IP Address, DNS Suffix #comment`. Например: 

   ```
   10.0.0.4 AGNode1.ag.wgcluster.example.com #Availability group node
   10.0.0.5 AGNode2.ag.wgcluster.example.com #Availability group node
   10.0.0.6 AGWGAG.ag.wgcluster.example.com #Cluster IP
   10.0.0.7 AGListener.ag.wgcluster.example.com #Listener IP
   ```
 
   ![Добавление записей для IP-адреса, кластера и прослушивателя в файл hosts](./media/availability-group-clusterless-workgroup-configure/4-host-file.png)

## <a name="set-permissions"></a>Установка разрешений

Поскольку Active Directory для управления разрешениями не используется, для создания кластера потребуется вручную разрешить учетную запись локального администратора, которая не является встроенной. 

Для этого выполните следующий командлет PowerShell в административном сеансе PowerShell на каждом узле: 

```PowerShell

new-itemproperty -path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System -Name LocalAccountTokenFilterPolicy -Value 1
```

## <a name="create-the-failover-cluster"></a>Создание отказоустойчивого кластера.

На этом этапе создается отказоустойчивый кластер. Если вы не знакомы с этой процедурой, вы можете воспользоваться инструкциями из [учебника по отказоустойчивым кластерам](failover-cluster-instance-storage-spaces-direct-manually-configure.md).

Вот несколько важных различий между процедурой, описанной в учебнике, и действиями, которые необходимо выполнить для кластера рабочей группы:
- Снимите флажки **Хранение** и **Локальные дисковые пространства** перед запуском проверки кластера. 
- При добавлении узлов в кластер укажите полное имя, например:
   - `AGNode1.ag.wgcluster.example.com`
   - `AGNode2.ag.wgcluster.example.com`
- Снимите флажок **Добавление всех допустимых хранилищ в кластер**. 

После создания кластера назначьте ему статический IP-адрес. Для этого выполните следующие действия.

1. На одном из узлов откройте **диспетчер отказоустойчивости кластеров**, выберите кластер, затем щелкните правой кнопкой мыши элемент **Имя: \<ClusterNam>** в разделе **Основные ресурсы кластера** и выберите **Свойства**. 

   ![Свойства запуска для кластера с указанным именем](./media/availability-group-clusterless-workgroup-configure/5-launch-cluster-name-properties.png)

1. Выберите IP-адрес в списке **IP-адреса** и нажмите **изменить**. 
1. Выберите вариант **Использовать статический**, укажите IP-адрес кластера, а затем нажмите **ОК**. 

   ![Установка статического IP-адреса для кластера](./media/availability-group-clusterless-workgroup-configure/6-provide-static-ip-for-cluster.png)

1. Проверьте правильность параметров и нажмите **ОК**, чтобы сохранить их:

   ![Проверка свойств кластера](./media/availability-group-clusterless-workgroup-configure/7-verify-cluster-properties.png)

## <a name="create-a-cloud-witness"></a>Создание облака-свидетеля 

На этом этапе настраивается облачный ресурс-свидетель. Если вы не знакомы с этими шагами, см. статью [развертывание облака-свидетеля для отказоустойчивого кластера](/windows-server/failover-clustering/deploy-cloud-witness). 

## <a name="enable-the-availability-group-feature"></a>Включение функции группы доступности 

На этом этапе включается компонент "Группы доступности". Если вы не знакомы с этой процедурой, см. [учебник по группам доступности](availability-group-manually-configure-tutorial.md#enable-availability-groups). 

## <a name="create-keys-and-certificates"></a>Создание ключей и сертификатов

На этом этапе создаются сертификаты, которые используются для входа в SQL на зашифрованной конечной точке. Создайте на каждом узле папку для хранения резервных копий сертификатов, например `c:\certs`. 

Чтобы настроить первый узел, выполните следующие действия: 

1. Откройте **SQL Server Management Studio** и подключитесь к первому узлу, например `AGNode1`. 
1. Откройте окно **нового запроса** и выполните следующую инструкцию Transact-SQL (T-SQL) после установки сложного и безопасного пароля:

   ```sql
   USE master;  
   CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'PassWOrd123!';  
   GO

   --create a cert from the master key
   USE master;  
   CREATE CERTIFICATE AGNode1Cert   
      WITH SUBJECT = 'AGNode1 Certificate';  
   GO  

   --Backup the cert and transfer it to AGNode2
   BACKUP CERTIFICATE AGNode1Cert TO FILE = 'C:\certs\AGNode1Cert.crt';  
   GO  
   ```

1. Затем создайте конечную точку HADR и воспользуйтесь сертификатом для проверки подлинности, выполнив следующую инструкцию Transact-SQL (T-SQL):

   ```sql
   --CREATE or ALTER the mirroring endpoint
   CREATE ENDPOINT hadr_endpoint  
      STATE = STARTED  
      AS TCP (  
         LISTENER_PORT=5022  
         , LISTENER_IP = ALL  
      )   
      FOR DATABASE_MIRRORING (   
         AUTHENTICATION = CERTIFICATE AGNode1Cert  
         , ENCRYPTION = REQUIRED ALGORITHM AES  
         , ROLE = ALL  
      );  
   GO  
   ```

1. В **проводнике** откройте папку, где находится сертификат, например `c:\certs`. 
1. Вручную создайте копию сертификата (например, `AGNode1Cert.crt`) с первого узла и отправьте ее в ту же папку на втором узле. 

Чтобы настроить второй узел, выполните следующие действия: 

1. Подключитесь с помощью **SQL Server Management Studio** ко второму узлу, например `AGNode2`. 
1. В окне **нового запроса** выполните следующую инструкцию Transact-SQL (T-SQL) после установки сложного и безопасного пароля: 

   ```sql
   USE master;  
   CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'PassWOrd123!';  
   GO 
   
   --create a cert from the master key
   USE master;  
   CREATE CERTIFICATE AGNode2Cert   
      WITH SUBJECT = 'AGNode2 Certificate';  
   GO  
   --Backup the cert and transfer it to AGNode1
   BACKUP CERTIFICATE AGNode2Cert TO FILE = 'C:\certs\AGNode2Cert.crt';  
   GO
   ```

1. Затем создайте конечную точку HADR и воспользуйтесь сертификатом для проверки подлинности, выполнив следующую инструкцию Transact-SQL (T-SQL):

   ```sql
   --CREATE or ALTER the mirroring endpoint
   CREATE ENDPOINT hadr_endpoint  
      STATE = STARTED  
      AS TCP (  
         LISTENER_PORT=5022  
         , LISTENER_IP = ALL  
      )   
      FOR DATABASE_MIRRORING (   
         AUTHENTICATION = CERTIFICATE AGNode2Cert  
         , ENCRYPTION = REQUIRED ALGORITHM AES  
         , ROLE = ALL  
      );  
   GO  
   ```

1. В **проводнике** откройте папку, где находится сертификат, например `c:\certs`. 
1. Вручную создайте копию сертификата (например, `AGNode2Cert.crt`) со второго узла и отправьте ее в ту же папку на первом узле. 

Если в кластере есть и другие узлы, повторите для них эти действия, меняя соответствующие названия сертификатов. 

## <a name="create-logins"></a>Создание учетных данных для входа

Для синхронизации данных между узлами используется проверка подлинности с помощью сертификатов. Для этого создайте учетные данные для входа для другого узла, создайте пользователя для входа, создайте сертификат для входа (для использования резервного сертификата), а затем предоставьте разрешение на подключение на зеркалирующей конечной точке. 

Для этого сначала выполните следующий запрос Transact-SQL (T-SQL) на первом узле, например `AGNode1`: 

```sql
--create a login for the AGNode2
USE master;  
CREATE LOGIN AGNode2_Login WITH PASSWORD = 'PassWord123!';  
GO  

--create a user from the login
CREATE USER AGNode2_User FOR LOGIN AGNode2_Login;  
GO  

--create a certificate that the login uses for authentication
CREATE CERTIFICATE AGNode2Cert  
   AUTHORIZATION AGNode2_User  
   FROM FILE = 'C:\certs\AGNode2Cert.crt'  
GO 

--grant connect for login
GRANT CONNECT ON ENDPOINT::hadr_endpoint TO [AGNode2_login];  
GO
```

Затем выполните следующий запрос Transact-SQL (T-SQL) на втором узле, например `AGNode2`: 

```sql
--create a login for the AGNode1
USE master;  
CREATE LOGIN AGNode1_Login WITH PASSWORD = 'PassWord123!';  
GO  

--create a user from the login
CREATE USER AGNode1_User FOR LOGIN AGNode1_Login;  
GO  

--create a certificate that the login uses for authentication
CREATE CERTIFICATE AGNode1Cert  
   AUTHORIZATION AGNode1_User  
   FROM FILE = 'C:\certs\AGNode1Cert.crt'  
GO 

--grant connect for login
GRANT CONNECT ON ENDPOINT::hadr_endpoint TO [AGNode1_login];  
GO
```

Если в кластере есть и другие узлы, повторите для них эти действия, меняя соответствующие названия сертификатов и имена пользователей. 

## <a name="configure-an-availability-group"></a>Настройка группы доступности

На этом этапе настраивается группа доступности и в нее добавляются базы данных. Не создавайте прослушиватель на этом шаге. Если вы не знакомы с этой процедурой, см. [учебник по группам доступности](availability-group-manually-configure-tutorial.md#create-the-availability-group). Не забудьте запустить процедуру отработки отказа и восстановления размещения, чтобы убедиться, что все работает должным образом. 

   > [!NOTE]
   > Если при синхронизации возникнет сбой, вам может потребоваться временно предоставить учетной записи `NT AUTHORITY\SYSTEM` права системного администратора для создания ресурсов кластера на первом узле, например `AGNode1`. 

## <a name="configure-a-load-balancer"></a>Настройка балансировщика нагрузки

На этом последнем шаге настройте подсистему балансировки нагрузки с помощью [портал Azure](availability-group-load-balancer-portal-configure.md) или [PowerShell](availability-group-listener-powershell-configure.md).


## <a name="next-steps"></a>Дальнейшие действия

Для настройки группы доступности также можно использовать [интерфейс командной строки виртуальной машины AZ SQL](./availability-group-az-commandline-configure.md).