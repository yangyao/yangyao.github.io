---
title: 结构化CSS
tags:
  - DNN实习
  - DNN
  - DNN皮肤
date: 2011-08-08 10:36:10
---

如何增加css代码的复用率，这个制约着开发的效率。 这么多天学习DNN皮肤，我懂得了一点就是结构化。在DNN皮肤的制作中，我发现了DNN在写html和css代码的连个很大的优点。 其中一个就是固定的一个HTML代码块，叫做pane结构，还有一个代码块叫做container，container代码是书写在pane结构中的。 在书写了很长时间的CSS之后很多人都会发现自己写的结构放佛是固定的。 从上到下，一次是head,logo menu,然后是banner，下面再是某种左右或者上下结构。。 在DNN中pane结构就是用了解决下面的问题的，pane结构大致是这样子的。 &#60;div id=&#8221;allPane&#8221;&#62; &#60;div id=&#8221;TopPane&#8221; runat=&#8221;server&#8221; visible=&#8221;false&#8221;&#62;&#60;/div&#62; &#60;table border=&#8221;0&#8243; cellspacing=&#8221;0&#8243; cellpadding=&#8221;0&#8243; width=&#8221;100%&#8221;&#62; &#60;tr&#62; &#60;td id=&#8221;LeftPane&#8221; runat=&#8221;server&#8221; visible=&#8221;false&#8221;&#62;&#60;/td&#62; &#60;td&#62; &#60;div id=&#8221;HeadPane&#8221; runat=&#8221;server&#8221; visible=&#8221;false&#8221;&#62;&#60;/div&#62; &#60;div&#62; &#60;div id=&#8221;CenterTopPane_AB_A&#8221;&#62;&#60;div id=&#8221;CenterTopPane_A&#8221; runat=&#8221;server&#8221;&#62;&#60;/div&#62;&#60;/div&#62; &#60;div id=&#8221;CenterTopPane_AB_B&#8221;&#62;&#60;div id=&#8221;CenterTopPane_B&#8221; runat=&#8221;server&#8221;&#62;&#60;/div&#62;&#60;/div&#62; &#60;/div&#62; &#60;table border=&#8221;0&#8243; cellspacing=&#8221;0&#8243; cellpadding=&#8221;0&#8243; width=&#8221;100%&#8221;&#62; &#60;tr&#62; &#60;td id=&#8221;MiddleLeftPane_A&#8221; runat=&#8221;server&#8221; visible=&#8221;false&#8221;&#62;&#60;/td&#62; &#60;td id=&#8221;MiddleCenterPane_A&#8221; runat=&#8221;server&#8221;&#62;&#60;/td&#62; &#60;td id=&#8221;MiddleRightPane_A&#8221; runat=&#8221;server&#8221; visible=&#8221;false&#8221;&#62;&#60;/td&#62; [...]