---
title: Развертывание моделей машинного обучения в FPGA
titleSuffix: Azure Machine Learning
description: Сведения о программируемых массивах шлюзов. Вы можете развернуть веб-службу в FPGA с Машинное обучение Azure для вывода невысокой задержки.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: jordane
author: jpe316
ms.date: 09/24/2020
ms.topic: conceptual
ms.custom: how-to, contperf-fy21q2, devx-track-python, deploy
ms.openlocfilehash: 39c7d980bf9a90e5f72dfc9366d0ec44204b1ed2
ms.sourcegitcommit: f7eda3db606407f94c6dc6c3316e0651ee5ca37c
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/05/2021
ms.locfileid: "102212796"
---
# <a name="deploy-ml-models-to-field-programmable-gate-arrays-fpgas-with-azure-machine-learning"></a>Развертывание моделей машинного обучения в программируемых массивах Gate (FPGA) с Машинное обучение Azure 

В этой статье вы узнаете о FPGA и о том, как развернуть модели машинного обучения в Azure FPGA с помощью [пакета Python для моделей с аппаратным ускорением](/python/api/azureml-accel-models/azureml.accel?preserve-view=true&view=azure-ml-py) из [машинное обучение Azure](overview-what-is-azure-ml.md).

## <a name="what-are-fpgas"></a>Что такое FPGA?
ППВМ содержат массив программируемых логических блоков и иерархию настраиваемых взаимоподключений. Взаимоподключения позволяют настраивать эти блоки различными способами после производства. В отличие от других микросхем, ППВМ обеспечивают сочетание программируемости и производительности. 

FPGA позволяет достичь низкой задержки при выполнении запросов на вывод в режиме реального времени (или в результате оценки модели). Асинхронные запросы (пакетная обработка) не нужны. Пакетная обработка может повысить задержку, так как необходимо обработать больше данных. Для реализации модулей обработки нейронных процессоров не требуется пакетирование. Поэтому задержка может быть меньше, чем в сравнении с процессорами ЦП и GPU.

ППВМ можно перенастроить для различных типов моделей машинного обучения. Такая гибкость упрощает ускорение приложений на основе наиболее оптимальной точности числовых значений и используемой модели памяти. Так как ППВМ перенастраиваются, они могут поддерживать актуальные требования быстро меняющихся алгоритмов ИИ.

![Схема сравнения Машинное обучение Azure FPGA](./media/how-to-deploy-fpga-web-service/azure-machine-learning-fpga-comparison.png)

|Процессор| Сокращение |Описание|
|---|:-------:|------|
|Микросхемы приложений|ASIC|Пользовательские каналы, такие как единицы процессора тензорные Google (ТПУ), обеспечивают максимальную эффективность. Их нельзя настраивать повторно в случае изменения требований.|
|Программируемые пользователем вентильные матрицы|FPGA|ППВМ (например те, что доступны в Azure) обеспечивают производительность, близкую к микросхемам ASIC. Они также гибкие и поддерживают перенастройку для реализации новой логики.|
|Графические процессоры|GPU|Широко применяются для вычислений с использованием ИИ. Они обеспечивают возможности параллельной обработки, что позволяет выполнять отрисовку изображений быстрее, чем ЦП.|
|Центральные процессоры|ЦП|Процессоры общего назначения, производительность которых не подходит для обработки графики и видео.|

## <a name="fpga-support-in-azure"></a>Поддержка FPGA в Azure

Microsoft Azure – крупнейшая в мире облачная инвестиция в FPGA. Корпорация Майкрософт использует FPGA для оценки глубокой нейронной сети (DNN), ранжирования поиска Bing и ускорения программно-определяемой сети (SDN) для сокращения задержки, при этом освобождая процессоры для других задач.

FPGA в Azure базируется на устройствах Intel FPGA, которые специалисты по обработке и анализу данных и разработчики используют для ускорения вычислений искусственного интеллекта в реальном времени. Эта архитектура с поддержкой ППВМ доступна в Azure и обеспечивает производительность, гибкость и масштабирование.

Azure FPGAs интегрируется с Машинное обучение Azure. Azure может обучить предварительно обученные DNN по FPGA, чтобы масштабировать службу. DNN можно предварительно обучить как глубокий характеризатор для переноса обучения или настроить с использованием обновленных весовых коэффициентов.

