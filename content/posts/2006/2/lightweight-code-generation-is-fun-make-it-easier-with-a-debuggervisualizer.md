+++
title = "Lightweight Code Generation is fun, make it easier with a DebuggerVisualizer"
date = 2006-02-14T22:12:00.000-06:00
updated = 2007-11-19T00:19:35.596-06:00
draft = false
url = '/2006/02/lightweight-code-generation-is-fun.html'
tags = ["Emit","LCG","DynamicMethod","IL","Dynamic","lightweight code generation"]
+++

When you're doing lightweight code generation ([LCG](http://blogs.msdn.com/joelpob/archive/2004/04/01/105862.aspx)) using [DynamicMethod](http://msdn2.microsoft.com/en-us/library/system.reflection.emit.dynamicmethod(VS.80).aspx) under .Net 2.0, it is easy to get lost and not be sure what your resulting code really looks like. **Luckily** for us, [Haibo Luo](http://blogs.msdn.com/haibo_luo) knows enough about ripping IL and the new [DebuggerVisualizer](http://msdn2.microsoft.com/en-us/library/zayyhzts.aspx) plug-in infrastructure of Visual Studio 2005. [DebuggerVisualizer for DynamicMethod (Show me the IL)](http://blogs.msdn.com/haibo_luo/archive/2005/10/25/484861.aspx) It's like having [Reflector](http://www.aisto.com/roeder/dotnet) (in IL mode) inside the debugger.
