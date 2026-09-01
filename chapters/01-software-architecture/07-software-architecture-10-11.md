# AI Software Architect — Software Architecture, темы 1.10–1.11

## О чём этот материал

Этот файл продолжает главу **01. Software Architecture** и фиксирует темы:

- **1.10 — Архитектурные границы и Bounded Context**
- **1.11 — Domain, Subdomain и Core Domain**

Цель блока — научиться понимать, где заканчивается одна ответственность и начинается другая, а затем связывать архитектурные границы с реальной бизнес-ценностью продукта.

---

# 1.10 — Архитектурные границы и Bounded Context

## Цель темы

До этого мы говорили:

```text
разделяй ответственности
создавай модули
контролируй зависимости
```

Но главный практический вопрос:

> Где именно провести границу?

Почему Analytics — один модуль, а Reports — другой? Почему User и Auth иногда разделяют, а иногда нет? Почему Notifications — отдельная область, а TelegramNotifier — просто реализация внутри неё?

## Boundary

**Boundary** — граница ответственности между частями системы.

Она задаёт правило:

> Всё внутри области принадлежит одной модели и ответственности, а внешние части взаимодействуют с ней через контролируемую точку.

Пример:

```text
┌─────────────────────┐
│      Analytics      │
│                     │
│ calculate_ctr       │
│ positions           │
│ traffic_change      │
│ aggregation         │
└──────────┬──────────┘
           │
        Contract
           │
           ↓
┌─────────────────────┐
│       Reports       │
└─────────────────────┘
```

Граница вокруг Analytics защищает его внутреннюю модель.

## Как понять, что функции принадлежат одной границе

Полезный вопрос:

> Они меняются по одной и той же причине?

Например:

```text
calculate_ctr()
calculate_avg_position()
calculate_traffic_change()
```

Все меняются при изменении логики SEO-аналитики.

Значит Analytics — логичная общая граница.

А:

```text
send_telegram()
```

меняется из-за Telegram API или логики доставки.

Это уже другая причина изменения — Notifications.

## Общий язык как сигнал границы

Внутри Analytics используются понятия:

```text
CTR
Impressions
Clicks
Positions
Traffic
Period
```

Notifications говорит на другом языке:

```text
Recipient
Channel
Message
Delivery
Retry
Status
```

Разный словарь часто указывает на разные модели и потенциальные границы.

## Bounded Context

Термин приходит из **Domain-Driven Design (DDD)**.

**Bounded Context** — ограниченная область системы, внутри которой термины, правила и модели имеют однозначный смысл.

Упрощённо:

```text
Bounded Context
=
граница, внутри которой действует своя модель предметной области
```

## Один объект реального мира — разные модели

Возьмём слово:

```text
User
```

В Auth:

```text
User
=
id
email
password_hash
permissions
```

В Billing:

```text
Customer
=
id
billing_address
payment_method
invoices
```

В Analytics:

```text
Account
=
site_ids
plan
limits
```

Физически это может быть один человек, но для разных областей важны разные аспекты.

## Почему один огромный User плох

```text
User
├── id
├── email
├── password
├── permissions
├── payment_card
├── invoices
├── seo_sites
├── notification_settings
├── telegram_chat_id
├── analytics_preferences
├── ai_settings
└── ...
```

Такая модель быстро превращается в God Object данных.

Bounded Context позволяет использовать более точные модели:

```text
Auth Context
→ Identity

Billing Context
→ Customer

SEO Context
→ Account / SiteOwner

Notifications Context
→ Recipient
```

## Фундаментальная идея

> Один объект реального мира не обязан иметь одну универсальную модель во всей системе.

Пример автомобиля:

Для магазина:

```text
Product
price
stock
SKU
```

Для сервиса:

```text
Vehicle
mileage
service_history
VIN
```

Для страховой:

```text
InsuredAsset
risk
owner
policy
```

Автомобиль один, модели разные.

## Bounded Context на SEO-проекте

Потенциальные области:

```text
Identity / Access
Collectors
Analytics
Rules
Reports
Notifications
AI Advisor
```

Но это не означает семь микросервисов.

Bounded Context — прежде всего логическая граница.

Он может жить:

```text
в модуле модульного монолита
```

а позже при необходимости стать отдельным сервисом.

