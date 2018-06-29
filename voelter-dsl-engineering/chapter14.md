# Chapter 14 – Testing DSLs



## 14.1 – Syntax Testing

- The syntax can be tested by writing a **large number of relevant programs** and by writing **syntactically incorrect programs,** which then have to be detected for the tests to pass.

- **An Example with Xtext:**

  - Using the **Xtext testing utilities:**

    ```java
    @RunWith(XtextRunner.class) @InjectWith(CoolingLanguageInjectorProvider.class) 
    public class InterpreterTests extends XtextTest {
        @Test
        public void testET0() throws Exception {
            testFileNoSerializer(
                "interpreter/engine0.cool", 
                "tests.appl", 
                "stdparams.cool" 
            );
        } 
    }
    ```

    The `testFileNoSerializer` method loads the `engine0.cool` file (with dependencies on `tests.appl` and `stdparams.cool`), **parses it and checks constraints.**

  - We can also **test partial sentences:**

    ```java
    @Test
    public void testStateParserRule() throws Exception {
        testParserRule("state s:", "CustomState");
        testParserRule("state s: entry { do fach1->anOperation }",
                       "CustomState");
        testParserRule("state s: entry { do fach1->anOperation }", 
                       "State");
    }
    ```

- **Syntax Testing with MPS:**

  - Strict syntax testing in MPS is not useful, because it is a **projectional editor.**
  - It is, however, useful to write relevant programs, especially for **regression testing.** See Figure 14.2 on page 346.



## 14.2 – Constraints Testing

- **An Example with Xtext:**

  - We want to test whether a program **written to make a constraint fail** leads to **all expected/required errors.**

  - **Example:**

    ```java
    @Test
    public void testTypesOfParams() throws Exception {
        testFileNoSerializer(
            "typesystem/tst1.cool", 
            "tests.appl", 
            "stdparams. cool"
        );
        // Make sure that the file does not contain additional errors.
        assertConstraints(issues.sizeIs(3));
        // Check that the Variable named v1 has exactly one error with
        // "incompatible type" in the error message.
        assertConstraints(
            issues.forElement(Variable.class, "v1")
                  .theOneAndOnlyContains("incompatible type") 
        ); 
        // Check that there are exactly two errors anywhere in the 
        // subtree of Variable w1. One of these errors has 
        // "incompatible type" in the error message.
        assertConstraints( 
            issues.under(Variable.class, "w1")
                  .errorsOnly().sizeIs(2)
                  .oneOfThemContains("incompatible type")
        );
    //1 //2 //2 //3 // 3
    }
    ```

- **An Example with MPS:**

  - MPS provides the **`NodesTestCase`** for testing constraints and type system rules.
  - Assertions about a program can be **directly annotated** on program elements. See Figure 14.3 on page 348 for an example. The green parts are the annotations, which wrap around valid statements or expressions.
  - Additionally, developers can create **more complex test cases** (e.g. about structure or type of programs). This corresponds to the `testReference` test method in the example above.
  - Figure 14.4 on page 348 shows a test case that tests **scoping.**



## 14.3 – Semantics Testing

- We **test the execution semantics** of a program by executing it and by making assertions about the results.

- The unit tests that test a DSL program are written in the **target language** of the DSL.

  - There are **DSLs with built-in test support.** In such a case, we don't have to write the test in the target language.

- Testing **interpreters** requires the interpreter to provide a way in which its execution can be inspected by the test code.

- This way, we can also ensure that **an interpreter and a compiler** share the same semantics (especially if we execute a large number of tests).

