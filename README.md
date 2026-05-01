# bse-lr2-konovalov

Лабораторна робота №2 з дисципліни "Основи програмної інженерії"

**Тема:** Моделювання системи (UML)

## Автори

- **Коновалов Олександр**, група ПЗПІ-25-6, oleksandr.konovalov1@nure.ua

## Технології

- Діаграми: Mermaid
- Експорт: @mermaid-js/mermaid-cli
- VCS: Git + GitHub

## Опис проєкту

**Система створення академічних бібліографічних посилань** — веб-сервіс для автоматичного формування бібліографічних цитат на основі ідентифікатора публікації (DOI, URL або ISBN).

Сервіс підтримує популярні стилі цитування: APA, MLA, Chicago. Користувач вводить ідентифікатор, система отримує метадані через зовнішні API (Crossref для DOI, Open Library для ISBN, веб-скрапінг для URL) і генерує готове посилання, яке можна скопіювати або зберегти.

**Актори системи:**

- **Гість** — може генерувати та копіювати цитати без реєстрації
- **Зареєстрований користувач** — зберігає цитати, переглядає історію, керує обліковим записом
- **Адміністратор** — управління акаунтами, налаштування доступних стилів цитування

## Функціональні вимоги

| ID | Вимога | Актор | Пріоритет |
|----|--------|-------|-----------|
| FR-01 | Система дозволяє ввести ідентифікатор публікації (DOI, URL або ISBN) | Гість, Користувач | Must |
| FR-02 | Система отримує метадані публікації через зовнішні API (Crossref, Open Library, веб-скрапінг) | Система | Must |
| FR-03 | Система генерує бібліографічне посилання у вибраному форматі (APA, MLA, Chicago) | Гість, Користувач | Must |
| FR-04 | Користувач може скопіювати згенероване посилання у буфер обміну | Гість, Користувач | Must |
| FR-05 | Зареєстрований користувач може зберігати посилання та переглядати історію | Зареєстрований користувач | Should |
| FR-06 | Адміністратор може керувати обліковими записами та налаштуваннями стилів цитування | Адміністратор | Should |
| FR-07 | Система підтримує автентифікацію користувачів через email-реєстрацію | Гість → Користувач | Must |

## Діаграма прецедентів

Виконав: Коновалов Олександр

```mermaid
flowchart LR
    Guest(["Гість"])
    User(["Зареєстрований користувач"])
    Admin(["Адміністратор"])

    subgraph system["Система бібліографічних посилань"]
        UC01["UC-01: Ввести ідентифікатор"]
        UC02["UC-02: Отримати метадані"]
        UC03["UC-03: Згенерувати цитату"]
        UC04["UC-04: Скопіювати цитату"]
        UC05["UC-05: Зберегти цитату / Переглянути історію"]
        UC06["UC-06: Керувати акаунтами та стилями"]
        UC07["UC-07: Автентифікуватися"]
    end

    Guest --> UC01
    Guest --> UC03
    Guest --> UC04

    User --> UC01
    User --> UC03
    User --> UC04
    User --> UC05
    User --> UC07

    Admin --> UC06
    Admin --> UC07

    UC03 -.->|"include"| UC02
    UC05 -.->|"extend"| UC03
```

## Діаграма класів


