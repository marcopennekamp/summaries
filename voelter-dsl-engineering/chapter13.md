# Chapter 13 – IDE Services

This chapter discusses IDE services that are **not automatically derived** from the language definition.



## 13.1 – Code Completion

- As we have seen, code completion is **influenced by scopes.**

- **Code Completion for a Reference in Xtext:**

  - In the cooling DSL, different kinds of symbols can be referenced. In the code completion menu, we want to **show the symbol kind.**

  - Code completion is customized with a method in a `ProposalProvider`:

    ```java
    public class CoolingLanguageProposalProvider 
            extends AbstractCoolingLanguageProposalProvider {
        @Override
        public void completeAtomic_Symbol(
                EObject model, Assignment assignment,
                ICompletionProposalAcceptor acceptor) {
            CrossReference crossReference =
                ((CrossReference)assignment.getTerminal()); 
            EReference ref = GrammarUtil.getReference(crossReference);
            IScope scope = getScopeProvider().getScope(model, ref);
            Iterable<IEObjectDescription> candidates = 
                scope.getAllElements();
            for (IEObjectDescription od : candidates) {
                String ccText = od.getName() + " (" 
                    + od.getEClass().getName() + ")"; 
                String ccInsert = od.getName().toString();
                acceptor.accept(createCompletionProposal(
                    ccInsert, ccText, null, context
                ));
            }
        }
    }
    ```

    The `model` parameter represents the program element for which the symbol should be completed.

- **An Example with MPS:**

  - The following code customizes the code completion menu for function calls:

    ```
    link {function} 
      ...
      presentation :
        (parameterNode, visible, smartReference, 
           inEditor, ...)->string {
          parameterNode.signatureInfo(); 
        }
    ```

    The method `signatureInfo` creates a string that shows the complete function signature.

  - In contrast to Xtext, we don't have to specify the **inserted text,** because MPS establishes the reference based on the UUID of the target node.

- **Code Completion for Simple Properties:**

  - In **Xtext,** **other kinds of properties** can also be auto-completed. Instead of consulting a scope, which is reserved for references, we have to build a list of proposals.
  - In **MPS,** such behavior is customized in the **editor definition.** Figure 13.2 on page 316 shows how to suggest a variable name in a local variable declaration.

- **Editor Templates:**

  - **Templates** are code snippets that can be selected in the code completion menu.
  - In **Xtext,** templates can be defined as part of the language or by IDE users (see figure 13.3 on page 317).
  - In **MPS,** you can use intentions or cell menus. The latter is shown in figure 13.4 on page 317.



## 13.2 – Syntax Coloring

- Syntax coloring has two purposes: **Syntactic highlighting** (e.g. for keywords) and **semantic highlighting** (calculated based on the AST, e.g. unreachable code).

- **An Example with MPS:**

  - In MPS, syntax coloring is achieved with **style properties.** See figure 13.5 on page 318 for an example.
  - **Semantic highlighting** can be achieved by embedding query expression. See figure 13.6 on page 318.

- **An Example with Xtext:**

  - Syntax coloring in Xtext is performed in **two phases:**

    1. **Define the styles** to be applied to text parts:

       ```java
       public class CLHighlightingConfiguration extends
               DefaultHighlightingConfiguration {
           public static final String VAR = "var";
           
           @Override
           public void configure(
                   IHighlightingConfigurationAcceptor acceptor) {
               super.configure(acceptor);
               acceptor.acceptDefaultHighlighting(
                   VAR, "variables", varTextStyle()
               ); 
           }
       
           private TextStyle varTextStyle() { 
               TextStyle t = defaultTextStyle().copy(); 
               t.setColor(new RGB(100,100,200)); 
               t.setStyle(SWT.ITALIC | SWT.BOLD ); 
               return t;
           } 
       }
       ```

       The user can change the style in their own IDE preferences.

    2. **Associate the style with the program syntax:**

       ```java
       public void provideHighlightingFor(XtextResource resource,
               IHighlightedPositionAcceptor acceptor) {
           EObject root = resource.getContents().get(0);
           TreeIterator<EObject> eAllContents = root.eAllContents();
           while (eAllContents.hasNext()) {
               EObject ref = (EObject) eAllContents.next(); 
               if (ref instanceof SymbolRef) {
                   SymbolDeclaration sym = ((SymbolRef) o)
                       .getSymbol(); 
                   if (sym instanceof Variable) {
                       ICompositeNode n =
                           NodeModelUtils.findActualNodeFor(ref);
                       acceptor.addPosition(
                           n.getOffset(),
                           n.getLength(),
                           CLHighlightingConfiguration.VAR
                       );
                   }
               }
           }
       }
       ```

       We check whether a symbol is a variable, because there are other kinds of symbols we don't want to highlight with the `VAR` highlighting configuration.



