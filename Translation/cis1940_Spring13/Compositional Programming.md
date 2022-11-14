[![ACM Logo](https://dl.acm.org/pubs/lib/images/acm_logo.jpg)](http://www.acm.org/)

#Haskell #Functional #Compositional
# Compositional Programming

WEIXIN ZHANG, [](https://dl.acm.org/doi/fullHtml/10.1145/3460228#aff1)

University of Bristol, United Kingdom and The University of Hong Kong, China

YAOZHU SUN **and** [](https://dl.acm.org/doi/fullHtml/10.1145/3460228#aff3)

BRUNO C. D. S. OLIVEIRA, [](https://dl.acm.org/doi/fullHtml/10.1145/3460228#aff3)

The University of Hong Kong, China

  

ACM Trans. Program. Lang. Syst., Vol. 43, No. 3, Article 9, Publication date: August 2021.  
DOI: [https://doi.org/10.1145/3460228](https://doi.org/10.1145/3460228)

Modularity is a key concern in programming. However, programming languages remain limited in terms of modularity and extensibility. Small canonical problems, such as the Expression Problem (EP), illustrate some of the basic issues: the dilemma between choosing one kind of extensibility over another one in most programming languages. Other problems, such as how to express dependencies in a modular way, add up to the basic issues and remain a significant challenge.

This article presents a new statically typed modular programming style called _Compositional Programming_. In Compositional Programming, there is no EP: It is easy to get extensibility in multiple dimensions (i.e., it is easy to add new variants as well as new operations). Compositional Programming offers an alternative way to model data structures that differs from both algebraic datatypes in functional programming and conventional OOP class hierarchies. We introduce four key concepts for Compositional Programming: _compositional interfaces_, _compositional traits_, _method patterns_, and _nested trait composition_. Altogether, these concepts allow us to naturally solve challenges such as the Expression Problem, model attribute-grammar-like programs, and generally deal with modular programs with _complex dependencies_. We present a language design, called CP , which is proved to be type-safe, together with several examples and three case studies.

**CCS Concepts:** **• Software and its engineering →** **Object oriented languages**;

  

**Additional Key Words and Phrases:** Expression problem, compositionality, traits

  

ACM Reference format:  
Weixin Zhang, Yaozhu Sun, and Bruno C. D. S. Oliveira. 2021. Compositional Programming. _ACM Trans. Program. Lang. Syst._ 43, 3, Article 9 (August 2021), 61 pages, DOI: [https://doi.org/10.1145/3460228](https://doi.org/10.1145/3460228).

## 1 INTRODUCTION

Modularity is a key concern in programming. Programming languages support the development of modular programs by providing language constructs for modularization. For instance, many languages support some notion of modules that can group various kinds of definitions and functions, and can be separately compiled. At a smaller scale, most **Object-Oriented Programming (OOP)** languages support _classes_, which can also be separately defined, compiled, and reused by subclassing.

An important aspect of programming is how to define data structures and the operations over those data structures. Different language designs offer different mechanisms for this purpose. OOP languages model data structures using class hierarchies and techniques such as the Composite pattern [Gamma et al. [1994](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0031)]. Typically, there is an interface that specifies all the operations (methods) of interest for the data structure. Multiple classes implement different types of nodes in the data structure, supporting all the operations in the interface. Many functional languages employ _algebraic datatypes_ [Burstall et al. [1981](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0015)] to model data structures, and use functions (typically defined by pattern matching) to model the operations over those data structures. For example, in Haskell, we can model a simple form of arithmetic expressions and their evaluation as:

![](https://dl.acm.org/cms/attachment/74c5469f-a388-4812-9cca-2eca3085b496/toplas4303-09-uf01.gif)

The datatype definition Exp defines a (binary) tree structure that models arithmetic expressions with two _constructors_ for numeric literals (Lit) and additions (Add). The eval function defines evaluation by pattern matching over values of Exp and calling itself recursively on the child nodes.

As widely acknowledged by the **Expression Problem (EP)** [Wadler [1998](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0083)], both algebraic datatypes and class hierarchies have modularity problems. With algebraic datatypes and pattern matching, adding new operations is easy, but adding constructors is hard. Conversely, the OOP approach makes it easy to add new classes, but makes it hard to add new methods at the same time.

This article presents a new statically typed modular programming style called _Compositional Programming_. In Compositional Programming, there is no EP: It is easy to get extensibility in two dimensions (i.e., it is easy to add new constructors as well as new operations). Compositional Programming offers an alternative way to model data structures that differs from both algebraic datatypes in functional programming and conventional OOP class hierarchies. Thus, compositional Programming can be viewed as an alternative programming paradigm, since programs are structured differently from traditional FP and OOP. The key ideas of Compositional Programming are implemented in a new programming language called CP . For example, the code for modeling arithmetic expressions can be expressed in CP as:

![](https://dl.acm.org/cms/attachment/f56099f6-f3bc-4c0d-b8a7-0b02fb36af77/toplas4303-09-uf02.gif)

In the CP code above, the definition ExpSig<Exp> (a compositional interface) plays a similar role to the algebraic datatype Exp in functional languages. The special type parameter Exp in ExpSig<Exp> is called a _sort_, and models types that represent datatypes in CP . All constructors in CP must have a return type that is a sort. Like in functional languages, in CP adding new operations is easy using a special form of _traits_ [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)]. Unlike functional programming, where adding new constructors to Exp is difficult and non-modular, in CP the addition of new constructors is also easy. We will later illustrate the modularity of CP in detail. In essence, there are four new key concepts in CP :

-    **Compositional interfaces** can be viewed as an extension of traditional OOP interfaces, such as those found in languages like Java or Scala. In addition to declaring method signatures, compositional interfaces also allow specification of the _constructor signatures_. In turn, this enables _programming the construction of objects against an interface_, instead of a concrete implementation. Compositional interfaces can be parametrized by sorts (such as Exp), which abstract over concrete datatypes, and are used to determine which kind of object is built.  
    
-    **Compositional traits** extend traditional traits [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)] and _first-class traits_ [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)]. Compositional traits allow not only the definition of virtual methods but also the definition of _virtual constructors_. They also allow nested traits, which are used to support _trait families_. Trait families are akin to class families in _family polymorphism_ [Ernst [2001](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0027)]. Both virtual constructor definitions and nested traits are not possible with Schärli et al. [[2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)]’s traits and previous models of first-class traits.  
    
-    **Method patterns**, such as (Lit n).eval, provide a lightweight syntax to define method implementations for nested traits, which arise from virtual constructors. This enables compact method definitions for trait families, which resembles programs defined by pattern matching in functional languages and programs written with attributes in _attribute grammars_ [Knuth [1968](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0048); , [1990](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0049)].  
    
-    **Nested trait composition** is the mechanism used to compose compositional traits. The foundations for this mechanism originate from nested composition, which has been investigated in recent calculi with disjoint intersection types and polymorphism [Oliveira et al. [2016](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0065); Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001); Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007); , [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)]. Our work shows how nested composition can smoothly be integrated into compositional traits. Nested trait composition plays a similar role to traditional class inheritance, but generalizes to the composition of nested traits. Thus, it enables a form of inheritance of whole hierarchies, similar to the forms of composition found in family polymorphism. Nested trait composition is _associative_ and _commutative_ [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)], just like the composition for traditional traits [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)].  
    

Altogether, these concepts allow us to solve various challenges naturally. For instance, they enable a very natural and simple solution to the EP. More interestingly, Compositional Programming can deal with harder modularity challenges, such as modeling interesting classes of attribute grammars or more complex programs, which often contain non-trivial dependencies.

Dependencies are an important concern for modularity: The weaker dependencies between program parts are, the more modular a program is. Realistic programs will depend on some other program parts. A common case of program dependencies is using library functions or functions defined in other parts of the program. A program may also depend on existing _types_ and _constructors_. For instance, in OOP, when using a class defined in another part of the program, we may need to refer to the class type and the constructor of the class to create an instance of the class. Unfortunately, in most languages, most dependencies introduce _strong coupling_ between various program parts. A simple example of a program with a dependency that introduces strong coupling in functional programming would be a simple pretty-printer for expressions, which depends on eval. In a functional language, it is straightforward to write such a program in a non-modular way:

![](https://dl.acm.org/cms/attachment/42691dd3-eac0-4e51-a89c-c23d8f2c19fc/toplas4303-09-uf03.gif)

Such definition of printChild is tightly coupled with the concrete implementation of eval. Moreover, both eval and printChild are non-extensible. For modularization to be effective, it is desirable to have _weak dependencies_. In a modular setting, the definition of printChild should depend only on the interface of eval, without sticking to a particular implementation. Also, both eval and printChild should be open to the addition of new cases.

In Compositional Programming, programs with weak dependencies can be expressed concisely, in a statically typed and safe way, and without giving up modularity. More generally, Compositional Programming offers a range of mechanisms to deal with modular programs with various _complex forms of dependencies_. In many existing solutions to the EP, dealing with such dependencies in a modular way is highly non-trivial or simply not possible. There are various _design patterns_ [Gamma et al. [1994](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0031)] partly addressing the problem of strong coupling by using existing programming language features. For example, the Abstract Factory pattern [Gamma et al. [1994](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0031)], _Object Algebras_ [Oliveira and Cook [2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)], _Polymorphic Embeddings_ [Hofer et al. [2008](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0038)], _Finally Tagless_ [Carette et al. [2009](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0018)], _Datatypes à la Carte_ [Swierstra [2008](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0077)], or the _Cake Pattern_ [Odersky and Zenger [2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0062)] provide ways to abstract over the construction of objects or datatypes. However, such patterns often result in heavily parametrized and boilerplate code. Moreover, the lack of sufficiently powerful composition mechanisms and mechanisms to express weaker forms of dependencies makes it hard to deal with dependencies in such design patterns [Zhang and Oliveira [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0086); Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066)].

The CP language is inspired by the SEDEL language [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)]. The main novelties over SEDEL are the four compositional programming mechanisms listed above: compositional interfaces, compositional traits, method patterns, and nested trait composition. Compositional Programming is partly inspired by _generalized Object Algebras_ [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066)], but the built-in language mechanisms make modular programming natural, fully statically typed, and without boilerplate code and excessive parametrization. We present several examples and three case studies in CP . We also introduce a technique called _polymorphic contexts_ to deal with components that require some form of context in a modular way. In turn, polymorphic contexts are helpful to model L-attributed grammars. Our first case study is on the design of an **Embedded Domain-Specific Language (EDSL)** for circuits [Hinze [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0036); Gibbons and Wu [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0033)]. This EDSL is interesting, because it has various extensions that can be modularly defined, as well as various dependencies between components. Our second case study is a mini interpreter, which is a larger study and can be extended in several ways. The last case study is an implementation of the C0 compiler, inspired by the work of Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)]. In this case study, various extensions can be formulated as attributes and those attributes contain non-trivial dependencies to other attributes.

Finally, we present a small calculus that captures the essence of CP . This calculus is shown to be type-safe via an elaboration to the F+iFi+ calculus [Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)], which is a recently proposed calculus that supports _disjoint intersection types_ [Oliveira et al. [2016](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0065)], _disjoint polymorphism_ [Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001)], and _nested composition_ [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)].

In summary, the contributions of our work are:

-    **Compositional Programming:** We propose a new programming style that encourages weaker dependencies and increases the modularity of programs. Compositional Programming eliminates the EP and can deal with modular programs with complex dependencies.  
    
-   **A language design for Compositional Programming:** We present a concrete language design in the form of the CP calculus. The semantics of this calculus is given by elaboration to the F+iFi+ calculus and we prove the _type safety_ of the elaboration.  
    
-    **Attribute Grammars in** **CP** **:** We show that CP is powerful enough to implement programs with attributes that are expressible in attribute grammars [Knuth [1968](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0048); , [1990](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0049)] in a concise and fully statically type-checked manner. Our technique is partly inspired by the encoding of Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)], but CP avoids explicit definitions of composition operators, which are necessary in Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)]’s encoding.  
    
-    **Polymorphic contexts:** We introduce a simple technique that combines disjoint polymorphism and Compositional Programming to allow for modular contexts in modular components.  
    
-    **Implementation, case studies, and examples:** We have an implementation of CP . We present several examples and three case studies in CP . Altogether, these examples and case studies illustrate how to naturally solve challenges such as the EP or modeling attribute-grammar-like programs. The implementation and case studies can be found in:  
    
    [https://github.com/wxzh/CP](https://github.com/wxzh/CP).
    
      
    

## 2 BACKGROUND

This section gives the necessary background to this article. We first review the well-known **Expression Problem (EP)** [Wadler [1998](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0083)] and then move on to the closely related work, namely, Object Algebras [Oliveira and Cook [2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)] and SEDEL [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)].

### 2.1 The Inescapable Expression Problem

In an article talking about modularity and extensibility, it is hard to avoid the infamous EP, which is a minimal problem illustrating a long-standing extensibility dilemma in programming languages [Wadler [1998](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0083)]. At the heart of the EP is how to modularly extend a datatype and the operations over it simultaneously. The concrete problem is about how to create a very simple form of expressions (such as numeric literals and addition) and operations over those expressions (such as evaluation and pretty-printing) in a modular way. In OOP, we usually define an abstract class (or interface) for method signatures and then implement it with various concrete operations:

![](https://dl.acm.org/cms/attachment/3e33ae3e-3110-4b79-adac-e15d0985593b/toplas4303-09-uf04.gif)

In this case, it is quite easy to add more data variants (such as multiplication) by creating new classes. However, adding new operations (such as logging) is difficult, because every class needs amendments for new methods. Such amendments violate the _open-closed principle_ in OOP [Meyer [1988](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0057)]. The situation is exactly the opposite when it comes to functional programming. As shown by eval and printChild in Section [1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-4), new operations are just new functions, while adding data variants becomes difficult, since it requires modification to every function.

Wadler [[1998](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0083)] and Zenger and Odersky [[2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0085)] summarize various requirements that a solution to the EP should satisfy. In short, a solution to the EP should allow modularly adding new forms of expressions and new operations over those expressions, while preserving type-safety and separate compilation. Also desirable is the ability to combine multiple independently developed extensions.

### 2.2 Object Algebras

Object Algebras [Oliveira and Cook [2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)] are a well-known OOP solution to the EP. The abstract syntax of the expressions is described by an Object Algebra interface (a Scala trait):

![](https://dl.acm.org/cms/attachment/f8147970-bce8-4b16-9b7b-0fc346b0717d/toplas4303-09-uf05.gif)

Essentially ExpAlg is an _abstract factory_, where the type parameter Exp captures the expression type and capitalized methods are factory methods returning variants of expressions.

Operations over the expressions are _concrete factories_ that implement the ExpAlg interface. For example, evaluation can be defined as:

![](https://dl.acm.org/cms/attachment/bf298148-aee5-441b-8355-2febb40d1ce5/toplas4303-09-uf06.gif)

Eval implements ExpAlg by instantiating Exp as IEval and defining each factory method accordingly. eval is implemented with a trait instead of a class for allowing mixin composition. To conveniently use eval, an object eval is also defined.

_Adding new operations_. It is easy to add new operations by reimplementing the ExpAlg interface. For example, printing is defined in a way similar to evaluation:

![](https://dl.acm.org/cms/attachment/998da93e-e3af-4d6a-a500-bbfb1d752b39/toplas4303-09-uf07.gif)

where Print instantiates Exp as IPrint and implements each factory method accordingly.

_Adding new variants_. It is also easy to add new variants by extending the Object Algebra interface with new factory methods. For example, multiplications are modularly introduced as follows:

![](https://dl.acm.org/cms/attachment/7f57bd57-214f-41a2-89f0-f3b7903e4a9d/toplas4303-09-uf08.gif)

where MulAlg extends ExpAlg with a new factory method Mul. Existing operations such as eval can be modularly reused in extensions:

![](https://dl.acm.org/cms/attachment/8bc97dc1-dcac-439c-b0ef-f887ea976040/toplas4303-09-uf09.gif)

where EvalMul inherits eval and complements the definition for Mul only.

_Modular terms_. Now, we show how to modularly construct a simple addition expression:

![](https://dl.acm.org/cms/attachment/72103481-4bcb-426f-9fdc-ab419737cc22/toplas4303-09-uf10.gif)

The generic function Exp builds an addition expression via the factory methods exposed by the abstract algebra f. Expressions constructed in this way are modular because concrete expressions can be obtained by calling Exp with concrete algebras such as eval and Print:

![](https://dl.acm.org/cms/attachment/c4f8d6b2-4cc2-404c-88dc-6f419291b63e/toplas4303-09-uf11.gif)

By supplying eval and Print, the expression can be, respectively, evaluated and printed.

### 2.3 Limitations of Object Algebras

Object Algebras, in their basic form, have several limitations. We discuss the limitations one-by-one and show how generalized Object Algebras [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066); Rendel et al. [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)] try to address these limitations.

_Object Algebra combinators_. The first issue is that _there is no proper composition mechanism for Object Algebras._ In the previous section, the expression is indeed constructed _twice_, respectively, for evaluating and printing. A more efficient way is to compose eval and Print into a single algebra so the expression can be constructed _only once_ for both evaluating and printing. A workaround is to use pair-based Object Algebra combinators [Oliveira and Cook [2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)], which require explicit projections and hence are inconvenient for use. Fortunately, explicit projections can be eliminated with the help of intersection types. In Scala, the type A **with** B denotes the intersection of two types A and B, where A **with** B is a subtype of both A and B. The intersection-based Object Algebra combinator for ExpAlg can be defined as:

![](https://dl.acm.org/cms/attachment/95a9d9ca-a28b-42cb-ae07-17a4faa83faa/toplas4303-09-uf12.gif)

ExpMerge captures the two algebras to be composed as fields alg1 and alg2. ExpMerge implements ExpAlg by instantiating the type parameter as A **with** B and defining each factory method by first calling the corresponding factory method respectively defined on alg1 and alg2 then merging the results via the lift method. Concrete composition is done by implementing ExpMerge with the values alg1, alg2, and the definition of lift. For example, the composition of eval and Print is done like this:

![](https://dl.acm.org/cms/attachment/ec670895-3487-40c0-9046-725accd9bfd2/toplas4303-09-uf13.gif)

With the composed algebra evalPrint, the expression can be constructed only once:

![](https://dl.acm.org/cms/attachment/5a76f4b5-1c16-477d-aa83-9916745f9f8a/toplas4303-09-uf14.gif)

Unfortunately, the composition is still done in a cumbersome way. Specialized combinators and a lot of boilerplate code are needed for each Object Algebra interface and for each composition. Workarounds are further proposed such as using generic combinators and reflection [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066)] or a combination of macros and implicits [Rendel et al. [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)].

_Dependent operations_. The second issue is that _it is hard to model dependent operations._ With conventional Object Algebras, the dependent operation has to be defined together with what it depends on. Recall the pretty-printer that depends on the evaluation discussed in Section [1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-4). It is defined as follows:

![](https://dl.acm.org/cms/attachment/e84cc300-d8f7-45a7-90bf-86784df482f9/toplas4303-09-uf15.gif)

The type parameter Exp is instantiated as IEval **with** IPrint for allowing both eval and Print invocations on expressions. Such an implementation is non-modular, because the implementation of eval is repeated inside printChild and printChild and is tightly coupled with a particular implementation of evaluation.

_Generalized Object Algebras_. To account for dependencies modularly, a generalization of Object Algebras has been proposed by Oliveira et al. [[2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066)]. The expression Object Algebra interface is generalized by distinguishing negative (input) and positive (output) occurrences of the expression type. For example, ExpAlg can be generalized in the following way:

![](https://dl.acm.org/cms/attachment/9aa5512e-6f06-488b-8df0-1d7720725017/toplas4303-09-uf16.gif)

where an additional type parameter OExp is introduced for capturing positive (output) occurrences of the expression type. Here, the return type positions of the two factory methods are positive and hence are replaced by OExp. The ordinary Object Algebra interface can be restored by making Exp and OExp consistent, which is convenient for defining algebras without dependencies:

![](https://dl.acm.org/cms/attachment/0456e274-fa82-429e-8510-adcf56145766/toplas4303-09-uf17.gif)

By doing this, evaluation and printing algebras can be defined as before. Furthermore, dependent printing can be modularly defined with generalized Object Algebras:

![](https://dl.acm.org/cms/attachment/902688e2-c4dc-4e08-a885-db9c49fee42d/toplas4303-09-uf18.gif)

The dependency on evaluation is expressed by instantiating the input type as IEval **with** IPrint and the output type as IPrint. The dependency is modular, because printChild does not depend on a particular implementation of evaluation. The dependency can be fulfilled by composing printChild with any other algebra that implements ExpAlg[IEval] such as eval. However, this requires a generalized Object Algebra combinator for performing such composition.

_Summary_. The Expression Problem illustrates some fundamental difficulties of writing modular code in current programming languages. Techniques such as Object Algebras provide a solution for such problems, but they have their own limitations. These limitations partly arise from the lack of programming language support. In particular:

-    **Unconventional programming style:** The programming style required to program with Object Algebras is quite unconventional compared to standard OOP code. For instance, since constructors are avoided, all objects must be constructed relative to an Object Algebra (or factory), similarly to the method Exp. Moreover, the code required for programming with Object Algebras is somewhat verbose.  
    
-    **No built-in composition:** Scala (or other OOP languages) have built-in support for a form of inheritance, which provides a mechanism to compose code. However, such OOP languages do not support the composition of Object Algebras. Composing Object Algebras in such languages is possible, but requires the explicit definition of composition operators, which have to be defined for each Object Algebra interface.  
    
-    **No built-in support for modular dependencies:** Dependencies are quite common in programming. All realistic software will involve multiple forms of dependencies. However, using simple Object Algebras forces dependencies to be written in a non-modular way. Writing modular dependencies is possible with a generalization of Object Algebra interfaces and specialized composition operators. However, such composition operators require a lot of boilerplate code, generalized Object Algebras require a careful manual encoding that distinguishes positive and negative occurrences, and the programming style involved in such code is generally quite heavyweight and unconventional.  
    

In Section [3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-10), we will show how Compositional Programming addresses these issues and leads to a natural, boilerplate-free, modular programming style.

### 2.4 SEDEL and First-class Traits

Intersection types are useful to model modular programs, as we have seen in the previous section. In particular, the Object Algebra combinators use intersection types. However, one useful feature missing in Scala for programming with intersection types is a merge operator. Without this operator, it is sometimes necessary to simulate a merge operator in Scala using meta-programming or reflection [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066); Rendel et al. [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)], which results in convoluted error messages, performance penalties and, more generally, lack of modular type-checking.

Recent developments on disjoint intersection types [Oliveira et al. [2016](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0065)] provide an alternative approach that supports a native merge operator. The λiλi calculus [Oliveira et al. [2016](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0065)] was the first calculus with disjoint intersection types and addressed the incoherence problem of intersection types with a merge operator [Dunfield [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0025)] by introducing the notion of disjointness. The FiFi calculus [Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001)] extends λiλi with a form of parametric polymorphism called _disjoint polymorphism_, where type parameters can be constrained to be disjoint with a specific type. However, λ+iλi+ [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)] extends λiλi with BCD-style distributive subtyping, enabling nested composition. The F+iFi+calculus [Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)] combines disjoint intersection types, disjoint polymorphism, and nested composition, enabling all the foundational ingredients for Compositional Programming. However, F+iFi+ lacks higher-level programming abstractions to make programming convenient. SEDEL [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)] is a surface language built upon FiFi that supports _first-class_ traits. That is, unlike Scala traits, SEDEL traits are expressions that can be passed to a function, assigned to a variable, or returned as values. Furthermore, they can be composed to achieve a form of multiple inheritance. Such composition is enabled by the merge operator of FiFi.

_Trait expressions and self-types_. Like Scala traits, SEDEL traits can be annotated with self-types for expressing the dependency on some methods that are implemented by other traits. For example, we can have two mutually dependent traits that, respectively, implement the methods for testing whether a number is even or odd:

![](https://dl.acm.org/cms/attachment/5e814618-f8f4-485f-8c1e-05cb4359376a/toplas4303-09-uf19.gif)

Even and Odd are type aliases respectively bound to the record types declaring isEven and isOdd. By annotating the self-type as [self: Odd], even can call isOdd via self for implementing isEven in its body. The Even annotation in the end of the trait expression makes sure that isEven has been implemented.

_Trait types_. In the previous example, the type of even is **Trait**[Odd,Even] and the type of odd is **Trait**[Even,Odd]. Trait types in SEDEL distinguish between the _required_ and the _provided_ interface. The required interface describes the methods that the trait needs for providing its functionality, playing a similar role to _abstract methods_ in other OOP languages. Meanwhile, the provided interface describes the functionality that the trait offers. For the case of **Trait**[Odd,Even], Odd is the required interface while Even is the provided interface. When nothing is required, we can just write **Trait**[A] instead of **Trait**[Top,A].

_Trait instantiations_. A trait can be instantiated into an object (a record) using a **new** expression only when its required interface is met. For example, the following attempt to instantiate even fails:

![](https://dl.acm.org/cms/attachment/76dce98d-ef6e-4146-821a-b821fce4b588/toplas4303-09-uf20.gif)

Here, the object's type, Even, must be explicitly specified inside [] of the **new** expression. The above instantiation fails because the required interface of even, Odd, is neither implemented by the trait even nor its parents. Nevertheless, even can be instantiated together with odd, since the two traits implement each other's required interface:

![](https://dl.acm.org/cms/attachment/4871e445-28cf-4de4-a8eb-22f3027b2324/toplas4303-09-uf21.gif)

The two traits are instantiated into a single object of type Even **&** Odd that implements both isEven and isOdd methods.

_First-class traits, disjoint polymorphism, and dynamic inheritance_. An example that differentiates SEDEL's traits from Scala's traits is:

![](https://dl.acm.org/cms/attachment/5ff7986b-1653-4b76-8059-97903699a188/toplas4303-09-uf22.gif)

The function combine takes two traits, x and y, and returns a trait that inherits both x and y. The definition of combine illustrates three key features of SEDEL: _disjoint polymorphism_, _first-class traits,_ and _dynamic inheritance_. First, combine is a polymorphic function and the notation [B ***** A], means that the type parameter B is _disjoint_ with A. This constraint ensures that inheriting x and y simultaneously will have no conflicts. Second, traits are passed as arguments (x and y) and a trait is the return value of combine, showing that traits are _first-class_ values. Third, note that what the **trait** expression inherits (x and y) are parameters of combine, which are statically unknown. Such kind of _dynamic inheritance_ is not possible in conventional statically typed OOP languages (such as Scala or Java), where classes must be statically known, and they cannot be passed as arguments.

_Resolving conflicts_. Trait composition (or inheritance) in SEDEL follows the traditional trait model [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)] where two traits cannot have conflicts for composition to be successful. A benefit of the trait model is that composition is _commutative_ and _associative_, which ensures that the order of composition is irrelevant. In the implementation of SEDEL, trait composition is encoded in terms of the merge operator, which is itself _commutative_ and _associative_ and does not allow for overlapping (or conflicting) values [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)]. In the case that two traits being composed have conflicts, those conflicts must be explicitly resolved in SEDEL. Suppose that we have two traits t1 and t2 that contain conflicting fields:

![](https://dl.acm.org/cms/attachment/1a5dd1df-1f87-4053-a4c8-8b5121596221/toplas4303-09-uf23.gif)

For a trait that would like to inherit from both t1 and t2, the conflicts must be explicitly resolved. Otherwise, a type error will be reported saying that t1 and t2 are not disjoint. One way to resolve the conflicts is:

![](https://dl.acm.org/cms/attachment/44893040-9ba1-4d59-8244-db31ec6c6178/toplas4303-09-uf24.gif)

The conflicts are resolved by using the _exclusion_ operator () to remove f and g, respectively, from t1 and t2. Together with a self-type annotation [self: Top], the excluded methods from parents can still be accessed via the _forwarding_ operator (^^) [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)]. Here, the overridden f sums up the inherited f from t2 via **super** and the excluded f from t1 via the forwarding expression. We can construct an object from t3 to test that f actually returns 3:

![](https://dl.acm.org/cms/attachment/e10924e7-874b-4e6d-ac9c-751a884ea688/toplas4303-09-uf25.gif)

## 3 AN OVERVIEW OF COMPOSITIONAL PROGRAMMING

This section presents an overview of Compositional Programming. We start by introducing the basic mechanisms of Compositional Programming with the **Expression Problem (EP)**. We then move on to harder modularity issues, such as various forms of dependencies, which arise in modular programs that compute various forms of attributes. All the CP programs presented here can run in our implementation of CP and can be found in the companion material of this article.

### 3.1 The Expression Problem with Compositional Programming

In Compositional Programming, there is no EP: It is both easy to add new variants as well as new operations. Indeed, an explicit goal of Compositional Programming is that programmers do not need to face the tension of choosing one dimension for extensibility. Therefore, Compositional Programming offers an alternative way to model data structures that differs from both algebraic datatypes and conventional OOP class hierarchies. Because of such fundamental differences, and the more modular programming style, we can think of Compositional Programming as an alternative programming paradigm. We introduce the mechanisms used in the programming language CP by solving the EP next.

_Compositional interfaces and sorts_. In CP , we use a compositional interface to declare a datatype. Compositional interfaces generalize conventional OOP interfaces: We can define not only the signatures of methods but also those of constructors, which is not possible in conventional OOP languages like Java. The compositional interface for basic arithmetic expressions is:

![](https://dl.acm.org/cms/attachment/80ad0d73-14f9-4971-bfff-fb535ad4adbf/toplas4303-09-uf26.gif)

There are two kinds of expressions for now: literals and addition. The type parameter Exp wrapped in angle brackets is called a _sort_ of the compositional interface, working as the type of both kinds of expressions. Lit and Add are the signatures of the _constructors_ for arithmetic expressions. In CP , constructors always start with a capital letter, while all methods start with a lowercase letter. The return type of a constructor must always be a sort, and there can be arbitrarily many sorts in a compositional interface.

Note that sorts in compositional interfaces are handled differently from normal type parameters, which is why they have a special syntax of angle brackets around them. In essence, compositional interfaces are based on several ideas related to generalized Object Algebras (see Section [2.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-8)), and the special treatment for sorts involves, among other things, distinguishing uses of positive and negative occurrences of the type variables. The semantics of sorts is discussed in detail in Section [5](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-17). Nonetheless, these subtle semantic differences are essentially only relevant for programs with dependencies, such as those presented in Section [3.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-12). For now, it suffices to think of sorts as type parameters.

_Analogy with algebraic datatypes_. Compositional interfaces defining only constructors play a similar role to algebraic datatypes in functional programming. Like algebraic datatypes, they can be used to define data structures by specifying the name of the data structure and the signatures of the constructors. For example, a Haskell counterpart of the compositional interface above is:

![](https://dl.acm.org/cms/attachment/f58967d0-a213-4177-8b08-f9bb16b9f91a/toplas4303-09-uf27.gif)

The sort Exp of the compositional interface corresponds to the name of the datatype, whereas in both cases the constructors essentially specify the same information: the signatures of the constructors.

_Compositional traits_. The unit of code reuse in CP is a generalization of _(first-class) traits_ [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074); Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)]. The generalization stems from the fact that we can define not only method implementations but also _(virtual) trait constructors_ within a trait. In CP , the implementation of the eval operation for the basic form of expressions can be done in a trait:

![](https://dl.acm.org/cms/attachment/11e3aedb-2459-4fad-9263-968a61325cb0/toplas4303-09-uf28.gif)

This trait implements the compositional interface ExpSig with the sort instantiated as eval, corresponding to its actual operation. eval is another example of a compositional interface, which in this case, because it has no sorts, just degenerates into a conventional OOP-style interface with a method eval returning Int. Within the trait evalNum, we implement the eval method for Lit and Add to evaluate the arithmetic expressions.

_Method patterns_. The _method pattern_ (Lit n).eval is a lightweight syntax used to define the eval method within the trait (which implements the interface eval) that the constructor Lit returns. In CP , method patterns are used to make trait definitions concise. Method patterns allow definitions that resemble functional programming definitions by pattern matching, or attributes in attribute grammars. An alternative way of defining constructors would explicitly use first-class traits, which is essentially what method patterns are desugared to:

![](https://dl.acm.org/cms/attachment/f46c2a78-e906-4d8f-bd19-f84a000071cb/toplas4303-09-uf29.gif)

_Nested traits and trait families_. As shown above, method patterns inside a trait are essentially defining _nested traits_. The outer trait, evalNum, is called a _trait family_. The terminology _trait family_ is borrowed from _family polymorphism_ [Ernst [2001](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0027)]. In family polymorphism, a family is a class that contains nested virtual classes. In CP , a trait family is a trait that contains nested virtual traits.

_Virtual constructors_. One significant difference from most existing languages is that CP ’s constructors are _virtual_. In languages like Java, there are virtual methods, whose concrete implementation is unknown at the time a class is defined. In this way, the references to methods are loose and determined only when objects are instantiated. However, languages like Java do not support virtual constructors or _virtual classes_ [Madsen and Moller-Pedersen [1989](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0055)]. A _virtual constructor_ is not bound to a specific implementation of a trait. Instead, it conforms to the signature in the compositional interface. With virtual constructors, it is possible to create a trait that constructs an expression _without sticking to a particular implementation_ of the compositional interface:

![](https://dl.acm.org/cms/attachment/52eaf01e-3017-4801-a26e-489a83f15f77/toplas4303-09-uf30.gif)

In CP , term parameters start with a lowercase letter while type parameters are capitalized, so Exp is a type parameter, serving as the sort of ExpSig. Thus, expAdd contains an abstract expression that is not associated with any concrete method implementations.

_Self-type annotations_. The expAdd trait above is parameterized by Exp and specifies its self-type as ExpSig<Exp>. Such self-type annotations are similar to those in Scala [Odersky et al. [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0061)], which enable a modular way of injecting dependencies on other operations/constructors. The self-type annotation of expAdd expresses that expAdd must be merged with some trait that concretely implements ExpSig for instantiation. By specifying the self-type, the constructors declared inside ExpSig, i.e., Lit and Add, are directly available for building an expression named test. Section [3.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-12) will discuss additional mechanisms in CP to deal with other forms of dependencies.

_Extensibility: adding new operations_. Like in functional programming with algebraic datatypes and pattern matching, adding a new operation is trivial in CP . We can just create an independent trait family that implements the compositional interface:

![](https://dl.acm.org/cms/attachment/1ff4f593-2735-4e3b-a4c1-2c30c4e09b40/toplas4303-09-uf31.gif)

The trait printNum implements ExpSig<Print> and defines Lit and Add using method patterns, in a way similar to evalNum. It modularly supports pretty-printing for expressions.

_Nested trait composition_. At the heart of Compositional Programming is a powerful composition mechanism called _nested composition_ [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)]. Nested composition allows the composition of multiple trait families. This mechanism can be viewed as a form of multiple (trait) inheritance, but with the added ability to recursively compose nested traits automatically. Thus, it provides composition mechanisms that are similar to the ones found in languages with family polymorphism. Like trait composition [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)], nested composition in CP is _associative_ and _commutative_ [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)]. Furthermore, following the trait model and SEDEL, conflicts arising during composition are rejected. In CP , which is statically typed, such conflicts are statically rejected as type errors, following a similar approach to trait composition in SEDEL. Conflict resolution in CP can be done using method overriding or an explicit exclusion operator (like in many models of traits).

In CP , nested composition is performed by the merge operator ,, [Dunfield [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0025)]. For example, to compose the trait families evalNum and printNum with the aforementioned expAdd, we can write:

![](https://dl.acm.org/cms/attachment/7fafcb83-750d-4bda-be51-a3a07eb2e81d/toplas4303-09-uf32.gif)

Note that the merge operator **,,** binds tighter than **new** (but application and type application still bind tighter than **,,**). The type application expAdd **@**(Eval**&**Print) makes the generic trait expAdd concrete by instantiating the type parameter Exp as the argument Eval**&**Print. Since the self-type annotation of expAdd has been instantiated as ExpSig<Eval**&**Print>, to meet this self-type requirement, the trait has to be merged with some trait that simultaneously implements the eval and Print operations for the constructors exposed by ExpSig. The requirement is met by merging expAdd **@**(Eval**&**Print) with evalNum and printNum. Thus, the merged trait is successfully instantiated into an object using a **new** expression.

It is useful to take a moment and understand why the trait composition performed for e and the method calls in the expression above pass type-checking and work as expected. Note that the types of evalNum, printNum, and expAdd @(Eval**&**Print) are, respectively, **Trait**[ExpSig<Eval>], **Trait**[ExpSig<Print>], and **Trait**[ExpSig<Eval**&**Print>,{test:Eval**&**Print}]. Their merge is then of the type **Trait**[ExpSig<Eval**&**Print> ,ExpSig<Eval>**&**ExpSig<Print>**&**{test:Eval**&**Print}]. The merged trait type is an _instantiatable_ trait type (i.e., the provided type is a subtype of the required type) because ExpSig<Eval>**&** ExpSig<Print> is a subtype of ExpSig<Eval**&**Print> in CP . In essence, the subtyping relation employed by CP supports _distributivity_ of intersections over other constructs [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)]. In CP , intersections can distribute over function types, records, and trait types. Through the object e, we can both evaluate and print the addition expression:

![](https://dl.acm.org/cms/attachment/21ab15f4-567c-4d1d-a355-c676b9f82c70/toplas4303-09-uf33.gif)

_Extensibility: adding new variants_. Finally, we show how to add multiplications to the language modularly. Note that this is where Compositional Programming differs from algebraic datatypes and definitions by pattern matching, where such extensions cannot be modularly added. We first define a compositional interface that extends ExpSig with a Mul constructor:

![](https://dl.acm.org/cms/attachment/cb01a2c3-ecd1-4311-91d6-80c48f7716da/toplas4303-09-uf34.gif)

We then implement evaluation and printing by defining two trait families evalMul and printMul that, respectively, inherit evalNum and printNum and complement a definition for Mul:

![](https://dl.acm.org/cms/attachment/d7dd7bbf-4cc9-46ed-815d-6dac095750ac/toplas4303-09-uf35.gif)

Without editing any existing code, we modularly add new data variants. Note that, here, we use a new keyword **inherits**. In CP , inheritance is based on the merge operator but given a more convenient syntax that resembles conventional OOP. Besides nested composition, **inherits** additionally allows fields defined in the parent trait to be overridden but still can be used via **super** calls in the trait body. With **super** calls and inheritance, we can, for instance, construct a slightly more complex expression:

![](https://dl.acm.org/cms/attachment/65e10b59-a706-410a-abf2-672985dd8ea6/toplas4303-09-uf36.gif)

![](https://dl.acm.org/cms/attachment/f9e49ed1-3cc1-43ea-b2d7-cfd942b6eaed/toplas4303-09-uf37.gif)

The trait expMul inherits expAdd, refines the self-type to MulSig, and overrides the test field. The overridden expression reuses the inherited expression via **super.**test. Finally, the object e′e′ supports all of the three constructors together with the evaluation and printing operations.

_Summary_. For the basic EP, there are already many solutions in the literature, including some in mainstream programming languages [Garrigue [2000](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0032); Torgersen [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0080); Oliveira et al. [2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0064); Zenger and Odersky [2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0085); Swierstra [2008](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0077); Oliveira and Cook [2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)]. One interesting aspect of the solution presented here is that it is quite elegant and natural, while solutions in mainstream languages tend to have significant amounts of boilerplate code or are written in highly parametrized code that is not programmer-friendly. Additionally, the more interesting aspect of Compositional Programming is its wide support for various forms of modular code with dependencies, which is shown next.

### 3.2 Dependencies and S-attributed Grammars

In the original EP by Wadler [[1998](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0083)], there are very few _dependencies_. In particular, the operations (eval and print) depend only on themselves, but they do not depend on other operations. Matters become significantly more complicated in the presence of more advanced forms of _dependencies_, and very few existing solutions to the EP have effective mechanisms to deal with dependencies in a modular way. Two primary mechanisms are used to deal with dependencies in CP :

-    **Compositional interface type refinement:** When a trait implements some compositional interface, it can refine the _sort types in input positions_. This allows the child nodes in a data structure to assume some functionality that is not implemented in the enclosing trait, but will eventually be part of the final composition later.  
    
-    **Self-type annotations:** Like in Scala, the types of self-references can be specified/refined. This enables the self-references to assume functionality that will be implemented by a different trait that will eventually be composed with the current trait. In the context of trait families, there are two kinds of _self-references_: _object self-references_ and _family self-references_ [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066)]. The trait expAdd in Section [3.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-11) already illustrates family self-references, so in what follows, we illustrate only object self-references.  
    

We use examples inspired from _S-attributed grammars_ [Knuth [1968](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0048)] and the work from Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)] to illustrate how Compositional Programming addresses programs with different kinds of dependencies in a modular way. S-attributed grammars deal with synthesized attributes, which are computed from the children. Compositional Programming also allows dependencies on self. The simplest form of dependencies is dependencies on the same attribute of children, which occurred several times in Section [3.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-11). Therefore, we focus on two kinds of non-trivial dependencies on other synthesized attributes, which, hereafter, we call _child dependencies_ and _self dependencies_.

_Child dependencies_. Child dependencies occur when an attribute depends on other synthesized attributes of the children. Here is the printChild example from Section [1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-4) in CP :

![](https://dl.acm.org/cms/attachment/81bf9f9a-effb-4ddc-9bd0-f5440a597037/toplas4303-09-uf38.gif)

Here, (Add e1 e2).print depends on e2.eval. However, there is no implementation of eval in the trait printChild. To make this child dependency feasible, we use compositional interface type refinement: We change the concrete interface being implemented to ExpSig<Eval **%** Print> (instead of just using ExpSig<Print>). This syntax means that the input type for the expressions in the Add constructor (and other constructors, if any, referring to the sort Exp) is Eval**&**Print, while the output type is still Print. By doing this, we enable the child nodes of Exp to depend on an attribute whose implementation is modularly defined somewhere else. Two examples of trait instantiations are:

![](https://dl.acm.org/cms/attachment/af5e1d33-3b63-41f3-a003-9730e83ced36/toplas4303-09-uf39.gif)

This first instantiation attempt fails type-checking, because it depends on eval, which is missing. The second one works, since we merge printChild with the trait evalNum (which implements Eval). Importantly, instead of evalNum, we could have used any implementation with the same type.

_Contrast with inheritance_. It is useful to compare the previous example with the more common OOP approach based on inheritance:

![](https://dl.acm.org/cms/attachment/0cc558ff-049e-4f04-86d2-dc19d70a3ffc/toplas4303-09-uf40.gif)

The first thing to notice is that the code in the trait body of printInh is exactly the same as that in printChild. Conforming to the compositional interface instantiated with Eval**&**Print, printInh essentially implements both eval and print instead of only print, because the evalNum trait family is inherited. Although both approaches work, printInh is tightly coupled with the particular implementation of evaluation coming from evalNum. In contrast, printChild declares a weaker dependency on the abstract interface of Eval. We can delay the combination with a concrete implementation of Eval until the instantiation phase. In short, printChild allows for _weak_ child dependencies, which are not coupled with a particular implementation, while preserving strong static type safety. More generally, most uses of inheritance in CP can be converted into code that has weaker dependencies.

_Self dependencies_. A second interesting case is dependencies on other synthesized attributes of the self-reference. In the following example, the attribute print depends on self.eval:

![](https://dl.acm.org/cms/attachment/9e034d14-bfe3-4409-9e8f-18a56efb04fa/toplas4303-09-uf41.gif)

To deal with this dependency without sticking to a particular implementation of eval, we add an (object) self-type annotation [self:Eval] to use self.eval in Add. Note that we also need to change the sort instantiation to ExpSig<Eval **%** Print> like the child dependencies to change the self-type of the returning trait correspondingly. The static type-checker will check whether the trait is later merged with another trait that implements ExpSig<Eval>. With no compromises on type safety, CP enables modular weak self dependencies on other attributes.

_Mutual dependencies_. Finally, a more general form of dependencies is _mutual dependencies_, which happen when two attributes are inter-defined, i.e., they depend on each other. Mutual dependencies can involve both child and self dependencies, as illustrated in the following example:

![](https://dl.acm.org/cms/attachment/8a5af857-2318-4089-aa6a-d51fcacae485/toplas4303-09-uf42.gif)

The two trait families printMutual and printAux cooperate to omit the outermost parentheses in pretty-printing. We can see that (Add e1 e2).print depends on the printAux while printAux depends on print, thus print and printAux are mutually dependent. CP handles such mutual dependencies modularly. We can combine the traits and use them as before:

![](https://dl.acm.org/cms/attachment/1b23bce2-32ca-4578-bb26-f795cd25eae2/toplas4303-09-uf43.gif)

_Summary_. Many attribute grammar systems allow the modular definition of attributes, but this is usually done at the cost of modular type-safety. The compositional mechanisms in CP retain the ability of modularly defining attributes from S-attributed grammars or even attributes from self-references, but in a statically type-safe setting. Moreover, the implementations of the attributes are changeable in the final composition. For example, the aforementioned traits printNum, printChild, printSelf, and printMutual all implement the print method. A programmer can freely pick his favorite implementation to combine with other attributes, such as eval.

## 4 PARAMETRIC POLYMORPHISM AND L-ATTRIBUTED GRAMMARS

In Section [3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-10), we introduced the basic mechanisms for Compositional Programming and illustrated how CP deals with dependencies. One feature that practically all modern languages support is some form of _parametric polymorphism_ (or _generics_ in OOP languages). A reasonable question to ask is how Compositional Programming interacts with parametric polymorphism. Furthermore, one may wonder if the combination of parametric polymorphism and Compositional Programming enables novel techniques that are useful for programming.

In this section, we explore this question. CP has full support for a form of parametric polymorphism called _disjoint polymorphism_ [Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001)]. We introduce a novel technique called _polymorphic contexts_, which addresses the problem of different modular components requiring different kinds of contextual information. The technique provides encapsulation of contexts: Modular components have access to the contextual information they require but do not have access to contextual information used by other components. This technique is useful to model _inherited attributes_, where it is possible to access attributes from parents or siblings. To achieve this, a context should be attached to pass attributes from top to bottom. There is a close relationship between contextual evaluation and _L-attributed grammars_ [Knuth [1968](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0048); Rendel et al. [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)].

### 4.1 Contexts and Modular Components

There are plenty of scenarios where we need to add contexts to our code. One of the most common examples is variable binding in an interpreter. Recall that, in Section [3.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-11), we defined the type of eval as Int. But this interface is not suitable for an expression language with variable binding. A naive fix is to modify the existing code and add a context to all the trait families:

![](https://dl.acm.org/cms/attachment/e1d82dea-ff99-45eb-86fa-97f6e0800810/toplas4303-09-uf44.gif)

Note that we add an EnvN parameter to eval. Since CP ’s support for type inference is still limited, while n, e1, and e2 do not require type annotations, we have to annotate env with EnvN here. EnvN is a map from String to Int, serving as the variable environment, while insert and lookup are auxiliary functions on maps. For the Let expression in evalVar, we evaluate e1 and insert the evaluation result into env before evaluating e2 (the body of the Let expression). For the Var expression, we look up the variable name to get its value. Although evalNum does not need the context for evaluating arithmetic expressions, we still have to pass the context to the recursive calls to make env available everywhere.

So far, it seems that everything works well. But what if we add a new context? For example, many interpreters need to pre-define some primitive functions, which are called _intrinsics_. Therefore, we should add an intrinsic environment to store these intrinsic functions. Just like Common Lisp, functions and values do not share the same namespace in this expression language, so these two environments are independent of each other. We need a second parameter for eval, which requires modifying all existing code again:

![](https://dl.acm.org/cms/attachment/7114e2cd-e9b8-4320-b4bf-f4682c4bb22d/toplas4303-09-uf45.gif)

Such an approach to adding contextual information has two main problems:

-    **It is highly non-modular:** Every time a new context is needed, all the existing code has to be modified. What is worse, we cannot easily modify the type of context if the previous definitions are from a library. In other words, the library author has to anticipate what kind of contexts will be needed in the future, which is impossible.  
    
-    **It does not encapsulate contexts:** Interfaces of contexts are fully exposed, even if they are not directly used. For example, Let and Var do not touch envF, while LetF and AppF do not touch envN. Such unnecessary exposure may lead to unexpected modifications to the contexts, which ought to be hidden to avoid exploitation by malicious code.  
    

### 4.2 Polymorphic Contexts

To address the two problems identified in the previous section, we propose a technique that relies on _disjoint polymorphism_, _intersection types,_ and _nested composition_. This technique enables modular, encapsulated contexts for modular components. Since we cannot anticipate what a context will evolve to in the future, our idea is to make contexts subject to change using parametric polymorphism.

_Evaluation with a polymorphic context_. Instead of creating an interface for evaluation with specific contexts, we can parametrize the type of the context:

![](https://dl.acm.org/cms/attachment/303494e0-a459-432e-b03b-498803160459/toplas4303-09-uf46.gif)

Here, Context is a type parameter. In essence, Eval becomes a variant of the _Reader_ Monad [Wadler [1992](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0082)], which is commonly used in functional programming. Similarly to Monads, some initial planning is necessary to adapt the existing code to use polymorphic contexts:

![](https://dl.acm.org/cms/attachment/234c0b8b-8670-4a79-b045-87dc7d657608/toplas4303-09-uf47.gif)

The trait evalNum has a type parameter Context, which is used as the type of the context in evaluation. Importantly, when implementing evalNum, the only thing one can do with ctx is to pass it to the children's call on eval, since ctx is the only value of type Context in scope. In other words, parametric polymorphism enforces the encapsulation of the context and ensures that the correct context is passed to the evaluation of the children. To illustrate how CP enforces encapsulation of the context, suppose that we try to extract information from the polymorphic context, for example, by trying to look up a variable in the context:

![](https://dl.acm.org/cms/attachment/9829ba15-51da-4d58-be12-8bd586d3239d/toplas4303-09-uf48.gif)

This code fails to type check, because the type of ctx (i.e., Context) is not a subtype of EnvN, and there is no dynamic casting in CP that can change its type to EnvN. In short, if the polymorphic context is completely polymorphic (i.e., its type is just a type variable), then there is not much that can be done with the context except passing it down to recursive calls.

To instantiate evalNum, which requires no context on its own, we can specify Context as Top and pass () (the canonical top value) to eval:

![](https://dl.acm.org/cms/attachment/c4c0da9d-7150-4c26-853e-91b50ff7fcc7/toplas4303-09-uf49.gif)

While the code requires an initial modification to be adapted to polymorphic contexts, no additional changes are necessary when future contexts are added to the program. Moreover, a nice quality of the code is that it is written in a direct style. Using more sophisticated solutions, such as Monads, would give additional expressive power, but often at the cost of writing code in a different style (for instance, with the monadic do notation).

_Adding components with contexts_. Let us revisit the variable binding example. To add constructs that deal with variables and binders, a context with an envN field is needed:

![](https://dl.acm.org/cms/attachment/4997f045-f948-4bdb-886a-30181e4f117d/toplas4303-09-uf50.gif)

Now, we have to write the code for a trait that deals with the evaluation of variables and binders. But what should be the type of context in this case? Using a fully polymorphic context, as we did for literal and addition expressions, will not work, because we need to extract and update information from the environment. Furthermore, using the type CtxN directly as the type of the environment is too specific, because it forces the contexts to contain exactly CtxN and nothing else. This would prevent modular context evolution. The answer is to use a context with the intersection type CtxN**&**Context:

![](https://dl.acm.org/cms/attachment/aa792c24-e51f-49a0-a86b-b39f22129536/toplas4303-09-uf51.gif)

Context is a type variable _disjoint_ with CtxN (expressed as the annotation Context ***** CtxN). This is an example of disjoint polymorphism [Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001)], which is supported in CP . The disjointness constraint ensures that when the type variable is instantiated to a concrete type, that type cannot share a common supertype with CtxN. By using the type CtxN**&**Context as the type of context, we ensure that we can access the environment while being oblivious of other information in the context. Thus, the context remains partly polymorphic and adaptable to future extensions while retaining encapsulation and ensuring that the other information in Context cannot be altered.

_A record update problem_. The variable environment should be updated during the evaluation of the Let expression, whereas any other information in the context should be retained as well. Note that the type of {envN **=** insert ...} is CtxN, which does not match CtxN**&**Context. So, we have to merge it with (ctx:Context) to get back the Context part. This upcasting is possible because Context is a supertype of CtxN**&**Context. The disjointness constraint also ensures that the merge passes type-checking. The code illustrates that, in CP , we can do a _polymorphic record update_ [Cardelli and Mitchell [1991](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0017)] in the presence of subtyping, which is a notorious problem in many calculi with polymorphic records. For instance, it is well-known that F<:F<: with records (and many other calculi with bounded quantification) cannot solve the polymorphic record update problem. There are only a few calculi with polymorphic records and subtyping that can deal with this problem [Cardelli and Mitchell [1991](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0017); Cardelli [1994](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0016); Poll [1997](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0069)], and CP and its core foundation F+iFi+ are among them. Note that the problem is simpler _without subtyping_, and various calculi with row polymorphism can address a polymorphic record update (for instance, Leijen [[2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0052)]; Chlipala [[2010](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0020)], among others).

_A second component with a context_. To support intrinsic functions, we need a second environment EnvF and a corresponding trait family:

![](https://dl.acm.org/cms/attachment/951bd2f0-001f-4176-8779-8f7e052af6bb/toplas4303-09-uf52.gif)

Similarly, a polymorphic record update is needed for the LetF expression. Although these polymorphic trait families are defined separately, we can still merge them together using nested composition:

![](https://dl.acm.org/cms/attachment/8009950c-13c5-4da7-8052-262879fc3397/toplas4303-09-uf53.gif)

During the composition of e, the context types used by other trait families are passed as type arguments to make the final context consistent.

### 4.3 L-Attributed Grammars

The previous examples only access attributes from parents. It is also possible for inherited attributes to depend on siblings. There is a superset of the aforementioned S-attributed grammars called _L-attributed grammars_ [Knuth [1968](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0048)], where inherited attributes depend on parents and left siblings. In such grammars, attributes can be easily evaluated by a left-to-right depth-first traversal.

To illustrate L-attributed grammars, we take pretty-printing as an example. We use a position number to represent each terminal node in this pretty-printing function. The position number is determined by the pre-order traversal of the syntax tree. Before computing it, we need an auxiliary attribute called cnt that calculates the total number of nodes in the current subtree:

![](https://dl.acm.org/cms/attachment/87b7daa8-33bb-43d2-8b13-0438c98281a6/toplas4303-09-uf54.gif)

With the help of cnt, we can compute that e1.pos=e0.pos+1e1.pos=e0.pos+1 and e2.pos=e0.pos+e1.cnt+1e2.pos=e0.pos+e1.cnt+1, where e0e0 is the parent node of e1e1 and e2e2. The latter equation is a typical example of L-attributed grammars, because the attribute depends on both its parent (e0.pose0.pos) and left sibling (e1.cnte1.cnt). In our encoding, such computation is done on the parent side and the result is passed down using polymorphic contexts:

![](https://dl.acm.org/cms/attachment/bea0687e-4d2c-4b06-9cf8-e50eff919813/toplas4303-09-uf55.gif)

To compute the inherited attribute pos, we introduce two auxiliary functions in InhPos. They are implemented in accordance with the equations stated before. The self-type annotation [self:InhPos] is added for (Add e1 e2).print to access these two functions. Just like previous environments in interpreters, pos serves as a part of the context parameter of print. Since there is a child dependency on cnt, the sort is instantiated as <Cnt **%** PrintPos Ctx>. The position number is calculated and passed down via print calls in Add, while inh.pos is finally used in Lit.

With all of the traits ready, we can compose an expression and call the print function like before:

![](https://dl.acm.org/cms/attachment/c10ed341-b2cc-43a4-a49b-353e0d2fb064/toplas4303-09-uf56.gif)

Since no other contexts need to be mixed together, we just pass Top as the type argument of printPos. In the last invocation on print, we set the initial context to {pos = 0}. This means that the root node of the whole syntax tree is marked as 0, then the parent of the left subtree is 1 and the leaves are 2 and 3. Similarly, the parent of the right subtree is 4 and the leaves are 5 and 6. We finally print the position numbers of leaf nodes, as the code shows above.

In fact, our encoding does not only allow L-attributed grammars, any inherited and synthesized attributes can be implemented by contextual evaluation. Nevertheless, L-attributed grammars correspond to a one-pass traversal and ensure termination, so they are the most common usage of attribute grammars and a good example for polymorphic contexts.

_Summary_. With polymorphic contexts, L-attribute-grammar-like programs are expressed in a statically safe way. Such programs are not uncommon in real-world applications. Interpreters or other operations over ASTs are typical applications where non-trivial forms of attributes can occur. We have shown how variable and intrinsic environments are supported in the previous examples. Besides the two, more contexts may emerge when a language evolves, such as dynamic scoping, mutable parameters, and error handling.

There are three advantages of our approach to polymorphic contexts: (1) it enforces the recursive calls to take the full context as an argument; (2) the polymorphic portion of the context cannot be fiddled with, i.e., the only thing one can do is to pass it unchanged; (3) nonetheless, polymorphic contexts can still be refined for particular uses and expose just the right amount of information while hiding the remaining information. Polymorphic contexts, as well as the ability to express complex forms of attributes, are a valuable supplement to the modularity of Compositional Programming.

## 5 FORMALIZATION

This section presents the syntax and semantics of CP . The syntax of CP extends SEDEL [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)] with constructs for Compositional Programming (i.e., compositional interfaces, compositional traits, method patterns, and nested trait composition). The semantics of CP is given by elaborating to a call-by-name formulation of F+iFi+ [Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)], a typed calculus combining disjoint intersection types and polymorphism with BCD-style subtyping [Barendregt et al. [1983](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0004)]. Nested trait composition is built on top of F+iFi+’s nested composition [Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006)]. We prove that the elaboration to F+iFi+ is type-safe and coherent.

### 5.1 Syntax

Figure [1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig1) gives the syntax of CP . A program is a sequence of declarations followed by an expression.

![Figure 1](https://dl.acm.org/cms/attachment/fca08aac-6111-4a0f-81c2-6bc4ebc1c34c/toplas4303-09-f01.jpg)

Fig. 1. Syntax of CP .

_Declarations_. There are two kinds of declarations: type declarations and term declarations. A type declaration typeX⟨¯¯¯¯α⟩extendsA=BtypeX⟨α¯⟩extendsA=B is used for two purposes: introducing type aliases and declaring compositional interfaces. To simplify the formalization, we do not formalize type declarations with conventional type parameters (i.e., we only consider type declarations with sorts). However, adding type parameters can be done in standard ways and we have such declarations in our implementation. The type (or type constructor) XX is parametrized with a sequence of sorts ⟨¯¯¯¯α⟩⟨α¯⟩ and can optionally extend another type. There are two forms of term declarations: x=Ex=E is for simple variable bindings; (L¯¯¯¯¯¯¯¯¯¯¯¯x:A[self:B]).l=E(Lx:A¯[self:B]).l=E is a _method pattern_ serving as syntactic sugar for defining single-field traits conveniently.

_Types_. Metavariables AA and BB range over types. Types include integers IntInt, type variables αα, the top type ⊤⊤, the bottom type ⊥⊥, arrows A→BA→B, disjoint quantification ∀(α∗A).B∀(α∗A).B, intersections A&BA&B, single-field record types {l:A}{l:A} (multi-field record types are syntax sugar for intersections of multiple single-field record types) , trait types Trait[A,B],Trait[A,B],, and type aliases with sorts instantiated X⟨¯¯¯¯S⟩X⟨S¯⟩, where each sort can be either instantiated using a type or a pair of types.

_Expressions_. Metavariable EE ranges over expressions. Expressions include integer literals ii, term variables xx, the top value ⊤⊤, lambda abstractions λx.Eλx.E, term applications E1E2E1E2, type abstractions Λ(α∗A).EΛ(α∗A).E, type applications E@AE@A, merges E1,,E2E1,,E2, multi-field records {¯¯¯¯¯¯M}{M¯}, record projections E.lE.l, annotated expressions E:AE:A, and (recursive) let bindings letx:A=E1inE2letx:A=E1inE2. There are also a few trait-related constructs. trait[self:A]implementsBinheritsE1=>E2trait[self:A]implementsBinheritsE1=>E2 specifies an explicit selfself reference of type AA, a type BB to implement, an inherited trait expression E1E1 and a body expression E2E2. The newEnewE construct instantiates a trait expression. E1^E2E1^E2 is the forwarding expression inherited from SEDEL [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)]. Inspired by ML-like modules [MacQueen [1984](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0054)], openE1inE2openE1inE2 is a new construct for directly accessing fields from a record without explicit projections.

### 5.2 An Informal Introduction to the Elaboration

The semantics of CP is defined by elaborating to F+iFi+ extended with recursive let bindings. Elaborating a CP program into a F+iFi+ expression takes several steps, including desugaring, sort transformation, type expansion, and so on. The elaboration builds on two ideas from the literature: the denotational model of inheritance by Cook and Palsberg [[1989](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0022)] and generalized Object Algebras [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066)]. To better understand the elaboration, let us revisit some of the examples presented in Section [3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-10) and show their concrete elaborations into CP code using more atomic features, such as record types and records, and recursive let bindings.

_Elaborating compositional interfaces and sorts._. First, the compositional interface ExpSig:

![](https://dl.acm.org/cms/attachment/7697576a-656d-47c5-b0a7-2938efd5eda6/toplas4303-09-uf57.gif)

is translated to an equivalent type using only type parameters:

![](https://dl.acm.org/cms/attachment/826d0f60-f5d5-4914-b39f-53021ec8147f/toplas4303-09-uf58.gif)

The sort Exp is represented by two type parameters, Exp and OExp, for, respectively, capturing the negative and positive occurrences of Exp. Negative occurrences of Exp (at input positions) are kept unchanged, while positive occurrences of Exp (at output positions) are changed to OExp. For Add, the two parameters of type Exp are negative. Therefore, only the return type of Add is transformed. Since Add is a constructor, positive occurrences of Exp are further translated to a trait type **Trait**[Exp,OExp]. As a type synonym, ExpSig and its right-hand side are put into the type context that tracks type declarations, among other things. In essence, this transformation is inspired by generalized Object Algebra interfaces [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066)] (see also Section [2.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-8)), and the distinction of positive and negative occurrences is helpful for expressing dependencies. If we just wanted to model modular programs _without dependencies_, then distinguishing between positive and negative occurrences would not be necessary.

Similarly, the extended compositional interface MulSig<Exp>:

![](https://dl.acm.org/cms/attachment/44658401-daa7-4b88-9150-ac654aec94f8/toplas4303-09-uf59.gif)

is translated to a type equivalent to:

![](https://dl.acm.org/cms/attachment/083b2bbc-c1a4-4b2c-9f5a-4d82c971fe97/toplas4303-09-uf60.gif)

where the type appearing in the **extends** clause is expanded by looking up the type context and intersecting that type with the type on the original right-hand side.

_Elaborating traits._. The trait family evalNum implements ExpSig:

![](https://dl.acm.org/cms/attachment/d946a9ef-7098-45fb-9434-2bb4cd88cc4d/toplas4303-09-uf61.gif)

which is desugared to:

![](https://dl.acm.org/cms/attachment/6a0c8092-0eff-49b3-9dda-2c6bd42adaa0/toplas4303-09-uf62.gif)

The sort instantiation <Eval> indicates that there are no dependencies. Therefore, both type parameters (Exp and OExp) in the elaborated code are instantiated as Eval. Since no self-type is specified, the self-reference has type Top by default. We can ignore the **open** expression for the moment, since it has no effect in this case, as the self type is Top. Method patterns are desugared to functions returning traits and multi-field records are desugared to a merge of multiple single-field records. After type-checking, evalNum is further elaborated into a F+iFi+ expression:

![](https://dl.acm.org/cms/attachment/50cf2c34-b3f1-4ec4-88ee-d746ad2235c9/toplas4303-09-uf63.gif)

The term declaration is elaborated to a let expression and the trait expressions are elaborated to functions. Since no self-types are specified, the self parameters are all of type Top.

_Elaborating child dependencies._. printChild also implements ExpSig but contains child dependencies:

![](https://dl.acm.org/cms/attachment/78a00652-c26f-4ae8-9358-1b16761b1540/toplas4303-09-uf64.gif)

This trait is desugared to:

![](https://dl.acm.org/cms/attachment/d08fdb02-77b8-49d7-930c-62e45d2e8797/toplas4303-09-uf65.gif)

Since printChild instantiates the sort as <Eval **%** Print>, Exp and OExp are, respectively, instantiated as Eval**&**Print (i.e., the intersection of the two types in <Eval **%** Print>) and Print (i.e., the second type in <Eval **%** Print>). Through some type inference, the arguments with expression types in the constructors (such as e1 and e2) are of type Eval**&**Print. This enables eval to be called on e1 and e2 without an implementation of that operation in the current trait. In short, printChild is elaborated to an expression similar to evalNum except that expressions arguments are of different type (Eval**&**Print) from the output. Importantly, for the modular definition of printChild to work, it is crucial to use different types in the instantiation of Exp for input (negative) positions and output (positive) positions.

_Elaborating self-type annotations_. However, traits with explicit self-type annotations are desugared differently. For instance, expAdd:

![](https://dl.acm.org/cms/attachment/6e3e60e6-050c-4e87-a197-24513b513ff9/toplas4303-09-uf66.gif)

is desugared to:

![](https://dl.acm.org/cms/attachment/a91c7848-e2a6-486a-ae82-0f605311440c/toplas4303-09-uf67.gif)

The original trait body is wrapped into an **open** expression so the constructors/methods exposed by the self-type are directly in scope. Further elaboration into F+iFi+ results in:

![](https://dl.acm.org/cms/attachment/4a90751a-8836-4f5f-92f1-5770898e21a2/toplas4303-09-uf68.gif)

The type alias used in specifying the self-type is expanded. The type translation rewrites trait types as function types. The **open** self expression is elaborated into a series of let bindings, one for each record label. The **new** expressions are elaborated to a lazy fixed point of self, following Cook and Palsberg's denotational model of inheritance [Cook and Palsberg [1989](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0022)].

_Elaborating inheritance and overriding_. We now show the elaboration of a trait that additionally uses **inherits** and **override**. An instance is expMul,

![](https://dl.acm.org/cms/attachment/cd0ee106-60ab-4d4d-966d-8cc8398620d6/toplas4303-09-uf69.gif)

which is elaborated as:

![](https://dl.acm.org/cms/attachment/865a3a20-3160-4a34-bc22-a9e87e6b7f59/toplas4303-09-uf70.gif)

The inherited trait expression (expAdd) is elaborated as a function and applied to the self-reference and the result is bound to **super**. Then **super** is merged with the body of the trait. Since test is overridden, it should be excluded from super, otherwise a conflict will occur when merging super with the body. The exclusion of test from **super** is done by injecting a type annotation Top.

_Elaborating nested composition of traits_. Finally, we show how the merge of traits is elaborated using (**new** evalNum **,,** expAdd **@**Eval).test.eval as an example:

![](https://dl.acm.org/cms/attachment/378e883b-384a-4a50-ab62-36cb47b8abe8/toplas4303-09-uf71.gif)

where we have renamed the outer and inner self as oself and iself for distinction. evalNum **,,** expAdd **@**Eval is elaborated to a function that returns the merge of respectively applying evalNum and expAdd **@**Eval, which are already elaborated as functions, to iself. Since the merged trait is of type **Trait**[ExpSig<Eval> ,ExpSig<Eval>**&**{test:Eval}], iself has the elaborated type of ExpSig<Eval> while oself has the elaborated type of ExpSig<Eval>**&**{test:Eval}.

### 5.3 Static Semantics

_Overview_. Figure [2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig2) gives an overview of all the relations involved in the elaboration, where →→ denotes the transformation flow and ![](https://dl.acm.org/cms/attachment/17a5c698-5c13-46ca-95c6-18cbdb1e299b/toplas4303-09-inline72.gif) denotes the dependencies between the relations. First, patterns and multi-field records are desugared by ![](https://dl.acm.org/cms/attachment/c929a46d-a8f8-430c-bf77-929c2011a927/toplas4303-09-inline73.gif). Then the program PP is elaborated into a F+iFi+expression ee by ![](https://dl.acm.org/cms/attachment/05fef847-a59e-4908-8578-ed2b0b4005a0/toplas4303-09-inline77.gif). During the elaboration process, some transformations and checks are performed: type synonyms and compositional interfaces X⟨¯¯¯¯S⟩X⟨S¯⟩ are expanded by Δ,Σ⊢A⇒BΔ,Σ⊢A⇒B; the right-hand side of a type declaration is transformed by Σ⊢cpA⇒BΣ⊢pcA⇒B for distinguishing positive and negative occurrences of sorts; the subtyping relation between two types is checked by A<:BA<:B; the disjointness of two types is checked by Δ⊢A∗BΔ⊢A∗B. Both subtyping- and disjointness-checking rely on top-like types (⌉A⌈⌉A⌈). The latter further relies on disjointness axioms (A∗axBA∗axB).

![Figure 2](https://dl.acm.org/cms/attachment/c50c5570-0462-4e44-b8ef-c244f4e78eed/toplas4303-09-f02.jpg)

Fig. 2. The relation of relations.

_Desugaring_. The core desugaring rules are:

![](https://dl.acm.org/cms/attachment/cbf78143-a678-4c97-8c15-09a083ea8414/toplas4303-09-ueqn1.gif)

The first rule desugars a multi-field record into an intersection of singleton records. The second rule desugars a method pattern into an ordinary variable binding to a function that returns a single-field trait. The last rule implicitly opens self for the trait body so members declared by the self-type can be directly accessed without prefixing self.

_Type-checking_. Figure [5](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig3) shows the selected typing rules. The gray parts could be ignored for the moment. They will be discussed later in Section [5.5](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-22). The type system of CP is bidirectional: under the contexts ΔΔ and ΓΓ, the inference mode (⇒⇒) synthesizes a type AA while the checking mode (⇐⇐) checks against AA. A lot of the rules are presented previously in the literature [Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)], thus, we discuss only the novel ones, which mostly relate to traits and declarations.

![Figure 3](https://dl.acm.org/cms/attachment/d4e5c1d7-5df1-4c4f-b3b1-5591e2264fa6/toplas4303-09-f03.jpg)

Fig. 3. Selected typing rules.

The rule T-tyDecl adds a type alias to the type context ΔΔ. The right-hand side, type AA, is expanded and transformed before it is added to ΔΔ for type-checking the remaining program. Each sort αα has a companion fresh variable ββ (corresponding to OExp in the examples) for distinguishing negative and positive occurrences of sorts. The rule T-tmDecl deals with term declarations. The bound expression, EE, is inferred with a type AA. Then, xx of type AA is added to the term context ΓΓ for type-checking the remaining program.

The rule T-trait is the most complicated one, since multiple types and expressions are involved and several validity checks are performed. It first expands the self type AA and the type to implement BB as A1A1 and B1B1. Then selfself is added to ΓΓ in type-checking the inherited expression E1E1 and the body E2E2. The inherited expression E1E1 is valid only when it is of a trait type Trait[A2,B2],Trait[A2,B2], and the requirement A2A2 is met by A1A1. After validity-checking, supersuper can be added to ΓΓ in type-checking the body E2E2. The type of body should be disjoint to the type of the inherited expression E1E1 (C∗B2C∗B2). Meanwhile, the body and the inherited expression should together implement what B1B1 specifies (C&B2<:B1C&B2<:B1). The rule T-new instantiates a trait. The expression EE should be of type Trait[A,B],Trait[A,B], and the requirement AA should be met by BB. The rule T-open collects the record types from the inferred type of E1E1 and adds every label type pair to the term context in inferring E2E2. Besides a rule for ordinary merges, T-mergeTrait is a novel rule specially for the merge of two traits. T-mergeTrait says that if the two expressions are of type Trait[A1,B1],Trait[A1,B1], and Trait[A2,B2],Trait[A2,B2], and B1B1 and B2B2 are disjoint, then the merged expression is of type Trait[A1&A2,B1&B2],Trait[A1&A2,B1&B2],. Inferring the merged traits as a trait type rather than an intersection type brings several advantages, which will be discussed in Section [7](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-28).

_Sort transformation_. Figure [4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig4) shows the sort transformation, which is a phase that replaces positive occurrences of a sort appearing in a type declaration with some other type. Sort transformation is illustrated in the previous section by elaborating the right-hand side of ExpSig and MulSig. From the names, we cannot distinguish sorts from type parameters and therefore a sort context ΣΣ is needed. How a sort is transformed is further determined by two conditions: pp and cc. The condition pp tracks the positions where sorts appear: ++ indicates a positive position and −− indicates a negative position. Positive and negative positions are treated differently: Negative positions are kept unchanged while positive positions are transformed. Specifically, for a function type A→BA→B, pp is flipped in processing AA but unchanged in processing BB (rule TR-arr). The same applies for Trait[A,B],Trait[A,B], (rule TR-trait). The Boolean value cc bookmarks whether the type appears inside a constructor. By default false, cc is set to be true when the label of a record is capitalized (rule TR-rcd). According to pp and cc, rules TR-positive and TR-ctrPositive transform the sort αα to ββ and Trait[α,β],Trait[α,β],, respectively.

![Figure 4](https://dl.acm.org/cms/attachment/bf243618-7330-4f67-be0b-6bd05932d093/toplas4303-09-f04.jpg)

Fig. 4. Selected sort transformation and type expansion rules.

_Type expansion_. Type expansion plays two roles: eliminating type aliases and ensuring the well-formedness of the types. It is used, for example, when elaborating the extends clause on MulSig and implements clauses on evalNum and printChild. Figure [4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig4) also shows the relevant type expansion rules. The rule E-tvar checks whether the type variable is in the type context. The rule E-sig eliminates X⟨¯¯¯¯S⟩X⟨S¯⟩ by looking up XX in the type context ΔΔ and substituting the negative and positive occurrences of sorts (αα and ββ). An instantiated sort SS is expanded to a pair of types for substituting αα and ββ, respectively. There are three cases for expanding SS: If SS is a sort, then αα and ββ are kept abstract (E-sort1Sort); if SS has only one type, then both αα and ββ are substituted by that type (E-sort1); otherwise, αα and ββ are substituted by the intersection of the two types (A&BA&B) and the second type (BB) from the pair (E-sort2).

_Subtyping_. Figure [5](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig5) shows the subtyping rules. Most rules come from multiple sources in previous work [Barendregt et al. [1983](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0004); Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005); Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006); , [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)]. The main novelty is the rule that deals with distributivity of trait types (rules S-distTrait), which essentially follows the TL-arr rule for functions (which is inspired by BCD subtyping [Barendregt et al. [1983](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0004)]). The rule S-distTrait distributes intersections over the **Trait** type. In the original work on SEDEL, the elaboration targeted the FiFi calculus, which is a precursor of F+iFi+without distributivity rules. Thus, that work did not have distributivity for traits. The novel distributivity rules for traits are essential for achieving the nested composition of traits (and trait families). The trait rules and other distributive rules allow, for example, ExpSig<Eval>**&** ExpSig<Print> to be subtype of ExpSig<Eval**&** Print>. The rule S-topLike is also novel. It states that any type is a subtype of a top-like type.

![Figure 5](https://dl.acm.org/cms/attachment/e3138ad4-46f0-4f19-8475-55a5aa86b19d/toplas4303-09-f05.jpg)

Fig. 5. Subtyping.

_Top-like types_. Top-like types. Figure [6](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig6) defines top-like types. Top-like types are types isomorphic to ⊤⊤ (i.e., both sub- and supertypes of ⊤⊤), which was a concept first introduced by Barendregt et al. [[1983](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0004)] and then employed by F+iFi+ and its precursors [Oliveira et al. [2016](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0065); Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001); Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)] for proving coherence. We add a new rule TL-trait for trait types, which states that a trait type is top-like when its provided interface is top-like. An important property of top-like types is that they are disjoint to any other types [Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001)]. Furthermore, the definition of top-like types is crucial for defining disjointness and the inclusion of types such as Int→⊤Int→⊤ in the class of top-like types, which is important to ensure the disjointness of function types and in turn enables merges with multiple functions. A more detailed discussion can be found in work by Huang and Oliveira [[2020](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0039)].

![Figure 6](https://dl.acm.org/cms/attachment/4f96e912-6870-4421-adba-4348dfa932d5/toplas4303-09-f06.jpg)

Fig. 6. Top-like types.

_Disjointness_. The disjointness judgment detects the conflicts when merging two expressions of type AA and BB. These rules are omitted here, since they are merely a combination of the rules from F+iFi+ (which can be found in Figure [10](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig10)) and SEDEL. Interested readers can refer to Appendix [A](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-31) for the full disjointness rules of CP .

### 5.4 The Target Language: F+iFi+

The dynamic semantics of CP is given by an elaboration to F+iFi+, which is our target language. F+iFi+ is a calculus with disjoint intersection types, disjoint polymorphism, and nested composition. The semantics and metatheory (including type-safety and coherence) of F+iFi+ have been studied in previous work [Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)]. Here, we give an overview of F+iFi+, focusing on the typing, subtyping, and disjointness relations, which are the necessary aspects to establish our type-safety theorem in Section [5.5](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-22). For more details about the F+iFi+ calculus, the interested reader can consult Bi et al. [[2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)]’s work.

_Syntax_. Figure [7](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig7) gives the syntax of F+iFi+. Metavariable ττ ranges over types. Types include integers IntInt, type variables αα, the top type ⊤⊤, the bottom type ⊥⊥, arrows τ1→τ2τ1→τ2, disjoint quantification ∀(α∗τ1).τ2∀(α∗τ1).τ2, intersections τ1&τ2τ1&τ2, and single-field record types {l:τ}{l:τ}. Metavariable ee ranges over expressions. Expressions include integer literals ii, term variables xx, the top value ⊤⊤, lambda abstractions λx.eλx.e, term applications e1e2e1e2, type abstractions Λ(α∗τ).eΛ(α∗τ).e with αα constrained to be disjoint with ττ, type applications eτeτ, merges e1,,e2e1,,e2, single-field records {l=e}{l=e}, record projections e.le.l and (recursive) let expressions letx:τ=e1ine2letx:τ=e1ine2.

![Figure 7](https://dl.acm.org/cms/attachment/4a57e352-3229-4756-96a1-83f727b70f41/toplas4303-09-f07.jpg)

Fig. 7. F+iFi+ syntax extended with (recursive) let bindings.

_Subtyping_. Figure [8](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig8) shows the subtyping rules of the form τ1<:τ2τ1<:τ2. The subtyping relation is reflective (TS-refl) and transitive (TS-trans). Rules for top types (TS-top), bottom types (TS-bot), function types (TS-arr) and record types (TS-rcd) are standard. The three rules on intersection types (TS-andL, TS-andR, and TS-and) state that τ1&τ2τ1&τ2 is the greatest lower bound for τ1τ1 and τ2τ2. The BCD-style distributive rules [Barendregt et al. [1983](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0004)] (TS-distArr, TS-distRcd, and TS-distAll) are particularly interesting, since they enable nested composition.

![Figure 8](https://dl.acm.org/cms/attachment/a22bac20-e984-49c9-b27e-5aecd1725c0d/toplas4303-09-f08.jpg)

Fig. 8. F+iFi+ subtyping.

_Typing_. Figure [9](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig9) gives the bidirectional type system employed by F+iFi+: under the contexts ΔΔ and ΓΓ, the inference mode (⇒⇒) synthesizes a type ττ from an expression ee while the checking mode (⇐⇐) checks the type of an expression against ττ. ⊢Δ⊢Δ and Δ⊢ΓΔ⊢Γ ensure the well-formedness of contexts. The rule TT-merge infers the merge of two expressions as an intersection type if their types are _disjoint_. The rule TT-tapp additionally checks that the supplied type τ1τ1 is disjoint with the constraining type τ2τ2.

![Figure 9](https://dl.acm.org/cms/attachment/c77f16a8-6a73-4abe-b08a-6e2c3f0b4459/toplas4303-09-f09.jpg)

Fig. 9. F+iFi+ typing rules.

_Disjointness_. The disjointness judgment (Δ⊢τ1∗τ2Δ⊢τ1∗τ2), shown in Figure [10](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig10), ensures[](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig11) that the merge of τ1τ1 and τ2τ2 is conflict-free. The disjointness judgment further relies on the definition of top-like types (⌉τ⌈⌉τ⌈) and a disjoint axiom (τ1∗axτ2τ1∗axτ2). The disjointness axiom contains rules stating that distinct types are disjoint.

![Figure 10](https://dl.acm.org/cms/attachment/a70dbfb6-7a8f-4aff-8515-fda37b4d9d7b/toplas4303-09-f10.jpg)

Fig. 10. F+iFi+ disjointness.

_Semantics of F+iFi+._. The semantics of F+iFi+ is given by elaborating to FcoFco, a variant of System FF extended with products and explicit coercions. Details of FcoFco are beyond the scope of this article. We refer interested readers to the original F+iFi+ paper [Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)]. In the original paper by Bi et al. [[2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)], FcoFco has a call-by-value semantics. Since F+iFi+ is defined by elaboration to FcoFco, it inherits the call-by-value semantics of FcoFco. Here, we assume a call-by-name variant FcoFco, which we expect to be type-sound. We expect this result to be straightforward, since FcoFco is just a minor variant of System F and System F is known to be type-sound in both call-by-value and call-by-name. The translations from CP to F+iFi+ and then to FcoFco are unaffected by the choice of evaluation strategy in FcoFco and simply inherit the evaluation strategy from FcoFco. As we have mentioned earlier, we also assume lazy recursive let bindings, which are not present in F+iFi+. Lazy recursive let bindings are in fact the main motivation to switch to a call-by-name semantics, since they are necessary for the encodings of self references. Although we expect the proof of type-soundness of call-by-name FcoFco and coherence of call-by-name F+iFi+ to easily hold, we leave such validation for future work.

### 5.5 Formal Elaboration

F+iFi+ is a subset of CP excluding declarations and trait-related constructs. Therefore, elaborating the shared constructs is straightforward, and only elaborating constructs specific to CP requires some explanations.

Let us focus on the the elaborated F+iFi+ expressions (gray parts) shown in Figure [5](https://dl.acm.org/doi/fullHtml/10.1145/3460228#fig3). Intuitively, T-tmDecl elaborates a term declaration into a let expression, as illustrated by the elaborations on evalNum, printChild, and expAdd in Section [5.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-19). The gray parts of T-trait and T-new are inherited from SEDEL, which follows Cook and Palsberg [[1989](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0022)]’s denotational semantics of inheritance. Concretely, a trait expression is elaborated to a function that takes selfself as an argument, binds the application result of e1selfe1self to supersuper and returns a merge of the body (e2e2) and supersuper. T-trait is also illustrated by the elaborations on evalNum, printChild, and expAdd. As shown by expAdd, T-new elaborates the expression into a lazy fixed point of selfself. T-mergeTrait elaborates the merge of two traits into a function that takes selfself as an argument and returns the merge of applying the two elaborated traits e1e1 and e2e2 to selfself. Note that the type of the selfself argument for the merged trait is an intersection of the two self types from the two traits being merged. Elaboration on evalNum **,,** expAdd **@**Eval illustrates T-mergeTrait. T-open elaborates an openopen expression into a sequence of letlet expressions, one for each field from the record e2e2. These letlet expressions bind labels to their projections so e2e2 can directly access all the fields from e1e1 without explicit projections. The elaboration on expAdd is a showcase of T-open.

Finally, note that CP ’s types also need to be translated to F+iFi+’s types, because in F+iFi+ there is no Trait[A,B],Trait[A,B], construct. All types, except Trait[A,B],Trait[A,B], have straightforward translations. Trait types are translated to function types in F+iFi+ (following SEDEL) with the rule |Trait[A,B],|=|A|→|B||Trait[A,B],|=|A|→|B|.

### 5.6 Type Safety and Coherence

The elaboration of CP into F+iFi+ is type-safe and coherent. We summarize the key results here. Detailed proofs and other auxiliary lemmas can be found in Section [B](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-32).

The first result is that elaboration is type-safe. To prove this result, we need a few results for some of the auxiliary relations, which are shown next. Note that |⋅||⋅| extends on contexts, denoting that each CP ’s type in the context is translated to a F+iFi+’s type.

Lemma 5.1 (Well-formedness Preservation). If Δ,Σ⊢A⇒BΔ,Σ⊢A⇒B, then |Δ|⊢|B||Δ|⊢|B|.

Proof. By induction on the derivation of the judgment.□

Lemma 5.2 (Disjointness Axiom Preservation). If A∗axBA∗axB, then |A|∗ax|B||A|∗ax|B|.

Proof. Note that |Trait[A,B],|=|A|→|B||Trait[A,B],|=|A|→|B|; the rest are immediate.□

Lemma 5.3 (Subtyping Preservation). If A<:BA<:B, then |A|<:|B||A|<:|B|.

Proof. By structural induction on the subtyping judgment.□

Lemma 5.4 (Disjointness Preservation). If Δ⊢A∗BΔ⊢A∗B, then |Δ|⊢|A|∗|B||Δ|⊢|A|∗|B|.

Proof. By structural induction on the disjointness judgment.□

Then the main type-safety theorem is:

Theorem 5.5 (Type-safety). We have that:

-    If ![](https://dl.acm.org/cms/attachment/9298b773-5484-4f1a-b0ae-9c59785931e9/toplas4303-09-inline290.gif), then |Δ|;|Γ|⊢e⇒|A||Δ|;|Γ|⊢e⇒|A|.  
    

Proof. By structural induction on the typing judgment.□

The second theorem is the coherence of the elaboration:

Theorem 5.6 (Coherence). Each well-typed CP program has a unique elaboration.

Proof. For each elaboration rule, the elaborated F+iFi+ expression in the conclusion is uniquely determined by the elaborated F+iFi+ expressions in the premises. By the coherence property of F+iFi+, we conclude that each well-typed CP program has a unique elaboration. Therefore, CP is coherent.□

_Additional Properties_. There are more properties proved in the F+iFi+ paper, including the decidability of the type system. These properties should easily hold for CP by extending the proofs with a case for trait types. The cases for trait types are essentially similar to the cases of function types (trait types are actually encoded as function types in the elaboration to F+iFi+), so the proof extensions should be straightforward. One thing to notice is that the subtyping relation presented in this article is non-algorithmic due to the existence of a transitive rule (S-trans). An algorithmic variant of the subtyping relation is shown by Bi et al. [[2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)]. For proving decidability, we would need to use an extended version of such algorithmic subtyping with trait types.

## 6 CASE STUDIES

To further demonstrate the applicability of CP , we conducted three case studies. The first case study is a small shallow EDSL for parallel prefix circuits, originally proposed by Hinze [[2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0036)] and further studied by Gibbons and Wu [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0033)]. The second one is a mini interpreter, which integrates and extends the examples in Section [3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-10) and Section [4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-13). The last case study is an implementation of the C0 compiler, inspired by the work of Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)], which compiles a subset of C to JVM instructions. In all three case studies, the need for dealing with extensibility and complex dependencies arises.

### 6.1 Scans

_Overview_. Scans [Hinze [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0036)] is a DSL for describing parallel prefix circuits. Initially, there are five circuit constructors and an operation on circuits: Identity consists of parallel wires; Fan connects the leftmost wire to the rest; Above is a vertical combinator; Beside is a horizontal combinator; Stretch inserts identity wires to stretch the circuit; the operation width computes the width of a circuit. A few constructs and operations are added later, as extensions: RStretch is similar to Stretch but stretches the circuit in the opposite direction; depth computes the depth of a circuit; wellSized checks the well-formedness of a circuit; layout compresses the representation of a circuit. Moreover, wellSized and layout depend on width, and layout is context-sensitive. Thus, the case study of circuits is interesting, because it contains some forms of dependencies. Implementing Scans modularly requires an approach not only to solving the EP but also capable of expressing dependencies. There are already a few implementations written in different languages [Gibbons and Wu [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0033); Zhang and Oliveira [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0087); Bi et al. [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)]. Still, these implementations are not fully satisfying. We compare our implementation with respect to these implementations. Note, however, that there are no dependencies on self-references in this case study. Thus, this case study does not exercise such a form of dependencies.

Scans _in_ _CP_ . The techniques shown in Section [3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-10) are used in modularizing Scanswith CP . The syntax of Scansis captured by a compositional interface:

![](https://dl.acm.org/cms/attachment/3897b2f2-ae4b-494c-8e54-203387ae43dd/toplas4303-09-uf72.gif)

Operations are modeled as trait families that concretely implement the compositional interface. For example, the implementation of wellSized is given below:

![](https://dl.acm.org/cms/attachment/88a5d511-4a29-4fb4-9356-c583aa2b052a/toplas4303-09-uf73.gif)

The trait family wellSized implements CircuitSig<Width \% WellSized>, indicating that it depends on another trait family of CircuitSig that implements Width. As discussed in Section [3.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-12), such dependency is weak and allows us to, for example, call width on c in Stretch. Though the definitions of layout and other trait families are omitted here, they can be similarly defined.

_New variants_. The new constructor RStretch is added by extending the compositional interface:

![](https://dl.acm.org/cms/attachment/7b9c96d2-8c13-4b84-8316-503e68866518/toplas4303-09-uf74.gif)

Accordingly, existing trait families are extended with a case for RStretch, for example:

![](https://dl.acm.org/cms/attachment/84bef813-8ba0-478d-877f-6895ac856527/toplas4303-09-uf75.gif)

Similarly, nWidth, nDepth, and nLayout extend their respective trait families.

Finally, we obtain the full implementation of Scansby composing all the operations as well as a generic trait that constructs a modular circuit (circuit is analogous to expMul in Section [3.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-11)):

![](https://dl.acm.org/cms/attachment/13ae7457-430c-4350-b703-2e51066910bd/toplas4303-09-uf76.gif)

Scans _in Haskell_. Gibbons and Wu [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0033)] implement Scansas a shallow EDSL in Haskell. Variants of Scansare modeled as functions, thus adding new variants is simple through defining new functions. However, multiple (possibly dependent) operations are defined using tuples. For example, wellSized is defined as follows:

![](https://dl.acm.org/cms/attachment/73771792-da18-41b6-b2a9-eb8894557fe7/toplas4303-09-uf77.gif)

In essence, we have to provide implementations for both wellSized and width together in a tuple. Such an implementation is not modular, because whenever a new operation is needed, existing code has to be modified for accommodating new operations.

A follow-up Haskell implementation [Zhang and Oliveira [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0087)] employs a technique commonly known as Finally Tagless [Carette et al. [2009](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0018)], together with a type-class-based encoding of subtyping on tuples, to modularize operation extensions. In essence, such an approach simulates subtyping and inheritance for enabling modular composition of the operations. However, this comes at the cost of boilerplate code and extra complexity due to additional parametrization that is needed to make the encoding work. Moreover, explicit delegations are required for defining dependent operations in the Haskell approach, making the code cumbersome to write. In contrast, CP avoids those issues by having built-in language support for nested composition. Furthermore, compositional interfaces/traits and method patterns make the code quite easy to write.

Scans _in Scala_. Zhang and Oliveira [[2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0087)] present a modular solution in Scala by combining the extensible Interpreter pattern [Wang and Oliveira [2016](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0084)] and Object Algebras [Oliveira and Cook [2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)]. Scansis modeled by a hierarchy of traits, where the root is an interface describing the operations a circuit supports while other traits concretely implement that interface. Adding new operations is done by defining another trait hierarchy that implements the extended interface and inherits existing traits. Through covariantly refining the circuit fields to the type of the extended interface, previously defined operations can be used as dependencies. Although such an implementation is modular, the extensions are linearly added and the dependencies are strong due to the use of inheritance to express dependencies. Alternatively, the new trait hierarchy can be separately defined without inheriting existing ones. This weakens the dependencies but requires some boilerplate for gluing the hierarchies using mixin composition. For the same example of the multiple interpretation of Width and WellSized, we need to do extra compositions after defining them separately:

![](https://dl.acm.org/cms/attachment/b6292514-90e3-423e-a36a-5d9d9c8ba4cc/toplas4303-09-uf78.gif)

Unlike CP , where the composition is done _once_ in the family level, _every_ trait in Scala needs to be composed, because Scala lacks support for nested composition. Another drawback is that Scala's constructors are _not virtual_. Directly calling **new** on constructors for creating objects results in non-modular code. Therefore, Object Algebras [Oliveira and Cook [2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)] are used for abstracting over the constructor calls, resulting in more boilerplate.

Scans _in F+iFi+_ . Bi et al. [[2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)] modularize Scansdirectly in F+iFi+ using extensible records. Note that, because Scansdo not have self-reference dependencies, there is no strict need for using traits, which are not directly supported in F+iFi+. The syntax of Scansis defined similarly to CircuitSig except that Circuit is captured as a type parameter rather than a sort. The operation extensibility is achieved by defining new record instances, while the variant extensibility is achieved by the intersection types and the merge operator. A key difference is how dependent operations are defined:

![](https://dl.acm.org/cms/attachment/2b107033-1d30-4e9a-9d2a-8f6a06d32d34/toplas4303-09-uf79.gif)

Note that wellSized is not given a type. Instead, it defines a record with various fields modeling constructors, and all the “constructor” arguments are explicitly annotated. This illustrates a crucial difference to the CP solution: While in CP wellSized (and other operations) implement a proper interface (via **implements**), in the F+iFi+ encoding that is not the case. The dependency on width is loosely expressed by refining the circuit type as WellSized**&** Width. Such dependency is repeated several times in above and stretch. The lack of a proper interface when implementing operations allows for a relatively compact solution in F+iFi+, but it has some important disadvantages. By implementing an interface in CP , we can ensure various things at the point of the operation definition. For instance, CP checks that: We implement all constructors, and all the constructors are defined with the right number of parameters and types for the parameters. In F+iFi+, such checks are not done when defining operations, which is very error-prone. Errors such as forgetting to implement a constructor or implementing a constructor with the wrong number of parameters or the wrong parameter types cannot be checked at the definition point, but are delayed to the composition point.

_Evaluation_. Besides a qualitative analysis of the aforementioned modular implementations, we further evaluate them in terms of **source lines of code (SLOC)**. To make the comparison fair, we have adapted their implementations to ensure that all the implementations are written in a similar programming style and provide the same functionalities. The SLOC for the modular implementations in Haskell, Scala, F+iFi+, and CP are 87, 129, 72, and 70, respectively. CP ’s solution is the most compact one among the four implementations, while also being the most modular one.

### 6.2 Mini Interpreter

_Overview_. The second case study is a mini interpreter for an expression language. This case study is larger (around 700 SLOC) and more comprehensive than the previous one. Besides the EP and simple dependencies, it covers more forms of dependencies. In particular, self dependencies occur in this case study. Furthermore, the case study contains several uses of S-attributed grammars, polymorphic contexts, as well as multiple sorts.

The expression language consists of various sublanguages, including numeric and Boolean literals, arithmetic expressions, logical expressions, comparisons, if-then-else branches, variable bindings, function closures, and function applications. The supported operations include a few variants of evaluation, pretty-printing, and logging. The sublanguages are separately defined as features [Prehofer [1997](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0070)], using different compositional interfaces and trait families. Through nested composition, these features can be arbitrarily combined to form a _product line of interpreters_ [Pohl et al. [2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0068)].

_Dependencies_. The operations on the expression language contain non-trivial dependencies. Table [1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#tab1) summarizes the different kinds of dependencies used in the mini interpreter. With techniques shown in Section [3.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-12), these dependencies are expressed in a modular way. For example, log is a simple form of logging, which shows the printing and evaluation results of an expression for debugging purposes. Here is a simplified logging implementation for numeric expressions:

Table 1. Different Kinds of Dependencies Used in the Mini Interpreter

**Operation**

**Dependency**

eval

print

print(aux)

log

Child dependencies

![](https://dl.acm.org/cms/attachment/d86734fd-c94c-43ab-be3b-e61976352c20/toplas4303-09-inline304.gif)

Self dependencies

![](https://dl.acm.org/cms/attachment/1ac16617-8efe-4c70-8b59-89538fbaf05a/toplas4303-09-inline305.gif)

![](https://dl.acm.org/cms/attachment/512ee842-bc68-4c9f-a319-6276352278e4/toplas4303-09-inline306.gif)

Mutual dependencies

![](https://dl.acm.org/cms/attachment/3d02ea72-b81b-45be-9b6c-bb140cb56c5f/toplas4303-09-inline307.gif)

Inherited attributes

![](https://dl.acm.org/cms/attachment/912aa9a8-3f44-4302-95f1-dee3843fc3c1/toplas4303-09-inline308.gif)

![](https://dl.acm.org/cms/attachment/b959f1fe-99cc-4835-85f0-0df7b49ca128/toplas4303-09-uf80.gif)

To express self dependencies on eval and print, we annotate Add’s self-type as Eval**&** Print.

_Polymorphic contexts_. There are four different kinds of contexts for evaluation in total: an empty context (Top); a map from strings to numbers for evaluating variable bindings; a map for dynamic scoping; an environment for intrinsic functions. Using techniques shown in Section [4.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-15), we model these contexts as polymorphic contexts to make the code with contexts modular and evolvable.

_Multi-sorted languages_. Examples presented in the article so far are all based on a single-sorted expression language. Essentially, CP allows compositional interfaces to be parameterized by multiple sorts. This ability is demonstrated by the following code excerpt extracted from this case study:

![](https://dl.acm.org/cms/attachment/1214e1d0-0b17-4982-913d-ac8e9e56c3eb/toplas4303-09-uf81.gif)

CmpSig models equality and three-way comparison (also known as the _spaceship_ operator), which returns 0 if the two operands are equal, 1 if the left operand is greater, or −−1 otherwise. They take Numeric arguments and construct Boolean and Numeric traits, respectively. Notice that CmpSig is developed as an independent feature. It can later be combined with other independently developed features such as numeric expressions:

![](https://dl.acm.org/cms/attachment/85dae032-7e66-45ef-b1ee-c3f417cbcf79/toplas4303-09-uf82.gif)

In cmp, we construct an expression with the new constructor Cmp, as well as Lit and Add, which are independently developed before.

### 6.3 C0 Compiler

_Overview_. The C0 compiler was originally an educational one-pass compiler developed for the compilation course at Aarhus University. Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)] translated this compiler into their encoding of Object Algebras, whereas we will present this case study in our approach with Compositional Programming. Then, we will make a comparison with the implementation by Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)], as well as the original non-modular version.

C0 selected a subset of the C programming language, consisting of only integer types, arithmetic/bitwise/comparison operations, a few control flow statements, functions, and basic I/O. In other words, it is also a multi-sorted language, whose interfaces are parameterized by Program, Function, Statement, Expression, and so on. A C0 program consists of function declarations and definitions, which will be compiled into Java bytecode. The original implementation was written in Java and later reimplemented in Scala using Object Algebras by Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)]. Both implementations include a bytecode generator from AST nodes to JVM instructions as well as a recursive descent parser. Since the CP language is currently a research prototype and does not support I/O or complex string manipulation, we eliminate the parsing phase from this case study. Lexical analysis is not the core part of C0 and will not affect the validity of our evaluation.

_Chained attributes_. We have shown various forms of dependencies in terms of _S-attributed grammars_ in Section [3.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-12) and introduced polymorphic contexts to tackle _L-attributed grammars_ in Section [4.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-16). However, there are still other kinds of dependencies in attribute grammars. In the C0 compiler, an attribute can depend on both its children and its parent or siblings. For example, consider an attribute that counts the number of leaf nodes (terminal symbols) occurring to the left or in the subtree of the current node. The attribute equations together with the production rules of Lit and Add are listed below:

E→n{Lit}E→n{Lit}

E.counts=E.counti+1E.counts=E.counti+1(1)

E0→E1 ′′+′′ E2{ Add} E0→E1 ′′+′′ E2{ Add} 

E1.counti=E0.countiE1.counti=E0.counti(2)

E2.counti=E1.countsE2.counti=E1.counts(3)

E0.counts=E2.countsE0.counts=E2.counts(4)

Note that count has two roles: the subscript ii stands for _inherited_ attributes, while ss stands for _synthesized_ ones.

For terminal symbols, we just add one to the inherited number, as Equation ([1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#eq1)) shows. For nonterminal symbols, there are three attribute evaluation rules: Equation ([2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#eq2)) is a trivial inherited attribute depending on its parent; Equation ([3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#eq3)) is more interesting, because it depends on its left sibling; Equation ([4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#eq4)) is a trivial synthesized attribute depending on its child. These three equations compose a traversal of the syntax tree: E1.countiE1.counti inherits from its parent E0.countiE0.counti and then does its own computation on the left subtree to obtain E1.countsE1.counts; E2.countiE2.counti inherits from its left sibling and then traverses the right subtree to obtain E2.countsE2.counts; finally, E0.countsE0.counts synthesizes the attribute from its child.

Such a tree traversal reveals an interesting class of attributes called _chained attributes_ [Kastens and Waite [1994](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0045)]. A chained attribute is both inherited and synthesized. If we regard inherited attributes as _Reader_ Monads and synthesized as _Writer_, then chained attributes correspond to _State_ Monads.

In this case study, HasVariables and HasFunctions are chained attributes. They are used to do bookkeeping for variable and function declarations. Like inherited attributes, we model them with polymorphic contexts, where the inherited part serves as the context of the corresponding synthesized part:

![](https://dl.acm.org/cms/attachment/9ab56ff7-a025-463f-9b96-14b05717274c/toplas4303-09-uf83.gif)

If the chained attribute depends on other attributes, then the context can be easily extended as we described in Section [4.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-15). Here is an example of extending the context with HasOffset:

![](https://dl.acm.org/cms/attachment/807a7a87-0d72-4e47-9492-e33ed8159bfd/toplas4303-09-uf84.gif)

The constructor Param is used to declare a parameter within a function, and the trait implements the chained attribute of the variable environment. (Param id).variables will update the previous environment inh.variables with a new mapping from the given identifier to the current offset, which works as the variable index. Note that, although these two variables have the same name, the former is a chained attribute while the latter is an inherited attribute. Such a delegation forms a tree traversal to do bookkeeping for parameter declarations.

_Comparision_. The code statistics of the aforementioned three C0 implementations are shown in Table [2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#tab2). The original Java implementation inlines semantic actions into the handwritten parser. For the sake of fairness, lexical analysis related lines are not counted and the bytecode prelude is reformatted in the same style as the other two. Although the original code is slightly shorter than CP , it is highly entangled and hinders modularity and extensibility. The operation of code generation is hardcoded in the parsing functions, so it is impossible to add new operations modularly. What is worse, there is no data structure for a syntax tree, leaving no space for extension.

Table 2. Source Lines of Code for the Three Implementations of the C0 Compiler

**Java** (Aarhus University)

SLOC

**Scala** (Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)])

SLOC

**CP**

SLOC

Entangled Compiler

235

Generic

140

Maybe Algebra

12

(Tokenizer excluded)

Trees, Signatures and Combinators

558

Compositional Interfaces

32

Composition and Assembly

101

Attribute Interfaces

32

Attribute Interfaces

8

Algebra Implementations

191

Trait Implementations

216

Bytecode (Reformatted)

25

Bytecode Prelude

25

Bytecode Prelude

25

Main

14

Main

5

Main Example

21

**Total**

274

**Total**

1,052

**Total**

314

To modularize the original C0 implementation, Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)] use an extended form of generalized Object Algebras to model attribute grammars in Scala. It allows L-attributed grammars with different kinds of dependencies to be modularly defined. Due to the lack of the proper composition mechanisms in Scala, the attributes cannot be easily composed, and specialized composition operators have to be explicitly defined. Such boilerplate code largely accounts for the SLOC reported in “Trees, Signatures and Combinators.” In addition, “Composition and Assembly” is the handwritten code to deal with various dependencies. Their “Attribute Interfaces” and “Algebra Implementations” are counterparts of our “Attribute Interfaces” and “Trait Implementations.”

Compared to CP , Rendel et al. [[2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)]’s approach is significantly more verbose (about 3.5 ×× SLOC). In CP , we do not need to write boilerplate code for trait composition and only a few lines of “Compositional Interfaces” are necessary. A workaround they propose to avoid such boilerplate code is to employ meta-programming for generating the specialized signatures and combinators. However, their source code will not be type-checked before macro expansion. Compilation errors will be reported in terms of generated code, which could be confusing for programmers and make it hard to debug. Nevertheless, their assembly mechanism, which relies on specialized combinators that have to be hand-written, encodes the pattern of chained attributes and simplifies the algebra implementations. Our approach of polymorphic contexts is a little more complicated than theirs, but does not require any specialized combinators and works smoothly with nested trait composition.

## 7 RELATED WORK

_Mainstream statically typed OOP and virtual methods_. _Virtual methods_ in OOP provide a way to weaken _method dependencies_. In a virtual method call, such as **this**.m() in a method of a class A, the call _does not_ necessarily refer to the implementation of m() in A. Instead, it may refer to a later implementation in a subclass of A. The choice of the implementation of **this**.m() depends on which subclass of A is used to instantiate the object on which **this**.m() is called. However, virtual methods alone are insufficient to weaken other kinds of dependencies. Most programming languages tend to have _static_ references to both _constructors_ and _types_. For instance, if we refer to a constructor in Java, say **new** A(), then the constructor (unlike the method **this**.m()) _will always refer to the same constructor of class_ A. Such _static_ dependencies create a tight coupling between the use of the constructor and the class A, and make programs less modular than they ought to be. Moreover, most statically typed OOP languages use _(static) inheritance_ pervasively. Inheritance often creates more coupling than needed between method implementations in subclasses and method implementations in the superclass. In a subclass declaration, such as **class** A **extends** B {...}, B must be some concrete class, with (possibly) some concrete method implementations. In other words, inheritance cannot be parametrized, and we cannot program against the _interface_ of the superclass: We must program against some concrete class implementation.

CP adopts virtual methods while also supporting virtual constructors to avoid static references to constructors. Static references to types, which would typically arise from constructors, are avoided by using sorts. Moreover, in CP , most uses of inheritance can be replaced by code with weaker dependencies (see discussion in Section [3.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-12)), thus avoiding the coupling problems introduced by inheritance. Altogether, this leads to code that is more modular and has weaker dependencies than in conventional statically typed OOP.

_Mixins and traits_. Single inheritance supported by many class-based OOP languages is insufficient for code reuse. However, multiple inheritance is hard to do correctly due to the existence of the _diamond problem_. Mixins [Bracha and Cook [1990](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0011)] provide one form of multiple inheritance. The Jigsaw framework [Bracha [1992](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0010)] formalizes mixins and provides a set of operators on mixins. There are other formalizations of mixins proposed for different languages such as ML-like languages [Ancona and Zucca [2002](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0002); Duggan and Sourelis [1996](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0024)] and Java-like languages [Flatt et al. [1998](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0030); Lagorio et al. [2009](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0050)]. Traits [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)] are an alternative to mixins. The fundamental difference between traits and mixins is the way of dealing with conflicts when composing multiple traits/mixins. Mixins _implicitly_ resolve the conflicts according to the composition order whereas the programmer must _explicitly_ resolve the conflicts for traits. The trait model avoids unexpected errors caused by the wrong choice of implementation through implicit resolution. Furthermore, it makes trait composition associative and commutative, and the order of traits being composed does not affect semantics (all permutations have the same behavior). This is in contrast with mixins, where the composition is order-sensitive. Typically classes, mixins, and traits in statically typed languages (such as Scala) are second-class and do not support dynamic inheritance. Dynamic languages like JavaScript can encode quite general forms of mixins and support dynamic inheritance. However, type-checking dynamic inheritance is hard. There is little work on typing first-class classes/mixins/traits. Takikawa et al. [[2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0078)]’s first-class classes in Typed Racket, Lee et al. [[2015](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0051)]’s tagged objects and Bi and Oliveira [[2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)]’s first-class traits are three notable works, which support such features in statically typed languages. Our work follows the first-class trait model by Bi and Oliveira [[2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)]: Traits in CP are first-class, statically typed, and support dynamic inheritance.

_First-class traits and disjoint intersection types_. As discussed in Section [2.4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-9), the work on _disjoint intersection types_ [Oliveira et al. [2016](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0065); Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001); Bi et al. [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0006); , [2019](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0007)] provides a suitable foundation for Compositional Programming. In particular, the F+iFi+ calculus supports disjoint intersection types, disjoint polymorphism, and nested composition, which are key ingredients for Compositional Programming. However, F+iFi+ is still a core calculus and is inconvenient to directly program with. Built upon disjoint intersection types, SEDEL [Bi and Oliveira [2018](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0005)] is a surface language that adds high-level abstraction mechanisms, particularly on the support for _first-class traits_. However, SEDEL is elaborated to FiFi [Alpuim et al. [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0001)], which is a predecessor of F+iFi+ and does not support nested composition. The design of CP improves on the design of SEDEL. The support for _compositional interfaces_, _compositional traits_, _method patterns,_ and _nested trait composition_ are all novel to CP . Thus, the main advantages of CP over SEDEL are the language mechanisms supporting virtual constructors, weak references to datatypes, several new mechanisms to express weaker dependencies (such as compositional interface type refinement), and the ability to compose nested traits via nested composition.

In addition to the new features designed for Compositional Programming, CP further takes the advantage of the unrestricted intersections brought by targeting to F+iFi+ in improving the trait model proposed by SEDEL. The additional support for nested composition on traits not only unifies the syntax by making the merge operator also work on the **inherits** and **new** clauses (see the difference of the code shown, respectively, in Section [2.4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-9) and Section [3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-10)) but also simplifies the typing rules to SEDEL. For example, the typing rule for defining traits in SEDEL is:

¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯Γ,self:B⊢Ei⇒Trait[Bi,Ci],⇝eii∈1..nΓ,self:B,super:C1&..&Cn⊢{¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯lj=E′jj∈1..m}⇒C⇝e¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯B<:Bii∈1..nΓ⊢C1&..&Cn&CC1&..&Cn&C<:AΓ⊢trait[self:B]inherits¯¯¯¯¯¯Eii∈1..n{¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯lj=E′jj∈1..m}:A⇒Trait[B,A],⇝λ(self:|B|).letsuper=¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯(eiself)i∈1..ninsuper,,eΓ,self:B⊢Ei⇒Trait[Bi,Ci],⇝ei¯i∈1..nΓ,self:B,super:C1&..&Cn⊢{lj=Ej′¯j∈1..m}⇒C⇝eB<:Bi¯i∈1..nΓ⊢C1&..&Cn&CC1&..&Cn&C<:AΓ⊢trait[self:B]inheritsEi¯i∈1..n{lj=Ej′¯j∈1..m}:A⇒Trait[B,A],⇝λ(self:|B|).letsuper=(eiself¯)i∈1..ninsuper,,e

This rule, among others, is clearly more complicated than its counterpart ( T-trait) in CP . The complexity is mainly caused by dealing with expression sequences: Every expression needs to be translated and validated. In contrast, CP processes only one expression thanks to the newly introduced rule mergeTrait. mergeTrait checks the disjointness of two traits and infers their merge as a trait type rather than an intersection type, thus reducing the complexity and duplication. Another important benefit of the CP design is the improved support for _type inference_. For example, in SEDEL, a type must be provided to instantiate a trait, but this type is inferred in CP . Moreover, parameters of a method pattern inside a trait can omit types in CP if they are declared by the type specified in the implementsimplements clause. This is quite handy for defining trait families.

_Virtual classes and family polymorphism_. Ideas such as _virtual classes_ [Madsen and Moller-Pedersen [1989](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0055); Ernst et al. [2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0029)] and _family polymorphism_ [Ernst [2001](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0027)] extend the idea of virtual methods to classes. Thus, _classes_ and _constructors_ can themselves be _virtual_, weakening the dependencies to classes and constructors. Virtual classes were first introduced in the BETA programming language [Madsen et al. [1993](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0056)]. BETA supports only single, static inheritance. The composition of BETA programs is done through the fragment system [Knudsen et al. [1994](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0047)]. The gbeta language [Ernst [1999](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0026)] extends BETA with mixins and, more importantly, supports family polymorphism [Ernst [2001](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0027)]. Family polymorphism is a powerful mechanism for extensible and composable software design, which can solve the EP [Ernst [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0028)]. Clarke et al. [[2007](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0021)] classify family polymorphism approaches into object family approaches and class family approaches. In an object family approach, nested classes are attributes of objects of the family class. Some examples of object family approaches are BETA, gbeta, CaesarJ [Aracic et al. [2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0003)] and Newspeak [Bracha et al. [2010](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0012)]. Whereas in a class family approach, nested classes are attributes of the family class. Class family approaches include Concord [Jolly et al. [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0044)], .FJ [Igarashi et al. [2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0041)], Jx [Nystrom et al. [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0059)], J& [Nystrom et al. [2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0060)], and Familia [Zhang and Myers [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0088)]. There are also hybrid approaches like Tribe [Clarke et al. [2007](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0021)]. Object family approaches are typically more expressive but require a more complex dependent type system, e.g., _vc_ [Ernst et al. [2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0029)]. The closest approach is Familia [Zhang and Myers [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0088)], which also supports subtype polymorphism, family polymorphism, and parametric polymorphism but does not support the merge operator.

One difference between CP and the family polymorphism systems is that in those systems, inheritance is still used as a primary mechanism to express dependencies. Similarly to (regular) classes, the use of inheritance in family polymorphism sometimes creates more coupling than necessary between sub- and super-classes/families. In contrast, such dependencies can be weakened via CP ’s support for self-references and compositional interface type refinement, leading to more modular programs. Another difference is that conflicts are often implicitly resolved based on some order in those systems (e.g., gbeta uses the composition order and Jx uses the dispatch order). In contrast, CP adopts an approach where conflicts are explicitly resolved, following the trait model [Schärli et al. [2003](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0074)].

_SOP, MDSoC, AOP, and FOP_. _**Subject-oriented programming**_ **(SOP)** [Harrison and Ossher [1993](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0035)], _**multi-dimensional separation of concerns**_ **(MDSoC)** [Tarr et al. [1999](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0079)], _**aspect-oriented programming**_ **(AOP)** [Kiczales et al. [1997](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0046)], and _**feature-oriented programming**_ **(FOP)** [Prehofer [1997](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0070)] are software paradigms that share a similar vision of _separation of concerns_: i.e., separating a program into different parts so each part addresses a separate concern. Since in those paradigms, a complete program has been separated, a _composition_ mechanism to assemble the parts back together is necessary. Typically SOP, MDSoC, AOP, and FOP employ a _meta-programming_ approach to software composition. Such meta-programming approaches are usually an extension to an existing programming language, such as Hyper/J and AspectJ for Java. A source-to-source compiler will combine the separated aspects before producing plain Java code. Quite often, many of those tools do not fully support modular type-checking or separate compilation.

In contrast, Compositional Programming is a language-based approach, with both clearly defined static and dynamic semantics. The merge operator provides the composition mechanism in Compositional Programming. What distinguishes the elaboration adopted by CP from general meta-programming is that the elaboration is completely transparent for a programmer: (1) Type-checking is done directly in the source language, where type errors (and other well-formedness errors) in programs are reported in terms of the source rather than the target; (2) Type-checking is modular: Each definition can be type-checked with only its implementation and type signatures of the dependencies. Worth noting is that the style of elaboration employed by CP to give the semantics to the language is also adopted by other languages. Most notably the GHC Haskell compiler elaborates the source language (Haskell) into a small core language [Sulzmann et al. [2007](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0076)]. Like CP , all type-checking is done at the source level and the elaboration process is transparent to Haskell programmers. In contrast, in many approaches that employ meta-programming, often there is no source-level type-checking or even some more basic error-checking like syntax well-formedness. Consequently, no modular type-checking is offered and errors are reported on the generated program, which are hard to understand.

_ML modules_. The design of CP is partly inspired by ML module systems [MacQueen [1984](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0054)]. Analogously to compositional interfaces and trait families in CP , ML signatures and structures can be used to specify and implement the constructors. However, unlike CP , ML modules are neither extensible nor first-class. There are many proposals to extend the ML modules with additional expressiveness, such as making modules first-class [Russo [2000](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0072)] or recursive [Russo [2001](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0073)]. Together with other features, ML modules can also be used in solving the EP [Nakata and Garrigue [2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0058)].

_Scala_. CP is also influenced by Scala [Odersky et al. [2004](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0061)], where features such as intersection types, traits, and self-types are shared. As compared in Section [2.4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-9), Scala's traits are not first-class and do not support dynamic inheritance and nested composition. Nevertheless, Scala supports virtual types [Madsen and Moller-Pedersen [1989](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0055); Igarashi and Pierce [1999](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0040)], which can be used for simulating family polymorphism but in a much more verbose way [Odersky and Zenger [2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0062)]. In CP , sorts are elaborated to type parameters. Another option is to use virtual types. The strengths and weaknesses of type parameters and virtual types are summarized by Bruce et al. [[1998](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0014)]: Type parameters are flexible in composing and decomposing types while virtual types are good at specifying mutually recursive families of classes whose relationships are preserved in the derived family. Type parameters are a natural choice for CP , since the underlying F+iFi+ calculus supports disjoint polymorphism. In future work, we would like to explore design with virtual types.

_Algebraic signatures and Object Algebras_. Readers familiar with algebraic signatures [Guttag and Horning [1978](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0034)] used in algebraic specification languages may observe some similarities to compositional interfaces. Algebraic signatures also allow the definition of constructors. However, the semantics of constructors in compositional interfaces differs from those in conventional algebraic signatures. The key difference is that the positive and negative occurrences of sorts are distinguished in CP , which is important to support an advanced form of modular dependencies but not well-supported with algebraic signatures.

Oliveira and Cook [[2012](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0063)] explored an encoding of algebraic signatures in OOP languages, yielding a solution to the EP called _Object Algebras_. Other design patterns, such as _Polymorphic Embeddings_ [Hofer et al. [2008](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0038)] and _Finally Tagless_ [Carette et al. [2009](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0018)] use similar encodings. Such encodings originate from the work by Hinze [[2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0037)] and Oliveira et al. [[2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0064)]. Hinze [[2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0037)] showed how to model Church encodings of GADTs [Cheney and Hinze [2002](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0019)] using Haskell type classes. Oliveira et al. [[2006](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0064)] then showed that such type class based encoding can also solve the EP. Object Algebras use parametric interfaces and classes to represent and implement the algebraic signatures, respectively. As discussed in Section [2.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-8), Object Algebras, in their basic form, are hard to be composed and express dependencies. These limitations are later addressed in Scala by means of _generalized Object Algebras_ and intersection types, together with specialized combinators [Oliveira et al. [2013](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0066); Rendel et al. [2014](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0071)]. However, the Scala approaches based on reflection or meta-programming have important drawbacks, as discussed in Section [6.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#sec-27). In contrast, CP has built-in language support for sorts and nested composition, which is more convenient to use and avoids the use of reflection or meta-programming techniques.

_Alternatives to context evolution._. There are other approaches to context evolution. Monads [Wadler [1992](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0082)], and in particular Reader Monads, can be helpful for context evolution. Nevertheless, to deal with multiple contexts, we would like to have multiple Reader Monads. Unfortunately, this causes some issues in a monadic setting. A notorious problem with Monads is that the composition of two Monads (which would be useful to compose contexts) is not always a Monad. Thus, there is no simple way to compose multiple Monads in general. It is possible to use _Monad transformers_ [Liang et al. [1995](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0053)] to compose multiple Monads of different kinds (for example, a Reader and a Writer Monad or a Reader and the IO Monad). However, composing Monads of the same kind (like two Reader Monads) is problematic. Only advanced techniques [Jaskelioff [2008](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0043); Schrijvers and Oliveira [2011](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0075)], which require significant sophistication, improve on traditional Monad transformers and can deal smoothly with multiple modular contexts. _Implicit context propagation_ [Inostroza and Storm [2015](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0042)] is another approach in OOP, which requires writing boilerplate code for adapting previous code to accept new contexts (although such boilerplate can be generated). Polymorphic contexts provide us with a simple approach to modularly handle them without changing the existing code, while keeping strong static type safety.

## 8 CONCLUSION

We have presented key concepts of Compositional Programming together with a language design and implementation called CP . CP ’s support for compositional interfaces, compositional traits, virtual constructors, and method patterns enables a programming style that allows programs with non-trivial dependencies to be modularized. The applicability of CP is demonstrated by various examples and case studies. The calculus that captures the essence of CP is proved to be type-safe.

We envision Compositional Programming as an alternative programming style and paradigm to what is currently offered in both functional programming and object-oriented programming. Compositional Programming borrows many ideas from both paradigms. The style of Compositional Programming presented in this article is essentially a purely functional programming style and draws inspiration from languages like Haskell. A purely functional style has benefits in terms of reasoning about programs and it also simplifies some issues related to the composition of code. Multiple inheritance in the presence of mutable state is a notorious problem, for instance. However, Compositional Programming also borrows ideas from object-oriented programming, namely, by employing subtyping and nested composition (which is closely related to inheritance). This mix of ideas, together with some new ideas, results in a language that supports highly modular programs in a natural way.

_Future work_. As CP is a prototype design for Compositional Programming, there is still a lot of room for making it more expressive and practical. Some possible directions for future work include the addition of _recursive types_ and _type constructors_, _mutable state_ for modeling imperative objects, and improvements on _type inference_.

In our view, the addition of recursive types is most pressing, as there are many use cases for such signatures. For example, with recursive types, we can model binary methods [Bruce et al. [1996](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0013)] or operations that return the same type that is being processed. For example, with recursive types, we should be able to model the double operation described by Zenger and Odersky [[2005](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0085)]. This operation doubles the literals in an arithmetic expression, where each constructor implements the following interface:

![](https://dl.acm.org/cms/attachment/d395a592-9936-4341-9885-cac4bd20bef7/toplas4303-09-uf85.gif)

Exp is captured as a recursive type. However, supporting type constructors allows us to model, for example, the compositional interface of streams adapted from Biboudis et al. [[2015](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0008)]’s work:

![](https://dl.acm.org/cms/attachment/89b16a2b-1b3f-48c8-b918-df6167bc3c51/toplas4303-09-uf86.gif)

where the sort F is a type constructor (i.e., a function on types). Extending CP with recursive types and type constructors is non-trivial. The first step is to study how recursive types and type constructors interact with disjoint intersection types and other features of CP .

Although the programming style of CP is functional, a natural question is whether the ideas of Compositional Programming can be adapted into a programming model with imperative objects. There are several challenges here. One of them is to see how mutable state can be integrated into calculi with disjoint intersection types and a merge operator. A starting point in this direction is the work by Blaauwbroek [[2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0009)], which studies the addition of mutable references into calculus with intersection types and a merge operator. Another general challenge is that, once imperative objects are supported, we must face the issues of multiple inheritance in the presence of mutable state, which is a well-known source of problems.

Finally, CP has some support for type inference, such as inferring the constructor parameters from the **implements** clause. However, this support is rather limited. In particular, uses of polymorphic definitions must explicitly pass all type arguments. It will be interesting to investigate _local type inference_ [Pierce and Turner [2000](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0067)] to infer some of those arguments and improve the convenience of using polymorphic definitions. A more ambitious direction would be looking into MLsub [Dolan and Mycroft [2017](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0023)] and see whether it would be possible to adapt or extend MLsub type inference to CP . The work by van den Berg [[2020](https://dl.acm.org/doi/fullHtml/10.1145/3460228#Bib0081)] is a starting point in this direction.

## APPENDIX

## A FULL TYPE SYSTEM

Δ,Σ⊢A⇒BΔ,Σ⊢A⇒B

![](https://dl.acm.org/cms/attachment/66f3226b-fb69-4fc8-a768-d9a86a2fcc42/toplas4303-09-ueqn5.gif)

Σ⊢S⇒⟨A,B⟩Σ⊢S⇒⟨A,B⟩

![](https://dl.acm.org/cms/attachment/a4d1d642-6c49-4a6a-8c80-efda3cd1ef22/toplas4303-09-ueqn6.gif)

Σ⊢cpA⇒BΣ⊢pcA⇒B

![](https://dl.acm.org/cms/attachment/a1f5b2b4-19cc-4ce3-937d-f0314b80b661/toplas4303-09-ueqn7.gif)

A<:BA<:B

![](https://dl.acm.org/cms/attachment/6afa750c-a41c-4f44-b546-a9980bcfb0a0/toplas4303-09-ueqn8.gif)

⌉A⌈⌉A⌈

![](https://dl.acm.org/cms/attachment/478c8d47-4f28-4c40-96ec-5ea25cfae7b0/toplas4303-09-ueqn9.gif)

Δ⊢A∗BΔ⊢A∗B

![](https://dl.acm.org/cms/attachment/2fd210ab-119b-4315-9965-a26930f69f7c/toplas4303-09-ueqn10.gif)

A∗axBA∗axB

![](https://dl.acm.org/cms/attachment/d8e49c7f-59be-408c-9625-56ae60f9883c/toplas4303-09-inline333.gif)

![](https://dl.acm.org/cms/attachment/e1cd16e7-b126-4263-b913-a661d3a7bea4/toplas4303-09-ueqn11.gif)

![](https://dl.acm.org/cms/attachment/6e0f6d52-16da-49d6-bc53-9ce641f898f1/toplas4303-09-ueqn11a.gif)

![](https://dl.acm.org/cms/attachment/d2868d16-e667-4a17-a38f-f8393d5096aa/toplas4303-09-inline334.gif)

![](https://dl.acm.org/cms/attachment/ebec8972-317a-4ce9-a44d-41821e378549/toplas4303-09-ueqn12.gif)

![](https://dl.acm.org/cms/attachment/f2f3eae6-692d-43c1-a292-65c7d282c05a/toplas4303-09-ueqn13.gif)

![](https://dl.acm.org/cms/attachment/daba3897-3abc-4857-b223-a2da532ddbd3/toplas4303-09-inline335.gif)

![](https://dl.acm.org/cms/attachment/c36b0fce-b66e-4111-bae7-f000150a4e4d/toplas4303-09-ueqn14.gif)

## B METATHEORY

Lemma B.1 (Well-formedness Preservation). If Δ,Σ⊢A⇒BΔ,Σ⊢A⇒B, then |Δ|⊢|B||Δ|⊢|B|.

Proof. By simple induction on the derivation of the judgment.□

Lemma B.2 (Disjointness Axiom Preservation). If A∗axBA∗axB, then |A|∗ax|B||A|∗ax|B|.

Proof. Note that |Trait[A,B],|=|A|→|B||Trait[A,B],|=|A|→|B|; the rest are immediate.□

Lemma B.3 (Subtyping Preservation). If A<:BA<:B, then |A|<:|B||A|<:|B|.

Proof. Most of them are just F+iFi+ subtyping. We only show the rule S-trait

![](https://dl.acm.org/cms/attachment/cfdcdbf6-55b7-4280-9033-080078d8c2ad/toplas4303-09-ueqn16.gif)

|A2|<:|A1|By i.h.|A2|<:|A1|By i.h.

|B1|<:|B2|By i.h.|B1|<:|B2|By i.h.

|A1|→|B1|<:|A2|→|B2|By TS-ARR|A1|→|B1|<:|A2|→|B2|By TS-ARR □

Lemma B.4 (Disjointness Preservation). If Δ⊢A∗BΔ⊢A∗B, then |Δ|⊢|A|∗|B||Δ|⊢|A|∗|B|.

Proof. By induction on the derivation of the judgment.

-    D-topL, D-topR, and D-rcdNeq are immediate.  
    
-   ![](https://dl.acm.org/cms/attachment/0c6a4b6e-57b8-4c00-b2b1-5ca5dc98b184/toplas4303-09-ueqn17.gif)
    
    |A|<:|B||A|<:|B|By Lemma [B.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm9)  
    a∗A∈Δa∗A∈ΔGiven  
    a∗|A|∈|Δ|a∗|A|∈|Δ|     Above  
    |Δ|⊢α∗|B||Δ|⊢α∗|B|     By TD-tvarL  
    
-   ![](https://dl.acm.org/cms/attachment/8f3ee253-6cdb-4350-8a44-2028d74bdecf/toplas4303-09-ueqn18.gif)
    
    |A|<:|B||A|<:|B| By Lemma [B.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm9)  
    a∗A∈Δa∗A∈Δ Given  
    a∗|A|∈|Δ|a∗|A|∈|Δ| Above  
    |Δ|⊢|B|∗α|Δ|⊢|B|∗α By TD-tvarR  
    
-   ![](https://dl.acm.org/cms/attachment/1c62b07c-9b33-4163-9b3f-385e6b99b810/toplas4303-09-ueqn19.gif)
    
    |Δ|,α∗|A1|&|A2|⊢|B1|∗|B2||Δ|,α∗|A1|&|A2|⊢|B1|∗|B2| By i.h.  
    |Δ|⊢∀(α∗|A1|).B1∗∀(α∗|A2|).|B2||Δ|⊢∀(α∗|A1|).B1∗∀(α∗|A2|).|B2|By TD-forall  
    
-   ![](https://dl.acm.org/cms/attachment/9cf13229-b979-410b-b033-16ae241cd1c3/toplas4303-09-ueqn20.gif)
    
    |Δ|⊢|A|∗|B||Δ|⊢|A|∗|B|By i.h.  
    |Δ|⊢{l:|A|}∗{l:|B|}|Δ|⊢{l:|A|}∗{l:|B|}By TD-rcdEq  
    
-   ![](https://dl.acm.org/cms/attachment/497c473e-6222-436a-b6bc-0548e155ead6/toplas4303-09-ueqn21.gif)
    
    |Δ|⊢|A2|∗|B2||Δ|⊢|A2|∗|B2| By i.h.  
    |Δ|⊢|A1|→|A2|∗|B1|→|B2||Δ|⊢|A1|→|A2|∗|B1|→|B2| By TD-arr  
    
-   ![](https://dl.acm.org/cms/attachment/cc4ed74e-157d-4d5f-b097-ff5df12b485e/toplas4303-09-ueqn22.gif)
    
    |Δ|⊢|A1|∗|B||Δ|⊢|A1|∗|B| By i.h..  
    |Δ|⊢|A2|∗|B||Δ|⊢|A2|∗|B| By i.h..  
    |Δ|⊢|A1|&|A2|∗|B||Δ|⊢|A1|&|A2|∗|B| By TD-andL  
    
-   ![](https://dl.acm.org/cms/attachment/70351856-4d70-407c-993b-420054822796/toplas4303-09-ueqn23.gif)
    
    |Δ|⊢|A|∗|B1||Δ|⊢|A|∗|B1| By i.h..  
    |Δ|⊢|A|∗|B2||Δ|⊢|A|∗|B2| By i.h..  
    |Δ|⊢|A|∗|B1|&|B2||Δ|⊢|A|∗|B1|&|B2| By TD-andR  
    
-   ![](https://dl.acm.org/cms/attachment/6025f999-3b42-40c5-859f-40242aa41edd/toplas4303-09-ueqn24.gif)
    
    |Δ|⊢|B1|∗|B2||Δ|⊢|B1|∗|B2| By i.h..  
    |Δ|⊢|A1|→|B1|∗|A2|→|B2||Δ|⊢|A1|→|B1|∗|A2|→|B2| By TD-arr  
    
-   ![](https://dl.acm.org/cms/attachment/8d902df5-d73c-42b8-8cf6-af1a310033be/toplas4303-09-ueqn25.gif)
    
    |Δ|⊢|B1|∗|B2||Δ|⊢|B1|∗|B2| By i.h..  
    |Δ|⊢|A1|→|B1|∗|A2|→|B2||Δ|⊢|A1|→|B1|∗|A2|→|B2| By TD-arr  
    
-   ![](https://dl.acm.org/cms/attachment/13b5e30f-dc04-421d-b2f3-d0fd44e136d1/toplas4303-09-ueqn26.gif)
    
    |Δ|⊢|B1|∗|B2||Δ|⊢|B1|∗|B2| By i.h..  
    |Δ|⊢|A1|→|B1|∗|A2|→|B2||Δ|⊢|A1|→|B1|∗|A2|→|B2| By TD-arr  
    
-   ![](https://dl.acm.org/cms/attachment/2ff4b138-c33d-4d03-a9bb-c1296f6d1cf0/toplas4303-09-ueqn27.gif)
    
    |A|∗ax|B||A|∗ax|B| By Lemma [B.2](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm8)  
    |Δ|⊢|A|∗|B||Δ|⊢|A|∗|B| By TD-ax  
    

□

Theorem B.5 (Type-safety). We have that:

-    If ![](https://dl.acm.org/cms/attachment/e18865bc-1427-4824-8407-fe33f7cf5a69/toplas4303-09-inline377.gif), then |Δ|;|Γ|⊢e⇒|A||Δ|;|Γ|⊢e⇒|A|.  
    
-    If ![](https://dl.acm.org/cms/attachment/27890f2f-15c1-4457-acd8-2d79d9e5ffe5/toplas4303-09-inline379.gif), then |Δ|;|Γ|⊢e⇒|A||Δ|;|Γ|⊢e⇒|A|.  
    
-    If ![](https://dl.acm.org/cms/attachment/e892d1bb-e6ff-41d7-90aa-773cbd768763/toplas4303-09-inline381.gif), then |Δ|;|Γ|⊢e⇐|A||Δ|;|Γ|⊢e⇐|A|.  
    

Proof. By induction on the typing judgment.

-   ![](https://dl.acm.org/cms/attachment/ae0a08c7-b48a-4ce0-a8c6-096c8018bd90/toplas4303-09-ueqn28.gif)
    
    |Δ|;|Γ|⊢e⇒|C||Δ|;|Γ|⊢e⇒|C| By i.h..  
    
-   ![](https://dl.acm.org/cms/attachment/1f2fcfdd-eb55-47cc-bfb1-c4b8601989b6/toplas4303-09-ueqn29.gif)
    
    |Δ|;|Γ|⊢e1⇒|A||Δ|;|Γ|⊢e1⇒|A| By i.h..  
    |Δ|;|Γ|,x:|A|⊢e2⇒|B||Δ|;|Γ|,x:|A|⊢e2⇒|B| By i.h..  
    |Δ|;|Γ|⊢letx:|A|=e1ine2⇒|B||Δ|;|Γ|⊢letx:|A|=e1ine2⇒|B| By TT-let  
    
-    T-top, T-nat, and T-var are immediate.  
    
-   ![](https://dl.acm.org/cms/attachment/cc3ba27d-78bc-4aab-8cc3-4d3ac84da105/toplas4303-09-ueqn30.gif)
    
    |Δ|;|Γ|⊢e1⇒|A1|→|A2||Δ|;|Γ|⊢e1⇒|A1|→|A2| By i.h..  
    |Δ|;|Γ|⊢e2⇐|A2||Δ|;|Γ|⊢e2⇐|A2| By i.h..  
    |Δ|;|Γ|⊢e1e2⇒|A2||Δ|;|Γ|⊢e1e2⇒|A2| By TT-app  
    
-   ![](https://dl.acm.org/cms/attachment/05fbe497-2b86-4ae7-a80f-5cd807320335/toplas4303-09-ueqn31.gif)
    
    |Δ|;|Γ|⊢e⇐|B||Δ|;|Γ|⊢e⇐|B| By i.h..  
    |Δ|;|Γ|⊢e:|B|⇒|B||Δ|;|Γ|⊢e:|B|⇒|B| By TT-anno  
    
-   ![](https://dl.acm.org/cms/attachment/db58a839-1014-4a96-98ea-f1d3a208f746/toplas4303-09-ueqn32.gif)
    
    |Δ|;|Γ|⊢e⇒|A||Δ|;|Γ|⊢e⇒|A| By i.h..  
    |Δ|;|Γ|⊢{l=e}⇒{l:|A|}|Δ|;|Γ|⊢{l=e}⇒{l:|A|} By TT-rcd  
    
-   ![](https://dl.acm.org/cms/attachment/39251bd1-0c58-45e4-a44a-3c4b948b36cf/toplas4303-09-ueqn33.gif)
    
    |Δ|;|Γ|⊢e⇒{l:|A|}|Δ|;|Γ|⊢e⇒{l:|A|} By i.h..  
    |Δ|;|Γ|⊢e.l⇒|A||Δ|;|Γ|⊢e.l⇒|A| By TT-proj  
    
-   ![](https://dl.acm.org/cms/attachment/9e44b2b2-6412-4ed4-9141-a6d6e9b6bd66/toplas4303-09-ueqn34.gif)
    
    |Δ|⊢|A1||Δ|⊢|A1| By Lemma [B.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm7)  
    |Δ|;|Γ|,α∗|A|⊢e⇒|B||Δ|;|Γ|,α∗|A|⊢e⇒|B| By i.h..  
    |Δ|;|Γ|⊢Λ(α∗|A|).e⇒∀(α∗|A|).|B||Δ|;|Γ|⊢Λ(α∗|A|).e⇒∀(α∗|A|).|B| By TT-tabs  
    
-   ![](https://dl.acm.org/cms/attachment/5be1cb6a-fd23-4b92-b895-6c1f8ab938d1/toplas4303-09-ueqn35.gif)
    
    |Δ|⊢|A1||Δ|⊢|A1| By Lemma [B.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm7)  
    |Δ|;|Γ|⊢e⇒∀(α∗|B|).|C||Δ|;|Γ|⊢e⇒∀(α∗|B|).|C| By i.h..  
    |Δ|;|Γ|⊢|A1|∗|B||Δ|;|Γ|⊢|A1|∗|B| By Lemma [B.4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm10)  
    |Δ|;|Γ|⊢e|A|⇒[|A1|/α]|C||Δ|;|Γ|⊢e|A|⇒[|A1|/α]|C| By TT-tapp  
    
-   ![](https://dl.acm.org/cms/attachment/3bea532b-f60f-4baa-a5b9-f6274ee16a72/toplas4303-09-ueqn36.gif)
    
    |Δ|⊢|A1||Δ|⊢|A1| By Lemma [B.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm7)  
    |Δ|;|Γ|,x:|A1|⊢e1⇐|A||Δ|;|Γ|,x:|A1|⊢e1⇐|A| By i.h..  
    |Δ|;|Γ|,x:|A1|⊢e2⇒|B||Δ|;|Γ|,x:|A1|⊢e2⇒|B| By i.h..  
    |Δ|;|Γ|⊢letx:|A1|=E1inE2⇒|B||Δ|;|Γ|⊢letx:|A1|=E1inE2⇒|B| By TT-let  
    
-   ![](https://dl.acm.org/cms/attachment/855d5041-1512-4031-9780-3f855e99340c/toplas4303-09-ueqn37.gif)
    
    |Δ|;|Γ|⊢e1⇒|A1→B1||Δ|;|Γ|⊢e1⇒|A1→B1| By i.h..  
    |Δ|;|Γ|⊢e2⇒|A2→B2||Δ|;|Γ|⊢e2⇒|A2→B2| By i.h..  
    |Δ|;|Γ|,self:|A1&A2|⊢self⇒|A1&A2||Δ|;|Γ|,self:|A1&A2|⊢self⇒|A1&A2| By TT-var  
    |A1&A2|<:|A1||A1&A2|<:|A1| By TS-andL  
    |Δ|;|Γ|⊢self⇐|A1||Δ|;|Γ|⊢self⇐|A1| By TT-sub  
    |Δ|;|Γ|⊢e1self⇒|B1||Δ|;|Γ|⊢e1self⇒|B1| By TT-app  
    |A1&A2|<:|A2||A1&A2|<:|A2| By TS-andR  
    |Δ|;|Γ|⊢self⇐|A2||Δ|;|Γ|⊢self⇐|A2| By TT-sub  
    |Δ|;|Γ|⊢e2self⇒|B2||Δ|;|Γ|⊢e2self⇒|B2| By TT-app  
    |Δ|;|Γ|⊢e1self,,e2self⇒|B1&B2||Δ|;|Γ|⊢e1self,,e2self⇒|B1&B2| By TT-merge  
    |Δ|;|Γ|⊢λ(self:|A1&A2|).e1self,,e2self⇒|A1&A2|→|B1&B2||Δ|;|Γ|⊢λ(self:|A1&A2|).e1self,,e2self⇒|A1&A2|→|B1&B2| By TT-abs  
    
-   ![](https://dl.acm.org/cms/attachment/12218f2e-8db2-4443-8956-297ecb527b2b/toplas4303-09-ueqn38.gif)
    
    |Δ|;|Γ|⊢e1⇒|A||Δ|;|Γ|⊢e1⇒|A| By i.h..  
    |Δ|;|Γ|⊢e2⇒|B||Δ|;|Γ|⊢e2⇒|B| By i.h..  
    |Δ|;|Γ|⊢|A|∗|B||Δ|;|Γ|⊢|A|∗|B| By Lemma [B.4](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm10)  
    |Δ|;|Γ|⊢e1,,e2⇒|A|&|B||Δ|;|Γ|⊢e1,,e2⇒|A|&|B| By TT-merge  
    
-   ![](https://dl.acm.org/cms/attachment/ccc81503-bdfa-4935-8471-b8869d5c7d2c/toplas4303-09-ueqn39.gif)
    
    |Δ|;|Γ|⊢e⇒|A|→|B||Δ|;|Γ|⊢e⇒|A|→|B| By i.h..  
    |B|<:|A||B|<:|A| By Lemma [B.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm9)  
    |Δ|;|Γ|,self:|B|⊢self⇒|B||Δ|;|Γ|,self:|B|⊢self⇒|B| By TT-var  
    |Δ|;|Γ|,self:|B|⊢self⇐|A||Δ|;|Γ|,self:|B|⊢self⇐|A| By TT-sub  
    |Δ|;|Γ|,self:|B|⊢eself⇐|B||Δ|;|Γ|,self:|B|⊢eself⇐|B| By TT-app  
    |Δ|;|Γ|⊢letself:|B|=eselfinself⇒|B||Δ|;|Γ|⊢letself:|B|=eselfinself⇒|B| By TT-let  
    
-   ![](https://dl.acm.org/cms/attachment/599effee-f728-4edb-9d28-c6fc56994179/toplas4303-09-ueqn40.gif)
    
    |Δ|;|Γ|⊢e1⇒|¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯{li:Ai}||Δ|;|Γ|⊢e1⇒|{li:Ai}¯| By i.h..  
    |Δ|;|Γ|⊢e1.li⇒|Ai||Δ|;|Γ|⊢e1.li⇒|Ai| By TT-proj  
    |Δ|;|Γ|⊢¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯¯letli:|Ai|=e1.liine2⇒|B||Δ|;|Γ|⊢letli:|Ai|=e1.liin¯e2⇒|B| By TT-let  
    
-   ![](https://dl.acm.org/cms/attachment/04e7458f-d274-4d16-a4d3-a116937cf882/toplas4303-09-ueqn41.gif)
    
    |Δ|⊢|A1||Δ|⊢|A1| By Lemma [B.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm7)  
    |Δ|⊢|B1||Δ|⊢|B1| By Lemma [B.1](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm7)  
    |Δ|;|Γ|,self:|A1|⊢e1⇒|A2|→|B2||Δ|;|Γ|,self:|A1|⊢e1⇒|A2|→|B2| By i.h..  
    |A1|<:|A2||A1|<:|A2| By Lemma [B.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm9)  
    |Δ|;|Γ|,self:|A1|,super:|B2|⊢e2⇒|C||Δ|;|Γ|,self:|A1|,super:|B2|⊢e2⇒|C| By i.h..  
    |Δ|;|Γ|,self:|A1|⊢self⇒|A1||Δ|;|Γ|,self:|A1|⊢self⇒|A1| By TT-var  
    |Δ|;|Γ|,self:|A1|⊢self⇐|B1||Δ|;|Γ|,self:|A1|⊢self⇐|B1| By TT-sub  
    |Δ|;|Γ|,self:|A1|⊢e1self⇒|B2||Δ|;|Γ|,self:|A1|⊢e1self⇒|B2| By TT-tapp  
    |Δ|;|Γ|,self:|A1|,super:|B2|⊢e2,,super⇒|C|&|B2||Δ|;|Γ|,self:|A1|,super:|B2|⊢e2,,super⇒|C|&|B2| By TT-merge  
    |Δ|;|Γ|⊢letsuper=e1selfine2,,super⇒|C|&|B2||Δ|;|Γ|⊢letsuper=e1selfine2,,super⇒|C|&|B2| By TT-let  
    |Δ|;|Γ|⊢λ(self:|A1|).letsuper=e1selfine2,,super⇒|A1|→(|C|&|B2|)|Δ|;|Γ|⊢λ(self:|A1|).letsuper=e1selfine2,,super⇒|A1|→(|C|&|B2|) By TT-abs  
    
-   ![](https://dl.acm.org/cms/attachment/0388bb17-e049-4bbf-9b84-735c4a8d9a87/toplas4303-09-ueqn42.gif)
    
    |Δ|;|Γ|⊢e1⇒|A|→|B||Δ|;|Γ|⊢e1⇒|A|→|B| By i.h..  
    |Δ|;|Γ|⊢e2⇐|A||Δ|;|Γ|⊢e2⇐|A| By i.h..  
    |Δ|;|Γ|⊢e1e2|Δ|;|Γ|⊢e1e2 By TT-app  
    
-   ![](https://dl.acm.org/cms/attachment/3bdfd8ce-d471-47a5-954f-98b6f33615c0/toplas4303-09-ueqn43.gif)
    
    |Δ|;|Γ|,x:|A|⊢e⇐|B||Δ|;|Γ|,x:|A|⊢e⇐|B| By i.h..  
    |Δ|;|Γ|⊢λ(x:e).⇐|A|→|B||Δ|;|Γ|⊢λ(x:e).⇐|A|→|B| By TT-abs  
    
-   ![](https://dl.acm.org/cms/attachment/ee2f716e-f80d-436e-88e4-cd26168cc2a9/toplas4303-09-ueqn44.gif)
    
    |Δ|;|Γ|⊢e⇒|B||Δ|;|Γ|⊢e⇒|B| By i.h..  
    |B|<:|A||B|<:|A| By Lemma [B.3](https://dl.acm.org/doi/fullHtml/10.1145/3460228#thm9)  
    |Δ|;|Γ|⊢e⇐|B||Δ|;|Γ|⊢e⇐|B| By TT-sub  
    

□

## ACKNOWLEDGMENTS

We thank anonymous reviewers for their helpful comments. The first author conducted this research while at the University of Hong Kong.

## REFERENCES

-   João Alpuim, Bruno C. d. S. Oliveira, and Zhiyuan Shi. 2017. Disjoint polymorphism. In _Proceedings of the European Symposium on Programming_. Springer, 1–28.
-   Davide Ancona and Elena Zucca. 2002. A calculus of module systems. _J. Funct. Prog._ 12, 2 (2002), 91–132.
-   Ivica Aracic, Vaidas Gasiunas, Mira Mezini, and Klaus Ostermann. 2006. An overview of CaesarJ. In _Transactions on Aspect-oriented Software Development I_. Springer, 135–173.
-   Henk Barendregt, Mario Coppo, and Mariangiola Dezani-Ciancaglini. 1983. A filter lambda model and the completeness of type assignment 1. _J. Symb. Logic_ 48, 4 (1983), 931–940.
-   Xuan Bi and Bruno C. d. S. Oliveira. 2018. Typed first-class traits. In _Proceedings of the 32nd European Conference on Object-oriented Programming (ECOOP’18)_. Schloss Dagstuhl-Leibniz-Zentrum fuer Informatik.
-   Xuan Bi, Bruno C. d. S. Oliveira, and Tom Schrijvers. 2018. The essence of nested composition. In _Proceedings of the 32nd European Conference on Object-oriented Programming (ECOOP’18)_. Schloss Dagstuhl-Leibniz-Zentrum fuer Informatik.
-   Xuan Bi, Ningning Xie, Bruno C. d. S. Oliveira, and Tom Schrijvers. 2019. Distributive disjoint polymorphism for compositional programming. In _Proceedings of the European Symposium on Programming_. Springer, 381–409.
-   Aggelos Biboudis, Nick Palladinos, George Fourtounis, and Yannis Smaragdakis. 2015. Streams a la carte: Extensible pipelines with object algebras. In _Proceedings of the 29th European Conference on Object-oriented Programming (ECOOP’15)_. Schloss Dagstuhl-Leibniz-Zentrum fuer Informatik.
-   Lasse Blaauwbroek. 2017. _On the Interaction Between Unrestricted Union and Intersection Types and Computational Effects_. Master's thesis . Technical University Eindhoven.
-   Gilad Bracha. 1992. _The Programming Language Jigsaw: Mixins, Modularity and Multiple Inheritance_. Ph.D. Dissertation . Dept. of Computer Science, University of Utah.
-   Gilad Bracha and William Cook. 1990. Mixin-based Inheritance. In _Proceedings of the European Conference on Object-oriented Programming and on Object-oriented Programming Systems, Languages, and Applications (OOPSLA/ECOOP’90)_.
-   Gilad Bracha, Peter Von Der Ahé, Vassili Bykov, Yaron Kashai, William Maddox, and Eliot Miranda. 2010. Modules as objects in newspeak. In _Proceedings of the European Conference on Object-oriented Programming_. Springer, 405–428.
-   K. Bruce, L. Cardelli, G. Castagna, The Hopkins Object Group, G. Leavens, and B. Pierce. 1996. On binary methods. _Theor. Pract. Obj. Syst._ 1, 3 (1996).
-   Kim Bruce, Martin Odersky, and Philip Wadler. 1998. A statically safe alternative to virtual types. In _Proceedings of the European Conference on Object-oriented Programming_.
-   R. M. Burstall, D. B. MacQueen, and D. T. Sannella. 1981. _HOPE: An Experimental Applicative Language_. Technical Report CSR-62-80. Computer Science Dept, University of Edinburgh.
-   Luca Cardelli. 1994. _Extensible Records in a Pure Calculus of Subtyping_. The MIT Press, Cambridge, MA.
-   Luca Cardelli and John Mitchell. 1991. Operations on records. _Math. Struct. Comput. Sci._ 1 (1991), 3–48.
-   Jacques Carette, Oleg Kiselyov, and Chung-chieh Shan. 2009. Finally tagless, partially evaluated: Tagless staged interpreters for simpler typed languages. _J. Funct. Prog._ 19, 05 (2009), 509–543. DOI: [https://doi.org/10.1017/S0956796809007205](https://doi.org/10.1017/S0956796809007205)
-   James Cheney and Ralf Hinze. 2002. A lightweight implementation of generics and dynamics. In _Proceedings of the ACM SIGPLAN Workshop on Haskell (Haskell’02)_, Manuel M. T. Chakravarty (Ed.). ACM, New York, NY, 90–104. DOI: [https://doi.org/10.1145/581690.581698](https://doi.org/10.1145/581690.581698)
-   Adam Chlipala. 2010. Ur: Statically-typed metaprogramming with type-level record computation. _ACM SIGPLAN Not._ 45, 6 (2010), 122–133.
-   Dave Clarke, Sophia Drossopoulou, James Noble, and Tobias Wrigstad. 2007. Tribe: A simple virtual class calculus. In _Proceedings of the 6th International Conference on Aspect-oriented Software Development_. 121–134.
-   William Cook and Jens Palsberg. 1989. A denotational semantics of inheritance and its correctness. In _Proceedings of the Conference on Object-oriented Programming Systems, Languages, and Applications (OOPSLA’89)_. ACM, New York, NY, 433–443. DOI: [https://doi.org/10.1145/74877.74922](https://doi.org/10.1145/74877.74922)
-   Stephen Dolan and Alan Mycroft. 2017. Polymorphism, subtyping, and type inference in MLsub. In _Proceedings of the 44th ACM SIGPLAN Symposium on Principles of Programming Languages (POPL’17)_. ACM, New York, NY, 60–72. DOI: [https://doi.org/10.1145/3009837.3009882](https://doi.org/10.1145/3009837.3009882)
-   Dominic Duggan and Constantinos Sourelis. 1996. Mixin modules. _ACM SIGPLAN Not._ 31, 6 (1996), 262–273.
-   Joshua Dunfield. 2014. Elaborating intersection and union types. _J. Funct. Prog._ 24, 2-3 (2014), 133–165. DOI: [https://doi.org/10.1017/S0956796813000270](https://doi.org/10.1017/S0956796813000270)
-   Erik Ernst. 1999. _gbeta-a Language with Virtual Attributes, Block Structure, and Propagating, Dynamic Inheritance_. Ph.D. Dissertation . University of Aarhus.
-   Erik Ernst. 2001. Family polymorphism. In _Proceedings of the 15th European Conference on Object-oriented Programming (ECOOP’01)_.
-   Erik Ernst. 2004. The expression problem, Scandinavian style. In _Proceedings of the On Mechanisms for Specialization_. 27.
-   Erik Ernst, Klaus Ostermann, and William R. Cook. 2006. A virtual class calculus. In _Proceedings of the 33rd ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages (POPL’06)_.
-   Matthew Flatt, Shriram Krishnamurthi, and Matthias Felleisen. 1998. Classes and mixins. In _Proceedings of the 25th ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages_. 171–183.
-   Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides. 1994. _Design Patterns: Elements of Reusable Object-oriented Software_. Addison-Wesley.
-   Jacques Garrigue. 2000. Code reuse through polymorphic variants. In _Proceedings of the Workshop on Foundations of Software Engineering_.
-   Jeremy Gibbons and Nicolas Wu. 2014. Folding domain-specific languages: Deep and shallow embeddings (functional Pearl). In _Proceedings of the 19th ACM SIGPLAN International Conference on Functional Programming (ICFP’14)_. 339–347. DOI: [https://doi.org/10.1145/2628136.2628138](https://doi.org/10.1145/2628136.2628138)
-   John V. Guttag and James J. Horning. 1978. The algebraic specification of abstract data types. _Acta Inform._ 10, 1 (1978), 27–52.
-   William Harrison and Harold Ossher. 1993. Subject-oriented programming: A critique of pure objects. In _Proceedings of the 8th Conference on Object-oriented Programming Systems, Languages, and Applications_. 411–428.
-   Ralf Hinze. 2004. An algebra of scans. In _Mathematics of Program Construction_. Springer Berlin Heidelberg, 186–210. DOI: [https://doi.org/10.1007/978-3-540-27764-4_11](https://doi.org/10.1007/978-3-540-27764-4_11)
-   Ralf Hinze. 2006. Generics for the masses. _J. Funct. Prog._ 16, 4-5 (2006), 451–483. DOI: [https://doi.org/10.1017/S0956796806006022](https://doi.org/10.1017/S0956796806006022)
-   Christian Hofer, Klaus Ostermann, Tillmann Rendel, and Adriaan Moors. 2008. Polymorphic embedding of DSLs. In _Proceedings of the 7th International Conference on Generative Programming and Component Engineering (GPCE’08)_.
-   Xuejing Huang and Bruno C. d. S. Oliveira. 2020. A type-directed operational semantics for a calculus with a merge operator. In _Proceedings of the 34th European Conference on Object-oriented Programming (ECOOP’20)_( Leibniz International Proceedings in Informatics (LIPIcs) , Vol. 166). DOI: [https://doi.org/10.4230/LIPIcs.ECOOP.2020.26](https://doi.org/10.4230/LIPIcs.ECOOP.2020.26)
-   Atsushi Igarashi and Benjamin C. Pierce. 1999. Foundations for virtual types. In _Proceedings of the European Conference on Object-oriented Programming_. Springer, 161–185.
-   Atsushi Igarashi, Chieri Saito, and Mirko Viroli. 2005. Lightweight family polymorphism. In _Proceedings of the Asian Symposium on Programming Languages and Systems_. Springer, 161–177.
-   Pablo Inostroza and Tijs van der Storm. 2015. Modular interpreters for the masses: Implicit context propagation using object algebras. In _Proceedings of the ACM SIGPLAN International Conference on Generative Programming: Concepts and Experiences_.
-   Mauro Jaskelioff. 2008. Monatron: An extensible monad transformer library. In _Proceedings of the Symposium on Implementation and Application of Functional Languages_. Springer, 233–248.
-   Paul Jolly, Sophia Drossopoulou, Christopher Anderson, and Klaus Ostermann. 2004. Simple dependent types: Concord. In _Proceedings of the ECOOP Workshop on Formal Techniques for Java Programs (FTfJP’04)_.
-   Uwe Kastens and William M. Waite. 1994. Modularity and reusability in attribute grammars. _Acta Inform._ 31, 7 (1994), 601–627.
-   Gregor Kiczales, John Lamping, Anurag Mendhekar, Chris Maeda, Cristina Lopes, Jean-Marc Loingtier, and John Irwin. 1997. Aspect-oriented programming. In _Proceedings of the European Conference on Object-oriented Programming_. Springer, 220–242.
-   Jorgen Lindskov Knudsen, Boris Magnusson, Mats Lofgren, and Ole L. Madsen. 1994. _Object Oriented Software Development Environments: The Mjolner Approach_. Prentice-Hall, Inc.
-   Donald E. Knuth. 1968. Semantics of context-free languages. _Math. Syst. Theor._ 2, 2 (1968), 127–145.
-   Donald E. Knuth. 1990. The genesis of attribute grammars. In _Proceedings of the International Conference WAGA on Attribute Grammars and their Applications_. 1–12.
-   Giovanni Lagorio, Marco Servetto, and Elena Zucca. 2009. Featherweight jigsaw: A minimal core calculus for modular composition of classes. In _Proceedings of the European Conference on Object-oriented Programming_. Springer, 244–268.
-   Joseph Lee, Jonathan Aldrich, Troy Shaw, and Alex Potanin. 2015. A theory of tagged objects. In _Proceedings of the 29th European Conference on Object-oriented Programming (ECOOP’15)_. Schloss Dagstuhl-Leibniz-Zentrum fuer Informatik.
-   Daan Leijen. 2005. Extensible records with scoped labels. _Trends Funct. Prog._ 6 (2005), 179–194.
-   Sheng Liang, Paul Hudak, and Mark Jones. 1995. Monad transformers and modular interpreters. In _Proceedings of the 22nd ACM SIGPLAN-SIGACT Symposium on Principles of Programming Languages_. 333–343.
-   David MacQueen. 1984. Modules for standard ML. In _Proceedings of the ACM Symposium on LISP and Functional Programming_. 198–207.
-   Ole Lehrmann Madsen and Birger Moller-Pedersen. 1989. Virtual classes: A powerful mechanism in object-oriented programming. In _Proceedings of the Conference on Object-oriented Programming Systems, Languages, and Applications_. 397–406.
-   Ole Lehrmann Madsen, Birger Møller-Pedersen, and Kristen Nygaard. 1993. _Object-oriented Programming in the BETA Programming Language_. Addison-Wesley.
-   Bertrand Meyer. 1988. _Object-oriented Software Construction_. Prentice Hall.
-   Keiko Nakata and Jacques Garrigue. 2006. Recursive modules for programming. _ACM SIGPLAN Not._ 41, 9 (2006), 74–86.
-   Nathaniel Nystrom, Stephen Chong, and Andrew C. Myers. 2004. Scalable extensibility via nested inheritance. In _Proceedings of the 19th ACM SIGPLAN Conference on Object-oriented Programming, Systems, Languages, and Applications_. 99–115.
-   Nathaniel Nystrom, Xin Qi, and Andrew C. Myers. 2006. J& nested intersection for scalable software composition. _ACM SIGPLAN Not._ 41, 10 (2006), 21–36.
-   Martin Odersky, Philippe Altherr, Vincent Cremet, Burak Emir, Sebastian Maneth, Stéphane Micheloud, Nikolay Mihaylov, Michel Schinz, Erik Stenman, and Matthias Zenger. 2004. _An Overview of the Scala Programming Language_. Technical Report . EPFL Lausanne, Switzerland.
-   Martin Odersky and Matthias Zenger. 2005. Scalable component abstractions. In _Proceedings of the 20th ACM SIGPLAN Conference on Object-oriented Programming, Systems, Languages, and Applications (OOPSLA’05)_.
-   Bruno C. d. S. Oliveira and William R. Cook. 2012. Extensibility for the masses: Practical extensibility with object algebras. In _Proceedings of the 26th European Conference on Object-oriented Programming (ECOOP’12)_. DOI: [https://doi.org/10.1007/978-3-642-31057-7_2](https://doi.org/10.1007/978-3-642-31057-7_2)
-   Bruno C. d. S. Oliveira, Ralf Hinze, and Andres Löh. 2006. Extensible and modular generics for the masses. In _Trends in Functional Programming_. 199–216.
-   Bruno C. d. S. Oliveira, Zhiyuan Shi, and João Alpuim. 2016. Disjoint intersection types. In _Proceedings of the 21st ACM SIGPLAN International Conference on Functional Programming_. 364–377.
-   Bruno C. d. S. Oliveira, Tijs van der Storm, Alex Loh, and William R. Cook. 2013. Feature-oriented programming with object algebras. In _Proceedings of the 27th European Conference on Object-oriented Programming_. DOI: [https://doi.org/10.1007/978-3-642-39038-8_2](https://doi.org/10.1007/978-3-642-39038-8_2)
-   Benjamin C. Pierce and David N. Turner. 2000. Local type inference. _ACM Trans. Prog. Lang. Syst._ 22, 1 ( Jan. 2000), 44.
-   Klaus Pohl, Günter Böckle, and Frank J. van Der Linden. 2005. _Software Product Line Engineering: Foundations, Principles and Techniques_. Springer Science & Business Media.
-   Erik Poll. 1997. System F with width-subtyping and record updating. In _Proceedings of the 3rd International Symposium on Theoretical Aspects of Computer Software (TACS’97)_. Springer-Verlag, Berlin.
-   Christian Prehofer. 1997. Feature-oriented programming: A fresh look at objects. In _Proceedings of the European Conference on Object-oriented Programming_. Springer, 419–443.
-   Tillmann Rendel, Jonathan Immanuel Brachthäuser, and Klaus Ostermann. 2014. From object algebras to attribute grammars. In _Proceedings of the ACM International Conference on Object-oriented Programming Systems, Languages, and Applications_.
-   Claudio V. Russo. 2000. First-class structures for Standard ML. In _Proceedings of the European Symposium on Programming_. Springer, 336–350.
-   Claudio V. Russo. 2001. Recursive structures for Standard ML. In _Proceedings of the 6th ACM SIGPLAN International Conference on Functional Programming_. 50–61.
-   Nathanael Schärli, Stéphane Ducasse, Oscar Nierstrasz, and Andrew P. Black. 2003. Traits: Composable units of behaviour. In _Proceedings of the European Conference on Object-oriented Programming_. Springer, 248–274.
-   Tom Schrijvers and Bruno C. d. S. Oliveira. 2011. Monads, zippers and views: Virtualizing the monad stack. In _Proceedings of the 16th ACM SIGPLAN International Conference on Functional Programming_. 32–44.
-   M. Sulzmann, M. M. T. Chakravarty, S. L. Peyton-Jones, and K. Donnelly. 2007. System F with type equality coercions. In _Proceedings of the ACM SIGPLAN Workshop on Types in Language in Design and Implementation_.
-   Wouter Swierstra. 2008. Data Types à la Carte. _J. Funct. Prog._ 18, 04 (2008), 423–436. DOI: [https://doi.org/10.1017/S0956796808006758](https://doi.org/10.1017/S0956796808006758)
-   Asumu Takikawa, T. Stephen Strickland, Christos Dimoulas, Sam Tobin-Hochstadt, and Matthias Felleisen. 2012. Gradual typing for first-class classes. In _Proceedings of the ACM International Conference on Object-oriented Programming Systems, Languages, and Applications_. 793–810.
-   Peri Tarr, Harold Ossher, William Harrison, and Stanley M. Sutton. 1999. N degrees of separation: Multi-dimensional separation of concerns. In _Proceedings of the International Conference on Software Engineering_. IEEE, 107–119.
-   Mads Torgersen. 2004. The expression problem revisited. In _Proceedings of the European Conference on Object-oriented Programming (ECOOP’04)_.
-   Birthe van den Berg. 2020. ICFP: G: Type Inference for Disjoint Intersection Types. [https://people.cs.kuleuven.be/birthe.vandenberg/cite_SRC.html](https://people.cs.kuleuven.be/%20birthe.vandenberg/cite_SRC.html).
-   Philip Wadler. 1992. The essence of functional programming. In_Proceedings of the ACM SIGPLAN Symposium on Principles of Programming Languages_. 1–14.
-   Philip Wadler. 1998. The Expression Problem. ( Nov. 1998). Note to Java Genericity mailing list. [https://homepages.inf.ed.ac.uk/wadler/papers/expression/expression.txt](https://homepages.inf.ed.ac.uk/wadler/papers/expression/expression.txt).
-   Yanlin Wang and Bruno C. d. S. Oliveira. 2016. The expression problem, trivially! In _Proceedings of the 15th International Conference on Modularity (MODULARITY’16)_. 37–41. DOI: [https://doi.org/10.1145/2889443.2889448](https://doi.org/10.1145/2889443.2889448)
-   Mathhias Zenger and Martin Odersky. 2005. Independently extensible solutions to the expression problem. In _Proceedings of the Foundations of Object-oriented Languages Conference (FOOL’05)_.
-   Weixin Zhang and Bruno C. d. S. Oliveira. 2017. EVF: An extensible and expressive visitor framework for programming language reuse. In _Proceedings of the 31st European Conference on Object-oriented Programming_. DOI: [https://doi.org/10.4230/LIPIcs.ECOOP.2017.29](https://doi.org/10.4230/LIPIcs.ECOOP.2017.29)
-   Weixin Zhang and Bruno C. d. S. Oliveira. 2019. Shallow EDSLs and object-oriented programming: Beyond simple compositionality. _Art, Sci., Eng. Prog._ 3, 3 (2019), 1–25.
-   Yizhou Zhang and Andrew C. Myers. 2017. Familia: Unifying interfaces, type classes, and family polymorphism. _Proc. ACM Prog. Lang._ 1, OOPSLA (2017), 1–31.

## Footnote

This work has been sponsored by the Hong Kong Research Grant Council project numbers 17210617 and 17209519.

Authors’ addresses: W. Zhang, University of Bristol, Bristol, United Kingdom, The University of Hong Kong, Hong Kong, China; email: [wxzhang2@cs.hku.hk](mailto:wxzhang2@cs.hku.hk); Y. Sun and B. C. d. S. Oliveira, The University of Hong Kong, Hong Kong, China; emails: [yzsun@cs.hku.hk](mailto:yzsun@cs.hku.hk), [bruno@cs.hku.hk](mailto:bruno@cs.hku.hk).

Permission to make digital or hard copies of all or part of this work for personal or classroom use is granted without fee provided that copies are not made or distributed for profit or commercial advantage and that copies bear this notice and the full citation on the first page. Copyrights for components of this work owned by others than ACM must be honored. Abstracting with credit is permitted. To copy otherwise, or republish, to post on servers or to redistribute to lists, requires prior specific permission and/or a fee. Request permissions from [permissions@acm.org](mailto:permissions@acm.org).

©2021 Association for Computing Machinery.  
0164-0925/2021/08-ART9 $15.00  
DOI: [https://doi.org/10.1145/3460228](https://doi.org/10.1145/3460228)

Publication History: Received October 2020; accepted April 2021