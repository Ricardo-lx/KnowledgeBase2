arising from the ambiguity check for ‘it’
from the context
(Num (a1 -> a), Num a1, Num a)
bound by the inferred type for ‘it’:
(Num (a1 -> a), Num a1, Num a) => a -> a
at <interactive>:9:1-13
The type variable ‘a0’ is ambiguous
When checking that ‘it’
has the inferred type ‘forall a a1.
(Num (a1 -> a), Num a1, Num a) =>
a -> a’
Probable cause:
the inferred type is ambiguous
The type error Could not deduce (Num (a0 -> a)) is because
you can’t use Num a => a values as if they were functions. To
the computer, it looks like you’re trying to use 3 as a function
and apply 3 to 1. Here the itreferred to is 3 1which it thinks
is3applied to 1as if3were a function.1
1In GHCi error messages, itrefers to the last expression you entered.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 342
Exercises: Grab Bag
Note the following exercises are from source code files, not
written for use directly in the REPL. Of course, you can change
them to test directly in the REPL if you prefer.
1.Which (two or more) of the following are equivalent?
a)mThx y z=x*y*z
b)mThx y=\z->x*y*z
c)mThx=\y->\z->x*y*z
d)mTh=\x->\y->\z->x*y*z
2.The type of mTh(above) is Num a => a -> a -> a -> a .
Which is the type of mTh 3?
a)Integer -> Integer -> Integer
b)Num a => a -> a -> a -> a
c)Num a => a -> a
d)Num a => a -> a -> a
3.Next, we’ll practice writing anonymous lambda syntax.
For example, one could rewrite:
addOnex=x+1
Into:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 343
addOne=\x->x+1
Try to make it so it can still be loaded as a top-level def-
inition by GHCi. This will make it easier to validate your
answers.
a)Rewrite the ffunction in the where clause.
addOneIfOdd n= caseodd nof
True->f n
False->n
wheref n=n+1
b)Rewrite the following to use anonymous lambda syn-
tax:
addFive x y=(ifx>ythenyelsex)+5
c)Rewrite the following so that it doesn’t use anony-
mous lambda syntax:
mflipf=\x->\y->f y x
The utility of lambda syntax
You’re going to see this anonymous syntax a lot as we proceed
through the book, but right now it may not seem to be that
useful — it’s just another way to write functions.
You most often use this syntax when you’re passing a func-
tion in as an argument to a higher-order function (more on
this soon!) and that’s the only place in your program where

CHAPTER 7. MORE FUNCTIONAL PATTERNS 344
that particular function will be used. If you’re never going to
call it, then it doesn’t need to be given a name.
We won’t go into a lot of detail about this yet, but named
entities and anonymous entities evaluate a bit diﬀerently in
Haskell, and that can be one reason to use an anonymous
function in some cases.
7.4 Pattern matching
Pattern matching is an integral and ubiquitous feature of
Haskell — so integral and ubiquitous that we’ve been using it
throughout the book without saying anything about it. Once
you start, you can’t stop.
Pattern matching is a way of matching values against pat-
terns and, where appropriate, binding variables to successful
matches. It is worth noting here that patterns can include things
as diverse as undefined variables, numeric literals, and list syn-
tax. As we will see, pattern matching matches on any and all
data constructors.
Pattern matching allows you to expose data and dispatch
diﬀerent behaviors based on that data in your function defini-
tions by deconstructing values to expose their inner workings.
There is a reason we describe values as “data constructors ”, al-
though we haven’t explored that much yet. Pattern matching
also allows us to write functions that can decide between two
or more possibilities based on which value it matches.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 345
Patterns are matched against values, or data constructors,
nottypes. Matching a pattern may fail, proceeding to the next
available pattern to match or succeed. When a match suc-
ceeds, the variables exposed in the pattern are bound. Pattern
matching proceeds from left to right and outside to inside.
We can pattern match on numbers. In the following exam-
ple, when the Integer argument to the function equals 2, this
returns True, otherwise, False:
isItTwo ::Integer ->Bool
isItTwo 2=True
isItTwo _ =False
You can enter the same function directly into GHCi using
the:{and:}block syntax, enter :}and “return” to end the
block.
Prelude> :{
*Main| let isItTwo :: Integer -> Bool
*Main| isItTwo 2 = True
*Main| isItTwo _ = False
*Main| :}
Note the use of the underscore _after the match against the
value2. This is a means of defining a universal pattern that
never fails to match, a sort of “anything else” case.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 346
Prelude> isItTwo 2
True
Prelude> isItTwo 3
False
Handling all the cases
The order of pattern matches matters! The following version
of the function will always return Falsebecause it will match
the “anything else” case first — and match it to everything —
so nothing will get through that to match with the pattern you
do want to match:
isItTwo ::Integer ->Bool
isItTwo _ =False
isItTwo 2=True
<interactive>:9:33: Warning:
Pattern match(es) are overlapped
In an equation for ‘isItTwo’:
isItTwo 2 = ...
Prelude> isItTwo 2
False
Prelude> isItTwo 3
False

CHAPTER 7. MORE FUNCTIONAL PATTERNS 347
Try to order your patterns from most specific to least spe-
cific, particularly as it concerns the use of _to unconditionally
match any value. Unless you get fancy, you should be able
to trust GHC’s pattern match overlap warning and should
triple-check your code when it complains.
What happens if we forget to match a case in our pattern?
isItTwo ::Integer ->Bool
isItTwo 2=True
Notice that now our function can only pattern match on the
value 2. This is an incomplete pattern match because it can’t
match any other data. Incomplete pattern matches applied to
data they don’t handle will return bottom , a non-value used to
denote that the program cannot return a value or result. This
will throw an exception, which if unhandled, will make your
program fail:
Prelude> isItTwo 2
True
Prelude> isItTwo 3
*** Exception: :50:33-48:
Non-exhaustive patterns
in function isItTwo
We’re going to get well acquainted with the idea of bottom
in upcoming chapters. For now, it’s enough to know that this
is what you get when you don’t handle all the possible data.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 348
Fortunately, there’s a way to know at compile time when
your pattern matches are non-exhaustive and don’t handle
every case:
Prelude> :set -Wall
Prelude> :{
*Main| let isItTwo :: Integer -> Bool
*Main| isItTwo 2 = True
*Main| :}
<interactive>:28:5: Warning:
This binding for ‘isItTwo’ shadows
the existing binding
defined at <interactive>:20:5
<interactive>:28:5: Warning:
Pattern match(es) are non-exhaustive
In an equation for ‘isItTwo’:
Patterns not matched:
#x with #x `notElem` [2#]
By turning on all warnings with -Wall, we’re now told ahead
of time that we’ve made a mistake. Do notignore the warnings
GHC provides for you!

CHAPTER 7. MORE FUNCTIONAL PATTERNS 349
Pattern matching against data constructors
Pattern matching serves a couple of purposes. It enables us to
vary what our functions do given diﬀerent inputs. It also allows
us to unpack and expose the contents of our data. The values
TrueandFalsedon’t have any other data to expose, but some
data constructors have parameters, and pattern matching can
let us expose and make use of the data in their arguments.
The next example uses newtype which is a special case of data
declarations. newtype is diﬀerent in that it permits only one
constructor and only one field. We will talk about newtype more
later. For now, we want to focus on how pattern matching can
be used to expose the contents of data and specify behavior
based on that data:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 350
-- registeredUser1.hs
moduleRegisteredUser where
newtype Username =
Username String
newtype AccountNumber =
AccountNumber Integer
dataUser=
UnregisteredUser
|RegisteredUser Username AccountNumber
With the type User, we can use pattern matching to ac-
complish two things. First, Useris a sum with two construc-
tors,UnregisteredUser andRegisteredUser . We can use pattern
matching to dispatch our function diﬀerently depending on
which value we get. Then with the RegisteredUser construc-
tor we see that it is a product of two newtype s,Username and
AccountNumber . We can use pattern matching to break down
not only RegisteredUser ’s contents, but also that of the newtype s
if all the constructors are in scope. Let’s write a function to
pretty-print Uservalues:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 351
-- registeredUser2.hs
moduleRegisteredUser where
newtype Username =
Username String
newtype AccountNumber =
AccountNumber Integer
dataUser=
UnregisteredUser
|RegisteredUser Username AccountNumber
printUser ::User->IO()
printUser UnregisteredUser =
putStrLn "UnregisteredUser"
printUser (RegisteredUser
(Username name)
(AccountNumber acctNum)) =
putStrLn $name++" "++show acctNum
Note that you can continue the pattern on the next line if it
gets too long. Next, let’s load this into the REPL and look at
the types:
Prelude> :l code/registeredUser2.hs

CHAPTER 7. MORE FUNCTIONAL PATTERNS 352
...
Prelude> :t RegisteredUser
RegisteredUser :: Username
-> AccountNumber
-> User
Prelude> :t Username
Username :: String -> Username
Prelude> :t AccountNumber
AccountNumber :: Integer -> AccountNumber
Notice how the type of RegisteredUser is a function that con-
structs a Userout of two arguments: Username andAccountNumber .
This is what we mean when we refer to a value as a “data con-
structor.”
Now, let’s use our functions. The argument names are te-
dious to type in, but they were chosen to ensure clarity. Passing
the function an UnregisteredUser returns the expected value:
Prelude> printUser UnregisteredUser
UnregisteredUser
The following, though, asks it to match on data constructor
RegisteredUser and allows us to construct a Userout of the String
“callen” and the Integer 10456:
Prelude> let myUser = Username "callen"
Prelude> let myAcct = AccountNumber 10456

CHAPTER 7. MORE FUNCTIONAL PATTERNS 353
Prelude> :{
*Main| let rUser =
*Main| RegisteredUser myUser myAcct
*Main| :}
Prelude> printUser rUser
callen 10456
Through the use of pattern matching, we were able to un-
pack the RegisteredUser value of the Usertype and vary behav-
ior over the diﬀerent constructors of types.
This idea of unpacking and dispatching on data is impor-
tant, so let us examine another example. First, we’re going to
write a couple of new datatypes. Writing your own datatypes
won’t be fully explained until a later chapter, but most of the
structure here should be familiar already. We have a sum type
calledWherePenguinsLive :
dataWherePenguinsLive =
Galapagos
|Antarctica
|Australia
|SouthAfrica
|SouthAmerica
deriving (Eq,Show)
And a product type called Penguin . We haven’t given product
types much attention yet, but for now you can think of Penguin

CHAPTER 7. MORE FUNCTIONAL PATTERNS 354
as a type with only one value, Peng, and that value is a sort of
box that contains a WherePenguinsLive value:
dataPenguin =
PengWherePenguinsLive
deriving (Eq,Show)
Given these datatypes, we will write a couple functions for
processing the data:
-- is it South Africa? If so, return True
isSouthAfrica ::WherePenguinsLive ->Bool
isSouthAfrica SouthAfrica =True
isSouthAfrica Galapagos =False
isSouthAfrica Antarctica =False
isSouthAfrica Australia =False
isSouthAfrica SouthAmerica =False
But that is redundant. We can use _to indicate an uncondi-
tional match on a value we don’t care about. The following is
better (more concise, easier to read) and does the same thing:
isSouthAfrica' ::WherePenguinsLive ->Bool
isSouthAfrica' SouthAfrica =True
isSouthAfrica' _ = False
We can also use pattern matching to unpack Penguin values
to get at the WherePenguinsLive value it contains:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 355
gimmeWhereTheyLive ::Penguin
->WherePenguinsLive
gimmeWhereTheyLive (Pengwhereitlives) =
whereitlives
Try using the gimmeWhereTheyLive function on some test data.
When you enter the name of the penguin (note the lowercase),
it will unpack the Pengvalue to return the WherePenguinsLive
that’s inside:
humboldt =PengSouthAmerica
gentoo=PengAntarctica
macaroni =PengAntarctica
little=PengAustralia
galapagos =PengGalapagos
Now a more elaborate example. We’ll expose the contents
ofPengand match on what WherePenguinLives value we care
about in one pattern match:
galapagosPenguin ::Penguin ->Bool
galapagosPenguin (PengGalapagos )=True
galapagosPenguin _ = False
antarcticPenguin ::Penguin ->Bool
antarcticPenguin (PengAntarctica )=True
antarcticPenguin _ = False

CHAPTER 7. MORE FUNCTIONAL PATTERNS 356
In this final function, the (||)operator is an orfunction that
will return Trueif either value is True:
antarcticOrGalapagos ::Penguin ->Bool
antarcticOrGalapagos p=
(galapagosPenguin p)
||(antarcticPenguin p)
Note that we’re using pattern matching to accomplish two
things here. We’re using it to unpack the Penguin datatype.
We’re also specifying which WherePenguinsLive value we want
to match on.
Pattern matching tuples
You can also use pattern matching rather than functions for
operating on the contents of tuples. Remember this example
from Chapter 4?
f::(a, b)->(c, d)->((b, d), (a, c))
f=undefined
When you did that exercise, you may have written it like
this:
f::(a, b)->(c, d)->((b, d), (a, c))
fx y=((snd x, snd y), (fst x, fst y))

CHAPTER 7. MORE FUNCTIONAL PATTERNS 357
But we can use pattern matching on tuples to make a some-
what cleaner version of it:
f::(a, b)->(c, d)->((b, d), (a, c))
f(a, b) (c, d) =((b, d), (a, c))
One nice thing about this is that the tuple syntax allows the
function to look a great deal like its type. Let’s look at more
examples of pattern matching on tuples. Note that the second
example below is nota pattern match but the others are:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 358
-- matchingTuples1.hs
moduleTupleFunctions where
-- These have to be the same type because
-- (+) is a -> a -> a
addEmUp2 ::Numa=>(a, a)->a
addEmUp2 (x, y)=x+y
-- addEmUp2 could also be written like so
addEmUp2Alt ::Numa=>(a, a)->a
addEmUp2Alt tup=(fst tup) +(snd tup)
fst3::(a, b, c) ->a
fst3(x,_,_)=x
third3::(a, b, c) ->c
third3(_,_, x)=x
Prelude> :l code/matchingTuples1.hs
[1 of 1] Compiling TupleFunctions
Ok, modules loaded: TupleFunctions.
Now we’re going to use GHCi’s :browse to see a list of the
type signatures and functions we loaded from the module
TupleFunctions :

CHAPTER 7. MORE FUNCTIONAL PATTERNS 359
Prelude> :browse TupleFunctions
addEmUp2 :: Num a => (a, a) -> a
addEmUp2Alt :: Num a => (a, a) -> a
fst3 :: (a, b, c) -> a
third3 :: (a, b, c) -> c
Prelude> addEmUp2 (10, 20)
30
Prelude> addEmUp2Alt (10, 20)
30
Prelude> fst3 ("blah", 2, [])
"blah"
Prelude> third3 ("blah", 2, [])
[]
Sweet. Let’s do some exercises. Pausing to exercise keeps
the muscles flexible, even the mental ones.
Exercises: Variety Pack
1.Given the following declarations
k(x, y)=x
k1=k ((4-1),10)
k2=k ("three", (1+2))
k3=k (3,True)

CHAPTER 7. MORE FUNCTIONAL PATTERNS 360
a)What is the type of k?
b)What is the type of k2? Is it the same type as k1ork3?
c)Ofk1, k2, k3 , which will return the number 3 as the
result?
2.Fill in the definition of the following function:
-- Remember: Tuples have the
same syntax for their
-- type constructors and
-- their data constructors.
f::(a, b, c)
->(d, e, f)
->((a, d), (c, f))
f=undefined
7.5 Case expressions
Caseexpressionsareaway, similarinsomerespectsto if-then-else ,
of making a function return a diﬀerent result based on diﬀer-
ent inputs. You can use case expressions with any datatype that
has visible data constructors. When we consider the datatype
Bool:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 361
dataBool=False|True
-- [1] [2] [3]
1.Type constructor, we only use this in type signatures, not
in term-level code like case expressions.
2.Data constructor for the value of Boolnamed False— we
can match on this.
3.Data constructor for the value of Boolnamed True— we
can match on this as well.
Any time we case match or pattern match on a sum type
likeBool, we should define how we handle each constructor
or provide a default that matches all of them. In fact, we must
handle both cases or use a function that handles both or we
will have written a partial function that can throw an error
at runtime. There is rarely a good reason to do this: write
functions that handle all possible inputs!
Let’s start by looking at an if-then-else expression that we
saw in a previous chapter:
ifx+1==1then"AWESOME" else"wut"
We can rewrite this as a case expression, matching on the
constructors of Bool:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 362
funcZx=
casex+1==1of
True->"AWESOME"
False->"wut"
Note that while the syntax is considerably diﬀerent here,
the results will be the same. Be sure to load it in the REPL and
try it out.
We could also write a case expression to tell us whether or
not something is a palindrome:
palxs=
casexs==reverse xs of
True->"yes"
False->"no"
The above can also be written with a whereclause in cases
where you might need to reuse the 𝑦:
pal'xs=
caseyof
True->"yes"
False->"no"
wherey=xs==reverse xs
In either case, the function will first check if the input string
is equal to the reverse of it. If that returns True, then the string

CHAPTER 7. MORE FUNCTIONAL PATTERNS 363
is a palindrome, so your function says, “yes.” If not, then it’s
not.
Here is one more example, also matching on the data con-
structors from Bool, and you can compare its syntax to the
if-then-else version we’ve seen before:
-- greetIfCool3.hs
moduleGreetIfCool3 where
greetIfCool ::String->IO()
greetIfCool coolness =
casecoolof
True->
putStrLn "eyyyyy. What's shakin'?"
False->
putStrLn "pshhhh."
wherecool=
coolness =="downright frosty yo"
So far, the case expressions we’ve looked at rely on a straight-
forward pattern match with TrueandFalseexplicitly. In an
upcoming section, we’ll look at another way to write a case
expression.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 364
Exercises: Case Practice
We’re going to practice using case expressions by rewriting
functions. Some of these functions you’ve seen in previous
chapters (and some you’ll see later using diﬀerent syntax yet
again!), but you’ll be writing new versions now. Please note
these are all written as they would be in source code files, and
we recommend you write your answers in source files and
then load into GHCi to check, rather than trying to do them
directly into the REPL.
First, rewrite if-then-else expressions into case expressions.
1.The following should return xwhenxis greater than y.
functionC x y= if(x>y)thenxelsey
2.The following will add 2 to even numbers and otherwise
simply return the input value.
ifEvenAdd2 n= ifeven nthen(n+2)elsen
The next exercise doesn’t have all the cases covered. See
if you can fix it.
3.The following compares a value, x, to zero and returns an
indicator for whether xis a postive number or negative
number. But what if xis 0? You may need to play with
thecompare function a bit to find what to do.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 365
numsx=
casecompare x 0of
LT-> -1
GT->1
7.6 Higher-order functions
Higher-order functions (HOFs) are functions that accept func-
tions as arguments. Functions are values — why couldn’t they
be passed around like any other values? This is an important
component of functional programming and gives us a way to
combine functions efficiently.
Let’s examine a standard higher-order function, flip:
Prelude> :t flip
flip :: (a -> b -> c) -> b -> a -> c
-- using (-) as our (a -> b -> c)
Prelude> (-) 10 1
9
Prelude> let fSub = flip (-)
Prelude> fSub 10 1
-9
Prelude> fSub 5 10
5

CHAPTER 7. MORE FUNCTIONAL PATTERNS 366
The first parameter of flipis a function, such as (-), that
itself has two parameters. flipflips the order of the arguments.
We can implement fliplike this, using the variable 𝑓to
represent the function (a -> b -> c) :
flip::(a->b->c)->b->a->c
flipf x y=f y x
Alternately, it could’ve been written as:
myFlip::(a->b->c)->b->a->c
myFlipf=\x y->f y x
There’s no diﬀerence in what flipandmyFlip do: one de-
clares parameters in the function definition, and the other
declares them instead in the anonymous function value being
returned. But what makes flip a higher-order function? Well,
it’s this:
flip::(a->b->c)->b->a->c
[1]
flipf x y=f y x
[2] [3]
1.When we want to express a function argument within a
function type, we must use parentheses to nest it.
2.The argument 𝑓is the function a -> b -> c .

CHAPTER 7. MORE FUNCTIONAL PATTERNS 367
3.We apply 𝑓to𝑥and𝑦butflipwill flip the order of ap-
plication and apply 𝑓to𝑦and then 𝑥instead of the usual
order.
To better understand how HOFs work syntactically, it’s
worth remembering how parentheses associate in type signa-
tures.
Let’s look at the type of the following function:
returnLast ::a->b->c->d->d
returnLast _ _ _d=d
If we explicitly parenthesize returnLast , it must match the
associativity of ->, which is right-associative. The following
parenthesization works fine. Note that this makes the default
currying explicit:
returnLast' ::a->(b->(c->(d->d)))
returnLast' _ _ _d=d
However, this will not work. This is not how ->associates:
returnBroke ::(((a->b)->c)->d)->d
returnBroke _ _ _d=d
If you attempt to load returnBroke , you’ll get a type error.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 368
Couldn't match expected type
‘t0 -> t1 -> t2 -> t2’
with actual type ‘d’
‘d’ is a rigid type variable bound by
the type signature for
returnBroke :: (((a -> b) -> c) -> d) -> d
Relevant bindings include
returnBroke :: (((a -> b) -> c) -> d) -> d
The equation(s) for ‘returnBroke’
have four arguments,
but its type ‘(((a -> b) -> c) -> d) -> d’
has only one
This type error is telling us that the type of returnBroke only
specifies one argument that has the type ((a -> b) -> c) -> d ,
yet our function definition seems to expect fourarguments.
The type signature of returnBroke specifies a single function as
the sole argument to returnBroke .2
Wecanhave a type that is parenthesized in that fashion as
long as we want to do something diﬀerent than what returnLast
does:
2Fun fact: returnBroke is an impossible function.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 369
returnAfterApply ::(a->b)->a->c->b
returnAfterApply f a c=f a
What we’re doing here is parenthesizing to the leftso that
we can refer to a separate function, with its own parameters
and result, as an argument to our top level function. Here the
(a -> b) is the𝑓argument we use to produce a value of type 𝑏
from a value of type 𝑎.
One reason we want HOFs is to manipulate how functions
are applied to arguments. To understand another reason, let’s
revisit the compare function from the Ordtypeclass:
Prelude> :t compare
compare :: Ord a => a -> a -> Ordering
Prelude> :info Ordering
data Ordering = LT | EQ | GT
Prelude> compare 10 9
GT
Prelude> compare 9 9
EQ
Prelude> compare 9 10
LT
Now we’ll write a function that makes use of this:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 370
dataEmployee =Coder
|Manager
|Veep
|CEO
deriving (Eq,Ord,Show)
reportBoss ::Employee ->Employee ->IO()
reportBoss e e'=
putStrLn $show e++
" is the boss of " ++
show e'
employeeRank ::Employee
->Employee
->IO()
employeeRank e e'=
casecompare e e' of
GT->reportBoss e e'
-- [ 1 ]
EQ->putStrLn "Neither employee \
\is the boss"
-- [ 2 ]
LT->(flip reportBoss) e e'
-- [ 3 ]

CHAPTER 7. MORE FUNCTIONAL PATTERNS 371
Thecasein theemployeeRank function is a case expression.
This function says: case expression
1.In the case of comparing 𝑒and𝑒′and finding 𝑒is greater
than𝑒′, return reportBoss e e' .
2.In the case of finding them equal, return the string “Nei-
ther employee is the boss.”
3.Inthecaseoffinding 𝑒lessthan 𝑒′, flipthefunction reportBoss .
This could also have been written reportBoss e' e .
Thecompare function uses the behavior of the Ordinstance
defined for a given type in order to compare them. In this
case, our data declaration lists them in order from Coderin
the lowest rank and CEOin the top rank, so compare will use that
ordering to evaluate the result of the function.
If we load this up and try it out:
Prelude> employeeRank Veep CEO
CEO is the boss of Veep
That’s probably true in most companies! Being industrious
programmers, we naturally want to refactor this a bit to be
more flexible — notice how we change the type of employeeRank :

CHAPTER 7. MORE FUNCTIONAL PATTERNS 372
dataEmployee =Coder
|Manager
|Veep
|CEO
deriving (Eq,Ord,Show)
reportBoss ::Employee ->Employee ->IO()
reportBoss e e'=
putStrLn $show e++
" is the boss of " ++
show e'
employeeRank ::(Employee
->Employee
->Ordering )
->Employee
->Employee
->IO()
employeeRank f e e'=
casef e e'of
GT->reportBoss e e'
EQ->putStrLn "Neither employee \
\is the boss"
LT->(flip reportBoss) e e'

CHAPTER 7. MORE FUNCTIONAL PATTERNS 373
Now our employeeRank function will accept a function argu-
ment with the type Employee -> Employee -> Ordering , which we
named 𝑓, in the place where we had compare before. You’ll no-
tice we have the same case expressions here again. We can get
the same behavior we had last time by passing it compare as the
function argument:
Prelude> employeeRank compare Veep CEO
CEO is the boss of Veep
Prelude> employeeRank compare CEO Veep
CEO is the boss of Veep
But since we’re clever hackers, we can subvert the hierarchy
with a comparison function that does something a bit diﬀerent
with the following code:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 374
dataEmployee =Coder
|Manager
|Veep
|CEO
deriving (Eq,Ord,Show)
reportBoss ::Employee ->Employee ->IO()
reportBoss e e'=
putStrLn $show e++
" is the boss of " ++
show e'
codersRuleCEOsDrool ::Employee
->Employee
->Ordering
codersRuleCEOsDrool CoderCoder=EQ
codersRuleCEOsDrool Coder_ =GT
codersRuleCEOsDrool _Coder=LT
codersRuleCEOsDrool e e'=
compare e e'
employeeRank ::(Employee
->Employee
->Ordering )
->Employee
->Employee
->IO()
employeeRank f e e'=
casef e e'of
GT->reportBoss e e'
EQ->putStrLn "Neither employee \
\is the boss"
LT->(flip reportBoss) e e'

CHAPTER 7. MORE FUNCTIONAL PATTERNS 375
Herewe’vecreateda newfunction that changesthe behavior
of the normal compare function by pattern matching on our
data constructor, Coder. In a case where Coderis the first value
(and the second value is anything — note the underscore used
as a catchall), the result will be GTor greater than. In a case
whereCoderis the second value passed, this function will return
LT, orless than . In any case where Coderis not one of the values,
compare will exhibit its normal behavior. The case expression
in theemployeeRank function is otherwise unchanged.
And here’s how that works:
Prelude> employeeRank compare Coder CEO
CEO is the boss of Coder
Prelude> let cs = codersRuleCEOsDrool
Prelude> employeeRank cs Coder CEO
Coder is the boss of CEO
Prelude> employeeRank cs CEO Coder
Coder is the boss of CEO
If we use compare as our 𝑓argument, then the behavior
is unchanged. If, on the other hand, we use our new func-
tion,codersRuleCEOsDrool as the𝑓argument, then the behavior
changes and we unleash anarchy in the cubicle farm.
We were able to rely on the behavior of compare but make
changes in the part we wanted to change. This is the value of
HOFs. They give us the beginnings of a powerful method for
reusing and composing code.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 376
Exercises: Artful Dodgy
Given the following definitions tell us what value results from
further applications. When you’ve written down at least some
of the answers and think you know what’s what, type the def-
initions into a file and load them in GHCi to test your answers.
-- Types not provided,
-- try filling them in yourself.
dodgyx y=x+y*10
oneIsOne =dodgy1
oneIsTwo =(flip dodgy) 2
1.For example, given the expression dodgy 1 0 , what do you
think will happen if we evaluate it? If you put the def-
initions in a file and load them in GHCi, you can do the
following to see the result.
Prelude> dodgy 1 0
1
Now attempt to determine what the following expressions
reduce to. Do it in your head, verify in your REPL after
you think you have an answer.
2.dodgy 1 1

CHAPTER 7. MORE FUNCTIONAL PATTERNS 377
3.dodgy 2 2
4.dodgy 1 2
5.dodgy 2 1
6.oneIsOne 1
7.oneIsOne 2
8.oneIsTwo 1
9.oneIsTwo 2
10.oneIsOne 3
11.oneIsTwo 3
7.7 Guards
We have played around with booleans and expressions that
evaluate to their truth value including if-then-else expressions
which rely on boolean evaluation to decide between two out-
comes. In this section, we will look at another syntactic pattern
called guards that relies on truth values to decide between two
or more possible results.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 378
if-then-else
Let’s begin with a quick review of what we learned about
if-then-else expressions in the Basic Datatypes chapter. Note,
if-then-else isnotguards! This is review, before moving on to
guards themselves. The pattern is this:
if <condition>
then <result if True>
else <result if False>
where the ifcondition is an expression that results in a Bool
value. We saw how this allows us to write functions like this:
Prelude> let x = 0
Prelude> let a = "AWESOME"
Prelude> let w = "wut"
Prelude> if (x + 1 == 1) then a else w
"AWESOME"
The next couple of examples will demonstrate how to use
the multiline block syntax for an ifexpression:
-- alternately
Prelude> let x = 0
Prelude> :{
Prelude| if (x + 1 == 1)
Prelude| then "AWESOME"

CHAPTER 7. MORE FUNCTIONAL PATTERNS 379
Prelude| else "wut"
Prelude| :}
"AWESOME"
The indentation isn’t required:
Prelude> let x = 0
Prelude> :{
Prelude| if (x + 1 == 1)
Prelude| then "AWESOME"
Prelude| else "wut"
Prelude| :}
"AWESOME"
In the exercises at the end of Chapter 4, you were asked to
write a function called myAbsthat returns the absolute value of
a real number. You would have implemented that function
with an if-then-else expression similar to the following:
myAbs::Integer ->Integer
myAbsx= ifx<0then(-x)elsex
We’re going to look at another way to write this using guards.
Writing guard blocks
Guard syntax allows us to write compact functions that allow
for two or more possible outcomes depending on the truth of

CHAPTER 7. MORE FUNCTIONAL PATTERNS 380
the conditions. Let’s start by looking at how we would write
myAbswith a guard block instead of with an if-then-else :
myAbs::Integer ->Integer
myAbsx
|x<0=(-x)
|otherwise =x
Notice that each guard has its own equals sign. We didn’t
put one after the argument in the first line of the function def-
inition because each case needs its own expression to return
if its branch succeeds. Now we’ll enumerate the components
for clarity:
myAbs::Integer ->Integer
myAbs x
-- [1] [2]
|x<0=(-x)
-- [3] [4] [5] [6]
|otherwise =x
-- [7] [8] [9] [10]
1.The name of our function, myAbsstill comes first.
2.There is one parameter named 𝑥.
3.Here’s where it gets diﬀerent. Rather than an =imme-
diately after the introduction of any parameter(s), we’re

CHAPTER 7. MORE FUNCTIONAL PATTERNS 381
starting a new line and using the pipe|to begin a guard
case.
4.This is the expression we’re using to test to see if this
branch should be evaluated or not. The guard case ex-
pression between the |and=must evaluate to Bool.
5.The=denotes that we’re declaring what expression to
return should our x < 0beTrue.
6.Then after the =we have the expression (-x)which will
be returned if x < 0.
7.Another new line and a |to begin a new guard case.
8.otherwise is another name for True, used here as a fallback
case in case x < 0wasFalse.
9.Another =to begin declaring the expression to return if
we hit the otherwise case.
10.We kick 𝑥back out if it wasn’t less than 0.
Let’s see how this evaluates:
Prelude> myAbs (-10)
10
Prelude> myAbs 10
10

CHAPTER 7. MORE FUNCTIONAL PATTERNS 382
In the first example, when it is passed a negative number
as an argument, it looks at the first guard and sees that (-10)
is indeed less than 0, evaluates that as True, and so returns
the result of (-x), in this case, (-(-10)) or 10. In the second
example, it looks at the first guard, sees that 10 does not meet
that condition, so it is False, and goes to the next guard. The
otherwise is always True, so it returns 𝑥, in this case, 10. Guards
always evaluate sequentially, so your guards should be ordered
from the case that is most restrictive to the case that is least
restrictive.
Let’s look next at a function that will have more than two
possible outcomes, in this case the results of a test of sodium
(Na) levels in the blood. We want a function that looks at the
numbers (the numbers represent mEq/L or milliequivalents
per liter) and tells us if the blood sodium levels are normal or
not:
bloodNa ::Integer ->String
bloodNa x
|x<135="too low"
|x>145="too high"
|otherwise ="just right"
We can incorporate diﬀerent types of expressions into the
guard block, as long as each guard can be evaluated to a Bool
value. For example, the following takes 3 numbers and tells

CHAPTER 7. MORE FUNCTIONAL PATTERNS 383
you if the triangle whose sides they measure is a right triangle
or not (using the Pythagorean theorem):
-- c is the hypotenuse of
-- the triangle.
isRight ::(Numa,Eqa)
=>a->a->a->String
isRight a b c
|a^2+b^2==c^2="RIGHT ON"
|otherwise ="not right"
And the following function will take your dog’s age and tell
you how old your dog is in human years:
dogYrs::Integer ->Integer
dogYrsx
|x<=0=0
|x<=1=x*15
|x<=2=x*12
|x<=4=x*8
|otherwise =x*6
Why the diﬀerent numbers? Because puppies reach matu-
rity much faster than human babies do, so a year-old puppy
isn’t equivalent to a 6- or 7-year-old child (there is more com-
plexity to this conversion than this function uses, because

CHAPTER 7. MORE FUNCTIONAL PATTERNS 384
other factors such as the size of the dog play a role as well. You
can certainly experiment with that if you like).
We can also use wheredeclarations within guard blocks. Let’s
say you gave a test that had 100 questions and you wanted a
simple function for translating the number of questions the
student got right into a letter grade:
avgGrade ::(Fractional a,Orda)
=>a->Char
avgGrade x
|y>=0.9='A'
|y>=0.8='B'
|y>=0.7='C'
|y>=0.59='D'
|y<0.59='F'
wherey=x/100
No surprises there. Notice the variable 𝑦is introduced, not
as an argument to the named function but in the guard block
and is defined in the whereclause. By defining it there, it is in
scope for all the guards above it. There were 100 problems on
the hypothetical test, so any 𝑥we give it will be divided by 100
to return the letter grade.
Also notice we left out the otherwise ; we could have used it
for the final case but chose instead to use less than . That is fine
because in our guards we’ve handled all possible values. It is

CHAPTER 7. MORE FUNCTIONAL PATTERNS 385
important to note that GHCi cannot always tell you when you
haven’t accounted for all possible cases, and it can be difficult
to reason about it, so it is wise to use otherwise in your final
guard.
Remember: You can use :set -Wall in GHCi to turn on
warnings, and then it will tell you if you have non-exhaustive
patterns.
Exercises: Guard Duty
1.Itisprobablycleartoyouwhyyouwouldn’tputan otherwise
in your top-most guard, but try it with avgGrade anyway
and see what happens. It’ll be more clear if you rewrite
it as anotherwise match: | otherwise = 'F' . What happens
now if you pass a 90 as an argument? 75? 60?
2.What happens if you take avgGrade as it is written and
reorder the guards? Does it still typecheck and work the
same? Try moving | y >= 0.7 = 'C' and passing it the
argument 90, which should be an ‘A.’ Does it return an ‘A’?
3.The following function returns
palxs
|xs==reverse xs =True
|otherwise =False
a)xswritten backwards when it’s True

CHAPTER 7. MORE FUNCTIONAL PATTERNS 386
b)Truewhenxsis a palindrome
c)Falsewhenxsis a palindrome
d)Falsewhenxsis reversed
4.What types of arguments can paltake?
5.What is the type of the function pal?
6.The following function returns
numbers x
|x<0= -1
|x==0=0
|x>0=1
a)the value of its argument plus or minus 1
b)the negation of its argument
c)an indication of whether its argument is a positive or
negative number or zero
d)binary machine language
7.What types of arguments can numbers take?
8.What is the type of the function numbers ?

CHAPTER 7. MORE FUNCTIONAL PATTERNS 387
7.8 Function composition
Function composition is a type of higher-order function that
allows us to combine functions such that the result of applying
one function gets passed to the next function as an argument.
It is a very concise style, in keeping with the terse functional
style Haskell is known for. At first, it seems complicated and
difficult to unpack, but once you get the hang of it, it’s fun!
Let’s begin by looking at the type signature and what it means:
(.)::(b->c)->(a->b)->a->c
-- [1] [2] [3] [4]
1.is a function from 𝑏to𝑐, passed as an argument (thus the
parentheses).
2.is a function from 𝑎to𝑏.
3.is a value of type 𝑎, the same as [2]expects as an argument.
4.is a value of type 𝑐, the same as [1]returns as a result.
Then with the addition of one set of parentheses:
(.)::(b->c)->(a->b)->(a->c)
-- [1] [2] [3]
In English:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 388
1.given a function 𝑏to𝑐
2.given a function 𝑎to𝑏
3.return a function 𝑎to𝑐.
The result of (a -> b) is the argument of (b -> c) so this is
how we get from an 𝑎argument to a 𝑐result. We’ve stitched
the result of one function into being the argument of another.
Next let’s start looking at composed functions and how
to read and work with them. The basic syntax of function
composition looks like this:
(f.g) x=f (g x)
This composition operator, (.), takes two functions here,
named 𝑓and𝑔. The𝑓function corresponds to the (b -> c) in
the type signature, while the 𝑔function corresponds to the
(a -> b) . The𝑔function is applied to the (polymorphic) 𝑥
argument. The result of that application then passes to the 𝑓
function as its argument. The 𝑓function is in turn applied to
that argument and evaluated to reach the final result.
Let’s go step by step through this transformation. We can
think of the (.)or composition operator as being a way of
pipelining data through multiple functions. The following
composed functions will first add the values in the list together
and then negate the result of that:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 389
Prelude> negate . sum $ [1, 2, 3, 4, 5]
-15
-- which is evaluated like this
negate . sum $ [1, 2, 3, 4, 5]
-- note: this code works as well
negate (sum [1, 2, 3, 4, 5])
negate (15)
-15
Notice that we did this directly in our REPL, because the
composition operator is already in scope in Prelude . The sum
of the list is 15. That result gets passed to the negate function
and returns a result of (-15).
You may be wondering why we need the $operator. You
might remember way back when we talked about the prece-
dence of various operators that we said that operator has a
lower precedence than an ordinary function call (white space,
usually). Ordinary function application has a precedence of
10 (out of 10). The composition operator has a precedence of
9. If we left white space as our function application, this would
be evaluated like this:
negate . sum [1, 2, 3, 4, 5]
negate . 15

CHAPTER 7. MORE FUNCTIONAL PATTERNS 390
Because function application has a higher precedence than
the composition operator, that function application would
happen before the two functions composed. We’d be trying to
pass a numeric value where our composition operator needs a
function. By using the $we signal that application to the argu-
ments should happen afterthe functions are already composed.
We can also parenthesize it instead of using the $operator.
In that case, it looks like this:
Prelude> (negate . sum) [1, 2, 3, 4, 5]
-15
The choice of whether to use parentheses or the dollar sign
isn’t important; it is a question of style and ease of writing and
reading.
The next example uses two functions, takeandreverse , and
is applied to an argument that is a list of numbers from 1 to 10.
What we expect to happen is that the list will first be reversed
(from 10 to 1) and then the first 5 elements of the new list will
be returned as the result.
Prelude> take 5 . reverse $ [1..10]
[10,9,8,7,6]
Given the next bit of code, how could we rewrite it to use
function composition instead of parentheses?

