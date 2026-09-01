# AI Software Architect — Software Architecture, темы 1.7–1.9

## О чём этот материал

Этот файл продолжает главу **01. Software Architecture** и фиксирует темы:

- **1.7 — Interface, Contract и Dependency Inversion на практике**
- **1.8 — SOLID без зубрёжки**
- **1.9 — Архитектурные anti-patterns**

Главная цель блока — научиться проектировать устойчивые границы между частями системы, правильно использовать абстракции и распознавать архитектурные ловушки до того, как они превратятся в системную проблему.

---

# 1.7 — Interface, Contract и Dependency Inversion на практике

## Цель темы

Связать воедино:

- Contracts;
- Encapsulation;
- Low Coupling;
- Dependency Direction;
- Dependency Inversion;
- Dependency Injection.

## Interface

**Interface** — способ взаимодействия с компонентом без знания его внутренней реализации.

Он отвечает на вопрос:

> Какие операции доступны?

Например, Analytics может предоставлять:

```text
analyze(site_data)
```

А внутри использовать:

```text
calculate_ctr()
calculate_position_change()
calculate_conversion()
aggregate_periods()
```

Внешнему модулю знать это не нужно.

```text
Report
↓
Analytics Interface
↓
Analytics Internals
```

## Interface и Contract

Эти понятия близки, но не идентичны.

**Interface** отвечает:

> Какие операции доступны?

**Contract** отвечает шире:

> Что можно передать, что будет возвращено и какое поведение гарантируется?

Пример:

```text
Input:
{
  clicks,
  impressions,
  positions
}

Output:
{
  ctr,
  avg_position,
  traffic_change
}
```

И часть поведения:

```text
если impressions = 0
→ ctr = null
```

Упрощённо:

```text
Interface
= как обратиться

Contract
= формат + правила + обещанное поведение
```

## Единый интерфейс для Collector

```text
YandexCollector
GoogleCollector
AhrefsCollector
      ↓
Collector Interface
      ↓
Analytics
```

Все реализации могут предоставлять:

```text
collect(site_id, period)
```

Analytics работает с единым внутренним контрактом и не обязан знать специфику внешнего поставщика.

## Dependency Inversion

Плохо:

```text
Analytics
↓
YandexCollector
```

Теперь важная логика привязана к конкретной реализации.

Лучше:

```text
Analytics
↓
Collector Interface
↑
YandexCollector
↑
GoogleCollector
↑
AhrefsCollector
```

Высокоуровневая логика зависит от контракта, а конкретные технические реализации подстраиваются под него.

## Почему это называется inversion

Интуитивно кажется:

```text
Analytics использует Yandex
→ Analytics зависит от Yandex
```

Но архитектурно можно перевернуть направление знания:

```text
Analytics определяет, какие данные ему нужны
↓
Collector Contract
↑
Yandex адаптируется к этому контракту
```

Техническая деталь становится реализацией потребности важной логики.

## Пример в коде

Жёсткая зависимость:

```python
def analyze_site():
    yandex = YandexAPI(...)
    data = yandex.fetch(...)
```

Более гибко:

```python
def analyze_site(collector):
    data = collector.collect(...)
```

Теперь можно передать:

```text
YandexCollector
GoogleCollector
FakeCollector
```

## Тестируемость

При жёсткой зависимости:

```text
Analytics
↓
Yandex API
```

тест зависит от:

- интернета;
- токена;
- доступности API;
- лимитов;
- реальных данных.

Через интерфейс:

```text
Analytics
↓
Collector Interface
↑
FakeCollector
```

тест может использовать заранее подготовленные данные и проверять только Analytics.

## Dependency Injection

**Dependency Inversion** — архитектурный принцип.

**Dependency Injection** — техника передачи зависимости извне.

Плохо:

```text
Analytics сам создаёт YandexCollector
```

Лучше:

```text
Analytics(collector=yandex_collector)
```

То есть зависимость передаётся объекту снаружи.

## Различие терминов

```text
Dependency
→ одна часть использует другую

Dependency Inversion
→ важная логика зависит от абстракции, а не детали

Dependency Injection
→ зависимость передаётся извне
```

## Пример с хранилищем

