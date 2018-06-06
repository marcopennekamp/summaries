# Chapter 12 – Building Interpreters 

- **Interpreters** read a model, traverse its AST and execute the model according to the execution semantics.
- The **complexity** of an interpreter depends on the complexity of the language.
- **Common interpreter parts:**
  - To evaluate **expressions**, interpreters usually contain a function `eval` that is defined (recursively) for each expression concept.
  - **Statements** are evaluated by a function `execute` that is likewise defined for each statement. Since statements don't produce values, but rather side effects, the implementation will need to apply these side effects.
  - The **environment** keeps track of variable values. 
  - A **call stack** saves the local environments of (nested) function calls.



## 12.1 – Building an Interpreter with Xtext

- This section describes an interpreter for the **cooling language.**

- The intepreter is **built with Java,** but could be written in any JVM language, including Xtext.

- The following **aspects** have to be handled by the interpreter:

  - Expressions and statements
  - State machine behavior (states, events, transitions)
  - Deferred execution
  - Tests or cooling programs, with mock behavior for hardware elements

- **Expressions and Statements:**

  - Since Java does not support **polymorphic dispatch,** the dispatch has to be simulated:

    ```java
    public abstract class AbstractCoolingLanguageExpressionEvaluator      
                          extends AbstractExpressionEvaluator {
      public AbstractCoolingLanguageExpressionEvaluator(
          ExecutionContext ctx) { super(ctx); }
        
      public Object eval(EObject expr, LogEntry parentLog) 
          throws InterpreterException {
        LogEntry localLog = parentLog.child(LogEntry.Kind.eval, expr, 
                            "evaluating " + expr.eClass().getName());
        if (expr instanceof Equals) {
          return evalEquals((Equals) expr, localLog);
        }
        if (expr instanceof Unequals) {
          return evalUnequals((Unequals) expr, localLog);        
        }
        if (expr instanceof Greater) {
          return evalGreater((Greater) expr, localLog);
        }
        // the others... 
      }
      
      protected Object evalEquals(Equals expr, LogEntry log) 
          throws InterpreterException {
        throw new MethodNotImplementedException(expr, 
          "evalEquals not implemented");
      }
      
      protected Object evalUnequals(Unequals expr, LogEntry log)
          throws InterpreterException {
        throw new MethodNotImplementedException(expr, 
          "evalUnequals not implemented");
      }
    
      // the others... 
    }
    ```

    The specific `eval` methods are supposed to be overridden.

  - **Example:** Number literal evaluation.

    ```java
    protected Object evalNumberLiteral(NumberLiteral expr, +
                                       LogEntry log) { 
        String v = ((NumberLiteral) expr).getValue();
        EObject type = eec().typesystem.typeof(
            expr,
            new TypeCalculationTrace()
        ); 
        if (type instanceof DoubleType) {
            log.child(Kind.debug, expr, "value is a double, " + v);
            return Double.valueOf(v);
        } else if (type instanceof IntType) {
            log.child(Kind.debug, expr, "value is a int, " + v);
            return Integer.valueOf(v); 
        }
        return null; 
    }
    ```

  - **Example:** Logical AND evaluation.

    ```java
    protected Object evalLogicalAnd(LogicalAnd expr, LogEntry log) {
        boolean leftVal = ((Boolean) evalCheckNullLog(
            expr.getLeft(), log)).booleanValue(); 
        if (!leftVal) return false;
        boolean rightVal = ((Boolean) evalCheckNullLog(
            expr.getRight(), log)).booleanValue();
        return rightVal; 
    }
    ```

  - **Example:** Using the environment in an assignment.

    ```java
    protected void executeAssignmentStatement(AssignmentStatement s, 
                                              LogEntry log) {
        Object l = s.getLeft();
        Object r = evalCheckNullLog(s.getRight(), log);
        SymbolRef sr = (SymbolRef) l;
        SymbolDeclaration symbol = sr.getSymbol();
        eec().environment.put(symbol, r);
        log.child(Kind.debug, s, "setting " + symbol.getName() + 
                  " to " + r);
    }
    ```

  - **Example:** Retrieving the value of a symbol (variable).

    ```java
    protected Object evalSymbolRef(SymbolRef expr, LogEntry log) {
        SymbolDeclaration s = expr.getSymbol();
        Object val = eec().environment.get(s);
        if (val == null) {
            EObject type = eec().typesystem.typeof(
                expr, new TypeCalculationTrace());
            Object neutral = intDoubleNeutralValue(type);
            log.child(Kind.debug, expr, "looking up value;" + 
                      " nothing found, using neutral value: " +
                      neutral); 
            return neutral;
        } else {
            log.child(Kind.debug, expr, "looking up value: " + val);
            return val;
        } 
    }
    ```

  - **Example:** Function calls (of another language, because the cooling language does not have function calls). The grammar:

    ```
    FunctionDeclaration returns Symbol:
        {FunctionDeclaration} "function" type=Type name=ID "("
            (params+=Parameter ("," params+=Parameter)* )? ")" "{"
            (statements+=Statement)* 
        "}";
        
    Atomic returns Expression: 
        ...
        {SymbolRef} symbol=[Symbol|QID]
            ("(" (actuals+=Expr)? ("," actuals+=Expr)* ")")?;
    ```

    The `eval` function:

    ```java
    protected Object evalSymbolRef(SymbolRef expr, LogEntry log) {
        Symbol symbol = expr.getSymbol();
        if (symbol instanceof VarDecl) {
            return log(symbol,
              eec().environment.getCheckNull(symbol, log)); 
        }
        if (symbol instanceof FunctionDeclaration) {
            FunctionDeclaration fd = (FunctionDeclaration) symbol;
            return callAndReturnWithPositionalArgs("calling " +
              fd.getName(), fd.getParams(), expr.getActuals(),
              fd.getElements(), RETURN_SYMBOL, log);
        }
        throw new InterpreterException(expr, "interpreter failed;" + 
          " cannot resolve symbol reference " +
          expr.eClass().getName());
    }
    ```

    ```java
    protected Object callAndReturnWithPositionalArgs(String name,
            EList<? extends EObject> formals, 
            EList<? extends EObject> actuals, 
            EList<? extends EObject> bodyStatements) {
        eec().environment.push(name);
        for(int i = 0; i < actuals.size(); i++) {
            EObject actual = actuals.get(i);
            EObject formal = formals.get(i);
            eec().environment.put(formal, 
              evalCheckNullLog(actual, log));
        }
        eec().getExecutor().execute( bodyStatements, log ); 
        Object res = eec().environment.get(RETURN_SYMBOL);
        eec().environment.pop();
        return res;
    }
    ```

