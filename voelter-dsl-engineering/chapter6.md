# Chapter 6 – Process Issues



## 6.1 – DSL Development

#### 6.1.1 – Requirements for the Language

- Finding the **exact requirements** for the language is hard, because you are trying to understand a **class of problems** instead of just a problem.
- Building **technical DSLs** is about putting already existing knowledge into a language by defining notation, formal language, generators and reasonable defaults.
- When building **business domain DSLs,** it is often possible to find out the existing knowledge from domain experts. 
  - **Experts** can often precisely express their domain knowledge.  
  - **Non-expert artifacts** of the domain can also be exploited: Hardware structures and device features, existing user interfaces, industry standards, training material, or domain ontologies/wikis.
- In the **third (and hardest) case,** because no domain knowledge is directly available, there has to be a **domain analysis** with lots of iteration.

#### 6.1.2 – Iterative Development

- **Iteration** is important in DSL development.
- You iterate by understanding a **small part of the domain** and then develop a functioning language for that part.
- As you keep adding corner cases and exceptions to the language (making it more complex), you should try to **abstract** if possible.
- The language has to approach a **stable state** over time, i.e. language changes should become smaller.

#### 6.1.3 – Co-Evolution of Concepts and Language

- When performing domain analysis, you should **evolve the language** with the concepts.
- Due to its formal nature, language design can **improve your understanding** of the domain concepts.

#### 6.1.4 – Let People Do What They are Good At    

- People on the team should be assigned according to their **strengths.**
- **Target technology experts** can be paramount in arriving at a good implementation of the DSL.
- **Language designers** work with domain experts to define the language, and with platform experts to define code generators or interpreters.

#### 6.1.5 – Domain Users vs. Domain Experts

- **Domain experts** participate in domain analysis and the definition of the DSL.
- **Domain users** use the DSL to create models. They are often not as familiar with the domain as the domain experts.
- Since the users are the **target audience,** you have to make sure that they can actually understand and use the language. Only consulting with domain experts may  lead to a language that is hard to understand for non-expert users.

#### 6.1.7 – Documentation is Still Necessary

- The following things should be **documented:** Structure and syntax, editor usage, generator usage, how and where to write manual code, how to integrate manual code into generated code, platform/framework decisions.
- **Possible media:** Text, screencasts, videos, podcast.
- Nobody reads reference documentations, **example-driven/task-based tutorials** are better.





## 6.2 – Using DSLs

#### 6.2.1 – Reviews

- Regular **model reviews** help with finding mistakes that users make when writing DSL models.
- This is especially important in **early stages** of language usage, when users are still learning.
- Model reviews are **easier** than GPL code reviews, because DSLs are more concise.
- **Recurring mistakes** can be combated with constraints or influence the language design process (if the mistake shows an error in the language design).

#### 6.2.3 – Domain Users Programming?

- In some domains, especially non-formal ones, it might be helpful to **pair a developer and domain expert.**
- Domain experts in such domains are often not able to sufficiently **formalize** their knowledge.
- If domain users are not able to use a language, it might be a **language design problem.**

#### 6.2.4 – DSL Evolution

- **Target Platform Changes:**
  - **New technologies** might have an effect on the target platform choice.
  - *Ideal:* A **new execution engine** for the new platform can be created without changing the language or the models.
  - *Sometimes:* DSLs may make **assumptions** about the target platform, which have to be removed from language and models.
- **Domain Changes:**
  - The language will need to **evolve with the domain.**
  - **Existing models** will need to be evolved with the language.
  - Ways to **keep models functioning:**
    - Make the language **backward compatible,** including **deprecation** when you want to delete something.
    - Provide **quick fixes** for deprecated concepts.
    - **Automatically transform** the old model to a new version, which is only feasible if you have access to all existing models.
- **DSL Tool Changes:**
  - The language has to adapt if the **DSL tool changes.**
  - If the DSL tool is no longer supported, you have to **switch the tool.** Language definitions have to be redone, but models should be **automatically transformable.**