CHAPTER 7. MORE FUNCTIONAL PATTERNS 391
Prelude> take 5 (enumFrom 3)
[3,4,5,6,7]
We know that we will have to eliminate the parentheses,
add the composition operator, and add the $operator. It will
then look like this:
Prelude> take 5 . enumFrom $ 3
[3,4,5,6,7]
You may also define it this way, which is more similar to
how composition is written in source files:
Prelude> let f x = take 5 . enumFrom $ x
Prelude> f 3
[3,4,5,6,7]
You may be wondering why bother with this if it simply
does the same thing as nesting functions in parentheses. One
reason is that it is quite easy to compose more than two func-
tions this way.
Thefilter odd function is new for us, but it simply filters the
odd numbers (you can change it to filter even if you wish) out
of the list that enumFrom builds for us. Finally, takewill return
as the result only the number of elements we have specified
as the argument of take. Feel free to experiment with varying
any of the arguments.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 392
Prelude> take 5 . filter odd . enumFrom $ 3
[3,5,7,9,11]
As you compose more functions, you can see that nesting
all the parentheses would become tiresome. This operator
allows us to do away with that. It also allows us to write in an
even more terse style known as “pointfree.”
7.9 Pointfree style
Pointfree refers to a style of composing functions without
specifying their arguments. The “point” in “pointfree” refers
to the arguments, not (as it may seem) to the function compo-
sition operator. In some sense, we add “points” (the operator)
to be able to drop points (arguments). Quite often, pointfree
code is tidier on the page and easier to read as it helps the
reader focus on the functions rather than the data that is being
shuffled around.
We said above that function composition looks like this:
(f . g) x = f (g x)
As you put more functions together, composition can make
them easier to read. For example, (f. g. h) x can be easier
to read than f (g (h x)) and it also brings the focus to the
functions rather than the arguments. Pointfree is an extension
of that idea but now we drop the argument altogether:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 393
f . g = \x -> f (g x)
f . g . h = \x -> f (g (h x))
To see what this looks like in practice, we’ll start by rewriting
in pointfree style some of the functions we used in the section
above:
Prelude> let f = negate . sum
Prelude> f [1, 2, 3, 4, 5]
-15
Notice that when we define our function 𝑓we don’t spec-
ify that there will be any arguments. Yet when we apply the
function to an argument, the same thing happens as before.
How would we rewrite:
f::Int->[Int]->Int
fz xs=foldr (+) z xs
as a pointfree function?
Prelude> let f = foldr (+)
Prelude> f 0 [1..5]
15
And now because we named the function, it can be reused
with diﬀerent arguments.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 394
Here is another example of a short pointfree function and
its result. It involves a new use of filter that uses the Bool
operator ==. Look at it carefully and, on paper or in your head,
walk through the evaluation process involved:
Prelude> let f = length . filter (== 'a')
Prelude> f "abracadabra"
5
Next, we’ll look at a set of functions that work together, in a
single module, and rely on both composition and pointfree
style:
-- arith2.hs
moduleArith2where
add::Int->Int->Int
addx y=x+y
addPF::Int->Int->Int
addPF=(+)
addOne::Int->Int
addOne=\x->x+1
addOnePF ::Int->Int
addOnePF =(+1)

CHAPTER 7. MORE FUNCTIONAL PATTERNS 395
main::IO()
main= do
print (0::Int)
print (add 10)
print (addOne 0)
print (addOnePF 0)
print ((addOne .addOne) 0)
print ((addOnePF .addOne) 0)
print ((addOne .addOnePF) 0)
print ((addOnePF .addOnePF) 0)
print (negate (addOne 0))
print ((negate .addOne) 0)
print ((addOne .addOne.addOne
.negate.addOne) 0)
Take your time and work through what each function is
doing, whether on paper or in your head. Then load this code
as a source file and run it in GHCi and see if your results were
accurate.
You should now have a good understanding of how you
can use (.)tocompose functions. It’s important to remember
that the functions in composition are applied from right to
left, like a Pacman munching from the right side, reducing the
expressions as he goes.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 396
7.10 Demonstrating composition
You may recall back in Chapter 3 we mentioned that the func-
tionsprintandputStr seem similar on the surface but behave
diﬀerently because they have diﬀerent underlying types. Let’s
take a closer look at that now.
First,putStrLn andputStr have the same type:
putStr :: String -> IO ()
putStrLn :: String -> IO ()
But the type of printis diﬀerent:
print :: Show a => a -> IO ()
They all return a result of IO ()for reasons we discussed
in the previous chapter. But the parameters here are quite
diﬀerent. The first two take String s as arguments, while print
has a constrained polymorphic parameter, Show a => a . The
first two work fine if we need to display values that are already
of type String . But how do we display numbers (or other non-
string values)? First we have to convert those numbers to
strings, then we can print the strings.
You may also recall a function from our discussion of the
Showtypeclass called show. Here’s the type of showagain:
show::Showa=>a->String

CHAPTER 7. MORE FUNCTIONAL PATTERNS 397
Fortunately, it was understood that combining putStrLn and
showwould be a common pattern, so the function named print
is the composition of showandputStrLn . We do it this way
because it’s simpler . The printing function concerns itself only
with printing, while the stringification function concerns itself
only with that.
Here are two ways to implement printwithputStrLn and
show:
print::Showa=>a->IO()
printa=putStrLn (show a)
-- using the . operator for
-- composing functions.
(.)::(b->c)->(a->b)->a->c
-- we can write print as:
print::Showa=>a->IO()
printa=(putStrLn .show) a
Now let’s go step by step through this use of (.),putStrLn ,
andshow:

CHAPTER 7. MORE FUNCTIONAL PATTERNS 398
(.) ::(b->c)->(a->b)->a->c
putStrLn ::String->IO()
-- [1] [2]
show ::Showa=>a->String
-- [3] [4]
putStrLn .show::Showa=>a->IO()
-- [5] [6]
(.)::(b->c)->(a->b)->a->c
-- [1] [2] [3] [4] [5] [6]
-- If we replace the variables with
-- the specific types they take on
-- in this application of (.)
(.)::Showa=>(String->IO())
->(a->String)
->a->IO()

CHAPTER 7. MORE FUNCTIONAL PATTERNS 399
(.)::(b ->c)
-- (String -> IO ())
->(a->b)
-- (a -> String)
->a->c
-- a -> IO ()
1.is the string that putStrLn accepts as an argument.
2.is theIO ()thatputStrLn returns, that is, performing the
side eﬀect of printing and returning unit.
3.is𝑎that must implement the Showtypeclass; this is the Show
a => a from the showfunction which is a method on the
Showtypeclass.
4.is the string that showreturns. This is what the Show a => a
value got stringified into.
5.is theShow a => a the final composed function expects.
6.is theIO ()the final composed function returns.
We can now make it pointfree. When we are working with
functions primarily in terms of composition rather than appli-
cation, the pointfree version can sometimes (not always) be
more elegant.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 400
Here’s the previous version of the function:
print::Showa=>a->IO()
printa=(putStrLn .show) a
And here’s the pointfree version of print:
print::Showa=>a->IO()
print=putStrLn .show
The point of printis to compose putStrLn andshowso that
we don’t have to call showon its argument ourselves. That is,
printis principally about the composition of two functions,
so it comes out nicely as a pointfree function. Saying that
we could apply putStrLn . show to an argument in this case is
redundant.
7.11 Chapter Exercises
Multiple choice
1.A polymorphic function
a)changes things into sheep when invoked
b)has multiple arguments
c)has a concrete type

CHAPTER 7. MORE FUNCTIONAL PATTERNS 401
d)may resolve to values of diﬀerent types, depending
on inputs
2.Two functions named fandghave types Char -> String
andString -> [String] respectively. The composed func-
tiong . fhas the type
a)Char -> String
b)Char -> [String]
c)[[String]]
d)Char -> String -> [String]
3.A function fhas the type Ord a => a -> a -> Bool and we
apply it to one numeric value. What is the type now?
a)Ord a => a -> Bool
b)Num -> Num -> Bool
c)Ord a => a -> a -> Integer
d)(Ord a, Num a) => a -> Bool
4.A function with the type (a -> b) -> c
a)requires values of three diﬀerent types
b)is a higher-order function
c)must take a tuple as its first argument
d)has its parameters in alphabetical order

CHAPTER 7. MORE FUNCTIONAL PATTERNS 402
5.Given the following definition of f, what is the type of f
True?
f::a->a
fx=x
a)f True :: Bool
b)f True :: String
c)f True :: Bool -> Bool
d)f True :: a
Let’s write code
1.The following function returns the tens digit of an integral
argument.
tensDigit ::Integral a=>a->a
tensDigit x=d
wherexLast=x `div` 10
d=xLast `mod` 10
a)First, rewrite it using divMod .
b)Does the divMod version have the same type as the
original version?
c)Next, let’s change it so that we’re getting the hundreds
digit instead. You could start it like this (though that
may not be the only possibility):

CHAPTER 7. MORE FUNCTIONAL PATTERNS 403
hunsDx=d2
whered=undefined
...
2.Implement the function of the type a -> a -> Bool -> a
once each using a case expression and once with a guard.
foldBool ::a->a->Bool->a
foldBool =
error
"Error: Need to implement foldBool!"
The result is semantically similar to if-then-else expres-
sions but syntactically quite diﬀerent. Here is the pattern
matching version to get you started:
foldBool3 ::a->a->Bool->a
foldBool3 x_False=x
foldBool3 _yTrue=y
3.Fill in the definition. Note that the first argument to our
function is alsoa function which can be applied to values.
Your second argument is a tuple, which can be used for
pattern matching:
g::(a->b)->(a, c)->(b, c)
g=undefined

CHAPTER 7. MORE FUNCTIONAL PATTERNS 404
4.For this next exercise, you’ll experiment with writing
pointfree versions of existing code. This involves some
new information, so read the following explanation care-
fully.
Typeclasses are dispatched by type. Readis a typeclass like
Show, but it is the dual or “opposite” of Show. In general, the
Readtypeclass isn’t something you should plan to use a
lot, but this exercise is structured to teach you something
about the interaction between typeclasses and types.
The function readin theReadtypeclass has the type:
read::Reada=>String->a
Notice a pattern?
read::Reada=>String->a
show::Showa=>a->String
Write the following code into a source file. Then load it
and run it in GHCi to make sure you understand why the
evaluation results in the answers you see.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 405
-- arith4.hs
moduleArith4where
-- id :: a -> a
-- id x = x
roundTrip ::(Showa,Reada)=>a->a
roundTrip a=read (show a)
main= do
print (roundTrip 4)
print (id 4)
5.Next, write a pointfree version of roundTrip . (n.b., This
refers to the function definition, not to its application in
main.)
6.We will continue to use the code in module Arith4 for this
exercise as well.
When we apply showto a value such as (1 :: Int) , the𝑎that
implements Show is Int, so GHC will use the Int instance
of the Show typeclass to stringify our Int of 1.
However, readexpects a String argument in order to re-
turn an 𝑎. TheString argument that is the first argument
toreadtells the function nothing about what type the de-
stringified result should be. In the type signature roundTrip

CHAPTER 7. MORE FUNCTIONAL PATTERNS 406
currently has, it knows because the type variables are the
same, so the type that is the input to showhas to be the
same type as the output of read.
Your task now is to change the type of roundTrip inArith4 to
(Show a, Read b) => a -> b . How might we tell GHC which
instance of Readto dispatch against the String now? Make
the expression print (roundTrip 4) work. You will only
need the has the type syntax of ::and parentheses for
scoping.
7.12 Chapter Definitions
1.Binding orbound is a common word used to indicate con-
nection, linkage, or association between two objects. In
Haskell we’ll use it to talk about what value a variable has,
e.g., a parameter variable is bound to an argument value,
meaning the value is passed into the parameter as input
and each occurrence of that named parameter will have
the same value. Bindings as a plurality will usually refer
to a collection of variables and functions which can be
referenced by name.
blah::Int
blah=10
Here the variable blahis bound to the value 10.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 407
2.Ananonymous function is a function which is not bound to
an identifier and is instead passed as an argument to an-
other function and/or used to construct another function.
See the following examples.
\x->x
-- anonymous version of id
idx=x
-- not anonymous, it's bound to 'id'
3.Currying is the process of transforming a function that
takes multiple arguments into a series of functions which
each take one argument and return one result. This is ac-
complished through the nesting. In Haskell, all functions
are curried by default. You don’t need to do anything
special yourself.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 408
-- curry and uncurry already
-- exist in Prelude
curry'::((a, b) ->c)->a->b->c
curry'f a b=f (a, b)
uncurry' ::(a->b->c)->((a, b) ->c)
uncurry' f (a, b) =f a b
-- uncurried function,
-- takes a tuple of its arguments
add::(Int,Int)->Int
add(x, y)=x+y
add'::Int->Int->Int
add'=curry' add
A function that appears to take two arguments is two func-
tions that each take one argument and return one result.
What makes this work is that a function can return another
function.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 409
fa b=a+b
-- is equivalent to
f=\a->(\b->a+b)
4.Pattern matching is a syntactic way of deconstructing prod-
uct and sum types to get at their inhabitants. With re-
spect to products, pattern matching gives you the means
for destructuring and exposing the contents of products,
binding one or more values contained therein to names.
With sums, pattern matching lets you discriminate which
inhabitant of a sum you mean to handle in that match.
It’s best to explain pattern matching in terms of how
datatypes work, so we’re going to use terminology that
you may not fully understand yet. We’ll cover this more
deeply soon.
-- nullary data constructor,
-- not a sum or product.
-- Just a single value.
dataBlah=Blah
Pattern matching on Blahcan only do one thing.
blahFunc ::Blah->Bool
blahFunc Blah=True

CHAPTER 7. MORE FUNCTIONAL PATTERNS 410
dataIdentity a=
Identity a
deriving (Eq,Show)
Identity is a unary data constructor. Still not a product,
only contains one value.
-- when you pattern match on Identity
-- you can unpack and expose the 'a'
unpackIdentity ::Identity a->a
unpackIdentity (Identity x)=x
-- But you can choose to ignore
-- the contents of Identity
ignoreIdentity ::Identity a->Bool
ignoreIdentity (Identity _)=True
-- or ignore it completely since
-- matching on a non-sum data constructor
-- changes nothing.
ignoreIdentity' ::Identity a->Bool
ignoreIdentity' _ =True

CHAPTER 7. MORE FUNCTIONAL PATTERNS 411
dataProduct a b=
Product a b
deriving (Eq,Show)
Now we can choose to use none, one, or both of the values
in the product of 𝑎and𝑏:
productUnpackOnlyA ::Product a b->a
productUnpackOnlyA (Product x_)=x
productUnpackOnlyB ::Product a b->b
productUnpackOnlyB (Product _y)=y
Or we can bind them both to a diﬀerent name:
productUnpack ::Product a b->(a, b)
productUnpack (Product x y)=(x, y)
What happens if you try to bind the values in the product
to the same name?
dataSumOfThree a b c=
FirstPossible a
|SecondPossible b
|ThirdPossible c
deriving (Eq,Show)

CHAPTER 7. MORE FUNCTIONAL PATTERNS 412
Now we can discriminate by the inhabitants of the sum
and choose to do diﬀerent things based on which con-
structor in the sum they were.
sumToInt ::SumOfThree a b c->Integer
sumToInt (FirstPossible _)=0
sumToInt (SecondPossible _)=1
sumToInt (ThirdPossible _)=2
-- We can selectively ignore
-- inhabitants of the sum
sumToInt ::SumOfThree a b c->Integer
sumToInt (FirstPossible _)=0
sumToInt _ = 1
-- We still need to handle
-- every possible value
Pattern matching is about your data.
5.Bottom is a non-value used to denote that the program
cannot return a value or result. The most elemental
manifestation of this is a program that loops infinitely.
Other forms can involve things like writing a function

CHAPTER 7. MORE FUNCTIONAL PATTERNS 413
that doesn’t handle all of its inputs and fails on a pattern
match. The following are examples of bottom:
-- If you apply this to any values,
-- it'll recurse indefinitely.
fx=f x
-- It'll a'splode if you pass a False value
dontDoThis ::Bool->Int
dontDoThis True=1
-- morally equivalent to
definitelyDontDoThis ::Bool->Int
definitelyDontDoThis True=1
definitelyDontDoThis False=error"oops"
-- don't use error.
-- We'll show you a better way soon.
Bottom can be useful as a canary for signaling when code
paths arebeing evaluated. We usually do this to determine
how lazy a program is or isn’t. You’ll see a lotof this in
our chapter on non-strictness later on.
6.Higher-orderfunctions are functions which themselves take

CHAPTER 7. MORE FUNCTIONAL PATTERNS 414
functions as arguments or return functions as results. Due
to currying, technically any function that appears to take
more than one argument is higher order in Haskell.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 415
-- Technically higher order
-- because of currying
Int->Int->Int
-- See? Returns another function
-- after applying the first argument
Int->(Int->Int)
-- The rest of the following examples are
-- types of higher order functions
(a->b)->a->b
(a->b)->[a]->[b]
(Int->Bool)->[Int]->[Bool]
-- also higher order, this one
-- takes a function argument which itself
-- is higher order as well.
((a->b)->c)->[a]->[c]
7.Composition is the application of a function to the result
of having applied another function. The composition op-

CHAPTER 7. MORE FUNCTIONAL PATTERNS 416
erator is a higher-order function as it takes the functions
it composes as arguments and then returns a function of
the composition:
(.)::(b->c)->(a->b)->a->c
-- is
(.)::(b->c)->(a->b)->(a->c)
-- or
(.)::(b->c)->((a->b)->(a->c))
-- can be implemented as
comp::(b->c)->((a->b)->(a->c))
compf g x=f (g x)
The function 𝑔is applied to 𝑥,𝑓is applied to the result of
g x.
8.Pointfree is programming tacitly, or without mentioning
arguments by name. This tends to look like “plumby”
code where you’re routing data around implicitly or leav-
ing oﬀ unnecessary arguments thanks to currying. The
“point” referred to in the term pointfree is an argument.

CHAPTER 7. MORE FUNCTIONAL PATTERNS 417
-- not pointfree
blahx=x
addAndDrop x y=x+1
reverseMkTuple a b=(b, a)
reverseTuple (a, b)=(b, a)
-- pointfree versions of the above
blah=id
addAndDrop =const.(1+)
reverseMkTuple =flip (,)
reverseTuple =uncurry (flip (,))
To see more examples like this, check out the Haskell Wiki
page on Pointfree at https://wiki.haskell.org/Pointfree .
7.13 Follow-up resources
1.Paul Hudak; John Peterson; Joseph Fasel. A Gentle In-
troduction to Haskell, chapter on case expressions and
pattern matching.
https://www.haskell.org/tutorial/patterns.html
2.Simon Peyton Jones. The Implementation of Functional
Programming Languages, pages 53-103.
http://research.microsoft.com/en-us/um/people/simonpj/papers/
slpj-book-1987/index.htm

CHAPTER 7. MORE FUNCTIONAL PATTERNS 418
3.Christopher Strachey. Fundamental Concepts in Pro-
gramming Languages, page 11 for explanation of curry-
ing.
http://www.cs.cmu.edu/~crary/819-f09/Strachey67.pdf
4.J.N. Oliveira. An introduction to pointfree programming.
http://www.di.uminho.pt/~jno/ps/iscalc_1.ps.gz
5.Manuel Alcino Pereira da Cunha. Point-free Program
Calculation.
http://www4.di.uminho.pt/~mac/Publications/phd.pdf

