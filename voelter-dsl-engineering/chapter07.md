# Chapter 7 – Concrete and Abstract Syntax

- The **concrete syntax** is what the user sees.
- The **abstract syntax** is a tree structure with optional cross-references.
- The concrete syntax is like a **user interface,** while the abstract syntax is like an **API** for processing tools.
- Two ways of defining the **relationship between concrete and abstract syntax:**
  - **CS first:** The abstract syntax is derived from the concrete syntax. This is what Xtext does by default.
  - **AS first:** The abstract syntax already stands and a concrete syntax is created, referring to the abstract syntax.
- Two ways in which the abstract syntax and concrete syntax can **relate during usage:**
  - **Parsing:** The AST is constructed from the concrete syntax.
  - **Projection:** The AST is built by editor actions and the concrete syntax is rendered from the AST by projection rules.



## 7.1 – Fundamentals of Free Text Editing and Parsing

- **Manually** writing a parser requires significant experience and effort. This makes sense for **standardized programming languages** with a large user base, because it leads to faster parsing, and better error reporting and error recovery.

- Parsers can also be **generated** (from a **grammar**) with bounded performance guarantees that make them *fast enough.* This reduces the effort of building and changing the concrete syntax by a huge margin.

- **Parsing versus Scanning:**

  - **Scanning:** The input string is separated into a sequence of tokens.
  - **Parsing:** The token sequence is parsed into an AST.
  - **Scanners can be generated** from the grammar as well.
  - Problems of **context-unaware scanners:**
    - **Keywords** can't be used as identifiers (e.g. Java).
    - **Grammar extension** might be problematic. In Java's case, introducing new keywords was problematic.
    - **Grammar composition** may also be problematic, because constituent languages may have incompatible keywords or regular expressions for lexicals.
  - **Context-aware scanning:** The lexer relies on the state of the parser during scanning.
  - **Scannerless parsing:** The parser processes lexicals and keywords with regards to the context.

- **Grammars:**

  - A **grammar** is a set of production rules that define what textual input may be.

  - **Example:** Backus-Naur Form

    ```
    Exp ::= NUM
          | Exp "+" Exp
          | Exp "*" Exp
    ```

  - **Practical challenges with recursion:**

    - **Precedence and associativity** is not easy to encode.
    - Not all parser generators can provide **full support** for recursion.

- **Grammar classes:**

  - **Context-free grammars:** Parsers can map the sentence to the AST only by looking at input symbols.
  - **Context-sensitive grammars:** Parsing depends on the context in which a partial sentence occurs.
  - Typical grammars for DSLs only need **finite lookahead** (e.g. LL(k) or LR(k)-class grammars) and the $k$ is computed automatically.
  - Sometimes, a parser generator **does not support** a grammar class. There are languages for which there is **no grammar** in a particular class. Other languages may have **an intuitive grammar** that can't be used with the parser generator, and instead an equivalent but awkward grammar has to be used.
  - Parser generators can **detect grammar class conformance** and give warnings accordingly.

- **General parsers:** There are parsers that can efficiently parse all context-free grammars, such as GLR and Earley parsers. They are particularly efficient if the grammar is unambiguous. Spoofax uses this approach.

- **Ambiguity:** A grammar is **ambiguous** iff the same sentence can be represented by multiple ASTs.

  - **Ambiguity with Grammar Classes:**

    - LL and LR parsers are **deterministic,** i.e. they can't handle ambiguous grammars.
    - The question whether a given grammar is ambiguous is **undecidable,** but resolving conflict instances may iteratively lead to a (practically) unambiguous grammar.

  - **Ambiguity with Generalized Parsers:** 

    - In the case of ambiguous grammars, the generalized parser returns **all possible ASTs.**

    - **Disambiguation filter:** A way of programmatically selecting one AST. For example, in Spoofax, left-associativity can be indicated with:

      ```
      Exp ::= NUM
            | Exp "+" Exp {left}
            | Exp "*" Exp {left}
      ```

      Precedence can also be indicated:

      ```
      Exp ::= Exp "*" Exp {left} 
      >
      Exp ::= Exp "+" Exp {left}
      ```