```mermaid
classDiagram
    class User {
        <<abstract>>
        #id: string
        #email: string
        +getRole() string
    }

    class Guest {
        +generateCitation(request: SearchRequest, style: CitationStyle) Citation
    }

    class RegisteredUser {
        -savedCitations: Citation[]
        +generateCitation(request: SearchRequest, style: CitationStyle) Citation
        +saveCitation(citation: Citation) void
        +getHistory() Citation[]
        +login(email: string, password: string) boolean
    }

    class Admin {
        +manageUsers() User[]
        +manageStyles() CitationStyle[]
        +deleteUser(userId: string) boolean
        +login(email: string, password: string) boolean
    }

    class Citation {
        -id: string
        -rawMetadata: object
        -formattedText: string
        -style: CitationStyle
        -createdAt: Date
        +format(style: CitationStyle) string
        +copyToClipboard() void
    }

    class CitationStyle {
        -name: string
        -template: string
        +formatCitation(metadata: object) string
    }

    class SearchRequest {
        -identifier: string
        -type: IdentifierType
        +validate() boolean
        +getType() IdentifierType
    }

    class MetadataProvider {
        <<interface>>
        +fetchMetadata(identifier: string) object
    }

    class CrossrefProvider {
        -apiUrl: string
        +fetchMetadata(identifier: string) object
    }

    class OpenLibraryProvider {
        -apiUrl: string
        +fetchMetadata(identifier: string) object
    }

    class WebScraperProvider {
        +fetchMetadata(url: string) object
    }

    class CitationGenerator {
        -providers: MetadataProvider[]
        -styles: CitationStyle[]
        +generate(request: SearchRequest, style: CitationStyle) Citation
        +resolveProvider(type: IdentifierType) MetadataProvider
    }

    User <|-- Guest
    User <|-- RegisteredUser
    User <|-- Admin
    MetadataProvider <|.. CrossrefProvider
    MetadataProvider <|.. OpenLibraryProvider
    MetadataProvider <|.. WebScraperProvider
    CitationGenerator o-- "1..*" MetadataProvider : aggregation
    CitationGenerator --> CitationStyle : uses
    CitationGenerator --> Citation : creates
    Citation *-- "1" CitationStyle : composition
    RegisteredUser o-- "0..*" Citation : aggregation
    CitationGenerator --> SearchRequest : uses
    Guest --> CitationGenerator : uses
    RegisteredUser --> CitationGenerator : uses
```

## Діаграма послідовності


Сценарій: Зареєстрований користувач генерує цитату у форматі APA за DOI

```mermaid
sequenceDiagram
    actor User as Зареєстрований користувач
    participant RU as RegisteredUser
    participant CG as CitationGenerator
    participant SR as SearchRequest
    participant CP as CrossrefProvider
    participant OLP as OpenLibraryProvider
    participant WSP as WebScraperProvider
    participant CS as CitationStyle
    participant C as Citation

    User->>CG: generate(request, apaStyle)
    activate CG

    CG->>SR: validate()
    activate SR
    SR-->>CG: true
    deactivate SR

    alt type == DOI
        CG->>CP: fetchMetadata(doi)
        activate CP
        CP-->>CG: metadata
        deactivate CP
    else type == ISBN
        CG->>OLP: fetchMetadata(isbn)
        activate OLP
        OLP-->>CG: metadata
        deactivate OLP
    else type == URL
        CG->>WSP: fetchMetadata(url)
        activate WSP
        WSP-->>CG: metadata
        deactivate WSP
    end

    CG->>CS: formatCitation(metadata)
    activate CS
    CS-->>CG: formattedText
    deactivate CS

    CG->>C: new Citation(metadata, formattedText, style)

    CG-->>User: citation
    deactivate CG

    User->>C: copyToClipboard()

    opt authenticated
        User->>RU: saveCitation(citation)
    end
```

## Матриця трасовності

| FR | Прецедент | Задіяні класи | Діаграма послідовності |
|----|-----------|---------------|----------------------|
| FR-01 | UC-01: Ввести ідентифікатор | SearchRequest | Кроки 1-3 (validate) |
| FR-02 | UC-02: Отримати метадані | MetadataProvider, CrossrefProvider, OpenLibraryProvider, WebScraperProvider | Кроки 4-5 (fetchMetadata) |
| FR-03 | UC-03: Згенерувати цитату | CitationGenerator, CitationStyle, Citation | Кроки 6-9 (formatCitation, create) |
| FR-04 | UC-04: Скопіювати цитату | Citation | Крок 10 (copyToClipboard) |
| FR-05 | UC-05: Зберегти / Історія | RegisteredUser, Citation | Крок 11 (saveCitation) |
| FR-06 | UC-06: Керувати акаунтами | Admin, User, CitationStyle | Не показано (сценарій адміна) |
| FR-07 | UC-07: Автентифікуватися | User, RegisteredUser, Admin | Не показано (сценарій автентифікації) |

## Встановлення

```bash
git clone https://github.com/oleksandrkonovalov1/bse-lr2-konovalov.git
cd bse-lr2-konovalov
npm install
npm run export:all
```

## Ліцензія

MIT License
