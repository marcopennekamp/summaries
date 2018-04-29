# Chapter 5 – Fundamental Paradigms



## 5.1 – Structure

- Languages provide **structuring features** to keep large programs manageable.

#### 5.1.1 – Modularization and Visibility

- DSLs often have a **logical unit structure,** e.g. namespaces or modules.
- **Visibility** can be restricted to the same unit, or allow referencing and `public`/`private` symbols.
- Often, **domain concepts** can play the role of the module.

#### 5.1.2 – Partitioning

- **Partitioning** is the breaking down of programs into physical units (e.g. files).
- Physical units **do not have to correspond** to logical modularization.
- **References** to an element should ignore the partition, so that logical elements are not locked into one physical unit.
- Concerns influencing **partition design:**
  - **Change Impact:** Which partitions change due to a model change?
  - **Link Storage:** Where are references stored?
  - **Model Organization:** Using partitions for model organization, for example when the tool does not provide a good logical view layer.
  - **Tool Chain Integration:** Integration with existing, file-based tool chains.
- Partitions should often be **separately processable** for better processing times.
- Affects both real-time and commit-based **team collaboration.**

#### 5.1.3 – Specification vs. Implementation

- **Separating specification from implementation** allows giving different implementations for a specification.
- It has to be ensured that all implementations are **consistent** with the specification. **Semantic checking** can be achieved with pre- and post-conditions, invariants and protocol state machines.

#### 5.1.4 – Specialization

- **Specialization** allows one entity to be a more specific version of another entity.
- In the context of DSLs, specialization is used for **implementing variants** or **evolving a program over time.**
- Implementing inheritance is **not trivial** and comes with problems like the diamond problem and code duplication.

#### 5.1.5 – Types and Instances

- Having **instantiable types** allows defining structures that are parametrized upon instantiation.

#### 5.1.6 – Superposition and Aspects

- **Superposition** is the ability to merge multiple model fragments with a DSL-specific *merge operator.*
- **Aspects** allow the adaptation of program elements that the aspect is pointing to (with a *pointcut operator*).

#### 5.1.7 – Versioning

- **Alternative 1:** Version the model using existing VCS tools or the versioning method of the language workbench.
  - Requires interaction with **complex** version control system.
  - Does not take into account **domain-specific possibilities for versioning.**
- **Alternative 2:** Make versioning part of the language.
  - **Example:** Business rules may be **time dependent,** so you can version them according to dates and apply the right rule to the right date.





## 5.2 – Behavior

- The **behavior** for a particular domain can often be derived from already established **behavioral paradigms.**
- Some DSLs **don't have behavior descriptions:** 
  - **Structure defining DSLs.**
  - DSLs where the behavior is derived from a **set of expectations** that is specified by the programmer declaratively.
- **Advantages** of using established behavioral paradigms:
  - Defining **consistent and correct semantics** is not trivial.
  - A paradigm may come with existing **analysis approaches.**
  - **A generator** may already exist that generates an efficient executable for a platform (e.g. state machines).

#### 5.2.1 – Imperative

- Imperative programs are a **sequence of statements** that can be executed and change the state of the program.
- **Analysis:** Expensive because of aliasing (multiple variables point to the same mutable location) and side effects.
- **Debugging:** Easy.

#### 5.2.2 – Functional

- **Functions** are the core abstraction.
- **Referential Transparency:** The return value of a function only depends on its arguments.
- Purely functional programs are almost useless, because they **can't interact with their environment.** So DSLs often use functional programming for a **calculation core.**
- **Analysis and Optimization:** Easy.
- **Debugging:** Showing intermediate results of function calls.

#### 5.2.3 – Debugging

- Declarative programs specify **what** the program should accomplish, not **how.**
- **Possible elements:** Properties, equations, relationships, constraints.
- **Advantage:** The **evaluation engine** can decide how to evaluate a program, so the evaluation can be evovled over time.
- **Disadvantage:** Finding the solution is often expensive, using trial and error, backtracking or exhaustive search.
- **Debugging:** Hard, because the evaluation engine may be a complex black box.
- **Sub-Paradigms:**
  - **Declarative Concurrent Programming**
  - **Constraint Programming:** Find all values for variables that adhere to some constraints.
  - **Logic Programming:** Facts, rules and queries.

#### 5.2.4 – Reactive/Event-based/Agent

- Behavior is triggered by **received events.**
- Events can be **created** by an entity or the environment.
- **Reactions** mean the creation of subsequent events.
- **Debugging:** Simple, if the timing of events can be controlled. If the timing can not be controlled, simulators may be an option.

#### 5.2.5 – Dataflow

- Variables have **dependencies** expressed by **calculation rules** to each other.
- If a variable $a$ **changes,** all variables depending on $a$ are recalculated.
- **Use Cases:**
  - Spreadsheets
  - Data Flow Diagrams
  - Frameworks like React/VueJS/Angular make use of dataflow programming
- **Execution Modes:**
  - **Approach 1:** All dependent variables are recalculated right away.
  - **Approach 2:** Value changes are seen as messages. Variables are only recalculated if all dependencies of a variable have sent a change message.
  - **Approach 3:** A scheduler determines when calculations should be performed.
- **Debugging:** Easy, because visualization is simple and calculations are always in distinct states.

#### 5.2.6 – State-based

- Describes system behavior as a set of **states** the system can be in, **transitions** from state to state, **events** that trigger transitions and **actions** that are executed on state change.
- **Debugging:** Easy, because we can visualize the state machine.
- **Model checking** can be used to analyse the system.





## 5.3 – Combinations

- **Language composition** can become challenging if the two languages have different behavioral paradigms.
- **Typical combinations in DSLs:**
  - **Dataflow languages** with functional, imperative or declarative calculation rules.
  - **State-based languages** with simple expressions for transition guards and imperative actions.
  - **Reactive Programming:** Dataflow or state-based programs react to events (event-based language).
  - **Structural languages** with functional pre- and post-conditions and a state-based language for protocol state machines.