- **Grammar Evolution and Composition:**

  - **Grammar composition** is a quick and easy way to create or extend a grammar.
  - An example for a grammar with an **excellent potential for reuse** is an expression grammar.
  - **Challenges with composition:**
    - Completely **new ambiguities** may arise from the composition.
    - Transforming grammars so that they conform to a precedence scheme and a particular grammar class makes them **resistant to composition:** Composing two grammars from a grammar class can lead to a grammar that is not in that class.
    - **Scanner conflicts** can arise from composition. These can be fixed with context-aware scanners or scannerless parsers.



## 7.2 – Fundamentals of Projectional Editing

- In projectional editors, the **AST is modified directly** as the user edits the program.
- A projection engine **renders** the AST accordingly to projection rules. This representation is the basis for user interaction.
- AST objects are created by picking the right concept with **code completion.**
- If two concepts are triggered by the same keyword, the user can simply decide which concept to use and **no ambiguity** is introduced.
- Each AST node has a **UID (unique ID).** Program elements point to each other with this UID.
- Programs are often **stored** in XML.
- **Event handlers** can be defined which react to user gestures and modify the AST accordingly.



## 7.3 – Comparing Parsing and Projection

#### 7.3.1 – Editing Experience

- With free text editing, **any text editor** can be used, but users **expect a modern IDE.**
- With projectional editing, a normal text editor is not sufficient. A **specialized editor** has to render the projections and manipulate the AST.
  - If the notation is **textual-looking,** the editor has to make the experience text-like. MPS does the following things to achieve this: Aliased concepts in code completion, side transforms, smart references, smart delimiters for lists.

#### 7.3.2 – Language Modularity

- Composing grammars in **parser-based systems:**
  - Issues with **grammar classes** and **ambiguity** that have to be solved **invasively.**
  - Better support in systems like Spoofax where a GLR parser allows adding **disambiguation rules** without changing source grammars.
- **In projectional editors:** Language composition is **not a problem** at all.

#### 7.3.3 – Notational Freedom

- **Parser-based languages** are obviously limited to unicode characters. 
- **Projectional editors** support text, tables, fractions, math symbols, graphicals notations, diagrams, and so on.

#### 7.3.4 – Language Evolution

- In projectional editors, **model migrations** have to be written if the structure behind the scenes changes. MPS supports language migrations and quick fixes for deprecated concepts.
- Text-based languages have the advantage, that a model can **always be opened** and migrated manually. However, there is usually no built-in framework for model migrations.

#### 7.3.5 – Infrastructure Integration

- Current software development tools are typically **text-oriented.**
- For projectional languages, **special support for infrastructure integration** is needed (e.g. VCS integration).

#### 7.3.6 – Tool Lock-In

- Of course, **textual languages** have the advantage that they don't lock the user into using an IDE.
- Practically speaking, however, **both approaches** effectively lock the DSL user to an IDE with language support.

#### 7.3.7 – Hidden Information

- In **textual languages,** all information in the fragment must (trivially) be visible.
- In projectional languages, **any kind of information** can be hidden in a particular projection (and visible in another).



## 7.4 – Characteristics of AST Formalisms

- The **Eclipse Modeling Framework (EMF)** is the core of all Eclipse modeling tools. **ECore** is EMF's meta meta model.
- Spoofax uses **ATerm** to represent abstract syntax.
- **MPS** represents the abstract syntax directly by **nodes** that are instances of specific **concepts.** Conceptually very close to ECore.



## 7.5 – Xtext Example

- **Grammar Basics:**

  Example for a **parser rule:**

  ```
  CoolingProgram:
    "cooling" "program" name=ID "{"
      (events+=CustomEvent | variables+=Variable)*
      (initBlock=InitBlock)?
      (states+=State)* 
    "}";
  ```

  ID is a **terminal rule** defined in the **parent grammar:**

  ```
  terminal ID: (’a’..’z’|’A’..’Z’|’_’) 
               (’a’..’z’|’A’..’Z’|’_’|’0’..’9’)*;
  ```

  Xtext/ANTLR grammars also determine the **mapping to the AST** (e.g. via `name=ID`, `initBlock=InitBlock`, `states+=State`).

  **Alternatives** reflected as inheritance in the meta model:

  ```
  State: BackgroundState | StartState | CustomState;
  ```

  If these alternatives **share some properties,** they are pulled up to the State meta class.

- **References:**

  ```
  ChangeStateStatement: "state" targetState=[State];
  ```

  The square brackets indicate that `targetState` is a **reference** to an existing state, instead of a new one. This allows Xtext to support **automatic name resolution.**

  The **concrete syntax of the reference** can also be changed:

  ```
  ChangeStateStatement: "state" targetState=[State|QID];
  QID: ID ("." ID)*;
  ```

  Here, the **vertical bar** does not represent an alternative, but is syntax that indicates the custom syntax.

