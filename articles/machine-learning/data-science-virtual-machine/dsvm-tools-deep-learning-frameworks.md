---
title: Глубокое обучение & платформах искусственного интеллекта
titleSuffix: Azure Data Science Virtual Machine
description: Доступные платформы и средства глубокого обучения на виртуальной машине Azure для обработки и анализа данных.
keywords: средства анализа и обработки данных, виртуальная машина для анализа и обработки данных, средства для анализа и обработки данных, анализ и обработка данных Linux
services: machine-learning
ms.service: data-science-vm
ms.custom: devx-track-python
author: lobrien
ms.author: laobri
ms.topic: conceptual
ms.date: 07/17/2020
ms.openlocfilehash: 9ee0e8d8fa660313cb007d53c2bbbfc9b19feec9
ms.sourcegitcommit: a67b972d655a5a2d5e909faa2ea0911912f6a828
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/23/2021
ms.locfileid: "104889779"
---
# <a name="deep-learning-and-ai-frameworks-for-the-azure-data-science-vm"></a>Платформы глубокого обучения и искусственного интеллекта для виртуальной машины Azure для обработки и анализа данных
Платформы глубокого обучения в DSVM перечислены ниже.

## <a name="caffe"></a>[Caffe](https://github.com/BVLC/caffe)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | |
| Поддерживаемые выпуски DSVM      | Ubuntu 16.04    |
| Настройка и установка на DSVM  | Caffe устанавливается в `/opt/caffe`.   Примеры приведены в `/opt/caffe/examples` .|
| Как запустить      | Используйте X2Go для входа в виртуальную машину, а затем запустите новый терминал и введите следующее:<br/>`cd /opt/caffe/examples`<br/>`source activate root`<br/>`jupyter notebook`<br/><br/>Откроется новое окно браузера с примерами записных книжек. Двоичные файлы устанавливаются в каталог /opt/caffe/build/install/bin.<br/><br/>Для установленной версии Caffe требуется Python 2,7 и не будет работать с Python 3,6, которая активируется по умолчанию. Чтобы переключиться на Python 2,7, выполните команду, `source activate root` чтобы переключиться в среду Anaconda.|    