- **Testing an Interpreter with Xtext:**

  - The **cooling language** allows specifying test cases in the cooling language itself.

  - **Example with test case:**

    ```
    cooling program EngineProgram0 for Einzonengeraet uses stdlib {
        var v: int
        event e1
        init { set v = 1 }
        
        start:
            entry { set v = v * 2 } 
            on e1 { state s2 }
        state s2:
            entry { set v = 0 }
    }
    
    test EngineTest0 for EngineProgram0 {
        assert-currentstate-is ^start 
        assert-value v is 2
        step
        event e1
        step 
        assert-currentstate-is s2 
        assert-value v is 0
    }
    ```

  - Tests can be **run from the IDE.** They can also be **run with JUnit:**

    ```java
    @RunWith(XtextRunner.class) @InjectWith(CoolingLanguageInjectorProvider.class)
    public class InterpreterTests extends PKInterpreterTestCase {
        @Test
        public void testET0() throws Exception {
            testFileNoSerializer(
                "interpreter/engine0.cool", 
                "tests.appl", 
                "stdparams.cool" 
            );
            runAllTestsInFile((Model) getModelRoot()); 
        }
    }
    ```

    Using the method:

    ```java
    protected void runAllTestsInFile(Model m) { 
        CLTypesystem ts = new CLTypesystem(); 
        EList<CoolingTest> tests = m.getTests(); 
        for (CoolingTest test : tests) {
            TestExecutionEngine e = new TestExecutionEngine(test, ts);
            final LogEntry logger = LogEntry.root("test execution");
            LogEntry.setMostRecentRoot(logger);
            e.runTest(logger);
        } 
    }
    ```

  - Additionally, to **ensure that the generated code has the same sematics,** we also run the tests with the generated code.

- **Testing a Generator with MPS:** 

  - **Example:**

    ```
    module UnitTestDemo {
      int32 main(int32 argc, int8*[ ] argv) { 
        return test testMultiply;
      }
      
      test case testMultiply {
         assert (0) times2(21) == 42; 
         assert (1) times2(0) == 0; 
         assert (2) times2(-10) == -20;
      }
    
      int8 times2(int8 a) { 
        return 2 * a;
      } 
    }
    ```

  - **Example:** A test for a state machine.

    ```
    exported test case test1 {
      initsm(c1);
      assert (0) isInState<c1, initialState>; 
      test statemachine c1 {
        start -> countState 
        step(1) -> countState 
        step(2) -> countState 
        step(7) -> countState 
        step(1) -> initialState
      } 
    }
    ```

    The `test statemachine` statement is a special statement that accepts event/state pairs, e.g. `start -> countState`. The test checks, for example, that after the `start` event is triggered, we expect the state machine to be in the state `countState`.

  - **Example:** A test that checks that an interface is correctly used.

    ```
    interface PersistenceProvider { 
      boolean isReady()
      void store(DataPacket* data) 
      void flush()
    }
    ```

    We want to check that a client calls `isReady` first, only calls `store` if `isReady` is true and after calling `store`, a client has to call `flush`.

    To ensure the correct behavior, we have to assess the call order somehow. We can do this by using a **mock** object:

    ```
    exported mock component PersistenceMock { 
      ports:
        provides PersistenceProvider pp 
      expectations:
        total no. of calls: 4 
        sequence {
          0: pp.isReady return false; 
          1: pp.isReady return true; 
          2: pp.store {
            0: parameter data: data != null 
          }
          3: pp.flush 
        }
    }
    ```

    The mock prescribes a sequence of calls that have to be executed. The test:

    ```
    exported test case runTest { 
      client.run();
      validate mock persistenceMock
    }
    ```

- **Structural Testing:**

  - The above testing methodology only works for **programs with behavior.**

  - **Structural tests** are used for DSLs that only specify a structure of some kind.

  - **Approach:**

    1. Write an example model.
    2. Generate the target code.
    3. Inspect the target code.

  - Since inspecting the target code is **brittle,** one should prefer using semantic testing.

  - **Model-to-model transformations** can also benefit from structural testing if we want to test the transformation in isolation (without the generator producing target code). For example, we can check that an emergency stop state is added to a state machine:

    ```
    // run transformation 
    val tp = p.transform
    
    // test result structurally
    val states = tp.states.filter(typeof(CustomState))
    assert(states.filter(s|s.name.equals("EMERGENCY_STOP")).size == 1)
    val emergencyState = 
      states.findFirst(s|s.name.equals("EMERGENCY_STOP"))
    states
      .findFirst(s|s.name.equals("noCooling"))
      .eAllContents
      .filter(typeof(ChangeStateStatement))
      .exists(css|css.targetState == emergencyState)
    ```



## 14.4 – Formal Verification

