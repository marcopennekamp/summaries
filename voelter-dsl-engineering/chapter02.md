# Chapter 2 – Introduction to DSLs



## 2.1 – Terminology

- **Programming Language:** General-purpose programming languages (GPLs) like Java, C++, Lisp, Haskell, and so on.
- **Internal DSL:** A DSL that is written inside a host language.
- **External DSL:** A DSL that has its own syntax and exists on its own.
- **Target Platform:** The platform your DSL program runs on in the end. It is assumed that the platform itself cannot be changed significantly.
- **Execution Engine:** The changeable engine that bridges the gap between the DSL and the platform.
  - **Interpreter:** A program that loads the DSL program and runs it.
  - **Generator:** A compiler that compiles a DSL program into a representation that can be run on the target platform.
- **Main Ingredients of Languages:**
  - **Concrete syntax:** The notation (not necessarily textual) in which programs are expressed.
  - **Abstract syntax:** A data structure that holds the semantically relevant information expressed by a program.
  - **Static semantics:** Constraints and type system rules that programs have to conform to to be valid.
  - **Execution semantics:** The rules that define how and when various language constructs produce program behavior.
- **Technical DSLs** are designed for programmers, while **Application Domain-specific DSLs** are designed for non-programmers.
- **Meta Model:** A model that defines the language used to describe a model. For example, the meta model of UML is a model that defines all language concepts in UML, e.g. classes, associations and properties.





## 2.2 – From General Purpose Languages to DSLs

- **Domain Specific Language:** A language that is optimized for a domain, which is a set of related problems.
- The **abstractions of the given domain** are a good starting point for expressing programs in this domain. For example, databases have tables, rows and columns, which are direct entities in SQL.
- **Separation of domain concerns:**
  - *Variable concerns* are different for each program written in the DSL. The DSL supports expressing succinct solutions.
  - *Fixed concerns* are the same for every program in the domain. These are encoded in the platform.
  - *Derivable concerns* can be derived by fixed rules from the DSL program. These are always the same for a given DSL program structure and handled by the execution engine.
- Because of performance concerns, you should **be aware of the target platform** when building a language for a given domain. 
- **Advantages of DSLs over APIs:**
  - Languages are **clean abstractions**. They can help express models in extremely succinct ways.
  - DSLs are **restricted** to relevant domain programs. They may even be so defined that only correct programs can be created. 
  - You can provide **IDEs and analysis tools** that go beyond the possibilities of APIs in GPLs.
- The difference between GPLs and DSLs is not clear-cut. Instead, languages can be **more or less domain-specific**. For specific characteristics, see the table on page 31.
- DSLs are typically **much smaller and simpler** than GPLs.





## 2.3 – Modeling and Model-Driven Development

- **Modeling:**
  - *Descriptive:* A model that represents an existing system.
  - *Prescriptive:* A model that can be used to construct the target system.
- In the book, **models are prescriptive**.
- **Model-Driven Software Development (MDSD):** Using models in a prescriptive way.
- **DSL Engineering** is one aspect of MDSD.
- **Difference between programming and prescriptive modeling:** See the table on page 33.
- **Opinion:** There should not be a difference between programming and modeling. Some specific purposes require specific tools, and the integration of these should be easy. This leads to the concept of modular languages.





## 2.4 – Modular Languages

- **Language Size:** The number of concepts in a language.
- **Language Scope:** The size of the application domain.
- **Big languages** have a large set of specific language concepts, while **small languages** have few but powerful and composable language concepts.
- **Modular Languages** are a third option. A modular language consists of a *small* language core and a *big* library of language modules.
- **Language modules** add first-class concepts supporting a target domain. New modules can be created and used at any time. They come with their own syntax, editor, type system and tooling.
- **Language workbenches** are tools where:
  - *Central Idea:* Language designers can **freely define** languages that are **fully integrated** with each other.
  - *Nice to have:* Users manipulate the **abstract information as a primary source** through a **projectional editor.**
  - *Nice to have:* Language designers define a DSL in **three steps:** Meta model, editor, generator. However, the abstract syntax (meta model) may be derived from the concrete syntax (gramar or editor).
  - Incomplete or contradictory information is **persisted.**
  - **Tool support** (syntax highlighting, code completion, static analysis, debugger) is available.
  - Language designers can **implement the complete system.** This usually means that general-purpose languages should also be available.
- **Concrete Syntax:**
  - The **default** concrete syntax should be textual.
  - However, there are also useful additions that go **beyond a textual representation**, for example:
    - Mathematical notation
    - Tables
    - State machine diagrams
    - Data flow systems
  - Textual and graphical notations **must be integrated.**
  - **Visualizations:** Read-only graphical representations of some aspect of a program.
- **Language Libraries:**
  - Many language modules are **sufficiently abstract** to allow **reuse.** These could be published in a library of language modules. See page 38 for examples.
  - Modules have to be able to work with each other. Clear **depency structures** have to be established.
  - Language abstractions have to be **independent of specific technologies,** if they have to be implemented with third-party solutions (e.g. databases for persisting relational data).

  ​



## 2.5 – Benefits of using DSLs

- **Productivity:**
  - A focused DSL can help **reduce grunt work.**
  - Reducing the number of code lines in an application **reduces the complexity** of the program.
- **Quality** improves because of a **removal of unnecessary freedom:**
  - Fewer bugs
  - Better architectural conformance
  - Increased maintainability
