# Chapter 10 – Type Systems

- Type systems are a **syntactic framework** that classify expressions based on the **type** of values they compute. They check whether such expression types conform to **typing rules.**
- This chapter only covers **static type checks.**



## 10.1 – Type Systems Basics

- Type systems should declare **fixed types.** These types are constant and known in advance. For example, integer constants have fixed types.
- Type systems should provide means with which types can be **derived** from the type of other elements.
- A **type hierarchy** should be supported by the type system.
- The type system must be able to perform **type checks** and report errors.
- In the language workbenches discussed in the book, types are **represented by language concepts.**



## 10.2 – Type Calculation Strategies

This section discusses three different methods for **calculating the type** of a program element.

#### 10.2.1 – Recursion

- The **typeof** function is implemented recursively. It returns a type for a program element and calculates this type by recursively calculating the types of elements it depends on.

- **Example:**

  ```
  // Grammar:
  LocalVarDecl: "var" name=ID ":" type=Type ("=" init=Expr)?;
  
  // Typing functions:
  typeof(IntType it) { return it } 
  typeof(DoubleType dt) { return dt }
  
  typeof(LocalVarDecl lvd) {
      // Ensures that the specified variable type is the same as the
      // type of the init expression.
      if isSpecified( lvd.init ) {
          assert typeof(lvd.init) isSameOrSubtypeOf typeof(lvd.type) 
      }
      return typeof( lvd.type ) 
  }
  ```


#### 10.2.2 – Unification

- Language developers specify **type equations** containing type variables and type values.

- **Unification** tries to make all type equations true for each specific type relationship, by substituting type values for the type variables.

- This approach **unifies typing rules and type checks,** so with the same set of equations, we can perform both type checking and type inference.

- **Type errors** are detected when an equation can not be satisfied for a given type relationship.

- **Example equations:**

  ```
  typeof(LocalVarDecl.type) :>=: typeof(LocalVarDecl.init) 
  typeof(LocalVarDecl) :==: typeof(LocalVarDecl.type)
  ```

- **Example checks:**

  ```
  // var i: int = 42
  typeof(int) :>=: typeof(int) // true 
  typeof(T) :==: typeof(int) // T := int
  
  // var i: int = 33.33
  typeof(int) :>=: typeof(double) // error! 
  typeof(T) :==: typeof(int) // T := int
  
  // var i = 42
  typeof(U) :>=: typeof(int) // U := int 
  typeof(T) :==: typeof(U) // T := int
  ```

  The last example illustrates **type inference.**

- Typing equations for **array construction:**

  ```
  typevar T
  foreach (e: init.elements)
      typeof(e) :<=: T
      
  typeof(LocalVarDecl.type) :>=: new ArrayType(T)
  typeof(LocalVarDecl) :==: typeof(LocalVarDecl.type)
  ```

#### 10.2.3 – Pattern Matching

With pattern matching, the **possible type combinations** are listed in a table (see page 259).



## 10.3 – Xtext Example

- Since version 2.0, Xtext supports a type system based on **JVM's type system,** which is limited to JVM-related types.

- This section covers the author's own **Xtext Typesystem Framework,** a third party library.

- **Xtext Typesystem Framework:**

  - Utilizes **recursive** type calculation.

  - An interface **`ITypesystem`** declares a method `typeof(EObject)` that return the type of an element.

  - **Manual integration** with Xtext (for error reporting):

    ```java
    @Inject
    private ITypesystem ts;
    
    @Check(CheckType.NORMAL)
    public void validateTypes(EObject m) {
        ts.checkTypesystemConstraints(m, this);
    }
    ```

  - The **`DefaultTypesystem`** class provides declarative support for common typing strategies.

  - **Example:**

    ```java
    public class CLTypesystem extends DefaultTypesystem {
        private CoolingLanguagePackage cl = 
            CoolingLanguagePackage.eINSTANCE;
        
        @Override
        protected void initialize() {
            useCloneAsType(cl.getIntType());
            ensureFeatureType(cl.getIfStatement(),
                              cl.getIfStatement_Expr(),
                              cl.getBoolType());
        }
        
        public EObject type(NumberLiteral s, 
                            TypeCalculationTrace trace) { 
            if (s.getValue().contains(".")) {
                return create(cl.getDoubleType());
            }
            return create(cl.getIntType()); 
        }
    }
    ```

  - The framework also provides a **textual DSL for typing rules** (see figure 10.2 on page 262). This DSL has the following **advantages:** Concise notation, referential integrity, code completion, static errors instead of runtime errors, easy navigation.

  - The relatively simple type DSL can be supplemented by manually written Java code (**generation gap** pattern).