- Verification checks the **whole program** instead of single (test) cases, but is not perfect (e.g. halting problem).

- Different **algorithms** are used to perform formal verification, including model checking, SAT solving, SMT solving and abstract execution.

- **Model Checking State Machines:**

  - **Model checking** can verify state machines as such:

    - **Functionality** is expressed as a state machine.
    - **Properties** about the behavior of the state machine can be specified, which must be true for every execution of the state machine. For **example:** When you go to state X, the preceding state must be Y.
    - The **model checker** is run with the state machine and the properties as inputs.
    - **Output:** Approval or a counterexample. (Or out of memory.)

  - Naively, the model checker would perform an **exhaustive search.**

    - **State space explosion:** The more complex the state machine, the more possibilites – up to an untenable amount of different states.

  - **Other algorithms** don't need to perform an exhaustive search (e.g. bounded model checking stops after a set amount of steps). 

    - Some inputs are still **too large.** Reformulating the input sometimes helps.

  - Properties can be **elaborate,** often expressed in some **temporal logic.**

    - **Example:** *It is always true that after we have been in state X, we will eventually reach state Y.* (**Fairness;** the state machine does not get stuck in some state.)
    - **Example:** *Whatever the state we are currently in, it is always possible to get to state X.* (**Liveliness;** a state can always be reached.)
    - **Example:** *It is never possible to get to a state X without having gone through state Y before.* (**Safety;** transitioning to a state (x) ensures that other measures (y) have been taken before the state is reached.)

  - Since model checking inputs are sometimes hard to work with, mbeddr C provides a **custom syntax** that is then translated to input data for the NuSMV model checker. Notably, the syntax **abstracts** from the underlying logic.

  - **Example:** We want to test the following counter state machine:

    ```
    verifiable statemachine Counter { 
      in events
        start()
        step(int[0..10] size) 
      local variables
        int[0..100] currentVal = 0
        int[0..100] LIMIT = 10
      states ( initial = initialState )
        state initialState {
          on start [ ] -> countState { }
        }
        state countState {
          on step [currentVal + size > LIMIT] -> initialState { } 
          on step [currentVal + size <= LIMIT] -> countState {
            currentVal = currentVal + size; 
          }
          on start [ ] -> initialState { } 
        }
    }
    ```

    Output for some properties that the model checker checks **by default:**

    ```
    State ’initialState’ can be reached                        SUCCESS
    Variable ’currentVal’ is always between its defined bounds SUCCESS
    State ’countState’ has deterministic transitions           SUCCESS
    Transition 0 of state ’initialState’ is not dead           SUCCESS
    ```

    To **provoke an error,** we change the following code in the model:

    ```
    on step [currentVal + size >= LIMIT] -> initialState { } 
    on step [currentVal + size <= LIMIT] -> countState {
      currentVal = currentVal + size; 
    }
    ```

    It leads to the following message (among others) of the model checker:

    ```
    State ’countState’ contains nondeterministic transitions    FAIL 4
    ```

    The `4` means that there is a **trace** with four steps. You can click on the property to view the trace:

    ```
    State initialState
      LIMIT 10 
      currentVal 0
    State initialState
      in_event: start start()
      LIMIT 10 
      currentVal 0
    State countState
      in_event: step step(10)
      LIMIT 10 
      currentVal 0
    State initialState
      LIMIT 10 
      currentVal 10
    ```

  - **Custom properties:** 

    ```
    verification conditions
      never LIMIT != 10
      always eventually reachable initialState
    ```

    Property 1 ensures that the `LIMIT` "constant" is never changed.

- **SAT/SMT Solving:**

  - **SAT:** Satisfiability of sets of boolean equations (NP-complete).
  - **SMT:** An extension of SAT. Allows additional constructs such as linear arithmetic, arrays or bit-vectors.
  - SMT solving can be used to **verify decision tables** in mbeddr (see Figure 14.7 on page 362 for an example decision table), by checking that all cases are handled.

- **Model Checking and Transformations:** Even though models may pass the model checker on the **DSL-level,** the generator may introduce problems. Approaches:

  - **Manually test the generator.**
  - Some model checking tools **generate test cases.**
  - **Verify the generated code.**

