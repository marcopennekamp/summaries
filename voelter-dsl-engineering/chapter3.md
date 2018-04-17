# Chapter 3 – Conceptual Foundations



## 3.1 – Programs, Languages and Domains

- **Programs:**
  - Let $P$ be the set of **all conceivable programs.**
  - A **program** $p \in P$ is the *conceptual* representation of some *computation*.
  - A **language** $l$ defines a structure and notation for *expressing* such programs from $P$.
  - A program $p \in P$ may have an **expression** in $l$, denoted $p_l$.
  - There can be **several languages** for the same conceptual program $p$. For example, programs $p_{l_1}$ and $p_{l_2}$.
  - A **transformation** $T$ between languages $l_1$ and $l_2$ maps programs $p_{l_1}$ to programs $p_{l_2}$, as such: $T(p_{l_1}) = p_{l_2}$.
  - Let $P_l \subseteq P$ be the set of programs that **can be expressed** in $l$.
- **Domains:**
  - **Inductive/Bottom-up approach:** A domain D is defined as some set of programs with similar characteristics and purpose.
    - We can look at programs and try to identify their common elements, then **build a DSL from that.**
    - **Example:** mbeddr C
  - **Deductive/Top-down approach:** A domain $D$ is a body of knowledge for which some software support has to be provided.
    - This approach is **harder,** because you first have to understand the domain and then identify programs in the domain.
  - $P_D$ is the **set of programs** in a domain $D$.
- **Domain-Specific Languages:**
  - A **domain-specific language** $l_D$ for a domain $D$ is *specialized* for expressing programs from $P_D$.
  - The language is **more efficient** at representing programs from $P_D$.
  - **Challenge:** Constructing a DSL that can express virtually all programs in $P_D$, but not any other programs outside $P_D$.
- **Domain Hierarchy:**
  - We can attempt to **organize domains** in a hierarchy.
  - $D_0$: The domain of **all possible programs** $P$.
  - Domains $D_n$ get **more specialized** as $n$ increases. See **Figure 3.3** on page 61.
  - Languages are **built for a particular domain** ($D_0$: general-purpose languages).





## 3.2 – Model Purpose

- A **model purpose** is the purpose that models expressed in the DSL are intended to fulfill.
- Considering **model purpose** helps us decide which abstractions should be part of a DSL.
- Changes in model purpose may drive **language evolution.**





## 3.3 – The Structure of Programs and Languages

- **Concrete and Abstract Syntax:**
  - **Concrete syntax:** The notation (not necessarily textual) in which programs are expressed.
  - **Abstract syntax:** A data structure that holds the semantically relevant information expressed by a program.
    - More directly, the abstract syntax of a program is usually **tree of program elements.**
    - There may be non-containing **cross-links** between otherwise unrelated nodes.
  - **Parser-based systems** map the concrete syntax to the abstract syntax.
  - User action in **projectional editors** leads to direct changes to the abstract syntax (while the editor shows the concrete syntax as a projection).
- **Fragments:**
  - A **fragment** is a standalone abstract syntax tree.
  - A program may be composed of multiple **fragments.** 
  - $E_f$ is the **set of program elements** in a fragment $f$.
  - The **fragment-of function** $fo : element \to fragment$ maps an element to its containing fragment.
- **Languages:**
  - A **language** $l$ consists of a language concept set $C_l$ and the relationships between concepts.
  - **Concept: ** Any language aspect, including syntax, types and semantics.
  - An **element** $e$ in a fragment $f$ **instantiates** a concept $c \in l$.
  - The **concept-of function** $co : element \to concept$ maps an element to its corresponding language concept.
  - The **language-of function** $lo : concept \to language$ maps a concept to its defining language.
- **Relationships:**
  - $Cdn_f$: The set of **parent-child relationships** in a fragment $f$ with properties $parent$ and $child$.
  - $Refs_f$: The set of **non-containing cross-references** in a fragment $f$ with properties $from$ and $to$.
  - $Inh_l$: The set of **concept inheritance relationships** for a language $l$ with properties $super$ and $sub$.
  - **Liskov Substitution Principle for Language Concepts:** A concept $c_{sub}$ that extends another concept $c_{super}$ can be used in place of $c_{super}$. 
- **Independence:** 
  - **Independent language:** A language that does not depend on other languages.
    - All nodes participating in the relationships defined above are elements of **the same language.** See also **formulas on page 66.**
  - **Independent fragment:** A fragment in which all $r \in Refs_f​$ point to elements within the same fragment.
- **Homogeneity:**
  - **Homogenous fragment:** All elements are expressed with the same language.
  - **Heterogenous fragment:** Some elements are from different languages.





## 3.4 – Parsing vs. Projection

- **Parsing:** 
  - A **parser** is generated from a **grammar**, which defines the concrete syntax.
  - The parser creates an **abstract syntax tree,** which is used for further **analysis and code generation.**
- **Projection:**
  - A **projectional language** is specified by defining the abstract syntax tree.
  - **Projection rules** render the abstract syntax as concrete syntax.
  - **Editing** directly modifies the abstract syntax tree.
  - Programs are **stored** as abstract syntax trees (e.g. XML).

