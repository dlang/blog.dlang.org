---
author: RazvanNitu
comments: false
date: 2024-02-22 09:41:31+00:00
layout: post
link: https://dlang.org/blog/2024/02/22/dmd-compiler-as-a-library-a-call-to-arms/
slug: dmd-compiler-as-a-library-a-call-to-arms
title: 'DMD Compiler as a Library: A Call to Arms'
wordpress_id: 3152
categories:
- Community
- Compilers &amp; Tools
- Core Team
- D Foundation
permalink: /dmd-compiler-as-a-library-a-call-to-arms/
redirect_from: /2024/02/22/dmd-compiler-as-a-library-a-call-to-arms/
---

![Digital Mars D logo]({{ '/assets/images/dmd-compiler-as-a-library-a-call-to-arms/d6.png' | relative_url }})

Having a flexible and powerful compiler library has been one of the stated goals of the D Language Foundation for some time now. This makes sense, as a proper compiler library will channel the efforts of contributors into building developer tools, which in turn, will increase the adoption rate of the language. However, progress on this topic has been slow, mainly due to two aspects: (1) the lack of a clear direction, and (2) the intimidating complexity of the DMD frontend, which requires significant work on the compiler codebase.





The good news is that we now have a plan, which I will outline in this blog post. The bad news is that implementing this plan requires significant effort, and we need more contributors. However, the silver lining is that the work, while extensive, mostly involves refactoring the code. This provides an excellent opportunity for contributors to familiarize themselves with the compiler codebase while delivering real value. Before delving into the specifics, let me give you some background.





## Current Status And How We Got Here





To fully understand the work done so far on the compiler-as-a-library project, I highly recommend watching [my talk](https://youtu.be/eKT3MYpRgYA?si=FwY84BMXYGLtpGgQ) on this subject.





In summary:







  * Several years ago, we began packaging the compiler as a library.


  * Our goal was to clearly separate compilation phases: lexing, parsing, semantic analysis, optimizations, and code generation.


  * The parsing and semantic analysis modules were interdependent, necessitating a method for separation.


  * We opted to template the parser with an ASTFamily template parameter, defining the AST nodes required for parsing.


  * We created ASTBase (containing AST nodes essential for parsing) and ASTCodegen (containing AST nodes needed for code generation).


  * ASTBase, as it stands, is code duplicated from ASTCodegen.


  * We started extracting semantic routines and fields from AST nodes to eliminate ASTBase's code duplication by importing a subset of modules used by ASTCodegen.


  * Additionally, we began replacing third-party libraries (like [libdparse](https://github.com/dlang-community/libdparse/tree/master/src)) with the DMD-as-a-library package.





For more detailed information on each of these points, I recommend watching [the talk](https://youtu.be/eKT3MYpRgYA?si=FwY84BMXYGLtpGgQ) I referenced.





Recently, I proposed to Walter a modification to the codebase that would significantly enhance the flexibility of the compiler library, allowing any AST node to be overwritten. Walter was hesitant to accept my proposal, concerned about the potential "ugliness" it would introduce to the codebase. He cited the addition of ASTBase and the resulting code duplication as a precedent. He then suggested that if we eliminate ASTBase, he would reconsider my proposal.





## What You Can Do To Help





We are now focused on eliminating the duplication in ASTBase. To achieve this, we need to extract all information related to semantic analysis from the existing AST nodes. The challenge is the sheer number of AST nodes and the multitude of functions associated with each. I have been working on this sporadically over the past few months, and progress is slow due to the nature of the work: it mostly involves moving code, creating visitors, breaking dependencies, etc. While not overly complex, it isn't particularly creative work either. However, for someone interested in understanding a real-life compiler codebase, it's an ideal starting point.





If you're willing to support this initiative, I've put together [a guide on where to start and what you can do](https://github.com/dlang/dmd/blob/master/CONTRIBUTING.md#refactoring-the-dmd-ast). Feel free to contact me on Slack (razvan.nitu), Discord, or email (razvan.nitu1305@gmail.com) for more details or to request a review of your PR.





I see this as an excellent opportunity to onboard new people into compiler development in a way that benefits both the language and the contributor. So, if you have some spare time, please join us in getting this work done!
