---
title: Сведения о резервном копировании виртуальных машин Azure
description: В этой статье вы узнаете, как служба Azure Backup создает резервные копии виртуальных машин Azure и придерживайтесь рекомендаций.
ms.topic: conceptual
ms.date: 09/13/2019
ms.openlocfilehash: 691fe991ad141696c0c68e915d7225001a1befd0
ms.sourcegitcommit: 910a1a38711966cb171050db245fc3b22abc8c5f
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "98733576"
---
# <a name="an-overview-of-azure-vm-backup"></a>Общие сведения о резервном копировании виртуальных машин Azure

В этой статье описывается, как [служба Azure Backup](./backup-overview.md) создает резервные копии виртуальных машин Azure.

Azure Backup создает независимые, изолированные резервные копии для защиты от непреднамеренного уничтожения данных на виртуальных машинах. Резервные копии хранятся в хранилище служб восстановления со встроенным управлением точками восстановления. Конфигурация и масштабирование просты, резервное копирование оптимизировано, а восстановление выполняется легко.

В рамках процесса резервного копирования создается [моментальный снимок](#snapshot-creation), а данные передаются в хранилище служб восстановления без влияния на рабочие нагрузки. Моментальный снимок обеспечивает различные уровни согласованности, как описано [здесь](#snapshot-consistency).

Azure Backup также имеет специализированные предложения для рабочих нагрузок баз данных, таких как [SQL Server](backup-azure-sql-database.md) и [SAP HANA](sap-hana-db-about.md) , которые поддерживают рабочую нагрузку, предлагают 15-минутную RPO (цель точки восстановления) и позволяют выполнять резервное копирование и восстановление отдельных баз данных.

## <a name="backup-process"></a>Процесс резервного копирования

Вот как Azure Backup выполняет резервное копирование для виртуальных машин Azure.

1. Для виртуальных машин Azure, выбранных для резервного копирования, Azure Backup запускает задание резервного копирования в соответствии с заданным расписанием архивации.
1. Во время первой архивации на виртуальной машине устанавливается расширение резервного копирования, если виртуальная машина запущена.
    - Для виртуальных машин Windows установлено [расширение VMSnapshot](../virtual-machines/extensions/vmsnapshot-windows.md) .
    - Для виртуальных машин Linux установлено [расширение VMSnapshotLinux](../virtual-machines/extensions/vmsnapshot-linux.md) .
1. Для виртуальных машин Windows, работающих под управлением, резервное копирование координат с помощью Windows служба теневого копирования томов (VSS) для получения моментального снимка виртуальной машины, совместимой с приложениями.
    - По умолчанию резервное копирование занимает все резервные копии VSS.
    - Если резервное копирование не может принять моментальный снимок, основанное на файлах, то он принимает моментальный снимок базового хранилища (так как при остановке виртуальной машины не происходит операций записи приложений).
1. Для виртуальных машин Linux резервное копирование выполняется в соответствии с файлами. Для моментальных снимков, совместимых с приложениями, необходимо вручную настроить скрипты до и после.
1. После создания резервной копии моментальный снимок передает данные в хранилище.
    - В целях оптимизации резервное копирование дисков виртуальных машин осуществляется параллельно.
    - Azure Backup считывает блоки с каждого диска, для которого создается резервная копия, после чего определяет и передает только те блоки данных, в которых с момента предыдущего резервного копирования произошли изменения (дельта).
    - Данные моментального снимка могут копироваться в хранилище не сразу. Это может занять несколько часов при высокой нагрузке. Общее время архивации виртуальной машины при ежедневном резервном копировании составляет менее 24 часов.
1. Изменения, внесенные в виртуальную машину Windows после включения Azure Backup, приведены ниже.
    - Распространяемый пакет Microsoft Visual C++ 2013 (x64) — 12.0.40660 установлен на виртуальной машине.
    - Тип запуска службы теневого копирования томов (VSS) изменен на автоматический с ручной
    - Добавлена служба Windows IaaSVmProvider

1. После завершения перемещения данных моментальный снимок удаляется и создается точка восстановления.

![Архитектура резервного копирования виртуальных машин Azure](./media/backup-azure-vms-introduction/vmbackup-architecture.png)

## <a name="encryption-of-azure-vm-backups"></a>Шифрование резервных копий виртуальных машин Azure

При создании резервных копий виртуальных машин Azure с помощью Azure Backup виртуальные машины шифруются в неактивном состоянии с использованием Шифрования службы хранилища. Azure Backup также могут выполнять резервное копирование виртуальных машин Azure, зашифрованных с помощью шифрования дисков Azure.

**Шифрование** | **Сведения** | **Поддержка**
--- | --- | ---
**SSE** | Благодаря SSE служба хранилища Azure обеспечивает шифрование неактивных данных, автоматически шифруя данные перед их сохранением. Служба хранилища Azure также расшифровывает данные перед их получением. Azure Backup поддерживает резервное копирование виртуальных машин с двумя типами Шифрование службы хранилища:<li> **SSE с ключами, управляемыми платформой**: это шифрование по умолчанию для всех дисков на виртуальных машинах. Дополнительные сведения см. [здесь](../virtual-machines/disk-encryption.md#platform-managed-keys).<li> **SSE с ключами, управляемыми клиентом**. С помощью CMK вы управляете ключами, используемыми для шифрования дисков. Дополнительные сведения см. [здесь](../virtual-machines/disk-encryption.md#customer-managed-keys). | Azure Backup использует SSE для шифрования неактивных данных виртуальных машин Azure.
**Дисковое шифрование Azure** | Шифрование дисков Azure шифрует диски операционной системы и дисков данных для виртуальных машин Azure.<br/><br/> Шифрование дисков Azure интегрируется с ключами шифрования BitLocker (Бекс), которые защищается в хранилище ключей как секреты. Шифрование дисков Azure также интегрируется с ключами шифрования Azure Key Vault ключей (ключи обмена ключами). | Azure Backup поддерживает резервное копирование управляемых и неуправляемых виртуальных машин Azure, зашифрованных только с помощью Бекс, или с помощью Бекс вместе с ключи обмена ключами.<br/><br/> Бекс и ключи обмена ключами архивируются и шифруются.<br/><br/> Так как резервное копирование ключи обмена ключами и Бекс выполняется, пользователи с необходимыми разрешениями могут восстановить ключи и секреты в хранилище ключей, если это необходимо. Эти пользователи также могут восстановить зашифрованную виртуальную машину.<br/><br/> Зашифрованные ключи и секреты не могут быть прочитаны неавторизованными пользователями или Azure.

Для управляемых и неуправляемых виртуальных машин Azure служба архивации поддерживает как виртуальные машины, зашифрованные только с помощью Бекс, так и виртуальные машины, зашифрованные с помощью Бекс вместе с ключи обмена ключами.

Резервные Бекс (секреты) и ключи обмена ключами (ключи) шифруются. Они могут быть прочитаны и использованы только при восстановлении их в хранилище ключей полномочными пользователями. Ни неавторизованные пользователи, ни Azure, могут читать или использовать резервные ключи или секреты.

Также выполняется резервное копирование Бекс. Таким образом, если Бекс потеряны, полномочные пользователи могут восстановить Бекс в хранилище ключей и восстановить зашифрованные виртуальные машины. Только пользователи с достаточным уровнем разрешений могут выполнять резервное копирование и восстановление зашифрованных виртуальных машин или ключей и секретов.

## <a name="snapshot-creation"></a>Создание моментального снимка

Azure Backup создает моментальные снимки в соответствии с расписанием резервного копирования.

- **Виртуальные машины Windows:** Для виртуальных машин Windows служба резервного копирования координируется с VSS для создания моментального снимка дисков виртуальных машин, совместимых с приложениями.  По умолчанию Azure Backup создает полную резервную копию VSS (она усекает журналы приложения, такие как SQL Server во время резервного копирования, чтобы получить резервное копирование, обеспечивающее единообразие на уровне приложения).  Если вы используете базу данных SQL Server в резервной копии виртуальной машины Azure, вы можете изменить этот параметр, чтобы создать резервную копию копии VSS (для сохранения журналов). Дополнительные сведения см. в [этой статье](./backup-azure-vms-troubleshoot.md#troubleshoot-vm-snapshot-issues).

- **Виртуальные машины Linux:** Чтобы получить моментальные снимки виртуальных машин Linux с согласованием приложений, используйте платформу предварительных сценариев и пост-сценариев Linux для создания собственных пользовательских сценариев для обеспечения согласованности.

  - Azure Backup вызывает только скрипты, предшествующие или POST, которые вы написали.
  - Если скрипты предварительного и последующей выполнения выполняются успешно, Azure Backup помечает точку восстановления как совместимую с приложениями. Однако при использовании пользовательских сценариев вы в конечном итоге отвечаете за согласованность приложений.
  - Дополнительные [сведения](backup-azure-linux-app-consistent.md) о настройке скриптов.

## <a name="snapshot-consistency"></a>Согласованность моментального снимка

В следующей таблице описаны различные типы согласованности моментальных снимков.

**Моментальный снимок** | **Сведения** | **Восстановление** | **Рассмотрение**
--- | --- | --- | ---
**Согласованность с приложениями** | Согласованные с приложениями резервные копии включают содержимое памяти и ожидающие операции ввода-вывода. Моментальные снимки с согласованием приложений используют модуль записи VSS (или скрипты предварительной или публикации для Linux) для обеспечения согласованности данных приложения перед резервным копированием. | При восстановлении виртуальной машины с помощью моментального снимка, совместимого с приложениями, виртуальная машина загружается. В этом случае не происходит потери или повреждения данных. Приложение запускается в согласованном состоянии. | Windows: все модули записи VSS были успешной<br/><br/> Linux: скрипты предварительной и публикации настроены и выполняются
**Целостность файловой системы** | Согласованные с файловой системой резервные копии обеспечивают согласованность за счет моментального снимка всех файлов в одно и то же время.<br/><br/> | При восстановлении виртуальной машины с моментальным снимком, совместимым с файловой системой, виртуальная машина загружается. В этом случае не происходит потери или повреждения данных. Чтобы гарантировать согласованность восстановленных данных, в приложениях необходимо реализовать собственный механизм исправления. | Windows: сбой некоторых модулей записи VSS <br/><br/> Linux: по умолчанию (если скрипты pre и POST не настроены или не прошли сбой)
**Отказоустойчивость** | Моментальные снимки с постоянным аварийным завершением обычно возникают, если виртуальная машина Azure завершает работу во время резервного копирования. Во время архивации создается резервная копия только тех данных, которые уже существуют на диске. | Начинается с процесса загрузки виртуальной машины, за которым следует проверка диска для исправления ошибок повреждения. Все данные в памяти или операции записи, которые не были переданы на диск перед сбоем, теряются. В приложениях должна быть реализована собственная проверка данных. Например, приложение базы данных может использовать свой журнал транзакций для проверки. Если журнал транзакций содержит записи, которых нет в базе данных, то программное обеспечение базы данных выполняет откат транзакций до тех пор, пока данные не будут согласовываться. | Виртуальная машина находится в состоянии завершения работы (остановлена или освобождена).

>[!NOTE]
> Если состояние подготовки — " **успех**", Azure Backup создает резервные копии, совместимые с файловой системой. Если состояние подготовки **недоступно** или **завершилось сбоем**, создаются резервные копии с постоянным аварийным завершением. Если состояние подготовки — **Создание** или **Удаление**, это означает, что Azure Backup пытается повторить операции.

## <a name="backup-and-restore-considerations"></a>Вопросы резервного копирования и восстановления

**Рассмотрение** | **Сведения**
--- | ---
**Диск** | Резервное копирование дисков виртуальной машины параллельно. Например, если виртуальная машина содержит четыре диска, служба резервного копирования пытается выполнить резервное копирование всех четырех дисков параллельно. Резервное копирование является добавочным (только измененные данные).
**Планирование** |  Чтобы уменьшить трафик резервного копирования, выполните резервное копирование различных виртуальных машин в разное время суток и убедитесь, что время не перекрывается. Резервное копирование виртуальных машин в одно и то же время создает перегрузку.
**Подготовка резервных копий** | Учитывайте время, необходимое для подготовки резервной копии. Это время включает установку или обновление расширения резервного копирования, а также запуск моментального снимка в соответствии с расписанием резервного копирования.
**Перенос данных** | Рассмотрите время, необходимое для Azure Backup, чтобы определить добавочные изменения из предыдущей резервной копии.<br/><br/> Чтобы определить добавочные изменения, служба Azure Backup вычисляет контрольную сумму блоков. Если блок изменен, он помечается для перемещения в хранилище. Служба анализирует идентифицированные блоки, чтобы сократить объем передаваемых данных. После вычисления всех измененных блоков служба Azure Backup передает изменения в хранилище.<br/><br/> Возможна небольшая задержка между созданием моментального снимка и его копированием в хранилище. В пиковое время для передачи моментальных снимков в хранилище может потребоваться до восьми часов. Общее время резервного копирования виртуальной машины составляет менее 24 часов для ежедневного резервного копирования.
**Начальное резервное копирование** | Хотя для добавочных резервных копий общая продолжительность резервного копирования составляет менее 24 часов, на создание первой резервной копии может затрачиваться больше времени. Время, необходимое для создания первой резервной копии, зависит от размера данных и времени обработки резервной копии.
**Очередь восстановления** | Azure Backup одновременно обрабатывает задания восстановления из нескольких учетных записей хранения и помещает запросы на восстановление в очередь.
**Копирование при восстановлении** | Во время восстановления данные копируются из хранилища в учетную запись хранения.<br/><br/> Общее время восстановления зависит от количества операций ввода-вывода в секунду и пропускной способности учетной записи хранения.<br/><br/> Чтобы сократить время копирования, выберите учетную запись хранения, которая не загружена другими операциями чтения и записи приложений.

### <a name="backup-performance"></a>Производительность резервного копирования

Эти распространенные сценарии могут повлиять на общее время резервного копирования:

- **Добавление нового диска в защищенную виртуальную машину Azure:** Если виртуальная машина проходит добавочное резервное копирование и добавляется новый диск, то время архивации увеличится. Общее время архивации может соблюдаться в течение более 24 часов из-за начальной репликации нового диска, а также разностной репликации существующих дисков.
- **Фрагментированные диски:** Операции резервного копирования выполняются быстрее, если изменения диска являются смежными. Если изменения распределены по всему диску, резервное копирование будет выполняться медленнее.
- **Обработка диска:** Если защищенные диски, на которых выполняется добавочное резервное копирование, имеют ежедневное обновление более 200 ГБ, для завершения архивации может потребоваться длительное время (более восьми часов).
- **Версии резервных копий:** Последняя версия резервного копирования (известная как версия мгновенного восстановления) использует более оптимизированный процесс, чем сравнение контрольных сумм для выявления изменений. Но если вы используете мгновенное восстановление и удалили моментальный снимок резервной копии, резервное копирование переключается на сравнение контрольных сумм. В этом случае операция резервного копирования превысит 24 часа (или не будет работать).

### <a name="restore-performance"></a>Производительность восстановления

Эти распространенные сценарии могут повлиять на общее время восстановления:

- Общее время восстановления зависит от количества операций ввода-вывода в секунду и пропускной способности учетной записи хранения.
- Общее время восстановления может быть затронуто, если Целевая учетная запись хранения загружена с другими операциями чтения и записи приложения. Чтобы улучшить операцию восстановления, выберите учетную запись хранения, которая не загружена с другими данными приложений.

## <a name="best-practices"></a>Рекомендации

Ниже приведены рекомендации, которые следует учитывать при настройке резервного копирования виртуальных машин.

- Измените запланированное время по умолчанию в политике. Например, если время по умолчанию в политике — 12:00, увеличьте его шагами по несколько минут, чтобы обеспечить оптимальное использование ресурсов.
- При восстановлении виртуальных машин из одного хранилища настоятельно рекомендуется использовать разные [учетные записи хранения общего назначения версии 2](../storage/common/storage-account-upgrade.md) , чтобы гарантировать, что Целевая учетная запись хранения не будет отрегулирована. Например, каждая виртуальная машина должна иметь другую учетную запись хранения. Например, если восстановить 10 виртуальных машин, используйте 10 разных учетных записей хранения.
- Для резервного копирования виртуальных машин, использующих хранилище класса Premium с помощью мгновенного восстановления, рекомендуется выделить *50%* свободного пространства для всего выделенного пространства, которое требуется **только** для первой резервной копии. 50% свободного места не является обязательным для резервных копий после завершения первой архивации
- Ограничение на количество дисков в учетной записи хранения зависит от того, как часто к дискам обращаются приложения, работающие на виртуальной машине с поддержкой инфраструктуры как услуги (IaaS). В общем случае, если в одной учетной записи хранения есть от 5 до 10 дисков, сбалансируйте нагрузку, переместив некоторые диски в отдельные учетные записи хранения.
- Чтобы восстановить виртуальные машины с управляемыми дисками с помощью PowerShell, укажите дополнительный параметр ***TargetResourceGroupName*** , чтобы указать группу ресурсов, в которую будут восстановлены управляемые диски. [Дополнительные сведения](./backup-azure-vms-automation.md#restore-managed-disks)см. здесь.

## <a name="backup-costs"></a>Оплата резервного копирования

В отношении виртуальных машин Azure, резервное копирование которых выполняется с помощью службы Azure Backup, действует [прейскурант на использование службы Azure Backup](https://azure.microsoft.com/pricing/details/backup/).

Выставление счетов не начинается до завершения первой успешной архивации. На этом этапе начинается выставление счетов за службу хранилища и защищенные виртуальные машины. Выставление счетов выполняется, пока все данные резервных копий для виртуальной машины хранятся в хранилище. Если вы остановите защиту виртуальной машины, но данные резервного копирования виртуальной машины остаются в хранилище, выставление счетов продолжится.

Выставление счетов для определенной виртуальной машины прекращается, только если останавливается защита и удаляются все данные резервных копий. Когда защита останавливается и нет активных заданий резервного копирования, размер виртуальной машины во время последнего успешного резервного копирования становится размером защищенного экземпляра, на основе которого выставляется ежемесячный счет.

Вычисление размера защищенного экземпляра зависит от *фактического* размера виртуальной машины. Размер виртуальной машины — это сумма всех данных в виртуальной машине, исключая временное хранилище. Цены основываются на фактических данных, которые хранятся на дисках данных, а не на максимальном поддерживаемом размере для каждого диска данных, подключенного к виртуальной машине.

Аналогично, счет за хранилище резервных копий основывается на объеме данных, хранящихся в Azure Backup, который является суммой фактических данных в каждой точке восстановления.

Например, возьмем виртуальную машину размера "a2 — Стандартный" с двумя дополнительными дисками данных с максимальным размером в 32 ТБ. В следующей таблице показаны фактические данные, хранящиеся на каждом из этих дисков.

**Диск** | **Максимальный размер** | **Присутствующие фактические данные**
--- | --- | ---
Диск ОС | 32 ТБ | 17 ГБ
Локальный или временный диск | 135 ГБ | 5 ГБ (не учитывается для архивации)
Диск данных 1 | 32 ТБ| 30 ГБ
Диск данных 2 | 32 ТБ | 0 ГБ

Фактический размер виртуальной машины в этом случае составляет 17 ГБ + 30 ГБ + 0 ГБ = 47 ГБ. Этот размер защищенного экземпляра (47 ГБ) станет основанием для ежемесячного счета. По мере роста объема данных на виртуальной машине размер защищенного экземпляра, используемый для выставления счетов, изменяется на соответствие.

## <a name="next-steps"></a>Дальнейшие действия

- [Подготовка к резервному копированию виртуальных машин Azure](backup-azure-arm-vms-prepare.md).