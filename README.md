## Code Optimization Methods

*A summary of various code optimization methods*

### Translations

* [Turkish](/tr.md) (by [@umutphp](https://github.com/umutphp))


### Contents

* [General Principles](#general-principles)
* [Low-level](#low-level)
* [Language-dependent optimization](#language-dependent-optimization)
* [Language-independent optimization](#language-independent-optimization)
* [Databases](#databases)
* [Web](#web)
* [References](#references)


__A note on the relation between Computational Complexity of Algorithms and Code Optimization Techniques__

Both computational complexity theory <sup> [2](#r2 "Computational complexity theory, wikipedia") </sup> and code optimization techniques <sup> [1](#r1 "Code optimization, wikipedia") </sup> , have the common goal of efficient problem solution. While they are related to each other and share some concepts, the difference lies in what is emphasized at each time.

Computational complexity theory, studies the performance with respect to input data size. Trying to design algorithmic solutions that have the least / fastest dependence on data size, regardless of underlying architecture. Code optimization techniques, on the other hand, focus on the architecture and the specific constants which enter those computational complexity estimations.

Systems that operate in Real-Time are examples where both factors can be critical (eg. [Real-time Image Processing in JavaScript](https://github.com/foo123/FILTER.js) , yeah i know i am the author :) ).


### General Principles


* __Keep it `DRY` and Cache__ : The general concept of caching involves avoiding re-computation/re-loading of a result if not necessary. This can be seen as a variation of Dont Repeat Yourself principle <sup> [3](#r3 "DRY principle, wikipedia") </sup> . Even dynamic programming can be seen as a variation of caching, in the sense that it stores intermediate results saving re-computation time and resources.

* __`KISS` it, simpler can be faster__  :  Keep it simple <sup> [4](#r4 "KISS principle, wikipedia") </sup> , makes various other techniques easier to apply and modify. ( A plethora of Software Engineering methods can help in this <sup> [34](#r34 "Software development philosophies, wikipedia"), [35](#r35 "97 Things every programmer should know"), [55](#r55 "Stream processing, wikipedia"), [56](#r56 "Dataflow programming, wikipedia") </sup>  )

* __Sosi ample free orginizd, So simple if re-organized__ : Dont hesitate to re-organize if needed. Many times sth can be re-organized, re-structured in a much simpler / faster way while retaining its original functionality (concept of isomorphism <sup> [22](#r22 "Isomorphism, wikipedia") </sup> , change of representation <sup> [23](#r23 "Representation, wikipedia"), [24](#r24 "Symmetry, wikipedia"), [36](#r36 "Data structure, wikipedia") </sup>), yet providing other advantages. For example, the expression `(10+5*2)^2` is the simple constant `400`, another example is the transformation from `infix` expression notation to `prefix` (Polish) notation which can be parsed faster in one pass.

* __Divide into subproblems and Conquer the solution__ : Subproblems (or smaller, trivial, special cases) can be easier/faster to solve and combine for the global solution. Sorting algorithms are great examples of that <sup> [5](#r5 "Divide and conquer algorithm, wikipedia"), [sorting algorithms](https://github.com/foo123/SortingAlgorithms) </sup>

* __More helping hands are always welcome__ : Spread the work load, subdivide, share, parallelize if possible <sup> [6](#r6 "Parallel computation, wikipedia"), [54](#r54 "Heterogeneous computing, wikipedia"), [68](#r68 "A Practical Wait-Free Simulation for Lock-Free Data Structures"), [69](#r69 "A Highly-Efficient Wait-Free Universal Construction"), [70](#r70 "A Methodology for Creating Fast Wait-Free Data Structures") </sup> .

* __United we Stand and Deliver__ : Having data together in a contiguous chunk, instead of scattered around here and there, makes it faster to load and process as a single block, instead of (fetching and) accessing many smaller chunks (eg. cache memory, vector/pipeline machines, database queries) <sup> [51](#r51 "Locality of reference, wikipedia"), [52](#r52 "Memory access pattern, wikipedia"), [53](#r53 "Memory hierarchy, wikipedia") </sup>.

* __A little Laziness never hurt anyone__ : So true, each time a program is executed, only some of its data and functionality are used. Delaying to load and initialize (being lazy) all the data and functionality untill needed, can go a long way <sup> [7](#r7 "Lazy load, wikipedia") </sup> .


__Further Notes__

Before trying to optimize, one has to measure and identify what needs to be optimized, if any. Blind "optimization" can be as good as no optimization at all, if not worse.

That being said, one should always try to optimize and produce efficient solutions. A non-efficient "solution" can be as good as no solution at all, if not worse.

**Pre-optimisation is perfectly valid given pre-knowledge**. For example that instantiating a whole `class` or `array` is slower than just returning an `integer` with the appropriate information.

Some of the optimization techniques can be automated (eg in compilers), while others are better handled manually. 

Some times there is a trade-off between space/time resources. Increasing speed might result in increasing space/memory requirements (__caching__ is a classic example of that). 

The `90-10` (or `80-20` or other variations) rule of thumb, states that __`90` percent of the time__ is spent on __`10` percent of the code__ (eg a loop). Optimizing this part of the code can result in great benefits. (see for example  Knuth <sup> [8](#r8 "An empirical study of Fortran programs") </sup> )

One optimization technique (eg simplification) can lead to the application of another optimization technique (eg constant substitution) and this in turn can lead back to the further application of the first optimization technique (or others). Doors can open.


__References:__ [9](#r9 "Compiler optimizations, wikipedia"), [11](#r11 "Compiler Design Theory"), [12](#r12 "The art of compiler design - Theory and Practice"), [46](#r46 "Optimisation techniques"), [47](#r47 "Notes on C Optimisation"), [48](#r48 "Optimising C++"), [49](#r49 "Programming Optimization"), [50](#r50 "CODE OPTIMIZATION - USER TECHNIQUES")


### Low-level


#### Generalities<sup> [44](#r44 "What Every Programmer Should Know About Floating-Point Arithmetic"), [45](#r45 "What Every Computer Scientist Should Know About Floating-Point Arithmetic") </sup>

__Data Allocation__

* Disk access is slow (Network access is even slower)
* Main Memory access is faster than disk
* CPU Cache Memory (if exists) is faster than main memory
* CPU Registers are fastest


__Binary Formats__

* Double precision arithmetic is slow
* Floating point arithmetic is faster than double precision
* Long integer arithmetic is faster than floating-point
* Short Integer, fixed-point arithmetic is faster than long arithmetic
* Bitwise arithmetic is fastest


__Arithmetic Operations__

* Exponentiation is slow
* Division is faster than exponentiation
* Multiplication is faster than division
* Addition/Subtraction is faster than multiplication
* Bitwise operations are fastest


#### Methods


* __Register allocation__ : Since register memory is fastest way to access heavily used data, it is desirable (eg compilers, real-time systems) to allocate some data in an optimum sense in the cpu registers during a heavy-load operation. There are various algorithms (based on the graph coloring problem) which provide an automated way for this kind of optimization. Other times a programmer can explicitly declare a variable that is allocated in the cpu registers during some part of an operation <sup> [10](#r10 "Register allocation, wikipedia") </sup> 


* __Single Atom Optimizations__ : This involves various operations which optimize one cpu instruction (atom) at a time. For example some operands in an instruction, can be constants, so their values can be replaced instead of the variables. Another example is replacing exponentiation with a power of `2` with a multiplication, etc..


* __Optimizations over a group of Atoms__ : Similar to previous, this kind of optimization, involves examining the control flow over a group of cpu instructions and re-arranging so that the functionality is retained, while using simpler/fewer instructions. For example a complex `IF THEN` logic, depending on parameters, can be simplified to a single `Jump` statement, and so on.


### Language-dependent optimization

* Check carefuly the **documentation and manual** for the underlying mechanisms the language is using to implement specific features and operations and use them to estimate the cost of a certain code and the alternatives provided.



### Language-independent optimization


* __Re-arranging Expressions__ : More efficient code for the evaluation of an expression (or the computation of a process) can often be produced if the operations occuring in the expression are evaluated in a different order. This works because by re-arranging expression/operations, what gets added or multiplied to what, gets changed, including the relative number of additions and multiplications, and thus the (overall) relative (computational) costs of each operation. In fact, this is not restricted to arithmetic operations, but any operations whatsoever using symmetries (eg commutative laws, associative laws and distributive laws, when they indeed hold, are actualy examples of arithmetic operator symmetries) of the process/operators and re-arrange to produce same result while having other advantages. That is it, so simple. Classic examples are Horner's Rule <sup> [13](#r13 "Horner rule, wikipedia") </sup>,  Karatsuba Multiplication <sup> [14](#r14 "Karatsuba algorithm, wikipedia") </sup>,  fast complex multiplication <sup> [15](#r15 "Fast multiplication of complex numbers") </sup>, fast matrix multiplication <sup> [18](#r18 "Strassen algorithm, wikipedia"),  [19](#r19 "Coppersmith-Winograd algorithm, wikipedia") </sup>,  fast exponentiation <sup> [16](#r16 "Exponentiation by squaring, wikipedia"),  [17](#r17 "Fast Exponentiation") </sup>, fast gcd computation <sup> [78](#r78 "A Binary Recursive Gcd Algorithm") </sup>, fast factorials/binomials <sup> [20](#r20 "Comments on Factorial Programs"), [21](#r21 "Fast Factorial Functions") </sup>, fast fourier transform <sup> [57](#r57 "Fast Fourier transform, wikipedia") </sup>, fast fibonacci numbers <sup> [76](#r76 "Fast Fibonacci numbers") </sup>, sorting by merging <sup> [25](#r25 "Merge sort, wikipedia") </sup>, sorting by powers <sup> [26](#r26 "Radix sort, wikipedia") </sup>.


* __Constant Substitution/Propagation__ : Many times an expression is under all cases evaluated to a single constant, the constant value can be replaced instead of the more complex and slower expression (sometimes compilers do that).


* __Inline Function/Routine Calls__ : Calling a function or routine, involves many operations from the part of the cpu, it has to push onto the stack the current program state and branch to another location, and then do the reverse procedure. This can be slow when used inside heavy-load operations, inlining the function body can be much faster without all this overhead. Sometimes compilers do that, other times a programmer can declare or annotate a function as `inline` explicitly. <sup> [27](#r27 "Function inlining, wikipedia") </sup> 


* __Combining Flow Transfers__ : `IF/THEN` instructions and logic are, in essence, cpu `branch` instructions. Branch instructions involve changing the program `pointer` and going to a new location. This can be slower if many `jump` instructions are used. However re-arranging the `IF/THEN` statements (factorizing common code, using De Morgan's rules for logic simplification etc..) can result in *isomorphic* functionality with fewer and more efficient logic and as a result fewer and more efficient `branch` instructions


* __Dead Code Elimination__ : Most times compilers can identify code that is never accessed and remove it from the compiled program. However not all cases can be identified. Using previous simplification schemes, the programmer can more easily identify "dead code" (never accessed) and remove it. An alternative approach to "dead-code elimination" is "live-code inclusion" or "tree-shaking" techniques.


* __Common Subexpressions__ : This optimization involves identifying subexpressions which are common in various parts of the code and evaluating them only once and use the value in all subsequent places (sometimes compilers do that).


* __Common Code Factorisation__ : Many times the same block of code is present in different branches, for example the program has to do some common functionality and then something else depending on some parameter. This common code can be factored out of the branches and thus eliminate unneeded redundancy , latency and size.


* __Strength Reduction__ : This involves transforming an operation (eg an expression) into an equivalent one which is faster. Common cases involve replacing `exponentiation` with `multiplication` and `multiplication` with `addition` (eg inside a loop). This technique can result in great efficiency stemming from the fact that simpler but equivalent operations are several cpu cycles faster (usually implemented in hardware) than their more complex equivalents (usually implemented in software) <sup> [28](#r28 "Strength reduction, wikipedia") </sup> 


* __Handling Trivial/Special Cases__ : Sometimes a complex computation has some trivial or special cases which can be handled much more efficiently by a reduced/simplified version of the computation (eg computing `a^b`, can handle the special cases for `a,b=0,1,2` by a simpler method). Trivial cases occur with some frequency in applications, so simplified special case code can be quite useful.  <sup> [42](#r42 "Three optimization tips for C"), [43](#r43 "Three optimization tips for C, slides") </sup> . Similar to this, is the handling of common/frequent computations (depending on application) with fine-tuned or faster code or even hardcoding results directly.


* __Exploiting Mathematical Theorems/Relations__ : Some times a computation can be performed in an equivalent but more efficient way by using some mathematical theorem, transformation, symmetry <sup> [24](#r24 "Symmetry, wikipedia") </sup> or knowledge (eg. Gauss method of solving Systems of Linear equations <sup> [58](#r58 "Gaussian elimination, wikipedia") </sup>, Euclidean Algorithm <sup> [71](#r71 "Euclidean Algorithm") </sup>, or both <sup> [72](#r72 "Gröbner basis") </sup>, Fast Fourier Transforms <sup> [57](#r57 "Fast Fourier transform, wikipedia") </sup>, Fermat's Little Theorem <sup> [59](#r59 "Fermat's little theorem, wikipedia") </sup>,  Taylor-Mclaurin Series Expasions, Trigonometric Identities <sup> [60](#r60 "Trigonometric identities, wikipedia") </sup>, Newton's Method <sup> [73](#r73 "Newton's Method"),[74](#r74 "Fast Inverse Square Root") </sup>, etc..<sup> [75](#r75 "Methods of computing square roots") </sup>). This can go a long way. It is good to refresh your mathematical knowledge every now and then.


* __Using Efficient Data Structures__ : Data structures are the counterpart of algorithms (in the space domain), each efficient algorithm needs an associated efficient data structure for the specific task. In many cases using an appropriate data structure (representation) can make all the difference (eg. database designers and search engine developers know this very well) <sup> [36](#r36 "Data structure, wikipedia"), [37](#r37 "List of data structures, wikipedia"), [23](#r23 "Representation, wikipedia"), [62](#r62 "Dancing links algorithm"), [63](#r63 "Data Interface + Algorithms = Efficient Programs"), [64](#r64 "Systems Should Automatically Specialize Code and Data"), [65](#r65 "New Paradigms in Data Structure Design: Word-Level Parallelism and Self-Adjustment"), [68](#r68 "A Practical Wait-Free Simulation for Lock-Free Data Structures"), [69](#r69 "A Highly-Efficient Wait-Free Universal Construction"), [70](#r70 "A Methodology for Creating Fast Wait-Free Data Structures"), [77](#r77 "Fast k-Nearest Neighbors (k-NN) algorithm") </sup> 



__Loop Optimizations__

Perhaps the most important code optimization techniques are the ones involving loops.


* __Code Motion / Loop Invariants__ : Sometimes code inside a loop is independent of the loop index, can be moved out of the loop and computed only once (it is a loop invariant). This results in the loop doing fewer operations (sometimes compilers do that) <sup> [29](#r29 "Loop invariant, wikipedia"), [30](#r30 "Loop-invariant code motion, wikipedia") </sup> 

__example:__

```javascript

// this can be transformed
for (i=0; i<1000; i++)
{
    invariant = 100*b[0]+15; // this is a loop invariant, not depending on the loop index etc..
    a[i] = invariant+10*i;
}

// into this
invariant = 100*b[0]+15; // now this is out of the loop
for (i=0; i<1000; i++)
{
    a[i] = invariant+10*i; // loop executes fewer operations now
}

```


* __Loop Fusion__ : Sometimes two or more loops can be combined into a single loop, thus reducing the number of test and increment instructions executed.

__example:__

```javascript

// 2 loops here
for (i=0; i<1000; i++)
{
    a[i] = i;
}
for (i=0; i<1000; i++)
{
    b[i] = i+5;
}


// one fused loop here
for (i=0; i<1000; i++)
{
    a[i] = i;
    b[i] = i+5;
}

```


* __Unswitching__ : Some times a loop can be split into two or more loops, of which only one needs be executed at any time.

__example:__

```javascript

// either one of the cases will be executing in each time
for (i=0; i<1000; i++)
{
    if (X>Y)  // this is executed every time inside the loop
        a[i] = i;
    else
        b[i] = i+10;
}

// loop split in two here
if (X>Y)  // now executed only once
{
    for (i=0; i<1000; i++)
    {
        a[i] = i;
    }
}
else
{
    for (i=0; i<1000; i++)
    {
        b[i] = i+10;
    }
}

```


* __Array Linearization__ : This involves handling a multi-dimensional array in a loop, as if it was a (simpler) one-dimensional array. Most times multi-dimensional arrays (eg `2D` arrays `NxM`) use a linearization scheme, when stored in memory. Same scheme can be used to access the array data as if it is one big `1`-dimensional array. This results in using a single loop instead of multiple nested loops <sup> [31](#r31 "Array linearisation, wikipedia"), [32](#r32 "Vectorization, wikipedia"), [61](#r61 "The NumPy array: a structure for efficient numerical computation") </sup> 

__example:__

```javascript

// nested loop
// N = M = 20
// total size = NxM = 400
for (i=0; i<20; i+=1)
{
    for (j=0; j<20; j+=1)
    {
        // usually a[i, j] means a[i + j*N] or some other equivalent indexing scheme, 
        // in most cases linearization is straight-forward
        a[i, j] = 0;
    }
}

// array linearized single loop
for (i=0; i<400; i++)
    a[i] = 0;  // equivalent to previous with just a single loop
    
    
```


* __Loop Unrolling__ : Loop unrolling involves reducing the number of executions of a loop by performing the computations corresponding to two (or more) loop iterations in a single loop iteration. This is partial loop unrolling, full loop unrolling involves eliminating the loop completely and doing all the iterations explicitly in the code (for example for small loops where the number of iterations is fixed). Loop unrolling results in the loop (and as a consequence all the overhead associated with each loop iteration) executing fewer times. In processors which allow pipelining or parallel computations, loop unroling can have an additional benefit, the next unrolled iteration can start while the previous unrolled iteration is being computed or loaded without waiting to finish. Thus loop speed can increase even more <sup> [33](#r33 "Loop unrolling, wikipedia") </sup> 

__example:__

```javascript

// "rolled" usual loop
for (i=0; i<1000; i++)
{
    a[i] = b[i]*c[i];
}    
    
// partially unrolled loop (half iterations)
for (i=0; i<1000; i+=2)
{
    a[i] = b[i]*c[i];
    // unroled the next iteration into the current one and increased the loop iteration step to 2
    a[i+1] = b[i+1]*c[i+1];
}

// sometimes special care is needed to handle cases
// where the number of iterations is NOT an exact multiple of the number of unrolled steps
// this can be solved by adding the remaining iterations explicitly in the code, after or before the main unrolled loop

```


### Databases

#### Generalities

Database Access can be expensive, this means it is usually better to fetch the needed data using as few DB connections and calls as possible


#### Methods


* __Lazy Load__ : Avoiding the DB access unless necessary can be efficient, provided that during the application life-cycle there is a frequency of cases where the extra data are not needed or requested


* __Caching__ : Re-using previous fetched data-results for same query, if not critical and if a slight-delayed update is tolerable


* __Using Efficient Queries__ : For Relational DBs, the most efficient query is by using an index (or a set of indexes) by which data are uniquely indexed in the DB <sup> [66](#r66 "10 tips for optimising Mysql queries"), [67](#r67 "Mysql Optimisation") </sup>.  


* __Exploiting Redundancy__ : Adding more helping hands(DBs) to handle the load instead of just one. In effect this means copying (creating redundancy) of data in multiple places, which can subdivide the total load and handle it independantly


### Web

* __Minimal Transactions__ : Data over the internet (and generally data over a network), take some time to be transmitted. More so if the data are large, therefore it is best to transmit only the necessary data, and even these in a compact form. That is one reason why `JSON` replaced the verbose `XML` for encoding of arbitrary data on the web.


* __Minimum Number of Requests__ : This can be seen as a variaton of the previous principle. It means that not only each request should transmit only necessary data in a compact form, but also that the number of requests should be minimized. This can include, minifying `.css` files into one `.css` file (even embedding images if needed), minifying `.js` files into one `.js` file, etc.. This can sometimes generate large data (files), however coupled with the next tip, can result in better performance.


* __Cache, cache and then cache some more__ : This can include everything, from whole pages to `.css` files, `.js` files, images etc.. Cache in the server, cache in the client, cache in-between, cache everywhere..


* __Exploiting Redundancy__ : For web applications, this is usually implemented by exploiting some [cloud architecture](http://en.wikipedia.org/wiki/Cloud_computing) in order to store (static) files, which can be loaded (through the cloud) from more than one location. Other approaches include, [Load balancing](http://en.wikipedia.org/wiki/Load_balancing_%28computing%29) ( having redundancy not only for static files, but also for servers ).


* __Make application code faster/lighter__ : This draws from the previous principles about code optimization in general. Efficient application code can save both server and user resources. There is a reason why Facebook created `HipHop VM` ..


* __Minimalism is an art form__ : Having web pages and applications with tons of html, images, (not to mention a ton of advertisements) etc, etc.. is not necessarily better design, and of course makes page load time slower. Therefore having minimal pages and doing updates in small chunks using `AJAX` and `JSON` (that is what `web 2.0` was all about), instead of reloading a whole page each time, can go a long way. This is one reason why [Template Engines](http://en.wikipedia.org/wiki/Template_engine) and [MVC Frameworks](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) were created. Minimalism does not need to sacrifice the artistic dimension, [Minimalism](http://en.wikipedia.org/wiki/Minimalism) __IS__ an art form.

[το λακωνίζειν εστί φιλοσοφείν (i.e to be laconic in speech and deeds, simple and to the point, is the art of philosophy)](https://en.wikipedia.org/wiki/Laconic_phrase)


[![Zen Circle](/zen-circle.jpg)](http://en.wikipedia.org/wiki/Ens%C5%8D)



### References

1. <a id="r1" href="http://en.wikipedia.org/wiki/Code_optimization">Code optimization, wikipedia</a>
2. <a id="r2" href="http://en.wikipedia.org/wiki/Computational_complexity_theory">Computational complexity theory, wikipedia</a>
3. <a id="r3" href="http://en.wikipedia.org/wiki/Don%27t_repeat_yourself">DRY principle, wikipedia</a>
4. <a id="r4" href="http://en.wikipedia.org/wiki/KISS_principle">KISS principle, wikipedia</a>
5. <a id="r5" href="http://en.wikipedia.org/wiki/Divide_and_conquer_algorithm">Divide and conquer algorithm, wikipedia</a>
6. <a id="r6" href="http://en.wikipedia.org/wiki/Parallel_computation">Parallel computation, wikipedia</a>
7. <a id="r7" href="http://en.wikipedia.org/wiki/Lazy_load">Lazy load, wikipedia</a>
8. <a id="r8" href="http://www.cs.tufts.edu/~nr/cs257/archive/don-knuth/empirical-fortran.pdf">An empirical study of Fortran programs</a>
9. <a id="r9" href="http://en.wikipedia.org/wiki/Category:Compiler_optimizations">Compiler optimizations, wikipedia</a>
10. <a id="r10" href="http://en.wikipedia.org/wiki/Register_allocation">Register allocation, wikipedia</a>
11. <a id="r11" href="https://www.amazon.com/Compiler-Design-Theory-Systems-programming/dp/0201144557">Compiler Design Theory</a>
12. <a id="r12" href="https://www.amazon.com/Art-Compiler-Design-Theory-Practice/dp/B00DJAX3KC">The art of compiler design - Theory and Practice</a>
13. <a id="r13" href="http://en.wikipedia.org/wiki/Horner_rule">Horner rule, wikipedia</a>
14. <a id="r14" href="http://en.wikipedia.org/wiki/Karatsuba_algorithm">Karatsuba algorithm, wikipedia</a>
15. <a id="r15" href="http://www.embedded.com/design/real-time-and-performance/4007256/Digital-Signal-Processing-Tricks--Fast-multiplication-of-complex-numbers">Fast multiplication of complex numbers</a>
16. <a id="r16" href="http://en.wikipedia.org/wiki/Exponentiation_by_squaring">Exponentiation by squaring, wikipedia</a>
17. <a id="r17" href="http://homepages.math.uic.edu/~leon/cs-mcs401-f07/handouts/fastexp.pdf">Fast Exponentiation</a>
18. <a id="r18" href="http://en.wikipedia.org/wiki/Strassen_algorithm">Strassen algorithm, wikipedia</a>
19. <a id="r19" href="http://en.wikipedia.org/wiki/Coppersmith%E2%80%93Winograd_algorithm">Coppersmith-Winograd algorithm, wikipedia</a>
20. <a id="r20" href="http://www.cs.berkeley.edu/~fateman/papers/factorial.pdf">Comments on Factorial Programs</a>
21. <a id="r21" href="https://github.com/PeterLuschny/Fast-Factorial-Functions">Fast Factorial Functions</a>
22. <a id="r22" href="https://en.wikipedia.org/wiki/Isomorphism">Isomorphism, wikipedia</a>
23. <a id="r23" href="https://en.wikipedia.org/wiki/Representation_(mathematics)">Representation, wikipedia</a>
24. <a id="r24" href="https://en.wikipedia.org/wiki/Symmetry">Symmetry, wikipedia</a>
25. <a id="r25" href="https://en.wikipedia.org/wiki/Merge_sort">Merge sort, wikipedia</a>
26. <a id="r26" href="https://en.wikipedia.org/wiki/Radix_sort">Radix sort, wikipedia</a>
27. <a id="r27" href="http://en.wikipedia.org/wiki/Function_inlining">Function inlining, wikipedia</a>
28. <a id="r28" href="http://en.wikipedia.org/wiki/Strength_reduction">Strength reduction, wikipedia</a>
29. <a id="r29" href="http://en.wikipedia.org/wiki/Loop_invariant">Loop invariant, wikipedia</a>
30. <a id="r30" href="http://en.wikipedia.org/wiki/Loop-invariant_code_motion">Loop-invariant code motion, wikipedia</a>
31. <a id="r31" href="http://en.wikipedia.org/wiki/Row-major_order">Array linearisation, wikipedia</a>
32. <a id="r32" href="http://en.wikipedia.org/wiki/Vectorization_%28mathematics%29">Vectorization, wikipedia</a>
33. <a id="r33" href="http://en.wikipedia.org/wiki/Loop_unwinding">Loop unrolling, wikipedia</a>
34. <a id="r34" href="http://en.wikipedia.org/wiki/List_of_software_development_philosophies">Software development philosophies, wikipedia</a>
35. <a id="r35" href="http://programmer.97things.oreilly.com/wiki/index.php/97_Things_Every_Programmer_Should_Know">97 Things every programmer should know</a>
36. <a id="r36" href="http://en.wikipedia.org/wiki/Data_structure">Data structure, wikipedia</a>
37. <a id="r37" href="http://en.wikipedia.org/wiki/List_of_data_structures">List of data structures, wikipedia</a>
38. <a id="r38" href="http://en.wikipedia.org/wiki/Cloud_computing">Cloud computing, wikipedia</a>
39. <a id="r39" href="http://en.wikipedia.org/wiki/Load_balancing_%28computing%29">Load balancing, wikipedia</a>
40. <a id="r40" href="http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller">Model-view-controller, wikipedia</a>
41. <a id="r41" href="http://en.wikipedia.org/wiki/Template_engine">Template engine, wikipedia</a>
42. <a id="r42" href="https://www.facebook.com/notes/facebook-engineering/three-optimization-tips-for-c/10151361643253920">Three optimization tips for C</a>
43. <a id="r43" href="http://www.slideshare.net/andreialexandrescu1/three-optimization-tips-for-c-15708507">Three optimization tips for C, slides</a>
44. <a id="r44" href="http://floating-point-gui.de/">What Every Programmer Should Know About Floating-Point Arithmetic</a>
45. <a id="r45" href="http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html">What Every Computer Scientist Should Know About Floating-Point Arithmetic</a>
46. <a id="r46" href="http://dragonbook.stanford.edu/lecture-notes/Stanford-CS143/20-Optimization.pdf">Optimisation techniques</a>
47. <a id="r47" href="https://cs.brown.edu/courses/cs033/docs/guides/c_optimization_notes.pdf">Notes on C Optimisation</a>
48. <a id="r48" href="http://www.tantalon.com/pete/cppopt/main.htm">Optimising C++</a>
49. <a id="r49" href="http://www.azillionmonkeys.com/qed/optimize.html">Programming Optimization</a>
50. <a id="r50" href="http://www.ibiblio.org/pub/languages/fortran/ch1-9.html">CODE OPTIMIZATION - USER TECHNIQUES</a>
51. <a id="r51" href="https://en.wikipedia.org/wiki/Locality_of_reference">Locality of reference, wikipedia</a>
52. <a id="r52" href="https://en.wikipedia.org/wiki/Memory_access_pattern">Memory access pattern, wikipedia</a>
53. <a id="r53" href="https://en.wikipedia.org/wiki/Memory_hierarchy">Memory hierarchy, wikipedia</a>
54. <a id="r54" href="https://en.wikipedia.org/wiki/Heterogeneous_computing">Heterogeneous computing, wikipedia</a>
55. <a id="r55" href="https://en.wikipedia.org/wiki/Stream_processing">Stream processing, wikipedia</a>
56. <a id="r56" href="https://en.wikipedia.org/wiki/Dataflow_programming">Dataflow programming, wikipedia</a>
57. <a id="r57" href="https://en.wikipedia.org/wiki/Fast_Fourier_transform">Fast Fourier transform, wikipedia</a>
58. <a id="r58" href="https://en.wikipedia.org/wiki/Gaussian_elimination">Gaussian elimination, wikipedia</a>
59. <a id="r59" href="https://en.wikipedia.org/wiki/Fermat%27s_little_theorem">Fermat's little theorem, wikipedia</a>
60. <a id="r60" href="https://en.wikipedia.org/wiki/Trigonometric_identities">Trigonometric identities, wikipedia</a>
61. <a id="r61" href="http://arxiv.org/abs/1102.1523v1">The NumPy array: a structure for efficient numerical computation</a>
62. <a id="r62" href="https://arxiv.org/abs/cs/0011047">Dancing links algorithm</a>
63. <a id="r63" href="http://soft.vub.ac.be/~madewael/w-JITds/paper.pdf">Data Interface + Algorithms = Efficient Programs</a>
64. <a id="r64" href="http://brandonlucia.com/pubs/approx2014.pdf">Systems Should Automatically Specialize Code and Data</a>
65. <a id="r65" href="http://www.cs.le.ac.uk/ac/LACFunding/LACNewParDS/finalreport.pdf">New Paradigms in Data Structure Design: Word-Level Parallelism and Self-Adjustment</a>
66. <a id="r66" href="http://20bits.com/article/10-tips-for-optimizing-mysql-queries-that-dont-suck">10 tips for optimising Mysql queries</a>
67. <a id="r67" href="https://www.percona.com/blog/">Mysql Optimisation</a>
68. <a id="r68" href="http://www.cs.technion.ac.il/~erez/Papers/wf-simulation-ppopp14.pdf">A Practical Wait-Free Simulation for Lock-Free Data Structures</a>
69. <a id="r69" href="http://www.cs.uoi.gr/tech_reports/publications/TR2011-01.pdf">A Highly-Efficient Wait-Free Universal Construction</a>
70. <a id="r70" href="http://www.cs.technion.ac.il/~erez/Papers/wf-methodology-ppopp12.pdf">A Methodology for Creating Fast Wait-Free Data Structures</a>
71. <a id="r71" href="https://en.wikipedia.org/wiki/Polynomial_greatest_common_divisor#Euclid.27s_algorithm">Euclidean Algorithm</a>
72. <a id="r72" href="https://en.wikipedia.org/wiki/Gr%C3%B6bner_basis">Gröbner basis</a>
73. <a id="r73" href="https://en.wikipedia.org/wiki/Newton%27s_method">Newton's Method</a>
74. <a id="r74" href="https://en.wikipedia.org/wiki/Fast_inverse_square_root">Fast Inverse Square Root</a>
75. <a id="r75" href="https://en.wikipedia.org/wiki/Methods_of_computing_square_roots">Methods of computing square roots</a>
76. <a id="r76" href="https://www.nayuki.io/page/fast-fibonacci-algorithms">Fast Fibonacci numbers</a>
77. <a id="r77" href="https://booking.ai/k-nearest-neighbours-from-slow-to-fast-thanks-to-maths-bec682357ccd">Fast k-Nearest Neighbors (k-NN) algorithm</a>
78. <a id="r78" href="https://hal.inria.fr/file/index/docid/71533/filename/RR-5050.pdf">A Binary Recursive Gcd Algorithm</a> 

