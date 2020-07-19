---
title: C# Deferred Gotchas
date: Y2020-07-19 22:40:20 +0800
categories: [Tech, Csharp]
tags: [Csharp]
---

Linq? IEnumerable? We use it everyday for good reasons, performance, abstraction, developer productivity and etc. You name it. Normally everything works well until the "gotchas" of Linq and IEnumerable takes you into hours of "phantom" production debugging. The reason is because it does not produce any errors and code execution does not get interrupted like a "phantom". In this post, I'll try to explain the deferred nature of Linq and IEnumerable so we can be more aware when we use them and save us from hours of "phantom" bugs.