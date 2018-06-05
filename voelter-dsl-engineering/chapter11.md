# Chapter 11 – Transformation and Generation

- **Model Transformation:** The model is transformed into another model (as an AST).
- **Code Generation:** The model is transformed to text, such as source code or XML.



## 11.1 – Overview of the approaches

- **Classical Code Generation:** The generator traverses the program's AST and outputs text (e.g. source code).
  - Clear distinction between **models,** which are represented by an AST described by some abstract syntax formalism, and the **generated code,** which is simply treated as text.
  - **Template languages** allow relatively comfortable text generation.
- **Classical Model Transformation:** The generator traverses the program's AST and creates an AST for the target language with the appropriate APIs.
  - **Model Transformation Languages** support model navigation and tree construction (of the target AST).
- **Hybrid Approaches** are based on the composability of the template language and the target language.
  - **In MPS,** template code (controlling the transformation) and target language code can be nested inside each other. 



## 11.2 – Xtext Example

- Xtext is **based on EMF,** so any tool that can generate code from EMF models can be used to build generators. These include Acceleo, Jet and Xtend.

#### 11.2.1 – Generator

- This section is guided by the **cooling programs** example.

- Our **generator** (`CoolingLanguageGenerator`) is an Xtend class that implements the `IGenerator` interface. It declares the method `doGenerate`, which is called for each changed model file.

- There are **two approaches** to code generation:

  - Using the **template language** built into Xtend. This is useful when most of the generated code is fixed.
  - A more **functional approach** that can be used for complex structures such as expressions, which are tree structures.

- **Example:** Top-level `compile` function.

  ```
  def compile(CoolingProgram program) { 
      ’’’
      <<FOR appl: program.moduleImports.map(mi|mi.module).filter(typeof( Appliance))>>
          <<FOR c: appl.contents>>
              #define <<c.name>> <<c.index>>
          <<ENDFOR>> 
      <<ENDFOR>>
      // more ...
      ’’’ 
  }
  ```

  Note that the `#define` is not in guillemets, so it belongs to the generated text.

- **Example:** Generating an enum for the cooling states.

  ```
  typedef enum states { 
      null_state,
      <<FOR s : program.concreteStates SEPARATOR ",">> 
          <<s.name>>
      <<ENDFOR>> 
  };
  ```

  The `SEPARATOR` keyword puts a comma between states. The property `concreteStates` is defined as an **extension method:**

  ```
  def concreteStates(CoolingProgram p) {
      p.states.filter(s | !(s instanceof BackgroundState) 
                       && !(s instanceof ShadowState))
  }
  ```

- **Example:** The generator for state transitions (exit actions of the current state, state change, entry actions of the new state).

  ```
  if (new_state != current_state) {
      <<IF program.concreteStatesWitExitActions.size > 0>>
          // execute exit action for state if necessary 
          switch (current_state) {
              <<FOR s: p.concreteStatesWitExitActions>> 
                  case <<s.name>>:
                      <<FOR st: s.exitStatements>>
                          <<st.compileStatement>>
                      <<ENDFOR>>
                      break;
              <<ENDFOR>>
              default:
                  break;
          }
      <<ENDIF>>
      
      // The state change
      current_state = new_state;
      
      // similar as above, but for entry actions
  }
  ```

- **Example:** Statement compilation.

  ```
  class StatementExtensions {
      def dispatch compileStatement(Statement s) {
          // raise error if the overload for the abstract class 
          // is called
      }
  
      def dispatch compileStatement(AssignmentStatement s) {   
          s.left.compileExpr + " = " + s.right.compileExpr +";"
      }
      
      def dispatch compileStatement(IfStatement s) { 
          ’’’
          if (<<s.expr.compileExpr>>) { 
              <<FOR st : s.statements>>
                  <<st.compileStatement>> 
              <<ENDFOR>>
          }<<IF s.elseStatements.size > 0>> else { 
              <<FOR st : s.elseStatements>>
                  <<st.compileStatement>> 
              <<ENDFOR>>
          }<<ENDIF>>
          ’’’ 
      }
      
      // more ... 
  }
  ```

  A `StatementExtensions` object is injected into `CoolingLanguageGenerator`:

  ```
  @Inject extension StatementExtensions
  ```

