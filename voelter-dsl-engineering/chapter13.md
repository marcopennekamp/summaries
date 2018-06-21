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



