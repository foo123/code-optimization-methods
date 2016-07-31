##Code Optimization Methods

*A summary of various code optimization methods*


###Contents

* [General Principles](#general-principles)
* [Low-level](#low-level)
* [Language-independent optimization](#language-independent-optimization)
* [Databases](#databases)
* [Web](#web)
* [References](#references)


__A note on the relation between Computational Complexity of Algorithms and Code Optimization Techniques__

Both computational complexity theory <sup> [24] </sup> and code optimization techniques <sup> [1] </sup> , have the common goal of efficient problem solution. While they are related to each other and share some concepts, the difference lies in what is emphasized at each time.

Computational complexity theory, studies the performance with respect to input data size. Trying to design algorithmic solutions that have the least / fastest dependence on data size, regardless of underlying architecture. Code optimization techniques, on the other hand, focus on the architecture and the specific constants which enter those computational complexity estimations.

Systems that operate in Real-Time are examples where both factors can be critical (eg. [Real-time Image Processing in JavaScript](https://github.com/foo123/FILTER.js) , yeah i know i am the author :) ).


###General Principles


* __Keep it DRY and Cache__ : The general concept of caching involves avoiding re-computation/re-loading of a result if not necessary. This can be seen as a variation of Dont Repeat Yourself principle <sup> [2] </sup> . Even dynamic programming can be seen as a variation of caching, in the sense that it stores intermediate results saving re-computation time and resources.

* __KISS it, simpler can be faster__  :  Keep it simple <sup> [3] </sup> , makes various other techniques easier to apply and modify. ( A plethora of Software Engineering methods can help in this <sup> [25], [26], [46], [47] </sup>  )

* __Sosi ample free orginizd, So simple if re-organized__ : Dont hesitate to re-organize if needed. Many times sth can be re-organized, re-structured in a much simpler / faster way while retaining its original functionality (concept of isomorphism <sup> [48] </sup> , change of representation <sup> [49], [50], [27] </sup>), yet providing other advantages. For example, the expression (10+5*2)^2 is the simple constant 400, another example is the transformation from infix expression notation to prefix (Polish) notation which can be evaluated faster in one pass.

* __Divide into subproblems and Conquer the solution__ : Subproblems (or smaller, trivial, special cases) can be easier/faster to solve and combine for the global solution. Sorting algorithms are great examples of that <sup> [4], [Sorting Algorithms in JavaScript](https://github.com/foo123/SortingAlgorithms) </sup>

* __More helping hands are always welcome__ : Spread the work load, subdivide, share, parallelize if possible <sup> [5], [45] </sup> .

* __United we Stand and Deliver__ : Having data together in a contiguous chunk, instead of scattered around here and there, makes it faster to load and process as a single block, instead of (fetching and) accessing many smaller chunks (eg. cache memory, vector/pipeline machines, database queries) <sup> [42], [43], [44] </sup>.

* __A little Laziness never hurt anyone__ : So true, each time a program is executed, only some of its data and functionality are used. Delaying to load and initialize (being lazy) all the data and functionality untill needed, can go a long way <sup> [6] </sup> .


__Further Notes__

Before trying to optimize, one has to measure and identify what needs to be optimized, if any. Blind "optimization" can be as good as no optimization at all, if not worse.

That being said, one should always try to optimize and produce efficient solutions. A non-efficient "solution" can be as good as no solution at all, if not worse.

Some of the optimization techniques can be automated (eg in compilers), while others are better handled manually. 

Some times there is a trade-off between space/time resources. Increasing speed might result in increasing space/memory requirements (__caching__ is a classic example of that). 

The 90-10 (or 80-20 or other variations) rule of thumb, states that __90 percent of the time__ is spent on __10 percent of the code__ (eg a loop). Optimizing this part of the code can result in great benefits. (see for example  Knuth <sup> [7] </sup> )

One optimization technique (eg simplification) can lead to the application of another optimization technique (eg constant substitution) and this in turn can lead back to the further application of the first optimization technique (or others). Doors can open.


__References:__ [8], [10], [11], [35], [36], [37], [38], [39]


###Low-level


####Generalities<sup> [33], [34] </sup>

__Data Allocation__

* Disk access is slow
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


####Methods


* __Register allocation__ : Since register memory is fastest way to access heavily used data, it is desirable (eg compilers, real-time systems) to allocate some data in an optimum sense in the cpu registers during a heavy-load operation. There are various algorithms (based on the graph coloring problem) which provide an automated way for this kind of optimization. Other times a programmer can explicitly declare a variable that is allocated in the cpu registers during some part of an operation <sup> [9] </sup> 


* __Single Atom Optimizations__ : This involves various operations which optimize one cpu instruction (atom) at a time. For example some operands in an instruction, can be constants, so their values can be replaced instead of the variables. Another example is replacing exponentiation with a power of 2 with a multiplication, etc..


* __Optimizations over a group of Atoms__ : Similar to previous, this kind of optimization, involves examining the control flow over a group of cpu instructions and re-arranging so that the functionality is retained, while using simpler/fewer instructions. For example a complex IF THEN logic, depending on parameters, can be simplified to a single Jump statement, and so on.



###Language-independent optimization


* __Re-arranging Expressions__ : More efficient code for the evaluation of an expression can often be produced if the operations occuring in the expression are evaluated in a different order. Classic examples are Horner's Rule <sup> [12] </sup>,  Karatsuba Multiplication <sup> [13] </sup>,  fast complex multiplication <sup> [14] </sup>, fast matrix multiplication <sup> [15],  [16] </sup>,  fast exponentiation <sup> [29],  [30] </sup>, fast factorials/binomials <sup> [40], [41] </sup>.


* __Constant Substitution/Propagation__ : Many times an expression is under all cases evaluated to a single constant, the constant value can be replaced instead of the more complex and slower expression (sometimes compilers do that).


* __Inline Function/Routine Calls__ : Calling a function or routine, involves many operations from the part of the cpu, it has to push onto the stack the current program state and branch to another location, and then do the reverse procedure. This can be slow when used inside heavy-load operations, inlining the function body can be much faster without all this overhead. Sometimes compilers do that, other times a programmer can declare or annotate a function as **inline** explicitly. <sup> [17] </sup> 


* __Combining Flow Transfers__ : IF/THEN instructions and logic are, in essence, cpu branch instructions. Branch instructions involve changing the program pointer and going to a new location. This can be slower if many jump instructions are used. However re-arranging the IF/THEN statements (factorizing common code, using De Morgan's rules for logic simplification etc..) can result in isomorphic functionality with fewer and more efficient logic and as a result fewer and more efficient branch instructions


* __Dead Code Elimination__ : Most times compilers can identify code that is never accessed and remove it from the compiled program. However not all cases can be identified. Using previous simplification schemes, the programmer can more easily identify "dead code" (never accessed) and remove it.


* __Common Subexpressions__ : This optimization involves identifying subexpressions which are common in various parts of the code and evaluating them only once and use the value in all subsequent places (sometimes compilers do that).


* __Common Code Factorisation__ : Many times the same block of code is present in different branches, for example the program has to do some common functionality and then something else depending on some parameter. This common code can be factored out of the branches and thus eliminate unneeded redundancy , latency and size.


* __Strength Reduction__ : This involves transforming an operation (eg an expression) into an equivalent one which is faster. Common cases involve replacing exponentiation with multiplication and multiplication with addition (eg inside a loop). This technique can result in great efficiency stemming from the fact that simpler but equivalent operations are several cpu cycles faster (usually implemented in hardware) than their more complex equivalents (usually implemented in software) <sup> [18] </sup> 


* __Handling Trivial/Special Cases__ : Sometimes a complex computation has some trivial or special cases which can be handled much more efficiently by a reduced/simplified version of the computation (eg computing a^b, can handle the special cases for a,b=0,1,2 by a simpler method). Trivial cases occur with some frequency in applications, so simplified special case code can be quite useful.  <sup> [31], [32] </sup> . Similar to this, is the handling of common/frequent computations (depending on application) with fine-tuned or faster code.


* __Exploiting Mathematical Theorems/Relations__ : Some times a computation can be performed in an equivalent but more efficient way by using some mathematical theorem, transformation, symmetry <sup> [50] </sup> or knowledge (eg. Gauss method of solving Systems of Linear equations, Fast Fourier Transforms, Fermat's Little Theorem,  Taylor-Mclaurin Series Expasions, Trigonometric Identities, etc..). This can go a long way. It is good to refresh your mathematical knowledge every now and then.

* __Using Efficient Data Structures__ : Data structures are the counterpart of algorithms (in the space domain), each efficient algorithm needs an associated efficient data structure for the specific task. In many cases using an appropriate data structure (representation) can make all the difference (eg. database designers and search engine developers know this very well) <sup> [27], [28], [49] </sup> 



__Loop Optimizations__

Perhaps the most important code optimization techniques are the ones involving loops.


* __Code Motion / Loop Invariants__ : Sometimes code inside a loop is independent of the loop index, can be moved out of the loop and computed only once (it is a loop invariant). This results in the loop doing fewer operations (sometimes compilers do that) <sup> [19], [20] </sup> 

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


* __Array Linearization__ : This involves handling a multi-dimensional array in a loop, as if it was a (simpler) one-dimensional array. Most times multi-dimensional arrays (eg 2D arrays NxM) use a linearization scheme, when stored in memory. Same scheme can be used to access the array data as if it is one big 1-dimensional array. This results in using a single loop instead of multiple nested loops <sup> [21], [22] </sup> 

__example:__

```javascript

// nested loop
// N = M = 20
// total size = NxM = 400
for (i=0; i<20; i++)
{
    for (j=0; j<20; j++)
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


* __Loop Unrolling__ : Loop unrolling involves reducing the number of executions of a loop by performing the computations corresponding to two (or more) loop iterations in a single loop iteration. This is partial loop unrolling, full loop unrolling involves eliminating the loop completely and doing all the iterations explicitly in the code (for example for small loops where the number of iterations is fixed). Loop unrolling results in the loop (and as a consequence all the overhead associated with each loop iteration) executing fewer times. In processors which allow pipelining or parallel computations, loop unroling can have an additional benefit, the next unrolled iteration can start while the previous unrolled iteration is being computed or loaded without waiting to finish. Thus loop speed can increase even more <sup> [23] </sup> 

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


###Databases

####Generalities

Database Access can be expensive, this means it is usually better to fetch the needed data using as few DB connections and calls as possible


####Methods


* __Lazy Load__ : Avoiding the DB access unless necessary can be efficient, provided that during the application life-cycle there is a frequency of cases where the extra data are not needed or requested


* __Caching__ : Re-using previous fetched data-results for same query, if not critical and if a slight-delayed update is tolerable


* __Using Efficient Queries__ : For Relational DBs, the most efficient query is by using an index (or a set of indexes) by which data are uniquely indexed in the DB.  


* __Exploiting Redundancy__ : Adding more helping hands(DBs) to handle the load instead of just one. In effect this means copying (creating redundancy) of data in multiple places, which can subdivide the total load and handle it independantly


###Web

* __Minimal Transactions__ : Data over the internet (and generally data over a network), take some time to be transmitted. More so if the data are large, therefore it is best to transmit only the necessary data, and even these in a compact form. That is one reason why JSON replaced the verbose XML for encoding of arbitrary data on the web.


* __Minimum Number of Requests__ : This can be seen as a variaton of the previous principle. It means that not only each request should transmit only necessary data in a compact form, but also that the number of requests should be minimized. This can include, minifying .css files into one .css file (even embedding images if needed), minifying .js files into one .js file, etc.. This can sometimes generate large data (files), however coupled with the next tip, can result in better performance.


* __Cache, cache and then cache some more__ : This can include everything, from whole pages, to .css files, .js files, images etc.. Cache in the server, cache in the client, cache in-between, cache everywhere..


* __Exploiting Redundancy__ : For web applications, this is usually implemented by exploiting some [cloud architecture](http://en.wikipedia.org/wiki/Cloud_computing) in order to store (static) files, which can be loaded (through the cloud) from more than one location. Other approaches include, [Load balancing](http://en.wikipedia.org/wiki/Load_balancing_%28computing%29) ( having redundancy not only for static files, but also for servers ).


* __Make application code faster/lighter__ : This draws from the previous principles about code optimization in general. Efficient application code can save both server and user resources. There is a reason why Facebook created HipHop VM ..


* __Minimalism is an art form__ : Having web pages and applications with tons of html, images, (not to mention a ton of advertisements) etc, etc.. is not necessarily better design, and of course makes page load time slower. Therefore having minimal pages and doing updates in small chunks using AJAX and JSON (that is what web 2.0 was all about), instead of reloading a whole page each time, can go a long way. This is one reason why [Template Engines](http://en.wikipedia.org/wiki/Template_engine) and [MVC Frameworks](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) were created. Minimalism does not need to sacrifice the artistic dimension, [Minimalism](http://en.wikipedia.org/wiki/Minimalism) __IS__ an art form.


[![Zen Circle](/zen-circle.jpg)](http://en.wikipedia.org/wiki/Ens%C5%8D)


###References

* http://en.wikipedia.org/wiki/Code_optimization
* http://en.wikipedia.org/wiki/Computational_complexity_theory
* http://en.wikipedia.org/wiki/Don%27t_repeat_yourself
* http://en.wikipedia.org/wiki/KISS_principle
* http://en.wikipedia.org/wiki/Divide_and_conquer_algorithm
* http://en.wikipedia.org/wiki/Parallel_computation
* http://en.wikipedia.org/wiki/Lazy_load
* http://www.cs.tufts.edu/~nr/cs257/archive/don-knuth/empirical-fortran.pdf
* http://en.wikipedia.org/wiki/Category:Compiler_optimizations
* http://en.wikipedia.org/wiki/Register_allocation
* Compiler Design Theory, 1976, Lewis, Rosenkrantz, Stearns
* The art of compiler design - Theory and Practice, 1992, Pittman, Peters
* http://en.wikipedia.org/wiki/Horner_rule
* http://en.wikipedia.org/wiki/Karatsuba_algorithm
* http://www.embedded.com/design/real-time-and-performance/4007256/Digital-Signal-Processing-Tricks--Fast-multiplication-of-complex-numbers
* http://en.wikipedia.org/wiki/Strassen_algorithm
* http://en.wikipedia.org/wiki/Coppersmith%E2%80%93Winograd_algorithm
* http://en.wikipedia.org/wiki/Function_inlining
* http://en.wikipedia.org/wiki/Strength_reduction
* http://en.wikipedia.org/wiki/Loop_invariant
* http://en.wikipedia.org/wiki/Loop-invariant_code_motion
* http://en.wikipedia.org/wiki/Row-major_order
* http://en.wikipedia.org/wiki/Vectorization_%28mathematics%29
* http://en.wikipedia.org/wiki/Loop_unwinding
* http://en.wikipedia.org/wiki/List_of_software_development_philosophies
* http://programmer.97things.oreilly.com/wiki/index.php/97_Things_Every_Programmer_Should_Know
* http://en.wikipedia.org/wiki/Data_structure
* http://en.wikipedia.org/wiki/List_of_data_structures
* http://en.wikipedia.org/wiki/Cloud_computing
* http://en.wikipedia.org/wiki/Load_balancing_%28computing%29
* http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller
* http://en.wikipedia.org/wiki/Template_engine
* https://www.facebook.com/notes/facebook-engineering/three-optimization-tips-for-c/10151361643253920
* http://www.slideshare.net/andreialexandrescu1/three-optimization-tips-for-c-15708507
* http://floating-point-gui.de/
* http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html
* http://dragonbook.stanford.edu/lecture-notes/Stanford-CS143/20-Optimization.pdf
* https://cs.brown.edu/courses/cs033/docs/guides/c_optimization_notes.pdf
* http://www.tantalon.com/pete/cppopt/main.htm
* http://www.azillionmonkeys.com/qed/optimize.html
* http://www.ibiblio.org/pub/languages/fortran/ch1-9.html
* http://www.cs.berkeley.edu/~fateman/papers/factorial.pdf
* https://github.com/PeterLuschny/Fast-Factorial-Functions
* https://en.wikipedia.org/wiki/Locality_of_reference
* https://en.wikipedia.org/wiki/Memory_access_pattern
* https://en.wikipedia.org/wiki/Memory_hierarchy
* https://en.wikipedia.org/wiki/Heterogeneous_computing
* https://en.wikipedia.org/wiki/Stream_processing
* https://en.wikipedia.org/wiki/Dataflow_programming
* https://en.wikipedia.org/wiki/Isomorphism
* https://en.wikipedia.org/wiki/Representation_(mathematics)
* https://en.wikipedia.org/wiki/Symmetry

[1]: http://en.wikipedia.org/wiki/Code_optimization
[2]: http://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[3]: http://en.wikipedia.org/wiki/KISS_principle
[4]: http://en.wikipedia.org/wiki/Divide_and_conquer_algorithm
[5]: http://en.wikipedia.org/wiki/Parallel_computation
[6]: http://en.wikipedia.org/wiki/Lazy_load
[7]: http://www.cs.tufts.edu/~nr/cs257/archive/don-knuth/empirical-fortran.pdf
[8]: http://en.wikipedia.org/wiki/Category:Compiler_optimizations
[9]: http://en.wikipedia.org/wiki/Register_allocation
[10]: Compiler Design Theory, 1976, Lewis, Rosenkrantz, Stearns
[11]: The art of compiler design - Theory and Practice, 1992, Pittman, Peters
[12]: http://en.wikipedia.org/wiki/Horner_rule
[13]: http://en.wikipedia.org/wiki/Karatsuba_algorithm
[14]: http://www.embedded.com/design/real-time-and-performance/4007256/Digital-Signal-Processing-Tricks--Fast-multiplication-of-complex-numbers
[15]: http://en.wikipedia.org/wiki/Strassen_algorithm
[16]: http://en.wikipedia.org/wiki/Coppersmith%E2%80%93Winograd_algorithm
[17]: http://en.wikipedia.org/wiki/Function_inlining
[18]: http://en.wikipedia.org/wiki/Strength_reduction
[19]: http://en.wikipedia.org/wiki/Loop_invariant
[20]: http://en.wikipedia.org/wiki/Loop-invariant_code_motion
[21]: http://en.wikipedia.org/wiki/Row-major_order
[22]: http://en.wikipedia.org/wiki/Vectorization_%28mathematics%29
[23]: http://en.wikipedia.org/wiki/Loop_unwinding
[24]: http://en.wikipedia.org/wiki/Computational_complexity_theory
[25]: http://en.wikipedia.org/wiki/List_of_software_development_philosophies
[26]: http://programmer.97things.oreilly.com/wiki/index.php/97_Things_Every_Programmer_Should_Know
[27]: http://en.wikipedia.org/wiki/Data_structure
[28]: http://en.wikipedia.org/wiki/List_of_data_structures
[29]: http://en.wikipedia.org/wiki/Exponentiation_by_squaring
[30]: http://homepages.math.uic.edu/~leon/cs-mcs401-f07/handouts/fastexp.pdf
[31]: https://www.facebook.com/notes/facebook-engineering/three-optimization-tips-for-c/10151361643253920
[32]: http://www.slideshare.net/andreialexandrescu1/three-optimization-tips-for-c-15708507
[33]: http://floating-point-gui.de/
[34]: http://docs.oracle.com/cd/E19957-01/806-3568/ncg_goldberg.html
[35]: http://dragonbook.stanford.edu/lecture-notes/Stanford-CS143/20-Optimization.pdf
[36]: https://cs.brown.edu/courses/cs033/docs/guides/c_optimization_notes.pdf
[37]: http://www.tantalon.com/pete/cppopt/main.htm
[38]: http://www.azillionmonkeys.com/qed/optimize.html
[39]: http://www.ibiblio.org/pub/languages/fortran/ch1-9.html
[40]: http://www.cs.berkeley.edu/~fateman/papers/factorial.pdf
[41]: https://github.com/PeterLuschny/Fast-Factorial-Functions
[42]: https://en.wikipedia.org/wiki/Locality_of_reference
[43]: https://en.wikipedia.org/wiki/Memory_access_pattern
[44]: https://en.wikipedia.org/wiki/Memory_hierarchy
[45]: https://en.wikipedia.org/wiki/Heterogeneous_computing
[46]: https://en.wikipedia.org/wiki/Stream_processing
[47]: https://en.wikipedia.org/wiki/Dataflow_programming
[48]: https://en.wikipedia.org/wiki/Isomorphism
[49]: https://en.wikipedia.org/wiki/Representation_(mathematics)
[50]: https://en.wikipedia.org/wiki/Symmetry