## 13.3 – Go-to-Definition and Find References

- A **Go-to-Definition** feature allows the user to follow a reference to its definition.

- Following and finding references **works automatically** in all discussed language workbenches.

- The following examples illustrate how the **default behavior can be changed.**

- **Customizing the Target with Xtext:**

  - "Changing" the go-to behavior means **defining a new hyperlink.** Thus you can provide a hyperlink for an element that is not a reference. You can also have **multiple hyperlinks** for the same element.

  - You have to **implement** the `IHyperlinkHelper` interface with the following method:

    ```java
    public void createHyperlinksTo(XtextResource from, Region region,
            EObject to, IHyperlinkAcceptor acceptor) {
        if (to instanceof TheEConceptIAmInterestedIn) {
            EObject target = // find the target of the hyperlink 
            super.createHyperlinksTo(from, region, target, acceptor);
        } else {
            super.createHyperlinksTo(from, region, to, acceptor);
        }
    }
    ```

- **Customized Finders in MPS:**

  - In mbeddr C, **references to interfaces** can mean sub-interfaces, or a provided or required interface in a component.

  - In this example, we want to distinguish between these usage patterns when **finding usages** of an interface.

  - MPS provides **finders,** which act like filters in the Find Usages dialog (see Figure 13.8 on page 322).

  - A finder **implementation:**

    ```
    finder findProviders for concept Interface 
      description: Providers
      
      find(node, scope)->void {
        nlist<> refs = execute NodeUsages ( node , <same scope> );   
        foreach r in refs.select(it|it.isInstanceOf(ProvidedPort)) {
          add result r.parent ; 
        }
      }
      
      getCategory(node)->string { 
        "Providers";
      }
    ```

    The `execute` call **delegates** to the existing finder `NodeUsages`. The function `getCategory` returns a string that is used to categorize the results (when multiple finders have been used, see Figure 13.9 on page 322).



## 13.4 – Pretty-Printing

- A **pretty printer** creates human-readable code from an AST.

- Pretty printers are useful for automatic **formatting** and also AST modifications (e.g. for quick fixes).

- **Pretty-Printing in MPS:** Since MPS is a projectional editor, pretty-printing does not really exist, since the projection defines how the AST is rendered.

- **Pretty-Printing in Xtext:**

  - The language's **`Formatter`** can be used to pretty-print.

  - **Example:** The following code should be formatted:

    ```
    state Hello : entry { if true { } }
    ```

    This should be the result:

    ```
    state Hello: 
        entry {
            if true { } 
        }
    ```

    We have the following formatter to accomplish this task:

    ```java
    protected void configureFormatting(FormattingConfig c) {
        CoolingLanguageGrammarAccess f =
            (CoolingLanguageGrammarAccess) getGrammarAccess();
        c.setNoSpace().before(f.getCustomStateAccess()
            .getColonKeyword_3());
        c.setIndentationIncrement().after(f.getCustomStateAccess()
            .getColonKeyword_3());
        c.setLinewrap().before(f.getCustomStateAccess()
            .getEntryKeyword_5_0());
        c.setLinewrap().after(f.getCustomStateAccess()
            .getLeftCurlyBracketKeyword_5_1());
        c.setIndentationIncrement().after(f.getCustomStateAccess()
            .getLeftCurlyBracketKeyword_5_1());
        c.setLinewrap().before(f.getCustomStateAccess()
            .getRightCurlyBracketKeyword_5_3());
        c.setIndentationDecrement().before(f.getCustomStateAccess()
            .getRightCurlyBracketKeyword_5_3());
    }
    ```



## 13.5 – Quick Fixes

- **Quick fixes** are automatic fixes for constraint violations, which have to be activated manually by the user.

