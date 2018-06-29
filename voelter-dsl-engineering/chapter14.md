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







