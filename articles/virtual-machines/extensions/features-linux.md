---
title: Расширения и компоненты виртуальной машины Azure для Linux
description: Узнайте, какие расширения доступны для виртуальных машин Azure в Linux, сгруппированные по их обеспечению или улучшению.
ms.topic: article
ms.service: virtual-machines
ms.subservice: extensions
author: amjads1
ms.author: amjads
ms.collection: linux
ms.date: 03/30/2018
ms.openlocfilehash: 8ff7b2940ca9ce3e5dc5913c0e676a32857771e7
ms.sourcegitcommit: ed7376d919a66edcba3566efdee4bc3351c57eda
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/24/2021
ms.locfileid: "105046884"
---
# <a name="virtual-machine-extensions-and-features-for-linux"></a>Обзор расширений и компонентов виртуальных машин под управлением Linux

Расширения виртуальных машин Azure — это небольшие приложения, которые выполняют задачи настройки и автоматизации после развертывания виртуальных машин Azure. Например, если на виртуальную машину требуется установить программное обеспечение, антивирусную защиту или выполнить сценарий внутри него, можно использовать расширение виртуальной машины. Расширения виртуальных машин Azure можно запускать через интерфейс командной строки Azure, PowerShell, шаблоны Azure Resource Manager или портал Azure. Их можно включить в пакет для развертывания виртуальной машины или запустить на любой из уже установленных систем.

Эта статья содержит обзор расширений виртуальной машины Azure, предварительные требования для их использования, рекомендации по их обнаружению, удалению и администрированию. Этот документ содержит только общие сведения, так как существует множество расширений виртуальных машин с разными параметрами настройки. Информацию о нужном расширении можно найти в соответствующей документации.

## <a name="use-cases-and-samples"></a>Варианты использования и примеры

Существует несколько разных расширений ВМ Azure, которые используются в определенных сценариях. Некоторые примеры:

- Применение к виртуальной машине конфигураций требуемого состояния PowerShell с помощью расширения DSC для Linux. Подробнее см. [Общие сведения об обработчике расширения Desired State Configuration в Azure](https://github.com/Azure/azure-linux-extensions/tree/master/DSC);
- Настройка мониторинга виртуальной машины с помощью расширения виртуальной машины Microsoft Monitoring Agent. Дополнительные сведения см. в статье [Мониторинг виртуальных машин Linux в Azure](/previous-versions/azure/virtual-machines/linux/tutorial-monitor).
- Настройка мониторинга инфраструктуры Azure с помощью расширения Chef или Datadog. Дополнительную информацию см. в [документации по Chef](https://docs.chef.io/azure_portal.html) или [блоге Datadog](https://www.datadoghq.com/blog/introducing-azure-monitoring-with-one-click-datadog-deployment/).

Помимо расширений, созданных для конкретных процессов, существует расширение пользовательских сценариев для виртуальных машин под управлением Windows и Linux. Расширение пользовательских сценариев для Linux позволяет запустить на виртуальной машине любой сценарий Bash. Пользовательские сценарии могут пригодиться при проектировании развертывания Azure, для которого требуется дополнительная настройка, ее невозможно выполнить собственными средствами Azure. Дополнительные сведения см. в статье [Использование расширения пользовательских сценариев Azure на виртуальных машинах Linux](custom-script-linux.md).

## <a name="prerequisites"></a>Предварительные требования

Чтобы расширения работали на виртуальной машине, на ней нужно установить агент Linux для Azure. Некоторые расширения имеют дополнительные требования, например доступ к определенным ресурсам или зависимости.

### <a name="azure-vm-agent"></a>Агент виртуальной машины Azure

Агент виртуальной машины Azure управляет взаимодействием виртуальной машины с контроллером структуры Azure. Агент отвечает за многие функциональные аспекты развертывания виртуальных машин Azure и управления ими, включая запуск расширений виртуальной машины. Агент виртуальной машины Azure предварительно установлен во всех образах Azure Marketplace. Его также можно установить вручную в поддерживаемых операционных системах. Агент виртуальных машин Azure для Linux часто называют агентом Linux.

Сведения о поддерживаемых операционных системах и инструкции по установке см. в статье [Агент виртуальной машины Azure](agent-linux.md).

#### <a name="supported-agent-versions"></a>Поддерживаемые версии агента

Чтобы обеспечить максимальное удобство, мы предоставили минимальные версии агента. Дополнительные сведения см. в [этой статье](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support).

#### <a name="supported-oses"></a>Поддерживаемые операционные системы

Гостевой агент Linux работает на многих операционных системах, но платформа расширений налагает некоторые ограничения на совместимость расширений с операционными системами. Дополнительные сведения см. в [этой статье](https://support.microsoft.com/en-us/help/4078134/azure-extension-supported-operating-systems
).

Некоторые расширения поддерживаются не на всех операционных системах и вы можете столкнуться с *ошибкой 51 "Неподдерживаемая ОС"*. Сведения о совместимости приводятся в документации по отдельным расширениям.

#### <a name="network-access"></a>Доступ к сети

Пакеты расширений можно скачать из репозитория расширений в службе хранилища Azure. В ней также публикуются новости об актуальном состоянии расширений. Если вы используете [поддерживаемые](https://support.microsoft.com/en-us/help/4049215/extensions-and-virtual-machine-agent-minimum-version-support) версии агентов, доступ к службе хранилища Azure в регионе виртуальной машины предоставлять не обязательно, так как агент может перенаправлять свои запросы к контроллеру структуры Azure. Если вы используете неподдерживаемую версию агента, необходимо разрешить исходящий доступ от виртуальной машины к службе хранилище Azure в том же регионе.

> [!IMPORTANT]
> Если в гостевом брандмауэре вы заблокировали доступ к IP-адресу *168.63.129.16*, запуск расширения всегда будет завершаться ошибкой, независимо от описанных выше условий.

Агенты можно использовать только для скачивания пакетов расширений и отправки отчетов о состоянии. Например, если для установки расширения требуется скачать сценарий из GitHub (пользовательский сценарий) или требуется доступ к службе хранилища Azure (Azure Backup), откройте дополнительные порты в брандмауэре или в группе безопасности сети (NSG). Разные расширения имеют разные требования, ведь они по сути являются самостоятельными приложениями. Для расширений, которым нужен доступ к службе хранилища Azure, вы можете разрешить доступ с помощью тегов службы Azure NSG для [хранилища](../../virtual-network/network-security-groups-overview.md#service-tags).

Агент Linux поддерживает перенаправление запросов через прокси-сервер. Однако при такой поддержке прокси-сервера не применяются расширения. Чтобы работать с прокси-сервером, необходимо настроить каждое отдельное расширение.

## <a name="discover-vm-extensions"></a>Поиск расширений ВМ

Существует множество разных расширений виртуальных машин, которые можно использовать с виртуальными машинами Azure. Чтобы просмотреть полный список, выполните команду [az vm extension image list](/cli/azure/vm/extension/image#az-vm-extension-image-list). В следующем примере перечисляются все доступные расширения в расположении *westus* :

```azurecli
az vm extension image list --location westus --output table
```

## <a name="run-vm-extensions"></a>Запуск расширений ВМ

Расширения виртуальных машин Azure можно запускать на имеющихся виртуальных машинах, например для изменения конфигурации или восстановления подключения на ранее развернутой виртуальной машине. Также расширения можно использовать при развертывании с помощью шаблонов Azure Resource Manager. Включив расширения в шаблоны Resource Manager, вы сможете развернуть виртуальные машины Azure сразу со всеми нужными настройками, без дополнительных действий после развертывания.

Запустить расширение на имеющейся виртуальной машине можно несколькими способами.

### <a name="azure-cli"></a>Azure CLI

Расширения виртуальных машин Azure можно запускать на имеющихся виртуальных машинах с помощью команды [az vm extension set](/cli/azure/vm/extension#az-vm-extension-set). В следующем примере выполняется запуск расширения пользовательского скрипта для виртуальной машины с именем *myVM* в группе ресурсов с именем *myResourceGroup*. Замените пример имени группы ресурсов, имя виртуальной машины и скрипт для запуска (HTTPS: \/ /RAW.githubusercontent.com/Me/Project/Hello.sh) собственными сведениями. 

```azurecli
az vm extension set `
  --resource-group myResourceGroup `
  --vm-name myVM `
  --name customScript `
  --publisher Microsoft.Azure.Extensions `
  --settings '{"fileUris": ["https://raw.githubusercontent.com/me/project/hello.sh"],"commandToExecute": "./hello.sh"}'
```

Если расширение работает правильно, вы должны увидеть результат, аналогичный приведенному ниже.

```bash
info:    Executing command vm extension set
+ Looking up the VM "myVM"
+ Installing extension "CustomScript", VM: "mvVM"
info:    vm extension set command OK
```

### <a name="azure-portal"></a>Портал Azure

Расширения виртуальных машин можно устанавливать на имеющиеся виртуальные машины через портал Azure. Выберите на портале нужную виртуальную машину, затем щелкните **Расширения** и **Добавить**. Выберите нужное расширение из списка доступных расширений и следуйте инструкциям мастера.

На рисунке ниже показана установка расширения пользовательских сценариев для Linux с помощью портала Azure.

![Установка расширения пользовательских сценариев](./media/features-linux/installscriptextensionlinux.png)

### <a name="azure-resource-manager-templates"></a>Шаблоны Azure Resource Manager

Расширения ВМ можно добавлять в шаблоны Azure Resource Manager. Так они смогут выполняться при развертывании соответствующего шаблона. Развертывая расширение вместе с шаблоном, можно создать полностью настроенное развертывание Azure. Например, следующий JSON взят из шаблона диспетчер ресурсов, который развертывает набор виртуальных машин с балансировкой нагрузки и базу данных SQL Azure, а затем устанавливает приложение .NET Core на каждой виртуальной машине. Расширение ВМ отвечает за установку программного обеспечения.

Дополнительные сведения вы можете получить, ознакомившись с полным [шаблоном Resource Manager](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-linux).

```json
{
    "apiVersion": "2015-06-15",
    "type": "extensions",
    "name": "config-app",
    "location": "[resourceGroup().location]",
    "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
    ],
    "tags": {
    "displayName": "config-app"
    },
    "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
        ]
    },
    "protectedSettings": {
        "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
    }
}
```

Дополнительные сведения о создании шаблонов Resource Manager см. в разделе [Расширения](../windows/template-description.md#extensions).

## <a name="secure-vm-extension-data"></a>Защита данных в расширениях ВМ

Для запуска расширений виртуальных машин может потребоваться конфиденциальная информация, например учетные данные, имена учетных записей хранения или ключи доступа к учетной записи хранения. Многие расширения виртуальных машин поддерживают защищенную конфигурацию, при использовании которой данные хранятся в зашифрованном виде и расшифровываются только на целевой виртуальной машине. Каждое расширение имеет собственную схему защищенной конфигурации, которая описывается в документации по этим расширениям.

В следующем примере показан экземпляр расширения пользовательских сценариев для Linux. Выполняемая команда содержит набор учетных данных. В этом примере выполняемая команда не зашифрована.

```json
{
  "apiVersion": "2015-06-15",
  "type": "extensions",
  "name": "config-app",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
      ],
      "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
  }
}
```

Переместив свойство **commandToExecute** в конфигурацию **protected**, вы защитите данные из строки выполнения, как показано в следующем примере:

```json
{
  "apiVersion": "2015-06-15",
  "type": "extensions",
  "name": "config-app",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', concat(variables('vmName'),copyindex()))]"
  ],
  "tags": {
    "displayName": "config-app"
  },
  "properties": {
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
      "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
      ]
    },
    "protectedSettings": {
      "commandToExecute": "[concat('sudo sh config-music.sh ',variables('musicStoreSqlName'), ' ', parameters('adminUsername'), ' ', parameters('sqlAdminPassword'))]"
    }
  }
}
```

### <a name="how-do-agents-and-extensions-get-updated"></a>Как обновляются агенты и расширения?

Агенты и расширения используют общий механизм обновления. Некоторые обновления не требуют дополнительных правил брандмауэра.

Когда обновление становится доступно, оно устанавливается на виртуальных машинах только при внесении изменений в расширения или другие элементы модели виртуальной машины, как например:

- Диски данных
- Расширения
- контейнер диагностики загрузки;
- секреты гостевой ОС;
- Размер виртуальной машины
- сетевой профиль.

Издатели предоставляют обновления для разных регионов в разное время, а значит виртуальные машины в нескольких регионах могут одновременно использовать разные версии.

#### <a name="agent-updates"></a>Обновления агентов

Агент виртуальной ашины Linux содержит *код подготовки агента* и *код обработки расширения* в одном пакете, который нельзя разделить. Вы можете отключить *агент подготовки*, если необходимо подготовить Azure с помощью cloud-init. Чтобы сделать это, ознакомьтесь со статьей об [использовании cloud-init](../linux/using-cloud-init.md).

Поддерживаемые версии агентов могут использовать автоматические обновления. Единственный код, который можно обновить, — это *код обработки расширения*, а не код подготовки. *Код подготовки агента* запускается только один раз.

*Код обработки расширений* отвечает за взаимодействие со структурой Azure и обработку операций расширения виртуальных машин, такие как установка, отправка отчетов о состоянии, обновление и удаление отдельных расширений. Обновления содержат исправления безопасности, исправления ошибок и улучшения в *коде обработки расширения*.

При установке агента создается родительская управляющая программа. Эта родительская программа затем порождает дочерний процесс, используемый для обработки расширений. Если для агента доступно обновление, оно скачивается, родительская программа останавливает дочерний процесс, обновляет его, а затем перезапускает. При возникновении проблем с обновлением выполняется откат родительского процесса к предыдущей дочерней версии.

Родительский процесс не может быть обновлен автоматически. Его можно обновить только с помощью обновления пакета дистрибутива.

Чтобы узнать установленную версию, проверьте `waagent` следующим образом:

```bash
waagent --version
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```bash
WALinuxAgent-2.2.17 running on ubuntu 16.04
Python: 3.6.0
Goal state agent: 2.2.18
```

