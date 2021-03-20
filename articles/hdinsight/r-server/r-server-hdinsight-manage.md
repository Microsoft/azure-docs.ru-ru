---
title: Управление кластером Служб машинного обучения в HDInsight в Azure
description: Узнайте, как управлять различными задачами в кластере служб ML в Azure HDInsight.
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 06/19/2019
ms.openlocfilehash: e4c9124ebd0b61b8db1b1da964355a3c36b5bba5
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98930555"
---
# <a name="manage-ml-services-cluster-on-azure-hdinsight"></a>Управление кластером служб машинного обучения в Azure HDInsight

Из этой статьи вы узнаете, как управлять существующим кластером служб ML в Azure HDInsight для выполнения таких задач, как добавление нескольких параллельных пользователей, удаленное подключение к кластеру служб машинного обучения, изменение контекста вычислений и т. д.

## <a name="prerequisites"></a>Предварительные условия

* Кластер служб машинного обучения в HDInsight. Ознакомьтесь со статьей [Create Linux-based clusters in HDInsight by using the Azure portal](../hdinsight-hadoop-create-linux-clusters-portal.md) (Создание кластеров под управлением Linux в HDInsight с помощью портала Azure) и выберите **Службы машинного обучения ML Services** для параметра **Тип кластера**.

* Клиент Secure Shell (SSH). Клиент SSH используется для удаленного подключения к кластеру HDInsight и выполнения команд непосредственно в кластере. Дополнительные сведения см. в статье [Использование SSH с HDInsight.](../hdinsight-hadoop-linux-use-ssh-unix.md)

## <a name="enable-multiple-concurrent-users"></a>Включение нескольких параллельных подключений пользователей

Вы можете включить несколько одновременно работающих пользователей в кластер служб машинного обучения в HDInsight, добавив дополнительных пользователей в граничном узле, на котором выполняется версия RStudio Community. При создании кластера HDInsight необходимо указать двух пользователей: для HTTP и SSH.

![HDI портал Azure параметры входа](./media/r-server-hdinsight-manage/hdi-concurrent-users1.png)

- **Имя пользователя для входа в кластер** — пользователь HTTP для проверки подлинности с помощью шлюза HDInsight, который применяется для защиты созданных вами кластеров HDInsight. Эти учетные данные пользователя HTTP нужны для доступа к пользовательскому интерфейсу Apache Ambari, пользовательскому интерфейсу Apache Hadoop YARN, а также другим компонентам пользовательского интерфейса.
- **Secure Shell (SSH) username**— пользователь SSH для доступа к кластеру через Secure Shell. Этот пользователь применяется в системе Linux для всех головных, рабочих и граничных узлов. Таким образом, с помощью безопасной оболочки вы можете получить доступ к любому узлу в удаленном кластере.

Версия RStudio Server Community, которая используется в кластере служб машинного обучения в HDInsight, поддерживает только один метод входа: имя пользователя и пароль Linux. Эта версия не поддерживает передачу токенов. Таким образом, при первом входе в RStudio в кластере служб машинного обучения потребуется ввести учетные данные два раза.

- Сначала используйте учетные данные пользователя HTTP через шлюз HDInsight.

- Затем введите учетные данные SSH для входа в RStudio.
  
Сейчас при подготовке кластера HDInsight можно создать только одну учетную запись пользователя SSH. Поэтому, чтобы предоставить нескольким пользователям доступ к кластеру служб машинного обучения в HDInsight, необходимо создать дополнительных пользователей в системе Linux.

Так как RStudio работает на граничном узле кластера, нужно выполнить несколько дополнительных шагов:

1. войти на граничный узел с учетными данными существующего пользователя SSH
2. добавить нескольких пользователей Linux на граничный узел;
3. использовать версию RStudio Community с помощью созданного пользователя.

### <a name="step-1-use-the-created-ssh-user-to-sign-in-to-the-edge-node"></a>Шаг 1. Вход на граничный узел с помощью созданного пользователя SSH

Чтобы получить доступ к граничному узлу, следуйте инструкциям по [подключению к HDInsight (Apache Hadoop) с помощью SSH](../hdinsight-hadoop-linux-use-ssh-unix.md). Адрес граничного узла кластера служб машинного обучения в HDInsight: `CLUSTERNAME-ed-ssh.azurehdinsight.net`.

### <a name="step-2-add-more-linux-users-in-edge-node"></a>Шаг 2. Добавление дополнительных пользователей Linux на граничный узел

Чтобы добавить пользователя на граничный узел, выполните эти команды:

```bash
# Add a user 
sudo useradd <yournewusername> -m

# Set password for the new user
sudo passwd <yournewusername>
```

Выходные данные показаны на снимке экрана ниже.

![одновременные пользователи выводят на экран снимок экрана](./media/r-server-hdinsight-manage/hdi-concurrent-users2.png)

