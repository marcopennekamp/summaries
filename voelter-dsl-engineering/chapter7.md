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

