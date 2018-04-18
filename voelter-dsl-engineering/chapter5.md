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