Chapter 8
Recursion
Imagine a portion of the
territory of England has
been perfectly levelled,
and a cartographer traces
a map of England. The
work is perfect. There is
no particular of the
territory of England,
small as it can be, that has
not been recorded in the
map. Everything has its
own correspondence.
The map, then, must
contain a map of the map,
that must contain a map
of the map of the map,
and so on to infinity.
Jorge Luis Borges, citing
Josiah Royce
419

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 420
8.1 Recursion
Recursion is defining a function in terms of itself via self-
referential expressions. It means that the function will con-
tinue to call itself and repeat its behavior until some condition
is met to return a result. It’s an important concept in Haskell
and in mathematics because it gives us a means of express-
ingindefinite or incremental computation without forcing us
to explicitly repeat ourselves and allowing the data we are
processing to decide when we are done computing.
Recursion is a natural property of many logical and math-
ematical systems, including human language. That there is
no limit on the number of expressible, valid sentences in hu-
man language is due to recursion. A sentence in English can
have another sentence nested within it. Sentences can be
roughly described as structures which have a noun phrase, a
verb phrase, and optionally another sentence. This possibility
for unlimited nested sentences is recursive and enables the
limitless expressibility therein. Recursion is a means of ex-
pressing code that must take an indefinite number of steps to
return a result.
But the lambda calculus does not appear on the surface
to have any means of recursion, because of the anonymity
of expressions. How do you call something without a name?
Being able to write recursive functions, though, is essential to
Turing completeness. We use a combinator — known as the Y

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 421
combinator or fixed-point combinator — to write recursive
functions in the lambda calculus. Haskell has native recursion
ability based on the same principle as the Y combinator.
It is important to have a solid understanding of the behavior
of recursive functions. In later chapters, we will see that, in
fact, it is not often necessary to write our own recursive func-
tions, as many standard higher-order functions have built-in
recursion. But without understanding the systematic behav-
ior of recursion itself, it can be difficult to reason about those
HOFs. In this chapter, we will
•explore what recursion is and how recursive functions
evaluate;
•go step-by-step through the process of writing recursive
functions;
•have fun with bottom .
8.2 Factorial!
One of the classic introductory functions for demonstrating
recursion in functional languages is factorial. In arithmetic,
you might’ve seen expressions like 4!. Thebangyou’re seeing
next to the number 4 is the notation for the factorial function.
Let’s evaluate 4!:

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 422
4! = 4 * 3 * 2 * 1
12 * 2 * 1
24 * 1
24
4! = 24
Now let’s do it the silly way in Haskell:
fourFactorial ::Integer
fourFactorial =4*3*2*1
This will return the correct result, but it only covers one
possible result for factorial . This is less than ideal. We want
to express the general idea of the function, not encode specific
inputs and outputs manually.
Now we’ll look at some broken code to introduce the con-
cept of a base case :
-- This won't work. It never stops.
brokenFact1 ::Integer ->Integer
brokenFact1 n=n*brokenFact1 (n -1)
Let’s apply this to 4 and see what happens:

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 423
brokenFact1 4=
4*(4-1)
*((4-1)-1)
*(((4-1)-1)-1)
...this series never stops
The way we can stop a recursive expression is by having a
base case that stops the self-application to further arguments.
Understanding this is critical for writing functions which are
correct and terminate properly. Here’s what this looks like for
factorial :
moduleFactorial where
factorial ::Integer ->Integer
factorial 0=1
factorial n=n*factorial (n -1)
brokenFact1 4=
4*(4-1)
*((4-1)-1)
*(((4-1)-1)-1)
*((((4-1)-1)-1)-1)
*(((((4-1)-1)-1)-1)-1)
...never stops

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 424
But the base case factorial 0 = 1 in the fixed version gives
our function a stopping point, so the reduction changes:
-- Changes to
-- n = n * factorial (n - 1)
factorial 4=
4*factorial ( 4-1)
-- evaluate (-) applied to 4 and 1
4*factorial 3
-- evaluate factorial applied to 3
-- expands to 3 * factorial (3 - 1)
4*3*factorial ( 3-1)
-- beta reduce (-) applied to 3 and 1
4*3*factorial 2
-- evaluate factorial applied to 2
4*3*2*factorial ( 2-1)
-- evaluate (-) applied to 2 and 1
4*3*2*factorial 1
-- evaluate factorial applied to 1
4*3*2*1*factorial ( 1-1)

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 425
-- evaluate (-) applied to 1 and 1
-- we know factorial 0 = 1
-- so we evaluate that to 1
4*3*2*1*1
-- And when we evaluate
-- our multiplications
24
Making our base case an identity value for the function
(multiplication in this case) means that applying the function
to that case doesn’t change the result of previous applications.
Another way to look at recursion
In the last chapter, we looked at a higher-order function called
composition. Function composition is a way of tying two (or
more) functions together such that the result of applying the
first function gets passed as an argument to the next function.
This is the same thing recursive functions are doing — taking
the result of the first application of the function and passing it
to the next function — except in the case of recursive functions,
thefirstresultgetspassedbacktothesamefunctionratherthan
a diﬀerent one, until it reaches the base case and terminates.

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 426
Where function composition as we normally think of it is
static and definite, recursive compositions are indefinite. The
number of times the function may be applied depends on the
arguments to the function, and the applications can be infinite
if a stopping point is not clearly defined.
Let’s recall that function composition has the following
type:
(.) :: (b -> c) -> (a -> b) -> a -> c
And when we use it like this:
take5.filter odd .enumFrom $3
we know that the first result will be a list generated by
enumFrom which will be passed to filter odd , giving us a list of
only the odd results, and that list will be passed to take 5 and
our final result will be the first five members of that list. Thus,
results get piped through a series of functions.
Recursion is self-referential composition.1We apply a func-
tion to an argument, then pass that result on as an argument
to a second application of the same function and so on.
Now look again at how the compose function (.)is written:
(.)::(b->c)->(a->b)->a->c
(.) f g=\x->f (g x)
1Many thanks to George Makrydakis for discussing this with us.

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 427
A programming language, such as Haskell, that is built
purely on lambda calculus has one verb for expressing compu-
tations that can be evaluated: apply. We apply a function to an
argument. Applying a function to an argument and potentially
doing something with the result is all we can do, no matter
what syntactic conveniences we construct to make it seem that
we are doing more than that. While we give function compo-
sition a special name and operator to point up the pattern and
make it convenient to use, it’s only a way of saying:
•given two functions, 𝑓and𝑔, as arguments to (.),
•when we get an argument 𝑥, apply𝑔to𝑥,
•then apply 𝑓to the result of (g x); or,
•to rephrase, in code:
(.) f g=\x->f (g x)
With function recursion, you might notice that it is func-
tion application in the same way that composition is. The
diﬀerence is that instead of a fixed number of applications,
recursive functions rely on inputs to determine when to stop
applying functions to successive results. Without a specified

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 428
stopping point, the result of (g x)will keep being passed back
to𝑔indefinitely.2
Let’s look at some code to see the similarity in patterns:
inc::Numa=>a->a
inc=(+1)
three=inc.inc.inc$0
-- different syntax, same thing
three'=(inc.inc.inc)0
Our composition of incbakes the number of applications
into the source code. We don’t presently have a means of
changing how many times we want it to apply incwithout
writing a new function.
So, we might want to make a general function that can apply
incan indefinite number of times and allow us to specify as
an argument how many times it should be applied:
incTimes ::(Eqa,Numa)=>a->a->a
incTimes 0n=
n
incTimes times n =
1+(incTimes (times -1) n)
2Because Haskell is built on pure lambda calculus, recursion is implemented in the
language through the Y, or fixed-point combinator. You can read a very good explanation
of that at http://mvanier.livejournal.com/2897.html if you are interested in knowing how it
works.

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 429
Here,𝑡𝑖𝑚𝑒𝑠is a variable representing the number of times
the incrementing function (not called inchere but written as
1 +in the function body) should be applied to the argument
𝑛. If we want to apply it zero times, it will return our 𝑛to us.
Otherwise, the incrementing function will be applied as many
times as we’ve declared:
Prelude> incTimes 10 0
10
Prelude> incTimes 5 0
5
Prelude> incTimes 5 5
10
--does this look familiar?
In a function such as this, the looming threat of unending
recursion is minimized because the number of times to apply
the function is an argument to the function itself, and we’ve
defined a stopping point: when (times - 1) is equal to zero, it
returns 𝑛and that’s all the applications we get.
We can abstract the recursion out of incTimes , too:

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 430
applyTimes ::(Eqa,Numa)=>
a->(b->b)->b->b
applyTimes 0f b=b
applyTimes n f b=f (applyTimes (n -1) f b)
incTimes' ::(Eqa,Numa)=>a->a->a
incTimes' times n =applyTimes times ( +1) n
When we do, we can make the composition more obvious
inapplyTimes :
applyTimes ::(Eqa,Numa)=>
a->(b->b)->b->b
applyTimes 0f b=
b
applyTimes n f b=
f.applyTimes (n -1) f$b
We’re recursively composing our function 𝑓withapplyTimes
(n-1) f however many subtractions it takes to get nto 0!
Intermission: Exercise
Write out the evaluation of the following. It might be a little
less noisy if you do so with the form that didn’t use (.).
applyTimes 5(+1)5

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 431
8.3 Bottom
⊥orbottom is a term used in Haskell to refer to computations
that do not successfully result in a value. The two main vari-
eties of bottom are computations that failed with an error or
those that failed to terminate. In logic, ⊥corresponds to false.
Let us examine a few ways by which we can have bottom in
our programs:
Prelude> let x = x in x
*** Exception: <<loop>>
Here GHCi detected that let x = x in x was never going
to return and short-circuited the never-ending computation.
This is an example of bottom because it was never going to
return a result. Note that if you’re using a Windows com-
puter, this example may freeze your GHCi and not throw an
exception.
Next let’s define a function that will return an exception:
f::Bool->Int
fTrue=error"blah"
fFalse=0
And let’s try that out in GHCi:
Prelude> f False

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 432
0
Prelude> f True
*** Exception: blah
In the first case, when we evaluated f False and got 0, that
didn’t result in a bottom value. But, when we evaluated f True ,
we got an exception which is a means of expressing that a
computation failed. We got an exception because we specified
that this value should return an error. But this, too, is an
example of bottom.
Another example of a bottom would be a partial function.
Let’s consider a rewrite of the previous function:
f::Bool->Int
fFalse=0
This has the same type and returns the same output. What
we’ve done is elided the f True = error "blah" case from the
function definition. This is nota solution to the problem with
the previous function, but it will give us a diﬀerent exception.
We can observe this for ourselves in GHCi:
Prelude> let f :: Bool -> Int; f False = 0
Prelude> f False
0
Prelude> f True
*** Exception: 6:23-33:

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 433
Non-exhaustive patterns in function f
Theerrorvalue is still there, but our language implemen-
tation is making it the fallback case because we didn’t write
atotalfunction, that is, a function which handles all of its in-
puts. Because we failed to define ways to handle all potential
inputs, for example through an “otherwise” case, the previous
function was really:
f::Bool->Int
fFalse=0
f_ =error$"*** Exception: "
++"Non-exhaustive"
++"patterns in function f"
A partial function is one which does not handle all of its
inputs. A total function is one that does. How do we make our
𝑓into a total function? One way is with the use of the datatype
Maybe.
dataMaybea=Nothing |Justa
TheMaybedatatype can take an argument. In the first case,
Nothing , there is no argument; this is our way to say that there
is no result or data from the function without hitting bottom.
The second case, Just a takes an argument and allows us to
return the data we’re wanting. Maybemakes all uses of nilvalues

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 434
and most uses of bottom unnecessary. Here’s how we’d use it
with𝑓:
f::Bool->MaybeInt
fFalse=Just0
f_ =Nothing
Note that the type and both cases all change. Not only do
we replace the errorwith the Nothing value from Maybe, but we
also have to wrap 0 in the Justconstructor from Maybe. If we
don’t do so, we’ll get a type error when we try to load the code,
as you can see:
f::Bool->MaybeInt
fFalse=0
f_ =Nothing
Prelude> :l code/brokenMaybe1.hs
[1 of 1] Compiling Main
code/brokenMaybe1.hs:3:11:
No instance for (Num (Maybe Int))
arising from the literal ‘0’
In the expression: 0
In an equation for ‘f’: f False = 0

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 435
This type error is because, as before, 0 has the type Num a
=> a, so it’s trying to get an instance of NumforMaybe Int . We
can clarify our intent a bit:
f::Bool->MaybeInt
fFalse=0::Int
f_ =Nothing
And then get a better type error in the bargain:
Prelude> :l code/brokenMaybe2.hs
[1 of 1] Compiling Main
code/brokenMaybe2.hs:3:11:
Couldn't match expected type
‘Maybe Int’ with actual type ‘Int’
In the expression: 0 :: Int
In an equation for ‘f’: f False = 0 :: Int
We’ll explain Maybein more detail a bit later.
8.4 Fibonacci numbers
Another classic demonstration of recursion in functional pro-
gramming is a function that calculates the 𝑛th number in a
Fibonacci sequence. The Fibonacci sequence is a sequence
of numbers in which each number is the sum of the previous

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 436
two: 1, 1, 2, 3, 5, 8, 13, 21, 34... and so on. It’s an indefinite
computation that relies on adding two of its own members, so
it’s a perfect candidate for a recursive function. We’re going to
walk through the steps of how we would write such a function
for ourselves to get a better understanding of the reasoning
process.
1.Consider the types
The first thing we’ll consider is the possible type signature
for our function. The Fibonacci sequence only involves
positive whole numbers. The argument to our Fibonacci
function is going to be a positive whole number, because
we’re trying to return the value that is the 𝑛th member of
the Fibonacci sequence. Our result will also be a positive
whole number, since that’s what Fibonacci numbers are.
We would be looking, then, for values that are of the Intor
Integer types. We could use one of those concrete types
or use a typeclass for constrained polymorphism. Specif-
ically, we want a type signature that takes one integral
argument and returns one integral result. So, our type
signature will look something like this:
fibonacci ::Integer ->Integer
-- or
fibonacci ::Integral a=>a->a
2.Consider the base case

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 437
It may sometimes be difficult to determine your base case
up front, but it’s worth thinking about. For one thing,
you do want to ensure that your function will terminate.
For another thing, giving serious consideration to your
base case is a valuable part of understanding how your
function works. Fibonacci numbers are positive integers,
so a reasonable base case is zero. When the recursive
process hits zero, it should terminate.
The Fibonacci sequence is a bit trickier than some, though,
because it needs two base cases. The sequence has to start
oﬀ with two numbers, since two numbers are involved in
computing the next. The next number after zero is 1, and
we add zero to 1 to start the sequence so those will be our
base cases:
fibonacci ::Integral a=>a->a
fibonacci 0=0
fibonacci 1=1
3.Consider the arguments
We’ve already determined that the argument to our func-
tion, the value to which the function is applied, is an inte-
gral number and represents the member of the sequence
we want to be evaluated. That is, we want to pass a value
such as 10 to this function and have it calculate the 10th

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 438
number in the Fibonacci sequence. We only need to have
one variable as a parameter to this function then.
But that argument will also be used as arguments within
the function due to the recursive process. Every Fibonacci
number is the result of adding the preceding two numbers.
So, in addition to a variable 𝑥, we will need to use (x - 1)
and(x - 2) to get both the numbers before our argument.
fibonacci ::Integral a=>a->a
fibonacci 0=0
fibonacci 1=1
fibonacci x=(x-1) (x-2)
-- note: this doesn't work yet.
4.Consider the recursion
All right, now we come to the heart of the matter. In what
way will this function refer to itself and call itself? Look at
what we’ve worked out so far: what needs to happen next
to produce a Fibonacci number? One thing that needs
to happen is that (x - 1) and(x - 2) need to be added
together to produce a result. Try simply adding those two
together and running the function that way.

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 439
fibonacci ::Integral a=>a->a
fibonacci 0=0
fibonacci 1=1
fibonacci x=(x-1)+(x-2)
If you pass the value 6 to that function, what will happen?
Prelude> fibonacci 6
9
Why? Because ((6 - 1) + (6 - 2)) equals 9. But this isn’t how
we calculate Fibonacci numbers! The sixth member of the
Fibonacci sequence is not ((6 - 1) + (6 - 2)). What we want is
to add the fifth member of the Fibonacci sequence to the
fourth member. That result will be the sixth member of
the sequence. We do this by making the function refer to
itself. In this case, we have to specify that both (x - 1) and
(x - 2) are themselves Fibonacci numbers, so we have to
call the function to itself twice.
fibonacci ::Integral a=>a->a
fibonacci 0=0
fibonacci 1=1
fibonacci x=
fibonacci (x -1)+fibonacci (x -2)

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 440
Now, if we apply this function to the value 6, we will get a
diﬀerent result:
Prelude> fibonacci 6
8
Why? Because it evaluates this recursively:
fibonacci 6=fibonacci 5+fibonacci 4
fibonacci 5=fibonacci 4+fibonacci 3
fibonacci 4=fibonacci 3+fibonacci 2
fibonacci 3=fibonacci 2+fibonacci 1
fibonacci 2=fibonacci 1+fibonacci 0
Zero and 1 have been defined as being equal to zero and
1. So here our recursion stops, and it starts adding the
result:
fibonacci 0 + 0
fibonacci 1 + 1
fibonacci 2 + (1 + 0 =) 1
fibonacci 3 + (1 + 1 =) 2

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 441
fibonacci 4 + (1 + 2 =) 3
fibonacci 5 = (2 + 3 =) 5
fibonacci 6 = (3 + 5 =) 8
It can be daunting at first to think how you would write a
recursive function and what it means for a function to call
itself. But as you can see, it’s useful when the function will
make reference to its own results in a repeated fashion.
8.5 Integral division from scratch
Many people learned multiplication by memorizing multi-
plication tables, usually up to 10x10 or 12x12 (dozen). In fact,
one can perform multiplication in terms of addition, repeated
over and over. Similarly, one can define integral division in
terms of subtraction.
Let’s think through our recursive division function one step
at a time. First, let’s consider the types we would want to use for
such a function and see if we can construct a reasonable type
signature. When we divide numbers, we have a numerator
and a denominator. When we evaluate 10 / 5 to get the answer
2, 10 is the numerator, 5 the denominator, and 2 the quotient.
So we have at least three numbers here. So, perhaps a type
likeInteger -> Integer -> Integer would be suitable. You could

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 442
even add some type synonyms to make it more obvious if you
wished:
dividedBy ::Integer ->Integer ->Integer
dividedBy =div
-- changes to
typeNumerator =Integer
typeDenominator =Integer
typeQuotient =Integer
dividedBy ::Numerator
->Denominator
->Quotient
dividedBy =div
Thetypekeyword, instead of the more familiar dataor
newtype , declares a type synonym, or type alias. Those are all
Integer types, but we can give them diﬀerent names to make
them easier for human eyes to distinguish in type signatures.
For this example, we didn’t write out the recursive imple-
mentation of dividedBy we had in mind. As it turns out, when
we write the function, we will want to change the final type sig-
nature a bit, for reasons we’ll see in a minute. Sometimes the

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 443
use of type synonyms can improve the clarity and purpose of
your type signatures, so this is something you’ll see, especially
in more complex code. For our relatively simple function, it
may not be necessary.
Next, let’s think through our base case. The way we divide in
terms of subtraction is by stopping when our result of having
subtracted repeatedly is lower than the divisor. If it divides
evenly, it’ll stop at 0:
Solve 20 divided by 4
-- [1] [2]
-- [1]: Dividend or numerator
-- [2]: Divisor or denominator
-- Result is quotient
20 divided by 4 == 20 - 4, 16
- 4, 12
- 4, 8
- 4, 4
- 4, 0
-- 0 is less than 4, so we stopped.
-- We subtracted 5 times, so 20 / 4 == 5
Otherwise, we’ll have a remainder. Let’s look at a case where
it doesn’t divide evenly:
Solve 25 divided by 4

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 444
25 divided by 4 == 25 - 4, 21
- 4, 17
- 4, 13
- 4, 9
- 4, 5
- 4, 1
-- we stop at 1, because it's less than 4
In the case of 25 divided by 4, we subtracted 4 six times and
had 1 as our remainder. We can generalize this process of di-
viding whole numbers, returning the quotient and remainder,
into a recursive function which does the repeated subtraction
and counting for us. Since we’d like to return the quotient
andthe remainder, we’re going to return the 2-tuple (,)as the
result of our recursive function.
dividedBy ::Integral a=>a->a->(a, a)
dividedBy num denom =go num denom 0
wherego n d count
|n<d=(count, n)
|otherwise =
go (n-d) d (count +1)
We’ve changed the type signature from the one we had
originally worked out, both to make it more polymorphic

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 445
(Integral a => a versusInteger ) and also to return the tuple
instead of just an integer.
Here we used a common Haskell idiom called a gofunction.
This allows us to define a function via a where-clause that can
accept more arguments than the top-level function dividedBy
does. In this case, the top-level function takes two arguments,
numanddenom, but we need a third argument in order to keep
trackofhowmanytimeswedothesubtraction. Thatargument
is called countand is defined with a starting value of zero and
is incremented by 1 every time the otherwise case is invoked.
There are two branches in our gofunction. The first case
is the most specific; when the numerator 𝑛is less than the
denominator 𝑑, the recursion stops and returns a result. It
is not significant that we changed the argument names from
𝑛𝑢𝑚and𝑑𝑒𝑛𝑜𝑚 to𝑛and𝑑. Thegofunction has already been
applied to those arguments in the definition of dividedBy so
the𝑛𝑢𝑚,𝑑𝑒𝑛𝑜𝑚 , and 0 are bound to 𝑛,𝑑, and𝑐𝑜𝑢𝑛𝑡in thewhere
clause.
The result is a tuple of 𝑐𝑜𝑢𝑛𝑡and the last value for 𝑛. This is
our base case that stops the recursion and gives a final result.
Here’s an example of how dividedBy expands but with the
code inlined:

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 446
dividedBy 102
-- first we'll do this the previous way,
-- but we'll keep track of how many
-- times we subtracted.
10divided by 2==
10-2,8(subtracted 1time)
-2,6(subtracted 2times)
-2,4(subtracted 3times)
-2,2(subtracted 4times)
-2,0(subtracted 5times)
Since the final number was 0, there’s no remainder. We
subtracted five times. So 10 / 2 == 5 .
Now we’ll expand the code:
dividedBy 102=
go1020
|10<2= ...
-- false, skip this branch
|otherwise =go (10-2)2(0+1)
-- otherwise is literally the value True
-- so if first branch fails,
-- this always succeeds

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 447
go821
-- 8 isn't < 2, so the otherwise branch
go (8-2)2(1+1)
-- n == 6, d == 2, count == 2
go622
go (6-2)2(2+1)
-- 6 isn't < 2, so the otherwise branch
-- n == 4, d == 2, count == 3
go423
go (4-2)2(3+1)
-- 4 isn't < 2, so the otherwise branch
-- n == 2, d == 2, count == 4
go224
go (2-2)2(4+1)
-- 2 isn't < 2, so the otherwise branch
-- n == 0, d == 2, count == 5

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 448
go025
-- the n < d branch is finally evaluated
-- because 0 < 2 is true
-- n == 0, d == 2, count == 5
|0<2=(5,0)
(5,0)
The result of 𝑐𝑜𝑢𝑛𝑡is the quotient, that is, how many times
you can subtract 2 from 10. In a case where there was a remain-
der, that number would be the final value for your numerator
and would be returned as the remainder.
8.6 Chapter Exercises
Review of types
1.What is the type of [[True, False], [True, True], [False,
True]] ?
a)Bool
b)mostly True
c)[a]
d)[[Bool]]
2.Whichofthefollowinghasthesametypeas [[True, False],
[True, True], [False, True]] ?

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 449
a)[(True, False), (True, True), (False, True)]
b)[[3 == 3], [6 > 5], [3 < 4]]
c)[3 == 3, 6 > 5, 3 < 4]
d)["Bool", "more Bool", "Booly Bool!"]
3.For the following function
func ::[a]->[a]->[a]
funcx y=x++y
which of the following is true?
a)xandymust be of the same type
b)xandymust both be lists
c)ifxis aString thenymust be a String
d)all of the above
4.For the funccode above, which is a valid application of
functo both of its arguments?
a)func "Hello World"
b)func "Hello" "World"
c)func [1, 2, 3] "a, b, c"
d)func ["Hello", "World"]

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 450
Reviewing currying
Given the following definitions, tell us what value results from
further applications.
cattyConny ::String->String->String
cattyConny x y=x++" mrow " ++y
-- fill in the types
flippy=flip cattyConny
appedCatty =cattyConny "woops"
frappe=flippy"haha"
1.What is the value of appedCatty "woohoo!" ? Try to deter-
mine the answer for yourself, then test in the REPL.
2.frappe "1"
3.frappe (appedCatty "2")
4.appedCatty (frappe "blue")
5.cattyConny (frappe "pink")
(cattyConny "green" (appedCatty "blue"))
6.cattyConny (flippy "Pugs" "are") "awesome"

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 451
Recursion
1.Write out the steps for reducing dividedBy 15 2 to its final
answer according to the Haskell code.
2.Write a function that recursively sums all numbers from
1 to n, n being the argument. So that if n was 5, you’d add
1 + 2 + 3 + 4 + 5 to get 15. The type should be (Eq a, Num a)
=> a -> a .
3.Write a function that multiplies two integral numbers
using recursive summation. The type should be (Integral
a) => a -> a -> a .
Fixing dividedBy
Our dividedBy function wasn’t quite ideal. For one thing. It
was a partial function and doesn’t return a result (bottom)
when given a divisor that is 0 or less.
Using the pre-existing divfunction we can see how negative
numbers should be handled:
Prelude> div 10 2
5
Prelude> div 10 (-2)
-5
Prelude> div (-10) (-2)
5

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 452
Prelude> div (-10) (2)
-5
The next issue is how to handle zero. Zero is undefined for
division in math, so we ought to use a datatype that lets us say
there was no sensible result when the user divides by zero. If
you need inspiration, consider using the following datatype
to handle this.
dataDividedResult =
ResultInteger
|DividedByZero
McCarthy 91 function
We’re going to describe a function in English, then in math
notation, then show you what your function should return for
some test inputs. Your task is to write the function in Haskell.
The McCarthy 91 function yields 𝑥−10 when𝑥 > 100 and91
otherwise. The function is recursive.
𝑀𝐶(𝑛) =⎧{
⎨{⎩𝑛−10 if𝑛 >100
𝑀𝐶(𝑀𝐶(𝑛+11)) if𝑛 ≤100
mc91=undefined
You haven’t seen mapyet, but all you need to know right
now is that it applies a function to each member of a list and

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 453
returns the resulting list. It’ll be explained in more detail in
the next chapter.
Prelude> map mc91 [95..110]
[91,91,91,91,91,91,91,92,93,94,95,96,97,98,99,100]
Numbers into words
moduleWordNumber where
importData.List (intersperse )
digitToWord ::Int->String
digitToWord n=undefined
digits::Int->[Int]
digitsn=undefined
wordNumber ::Int->String
wordNumber n=undefined
Hereundefined is a placeholder to show you where you need
to fill in the functions. The nto the right of the function names
is the argument which will be an integer.
Fill in the implementations of the functions above so that
wordNumber returns the English word version of the Int value.

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 454
You will first write a function that turns integers from 0-9 into
their corresponding English words, ”one,” ”two,” and so on.
Then you will write a function that takes the integer, separates
the digits, and returns it as a list of integers. Finally you will
need to apply the first function to the list produced by the sec-
ond function and turn it into a single string with interspersed
hyphens.
We’ve laid out multiple functions for you to consider as you
tackle the problem. You may not need all of them, depend-
ing on how you solve it — these are suggestions. Play with
them and look up their documentation to understand them
in deeper detail.
You will probably find this difficult.
div ::Integral a=>a->a->a
mod ::Integral a=>a->a->a
map ::(a->b)->[a]->[b]
concat ::[[a]]->[a]
intersperse ::a->[a]->[a]
(++) ::[a]->[a]->[a]
(:[]) ::a->[a]
Also consider:
Prelude> div 135 10
13

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 455
Prelude> mod 135 10
5
Prelude> div 13 10
1
Prelude> mod 13 10
3
Here is what your output should look in the REPL when it’s
working:
Prelude> wordNumber 12324546
"one-two-three-two-four-five-four-six"
Prelude>
8.7 Definitions
1.Recursion is a means of computing results that may require
an indefinite amount of work to obtain through the use of
repeated function application. Most recursive functions
that terminate or otherwise do useful work will often have
a case that calls itself and a base case that acts as a backstop
of sorts for the recursion.

CHAPTER 8. FUNCTIONS THAT CALL THEMSELVES 456
-- not recursive
lessOne ::Int->Int
lessOne n=n-1
-- recursive
zero::Int->Int
zero0=0
zeron=zero (n -1)

Chapter 9
Lists
If the doors of perception
were cleansed, everything
would appear to man as it
is - infinite.
William Blake
457

CHAPTER 9. THIS THING AND SOME MORE STUFF 458
9.1 Lists
Lists do double duty in Haskell. The first purpose lists serve
is as a way to refer to and process a collection or plurality of
values. The second is as an infinite series of values, usually
generated by a function, which allows them to act as a stream
datatype.
In this chapter, we will:
•explain list’s datatype and how to pattern match on lists;
•practice many standard library functions for operating
on lists;
•learn about the underlying representations of lists;
•see what that representation means for their evaluation;
•and do a whole bunch of exercises!
9.2 The list datatype
The list datatype in Haskell is defined like this:
data[]a=[]|a:[a]
Here[]is the type constructor for lists as well as the data
constructor for the empty list. The []data constructor is a

CHAPTER 9. THIS THING AND SOME MORE STUFF 459
nullary constructor because it takes no arguments. The second
data constructor, in contrast, has arguments. (:)is an infix
operator usually called ‘cons’ (short for construct ). Here cons
takes a value of type 𝑎and a list of type [a]and evaluates to
[a].
Whereas the list datatype as a whole is a sum type, as we can
tell from the |in the definition, the second data constructor (:)
`cons` is aproduct because it takes two arguments. Remember,
a sum type can be read as an “or” as in the Booldatatype where
you get FalseorTrue. A product is like an “and.” We’re going
to talk more about sum and product types in another chapter,
but for now it will suffice to recognize that a : [a] constructs
a value from two arguments, by adding the 𝑎to the front of
the list [a]. The list datatype is a sum type, though, because
it iseitheran empty list ora single value with more list — not
both.
In English, one can read this as:
data[]a=[]|a:[a]
-- [1] [2] [3] [4] [5] [6]
1.The datatype with the type constructor []
2.takes a single type constructor argument ‘a’
3.at the term level can be constructed via
4.nullary constructor []

CHAPTER 9. THIS THING AND SOME MORE STUFF 460
5.orit can be constructed by
6.data constructor (:)which is a product of a value of the
typeawe mentioned in the type constructor anda value
of type [a], that is, “more list.”
The cons constructor (:)is an infix data constructor and
goes between the two arguments 𝑎and[a]that it accepts. Since
it takes two arguments, it is a product of those two arguments,
like the tuple type (a, b). Unlike a tuple, however, this con-
structor is recursive because it mentions its own type [a]as
one of the members of the product.
If you’re an experienced programmer or took a CS class at
some point, you may be familiar with singly-linked lists. This
is a fair description of the list datatype in Haskell, although
average case performance in some situations changes due
to nonstrict evaluation; however, it can contain infinite data
which makes it also work as a stream datatype, but one that has
the option of ending the stream with the []data constructor.
9.3 Pattern matching on lists
We know we can pattern match on data constructors, and the
data constructors for lists are no exceptions. Here we match
on the first argument to the infix (:)constructor, ignoring the
rest of the list, and return that value:

CHAPTER 9. THIS THING AND SOME MORE STUFF 461
Prelude> let myHead (x : _) = x
Prelude> :t myHead
myHead :: [t] -> t
Prelude> myHead [1, 2, 3]
1
We can do the opposite as well:
Prelude> let myTail (_ : xs) = xs
Prelude> :t myTail
myTail :: [t] -> [t]
Prelude> myTail [1, 2, 3]
[2,3]
We do need to be careful with functions like these. Neither
myHead normyTail has a case to handle an empty list — if we try
to pass them an empty list as an argument, they can’t pattern
match:
Prelude> myHead []
*** Exception:
Non-exhaustive patterns
in function myHead
Prelude> myTail []
*** Exception:
Non-exhaustive patterns

CHAPTER 9. THIS THING AND SOME MORE STUFF 462
in function myTail
The problem is that the type [a] -> a ofmyHead is deceptive
because the [a]type doesn’t guarantee that it’ll have an 𝑎value.
It’s not guaranteed that the list will have at least one value, so
myTail can fail as well. One possibility is putting in a base case:
myTail ::[a]->[a]
myTail[] =[]
myTail(_:xs)=xs
In that case, our function now evaluates like this:
Prelude> myTail [1..5]
[2,3,4,5]
Prelude> myTail []
[]
Using Maybe A better way to handle this situation is with a
datatype called Maybe. We’ll save a full treatment of Maybefor
a later chapter, but this should give you some idea of how it
works. The idea here is that it makes your failure case explicit,
and as programs get longer and more complex that can be
quite useful.
Let’s try an example using MaybewithmyTail. Instead of
having a base case that returns an empty list, the function

CHAPTER 9. THIS THING AND SOME MORE STUFF 463
written with Maybewould return a result of Nothing . As we can
see below, the Maybedatatype has two potential values, Nothing
orJust a :
Prelude> :info Maybe
data Maybe a = Nothing | Just a
Rewriting myTail to useMaybeis fairly straightforward:
safeTail ::[a]->Maybe[a]
safeTail []=Nothing
safeTail (x:[])=Nothing
safeTail (_:xs)=Justxs
Notice that our function is still pattern matching on the list.
We’ve made the second base case safeTail (x:[]) = Nothing to
reflect the fact that if your list has only one value inside it, its
tail is an empty list. If you leave this case out, then this function
will return Just [] for lists that have only a head value. Take
a few minutes to play around with this and see how it works.
Then see if you can rewrite the myHead function above using
Maybe.
Later in the book, we’ll also cover a datatype called NonEmpty
which always has at least one value and avoids the empty list
problem.

CHAPTER 9. THIS THING AND SOME MORE STUFF 464
9.4 List’s syntactic sugar
Haskell has some syntactic sugar to accommodate the use of
lists, so that you can write:
Prelude> [1, 2, 3] ++ [4]
[1, 2, 3, 4]
Rather than:
Prelude> (1 : 2 : 3 : []) ++ 4 : []
[1,2,3,4]
The syntactic sugar is here to allow building lists in terms
of the successive applications of ‘cons‘ (:)to a value without
having to tediously type it all out.
When we talk about lists, we often talk about them in terms
of “cons cells” and spines. The syntactic sugar obscures this
underlying construction, but looking at the desugared ver-
sion above may make it more clear. The cons cells are the
list datatype’s second data constructor, a : [a] , the result of
recursively prepending a value to “more list.” The cons cell is
aconceptual space that values may inhabit.
The spine is the connective structure that holds the cons
cells together and in place. As we will soon see, this structure
nests the cons cells rather than ordering them in a right-to-
left row. Because diﬀerent functions may treat the spine and

CHAPTER 9. THIS THING AND SOME MORE STUFF 465
the cons cells diﬀerently, it is important to understand this
underlying structure.
9.5 Using ranges to construct lists
There are several ways we can construct lists. One of the
simplest is with ranges. The basic syntax is to make a list that
has the element you want to start the list from followed by
two dots followed by the value you want as the final element
in the list. Here are some examples using the range syntax,
followed by the desugared equivalents using functions from
theEnumtypeclass:
Prelude> [1..10]
[1,2,3,4,5,6,7,8,9,10]
Prelude> enumFromTo 1 10
[1,2,3,4,5,6,7,8,9,10]
Prelude> [1,2..10]
[1,2,3,4,5,6,7,8,9,10]
Prelude> enumFromThenTo 1 2 10
[1,2,3,4,5,6,7,8,9,10]
Prelude> [1,3..10]
[1,3,5,7,9]
Prelude> enumFromThenTo 1 3 10

CHAPTER 9. THIS THING AND SOME MORE STUFF 466
[1,3,5,7,9]
Prelude> [2,4..10]
[2,4,6,8,10]
Prelude> enumFromThenTo 2 4 10
[2,4,6,8,10]
Prelude> ['t'..'z']
"tuvwxyz"
Prelude> enumFromTo 't' 'z'
"tuvwxyz"
The types of the functions underlying the range syntax are:
enumFrom ::Enuma
=>a->[a]
enumFromThen ::Enuma
=>a->a->[a]
enumFromTo ::Enuma
=>a->a->[a]
enumFromThenTo ::Enuma
=>a->a->a->[a]
All of these functions require that the type being “ranged”
have an instance of the Enumtypeclass. The first two functions,

CHAPTER 9. THIS THING AND SOME MORE STUFF 467
enumFrom andenumFromThen , generate lists of indefinite, possibly
infinite, length. For it to create an infinitely long list, you
must be ranging over a type that has no upper bound in its
enumeration. Integer is such a type. You can make Integer
values as large as you have memory to describe.
Be aware that enumFromTo must have its first argument be
lower than the second argument:
Prelude> enumFromTo 3 1
[]
Prelude> enumFromTo 1 3
[1,2,3]
Otherwise you’ll get an empty list.
Exercise: EnumFromTo
Some things you’ll want to know about the Enumtypeclass:
Prelude> :info Enum
class Enum a where
succ :: a -> a
pred :: a -> a
toEnum :: Int -> a
fromEnum :: a -> Int
enumFrom :: a -> [a]
enumFromThen :: a -> a -> [a]

CHAPTER 9. THIS THING AND SOME MORE STUFF 468
enumFromTo :: a -> a -> [a]
enumFromThenTo :: a -> a -> a -> [a]
Prelude> succ 0
1
Prelude> succ 1
2
Prelude> succ 'a'
'b'
Write your own enumFromTo definitions for the types pro-
vided. Do not use range syntax to do so. It should return the
same results as if you did [start..stop] . Replace the undefined ,
an value which results in an error when evaluated, with your
own definition.

CHAPTER 9. THIS THING AND SOME MORE STUFF 469
eftBool ::Bool->Bool->[Bool]
eftBool =undefined
eftOrd::Ordering
->Ordering
->[Ordering ]
eftOrd=undefined
eftInt::Int->Int->[Int]
eftInt=undefined
eftChar ::Char->Char->[Char]
eftChar =undefined
9.6 Extracting portions of lists
In this section, we’ll take a look at some useful functions for
extracting portions of a list and dividing lists into parts. The
first three functions have similar type signatures, taking Int
arguments and applying them to a list argument:
take::Int->[a]->[a]
drop::Int->[a]->[a]
splitAt ::Int->[a]->([a], [a])

CHAPTER 9. THIS THING AND SOME MORE STUFF 470
We have seen examples of some of the above functions in
previous chapters, but they are common and useful enough
they deserve review.
Thetakefunction takes the specified number of elements
out of a list and returns a list containing just those elements.
As you can see it takes one argument that is an Intand applies
that to a list argument. Here’s how it works:
Prelude> take 7 ['a'..'z']
"abcdefg"
Prelude> take 3 [1..10]
[1,2,3]
Prelude> take 3 []
[]
Notice that when we pass it an empty list as an argument,
it returns an empty list. These lists use the syntactic sugar
for building lists with ranges. We can also use takewith a list-
building function, such as enumFrom . Reminder: enumFrom can
generate an infinite list if the type of list inhabitants is, such
asInteger , an infinite set. But as long as we’re only taking a
certain number of elements from that, it won’t generate an
infinite list:
Prelude> take 10 (enumFrom 10)

CHAPTER 9. THIS THING AND SOME MORE STUFF 471
[10,11,12,13,14,15,16,17,18,19]
Thedropfunction is similar to takebut drops the specified
number of elements oﬀ the beginning of the list. Again, we
can use it with ranges or list-building functions:
Prelude> drop 5 [1..10]
[6,7,8,9,10]
Prelude> drop 8 ['a'..'z']
"ijklmnopqrstuvwxyz"
Prelude> drop 4 []
[]
Prelude> drop 2 (enumFromTo 10 20)
[12,13,14,15,16,17,18,19,20]
ThesplitAt function cuts a list into two parts at the element
specified by the Intand makes a tuple of two lists:
Prelude> splitAt 5 [1..10]
([1,2,3,4,5],[6,7,8,9,10])
Prelude> splitAt 10 ['a'..'z']
("abcdefghij","klmnopqrstuvwxyz")

CHAPTER 9. THIS THING AND SOME MORE STUFF 472
Prelude> splitAt 5 []
([],[])
Prelude> splitAt 3 (enumFromTo 5 15)
([5,6,7],[8,9,10,11,12,13,14,15])
The higher-order functions takeWhile anddropWhile are a bit
diﬀerent, as you can see from the type signatures:
takeWhile ::(a->Bool)->[a]->[a]
dropWhile ::(a->Bool)->[a]->[a]
So these take and drop items out of a list that meet some
condition, as we can see from the presence of Bool.takeWhile
will take elements out of a list that meet that condition and
then stop when it meets the first element that doesn’t satisfy
the condition:
Take the elements that are less than 3:
Prelude> takeWhile (<3) [1..10]
[1,2]
Take the elements that are less than 8:
Prelude> takeWhile (<8) (enumFromTo 5 15)
[5,6,7]

CHAPTER 9. THIS THING AND SOME MORE STUFF 473
The next example returns an empty list because it stops
taking as soon as the condition isn’t met, which in this case is
the first element:
Prelude> takeWhile (>6) [1..10]
[]
In the final example above, why does it only return a single
𝑎?
Prelude> takeWhile (=='a') "abracadabra"
"a"
Now, we’ll look at dropWhile whose behavior is probably
predictable based on the functions and type signatures we’ve
already seen in this section. We will use the same arguments
as we used with takeWhile so the diﬀerence between them is
easy to see:
Prelude> dropWhile (<3) [1..10]
[3,4,5,6,7,8,9,10]
Prelude> dropWhile (<8) (enumFromTo 5 15)
[8,9,10,11,12,13,14,15]
Prelude> dropWhile (>6) [1..10]
[1,2,3,4,5,6,7,8,9,10]

CHAPTER 9. THIS THING AND SOME MORE STUFF 474
Prelude> dropWhile (=='a') "abracadabra"
"bracadabra"
Exercises: Thy Fearful Symmetry
1.UsingtakeWhile anddropWhile , write a function that takes a
string and returns a list of strings, using spaces to separate
the elements of the string into words, as in the following
sample:
Prelude> myWords "sheryl wants fun"
["wallfish", "wants", "fun"]
2.Next, write a function that takes a string and returns a list
of strings, using newline separators to break up the string
as in the following (your job is to fill in the undefined
function):

CHAPTER 9. THIS THING AND SOME MORE STUFF 475
modulePoemLines where
firstSen ="Tyger Tyger, burning bright \n"
secondSen ="In the forests of the night \n"
thirdSen ="What immortal hand or eye \n"
fourthSen ="Could frame thy fearful \
\symmetry?"
sentences =firstSen ++secondSen
++thirdSen ++fourthSen
-- putStrLn sentences -- should print
-- Tyger Tyger, burning bright
-- In the forests of the night
-- What immortal hand or eye
-- Could frame thy fearful symmetry?
-- Implement this
myLines ::String->[String]
myLines =undefined

CHAPTER 9. THIS THING AND SOME MORE STUFF 476
-- What we want 'myLines sentences'
-- to equal
shouldEqual =
["Tyger Tyger, burning bright"
,"In the forests of the night"
,"What immortal hand or eye"
,"Could frame thy fearful symmetry?"
]
-- The main function here is a small test
-- to ensure you've written your function
-- correctly.
main::IO()
main=
print$
"Are they equal? "
++show (myLines sentences
==shouldEqual)
3.Now let’s look at what those two functions have in com-
mon. Try writing a new function that parameterizes the
character you’re breaking the string argument on and
rewrite myWords andmyLines using it.

CHAPTER 9. THIS THING AND SOME MORE STUFF 477
9.7 List comprehensions
List comprehensions are a means of generating a new list
from a list or lists. They come directly from the concept of
set comprehensions in mathematics, including similar syntax.
They must have at least one list, called the generator, that gives
the input for the comprehension, that is, provides the set of
items from which the new list will be constructed. They may
have conditions to determine which elements are drawn from
the list and/or functions applied to those elements.
Let’s start by looking at a very simple example:
[ x^2|x<-[1..10]]
-- [1] [2] [ 3 ]
1.This is the output function that will apply to the members
of the list we indicate.
2.The pipe here designates the separation between the out-
put function and the input.
3.This is the input set: a generator list and a variable that
represents the elements that will be drawn from that list.
This says, “from a list of numbers from 1-10, take (<-)
each element as an input to the output function.”
In plain English, that list comprehension will produce a
new list that includes the square of every number from 1 to 10:

CHAPTER 9. THIS THING AND SOME MORE STUFF 478
Prelude> [x^2 | x <- [1..10]]
[1,4,9,16,25,36,49,64,81,100]
Now we’ll look at some ways to vary what elements are
drawn from the generator list(s).
Adding predicates
List comprehensions can optionally take predicates that limit
the elements drawn from the generator list. The predicates
must evaluate to Boolvalues, as in other condition-placing
function types we’ve looked at (for example, guards). Then the
items drawn from the list and passed to the output function
will only be those that met the Truecase in the predicate.
For example, let’s say we wanted a similar list comprehen-
sion as we used above, but this time we wanted our new list to
contain the squares of only the even numbers while ignoring
the odds. In that case, we put a comma after our generator list
and add the condition:
Prelude> [x^2 | x <- [1..10], rem x 2 == 0]
[4,16,36,64,100]
Here we’ve specified that the only elements to take from
the generator list as 𝑥are those that, when divided by 2, have
a remainder of zero — that is, even numbers.

CHAPTER 9. THIS THING AND SOME MORE STUFF 479
We can also write list comprehensions that have multiple
generators. One thing to note is that the rightmost generator
will be exhausted first, then the second rightmost, and so on.
For example, let’s say you wanted to make a list of 𝑥to
the𝑦power, instead of squaring all of them as we did above.
Separate the two inputs with a comma as below:
Prelude> [x^y | x <- [1..5], y <- [2, 3]]
[1,1,4,8,9,27,16,64,25,125]
When we examine the resulting list, we see that it is each
𝑥value first to the second power and then to the third power,
followed by the next 𝑥value to the second and then to the
third and so on, ending with the result of 5^2and5^3. We are
applying the function to each possible pairing of values from
the two lists we’re binding values out of. It begins by trying
to get a value out of the leftmost generator, from which we’re
getting 𝑥.
We could put a condition on that, too. Let’s say we only
want to return the list of values that are less than 200. We add
another comma and write our predicate:
Prelude> :{
Prelude| [x ^ y |
Prelude| x <- [1..10],
Prelude| y <- [2, 3],
Prelude| x ^ y < 200]

CHAPTER 9. THIS THING AND SOME MORE STUFF 480
Prelude| :}
[1,1,4,8,9,27,16,64,25,125,36,49,64,81,100]
We can use multiple generators to turn two lists into a list
of tuples containing those elements as well. The generator
lists don’t even have to be of the same length or, due to the
nature of the tuple type, even the same type:
Prelude> :{
Prelude| [(x, y) |
Prelude| x <- [1, 2, 3],
Prelude| y <- [6, 7]]
Prelude| :}
[(1,6),(1,7),(2,6),(2,7),(3,6),(3,7)]
Prelude> :{
Prelude| [(x, y) |
Prelude| x <- [1, 2, 3],
Prelude| y <- ['a', 'b']]
Prelude| :}
[(1,'a'),(1,'b'),(2,'a'),
(2,'b'),(3,'a'),(3,'b')]
Again the pattern is that it generates every possible tuple
for the first 𝑥value, then it moves to the next 𝑥value and so
on.

CHAPTER 9. THIS THING AND SOME MORE STUFF 481
Recall that the first list comprehension we looked at gen-
erated a list of all the values of 𝑥^2when𝑥is a number from
1-10. Let’s say you wanted to use that list in another list com-
prehension. First, you’d want to give that list a name. Let’s call
itmySqr:
Prelude> let mySqr = [x^2 | x <- [1..10]]
Now we can use that list as the generator for another list
comprehension. Here, we will limit our input values to those
that are less than 4 for the sake of brevity:
Prelude> let mySqr = [x^2 | x <- [1..10]]
Prelude> :{
Prelude| [(x, y) |
Prelude| x <- mySqr,
Prelude| y <- [1..3], x < 4]
Prelude| :}
[(1,1),(1,2),(1,3)]
Exercises: Comprehend Thy Lists
Take a look at the following functions, figure what you think
the output lists will be, and then run them in your REPL to
verify (note that you will need the mySqrlist from above in
scope to do this):
[x | x <- mySqr, rem x 2 == 0]

CHAPTER 9. THIS THING AND SOME MORE STUFF 482
[(x, y) | x <- mySqr,
y <- mySqr,
x < 50, y > 50]
take 5 [ (x, y) | x <- mySqr,
y <- mySqr,
x < 50, y > 50 ]
List comprehensions with Strings
It’s worth remembering that strings are lists, so list comprehen-
sions can also be used with strings. We’re going to introduce
a standard function called elem1that tells you whether an el-
ement is in a list or not. It evaluates to a Boolvalue, so it is
useful as a predicate in list comprehensions:
Prelude> :t elem
elem :: Eq a => a -> [a] -> Bool
Prelude> elem 'a' "abracadabra"
True
Prelude> elem 'a' "Julie"
False
In the first case, ‘a’ is an element of “abracadabra” so that
evaluates to True, but in the second case, there is no ‘a’ in
1Reminder, pretend Foldable in the type of elemmeans it’s a list until we cover Foldable
later.

CHAPTER 9. THIS THING AND SOME MORE STUFF 483
“Julie” so we get a Falseresult. As you can see from the type
signature, elemdoesn’t only work with characters and strings,
but that’s what we’ll use it for here. Let’s see if we can write a
list comprehension to remove all the lowercase letters from
a string. Here our condition is that we only want to take 𝑥
from our generator list when it meets the condition that it is
an element of the list of capital letters:
Prelude> :{
Prelude| [x |
Prelude| x <- "Three Letter Acronym",
Prelude| elem x ['A'..'Z']]
Prelude| :}
"TLA"
Let’s see if we can now generalize this into an acronym
generator that will accept diﬀerent strings as inputs, instead of
forcing us to rewrite the whole list comprehension for every
string we might want to feed it. We will do this by naming
a function that will take one argument and use that as the
generator string for our list comprehension. So the function
argument and the generator string will need to be the same
thing:
Prelude> :{
Prelude| let acro xs =
Prelude| [x | x <- xs,

CHAPTER 9. THIS THING AND SOME MORE STUFF 484
Prelude| elem x ['A'..'Z']]
Prelude| :}
We use𝑥𝑠for our function argument to indicate to ourselves
that it’s a list, that the 𝑥is plural. It doesn’t have to be; you
could use a diﬀerent variable there and obtain the same result.
It is idiomatic to use a “plural” variable for list arguments, but
it is not necessary.
All right, so we have our acrofunction with which we can
generate acronyms from any string:
Prelude> acro "Self Contained Underwater Breathing Apparatus"
"SCUBA"
Prelude> acro "National Aeronautics and Space Administration"
"NASA"
Given the above, what do you think this function would do:
Prelude> let myString xs = [x | x <- xs, elem x "aeiou"]
Exercises: Square Cube
Given the following:
Prelude> let mySqr = [x^2 | x <- [1..5]]
Prelude> let myCube = [y^3 | y <- [1..5]]
1.First write an expression that will make tuples of the out-
puts of mySqrandmyCube .

CHAPTER 9. THIS THING AND SOME MORE STUFF 485
2.Now alter that expression so that it only uses the x and y
values that are less than 50.
3.Apply another function to that list comprehension to
determine how many tuples inhabit your output list.
9.8 Spines and nonstrict evaluation
As we have seen, lists are a recursive series of cons cells a : [a]
terminated by the empty list [], but we want a way to visu-
alize this structure in order to understand the ways lists get
processed. When we talk about data structures in Haskell, par-
ticularly lists, sequences, and trees, we talk about them having
aspine. This is the connective structure that ties the collection
of values together. In the case of a list, the spine is usually tex-
tually represented by the recursive cons (:)operators. Given
the data: [1, 2, 3] , we get a list that looks like:
1 : 2 : 3 : []
or
1 : (2 : (3 : []))
:
/ \
1 :
/ \

CHAPTER 9. THIS THING AND SOME MORE STUFF 486
2 :
/ \
3 []
The problem with the 1 : (2 : (3 : [])) representation we
used earlier is that it makes it seem like the value 1 exists
“before” the cons (:)cell that contains it, but actually, the cons
cells contain the values. Because of this and the way nonstrict
evaluation works, you can evaluate cons cells independently
of what they contain. It is possible to evaluate only the spine of
the list without evaluating individual values. It is also possible
to evaluate only part of the spine of a list and not the rest of it.
Evaluation of the list in this representation proceeds down
the spine. However, constructing the list (when that is neces-
sary) proceeds upthe spine. In the example above, then, we
start with an infix operator, evaluate the arguments 1 and a
new cons cell, and proceed downward to the 3 and empty list.
But when we need to build the list, to print it in the REPL for
example, it proceeds from the bottom of the list up the spine,
first putting the 3 into the empty list, then adding the 2 to
the front of that list, then, finally, putting the 1 in the front of
that. Because Haskell’s evaluation is nonstrict, the list isn’t con-
structed until it’s consumed — indeed, nothing is evaluated
until it must be. Until it’s consumed or you force strictness
in some way, there are a series of placeholders as a blueprint

CHAPTER 9. THIS THING AND SOME MORE STUFF 487
of the list that can be constructed when it’s needed. We’ll talk
more about nonstrictness soon.
We’regoingtobring ⊥orbottom backintheformof undefined
in order to demonstrate some of the eﬀects of nonstrict evalu-
ation. Here we’re going to use _to syntactically signify values
we are ignoring and not evaluating. The underscores repre-
sent the values contained by the cons cells. The spine is the
recursive series of cons constructors signified by (:)as you
can see below:
: <------|
/ \ |
_ : <----| This is the "spine"
/ \ |
_ : <--|
/ \
_ []
You’ll see the term ‘spine’ used in reference to data struc-
tures, such as trees, that aren’t lists. In the case of a list, the
spine is a linear succession of one cons cell wrapping another
cons cell. With data structures like trees, which we will cover
later, you’ll see that the spine can be nodes that contain 2 or
more nodes.

CHAPTER 9. THIS THING AND SOME MORE STUFF 488
Using GHCi’s :sprint command
We can use a special command in GHCi called sprint to print
variables and see what has been evaluated already, with the un-
derscore representing expressions that haven’t been evaluated
yet.
A warning : We always encourage you to experiment and
explore for yourself after seeing the examples in this book, but
:sprint has some behavioral quirks that can be a bit frustrating.
GHC Haskell has some opportunistic optimizations which
introduce strictness to make code faster when it won’t change
how your code evaluates. Additionally polymorphism means
values like Num a => a are really waiting for a sort of argument
which will make it concrete (this will be covered in more detail
in a later chapter). To avoid this, you have to assign a more
concrete type such as IntorDouble, otherwise it stays uneval-
uated,_, in:sprint ’s output. If you can keep these caveats to
:sprint ’s behavior in mind, it can be useful. Otherwise if you
find it confusing, don’t sweat it and wait for us to elaborate
more deeply in the chapter on nonstrictness.
Let’s define a list using enumFromTo , which is tantamount to
using syntax like ['a'..'z'] , then ask for the state of blahwith
respect to whether it has been evaluated:
Prelude> let blah = enumFromTo 'a' 'z'
Prelude> :sprint blah

CHAPTER 9. THIS THING AND SOME MORE STUFF 489
blah = _
Theblah = _ indicates that blahis totally unevaluated.
Next we’ll take one value from blahand then evaluate it by
forcing GHCi to print the expression:
Prelude> take 1 blah
"a"
Prelude> :sprint blah
blah = 'a' : _
So we’ve evaluated a cons cell :and the first value 'a'.
Then we take two values and print them — which forces
evaluation of the second cons cell and the second value:
Prelude> take 2 blah
"ab"
Prelude> :sprint blah
blah = 'a' : 'b' : _
Assuming this is a contiguous GHCi session, the first cons
cell and value were already forced.
We can keep going with this, evaluating the list one value
at a time:
Prelude> take 3 blah
"abc"
Prelude> :sprint blah

CHAPTER 9. THIS THING AND SOME MORE STUFF 490
blah = 'a' : 'b' : 'c' : _
Thelength function is only strict in the spine, meaning it
only forces evaluation of the spine of a list, not the values,
something we can see if we try to find the length of a list of
undefined values. But when we use length onblah,:sprint will
behave as though we had forced evaluation of the values as
well:
Prelude> length blah
26
Prelude> :sprint blah
blah = "abcdefghijklmnopqrstuvwxyz"
That the individual characters were shown as evaluated
and not exclusively the spine after getting the length of blahis
one of the unfortunate aforementioned quirks of how GHCi
evaluates code.
Spines are evaluated independently of values
Values in Haskell get reduced to weak head normal form by
default. By ‘normal form’ we mean that the expression is fully
evaluated. ‘Weak head normal form’ means the expression is
only evaluated as far as is necessary to reach a data constructor.

CHAPTER 9. THIS THING AND SOME MORE STUFF 491
Weak head normal form (WHNF) is a larger set and con-
tains both the possibility that the expression is fully evalu-
ated (normal form) and the possibility that the expression has
been evaluated to the point of arriving at a data constructor
or lambda awaiting an argument. For an expression in weak
head normal form, further evaluation may be possible once
another argument is provided. If no further inputs are pos-
sible, then it is still in WHNF but also in normal form (NF).
We’re going to explain this more fully later in the book in the
chapter on nonstrictness when we show you how call-by-need
works and the implications for Haskell. For now, we’ll look at
a few examples to get a sense for what might be going on.
Below we list some expressions and whether they are in
WHNF, NF, both, or neither:
(1,2)-- WHNF & NF
This first example is in normal form and is fully evaluated.
Anything in normal form is by definition also in weak head
normal form, because weak head is an expression which is
evaluated up to at least the first data constructor. Normal
form exceeds that by requiring that all subexpressions be fully
evaluated. Here the components of the value are the tuple
data constructor and the values 1 and 2.
(1,1+1)

CHAPTER 9. THIS THING AND SOME MORE STUFF 492
This example is in WHNF, but not NF. The (+)applied to
its arguments could be evaluated but hasn’t been yet.
\x->x*10-- WHNF & NF
This anonymous function is in normal form because while
(*)has been applied to two arguments of a sort, it cannot be
reduced further until the outer x -> ... has been applied.
With nothing further to reduce, it is in normal form.
"Papu"++"chon"
This string concatenation is in neither WHNF nor NF, this
is because the outermost component of the expression is a
function, (++), whose arguments are fully applied but it hasn’t
been evaluated. Whereas, the following would be in WHNF
but not NF:
(1,"Papu"++"chon")
When we define a list and define all its values, it is in NF
and all its values are known. There’s nothing left to evaluate
at that point, such as in the following example:
Prelude> let num :: [Int]; num = [1, 2, 3]
Prelude> :sprint num
num = [1,2,3]

CHAPTER 9. THIS THING AND SOME MORE STUFF 493
We can also construct a list through ranges or functions.
In this case, the list is in WHNF but not NF. The compiler
only evaluates the head or first node of the graph, but just the
cons constructor, not the value or rest of the list it contains.
We know there’s a value of type 𝑎in the cons cell we haven’t
evaluated and a “rest of list” which might either be the empty
list[]which ends the list or another cons cell — we don’t know
which because we haven’t evaluated the next [a]value yet. We
saw that above in the :sprint section, and you can see that
evaluation of the first values does not force evaluation of the
rest of the list:
Prelude> let myNum :: [Int]; myNum = [1..10]
Prelude> :sprint myNum
myNum = _
Prelude> take 2 myNum
[1,2]
Prelude> :sprint myNum
myNum = 1 : 2 : _
This is an example of WHNF evaluation. It’s weak head
normal form because the list has to be constructed by the
range and it’s only going to evaluate as far as it has to. With
take 2, we only need to evaluate the first two cons cells and
the values they contain, which is why when we used :sprint
we only saw 1 : 2 : _ . Evaluating to normal form would’ve

CHAPTER 9. THIS THING AND SOME MORE STUFF 494
meant recursing through the entire list, forcing not only the
entire spine but also the values each cons cell contained.
In these tree representations, evaluation or consumption of
the list goes downthe spine. The following is a representation
of a list that isn’t spine strict and is awaiting something to force
the evaluation:
:
/ \
_ _
By default, it stops here and never evaluates even the first
cons cell unless it’s forced to, as we saw.
However, functions that are spine strict can force complete
evaluation of the spine of the list even if they don’t force eval-
uation of each value. Pattern matching is strict by default, so
pattern matching on cons cells can mean forcing spine strict-
ness if your function doesn’t stop recursing the list. It can
evaluate the spine only or the spine as well as the values that
inhabit each cons cell, depending on context.
On the other hand, length is strict in the spine but not the
values. If we defined a list such as [1, 2, 3] , usinglength on it
would force evaluation of the entire spine without accompa-
nying strictness in the values:
:
/ \

CHAPTER 9. THIS THING AND SOME MORE STUFF 495
_ :
/ \
_ :
/ \
_ []
We can see this if we use length but make one of the values
bottom with the undefined value, and see what happens:
Prelude> let x = [1, undefined, 3]
Prelude> length x
3
The first and third values in the list were numbers, but the
second value was undefined andlength didn’t make it crash.
Why? Because length measures the length of a list, which only
requires recursing the spine and counting how many cons cells
there are. We could define our own length function ourselves
like so:
-- *Not* identical to the length
-- function in Prelude
length::[a]->Integer
length[]=0
length(_:xs)=1+length xs

CHAPTER 9. THIS THING AND SOME MORE STUFF 496
One thing to note is that we use _to ignore the values in our
arguments or that are part of a pattern match. In this case, we
pattern-matched on the (:)data constructor, but wanted to
ignore the value which is the first argument. However, it’s not
a mere convention to bind references we don’t care about on
the left-hand side to _. You can’t bind arguments to the name
”_”; it’s part of the language. This is partly so the compiler
knows for a certainty you won’t ever evaluate something in
that particular case. Currently, if you try using _on the right-
hand side in the definition, it’ll think you’re trying to refer to
a hole.
We’re only forcing the (:)constructors and the []at the
end in order to count the number of values contained by the
list:
: <-|
/ \ |
|-> _ : <-|
| / \ | These got evaluated (forced)
|-> _ : <-|
| / \ |
|-> _ [] <-|
|
| These did not
However, length will throw an error on a bottom value if
part of the spine itself is bottom:

CHAPTER 9. THIS THING AND SOME MORE STUFF 497
Prelude> let x = [1] ++ undefined ++ [3]
Prelude> x
[1*** Exception: Prelude.undefined
Prelude> length x
*** Exception: Prelude.undefined
Printing the list fails, although it gets as far as printing the
first[and the first value, and attempting to get the length also
fails because it can’t count undefined spine values.
It’s possible to write functions which will force both the
spine and the values. sumis an example because in order to
return a result at all, it must return the sum of all values in the
list.
We’ll write our own sumfunction for the sake of demonstra-
tion:
mySum::Numa=>[a]->a
mySum[]=0
mySum(x:xs)=x+mySum xs
First, the +operator is strict in both of its arguments, so that
will force evaluation of the values and the mySum xs . Therefore
mySumwill keep recursing until it hits the empty list and must
stop. Then it will start going back up the spine of the list,
summing the inhabitants as it goes. It looks something like
this (the zero represents our empty list):

CHAPTER 9. THIS THING AND SOME MORE STUFF 498
Prelude> mySum [1..5]
1 + (2 + (3 + (4 + (5 + 0))))
1 + (2 + (3 + (4 + 5)))
1 + (2 + (3 + 9))
1 + (2 + 12)
1 + 14
15
We will be returning to this topic at various points in the
book because developing intuition for Haskell’s evaluation
strategies takes time and practice. If you don’t feel like you
fully understand it at this point, that’s OK. It’s a complex topic,
and it’s better to approach it in stages.
Exercises: Bottom Madness
Will it blow up?
Will the following expressions return a value or be ⊥?
1.[x^y|x<-[1..5], y<-[2, undefined]]
2.take1$
[x^y|x<-[1..5], y<-[2, undefined]]
3.sum[1, undefined, 3]
4.length[1,2, undefined]

CHAPTER 9. THIS THING AND SOME MORE STUFF 499
5.length$[1,2,3]++undefined
6.take1$filter even [ 1,2,3, undefined]
7.take1$filter even [ 1,3, undefined]
8.take1$filter odd [ 1,3, undefined]
9.take2$filter odd [ 1,3, undefined]
10.take3$filter odd [ 1,3, undefined]
Intermission: Is it in normal form?
For each expression below, determine whether it’s in:
1.normal form, which implies weak head normal form;
2.weak head normal form only; or,
3.neither.
Remember that an expression cannot be in normal form or
weak head normal form if the outermost part of the expression
isn’t a data constructor. It can’t be in normal form if any part
of the expression is unevaluated.
1.[1,2,3,4,5]
2.1:2:3:4:_

CHAPTER 9. THIS THING AND SOME MORE STUFF 500
3.enumFromTo 110
4.length[1,2,3,4,5]
5.sum(enumFromTo 110)
6.['a'..'m']++['n'..'z']
7.(_,'b')
9.9 Transforming lists of values
We have already seen how we can make recursive functions
with self-referential expressions. It’s a useful tool and a core
part of the logic of Haskell. In truth, in part because Haskell
uses nonstrict evaluation, we tend to use higher-order func-
tions for transforming data rather than manually recursing
over and over.
For example, one common thing you would want to do is
return a list with a function applied uniformly to all values
within the list. To do so, you need a function that is inherently
recursive and can apply that function to each member of the
list. For this purpose we can use either the maporfmapfunctions.
mapcan only be used with [].fmapis defined in a typeclass
named Functor and can be applied to data other than lists. We
will learn more about Functor later; for now, we’ll focus on the
list usage. Here are some examples using mapandfmap:

CHAPTER 9. THIS THING AND SOME MORE STUFF 501
Prelude> map (+1) [1, 2, 3, 4]
[2,3,4,5]
Prelude> map (1-) [1, 2, 3, 4]
[0,-1,-2,-3]
Prelude> fmap (+1) [1, 2, 3, 4]
[2,3,4,5]
Prelude> fmap (2*) [1, 2, 3, 4]
[2,4,6,8]
Prelude> fmap id [1, 2, 3]
[1,2,3]
Prelude> map id [1, 2, 3]
[1,2,3]
The types of mapandfmaprespectively are:
map:: (a->b)->[a]->[b]
fmap::Functor f=>(a->b)->f a->f b
Let’s look at how the types line up with a program, starting
withmap:
map::(a->b)->[a]->[b]
map(+1)
The(a -> b) becomes more specific and resolves to Num a
=> a -> a :

CHAPTER 9. THIS THING AND SOME MORE STUFF 502
Prelude>:t map (+1)
map(+1)::Numb=>[b]->[b]
Now we see it will take one list of Numas an argument and
return a list of Numas a result.
The type of fmapwill behave similarly:
fmap::Functor f=>(a->b)->f a->f b
-- notice the Functor typeclass constraint
fmap(+1)
-- again, (a -> b) is now more specific
It’s a bit diﬀerent from mapbecause the Functor typeclass
includes more than lists:
Prelude> :t fmap (+1)
fmap (+1) :: (Num b, Functor f) => f b -> f b
Here’s how mapis defined in base:
map::(a->b)->[a]->[b]
map_[]=[]
-- [1] [2] [3]
mapf (x:xs)=f x:map f xs
-- [4] [5] [6] [7] [8]

CHAPTER 9. THIS THING AND SOME MORE STUFF 503
1._is used here to ignore the function argument because
we don’t need it.
2.We are pattern matching on the []empty list case because
List is a sum type with two cases and we must handle both
every time we pattern match or case on a list value.
3.We return the []empty list value because when there are
no values, it’s the only correct thing we can do. If you
attempt to do anything else, the typechecker will swat
you.
4.We bind the function argument to the name 𝑓as it merits
no name more specific than this. 𝑓and𝑔are common
names for nonspecific function values in Haskell. This is
the function we are mapping over the list value with map
5.We do not leave the entire list argument bound as a single
name. Since we’ve already pattern-matched the []empty
list case, we know there must be at least one value in
the list. Here we pattern match into the (:)second data
constructor of the list, which is a product. 𝑥is the single
value of the cons product. 𝑥𝑠is the rest of the list.
6.We apply our function 𝑓to the single value 𝑥. This part
of themapfunction is what applies the function argument
to the contents of the list.

CHAPTER 9. THIS THING AND SOME MORE STUFF 504
7.We(:)cons the value returned by the expression f xonto
the head of the result of map’ing the rest of the list. Data is
immutable in Haskell. When we map, we do not mutate
the existing list, but build a new list with the values that
result from applying the function.
8.We call mapitself applied to 𝑓and𝑥𝑠. This expression is the
rest of the list with the function 𝑓applied to each value.
How do we write out what map fdoes? Note, this order of
evaluation doesn’t represent the proper nonstrict evaluation
order, but does give an idea of what’s going on:
map(+1) [1,2,3]
-- desugared, (:) is infixr 5,
-- so it's right-associative
map(+1) (1:(2:(3:[])))
-- Not an empty list, so second
-- pattern-match in map fires.
-- Apply (+1) to value, then map
(+1)1:
map (+1)
(2:(3:[]))

CHAPTER 9. THIS THING AND SOME MORE STUFF 505
-- Apply (+1) to the next value, cons onto
-- the result of mapping over the rest
(+1)1:
((+1)2:
(map (+1)
(3:[])))
-- Last time we'll trigger the
-- second-case of map
(+1)1:
((+1)2:
((+1)3:
(map (+1)[])))
-- Now we trigger the base-case that
-- handles empty list and return the
-- empty list.
(+1)1:
((+1)2:
((+1)3:[]))
-- Now we reduce
2:((+1)2:((+1)3:[]))
2:3:(+1)3:[]
2:3:4:[]==[2,3,4]

CHAPTER 9. THIS THING AND SOME MORE STUFF 506
Using the syntactic sugar of list, here’s an approximation of
whatmapis doing for us:
mapf [1,2,3]==[f1, f2, f3]
map(+1) [1,2,3]
[(+1)1, (+1)2, (+1)3]
[2,3,4]
Or using the spine syntax we introduced earlier:
:
/ \
1 :
/ \
2 :
/ \
3 []
map (+1) [1, 2, 3]
:
/ \
(+1) 1 :
/ \
(+1) 2 :
/ \

CHAPTER 9. THIS THING AND SOME MORE STUFF 507
(+1) 3 []
As we mentioned above, these representations do not ac-
count for nonstrict evaluation. Crucially, mapdoesn’t traverse
the whole list and apply the function immediately. The func-
tion is applied to the values you force out of the list one by one.
We can see this by selectively leaving some values undefined:
Prelude> map (+1) [1, 2, 3]
[2,3,4]
-- the whole list was forced because
-- GHCi printed the list that resulted
Prelude> (+1) undefined
*** Exception: Prelude.undefined
Prelude> (1, undefined)
(1,*** Exception: Prelude.undefined
Prelude> fst (1, undefined)
1
Prelude> map (+1) [1, 2, undefined]
[2,3,*** Exception: Prelude.undefined

CHAPTER 9. THIS THING AND SOME MORE STUFF 508
Prelude> take 2 $ map (+1) [1, 2, undefined]
[2,3]
In the final example, the undefined value was never forced
and there was no error because we used take 2 to request only
the first two elements. With map (+1) we only force as many
values as cons cells we forced. We’ll only force the values if
we evaluate the result value in the list that the map function
returns.
The significant part here is that strictness doesn’t proceed
only outside-in. We can have lazily evaluated code (e.g., map)
wrapped around a strict core (e.g., +). In fact, we can choose to
apply laziness and strictness in how we evaluate the spine or
the leaves independently. A common mantra for performance
sensitive code in Haskell is, “lazy in the spine, strict in the
leaves.” We’ll cover this properly later when we talk about
nonstrictness and data structures, although many Haskell users
rarely worry about this.
You can use mapandfmapwith other functions and list types
as well. In this example, we use the fstfunction to return a
list of the first element of each tuple in a list of tuples:
Prelude> map fst [(2, 3), (4, 5), (6, 7), (8, 9)]
[2,4,6,8]
Prelude> fmap fst [(2, 3), (4, 5), (6, 7), (8, 9)]
[2,4,6,8]

CHAPTER 9. THIS THING AND SOME MORE STUFF 509
In this example we map a partially applied takefunction:
Prelude> map (take 3) [[1..5], [1..5], [1..5]]
[[1,2,3],[1,2,3],[1,2,3]]
Next, we’ll map an if-then-else over a list using an anony-
mous function. This list will find any value equal to 3, negate
it, and then return the list:
Prelude> map (\x -> if x == 3 then (-x) else (x)) [1..10]
[1,2,-3,4,5,6,7,8,9,10]
At this point, you can try your hand at mapping diﬀerent
functions using this as a model. We recommend getting com-
fortable with mapping before moving on to the Folds chapter.
Exercises: More Bottoms
As always, we encourage you to try figuring out the answers
before you enter them into your REPL.
1.Will the following expression return a value or be ⊥?
take1$map (+1) [undefined, 2,3]
2.Will the following expression return a value?
take1$map (+1) [1, undefined, 3]
3.Will the following expression return a value?

CHAPTER 9. THIS THING AND SOME MORE STUFF 510
take2$map (+1) [1, undefined, 3]
4.What does the following mystery function do? What is its
type? Describe it (to yourself or a loved one) in standard
English and then test it out in the REPL to make sure you
were correct.
itIsMystery xs=
map (\x->elem x"aeiou") xs
5.What will be the result of the following functions:
a)map(^2) [1..10]
b)mapminimum [[ 1..10], [10..20], [20..30]]
-- n.b. `minimum` is not the same function
-- as the `min` that we used before
c)mapsum [[1..5], [1..5], [1..5]]
6.Back in chapter 7, you wrote a function called foldBool .
That function exists in a module known as Data.Bool and
is called bool. Write a function that does the same (or
similar, if you wish) as the map (if-then-else) function you
saw above but uses boolinstead of the if-then-else syntax.
Your first step should be bringing the boolfunction into
scope by typing import Data.Bool at your Prelude prompt.

CHAPTER 9. THIS THING AND SOME MORE STUFF 511
9.10 Filtering lists of values
When we talked about function composition in Chapter 7,
we used a function called filter that takes a list as input and
returns a new list consisting solely of the values in the input
list that meet a certain condition, as in this example which
finds the even numbers of a list and returns a new list of those
values:
Prelude> filter even [1..10]
[2,4,6,8,10]
Let’s now take a closer look at filter .filter has the follow-
ing definition:
filter::(a->Bool)->[a]->[a]
filter_[]=[]
filterpred (x:xs)
|pred x =x:filter pred xs
|otherwise =filter pred xs
Filtering takes a function that returns a Boolvalue, maps
that function over a list, and returns a new list of all the values
that met the condition. It’s important to remind ourselves that
this function, as we can see in the definition, builds a new list
including values that meet the condition and excluding the
ones that do not — it does not mutate the existing list.

CHAPTER 9. THIS THING AND SOME MORE STUFF 512
We have seen how filter works with oddandevenalready.
We have also seen one example along the lines of this:
Prelude> filter (== 'a') "abracadabra"
"aaaaa"
Asyoumightsuspectfromwhatwe’veseenofHOFs, though,
filter can handle many types of arguments. The following ex-
ample does the same thing as filter even but with anonymous
function syntax:
Prelude> filter (\x -> (rem x 2) == 0) [1..20]
[2,4,6,8,10,12,14,16,18,20]
We covered list comprehensions earlier as a way of filtering
lists as well. Compare the following:
Prelude> filter (\x -> elem x "aeiou") "abracadabra"
"aaaaa"
Prelude> [x | x <- "abracadabra", elem x "aeiou"]
"aaaaa"
As they say, there’s more than one way to skin a cat.
Again, we recommend at this point you try writing some
filter functions of your own to get comfortable with the pat-
tern.

CHAPTER 9. THIS THING AND SOME MORE STUFF 513
Exercises: Filtering
1.Given the above, how might we write a filter function that
would give us all the multiples of 3 out of a list from 1-30?
2.Recalling what we learned about function composition,
how could we compose the above function with the length
function to tell us *how many* multiples of 3 there are
between 1 and 30?
3.Next we’re going to work on removing all articles (’the’, ’a’,
and ’an’) from sentences. You want to get to something
that works like this:
Prelude> myFilter "the brown dog was a goof"
["brown","dog","was","goof"]
You may recall that earlier in this chapter we asked you
to write a function that separates a string into a list of
strings by separating them at spaces. That is a standard
library function called words. You may consider starting
this exercise by using words(or your version, of course).
9.11 Zipping lists
Zipping lists together is a means of combining values from
multiple lists into a single list. Related functions like zipWith

CHAPTER 9. THIS THING AND SOME MORE STUFF 514
allow you to use a combining function to produce a list of
results from two lists.
First let’s look at zip:
Prelude> :t zip
zip :: [a] -> [b] -> [(a, b)]
Prelude> zip [1, 2, 3] [4, 5, 6]
[(1,4),(2,5),(3,6)]
One thing to note is that zipstops as soon as one of the lists
runs out of values:
Prelude> zip [1, 2] [4, 5, 6]
[(1,4),(2,5)]
Prelude> zip [1, 2, 3] [4]
[(1,4)]
And will return an empty list if either of the lists is empty:
Prelude> zip [] [1..1000000000000000000]
[]
zipproceeds until the shortest list ends.
Prelude> zip ['a'] [1..1000000000000000000]
[('a',1)]
Prelude> zip [1..100] ['a'..'c']
[(1,'a'),(2,'b'),(3,'c')]

CHAPTER 9. THIS THING AND SOME MORE STUFF 515
We can use unzipto recover the lists as they were before
they were zipped:
Prelude> zip [1, 2, 3] [4, 5, 6]
[(1,4),(2,5),(3,6)]
Prelude> unzip $ zip [1, 2, 3] [4, 5, 6]
([1,2,3],[4,5,6])
Prelude> fst $ unzip $ zip [1, 2, 3] [4, 5, 6]
[1,2,3]
Prelude> snd $ unzip $ zip [1, 2, 3] [4, 5, 6]
[4,5,6]
Be aware that information can be lost in this process because
zipmust stop on the shortest list:
Prelude> snd $ unzip $ zip [1, 2] [4, 5, 6]
[4,5]
We can also use zipWith to apply a function to the values of
two lists in parallel:
zipWith ::(a->b->c)
-- [1]
->[a]->[b]->[c]
-- [2] [3] [4]

CHAPTER 9. THIS THING AND SOME MORE STUFF 516
1.A function with two arguments. Notice how the type
variables of the arguments and result align with the type
variables in the lists.
2.The first input list.
3.The second input list.
4.The output list created from applying the function to the
values in the input lists.
A brief demonstration of how zipWith works:
Prelude> zipWith (+) [1, 2, 3] [10, 11, 12]
[11,13,15]
Prelude> zipWith (*) [1, 2, 3] [10, 11, 12]
[10,22,36]
Prelude> zipWith (==) ['a'..'f'] ['a'..'m']
[True,True,True,True,True,True]
Prelude> let xs = [10, 5, 34, 9]
Prelude> let xs' = [6, 8, 12, 7]
Prelude> zipWith max xs xs'
[10,8,34,9]

CHAPTER 9. THIS THING AND SOME MORE STUFF 517
Zipping exercises
1.Write your own version of zipand ensure it behaves the
same as the original.
zip::[a]->[b]->[(a, b)]
zip=undefined
2.Do what you did for zip, but now for zipWith :
zipWith ::(a->b->c)
->[a]->[b]->[c]
zipWith =undefined
3.Rewrite your zipin terms of the zipWith you wrote.
9.12 Chapter Exercises
The first set of exercises here will mostly be review but will
also introduce you to some new things. The second set is
more conceptually challenging but does not use any syntax or
concepts we haven’t already studied. If you get stuck, it may
help to flip back to a relevant section and review.
Data.Char
These first few exercises are straightforward but will introduce
you to some new library functions and review some of what

CHAPTER 9. THIS THING AND SOME MORE STUFF 518
we’ve learned so far. Some of the functions we will use here
are not standard in Prelude and so have to be imported from
a module called Data.Char . You may do so in a source file
(recommended) or at the Prelude prompt with the same phrase:
import Data.Char (write that at the top of your source file). This
brings into scope a bunch of new standard functions we can
play with that operate on CharandString types.
1.Query the types of isUpper andtoUpper .
2.Given the following behaviors, which would we use to
write a function that filters all the uppercase letters out
of aString ? Write that function such that, given the input
“HbEfLrLxO,” your function will return “HELLO.”
Prelude Data.Char> isUpper 'J'
True
Prelude Data.Char> toUpper 'j'
'J'
3.Write a function that will capitalize the first letter of a
string and return the entire string. For example, if given
the argument “julie,” it will return “Julie.”
4.Now make a new version of that function that is recursive
such that if you give it the input “woot” it will holler back
at you “WOOT.” The type signature won’t change, but
you will want to add a base case.

CHAPTER 9. THIS THING AND SOME MORE STUFF 519
5.To do the final exercise in this section, we’ll need another
standard function for lists called head. Query the type of
headand experiment with it to see what it does. Now write
a function that will capitalize the first letter of a String
and return only that letter as the result.
6.Cool. Good work. Now rewrite it as a composed function.
Then, for fun, rewrite it pointfree.
Ciphers
We’ll still be using Data.Char for this next exercise. You should
save these exercises in a module called Cipher because we’ll
be coming back to them in later chapters. You’ll be writing a
Caesar cipher for now, but we’ll suggest some variations on
the basic program in later chapters.
A Caesar cipher is a simple substitution cipher, in which
each letter is replaced by the letter that is a fixed number of
places down the alphabet from it. You will find variations on
this all over the place — you can shift leftward or rightward,
for any number of spaces. A rightward shift of 3 means that
’A’ will become ’D’ and ’B’ will become ’E,’ for example. If you
did a leftward shift of 5, then ’a’ would become ’v’ and so forth.
Your goal in this exercise is to write a basic Caesar cipher
that shifts rightward. You can start by having the number of
spaces to shift fixed, but it’s more challenging to write a cipher

CHAPTER 9. THIS THING AND SOME MORE STUFF 520
that allows you to vary the number of shifts so that you can
encode your secret messages diﬀerently each time.
There are Caesar ciphers written in Haskell all over the
internet, but to maximize the likelihood that you can write
yours without peeking at those, we’ll provide a couple of tips.
When yours is working the way you want it to, we would
encourage you to then look around and compare your solution
to others out there.
The first lines of your text file should look like this:
moduleCipherwhere
importData.Char
Data.Char includes two functions called ordandchrthat can
be used to associate a Charwith its Intrepresentation in the
Unicode system and vice versa:
*Cipher>:t chr
chr::Int->Char
*Cipher>:t ord
ord::Char->Int
Using these functions is optional; there are other ways you
can proceed with shifting, but using chrandordmight simplify
the process a bit.

CHAPTER 9. THIS THING AND SOME MORE STUFF 521
You want your shift to wrap back around to the beginning of
the alphabet, so that if you have a rightward shift of 3 from ’z,’
you end up back at ’c’ and not somewhere in the vast Unicode
hinterlands. Depending on how you’ve set things up, this
might be a bit tricky. Consider starting from a base character
(e.g., ’a’) and using modto ensure you’re only shifting over the
26 standard characters of the English alphabet.
You should include an unCaesar function that will decipher
your text as well. In a later chapter, we will test it.
Writing your own standard functions
Below are the outlines of some standard functions. The goal
here is to write your own versions of these to gain a deeper
understanding of recursion over lists and how to make func-
tions flexible enough to accept a variety of inputs. You could
figure out how to look up the answers, but you won’t do that
because you know you’d only be cheating yourself out of the
knowledge. Right?
Let’s look at an example of what we’re after here. The and2
function can take a list of Boolvalues and returns True if and
only if no values in the list are False. Here’s how you might
write your own version of it:
2Note that if you’re using GHC 7.10 or newer, the functions and,any, andallhave
been abstracted from being usable only with lists to being usable with any datatype that
has an instance of the typeclass Foldable . It still works with lists, the same as it did before.
Proceed assured that we’ll cover this later.

CHAPTER 9. THIS THING AND SOME MORE STUFF 522
-- direct recursion, not using (&&)
myAnd::[Bool]->Bool
myAnd[]=True
myAnd(x:xs)=
ifx==False
thenFalse
elsemyAnd xs
-- direct recursion, using (&&)
myAnd::[Bool]->Bool
myAnd[]=True
myAnd(x:xs)=x&&myAnd xs
And now the fun begins:
1.myOrreturns Trueif anyBoolin the list is True.
myOr::[Bool]->Bool
myOr=undefined
2.myAnyreturns Trueifa -> Bool applied to any of the values
in the list returns True.
myAny::(a->Bool)->[a]->Bool
myAny=undefined
Example for validating myAny:

CHAPTER 9. THIS THING AND SOME MORE STUFF 523
Prelude> myAny even [1, 3, 5]
False
Prelude> myAny odd [1, 3, 5]
True
3.After you write the recursive myElem , write another version
that uses any. The built-in version of elemin GHC 7.10 and
newer has a type that uses Foldable instead of the list type
specifically. You can ignore that and write the concrete
version that works only for list.
myElem::Eqa=>a->[a]->Bool
Prelude> myElem 1 [1..10]
True
Prelude> myElem 1 [2..10]
False
4.Implement myReverse .
myReverse ::[a]->[a]
myReverse =undefined
Prelude> myReverse "blah"
"halb"
Prelude> myReverse [1..5]
[5,4,3,2,1]

CHAPTER 9. THIS THING AND SOME MORE STUFF 524
5.squish flattens a list of lists into a list
squish::[[a]]->[a]
squish=undefined
6.squishMap maps a function over a list and concatenates the
results.
squishMap ::(a->[b])->[a]->[b]
squishMap =undefined
Prelude> squishMap (\x -> [1, x, 3]) [2]
[1,2,3]
Prelude> squishMap (\x -> "WO "++[x]++" HOO ") "123"
"WO 1 HOO WO 2 HOO WO 3 HOO "
7.squishAgain flattens a list of lists into a list. This time re-use
thesquishMap function.
squishAgain ::[[a]]->[a]
squishAgain =undefined
8.myMaximumBy takes a comparison function and a list and
returns the greatest element of the list based on the last
value that the comparison returned GTfor. If you import
maximumBy fromData.List , you’ll see the type is:
Foldable t
=>(a->a->Ordering )->t a->a

CHAPTER 9. THIS THING AND SOME MORE STUFF 525
rather than
(a->a->Ordering )->[a]->a
myMaximumBy ::(a->a->Ordering )
->[a]->a
myMaximumBy =undefined
Prelude> let xs = [1, 53, 9001, 10]
Prelude> myMaximumBy compare xs
9001
9.myMinimumBy takes a comparison function and a list and
returns the least element of the list based on the last value
that the comparison returned LT for.
myMinimumBy ::(a->a->Ordering )
->[a]->a
myMinimumBy =undefined
Prelude> let xs = [1, 53, 9001, 10]
Prelude> myMinimumBy compare xs
1
10.Usingthe myMinimumBy andmyMaximumBy functions, writeyour
own versions of maximum andminimum . If you have GHC 7.10

CHAPTER 9. THIS THING AND SOME MORE STUFF 526
ornewer, you’llseeatypeconstructorthatwantsa Foldable
instance instead of a list as has been the case for many
functions so far.
myMaximum ::(Orda)=>[a]->a
myMaximum =undefined
myMinimum ::(Orda)=>[a]->a
myMinimum =undefined
9.13 Definitions
1.In type theory, a producttype is a type made of a set of types
compounded over each other. In Haskell we represent
products using tuples or data constructors with more than
one argument. The “compounding” is from each type
argument to the data constructor representing a value that
coexists with all the other values simultaneously. Products
of types represent a conjunction, “and,” of those types. If
you have a product of BoolandInt, your terms will each
contain a BoolandIntvalue.
2.In type theory, a sum type of two types is a type whose
terms are terms in either type, but not simultaneously. In
Haskell sum types are represented using the pipe, |, in a
datatype definition. Sums of types represent a disjunction,

CHAPTER 9. THIS THING AND SOME MORE STUFF 527
“or,” of those types. If you have a sum of BoolandInt, your
terms will be eitheraBoolvalueoranIntvalue.
3.Consis ordinarily used as a verb to signify that a list value
has been created by cons’ing a value onto the head of
another list value. In Haskell, (:)is the cons operator for
the list type. It is a data constructor defined in the list
datatype:
1:[2,3]
-- [a] [b]
[1,2,3]
-- [c]
(:)::a->[a]->[a]
-- [d] [e] [f]
a)The number 1, the value we are consing.
b)A list of the number 2 followed by the number 3.
c)The final result of consing 1onto[2, 3] .
d)The type variable 𝑎corresponds to 1, the value we
consed onto the list value.
e)The first occurrence of the type [a]in the cons oper-
ator’s type corresponds to the second and final argu-
ment(:)accepts, which was [2, 3] .

