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







