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