- **Validation and Verification:** DSL programs are **more semantically rich** than GPL programs. Being able to use domain concepts leads to:
  - **Analyses** that are easier to implement.
  - More **meaningful error messages.**
  - Easier **manual** review and validation.
- **Data Longevity:**
  - Models expressed in DSLs are **independent of specific implementations**.
  - Changing the DSL implementation is possible without having to change the programs written in the DSL. 
- **A Thinking and Communication Tool:** 
  - Programs written in DSLs are **more effective at communicating** core ideas relevant to the domain.
  - **Building the DSL** can help improve understanding of the given domain, both for the individual and for the team.
- **Domain Expert Involvement:** DSLs can help **involve domain experts** directly, so that they can read or even write program code.
- **Productive Tooling:** 
  - External DSLs come with **tooling**, for example with an IDE. 
  - **Improves productivity** and **reduces the learning curve**.
- **No Overhead:** When generating source code, **domain abstractions** can be compiled out, so that the resulting code can be efficient.
- **Platform Independency / Isolation:** As noted in **Data Longevity**, programs written in a DSL are **independent of the underlying platform and specific implementations.** This enhances:
  - **Portability** (new execution engines for new platforms can be created)
  - **Maintainability** (isolated from platform specifics)





## 2.6 – Challenges of using DSLs

- **Efforts of Building the DSLs:** 
  - Building a DSL is a **relatively large effort.**
  - **Technical DSLs** have huge potential for reuse. Creation is easier to justify.
  - **Application Domain-specific DSLs** have a more limited applicability. However, the cost may be justified nonetheless, because building a DSL is a way of **formalizing business knowledge.**
  - **Modern tools** make building DSLs increasingly cheaper.
- **Language Engineering Skills:** 
  - Building *and designing* DSLs **requires experience and skill.**
  - Language workbenches have a non-trivial **learning curve.**
- **Process Issues:** 
  - Language designers and language users may not be the same people.
  - This can pose a **communication challenge.**
- **Evolution and Maintenance:** You have to plan ahead for maintenance and the evolution of requirements.
- **DSL Hell:** Once developing DSLs becomes easier, developers may be tempted to create their own DSL instead of first searching for an existing one. This may result in many half-baked DSLs.
- **Investment Prison:** Having a significant investment in existing technology may discourage a business to re-evaluate its strategy.
- **Tool Lock-In:** 
  - There is **no interoperability** between DSL tools.
  - This means you get **locked into** using one tool for a specific project.
- **Cultural Challenges:** 
  - There are many **prejudices** against language design that hinder the adoption of DSLs.
  - You have to be prepared to do some **convincing**, independent of technical arguments.
- **When to *not* use DSLs:**
  - The domain is **hard to understand** or there is **insufficient information** about the domain.
  - Domain experts **can't agree on the specifics** of the domain.
  - You don't know enough about the **target platform.**
  - You don't know enought about **software development techniques** (testing and quality assurance, software design, continuous builds, issue tracking, version management). These should be mastered first.





## 2.7 – Applications of DSLs

- A **Utility DSL** automates a specific and well-bounded problem of software development.
- **Architecture DSL:** 
  - A DSL that is used to **describe the architecture** of a larger software system.
  - *Advantage over UML or existing ADLs:* The abstractions of the DSL can be **tailored specifically to the problem domain.**
  - From the architecture model, a code skeleton is **generated.**
  - Help ensure that the system is **consistently implemented** by a **large team.**
- A **Full Technical DSL** supports writing **application logic** of the system in addition to just the structure.
- An **Application Domain DSL** describes the **core business logic** of an application, without giving regard to the technical implementation.
  - They are intended to be **used by domain experts** (non-programmers).
- **DSLs in Requirements Engineering:**
  - *Goal:* Precise and checkable **description of requirements.**
  - Often connected to **prose text.**
- **DSLs used for Analysis:**
  - *Buzzwords:* Analysis, checking, proofs.
  - *Goal:* Express concerns in a formalism that lends itself to **formal verification.**
- **DSLs used in Product Line Engineering:** Using DSLs to describe a specific product in a product line, where the product line as a whole is the domain of the DSL.




## 2.8 – Differentiation from other Approaches

#### 2.8.1 – Fluent APIs and Internal DSLs

- **Fluent APIs:** An API that allows chaining method calls, thereby making it feel more natural and resulting in concise code.
  - The objects returned from each method call may differ. This means that subsequently provided interfaces can be **constrained** to particular method sets (similar to a grammar which constrains the set of possible operations).
- Host languages with more flexible syntax can support **internal DSLs** instead of just fluent APIs.
- Internal DSLs are **missing IDE support.**

#### 2.8.2 – Compiler Construction

- Many techniques from compiler construction are **relevant to DSL engineering.**
- **Differences:**
  - DSL tools are more **powerful**, **convenient** and include **IDE support.**
  - **Optimizations** are more important in compiler construction.

#### 2.8.3 – UML

- UML is not a DSL, but it provides **profiles** for adding new concepts to UML.
- UML is also **used in model-driven software development.**
- The opinion of the author is that **building DSLs is a better idea** most of the time.

#### 2.8.4 – Graphical vs. Textual

- Both graphical and textual languages have their advantages.
- The book is **biased towards textual languages:**
  - More generally useful
  - Scale better
  - Less development effort
- You should **start** with a textual language and introduce graphical representations only when the need arises.