## Logical Boundary ≠ Deployment Boundary

Можно иметь:

```text
SEO Backend
├── Analytics Context
├── Reports Context
├── Notification Context
└── Identity Context
```

в одном процессе.

Архитектурная польза уже есть, даже без сетевого разделения.

## Взаимодействие между contexts

Плохо:

```text
Reports
↓
analytics.internal_table
```

Лучше:

```text
Analytics Context
↓
Analytics Result Contract
↓
Reports Context
```

Каждый context защищает свою модель.

## Rules и Analytics

Граница здесь не такая очевидная.

```text
Analytics
→ считает факты

Rules
→ интерпретирует факты
```

Например:

```text
Analytics:
CTR = 1.3%

Rules:
CTR < 2%
→ LOW_CTR
```

У них разные ответственности, поэтому логическое разделение полезно.

Но они тесно связаны предметно, поэтому в MVP спокойно могут жить внутри общей области:

```text
seo_analysis/
├── analytics/
└── rules/
```

## Granularity — размер границ

Слишком крупно:

```text
SEO/
└── всё
```

Слишком мелко:

```text
CtrContext
PositionContext
TrafficContext
ClickContext
ImpressionContext
```

Нужен разумный уровень детализации.

## Сигналы хорошей границы

### 1. Разные причины изменения

```text
Analytics
→ формулы

Notifications
→ каналы доставки
```

### 2. Разный словарь

```text
CTR / Position / Traffic
```

против:

```text
Recipient / Delivery / Channel
```

### 3. Разные правила

Analytics:

```text
CTR = clicks / impressions
```

Notifications:

```text
retry max = 3
```

### 4. Разные внешние зависимости

```text
Collectors
→ Yandex / Google APIs

Notifications
→ Telegram / Email APIs
```

### 5. Разный lifecycle

Одни модули могут активно меняться, другие — редко.

### 6. Назначение можно объяснить одним предложением

Если нельзя — граница, возможно, слишком большая.

## Плохая граница

```text
SeoProcessing
├── collect()
├── analyze()
├── check_rules()
├── ask_ai()
├── send_notification()
└── generate_pdf()
```

Название охватывает почти всю систему и не задаёт ясного владельца ответственности.

## Хорошая граница

```text
Analytics
```

> Рассчитывает аналитические показатели на основе нормализованных SEO-данных.

```text
Notifications
```

> Доставляет сообщения пользователю через поддерживаемые каналы.

## Context Ownership

Каждый важный набор данных должен иметь понятного владельца.

Например:

```text
Analytics
```

владеет:

```text
AnalysisResult
```

Reports может использовать результат, но не должен бесконтрольно менять его внутреннюю модель.

Плохо:

```text
Reports
↓
UPDATE analytics_results
```

Лучше:

```text
Reports
↓
Analytics API
↓
Analytics изменяет свои данные
```

Это **ownership boundary**.

## Владение данными в монолите

База физически может быть одна:

```text
PostgreSQL
```

но логическое владение таблицами можно разделять:

```text
analytics_*
→ Analytics

reports_*
→ Reports

users_*
→ Identity
```

Другие модули не должны бесконтрольно изменять чужие таблицы.

## Общие данные между contexts

Если нескольким contexts нужен Site, не обязательно использовать одну огромную общую модель.

Можно передавать:

```text
SiteId
```

или небольшой контракт:

```text
SiteReference
{
  id
  domain
}
```

Каждый context использует только нужную ему часть информации.

> [!IMPORTANT]
> ## Ключевые идеи — обязательно запомнить
>
> - **Boundary** отделяет одну область ответственности от другой.
> - **Bounded Context** — область, внутри которой модель и термины имеют однозначный смысл.
> - **Один объект реального мира может иметь разные модели в разных contexts.**
> - **Bounded Context не означает микросервис.**
> - Сначала можно создавать логические границы внутри модульного монолита.
> - Хорошая граница часто определяется причиной изменения, языком, правилами и владельцем данных.
> - Каждый важный набор данных должен иметь понятного владельца.
> - Другие модули должны взаимодействовать с владельцем через контракт, а не менять его внутреннее состояние напрямую.

---

# 1.11 — Domain, Subdomain и Core Domain

## Цель темы

Понять, какую бизнес-задачу решает каждая часть системы и насколько она важна для продукта.