- **Example:** Expression compilation.

  ```
  def dispatch String compileExpr (Equals e) { 
      e.left.compileExpr + " == " + e.right.compileExpr
  }
  def dispatch String compileExpr (Greater e) { 
      e.left.compileExpr + " > " + e.right.compileExpr
  }
  def dispatch String compileExpr (Plus e) { 
      e.left.compileExpr + " + " + e.right.compileExpr
  }
  def dispatch String compileExpr (NotExpression e) { 
      "!(" + e.expr.compileExpr + ")"
  }
  def dispatch String compileExpr (TrueExpr e) { 
      "TRUE"
  }
  def dispatch String compileExpr (ParenExpr pe) { 
      "(" + pe.expr.compileExpr + ")"
  }
  def dispatch compileExpr (NumberLiteral nl) { 
      nl.value
  }
  ```

#### 11.2.2 – Model-to-Model Transformation

- In the section example, we extend the cooling language with a **"preprocessor" model-to-model transformation.** The preprocessor adds emergency stop behavior to existing state machines.

- We process the model *before* it is compiled:

  ```
  def compile(CoolingProgram program) {
      val transformedProgram = program.transform
      ’’’
      <<FOR appl : transformedProgram.modules.map(m|m.module)
                     .filter(typeof(Appliance))>>
          <<FOR c : appl.contents>>
              #define <<c.name>> <<c.index>> 
          <<ENDFOR>>
      <<ENDFOR>>
      
      // more ...
      ’’’ 
  }
  ```

- The corresponding `transform` function is defined as follows:

  ```
  class Transformation {
      
      @Inject extension CoolingBuilder
      CoolingLanguageFactory factory = CoolingLanguageFactory::eINSTANCE
  
      def CoolingProgram transform(CoolingProgram p ) {
          p.states += emergencyState
          p.events += emergencyEvent
          for (s: p.states.filter(typeof(CustomState))
                          .filter(s|s != emergencyState)) {
              s.events += s.eventHandler [
                  symbolRef [ 
                      emergencyEvent()
                  ]
                  changeStateStatement(emergencyState())
              ]
          }
          return p;
      }
      
      def create result: factory.createCustomState emergencyState() {
          result.name = "EMERGENCY_STOP"
      }
      
      def create result: factory.createCustomEvent emergencyEvent() {
          result.name = "emergency_stop_button_pressed"
      } 
  }
  ```

  The special feature of `create` methods (and the reason to use them) is that they cache their results. That way, we can be sure that we always reference the same emergency state. 

- The square bracket notation uses the **builder** concept, which allows building a model in a tree-like manner. Our builder is defined as follows:

  ```
  class CoolingBuilder {
      CoolingLanguageFactory factory = CoolingLanguageFactory::eINSTANCE
      
      def eventHandler(CustomState it, (EventHandler)=>void handler) {
          val res = factory.createEventHandler
          res
      }
      
      def symbolRef(EventHandler it, (SymbolRef)=>void symref) { 
          val res = factory.createSymbolRef
          it.events += res
      }
      
      def symbol(SymbolRef it, CustomEvent event) { 
          it.symbol = event
      }
      
      def changeStateStatement(EventHandler it, CustomState target) {
          val res = factory.createChangeStateStatement
          it.statements += res
          res.targetState = target
      } 
  }
  ```



## 11.3 – MPS Example

- MPS provides a **textgen** language for text generation at the end of the transformation chain. 

  - The language is not really suitable for generating text from an AST that is **structurally different** from the generated text. In this case, a model-to-model transformation to an intermediate language should be preferred.
  - **Figure 11.2** on page 282 gives an example for a text generator.

- DSLs and language extensions usually use **model-to-model transformations.**

  - **Templates** define the transformation. They are written in the target language and are annotated with macros that act as transformation instructions. 
    - For example, to generate a condition for an `if` statement, we could create a dummy condition `true` and annotate it as such: `COPY_SRC[true] `. Inside the `COPY_SRC` *macro,* we define that the dummy condition `true` should be replaced by the value of `node.guard`, which is the actual condition.
  - **Mapping configurations** define which templates are run when and where.