```text
Rule Engine
↓
MetricsRepository Interface
↑
PostgresMetricsRepository
```

Позже другая реализация может соответствовать тому же контракту.

Но абстракция не означает, что разные базы данных полностью взаимозаменяемы: модели данных, запросы, консистентность и производительность могут существенно отличаться.

## Где интерфейсы особенно полезны

В реальных точках вариативности:

```text
Data Sources
LLM Providers
Notification Channels
Storage Implementations
Payment Providers
External APIs
```

Например:

```text
Collector
├── YandexCollector
├── GoogleCollector
└── AhrefsCollector
```

```text
AIProvider
├── OpenAIProvider
├── ClaudeProvider
└── OllamaProvider
```

```text
Notifier
├── TelegramNotifier
├── EmailNotifier
└── SlackNotifier
```

## Когда интерфейс лишний

Не нужно механически абстрагировать каждую функцию.

Например, создание:

```text
CtrCalculatorInterface
↓
DefaultCtrCalculator
```

ради единственной простой формулы может не давать никакой реальной пользы.

> Абстракция должна защищать реальную точку изменений, а не существовать ради архитектурной красоты.

## Composition Root

**Composition Root** — место, где конкретные реализации создаются и связываются между собой.

Например:

```text
main.py
```

может:

```text
создать YandexCollector
создать PostgresRepository
создать OpenAIProvider
↓
передать их нужным компонентам
↓
запустить приложение
```

Бизнес-модули не должны заниматься созданием всей инфраструктуры сами.

> [!IMPORTANT]
> ## Ключевые идеи — обязательно запомнить
>
> - **Interface показывает доступные операции.**
> - **Contract определяет ещё и формат данных, правила и обещанное поведение.**
> - **Dependency Inversion защищает важную логику от конкретных технических деталей.**
> - **Dependency Injection — способ передать зависимость извне.**
> - **Интерфейсы особенно полезны в реальных точках вариативности.**
> - **Не нужно создавать абстракции для всего подряд.**
> - **Composition Root — место, где конкретные реализации связываются между собой.**

---

# 1.8 — SOLID без зубрёжки

## Цель темы

Понять пять принципов SOLID как продолжение уже изученных идей, а не как набор букв для механического заучивания.

```text
S — Single Responsibility Principle
O — Open/Closed Principle
L — Liskov Substitution Principle
I — Interface Segregation Principle
D — Dependency Inversion Principle
```

## S — Single Responsibility Principle

У части системы должна быть одна основная причина для изменения.

Плохо:

```text
Analytics
├── calculate_ctr()
├── send_telegram()
├── generate_pdf()
└── authenticate_user()
```

Лучше:

```text
Analytics
→ расчёты

Notifications
→ уведомления

Reports
→ отчёты

Auth
→ идентификация пользователя
```

Связь с предыдущими темами:

```text
SRP
→ High Cohesion
```

## O — Open/Closed Principle

Идея:

> Новое поведение желательно добавлять расширением, а не постоянным переписыванием уже работающего кода.

Плохо:

```python
if source == "yandex":
    ...
elif source == "google":
    ...
elif source == "ahrefs":
    ...
```

Лучше:

```text
Collector Interface
├── YandexCollector
├── GoogleCollector
└── AhrefsCollector
```

Новая реализация может добавляться без переписывания существующих реализаций.

Но не нужно строить plugin-framework заранее, если реальной вариативности пока нет.

## L — Liskov Substitution Principle

Если реализации обещают соответствовать одному контракту, они должны быть взаимозаменяемы в рамках ожидаемого поведения.

```text
Collector
├── YandexCollector
├── GoogleCollector
└── FakeCollector
```

Все обещают:

```text
collect(site_id, period)
```

Плохо, если одна реализация возвращает нормализованные данные, а другая внезапно возвращает HTML-строку.

Формально метод существует, но контракт поведения нарушен.

## I — Interface Segregation Principle

Клиент не должен зависеть от методов, которые ему не нужны.

Плохо:

```text
SEOService
├── collect()
├── analyze()
├── generate_pdf()
├── send_telegram()
├── delete_user()
└── process_payment()
```

Лучше:

