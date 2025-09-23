# Langraph

## Challenges in langchain

1. for very complex workflow like if there are loops, conditional statements and jumps in our workflow. so we need glue code(extra python code) to implement it in lang chain, as application progresses, the glue code gets increases and maianting that will become difficult. so for creating non linear complex application in langchain is difficult. 

    ### Solution with langraph
    - so we can implement this by langraph. langraph implement this by using graphs - it treats a task as node(simple python functions) and connect these graphs through edges (connectors).

    - For conditions, in langraph we can use conditional edges, with zero glue code. and langraph will do all the things for you to solve above problem.




2. Handling State.


