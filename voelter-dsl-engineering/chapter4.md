# Chapter 4 – Design Dimensions



## 4.1 – Expressivity

- Expressivity measures the **effectiveness** of writing programs in a given language for a given domain:
  - Increased expressivity means that **programs are shorter.** Usually, shorter programs are **more efficient** than longer programs.
  - $|p_L|$ is the **program size** of $p$ encoded in language $L$.
  - **Formal Definition:** A language $L_1$ is *more expressive in domain* $D$ than a language $L_2$, written $L_1 \prec_D L_2$, if for each $p \in P_D \cap P_{L_1} \cap P_{L_2}$ it follows that $|p_{L_1}| < |p_{L_2}|$.
- **Realistic version:** The language is *mostly* more expressive.
- **Expressivity and the Domain Hierarchy: **Progressive specialisation of a domain enables progressively **more expressive languages.**

#### 4.1.2 – Abstraction

- **Linguistic Abstraction** is the introduction of new abstractions for specific domain concepts. This makes the concepts easy to identify and their structure and semantics well defined.
  - **Declarativeness:** The property of a language of having first-class concepts for abstractions in the specific domain. **No semantic analysis** has to be performed to understand the semantics of the abstraction.
  - **Advantages:** 
    - The language is simple to process.
    - It makes the language suitable for domain experts (who are not trained in building their own abstractions).
  - **Disadvantages:**
    - The abstractions must be known in advance or the language must be evolved frequently.
    - Too many different abstractions can lead to large, bloated or inelegant languages.
- **In-Language Abstraction** means building new abstractions from already existing language features (e.g. classes, higher-order functions, generics, traits, monads).
  - Does not provide **declarativeness.** We need **semantic analysis to understand** what the user wanted to express with the in-language abstraction.
- In-language abstractions can be used to provide domain-specific abstractions through a **standard library** without having to add abstractions to the original language.

#### 4.1.3 – Language Evolution Support

- Language evolution **should be carefully done,** because it may break existing models.
- **Solutions:**
  - Strict configuration management discipline.
  - Version models to trigger the right editors and processors.
  - Model migration tools to migrate models from old to new versions.
- Breaking changes should be **minimized.**
- **Deprecation** may be used.

#### 4.1.6 – Platform Influence

- There are two reasons why **a platform may influence a language:** *Runtime Efficiency and *
- **Runtime Efficiency:** The wider the semantic gap between language and platform, the harder it is to write an efficient code generator.
- **Platform Limitations:** Memory, disk space, bandwidth, etc. may limit or influence language design.





## 4.2 – Coverage

- There is always a domain $D_L$, called the **domain determined by** $L$, with $P_{D_L} = P_L$.
- A language $L$ **fully covers** a domain $D$ if $P_D \subseteq P_L$.
- There is **not always** a language that fully covers a domain, unless we revert to GPLs. Many languages do not *fully* cover their domain.
- **Coverage of a Domain by a Language:** $C_D(L) = \frac{|P_D \cap P_L|}{|P_D|}$.
- **Reasons not to *fully* cover a domain:**
  - **Requirements evolution** changes the domain.
  - The language is **deficient** and should be redesigned.
  - Covering just a **subset** of $D$ is good enough. Covering the whole domain would be too costly compared to the benefits or make the language too bloated.





## 4.3 – Semantics and Execution

- **Static Semantics** are implemented by constraints and type system rules.
- **Execution Semantics** denote the observable behavior of a program.
- $semantics(p_{L_D}) := q_{L_{D-1}}$ where $OB(p_{L_D}) = OB(q_{L_{D-1}})$ with $OB$ being a function that defines observable behavior. Thus, the semantics of a language $L_D$ can be defined in terms of a language $L_{D-1}$.
- The equality above is often **tested** by using a sufficient amount of tests or model checking.

#### 4.3.1 – Static Semantics/Validation

- **Constraints:** Boolean expressions that check some property of a model.
  - **Example:** Attribute names in a record should be unique.
  - For a model to be **correct,** all constraints must be satisfied.
  - **Tip:** Make sure that **all constraints are actually implemented.** Especially think about the obvious ones!
  - Some constraints can be **costly to process,** so many DSL tools allow language developers to classify constraints according to their cost.