При появлении запроса на ввод текущего пароля Kerberos нажмите клавишу **ВВОД**, чтобы игнорировать его. Параметр `-m` в команде `useradd` указывает, что система создаст домашнюю папку пользователя, которая обязательна для версии RStudio Community.

### <a name="step-3-use-rstudio-community-version-with-the-user-created"></a>Шаг 3. Использование версии RStudio Community с помощью созданного пользователя

Войдите в RStudio из `https://CLUSTERNAME.azurehdinsight.net/rstudio/`. Если вы впервые входите в систему после создания кластера, сначала введите учетные данные администратора кластера, а затем созданные учетные данные пользователя SSH. Если это уже не первый вход, достаточно ввести учетные данные нового пользователя SSH.

Вы также можете войти параллельно с помощью исходных учетных данных (по умолчанию — *sshuser*) в другом окне браузера.

Обратите внимание, что у добавленных пользователей нет привилегированных прав в системе Linux, но у них есть тот же доступ ко всем файлам в удаленном хранилище HDFS и WASB.

## <a name="connect-remotely-to-microsoft-ml-services"></a>Удаленное подключение к серверу или клиенту служб машинного обучения Microsoft

Вы можете получить доступ к контексту вычислений Spark в HDInsight из удаленного экземпляра ML Client, запущенного на персональном компьютере. Для этого укажите на ноутбуке параметры dfsShareDir, shareDir, sshUsername, sshHostname, sshSwitches и sshProfileScript, определяя контекст вычислений RxSpark.

```r
myNameNode <- "default"
myPort <- 0

mySshHostname  <- '<clustername>-ed-ssh.azurehdinsight.net'  # HDI secure shell hostname
mySshUsername  <- '<sshuser>'# HDI SSH username
mySshSwitches  <- '-i /cygdrive/c/Data/R/davec'   # HDI SSH private key

myhdfsShareDir <- paste("/user/RevoShare", mySshUsername, sep="/")
myShareDir <- paste("/var/RevoShare" , mySshUsername, sep="/")

mySparkCluster <- RxSpark(
    hdfsShareDir = myhdfsShareDir,
    shareDir     = myShareDir,
    sshUsername  = mySshUsername,
    sshHostname  = mySshHostname,
    sshSwitches  = mySshSwitches,
    sshProfileScript = '/etc/profile',
    nameNode     = myNameNode,
    port         = myPort,
    consoleOutput= TRUE
)
```

