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

#### 15.2.5 – Debuggers for an Extensible Language

- This section describes an **exensible debugger** for mbeddr C.

- **Requirements of the Debugger:** 

  - We want **features** such as breakpoints, single-step execution, watches (display variables, arguments and other state), and stack frames (display call hierarchy).
  - **Language extensions** lead to different kinds of breakpoints, watches, etc. than the base language. Each language extension is thus supported by a **debugger extension.**
  - **Modularity:** Since language extensions are modular, debugger extensions must be as well. New extensions must not require changing base language debugging.
  - **Framework Genericity:** The core debugger infrastructure must be flexible enough so that no changes to it are required when adding an extension.
  - **Simple Debugger Definition:** Language extensions are relatively simple to define, so debugger extensions should be too.
  - **Limited Overhead:** Since mbeddr is used for embedded systems, there has to be a limit on binary size added by the debugger.
  - **Debugger Backend Independence**

- **An Example (Language) Extension:**

  - We add a **`foreach` statement:**

    ```c
    int8 s = 0;
    int8[] a = {1, 2, 3}; 
    foreach (a sized 3) {
        s += it; 
    }
    ```

    For simplicity, the `foreach` can't be nested.

  - The example should be **compiled to:**

    ```c
    int8 s = 0;
    int8[] a = {1, 2, 3};
    for (int __c = 0; __c < 3; __c++) {
        int8 __it = a[__c];
        s += __it; 
    }
    ```

  - The language extension lives in the language module **ForeachLanguage.**

- **Developing the Language Extension:**

  - The new language defines the **ForeachStatement,** which extends C's **Statement.** It is defined as you'd expect (confer Figures 15.8/15.9 on page 383).

  - We add the following **type constraint:**

    ```java
    rule typeof_ForeachStatement for ForeachStatement as fes do {
      typeof(fes.len) :<=: <int64>;
      if (!(fes.array.type.isInstanceOf(ArrayType))) {
        error "array required" -> fes.array; 
      }
    }
    ```

  - An **ItExpression** (`__it`) refers to the value of the iterated element. An ItExpression may only be used inside a `foreach`, which is enforced by the following constraint:

    ```java
    concept constraints ItExpression { 
      can be child
        (context, scope, parentNode, link, childConcept)->boolean {
          parentNode.ancestor<ForeachStatement, +>.isNotNull &&
             parentNode.ancestor<StatementList, +>.isNotNull;
        }
    }
    ```

    It also carries the following **type constraint:**

    ```
    node<Type> basetype = typeof(it.ancestor<ForeachStatement>.array) 
                            : ArrayType.baseType;
    typeof(it) :==: basetype.copy;
    ```

  - Figure 15.10 on page 384 shows the **generator.**

- **Developing the Debug Behavior:**

  - The **debugger specification** is contained in the ForeachLanguage.

  - For **breakpoint** support, the *concept* has to implement the `IBreakpointSupport` marker interface. 

    - For the example (`ForEachStatement`), it is already implemented, because `Statement` already does.

  - **Single-step execution** is supported by the `ISteppable` interface.

    - For the example, we have to **override** the methods that define step over and step into behavior. The step methods are implemented by setting breakpoints at all possible end locations.

    - **Step over:** If the array is empty/iteration is finished, we end up at a statement *after* the `foreach`. Otherwise, we go to the first line of the body.

      ```java
      void contributeStepOverStrategies(list<IDebugStrategy> res) {
        ancestor
        statement list: this.body 
      }
      ```

      This definition is using a DSL included in the mbeddr debugger framework!

    - **Step into:** We instruct the debugger to step into expressions for `array` and `len` (via `IStepIntoable)`:

      ```java
      void contributeStepIntoStrategies(list<IDebugStrategy> res) {   
        subtree: this.array
        subtree: this.len
      }
      ```

    - **Customize watches** with `IWatchProvider`:

      ```java
      void contributeWatchables(list<UnmappedVariable> unmapped, 
          list<IWatchable> mapped) {
        hide "__c"
        map "__it" to "it"
          type: this.array.type : ArrayType.baseType 
          category: WatchableCategories.LOCAL_VARIABLES 
          context: this
      }
      ```

      First, we hide the counter `__c`. Then we map `__it` to the watch variable `it`.

- **Debugger Framework Architecture:**

  - The framework uses a **native C debugger** for the core debugging.
  - **Trace data** is used to link the C code back to the DSL-level.
  - See Figure 15.11 on page 386 for an overview. The **Mapper** is driven by the **Debugger UI** and controls the C debugger. It uses the debug specification of the language.
  - A **step over** is executed in the following way:
    - Ask the current node for **step over strategies,** which are the locations the debugger could end up after the operation. Query trace data for these *end locations* in C code.
    - Set **breakpoints** for the end locations in the C code.
    - **Resume execution** and then **get the call stack.**
    - Query trace data to translate the **stack to the DSL-level.**
    - Get **watchable symbols** and their values.
    - **Create watchables** from symbols and values (via `WatchableProvider`).

- **Debugger Specification:**

  - See the example above!
  - **Stack frames:** Callables on the DSL-level are not necessarily callables on the target language level and vice-versa. Concepts that are callable implement `IStackFrameContributor`.

#### 15.2.6 – What's Missing?

- At the time of writing, **debugging support in language workbenches** is not optimal (especially regarding extensible languages/language composition). The framework described above is not part of MPS itself.
- The support for **multi-step transformation debugging** should be better.

