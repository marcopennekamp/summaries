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



