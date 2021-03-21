---
title: Просмотр данных и модели в Windows
titleSuffix: Azure Data Science Virtual Machine
description: Выполнение задач по исследованию и моделированию данных на виртуальной машине Windows для обработки и анализа данных.
services: machine-learning
ms.service: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 05/08/2020
ms.openlocfilehash: c44c40f2ddab53cf0beb5ecd48127d126fdb4333
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "101657406"
---
# <a name="data-science-with-a-windows-data-science-virtual-machine"></a>Обработка и анализ данных с помощью виртуальной машины Windows для обработки и анализа данных

Виртуальная машина Windows для обработки и анализа данных (DSVM) — это мощная среда разработки для обработки и анализа данных, в которой можно выполнять задачи по исследованию и моделированию данных. Среда уже создана и объединена с несколькими популярными средствами анализа данных, которые упрощают начало работы с анализом для локальных, облачных и гибридных развертываний. 

DSVM тесно работает со службами Azure. Он может считывать и обрабатывать данные, которые уже хранятся в Azure, в Azure синапсе (ранее — SQL DW), Azure Data Lake, службе хранилища Azure или Azure Cosmos DB. Он также может использовать другие средства аналитики, такие как Машинное обучение Azure.

В этой статье вы узнаете, как использовать DSVM для выполнения задач обработки и анализа данных и взаимодействия с другими службами Azure. Вот некоторые задачи, которые можно выполнить в DSVM:

- Используйте Jupyter Notebook для экспериментирования с данными в браузере с помощью Python 2, Python 3 и Microsoft R. (Microsoft R — это готовая к использованию Корпоративная версия R, предназначенная для повышения производительности.)
- Изучите данные и разрабатывайте модели локально в DSVM с помощью Microsoft Machine Learning Server и Python.
- Администрирование ресурсов Azure с помощью портал Azure или PowerShell.
- Расширьте пространство в хранилище и делитесь большими наборами данных и кодом во всей группе, создав файловый ресурс Azure в качестве подключаемого диска в DSVM.
- Поделитесь кодом с командой с помощью GitHub. Доступ к репозиторию с помощью предварительно установленных клиентов Git: Git Bash и GUI Git.
- Доступ к службам данных и аналитикам Azure, таким как хранилище BLOB-объектов Azure, Azure Cosmos DB, Azure синапсе (ранее SQL DW) и база данных SQL Azure.
- Создание отчетов и панели мониторинга с помощью экземпляра Power BI Desktop, предварительно установленного на DSVM, и развертывание их в облаке.

- Установите на виртуальной машине дополнительные средства.   

