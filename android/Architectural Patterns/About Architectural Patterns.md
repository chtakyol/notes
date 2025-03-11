1. Uygulama Mimarileri: * MVVM (Model-View-ViewModel) mimarisini öğrenmek. Bunun yanı sıra başka uygulama mimarilerini bilmek. (örneğin, MVC, MVP, Clean Architecture). 2. SOLID Prensipleri: * SOLID prensipleri hakkında bilgi sahibi olunması. 3. Mimariler Arasındaki Farklar: * Öğrenilen uygulama mimarilerinin farkları, avantajları ve dezavantajları hakkında kısa bir yazı hazırlanmalıdır. Bu yazıda mimarilerin kullanım alanları, hangi senaryolarda daha uygun oldukları gibi konulara değinilmelidir. 4. Mentör’e Sunum ya da Yazı: * Konularla ilgili olarak mentöre bir sunum yapılmalı veya yazılı bir doküman hazırlanarak sunulmalıdır.

# What is an Architectural Pattern?
#### Intro
An architectural pattern is a general, reusable solution to organize the structure and responsibilities of your codebase.

Architectural patterns define a way to structure your app by clearly separating responsibilities between UI, logic, and data — improving maintainability, testability, and scalability.

It defines how different parts of your application (like UI, data, business logic) communicate and depend on each other.

**In Android development, architectural patterns help you**
- Separate concerns (UI, logic, data)
- Make code more testable, readable, and maintainable
- Handle scalability as your project grows
- Avoid “Massive Activities/Fragments” (which handle too much logic)

**Key Goals of an Architectural Pattern**
- **Modularity**: Each component should do one thing well.
- **Separation of concerns**: UI logic, business logic, and data management are in separate layers.
- **Testability**: Logic should be testable independent of Android framework.
- **Scalability & Maintainability**: Easy to modify or extend features in the future.

> Beyond structural clarity, architectural patterns make your codebase easier to collaborate on, more readable, and more adaptable to future changes — whether that's swapping a data source, refactoring modules, or onboarding new team members.

Let's clear the meaning and definitions first;
#### What is a Model
The part of your app that holds and manages the data and business logic.
Responsibilities:
- Fetch or store data (from API, database, file, etc.)
- Apply business rules (e.g., calculations, filtering, etc.)

**Typical examples in Android:**
- Repository classes
- Room database or Realm
- Retrofit API services
- Data classes / DTOs

	“Model is all about the data — where it comes from and how it’s processed.”
#### What is Business Logic?
Business logic is **the core set of rules**, **decisions**, and **processes that define how your app behaves based on its purpose** — it’s what makes your app do what it’s supposed to do.

It’s independent of UI, meaning it has nothing to do with how your app looks or how users interact visually — it’s purely about what your app does.

**For example:** 
- In a e-commerce app discount calculation
- In a fitness app calorie burn calculation

	Business logic is the smart part of your app that applies rules, makes decisions, and processes data based on the app's purpose.

#### What is a Domain Layer
**Dictionary definition of Domain**; an area of knowledge, activity, or interest.
In software, domain refers to; The core business logic and rules that define what the application does, independent of how it does it.

	The domain is the heart of your application — the part that models your real
	world problem and solves it, regardless of UI, frameworks, databases, or
	networking.

What is the Domain Layer in Software Architecture? It’s a layer that contains:
- Business rules
- Entities (core models)
- Use Cases (interactors)

It knows nothing about UI, databases, or external libraries. It is meant to be pure Kotlin code, independent, and reusable.**
#### What is a View
The UI layer that displays the data and listens for user actions.
Render data on the screen
- Capture user interactions (clicks, input, scrolls)
- Forward those interactions to a controller/presenter/viewmodel

**Typical examples in Android:**
- Activities / Fragments / Composables
- XML Layouts

	“View shows data and reacts to user inputs — it doesn’t contain logic. (In best case😁)”


### Popular architectural design patterns
These are the most used architectural patterns for android; MVC (Model View Controller), MVP (Model View Presenter), MVI (Model View Intent), MVVM; and apart from these there are also some principles called Clean Architecture.
#### Model View Controller

In MVC, the architecture is split into three parts:
- Model -> Manages data and business logic
- View -> Displays UI and receives user input
- Controller -> Handles input events, coordinates between View and Model

**In Android**, Activities or Fragments often act as Controllers.
- In Android’s traditional implementation:
- Activity/Fragment is both Controller and View, which creates tight coupling.
- Not ideal for scalability.
- With Jetpack Compose, things shift a bit — but you can still explore the MVC pattern structurally.

**Downsides** of MVC in Android
- In real Android apps, Activities often become both View + Controller, making them bloated.
- MVC doesn't scale well in Android due to this coupling.
- No clear separation for business logic or domain rules in complex cases.

#### Model View Presenter

In MVP, the architecture improves over MVC by clearly separating responsibilities:
- Model -> Manages data and business logic
- View -> Displays UI and passes user interactions to the Presenter
- Presenter -> Handles the logic, talks to the Model, and updates the View
- 
Unlike MVC, the View never directly talks to the Model. The Presenter becomes the god — it coordinates everything.

#### Model View ViewModel
**MVVM (Model–View–ViewModel)** is an architectural pattern that promotes a **clear separation of concerns** between UI and business logic by introducing an intermediate component: the **ViewModel**.
- **Model** -> Handles data and business logic (e.g., database, API)
- **View** -> UI layer (Composables, XML, etc.) that observes ViewModel
- **ViewModel** -> Holds UI-related state, exposes observable data (e.g., State, LiveData, StateFlow), and communicates with Model