- **Quick Fixes in Xtext:**

  - Quick fixes can be **implemented** on the concrete syntax or the abstract syntax level. The latter requires a formatting step to update the code in question.

  - **Example:** Consider the following constraint:

    ```java
    public static final String VARIABLE_LOWER_CASE = 
        "VARIABLE_LOWER_CASE";
    
    @Check
    public void checkVariable(Variable v) {
        if (!Character.isLowerCase(v.getName().charAt(0))) {
            warning(
                "Variable name should start with a lower case letter",
                al.getSymbolDeclaration_Name(), 
                VARIABLE_LOWER_CASE
            );
        } 
    }
    ```

    The quick fix is tied to the constant `VARIABLE_LOWER_CASE`. This is referenced by the quick fix:

    ```java
    @Fix(CoolingLanguageJavaValidator.VARIABLE_LOWER_CASE) 
    public void capitalizeName(final Issue issue,
            IssueResolutionAcceptor acceptor) { 
        acceptor.accept(issue, "Decapitalize name", 
            "Decapitalize the name.", "upcase.png", 
            new IModification() {
                public void apply(IModificationContext context) 
                        throws BadLocationException {
                    IXtextDocument xtextDocument =
                        context.getXtextDocument(); 
                    String firstLetter =
                        xtextDocument.get(issue.getOffset(), 1);
                    xtextDocument.replace(
                        issue.getOffset(), 1,
                        firstLetter.toLowerCase()
                    );
                } 
            }
        );
    }
    ```

    The fix above uses the text API to replace the first letter. You can also affect the model on the **AST level,** as mentioned above:

    ```java
    @Fix(CoolingLanguageJavaValidator.VARIABLE_LOWER_CASE)
    public void fixName(final Issue issue, 
            IssueResolutionAcceptor acceptor) {
        acceptor.accept(issue, "Decapitalize name", 
            "Decapitalize the name", "upcase.png", 
            new ISemanticModification() {
                public void apply(EObject element,
                        IModificationContext context) {
                    ((Variable) element).setName(
                        Strings.toFirstLower(issue.getData()[0])
                    );
                } 
            }
        );
    }
    ```

    After the change, the formatter is invoked to serialize the AST back to concrete syntax.

- **Quick Fixes in MPS:**

  - Being a projectional editor, quick fixes in MPS always work on the **abstract syntax.** They can be selected in the intentions menu.

  - **Example:** Take the following constraint:

    ```
    checking rule check_INameAllUpperCase { 
      applicable for concept = INameAllUpperCase as a   
    
      do {
        if (!(a.name.equals(a.name.toUpperCase()))) {
          warning "name should be all upper case" -> a; 
        }
      }
    }
    ```

    The following quick fix uppercases the name and thus fixes the problem of the constraint:

    ```
    quick fix fixAllUpperCase 
    
    arguments:
      node<IIdentifierNamedConcept> node
    
    description(node)->string { 
      "Fix name";
    }
    
    execute(node)->void {
      node.name = node.name.toUpperCase();
    }
    ```

- **Model Synchronization via Quick Fixes:**

  - Quick fixes in MPS can be **executed automatically.**
  - Different model parts can be **synchronized** with this approach. A constraint check detects an inconsistency and the quick fix is automatically applied.
  - **Example:** Class methods that are declared by an interface can have their signature synchronized when the interface changes.



##13.6 – Refactoring

- **Renaming in Xtext** is available without configuration.

- **Introduce Local Variable in MPS:**

  - **Example:** We have the following code:

    ```c
    int8 someFunction(int8 v) {
        int8 y = somethingElse(v * FACTOR); 
        if (v * FACTOR > 20) {
            return 1; 
        } else {
            return 0; 
        }
    }
    ```

    Since `v * FACTOR` is duplicated, we want to refactor it so that it's contained in a local variable:

    ```c
    int8 someFunction(int8 v) {
        int8 product = v * FACTOR;
        int8 y = somethingElse(product); 
        if (product > 20) {
            return 1; 
        } else {
            return 0; 
        }
    }
    ```

    The following code is a bit outdated (MPS has since introduced a better separation between keystrokes and choosers on the one hand, and the refactoring itself on the other), but it should illustrate the general idea. We first **declare** the refactoring:

    ```
    refactoring introduceLocalVariable ( "Introduce Local Variable" )
    
    keystroke: <ctrl+alt>+<V>
    target: node<Expression> 
    allow multiple: false
    
    isApplicableToNode(node)->boolean {
      node.ancestor<Statement>.isNotNull;
    }
    ```

    The constraint that an expression can only be extracted when it is inside a statement ensures that we don't try to introduce a local variable in a global scope. The refactoring also requires a **parameter:**

    ```
    parameters:
      varName chooser: type: string
                       title: Name of the new Variable 
    init(refactoringContext)->boolean {
      return ask for varName; 
    }
    ```

    The `init` block asks the user for the `varName`. Implementation of the **refactoring algorithm:**

    ```
    node<Expression> targetExpr = refactoringContext.node; node<Statement> targetStmt = targetExpr.ancestor<Statement>; 
    int index = targetStmt.index;
    
    // Find all expressions matching the extracted expression.
    nlist<Expression> matchingExpressions = new nlist<Expression>; sequence<node<>> siblings = targetStmt.siblings
        .union(new singleton<node<Statement>>(stmt)); 
    foreach s in siblings {
      if (s.index >= index) {
        foreach e in s.descendants<Expression> {
          if (MatchingUtil.matchNodes(targetExpr, e)) {
            matchingExpressions.add(e);
          }
        }
      }
    }
    
    // Add the local variable before the first expression.
    node<LocalVariableDeclaration> lvd = 
        new node<LocalVariableDeclaration>(); 
    lvd.name = varName;
    lvd.type = targetExpr.type.copy;
    lvd.init = targetExpr.copy;
    targetStmt.add prev-sibling(lvd);
    
    // Replace the matching expressions with our new local variable.
    foreach e in matchingExpressions { 
      node<LocalVarRef> ref = new node<LocalVarRef>(); 
      ref.var = lvd;
      e.replace with(ref);
    }
    ```