CHAPTER 9. THIS THING AND SOME MORE STUFF 528
f)The second and final occurrence of the type [a]in the
cons operator’s type corresponds to the final result
[1, 2, 3] .
4.Cons cell is a data constructor and a product of the types
aand[a]as defined in the list datatype. Because it refer-
ences the list type constructor itself in the second argu-
ment, it allows for nesting of multiple cons cells, possibly
indefinitely with the use of recursive functions, for repre-
senting an indefinite number of values in series:
data[]a=[]|a:[a]
-- ^ cons operator
-- Defining it ourselves
dataLista=Nil|Consa (Lista)
-- Creating a list using our list type
Cons1(Cons2(Cons3Nil))
Here(Cons 1 ...) ,(Cons 2 ...) and(Cons 3 Nil) are all
individual cons cells in the list [1, 2, 3] .
5.Thespineis a way to refer to the structure that glues a
collection of values together. In the list datatype it is

CHAPTER 9. THIS THING AND SOME MORE STUFF 529
formed by the recursive nesting of cons cells. The spine is,
in essence, the structure of collection that isn’tthe values
contained therein. Often spine will be used in reference
to lists, but it applies with tree data structures as well:
-- Given the list [1, 2, 3]
1:--------| The nested cons operators
(2:-----| here represent the spine.
(3:--|
[]))
-- Blanking the irrelevant values out
_:----------|
(_:-------|
(_:----> Spine
[]))
9.14 Follow-up resources
1.Data.List documentation for the baselibrary.
http://hackage.haskell.org/package/base/docs/Data-List.html
2.Ninety-nine Haskell problems.
https://wiki.haskell.org/H-99:_Ninety-Nine_Haskell_Problems

Chapter 10
Folding lists
The explicit teaching of
thinking is no trivial task,
but who said that the
teaching of programming
is? In our terminology,
the more explicitly
thinking is taught, the
more of a scientist the
programmer will
become.
Edsger Dijkstra
530

CHAPTER 10. DATA STRUCTURE ORIGAMI 531
10.1 Folds
Folding is a concept that extends in usefulness and importance
beyond lists, but lists are often how they are introduced. Folds
as a general concept are called catamorphisms. You’re famil-
iar with the root, “morphism” from polymorphism. “Cata-”
means “down” or “against”, as in “catacombs.” Catamorphisms
are a means of deconstructing data. If the spine of a list is the
structure of a list, then a fold is what can reduce that structure.1
This chapter is a thorough look at the topic of folding lists
in Haskell. We will:
•explain what folds are and how they work;
•detail the evaluation processes of folds;
•walk through writing folding functions;
•introduce scans, functions that are related to folds.
10.2 Bringing you into the fold
Let’s start with a quick look at foldr, short for “fold right.” This
is the fold you’ll most often want to use with lists. The follow-
ing type signature may look a little hairy, but let’s compare it
1Note that a catamorphism canbreak down the structure but that structure might be
rebuilt, so to speak, during evaluation. That is, folds can return lists as results.

CHAPTER 10. DATA STRUCTURE ORIGAMI 532
to what we know about mapping. Note that the type of foldr
changed with GHC 7.10:
-- GHC 7.8 and older
foldr::(a->b->b)->b->[a]->b
-- GHC 7.10 and newer
foldr::Foldable t
=>(a->b->b)
->b
->t a
->b
Lined up next to each other:
foldr::Foldable t=>
(a->b->b)->b->t a->b
foldr::(a->b->b)->b->[]a->b
For now, all you need to know is that GHC 7.10 abstracted
out the list-specific part of folding into a typeclass that lets you
reuse the same folding functions for any datatype that can be
folded — not just lists. We can even recover the more concrete
type because we can always make a type more concrete, but
never more generic:
Prelude> :{

CHAPTER 10. DATA STRUCTURE ORIGAMI 533
Prelude| let listFoldr :: (a -> b -> b)
Prelude| -> b
Prelude| -> [] a
Prelude| -> b
Prelude| listFoldr = foldr
Prelude| :}
Prelude> :t listFoldr
listFoldr :: (a -> b -> b) -> b -> [a] -> b
Now let’s notice a parallel between mapandfoldr:
foldr::(a->b->b)->b->[a]->b
-- Remember how map worked?
map::(a->b)->[a]->[b]
map(+1)1: 2: 3:[]
(+1)1:(+1)2:(+1)3:[]
-- Given the list
foldr(+)0(1:2:3:[])
1+(2+(3+0))
Where mapapplies a function to each member of a list and
returns a list, a fold replaces the cons constructors with the
function and reduces the list.

CHAPTER 10. DATA STRUCTURE ORIGAMI 534
10.3 Recursive patterns
Let’s revisit sum:
Prelude> sum [1, 5, 10]
16
As we’ve seen, it takes a list, adds the elements together,
and returns a single result. You might think of it as similar
to themapfunctions we’ve looked at, except that it’s mapping
(+)over the list, replacing the cons operators themselves, and
returning a single result, instead of mapping, for example, (+1)
into each cons cell and returning a whole list of results back
to us. This has the eﬀect of both mapping an operator over a
list and also reducing the list. In a previous section, we wrote
sumin terms of recursion:
sum::[Integer]->Integer
sum[]=0
sum(x:xs)=x+sum xs
And if we bring back our length function from earlier:
length::[a]->Integer
length[]=0
length(_:xs)=1+length xs

CHAPTER 10. DATA STRUCTURE ORIGAMI 535
Do you see some structural similiarity? What if you look at
product andconcat as well?
product ::[Integer]->Integer
product []=1
product (x:xs)=x*product xs
concat::[[a]]->[a]
concat[]=[]
concat(x:xs)=x++concat xs
In each case, the base case is the identity for that function.
So the identity for sum,length ,product , andconcat respectively
are 0, 0, 1, and []. When we do addition, adding zero gives us
the same result as our initial value: 1 + 0 = 1 . But when we do
multiplication, it’s multiplying by 1 that gives us the identity:
2 * 1 = 2 . With list concatenation, the identity is the empty
list, such that [1, 2, 3] ++ [] == [1, 2, 3] .
Also, each of them has a main function with a recursive
pattern that associates to the right. The head of the list gets
evaluated, set aside, and then the function moves to the right,
evaluates the next head, and so on.
10.4 Fold right
We call foldrthe “right fold” because the fold is right asso-
ciative; that is, it associates to the right. This is syntactically

CHAPTER 10. DATA STRUCTURE ORIGAMI 536
reflected in a straightforward definition of foldras well:
foldr::(a->b->b)->b->[a]->b
foldrf z[]=z
foldrf z (x:xs)=f x (foldr f z xs)
The similarities between this and the recursive patterns we
saw above should be clear. The “rest of the fold,” (foldr f z xs)
is an argument to the function 𝑓we’re folding with. The 𝑧is
the zero of our fold. It provides a fallback value for the empty
list case and a second argument to begin our fold with. The
zero is often the identity for whatever function we’re folding
with, such as 0 for (+)and 1 for (*).
How foldr evaluates
We’re going to rejigger our definition of foldra little bit. It
won’t change the semantics, but it’ll make it easier to write out
what’s happening:
foldr::(a->b->b)->b->[a]->b
foldrf z xs=
casexsof
[]->z
(x:xs)->f x (foldr f z xs)
Here we see how the right fold associates to the right. This
will reduce like the sumexample from earlier:

CHAPTER 10. DATA STRUCTURE ORIGAMI 537
foldr(+)0[1,2,3]
When we reduce that fold, the first step is substituting 𝑥𝑠in
our case expression:
foldr(+)0[1,2,3]=
case[1,2,3]of
...
Which case of the expression matches?
foldr(+)0[1,2,3]=
case[1,2,3]of
[]->0
(x:xs)->
f x (foldr f z xs) --<---this one
What are f, x, xs, and z in that branch of the case?
foldr(+)0[1,2,3]=
case[1,2,3]of
[] ->0
(1:[2,3])->
(+)1(foldr ( +)0[2,3])
Critically, we’re going to expand (foldr (+) 0 [2, 3]) only
because (+)is strict in both of its arguments, so it forces the

CHAPTER 10. DATA STRUCTURE ORIGAMI 538
next iteration. We could have a function which doesn’t contin-
ually force the rest of the fold. If it were to stop on the first case
here, then it would’ve returned the value 1. One such function
isconstwhich always returns the first argument. We’ll show
you how that behaves in a bit. Our next recursion is the (foldr
(+) 0 [2, 3]) :
foldr(+)0[2,3]=
case[2,3]of
[] ->
0-- this didn't match again
(2:[3])->(+)2(foldr ( +)0[3])
There is (+) 1implicitly wrapped around this continuation
of the recursive fold. (+)is not only strict in both of its argu-
ments, but it’s unconditionally so, so we’re going to proceed to
the next recursion of foldr. Note that the function calls bounce
between our folding function 𝑓andfoldr. This bouncing back
and forth gives more control to the folding function. A hypo-
thetical folding function, such as const, which doesn’t need the
second argument has the opportunity to do less work by not
evaluating its second argument which is “more of the fold.”
There is (+) 1 ((+) 2 ...) implicitly wrapped around this
next step of the recursive fold:

