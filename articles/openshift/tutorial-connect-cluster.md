---
title: Руководство по подключению к кластеру Azure Red Hat OpenShift 4
description: Сведения о подключении кластера Microsoft Azure Red Hat OpenShift
author: sakthi-vetrivel
ms.author: suvetriv
ms.topic: tutorial
ms.service: azure-redhat-openshift
ms.date: 04/24/2020
ms.openlocfilehash: 5295f0fbd28140bc0b278f3a44915239df200a70
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/20/2021
ms.locfileid: "102215669"
---
# <a name="tutorial-connect-to-an-azure-red-hat-openshift-4-cluster"></a>Руководство по подключению к кластеру Azure Red Hat OpenShift 4

В этом учебнике (часть 2 из 3) вы будете подключаться к кластеру Azure Red Hat OpenShift с OpenShift 4 в качестве пользователя kubeadm через веб-консоль OpenShift. Вы узнаете, как выполнять следующие задачи:
> [!div class="checklist"]
> * Получение учетных данных `kubeadmin` для кластера
> * Установка интерфейса командной строки OpenShift
> * Подключение к кластеру Azure Red Hat OpenShift с помощью интерфейса командной строки OpenShift

## <a name="before-you-begin"></a>Перед началом

Создание кластера Azure Red Hat OpenShift описывалось в предыдущих учебниках. Если вы не выполнили эти действия, вы можете начать с [руководства 1 по созданию кластера Azure Red Hat OpenShift 4](tutorial-create-cluster.md).

Если вы решили установить и использовать интерфейс командной строки локально, то для работы с этим руководством вам понадобится Azure CLI 2.6.0 или более поздней версии. Чтобы узнать версию, выполните команду `az --version`. Если вам необходимо выполнить установку или обновление, см. статью [Установка Azure CLI 2.0](/cli/azure/install-azure-cli).

## <a name="connect-to-the-cluster"></a>Подключение к кластеру

Вы можете войти в кластер как пользователь `kubeadmin`.  Выполните следующую команду, чтобы найти пароль для пользователя `kubeadmin`.

```azurecli-interactive
az aro list-credentials \
  --name $CLUSTER \
  --resource-group $RESOURCEGROUP
```

В следующем примере вывода показано, какой пароль будет указан в `kubeadminPassword`.

```json
{
  "kubeadminPassword": "<generated password>",
  "kubeadminUsername": "kubeadmin"
}
```

URL-адрес консоли кластера можно узнать, выполнив следующую команду. URL-адрес должен выглядеть вот так: `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`.

```azurecli-interactive
 az aro show \
    --name $CLUSTER \
    --resource-group $RESOURCEGROUP \
    --query "consoleProfile.url" -o tsv
```

Перейдите по URL-адресу консоли в браузере и войдите с помощью учетных данных `kubeadmin`.

![Экран входа в Azure Red Hat OpenShift](media/aro4-login.png)

## <a name="install-the-openshift-cli"></a>Установка интерфейса командной строки OpenShift

После входа в веб-консоль OpenShift щелкните **?** в правом верхнем углу, а затем выберите **Программы командной строки**. Скачайте выпуск, подходящий вашему компьютеру.

![Снимок экрана: в списке, который открывается при выборе элемента ?, выделен пункт "Программы командной строки". .](media/aro4-download-cli.png)

Вы также можете скачать последнюю версию интерфейса командной строки, подходящую для компьютера, по адресу <https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/>.

Если вы используете команды на Azure Cloud Shell, скачайте последнюю версию интерфейса командной строки OpenShift 4 для Linux.

```azurecli-interactive
cd ~
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz

mkdir openshift
tar -zxvf openshift-client-linux.tar.gz -C openshift
echo 'export PATH=$PATH:~/openshift' >> ~/.bashrc && source ~/.bashrc
```

## <a name="connect-using-the-openshift-cli"></a>Подключение с помощью интерфейса командной строки OpenShift

Получите адрес сервера API.

```azurecli-interactive
apiServer=$(az aro show -g $RESOURCEGROUP -n $CLUSTER --query apiserverProfile.url -o tsv)
```

Войдите на сервер API кластера OpenShift с помощью следующей команды. Замените **\<kubeadmin password>** паролем, который вы только что получили.

```azurecli-interactive
oc login $apiServer -u kubeadmin -p <kubeadmin password>
```

## <a name="next-steps"></a>Дальнейшие действия

В этой части руководства вы узнали, как выполнить следующие действия:
> [!div class="checklist"]
> * Получение учетных данных `kubeadmin` для кластера
> * Установка интерфейса командной строки OpenShift
> * Подключение к кластеру Azure Red Hat OpenShift с помощью интерфейса командной строки OpenShift

Перейдите к следующему руководству:
> [!div class="nextstepaction"]
> [Удаление кластера Azure Red Hat OpenShift](tutorial-delete-cluster.md)