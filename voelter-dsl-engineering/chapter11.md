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