|Сценарии & конфигурации в Azure|Поддерживаемые модели DNN|Региональная поддержка|
|--------------------------|--------------------|----------------|
|+ Классификация образов и сценарии распознавания<br/>+ TensorFlow развертывание (требуется Tensorflow 1. x)<br/>+ Оборудование Intel FPGA|-ResNet 50<br/>-ResNet 152<br/>-Денсенет-121<br/>-ВГГ-16<br/>-SSD-ВГГ|-Восточная часть США<br/>— Юго-Восточная Азия<br/> — Западная Европа<br/>-Западная часть США 2|

Чтобы оптимизировать задержку и пропускную способность, клиент, отправляющий данные в модель FPGA, должен находиться в одном из регионов выше (то, в котором была развернута модель).

**Семейство виртуальных машин Azure** содержит Intel АРРИА 10 FPGA. При проверке выделения квоты Azure она будет отображаться как "Стандартная гарнитура виртуальных ЦП". Виртуальная машина PB6 имеет шесть виртуальных ЦП и одно FPGA. Виртуальная машина PB6 автоматически подготавливается Машинное обучение Azure во время развертывания модели в FPGA. Он используется только с МАШИНным обучением Azure и не может выполнять произвольный битстреамс. Например, вы не сможете зафлэшировать FPGA с битстреамс для шифрования, кодирования и т. д.

## <a name="deploy-models-on-fpgas"></a>Развертывание моделей в ППВМ

Вы можете развернуть модель как веб-службу на FPGA с помощью [Машинное обучение Azure модели с аппаратным ускорением](/python/api/azureml-accel-models/azureml.accel?preserve-view=true&view=azure-ml-py). Использование FPGA обеспечивает немалое определение задержки, даже с одним размером пакета. 

В этом примере вы создадите граф TensorFlow для предварительной обработки входного изображения, сделайте его характеризатора с помощью ResNet 50 для FPGA, а затем запустите эти функции с помощью классификатора, обученного в наборе данных ImageNet. Затем модель развертывается в кластере AKS.

### <a name="prerequisites"></a>Предварительные требования

- Подписка Azure. Если у вас ее нет, создайте учетную запись с [оплатой по мере](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go) использования (свободные учетные записи Azure не подходят для квоты FPGA).

- Установлена рабочая область Машинное обучение Azure и пакет SDK для Машинное обучение Azure для Python, как описано в разделе [Создание рабочей области](how-to-manage-workspace.md).
 
- Пакет моделей с аппаратным ускорением:  `pip install --upgrade azureml-accel-models[cpu]`    
    
- [Интерфейс командной строки Azure](/cli/azure/install-azure-cli)

- Квота FPGA. Отправьте [запрос на квоту](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR2nac9-PZhBDnNSV2ITz0LNUN0U5S0hXRkNITk85QURTWk9ZUUFUWkkyTC4u)или выполните команду CLI, чтобы проверить квоту: 

    ```azurecli-interactive
    az vm list-usage --location "eastus" -o table --query "[?localName=='Standard PBS Family vCPUs']"
    ```

   Убедитесь, что в __куррентвалуе__ возвращено по меньшей мере 6 виртуальных ЦП.  

### <a name="define-the-tensorflow-model"></a>Определение модели TensorFlow

Начните с использования [пакета SDK для машинное обучение Azure для Python](/python/api/overview/azure/ml/intro?preserve-view=true&view=azure-ml-py) , чтобы создать определение службы. Определение службы представляет собой файл, описывающий конвейер графов (входящие данные, характеризатор и классификатор) на основе TensorFlow. Команда развертывания сжимает определение и графики в ZIP-файл и загружает ZIP-архив в хранилище больших двоичных объектов Azure. DNN уже развернут для запуска в FPGA.

1. Рабочая область Машинное обучение Azure нагрузки

   ```python
   import os
   import tensorflow as tf
   
   from azureml.core import Workspace
  
   ws = Workspace.from_config()
   print(ws.name, ws.resource_group, ws.location, ws.subscription_id, sep='\n')
   ```

