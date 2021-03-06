---
title: Задачи руководителя команды по обработке и анализу данных команды
description: Подробное пошаговое руководство по задачам руководителя команды в группе процессов обработки и анализа данных группы
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: df7d2278487c1b098615a14562c498b9187c56eb
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "96000034"
---
# <a name="tasks-for-the-team-lead-on-a-team-data-science-process-team"></a>Задачи для руководителя команды в команде обработки и анализа данных группы

В этой статье описываются задачи, которые *руководитель команды* завершает для своей группы обработки и анализа данных. Цель руководителя группы — создать среду коллективной работы, которая стандартизует [процесс обработки и анализа данных группы](overview.md) (TDSP). TDSP предназначен для улучшения совместной работы и коллективного обучения. 

TDSP — это гибкая, итеративная методология обработки и анализа данных для эффективного предоставления решений прогнозной аналитики и интеллектуальных приложений. Этот процесс по-прежнему соответствует рекомендациям и структурам корпорации Майкрософт и отрасли.  Целью является успешная реализация инициатив по обработке и анализу данных и полное внедрение преимуществ своих программ анализа. Общие сведения о ролях персонала и связанных задачах для группы обработки и анализа данных, стандартизации на TDSP, см. в статье [роли и задачи процесса анализа данных группы](roles-tasks.md).

Руководитель команды управляет группой, состоящей из нескольких специалистов по обработке и анализу данных в рамках предприятия. В зависимости от размера и структуры единицы обработки и анализа данных [менеджер группы](group-manager-tasks.md) и руководитель команды могут быть одним человеком или делегировать свои задачи в суррогаты. Но сами задачи не изменяются. 

На следующей схеме показан рабочий процесс для задач, завершенных руководителем команды для настройки среды команды.

![Рабочий процесс задачи руководителя команды](./media/team-lead-tasks/team-leads-1-creating-teams.png)

1. Создание **командного проекта** в организации группы в Azure DevOps. 
  
1. Переименуйте репозиторий команд по умолчанию в **TeamUtilities**.
  
1. Создайте новый репозиторий **теамтемплате** в командном проекте. 
  
1. Импортируйте содержимое репозиториев **GroupUtilities** и **GroupProjectTemplate** группы в репозитории **TeamUtilities** и **теамтемплате** . 
  
1. Настройте **контроль безопасности** , добавив членов команды и настроив их разрешения.
  
1. При необходимости создайте данные группы и ресурсы аналитики:
   - Добавьте служебные программы для конкретной группы в репозиторий **TeamUtilities** . 
   - Создайте **хранилище файлов Azure** для хранения ресурсов данных, которые могут быть полезны для всей команды. 
   - Подключите хранилище файлов Azure к виртуальной машине для обработки и **анализа данных** руководителя команды (DSVM) и добавьте в нее ресурсы данных.

В следующем руководстве подробно рассматриваются этапы.

> [!NOTE] 
> В этой статье используются Azure DevOps и DSVM для настройки среды группы TDSP, так как это способ реализации TDSP в корпорации Майкрософт. Если ваша команда использует другие платформы размещения кода или разработки, то задачи руководителя команды одинаковы, но способ их выполнения может отличаться.

## <a name="prerequisites"></a>Предварительные условия

В этом учебнике предполагается, что [менеджер группы](group-manager-tasks.md)настроил следующие ресурсы и разрешения:

- **Организация** Azure DevOps для вашей единицы данных
- Репозитории **GroupProjectTemplate** и **GroupUtilities** , заполненные содержимым репозиториев **ProjectTemplate** и **служебных программ** группы Microsoft TDSP
- Разрешения в учетной записи организации для создания проектов и репозиториев для команды

Чтобы иметь возможность клонировать репозитории и изменить их содержимое на локальном компьютере или DSVM или настроить хранилище файлов Azure и подключить его к DSVM, вам потребуется следующее:

