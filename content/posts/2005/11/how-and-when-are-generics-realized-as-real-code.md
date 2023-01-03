+++
title = "How and when are generics realized as real code?"
date = 2005-11-08T09:30:00.000-06:00
updated = 2005-11-08T13:35:50.076-06:00
draft = false
url = '/2005/11/how-and-when-are-generics-realized-as.html'
tags = []
+++

Questions often come up regarding .Net 2.0's generics; how much code is shared, when are the specialized versions created, and how much does it cost? While I want to repeat the refrain I often use—_You don't really **need** to know this_—it is useful information. The short version:

*   There is **one** copy of the generic IL
*   The **JIT** creates specializations as they are needed
*   All reference-type specializations share **one** JITted copy
*   Each value-type spawns a **separate** specialization

For more information, please refer to [this](http://ognjenbajic.com/blog/2005/11/generics-where-does-generic-code-get.html) excellent post from [Ognjen Bajić](http://ognjenbajic.com/blog/)