CHAPTER 10. DATA STRUCTURE ORIGAMI 539
foldr(+)0[3]=
case[3]of
[] ->
0-- this didn't match again
(3:[])->(+)3(foldr ( +)0[])
We’re going to ask for more foldrone last time. There is,
again,(+) 1 ((+) 2 ((+) 3 ...)) implicitly wrapped around
this final step of the recursive fold. Now we hit our base case
and and hit our base case:
foldr(+)0[]=
case[]of
[] ->
0--<--Thisone finally matches
-- ignore the other case, didn't happen
So one way to think about the way Haskell evaluates is that
it’s like a text rewriting system. Our expression has thus far
rewritten itself from:
foldr(+)0[1,2,3]
Into:
(+)1((+)2((+)30))

CHAPTER 10. DATA STRUCTURE ORIGAMI 540
If you wanted to clean it up a bit without changing how it
evaluates, you could make it the following:
1+(2+(3+0))
As in arithmetic, we evaluate innermost parentheses first:
1+(2+(3+0))
1+(2+3)
1+5
6
And now we’re done, with the result of 6.
We can also use a trick popularized by some helpful users
in the Haskell IRC community to see how the fold associates.2
xs=map show [ 1..5]
y=foldr (\x y->concat
["(",x,"+",y,")"])"0"xs
When we call 𝑦in the REPL, we can see how the foldreval-
uates:
2Idea borrowed from Cale Gibbard from the haskell Freenode IRC channel and on
the Haskell.org wiki https://wiki.haskell.org/Fold#Examples

CHAPTER 10. DATA STRUCTURE ORIGAMI 541
Prelude> y
"(1+(2+(3+(4+(5+0)))))"
One initially nonobvious aspect of folding is that it happens
in two stages, traversal and folding. Traversal is the stage
in which the fold recurses over the spine. Folding refers to
the evaluation or reduction of the folding function applied
to the values. All folds recurse over the spine in the same
direction; the diﬀerence between left folds and right folds is
in the association, or parenthesization, of the folding function
and, thus, which direction the folding or reduction proceeds.
Withfoldr, the rest of our fold is an argument to the func-
tion we’re folding with:
foldrf z (x:xs)=f x (foldr f z xs)
-- ^--------------^
-- rest of the fold
Given this two-stage process and nonstrict evaluation, if
𝑓doesn’t evaluate its second argument (rest of the fold), no
more of the spine will be forced. One of the consequences of
this is that foldrcan avoid evaluating not only some or all of
the values in the list, but some or all of the list’s spineas well!
For this reason, foldrcan be used with lists that are potentially
infinite. For example, compare the following sets of results

CHAPTER 10. DATA STRUCTURE ORIGAMI 542
(recall that (+)will unconditionally evaluate the entire spine
and all of the values):
Prelude> foldr (+) 0 [1..5]
15
While you cannot use foldrwith addition on an infinite list,
you can use functions that are not strict in both arguments and
therefore do not require evaluation of every value in order to
return a result. The function myAny, for example, can return a
Trueresult as soon as it finds one True:
myAny::(a->Bool)->[a]->Bool
myAnyf xs=
foldr (\x b->f x||b)Falsexs
The following should work despite being an infinite list:
Prelude> myAny even [1..]
True
The following will never finish evaluating because it’s always
an odd number:
Prelude> myAny even (repeat 1)
Another term we use for this never-ending evaluation is
bottom orundefined . There’s no guarantee that a fold of an

CHAPTER 10. DATA STRUCTURE ORIGAMI 543
infinite list will finish evaluating even if you used foldr, it of-
ten depends on the input data and the fold function. Let us
consider some more examples with a less inconvenient bottom :
Prelude> let u = undefined
-- here, we give an undefined value
Prelude> foldr (+) 0 [1, 2, 3, 4, u]
*** Exception: Prelude.undefined
Prelude> let xs = take 4 [1, 2, 3, 4, u]
Prelude> foldr (+) 0 xs
10
-- here, undefined is part of the spine
Prelude> let xs = [1, 2, 3, 4] ++ u
Prelude> foldr (+) 0 xs
*** Exception: Prelude.undefined
Prelude> let xs = take 4 ([1, 2, 3, 4]++u)
Prelude> foldr (+) 0 xs
10
By taking only the first four elements, we stop the recursive
folding process at the first four values so our addition function
does not run into bottom, and that works whether undefined is
one of the values or part of the spine.

CHAPTER 10. DATA STRUCTURE ORIGAMI 544
Thelength function behaves diﬀerently; it evaluates the
spine unconditionally, but not the values:
Prelude> length [1, 2, 3, 4, undefined]
5
Prelude> length ([1, 2, 3, 4] ++ undefined)
*** Exception: Prelude.undefined
However, if we drop the part of the spine that includes the
bottom before we use length, we can get an expression that
works:
Prelude> let xs = [1, 2, 3, 4] ++ undefined
Prelude> length (take 4 xs)
4
takeis nonstrict like everything else you’ve seen so far, and
in this case, it only returns as much list as you ask for. The dif-
ference in what it does, is it stopsreturning elements of the list
it was given when it hits the length limit you gave it. Consider
this:
Prelude> let xs = [1, 2] ++ undefined
Prelude> length $ take 2 $ take 4 xs
2
It doesn’t matter that take 4 could’ve hit the bottom! Noth-
ing forced it to because of the take 2 between it and length .

CHAPTER 10. DATA STRUCTURE ORIGAMI 545
Now that we’ve seen how the recursive second argument to
foldr’s folding function works, let’s consider the first argument:
foldr::(a->b->b)->b->[a]->b
foldrf z[]=z
foldrf z (x:xs)=f x (foldr f z xs)
-- [1]
The first argument, [1], involves a pattern match that is
strict by default — the 𝑓only applies to 𝑥if there is an 𝑥value
and not just an empty list. This means that foldrmust force
an initial cons cell in order to discriminate between the []and
the(x : xs) cases, so the first cons cell cannot be undefined.
Now we’re going to try something unusual to demonstrate
that the first bit of the spine must be evaluated by foldr. We
have a somewhat silly anonymous function that will ignore
all its arguments and return a value of 9001. We’re using it
withfoldrbecause it will never force evaluation of any of its
arguments, so we can have a bottom as a value or as part of
the spine, and it will not force an evaluation:
Prelude> foldr (\_ _ -> 9001) 0 [1..5]
9001
Prelude> let xs = [1, 2, 3, undefined]
Prelude> foldr (\_ _ -> 9001) 0 xs
9001

CHAPTER 10. DATA STRUCTURE ORIGAMI 546
Prelude> let xs = [1, 2, 3] ++ undefined
Prelude> foldr (\_ _ -> 9001) 0 xs
9001
Everything is fine unless the first piece of the spine is bot-
tom:
Prelude> foldr (\_ _ -> 9001) 0 undefined
*** Exception: Prelude.undefined
Prelude> let xs = [1, undefined]
Prelude> foldr (\_ _ -> 9001) 0 xs
9001
Prelude> let xs = [undefined, undefined]
Prelude> foldr (\_ _ -> 9001) 0 xs
9001
The final two examples work because it isn’t the first cons
cellthat is bottom — the undefined values are inside the cons
cells, not in the spine itself. Put diﬀerently, the cons cells
contain bottom values but are not themselves bottom. We will
experiment later with nonstrictness and strictness to see how
it aﬀects the way our programs evaluate.
Traversing the rest of the spine doesn’t occur unless the
function asks for the results of having folded the rest of the
list. In the following examples, we don’t force traversal of the

CHAPTER 10. DATA STRUCTURE ORIGAMI 547
spine because constthrows away its second argument, which
is the rest of the fold:
-- reminder:
-- const :: a -> b -> a
-- const x _ = x
Prelude> const 1 2
1
Prelude> const 2 1
2
Prelude> foldr const 0 [1..5]
1
Prelude> foldr const 0 [1,undefined]
1
Prelude> foldr const 0 ([1,2] ++ undefined)
1
Prelude> foldr const 0 [undefined,2]
*** Exception: Prelude.undefined
Now that we’ve seen how foldrevaluates, we’re going to
look atfoldlbefore we move on to learning how to write and
use folds.

CHAPTER 10. DATA STRUCTURE ORIGAMI 548
10.5 Fold left
Because of the way lists work, folds must first recurse over
the spine of the list from the beginning to the end. Left folds
traverse the spine in the same direction as right folds, but their
folding process is left associative and proceeds in the opposite
direction as that of foldr.
Here’s a simple definition of foldl. Note that to see the same
type for foldlin your GHCi REPL you will need to import
Data.List for the same reasons as with foldr:
-- again, different type in
-- GHC 7.10 and newer.
foldl::(b->a->b)->b->[a]->b
foldlf acc[]=acc
foldlf acc (x :xs)=foldl f (f acc x) xs
foldl::(b->a->b)->b->[a]->b
-- Given the list
foldl(+)0(1:2:3:[])
-- foldl associates like this
((0+1)+2)+3

CHAPTER 10. DATA STRUCTURE ORIGAMI 549
We can also use the same trick we used to see the associa-
tivity of foldrto see the associativity of foldl:
Prelude> let conc = concat
Prelude> let f x y = conc ["(",x,"+",y,")"]
Prelude> foldl f "0" (map show [1..5])
"(((((0+1)+2)+3)+4)+5)"
We can see from this that foldlbegins its reduction process
by adding the acc(accumulator) value to the head of the list,
whereas foldrhad added it to the final element of the list first.
We can also use functions called scansto see how folds eval-
uate. Scans are similar to folds but return a list of all the inter-
mediate stages of the fold. We can compare scanrandscanlto
their accompanying folds to see the diﬀerence in evaluation:
Prelude> foldr (+) 0 [1..5]
15
Prelude> scanr (+) 0 [1..5]
[15,14,12,9,5,0]
Prelude> foldl (+) 0 [1..5]
15
Prelude> scanl (+) 0 [1..5]
[0,1,3,6,10,15]

CHAPTER 10. DATA STRUCTURE ORIGAMI 550
The relationship between the scans and folds are as follows:
last(scanl f z xs) =foldl f z xs
head(scanr f z xs) =foldr f z xs
Each fold will return the same result for this operation, but
we can see from the scans that they arrive at that result in a
diﬀerent order, due to the diﬀerent associativity. We’ll talk
more about scans later.
Associativity and folding
Next we’ll take a closer look at some of the eﬀects of the asso-
ciativity of foldl. As we’ve said, both folds traverse the spine
in the same direction. What’s diﬀerent is the associativity of
the evaluation.
The fundamental way to think about evaluation in Haskell
is as substitution. When we use a right fold on a list with the
function 𝑓and start value 𝑧, we’re, in a sense, replacing the
cons constructors with our folding function and the empty list
constructor with our start value 𝑧:

CHAPTER 10. DATA STRUCTURE ORIGAMI 551
[1..3]==1:2:3:[]
foldrf z [1,2,3]
1`f` (foldr f z [ 2,3])
1`f` (2`f` (foldr f z [ 3]))
1`f` (2`f` (3`f` (foldr f z [])))
1`f` (2`f` (3`f` z))
Furthermore, lazy evaluation lets our functions, rather than
the ambient semantics of the language, dictate what order
things get evaluated in. Because of this, the parentheses are real .
In the above, the 3 `f` z pairing gets evaluated first because
it’s in the innermost parentheses. Right folds have to traverse
the list outside-in, but the folding itself starts from the end of
the list.
It’s hard to see this with arithmetic functions that are as-
sociative, such as addition, but it’s an important point to un-
derstand, so we’ll run through some diﬀerent examples. Let’s
start by using an arithmetic operation that isn’t associative:
Prelude> foldr (^) 2 [1..3]
1
Prelude> foldl (^) 2 [1..3]
64
This time we can see clearly that we got diﬀerent results, and

CHAPTER 10. DATA STRUCTURE ORIGAMI 552
that diﬀerence results from the way the functions associate.
Here’s a breakdown:
-- if you want to follow along,
-- use paper and not the REPL.
foldr(^)2[1..3]
(1^(2^(3^2)))
(1^(2^9))
1^512
1
Contrast that with this:
foldl(^)2[1..3]
((2^1)^2)^3
(2^2)^3
4^3
64
In this next set of comparisons, we will demonstrate the
eﬀect of associativity on argument order by folding the list
into a new list, like this:
Prelude> foldr (:) [] [1..3]
[1,2,3]
Prelude> foldl (flip (:)) [] [1..3]
[3,2,1]

CHAPTER 10. DATA STRUCTURE ORIGAMI 553
We must use flipwithfoldl. Let’s examine why.
Like a right fold, a left fold cannot perform magic and go to
the end of the list instantly; it must start from the beginning
of the list. However, the parentheses dictate how our code
evaluates. The type of the argument to the folding function
changes in addition to the associativity:
foldr::(a->b->b)->b->[a]->b
-- [1] [2] [3]
foldl::(b->a->b)->b->[a]->b
-- [4] [5] [6]
1.The parameter of type 𝑎represents one of the list element
arguments the folding function of foldris applied to.
2.The parameter of type 𝑏will either be the start value or
the result of the fold accumulated so far, depending on
how far you are into the fold.
3.The final result of having combined the list element and
the start value or fold so far to compute the fold.
4.The start value or fold accumulated so far is the first ar-
gument to foldl’s folding function.
5.The list element is the second argument to foldl’s folding
function.

CHAPTER 10. DATA STRUCTURE ORIGAMI 554
6.The final result of foldl’s fold function is of type 𝑏, like
that offoldr.
The type of (:)requires that a value be the first argument
and a list be the second argument:
(:) :: a -> [a] -> [a]
So the value is prepended, or “consed onto,” the front of
that list.
In the following examples, the tilde means “is equivalent or
equal to.” If we write a right fold that has the cons constructor
as our𝑓and the empty list as our 𝑧, we get:
-- foldr f z [1, 2, 3]
-- f ~ (:); z ~ []
-- Run it in your REPL. It'll return True.
foldr (:)[](1:2:3:[])
==1:(2:(3:[]))
The consing process for foldrmatches the type signature
for(:). It also reproduces the same list because we’re replacing
the cons constructors with cons constructors and the null list
with null list. However, for it to be identical, it also has to be
right associative.
Doing the same with foldldoes not produce the same result.
When using foldl, the result we’ve accumulated so far is the

CHAPTER 10. DATA STRUCTURE ORIGAMI 555
first argument instead of the list element. This is opposite of
what(:)expects if we’re accumulating a list. Trying to fold
the identity of the list as above but with foldlwould give us a
type error because the reconstructing process for foldlwould
look like this:
foldlf z [1,2,3]
-- f ~ (:); z ~ []
-- (((z `f` 1) `f` 2) `f` 3)
((([]:1):2):3)
That won’t work because the 𝑧is an empty list and the 𝑓is
cons, so we have the order of arguments backwards for cons.
Enterflip, whose job is to take backwards arguments and turn
that frown upside down. It will flip each set of arguments
around for us, like this:
foldlf z [1,2,3]
-- f ~ (flip (:)); z ~ []
-- (((z `f` 1) `f` 2) `f` 3)
f=flip (:)
((([]`f`1) `f`2) `f`3)
(([1] `f`2) `f`3)
([2,1] `f`3)
[3,2,1]

CHAPTER 10. DATA STRUCTURE ORIGAMI 556
Evenwhenwe’vesatisfiedthetypesbyflippingthingsaround,
the left-associating nature of foldlleads to a diﬀerent result
from that of foldr.
For the next set of comparisons, we’re going to use a func-
tion called constthat takes two arguments and always returns
the first one. When we fold constover a list, it will take as its
first pair of arguments the accvalue and a value from the list
— which value it takes first depends on which type of fold it is.
We’ll show you how it evaluates for the first example:
Prelude> foldr const 0 [1..5]
(const 1 _)
1
Sinceconstdoesn’t evaluate its second argument the rest
of the fold is never evaluated. The underscore represents the
rest of the unevaluated fold. Now, let’s look at the eﬀect of
flipping the arguments. The 0 result is because zero is our
accumulator value here, so it’s the first (or last) value of the
list:
Prelude> foldr (flip const) 0 [1..5]
0
Next let’s look at what happens when we use the same func-
tions but this time with foldl. Take a few moments to under-
stand the evaluation process that leads to these results:

CHAPTER 10. DATA STRUCTURE ORIGAMI 557
Prelude> foldl (flip const) 0 [1..5]
5
Prelude> foldl const 0 [1..5]
0
This is the eﬀect of left associativity. The spine traversal
happens in the same order in a left or right fold — it must, be-
cause of the way lists are defined. Depending on your folding
function, a left fold can lead to a diﬀerent result than a right
fold of the same.
Exercises: Understanding Folds
1.foldr(*)1[1..5]
will return the same result as which of the following:
a)flip(*)1[1..5]
b)foldl(flip (*))1[1..5]
c)foldl(*)1[1..5]
2.Write out the evaluation steps for
foldl(flip (*))1[1..3]
3.One diﬀerence between foldrandfoldlis:
a)foldr, but not foldl, traverses the spine of a list from
right to left

CHAPTER 10. DATA STRUCTURE ORIGAMI 558
b)foldr, but not foldl, always forces the rest of the fold
c)foldr, but not foldl, associates to the right
d)foldr, but not foldl, is recursive
4.Folds are catamorphisms, which means they are generally
used to
a)reduce structure
b)expand structure
c)render you catatonic
d)generate infinite data structures
5.The following are simple folds very similar to what you’ve
already seen, but each has at least one error. Please fix
them and test in your REPL:
a)foldr(++) ["woot","WOOT","woot"]
b)foldrmax[]"fear is the little death"
c)foldrandTrue[False,True]
d)This one is more subtle than the previous. Can it ever
return a diﬀerent answer?
foldr(||)True[False,True]
e)foldl((++).show)""[1..5]
f)foldrconst'a'[1..5]

CHAPTER 10. DATA STRUCTURE ORIGAMI 559
g)foldrconst0"tacos"
h)foldl(flip const) 0"burritos"
i)foldl(flip const) 'z'[1..5]
Unconditional spine recursion
An important diﬀerence between foldrandfoldlis that a left
fold has the successive steps of the fold as its first argument.
The next recursion of the spine isn’t intermediated by the
folding function as it is in foldr, which also means recursion of
the spine is unconditional. Having a function that doesn’t force
evaluation of either of its arguments won’t change anything.
Let’s review const:
Prelude> const 1 undefined
1
Prelude> (flip const) 1 undefined
*** Exception: Prelude.undefined
Prelude> (flip const) undefined 1
1
Now compare:
Prelude> let xs = [1..5] ++ undefined
Prelude> foldr const 0 xs
1
Prelude> foldr (flip const) 0 xs

CHAPTER 10. DATA STRUCTURE ORIGAMI 560
*** Exception: Prelude.undefined
Prelude> foldl const 0 xs
*** Exception: Prelude.undefined
Prelude> foldl (flip const) 0 xs
*** Exception: Prelude.undefined
However, while foldlunconditionally evaluates the spine
you can still selectively evaluate the values in the list. This will
throw an error because the bottom is part of the spine and
foldlmust evaluate the spine:
Prelude> let xs = [1..5] ++ undefined
Prelude> foldl (\_ _ -> 5) 0 xs
*** Exception: Prelude.undefined
But this is OK because bottom is a value here:
Prelude> let xs = [1..5] ++ [undefined]
Prelude> foldl (\_ _ -> 5) 0 xs
5
This feature means that foldlis generally inappropriate
with lists that are or could be infinite, but the combination of
the forced spine evaluation with nonstrictness means that it is
also usually inappropriate even for long lists, as the forced eval-
uation of the spine aﬀects performance negatively. Because

CHAPTER 10. DATA STRUCTURE ORIGAMI 561
foldlmust evaluate its whole spine before it starts evaluating
values in each cell, it accumulates a pile of unevaluated values
as it traverses the spine.
In most cases, when you need a left fold, you should use
foldl'. This function, called “fold-l-prime,” works the same
except it is strict. In other words, it forces evaluation of the
values inside cons cells as it traverses the spine, rather than
accumulating unevaluated expressions for each element of
the list. The strict evaluation here means it has less negative
eﬀect on performance over long lists.
10.6 How to write fold functions
When we write folds, we begin by thinking about what our
start value for the fold is. This is usually the identity value for
the function. When we sum the elements of a list, the identity
of summation is 0. When we multiply the elements of the list,
the identity is 1. This start value is also our fallback in case the
list is empty.
Next we consider our arguments. A folding function takes
two arguments, 𝑎and𝑏, where 𝑎is going to always be one of
the elements in the list and 𝑏is either the start value or the
value accumulated as the list is being processed.
Let’s say we want to write a function to take the first three
letters of each String value in a list of strings and concatenate

CHAPTER 10. DATA STRUCTURE ORIGAMI 562
that result into a final String . The type of the right fold for lists
is:
foldr::(a->b->b)->b->[a]->b
First, we’ll set up the beginnings of our expression:
foldr(\a b->undefined) []
["Pizza","Apple","Banana" ]
We used an empty list as the start value, but since we plan to
return a String as our result, we could be a little more explicit
about our intent to build a String and make a small syntactic
change:
foldr(\a b->undefined) ""
["Pizza","Apple","Banana" ]
Of course, because a String is a list, these are the same value:
Prelude> "" == []
True
But""signals intent with respect to the types involved:
Prelude> :t ""
"" :: [Char]
Prelude> :t []
[] :: [t]

CHAPTER 10. DATA STRUCTURE ORIGAMI 563
Moving along, we next want to work on the function. We
already know how to take the first three elements from a list
and we can reuse this for String :
foldr(\a b->take3a)""
["Pizza","Apple","Banana" ]
Now this will already typecheck and work, but it doesn’t
match the semantics we asked for:
Prelude> :{
*Main| let pab =
*Main| ["Pizza", "Apple", "Banana"]
*Main| :}
Prelude> foldr (\a b -> take 3 a) "" pab
"Piz"
Prelude> foldl (\b a -> take 3 a) "" pab
"Ban"
We’re only getting the first three letters of the first or the
last string, depending on whether we did a right or left fold.
Note the argument naming order due to the diﬀerence in the
types of foldrandfoldl:
foldr::(a->b->b)->b->[a]->b
foldl::(b->a->b)->b->[a]->b

CHAPTER 10. DATA STRUCTURE ORIGAMI 564
The problem here is that right now we’re not folding the
list. We’re only mapping our take 3 over the list and selecting
the first or last result:
Prelude> map (take 3) pab
["Piz","App","Ban"]
Prelude> head $ map (take 3) pab
"Piz"
Prelude> last $ map (take 3) pab
"Ban"
So let us make this a proper fold and accumulate the result
by making use of the 𝑏argument. Remember the 𝑏is the
start value. Technically we could use concat on the result of
having mapped take 3 over the list (or its reverse, if we want
to simulate foldl):
Prelude> concat $ map (take 3) pab
"PizAppBan"
Prelude> let rpab = reverse pab
Prelude> concat $ map (take 3) rpab
"BanAppPiz"
But we need an excuse to play with foldrandfoldl, so we’ll
pretend none of this happened!
Prelude> let f = (\a b -> take 3 a ++ b)

CHAPTER 10. DATA STRUCTURE ORIGAMI 565
Prelude> foldr f "" pab
"PizAppBan"
Prelude> let f' = (\b a -> take 3 a ++ b)
Prelude> foldl f' "" pab
"BanAppPiz"
Here we concatenated the result of having taken three el-
ements from the string value in our input list onto the front
of the string we’re accumulating. If we want to be explicit, we
can assert types for the values:
Prelude> :{
*Prelude| let f a b = take 3
*Prelude| (a :: String) ++
*Prelude| (b :: String)
*Prelude| :}
Prelude> foldr f "" pab
"PizAppBan"
Ifweassertsomethingthatisn’ttrue, thetypecheckercatches
us:
Prelude> :{
*Prelude| let f a b = take 3 (a :: String)
*Prelude| ++ (b :: [String])
*Prelude| :}

CHAPTER 10. DATA STRUCTURE ORIGAMI 566
<interactive>:12:42:
Couldn't match type ‘Char’ with ‘[Char]’
Expected type: [String]
Actual type: [Char]
In the second argument of ‘(++)’,
namely ‘(b :: [String])’
In the expression:
take 3 (a :: String) ++ (b :: [String])
This can be useful for checking that your mental model of
the code is accurate.
Exercises: Database Processing
Write the following functions for processing this data.
importData.Time
dataDatabaseItem =DbString String
|DbNumber Integer
|DbDate UTCTime
deriving (Eq,Ord,Show)

CHAPTER 10. DATA STRUCTURE ORIGAMI 567
theDatabase ::[DatabaseItem ]
theDatabase =
[DbDate(UTCTime
(fromGregorian 191151)
(secondsToDiffTime 34123))
,DbNumber 9001
,DbString "Hello, world!"
,DbDate(UTCTime
(fromGregorian 192151)
(secondsToDiffTime 34123))
]
1.Write a function that filters for DbDate values and returns
a list of the UTCTime values inside them.
filterDbDate ::[DatabaseItem ]
->[UTCTime]
filterDbDate =undefined
2.Write a function that filters for DbNumber values and returns
a list of the Integer values inside them.
filterDbNumber ::[DatabaseItem ]
->[Integer]
filterDbNumber =undefined
3.Write a function that gets the most recent date.

CHAPTER 10. DATA STRUCTURE ORIGAMI 568
mostRecent ::[DatabaseItem ]
->UTCTime
mostRecent =undefined
4.Write a function that sums all of the DbNumber values.
sumDb::[DatabaseItem ]
->Integer
sumDb=undefined
5.Write a function that gets the average of the DbNumber val-
ues.
-- You'll probably need to use fromIntegral
-- to get from Integer to Double.
avgDb::[DatabaseItem ]
->Double
avgDb=undefined
10.7 Folding and evaluation
What diﬀerentiates foldrandfoldlis associativity. The right
associativity of foldrmeans the folding function evaluates
from the innermost cons cell to the outermost (the head). On
the other hand, foldlrecurses unconditionally to the end of the

CHAPTER 10. DATA STRUCTURE ORIGAMI 569
list through self-calls and then the folding function evaluates
from the outermost cons cell to the innermost:
Prelude> let rcf = foldr (:) []
Prelude> let xs = [1, 2, 3] ++ undefined
Prelude> take 3 $ rcf xs
[1,2,3]
Prelude> let lcf = foldl (flip (:)) []
Prelude> take 3 $ lcf xs
*** Exception: Prelude.undefined
Let’s dive into our constexample a little more carefully:
foldr const 0 [1..5]
Withfoldr, you’ll evaluate const 1 (...) , butconstignores
the rest of the fold that would have occurred from the end of
the list up to the number 1, so this returns 1 without having
evaluated any more of the values or the spine. One way you
could examine this for yourself would be:
Prelude> foldr const 0 ([1] ++ undefined)
1
Prelude> head ([1] ++ undefined)
1
Prelude> tail ([1] ++ undefined)
*** Exception: Prelude.undefined

CHAPTER 10. DATA STRUCTURE ORIGAMI 570
Similarly for foldl:
foldl (flip const) 0 [1..5]
Herefoldlwill recurse to the final cons cell, evaluate (flip
const) (...) 5 , ignore the rest of the fold that would occur
from the beginning up to the number 5, and return 5.
The relationship between foldrandfoldlis such that:
foldr f z xs =
foldl (flip f) z (reverse xs)
Butonlyfor finite lists! Consider:
Prelude> let xs = repeat 0 ++ [1,2,3]
Prelude> foldr const 0 xs
0
Prelude> let xs' = repeat 1 ++ [1,2,3]
Prelude> let rxs = reverse xs'
Prelude> foldl (flip const) 0 rxs
^CInterrupted.
-- ^^ bottom.
If we flip our folding function 𝑓and reverse the list 𝑥𝑠,foldr
andfoldlwill return the same result:
Prelude> let xs = [1..5]
Prelude> foldr (:) [] xs

CHAPTER 10. DATA STRUCTURE ORIGAMI 571
[1,2,3,4,5]
Prelude> foldl (flip (:)) [] xs
[5,4,3,2,1]
Prelude> foldl (flip (:)) [] (reverse xs)
[1,2,3,4,5]
Prelude> reverse $ foldl (flip (:)) [] xs
[1,2,3,4,5]
10.8 Summary
We presented a lot of material in this chapter. You might be
feeling a little weary of folds right now. So what’s the executive
summary?
foldr
1.The rest of the fold (recursive invocation of foldr) is an
argument to the folding function you passed to foldr. It
doesn’t directly self-call as a tail-call like foldl. You could
think of it as alternating between applications of foldrand
your folding function 𝑓. The next invocation of foldris
conditional on 𝑓having asked for more of the results of
having folded the list. That is:
foldr::(a->b->b)->b->[a]->b
-- ^

CHAPTER 10. DATA STRUCTURE ORIGAMI 572
That𝑏we’re pointing at in (a -> b -> b) istherestofthefold.
Evaluating that evaluates the next application of foldr.
2.Associates to the right.
3.Works with infinite lists. We know this because:
Prelude> foldr const 0 [1..]
1
4.Is a good default choice whenever you want to transform
data structures, be they finite or infinite.
foldl
1.Self-calls (tail-call) through the list, only beginning to
produce values after reaching the end of the list.
2.Associates to the left.
3.Cannot be used with infinite lists. Try the infinite list
example earlier and your REPL will hang.
4.Is nearly useless and should almost always be replaced
withfoldl' for reasons we’ll explain later when we talk
about writing efficient Haskell programs.

CHAPTER 10. DATA STRUCTURE ORIGAMI 573
10.9 Scans
Scans, which we have mentioned above, work similarly to
maps and also to folds. Like folds, they accumulate values
instead of keeping the list’s individual values separate. Like
maps, they return a list of results. In this case, the list of results
shows the intermediate stages of evaluation, that is, the values
that accumulate as the function is doing its work.
Scans are not used as frequently as folds, and once you
understand the basic mechanics of folding, there isn’t a whole
lot new to understand. Still, it is useful to know about them
and get an idea of why you might need them.3
First, let’s take a look at the types. We’ll do a direct com-
parison of the types of folds and scans so the diﬀerence is
clear:
foldr :: (a -> b -> b) -> b -> [a] -> b
scanr :: (a -> b -> b) -> b -> [a] -> [b]
foldl :: (b -> a -> b) -> b -> [a] -> b
scanl :: (b -> a -> b) -> b -> [a] -> [b]
The primary diﬀerence is that the final result is a list (folds
canreturn a list as a result as well, but they don’t always). This
3The truth is scans are not used often, but there are times when you want to fold
a function over a list and return a list of the intermediate values that you can then use
as input to some other function. For a particularly elegant use of this, please see Chris
Done’s blog post about this solution to the waterfall problem at http://chrisdone.com/
posts/twitter-problem-loeb .

CHAPTER 10. DATA STRUCTURE ORIGAMI 574
means that they are not catamorphisms and, in an important
sense, aren’t folds at all. But no matter! The type signatures
are similar, and the routes of spine traversal and evaluation
are similar. This does mean that you can use scans in places
that you can’t use a fold, precisely because you return a list of
results rather than reducing the spine of the list.
The results that scans produce can be represented like this:
scanr (+) 0 [1..3]
[1 + (2 + (3 + 0)), 2 + (3 + 0), 3 + 0, 0]
[6, 5, 3, 0]
scanl (+) 0 [1..3]
[0, 0 + 1,0 + 1 + 2, 0 + 1 + 2 + 3]
[0, 1, 3, 6]

CHAPTER 10. DATA STRUCTURE ORIGAMI 575
scanl(+)1[1..3]
-- unfolding the
-- definition of scanl
=[1,1+1
, (1+1)+2
, ((1+1)+2)+3
]
-- evaluating addition
=[1,2,4,7]
Then to make this more explicit and properly equational,
we can follow along with how scanlexpands for this expression
based on the definition. First, we must see how scanlis defined.
We’re going to show you a version of it from a slightly older
baselibrary for GHC Haskell. The diﬀerences don’t change
anything important for us here:
scanl::(a->b->a)->a->[b]->[a]
scanlf q ls=
q:(caselsof
[]->[]
x:xs->scanl f (f q x) xs)
In an earlier chapter, we wrote a recursive function that

CHAPTER 10. DATA STRUCTURE ORIGAMI 576
returned the nth Fibonacci number to us. You can use a scan
function to return a list of Fibonacci numbers. We’re going
to do this in a source file because this will, in this state, return
an infinite list (feel free to try loading it into your REPL and
running it, but be quick with the ctrl-c):
fibs=1:scanl (+)1fibs
We start with a value of 1 and cons that onto the front of the
list generated by our scan. The list itself has to be recursive
because, as we saw previously, the idea of Fibonacci numbers
is that each one is the sum of the previous two in the sequence;
scanning the results of (+)over a nonrecursive list of numbers
whose start value is 1 would give us this:
scanl (+) 1 [1..3]
[1, 1 + 1, (1 + 1) + 2, ((1 + 1) + 2) + 3]
[1,2,4,7]
instead of the [1, 1, 2, 3, 5...] that we’re looking for.
Getting the fibonacci number we want
But we don’t really want an infinite list of Fibonacci numbers;
that isn’t very useful. We need a method to either take some
number of elements from that list or find the 𝑛th element as
we had done before. Fortunately, that’s the easy part. We’ll

CHAPTER 10. DATA STRUCTURE ORIGAMI 577
use the “bang bang” operator, !!, to find the 𝑛th element. This
operator is a way to index into a list, and indexing in Haskell
starts from zero. That is, the first value in your list is indexed
as zero. But otherwise the operator is straightforward:
(!!) :: [a] -> Int -> a
It needs a list as its first argument, an Intas its second argu-
ment and it returns one element from the list. Which item it
returns is the value that is in the 𝑛th spot where 𝑛is ourInt.
We will modify our source file:
fibs =1:scanl (+)1fibs
fibsNx=fibs!!x
Once we load the file into our REPL, we can use fibsNto
return the 𝑛th element of our scan:
Prelude> fibsN 0
1
Prelude> fibsN 2
2
Prelude> fibsN 6
13
Now you can modify your source code to use the takeor
takeWhile functions or to filter it in any way you like. One

CHAPTER 10. DATA STRUCTURE ORIGAMI 578
note: filtering without also taking won’t work too well, because
you’re still getting an infinite list. It’s a filtered infinite list, sure,
but still infinite.
Scans Exercises
1.Modify your fibsfunction to only return the first 20 Fi-
bonacci numbers.
2.Modify fibsto return the Fibonacci numbers that are less
than 100.
3.Try to write the factorial function from Recursion as a
scan. You’ll want scanlagain, and your start value will be
1. Warning: this will also generate an infinite list, so you
may want to pass it through a takefunction or similar.
10.10 Chapter Exercises
Warm-up and review
For the following set of exercises, you are not expected to use
folds. These are intended to review material from previous
chapters. Feelfreetouseanysyntaxorstructurefromprevious
chapters that seems appropriate.
1.Given the following sets of consonants and vowels:

CHAPTER 10. DATA STRUCTURE ORIGAMI 579
stops = "pbtdkg"
vowels = "aeiou"
a)Write a function that takes inputs from stopsand
vowels and makes 3-tuples of all possible stop-vowel-
stop combinations. These will not all correspond to
real words in English, although the stop-vowel-stop
pattern is common enough that many of them will.
b)Modify that function so that it only returns the com-
binations that begin with a p.
c)Now set up lists of nouns and verbs (instead of stops
and vowels) and modify the function to make tuples
representing possible noun-verb-noun sentences.
2.What does the following mystery function do? What is
its type? Try to get a good sense of what it does before
you test it in the REPL to verify it.
seekritFunc x=
div (sum (map length (words x)))
(length (words x))
3.We’d really like the answer to be more precise. Can you
rewrite that using fractional division?

CHAPTER 10. DATA STRUCTURE ORIGAMI 580
Rewriting functions using folds
In the previous chapter, you wrote these functions using direct
recursion over lists. The goal now is to rewrite them using
folds. Where possible, to gain a deeper understanding of
folding, try rewriting the fold version so that it is point-free.
Point-free versions of these functions written with a fold
should look like:
myFunc=foldr f z
So for example with the andfunction:
-- Again, this type will be less
-- reusable than the one in GHC 7.10
-- and newer. Don't worry.
-- direct recursion, not using (&&)
myAnd::[Bool]->Bool
myAnd[]=True
myAnd(x:xs)=
ifx==False
thenFalse
elsemyAnd xs

CHAPTER 10. DATA STRUCTURE ORIGAMI 581
-- direct recursion, using (&&)
myAnd::[Bool]->Bool
myAnd[]=True
myAnd(x:xs)=x&&myAnd xs
-- fold, not point-free
-- in the folding function
myAnd::[Bool]->Bool
myAnd=foldr
(\a b->
ifa==False
thenFalse
elseb)True
-- fold, both myAnd and the folding
-- function are point-free now
myAnd::[Bool]->Bool
myAnd=foldr (&&)True
The goal here is to converge on the final version where
possible. You don’t need to write all variations for each ex-
ample, but the more variations you write, the deeper your
understanding of these functions will become.
1.myOrreturns Trueif anyBoolin the list is True.

CHAPTER 10. DATA STRUCTURE ORIGAMI 582
myOr::[Bool]->Bool
myOr=undefined
2.myAnyreturns Trueifa -> Bool applied to any of the values
in the list returns True.
myAny::(a->Bool)->[a]->Bool
myAny=undefined
Example for validating myAny:
Prelude> myAny even [1, 3, 5]
False
Prelude> myAny odd [1, 3, 5]
True
3.Write two versions of myElem. One version should use
folding and the other should use any.
myElem::Eqa=>a->[a]->Bool
Prelude> myElem 1 [1..10]
True
Prelude> myElem 1 [2..10]
False
4.Implement myReverse, don’t worry about trying to make
it lazy.

CHAPTER 10. DATA STRUCTURE ORIGAMI 583
myReverse ::[a]->[a]
myReverse =undefined
Prelude> myReverse "blah"
"halb"
Prelude> myReverse [1..5]
[5,4,3,2,1]
5.WritemyMapin terms of foldr. It should have the same
behavior as the built-in map.
myMap::(a->b)->[a]->[b]
myMap=undefined
6.WritemyFilter in terms of foldr. It should have the same
behavior as the built-in filter .
myFilter ::(a->Bool)->[a]->[a]
myFilter =undefined
7.squish flattens a list of lists into a list
squish::[[a]]->[a]
squish=undefined
8.squishMap maps a function over a list and concatenates the
results.

CHAPTER 10. DATA STRUCTURE ORIGAMI 584
squishMap ::(a->[b])->[a]->[b]
squishMap =undefined
Prelude> squishMap (\x -> [1, x, 3]) [2]
[1,2,3]
Prelude> let f x = "WO " ++ [x] ++ " OT "
Prelude> squishMap f "blah"
"WO b OT WO l OT WO a OT WO h OT "
9.squishAgain flattens a list of lists into a list. This time re-use
thesquishMap function.
squishAgain ::[[a]]->[a]
squishAgain =undefined
10.myMaximumBy takes a comparison function and a list and
returns the greatest element of the list based on the last
value that the comparison returned GTfor.
myMaximumBy ::(a->a->Ordering )
->[a]
->a
myMaximumBy =undefined
Prelude> myMaximumBy (\_ _ -> GT) [1..10]
1
Prelude> myMaximumBy (\_ _ -> LT) [1..10]

