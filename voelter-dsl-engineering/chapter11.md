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

