# Chapter 12 – Building Interpreters 

- **Interpreters** read a model, traverse its AST and execute the model according to the execution semantics.
- The **complexity** of an interpreter depends on the complexity of the language.
- **Common interpreter parts:**
  - To evaluate **expressions**, interpreters usually contain a function `eval` that is defined (recursively) for each expression concept.
  - **Statements** are evaluated by a function `execute` that is likewise defined for each statement. Since statements don't produce values, but rather side effects, the implementation will need to apply these side effects.
  - The **environment** keeps track of variable values. 
  - A **call stack** saves the local environments of (nested) function calls.



