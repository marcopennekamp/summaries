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
  - It is, however, useful to write relevant programs, especially for **regression testing.**