- **States, Events and the Main program:**

  - **Example:** Changing a (state machine) state.

    ```java
    protected void executeChangeStateStatement(ChangeStateStatement s,
                                               LogEntry l) {
        engine.enterState(s.getTargetState(), log); 
    }
    
    public void enterState(State targetState, LogEntry logger) throws
            TestFailedException, InterpreterException,
            TestStoppedException { 
        logger.child(Kind.info, targetState, "entering state " + 
                     targetState.getName()); 
        context.currentState = targetState;
        executor.execute(ss.getEntryStatements(), logger);
        throw new NewStateEntered(); 
    }
    ```

    The exception is thrown to interrupt the current step, because the engine is step-based and terminates after a state change.

  - The **main program** of the interpreter is the following `step` function:

    ```java
    public int step(LogEntry logger) { 
        try {
            context.currentStep++;
            executor.execute(
               getCurrentState().getEachTimeStatements(),
               stepLogger
            ); 
            executeAsyncStuff(logger);
            if (!context.eventQueue.isEmpty()) {
                CustomEvent event = context.eventQueue.remove(0); 
                LogEntry evLog = logger.child(Kind.info, null,
                  "processing event from queue: "+event.getName());
                processEventFromQueue(event, evLog);
                return context.currentStep;
            }
            processSignalHandlers(stepLogger); 
        } catch ( NewStateEntered ignore ) { } 
        return context.currentStep;
    }
    
    private void processEventFromQueue(CustomEvent event, 
            LogEntry logger) { 
        for (EventHandler eh : getCurrentState().getEventHandlers()) {
            if (reactsOn(eh, event)) {
                executor.execute(eh.getStatements(), logger);
            } 
        }
    }
    ```



