---
title: Создание runbook Python в службе автоматизации Azure
description: Эта статья содержит инструкции по созданию, тестированию и публикации простого runbook на Python.
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: e12327651165606e6a9b571d410f547a09a8ec8e
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "87847930"
---
# <a name="tutorial-create-a-python-runbook"></a>Руководство по Создание модуля runbook в Python

В этом руководстве описана процедура создания [модуля Runbook Python](../automation-runbook-types.md#python-runbooks) в службе автоматизации Azure. Модули Runbook Python компилируются в среде Python 2. Код модуля Runbook можно редактировать в текстовом редакторе на портале Azure.

> [!div class="checklist"]
> * Создание простого модуля runbook в Python
> * Тестирование и публикация runbook
> * Выполнение задания модуля runbook и отслеживание его состояния
> * Обновление runbook для запуска виртуальной машины Azure с указанными в runbook параметрами

> [!NOTE]
> Использование веб-перехватчиков для запуска runbook Python не поддерживается.

## <a name="prerequisites"></a>Предварительные требования

Для работы с этим учебником требуется:

- Подписка Azure. Если у вас ее нет, [активируйте преимущества для подписчиков MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) или [зарегистрируйте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Учетная запись службы автоматизации](../index.yml) , чтобы хранить модуль Runbook и выполнять проверку подлинности ресурсов Azure. Эта учетная запись должна иметь разрешение на запуск и остановку виртуальной машины.
- Виртуальная машина Azure. Это не должна быть рабочая виртуальная машина, так как в процессе изучения этого руководства ее нужно будет остановить и запустить заново.

## <a name="create-a-new-runbook"></a>Создание модуля Runbook

Для начала создайте простой модуль runbook, выводящий на экран текст *Hello World*.

1. На портале Azure выберите свою учетную запись службы автоматизации.

    Страница учетной записи в службе автоматизации позволяет быстро получить представление о ресурсах, доступных в этой учетной записи. Некоторые ресурсы уже должны быть доступны. Большинство из них — это модули, которые добавляются в каждую новую учетную запись службы автоматизации по умолчанию. Кроме того, вам потребуется ресурс учетных данных, упомянутый в [предварительных требованиях](#prerequisites).

2. Щелкните **Модули Runbook** в разделе **Автоматизация процессов**, чтобы открыть список модулей runbook.

3. Чтобы создать новый модуль runbook, выберите **Добавить Runbook**.

4. Присвойте модулю Runbook имя **MyFirstRunbook-Python**.

5. Для поля **Тип Runbook** выберите значение **Python 2**.

6. Щелкните **Создать** , чтобы создать модуль Runbook и открыть текстовый редактор.

## <a name="add-code-to-the-runbook"></a>Добавление кода в модуль Runbook

Теперь добавьте простую команду для вывода текста `Hello World`.

```python
print("Hello World!")
```

Нажмите кнопку **Сохранить**, чтобы сохранить модуль runbook.

## <a name="test-the-runbook"></a>Тестирование модуля Runbook

Прежде чем опубликовать модуль runbook и, таким образом, сделать его доступным для рабочей среды, необходимо проверить, правильно ли он работает. Чтобы протестировать модуль Runbook, нужно запустить его черновую версию и проверить его работу в интерактивном режиме.

1. Щелкните **Область тестирования**.

2. Щелкните **Пуск** , чтобы начать тестирование. Активным должен быть только этот параметр.

3. При этом создается [задание Runbook](../automation-runbook-execution.md) и отображается его состояние.
   Сначала задание получает состояние "В очереди". Это означает, что задание ожидает доступа к рабочей роли runbook в облаке. Когда рабочая роль запросит задание, оно получит состояние "Запуск", а после начала выполнения runbook устанавливается состояние "Выполняется".

4. Когда задание модуля Runbook будет выполнено, на экране появится результат. В нашем примере отображается текст `Hello World`.

5. Закройте область тестирования, чтобы вернуться на холст.

## <a name="publish-and-start-the-runbook"></a>Публикация и запуск модуля Runbook

Модуль runbook, который вы только что создали, все еще находится в режиме черновика. Прежде чем запустить модуль в рабочей среде, его нужно опубликовать. При публикации модуля Runbook существующая опубликованная версия перезаписывается черновой. В нашем случае опубликованной версии не существует, так как модуль runbook создан только что.

1. Щелкните **Опубликовать**, чтобы опубликовать модуль Runbook, а затем нажмите кнопку **Да** в появившемся запросе.

2. Если прокрутить экран влево до элемента runbook на странице "Модули Runbook", для параметра **Состояние создания** должно отображаться значение **Опубликовано**.

3. Прокрутите экран вправо до области **MyFirstRunbook-Python**.

   Параметры в верхней части экрана позволяют запускать и просматривать модуль runbook или назначать его запуск на определенное время в будущем.

4. Щелкните **Запустить**, а затем нажмите кнопку **ОК**, когда откроется колонка "Запуск Runbook".

5. Откроется область заданий с созданным заданием runbook. Эту область можно закрыть, но пока лучше оставить ее открытой, чтобы следить за ходом выполнения задания.

6. Состояние задания отображается в поле **Сводка по заданию** и отражает состояния, которые вы наблюдали при тестировании модуля runbook.

7. Как только состояние runbook изменится на "Выполнено", щелкните **Выходные данные**. Откроется область "Выходные данные", где отображается `Hello World`.

8. Закройте область выходных данных.

9. Щелкните **Все журналы** , чтобы открыть область "Потоки" для задания Runbook. В потоке выходных данных должно отобразиться только `Hello World`. Но на панели "Потоки" могут быть и другие потоки задания runbook, например "Подробные сведения" и "Ошибка", если этот модуль runbook записывает в них какие-то данные.

10. Закройте область потоков и область заданий, чтобы вернуться в область MyFirstRunbook-Python.

11. Щелкните **Задания**, чтобы открыть область "Задания" для этого модуля runbook. Откроется страница всех заданий, созданных этим модулем runbook. В нем должно быть только одно задание, так как вы запустили задание только один раз.

12. Если щелкнуть это задание, откроется та же область заданий, что и при запуске модуля runbook. С помощью этой области можно вернуться назад и просмотреть сведения о любом задании, созданном для конкретного модуля runbook.

## <a name="add-authentication-to-manage-azure-resources"></a>Добавление аутентификации для управления ресурсами Azure

Вы протестировали и опубликовали свой модуль runbook, но пока он не выполняет никаких полезных действий. Нужно, чтобы он управлял ресурсами Azure.
Для этого скрипт должен пройти проверку подлинности с учетными данными из учетной записи службы автоматизации. Вы можете использовать [пакет служебной программы службы автоматизации Azure](https://github.com/azureautomation/azure_automation_utility) для упрощения проверки подлинности и взаимодействия с ресурсами Azure.

> [!NOTE]
> Учетную запись службы автоматизации необходимо создать с помощью функции субъекта-службы, чтобы в ней существовал сертификат запуска от имени.
> Если учетная запись службы автоматизации не была создана с помощью субъекта-службы, то для аутентификации можно использовать инструкции из статьи [Проверка подлинности с помощью библиотек управления Azure для Python](/azure/python/python-sdk-azure-authenticate).

1. Откройте текстовый редактор, щелкнув **Изменить** в области MyFirstRunbook-Python.

2. Добавьте следующий код для аутентификации в Azure:

   ```python
   import os
   from azure.mgmt.compute import ComputeManagementClient
   import azure.mgmt.resource
   import automationassets

   def get_automation_runas_credential(runas_connection):
       from OpenSSL import crypto
       import binascii
       from msrestazure import azure_active_directory
       import adal

       # Get the Azure Automation RunAs service principal certificate
       cert = automationassets.get_automation_certificate("AzureRunAsCertificate")
       pks12_cert = crypto.load_pkcs12(cert)
       pem_pkey = crypto.dump_privatekey(crypto.FILETYPE_PEM,pks12_cert.get_privatekey())

       # Get run as connection information for the Azure Automation service principal
       application_id = runas_connection["ApplicationId"]
       thumbprint = runas_connection["CertificateThumbprint"]
       tenant_id = runas_connection["TenantId"]

       # Authenticate with service principal certificate
       resource ="https://management.core.windows.net/"
       authority_url = ("https://login.microsoftonline.com/"+tenant_id)
       context = adal.AuthenticationContext(authority_url)
       return azure_active_directory.AdalAuthentication(
       lambda: context.acquire_token_with_client_certificate(
               resource,
               application_id,
               pem_pkey,
               thumbprint)
       )

   # Authenticate to Azure using the Azure Automation RunAs service principal
   runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
   azure_credential = get_automation_runas_credential(runas_connection)
   ```

## <a name="add-code-to-create-python-compute-client-and-start-the-vm"></a>Добавление кода для создания вычислительного клиента Python и запуска виртуальной машины

Для работы с виртуальными машинами Azure создайте экземпляр [вычислительного клиента Azure для Python](/python/api/azure-mgmt-compute/azure.mgmt.compute.computemanagementclient).

Используйте вычислительный клиент для запуска виртуальной машины. Добавьте в модуль Runbook следующий код:

```python
# Initialize the compute management client with the RunAs credential and specify the subscription to work against.
compute_client = ComputeManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"])
)


print('\nStart VM')
async_vm_start = compute_client.virtual_machines.start(
    "MyResourceGroup", "TestVM")
async_vm_start.wait()
```

В этом фрагменте `MyResourceGroup` — это имя группы ресурсов, в которую входит виртуальная машина, а `TestVM` — это имя виртуальной машины, которую требуется запустить.

Протестируйте и выполните модуль Runbook еще раз, чтобы убедиться, что он запускает виртуальную машину.

## <a name="use-input-parameters"></a>Использование входных параметров

Сейчас в качестве имен группы ресурсов и виртуальной машины модуль Runbook использует жестко заданные значения. Теперь давайте добавим код, который извлекает эти значения из входных параметров.

Используйте переменную `sys.argv`, чтобы получить значения параметров. Добавьте следующий код в модуль Runbook сразу после других инструкций `import`:

```python
import sys

resource_group_name = str(sys.argv[1])
vm_name = str(sys.argv[2])
```

Он импортирует модуль `sys` и создает две переменные для хранения имен группы ресурсов и виртуальной машины. Обратите внимание, что элемент списка аргументов (`sys.argv[0]`) является именем сценария, а не вводится пользователем.

Теперь можно изменить две последние строки модуля runbook, чтобы вместо жестко заданных значений использовались значения входных параметров:

```python
async_vm_start = compute_client.virtual_machines.start(
    resource_group_name, vm_name)
async_vm_start.wait()
```

При запуске модуля runbook Python (со страницы "Тест" или как опубликованного модуля) можно ввести значения параметров на странице "Запуск Runbook" в разделе **Параметры**.

Когда вы начнете вводить значение в первое поле, отобразится второе поле и так далее. Таким образом можно ввести любое количество значений параметров.

Эти значения доступны скрипту в виде массива `sys.argv`, как показано в ранее добавленном фрагменте кода.

В качестве значения первого параметра введите имя своей группы ресурсов, а в качестве значения второго параметра — имя виртуальной машины, которую необходимо запустить.

![Введение значений параметров](../media/automation-tutorial-runbook-textual-python/runbook-python-params.png)

Нажмите кнопку **ОК**, чтобы запустить модуль Runbook. Будет выполнен модуль Runbook, который запустит указанную виртуальную машину.

## <a name="error-handling-in-python"></a>Обработка ошибок в Python

Вы также можете использовать указанные далее соглашения для получения разных потоков из runbook Python, например WARNING, ERROR и DEBUG.

```python
print("Hello World output")
print("ERROR: - Hello world error")
print("WARNING: - Hello world warning")
print("DEBUG: - Hello world debug")
print("VERBOSE: - Hello world verbose")
```

Приведенный ниже пример демонстрирует применение этой конвенции в блоке `try...except`.

```python
try:
    raise Exception('one', 'two')
except Exception as detail:
    print 'ERROR: Handling run-time error:', detail
```

> [!NOTE]
> Служба автоматизации Azure не поддерживает `sys.stderr`.

## <a name="next-steps"></a>Дальнейшие действия

- Сведения о том, как начать работу с runbook, см. в статье о [создании runbook PowerShell](automation-tutorial-runbook-textual-powershell.md).
- Чтобы начать работу с графическими модулями runbook, см. инструкции в [этой статье](automation-tutorial-runbook-graphical.md).
- Чтобы приступить к работе с runbook рабочего процесса PowerShell, обратитесь к [этом руководству](automation-tutorial-runbook-textual.md).
- Дополнительные сведения о типах модулей runbook в службе автоматизации Azure, их преимуществах и ограничениях можно найти в [этой статье](../automation-runbook-types.md).
- Дополнительные сведения о разработке для Azure с использованием Python см. в статье [Azure для разработчиков Python](/azure/python/).
- Примеры модулей Runbook Python 2 см. на странице [Azure для разработчиков Python](https://github.com/azureautomation/runbooks/tree/master/Utility/Python).