CHAPTER 10. DATA STRUCTURE ORIGAMI 585
10
Prelude> myMaximumBy compare [1..10]
10
11.myMinimumBy takes a comparison function and a list and
returns the least element of the list based on the last value
that the comparison returned LTfor.
myMinimumBy ::(a->a->Ordering )
->[a]
->a
myMinimumBy =undefined
Prelude> myMinimumBy (\_ _ -> GT) [1..10]
10
Prelude> myMinimumBy (\_ _ -> LT) [1..10]
1
Prelude> myMinimumBy compare [1..10]
1
10.11 Definitions
1.Afoldis a higher-order function which, given a function
to accumulate the results and a recursive data structure,
returns the built up value. Usually a “start value” for the
accumulation is provided along with a function that can

CHAPTER 10. DATA STRUCTURE ORIGAMI 586
combine the type of values in the data structure with the
accumulation. The term fold is typically used with ref-
erence to collections of values referenced by a recursive
datatype. For a generalization of “breaking down struc-
ture”, see catamorphism .
2.Acatamorphism is a generalization of folds to arbitrary
datatypes. Where a fold allows you to break down a list
into an arbitrary datatype, a catamorphism is a means of
breaking down the structure of any datatype. The bool
:: a -> a -> Bool -> a function in Data.Bool is an example
of a simple catamorphism for a simple, non-collection
datatype. Similarly, maybe :: b -> (a -> b) -> Maybe a ->
bis the catamorphism for Maybe. See if you can notice a
pattern:

CHAPTER 10. DATA STRUCTURE ORIGAMI 587
dataBool=False|True
bool::a->a->Bool->a
dataMaybea=Nothing |Justa
maybe::b->(a->b)->Maybea->b
dataEithera b=Lefta|Rightb
either::(a->c)
->(b->c)
->Eithera b
->c
3.Atail call is the final result of a function. Some examples
of tail calls in Haskell functions:
fx y z=h (subFunction x y z)
wheresubFunction x y z =g x y z
-- the ``tail call'' is
-- h (subFunction x y z)
-- or more precisely, h.
4.Tail recursion is a function whose tail calls are recursive
invocations of itself. This is distinguished from functions
that call other functions in their tail call.
fx y z=h (subFunction x y z)
wheresubFunction x y z =g x y z

CHAPTER 10. DATA STRUCTURE ORIGAMI 588
The above is not tail recursive, calls ℎ, not itself.
fx y z=h (f (x -1) y z)
Still not tail recursive. 𝑓is invoked again but not in the
tail call of 𝑓; it’s an argument to the tail call, ℎ:
fx y z=f (x-1) y z
This is tail recursive. 𝑓is calling itself directly with no
intermediaries.
foldrf z[]=z
foldrf z (x:xs)=f x (foldr f z xs)
Not tail recursive, we give up control to the combining
function 𝑓before continuing through the list. foldr’s re-
cursive calls will bounce between foldrand𝑓.
foldlf z[]=z
foldlf z (x:xs)=foldl f (f z x) xs
Tail recursive. foldlinvokes itself recursively. The com-
bining function is only an argument to the recursive fold.

CHAPTER 10. DATA STRUCTURE ORIGAMI 589
10.12 Follow-up resources
1.Haskell Wiki. Fold.
https://wiki.haskell.org/Fold
2.Richard Bird. Sections 4.5 and 4.6 of Introduction to
Functional Programming using Haskell (1998).
3.Antoni Diller. Introduction to Haskell.
4.Graham Hutton. A tutorial on the universality and ex-
pressiveness of fold.
http://www.cs.nott.ac.uk/~gmh/fold.pdf

