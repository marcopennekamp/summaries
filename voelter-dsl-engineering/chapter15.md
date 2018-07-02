# Chapter 15 – Debugging DSLs



## 15.1 – Debugging the DSL Definition

This section is concerned with debugging the language definition itself, instead of providing language users with debugging tools.

#### 15.1.1 – Understanding and Debugging the Language Structure

- Text to AST transformation in parser-based tools can be non-trivial, so **debugging the parsing process** might be useful.
- Since **Xtext** uses **ANTLR,** we have to understand and debug the ANTLR parsing process. Xtext can generate a **debug grammar,** which contains no action code. It can be used by ANTLRWorks.
- **MPS** can aid the language developer by allowing him to **inspect any program element** in the Explorer (see Figure 15.1 on page 370). MPS can also show the (projection's) **cell structure.** 

#### 15.1.2 – Debugging Scopes, Constraints and Type Systems

- All aspects of a language in **Xtext** are defined in Java. Thus, the **Java debugger** can be used to debug scopes, constraints and type systems.
- **MPS:**
  - The IDE instance can run an **inner instance** of MPS, which can be debugged by the outer instance.
  - MPS can **analyze stack traces** thrown by Java code (which is generated from the language definitions): You can paste the stack trace into a dialogue, which is transformed into a stack trace that shows the locations in the original language definitions.
  - Pressing Ctrl+Shift+T **shows the type** of a program element. This interface also shows which rules produce a type error.
  - MPS can also show the **type system trace** for a node in the program (see Figure 15.3 on page 372), which shows the state of the solver that tries to solve typing constraints.

