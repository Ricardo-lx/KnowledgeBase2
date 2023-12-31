as the input value to the next. While dosyntax works with
any monad — not just IO— it is most commonly seen when
usingIO. This section is going to talk about why dois sugar and
demonstrate what the joinofMonadcan do for us. We will be
using the IO Monad to demonstrate here, but later on we’ll see
some examples of dosyntax without IO.
To begin, let’s look at some correspondences:
(*>) :: Applicative f => f a -> f b -> f b
(>>) :: Monad m => m a -> m b -> m b
For our purposes, (*>)and(>>)are the same thing: sequenc-
ing functions, but with two diﬀerent constraints. They should
in all cases do the same thing:
Prelude> putStrLn "Hello, " >> putStrLn "World!"
Hello,
World!

CHAPTER 18. MONAD 1155
Prelude> putStrLn "Hello, " *> putStrLn "World!"
Hello,
World!
Not observably diﬀerent. Good enough for government
work!
We can see what dosyntax looks like after the compiler
desugars it for us by manually transforming it ourselves:
importControl.Applicative ((*>))
sequencing ::IO()
sequencing = do
putStrLn "blah"
putStrLn "another thing"
sequencing' ::IO()
sequencing' =
putStrLn "blah">>
putStrLn "another thing"
sequencing'' ::IO()
sequencing'' =
putStrLn "blah"*>
putStrLn "another thing"

CHAPTER 18. MONAD 1156
You should have had the same results for each of the above.
We can do the same with the variable binding that dosyntax
includes:
binding ::IO()
binding = do
name<-getLine
putStrLn name
binding' ::IO()
binding' =
getLine >>=putStrLn
Instead of naming the variable and passing that as an argu-
ment to the next function, we use >>=which passes it directly.
Whenfmapalone isn’t enough
Note that if you try to fmap putStrLn overgetLine , it won’t do
anything. Try typing this into your REPL:
Prelude> putStrLn <$> getLine
You’ve used getLine , so when you hit enter it should await
your input. Type something in, hit enter again and see what
happens.

CHAPTER 18. MONAD 1157
Whatever input you gave it didn’t print, although it seems
like it should have due to the putStrLn being mapped over the
getLine . We evaluated the IOaction that requests input, but not
the one that prints it. So, what happened?
Well, let’s start with the types. The type of what you tried
to do is this:
Prelude> :t putStrLn <$> getLine
putStrLn <$> getLine :: IO (IO ())
We’re going to break it down a little bit so that we’ll under-
stand why this didn’t work. First, getLine performs I/O to get
aString :
getLine ::IOString
AndputStrLn takes a String argument, performs I/O, and
returns nothing interesting — parents of children with an
allowance can sympathize:
putStrLn ::String->IO()
What is the type of fmapas it concerns putStrLn andgetLine ?

CHAPTER 18. MONAD 1158
-- The type we start with
<$> ::Functor f=>(a->b)->f a->f b
-- Our (a -> b) is putStrLn
(a ->b )
putStrLn ::String->IO()
That𝑏gets specialized to the type IO (), which is going to
jam another IOactioninside of the I/O that getLine performs.
Perhaps this looks familiar from our demonstration of what
happens when you use fmapto map a function with type (a ->
m b)instead of (a -> b) — that is what’s happening here. This
is what is happening with our types:
f::Functor f=>fString->f (IO())
fx=putStrLn <$>x
g::(String->b)->IOb
gx=x<$>getLine
putStrLn <$>getLine ::IO(IO())
Okay...so, which IOis which, and why does it ask for input
but not print what we typed in?

CHAPTER 18. MONAD 1159
-- [1] [2] [3]
h::IO(IO())
h=putStrLn <$>getLine
1.This outermost IOstructure represents the eﬀects getLine
must perform to get you a String that the user typed in.
2.This inner IOstructure represents the eﬀects that would
be performed ifputStrLn was evaluated.
3.The unit here is the unit that putStrLn returns.
One of the strengths of Haskell is that we can refer to, com-
pose, and map over eﬀectful computations without perform-
ing them or bending over backwards to make that pattern
work. For a simpler example of how we can wait to evalu-
ateIOactions (or any computation in general), consider the
following:
Prelude> let printOne = putStrLn "1"
Prelude> let printTwo = putStrLn "2"
Prelude> let twoActions = (printOne, printTwo)
Prelude> :t twoActions
twoActions :: (IO (), IO ())
With that tuple of two IOactions defined, we can now grab
one and evaluate it:

CHAPTER 18. MONAD 1160
Prelude> fst twoActions
1
Prelude> snd twoActions
2
Prelude> fst twoActions
1
Note that we are able to evaluate IOactions multiple times.
This will be significant later.
Back to our conundrum of why we can’t fmap putStrLn over
getLine . Perhaps you’ve already figured out what we need to do.
We need to join those two IOlayers together. To get what we
want, we need the unique thing that Monadoﬀers:join. Watch
it work:
Prelude> import Control.Monad (join)
Prelude> join $ putStrLn <$> getLine
blah
blah
Prelude> :t join $ putStrLn <$> getLine
join $ putStrLn <$> getLine :: IO ()
Whatjoindidhereis mergetheeﬀectsof getLine andputStrLn
into a single IOaction. This merged IOaction performs the
eﬀects in the order determined by the nesting of the IOactions.
As it happens, the cleanest way to express ordering in a lambda

CHAPTER 18. MONAD 1161
calculus without bolting on something unpleasant is through
nesting of expressions or lambdas.
That’s right. We still haven’t left the lambda calculus behind.
Monadic sequencing and dosyntax seem on the surface to
be very far removed from that. But they aren’t. As we said,
monadic actions are still pure, and the sequencing operations
we use here are ways of nesting lambdas. Now, IOis a bit dif-
ferent, as it does allow for side eﬀects, but since those eﬀects
are constrained within the IOtype, all the rest of it is still a
pure lambda calculus.
Sometimes it is valuable to suspend or otherwise not per-
form an I/O action until some determination is made, so types
likeIO (IO ()) aren’t necessarily invalid, but you should be
aware of what’s needed to make this example work.
Let’sgetbacktodesugaring dosyntaxwithournow-enriched
understanding of what monads do for us:

CHAPTER 18. MONAD 1162
bindingAndSequencing ::IO()
bindingAndSequencing = do
putStrLn "name pls:"
name<-getLine
putStrLn ( "y helo thar: " ++name)
bindingAndSequencing' ::IO()
bindingAndSequencing' =
putStrLn "name pls:" >>
getLine >>=
\name->
putStrLn ( "y helo thar: " ++name)
As the nesting intensifies, you can see how dosyntax can
make things a bit cleaner and easier to read:
twoBinds ::IO()
twoBinds = do
putStrLn "name pls:"
name<-getLine
putStrLn "age pls:"
age<-getLine
putStrLn ( "y helo thar: "
++name++" who is: "
++age++" years old." )

CHAPTER 18. MONAD 1163
twoBinds' ::IO()
twoBinds' =
putStrLn "name pls:" >>
getLine >>=
\name->
putStrLn "age pls:" >>
getLine >>=
\age->
putStrLn ( "y helo thar: "
++name++" who is: "
++age++" years old." )
18.4 Examples of Monaduse
All right, we’ve seen what is diﬀerent about Monadand seen a
small demonstration of what that does for us. What we need
now is to see how monads work in code, with Monads other than
IO.
List
We’ve been starting oﬀ our examples of these typeclasses in
use with list examples because they can be quite easy to see
and understand. We will keep this section brief, though, as we
have more exciting things to show you.

CHAPTER 18. MONAD 1164
Specializing the types
This process should be familiar to you by now:
(>>=)::Monadm
=>m a->(a->m b)->m b
(>>=)::[ ] a->(a->[ ] b)->[ ] b
-- or more syntactically common
(>>=)::[a]->(a->[b])->[b]
-- same as pure
return::Monadm=>a->m a
return:: a->[ ] a
return:: a->[a]
Excellent. It’s like fmapexcept the order of arguments is
flipped and we can now generate more list (or an empty list)
inside of our mapped function. Let’s take it for a spin.
Example of the List Monadin use
Let’s start with a function and identify how the parts fit with
our monadic types:

CHAPTER 18. MONAD 1165
twiceWhenEven ::[Integer]->[Integer]
twiceWhenEven xs= do
x<-xs
ifeven x
then[x*x, x*x]
else[x*x]
Thex <- xs line binds individual values out of the list input,
like a list comprehension, giving us an 𝑎. Theif-then-else is
oura -> m b . It takes the individual 𝑎values that have been
bound out of our m aand can generate more values, thereby
increasing the size of the list.
Them athat is our first input will be the argument we pass
to it below:
Prelude> twiceWhenEven [1..3]
[1,4,4,9]
Now try this:
twiceWhenEven ::[Integer]->[Integer]
twiceWhenEven xs= do
x<-xs
ifeven x
then[x*x, x*x]
else[]

CHAPTER 18. MONAD 1166
And try giving it the same input as above (for easy compar-
ison). Was the result what you expected? Keep playing around
with this, forming hypotheses about what will happen and
why and testing them in the REPL to develop an intuition for
how monads are working on a simple example. The examples
in the next sections are longer and more complex.
Maybe Monad
Now we come to a more exciting demonstration of what we
can do with our newfound power.
Specializing the types
It is the season for examining the types:
-- type M = Maybe
-- m ~ Maybe
(>>=)::Monadm
=>m a->(a->m b)->m b
(>>=)::
Maybea->(a->Maybeb)->Maybeb
-- same as pure
return::Monadm=>a->m a
return:: a->Maybea

CHAPTER 18. MONAD 1167
There should have been nothing surprising there, so let’s
get to the meat of the matter.
Using the Maybe Monad
This example looks like the one from the Applicative chapter,
but it’s diﬀerent. We encourage you to compare the two, al-
though we’ve been explicit about what exactly is happening
here. You developed some intutions above for dosyntax and
the listMonad; here we’ll be quite explicit about what’s happen-
ing, and by the time we get to the Either demonstration below,
it should be clear. Let’s get started:
dataCow=Cow{
name::String
, age ::Int
, weight ::Int
}deriving (Eq,Show)
noEmpty ::String->MaybeString
noEmpty ""=Nothing
noEmpty str=Juststr
noNegative ::Int->MaybeInt
noNegative n|n>=0=Justn
|otherwise =Nothing

CHAPTER 18. MONAD 1168
-- if Cow's name is Bess, must be under 500
weightCheck ::Cow->MaybeCow
weightCheck c=
letw=weight c
n=name c
in ifn=="Bess"&&w>499
thenNothing
elseJustc
mkSphericalCow ::String
->Int
->Int
->MaybeCow
mkSphericalCow name' age' weight' =
casenoEmpty name' of
Nothing ->Nothing
Justnammy->
casenoNegative age' of
Nothing ->Nothing
Justagey->
casenoNegative weight' of
Nothing ->Nothing
Justweighty ->
weightCheck
(Cownammy agey weighty)

CHAPTER 18. MONAD 1169
Prelude> mkSphericalCow "Bess" 5 499
Just (Cow {name = "Bess", age = 5, weight = 499})
Prelude> mkSphericalCow "Bess" 5 500
Nothing
First, we’ll clean it up with dosyntax, then we’ll see why we
can’t do this with Applicative :
-- Do syntax isn't just for IO.
mkSphericalCow' ::String
->Int
->Int
->MaybeCow
mkSphericalCow' name' age' weight' = do
nammy<-noEmpty name'
agey<-noNegative age'
weighty <-noNegative weight'
weightCheck ( Cownammy agey weighty)
And this works as expected.
Prelude> mkSphericalCow' "Bess" 5 500
Nothing
Prelude> mkSphericalCow' "Bess" 5 499
Just (Cow {name = "Bess", age = 5, weight = 499})

CHAPTER 18. MONAD 1170
Can we write it with (>>=)? Sure!
-- Stack up the nested lambdas.
mkSphericalCow'' ::String
->Int
->Int
->MaybeCow
mkSphericalCow'' name' age' weight' =
noEmpty name' >>=
\nammy->
noNegative age' >>=
\agey->
noNegative weight' >>=
\weighty ->
weightCheck ( Cownammy agey weighty)
So why can’t we do this with Applicative ? Because our
weightCheck function depends on the prior existence of a Cow
value and returns more monadic structure in its return type
Maybe Cow .
If your dosyntax looks like this:

CHAPTER 18. MONAD 1171
doSomething = do
a<-f
b<-g
c<-h
pure (a, b, c)
You can rewrite it using Applicative . On the other hand, if
you have something like this:
doSomething' n= do
a<-f n
b<-g a
c<-h b
pure (a, b, c)
You’re going to need Monadbecause 𝑔andℎare producing
monadic structure based on values that can only be obtained
by depending on values generated from monadic structure.
You’ll need jointo crunch the nesting of monadic structure
back down. If you don’t believe us, try translating doSomething'
toApplicative : so no resorting to >>=orjoin.
Here’s some code to kick that around:
f::Integer ->MaybeInteger
f0=Nothing
fn=Justn

CHAPTER 18. MONAD 1172
g::Integer ->MaybeInteger
gi=
ifeven i
thenJust(i+1)
elseNothing
h::Integer ->MaybeString
hi=Just("10191" ++show i)
doSomething' n= do
a<-f n
b<-g a
c<-h b
pure (a, b, c)
The long and short of it:
1.With the Maybe Applicative , eachMaybecomputation fails
or succeeds independently of each other. You’re lifting
functions that are also JustorNothing overMaybevalues.
2.With the Maybe Monad , computations contributing to the
final result can choose to return Nothing based on previous
computations.

CHAPTER 18. MONAD 1173
Exploding a spherical cow
We said we’d be quite explicit about what’s happening in the
above, so let’s do this thing. Let’s get in the guts of this code
and how binding over Maybevalues works.
For once, this example instance is what’s in GHC’s base
library at time of writing:
instance MonadMaybewhere
return x =Justx
(Justx)>>=k=k x
Nothing >>= _ = Nothing
mkSphericalCow'' ::String
->Int
->Int
->MaybeCow
mkSphericalCow'' name' age' weight' =
noEmpty name' >>=
\nammy->
noNegative age' >>=
\agey->
noNegative weight' >>=
\weighty ->
weightCheck ( Cownammy agey weighty)

CHAPTER 18. MONAD 1174
And what happens if we pass it some arguments?
-- Proceeding outermost to innermost.
mkSphericalCow'' "Bess"5499=
noEmpty "Bess">>=
\nammy->
noNegative 5>>=
\agey->
noNegative 499>>=
\weighty ->
weightCheck ( Cownammy agey weighty)
-- "Bess" /= "", so skipping this pattern
-- noEmpty "" = Nothing
noEmpty "Bess"=Just"Bess"
So we produced the value Just "Bess" ; however, nammywill
be theString and not also the Maybestructure because >>=passes
𝑎to the function it binds over the monadic value, not 𝑚𝑎. Here
we’ll use the Maybe Monad instance to examine why:

CHAPTER 18. MONAD 1175
instance MonadMaybewhere
return x =Justx
(Justx)>>=k=k x
Nothing >>= _ = Nothing
noEmpty "Bess">>=\nammy->
(restofthe computation)
-- noEmpty "Bess" evaluated
-- to Just "Bess". So the first
-- Just case matches.
(Just"Bess")>>=\nammy-> ...
(Justx)>>=k=k x
-- k is \nammy et al.
-- x is "Bess" by itself.
Sonammyis bound to ”Bess”, and the following is the whole 𝑘:
\"Bess"->
noNegative 5>>=
\agey->
noNegative 499>>=
\weighty ->
weightCheck ( Cownammy agey weighty)

CHAPTER 18. MONAD 1176
Then how does the age check go?
mkSphericalCow'' "Bess"5499=
noEmpty "Bess">>=
\"Bess"->
noNegative 5>>=
\agey->
noNegative 499>>=
\weighty ->
weightCheck ( Cow"Bess"agey weighty)
-- 5 >= 0 is true, so we get Just 5
noNegative 5|5>=0=Just5
|otherwise =Nothing
Again, although noNegative returns Just 5 , thebindfunction
will pass 5 on:

CHAPTER 18. MONAD 1177
mkSphericalCow'' "Bess"5499=
noEmpty "Bess">>=
\"Bess"->
noNegative 5>>=
\5->
noNegative 499>>=
\weighty ->
weightCheck ( Cow"Bess"5weighty)
-- 499 >= 0 is true, so we get Just 499
noNegative 499|499>=0=Just499
|otherwise =Nothing
Passing 499 on:
mkSphericalCow'' "Bess"5499=
noEmpty "Bess">>=
\"Bess"->
noNegative 5>>=
\5->
noNegative 499>>=
\499->
weightCheck ( Cow"Bess"5499)

CHAPTER 18. MONAD 1178
weightCheck (Cow"Bess"5499)=
let499=weight ( Cow"Bess"5499)
"Bess"=name (Cow"Bess"5499)
-- fyi, 499 > 499 is False.
in if"Bess"=="Bess"&&499>499
thenNothing
elseJust(Cow"Bess"5499)
So in the end, we return Just (Cow "Bess" 5 499) .
Fail fast, like an overfunded startup
But what if we had failed? We’ll dissect the following compu-
tation:
Prelude> mkSphericalCow'' "" 5 499
Nothing
And how do the guts fall when we explode this poor bovine?

CHAPTER 18. MONAD 1179
mkSphericalCow'' ""5499=
noEmpty "">>=
\nammy->
noNegative 5>>=
\agey->
noNegative 499>>=
\weighty ->
weightCheck ( Cownammy agey weighty)
-- "" == "", so we get the Nothing case
noEmpty ""=Nothing
-- noEmpty str = Just str
After we’ve evaluated noEmpty "" and gotten a Nothing value,
we use(>>=). How does that go?

CHAPTER 18. MONAD 1180
instance MonadMaybewhere
return x =Justx
(Justx)>>=k=k x
Nothing >>= _ = Nothing
-- noEmpty "" := Nothing
Nothing >>=
\nammy->
-- Just case doesn't match, so skip it.
-- (Just x) >>= k = k x
-- This is what we're doing.
Nothing >>= _ = Nothing
So it turns out that the bindfunction will drop the entire
rest of the computation on the floor the moment anyof the
functions participating in the Maybe Monad actions produce a
Nothing value:
mkSphericalCow'' ""5499=
Nothing >>=-- NOPE.
In fact, you can demonstrate to yourself that that stuﬀ never
gets used with bottom , but does with a Justvalue:

CHAPTER 18. MONAD 1181
Prelude> Nothing >>= undefined
Nothing
Prelude> Just 1 >>= undefined
*** Exception: Prelude.undefined
But why do we use the Maybe Applicative andMonad? Because
this:
mkSphericalCow' ::String
->Int
->Int
->MaybeCow
mkSphericalCow' name' age' weight' = do
nammy<-noEmpty name'
agey<-noNegative age'
weighty <-noNegative weight'
weightCheck ( Cownammy agey weighty)
is a lot nicer than case matching the Nothing case over and
over just so we can say Nothing -> Nothing a million times. Life
is too short for repetition when computers lovetaking care of
repetition.
Either
Whew. Let’s all be thankful that cow was full of Maybevalues
and not tripe. Moving along, we’re going to demonstrate use

CHAPTER 18. MONAD 1182
of theEither Monad , step back a bit, and let your intuitions and
what you learned about Maybeguide you through.
Specializing the types
As always, we present the types:
-- m ~ Either e
(>>=)::Monadm
=> m a
->(a-> m b)
-> m b
(>>=)::Eithere a
->(a->Eithere b)
->Eithere b
-- same as pure
return::Monadm=>a-> m aq
return:: a->Eithere a
Why do we keep doing this? To remind you that the types
always show you the way, once you’ve figured them out.
Using the Either Monad
Use what you know to go carefully through this code and
follow the types. First, we define our datatypes:

CHAPTER 18. MONAD 1183
moduleEitherMonad where
-- years ago
typeFounded =Int
-- number of programmers
typeCoders=Int
dataSoftwareShop =
Shop{
founded ::Founded
, programmers ::Coders
}deriving (Eq,Show)
dataFoundedError =
NegativeYears Founded
|TooManyYears Founded
|NegativeCoders Coders
|TooManyCoders Coders
|TooManyCodersForYears Founded Coders
deriving (Eq,Show)
Let’s bring some functions now:

CHAPTER 18. MONAD 1184
validateFounded
::Int
->EitherFoundedError Founded
validateFounded n
|n<0=Left$NegativeYears n
|n>500=Left$TooManyYears n
|otherwise =Rightn
-- Tho, many programmers *are* negative.
validateCoders
::Int
->EitherFoundedError Coders
validateCoders n
|n<0=Left$NegativeCoders n
|n>5000=Left$TooManyCoders n
|otherwise =Rightn

CHAPTER 18. MONAD 1185
mkSoftware
::Int
->Int
->EitherFoundedError SoftwareShop
mkSoftware years coders = do
founded <-validateFounded years
programmers <-validateCoders coders
ifprogrammers >div founded 10
thenLeft$
TooManyCodersForYears
founded programmers
elseRight$Shopfounded programmers
Note that Either always short-circuits on the firstthing to
have failed. It mustbecause in the Monad, later values can depend
on previous ones:
Prelude> mkSoftware 0 0
Right (Shop {founded = 0, programmers = 0})
Prelude> mkSoftware (-1) 0
Left (NegativeYears (-1))
Prelude> mkSoftware (-1) (-1)
Left (NegativeYears (-1))
Prelude> mkSoftware 0 (-1)

CHAPTER 18. MONAD 1186
Left (NegativeCoders (-1))
Prelude> mkSoftware 500 0
Right (Shop {founded = 500, programmers = 0})
Prelude> mkSoftware 501 0
Left (TooManyYears 501)
Prelude> mkSoftware 501 501
Left (TooManyYears 501)
Prelude> mkSoftware 100 5001
Left (TooManyCoders 5001)
Prelude> mkSoftware 0 500
Left (TooManyCodersForYears 0 500)
So, there is no MonadforValidation .Applicative andMonadin-
stances must have the same behavior. This is usually expressed
in the form:
importControl.Monad (ap)
(<*>)==ap
This is a way of saying the Applicative apply for a type must
not change behavior if derived from the Monadinstance’s bind

