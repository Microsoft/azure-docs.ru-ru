---
title: Краткое руководство. Создание управляемого экземпляра Azure для кластера Apache Cassandra на портале Microsoft Azure
description: В этом кратком руководстве описывается создание управляемого экземпляра Azure для кластера Apache Cassandra на портале Microsoft Azure.
author: TheovanKraay
ms.author: thvankra
ms.service: managed-instance-apache-cassandra
ms.topic: quickstart
ms.date: 03/02/2021
ms.custom: references_regions
ms.openlocfilehash: a05769c66c4b13de5c7197ef5612d64781574987
ms.sourcegitcommit: c27a20b278f2ac758447418ea4c8c61e27927d6a
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/03/2021
ms.locfileid: "101747909"
---
# <a name="quickstart-create-an-azure-managed-instance-for-apache-cassandra-cluster-from-the-azure-portal-preview"></a>Краткое руководство. Создание управляемого экземпляра Azure для кластера Apache Cassandra на портале Microsoft Azure (предварительная версия)
 
Управляемый экземпляр базы данных SQL Azure для Apache Cassandra позволяет автоматизировать операции развертывания и масштабирования для управляемых центров обработки данных Apache Cassandra с открытым кодом, ускоряя работу в гибридных сценариях и сокращая необходимый объем текущего обслуживания.

