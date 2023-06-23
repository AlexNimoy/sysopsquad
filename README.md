# Архитектурное ката Sysops Squad, автор O'Reilly, апрель - май 2021г

## Команда семь

**Pavel, Suheyl, Nikita, Hassan**

## Содержание

<img align="right" width="210" height="368" src="images/badge.png">

- [Добро пожаловать](#добро-пожаловать)
  - [О названии](#о-названии)
- [Бизнес-кейс](#бизнес-кейс)
- [Системные требования](#системные-требования)
  - [Функциональные требования](#функциональные-требования)
  - [Архитектурные требования](#архитектурные-требования)
  - [Ограничения](#ограничения)
  - [Допущения](#допущения)
- [Текущая архитектура](#текущая-архитектура)
- [Целевая архитектура](#целевая-архитектура)
  - [Use Case Model](#use-case-model)
  - [System Context](#system-context)
  - [Containers](#containers)
  - [Process Views](#process-views)
  - [Deployment](#deployment)
- [Transition Architecture](#transition-architecture)
- [Architecture Decision Records](#architecture-decision-records)

## Добро пожаловать

> _Все в архитектуре программного обеспечения является компромиссом.  
> Первый закон архитектуры программного обеспечения_

Добро пожаловать на Архитектурный ката Sysops Squad, проводимый компанией O'Reilly в апреле - мае 2021 года.

Эта страница представляет собой архитектурную документацию для предложения решения от **Команды Семь**.

### О названии

<img align="right" width="200" height="183" src="images/jersey.jpg">
Magic number Seven... This is not a random number in our name. This number joined our team together. One of us has a birthday at 07/07 thus joined the Team Seven. Somebody had a successful career in a football team under number 7 thus joined the Team Seven. Somebody believes that this is his lucky number throughout the whole life.  
And of course, the average number of services in Service-based architecture that we defined as our initial candidate, is about 7.
That's why we decided to make this number a symbol of our team.

## Бизнес-кейс

Penultimate Electronics - это крупный электронный гигант, у которого имеется множество розничных магазинов по всей стране. Когда клиенты покупают компьютеры, телевизоры, стереосистемы и другую электронную технику, они могут выбрать план поддержки. Технические эксперты, работающие с клиентами («Sysops Squad»), затем приезжают на дом клиента (или в офис) для устранения проблем с электронным устройством.

Существующая система управления заявками представляет собой большое монолитное приложение, разработанное много лет назад. Клиенты жалуются, что консультанты никогда не приходят из-за потерянных заявок, а часто приходит неправильный консультант, который не разбирается в проблеме. Клиенты и сотрудники колл-центра жалуются, что система не всегда доступна для ввода заявок через веб или звонки. Внесение изменений в этот большой монолит является сложным и рискованным процессом - каждое изменение занимает слишком много времени, а часто что-то ломается. Из-за проблем с надежностью монолитная система часто "зависает" или выходит из строя - это обычно происходит из-за резкого увеличения использования и числа клиентов, пользующихся системой. Если что-то не будет сделано в ближайшее время, Penultimate Electronics будет вынуждена отказаться от этой очень прибыльной линии бизнеса и уволить всех экспертов.

Желаемое решение с функциональной точки зрения представлено на следующей диаграмме marketecture.

![Marketecture](images/marketecture.jpg)

**Бизнес-факторы**

Какие бизнес-факторы мы можем выявить из данной ситуации:

- Низкая производительность системы отслеживания заявок.
- Возможность полного прекращения данной линии бизнеса.
- Многие люди находятся под угрозой потери своих рабочих мест.

**Бизнес-цели**

Компания устанавливает следующую бизнес-цель для улучшения ситуации:

- Разработать новую систему отслеживания заявок, которая будет удовлетворять требуемым характеристикам.

Компания страдает от неэффективной системы поддержки клиентов, что может привести к прекращению данной линии бизнеса. Они хотят разработать новую надежную и высокопроизводительную систему, которая позволит им остаться в бизнесе и обеспечить будущий рост.

## Системные требования

### Заинтересованные стороны (Stakeholders)

В этом разделе описываются ключевые заинтересованные стороны системы и их архитектурные проблемы.

- **SH-1**: **Администратор (Administrator)** (security)

  - безопасность - это второе имя администратора; эти люди занимаются учетными записями пользователей и системой выставления счетов.

- **SH-2**: **Клиент (Customer)** (availability, performance, scalability, robustness)

  - клиенты хотят, чтобы система, к которой они обращаются, была доступна в любое время, когда им нужно ее использовать, и чтобы она быстро реагировала на их действия;
  - они также не хотят, чтобы их заявки обрабатывались и не терялись.

- **SH-3**: **Эксперт (Expert)** (availability, performance)

  - каждый раз, когда эксперт работает на месте, успех разрешения проблемы может зависеть от того, имеет ли он доступ к базе знаний системы заявок;
  - неэффективная система может существенно влиять на разрешение проблемы и тратить время клиента и эксперта.

- **SH-4**: **Менеджер (Manager)** (reportability)

  - данным людям необходимо тщательно следить за происходящим: есть ли недовольные клиенты, заявки, для которых слишком долго не назначается эксперт, возникли ли какие-либо проблемы с выставлением счетов и т.д.

- **SH-5**: **Служба технической поддержки (Helpdesk)** (availability, performance)

  - это первая линия поддержки для клиентов, поэтому им необходимо иметь доступ к статьям по устранению неполадок и заявкам клиентов во время звонков;
  - они предоставляют прямую телефонную поддержку, поэтому ответы должны быть найдены как можно быстрее.

- **SH-6**: **Команда разработки (Development team)** (extensibility)
  - у этих специалистов возникают сложности с развертыванием изменений в продакшн-среде из-за высокой связности и недостаточной модульности текущей системы.

### Функциональные требования

- **UC-1**: **Управление пользователями**:

  - Администратор поддерживает внутренние учетные записи пользователей (SH-1);
  - Администратор поддерживает навыки, местоположение и доступность экспертов (SH-1);

- **UC-2**: **Регистрация клиента**:

  - Клиенты регистрируют свои профили, кредитные карты и планы поддержки (SH-2);

- **UC-3**: **Рабочий процесс заявок**:

  - Клиенты отправляют заявки через веб-интерфейс или по телефону (SH-2, SH-5);
  - Эксперты используют мобильное приложение для чтения заявок и изменения их статуса (SH-3);
  - Эксперты могут использовать мобильное приложение для поиска в базе знаний (SH-3);

- **UC-4**: **Отправка опросов**:

  - Клиенты заполняют и отправляют опросы о удовлетворенности (SH-2);

- **UC-5**: **Обновление базы знаний**:

  - Эксперты обновляют базу знаний (SH-3);

- **UC-6**: **Формирование отчетов**:

  - Менеджеры отслеживают операции с заявками (SH-4);
  - Менеджеры создают отчеты: финансовые, о производительности экспертов, о заявках (SH-4);

- **UC-7**: **Выставление счетов**:

  - Клиентам автоматически выставляются счета ежемесячно (SH-2);
  - Клиенты могут просматривать историю и выписки по счетам (SH-2);
  - Администратор управляет процессом выставления счетов для клиентов (SH-1);

- **UC-8**: **Уведомления**:

  - Клиенты получают SMS или электронное письмо о назначении эксперта (SH-2);
  - Клиенты получают электронное письмо с ссылкой на веб-форму опроса (SH-2);
  - эксперты получают SMS-уведомление о назначении заявки (SH-3);

- **UC-9**: **Поиск заявок**:
  - Сотрудникам службы технической поддержки необходим доступ к базе заявок для уточнения их статуса (SH-5);

### Архитектурные требования

- **QA-1**: **доступность (scalability)** (UC-3)

  - География национального масштаба (США?);
  - Количество клиентов - миллионы;
    Количество заявок на клиента <= 100 (допустим, что-то экстремальное);

- **QA-2**: **доступность (availability)** (UC-2, UC-3, UC-4)

  - Клиентские сервисы и база знаний должны быть высокодоступными, поскольку сбои окажут негативное влияние на бизнес;
  - 99,9% кажется разумным в данном случае;

- **QA-3**: **производительность (performance)** (UC-2, UC-3, UC-6)

  - Время отклика < 2 секунд для загрузки страницы;
  - Время поиска в базе знаний несколько секунд;
  - Генерация отчетов не должна занимать чрезмерное количество времени;

- **QA-4**: **надежность (robustness)** (UC-3)

  - Потерянные заявки или неправильно назначенные эксперты могут привести к закрытию бизнеса;

- **QA-5**: **безопасность (security)** (UC-2, UC-7)

  - Личная информация клиентов и данные кредитных карт должны храниться в безопасном и соответствующем требованиям PCI формате;

- **QA-6**: **расширяемость (extensibility)** (all use cases, SH-6)
  - Одна из ключевых проблем текущей системы заключается в том, что внесение изменений занимает слишком много времени, а что-то обычно ломается. Это заставляет задуматься о улучшенной модульности новой системы.

### Ограничения

- **CON-1**: Интеграция? Облако/локальное развертывание? Стек технологий?

### Допущения

- **ASM-1**: Мобильное приложение является частью системы и может быть изменено.
- **ASM-2**: Помощники службы поддержки (также известные как колл-центр) нуждаются в доступе к подсистеме заявок и некоторой информации о клиентах (контакты, возможно, информация о поддержке). Они также являются пользователями системы, хотя и не указаны в "Основных четырех пользователях" изначальных требований.
- **ASM-3**: Система отслеживания вызовов (call tracking system) не является частью системы Sysops Squad.
- **ASM-4**: Компания хранит информацию о кредитных картах клиентов локально и не взаимодействует с третьей стороной (например, authorize.net в США).

## Текущая архитектура

В данном разделе описывается архитектура текущей системы заявок.

Обратите внимание, что все представления документируются в стиле [C4 model](https://c4model.com), хотя представлены только системная контекстная, контейнерная и динамическая представления. Большинство диаграмм используют неформальный стиль обозначений. Все диаграммы сопровождаются легендой, объясняющей значение каждой формы на диаграмме.

Текущая система заявок проявляет очень слабые характеристики доступности, поддерживаемости, развертываемости и производительности. Нашей целью является разработка новой системы, которая решает вышеупомянутые проблемы.

Далее изображена диаграмма контейнеров текущей системы заявок:

![Текущая архитектура](images/baseline.jpg 'Текущая архитектура')

## Целевая архитектура

В данном разделе описывается целевая архитектура программного обеспечения.

Обратите внимание, что все представления документируются в стиле [C4 model](https://c4model.com), хотя представлены только системная контекстная, контейнерная и динамическая представления. Большинство диаграмм используют неформальный стиль обозначений. Все диаграммы сопровождаются ключом, объясняющим значение каждой формы на диаграмме.

### Use Case Model

The following diagram shows mapping of architecture characteristics requirements on the key use cases based on discovered [requirements](Requirements.md):

![Use Case Model](images/use-case-model.jpg 'Use Case Model')

### System Context

The system context diagram below depicted key users of the system and its external dependencies:

![System Context](images/system-context.jpg 'System Context')

### Containers

The containers diagram that follows shows the high-level shape of the software architecture and how responsibilities are distributed across containers. It also shows the major technology choices and how the containers communicate with one another.

The architecture is build around four main domains that have been discovered during the problem analysis:

- customer-facing services, such as ticket submission, customer profiles, survey submission etc;
- expert services, such as ticket acceptance and knowledge base search;
- administration services, such as reporting, survey analysis, ticket tracking etc;
- billing service, which require high attention to security.

The architectural style used here as the bases is Service-based architecture (see [ADR-1](ADR/ADR-1-service-based.md) for details).

![Containers](images/containers.jpg 'Containers')

### Process Views

This section explains some key use cases to demonstrate how corresponding workflows pass through containers.

#### UC-2: Customer registration

The following sequence diagram highlights some key requests that the customer performs during registration in the system.
One worth paying attention is registration of a credit card. In the customer database we store only some minimal credit card data to let the customer possibility identify which card do they have already registered. All the details of the credit card are encrypted and securely passed to the billing system (see [ADR-4](ADR/ADR-4-extract-billing-quanta.md)).

![UC-2: Customer registration](images/customer-registration.jpg 'Customer Registration')

#### UC-3: Ticket submission

The following diagram illustrates the process of a ticket registration by the customer.

![UC-3: Ticket submission](images/ticket-submission.jpg 'Ticket Submission')

Important thing to note is that the requests succeeds after the ticket is saved in the customer database and the corresponding event is fired for the ticket processing area. This way the customer will be able to see the new ticket immediately after the page refresh and will not have to wait on any further actions on the ticket.

#### UC-3: Ticket assignment

The diagram below explains how the system processes a new ticket and assigns it an expert.

![UC-3: Ticket Created](images/ticket-assignment.jpg)

Since Ticket Process is a job that runs periodically, tickets that cannot be assigned at the given moment will never be lost, they we bill processed next time the job will run.

Also, notice that an assignment is a separate entity. This way we can store a history of assignments.

#### UC-3: Ticket acceptance

This diagram continues the ticket workflow and shows how the Ticket Assigned event is processed by the Sysops Expert user.

![UC-3: Ticket Assigned](images/ticket-acceptance.jpg)

The experts operation succeeds as soon as the ticket status is saved in the database. And in case of acceptance the corresponding even is fired to the customer area.

#### UC-3: Ticket in-progress

This diagram demonstrates how the customer is notified when the Sysops Expert accepted the ticket.

![UC-3: Ticket In-Progress](images/ticket-inprogress.jpg)

Important to notice that the ticket is saved in the customer database prior to the notification event so that the customer will see the actual ticket status upon the notification receive.

#### UC-3: Ticket completion

This diagram explains the process when the Sysops Expert solved the problem and marks the ticket as completed.

![UC-3: Ticket Completed](images/ticket-completion.jpg)

#### UC-3: Ticket Resolved

This diagram illustrates how the customer receives a notification about the ticket resolution and link to the survey form.

![UC-3: Ticket Resolved](images/ticket-resolved.jpg)

First, the ticket status has to be updated in the customer database, so that upon receiving any notifications the customer will see the actual ticket status on the Customer Portal.

#### UC-4: Survey Submission

And finally the last step in the ticket resolution flow is survey submission by the customer.

![UC-4: Survey Submission](images/survey-submission.jpg)

From the customer perspective this is a fire-and-forget even so the operation succeeds as soon as the "Submit" button is clicked.

Analytics API can perform some preliminary processing of the survey if necessary or simply store it in the database for the reporting.

#### UC-7: Monthly billing

The diagram illustrates the monthly billing workflow.

![UC-7: Monthly billing](images/billing-sequence.jpg 'Monthly Billing')

### Deployment

The deployment diagram illustrates how the system containers are mapped to the infrastructure:

![Deployment](images/deployment.jpg 'Deployment')
Note the colors have not special meaning, they are just to distinguish thing from one another.

The deployment strategy here is cloud-agnostic, assuming you can use any cloud provider of your choice or stay totally on-prem. An exception is the billing stuff, which is recommended to remain on-prem anyway for security considerations.

## Transition Architecture

The solution proposed in the [Target Architecture](#containers) section is the final ambition that solves most of the problems and risks, but can require significant development efforts because of the database split required. Thus we can divide the whole work into two phases:

1. Solve critical problems an stick with a monolithic database until it becomes a bottleneck.
2. Migrate further to the target architecture to deal with all remaining risks.

Here is the transitional architecture proposal that solves critical problems but leaves some risks (analysis follows).
Note that we still leverage asynchronous messaging for ticket processing here to enable independent scalability and availability for different parts of the system. In this case, messages can contain mach less information because all the details can be taken by the receiver from the database.

![Transition Architecture](images/transition.jpg)

Since we have a single monolithic database we can save some efforts on additional messaging and replication.

### Risk Analysis

These are the possible high risks of the transition architecture.

#### Performance

Because this is a monolithic database it can become a performance bottleneck. The same concern regarding the single API Gateway - if not scaled properly may also become a bottleneck.

#### Availability

A single API Gateway may introduce a single point of failure for the whole system (see [ADR-12](ADR/ADR-12-gateways.md)).

#### Security

There is a risk that admin staff can get access to the customer credit card data. We certainly want to prevent that by extracting billing into a separate architectural quantum (see [ADR-4](ADR/ADR-4-extract-billing-quanta.md)) and isolating it in a separate network zone with strict access permissions.

The same concern is regarding the customer services - we don't want to allow an attacker to get access to the reset of the system. A significant security improvement would be to migrate customer services and data in a separate quantum in isolate it in a separate network zone (see [ADR-5](ADR/ADR-5-extract-customer-quantum.md)).

#### Other

Additional concerns regarding the API Gateway:

- Adds coupling between the gateway and the internal service.
- If developed by a single development team, may become a development bottleneck.

## Architecture Decision Records

> _Why is more important than how.  
> Second Law of Software Architecture_

- [ADR-1](ADR/ADR-1-service-based.md) Use Service-based architectural style as the basic style.
- [ADR-2](ADR/ADR-2-event-driven-broker.md) Use message queues with guaranteed delivery for ticket workflow.
- [ADR-3](ADR/ADR-3-search-expert.md) Extract ticket assignment into a separate batch processing job.
- [ADR-4](ADR/ADR-4-extract-billing-quanta.md) Extract billing architectural quantum.
- [ADR-5](ADR/ADR-5-extract-customer-quantum.md) Extract customer architectural quantum.
- [ADR-6](ADR/ADR-6-separate-customer-db.md) Use separate customer database.
- [ADR-7](ADR/ADR-7-separate-reporting-db.md) Separate analytics and reporting database.
- [ADR-9](ADR/ADR-9-notification-service.md) Extract notification service.
- [ADR-10](ADR/ADR-10-modular-services.md) Use sub-domain partitioning for service design.
- [ADR-11](ADR/ADR-11-extract-payment-job.md) Extract payment processing into a separate component (Payment Job).
- [ADR-12](ADR/ADR-12-gateways.md) Offload operational concerns into API Gateways.
