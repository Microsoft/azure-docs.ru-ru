---
title: Видео о приложении SaaS SQL
description: В этой статье индексированы различные точки во времени нашего 81-минутного видео о создании клиентских приложений базы данных SaaS с конференции Ignite, проведенной 11 октября 2017 года. Можно сразу перейти к интересующей вас части. Описано как минимум 3 шаблона. Описываются компоненты Azure, которые упрощают разработку и управление.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.custom: sqldbrb=1
ms.date: 12/18/2018
ms.topic: conceptual
ms.openlocfilehash: 8cceeef7e41e8fc5c48b4e67cdb1adf3aa4f02a1
ms.sourcegitcommit: 772eb9c6684dd4864e0ba507945a83e48b8c16f0
ms.translationtype: MT
ms.contentlocale: ru-RU
ms.lasthandoff: 03/19/2021
ms.locfileid: "89483870"
---
# <a name="video-indexed-and-annotated-for-multi-tenant-saas-app-using-azure-sql-database"></a>Видео индексировано и снабжено заметками для мультитенатного приложения SaaS, использующего базу данных SQL Azure
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Эта статья является индексом с заметками по точкам во времени в 81-минутном видеоролике о моделях или шаблонах клиента SaaS. Эта статья позволяет переходить по видео к той части, которая вас интересует. В видео описаны основные варианты создания мультитенатного приложения базы данных в Базе данных Azure SQL. Видео включает демонстрации, пошаговые руководства по коду управления и в некоторых случаях более подробно рассматриваются сведения, которые можно найти в письменной документации.

В видео представлена расширенная информация из письменной документации: 
- *Основные понятия.* [Мультитенантные шаблоны клиента в базе данных SaaS][saas-concept-design-patterns-563e].
- *Руководства.* [SaaS-приложение Wingtip Tickets][saas-how-welcome-wingtip-app-679t].

В видео и статьях описано множество этапов создания мультитенантных приложений в Базе данных SQL Azure в облаке. Специальные возможности Базы данных SQL Azure упрощают процесс разработки и реализации мультенантных приложений, которые просты в управлении, надежны и высокопроизводительны.

Мы регулярно обновляем нашу письменную документацию. Видео не будет изменяться или обновляться, поэтому все больше его частей будут устаревать.



## <a name="sequence-of-38-time-indexed-screenshots"></a>Последовательность 38 индексированных по времени снимков экрана

В этом разделе индексируется временное местоположение 38 обсуждений на протяжении 81-минутного видео. Каждый раз индекс времени помечается снимком экрана из видео и иногда — с дополнительными сведениями.

Каждый индекс времени имеет формат *ч:мм:сс*. Например, второе индексированное временное местоположение имеет метку **Цели сеанса** и начинается приблизительно с метки времени **0:03:11**.


### <a name="compact-links-to-video-indexed-time-locations"></a>Компактные ссылки на индексированное временное местоположение в видео

Следующие заголовки — это ссылки на соответствующие разделы с заметками данной статьи:

- [1. **(начало)** слайд приветствия, 0:00:03](#anchor-image-wtip-min00001)
- [2. цели сеанса, 0:03:11](#anchor-image-wtip-min00311)
- [3. Повестка, 0:04:17](#anchor-image-wtip-min00417)
- [4. веб-приложение с несколькими клиентами, 0:05:05](#anchor-image-wtip-min00505)
- [5. веб-форма приложения в действии, 0:05:55](#anchor-image-wtip-min00555)
- [6. стоимость каждого клиента (масштабирование, изоляция, восстановление), 0:09:31](#anchor-image-wtip-min00931)
- [7. модели баз данных для нескольких клиентов: достоинства и недостатки, 0:11:59](#anchor-image-wtip-min01159)
- [8. Гибридная модель сочетает в себе преимущества MT/ST, 0:13:01](#anchor-image-wtip-min01301)
- [9. один клиент и несколько клиентов: достоинства и недостатки, 0:16:44](#anchor-image-wtip-min01644)
- [10. пулы экономичны для непредсказуемых рабочих нагрузок, 0:19:36](#anchor-image-wtip-min01936)
- [11. Демонстрация базы данных на клиентские и гибридные версии Сент-MT, 0:20:08](#anchor-image-wtip-min02008)
- [12. форма Live App, в которой отображается Dojo, 0:20:29](#anchor-image-wtip-min02029)
- [13. MYOB, а не администратор базы данных, 0:28:54](#anchor-image-wtip-min02854)
- [14. Пример использования эластичного пула MYOB, 0:29:40](#anchor-image-wtip-min02940)
- [15. обучение от MYOB и других независимых поставщиков программного обеспечения, 0:31:36](#anchor-image-wtip-min03136)
- [16. шаблоны, составленные в сценарии SaaS E2E, 0:43:15](#anchor-image-wtip-min04315)
- [17. каноническое гибридное многоклиентское приложение SaaS, 0:47:33](#anchor-image-wtip-min04733)
- [18. пример приложения SaaS Wingtip, 0:48:10](#anchor-image-wtip-min04810)
- [19. сценарии и шаблоны, которые рассматриваются в руководствах, 0:49:10](#anchor-image-wtip-min04910)
- [20. Демонстрация учебников и репозитория GitHub, 0:50:18](#anchor-image-wtip-min05018)
- [21. репозиторий GitHub Microsoft/WingtipSaaS, 0:50:38](#anchor-image-wtip-min05038)
- [22. Изучение шаблонов, 0:56:20](#anchor-image-wtip-min05620)
- [23. подготовка клиентов и адаптации, 0:57:44](#anchor-image-wtip-min05744)
- [24. подготовка клиентов и подключение приложений, 0:58:58](#anchor-image-wtip-min05858)
- [25. Демонстрация сценариев управления, подготавливающих один клиент, 0:59:43](#anchor-image-wtip-min05943)
- [26. PowerShell для инициализации и каталогизации, 1:00:02](#anchor-image-wtip-min10002)
- [27. T-SQL SELECT * FROM TenantsExtended, 1:03:30](#anchor-image-wtip-min10330)
- [28. Управление непредсказуемыми рабочими нагрузками клиента, 1:04:36](#anchor-image-wtip-min10436)
- [29. Мониторинг эластичного пула, 1:06:39](#anchor-image-wtip-min10639)
- [30. создание нагрузки и мониторинг производительности, 1:09:42](#anchor-image-wtip-min10942)
- [31. Управление схемами в масштабе, 1:10:33](#anchor-image-wtip-min11033)
- [32. распределенный запрос между базами данных клиента, 1:12:21](#anchor-image-wtip-min11221)
- [33. Демонстрация создания билетов, 1:12:32](#anchor-image-wtip-min11232)
- [34. нерегламентированная аналитика SSMS, 1:12:46](#anchor-image-wtip-min11246)
- [35. Извлечение данных клиента в Azure синапсе Analytics, 1:16:32](#anchor-image-wtip-min11632)
- [36. график распределения ежедневных продаж, 1:16:48](#anchor-image-wtip-min11648)
- [37. заключение и вызов действия, 1:19:52](#anchor-image-wtip-min11952)
- [38. ресурсы для получения дополнительных сведений, 1:20:42](#anchor-image-wtip-min12042)


&nbsp;

### <a name="annotated-index-time-locations-in-the-video"></a>Отмеченные местоположения индекса времени в видео

Щелкнув любое изображение снимка экрана, можно перейти к точному временному расположению в видео.


&nbsp; <a name="anchor-image-wtip-min00001"></a>
#### <a name="1-start-welcome-slide-00001"></a>1. *(начало)* слайд приветствия, 0:00:01

*Обучение от MYOB. Конструктивные шаблоны для приложений SaaS в Базе данных SQL Azure — BRK3120*

[![Слайд приветствия][image-wtip-min00003-brk3120-whole-welcome]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1)

- Заголовок. Обучение от MYOB. Конструктивные шаблоны для приложений SaaS в Базе данных SQL Azure
- Bill.Gibson@microsoft.com
- Главный менеджер по программам, база данных SQL Azure
- Сеанс Microsoft Ignite BRK3120, Орландо, штат Флорида (США), октябрь 2017 года


&nbsp; <a name="anchor-image-wtip-min00311"></a>
#### <a name="2-session-objectives-00153"></a>2. цели сеанса, 0:01:53
[![Цели сеанса][image-wtip-min00311-session]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=113)

- Альтернативные модели для мультитенатных приложений, преимущества и недостатки.
- Шаблоны SaaS, снижающие затраты на разработку, управление и ресурсы.
- Пример приложения и скрипты.
- Возможности PaaS и шаблоны SaaS делают базу данных SQL высокомасштабируемой и экономичной платформой данных для мультитенантных SaaS.


&nbsp; <a name="anchor-image-wtip-min00417"></a>
#### <a name="3-agenda-00409"></a>3. Повестка, 0:04:09
[![Программа][image-wtip-min00417-agenda]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=249)


&nbsp; <a name="anchor-image-wtip-min00505"></a>
#### <a name="4-multi-tenant-web-app-00500"></a>4. веб-приложение с несколькими клиентами, 0:05:00
[![Приложение SaaS Wingtip. Мультитенантное веб-приложение][image-wtip-min00505-web-app]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=300)


&nbsp; <a name="anchor-image-wtip-min00555"></a>
#### <a name="5-app-web-form-in-action-00539"></a>5. веб-форма приложения в действии, 0:05:39
[![Форма веб-приложения в действии][image-wtip-min00555-app-web-form]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=339)


&nbsp; <a name="anchor-image-wtip-min00931"></a>
#### <a name="6-per-tenant-cost-scale-isolation-recovery-00658"></a>6. стоимость каждого клиента (масштабирование, изоляция, восстановление), 0:06:58
[![Стоимость, масштабирование, изоляция и восстановление каждого клиента][image-wtip-min00931-per-tenant-cost]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=418)


&nbsp; <a name="anchor-image-wtip-min01159"></a>
#### <a name="7-database-models-for-multi-tenant-pros-and-cons-00952"></a>7. модели баз данных для нескольких клиентов: достоинства и недостатки, 0:09:52
[![Модели базы данных для нескольких клиентов: достоинства и недостатки][image-wtip-min01159-db-models-pros-cons]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=592)


&nbsp; <a name="anchor-image-wtip-min01301"></a>
#### <a name="8-hybrid-model-blends-benefits-of-mtst-01229"></a>8. Гибридная модель сочетает в себе преимущества MT/ST, 0:12:29
[![Гибридная модель сочетает в себе преимущества MT/ST][image-wtip-min01301-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=749)


&nbsp; <a name="anchor-image-wtip-min01644"></a>
#### <a name="9-single-tenant-vs-multi-tenant-pros-and-cons-01311"></a>9. один клиент и несколько клиентов: достоинства и недостатки, 0:13:11
[![Сравнение одного клиента и нескольких клиентов. Преимущества и недостатки][image-wtip-min01644-st-vs-mt]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=791)


&nbsp; <a name="anchor-image-wtip-min01936"></a>
#### <a name="10-pools-are-cost-effective-for-unpredictable-workloads-01749"></a>10. пулы экономичны для непредсказуемых рабочих нагрузок, 0:17:49
[![Пулы экономически эффективны при непредсказуемых рабочих нагрузках][image-wtip-min01936-pools-cost]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1069)


&nbsp; <a name="anchor-image-wtip-min02008"></a>
#### <a name="11-demo-of-database-per-tenant-and-hybrid-stmt-01959"></a>11. Демонстрация базы данных на клиентские и гибридные версии Сент-MT, 0:19:59
[![Демонстрация базы данных на клиент и Гибридная версия][image-wtip-min02008-demo-st-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1199)


&nbsp; <a name="anchor-image-wtip-min02029"></a>
#### <a name="12-live-app-form-showing-dojo-02010"></a>12. форма Live App, в которой отображается Dojo, 0:20:10
[![Форма Live App, показывающая Dojo][image-wtip-min02029-live-app-form-dojo]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1210)

&nbsp; <a name="anchor-image-wtip-min02854"></a>
#### <a name="13-myob-and-not-a-dba-in-sight-02506"></a>13. MYOB, а не администратор базы данных, 0:25:06
[![MYOB, а не администратор базы данных в аналитике][image-wtip-min02854-myob-no-dba]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1506)


&nbsp; <a name="anchor-image-wtip-min02940"></a>
#### <a name="14-myob-elastic-pool-usage-example-02930"></a>14. Пример использования эластичного пула MYOB, 0:29:30
[![Пример использования эластичного пула MYOB][image-wtip-min02940-myob-elastic]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1770)


&nbsp; <a name="anchor-image-wtip-min03136"></a>
#### <a name="15-learning-from-myob-and-other-isvs-03125"></a>15. обучение от MYOB и других независимых поставщиков программного обеспечения, 0:31:25
[![Обучение от MYOB и других независимых поставщиков программного обеспечения][image-wtip-min03136-learning-isvs]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1885)


&nbsp; <a name="anchor-image-wtip-min04315"></a>
#### <a name="16-patterns-compose-into-e2e-saas-scenario-03142"></a>16. шаблоны, составленные в сценарии SaaS E2E, 0:31:42
[![Шаблоны, созданные в сценарии SaaS E2E][image-wtip-min04315-patterns-compose]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1902)


&nbsp; <a name="anchor-image-wtip-min04733"></a>
#### <a name="17-canonical-hybrid-multi-tenant-saas-app-04604"></a>17. каноническое гибридное многоклиентское приложение SaaS, 0:46:04
[![Каноническое гибридное мультитенантное приложение SaaS][image-wtip-min04733-canonical-hybrid]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2764)


&nbsp; <a name="anchor-image-wtip-min04810"></a>
#### <a name="18-wingtip-saas-sample-app-04801"></a>18. пример приложения SaaS Wingtip, 0:48:01
[![Пример приложения Wingtip SaaS][image-wtip-min04810-wingtip-saas-app]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2881)


&nbsp; <a name="anchor-image-wtip-min04910"></a>
#### <a name="19-scenarios-and-patterns-explored-in-the-tutorials-04900"></a>19. сценарии и шаблоны, которые рассматриваются в руководствах, 0:49:00
[![Сценарии и шаблоны, которые рассматриваются в учебниках][image-wtip-min04910-scenarios-tutorials]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=2940)


&nbsp; <a name="anchor-image-wtip-min05018"></a>
#### <a name="20-demo-of-tutorials-and-github-repository-05012"></a>20. Демонстрация учебников и репозитория GitHub, 0:50:12
[![Демонстрации руководств и репозиторий GitHub][image-wtip-min05018-demo-tutorials-github]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3012)


&nbsp; <a name="anchor-image-wtip-min05038"></a>
#### <a name="21-github-repo-microsoftwingtipsaas-05032"></a>21. репозиторий GitHub Microsoft/WingtipSaaS, 0:50:32
[![Репозиторий GitHub Microsoft/WingtipSaaS][image-wtip-min05038-github-wingtipsaas]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3032)


&nbsp; <a name="anchor-image-wtip-min05620"></a>
#### <a name="22-exploring-the-patterns-05615"></a>22. Изучение шаблонов, 0:56:15
[![Изучение шаблонов][image-wtip-min05620-exploring-patterns]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3375)


&nbsp; <a name="anchor-image-wtip-min05744"></a>
#### <a name="23-provisioning-tenants-and-onboarding-05619"></a>23. подготовка клиентов и адаптации, 0:56:19
[![Подготовка клиентов и адаптации][image-wtip-min05744-provisioning-tenants-onboarding-1]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3379)


&nbsp; <a name="anchor-image-wtip-min05858"></a>
#### <a name="24-provisioning-tenants-and-application-connection-05752"></a>24. подготовка клиентов и подключение приложений, 0:57:52
[![Подготовка клиентов и подключение приложения][image-wtip-min05858-provisioning-tenants-app-connection-2]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3472)


&nbsp; <a name="anchor-image-wtip-min05943"></a>
#### <a name="25-demo-of-management-scripts-provisioning-a-single-tenant-05936"></a>25. Демонстрация сценариев управления, подготавливающих один клиент, 0:59:36
[![Демонстрация сценариев управления, подготавливающих один клиент][image-wtip-min05943-demo-management-scripts-st]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3576)


&nbsp; <a name="anchor-image-wtip-min10002"></a>
#### <a name="26-powershell-to-provision-and-catalog-05956"></a>26. PowerShell для инициализации и каталогизации, 0:59:56
[![PowerShell для инициализации и каталогизации][image-wtip-min10002-powershell-provision-catalog]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3596)


&nbsp; <a name="anchor-image-wtip-min10330"></a>
#### <a name="27-t-sql-select--from-tenantsextended-10325"></a>27. T-SQL SELECT * FROM TenantsExtended, 1:03:25
[![T-SQL SELECT * FROM TenantsExtended][image-wtip-min10330-sql-select-tenantsextended]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3805)


&nbsp; <a name="anchor-image-wtip-min10436"></a>
#### <a name="28-managing-unpredictable-tenant-workloads-10334"></a>28. Управление непредсказуемыми рабочими нагрузками клиента, 1:03:34
[![Управление непредсказуемыми рабочими нагрузками клиента][image-wtip-min10436-managing-unpredictable-workloads]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3814)


&nbsp; <a name="anchor-image-wtip-min10639"></a>
#### <a name="29-elastic-pool-monitoring-10632"></a>29. Мониторинг эластичного пула, 1:06:32
[![Мониторинг эластичного пула][image-wtip-min10639-elastic-pool-monitoring]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=3992)


&nbsp; <a name="anchor-image-wtip-min10942"></a>
#### <a name="30-load-generation-and-performance-monitoring-10937"></a>30. создание нагрузки и мониторинг производительности, 1:09:37
[![Создание нагрузки и мониторинг производительности][image-wtip-min10942-load-generation]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4117)


&nbsp; <a name="anchor-image-wtip-min11033"></a>
#### <a name="31-schema-management-at-scale-10940"></a>31. Управление схемами в масштабе, 1:09:40
[![Управление схемой в масштабе][image-wtip-min11033-schema-management-scale]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=34120)


&nbsp; <a name="anchor-image-wtip-min11221"></a>
#### <a name="32-distributed-query-across-tenant-databases-11118"></a>32. распределенный запрос между базами данных клиента, 1:11:18
[![Распределенный запрос между базами данных клиента][image-wtip-min11221-distributed-query]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4278)


&nbsp; <a name="anchor-image-wtip-min11232"></a>
#### <a name="33-demo-of-ticket-generation-11228"></a>33. Демонстрация создания билетов, 1:12:28
[![Демонстрация создания билетов][image-wtip-min11232-demo-ticket-generation]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4348)


&nbsp; <a name="anchor-image-wtip-min11246"></a>
#### <a name="34-ssms-adhoc-analytics-11235"></a>34. нерегламентированная аналитика SSMS, 1:12:35
[![Аналитика нерегламентированных сред SSMS][image-wtip-min11246-ssms-adhoc-analytics]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4355)


&nbsp; <a name="anchor-image-wtip-min11632"></a>
#### <a name="35-extract-tenant-data-into-azure-synapse-analytics-11546"></a>35. Извлечение данных клиента в Azure синапсе Analytics, 1:15:46
[![Извлечение данных клиента в Azure синапсе Analytics][image-wtip-min11632-extract-tenant-data-sql-dw]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4546)


&nbsp; <a name="anchor-image-wtip-min11648"></a>
#### <a name="36-graph-of-daily-sale-distribution-11638"></a>36. график распределения ежедневных продаж, 1:16:38
[![График распределения ежедневных продаж][image-wtip-min11648-graph-daily-sale-distribution]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4598)


&nbsp; <a name="anchor-image-wtip-min11952"></a>
#### <a name="37-wrap-up-and-call-to-action-11743"></a>37. заключение и вызов действия, 1:17:43
[![Подведение итогов и призыв к действию][image-wtip-min11952-wrap-up-call-action]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4663)


&nbsp; <a name="anchor-image-wtip-min12042"></a>
#### <a name="38-resources-for-more-information-12035"></a>38. ресурсы для получения дополнительных сведений, 1:20:35
[![Дополнительные сведения][image-wtip-min12042-resources-more-info]](https://www.youtube.com/watch?v=jjNmcKBVjrc&t=4835)

- [Запись блога, 22 мая 2017 года][resource-blog-saas-patterns-app-dev-sql-db-768h]

- *Основные понятия.* [Мультитенантные шаблоны клиента в базе данных SaaS][saas-concept-design-patterns-563e].

- *Руководства.* [SaaS-приложение Wingtip Tickets][saas-how-welcome-wingtip-app-679t].

- Репозитории GitHub для разновидностей клиентских приложений SaaS Wingtip Tickets:
    - [Репозиторий GitHub для изолированной модели приложения][github-wingtip-standaloneapp].
    - [Репозиторий GitHub для модели базы данных клиента][github-wingtip-dbpertenant].
    - [Репозиторий GitHub для модели мультитенантной базы данных][github-wingtip-multitenantdb].





## <a name="next-steps"></a>Дальнейшие действия

- [Первая статья руководства][saas-how-welcome-wingtip-app-679t]




<!-- Image link reference IDs. -->

[image-wtip-min00003-brk3120-whole-welcome]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00003-brk3120-welcome-myob-design-saas-app-sql-db.png "Слайд приветствия"

[image-wtip-min00311-session]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00311-session-objectives-takeaway.png "Цели сеанса."

[image-wtip-min00417-agenda]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00417-agenda-app-management-models-patterns.png "Дня."

[image-wtip-min00505-web-app]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00505-wingtip-saas-app-mt-web.png "Приложение SaaS Wingtip. Мультитенантное веб-приложение"

[image-wtip-min00555-app-web-form]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00555-app-form-contoso-concert-hall-night-opera.png "Форма веб-приложения в действии"

[image-wtip-min00931-per-tenant-cost]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min00931-saas-data-management-concerns.png "Стоимость, масштабирование, изоляция и восстановление каждого клиента"

[image-wtip-min01159-db-models-pros-cons]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01159-db-models-multi-tenant-saas-apps.png "Модели мультитенатной базы данных. Преимущества и недостатки"

[image-wtip-min01301-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01301-hybrib-model-blends-benefits-mt-st.png "Гибридная модель объединяет преимущества приложений с несколькими клиентами и одним клиентом"

[image-wtip-min01644-st-vs-mt]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01644-st-mt-pros-cons.png "Сравнение одного клиента и нескольких клиентов. Преимущества и недостатки"

[image-wtip-min01936-pools-cost]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min01936-pools-cost-effective-unpredictable-workloads.png "Пулы экономически эффективны при непредсказуемых рабочих нагрузках"

[image-wtip-min02008-demo-st-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02008-demo-st-hybrid.png "Демонстрация однотенантной базы данных и гибрида приложений с несколькими клиентами и одним клиентом"

[image-wtip-min02029-live-app-form-dojo]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02029-app-form-dogwwod-dojo.png "Форма приложения в реальном времени, показывающая Dojo"

[image-wtip-min02854-myob-no-dba]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02854-myob-no-dba.png "MYOB без администратора базы данных"

[image-wtip-min02940-myob-elastic]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min02940-myob-elastic-pool-usage-example.png "Пример использования эластичного пула MYOB"

[image-wtip-min03136-learning-isvs]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min03136-myob-isv-saas-patterns-design-scale.png "Обучение от MYOB и других независимых поставщиков программного обеспечения"

[image-wtip-min04315-patterns-compose]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04315-patterns-compose-into-e2e-saas-scenario-st-mt.png "Шаблоны, созданные в сценарии SaaS E2E"

[image-wtip-min04733-canonical-hybrid]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04733-canonical-hybrid-mt-saas-app.png "Каноническое гибридное мультитенантное приложение SaaS"

[image-wtip-min04810-wingtip-saas-app]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04810-saas-sample-app-descr-of-modules-links.png "Пример приложения Wingtip SaaS"

[image-wtip-min04910-scenarios-tutorials]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min04910-scenarios-patterns-explored-tutorials.png "Сценарии и шаблоны, рассмотренные в руководствах"

[image-wtip-min05018-demo-tutorials-github]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05018-demo-saas-tutorials-github-repo.png "Демонстрация учебников и репозитория GitHub"

[image-wtip-min05038-github-wingtipsaas]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05038-github-repo-wingtipsaas.png "Репозиторий GitHub Microsoft/WingtipSaaS"

[image-wtip-min05620-exploring-patterns]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05620-exploring-patterns-tutorials.png "Изучение шаблонов"

[image-wtip-min05744-provisioning-tenants-onboarding-1]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05744-provisioning-tenants-connecting-run-time-1.png "Подготовка клиентов и подключение"

[image-wtip-min05858-provisioning-tenants-app-connection-2]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05858-provisioning-tenants-connecting-run-time-2.png "Подготовка клиентов и подключение к приложению"

[image-wtip-min05943-demo-management-scripts-st]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min05943-demo-management-scripts-provisioning-st.png "Демонстрация скриптов управления, подготавливающих один клиент"

[image-wtip-min10002-powershell-provision-catalog]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10002-powershell-code.png "PowerShell, используемый для подготовки и занесения в каталог"

[image-wtip-min10330-sql-select-tenantsextended]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10330-ssms-tenantcatalog.png "T-SQL SELECT * FROM TenantsExtended"

[image-wtip-min10436-managing-unpredictable-workloads]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10436-managing-unpredictable-tenant-workloads.png "Управление непредсказуемыми рабочими нагрузками клиента"

[image-wtip-min10639-elastic-pool-monitoring]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10639-elastic-pool-monitoring.png "Мониторинг эластичного пула"

[image-wtip-min10942-load-generation]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min10942-schema-management-scale.png "Создание нагрузки и мониторинг производительности"

[image-wtip-min11033-schema-management-scale]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11033-schema-manage-1000s-dbs-one.png "Управление схемой в масштабе"

[image-wtip-min11221-distributed-query]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11221-distributed-query-all-tenants-asif-single-db.png "Распределенный запрос между базами данных клиента"

[image-wtip-min11232-demo-ticket-generation]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11232-demo-ticket-generation-distributed-query.png "Демонстрация создания билета"

[image-wtip-min11246-ssms-adhoc-analytics]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11246-tsql-adhoc-analystics-db-elastic-query.png "adhoc-аналитика SSMS"

[image-wtip-min11632-extract-tenant-data-sql-dw]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11632-extract-tenant-data-analytics-db-dw.png "Извлечение данных клиента в Azure синапсе Analytics"

[image-wtip-min11648-graph-daily-sale-distribution]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11648-graph-daily-sale-contoso-concert-hall.png "График распределения ежедневных продаж"

[image-wtip-min11952-wrap-up-call-action]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min11952-wrap-call-action-saasfeedback.png "Подведение итогов и призыв к действию"

[image-wtip-min12042-resources-more-info]: media/saas-tenancy-video-index-wingtip-brk3120-20171011/wingtip-20171011-min12042-resources-blog-github-tutorials-get-started.png "Дополнительные сведения"




<!-- Article link reference IDs. -->

[saas-concept-design-patterns-563e]: saas-tenancy-app-design-patterns.md

[saas-how-welcome-wingtip-app-679t]: saas-tenancy-welcome-wingtip-tickets-app.md


[video-on-youtube-com-478y]: https://www.youtube.com/watch?v=jjNmcKBVjrc&t=1

[video-on-channel9-479c]: https://channel9.msdn.com/Events/Ignite/Microsoft-Ignite-Orlando-2017/BRK3120


[resource-blog-saas-patterns-app-dev-sql-db-768h]: https://azure.microsoft.com/blog/saas-patterns-accelerate-saas-application-development-on-sql-database/


[github-wingtip-standaloneapp]: https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp/

[github-wingtip-dbpertenant]: https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant/

[github-wingtip-multitenantdb]: https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDB/

