---
title: Перемещение базы данных Azure AD Connect с SQL Server Express на SQL Server | Документы Майкрософт
description: В этом документе описано, как переместить базу данных Azure AD Connect с локального сервера SQL Server Express на удаленный сервер SQL Server.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/29/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 94710e99fa7d04d757f2ad5fd7b2d3f6e01371d1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "91306348"
---
# <a name="move-azure-ad-connect-database-from-sql-server-express-to-sql-server"></a>Перемещение базы данных Azure AD Connect с SQL Server Express на SQL Server 

В этом документе описано, как переместить базу данных Azure AD Connect с локального сервера SQL Server Express на удаленный сервер SQL Server.  Для выполнения этой задачи можно использовать описанные ниже процедуры.

## <a name="about-this-scenario"></a>Сведения об этом сценарии
Ниже представлена краткая информация об этом сценарии.  В соответствии с ним база данных Azure AD Connect версии (1.1.819.0) установлена на одном контроллере домена Windows Server 2016.  Для своей базы данных он использует встроенный SQL Server 2012 Express Edition.  База данных будет перемещена на сервер SQL Server 2017.

![Архитектура сценария](media/how-to-connect-install-move-db/move1.png)

## <a name="move-the-azure-ad-connect-database"></a>Перемещение базы данных Azure AD Connect
Выполните следующие действия, чтобы переместить базу данных Azure AD Connect на удаленный сервер SQL Server.

1. На сервере Azure AD Connect перейдите в раздел **Службы** и остановите службу **Microsoft Azure AD Sync**.
2. Откройте папку **%ProgramFiles%\Microsoft Azure AD синк\дата** и скопируйте файлы **ADSync. mdf** и **ADSync_log. ldf** на удаленный SQL Server.
3. Перезапустите службу **Microsoft Azure AD Sync** на сервере Azure AD Connect.
4. Удалите базу данных Azure AD Connect, последовательно выбрав "Панель управления" > "Программы" > "Программы и компоненты".  Выберите Microsoft Azure AD Connect и нажмите кнопку "Удалить" вверху.
5. На удаленном сервере SQL Server откройте SQL Server Management Studio.
6. Правой кнопкой мыши щелкните элемент "Базы данных" и выберите пункт "Вложить".
7. На экране **Присоединение баз данных** щелкните **Добавить** и перейдите к файлу ADSync.mdf.  Нажмите кнопку **OK**.
   ![Присоединение базы данных](media/how-to-connect-install-move-db/move2.png)

8. После присоединения базы данных вернитесь на сервер Azure AD Connect и установите базу данных Azure AD Connect.
9. По завершении установки MSI мастер Azure AD Connect запускается с настройкой режима Express. Закройте экран, щелкнув значок "Выход".
   ![Снимок экрана, на котором показана страница "Добро пожаловать в Azure A D Connect" с "Экспресс-параметрами" в выделенном слева меню.](./media/how-to-connect-install-move-db/db1.png)
10. Запустите новую командную строку или сеанс PowerShell. Перейдите к папке \<drive>\Program Files\Microsoft Azure AD Connect. Выполните команду \AzureADConnect.exe /useexistingdatabase, чтобы запустить мастер Azure AD Connect в режиме установки "Использовать существующую базу данных".
    ![PowerShell](./media/how-to-connect-install-move-db/db2.png)
11. Появится экран приветствия Azure AD Connect. После принятия условий лицензии и заявления о конфиденциальности, щелкните **Продолжить**.
    ![Снимок экрана, показывающий страницу "Добро пожаловать в Azure A D Connect"](./media/how-to-connect-install-move-db/db3.png)
12. На экране **Установить требующиеся компоненты** включен параметр **Использовать существующий SQL Server**. Укажите имя сервера SQL, на котором размещена база данных ADSync. Если экземпляр ядра SQL, используемый для размещения базы данных, не является экземпляром по умолчанию сервера SQL, необходимо указать ядро SQL и имя экземпляра. Кроме того, если просмотр SQL не включен, также необходимо указать номер порта экземпляра ядра SQL. Пример:         
    ![Снимок экрана, на котором показана страница "Установка необходимых компонентов".](./media/how-to-connect-install-move-db/db4.png)           

13. На экране **Подключение к Azure AD** необходимо предоставить учетные данные глобального администратора для каталога Azure AD. Рекомендуется использовать учетную запись в домене onmicrosoft.com по умолчанию. Эта учетная запись используется только для создания учетной записи службы в Azure AD и не используется после завершения работы мастера.
    ![Подключить](./media/how-to-connect-install-move-db/db5.png)
 
14. На экране **Подключить каталоги** имеющийся лес AD, настроенный для синхронизации каталогов, помечен красным значком с крестиком. Для синхронизации изменений из локального леса AD необходима учетная запись AD DS. Мастер Azure AD Connect не может извлечь учетные данные учетной записи AD DS, хранимые в базе данных ADSync, так как они зашифрованы и могут быть расшифрованы только предыдущим сервером Azure AD Connect. Щелкните **Изменить учетные данные**, чтобы указать учетную запись AD DS для леса AD.
    ![Directories](./media/how-to-connect-install-move-db/db6.png)
 

15. Во всплывающем диалоговом окне можно (а) предоставить учетные данные администратора предприятия и позволить Azure AD Connect создать учетную запись AD DS или (б) создать учетную запись AD DS самостоятельно и предоставить ее учетные данные Azure AD Connect. После выбора варианта и предоставления необходимых учетных данных щелкните **ОК**, чтобы закрыть всплывающее диалоговое окно.
    ![Снимок экрана всплывающего диалогового окна "учетная запись для леса" с выбранным параметром "создать новую учетную запись D".](./media/how-to-connect-install-move-db/db7.png)
 

16. После предоставления учетных данных красный значок с крестиком заменяется зеленым значком с галочкой. Щелкните **Далее**.
    ![Снимок экрана, на котором показана страница "подключение каталогов" после ввода учетных данных учетной записи.](./media/how-to-connect-install-move-db/db8.png)
 

17. На экране **все готово для настройки** нажмите кнопку **установить**.
    ![Добро пожаловать!](./media/how-to-connect-install-move-db/db9.png)
 

18. По завершении установки сервер Azure AD Connect автоматически включается в промежуточном режиме. Перед отключением промежуточного режима рекомендуется проверить конфигурацию сервера и ожидающие операции экспорта на наличие неожиданных изменений. 

## <a name="next-steps"></a>Дальнейшие действия

- Узнайте больше об [интеграции локальных удостоверений с Azure Active Directory](whatis-hybrid-identity.md).
- [Установка Azure AD Connect с помощью имеющейся базы данных ADSync](how-to-connect-install-existing-database.md)
- [Установка Azure AD Connect с использованием делегированных разрешений администратора SQL](how-to-connect-install-sql-delegation.md)