- **Template-based Translation of the State Machine:**

  - **Example source code:**

    ```
    module Statemachine imports nothing {
      statemachine Counter { 
        in events
          start()
          step(int[0..10] size) 
        out events
          started()
          resetted()
          incremented(int[0..10] newVal)
        local variables
          int[0..10] currentVal = 0 
          int[0..10] LIMIT = 10
        states ( initial = start ) 
          state start {
            on start [ ] -> countState { send started(); } 
          }
          state countState {
            on step [currentVal + size > LIMIT] -> start { 
              send resetted(); 
            }
            on step [currentVal + size <= LIMIT] -> countState {
              currentVal = currentVal + size; 
              send incremented(currentVal);
            }
            on start [ ] -> start { send resetted(); } 
          }
      }
      
      Counter c1; 
      Counter c2;
      
      void aFunction() { 
        trigger(c1, start);
      } 
    }
    ```

  - State machines are **translated** to the following C features:

    - An `enum` for **states.**
    - An `enum` for **events.**
    - A `struct` that holds the **current state** and **local variables.**
    - A function implementing the **behavior** of the state machine. The function takes the struct mentioned above as first argument, called `instance`, and an incoming `event`.

  - The translation is organised into **phases.** Each phase either defines a transformation for an element, or the element is simply copied over.

    - The **first phase** transforms the *core + statemachine* model to a *core* (the mbeddr core language) model. From the state machine, the features mentioned above are generated.
    - The **second phase** generates C source code from the *core* model.

  - **Figure 11.3** on page 283 shows a generator that inserts the enum and struct definitions into the module.

    - The **template fragments** (`<TF ... TF>`) highlight the parts that are executed by the transformation.
    - The `module dummy` is only needed because structs and enums must live in a module.
    - Some information about the **LOOPs** is hidden behind the Inspector (see **Figure 11.5** on page 284).

  - A **property macro** (denoted with a `$`) is used to replace the `name` property of each generated `EnumLiteral` with the name of the actual state or event:

    ```
    node.cEnumLiteralName();
    ```

    `cEnumLiteralName` is a **behavior method:**

    ```
    concept behavior State {
      public string cEnumLiteralName() {
        return this.parent : Statemachine.name + "__state_" 
                             + this.name;
      } 
    }
    ```

  - Figure 11.6 on page 285 shows the expression of a **reference macro** (`->$`) that is used to target the actual enum declaration. Confer Figure 11.3, where the reference macro `->$[statemachineStates]` is placed.

  - **Figure 11.7** on page 286 shows the generator that creates the body of the execution function (not the function itself, so we can embed the body in other concepts).

    - `COPY_SRCL` replaces a node with a list of nodes.

    - **Figure 11.8** on page 287 shows how event arguments, which are mapped to a ` void*` array, are accessed.

    - The `int8 exitActions` statement is just a **dummy that will be replaced** by completely different statements. Don't be confused by that! The real list of statements is created by the following function:

      ```
      (node, genContext, operationContext)->sequence<node<>> { 
        if (node.parent : State.exitAction != null) {
          return node.parent : State.exitAction.statements; 
        }
        new sequence<node<>>(empty); 
      }
      ```

- **Procedural Transformation of a Test Case:**

  - Transformations can be written **manually** with the MPS API, by invoking a **mapping script** from the mapping configuration.

  - **Example:** We will build the following code during the example.

    ```
    module SomeModule imports nothing {
      exported test case testCase1 { }
        
      exported int32 main(int32 argc, string*[] argv) { 
        return test testCase1;
      } 
    }
    ```

    The following script builds the code above:

    ```
    node<ImplementationModule> immo = build ImplementationModule 
      name = #(aNamePassedInFromAUserDialog)
      contents += tc:TestCase
              name = "testCase1"
              type = VoidType
              contents += #(MainFunctionHelper.createMainFunction())
                body = StatementList statements += ReturnStatement
                        expression = ExecuteTestExpression
                                      tests += TestCaseRef 
                                                 testcase -> tc
    ```

