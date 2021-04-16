---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: 51dc04fbef8d09878f33d7fda6f15039d3afba3e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/29/2021
ms.locfileid: "96026194"
---
### <a name="configure-a-dns-label-for-the-public-ip-address"></a>Настройка имени DNS для общедоступного IP-адреса

Для подключения к СУБД SQL Server через Интернет рекомендуем создать имя DNS для вашего общедоступного IP-адреса. Вы можете подключиться по IP-адресу, но метка DNS создает запись А, которую легче идентифицировать, и абстрагирует базовый общедоступный IP-адрес.

> [!NOTE]
> DNS-метки не обязательны, если вы подключаетесь к экземпляру SQL Server в той же виртуальной сети или на локальном компьютере.

Чтобы создать DNS-метку, сначала выберите **Виртуальные машины** на портале. Выберите виртуальную машину SQL Server, чтобы открыть ее свойства.

1. В обозревателе виртуальной машины выберите **общедоступный IP-адрес**.

    ![общедоступный IP-адрес](./media/virtual-machines-sql-server-connection-steps/rm-public-ip-address.png)

1. В свойствах общедоступного IP-адреса разверните раздел **Конфигурация**.

1. Введите имя DNS. Это имя, которое можно использовать для подключения к виртуальной машине SQL Server по имени, а не по IP-адресу.

1. Нажмите кнопку **Сохранить**.

    ![имя DNS](./media/virtual-machines-sql-server-connection-steps/rm-dns-label.png)

### <a name="connect-to-the-database-engine-from-another-computer"></a>Подключение к ядру СУБД с другого компьютера

1. На компьютере, подключенном к сети Интернет, откройте SQL Server Management Studio (SSMS). Если у вас нет SQL Server Management Studio, его можно скачать [здесь](/sql/ssms/download-sql-server-management-studio-ssms).

1. В диалоговом окне **Подключение к серверу** или **Подключение к ядру СУБД** измените значение **Имя сервера**. Введите IP-адрес или полное DNS-имя виртуальной машины (определено в предыдущей задаче). Кроме того, можно также после запятой указать TCP-порт SQL Server. Например, `mysqlvmlabel.eastus.cloudapp.azure.com,1433`.

1. В поле **Проверка подлинности** выберите **Проверка подлинности SQL Server**.

1. В поле **Имя пользователя** введите допустимое имя пользователя SQL.

1. В поле **Пароль** введите пароль для этого пользователя.

1. Нажмите кнопку **Соединить**.

    ![подключение SSMS](./media/virtual-machines-sql-server-connection-steps/rm-ssms-connect.png)