> [!NOTE]
> Дополнительные платы за использование относятся ко многим службам хранения данных и аналитики, перечисленным в этой статье. Дополнительные сведения см. на странице [цен на Azure](https://azure.microsoft.com/pricing/) .
> 
> 

## <a name="prerequisites"></a>Предварительные условия

* Вам понадобится подписка Azure. Вы можете [воспользоваться бесплатной пробной версией](https://azure.microsoft.com/free/).
* Инструкции по подготовке виртуальной машины для обработки и анализа данных на портал Azure доступны при [создании виртуальной машины](https://portal.azure.com/#create/microsoft-dsvm.dsvm-windowsserver-2016).


[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]


## <a name="use-jupyter-notebooks"></a>Использование записных книжек Jupyter
Jupyter Notebook предоставляет интегрированную среду разработки для просмотра и моделирования данных на основе браузера. В Jupyter Notebook можно использовать Python 2, Python 3 или R (с открытым исходным кодом и Microsoft R Server).

Чтобы запустить Jupyter Notebook, выберите значок **Jupyter Notebook** в меню " **Пуск** " или на рабочем столе. В командной строке DSVM можно также выполнить команду ```jupyter notebook``` из каталога, в котором находятся существующие записные книжки или где нужно создать новые записные книжки.  

После запуска Jupyter перейдите в каталог, например `/notebooks` записные книжки, которые предварительно упакованы в DSVM. Теперь вы можете:

* Выберите записную книжку, чтобы просмотреть код.
* Выполните каждую ячейку, нажав клавиши SHIFT + ВВОД.
* Запустите всю записную книжку , выбрав пункт  >  **запустить** ячейку.
* Создайте новую записную книжку, щелкнув значок Jupyter (в левом верхнем углу), нажав кнопку **создать** справа, а затем выбрав язык записной книжки (также известный как ядра).   

> [!NOTE]
> В настоящее время поддерживаются ядра Python 2,7, Python 3,6, R, Julia и PySpark в Jupyter. Ядро R поддерживает программирование на языке R и Microsoft R с открытым исходным кодом.   
> 
> 

Когда Вы находитесь в записной книжке, вы можете исследовать данные, построить модель и протестировать модель с помощью выбора библиотек.

## <a name="explore-data-and-develop-models-with-microsoft-machine-learning-server"></a>Изучите данные и разрабатывайте модели с помощью Microsoft Machine Learning Server
Используя такие языки, как R и Python, анализ данных можно выполнять непосредственно в DSVM.

Для R можно использовать интегрированную среду разработки, например RStudio, которую можно найти в меню "Пуск" или на рабочем столе. Или можно использовать Инструменты R для Visual Studio. Корпорация Майкрософт предоставила дополнительные библиотеки на основе CRAN R с открытым исходным кодом, чтобы обеспечить масштабируемую аналитику и возможность анализировать данные, превышающие размер памяти, допустимый при параллельном анализе фрагментированных данных. 

Для Python можно использовать такую интегрированную среду разработки, как Visual Studio Community Edition, в которой предустановлено расширение Python Tools for Visual Studio (PTVS). По умолчанию в PTVS настраивается только Python 3,6, Корневая среда Conda. Чтобы включить Anaconda Python 2,7, выполните следующие действия.

1. Создайте пользовательские среды для каждой версии, выбрав **инструменты**  >  **Python инструменты**  >  **Python окружения**, а затем выбрав **+ Custom** в Visual Studio Community Edition.
1. Введите описание и задайте путь к префиксу окружения в виде **c:\anaconda\envs\python2** для anaconda Python 2,7.
1. Выберите **Автоматическое обнаружение**  >  **Применить** , чтобы сохранить окружение.

Дополнительные сведения о создании окружений Python см. в [документации по PTVS](/visualstudio/python/) .

Теперь вы настроили создание нового проекта Python. Последовательно выберите **файл**  >  **создать**  >  **проект**  >  **Python** и тип создаваемого приложения Python. Можно задать среду Python для текущего проекта до нужной версии (Python 2,7 или 3,6), щелкнув правой кнопкой мыши **среду Python** и выбрав пункт **Добавить или удалить окружения Python**. Дополнительные сведения о работе с PTVS можно найти в документации по [продукту](/visualstudio/python/).



## <a name="manage-azure-resources"></a>Управление ресурсами Azure
DSVM не только позволяет создавать аналитические решения локально на виртуальной машине. Она также позволяет получить доступ к службам на облачной платформе Azure. Azure предоставляет несколько вычислений, хранения, аналитики данных и других служб, которые можно администрировать и получать к ним доступ из DSVM.

Для администрирования подписки Azure и облачных ресурсов можно использовать два варианта:
+ Используйте браузер и перейдите к [портал Azure](https://portal.azure.com).

+ Используйте скрипты PowerShell. Запустите Azure PowerShell с помощью ярлыка на рабочем столе или из меню " **Пуск** ". Полные сведения см. в [документации по Microsoft Azure PowerShell](../../azure-resource-manager/management/manage-resources-powershell.md) . 

## <a name="extend-storage-by-using-shared-file-systems"></a>Расширение хранилища с помощью общих файловых систем
Специалисты по обработке и анализу данных могут использовать большие наборы данных, код или другие ресурсы вместе с участниками группы. DSVM имеет около 45 ГБ свободного места. Чтобы расширить хранилище, можно использовать файлы Azure и либо подключить его к одному или нескольким экземплярам DSVM, либо получить доступ к нему через REST API. Можно также использовать [портал Azure](../../virtual-machines/windows/attach-managed-disk-portal.md) или использовать [Azure PowerShell](../../virtual-machines/windows/attach-disk-ps.md) для добавления дополнительных выделенных дисков данных. 

> [!NOTE]
> Максимальный объем пространства в общей папке службы файлов Azure составляет 5 ТБ. Максимальный размер каждого файла составляет 1 ТБ. 

Этот скрипт можно использовать в Azure PowerShell для создания общей папки службы файлов Azure:

```powershell
# Authenticate to Azure.
Connect-AzAccount
# Select your subscription
Get-AzSubscription –SubscriptionName "<your subscription name>" | Select-AzSubscription
# Create a new resource group.
New-AzResourceGroup -Name <dsvmdatarg>
# Create a new storage account. You can reuse existing storage account if you want.
New-AzStorageAccount -Name <mydatadisk> -ResourceGroupName <dsvmdatarg> -Location "<Azure Data Center Name For eg. South Central US>" -Type "Standard_LRS"
# Set your current working storage account
Set-AzCurrentStorageAccount –ResourceGroupName "<dsvmdatarg>" –StorageAccountName <mydatadisk>

# Create an Azure Files share
$s = New-AzStorageShare <<teamsharename>>
# Create a directory under the file share. You can give it any name
New-AzStorageDirectory -Share $s -Path <directory name>
# List the share to confirm that everything worked
Get-AzStorageFile -Share $s
```

Теперь, когда вы создали общую папку службы файлов Azure, ее можно подключить к любой виртуальной машине в Azure. Рекомендуется разместить виртуальную машину в том же центре обработки данных Azure, что и учетная запись хранения, чтобы избежать расходов на задержку и передачу данных. Ниже приведены Azure PowerShell команды для подключения диска к DSVM:

```powershell
# Get the storage key of the storage account that has the Azure Files share from the Azure portal. Store it securely on the VM to avoid being prompted in the next command.
cmdkey /add:<<mydatadisk>>.file.core.windows.net /user:<<mydatadisk>> /pass:<storage key>

# Mount the Azure Files share as drive Z on the VM. You can choose another drive letter if you want.
net use z:  \\<mydatadisk>.file.core.windows.net\<<teamsharename>>
```

Теперь к этому диску можно обращаться как к обычному диску на виртуальной машине.

## <a name="share-code-in-github"></a>Совместное использование кода в GitHub
GitHub — это репозиторий кода, в котором можно найти примеры кода и источники для различных средств с помощью технологий, совместно используемых сообществом разработчиков. Git выступает в качестве технологии для отслеживания и хранения версий файлов кода. GitHub также является платформой, где можно создать собственный репозиторий для хранения общего кода и документации команды, реализовать управление версиями и контролировать доступ к коду для просмотра и Contribute. 

Дополнительные сведения об использовании Git см. на [страницах справки GitHub](https://help.github.com/). GitHub можно использовать как один из способов совместной работы с командой, использования кода, разработанного сообществом, и обратного участия в сообществе.

DSVM загружается с помощью клиентских средств из командной строки и графического пользовательского интерфейса для доступа к репозиторию GitHub. Программа командной строки, которая работает с Git и GitHub, называется git bash. Visual Studio устанавливается на DSVM и имеет расширения Git. Значки для этих инструментов можно найти в меню **Пуск** и на рабочем столе.

Для скачивания кода из репозитория GitHub используйте команду ```git clone```. Например, чтобы скачать репозиторий обработки и анализа данных, опубликованный корпорацией Майкрософт в текущем каталоге, можно выполнить следующую команду в Git Bash:

```bash
git clone https://github.com/Azure/DataScienceVM.git
```

В Visual Studio можно выполнить ту же операцию клонирования. На следующем снимке экрана показано, как получить доступ к средствам Git и GitHub в Visual Studio:

![Снимок экрана Visual Studio с отображением соединения с GitHub](./media/vm-do-ten-things/VSGit.png)

Дополнительные сведения об использовании Git для работы с репозиторием GitHub можно найти в ресурсах, доступных в github.com. Вы также найдете полезную информацию в этой [памятке](https://services.github.com/on-demand/downloads/github-git-cheat-sheet.pdf) .

## <a name="access-azure-data-and-analytics-services"></a>Доступ к данным и службам аналитики Azure
### <a name="azure-blob-storage"></a>Хранилище BLOB-объектов Azure
Хранилище BLOB-объектов Azure — это надежная и экономичная служба облачного хранилища для больших и малых данных. В этом разделе описывается, как можно переместить данные в хранилище BLOB-объектов и получить доступ к данным, хранящимся в большом двоичном объекте Azure.

#### <a name="prerequisites"></a>Предварительные условия

* Создайте учетную запись хранилища BLOB-объектов Azure из [портал Azure](https://portal.azure.com).

   ![Снимок экрана процесса создания учетной записи хранения в портал Azure](./media/vm-do-ten-things/create-azure-blob.png)

* Убедитесь, что предварительно установлена программа командной строки AzCopy: ```C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy.exe``` . Каталог, содержащий azcopy.exe, уже находится в переменной среды PATH, поэтому при запуске этого средства можно избежать ввода полного пути к команде. Дополнительные сведения о средстве AzCopy см. в [документации по AzCopy](../../storage/common/storage-use-azcopy-v10.md).
* Запустите инструмент Azure Storage Explorer. Его можно загрузить на  [веб-странице Обозреватель службы хранилища](https://storageexplorer.com/). 

   ![Снимок экрана Обозреватель службы хранилища Azure доступ к учетной записи хранения](./media/vm-do-ten-things/AzureStorageExplorer_v4.png)

#### <a name="move-data-from-a-vm-to-an-azure-blob-azcopy"></a>Перемещение данных из виртуальной машины в большой двоичный объект Azure: AzCopy

Чтобы переместить данные между локальными файлами и хранилищем BLOB-объектов, можно использовать AzCopy в командной строке или в PowerShell:

```powershell
AzCopy /Source:C:\myfolder /Dest:https://<mystorageaccount>.blob.core.windows.net/<mycontainer> /DestKey:<storage account key> /Pattern:abc.txt
```

Замените **C:\MyFolder** на путь, по которому хранится файл, **mystorageaccount** с именем учетной записи хранилища BLOB-объектов, **MyContainer** с именем контейнера и **ключом учетной записи хранения** с ключом доступа к хранилищу BLOB-объектов. Учетные данные учетной записи хранения можно найти в [портал Azure](https://portal.azure.com).

Выполните команду AzCopy в PowerShell или из командной строки. Ниже приведен пример использования команды AzCopy:

```powershell
# Copy *.sql from a local machine to an Azure blob
"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Source:"c:\Aaqs\Data Science Scripts" /Dest:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /DestKey:[ENTER STORAGE KEY] /S /Pattern:*.sql

# Copy back all files from an Azure blob container to a local machine

"C:\Program Files (x86)\Microsoft SDKs\Azure\AzCopy\azcopy" /Dest:"c:\Aaqs\Data Science Scripts\temp" /Source:https://[ENTER STORAGE ACCOUNT].blob.core.windows.net/[ENTER CONTAINER] /SourceKey:[ENTER STORAGE KEY] /S
```

После выполнения команды AzCopy для копирования в большой двоичный объект Azure файл появится в Обозреватель службы хранилища Azure.

![Снимок экрана учетной записи хранения с отправленным CSV-файлом](./media/vm-do-ten-things/AzCopy_run_finshed_Storage_Explorer_v3.png)

#### <a name="move-data-from-a-vm-to-an-azure-blob-azure-storage-explorer"></a>Перемещение данных из виртуальной машины в большой двоичный объект Azure: Обозреватель службы хранилища Azure

Вы также можете передать данные из локального файла на виртуальной машине с помощью Обозреватель службы хранилища Azure:

* Чтобы отправить данные в контейнер, выберите целевой контейнер и нажмите кнопку Upload ( **Отправить** ). ![ Снимок экрана кнопки "Отправить" в Обозреватель службы хранилища Azure](./media/vm-do-ten-things/storage-accounts.png)
* Нажмите кнопку с многоточием (**...**) справа от поля **файлы** , выберите один или несколько файлов для отправки из файловой системы и нажмите кнопку **Отправить** , чтобы начать отправку файлов. ![ Снимок экрана: диалоговое окно "Отправка файлов"](./media/vm-do-ten-things/upload-files-to-blob.png)

#### <a name="read-data-from-an-azure-blob-python-odbc"></a>Чтение данных из большого двоичного объекта Azure: ODBC Python

Библиотеку BlobService можно использовать для чтения данных непосредственно из большого двоичного объекта в Jupyter Notebook или в программе Python.

Сначала импортируйте необходимые пакеты:

```python
import pandas as pd
from pandas import Series, DataFrame
import numpy as np
import matplotlib.pyplot as plt
from time import time
import pyodbc
import os
from azure.storage.blob import BlobService
import tables
import time
import zipfile
import random
```

Затем подсоедините учетные данные учетной записи хранилища BLOB-объектов и прочтите данные из большого двоичного объекта:

```python
CONTAINERNAME = 'xxx'
STORAGEACCOUNTNAME = 'xxxx'
STORAGEACCOUNTKEY = 'xxxxxxxxxxxxxxxx'
BLOBNAME = 'nyctaxidataset/nyctaxitrip/trip_data_1.csv'
localfilename = 'trip_data_1.csv'
LOCALDIRECTORY = os.getcwd()
LOCALFILE =  os.path.join(LOCALDIRECTORY, localfilename)

#download from blob
t1 = time.time()
blob_service = BlobService(account_name=STORAGEACCOUNTNAME,account_key=STORAGEACCOUNTKEY)
blob_service.get_blob_to_path(CONTAINERNAME,BLOBNAME,LOCALFILE)
t2 = time.time()
print(("It takes %s seconds to download "+BLOBNAME) % (t2 - t1))

#unzip downloaded files if needed
#with zipfile.ZipFile(ZIPPEDLOCALFILE, "r") as z:
#    z.extractall(LOCALDIRECTORY)

df1 = pd.read_csv(LOCALFILE, header=0)
df1.columns = ['medallion','hack_license','vendor_id','rate_code','store_and_fwd_flag','pickup_datetime','dropoff_datetime','passenger_count','trip_time_in_secs','trip_distance','pickup_longitude','pickup_latitude','dropoff_longitude','dropoff_latitude']
print 'the size of the data is: %d rows and  %d columns' % df1.shape
```

Данные считываются как кадр данных:

![Снимок экрана первых 10 строк данных](./media/vm-do-ten-things/IPNB_data_readin.png)


### <a name="azure-synapse-analytics-and-databases"></a>Azure синапсе Analytics и базы данных
Azure синапсе Analytics — это хранилище эластичных данных как услуга с SQL Server корпоративного класса.

Вы можете подготавливать Azure синапсе Analytics, выполнив инструкции, приведенные в [этой статье](../../synapse-analytics/sql-data-warehouse/create-data-warehouse-portal.md). После того как вы подготавливаете Azure синапсе Analytics, вы можете использовать [это пошаговое руководство](../team-data-science-process/sqldw-walkthrough.md) для отправки, просмотра и моделирования данных с помощью данных в Azure синапсе Analytics.

#### <a name="azure-cosmos-db"></a>Azure Cosmos DB
Azure Cosmos DB — это база данных NoSQL в облаке. Его можно использовать для работы с документами, такими как JSON, а также для хранения и запроса документов.

Используйте следующие шаги, чтобы получить доступ к Azure Cosmos DB из DSVM:

1. Пакет SDK для Azure Cosmos DB Python уже установлен на DSVM. Чтобы обновить его, выполните ```pip install pydocumentdb --upgrade``` команду из командной строки.
2. Создайте учетную запись Azure Cosmos DB и базу данных из [портал Azure](https://portal.azure.com).
3. Скачайте средство переноса данных Azure Cosmos DB из [центра загрузки Майкрософт](https://www.microsoft.com/download/details.aspx?id=53595) и извлеките его в нужный каталог.
4. Импортируйте данные JSON (о вулканах Data), хранящиеся в [общедоступном большом двоичном объекте](https://data.humdata.org/dataset/a60ac839-920d-435a-bf7d-25855602699d/resource/7234d067-2d74-449a-9c61-22ae6d98d928/download/volcano.json) , в Azure Cosmos DB со следующими параметрами команды для средства миграции. (Используйте dtui.exe из каталога, в котором установлен инструмент переноса данных Azure Cosmos DB.) Введите исходное и целевое расположение с этими параметрами:
   
    `/s:JsonFile /s.Files:https://data.humdata.org/dataset/a60ac839-920d-435a-bf7d-25855602699d/resource/7234d067-2d74-449a-9c61-22ae6d98d928/download/volcano.json /t:DocumentDBBulk /t.ConnectionString:AccountEndpoint=https://[DocDBAccountName].documents.azure.com:443/;AccountKey=[[KEY];Database=volcano /t.Collection:volcano1`

После импорта данных можно перейти в Jupyter и открыть записную книжку под названием *DocumentDBSample*. Он содержит код Python для доступа к Azure Cosmos DB и выполнения некоторых базовых запросов. Дополнительные сведения о Azure Cosmos DB можно получить на [странице документации](../../cosmos-db/index.yml)по службе.

## <a name="use-power-bi-reports-and-dashboards"></a>Использование Power BI отчетов и панелей мониторинга 
Вы можете визуализировать файл JSON О вулканах из предыдущего Azure Cosmos DB примера в Power BI Desktop, чтобы получить наглядную информацию о данных. Подробные указания см. в статье о [Power BI](../../cosmos-db/powerbi-visualize.md). Ниже приведены основные действия.

1. Откройте Power BI Desktop и выберите **Получение данных**. Укажите URL-адрес как: `https://cahandson.blob.core.windows.net/samples/volcano.json` .
2. Вы должны увидеть записи JSON, импортированные в виде списка. Преобразуйте список в таблицу, чтобы Power BI может работать с ним.
4. Разверните столбцы, щелкнув значок развернуть (стрелка).
5. Обратите внимание, что расположение является полем **записи** . Разверните запись и выберите только координаты. **Координата** — это столбец списка.
6. Добавьте новый столбец, чтобы преобразовать столбец координат списка в **LatLong** столбец с разделителями-запятыми. Объедините два элемента в поле списка координат с помощью формулы ```Text.From([coordinates]{1})&","&Text.From([coordinates]{0})``` .
7. Преобразуйте столбец **повышения** в тип Decimal и выберите кнопки **Закрыть** и **Применить** .

Вместо предыдущих шагов можно вставить следующий код. Он включает в себя действия, используемые в Расширенный редактор в Power BI для записи преобразований данных на языке запросов.

```pqfl
let
    Source = Json.Document(Web.Contents("https://cahandson.blob.core.windows.net/samples/volcano.json")),
    #"Converted to Table" = Table.FromList(Source, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    #"Expanded Column1" = Table.ExpandRecordColumn(#"Converted to Table", "Column1", {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}, {"Volcano Name", "Country", "Region", "Location", "Elevation", "Type", "Status", "Last Known Eruption", "id"}),
    #"Expanded Location" = Table.ExpandRecordColumn(#"Expanded Column1", "Location", {"coordinates"}, {"coordinates"}),
    #"Added Custom" = Table.AddColumn(#"Expanded Location", "LatLong", each Text.From([coordinates]{1})&","&Text.From([coordinates]{0})),
    #"Changed Type" = Table.TransformColumnTypes(#"Added Custom",{{"Elevation", type number}})
in
    #"Changed Type"
```

Теперь ваши данные находятся в модели данных Power BI. Экземпляр Power BI Desktop должен выглядеть следующим образом:

![Power BI Desktop](./media/vm-do-ten-things/PowerBIVolcanoData.png)

Вы можете приступить к созданию отчетов и визуализаций с помощью модели данных. Для создания отчета можно выполнить действия, описанные в [этой Power BI статье](../../cosmos-db/powerbi-visualize.md#build-the-reports) .

## <a name="scale-the-dsvm-dynamically"></a>Динамическое масштабирование DSVM 
Вы можете масштабировать DSVM в соответствии с потребностями проекта. Если вы не хотите использовать виртуальную машину в вечером или в выходные дни, можно завершить работу виртуальной машины из [портал Azure](https://portal.azure.com).

> [!NOTE]
> Если для операционной системы на виртуальной машине используется только кнопка завершения работы, взимается плата за использование вычислений. Вместо этого следует освободить DSVM с помощью портал Azure или Cloud Shell.
> 
> 

Может потребоваться выполнить какой-либо крупномасштабный анализ и потребовать больше ресурсов ЦП, памяти или дискового пространства. Если это так, можно выбрать размеры виртуальных машин в терминах ядер ЦП, экземпляров на основе GPU для глубокого обучения, объема памяти и типов дисков (включая твердотельные накопители), которые соответствуют потребностям вычислений и бюджета. Полный список виртуальных машин, а также их почасовые расчетные цены доступны на странице [цен на виртуальные машины Azure](https://azure.microsoft.com/pricing/details/virtual-machines/) .


## <a name="add-more-tools"></a>Добавить дополнительные инструменты
Средства, встроенные в DSVM, могут решать многие распространенные потребности в аналитике данных. Это экономит время, так как вам не нужно устанавливать и настраивать среды по одному. Это также экономит деньги, так как вы платите только за используемые ресурсы.

Вы можете использовать другие данные и службы аналитики Azure, профилированные в этой статье, для улучшения среды аналитики. В некоторых случаях могут потребоваться дополнительные средства, в том числе некоторые специализированные средства для партнеров. У вас есть полный административный доступ к виртуальной машине для установки новых необходимых средств. Вы также можете установить дополнительные пакеты на Python и R, которые не предустановлены. Для Python можно использовать либо ```conda``` или ```pip``` . Для r можно использовать ```install.packages()``` в консоли R или воспользоваться интегрированной средой разработки и выбрать **пакеты**  >  **установить пакеты**.

## <a name="deep-learning"></a>Глубокое обучение

В дополнение к примерам на основе платформы можно получить набор подробных руководств, которые были проверены на DSVM. Эти пошаговые руководства помогут вам быстро приступить к разработке приложений для глубокого обучения в таких доменах, как изображение и текст/понимание языка.   


- [Выполнение нейронных сетей в разных платформах](https://github.com/ilkarman/DeepLearningFrameworks). в этом пошаговом руководстве показано, как перенести код из одной платформы в другую. В нем также показано, как сравнивать модели и производительность среды выполнения между различными платформами. 

- [Руководство по созданию комплексного решения по обнаружению продуктов на изображениях.](https://github.com/Azure/cortana-intelligence-product-detection-from-images) Обнаружение изображений — это метод, с помощью которого можно находить и классифицировать объекты на изображении. Эта технология может привести к огромным преимуществам во многих реальных бизнес-доменах. Например, розничные продавцы могут использовать этот метод для определения того, какой продукт клиент взял с полки. Эти сведения, в свою очередь, помогают управлять запасами продукции. 

- [Глубокое обучение для аудио](/archive/blogs/machinelearning/hearing-ai-getting-started-with-deep-learning-for-audio-on-azure): в этом руководстве показано, как обучить модель глубокого обучения для обнаружения событий аудио в [наборе данных городской звук](https://serv.cusp.nyu.edu/projects/urbansounddataset/urbansound8k.html). В нем также приводятся общие сведения о работе с звуковыми данными.

- [Классификация текстовых документов](https://github.com/anargyri/lstm_han). в этом пошаговом руководстве показано, как создать и обучить две архитектуры нейронных сетей: сетевое внимание и сеть с длительной краткосрочной памятью (LSTM). Эти нейронные сети выполняют классификацию текстовых документов с помощью API Keras для глубокого обучения. 

## <a name="summary"></a>Итоги
В этой статье описаны некоторые действия, которые можно выполнить на виртуальной машине Майкрософт для обработки и анализа данных. Существует множество вещей, которые можно сделать, чтобы сделать DSVM эффективной средой аналитики.