Chapter 11
Algebraic datatypes
The most depressing
thing about life as a
programmer, I think, is if
you’re faced with a chunk
of code that either
someone else wrote or,
worse still, you wrote
yourself but no longer
dare to modify. That’s
depressing.
Simon Peyton Jones
590

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 591
11.1 Algebraic datatypes
We have spent a lot of time talking about datatypes already, so
you may think we’ve covered everything that needs to be said
about those. This chapter’s purpose is ultimately to explain
how to construct your own datatypes in Haskell. Writing your
own datatypes can help you leverage some of Haskell’s most
powerful features — pattern matching, type checking, and
inference — in a way that makes your code more concise and
safer. But to understand that, first we need to explain the dif-
ferences among datatypes more fully and understand what it
means when we say datatypes are algebraic .
A type can be thought of as an enumeration of constructors
that have zero or more arguments.1We will return to this
description throughout the chapter, each time emphasizing a
diﬀerent portion of it.
Haskell oﬀers sum types, product types, product types with
record syntax, type aliases (for example, String is a type alias
for[Char] ), and a special datatype called a newtype that provides
for a diﬀerent set of options and constraints from either type
synonyms or data declarations. We will explain each of these
in detail in this chapter and show you how to exploit them for
maximum utility and type safety.
This chapter will:
1This description, slightly edited for our purposes, was proposed by Orah Kittrell in
the#haskell-beginners IRC channel.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 592
•explain the “algebra” of algebraic datatypes;
•analyze the construction of data constructors;
•spell out when and how to write your own datatypes;
•clarify usage of type synonyms and newtype ;
•introduce kinds.
11.2 Data declarations review
We often want to create custom datatypes for structuring and
describing the data we are processing. Doing so can help you
analyze your problem by allowing you to focus first on how
youmodel the domain before you begin thinking about how
you write computations that solve your problem. It can also
make your code easier to read and use because it lays the
domain model out clearly.
In order to write your own types, though, you must under-
stand the way datatypes are constructed in more detail than
we’ve covered so far. Let’s begin with a review of the important
parts of datatypes, using the data declarations for Booland lists:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 593
dataBool=False|True
-- [1] [2] [3] [4] [5] [6]
data[]a=[ ]|a:[a]
-- [ 7 ] [8] [9]
1.Keyword datato signal that what follows is a data declara-
tion, or a declaration of a datatype.
2.Type constructor (with no arguments).
3.Equals sign divides the type constructor from its data
constructors.
4.Data constructor. In this case, a data constructor that takes
no arguments and so is called a nullary constructor. This
is one of the possible values of this type that can show up
in term-level code.
5.The pipe denotes a sum type which indicates a logical
disjunction (colloquially, or) in what values can have that
type.
6.Constructor for the value True, another nullary construc-
tor.
7.Type constructor with an argument. An empty list has
to be applied to an argument in order to become a list

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 594
ofsomething . Here the argument is a polymorphic type
variable, so the list’s argument can be of diﬀerent types.
8.Data constructor for the empty list.
9.Data constructor that takes two arguments: an 𝑎and also
a[a].
When we talk about a data declaration, we are talking about
the definition of the entiretype. If wethink of a type as “anenu-
meration of constructors that have zeroor more arguments,”
thenBoolis an enumeration of two possible constructors, each
of which takes zeroarguments, while the type constructor []
enumerates two possible constructors and one of them takes
twoarguments. The pipe denotes what we call a sum type , a
type that has more than one constructor inhabiting it.
In addition to sum types, Haskell also has product types , and
we’ll talk more about those in a bit. The data constructors in
product types have more than one parameter. But first, let’s
turn our attention to the meaning of the word constructors .
11.3 Data and type constructors
There are two kinds of constructors in Haskell: type construc-
tors and data constructors. Type constructors are used only
at the type level, in type signatures and typeclass declarations
and instances. Types are static and resolve at compile time.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 595
Data constructors construct the values at term level, values
you can interact with at runtime. We call them constructors
because they define a means of creating or building a type or
a value.
Although the term constructor is often used to describe all
type constructors and data constructors, we can make a dis-
tinction between constants andconstructors . Type and data con-
structors that take no arguments are constants. They can only
store a fixed type and amount of data. So, in the Booldatatype,
for example, Boolis a type constant, a concrete type that isn’t
waiting for any additional information in the form of an argu-
ment in order to be fully realized as a type. It enumerates two
values that are also constants, TrueandFalse, because they take
no arguments. While we call TrueandFalse“data constructors,”
in fact since they take no arguments, their value is already es-
tablished and not being constructed in any meaningful sense.
However, sometimes we need the flexibility of allowing dif-
ferent types or amounts of data to be stored in our datatypes.
For those times, type and data constructors may be parame-
terized. When a constructor takes an argument, then it’s like a
function in at least one sense — it must be applied to become a
concrete type or value. The following datatypes are pseudony-
mous versions of real datatypes in Haskell. We’ve given them
pseudonyms because we want to focus on the syntax, not the
semantics, for now.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 596
dataTrivial =Trivial'
-- [1] [2]
dataUnaryTypeCon a=UnaryValueCon a
-- [3] [4]
1.Here the type constructor Trivial is like a constant value,
but at the type level. It takes no arguments and is thus
nullary . The Haskell Report calls these typeconstants to dis-
tinguish them from type constructors that take arguments.
2.The data constructor Trivial' is also like a constant value,
but it exists in value, term, or runtime space. These are
not three diﬀerent things, but three diﬀerent words for
the same space that types serve to describe.
3.UnaryTypeCon is a type constructor of one argument. It’s a
constructor awaiting a type constant to be applied to, but
it has no behavior in the sense that we think of functions
as having. Such type-level functions exist but are not
covered in this book.2
4.UnaryValueCon is a data constructor of one argument await-
ing a value to be applied to. Again, it doesn’t behave like
2If you’re interested in learning about this topic, Brent Yorgey’s blog posts about type
families and functional dependencies are a good place to start. https://byorgey.wordpress.
com/2010/06/29/typed-type-level-programming-in-haskell-part-i-functional-dependencies/

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 597
a term-level function in the sense of performing an oper-
ation on data. It’s more like a box to put values into. Be
careful with the box/container analogy as it will betray
you later — not all type arguments to constructors have
value-level witnesses! Some are phantom .
Each of these datatypes only enumerates one data construc-
tor. Whereas Trivial' is the only possible concrete value for
typeTrivial ,UnaryValueCon could show up as diﬀerent literal
values at runtime, depending on what type of 𝑎it is applied to.
Think back to the list datatype: at the type level, you have a :
[a]where the 𝑎is a variable. At the term level, in your code,
that will be applied to some type of values and become, for
example, [Char] or[Integer] (or list of whatever other concrete
type — obviously the set of possible lists is large).
11.4 Type constructors and kinds
Let’s look again at the list datatype:
data[]a=[]|a:[a]
This must be applied to a concrete type before you have a
list. We can see the parallel with functions when we look at
thekindsignature.
Kinds are the types of types, or types one level up. We
represent kinds in Haskell with *. We know something is a

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 598
fully applied, concrete type when it is represented as *. When
it is* -> * , it, like a function, is still waiting to be applied.
Compare the following:
Prelude> let f = not True
Prelude> :t f
f :: Bool
Prelude> let f x = x > 3
Prelude> :t f
f :: (Ord a, Num a) => a -> Bool
The first 𝑓takes no arguments and is not awaiting appli-
cation to anything in order to produce a value, so its type
signature is a concrete type — note the lack of a function ar-
row. But the second 𝑓is awaiting application to an 𝑥so its type
signature has a function arrow. Once we apply it to a value, it
also has a concrete type:
Prelude> let f x = x > 3
Prelude> :t f 5
f 5 :: Bool
We query the kind signature of a type constructor (not a
data constructor) in GHCi with a :kindor:k. We see that kind
signatures give us similar information about type constructors:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 599
Prelude> :k Bool
Bool :: *
Prelude> :k [Int]
[Int] :: *
Prelude> :k []
[] :: * -> *
BothBooland[Int]are fully applied, concrete types, so their
kind signatures have no function arrows. That is, they are not
awaiting application to anything in order to be fully realized.
The kind of [], though, is * -> * because it still needs to be
applied to a concrete type before it is itself a concrete type.
This is what the constructor of “type constructor” is referring
to.
11.5 Data constructors and values
We mentioned a bit ago that the Haskell Report draws a distinc-
tion between type constants and type constructors . We can draw
a similar distinction between data constructors and constant
values.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 600
dataPugType =PugData
-- [1] [2]
dataHuskyType a=HuskyData
-- [3] [4]
dataDogueDeBordeaux doge=
-- [5]
DogueDeBordeaux doge
-- [6]
1.PugType is the type constructor, but it takes no arguments
so we can think of it as being a type constant . This is
how the Haskell Report refers to such types. This type
enumerates one constructor.
2.PugData is the only data constructor for the type PugType .
It also happens to be a constant value because it takes no
arguments and stands only for itself. For any function
that requires a value of type PugType , you know that value
will bePugData .
3.HuskyType is the type constructor and it takes a single para-
metrically polymorphic type variable as an argument. It
also enumerates one data constructor.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 601
4.HuskyData is the data constructor for HuskyType . Note that
the type variable argument 𝑎doesnotoccur as an argu-
ment to HuskyData or anywhere else after the =. That means
our type argument 𝑎isphantom , or, “has no witness.” We
will elaborate on this later. Here HuskyData is a constant
value, like PugData .
5.DogueDeBordeaux is a type constructor and has a single type
variable argument like HuskyType , but called 𝑑𝑜𝑔𝑒instead
of𝑎. Why? Because the names of variables don’t matter.
At any rate, this type also enumerates one constructor.
6.DogueDeBordeaux is the lone data constructor. It has the
same name as the type constructor, but they are not the
same thing. The 𝑑𝑜𝑔𝑒type variable in the type construc-
tor occurs also in the data constructor. Remember that,
because they are the same type variable, these must agree
with each other: 𝑑𝑜𝑔𝑒must equal 𝑑𝑜𝑔𝑒. If your type is
DogueDeBordeaux [Person] , you must necessarily have a list
ofPerson values contained in the DogueDeBordeaux value.
But because DogueDeBordeaux must be applied before it’s a
concrete value, its literal value at runtime can change:
Prelude> :t DogueDeBordeaux
DogueDeBordeaux :: doge
-> DogueDeBordeaux doge

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 602
We can query the type of the value (not the type construc-
tor but the data constructor — it can be confusing when
the type constructor and the data constructor have the
same name, but it’s pretty common to do that in Haskell
because the compiler doesn’t confuse type names with
value names the way we mortals do). It tells us that once
𝑑𝑜𝑔𝑒is bound to a concrete type, then this will be a value
of type DogueDeBordeaux doge . It isn’t a value yet, but it’s a
definition for how to construct a value of that type.
Here’s how to make a value of the type of each:
myPug=PugData ::PugType
myHusky ::HuskyType a
myHusky =HuskyData
myOtherHusky ::Numa=>HuskyType a
myOtherHusky =HuskyData
myOtherOtherHusky ::HuskyType [[[[Int]]]]
myOtherOtherHusky =HuskyData
-- no witness to the contrary ^
This will work because the value 10 agrees with the type
variable being bound to Int:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 603
myDoge::DogueDeBordeaux Int
myDoge=DogueDeBordeaux 10
This will not work because 10 cannot be reconciled with
the type variable being bound to String :
badDoge ::DogueDeBordeaux String
badDoge =DogueDeBordeaux 10
Given this, we can see that constructors are how we create
values of types and refer to types in type signatures. There’s a
parallel here between type constructors and data constructors
that should be noted. We can illustrate this with a new canine-
oriented datatype:
dataDoggies a=
Huskya
|Mastiff a
deriving (Eq,Show)
-- type constructor awaiting an argument
Doggies
Note that the kind signature for the type constructor looks
like a function, and the type signature for either of its data
constructors looks similar.
This needs to be applied to become a concrete type:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 604
Prelude> :k Doggies
Doggies :: * -> *
And this needs to be applied to become a concrete value:
Prelude> :t Husky
Husky :: a -> Doggies a
So the behavior of constructors is such that if they don’t take
any arguments, they behave like (type or value-level) constants.
If they do take arguments, they act like (type or value-level)
functions that don’t doanything except get applied.
Exercises: Dog Types
Given the datatypes defined in the above sections,
1.IsDoggies a type constructor or a data constructor?
2.What is the kind of Doggies ?
3.What is the kind of Doggies String ?
4.What is the type of Husky 10 ?
5.What is the type of Husky (10 :: Integer) ?
6.What is the type of Mastiff "Scooby Doo" ?
7.IsDogueDeBordeaux a type constructor or a data constructor?

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 605
8.What is the type of DogueDeBordeaux ?
9.What is the type of DogueDeBordeaux "doggie!"
11.6 What’s a type and what’s data?
As we’ve said, types are static and resolve at compile time.
Types are known before runtime, whether through explicit
declaration or type inference, and that’s what makes them
static types. Information about types does not persist through
to runtime. Data are what we’re working with at runtime.
Here compile time is literally when your program is getting
compiled by GHC or checked before execution in a REPL
like GHCi. Runtime is the actual execution of your program.
Types circumscribe values and in that way, they describe which
values are flowing through what parts of your program.
type constructors -- compile-time
-------------------- phase separation
data constructors -- runtime
Both data constructors and type constructors begin with
capital letters, but a constructor beforethe=in a datatype defini-
tion is a type constructor, while constructors afterthe=are data
constructors. Data constructors are usually generated by the

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 606
declaration. One tricky bit here is that when data constructors
take arguments, those arguments refer to other types . Because
of this, not everything referred to in a datatype declaration is
necessarily generated by that datatype itself. Let’s take a look at
a short example with diﬀerent datatypes to demonstrate what
we mean by this.
We start with a datatype Pricethat has one type construc-
tor, one data constructor, and one type argument in the data
constructor:
dataPrice=
-- (a)
PriceInteger deriving (Eq,Show)
-- (b) [1]
The type constructor is (a). The data constructor is (b), and
that takes one type argument, [1].
The value Pricedoes not depend solely on this datatype
definition. It depends on the type Integer as well. If, for some
reason, Integer wasn’t in scope, we’d be unable to generate
Pricevalues.
Next, we’ll define two datatypes, Manufacturer andAirline ,
that are each sum types with three data constructors. Each data
constructor in these is a possible value of that type, and since
none of them take arguments, all are generated by their decla-

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 607
rations and are more like constant values than constructors:
dataManufacturer =
-- (c)
Mini
-- (d)
|Mazda
-- (e)
|Tata
-- (f)
deriving (Eq,Show)
Manufacturer has the type constructor (c). Manufacturer has
three data constructors (d), (e), and (f).
dataAirline =
-- (g)
PapuAir
-- (h)
|CatapultsR'Us
-- (i)
|TakeYourChancesUnited
-- (j)
deriving (Eq,Show)

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 608
The type constructor is (g). Airline has three data construc-
tors (h), (i), and (j).
Next we’ll look at another sum type, but this one has data
constructors that take arguments. For the type Vehicle , the
data constructors are CarandPlane, so aVehicle is either a Car
value or a Planevalue. They each take types as arguments, just
asPriceitself took the type Integer as an argument:
dataVehicle =CarManufacturer Price
-- (k) (l) [2] [3]
|PlaneAirline
-- (m) [4]
deriving (Eq,Show)
The type constructor is (k). There are two data constructors,
(l) and (m). The type arguments are numbered [2], [3], and
[4]. [2] and [3] are type arguments to the data constructor Car,
while [4] is the type argument to the data constructor Plane. To
construct a Planevalue, therefore, we need a value from the
Airline type.
In the above, the datatypes are generating the constructors
marked with a letter. The type arguments marked with a
number existed prior to the declarations. Their definitions
exist outside of this declaration, and they must be in scope to
be used as part of this declaration.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 609
Each of the above datatypes has a deriving clause. We have
seen this before, as it is usually true that you will want to de-
rive an instance of Showfor datatypes you write. The instance
allows your data to be printed to the screen as a string. De-
rivingEqis also common and allows you to derive equality
operations automatically for most datatypes where that would
make sense. There are other typeclasses that allow derivation
in this manner, and it obviates the need for manually writing
instances for each datatype and typeclass (reminder: you saw
an example of this in the Typeclasses chapter).
As we’ve seen, data constructors can take arguments. Those
arguments will be specific types, but not specific values. In
standard Haskell, we can’t choose specific values of types as
type arguments. We can’t say, for example, “ Boolwithout the
possibility of Falseas a value.” If you accept Boolas a valid type
for a function or as the component of a datatype, you must
accept all of Bool.
Exercises: Vehicles
For these exercises, we’ll use the datatypes defined in the above
section. It would be good if you’d typed them all into a source
file already, but if you hadn’t, please do so now. You can then
define some sample data on your own, or use these to get you
started:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 610
myCar =CarMini(Price14000)
urCar =CarMazda(Price20000)
clownCar =CarTata(Price7000)
doge =PlanePapuAir
1.What is the type of myCar?
2.Given the following, define the functions:
isCar::Vehicle ->Bool
isCar=undefined
isPlane ::Vehicle ->Bool
isPlane =undefined
areCars ::[Vehicle]->[Bool]
areCars =undefined
3.Now we’re going to write a function to tell us the manu-
facturer of a piece of data:
getManu ::Vehicle ->Manufacturer
getManu =undefined
4.Given that we’re returning the Manufacturer , what will hap-
pen if you use this on Planedata?

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 611
5.All right. Let’s say you’ve decided to add the size of the
plane as an argument to the Planeconstructor. Add that
to your datatypes in the appropriate places and change
your data and functions appropriately.
11.7 Data constructor arities
Now that we have a good understanding of the anatomy of
datatypes, we want to start demonstrating why we call them “al-
gebraic.” We’ll start by looking at something called arity. Arity
refers to the number of arguments a function or constructor
takes. A function that takes no arguments is called nullary ,
where nullary is a contraction of “null” and “-ary”. Null means
zero, the “-ary” suffix means “of or pertaining to”. “-ary” is a
common suffix used when talking about mathematical arity,
such as with nullary, unary, binary, and the like.
Data constructors which take no arguments are also called
nullary. Nullary data constructors, such as TrueandFalse, are
constant values at the term level and, since they have no argu-
ments, they can’t construct or represent any data other than
themselves. They are values which stand for themselves and
act as a witness of the datatype they were declared in.
We’ve said that “A type can be thought of as an enumeration
of constructors that have zero or morearguments.” We’ll look
next at constructors with arguments.
We’ve seen how data constructors may take arguments and

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 612
that makes them more like a function in that they must be
applied to something before you have a value. Data construc-
tors that take one argument are called unary. As we will see
later in this chapter, data constructors that take more than one
argument are called products .
All of the following are valid data declarations:
-- nullary
dataExample0 =
Example0
deriving (Eq,Show)
-- unary
dataExample1 =
Example1 Int
deriving (Eq,Show)
-- product of Int and String
dataExample2 =
Example2 IntString
deriving (Eq,Show)
Prelude> Example0
Example0
Prelude> Example1 10
Example1 10

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 613
Prelude> Example1 10 == Example1 42
False
Prelude> let nc = Example2 1 "NC"
Prelude> Example2 10 "FlappityBat" == nc
False
OurExample2 is an example of a product , like tuples, which
we’ve seen before. Tuples can take several arguments — as
many as there are inhabitants of each tuple — and are consid-
ered the canonical product type; they are anonymous products
because they have no name. We’ll talk more about product
types soon.
Unary (one argument) data constructors contain a single
value of whatever type their argument was. The following is a
data declaration that contains the data constructor MyVal.MyVal
takes one Intargument and creates a type named MyType :
dataMyType=MyValInt
-- [1] [2] [3]
deriving (Eq,Show)
-- [4] [5]
1.Type constructor.
2.Data constructor. MyValtakes one type argument, so it is
called a unary data constructor.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 614
3.Type argument to the definition of the data constructor
from [2].
4.Deriving clause.
5.Typeclass instances being derived. We’re getting equality
Eqand value stringification Showfor free.
Prelude> :t MyVal
MyVal :: Int -> MyType
Prelude> MyVal 10
MyVal 10
Prelude> MyVal 10 == MyVal 10
True
Prelude> MyVal 10 == MyVal 9
False
Because MyValhas one Intargument, a value of type MyType
must contain one — only one — Intvalue.
11.8 What makes these datatypes
algebraic?
Algebraic datatypes in Haskell are algebraic because we can
describe the patterns of argument structures using two basic
operations: sum and product. The most direct way to explain
why they’re called sum and product is to demonstrate sum

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 615
and product in terms of cardinality . This can be understood in
terms of the cardinality you see with finite sets.3This doesn’t
map perfectly as we can have infinite data structures in Haskell,
but it’s a good way to begin understanding and appreciating
how datatypes work. When it comes to programming lan-
guages we are concerned with computable functions, not just
those which can generate a set.
The cardinality of a datatype is the number of possible
values it defines. That number can be as small as 0 or as large
as infinite (for example, numeric datatypes, lists). Knowing
how many possible values inhabit a type can help you reason
about your programs. In the following sections we’ll show
you how to calculate the cardinality of a given datatype based
solely on how it is defined. From there, we can determine
how many diﬀerent possible implementations there are of a
function for a given type signature.
Before we get into the specifics of how to calculate cardi-
nality in general, we’re going to take cursory glances at some
datatypes with easy to understand cardinalities: BoolandInt.
We’ve looked extensively at the Booltype already so you
already know it only has two inhabitants that are both nullary
data constructors, so Boolonly has two possible values. The
cardinality of Boolis, therefore, 2. Even without understanding
3Type theory was developed as an alternative mathematical foundation to set theory.
We won’t write formal proofs based on this, but the way we reason informally about types
as programmers derives in part from their origins as sets. Finite sets contain a number of
unique objects; that number is called cardinality.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 616
the rules of cardinality of sum types, we can see why this is
true.
Another set of datatypes with cardinality that is reasonably
easy to understand are the Inttypes. In part this is because Int
and related types Int8,Int16, andInt32have clearly delineated
upper and lower bounds, defined by the amount of memory
they are permitted to use. We’ll use Int8here, even though it
isn’t very common in Haskell, because it has the smallest set
of possible inhabitants and thus the arithmetic is a bit easier
to do. Valid Int8values are whole numbers from (-128) to 127.
Int8is not included in the standard Prelude , unlike standard
Int, so we need to import it to see it in the REPL, but after
we do that we can use maxBound andminBound from the Bounded
typeclass to view the upper and lower values:
Prelude> import Data.Int
Prelude Data.Int> minBound :: Int8
-128
Prelude Data.Int> maxBound :: Int8
127
Given that this range includes the value 0, we can easily
figure out the cardinality of Int8with some quick addition:
128 + 127 + 1 = 256. So the cardinality of Int8is 256. Anywhere
in your code where you’d have a value of type Int8, there are
256 possible runtime values.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 617
Exercises: Cardinality
While we haven’t explicitly described the rules for calculating
the cardinality of datatypes yet, you might already have an idea
of how to do it for simple datatypes with nullary constructors.
Try not to overthink these exercises — follow your intuition
based on what you know.
1.dataPugType =PugData
2.For this one, recall that Boolis also defined with the |:
dataAirline =
PapuAir
|CatapultsR'Us
|TakeYourChancesUnited
3.Given what we know about Int8, what’s the cardinality of
Int16?
4.Use the REPL and maxBound andminBound to examine Int
andInteger . What can you say about the cardinality of
those types?
5.Extra credit (impress your friends!): What’s the connec-
tion between the 8 in Int8and that type’s cardinality of
256?

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 618
Simple datatypes with nullary data constructors
We’llstartourexplorationofcardinalitybylookingatdatatypes
with nullary data constructors:
dataExample =MakeExample deriving Show
Example is our type constructor, and MakeExample is our only
data constructor. Since MakeExample takes no type arguments, it
is a nullary constructor. We know that nullary data construc-
tors are constants and represent only themselves as values. It
is a single value whose only content is its name, not any other
data. Nullary constructors represent onevalue when reasoning
about the cardinality of the types they inhabit.
All you can say about MakeExample is that the constructor is
the value MakeExample and that it inhabits the type Example .
Theretheonlyinhabitantis MakeExample . Giventhat MakeExample
is a single nullary value, so the cardinality of the type Example is
1. This is useful because it tells us that any time we see Example
in the type signature of a function, we only have to reason
about one possible value.
Exercises: For Example
1.You can query the type of a value in GHCi with the :type
command, also abbreviated :t.
Example:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 619
Prelude> :t False
False :: Bool
What is the type of data constructor MakeExample ? What
happens when you request the type of Example ?
2.What if you try :infoonExample in GHCi? Can you deter-
mine what typeclass instances are defined for the Example
type using :infoin GHCi?
3.Try making a new datatype like Example but with a single
type argument added to MakeExample , such as Int. What has
changed when you query MakeExample with:typein GHCi?
Unary constructors
In the last section, we asked you to add a single type argument
to theMakeExample data constructor. In doing so, you changed
it from a nullary constructor to a unary one. A unary data con-
structor takes one argument. In the declaration of the datatype,
that parameter will be a type, not a value. Now, instead of your
data constructor being a constant, or a known value, the value
will be constructed at runtime from the argument we applied
it to.
Datatypes that only contain a unary constructor always have
the same cardinality as the type they contain. In the following,
Goatshas the same number of inhabitants as Int:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 620
dataGoats=GoatsIntderiving (Eq,Show)
Anything that is a valid Int, must also be a valid argument
to theGoatsconstructor. Anything that isn’t a valid Intalso
isn’t a valid count of Goats.
For cardinality, this means unary constructors are the iden-
tity function.
11.9 newtype
We will now look at a way to define a type that can only ever
have a single unary data constructor. We use the newtype key-
word to mark these types, as they are diﬀerent from type
declarations marked with the datakeyword as well as from
type synonym definitions marked by the typekeyword. Like
other datatypes that have a single unary constructor, the car-
dinality of a newtype is the same as that of the type it contains.
Anewtype cannot be a product type, sum type, or contain
nullary constructors, but it has a few advantages over a vanilla
datadeclaration. One is that it has no runtime overhead, as
it reuses the representation of the type it contains. It can do
this because it’s not allowed to be a record (product type) or
tagged union (sum type). The diﬀerence between newtype and
the type it contains is gone by the time the compiler generates
the code.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 621
To illustrate, let’s say we have a function from Int -> Bool
for checking whether we have too many goats:
tooManyGoats ::Int->Bool
tooManyGoats n=n>42
We might run into a problem here if we had diﬀerent limits
for diﬀerent sorts of livestock. What if we mixed up the Int
value of cows where we meant goats? Fortunately, there’s a
way to address this with unary constructors:
newtype Goats=
GoatsIntderiving (Eq,Show)
newtype Cows=
CowsIntderiving (Eq,Show)
Now we can rewrite our type to be safer, pattern matching
in order to access the Intinside our data constructor Goats:
tooManyGoats ::Goats->Bool
tooManyGoats (Goatsn)=n>42
Now we can’t mix up our livestock counts:
Prelude> tooManyGoats (Goats 43)
True
Prelude> tooManyGoats (Cows 43)

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 622
Couldn't match expected type
‘Goats’ with actual type ‘Cows’
In the first argument of
‘tooManyGoats’, namely ‘(Cows 43)’
In the expression: tooManyGoats (Cows 43)
Usingnewtype can deliver other advantages related to type-
class instances. To see these, we need to compare newtypes to
type synonyms and regular data declarations. We’ll start with
a short comparison to type synonyms.
Anewtype is similar to a type synonym in that the represen-
tations of the named type and the type it contains are identical
and any distinction between them is gone at compile time. So,
aString really is a [Char] andGoatsabove is really an Int. On
the surface, for the human writers and readers of code, the
distinction can be helpful in tracking where data came from
and what it’s being used for, but the diﬀerence is irrelevant to
the compiler.
However, one key contrast between a newtype and a type
alias is that you can define typeclass instances for newtype s that
diﬀer from the instances for their underlying type. You can’t
do that for type synonyms. Let’s take a look at how that works.
We’ll first define a typeclass called TooMany and an instance for
Int:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 623
classTooMany awhere
tooMany ::a->Bool
instance TooMany Intwhere
tooMany n =n>42
We can use that instance in the REPL but only if we assign
the type Intto whatever numeric literal we’re passing as an
argument, because numeric literals are polymorphic. That
looks like this:
Prelude> tooMany (42 :: Int)
Take a moment and play around with this — try leaving oﬀ
the type declaration and giving it diﬀerent arguments.
Now, let’s say for your goat counting you wanted a special
instance of TooMany that will have diﬀerent behavior from the
Intinstance. Under the hood, Goatsis stillIntbut the newtype
declaration will allow you to define a custom instance:
newtype Goats=GoatsIntderiving Show
instance TooMany Goatswhere
tooMany ( Goatsn)=n>43
Try loading this and passing diﬀerent arguments to it. Does
it behave diﬀerently than the Intinstance above? Do you still

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 624
need to explicitly assign a type to your numeric literals? What
is the type of tooMany ?
Here we were able to make the Goatsnewtype have an in-
stance of TooMany which had diﬀerent behavior than the type
Intwhich it contains. We can’t do this if it’s a type synonym.
Don’t believe us? Try it.
On the other hand, what about the case where we want to
reuse the typeclass instances of the type our newtype contains?
For common typeclasses built into GHC like Eq,Ord,Enum, and
Showwe get this facility for free, as you’ve seen with the deriving
clauses in most datatypes.
For user-defined typeclasses, we can use a language exten-
sion called GeneralizedNewtypeDeriving . Language extensions,
enabled in GHC by the LANGUAGE pragma,4tell the compiler to
process input in ways beyond what the standard provides for.
In this case, this extension will tell the compiler to allow our
newtype to rely on a typeclass instance for the type it contains.
We can do this because the representations of the newtype and
the type it contains are the same. Still, it is outside of the
compiler’s standard behavior so we must give it the special
instruction to allow us to do this.
First, let’s take the case of what we must do without gener-
alized newtype deriving:
4Apragma is a special instruction to the compiler placed in source code. The LANGUAGE
pragma is perhaps more common in GHC Haskell than the other pragmas, but there are
other pragmas we will see later in the book.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 625
classTooMany awhere
tooMany ::a->Bool
instance TooMany Intwhere
tooMany n =n>42
newtype Goats=
GoatsIntderiving (Eq,Show)
instance TooMany Goatswhere
tooMany ( Goatsn)=tooMany n
TheGoatsinstance will do the same thing as the Intinstance,
but we still have to define it separately.
You can test this yourself to see that they’ll return the same
answers.
Now we’ll add the pragma at the top of our source file:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 626
{-# LANGUAGE GeneralizedNewtypeDeriving #-}
classTooMany awhere
tooMany ::a->Bool
instance TooMany Intwhere
tooMany n =n>42
newtype Goats=
GoatsIntderiving (Eq,Show,TooMany)
Now we don’t have to define an instance of TooMany forGoats
that’s identical to the Intinstance. We can reuse the instance
that we already have.
This is also nice for times when we want every typeclass
instance to be the same except for the one we want to change.
Exercises: Logic Goats
1.Reusing the TooMany typeclass, write an instance of the
typeclass for the type (Int, String) . This will require
adding a language pragma named FlexibleInstances5if
you do not use a newtype — GHC will tell you what to do.
5https://ghc.haskell.org/trac/haskell-prime/wiki/FlexibleInstances

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 627
2.Make another TooMany instance for (Int, Int) . Sum the
values together under the assumption this is a count of
goats from two fields.
3.Makeanother TooMany instance, thistimefor (Num a, TooMany
a) => (a, a) . This can mean whatever you want, such as
summing the two numbers together.
11.10 Sum types
Now that we’ve looked at data constructor arities, we’re ready
to define the algebra of algebraic datatypes. The first that we’ll
look at is the sum type such as Bool:
dataBool=False|True
We’ve mentioned previously that the |represents logical
disjunction—thatis, “or.” Thisisthe suminalgebraicdatatypes.
To know the cardinality of sum types, we addthe cardinalities
of their data constructors. TrueandFalsetake no type argu-
ments and thus are nullary constructors, each with a value of
1.
Now we do some arithmetic. As we said earlier, nullary
constructors are 1, and sum types are +or addition, when we
are talking about cardinality:
dataBool=False|True

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 628
How many values inhabit Bool? There are two data con-
structors, each representing only one possible value. Given
that the |syntax represents (+)or addition:
-- ?? represents the cardinality
True|False= ??
True+False== ??
-- False and True both == 1
1+1== ??
We see that the cardinality of Boolis:
1+1==2
-- List of all possible values for Bool
[True,False]-- length is 2
You can check that in your REPL:
Prelude> length (enumFrom False)
2
From this, we see that when working with a Boolvalue we
must reason about two possible values. Sum types are a way
of expressing alternate possibilities within a single datatype.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 629
Signed 8-bit hardware integers in Haskell are defined using
the aforementioned Int8datatype with a range of values from
-128 to 127. It’s not defined this way, but you could think of
it as a sum type of the numbers in that range, leading to the
cardinality of 256 as we saw.
Exercises: Pity the Bool
1.Given a datatype
dataBigSmall =
BigBool
|SmallBool
deriving (Eq,Show)
What is the cardinality of this datatype? Hint: We already
knowBool’s cardinality. Show your work as demonstrated
earlier.
2.Given a datatype

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 630
-- bring Int8 in scope
importData.Int
dataNumberOrBool =
NumbaInt8
|BoolyBool Bool
deriving (Eq,Show)
-- parentheses due to syntactic
-- collision between (-) minus
-- and the negate function
letmyNumba =Numba(-128)
What is the cardinality of NumberOrBool ? What happens if
you try to create a Numbawith a numeric literal larger than
127? And with a numeric literal smaller than (-128)?
If you choose (-128) for a value precisely, you’ll notice
you get a spurious warning:
Prelude> let n = Numba (-128)
Literal 128 is out of the
Int8 range -128..127
If you are trying to write a large negative
literal, use NegativeLiterals

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 631
Now, since -128 is a perfectly valid Int8value you could
choose to ignore this. What happens is that (-128) desug-
ars into (negate 128) . The compiler sees that you expect
the type Int8, butInt8’s max boundary is 127. So even
though you’re negating 128, it hasn’t done that step yet
andimmediately whines about 128 being larger than 127.
One way to avoid the warning is the following:
Prelude> let n = (-128)
Prelude> let x = Numba n
Or you can use the NegativeLiterals extension as it recom-
mends:
Prelude> :set -XNegativeLiterals
Prelude> let n = Numba (-128)
Note that the negative literals extension doesn’t prevent
the warning if you use negate .
11.11 Product types
What does it mean for a type to be a product? A product type’s
cardinality is the product of the cardinalities of its inhabitants.
Arithmetically, products are the result of multiplication . Where
a sum type was expressing or, a product type expresses and.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 632
For those that have programmed in C-like languages before,
a product is like a struct. For those that haven’t, a product is a
way to carry multiple values around in a single data construc-
tor. Any data constructor with two or more type arguments is
a product.
We said previously that tuples are anonymous products.
The declaration of the tuple type looks like this:
( , ) :: a -> b -> (a, b)
This is a product, like a product type: it gives you a way
to encapsulate two pieces of data, of possibly (though not
necessarily) diﬀerent types, in a single value.
We’ll look next at a somewhat silly sum type:
dataQuantumBool =QuantumTrue
|QuantumFalse
|QuantumBoth
deriving (Eq,Show)
What is the cardinality of this sum type?
For reasons that will become obvious, a cardinality of 2
makes it harder to show the diﬀerence between sum and prod-
uct cardinality, so QuantumBool has a cardinality of 3. Now we’re
going to define a product type that contains two QuantumBool
values:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 633
dataTwoQs=
MkTwoQs QuantumBool QuantumBool
deriving (Eq,Show)
The datatype TwoQshas one data constructor, MkTwoQs , that
takes two arguments, making it a product of the two types that
inhabit it. Each argument is of type QuantumBool , which has a
cardinality of 3.
You can write this out to help you visualize it if you like. A
MkTwoQs value could be:
MkTwoQs QuantumTrue QuantumTrue
MkTwoQs QuantumTrue QuantumFalse
MkTwoQs QuantumTrue QuantumBoth
MkTwoQs QuantumFalse QuantumFalse
-- ...... and so on
Note that there is no special syntax denoting product types
as there was with sums and |.MkTwoQs is a data constructor
taking two type arguments, which both happen to be the same
type. It is a product type, the product of two QuantumBool s. The
number of potential values that can manifest in this type is the
cardinality of one of its type arguments times the cardinality
of the other. So, what is the cardinality of TwoQs?
We could have also written the TwoQstype using a type alias
and the tuple data constructor. Type aliases create type con-
structors, not data constructors:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 634
typeTwoQs=(QuantumBool ,QuantumBool )
The cardinality of this will be the same as it was previously.
The reason it’s important to understand cardinality is that
the cardinality of a datatype roughly equates to how difficult
it is to reason about.
Record syntax
Records in Haskell are product types with additional syntax to
provide convenient accessors to fields within the record. Let’s
begin by definining a simple product type:
dataPerson=
MkPerson StringInt
deriving (Eq,Show)
That is the familiar product type structure: the MkPerson
data constructor takes two type arguments in its definition, a
String value (a name) and an Intvalue (an age). The cardinality
of this is frankly terrifying.
As we’ve seen in previous examples, we can unpack the
contents of this type using functions that return the value we
want from our little box of values:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 635
-- sample data
jm=MkPerson "julie" 108
ca=MkPerson "chris" 16
namae::Person->String
namae(MkPerson s_)=s
If you use the namaefunction in your REPL, it will return
theString value from your data.
Now let’s see how we could define a similar product type
but with record syntax:
dataPerson=
Person{ name::String
, age::Int}
deriving (Eq,Show)
You can see the similarity to the Person type defined above,
but defining it as a record means there are now named record
field accessors. They’re just functions that go from the product
type to a member of product:
Prelude> :t name
name :: Person -> String
Prelude> :t age
age :: Person -> Int

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 636
You can use this directly in GHCi:
Prelude> Person "Papu" 5
Person {name = "Papu", age = 5}
Prelude> let papu = Person "Papu" 5
Prelude> age papu
5
Prelude> name papu
"Papu"
You can also do it from data that is in a file. Change the jm
andcadata above so that it is now of type Person , reload your
source file, and try using the record field accessors in GHCi to
query the values.
11.12 Normal form
We’velookedatthealgebrabehindHaskell’salgebraicdatatypes,
and explored how this is useful for understanding the cardi-
nality of datatypes. But the algebra doesn’t stop there. All the
existing algebraic rules for products and sums apply in type
systems, and that includes the distributive property. Let’s take
a look at how that works in arithmetic:
2 * (3 + 4)
2 * (7)

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 637
14
We can rewrite this with the multiplication distributed over
the addition and obtain the same result:
2 * 3 + 2 * 4
(6) + (8)
14
This is known as a “sum of products.” In normal arithmetic,
the expression is in normal form when it’s been reduced to
a final result. However, if you think of the numerals in the
above expressions as representations of set cardinality, then
the sum of products expression is in normal form, as there is
no computation to perform.
The distributive property can be generalized:
a * (b + c) -> (a * b) + (a * c)
And this is true of Haskell’s types as well! Product types
distribute over sum types. To play with this, we’ll first define
some datatypes:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 638
dataFiction =Fiction deriving Show
dataNonfiction =Nonfiction deriving Show
dataBookType =FictionBook Fiction
|NonfictionBook Nonfiction
deriving Show
We define two types with only single, nullary inhabitants:
Fiction andNonfiction . The reasons for doing that may not
be immediately clear but recall that we said you can’t use a
type while only permitting one of its inhabitants as a possible
value. You can’t ask for a value of type Boolwhile declaring
in your types that it must always be True— you must admit
the possibility of either Boolvalue. So, declaring the Fiction
andNonfiction types will allow us to factor out the book types
(below).
Then we have a sum type, BookType , with constructors that
take the Fiction andNonfiction typesas arguments. It’s impor-
tant to remember that, although the type constructors and data
constructors of Fiction andNonfiction have the same name,
they are not the same, and it is the type constructors that
are the arguments to FictionBook andNonfictionBook . Take a
moment and rename them to demonstrate this to yourself.
So, we have our sum type. Next we’re going to define a type
synonym called AuthorName and a product type called Author.
The type synonym doesn’t really do anything except help us

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 639
keep track of which String we’re using in the Author type:
typeAuthorName =String
dataAuthor=Author(AuthorName ,BookType )
This isn’t a sum of products, so it isn’t normal form. It
can, in some sense, be evaluated to tease apart the values that
are hiding in the sum type, BookType . Again, we can apply the
distributive property and rewrite Author in normal form:
typeAuthorName =String
-- If you have them in the same
-- file, you'll need to comment
-- out previous definitions of
-- Fiction and Nonfiction.
dataAuthor=
Fiction AuthorName
|Nonfiction AuthorName
deriving (Eq,Show)
Products distribute over sums. Just as we would do with
the expression a * (b + c) , where the inhabitants of the sum
typeBookType are the 𝑏and𝑐, we broke those values out and

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 640
made a sum of products. Now it’s in normal form because
no further evaluation can be done of these constructors until
some operation or computation is done using these types.
Another example of normal form can be found in the Expr
type which is very common to papers about type systems and
programming languages:
dataExpr=
NumberInt
|AddExprExpr
|MinusExpr
|MultExprExpr
|DivideExprExpr
This is in normal form because it’s a sum (type) of products:
(Number Int) + Add (Expr Expr) + …
A stricter interpretation of normal form or “sum of prod-
ucts” would require representing products with tuples and
sums with Either. The previous datatype in that form would
look like the following:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 641
typeNumber=Int
typeAdd=(Expr,Expr)
typeMinus=Expr
typeMult=(Expr,Expr)
typeDivide=(Expr,Expr)
typeExpr=
EitherNumber
(EitherAdd
(EitherMinus
(EitherMultDivide)))
This representation finds applications in problems where
one is writing functions or foldsover the representations of
datatypes, such as with generics and metaprogramming. Some
of these methods have their application in Haskell but should
be used judiciously and aren’t always easy to use.
TheEither type will be explained in detail in the next chap-
ter.
Exercises: How Does Your Garden Grow?
1. Given the type

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 642
dataFlowerType =Gardenia
|Daisy
|Rose
|Lilac
deriving Show
typeGardener =String
dataGarden=
GardenGardener FlowerType
deriving Show
What is the sum of products normal form of Garden ?
11.13 Constructing and deconstructing
values
There are essentially two things we can do with a value: we can
generate or construct it or we can match on it and consume
it. We talked above about why data and type constructors are
calledconstructors , and this section will elaborate on that and
how to construct values of diﬀerent types. You have already
been doing this in previous chapters, but we hope this section
will lead you to a deeper understanding.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 643
Construction and deconstruction of values form a duality.
Data is immutable in Haskell, so values carry with them the
information about how they were created. We can use that
information when we consume or deconstruct the value.
We’ll start by defining a collection of datatypes:
dataGuessWhat =
Chickenbutt deriving (Eq,Show)
dataIda=
MkIdaderiving (Eq,Show)
dataProduct a b=
Product a bderiving (Eq,Show)
dataSuma b=
Firsta
|Secondb
deriving (Eq,Show)
dataRecordProduct a b=
RecordProduct { pfirst ::a
, psecond ::b }
deriving (Eq,Show)
Now that we have diﬀerent sorts of datatypes to work with,

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 644
we’ll move on to constructing values of those types.
Sum and Product
HereSumandProduct are ways to represent arbitrary sums and
products in types. In ordinary Haskell code, it’s unlikely you’d
need or want nestable sums and products unless you were
doing something fairly advanced, but here we use them as a
means of demonstration.
If you have two values in a product, then the conversion
to using Product is straightforward (n.b.: The SumandProduct
declarations from above will need to be in scope for all the
following examples):

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 645
newtype NumCow=
NumCowInt
deriving (Eq,Show)
newtype NumPig=
NumPigInt
deriving (Eq,Show)
dataFarmhouse =
Farmhouse NumCowNumPig
deriving (Eq,Show)
typeFarmhouse' =Product NumCowNumPig
Farmhouse andFarmhouse' are the same.
For an example with three values in the product instead of
two, we must begin to take advantage of the fact that Product
takes two arguments, one of which can also be another Product
of values. In fact, you can nest them as far as you can stomach
or until the compiler chokes:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 646
newtype NumSheep =
NumSheep Int
deriving (Eq,Show)
dataBigFarmhouse =
BigFarmhouse NumCowNumPigNumSheep
deriving (Eq,Show)
typeBigFarmhouse' =
Product NumCow(Product NumPigNumSheep )
We can perform a similar trick with Sum:
typeName=String
typeAge=Int
typeLovesMud =Bool
Sheep can produce between 2 and 30 pounds (0.9 and 13
kilos) of wool per year! Icelandic sheep don’t produce as much
wool per year as other breeds but the wool they do produce is
a finer wool.
typePoundsOfWool =Int
dataCowInfo =
CowInfo NameAge
deriving (Eq,Show)

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 647
dataPigInfo =
PigInfo NameAgeLovesMud
deriving (Eq,Show)
dataSheepInfo =
SheepInfo NameAgePoundsOfWool
deriving (Eq,Show)
dataAnimal=
CowCowInfo
|PigPigInfo
|SheepSheepInfo
deriving (Eq,Show)
-- Alternately
typeAnimal' =
SumCowInfo (SumPigInfo SheepInfo )
Again in the REPL, we use FirstandSecond to pattern match
on the data constructors of Sum:
-- Getting it right
Prelude> let bess' = (CowInfo "Bess" 4)
Prelude> let bess = First bess' :: Animal'

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 648
Prelude> :{
*Main| let e' =
*Main| Second (SheepInfo "Elmer" 5 5)
*Main| :}
Prelude> let elmer = Second e' :: Animal'
-- Making a mistake
Prelude> :{
*Main| let elmo' =
*Main| Second (SheepInfo "Elmo" 5 5)
*Main| :}
Prelude> let elmo = First elmo' :: Animal'
Couldn't match expected type ‘CowInfo’
with actual type ‘Sum a0 SheepInfo’
In the first argument of ‘First’, namely
‘(Second (SheepInfo "Elmo" 5 5))’
In the expression:
First (Second (SheepInfo "Elmo" 5 5))
:: Animal'
The first data constructor, First, has the argument CowInfo ,
butSheepInfo is nested within the Second constructor (it is the
Second of the Second). We can see how they don’t match and
the mistaken attempt nests in the wrong direction.
Prelude> let sheep = SheepInfo "Baaaaa" 5 5

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 649
Prelude> :t First (Second sheep)
First (Second (SheepInfo "Baaaaa" 5 5))
:: Sum (Sum a SheepInfo) b
Prelude> :info Animal'
type Animal' =
Sum CowInfo (Sum PigInfo SheepInfo)
-- Defined at code/animalFarm1.hs:61:1
As we said, the actual types SumandProduct themselves aren’t
used very often in standard Haskell code, but it can be useful to
develop an intuition about this structure to sum and product
types.
Constructing values
Our first datatype, GuessWhat , is trivial, equivalent to the ()unit
type:
trivialValue ::GuessWhat
trivialValue =Chickenbutt
Types like this are sometimes used to signal discrete con-
cepts that you don’t want to flatten into the unit type. We’ll
elaborate on how this can make code easier to understand or
better abstracted later. There is nothing special in the syntax

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 650
here. We define trivialValue to be the nullary data constructor
Chickenbutt and we have a value of the type GuessWhat .
Next we look at a unary type constructor that contains one
unary data constructor:
dataIda=
MkIdaderiving (Eq,Show)
Because Idhas an argument, we have to apply it to some-
thing before we can construct a value of that type:
-- note:
-- MkId :: a -> Id a
idInt::IdInteger
idInt=MkId10
We turn our attention to our product type with two argu-
ments. We’re going to define some type synonyms first to
make this more readable:
typeAwesome =Bool
typeName=String
person::Product NameAwesome
person=Product "Simon" True

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 651
The type synonyms Awesome andNamehere are for clarity.
They don’t obligate us to change our terms. We could have
used datatypes instead of type synonyms, as we will in the sum
type example below, but this is a quick and painless way to
construct the value that we need. Notice that we’re relying on
theProduct data constructor that we defined above. The Product
data constructor is a function of two arguments, the Nameand
Awesome . Notice, also, that Simons are invariably awesome.
Now we’ll use the Sumtype defined above:
dataSuma b=
Firsta
|Secondb
deriving (Eq,Show)
dataTwitter =
Twitter deriving (Eq,Show)
dataAskFm=
AskFmderiving (Eq,Show)
socialNetwork ::SumTwitter AskFm
socialNetwork =FirstTwitter
Here our type is a sum of Twitter orAskFm. We don’t have
both values at the same time without the use of a product

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 652
because sums are a means of expressing disjunction or the
ability to have one of several possible values. We have to use
one of the data constructors generated by the definition of Sum
in order to indicate which of the possibilities in the disjunction
we mean to express. Consider the case where we mix them
up:
Prelude> type SN = Sum Twitter AskFm
Prelude> Second Twitter :: SN
Couldn't match expected type ‘AskFm’ with
actual type ‘Twitter’
In the first argument of ‘Second’,
namely ‘Twitter’
In the expression:
Second Twitter :: Sum Twitter AskFm
Prelude> First AskFm :: Sum Twitter AskFm
Couldn't match expected type ‘Twitter’ with
actual type ‘AskFm’
In the first argument of ‘First’,
namely ‘AskFm’
In the expression:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 653
First AskFm :: Sum Twitter AskFm
The appropriate assignment of types to specific construc-
tors is dependent on the assertions in the type. The type signa-
tureSum Twitter AskFm tells you which goes with the data con-
structor Firstand which goes with the data constructor Second .
We can assert that ordering directly by writing a datatype like
this:
dataSocialNetwork =
Twitter
|AskFm
deriving (Eq,Show)
Now the data constructors for Twitter andAskFmare direct
inhabitants of the sum type SocialNetwork , where before they
inhabited the Sumtype. Now let’s consider how this might look
with type synonyms:
typeTwitter =String
typeAskFm=String
twitter ::SumTwitter AskFm
twitter =First"Twitter"
askfm::SumTwitter AskFm
askfm=First"AskFm"

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 654
There’s a problem with the above example. The name
ofaskfmimplies we meant Second "AskFm" , but we messed up.
Because we used type synonyms instead of defining datatypes,
the type system didn’t catch the mistake. The typechecker has
no way of knowing we made a mistake because both values are
Strings . Try to avoid using type synonyms with unstructured
data like text or binary. Type synonyms are best used when
you want something lighter weight than newtypes but also
want your type signatures to be more explicit.
Finally, we’ll consider the product that uses record syntax:
Prelude> :t RecordProduct
RecordProduct :: a
-> b
-> RecordProduct a b
Prelude> :t Product
Product :: a -> b -> Product a b
The first thing to notice is that you can construct values of
products that use record syntax in a manner identical to that
of non-record products. Records are just syntax to create field
references. They don’t do much heavy lifting in Haskell, but
they are convenient:
myRecord ::RecordProduct Integer Float
myRecord =RecordProduct 420.00001

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 655
We can take advantage of the fields that we defined on our
record to construct values in a slightly diﬀerent style. This can
be convenient for making things a little more obvious:
myRecord ::RecordProduct Integer Float
myRecord =
RecordProduct { pfirst =42
, psecond =0.00001 }
This is a bit more compelling when you have domain-
specific names for things:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 656
dataOperatingSystem =
GnuPlusLinux
|OpenBSDPlusNevermindJustBSDStill
|Mac
|Windows
deriving (Eq,Show)
dataProgLang =
Haskell
|Agda
|Idris
|PureScript
deriving (Eq,Show)
dataProgrammer =
Programmer { os::OperatingSystem
, lang::ProgLang }
deriving (Eq,Show)
Then we can construct a value from the record product
Programmer :
Prelude> :t Programmer
Programmer :: OperatingSystem
-> ProgLang
-> Programmer

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 657
nineToFive ::Programmer
nineToFive =Programmer { os=Mac
, lang=Haskell }
-- We can reorder stuff
-- when we use record syntax
feelingWizardly ::Programmer
feelingWizardly =
Programmer { lang=Agda
, os=GnuPlusLinux }
Exercise: Programmers
Write a function that generates all possible values of Programmer .
Use the provided lists of inhabitants of OperatingSystem and
ProgLang .

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 658
allOperatingSystems ::[OperatingSystem ]
allOperatingSystems =
[GnuPlusLinux
,OpenBSDPlusNevermindJustBSDStill
,Mac
,Windows
]
allLanguages ::[ProgLang ]
allLanguages =
[Haskell,Agda,Idris,PureScript ]
allProgrammers ::[Programmer ]
allProgrammers =undefined
Programmer is a product of two types, you can determine how
many inhabitants of Programmer you have by calculating:
length allOperatingSystems
*length allLanguages
This is the essence of how product types and the number
of inhabitants relate.
There are several ways you could write a function to do
that, and some may produce a list that has duplicate values
in it. If your resulting list has duplicate values in it, you can

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 659
usenubfromData.List to remove duplicate values over your
allProgrammers value. Either way, if your result (minus any
duplicate values) equals the number returned by multiplying
those lengths together, you’ve probably got it figured out. Try
to be clever and make it work without manually typing out
the values.
Accidental bottoms from records
We’re going to reuse the previous Programmer datatype to see
what happens if we construct a value using record syntax but
forget a field:
Prelude> :{
*Main| let partialAf =
*Main| Programmer {os = GnuPlusLinux}
*Main| :}
Fields of ‘Programmer’
not initialised: lang
In the expression:
Programmer {os = GnuPlusLinux}
In an equation for ‘partialAf’:
partialAf =
Programmer {os = GnuPlusLinux}
-- and if we don't heed this warning...

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 660
Prelude> partialAf
Programmer {os = GnuPlusLinux, lang =
*** Exception:
Missing field in
record construction lang
Donotdo this in your code! Either define the whole record
at once or not at all. If you think you need this, your code needs
to be refactored. Partial application of the data constructor
suffices to handle this:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 661
-- Works the same as if
-- we'd used record syntax.
dataThereYet =
ThereFloatIntBool
deriving (Eq,Show)
-- who needs a "builder pattern"?
notYet::Int->Bool->ThereYet
notYet=nope25.5
notQuite ::Bool->ThereYet
notQuite =notYet10
yusssss ::ThereYet
yusssss =notQuite False
-- Not I, said the Haskell user.
Notice the way our types progressed.
There::Float->Int->Bool->ThereYet
notYet:: Int->Bool->ThereYet
notQuite :: Bool->ThereYet
yusssss :: ThereYet

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 662
Percolate values through your programs, not bottoms.6
Deconstructing values
When we discussed folds, we mentioned the idea of catamor-
phism. We explained that catamorphism was about deconstruct-
inglists. This idea is generally applicable to any datatype that
has values. Now that we’ve thoroughly explored constructing
values, the time has come to destroy what we have built. Wait,
no — we mean deconstruct.
We begin, as always, with some datatypes:
6A favorite snack of the North American Yeti is bottom-propagating Haskellers.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 663
newtype Name =NameStringderiving Show
newtype Acres =AcresIntderiving Show
-- FarmerType is a Sum
dataFarmerType =DairyFarmer
|WheatFarmer
|SoybeanFarmer
deriving Show
-- Farmer is a plain ole product of
-- Name, Acres, and FarmerType
dataFarmer=
FarmerNameAcresFarmerType
deriving Show
Now we’re going to write a very basic function that breaks
down and unpacks the data inside our constructors:
isDairyFarmer ::Farmer->Bool
isDairyFarmer (Farmer_ _DairyFarmer )=
True
isDairyFarmer _ =
False
DairyFarmer is one value of the FarmerType type that is packed
up inside our Farmer product type. But our function can pull

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 664
that value out, pattern match on it, and tell us just what we’re
looking for.
Now an alternate formulation with a product that uses
record syntax:
dataFarmerRec =
FarmerRec { name ::Name
, acres ::Acres
, farmerType ::FarmerType }
deriving Show
isDairyFarmerRec ::FarmerRec ->Bool
isDairyFarmerRec farmer=
casefarmerType farmer of
DairyFarmer ->True
_ -> False
This is just another way of unpacking or deconstructing the
contents of a product type.
Accidental bottoms from records
We take bottoms very seriously. You can easily propagate bottoms
through record types, and we implore you not to do so. Please,
do not do this:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 665
dataAutomobile =Null
|Car{ make::String
, model ::String
, year::Integer }
deriving (Eq,Show)
This is a terrible thing to do, for a couple of reasons. One
is thisNullnonsense. Haskell oﬀers you the perfectly lovely
datatype Maybe, which you should use instead. Secondly, con-
sider the case where one has a Nullvalue, but you’ve used one
of the record accessors:
Prelude> make Null
"*** Exception: No match in
record selector make
-- Don't.
How do we fix this? Well, first, whenever we have a product
that uses record accessors, keep it separate of any sum type
that is wrapping it. To do this, split out the product into an
independent type with its own type constructor instead of
only as an inline data constructor product:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 666
-- Split out the record/product
dataCar=Car{ make::String
, model ::String
, year::Integer }
deriving (Eq,Show)
-- The Null is still not great, but
-- we're leaving it in to make a point
dataAutomobile =Null
|Automobile Car
deriving (Eq,Show)
Now if we attempt to do something silly, the type system
catches us:
Prelude> make Null
Couldn't match expected type ‘Car’
with actual type ‘Automobile’
In the first argument of ‘make’,
namely ‘Null’
In the expression: make Null
In Haskell, we want the typechecker to catch us doing things
wrong, so we can fix it before problems multiply and things go

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 667
wrong at runtime. But the typechecker can best help those
who help themselves.
11.14 Function type is exponential
In the arithmetic of calculating inhabitants of types, function
type is the exponent operator. Given a function a -> b , we can
calculate the inhabitants with the formula 𝑏u?.
So if𝑏and𝑎areBool, then22is how you could express the
number of inhabitants in a function of Bool -> Bool . Similarly,
a function of Boolto something of 3 inhabitants would be 32
and thus have nine possible implementations.
a->b->c
(c^b)^a
-- given arithmetic laws,
-- can be rewritten as
c^(b*a)
Earlier we identified the type (Bool, Bool) as having four
inhabitants. This can be determined by either writing out all
the possible unique inhabitants or, more easily, by doing the
arithmetic of (1 + 1) * (1 + 1) . Next we’ll see that the type of
functions (->)is, in the algebra of types, the exponentiation

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 668
operator. We’ll use a datatype with three cases because Bool
has one difficulty: two plus two, two times two, and two to
the power of two all equal the same thing. Let’s review the
arithmetic of sum types:
dataQuantum =
Yes
|No
|Both
deriving (Eq,Show)
-- 3 + 3
quantSum1 ::EitherQuantum Quantum
quantSum1 =RightYes
quantSum2 ::EitherQuantum Quantum
quantSum2 =RightNo
quantSum3 ::EitherQuantum Quantum
quantSum3 =RightBoth
quantSum4 ::EitherQuantum Quantum
quantSum4 =LeftYes
-- You can fill in the next two.
And now the arithmetic of product types:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 669
-- 3 * 3
quantProd1 ::(Quantum,Quantum)
quantProd1 =(Yes,Yes)
quantProd2 ::(Quantum,Quantum)
quantProd2 =(Yes,No)
quantProd3 ::(Quantum,Quantum)
quantProd3 =(Yes,Both)
quantProd4 ::(Quantum,Quantum)
quantProd4 =(No,Yes)
quantProd5 ::(Quantum,Quantum)
quantProd5 =(No,No)
quantProd6 ::(Quantum,Quantum)
quantProd6 =(No,Both)
quantProd7 ::(Quantum,Quantum)
quantProd7 =(Both,Yes)
-- You can determine the final two.
And now a function type. Each possible unique implemen-
tation of the function is an inhabitant:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 670
-- 3 ^ 3
quantFlip1 ::Quantum ->Quantum
quantFlip1 Yes=Yes
quantFlip1 No=Yes
quantFlip1 Both=Yes
quantFlip2 ::Quantum ->Quantum
quantFlip2 Yes=Yes
quantFlip2 No=Yes
quantFlip2 Both=No
quantFlip3 ::Quantum ->Quantum
quantFlip3 Yes=Yes
quantFlip3 No=Yes
quantFlip3 Both=Both

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 671
quantFlip4 ::Quantum ->Quantum
quantFlip4 Yes=Yes
quantFlip4 No=No
quantFlip4 Both=Yes
quantFlip5 ::Quantum ->Quantum
quantFlip5 Yes=Yes
quantFlip5 No=Both
quantFlip5 Both=Yes
quantFlip6 ::Quantum ->Quantum
quantFlip6 Yes=No
quantFlip6 No=Yes
quantFlip6 Both=Yes

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 672
quantFlip7 ::Quantum ->Quantum
quantFlip7 Yes=Both
quantFlip7 No=Yes
quantFlip7 Both=Yes
quantFlip8 ::Quantum ->Quantum
quantFlip8 Yes=Both
quantFlip8 No=Yes
quantFlip8 Both=No
quantFlip9 ::Quantum ->Quantum
quantFlip9 Yes=Both
quantFlip9 No=No
quantFlip9 Both=No
quantFlip10 ::Quantum ->Quantum
quantFlip10 Yes=Both
quantFlip10 No=No
quantFlip10 Both=Both
-- You can figure out the remaining
-- possibilities yourself.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 673
Exponentiation in what order?
Consider the following function:
convert ::Quantum ->Bool
convert =undefined
According to the equality of a -> b and𝑏u?there should be 23
or 8 implementations of this function. Does this hold? Write
it out and prove it for yourself.
Exercises: The Quad
Determine how many unique inhabitants each type has.
Suggestion: do the arithmetic unless you want to verify.
Writing them out gets tedious quickly.
1.dataQuad=
One
|Two
|Three
|Four
deriving (Eq,Show)
-- how many different forms can this take?
eQuad::EitherQuadQuad
eQuad= ???

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 674
2.prodQuad ::(Quad,Quad)
3.funcQuad ::Quad->Quad
4.prodTBool ::(Bool,Bool,Bool)
5.gTwo::Bool->Bool->Bool
6.Hint: 5 digit number
fTwo::Bool->Quad->Quad
11.15 Higher-kinded datatypes
You may recall we discussed kinds earlier in this chapter. Kinds
are the types of type constructors, primarily encoding the
number of arguments they take. The default kind in Haskell is
*. Kind signatures work like type signatures, using the same ::
and->syntax, but there are only a few kinds and you’ll most
often see *.
Kinds are not types until they are fully applied. Only types
have inhabitants at the term level. The kind * -> * is waiting
for a single *before it is fully applied. The kind * -> * -> *
must be applied twice before it will be a real type. This is
known as a higher-kinded type . Lists, for example, are higher-
kinded datatypes in Haskell.
Because types can be generically polymorphic by taking
type arguments, they can be applied at the type level:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 675
-- identical to (a, b, c, d)
dataSillya b c d =
MkSilly a b c d deriving Show
-- in GHCi
Prelude>:kindSilly
Silly:: * -> * -> * -> * -> *
Prelude>:kindSillyInt
SillyInt:: * -> * -> * -> *
Prelude>:kindSillyIntString
SillyIntString:: * -> * -> *
Prelude>:kindSillyIntStringBool
SillyIntStringBool:: * -> *
Prelude>:kindSillyIntStringBoolString
SillyIntStringBoolString:: *
-- Identical to (a, b, c, d)
Prelude>:kind (,,,)
(,,,):: * -> * -> * -> * -> *
Prelude>:kind (Int,String,Bool,String)
(Int,String,Bool,String):: *

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 676
Getting comfortable with higher-kinded types is important
as type arguments provide a generic way to express a “hole”
to be filled by consumers of your datatype later. Take the
following as an example from a library one of the authors
maintains called Bloodhound.7
dataEsResultFound a=
EsResultFound { _version ::DocVersion
, _source ::a
}deriving (Eq,Show)
We know that this particular kind of response from Elastic-
search will include a DocVersion value, so that’s been assigned a
type. On the other hand, _source has type 𝑎because we have
no idea what the structure of the documents they’re pulling
from Elasticsearch look like. In practice, we do need to be able
to dosomething with that value of type 𝑎. The thing we will
want to do with it — the way we will consume or use that data
— will usually be a FromJSON typeclass instance for deserializing
JSON data into a Haskell datatype. But in Haskell, we do not
conventionally put constraints on datatypes. That is, we don’t
want to constrain that polymorphic 𝑎in the datatype. The
FromJSON typeclass will likely (assuming that’s what is needed in
7http://hackage.haskell.org/package/bloodhound If you are not a programmer and do
not know what Elasticsearch and JSON are, try not to worry too much about the specifics.
Elasticsearch is a search engine and JSON is a format for transmitting data, especially
between servers and web applications.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 677
a given context) constrain the variable in the type signature(s)
for the function(s) that will process this data.
Accordingly, the FromJSON typeclassinstancefor EsResultFound
requires a FromJSON instance for that 𝑎:
instance (FromJSON a)=>
FromJSON (EsResultFound a)where
parseJSON ( Objectv)=
EsResultFound
<$>v.:"_version"
<*>v.:"_source"
parseJSON _ =empty
As you can hopefully see from this, by not fully applying
the type — by leaving it higher-kinded — space is left for the
type of the response to vary, for the “hole” to be filled in by
the end user.
11.16 Lists are polymorphic
What makes a list polymorphic? In what way can it take many
forms? What makes them polymorphic is that lists in Haskell
can contain values of any type. You do not have an 𝑎until the
list type’s type argument has been fully applied:
data[]a=[]|a:[a]
-- [1] [2] [3] [4] [5] [6]

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 678
1.Type constructor for list has special []syntax.
2.Single type argument to []. This is the type of value our
list contains.
3.Nil / empty list value constructor, again with the special
[] syntax. [] marks the end of the list.
4.A single value of type 𝑎.
5.:is an infix data constructor. It is a product of 𝑎[4] and
[a][6]
6.The rest of our list.
Infix type and data constructors When we give an operator
a nonalphanumeric name, it is infix by default. For example,
all the nonalphanumeric arithmetic functions are infix opera-
tors, while we have some alphanumeric arithmetic functions,
such as divandmodthat are prefix by default. So far, we’ve only
seen alphanumeric data constructors, except for this cons con-
structor in the list type, but the same rule applies to them.
Any operator that starts with a colon ( :) must be an in-
fix type or data constructor. All infix data constructors must
start with a colon. The type constructor of functions, (->), is
the only infix type constructor that doesn’t start with a colon.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 679
Another exception is that they cannot be ::as this syntax is
reserved for type assertions.
In the following example, we’ll define the list type without
using an infix constructor:
-- Same type, redefined
-- with different syntax
dataLista=Nil|Consa (Lista)
-- [1] [2] [3] [5] [4] [6]
1.TheListtype constructor.
2.The𝑎type parameter to List.
3.Nil / empty list value, which also marks the end of a list.
4.A single value of type 𝑎in theConsproduct.
5.TheConsconstructor, product of 𝑎andList a .
6.The rest of our list.
How do we use our Listtype?
Prelude> let nil = Nil
Prelude> :t nil
nil :: List a

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 680
Thetypeparameterisn’tappliedbecause Nilbyitselfdoesn’t
tell the type inference what the Listcontains. But if we give it
some information, then the 𝑎can be assigned a concrete type:
Prelude> let oneItem = (Cons "woohoo!" Nil)
Prelude> :t oneItem
oneItem :: List [Char]
And how are our list types kinded?
Prelude> :kind List
List :: * -> *
Prelude> :kind []
[] :: * -> *
Prelude> :kind List Int
List Int :: *
Prelude> :kind [Int]
[Int] :: *
Much as we can refer to the function notbefore we’ve ap-
plied its argument, we can refer to the list type constructor, [],
before we’ve applied it to a type argument:
Prelude> :t not
not :: Bool -> Bool

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 681
Prelude> :t not True
not True :: Bool
Prelude> :k []
[] :: * -> *
Prelude> :k [Int]
[Int] :: *
The diﬀerence is that the argument of notis any value of
typeBool, and the argument of []is any type of kind *. So,
they’re similar, but type constructors are functions one level
up, structuring things that cannot exist at runtime — it’s purely
static and describes the structure of your types.
11.17 Binary Tree
Now we turn our attention to a type similar to list. The type
constructor for binary trees can take an argument, and it is
also recursive like lists:
dataBinaryTree a=
Leaf
|Node(BinaryTree a) a (BinaryTree a)
deriving (Eq,Ord,Show)
This tree has a value of type 𝑎at each node. Each node
could be a terminal node, called a leaf, or it could branch and

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 682
have two subtrees. The subtrees are also of type BinaryTree a ,
so this type is recursive. Each binary tree can store yet another
binary tree, which allows for trees of arbitrary depth.
In some cases, binary trees can be more efficient for struc-
turing and accessing data than a list, especially if you know
how to order your values in a way that lets you know whether
to look “left” or “right” to find what you want. On the other
hand, a tree that only branches to the right is indistinguishable
from an ordinary list. For now, we won’t concern ourselves
too much with this as we’ll talk about the proper application
of data structures later. Instead, you’re going to write some
functions for processing BinaryTree values.
Inserting into trees
The first thing to be aware of is that we need Ordin order to have
enough information about our values to know how to arrange
them in our tree. Accordingly, if something is lower, we want
to insert it somewhere on the left-hand part of our tree. If it’s
greater than the current node value, it should go somewhere
to the right. Left lesser, right greater is a common convention
for arranging binary trees — it could be the opposite and not
really change anything, but this matches our usual intuitions
of ordering as we do with, say, number lines. The point is you
want to be able to know where to look in the tree for values
greater or less than the current one you’re looking at.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 683
Ourinsert function will insert a value into a tree or, if no
tree exists yet, give us a means of building a tree by inserting
values. It’s important to remember that data is immutable in
Haskell. We do not insert a value into an existing tree; each
time we want to insert a value into the data structure, we build
a whole new tree:
insert' ::Orda
=>a
->BinaryTree a
->BinaryTree a
insert' bLeaf=NodeLeafbLeaf
insert' b (Nodeleft a right)
|b==a=Nodeleft a right
|b<a=Node(insert' b left) a right
|b>a=Nodeleft a (insert' b right)
The base case in our insert' function serves a couple pur-
poses. It handles inserting into an empty tree ( Leaf) and begin-
ning the construction of a new tree and also the case of having
reached the bottom of a much larger tree. The simplicity here
lets us ignore any inessential diﬀerences between those two
cases.
-- Leaf being the "empty tree" case

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 684
Prelude> let t1 = insert' 0 Leaf
Prelude> t1
Node Leaf 0 Leaf
Prelude> let t2 = insert' 3 t1
Prelude> t2
Node Leaf 0 (Node Leaf 3 Leaf)
Prelude> let t3 = insert' 5 t2
Prelude> t3
Node Leaf 0
(Node Leaf 3
(Node Leaf 5 Leaf))
We will examine binary trees and their properties later in
the book. For now, we want to focus not on the properties
of binary trees themselves, but on the structure of their type.
You might find the following exercises tricky or tedious, but
they will deepen your intuition for how recursive types work.
Write map for BinaryTree
Given the definition of BinaryTree above, write a map function
for the data structure. You don’t really need to know anything
about binary trees to write these functions. The structure
inherent in the definition of the type is all you need. All you
need to do is write the recursive functions.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 685
No special algorithms are needed, and we don’t expect you
to keep the tree balanced or ordered. Also, remember that
we’ve never once mutated anything. We’ve only built new
values from input data. Given that, when you go to implement
mapTree , you’re not changing an existing tree — you’re building
a new one based on an existing one (as when you are mapping
functions over lists).
Note, you do notneed to use insert' for this. Retain the
original structure of the tree.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 686
mapTree ::(a->b)
->BinaryTree a
->BinaryTree b
mapTree _Leaf=Leaf
mapTree f (Nodeleft a right) =
Nodeundefined undefined undefined
testTree' ::BinaryTree Integer
testTree' =
Node(NodeLeaf3Leaf)
1
(NodeLeaf4Leaf)
mapExpected =
Node(NodeLeaf4Leaf)
2
(NodeLeaf5Leaf)
-- acceptance test for mapTree
mapOkay =
ifmapTree ( +1) testTree' ==mapExpected
thenprint"yup okay!"
elseerror"test failed!"
Some hints for implementing mapTree follow.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 687
The first pattern match in our mapTree function is the base
case, where we have a Leafvalue. We can’t apply the 𝑓there
because we don’t have an 𝑎, so we ignored it. Since we have
to return a value of type BinaryTree b whatever happens, we
return a Leafvalue.
We return a Nodein the second pattern match of our mapTree
function. Note that the Nodedata constructor takes three argu-
ments:
Prelude> :t Node
Node :: BinaryTree a
-> a
-> BinaryTree a
-> BinaryTree a
So you need to pass it more BinaryTree , a single value, and
moreBinaryTree . You have the following terms available to
you:
1.f::(a->b)
2.left::BinaryTree a
3.a::a
4.right::BinaryTree a

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 688
5.mapTree ::(a->b)
->BinaryTree a
->BinaryTree b
Now the Nodereturn needs to have a value of type 𝑏and
BinaryTree values with type 𝑏inside them. You have two func-
tions at your disposal. One gets you (a -> b) , the other maps
BinaryTree s of type 𝑎intoBinaryTree s of type 𝑏. Get ’em tiger.
A few suggestions that might help you with this exercise.
1.Split out the patterns your function should match on first.
2.Implement the base case first.
3.Try manually writing out the steps of recursion at first,
then collapse them into a single step that is recursive.
Convert binary trees to lists
Write functions to convert BinaryTree values to lists. Make
certain your implementation passes the tests.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 689
preorder ::BinaryTree a->[a]
preorder =undefined
inorder ::BinaryTree a->[a]
inorder =undefined
postorder ::BinaryTree a->[a]
postorder =undefined
testTree ::BinaryTree Integer
testTree =
Node(NodeLeaf1Leaf)
2
(NodeLeaf3Leaf)
testPreorder ::IO()
testPreorder =
ifpreorder testTree ==[2,1,3]
thenputStrLn "Preorder fine!"
elseputStrLn "Bad news bears."
testInorder ::IO()
testInorder =
ifinorder testTree ==[1,2,3]
thenputStrLn "Inorder fine!"
elseputStrLn "Bad news bears."
testPostorder ::IO()
testPostorder =
ifpostorder testTree ==[1,3,2]
thenputStrLn "Postorder fine!"
elseputStrLn "postorder failed check"
main::IO()
main= do
testPreorder
testInorder
testPostorder

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 690
Write foldr for BinaryTree
Given the definition of BinaryTree we have provided, write a
catamorphism for the binary trees.
-- any traversal order is fine
foldTree ::(a->b->b)
->b
->BinaryTree a
->b
11.18 Chapter Exercises
Multiple choice
1.Given the following datatype:
dataWeekday =
Monday
|Tuesday
|Wednesday
|Thursday
|Friday
we can say:
a)Weekday is a type with five data constructors

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 691
b)Weekday is a tree with five branches
c)Weekday is a product type
d)Weekday takes five arguments
2.and with the same datatype definition in mind, what is
the type of the following function, f?
fFriday="Miller Time"
a)f :: [Char]
b)f :: String -> String
c)f :: Weekday -> String
d)f :: Day -> Beer
3.Types defined with the datakeyword
a)must have at least one argument
b)must begin with a capital letter
c)must be polymorphic
d)cannot be imported from modules
4.The function g xs = xs !! (length xs - 1)
a)is recursive and may not terminate
b)delivers the head of xs
c)delivers the final element of xs
d)has the same type as xs

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 692
Ciphers
In the Lists chapter, you wrote a Caesar cipher. Now, we want
to expand on that idea by writing a Vigenère cipher. A Vi-
genère cipher is another substitution cipher, based on a Caesar
cipher, but it uses a series of Caesar ciphers for polyalphabetic
substitution. The substitution for each letter in the plaintext
is determined by a fixed keyword.
So, for example, if you want to encode the message “meet
at dawn,” the first step is to pick a keyword that will determine
which Caesar cipher to use. We’ll use the keyword “ALLY”
here. You repeat the keyword for as many characters as there
are in your original message:
MEET AT DAWN
ALLY AL LYAL
Now the number of rightward shifts to make to encode each
character is set by the character of the keyword that lines up
with it. The ’A’ means a shift of 0, so the initial M will remain
M. But the ’L’ for our second character sets a rightward shift
of 11, so ’E’ becomes ’P’. And so on, so “meet at dawn” encoded
with the keyword “ALLY” becomes “MPPR AE OYWY.”
Like the Caesar cipher, you can find all kinds of resources to
help you understand the cipher and also many examples writ-
ten in Haskell. Consider using a combination of chr,ord, and

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 693
modagain, possibly very similar to what you used for writing
the original Caesar cipher.
As-patterns
As-patterns in Haskell are a nifty way to be able to pattern match
on part of something and still refer to the entire original value.
Some examples:
f::Showa=>(a, b)->IO(a, b)
ft@(a,_)= do
print a
return t
Here we pattern-matched on a tuple so we could get at the
first value for printing, but used the @symbol to introduce a
binding named 𝑡in order to refer to the whole tuple rather
than just a part.
Prelude> f (1, 2)
1
(1,2)
We can use as-patterns with pattern matching on arbitrary
data constructors, which includes lists:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 694
doubleUp ::[a]->[a]
doubleUp []=[]
doubleUp xs@(x:_)=x:xs
Prelude> doubleUp []
[]
Prelude> doubleUp [1]
[1,1]
Prelude> doubleUp [1, 2]
[1,1,2]
Prelude> doubleUp [1, 2, 3]
[1,1,2,3]
Use as-patterns in implementing the following functions:
1.This should return Trueif (and only if) all the values in
the first list appear in the second list, though they need
not be contiguous.
isSubseqOf ::(Eqa)
=>[a]
->[a]
->Bool
The following are examples of how this function should
work:

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 695
Prelude> isSubseqOf "blah" "blahwoot"
True
Prelude> isSubseqOf "blah" "wootblah"
True
Prelude> isSubseqOf "blah" "wboloath"
True
Prelude> isSubseqOf "blah" "wootbla"
False
Prelude> isSubseqOf "blah" "halbwoot"
False
Prelude> isSubseqOf "blah" "blawhoot"
True
Remember that the sub-sequence has to be in the original
order!
2.Split a sentence into words, then tuple each word with the
capitalized form of each.
capitalizeWords ::String
->[(String,String)]
Prelude> capitalizeWords "hello world"
[("hello", "Hello"), ("world", "World")]
Language exercises
1.Write a function that capitalizes a word.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 696
capitalizeWord ::String->String
capitalizeWord =undefined
Example output.
Prelude> capitalizeWord "Chortle"
"Chortle"
Prelude> capitalizeWord "chortle"
"Chortle"
2.Write a function that capitalizes sentences in a paragraph.
Recognize when a new sentence has begun by checking
for periods. Reuse the capitalizeWord function.
capitalizeParagraph ::String->String
capitalizeParagraph =undefined
Example result you should get from your function:
Prelude> let s = "blah. woot ha."
Prelude> capitalizeParagraph s
"Blah. Woot ha."
Phone exercise
This exercise by geophf8originally for 1HaskellADay.9Thank
you for letting us use this exercise!
8https://twitter.com/geophf
9https://twitter.com/1haskelladay

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 697
Remember old-fashioned phone inputs for writing text
where you had to press a button multiple times to get diﬀerent
letters to come up? You may still have to do this when you try
to search for a movie to watch using your television remote
control. You’re going to write code to translate sequences of
button presses into strings and vice versa.
So! Here is the layout of the phone:
-----------------------------------------
| 1 | 2 ABC | 3 DEF |
_________________________________________
| 4 GHI | 5 JKL | 6 MNO |
-----------------------------------------
| 7 PQRS | 8 TUV | 9 WXYZ |
-----------------------------------------
| * ^ | 0 + _ | # ., |
-----------------------------------------
Where star (*) gives you capitalization of the letter you’re
writing to your friends, and 0 is your space bar. To represent
the digit itself, you press that digit once more than the letters it
represents. If you press a button one more than is required to
type the digit, it wraps around to the first letter. For example,
2 -> 'A'
22 -> 'B'
222 -> 'C'

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 698
2222 -> '2'
22222 -> 'A'
So on and so forth. We’re going to kick this around.
1.Create a data structure that captures the phone layout
above. Thedatastructureshouldbeabletoexpressenough
of how the layout works that you can use it to dictate the
behavior of the functions in the following exercises.
-- fill in the rest.
dataDaPhone =DaPhone
2.Convert the following conversations into the keypresses
required to express them. We’re going to suggest types
and functions to fill in order to accomplish the goal, but
they’re not obligatory. If you want to do it diﬀerently, go
right ahead.

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 699
convo::[String]
convo=
["Wanna play 20 questions" ,
"Ya",
"U 1st haha" ,
"Lol ok. Have u ever tasted alcohol" ,
"Lol ya" ,
"Wow ur cool haha. Ur turn" ,
"Ok. Do u think I am pretty Lol" ,
"Lol ya" ,
"Just making sure rofl ur turn" ]

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 700
-- validButtons = "1234567890*#"
typeDigit=Char
-- Valid presses: 1 and up
typePresses =Int
reverseTaps ::DaPhone
->Char
->[(Digit,Presses)]
reverseTaps =undefined
-- assuming the default phone definition
-- 'a' -> [('2', 1)]
-- 'A' -> [('*', 1), ('2', 1)]
cellPhonesDead ::DaPhone
->String
->[(Digit,Presses)]
cellPhonesDead =undefined
3.How many times do digits need to be pressed for each
message?
fingerTaps ::[(Digit,Presses)]->Presses
fingerTaps =undefined
4.What was the most popular letter for each message? What

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 701
wasitscost? You’llwanttocombine reverseTaps andfingerTaps
to figure out what it cost in taps. reverseTaps is a list be-
cause you need to press a diﬀerent button in order to get
capitals.
mostPopularLetter ::String->Char
mostPopularLetter =undefined
5.What was the most popular letter overall? What was the
most popular word?
coolestLtr ::[String]->Char
coolestLtr =undefined
coolestWord ::[String]->String
coolestWord =undefined
Hutton’s Razor
Hutton’s Razor10is a very simple expression language that
expresses integer literals and addition of values in that expres-
sion language. The “trick” to it is that it’s recursive and the
two expressions you’re summing together could be literals or
themselves further addition operations. This sort of datatype
is stereotypical of expression languages used to motivate ideas
in research papers and functional pearls. Evaluating or folding

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 702
a datatype is also in some sense what you’re doing most of the
time while programming anyway.
1.Your first task is to write the “eval” function which reduces
an expression to a final sum.
dataExpr
=LitInteger
|AddExprExpr
eval::Expr->Integer
eval=error"do it to it"
Example of expected output:
Prelude> eval (Add (Lit 1) (Lit 9001))
9002
2.Write a printer for the expressions.
printExpr ::Expr->String
printExpr =undefined
Expected output:
10http://www.cs.nott.ac.uk/~pszgmh/bib.html#semantics

