==================
 Boring Protocols
==================
:tags: java, oop
:category: java

.. contents::

之前5章看起来还真是 `Boring` 。

这一章的名称中虽然有 `Boring` ，但是内容却是很有趣的。

.. tip::

   这里的 `Protocols` 我觉得应该指的是 `Interface` 的意思。

----------------------------------------

本章接着上一章节的最后代码继续讲解。

这次将 `remFn`  `substFn` 放入到参数的位置。

.. code-block:: java

   abstract class PieD {
       abstract PieD rem(RemV remFn, Object o);
       abstract PieD subst(SubstV substFn, Object n, Object o);
   }

   class Top extends PieD {
       Object t;
       PieD r;
       Top(Object _t, PieD _r) {
           t = _t;
           r = _r;
       }
       PieD rem(RemV remFn, Object o) {
           return remFn.forTop(t, r, o);
       }
       PieD subst(SubstV substFn, Object n, Object o) {
           return sbustFn.forTop(t, r, n, o);
       }
   }

   class Bot extends PieD {
       Object t;
       PieD r;
       Bot(Object _t, PieD _r) {
           t = _t;
           r = _r;
       }
       PieD rem(RemV remFn, Object o) {
           return remFn.forBot(t, r, o);
       }
       PieD subst(SubstV substFn, Object n, Object o) {
           return sbustFn.forBot(t, r, n, o);
       }
   }

引入 `this` 关键字，指代访问者本身，同步修改对应的访问者类。

.. code-block:: java

   class RemV {
       PieD forBot(Object o){
           return new Bot();
       }
       PieD forTop(Object t, PieD r, Object o){
           if (o.equals(t))
               return r.rem(this, o);
           else
               return new Top(t, r.rem(this, o));
       }
   }

   class SubstV {
       PieD forBot(Object n, Object o){
           return new Bot();
       }
       PieD forTop(Object t, PieD r, Object n, Object o){
           if (o.equals(t))
               return new Top(n, r.subst(this, n, o));
           else
               return new Top(t, r.subst(this, n, o));
       }
   }

再修改访问者类，将更多的参数传入到访问者类中。

.. code-block:: java

   class RemV {
       Object o;
       RemV(Object _o) {
           o = _o;
       }
       PieD forBot(Object o){
           return new Bot();
       }
       PieD forTop(Object t, PieD r){
           if (o.equals(t))
               return r.rem(this);
           else
               return new Top(t, r.rem(this));
       }
   }

   class SubstV {
       Object n;
       Object o;
       SubstV(Object _n, Object _o){
           n = _n;
           o = _o;
       }

       PieD forBot(Object n, Object o){
           return new Bot();
       }
       PieD forTop(Object t, PieD r){
           if (o.equals(t))
               return new Top(n, r.subst(this));
           else
               return new Top(t, r.subst(this));
       }
   }

上面的形式就有点函数式编程里面的 **闭包** 的意味了。

好了，根据上面修改后的 `SubstV` ，重新修改一下 `PieD` 及其 `Bot` 和 `Top`

.. code-block:: java

   abstract class PieD {
       abstract PieD rem(RemV remFn);
       abstract PieD subst(SubstV substFn);
   }

   class Top extends PieD {
       Object t;
       PieD r;
       Top(Object _t, PieD _r) {
           t = _t;
           r = _r;
       }
       PieD rem(RemV remFn) {
           return remFn.forTop(t, r);
       }
       PieD subst(SubstV substFn) {
           return sbustFn.forTop(t, r);
       }
   }

   class Bot extends PieD {
       Object t;
       PieD r;
       Bot(Object _t, PieD _r) {
           t = _t;
           r = _r;
       }
       PieD rem(RemV remFn) {
           return remFn.forBot();
       }
       PieD subst(SubstV substFn) {
           return sbustFn.forBot();
       }
   }

在 `Top` 和 `Bot` 类中， `rem` 和 `subst` 的代码都很类似，

所以我们可以进一步抽象。

.. warning::

   前方高能预警，高潮就要到来了。

.. code-block:: java

   // abstract class PieVisitorD {
   //     abstract PieD forBot();
   //     abstract PieD forTop(Object t, PieD r);
   // }

   interface PieVisitorI {
       PieD forBot();
       PieD forTop(Object t, PieD r);
   }

   class RemV implements PieVisitorI {
       Object o;
       RemV(Object _o) {
           o = _o;
       }
       public PieD forBot() {
           return new Bot();
       }
       public PieD forTop(Object t, PieD r) {
           if (o.equals(t))
               return r.accept(this);
           else
               return new Top(t, r.accept(this));
       }
   }

   class SbustV implements PieVisitorI {
       Object n;
       Object o;
       SubstV(Object _n, Object _o) {
           n = _n;
           o = _o;
       }
       public PieD forBot() {
           return new Bot();
       }
       public PieD fotTop(Object t, PieD r) {
           if (o.equals(t))
               return new Top(n, r.accept(this));
           else
               return new Top(t, r.accept(this));
       }
   }

同步修改 `PieD`

.. code-block:: java

   abstract class PieD {
       abstract PieD accept(PieVisitorI ask);
   }

   class Bot extends PieD {
       PieD accept(PieVisitorI ask) {
           return ask.forBot();
       }
   }

   class Top extends PieD {
       Object t;
       PieD r;
       Top(Object _t, PieD _r) {
           t = _t;
           r = _r;
       }
       PieD accept(PieVisitorI ask) {
           return ask.forTop(t, r);
       }
   }

.. code-block:: java

   // 有限制次数的替换
   class LtdSubstV implements PieVisitorI {
       int c;
       Object n;
       Object o;
       LtdSubstV(int _c, Object _n, Object _o) {
           c = _c;
           n = _n;
           o = _o;
       }
       public PieD forBot() {
           return new Bot();
       }
       public PieD forTop(Object t, PieD r) {
           if (c == 0)
               return new Top(t, r);
           else
               if (o.equals(t))
                   return new Top(n, r.accept(LtdSubstV(c-1, n, o)));
               else
                   return new Top(t, r.accept(this));
       }
   }

.. tip::

   **第六条建议**

   When the additional consumed values

   change for a self-referenced use of a

   visitor, don't forget to create a new visitor.