## <a name="caffe2"></a>[Caffe2](https://github.com/caffe2/caffe2)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | |
| Поддерживаемые выпуски DSVM      | Ubuntu 16.04     |
| Настройка и установка на DSVM  | Caffe2 устанавливается в среде [Python 2,7 (root) conda. |
| Как запустить      | Терминал: запустите Python и импортируйте Caffe2. <br/> * JupyterHub: [подключитесь к JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), а затем перейдите в каталог Caffe2, чтобы найти примеры записных книжек. Для некоторых записных книжек требуется корневой каталог Caffe2, чтобы их можно было задать в коде Python. Введите /opt/caffe2. |

## <a name="chainer"></a>[Chainer](https://chainer.org/)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 5,2 |
| Поддерживаемые выпуски DSVM      | Ubuntu 16.04    |
| Настройка и установка на DSVM  | Chainer устанавливается с Python 3.5. |
| Как запустить      | Терминал: активируйте среду Python 3,5, запустите `python` , а затем `import chainer` . <br/> * JupyterHub: [подключитесь к JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), а затем перейдите в каталог Chain, чтобы найти примеры записных книжек.| 

## <a name="cuda-cudnn-nvidia-driver"></a>[CUDA, cuDNN, драйвер NVIDIA](https://developer.nvidia.com/cuda-toolkit)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 10.0.130|
| Поддерживаемые выпуски DSVM      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016<br> Ubuntu 16.04  |
| Настройка и установка на DSVM  |_nvidia-smi_ можно найти в системном пути.  |
| Как запустить      | Откройте командную строку (в Windows) или терминал (в Linux), а затем запустите _NVIDIA-SMI_. |


## <a name="horovod"></a>[Horovod](https://github.com/uber/horovod)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 0.16.1|
| Поддерживаемые выпуски DSVM      | Ubuntu 18.04<br> Ubuntu 16.04   |
| Настройка и установка на DSVM  | Хоровод устанавливается в Python 3,5 |
| Как запустить      | Активируйте нужную среду в терминале, а затем запустите Python. |

## <a name="keras"></a>[Keras](https://keras.io/);

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 2.2.4 |
| Поддерживаемые выпуски DSVM      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016 <br> Ubuntu 16.04   |
| Настройка и установка на DSVM  | Keras устанавливается в Python 3,6 в Windows и в Python 3,5 в Linux |
| Как запустить      | Активируйте нужную среду в терминале, а затем запустите Python. |

## <a name="microsoft-cognitive-toolkit-cntk"></a>[Microsoft Cognitive Toolkit (CNTK)](/cognitive-toolkit/)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 2.5.1 |
| Поддерживаемые выпуски DSVM      | Windows 2016 <br> Ubuntu 16.04   |
| Настройка и установка на DSVM  | CNTK устанавливается в Python 3,6 в [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) и в Python 3,5 в [Linux](./dsvm-tools-languages.md#python-linux-edition)) |
| Как запустить      | Терминал: активируйте правильную среду и запустите Python. <br/>Jupyter: подключитесь к [Jupyter](provision-vm.md) или [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), а затем откройте каталог CNTK для примеров. |

## <a name="mxnet"></a>[MXNet](https://mxnet.apache.org/)
| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 1.3.0 |
| Поддерживаемые выпуски DSVM      | Windows 2016 <br> Ubuntu 16.04    |
| Настройка и установка на DSVM  | MXNet устанавливается в `C:\dsvm\tools\mxnet` в Windows и `/dsvm/tools/mxnet` Ubuntu. Привязки Python устанавливаются в Python 3,6 в [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) и в Python 3,5 в [Linux](./dsvm-tools-languages.md#python-linux-edition)) привязки R также входят в DSVM Ubuntu. |
| Как запустить      | Терминал: активируйте правильную среду conda, а затем запустите `import mxnet` . <br/>Jupyter: подключитесь к [Jupyter](provision-vm.md#access-the-dsvm) или [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), а затем откройте `mxnet` каталог для примеров. |

## <a name="mxnet-model-server"></a>[Сервер модели MXNet](https://github.com/awslabs/mxnet-model-server#quick-start)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 1.0.1 |
| Поддерживаемые выпуски DSVM      | Windows 2016 <br> Ubuntu 16.04    |
| Настройка и установка на DSVM  | MXNet Model Server устанавливается в Python 3,6 в [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) и в Python 3,5 в [Linux](./dsvm-tools-languages.md#python-linux-edition)). |
| Как запустить      | Терминал: выполните `sudo systemctl stop jupyterhub` , чтобы сначала запустить службу JupyterHub, так как оба прослушивают один и тот же порт. Затем активируйте правильную среду conda и запустите `mxnet-model-server --start --models squeezenet=https://s3.amazonaws.com/model-server/model_archive_1.0/squeezenet_v1.1.mar` |

## <a name="nvidia-system-management-interface-nvidia-smi"></a>[Интерфейс управления системами NVidia (NVIDIA-SMI)](https://developer.nvidia.com/nvidia-system-management-interface)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии |  |
| Поддерживаемые выпуски DSVM      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016<br> Ubuntu 16.04   |
| Для чего она нужна? | Средство NVIDIA для выполнения запросов активности GPU |
| Настройка и установка на DSVM  | `nvidia-smi` находится в системном пути. |
| Как запустить      | На виртуальной машине **с графическим процессором** откройте командную строку (в Windows) или терминал (в Linux) и запустите `nvidia-smi` . |

## <a name="pytorch"></a>[PyTorch](https://pytorch.org/)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 1.2.0 (Ubuntu 16,04), 1.7.1 (Ubuntu 18,04, Windows 2019) |
| Поддерживаемые выпуски DSVM      | Windows Server 2019<br>Ubuntu 18.04<br> Ubuntu 16.04 |
| Настройка и установка на DSVM  | Устанавливается в [Python 3,7](dsvm-tools-languages.md#python-linux-edition) в среде py37_pytorch conda. Включены образцы записных книжек Jupyter, а примеры находятся в папке/dsvm/Samples/pytorch. |
| Как запустить      | Терминал: активируйте правильную среду, а затем запустите Python.<br/>* [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine): Connect, а затем откройте каталог PyTorch для примеров.  |

## <a name="tensorflow"></a>[TensorFlow](https://www.tensorflow.org/);

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 1.13 |
| Поддерживаемые выпуски DSVM      | Windows Server 2019<br>Ubuntu 18.04<br> Windows 2016 <br> Ubuntu 16.04 |
| Настройка и установка на DSVM  | Устанавливается в Python 3,5 в [Linux](dsvm-tools-languages.md#python-linux-edition) и Python 3,6 в [Windows 2016](dsvm-tools-languages.md#python-windows-server-2016-edition) |
| Как запустить      | Терминал: активируйте правильную среду, а затем запустите Python. <br/> * Jupyter: подключитесь к [Jupyter](provision-vm.md) или [JupyterHub](dsvm-ubuntu-intro.md#how-to-access-the-ubuntu-data-science-virtual-machine), а затем откройте каталог TensorFlow для примеров.   |

## <a name="tensorflow-serving"></a>[TensorFlow Serving](https://www.tensorflow.org/serving/)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 1.12 |
| Поддерживаемые выпуски DSVM      | Ubuntu 16.04 |
| Настройка и установка на DSVM  | tensorflow_model_server доступен в терминале. |
| Как запустить      |  Примеры можно найти в [Интернете](https://www.tensorflow.org/serving/).   |


## <a name="theano"></a>[Theano](https://github.com/Theano/Theano)

| Категория | Значение |
| ------------- | ------------- |
| Поддерживаемые версии | 1.0.3 |
| Поддерживаемые выпуски DSVM      | Ubuntu 16.04 |
| Настройка и установка на DSVM  |Theano устанавливается в среде Python 2,7 (_root_) и в окружении Python 3,5 (_PY35_). |
| Как запустить      |  Терминал: активируйте нужную версию Python (root или PY35), запустите Python, а затем импортируйте Theano.<br/>* Jupyter: выберите ядро Python 2,7 или 3,5, а затем импортируйте Theano.  <br/>Чтобы обойти недавную ошибку математической библиотеки ядра (MKL), необходимо сначала установить MKL потоковый уровень следующим образом:<br/><br/>`export MKL_THREADING_LAYER=GNU`  |