1. Предварительная обработка образа. Входными данными для веб-службы являются изображения JPEG.  Первым шагом является декодирование изображения JPEG и его предварительная обработка.  Изображения JPEG обрабатываются как строки, а результатом являются десятки, которые будут входными данными для модели ResNet 50.

   ```python
   # Input images as a two-dimensional tensor containing an arbitrary number of images represented a strings
   import azureml.accel.models.utils as utils
   tf.reset_default_graph()
   
   in_images = tf.placeholder(tf.string)
   image_tensors = utils.preprocess_array(in_images)
   print(image_tensors.shape)
   ```

1. Загрузите характеризатора. Инициализируйте модель и загрузите контрольную точку TensorFlow квантованной версии ResNet50 для использования в качестве характеризатора.  Замените "QuantizedResnet50" в фрагменте кода для импорта других глубоких нейронных сетей:

   - QuantizedResnet152
   - QuantizedVgg16
   - Densenet121

   ```python
   from azureml.accel.models import QuantizedResnet50
   save_path = os.path.expanduser('~/models')
   model_graph = QuantizedResnet50(save_path, is_frozen=True)
   feature_tensor = model_graph.import_graph_def(image_tensors)
   print(model_graph.version)
   print(feature_tensor.name)
   print(feature_tensor.shape)
   ```

1. Добавьте классификатор. Этот классификатор был обучен по набору данных ImageNet.

   ```python
   classifier_output = model_graph.get_default_classifier(feature_tensor)
   print(classifier_output)
   ```

1. Сохранение модели. Теперь, когда препроцессор, ResNet 50 характеризатора и классификатор загружены, сохраните граф и связанные с ним переменные в качестве модели.

   ```python
   model_name = "resnet50"
   model_save_path = os.path.join(save_path, model_name)
   print("Saving model in {}".format(model_save_path))
   
   with tf.Session() as sess:
       model_graph.restore_weights(sess)
       tf.saved_model.simple_save(sess, model_save_path,
                                  inputs={'images': in_images},
                                  outputs={'output_alias': classifier_output})
   ```

1. Сохраняйте входные и выходные десятки, **так как они будут использоваться для преобразования модели и запросов вывода**. 

   ```python
   input_tensors = in_images.name
   output_tensors = classifier_output.name

   print(input_tensors)
   print(output_tensors)
   ```

   При использовании классификатора по умолчанию в списке доступны следующие модели со следующими моделями выходных данных классификатора для вывода.

   + Resnet50, QuantizedResnet50
     ```python
     output_tensors = "classifier_1/resnet_v1_50/predictions/Softmax:0"
     ```
   + Resnet152, QuantizedResnet152
     ```python
     output_tensors = "classifier/resnet_v1_152/predictions/Softmax:0"
     ```
   + Densenet121, QuantizedDensenet121
     ```python
     output_tensors = "classifier/densenet121/predictions/Softmax:0"
     ```
   + Vgg16, QuantizedVgg16
     ```python
     output_tensors = "classifier/vgg_16/fc8/squeezed:0"
     ```
   + Ссдвгг, Куантизедссдвгг
     ```python
     output_tensors = ['ssd_300_vgg/block4_box/Reshape_1:0', 'ssd_300_vgg/block7_box/Reshape_1:0', 'ssd_300_vgg/block8_box/Reshape_1:0', 'ssd_300_vgg/block9_box/Reshape_1:0', 'ssd_300_vgg/block10_box/Reshape_1:0', 'ssd_300_vgg/block11_box/Reshape_1:0', 'ssd_300_vgg/block4_box/Reshape:0', 'ssd_300_vgg/block7_box/Reshape:0', 'ssd_300_vgg/block8_box/Reshape:0', 'ssd_300_vgg/block9_box/Reshape:0', 'ssd_300_vgg/block10_box/Reshape:0', 'ssd_300_vgg/block11_box/Reshape:0']
     ```

### <a name="convert-the-model-to-the-open-neural-network-exchange-format-onnx"></a>Преобразование модели в формат открытого нейронного сетевого обмена (ONNX)