CHAPTER 18. MONAD 1187
operation.
-- Keeping in mind
(<*>)::Applicative f
=>f (a->b)->f a->f b
ap::Monadm
=>m (a->b)->m a->m b
Then deriving Applicative (<*>) from the stronger instance:
ap::(Monadm)=>m (a->b)->m a->m b
apm m'= do
x<-m
x'<-m'
return (x x')
The problem is you can’t make a MonadforValidation that
accumulates the errors like the Applicative does. Instead, any
Monadinstance for Validation would be identical to Either ’sMonad
instance.
Short Exercise: Either Monad
Implement the Either Monad .

CHAPTER 18. MONAD 1188
dataSuma b=
Firsta
|Secondb
deriving (Eq,Show)
instance Functor (Suma)where
fmap=undefined
instance Applicative (Suma)where
pure=undefined
(<*>)=undefined
instance Monad(Suma)where
return=pure
(>>=)=undefined
18.5 Monad laws
TheMonadtypeclass has laws, as the other typeclasses do. These
laws exist, as with all the other typeclass laws, to ensure that
your code does nothing surprising or harmful. If the Monad
instance you write for your type abides by these laws, then
your monads should work as you want them to. To write your
own instance, you only have to define a >>=operation, but you
want your binding to be as predictable as possible.

CHAPTER 18. MONAD 1189
Identity laws
Monadhas two identity laws:
-- right identity
m>>=return =m
-- left identity
returnx>>=f=f x
Basically both of these laws are saying that return should be
neutral and not perform any computation. We’ll line them up
with the type of >>=to clarify what’s happening:
(>>=)::Monadm
=>m a->(a->m b)->m b
-- [1] [2] [3]
First, right identity:
return::a->m a
m>>=return =m
-- [1] [2] [3]
The𝑚does represent an m aandm b, respectively, so the
structure is there even if it’s not apparent from the way the
law is written.

CHAPTER 18. MONAD 1190
And left identity:
-- applying return to x gives us an
-- m a value to start
return x >>=f=f x
-- [1] [2] [3]
Likepure,return shouldn’t change any of the behavior of the
rest of the function; it is only there to put things into structure
when we need to, and the existence of the structure should
not aﬀect the computation.
Associativity
The law of associativity is not so diﬀerent from other laws of
associativity we have seen. It does look a bit diﬀerent because
of the nature of >>=:
(m>>=f)>>=g=m>>=(\x->f x>>=g)
Regrouping the functions should not have any impact on
the final result, same as the associativity of Monoid . The syntax
there, in which, for the right side of the equals sign, we had to
pass in an 𝑥argument might seem confusing at first. So, let’s
look at it more carefully.
This side looks the way we expect it to:

CHAPTER 18. MONAD 1191
(m>>=f)>>=g
But remember that (>>=)allows the result value of one func-
tion to be passed as input to the next, like function application
but with our value at the left and successive functions proceed-
ing to the right. Remember this code?
getLine >>=putStrLn
The IO action for getLine is evaluated first, then putStrLn is
passed the input string that resulted from running getLine ’s
eﬀects. This left-to-right is partly down to the history of IOin
Haskell — it’s so the “order” of the code reads top to bottom.
We’ll explain this more later in the book.
When we reassociate them, we need to apply 𝑓so that𝑔has
an input value of type m ato start the whole thing oﬀ. So, we
pass in the argument 𝑥via an anonymous function:
m>>=(\x->f x>>=g)
And bada bing, now nothing can slow this roll.
We’re doing that thing again
Out of mercy, we’ll be using checkers (not Nixon’s dog) again.
The argument the Monad TestBatch wants is identical to the
Applicative , a tuple of three value types embedded in the struc-
tural type.

CHAPTER 18. MONAD 1192
Prelude> quickBatch (monad [(1, 2, 3)])
monad laws:
left identity: +++ OK, passed 500 tests.
right identity: +++ OK, passed 500 tests.
associativity: +++ OK, passed 500 tests.
Going forward we’ll be using this to validate Monadinstances.
Let’s write a bad Monadto see what it can catch for us.
BadMonads and their denizens
We’re going to write an invalid Monad(andFunctor ). You could
pretend it’s Identity with an integer thrown in which gets in-
cremented on each fmapor bind.
moduleBadMonad where
importTest.QuickCheck
importTest.QuickCheck.Checkers
importTest.QuickCheck.Classes

CHAPTER 18. MONAD 1193
dataCountMe a=
CountMe Integer a
deriving (Eq,Show)
instance Functor CountMe where
fmap f ( CountMe i a)=
CountMe (i+1) (f a)
instance Applicative CountMe where
pure=CountMe 0
CountMe n f<*>CountMe n' a=
CountMe (n+n') (f a)
instance MonadCountMe where
return=pure
CountMe n a>>=f=
letCountMe _b=f a
inCountMe (n+1) b

CHAPTER 18. MONAD 1194
instance Arbitrary a
=>Arbitrary (CountMe a)where
arbitrary =
CountMe <$>arbitrary <*>arbitrary
instance Eqa=>EqProp(CountMe a)where
(=-=)=eq
main= do
lettrigger ::CountMe (Int,String,Int)
trigger =undefined
quickBatch $functor trigger
quickBatch $applicative trigger
quickBatch $monad trigger
When we run the tests, the Functor andMonadwill fail top
to bottom. The Applicative technically only failed the laws
because Functor did; in the Applicative instance we were using
a proper monoid-of-structure.
Prelude> main
functor:
identity: *** Failed! Falsifiable (after 1 test):
CountMe 0 0
compose: *** Failed! Falsifiable (after 1 test):

CHAPTER 18. MONAD 1195
<function>
<function>
CountMe 0 0
applicative:
identity: +++ OK, passed 500 tests.
composition: +++ OK, passed 500 tests.
homomorphism: +++ OK, passed 500 tests.
interchange: +++ OK, passed 500 tests.
functor: *** Failed! Falsifiable (after 1 test):
<function>
CountMe 0 0
monad laws:
left identity: *** Failed! Falsifiable (after 1 test):
<function>
0
right identity: *** Failed! Falsifiable (after 1 test):
CountMe 0 0
associativity: *** Failed! Falsifiable (after 1 test):
CountMe 0 0
We can reapply the weird, broken increment semantics and
get a broken Applicative as well.

CHAPTER 18. MONAD 1196
instance Applicative CountMe where
pure=CountMe 0
CountMe n f<*>CountMe _a=
CountMe (n+1) (f a)
Now it’s allbroken.
applicative:
identity:
*** Failed! Falsifiable (after 1 test):
CountMe 0 0
composition:
*** Failed! Falsifiable (after 1 test):
CountMe 0 <function>
CountMe 0 <function>
CountMe 0 0
homomorphism:
*** Failed! Falsifiable (after 1 test):
<function>
0
interchange:
*** Failed! Falsifiable (after 3 tests):
CountMe (-1) <function>
0
Understanding what makes sense structurally for a Functor ,
Applicative , andMonoid can tell you what is potentially an in-

CHAPTER 18. MONAD 1197
valid instance before you’ve written any code. Incidentally,
even if you fix the Functor andApplicative instances, the Monad
instance is not yet fixed.
instance Functor CountMe where
fmap f ( CountMe i a)=CountMe i (f a)
instance Applicative CountMe where
pure=CountMe 0
CountMe n f<*>CountMe n' a=
CountMe (n+n') (f a)
instance MonadCountMe where
return=pure
CountMe _a>>=f=f a
This’ll pass as a valid Functor andApplicative , but it’s not a
validMonad. The problem is that while puresetting the integer
value to zero is fine for the purposes of the Applicative , but it
violates the right identity law of Monad.
Prelude> CountMe 2 "blah" >>= return
CountMe 0 "blah"
So ourpureis too opinionated. Still a valid Applicative and
Functor , but what if puredidn’t agree with the Monoid of the

CHAPTER 18. MONAD 1198
structure? The following will pass the Functor laws but it isn’t a
validApplicative .
instance Functor CountMe where
fmap f ( CountMe i a)=CountMe i (f a)
instance Applicative CountMe where
pure=CountMe 1
CountMe n f<*>CountMe n' a=
CountMe (n+n') (f a)
As it happens, if we change the monoid-of-structure to
match the identity such that we have addition and the number
zero, it’s a valid Applicative again.
instance Applicative CountMe where
pure=CountMe 0
CountMe n f<*>CountMe n' a=
CountMe (n+n') (f a)
As you gain experience with these structures, you’ll learn to
identify what might have a valid Applicative but no valid Monad
instance. But how do we fix the Monadinstance? By fixing the
underlying Monoid !

CHAPTER 18. MONAD 1199
instance MonadCountMe where
return=pure
CountMe n a>>=f=
letCountMe n' b=f a
inCountMe (n+n') b
Once our Monadinstance starts summing the counts like the
Applicative did, it works fine! It can be easy at times to acciden-
tally write an invalid Monadthat typechecks, so it’s important to
useQuickCheck to validate your Monoid ,Functor ,Applicative , and
Monadinstances.
18.6 Application and composition
What we’ve seen so far has been primarily about function
application. We probably weren’t thinking too much about the
relationship between function application and composition
because with Functor andApplicative it hadn’t mattered much.
Both concerned functions that looked like the usual (a -> b)
arrangement, so composition “just worked” and that this was
true was guaranteed by the laws of those typeclasses:

CHAPTER 18. MONAD 1200
fmapid=id
-- guarantees
fmapf.fmap g=fmap (f .g)
Which means composition under functors just works:
Prelude> fmap ((+1) . (+2)) [1..5]
[4,5,6,7,8]
Prelude> fmap (+1) . fmap (+2) $ [1..5]
[4,5,6,7,8]
WithMonadthe situation seems less neat at first. Let’s attempt
to define composition for monadic functions in a simple way:
mcomp::Monadm=>
(b->m c)
->(a->m b)
->a->m c
mcompf g a=f (g a)
If we try to load this, we’ll get an error like this:
Couldn't match expected type ‘b’
with actual type ‘m b’
‘b’ is a rigid type variable bound

CHAPTER 18. MONAD 1201
by the type signature for
mcomp :: Monad m =>
(b -> m c)
-> (a -> m b)
-> a -> m c
at kleisli.hs:21:9
Relevant bindings include
g :: a -> m b (bound at kleisli.hs:22:8)
f :: b -> m c (bound at kleisli.hs:22:6)
mcomp :: (b -> m c)
-> (a -> m b)
-> a -> m c
(bound at kleisli.hs:22:1)
In the first argument of ‘f’, namely ‘(g a)’
In the expression: f (g a)
Failed, modules loaded: none.
Well, that didn’t work. That error message is telling us that
𝑓is expecting a 𝑏for its first argument, but 𝑔is passing an m b
to𝑓. So, how do we apply a function in the presence of some
context that we want to ignore? We use fmap. That’s going to
give us an m (m c) instead of an m c, so we’ll want to jointhose
two monadic structures.

CHAPTER 18. MONAD 1202
mcomp::Monadm=>
(b->m c)
->(a->m b)
->a->m c
mcompf g a=join (f <$>(g a))
But using joinandfmaptogether means we can go ahead
and use (>>=).
mcomp'' ::Monadm=>
(b->m c)
->(a->m b)
->a->m c
mcomp'' f g a=g a>>=f
You don’t need to write anything special to make monadic
functions compose (as long as the monadic contexts are the
sameMonad) because Haskell has it covered: what you want is
Kleisli composition . Don’t sweat the strange name; it’s not as
weird as it sounds. As we saw above, what we need is function
composition written in terms of >>=to allow us to deal with
the extra structure, and that’s what the Kleisli fish gives us.
Let’s remind ourselves of the types of ordinary function
composition and >>=:

CHAPTER 18. MONAD 1203
(.)::(b->c)->(a->b)->a->c
(>>=)::Monadm
=>m a->(a->m b)->m b
To get Kleisli composition oﬀ the ground, we have to flip
some arguments around to make the types work:
importControl.Monad
-- the order is flipped to match >>=
(>=>)
::Monadm
=>(a->m b)->(b->m c)->a->m c
See any similarities to something you know yet?
(>=>)
::Monadm
=>(a->m b)->(b->m c)->a->m c
flip(.)
::(a->b)->(b->c)->a->c
It’s function composition with monadic structure hanging
oﬀ the functions we’re composing. Let’s see an example!

CHAPTER 18. MONAD 1204
importControl.Monad ((>=>))
sayHi::String->IOString
sayHigreeting = do
putStrLn greeting
getLine
readM::Reada=>String->IOa
readM=return.read
getAge::String->IOInt
getAge=sayHi>=>readM
askForAge ::IOInt
askForAge =
getAge"Hello! How old are you? "
We used return composed with readto turn it into some-
thing that provides monadic structure after being bound over
the output of sayHi. We needed the Kleisli composition opera-
tor to stitch sayHiandreadMtogether:

CHAPTER 18. MONAD 1205
sayHi::String->IOString
readM::Reada=>String->IOa
-- [1] [2] [3]
(a->m b)
String->IOString
-- [4] [5] [6]
->(b->m c)
String->IOa
-- [7] [8] [9]
->a->m c
String IOa
1.The first type is the type of the input to sayHi,String .
2.TheIOthatsayHiperforms in order to present a greeting
and receive input.
3.TheString input from the user that sayHireturns.
4.TheString thatreadMexpects as an argument and which
sayHiwill produce.
5.TheIO readM returns into. Note that return/pure produce
IOvalues which perform no I/O.

CHAPTER 18. MONAD 1206
6.TheIntthatreadMreturns.
7.The original, initial String inputsayHiexpects so it knows
how to greet the user and ask for their age.
8.The final combined IOaction which performs all eﬀects
necessary to produce the final result.
9.The value inside of the final IOaction; in this case, this is
theIntvalue that readMreturned.
18.7 Chapter Exercises
WriteMonadinstancesforthefollowingtypes. Usethe QuickCheck
properties we showed you to validate your instances.
1.Welcome to the Nope Monad , where nothing happens and
nobody cares.
dataNopea=
NopeDotJpg
-- We're serious. Write it anyway.
2.dataPhhhbbtttEither b a=
Lefta
|Rightb

CHAPTER 18. MONAD 1207
3.Write a Monadinstance for Identity .
newtype Identity a=Identity a
deriving (Eq,Ord,Show)
instance Functor Identity where
fmap=undefined
instance Applicative Identity where
pure=undefined
(<*>)=undefined
instance MonadIdentity where
return=pure
(>>=)=undefined
4.This one should be easier than the Applicative instance
was. Remember to use the Functor thatMonadrequires, then
see where the chips fall.
dataLista=
Nil
|Consa (Lista)
Write the following functions using the methods provided
byMonadandFunctor . Using stuﬀ like identity and composition
is fine, but it has to typecheck with types provided.

CHAPTER 18. MONAD 1208
1.j::Monadm=>m (m a) ->m a
Expecting the following behavior:
Prelude> j [[1, 2], [], [3]]
[1,2,3]
Prelude> j (Just (Just 1))
Just 1
Prelude> j (Just Nothing)
Nothing
Prelude> j Nothing
Nothing
2.l1::Monadm=>(a->b)->m a->m b
3.l2::Monadm
=>(a->b->c)->m a->m b->m c
4.a::Monadm=>m a->m (a->b)->m b
5.You’ll need recursion for this one.
meh::Monadm
=>[a]->(a->m b)->m [b]
6.Hint: reuse “meh”
flipType ::(Monadm)=>[m a]->m [a]

CHAPTER 18. MONAD 1209
18.8 Definition
1.Monad is a typeclass reifying an abstraction that is com-
monly used in Haskell. Instead of an ordinary function of
type𝑎to𝑏, you’re functorially applying a function which
produces more structure itself and using jointo reduce
the nested structure that results.
fmap::(a->b)->f a->f b
(<*>)::f (a->b)->f a->f b
(=<<)::(a->f b)->f a->f b
2.Amonadic function is one which generates more structure
after having been lifted over monadic structure. Contrast
the function arguments to fmapand(>>=)in:
fmap::(a->b)->f a->f b
(>>=)::m a->(a->m b)->m b
The significant diﬀerence is that the result is m band re-
quiresjoining the result after lifting the function over 𝑚.
What does this mean? That depends on the Monadinstance.
The distinction can be seen with ordinary function com-
position and Kleisli composition as well:

CHAPTER 18. MONAD 1210
(.)
::(b->c)->(a->b)->a->c
(>=>)
::Monadm
=>(a->m b)->(b->m c)->a->m c
3.bindis unfortunately a somewhat overloaded term. You
first saw it used early in the book with respect to binding
variables to values, such as with the following:
letx=2inx+2
Where 𝑥is a variable bound to 2. However, when we’re
talking about a Monadinstance typically bind will refer
to having used >>=to lift a monadic function over the
structure. The distinction being:
-- lifting (a -> b) over f in f a
fmap::(a->b)->f a->f b
-- binding (a -> m b) over m in m a
(>>=)::m a->(a->m b)->m b
You’ll sometimes see us talk about the use of the bind
do-notation <-or(>>=)as “binding over.” When we do, we

CHAPTER 18. MONAD 1211
mean that we lifted a monadic function and we’ll even-
tuallyjoinor smush the structure back down when we’re
done monkeying around in the Monad.Don’tpanic if we’re a
little casual about describing the use of <-as having bound
over/out some 𝑎out ofm a.
18.9 Follow-up resources
1.What a Monad is not
https://wiki.haskell.org/What_a_Monad_is_not
2.Gabriel Gonzalez; How to desugar Haskell code
3.Stephen Diehl; What I wish I knew when Learning Haskell
http://dev.stephendiehl.com/hask/#monads
4.Stephen Diehl; Monads Made Difficult
http://www.stephendiehl.com/posts/monads.html
5.Brent Yorgey; Typeclassopedia
https://wiki.haskell.org/Typeclassopedia

Chapter 19
Applying structure
I often repeat repeat
myself, I often repeat
repeat. I don’t don’t know
why know why, I simply
know that I I I am am
inclined to say to say a lot
a lot this way this way- I
often repeat repeat
myself, I often repeat
repeat.
Jack Prelutsky
1212

CHAPTER 19. MONADS GONE WILD 1213
19.1 Applied structure
We thought you’d like to see Monoid,Functor ,Applicative , and
Monadin the wild as it were. Since we’d like to finish this book
before we have grandchildren, this will notbe accompanied
by the painstaking explanations and exercise regime you’ve
experienced up to this point. Don’t understand something?
Figure it out! We’ll do our best to leave a trail of breadcrumbs
for you to follow up on the code we show you. Consider this a
breezy survey of how Haskellers write code when they think
no one is looking and a pleasant break from your regularly
scheduled exercises. The code demonstrated will not always
include all necessary context to make it run, so don’t expect
to be able to load the snippets in GHCi and have them work.
If you don’t have a lot of previous programming experience
and some of the applications are difficult for you to follow,
you might prefer to return to this chapter at a later time, once
you start trying to read and use Haskell libraries for practical
projects.
19.2Monoid
Monoids are everywhere once you recognize the pattern and
start looking for them, but we’ve tried to choose a few good
examples to illustrate typical use-cases.

CHAPTER 19. MONADS GONE WILD 1214
Templating content in Scotty
Here the scotty web framework’s “Hello, World” example uses
mconcat to inject the parameter “word” into the HTML page
returned:
{-# LANGUAGE OverloadedStrings #-}
importWeb.Scotty
importData.Monoid (mconcat)
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
html
(mconcat
["<h1>Scotty, "
, beam
," me up!</h1>" ])
If you’re interested in following up on this example, you
can find this example and a tutorial on the scotty Github repos-
itory.

CHAPTER 19. MONADS GONE WILD 1215
Concatenating connection parameters
The next example is from Aditya Bhargava’s “Making A Web-
site With Haskell,” a blog post that walks you through several
steps for, well, making a simple website in Haskell. It also uses
thescotty web framework.
Here we’re using foldrandMonoid to concatenate connection
parameters for connecting to the database:
runDb::SqlPersist (ResourceT IO) a
->IOa
runDbquery= do
letconnStr =
foldr (\(k,v) t ->
t<>(encodeUtf8 $
k<>"="<>v<>" "))
""params
runResourceT
.withPostgresqlConn connStr
$runSqlConn query
If you’re interested in following up on this, this blog post
is one of many that shows you step by step how to use scotty ,
although many of them breeze through each step without a
great deal of explanation. It will be easier to understand scotty
in detail once you’ve worked through monad transformers, but

CHAPTER 19. MONADS GONE WILD 1216
if you’d like to start playing around with some basic projects,
you may want to try them out.
Concatenating key configurations
The next example is going to be a bit meatier than the two
previous ones.
xmonad is a windowing system for X11 written in Haskell.
The configuration language is Haskell — the binary that runs
your WM is compiled from your personal configuration. The
following is an example of using mappend to combine the default
configuration’s key mappings and a modification of those keys:

CHAPTER 19. MONADS GONE WILD 1217
importXMonad
importXMonad.Actions.Volume
importData.Map.Lazy (fromList )
importData.Monoid (mappend)
main= do
xmonad def { keys =
\c->fromList [
((0, xK_F6),
lowerVolume 4>>return()),
((0, xK_F7),
raiseVolume 4>>return())
] `mappend` keys defaultConfig c
}
The type of keysis a function:
keys:: !(XConfig Layout
->Map(ButtonMask ,KeySym) (X()))
You don’t need to get too excited about the exclamation
point right now; it’s the syntax for a nifty thing called a strictness
annotation , which makes a field in a product strict. That is, you
won’t be able to construct the record or product that contains
the value without also forcing that field to weak head normal
form. We’ll explain this in more detail later in the book.

CHAPTER 19. MONADS GONE WILD 1218
The gist of the mainabove is that it allows your keymapping
to be based on the current configuration of your environment.
Whenever you type a key, xmonad will pass the current config to
yourkeysfunction in order to determine what (if any) action it
should take based on that. We’re using the Monoid here to add
new keyboard shortcuts for lowering and raising the volume
with F6 and F7. The monoid of the keysfunctions is combining
all of the key maps each function produces when applied to
theXConfig to produce a final canonical key map.
Say what?
This is a Monoid instance we hadn’t covered in the Monoid
chapter, so let’s take a look at it now:
instance Monoidb
=>Monoid(a->b)
-- Defined in ‘GHC.Base’
This, friends, is the Monoid of functions.
But how does it work? First, let’s set up some very trivial
functions for demonstration:
Prelude> import Data.Monoid
Prelude> let f = const (Sum 1)
Prelude> let g = const (Sum 2)
Prelude> f 9001
Sum {getSum = 1}
Prelude> g 9001

CHAPTER 19. MONADS GONE WILD 1219
Sum {getSum = 2}
Query the types of those functions and see how you think
they will match up to the Monoid instance above.
We know that whatever arguments we give to 𝑓and𝑔, they
will always return their first arguments, which are Summonoids.
So if we mappend 𝑓and𝑔, they’re going to ignore whatever argu-
ment we tried to apply them to and use the Monoid to combine
the results:
Prelude> (f <> g) 9001
Sum {getSum = 3}
So this Monoid instance allows to mappend the results of two
function applications:
(a->b)<>(a->b)
Just as long as the 𝑏has aMonoid instance.
We’re going to oﬀer a few more examples that will get you
closer to what the particular use of mappend in thexmonad ex-
ample is doing. We mentioned Data.Map back in the Testing
chapter. It gives us ordered pairs of keys and values:
Prelude> import qualified Data.Map as M
Prelude M> :t M.fromList
M.fromList :: Ord k => [(k, a)] -> Map k a

CHAPTER 19. MONADS GONE WILD 1220
Prelude M> let f = M.fromList [('a', 1)]
Prelude M> let g = M.fromList [('b', 2)]
Prelude M> :t f
f :: Num a => Map Char a
Prelude M> import Data.Monoid
Prelude M Data.Monoid> f <> g
fromList [('a',1),('b',2)]
Prelude M Data.Monoid> :t (f <> g)
(f <> g) :: Num a => Map Char a
Prelude M Data.Monoid> mappend f g
fromList [('a',1),('b',2)]
Prelude M Data.Monoid> f `mappend` g
fromList [('a',1),('b',2)]
-- but note what happens here:
Prelude> f <> g
fromList [('a',1)]
So, returning to the xmonad configuration we started with.
Thekeysfield is a function which, given an XConfig , produces a
keymapping. It uses the monoid of functions to combine the
pre-existing function that generates the keymap to produce as
many maps as you have mappended functions, then combine
all the key maps into one.
This part:

CHAPTER 19. MONADS GONE WILD 1221
>>return()
says that the key assignment is performing some eﬀects and
only performing some eﬀects. Functions have to reduce to
some result, but sometimes their only purpose is to perform
someeﬀectsandyoudon’twanttodoanythingwiththe“result”
of evaluating the terms.
As we’ve said and other people have noted as well, monoids
areeverywhere — not just in Haskell but in all of programming.
19.3 Functor
There’s a reason we chose that Michael Neale quotation for
theFunctor chapter epigraph: lifting really is the cheat mode.
fmapis ubiquitous in Haskell, for all sorts of applications, but
we’ve picked a couple that we found especially demonstrative
of why it’s so handy.
Lifting over IO
Herewe’retakingafunctionthatdoesn’tperformI/O, addUTCTime ,
partially applying it to the oﬀset we’re going to add to the sec-
ond argument, then mapping it over the IOaction that gets us
the current time:

CHAPTER 19. MONADS GONE WILD 1222
importData.Time.Clock
offsetCurrentTime ::NominalDiffTime
->IOUTCTime
offsetCurrentTime offset=
fmap (addUTCTime (offset *24*3600))$
getCurrentTime
Context for the above:
1.NominalDiffTime is a newtype of Picoand has a Numinstance,
that’s why the arithmetic works.
addUTCTime ::NominalDiffTime
->UTCTime
->UTCTime
2.getCurrentTime ::IOUTCTime
3.fmap’s type got specialized.
fmap::(UTCTime ->UTCTime)
->IOUTCTime
->IOUTCTime
Here we’re lifting some data conversion stuﬀ over the fact
that the UUID library has to touch an outside resource (ran-
dom number generation) to give us a random identifier. The

CHAPTER 19. MONADS GONE WILD 1223
UUID library used is named uuidon Hackage. The Textpack-
age used is named… text:
import Data.Text (Text)
import qualified Data.Text asT
import qualified Data.UUID asUUID
import qualified Data.UUID.V4 asUUIDv4
textUuid ::IOText
textUuid =
fmap (T.pack.UUID.toString)
UUIDv4.nextRandom
1.nextRandom ::IOUUID
2.toString ::UUID->String
3.pack::String->Text
4.fmap::(UUID->Text)
->IOUUID
->IOText
Lifting over web app monads
Frequently when you write web applications, you’ll have a
custom datatype to describe the web application which is also

CHAPTER 19. MONADS GONE WILD 1224
aMonad. It’s a Monadbecause your “app context” will have a
type parameter to describe what result was produced in the
course of a running web application. Often these types will
abstract out the availability of a request or other configuration
data with a Reader (explained in a later chapter), as well as the
performance of eﬀects via IO. In the following example, we’re
lifting over AppHandler andMaybe:
userAgent ::AppHandler (MaybeUserAgent )
userAgent =
(fmap.fmap) userAgent' getRequest
userAgent' ::Request ->MaybeUserAgent
userAgent' req=
getHeader "User-Agent" req
We need the Functor here because while we can pattern
match on the Maybevalue, an AppHandler isn’t something we can
pattern match on. It’s a convention in this web framework
library, snap, to make a type alias for your web application type.
It usually looks like this:
typeAppHandler =Handler AppApp
The underlying infrastructure for snapis more complicated
than we can cover to any depth here, but suffice to say there
are a few things floating around:

CHAPTER 19. MONADS GONE WILD 1225
1.HTTP request which triggered the processing currently
occurring.
2.The current (possibly empty or default) response that will
be returned to the client when the handlers and middle-
ware are done.
3.A function for updating the request timeout.
4.A helper function for logging.
5.And a fair bit more than this.
The issue here is that your AppHandler is meant to be slotted
into a web application which requires the reading in of con-
figuration, initialization of a web server, and the sending of a
request to get everything in motion. This is essentially a bunch
of functions waiting for arguments — waiting for something
to do. It doesn’t make sense to do all that yourself every time
you want a value that can only be obtained in the course of
the web application doing its thing. Accordingly, our Functor
is letting us write functions over structure which handles all
this work. It’s like we’re saying, “here’s a function, apply it to a
thing that resulted from an HTTP request coming down the
pipe, if one comes along.”

CHAPTER 19. MONADS GONE WILD 1226
19.4Applicative
Applicative is somewhat new to Haskell, but it’s useful enough,
particularlywithparsers, thatit’seasytofindexamples. There’s
a whole chapter on parsers coming up later, but we thought
these examples were mostly comprehensible even without
that context.
hgrev
This is an example from Luke Hoersten’s hgrevproject. The
example in the README is a bit dense, but uses Monoid and
Applicative to combine parsers of command line arguments:
jsonSwitch ::Parser(a->a)
jsonSwitch =
infoOption $(hgRevStateTH jsonFormat)
$long"json"
<>short'J'
<>help
"Display JSON version information"
parserInfo ::ParserInfo (a->a)
parserInfo =
info (helper <*>verSwitch <*jsonSwitch)
fullDesc

CHAPTER 19. MONADS GONE WILD 1227
You might be wondering what the <*operator is. It’s an-
other operator from the Applicative typeclass. It allows you to
sequence actions, discarding the result of the second argument.
Does this look familiar?
Prelude> :t (<*)
(<*) :: Applicative f => f a -> f b -> f a
Prelude> :t const
const :: a -> b -> a
Basically the (<*)operator (like its sibling, (*>), and the
monadic operator, >>) is useful when you’re emitting eﬀects.
In this case, you’ve done something with eﬀects and want to
discard any value that resulted.
More parsing
Here we’re using Applicative to lift the data constructor for the
Payload type over the Parser returned by requesting a value by
key out of a JSON object, which is basically an association of
text keys to further more JSON values which may be strings,
numbers, arrays, or more JSON objects:

CHAPTER 19. MONADS GONE WILD 1228
parseJSON ::Value->Parsera
(.:)::FromJSON a
=>Object
->Text
->Parsera
instance FromJSON Payload where
parseJSON ( Objectv)=
Payload <$>v.:"from"
<*>v.:"to"
<*>v.:"subject"
<*>v.:"body"
<*>v.:"offset_seconds"
parseJSON v =typeMismatch "Payload" v
This is the same as the JSON but for CSV1data:
parseRecord ::Record->Parsera
1CSV stands for comma-separated values, a common, though not entirely standard-
ized file format.

CHAPTER 19. MONADS GONE WILD 1229
instance FromRecord Release where
parseRecord v
|V.length v ==5=Release <$>v.!0
<*>v.!1
<*>v.!2
<*>v.!3
<*>v.!4
|otherwise =mzero
This one uses liftA2 to lift the tuple data constructor over
parseKey andparseValue to give key-value pairings. You can see
the(<*)operator in there again as well, along with the infix
operator for fmapand=<<as well:

CHAPTER 19. MONADS GONE WILD 1230
instance Deserializeable ShowInfoResp where
parser=
e2err=<<convertPairs
.HM.fromList <$>parsePairs
where
parsePairs ::Parser[(Text,Text)]
parsePairs =
parsePair `sepBy` endOfLine
parsePair =
liftA2 (,) parseKey parseValue
parseKey =
takeTill ( ==':')<*kvSep
kvSep=string": "
parseValue =takeTill isEndOfLine
This one instance is a virtual cornucopia of applications
of the previous chapters and we believe it demonstrates how
much cleaner and more readable these can make your code.
And now for something diﬀerent
This next example is also using an applicative, but this is a bit
diﬀerent than the above examples. We’ll spend more time

CHAPTER 19. MONADS GONE WILD 1231
explaining this one, as this pattern for writing utility functions
is common:
moduleWeb.Shipping.Utils ((<||>))where
importControl.Applicative (liftA2)
(<||>)::(a->Bool)
->(a->Bool)
->a
->Bool
(<||>)=liftA2 ( ||)
At first glance, this doesn’t seem too hard to understand,
but some examples will help you develop an understanding
of what’s going on. We start with the operator for boolean
disjunction, (||), which is an or:
Prelude> True || False
True
Prelude> False || False
False
Prelude> (2 > 3) || (3 == 3)
True
And now we want to be able to keep that as an infix operator
but lift it over some context, so we use liftA2 :

CHAPTER 19. MONADS GONE WILD 1232
Prelude> import Control.Applicative
Prelude> let (<||>) = liftA2 (||)
Andwe’llmakesometrivialfunctionsagainforthepurposes
of demonstration:
Prelude> let f 9001 = True; f _ = False
Prelude> let g 42 = True; g _ = False
Prelude> :t f
f :: (Eq a, Num a) => a -> Bool
Prelude> f 42
False
Prelude> f 9001
True
Prelude> g 42
True
Prelude> g 9001
False
We can compose the two functions 𝑓and𝑔to take one input
and give one summary result like this:
Prelude> (\n -> f n || g n) 0
False
Prelude> (\n -> f n || g n) 9001
True
Prelude> :t (\n -> f n || g n)

CHAPTER 19. MONADS GONE WILD 1233
(\n -> f n || g n)
:: (Eq a, Num a) => a -> Bool
But we have to pass in that argument 𝑛in order to do it that
way. Our utility function gives us a cleaner way:
Prelude> (f <||> g) 0
False
Prelude> (f <||> g) 9001
True
It’s parallel application of the functions against an argument.
That application produces two values, so we monoidally com-
bine the two values so that we have a single value to return.
We’ve set up an environment so that two (a -> Bool) functions
that don’t have an 𝑎argument yet can return a result based on
those two Boolvalues when the combined function is eventu-
ally applied against an 𝑎.
19.5Monad
BecauseeﬀectfulprogrammingisconstrainedinHaskellthrough
the use of IO, andIOhas an instance of Monad, examples of Monad
in practical Haskell code are everywhere. We tried to find
some examples that illustrate diﬀerent interesting use cases.

CHAPTER 19. MONADS GONE WILD 1234
Opening a network socket
Here we’re using dosyntax for IO’sMonadin order to bind a
socket handle from the socket smart constructor, connect it
to an address, then return the handle for reading and writing.
This example is from haproxy-haskell by Michael Xavier. See
thenetwork library on Hackage for use and documentation:
importNetwork.Socket
openSocket ::FilePath ->IOSocket
openSocket p= do
sock<-socketAF_UNIX
Stream
defaultProtocol
connect sock sockAddr
return sock
wheresockAddr =
SockAddrUnix .encodeString $p
This isn’t too unlike anything you saw in previous chapters,
at least since we built the hangman game. The next example
is a bit richer.

CHAPTER 19. MONADS GONE WILD 1235
Binding over failure in initialization
Michael Xavier’s Seraph is a process monitor and has a main
entry point which is typical of more developed libraries and
applications. The outermost MonadisIO, but the monad trans-
former variant of Either , calledEitherT , is used to bind over the
possibility of failure in constructing an initialization function.
This possibility of failure centers on being able to pull up a
correct configuration:

CHAPTER 19. MONADS GONE WILD 1236
main::IO()
main= do
initAndFp <-runEitherT $ do
fp<-tryHead NoConfig =<<lift getArgs
initCfg <-load' fp
return (initCfg, fp)
either bail (uncurry boot) initAndFp
where
boot initCfg fp =
void$runMVC mempty
oracleModel (core initCfg fp)
bailNoConfig =
errorExit "Please pass a config"
bail (InvalidConfig e)=
errorExit
("Invalid config " ++show e)
load' fp =
hoistEither
.fmapLInvalidConfig
=<<lift (load fp)
If you found that very dense and difficult to follow at this
point, we’d encourage you to have another look at it after we’ve
covered monad transformers.

CHAPTER 19. MONADS GONE WILD 1237
19.6 An end-to-end example: URL
shortener
In this section, we’re going to walk through an entire program,
beginning to end.2There are some pieces we are not going to
explain thoroughly; however, this is something you can build
and work with if you’re interested in doing so.
First, the .cabal file for the project:
name: shawty
version: 0.1.0.0
synopsis: URI shortener
description: Please see README.md
homepage: http://github.com/
license: BSD3
license-file: LICENSE
author: Chris Allen
maintainer: cma@bitemyapp.com
copyright: 2015, Chris Allen
category: Web
build-type: Simple
cabal-version: >=1.10
executable shawty
2The code in this example can be found here: https://github.com/bitemyapp/
shawty-prime/blob/master/app/Main.hs

CHAPTER 19. MONADS GONE WILD 1238
hs-source-dirs: app
main-is: Main.hs
ghc-options: -threaded
build-depends: base
, bytestring
, hedis
, mtl
, network-uri
, random
, scotty
, semigroups
, text
, transformers
default-language: Haskell2010
And the project layout:
$ tree
.
├── LICENSE
├── Setup.hs
├── app
│   └── Main.hs
├── shawty.cabal
└── stack.yaml

CHAPTER 19. MONADS GONE WILD 1239
You may choose to use Stack or not. That is how we got the
template for the project in place. If you’d like to learn more,
check out Stack’s Github repo3and the Stack video tutorial4
we worked on together. The code following from this point is
inMain.hs .
We need to start our program oﬀ with a language extension:
{-# LANGUAGE OverloadedStrings #-}
OverloadedStrings is a way to make String literals polymor-
phic, the way numeric literals are polymorphic over the Num
typeclass. String literals are not ordinarily polymorphic; String
is a concrete type. Using OverloadedStrings allows us to use
String literals as TextandByteString values.
Brief aside about polymorphic literals
We mentioned that the integral number literals in Haskell
are typed Num a => a by default. Now that we have another
example to work with, it’s worth examining how they work
under the hood, so to speak. First, let’s look at a typeclass from
a module in base:
Prelude> import Data.String
3Stack Github repo https://github.com/commercialhaskell/stack
4The video Stack mega-tutorial! The whole video is long, but covers a lot of abnormal
use cases. Use the time stamps to jump to what you need to learn. https://www.youtube.
com/watch?v=sRonIB8ZStw&feature=youtu.be

CHAPTER 19. MONADS GONE WILD 1240
Prelude> :info IsString
class IsString a where
fromString :: String -> a
-- Defined in ‘Data.String’
instance IsString [Char]
-- Defined in ‘Data.String’
Then we may notice something in NumandFractional :
classNumawhere
-- irrelevant bits elided
fromInteger ::Integer ->a
classNuma=>Fractional awhere
-- elision again
fromRational ::Rational ->a
OK, and what about our literals?
Prelude> :set -XOverloadedStrings
Prelude> :t 1
1 :: Num a => a
Prelude> :t 1.0
1.0 :: Fractional a => a
Prelude> :t "blah"
"blah" :: IsString a => a

CHAPTER 19. MONADS GONE WILD 1241
The basic design is that the underlying representation is
concrete, butGHCautomaticallywrapsitin fromString /fromInteger /fromRational .
So it’s as if:
{-# LANGUAGE OverloadedStrings #-}
"blah"::Text
==fromString ( "blah"::String)
1::Int
==fromInteger ( 1::Integer)
2.5::Double
==fromRational ( 2.5::Rational )
Librarieslike textandbytestring provideinstancesfor IsString
in order to perform the conversion. Assuming you have those
libraries installed, you can kick it around a little. Note that,
due to the monomorphism restriction, the following will work
in the REPL but would not work if we loaded it from a source
file (because it would default to a concrete type; we’ve seen
this a couple times earlier in the book):
Prelude> :set -XOverloadedStrings
Prelude> let a = "blah"
Prelude> a

CHAPTER 19. MONADS GONE WILD 1242
"blah"
Prelude> :t a
a :: Data.String.IsString a => a
Then you can make it a TextorByteString value:
Prelude> import Data.Text (Text)
Prelude> :{
*Main| import Data.ByteString (ByteString)
*Main| :}
Prelude> let t = "blah" :: Text
Prelude> let bs = "blah" :: ByteString
Prelude> t == bs
Couldn't match expected type ‘Text’ with
actual type ‘ByteString’
In the second argument of ‘(==)’,
namely ‘bs’
In the expression: t == bs
OverloadedStrings is a convenience that originated in the
desire of working Haskell programmers to use String literals
forTextandByteString values. It’s not too big a deal, but it
can be nice and saves you manually wrapping each literal in
fromString .

CHAPTER 19. MONADS GONE WILD 1243
Back to the show
Next, the module name must be Mainas that is required for
anything exporting a mainexecutable to be invoked when the
program runs. We follow the OverloadedStrings extension with
our imports:
moduleMainwhere
importControl.Monad (replicateM )
importControl.Monad.IO.Class (liftIO)
import qualified Data.ByteString.Char8
asBC
importData.Text.Encoding
(decodeUtf8 ,encodeUtf8 )
import qualified Data.Text.Lazy asTL
import qualified Database.Redis asR
importNetwork.URI (URI,parseURI )
import qualified System.Random asSR
importWeb.Scotty
Where we import something “qualified (…) as (…)” we are
doing two things. Qualifying the import means that we can
only refer to values in the module with the full module path,
and we use asto give the module that we want in scope a name.
For example, Data.ByteString.Char8.pack is a fully qualified ref-

CHAPTER 19. MONADS GONE WILD 1244
erence to pack. We qualify the import so that we don’t import
declarations that would conflict with bindings that already
exist in Prelude . By specifying a name using as, we can give the
value a shorter, more convenient name. Where we import the
module name followed by parentheses, such as with replicateM
orliftIO , we are saying we only want to import the functions
or values of that name and nothing else. In the case of import
Web.Scotty , we are importing everything Web.Scotty exports. An
unqualified and unspecific import should be avoided except in
those cases where the provenance of the imported functions
will be obvious, or when the import is a toolkit you must use
all together, such as scotty .
Next we need to generate our shortened URLs that will refer
to the links people post to the service. We will make a String
of the characters we want to select from:
alphaNum ::String
alphaNum =['A'..'Z']++['0'..'9']
Now we need to pick random elements from alphaNum . The
general idea here should be familiar from the hangman game.
First, we find the length of the list to determine a range to
select from, then get a random number in that range, using IO
to handle the randomness:

CHAPTER 19. MONADS GONE WILD 1245
randomElement ::String->IOChar
randomElement xs= do
letmaxIndex ::Int
maxIndex =length xs -1
-- Right of arrow is IO Int,
-- so randomDigit is Int
randomDigit <-SR.randomRIO ( 0, maxIndex)
return (xs !!randomDigit)
Next, we apply randomElement toalphaNum to get a single ran-
domletterornumberfromouralphabet. Thenweuse replicateM
7to repeat this action 7 times, giving a list of 7 random letters
or numbers:
shortyGen ::IO[Char]
shortyGen =
replicateM 7(randomElement alphaNum)
For additional fun, see what replicateM 2 [1, 3] does and
whether you can figure out why. Compare it to the Prelude
function, replicate .
You may have noticed a mention of Redis in our imports
and wondered what was up. If you’re not already familiar with
it, Redis is in-memory, key-value data storage. The details
of how Redis works are well beyond the scope of this book
and they’re not very important here. Redis can be convenient

CHAPTER 19. MONADS GONE WILD 1246
for some common use cases like caching, or when you want
persistence without a lot of ceremony, as was the case here.
You will need to install and have Redis running in order for
the project to work; otherwise, the web server will throw an
error upon failing to connect to Redis.
This next bit is a function whose arguments are our con-
nection to Redis ( R.Connection ), the key we are setting in Redis,
and the value we are setting the key to. We also perform side
eﬀects in IOto getEither R.Reply R.Status as a result. The key
in this case is the randomly generated URI we created, and
the value is the URL the user wants the shortener to provide
at that address:
saveURI ::R.Connection
->BC.ByteString
->BC.ByteString
->IO(EitherR.ReplyR.Status)
saveURI conn shortURI uri =
R.runRedis conn $R.set shortURI uri
The next function, getURI , takes the connection to Redis and
the shortened URI key in order to get the URI associated with
that short URL and show users where they’re headed:

CHAPTER 19. MONADS GONE WILD 1247
getURI ::R.Connection
->BC.ByteString
->IO(EitherR.Reply
(MaybeBC.ByteString ))
getURIconn shortURI =
R.runRedis conn $R.get shortURI
Next some basic templating functions for returning output
to the web browser:
linkShorty ::String->String
linkShorty shorty=
concat
["<a href= \""
, shorty
,"\">Copy and paste your short URL</a>"
]
The final output to scotty has to be a Textvalue, so we’re
concatenating lists of Textvalues to produce responses to the
browser:

CHAPTER 19. MONADS GONE WILD 1248
-- TL.concat :: [TL.Text] -> TL.Text
shortyCreated ::Showa
=>a
->String
->TL.Text
shortyCreated resp shawty =
TL.concat [ TL.pack (show resp)
," shorty is: "
,TL.pack (linkShorty shawty)
]
shortyAintUri ::TL.Text->TL.Text
shortyAintUri uri=
TL.concat
[ uri
," wasn't a url,"
," did you forget http://?"
]
shortyFound ::TL.Text->TL.Text
shortyFound tbs=
TL.concat
["<a href= \""
, tbs,"\">"
, tbs,"</a>"]

CHAPTER 19. MONADS GONE WILD 1249
Now we get to the bulk of web-appy bits in the form of our
application. We’ll enumerate the application in chunks, but
they’re all in one appfunction:
app::R.Connection
->ScottyM ()
apprConn= do
-- [1]
get"/"$ do
-- [2]
uri<-param"uri"
-- [3]
1.Redis connection that should’ve been fired up before the
web server started.
2.getis a function that takes a RoutePattern , an action that
returns an HTTP response, and adds the route to the
Scotty server it’s embedded in. As you might suspect,
RoutePattern has anIsString instance so that the pattern
can be a String literal. The top-level route is expressed as
”/”, i.e., likegoingto https://google.com/ orhttps://bitemyapp.
com/. That final /character is what’s being expressed.
3.Theparamfunction is a means of getting…parameters.

CHAPTER 19. MONADS GONE WILD 1250
param::Parsable a
=>Data.Text.Internal .Lazy.Text
->ActionM a
It’s sort of like Read, but it’s parsing a value of the type
you ask for. The paramfunction can find arguments via
URL path captures (see below with :short), HTML form
inputs, or query parameters. The first argument to param
is the “name” or key for the input. We cannot explain the
entirety of HTTP and HTML here, but the following are
means of getting a param with the key name:
a)URL path capture
get"/user/:name/view" $ do
-- requesting the URL /user/Blah/view
-- would make name = "Blah"
-- such as:
-- http://localhost:3000/user/Blah/view
b)HTML input form. Here the name attribute for the
input field is ”name”.
<form>
Name:<br>
<input type="text" name="name">
</form>

CHAPTER 19. MONADS GONE WILD 1251
c)Query parameters for URIs are pairings of keys and
values following a question mark.
http://localhost:3000/?name=Blah
You can define more than one by using ampersand to
separate the key value pairs.
/?name=Blah&state=Texas
Now for the next chunk of the appfunction:
letparsedUri ::MaybeURI
parsedUri =
parseURI ( TL.unpack uri)
caseparsedUri of
-- [1]
Just_ -> do
shawty<-liftIO shortyGen
-- [2]
letshorty=BC.pack shawty
-- [3]

CHAPTER 19. MONADS GONE WILD 1252
uri'=
encodeUtf8 ( TL.toStrict uri)
-- [4]
resp<-
liftIO (saveURI rConn shorty uri')
-- [5]
html (shortyCreated resp shawty)
-- [6]
Nothing ->text (shortyAintUri uri)
-- [7]
1.We test that the user gave us a valid URI by using the
network-uri library’s parseURI function. We don’t really
care about the datatype it got wrapped in, so when we
check if it’s JustorNothing , we drop it on the floor.
2.TheMonadhere is ActionM (an alias of ActionT ), which is a
datatype representing code that handles web requests
and returns responses. You can perform IOactions in
thisMonad, but you have to lift the IOaction over the addi-
tional structure. Conventionally, one uses MonadIO as a sort
of auto-lift for IOactions, but you could do it manually.
We won’t demonstrate this here. We will explain monad
transformers in a later chapter so that ActionT will be less
mysterious.

CHAPTER 19. MONADS GONE WILD 1253
3.ConvertingtheshortcodefortheURIintoa Char8ByteString
for storage in Redis.
4.Converting the URI the user provided from a lazy Text
value into a strict Textvalue, then encoding as a UTF-8 (a
common Unicode format) ByteString for storage in Redis.
5.Again using liftIO so that we can perform an IOaction
inside a scotty ActionM . In this case, we’re saving the short
code and the URI in Redis so that we can look things up
with the short code as a key, then get the URI back as a
value if it has been stored in the past.
6.The templated response we return when we successfully
saved the short code for the URI. This gives the user a
shortened URI to share.
7.Error response in case the user gave us a URI that wasn’t
valid.
The second handler handles requests to a shortened URI
and returns the unshortened URL to follow:

CHAPTER 19. MONADS GONE WILD 1254
get"/:short" $ do
-- [1]
short<-param"short"
-- [2]
uri<-liftIO (getURI rConn short)
-- [3]
caseuriof
Leftreply->
text (TL.pack (show reply))
-- [4] [5]
RightmbBS-> case mbBSof
-- [6]
Nothing ->text"uri not found"
-- [7]
Justbs->html (shortyFound tbs)
-- [8]
wheretbs::TL.Text
tbs=
TL.fromStrict
(decodeUtf8 bs)
-- [9]
1.This is the URL path capture we mentioned earlier, such
that requesting /blahfrom the server will cause it to get
the key “blah” from Redis and, if there’s a value stored in

CHAPTER 19. MONADS GONE WILD 1255
that key, return that URI in the response. To do that in a
web browser or with curl/wget, you’d point your client at
http://localhost:3000/blah to test it.
2.Same parameter fetching as before. This time we expect
it to be part of the path capture rather than a query argu-
ment.
3.Lifting an IOaction inside ActionM again, this time to get
the short code as the lookup key from Redis.
4.Lefthere (in the Either we get back from Redis) signifies
some kind of failure, usually an error.
5.Textresponse returning an error in case we got Leftso that
the user knows what the error was, taking advantage of
Redis having Showable errors to render it in the response.
6.Happy path.
7.Just because an error didn’t happen doesn’t mean the key
was in the database.
8.Wefetchakeythatexistsinthedatabase, getthe ByteString
out of the Justdata constructor and render the URI in the
success template to show the user the URI we stored.
9.Going in the opposite direction we went in before — de-
coding the ByteString on the assumption it’s encoded as

CHAPTER 19. MONADS GONE WILD 1256
UTF-8, then converting from a strict Textvalue to a lazy
Textvalue.
Now we come to the mainevent.mainreturns IO ()and acts as
the entry point for our web server when we start the executable.
We begin by invoking scotty 3000 , a helper function from the
scotty framework which, given a port to run on and a scotty
application, will listen for requests and respond to them:
main::IO()
main= do
rConn<-R.connect R.defaultConnectInfo
scotty3000(app rConn)
And that is the entirety of this URL shortener. We have a
couple of exercises based on this code, and we encourage you
to come back to it after we’ve covered monad transformers as
well and see how your comprehension is growing.
Exercise
In the URL shortener, an important step was omitted. We’re
not checking if we’re overwriting an existing short code, which
is entirely possible despite them being randomly generated.
We can calculate the odds of this by examining the cardinality
of the values.

CHAPTER 19. MONADS GONE WILD 1257
-- alphaNum = ['A'..'Z'] ++ ['0'..'9']
-- shortyGen =
-- replicateM 7 (randomElement alphaNum)
lengthalphaNum ^7==78364164096
So, the problem is, what if we accidentally clobber a previ-
ously generated short URI? There are a few ways of solving
this. One is to check to see if the short URI already exists in the
database before saving it and throwing an error if it does. This
is going to be vanishingly unlikely to happen unless you’ve
suddenly become a very popular URI shortening service, but
it’d prevent the loss of any data. Your exercise is to devise
some means of making this less likely. The easiest way would
be to simply make the short codes long enough that you’d
need to run a computer until the heat death of the universe
to get a collision, but you should try throwing an error in the
first handler we showed you first.
19.7 That’s a wrap!
We hope this chapter gave you some idea of how Haskellers
use the typeclasses we’ve been talking about in real code, to
handle various types of problems. In the next two chapters,
we’ll be looking at Foldable andTraversable , two typeclasses
with some interesting properties that rely on these four alge-

CHAPTER 19. MONADS GONE WILD 1258
braic structures (monoid, functor, applicative, and monad),
so we encourage you to take some time to explore some of
the uses we’ve demonstrated here. Consider going back to
anything you didn’t understand very well the first time you
went through those chapters.
19.8 Follow-up resources
1.The case of the mysterious explosion in space; Bryan
O’Sullivan; Explains how GHC handles string literals.

Chapter 20
Foldable
You gotta know when to
hold ’em, know when to
fold ’em, know when to
walk away, know when to
run.
Kenny Rogers
1259

CHAPTER 20. FOLDABLE 1260
20.1 Foldable
This typeclass has been appearing in type signatures at least
since Chapter 3, but for your purposes in those early chapters,
we said you could think of a Foldable thing as a list. As you
saw in the chapter on folds, lists are certainly foldable data
structures. But it is also true that lists are not the only foldable
data structures, so this chapter will expand on the idea of
catamorphisms and generalize it to many datatypes.
A list fold is a way to reduce the values inside a list to one
summary value by recursively applying some function. It is
sometimes difficult to appreciate that, as filtering and mapping
functions may be implemented in terms of a fold and yet
return an entirely new list! The new list is the summary value
of the old list after being reduced, or transformed, by function
application.
The folding function is always dependent on some Monoid
instance. The folds we wrote previously mostly relied on
implicit monoidal operations. As we’ll see in this chapter,
generalizing catamorphisms to other datatypes depends on
understanding the monoids for those structures and, in some
cases, making them explicit.
This chapter will cover:
•theFoldable class and its core operations;
•the monoidal nature of folding;

CHAPTER 20. FOLDABLE 1261
•standard operations derived from folding.
20.2 The Foldable class
The Hackage documentation for the Foldable typeclass de-
scribes it as being a, “class of data structures that can be folded
to a summary value.” The folding operations that we’ve seen
previously fit neatly into that definition, but this typeclass in-
cludes many operations. We’re going to go through the full
definition a little at a time. The definition in the library begins:
classFoldable twhere
{-# MINIMAL foldMap | foldr #-}
TheMINIMAL annotation on the typeclass tells you that a
minimally complete definition of the typeclass will define
foldMap orfoldrfor a datatype. As it happens, foldMap andfoldr
can each be implemented in terms of the other, and the other
operations included in the typeclass can be implemented in
terms of either of them. As long as at least one is defined,
you have a working instance of Foldable . Some methods in the
typeclass have default implementations that can be overridden
when needed. This is in case there’s a more efficient way to
do something that’s specific to your datatype.
If you query the info about the typeclass in GHCi, the first
line of the definition includes the kind signature for 𝑡:

CHAPTER 20. FOLDABLE 1262
class Foldable (t :: * -> *) where
That𝑡should be a higher-kinded type is not surprising: lists
are higher-kinded types. We need 𝑡to be a type constructor
for the same reasons we did with Functor , and we will see that
the eﬀects are very similar. Types that take more than one
type argument, such as tuples and Either , will necessarily have
their first type argument included as part of their structure.
Please note that you will need to use GHC 7.10 or later ver-
sions for all the examples in this chapter to work. Also, while
thePrelude as of GHCi 7.10 includes many changes related to
theFoldable typeclass, not all of Foldable is in the Prelude . To
follow along with the examples in the chapter, you may need
to import Data.Foldable andData.Monoid (for some of the Monoid
newtypes).
20.3 Revenge of the monoids
One thing we did not talk about when we covered folds pre-
viously is the importance of monoids. Folding necessarily
implies a binary associative operation that has an identity
value. The first two operations defined in Foldable make this
explicit:

CHAPTER 20. FOLDABLE 1263
classFoldable (t:: * -> * )where
fold::Monoidm=>t m->m
foldMap ::Monoidm
=>(a->m)->t a->m
Whilefoldallows you to combine elements inside a Foldable
structure using the Monoid defined for those elements, foldMap
first maps each element of the structure to a Monoid and then
combines the results using that instance of Monoid .
Thesemightseemalittleweirduntilyourealizethat Foldable
is requiring that you make the implicit Monoid visible in folding
operations. Let’s take a look at a very basic foldroperation
and see how it compares to foldandfoldMap :
Prelude> foldr (+) 0 [1..5]
15
The binary associative operation for that fold is (+), so we’ve
specified it without thinking of it as a monoid. The fact that
the numbers in our list have other possible monoids is not
relevant once we’ve specified which operation to use.
We can already see from the type of foldthat it’s not going to
work the same as foldr, because it doesn’t take a function for its
first argument. But we also can’t just fold up a list of numbers,
because the foldfunction doesn’t have a Monoid specified:
Prelude> fold (+) [1, 2, 3, 4, 5]

CHAPTER 20. FOLDABLE 1264
-- error message resulting from incorrect
-- number of arguments
Prelude> fold [1, 2, 3, 4, 5]
-- error message resulting from not having
-- an instance of Monoid
So, what we need to do to make foldwork is specify a Monoid
instance:
Prelude> let xs = map Sum [1..5]
Prelude> fold xs
Sum {getSum = 15}
Or, less tediously:
Prelude> :{
*Main| let xs :: Sum Integer
*Main| xs = [1, 2, 3, 4, 5]
*Main| :}
Prelude> fold xs
Sum {getSum = 15}
Prelude> :{
*Main| let xs :: Product Integer
*Main| xs = [1, 2, 3, 4, 5]
*Main| :}
Prelude> fold xs

CHAPTER 20. FOLDABLE 1265
Product {getProduct = 120}
In some cases, the compiler can identify and use the stan-
dardMonoid for a type, without us being explicit:
Prelude> foldr (++) "" ["hello", " julie"]
"hello julie"
Prelude> fold ["hello", " julie"]
"hello julie"
The default Monoid instance for lists gives us what we need
without having to specify it.
And now for something diﬀerent
Let’s turn our attention now to foldMap . Unlike fold,foldMap has
a function as its first argument. Unlike foldr, the first (function)
argument of foldMap must explicitly map each element of the
structure to a Monoid :
Prelude> foldMap Sum [1, 2, 3, 4]
Sum {getSum = 10}
Prelude> foldMap Product [1, 2, 3, 4]
Product {getProduct = 24}
Prelude> foldMap All [True, False, True]
All {getAll = False}

CHAPTER 20. FOLDABLE 1266
Prelude> foldMap Any [(3 == 4), (9 > 5)]
Any {getAny = True}
Prelude> let xs = [Just 1, Nothing, Just 5]
Prelude> foldMap First xs
First {getFirst = Just 1}
Prelude> foldMap Last xs
Last {getLast = Just 5}
In the above examples, the function being applied is a data
constructor. The data constructor identifies the Monoid instance
— themappend — for those types. It already contains enough
information to allow foldMap to reduce the collection of values
to one summary value.
However, foldMap can also have a function to map that is
diﬀerent from the Monoid it’s using:
Prelude> let xs = map Product [1..3]
Prelude> foldMap (*5) xs
Product {getProduct = 750}
-- 5 * 10 * 15
750
Prelude> let xs = map Sum [1..3]
Prelude> foldMap (*5) xs
Sum {getSum = 30}

CHAPTER 20. FOLDABLE 1267
-- 5 + 10 + 15
30
It can map the function to each value first and then use the
Monoid instance to reduce them to one value. Compare this to
foldrin which the function has the Monoid instance baked in:
Prelude> foldr (*) 5 [1, 2, 3]
-- (1 * (2 * (3 * 5)))
30
In fact, due to the way foldrworks, declaring a Monoid in-
stance that is diﬀerent from what is implied in the folding
function doesn’t change the final result:
Prelude> let sumXs = map Sum [2..4]
Prelude> foldr (*) 3 sumXs
Sum {getSum = 72}
Prelude> let productXs = map Product [2..4]
Prelude> foldr (*) 3 productXs
Product {getProduct = 72}
However, it is worth pointing out that if what you’re trying
to fold only contains one value, declaring a Monoid instance
won’t change the behavior of foldMap either:
Prelude> let fm = foldMap (*5)
Prelude> fm (Just 100) :: Product Integer

CHAPTER 20. FOLDABLE 1268
Product {getProduct = 500}
Prelude> fm (Just 5) :: Sum Integer
Sum {getSum = 25}
With only one value, it doesn’t need the Monoid instance.
Specifying the Monoid instance is necessary to satisfy the type-
checker, but with only one value, there is nothing to mappend .
It just applies the function. It will use the mempty value from
the declared Monoid instance, though, in cases where what you
are trying to fold is empty:
Prelude> fm Nothing :: Sum Integer
Sum {getSum = 0}
Prelude> fm Nothing :: Product Integer
Product {getProduct = 1}
So, what we’ve seen so far is that Foldable is a way of general-
izing catamorphisms — folding — to diﬀerent datatypes, and
at least in some cases, it forces you to think about the monoid
you’re using to combine values.
20.4 Demonstrating Foldable instances
As we said above, a minimal Foldable instance must have either
foldrorfoldMap . Any of the other functions in this typeclass
can be derived from one or the other of those. With that said,

CHAPTER 20. FOLDABLE 1269
let’s turn our attention to implementing Foldable instances for
diﬀerent types.
Identity
We’ll kick things oﬀ by writing a Foldable instance for Identity :
dataIdentity a=
Identity a
We’re only obligated to write foldrorfoldMap , but we’ll write
both plus foldlso you have the gist of it.
instance Foldable Identity where
foldr f z ( Identity x)=f x z
foldl f z ( Identity x)=f z x
foldMap f ( Identity x)=f x
Withfoldrandfoldl, we’re doing basically the same thing,
but with the arguments swapped. We didn’t need to do any-
thing special for foldMap .
It may seem strange to think of folding one value. When
we’ve talked about catamorphisms previously, we’ve focused
on how they can reduce a bunch of values down to one sum-
mary value. In the case of this Identity catamorphism, though,

CHAPTER 20. FOLDABLE 1270
the point is less to reduce the values inside the structure to
one value and more to consume, or use, the value:
Prelude> foldr (*) 1 (Identity 5)
5
Prelude> foldl (*) 5 (Identity 5)
25
Prelude> let fm = foldMap (*5)
Prelude> type PI = Product Integer
Prelude> fm (Identity 100) :: PI
Product {getProduct = 500}
Maybe
Thisoneisalittlemoreinterestingbecause, unlikewith Identity ,
we have to account for the Nothing cases. When the Maybevalue
that we’re folding is Nothing , we need to be able to return some
“zero” value, while doing nothing with the folding function
but also disposing of the Maybestructure. For foldrandfoldl,
that zero value is the start value provided:
Prelude> foldr (+) 1 Nothing
1
On the other hand, for foldMap we use the Monoid’s identity
value as our zero:

CHAPTER 20. FOLDABLE 1271
Prelude> let fm = foldMap (+1)
Prelude> fm Nothing :: Sum Integer
Sum {getSum = 0}
When the value is a Justvalue, though, we need to apply
the folding function to the value and, again, dispose of the
structure:
Prelude> foldr (+) 1 (Just 3)
4
Prelude> fm $ Just 3 :: Sum Integer
Sum {getSum = 4}
So, let’s look at the instance. We’ll use a fake Maybetype
again, to avoid conflict with the Maybe instance that already
exists:
instance Foldable Optional where
foldr_zNada=z
foldr f z ( Yepx)=f x z
foldl_zNada=z
foldl f z ( Yepx)=f z x
foldMap _Nada=mempty
foldMap f ( Yepa)=f a

CHAPTER 20. FOLDABLE 1272
Note that if you don’t tell it what Monoid you mean, it will
complain about the type being ambiguous:
Prelude> foldMap (+1) Nada
No instance for (Num a0) arising
from a use of ‘it’
The type variable ‘a0’ is ambiguous
(... blah blah who cares ...)
So, we need to assert a type that has a Monoid for this to work:
Prelude> import Data.Monoid
Prelude> foldMap (+1) Nada :: Sum Int
Sum {getSum = 0}
Prelude> foldMap (+1) Nada :: Product Int
Product {getProduct = 1}
Prelude> foldMap (+1) (Just 1) :: Sum Int
Sum {getSum = 2}
With aNadavalue and a declared type of Sum Int (giving us
ourMonoid ),foldMap gave us Sum 0because that was the mempty or
identity for Sum. Similarly with NadaandProduct , we got Product
1because that was the identity for Product .

CHAPTER 20. FOLDABLE 1273
20.5 Some basic derived operations
TheFoldable typeclass includes some other operations that
we haven’t covered in this context yet. Some of these, such
aslength, were previously defined for use with lists, but their
types have been generalized now to make them useful with
other types of data structures. Below are descriptions, type
signatures, and examples for several of these:
-- | List of elements of a structure,
-- from left to right.
toList::t a->[a]
Prelude> toList (Just 1)
[1]
Prelude> let xs = [Just 1, Just 2, Just 3]
Prelude> map toList xs
[[1],[2],[3]]
Prelude> concatMap toList xs
[1,2,3]
Prelude> let xs = [Just 1, Just 2, Nothing]
Prelude> concatMap toList xs
[1,2]
Prelude> toList (1, 2)
[2]

CHAPTER 20. FOLDABLE 1274
Why doesn’t it put the 1 in the list? For the same reason that
fmapdoesn’t apply a function to the 1.
-- | Test whether the structure is empty.
null::t a->Bool
Notice that nullreturns TrueonLeftandNothing values, just
as it does on empty lists and so forth:
Prelude> null (Left 3)
True
Prelude> null []
True
Prelude> null Nothing
True
Prelude> null (1, 2)
False
Prelude> let xs = [Just 1, Just 2, Nothing]
Prelude> fmap null xs
[False,False,True]
The next one, length , returns a count of how many 𝑎values
inhabit the t a. In a list, that could be multiple 𝑎values due to
the definition of that datatype. It’s important to note, though,
that for tuples, the first argument (as well as the leftmost, or
outermost, type arguments of datatypes such as Maybeand
Either ) is part of the 𝑡here, not part of the 𝑎.

CHAPTER 20. FOLDABLE 1275
-- | Returns the size/length of a finite
-- structure as an 'Int'.
length::t a->Int
Prelude> length (1, 2)
1
Prelude> let xs = [(1, 2), (3, 4), (5, 6)]
Prelude> length xs
3
Prelude> fmap length xs
[1,1,1]
Prelude> fmap length Just [1, 2, 3]
1
The last example looks strange, we know. But if you run
it in your REPL, you’ll see it returns the result we promised.
Why? And why does this
Prelude> length $ Just [1, 2, 3]
1
return the same result?
The𝑎ofJust a in the last case above is a list. There is only
one list.

CHAPTER 20. FOLDABLE 1276
Prelude> let xs = [Just 1, Just 2, Just 3]
Prelude> fmap length xs
[1,1,1]
Prelude> let xs = [Just 1, Just 2, Nothing]
Prelude> fmap length xs
[1,1,0]
-- | Does the element occur
-- in the structure?
elem::Eqa=>a->t a->Bool
We’ve used Either in the following example set to demon-
strate the behavior of Foldable functions with Leftvalues. As
we saw with Functor , you can’t map over the left data construc-
tor, because the left type argument is part of the structure. In
the following example set, that means that elemcan’t see inside
theLeftconstructor to whatever the value is, so the result will
beFalse, even if the value matches:
Prelude> elem 2 (Just 3)
False
Prelude> elem True (Left False)
False
Prelude> elem True (Left True)
False
Prelude> elem True (Right False)
False

CHAPTER 20. FOLDABLE 1277
Prelude> elem True (Right True)
True
Prelude> let xs = [Right 1,Right 2,Right 3]
Prelude> fmap (elem 3) xs
[False,False,True]
-- | The largest element
-- of a non-empty structure.
maximum ::Orda=>t a->a
-- | The least element
-- of a non-empty structure.
minimum ::Orda=>t a->a
Here, notice that LeftandNothing (and similar) values are
empty for the purposes of these functions:
Prelude> maximum [10, 12, 33, 5]
33
Prelude> let xs = [Just 2, Just 10, Just 4]
Prelude> fmap maximum xs
[2,10,4]
Prelude> fmap maximum (Just [3, 7, 10, 2])
Just 10

CHAPTER 20. FOLDABLE 1278
Prelude> minimum "julie"
'e'
Prelude> fmap minimum (Just "julie")
Just 'e'
Prelude> let xs = map Just "jul"
Prelude> xs
[Just 'j',Just 'u',Just 'l']
Prelude> fmap minimum xs
"jul"
Prelude> let xs = [Just 4, Just 3, Nothing]
Prelude> fmap minimum xs
[4,3,*** Exception:
minimum: empty structure
Prelude> minimum (Left 3)
*** Exception: minimum: empty structure
We’ve seen sumandproduct before, and they do what their
names suggest: return the sum and product of the members
of a structure:
sum::(Foldable t,Numa)=>t a->a
product ::(Foldable t,Numa)=>t a->a
And now for some examples:

CHAPTER 20. FOLDABLE 1279
Prelude> sum (7, 5)
5
Prelude> fmap sum [(7, 5), (3, 4)]
[5,4]
Prelude> fmap sum (Just [1, 2, 3, 4, 5])
Just 15
Prelude> product Nothing
1
Prelude> fmap product (Just [])
Just 1
Prelude> fmap product (Right [1, 2, 3])
Right 6
Exercises: Library Functions
Implement the functions in terms of foldMap orfoldrfrom
Foldable , then try them out with multiple types that have
Foldable instances.
1.This and the next one are nicer with foldMap , butfoldris
fine too.
sum::(Foldable t,Numa)=>t a->a
2.product ::(Foldable t,Numa)=>t a->a
3.elem::(Foldable t,Eqa)
=>a->t a->Bool

CHAPTER 20. FOLDABLE 1280
4.minimum ::(Foldable t,Orda)
=>t a->Maybea
5.maximum ::(Foldable t,Orda)
=>t a->Maybea
6.null::(Foldable t)=>t a->Bool
7.length::(Foldable t)=>t a->Int
8.Some say this is all Foldable amounts to.
toList::(Foldable t)=>t a->[a]
9.Hint: use foldMap .
-- | Combine the elements
-- of a structure using a monoid.
fold::(Foldable t,Monoidm)=>t m->m
10.DefinefoldMap in terms of foldr.
foldMap ::(Foldable t,Monoidm)
=>(a->m)->t a->m
20.6 Chapter Exercises
WriteFoldable instances for the following datatypes.

CHAPTER 20. FOLDABLE 1281
1.dataConstant a b=
Constant b
2.dataTwoa b=
Twoa b
3.dataThreea b c=
Threea b c
4.dataThree'a b=
Three'a b b
5.dataFour'a b=
Four'a b b b
Thinking cap time. Write a filter function for Foldable types
usingfoldMap .
filterF ::(Applicative f
,Foldable t
,Monoid(f a))
=>(a->Bool)->t a->f a
filterF =undefined
20.7 Follow-up resources
1.Jakub Arnold; Foldable and Traversable.

Chapter 21
Traversable
O, Thou hast damnable
iteration; and art, indeed,
able to corrupt a saint.
Shakespeare
1282

CHAPTER 21. TRAVERSABLE 1283
21.1 Traversable
Functor gives us a way to transform any values embedded in
structure. Applicative gives us a way to transform any val-
ues contained within a structure using a function that is also
embedded in structure. This means that each application pro-
duces the eﬀect of adding structure which is then applicatively
combined. Foldable gives us a way to process values embedded
in a structure as if they existed in a sequential order, as we’ve
seen ever since we learned about list folding.
Traversable was introduced in the same paper as Applicative
and its introduction to Prelude didn’t come until the release
of GHC 7.10. However, it was available as part of the base
library before that. Traversable depends on Applicative , and
thusFunctor , and is also superclassed by Foldable .
Traversable allows you to transform elements inside the
structure like a functor, producing applicative eﬀects along the
way, and lift those potentially multiple instances of applicative
structure outside of the traversable structure. It is commonly
described as a way to traverse a data structure, mapping a
function inside a structure while accumulating the applicative
contexts along the way. This is easiest to see, perhaps, through
liberal demonstration of examples, so let’s get to it.
In this chapter, we will:
•explain the Traversable typeclass and its canonical func-

CHAPTER 21. TRAVERSABLE 1284
tions;
•explore examples of Traversable in practical use;
•tidy up some code using this typeclass;
•and, of course, write some Traversable instances.
21.2 The Traversable typeclass definition
This is the typeclass definition as it appears in the library
Data.Traversable :
class(Functor t,Foldable t)
=>Traversable twhere
traverse ::Applicative f=>
(a->f b)
->t a
->f (t b)
traverse f =sequenceA .fmap f
traverse maps each element of a structure to an action, eval-
uates the actions from left to right, and collects the results. So,
for example, if you have a list (structure) of IOactions, at the
end you’d have

CHAPTER 21. TRAVERSABLE 1285
-- | Evaluate each action in the
-- structure from left to right,
-- and collect the results.
sequenceA ::Applicative f
=>t (f a) ->f (t a)
sequenceA =traverse id
{-# MINIMAL traverse | sequenceA #-}
A minimal instance for this typeclass provides an imple-
mentation of either traverse orsequenceA , because as you can
see they can be defined in terms of each other. As with Foldable ,
we can define more than the bare minimum if we can leverage
information specific to our datatype to make the behavior
more efficient. We’re going to focus on these two functions in
this chapter, though, as those are the most typically useful.
21.3sequenceA
We will start with some examples of sequenceA in action, as it is
the simpler of the two. You can see from the type signature
above that the eﬀect of sequenceA is flipping two contexts or
structures. It doesn’t by itself allow you to apply any function
to the𝑎value inside the structure; it only flips the layers of
structure around. Compare these:
Prelude> sum [1, 2, 3]

CHAPTER 21. TRAVERSABLE 1286
6
Prelude> fmap sum [Just 1, Just 2, Just 3]
[1,2,3]
Prelude> (fmap . fmap) sum Just [1, 2, 3]
Just 6
Prelude> fmap product [Just 1, Just 2, Nothing]
[1,2,1]
To these:
Prelude> fmap Just [1, 2, 3]
[Just 1,Just 2,Just 3]
Prelude> sequenceA $ fmap Just [1, 2, 3]
Just [1,2,3]
Prelude> sequenceA [Just 1, Just 2, Just 3]
Just [1,2,3]
Prelude> sequenceA [Just 1, Just 2, Nothing]
Nothing
Prelude> fmap sum $ sequenceA [Just 1, Just 2, Just 3]
Just 6
Prelude> let xs = [Just 3, Just 4, Nothing]
Prelude> fmap product (sequenceA xs)
Nothing
In the first example, using sequenceA flips the structures
around — instead of a list of Maybevalues, you get a Maybeof a
list value. In the next two examples, we can lift a function over

CHAPTER 21. TRAVERSABLE 1287
theMaybestructure and apply it to the list that is inside, if we
have aJust a value after applying the sequenceA .
It’s worth mentioning here that the Data.Maybe module has a
function called catMaybes that oﬀers a diﬀerent way of handling
a list of Maybevalues:
Prelude> import Data.Maybe
Prelude> catMaybes [Just 1, Just 2, Just 3]
[1,2,3]
Prelude> catMaybes [Just 1, Just 2, Nothing]
[1,2]
Prelude> let xs = [Just 1, Just 2, Just 3, Nothing]
Prelude> sum $ catMaybes xs
6
Prelude> fmap sum $ sequenceA xs
Nothing
UsingcatMaybes allows you to sum (or otherwise process) the
list ofMaybevalues even if there’s potentially a Nothing value
lurking within.
21.4traverse
Let’s look next at the type of traverse :

CHAPTER 21. TRAVERSABLE 1288
traverse
::(Applicative f,Traversable t)
=>(a->f b)->t a->f (t b)
You might notice a similarity between that and the types of
fmapand(=<<)(flip bind):
fmap ::(a->b)->f a->f b
(=<<)::(a->m b)->m a->m b
traverse ::(a->f b)->t a->f (t b)
We’re still mapping a function over some embedded value(s), like
fmap, but similar to flip bind, that function is itself generating
more structure. However, unlike flip bind, that structure can
be of a diﬀerent type than the structure we lifted over to apply
the function. And at the end, it will flip the two structures
around, as sequenceA did.
In fact, as we saw in the typeclass definition, traverse isfmap
composed with sequenceA :
traverse f=sequenceA .fmap f
Let’s look at how that works in practice:
Prelude> fmap Just [1, 2, 3]
[Just 1,Just 2,Just 3]
Prelude> sequenceA $ fmap Just [1, 2, 3]

CHAPTER 21. TRAVERSABLE 1289
Just [1,2,3]
Prelude> sequenceA . fmap Just $ [1, 2, 3]
Just [1,2,3]
Prelude> traverse Just [1, 2, 3]
Just [1,2,3]
We’ll run through some longer examples in a moment, but
the general idea is that anytime you’re using sequenceA . fmap
f, you can use traverse to achieve the same result in one step.
mapMistraverse
Youmayhaveseenaslightlydiﬀerentwayofexpressing traverse
before, in the form of mapM.
In versions of GHC prior to 7.10, the type of mapMwas the
following:
mapM::Monadm
=>(a->m b)->[a]->m [b]
-- contrast with
traverse ::Applicative f
=>(a->f b)->t a->f (t b)
We can think of traverse inTraversable as abstracting the []
inmapMto being any traversable data structure and generalizing

CHAPTER 21. TRAVERSABLE 1290
theMonadrequirement to only need an Applicative . This is
valuable as it means we can use this pattern more widely and
with more code. For example, the list datatype is fine for
small pluralities of values but in more performance-sensitive
code, you may want to use a Vector from the vector1library.
Withtraverse , you won’t have to change your code because
the primary Vector datatype has a Traversable instance and so
should work.
Similarly, the type for sequence in GHC versions prior to
7.10 is a less useful sequenceA :
sequence ::Monadm
=>[m a]
->m [a]
-- contrast with
sequenceA ::(Applicative f,Traversable t)
=>t (f a)
->f (t a)
Again we’re generalizing the list to any Traversable and weak-
ening the Monadrequirement to Applicative .
1http://hackage.haskell.org/package/vector

CHAPTER 21. TRAVERSABLE 1291
21.5 So, what’s Traversable for?
In a literal sense, anytime you need to flip two type construc-
tors around, or map something and then flip them around,
that’s probably Traversable :
sequenceA ::Applicative f
=>t (f a) ->f (t a)
traverse ::Applicative f
=>(a->f b)->t a->f (t b)
We’ll kick around some hypothetical functions and values
without bothering to implement them in the REPL to see when
we may want traverse orsequenceA :
Prelude> let f = undefined :: a -> Maybe b
Prelude> let xs = undefined :: [a]
Prelude> :t map f xs
map f xs :: [Maybe b]
But what if we want a value of type Maybe [b] ? The following
will work, but we can do better:
Prelude> :t sequenceA $ map f xs
sequenceA $ map f xs :: Maybe [a]

CHAPTER 21. TRAVERSABLE 1292
It’s usually better to use traverse whenever we see a sequence
orsequenceA combined with a maporfmap:
Prelude> :t traverse f xs
traverse f xs :: Maybe [b]
Next we’ll start looking at real examples of when you’d want
to do this.
21.6 Morse code revisited
We’re going to call back to what we did in the Testing chapter
with the Morse code to look at a nice example of how to use
traverse . Let’s recall what we had done there:
stringToMorse ::String->Maybe[Morse]
stringToMorse s=
sequence $fmap charToMorse s
-- what we want to do:
stringToMorse ::String->Maybe[Morse]
stringToMorse =traverse charToMorse
Normally, you might expect that if you mapped an (a ->
f b)over at a, you’d end up with t (f b) buttraverse flips
that around. Remember, we had each character conversion
wrapped in a Maybedue to the possibility of getting characters in

CHAPTER 21. TRAVERSABLE 1293
a string that aren’t translatable into Morse (or, in the opposite
conversion, aren’t Morse characters):
Prelude> morseToChar "gobbledegook"
Nothing
Prelude> morseToChar "-.-."
Just 'c'
We can use fromMaybe to remove the Maybelayer:
Prelude> import Data.Maybe
Prelude Data.Maybe> fromMaybe ' ' (morseToChar "-.-.")
'c'
Prelude> stringToMorse "chris"
Just ["-.-.","....",".-.","..","..."]
Prelude> import Data.Maybe
Prelude> fromMaybe [] (stringToMorse "chris")
["-.-.","....",".-.","..","..."]
We’ll define a little helper for use in the following examples:
Prelude> let morse s = fromMaybe [] (stringToMorse s)
Prelude> :t morse
morse :: String -> [Morse]
Now, if we fmap morseToChar , we get a list of Maybevalues:

CHAPTER 21. TRAVERSABLE 1294
Prelude> fmap morseToChar (morse "chris")
[Just 'c',Just 'h',Just 'r',Just 'i',Just 's']
We don’t want catMaybes here because it drops the Nothing
values. What we want here is for any Nothing values to make the
final result Nothing . The function that gives us what we want for
this issequence . We did use sequence in the original version of
thestringToMorse function. sequence is useful for flipping your
types around as well (note the positions of the 𝑡and𝑚). There
is asequence inPrelude and another, more generic, version in
Data.Traversable :
Prelude> :t sequence
sequence :: (Monad m, Traversable t) =>
t (m a) -> m (t a)
-- more general, can be used with types
-- other than List
Prelude> import Data.Traversable as T
Prelude T> :t T.sequence
T.sequence :: (Traversable t, Monad m)
=> t (m a) -> m (t a)
To use this over a list of Maybe(or other monadic) values, we
need to compose it with fmap:
Prelude> :t (sequence .) . fmap

CHAPTER 21. TRAVERSABLE 1295
(sequence .) . fmap
:: (Monad m, Traversable t) =>
(a1 -> m a) -> t a1 -> m (t a)
Prelude> sequence $ fmap morseToChar (morse "chris")
Just "chris"
Prelude> sequence $ fmap morseToChar (morse "julie")
Just "julie"
The weird looking composition, which you’ve possibly also
seen in the form of (join .) . fmap is because fmaptakes two
(not one) arguments, so the expressions aren’t proper unless
we compose twice to await a second argument for fmapto get
applied to.
-- we want this
(sequence .).fmap=
\f xs->sequence (fmap f xs)
-- not this
sequence .fmap=
\f->sequence (fmap f)
This composition of sequence andfmapis so common that
traverse is now a standard Prelude function. Compare the
above to the following:

CHAPTER 21. TRAVERSABLE 1296
*Morse T> traverse morseToChar (morse "chris")
Just "chris"
*Morse T> traverse morseToChar (morse "julie")
Just "julie"
So,traverse isjustfmapandthe Traversable versionof sequence
bolted together into one convenient function. sequence is the
unique bit, but you need to do the fmapfirst most of the time,
so you end up using traverse . This is very similar to the way
>>=is justjoincomposed with fmapwherejoinis the bit that is
unique to Monad.
21.7 Axing tedious code
Try to bear with us for a moment and realize that the following
is real but also intentionally fake code. That is, one of the
authors helped somebody with refactoring their code, and
this simplified version is what your author was given. One of
the strengths of Haskell is that we can work in terms of types
without worry about code that actually runs sometimes. This
code is from Alex Petrov:

CHAPTER 21. TRAVERSABLE 1297
-- Thanks for the great example, Alex
dataQuery =Query
dataSomeObj =SomeObj
dataIoOnlyObj =IoOnlyObj
dataErr =Err
-- There's a decoder function that makes
-- some object from String
decodeFn ::String->EitherErrSomeObj
decodeFn =undefined
-- There's a query, that runs against the
-- DB and returns array of strings
fetchFn ::Query->IO[String]
fetchFn =undefined
-- an additional "context initializer",
-- that also has IO
makeIoOnlyObj ::[SomeObj]
->IO[(SomeObj,IoOnlyObj )]
makeIoOnlyObj =undefined

CHAPTER 21. TRAVERSABLE 1298
-- before
pipelineFn
::Query
->IO(EitherErr[(SomeObj,IoOnlyObj )])
pipelineFn query= do
a<-fetchFn query
casesequence (map decodeFn a) of
(Lefterr)->return$Left$err
(Rightres)-> do
a<-makeIoOnlyObj res
return$Righta
The objective was to clean up this code. A few things made
them suspicious:
1.The use of sequence andmap.
2.Manually casing on the result of the sequence and the map.
3.Binding monadically over the Either only to perform an-
other monadic ( IO) action inside of that.
We pared the pipeline function down to this:

CHAPTER 21. TRAVERSABLE 1299
pipelineFn
::Query
->IO(EitherErr[(SomeObj,IoOnlyObj )])
pipelineFn query= do
a<-fetchFn query
traverse makeIoOnlyObj (mapM decodeFn a)
Thanks to merijn on the IRC channel for helping with this.
We can make it pointfree if we want to:
pipelineFn
::Query
->IO(EitherErr[(SomeObj,IoOnlyObj )])
pipelineFn =
(traverse makeIoOnlyObj
.mapM decodeFn =<<).fetchFn
And since mapMis justtraverse with a slightly diﬀerent type:
pipelineFn
::Query
->IO(EitherErr[(SomeObj,IoOnlyObj )])
pipelineFn =
(traverse makeIoOnlyObj
.traverse decodeFn =<<).fetchFn

CHAPTER 21. TRAVERSABLE 1300
This is the terse, clean style many Haskellers prefer. As we
said back when we first introduced it, pointfree style can help
focus the attention on the functions, rather than the specifics
of the data that are being passed around as arguments. Using
functions like traverse cleans up code by drawing attention to
the ways the types are changing and signaling the program-
mer’s intent.
21.8 Do all the things
We’re going to use an HTTP client library named wreq2for
this demonstration so we can make calls to a handy-dandy
website for testing HTTP clients at http://httpbin.org/ . Feel
free to experiment and substitute your own ideas for HTTP
services or websites you could poke and prod.
2http://hackage.haskell.org/package/wreq

CHAPTER 21. TRAVERSABLE 1301
moduleHttpStuff where
importData.ByteString.Lazy hiding(map)
importNetwork.Wreq
-- replace with other websites
-- if desired or needed
urls::[String]
urls=["http://httpbin.org/ip"
,"http://httpbin.org/bytes/5"
]
mappingGet ::[IO(Response ByteString )]
mappingGet =map get urls
But what if we don’t want a list of IOactions we can perform
to get a response, but rather one big IOaction that produces a
list of responses? This is where Traversable can be helpful:
traversedUrls ::IO[Response ByteString ]
traversedUrls =traverse get urls
We hope that these examples have helped demonstrate that
Traversable is a useful typeclass. While Foldable seems trivial,
it is a necessary superclass of Traversable , andTraversable , like

CHAPTER 21. TRAVERSABLE 1302
Functor andMonad, is now widely used in everyday Haskell code,
due to its practicality.
Strength for understanding
Traversable is stronger than Functor andFoldable . Because of
this, we can recover the Functor andFoldable instance for a
type from the Traversable , just as we can recover the Functor
andApplicative from the Monad. Here we can use the Identity
type to get something that is essentially just fmapall over again:
Prelude> import Data.Functor.Identity
Prelude> traverse (Identity . (+1)) [1, 2]
Identity [2,3]
Prelude> runIdentity $ traverse (Identity . (+1)) [1, 2]
[2,3]
Prelude> :{
Prelude| let edgeMap f t =
Prelude| runIdentity $ traverse (Identity . f) t
Prelude| :}
Prelude> :t edgeMap
edgeMap :: Traversable t => (a -> b) -> t a -> t b
Prelude> edgeMap (+1) [1..5]
[2,3,4,5,6]

CHAPTER 21. TRAVERSABLE 1303
UsingConstorConstant , wecanrecoverafoldMappy-looking
Foldable as well:
Prelude> import Data.Monoid
-- from `transformers`
Prelude> import Data.Functor.Constant
Prelude> let xs = [1, 2, 3, 4, 5] :: [Sum Integer]
Prelude> traverse (Constant . (+1)) xs
Constant (Sum {getSum = 20})
Prelude> :{
Prelude| let foldMap' f t =
Prelude| getConstant $ traverse (Constant . f) t
Prelude| :}
Prelude> :t foldMap'
foldMap' :: (Traversable t, Monoid a)
=> (a1 -> a) -> t a1 -> a
Prelude> :t foldMap
foldMap :: (Foldable t, Monoid m) => (a -> m) -> t a -> m
Doing exercises like this can help strengthen your intuitions
for the relationships of these typeclasses and their canonical
functions. We know it sometimes feels like these things are
pure intellectual exercise, but getting comfortable with ma-
nipulating functions like these is ultimately the key to getting

CHAPTER 21. TRAVERSABLE 1304
comfortable with Haskell. This is how you learn to play type
Tetris with the pros.
21.9 Traversable instances
You knew this was coming.
Either
TheTraversable instance that follows here is identical to the
one in the Data.Traversable module in base, but we’ve added
aFunctor ,Foldable , andApplicative so that you might see a
progression:

CHAPTER 21. TRAVERSABLE 1305
dataEithera b=
Lefta
|Rightb
deriving (Eq,Ord,Show)
instance Functor (Eithera)where
fmap_(Leftx)=Leftx
fmap f ( Righty)=Right(f y)
instance Applicative (Eithere)where
pure =Right
Lefte<*> _ = Lefte
Rightf<*>r=fmap f r
instance Foldable (Eithera)where
foldMap _(Left_)=mempty
foldMap f ( Righty)=f y
foldr_z (Left_)=z
foldr f z ( Righty)=f y z
instance Traversable (Eithera)where
traverse _(Leftx)=pure (Leftx)
traverse f ( Righty)=Right<$>f y

CHAPTER 21. TRAVERSABLE 1306
Given what you’ve seen above, this hopefully isn’t too sur-
prising. We have function application and type-flipping, in an
Either context.
Tuple
As above, we’ve provided a progression of instances, but for
the two-tuple or anonymous product:
instance Functor ((,) a) where
fmap f (x,y) =(x, f y)
instance Monoida
=>Applicative ((,) a) where
pure x=(mempty, x)
(u, f)<*>(v, x)=
(u `mappend` v, f x)
instance Foldable ((,) a) where
foldMap f ( _, y)=f y
foldr f z ( _, y)=f y z
instance Traversable ((,) a) where
traverse f (x, y) =(,) x<$>f y
Here, we have much the same, but for a tuple context.

CHAPTER 21. TRAVERSABLE 1307
21.10Traversable Laws
Thetraverse function must satisfy the following laws:
1.Naturality
t.traverse f =traverse (t .f)
This law tells us that function composition behaves in
unsurprising ways with respect to a traversed function.
Since a traversed function 𝑓is generating the structure
that appears on the “outside” of the traverse operation,
there’s no reason we shouldn’t be able to float a function
over the structure into the traversal itself.
2.Identity
traverse Identity =Identity
This law says that traversing the data constructor of the
Identity type over a value will produce the same result
as just putting the value in Identity . This tells us Identity
represents a structural identity for traversing data. This is
another way of saying that a Traversable instance cannot
add or inject any structure or eﬀects.
3.Composition
traverse (Compose .fmap g.f)=
Compose .fmap (traverse g) .traverse f

CHAPTER 21. TRAVERSABLE 1308
This law demonstrates how we can collapse sequential
traversals into a single traversal, by taking advantage of
theCompose datatype, which combines structure.
ThesequenceA function must satisfy the following laws:
1.Naturality
t.sequenceA =sequenceA .fmap t
2.Identity
sequenceA .fmapIdentity =Identity
3.Composition
sequenceA .fmapCompose =
Compose .fmap sequenceA .sequenceA
Noneofthisshould’vebeentoosurprisinggivenwhatyou’ve
seen with traverse .
21.11 Quality Control
Great news! You can QuickCheck your Traversable instances as
well, since they have laws. Conveniently, the checkers library
we’ve been using already has the laws for us. You can add the
following to a module and change the type alias to change
what instances are being tested:

CHAPTER 21. TRAVERSABLE 1309
typeTI=[]
main= do
lettrigger ::TI(Int,Int, [Int])
trigger =undefined
quickBatch (traversable trigger)
21.12 Chapter Exercises
Traversable instances
Write a Traversable instance for the datatype provided, filling
in any required superclasses. Use QuickCheck to validate your
instances.
Identity
Write a Traversable instance for Identity .
newtype Identity a=Identity a
deriving (Eq,Ord,Show)
instance Traversable Identity where
traverse =undefined

CHAPTER 21. TRAVERSABLE 1310
Constant
newtype Constant a b=
Constant { getConstant ::a }
Maybe
dataOptional a=
Nada
|Yepa
List
dataLista=
Nil
|Consa (Lista)
Three
dataThreea b c=
Threea b c
Pair
dataPaira b=
Paira b

CHAPTER 21. TRAVERSABLE 1311
Big
When you have more than one value of type 𝑏, you’ll want
to useMonoid andApplicative for the Foldable andTraversable
instances respectively.
dataBiga b=
Biga b b
Bigger
Same as for Big.
dataBiggera b=
Biggera b b b
S
This may be difficult. To make it easier, we’ll give you the
constraints and QuickCheck instances:
{-# LANGUAGE FlexibleContexts #-}
moduleSkiFree where
importTest.QuickCheck
importTest.QuickCheck.Checkers

CHAPTER 21. TRAVERSABLE 1312
dataSn a=S(n a) a deriving (Eq,Show)
instance (Functor n
,Arbitrary (n a)
,Arbitrary a )
=>Arbitrary (Sn a)where
arbitrary =
S<$>arbitrary <*>arbitrary
instance (Applicative n
,Testable (nProperty )
,EqPropa )
=>EqProp(Sn a)where
(Sx y)=-=(Sp q)=
(property $(=-=)<$>x<*>p)
.&.(y=-=q)
instance Traversable n
=>Traversable (Sn)where
traverse =undefined
main=
sample' (arbitrary ::Gen(S[]Int))

CHAPTER 21. TRAVERSABLE 1313
Instances for Tree
This might be hard. Write the following instances for Tree.
dataTreea=
Empty
|Leafa
|Node(Treea) a (Treea)
deriving (Eq,Show)
instance Functor Treewhere
fmap=undefined
-- foldMap is a bit easier
-- and looks more natural,
-- but you can do foldr too
-- for extra credit.
instance Foldable Treewhere
foldMap =undefined
instance Traversable Treewhere
traverse =undefined
Hints:
1.ForfoldMap , thinkFunctor but with some Monoid thrown in.

CHAPTER 21. TRAVERSABLE 1314
2.Fortraverse , thinkFunctor but with some Functor3thrown
in.
21.13 Follow-up resources
1.Foldable and Traversable; Jakub Arnold.
2.The Essence of the Iterator Pattern; Jeremy Gibbons and
Bruno Oliveira.
3.Applicative Programming with Eﬀects; Conor McBride
and Ross Paterson.
3Not a typo.

Chapter 22
Reader
The tears of the world are
a constant quantity. For
each one who begins to
weep somewhere else
another stops. The same
is true of the laugh.
Samuel Beckett
1315

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1316
22.1 Reader
The last two chapters were focused on some typeclasses that
might still seem strange and difficult to you. The next three
chapters are going to focus on some patterns that might still
seem strange and difficult. Foldable ,Traversable ,Reader,State,
and parser combinators are not strictly necessary to under-
standing and using Haskell. We do have reasons for introduc-
ing them now, but those reasons might not seem clear to you
for a while. If you don’t quite grasp all of it on the first pass,
that’s completely fine. Read it through, do your best with the
exercises, come back when you feel like you’re ready.
Whenwritingapplications, programmersoftenneedtopass
around some information that may be needed intermittently
or universally throughout an entire application. We don’t want
to simply pass this information as arguments because it would
be present in the type of almost every function. This can make
the code harder to read and harder to maintain. To address
this, we use Reader .
In this chapter, we will:
•examine the Functor ,Applicative , andMonadinstances for
functions;
•learn about the Reader newtype;
•see some examples of using Reader .

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1317
22.2 A new beginning
We’re going to set this chapter up a bit diﬀerently from previ-
ous chapters, because we’re hoping that this will help demon-
strate that what we’re doing here is not that diﬀerent from
things you’ve done before. So, we’re going to start with some
examples. Start a file like this:
importControl.Applicative
boop=(*2)
doop=(+10)
bip::Integer ->Integer
bip=boop.doop
We know that the bipfunction will take one argument be-
cause of the types of boop,doop, and(.). Note that if you do not
specify the types and load it from a file, it will be monomor-
phic by default; if you wish to make bippolymorphic, you
may change its signature but you also need to specify a poly-
morphic type for the two functions it’s built from. The rest of
the chapter will wait while you verify these things.
When we apply bipto an argument, doopwill be applied to
that argument first, and then the result of that will be passed
as input to boop. So far, nothing new.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1318
We can also write that function composition this way:
bloop::Integer ->Integer
bloop=fmap boop doop
We aren’t accustomed to fmapping a function over another
function, and you may be wondering what the functorial con-
text here is. By “functorial context” we mean the structure
(datatype) that the function is being lifted over in order to
apply to the value inside. For example, a list is a functorial
context we can lift functions over. We say that the function gets
lifted over the structure of the list and applied to or mapped
over the values that are inside the list.
Inbloop, the context is a partially applied function. As in
function composition, fmapcomposes the two functions before
applying them to the argument. The result of the one can then
get passed to the next as input. Using fmaphere lifts the one
partially applied function over the next, in a sense setting up
something like this:
fmapboop doop x ==(*2) ((+10) x)
-- when this x comes along, it's the
-- first necessary argument to (+10)
-- then the result for that is the
-- first necessary argument to (*2)

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1319
This is the Functor of functions. We’re going to go into more
detail about this soon.
For now, let’s turn to another set of examples. Put these in
the same file so boopanddoopare still in scope:
bbop::Integer ->Integer
bbop=(+)<$>boop<*>doop
duwop::Integer ->Integer
duwop=liftA2 ( +) boop doop
Now we’re in an Applicative context. We’ve added another
function to lift over the contexts of our partially applied func-
tions. This time, we still have partially applied functions that
are awaiting application to an argument, but this will work
diﬀerently than fmapping did. This time, the argument will
get passed to both boopanddoopin parallel, and the results will
be added together.
boopanddoopare each waiting for an input. We can apply
them both at once like this:
Prelude> bbop 3
19
That does something like this:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1320
((+)<$>(*2)<*>(+10))3
-- First the fmap
(*2)::Numa=>a->a
(+)::Numa=>a->a->a
(+)<$>(*2)::Numa=>a->a->a
Mapping a function awaiting two arguments over a function
awaiting one produces a two argument function.
Remember, this is identical to function composition:
(+).(*2)::Numa=>a->a->a
With the same result:
Prelude> ((+) . (*2)) 5 3
13
Prelude> ((+) <$> (*2)) 5 3
13
So what’s happening?

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1321
((+)<$>(*2))53
-- Keeping in mind that this
-- is (.) under the hood
((+).(*2))53
-- f . g = \ x -> f (g x)
((+).(*2))==\x->(+) (2*x)
The tricky part here is that even after we apply 𝑥, we’ve got
(+)partially applied to the first argument which was doubled
by(*2). There’s a second argument, and that’s what will get
added to the first argument that got doubled:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1322
-- The first function to get
-- applied is (*2), and the
-- first argument is 5. (*2)
-- takes one argument, so we get:
((+).(*2))53
(\x->(+) (2*x))53
(\5->(+) (2*5))3
((+)10)3
-- Then it adds 10 and 3
13
Okay, but what about the second bit?
((+)<$>(*2)<*>(+10))3
-- Wait, what? What happened to the
-- first argument?
((+)<$>(*2)<*>(+10))::Numb=>b->b
One of the nice things about Haskell is we can assert a more
concrete type for functions like (<*>)and see if the compiler
agrees we’re putting forth something hypothetically possible.
Let’s remind ourselves of the type of (<*>):
Prelude> :t (<*>)

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1323
(<*>) :: Applicative f => f (a -> b) -> f a -> f b
-- in this case, we know f is ((->) a)
-- so we concretize it thusly
Prelude> :t (<*>) :: (a -> a -> b) -> (a -> a) -> (a -> b)
(<*>) :: (a -> a -> b) -> (a -> a) -> (a -> b)
The compiler agrees that this is a possible type for (<*>).
So how does that work? What’s happening is we’re feeding
a single argument to the (*2)and(+10)and the two results
form the two arguments to (+):
((+)<$>(*2)<*>(+10))3
(3*2)+(3+10)
6+13
19
We’d use this when two functions would share the same
input and we want to apply some other function to the result
of those to reach a final result. This happens more than you
might think, and we saw an example of it back in the Abstract
Structure Applied chapter:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1324
moduleWeb.Shipping.Utils ((<||>))where
importControl.Applicative (liftA2)
(<||>)::(a->Bool)
->(a->Bool)
->a
->Bool
(<||>)=liftA2 ( ||)
That is the same idea as duwopabove.
Finally, another example:
boopDoop ::Integer ->Integer
boopDoop = do
a<-boop
b<-doop
return (a +b)
This will do precisely the same thing as the Applicative ex-
ample, but this time the context is monadic. This distinction
doesn’t much matter with this particular function. We assign
the variable 𝑎to the partially applied function boop, and𝑏to
doop. As soon as we receive an input, it will fill the empty slots
inboopanddoop. The results will be bound to the variables 𝑎
and𝑏and passed into return .

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1325
So, we’ve seen here that we can have a Functor ,Applicative ,
andMonadfor partially applied functions. In all cases, these
are awaiting application to one argument that will allow both
functions to be evaluated. The Functor of functions is function
composition. The Applicative andMonadchain the argument
forward in addition to the composition (applicatives and mon-
ads are both varieties of functors, so they retain that core
functorial behavior).
This is the idea of Reader. It is a way of stringing functions
together when all those functions are awaiting one input from
a shared environment. We’re going to get into the details of
how it works, but the important intuition here is that it’s an-
other way of abstracting out function application and gives us
a way to do computation in terms of an argument that hasn’t
been supplied yet. We use this most often when we have a con-
stant value that we will obtain from somewhere outside our
program that will be an argument to a whole bunch of func-
tions. Using Reader allows us to avoid passing that argument
around explicitly.
Short Exercise: Warming Up
We’ll be doing something here very similar to what you saw
above, to give you practice and try to develop a feel or intuition
for what is to come. These are similar enough to what you just
saw that you can almost copy and paste, so try not to overthink

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1326
them too much.
First, start a file oﬀ like this:
importData.Char
cap::[Char]->[Char]
capxs=map toUpper xs
rev::[Char]->[Char]
revxs=reverse xs
Two simple functions with the same type, taking the same
type of input. We could compose them, using (.)orfmap:
composed ::[Char]->[Char]
composed =undefined
fmapped ::[Char]->[Char]
fmapped =undefined
The output of those two should be identical: one string that
is made all uppercase and reversed, like this:
Prelude> composed "Julie"
"EILUJ"
Prelude> fmapped "Chris"
"SIRHC"

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1327
Now we want to return the results of capandrevboth, as a
tuple, like this:
Prelude> tupled "Julie"
("JULIE","eiluJ")
-- or
Prelude> tupled' "Julie"
("eiluJ","JULIE")
We will want to use an Applicative here. The type will look
like this:
tupled::[Char]->([Char], [Char])
There is no special reason such a function needs to be
monadic, but let’s do that, too, to get some practice. Do it
one time using dosyntax; then try writing a new version using
(>>=). The types will be the same as the type for tupled .
22.3 This is Reader
As we saw above, functions have Functor ,Applicative , andMonad
instances. Usually when you see or hear the term Reader, it’ll
be referring to the Monadinstance.
We use function composition because it lets us compose two
functions without explicitly having to recognize the argument
that will eventually arrive; the Functor of functions is function

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1328
composition. With the Functor of functions, we are able to map
an ordinary function over another to create a new function
awaiting a final argument. The Applicative andMonadinstances
for the function type give us a way to map a function that is
awaiting an 𝑎over another function that is also awaiting an 𝑎.
Giving it a name helps us know the what and why of what
we’re doing: reading an argument from the environment into
functions. It’ll be especially nice for clarity’s sake later when
we make the ReaderT monad transformer.
Exciting, right? Let’s back up here and go into more detail
about how Reader works.
22.4 Breaking down the Functor of
functions
If you type :info Functor in your REPL, one of the instances
you might notice is the one for the partially applied type
constructor of functions ((->) r) :
instance Functor ((->) r)
This can be a little confusing, so we’re going to unwind it
until hopefully it’s a bit more comfortable. First, let’s see what
we can accomplish with this:
Prelude> fmap (+1) (*2) 3

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1329
7
-- Rearranging a little bit
Prelude> fmap (+1) (*2) $ 3
7
Prelude> (fmap (+1) (*2)) 3
7
This should look familiar:
Prelude> (+1) . (*2) $ 3
7
Prelude> (+2) . (*1) $ 2
4
Prelude> fmap (+2) (*1) $ 2
4
Prelude> (+2) `fmap` (*1) $ 2
4
Fortunately, there’s nothing weird going on here. If you
check the implementation of the instance in base, you’ll find
the following:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1330
instance Functor ((->) r)where
fmap=(.)
Let’s unravel the types. Remember that (->)takes two argu-
ments and therefore has kind * -> * -> * . So, we know upfront
that we have to apply one of the type arguments before we
can have a Functor . With the Either Functor , we know that we
will lift over the Either a and if our function will be applied, it
will be applied to the 𝑏value. With the function type:
data(->) a b
the same rule applies: you have to lift over the (->) a and
only transform the 𝑏value. The 𝑎is conventionally called 𝑟
forReader in these instances, but a type variable of any other
name smells as sweet. Here, 𝑟is the first argument of (a -> b) :

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1331
-- Type constructor of functions
(->)
-- Fully applied
a->b
((->) r)
-- is
r->
-- so r is the type of the
-- argument to the function
From this, we can determine that 𝑟, the argument type for
functions, is part of the structure being lifted over when we lift
over a function, not the value being transformed or mapped
over.
This leaves the result of the function as the value being
transformed. This happens to line up neatly with what func-
tion composition is about:
(.)::(b->c)->(a->b)->a->c
-- or perhaps
(.)::(b->c)->(a->b)->(a->c)
Now how does this line up with Functor ?

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1332
(.)::(b->c)->(a->b)->(a->c)
fmap::Functor f=>(a->b)->f a->f b
We’ll remove the names of the functions and the typeclass
constraint as we can take them for granted from here on out:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1333
::(b->c)->(a->b)->(a->c)
::(a->b)->f a->f b
-- Changing up the letters
-- without changing the meaning
::(b->c)->(a->b)->(a->c)
::(b->c)->f b->f c
-- f is ((->) a)
::(b->c)
-> (a->b)
-> (a->c)
::(b->c)
->((->) a) b
->((->) a) c
-- Unroll the prefix notation into infix
::(b->c)->(a->b)->(a->c)
::(b->c)->(a->b)->(a->c)
Bada bing. Functorial lifting for functions.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1334
22.5 But uh, Reader?
Ah yes, right. Reader is a newtype wrapper for the function
type:
newtype Readerr a=
Reader{ runReader ::r->a }
The𝑟is the type we’re reading in and 𝑎is the result type of
our function.
TheReader newtype has a handy runReader accessor to get
the function out of Reader. Let us prove for ourselves that
this is the same thing, but with a touch of data constructor
jiggery-pokery mixed in. What does the Functor for this look
like compared to function composition?

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1335
instance Functor (Readerr)where
fmap::(a->b)
->Readerr a
->Readerr b
fmap f ( Readerra)=
Reader$\r->f (ra r)
-- same as (.)
compose ::(b->c)->(a->b)->(a->c)
compose f g=\x->f (g x)
-- see it?
\r->f (ra r)
\x->f (g x)
Basically the same thing right? In the Reader functor, rahas
the type r -> a, andfhas the type a -> b. Applying rato the
valueryields a value of type a, which fis then applied to,
yielding a value of type b. Function composition!
We can use the fact that we recognize this as function com-
position to make a slightly diﬀerent instance for Reader :

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1336
instance Functor (Readerr)where
fmap::(a->b)
->Readerr a
->Readerr b
fmap f ( Readerra)=
Reader$(f.ra)
So what we’re doing here is basically:
1.Unpack r -> a out ofReader
2.Compose 𝑓with the function we unpacked out of Reader .
3.Put the new function made from the composition back
intoReader .
Without the Reader newtype, we drop steps 1 and 3 and have
function composition.
Exercise: Ask
Implement the following function. If you get stuck, remem-
ber it’s less complicated than it looks. Write down what you
know. What do you know about the type 𝑎? What does the
type simplify to? How many inhabitants does that type have?
You’ve seen the type before.
ask::Readera a
ask=Reader???

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1337
22.6 Functions have an Applicative too
We’ve seen a couple of examples already of the Applicative of
functions and how it works. Now we’ll get into the details.
The first thing we want to do is notice how the types spe-
cialize:
-- Applicative f =>
-- f ~ (->) r
pure::a->f a
pure::a->(r->a)
(<*>)::f (a->b)
->f a
->f b
(<*>)::(r->a->b)
->(r->a)
->(r->b)
As we saw in the Functor instance, the 𝑟ofReader is part of
the𝑓structure. We have two arguments in this function, and
both of them are functions waiting for the 𝑟input. When that
comes, both functions will be applied to return a final result
of𝑏.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1338
Demonstrating the function Applicative
This example is similar to other demonstrations we’ve done
previously in the book, but this time we’ll be aiming to show
you what specific use the Applicative of functions typically has.
We start with some newtypes for tracking our diﬀerent String
values:
newtype HumanName =
HumanName String
deriving (Eq,Show)
newtype DogName =
DogName String
deriving (Eq,Show)
newtype Address =
Address String
deriving (Eq,Show)
We do this so that our types are more self-explanatory, to
express intent, and so we don’t accidentally mix up our inputs.
A type like this:
String->String->String
is difficult when:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1339
1.They aren’t strictly any string value.
2.They aren’t processed in an identical fashion. You don’t
handle addresses the same as names.
So make the diﬀerence explicit.
We’ll make two record types:
dataPerson=
Person{
humanName ::HumanName
, dogName ::DogName
, address ::Address
}deriving (Eq,Show)
dataDog=
Dog{
dogsName ::DogName
, dogsAddress ::Address
}deriving (Eq,Show)
The following are sample data to use. You can modify them
as you’d like:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1340
pers::Person
pers=
Person(HumanName "Big Bird" )
(DogName "Barkley" )
(Address "Sesame Street" )
chris::Person
chris=Person(HumanName "Chris Allen" )
(DogName "Papu")
(Address "Austin" )
And here is how we’d write it with and without Reader :
-- without Reader
getDog::Person->Dog
getDogp=
Dog(dogName p) (address p)
-- with Reader
getDogR ::Person->Dog
getDogR =
Dog<$>dogName <*>address
Can’t see the Reader ? What if we concrete the types a bit?

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1341
(<$->>)::(a->b)
->(r->a)
->(r->b)
(<$->>)=(<$>)
(<*->>)::(r->a->b)
->(r->a)
->(r->b)
(<*->>)=(<*>)
-- with Reader
getDogR' ::Person->Dog
getDogR' =
Dog<$->>dogName <*->>address
What we’re trying to highlight here is that Reader is not
alwaysReader , sometimes it’s the ambient Applicative orMonad
associated with the partially applied function type, here that
isr ->.
The pattern of using Applicative in this manner is common,
so there’s an alternate way to do this using liftA2 :

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1342
importControl.Applicative (liftA2)
-- with Reader, alternate
getDogR' ::Person->Dog
getDogR' =
liftA2DogdogName address
Here’s the type of liftA2.
liftA2::Applicative f=>
(a->b->c)
->f a->f b->f c
Again, we’re waiting for an input from elsewhere. Rather
than having to thread the argument through our functions,
we elide it and let the types manage it for us.
Exercise: Reading Comprehension
1.WriteliftA2 yourself. Think about it in terms of abstract-
ing out the diﬀerence between getDogR andgetDogR' if that
helps.
myLiftA2 ::Applicative f=>
(a->b->c)
->f a->f b->f c
myLiftA2 =undefined

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1343
2.Write the following function. Again, it is simpler than it
looks.
asks::(r->a)->Readerr a
asksf=Reader???
3.Implement the Applicative forReader .
To write the Applicative instance for Reader, we’ll use an
extension called InstanceSigs . It’s an extension we need
in order to assert a type for the typeclass methods. You
ordinarily cannot assert type signatures in instances. The
compiler already knows the type of the functions, so it’s
not usually necessary to assert the types in instances any-
way. We did this for the sake of clarity, to make the Reader
type explicit in our signatures.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1344
-- you'll need this pragma
{-# LANGUAGE InstanceSigs #-}
instance Applicative (Readerr)where
pure::a->Readerr a
pure a=Reader$ ???
(<*>)::Readerr (a->b)
->Readerr a
->Readerr b
(Readerrab)<*>(Readerra)=
Reader$\r-> ???
Some instructions and hints.
a)When writing the purefunction for Reader , remember
that what you’re trying to construct is a function that
takes a value of type 𝑟, which you know nothing about,
and return a value of type 𝑎. Given that you’re not
really doing anything with 𝑟, there’s really only one
thing you can do.
b)We got the definition of the apply function started for
you, we’ll describe what you need to do and you write
the code. If you unpack the type of Reader’s apply
above, you get the following:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1345
<*> ::(r->a->b)
->(r->a)
->(r->b)
-- contrast this with the type of fmap
fmap::(a->b)
->(r->a)
->(r->b)
So what’s the diﬀerence? The diﬀerence is that apply,
unlikefmap, also takes an argument of type 𝑟.
Make it so.
22.7 The Monadof functions
Functions also have a Monadinstance. You saw this in the be-
ginning of this chapter, and you perhaps have some intuition
now for how this must work. We’re going to walk through a
simplified demonstration of how it works before we get to the
types and instance. Feel free to work through this section as
quickly or slowly as you think appropriate to your own grasp
of what we’ve presented so far.
Let’s start by supposing that we could write a couple of
functions like so:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1346
foo::(Functor f,Numa)=>f a->f a
foor=fmap (+1) r
bar::Foldable f=>t->f a->(t,Int)
barr t=(r, length t)
Now, as it happens in our program, we want to make one
function that will do both — increment the values inside our
structure and also tell us the length of the value. We could
write that like this:
froot::Numa=>[a]->([a],Int)
frootr=(map (+1) r, length r)
Or we could write the same function by combining the
two functions we already had. As it is written above, bartakes
two arguments. We could write a version that takes only one
argument, so that both parts of the tuple apply to the same
argument. That is easy enough to do (notice the change in the
type signature as well):
barOne::Foldable t=>t a->(t a,Int)
barOner=(r, length r)
That gave us the reduction to one argument that we wanted
but didn’t increment the values in the list as our foofunction
does. We can add that this way:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1347
barPlus r=(foo r, length r)
But we can also do that more compactly by making (foo r)
the first argument to bar:
frooty::Numa=>[a]->([a],Int)
frootyr=bar (foo r) r
Now we have an environment in which two functions are
waiting for the same argument to come in. They’ll both apply
to that argument in order to produce a final result.
Let’s make a small change to make it look a little more
Reader -y:
frooty' ::Numa=>[a]->([a],Int)
frooty' =\r->bar (foo r) r
Then we abstract this out so that it’s not specific to these
functions:
fooBind m k=\r->k (m r) r
In this very polymorphic version, the type signature will
look like this:
fooBind ::(t2->t1)
->(t1->t2->t)
->t2
->t

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1348
So many 𝑡types! That’s because we can’t know very much
about those types once our function is that abstract. We can
make it a little more clear by making some substitutions. We’ll
use the 𝑟to represent the argument that both of our functions
are waiting on — the Reader -y part:
fooBind ::(r->a)
->(a->r->b)
->(r->b)
If we could take the 𝑟parts out, we might notice that fooBind
itself looks like a very abstract and simplified version of some-
thing we’ve seen before (overparenthesizing a bit, for clarity):
(>>=) :: Monad m =>
m a -> (a -> (m b)) -> m b
(r -> a) -> (a -> (r -> b)) -> (r -> b)
This is how we get to the Monadof functions. Just as with the
Functor andApplicative instances, the ((->) r) is our structure
— the𝑚in the type of (>>=). In the next section, we’ll work
forward from the types.
TheMonadinstance
As we noted, the 𝑟argument remains part of our (monadic)
structure:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1349
(>>=)::Monadm
=>m a->(a->m b) -> m b
(>>=)::
(->) r a->(a->(->) r b)->(->) r b
(>>=)::
(r->a)->(a->r->b)->r->b
return::Monadm=>a-> m a
return:: a->(->) r a
return:: a->r->a
You may notice that return looks like a function we’ve seen
a lot of in this book.
Let’s look at it side by side with the Applicative :
(<*>)::(r->a->b)
->(r->a)
->(r->b)
(>>=)::(r->a)
->(a->r->b)
->(r->b)
Or with the flipped bind:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1350
(<*>)::(r->a->b)
->(r->a)
->(r->b)
(=<<)::(a->r->b)
->(r->a)
->(r->b)
So you’ve got this ever-present type 𝑟following your func-
tions around like a lonely puppy.
Example uses of the Reader type
Remember the earlier example with Person andDog? Here’s the
same but with the Reader Monad anddosyntax:
-- with Reader Monad
getDogRM ::Person->Dog
getDogRM = do
name<-dogName
addy<-address
return$Dogname addy
Exercise: Reader Monad
1.Implement the Reader Monad .

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1351
-- Don't forget instancesigs.
instance Monad(Readerr)where
return=pure
(>>=)::Readerr a
->(a->Readerr b)
->Readerr b
(Readerra)>>=aRb=
Reader$\r-> ???
Hint: constrast the type with the Applicative instance and
perform the most obvious change you can imagine to
make it work.
2.Rewrite the monadic getDogRM to use your Reader datatype.
22.8Reader Monad by itself is boring
It can’t do anything the Applicative cannot.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1352
{-# LANGUAGE NoImplicitPrelude #-}
modulePrettyReader where
flip::(a->b->c)->(b->a->c)
flipf a b=f b a
const::a->b->a
consta b=a
(.)::(b->c)->(a->b)->(a->c)
f.g=\a->f (g a)
classFunctor fwhere
fmap::(a->b)->f a->f b
classFunctor f=>Applicative fwhere
pure::a->f a
(<*>)::f (a->b)->f a->f b
classApplicative f=>Monadfwhere
return::a->f a
(>>=)::f a->(a->f b)->f b

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1353
instance Functor ((->) r)where
fmap=(.)
instance Applicative ((->) r)where
pure=const
f<*>a=\r->f r (a r)
instance Monad((->) r)where
return=pure
m>>=k=flip k<*>m
Speaking generally in terms of the algebras alone, you can-
not get a Monadinstance from the Applicative . You can get
anApplicative from the Monad. However, our instances above
aren’t in terms of an abstract datatype; we know it’s the type
of functions. Because it’s not hiding behind a Reader newtype,
we can use flipandapplyto make the Monadinstance. We need
specific type information to augment what the Applicative is
capable of before we can get our Monadinstance.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1354
22.9 You can change what comes below,
but not above
The “read-only” nature of the type argument 𝑟means that you
can swap in a diﬀerent type or value of 𝑟for functions that
you call, but not for functions that call you. The best way to
demonstrate this is with the withReaderT function which lets
us start a new Reader context with a diﬀerent argument being
provided:
withReaderT
::(r'->r)
-- ^ The function to modify
-- the environment.
->ReaderT r m a
-- ^ Computation to run in the
-- modified environment.
->ReaderT r' m a
withReaderT f m=
ReaderT $runReaderT m .f
In the next chapter, we’ll see the Statemonad where we can
not only read in a value, but provide a new one which will
change the value carried by the functions that called us, not
only those we called.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1355
22.10 You tend to see ReaderT , notReader
Reader rarely stands alone. Usually it’s one Monadin a stack of
multiple types providing a Monadinstance such as with a web
application that uses Reader to give you access to context about
the HTTP request. When used in that fashion, it’s a monad
transformer and we put a letter T after the type to indicate
when we’re using it as such, so you’ll usually see ReaderT in
production Haskell code rather than Reader .
Further, a Reader ofIntisn’t all that useful or compelling.
Usually if you have a Reader , it’s of a record of several (possibly
many) values that you’re getting out of the Reader .
22.11 Chapter Exercises
A warm-up stretch
These exercises are designed to be a warm-up and get you
using some of the stuﬀ we’ve learned in the last few chap-
ters. While these exercises comprise code fragments from
real code, they are simplified in order to be discrete exercises.
That will allow us to highlight and practice some of the type
manipulation from Traversable andReader, both of which are
tricky.
The first simplified part is that we’re going to set up some
toy data; in the real programs these are taken from, the data

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1356
is coming from somewhere else — a database, for example.
We just need some lists of numbers. We’re going to use some
functions from Control.Applicative andData.Maybe , so we’ll im-
port those at the top of our practice file. We’ll call our lists of
toy data by common variable names for simplicity.
moduleReaderPractice where
importControl.Applicative
importData.Maybe
x=[1,2,3]
y=[4,5,6]
z=[7,8,9]
The next thing we want to do is write some functions that
zip those lists together and use lookup to find the value associ-
ated with a specified key in our zipped lists. For demonstration
purposes, it’s nice to have the outputs be predictable, so we
recommend writing some that are concrete values, as well as
one that can be applied to a variable:
lookup::Eqa=>a->[(a, b)] ->Maybeb
-- zip x and y using 3 as the lookup key
xs::MaybeInteger
xs=undefined

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1357
-- zip y and z using 6 as the lookup key
ys::MaybeInteger
ys=undefined
-- it's also nice to have one that
-- will return Nothing, like this one
-- zip x and y using 4 as the lookup key
zs::MaybeInteger
zs=lookup4$zip x y
-- now zip x and z using a
-- variable lookup key
z'::Integer ->MaybeInteger
z'n=undefined
Now we want to add the ability to make a Maybe (,) of values
usingApplicative . Have x1make a tuple of xsandys, andx2
make a tuple of of ysandzs. Also, write x3which takes one
input and makes a tuple of the results of two applications of
z'from above.

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1358
x1::Maybe(Integer,Integer)
x1=undefined
x2::Maybe(Integer,Integer)
x2=undefined
x3::Integer
->(MaybeInteger,MaybeInteger)
x3=undefined
Your outputs from those should look like this:
*ReaderPractice> x1
Just (6,9)
*ReaderPractice> x2
Nothing
*ReaderPractice> x3 3
(Just 9,Just 9)
Next, we’re going to make some helper functions. Let’s use
uncurry to allow us to add the two values that are inside a tuple:

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1359
uncurry ::(a->b->c)->(a, b)->c
-- that first argument is a function
-- in this case, we want it to be addition
-- summed is uncurry with addition as
-- the first argument
summed::Numc=>(c, c)->c
summed=undefined
And now we’ll make a function similar to some we’ve seen
before that lifts a boolean function over two partially applied
functions:
bolt::Integer ->Bool
-- use &&, >3, <8
bolt=undefined
Finally, we’ll be using fromMaybe in themainexercise, so let’s
look at that:
fromMaybe ::a->Maybea->a
You give it a default value and a Maybevalue. If the Maybe
value is a Just a, it will return the 𝑎value. If the value is a
Nothing , it returns the default value instead:
*ReaderPractice> fromMaybe 0 xs

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1360
6
*ReaderPractice> fromMaybe 0 zs
0
Now we’ll cobble together a main, so that in one call we can
execute several things at once.
main::IO()
main= do
print$
sequenceA [ Just3,Just2,Just1]
print$sequenceA [x, y]
print$sequenceA [xs, ys]
print$summed<$>((,)<$>xs<*>ys)
print$fmap summed ((,) <$>xs<*>zs)
print$bolt7
print$fmap bolt z
When you run this in GHCi, your results should look like
this:
*ReaderPractice> main
Just [3,2,1]
[[1,4],[1,5],[1,6],[2,4],[2,5],[2,6],[3,4],[3,5],[3,6]]
Just [6,9]
Just 15
Nothing

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1361
True
[True,False,False]
Next, we’re going to add one that combines sequenceA and
Reader in a somewhat surprising way (add this to main):
print$sequenceA [( >3), (<8), even] 7
The type of sequenceA is
sequenceA ::(Applicative f,Traversable t)
=>t (f a) ->f (t a)
-- so in this:
sequenceA [(>3), (<8), even] 7
-- f ~ (->) a and t ~ []
Wehavea Reader fortheApplicative (functions)andatraversable
for the list. Pretty handy. We’re going to call that function
sequAfor the purposes of the following exercises:
sequA::Integral a=>a->[Bool]
sequAm=sequenceA [( >3), (<8), even] m
And henceforth let
summed<$>((,)<$>xs<*>ys)

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1362
be known as s'.
OK, your turn. Within the mainabove, write the following
(you can delete everything after donow if you prefer — just
remember to use printto be able to print the results of what
you’re adding):
1.fold the boolean conjunction operator over the list of
results of sequA(applied to some value).
2.applysequAtos'; you’ll need fromMaybe .
3.applybolttoys; you’ll need fromMaybe .
Rewriting Shawty
Remember the URL shortener? Instead of manually passing
the database connection rConnfrommainto the app function
that generates a Scotty app, use ReaderT to make the database
connection available. We know you haven’t seen the trans-
former variant yet and we’ll explain them soon, but you should
try to do the transformation mechanically. Research as neces-
sary using a search engine. Use this version of the app: https:
//github.com/bitemyapp/shawty-prime/blob/master/app/Main.hs
22.12 Definition
A monad transformer is a special type that takes a monad as
an argument and returns a monad as a result. It allows us to

CHAPTER 22. FUNCTIONS WAITING FOR INPUT 1363
combine two monads into one that shares the behaviors of
both, such as allowing us to add exception handling to a State
monad. It is somewhat common to create a stack of transform-
ers to create one large monad that has features from several
monads, for example, rolling Reader,Either, andIOtogether
to get a monad that captures the behavior of waiting for an
argument that will get passed around to multiple functions
but is likely to come in via some kind of I/O action and has the
possibility of failure we might like to catch. Often this stack
will be given a type alias for convenience.
22.13 Follow-up resources
1.Reader Monad; All About Monads
https://wiki.haskell.org/All_About_Monads
2.Reader Monad; Programming with Monads; Real World
Haskell

Chapter 23
State
Four centuries ago,
Descartes pondered the
mind-body problem:
how can incorporeal
minds interact with
physical bodies? Today,
computing scientists face
their own version of the
mind-body problem:
how can virtual software
interact with the real
world?
Philip Wadler
1364

CHAPTER 23. STATE 1365
23.1 State
What if I need state? In Haskell we have many means of repre-
senting, accessing, and modifying state. We can think of state
as data that exists in addition to the inputs and outputs of our
functions, data that can potentially change after each function
is evaluated.
In this chapter, we will:
•talk about what state means;
•explore some ways of handling state in Haskell;
•generate some more random numbers;
•and examine the Statenewtype and Monadinstance.
23.2 What is state?
The concept of state originates in the circuit and automata
theory that much of computer science and programming be-
gan with. The simplest form of state could be understood as a
light switch. A light switch has two possible states, on or oﬀ.
That disposition of the light switch, being on or oﬀ, could be
understood as its state. Similarly, transistors in computers
have binary states of being on or oﬀ. This is a very low-level
way of seeing it, but this maps onto the state that exists in
computer memory.

CHAPTER 23. STATE 1366
In most imperative programming languages, this stateful-
ness is pervasive, implicit, and not referenced in the types
of your functions. In Haskell, we’re not allowed to secretly
change some value; all we can do is accept arguments and
return a result. The Statetype in Haskell is a means of express-
ing state that may change in the course of evaluating code
without resort to mutation. The monadic interface for State
is, much as you’ve seen already, more of a convenience than a
strict necessity for working with State.
We have the option to capture the idea and convenience
of a value which potentially changes with each computation
without resorting to mutability. Statecaptures this idea and
cleans up the bookkeeping required. If you need in-place
mutation, then the STtype is what you want, and we address
that briefly in later chapters.
In Haskell, if we use the Statetype and its associated Monad
(for convenience, not strictly necessary), we can have state
which:
1.doesn’t require IO;
2.is limited only to the data in our Statecontainer;
3.maintains referential transparency;
4.is explicit in the types of our functions.

CHAPTER 23. STATE 1367
There are other means of sharing data within a program
that are designed for diﬀerent needs than the Statedatatype
itself.Stateis appropriate when you want to express your
program in terms of values that potentially vary with each
evaluation step, which can be read and modified, but don’t
otherwise have specific operational constraints.
23.3 Random numbers
As we did in the previous chapter, we’ll start with an extended
example. This will help you get an idea of the problem we’re
trying to solve with the Statedatatype.
We’ll be using the random1library, version 1.1, in this ex-
ample.
First, let’s give an overview of some of the functions we’ll
be using here. We used the System.Random library back in the
chapter where we built the hangman game, but we’ll be using
some diﬀerent functions for this example. This is in broad
strokes; it isn’t meant to go into great detail about how these
generators work.
System.Random is designed to generate pseudorandom values.
You can generate those values through providing a seed value
or by using the system-initialised generator. We’ll be using
the following from that library:
1https://hackage.haskell.org/package/random

CHAPTER 23. STATE 1368
1.One of the types we’ll be seeing here, StdGen , is a datatype
that is a product of two Int32values. So a value of type
StdGen always comprises two Int32values. They are the
seed values used to generate the next random number.
2.mkStdGen has the type:
mkStdGen ::Int->StdGen
We’ll ignore the implementation at this point because
those details aren’t important here. The idea is that it takes
anIntargument and maps it into a generator to return a
value of type StdGen , which is a pair of Int32values.
3.nexthas the type:
next::g->(Int, g)
where𝑔is a value of type StdGen . TheIntthat is first in the
tuple is the pseudorandom number generated from the
StdGen value; the second value is a new StdGen value.
4.random has the type:
random::(RandomGen g,Randoma)
=>g->(a, g)
This is similar to nextbut allows us to generate random
values that aren’t numbers. The range generated will be
determined by the type.

CHAPTER 23. STATE 1369
Now, let’s have a little demonstration of these:
Prelude> import System.Random
Prelude> mkStdGen 0
1 1
Prelude> :t mkStdGen 0
mkStdGen 0 :: StdGen
Prelude> let sg = mkStdGen 0
Prelude> :t next sg
next sg :: (Int, StdGen)
Prelude> next sg
(2147482884,40014 40692)
Prelude> next sg
(2147482884,40014 40692)
We get the same answer twice because the underlying func-
tionthat’sdecidingthevaluesreturnedispure; thetypedoesn’t
permit the performance of any eﬀects to get spooky action.
Define a new version of sgthat provides a diﬀerent input value
tomkStdGen and see what happens.
So, we have a value called next sg . Now, if we want to use
that to generate the next random number, we need to feed the
StdGen value from that tuple to nextagain. We can use sndto
extract that StdGen value and pass it as an input to next:
Prelude> snd (next sg)
40014 40692

CHAPTER 23. STATE 1370
Prelude> let newSg = snd (next sg)
Prelude> :t newSg
newSg :: StdGen
Prelude> next newSg
(2092764894,1601120196 1655838864)
You’ll keep getting the same results of nextthere, but you
can extract that StdGen value and pass it to nextagain to get a
new tuple:
Prelude> next (snd (next newSg))
(1679949200,1635875901 2103410263)
Now we’ll look at a few examples using random. Because
random can generate values of diﬀerent types, we need to specify
the type to use:
Prelude> :t random newSg
random newSg :: Random a => (a, StdGen)
Prelude> random newSg :: (Int, StdGen)
(138890298504988632,439883729 1872071452)
Prelude> random newSg :: (Double, StdGen)
(0.41992072972993366,439883729 1872071452)
Simple enough, but what if we want a number within a
range?

CHAPTER 23. STATE 1371
Prelude> :t randomR
randomR :: (RandomGen g, Random a) => (a, a) -> g -> (a, g)
Prelude> randomR (0, 3) newSg :: (Int, StdGen)
(1,1601120196 1655838864)
Prelude> randomR (0, 3) newSg :: (Double, StdGen)
(1.259762189189801,439883729 1872071452)
We have to pass the new state of the random number gen-
erator to the nextfunction to get a new value:
Prelude> let rx :: (Int, StdGen); rx = random (snd sg3)
Prelude> rx
(2387576047905147892,1038587761 535353314)
Prelude> snd rx
1038587761 535353314
This chaining of state can get tedious. Addressing this te-
dium is our aim in this chapter.
23.4 The Statenewtype
Stateis defined in a newtype, like Reader in the previous chap-
ter, and that type looks like this:
newtype States a=
State{ runState ::s->(a, s) }

CHAPTER 23. STATE 1372
It’s initially a bit strange looking, but you might notice some
similarity to the Reader newtype:
newtype Readerr a=
Reader{ runReader ::r->a }
We’ve seen several newtypes whose contents are a function,
particularly with our Monoid newtypes ( Sum,Product , etc.). New-
types must have the same underlying representation as the
type they wrap, as the newtype wrapper disappears at compile
time. So the function contained in the newtype must be iso-
morphic to the type it wraps. That is, there must be a way to go
from the newtype to the thing it wraps and back again without
losing information. For example, the following demonstrates
an isomorphism:
typeIsoa b=(a->b, b->a)
newtype Suma=Sum{ getSum ::a }
sumIsIsomorphicWithItsContents
::Isoa (Suma)
sumIsIsomorphicWithItsContents =
(Sum, getSum)
Whereas the following do not:

CHAPTER 23. STATE 1373
-- Not an isomorphism, because
-- it might not work.
(a->Maybeb, b->Maybea)
-- Not an isomorphism for two reasons.
-- You lose information whenever there
-- was more than one element in [a]. Also,
-- [a] -> a is partial because there
-- might not be any elements.
[a]->a, a->[a]
With that in mind, let us look at the Statedata constructor
andrunState record accessor as our means of putting a value
in and taking a value out of the Statetype:
State::(s->(a, s)) ->States a
runState ::States a->s->(a, s)
Stateis a function that takes input state and returns an out-
put value, 𝑎, tupled with the new state value. The key is that
the previous state value from each application is chained to
the next one, and this is not an uncommon pattern. Stateis
often used for things like random number generators, solvers,
games, and carrying working memory while traversing a data
structure. The polymorphism means you don’t have to make
a new state for each possible instantiation of 𝑠and𝑎.

CHAPTER 23. STATE 1374
Let’s get back to our random numbers:
Note that random looks an awful lot like Statehere:
random::(Randoma)
=>StdGen->(a,StdGen)
State{ runState
::s->(a, s) }
If we look at the type of randomR , once partially applied, it
should also remind you of State:
randomR ::(...)=>(a, a)->g->(a, g)
State{ runState :: s->(a, s) }
23.5 Throw down
Now let us use this kit to generate die such as for a game:

CHAPTER 23. STATE 1375
moduleRandomExample where
importSystem.Random
-- Six-sided die
dataDie=
DieOne
|DieTwo
|DieThree
|DieFour
|DieFive
|DieSix
deriving (Eq,Show)
As you might expect, we’ll be using the random library, and a
simpleDiedatatype to represent a six-sided die.

CHAPTER 23. STATE 1376
intToDie ::Int->Die
intToDie n=
casenof
1->DieOne
2->DieTwo
3->DieThree
4->DieFour
5->DieFive
6->DieSix
-- Use 'error'
-- _extremely_ sparingly.
x->
error$
"intToDie got non 1-6 integer: "
++show x
Don’t use erroroutside of experiments like this, or in cases
where the branch you’re ignoring is provably impossible. We
do not use the word provably here lightly.2
Now we need to roll the dice:
2Because partial functions are a pain, you should only use an error like this when
the branch that would spawn the error can literally never happen. Unexpected software
failures are often due to things like this. It is also completely unnecessary in Haskell; we
have good alternatives, like using MaybeorEither. The only reason we didn’t here is to
keep it simple and focus attention on the State Monad .

CHAPTER 23. STATE 1377
rollDieThreeTimes ::(Die,Die,Die)
rollDieThreeTimes = do
lets=mkStdGen 0
(d1, s1) =randomR ( 1,6) s
(d2, s2) =randomR ( 1,6) s1
(d3,_)=randomR ( 1,6) s2
(intToDie d1, intToDie d2, intToDie d3)
This code isn’t optimal, but it does work. It will produce
the same results every time, because it is free of eﬀects, but
you can make it produce a new result on a new dice roll if you
modify the start value. Try it a couple of times to see what we
mean. It seems unlikely that this will develop into a gambling
addiction, but in the event it does, the authors disclaim liability
for such.
So, how can we improve our suboptimal code there. With
State, of course!
moduleRandomExample2 where
importControl.Applicative (liftA3)
importControl.Monad (replicateM )
importControl.Monad.Trans.State
importSystem.Random
importRandomExample

CHAPTER 23. STATE 1378
First, we’ll add some new imports. You’ll need transformers
to be installed for the Stateimport to work, but that should
have come with your GHC install, so you should be good to
go.
UsingStatewill allow us to factor out the generation of a
singleDie:
rollDie ::StateStdGenDie
rollDie =state$ do
(n, s)<-randomR ( 1,6)
return (intToDie n, s)
For our purposes, the statefunction is a constructor that
takes aState-like function and embeds it in the Statemonad
transformer. Ignore the transformer part for now — we’ll get
there. The state function has the following type:
state::Monadm
=>(s->(a, s))
->StateTs m a
Note that we’re binding the result of randomR out of the State
monad the doblock is in rather than using let. This is still more
verbose than is necessary. We can lift our intToDie function
over the State:

CHAPTER 23. STATE 1379
rollDie' ::StateStdGenDie
rollDie' =
intToDie <$>state (randomR ( 1,6))
State StdGen had a final type argument of Int. We lifted Int
-> Die over it and transformed that final type argument to Die.
We’ll exercise more brevity upfront in the next function:
rollDieThreeTimes'
::StateStdGen(Die,Die,Die)
rollDieThreeTimes' =
liftA3 (,,) rollDie rollDie rollDie
Lifting the three-tuple constructor over three Stateactions
that produce Dievalues when given an initial state to work
with. How does this look in practice?
Prelude> evalState rollDieThreeTimes' (mkStdGen 0)
(DieSix,DieSix,DieFour)
Prelude> evalState rollDieThreeTimes' (mkStdGen 1)
(DieSix,DieFive,DieTwo)
Seems to work fine. Again, the same inputs give us the same
result. What if we want a list of Dieinstead of a tuple?

CHAPTER 23. STATE 1380
-- Seems appropriate?
repeat::a->[a]
infiniteDie ::StateStdGen[Die]
infiniteDie =repeat<$>rollDie
Does this infiniteDie function do what we want or expect?
What is it repeating?
Prelude> take 6 $ evalState infiniteDie (mkStdGen 0)
[DieSix,DieSix,DieSix,DieSix,DieSix,DieSix]
We already know based on previous inputs that the first 3
values shouldn’t be identical for a seed value of 0. So what
happened? What happened is we repeated a single die value
— we didn’t repeat the state action that produces a die. This is
what we need:
replicateM ::Monadm
=>Int->m a->m [a]
nDie::Int->StateStdGen[Die]
nDien=replicateM n rollDie
And when we use it?
Prelude> evalState (nDie 5) (mkStdGen 0)

CHAPTER 23. STATE 1381
[DieSix,DieSix,DieFour,DieOne,DieFive]
Prelude> evalState (nDie 5) (mkStdGen 1)
[DieSix,DieFive,DieTwo,DieSix,DieFive]
We get precisely what we wanted.
Keep on rolling
In the following example, we keep rolling a single die until we
reach or exceed a sum of 20.
rollsToGetTwenty ::StdGen->Int
rollsToGetTwenty g=go00g
where
go::Int->Int->StdGen->Int
go sum count gen
|sum>=20=count
|otherwise =
let(die, nextGen) =
randomR ( 1,6) gen
ingo (sum +die)
(count+1) nextGen
Then seeing it in action:
Prelude> rollsToGetTwenty (mkStdGen 0)
5

CHAPTER 23. STATE 1382
Prelude> rollsToGetTwenty (mkStdGen 0)
5
We can also use randomIO , which uses IOto get a new value
each time without needing to create a unique value for the
StdGen :
Prelude> :t randomIO
randomIO :: Random a => IO a
Prelude> (rollsToGetTwenty . mkStdGen) <$> randomIO
6
Prelude> (rollsToGetTwenty . mkStdGen) <$> randomIO
7
Under the hood, it’s the same interface and State Monad
driven mechanism, but it’s mutating a single globally used
StdGen to walk the generator forward on each use. See the
random library source code to see how this works.
Exercises: Roll Your Own
1.Refactor rollsToGetTwenty into having the limit be a func-
tion argument.
rollsToGetN ::Int->StdGen->Int
rollsToGetN =undefined
2.Change rollsToGetN to recording the series of die that oc-
curred in addition to the count.

CHAPTER 23. STATE 1383
rollsCountLogged ::Int
->StdGen
->(Int, [Die])
rollsCountLogged =undefined
23.6 Write Statefor yourself
Use the datatype definition from the beginning of this chapter,
with the name changed to avoid conflicts in case you have
Stateimported from the libraries transformers ormtl. We’re
calling it Moi, because we enjoy allusions to famous quotations3;
feel free to change the name if you wish to protest absolute
monarchy, but change them consistently throughout.
newtype Mois a=
Moi{ runMoi ::s->(a, s) }
State Functor
Implement the Functor instance for State.
instance Functor (Mois)where
fmap::(a->b)->Mois a->Mois b
fmap f ( Moig)= ???
3We are referring to the (possibly apocryphal) quotation attributed to the French
King Louis XIV, “L’Etat, c’est moi.” For those of you who do not speak French, it means,
“I am the State.” Cheers.

CHAPTER 23. STATE 1384
Prelude> runMoi ((+1) <$> (Moi $ \s -> (0, s))) 0
(1,0)
State Applicative
Write the Applicative instance for State.
instance Applicative (Mois)where
pure::a->Mois a
pure a= ???
(<*>)::Mois (a->b)
->Mois a
->Mois b
(Moif)<*>(Moig)=
???
State Monad
Write the Monadinstance for State.

CHAPTER 23. STATE 1385
instance Monad(Mois)where
return=pure
(>>=)::Mois a
->(a->Mois b)
->Mois b
(Moif)>>=g=
???
23.7 Get a coding job with one weird
trick
Some companies will use FizzBuzz4to screen (not so much
test) candidates applying to software positions. The problem
statement goes:
Write a program that prints the numbers from 1 to
100. But for multiples of three print “Fizz” instead of
the number and for the multiples of five print “Buzz”.
For numbers which are multiples of both three and
five print “FizzBuzz”.
A typical fizzbuzz solution in Haskell looks something like:
4http://c2.com/cgi/wiki?FizzBuzzTest

CHAPTER 23. STATE 1386
fizzBuzz ::Integer ->String
fizzBuzz n|n `mod` 15==0="FizzBuzz"
|n `mod` 5==0="Buzz"
|n `mod` 3==0="Fizz"
|otherwise =show n
main::IO()
main=
mapM_ (putStrLn .fizzBuzz) [ 1..100]
A fizzbuzz using Stateis a suitable punishment for asking
a software candidate to write this in person after presumably
getting through a couple phone screens. Let’s look at what a
version with Statemight look like:
importControl.Monad
importControl.Monad.Trans.State
fizzBuzz ::Integer ->String
fizzBuzz n|n `mod` 15==0="FizzBuzz"
|n `mod` 5==0="Buzz"
|n `mod` 3==0="Fizz"
|otherwise =show n

CHAPTER 23. STATE 1387
fizzbuzzList ::[Integer]->[String]
fizzbuzzList list=
execState (mapM_ addResult list) []
addResult ::Integer ->State[String]()
addResult n= do
xs<-get
letresult=fizzBuzz n
put (result :xs)
Note that Stateis a type alias of StateT you imported.
main::IO()
main=
mapM_ putStrLn $
reverse $fizzbuzzList [ 1..100]
The good part here is that we’re collecting data initially
before dumping the results to standard output via putStrLn .
The bad is that we’re reversing a list. Reversing singly-linked
lists is not great, even in Haskell, and won’t terminate on an
infinite list. One of the issues is that we’re accepting an input
that defines the numbers we’ll use fizzbuzz on linearly from
beginning to end.
There are a couple ways we could handle this. One is to
use a data structure with cheaper appending to the end. Using
(++)recursively can be very slow, so let’s use something that

CHAPTER 23. STATE 1388
can append in constant time. The counterpart to []which has
this property is the diﬀerence list5which has O(1) append.
importControl.Monad
importControl.Monad.Trans.State
-- http://hackage.haskell.org/package/dlist
import qualified Data.DList asDL
fizzBuzz ::Integer ->String
fizzBuzz n|n `mod` 15==0="FizzBuzz"
|n `mod` 5==0="Buzz"
|n `mod` 3==0="Fizz"
|otherwise =show n
5https://github.com/spl/dlist

CHAPTER 23. STATE 1389
fizzbuzzList ::[Integer]->[String]
fizzbuzzList list=
letdlist=
execState (mapM_ addResult list)
DL.empty
-- convert back to normal list
inDL.apply dlist []
addResult ::Integer
->State(DL.DListString)()
addResult n= do
xs<-get
letresult=fizzBuzz n
-- snoc appends to the end, unlike
-- cons which adds to the front
put (DL.snoc xs result)
main::IO()
main=
mapM_ putStrLn $fizzbuzzList [ 1..100]
We can clean this up further. If you have GHC 7.10 or newer,
mapM_will specify a Foldable type, not only a list:
Prelude> :t mapM_
mapM_ :: (Monad m, Foldable t) => (a -> m b) -> t a -> m ()

CHAPTER 23. STATE 1390
By letting DList’sFoldable instance do the conversion to a
list for us, we can eliminate some code:
fizzbuzzList ::[Integer]
->DL.DListString
fizzbuzzList list=
execState (mapM_ addResult list) DL.empty
addResult ::Integer
->State(DL.DListString)()
addResult n= do
xs<-get
letresult=fizzBuzz n
put (DL.snoc xs result)
main::IO()
main=
mapM_ putStrLn $fizzbuzzList [ 1..100]
DList’sFoldable instance converts to a list before folding
because of limitations specific to the datatype. You get cheap
appending, but you give up the ability to “see” what you’ve
built unless you’re willing to do all the work of building the
structure. We’ll discuss this in more detail in a forthcoming
chapter.

CHAPTER 23. STATE 1391
One thing that may strike you here is that the use of State
was superfluous. That’s good! It’s not common you need State
as such in Haskell. You might use a diﬀerent form of State
calledSTas a selective optimization, but Stateitself is a stylistic
choice that falls out of what the code is telling you. Don’t
feel compelled to use or not use State. Please frighten some
interviewers with a spooky fizzbuzz. Make something even
weirder than what we’ve shown you here!
Fizzbuzz Diﬀerently
It’s an exercise! Rather than changing the underlying data
structure, fix our reversing fizzbuzz by changing the code in
the following way:
fizzbuzzFromTo ::Integer
->Integer
->[String]
fizzbuzzFromTo =undefined
Continue to use consing in the construction of the result
list, but have it come out in the right order to begin with by
enumerating the sequence backwards. This sort of tactic is
more commonly how you’ll want to fix your code when you’re
quashing unnecessary reversals.

CHAPTER 23. STATE 1392
23.8 Chapter exercises
Write the following functions. You’ll want to use your own
Statetype for which you’ve defined the Functor ,Applicative ,
andMonad.
1.Construct a Statewhere the state is also the value you
return.
get::States s
get= ???
Expected output
Prelude> runState get "curryIsAmaze"
("curryIsAmaze","curryIsAmaze")
2.Construct a Statewhere the resulting state is the argument
provided and the value is defaulted to unit.
put::s->States()
puts= ???
Prelude> runState (put "blah") "woot"
((),"blah")
3.Run the Statewith𝑠and get the state that results.

CHAPTER 23. STATE 1393
exec::States a->s->s
exec(Statesa) s= ???
Prelude> exec (put "wilma") "daphne"
"wilma"
Prelude> exec get "scooby papu"
"scooby papu"
4.Run the Statewith𝑠and get the value that results.
eval::States a->s->a
eval(Statesa)= ???
Prelude> eval get "bunnicula"
"bunnicula"
Prelude> eval get "stake a bunny"
"stake a bunny"
5.Write a function which applies a function to create a new
State.
modify::(s->s)->States()
modify=undefined
Should behave like the following:

CHAPTER 23. STATE 1394
Prelude> runState (modify (+1)) 0
((),1)
Prelude> runState (modify (+1) >> modify (+1)) 0
((),2)
You don’t need to compose them, you can throw away the
result because it returns unit for 𝑎anyway.
23.9 Follow-up resources
1.State Monad; All About Monads; Haskell Wiki
https://wiki.haskell.org/All_About_Monads
2.State Monad; Haskell Wiki
https://wiki.haskell.org/State_Monad
3.Understanding Monads; Haskell Wikibook

Chapter 24
Parser combinators
Within a computer,
natural language is
unnatural.
Alan Perlis
1395

CHAPTER 24. PARSER COMBINATORS 1396
24.1 Parser combinators
The word parse comes from the Latin word for parts, and
means to analyze a sentence and label the syntactic role, or part
of speech, of each component. Language teachers once em-
phasized this ability because it forced students to think about
the structure of sentences, the relationships among the parts,
and the connection between the structure and the meaning of
the whole. Diagramming sentences was common because it
made parsing visual and somewhat concrete.
It is now common to represent grammatical structures of
natural languages as trees, so that a sentence such as
Boy plays with dog.
might be thought to have an underlying representation
such as
S(entence)
/ \
Boy plays (verb)
(subject) \
with (preposition)
\
dog (object)
We are not here to become linguists, but parsing in com-
puter science is related to the parsing of natural language

CHAPTER 24. PARSER COMBINATORS 1397
sentences. The core idea of parsing in programming is to
accept serialized input in the form of a sequence of characters
(textual data) or bytes (raw binary data) and turn that into a
value of a structured datatype. Serialized data is data that has
been translated into a format, such as JSON or XML1, that can
be stored or transmitted across a network connection. Parsing
breaks up that chunk of data and allows you to find and process
the parts you care about.
If we wrote a computer program to parse a sentence into
a very simplified model of English grammar, it could look
something like the tree above. Often when we are parsing
things, the structured datatype that results will look something
like a tree. In Haskell, we can sometimes end up having a tree
because recursive types are so easy to express in Haskell.
In this chapter, we will
•use a parsing library to cover the basics of parsing;
•demonstrate the awesome power of parser combinators;
•marshall and unmarshall some JSON data;
•talk about tokenization.
1If you do not know what JSON and XML are yet, try not to get too hung up on that.
All that matters at this point is that they are standard data formats. We’ll look at JSON in
more detail later in the chapter.

CHAPTER 24. PARSER COMBINATORS 1398
24.2 A few more words of introduction
In this chapter, we will not look too deeply into the types of
the parsing libraries we’re using, learn every sort of parser
there is, or artisanally handcraft all of our parsing functions
ourselves.
These are thoroughly considered decisions. Parsing is a
huge field of research in its own right with connections that
span natural language processing, linguistics, and program-
ming language theory. This topic could easily fill a book in
itself (in fact, it has). The underlying types and typeclasses of
the libraries we’ll be using are complicated. To be sure, if you
enjoy parsing and expect to do it a lot, those are things you’d
want to learn; they are simply out of the scope of this book.
This chapter takes a diﬀerent approach than previous chap-
ters. The focus is on enabling you to use Haskell’s parsing
libraries — not to be a master of parsing and writing parsers
in general. This is not the bottom-up approach you may be
accustomed to; by necessity, we’re working outside-in and
trying to cover what you’re likely to need. Depending on your
specific interests, you may find this chapter too long or not
nearly long enough.

CHAPTER 24. PARSER COMBINATORS 1399
24.3 Understanding the parsing process
A parser is a function that takes some textual input (it could
be aString , or another datatype such as ByteString orText) and
returns some structure as an output. That structure might
be a tree, for example, or an indexed map of locations in the
parsed data. Parsers analyze structure in conformance with
rules specified in a grammar, whether it’s a grammar of a
human language, a programming language, or a format such
as JSON.
A parser combinator is a higher-order function that takes
parsers as input and returns a new parser as output. You may
remember our brief discussion of combinators way back in
the lambda calculus chapter. Combinators are expressions
with no free variables.
The standard for what constitutes a combinator with respect
to parser combinators is a little looser. Parsers are functions,
so parser combinators are higher-order functions that can take
parsers as arguments. Usually the argument passing is elided
because the interface of parsers will often be like the State
monad which permits Reader -style implicit argument passing.
Among other things, combinators allow for recursion and for
gluing together parsers in a modular fashion to parse data
according to complex rules.
For computers, parsing is something like reading when
you’re really young. Perhaps you were taught to trace the

CHAPTER 24. PARSER COMBINATORS 1400
letters with your finger for phonetic pronunciation. Later, you
were able to follow word by word, then you started scanning
with your eyes. Eventually, you learned how to read with
subvocalization.
Since we didn’t use an analogy for Monad
We’re going to run through some code now that will demon-
strate the idea of parsing. Let’s begin by installing the parsing
librarytrifecta ,2then work through a short demonstration of
what it does. We’ll talk more about the design of trifecta in
a while. For now, we’re going to use it in a state of somewhat
ignorant bliss.
Let’s put up some code:
moduleLearnParsers where
importText.Trifecta
stop::Parsera
stop=unexpected "stop"
unexpected is a means of throwing errors in parsers like
trifecta which are an instance of the Parsing typeclass. Here
2We’ll be using this version of trifecta
http://hackage.haskell.org/package/trifecta-1.5.2

CHAPTER 24. PARSER COMBINATORS 1401
we’re using it to make the parser fail for demonstration pur-
poses.
What demonstration purposes?
We’re glad you asked! The basic idea behind a parser is that
you’re moving a sort of cursor around a linear stream of text.
It’ssimplesttothinkoftheindividualunitswithinthestreamas
characters or ideographs, though you’ll want to start thinking
of your parsing problems in chunkier terms as you progress.
The idea is that this cursor is a bit like you’re reading the text
with your finger:
Julie bit Papuchon
^
Then let us say we parsed the word “Julie” — we’ve now
consumed that input, so the cursor will be at “bit”:
Julie bit Papuchon
^
If we weren’t expecting the word “bit,” our parser could fail
here, and we’d get an error at the word “bit” like that. However,
if we did parse the word “bit” successfully and thus consumed
that input, it might look something like this:
Julie bit Papuchon
^

CHAPTER 24. PARSER COMBINATORS 1402
The analogy we’re using here isn’t perfect. One of the hard-
est problems in writing parsers, especially the parser libraries
themselves, is making it easy to express things the way the
programmer would like, but still have the resulting parser be
fast.
Back to the code
With the cursor analogy in mind, let’s return to the module
we started.
We’ll first make a little function that only parses one charac-
ter, and then sequence that with stopto make it read that one
character and then die:
-- read a single character '1'
one=char'1'
-- read a single character '1', then die
one'=one>>stop
-- equivalent to char '1' >> stop
Forone', we’re using the sequencing operator from Monadto
combine two parsers, stopandchar '1' . Given the type of >>:
(>>)::Monadm=>m a->m b->m b
it’s safe to assume that whatever char '1' returns in the
expression

CHAPTER 24. PARSER COMBINATORS 1403
char'1'>>stop
gets thrown away. Critically, any eﬀect the m aaction had
upon the monadic context remains. The result value of the
parse function gets thrown away, but the eﬀect of “moving the
cursor” remains. Another possible eﬀect is causing the parse
to fail.
A bit like…
State. Plus failure. No seriously, take a look at this definition
of theParser type:
typeParsera=String->Maybe(a,String)
You can read this as:
1.Await a string value
2.Produce a result which may or may not succeed. (A
Nothing value means the parse failed.)
3.Return a tuple of the value you wanted and whatever’s
left of the string that you didn’t consume to produce the
value of type 𝑎.
Then remind yourself of what Reader andStatelook like:

CHAPTER 24. PARSER COMBINATORS 1404
newtype Readerr a=
Reader{ runReader ::r->a }
newtype States a=
State{ runState ::s->(a, s) }
If you have convinced yourself that Stateis an elaboration
ofReader and that you can see how the Parser type looks sorta
likeState, we can move on.
The idea here with the Parser type is that the Stateis han-
dling the fact that you need to await an eventual text input and
that having parsed something out of that text input results in
a new state of the input stream. It also lets you return a value
independent of the state, while Maybehandles the possibility
of the parser failure.
If we were to look at the underlying pattern of a parsing
function such as char, you can see the State-ish pattern. Please
understand that while this should work as a character-parsing
function, we are simplifying here and this is not what the
source code of any modern parsing library will look like:

CHAPTER 24. PARSER COMBINATORS 1405
-- rudimentary char
-- demo only, this won't work as is.
char::Char->ParserChar
charc=
Parser$\s->
casesof
(x:xs)-> ifc==x
then[(c, xs)]
else[]
_ ->[]
We could encode the possibility of failure in that by adding
Maybebut at this point, that isn’t important because we’re using
a library that has encoded the possibility of failure for us. It has
also optimized the heck out of charfor us. But we wanted to
show you how the underlying function is the s ->embedded
in theParser data constructor.
Consider the type of a Hutton-Meijer parser:

CHAPTER 24. PARSER COMBINATORS 1406
-- from Text.ParserCombinators.HuttonMeijer
-- polyparse-1.11
typeToken=Char
newtype Parsera=
P([Token]->[(a, [Token])])
-- Same thing, differently formatted:
typeParser' a=String->[(a,String)]
This changes things from the previous, less common but
simpler variant, by allowing you to express a range of possibly
valid parses starting from the input provided. This is more
powerful than the Maybevariant, but this design isn’t used in
popular Haskell parser combinator libraries any longer. Al-
though the underlying implementation has changed dramati-
cally with new discoveries and designs, most parsing libraries
in Haskell are going to have an interface that behaves a bit like
Statein that the act of parsing things has an observable eﬀect
on one or more bits of state.
If we were talking about State, this means any putto the
Statevalue would be observable to the next action in the same
Monad(you can verify what follows in your REPL by import-
ingControl.Monad.Trans.State ). These examples use the trans-
former variant of State, but if you ignore the T, you should be
able to get the basic idea:

CHAPTER 24. PARSER COMBINATORS 1407
get::Monadm=>StateTs m s
put::Monadm=>s->StateTs m()
runStateT ::StateTs m a->s->m (a, s)
Prelude> runStateT (put 8) 7
((),8)
Prelude> runStateT get 8
(8,8)
Prelude> runStateT (put 1 >> get) 8
(1,1)
Prelude> (runStateT $ put 1 >> get) 0
(1,1)
Prelude> (runStateT $ put 2 >> get) 10021490234890
(2,2)
Prelude> (runStateT $ put 2 >> return 9001) 0
(9001,2)
Nowputreturns a unit value, a throwaway value, so we’re
only evaluating it for eﬀect anyway. It modifies the state but
doesn’t have any value of its own. So when we throw away its
value, we’re left with its eﬀect on the state, although getputs
that value into both the 𝑎and𝑠slots in the tuple.
This is an awful lot like what happens when we sequence a
parsing function such as charwithstop, as above. There is no
real result of char, but it does change the state. The state here
is the location of the cursor in the input stream. In reality, a

CHAPTER 24. PARSER COMBINATORS 1408
modern and mature parser design in Haskell will often look
about as familiar to you as the alien hellscape underneath the
frozen crust of one of the moons of Jupiter. Don’t take the
idea of there being an actual cursor too literally, but there may
be some utility in imagining it this way.
Back to our regularly scheduled coding
Onward with the code:
-- read two characters, '1', and '2'
oneTwo=char'1'>>char'2'
-- read two characters,
-- '1' and '2', then die
oneTwo' =oneTwo>>stop
testParse ::ParserChar->IO()
testParse p=
print$parseString p mempty "123"
The𝑝argument is a parser. Specifically, it’s a character
parser. The functions oneandoneTwo have the type Parser Char .
You can check the types of one'andoneTwo' yourself.
We needed to declare the type of testParse in order to Show
what we parsed because of ambiguity.

CHAPTER 24. PARSER COMBINATORS 1409
The key thing to realize here is that we’re using parsers like
values and combining them using the same stuﬀ we use with
ordinary functions or operators from the Applicative andMonad
typeclasses. The structure that makes up the Applicative or
Monadin this case is the Parser itself.
Next we’ll write a function to print a string to standard
output (stdout) with a newline prefixed, and then use that
function as part of a mainthat will show us what we’ve got so
far:
pNLs=
putStrLn ( '\n':s)
main= do
pNL"stop:"
testParse stop
pNL"one:"
testParse one
pNL"one':"
testParse one'
pNL"oneTwo:"
testParse oneTwo
pNL"oneTwo':"
testParse oneTwo'
Let’s run it and interpret the results. Since it’s text on a

CHAPTER 24. PARSER COMBINATORS 1410
computer screen instead of tea leaves, we’ll call it science. If
you remain unconvinced, you have our permission to don a
white labcoat and print the output using a dot-matrix printer.
Some of you kids probably don’t even know what a dot-matrix
printer is.3
Runmainand see what happens:
Prelude> main
stop:
Failure (interactive):1:1: error: unexpected
stop
123<EOF>
^
We failed immediately before consuming any input in the
above, so the caret in the error is at the beginning of our string
value.
Next result:
one:
Success '1'
We parsed a single character, the digit 1. The result is know-
ing we succeeded. But what about the rest of the input stream?
Well, the thing we used to run the parser dropped the rest of
3shakes fist at sky

CHAPTER 24. PARSER COMBINATORS 1411
the input on the floor. There are ways to change this behavior
which we’ll explain in the exercises.
Next up:
one':
Failure (interactive):1:2: error: unexpected
stop
123<EOF>
^
We parsed a single character successfully, then dropped it
because we used >>to sequence it with stop. This means the
cursor was one character forward due to the previous parser
succeeding. Helpfully, trifecta tells us where our parser failed.
And for our last result:
oneTwo:
Success '2'
oneTwo':
Failure (interactive):1:3: error: unexpected
stop
123<EOF>
^
It’s the same as before, but we parsed two characters indi-
vidually. What if we we don’t want to discard the first character
we parsed and instead parse “12?” See the exercises below!

CHAPTER 24. PARSER COMBINATORS 1412
Exercises: Parsing Practice
1.There’s a combinator that’ll let us mark that we expect
an input stream to be finished at a particular point in our
parser. In the parsers library this is simply called eof(end-
of-file) and is in the Text.Parser.Combinators module. See
if you can make the oneandoneTwo parsers fail because
they didn’t exhaust the input stream!
2.Usestring to make a Parser that parses “1”, “12”, and “123”
out of the example input respectively. Try combining it
withstoptoo. That is, a single parser should be able to
parse all three of those strings.
3.Try writing a Parser that does what string does, but using
char.
Intermission: parsing free jazz
Let us play with these parsers! We typically use the parseString
function to run parsers, but if you figure some other way that
works for you, so be it! Here’s some parsing free jazz, if you
will, meant only to help develop your intuition about what’s
going on:
Prelude> import Text.Trifecta
Prelude> :t char
char :: CharParsing m => Char -> m Char

CHAPTER 24. PARSER COMBINATORS 1413
Prelude> :t parseString
parseString
:: Parser a
-> Text.Trifecta.Delta.Delta
-> String
-> Result a
Prelude> let gimmeA = char 'a'
Prelude> :t parseString gimmeA mempty
parseString gimmeA mempty :: String -> Result Char
Prelude> parseString gimmeA mempty "a"
Success 'a'
Prelude> parseString gimmeA mempty "b"
Failure (interactive):1:1: error: expected: "a"
b<EOF>
^
Prelude> parseString (char 'b') mempty "b"
Success 'b'
Prelude> parseString (char 'b' >> char 'c') mempty "b"
Failure (interactive):1:2: error: unexpected
EOF, expected: "c"
b<EOF>
^
Prelude> parseString (char 'b' >> char 'c') mempty "bc"
Success 'c'

CHAPTER 24. PARSER COMBINATORS 1414
Prelude> parseString (char 'b' >> char 'c') mempty "abc"
Failure (interactive):1:1: error: expected: "b"
abc<EOF>
^
Seems like we ought to have a way to say, “parse this string”
rather than having to sequence the parsers of individual char-
acters bit by bit, right? Turns out, we do:
Prelude> parseString (string "abc") mempty "abc"
Success "abc"
Prelude> parseString (string "abc") mempty "bc"
Failure (interactive):1:1: error: expected: "abc"
bc<EOF>
^
Prelude> parseString (string "abc") mempty "ab"
Failure (interactive):1:1: error: expected: "abc"
ab<EOF>
^
Importantly, it’s not a given that a single parser exhausts all
of its input — they only consume as much text as they need
to produce the value of the type requested.
Prelude> parseString (char 'a') mempty "abcdef"
Success 'a'
Prelude> let stop = unexpected "stop pls"

CHAPTER 24. PARSER COMBINATORS 1415
Prelude> parseString (char 'a' >> stop) mempty "abcdef"
Failure (interactive):1:2: error: unexpected
stop pls
abcdef<EOF>
^
Prelude> parseString (string "abc") mempty "abcdef"
Success "abc"
Prelude> parseString (string "abc" >> stop) mempty "abcdef"
Failure (interactive):1:4: error: unexpected
stop pls
abcdef<EOF>
^
Note that we can also parse UTF-8 encoded ByteString s with
trifecta :
Prelude> import Text.Trifecta
Prelude> :t parseByteString
parseByteString
:: Parser a
-> Text.Trifecta.Delta.Delta
-> Data.ByteString.Internal.ByteString
-> Result a
Prelude> parseByteString (char 'a') mempty "a"
Success 'a'

CHAPTER 24. PARSER COMBINATORS 1416
This ends the free jazz session. We now return to serious
matters.
24.4 Parsing fractions
Now that we have some idea of what parsing is, what parser
combinators are, and what the monadic underpinnings of
parsing look like, let’s move on to parsing fractions. The top
of this module should look like this:
{-# LANGUAGE OverloadedStrings #-}
moduleText.Fractions where
importControl.Applicative
importData.Ratio ((%))
importText.Trifecta
We named the module Text.Fractions because we’re pars-
ing fractions out of text input, and there’s no need to be more
clever about it than that. We’re going to be using String in-
puts with trifecta at first, but you’ll see why we threw an
OverloadedStrings extension in there later.
Now, on to parsing fractions! We’ll start with some test
inputs:

CHAPTER 24. PARSER COMBINATORS 1417
badFraction ="1/0"
alsoBad ="10"
shouldWork ="1/2"
shouldAlsoWork ="2/1"
Then we’ll write our actual parser:
parseFraction ::ParserRational
parseFraction = do
numerator <-decimal
-- [2] [1]
char'/'
-- [3]
denominator <-decimal
-- [ 4 ]
return (numerator %denominator)
-- [5] [6]
1.decimal ::Integral a=>Parsera
This is the type of decimal within the context of those
functions. If you use GHCi to query the type of decimal ,
you will see a more polymorphic type signature.
2.Herenumerator has the type Integral a => a .
3.char::Char->ParserChar

CHAPTER 24. PARSER COMBINATORS 1418
As with decimal , if you query the type of charin GHCi,
you’ll see a more polymorphic type, but this is the type
ofcharin context.
4.Same deal as numerator , but when we match an integral
number we’re binding the result to the name denominator .
5.The final result has to be a parser, so we embed our inte-
gral value in the Parser type by using return.
6.We construct ratios using the %infix operator:
(%)::Integral a
=>a->a->GHC.Real.Ratioa
Then the fact that our final result is a Rational makes the
Integral a => a values into concrete Integer values.
typeRational =GHC.Real.RatioInteger
We’ll put together a quick shim main function to run the
parser against the test inputs and see the results:

CHAPTER 24. PARSER COMBINATORS 1419
main::IO()
main= do
letparseFraction' =
parseString parseFraction mempty
print$parseFraction' shouldWork
print$parseFraction' shouldAlsoWork
print$parseFraction' alsoBad
print$parseFraction' badFraction
Try not to worry about the mempty values; it might give you
a clue about what’s going on in trifecta under the hood, but
it’s not something we’re going to explore in this chapter.
We will briefly note the type of parseString , which is how
we’re running the parser we created:
parseString ::Parsera
->Text.Trifecta .Delta.Delta
->String
->Resulta
The first argument is the parser we’re going to run against
the input, the second is a Delta, the third is the String we’re
parsing, and then the final result is either the thing we wanted
of type 𝑎or an error string to let us know something went
wrong. You can ignore the Deltathing — use mempty to provide
the do-nothing input. We won’t be covering deltas in this book
so consider it extra credit if you get curious.

CHAPTER 24. PARSER COMBINATORS 1420
Anyway, when we run the code, the results look like this:
Prelude> main
Success (1 % 2)
Success (2 % 1)
Failure (interactive):1:3: error: unexpected
EOF, expected: "/", digit
10<EOF>
^
Success *** Exception: Ratio has zero denominator
The first two succeeded properly. The third failed because it
couldn’t parse a fraction out of the text “10”. The error is telling
us that it ran out of text in the input stream while still waiting
for the character '/'. The final error did not result from the
process of parsing; we know that because it is a Success data
constructor. The final error resulted from trying to construct
a ratio with a denominator that is zero — which makes no
sense. We can reproduce the issue in GHCi:
Prelude> 1 % 0
*** Exception: Ratio has zero denominator
-- So the parser result is which is tantamount to
Prelude> Success (1 % 0)
Success *** Exception: Ratio has zero denominator
This is sort of a problem because exceptions end our pro-
grams. Observe:

CHAPTER 24. PARSER COMBINATORS 1421
main::IO()
main= do
letparseFraction' =
parseString parseFraction mempty
print$parseFraction' badFraction
print$parseFraction' shouldWork
print$parseFraction' shouldAlsoWork
print$parseFraction' alsoBad
We’ve put the expression that throws an exception in the
first line this time, when we run it we get:
Prelude> main
Success *** Exception: Ratio has zero denominator
So, our program halted on the error. This is not great. You
may be tempted to “handle” the error. Catching exceptions
is okay, but this is a particular class of exceptions that means
something is quite wrong with your program. You should elim-
inate the possibility of exceptions occurring in your programs
where possible.
We’ll talk more about error handling in a later chapter, but
the idea here is that a Parser type already explicitly encodes
the possibility of failure. It’s better for a value of type Parser
ato have only one vector for errors and that vector is the
parser’s ability to encode failure. There may be an edge case

CHAPTER 24. PARSER COMBINATORS 1422
that doesn’t suit this design preference, but it’s a very good
idea to not have exceptions or bottoms that aren’t explicitly
called out as a possibility in the types whenever possible.
We could modify our program to handle the 0 denominator
case and change it into a parse error:
virtuousFraction ::ParserRational
virtuousFraction = do
numerator <-decimal
char'/'
denominator <-decimal
casedenominator of
0->fail"Denominator cannot be zero"
_ ->return (numerator %denominator)
Here is our first explicit use of fail, which by historical ac-
cident is part of the Monadtypeclass. Realistically, not all Monads
have a proper implementation of fail, so it will be moved out
into aMonadFail class eventually. For now, it suffices to know
that it is our means of returning an error for the Parser type
here.
Now for another run of our test inputs, but with our more
cautious parser:

CHAPTER 24. PARSER COMBINATORS 1423
testVirtuous ::IO()
testVirtuous = do
letvirtuousFraction' =
parseString virtuousFraction mempty
print$virtuousFraction' badFraction
print$virtuousFraction' alsoBad
print$virtuousFraction' shouldWork
print$virtuousFraction' shouldAlsoWork
When we run this, we’re going to get a slightly diﬀerent
result at the end:
Prelude> testVirtuous
Failure (interactive):1:4: error: Denominator
cannot be zero, expected: digit
1/0<EOF>
^
Failure (interactive):1:3: error: unexpected
EOF, expected: "/", digit
10<EOF>
^
Success (1 % 2)
Success (2 % 1)
Now we have no bottom causing the program to halt and
we get a Failure value which explains the cause for the failure.
Much better!

CHAPTER 24. PARSER COMBINATORS 1424
Exercise: Unit of Success
This should not be unfamiliar at this point, even if you do not
understand all the details:
Prelude> parseString integer mempty "123abc"
Success 123
Prelude> parseString (integer >> eof) mempty "123abc"
Failure (interactive):1:4: error: expected: digit,
end of input
123abc<EOF>
^
Prelude> parseString (integer >> eof) mempty "123"
Success ()
You may have already deduced why it returns ()as aSuccess
result here; it’s consumed all the input but there is no result
to return from having done so. The result Success () tells you
the parse was successful and consumed the entire input, so
there’s nothing to return.
What we want you to try now is rewriting the final example
so it returns the integer that it parsed instead of Success () .
It should return the integer successfully when it receives an
input with an integer followed by an EOF and fail in all other
cases:
Prelude> parseString (yourFuncHere) mempty "123"

CHAPTER 24. PARSER COMBINATORS 1425
Success 123
Prelude> parseString (yourFuncHere) mempty "123abc"
Failure (interactive):1:4: error: expected: digit,
end of input
123abc<EOF>
^
24.5 Haskell’s parsing ecosystem
Haskell has several excellent parsing libraries available. parsec
andattoparsec are perhaps the two most well known parser
combinator libraries in Haskell, but there is also megaparsec
and others. aesonandcassava are among the libraries designed
for parsing specific types of data (JSON data and CSV data,
respectively).
For this chapter, we opted to use trifecta , as you’ve seen.
One reason for that decision is that trifecta has error messages
that are very easy to read and interpret, unlike some other
libraries. Also, trifecta does not seem likely to undergo major
changes in its fundamental design. Its design is somewhat
unusual and complex, but most of the things that make it
unusual will be irrelevant to you in this chapter. If you intend
to do a lot of parsing in production, you may need to get
comfortable using attoparsec , as it is particularly known for
very speedy parsing; you will see some attoparsec (andaeson)
later in the chapter.

CHAPTER 24. PARSER COMBINATORS 1426
The design of trifecta has evolved such that the API4is split
across two libraries, parsers5andtrifecta . The reason for this
is that the trifecta package itself provides the concrete im-
plementation of the trifecta parser as well as trifecta -specific
functionality, but the parsers API is a collection of typeclasses
that abstract over diﬀerent kinds of things parsers can do. The
Text.Trifecta module handles exporting what you need to get
started from each package, so this information is mostly so
you know where to look if you need to start spelunking.
Typeclasses of parsers
As we noted above, trifecta relies on the parsers library for
certain typeclasses. These typeclasses abstract over common
kinds of things parsers do. We’re only going to note a few
things here that we’ll be seeing in the chapter so that you have
a sense of their provenance.
Note that the following is a discussion of code provided for
you by the parsers library, you do not need to type this in!
1.The typeclass Parsing hasAlternative as a superclass. We’ll
talk more about Alternative in a bit. The Parsing typeclass
4API stands for application programming interface. When we write software that
relies on libraries or makes requests to a service such as Twitter — basically, software
that relies on other software — we rely on a set of defined functions. The API is that set
of functions that we use to interface with that software without having to write those
functions or worry too much about their source code. When you look at a library on
Hackage, (unless you click to view the source code), you’re looking at the API of that
library.
5http://hackage.haskell.org/package/parsers

CHAPTER 24. PARSER COMBINATORS 1427
provides for functionality needed to describe parsers in-
dependent of input type. A minimal complete instance of
this typeclass defines the following functions: try,(<?>),
andnotFollowedBy . Let’s start with try:
-- Text.Parser.Combinators
classAlternative m=>Parsing mwhere
try::m a->m a
Thistakesaparserthatmayconsumeinputand, onfailure,
goes back to where it started and fails if we didn’t consume
input.
It also gives us the function notFollowedBy which does not
consume input but allows us to match on keywords by
matching on a string of characters that is not followed by
some thing we do not want to match:
notFollowedBy ::Showa=>m a->m()
-- > noAlpha = notFollowedBy alphaNum
-- > keywordLet =
-- try $ string "let" <* noAlpha
2.TheParsing typeclass also includes unexpected which is
used to emit an error on an unexpected token, as we saw
earlier, and eof. Theeoffunction only succeeds at the end
of input:

CHAPTER 24. PARSER COMBINATORS 1428
eof::m()
-- > eof =
-- notFollowedBy anyChar
-- <?> "end of input"
We’ll be seeing more of this one in upcoming sections.
3.The library also defines the typeclass CharParsing , which
hasParsing as a superclass. This handles parsing individ-
ual characters.
-- Text.Parser.Char
classParsing m=>CharParsing mwhere
We’ve already seen charfrom this class, but it also includes
these:
-- Parses any single character other
-- than the one provided. Returns
-- the character parsed.
notChar ::Char->mChar
-- Parser succeeds for any character.
-- Returns the character parsed.
anyChar ::mChar

CHAPTER 24. PARSER COMBINATORS 1429
-- Parses a sequence of characters, returns
-- the string parsed.
string::String->mString
-- Parses a sequence of characters
-- represented by a Text value,
-- returns the parsed Text fragment.
text::Text->mText
Theparsers library has much more than this, but for our
immediate purposes these will suffice. The important point is
that it defines for us some typeclasses and basic combinators
for common parsing tasks. We encourage you to explore the
documentation more on your own.
24.6 Alternative
Let’s say we had a parser for numbers and one for alphanu-
meric strings:
Prelude> import Text.Trifecta
Prelude> parseString (some letter) mempty "blah"
Success "blah"
Prelude> parseString integer mempty "123"
Success 123
What if we had a type that could be an Integer or aString ?

CHAPTER 24. PARSER COMBINATORS 1430
moduleAltParsing where
importControl.Applicative
importText.Trifecta
typeNumberOrString =
EitherInteger String
a="blah"
b="123"
c="123blah789"
parseNos ::ParserNumberOrString
parseNos =
(Left<$>integer)
<|>(Right<$>some letter)
main= do
letp f i=
parseString f mempty i
print$p (some letter) a
print$p integer b
print$p parseNos a
print$p parseNos b
print$p (many parseNos) c
print$p (some parseNos) c

CHAPTER 24. PARSER COMBINATORS 1431
We can read <|>as being an or, or disjunction, of our two
parsers; manyis zero or more and someis one or more.
Prelude> parseString (some integer) mempty "123"
Success [123]
Prelude> parseString (many integer) mempty "123"
Success [123]
Prelude> parseString (many integer) mempty ""
Success []
Prelude> parseString (some integer) mempty ""
Failure (interactive):1:1: error: unexpected
EOF, expected: integer
<EOF>
^
What we’re taking advantage of here with some,many, and
(<|>)is theAlternative typeclass:

CHAPTER 24. PARSER COMBINATORS 1432
classApplicative f=>Alternative fwhere
-- | The identity of '<|>'
empty::f a
-- | An associative binary operation
(<|>)::f a->f a->f a
-- | One or more.
some::f a->f [a]
some v=some_v
where
many_v=some_v<|>pure[]
some_v=(fmap (:) v)<*>many_v
-- | Zero or more.
many::f a->f [a]
many v=many_v
where
many_v=some_v<|>pure[]
some_v=(fmap (:) v)<*>many_v
If you use the :infocommand in the REPL after importing
Text.Trifecta or loading the above module, you’ll find some
andmanyare defined in GHC.Base because they come from this
typeclass rather than being specific to a particular parser or to
theparsers library, or even to this particular problem domain.

CHAPTER 24. PARSER COMBINATORS 1433
What if we wanted to require that each value be separated
by newline? QuasiQuotes lets us have a multiline string without
the newline separators and use it as a single argument:
{-# LANGUAGE QuasiQuotes #-}
moduleAltParsing where
importControl.Applicative
importText.RawString.QQ
importText.Trifecta
typeNumberOrString =
EitherInteger String
eitherOr ::String
eitherOr =[r|
123
abc
456
def
|]

CHAPTER 24. PARSER COMBINATORS 1434
QuasiQuotes
Above, the [r|is beginning a quasiquoted6section, using the
quasiquoter named r. Note we had to enable the QuasiQuotes
language extension to use this syntax. At time of writing ris
defined in raw-strings-qq version 1.1 as follows:
r::QuasiQuoter
r=QuasiQuoter {
-- Extracted from dead-simple-json.
quoteExp =
return.LitE.StringL
.normaliseNewlines,
-- error messages elided
quotePat =
\_ ->fail"some error message"
quoteType =
\_ ->fail"some error message"
quoteDec =
\_ ->fail"some error message"
The idea here is that this is a macro that lets us write ar-
bitrary text inside of the block that begins with [r|and ends
6There’s a rather nice wiki page and tutorial example at: https://wiki.haskell.org/
Quasiquotation

CHAPTER 24. PARSER COMBINATORS 1435
with|]. This specific quasiquoter exists to allow writing mul-
tiline strings without manual escaping. The quasiquoter is
generating the following for us:
"\n\
\123\n\
\abc\n\
\456\n\
\def\n"
Not as nice right? As it happens, if you want to see what
a quasiquoter or Template Haskell7is generating at compile-
time, you can enable the -ddump-splices flag to see what it does.
Here’s an example using a minimal stub file:
7https://wiki.haskell.org/Template_Haskell

CHAPTER 24. PARSER COMBINATORS 1436
{-# LANGUAGE QuasiQuotes #-}
moduleQuasimodo where
importText.RawString.QQ
eitherOr ::String
eitherOr =[r|
123
abc
456
def
|]
Then in GHCi we use the :setcommand to turn on the
splice dumping flag so we can see what the quasiquoter gener-
ated:
Prelude> :set -ddump-splices
Prelude> :l code/quasi.hs
[1 of 1] Compiling Quasimodo
code/quasi.hs:(8,12)-(12,2): Splicing expression
"\n\
\123\n\
\abc\n\
\456\n"

CHAPTER 24. PARSER COMBINATORS 1437
======>
"\n\
\123\n\
\abc\n\
\456\n"
Right, so back to the parser we were going to write!
Return to Alternative
All right, we return now to our AltParsing module. We’re going
to use this fantastic function:
parseNos ::ParserNumberOrString
parseNos =
(Left<$>integer)
<|>(Right<$>some letter)
and rewrite mainto apply that to the eitherOr value:
main= do
letp f i=parseString f mempty i
print$p parseNos eitherOr
Note that we lifted LeftandRightover their arguments.
This is because there is Parser structure between the (potential)
value obtained by running the parser and what the data con-
structor expects. A value of type Parser Char is a parser that

CHAPTER 24. PARSER COMBINATORS 1438
will possibly produce a Charvalue if it is given an input that
doesn’t cause it to fail. The type of some letter is the following:
Prelude> import Text.Trifecta
Prelude> :t some letter
some letter :: CharParsing f => f [Char]
However, for our purposes we can say that the type is specif-
icallytrifecta ’sParser type:
Prelude> let someLetter = some letter :: Parser [Char]
Prelude> let someLetter = some letter :: Parser String
If we try to mash a data constructor expecting a String and
our parser-of-string together like a kid playing with action
figures, we get a type error:
Prelude> data MyName = MyName String deriving Show
Prelude> MyName someLetter
Couldn't match type ‘Parser String’ with ‘[Char]’
Expected type: String
Actual type: Parser String
In the first argument of ‘MyName’, namely ‘someLetter’
In the expression: MyName someLetter
Unless we lift it over the Parser structure, since Parser is a
Functor !

CHAPTER 24. PARSER COMBINATORS 1439
Prelude> :info Parser
{... content elided ...}
instance Monad Parser
instance Functor Parser
instance Applicative Parser
instance Monoid a => Monoid (Parser a)
instance Errable Parser
instance DeltaParsing Parser
instance TokenParsing Parser
instance Parsing Parser
instance CharParsing Parser
We should need an fmapright?
-- same deal
Prelude> :t MyName <$> someLetter
MyName <$> someLetter :: Parser MyName
Prelude> :t MyName `fmap` someLetter
MyName `fmap` someLetter :: Parser MyName
Then running either of them:
Prelude> parseString someLetter mempty "Chris"
Success "Chris"
Prelude> let mynameParser = MyName <$> someLetter
Prelude> parseString mynameParser mempty "Chris"

CHAPTER 24. PARSER COMBINATORS 1440
Success (MyName "Chris")
Cool.
Back to our original code, which will spit out an error:
Prelude> main
Failure (interactive):1:1: error: expected: integer,
letter
It’s easier to see why if we look at the test string:
Prelude> eitherOr
"\n123\nabc\n456\ndef\n"
One way to fix this is to amend the quasiquoted string:
eitherOr ::String
eitherOr =[r|123
abc
456
def
|]
What if we wanted to permit a newline before attempting
to parse strings or integers?

CHAPTER 24. PARSER COMBINATORS 1441
eitherOr ::String
eitherOr =[r|
123
abc
456
def
|]
parseNos ::ParserNumberOrString
parseNos =
skipMany (oneOf "\n")
>>
(Left<$>integer)
<|>(Right<$>some letter)
main= do
letp f i=parseString f mempty i
print$p parseNos eitherOr
Prelude> main
Success (Left 123)
OK, but we’d like to keep parsing after each line. If we try
the obvious thing and use someto ask for one-or-more results,
we’ll get a somewhat mysterious error:

CHAPTER 24. PARSER COMBINATORS 1442
Prelude> parseString (some parseNos) mempty eitherOr
Failure (interactive):6:1: error: unexpected
EOF, expected: integer, letter
<EOF>
^
The issue here is that while skipMany lets us skip zero or more
times, it means we started the next run of the parser before we
hit EOF. This means it expects us to match an integer or some
letters after having seen the newline character after “def”. We
can simply amend the input:
eitherOr ::String
eitherOr =[r|
123
abc
456
def|]
Our previous attempt will now work fine:
Prelude> parseString (some parseNos) mempty eitherOr
Success [Left 123,Right "abc",Left 456,Right "def"]
If we’re dissatisfied with simply changing the rules of the
game, there are a couple ways we can make our parser cope
withspuriousterminalnewlines. Oneistoaddanother skipMany
rule after we parse our value:

CHAPTER 24. PARSER COMBINATORS 1443
parseNos ::ParserNumberOrString
parseNos = do
skipMany (oneOf "\n")
v<-(Left<$>integer)
<|>(Right<$>some letter)
skipMany (oneOf "\n")
return v
Another option is to keep the previous version of the parser
which skips a potential leading newline:
parseNos ::ParserNumberOrString
parseNos =
skipMany (oneOf "\n")
>>
(Left<$>integer)
<|>(Right<$>some letter)
But then tokenize it with the default tokenbehavior:
Prelude> parseString (some (token parseNos)) mempty eitherOr
Success [Left 123,Right "abc",Left 456,Right "def"]
We’ll explain soon what this token stuﬀ is about, but we
want to be a bit careful here as token parsers and character
parsers are diﬀerent sorts of things. What applying tokento
parseNos did for us here is make it optionally consume trailing

CHAPTER 24. PARSER COMBINATORS 1444
whitespace we don’t care about, where whitespace includes
newline characters.
Exercise: Try Try
Make a parser, using the existing fraction parser plus a new dec-
imal parser, that can parse either decimals or fractions. You’ll
want to use <|>fromAlternative to combine the…alternative
parsers. If you find this too difficult, write a parser that parses
straightforward integers or fractions. Make a datatype that
contains either an integer or a rational and use that datatype as
the result of the parser. Or use Either . Run free, grasshopper.
Hint: we’ve not explained it yet, but you may want to try
try.
24.7 Parsing configuration files
For our next examples, we’ll be using the INI8configuration
file format, partly because it’s an informal standard so we can
play fast and loose for learning and experimentation purposes.
We’re also using INI because it’s relatively uncomplicated.
Here’s a teensy example of an INI config file:
8INI is an informal standard for configuration files on some platforms. The name
comes from the file extension, .ini, short for “initialization.”

CHAPTER 24. PARSER COMBINATORS 1445
; comment
[section]
host=wikipedia .org
alias=claw
The above contains a comment, which contributes noth-
ing to the data parsed out of the configuration file but which
may provide context to the settings being configured. It’s fol-
lowed by a section header named "section" which contains
two settings: one named "host" with the value "wikipedia.org" ,
another named "alias" with the value "claw" .
We’ll begin this example with our pragmas, module decla-
ration, and imports:

CHAPTER 24. PARSER COMBINATORS 1446
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE QuasiQuotes #-}
moduleData.Ini where
importControl.Applicative
importData.ByteString (ByteString )
importData.Char (isAlpha)
importData.Map (Map)
import qualified Data.Map asM
importData.Text (Text)
import qualified Data.Text.IO asTIO
importTest.Hspec
importText.RawString.QQ
-- parsers 0.12.3, trifecta 1.5.2
importText.Trifecta
OverloadedStrings andQuasiQuotes should be familiar by now.
When writing parsers in Haskell, it’s often easiest to work in
terms of smaller parsers that deal with a sub-problem of the
overall parsing problem you’re solving, then combine them
into the final parser. This isn’t a perfect recipe for understand-
ing your parser, but being able to compose them straightfor-
wardly like functions is pretty nifty. Let’s start by creating a
test input for an INI header, a datatype, and then the parser

CHAPTER 24. PARSER COMBINATORS 1447
for it:
headerEx ::ByteString
headerEx ="[blah]"
-- "[blah]" -> Section "blah"
newtype Header=
HeaderString
deriving (Eq,Ord,Show)
parseBracketPair ::Parsera->Parsera
parseBracketPair p=
char'['*>p<*char']'
-- these operators mean the brackets
-- will be parsed and then discarded
-- but the p will remain as our result
parseHeader ::ParserHeader
parseHeader =
parseBracketPair ( Header<$>some letter)
Here we’ve combined two parsers in order to parse a Header .
We can experiment with each of them in the REPL. First
we’ll examine the types of the some letter parser we passed to
parseBracketPair :

CHAPTER 24. PARSER COMBINATORS 1448
Prelude> :t some letter
some letter :: CharParsing f => f [Char]
Prelude> :t Header <$> some letter
Header <$> some letter :: CharParsing f => f Header
Prelude> let slp = Header <$> some letter :: Parser Header
The first type is some parser that can understand characters
which will produce a String value if it succeeds. The second
type is the same, but produces a Header value instead of a String .
Parser types in Haskell almost always encode the possibility
of failure; we’ll cover how later in this chapter. The third type
gives us concrete Parser type from trifecta where there had
been the polymorphic type 𝑓.
Theletter function parses a single character, while some
letter parses one or more characters. We need to wrap the
Header constructor around that so that our result there — what-
everlettersmightbeinsidethebrackets, the 𝑝ofparseBracketPair
— will be labeled as the Header of the file in the final parse.
Next,assignmentEx is just some test input so we can begin
kicking around our parser. The type synonyms are to make
the types more readable as well. Nothing too special here:

CHAPTER 24. PARSER COMBINATORS 1449
assignmentEx ::ByteString
assignmentEx ="woot=1"
typeName=String
typeValue=String
typeAssignments =MapNameValue
parseAssignment ::Parser(Name,Value)
parseAssignment = do
name<-some letter
_ <-char'='
val<-some (noneOf "\n")
skipEOL -- important!
return (name, val)
-- | Skip end of line and
-- whitespace beyond.
skipEOL ::Parser()
skipEOL =skipMany (oneOf "\n")
Let us explain parseAssignment step by step. For parsing the
initial key or name of an assignment, we parse one or more
letters:
name<-some letter

CHAPTER 24. PARSER COMBINATORS 1450
Then we parse and throw away the “=” used to separate keys
and values:
_ <-char'='
Then we parse one or more characters as long as they aren’t
a newline. This is so letters, numbers, and whitespace are
permitted:
val<-some (noneOf "\n")
We skip “end-of-line” until we stop getting newline charac-
ters:
skipEOL -- important!
This is so we can delineate the end of assignments and
parse more than one assignment in a straightforward manner.
Consider an alternative variant of this same parser that doesn’t
haveskipEOL :
parseAssignment' ::Parser(Name,Value)
parseAssignment' = do
name<-some letter
_ <-char'='
val<-some (noneOf "\n")
return (name, val)

CHAPTER 24. PARSER COMBINATORS 1451
Then trying out this variant of the parser:
Prelude> let spa' = some parseAssignment'
Prelude> let s = "key=value\nblah=123"
Prelude> parseString spa' mempty s
Success [("key","value")]
Pity. Can’t parse the second assignment. But the first ver-
sion that includes the skipEOL should work:
Prelude> let spa = some parseAssignment
Prelude> parseString spa mempty s
Success [("key","value"),("blah","123")]
Prelude> let d = "key=value\n\n\ntest=data"
Prelude> parseString spa mempty d
Success [("key","value"),("test","data")]
We have to skip the one-or-more newline characters sepa-
rating the first and second assignment in order for the rerun
of the assignment parser to begin successfully parsing the
letters that make up the key of the second assignment. Happy-
making, right?
We finish things oﬀ for parseAssignment by tupling name and
value together and re-embedding the result in the Parser type:
return(name, val)

CHAPTER 24. PARSER COMBINATORS 1452
Then for dealing with INI comments, that is, skipping them
in the parser and discarding the data:
commentEx ::ByteString
commentEx =
"; last modified 1 April \
\2001 by John Doe"
commentEx' ::ByteString
commentEx' =
"; blah\n; woot\n\n;hah"
-- | Skip comments starting at the
-- beginning of the line.
skipComments ::Parser()
skipComments =
skipMany ( do _ <- char';'<|>char'#'
skipMany (noneOf "\n")
skipEOL)
We made a couple of comment examples for testing the
parser. Note that comments can begin with #or;.
Next, we need section parsing. We’ll make some data for
testing that out, as we did with comments above. This is also
where we’ll put that QuasiQuotes extension to use, allowing us
to make multiline strings nicer to write:

CHAPTER 24. PARSER COMBINATORS 1453
sectionEx ::ByteString
sectionEx =
"; ignore me \n[states] \nChris=Texas"
sectionEx' ::ByteString
sectionEx' =[r|
; ignore me
[states]
Chris=Texas
|]
sectionEx'' ::ByteString
sectionEx'' =[r|
; comment
[section]
host=wikipedia .org
alias=claw
[whatisit]
red=intoothandclaw
|]
Then we get into the section parsing:

CHAPTER 24. PARSER COMBINATORS 1454
dataSection =
Section HeaderAssignments
deriving (Eq,Show)
newtype Config=
Config(MapHeaderAssignments )
deriving (Eq,Show)
skipWhitespace ::Parser()
skipWhitespace =
skipMany (char ' '<|>char'\n')
parseSection ::ParserSection
parseSection = do
skipWhitespace
skipComments
h<-parseHeader
skipEOL
assignments <-some parseAssignment
return$
Section h (M.fromList assignments)
Above, we defined datatypes for a section and an entire INI
config. You’ll notice that parseSection skips both whitespace
and comments now. And it returns the parsed section with

CHAPTER 24. PARSER COMBINATORS 1455
the header (that’s the ℎ) and a map of assignments:
*Data.Ini> parseByteString parseSection mempty sectionEx
Success (Section (Header "states")
(fromList [("Chris","Texas")]))
So far, so good. Next, let’s roll the sections up into a Map
that keys section data by section name, with the values being
further more Maps of assignment names mapped to their
values. We use foldrto aggregate the list of sections into a
single Map value:
rollup::Section
->MapHeaderAssignments
->MapHeaderAssignments
rollup(Section h a) m=
M.insert h a m
parseIni ::ParserConfig
parseIni = do
sections <-some parseSection
letmapOfSections =
foldr rollup M.empty sections
return ( ConfigmapOfSections)
After you load this code into your REPL, try running:

CHAPTER 24. PARSER COMBINATORS 1456
parseByteString parseIni mempty sectionEx
and comparing it to the output of:
parseByteString parseSection mempty sectionEx
that you saw above.
Now we’ll put these things together. We’re interested in
whether our parsers do what they should do rather than pars-
ing an actual INI file, so we’ll have mainrun some hspectests.
We’ll use a helper function, maybeSuccess , as part of the tests:
maybeSuccess ::Resulta->Maybea
maybeSuccess (Success a)=Justa
maybeSuccess _ =Nothing
main::IO()
main=hspec$ do
describe "Assignment Parsing" $
it"can parse a simple assignment" $ do
letm=parseByteString
parseAssignment
mempty assignmentEx
r'=maybeSuccess m
print m
r' `shouldBe` Just("woot","1")

CHAPTER 24. PARSER COMBINATORS 1457
describe "Header Parsing" $
it"can parse a simple header" $ do
letm=
parseByteString parseHeader
mempty headerEx
r'=maybeSuccess m
print m
r' `shouldBe` Just(Header"blah")
describe "Comment parsing" $
it"Skips comment before header" $ do
letp=skipComments >>parseHeader
i="; woot\n[blah]"
m=parseByteString p mempty i
r'=maybeSuccess m
print m
r' `shouldBe` Just(Header"blah")

CHAPTER 24. PARSER COMBINATORS 1458
describe "Section parsing" $
it"can parse a simple section" $ do
letm=parseByteString parseSection
mempty sectionEx
r'=maybeSuccess m
states=
M.fromList [( "Chris","Texas")]
expected' =
Just(Section (Header"states" )
states)
print m
r' `shouldBe` expected'

CHAPTER 24. PARSER COMBINATORS 1459
describe "INI parsing" $
it"Can parse multiple sections" $ do
letm=
parseByteString parseIni
mempty sectionEx''
r'=maybeSuccess m
sectionValues =
M.fromList
[ ("alias","claw")
, ("host","wikipedia.org" )]
whatisitValues =
M.fromList
[("red","intoothandclaw" )]
expected' =
Just(Config
(M.fromList
[ (Header"section"
, sectionValues)
, (Header"whatisit"
, whatisitValues)]))
print m
r' `shouldBe` expected'
We leave it to you to run this and experiment with it.

CHAPTER 24. PARSER COMBINATORS 1460
24.8 Character and token parsers
All right, that was a lot of code. Let’s all step back and take a
deep breath.
You probably have some idea by now of what we mean by
tokenizing, but the time has come for more detail. Tokeniza-
tion is a handy parsing tactic, so it’s baked into some of the
library functions we’ve been using. It’s worth diving in and
exploring what it means.
Traditionally, parsing has been done in two stages, lexing
and parsing. Characters from a stream will be fed into the
lexer, which will then emit tokens on demand to the parser
until it has no more to emit.9The parser then structures the
stream of tokens into a tree, commonly called an “abstract
syntax tree” or AST:
-- hand-wavy types: Stream because
-- production-grade parsers in Haskell
-- won't use [] for performance reasons
lexer::StreamChar->StreamToken
parser::StreamToken->AST
Lexers are simpler, typically performing parses that don’t
require looking ahead into the input stream by more than
9Lexers and tokenizers are similar, separating a stream of text into tokens based on
indicators such as whitespace or newlines; lexers often attach some context to the tokens,
where tokenizers typically do not.

CHAPTER 24. PARSER COMBINATORS 1461
one character or token at a time. Lexers are at times called
tokenizers. Lexing is sometimes done with regular expres-
sions, but a parsing library in Haskell will usually intend that
you do your lexing and parsing with the same API. Lexers (or
tokenizers) and parsers have a lot in common, being primarily
diﬀerentiated by their purpose and class of grammar.10
Insert tokens to play
Let’s play around with some things to see what tokenizing
does for us:
Prelude> parseString (some digit) mempty "123 456"
Success "123"
Prelude> parseString (some (some digit)) mempty "123 456"
Success ["123"]
Prelude> parseString (some integer) mempty "123"
Success [123]
Prelude> parseString (some integer) mempty "123456"
Success [123456]
The problem here is that if we wanted to recognize 123 and
456 as independent strings, we need some kind of separator.
Now we can go ahead and do that manually, but the tokenizers
10Formal grammars — rules for generating strings in a formal language — are placed
in a hierarchy, often called the Chomsky hierarchy after the linguist Noam Chomsky.

CHAPTER 24. PARSER COMBINATORS 1462
inparsers can do it for you too, also handling a mixture of
whitespace and newlines:
Prelude> parseString (some integer) mempty "123 456"
Success [123,456]
Prelude> parseString (some integer) mempty "123\n\n 456"
Success [123,456]
Or even space and newlines interleaved:
Prelude> parseString (some integer) mempty "123 \n \n 456"
Success [123,456]
But simply applying tokentodigitdoesn’t do what you
think:
Prelude> let s = "123 \n \n 456"
Prelude> parseString (token (some digit)) mempty s
Success "123"
Prelude> parseString (token (some (token digit))) mempty s
Success "123456"
Prelude> parseString (some decimal) mempty s
Success [123]
Prelude> parseString (some (token decimal)) mempty s
Success [123,456]
Compare that to the integer function, which is already a
tokenizer:

CHAPTER 24. PARSER COMBINATORS 1463
Prelude> parseString (some integer) mempty "1\n2\n 3\n"
Success [1,2,3]
We can write a tokenizing parser like some integer like this:
p'::Parser[Integer]
p'=some$ do
i<-token (some digit)
return (read i)
And we can compare the output of that to the output of
applying tokentodigit:
Prelude> let s = "1\n2\n3"
Prelude> parseString p' mempty s
Success [1,2,3]
Prelude> parseString (token (some digit)) mempty s
Success "1"
Prelude> parseString (some (token (some digit))) mempty s
Success ["1","2","3"]
You’ll want to think carefully about the scope at which
you’re tokenizing as well:
Prelude> let tknWhole = token $ char 'a' >> char 'b'
Prelude> parseString tknWhole mempty "a b"
Failure (interactive):1:2: error: expected: "b"

CHAPTER 24. PARSER COMBINATORS 1464
a b<EOF>
^
Prelude> parseString tknWhole mempty "ab ab"
Success 'b'
Prelude> parseString (some tknWhole) mempty "ab ab"
Success "bb"
If we wanted that first example to work, we need to tokenize
the parse of the first character, not the whole a-then-b parse:
Prelude> let tknCharA = (token (char 'a')) >> char 'b'
Prelude> parseString tknCharA mempty "a b"
Success 'b'
Prelude> parseString (some tknCharA) mempty "a ba b"
Success "bb"
Prelude> parseString (some tknCharA) mempty "a b a b"
Success "b"
The last example stops at the first 𝑎 𝑏parse because the
parser doesn’t say anything about a space after 𝑏and the tok-
enization behavior only applies to what followed 𝑎. We can
tokenize both character parsers though:
Prelude> let tknBoth = token (char 'a') >> token (char 'b')
Prelude> parseString (some tknBoth) mempty "a b a b"
Success "bb"

CHAPTER 24. PARSER COMBINATORS 1465
A mild warning: don’t get too tokenization happy. Try to
make it coarse-grained and selective. Overuse of tokenizing
parsersormixturewithcharacterparserscanmakeyourparser
slow or hard to understand. Use your judgment. Keep in mind
that tokenization isn’t exclusively about whitespace; it’s about
ignoring noise so you can focus on the structures you are
parsing.
24.9 Polymorphic parsers
If we take the time to assert polymorphic types for our parsers,
we can get parsers that can be run using attoparsec ,trifecta ,
parsec, or anything else that has implemented the necessary
typeclasses. Let’s give it a whirl, shall we?
{-# LANGUAGE OverloadedStrings #-}
moduleText.Fractions where
importControl.Applicative
importData.Attoparsec.Text (parseOnly )
importData.Ratio ((%))
importData.String (IsString )
importText.Trifecta

CHAPTER 24. PARSER COMBINATORS 1466
badFraction ::IsString s=>s
badFraction ="1/0"
alsoBad ::IsString s=>s
alsoBad ="10"
shouldWork ::IsString s=>s
shouldWork ="1/2"
shouldAlsoWork ::IsString s=>s
shouldAlsoWork ="2/1"
parseFraction ::(Monadm,TokenParsing m)
=>mRational
parseFraction = do
numerator <-decimal
_ <-char'/'
denominator <-decimal
casedenominator of
0->fail"Denominator cannot be zero"
_ ->return (numerator %denominator)
We’ve left some typeclass-constrained polymorphism in
our type signatures for flexibility. Our mainwill run both
attoparsec andtrifecta versions for us so we can compare the
outputs directly:

CHAPTER 24. PARSER COMBINATORS 1467
main::IO()
main= do
-- parseOnly is Attoparsec
letattoP=parseOnly parseFraction
print$attoP badFraction
print$attoP shouldWork
print$attoP shouldAlsoWork
print$attoP alsoBad
-- parseString is Trifecta
letp f i=
parseString f mempty i
print$p parseFraction badFraction
print$p parseFraction shouldWork
print$p parseFraction shouldAlsoWork
print$p parseFraction alsoBad
Prelude> main
Left "Failed reading: Denominator cannot be zero"
Right (1 % 2)
Right (2 % 1)
Left "\"/\": not enough input"
Failure (interactive):1:4: error: Denominator
cannot be zero, expected: digit
1/0<EOF>

CHAPTER 24. PARSER COMBINATORS 1468
^
Success (1 % 2)
Success (2 % 1)
Failure (interactive):1:3: error: unexpected
EOF, expected: "/", digit
10<EOF>
^
See what we meant earlier about the error messages?
It’s not perfect and could bite you
While the polymorphic parser combinators in the parsers li-
brary enable you to write parsers which can then be run with
various parsing libraries, this doesn’t free you of understand-
ing the particularities of each. In general, trifecta tries to
matchparsec ’s behaviors in most respects, the latter of which
is more extensively documented.
Failure and backtracking
Returning to our cursor model of parsers, backtracking is
returning the cursor to where it was before a failing parser
consumed input. In some cases, it can be a little confusing
to debug the same error in two diﬀerent runs of the same
parser doing essentially the same things in trifecta ,parsec,

CHAPTER 24. PARSER COMBINATORS 1469
andattoparsec , but the errors themselves might be diﬀerent.
Let’s consider an example of this.
{-# LANGUAGE OverloadedStrings #-}
We use OverloadedStrings so that we can use string literals as
if they were ByteStrings when testing attoparsec :
moduleBTwhere
importControl.Applicative
import qualified Data.Attoparsec.ByteString
asA
importData.Attoparsec.ByteString
(parseOnly )
importData.ByteString (ByteString )
importText.Trifecta hiding(parseTest )
importText.Parsec (Parsec,parseTest )
trifP::Showa
=>Parsera
->String->IO()
trifPp i=
print$parseString p mempty i
Helper function to run a trifecta parser and print the result:

CHAPTER 24. PARSER COMBINATORS 1470
parsecP ::(Showa)
=>ParsecString()a
->String->IO()
parsecP =parseTest
Helper function to run a parsec parser and print the result:
attoP::Showa
=>A.Parsera
->ByteString ->IO()
attoPp i=
print$parseOnly p i
Helper function for attoparsec — same deal as before:
nobackParse ::(Monadf,CharParsing f)
=>fChar
nobackParse =
(char'1'>>char'2')
<|>char'3'
Here’s our first parser. It attempts to parse '1'followed by
'2'or'3'. This parser does not backtrack:

CHAPTER 24. PARSER COMBINATORS 1471
tryParse ::(Monadf,CharParsing f)
=>fChar
tryParse =
try (char '1'>>char'2')
<|>char'3'
This parser has similar behavior to the previous one, except
it backtracks if the first parse fails. Backtracking means that the
input cursor returns to where it was before the failed parser
consumed input.
main::IO()
main= do
-- trifecta
trifP nobackParse "13"
trifP tryParse "13"
-- parsec
parsecP nobackParse "13"
parsecP tryParse "13"
-- attoparsec
attoP nobackParse "13"
attoP tryParse "13"
The error messages you get from each parser are going
to vary a bit. This isn’t because they’re wildly diﬀerent, but

CHAPTER 24. PARSER COMBINATORS 1472
is mostly due to how they attribute errors. You should see
something like:
Prelude> main
Failure (interactive):1:2:
error: expected: "2"
13<EOF>
^
Failure (interactive):1:1: error:
expected: "3"
13<EOF>
^
parse error at (line 1, column 2):
unexpected "3"
expecting "2"
parse error at (line 1, column 2):
unexpected "3"
expecting "2"
Left "\"3\": satisfyElem"
Left "\"3\": satisfyElem"
Conversely, if you try the valid inputs "12"and"3"with
nobackParse and each of the three parsers, you should see all of
them succeed.
This can be confusing. When you add backtracking to a
parser, error attribution can become more complicated at

CHAPTER 24. PARSER COMBINATORS 1473
times. To avoid this, consider using the <?>operator to anno-
tate parse rules any time you use try.11
tryAnnot ::(Monadf,CharParsing f)
=>fChar
tryAnnot =
(try (char '1'>>char'2')
<?>"Tried 12" )
<|>(char'3'<?>"Tried 3" )
Then running this in the REPL:
Prelude> trifP tryAnnot "13"
Failure (interactive):1:1: error: expected: Tried 12,
Tried 3
13<EOF>
^
Now the error will list the parses it attempted before it failed.
You’ll want to make the annotations more informative than
what we demonstrated in your own parsers.
11Parsec “try a <|> b” considered harmful; Edward Z. Yang

CHAPTER 24. PARSER COMBINATORS 1474
24.10 Marshalling from an AST to a
datatype
Fair warning: This section relies on a little more background
knowledge from you than previous sections have. If you are
not a person who already has some programming experience,
the following may not seem terribly useful to you, and there
may be some unfamiliar terminology and concepts.
The act of parsing, in a sense, is a means of necking down
the cardinality of our inputs to the set of things our programs
have a sensible answer for. It’s unlikely you can do some-
thing meaningful and domain-specific when your input type
isString ,Text, orByteString . However, if you can parse one of
those types into something structured, rejecting bad inputs,
then you might be able to write a proper program. One of the
mistakes programmers make in writing programs handling
text is in allowing their data to stay in the textual format, doing
mind-bending backflips to cope with the unstructured nature
of textual inputs.
In some cases, the act of parsing isn’t enough. You might
have a sort of AST or structured representation of what was
parsed, but from there, you might expect that AST or repre-
sentation to take a particular form. This means we want to
narrow the cardinality and get even more specific about how
our data looks. Often this second step is called unmarshalling

CHAPTER 24. PARSER COMBINATORS 1475
our data. Similarly, marshalling is the act of preparing data
for serialization, whether via memory alone (foreign function
interface boundary) or over a network interface.
The whole idea here is that you have two pipelines for your
data:
Text->Structure ->Meaning
-- parse -> unmarshall
Meaning ->Structure ->Text
-- marshall -> serialize
There isn’t only one way to accomplish this, but we’ll show
you a commonly used library and how it has this two-stage
pipeline in the API.
Marshalling and unmarshalling JSON data
aesonis presently the most popular JSON12library in Haskell.
One of the things that’ll confuse programmers coming to
Haskell from Python, Ruby, Clojure, JavaScript, or similar
languages, is that there’s usually no unmarshall/marshall step.
Instead, the raw JSON AST will be represented directly as an
untyped blob of data. Users of typed languages are more likely
12JSON stands for JavaScript Object Notation. JSON is, for better or worse, a very
common open-standard data format used to transmit data, especially between browsers
and servers. As such, dealing with JSON is a common programming task, so you might
as well get used to it now.

CHAPTER 24. PARSER COMBINATORS 1476
to have encountered something like this. We’ll be using aeson
0.10.0.0 for the following examples.
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE QuasiQuotes #-}
moduleMarshalling where
importData.Aeson
importData.ByteString.Lazy (ByteString )
importText.RawString.QQ
sectionJson ::ByteString
sectionJson =[r|
{"section" :{"host":"wikipedia.org" },
"whatisit" :{"red":"intoothandclaw" }
}
|]
Notethatwe’resayingthetypeof sectionJson isalazy ByteString .
If you get strict and lazy ByteString types mixed up you’ll get
errors.
Provided a strict ByteString when a lazy one was expected:
<interactive>:10:8:
Couldn't match expected type

CHAPTER 24. PARSER COMBINATORS 1477
Data.ByteString.Lazy.Internal.ByteString
with actual type ByteString
NB:
Data.ByteString.Lazy.Internal.ByteString
is defined in
Data.ByteString.Lazy.Internal
ByteString
is defined in
Data.ByteString.Internal
The actual type is what we provided; the expected type is
what the types wanted. The NB:in the type error stands for
nota bene. Either we used the wrong code (so expected type
needs to change), or we provided the wrong values (actual
type, our types/values, need to change). You can reproduce
this error by making the following mistake in the marshalling
module:
-- Change the import of the ByteString
-- type constructor from:
importData.ByteString.Lazy (ByteString )
-- Into:
importData.ByteString (ByteString )

CHAPTER 24. PARSER COMBINATORS 1478
Provided a lazy ByteString when a strict one was expected:
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE QuasiQuotes #-}
moduleWantedStrict where
importData.Aeson
importData.ByteString.Lazy (ByteString )
importText.RawString.QQ
sectionJson ::ByteString
sectionJson =[r|
{"section" :{"host":"wikipedia.org" },
"whatisit" :{"red":"intoothandclaw" }
}
|]
main= do
letblah::MaybeValue
blah=decodeStrict sectionJson
print blah
You’ll get the following type error if you load that up:
code/wantedStrictGotLazy.hs:19:27:

CHAPTER 24. PARSER COMBINATORS 1479
Couldn't match expected type
‘Data.ByteString.Internal.ByteString’
with actual type ‘ByteString’
NB:
‘Data.ByteString.Internal.ByteString’
is defined in ‘Data.ByteString.Internal’
‘ByteString’ is defined in ‘Data.ByteString.Lazy.Internal’
In the first argument of ‘decodeStrict’,
namely ‘sectionJson’
In the expression: decodeStrict sectionJson
The more useful information is in the NB:or nota bene,
where the internal modules are mentioned. The key is to re-
member actual type means “your code”, expected type means
“what they expected,” and that the ByteString module that
doesn’t have Lazyin the name is the strict version. We can
modify our code a bit to get nicer type errors:

CHAPTER 24. PARSER COMBINATORS 1480
-- replace the (ByteString)
-- import with these
import qualified Data.ByteString asBS
import qualified Data.ByteString.Lazy
asLBS
-- edit the type sig for this one
sectionJson ::LBS.ByteString
Then we’ll get the following type error instead:
Couldn't match expected type ‘BS.ByteString’
with actual type ‘LBS.ByteString’
NB: ‘BS.ByteString’ is defined in
‘Data.ByteString.Internal’
‘LBS.ByteString’ is defined in
‘Data.ByteString.Lazy.Internal’
In the first argument of ‘decodeStrict’,
namely ‘sectionJson’
In the expression: decodeStrict sectionJson
This is helpful because we have both versions available as
qualified modules. You may not always be so fortunate and
will need to remember which is which.

CHAPTER 24. PARSER COMBINATORS 1481
Back to the JSON
Let’s get back to handling JSON. The most common functions
for using aesonare the following:
Prelude> import Data.Aeson
Prelude> :t encode
encode :: ToJSON a => a -> LBS.ByteString
Prelude> :t decode
decode :: FromJSON a => LBS.ByteString -> Maybe a
These functions are sort of eliding the intermediate step
that passes through the Value type in aeson, which is a datatype
JSON AST — “sort of,” because you can decode the raw JSON
data into a Valueanyway:
Prelude> decode sectionJson :: Maybe Value
Just (Object (fromList [
("whatisit",
Object (fromList [("red",
String "intoothandclaw")])),
("section",
Object (fromList [("host",
String "wikipedia.org")]))]))
Not, uh, super pretty. We’ll figure out something nicer in
a moment. Also do not forget to assert a type, or the type-
defaulting in GHCi will do silly things:

CHAPTER 24. PARSER COMBINATORS 1482
Prelude> decode sectionJson
Nothing
Now what if we do want a nicer representation for this JSON
noise? Well, let’s define our datatypes and see if we can decode
the JSON into our type:

CHAPTER 24. PARSER COMBINATORS 1483
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE QuasiQuotes #-}
moduleMarshalling where
importControl.Applicative
importData.Aeson
importData.ByteString.Lazy (ByteString )
import qualified Data.Text asT
importData.Text (Text)
importText.RawString.QQ
sectionJson ::ByteString
sectionJson =[r|
{"section" :{"host":"wikipedia.org" },
"whatisit" :{"red":"intoothandclaw" }
}
|]
dataTestData =
TestData {
section ::Host
, what::Color
}deriving (Eq,Show)
newtype Host=
HostString
deriving (Eq,Show)
typeAnnotation =String
dataColor=
RedAnnotation
|BlueAnnotation
|YellowAnnotation
deriving (Eq,Show)
main= do
letd::MaybeTestData
d=decode sectionJson
print d

CHAPTER 24. PARSER COMBINATORS 1484
This will in fact net you a type error complaining about
there not being an instance of FromJSON forTestData . Which is
true! GHC has no idea how to unmarshall JSON data (in the
form of a Value) into a TestData value. Let’s add an instance so
it knows how:

CHAPTER 24. PARSER COMBINATORS 1485
instance FromJSON TestData where
parseJSON ( Objectv)=
TestData <$>v.:"section"
<*>v.:"whatisit"
parseJSON _ =
fail"Expected an object for TestData"
instance FromJSON Hostwhere
parseJSON ( Objectv)=
Host<$>v.:"host"
parseJSON _ =
fail"Expected an object for Host"
instance FromJSON Colorwhere
parseJSON ( Objectv)=
(Red<$>v.:"red")
<|>(Blue<$>v.:"blue")
<|>(Yellow<$>v.:"yellow" )
parseJSON _ =
fail"Expected an object for Color"
Also note that you can use quasiquotes to avoid having to
escape quotation marks in the REPL as well:
Prelude> :set -XOverloadedStrings
Prelude> decode "{\"blue\": \"123\"}" :: Maybe Color

CHAPTER 24. PARSER COMBINATORS 1486
Just (Blue "123")
Prelude> :set -XQuasiQuotes
Prelude> decode [r|{"red": "123"}|] :: Maybe Color
Just (Red "123")
To relate what we just did back to the relationship between
parsing and marshalling, the idea is that our FromJSON instance
is accepting the Valuetype and ToJSON instances generate the
Valuetype, closing the following loop:
-- FromJSON
ByteString ->Value->yourType
-- parse -> unmarshall
-- ToJSON
yourType ->Value->ByteString
-- marshall -> serialize
The definition of Valueat time of writing is the following:

CHAPTER 24. PARSER COMBINATORS 1487
-- | A JSON value represented
-- as a Haskell value.
dataValue=Object!Object
|Array!Array
|String!Text
|Number!Scientific
|Bool!Bool
|Null
deriving (Eq,Read,Show,
Typeable ,Data)
What if we want to unmarshall something that could be a
Number or aString ?
dataNumberOrString =
NumbaInteger
|Stringy Text
deriving (Eq,Show)
instance FromJSON NumberOrString where
parseJSON ( Numberi)=return$Numbai
parseJSON ( Strings)=return$Stringy s
parseJSON _ =
fail"NumberOrString must \
\be number or string"

CHAPTER 24. PARSER COMBINATORS 1488
This won’t quite work at first. The trouble is that JSON (and
JavaScript, as it happens) only has one numeric type and that
type is a IEEE-754 float. JSON (and JavaScript, terrifyingly)
have no integral types or integers, so aesonhas to pick one
representation that works for all possible JSON numbers. The
most precise way to do that is the Scientific type which is an
arbitrarily precise numerical type (you may remember this
from way back in Chapter 4, Basic Datatypes). So we need to
convert from a Scientific to anInteger :
importControl.Applicative
importData.Aeson
importData.ByteString.Lazy (ByteString )
import qualified Data.Text asT
importData.Text (Text)
importText.RawString.QQ
importData.Scientific (floatingOrInteger )
dataNumberOrString =
NumbaInteger
|Stringy Text
deriving (Eq,Show)

CHAPTER 24. PARSER COMBINATORS 1489
instance FromJSON NumberOrString where
parseJSON ( Numberi)=
casefloatingOrInteger i of
(Left_)->
fail"Must be integral number"
(Rightinteger) ->
return$Numbainteger
parseJSON ( Strings)=return$Stringy s
parseJSON _ =
fail"NumberOrString must \
\be number or string"
-- so it knows what we want to parse
dec::ByteString
->MaybeNumberOrString
dec=decode
eitherDec ::ByteString
->EitherStringNumberOrString
eitherDec =eitherDecode
main= do
print$dec"blah"
Now let’s give it a whirl:

CHAPTER 24. PARSER COMBINATORS 1490
Prelude> main
Nothing
Butwhathappened? Wecanrewritethecodetouse eitherDec
to get a slightly more helpful type error:
main= do
print$dec"blah"
print$eitherDec "blah"
Then reloading the code and trying again in the REPL:
Prelude> main
Nothing
Left "Error in $: Failed reading:
not a valid json value"
By that means, we are able to get more informative errors
fromaeson. If we wanted some examples that worked, we
could try things like the following:
Prelude> dec "123"
Just (Numba 123)
Prelude> dec "\"blah\""
Just (Stringy "blah")
It’s worth getting comfortable with aesoneven if you don’t
plan to work with much JSON because many serialization

CHAPTER 24. PARSER COMBINATORS 1491
libraries in Haskell follow a similar API pattern. Play with the
example and see how you need to change the type of decto
be able to parse a list of numbers or strings.
24.11 Chapter Exercises
1.Write a parser for semantic versions as defined by http:
//semver.org/ . After making a working parser, write an Ord
instance for the SemVer type that obeys the specification
outlined on the SemVer website.

CHAPTER 24. PARSER COMBINATORS 1492
-- Relevant to precedence/ordering,
-- cannot sort numbers like strings.
dataNumberOrString =
NOSSString
|NOSIInteger
typeMajor=Integer
typeMinor=Integer
typePatch=Integer
typeRelease =[NumberOrString ]
typeMetadata =[NumberOrString ]
dataSemVer=
SemVerMajorMinorPatchRelease Metadata
parseSemVer ::ParserSemVer
parseSemVer =undefined
Expected results:
Prelude> parseString parseSemVer mempty "2.1.1"
Success (SemVer 2 1 1 [] [])
Prelude> parseString parseSemVer mempty "1.0.0-x.7.z.92"
Success (SemVer 1 0 0
[NOSS "x", NOSI 7, NOSS "z", NOSI 92] [])

CHAPTER 24. PARSER COMBINATORS 1493
Prelude> SemVer 2 1 1 [] [] > SemVer 2 1 0 [] []
True
2.Write a parser for positive integer values. Don’t reuse the
pre-existing digitorinteger functions, but you can use
the rest of the libraries we’ve shown you so far. You are
not expected to write a parsing library from scratch.
parseDigit ::ParserChar
parseDigit =undefined
base10Integer ::ParserInteger
base10Integer =undefined
Expected results:
Prelude> parseString parseDigit mempty "123"
Success '1'
Prelude> parseString parseDigit mempty "abc"
Failure (interactive):1:1: error: expected: parseDigit
abc<EOF>
^
Prelude> parseString base10Integer mempty "123abc"
Success 123
Prelude> parseString base10Integer mempty "abc"
Failure (interactive):1:1: error: expected: integer

CHAPTER 24. PARSER COMBINATORS 1494
abc<EOF>
^
Hint: Assume you’re parsing base-10 numbers. Use arith-
metic as a cheap “accumulator” for your final number as
you parse each digit left-to-right.
3.Extend the parser you wrote to handle negative and pos-
itive integers. Try writing a new parser in terms of the
one you already have to do this.
Prelude> parseString base10Integer' mempty "-123abc"
Success (-123)
4.Write a parser for US/Canada phone numbers with vary-
ing formats.

CHAPTER 24. PARSER COMBINATORS 1495
-- aka area code
typeNumberingPlanArea =Int
typeExchange =Int
typeLineNumber =Int
dataPhoneNumber =
PhoneNumber NumberingPlanArea
Exchange LineNumber
deriving (Eq,Show)
parsePhone ::ParserPhoneNumber
parsePhone =undefined
With the following behavior:
Prelude> parseString parsePhone mempty "123-456-7890"
Success (PhoneNumber 123 456 7890)
Prelude> parseString parsePhone mempty "1234567890"
Success (PhoneNumber 123 456 7890)
Prelude> parseString parsePhone mempty "(123) 456-7890"
Success (PhoneNumber 123 456 7890)
Prelude> parseString parsePhone mempty "1-123-456-7890"
Success (PhoneNumber 123 456 7890)
Cf. Wikipedia’s article on “National conventions for writ-
ing telephone numbers”. You are encouraged to adapt the

CHAPTER 24. PARSER COMBINATORS 1496
exercise to your locality’s conventions if they are not part
of the NNAP scheme.
5.Write a parser for a log file format and sum the time
spent in each activity. Additionally, provide an alterna-
tive aggregation of the data that provides average time
spent per activity per day. The format supports the use
of comments which your parser will have to ignore. The
#characters followed by a date mark the beginning of a
particular day.
Log format example:
-- wheee a comment
# 2025-02-05
08:00 Breakfast
09:00 Sanitizing moisture collector
11:00 Exercising in high-grav gym
12:00 Lunch
13:00 Programming
17:00 Commuting home in rover
17:30 R&R
19:00 Dinner
21:00 Shower
21:15 Read
22:00 Sleep

CHAPTER 24. PARSER COMBINATORS 1497
# 2025-02-07 -- dates not nececessarily sequential
08:00 Breakfast -- should I try skippin bfast?
09:00 Bumped head, passed out
13:36 Wake up, headache
13:37 Go to medbay
13:40 Patch self up
13:45 Commute home for rest
14:15 Read
21:00 Dinner
21:15 Read
22:00 Sleep
You are to derive a reasonable datatype for represent-
ing this data yourself. For bonus points, make this bi-
directionalbymakingaShowrepresentationforthedatatype
which matches the format you are parsing. Then write a
generator for this data using QuickCheck’s Gen and see if
you can break your parser with QuickCheck.
6.Write a parser for IPv4 addresses.
importData.Word
dataIPAddress =
IPAddress Word32
deriving (Eq,Ord,Show)

CHAPTER 24. PARSER COMBINATORS 1498
A 32-bit word is a 32-bit unsigned int. Lowest value is 0
rather than being capable of representing negative num-
bers, but the highest possible value in the same number
of bits is twice as high. Note:
Prelude> import Data.Int
Prelude> import Data.Word
Prelude> maxBound :: Int32
2147483647
Prelude> maxBound :: Word32
4294967295
Prelude> div 4294967295 2147483647
2
Word32 is an appropriate and compact way to represent
IPv4 addresses. You are expected to figure out not only
how to parse the typical IP address format, but how IP
addresses work numerically insofar as is required to write
a working parser. This will require using a search engine
unless you have an appropriate book on internet network-
ing handy.
Example IPv4 addresses and their decimal representa-
tions:
172.16.254.1 -> 2886794753
204.120.0.15 -> 3430416399

CHAPTER 24. PARSER COMBINATORS 1499
7.Same as before, but IPv6.
importData.Word
dataIPAddress6 =
IPAddress6 Word64Word64
deriving (Eq,Ord,Show)
Example IPv6 addresses and their decimal representa-
tions:
0:0:0:0:0:ffff:ac10:fe01 -> 281473568538113
0:0:0:0:0:ffff:cc78:f -> 281474112159759
FE80:0000:0000:0000:0202:B3FF:FE1E:8329 ->
338288524927261089654163772891438416681
2001:DB8::8:800:200C:417A ->
42540766411282592856906245548098208122
One of the trickier parts about IPv6 will be full vs. col-
lapsed addresses and the abbrevations. See this Q&A
thread13about IPv6 abbreviations for more.
Ensure you can parse abbreviated variations of the earlier
examples like:
13http://answers.google.com/answers/threadview/id/770645.html

CHAPTER 24. PARSER COMBINATORS 1500
FE80::0202:B3FF:FE1E:8329
2001:DB8::8:800:200C:417A
8.Removethederived ShowinstancesfromtheIPAddress/IPAd-
dress6 types, and write your own Showinstance for each
type that renders in the typical textual format appropriate
to each.
9.Write a function that converts between IPAddress and
IPAddress6.
10.Write a parser for the DOT language14that Graphviz uses
to express graphs in plain-text.
We suggest you look at the AST datatype in Haphviz15for
ideas on how to represent the graph in a Haskell datatype.
If you’re feeling especially robust, you can try using fgl16.
24.12 Definitions
1.A parser parses.
You read the chapter right?
2.A parser combinator combines two or more parsers to
produce a new parser. Good examples of this are things
14http://www.graphviz.org/doc/info/lang.html
15http://hackage.haskell.org/package/haphviz
16http://hackage.haskell.org/package/fgl

CHAPTER 24. PARSER COMBINATORS 1501
like using <|>fromAlternative to produce a new parser
from the disjunction of two parser arguments to <|>. Or
some. Ormany. Ormappend . Or(>>).
3.Marshalling is transforming a potentially nonlinear rep-
resentation of data in memory into a format that can be
stored on disk or transmitted over a network socket. Go-
ing in the opposite direction is called unmarshalling. Cf.
serialization and deserialization.
4.A token(izer) converts text, usually a stream of characters,
into more meaningful or “chunkier” structures such as
words, sentences, or symbols. The linesandwordsfunc-
tions you’ve used earlier in this book are like very unso-
phisticated tokenizers.
5.Lexer — see tokenizer.
24.13 Follow-up resources
1.Parsec try a-or-b considered harmful; Edward Z. Yang
2.Code case study: parsing a binary data format; Real World
Haskell
3.The Parsec parsing library; Real World Haskell

CHAPTER 24. PARSER COMBINATORS 1502
4.An introduction to parsing text in Haskell with Parsec;
James Wilson;
http://unbui.lt/#!/post/haskell-parsec-basics
5.Parsing CSS with Parsec; Jakub Arnold
6.Parsec: A practical parser library; Daan Leijen, Erik Mei-
jer;
http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.24.
5200
7.How to Replace Failure by a List of Successes; Philip
Wadler;
http://dl.acm.org/citation.cfm?id=5288
8.How to Replace Failure by a Heap of Successes; Edward
Kmett
9.Two kinds of backtracking; Samuel Gélineau (gelisam);
http://gelisam.blogspot.ca/2015/09/two-kinds-of-backtracking.
html
10.LL and LR in Context: Why Parsing Tools Are Hard; Josh
Haberman
http://blog.reverberate.org/2013/09/ll-and-lr-in-context-why-parsing-tools.
html
11.Parsing Techniques, a practical guide; second edition;
Grune & Jacobs

CHAPTER 24. PARSER COMBINATORS 1503
12.Parsing JSON with Aeson; School of Haskell
13.aeson; 24 days of Hackage; Oliver Charles

Chapter 25
Composing types
The last thing one
discovers in composing a
work is what to put first.
T. S. Eliot
1504

CHAPTER 25. E PLURIBUS MONAD 1505
25.1 Composing types
This chapter and the next are about monad transformers, both
the principles behind them and the practicalities of using them.
For many programmers, monad transformers are indistin-
guishable from magick, so we want to approach them from
both angles and demonstrate that they are both comprehen-
sible via their types and practical in normal programming.
Functors and applicatives are both closed under composi-
tion: this means that you can compose two functors (or two
applicatives) and return another functor (or applicative, as the
case may be). This is not true of monads, however; when you
compose two monads, the result is not necessarily another
monad. We will see this soon.
However, there are times when composing monads is desir-
able. Diﬀerent monads allow us to work with diﬀerent eﬀects.
Composing monads allows you to build up computations with
multiple eﬀects. By stacking, for example, a Maybemonad with
anIO, you can be performing IOactions while also building up
computations that have a possibility of failure, handled by the
Maybemonad.
A monad transformer is a variant of an ordinary type that
takes an additional type argument which is assumed to have a
Monadinstance. For example, MaybeT is the transformer variant
of theMaybetype. The transformer variant of a type gives us

CHAPTER 25. E PLURIBUS MONAD 1506
aMonadinstance that binds over both bits of structure. This
allows us to compose monads and combine their eﬀects. Get-
ting comfortable with monad transformers is important to
becoming proficient in Haskell, so we’re going to take it pretty
slowly and go step by step. You won’t necessarily want to start
out early on defining a bunch of transformer stacks yourself,
but familiarity with them will help a great deal in using other
people’s libraries.
In this chapter, we will
•demonstrate why composing two monads does not give
you another monad;
•examine the Identity andCompose types;
•manipulate types until we can make monads compose;
•meet some common monad transformers;
•work through an Identity crisis.
25.2 Common functions as types
We’ll start in a place that may seem a little strange and point-
less at first, with newtypes that correspond to some very basic
functions. We can construct types that are like those func-
tions because we have types that can take arguments — that

CHAPTER 25. E PLURIBUS MONAD 1507
is, type constructors. In particular, we’ll be using types that
correspond to idand(.).
You’ve seen some of the types we’re going to use in the
following sections before, but we’ll be putting them to some
novel uses. The idea here is to use these datatypes as helpers in
order to demonstrate the problems with composing monads,
and we’ll see how these type constructors can also serve as
monad transformers, because a monad transformer is a type
constructor that takes a monad as an argument.
Identity is boring
You’ve seen this type in previous chapters, sometimes as a
datatype and sometimes as a newtype. We’ll construct the
type diﬀerently this time, as a newtype with a helper function
of the sort we saw in Reader andState:
newtype Identity a=
Identity { runIdentity ::a }
We’ll be using the newtype in this chapter because the
monad transformer version, IdentityT , is usually written as
a newtype. The use of the prefixes runorgetindicates that
these accessor functions are means of extracting the underly-
ing value from the type. There is no real diﬀerence in meaning
between runandget. You’ll see these accessor functions often,

CHAPTER 25. E PLURIBUS MONAD 1508
particularly with utility types like Identity or transformer vari-
ants reusing an original type.
Anoteaboutnewtypes Whilemonadtransformertypescould
be written using the datakeyword, they are most commonly
written as newtypes, and we’ll be sticking with that pattern
here. They are only newtyped to avoid unnecessary overhead,
as newtypes, as we recall, have an underlying representation
identical to the type they contain. The important thing is that
monad transformers are never sum or product types; they are
always a means of wrapping one extra layer of (monadic) struc-
ture around a type, so there is never a reason they couldn’t
be newtypes. Haskellers have a general tendency to avoid
adding additional runtime overhead if they can, so if they can
newtype it, they most often will.
Another thing we want to notice about Identity is the sim-
ilarity of the kind of our Identity type to the type of the id
function, although the fidelity of the comparison isn’t perfect
given the limitations of type-level computation in Haskell:
Prelude> :t id
id :: a -> a
Prelude> :k Identity
Identity :: * -> *
The kind signature of the type resembles the type signature

CHAPTER 25. E PLURIBUS MONAD 1509
of the function, which we hope isn’t too much of a surprise.
Fine, so far — not much new here. Yet.
Compose
We mentioned above that we can also construct a datatype
that corresponds to function composition.
Here is the Compose type. It should look to you much like
function composition, but in this case, the 𝑓and𝑔represent
type constructors, not term-level functions:
newtype Compose f g a=
Compose { getCompose ::f (g a) }
deriving (Eq,Show)
So, we have a type constructor that takes three type argu-
ments: 𝑓and𝑔must be type constructors themselves, while 𝑎
will be a concrete type (consider the relationship between type
constructors and term-level functions on the one hand, and
values and type constants on the other). As we did above, let’s
look at the kind of Compose — note the kinds of the arguments
to the type constructor:
Compose :: (* -> *) -> (* -> *) -> * -> *
Does that remind you of anything?
(.)::(b->c)->(a->b)->a->c

CHAPTER 25. E PLURIBUS MONAD 1510
So, what does that look like in practice? Something like this:
Prelude> Compose [Just 1, Nothing]
Compose {getCompose = [Just 1,Nothing]}
Prelude> let xs = [Just (1::Int), Nothing]
Prelude> :t Compose xs
Compose [Just (1 :: Int), Nothing]
:: Compose [] Maybe Int
Given the above value, the type variables get bound accord-
ingly:
Compose [Just(1::Int),Nothing]
Compose { getCompose ::f (g a) }
Compose []MaybeInt
f~[]
g~Maybe
a~Int
We have one bit of structure wrapped around another, then
a value type (the 𝑎) because the whole thing still has to be kind
*in the end.
We’ve made the point in previous chapters that type con-
structors are functions. Type constructors can take other type

CHAPTER 25. E PLURIBUS MONAD 1511
constructors as arguments, too, just as functions can take other
functions as arguments. This is what allows us to compose
types.
25.3 Two little functors sittin’ in a tree,
L-I-F-T-I-N-G
Let’s start with composing functors, using the types we saw
above. We know we can lift over Identity ; you’ve seen this
Functor before:
instance Functor Identity where
fmap f ( Identity a)=Identity (f a)
Identity here gives us a sort of vanilla Functor that doesn’t do
anything interesting but captures the essence of what Functor is
about. The function gets lifted into the context of the Identity
type and then mapped over the 𝑎value.
It turns out we can get a Functor instance for Compose , too, if
we ask that the 𝑓and𝑔both have Functor instances:
instance (Functor f,Functor g)=>
Functor (Compose f g)where
fmap f ( Compose fga)=
Compose $(fmap.fmap) f fga

CHAPTER 25. E PLURIBUS MONAD 1512
Now the 𝑓and the 𝑔both have to be part of the structure
that we’re lifting over, so they both have to be Functor s them-
selves. We need to be able to jump over both those layers in
order to apply to the value that’s ultimately inside. We have
tofmaptwice to get to that value inside because of the layered
structures.
To return to the example we used above, we have this type:
newtype Compose f g a=
Compose { getCompose ::f (g a) }
deriving (Eq,Show)
Compose { getCompose ::f (g a) }
Compose []MaybeInt
And if we use our Functor instance, we can apply a function
to theIntvalue wrapped up in all that structure:
Prelude> let xs = [Just 1, Nothing]
Prelude> Compose xs
Compose {getCompose = [Just 1,Nothing]}
Prelude> fmap (+1) (Compose xs)
Compose {getCompose = [Just 2,Nothing]}

CHAPTER 25. E PLURIBUS MONAD 1513
We can generalize this to diﬀerent amounts of structure,
such as with one less bit of structure. You may remember this
from a previous chapter:
newtype Onef a=
One(f a)
deriving (Eq,Show)
instance Functor f=>
Functor (Onef)where
fmap f ( Onefa)=One$fmap f fa
Or one more layer of structure than Compose :
newtype Threef g h a =
Three(f (g (h a)))
deriving (Eq,Show)
instance (Functor f,Functor g,Functor h)
=>Functor (Threef g h)where
fmap f ( Threefgha)=
Three$(fmap.fmap.fmap) f fgha
As with the anonymous product (,)and the anonymous
sumEither, theCompose type allows us to express arbitrarily
nested types:

CHAPTER 25. E PLURIBUS MONAD 1514
v::Compose []
Maybe
(Compose Maybe[]Integer)
v=Compose [Just(Compose $Just[1])]
The way to think about this is that the composition of two
datatypes that have a Functor instance gives rise to a new Functor
instance. You’ll sometimes see people refer to this as functors
being closed under composition which means that when you
compose two Functor s, you get another Functor .
25.4 Twinplicative
You probably guessed this was our next step in Compose -landia.
Applicatives, it turns out, are also closed under composition.
We can compose two types that have Applicative instances and
get a new Applicative instance. But you’re going to write it.

CHAPTER 25. E PLURIBUS MONAD 1515
GOTCHA! Exercise time
-- instance types provided as
-- they may help.
{-# LANGUAGE InstanceSigs #-}
instance (Applicative f,Applicative g)
=>Applicative (Compose f g)where
pure::a->Compose f g a
pure=undefined
(<*>)::Compose f g (a->b)
->Compose f g a
->Compose f g b
(Compose f)<*>(Compose a)=undefined
We mentioned in an earlier chapter that Applicative is a
weaker algebra than Monad, and that sometimes there are bene-
fits to preferring an Applicative when you don’t need the full
power of the Monad. This is one of those benefits. To compose
Applicative s, you don’t need to do the legwork that Monads re-
quire in order to compose and still have a Monad. Oh, yes, right
— we still haven’t quite made it to monads composing, but
we’re about to.

CHAPTER 25. E PLURIBUS MONAD 1516
25.5 Twonad?
What about Monad? There’s no problem composing two arbi-
trary datatypes that have Monadinstances. We saw this already
when we used Compose withMaybeand list, which both have Monad
instances defined. However, the result of having done so does
not give you a Monad.
The issue comes down to a lack of information. Both types
Compose is working with are polymorphic, so when you try to
write bind for the Monad, you’re trying to combine two poly-
morphic binds into a single combined bind. This, it turns out,
is not possible:
{-# LANGUAGE InstanceSigs #-}
-- impossible.
instance (Monadf,Monadg)
=>Monad(Compose f g)where
return=pure
(>>=)::Compose f g a
->(a->Compose f g b)
->Compose f g b
(>>=)= ???

CHAPTER 25. E PLURIBUS MONAD 1517
These are the types we’re trying to combine, because 𝑓and
𝑔are necessarily both monads with their own Monadinstances:
Monadf=>f a->(a->f b)->f b
Monadg=>g a->(a->g b)->g b
From those, we are trying to write this bind:
(Monadf,Monadg)
=>f (g a) ->(a->f (g b)) ->f (g b)
Or formulated diﬀerently:
(Monadf,Monadg)
=>f (g (f (g a))) ->f (g a)
And this is not possible. There’s not a good way to jointhat
final𝑓and𝑔. It’s a great exercise to try to make it work, because
the barriers you’ll run into are instructive in their own right.
You can also read Composing monads1by Mark P. Jones and
Luc Duponcheel to see why it’s not possible.
No free burrito lunches
Since getting another Monadgiven the composition of two arbi-
trary types that have a Monadinstance is impossible, what can
we do to get a Monadinstance for combinations of types? The
1http://web.cecs.pdx.edu/~mpj/pubs/RR-1004.pdf

CHAPTER 25. E PLURIBUS MONAD 1518
answer is, monad transformers. We’ll get to that after a little
break for some exercises.
25.6 Exercises: Compose Instances
1.Write the Compose Foldable instance.
ThefoldMap = undefined bit is a hint to make it easier and
look more like what you’ve seen already.
instance (Foldable f,Foldable g)=>
Foldable (Compose f g)where
foldMap =undefined
2.Write the Compose Traversable instance.
instance (Traversable f,Traversable g)=>
Traversable (Compose f g)where
traverse =undefined
And now for something completely diﬀerent
This has nothing to do with anything else in this chapter, but
it makes for a fun exercise.

CHAPTER 25. E PLURIBUS MONAD 1519
classBifunctor pwhere
{-# MINIMAL bimap | first, second #-}
bimap::(a->b)
->(c->d)
->p a c
->p b d
bimap f g =first f .second g
first::(a->b)->p a c->p b c
first f =bimap f id
second::(b->c)->p a b->p a c
second=bimap id
It’s a functor that can map over two type arguments instead
of one. Write Bifunctor instances for the following types:
1.The less you think, the easier it’ll be.
dataDeuxa b=Deuxa b
2.dataConsta b=Consta
3.dataDreia b c=Dreia b c
4.dataSuperDrei a b c=SuperDrei a b

CHAPTER 25. E PLURIBUS MONAD 1520
5.dataSemiDrei a b c=SemiDrei a
6.dataQuadriceps a b c d =
Quadzzz a b c d
7.dataEithera b=
Lefta
|Rightb
25.7 Monad transformers
We’ve now seen what the problem with Monadis: you can put
two together but you don’t get a new Monadinstance out of it.
When we need to get a new Monadinstance, we need a monad
transformer. It’s not magic; the answer is in the types.
We said above that a monad transformer is a type construc-
tor that takes a Monadas an argument and returns a Monadas
a result. We also noted that the fundamental problem with
composing two monads lies in the impossibility of joining two
unknown monads. In order to make that joinhappen, we need
to reduce the polymorphism and get concrete information
about one of the monads that we’re working with. The other
monad remains polymorphic as a variable type argument to
our type constructor. Transformers help you make a monad
out of multiple (2, 3, 4...) types that each have a Monadinstance

CHAPTER 25. E PLURIBUS MONAD 1521
by wrapping around existing monads that provide each bit of
wanted functionality.
The types are tricky here, so we’re going to be walking
through writing monad transformers very slowly. Parts of
what follows may seem tedious, so work through it as slowly
or quickly as you need to.
Monadic stacking
Applicative allows us to apply functions of more than one ar-
gument in the presence of functorial structure, enabling us to
cope with this transition:
-- from this:
fmap(+1) (Just1)
-- to this:
(,,)
<$>Just1
<*>Just"lol"
<*>Just[1,2]
Sometimes we want a (>>=)which can address more than
oneMonadat once. You’ll often see this in applications that have
multiple things going on, such as a web app where combining
Reader andIOis common. You want IOso you can perform ef-
fectful actions like talking to a database and also Reader for the

CHAPTER 25. E PLURIBUS MONAD 1522
database connection(s) and/or HTTP request context. Some-
times you may even want multiple Readers (app-specific data
vs. what the framework provides by default), although usually
there’s a way to add only the data you want to a product type
of a single Reader .
So the question becomes, how do we get one big bind over
a type like the following?
IO(ReaderString[a])
-- where the Monad instances involved
-- are that of IO, Reader, and []
Doing it badly
We could make one-oﬀ types for each combination, but this
will get tiresome quickly. For example:
newtype MaybeIO a=
MaybeIO { runMaybeIO ::IO(Maybea) }
newtype MaybeList a=
MaybeList { runMaybeList ::[Maybea] }
We don’t need to resort to this; we can get a Monadfor two
types, as long as we know what one of the types is. Transform-

CHAPTER 25. E PLURIBUS MONAD 1523
ers are a means of avoiding making a one-oﬀ Monadfor every
possible combination of types.
25.8 IdentityT
Much as Identity helps show oﬀ the most basic essence of
Functor ,Applicative , andMonad,IdentityT is going to help you
begin to understand monad transformers. Using this type that
doesn’t have a lot of interesting stuﬀ going on with it will help
keep us focused on the types and the important fundamentals
of transformers. What we see here will be applicable to other
transformers as well, but types like Maybeand list introduce
other possibilities (failure cases, empty lists) that complicate
things a bit.
First, let’s compare the Identity type you’ve seen up to this
point and our new IdentityT datatype:
-- Plain old Identity. 'a' can be
-- something with more structure,
-- but it's not required and Identity
-- won't know anything about it.
newtype Identity a=
Identity { runIdentity ::a }
deriving (Eq,Show)

CHAPTER 25. E PLURIBUS MONAD 1524
-- The identity monad transformer, serving
-- only to to specify that additional
-- structure should exist.
newtype IdentityT f a=
IdentityT { runIdentityT ::f a }
deriving (Eq,Show)
What changed here is that we added an extra type argument.
Thenwewant Functor instancesforboth Identity andIdentityT :
instance Functor Identity where
fmap f ( Identity a)=Identity (f a)
instance (Functor m)
=>Functor (IdentityT m)where
fmap f ( IdentityT fa)=
IdentityT (fmap f fa)
TheIdentityT instancehereshouldlooksimilartothe Functor
instance for the Onedatatype above — the 𝑓𝑎argument is the
value inside the IdentityT with the (untouchable) structure
wrapped around it. All we know about that additional layer of
structure wrapped around the 𝑎value is that it is a Functor .
We also want Applicative instances for each:

CHAPTER 25. E PLURIBUS MONAD 1525
instance Applicative Identity where
pure=Identity
(Identity f)<*>(Identity a)=
Identity (f a)
instance (Applicative m)
=>Applicative (IdentityT m)where
pure x=IdentityT (pure x)
(IdentityT fab)<*>(IdentityT fa)=
IdentityT (fab<*>fa)
TheIdentity instance should be familiar. In the IdentityT
instance, the 𝑓𝑎𝑏variable represents the f (a -> b) that is the
first argument of (<*>). Since this can rely on the Applicative
instance for 𝑚to handle that bit, this instance defines how
to applicatively apply in the presence of that outer IdentityT
layer.
Finally, we want some Monadinstances:

CHAPTER 25. E PLURIBUS MONAD 1526
instance MonadIdentity where
return=pure
(Identity a)>>=f=f a
instance (Monadm)
=>Monad(IdentityT m)where
return=pure
(IdentityT ma)>>=f=
IdentityT $ma>>=runIdentityT .f
TheMonadinstance is tricky, so we’re going to do a few things
to break it down. Keep in mind that Monadis where we have to
really use concrete type information from IdentityT in order
to make the types fit.
The bind breakdown
We’ll start with a closer look at the instance as written above:

CHAPTER 25. E PLURIBUS MONAD 1527
instance (Monadm)
=>Monad(IdentityT m)where
return=pure
(IdentityT ma)>>=f=
-- [ 1 ] [2] [3]
IdentityT $ma
-- [8] [4]
>>=runIdentityT .f
-- [5] [7] [6]
1.First we pattern match or unpack the m avalue of IdentityT
m avia the data constructor. Doing this has the type
IdentityT m a -> m a and the type of maism a. This nomen-
clature doesn’t mean anything beyond mnemonic signal-
ing, but it is intended to be helpful.
2.The type of the bind we are implementing is the follow-
ing:
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
This is the instance we are defining.
3.The function we’re binding over is IdentityT m a . It has
the following type:

CHAPTER 25. E PLURIBUS MONAD 1528
(a->IdentityT m b)
4.Heremais the same one we unpacked out of the IdentityT
data constructor and has the type m a. Removed from its
IdentityT context, this is now the m athat this bind takes
as its first argument.
5.This is a diﬀerent bind! The first bind is the bind we’re
trying to implement; this bind is its definition or imple-
mentation. We’re now using the Monadwe asked for in the
instance declaration with the constraint Monad m => . This
will have the type:
(>>=)::m a->(a->m b)->m b
This is with respect to the 𝑚in the type IdentityT m a , not
the class of Monadinstances in general. In other words,
since we have already unpacked the IdentityT bit and, in
a sense, gotten it out of the way, this bind will be the bind
for the type 𝑚in the type IdentityT m . We don’t know
whatMonadthat is yet, and we don’t need to; since it has
theMonadtypeclass constraint on that variable, we know it
already has a Monadinstance defined for it, and this second
bind will be the bind defined for that type. All we’re doing
here is defining how to use that bind in the presence of
the additional IdentityT structure.

CHAPTER 25. E PLURIBUS MONAD 1529
6.This is the same 𝑓which was an argument to the Monad
instance we are defining, of type:
(a->IdentityT m b)
7.We need runIdentityT because 𝑓returns IdentityT m b , but
the>>=for the Monad m => has the type m a -> (a -> m b)
-> m b. It’ll end up trying to join m (IdentityT m b) , which
won’t work because mandIdentityT m are not the same
type. We use runIdentityT to unpack the value. Doing
this has the type IdentityT m b -> m b and the composition
runIdentityT . f in this context has the type a -> m b . You
can use undefined in GHCi to demonstrate this for yourself:
Prelude> :{
*Main| let f :: (a -> IdentityT m b)
*Main| f = undefined
*Main| :}
Prelude> :t f
f :: a -> IdentityT m b
Prelude> :t runIdentityT
runIdentityT :: IdentityT f a -> f a
Prelude> :t (runIdentityT . f)
(runIdentityT . f) :: a1 -> f a

CHAPTER 25. E PLURIBUS MONAD 1530
OK, the type variables don’t have the same name, but you
can see how a1 -> f a anda -> m b are the same type.
8.To satisfy the type of the outer bind we are implementing
for the MonadofIdentityT m , which expects a final result
of the type IdentityT m b , we must take the m bwhich the
expression ma >>= runIdentityT . f returns and repack it
inIdentityT . Note:
Prelude> :t IdentityT
IdentityT :: f a -> IdentityT f a
Prelude> :t runIdentityT
runIdentityT :: IdentityT f a -> f a
Now we have a bind we can use with IdentityT and some
otherMonad— in this example, a list:
Prelude> let sumR = return . (+1)
Prelude> IdentityT [1, 2, 3] >>= sumR
IdentityT {runIdentityT = [2,3,4]}
Implementing the bind, step by step
Now we’re going to backtrack and go through implementing
that bind step by step. The goal here is to demystify what
we’ve done and enable you to write your own instances for
whatever monad transformer you might need to implement

CHAPTER 25. E PLURIBUS MONAD 1531
yourself. We’ll go ahead and start back at the beginning, but
withInstanceSigs turned on so we can see the type:
{-# LANGUAGE InstanceSigs #-}
instance (Monadm)
=>Monad(IdentityT m)where
return=pure
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
undefined
Let’s leave the undefined as our final return expression, then
useletbindings and contradiction to see the types of our
attempts at making a Monadinstance. We’re going to use the
bottom value ( undefined ) to defer the parts of the proof we’re
obligated to produce until we’re ready. First, let’s get a let
binding in place and see it load, even if the code doesn’t work:

CHAPTER 25. E PLURIBUS MONAD 1532
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
letaimb=ma>>=f
inundefined
We’re using 𝑎𝑖𝑚𝑏as a mnemonic for the parts of the whole
thing that we’re trying to implement.
Here we get an error:
Couldn't match type ‘m’ with ‘IdentityT m’
That type error isn’t the most helpful thing in the world.
It’s hard to know what’s wrong from that. So, we’ll poke at this
a bit in order to get a more helpful type error.
First, we’ll do something we know should work. We’ll use
fmapinstead. Because that will typecheck (but not give us the
same result as (>>=)), we need to do something to give the
compiler a chance to contradict us and tell us the real type.
We force that type error by asserting a fully polymorphic type
for𝑎𝑖𝑚𝑏:

CHAPTER 25. E PLURIBUS MONAD 1533
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
letaimb::a
aimb=fmap f ma
inundefined
The type we asserted for 𝑎𝑖𝑚𝑏is impossible; we’ve said it
could be every type, and it can’t. The only thing that can have
that type is bottom, as bottom inhabits all types.
Conveniently, GHC will let us know what 𝑎𝑖𝑚𝑏is:
Couldn't match expected type ‘a1’
with actual type ‘m (IdentityT m b)’
Withthecurrentimplementation, 𝑎𝑖𝑚𝑏hasthetype m (IdentityT
m b). Now we can see the real problem: there is an IdentityT
layer in between the two bits of 𝑚that we need to join in order
to have a monad.
Here’s a breakdown:
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
The pattern match on IdentityT comes from having lifted
over it:

CHAPTER 25. E PLURIBUS MONAD 1534
(a->IdentityT m b)
The problem is, we used >>=over
ma
-- and got
m(IdentityT m b)
It doesn’t typecheck because (>>=)merges structure of the
same type after lifting (remember: it’s fmapcomposed with
joinunder the hood). Had our type been m (m b) after binding
fovermait would’ve worked fine. As it is, we need to find a
way to get the two bits of 𝑚together without an intervening
IdentityT layer.
We’re going to continue with having separate fmapandjoin
instead of using (>>=)because it makes the step-wise manip-
ulation of structure easier to see. How do we get rid of the
IdentityT in the middle of the two 𝑚structures? Well, we know
𝑚is aMonad, which means it’s also a Functor . So, we can use
runIdentityT to get rid of the IdentityT structure in the middle
of the stack of types:

CHAPTER 25. E PLURIBUS MONAD 1535
-- Change m (IdentityT m b)
-- into m (m b)
-- Note:
runIdentityT ::IdentityT f a->f a
fmaprunIdentityT ::Functor f
=>f (IdentityT f1 a)->f (f1 a)
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
letaimb::a
aimb=fmap runIdentityT (fmap f ma)
inundefined
And when we load this code, we get an encouraging type
error:
Couldn't match expected type ‘a1’
with actual type ‘m (m b)’
It’s telling us we have achieved the type m (m b) , so now we
know how to get where we want. The 𝑎1here is the 𝑎we had
assigned to 𝑎𝑖𝑚𝑏, but it’s telling us that our actual type is not
what we asserted but this other type. Thus we have discovered

CHAPTER 25. E PLURIBUS MONAD 1536
what our actual type is, which gives us a clue about how to fix
it.
We’ll use joinfromControl.Monad to merge the nested 𝑚
structure:
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
letaimb::a
aimb=
join (fmap runIdentityT (fmap f ma))
inundefined
And when we load it, the compiler tells us we finally have
anm bthat we can return:
Couldn't match expected type ‘a1’
with actual type ‘m b’
In fact, before we begin cleaning up our code, we can verify
this is the case real quick:

CHAPTER 25. E PLURIBUS MONAD 1537
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
letaimb=
join (fmap runIdentityT (fmap f ma))
inaimb
We removed the type declaration for aimband also changed
thein undefined . But we know that 𝑎𝑖𝑚𝑏has the actual type m
b, so this won’t work. Why? If we take a look at the type error:
Couldn't match type ‘m’ with ‘IdentityT m’
The(>>=)we are implementing has a final result of type
IdentityT m b , so the type of 𝑎𝑖𝑚𝑏doesn’t match it yet. We need
to wrap m binIdentityT to make it typecheck:

CHAPTER 25. E PLURIBUS MONAD 1538
-- Remember:
IdentityT ::f a->IdentityT f a
instance (Monadm)
=>Monad(IdentityT m)where
return=pure
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
letaimb=
join (fmap runIdentityT
(fmap f ma))
inIdentityT aimb
This should compile. We rewrap m bback in the IdentityT
type and we should be good to go.
Refactoring
Now that we have something that works, let’s refactor. We’d
like to improve our implementation of (>>=). Taking things
one step at a time is usually more successful than trying to
rewrite all at once, especially once you have a baseline version
that you know should work. How should we improve this line?

CHAPTER 25. E PLURIBUS MONAD 1539
IdentityT $
join(fmap runIdentityT (fmap f ma))
Well, oneofthe Functor lawstellsussomethingabout fmapping
twice:
-- Functor law:
fmap(f.g)==fmap f.fmap g
Indeed! So we can change that line to the following and it
should be identical:
IdentityT $
join(fmap (runIdentityT .f) ma)
Now it seems suspicious that we’re fmapping and also us-
ingjoinon the result of having fmapped the two functions we
composed. Isn’t joincomposed with fmapjust(>>=)?
x>>=f=join (fmap f x)
Accordingly, we can change our Monadinstance to the fol-
lowing:

CHAPTER 25. E PLURIBUS MONAD 1540
instance (Monadm)
=>Monad(IdentityT m)where
return=pure
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
IdentityT $ma>>=runIdentityT .f
And that should work still! We have a type constructor now
(IdentityT ) that takes a monad as an argument and returns a
monad as a result.
This implementation can be written other ways. In the
transformers library, for example, it’s written like this:
m>>=k=
IdentityT $runIdentityT .k
=<<runIdentityT m
Take a moment and work out for yourself how that is func-
tionally equivalent to our implementation.
The essential extra of monad transformers
It may not seem like it, but the IdentityT monad transformer
captures the essence of transformers generally. We only em-

CHAPTER 25. E PLURIBUS MONAD 1541
barked on this quest because we couldn’t be guaranteed a Monad
instance given the composition of two types. Given that, we
know having Functor ,Applicative , andMonadat our disposal isn’t
enough to make that new Monadinstance. So what was novel in
the following code?
(>>=)::IdentityT m a
->(a->IdentityT m b)
->IdentityT m b
(IdentityT ma)>>=f=
IdentityT $ma>>=runIdentityT .f
It wasn’t the pattern match on IdentityT ; we get that from
theFunctor anyway:
-- Not this
(IdentityT ma)...
It wasn’t the ability to (>>=)functions over the mavalue of
type𝑚𝑎, we get that from the Monadconstraint on 𝑚anyway.
-- Not this
...ma>>= ...
We needed to know one of the types concretely so that
we could use runIdentityT (essentially fmapping a fold of the
IdentityT structure) and then repack the value in IdentityT :

CHAPTER 25. E PLURIBUS MONAD 1542
-- We need to know IdentityT
-- concretely to do this
IdentityT ..runIdentityT ...
As you’ll recall, until we used runIdentityT we couldn’t get
the types to fit because IdentityT was wedged in the middle of
two bits of 𝑚. It turns out to be impossible to fix that using
onlyFunctor ,Applicative , andMonad. This is an example of why
we can’t make a Monadinstance for the Compose type, but we
can make a transformer type like IdentityT where we leverage
information specific to the type and combine it with any other
type that has a Monadinstance. In general, in order to make the
types fit, we’ll need some way to fold and reconstruct the type
we have concrete information for.
25.9 Finding a pattern
Transformers are bearers of single-type concrete information
that let you create ever-bigger monads in a sense. Nesting
such as
(Monadm)=>m (m a)
is addressed by joinalready. We use transformers when
we want a >>=operation over 𝑓and𝑔of diﬀerent types (but
both have Monadinstances). You have to create new types called

CHAPTER 25. E PLURIBUS MONAD 1543
monad transformers and write Monadinstances for those types
to have a way of dealing with the extra structure generated.
The general pattern is this: You want to compose two poly-
morphic types, 𝑓and𝑔, that each have a Monadinstance. But
you’ll end up with this pattern:
f(g (f b))
Monad’s bind can’t join those types, not with that intervening
𝑔. So you need to get to this:
f(f b)
You won’t be able to unless you have some way of folding
the𝑔in the middle. You can’t do that with Monad. The essence
ofMonadisjoin, but here you have only one bit of 𝑔structure,
notg (g ...) , so that’s not enough. The straightforward thing
to do is to make 𝑔concrete. With concrete type information
for the inner bit of structure, we can fold out the 𝑔and get on
with it. The good news is that transformers don’t require 𝑓be
concrete; 𝑓can remain polymorphic so long as it has a Monad
instance, so we only write a transformer once for each type.
We can see this pattern with IdentityT as well. You may
recall this step in our process of writing IdentityT ’sMonad:
(IdentityT ma)>>=f=
letaimb::m (IdentityT m b)
aimb=fmap f ma

CHAPTER 25. E PLURIBUS MONAD 1544
We have something that’ll typecheck, but it’s not quite in
the shape we would like. Of course, the underlying type once
we throw away the IdentityT data constructor is m (m b) which’ll
suit us just fine, but we have to fold out the IdentityT before we
can use the joinfromMonad m => m . That leads us to the next
step:
letaimb::m (m b)
aimb=fmap runIdentityT (fmap f ma)
Now we finally have something we can join because we lifted
the record accessor for IdentityT over the 𝑚! Since IdentityT
is so simple, the record accessor is sufficient to fold up the
structure. From there the following transitions become easy:
m(m b)->m b->IdentityT m b
The final type is what our definition of (>>=)forIdentityT
must result in.
The basic pattern that many monad transformers are en-
abling us to cope with is the following type transitions, where
𝑚is the polymorphic, outer structure and 𝑇is some concrete
type the transformer is for. For example, in the above, 𝑇would
beIdentityT .

CHAPTER 25. E PLURIBUS MONAD 1545
m (Tm b)
->m (m b)
->m b
->Tm b
Don’t consider this a hard and fast rule for what types you’ll
encounter in implementing transformers, but rather some
intuition for why transformers are necessary to begin with.

Chapter 26
Monad transformers
I do not say such things
except insofar as I
consider this to permit
some transformation of
things. Everything I do, I
do in order that it may be
of use.
Michel Foucault
1546

CHAPTER 26. STACK ‘EM UP 1547
26.1 Monad transformers
The last chapter demonstrated why we need monad transform-
ers and the basic type manipulation that’s going on to make
that bit of magick happen. Monad transformers are important
in a lot of everyday Haskell code, though, so we want to dive
deeper and make sure we have a good understanding of how
to use them in practice. Even after you know how to write all
the transformer instances, managing stacks of transformers
in an application can be tricky. The goal of this chapter is to
get comfortable with it.
In this chapter, we will
•work through more monad transformer types and in-
stances;
•look at the ordering and wrapping of monad transformer
stacks;
•lift, lift, lift, and lift some more.
26.2 MaybeT
In the last chapter, we worked through an extended break-
down of the IdentityT transformer. IdentityT is, as you might
imagine, not the most useful of the monad transformers, al-
though it is not without practical applications (more on this

CHAPTER 26. STACK ‘EM UP 1548
later). As we’ve seen, though, the Maybe Monad can be useful, and
so it is that the tranformer variant, MaybeT, finds its way into
the pantheon of important transformers.
TheMaybeT transformer is a bit more complex than IdentityT .
If you worked through all the exercises of the previous chapter,
then this section will not be too surprising, because this will
rely on things you’ve seen with IdentityT and the Compose type
already. However, to ensure that transformers are thoroughly
demystified for you, it’s worth working through them carefully.
We begin with the newtype for our transformer:
newtype MaybeTm a=
MaybeT{ runMaybeT ::m (Maybea) }
The structure of our MaybeT type and the Compose type are
similar so we can reuse the basic patterns of the Compose type
for theFunctor andApplicative instances:

CHAPTER 26. STACK ‘EM UP 1549
-- Remember the Functor for Compose?
instance (Functor f,Functor g)
=>Functor (Compose f g)where
fmap f ( Compose fga)=
Compose $(fmap.fmap) f fga
-- compare to the instance for MaybeT
instance (Functor m)
=>Functor (MaybeTm)where
fmap f ( MaybeTma)=
MaybeT$(fmap.fmap) f ma
We don’t need to do anything diﬀerent for the Functor in-
stance, because transformers are needed for the Monad, not the
Functor .
Spoiler alert!
If you haven’t yet written the Applicative instance for Compose
from the previous chapter, you may want to stop right here.
We’ll start with what might seem like an obvious way to
write the MaybeT Applicative and find out why it doesn’t work.
This does not compile:

CHAPTER 26. STACK ‘EM UP 1550
instance (Applicative m)
=>Applicative (MaybeTm)where
pure x=MaybeT(pure (pure x))
(MaybeTfab)<*>(MaybeTmma)=
MaybeT$fab<*>mma
The𝑓𝑎𝑏represents the function m (Maybe (a -> b)) and the
𝑚𝑚𝑎represents the m (Maybe a) .
You’ll get this error if you try it:
Couldn't match type ‘Maybe (a -> b)’
with ‘Maybe a -> Maybe b’
Here is the Applicative instance for Compose as a comparison
with the MaybeT instance we’re trying to write:
instance (Applicative f,Applicative g)
=>Applicative (Compose f g)where
pure x=Compose (pure (pure x))
Compose f<*>Compose x=
Compose ((<*>)<$>f<*>x)
Let’s break this down a bit in case you felt confused when
you wrote this for the last chapter’s exercise. Because you did
that exercise…right?

CHAPTER 26. STACK ‘EM UP 1551
The idea here is that we have to lift an Applicative apply
over the outer structure 𝑓to get the g (a -> b) intog a -> g b
so that the Applicative instance for 𝑓can be leveraged. We can
stretch this idea a bit and use concrete types:
innerMost
::[Maybe(Identity (a->b))]
->[Maybe(Identity a->Identity b)]
innerMost =(fmap.fmap) (<*>)
second'
::[Maybe(Identity a->Identity b)]
->[Maybe(Identity a)
->Maybe(Identity b) ]
second' =fmap (<*>)
final'
::[Maybe(Identity a)
->Maybe(Identity b) ]
->[Maybe(Identity a)]
->[Maybe(Identity b)]
final'=(<*>)
The function that could be the Applicative instance for such
a hypothetical type would look like:

CHAPTER 26. STACK ‘EM UP 1552
lmiApply ::[Maybe(Identity (a->b))]
->[Maybe(Identity a)]
->[Maybe(Identity b)]
lmiApply f x=
final' (second' (innerMost f)) x
TheApplicative instance for our MaybeT type will employ this
same idea, because applicatives are closed under composition,
as we noted in the last chapter. We only need to do something
diﬀerent from the Compose instances once we get to Monad.
So, we took the long way around to this:
instance (Applicative m)
=>Applicative (MaybeTm)where
pure x=MaybeT(pure (pure x))
(MaybeTfab)<*>(MaybeTmma)=
MaybeT$(<*>)<$>fab<*>mma
MaybeT Monad instance
At last, on to the Monadinstance! Note that we’ve given some of
the intermediate types:
instance (Monadm)
=>Monad(MaybeTm)where
return=pure

CHAPTER 26. STACK ‘EM UP 1553
(>>=)::MaybeTm a
->(a->MaybeTm b)
->MaybeTm b
(MaybeTma)>>=f=
-- [2] [3]
MaybeT$ do
-- [ 1 ]
-- ma :: m (Maybe a)
-- v :: Maybe a
v<-ma
-- [4]
casevof
-- [5]
Nothing ->returnNothing
-- [ 6 ]
Justy->runMaybeT (f y)
-- [7] [8]
-- y :: a
-- f :: a -> MaybeT m b
-- f y :: MaybeT m b
-- runMaybeT (f y) :: m (Maybe b)
Explaining it step by step:

CHAPTER 26. STACK ‘EM UP 1554
1.We have to return a MaybeT value at the end, so the doblock
has the MaybeT data constructor in front of it. This means
the final value of our doblock expression must be of type
m (Maybe b) in order to typecheck because our goal is to
go from MaybeT m a toMaybeT m b .
2.The first argument to bind here is MaybeT m a . We unbun-
dled that from MaybeT by pattern matching on the MaybeT
newtype data constructor.
3.The second argument to bind is (a -> MaybeT m b) .
4.In the definition of MaybeT , notice something:
newtype MaybeTm a=
MaybeT{ runMaybeT ::m (Maybea) }
-- ^---------^
It’s aMaybevalue wrapped in some other type for which
all we know is that it has a Monadinstance. Accordingly, we
begin in our doblock by using the left arrow bind syntax.
This gives us a reference to the hypothetical Maybevalue
out of the 𝑚structure which is unknown.
5.Since using <-to bind Maybe a out ofm (Maybe a) left us
with aMaybevalue, we use a case expression on the Maybe
value.

CHAPTER 26. STACK ‘EM UP 1555
6.If we get Nothing , we kick Nothing back out, but we have to
embed it in the 𝑚structure. We don’t know what 𝑚is, but
being a Monad(and thus also an Applicative ) means we can
usereturn (pure) to perform that embedding.
7.If we get Just, we now have a value of type 𝑎that we can
pass to our function fof type a -> MaybeT m b .
8.We have to fold the m (Maybe b) value out of the MaybeT
since the MaybeT constructor is already wrapped around
the whole doblock, then we’re done.
Don’t be afraid to get a pen and paper and work all that out
until you understand how things are happening before you
move on.
26.3 EitherT
Just asMaybehas a transformer variant in the form of MaybeT , we
can make a transformer variant of Either . We’ll call it EitherT .
Your task is to implement the instances for the transformer
variant:
newtype EitherT e m a=
EitherT { runEitherT ::m (Eithere a) }

CHAPTER 26. STACK ‘EM UP 1556
Exercises: EitherT
1.Write the Functor instance for EitherT :
instance Functor m
=>Functor (EitherT e m)where
fmap=undefined
2.Write the Applicative instance for EitherT :
instance Applicative m
=>Applicative (EitherT e m)where
pure=undefined
f<*>a=undefined
3.Write the Monadinstance for EitherT :
instance Monadm
=>Monad(EitherT e m)where
return=pure
v>>=f=undefined
4.Write the swapEitherT helper function for EitherT .

CHAPTER 26. STACK ‘EM UP 1557
-- transformer version of swapEither.
swapEitherT ::(Functor m)
=>EitherT e m a
->EitherT a m e
swapEitherT =undefined
Hint: write swapEither first, then swapEitherT in terms of
the former.
5.Write the transformer variant of the either catamorphism.
eitherT ::Monadm=>
(a->m c)
->(b->m c)
->EitherT a m b
->m c
eitherT =undefined
26.4 ReaderT
ReaderT is one of the most commonly used transformers in
conventional Haskell applications. It is like Reader, except in
the transformer variant we’re generating additional structure
in the return type of the function:
newtype ReaderT r m a=
ReaderT { runReaderT ::r->m a }

CHAPTER 26. STACK ‘EM UP 1558
The value inside the ReaderT is a function. Type constructors
such as Maybeare also functions in some senses, but we have
to handle this case a bit diﬀerently. The first argument to the
function inside ReaderT is part of the structure we’ll have to
bind over.
This time we’re going to give you the instances. If you want
to try writing them yourself, do not read on!

CHAPTER 26. STACK ‘EM UP 1559
instance (Functor m)
=>Functor (ReaderT r m)where
fmap f ( ReaderT rma)=
ReaderT $(fmap.fmap) f rma
instance (Applicative m)
=>Applicative (ReaderT r m)where
pure a=ReaderT (pure (pure a))
(ReaderT fmab)<*>(ReaderT rma)=
ReaderT $(<*>)<$>fmab<*>rma
instance (Monadm)
=>Monad(ReaderT r m)where
return=pure
(>>=)::ReaderT r m a
->(a->ReaderT r m b)
->ReaderT r m b
(ReaderT rma)>>=f=
ReaderT $\r-> do
-- [1]
a<-rma r
-- [3] [ 2 ]
runReaderT (f a) r
-- [5] [ 4 ] [6]

CHAPTER 26. STACK ‘EM UP 1560
1.Again, the type of the value in a ReaderT must be a function,
so the act of binding a function over a ReaderT must itself
be a function awaiting the argument of type 𝑟, which we’ve
chosen to name 𝑟as a convenience in our terms. Also note
that we’re repacking our lambda inside the ReaderT data
constructor.
2.Wepattern-matchedthe r -> m a (representedinourterms
by𝑟𝑚𝑎) out of the ReaderT data constructor. Now we’re ap-
plying it to the 𝑟that we’re expecting in the body of the
anonymous lambda.
3.The result of applying r -> m a to a value of type 𝑟ism
a. We need a value of type 𝑎in order to apply our a ->
ReaderT r m b function. To be able to write code in terms
of that hypothetical 𝑎, we bind ( <-) the𝑎out of the 𝑚
structure. We’ve bound that value to the name 𝑎as a
mnemonic to remember the type.
4.Applying 𝑓, which has type a -> ReaderT r m b , to the value
𝑎results in a value of type ReaderT r m b .
5.We unpack the r -> m b out of the ReaderT structure.
6.Finally, we apply the resulting r -> m b to the𝑟we had at
the beginning of our lambda, that eventual argument that
Reader abstracts for us. We have to return m bas the final
expression in this anonymous lambda or the function is

CHAPTER 26. STACK ‘EM UP 1561
not valid. To be valid, it must be of type r -> m b which
expresses the constraint that if it is applied to an argument
of type 𝑟, it must produce a value of type m b.
No exercises this time. You deserve a break.
26.5 StateT
Similar to Reader andReaderT ,StateT isStatebut with additional
monadic structure wrapped around the result. StateT is some-
what more useful and common than the State Monad you saw
earlier. Like ReaderT , its value is a function:
newtype StateTs m a=
StateT{ runStateT ::s->m (a,s) }
Exercises: StateT
If you’re familiar with the distinction, you’ll be implementing
the strict variant of StateT here. To make the strict variant,
you don’t have to do anything special. Write the most obvious
thing that could work. The lazy (lazier, anyway) variant is
the one that involves adding a bit extra. We’ll explain the
diﬀerence in the chapter on nonstrictness.
1.You’ll have to do the Functor andApplicative instances
first, because there aren’t Functor andApplicative instances
ready to go for the type Monad m => s -> m (a, s) .

CHAPTER 26. STACK ‘EM UP 1562
instance (Functor m)
=>Functor (StateTs m)where
fmap f m =undefined
2.As with Functor , you can’t cheat and reuse an underlying
Applicative instance, so you’ll have to do the work with
thes -> m (a, s) type yourself.
instance (Monadm)
=>Applicative (StateTs m)where
pure=undefined
(<*>)=undefined
Also note that the constraint on 𝑚is notApplicative as you
expect, but rather Monad. This is because you can’t express
the order-dependent computation you’d expect the StateT
Applicative to have without having a Monad for 𝑚. To
learn more, see this Stack Overflow question1about this
issue. Also see this Github issue2on the NICTA Course
Github repository. Beware ! The NICTA course issue
gives away the answer. In essence, the issue is that without
Monad, you’re feeding the initial state to each computation
inStateT rather than threading it through as you go. This
1Is it possible to implement ‘(Applicative m) => Applica-
tive (StateT s m)‘? http://stackoverflow.com/questions/18673525/
is-it-possible-to-implement-applicative-m-applicative-statet-s-m
2https://github.com/NICTA/course/issues/134

CHAPTER 26. STACK ‘EM UP 1563
is a general pattern contrasting Applicative andMonadand
is worth contemplating.
3.TheMonadinstance should look fairly similar to the Monad
instance you wrote for ReaderT .
instance (Monadm)
=>Monad(StateTs m)where
return=pure
sma>>=f=undefined
ReaderT, WriterT, StateT
We’d like to point something out about these three types:
newtype Readerr a=
Reader{ runReader ::r->a }
newtype Writerw a=
Writer{ runWriter ::(a, w) }
newtype States a=
State{ runState ::s->(a, s) }
and their transformer variants:

CHAPTER 26. STACK ‘EM UP 1564
newtype ReaderT r m a=
ReaderT { runReaderT ::r->m a }
newtype WriterT w m a=
WriterT { runWriterT ::m (a, w) }
newtype StateTs m a=
StateT{ runStateT ::s->m (a, s) }
You’re already familiar with Reader andState. We haven’t
shown you Writer orWriterT up to this point because, quite
frankly, you shouldn’t use it. We’ll explain why not in a section
later in this chapter.
For the purposes of the progression we’re trying to demon-
strate here, it suffices to know that the Writer Applicative and
Monadwork by combining the 𝑤values monoidally. With that
in mind, what we can see is that Reader lets us talk about val-
ues we need, Writer lets us deal with values we can emit and
combine (but not read), and Statelets us both read and write
values in any manner we desire — including monoidally, like
Writer . This is one reason you needn’t bother with Writer since
Statecan replace it anyway. That’s why you don’t need Writer ;
we’ll talk more about why you don’t want Writer later.
In fact, there is a type in the transformers library that com-
binesReader ,Writer , andStateinto one big type:

CHAPTER 26. STACK ‘EM UP 1565
newtype RWSTr w s m a =
RWST{ runRWST ::r->s->m (a, s, w) }
Because of the Writer component, you probably wouldn’t
want to use that in most applications either, but it’s good to
know it exists.
Correspondence between StateT and Parser
You may recall what a simple parser type looks like:
typeParsera=String->Maybe(a,String)
You may remember our discussion about the similarities
between parsers and Statein the Parsers chapter. Now, we
could choose to define a Parser type in the following manner:
newtype StateTs m a=
StateT{ runStateT ::s->m (a,s) }
typeParser=StateTStringMaybe
Nobody does this in practice, but it’s useful to consider the
similarity to get a feel for what StateT is all about.

CHAPTER 26. STACK ‘EM UP 1566
26.6 Types you probably don’t want to
use
Not every type will necessarily be performant or make sense.
ListTandWriter /WriterT are examples of this.
Why not use Writer or WriterT?
It’s a bit too easy to get into a situation where Writer is either
too lazy or too strict for the problem you’re solving, and then
it’ll use more memory than you’d like. Writer can accumulate
unevaluated thunks, causing memory leaks. It’s also inappro-
priate for logging long-running or ongoing programs due to
the fact that you can’t retrieve any of the logged values until
the computation is complete.3
Usually when Writer is used in an application, it’s not called
Writer . Instead a one-oﬀ is created for a specific type 𝑤. Given
that, it’s still useful to know when you’re looking at something
that’s a Reader,Writer, orState, even if the author didn’t use
the types by those names from the transformers library. Some-
times this is because they wanted a stricter Writer than the one
already available.
Determining and measuring when more strictness (more
eagerly evaluating your thunks) is needed in your programs is
3If you’d like to understand this better, Gabriel Gonzalez has a helpful blog post on
the subject. http://www.haskellforall.com/2014/02/streaming-logging.html

CHAPTER 26. STACK ‘EM UP 1567
the topic of the upcoming chapter on nonstrictness.
The ListT you want isn’t made from the List type
The most obvious way to implement ListTis generally not
recommended for a variety of reasons, including:
1.Most people’s first attempt won’t pass the associativity law.
We’re not going to show you a way to write it that does
pass that law because it’s not worth it for the reasons listed
below.
2.It’s not very fast.
3.Streaming libraries like pipes4andconduit5do it better for
most use cases.
Prior art for “ ListTdone right” also includes AmbT6by Conal
