[](# [Conversion and Promotion](@id conversion-and-promotion))
# [変換と昇格](@id conversion-and-promotion)



```@raw html
<!--
Julia has a system for promoting arguments of mathematical operators to a common type, which has
been mentioned in various other sections, including [Integers and Floating-Point Numbers](@ref),
[Mathematical Operations and Elementary Functions](@ref), [Types](@ref man-types), and [Methods](@ref).
In this section, we explain how this promotion system works, as well as how to extend it to new
types and apply it to functions besides built-in mathematical operators. Traditionally, programming
languages fall into two camps with respect to promotion of arithmetic arguments:
-->
```
Juliaには、[整数と浮動小数点数](@ref)、[算術処理と基本的な関数](@ref)、[型](@ref man-types)、[メソッド](@ref)、その他のさまざまな章に記述されているように、算術演算子の引数を共通の型に昇格するシステムが備わっています。
この章では、この昇格システムが動作するしくみや、昇格システムを新しい型に拡張して標準装備の算術演算子や関数でも動作させる方法について説明します。通常、プログラミング言語は算術演算の引数がどのように昇格するかによって、2つの陣営に分類されています。

 
 ```@raw html
 <!--
  * **Automatic promotion for built-in arithmetic types and operators.** In most languages, built-in
    numeric types, when used as operands to arithmetic operators with infix syntax, such as `+`,
    `-`, `*`, and `/`, are automatically promoted to a common type to produce the expected results.
    C, Java, Perl, and Python, to name a few, all correctly compute the sum `1 + 1.5` as the floating-point
    value `2.5`, even though one of the operands to `+` is an integer. These systems are convenient
    and designed carefully enough that they are generally all-but-invisible to the programmer: hardly
    anyone consciously thinks of this promotion taking place when writing such an expression, but
    compilers and interpreters must perform conversion before addition since integers and floating-point
    values cannot be added as-is. Complex rules for such automatic conversions are thus inevitably
    part of specifications and implementations for such languages.
-->
 ```
 *  **標準装備の算術型と演算子の自動昇格** ほとんどの言語では、標準装備の数値型が、`+`、 `-`、`*`、`/`などの中置記法の算術演算子の被演算子として使われるときは、自動的に共通の型に昇格してから、演算結果が生成されます。少し例を挙げると、 C、Java、Perl、Pythonなどはすべて、`1 + 1.5`の合計を、浮動小数点値の`2.5`として正しく計算することができますが、`+`の被演算子の一方は整数です。こういうシステムは便利であり、通常はプログラマにはまったく見えないように慎重に設計されています。このような式を書くときに昇格が起こっていると意識する人はほとんどいませんが、コンパイラやインタプリタは、足し算を行う前に変換を必ずおこないます。整数と浮動小数点数はそのままでは足せないからです。このような自動変換の複雑な規則は、必然的にこの陣営の言語では仕様や実装の一部となります。

 
 ```@raw html
 <!--
   * **No automatic promotion.** This camp includes Ada and ML -- very "strict" statically typed languages.
    In these languages, every conversion must be explicitly specified by the programmer. Thus, the
    example expression `1 + 1.5` would be a compilation error in both Ada and ML. Instead one must
    write `real(1) + 1.5`, explicitly converting the integer `1` to a floating-point value before
    performing addition. Explicit conversion everywhere is so inconvenient, however, that even Ada
    has some degree of automatic conversion: integer literals are promoted to the expected integer
    type automatically, and floating-point literals are similarly promoted to appropriate floating-point
    types.
-->
 ```
   * **非自動昇格**この陣営にはAdaとMLなどの非常に「厳密」な静的に型付けされた言語があります。こういった言語では、すべての変換をプログラマが明示的に指定する必要があります。したがって、例に挙げた`1 + 1.5`という式は、AdaやMLでは共にコンパイルエラーになります。これを避けるには`real(1) + 1.5`のように書いて、足し算を実行する前に整数`1`を浮動小数点数に明示的に変換する必要があります。しかし、常に明示的に変換しなければならないのは不便なので、Adaでさえもある程度の自動変換が行われます。整数リテラルは期待されるように整数型に自動的に昇格され、浮動小数点リテラルも同様に適切な浮動小数点型に昇格されます。



```@raw html
<!--
In a sense, Julia falls into the "no automatic promotion" category: mathematical operators are
just functions with special syntax, and the arguments of functions are never automatically converted.
However, one may observe that applying mathematical operations to a wide variety of mixed argument
types is just an extreme case of polymorphic multiple dispatch -- something which Julia's dispatch
and type systems are particularly well-suited to handle. "Automatic" promotion of mathematical
operands simply emerges as a special application: Julia comes with pre-defined catch-all dispatch
rules for mathematical operators, invoked when no specific implementation exists for some combination
of operand types. These catch-all rules first promote all operands to a common type using user-definable
promotion rules, and then invoke a specialized implementation of the operator in question for
the resulting values, now of the same type. User-defined types can easily participate in this
promotion system by defining methods for conversion to and from other types, and providing a handful
of promotion rules defining what types they should promote to when mixed with other types.
-->
```

Juliaでは、算術演算子は特殊な構文を持つ関数に過ぎず、関数の引数は決して自動的に変換されません。
その意味では、Juliaは「非自動昇格」の陣営に分類されるでしょう。
しかし、様々な型の混合した引数に算術演算を適用することは、多相的な多重ディスパッチの極端な事例に過ぎません。これは型によるディスパッチをおこなうJuliaのシステムに非常に適しています。
算術演算における被演算子の「自動」昇格は、特殊な適用がおきただけです。Juliaには、算術演算子を全捕捉するディスパッチ規則が事前に定義されており、被演算子の型の組み合わせに対して特化した実装が存在しないときに呼び出されます。
この全捕捉規則は、まずユーザーが定義できる昇格規則を利用してすべての被演算子を共通の型に昇格し、こうして同一の型となった演算結果の型に特化した演算子の実装を呼び出します。
ユーザーの定義した型もこの昇格システムに簡単に追加できます。
ユーザー定義型に対して、他の型との変換メソッドを相互に定義し、他の型が混在する場合にどの型に昇格するかを定義する少数の昇格規則を決めてやればいいのです。

[](## Conversion)
## 変換


```@raw html
<!--
Conversion of values to various types is performed by the `convert` function. The `convert` function
generally takes two arguments: the first is a type object while the second is a value to convert
to that type; the returned value is the value converted to an instance of given type. The simplest
way to understand this function is to see it in action:
-->
```
値をさまざまな型に変換するのは、`convert`関数です。
この`convert`関数は通常、2つの引数をとります。1番目は型オブジェクトで、2番目はその型に変換される値です。
戻り値は指定された型のインスタンスに変換された値です。
この関数を理解する最も簡単な方法は、実際の動作を確認することです。


```jldoctest
julia> x = 12
12

julia> typeof(x)
Int64

julia> convert(UInt8, x)
0x0c

julia> typeof(ans)
UInt8

julia> convert(AbstractFloat, x)
12.0

julia> typeof(ans)
Float64

julia> a = Any[1 2 3; 4 5 6]
2×3 Array{Any,2}:
 1  2  3
 4  5  6

julia> convert(Array{Float64}, a)
2×3 Array{Float64,2}:
 1.0  2.0  3.0
 4.0  5.0  6.0
```


```@raw html
<!--
Conversion isn't always possible, in which case a no method error is thrown indicating that `convert`
doesn't know how to perform the requested conversion:
-->
```
変換は常に可能であるとは限りません。そんな時は、ノーメソッドエラーが投げられて、`convert`関数は要求された変換の実行方法を知らないことを通知します。

```jldoctest
julia> convert(AbstractFloat, "foo")
ERROR: MethodError: Cannot `convert` an object of type String to an object of type AbstractFloat
This may have arisen from a call to the constructor AbstractFloat(...),
since type constructors fall back to convert methods.
```


```@raw html
<!--
Some languages consider parsing strings as numbers or formatting numbers as strings to be conversions
(many dynamic languages will even perform conversion for you automatically), however Julia does
not: even though some strings can be parsed as numbers, most strings are not valid representations
of numbers, and only a very limited subset of them are. Therefore in Julia the dedicated `parse()`
function must be used to perform this operation, making it more explicit.
-->
```
言語の中には文字列を解析して数値とみなしたり、書式付きの数値を変換して文字列とみなしたりするものがありますが、（多くの動的言語では自動的に変換が行われます）、Juliaはそうではありません。文字列の中には数値として解析できるものもありますが、ほとんどの文字列は数値としては妥当な表現ではなく、非常に限られた一部のみです。そのため、Juliaでは、こういった操作は、専用の関数`parse()` を使って明示的に行う必要があります。

[](### Defining New Conversions)
### 新しい変換の定義


```@raw html
<!--
To define a new conversion, simply provide a new method for `convert()`. That's really all there
is to it. For example, the method to convert a real number to a boolean is this:
-->
```

新しく変換を定義するには、`convert()`に新たなメソッドを加えるだけです。それがすべてです。たとえば、実数をブール値に変換するメソッドは次のとおりです。



```julia
convert(::Type{Bool}, x::Real) = x==0 ? false : x==1 ? true : throw(InexactError())
```


```@raw html
<!--
The type of the first argument of this method is a [singleton type](@ref man-singleton-types),
`Type{Bool}`, the only instance of which is [`Bool`](@ref). Thus, this method is only invoked
when the first argument is the type value `Bool`. Notice the syntax used for the first
argument: the argument name is omitted prior to the `::` symbol, and only the type is given.
This is the syntax in Julia for a function argument whose type is specified but whose value
is never used in the function body. In this example, since the type is a singleton, there
would never be any reason to use its value within the body. When invoked, the method
determines whether a numeric value is true or false as a boolean,
by comparing it to one and zero:
-->
```
このメソッドの1番目の引数の型は[シングルトン型]（@ ref man-singleton-types）の`Type{Bool}`で、この型のインスタンスは[`Bool`](@ref)だけです。
したがって、このメソッドは、最初の引数が型の値を示す`Bool`である場合にのみ呼び出されます。
最初の引数の構文に注目してください。
`::`という記号の前にあるはずの引数名は省略され、型だけが指定されています。
これはJuliaの関数の構文で、引数の型は指定するけれども、引数の値は関数本体でまったく使わない時に用います。
この例では、型はシングルトンであるため、その値を関数本体内で使う理由がありません。
このメソッドを呼び出すと、数値を1や0と比較して、真偽値を決定します。



```jldoctest
julia> convert(Bool, 1)
true

julia> convert(Bool, 0)
false

julia> convert(Bool, 1im)
ERROR: InexactError()
Stacktrace:
 [1] convert(::Type{Bool}, ::Complex{Int64}) at ./complex.jl:31

julia> convert(Bool, 0im)
false
```


```@raw html
<!--
The method signatures for conversion methods are often quite a bit more involved than this example,
especially for parametric types. The example above is meant to be pedagogical, and is not the
actual Julia behaviour. This is the actual implementation in Julia:
-->
```
変換メソッドのシグネチャは、この例よりもかなり複雑です。特にパラメトリック型のシグネチャは複雑です。上記の例は教育用で、実際のJuliaの動作ではありません。実際のJuliaの実装はこうです。


```julia
convert(::Type{T}, z::Complex) where {T<:Real} =
    (imag(z) == 0 ? convert(T, real(z)) : throw(InexactError()))
```

[](### [Case Study: Rational Conversions](@id man-rational-conversion))
### [事例研究: 有理数の変換](@id man-rational-conversion)

```@raw html
<!--
To continue our case study of Julia's [`Rational`](@ref) type, here are the conversions declared in
[`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl),
right after the declaration of the type and its constructors:
-->
```
Juliaの[`Rational`](@ref)型の事例研究を継続し、ここでは、[`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)で宣言されている変換を見てみましょう。これは、型とコンストラクタの宣言の直後に書かれています。

```julia
convert(::Type{Rational{T}}, x::Rational) where {T<:Integer} = Rational(convert(T,x.num),convert(T,x.den))
convert(::Type{Rational{T}}, x::Integer) where {T<:Integer} = Rational(convert(T,x), convert(T,1))

function convert(::Type{Rational{T}}, x::AbstractFloat, tol::Real) where T<:Integer
    if isnan(x); return zero(T)//zero(T); end
    if isinf(x); return sign(x)//zero(T); end
    y = x
    a = d = one(T)
    b = c = zero(T)
    while true
        f = convert(T,round(y)); y -= f
        a, b, c, d = f*a+c, f*b+d, a, b
        if y == 0 || abs(a/b-x) <= tol
            return a//b
        end
        y = 1/y
    end
end
convert(rt::Type{Rational{T}}, x::AbstractFloat) where {T<:Integer} = convert(rt,x,eps(x))

convert(::Type{T}, x::Rational) where {T<:AbstractFloat} = convert(T,x.num)/convert(T,x.den)
convert(::Type{T}, x::Rational) where {T<:Integer} = div(convert(T,x.num),convert(T,x.den))
```


```@raw html
<!--
The initial four convert methods provide conversions to rational types. The first method converts
one type of rational to another type of rational by converting the numerator and denominator to
the appropriate integer type. The second method does the same conversion for integers by taking
the denominator to be 1. The third method implements a standard algorithm for approximating a
floating-point number by a ratio of integers to within a given tolerance, and the fourth method
applies it, using machine epsilon at the given value as the threshold. In general, one should
have `a//b == convert(Rational{Int64}, a/b)`.
-->
```

最初の4つの変換メソッドは、有理数型への変換を行います。
第1のメソッドは、分子と分母を適切な整数型に変換して、ある有理数の型から別の有理数の型に変換します。
第2のメソッドは、分母に1を置いて、整数に対して同様な変換を行います。
第3のメソッドは、整数の比を与えられた許容誤差内で浮動小数点数に近似する標準的なアルゴリズムを実装して、
第4のメソッドは、与えられた計算機イプシロンを閾値として第3のメソッドを適用します。
概ね、`a//b == convert(Rational{Int64}, a/b)`という式が成り立たなければなりません。


```@raw html
<!--
The last two convert methods provide conversions from rational types to floating-point and integer
types. To convert to floating point, one simply converts both numerator and denominator to that
floating point type and then divides. To convert to integer, one can use the `div` operator for
truncated integer division (rounded towards zero).
-->
```
最後の2つの変換メソッドは、有理数型から浮動小数点型および整数型への変換を行います。
浮動小数点数に変換するには、分子と分母の両方を浮動小数点数型に変換してから除算を行うだけです。
整数に変換するには、整数の除算を切り捨てる（ゼロに丸める）`div`演算子を使用できます。

[](## Promotion)
## 昇格


```@raw html
<!--
Promotion refers to converting values of mixed types to a single common type. Although it is not
strictly necessary, it is generally implied that the common type to which the values are converted
can faithfully represent all of the original values. In this sense, the term "promotion" is appropriate
since the values are converted to a "greater" type -- i.e. one which can represent all of the
input values in a single common type. It is important, however, not to confuse this with object-oriented
(structural) super-typing, or Julia's notion of abstract super-types: promotion has nothing to
do with the type hierarchy, and everything to do with converting between alternate representations.
For instance, although every [`Int32`](@ref) value can also be represented as a [`Float64`](@ref) value,
`Int32` is not a subtype of `Float64`.
-->
```
昇格とは、型の混在した複数の値を単一の共通の型に変換することを指します。
値の変換される共通の型は元の値をすべて忠実に表現できるという条件は、必須ではありませんが、通常は満たすことが想定されています。
この意味では、「昇格」という用語は適切で、値は「より大きな」型に変換されます。
そして、すべての入力値を単一の共通型で表すことができます。
しかし重要なことですが、昇格をオブジェクト指向の（構造的）スーパータイプやJuliaの抽象型のスーパータイプという概念と混同しないでください。
昇格は型の階層とは関連がなく、代替的な表現どうしの変換のみに関連する概念です。
たとえば、すべての [`Int32`](@ref) の値は[`Float64`](@ref)の値として表現できますが、 `Int32`は`Float64`のサブタイプではありません。



```@raw html
<!--
Promotion to a common "greater" type is performed in Julia by the `promote` function, which takes
any number of arguments, and returns a tuple of the same number of values, converted to a common
type, or throws an exception if promotion is not possible. The most common use case for promotion
is to convert numeric arguments to a common type:
-->
```

共通の「より大きな」型に昇格するには、Juliaでは`promote`関数を使います。
この関数は、任意の数の引数をとって、同じ数の共通の型に変換された値のタプルを返し、また昇格が不可能な場合は例外を投げます。昇格の一番よくある使い方は、数値の引数を共通の型に変換することです。

```jldoctest
julia> promote(1, 2.5)
(1.0, 2.5)

julia> promote(1, 2.5, 3)
(1.0, 2.5, 3.0)

julia> promote(2, 3//4)
(2//1, 3//4)

julia> promote(1, 2.5, 3, 3//4)
(1.0, 2.5, 3.0, 0.75)

julia> promote(1.5, im)
(1.5 + 0.0im, 0.0 + 1.0im)

julia> promote(1 + 2im, 3//4)
(1//1 + 2//1*im, 3//4 + 0//1*im)
```


```@raw html
<!--
Floating-point values are promoted to the largest of the floating-point argument types. Integer
values are promoted to the larger of either the native machine word size or the largest integer
argument type. Mixtures of integers and floating-point values are promoted to a floating-point
type big enough to hold all the values. Integers mixed with rationals are promoted to rationals.
Rationals mixed with floats are promoted to floats. Complex values mixed with real values are
promoted to the appropriate kind of complex value.
-->
```

浮動小数点数は、浮動小数点数の引数の型の中で最大のものに昇格されます。
整数値は、ネイティブマシンのワードサイズと整数の引数の型の最大のものとのどちらか大きい方に昇格されます。
整数と浮動小数点数の混合した場合は、すべての値を保持するのに十分な大きさの浮動小数点型に昇格されます。
整数と有理数の混合した場合は有理数に昇格されます。
有理数と浮動小数点数の混合した場合は、浮動小数点数に昇格されます。
複素数と実数の混合した場合は、複素数の適切な型に昇格されます。

```@raw html
<!--
That is really all there is to using promotions. The rest is just a matter of clever application,
the most  "clever" application being the definition of catch-all methods for numeric operations
like the arithmetic operators `+`, `-`, `*` and `/`. Here are some of the catch-all method definitions
given in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl):
-->
```
昇格の使用法についてはこれがすべてです。あとはうまい適用の話だけです。
最も「うまい」適用は、 `+`、 `-`、 `*` 、`/`のような算術演算子用の全捕捉メソッドの定義です。
ここで [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)で定義された
全捕捉メソッドの一部を見てみましょう。

```julia
+(x::Number, y::Number) = +(promote(x,y)...)
-(x::Number, y::Number) = -(promote(x,y)...)
*(x::Number, y::Number) = *(promote(x,y)...)
/(x::Number, y::Number) = /(promote(x,y)...)
```


```@raw html
<!--
These method definitions say that in the absence of more specific rules for adding, subtracting,
multiplying and dividing pairs of numeric values, promote the values to a common type and then
try again. That's all there is to it: nowhere else does one ever need to worry about promotion
to a common numeric type for arithmetic operations -- it just happens automatically. There are
definitions of catch-all promotion methods for a number of other arithmetic and mathematical functions
in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl), but beyond
that, there are hardly any calls to `promote` required in the Julia standard library. The most
common usages of `promote` occur in outer constructors methods, provided for convenience, to allow
constructor calls with mixed types to delegate to an inner type with fields promoted to an appropriate
common type. For example, recall that [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)
provides the following outer constructor method:
-->
```
これらのメソッド定義では、数値のペアに対して、例えば、加算、減算、乗算、除算を行う、より特化した規則がない場合、値を共通型に昇格してから演算を行います。
昇格を行うのはこれらだけです。他の場所で算術演算の共通の数値型への昇格を心配する必要はありません。
自動的に処理されます。
他にいくつもの算術関数や数学関数の全捕捉昇格メソッドの定義が[`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)にありますが、
それら以外で、Juliaの標準ライブラリで必要となる`promote`の呼び出しはほとんどありません。
外部コンストラクターメソッドで一番よく見かける`promote`の使い方は、利便性をあげるため、異なる型が混ざったコンストラクター呼び出しを可能にすることでしょう。
これば、引数を適切な共通型に昇格し、その型をフィールドに持つ内部型に、コンストラクター呼び出しを委譲して実現します。
たとえば [`rational.jl`](https://github.com/JuliaLang/julia/blob/master/base/rational.jl)では、以下のような外部コンストラクタメソッドが利用可能なことを思い出してください。

```julia
Rational(n::Integer, d::Integer) = Rational(promote(n,d)...)
```


```@raw html
<!--
This allows calls like the following to work:
-->
```
これにより、次のような呼び出しが可能になります。

```jldoctest
julia> Rational(Int8(15),Int32(-5))
-3//1

julia> typeof(ans)
Rational{Int32}
```


```@raw html
<!--
For most user-defined types, it is better practice to require programmers to supply the expected
types to constructor functions explicitly, but sometimes, especially for numeric problems, it
can be convenient to do promotion automatically.
-->
```

大抵のユーザー定義型では、プログラマーがコンストラクター関数に想定される型を明示的に指定するのがよいと思いますが、時には自動昇格を使うと、特に数値問題の場合は、便利なものです。

[](### Defining Promotion Rules)
### 昇格規則の定義


```@raw html
<!--
Although one could, in principle, define methods for the `promote` function directly, this would
require many redundant definitions for all possible permutations of argument types. Instead, the
behavior of `promote` is defined in terms of an auxiliary function called `promote_rule`, which
one can provide methods for. The `promote_rule` function takes a pair of type objects and returns
another type object, such that instances of the argument types will be promoted to the returned
type. Thus, by defining the rule:
-->
```

原理的には`promote`関数のメソッドを直接定義することができますが、そのためには、すべてのおこりうる引数の型の置換に対して多くの冗長な定義が必要になります。
その代わりに、`promote`関数の挙動の定義を、`promote_rule`という補助的な関数のメソッドを定義して行うことができます。
この`promote_rule`関数は、型オブジェクトのペアを引数に取って、別の型オブジェクトを返しますが、
これは引数の型のインスタンスが戻り値の型に昇格されることを意味しているので、そうなるように規則を定義します。


```julia
promote_rule(::Type{Float64}, ::Type{Float32}) = Float64
```


```@raw html
<!--
one declares that when 64-bit and 32-bit floating-point values are promoted together, they should
be promoted to 64-bit floating-point. The promotion type does not need to be one of the argument
types, however; the following promotion rules both occur in Julia's standard library:
-->
```

64ビット浮動小数点数と32ビット浮動小数点数を一緒に昇格するときは、64ビット浮動小数点数に昇格する必要があります。
しかし昇格後の型は引数の型の1つである必要はありません。
次の昇格規則は共にJuliaの標準ライブラリにあるものです。

```julia
promote_rule(::Type{UInt8}, ::Type{Int8}) = Int
promote_rule(::Type{BigInt}, ::Type{Int8}) = BigInt
```


```@raw html
<!--
In the latter case, the result type is [`BigInt`](@ref) since `BigInt` is the only type
large enough to hold integers for arbitrary-precision integer arithmetic. Also note that
one does not need to define both `promote_rule(::Type{A}, ::Type{B})` and
`promote_rule(::Type{B}, ::Type{A})` -- the symmetry is implied by the way `promote_rule`
is used in the promotion process.
-->
```

後者の場合、昇格後の型は[`BigInt`](@ref)になります。
というのも`BigInt`だけが、任意の精度の整数演算に対して整数を保持する大きさを持つの唯一の型だからです。
`promote_rule(::Type{A}, ::Type{B})`と `promote_rule(::Type{B}, ::Type{A})`の両方を定義する必要はない点に注意してください。`promote_rule`は、昇格の処理の際に、対称性を前提として使われます。


```@raw html
<!--
The `promote_rule` function is used as a building block to define a second function called `promote_type`,
which, given any number of type objects, returns the common type to which those values, as arguments
to `promote` should be promoted. Thus, if one wants to know, in absence of actual values, what
type a collection of values of certain types would promote to, one can use `promote_type`:
-->
```
`promote_rule`関数を構成要素として使って、`promote_type`という二次的な関数を定義します。
`promote_type`関数は、任意の数の型オブジェクトを引数にとり、これらの値の共通の型を返します。
この型が`promote`関数が引数を昇格後にとるべき型となります。
したがって、実際の値が存在しなくても、`promote_type`を使えば、型の値の集合が昇格するとどんな型になるかを調べることができます。


```jldoctest
julia> promote_type(Int8, UInt16)
Int64
```


```@raw html
<!--
Internally, `promote_type` is used inside of `promote` to determine what type argument values
should be converted to for promotion. It can, however, be useful in its own right. The curious
reader can read the code in [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl),
which defines the complete promotion mechanism in about 35 lines.
-->
```

内部的には、`promote_type`は`promote`の内部で、引数値を昇格してどんな型に変換するかを決定するために使用されますが、`promote_type`単体でも有用なことがあります。
興味のある読者は約35行で完全な昇格の仕組みを定義するコードを [`promotion.jl`](https://github.com/JuliaLang/julia/blob/master/base/promotion.jl)で読むことができます。


[](### Case Study: Rational Promotions)
### 事例研究: 有理数の昇格


```@raw html
<!--
Finally, we finish off our ongoing case study of Julia's rational number type, which makes relatively
sophisticated use of the promotion mechanism with the following promotion rules:
-->
```
とうとう、ここまで進めてきたJuliaの有理数型の事例研究が完成します。ここでは、以下の昇格規則による昇格メカニズムを比較的洗練された手法で利用しています。

```julia
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{Rational{S}}) where {T<:Integer,S<:Integer} = Rational{promote_type(T,S)}
promote_rule(::Type{Rational{T}}, ::Type{S}) where {T<:Integer,S<:AbstractFloat} = promote_type(T,S)
```


```@raw html
<!--
The first rule says that promoting a rational number with any other integer type promotes to a
rational type whose numerator/denominator type is the result of promotion of its numerator/denominator
type with the other integer type. The second rule applies the same logic to two different types
of rational numbers, resulting in a rational of the promotion of their respective numerator/denominator
types. The third and final rule dictates that promoting a rational with a float results in the
same type as promoting the numerator/denominator type with the float.
-->
```

第1の規則は、有理数型と整数型を昇格すると、有理数型に昇格し、その分子/分母の型は元の有理数の分子/分母の型と整数型を昇格した型になることを示しています。
第2の規則は、2つの異なる有理数型を昇格すると、同様の論理を適用して、各有理数型の分子/分母の型を昇格した型を分子/分母の型とするような有理数型に昇格することを示しています。
最後の3つ目の規則は、有理数型と浮動小数点型を昇格すると、浮動小数点型に昇格し、その型は有理数型の分子/分母の型と浮動小数点数型を昇格した結果と同じ型になることを示しています。


```@raw html
<!--
This small handful of promotion rules, together with the [conversion methods discussed above],
are sufficient to make rational numbers interoperate completely naturally with all of Julia's
other numeric types -- integers, floating-point numbers, and complex numbers. By providing appropriate
conversion methods and promotion rules in the same manner, any user-defined numeric type can interoperate
just as naturally with Julia's predefined numerics.
-->
```

この少数の昇格規則と、[前述の変換メソッド]だけで十分、有理数型をとても自然にJuliaの他の数値型、つまり 整数、浮動小数点数、複素数と一緒に使うことができます。
同様に、適切な変換メソッドと昇格規則を定義すれば、どんなユーザー定義の数値型でも自然に、Juliaで事前定義されている数値型と一緒に使うことができます。