- Подписка Azure.
- На компьютере установлен Git. Если вы используете DSVM, то git предварительно установлен. Если система не установлена, см. [описание платформ и средств](platforms-and-tools.md#appendix).
- Если вы хотите использовать DSVM, DSVM Windows или Linux, созданные и настроенные в Azure. Дополнительные сведения и инструкции см. в [документации по виртуальным машинам](../data-science-virtual-machine/index.yml)для обработки и анализа данных.
- Для Windows DSVM — [Диспетчер учетных данных Git (gcm)](https://github.com/Microsoft/Git-Credential-Manager-for-Windows) , установленный на компьютере. В файле *readme.md* прокрутите вниз до раздела **скачать и установить** и выберите **последнюю версию установщика**. Скачайте *exe* Installer с страницы установщика и запустите его. 
- Для DSVM Linux открытый ключ SSH, настроенный в DSVM и добавленный в DevOps Azure. Дополнительные сведения и инструкции см. в разделе **Создание открытого ключа SSH** в [приложении Platforms and Tools](platforms-and-tools.md#appendix). 

## <a name="create-a-team-project-and-repositories"></a>Создание командного проекта и репозиториев

В этом разделе вы создадите следующие ресурсы в Организации Azure DevOps вашей группы:

- Проект **myTeam** в Azure DevOps
- Репозиторий **теамтемплате**
- Репозиторий **TeamUtilities**

В именах, заданных для репозиториев и каталогов в этом учебнике, предполагается, что вы хотите создать отдельный проект для своей команды в рамках вашей крупной организации обработки и анализа данных. Однако вся группа может работать в рамках одного проекта, созданного руководителем группы или администратором организации. Затем все команды обработки и анализа данных создают репозитории в этом отдельном проекте. Этот сценарий может быть допустимым для:
- Небольшая группа обработки и анализа данных, не имеющая нескольких команд обработки и анализа данных. 
- Более крупная группа обработки и анализа данных с несколькими группами обработки и анализа данных, которая, тем не менее, хочет оптимизировать совместную работу между группами с помощью таких действий, как планирование спринта уровня группы. 

Если команды выбирают репозитории для отдельных групп в рамках одного проекта группы, руководители группы должны создавать репозитории с такими именами, как *\<TeamName> шаблон* и *\<TeamName> Служебные программы*. Например: *теаматемплате* и *теамаутилитиес*. 

В любом случае руководители рабочих групп должны знать, какие шаблоны и служебные программы для настройки и клонирования. Руководители проектов должны следовать [задачам руководителя проекта для группы обработки и анализа данных](project-lead-tasks.md) создавать репозитории проектов, как в отдельных проектах, так и в одном проекте. 

### <a name="create-the-myteam-project"></a>Создание проекта MyTeam

Чтобы создать отдельный проект для команды, выполните следующие действия.

1. В веб-браузере перейдите на домашнюю страницу своей группы Azure DevOps Organization по URL-адресу *HTTPS \/ / \<server name> / \<organization name> :* и выберите **Новый проект**. 
   
   ![Выбор нового проекта](./media/team-lead-tasks/team-leads-2-create-new-team.png)
   
1. В диалоговом окне **Создание проекта** введите имя команды, например *myTeam*, в поле **имя проекта**, а затем выберите **Дополнительно**. 
   
1. В разделе **Управление версиями** выберите **Git**, а в разделе **процесс рабочего элемента** выберите **Agile**. Щелкните **Создать**. 
   
   ![Создание проекта](./media/team-lead-tasks/team-leads-3-create-new-team-2.png)
   
Откроется страница **сводки** командного проекта с URL-адресом страницы *https: \/ / \<server name> / \<organization name> / \<team name>*.

### <a name="rename-the-myteam-default-repository-to-teamutilities"></a>Переименуйте репозиторий MyTeam по умолчанию в TeamUtilities

1. На странице **Сводка по** проекту **myTeam** в разделе **какую службу вы хотите начать?** выберите **репозиториев**. 
   
   ![Выбор репозиториев](./media/team-lead-tasks/team-leads-6-rename-team-project-repo.png)
   
1. На странице репозиторий **myTeam** выберите репозиторий **myTeam** в верхней части страницы, а затем выберите **управление репозиториями** из раскрывающегося списка. 
   
   ![Выберите управление репозиториями.](./media/team-lead-tasks/team-leads-7-rename-team-project-repo-2.png)
1. На странице **Параметры проекта** щелкните **...** рядом с репозиторием **myTeam** , а затем выберите **Переименовать репозиторий**. 
   
   ![Выбор переименования репозитория](./media/team-lead-tasks/team-leads-8-rename-team-project-repo-3.png)
   
1. В всплывающем окне **Переименование репозитория myTeam** введите *TeamUtilities*, а затем выберите **Переименовать**. 

### <a name="create-the-teamtemplate-repository"></a>Создание репозитория Теамтемплате

1. На странице **Параметры проекта** выберите **создать репозиторий.** 
   
   ![Выбрать новый репозиторий](./media/team-lead-tasks/team-leads-9-create-team-utilities.png)
   
   Или выберите **репозиториев** в левой области навигации страницы **сводки** проекта **myTeam** , выберите репозиторий в верхней части страницы, а затем в раскрывающемся списке выберите пункт **создать репозиторий** .
   
1. В диалоговом окне **Создание нового репозитория** убедитесь, что в разделе **Тип** выбрано значение **Git** . Введите *теамтемплате* в поле **Имя репозитория** и нажмите кнопку **создать**.
   
   ![Создать репозиторий](./media/team-lead-tasks/team-leads-10-create-team-utilities-2.png)
   
1. Убедитесь, что на странице параметров проекта доступны два репозитория **TeamUtilities** и **теамтемплате** . 
   
   ![Два репозитория команды](./media/team-lead-tasks/team-leads-11-two-repo-in-team.png)

### <a name="import-the-contents-of-the-group-common-repositories"></a>Импорт содержимого групп общих репозиториев

Чтобы заполнить репозитории команды содержимым групп общих репозиториев, настроенных руководителем группы:

1. На домашней странице проекта **myTeam** выберите **репозиториев** в левой области навигации. Если вы получаете сообщение о том, что шаблон **myTeam** не найден, выберите ссылку в **противном случае, перейдите к репозиторию теамтемплате по умолчанию.** 
   
   Откроется репозиторий **теамтемплате** по умолчанию. 
   
1. На странице **теамтемплате пусто** выберите **Импорт**. 
   
   ![Выбор импорта](./media/team-lead-tasks/import-repo.png)
   
1. В диалоговом окне **Импорт репозитория git** выберите **Git** в качестве **типа источника** и введите URL-адрес для репозитория общего шаблона группы в поле **URL-адрес клона**. URL-адрес — *https: \/ / \<server name> / \<organization name> /_git \<repository name> /*. Например: *https: \/ /dev.Azure.com/DataScienceUnit/GroupCommon/_git/GroupProjectTemplate*. 
   
1. Выберите **Импортировать**. Содержимое репозитория шаблонов групп импортируется в репозиторий шаблонов команд. 
   
   ![Импорт репозитория общих шаблонов групп](./media/team-lead-tasks/import-repo-2.png)
   
1. В верхней части страницы **репозиториев** проекта раскрывающийся список и выберите репозиторий **TeamUtilities** .
   
1. Повторите процесс импорта, чтобы импортировать содержимое репозитория общих служебных программ для группы, например *GroupUtilities*, в репозиторий **TeamUtilities** . 
   
Каждый из двух репозиториев команды теперь содержит файлы из соответствующего общего репозитория группы. 

### <a name="customize-the-contents-of-the-team-repositories"></a>Настройка содержимого репозиториев команды

Если вы хотите настроить содержимое репозиториев команды в соответствии с конкретными потребностями вашей команды, это можно сделать сейчас. Вы можете изменять файлы, изменять структуру каталогов или добавлять файлы и папки.

Чтобы изменить, передать или создать файлы или папки непосредственно в Azure DevOps, выполните следующие действия.

1. На странице **Сводка по** проекту **myTeam** выберите **репозиториев**. 
   
1. В верхней части страницы выберите репозиторий, который необходимо настроить.

1. В структуре каталога репозитория перейдите к папке или файлу, которые необходимо изменить. 
   
   - Чтобы создать новые папки или файлы, щелкните стрелку рядом с пунктом **создать**. 
     
     ![Создать новый файл](./media/team-lead-tasks/new-file.png)
     
   - Чтобы отправить файлы, нажмите кнопку **Отправить** файлы. 
     
     ![Отправка файлов](./media/team-lead-tasks/upload-files.png)
     
   - Чтобы изменить существующие файлы, перейдите к файлу и выберите **изменить**. 
     
     ![Изменение файла](./media/team-lead-tasks/edit-file.png)
     
1. После добавления или изменения файлов выберите пункт **зафиксировать**.
   
   ![Фиксация изменений](./media/team-lead-tasks/commit.png)

Чтобы работать с репозиториями на локальном компьютере или DSVM, сначала необходимо скопировать или *клонировать* репозитории на локальный компьютер, а затем зафиксировать и отправить изменения в общие репозитории команды. 

Клонирование репозиториев:

1. На странице **Сводка по** проекту **myTeam** выберите **репозиториев** и в верхней части страницы выберите репозиторий, который необходимо клонировать.
   
1. На странице репозиторий выберите **клонировать** в правом верхнем углу.
   
1. В диалоговом окне **Клонирование репозитория** в разделе **Командная строка** выберите **HTTPS** для подключения по протоколу HTTP или **SSH** для SSH-подключения, а затем скопируйте URL клона в буфер обмена.
   
   ![Копировать URL-адрес клона](./media/team-lead-tasks/clone.png)
   
1. На локальном компьютере создайте следующие каталоги:
   
   - Для Windows: **к:\гитрепос\митеам**
   - Для Linux **$Home/гитрепос/митеам** 
   
1. Перейдите в созданный каталог.
   
1. В Git Bash выполните команду `git clone <clone URL>` , где \<clone URL> — это URL-адрес, скопированный из диалогового окна **клон** .
   
   Например, выполните одну из следующих команд, чтобы клонировать репозиторий **TeamUtilities** в каталог *myTeam* на локальном компьютере. 
   
   **Подключение по протоколу HTTPS:**
   
   ```bash
   git clone https://DataScienceUnit@dev.azure.com/DataScienceUnit/MyTeam/_git/TeamUtilities
   ```
   
   **SSH-подключение:**
   
   ```bash
   git clone git@ssh.dev.azure.com:v3/DataScienceUnit/MyTeam/TeamUtilities
   ```

После внесения любых изменений в локальный клон репозитория зафиксируйте и отправьте изменения в общие репозитории команды. 

Выполните следующие команды git bash из локального каталога **гитрепос\митеам\теамтемплате** или **гитрепос\митеам\теамутилитиес** .

```bash
git add .
git commit -m "push from local"
git push
```

> [!NOTE]
> Если вы впервые зафиксируете в репозитории Git, может потребоваться настроить глобальные параметры *User.Name* и *User.email* перед выполнением `git commit` команды. Выполните эти две команды:
> 
> `git config --global user.name <your name>`
> 
> `git config --global user.email <your email address>`
> 
> При фиксации в нескольких репозиториях Git используйте одно и то же имя и адрес электронной почты для всех. Использование одних и тех же имен и адресов электронной почты удобно при создании Power BI панелей мониторинга для контроля действий Git в нескольких репозиториях.

## <a name="add-team-members-and-configure-permissions"></a>Добавление членов команды и настройка разрешений

Чтобы добавить участников в команду, сделайте следующее:

1. В DevOps Azure на домашней странице проекта **myTeam** в области навигации слева выберите **Параметры проекта** . 
   
1. В левой области навигации **Параметры проекта** выберите **команды**, а затем на странице **команды** выберите команду **myTeam**. 
   
   ![Настройка команд](./media/team-lead-tasks/teams.png)
   
1. На странице **профиль группы** выберите **Добавить**.
   
   ![Добавить в группу MyTeam](./media/team-lead-tasks/add-to-team.png)
   
1. В диалоговом окне **Добавление пользователей и групп** найдите и выберите члены, которые нужно добавить в группу, а затем нажмите кнопку **сохранить изменения**. 
   
   ![Добавление пользователей и групп](./media/team-lead-tasks/add-users.png)
   

Чтобы настроить разрешения для членов команды, выполните следующие действия.

1. В области навигации слева **Параметры проекта** выберите **разрешения**. 
   
1. На странице **разрешения** выберите группу, в которую нужно добавить участников. 
   
1. На странице этой группы выберите **элементы** и нажмите кнопку **Добавить**. 
   
1. В всплывающем окне **пригласить участников** найдите и выберите члены, которые нужно добавить в группу, а затем нажмите кнопку **сохранить**. 
   
   ![Предоставление разрешений членам](./media/team-lead-tasks/grant-permissions.png)

## <a name="create-team-data-and-analytics-resources"></a>Создание данных группы и ресурсов аналитики

Этот шаг является необязательным, но общий доступ к данным и аналитическим ресурсам для всей команды имеет свои преимущества в отношении производительности и стоимости. Члены команды могут выполнять свои проекты на общих ресурсах, сохранять данные в бюджетах и эффективнее сотрудничать. Вы можете создать хранилище файлов Azure и подключить его к DSVM для совместного использования с членами команды. 

Сведения о совместном использовании других ресурсов командой, например кластеров Azure HDInsight Spark, см. в разделе [Platforms and Tools](platforms-and-tools.md). Этот раздел содержит рекомендации с точки зрения обработки и анализа данных по выбору ресурсов, подходящих для ваших потребностей, и ссылки на страницы продуктов и другие важные и полезные учебники.

>[!NOTE]
> Чтобы избежать передачи данных между центрами обработки данных, которые могут быть слишком высокими и дорогостоящими, убедитесь, что группа ресурсов Azure, учетная запись хранения и DSVM размещены в одном регионе Azure. 

### <a name="create-azure-file-storage"></a>Создание хранилища файлов Azure

1. Выполните следующий скрипт, чтобы создать хранилище файлов Azure для ресурсов данных, полезных для всей команды. Сценарий предложит вам ввести сведения о подписке Azure, поэтому все готово к вводу. 

   - На компьютере Windows запустите сценарий из командной строки PowerShell:
     
     ```powershell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/CreateFileShare.ps1" -outfile "CreateFileShare.ps1"
     .\CreateFileShare.ps1
     ```
     
   - На компьютере Linux выполните сценарий из оболочки Linux:
     
     ```shell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/CreateFileShare.sh"
     bash CreateFileShare.sh
     ```
   
1. При появлении запроса войдите в учетную запись Microsoft Azure и выберите подписку, которую хотите использовать.
   
1. Выберите учетную запись хранения для использования или создайте новую в выбранной подписке. Для имени хранилища файлов Azure можно использовать символы в нижнем регистре, цифры и дефисы.
   
1. Чтобы упростить подключение и совместное использование хранилища, нажмите клавишу ВВОД или введите *Y* , чтобы сохранить сведения о хранилище файлов Azure в текстовый файл в текущем каталоге. Этот текстовый файл можно записать в репозиторий **теамтемплате** в идеале в **докс\датадиктионариес**, чтобы все проекты в команде могли получить к нему доступ. Кроме того, вам понадобятся сведения о файле для подключения хранилища файлов Azure к DSVM Azure в следующем разделе. 
   
### <a name="mount-azure-file-storage-on-your-local-machine-or-dsvm"></a>Подключение хранилища файлов Azure на локальном компьютере или DSVM

1. Чтобы подключить хранилище файлов Azure к локальному компьютеру или DSVM, используйте следующий скрипт.
   
   - На компьютере Windows запустите сценарий из командной строки PowerShell:
     
     ```powershell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/AttachFileShare.ps1" -outfile "AttachFileShare.ps1"
     .\AttachFileShare.ps1
     ```
     
   - На компьютере Linux выполните сценарий из оболочки Linux:
     
     ```shell
     wget "https://raw.githubusercontent.com/Azure/Azure-MachineLearning-DataScience/master/Misc/TDSP/AttachFileShare.sh"
     bash AttachFileShare.sh
     ```
   
1. Нажмите клавишу ВВОД или введите *Y* , чтобы продолжить, если файл сведений о хранилище файлов Azure сохранен на предыдущем шаге. Введите полный путь и имя созданного файла. 
   
   Если у вас нет информационного файла хранилища файлов Azure, введите *n* и следуйте инструкциям, чтобы ввести подписку, учетную запись хранения Azure и сведения о хранилище файлов Azure.
   
1. Введите имя локального или TDSP диска для подключения общей папки. На экране появится список существующих имен дисков. Укажите имя диска, которое еще не существует.
   
1. Убедитесь, что новый диск и хранилище успешно подключены к компьютеру.

## <a name="next-steps"></a>Дальнейшие действия

Ниже приведены ссылки на подробные описания других ролей и задач, определенных в процессе обработки и анализа данных группы.

- [Задачи менеджера группы для команды обработки и анализа данных](group-manager-tasks.md)
- [Задачи руководителя проекта для команды обработки и анализа данных](project-lead-tasks.md)
- [Задачи отдельного участника проекта для группы обработки и анализа данных](project-ic-tasks.md)