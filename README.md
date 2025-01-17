# **GPU Accelerated SAT Solver**

## **Team Member**
* Jiarui Wang (jiaruiwa)
* Tejas Badgujar (tbadguja)

[Project Link](https://github.com/JerryCMU/CUDA-Accelerated-SAT-Solver)

[Project Proposal](https://github.com/JerryCMU/CPU-GPU-Parallel-SAT-Solver/blob/main/Project_Proposal.pdf)

[Project Milestone Report](https://github.com/JerryCMU/CPU-GPU-Parallel-SAT-Solver/blob/main/Milestone_Report.pdf)

## **Summary**
We will implement a parallel boolean satisfiability problem solver. After implementing the solver with
thread-level parallelism on CPU and CUDA parallelism on GPU, we will run our program on the GHC
machine (or another machine similar in specs) to measure and compare the performance and speedup of
our algorithms.

## **Background**
As the first problem proven to be NP complete, SAT is undoubtedly one of the most iconic and fundamental
computation problem in computer science. It tightly connects to many problems such as type checking,
automated theorem proving, automated circuit verifying and many other fields.

Boolean satisfiability problem can be summarized into the follow sentence: given a boolean formula, is
there an assignment of variables that will evaluate the boolean formula to true.

To add more detail, a boolean formula is composed of some very basic elements. There are three logical
operators ¬(not), ∨(or) and ∧(and) that connect variables together. These logical operator follows the
following basic rules:
¬(not) rules:
| x | ¬x|        
| - | --|        
|true|false|     
|false|true|

∨(or) rules:
| x | y | x∨y |
| - | - | --- |
|true|true|true|
|true|false|true|
|false|true|true|
|false|false|false|

∧(and) rules:
| x | y | x∧y |
| - | - | --- |
|true|true|true|
|true|false|fa;se|
|false|true|false|
|false|false|false|

Parenthesis could be used to change the order of evaluation just like arithmetic expressions. Since the SAT problem is proven to be NP complete, in the worst case scenario, we would need to evaluate all possible assignment of variables to decide the satisfiability of the boolean expression. The number of possible assignments grows exponentially with the amount of variables since every variable have two possible assignments, true or false.

Note that this exhaustive search across an exponential amount of assignments would benefit greatly from parallelism since the evaluation of each assignment is completely independent from each other and each worker could be assigned with a certain segments of the search and evaluate different assignments themselves. Additionally, other approaches to SAT solving, such as those using DPLL logic and conflict-driven solving, offer speedup compared to the naive implementation and could possibly even further improve from a parallel environment.

Since we could use boolean algebra to convert any arbitrary boolean formula into conjunctive normal form,
our program will only solve SAT problems in conjunctive normal form. If we have time, we could write a
converter that will automatically convert any boolean formula into CNF.

A conjunctive normal form is where a boolean formula is only consist of conjunctive of clauses. A clause
is a disjunction of literals. Literals are either a variable or the negation of a variable.

A conjunctive normal form is where a boolean formula is only consist of conjunctive of clauses. A clause
is a disjunction of literals. Literals are either a variable or the negation of a variable.

**x, ¬x**

Similarly, x, ¬x and y are literals and if we connect them together with ∨ to form a disjunction, we have
a clause:

**x ∨ ¬x ∨ y**

To put everything together, if we connect the clauses together with ∧ to form a conjunction of disjunctions,
we have a boolean formula that is in conjunctive normal form:

**(x ∨ ¬x ∨ y) ∧ (z ∨ ¬y ∨ x) ∧ (¬x ∨ ¬z ∨ ¬y)**

Note that the CNF above is satisfiable with the assignment x = true, y = f alse, z = f alse since if we plug
in the assignment into the CNF, we have:

**(x ∨ ¬x ∨ y) ∧ (z ∨ ¬y ∨ x) ∧ (¬x ∨ ¬z ∨ ¬y)**

**⇐⇒(T ∨ ¬T ∨ F) ∧ (F ∨ ¬F ∨ T) ∧ (¬T ∨ ¬F ∨ ¬F)**

**⇐⇒(T ∨ F ∨ F) ∧ (F ∨ T ∨ T) ∧ (F ∨ T ∨ T)**

**⇐⇒T ∧ T ∧ T**

**⇐⇒T**

## **The Challenge**
**Workload:**

In the naive implementation, the entire problem requires exhaustive search through an exponentiallylarge number of variable assignments. The entire search space is going to be evenly distributed to the workers and every worker should have a local copy of the boolean formula and evaluate variable assignments that are assigned to them.

There is no dependency between the evaluation of variable assignments so there should be minimum communication between workers and it will be a highly computation intensive workload.

Nevertheless, the evaluation of variable assignments will have drastically different workload depending on the assignment. For example, for a CNF problem with one million clauses, some assignment might cause the first clause to be false and the worker don’t have to evaluate further because for a CNF problem to evaluate to true, every single clause in the CNF will have to be true. Meanwhile, some other assignment might fail on the last clause.

In other word, some assignment may only require the worker to go through a very small subset
of the CNF while other assignment may require the worker to go through the entire CNF. This will
make workload balancing very tricky.

There will be high spatial and temporal locality since the worker will refer to the same variable assignment various times during evaluation and the worker is highly likely to evaluate the next adjacent clause in the CNF.

However, after performing some research, there are certain parallel-specific algorithms for SAT solving that we would also like to explore and implement. One such approach is the portfolio approach: instead of splitting the search space amongst processors, it chooses to rather assign each processor the entire search space but have each processor use a different algorithm. Furthermore, with such an approach, we could also look at a mixed approach; with a sufficient amount of cores/nodes, we could group sets of nodes together to all use the same algorithm and also divide up the search space amongst the group. Again, such an approach would not have a lot of communication between processors and would also have high spatial and temporal locality.

Furthermore, we also plan on exploring on whether or not using a GPU may prove beneficial in terms of SAT solving. Our intial assumption may be that the naive divide-and-conquer approach can benefit from the high levels of parallelism a GPU can offer but the size of the problem itself may be too large to fit for a GPU unit. The results of this exploration will be presented in future writeups.

We could also explore the portfolio approach in GPU with each warp group performing a different algorithm and using divide and conquer algorithm within each warp group. But that will also require further research on the GPU architecture and SAT solving algorithms.

**Constraint:**

Note that since GPU has limited local memory per warp block, we probably would not be able to load the entire CNF problem into local block memory. We need to figure out a good way to break up CNF into chunks of clauses.

On the other hand, we need to figure out how to map variable assignment or different algorithms to each CPU or GPU thread. Mapping to different CPU core should be easier since every core has their individual instruction stream. But mapping to different GPU thread will be challenging since CUDA programming achieves its high parallelism with SIMD and the portfolio approach would require different instruction stream since different thread is supposed to use different algorithms.

## **Resources**
We will start from scratch for our implementation. As of currently, we are using Wikipedia as a general reference to the boolean satisfiability problem[1] and SAT solver[2].

We will be using GHC machines to develop and test our code since we have easy access to them and they have good CPU and GPU to test out the performance of our code.

## **Goals and Deliverables**
* 90%
  * Implement a robust naive single-threaded solution on CPU that will use brute force to search through the exponential amount of possible assignments.
  * Implement a thread level parallel solution on CPU that uses divide and conquer to search through the exponential amount of possible assignments.
  * Implement a CUDA parallel solution on GPU that uses divide and conquer to search through the exponential amount of possible assignments.
  * Implement strong sequential, single-core algorithms (DPLL and CPCL algorithms) on CPU for SAT solving
  * Combine DPLL and CPCL together to implement a portfolio based parallel SAT solver on CPU.
* 100%
  * Implement more complex single-threaded SAT solvers and see if we can find a ideal balance between number/type of algorithms, number of cores, and problem size to find the best fit for any SAT solver.
  * Combine the SIMD nature of each warp block and how each warp block could have different instruction streams to create a portfolio and divide and conquer hybrid solution in CUDA.
* 125%
  * Test our algorithm on a set of inputs comparable to that of the ”International SAT Solver Competition” and see how we perform compared to other SAT solvers.

## **Demo Plan**
We will be demonstrating speedup graph of our many parallel algorithms during the demo. If we are able
to achieve linear speedup (good workload balance in divide and conquer) or even super linear speedup (use
of better algorithm), we have done a good job implementing these parallel algorithms.

## **Platform Choice**
We will use the GHC machine to develop and debug the code. Since GHC machine has a high performance CPU (i7-9700 with 8 core count) and a high performance GPU (RTX 2080), it will be great for us to test out the speedup of our CPU and GPU parallel code. Both of the team member also have easy access to the machine which makes testing and developing much easier. However, we may choose to run on a local Windows machine that one of our groupmates has (which has a i7-9700K and a RTX 3080) for ease of access and possibly use the PSC Regular-Memory machines based on the results of our intial exploration.

## **Schedule**
| Week | Items |
| ---- | ----- |
| Week 1 (11/7-11/13)  | Finish writing project proposal and setting up development environment |
| Week 2 (11/14-11/20) | Finished writing uniform 3-SAT problem generator python script for test case generation. Found 3-SAT international benchmarks online for more standardized testing. Finished file I/O structure of the code. Designed and implemented two different boolean assignment representation. |
| Week 3 (11/21-11/27) | Finished basic sequential naive brute force algorithm. Finished basic sequential DPLL algorithm. |
| Week 4 (11/28-12/4)  | Implement basic sequential CDCL algorithm. (Tejas) Implement and benchmark vector based and thread based divide and conquer parallelism on naive brute force algorithm. (Jerry) Start on CUDA implementation of divide and conquer brute force algorithm. (Jerry) |
| Week 5 (12/5-12/11)  | Combining DPLL and CDCL together to create the portfolio algorithm. (Tejas) Finish CUDA implementation of the naive brute force algorithm. (Jerry) Combining DPLL and divide and conquer together to create a divide and conquer DPLL algorithm (Jerry) |
| Week 6 (12/12-12/18) | Prepare Final Presentation |

## **References**
[1] “Boolean Satisfiability Problem.” Wikipedia. Wikimedia Foundation, October 24, 2022. https://en.wikipedia.org/wiki/Boolean_satisfiability_problem.

[2] “SAT Solver.” Wikipedia. Wikimedia Foundation, September 1, 2022. https://en.wikipedia.org/wiki/SAT_solver.

[3] SATLIB - Benchmark Problems. https://www.cs.ubc.ca/~hoos/SATLIB/benchm.html
