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



