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