- **Type System for the Cooling Language:**

  - **Primitive types:**

    ```
    typeof BoolType -> clone 
    typeof IntType -> clone 
    typeof DoubleType -> clone 
    typeof StringType -> clone
    ```

    That is, a primitive type uses a **copy of itself** (the element) as its type.

  - **Fixed types:**

    ```
    typeof StringLiteral -> StringType
    ```

  - **Expression typing:**

    ```
    typeof Expr -> abstract
    ```

    Since `Expr` is abstract, we can provide an abstract typing rule here. This will need to be specialized for the various subclasses.

    ```
    typeof Plus -> common left right {
        ensureType left :<=: IntType, DoubleType 
        ensureType right :<=: IntType, DoubleType
    }
    ```

    The `common` keyword ensures that the type of `Plus` will be the common (super-)type of the left and right type.

    ```
    characteristic COMPARABLE { 
        IntType, DoubleType, BoolType
    }
    typeof Equals -> BoolType {
        ensureType left :<=: char(COMPARABLE) 
        ensureType right :<=: char(COMPARABLE) 
        ensureCompatibility left :<=>: right
    }
    ```

    A **type characteristic** is simply a set of types. The `:<=>:` operator describes that either type must be the subtype of the other type.

    ```
    typeof AssignmentStatement -> none { 
        ensureCompatibility right :<=: left
    }
    ```



## 10.4 – MPS Example

- MPS uses **unification** and **pattern matching** for binary operators.

- **Unification:**

  - **`LocalVariableReference` example:**

    ```
    rule typeof_LocalVariableReference {
      applicable for concept = LocalVariableReference as lvr 
      overrides false
    
      do {
        typeof(lvr) :==: typeof(lvr.variable);
      }
    }
    ```

  - **`NotExpression` example:**

    ```w
    // Ensure the operand is a boolean expression.
    typeof(notExpr.expression) :==: new node<BooleanType>(); 
    // Set the type of the not expression to boolean.
    typeof(notExpr) :==: <boolean>;
    ```

  - **C struct example:**

    ```c
    struct Person { 
        char* name;
        int age; 
    }
    
    int addToAge(Person p, int delta) { 
        return p.age + delta;
    }
    ```

    The `p` parameter and `p.age` expression have to be typed. **Parameter type:**

    ```
    typeof(parameter) :==: typeof(parameter.type);
    ```

    **StructAttributeReference** (concept of `p.age`):

    ```
    concept StructAttributeReference extends Expression 
                                     implements ILValue
      children:
        Expression context 1
        
      references:
        StructAttribute attribute 1
    ```

    **Typing rules for the attribute reference:**

    ```
    // The expression on the left side of the dot must be a struct.
    typeof(structAttrRef.context) :<=: new node<GenericStructType>(); 
    // The type of the expression is the type of the referenced field.
    typeof(structAttrRef) :==: typeof(structAttrRef.attribute);
    ```

- **Pattern Matching:**

  - Using pattern matching supports **language extension,** because new types can be added easily to existing binary operators.

  - **`complex` example:**

    ```
    complex c1 = (1, 2i);
    complex c2 = (3, 5i);
    complex c3 = c1 + c2; // results in (4, 7i)
    ```

    Typing rules are already **extended** for `int` and `double` types:

    ```
    overloaded operations rules binaryOperation
    
    operation concepts: PlusExpression | MinusExpression
    left operand type: <int>
    right operand type: <int>
    operation type: (operation, leftOperandType, 
                     rightOperandType)->node<> {
      <int>; 
    }
    
    operation concepts: PlusExpression | MinusExpression
    left operand type: <double>
    right operand type: <double>
    operation type: (operation, leftOperandType, 
                     rightOperandType)->node<> {
      <double>; 
    }
    ```

    The following typing rule **integrates** the above definitions with regular typing rules:

    ```
    rule typeof_BinaryExpression {
      applicable for concept = BinaryExpression as binex
      
      do {
        node<> optype = operation type(binex, left, right); 
        if (optype != null) {
          typeof(binex) :==: optype; 
        } else {
          error "operator " + be.concept.name + 
                " cannot be applied to " + left.concept.name + 
                "/" + right.concept.name -> be;
        } 
      }
    }
    ```

    We can simply **add another typing rule** for the `complex` type:

    ```
    PlusExpression | MinusExpression one operand type: <complex> operation type:
    (operation, leftOperandType, rightOperandType)->node<> {
      <complex>;
    }
    ```

- The DSL is quite sophisticated, but in **edge cases,** BaseLanguage code may be written as well.