```text
Collector
→ collect()

Analyzer
→ analyze()

ReportGenerator
→ generate()

Notifier
→ send()
```

Маленькие осмысленные интерфейсы проще понимать и тестировать.

Но не нужно превращать каждую функцию в отдельный interface.

## D — Dependency Inversion Principle

Высокоуровневая логика не должна без необходимости зависеть напрямую от конкретных low-level деталей.

Плохо:

```text
Analytics
↓
YandexAPI
```

Лучше:

```text
Analytics
↓
Collector Contract
↑
YandexCollector
```

Или:

```text
Rules
↓
MetricsRepository
↑
PostgresMetricsRepository
```

## Связь SOLID с уже изученным

```text
SRP
↓
High Cohesion

OCP
↓
Extensibility

LSP
↓
Reliable Contracts

ISP
↓
Small Interfaces / Lower Coupling

DIP
↓
Controlled Dependency Direction
```

SOLID — формальная упаковка многих идей, которые уже были разобраны раньше.

## Где SOLID полезен

Когда система:

- растёт;
- имеет несколько реализаций;
- часто меняется;
- пишется командой;
- должна хорошо тестироваться;
- интегрируется с внешними API.

## Где SOLID может навредить

Если применять механически.

Например:

```text
ICtrCalculator
↓
DefaultCtrCalculator
↓
CtrCalculatorFactory
↓
CtrCalculatorProvider
```

ради простой формулы CTR — это уже overengineering.

> SOLID должен снижать стоимость изменений, а не просто увеличивать количество файлов.

> [!IMPORTANT]
> ## Ключевые идеи — обязательно запомнить
>
> - **S** — одна основная причина для изменения.
> - **O** — новое поведение лучше добавлять расширением, а не постоянным переписыванием старого кода.
> - **L** — реализации одного контракта должны сохранять обещанное поведение.
> - **I** — модуль не должен зависеть от операций, которые ему не нужны.
> - **D** — важная логика зависит от абстракций, а не от конкретных технических деталей.
> - **SOLID — инструмент, а не самоцель.**

---

# 1.9 — Архитектурные anti-patterns

## Цель темы

Научиться узнавать распространённые формы плохой архитектуры в реальном проекте.

## Anti-pattern

**Anti-pattern** — распространённый подход, который сначала кажется удобным, но со временем создаёт системные проблемы.

## 1. God Object / God Class

```text
SeoManager
├── fetch_yandex()
├── fetch_google()
├── calculate_ctr()
├── calculate_positions()
├── check_rules()
├── call_llm()
├── save_database()
├── generate_pdf()
├── send_telegram()
└── authenticate_user()
```

Один объект знает слишком много и имеет слишком много причин для изменения.

```text
Low Cohesion
+
High Coupling
```

Исправление — разделение по ответственностям.

## 2. Spaghetti Code

Хаотичные связи, при которых сложно понять последствия изменения.

```text
A → B
↑   ↓
D ← C
↘ ↑
  E
```

Часто начинается с серии локальных «временных» исправлений.

## 3. Circular Dependency

```text
Analytics
↓
Reports
↓
Notifications
↓
Analytics
```

Циклические структурные зависимости усложняют понимание, тестирование и рефакторинг.

При этом runtime feedback-loop через события сам по себе не обязательно является плохим циклом: важно отличать поток событий от жёсткой кодовой зависимости.

## 4. Shared Database Anti-pattern

```text
Analytics Service
Report Service
Notification Service
       ↓
   same_database
```

Если сервисы напрямую знают внутренние таблицы друг друга, физическое разделение не даёт архитектурной независимости.

Изменение внутренней схемы одного сервиса может сломать другие.

## 5. Utils Hell

Сначала:

```text
utils/
├── date.py
└── uuid.py
```

Позже:

```text
utils/
├── seo.py
├── auth.py
├── payment.py
├── reports.py
├── notifications.py
├── ai.py
└── parser.py
```

Это часто означает, что у логики нет ясного владельца.

Вопрос:

> Эта функция действительно generic или она принадлежит конкретному домену?

## 6. Big Ball of Mud

Система, где архитектурные границы практически исчезли.

Симптомы:

- всё зависит от всего;
- непонятны владельцы ответственности;
- глобальное состояние;
- shared DB;
- shared utils;
- циклические зависимости;
- страшно удалять код;
- сложно предсказать последствия изменений.

## 7. Tight Coupling to Framework

Бизнес-логика начинает жить прямо внутри HTTP-framework слоя.

Плохо:

```python
@router.get(...)
def calculate_ctr(...):
    # вся бизнес-логика здесь
```

Лучше:

```text
HTTP Layer
↓
Application / Domain Logic
```

Framework остаётся оболочкой, а не владельцем бизнес-смысла.

## 8. Feature Envy

Один модуль слишком сильно интересуется внутренностями другого.

Например Report постоянно использует:

```text
analytics.raw_data
analytics.internal_metrics
analytics.private_config
analytics.cache
```

Это сигнал, что часть логики, возможно, находится не у своего владельца.

## 9. Shotgun Surgery

Одно небольшое изменение требует правок во множестве мест.

```text
добавили поле device_type
↓
Collector
Analytics
Rules
Report
Notifications
Database
AI Prompt
API
Frontend
```

Иногда сквозное изменение оправдано, но если так происходит почти всегда, coupling слишком высокий.

Ключевое понятие:

**Locality of Change** — изменение должно оставаться максимально локальным.

## 10. Premature Abstraction

```text
ISeoAnalyzerFactoryProvider
↓
SeoAnalyzerFactoryProviderImpl
↓
AbstractSeoAnalyzerBuilder
↓
DefaultSeoAnalyzerBuilder
```

ради простой операции — признак абстракции под проблему, которой ещё нет.

Связано с YAGNI.

## 11. Premature Microservices

```text
Auth Service
User Service
Analytics Service
Report Service
Rules Service
AI Service
Collector Service
Notification Service
```

для нескольких пользователей могут создать больше инфраструктурной работы, чем продуктовой пользы.

## Две крайности плохой архитектуры

```text
хаос
←────────────→
overengineering
```

Цель — не максимум структуры, а минимальная структура, достаточная для контроля сложности.

## Как AI усиливает anti-patterns

AI очень быстро выполняет локальные изменения.

Если постоянно просить:

```text
добавь это сюда
вызови сервис прямо отсюда
почини только этот файл
```

каждый отдельный commit может работать, но общая система постепенно превратится в Big Ball of Mud.

Перед изменением нужно проверять:

```text
Где владелец логики?
Какая граница?
Какой контракт?
Какая новая зависимость появляется?
```

> [!IMPORTANT]
> ## Ключевые идеи — обязательно запомнить
>
> - **God Object** — одна часть знает и делает слишком много.
> - **Spaghetti Code** — связи хаотичны и плохо предсказуемы.
> - **Circular Dependency** — кодовые модули структурно зависят друг от друга по кругу.
> - **Shared Database** может создать high coupling даже между отдельными сервисами.
> - **Utils Hell** часто означает отсутствие ясного владельца логики.
> - **Big Ball of Mud** — система, где архитектурные границы практически исчезли.
> - **Shotgun Surgery** — небольшое изменение требует правок во множестве мест.
> - **Premature Abstraction / Microservices** — архитектура под проблемы, которых ещё нет.
> - **Locality of Change** — один из главных показателей качества архитектуры.

## Архитектурный чеклист перед изменением

1. Кто владеет этой логикой?
2. Это новая ответственность или часть существующей?
3. Не создаём ли high coupling?
4. Не нарушаем ли encapsulation?
5. Не появляется ли цикл зависимостей?
6. Не лезем ли напрямую в чужие внутренности или таблицы?
7. Нужна ли новая абстракция вообще?
8. Насколько локальным останется следующее похожее изменение?

---

# Итог тем 1.7–1.9

```text
Interface / Contract
↓
задают устойчивую границу

Dependency Inversion
↓
защищает важную логику от технических деталей

SOLID
↓
формализует принципы управляемых изменений

Anti-patterns
↓
показывают, как эти границы разрушаются на практике
```

Для AI-разработки главный вопрос уже не только:

> Работает ли код?

Но и:

> Останется ли система понятной и изменяемой после ещё сотни таких изменений?
