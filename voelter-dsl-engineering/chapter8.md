# Chapter 8 – Scoping and Linking

- **Linking:** Resolution of name-based references to symbols in parser-based languages. Linking establishes the cross-links in the AST.
  - In **parser-based** languages, cross-references are resolved after the AST has been created from the source code. The textual representation of the program must be sufficient to resolve the reference.
  - In **projectional** languages, every element has an ID, so the language can simply reference the correct ID behind the scenes. References are first established by selecting the correct entry in the code completion menu.
- **Scoping:** Which elements are visible and referentiable by a specific cross-reference?
  - **Scope:** The set of program elements that can be referenced by a specific cross-reference.
  - The scope depends on the **target concept** (function, variable, type, and so on) and its **surroundings** (namespace, location in the program, and so on).
  - A scope has **two uses:**
    - Determines the elements that are shown in **code completion.**
    - A scope can determine the **validity of a reference.**
  - Scopes can be **hierarchical,** structured as a stack of element sets.



