---
author: VladimirPanteleev
comments: false
date: 2020-04-13 12:58:31+00:00
layout: post
link: https://dlang.org/blog/2020/04/13/dustmite-the-general-purpose-data-reduction-tool/
slug: dustmite-the-general-purpose-data-reduction-tool
title: 'DustMite: The General-Purpose Data Reduction Tool'
wordpress_id: 2354
categories:
- Compilers &amp; Tools
- Core Team
- Guest Posts
permalink: /dustmite-the-general-purpose-data-reduction-tool/
redirect_from: /2020/04/13/dustmite-the-general-purpose-data-reduction-tool/
---

If you've been around for a while, or are a particularly adventurous developer who enjoys mixing language features in interesting ways, you may have run into one compiler bug or two:






![](https://dump.thecybershadow.net/a40afda3f9ba4f774bcc6769a9a32202/dmd-crash.svgz)






Implementation bugs are inevitably a part of using cutting-edge programming languages. Should you run into one, the steps to proceed are generally as follows:







  1. Reduce the failing program to a minimal, self-contained example.


  2. Add a description of what happens and what you expect to happen.


  3. Post it on the bug tracker.





Nine years ago, [an observation was made](https://forum.dlang.org/post/im3guj$11cf$1@digitalmars.com) that when filing and fixing compiler bugs, a disproportionate amount of time was spent on the first step. When your program stops compiling "out of the blue", or when the bug stops reproducing after the code is taken out of its context, manually paring down a large codebase by repeatedly cutting out code and checking if the bug still occurs becomes a tedious and repetitive task.





Fortunately, tedious and repetitive tasks are what computers are good for; they just have to be tricked into doing them, usually by writing a program. [Enter DustMite](https://forum.dlang.org/post/op.vvsvhh1ptuzx1w@cybershadow.mshome.net).






![](https://dump.thecybershadow.net/6be0435f1acd6aa8a2f15cdf316cd743/v1.svgz)  

_The first version._





The basic operation is simple. The tool takes as inputs:







  * a data set to reduce (such as, a directory containing D source code which exhibits a particular compiler bug)


  * an oracle (or, more mundanely, a test script), which itself:


  * takes as input a variation of the data set, and


  * produces a yes-or-no answer on whether the input still satisfies the sought property (such as reproducing the particular compiler bug).





DustMite's output is some local minimum variation of the data set, which it reaches by consecutively trying to remove parts of the data set and saving the results which the oracle approves. In the compiler bug example, this means removing bits of code which are not required to reproduce the problem at hand.





DustMite wouldn't be very efficient if it attempted to remove things line-by-line or character-by-character. In order to maximize the chance of finding good reductions, the input is parsed into a tree according to the syntax of the input files.





Each tree node consists of a "head" (string), children (list of node pointers), and "tail" (string). Technically, it is redundant to have both "head" and "tail", but they make representing some constructs and performing reductions much simpler, such as paren/bracket pairs.






![](https://dump.thecybershadow.net/7e8db5926677e5e4886419e1e00ad9ec/tree-normal.svgz)  

_Nodes are arranged into a binary tree as an optimization._





Additionally, nodes may have a list of dependencies. The dependency relationship essentially means "if this node is removed, these nodes should be removed too". These constraints are not representable using just the tree structure described above, and are used to allow reducing things such as lists where trailing item delimiters are not allowed, or removing a function parameter and corresponding arguments from the entire code base at once.





In the case of D source code, declarations, statements, and subexpressions get their own tree nodes, so that they can be removed in one go if unneeded. The parser DustMite uses for D source code is intentionally very simple because it needs to handle potentially invalid D code, and you don't want your bug reduction tool to also crash on top of the compiler.






![](https://dump.thecybershadow.net/135d660b6d02bf5c2d7ea3e706afd506/out.svgz)  

_How DustMite sees a simple D program._





An algorithm decides the order in which nodes are queued for potential deletion; DustMite implements several (it calls them "strategies"). Fundamentally, a strategy's interface is (state_i_, result_i_) ⇒ (state_i_+1, reduction_i_+1), i.e., which reduction is chosen next depends on the previous reduction and its result. The default "inbreadth" strategy visits nodes in ascending depth order (row by row) and starts over from the top as long as it finds new reductions.





DustMite today supports quite a few more options:






![](https://dump.thecybershadow.net/8340a7f30c110d706f32d89492033d1f/vNow.svgz)  

_The current version._





Probably, the most interesting of these is the `-j` switch--one reason being that DustMite's task is inherently not parallelizable. Which reduction is chosen next, and the tree version to which that reduction is applied, depends on the previous reduction's result.





DustMite works around this by putting unused CPU cores to work on lookahead: using a [highly sophisticated predictor](https://github.com/CyberShadow/DustMite/blob/6746464fe8036342d0f657e11c442e9c24539085/dustmite.d#L1842), it guesses what the result of the current reduction will be, and based on that assumption, calculates the next reduction. If the guess was right, great! We get to use that result. Otherwise, the work is wasted. Implementing this meant that strategies now needed to have copyable state, and so [had to be refactored](https://github.com/CyberShadow/DustMite/commit/54321df719f779f44e7b46d139e09a764ba9e3d3#diff-bfae2a5bfcf57205d62a12180081fa75) from an imperative style to a state machine.





Unfortunately, although the highly expected feature was implemented four years ago, the initial implementation was rather underwhelming. DustMite still did too much work in the main thread and wasted too much CPU time on rescanning the data set tree on every reduction. The problem was so bad that, at high core counts, lookahead mode was even slower than single-threaded mode.





I have recently set out to resolve these inadequacies. The following obstacles stood in the way:





**Problem 1**: Hashing was too slow. Because the oracle's services (i.e., running the test script) are usually expensive, DustMite keeps track of a cache of previously attempted reductions and their outcome. This helps because not all tree transformations result in a change of output, and some strategies will retry reductions in successive iterations. A hash of the tree is used as the cache key; however, calculating it requires walking the entire tree every time, which is slow for large inputs.





Would it be possible to make the hash calculation incremental? One approach would be [Merkle trees](https://en.wikipedia.org/wiki/Merkle_tree) (each node's hash is the hash of its children's hashes), however that is suboptimal in the case of e.g., empty leaf nodes. CS erudite Ivan Kazmenko blesses us with an answer: [polynomial hashes](https://stackoverflow.com/a/42112687/21501)! By representing strings as polynomials, it is possible to use modulo arithmetic to calculate an incremental fixed-size hash and cache subtree hashes per node.






![](https://dump.thecybershadow.net/ad483c3d88a36f81a3ff069dde5dae85/treehash.pre.dot.post.svgz)  


_Each node holds its cumulative hash and length._





The number theory staggered me at first, so I recruited the assistance of _feep_ from `#d`. After we went through a few draft implementations, I could begin working on the final version. The first improvement was replacing the naive exponentiation algorithm with [exponentiation by squaring](https://github.com/CyberShadow/DustMite/commit/751ea2bec3a663e3b0e3f0b0d720c87f497aeac2) (D CTFE allowed precomputing a table at compile-time and a faster calculation than the classical method). Next, there was the matter of the modulo.





Initially, we used integer overflow for modulo arithmetic (i.e. _q_=264), however Ivan cautioned against using powers of two as the modulo, as this makes the algorithm [susceptible to Thue-Morse strings](http://codeforces.com/blog/entry/4898?locale=en). Not long ago I was experimenting with using long multiplication/division CPU instructions (where multiplying one machine word by another yields the result in two machine words with a high and low part, and vice-versa for division). D allows [generating assembler code](https://github.com/CyberShadow/DustMite/commit/19f0200688fd917dc2dc308307be543833c8adf7#diff-478cbff87f139dc726bb3269691e88c0R260) specific to the types that the function template is instantiated with, though in DustMite we only use the unsigned 64-bit variant (on x86 we fall back to using integer overflow).





With the hashing algorithm implemented, all that remained was to mark dirty nodes (they or their children had their content edited) and incrementally recalculate their hashes as needed. Dependencies posed a small obstacle: at the time, they were implemented as simply an array of pointers to the dependency node within the tree. As such, we didn't know how to get to their parents (to mark them dirty as well), however this was easily overcome by adding a "parent" pointer to each node.





Well, or so I thought, until I got to work on the next problem.





**Problem 2**: Copying the tree. At the time, the current version of the tree representing the data set was part of the global state. Because of this, applying a reduction was implemented twice:







  * Temporarily, when saving the data set with a proposed reduction for testing:  

[`void save(Reduction reduction, string savedir)`](https://github.com/CyberShadow/DustMite/blob/02f8b2ea99225b30ff58c218caa7678c7f35d001/dustmite.d#L1121)  

The tree here is not mutated, instead the reduction was applied "dynamically" to the output while walking the tree.



  * Permanently, when accepting a successful reduction, and irreversibly mutating the in-memory tree to apply it:  

[`void applyReduction(ref Reduction r)`](https://github.com/CyberShadow/DustMite/blob/02f8b2ea99225b30ff58c218caa7678c7f35d001/dustmite.d#L1197)






This was clumsy, but faster and less complicated than making a copy of the entire tree just to change one part of it to test a reduction. However, doing so was a requirement for proper lookahead, otherwise we would be unable to test reductions based on results where past tests predicted a positive outcome, or do nearly anything in a separate thread.





One issue was the tree "internal pointers"--making a copy would require updating all pointers within the tree to point to the new copies in the new tree. This was easy for children/parent pointers (since we can reliably visit every such pointer exactly once), but not quite for dependencies: because they were also implemented as simple pointers to nodes, we would have to keep track of a map of which node was copied where in order to update the dependency pointers.





One way to solve this would be to change the representation of node references from pointers to indices into a node array; this way, copying the tree would be as simple as a `.dup`. However, large inputs meant many nodes, and I wanted to see if it was possible to avoid iterating over every node in the tree (i.e. _O(n)_) for every reduction.





Was it possible? It would mean that we would copy only the modified nodes and their parents, leaving the rest of the tree in-place, and only reusing it as the copies' children. [This goal conflicted](https://github.com/CyberShadow/DustMite/commit/0e788b5ec6a18f15c6be3bb41d5b9e058a78810f) with the existence of "parent" pointers, because a parent would have to point towards either the old or new root, so to resolve this ambiguity every node would have to be copied. As a result, the way we handled dependencies needed to be rethought.






![](https://dump.thecybershadow.net/abade8db81bca9c9dfed1bc704f3ace8/tree-edit.svgz)  

_Editing trees with "copy on write" involves copying just the edited nodes (🔴), and their parents._





With internal pointers out, the next best thing to array indices for referencing a node was a series of instructions for how to reach the node from the tree root: [an address](https://github.com/CyberShadow/DustMite/blob/6746464fe8036342d0f657e11c442e9c24539085/splitter.d#L22-L39). The representation of these addresses that I chose was a bit string represented as a linked list, where each list node holds the child index at that depth, starting from the deep end. Such a representation can be arranged in a tree where the root-side ends are shared, mimicking the structure of the tree containing the nodes for the reduced data, and thus allowing us to reuse memory and minimize allocations.






![](https://dump.thecybershadow.net/d7e9c76fbae17208d14683a5132a4780/tree-address.svgz)  

_Nodes cannot hold their own address (as that would make them unmovable),  
which is why they need to be stored outside of the main tree._





For addresses to work, the object they point at needs to remain the same, which means that we can no longer simply remove children from tree nodes--an address going through the second child would become invalid if the first child was removed. Rewriting all affected addresses for every tree edit is, of course, impractical, which leads us to the introduction of tombstones--dead nodes that only serve to preserve the index of the children that follow it. Because one of the possible reduction types involves moving subtrees around the tree, we now also have "redirects" (which are just tombstones with a "see here" address attached).





With [the above changes in place](https://github.com/cybershadow/DustMite/commit/2ca05223be57fbd0f1592c9ac4fdb4abc9b54019), we can finally move forward with fixing and optimizing lookahead, as well as implementing incremental rehashing in a way that's compatible with the above! The mutable global "current" tree variable is gone, `save` now simply takes a tree root as an argument, and `applyReduction` is now:




    /// Apply a reduction to this tree, and return the resulting tree.
    /// The original tree remains unchanged.
    /// Copies only modified parts of the tree, and whatever references them.
    Entity applyReduction(Entity origRoot, ref Reduction r)

With the biggest hurdle behind us, and a few more rounds of applying [Walter Bright's secret weapon](https://forum.dlang.org/post/i53nti$1ja3$1@digitalmars.com), the performance metrics started to look more like what they should:






![](https://dump.thecybershadow.net/ccea7b70a9bbee89944b3b0a8e65ec37/times.svgz)  

_Going deeper would likely involve using OS-specific I/O APIs or rewriting D's GC._





A mere 3.5x speed-up from a 32-fold increase in computational power may seem underwhelming. Here are some reasons for this:







  * With a 50/50 predictor, predictions form a complete binary tree, so doubling the number of parallel jobs gives you +1x more speed. That's roughly log₂(jobs)-1, or 4 for 32 jobs - not far off!



  * The results will depend on the reduction being performed, so YMMV. For a certain artificial test case, one optimization (not pictured above) yielded a 500x speed-up!



  * DustMite does not try to keep all CPU cores busy all the time. If a prediction turns out false, all lookahead jobs based on it become wasted work, so DustMite only starts new lookahead tasks when a reduction's outcome is resolved. Perhaps ideally DustMite would keep starting new jobs but kill them as soon as it discovers they're based on a misprediction. As there is no cross-platform process group management in Phobos, the D standard library, this is something I left for future versions.



  * Some work is still done in the main thread, because moving it to a worker thread actually makes things slower due to the global GC lock.






There still remains one last place where DustMite iterates over every tree node per reduction: saving the tree to disk (so that it could be read by the test script). This seems unavoidable at first, but could actually be avoided by caching each node's full text contents within the node itself.





I opted to leave this one out. With the other related improvements, such as using `lockingBinaryWriter` and [aggregating writes of contiguous strings as one I/O operation](https://github.com/cybershadow/DustMite/commit/621991bd062a6967236f6dc262fdf59ad59aaede), the increase in memory usage was much more dramatic than the decrease in execution time, even when optimized to just one allocation per reduction (polynomial hashing gives us every node's total length for free). But, [for a brief instant](https://github.com/CyberShadow/DustMite/compare/9f5a4f1b930c57221192f615f6f563034d3290a8...4b165e6898c7382b1dc4603d058965e492bd2dab), DustMite processed reductions in sub-_O(n)_ time.





One more addition is worth mentioning: [Andrej Mitrovic suggested](https://github.com/CyberShadow/DustMite/issues/59) a switch which would replace removed text with whitespace, which would allow searching for exact line numbers in the test script. At the time, its addition posed significant challenges, as there needed to be some way to keep removed nodes in the tree but exclude them from future removal attempts. With the new tree representation, this became much easier, and also allowed creating the following animation:






![](https://dump.thecybershadow.net/6b58560174d8f1f5c0d15315fe6ab021/anim.svgz)





In conclusion, I'd like to bring up that DustMite is good at more than just reducing compiler test cases. The wiki lists some ideas:







  * Finding the source of ambiguous or misleading compiler error messages (e.g., errors with the file/line information pointing only inside the standard library).



  * Alternative (much slower, but also much more thorough) method of verifying unit test code coverage. Just because a line of code is executed, that doesn't mean it's _necessary_; DustMite can be made to remove all code that does not affect the execution of your unit tests.



  * Similarly, if you have complete test coverage, it can be used for reducing the source tree to a minimal tree which includes support for only enabled unittests. This can be used to create a version of a program or library with a test-defined subset of features.



  * The `--obfuscate` mode can obfuscate your code's identifiers. It can be used for preparing submission of proprietary code to bug trackers.



  * The `--fuzz` mode (a new feature) can help find bugs in compilers and tools by creating random programs (using fragments of other programs as input).






But DustMite is not limited to D programs (or any kind of programs) as input. With the `--split` option, we can tell DustMite how to parse and reduce other kinds of files. DustMite successfully handled the following scenarios:







  * reducing C++ programs (the D parser supports some C++-only syntax too);



  * reducing Python programs (using the `indent` split mode);



  * reducing a large commit to a minimal diff (using the `diff` split mode);



  * reducing a commit list, when `git bisect` is insufficient due to the problem being introduced across more than any single commit;



  * reducing a large data set to a minimal one, resulting in the same code coverage, with the purpose of creating a test suite;



  * and many more which I do not remember.






Today, some version of DustMite is readily available in major distributions (usually as part of some D-related package), so I'm happy having a favorite tool one `apt-get` / `pacman -S` away when I'm not at my PC.





Discovering a problem which can be elegantly reduced away by DustMite is always exciting for me, and I'm hoping you will find it useful too.