- **Type Systems:** A formalism for systematically establishing the type of an expression, and its related type checking constraints.
  - Type systems are a form of **constraint checking.**
- There are **two approaches** to designing constraints and type systems:
  - **Specification/Conformance:** First, the user **declares** the intent of using a constraint (a state machine is verifiable). The system then checks whether the program **conforms** to the properties required by the intent (the state machine passes the model checker).
  - **Inference/Consistency:** The system **infers** the intent from the program (state machines are verifiable when forbidden features aren't used) and then does constraint checking.
- **Multi-Level Constraints:** A constraint system can include constraints that are checked with every compilation, but also constraints that must be explicitly switched on or are only used in specific circumstances. This can result in a **tiered system,** where progressively more levels of constraints can be switched on.


#### 4.3.2 – Correctness of the Execution Engine

- It has to be ensured that the **execution engine** executes the DSL program correctly.
- We can write a **unit test** in the GPL implementing the DSL, which checks the correctness for one DSL program.
- We have to test many programs to ensure **full branch coverage.**
- It is also possible to **write tests in the DSL**, after extending it for testing.
  - More convenient.
  - Gives independence of the execution engine.
- **Penetration testing** can be used to catch unintended behavior.

#### 4.3.3 – Transformation

- **Transformations** define the **execution semantics** of a DSL by transforming DSL programs to programs of another language.
- DSL programs may be mapped to **multiple languages.**
  - **Example:** A DSL program is transformed into the executable code and additionally input for a model checker.
- **Multi-staged Transformation:** A DSL program goes through **multiple transformations** before arriving at the target format. This chain of transformations is known as **cascading.**
  - This is a form of **modularization,** so it breaks down the **semantic gap** between the DSL and the target language to smaller problems.
  - Carries the **potential for reuse** of a stage.
- **Efficiency and Optimization:** The transformer can take advantage of domain semantics expressed in the DSL to generate very efficient code.
  - Building optimizations can be **very expensive.**
- **Care about Generated Code:** For development, testing and maintenance of the DSL transformer, generated code should be **easy to read.**
  - Generating readable code may be **too hard** and not worth the effort.
  - Generating **boilerplate code** is sometimes the best option, even though it impairs readability.
  - Sometimes there are **performance or code size constraints,** for which you sacrifice readability.
- **Platform:** It is possible to add a **domain-specific platform** to the target language that provides many of the DSL functions that would have otherwise been generated.

#### 4.3.4 – Interpretation

- An **interpreter** is a program that runs a given DSL program.
- The **execution strategy** depends on the programming paradigm:
  - **Imperative programs** are executed step by step.
  - For **functional programs,** the interpreter evaluates functions.
  - A custom strategy is used for **declarative programs.**
- In practice, interpreters are often **written in GPLs.**

#### 4.3.5 – Transformation versus Interpretation

- **Code Inspection:** The structure of generated code can be analysed to find more instances where the DSL might be used. This is difficult with interpreters.
- **Debugging:** 
  - *Transformation:* Straightforward if the code is well-structured and a debugger exists.
  - *Interpretation:* It requires a lot of work to implement direct DSL-level debugging with interpreters.
- **Performance and Optimization:** Generated code is usually faster, because both the DSL compiler and the target language compiler can optimize the code. Additionally, an interpreter adds another layer, which typically worsens performance.
- **Platform Conformance:** Transformation can tailor generated code to any target platform. No support libraries are needed.
- **Modularization:** It is possible to chain transformations quite efficiently. 
- **Turnaround Time:** Since no code generation and compilation are required when using an interpreter, turnaround time is usually better.
- **Runtime Change:** When using an interpreter, the DSL program can be changed during runtime, possibly even with the language editor included.

#### 4.3.6 – Sufficiency

- A program fragment is **sufficient** for transformation $T$ if the fragment contains all necessary information for $T$. 
- A sufficient fragment can be transformed **without loading other fragments.** This is an advantage in large systems (compilation time and scalability).

#### 4.3.7 – Synchronizing Multiple Mappings

- When supporting multiple target languages, one needs to ensure that the semantics of all programs are **identical.**
- **Example:** Using an interpreter for development and a code generator for production.
- **Approach:** Tests can be written in a DSL and then executed for every target.

#### 4.3.9 – Reduced Expressiveness and Verification

- **Limiting expressiveness** makes code more analysable.
- Such code can sometimes be **statically verified.**
- Thus, it is useful to limit the expressiveness of a DSL **if verification is useful** for the model purpose.

#### 4.3.10 – Documentation

- Documentation can be useful to **convey the meaning** of a language feature to non-experts and non-programmers.
- **Approaches:** Prose documentation, test cases, simulators.





## 4.4 – Separation of Concerns

- A domain may consist of different **concerns,** where each concern covers a different aspect of a domain.
- There are two **approaches** to deal with the concerns in a domain:
  - A **single DSL** addresses all concerns of a domain.
  - Separate **concern-specific DSLs** address subsets of concerns.

#### 4.4.1 – Viewpoints for Concern Separation

- Concern-specific DSLs that only address a subset of concerns are called **viewpoints.** 
- **Dependencies** between viewpoints should be well-defined and cycle-free.
- **Advantages of viewpoints:**
  - If users work with different viewpoints, they only see the part of the model that is **relevant** to them.
  - The model fragments can be modified, stored and checked in/out **separately.**
  - Viewpoints support **1-to-n relationships** between a core concern and multiple enhancing concerns.

#### 4.4.2 – Viewpoints as Annotation Models

- An **annotation** provides additional data for technical or transformation purposes.
- In multi-stage transformation, an **intermediate model** may be generated for which we want to specify annotations.
- Since we can't edit the intermediate model directly, we can **externalize the annotation** into its own model (viewpoint).

#### 4.4.3 – Viewpoint Consistency

- Viewpoint fragments sometimes **depend** on other fragments via **references.**
- This should be checked with a **constraint checker.**
- Most tools support **checks** for the existence of the reference targets by default.
- Checking the other direction, that an element **is referenced** by some other element, is harder. There are two problems with this:
  - **Performance** due to a large search space.
  - The fragment that references the element may be **out of reach** of the constraint checker.

#### 4.4.4 – Cross-Cutting Concerns

- A **cross-cutting concern** does not fit into the chosen modularization approach (see Figure 4.20 on page 106).
- There are **different classes** of cross-cutting concerns:
  - **Handled by Execution Engine:** The cross-cutting concern can be handled completely by the execution engine.
  - **Modularized in DSL:** While the concern cuts across the system, it can be modularized with additional support by the execution engine (e.g. roles and permissions).
  - **Cross-Cutting in the DSL:** Cross-cutting concerns must be implemented by **inserting the code** in all relevant locations or by using some form of **aspect weaving.**

#### 4.4.5 – Views on Programs

- In **projectional editors,** all viewpoints can be stored in the same model tree.
- Different viewpoints are then materialized with **different projections.**
- **Advantages:** Additional views for concerns can be added later, handling references is easier.

#### 4.4.6 – Viewpoints for Progressive Refinement

- Viewpoints can be used in a DSL that supports representing **multiple project phases** (e.g. requirements, component design, non-functional properties, implementation). Each project phase has its own viewpoint.

#### 4.4.7 – Model Synchronization

- The **same information** may be represented in two or more viewpoint models.
- This information must be **synchronized.**
- When consistency between viewpoints can be described formally, synchronization can be **automated.**
- **Possible complications:**
  - A dependency is **bidirectional.** **Changes** are allowed in more than one model.
  - Changes to models with bidirectional dependencies made by two users may **occur simultaneously.**
  - Dependencies might have been **forgotten** when the language was designed.
- Sometimes, users have to express a correspondence between models **manually.**





## 4.5 – Completeness

- **Completeness** is the degree to which a language can express the information needed to execute a program.
- Programs expressed in an **incomplete DSL** require additional specification like configuration files or code written in another language.
- **Formally:**
  - Let $G : L_D \rightarrow L_{D-1}$ be a transformer function, $p \in L_D$ and $q \in L_{D-1}$.
  - For a **complete language:** $OB(p) = OB(G(p)) = OB(q)$.
  - For an **incomplete language:** $OB(G(p)) \subset OB(p)$. Additional code in $L_{D-1}$ needs to be written to obtain a complete program in $L_{D-1}$.

#### 4.5.1 – Compensating for Incompleteness

- **Integrating** $L_{D-1}$ can be done in several ways:
  - Calling **"black box" $L_{D-1}$ code** written in another file.
  - **Embedding $L_{D-1}$ code.**
  - **Generation Gap pattern:** Using composition mechanisms of $L_{D-1}$ to add the manually written code via new files. For example: Implementing an abstract class or interface, using `#include`, reflection, aspect-oriented programming, design patterns.
  - Inserting code into the generated code with **protected regions.** Code in protected regions is not overwritten when the file is regenerated. However, this solution leads to issues with version control and files generated from no longer existing models.
- Incompleteness is not a problem with **developer DSLs,** since they are comfortable writing code in the lower-level programming language.
- **Domain experts** are usually not able to write $D_{-1}$ code. Solutions include:
  - Having **developers** write the remaining code.
  - Develop a **standard library** that can be called into by domain experts.
- **Semantic Consistency:** 
  - You have to ensure that the promises made by the **DSL constraints** are also kept in the manually written code.
  - **Approach 1:** Generate code that does not allow violation of model promises.
  - **Approach 2:** Use code checkers or architecture analysis tools.

#### 4.5.2 – Roundtrip Transformation

- **Roundtrip Transformation:** An $L_D$ program can be recovered from a program in $L_{D-1}$.
- **Traditional Use Case:** Generating a UML model from code.
- Roundtripping support is **discouraged** by the book. 





## 4.6 – Language Modularity

- Language modularity describes the ability to **reuse (parts of) languages** in new contexts.
- This requires the **composition** of abstract syntax, concrete syntax, static semantics and execution semantics.
- **Composition Techniques:** Referencing, extension, reuse, embedding.
  - These are distinguished by **fragment structure** and **language dependencies.**
  - **Fragment Structure:** Whether two languages can be syntactically mixed (*heterogenous*) or whether separate viewpoints are used (*homogenous*).
  - **Language Dependencies:** Does the language's design depend on knowledge about a composition partner to be composable (*dependent*) or not (*independent*)?
    - A *dependent* language **can not be reused** without the language it depends on.
  - See **Figure 4.24** on page 117.
- DSL reuse helps avoid **DSL Hell.**

#### 4.6.1 – Language Referencing

- **Language Referencing** allows *homogenous* fragments with cross-references to other fragments, while the referencing language is *dependent* on a referenced language.
- The **referencing language** $l_2$ uses concepts from the **referenced language** $l_1$.
- **Viewpoints** as discussed in section 4.4 utilize language referencing.
- **Progressive refinement** as discussed in section 4.4.6 utilizes language referencing between the different project phases.

#### 4.6.2 – Language Extension

- **Language Extension** allows *heterogenous* fragments, while the extending language is *dependent* on a base language.
- The **extending language** $l_2$ adds concepts to the **base language** $l_1$.
- A **fragment** $f$ can contain concepts from both $l_1$ and $l_2$.
- This idea fits well with **hiearchical domains,** since a language $L_{D+1}$ may extend a language $L_{D}$.
- **Assimilation:** A heterogenous fragment containing concepts from both $L_D$ and $L_{D+1}$ is transformed to a homogenous fragment in $L_D$.
- Especially useful for **bottom-up domains,** since we can build the common patterns expressed in $L_{D-1}$ into a language $L_D$ that can be used in-place.
- **Two Types of Extension:**
  - **Extension Flavor:** Small language extensions that don't change the overall feel of the language (e.g. adding a `complex` type to C).
  - **Embedding Flavor:** A completely new language is created that reuses base language syntax (e.g. state machines in mbeddr C). Feels like language embedding, but is still language extension, because the extending language is dependent on the base language.
- Language extension is also useful for addressing **language evolution.**
- **Disadvantages:**
  - The extending language is **closely tied** to the base language. Supporting other base languages is hard.
  - Interactions with the base language can make **semantic analysis** of heterogenous fragments hard.
- **Restriction:** Language extension can be used to restrict the set of language features available in the subdomain.
  - Often implemented by adding **additional costraints.**
  - A **marker concept** may be added to indicate where restriction rules should be applied (e.g. marking some code module as `safe`).

#### 4.6.3 – Language Reuse

- **Language Reuse** allows *homogenous* fragments written in *independent* languages.
- The **reused language** $l_2$ is used in combination with the **context language** $l_1$.
- This makes $l_2$ **reusable** with different context languages.
- We can use an **adapter language** $l_A$ to combine the two independent languages: $l_A$ extends $l_2$ and references concepts from $l_1$.
- Useful for DSLs that could be **reused in many domains.**

#### 4.6.4 – Language Embedding

- **Language Embedding** allows *heterogenous* fragments to contain *independent* languages.
- The **embedded language** $l_2$ can be used in a fragment of the **host language** $l_1$.
- We can use an **adapter language** $l_A$ extending $l_1$ that contains $l_2$ and combines it with $l_1$.
- The embedded language must often be **extended as well,** for example to make local variables defined in a host language available to an embedded query language.
- **Embedding Meta Data:**
  - **Meta data** are program elements that are not essential to the semantics of a DSL program, such as documentation.
  - **Language Embedding** is the right composition mechanism for meta data.

#### 4.6.5 – Implementation Challenges and Solutions

- **Syntax:**
  - **Referencing/Reuse:** Since fragments are homogenous, concrete syntax is not mixed.
  - **Extension/Embedding** requires modular concrete syntax definitions, because these languages are mixed in the same fragment.
    - **Problematic** for some parser technologies.
    - **Projection editors** avoid composition problems by definition.
- **Static Semantics:**
  - **Referencing:** The static semantics of the referencing language have to take into account the referenced language.
  - **Extension:** The base language's type system must be extendable.
  - **Reuse/Embedding:** Since the languages are independent, the adapter language has to provide type mappings.
- **Transformation:**
  - **Referencing** languages can be handled in three ways:
    - **Single-Sourced:** References are **propagated** conceptually to the target fragment, although they may be changed (such as using fully qualified names in the target language). See Figure 4.31, page 129.
    - **Multi-Sourced:** A transformation that creates a **single homogenous fragment** of multiple fragments with a referencing relationship. Useful for cases where the referencing fragment augments the transformation of the referenced fragment (e.g. annotation models). The generator for the referenced language can not be simply reused. See Figure 4.32, page 130.
    - **Preprocessing Transformation:** A preprocessor **augments the referenced fragment** with the information from the referencing fragment. The referenced fragment can then be transformed normally (instead of having to define a new multi-sourced transformation). See Figure 4.33, page 130.
  - **Extension:** Transformation by assimilation (the heterogenous fragment consisting of $L_{D}$ and $L_{D-1}$ is compiled to $L_{D-1}$).
    - Sometimes requires **overriding base language transformations** (e.g. operator overloading).
    - Transformations of several different language extensions may **conflict** with each other.
  - **Language Reuse** can be handled in three ways:
    - **Both languages have existing transformations:** The existing transformations are used in conjunction with a generator for the **adapter language.** See Figure 4.35, page 131.
    - The existing transformation for the context fragment is **enhanced** with transformation code for the reused language. See Figure 4.36, page 132.
    - Three independent fragments are generated from the reused, context and adapter language fragments. These are then **weaved together externally.** See Figure 4.37, page 132.
  - **Embedding:** 
    - An embeddable language may come **without a generator.**
      - A generator has to be **implemented** for the specific use-case.
      - This generator will generate **host language code** or the same **target language** as the host language transformation.
    - For an embeddable language **with a suitable generator,** the **adapter language generator** can coordinate the transformations. See Figure 4.38, page 132.





## 4.7 – Concrete Syntax



