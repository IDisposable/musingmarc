+++
title = "IFilter Text Extracter now hosted on CodePlex"
date = 2006-12-29T02:31:00.000-06:00
updated = 2006-12-29T02:39:57.413-06:00
draft = false
url = '/2006/12/ifilter-text-extracter-now-hosted-on.html'
tags = ["CodePlex","IDL","IFilter","COM"]
+++

My IFilter Text Extractor that [I mentioned previously](http://musingmarc.blogspot.com/2006/12/com-and-idl-and-c-and-ifilter-oh-my.html) has now been uploaded to it's new [CodePlex home](http://www.codeplex.com/ifilter). If you have any enhancement requests or problems, please use the appropriate [forums on CodePlex](http://www.codeplex.com/ifilter/Project/ListForums.aspx).

In VB.Net, the code to extract up to 64K of text from the file _C:\\Test.doc_ is as simple as

```
Dim te As ExtractTextLib.TextExtractor = New ExtractTextLib.TextExtractor
Dim sText As String
sText = te.ExtractText("C:\Test.doc", 64 * 1024)
```