В предыдущем примере выходных данных, *WALinuxAgent-2.2.17* — это версия родительской программы или развернутого пакета.

Версия агента состояния цели обновляется автоматически.

Настоятельно рекомендуется всегда включать автоматическое обновление агента [AutoUpdate.Enabled=y](./update-linux-agent.md). Если эта функция отключена, вам потребуется обновлять агент вручную и избегать ошибок и исправлений безопасности.

#### <a name="extension-updates"></a>Обновление расширений

Когда становится доступно обновление расширения, агент Linux скачивает его и обновляет расширение. Автоматические обновления расширений классифицируются как *дополнения* или *исправления*. При подготовке расширения вы можете согласиться на установку обновлений категории *Дополнения* либо отказаться от них. В следующем примере показано, как можно автоматически обновлять дополнительные версии в шаблоне Resource Manager с помощью параметра *autoUpgradeMinorVersion": true,*:

```json
    "publisher": "Microsoft.Azure.Extensions",
    "type": "CustomScript",
    "typeHandlerVersion": "2.0",
    "autoUpgradeMinorVersion": true,
    "settings": {
        "fileUris": [
        "https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-linux/scripts/config-music.sh"
        ]
    },
```

Мы настоятельно рекомендуем включить автоматическое обновление для развертываний расширений, чтобы своевременно получать дополнительные исправления ошибок. У вас нет возможности отказаться от обновлений, которые содержат исправления безопасности или значительных ошибок.