CHAPTER 11. RULE THE TYPES, RULE THE UNIVERSE 703
Prelude> printExpr (Add (Lit 1) (Lit 9001))
"1 + 9001"
Prelude> let a1 = Add (Lit 9001) (Lit 1)
Prelude> let a2 = Add a1 (Lit 20001)
Prelude> let a3 = Add (Lit 1) a2
Prelude> printExpr a3
"1 + 9001 + 1 + 20001"
11.19 Definitions
1.Adatatype is how we declare and create data for our func-
tions to receive as inputs. Datatype declarations begin
with the keyword data. A datatype is made up of a type
constructor and zero or more data constructors which
each have zero or more arguments.

Chapter 12
Signaling adversity
Thank goodness we don’t
have only serious
problems, but ridiculous
ones as well
Edsger W. Dijkstra
704

CHAPTER 12. SIGNALING ADVERSITY 705
12.1 Signaling adversity
Sometimes it’s not convenient or possible for every value
in a datatype to make sense for your programs. When that
happens in Haskell, we use explicit datatypes to signal when
our functions receiveda combination of inputs that don’t make
sense. Later, we’ll see how to defend against those adverse
inputs at the time we construct our datatypes, but the Maybe
andEither datatypes we will demonstrate here are common.
This chapter will include:
•Nothing , orJust Maybe ;
•Either left or right, but not both;
•higher-kindedness;
•anamorphisms, but not animorphs.
12.2 How I learned to stop worrying
and love Nothing
Let’s consider the definition of Maybeagain:
dataMaybea=Nothing |Justa
You don’t need to define this yourself, as it’s included in the
Prelude by default. It’s also a very common datatype in Haskell

CHAPTER 12. SIGNALING ADVERSITY 706
because it lets us return a default Nothing value when we don’t
have any sensible values to return for our intended type 𝑎.
In the following intentionally simplistic function, we could
do several things with the odd numbers — we could return
them unmodified, we could modify them in some way dif-
ferent from the evens, we could return a zero, or we could
write an explicit signal that nothing happened because the
number wasn’t even:
ifEvenAdd2 ::Integer ->Integer
ifEvenAdd2 n=
ifeven nthenn+2else ???
What can we do to make it say, “hey, this number wasn’t
even so I have nothing for you, my friend?” Instead of promis-
ing anInteger result, we can return Maybe Integer :
ifEvenAdd2 ::Integer ->MaybeInteger
ifEvenAdd2 n=
ifeven nthenn+2elseNothing
This isn’t quite complete or correct either. While Nothing
has the type Maybe a , and𝑎can be assumed to be any type the
Maybeconstructor could contain, n+2is still of the type Integer .
We need to wrap that value in the other constructor Maybe
provides: Just. Here’s the error you’d get if you tried to load
it:

CHAPTER 12. SIGNALING ADVERSITY 707
<interactive>:9:75:
Couldn't match expected type
‘Maybe Integer’
with actual type ‘Integer’
In the first argument of ‘(+)’, namely ‘n’
In the expression: n + 2
And here’s how we fix it:
ifEvenAdd2 ::Integer ->MaybeInteger
ifEvenAdd2 n=
ifeven nthenJust(n+2)elseNothing
We had to parenthesize n+2because function application
binds the most tightly in Haskell (has the highest precedence),
so the compiler otherwise would’ve parsed it as (Just n) + 2 ,
which is wrong and throws a type error. Now our function
is correct and explicit about the possibility of not getting a
result!
Smart constructors for datatypes
Let’s consider a Person type which keeps track of two things,
their name and their age. We’ll write this up as a simple prod-
uct type (note that NameandAgeare type aliases):

CHAPTER 12. SIGNALING ADVERSITY 708
typeName=String
typeAge=Integer
dataPerson=PersonNameAgederiving Show
There are already a few problems here. One is that we could
construct a Person with an empty String for a name or make
a person who is negative years old. This is no problem to fix
withMaybe, though:
typeName=String
typeAge=Integer
dataPerson=PersonNameAgederiving Show
mkPerson ::Name->Age->MaybePerson
mkPerson name age
|name/=""&&age>=0=
Just$Personname age
|otherwise =Nothing
And if you load this into your REPL:
Prelude> mkPerson "John Browning" 160
Just (Person "John Browning" 160)
Cool. What happens when we feed it adverse data?

CHAPTER 12. SIGNALING ADVERSITY 709
Prelude> mkPerson "" 160
Nothing
Prelude> mkPerson "blah" 0
Just (Person "blah" 0)
Prelude> mkPerson "blah" (-9001)
Nothing
mkPerson is what we call a smart constructor . It allows us to
construct values of a type only when they meet certain criteria,
so that we know we have a valid value, and return an explicit
signal when we do not.
This is much better than our original, but what if we want
to know if it was the name, age, or both that was bad? We may
want to tell our user something was wrong with their input.
Fortunately, we have a datatype for that!
12.3 Bleating either
We want a way to express why we didn’t get a successful result
back from our mkPerson constructor. To handle that, we’ve got
theEither datatype which is defined as follows in the Prelude :
dataEithera b=Lefta|Rightb
What we want is a way to know whyour inputs were incor-
rectifthey were incorrect. So we’ll start by making a sum type
to enumerate our failure modes:

CHAPTER 12. SIGNALING ADVERSITY 710
dataPersonInvalid =NameEmpty
|AgeTooLow
deriving (Eq,Show)
By now, you know why we derived Show, but it’s important
that we derive Eqbecause otherwise we can’t equality check the
constructors. Pattern matching is a case expression, where the
data constructor is the condition. Case expressions and pattern
matching will work without an Eqinstance, but guards using
(==)will not. As we’ve shown you previously, you can write
your own Eqinstance for your datatype if you want a specific
behavior, but it’s usually not necessary to do, so we will usually
derive the Eqinstance. Here’s the diﬀerence demonstrated in
code:
moduleEqCaseGuard where
dataPersonInvalid =NameEmpty
|AgeTooLow
-- Compiles without Eq
toString ::PersonInvalid ->String
toString NameEmpty ="NameEmpty"
toString AgeTooLow ="AgeTooLow"

CHAPTER 12. SIGNALING ADVERSITY 711
instance ShowPersonInvalid where
show=toString
-- This does not work without an
-- Eq instance
blah::PersonInvalid ->String
blahpi
|pi==NameEmpty ="NameEmpty"
|pi==AgeTooLow ="AgeTooLow"
|otherwise ="???"
It’s worth considering that if you needed to have an Eqin-
stance to pattern match, how would you write the Eqinstances?
Next our constructor type is going to change to:
mkPerson ::Name
->Age
->EitherPersonInvalid Person
This signifies that we’re going to get a Person value if we suc-
ceed but a PersonInvalid if it fails. Now we need to change our
logic to return PersonInvalid values inside a Leftconstructor
when the data is invalid, discriminating by each case as we go:

CHAPTER 12. SIGNALING ADVERSITY 712
typeName=String
typeAge=Integer
dataPerson=PersonNameAgederiving Show
dataPersonInvalid =NameEmpty
|AgeTooLow
deriving (Eq,Show)
mkPerson ::Name
->Age
->EitherPersonInvalid Person
-- [1] [2] [3]
mkPerson name age
|name/=""&&age>=0=
Right$Personname age
-- [4]
|name==""=LeftNameEmpty
-- [5]
|otherwise =LeftAgeTooLow
1.OurmkPerson type takes a NameandAgereturns an Either
result.

CHAPTER 12. SIGNALING ADVERSITY 713
2.TheLeftresult of the Either is an invalid person, when
either the name or age is an invalid input.
3.TheRightresult is a valid person.
4.The first case of our mkPerson function, then, matches on
theRightconstructor of the Either and returns a Person
result. We could have written
name/=""&&age>=0=
Right(Personname age)
instead of using the dollar sign.
5.The next two cases match on the Leftconstructor and
allow us to tailor our invalid results based on the failure
reasons. We can pattern match on Leftbecause it’s one of
the constructors of Either .
We use Leftas our invalid or error constructor for a couple
of reasons. It is conventional to do so in Haskell, but that con-
vention came about for a reason. The reason has to do with the
ordering of type arguments and application of functions. Nor-
mally it is your error or invalid result that is going to cause a
stop to whatever work is being done by your program. Functor
will not map over the left type argument because it has been
applied away. You may remember Functor from our introduc-
tion offmapback in the chapter about lists; don’t worry, a full

CHAPTER 12. SIGNALING ADVERSITY 714
explanation of Functor is coming soon. Since you normally
want to apply functions and map over the case that doesn’t
stop your program (that is, notthe error case), it has become
convention that the LeftofEither is used for whatever case is
going to cause the work to stop.
Let’s see what it looks like when we have good data, although
Djali isn’t a person.1
Prelude> :t mkPerson "Djali" 5
mkPerson "Djali" 5 :: Either PersonInvalid Person
Prelude> mkPerson "Djali" 5
Right (Person "Djali" 5)
Then we can see what this does for us when dealing with
bad data:
Prelude> mkPerson "" 10
Left NameEmpty
Prelude> mkPerson "Djali" (-1)
Left AgeTooLow
Prelude> mkPerson "" (-1)
Left NameEmpty
Notice in the last example that when both the name and
the age are wrong, we’re only going to see the result of the first
failure case, not both.
1Don’t know what we mean? Check the name Djali on a search engine.

CHAPTER 12. SIGNALING ADVERSITY 715
This is imperfect in one respect, as it doesn’t let us express a
list of errors. We can fix this, too! One thing that will change is
that instead of validating all the data for a Person at once, we’re
going to make separate checking functions and then combine
the results. We’ll see means of abstracting patterns like this
out later. We’re adding a type alias that wasn’t in our previous
version; otherwise, these types are the same as above:
typeName=String
typeAge=Integer
typeValidatePerson a=
Either[PersonInvalid ] a
dataPerson=PersonNameAgederiving Show
dataPersonInvalid =NameEmpty
|AgeTooLow
deriving (Eq,Show)
Now we’ll write our checking functions. Although more
than one thing could hypothetically be wrong with the age
value, we’ll keep this simple and only check to make sure it’s a
positive Integer value:

CHAPTER 12. SIGNALING ADVERSITY 716
ageOkay ::Age
->Either[PersonInvalid ]Age
ageOkay age= caseage>=0of
True->Rightage
False->Left[AgeTooLow ]
nameOkay ::Name
->Either[PersonInvalid ]Name
nameOkay name= casename/=""of
True->Rightname
False->Left[NameEmpty ]
We can nest the PersonInvalid sum type right into the Left
position of Either, just as we saw in the previous chapter (al-
though we weren’t using Either there, but similar types).
A couple of things to note here:
•TheNamevalue will only return this invalid result when
it’s an empty String .
•SinceNameis only a String value, it can be any String with
characters inside it, so “42” is still going to be returned as
a valid name. Try it.
•If you try to put an Integer in for the name, you won’t get
aLeftresult, you’ll get a type error. Try it. You’ll get a

CHAPTER 12. SIGNALING ADVERSITY 717
similar result if you try to feed a string value to the ageOkay
function.
•We’re going to return a list of PersonInvalid results. That
will allow us to return bothNameEmpty andAgeTooLow in cases
where both of those are true.
Now that our functions rely on Either to validate that the
age and name values are independently valid, we can write a
mkPerson function that will use our type alias ValidatePerson :

CHAPTER 12. SIGNALING ADVERSITY 718
mkPerson ::Name
->Age
->ValidatePerson Person
-- [1] [2]
mkPerson name age =
mkPerson' (nameOkay name) (ageOkay age)
-- [3] [4] [5]
mkPerson' ::ValidatePerson Name
->ValidatePerson Age
->ValidatePerson Person
-- [6]
mkPerson' (RightnameOk) ( RightageOk)=
Right(PersonnameOk ageOk)
mkPerson' (LeftbadName) ( LeftbadAge) =
Left(badName ++badAge)
mkPerson' (LeftbadName) _ =LeftbadName
mkPerson' _(LeftbadAge) =LeftbadAge
1.A type alias for Either [PersonInvalid] a .
2.This is the 𝑎argument to ValidatePerson type.
3.Ourmainfunctionnowreliesonasimilarly-namedhelper

CHAPTER 12. SIGNALING ADVERSITY 719
function.
4.First argument to this function is the result of the nameOkay
function.
5.Second argument is the result of the ageOkay function.
6.The type relies on the synonym for Either .
The rest of our helper function mkPerson' consists of plain
old pattern matches.
Now let’s see what we get:
Prelude> mkPerson "" (-1)
Left [NameEmpty,AgeTooLow]
Ahh, that’s more like it. Now we can tell the user what was
incorrect in one go without them having to round-trip each
mistake! Later in the book, we’ll be able to replace mkPerson
andmkPerson' with the following:
mkPerson
::Name
->Age
->Validation [PersonInvalid ]Person
mkPerson name age =
liftA2
Person(nameOkay name) (ageOkay age)

CHAPTER 12. SIGNALING ADVERSITY 720
12.4 Kinds, a thousand stars in your
types
Kinds are types one level up. They are used to describe the
types of type constructors. One noteworthy feature of Haskell
is that it has higher-kinded types . Here the term ‘higher-kinded’
derives from higher-order functions, functions that take more
functions as arguments. Type constructors (that is, higher-
kinded types) are types that take more types as arguments. The
Haskell Report uses the term type constant to refer to types that
take no arguments and are already types. In the Report, type
constructor is used to refer to types which must have arguments
applied to become a type.
As we discussed in the last chapter, these are examples of
type constants :
Prelude> :kind Int
Int :: *
Prelude> :k Bool
Bool :: *
Prelude> :k Char
Char :: *
The::syntax usually means “has type of,” but it is used for
kind signatures as well as type signatures.

CHAPTER 12. SIGNALING ADVERSITY 721
The following is an example of a type that has a type con-
structor rather than a type constant :
dataExample a=Blah|RoofGoats |Woota
Example is a type constructor rather than a constant because
it takes a type argument 𝑎which is used with the Wootdata
constructor. In GHCi we can query kinds with :k:
Prelude> data Example a = Blah | RoofGoats | Woot a
Prelude> :k Example
Example :: * -> *
Example has one parameter, so it must be applied to one type
in order to become a concrete type represented by a single *.
The two-tuple takes two arguments, so it must be applied to
two types to become a concrete type:
Prelude> :k (,)
(,) :: * -> * -> *
Prelude> :k (Int, Int)
(Int, Int) :: *
TheMaybeandEither datatypes we’ve just reviewed also have
type constructors rather than constants. They have to be ap-
plied to an argument before they become concrete types. As
with the eﬀect of currying in type signatures, applying Maybe

CHAPTER 12. SIGNALING ADVERSITY 722
to an𝑎type constructor relieves us of one arrow and makes it
a kind star:
Prelude> :k Maybe
Maybe :: * -> *
Prelude> :k Maybe Int
Maybe Int :: *
On the other hand, Either has to be applied to two argu-
ments, an 𝑎and a𝑏, so the kind of Either is star to star to star:
Prelude> :k Either
Either :: * -> * -> *
And, again, we can query the eﬀects of applying it to argu-
ments:
Prelude> :k Either Int
Either Int :: * -> *
Prelude> :k Either Int String
Either Int String :: *
As we’ve said, the kind *represents a concrete type. There
is nothing left awaiting application.

CHAPTER 12. SIGNALING ADVERSITY 723
Lifted and unlifted types To be precise, kind *is the kind of
all standard lifted types, while types that have the kind #are
unlifted. A lifted type, which includes any datatype you could
define yourself, is any that can be inhabited by bottom . Lifted
types are represented by a pointer and include most of the
datatypes we’ve seen and most that you’re likely to encounter
and use. Unlifted types are any type which cannot be inhabited
by bottom. Types of kind #are often native machine types
and raw pointers. Newtypes are a special case in that they are
kind*, but are unlifted because their representation is identical
to that of the type they contain, so the newtype itself is not
creating any new pointer beyond that of the type it contains.
That fact means that the newtype itself cannot be inhabited
by bottom, only the thing it contains can be, so newtypes are
unlifted. The default kind of concrete, fully-applied datatypes
in GHC is kind *.
Now what happens if we let our type constructor take an
argument?
Prelude> data Identity a = Identity a
Prelude> :k Identity
Identity :: * -> *
As we discussed in the previous chapter, the arrow in the
kind signature, like the function arrow in type signatures, sig-
nals a need for application. In this case, we construct the type
by applying it to another type.

CHAPTER 12. SIGNALING ADVERSITY 724
Let’s consider the case of Maybe, which is defined as follows:
dataMaybea=Nothing |Justa
The type Maybeis a type constructor because it takes one
argument before it becomes a concrete type:
Prelude> :k Maybe
Maybe :: * -> *
Prelude> :k Maybe Int
Maybe Int :: *
Prelude> :k Maybe Bool
Maybe Bool :: *
Prelude> :k Int
Int :: *
Prelude> :k Bool
Bool :: *
Whereas the following will not work, because the kinds
don’t match up:
Prelude> :k Maybe Maybe
Expecting one more argument to ‘Maybe’

CHAPTER 12. SIGNALING ADVERSITY 725
The first argument of ‘Maybe’ should have kind ‘*’,
but ‘Maybe’ has kind ‘* -> *’
In a type in a GHCi command: Maybe Maybe
Maybeexpects a single type argument of kind *, which Maybe
is not.
If we give Maybea type argument that is kind *, it also be-
comes kind *so then it can be an argument to another Maybe:
Prelude> :k Maybe Char
Maybe Char :: *
Prelude> :k Maybe (Maybe Char)
Maybe (Maybe Char) :: *
OurExample datatype from earlier also won’t work as an
argument for Maybeby itself:
Prelude> data Example a = Blah | RoofGoats | Woot a
Prelude> :k Maybe Example
Expecting one more argument to ‘Example’
The first argument of ‘Maybe’ should have kind ‘*’,

CHAPTER 12. SIGNALING ADVERSITY 726
but ‘Example’ has kind ‘* -> *’
In a type in a GHCi command: Maybe Example
However, if we apply the Example type constructor, we can
make it work and create a value of that type:
Prelude> :k Maybe (Example Int)
Maybe (Example Int) :: *
Prelude> :t Just (Woot n)
Just (Woot n) :: Maybe (Example Int)
Note that the list type constructor []is also kind * -> * and
otherwise unexceptional save for the bracket syntax that lets
you type [a]and[Int]instead of [] aand[] Int :
Prelude> :k []
[] :: * -> *
Prelude :k [] Int
[] Int :: *
Prelude> :k [Int]
[Int] :: *
So, we can’t have a Maybe [] for the same reason we couldn’t
have aMaybe Maybe , but we can have a Maybe [Bool] :

CHAPTER 12. SIGNALING ADVERSITY 727
Prelude> :k Maybe []
Expecting one more argument to ‘[]’
The first argument of ‘Maybe’ should have kind ‘*’,
but ‘[]’ has kind ‘* -> *’
In a type in a GHCi command: Maybe []
Prelude> :k Maybe [Bool]
Maybe [Bool] :: *
If you recall, one of the first times we used Maybein the book
was to write a safe version of a tailfunction back in the chapter
on lists:
safeTail ::[a]->Maybe[a]
safeTail []=Nothing
safeTail (x:[])=Nothing
safeTail (_:xs)=Justxs
As soon as we apply this to a value, the polymorphic type
variables become constrained or concrete types:
Prelude> safeTail "julie"
Just "ulie"
Prelude> :t safeTail "julie"
safeTail "julie" :: Maybe [Char]

CHAPTER 12. SIGNALING ADVERSITY 728
Prelude> safeTail [1..10]
Just [2,3,4,5,6,7,8,9,10]
Prelude> :t safeTail [1..10]
safeTail [1..10] :: (Num a, Enum a) => Maybe [a]
Prelude> :t safeTail [1..10 :: Int]
safeTail [1..10 :: Int] :: Maybe [Int]
We can expand on type constructors that take a single argu-
ment and see how the kind changes as we go:
Prelude> data Trivial = Trivial
Prelude> :k Trivial
Trivial :: *
Prelude> data Unary a = Unary a
Prelude> :k Unary
Unary :: * -> *
Prelude> data TwoArgs a b = TwoArgs a b
Prelude> :k TwoArgs
TwoArgs :: * -> * -> *
Prelude> data ThreeArgs a b c = ThreeArgs a b c
Prelude> :k ThreeArgs
ThreeArgs :: * -> * -> * -> *

CHAPTER 12. SIGNALING ADVERSITY 729
It may not be clear why this is useful to know right now,
other than helping to understand when your type errors are
caused by things not being fully applied. The implications of
higher-kindedness will be clearer in a later chapter.
Data constructors are functions
In the previous chapter, we noted the diﬀerence between data
constants and data constructors and noted that data construc-
tors that haven’t been fully applied have function arrows in
them. Once you apply them to their arguments, they return a
value of the appropriate type. In other words, data construc-
tors are functions. As it happens, they behave like Haskell
functions in that they are curried as well.
First let’s observe that nullary data constructors, which are
values taking no arguments, are notlike functions:
Prelude> data Trivial = Trivial deriving Show
Prelude> Trivial 1
Couldn't match expected type ‘Integer -> t’
with actual type ‘Trivial’
(... etc ...)
However, data constructors that take arguments dobehave
like functions:

CHAPTER 12. SIGNALING ADVERSITY 730
Prelude> data UnaryC = UnaryC Int deriving Show
Prelude> :t UnaryC
UnaryC :: Int -> UnaryC
Prelude> UnaryC 10
UnaryC 10
Prelude> :t UnaryC 10
UnaryC 10 :: UnaryC
Like functions, their arguments are typechecked against
the specification in the type:
Prelude> UnaryC "blah"
Couldn't match expected type ‘Int’
with actual type ‘[Char]’
If we wanted a unary data constructor which could contain
any type, we would parameterize the type like so:
Prelude> data Unary a = Unary a deriving Show
Prelude> :t Unary
Unary :: a -> Unary a
Prelude> :t Unary 10
Unary 10 :: Num a => Unary a
Prelude> :t Unary "blah"
Unary "blah" :: Unary [Char]

CHAPTER 12. SIGNALING ADVERSITY 731
And again, this works just like a function, except the type
of the argument can be whatever we want.
Note that if we want to use a derived (GHC generated) Show
instance for Unary, it has to be able to also show the contents,
the type 𝑎value contained by Unary’s data constructor:
Prelude> :info Unary
data Unary a = Unary a
instance Show a => Show (Unary a)
If we try to use a type for 𝑎that does not have a Showinstance,
it won’t cause a problem until we try to show the value:
Prelude> :t (Unary id)
(Unary id) :: Unary (t -> t)
-- id doesn't have a Show instance
Prelude> show (Unary id)
<interactive>:53:1:
No instance for (Show (t0 -> t0))
...
The only way to avoid this would be to write an instance that
did not show the value contained in the Unarydata constructor,
but that would be somewhat unusual.

CHAPTER 12. SIGNALING ADVERSITY 732
Another thing to keep in mind is that you can’t ordinarily
hide polymorphic types from your type constructor, so the
following is invalid:
Prelude> data Unary = Unary a deriving Show
Not in scope: type variable ‘a’
In order for the type variable 𝑎to be in scope, we usually
need to introduce it with our type constructor. There are ways
around this, but they’re rarely necessary or a good idea and
not relevant to the beginning Haskeller.
Here’s an example using fmapand the Justdata constructor
fromMaybeto demonstrate how Justis also like a function:
Prelude> fmap Just [1, 2, 3]
[Just 1,Just 2,Just 3]
The significance and utility of this may not be immediately
obvious but will be more clear in later chapters.
12.5 Chapter Exercises
Determine the kinds
1.Given
id::a->a

CHAPTER 12. SIGNALING ADVERSITY 733
What is the kind of a?
2.r::a->f a
What are the kinds of aandf?
String processing
Because this is the kind of thing linguists ahemenjoy doing in
their spare time.
1.Write a recursive function named replaceThe which takes a
text/string, breaks it into words and replaces each instance
of “the” with “a”. It’s intended only to replace exactly
the word “the”. notThe is a suggested helper function for
accomplishing this.
-- example GHCi session
-- above the functions
-- >>> notThe "the"
-- Nothing
-- >>> notThe "blahtheblah"
-- Just "blahtheblah"
-- >>> notThe "woot"
-- Just "woot"
notThe::String->MaybeString
notThe=undefined

CHAPTER 12. SIGNALING ADVERSITY 734
-- >>> replaceThe "the cow loves us"
-- "a cow loves us"
replaceThe ::String->String
replaceThe =undefined
2.Write a recursive function that takes a text/string, breaks
it into words, and counts the number of instances of ”the”
followed by a vowel-initial word.
-- >>> countTheBeforeVowel "the cow"
-- 0
-- >>> countTheBeforeVowel "the evil cow"
-- 1
countTheBeforeVowel ::String->Integer
countTheBeforeVowel =undefined
3.Return the number of letters that are vowels in a word.
Hint: it’s helpful to break this into steps. Add any helper
functions necessary to achieve your objectives.
a)Test for vowelhood
b)Return the vowels of a string
c)Count the number of elements returned

CHAPTER 12. SIGNALING ADVERSITY 735
-- >>> countVowels "the cow"
-- 2
-- >>> countVowels "Mikolajczak"
-- 4
countVowels ::String->Integer
countVowels =undefined
Validate the word
Use the Maybetype to write a function that counts the number
of vowels in a string and the number of consonants. If the
number of vowels exceeds the number of consonants, the
function returns Nothing . In many human languages, vowels
rarely exceed the number of consonants so when they do, it
mayindicate the input isn’t a word (that is, a valid input to your
dataset):
newtype Word'=
Word'String
deriving (Eq,Show)
vowels="aeiou"
mkWord::String->MaybeWord'
mkWord=undefined

CHAPTER 12. SIGNALING ADVERSITY 736
It’s only Natural
You’ll be presented with a datatype to represent the natural
numbers. The only values representable with the naturals
are whole numbers from zero to infinity. Your task will be
to implement functions to convert Natural s toInteger s and
Integer s toNatural s. The conversion from Natural s toInteger s
won’t return Maybebecause Integer is a strict superset of Natural .
AnyNatural can be represented by an Integer , but the same is
nottrue of any Integer . Negative numbers are not valid natural
numbers.

CHAPTER 12. SIGNALING ADVERSITY 737
-- As natural as any
-- competitive bodybuilder
dataNat=
Zero
|SuccNat
deriving (Eq,Show)
-- >>> natToInteger Zero
-- 0
-- >>> natToInteger (Succ Zero)
-- 1
-- >>> natToInteger (Succ (Succ Zero))
-- 2
natToInteger ::Nat->Integer
natToInteger =undefined
-- >>> integerToNat 0
-- Just Zero
-- >>> integerToNat 1
-- Just (Succ Zero)
-- >>> integerToNat 2
-- Just (Succ (Succ Zero))
-- >>> integerToNat (-1)
-- Nothing
integerToNat ::Integer ->MaybeNat
integerToNat =undefined

CHAPTER 12. SIGNALING ADVERSITY 738
Small library for Maybe
Write the following functions. This may take some time.
1.Simple boolean checks for Maybevalues.
-- >>> isJust (Just 1)
-- True
-- >>> isJust Nothing
-- False
isJust::Maybea->Bool
-- >>> isNothing (Just 1)
-- False
-- >>> isNothing Nothing
-- True
isNothing ::Maybea->Bool
2.The following is the Maybecatamorphism. You can turn a
Maybevalue into anything else with this.
-- >>> mayybee 0 (+1) Nothing
-- 0
-- >>> mayybee 0 (+1) (Just 1)
-- 2
mayybee ::b->(a->b)->Maybea->b

CHAPTER 12. SIGNALING ADVERSITY 739
3.In case you just want to provide a fallback value.
-- >>> fromMaybe 0 Nothing
-- 0
-- >>> fromMaybe 0 (Just 1)
-- 1
fromMaybe ::a->Maybea->a
-- Try writing it in terms
-- of the maybe catamorphism
4.Converting between ListandMaybe.
-- >>> listToMaybe [1, 2, 3]
-- Just 1
-- >>> listToMaybe []
-- Nothing
listToMaybe ::[a]->Maybea
-- >>> maybeToList (Just 1)
-- [1]
-- >>> maybeToList Nothing
-- []
maybeToList ::Maybea->[a]
5.For when we want to drop the Nothing values from our list.

CHAPTER 12. SIGNALING ADVERSITY 740
-- >>> catMaybes [Just 1, Nothing, Just 2]
-- [1, 2]
-- >>> let xs = take 3 $ repeat Nothing
-- >>> catMaybes xs
-- []
catMaybes ::[Maybea]->[a]
6.You’ll see this called “sequence” later.
-- >>> flipMaybe [Just 1, Just 2, Just 3]
-- Just [1, 2, 3]
-- >>> flipMaybe [Just 1, Nothing, Just 3]
-- Nothing
flipMaybe ::[Maybea]->Maybe[a]
Small library for Either
Write each of the following functions. If more than one possi-
ble unique function exists for the type, use common sense to
determine what it should do.
1.Try to eventually arrive at a solution that uses foldr, even
if earlier versions don’t use foldr.
lefts'::[Eithera b]->[a]
2.Same as the last one. Use foldreventually.

CHAPTER 12. SIGNALING ADVERSITY 741
rights' ::[Eithera b]->[b]
3.partitionEithers' ::[Eithera b]
->([a], [b])
4.eitherMaybe' ::(b->c)
->Eithera b
->Maybec
5.This is a general catamorphism for Either values.
either' ::(a->c)
->(b->c)
->Eithera b
->c
6.Same as before, but use the either' function you just
wrote.
eitherMaybe'' ::(b->c)
->Eithera b
->Maybec
Mostofthefunctionsyoujustsawareinthe Prelude ,Data.Maybe ,
orData.Either but you should strive to write them yourself
without looking at existing implementations. You will deprive
yourself if you cheat.

CHAPTER 12. SIGNALING ADVERSITY 742
Unfolds
While the idea of catamorphisms is still relatively fresh in our
minds, let’s turn our attention to their dual: anamorphisms . If
folds, or catamorphisms, let us break data structures down
then unfolds let us build them up. There are, as with folds, a
few diﬀerent ways to unfold a data structure. We can use them
to create finite and infinite data structures alike.
-- iterate is like a limited
-- unfold that never ends
Prelude> :t iterate
iterate :: (a -> a) -> a -> [a]
-- because it never ends, we must use
-- take to get a finite list
Prelude> take 10 $ iterate (+1) 0
[0,1,2,3,4,5,6,7,8,9]
-- unfoldr is more general
Prelude> :t unfoldr
unfoldr :: (b -> Maybe (a, b)) -> b -> [a]
-- Using unfoldr to do
-- the same thing as iterate
Prelude> take 10 $ unfoldr (\b -> Just (b, b+1)) 0

CHAPTER 12. SIGNALING ADVERSITY 743
[0,1,2,3,4,5,6,7,8,9]
Why bother?
We bother with this for the same reason we abstracted direct
recursion into folds, such as with sum,product , andconcat .
importData.List
mehSum::Numa=>[a]->a
mehSumxs=go0xs
wherego::Numa=>a->[a]->a
go n[]=n
go n (x:xs)=(go (n+x) xs)
niceSum ::Numa=>[a]->a
niceSum =foldl' ( +)0
mehProduct ::Numa=>[a]->a
mehProduct xs=go1xs
wherego::Numa=>a->[a]->a
go n[]=n
