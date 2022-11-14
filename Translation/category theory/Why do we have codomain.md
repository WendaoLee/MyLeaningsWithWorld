# [Why do we have codomain?](https://math.stackexchange.com/questions/1922674/why-do-we-have-codomain)

[Ask Question](https://math.stackexchange.com/questions/ask)

Asked 6 years, 1 month ago

Modified [4 months ago](https://math.stackexchange.com/questions/1922674/why-do-we-have-codomain?lastactivity "2022-06-07 09:53:35Z")

Viewed 703 times

2

[](https://math.stackexchange.com/posts/1922674/timeline)

This really ties to the [question](https://math.stackexchange.com/q/1922242/347062) I asked last night about f:A→Bf:A→B, but I still don't understand some things. I'm only in AP Calculus BC, and we've never discussed this codomain set BB. Why does it exist? If the range of ff is {f(a)∣a∈A}{f(a)∣a∈A}, then why don't we use the range where BB is? The way I see it, one could simply say U=R∪{a+bi∣a∈R∧b∈R∧i2=−1}U=R∪{a+bi∣a∈R∧b∈R∧i2=−1} and set B=UB=U and never have to worry about it again, so how do you know what to set the codomain equal to?


asked Sep 11, 2016 at 14:49

[

![gen-ℤ ready to perish's user avatar](https://i.stack.imgur.com/S5jqZ.jpg?s=64&g=1)

](https://math.stackexchange.com/users/347062/gen-%e2%84%a4-ready-to-perish)

[gen-ℤ ready to perish](https://math.stackexchange.com/users/347062/gen-%e2%84%a4-ready-to-perish)

6,45233 gold badges2626 silver badges4242 bronze badges

[Add a comment](https://math.stackexchange.com/questions/1922674/why-do-we-have-codomain# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 2 Answers

[](https://math.stackexchange.com/posts/1923539/timeline)

**Short Answer** It is convenient to know where the output lies. I can tell whether a function is real-valued or complex-valued with an appropriate use of the codomain.

**Long Answer** Functions exist in many contexts; not just Calculus. In Linear Algebra, the functions of interest are linear transformations. In Algebra, the functions of interest are (group/ring) homomorphisms. In Topology, the functions of interest are continuous functions. In Calculus, (one of the) the functions of interest are real-valued functions from RR to RR. The "space" RR you are working in doesn't change. However in other contexts the "space" can change.

For example, in Linear Algebra the "spaces" are vector spaces (typically but not always finite-dimensional). There's not one vector space that's studied always, in fact there are infinitely many of them such as RnRn (n∈Nn∈N). In Algebra there are groups, rings, fields, etc. In Topology there are what's called topological spaces, which unlike Calculus does not have one space that's studied always. So functions from one space to another are best defined via a domain and codomain. You are right that in Calculus it seems that all codomains could theoretically be CC the complex numbers, but now that you know that spaces can change, the codomain serves as a means of telling you if the space changed or not.