Прежде чем можно будет выполнить развертывание в FPGA, преобразуйте модель в формат [ONNX](https://onnx.ai/) .

1. [Зарегистрируйте](concept-model-management-and-deployment.md) модель с помощью пакета SDK и ZIP-файла в хранилище BLOB-объектов Azure. Добавление тегов и других метаданных о модели помогает контролировать обученные модели.

   ```python
   from azureml.core.model import Model

   registered_model = Model.register(workspace=ws,
                                     model_path=model_save_path,
                                     model_name=model_name)

   print("Successfully registered: ", registered_model.name,
         registered_model.description, registered_model.version, sep='\t')
   ```

   Если вы уже зарегистрировали модель и хотите загрузить ее, вы можете получить ее.

   ```python
   from azureml.core.model import Model
   model_name = "resnet50"
   # By default, the latest version is retrieved. You can specify the version, i.e. version=1
   registered_model = Model(ws, name="resnet50")
   print(registered_model.name, registered_model.description,
         registered_model.version, sep='\t')
   ```

1. Преобразуйте граф TensorFlow в формат ONNX.  Необходимо указать имена десятков входных и выходных данных, чтобы клиент мог использовать их при использовании веб-службы.

   ```python
   from azureml.accel import AccelOnnxConverter

   convert_request = AccelOnnxConverter.convert_tf_model(
       ws, registered_model, input_tensors, output_tensors)

   # If it fails, you can run wait_for_completion again with show_output=True.
   convert_request.wait_for_completion(show_output=False)

   # If the above call succeeded, get the converted model
   converted_model = convert_request.result
   print("\nSuccessfully converted: ", converted_model.name, converted_model.url, converted_model.version,
         converted_model.id, converted_model.created_time, '\n')
   ```

### <a name="containerize-and-deploy-the-model"></a>Контейнеризовать и развертывание модели

Затем создайте образ DOCKER из преобразованной модели и всех зависимостей.  После этого образ DOCKER можно развернуть и создать экземпляр.  Поддерживаемые целевые объекты развертывания включают службу Azure Kubernetes Service (AKS) в облаке или пограничном устройстве, например [Azure Data Box Edge](../databox-online/azure-stack-edge-overview.md).  Вы также можете добавить теги и описания для зарегистрированного образа DOCKER.

   ```python
   from azureml.core.image import Image
   from azureml.accel import AccelContainerImage

   image_config = AccelContainerImage.image_configuration()
   # Image name must be lowercase
   image_name = "{}-image".format(model_name)

   image = Image.create(name=image_name,
                        models=[converted_model],
                        image_config=image_config,
                        workspace=ws)
   image.wait_for_creation(show_output=False)
   ```

   Выведите список образов по тегу и получите подробные журналы для любой отладки.

   ```python
   for i in Image.list(workspace=ws):
       print('{}(v.{} [{}]) stored at {} with build log {}'.format(
           i.name, i.version, i.creation_state, i.image_location, i.image_build_log_uri))
   ```

#### <a name="deploy-to-an-azure-kubernetes-service-cluster"></a>Развертывание в кластере службы Kubernetes Azure

1. Чтобы развернуть модель в качестве высокомасштабируемой рабочей веб-службы, используйте AKS. Создать новую можно с помощью Машинное обучение Azure пакета SDK, интерфейса командной строки или [машинное обучение Azure Studio](https://ml.azure.com).

    ```python
    from azureml.core.compute import AksCompute, ComputeTarget
    
    # Specify the Standard_PB6s Azure VM and location. Values for location may be "eastus", "southeastasia", "westeurope", or "westus2". If no value is specified, the default is "eastus".
    prov_config = AksCompute.provisioning_configuration(vm_size = "Standard_PB6s",
                                                        agent_count = 1,
                                                        location = "eastus")
    
    aks_name = 'my-aks-cluster'
    # Create the cluster
    aks_target = ComputeTarget.create(workspace=ws,
                                      name=aks_name,
                                      provisioning_configuration=prov_config)
    ```

    Развертывание AKS может занять около 15 минут.  Проверьте, прошло ли развертывание.

    ```python
    aks_target.wait_for_completion(show_output=True)
    print(aks_target.provisioning_state)
    print(aks_target.provisioning_errors)
    ```

1. Разверните контейнер в кластере AKS.

    ```python
    from azureml.core.webservice import Webservice, AksWebservice
    
    # For this deployment, set the web service configuration without enabling auto-scaling or authentication for testing
    aks_config = AksWebservice.deploy_configuration(autoscale_enabled=False,
                                                    num_replicas=1,
                                                    auth_enabled=False)
    
    aks_service_name = 'my-aks-service'
    
    aks_service = Webservice.deploy_from_image(workspace=ws,
                                               name=aks_service_name,
                                               image=image,
                                               deployment_config=aks_config,
                                               deployment_target=aks_target)
    aks_service.wait_for_deployment(show_output=True)
    ```

#### <a name="deploy-to-a-local-edge-server"></a>Развертывание на локальном пограничном сервере

Все [Azure Data Box Edge устройства](../databox-online/azure-stack-edge-overview.md
) содержат FPGA для запуска модели.  Только одна модель может быть запущена в FPGA за один раз.  Чтобы запустить другую модель, просто разверните новый контейнер. Инструкции и примеры кода можно найти в [этом примере Azure](https://github.com/Azure-Samples/aml-hardware-accelerated-models).

### <a name="consume-the-deployed-model"></a>Использовать развернутую модель.

Наконец, используйте пример клиента для вызова образа DOCKER, чтобы получить прогнозы из модели.  Пример кода клиента доступен:
- [Python](https://github.com/Azure/aml-real-time-ai/blob/master/pythonlib/amlrealtimeai/client.py)
- [C#](https://github.com/Azure/aml-real-time-ai/blob/master/sample-clients/csharp)

Образ DOCKER поддерживает gRPC и TensorFlow, обслуживающий API-интерфейс "Predict".

Вы также можете скачать пример клиента для TensorFlow.

```python
# Using the grpc client in Azure ML Accelerated Models SDK package
from azureml.accel import PredictionClient

address = aks_service.scoring_uri
ssl_enabled = address.startswith("https")
address = address[address.find('/')+2:].strip('/')
port = 443 if ssl_enabled else 80

# Initialize Azure ML Accelerated Models client
client = PredictionClient(address=address,
                          port=port,
                          use_ssl=ssl_enabled,
                          service_name=aks_service.name)
```

Поскольку этот классификатор был обучен в наборе данных [ImageNet](http://www.image-net.org/) , сопоставьте классы с легко читаемыми метками.

```python
import requests
classes_entries = requests.get(
    "https://raw.githubusercontent.com/Lasagne/Recipes/master/examples/resnet50/imagenet_classes.txt").text.splitlines()

# Score image with input and output tensor names
results = client.score_file(path="./snowleopardgaze.jpg",
                            input_name=input_tensors,
                            outputs=output_tensors)

# map results [class_id] => [confidence]
results = enumerate(results)
# sort results by confidence
sorted_results = sorted(results, key=lambda x: x[1], reverse=True)
# print top 5 results
for top in sorted_results[:5]:
    print(classes_entries[top[0]], 'confidence:', top[1])
```

### <a name="clean-up-resources"></a>Очистка ресурсов

Чтобы избежать ненужных затрат, очистите ресурсы **в следующем порядке**: веб-служба, затем изображение, а затем модель.

```python
aks_service.delete()
aks_target.delete()
image.delete()
registered_model.delete()
converted_model.delete()
```

## <a name="next-steps"></a>Дальнейшие действия

+ Узнайте, как [защитить документ веб-служб](how-to-secure-web-service.md) .

+ Сведения о ценах и [затратах](https://azure.microsoft.com/pricing/details/machine-learning/)на FPGA и машинное обучение Azure.

+ [Масштабирование оборудования: ML в масштабе на основе Azure + FPGA: сборка 2018 (видео)](https://channel9.msdn.com/events/Build/2018/BRK3202)

+ [Настраиваемое облако на основе Microsoft FPGA (видео)](https://channel9.msdn.com/Events/Build/2017/B8063)

+ [Проект Браинваве для искусственного интеллекта в реальном времени](https://www.microsoft.com/research/project/project-brainwave/)

+ [Система автоматической оптической проверки](https://blogs.microsoft.com/ai/build-2018-project-brainwave/)