Дополнительные сведения см. в разделе об использовании Microsoft Machine Learning Server в качестве клиента Apache Hadoop в статье [How to use RevoScaleR in a Spark compute context](/machine-learning-server/r/how-to-revoscaler-spark#more-spark-scenarios) (Как использовать RevoScaleR в контексте вычислений Spark).

## <a name="use-a-compute-context"></a>Использование контекста вычислений

Контекст вычислений позволяет определить, где будут выполнены вычисления — на локальном компьютере или пограничном узле, или же они будут распределены между узлами в кластере HDInsight.  Пример настройки контекста вычислений с RStudio Server см. [в статье выполнение скрипта R в кластере служб ML в Azure HDInsight с помощью сервера RStudio](machine-learning-services-quickstart-job-rstudio.md).

## <a name="distribute-r-code-to-multiple-nodes"></a>Распространение кода R на несколько узлов

Используя службы машинного обучения в HDInsight, можно выполнять имеющийся код R на нескольких узлах кластера с помощью `rxExec`. Эта функция удобна при очистке параметров или моделировании. Следующий код является примером использования `rxExec`.

```r
rxExec( function() {Sys.info()["nodename"]}, timesToRun = 4 )
```

Если вы все еще используете контекст Spark, эта команда вернет значение nodename рабочих узлов, на которых запущен код (`(Sys.info()["nodename"])`). Например, в кластере с четырьмя узлами будет получен примерно следующий результат фрагмента кода:

```r
$rxElem1
    nodename
"wn3-mymlser"

$rxElem2
    nodename
"wn0-mymlser"

$rxElem3
    nodename
"wn3-mymlser"

$rxElem4
    nodename
"wn3-mymlser"
```

## <a name="access-data-in-apache-hive-and-parquet"></a>Доступ к данным в Apache Hive и Parquet

Службы машинного обучения HDInsight обеспечивают прямой доступ к данным в Hive и Parquet для использования функций ScaleR в контексте вычислений Spark. Эти возможности, доступные в рамках новых функций источника данных ScaleR, называются RxHiveData и RxParquetData. Они используются со Spark SQL для загрузки данных непосредственно в Spark DataFrame и последующего анализа с помощью ScaleR.

Ниже представлен пример кода с использованием новых функций:

```r
#Create a Spark compute context:
myHadoopCluster <- rxSparkConnect(reset = TRUE)

#Retrieve some sample data from Hive and run a model:
hiveData <- RxHiveData("select * from hivesampletable",
                       colInfo = list(devicemake = list(type = "factor")))
rxGetInfo(hiveData, getVarInfo = TRUE)

rxLinMod(querydwelltime ~ devicemake, data=hiveData)

#Retrieve some sample data from Parquet and run a model:
rxHadoopMakeDir('/share')
rxHadoopCopyFromLocal(file.path(rxGetOption('sampleDataDir'), 'claimsParquet/'), '/share/')
pqData <- RxParquetData('/share/claimsParquet',
                        colInfo = list(
                            age    = list(type = "factor"),
                            car.age = list(type = "factor"),
                            type = list(type = "factor")
                        ) )
rxGetInfo(pqData, getVarInfo = TRUE)

rxNaiveBayes(type ~ age + cost, data = pqData)

#Check on Spark data objects, cleanup, and close the Spark session:
lsObj <- rxSparkListData() # two data objs are cached
lsObj
rxSparkRemoveData(lsObj)
rxSparkListData() # it should show empty list
rxSparkDisconnect(myHadoopCluster)
```

Дополнительные сведения об использовании этих новых функций см. в справочной документации по службам машинного обучения с помощью команд `?RxHivedata` и `?RxParquetData`.  

## <a name="install-additional-r-packages-on-the-cluster"></a>Установка дополнительных пакетов R в кластер

### <a name="to-install-r-packages-on-the-edge-node"></a>Установка пакетов R на граничный узел

Если вы хотите установить дополнительные пакеты R на граничный узел, подключитесь к граничному узлу по протоколу SSH и вызовите `install.packages()` непосредственно из консоли R. 

### <a name="to-install-r-packages-on-the-worker-node"></a>Установка пакетов R на рабочий узел

Чтобы установить пакеты R на рабочие узлы кластера, необходимо использовать действие скрипта. Действия скриптов — это скрипты Bash, которые используются для изменения конфигурации кластера HDInsight или установки дополнительного программного обеспечения, например дополнительных пакетов R. 

> [!IMPORTANT]  
> Использовать действия сценария для установки дополнительных пакетов R можно только после создания кластера. Не используйте эту процедуру во время создания кластера, так как скрипту требуется полностью настроенные службы машинного обучения.

1. Выполните действия по [настройке кластеров с помощью действия скрипта](../hdinsight-hadoop-customize-cluster-linux.md).

3. В разделе **Отправка действия скрипта** введите следующие сведения.

   * В поле **Тип скрипта** выберите **Пользовательский**.

   * В поле **Имя** укажите имя для действия скрипта.

     * В поле **URI bash-скрипта** введите `https://mrsactionscripts.blob.core.windows.net/rpackages-v01/InstallRPackages.sh`. Этот скрипт устанавливает дополнительные пакеты R на рабочий узел.

   * Установите флажок только для параметра **Рабочая роль**.

   * **Параметры**: устанавливаемые пакеты R. Например: `bitops stringr arules`

   * Установите флажок **Persist this script action** (Сохранить это действие скрипта).  

   > [!NOTE]
   > 1. По умолчанию все пакеты R устанавливаются из моментального снимка репозитория Microsoft MRAN, согласованного с установленной версией сервера Machine Learning Server. Если вы хотите установить более новые версии пакетов, существует риск несовместимости. Но эту установку можно выполнить, указав `useCRAN` в качестве первого элемента список пакетов, например `useCRAN bitops, stringr, arules`.  
   > 2. Для некоторых пакетов R требуются дополнительные системные библиотеки Linux. Для удобства были предварительно установлены службы машинного обучения HDInsight с помощью зависимостей, необходимых для 100 наиболее популярных пакетов R. Однако если для устанавливаемых пакетов R требуются дополнительные библиотеки, то необходимо скачать основной скрипт, используемый в этой статье, и добавить шаги для установки системных библиотек. Затем необходимо передать измененный сценарий в общедоступный контейнер больших двоичных объектов в службе хранилище Azure и использовать этот измененный сценарий для установки пакетов.
   >    Дополнительные сведения о разработке действий скриптов см. в разделе [Разработка действий скриптов](../hdinsight-hadoop-script-actions-linux.md).

   ![Действие "отправить скрипт" портал Azure](./media/r-server-hdinsight-manage/submit-script-action.png)

4. Нажмите кнопку **Создать**, чтобы выполнить скрипт. После завершения выполнения пакеты R будут доступны на всех рабочих узлах.

## <a name="next-steps"></a>Дальнейшие действия

* [Ввод в эксплуатацию кластера R Server в Azure HDInsight](r-server-operationalize.md)
* [Варианты контекста вычислений для R Server в HDInsight](r-server-compute-contexts.md)
* [Решения службы хранилища Azure для R Server в Azure HDInsight](r-server-storage.md)