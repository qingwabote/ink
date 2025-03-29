# The consistency of floating point
[Consistency: how to defeat the purpose of IEEE floating point](https://yosefk.com/blog/consistency-how-to-defeat-the-purpose-of-ieee-floating-point.html)

## Javascript
[The number type, representing the double-precision 64-bit format IEEE 754-2008 values as specified in the IEEE Standard for Binary Floating-Point Arithmetic, except NaN...](https://262.ecma-international.org/6.0/index.html#sec-ecmascript-language-types-number-type)

[Addition is a commutative operation, but not always associative.The result of an addition is determined using the rules of IEEE 754-2008 binary double-precision arithmetic:...](https://262.ecma-international.org/6.0/index.html#sec-applying-the-additive-operators-to-numbers), 这里提到“不总是满足结合律”，以及“运算结果由IEEE 754-2008决定”，是否可以理解为遵守了“IEEE 754-2008 10.4 Literal meaning and value-changing optimizations”？

[The math, the choice of algorithms is left to the implementation, just recommended that implementations use the approximation algorithms for IEEE 754-2008 arithmetic](https://262.ecma-international.org/6.0/index.html#sec-function-properties-of-the-math-object), 意味着计算诸如 sin, cos, tan 等等需要使用自己实现的数学库代替原生的，无独有偶 [Rapier determinism](https://rapier.rs/docs/user_guides/rust/determinism/) 推荐使用 rust 实现的 nalgebra.

### [dropping support for x87](https://lists.webkit.org/pipermail/webkit-dev/2019-March/030569.html)
https://bugs.webkit.org/show_bug.cgi?id=194853

[How to get identical results](https://learn.microsoft.com/en-us/cpp/porting/floating-point-migration-issues?view=msvc-170#how-to-get-identical-results)

## Webassembly
https://rapier.rs/docs/user_guides/rust/determinism/

## C Sharp
https://discussions.unity.com/t/state-of-determinism-in-unity/867770

*http://www.dsc.ufcg.edu.br/~cnum/modulos/Modulo2/IEEE754_2008.pdf*