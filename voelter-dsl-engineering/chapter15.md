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

#### 15.1.3 – Debugging Interpreters and Transformations

- **Debugging an interpreter** can be accomplished by using the debugger of the language the intepreter is written in.

- **Debugging transformations** is non-trivial, because:

  - Transformations are usually written in their own DSL. This requires a **specialized debugger.**
  - **Multi-step transformations** complicate debugging. It should be possible to access intermediate models and trace particular elements through the multiple steps.

- **Xtext:** Debugging transformations and interpreters written in **Xtend** is straight-forward, because they are normal programs. Thus, you can use the **Xtend debugger.**

- **MPS:**

  - Transformation debugging has two ingredients, showing the **mapping partitioning** (i.e. the transformation schedule/order, which also reflects transformation priorities) and **intermediate models.**

  - **Example:** We show the mapping partition for a program.

    ```
    module Simple imports nothing {
      message list messages {
        INFO aMessage() active: something happened
      }
    
      exported int32 main(int32 argc, int8*[] argv) { 
        report (0) messages.aMessage();
        return 0;
      } 
    }
    ```

    Mapping partition:

    ```
    [1] com.mbeddr.core.modules.gen.generator.template.main.removeCommentedCode 
    [2]
    com.mbeddr.core.util.generator.template.main.reportingPrintf
    [3]
    com.mbeddr.core.buildconfig.generator.template.main.desktop com.mbeddr.core.modules.gen.generator.template.main.main
    ```

  - **Example:** An intermediate model after the transformation of the `report` statement.

    ```
    module Simple imports nothing {
      exported int32 main(int32 argc, int8*[] argv) { 
        printf("$$ aMessage: something happened "); 
        printf("@ Simple:main:0#240337946125104144 \n "); 
        return 0;
      } 
    }
    ```



## 15.2 – Debugging DSL Programs

- Debugging on the **target language level** can be useful for the language designer, to find problems in the execution engine, or for programmers that know the target language.
- In other cases, debugging should happen at the **DSL level.**
- This kind of debugging only makes sense for **DSLs with behavior.**
- **Challenges** of building a debugger:
  - Creating the **debugging user interface** (usually provided by the IDE).
  - **Control and inspection** of the debugged program.
    - Easy with **interpreters,** since they can be directly controlled.
    - **Transformations:** Either the execution infrastructure provides debug support or we need to create a debug version of programs that communicate with the debugger. The latter has its limitations and ugliness, so should only be used as a last resort.

#### 15.2.1 – Print Statements – a Poor Man's Debugger

- Since implementing debuggers can be costly, we can also provide a simpler means with **print statements.**

- **Example:** mbeddr `report` statements.

- For non-imperative languages or languages without variables, we can use **inlined** print expressions:

  ```
  Collection[Type] argTypes = 
    aClass.operations.print("operations:")
          .arguments.print("arguments:")
          .type;
  ```

#### 15.2.2 – Automatic Program Tracing

- **Automatic tracing** logs all execution steps in a tree-like data structure.
- **Example:** See Figure 15.6 on page 379.

#### 15.2.3 – Simulation as an Approximation for Debugging

- An interactive interpreter (show variables, events, running tasks; provide a button to single-step; etc.) is essentially a **simulator.** A simulator is fit to be used for debugging, since it covers many of the areas a debugger would.
- Steps to **expand an interpreter** so that it becomes a simulator:
  - Make the **execution controllable** from the simulator (breakpoints, single-step).
  - Implement **inspection** functionality.
  - Allow **changing variable values** from the simulator.

#### 15.2.4 – Automatic Debugging for Xbase-based DSLs

- **Xbase** is Xtext's reusable expression language. A DSL using Xbase defines high-level aspects itself, but uses Xbase for statements and expressions.
- When generating Java code, the DSL AST is **mapped to the Java AST.**
- During this process, DSL code and the corresponding Java code are **automatically linked.**
- The **Java debugger** can use this trace information.