MVVM -or its variaties- is considered the de facto architecture pattern in Android development today. Google recommends it officially, naturally fits with Jetpack Compose and scales well with Clean Architecture.

MVVM helps:
- Keep UI logic and business logic decoupled
- Improve testability
- Handle reactive state management efficiently 

	The ViewModel acts as a bridge between Model and View, exposing state in a way the View can observe and update automatically.
#### Model View Intent
**MVI (Model–View–Intent)** is an architecture pattern based on **unidirectional data flow (UDF)**
It emphasizes:
- A **single source of truth (state)**.
- A **loop of state transformation**: View sends Intent → Business logic returns new State → View renders it.

**Advantages of MVI:**
- **Predictable** behavior (pure state changes)
- All UI logic is driven by state
- Easier to debug (single source of truth)
- Great for **Jetpack Compose**, because Compose re-renders on state change
------
### Clean Architecture
**Clean Architecture** term comes from Robert C. Martin's book/articles and it is a software architecture pattern that promotes:
- Separation of concerns
- Independence between layers
- Testability
- Maintainability
- Scalability
#### 🏛️ Core Idea: Layered Architecture with Dependency Rule

Clean Architecture is structured as a **layered architecture**, where each layer has a specific responsibility and dependencies always point inward (i.e., outer layers depend on inner layers, but never the other way around).
#### Why Clean Architecture?
- Avoids spaghetti code and tight coupling
- Improves testability (e.g., you can test UseCases without Android dependencies)
- Makes your app modular, reusable, and scalable
- Enables easier onboarding for new devs
- Flexible to changes (e.g., replace Room with another DB without touching domain logic)
- Makes collabrations easier
#### Google's Recommended Layered Architecture (Android App Architecture Guide)
Google’s architecture guidance is outlined in their official guide: 👉[https://developer.android.com/topic/architecture](https://developer.android.com/topic/architecture)

It encourages a practical three-layered architecture:

**UI Layer**
- Follows Unidirectional Data 
- Flow
- Jetpack Compose
- ViewModel acts as the bridge between UI & domain
- Holds UI state, exposes it as State, StateFlow
- Example: UserProfileViewModel, CounterScreen()

**Domain Layer** (optional but recommended for larger apps)
- Contains pure business logic and use cases
- Independent of Android framework
- Highly reusable & testable
- Example: UseCase's, Domain Objects

**Data Layer**
- Responsible for data sources
	- Room DB
	- Retrofit API
	- DataStore
	- Repositories
- GPS location providers or other sensors
- Bluetooth data providers
- Network connectivity status provider
-----
# SOLID Principles
The SOLID principles are a set of five object-oriented design principles intended to help software developers create more understandable, flexible, and maintainable code. 

These principles are widely used in object-oriented programming and software engineering in general.

In **Android development**, especially when applying **MVVM** and **Clean Architecture**, we naturally start aligning with the **SOLID principles** — sometimes without even consciously trying.

### Single Responsibility Principle (SRP) in Android
- **In practice**: Each layer in Clean Architecture (UI, Domain, Data) and each class in MVVM (ViewModel, UseCase, Repository) focuses on a single responsibility.
- **Examples**:
    - `Activity/Fragment`: Only handles UI rendering and user interactions.
    - `ViewModel`: Manages UI-related business logic.
    - `UseCase`: Encapsulates a single piece of business logic.
    - `Repository`: Abstracts data access (from DB, network, etc.).
- **Result**: Easy testability and modularity, because everything has one clear job.
### Open/Closed Principle (OCP) in Android
- **In practice**: We often rely on abstractions/interfaces to extend functionality without modifying existing code.
- **Examples**:
    - Adding a new `RemoteDataSource` or `LocalDataSource` without changing the `Repository` contract.
    - Using `sealed classes` for `UiState` or `Result<T>` — you can extend behavior via new subtypes.
- **Result**: Adding new features rarely requires touching core code, reducing risk of regression.
### Liskov Substitution Principle (LSP) in Android
- **In practice**: We build components that conform to abstract types, so they can be replaced freely.
- **Examples**:
    - `RepositoryImpl` can be replaced with a mock or fake during testing, thanks to interfaces.
    - A `BaseViewModel` subclass must behave in a way that doesn't break the UI flow.
    - Custom `Drawable`, `View`, or `Adapter` components that replace base ones without surprising side-effects.
- **Result**: You can swap implementations in tests or refactor code without breaking things.

### Interface Segregation Principle (ISP) in Android
- **In practice**: Define small interfaces tailored to specific responsibilities instead of bloated ones.
- **Examples**:
    - `LoginRepository` and `ProfileRepository` instead of one big `UserRepository`.
    - Splitting `Navigator`, `UserSessionManager`, `PreferencesManager` into separate, clear interfaces.
- **Result**: Easier testing, mocking, and less coupling between features.
### Dependency Inversion Principle (DIP) in Android
- **In practice**: High-level components (e.g., UseCases, ViewModels) depend on abstractions, not concrete implementations.
- **Examples**:
    - Using Dagger/Hilt to inject interfaces, not implementations.
    - `ViewModel` depending on a `UserUseCase` or `UserRepository` interface, not a `UserRepositoryImpl`.
    - `Room` DAO interfaces injected via Hilt — code depends on interfaces, not Room’s internal classes.
- **Result**: High flexibility and ease of testing/mocking.