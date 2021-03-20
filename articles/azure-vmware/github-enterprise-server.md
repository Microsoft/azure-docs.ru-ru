---
title: Настройка сервера GitHub Enterprise в частном облаке решения Azure VMware
description: Узнайте, как настроить GitHub Enterprise Server в частном облаке решения Azure VMware.
ms.topic: how-to
ms.date: 02/11/2021
ms.openlocfilehash: 59a76c3976f6fcda88423b7b78344f2abed1ea84
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "100382027"
---
# <a name="set-up-github-enterprise-server-on-your-azure-vmware-solution-private-cloud"></a>Настройка сервера GitHub Enterprise в частном облаке решения Azure VMware

В этой статье описано, как настроить GitHub Enterprise Server, локальную версию [GitHub.com](https://github.com/), в частном облаке решения Azure VMware. Здесь мы обсудим экземпляр сервера GitHub Enterprise Server, который может обслуживать до 3 000 разработчиков, использующих до 25 заданий в минуту на действия GitHub. Он включает в себя установку (на момент написания статьи) *предварительных* версий функций, таких как действия GitHub. Чтобы настроить настройку в соответствии с конкретными потребностями, ознакомьтесь с требованиями, приведенными в подразделе [Установка GitHub Enterprise Server в VMware](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#hardware-considerations).

## <a name="before-you-begin"></a>Перед началом

Для GitHub Enterprise Server требуется действительный лицензионный ключ. Вы можете зарегистрироваться для получения [пробной лицензии](https://enterprise.github.com/trial). Если вы хотите расширить возможности сервера GitHub Enterprise с помощью интеграции, вы можете воспользоваться бесплатной лицензией разработчика на пять мест. Примените эту лицензию с помощью [партнерской программы GitHub](https://partner.github.com/).

## <a name="installing-github-enterprise-server-on-vmware"></a>Установка GitHub Enterprise Server в VMware

Скачайте [Текущий выпуск GitHub Enterprise Server](https://enterprise.github.com/releases/2.19.0/download) для VMware ESXi/VSPHERE (OVA) и разверните скачанный [шаблон OVA](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-17BEDA21-43F6-41F4-8FB2-E01D275FE9B4.html) .

:::image type="content" source="media/github-enterprise-server/github-options.png" alt-text="Выберите Запуск GitHub локально или в облаке.":::  

:::image type="content" source="media/github-enterprise-server/deploy-ova-template.png" alt-text="Разверните шаблон OVA.":::  

Укажите понятное имя для новой виртуальной машины, например Гисубентерприсесервер. В имя виртуальной машины не нужно включать сведения о выпуске, так как эти сведения устаревают при обновлении экземпляра. Выберите все значения по умолчанию (в ближайшее время изменяйте эти сведения) и дождитесь завершения импорта OVA.

После импорта [Настройте конфигурацию оборудования](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#creating-the-github-enterprise-server-instance) в соответствии с вашими потребностями. В нашем примере сценария потребуется следующая конфигурация.

| Ресурс | Стандартная установка | Стандартная настройка + "компоненты бета-версии" (действия) |
| --- | --- | --- |
| Число виртуальных ЦП | 4 | 8 |
| Память | 32 ГБ | 61 ГБ |
| Подключаемое хранилище | 250 ГБ | 300 ГБ |
| Корневое хранилище | 200 ГБ | 200 ГБ |

Однако ваши потребности могут отличаться. Сведения об [установке GitHub Enterprise Server в VMware](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#hardware-considerations)см. в руководстве по оборудованию. См. также [Добавление ресурсов ЦП или памяти для VMware](https://docs.github.com/en/enterprise/admin/enterprise-management/increasing-cpu-or-memory-resources#adding-cpu-or-memory-resources-for-vmware) , чтобы настроить конфигурацию оборудования в зависимости от конкретной ситуации.

## <a name="configuring-the-github-enterprise-server-instance"></a>Настройка экземпляра GitHub Enterprise Server

:::image type="content" source="media/github-enterprise-server/install-github-enterprise.png" alt-text="Установите GitHub Enterprise.":::  

После включения новой подготовленной виртуальной машины [Настройте ее через браузер](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#configuring-the-github-enterprise-server-instance). Вам потребуется отправить файл лицензии и задать пароль консоли управления. Обязательно запишите этот пароль в безопасном месте.

:::image type="content" source="media/github-enterprise-server/ssh-access.png" alt-text="Доступ к консоли администрирования через SSH.":::    

Рекомендуется по крайней мере выполнить следующие действия:

1. Отправьте открытый ключ SSH в консоль управления, чтобы [получить доступ к административной оболочке через SSH](https://docs.github.com/en/enterprise/admin/configuration/accessing-the-administrative-shell-ssh). 

2. [Настройте TLS на своем экземпляре](https://docs.github.com/en/enterprise/admin/configuration/configuring-tls) , чтобы можно было использовать сертификат, подписанный доверенным центром сертификации.

:::image type="content" source="media/github-enterprise-server/configuring-your-instance.png" alt-text="Настройка экземпляра.":::

Примените свои параметры.  Пока экземпляр перезапускается, вы можете перейти к следующему шагу, **настройке хранилища BLOB-объектов для действий GitHub**.

:::image type="content" source="media/github-enterprise-server/create-admin-account.png" alt-text="Создайте учетную запись администратора.":::

После перезапуска экземпляра можно создать новую учетную запись администратора на экземпляре. Также обязательно запишите пароль этого пользователя.

### <a name="other-configuration-steps"></a>Другие шаги настройки

Чтобы защитить экземпляр для использования в рабочей среде, рекомендуется выполнить следующие дополнительные действия по установке:

1. Настроить [высокий уровень доступности](https://help.github.com/enterprise/admin/guides/installation/configuring-github-enterprise-for-high-availability/) для защиты:

    - Сбои программного обеспечения (уровень ОС или приложения)
    - Сбои оборудования (хранилище, ЦП, ОЗУ и т. д.)
    - Сбои системы узла виртуализации
    - Логическая или физически разорванная сеть

2. [Настройте](https://docs.github.com/en/enterprise/admin/configuration/configuring-backups-on-your-appliance) [Служебные программы резервного копирования](https://github.com/github/backup-utils), указав версии моментальных снимков для аварийного восстановления, размещенные в доступности, отдельно от основного экземпляра.
3. [Настройте изоляцию поддомена](https://docs.github.com/en/enterprise/admin/configuration/enabling-subdomain-isolation), используя действительный сертификат TLS, чтобы устранить неполадки межсайтовых сценариев и другие связанные уязвимости.

## <a name="configuring-blob-storage-for-github-actions"></a>Настройка хранилища BLOB-объектов для действий GitHub

> [!NOTE]
> [В настоящее время действия GitHub доступны в виде ограниченной бета-версии для сервера GitHub Enterprise Server версии 2,22](https://docs.github.com/en/enterprise/admin/github-actions).

Внешнее хранилище BLOB-объектов необходимо для включения действий GitHub на сервере GitHub Enterprise Server (в настоящее время доступен в качестве функции бета-версии). Это внешнее хранилище BLOB-объектов используется действиями для хранения артефактов и журналов. Действия на сервере GitHub Enterprise Server [поддерживают хранилище BLOB-объектов Azure в качестве поставщика хранилища](https://docs.github.com/en/enterprise/admin/github-actions/enabling-github-actions-and-configuring-storage#about-external-storage-requirements) (и другие). Итак, мы подготавливаем новую учетную запись хранения Azure с [типом учетной записи хранения](../storage/common/storage-account-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-storage-accounts) блобстораже:

:::image type="content" source="media/github-enterprise-server/storage-account.png" alt-text="Подготавливает учетную запись хранилища BLOB-объектов Azure.":::

После завершения развертывания нового ресурса Блобстораже скопируйте и запишите строку подключения (доступно в разделе ключи доступа). Эта строка будет потребоваться чуть ниже.

## <a name="setting-up-the-github-actions-runner"></a>Настройка средства запуска действий GitHub

> [!NOTE]
> [В настоящее время действия GitHub доступны в виде ограниченной бета-версии для сервера GitHub Enterprise Server версии 2,22](https://docs.github.com/en/enterprise/admin/github-actions).

На этом этапе у вас должен быть установлен экземпляр GitHub Enterprise Server с созданной учетной записью администратора. Кроме того, необходимо иметь внешнее хранилище BLOB-объектов, которое действия GitHub будут использовать для сохраняемости.

Теперь создадим место для выполнения действий GitHub. Опять же, мы будем использовать решение Azure VMware.

Сначала создадим новую виртуальную машину в кластере. Мы будем основывать виртуальную машину на [последнем выпуске Ubuntu Server](http://releases.ubuntu.com/20.04.1/).

:::image type="content" source="media/github-enterprise-server/provision-new-vm.png" alt-text="Подготавливает новую виртуальную машину.":::

:::image type="content" source="media/github-enterprise-server/provision-new-vm-2.png" alt-text="Подготавливает новую виртуальную машину шаг 2.":::

После создания виртуальной машины включите ее и подключите к ней через SSH.

Затем установите приложение [Runner Actions](https://github.com/actions/runner) , которое запускает задание из рабочего процесса GitHub. Найдите и скачайте последнюю версию пакета средств выполнения действий Linux x64 на [странице releases](https://github.com/actions/runner/releases) или запустите следующий быстрый сценарий. Для этого скрипта требуется, чтобы на виртуальной машине присутствовали как [JQ](https://stedolan.github.io/jq/) , так и.

`LATEST\_RELEASE\_ASSET\_URL=$( curl https://api.github.com/repos/actions/runner/releases/latest | \`

`  jq -r '.assets | .[] | select(.name | match("actions-runner-linux-arm64")) | .url' )`

`DOWNLOAD\_URL=$( curl $LATEST\_RELEASE\_ASSET\_URL | \`

`  jq -r '.browser\_download\_url' )`

`curl -OL $DOWNLOAD\_URL`

Теперь у вас должен быть локальный файл на виртуальной машине, Actions-Runner-Linux-arm64- \* . tar. gz. Извлеките этот tarball локально:

`tar xzf actions-runner-linux-arm64-\*.tar.gz`

Это извлечение распаковать несколько файлов локально, включая `config.sh` `run.sh` Скрипт и, который мы вернемся к началу.

## <a name="enabling-github-actions"></a>Включение действий GitHub

> [!NOTE]
> [В настоящее время действия GitHub доступны в виде ограниченной бета-версии для сервера GitHub Enterprise Server версии 2,22](https://docs.github.com/en/enterprise/admin/github-actions).

Почти готово! Давайте настроим и активируйте действия GitHub на экземпляре сервера GitHub Enterprise. Необходимо [получить доступ к административной оболочке экземпляра GitHub Enterprise Server по протоколу SSH](https://docs.github.com/en/enterprise/admin/configuration/accessing-the-administrative-shell-ssh), а затем выполнить следующие команды:

`# set an environment variable containing your Blob storage connection string`

`export CONNECTION\_STRING="<your connection string from the blob storage step>"`

`# configure actions storage`

`ghe-config secrets.actions.storage.blob-provider azure`

`ghe-config secrets.actions.storage.azure.connection-string "$CONNECTION\_STRING"`

`# apply these settings`

`ghe-config-apply`

`# execute a precheck, this install additional software required by Actions on GitHub Enterprise Server`

`ghe-actions-precheck -p azure -cs "$CONNECTION\_STRING"`

`# enable actions, and re-apply the config`

`ghe-config app.actions.enabled true`

`ghe-config-apply`

Следующее выполнение:

`ghe-actions-check -s blob`

Вы должны увидеть выходные данные: "хранилище BLOB-объектов работоспособно".

Теперь, когда действия GitHub настроены, включите ее для пользователей. Войдите на свой экземпляр сервера GitHub Enterprise Server в качестве администратора и щелкните ![ значок Rocket.](media/github-enterprise-server/rocket-icon.png) в верхнем правом углу любой страницы. На левой боковой панели выберите **Корпоративный обзор**, затем **политики**, **действия** и выберите параметр, чтобы **включить действия для всех организаций**.

Затем настройте средство выполнения на вкладке **средства саморазмещения** . В раскрывающемся списке выберите **Добавить новое** и **новое** средство выполнения.

На следующей странице будет представлен набор команд для выполнения. нам просто нужно скопировать команду для **настройки** средства запуска, например:

`./config.sh --url https://10.1.1.26/enterprises/octo-org --token AAAAAA5RHF34QLYBDCHWLJC7L73MA`

Скопируйте `config.sh` команду и вставьте ее в сеанс в средстве выполнения действий (созданном ранее).

:::image type="content" source="media/github-enterprise-server/actions-runner.png" alt-text="Средство выполнения действий.":::

Чтобы *запустить средство выполнения* , используйте команду Run.sh:

:::image type="content" source="media/github-enterprise-server/run-runner.png" alt-text="Запустите средство выполнения.":::

Чтобы сделать это средство можно доступным для организаций в Организации, измените доступ к нему.

:::image type="content" source="media/github-enterprise-server/edit-runner-access.png" alt-text="Изменение доступа к средству выполнения.":::

Здесь мы предоставляем доступ ко всем организациям, но вы можете ограничить доступ к подмножеству организаций и даже к конкретным репозиториям.

## <a name="optional-configuring-github-connect"></a>Используемых Настройка GitHub Connect

Хотя этот шаг является необязательным, его рекомендуется использовать, если планируется использование действий с открытым кодом, доступных в GitHub.com. Она позволяет создавать решения для других пользователей, ссылаясь на эти многократно используемые действия в рабочих процессах.

Чтобы включить GitHub Connect, выполните действия, описанные в разделе [Включение автоматического доступа к действиям GitHub.com с помощью GitHub Connect](https://docs.github.com/en/enterprise/admin/github-actions/enabling-automatic-access-to-githubcom-actions-using-github-connect).

После включения GitHub Connect выберите **сервер для использования действий из GitHub.com в параметре выполнения рабочего процесса** .

:::image type="content" source="media/github-enterprise-server/enable-using-actions.png" alt-text="Включите использование действий из GitHub.com в запусках рабочих процессов.":::

## <a name="setting-up-and-running-your-first-workflow"></a>Настройка и запуск первого рабочего процесса

Теперь, когда действия и подключение GitHub настроены, давайте разберем все эти работы в хорошем использовании. Ниже приведен пример рабочего процесса, который ссылается на отличное [октокит/Request-Action](https://github.com/octokit/request-action), что позволяет нам "создать скрипт" для GitHub посредством взаимодействия с помощью API GitHub на основе действий GitHub.

В этом базовом рабочем процессе мы будем использовать, `octokit/request-action` чтобы просто открыть вопрос в GitHub с помощью API.

:::image type="content" source="media/github-enterprise-server/workflow-example.png" alt-text="Пример рабочего процесса.":::

>[!NOTE]
>GitHub.com размещает действие, но при его выполнении на сервере GitHub Enterprise Server он *автоматически* использует API сервера GitHub Enterprise.

Если вы решили не включать GitHub Connect, можно использовать следующий альтернативный рабочий процесс.

:::image type="content" source="media/github-enterprise-server/workflow-example-2.png" alt-text="Альтернативный пример рабочего процесса.":::

Перейдите к репозиторию в экземпляре и добавьте указанный выше рабочий процесс как: `.github/workflows/hello-world.yml`

:::image type="content" source="media/github-enterprise-server/workflow-example-3.png" alt-text="Другой пример рабочего процесса.":::

На вкладке **действия** для репозитория дождитесь выполнения рабочего процесса.

:::image type="content" source="media/github-enterprise-server/executed-example-workflow.png" alt-text="Выполненный пример рабочего процесса.":::

Кроме того, вы можете следить за его обработкой.

:::image type="content" source="media/github-enterprise-server/workflow-processed-by-runner.png" alt-text="Рабочий процесс, обработанный средством Runner.":::

Если все успешно завершилось, в репозитории появится новая ошибка — "Hello World".

:::image type="content" source="media/github-enterprise-server/example-in-repo.png" alt-text="Пример в репозитории.":::

Поздравляем! Вы только что выполнили рабочий процесс первого действия на сервере GitHub Enterprise, работающем в частном облаке решения Azure VMware.

В этой статье мы настроили новый экземпляр GitHub Enterprise Server, самостоятельный аналог GitHub.com, поверх частного облака решения Azure VMware. Этот экземпляр включает поддержку действий GitHub и использует хранилище BLOB-объектов Azure для сохранения журналов и артефактов. Но мы просто попытаемся узнать, что можно сделать с помощью действий GitHub. Ознакомьтесь со списком действий в [Marketplace](https://github.com/marketplace)или [Создайте свой собственный](https://docs.github.com/en/actions/creating-actions).

## <a name="next-steps"></a>Дальнейшие действия

Теперь, когда вы настроили настройку сервера GitHub Enterprise Server в частном облаке решения Azure VMware, вам может потребоваться изучить следующее: 

- Начало [работы с действиями GitHub](https://docs.github.com/en/actions).
- [Присоединение к программе бета-тестирования](https://resources.github.com/beta-signup/).
- [Администрирование сервера GitHub Enterprise](https://githubtraining.github.io/admin-training/#/00_getting_started).