### <a name="how-to-identify-extension-updates"></a>Как определить параметры обновления расширений

#### <a name="identifying-if-the-extension-is-set-with-autoupgrademinorversion-on-a-vm"></a>Определение значения параметра autoUpgradeMinorVersion для расширения на виртуальной машине

Модель виртуальной машины позволяет узнать, был ли установлен параметр autoUpgradeMinorVersion при подготовке расширения. Для этого выполните команду [az vm show](/cli/azure/vm#az-vm-show), указав группу ресурсов и имя виртуальной машины следующим образом:

```azurecli
az vm show --resource-group myResourceGroup --name myVM
```

Выходные данные в следующем примере указывают, что параметр *autoUpgradeMinorVersion* имеет значение *true*:

```json
  "resources": [
    {
      "autoUpgradeMinorVersion": true,
      "forceUpdateTag": null,
      "id": "/subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM/extensions/CustomScriptExtension",
```

#### <a name="identifying-when-an-autoupgrademinorversion-occurred"></a>Определение наличия установленного обновления autoUpgradeMinorVersion

Чтобы узнать, когда становится доступно обновление расширения, изучите журналы агента на виртуальной машине, расположенные по адресу */var/log/waagent.log*.

В приведенном ниже примере на виртуальной машине установлено расширение *Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025*. Для расширения *Microsoft.OSTCExtensions.LinuxDiagnostic 2.3.9027* было доступно исправление:

```bash
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Expected handler state: enabled
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Decide which version to use
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Use version: 2.3.9027
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Current handler state is: NotInstalled
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Download extension package
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Unpack extension package
INFO Event: name=Microsoft.OSTCExtensions.LinuxDiagnostic, op=Download, message=Download succeeded
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Initialize extension directory
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Update settings file: 0.settings
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025] Disable extension.
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9025] Launch command:diagnostic.py -disable
...
INFO Event: name=Microsoft.OSTCExtensions.LinuxDiagnostic, op=Disable, message=Launch command succeeded: diagnostic.py -disable
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Update extension.
INFO [Microsoft.OSTCExtensions.LinuxDiagnostic-2.3.9027] Launch command:diagnostic.py -update
2017/08/14 20:21:57 LinuxAzureDiagnostic started to handle.
```

## <a name="agent-permissions"></a>Разрешения для агента

Чтобы агент работал должным образом, он должен запускаться *привилегированным пользователем*.

## <a name="troubleshoot-vm-extensions"></a>Устранение неполадок с расширениями ВМ

Для каждого расширения ВМ предусмотрена своя процедура устранения неполадок. Например, если вы используете расширение пользовательских сценариев, сведения о выполнении сценария можно найти на той виртуальной машине, где запускалось расширение. Действия по устранению неполадок для конкретных расширений описаны в документации по соответствующим расширениям.

Следующие инструкции по устранению неполадок применимы ко всем расширениям виртуальных машин.

1. Чтобы проверить журнал агента Linux, найдите в файле */var/log/waagent.log* действия, которые выполнялись при подготовке расширения.

2. Подробные сведения вы можете найти в текущих журналах расширения в каталоге */var/log/azure/\<extensionName>*.

3. Коды ошибок, известные проблемы и другие данные для устранения неполадок вы найдете в документации по конкретному расширению.

3. Просмотрите системные журналы. Проверьте наличие других операций, которые могут мешать работе расширения, например длительные установки других приложений, требующие монопольного доступа к диспетчеру пакетов.

### <a name="common-reasons-for-extension-failures"></a>Распространенные причины сбоев расширений

1. Расширению предоставляется на выполнение всего 20 минут (за исключением расширения пользовательских сценариев, Chef и DSC, которые могут работать 90 минут). Если в вашем развертывании требуется больше времени, для расширения фиксируется превышение времени ожидания. Это может быть вызвано нехваткой ресурсов на виртуальной машине или выполнением на той же виртуальной машине во время работы приложения других задач конфигурации или запуска, которые потребляют большие объемы ресурсов.

2. Не соблюдаются минимальные требования. Некоторые расширения имеют зависимость от номера SKU виртуальной машины, например от образа HPC. Расширения могут иметь требования к сетевому доступу, такие как наличие связи со службой хранилища Azure или другими общедоступными службами. Работе также могут помешать отсутствие доступа к репозиторию пакетов, нехватка места на диске или ограничения безопасности.

3. Монопольный доступ к диспетчеру пакетов. В некоторых случаях длительная настройка виртуальной машины может конфликтовать с установкой расширения, если оба этих процесса требуют монопольный доступ к диспетчеру пакетов.

### <a name="view-extension-status"></a>Просмотр состояния расширения

Запустив расширение ВМ на виртуальной машине, выполните команду [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view), чтобы получить состояние расширения следующим образом:

```azurecli
az vm get-instance-view \
    --resource-group rgName \
    --name myVM \
    --query "instanceView.extensions"
```

Вы должны увидеть результат, аналогичный приведенному ниже.

```bash
  {
    "name": "customScript",
    "statuses": [
      {
        "code": "ProvisioningState/failed/0",
        "displayStatus": "Provisioning failed",
        "level": "Error",
        "message": "Enable failed: failed to execute command: command terminated with exit status=127\n[stdout]\n\n[stderr]\n/bin/sh: 1: ech: not found\n",
        "time": null
      }
    ],
    "substatuses": null,
    "type": "Microsoft.Azure.Extensions.customScript",
    "typeHandlerVersion": "2.0.6"
  }
```

Кроме того, состояние выполнения расширения можно найти на портале Azure. Чтобы просмотреть состояние расширения, выберите виртуальную машину, затем щелкните **Расширения** и найдите нужное расширение.

### <a name="rerun-a-vm-extension"></a>Повторный запуск расширения ВМ

В некоторых случаях может потребоваться перезапуск расширения виртуальной машины. Можно сначала удалить расширение, затем повторно запустить его любым удобным способом. Чтобы удалить расширение, выполните команду [az vm extension delete](/cli/azure/vm/extension#az-vm-extension-delete) следующим образом:

```azurecli
az vm extension delete \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --name customScript
```

Кроме того, вы можете удалить расширение через портал Azure, сделав следующее:

1. Выберите виртуальную машину.
2. Выберите **Расширения**.
3. Выберите нужное расширение.
4. Выберите **Удалить**.

## <a name="common-vm-extension-reference"></a>Справочные материалы для всех расширений ВМ

| Имя расширения | Описание | Дополнительные сведения |
| --- | --- | --- |
| Расширение пользовательских сценариев для Linux |Выполняет сценарии на виртуальных машинах Azure. |[Расширение пользовательских сценариев для Linux](custom-script-linux.md) |
| Расширение для доступа к виртуальной машине |Восстанавливает доступ к виртуальной машине Azure. |[Расширение для доступа к виртуальной машине](https://github.com/Azure/azure-linux-extensions/tree/master/VMAccess) |
| Расширение службы "Диагностика Azure" |Управляет системой диагностики Azure |[Расширение службы "Диагностика Azure"](https://azure.microsoft.com/blog/windows-azure-virtual-machine-monitoring-with-wad-extension/) |
| Расширение Azure VM Access |Управляет пользователями и учетными данными. |[Расширение VM Access для Linux](https://azure.microsoft.com/blog/using-vmaccess-extension-to-reset-login-credentials-for-linux-vm/) |

## <a name="next-steps"></a>Дальнейшие действия

Дополнительные сведения о расширениях виртуальных машин см. в статье c [обзором расширений и компонентов виртуальной машины Azure](overview.md).