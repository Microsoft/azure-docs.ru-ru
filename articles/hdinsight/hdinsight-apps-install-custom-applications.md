---
title: Установка пользовательских Apache Hadoop приложений в Azure HDInsight
description: Узнайте, как установить приложения HDInsight для кластеров Apache Hadoop в Azure HDInsight.
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 11/29/2019
ms.openlocfilehash: 9591551b7d5657803e770bd71b47acb141b40fb6
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98942697"
---
# <a name="install-custom-apache-hadoop-applications-on-azure-hdinsight"></a>Установка пользовательских приложений Apache Hadoop в Azure HDInsight

В этой статье вы узнаете, как установить приложение [Apache Hadoop](https://hadoop.apache.org/) в Azure HDInsight, которое еще не опубликовано в портал Azure. Приложение, которое вы будете устанавливать в этой статье, является [оттенок](https://gethue.com/).

Пользователи могут устанавливать приложения HDInsight в кластере HDInsight.  Разработчиками этих приложений могут быть корпорация Майкрософт, независимые поставщики программного обеспечения или вы сами.  

## <a name="prerequisites"></a>Предварительные требования

Если вы хотите установить приложение HDInsight в существующем кластере HDInsight, вы должны создать кластер HDInsight. Инструкции по созданию кластера см. в [этом разделе](hadoop/apache-hadoop-linux-tutorial-get-started.md). Вы также можете установить приложения HDInsight во время создания кластера HDInsight.

## <a name="install-hdinsight-applications"></a>Установка приложений HDInsight

Приложения HDInsight можно устанавливать во время создания кластера или в уже существующем кластере HDInsight. Инструкции по определению шаблонов Azure Resource Manager см. в статье [Install an HDInsight application](/rest/api/hdinsight/hdinsight-application) (Установка приложения HDInsight) на сайте MSDN.

Далее перечислены файлы, необходимые для развертывания этого приложения (Hue).

* [azuredeploy.json](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/azuredeploy.json) — шаблон Resource Manager для установки приложения HDInsight. Инструкции по разработке собственного шаблона Resource Manager см. в статье [Install an HDInsight application](/rest/api/hdinsight/hdinsight-application) (Установка приложения HDInsight) на сайте MSDN.
* [hue-install_v0.sh](https://github.com/hdinsight/Iaas-Applications/blob/master/Hue/scripts/Hue-install_v0.sh) — действие скрипта, вызываемое шаблоном Resource Manager для настройки граничного узла.
* [hue-binaries.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz) — двоичный файл hue, вызываемый из hui-install_v0.sh.
* [hue-binaries-14-04.tgz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/hue-binaries-14-04.tgz) — двоичный файл hue, вызываемый из hui-install_v0.sh.
* [webwasb-tomcat.tar.gz](https://hdiconfigactions.blob.core.windows.net/linuxhueconfigactionv01/webwasb-tomcat.tar.gz) — пример веб-приложения (Tomcat), вызываемый из hui-install_v0.sh.

### <a name="to-install-hue-to-an-existing-hdinsight-cluster"></a>Установка приложения Hue в существующем кластере HDInsight

1. Выберите следующее изображение, чтобы войти в Azure и открыть шаблон диспетчер ресурсов в портал Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fhdinsight%2FIaas-Applications%2Fmaster%2FHue%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Шаблон диспетчер ресурсов находится в папке [https://github.com/hdinsight/Iaas-Applications/tree/master/Hue](https://github.com/hdinsight/Iaas-Applications/tree/master/Hue) .  Дополнительные сведения о создании шаблона Resource Manager см. в статье [Install an HDInsight application](/rest/api/hdinsight/hdinsight-application) (Установка приложения HDInsight) на сайте MSDN.

1. Выберите в раскрывающемся списке существующую **группу ресурсов** , содержащую кластер. Необходимо использовать ту же группу ресурсов, что и кластер.

1. Имя кластера, в котором вы хотите установить приложение. Это должен быть существующий кластер.

1. Установите флажок **я принимаю указанные выше условия**.

1. Щелкните **Приобрести**.

Состояние установки можно наблюдать на плитке, закрепленной на панели мониторинга, и в уведомлении портала (щелкните значок колокольчика в верхней части портала).  Установка приложения занимает около 10 минут.

### <a name="to-install-hue-while-creating-a-cluster"></a>Установка приложения Hue во время создания кластера

1. Выберите следующее изображение, чтобы войти в Azure и открыть шаблон диспетчер ресурсов в портал Azure.

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fhditutorialdata.blob.core.windows.net%2Fhdinsightapps%2Fcreate-linux-based-hadoop-cluster-in-hdinsight.json" target="_blank"><img src="./media/hdinsight-apps-install-custom-applications/hdi-deploy-to-azure1.png" alt="Deploy to Azure button for new cluster"></a>

    Шаблон диспетчер ресурсов находится в папке [https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json](https://hditutorialdata.blob.core.windows.net/hdinsightapps/create-linux-based-hadoop-cluster-in-hdinsight.json) .  Дополнительные сведения о создании шаблона Resource Manager см. в статье [Install an HDInsight application](/rest/api/hdinsight/hdinsight-application) (Установка приложения HDInsight) на сайте MSDN.

2. Выполните инструкции по созданию кластера и установите приложение Hue. Дополнительные сведения о создании кластеров HDInsight см. в статье [Создание кластеров Hadoop под управлением Linux в HDInsight](hdinsight-hadoop-provision-linux-clusters.md).

### <a name="other-installation-methods"></a>Другие методы установки

Вызывать шаблоны Resource Manager можно с помощью портала Azure, а также [Azure PowerShell](hdinsight-hadoop-create-linux-clusters-arm-templates.md#deploy-using-powershell) и [интерфейса командной строки Azure](hdinsight-hadoop-create-linux-clusters-arm-templates.md#deploy-using-azure-cli).

## <a name="validate-the-installation"></a>проверка установки

Вы можете просмотреть состояние приложения на портале Azure, чтобы проверить установку приложения. Кроме того, можно также проверить, что все конечные точки HTTP имеют ожидаемый результат, и веб-страницу, если она есть.

Для **оттенок** можно выполнить следующие действия.

### <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите кластер, в котором установлено приложение.
1. В меню **Параметры** выберите **приложения**.
1. Выберите **оттенок** в списке, чтобы просмотреть свойства.  
1. Выберите ссылку на веб-страницу, чтобы проверить сайт.

### <a name="azure-cli"></a>Azure CLI

Замените `CLUSTERNAME` и `RESOURCEGROUP` соответствующими значениями, а затем введите приведенные ниже команды.

* , Чтобы получить список всех приложений для кластера HDInsight.

    ```azurecli
    az hdinsight application list --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
    ```

* Для получения свойств указанного приложения.

    ```azurecli
    az hdinsight application show --name hue --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
    ```

## <a name="troubleshoot-the-installation"></a>Устранение неполадок, связанных с установкой

Состояние установки приложения можно наблюдать в уведомлении портала (щелкните значок колокольчика в верхней части портала).

Если установка приложения завершилась неудачно, можно просмотреть сообщения об ошибках и данные отладки из трех мест:

* Приложения HDInsight: общие сведения об ошибке.

    Откройте кластер на портале и выберите приложения из параметров:

    ![приложения hdinsight приложение ошибка установки](./media/hdinsight-apps-install-custom-applications/hdinsight-apps-error.png)

* Действие сценария HDInsight: если сообщение об ошибке приложений HDInsight указывает на сбой действия сценария, дополнительные сведения о сбое сценария отобразятся на панели действий сценария.

    Выберите действие скрипта в параметрах. Журнал действий сценария отображает сообщения об ошибках.

    ![приложения hdinsight ошибка действия сценария](./media/hdinsight-apps-install-custom-applications/hdinsight-apps-script-action-error.png)

* Веб-интерфейс Apache Ambari. Если сценарий установки был причиной сбоя, используйте пользовательский веб-интерфейс Ambari для проверки полных журналов сценариев установки.

    Дополнительные сведения см. в разделе [Устранение неполадок в сценариях](./troubleshoot-script-action.md).

## <a name="remove-hdinsight-applications"></a>Удаление приложений HDInsight

### <a name="azure-portal"></a>Портал Azure

1. Войдите на [портал Azure](https://portal.azure.com).
1. Выберите кластер, в котором установлено приложение.
1. В меню **Параметры** выберите **приложения**.
1. Щелкните правой кнопкой мыши приложение, которое необходимо удалить, и выберите пункт **Удалить**.
1. Выберите **Да** для подтверждения.

### <a name="azure-cli"></a>Azure CLI

Замените `NAME` , `CLUSTERNAME` и `RESOURCEGROUP` соответствующими значениями, а затем введите следующую команду:

```azurecli
az hdinsight application delete --name NAME --cluster-name CLUSTERNAME --resource-group RESOURCEGROUP
```

## <a name="next-steps"></a>Дальнейшие действия

* [Install an HDInsight application](/rest/api/hdinsight/hdinsight-application)(Установка приложения HDInsight) — узнайте, как разрабатывать шаблоны Resource Manager для развертывания приложений HDInsight.
* [Установка приложений в HDInsight](hdinsight-apps-install-applications.md). Узнайте, как установить приложения HDInsight в кластерах.
* [Публикация приложения HDInsight в Azure Marketplace](hdinsight-apps-publish-applications.md). Узнайте, как опубликовать пользовательские приложения HDInsight в Microsoft Azure Marketplace.
* [Настройка кластеров HDInsight под управлением Linux с помощью действия сценария](hdinsight-hadoop-customize-cluster-linux.md)— узнайте, как использовать действие скрипта для установки дополнительных приложений.
* [Создание кластеров Apache Hadoop под управлением Linux в HDInsight с помощью шаблонов Resource Manager](hdinsight-hadoop-create-linux-clusters-arm-templates.md) — узнайте, как вызывать шаблоны Resource Manager для создания кластеров HDInsight.
* [Использование пустых граничных узлов в HDInsight](hdinsight-apps-use-edge-node.md)— узнайте, как использовать пустой граничный узел для доступа к кластеру HDInsight, а также тестирования и размещения приложений HDInsight.