> [!IMPORTANT]
> Управляемый экземпляр базы данных SQL Azure для Apache Cassandra в настоящее время предоставляется в режиме общедоступной предварительной версии.
> Эта предварительная версия предоставляется без соглашения об уровне обслуживания и не рекомендована для использования рабочей среде. Некоторые функции могут не поддерживаться или их возможности могут быть ограничены.
> Дополнительные сведения см. в статье [Дополнительные условия использования предварительных выпусков Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

В этом кратком руководстве описывается, как создать управляемый экземпляр Azure для кластера Apache Cassandra на портале Microsoft Azure.

## <a name="prerequisites"></a>Предварительные требования

Если у вас еще нет подписки Azure, [создайте бесплатную учетную запись](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), прежде чем начинать работу.

## <a name="create-a-managed-instance-cluster"></a><a id="create-account"></a>Создание кластера с управляемым экземпляром базы данных SQL Azure

1. Войдите на [портал Azure](https://portal.azure.com/).

1. В строке поиска введите **Управляемый экземпляр базы данных SQL Azure для Apache Cassandra** и выберите результат.

   :::image type="content" source="./media/create-cluster-portal/search-portal.png" alt-text="Поиск управляемого экземпляра базы данных SQL Azure для Apache Cassandra" lightbox="./media/create-cluster-portal/search-portal.png" border="true":::

1. Нажмите кнопку **Создать управляемый экземпляр базы данных SQL Azure для кластера Apache Cassandra**.

   :::image type="content" source="./media/create-cluster-portal/create-cluster.png" alt-text="Создание кластера." lightbox="./media/create-cluster-portal/create-cluster.png" border="true":::

1. В области **Создать управляемый экземпляр базы данных SQL Azure для Apache Cassandra** укажите следующие сведения:

   * Для параметра **Подписка** выберите подписку Azure в раскрывающемся списке.
   * Для параметра **Группа ресурсов** укажите, нужно ли создать новую группу ресурсов или использовать существующую. Группа ресурсов — это контейнер, содержащий связанные ресурсы для решения Azure. Дополнительные сведения см. в обзорной статье [Группа ресурсов Azure](../azure-resource-manager/management/overview.md).
   * **Имя кластера**: введите имя кластера.
   * **Расположение**: расположение, в котором будет развернут кластер.
   * **SKU**: тип SKU для кластера.
   * **Количество узлов**: количество узлов в кластере. Эти узлы выполняют функцию реплик для ваших данных.
   * **Начальный пароль администратора Cassandra**: пароль, используемый для создания кластера.
   * **Подтвердите пароль администратора Cassandra**: введите пароль повторно.

    > [!NOTE]
    > В общедоступной предварительной версии можно создать кластер с управляемым экземпляром в *восточной части США, западной части США, восточной части США 2, западной части США 2, центральной части, юго-центральной части США, Северной Европе, Западной Европе, Юго-Восточной Азии и восточной Австралии*.

   :::image type="content" source="./media/create-cluster-portal/create-cluster-page.png" alt-text="Заполнение формы для создания кластера." lightbox="./media/create-cluster-portal/create-cluster-page.png" border="true":::

1. Перейдите на вкладку **Сеть**.

1. В области **Сеть** выберите имя **виртуальной сети Microsoft Azure** и **подсеть**. Можно выбрать существующую виртуальную сеть Microsoft Azure или создать новую.

   :::image type="content" source="./media/create-cluster-portal/networking.png" alt-text="Настройка параметров сетевого подключения." lightbox="./media/create-cluster-portal/networking.png" border="true":::

1. Если вы создали новую виртуальную сеть на последнем шаге, перейдите к шагу 9. Если вы выбрали существующую виртуальную сеть Microsoft Azure, перед созданием кластера необходимо применить некоторые специальные разрешения к виртуальной сети и подсети. Для этого необходимо получить идентификатор ресурса для существующей виртуальной сети Microsoft Azure. Выполните следующую команду в [Azure CLI](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli):

   ```azurecli-interactive
   # get the resource ID of the Virtual Network
   az network vnet show -n <VNet_name> -g <Resource_Group_Name> --query "id" --output tsv

1. Now apply the special permissions by using the `az role assignment create` command. Replace `<Resource ID>` with the output of previous command:

   ```azurecli-interactive
   az role assignment create --assignee e5007d2c-4b13-4a74-9b6a-605d99f03501 --role 4d97b98b-1d4f-4787-a291-c67834d212e7 --scope <Resource ID>
   ```

   > [!NOTE]
   > Значения `assignee` и `role` в предыдущей команде — это фиксированные идентификаторы субъекта-службы и роли соответственно.

1. После завершения работы настройки сети выберите **Просмотр и создание** > **Создать**

    > [!NOTE]
    > Создание кластера может занять до 15 минут.

   :::image type="content" source="./media/create-cluster-portal/review-create.png" alt-text="Проверка сводки для создания кластера." lightbox="./media/create-cluster-portal/review-create.png" border="true":::


1. После завершения развертывания проверьте группу ресурсов. В ней должен отображаться созданный кластер с управляемым экземпляром:

   :::image type="content" source="./media/create-cluster-portal/managed-instance.png" alt-text="Обзорная страница после создания кластера." lightbox="./media/create-cluster-portal/managed-instance.png" border="true":::

1. Чтобы просмотреть узлы кластера, перейдите к области виртуальной сети Microsoft Azure, которая использовалась для создания кластера, и откройте панель **Обзор**, чтобы увидеть узлы:

   :::image type="content" source="./media/create-cluster-portal/resources.png" alt-text="Просмотр ресурсов кластера." lightbox="./media/create-cluster-portal/resources.png" border="true":::



## <a name="connecting-to-your-cluster"></a>Подключение к кластеру

Управляемый экземпляр базы данных SQL Azure для Apache Cassandra не создает узлы с общедоступными IP-адресами, поэтому для подключения к созданному кластеру Cassandra необходимо создать другой ресурс в виртуальной сети Microsoft Azure. Это может быть приложение или виртуальная машина с установленным средством обработки запросов с открытым кодом [CQLSH](https://cassandra.apache.org/doc/latest/tools/cqlsh.html) от Apache. Можно использовать [шаблон](https://azure.microsoft.com/resources/templates/101-vm-simple-linux/) для развертывания виртуальной машины Ubuntu. При развертывании используйте SSH для подключения к компьютеру и установите CQLSH с помощью следующих команд:

```bash
# Install default-jre and default-jdk
sudo apt update
sudo apt install openjdk-8-jdk openjdk-8-jre

# Install the Cassandra libraries in order to get CQLSH:
echo "deb http://www.apache.org/dist/cassandra/debian 311x main" | sudo tee -a /etc/apt/sources.list.d/cassandra.sources.list
curl https://downloads.apache.org/cassandra/KEYS | sudo apt-key add -
sudo apt-get update
sudo apt-get install cassandra

# Export the SSL variables:
export SSL_VERSION=TLSv1_2
export SSL_VALIDATE=false

# Connect to CQLSH (replace <IP> with the private IP addresses of the nodes in your Datacenter):
host=("<IP>" "<IP>" "<IP>")
cqlsh $host 9042 -u cassandra -p cassandra --ssl
```

## <a name="clean-up-resources"></a>Очистка ресурсов

Если вы не планируете в дальнейшем использовать кластер с управляемым экземпляром, удалите его, выполнив следующие действия:

1. В меню слева на портале Microsoft Azure выберите **Группы ресурсов**.
1. Выберите из списка группу ресурсов, созданную для этого краткого руководства.
1. В области **Обзор** на странице группы ресурсов выберите **Удалить группу ресурсов**.
1. В следующем окне введите имя группы ресурсов, которую требуется удалить, и щелкните **Удалить**.

## <a name="next-steps"></a>Дальнейшие действия

Из этого краткого руководства вы узнали, как создать управляемый экземпляр Azure для кластера Apache Cassandra на портале Microsoft Azure. Теперь можно приступить к работе с кластером:

> [!div class="nextstepaction"]
> [Развертывание управляемого кластера Apache Spark с Azure Databricks](deploy-cluster-databricks.md)