Если Bounded Context отвечает:

> Где проходит граница модели?

то Domain/Subdomain отвечает:

> Какую часть бизнес-проблемы мы решаем и насколько она стратегически важна?

## Domain

**Domain** — предметная область, ради которой существует система.

Для SEO-проекта:

> Автоматизированная SEO-аналитика сайтов и рекомендации по улучшению поисковой видимости.

Domain — не FastAPI, PostgreSQL, Docker или OpenAI. Это технологии реализации.

## Subdomain

Большой domain делится на более узкие части:

```text
SEO Intelligence Platform
├── Data Collection
├── SEO Analytics
├── Problem Detection
├── Reporting
├── Notifications
├── Identity
└── AI Recommendations
```

Это потенциальные **subdomains**.

Каждый subdomain решает отдельную часть общей бизнес-задачи.

## Subdomain и Bounded Context — не одно и то же

```text
Subdomain
→ часть бизнеса / проблемы

Bounded Context
→ граница модели в программной системе
```

Упрощённо:

```text
Business world
↓
Subdomains

Software model
↓
Bounded Contexts
```

Они часто близко связаны, но не обязаны совпадать 1:1.

## Три типа Subdomain

В DDD часто выделяют:

```text
Core Domain
Supporting Subdomain
Generic Subdomain
```

## 1. Core Domain

**Core Domain** — часть системы, создающая главное конкурентное преимущество продукта.

Для SEO-платформы это может быть:

```text
SEO Analysis
+
Problem Detection
+
Prioritization
```

Особенно если мы лучше конкурентов:

- объединяем данные;
- определяем реальные SEO-проблемы;
- приоритизируем их;
- даём полезные рекомендации.

Вопрос:

> Если эта часть будет посредственной, продукт всё ещё будет конкурентоспособным?

Если ответ «нет» — это сильный кандидат на Core Domain.

## На Core Domain тратят больше внимания

Например:

```text
качественнее архитектура
больше тестов
больше экспертизы
осторожнее изменения
лучше модели данных
```

Потому что именно здесь находится основная ценность продукта.

## 2. Supporting Subdomain

**Supporting Subdomain** — важная часть системы, необходимая для доставки ценности Core Domain, но сама по себе не являющаяся главным конкурентным преимуществом.

Например:

```text
Report Generation
Notifications
Data Import
```

Core:

```text
SEOProblemDetector
→ LOW_CTR
```

Supporting:

```text
Report Generator
→ превращает результат в отчёт

Notification
→ сообщает пользователю
```

Supporting помогает доставить ценность.

## 3. Generic Subdomain

**Generic Subdomain** — стандартная задача, похожим образом решаемая во многих системах.

Примеры:

```text
Authentication
Email delivery
File storage
Logging
Payment processing
```

Обычно здесь не стоит изобретать уникальную технологию без причины.

Можно использовать:

```text
готовую библиотеку
framework
external provider
```

## Build vs Buy

Для каждого subdomain полезно спрашивать:

```text
Это Core?
Supporting?
Generic?
```

А затем:

```text
Build?
Buy?
Integrate?
Use library?
```

## Пример карты SEO-платформы

```text
SEO Intelligence Platform
│
├── SEO Analytics
│   └── CORE
│
├── Rule Engine
│   └── CORE
│
├── Recommendation Prioritization
│   └── CORE
│
├── Collectors
│   └── SUPPORTING
│
├── Reports
│   └── SUPPORTING
│
├── Notifications
│   └── SUPPORTING
│
├── Authentication
│   └── GENERIC
│
├── Logging
│   └── GENERIC
│
└── File Storage
    └── GENERIC
```

Это не абсолютная классификация. Она зависит от бизнес-модели.

Если продукт продаётся как лучшая система сбора данных из сотен источников, Collectors могут стать частью Core Domain.

## Core Domain определяется бизнес-ценностью, а не сложностью технологии

LLM может быть технически сложным компонентом.

Но если мы просто вызываем готовый API, сам AI provider не обязательно является Core Domain.

Core может находиться в том, как мы:

```text
формируем контекст
выбираем проблемы
приоритизируем рекомендации
валидируем вывод AI
```

То есть в уникальной бизнес-логике вокруг модели.

## Пример с интернет-магазином

Domain:

```text
E-commerce
```

