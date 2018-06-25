# Chapter 8 – Scoping and Linking

- **Linking:** Resolution of name-based references to symbols in parser-based languages. Linking establishes the cross-links in the AST.
  - In **parser-based** languages, cross-references are resolved after the AST has been created from the source code. The textual representation of the program must be sufficient to resolve the reference.
  - In **projectional** languages, every element has an ID, so the language can simply reference the correct ID behind the scenes. References are first established by selecting the correct entry in the code completion menu.
- **Scoping:** Which elements are visible and referentiable by a specific cross-reference?
  - **Scope:** The set of program elements that can be referenced by a specific cross-reference.
  - The scope depends on the **target concept** (function, variable, type, and so on) and its **surroundings** (namespace, location in the program, and so on).
  - A scope has **two uses:**
    - Determines the elements that are shown in **code completion.**
    - A scope can determine the **validity of a reference.**
  - Scopes can be **hierarchical,** structured as a stack of element sets.



## 8.2 – Scoping in Xtext

- Xtext provides **Java APIs** for scoping and other aspects of languages.
- Java classes **implementing** an aspect-specific interface are provided to Xtext via dependency injection.

#### 8.2.1 – Simple, Local Scopes

- The **scoping interface** is called `IScopeProvider`, which has a method `getScope` that returns an `IScope`.

  ```java
  public interface IScopeProvider {
      IScope getScope(EObject context, EReference reference);
  }
  ```

  An **`IScope`** is a collection of possible targets for a given reference.

- Xtext supports **declarative scope providers** with the base class `AbstractDeclarativeScopeProvider`.

  - There are **two naming conventions:**

    ```java
    // <X>, <R>: scoping the <R> reference of the <X> concept 
    public IScope scope_<X>_<R>(<X> ctx, EReference ref);
    
    // <X>: the language concept we are looking for as a reference target 
    // <Y>: the concept from under which we try to look for the reference 
    public IScope scope_<X>(<Y> ctx, EReference ref);
    ```

    The first convention is **specific** for a given reference **R** of a given concept **X**. The second convention is invoked when **X** (or a subconcept) is the reference target of a **Y** concept (or a subconcept).

  - **Example implementation:**

    ```java
    public IScope scope_ChangeStateStatement_targetState
                  (ChangeStateStatement ctx, EReference ref) {
        CoolingProgram owningProgram = 
            Utils.ancestor(ctx, CoolingProgram.class);
        return Scopes.scopeFor(owningProgram.getStates()); 
    }
    ```

#### 8.2.2 – Nested Scopes

- In languages with **nested blocks,** simple scoping is usually not sufficient.
- IScopes can reference **outer scopes.** They have the expected shadowing behavior.
- Since scopes are **maps** from a string to an element, where the string is used as the reference text in the code completion menu, we can change the way a reference is called in a given scope. Also see the code on page 232.

#### 8.2.3 – Global Scopes

- References can reference an element from another file using the **emphindex.** This index stores the **qualified name** of the element and an `IEOBjectDescription`, which includes custom data and an **URI** to the actual object (in any file).
- Files and objects are only loaded if they are **needed.**
- The index can be **customized** in two ways:
  - You can use the **`IQualifiedNameProvider`** to return a qualified name for each program element. If the name is `null`, the element is not stored in the index.
  - The **`IDefaultResourceDescriptionStrategy`** can be used to build custom `IEOBjectDescription` objects (to store user data).
- The **`ImportNamespacesAwareGlobalScopeProvider`** is the default **global scope provider** and allows referencing an element either trough its fully qualified name, or through its simple name if it has been imported.



## 8.3 – Scoping in MPS

- **References in MPS** are defined as part of the language structure. An **editor** determines how a referenced element is rendered.

- A **scoping function** determines which elements are the valid targets of a reference.

- **Simple Scopes:**

  - **Transition concept:**

    ```
    concept Transition
      references:
        State target 1
    ```

  - A **search scope constraint** can be used to implement the scope function:

    ```
    link {target}
      referent set handler:
        <none> 
      search scope:
        (referenceNode, linkTarget, enclosingNode, ...) 
                 ->join(ISearchScope | sequence<node<State>>) {
          enclosingNode.ancestor<Statemachine>.states; 
        }
      validator: 
        <default>
      presentation: 
        <none>
    ```

  - A **referent set handler** is executed when a new reference target is set.

  - Customized **presentation** in the code completion menu is also possible.

- **Nested Scopes:** For nested scopes in MPS, you can use the [inherited scopes](https://confluence.jetbrains.com/display/MPSD33/Scopes) feature (the book is outdated here).

- **Polymorphic References:** Each reference concept has their own editor and *their own scope definition.* If two references are possible in the same location, the user simply chooses with the code completion menu.