## 13.7 – Labels and Icons

- **Labels and icons** are used for language concepts, for example in the outline view and code completion menu.

- **Labels and Icons in Xtext:**

  - Xtext generates a **`LabelProvider`**, in which you can override a `text` method that returns the label text. The `image` method can be overridden to specify the icon.

  - **Examples:**

    ```java
    public class CoolingLanguageLabelProvider 
            extends DefaultEObjectLabelProvider {
        String text(CoolingProgram prg) {
            return "program " + prg.getName();
        }
    
        String image(CoolingProgram prg) { 
            return "program.png";
        }
    
        String text(Variable v) {
            return v.getName() + ": " + v.getType();
        }
    
        String image(Variable v) { 
            return "variable.png";
        } 
    }
    ```

- **Labels and Icons in MPS:**

  - The **`getPresentation`** behavior method can be overridden for a specific concept to define the **label.**
  - The **icon** can be selected in the language concept inspector.



## 13.8 – Outline

- An **outline** is an overview of some model contents, e.g. file contents.

- An outline often **presents the AST** down to a specific level, which has to be configured.

- Often, contents are also **grouped by their language concept.**

- **Customizing the Structure in Xtext:**

  - The **`OutlineTreeProvider`** can be used to customize the outline.

  - **Example:** Organize cooling language file contents by showing all programs and then all tests.

    ```java
    protected void _createChildren(DocumentRootNode parentNode, 
            Model m) { 
        for (EObject prg : m.getCoolingPrograms()) {
            createNode(parentNode, prg); 
        }
        for (EObject t : m.getTests()) {
            createNode(parentNode, t); 
        }
    }
    ```

  - **Example:** Show variables and states in separate sections.

    ```java
    protected void _createChildren(IOutlineNode parentNode,
            CoolingProgram p) { 
        TextOnlyOutlineNode vNode = new TextOnlyOutlineNode(
            parentNode, imageHelper.getImage("variable.png"),
            "variables"
        ); 
        for (EObject v: p.getVariables()) {
            createNode(vNode, v); 
        }
        TextOnlyOutlineNode sNode = new TextOnlyOutlineNode(
            parentNode, imageHelper.getImage("state.png"), 
            "states"
        );
        for (EObject s: p.getStates()) { 
            createNode(sNode, s);
        } 
    }
    ```

    Using the following class:

    ```java
    public class TextOnlyOutlineNode extends AbstractOutlineNode {
        protected TextOnlyOutlineNode(IOutlineNode parent, 
                Image image, Object text) {
            super(parent, image, text, false); 
        }
    }
    ```

- **The Outline in MPS:** The outline in MPS is not customizable. Additional views (*tools*) can be added to implement a whole custom outline view.



## 13.9 – Code Folding

- **Folding in Xtext:**

  - Code folding for concepts that cover **more than one line** is automatically provided.

  - The default behavior can be turned off by implementing **`DefaultFoldingRegionProvider`**:

    ```java
    public class CLFoldingRegionProvider extends
            DefaultFoldingRegionProvider {
        @Override
        protected boolean isHandled(EObject eObject) {
            if (eObject instanceof CustomState) { 
                return false;
            }
            return super.isHandled(eObject); 
        }
    }
    ```

- **Folding in MPS:**

  - Folding can be specified for **collections,** by setting the `uses folding` property to `true` or alternatively a `query`, which dynamically determines whether the collection should be foldable.
  - For foldable collections, we have to provide a **cell** that is rendered when the collection is folded. See Figure 13.13 on page 337.
  - You can also provide **conditonally projected parts,** which are only shown when a detailed view is desired. See Figures 13.14 and 13.15 on page 337 and Figure 13.16 on page 338.



## 13.10 – Tooltips/Hover

- **Tooltips** may, for example, provide information about documentation or the target element of a reference. Toolstips are also called hovers.
- **Xtext:**
  - Tooltips can be **customized** to varying levels. Xtext tooltips consist of a one-line summary and more extensive documentation.
  - **Simple customization:** Keep the structure of a tooltip, but change the text.
  - Override the `getFirstLine` method in `DefaultEObjectHoverProvider` to change the **one-line summary.**
  - Override the `getDocumentation` method in `IEObjectDocumentationProvider` to change the **documentation.**
- **MPS:** MPS does not provide tooltips. You can display additional information in the **inspector** (see Figure 13.17 on page 339 and Figure 13.18 on page 340).



