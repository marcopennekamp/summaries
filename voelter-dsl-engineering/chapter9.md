# Chapter 9 – Constraints

- **Constraints** are additional restrictions that a given program must satisfy. They belong to the static semantics of a program.

- Constraints are usually **associated** with a language concept.

- **Well-formedness** constraints are constraints like:

  - Names must be unique.
  - A variable must be defined before it is used.

- **Type sytems** consist of rules that verify the correctness of types in programs. These kinds of constraints are treated specially, because type systems can become complicated.

- A **language for defining constraints** should have the following features:

  - **Navigate and filter** the model.
  - **Higher-order functions** for generic algorithms and traversal.
  - **Good collection API** with higher-order function support.
  - **Declaratively** associate a constraint with a language concept.

- **Constraint pseudo-example:**

  ```
  constraint for: Class
  expression:
    this.operations.arguments.type.filter(ComplexNumber).isNotEmpty 
    && !this.imports.any(i|i.name == "ComplexNumberSupportLib")
  message:
    "class " + this.name + " uses complex numbers, " +
    "so the ComplexNumberSupportLib must be imported"
  ```

- Some constraints need a **dataflow graph,** for example dead code detection or reading uninitialized variables.

- **Impact analysis** reduces constraint reevaluation by analysing when a constraint has to be evaluated. This is usually only useful for non-local constraints. Neither Xtext, nor MPS or Spoofax support impact analysis by default.



## 9.1 – Constraints in Xtext

- Constraints are **implemented in Java** or another JVM language. 

- Developers can add constraint methods to a **generated validator class.**

- **Example constraint:**

  ```java
  @Check(CheckType.NORMAL)
  public void checkOrphanEndState(CustomState ctx) {
      CoolingProgram coopro = Utils.ancestor(ctx, CoolingProgram.class);     
      TreeIterator<EObject> all = coopro.eAllContents();
      while (all.hasNext()) {
          EObject s = all.next();
          if (s instanceof ChangeStateStatement) {
              ChangeStateStatement css = (ChangeStateStatement) s;
              if (css.getTargetState() == ctx) return; 
          }
      }
      error("no transition ever leads into this state",
            CoolingLanguagePackage.eINSTANCE.getState_Name());
  }
  ```

  The method name is arbitrary. The method's parameter has the type of the checked element. The constraint above checks that any custom state has at least one incoming transition.

- The following **CheckTypes** are allowed:

  - `CheckType.NORMAL`: The check is run when the file is saved.
  - `CheckType.FAST`: The check is run after each model change (e.g. keypress).
  - `CheckType.EXPENSIVE`: The check is only run if it's explicitly requested through the context menu.

- **Impact analysis** in Xtext is limited to the `CheckType.EXPENSIVE` option.



## 9.2 – Constraints in MPS

#### 9.2.1 – Simple Constraints

- Constraints in MPS (checking rules) are written in **BaseLanguage,** an extended version of Java.

- **Constraint example:**

  ```
  checking rule stateUnreachable { 
    applicable for concept = State as state 
    do {
      if (!state.initial && 
          state.ancestor<Statemachine>.descendants<Transition>.
          where({~it => it.target == state; }).isEmpty) 
      { 
        error "orphan state - can never be reached" -> state;
      }
    }
  }
  ```

  This is equivalent to the unreachable state constraint in the Xtext section.

- Constraints are evaluated based on **change tracking** performed by MPS.

#### 9.2.2 – Dataflow

- The **dataflow graph,** which describes the flow of data through a program's code, is the foundation of dataflow analysis.

- The goal of dataflow analysis is to **detect problems** in a program.

- MPS supports dataflow analysis.

- **Building a Dataflow Graph:**

  - A **dataflow builder** (DFB) builds the dataflow graph for instances of a concept.

    ```
    data flow builder for LocalVariableDeclaration { 
      (node)->void {
        if (node.init != null) { 
          code for node.init 
          write node = node.init
        } else { 
          nop
        }
      }
    }
    ```

    The `code for` statement executes the DFB for the init expression. Then `write` is used to state that the value from the init expression is now in the declared variable. `nop` has to be used to mark the node as visited by the DFB.

  - **Example for AssignmentStatement:**

    ```
    data flow builder for AssigmentStatement { 
      (node)->void {
        code for node.rvalue
        write node.lvalue = node.rvalue 
      }
    }
    ```

  - See page 244 for an **example dataflow graph** for a given piece of code.

  - DFB for an **if-statement:**

    ```
    nop
    code for node.condition
    ifjump after elseIfBlock
    code for node.thenPart
    { jump after node }
    
    label elseIfBlock
    foreach elseIf in node.elseIfs {
      code for elseIf 
    }
    if (node.elsePart != null) { 
      code for node.elsePart
    }
    ```

    `ifjump` indicates that a **jump** to the specified label is allowed. Since the `node.thenPart` may have a return statement, we might not reach `jump after node`, which is why it is marked optional by the curly braces.

    If the condition was false, we jump to the `elseIfBlock` label. If one of the else-if parts is executed, we jump after the whole if-statement.

    DFB for the **else-if:**

    ```
    code for node.condition
    ifjump after node
    code for node.body
    { jump after node.ancestor<IfStatement> }
    ```

  - DFB for a **for-loop:**

    ```
    code for node.iterator 
    label start
    code for node.condition 
    ifjump after node
    code for node.body 
    code for node.incr 
    jump after start
    ```

- **Analyses:**

  - Some dataflow analyses are **supported out of the box.**
  - See page 246 for an **unreachable code** example.



