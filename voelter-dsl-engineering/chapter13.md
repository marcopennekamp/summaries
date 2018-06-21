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