Subdomains:

```text
Catalog
Pricing
Orders
Payments
Delivery
Authentication
```

Для обычного магазина Payments может быть Generic/Supporting.

Для платёжной компании Payments уже может быть Core Domain.

Один и тот же функциональный блок имеет разную стратегическую важность в разных бизнесах.

## Почему это важно архитектору

Ресурсы ограничены.

Нельзя одинаково тщательно проектировать всё.

Если одинаково много усилий вкладывать в:

```text
SEO Rule Engine
```

и:

```text
email sender
```

можно потратить инженерные ресурсы не там, где создаётся ценность.

Архитектура должна учитывать бизнес-приоритеты.

## Core Domain и AI

Не следует автоматически считать:

```text
AI = Core Domain
```

Правильный вопрос:

> Если заменить одного LLM provider на другого, исчезнет ли наше конкурентное преимущество?

Если нет, значит provider — не Core.

Core может выглядеть так:

```text
SEO facts
↓
Problem detection
↓
Priority model
↓
Context builder
↓
AI recommendation
↓
Validation
```

Уникальной ценностью является система принятия решений, а не сам внешний AI API.

## Связь с Bounded Context

Core subdomain:

```text
SEO Analysis
```

может реализовываться внутри:

```text
SEO Analysis Context
```

Supporting:

```text
Notifications
```

в:

```text
Notification Context
```

Generic:

```text
Identity
```

может быть собственным context или внешним решением.

## Не каждый subdomain должен становиться сервисом

Важно различать уровни:

```text
Subdomain
→ бизнес-граница

Bounded Context
→ граница модели

Module
→ граница кода

Service
→ deployment/runtime граница
```

Они могут совпадать, но это не обязательное правило.

MVP всё ещё может быть:

```text
SEO Backend
└── Modular Monolith
    ├── analytics/
    ├── rules/
    ├── collectors/
    ├── reports/
    ├── notifications/
    └── identity/
```

## Как определить Core Domain

### 1. За что пользователь реально платит?

Не за JWT-токен, а, например, за понимание, почему сайт не растёт.

### 2. Что отличает продукт от конкурентов?

Например:

```text
точность диагностики
приоритизация проблем
качество рекомендаций
```

### 3. Где находится сложное бизнес-знание?

Много:

```text
правил
исключений
эвристик
экспертных решений
```

— сильный кандидат на Core.

### 4. Что нельзя легко заменить готовым SaaS без потери ценности?

Если можно заменить одной библиотекой без потери ценности, скорее всего это не Core.

## Пример развития Core Domain

Примитивно:

```text
CTR < 2%
→ LOW_CTR
```

Со временем:

```text
CTR expected by position
+
device
+
region
+
query intent
+
seasonality
+
historical trend
```

И проблема определяется сравнением:

```text
actual CTR
vs
expected CTR
```

Это уже накопленное уникальное бизнес-знание.

## Начало Domain Model

По мере роста появляются понятия:

```text
SearchPerformance
ExpectedCTR
RankingTrend
SEOIssue
IssueSeverity
Recommendation
Confidence
```

Это элементы будущей **Domain Model**.

> [!IMPORTANT]
> ## Ключевые идеи — обязательно запомнить
>
> - **Domain** — предметная область, ради которой существует продукт.
> - **Subdomain** — отдельная часть бизнес-задачи.
> - **Core Domain** — главное конкурентное преимущество продукта.
> - **Supporting Subdomain** — помогает доставить ценность Core Domain.
> - **Generic Subdomain** — стандартная задача, которую часто разумно решать готовыми инструментами.
> - **Core Domain определяется бизнес-ценностью, а не технической сложностью.**
> - Не каждый subdomain должен становиться отдельным сервисом.
> - Архитектурные усилия нужно концентрировать прежде всего вокруг Core Domain.
> - Для Generic Subdomains часто разумнее использовать готовые решения, чем строить собственные.

---

# Итог тем 1.10–1.11

```text
Boundary
↓
отделяет ответственность

Bounded Context
↓
задаёт границу модели и языка

Subdomain
↓
описывает часть бизнес-проблемы

Core Domain
↓
показывает, где находится ключевая ценность продукта
```

Следующая тема курса: **1.12 — Domain Model, Entity, Value Object и Aggregate**.