- **Expressions:**

  ```
  AssignmentStatement: "set" left=Expr "=" right=Expr;
  
  Expr: ComparisonLevel;
  
  ComparisonLevel returns Expression:
    AdditionLevel ((  ({Equals.left=current} "==") |
                      ({LogicalAnd.left=current} "&&") | 
                      ({Smaller.left=current} "<")
                   ) right=AdditionLevel)?;
  
  AdditionLevel returns Expression:
    MultiplicationLevel ((  ({Plus.left=current} "+") |
                            ({Minus.left=current} "-")
                         ) right=MultiplicationLevel)*;
  
  MultiplicationLevel returns Expression: 
    PrefixOpLevel ((  ({Multi.left=current} "*") |
                      ({Div.left=current} "/")
                   ) right=PrefixOpLevel)*;
                   
  PrefixOpLevel returns Expression: 
    ({NotExpression} "!" "(" expr=Expr ")") 
    | AtomicLevel;
    
  AtomicLevel returns Expression:
    ({TrueLiteral} "true") |
    ({FalseLiteral} "false") |
    ({ParenExpr} "(" expr=Expr ")") | 
    ({NumberLiteral} value=DECIMAL_NUMBER) | 
    ({SymbolRef} symbol=[SymbolDeclaration|QID]);
  ```

  Meta classes are only instantiated if a property is assigned. An action like `{TrueLiteral}` **forces the instantiation** of an object.

  The `current` keyword, which always refers to the most recently created object, can be used to assign the AST object itself to a property of another AST object.

  For rules with the `Level`-suffix, no meta classes are created. They are only used to **encode precedence.**



## 7.7 – MPS Example

- **Concept Definition:** First, the **abstract syntax** has to be defined by creating concepts. The syntax is different in the newest MPS, but this illustrates the concept definition nonetheless.

  ```
  concept Statemachine extends BaseConcept 
                       implements IModuleContent
                                  ILocalVarScopeProvider 
                                  IIdentifierNamedElement
    children:
      State   states   0..n
      InEvent inEvents 0..n
      
    references:
      State   initial  1
     
    concept properties:
      alias = statemachine
  ```

  **References** can be used to refer to already existing concepts, i.e. without creating a new instance of the concept.

- **Editor Definition:** Here, we have to define the **projection rules.** An editor is made of **cells** of many different types, which are arranged to achieve the desired syntax structure. See page 214 for example images.

- **Intentions:** Also known as **quick fix.** Intentions are actions available via a menu that transform the model in some way. Many projectional languages have features that are **only accessible through intentions,** because supporting some features with typing is tricky. Intentions are defined for a **specific language concept,** but are optionally available in child nodes.

  ```
  intention addEntryActions for concept State { 
    available in child nodes : true
    
    description(editorContext, node)->string { 
      "Add Entry Action";
    }
  
    isApplicable(editorContext, node)->boolean { 
      node.entryAction.isNull;
    }
  
    execute(editorContext, node)->void { 
      node.entryAction.set new(<default>);   
      editorContext.selectWRTFocusPolicy(node.entryAction);
    }
  }
  ```

- **Side Transforms:** Side transforms can be used to make typing linear text more comfortable.

  ```
  side transform actions makeArithmeticExpression
  
    right transformed node: Expression tag: default_
  
    actions :
      add custom items (output concept: PlusExpression)
        simple item
          matching text
            +
          do transform
            (operationContext, scope, model, 
             sourceNode, pattern)->node< > {
              node<PlusExpression> expr = new node<PlusExpression>();                               
              sourceNode.replace with(expr);
              expr.left = sourceNode;
              expr.right.set new(<default>);
              return expr.right; 
            }
  ```

- **Context Restrictions:** In some cases, child concepts should not be usable in place of a parent concept.

  ```
  concept constraints AssertStatement { 
    can be child
      (operationContext, scope, parentNode, link, 
       childConcept)->boolean { 
        parentNode.ancestor<TestCase, +>.isNotNull;
      } 
  }
  ```

  The constraint in the example above ensures that assert statements can only be used in test case functions, not common functions.

- **Tables and Graphics:** Tables can be rendered with the **table cell** (see page 218 for an example). The **inspector** for the table cell can then be used to define the table structure.

