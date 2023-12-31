Elliott, although you may find it challenging to understand if
you aren’t familiar with ContTand the motivation behind Amb.
Lists in Haskell are as much a control structure as a data
structure, so streaming libraries such as pipesgenerally suffice
if you need a transformer. This is less of a sticking point in
writing applications than you’d think.
4http://hackage.haskell.org/package/pipes
5http://hackage.haskell.org/package/conduit
6https://wiki.haskell.org/Amb

CHAPTER 26. STACK ‘EM UP 1568
26.7 Recovering an ordinary type from
a transformer
If you have a transformer variant of a type and want to use
it as if it was the non-transformer version, you need some 𝑚
structure that doesn’t do anything. Have we seen anything like
that? What about Identity ?
Prelude> runMaybeT $ (+1) <$> MaybeT (Identity (Just 1))
Identity {runIdentity = Just 2}
Prelude> runMaybeT $ (+1) <$> MaybeT (Identity Nothing)
Identity {runIdentity = Nothing}
Given that, we can get Identity fromIdentityT and so on in
the following manner:
typeMyIdentity a=IdentityT Identity a
typeMaybe a=MaybeTIdentity a
typeEithere a=EitherT eIdentity a
typeReaderr a=ReaderT eIdentity a
typeStates a=StateTsIdentity a
This works fine for recovering the non-transformer variant
of each type as the Identity type is acting as a bit of do-nothing
structural paste for filling in the gap.

CHAPTER 26. STACK ‘EM UP 1569
Yeah, but why? You don’t ordinarily need to do this if you’re
working with a transformer that has a corresponding non-
transformer type you can use. For example, it’s less common
to need ( ExceptT Identity ) because the Either type is already
there, so you don’t need to retrieve that type from the trans-
former. However, if you’re writing something with, say, scotty ,
where a ReaderT is part of the environment, you can’t easily
retrieve the Reader type out of that because Reader is not a type
that exists on its own and you can’t modify that ReaderT with-
out essentially rewriting all of scotty , and, wow, nobody wants
that for you. You might then have a situation where what
you’re doing only needs a Reader, not aReaderT , so you could
use (ReaderTIdentity ) to be compatible with scotty without hav-
ing to rewrite everything but still being able to keep your own
code a bit tighter and simpler.
Thetransformers library In general, don’t use hand-rolled
versions of these transformer types without good reason. You
can find many of them in base or the transformers library, and
that library should have come with your GHC installation.
A note on ExceptT Although a library called either exists on
Hackage and provides the EitherT type, most Haskellers are
moving to the identical ExceptT type in the transformers library.
Again, this has mostly to do with the fact that transformers

CHAPTER 26. STACK ‘EM UP 1570
comes packaged with GHC already, so ExceptT is ready-to-
hand; the underlying type is the same.
26.8 Lexically inner is structurally
outer
One of the trickier parts of monad transformers is that the
lexical representation of the types will violate your intuitions
with respect to the relationship it has with the structure of
your values. Let us note something in the definitions of the
following types:
-- definition in transformers may look
-- slightly different. It's not important.
newtype ExceptT e m a=
ExceptT { runExceptT ::m (Eithere a) }
newtype MaybeTm a=
MaybeT{ runMaybeT ::m (Maybea) }
newtype ReaderT r m a=
ReaderT { runReaderT ::r->m a }
A necessary byproduct of how transformers work is that
the additional structure 𝑚is always wrapped around our value.
One thing to note is that it’s only wrapped around things

CHAPTER 26. STACK ‘EM UP 1571
we can have, not things we need, such as with ReaderT . The
consequence of this is that a series of monad transformers in a
type will begin with the innermost type structurally speaking.
Consider the following:
moduleOuterInner where
importControl.Monad.Trans.Except
importControl.Monad.Trans.Maybe
importControl.Monad.Trans.Reader
-- We only need to use return once
-- because it's one big Monad
embedded ::MaybeT
(ExceptT String
(ReaderT ()IO))
Int
embedded =return1
We can sort of peel away the layers one by one:

CHAPTER 26. STACK ‘EM UP 1572
maybeUnwrap ::ExceptT String
(ReaderT ()IO) (MaybeInt)
maybeUnwrap =runMaybeT embedded
-- Next
eitherUnwrap ::ReaderT ()IO
(EitherString(MaybeInt))
eitherUnwrap =runExceptT maybeUnwrap
-- Lastly
readerUnwrap ::()
->IO(EitherString
(MaybeInt))
readerUnwrap =runReaderT eitherUnwrap
Then if we’d like to evaluate this code, we feed the unit
value to the function:
Prelude> readerUnwrap ()
Right (Just 1)
Why is this the result? Consider that we used return for a
Monadcomprising Reader ,Either , andMaybe:

CHAPTER 26. STACK ‘EM UP 1573
instance Monad((->) r)where
return=const
instance Monad(Eithere)where
return=Right
instance MonadMaybewhere
return=Just
We can treat having used return for theReader /Either /Maybe
stack as composition, consider how we get the same result as
readerUnwrap () here:
Prelude> (const . Right . Just $ 1) ()
Right (Just 1)
A terminological point to keep in mind when reading about
monad transformers is that when Haskellers say base monad
they usually mean what is structurally outermost.
typeMyTypea=IO[Maybea]
InMyType , the base monad is IO.
Exercise: Wrap It Up
TurnreaderUnwrap fromthepreviousexamplebackinto embedded
through the use of the data constructors for each transformer.

CHAPTER 26. STACK ‘EM UP 1574
-- Modify it to make it work.
embedded ::MaybeT
(ExceptT String
(ReaderT ()IO))
Int
embedded = ???(const ( Right(Just1)))
26.9 MonadTrans
We often want to lift functions into a larger context. We’ve
been doing this for a while with Functor , which lifts a function
into a context and applies it to the value inside. The facility
to do this also undergirds Applicative ,Monad, andTraversable .
However, fmapisn’t always enough, so we have some functions
that are essentially fmapfor diﬀerent contexts:
fmap :: Functor f
=> (a -> b) -> f a -> f b
liftA :: Applicative f
=> (a -> b) -> f a -> f b
liftM :: Monad m
=> (a -> r) -> m a -> m r

CHAPTER 26. STACK ‘EM UP 1575
You might notice the latter two examples have liftin the
function name. While we’ve encouraged you not to get too
excited about the meaning of function names, in this case they
do give you a clue of what they’re doing. They are lifting,
just asfmapdoes, a function into some larger context. The
underlying structure of the bind function from Monadis also a
lifting function — fmapagain! — composed with the crucial
joinfunction.
In some cases, we want to talk about more or diﬀerent
structure than these types permit. In other cases, we want
something that does as much lifting as is necessary to reach
some (structurally) outermost position in a stack of monad
transformers. Monad transformers can be nested in order
to compose various eﬀects into one monster function, but to
manage those stacks, we need to lift more.
The typeclass that lifts
MonadTrans is a typeclass with one core method: lift. Speaking
generally, it is about lifting actions in some Monadover a trans-
former type which wraps itself in the original Monad. Fancy!

CHAPTER 26. STACK ‘EM UP 1576
classMonadTrans twhere
-- | Lift a computation from
-- the argument monad to
-- the constructed monad.
lift::(Monadm)=>m a->t m a
Here the 𝑡is a (constructed) monad transformer type that
has an instance of MonadTrans defined.
We’re going to work through a relatively uncomplicated
example from scotty now.
Motivating MonadTrans
You may remember from previous chapters that scotty is a
web framework for Haskell. One thing to know about scotty,
without getting into all the gritty details of how it works, is that
the monad transformers the framework relies on are them-
selves newtypes for monad transformer stacks. Wait, what?
Well, look:

CHAPTER 26. STACK ‘EM UP 1577
newtype ScottyT e m a=
ScottyT
{ runS::State(ScottyState e m) a }
deriving (Functor,Applicative ,Monad)
newtype ActionT e m a=
ActionT
{ runAM
::ExceptT
(ActionError e)
(ReaderT ActionEnv
(StateTScottyResponse m))
a
}
deriving (Functor,Applicative )
typeScottyM =ScottyT TextIO
typeActionM =ActionT TextIO
We’ll use ActionM andActionT andScottyM andScottyT as if
they were the same thing, but the Mvariants are type synonyms
for the transformers with the inner types already set. This
roughly translates to the errors (the left side of the ExceptT ) in
ScottyM orActionM being returned as Text, while the right side
of theExceptT , whatever it does, is IO.ExceptT is the transformer

CHAPTER 26. STACK ‘EM UP 1578
version of Either, and a ReaderT and aStateT are stacked up
inside that as well. These internal mechanics don’t matter that
much to you, as a user of the scotty API, but it’s useful to see
how much is packed up in there.
Now, back to our example. This is the “hello, world” exam-
ple using scotty , but the following will cause a type error:
-- scotty.hs
{-# LANGUAGE OverloadedStrings #-}
moduleScottywhere
importWeb.Scotty
importData.Monoid (mconcat)
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
putStrLn "hello"
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]

CHAPTER 26. STACK ‘EM UP 1579
Reminder: in your terminal, you can follow along with this
like so:
$ stack build scotty
$ stack ghci
Prelude> :l scotty.hs
When you try to load it, you should get a type error:
Couldn't match expected type
‘Web.Scotty.Internal.Types.ActionT
Data.Text.Internal.Lazy.Text IO a0’
with actual type ‘IO ()’
In a stmt of a 'do' block: putStrLn "hello"
In the second argument of ‘($)’, namely
‘do { beam <- param "word";
putStrLn "hello";
html $ mconcat ["<h1>Scotty, ", beam, ....] }’
The reason for this type error is that putStrLn has the type
IO (), but it is inside a doblock inside our get, and the monad
that code is in is therefore ActionM /ActionT :
get::RoutePattern
->ActionM ()
->ScottyM ()

CHAPTER 26. STACK ‘EM UP 1580
OurActionT type eventually reaches IO, but there’s addi-
tional structure we need to lift over first. To fix this, we’ll start
by adding an import:
importControl.Monad.Trans.Class
And amend that line with putStrLn to the following:
lift(putStrLn "hello")
It should work.
You can assert a type for the liftembedded in the scotty
action:
lethello=putStrLn "hello"
(lift::IOa->ActionM a) hello
Let’s see what it does. Load the file again and call the main
function. You should see this message:
Setting phasers to stun... (port 3000) (ctrl-c to quit)
In the address bar of your web browser, type localhost:3000 .
You should notice two things: one is that there is nothing in
thebeamslot of the text that prints to your screen, and the other
is that it prints ‘hello’ to your terminal where the program is
running. Try adding a word to the end of the address:
localhost:3000/beam

CHAPTER 26. STACK ‘EM UP 1581
The text on your screen should change, and ‘hello’ should
print in your terminal again. That /:word parameter is what has
been bound via the variable beaminto that html line at the end
of thedoblock, while the ‘hello’ has been lifted over the ActionM
so that it can print in your terminal. It will print another ‘hello’
to your terminal every time something happens on the web
page.
We can concretize our use of liftin the following steps.
Please follow along by asserting the types for the application
ofliftin thescotty application above:
lift::(Monadm,MonadTrans t)
=>m a->t m a
lift::(MonadTrans t)
=>IOa->tIOa
lift::IOa->ActionM a
lift::IO()->ActionM ()
We go from (t IO a) to(ActionM a) because the IOis inside
theActionM .
Let’s examine ActionM more carefully:
Prelude> import Web.Scotty
Prelude> import Web.Scotty.Trans
Prelude> :info ActionM
type ActionM = ActionT Data.Text.Internal.Lazy.Text IO
-- Defined in ‘Web.Scotty’

CHAPTER 26. STACK ‘EM UP 1582
We can see for ourselves what this liftdid by looking at
theMonadTrans instance for ActionT , which is what ActionM is a
type alias of:
instance MonadTrans (ActionT e)where
lift=ActionT .lift.lift.lift
Part of the niceness here is that ActionT is itself defined in
terms of three more monad transformers. We can see this in
the definition of ActionT :
newtype ActionT e m a=
ActionT {
runAM
::ExceptT
(ActionError e)
(ReaderT ActionEnv
(StateTScottyResponse m))
a
}deriving (Functor,Applicative )
Let’s first replace the liftforActionT with its definition and
see if it still works:

CHAPTER 26. STACK ‘EM UP 1583
{-# LANGUAGE OverloadedStrings #-}
moduleScottywhere
importWeb.Scotty
importWeb.Scotty.Internal.Types
(ActionT(..))
importControl.Monad.Trans.Class
importData.Monoid (mconcat)
All the (..)means is that we want to import all the data
constructors of the ActionT type, rather than none or a partic-
ular list of them. You can look into the syntax in more detail
independently if you like. Now for the scotty application itself:
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
(ActionT .lift.lift.lift)
(putStrLn "hello")
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]

CHAPTER 26. STACK ‘EM UP 1584
This should still work! Note that we had to ask for the data
constructor for ActionT from an Internal module because the
implementation is hidden by default. We’ve got three lifts,
one each for ExceptT ,ReaderT , andStateT .
Next we’ll do ExceptT :
instance MonadTrans (ExceptT e)where
lift=ExceptT .liftMRight
To use that in our code, add the following import:
importControl.Monad.Trans.Except
And our app changes into the following:
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
(ActionT
.(ExceptT .liftMRight)
.lift
.lift) (putStrLn "hello")
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]

CHAPTER 26. STACK ‘EM UP 1585
Thenfor ReaderT ,wetakeaganderat Control.Monad.Trans.Reader
in thetransformers library and see the following:
instance MonadTrans (ReaderT r)where
lift=liftReaderT
liftReaderT ::m a->ReaderT r m a
liftReaderT m=ReaderT (const m)
For reasons, liftReaderT isn’t exported by transformers , but
we can redefine it ourselves. Add the following to the module:
importControl.Monad.Trans.Reader
liftReaderT ::m a->ReaderT r m a
liftReaderT m=ReaderT (const m)
Then our app can be defined as follows:

CHAPTER 26. STACK ‘EM UP 1586
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
(ActionT
.(ExceptT .fmapRight)
.liftReaderT
.lift
) (putStrLn "hello")
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]
Or instead of liftReaderT , we could’ve done:
.(\m->ReaderT (const m))
Or:
(ActionT
.(ExceptT .fmapRight)
.ReaderT .const
.lift
) (putStrLn "hello")

CHAPTER 26. STACK ‘EM UP 1587
Now for that last liftoverStateT ! Remembering that it was
the lazy StateT that the type of ActionT mentioned, we see the
following MonadTrans instance:
instance MonadTrans (StateTs)where
lift m=StateT$\s-> do
a<-m
return (a, s)
First, let’s get our import in place:
importControl.Monad.Trans.State.Lazy
hiding(get)
We needed to hide getbecause scotty already has a diﬀerent
getfunction defined and we don’t need the one from StateT.
Then inlining that into our app code:

CHAPTER 26. STACK ‘EM UP 1588
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
(ActionT
.(ExceptT .fmapRight)
.ReaderT .const
.\m->StateT(\s-> do
a<-m
return (a, s))
) (putStrLn "hello")
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]
Note that we needed an outer lambda before the StateT
in order to get the monadic action we were lifting. At this
point, we’re in the outermost position we can be, and since
ActionM defines ActionT ’s outermost monadic type as being IO,
that means our putStrLn works fine after all this lifting.
Typically a MonadTrans instance lifts over only one layer at
a time, but scotty abstracts away the underlying structure so
that you don’t have to care. That’s why it goes ahead and does
the next three lifts for you. The critical thing to realize here is
that lifting means you’re embedding an expression in a larger

CHAPTER 26. STACK ‘EM UP 1589
context by adding structure that doesn’t do anything.
MonadTrans instances
Now you see why we have MonadTrans and have a picture of
whatlift, the only method of MonadTrans , does.
Here are some examples of MonadTrans instances:
1.IdentityT
instance MonadTrans IdentityT where
lift=IdentityT
2.MaybeT
instance MonadTrans MaybeTwhere
lift=MaybeT.liftMJust

CHAPTER 26. STACK ‘EM UP 1590
lift
::(Monadm)
=>m a->t m a
(MaybeT.liftMJust)
::Monadm
=>m a->MaybeTm a
MaybeT
::m (Maybea)->MaybeTm a
(liftMJust)
::Monadm
=>m a->m (Maybea)
Roughly speaking, this has taken an m aand lifted it into
aMaybeT context.
Thegeneralpatternwith MonadTrans instancesdemonstrated
byMaybeT is that you’re usually going to lift the injection
of the known structure (with MaybeT , the known structure
isMaybe) over some Monad. Injection of structure usually
meansreturn , but since with MaybeT we know we want Maybe
structure, we use Just. That transforms an m aintom (T a)
where capital Tis some concrete type you’re lifting the m
ainto. Then to cap it all oﬀ, you use the data constructor
for your monad transformer, and the value is now lifted
into the larger context. Here’s a summary of the stages

CHAPTER 26. STACK ‘EM UP 1591
the type of the value goes through:
v::Monadm=>m a
liftMJust::Monadm=>m a->m (Maybea)
liftMJustv::m (Maybea)
MaybeT(liftMJustv)::MaybeTm a
See if you can work out the types of this one:
3.ReaderT
instance MonadTrans (ReaderT r)where
lift=ReaderT .const
And now, write some instances!
Exercises: Lift More
Keep in mind what these are doing, follow the types, lift till
you drop.
1.You thought you were done with EitherT .
instance MonadTrans (EitherT e)where
lift=undefined
2.OrStateT . This one’ll be more obnoxious. It’s fine if you’ve
seen this before.

CHAPTER 26. STACK ‘EM UP 1592
instance MonadTrans (StateTs)where
lift=undefined
Prolific lifting is the failure mode
Apologies to the original authors, but sometimes with the use
of concretely and explicitly typed monad transformers you’ll
see stuﬀ like this:
addSubWidget ::(YesodSubRoute sub master)
=>sub
->WidgetT sub master a
->WidgetT sub' master a
addSubWidget sub w=
domaster<-liftHandler getYesod
letsr=fromSubRoute sub master
i<-WidgetT $lift$lift$lift
$lift$lift$lift
$lift get

CHAPTER 26. STACK ‘EM UP 1593
w'<-liftHandler
$toMasterHandlerMaybe sr
(const sub) Nothing
$flip runStateT i $runWriterT
$runWriterT $runWriterT
$runWriterT $runWriterT
$runWriterT $runWriterT
$unWidgetT w
let((((((((a,
body),
title),
scripts),
stylesheets),
style),
jscript),
h),
i')=w'

CHAPTER 26. STACK ‘EM UP 1594
WidgetT $ do
tell body
lift$tell title
lift$lift$tell scripts
lift$lift$lift
$tell stylesheets
lift$lift$lift$lift
$tell style
lift$lift$lift$lift$lift
$tell jscript
lift$lift$lift$lift$lift
$lift$tell h
lift$lift$lift$lift
$lift$lift$lift$put i'
return a
Do not write code like this. Especially, do not write code
like this and then proceed to blog about how terrible monad
transformers are.
Wrap it, smack it, pre-lift it
OK, so how do we avoid that horror show? Well, there are a lot
of ways, but one of the most robust and common is newtyp-
ing your Monadstack and abstracting away the representation.

CHAPTER 26. STACK ‘EM UP 1595
From there, you provide the functionality leveraging the rep-
resentation as part of your API. A good example of this comes
to us from… scotty .
Let’s take a gander at the ActionM type we mentioned earlier:
Prelude> import Web.Scotty
-- again, to make the type read more nicely
-- we import some other modules.
Prelude> import Data.Text.Lazy
Prelude> :info ActionM
type ActionM = Web.Scotty.Internal.Types.ActionT Text IO
-- Defined in ‘Web.Scotty’
scotty hides the underlying type by default because you
ordinarily wouldn’t care or think about it in the course of writ-
ing your application. What scotty does here is good practice.
This design keeps the underlying implementation hidden by
default but lets us import an Internal module to get at the
representation in case we need to:
Prelude> import Web.Scotty.Internal.Types
-- more modules to clean up the types
Prelude> import Control.Monad.Trans.Reader
Prelude> import Control.Monad.Trans.State.Lazy
Prelude> import Control.Monad.Trans.Except
Prelude> :info ActionT

CHAPTER 26. STACK ‘EM UP 1596
type role ActionT nominal representational nominal
newtype ActionT e (m :: * -> *) a
= ActionT
{runAM :: ExceptT
(ActionError e)
(ReaderT ActionEnv
(StateT ScottyResponse m))
a}
instance (Monad m, ScottyError e) => Monad (ActionT e m)
instance Functor m => Functor (ActionT e m)
instance Monad m => Applicative (ActionT e m)
What’s nice about this approach is that it subjects the con-
sumers (which could include yourself) of your type to less
noise within an application. It also doesn’t require reading
papers written by people trying very hard to impress a thesis
advisor, although poking through prior art for ideas is rec-
ommended. It can reduce or eliminate manual lifting within
theMonadas well. Note that we only had to use liftonce to
perform an I/O action in ActionM even though the underly-
ing implementation has more than one transformer flying
around.

CHAPTER 26. STACK ‘EM UP 1597
26.10 MonadIO aka zoom-zoom
There’s more than one way to skin a cat and there’s more than
one way to lift an action over additional structure. MonadIO is
a diﬀerent design than MonadTrans because rather than lifting
through one layer at a time, MonadIO is intended to keep lifting
yourIOaction until it is lifted over all structure embedded in
the outermost IOtype. The idea here is that you’d write liftIO
once and it would instantiate to all of the following types:
liftIO::IOa->ExceptT eIOa
liftIO::IOa->ReaderT rIOa
liftIO::IOa->StateTsIOa
-- As Sir Mix-A-Lot once said,
-- stack 'em up deep
liftIO::IOa->StateTs (ReaderT rIO) a
liftIO::IOa
->ExceptT
e
(StateTs (ReaderT rIO))
a
You don’t have to lift multiple times if you’re trying to reach
a base (outermost) Monadthat happens to be IO, because you
haveliftIO .

CHAPTER 26. STACK ‘EM UP 1598
In thetransformers library, the MonadIO class resides in the
module Control.Monad.IO.Class :
class(Monadm)=>MonadIO mwhere
-- | Lift a computation
-- from the 'IO' monad.
liftIO::IOa->m a
The commentary within the module is reasonably helpful,
though it doesn’t highlight what makes MonadIO diﬀerent from
MonadTrans :
Monads in which IO computations may be embed-
ded. Any monad built by applying a sequence of
monad transformers to the IO monad will be an in-
stance of this class.
Instances should satisfy the following laws, which
state that liftIO is a transformer of monads:
1.liftIO.return=return
2.liftIO(m>>=f)=
liftIO m >>=(liftIO .f)
Let us modify the scotty example app to print a string:

CHAPTER 26. STACK ‘EM UP 1599
{-# LANGUAGE OverloadedStrings #-}
moduleMainwhere
importWeb.Scotty
importControl.Monad.IO.Class
importData.Monoid (mconcat)
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
liftIO (putStrLn "hello")
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]
If you then run mainin a REPL or build a binary and execute
it, you’ll be able to request a response from the server using
your web browser (as we showed you earlier) or a command
line application like curl. If you used a browser and see “hello”
printed more than once, it’s likely your browser made the
request more than once. You shouldn’t see this behavior if
you test it with curl.

CHAPTER 26. STACK ‘EM UP 1600
Example MonadIO instances
1.IdentityT
instance (MonadIO m)
=>MonadIO (IdentityT m)where
liftIO=IdentityT .liftIO
2.EitherT
instance (MonadIO m)
=>MonadIO (EitherT e m)where
liftIO=lift.liftIO
Exercises: Some Instances
1.MaybeT
instance (MonadIO m)
=>MonadIO (MaybeTm)where
liftIO=undefined
2.ReaderT
instance (MonadIO m)
=>MonadIO (ReaderT r m)where
liftIO=undefined
3.StateT

CHAPTER 26. STACK ‘EM UP 1601
instance (MonadIO m)
=>MonadIO (StateTs m)where
liftIO=undefined
Hint: your instances should be simple.
26.11 Monad transformers in use
MaybeT in use
These are some example of MaybeT in use; we will not comment
upon them and instead let you research them further yourself
if you want. Origins of the code are noted in the samples.
-- github.com/wavewave/hoodle-core
recentFolderHook
::MainCoroutine (MaybeFilePath )
recentFolderHook = do
xstate<-get
(r::MaybeFilePath )<-runMaybeT $ do
hset<-hoist (view hookSet xstate)
rfolder <-
hoist (H.recentFolderHook hset)
liftIO rfolder
return r

CHAPTER 26. STACK ‘EM UP 1602
-- github.com/devalot/hs-exceptions
-- src/maybe.hs
addT::FilePath
->FilePath
->IO(MaybeInteger)
addTf1 f2=runMaybeT $ do
s1<-sizeT f1
s2<-sizeT f2
return (s1 +s2)

CHAPTER 26. STACK ‘EM UP 1603
-- wavewave/ghcjs-dom-delegator
-- example/Example.hs
main::IO()
main= do
clickbarref <-
asyncCallback1 AlwaysRetain clickbar
clickbazref <-
asyncCallback1 AlwaysRetain clickbaz
r<-runMaybeT $ do
doc<-MaybeTcurrentDocument
bar<-lift.toJSRef
=<<MaybeT
(documentQuerySelector doc
(".bar"::JSString ))
baz<-lift.toJSRef
=<<MaybeT
(documentQuerySelector doc
(".baz"::JSString ))
lift$ do
ref<-newObj
del<-delegator ref
addEvent bar "click" clickbarref
addEvent baz "click" clickbazref
caserof
Nothing ->print"something wrong"
Just_ ->print"welldone"

CHAPTER 26. STACK ‘EM UP 1604
Temporary extension of structure
Although we commonly think of monad transformers as being
used to define one big context for an application, particularly
with things like ReaderT , there are other ways. One pattern that
is often useful is temporarily extending additional structure
to avoid boilerplate. Here’s an example using plain old Maybe
andscotty :

CHAPTER 26. STACK ‘EM UP 1605
{-# LANGUAGE OverloadedStrings #-}
moduleMainwhere
importControl.Monad.IO.Class
importData.Maybe (fromMaybe )
importData.Text.Lazy (Text)
importWeb.Scotty
param'::Parsable a
=>Text->ActionM (Maybea)
param'k=rescue ( Just<$>param k)
(const (return Nothing))
main=scotty3000$ do
get"/:word" $ do
beam'<-param'"word"
letbeam=fromMaybe ""beam'
i<-param'"num"
liftIO$print (i ::MaybeInteger)
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]

CHAPTER 26. STACK ‘EM UP 1606
This works well enough but could get tedious in a hurry if
we had a bunch of stuﬀ that returned ActionM (Maybe ...) and
we wanted to short-circuit the moment any of them failed.
So, we do something similar but with MaybeT and building up
more data in one go:

CHAPTER 26. STACK ‘EM UP 1607
{-# LANGUAGE OverloadedStrings #-}
moduleMainwhere
importControl.Monad.IO.Class
importControl.Monad.Trans.Class
importControl.Monad.Trans.Maybe
importData.Maybe (fromMaybe )
importData.Text.Lazy (Text)
importWeb.Scotty
param'::Parsable a
=>Text->MaybeTActionM a
param'k=MaybeT$
rescue ( Just<$>param k)
(const (return Nothing))
typeReco=
(Integer,Integer,Integer,Integer)
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
reco<-runMaybeT $ do
a<-param'"1"
liftIO$print a
b<-param'"2"
c<-param'"3"
d<-param'"4"
(lift.lift)$print b
return ((a, b, c, d) ::Reco)
liftIO$print reco
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]

CHAPTER 26. STACK ‘EM UP 1608
Some important things to note here:
1.We only had to use liftIO once, even in the presence of
additional structure, whereas with liftwe had to lift twice
to address MaybeT andActionM .
2.The one big bind of the MaybeT means we could take the
existence of 𝑎,𝑏,𝑐, and𝑑for granted in that context, but
therecovalue itself is Maybe Reco because any part of the
computation could fail in the absence of the needed pa-
rameter.
3.It knows what monad we mean for that doblock because
of therunMaybeT in front of the do. This serves the dual
purpose of unpacking the MaybeT into an ActionM (Maybe
Reco)which we can bind out into Maybe Reco .
ExceptT aka EitherT in use
The example with Maybeandscotty may not have totally satis-
fied because the failure mode isn’t helpful to an end-user —
all they know is “Nothing.” Accordingly, Maybeis usually some-
thing that should get handled early and often in a place local
to where it was produced so that you avoid mysterious Nothing
values floating around and short-circuiting your code. They’re
not something you want to return to end-users either. Fortu-
nately, we have Either for more descriptive short-circuiting
computations!

CHAPTER 26. STACK ‘EM UP 1609
Scotty, again
We’ll use scotty again to demonstrate this. Once again, we’ll
show you a plain example:
{-# LANGUAGE OverloadedStrings #-}
moduleMainwhere
importControl.Monad.IO.Class
importData.Text.Lazy (Text)
importWeb.Scotty
param'::Parsable a
=>Text->ActionM (EitherStringa)
param'k=
rescue ( Right<$>param k)
(const
(return
(Left$"The key: "
++show k
++" was missing!" )))

CHAPTER 26. STACK ‘EM UP 1610
main=scotty3000$ do
get"/:word" $ do
beam<-param"word"
a<-param'"1"
leta'=either (const 0) id a
liftIO$print (a ::EitherStringInt)
liftIO$print (a' ::Int)
html$
mconcat [ "<h1>Scotty, " ,
beam,
" me up!</h1>" ]
Note that we had to manually fold the Either if we wanted
to address the desired Intvalue. Try to avoid having default
fallback values in real code though. This could get nutty in
a hurry if we had many things we were pulling out of the
parameters.
Let’s do that but with ExceptT fromtransformers . Remember,
ExceptT is another name for EitherT :

CHAPTER 26. STACK ‘EM UP 1611
{-# LANGUAGE OverloadedStrings #-}
moduleMainwhere
importControl.Monad.IO.Class
importControl.Monad.Trans.Class
importControl.Monad.Trans.Except
importData.Text.Lazy (Text)
import qualified Data.Text.Lazy asTL
importWeb.Scotty

CHAPTER 26. STACK ‘EM UP 1612
param'::Parsable a
=>Text->ExceptT StringActionM a
param'k=
ExceptT $
rescue ( Right<$>param k)
(const
(return
(Left$"The key: "
++show k
++" was missing!" )))
typeReco=
(Integer,Integer,Integer,Integer)
tshow=TL.pack.show

CHAPTER 26. STACK ‘EM UP 1613
main=scotty3000$ do
get"/"$ do
reco<-runExceptT $ do
a<-param'"1"
liftIO$print a
b<-param'"2"
c<-param'"3"
d<-param'"4"
(lift.lift)$print b
return ((a, b, c, d) ::Reco)
caserecoof
(Lefte)->text (TL.pack e)
(Rightr)->
html$
mconcat [ "<h1>Success! Reco was: " ,
tshow r,
"</h1>"]
If you pass it a request like:
http://localhost:3000/?1=1
It’ll ask for the parameter 2because that was the next param
you asked for after 1.
If you pass it a request like:
http://localhost:3000/?1=1&2=2&3=3&4=4

CHAPTER 26. STACK ‘EM UP 1614
You should see the response in your browser or terminal
of:
Success! Reco was: (1,2,3,4)
As before, we get to benefit from one big bind under the
ExceptT .
Slightly more advanced code
From some code7by Sean Chalmers8.
Some context for the EitherT application you’ll see:
typeEta=EitherT SDLErrIOa
mkWindow ::HasSDLErr m=>
String
->CInt->CInt
->mSDL.Window
mkRenderer ::HasSDLErr m
=>SDL.Window->mSDL.Renderer
7https://github.com/mankyKitty/Meteor/
8http://mankykitty.github.io/

CHAPTER 26. STACK ‘EM UP 1615
hasSDLErr ::(MonadIO m,MonadError e m)
=>(a->b)
->(a->Bool)
->e->IOa->m b
hasSDLErr g f e a =
liftIO a
>>=\r->
bool (return $g r)
(throwError e) $f r
class(MonadIO m,MonadError SDLErrm)
=>HasSDLErr mwhere
decide ::(a->Bool)
->SDLErr->IOa->m a
decide' ::(Eqn,Numn)
=>SDLErr->IOn->m()
instance HasSDLErr
(EitherT SDLErrIO)where
decide =hasSDLErr id
decide' =hasSDLErr (const ()) (/=0)
Then in use:

CHAPTER 26. STACK ‘EM UP 1616
initialise ::Et(SDL.Window,SDL.Renderer )
initialise = do
initSDL [ SDL.SDL_INIT_VIDEO ]
win<-
mkWindow "Meteor!"
screenHeight
screenWidth
rdr<-mkRenderer win
return (win,rdr)
createMeteor ::IO(EitherSDLErrMeteorS)
createMeteor = do
eM<-runEitherT initialise
return$mkMeteor <$>eM
where
emptyBullets =V.empty
mkMeteor (w,r) =MeteorS w r
getInitialPlayer
emptyBullets
getInitialMobs
False

CHAPTER 26. STACK ‘EM UP 1617
26.12 Monads do not commute
Remember that monads in general do not commute, and
you aren’t guaranteed something sensible for every possible
combination of types. The kit we have for constructing and
using monad transformers is useful but is not a license to not
think!
Hypothetical Exercise
Consider ReaderT r Maybe andMaybeT (Reader r) — are these
types equivalent? Do they do the same thing? Try writing
otherwise similar bits of code with each and see if you can
prove they’re the same or diﬀerent.
26.13 Transform if you want to
If you find monad transformers difficult or annoying, then
don’t bother! Most of the time you can get by with liftIO and
plainIOactions, functions, Maybevalues, etc. Do the simplest
(for you) thing first when mapping out something new or un-
familiar. It’s better to let more structured formulations of
programs fall out naturally from having kicked around some-
thing uncomplicated than to blow out your working memory
budget in one go. Don’t worry about seeming unsophisticated;

CHAPTER 26. STACK ‘EM UP 1618
in our opinion, being happy and productive is better than
being fancy.
Keep it basic in your first attempt. Never make it more
elaborate initially than is strictly necessary. You’ll figure out
when the transformer variant of a type will save you complex-
ity in the process of writing your programs. We have taken
you through these topics because you’ll need at least a passing
familiarity to use modern Haskell libraries or frameworks, but
it’s not a design dictate you must follow.
26.14 Chapter Exercises
Write the code
1.rDecis a function that should get its argument in the con-
text ofReader and return a value decremented by one.
rDec::Numa=>Readera a
rDec=undefined
Prelude> import Control.Monad.Trans.Reader
Prelude> runReader rDec 1
0
Prelude> fmap (runReader rDec) [1..10]
[0,1,2,3,4,5,6,7,8,9]

CHAPTER 26. STACK ‘EM UP 1619
Note that “Reader” from transformers isReaderT ofIdentity
and that runReader is a convenience function throwing
awaythemeaninglessstructureforyou. Playwith runReaderT
if you like.
2.Once you have an rDecthat works, make it and any inner
lambdas pointfree if that’s not already the case.
3.rShowisshow, but in Reader .
rShow::Showa
=>ReaderT aIdentity String
rShow=undefined
Prelude> runReader rShow 1
"1"
Prelude> fmap (runReader rShow) [1..10]
["1","2","3","4","5","6","7","8","9","10"]
4.Once you have an rShowthat works, make it pointfree.
5.rPrintAndInc will first print the input with a greeting, then
return the input incremented by one.
rPrintAndInc ::(Numa,Showa)
=>ReaderT aIOa
rPrintAndInc =undefined

CHAPTER 26. STACK ‘EM UP 1620
Prelude> runReaderT rPrintAndInc 1
Hi: 1
2
Prelude> traverse (runReaderT rPrintAndInc) [1..10]
Hi: 1
Hi: 2
Hi: 3
Hi: 4
Hi: 5
Hi: 6
Hi: 7
Hi: 8
Hi: 9
Hi: 10
[2,3,4,5,6,7,8,9,10,11]
6.sPrintIncAccum first prints the input with a greeting, then
puts the incremented input as the new state, and returns
the original input as a String .
sPrintIncAccum ::(Numa,Showa)
=>StateTaIOString
sPrintIncAccum =undefined
Prelude> runStateT sPrintIncAccum 10
Hi: 10

CHAPTER 26. STACK ‘EM UP 1621
("10",11)
Prelude> mapM (runStateT sPrintIncAccum) [1..5]
Hi: 1
Hi: 2
Hi: 3
Hi: 4
Hi: 5
[("1",2),("2",3),("3",4),("4",5),("5",6)]
Fix the code
The code won’t typecheck as written; fix it so that it does. Feel
free to add imports if it provides something useful. Functions
will be used that we haven’t introduced. You’re not allowed
to change the types asserted. You may have to fix the code in
more than one place.
importControl.Monad.Trans.Maybe
importControl.Monad
isValid ::String->Bool
isValid v='!'`elem` v

CHAPTER 26. STACK ‘EM UP 1622
maybeExcite ::MaybeTIOString
maybeExcite = do
v<-getLine
guard$isValid v
return v
doExcite ::IO()
doExcite = do
putStrLn "say something excite!"
excite<-maybeExcite
caseexciteof
Nothing ->putStrLn "MOAR EXCITE"
Juste->
putStrLn
("Good, was very excite: " ++e)
Hit counter
We’re going to provide an initial scaﬀold of a scotty application
which counts hits to specific URIs. It also prefixes the keys
with a prefix defined on app initialization, retrieved via the
command line arguments.

CHAPTER 26. STACK ‘EM UP 1623
{-# LANGUAGE OverloadedStrings #-}
moduleMainwhere
importControl.Monad.Trans.Class
importControl.Monad.Trans.Reader
importData.IORef
import qualified Data.Map asM
importData.Maybe (fromMaybe )
importData.Text.Lazy (Text)
import qualified Data.Text.Lazy asTL
importSystem.Environment (getArgs)
importWeb.Scotty.Trans
dataConfig=
Config{
-- that's one, one click!
-- two...two clicks!
-- Three BEAUTIFUL clicks! ah ah ahhhh
counts::IORef(M.MapTextInteger)
, prefix ::Text
}
Stuﬀ inside ScottyT is, except for things that escape via IO,
eﬀectively read-only so we can’t use StateT . It would overcom-

CHAPTER 26. STACK ‘EM UP 1624
plicate things to attempt to do so and you should be using a
proper database for production applications.
typeScotty=
ScottyT Text(ReaderT ConfigIO)
typeHandler =
ActionT Text(ReaderT ConfigIO)
bumpBoomp ::Text
->M.MapTextInteger
->(M.MapTextInteger,Integer)
bumpBoomp k m=undefined
app::Scotty()
app=
get"/:key" $ do
unprefixed <-param"key"
letkey'=mappend undefined unprefixed
newInteger <-undefined
html$
mconcat [ "<h1>Success! Count was: "
,TL.pack$show newInteger
,"</h1>"
]

CHAPTER 26. STACK ‘EM UP 1625
main::IO()
main= do
[prefixArg] <-getArgs
counter <-newIORef M.empty
letconfig=undefined
runR=undefined
scottyT 3000runR app
Code is missing and broken. Your task is to make it work,
whatever is necessary.
You should be able to run the server from inside of GHCi,
passing arguments like so:
Prelude> :main lol
Setting phasers to stun... (port 3000) (ctrl-c to quit)
You could also build a binary and pass the arguments from
your shell, but do what you like. Once it’s running, you should
be able to bump the counts like so:
$ curl localhost:3000/woot
<h1>Success! Count was: 1</h1>
$ curl localhost:3000/woot
<h1>Success! Count was: 2</h1>
$ curl localhost:3000/blah
<h1>Success! Count was: 1</h1>

CHAPTER 26. STACK ‘EM UP 1626
Note that the underlying “key” used in the counter when
youGET /woot is"lolwoot" because we passed ”lol” to main. For
a giggle, try the URI for one of the keys in your browser and
mash refresh a bunch.
If you get stuck, consider checking for examples such as the
reader file in scotty ’s examples directory of the git repository.
Morra
1.Write the game Morra9usingStateT andIO. The state
being accumulated is the score of the player and the com-
puter AI the player is playing against. To start, make the
computer choose its play randomly.
On exit, report the scores for the player and the computer,
congratulating the winner.
2.Add a human vs. human mode to the game with inter-
stitial screens between input prompts so the players can
change out of the hotseat without seeing the other player’s
answer.
3.Improve the computer AI slightly by making it remem-
ber 3-grams of the player’s behavior, adjusting its answer
instead of deciding randomly when the player’s behavior
matches a known behavior. For example:
9You can find descriptions of the rules and gameplay of the Morra game online.

CHAPTER 26. STACK ‘EM UP 1627
-- p is Player
-- c is Computer
-- Player is odds, computer is evens.
P: 1
C: 1
- C wins
P: 2
C: 1
- P wins
P: 2
C: 1
- P wins
At this point, the computer should register the pattern (1,
2, 2) player picked 2 after 1 and 2. Next time the player
picks 1 followed by 2, the computer should assume the
next play will be 2 and pick 2 in order to win.
4.The 3-gram thing is pretty simple and dumb. Humans are
still bad at being random; they often have sub-patterns
in their moves.
26.15 Defintion
In general, the term leak refers to something that consumes
a resource in a way that renders it unusable or irrecoverable;

CHAPTER 26. STACK ‘EM UP 1628
specifically, when we talk about a memory leak, we’re talking
about consuming memory in a way that renders it not usable
or recoverable by other programs or parts of a program. This
can happen if your program is written in such a way that it
accumulates large amounts of unevaluated thunks or holds in
memory a reference to something that it’s not using anymore.
The garbage collector cannot sweep those things away, so the
amount of memory a program is using can increase, some-
times rapidly and alarmingly, while the amount of available
or free memory decreases.
26.16 Follow-up resources
1.Parallel and Concurrent Programming in Haskell; Simon
Marlow; http://chimera.labs.oreilly.com/books/1230000000929

Chapter 27
Nonstrictness
Progress doesn’t come
from early risers —
progress is made by lazy
men looking for easier
ways to do things.
Robert A. Heinlein
1629

CHAPTER 27. NONSTRICTNESS 1630
27.1 Laziness
This chapter concerns the ways in which Haskell programs are
evaluated. We’ve addressed this a bit in previous chapters, for
example, in the Folds chapter where we went into some detail
about how folds evaluate. In this chapter, our goal is to give
you enough information about Haskell’s evaluation strategy
that you’ll be able to reason confidently about the reduction
process of your expressions and introduce stricter evaluation
where that is wanted.
Most programming languages have strict evaluation seman-
tics. Haskell technically has nonstrict — not lazy — evaluation,
but the diﬀerence between lazy and nonstrict is not practi-
cally relevant, so you’ll hear Haskell referred to as either a lazy
language or a nonstrict one.
A very rough outline of Haskell’s evaluation strategy is this:
most expressions are only reduced, or evaluated, when neces-
sary. When the evaluation process begins, a thunk is created
for each expression. We’ll go into more detail about this in
the chapter, but a thunk is like a placeholder in the underly-
ing graph of the program. Whatever expression the thunk is
holding a place for can be evaluated when necessary, but if
it’s never needed, it never gets reduced, and then the garbage
collector comes along and sweeps it away. If it is evaluated,
because it’s in a graph, it can be often shared between expres-
sions — that is, once x = 1 + 1 has been evaluated, anytime 𝑥

CHAPTER 27. NONSTRICTNESS 1631
is forced it does not have to be re-computed.
This is the laziness of Haskell: don’t do more work than
needed. Don’t evaluate until necessary. Don’t re-evaluate if
you don’t have to. We’ll go through the details of how this
works, exceptions to the general principles, and how to control
the evaluation by adding strictness where desired.
Specifically, we will:
•define call-by-name and call-by-need evaluation;
•explain the main eﬀects of nonstrict evaluation;
•live the Thunk Life1;
•consider the runtime behavior of non-strict code in terms
of sharing;
•developmethodsforobservingsharingbehaviorandmea-
suring program efficiency;
•bottom out with the bottoms.
27.2 Observational Bottom Theory
In our discussion about nonstrictness in Haskell, we’re going
to be talking about bottom2a lot. This is partly because non-
strictness is defined by the ability to evaluate expressions that
1We love you, Jesse!
2Observational bottom theory is not a real thing. Do not email us about this.

CHAPTER 27. NONSTRICTNESS 1632
have bindings which are bottom in them, as long as the bot-
tom itself is never forced. Bottoms also give us a convenient
method of observing evaluation in Haskell. By causing the
program to halt immediately with an error, bottom serves as
our first means of understanding nonstrictness in Haskell. You
probably recall we have used this trick before.
Standards and obligations
Technically Haskell is only obligated to be nonstrict, not lazy.
A truly lazy language memoizes, or holds in memory, the
results of all the functions it does evaluate, and, outside of toy
programs, this tends to use unacceptably large amounts of
memory. Implementations of Haskell, such as GHC Haskell,
are only obligated to be nonstrict such that they have the same
behavior with respect to bottom; they are not required to take
a particular approach to how the program executes or how
efficiently it does so.
The essence of nonstrictness is that you can have an expres-
sion which results in a value, even if bottom or infinite data
lurks within. For example, the following would only work in a
nonstrict language:
Prelude> fst (1, undefined)
1
Prelude> snd (undefined, 2)
2

CHAPTER 27. NONSTRICTNESS 1633
The idea is that any given implementation of nonstrictness
is acceptable as long as it respects when it’s supposed to return
a value successfully or bottom out.
27.3 Outside in, inside out
Strict languages evaluate inside out; nonstrict languages like
Haskell evaluate outside in. Outside in means that evaluation
proceeds from the outermost parts of expressions and works
inward based on what values are forced. This means the order
of evaluation and what gets evaluated can vary depending on
inputs.
The following example is written in a slightly arcane way
to make the evaluation order more obvious:
possiblyKaboom =
\f->f fst snd ( 0, undefined)
-- booleans in lambda form
true::a->a->a
true=\a->(\b->a)
false::a->a->a
false=\a->(\b->b)
When we apply possiblyKaboom totrue,trueis the𝑓,fstis
the𝑎, andsndis the𝑏. Semantically, case matches, guards

CHAPTER 27. NONSTRICTNESS 1634
expressions, and if-then-else expressions could all be rewritten
in this manner (they are not in fact decomposed this way by the
compiler), by nesting lambdas and reducing from the outside
in:
(\f->
f fst snd ( 0, undefined))
(\a->(\b->a))
(\a->(\b->a)) fst snd ( 0, undefined)
(\b->fst) snd ( 0, undefined)
fst(0, undefined)
0
The next example is written in more normal Haskell but
will return the same result. When we apply the function to
Truehere, we case on the Trueto return the first value of the
tuple:
possiblyKaboom b=
casebof
True->fst tup
False->snd tup
wheretup=(0, undefined)
The bottom is inside a tuple, and the tuple is bound inside
of a lambda that cases on a boolean value and returns either the
first or second element of the tuple. Since we start evaluating

CHAPTER 27. NONSTRICTNESS 1635
from the outside, as long as this function is only ever applied
toTrue, that bottom will never cause a problem. However, at
the risk of stating the obvious, we do not encourage you to
write programs with bottoms lying around willy-nilly.
When we say evaluation works outside in, we’re talking
about evaluating a series of nested expressions, and not only
are we starting from the outside and working in, but we’re also
only evaluating some of the expressions some of the time. In
Haskell, we evaluate expressions when we need them rather
than when they are first referred to or constructed. This is one
of the ways in which nonstrictness makes Haskell expressive
— we can refer to values before we’ve done the work to create
them.
This pattern applies to data structures and lambdas alike.
You’ve already seen the eﬀects of outside-in evaluation in the
chapter on folds. Outside-in evaluation is why we can take the
length of a list without touching any of the contents. Consider
the following:
-- using an old definition of foldr
foldrk z xs=go xs
where
go[]=z
go (y:ys)=y `k` go ys
c=foldr const 'z'['a'..'e']

CHAPTER 27. NONSTRICTNESS 1636
Expanding the foldrin𝑐:
c=const'z'"abcde" =go"abcde"
where
go[]='z'
go ('a':"bcde")='a'`const` go "bcde"
-- So the first step of evaluating
-- of the fold here is:
const'a'(go"bcde")
constx y =x
const'a'(go"bcde")='a'
The second argument and step of the fold is never evalu-
ated:
const'a'_ ='a'
It doesn’t even matter if the next value is bottom:
Prelude> foldr const 'z' ['a', undefined]
'a'
This is outside-in showing itself. The constfunction was in
the outermost position so it was evaluated first.

CHAPTER 27. NONSTRICTNESS 1637
27.4 What does the other way look like?
In strict languages, you cannot ordinarily bind a computa-
tion to a name without having already done all the work to
construct it.
We’ll use this example program to compare inside-out and
outside-in (strict and non-strict) evaluation strategies:
moduleOutsideIn where
hypo::IO()
hypo= do
letx::Int
x=undefined
s<-getLine
casesof
"hi"->print x
_ ->putStrLn "hello"
For a strict language, this is a problem. A strict language
cannot evaluate hyposuccessfully unless the 𝑥isn’t bottom.
This is because strict languages will force the bottom before
binding 𝑥. A strict language is evaluating each binding as it
comes into scope, not when a binding is used.
In nonstrict Haskell, you can probably guess how this’ll go:
Prelude> hypo

CHAPTER 27. NONSTRICTNESS 1638
s
hello
Prelude> hypo
hi
*** Exception: Prelude.undefined
The idea is that evaluation is driven by demand, not by
construction. We don’t get the exception unless we’re forcing
evaluation of 𝑥— outside in.
27.5 Can we make Haskell strict?
Let’s see if we can replicate the results of a strict language,
though, which will give us a good picture of how Haskell is
diﬀerent. We can add strictness here in the following manner:
hypo'::IO()
hypo'= do
letx::Integer
x=undefined
s<-getLine
casex `seq` s of
"hi"->print x
_ ->putStrLn "hello"
Running it will give this result:

CHAPTER 27. NONSTRICTNESS 1639
Prelude> hypo'
asd
*** Exception: Prelude.undefined
Why? Because this little seqfunction magically forces eval-
uation of the first argument if and when the second argument
has to be evaluated. Adding seqmeans that anytime 𝑠is evalu-
ated,𝑥must also be evaluated. We’ll get into more detail in a
moment.
One thing to note before we investigate seqis that we man-
aged to run getLine before the bottom got evaluated, so this
still isn’t quite what a strict language would’ve done. Case
expressions are in general going to force evaluation. This
makes sense if you realize it has to evaluate the expression to
discriminate on the cases. A small example to demonstrate:
letb= ???
casebof
True-> ...
False
Here𝑏could be pretty much anything. It must evaluate 𝑏
to find out if the expression results in TrueorFalse.

CHAPTER 27. NONSTRICTNESS 1640
seq and ye shall find
Before we move any further with making Haskell stricter, let’s
talk about seqa little bit. One thing is that the type is, uh, a bit
weird:
seq::a->b->b
Clearly there’s more going on here than flip const . It might
help to know that in some old versions of Haskell, it used to
have the type:
seq::Evala=>a->b->b
Evalis short for evaluation to weak head normal form, and
it provided a method for forcing evaluation. Instances were
provided for all the types in base. It was elided in part so you
could use seqin your code without churning your polymor-
phic type variables and forcing a bunch of changes. With
respect to bottom, seqis defined as behaving in the following
manner:
seqbottom b =bottom
seqliterallyAnythingNotBottom b =b
Now why does seqlook like const’s gawky cousin? Because
evaluation in Haskell is demand driven, we can’t guarantee that
something will ever be evaluated period . Instead we have to

CHAPTER 27. NONSTRICTNESS 1641
create links between nodes in the graph of expressions where
forcing one expression will force yet another expression. Let’s
look at another example:
Prelude> :{
*Main| let wc x z =
*Main| let y =
*Main| undefined `seq` 'y' in x
*Main| :}
Prelude> foldr wc 'z' ['a'..'e']
'a'
Prelude> foldr (flip wc) 'z' ['a'..'e']
'z'
We never evaluated 𝑦, so we never forced the bottom. How-
ever, we can lash yet another data dependency from 𝑦to𝑥:
Prelude> let bot = undefined
Prelude> :{
*Main| let wc x z =
*Main| let y =
*Main| bot `seq` 'y'
*Main| in y `seq` x
*Main| :}
Prelude> foldr wc 'z' ['a'..'e']
*** Exception: Prelude.undefined
Prelude> foldr (flip wc) 'z' ['a'..'e']

CHAPTER 27. NONSTRICTNESS 1642
*** Exception: Prelude.undefined
Previously the evaluation dependency was between the
bottom value and 𝑦:
undefined `seq` y
-- forcing y necessarily forces undefined
y -> undefined
Changing the expression as we did caused the following to
happen:
undefined `seq` y `seq` x
-- forcing x necessarily forces y
-- forcing y necessarily forces undefined
x -> y -> undefined
We think of this as a chain reaction.
All we can do is chuck a life raft from one value to another
as a means of saying, “if you want to get him, you gotta get
through me!” We can even set our life-raft buddies adrift!
Check it out:

CHAPTER 27. NONSTRICTNESS 1643
notGonnaHappenBru ::Int
notGonnaHappenBru =
letx=undefined
y=2
z=(x `seq` y `seq` 10,11)
insnd z
The above will not bottom out! Our life-raft buddies are
bobbing in the ocean blue, with no tugboat evaluator to pull
them in.
seq and weak head normal form
Whatseqdoes is evaluate your expression up to weak head nor-
mal form. We’ve discussed it before, but if you’d like a deeper
investigation and contrast of weak head normal form and nor-
mal form, we strongly recommend Simon Marlow’s Parallel
and Concurrent Programming in Haskell3. WHNF evaluation
means it stops at the first data constructor or lambda. Let’s
test that hypothesis!
Prelude> let dc = (,) undefined undefined
Prelude> let noDc = undefined
Prelude> let lam = \_ -> undefined
Prelude> dc `seq` 1
1
3http://chimera.labs.oreilly.com/books/1230000000929

CHAPTER 27. NONSTRICTNESS 1644
Prelude> noDc `seq` 1
*** Exception: Prelude.undefined
Prelude> lam `seq` 1
1
Right-o. No surprises, right? Right? Okay.
Sincedchas a data constructor, seqdoesn’t need to care
about the values inside that constructor; weak head normal
form evaluation only requires it to evaluate the constructor.
On the other hand, noDchas no data constructor or lambda
outside the value, so there’s no head for the evaluation to stop
at. Finally, lamhas a lambda outside the expression which has
the same eﬀect on evaluation as a data constructor does.
Case matching also chains evaluation
This forcing behavior happens already without seq! For ex-
ample, when you case or pattern match on something, you’re
forcing the value you pattern matched on because it doesn’t
know which data constructor is relevant until it is evaluated
to the depth required to yield the depth of data constructors
you pattern matched. Let’s look at an example:

CHAPTER 27. NONSTRICTNESS 1645
dataTest=
ATest2
|BTest2
deriving (Show)
dataTest2=
CInt
|DInt
deriving (Show)
forceNothing ::Test->Int
forceNothing _ =0
forceTest ::Test->Int
forceTest (A_)=1
forceTest (B_)=2
forceTest2 ::Test->Int
forceTest2 (A(Ci))=i
forceTest2 (B(Ci))=i
forceTest2 (A(Di))=i
forceTest2 (B(Di))=i
We’ll test forceNothing first:
Prelude> forceNothing undefined

CHAPTER 27. NONSTRICTNESS 1646
0
Prelude> forceNothing (A undefined)
0
It’ll never bottom out because it never forces anything. It’s
just a constant value that drops its argument on the floor. What
aboutforceTest ?
Prelude> forceTest (A undefined)
1
Prelude> forceTest (B undefined)
2
Prelude> forceTest undefined
*** Exception: Prelude.undefined
We only get a bottom when the outermost Testvalue is
bottom because that’s the only value whose data constructors
we’re casing on. And then with forceTest2 :
Prelude> forceTest2 (A (C 0))
0
Prelude> forceTest2 (A (C undefined))
*** Exception: Prelude.undefined
Prelude> forceTest2 (A undefined)
*** Exception: Prelude.undefined

CHAPTER 27. NONSTRICTNESS 1647
Prelude> forceTest2 undefined
*** Exception: Prelude.undefined
There we go: outside -> in.
Core Dump
Not the usual core dump you might be thinking of. In this
case, we’re talking about the underlying language that GHC
Haskell gets simplified to after the compiler has desugared our
code.
Our first means of determining strictness was by injecting
bottoms into our expressions and observing the evaluation.
Injecting bottoms everywhere allows us to see clearly what’s
being evaluated strictly and what’s not. Our second means of
determining strictness in Haskell is examining GHC Core4.
Here’s the example we’ll be working with:
moduleCoreDump where
discriminatory ::Bool->Int
discriminatory b=
casebof
False->0
True->1
4https://ghc.haskell.org/trac/ghc/wiki/Commentary/Compiler/CoreSynType

CHAPTER 27. NONSTRICTNESS 1648
Load this up in GHCi in the following manner:
Prelude> :set -ddump-simpl
Prelude> :l code/coreDump.hs
[1 of 1] Compiling CoreDump
================ Tidy Core ==============
... some noise...
You should then get the following GHC Core output:
discriminatory ::Bool->Int
[GblId,Arity=1,
Caf=NoCafRefs ,
Str=DmdType]
discriminatory =
\(b_aZJ::Bool)->
caseb_aZJof _[Occ=Dead] {
False->GHC.Types.I#0;
True->GHC.Types.I#1
}
We’re not going to dissemble: GHC Core is a bit ugly. How-
ever, there are some means of cleaning it up. One is to use the
-dsuppress-all flag:
Prelude> :set -dsuppress-all

CHAPTER 27. NONSTRICTNESS 1649
Prelude> :r
Note that you may need to poke the file to force it to reload.
This then outputs:
discriminatory
discriminatory =
\b_aZY->
caseb_aZYof _{
False->I#0;
True->I#1
}
A titch more readable. The idea here is that the simpler
Core language gives us a clearer idea of when precisely some-
thing will be evaluated. For the sake of simplicity, we’ll revisit
a previous example:
forceNothing _ =0
In Core, it looks like this:
forceNothing =\_ ->I#0#
We’re looking for case expressions in GHC Core to find out
where the strictness is in our code, because case expressions
must be evaluated. There aren’t any cases here, so it forces

CHAPTER 27. NONSTRICTNESS 1650
strictly5nothing! The I# O#is the underlying representation
of anIntliteral which is exposed in GHC Core. On with the
show!
Let’s see what the Core for forceTest looks like:
forceTest =
\ds_d2oX ->
caseds_d2oX of _{
Ads1_d2pI ->I#1#;
Bds1_d2pJ ->I#2#
}
From the GHC Core for this we can see that we force one
value, the outermost data constructors of the Testtype. The
contents of those data constructors are given a name but never
used and so are never evaluated.
5HAAAAAAAAAAAA

CHAPTER 27. NONSTRICTNESS 1651
forceTest2 =
\ds_d2n2 ->
caseds_d2n2 of _{
Ads1_d2oV ->
caseds1_d2oV of _{
Ci_a1lo->i_a1lo;
Di_a1lq->i_a1lq
};
Bds1_d2oW ->
caseds1_d2oW of _{
Ci_a1lp->i_a1lp;
Di_a1lr->i_a1lr
}
}
WithforceTest2 the outsideness and insideness shows more
clearly. In the outer part of the function, we do the same as
forceTest , but the diﬀerence is that we end up also forcing the
contents of the outer Testdata constructors. The function has
four possible results that aren’t bottom and if it isn’t passed bot-
tom it’ll always force twice — once for Testand once for Test2.
It returns but does not itself force or evaluate the contents of
theTest2data constructor.
In Core, a case expression always evaluates what it cases
on — even if no pattern matching is performed — whereas in

CHAPTER 27. NONSTRICTNESS 1652
Haskell proper, values are forced when matching on data con-
structors. We recommend reading the GHC documentation
on the Core language in the footnote above if you’d like to
leverage Core to understand your Haskell code’s performance
or behavior more deeply.
Now let us use this to analyze something:
discriminatory ::Bool->Int
discriminatory b=
letx=undefined
in case bof
False->0
True->1
What does the Core for this look like?
discriminatory
discriminatory =
\b_a10c->
caseb_a10cof _{
False->I#0;
True->I#1
}
GHC is too clever for our shenanigans! It knows we’ll never
evaluate 𝑥, so it drops it. What if we force it to evaluate 𝑥before
we evaluate 𝑏?

CHAPTER 27. NONSTRICTNESS 1653
discriminatory ::Bool->Int
discriminatory b=
letx=undefined
in case x `seq` b of
False->0
True->1
Then the Core:
discriminatory =
\b_a10D->
let{
x_a10E
x_a10E=undefined } in
case
casex_a10Eof _{
__DEFAULT ->b_a10D
}of _{
False->I#0;
True->I#1
}
What’s happened here is that there are now two case ex-
pressions, one nested in another. The nesting is to make the
evaluation of 𝑥obligatory before evaluating 𝑏. This is how seq
changes your code.

CHAPTER 27. NONSTRICTNESS 1654
A Core diﬀerence In Haskell, case matching is strict — or, at
least, the pattern matching of it is — up to WHNF. In Core,
cases are always strict6to WHNF. This doesn’t seem to be a
distinction that matters, but there are times when the distinc-
tion becomes relevant. In Haskell, this will not bottom out:
caseundefined of{_ ->False}
When that gets transliterated into Core, it recognizes that
we didn’t actually use the case match for anything and drops
the case expression entirely, simplifying it to just the data
constructor False.
However, this Core expression is syntactically similar to the
Haskell above, but it will bottom out:
caseundefined of{DEFAULT ->False}
Case in Core is strict even if there’s one case and it doesn’t
match on anything. Core and Haskell are not the same lan-
guage, but anytime you need to know if two expressions in
Haskell are the same, one way to know for sure is by examining
the Core.
6https://ghc.haskell.org/trac/ghc/wiki/Commentary/Compiler/CoreSynType#
Caseexpressions

CHAPTER 27. NONSTRICTNESS 1655
A little bit stricter now
Okay, we had a nice little digression there into wonderland!
Let’s get back to the point which is…we still haven’t quite man-
aged to accomplish what a strict language would have done
with our hypofunction, because we did partially evaluate the
expression. We evaluated the 𝑠which forced the 𝑥which is
what finally gave us the exception. A strict language would not
even have evaluated 𝑠, because evaluating 𝑠would depend on
the𝑥inside already being evaluated.
What if we want our Haskell program to do as a strict lan-
guage would’ve done?
hypo''::IO()
hypo''= do
letx::Integer
x=undefined
s<-x `seq` getLine
casesof
"hi"->print x
_ ->putStrLn "hello"
Notice we moved the seqto the earliest possible point in
ourIOaction. This one’ll just pop without so much as a by-
your-leave:
Prelude> hypo''

CHAPTER 27. NONSTRICTNESS 1656
*** Exception: Prelude.undefined
The reason is that we’re forcing evaluation of the bottom
before we evaluate getLine , which would have performed the
eﬀect of awaiting user input. While this reproduces the ob-
servable results of what a strict language might have done,
it isn’t truly the same thing because we’re not firing oﬀ the
error upon the construction of the bottom. It’s not possible for
an expression to be evaluated until the path evaluation takes
through your program has reached that expression. In Haskell,
the tree doesn’t fall in the woods until you walk through the
forest and get to the tree. For that matter, the tree didn’t exist
until you walked up to it.
Exercises: Evaluate
Expand the expression in as much detail as possible. Then,
work outside-in to see what the expression evaluates to.
1.const1undefined
2.constundefined 1
3.flipconst undefined 1
4.flipconst1undefined
5.constundefined undefined

CHAPTER 27. NONSTRICTNESS 1657
6.foldrconst'z'['a'..'e']
7.foldr(flip const) 'z'['a'..'e']
27.6 Call by name, call by need
Another way we can talk about diﬀerent evaluation strategies
is by distinguishing them on the basis of call by name, call by
need, and call by value.
1.Call by value: Argument expressions have been evaluated
before entering a function. The expressions that bindings
reference are evaluated before creating the binding. This
is conventionally called strict. This is inside-out evalua-
tion.
2.Call by name: Expressions can be arguments to a function
without having been evaluated, or in some cases, never
being evaluated. You can create bindings to expressions
without evaluating them first. Nonstrictness includes this
evaluation strategy. This is outside-in.
3.Call by need: This is the same as call by name, but expres-
sions are only evaluated once. This only happens some
of the time in GHC Haskell, usually when an expression
isn’t a lambda that takes arguments and also has a name.
Results are typically shared within that name only in GHC

CHAPTER 27. NONSTRICTNESS 1658
Haskell (that is, other implementations of Haskell may
choose to do things diﬀerently). This is also nonstrict and
outside-in.
27.7 Nonstrict evaluation changes what
we can do
We’ll cover normal order evaluation (the nonstrict strategy
Haskell prescribes for its implementations) in more detail later.
Now, we’ll look at examples of what nonstrictness enables. The
following will work in languages with a strict or a nonstrict
evaluation strategy:
Prelude> let myList = [1, 2, 3]
Prelude> tail myList
[2,3]
That works in either strict or nonstrict languages because
there is nothing there that can’t be evaluated. However, if we
keep in mind that undefined as an instance of bottom will throw
an error when forced:
Prelude> undefined
*** Exception: Prelude.undefined
We’ll see a diﬀerence between strict and nonstrict. This will
only work in languages that are nonstrict:

CHAPTER 27. NONSTRICTNESS 1659
Prelude> let myList = [undefined, 2, 3]
Prelude> tail myList
[2,3]
A strict language would have crashed on construction of
myList due to the presence of bottom. This is because strict
languages eagerly evaluate all expressions as soon as they
are constructed. The moment [undefined, 2, 3] was declared,
undefined would’ve been evaluated as an argument to (:)and
raised the exception. In Haskell, however, nonstrict evaluation
means that bottom value won’t be evaluated unless it is needed
for some reason.
Take a look at the next example and, before going on, see
if you can figure out whether it will throw an exception and
why:
Prelude> head $ sort [1, 2, 3, undefined]
When we call headon a list that has been passed to sort, we
only need the lowest value in the list and that’s all the work we
will do. The problem is that in order for sortto know what the
lowest value is, it must evaluate undefined which then throws
the error.

CHAPTER 27. NONSTRICTNESS 1660
27.8 Thunk Life
A thunk is used to reference suspended computations that
might be performed or computed at a later point in your pro-
gram. You can get into considerably more detail7on this topic,
but essentially thunks are computations not yet evaluated up
to weak head normal form. If you read the GHC notes on
this you’ll see references to head normal form — it’s the same
thing as weak head normal form.
Not all values get thunked
We’re going to be using the GHCi command sprint in this
section as one means of showing when something is thunked.
You may remember this from the Lists chapter, but let’s refresh
our memories a bit.
Thesprint command allows us to show what has been eval-
uated already by printing in the REPL. An underscore is used
to represent values that haven’t been evaluated yet. We noted
before that this command can have some quirky behavior, al-
though this chapter will explain some of the things that cause
those seemingly unpredictable behaviors.
Let’s start with a simple example:
Prelude> let myList = [1, 2] :: [Integer]
7https://ghc.haskell.org/trac/ghc/wiki/Commentary/Rts/Storage/HeapObjects

CHAPTER 27. NONSTRICTNESS 1661
Prelude> :sprint myList
myList = [1,2]
Wait a second — what happened here? Why is the list shown
fully evaluated when it’s not been needed by anything? This
is an opportunistic strictness. GHC will not thunk (and thus
delay) data constructors. Data constructors are known to be
constant, which justifies the safety of the optimization. The
data constructors here are cons (:), theInteger s, and the empty
list — all of them are constants.
But aren’t data constructors functions? Data constructors
are like functions when they’re unapplied, and constants once
they are fully applied. Since all the data constructors in the
above example are fully applied already, evaluating to weak
headnormalformmeansevaluatingeverythingbecausethere’s
nothing left to apply.
Now back to the thunkery.
A graph of the values of myList looks like:
myList
|
:
/ \
1 :
/ \

CHAPTER 27. NONSTRICTNESS 1662
2 :
/ \
3 []
Here there aren’t any unevaluated thunks; it’s just the final
values that have been remembered. However, if we make it
more polymorphic:
Prelude> let myList2 = [1, 2, 3]
Prelude> :t myList2
myList2 :: Num t => [t]
Prelude> :sprint myList2
myList2 = _
we’ll see an unevaluated thunk represented by the under-
score at the very top level of the expression. Since the type
is not concrete, there’s an implicit function Num a -> a under-
neath, awaiting application to something that will force it to
evaluate to a concrete type. There’s nothing here triggering
that evaluation, so the whole list remains an unevaluated thunk.
We’ll get into more detail about how typeclass constraints eval-
uate soon.
GHC will also stop opportunistically evaluating as soon as
it hits a computation:
Prelude> let xs = [1, 2, id 1] :: [Integer]
Prelude> :sprint xs

CHAPTER 27. NONSTRICTNESS 1663
myList = [1,2,_]
It’s a trivial computation, but GHCi conveniently leaves it
be. Here’s the thunk graph for the above:
myList
|
:
/ \
1 :
/ \
2 :
/ \
_ []
Now let us consider another case that might be slightly
confusing initially for some:
Prelude> let xs = [1, 2, id 1] :: [Integer]
Prelude> let xs' = xs ++ undefined
Prelude> :sprint xs'
myList' = _
Whoa whoa whoa. What’s going on here? The whole thing
is thunked because it’s not in weak head normal form. Why
isn’t it in weak head normal form already? Because the out-
ermost term isn’t a data constructor like (:). The outermost
term is the function (++):

CHAPTER 27. NONSTRICTNESS 1664
myList' = (++) _ _
The function is outermost, despite the fact that it is super-
ficially an infix operator, because the function is the lambda.
The arguments are passed into the function body to be evalu-
ated.
27.9 Sharing is caring
Sharing here roughly means what we’ve implied above: that
when a computation is named, the results of evaluating that
computation can be shared between all references to that name
without re-evaluating it. We care about sharing because mem-
ory is finite, even today in the land of chickens in every pot
and smartphones in every pocket. The idea here is that non-
strictness is a fine thing, but call-by-name semantics aren’t
always enough to make it sufficiently efficient. What is suffi-
ciently efficient? That depends on context and whether it’s
your dissertation or not.
One of the points of confusion for people when trying to
figure out how GHC Haskell really runs code is that it turns
sharing on and oﬀ (that is, it oscillates between call-by-need
and call-by-name) based on necessity and what it thinks will
produce faster code. Part of the reason it can do this at all
without breaking your code is because the compiler knows
when your code does or does not perform I/O.

CHAPTER 27. NONSTRICTNESS 1665
Using trace to observe sharing
The base library has a module named Debug.Trace that has
functions useful for observing sharing. We’ll mostly use trace
here, but feel free to poke around for whatever else might catch
your fancy. Debug.Trace is a means of cheating the type system
and putting a putStrLn without having IOin the type. This is def-
initely something you want to restrict to experimentation and
education; do not use it as a logging mechanism in production
code — it won’t do what you think. However, it does give us a
convenient means of observing when things evaluate.
Let us demonstrate how we can use this to see when things
get evaluated:
Prelude> import Debug.Trace
Prelude> let a = trace "a" 1
Prelude> let b = trace "b" 2
Prelude> a + b
b
a
3
This isn’t an example of sharing, but it demonstrates how
tracecan be used to observe evaluation. We can see that 𝑏
got printed first because that was the first argument that the
addition function evaluated, but you cannot and should not
rely on the evaluation order of the arguments to addition.

CHAPTER 27. NONSTRICTNESS 1666
Here we’re talking about the order in which the arguments to
a single application of addition are forced, not associativity.
You can count on addition being left associative, but within
each pairing, which in the pair of arguments gets forced is not
guaranteed.
Let’s look at a longer example and see how it shows us where
the evaluations occur:
importDebug.Trace (trace)
inc=(+1)
twice=inc.inc
howManyTimes =
inc (trace "I got eval'd" (1+1))
+twice
(trace"I got eval'd" (1+1))
howManyTimes' =
letonePlusOne =
trace"I got eval'd" (1+1)
ininc onePlusOne +twice onePlusOne
Prelude> howManyTimes
I got eval'd

CHAPTER 27. NONSTRICTNESS 1667
I got eval'd
7
Prelude> howManyTimes'
I got eval'd
7
Cool, with that in mind, let’s talk about ways to promote
and prevent sharing.
What promotes sharing
Kindness. Also, names. Names turn out to be a pretty good
way to make GHC share something, if it could’ve otherwise
been shared. First, let’s consider the example of something
that won’t get shared:
Prelude> import Debug.Trace
Prelude> let x = trace "x" (1 :: Int)
Prelude> let y = trace "y" (1 :: Int)
Prelude> x + y
x
y
2
This seems intuitive and reasonable, but the values of 𝑥
and𝑦cannot be shared because they have diﬀerent names.

CHAPTER 27. NONSTRICTNESS 1668
So, even though they have the same value, they have to be
evaluated separately.
GHC does use this intuition that you’ll expect results to be
shared when they have the same name to make performance
more predictable. If we add two values that have the same
name, it will get evaluated once and only once:
Prelude> import Debug.Trace
Prelude> let a = trace "a" (1 :: Int)
Prelude> a + a
a
2
Prelude> a + a
2
Indirection won’t change this either:
Prelude> let x = trace "x" (1 :: Int)
Prelude> (id x) + (id x)
x
2
Prelude> (id x) + (id x)
2
GHC knows what’s up, despite the addition of identity func-
tions. Notice the second time we ran it, it didn’t evaluate 𝑥at

CHAPTER 27. NONSTRICTNESS 1669
all. The value of 𝑥is now held there in memory so whenever
your program calls 𝑥, it already knows the value.
In general, GHC relies on an intuition around names and
sharing to make performance more predictable. However,
this won’t always behave in ways you expect. Consider the
case of a list with a single character…and a String with a sin-
gle character. They’re actually the same thing, but the way
they get constructed is not. This produces diﬀerences in the
opportunistic strictness GHC will engage in.
Prelude> let a = Just ['a']
Prelude> :sprint a
a = Just "a"
Prelude> let a = Just "a"
Prelude> :sprint a
a = Just _
So uh, what gives? Well, the deal is that the strictness analy-
sis driven optimization GHC performs here is limited to data
constructors only, no computation! But where’s the function
you ask? Well if we turn on our night vision goggles…
Prelude> let a = Just ['a']
returnIO
(: ((Just (: (C# 'a') ([])))

CHAPTER 27. NONSTRICTNESS 1670
`cast` ...) ([]))
Prelude> let a = Just "a"
returnIO
(: ((Just (unpackCString# "a"#))
`cast` ...) ([]))
The issue is that a call to a primitive function in GHC.Base
interposes between Justand aCString literal. The reason string
literals aren’t actually lists of characters at time of construction
is mostly to present optimization opportunities, such as when
we convert string literals into ByteString orTextvalues. More
on that in the next chapter!
What subverts or prevents sharing
Sometimes we don’t want sharing. Sometimes we want to
know why sharing didn’t happen when we did want it. Un-
derstanding what kinds of things prevent sharing is therefore
useful.
Inlining expressions where they get used prevents sharing
because it creates independent thunks that will get computed
separately. In this example, instead of declaring the value of 𝑓
to equal 1, we make it a function:
Prelude> :{

CHAPTER 27. NONSTRICTNESS 1671
Prelude| let f :: a -> Int
Prelude| f _ = trace "f" 1
Prelude| :}
Prelude> f 'a'
f
1
Prelude> f 'a'
f
1
In the next examples you can directly compare the dif-
ference between assigning a name to the value of (2 + 2) versus
inlining it directly. When it’s named, it gets shared and not
re-evaluated:
Prelude> let a :: Int; a = trace "a" 2 + 2
Prelude> let b = (a + a)
Prelude> b
a
8
Prelude> b
8
Here we saw 𝑎once, which makes sense as we expect the
result to get shared.
Prelude> :{

CHAPTER 27. NONSTRICTNESS 1672
Prelude| let c :: Int;
Prelude| c = (trace "a" 2 + 2)
Prelude| + (trace "a" 2 + 2)
Prelude| :}
Prelude> c
a
a
8
Prelude> c
8
Here an expression equivalent to 𝑎didn’t get shared be-
cause the two occurrences of the expression weren’t bound
to the same name. This is a trivial example of inlining. This
illustrates the diﬀerence in how things evaluate when an ex-
pression is bound to a name versus when it gets repeated via
inlining in an expression.
Being a function with explicit, named arguments also pre-
vents sharing. Haskell is not fully lazy; it is merely nonstrict,
so it is not required to remember the result of every func-
tion application for a given set of arguments, nor would it be
desirable given memory constraints. A demonstration:
Prelude> :{
Prelude| let f :: a -> Int
Prelude| f = trace "f" const 1

CHAPTER 27. NONSTRICTNESS 1673
Prelude| :}
Prelude> f 'a'
f
1
Prelude> f 'a'
1
Prelude> f 'b'
1
The explicit, named arguments part here is critical! Eta re-
duction (i.e., writing pointfree code, thus dropping the named
arguments) will change the sharing properties of your code.
This will be explained in more detail in the next chapter.
Typeclass constraints also prevent sharing. If we forget to
add a concrete type to an earlier example, we evaluate 𝑎twice:
Prelude> let blah = Just 1
Prelude> fmap ((+1) :: Int -> Int) blah
Just 2
Prelude> :sprint blah
blah = _
Prelude> :t blah
blah :: Num a => Maybe a
Prelude> let bl = Just 1
Prelude> :t bl

CHAPTER 27. NONSTRICTNESS 1674
bl :: Num a => Maybe a
Prelude> :sprint bl
bl = _
Prelude> fmap (+1) bl
Just 2
Prelude> let fm = fmap (+1) bl
Prelude> :t fm
fm :: Num b => Maybe b
Prelude> :sprint fm
fm = _
Prelude> fm
Just 2
Prelude> :sprint fm
fm = _
Prelude> :{
Prelude| let fm' =
Prelude| fmap ((+1) :: Int -> Int) bla
Prelude| :}
Prelude> fm'
Just eval'd 1
2
Prelude> :sprint fm'
fm' = Just 2

CHAPTER 27. NONSTRICTNESS 1675
Again, that’s because typeclass constraints are a function
in Core. They are awaiting application to something that will
make them become concrete types. We’re going to go into a
bit more detail on this in the next section.
Implicit parameters are implemented similarly to type-
class constraints and have the same eﬀect on sharing. Sharing
doesn’t work in the presence of constraints (typeclasses or im-
plicit parameters) because typeclass constraints and implicit
parameters decay into function arguments when the compiler
simplifies the code:
Prelude> :set -XImplicitParams
Prelude> import Debug.Trace
Prelude> :{
Prelude| let add :: (?x :: Int) => Int
Prelude| add = trace "add" 1 + ?x
Prelude| :}
Prelude> let ?x = 1 in add
add
2
Prelude> let ?x = 1 in add
add
2
We won’t talk about implicit parameters too much more as
wedon’tthinkthey’reagoodideaforgeneraluse. Inmostcases

CHAPTER 27. NONSTRICTNESS 1676
where you believe you want implicit parameters, more likely
you want Reader ,ReaderT , or a plain old function argument.
Why polymorphic values never seem to get
forced
As we’ve said, GHC engages in opportunistic strictness when it
can do so safely without making an otherwise valid expression
result in bottom. This is one of the things that confounds the
use ofsprint to observe evaluation in GHCi — GHC will often
be opportunistically strict with data constructors if it knows
the contents definitely can’t be a bottom, such as when they’re
a literal value. It gets more complicated when we consider
that, under the hood, typeclass constraints are simplified into
additional arguments.
Reusing a similar example from earlier we will first observe
this in action, then we’ll talk about why it happens:
Prelude> :{
Prelude| let blah =
Prelude| Just (trace "eval'd 1" 1)
Prelude| :}
Prelude> :sprint blah
blah = _
Prelude> :t blah
blah :: Num a => Maybe a

CHAPTER 27. NONSTRICTNESS 1677
Prelude> fmap (+1) blah
Just eval'd 1
2
Prelude> fmap (+1) blah
Just eval'd 1
2
Prelude> :sprint blah
blah = _
So we have at least some evidence that we’re re-evaluating.
Does it change when it’s concrete?
Prelude> :{
Prelude| let blah =
Prelude| Just (trace "eval'd 1"
Prelude| (1 :: Int))
Prelude| :}
Prelude> :sprint blah
blah = Just _
TheIntvalue being obscured by traceprevented oppor-
tunistic evaluation there. However, eliding the Num a => a in
favor of a concrete type does bring sharing back:
Prelude> fmap (+1) blah
Just eval'd 1
2

CHAPTER 27. NONSTRICTNESS 1678
Prelude> fmap (+1) blah
Just 2
Now our trace gets emitted only once. The idea here is that
after the typeclass constraints get simplified to the underlying
GHC Core language, they’re really function arguments.
It doesn’t matter if you use a function that accepts a concrete
type and forces the Num a => a , it’ll re-do the work on each
evaluation because of the typeclass constraint. For example:
Prelude> fmap ((+1) :: Int -> Int) blah
Just 2
Prelude> :sprint blah
blah = _
Prelude> :t blah
blah :: Num a => Maybe a
Prelude> let bl = Just 1
Prelude> :t bl
bl :: Num a => Maybe a
Prelude> :sprint bl
bl = _
Prelude> fmap (+1) bl
Just 2
Prelude> let fm = fmap (+1) bl
Prelude> :t fm
fm :: Num b => Maybe b

CHAPTER 27. NONSTRICTNESS 1679
Prelude> :sprint fm
fm = _
Prelude> fm
Just 2
Prelude> :sprint fm
fm = _
Prelude> :{
Prelude| let fm' =
Prelude| fmap ((+1) :: Int -> Int)
Prelude| blah
Prelude| :}
Prelude> fm'
Just eval'd 1
2
Prelude> :sprint fm'
fm' = Just 2
So, what’s the deal here with the typeclass constraints? It’s as
ifNum a => a were really Num a -> a . In Core, they are. The only
way to apply that function argument is to reach an expression
that provides a concrete type satisfying the constraint. Here’s
a demonstration of the diﬀerence in behavior using values:
Prelude> let poly = 1
Prelude> let conc = poly :: Int
Prelude> :sprint poly

CHAPTER 27. NONSTRICTNESS 1680
poly = _
Prelude> :sprint conc
conc = _
Prelude> poly
1
Prelude> conc
1
Prelude> :sprint poly
poly = _
Prelude> :sprint conc
conc = 1
Num a => a is a function awaiting an argument, while Intis
not. Behold the Core:
moduleBlahwhere
a::Numa=>a
a=1
concrete ::Int
concrete =1
Prelude> :l code/blah.hs
[1 of 1] Compiling Blah

CHAPTER 27. NONSTRICTNESS 1681
================ Tidy Core ==============
Result size of Tidy Core =
{terms: 9, types: 9, coercions: 0}
concrete
concrete = I# 1
a
a =
\ @ a1_aRN $dNum_aRP ->
fromInteger $dNum_aRP (__integer 1)
Do you see how 𝑎has a lambda? In order to know what
instance of the typeclass to deploy at any given time, the type
has to be concrete. As we’ve seen, types can become concrete
through assignment or type defaulting. Whichever way it
becomes concrete, the result is the same: once the concrete
type is known, the typeclass constraint function gets applied
to the typeclass instance for that type. If you don’t declare the
concrete type, it will have to re-evaluate this function every
time, because it can’t know that the type didn’t change some-
where along the way. So, because it remains a function and
unapplied functions are not shareable values, polymorphic
expressions can’t be shared.
Mostly the behavior doesn’t change when it involves values
defined in terms of functions, but if you forget the type con-

CHAPTER 27. NONSTRICTNESS 1682
cretion it’ll stay _and you’ll be confused and upset. Observe:
Prelude> :{
Prelude| let blah :: Int -> Int
Prelude| blah x = x + 1
Prelude| :}
Prelude> let woot = blah 1
Prelude> :sprint blah
blah = _
Prelude> :sprint woot
woot = _
Prelude> woot
2
Prelude> :sprint woot
woot = 2
Values of a concrete, constant type can be shared, once
evaluated. Polymorphic values may be evaluated once but still
not shared because, underneath, they continue to be functions
awaiting application.
Preventing sharing on purpose
When do we want to prevent sharing? When we don’t want
a large datum hanging out in memory that was calculated to
provide a much smaller answer. First an example that demon-
strates sharing:

CHAPTER 27. NONSTRICTNESS 1683
Prelude> import Debug.Trace
Prelude> let f x = x + x
Prelude> f (trace "hi" 2)
hi
4
We see “hi” once because 𝑥got evaluated once. In the next
example, 𝑥gets evaluated twice:
Prelude> let f x = (x ()) + (x ())
Prelude> f (\_ -> trace "hi" 2)
hi
hi
4
Using unit ()as arguments to 𝑥turned𝑥into a very trivial,
weird-looking function, which is why the value of 𝑥can no
longer be shared. It doesn’t matter much since that “function”
𝑥doesn’t really do anything.
OK, that was weird; maybe it’ll be easier to see if we use
some more traditional-seeming argument to 𝑥:
Prelude> let f x = (x 2) + (x 10)
Prelude> f (\x -> trace "hi" (x + 1))
hi
hi
14

CHAPTER 27. NONSTRICTNESS 1684
Using a lambda that mentions the argument in some fash-
ion disables sharing:
Prelude> let g = \_ -> trace "hi" 2
Prelude> f g
hi
hi
4
However, this worked in part because the function passed
to𝑓had the argument as part of the declaration, even though
it used underscore to ignore it. Notice what happens if we
make it pointfree:
Prelude> let g = const (trace "hi" 2)
Prelude> f g
hi
4
We’re going to get into a little more detail about this dis-
tinction in the next chapter, but the idea here is that functions
aren’t shared when there are named arguments but are when
the arguments are elided, as in pointfree. So, one way to pre-
vent sharing is adding named arguments.

CHAPTER 27. NONSTRICTNESS 1685
Forcing sharing
You can force sharing by giving your expression a name. The
most common way of doing this is with let.
-- calculates 1 + 1 twice
(1+1)*(1+1)
-- shares 1 + 1 result under 'x'
letx=1+1
inx*x
With that in mind, if you take a look at the forever function
inControl.Monad , you might see something a little mysterious
looking:
forever ::(Monadm)=>m a->m b
forever a= leta'=a>>a'ina'
Why the letexpression? Well, we want sharing here so that
running a monadic action indefinitely doesn’t leak memory.
The sharing here causes GHC to overwrite the thunk as it runs
each step in the evaluation, which is quite handy. Otherwise,
it would keep constructing new thunks indefinitely and that
would be very unfortunate.

CHAPTER 27. NONSTRICTNESS 1686
27.10 Refutable and irrefutable patterns
When we’re talking about pattern matching, it’s important to
be aware that there are refutable and irrefutable patterns. An
irrefutable pattern is one which will never fail to match. A
refutable pattern is one which has potential failures. Often,
the problem is one of specificity.
refutable ::Bool->Bool
refutable True=False
refutable False=True
irrefutable ::Bool->Bool
irrefutable x=not x
oneOfEach ::Bool->Bool
oneOfEach True=False
oneOfEach _ =True
Remember, the pattern is refutable or not, not the function
itself. The function refutable is refutable because each case is
refutable; each case could be given an input that fails to match.
In contrast, irrefutable has an irrefutable pattern; that is, its
pattern doesn’t rely on matching with a specific value.
In the case of oneOfEach , the first pattern is refutable because
it pattern matches on the Truedata constructor. irrefutable

CHAPTER 27. NONSTRICTNESS 1687
and the second match of oneOfEach are irrefutable because they
don’t need to look inside the data they are applied to.
That said, the second pattern match of oneOfEach being ir-
refutable isn’t terribly semantically meaningful as Haskell will
have to inspect the data to see if it matches the first case any-
way.
Theirrefutable function works for any inhabitant (all two
of them) of Boolbecause it doesn’t specify which Boolvalue in
the pattern to match. You could think of an irrefutable pattern
as one which will never fail to match. If an irrefutable pattern
for a particular value comes before a refutable pattern, the
refutable pattern will never get invoked.
This little function appeared in an earlier chapter, but we’ll
bring it back for a quick demonstration:
isItTwo ::Integer ->Bool
isItTwo 2=True
isItTwo _ =False
In the case of Bool, the order of matching TrueandFalse
specifically doesn’t matter, but in cases like isItTwo where one
case is specific and the other is a catchall otherwise case, the
ordering will certainly matter. You can reorder the expressions
ofisItTwo to see what happens, although it’s probably clear.

CHAPTER 27. NONSTRICTNESS 1688
Lazy patterns
Lazy patterns are also irrefutable.
strictPattern ::(a, b)->String
strictPattern (a,b)=const"Cousin It" a
lazyPattern ::(a, b)->String
lazyPattern ~(a,b)=const"Cousin It" a
The tilde is how one makes a pattern match lazy. A caveat
is that since it makes the pattern irrefutable, you can’t use
it to discriminate cases of a sum — it’s useful for unpacking
products that might not get used.
Prelude> strictPattern undefined
*** Exception: Prelude.undefined
Prelude> lazyPattern undefined
"Cousin It"
And as we see here, in the lazy pattern version since const
didn’t actually need 𝑎from the tuple, we never forced the
bottom. The default behavior is to just go ahead and force
it before evaluating the function body, mostly for more pre-
dictable memory usage and performance.

CHAPTER 27. NONSTRICTNESS 1689
27.11 Bang patterns
Sometimes we want to evaluate an argument to a function
whether we use it or not. We can do this with seqas in the
following example:
{-# LANGUAGE BangPatterns #-}
moduleManualBang where
doesntEval ::Bool->Int
doesntEval b=1
manualSeq ::Bool->Int
manualSeq b=b `seq` 1
Or we can also do it with a bang pattern on 𝑏— note the
exclamation point:
banging ::Bool->Int
banging !b=1
Let’s look at the Core for those three:

CHAPTER 27. NONSTRICTNESS 1690
doesntEval
doesntEval =
\_ ->I#1#
manualSeq
manualSeq =
\b_a1ia->
caseb_a1iaof _
{ __DEFAULT ->I#1#}
banging
banging =
\b_a1ib->
caseb_a1ibof _
{ __DEFAULT ->I#1#}
If you try passing bottom to each function you’ll find that
manualSeq and banging are forcing their argument despite not
using it for anything. Remember that forcing something is
expressed in Core as a case expression and that case evaluates
up to weak head normal form in Core.
Bang patterns in data
When we evaluate the outer data constructor of a datatype,
at times we’d also like to evaluate the contents to weak head

CHAPTER 27. NONSTRICTNESS 1691
normal form just like with functions.
One way to see the diﬀerence between strict and nonstrict
constructor arguments is how they behave when they are un-
defined. Let’s look at an example (note the exclamation mark):
dataFoo=FooInt!Int
first(Foox_)=x
second(Foo_y)=y
Since the nonstrict argument isn’t evaluated by second , pass-
ing in undefined doesn’t cause a problem:
> second (Foo undefined 1)
1
But the strict argument can’t be undefined, even if we don’t
use the value:
> first (Foo 1 undefined)
*** Exception: Prelude.undefined
You could do this manually with seq, but it’s a little tedious.
Here’s another example with two equivalent datatypes, one
of them with strictness annotations on the contents and one
without:

CHAPTER 27. NONSTRICTNESS 1692
{-# LANGUAGE BangPatterns #-}
moduleManualBang where
dataDoesntForce =
TisLazy IntString
gibString ::DoesntForce ->String
gibString (TisLazy _s)=s
-- note the exclamation marks again
dataBangBang =
SheShotMeDown !Int!String
gimmeString ::BangBang ->String
gimmeString (SheShotMeDown _s)=s
Then testing those in GHCi:
Prelude> let x = TisLazy undefined "blah"
Prelude> gibString x
"blah"
Prelude> let s = SheShotMeDown
Prelude> let x = s undefined "blah"
Prelude> gimmeString x
"*** Exception: Prelude.undefined

CHAPTER 27. NONSTRICTNESS 1693
The idea here is that in some cases, it’s cheaper to just com-
pute something than to construct a thunk and then evaluate
it later. This case is particularly common in numerics code
where you have a lot of IntandDouble values running around
which are individually cheap to conjure. If the values are both
cheap to compute and small, then you may as well make them
strict unless you’re trying to dance around bottoms. Types
with underlying primitive representations IntandDouble most
assuredly qualify as small.
A good rule to follow is lazy in the spine, strict in the leaves!
Sometimes a “leak” isn’t really a leak but temporarily exces-
sive memory that subsides because you made 1,000,000 tiny
values into less-tiny thunks when you could’ve just computed
them as your algorithm progressed.
27.12 Strict and StrictData
If you’re using GHC 8.0 or newer, you can avail yourself of the
Strict andStrictData extensions. The key thing to realize is
Strict /StrictData are just letting you avoid putting in pervasive
uses of seqand bang patterns yourself. They don’t add any-
thing to the semantics of the language. Accordingly, it won’t
suddenly make lazy data structures defined elsewhere behave
diﬀerently, although it does make functions defined in that
module processing lazy data structures behave diﬀerently.

CHAPTER 27. NONSTRICTNESS 1694
Let’s play with that (if you have GHC 8.0 or newer; if not,
this code won’t work):
{-# LANGUAGE Strict #-}
moduleStrictTest where
blahx=1
main=print (blah undefined)
The above will bottom out because blahis defined under
the module with the Strict extension and will get translated
into the following:
blahx=x `seq` 1
-- or with bang patterns
blah!x=1
So, the Strict andStrictData extensions are a means of
avoiding noise when everything or almost everything in a
module is supposed to be strict. You can use the tilde for ir-
refutable patterns to recover laziness on a case by case basis:

CHAPTER 27. NONSTRICTNESS 1695
{-# LANGUAGE Strict #-}
moduleLazyInHostileTerritory where
willForce x=1
willNotForce ~x=1
Admittedlytheseareglorifiedrenamesof const, butitdoesn’t
matterforthepurposesofdemonstratingwhathappens. Here’s
what we’ll see in GHCi when we pass them bottom:
Prelude> willForce undefined
*** Exception: Prelude.undefined
Prelude> willNotForce undefined
1
So even when you’re using the Strict extension, you can
selectively recover laziness when desired.
27.13 Adding strictness
Now we shall examine how applying strictness to a datatype
and operations we’re already familiar with can change how
they behave in the presence of bottom through the list type.
This is intended to be mostly demonstrative rather than a
practical example.

CHAPTER 27. NONSTRICTNESS 1696
moduleStrictTest1 where
dataLista=
Nil
|Consa (Lista)deriving Show
sTake::Int->Lista->Lista
sTaken_
|n<=0=Nil
sTakenNil=Nil
sTaken (Consx xs)=
(Consx (sTake (n -1) xs))
twoEls =Cons1(Consundefined Nil)
oneEl =sTake1twoEls
The name of the module here is a bit of a misnomer. List
here is lazy, just like the built-in [a]in the Haskell prelude.
Ourtakederivative named sTakeis lazy too.
Now let’s load up this code in our REPL and test it out:
Prelude> twoEls
Cons 1 (Cons
*** Exception: Prelude.undefined
Prelude> oneEl

CHAPTER 27. NONSTRICTNESS 1697
Cons 1 Nil
Now let’s experiment with adding strictness to diﬀerent
parts of our program and observe what changes in our code’s
behavior.
First we’re going to add BangPatterns so that we have a syn-
tactically convenient way to denote when and where we want
strictness:
moduleStrictTest2 where
dataLista=
Nil
|Cons!a (Lista)deriving Show
sTake::Int->Lista->Lista
sTaken_
|n<=0=Nil
sTakenNil=Nil
sTaken (Consx xs)=
(Consx (sTake (n -1) xs))
twoEls=Cons1(Consundefined Nil)
oneEl=sTake1twoEls

CHAPTER 27. NONSTRICTNESS 1698
Noting the placement of the exclamation marks denoting
strictness, let’s run it in GHCi and see if it does what we want:
Prelude> twoEls
Cons 1 *** Exception: Prelude.undefined
Prelude> oneEl
Cons 1 Nil

CHAPTER 27. NONSTRICTNESS 1699
{-# LANGUAGE BangPatterns #-}
moduleStrictTest3 where
dataLista=
Nil
|Cons!a (Lista)deriving Show
sTake::Int->Lista->Lista
sTaken_
|n<=0=Nil
sTakenNil=Nil
sTaken (Consx!xs)=
(Consx (sTake (n -1) xs))
twoEls=Cons1(Consundefined Nil)
oneEl=sTake1twoEls
threeElements =Cons2twoEls
oneElT =sTake1threeElements
We added strictness to the 𝑥𝑠so thatsTakeis going to force
more of the list. Let’s see what happens:
Prelude> twoEls
Cons 1 *** Exception: Prelude.undefined

CHAPTER 27. NONSTRICTNESS 1700
Prelude> oneEl
*** Exception: Prelude.undefined
Prelude> threeElements
Cons 2 (Cons 1
*** Exception: Prelude.undefined
Prelude> oneElT
Cons 2 Nil
Let’s add more strictness:

CHAPTER 27. NONSTRICTNESS 1701
moduleStrictTest4 where
dataLista=
Nil
|Cons!a!(Lista)deriving Show
sTake::Int->Lista->Lista
sTaken_
|n<=0=Nil
sTakenNil=Nil
sTaken (Consx xs)=
(Consx (sTake (n -1) xs))
twoEls=Cons1(Consundefined Nil)
oneEl=sTake1twoEls
And run it again:
Prelude> twoEls
*** Exception: Prelude.undefined
Prelude> oneEl
*** Exception: Prelude.undefined
So, what’s the upshot of our experiments with adding strict-
ness here?

CHAPTER 27. NONSTRICTNESS 1702
NCons sTake
1Cons a (List a) Cons x xs
2Cons !a (List a) Cons x xs
3Cons !a (List a) Cons x !xs
4Cons !a !(List a) Cons x xs
Then the results themselves:
NtwoEls oneEl
1Cons 1 (Cons *** Cons 1 Nil
2Cons 1 *** Cons 1 Nil
3Cons 1 *** ***
4*** ***
You can see clearly what adding strictness in diﬀerent places
does to our evaluation in terms of bottom.
27.14 Chapter Exercises
Strict List
Try messing around with the following list type and compare
what it does with the bang-patterned list variants we experi-
mented with earlier:

CHAPTER 27. NONSTRICTNESS 1703
{-# LANGUAGE Strict #-}
moduleStrictList where
dataLista=
Nil|
Consa (Lista)
deriving (Show)
take'n_ | n<=0=Nil
take'_Nil =Nil
take'n (Consx xs) =
(Consx (take' (n -1) xs))
map'_Nil =Nil
map'f (Consx xs)=
(Cons(f x) (map' f xs))
repeat' x=xswherexs=(Consx xs)
main= do
print$take'10$map' (+1) (repeat' 1)

CHAPTER 27. NONSTRICTNESS 1704
What will :sprint output?
We show you a definition or multiple definitions, you deter-
mine what :sprint will output when passed the bindings listed
in your head before testing it.
1.letx=1
2.letx=['1']
3.letx=[1]
4.letx=1::Int
5.letf=\x->x
letx=f1
6.letf::Int->Int; f=\x->x
letx=f1
Will printing this expression result in bottom?
1.snd(undefined, 1)
2.letx=undefined
lety=x `seq` 1insnd (x, y)
3.length$[1..5]++undefined

CHAPTER 27. NONSTRICTNESS 1705
4.length$[1..5]++[undefined]
5.const1undefined
6.const1(undefined `seq` 1)
7.constundefined 1
Make the expression bottom
Using only bang patterns or seq, make the code bottom out
when executed.
1.x=undefined
y="blah"
main= do
print (snd (x, y))
27.15 Follow-up resources
1.The Incomplete Guide to Lazy Evaluation (in Haskell);
Heinrich Apfelmus
https://hackhands.com/guide-lazy-evaluation-haskell/
2.Chapter 2. Basic Parallelism: The Eval Monad; Parallel
and Concurrent Programming in Haskell; Simon Marlow;
http://chimera.labs.oreilly.com/books/1230000000929/ch02.html

CHAPTER 27. NONSTRICTNESS 1706
3.Lazy evaluation illustrated for Haskell divers; Takenobu
Tani
4.A Natural Semantics for Lazy Evaluation; John Launch-
bury
5.AnOperationalSemanticsforParallelCall-by-Need; Clem
Baker-Finch, David King, Jon Hall and Phil Trinder

Chapter 28
Basic libraries
Bad programmers worry
about the code. Good
programmers worry
about data structures and
their relationships.
Linus Torvalds
1707

CHAPTER 28. BASIC LIBRARIES 1708
28.1 Basic libraries and data structures
Data structures are kind of important. Insofar as computers
are fast, they aren’t getting much faster — at least, the CPU
isn’t. This is usually a lead-in for a parallelism/concurrency
sales pitch. But this isn’t that book.
The data structures you choose to represent your problem
aﬀect the speed and memory involved in processing your data,
perhaps to a larger extent than is immediately obvious. At the
level of your program, making the right decision about how
to represent your data is the first important step to writing
efficient programs. In fact, your choice of data structure can
aﬀect whether it’s worthwhile or even makes sense to attempt
to parallelize something.
This chapter is here to help you make the decision of the
optimal data structures for your programs. We can’t prescribe
one or the other of similar data structures because how ef-
fective they are will depend a lot on what you’re trying to
do. So, our first step will be to give you tools to measure for
yourself how diﬀerent structures will perform in your context.
We’ll also cover some of the mistakes that can cause your
memory usage and execution time to explode.
This chapter will
•demonstrate how to measure the usage of time and space
in your programs;

CHAPTER 28. BASIC LIBRARIES 1709
•oﬀer guidelines on when weak head normal form or nor-
mal form are appropriate when benchmarking code;
•define constant applicative forms and explain argument
saturation;
•demonstrate and critically evaluate when to use diﬀerent
data structures in diﬀerent circumstances;
•sacrifice some jargon for the jargon gods.
We’re going to kick this chapter oﬀ with some benchmark-
ing.
28.2 Benchmarking with Criterion
It’s a common enough thing to want to know how fast our
code is. If you can’t benchmark properly, then you can’t know
if you used six microseconds or only five, and can only ask
yourself, “Well, do I feel lucky?”
Well, do ya, punk?
If you’d rather not trust your performance to guesswork,
the best way to measure performance is to sample many times
in order to establish a confidence interval. Fortunately, that
work has already been done for us in the wonderful library
criterion1by Bryan O’Sullivan.
1http://hackage.haskell.org/package/criterion

CHAPTER 28. BASIC LIBRARIES 1710
As it happens, criterion comes with a pretty nice tutorial2,
but we’ll still work through an example so you can follow along
with this chapter. In our toy program here, we’re looking to
write a total version of (!!)which returns Maybeto make the
bottoms unnecessary. When you compile code for bench-
marking, make sure you’re using -Oor-O2in the build flags to
GHC. Those can be specified by running GHC manually:
-- with stack
$ stack ghc -- -O2 bench.hs
-- without stack
$ ghc -O2 bench.hs
Or via the Cabal setting ghc-options3.
Let’s get our module set up:
2http://www.serpentine.com/criterion/tutorial.html
3https://www.haskell.org/cabal/users-guide/

CHAPTER 28. BASIC LIBRARIES 1711
moduleMainwhere
importCriterion.Main
infixl9!?
_ !? n|n<0=Nothing
[]!? _ = Nothing
(x:_)!?0 =Justx
(_:xs)!?n =xs!?(n-1)
myList::[Int]
myList=[1..9999]
main::IO()
main=defaultMain
[ bench "index list 9999"
$whnf (myList !!)9998
, bench "index list maybe index 9999"
$whnf (myList !?)9998
]
Our version of (!!)shouldn’t have anything too surprising
going on. We have declared that it’s a left-associating infix op-
erator (infixl ) with a precedence of 9. We haven’t talked much
about the associativity or fixity of operators since Chapter 2.

CHAPTER 28. BASIC LIBRARIES 1712
This is the same associativity and precedence as the normal
(!!)operator in base.
Criterion.Main is the convenience module to import from
criterion if you’re running benchmarks in a Mainmodule. Usu-
ally you’ll have a benchmark stanza in your Cabal file that be-
haves like an executable. It’s also possible to do it as a one-oﬀ
using Stack:
$ stack build criterion
$ stack ghc -- -O2 benchIndex.hs
$ ./benchIndex
Heremainuses a function from criterion calledwhnf. The
functions whnfandnf(also in criterion ), as you might guess,
refer to weak head normal form and normal form, respectively.
Weak head normal form, as we said before, evaluates to the
first data constructor. That means that if your outermost data
constructor is a Maybe, it’s only going to evaluate enough to find
out if it’s a Nothing or aJust— if there is a Just a , it won’t count
the cost of evaluating the 𝑎value.
Usingnfwould mean you wanted to include the cost of
fully evaluating the 𝑎as well as the first data constructor. The
key when determining whether you want whnfornfis to think
about what you’re trying to benchmark and if reaching the first
data constructor will do all the work you’re trying to measure
or not. We’ll talk more about what the diﬀerence is here and
how to decide which you need in a bit.

CHAPTER 28. BASIC LIBRARIES 1713
In our case, what we want is to compare two things: the
weak head normal form evaluation of the original indexing
operator and that of our safe version, applied to the same long
list. We only need weak head normal form because (!!)and
(!?)don’t return a data constructor until they’ve done the
work already, as we can see by taking a look at the first three
cases:
_ !? n|n<0=Nothing
[]!? _ = Nothing
(x:_)!?0 =Justx
These first three cases aren’t reached until you’ve gone
through the list as far as you’re going to go. The recursive case
below doesn’t return a data constructor. Instead, it invokes
itself repeatedly until one of the above cases is reached. Eval-
uating to WHNF cannot and does not pause in a self-invoked
recursive case like this:
(_:xs)!?n =xs!?(n-1)
-- Self function call,
-- not yet in weak head.
When evaluated to weak head normal form the above will
continue until it reaches the index, you reach the element, or
you hit the end of the list. Let us consider an example:

CHAPTER 28. BASIC LIBRARIES 1714
[1,2,3]!?2
-- matches final case
(_:[2,3])!?2
=[2,3]!?(2-1)
-- not a data constructor, keep going
[2,3]!?1
-- matches final case
(_:[3])!?1
=[3]!?(1-1)
-- not a data constructor, keep going
[3]!?0
-- matches Just case
(x:[])!?0=Justx
-- We stop at Just
In the above, we happen to know 𝑥is 3, but it’ll get thunked
if it wasn’t opportunistically evaluated on construction of the
list.
Next, let’s look at the types of the following functions:

CHAPTER 28. BASIC LIBRARIES 1715
defaultMain ::[Benchmark ]->IO()
whnf::(a->b)->a->Benchmarkable
nf::Control.DeepSeq.NFDatab=>
(a->b)->a->Benchmarkable
The reason it wants a function it can apply an argument
to is so that the result isn’t shared, which we discussed in the
previous chapter. We want it to re-perform the work for each
sampling in the benchmark results, so this design prevents
that sharing. Keep in mind that if you want to use your own
datatype with nf, which has an NFData constraint you will need
to provide your own instance. You can find examples in the
deepseq library on Hackage.
Our goal with this example is to equalize the performance
diﬀerence between (!?)and(!!). In this case, we’ve derived
the implementation of (!?)from the Report version of (!!).
Here’s how it looks in base:

CHAPTER 28. BASIC LIBRARIES 1716
-- Go to the Data.List docs in `base`,
-- click the source link for (!!)
#ifdefUSE_REPORT_PRELUDE
xs!!n|n<0=
error"Prelude.!!: negative index"
[]!! _ =
error"Prelude.!!: index too large"
(x:_)!!0=x
(_:xs)!!n=xs!!(n-1)
#else
However, after you run the benchmarks, you’ll find our
version based on the above isn’t quite as fast.4Fair enough! It
turns out that most of the time when there’s a Report version as
well as a non-Report version of a function in base, it’s because
they found a way to optimize it and make it faster. If we look
down from the #else, we can find the version that replaced it:
4Note that if you get weird benchmark results, you’ll want to resort to the old pro-
grammer’s trick of wiping your build. With Stack you’d run stack clean , with Cabal it’d
becabal clean . Inexplicable things happen sometimes. You shouldn’t need to do this
regularly, though.

CHAPTER 28. BASIC LIBRARIES 1717
-- negIndex and tooLarge are a bottom
-- and a const bottom respectively.
{-# INLINABLE (!!) #-}
xs!!n
|n<0=negIndex
|otherwise =
foldr
(\x r k-> case kof
0->x
_ ->r (k-1))
tooLarge xs n
The non-Report version is written in terms of foldr, which
often benefits from the various rewrite rules and optimizations
attached to foldr— rules we will not be explaining here at all,
sorry. This version also has a pragma letting GHC know it’s
okay to inline the code of the function where it’s used when
the cost estimator thinks it’s worthwhile to do so. So, let’s
change our version of this operator to match this version to
make use of those same optimizations:

CHAPTER 28. BASIC LIBRARIES 1718
infixl9!?
{-# INLINABLE (!?) #-}
xs!?n
|n<0=Nothing
|otherwise =
foldr
(\x r k->
casekof
0->Justx
_ ->r (k-1))
(constNothing) xs n
If you run this, you’ll find that…things have not improved.
So, what can we do to improve the performance of our opera-
tor?
Well, unless you added one already, you’ll notice the type
signature is missing. If you add a declaration that the number
argument is an Int, it should now perform the same as the
original:

CHAPTER 28. BASIC LIBRARIES 1719
infixl9!?
{-# INLINABLE (!?) #-}
(!?)::[a]->Int->Maybea
xs!?n
|n<0=Nothing
|otherwise =
foldr
(\x r k->
casekof
0->Justx
_ ->r (k-1))
(constNothing) xs n
Change the function in your module to reflect this and run
the benchmark again to check.
The issue was that the version with an inferred type was
defaulting the Num a => a toInteger which compiles to a less
efficient version of this code than does one that specifies the
typeInt. TheIntversion will turn into a more primitive, faster
loop. You can verify this for yourself by specifying the type
Integer and re-running the code or comparing the GHC Core
output for each version.

CHAPTER 28. BASIC LIBRARIES 1720
More on whnf and nf
Let’s return now to the question of when we should use whnf
ornf. You want to use whnfwhen the first data constructor is a
meaningful indicator of whether the work you’re interested in
has been done. Consider the simplistic example of a program
that is meant to locate some data in a database, say, a person’s
name and whether there are any known addresses for that
person. If it finds any data, it might print that information
into a file.
Thepartyou’reprobablytryingtojudgetheperformanceof
is the lookup function that finds the data and assesses whether
it exists, not how fast your computer can print the list of ad-
dresses into a file. In that case, what you care about is at the
level of weak head normal form, and whnfwill tell you more
precisely how long it is taking to find the data and decide
whether you have a Nothing or aJust a .
On the other hand, if you are interested in measuring the
time it takes to print your results, in addition to looking up the
data, then you may want to evaluate to normal form. There
are times when measuring that makes sense. We’ll see some
examples shortly.
For now, let us consider each indexing operator, the (!!)
that exists in baseand the one we’ve written that uses Maybe
instead of bottoms.
In the former case, the final result has the type 𝑎. The

CHAPTER 28. BASIC LIBRARIES 1721
function doesn’t stop recursing until it either returns bottom
or the value at that index. In either case, it’s done all the work
you’d care to measure — traversing the list. Evaluation to
WHNF means stopping at your 𝑎value.
In the latter case with Maybe, evaluation to WHNF means
stopping at either JustorNothing . It won’t evaluate the contents
of theJustdata constructor under whnf, but it will under nf.
Either is sufficient for the purposes of the benchmark as, again,
we’re measuring how quickly this code reaches the value at an
index in the list.
Let us consider an example with a few changes:

CHAPTER 28. BASIC LIBRARIES 1722
moduleMainwhere
importCriterion.Main
importDebug.Trace
myList::[Int]
myList=trace"myList was evaluated"
([1..9999]++[undefined])
-- your version of (!?) here
main::IO()
main=defaultMain
[ bench "index list 9999"
$whnf (myList !!)9998
, bench "index list maybe index 9999"
$nf (myList !?)9999
]
Notice what we did here. We added an undefined in what
will be the index position 9999. With the (!!)operator, we are
accessing the index just before that bottom value because there
is no outer data constructor (such as Nothing orJust) where we
could stop the evaluation. Both whnfandnfwill necessarily
force that bottom value.

CHAPTER 28. BASIC LIBRARIES 1723
We also modified the whnftonffor the benchmark of (!?).
Now it will evaluate the undefined it found at that index under
the bottom in the first run of the benchmark and fail:
benchmarking index list maybe index 9999
criterion1: Prelude.undefined
A function value that returned bottom instead of a data
constructor would’ve also acted as a stopping point for WHNF.
Consider the following:
Prelude> (Just undefined) `seq` 1
1
Prelude> (\_ -> undefined) `seq` 1
1
Prelude> ((\_ -> Just undefined) 0) `seq` 1
1
Prelude> ((\_ -> undefined) 0) `seq` 1
*** Exception: Prelude.undefined
Much of the time, whnfis going to cover the thing you’re
trying to benchmark.
Making the case for nf
Let us now look at an example of when whnfisn’t sufficient for
benchmarking, something that uses guarded recursion, unlike
(!!):

CHAPTER 28. BASIC LIBRARIES 1724
moduleMainwhere
importCriterion.Main
myList::[Int]
myList=[1..9999]
main::IO()
main=defaultMain
[ bench "map list 9999" $
whnf (map ( +1)) myList
]
The above is an example of guarded recursion because a
data constructor is interposed between each recursion step.
The data constructor is the cons cell when we’re talking about
map. Guarded recursion lets us consume the recursion steps
up to weak head normal form incrementally on demand.
Importantly, foldrcan be used to implement guarded and
unguarded recursion, depending entirely on what the folding
function does rather than any special provision made by foldr
itself. So what happens when we benchmark this?
Linking bin/bench ...
time
8.844 ns (8.670 ns .. 9.033 ns)

CHAPTER 28. BASIC LIBRARIES 1725
0.998 R² (0.997 R² .. 1.000 R²)
mean
8.647 ns (8.567 ns .. 8.751 ns)
std dev
293.3 ps (214.7 ps .. 412.9 ps)
variance introduced by outliers:
57% (severely inflated)
Well, that’s suspect. Does it really take 8.8 nanoseconds
to traverse a 10,000 element linked list in Haskell? We saw
an example of how long it should take, roughly. This is an
example of our benchmark being too lazy. The issue is that
mapuses guarded recursion and the cons cells of the list are
interposed between each recursion of map. You may recall this
from the lists and folds chapters. So, it ends up evaluating
only this far:
(_ : _)
Ah, that first data constructor. It has neither done the work
of incrementing the value nor has it traversed the rest of the
list. It’s just sitting there at the first cons cell. Using bottoms,
you can progressively prove to yourself what whnfis evaluating
by replacing things and re-running the benchmark:

CHAPTER 28. BASIC LIBRARIES 1726
-- is it applying (+1)?
myList=(undefined :[2..9999])
-- Is it going any further in the list?
myList::[Int]
myList=(undefined :undefined)
-- This should s'plode because
-- it'll be looking for that first
-- data constructor or (->) to stop at
myList::[Int]
myList=undefined
No matter, we can fix this!
-- change this bit
whnf(map (+1)) myList
-- into:
nf(map (+1)) myList
Then we get:
time
122.5 μs (121.7 μs .. 123.9 μs)
0.999 R² (0.998 R² .. 1.000 R²)

CHAPTER 28. BASIC LIBRARIES 1727
mean
123.0 μs (122.0 μs .. 125.6 μs)
std dev
5.404 μs (2.806 μs .. 9.323 μs)
That is considerably more realistic considering we’ve eval-
uated the construction of a whole new list. This is slower than
the indexing operation because we’re not just kicking a new
value out, we’re also constructing a new list.
In general when deciding between whnfandnf, ask yourself,
“when I have reached the first data constructor, have I done
most or all of the work that matters?” Be careful not to use nf
too much. If you have a function that returns a nontrivial data
structure or collection for which it’s already done all the work
to produce, nfwill make your code look excessively slow and
lead you on a wild goose chase.
28.3 Profiling your programs
We’re going to do our best to convey what you should know
about profiling programs with GHC and what we think is con-
ceptually less well covered, but we aren’t going to presume to
replace the GHC User Guide. We strongly recommend you
read the guide5for more information.
5https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/profiling.html

CHAPTER 28. BASIC LIBRARIES 1728
Profiling time usage
Sometimes rather than seeing how fast our programs are, we
want to know why they’re slow or fast and where they’re spend-
ing their time. To that end, we use profiling. First, let’s put
together a simple example for motivating this:
-- profilingTime.hs
moduleMainwhere
f::IO()
f= do
print ([ 1..]!!999999)
putStrLn "f"
g::IO()
g= do
print ([ 1..]!!9999999)
putStrLn "g"
main::IO()
main= do
f
g
Given that we traverse 10 times as much list structure in the

CHAPTER 28. BASIC LIBRARIES 1729
case of 𝑔, we believe we should see something like 10 times
as much CPU time spent in 𝑔. We can do the following to
determine if that’s the case:
$ stack ghc -- -prof -fprof-auto \
> -rtsopts -O2 profile.hs
./profile +RTS -P
cat profile.prof
Breaking down what each flag does:
1.-profenables profiling. Profiling isn’t enabled by default
because it can lead to slower programs but this generally
isn’t an issue when you’re investigating the performance
of your programs. Used alone, -profwill require you to
annotate cost centers manually, places for GHC to mark
for keeping track of how much time is spent evaluating
something.
2.-fprof-auto assigns all bindings not marked inline a cost
center named after the binding. This is fine for little stuﬀ
or otherwise not terribly performance-sensitive stuﬀ, but
if you’re dealing with a large program or one sensitive
to perturbations from profiling, it may be better to not
use this and instead assign your “SCCs” manually. SCC is
what the GHC documentation calls a cost center.

CHAPTER 28. BASIC LIBRARIES 1730
3.-rtsopts enables you to pass GHC RTS options to the gen-
erated binary. This is optional so you can get a smaller
binary if desired. We need this to tell our program to
dump the profile to the .proffile named after our pro-
gram.
4.-O2enables the highest level of program optimizations.
This is wise if you care about performance but -Oby itself
also enables optimizations, albeit somewhat less aggres-
sive ones. Either option can make sense when bench-
marking; it’s a case by case thing, but most Haskell pro-
grammers feel pretty free to default to -O2.
After examining the .proffile which contains the profiler
output, this is roughly what we’ll see:
Sun Feb 14 21:34 2016
Time and Allocation
Profiling Report (Final)
profile +RTS -P -RTS
total time = 0.22 secs
(217 ticks @ 1000 us, 1 processor)
total alloc = 792,056,016 bytes
(excludes profiling overheads)

CHAPTER 28. BASIC LIBRARIES 1731
COST CENTRE MODULE %time %alloc ticks bytes
g Main 91.2 90.9 198 720004344
f Main 8.8 9.1 19 72012568
...later noise snipped,
we care about the above...
And indeed, 91.2% time spent in 𝑔, 8.8% time spent in 𝑓would
seem to validate our hypothesis here.
Time isn’t the only thing we can profile. We’d also like
to know about the space (or memory) diﬀerent parts of our
program are responsible for using.
Profiling heap usage
We have measured time; now we shall measure space. Well,
memory anyway; we’re not astrophysicists. We’re going to
keep this quick and boring so that we might be able to get to
the good stuﬀ:

CHAPTER 28. BASIC LIBRARIES 1732
moduleMainwhere
importControl.Monad
blah::[Integer]
blah=[1..1000]
main::IO()
main=
replicateM_ 10000(print blah)
ghc -prof -fprof-auto -rtsopts -O2 loci.hs
./loci +RTS -hc -p
hp2ps loci.hp
If you open the loci.ps postscript file with your PDF reader
of choice, you’ll see how much memory the program used
over the time the program ran. Note that you’ll need the
program to run a minimum amount of time for the profiler
to get any samples of the heap size.
28.4 Constant applicative forms
Whenwe’retalkingaboutmemoryusageandsharinginHaskell,
we have to also talk about CAFs: constant applicative forms.
CAFs are expressions that have no free variables and are held

CHAPTER 28. BASIC LIBRARIES 1733
in memory to be shared with all other expressions in a module.
They can be literal values or partially applied functions that
have no named arguments.
We’re going to construct a very large CAF here. Notice we
are mapping over an infinite list and want to know how much
memory this uses. You might consider betting on a lot:
moduleMainwhere
incdInts ::[Integer]
incdInts =map (+1) [1..]
main::IO()
main= do
print (incdInts !!1000)
print (incdInts !!9001)
print (incdInts !!90010)
print (incdInts !!9001000)
print (incdInts !!9501000)
print (incdInts !!9901000)
Now we can profile that:
Thu Jan 21 23:25 2016
Time and Allocation
Profiling Report (Final)

CHAPTER 28. BASIC LIBRARIES 1734
cafSaturation +RTS -p -RTS
total time = 0.28 secs
(283 ticks @ 1000 us, 1 processor)
total alloc = 1,440,216,712 bytes
(excludes profiling overheads)
COST CENTRE MODULE %time %alloc
incdInts Main 90.1 100.0
main Main 9.9 0.0
-- some irrelevant bits elided
COST CENTRE MODULE no. entries %time %alloc
MAIN MAIN 45 0 0.0 0.0
CAF Main 89 0 0.0 0.0
incdInts Main 91 1 90.1 100.0
main Main 90 1 9.9 0.0
Note how incdInts is its own constant applicative form (CAF)
here apart from main. And notice the size of that memory
allocation. It’s because that mapping over an infinite list is a
top-level value that can be shared throughout a module, so it
must be evaluated and the results held in memory in order to
be shared.
CAFs include
•values;

CHAPTER 28. BASIC LIBRARIES 1735
•partially applied functions without named arguments;
•fully applied functions such as incdInts above, although
that would be rare in real code.
CAFs can make some programs faster since you don’t have
to keep re-evaluating shared values; however, CAFs can be-
come memory-intensive quite quickly. The important take-
away is that, if you find your program using much more mem-
ory than you expected, find the golden CAF and kill it.
Fortunately, CAFs mostly occur in toy code. Real world
code is usually pulling the data from somewhere, which avoids
the problem of holding large amounts of data in memory.
Let’s look at a way to avoid creating a CAF by introducing
an argument into our incdInts example:
moduleMainwhere
-- not a CAF
incdInts ::[Integer]->[Integer]
incdInts x=map (+1) x
main::IO()
main= do
print (incdInts [ 1..]!!1000)
If you examine the profile:

CHAPTER 28. BASIC LIBRARIES 1736
CAF
main
incdInts
Pointfree top-level declarations will be CAFs, but pointful
ones will not. We’d discussed this to some degree in the pre-
vious chapter as well. The reason the diﬀerence matters is
often not because of the total allocation reported by the pro-
file, which is often misleading anyway. Rather it’s important
because lists are as much control structures as data structures
and it’s very cheap in GHC to construct a list which is thrown
away immediately. Doing so might increase how much mem-
ory you allocate in total, but unlike a CAF, it won’t stay in your
heap which may lead to lower peak memory usage and the
runtime spending less time collecting garbage.
Indeed, this is not a standalone CAF. But what if we eta
reduce it (that is, remove its named arguments) so that it is
pointfree?

CHAPTER 28. BASIC LIBRARIES 1737
moduleMainwhere
-- Gonna be a CAF this time.
incdInts ::[Integer]->[Integer]
incdInts =map (+1)
main::IO()
main= do
print (incdInts [ 1..]!!1000)
This time when you look at the profile, it’ll be its own CAF:
CAF
incdInts
main
incdInts
GREAT SCOTT!
It doesn’t really change the performance for something so
trivial, but you get the idea. The big diﬀerence between the
two is in the heap profiles. Check them and you will likely see
what we mean.
28.5 Map
We’re going to start our exploration of data structures with Map.
It’s worth pointing out here that most of the structures we’ll

CHAPTER 28. BASIC LIBRARIES 1738
look at are, in some sense, replacements for the lists we have
depended on throughout the book. Lists and strings are useful
for a lot of things, but they’re not the most performant or even
the most useful way to structure your data. What is the most
performant or useful for any given program can vary, so we
can’t give a blanket recommendation that you should always
use any one of the structures we’re going to talk about. You
have to judge that based on what problems you’re trying to
solve and use benchmarking and profiling tools to help you
fine tune the performance.
Most of the data structures we’ll be looking at are in the
containers6library. If you build it, Mapwill come. And also
Sequence andSetand some other goodies. You’ll notice a lot of
the data structures have a similar API, but each are designed
for diﬀerent sets of circumstances.
We’ve used the Maptype before to represent associations of
unique pairings of keys to values. You may remember it from
the Testing chapter in particular, where we used it to look up
Morse code translations of alphabetic characters. Those were
fun times, so carefree and innocent.
The structure of the Maptype looks like this:
6http://hackage.haskell.org/package/containers

CHAPTER 28. BASIC LIBRARIES 1739
dataMapk a
=Bin
{-# UNPACK #-}
!Size!k a
!(Mapk a)!(Mapk a)
|Tip
typeSize=Int
You may recognize the exclamation marks denoting strict-
nessfromthe sectionson bangpatterns inthe previouschapter.
Tip is a data constructor for capping oﬀ the branch of a tree.
If you’d like to find out about the unpacking of strict fields,
which is what the UNPACK pragma is for; see the GHC documen-
tation for more information.
What’s something that’s faster with Map?
Well, lookups by key in particular are what it’s used for. Con-
siderthefollowingcomparisonofanassociationlistand Data.Map :
moduleMainwhere
importCriterion.Main
import qualified Data.Map asM

CHAPTER 28. BASIC LIBRARIES 1740
genList ::Int->[(String,Int)]
genList n=go n[]
wherego0xs=("0",0):xs
go n' xs =
go (n'-1) ((show n', n') :xs)
pairList ::[(String,Int)]
pairList =genList 9001
testMap ::M.MapStringInt
testMap =M.fromList pairList
main::IO()
main=defaultMain
[ bench "lookup one thing, list" $
whnf (lookup "doesntExist" ) pairList
, bench "lookup one thing, map" $
whnf (M.lookup"doesntExist" ) testMap
]
Association lists such as pairList are fine if you need some-
thing cheap and cheerful for a very small series of pairs, but
you’re better oﬀ using Mapby default when you have keys and
values. You get a handy set of baked-in functions for looking
things up and an efficient means of doing so. Insert operations

CHAPTER 28. BASIC LIBRARIES 1741
also benefit from being able to find the existing key-value pair
inMapmore quickly than in association lists.
What’s slower with Map?
Using an Intas your key type is usually a sign you’d be better oﬀ
with aHashMap ,IntMap , orVector , depending on the semantics of
your problem. If you need good memory density and locality
— which will make aggregating and reading values of a large
Vector faster, then Mapmight be inappropriate and you’ll want
Vector instead.
28.6 Set
This is also in containers. It’s like a Map, but without the ‘value’
part of the key-value pair. You may be asking yourself, why
do I want only keys?
When we use Map, it has an Ordconstraint on the functions
to ensure the keys are in order. That is one of the things that
makes lookups in Mapparticularly efficient. Knowing that the
keys will be ordered divides the problem space up by halves:
if we’re looking for the key 6 in a set of keys from 1-10, we
don’t have to search in the first half of the set because those
numbers are less than 6. Set, likeMap, is structured associatively,
not linearly.

CHAPTER 28. BASIC LIBRARIES 1742
Functions with Sethave the same Ordconstraint, but now
we don’t have key-value pairs — we only have keys. Another
way to think of it is the keys are now the values. That means
thatSetrepresents a unique, ordered set of values.
Here is the datatype for Set:
dataSeta
=Bin
{-# UNPACK #-}
!Size!a!(Seta)!(Seta)
|Tip
typeSize=Int
It’s eﬀectively equivalent to a Maptype with unit values.
moduleMainwhere
importCriterion.Main
import qualified Data.Map asM
import qualified Data.Set asS
bumpIt(i, v)=(i+1, v+1)

CHAPTER 28. BASIC LIBRARIES 1743
m::M.MapIntInt
m=M.fromList $take10000stream
wherestream=iterate bumpIt ( 0,0)
s::S.SetInt
s=S.fromList $take10000stream
wherestream=iterate ( +1)0
membersMap ::Int->Bool
membersMap i=M.member i m
membersSet ::Int->Bool
membersSet i=S.member i s
main::IO()
main=defaultMain
[ bench "member check map" $
whnf membersMap 9999
, bench "member check set" $
whnf membersSet 9999
]
If you benchmark the above, you should get very similar
results for both, with Mapperhaps being a smidgen slower than
Set. They’re similar because they’re nearly identical data struc-
tures in the containers library.

CHAPTER 28. BASIC LIBRARIES 1744
There’s not much more to say. It has the same pros and
cons7asMap, with the same performance of the core operations,
save that you’re more limited.
Exercise: Benchmark Practice
Make a benchmark to prove for yourself whether MapandSet
have similar performance. Try operations other than mem-
bership testing, such as insertion or unions.
28.7 Sequence
Sequence is a nifty datatype built atop finger trees; we aren’t
going to address finger trees in this book, unfortunately, but we
encourage you to check them out.8Sequence appends cheaply
on the front and the back, which avoids a common problem
with lists where you can only cons to the front cheaply.
Here is the datatype for Sequence :
7HA HA GET IT?!
8see, for example, Finger Trees: A Simple General-purpose Data Structure by Ralf
Hinze and Ross Paterson

CHAPTER 28. BASIC LIBRARIES 1745
newtype Seqa=Seq(FingerTree (Elema))
-- Elem is so elements and nodes can be
-- distinguished in the types of the
-- implementation. Don't sweat it.
newtype Elema=Elem{ getElem ::a }
dataFingerTree a
=Empty
|Singlea
|Deep{-# UNPACK #-} !Int!(Digita)
(FingerTree (Nodea))!(Digita)
What’s faster with Sequence?
Updates (cons and append) to both ends of the data structure
and concatenation are what Sequence is particularly known for.
You won’t want to resort to using Sequence by default though,
as the list type is often competitive. Sequence will have more
efficient access to the tail than a list will. Here’s an example
whereSequence does better because the list is a bit big:

CHAPTER 28. BASIC LIBRARIES 1746
moduleMainwhere
importCriterion.Main
import qualified Data.Sequence asS
lists::[[Int]]
lists=replicate 10[1..100000]
seqs::[S.SeqInt]
seqs=
replicate 10(S.fromList [ 1..100000])
main::IO()
main=defaultMain
[ bench "concatenate lists" $
nf mconcat lists
, bench "concatenate sequences" $
nf mconcat seqs
]
Note that in the above, a substantial amount of the time in
the benchmark is spent traversing the data structure because
we’re evaluating to normal form to ensure all the work is done.
Incidentally, this accentuates the diﬀerence between a list and
Sequence because it’s faster to index or traverse a sequence than

CHAPTER 28. BASIC LIBRARIES 1747
a list. Consider the following:
moduleMainwhere
importCriterion.Main
import qualified Data.Sequence asS
lists::[Int]
lists=[1..100000]
seqs::S.SeqInt
seqs=S.fromList [ 1..100000]
main::IO()
main=defaultMain
[ bench "indexing list" $
whnf (\xs->xs!!9001) lists
, bench "indexing sequence" $
whnf (flip S.index9001) seqs
]
The diﬀerence between list and Sequence in the above when
we ran the benchmark was two orders of magnitude.

CHAPTER 28. BASIC LIBRARIES 1748
What’s slower with Sequence?
Sequence is a persistent data structure like Map, so the memory
density isn’t as good as it is with Vector (we’re getting there).
Indexing by Intwill be faster with Vector as well. List will be
faster with consing and concatenation in some cases, if the
lists are small. When you know you need cheap appending
to the end of a long list, it’s worth giving Sequence a try, but
it’s better to base the final decision on benchmarking data if
performance matters.
28.8 Vector
The next data structure we’re going to look at is not in contain-
ers. It’s in its own library called, unsurprisingly, vector9. You’ll
notice it says vectors are “efficient arrays.” We’re not going to
look at arrays, or Haskell’s Arraytype, specifically here, though
you may already be familiar with the idea.
Onerarelyusesarrays, ormorespecifically, Array10inHaskell.
Vector is almost always what you want instead of an array. The
default Vector type is implemented as a slice wrapper of Array;
we can see this in the definition of the datatype:
9https://hackage.haskell.org/package/vector
10http://hackage.haskell.org/package/array

CHAPTER 28. BASIC LIBRARIES 1749
-- | Boxed vectors, supporting
-- efficient slicing.
dataVectora=
Vector{-# UNPACK #-} !Int
{-# UNPACK #-} !Int
{-# UNPACK #-} !(Arraya)
deriving (Typeable )
There are many variants of Vector, although we won’t dis-
cuss them all here. These include boxed, unboxed, immutable,
mutable, and storable vectors, but the plain version above is
the usual one you’d use. We’ll talk about mutable vectors in
their own section. Boxed means the vector can reference any
datatype you want; unboxed represents raw values without
pointer indirection.11The latter can save a lot of memory but
is limited to types like Bool,Char,Double ,Float,Int,Word, tuples
of unboxable values. Since a newtype is unlifted and doesn’t
introduce any pointer indirection, you can unbox any newtype
that contains an unboxable type.
When does one want a Vector in Haskell?
You want a Vector when
11This book isn’t the right place to talk about what pointers are in detail. Briefly, they
are references to things in memory, rather than the things themselves.

CHAPTER 28. BASIC LIBRARIES 1750
•you need memory efficiency close to the theoretical max-
imum for the data you are working with;
•your data access is almost exclusively in terms of indexing
via anIntvalue;
•you want uniform access times for accessing each element
in the data structure; and/or,
•you will construct a Vector once and read it many times (al-
ternatively, you plan to use a mutable Vector for ongoing,
efficient updates).
What’s this about slicing?
Remember this from the definition of Vector ?
-- | Boxed vectors, supporting
-- efficient slicing.
Slicing refers to slicing oﬀ a portion, or creating a sub-array.
TheVector type is designed for making slicing much cheaper
than it otherwise would be. Consider the following:

CHAPTER 28. BASIC LIBRARIES 1751
moduleMainwhere
importCriterion.Main
import qualified Data.Vector asV
slice::Int->Int->[a]->[a]
slicefrom len xs =
take len (drop from xs)
l::[Int]
l=[1..1000]
v::V.VectorInt
v=V.fromList [ 1..1000]
main::IO()
main=defaultMain
[ bench "slicing list" $
whnf (head .slice100900) l
, bench "slicing vector" $
whnf (V.head.V.slice100900) v
]
If you run this benchmark, Vector should be faster than
list. Why? Because when we constructed that new Vector with

CHAPTER 28. BASIC LIBRARIES 1752
V.slice , all it had to do was the following:
-- from Data.Vector
instance G.VectorVectorawhere
-- other methods elided
{-# INLINE basicUnsafeSlice #-}
basicUnsafeSlice j n ( Vectori_arr)=
Vector(i+j) n arr
What makes Vector nicer than lists and Arrayin this respect
is that when you construct a slice or view of another Vector , it
doesn’t have to construct as much new data. It returns a new
wrapper around the original underlying array with a new index
and oﬀset with a reference to the same original Array. Doing
the same with an ordinary Arrayor a list would’ve required
copying more data. Speed comes from being sneaky and
skipping work.
Updating vectors
Persistent vectors are not great at handling updates on an
ongoing basis, but there are some situations in which they can
surprise you. One such case is fusion12. Fusion, or loop fusion,
12Stream Fusion; Duncan Coutts; http://code.haskell.org/~dons/papers/icfp088-coutts.
pdf

CHAPTER 28. BASIC LIBRARIES 1753
means that as an optimization the compiler can fuse several
loops into one megaloop and do it in one pass:
moduleMainwhere
importCriterion.Main
import qualified Data.Vector asV
testV'::Int->V.VectorInt
testV'n=
V.map (+n)$V.map (+n)$
V.map (+n)$V.map (+n)
(V.fromList [ 1..10000])
testV::Int->V.VectorInt
testVn=
V.map ( (+n).(+n)
.(+n).(+n) )
(V.fromList [ 1..10000])

CHAPTER 28. BASIC LIBRARIES 1754
main::IO()
main=defaultMain
[ bench "vector map prefused" $
whnf testV 9998
, bench "vector map will be fused" $
whnf testV' 9998
]
The vector library has loop fusion built in, so in a lot of
cases, such as with mapping, you won’t construct 4 vectors just
because you mapped four times. Through the use of GHC
RULES13the code in testV’ will change into what you see in
testV. It’s worth noting that part of the reason this optimization
is sound is because we know what code performs eﬀects and
what does not because we have types.
However, loop fusion isn’t a panacea and there will be situ-
ations where you want to change one element at a time, selec-
tively. Let’s consider more efficient ways of updating vectors.
We’re going to use the (//)operator from the vector library
here. It’s a batch update operator that allows you to modify
several elements of the vector at one time:
13https://wiki.haskell.org/GHC/Using_rules

CHAPTER 28. BASIC LIBRARIES 1755
moduleMainwhere
importCriterion.Main
importData.Vector ((//))
import qualified Data.Vector asV
vec::V.VectorInt
vec=V.fromList [ 1..10000]
slow::Int->V.VectorInt
slown=go n vec
wherego0v=v
go n v=go (n-1) (v//[(n,0)])
batchList ::Int->V.VectorInt
batchList n=vec//updates
whereupdates =
fmap (\n->(n,0)) [0..n]
main::IO()
main=defaultMain
[ bench "slow"$whnf slow 9998
, bench "batch list" $
whnf batchList 9998
]

CHAPTER 28. BASIC LIBRARIES 1756
The issue with the first example is that we’re using a batch
API… but not in batch. It’s much cheaper (500–1000x in our
tests) to construct the list of updates all at once and then feed
them to the (//)function. We can make it even faster still by
using the update function that uses a vector of updates:
batchVector ::Int->V.VectorInt
batchVector n=V.unsafeUpdate vec updates
whereupdates =
fmap (\n->(n,0))
(V.fromList [ 0..n])
Benchmarkingthis versionshould getyoucode thatis about
1.4x faster. But we’re greedy. So we want more.
Mutable Vectors
“Moria! Moria! Wonder of the Northern world. Too
deep we delved there, and woke the nameless fear.”
— Glóin, The Fellowship of the Ring
What if we want something even faster? Although many
people don’t realize this about Haskell, we can obtain the bene-
fits of in-place updates if we so desire. What sets Haskell apart
is that we cannot do so in a way that compromises referential
transparency or the nice equational properties our expressions

CHAPTER 28. BASIC LIBRARIES 1757
have. First we’ll demonstrate how to do this, then we’ll talk
about how this is designed to behave predictably.
Here comes an example:
moduleMainwhere
importControl.Monad.Primitive
importControl.Monad.ST
importCriterion.Main
import qualified Data.Vector asV
import qualified Data.Vector.Mutable asMV
import qualified
Data.Vector.Generic.Mutable asGM
mutableUpdateIO
::Int
->IO(MV.MVector RealWorld Int)
mutableUpdateIO n= do
mvec<-GM.new (n+1)
go n mvec
wherego0v=return v
go n v=
(MV.write v n 0)>>go (n-1) v

CHAPTER 28. BASIC LIBRARIES 1758
mutableUpdateST ::Int->V.VectorInt
mutableUpdateST n=runST$ do
mvec<-GM.new (n+1)
go n mvec
wherego0v=V.freeze v
go n v=
(MV.write v n 0)>>go (n-1) v
main::IO()
main=defaultMain
[ bench "mutable IO vector"
$whnfIO (mutableUpdateIO 9998)
, bench "mutable ST vector"
$whnf mutableUpdateST 9998
]
We’re going to talk about runSTshortly. For the moment,
let’s concentrate on the fact that the mutable IOversion is
approximately 7,000x faster than the original unbatched loop
in our tests. The STversion is about 1.5x slower than the IO
version, but still very fast. The added time in the STversion is
from freezing the mutable vector into an ordinary vector. We
won’t explain STfully here, but, as we’ll see, it can be handy
when you want to temporarily make something mutable and
ensure no mutable references are exposed outside of the ST
monad.

CHAPTER 28. BASIC LIBRARIES 1759
Here were the timings we got with the various vector oper-
ations on our computer:
Variant Microseconds
slow 133,600
batchList 244
batchVector 176
mutableUpdateST 32
mutableUpdateIO 19
Notably, most of the performance improvement was from
not doing something silly. Don’t resort to the use of mutation
viaIOorSTexcept where you know it’s necessary. It’s worth
noting that given our test involved updating almost 10,000
indices in the vector, we spent an average of 1.9 nanoseconds
per update when we used mutability and 17.6 ns when we did
it in batch with a persistent vector.
A sidebar on the ST Monad
STcan be thought of as a mutable variant of the strict State
monad. From another angle, it could be thought of as IO
restricted exclusively to mutation which is guaranteed safe.
Safe how? STis relying on the principle behind that old
philosophical saw about the tree that falls in the forest with no
one to see it fall. The idea behind this “morally eﬀect-free” un-
derstanding of mutable state was introduced in the paper Lazy

CHAPTER 28. BASIC LIBRARIES 1760
Functional State Threads.14It unfreezes your data, mutates
it, then refreezes it so that it can’t mutate anymore. Thus it
manages to mutate and still maintain referential transparency.
ForSTto work properly, the code that mutates the data must
not get reordered by the optimizer or otherwise monkeyed
with, much like the code in IO. So there must be something
underlying the type which prevents GHC ruining our day. Let
us examine the STtype:
Prelude> import Control.Monad.ST
Prelude> :info ST
type role ST nominal representational
newtype ST s a =
GHC.ST.ST (GHC.ST.STRep s a)
...
Prelude> import GHC.ST
Prelude> :info STRep
type STRep s a =
GHC.Prim.State# s
-> (# GHC.Prim.State# s, a #)
There’s a bit of ugliness in here that shouldn’t be too sur-
prising after you’ve seen GHC Core in the previous chapter.
What’s important is that 𝑠is getting its type from the thing
14Lazy Functional State Threads; John Launchbury and Simon Peyton Jones

CHAPTER 28. BASIC LIBRARIES 1761
you’re mutating, but it has no value-level witness. The State
monad here is therefore erasable; it can encapsulate this mu-
tation process and then melt away.
It’s important to understand that 𝑠isn’t the state you’re
mutating. The mutation is a side eﬀect of having entered
the closures that perform the eﬀect. This strict, unlifted state
transformer monad happens to structure your code in a way
that preserves the intended order of the computations and
their associated eﬀects.
By closures here, we mean lambda expressions. Of course
we do, because this whole book is one giant lambda expression
under the hood. Entering each lambda performs its eﬀects.
The eﬀects of a series of lambdas are not batched, because the
ordering is important when we’re performing eﬀects, as each
new expression might depend on the eﬀects of the previous
one. The eﬀects are performed and then, if the value that
should result from the computation is also going to be used,
the value is evaluated.
So what is the 𝑠type for? Well, while it’s all well and good to
ask politely that programmers freeze a mutable reference into
a persistent, immutable data structure as the final result…you
can’t count on that. STenforces it at compile time by making
it so that 𝑠will never unify with anything outside of the ST
Monad. The trick for this is called existential quantification15,
15Existentially quantified types; Haskell Wikibook
https://en.wikibooks.org/wiki/Haskell/Existentially_quantified_types

CHAPTER 28. BASIC LIBRARIES 1762
but having said this won’t necessarily mean anything to you
right now. But it does prevent you from accidentally leaking
mutable references to code outside ST, which could then lead
to code that does unpredictable things depending on the state
of the bits in memory.
Avoid dipping in and out of STover and over. The thaws
and freezes will cost you in situations where it might have
been better to just use IO. Batching your mutation is best for
performance and code comprehensibility.
Exercises: Vector
Setup a benchmark harness with criterion to profile how much
memory boxed and unboxed vectors containing the same
data uses. You can combine this with a benchmark to give it
something to do for a few seconds. We’re not giving you a lot
because you’re going to have to learn to read documentation
and source code anyway.
28.9 String types
The title is a slight bit of a misnomer as we’ll talk about two
string (or text) types and one type for representing sequences
ofbytes. Here’sabriefexplanationof String ,Text, andByteString :

CHAPTER 28. BASIC LIBRARIES 1763
String
You know what String is. It’s a type alias for a list of Char, yet
underneath it’s not quite as simple as an actual list of Char.
One of the benefits of using strings is the simplicity, and
they’re easy enough to explain. For most demonstration and
toy program purposes, they’re fine.
However, like lists themselves, they can be infinite. The
memory usage for even very large strings can get out of control
rapidly. Furthermore, they have very inefficient character-by-
character indexing into the String. The time taken to do a
lookup increases proportionately with the length of the list.
Text
This one comes in the text16library on Hackage. This is best
when you have plain text, but need to store the data more
efficiently — particularly as it concerns memory usage.
We’ve used this one a bit in previous chapters, when we
were playing with OverloadedStrings . The benefits here are that
you get:
•compact representation in memory; and
•efficient indexing into the string.
16http://hackage.haskell.org/package/text

CHAPTER 28. BASIC LIBRARIES 1764
However, Textis encoded as UTF-16, and that isn’t what
most people expect given UTF-8’s popularity. In Text’s defense,
UTF-16 is often faster due to cache friendliness via chunkier
and more predictable memory accesses.
Don’t trust your gut, measure
We mentioned Texthas a more compact memory represen-
tation, but what do you think the memory profile for the
following program will look up? High first, then low, or low
first, then high?

CHAPTER 28. BASIC LIBRARIES 1765
moduleMainwhere
importControl.Monad
import qualified Data.Text asT
import qualified Data.Text.IO asTIO
import qualified System.IO asSIO
-- replace "/usr/share/dict/words"
-- as appropriate
dictWords ::IOString
dictWords =
SIO.readFile "/usr/share/dict/words"
dictWordsT ::IOT.Text
dictWordsT =
TIO.readFile "/usr/share/dict/words"
main::IO()
main= do
replicateM_ 1000(dictWords >>=print)
replicateM_ 1000
(dictWordsT >>=TIO.putStrLn)
The issue is that Textwent ahead and loaded the entire file
into memory each of the ten times you forced the IOaction.

CHAPTER 28. BASIC LIBRARIES 1766
ThereadFile operation for String , however, was lazy and only
read as much of the file into memory as needed to print the
data to stdout. The proper way to handle incrementally pro-
cessing data is with streaming17, but this is not something we’ll
cover in detail in this book. However, there is a lazy way we
could change this:
-- Add the following to the module you
-- already made for profiling String
-- and Text.
import qualified Data.Text.Lazy asTL
import qualified Data.Text.Lazy.IO asTLIO
dictWordsTL ::IOTL.Text
dictWordsTL =
TLIO.readFile "/usr/share/dict/words"
main::IO()
main= do
replicateM_ 1000(dictWords >>=print)
replicateM_ 1000
(dictWordsT >>=TIO.putStrLn)
replicateM_ 1000
(dictWordsTL >>=TLIO.putStrLn)
17https://wiki.haskell.org/Pipes

CHAPTER 28. BASIC LIBRARIES 1767
Now you should see memory usage plummet again after
a middle plateau because you’re reading in as much text as
necessary to print, able to deallocate as you go. This is some,
but not all, of the benefits of streaming but we do strongly
recommend using streaming rather than relying on a lazy IO
API.
ByteString
Thisisnotastring. Ortext. Notnecessarilyanyway. ByteString s
are sequences of bytes represented (indirectly) as a vector of
Word8values. Text on a computer is made up of bytes, but it
has to have a particular encoding in order for it to be text.
Encodings of text can be ASCII, UTF-8, UTF-16, or UTF-32,
usually UTF-8 or UTF-16. The Texttype encodes the data as
UTF-16, partly for performance. It’s often faster to read larger
chunks of data at a time from memory, so the 16-bits-per-rune
encoding of UTF-16 performs a bit better in most cases.
The main benefit of ByteString is that it’s easy to use via the
OverloadedStrings extension but is bytes instead of text. This
addresses a larger problem space than mere text.
The flip side of that, of course, is that it encompasses byte
data that isn’t comprehensible text. That’s a drawback if you
didn’t mean to permit non-text byte sequences in your data.

CHAPTER 28. BASIC LIBRARIES 1768
ByteString examples
Here’s an example highlighting that ByteStrings are not always
text:
{-# LANGUAGE OverloadedStrings #-}
moduleBSwhere
import qualified Data.Text.IO asTIO
import qualified Data.Text.Encoding asTE
import qualified Data.ByteString.Lazy asBL
-- https://hackage.haskell.org/package/zlib
import qualified
Codec.Compression.GZip asGZip
We’re going to use the gzipcompression format to compress
some data. This is so we have an example of data that includes
bytes that aren’t a valid text encoding.
input::BL.ByteString
input="123"
compressed ::BL.ByteString
compressed =GZip.compress input
TheGZipmodule expects a lazy ByteString , probably so that
it’s streaming friendly.

CHAPTER 28. BASIC LIBRARIES 1769
main::IO()
main= do
TIO.putStrLn $TE.decodeUtf8 (s input)
TIO.putStrLn $
TE.decodeUtf8 (s compressed)
wheres=BL.toStrict
Theencodingmoduleinthetextlibraryexpectsstrict ByteString s,
so we have to make them strict before attempting a decoding.
The second text decode will fail because there’ll be a byte that
isn’t recognizably correct as an encoding of text information.
ByteString traps
You might think to yourself at some point, “I’d like to convert
aString to aByteString !” This is a perfectly reasonable thing to
want to do, but many Haskellers will mistakenly use the Char8
module in the bytestring library when that is not really what
they want. The Char8module is really a convenience for data
that mingles bytes and ASCII data18there. It doesn’t work for
Unicode and shouldn’t be used anywhere there’s even a hint
of possibility that there could be Unicode data. For example:
18Since ASCII is 7 bits and Char8is 8 bits you could use the eighth bit to represent
Latin-1 characters. However, since you will usually intend to convert the Char8data to
encodings like UTF-8 and UTF-16 which use the eighth bit diﬀerently that would be
unwise.

CHAPTER 28. BASIC LIBRARIES 1770
moduleChar8ProllyNotWhatYouWant where
import qualified Data.Text asT
import qualified Data.Text.Encoding asTE
import qualified Data.ByteString asB
import qualified
Data.ByteString.Char8 asB8
-- utf8-string
import qualified
Data.ByteString.UTF8 asUTF8
-- Manual unicode encoding of Japanese text
-- GHC Haskell allows UTF8 in source files
s::String
s="\12371\12435\12395\12385\12399\12289\
\20803\27671\12391\12377\12363\65311 "
utf8ThenPrint ::B.ByteString ->IO()
utf8ThenPrint =
putStrLn .T.unpack.TE.decodeUtf8

CHAPTER 28. BASIC LIBRARIES 1771
throwsException ::IO()
throwsException =
utf8ThenPrint ( B8.pack s)
bytesByWayOfText ::B.ByteString
bytesByWayOfText =TE.encodeUtf8 ( T.pack s)
-- letting utf8-string do it for us
libraryDoesTheWork ::B.ByteString
libraryDoesTheWork =UTF8.fromString s
thisWorks ::IO()
thisWorks =utf8ThenPrint bytesByWayOfText
alsoWorks ::IO()
alsoWorks =
utf8ThenPrint libraryDoesTheWork
Then we go to run the code that attempts to get a ByteString
via theChar8module which contains Unicode:
Prelude> throwsException
*** Exception: Cannot decode byte '\x93':
Data.Text.Internal.Encoding.decodeUtf8:
Invalid UTF-8 stream

CHAPTER 28. BASIC LIBRARIES 1772
You can use ordfromData.Char to get the Intvalue of the
byte of a character:
Prelude> import Data.Char (ord)
Prelude> :t ord
ord :: Char -> Int
Prelude> ord 'A'
65
Prelude> ord '\12435'
12435
The second example seems obvious, but when the data is
represented natively on your computer this is more useful.
Use non-English websites to get sample data to test.
We can now use the ordering of characters to find the first
character that breaks Char8:
Prelude> let xs = ['A'..'\12435']
Prelude> let cs = map (:[]) xs
Prelude> mapM_ (utf8ThenPrint . B8.pack) cs
... bunch of output ...
Then to narrow this down, we know we need to find what
came after the tilde and the \DELcharacter:
... some trial and error ...
Prelude> let f = take 3 $ drop 60

CHAPTER 28. BASIC LIBRARIES 1773
Prelude> mapM_ putStrLn (f cs)
}
~
Hrm, okay, but where is this in the ASCII table? We can
use the opposite of the ordfunction from Data.Char ,chrto
determine this:
Prelude> import Data.Char (chr)
Prelude> :t chr
chr :: Int -> Char
Prelude> map chr [0..128]
... prints the first 129 characters ...
What it printed corresponds to the ASCII table, which is
how UTF-8 represents the same characters. Now we can use
this function to narrow down precisely what our code fails at:
-- works fine
Prelude> utf8ThenPrint (B8.pack [chr 127])
-- fails
Prelude> utf8ThenPrint (B8.pack [chr 128])
*** Exception: Cannot decode byte '\x80':
Data.Text.Internal.Encoding.decodeUtf8:
Invalid UTF-8 stream

CHAPTER 28. BASIC LIBRARIES 1774
Don’t use Unicode characters with the Char8module! This
problem isn’t exclusive to Haskell — all programming lan-
guages must acknowledge the existence of diﬀerent encodings
for text.
Char8 is bad mmmmmkay TheChar8module is not for Uni-
code or for text more generally! The packfunction it contains
is for ASCII data only! This fools programmers because the
UTF-8 encoding of the English alphabet with some Latin ex-
tension characters intentionally overlaps exactly with the same
bytes ASCII uses to encode those characters. So the following
will work but is wrong in principle:
Prelude> utf8ThenPrint (B8.pack "blah")
blah
Getting a UTF-8 bytestring via the textorutf8-string li-
braries works fine, as you’ll see if you take a look at the result
ofthisWorks andalsoWorks .
When would I use ByteString instead of Text for
textual data?
This does happen sometimes, usually because you want to
keep data that arrived in a UTF-8 encoding in UTF-8. Of-
ten this happens because you read UTF-8 data from a file or
network socket and you don’t want the overhead of bouncing

CHAPTER 28. BASIC LIBRARIES 1775
it into and back out of Text. If you do this, you might want
to use newtypes to avoid accidentally mixing this data with
non-UTF-8 bytestrings.
28.10 Chapter Exercises
Diﬀerence List
Lists are really nice, but they don’t append or concatenate
cheaply. We covered Sequence as one potential solution to this,
but there’s a simpler data structure that solves slow appending
specifically, the diﬀerence list!
Rather than justify and explain diﬀerence lists, part of the
exercise is figuring out what it does and why (although feel
free to look up the documentation on Hackage). Attempt
the exercise before resorting to the tutorial in the follow-up
reading. First, the DListtype is built on top of ordinary lists,
but it’s a function:
newtype DLista=DL{ unDL::[a]->[a] }
The API that follows is based on code by Don Stewart and
Sean Leather. Here’s what you need to write:
1.empty::DLista
empty=undefined
{-# INLINE empty #-}

CHAPTER 28. BASIC LIBRARIES 1776
2.singleton ::a->DLista
singleton =undefined
{-# INLINE singleton #-}
3.toList::DLista->[a]
toList=undefined
{-# INLINE toList #-}
4.-- Prepend a single element to a dlist.
infixr`cons`
cons ::a->DLista->DLista
consx xs=DL((x:).unDL xs)
{-# INLINE cons #-}
5.-- Append a single element to a dlist.
infixl`snoc`
snoc::DLista->a->DLista
snoc=undefined
{-# INLINE snoc #-}
6.-- Append dlists.
append::DLista->DLista->DLista
append=undefined
{-# INLINE append #-}

CHAPTER 28. BASIC LIBRARIES 1777
What’s so nifty about DListis thatcons,snoc, andappend all
take the same amount of time no matter how long the dlist is.
That is to say, they take a constant amount of time rather than
growing with the size of the data structure.
Your goal is to get the following benchmark harness running
with the performance expected:
schlemiel ::Int->[Int]
schlemiel i=go i[]
wherego0xs=xs
go n xs =go (n-1) ([n]++xs)
constructDlist ::Int->[Int]
constructDlist i=toList$go i empty
wherego0xs=xs
go n xs =
go (n-1)
(singleton n `append` xs)
main::IO()
main=defaultMain
[ bench "concat list" $
whnf schlemiel 123456
, bench "concat dlist" $
whnf constructDlist 123456
]

CHAPTER 28. BASIC LIBRARIES 1778
If you run the above, the DListvariant should be about twice
as fast.
A simple queue
We’re going to write another data structure in terms of list, but
this time it’ll be a queue. The main feature of queues is that
we can add elements to the front cheaply and take items oﬀ
the back of the queue cheaply.
-- From Okasaki's Purely
-- Functional Data Structures
dataQueuea=
Queue{ enqueue ::[a]
, dequeue ::[a]
}deriving (Eq,Show)
-- adds an item
push::a->Queuea->Queuea
push=undefined
pop::Queuea->Maybe(a,Queuea)
pop=undefined
We’re going to give you less code this time, but your task is
to implement the above and write a benchmark comparing it

CHAPTER 28. BASIC LIBRARIES 1779
against performing alternating pushes and pops from a queue
based on a single list. Alternating so that you can’t take advan-
tage of reversing the list after a long series of pushes in order
to perform a long series of pops efficiently.
Don’t forget to handle the case where the dequeue is empty
and you need to shift items from the enqueue to the dequeue.
You need to do so without violating “first come, first served”.
Lastly, benchmark it against Sequence . Come up with a vari-
ety of tests. Add additional operations for your Queuetype if
you want.
28.11 Follow-up resources
1.Criterion tutorial; Bryan O’Sullivan
http://www.serpentine.com/criterion/tutorial.html
2.Demystifying DList; Tom Ellis
http://h2.jaguarpaw.co.uk/posts/demystifying-dlist/
3.Memory Management; GHC; Haskell Wiki
https://wiki.haskell.org/GHC/Memory_Management
4.Performance; Haskell Wiki
https://wiki.haskell.org/Performance
5.Pragmas, specifically UNPACK ; GHC Documentation

CHAPTER 28. BASIC LIBRARIES 1780
6.High Performance Haskell; Johan Tibell
http://johantibell.com/files/slides.pdf
7.Haskell Performance Patterns; Johan Tibell
8.Faster persistent data structures through hashing; Johan
Tibell
9.Lazy Functional State Threads; John Launchbury and
Simon Peyton Jones
10.Write Haskell as fast as C: exploiting strictness, laziness
and recursion; Don Stewart
11.Haskell as fast as C: A case study; Jan Stolarek
12.Haskell FFT 11: Optimisation Part 1; Ian Ross
13.Understanding the RealWorld; Edsko de Vries
14.Stream Fusion; Duncan Coutts
http://code.haskell.org/~dons/papers/icfp088-coutts.pdf
15.Purely functional data structures; Chris Okasaki

Chapter 29
IO
In those days, many
successful projects started
out as graffitis on a beer
mat in a very, very smoky
pub.
Peter J. Landin
1781

CHAPTER 29. IO 1782
29.1 IO
You should be proud of yourself for making it this far into the
book. You’re juggling monads. You’ve defeated an army of
monad transformers. You’re comfortable with using algebraic
structures in typeclass form. You’ve got a basic understand-
ing of how Haskell terms evaluate, nonstrictness, and sharing.
With those things in hand, let’s talk about IO.
We’ve used the IOtype at various times throughout the
book, with only cursory explanation. You no doubt know that
we use this type in Haskell as a means of keeping our chocolate
separate from our peanut butter — that is, our pure functions
from our eﬀectful ones. Perhaps you’re wondering how it all
works, what’s underneath that opaque type. To many people,
IOseems mysterious.
An eﬀectful function is one which has an observable impact
on the environment it is evaluated in, other than computing
and returning a result. Examples of eﬀects includes writing to
standard output (like putStrLn ), reading from standard input
(getChar ), or modifying state destructively ( ST). Implicit to this,
is that this almost always means the code requires it be evalu-
ated in a particular order. Haskell expressions which aren’t in
IOwill always return the same result regardless of what order
they are evaluated in; we lose this guarantee and others besides
onceIOis introduced.
Most explanations of the IOtype in Haskell don’t help much,

CHAPTER 29. IO 1783
either. They seem designed to confuse the reader rather than
convey anything useful. Don’t look now, but somebody is writ-
ing an explanation of IOright now that uses van Laarhoven
Free Monads and costate comonad coalgebras to explain some-
thing that’s much simpler than either of those topics.
We’re not going to do that here. We will instead try to
demystify IOa bit. The important thing about IOis that it’s a
special kind of datatype that disallows sharing in some cases.
In this chapter, we will
•explain how IOworks operationally;
•explore what it should mean to you when you read a type
that has IOin it;
•provide a bit more detail about the IO Functor ,Applicative ,
andMonad.
29.2 Where IO explanations go astray
A lot of explanations of IOare misleading or muddled. We’ve
already alluded to the overwrought and complex explanations
ofIO. Others call it “the IO Monad ” and seem to equate IOwith
Monad. While IOis a type that has a Monadinstance, it is not only
aMonadand monads are not only IO. Other presentations imply
that once you enter IO, you destroy purity and referential
transparency. And some references don’t bother to say much

CHAPTER 29. IO 1784
aboutIObecause the fact that it remains opaque won’t stop
you from doing most of what you want to use it for anyway.
We want to oﬀer some guidance in critically evaluating
explanations of IO. Let us consider one of the most popular
explanations of IO, the one that attempts to explain IOin terms
ofState.
Burn the State to the ground!
The temptation to use Stateto get someone comfortable with
the idea of IOis strong. Give the following passage early in the
documentation to GHC.IO a gander:
The IO Monad is just an instance of the ST monad,
where the state is the real world.
The motivation for these explanations is easy to understand
when you look at the underlying types:

CHAPTER 29. IO 1785
-- from ghc-prim
importGHC.Prim
importGHC.Types
newtype States a
=State{runState ::s->(a, s)}
-- :info IO
newtype IOa=
IO(State#RealWorld
->(#State#RealWorld , a#))
Yep, it sure looks like State! However, this is less meaningful
or helpful than you’d think at first.
The issue with this explanation is that you don’t usefully
see or interact with the underlying State#1inIO. It’s not State
in the sense that one uses State,StateT, or even ST, although
the behavior of the 𝑠here is certainly very like that of ST.
TheStatehere is a signalling mechanism for telling GHC
what order your IOactions are in and what a unique IOaction
is. If we look through the GHC.Prim documentation, we see:
State# is the primitive, unlifted type of states. It has
one type parameter, thus State# RealWorld, or State#
1The#indicates a primitive type. These are types that cannot be defined in Haskell
itself and are exported by the GHC.Prim module.

CHAPTER 29. IO 1786
𝑠, where 𝑠is a type variable. The only purpose of
the type parameter is to keep diﬀerent state threads
separate. It is represented by nothing at all.
RealWorld is deeply magical. It is primitive, but it
is not unlifted (hence ptrArg). We never manipulate
values of type RealWorld; it’s only used in the type
system, to parameterise State# .
When they say that RealWorld is “represented by nothing at
all,” they mean it literally uses zero bits of memory. The state
tokens underlying the IOtype are erased during compile time
and add no overhead to your runtime. So the problem with
explaining IOin terms of Stateis not precisely that it’s wrong;
it’s that it’s not a Stateyou can meaningfully interact with or
control in the way you’d expect from the other Statetypes.
29.3 The reason we need this type
So, let’s try to move from there to an understanding of IOthat
is meaningful to us in our day-to-day Haskelling. IOprimarily
exists to give us a way to order operations and to disable some
of the sharing that we talked so much about in the chapter on
nonstrictness.
GHC is ordinarily free to do a lot of reordering of opera-
tions, delaying of evaluation, sharing of named values, dupli-
cating code via inlining, and other optimizations in order to

CHAPTER 29. IO 1787
increase performance. The main thing the IOtype does is turn
oﬀ most of those abilities.
What?
No, really. That’s a lot of it.
Order and chaos
As we’ve seen in the previous chapters, GHC can normally
reorder operations. This is disabled in IO(as inST).IOactions
are instead enclosed within nested lambdas — nesting is the
only way to ensure that actions are sequenced within a pure
lambda calculus.
Nesting lambdas is how we guarantee that this
main= do
putStr "1"
putStr "2"
putStrLn "3"
will output “123” and we want that guarantee. The underly-
ing representation of IOallows the actions to be nested, and
therefore sequenced.
When we enter a lambda expression, any eﬀects that need to
beperformedwillbeperformedfirst, beforeanycomputations
are evaluated. Then if there is a computation to evaluate, that

CHAPTER 29. IO 1788
may be evaluated next, before we enter the next lambda to
perform the next eﬀect and so on. We’ve seen how this plays
out in previous chapters; think of the parsers that perform the
eﬀect of moving a “cursor” through the text without reducing
to any value; also recall what we saw with STand mutable
vectors.
In fact, the reason we have Monadis because it was a means
of abstracting away the nested lambda noise that underlies IO.
29.4 Sharing
In addition to enforcing ordering, IOturns oﬀ a lot of the shar-
ing we talked about in the nonstrictness chapter. As we’ll soon
see, it doesn’t disable all forms of sharing — it couldn’t, because
all Haskell programs have a mainaction with an obligatory IO
type. But we’ll get to that in a moment.
For now, let’s turn our attention to what sharing is disabled
and why. Usually in Haskell, we’re pretty confident that if a
function is going to be evaluated at all, it will result in a value
of a certain type, bearing in mind that this could be a Nothing
value or an empty list. When we declare the type, we say, “if
this is evaluated at all, we will have a value of this type as a
result.”
But with the IOtype, you’re not guaranteed anything. Values
oftypeIO aarenotan 𝑎; they’readescriptionofhowyoumight
get an𝑎. Something of type IO String is not a computation

CHAPTER 29. IO 1789
that, if evaluated, will result in a String; it’s a description of
how you might get that String from the “real world,” possibly
performing eﬀects along the way. Describing IOactions does
not perform them, just as having a recipe for a cake does not
give you a cake.2
In this environment, where you do not have a value but
only a means of getting a value, it wouldn’t make sense to say
that value could be shared.
The time has come
So, one of the key features of IOis that it turns oﬀ sharing.
Let’s use an example to think of why we want this. We have
this library function that gets the current UTC time from the
system clock:
-- from Data.Time.Clock
getCurrentTime ::IOUTCTime
Without IOpreventing sharing, how would this work? When
you fetched the time once, it would share that result, and the
time would be whatever time it was the first time you forced it.
Unfortunately, this is not a means of stopping time; we would
continue to age, but your program wouldn’t work at all the
way you’d intended.
2See Brent Yorgey’s explanation of IOfor the cis194 class at UPenn http://www.cis.
upenn.edu/~cis194/spring13/lectures/08-IO.html

CHAPTER 29. IO 1790
But if that’s so, and it’s clearly a value with a name that
could be shared, why isn’t it?
getCurrentTime ::IOUTCTime
-- ^-- that right there
Remember: what we have here is a description of how we
can get the current time when we need it. We do not have the
current time yet, so it isn’t a value that can be shared, and we
don’t want it to be shared anyway, because we want it to get a
new time each time we run it.
And the way we run it is by defining mainin that module
for the runtime system to find and execute. Everything inside
mainis within IOso that everything is nested and sequenced
and happens in the order you’re expecting.
Another example
Let’s look at another example of IOturning oﬀ sharing. You
remember the whnfandnffunctions from criterion that we
used in the last chapter. You may recall that we want to turn
oﬀ sharing for those so that they get evaluated over and over
again; if the result was shared, our benchmarking would only
tell us how long it takes to evaluate it once instead of giving us
an average of evaluating it many times. The way we disabled
sharing for those functions is by applying them to arguments.

CHAPTER 29. IO 1791
But the IOvariants of those functions do not require this
function application in order to disable sharing, because the
IOparameter itself disables sharing. Contrast the following
types:
whnf::(a->b)->a->Benchmarkable
nf::NFDatab
=>(a->b)->a->Benchmarkable
whnfIO::IOa->Benchmarkable
nfIO::NFDataa=>IOa->Benchmarkable
TheIOvariants don’t need a function argument to apply
because sharing is already prevented by being an IOaction —
it can be executed over and over without resorting to adding
an argument.
As we said earlier, IOdoesn’t turn oﬀ all sharing everywhere;
it couldn’t, or else sharing would be meaningless because main
is always in IO. But it’s important to understand when sharing
will be disabled and why, because if you’ve got this notion of
sharing running around in the back of your head you’ll have
the wrong intuitions for how Haskell code works. Which then
leads to…

CHAPTER 29. IO 1792
The code! It doesn’t work!
We’re going to use an example here that takes advantage of
theMVartype. This is based on a real code event that was how
Chris finally learned what IOmeans and the example he first
used to explain it to Julie.
TheMVartype is a means of synchronizing shared data in
Haskell. To give a very cursory overview, the MVarcan hold
one value at a time. You put a value into it; it holds onto it
until you take that value out. Then and only then can you
put another cat in the box.3We cannot hope to best Simon
Marlow’s work4on this front, so if you want more information
about it, we strongly recommend you peruse Marlow’s book.
OK, so we’ll set up some toy code here with the idea that
we want to put a value into an MVarand then take it back out:
3What you need is a cat gif. https://twitter.com/argumatronic/status/
631158432859488258
4Parallel & Concurrent Programming in Haskell http://chimera.labs.oreilly.com/
books/1230000000929

CHAPTER 29. IO 1793
moduleWhatHappens where
importControl.Concurrent
myData::IO(MVarInt)
myData=newEmptyMVar
main::IO()
main= do
mv<-myData
putMVar mv 0
mv'<-myData
zero<-takeMVar mv'
print zero
This will spew an error about being stuck or in a deadlock.
The problem here is that the type IO MVar a ofnewEmptyMVar is
a recipe for producing as many empty MVars as you need or
want; it is not a reference to a single, shared MVar. In other
words, the two references to myData here are not referring to
the same MVar.
Taking from an empty MVarblocks until something is put
into the MVar. Consider the following ordering:
take
put

CHAPTER 29. IO 1794
take
put
That will terminate successfully. An attempt to take a value
from the MVarblocked, then a value was put in it, then another
blocked take occurred, then there was another put to satisfy
the second take. This is fine.
The following is an example of something that will dead-
lock:
put
take
take
Whatever part of your program performed the second take
will now be blocked until a second put occurs. If your program
is designed such that no put ever occurs again, it’s deadlocked.
A deadlock error looks like the following:
Prelude> main
*** Exception:
thread blocked indefinitely
in an MVar operation
When you see a type like:
IOString

CHAPTER 29. IO 1795
You don’t have a String; you have a means of (possibly)
obtaining a String , with some eﬀects possibly performed along
the way. Similarly, what happened earlier is that we had two
MVars with two diﬀerent lifetimes and that looked something
like this:
mv mv'
put take (the final one)
The point here is that this type
IO(MVara)
tells you that you have a recipe for producing as many
empty MVars as you want, not a reference to a single shared
MVar.
You can share the MVar, but it has to be done explicitly rather
than implicitly. Failing to explicitly share the MVarreference
after binding it once will simply spew out new, empty MVars.
Again, we recommend Simon Marlow’s book when you’re
ready to explore MVars in more detail.
29.5 IO doesn’t disable sharing for
everything
As we mentioned earlier, IOdoesn’t disable sharing for every-
thing, and it wouldn’t make sense if it did. It only disables

CHAPTER 29. IO 1796
sharing for the terminal value it reduces to. Values that are
not dependent on IOfor their evaluation can still be shared,
even within a larger IOaction such as main.
In the following example, we’ll use Debug.Trace again to show
us when things are being shared. For blah, thetraceis outside
theIOaction, so we’ll use outer trace :
importDebug.Trace
blah::IOString
blah=return"blah"
blah'=trace"outer trace" blah
And for woot, we’ll use inner trace inside the IOaction:
woot::IOString
woot=return (trace "inner trace" "woot")
Then we throw both of them into a larger IOaction,main:

CHAPTER 29. IO 1797
main::IO()
main= do
b<-blah'
putStrLn b
putStrLn b
w<-woot
putStrLn w
putStrLn w
Prelude> main
outer trace
blah
blah
inner trace
woot
woot
We only saw inner and outer emitted once because IOis not
intended to disable sharing for values not in IOthat happen to
be used in the course of running of an IOaction.
29.6 Purity is losing meaning
It’s common at this time to use the words “purely functional”
or to talk about purity when one means without eﬀects. This is
inaccurate and not very useful as a definition, but we’re going

CHAPTER 29. IO 1798
to provide some context here and an alternative understand-
ing.
Semantically, pedantically accurate
Purity and “pure functional” have undergone a few changes in
connotation and denotation since the 1950s. What was origi-
nally meant when describing a pure functional programming
language is that the semantics of the language would only be
lambda calculus. For quite a long time, impure functional lan-
guages were more typical. They admitted the augmentation of
lambda calculus, usually so that the means to describe imper-
ative, eﬀectful programs was embedded within the semantics.
The strength of Haskell is that by sticking to lambda calculus,
we not only have a much simpler core language for describing
our language, but we retain referential transparency in the
language. We use nested lambdas (hidden behind a Monadab-
straction) to order and encapsulate eﬀects while maintaining
referential transparency.
Referential transparency
Referential transparency is something you are probably fa-
miliar with, even if you’ve never called it that before. Put
casually, it means that any function, when given the same in-
puts, returns the same result. More precisely, an expression

CHAPTER 29. IO 1799
is referentially transparent when it can be replaced with its
value without changing the behavior of a program.
One source of the confusion between purity as referential
transparency and purity as pure lambda calculus could be that
in a pure lambda calculus, referential transparency is assured.
Thus, a pure lambda calculus is necessarily pure in the other
sense as well.
The mistake people make with IOis that they conflate the
eﬀects with the semantics of the program. A function that
returns IO ais still referentially transparent, because given the
same arguments, it’ll generate the same IOaction every time!
To make this point:
moduleIORefTrans where
importControl.Monad (replicateM )
importSystem.Random (randomRIO )
gimmeShelter ::Bool->IO[Int]
gimmeShelter True=
replicateM 10(randomRIO ( 0,10))
gimmeShelter False=pure [0]
The trick here is to realize that while executing IO [Int] can
and does produce diﬀerent literal values when the argument
isTrue, it’s still producing the same result (i.e., a list of ran-

CHAPTER 29. IO 1800
dom numbers) for the same input. Referential transparency is
preserved because we’re still returning the same IO action, or
“recipe,” for the same argument, the same means of obtaining
a list of Int. Every Trueinput to this function will return a list
of random Ints:
Prelude> gimmeShelter True
[1,8,7,9,10,4,2,9,3,6]
Prelude> gimmeShelter True
[10,0,7,1,10,2,4,0,9,3]
Prelude> gimmeShelter False
[0]
The sense we’re trying to convey here is that as far as Haskell
is concerned, it’s a language for evaluating expressions and
constructing IOactions that get executed by mainat some point
later.
29.7 IO’s Functor, Applicative, and
Monad
Another mistake people make is in implying that IOis aMonad,
rather than accounting for the fact that, like all Monads,IOis
a datatype that has a Monadinstance — as well as Functor and
Applicative instances:

CHAPTER 29. IO 1801
fmap: construct an action which performs the same eﬀects
but transforms the 𝑎into a𝑏:
fmap::(a->b)->IOa->IOb
(<*>): construct an action that performs the eﬀects of both
the function and value arguments, applying the function to
the value:
(<*>)::IO(a->b)->IOa->IOb
join: merge the eﬀects of a nested IOaction:
join::IO(IOa)->IOa
The IO Functor
What does fmapmean with respect to IO? As always, we want
an example:
fmap(+1) (randomIO ::IOInt)
If we’re going to get that Intvalue, we will have to perform
some eﬀects. What fmapdoes here is lift our incrementing
function over the eﬀects that we might perform to obtain the
Intvalue. It doesn’t aﬀect the eﬀects, because the eﬀects here
are part of that IOstructure. Using fmaphere returns a recipe for
obtaining an Intthat also increments the result of the original
action that was lifted over.

CHAPTER 29. IO 1802
The key here is that we didn’t perform any eﬀects. We pro-
duced a new IOaction in terms of the old one by transforming
the final result of the IOaction.
Applicative and IO
IOalso has an Applicative instance, as we mentioned in the
Applicative chapter. You might remember an example like
this:
Prelude> (++) <$> getLine <*> getLine
hello
julie
"hellojulie"
There we fmapped the concatenation operator over two
(potential) IO String s to produce the final result. Let’s look at
another, more interesting example:
(+)
<$>(randomIO ::IOInt)
<*>(randomIO ::IOInt)
After the initial fmap, we have a means of obtaining a func-
tion which is monoidally lifted over a means of obtaining an
Int. What this means is that you’ll get a single new means
of obtaining the result of having applied the function which
performs the eﬀects of both.

CHAPTER 29. IO 1803
Monad and IO
ForIO,pureorreturn can be read as an eﬀect-free embedding
of a value in a recipe-creating environment. Let’s consider the
following examples.
First, GHCi does basically two things: it can print values
not inIO, such as these:
Prelude> "I'll pile on the candy"
"I'll pile on the candy"
Prelude> 1
1
It can also run IOactions and print their results, if any. When
you have values of type IO (IO a) , what you have is a recipe
for making a recipe that produces an 𝑎. Consider why the
following example using printdoes not print anything:
Prelude> :{
*Main| let embedInIO =
*Main| return :: a -> IO a
*Main| :}
Prelude> embedInIO 1
1
Prelude> :{
*Main| let s =
*Main| "I'll put in some ingredients"

CHAPTER 29. IO 1804
*Main| :}
Prelude> embedInIO (print s)
In order to merge those eﬀects and get a single IO awhich
will print a result in GHCi, we need join:
Prelude> let s = "It's a piece of cake"
Prelude> join $ embedInIO (print s)
"It's a piece of cake"
Prelude> embedInIO (embedInIO 1)
Prelude> join $ embedInIO (embedInIO 1)
1
What sets the IO Monad apart from the Applicative is that the
eﬀects performed by the outer IOaction can influence what
recipe you get in the inner part. The nesting also lets us express
order dependence, a useful trick for lambda calculi noted by
Peter J. Landin5.
An example for eﬀect:
5A correspondence between ALGOL 60 and Church’s Lambda-notations; P.J. Landin

CHAPTER 29. IO 1805
moduleNestedIO where
importData.Time.Calendar
importData.Time.Clock
importSystem.Random
huehue::IO(Either(IOInt) (IO()))
huehue= do
t<-getCurrentTime
let(_,_, dayOfMonth) =
toGregorian (utctDay t)
caseeven dayOfMonth of
True->
return$LeftrandomIO
False->
return$
Right(putStrLn "no soup for you" )
TheIOaction we return here is contingent on having per-
formed eﬀects and observed whether the day of the month
was an even number6or an odd one. Note this is inexpressible
withApplicative . If you’d like a way to run it and see what
happens, try the following:
Prelude> blah <- huehue
6Why? Monad chapter’s long passed, we need something to be spooky.

CHAPTER 29. IO 1806
Prelude> either (>>= print) id blah
-7077132465932290066
It was the 28th of January when we wrote this. Your mileage
may vary.
Monadic associativity
Haskellers will often get confused when they are told Monad’s
bind is associative because they’ll think of IOas a counterex-
ample. The mistake being made here is mistaking the con-
struction of IOactions for the execution of IOactions. As far
as Haskell is concerned, we only construct IOactions to be
executed when we call main. Semantically, IOactions aren’t
something we do, but something we talk about. Binding over
anIOaction doesn’t execute it, it produces a new IOaction in
terms of the old one.
You can reconcile yourself with this framing by remem-
bering how IOactions are like recipes, an analogy created by
Brent Yorgey that we’re fond of.
29.8 Well, then, how do we MVar?
Earlier in the chapter, we showed you an example of when
IOprevents sharing, using the MVartype. Our previous code
would block because the following:

CHAPTER 29. IO 1807
myData::IO(MVarInt)
myData=newEmptyMVar
is anIOaction that produces an empty MVar; it isn’t a stable
reference to a single given MVar. We have a couple ways of
fixing this. One is by passing the single stable reference as an
argument. The following will terminate successfully:
moduleWhatHappens where
importControl.Concurrent
main::IO()
main= do
mv<-newEmptyMVar
putMVar mv ( 0::Int)
zero<-takeMVar mv
print zero
There is a somewhat more evil and unnecessary way of
doing it. We’ll use this opportunity to examine an unsafe
means of enabling sharing for an IOaction: unsafePerformIO !
Consider that the following will also terminate:

CHAPTER 29. IO 1808
moduleWhatHappens where
importControl.Concurrent
importSystem.IO.Unsafe
myData::MVarInt
myData=unsafePerformIO newEmptyMVar
main::IO()
main= do
putMVar myData 0
zero<-takeMVar myData
print zero
The type of unsafePerformIO isIO a -> a , which is seemingly
impossible and not a good idea in general. In real code, you
should pass references to MVars as an argument or via ReaderT ,
but the combination of MVarandunsafePerformIO gives us an
opportunity to see in very stark terms what it means to use
unsafePerformIO in our code. The new empty MVarcan now be
shared implicitly, as often as you want, instead of creating a
new one each time.
Do not use unsafePerformIO when unnecessary or where it
could break referential transparency in your code! If you
aren’t sure — don’t use it! There are other unsafe IOfunctions,

CHAPTER 29. IO 1809
too, but there is rarely a need for any of them, and in general
you should prefer explicit rather than implicit.
29.9 Chapter Exercises
File I/O with Vigenère
ReusingtheVigenèrecipheryouwrotebackinalgebraicdatatypes
and wrote tests for in testing, make an executable that takes
a key and a mode argument. If the mode is -dthe executable
decrypts the input from standard in and writes the decrypted
text to standard out. If the mode is -ethe executable blocks
on input from standard input ( stdin) and writes the encrypted
output to stdout .
Consider this an opportunity to learn more about how file
handles and the following members of the baselibrary work:
System.Environment .getArgs ::IO[String]
System.IO.hPutStr
::Handle->String->IO()
System.IO.hGetChar ::Handle->IOChar
System.IO.stdout::Handle
System.IO.stdin::Handle
Whatever OS you’re on, you’ll need to learn how to feed
files as input to your utility and how to redirect standard out
to a file. Part of the exercise is figuring this out for yourself.

CHAPTER 29. IO 1810
You’ll want to use hGetChar more than once to accept a string
which is encrypted or decrypted.
Add timeouts to your utility
UsehWaitForInput to make your utility timeout if no input is
provided within a span of time of your choosing. You can
make it an optional command-line argument. Exit with a
nonzero error code and an error message printed to standard
error (stderr ) instead of stdout .
System.IO.hWaitForInput
::Handle->Int->IOBool
System.IO.stderr::Handle
Config directories
Reusing the INI parser from the Parsing chapter, parse a direc-
tory of INI config files into a Mapwhose key is the filename and
whose value is the result of parsing the INI file. Only parse
files in the directory that have the file extension .ini.
29.10 Follow-up resources
1.Referential Transparency; Haskell Wiki
https://wiki.haskell.org/Referential_transparency

CHAPTER 29. IO 1811
2.IO Inside; Haskell Wiki
https://wiki.haskell.org/IO_inside
3.Unraveling the mystery of the IO Monad; Edward Z. Yang
4.Primitive Haskell; Michael Snoyman
https://github.com/commercialhaskell/haskelldocumentation/
blob/master/content/primitive-haskell.md
5.Evaluation order and state tokens; Michael Snoyman
https://wiki.haskell.org/Evaluation_order_and_state_tokens
6.Haskell GHC Illustrated; Takenobu Tani
7.Tackling the Awkward Squad; Simon PEYTON JONES
http://research.microsoft.com/en-us/um/people/simonpj/papers/
marktoberdorf/mark.pdf
8.Note [IO hack in the demand analyser]; GHC source code
9.Monadic I/O in Haskell 1.3; Andrew D. Gordon and Kevin
Hammond
10.Notions of computation and monads; Eugenio Moggi
http://www.disi.unige.it/person/MoggiE/ftp/ic91.pdf
11.The Next 700 Programming Languages; P. J. Landin
12.Haskell Report 1.2

Chapter 30
When things go wrong
It is easier to write an
incorrect program than
understand a correct one
Alan Perlis
1812

CHAPTER 30. WHEN THINGS GO WRONG 1813
30.1 Exceptions
Let’s face it: in the execution of a program, a lot of things can
happen, not all of them expected or desired. In those unhappy
times when things have not gone as we wanted them to, we
will throw or raise an exception. The term exception refers
to the condition that has interrupted the expected execution
of the program. Encountering an exception causes an error,
or exception, message to appear, informing you that due to
some condition you weren’t prepared for, the execution of the
program has halted in an unfortunate way.
In previous chapters, we’ve covered ways of using Maybe,
Either , andValidation types to handle certain error conditions
explicitly. Raising exceptional conditions via such datatypes
isn’t always ideal, however. In some cases, exceptions can
be faster by eliding repeated checks for an adverse condition.
Exceptions are not explicitly part of the interfaces you’re using,
and that has immediate consequences when trying to reason
about the ways in which your program could fail.
Letting exceptions arise as they will — and the program
halt willy-nilly — is suboptimal. Exception handling is a way
of dealing with errors and giving the program some alternate
means of execution or termination should one arise. This
chapter is going to cover both exceptions and what they look
like as well as various means of handling them.
In this chapter, we will:

CHAPTER 30. WHEN THINGS GO WRONG 1814
•examine the Exception typeclass and methods;
•dip our toes into existential quantification;
•discuss ways of handling exceptions.
30.2 The Exception class and methods
Exceptions are plain old types and values like you’ve seen
throughout the book. The types that encode exceptions must
have an instance of the Exception typeclass. The origins of
exceptions as they exist in Haskell today are in Simon Mar-
low’s work on an extensible hierarchy of exceptions1which
are discriminated at runtime. Using this extensible hierarchy
allows you to both catch exceptions that may have various
types and also add new exception types as the need arises.
TheException typeclass definition looks like this:
class(Typeable e,Showe)=>
Exception ewhere
toException ::e->SomeException
fromException ::SomeException ->Maybee
displayException ::e->String
-- Defined in ‘GHC.Exception’
1http://community.haskell.org/~simonmar/papers/ext-exceptions.pdf

CHAPTER 30. WHEN THINGS GO WRONG 1815
We’ll take a look at those methods in a moment. The Show
constraint is there so that we can print the exception to the
screen in a readable form for whatever type 𝑒ends up being.
Typeable is a typeclass that defines methods of identifying types
at runtime. We will talk about this more and explain why these
constraints are necessary to our Exception class soon.
The list of types that have an Exception instance is long:
-- some instances elided
instance Exception IOException
instance Exception Deadlock
instance Exception BlockedIndefinitelyOnSTM
instance
Exception BlockedIndefinitelyOnMVar
instance Exception AsyncException
instance Exception AssertionFailed
instance Exception AllocationLimitExceeded
instance Exception SomeException
instance Exception ErrorCall
instance Exception ArithException
We won’t talk in detail about each of these, but you may be
able to figure out what, for example, BlockedIndefinitelyOnMVar
is used for. We’ll note that it’s simply a datatype with one
inhabitant:

CHAPTER 30. WHEN THINGS GO WRONG 1816
dataBlockedIndefinitelyOnMVar =
BlockedIndefinitelyOnMVar
-- Defined in ‘GHC.IO.Exception’
If we look at ArithException , we’ll find that it’s a sum type
with several values:
dataArithException
=Overflow
|Underflow
|LossOfPrecision
|DivideByZero
|Denormal
|RatioZeroDenominator
instance Exception ArithException
If you import the Control.Exception module, you can poke
atArithException ’s data constructors and see that they’re plain
old values, nothing unusual at all.
But there is something diﬀerent going on here
We’re going to start unpacking all this to see how the parts
work together. First, let’s take a look at the methods of the
Exception typeclass:

CHAPTER 30. WHEN THINGS GO WRONG 1817
toException ::e->SomeException
fromException ::SomeException ->Maybee
We don’t have much occasion to use the toException and
fromException functions themselves. Instead, we use other func-
tions that call them for us. As it turns out, the toException
methodisquitesimilartothedataconstructorfor SomeException .
You may have noticed that SomeException is also a type that is
listed as having an instance of the Exception typeclass, and now
here it is in the Exception methods. It seems a bit circular, but
it turns out that SomeException is ultimately the key to the way
we handle exceptions.
A brief introduction to existential quantification
SomeException acts as a sort of parent type for all the other ex-
ception types, so that we can handle many exception types at
once, without having to match all of them. Let’s examine how:
dataSomeException where
SomeException
::Exception e=>e->SomeException
This may not seem odd at first glance. That is due, in part,
to the fact that the weirdness is hiding in a construction called
a GADT, for generalized algebraic datatype. For the most

CHAPTER 30. WHEN THINGS GO WRONG 1818
part, GADTs are out of the scope of this book, being well
into intermediate Haskell territory that is fun to explore but
not strictly necessary to programming in Haskell. What the
GADT syntax is hiding there is something called existential
quantification.
We could rewrite the SomeException type like this without a
change in meaning:
dataSomeException =
forall e .Exception e=>SomeException e
Ordinarily, the forall quantifies variables universally, as
you might guess from the word all. However, the SomeException
type constructor doesn’t take an argument; the type variable
𝑒is a parameter of the data constructor. It takes an 𝑒and
results in a SomeException . Moving the quantifier to the data
constructor limits the scope of its application, and changes
the meaning from for all e to there exists some e. That is exis-
tential quantification. It means that any type that implements
theException class can be that 𝑒and be subsumed under the
SomeException type.
We aren’t going to examine existential quantification deeply
here; this is a mere taste. Usually when type constructors are
parameterized, they are universally quantified. Arguments
have to be supplied to satisfy them. Your Maybe a type is, as

CHAPTER 30. WHEN THINGS GO WRONG 1819
we’ve noted before, a sort of function waiting for an argument
to be supplied to be a fully realized type.
Butwhenweexistentiallyquantifyatype, aswith SomeException ,
we can’t do much with that polymorphic type variable in its
data constructor. We can’t concretize it. Other than adding
constraints, we can’t know anything about it. It must remain
polymorphic, and we can cram any value of any type that im-
plements its constraint into that role. It’s like a polymorphic
parasite just hanging out on your type.
So, any exception type — any type with an instance of
theException typeclass — can be that 𝑒and be handled as a
SomeException . We need Typeable andShowin order to determine
what type of exception we’re dealing with, as we will soon see.
So, wait, what?
For an example of what existential quantification lets us do,
we’re going to show you an example that doesn’t rely on the
magic of the runtime exception machinery. Here we’ll be
returning errors in Either of totally diﬀerent types without
having to unify them under a single sum type:

CHAPTER 30. WHEN THINGS GO WRONG 1820
{-# LANGUAGE ExistentialQuantification #-}
{-# LANGUAGE GADTs #-}
moduleWhySomeException where
importControl.Exception
(ArithException (..)
,AsyncException (..))
importData.Typeable
dataMyException =
forall e .
(Showe,Typeable e)=>MyException e
instance ShowMyException where
showsPrec p ( MyException e)=
showsPrec p e

CHAPTER 30. WHEN THINGS GO WRONG 1821
multiError ::Int
->EitherMyException Int
multiError n=
casenof
0->
Left(MyException DivideByZero )
1->
Left(MyException StackOverflow )
_ ->Rightn
What’s special about the above is that we have a Leftcase
in ourEither that includes error values of two totally diﬀerent
types without enumerating them in a sum type. MyException
doesn’t appear to have a polymorphic argument in the type
constructor, but it does in the data constructor. We are able
to apply the MyException data constructor to values of diﬀerent
types because of the existentially quantified type for 𝑒.
dataSomeError =
ArithArithException
|AsyncAsyncException
|SomethingElse
deriving (Show)

CHAPTER 30. WHEN THINGS GO WRONG 1822
discriminateError ::MyException
->SomeError
discriminateError (MyException e)=
casecast eof
(Justarith)->Aritharith
Nothing ->
casecast eof
(Justasync)->Asyncasync
Nothing ->SomethingElse
runDisc n=
either discriminateError
(constSomethingElse ) (multiError n)
Then trying this out:
Prelude> runDisc 0
Arith divide by zero
Prelude> runDisc 1
Async stack overflow
Prelude> runDisc 2
SomethingElse
This is the essence of why we need existential quantification
for exceptions — so that we can throw various exception types
without being forced to centralize and unify them under a
sum type. Don’t abuse this facility.

CHAPTER 30. WHEN THINGS GO WRONG 1823
Prior to this design, there were a few ways you could do
exception handling. Some of the more apparent methods
would’ve been one big sum type or strings. The problem is
that neither of them are meaningfully extensible to structured,
proper data types. We want, in a sense, a hierarchy of values
where catching a “parent” means catching any of the possible
“children.” The combination of SomeException and the Typeable
typeclass gives you a means of throwing diﬀerent exceptions
of diﬀerent types and then catching some or all of them in a
handler without wrapping them in a sum type.
Typeable
TheTypeable typeclasslivesinthe Data.Typeable module. Typeable
exists to permit types to be known at runtime, allowing for a
sort of dynamic typechecking. It allows you to learn the type
of a value at runtime and also to compare the types of two val-
ues and check that they are the same. Typeable is particularly
useful when you have code that needs to allow various types
to be passed to it but needs to enforce or trigger on specific
types.
This is ordinarily unwise, but it makes sense when you’re
talking about exceptions. When we’re concerned with excep-
tion handling, we want to be able to check whether values of
possibly varying types match the Exception type we’re trying to
handle, and we need to do that at runtime, when the exception

CHAPTER 30. WHEN THINGS GO WRONG 1824
occurs. Thus we need this runtime witness to the types of the
exceptions.
Let’s look at a method called cast, simplified from its im-
plementation in base:
cast::(Typeable a,Typeable b)
=>a->Maybeb
We don’t usually call this function directly, but it gets called
for us by the fromException function, and fromException is called
by thecatchfunction.
At runtime, when an exception is thrown, it starts rolling
back through the stack, looking for a catch. When it finds a
catch, it checks to see what type of exception this catchcatches.
It callsfromException andcastto check if the type of the excep-
tion that got thrown matches the type of an exception we’re
handling with the catch. Acatchthat handles a SomeException
will match any type of exception, due to the flexibility of that
type.
If they don’t match, we get a Nothing value; the exception
will keep rolling up through the stack, looking for a catchthat
can handle the exception that was thrown. If it doesn’t find
one, your program just dies an unseemly death.
If they do match, the Just a allows us to catch the exception.

CHAPTER 30. WHEN THINGS GO WRONG 1825
30.3 This machine kills programs
Exceptions can result from pure code:
Prelude> 2 `div` 0
*** Exception: divide by zero
However, running code is an I/O action (and GHCi is implic-
itly invoking IO), so most of the time when you need to worry
about exceptions, you’ll be in IO. Even when they happen in
pure code, exceptions may only be caught, or handled, in IO.
IOcontains the implicit contract, “You cannot expect this
computation to succeed unconditionally.” It turns out the
outside world is a harsh mistress — just about any IOaction
can fail, even putStrLn .
First, let’s demonstrate that any I/O action can fail. We will
assume that you do not currently have a file called aaain your
working directory. So, when you run this code, it will create
the file, write to it, print “wrote to file” in your terminal and
terminate successfully:
-- writePls.hs
moduleMainwhere
main= do
writeFile "aaa""hi"
putStrLn "wrote to file"

CHAPTER 30. WHEN THINGS GO WRONG 1826
You can fire up your REPL and load that, or you can compile
the binary like this (this is review, so if you already have all
this down, then go ahead and do it):
stack ghc -- <filename> -o <output file name>
And run it like this:
$ ./<output file name>
So, if you called the output file wp, for example, your termi-
nal session might look like this:
$ stack ghc -- writepls.hs -o wp
[stack compilation messages]
$ ./wp
wrote to file
$ cat aaa
hi
Cool, that all worked. That worked in part because writeFile
will go ahead and create a file and give it write permissions
if the file you’re trying to write to does not exist. But what if
you’re trying to write to a file that does already exist and does
not have write permissions?
Make a read-only file named zzzthat we can experiment
with. To make a file that cannot be written to on Linux or OSX,
the following suffices:

CHAPTER 30. WHEN THINGS GO WRONG 1827
$ touch zzz
$ chmod 400 zzz
Suppose that file cohabits a directory where we’re trying to
execute this program:
-- writePls.hs
moduleMainwhere
main= do
writeFile "zzz""hi"
putStrLn "wrote to file"
It’s the same program we had for the aaafile, just with the
file name changed. You can fire up your REPL and load that,
or you can compile the binary as we did above.
Then, if you run this program with such a file, you’ll get the
following result:
$ ./wp
wp: zzz: openFile: permission denied (Permission denied)
There’s a hole in our bucket, dear Liza: an exception.
Catch me if you can
Let’s fix that, dear Henry. We’ll start with some rudimentary
exception handling:

CHAPTER 30. WHEN THINGS GO WRONG 1828
moduleMainwhere
importControl.Exception
importData.Typeable
handler ::SomeException ->IO()
handler (SomeException e)= do
print (typeOf e)
putStrLn ( "We errored! It was: "
++show e)
main=
writeFile "zzz""hi"
`catch` handler
We’re still going to terminate without writing to the file, for
the same reasons as above. The program will run and termi-
nate successfully, but it’ll mention the error and say that it
failed with an IOException . We’ll get a bit more information
about why the program failed and be able to log that informa-
tion with our exception handler if we wish. Sometimes, that’s
exactly what you want: for your program to log the exception
and then die. Soon, we’ll look at some other options for han-
dling exceptions in a way that lets your program proceed with
an alternate execution.

CHAPTER 30. WHEN THINGS GO WRONG 1829
For now, let’s turn our attention to catch:
catch::Exception e
=>IOa
->(e->IOa)
->IOa
You may recall we mentioned catchearlier because it calls
fromException andcastfor us. It runs only if the exception
matching the type you specified gets thrown, and it gives you
an opportunitity to recover from the error and still satisfy
the original type that your IOaction purported to be. If no
exception gets thrown, then nothing happens with that 𝑒and
theIO aat the front is the same as the IO aat the end.
Let’s expand our rudimentary error handling in a way that
allows the program an alternate execution method instead
of allowing it to die. This time, the mainaction still wants to
write to that read-only file, but this time our handler gives
it an alternate file that does not exist to write to (if you do
have a file called bbbin your present working directory, you
can change the name of the writeFile argument to some other
name, anything as long as it doesn’t exist in your directory
yet):

CHAPTER 30. WHEN THINGS GO WRONG 1830
-- writePls.hs
moduleMainwhere
importControl.Exception
importData.Typeable
handler ::SomeException ->IO()
handler (SomeException e)= do
putStrLn ( "Running main caused an error! \
\It was: "
++show e)
writeFile "bbb""hi"
main=
writeFile "zzz""hi"
`catch` handler
When writing to zzzfails, it should print the error message
to the terminal. If you check your directory, you should see
your alternate file, named in the handler function, and if you
look inside that, it should say “hi” to you.
Let’s look at another, slightly more complex, use of catch.
This is taken from a program that deletes things from a Twitter
account and relies on the library twitter-conduit .2This portion
2https://www.stackage.org/package/twitter-conduit

CHAPTER 30. WHEN THINGS GO WRONG 1831
of the program can fail when it doesn’t have access to the
appropriate credentials for talking to a Twitter account. So,
we built an exception handler that tells it what to do when that
exception arises:
withCredentials action= do
twinfo<-
loadCredentials `catch` handleMissing
casetwinfoof
Nothing ->
getTWInfo >>=saveCredentials
Justtwinfo->action twinfo
wherehandleMissing ::IOException
->IO(MaybeTWInfo)
handleMissing _ =returnNothing
We turn an IOException into an IO (Maybe a) so we can case
on the Maybeto tell it what to do in the Nothing case. In this
case, if we throw an IOException and return a Nothing value, our
program will execute this:
getTWInfo >>=saveCredentials
By saving the credentials (the code that does the saving is
not shown here), we hopefully won’t encounter this exception
the next time we try to run it. In which case, we perform the

CHAPTER 30. WHEN THINGS GO WRONG 1832
action that is named in the Just twinfo line (said action is also
not shown here, sorry!).
30.4 Want either? Try!
Sometimes we’d like to lift exceptions out into explicit Either
values. This is quite doable, but you can’t erase the fact that
you performed I/O in the process. It’s also no guarantee you’ll
catch all exceptions. Here’s the function we need to turn im-
plicit exceptions into an explicit Either :
-- Control.Exception
try::Exception e
=>IOa
->IO(Eithere a)
Then to use it, we could write something like the following
code (please note, this will not compile to a binary the way
earlier examples did because it is not a Mainexecutable; use
GHCi):

CHAPTER 30. WHEN THINGS GO WRONG 1833
moduleTryExcept where
importControl.Exception
willIFail ::Integer
->IO(EitherArithException ())
willIFail denom=
try$print$div5denom
Here we print the result because you can only handle ex-
ceptions in IO, evidenced by the types of tryandcatch. If you
feed this some inputs, you’ll see something like the following:
Prelude> willIFail 1
5
Right ()
Prelude> willIFail 0
Left divide by zero
One thing to keep in mind is that exceptions in Haskell are
like exceptions in most other programming languages — they
are imprecise. An exception not caught by a particular bit of
code will get rolled up by the exception until it’s either caught
or kills your program.
If you wanted to get rid of the Right () that it’s printing in
the successful cases, here’s one way to get rid of it:

CHAPTER 30. WHEN THINGS GO WRONG 1834
onlyReportError ::Showe
=>IO(Eithere a)
->IO()
onlyReportError action= do
result<-action
caseresultof
Lefte->print e
Right_ ->return()
willFail ::Integer ->IO()
willFail denom=
onlyReportError $willIFail denom
Or you could use catch:
willIFail' ::Integer ->IO()
willIFail' denom=
print (div 5denom) `catch` handler
wherehandler ::ArithException
->IO()
handler e =print e
Let’s expand on this. We want to take the above examples
and turn them into an executable binary, which is a problem,
because in an executable, maincan’t take arguments. So, we’ll
have to do some serious modification in order to be able to

CHAPTER 30. WHEN THINGS GO WRONG 1835
pass arguments to mainwhen we call it. We’re going to import
System.Environment so that we can make use of a function called
getArgs that allows us to pass arguments in at the point where
we callmain:
moduleMainwhere
importControl.Exception
importSystem.Environment (getArgs)
willIFail ::Integer
->IO(EitherArithException ())
willIFail denom=
try$print$div5denom
onlyReportError ::Showe
=>IO(Eithere a)
->IO()
onlyReportError action= do
result<-action
caseresultof
Lefte->print e
Right_ ->return()

CHAPTER 30. WHEN THINGS GO WRONG 1836
testDiv ::String->IO()
testDiv d=
onlyReportError $willIFail (read d)
main::IO()
main= do
args<-getArgs
mapM_ testDiv args
The use of mapM_here might not be obvious, so let’s unpack
that a bit. It is essentially a less general traverse function that
throws away its end result and only produces the eﬀects. In this
case, those eﬀects are going to be the results of mapping our
testDiv function over a list of arguments — returning either
the result of a successful division or the type of an exception.
We’ll compile this one to an executable binary again, as we
did earlier in the chapter. To pass in the arguments, it will
look like this:
$ stack ghc -- writepls.hs -o wp
[stack noise]
$ ./wp 4 5 0 9
1
1
divide by zero
0

CHAPTER 30. WHEN THINGS GO WRONG 1837
IncaseyouwantedtotrythisintheREPL,reproducingwhat
you did above, use the indexmain@ :mainGHCi command and
pass the same arguments.
Prelude> :main 4 5 0 9
1
1
divide by zero
0
Notice that, now that the exception is handled, we can still
get that last result — we have survived an ArithException !
30.5 The unbearable imprecision of
trying
Let’s do another little experiment:
importControl.Exception
canICatch ::Exception e
=>e
->IO(EitherArithException ())
canICatch e=
try$throwIO e

CHAPTER 30. WHEN THINGS GO WRONG 1838
The new thing here is throwIO , a function that allows you
to throw an exception. Right now we want to demonstrate
that this handler doesn’t catch all types of exceptions, so we’re
usingthrowIO to cause exceptions of various types to be thrown.
TheLefthere can only handle or catch an ArithException ,
not any other kind. So when we throw a diﬀerent type of
exception, we get the following:
Prelude> canICatch DivideByZero
Left divide by zero
Prelude> canICatch StackOverflow
*** Exception: stack overflow
Prelude> :t DivideByZero
DivideByZero :: ArithException
Prelude> :t StackOverflow
StackOverflow :: AsyncException
The latter case blew past our trybecause we were trying to
catch an ArithException , not an AsyncException .
We’vementionedseveraltimesthat SomeException willmatch
on all types that implement the Exception typeclass, so try
rewriting the above such that the StackOverflow or any other
exception can also be caught.
We’ll continue the experiment by making a program that
runs until an unhandled exception stops the party:

CHAPTER 30. WHEN THINGS GO WRONG 1839
moduleStoppingTheParty where
importControl.Concurrent (threadDelay )
importControl.Exception
importControl.Monad (forever)
importSystem.Random (randomRIO )
randomException ::IO()
randomException = do
i<-randomRIO ( 1,10::Int)
ifi `elem` [ 1..9]
thenthrowIO DivideByZero
elsethrowIO StackOverflow
main::IO()
main=forever $ do
lettryS::IO()
->IO(EitherArithException ())
tryS=try
_ <-tryS randomException
putStrLn "Live to loop another day!"
-- microseconds
threadDelay ( 1*1000000)
We’ve talked about forever before; it causes the program
execution to loop indefinitely. We have added the threadDelay

CHAPTER 30. WHEN THINGS GO WRONG 1840
to slow the looping down so that what’s happening is more
noticeable. Note that the thread is delayed by a number of
microseconds.
ThetrySallows it to survive the ArithException s. We throw
away those exceptions and keep looping, but we can only throw
away the exception that we matched on ( ArithException ). At
some point, when our random number is 10, we will throw an
AsyncException instead of an ArithException , and our program
will die a rapid death. Try modifying this one so that both
exceptions are handled and the loop never terminates.
30.6 Why throwIO?
It may have seemed odd to you (or not!) to encounter throwIO
above. Why do we want to stop a program by purposely throw-
ing an exception? In the real world, we often do want to do
that — to stop the program when some condition occurs, but
it may be difficult to see that from what we’ve shown you so
far.
There’s a function called throwthat allows exceptions, such
as the arithmetic exceptions, but you rarely use it. It’s what
allows the divfunction to throw a DivideByZero exception when
that happens, but outside of such library functions, you don’t
need it.
The diﬀerence between throwandthrowIO can be seen in the
type:

CHAPTER 30. WHEN THINGS GO WRONG 1841
throwIO ::Exception e=>e->IOa
Partiality in the form of throwing an exception can be
thought of as an eﬀect. The conventional way to throw an
exception is to use throwIO , which has IOin its result. This is
the same thing as throw, butthrowIO embeds the exception in
IO. You always handle exceptions in IO3. Handling exceptions
must be done in IOeven if they were thrown without an IOtype.
You almost never want throwas it throws exceptions without
any warning in the type, even IO.
We’ll look at an example of an unconditionally thrown ex-
ception in IOso you can see how it aﬀects the control flow of
your program:
importControl.Exception
main::IO()
main= do
throwIO DivideByZero
putStrLn "lol"
Prelude> main
*** Exception: divide by zero
Likethrow,throwIO is often called for us, behind the scenes,
by library functions. Often, in interacting with the real world,
3Why? Because catching and handling exceptions means you could produce diﬀerent
results from the same inputs. That breaks referential transparency.

CHAPTER 30. WHEN THINGS GO WRONG 1842
we need to tell our program that in certain conditions, we want
it to stop or to give us an error message and let us know things
went wrong. We’ll take a look at a couple of examples from
real code, a library called http-client4by Michael Snoyman,
that uses throwIO to throw some exceptions when httpthings
haven’t gone the way we wanted them to:
connectionReadLine ::Connection
->IOByteString
connectionReadLine conn= do
bs<-connectionRead conn
when (S.null bs) $
throwIO IncompleteHeaders
connectionReadLineWith conn bs
In the above, throwIO will throw an IncompleteHeaders excep-
tion when the ByteString header is empty. In the next example,
it’s used to throw a ResponseTimeout exception when, well, the
response times out:
4https://www.stackage.org/package/http-client

CHAPTER 30. WHEN THINGS GO WRONG 1843
parseStatusHeaders ::Connection
->MaybeInt
->Maybe(IO())
->IOStatusHeaders
parseStatusHeaders conn timeout' cont
|Justk<-cont=
getStatusExpectContinue k
|otherwise =
getStatus
where
withTimeout = casetimeout' of
Nothing ->id
Justt->
timeout t >=>
maybe
(throwIO ResponseTimeout )
return
-- ... other code elided ...
You can use http-client without worrying about how he
makes the exceptions happen. But let’s next take a look at
making our own exception types for those times when you
do need to worry about it. Keep in mind that since time of
writing, http-client has changed how it defines and throws
exceptions, but the examples should still be useful.

CHAPTER 30. WHEN THINGS GO WRONG 1844
30.7 Making our own exception types
Often we’ll want our own exception types, like http-client has.
They enable us to be more precise about what’s going on in
our program. Let’s work through a small example to emit one
of a couple diﬀerent possible errors in an otherwise simple
function to see how we could do this:
moduleOurExceptions where
importControl.Exception
dataNotDivThree =
NotDivThree
deriving (Eq,Show)
instance Exception NotDivThree
dataNotEven =
NotEven
deriving (Eq,Show)
instance Exception NotEven
Note here that Exception instances are derivable — you don’t
need to write an instance. Continuing on:

CHAPTER 30. WHEN THINGS GO WRONG 1845
evenAndThreeDiv ::Int->IOInt
evenAndThreeDiv i
|rem i3/=0=throwIO NotDivThree
|odd i=throwIO NotEven
|otherwise =return i
Then we’ll see the error and success conditions:
*OurExceptions> evenAndThreeDiv 0
0
*OurExceptions> evenAndThreeDiv 1
*** Exception: NotDivThree
*OurExceptions> evenAndThreeDiv 2
*** Exception: NotDivThree
*OurExceptions> evenAndThreeDiv 3
*** Exception: NotEven
*OurExceptions> evenAndThreeDiv 6
6
*OurExceptions> evenAndThreeDiv 9
*** Exception: NotEven
*OurExceptions> evenAndThreeDiv 12
12
There is an issue with this setup, although it’s common.
What if we want to know what input or inputs caused the
error? We need to add context!

CHAPTER 30. WHEN THINGS GO WRONG 1846
Adding context
Convenient subsection titling! Anyhow, let’s modify that:
moduleOurExceptions where
importControl.Exception
dataNotDivThree =
NotDivThree Int
deriving (Eq,Show)
instance Exception NotDivThree
dataNotEven =
NotEven Int
deriving (Eq,Show)
instance Exception NotEven
evenAndThreeDiv ::Int->IOInt
evenAndThreeDiv i
|rem i3/=0=throwIO ( NotDivThree i)
|odd i=throwIO ( NotEven i)
|otherwise =return i
Now when we get errors, we can know what input caused
the error:

CHAPTER 30. WHEN THINGS GO WRONG 1847
*OurExceptions> evenAndThreeDiv 12
12
*OurExceptions> evenAndThreeDiv 9
*** Exception: NotEven 9
*OurExceptions> evenAndThreeDiv 8
*** Exception: NotDivThree 8
*OurExceptions> evenAndThreeDiv 3
*** Exception: NotEven 3
*OurExceptions> evenAndThreeDiv 2
Catch one, catch all
Now, you can probably figure out how to catch these two dif-
ferent errors:
catchNotDivThree ::IOInt
->(NotDivThree ->IOInt)
->IOInt
catchNotDivThree =catch
catchNotEven ::IOInt
->(NotEven ->IOInt)
->IOInt
catchNotEven =catch
Or perhaps with try:

CHAPTER 30. WHEN THINGS GO WRONG 1848
Prelude> type EA e = IO (Either e Int)
Prelude> try (evenAndThreeDiv 2) :: EA NotEven
*** Exception: NotDivThree 2
Prelude> try (evenAndThreeDiv 2) :: EA NotDivThree
Left (NotDivThree 2)
Thetypesynonymisn’tsemanticallyimportant, butitshrinks
the noise a bit. Now, you could handle both errors with the
catches function:
catches ::IOa->[Handler a]->IOa
catchBoth ::IOInt
->IOInt
catchBoth ioInt=
catches ioInt
[Handler
(\(NotEven _)->return maxBound)
,Handler
(\(NotDivThree _)->return minBound)
]
ThemaxBound /minBound thingisnotgoodcodeforrealuse, just
a convenience. Incidentally, the same trick the SomeException
type uses to hide type arguments is used by the Handler type
to wrap the values in the list of exception handlers: existential
quantification.

CHAPTER 30. WHEN THINGS GO WRONG 1849
dataHandler awhere
Handler ::Exception e
=>(e->IOa)->Handler a
-- Defined in ‘Control.Exception’
We can make a list of handlers that handle exceptions of
varying types because the exception types are existentially
quantified under Handler ’s datatype.
But what if this isn’t convenient enough? What if we have a
family of semantically related or otherwise similar exceptions
we want to catch as a group? For this we revive our old friend,
the sum type!

CHAPTER 30. WHEN THINGS GO WRONG 1850
moduleOurExceptions where
importControl.Exception
dataEATD=
NotEven Int
|NotDivThree Int
deriving (Eq,Show)
instance Exception EATD
evenAndThreeDiv ::Int->IOInt
evenAndThreeDiv i
|rem i3/=0=throwIO ( NotDivThree i)
|even i=throwIO ( NotEven i)
|otherwise =return i
Now when we want to catch either error, we only need one
handler and then we can pattern match on the exception type
just like good old fashioned datatypes:
Prelude> type EA e = IO (Either e Int)
Prelude> try (evenAndThreeDiv 0) :: EA EATD
Left (NotEven 0)
Prelude> try (evenAndThreeDiv 1) :: EA EATD
Left (NotDivThree 1)

CHAPTER 30. WHEN THINGS GO WRONG 1851
Nifty, eh? The notion here is to exercise the same taste
and judgment in designing your error types as you would in
your happy-path types. Preserve context and try to make it so
somebody could understand the problem you’re solving from
the types. If necessary. On a desert island. With a lot of rum.
And sea turtles.
30.8 Surprising interaction with
bottom
One thing to watch out for is situations where you catch an
exception for a value that might be bottom. Due to nonstrict-
ness, the bottom could’ve been forced before or after your
exception handler, so you might be surprised if you expected
either:
•that your exception handler was meant to catch the bot-
tom, or
•that no bottoms would cause your program to fail after
having caught, say, a SomeException .
The proper coping mechanism for this is a glass of scotch
and to realize the following things:
•The exception handling mechanism is not for, nor should
be used for, catching bottoms.

CHAPTER 30. WHEN THINGS GO WRONG 1852
•Having caught an exception, even SomeException , without
rethrowing an exception doesn’t mean your program
won’t fail.
To demonstrate the point, we’ll show you a case where we
caught an exception from a bottom and a case where a bottom
leap-frogged our handler:
importControl.Exception
noWhammies ::IO(EitherSomeException ())
noWhammies =
try undefined
megaButtums ::IO(EitherSomeException ())
megaButtums =
try$return undefined
Do you think these should have the same result? We’ve got
bad news:
Prelude> noWhammies
Left Prelude.undefined
Prelude> megaButtums
Right *** Exception: Prelude.undefined
The issue is that nonstrictness means burying the bottom
in areturn causes the bottom to not get forced until you’re

CHAPTER 30. WHEN THINGS GO WRONG 1853
already past the try, resulting in an uncaught error inside the
Rightconstructor. The take-away here shouldn’t be, “laziness
is terrifying,” but rather, “write total programs that don’t use
bottom.” It’s not only unforced bottoms that can cause pro-
grams that shouldn’t have any uncaught exceptions to fail
either, there’s also…
30.9 Asynchronous Exceptions
Asynchronousexceptionsarethepredatorshuntingyourhappy
little programs. You probably don’t have much experience
with anything like this unless you’ve written Erlang before.
Even then, Erlang’s asynchronous exceptions are handled by
a separate process. Most languages don’t have anything like
this if only because they don’t have a hope of making it safe
within their implementation runtimes.
moduleMainwhere
-- we haven't explained this.
-- tough cookies.
importControl.Concurrent
(forkIO,threadDelay )
importControl.Exception
importSystem.IO

CHAPTER 30. WHEN THINGS GO WRONG 1854
openAndWrite ::IO()
openAndWrite = do
h<-openFile "test.dat" WriteMode
threadDelay 1500
hPutStr h
(replicate 100000000 '0'++"abc")
hClose h
dataPleaseDie =
PleaseDie
deriving Show
instance Exception PleaseDie
main::IO()
main= do
threadId <-forkIO openAndWrite
threadDelay 1000
throwTo threadId PleaseDie
If you run this program, the intended result is that you’ll
have a file named test.dat with only zeroes that didn’t reach
the “abc” at the end. Since we can’t predict the future, if you
have a disk with preternaturally fast I/O, increase the argu-
ments to replicate to reproduce the intended issue. If it ain’t
broken, break it.

CHAPTER 30. WHEN THINGS GO WRONG 1855
What happened was that we threw an asynchronous excep-
tion from the main thread to our child thread, short-circuiting
what we were doing in the middle of doing it. If you did this
in a loop, you’d leak file handles, too. Done continually over a
period of time, leaking file handles can cause your process to
get killed or your computer to become unstable.
We can think of asynchronous exceptions as exceptions
raised from a diﬀerent thread than the one that’ll receive the
error. They’re immensely useful and give us a means of talk-
ing about error conditions that are quite real and possible in
languages that don’t have formal asynchronous exceptions.
Your process can get axe-murdered by the operating system
out of nowhere in any language. We just happen to have the
ability to do the same within the programming language at the
thread level as well. The issue is that we want to temporarily
ignore exceptions until we’ve finished what we’re doing. This
is so the state of the file is correct but also so that we don’t leak
resources like file handles or perhaps database connections or
something similar.5Never fear, we can fix this!
5In this case, leaking means having too many (files, database connections, etc.) open
at one time, thus consuming all the resources your OS can allocate, the way trying to
hold too much in memory for too long causes memory leaks.

CHAPTER 30. WHEN THINGS GO WRONG 1856
moduleMainwhere
-- we haven't explained this.
-- tough cookies.
importControl.Concurrent
(forkIO,threadDelay )
importControl.Exception
importSystem.IO
openAndWrite ::IO()
openAndWrite = do
h<-openFile "test.dat" AppendMode
threadDelay 1500
hPutStr h
(replicate 10000000 '0'++"abc")
hClose h
dataPleaseDie =
PleaseDie
deriving Show
instance Exception PleaseDie
main::IO()
main= do
threadId <-forkIO (mask_ openAndWrite)
threadDelay 1000
throwTo threadId PleaseDie

CHAPTER 30. WHEN THINGS GO WRONG 1857
Here we used mask_fromControl.Exception in order to mask
or delay exceptions thrown to our child thread until the IO
actionopenAndWrite wascomplete. Incidentally, sincetheendof
the mask is the last thing our child thread does, the exception
our main thread tried to throw to the child blows up in its
face, Wile E. Coyote style, and is now thrown within the main
thread.
Don’t panic!
Async exceptions are helpful and manifest in less obvious ways
in other language runtimes and ecosystems. Don’t try to catch
everything; just let it die, and make sure you have a process
supervisor and good logs. No execution is better than bad
execution.
30.10 Follow-up Reading
1.ABeginner’sGuidetoExceptionsinHaskell; ErinSwenson-
Healey
https://www.youtube.com/watch?v=PWS0Whf6-wc
2.Chapter 8. Overlapping Input/Output; Parallel and Con-
current Programming in Haskell; Simon Marlow;
http://chimera.labs.oreilly.com/books/1230000000929/ch08.html

CHAPTER 30. WHEN THINGS GO WRONG 1858
3.Chapter 9. Cancellation and Timeouts; Parallel and Con-
current Programming in Haskell; Simon Marlow;
http://chimera.labs.oreilly.com/books/1230000000929/ch09.html
4.An Extensible Dynamically-Typed Hierarchy of Excep-
tions; Simon Marlow
http://community.haskell.org/~simonmar/papers/ext-exceptions.
pdf

Chapter 31
Final project
1859

CHAPTER 31. FINAL PROJECT 1860
31.1 Final project
For our final project, we’re doing something a little weird, but
small and modernized a bit from the original design. Surely
no one who knows us from Twitter or IRC will be surprised
that we’ve chosen something eccentric for this, but we felt it
was important to show you an end-to-end project that brings
in so much real world it’ll make your head spin.
In this chapter,
•FINGER DAEMONS.
31.2 fingerd
Dating back to 1971, the finger1service was a means of figuring
out how to contact colleagues or other people on the same
computer network and whether they were on the network at
a given time, often on the same mainframe in a time when
computing was usually time-shared on the same physical ma-
chine.finger was originally intended to be used to share an
office number, email address, basic contact details like that.
By the time the 1990s and public internet access was widely
available finger was also used to deliver .planor.project files
as sort of pre-Twitter/Tumblr microblog.
1http://www.rajivshah.com/Case_Studies/Finger/Finger.htm

CHAPTER 31. FINAL PROJECT 1861
We’re going to be writing a finger daemon in this chapter.
Finger daemon programs are often called fingerd. A daemon
is a process that runs in the background without direct user
interaction; in the case of finger , the daemon acts as the server
side of the protocol, while the finger program itself is on the
client side. When you use finger from your command line,
it sends a request to the finger daemon, and the daemon re-
sponds with the requested information if it can.
We use this as an example in part because it’s not a typical
web app, only requires working with text, and because the
text-based protocol is spare and easy to debug once you know
how. This chapter is going to be somewhat more Unix/Linux-
oriented than previous ones, for a few reasons. Windows users
will find that not all of the examples can be followed along
literally, but the final version of the finger daemon2should
work.
Caveat for the Windows users
You will not be able to follow all of the instructions here ver-
batim. You can still build and hack on the project, but if you
aren’t willing to install a finger client for testing your finger
daemon via Cygwin then you’ll need to write your own client.
2A daemon is a computer program that runs as a background process

CHAPTER 31. FINAL PROJECT 1862
31.3 Exploring finger
If you had fingerd running on your local machine under the
username callen, the result of having done so might look some-
thing like:3
$ finger callen@localhost
Login: callen Name: callen
Directory: /home/callen Shell: /bin/zsh
On OS X, this will work, without having fired up or installed
a finger service, by not specifying a hostname to query:
$ finger callen
Login: callen Name: Chris Allen
Directory: /Users/callen Shell: /bin/bash
Spooky! Don’t ask. The finger protocol operates over Trans-
mission Control Protocol (TCP) sockets, something it has in
common with the protocol used by web browsers. However,
while they both use TCP, a finger daemon is not a web server.
It’s something much simpler. Rather than having an entire
application protocol layered atop TCP like the web (HTTP)
does, it’s a single message text protocol. Rather than go into a
long explanation of the internet, UDP, and TCP, let’s say TCP
3You can still use finger to check on the status of the bathrooms in the Random Hall
dormitory at MIT by typing finger @bathroom.mit.edu in your terminal. Try it.

CHAPTER 31. FINAL PROJECT 1863
is a protocol for sending messages back and forth between a
client and a server. Those messages can be raw bytes or text.
A socket is an address where a message can be delivered.4
Leaving aside the socket business, the way this should work
is roughly like this: the client requests some information, and
that request is trasmitted to the server with TCP magic. The
server (our friendly daemon) dishes up that information (if it
has it), TCP magic sends it to the client, then the client prints
the information in your terminal. We will start our project
with a little TCP echo server that prints the literal text the
client sent so that we can understand the cases we’re dealing
with.
Project overview
To kick this oﬀ, we’ll use Stack with the stack new command
like so:
$ stack new fingerd simple
This gets us a simple project with a single executable stanza
in the Cabal file. The final version after we’ve added Debug.hs
will have the following layout:
$ tree .
4If you’re new to networking and sockets, this guide by Julia Evans is a great intro-
duction. http://jvns.ca/zines/#networking-ack

CHAPTER 31. FINAL PROJECT 1864
.
├── LICENSE
├── Setup.hs
├── fingerd.cabal
├── src
│   ├── Debug.hs
│   └── Main.hs
└── stack.yaml
fingerd.cabal
Our Cabal file will mention an executable we’re not going to
give you yet, so you can leave the placeholder Stack generated
there for now. Note we have gently reformatted the text to fit
this book’s format.
name: fingerd
version: 0.1.0.0
synopsis: Simple project template
description: Please see README.md
homepage: https://github.com/u/fingerd
license: BSD3
license-file: LICENSE
author: Chris Allen
maintainer: cma@bitemyapp.com
copyright: 2016, Chris Allen
category: Web

CHAPTER 31. FINAL PROJECT 1865
build-type: Simple
cabal-version: >=1.10
executable debug
ghc-options: -Wall
hs-source-dirs: src
main-is: Debug.hs
default-language: Haskell2010
build-depends: base >= 4.7 && < 5
, network
executable fingerd
ghc-options: -Wall
hs-source-dirs: src
main-is: Main.hs
default-language: Haskell2010
build-depends: base >= 4.7 && < 5
, bytestring
, network
, raw-strings-qq
, sqlite-simple
, text
Now that we have taken care of that, let’s write some code.

CHAPTER 31. FINAL PROJECT 1866
src/Debug.hs
This is our first source file. We’re going to use this program to
show us what the client sends and send it back. In this respect,
it’s almost identical to the echo server demonstrated in the
documentation of the network5library we’re relying on. The
diﬀerence is that it also prints a literal representation of the
text that was sent.
Our debug program is a TCP server, similar to a web server
which provides a web page, but lower level and limited to
sending raw text back and forth. What is diﬀerent is that a web
server communicates with browsers over a TCP socket using a
structured protocol rich with metadata, routes, and a standard
describing that protocol. What we’re doing is older and more
primitive.
moduleMainwhere
importControl.Monad (forever)
importNetwork.Socket hiding(recv)
importNetwork.Socket.ByteString
(recv,sendAll)
5https://www.stackage.org/package/network The example we’re referring to is in the Net-
work.Socket.ByteString module. Click on it and look for the example.

CHAPTER 31. FINAL PROJECT 1867
logAndEcho ::Socket->IO()
logAndEcho sock=forever $ do
(soc,_)<-accept sock
printAndKickback soc
sClose soc
whereprintAndKickback conn = do
msg<-recv conn 1024
print msg
sendAll conn msg
This sets up our server. Its argument is a socket ( sock) that
listens for new client connections; due to our use of forever ,
that socket remains open indefinitely. The accept action will
block until a client connects to the server. The socket socis
the result of accept -ing a connection for communicating with
the client.
The server can receive up to 1024 bytes of text from the
client. All it does here is print the text literally, then echo
what the client sent right back to the client that made the
connection. Note that recvis permitted to return fewer than
the maximum bytes specified if that’s all the client sent. Then
the connection to the client is closed — we apply sClose to
socbut not to sock, sosock, the server socket, remains open.
Because this action loops forever, the next thing we do is await
another client connection.

CHAPTER 31. FINAL PROJECT 1868
main::IO()
main=withSocketsDo $ do
addrinfos <-getAddrInfo
(Just(defaultHints
{addrFlags =
[AI_PASSIVE ]}))
Nothing (Just"79")
letserveraddr =head addrinfos
sock<-socket (addrFamily serveraddr)
StreamdefaultProtocol
bindSocket sock (addrAddress serveraddr)
listen sock 1
logAndEcho sock
sClose sock
At the beginning of main,withSocketsDo is not going to do any-
thing at all unless you’re on Windows. If you are on Windows,
it’s obligatory to use the sockets API in the network library. The
address information stuﬀ is mostly noise and can be ignored
as a means for describing what kind of TCP server we’re firing
up and what port it’s listening on.
The important part is the (Just "79") part — that’s the port
we’re listening for connections on. Also note that you’ll need
administrative privileges on most operating systems to listen
on that port.

CHAPTER 31. FINAL PROJECT 1869
TCP socket libraries like network often call everything a
socket. Server listening for connections? That’s a socket.
Client connection that you were listening for? That’s a socket.
Everything’s a socket, and nothing’s a wrench.
The next bit constructs a sort of socket descriptor with
socket . Thenwebindthesockettotheaddress(port)wewanted.
Lastly, we let the operating system know we’re prepared to
listen for connections from clients with listen . From there, we
fire oﬀ our server logic which runs indefinitely. If and when
logAndEcho finishes, we’ll close the socket server and then our
story is over.
The next step, assuming your project is built, is to fire up the
debug server — note that it’ll want administrative privileges
for using port 79:
$ sudo `stack exec which debug`
{... build noise and a password prompt ...}
That will get our echo server set up, and we can now test
it using telnet to connect. Telnet is often used to debug TCP
services that use text to communicate. Note that you’ll need
to usesudoor otherwise make use of administrator powers to
start the program because it wants to use a network port that
only administrators or root accounts have access to in most
operating systems. Usually this is the first 1024 ports. Once
you have the debug server running in one terminal, you’ll
connect to it from a new terminal like so:

CHAPTER 31. FINAL PROJECT 1870
$ telnet localhost 79
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
From there, telnet is waiting for you to type something and
then hit enter:
blah
blah
Connection closed by foreign host.
In the above, we typed “blah,” hit enter, got “blah” echoed
back to us, then the server closed the connection. Remember
thatsClose is applied to socin ourlogAndEcho function, ensuring
that the temporary telnet connection is closed. However, the
server is still open, and you can make further requests by
reopening the telnet connection.
Let us take a look at the server side to see what it printed:
"blah\r\n"
We used printrather than putStrLn inlogAndEcho on purpose,
so we could get a literal representation of the data that was
sent. In this case, the string “blah” and the special characters
\rand\nwere sent. On Unix-based operating systems such

CHAPTER 31. FINAL PROJECT 1871
as Linux, \nis the default line-ending character. Microsoft
Windows uses \rfollowed by \nfor the same.
Having done that, let us now do the same with a finger
client:
$ finger callen@localhost
[localhost]
Trying 127.0.0.1...
callen
$ finger @localhost
[localhost]
Trying 127.0.0.1...
Particularly if you’re on a Mac, you may get some noise
here like this:
Trying ::1...
finger: connect: Connection refused
Trying 127.0.0.1...
It should connect after that. It attempts to use IPv6 first to
reach your finger daemon; when it can’t, it should use IPv4.
You can probably ignore this.
Then the output server-side for this would be:
"callen\r\n"
"\r\n"

CHAPTER 31. FINAL PROJECT 1872
The first command asked the finger daemon running at
localhost for information on the user callen; the second asked
for a listing of users. With the printed output the server gave
us, we now know what queries from a finger client will look
like to our TCP server. With that done, we’ll now write up the
final TCP server itself.
31.4 Slightly modernized fingerd
Historically, the data that finger returns about users was part
of the operating system. That information is still typically
stored in the OS, but for security reasons, it’s no longer rou-
tinely shared through finger requests. We’re going to update
the source of data for finger by using an embedded SQL6
database called SQLite. A database is a convenient yet robust
way of sorting and reading data, and SQLite is a lightweight
database. The data will be stored in a file within the main
project directory, so there won’t be a lot of mystery or magic
involved in interacting with it.
First we’ll show you the TCP server’s framing of the logic,
then we’ll show you how the database interaction works. From
here, all the code goes into your Main.hs file.
6Pronounced “squirrel.”

CHAPTER 31. FINAL PROJECT 1873
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE QuasiQuotes #-}
{-# LANGUAGE RecordWildCards #-}
OverloadedStrings you already know. QuasiQuotes is for the
literals, which you’ve seen before. RecordWildCards is the new
one and isn’t too difficult to figure out. It spares us manually
yanking the contents of a record into scope; instead, the record
accessors become bindings to the contents such that,
{-# LANGUAGE RecordWildCards #-}
moduleRWCDemo where
dataBlah=
Blah{ myThing ::Int}
wewBlah{..}=print myThing
wewwill print the myThing inside of the Blahargument it is
applied to without needing to apply myThing to aBlahvalue or
to destructure the contents of Blahin the pattern match. It’s
purely a convenience.

CHAPTER 31. FINAL PROJECT 1874
moduleMainwhere
importControl.Exception
importControl.Monad (forever)
importData.List (intersperse )
importData.Text (Text)
import qualified Data.Text asT
importData.Text.Encoding
(decodeUtf8 ,encodeUtf8 )
We’ll need the ability to decode a Textvalue from a UTF-8
ByteString andthenre-encodea TextvalueasaUTF-8 ByteString .
importData.Typeable
importDatabase.SQLite.Simple
hiding(close)
import qualified Database.SQLite.Simple
asSQLite
importDatabase.SQLite.Simple.Types
importNetwork.Socket hiding(close,recv)
importData.ByteString (ByteString )
import qualified Data.ByteString asBS
importNetwork.Socket.ByteString
(recv,sendAll)
importText.RawString.QQ

CHAPTER 31. FINAL PROJECT 1875
Creating the database We’re using the sqlite-simple library
to make a self-contained database stored in a file in the same
directory as our project. This will act as the repository of users
our finger daemon can report on.
dataUser=
User{
userId ::Integer
, username ::Text
, shell ::Text
, homeDirectory ::Text
, realName ::Text
, phone ::Text
}deriving (Eq,Show)
Now we dig into where the data comes from. Useris the
datatype describing our user records. It’s not super structured
or interesting, but gets things rolling. The only bit potentially
out of the ordinary here is that we have a userId field of type
Integer in order to provide the database with what’s called a
primary key. This is to provide a means of uniquely identify-
ing data in the database independent of the text fields in our
record type, among other things.
Weneedsomeboilerplatetypeclassinstancesformarshalling
and unmarshalling data to and from the SQLite database:

CHAPTER 31. FINAL PROJECT 1876
instance FromRow Userwhere
fromRow =User<$>field
<*>field
<*>field
<*>field
<*>field
<*>field
instance ToRowUserwhere
toRow (Userid_ username shell homeDir
realName phone) =
toRow (id_, username, shell, homeDir,
realName, phone)
This should remind you of FromJSON andToJSON .
createUsers ::Query
createUsers =[r|
CREATETABLEIFNOTEXISTSusers
(idINTEGER PRIMARY KEYAUTOINCREMENT ,
username TEXTUNIQUE,
shellTEXT, homeDirectory TEXT,
realName TEXT, phone TEXT)
|]

CHAPTER 31. FINAL PROJECT 1877
TheQuerytype is a newtype wrapper for a Textvalue. Con-
veniently, Queryhas anIsString instance, so string literals can
beQueryvalues. This isn’t really a query, though; it’s a SQL
statement defining the database table that will contain our
user data. The primary key stuﬀ is noise saying that the row is
namedidand that we want that field to autoincrement without
needing to do it ourselves. That is, if the last row we inserted
into the database had the id 1, then the new one will be auto-
assigned the primary key 2. The rest of it describes field names
and their representation (“TEXT”), but you’ll note we require
usernames to be unique so that there cannot be two Uservalues
with the same username.
insertUser ::Query
insertUser =
"INSERT INTO users \
\VALUES (?, ?, ?, ?, ?, ?)"
allUsers ::Query
allUsers =
"SELECT * from users"
getUserQuery ::Query
getUserQuery =
"SELECT * from users where username = ?"

CHAPTER 31. FINAL PROJECT 1878
This is utility stuﬀ for inserting a new user, getting all users
from the user table, and getting all the fields for a single user
with a particular username. The question marks are how the
sqlite-simple library parameterizes database queries.
dataDuplicateData =
DuplicateData
deriving (Eq,Show,Typeable )
instance Exception DuplicateData
The type above is a one-oﬀ exception we throw whenever
we get something other than zero or one users for a particular
username. That should be impossible, but you never know.
typeUserRow =
(Null,Text,Text,Text,Text,Text)
UserRow is a type synonym for the tuples we insert to create
a new user.

CHAPTER 31. FINAL PROJECT 1879
getUser ::Connection
->Text
->IO(MaybeUser)
getUser conn username = do
results <-
query conn getUserQuery ( Onlyusername)
caseresults of
[]->return$Nothing
[user]->return$Justuser
_ ->throwIO DuplicateData
TheOnlydata constructor is how we pass a single argument
instead of a 2-or-greater tuple to our query parameters when
using the sqlite-simple library. This is needed because basehas
no one-tuple type and getUserQuery takes a single parameter.
We check for none, one, or many results converting it into a
Nothing ,Just, orIOexception.
Finally, we need a utility function for creating the database
with a single example row of data:

CHAPTER 31. FINAL PROJECT 1880
createDatabase ::IO()
createDatabase = do
conn<-open"finger.db"
execute_ conn createUsers
execute conn insertUser meRow
rows<-query_ conn allUsers
mapM_ print (rows ::[User])
SQLite.close conn
wheremeRow::UserRow
meRow=
(Null,"callen" ,"/bin/zsh" ,
"/home/callen" ,"Chris Allen" ,
"555-123-4567" )
Stack may balk because you have a module called Mainthat
has nomaindefined. If that’s the case for you, you can do this:
main::IO()
main=createDatabase
We’ll change that mainlater, but that will get your executable
building for now.
Running this a second time will error without changing the
database. If you need or want to reset the database, you can
delete the finger.db file.

CHAPTER 31. FINAL PROJECT 1881
Before you continue The code that follows will assume and
require a SQLite database by the name of finger.db with the
schema outlined in createUsers exists in the same directory as
where you run your fingerd service.
To run createDatabase , you could do the following:
$ stack ghci --main-is fingerd:exe:fingerd
{... noise noise ...}
Prelude> createDatabase
User {userId = 1, ... noise ... }
With that in place, you can continue implementing your
finger daemon.
Let your fingers do the walking
We’re still in our Mainmodule here. You should have created
the database already, but now we’ll write the functions that
will allow the server to listen and respond to client queries.

CHAPTER 31. FINAL PROJECT 1882
returnUsers ::Connection
->Socket
->IO()
returnUsers dbConn soc = do
rows<-query_ dbConn allUsers
letusernames =map username rows
newlineSeparated =
T.concat$
intersperse "\n"usernames
sendAll soc (encodeUtf8 newlineSeparated)
returnUsers uses a database Connection and aSocket for talk-
ing to the user. The database connection is used to get a list
of all the users in the database which is then changed into
a newline separated Textvalue. Then that is encoded into a
UTF-8ByteString which is sent through the socket to the client.
formatUser ::User->ByteString
formatUser (User_username shell
homeDir realName _)=BS.concat
["Login: " , e username, "\t\t\t\t ",
"Name: " , e realName, "\n",
"Directory: " , e homeDir, "\t\t\t",
"Shell: " , e shell, "\n"]
wheree=encodeUtf8

CHAPTER 31. FINAL PROJECT 1883
This function is used to format Userrecords as a UTF-8
ByteString value. The format is intended to mimic popular
fingerd implementations but we’re not going for precision
here.
returnUser ::Connection
->Socket
->Text
->IO()
returnUser dbConn soc username = do
maybeUser <-
getUser dbConn ( T.strip username)
casemaybeUser of
Nothing -> do
putStrLn
("Couldn't find matching user \
\for username: "
++(show username))
return()
Justuser->
sendAll soc (formatUser user)
This is the single user query case, where we use formatUser
to provide detailed information to the client on a single user.
We have to handle the case where no user by the username
provided was found. As it stands, the Nothing case here will

CHAPTER 31. FINAL PROJECT 1884
print the report that no user was found by that username in the
server terminal but will not send that information — or any
information — to the client side. You may want to change that,
as it might be useful to tell the end user why no information
was returned.
If a user is found, we send the formatted ByteString of the
Userrecord to the client. The stripping of the username text
prior to querying is because the literal data sent for a user-
name query is "yourname\r\n" and in order for that to match
“yourname,” we need to strip the control characters from the
text, which stripfromData.Text does for us.
handleQuery ::Connection
->Socket
->IO()
handleQuery dbConn soc = do
msg<-recv soc 1024
casemsgof
"\r\n"->returnUsers dbConn soc
name->
returnUser dbConn soc
(decodeUtf8 name)
handleQuery receives up to 1024 bytes of data. Based on
that data the client sends to the server, the case discriminates
between when it should send a list of all users or only a single

CHAPTER 31. FINAL PROJECT 1885
user. Fortunately, the protocol is relatively uncomplicated,
so we don’t have to do any parsing as would ordinarily be
required for communicating with a more elaborate protocol.
handleQueries ::Connection
->Socket
->IO()
handleQueries dbConn sock =forever $ do
(soc,_)<-accept sock
putStrLn "Got connection, handling query"
handleQuery dbConn soc
sClose soc
It’s similar to the echo server, save for the additional ar-
gument of the database connection and the logging of when
connections were accepted.
Now we need to change mainto assemble our whole pro-
gram:

CHAPTER 31. FINAL PROJECT 1886
main::IO()
main=withSocketsDo $ do
addrinfos <-
getAddrInfo
(Just(defaultHints
{addrFlags =[AI_PASSIVE ]}))
Nothing (Just"79")
letserveraddr =head addrinfos
sock<-socket (addrFamily serveraddr)
StreamdefaultProtocol
bindSocket sock (addrAddress serveraddr)
listen sock 1
-- only one connection open at a time
conn<-open"finger.db"
handleQueries conn sock
SQLite.close conn
sClose sock
The only new bit above is the opening of a connection to a
SQLite database located in the same directory as your project.
The connection to the database is passed to the query-handling
code, which runs indefinitely like the echo-and-log server. If
it somehow stops without throwing an exception, we close the
server socket, just to be good little programmers.
Now we’re done, assuming you’ve created a SQLite database

CHAPTER 31. FINAL PROJECT 1887
usingcreateDatabase which is valid and accessible to your pro-
gram, the following should work. You’ll want to do this in one
terminal:
$ stack build
$ sudo `stack exec which fingerd`
Theninanother, diﬀerent, shellsessionthefollowingshould
work:
$ finger callen@localhost
Login: callen Name: Chris Allen
Directory: /home/callen Shell: /bin/zsh
And that’s it. In the exercises, we’ve given some ways to
extend this, and we hope you’ve enjoyed this little foray into
TCP sockets and basic networking. Security concerns aside,
thefinger protocol has been used over the years for some
pretty cool things. Perhaps most famously, John Carmack
used.planfiles as a kind of microblog to deliver updates on
the development process of Quake.7
31.5 Chapter Exercises
1.Try using the sqlite3 command line interface to add a
new user or modify an existing user in finger.db .
7http://atrophied.co.uk/read/john-carmacks-plan-archive

CHAPTER 31. FINAL PROJECT 1888
2.Write an executable separate of fingerd anddebugwhich
allows you to add new users to the database.
3.Add the ability to modify an existing user in the database.
4.Bound on a diﬀerent port, try creating a “control socket”
that permits inserting new data into the database while
the server is running. This will probably require, at mini-
mum, learning how to use forkIO and the basics of concur-
rency in Haskell among other things. Design the format
for representing the user rows passed over the TCP socket
yourself. For bonus points, write your own client exe-
cutable that takes the arguments from the command line
as well.
5.Celebrate completing this massive book.

Index
(), see unit
(->), see function type
constructor
(:), see cons
(<-), see bind
($), 79, 80, 82, 389–391, 1059
*, see kind
(++), see concatenation
::, see type signature
<*, seeApplicative
<*>, 1054, 1055, 1057, 1059,
1145, see also Applicative
<|>, seeAlternative
<$>, seefmap
=<<, see flip bind
=>, see typeclass constraint
>>, seeMonad
>>=, see bind
[], see list syntax
eta reduction, 1017, 1022~, see tilde
(||), 1231
|, see pipe
abs, 378, 382
abstract datatype, 1031, 1149
abstraction, 7, 32
accessor function, 1508
actual type, 190, 284
actual vs expected type, 1477
ad hoc polymorphism, see
constrained
polymorphism
aeson, 1425, 1476, 1481, 1484,
1488
algebra, 611, 615, 627, 636,
888, 889, 904, 941, 942
algebra, definition, 955
algebraic datatype, 627
All(newtype), 909
1889

INDEX 1890
all, 789
alpha equivalence, 9, 15, 218
Alternative , 1427, 1429, 1433,
1501
ambiguous type, 281, 839,
842, 929
AmbT, 1567
anamorphism, 742
anarchy, 375
anonymous function, 8, 339,
340, 344, 492, 509, 512
anonymous function
definition, 406
syntax, 200
anonymous product, 172,
613, 632
Any(newtype), 909
API definition, 1426
application, 3, 7, 21, 32
Applicative , 1053, 1054, 1127,
1131, 1143, 1150, 1170,
1197, 1226, 1227, 1231,
1283, 1319, 1337, 1341,
1353, 1514, 1525, 1563
Applicative
Compose , 1549IO, 1802
Reader , 1351
composition law, 1108
definition, 1138
homomorphism law, 1109
identity law, 1106
interchange law, 1111
applicative, 1110
Arbitrary , 835, 837, 843, 863,
869, 873, 928, 1012
arbitrary precision, 146
argument, 3, 7, 11, 45, 192,
330, 334, 336, 392, 611
argument
multiple, 15, 192, 194, 195,
198, 330, 332, 333, 459
type, see type argument
argument, definition, 95
arithmetic, 51, 68
arity, 161, 165, 611
arity, definition, 174
Array, 1752
array, 1748
as patterns, 693
ASCII, 1769, 1773
association list, see Map(type)

INDEX 1891
associativity, 53–55, 185, 196,
367, 541, 550–553, 557,
569, 885, 890, 892, 905,
915, 925, 927, 932, 941,
1190, 1712
associativity, Monad, 1806
AST, 1460, 1475
asynchronous exception,
1853, 1855
attoparsec , 1425, 1465, 1469
backtracking, 1469, 1471
bang bang, 121, see indexing
bang pattern, 1691, 1693,
1697, 1739
BangPatterns , 1689
base, 258, 502, 787, 937, 1665,
1712
base case, 422, 423, 425, 437,
443, 445, 462, 463, 535,
539
base monad, 1573
benchmarking, 1709, 1716,
1724, 1727, 1739, 1743,
1747, 1751
benchmarkingstring types, 1766
vectors, 1759
beta reduction, 10, 11, 13, 17
Bifunctor , 1519
binary tree, 681, 682, 746, 851
bind, 775, 777, 780, 781, 784,
797, 1144, 1145, 1148, 1156,
1170, 1191, 1202, 1296,
1516, 1522, 1526, 1528,
1531, 1534, 1539, 1543,
1575
bind, definition, 1210
binding, 46, 87, 88, 330, 334,
335, 338, 344
binding
definition, 406
local, 109, 117, 129
top level, 130
Bloodhound (library), 676
Bool, 134, 135, 148, 152, 155,
156, 250, 251, 286, 378,
382, 592, 594, 595, 616,
908
Bool, fun with, 153
bool, 510
Boole, George, 134

INDEX 1892
Boolean logic, 153
bottom, 236, 347, 431–434,
487, 495, 496, 498, 509,
543, 545, 546, 560, 570,
723, 897, 1180, 1531, 1533,
1632, 1635, 1640, 1658,
1676, 1702, 1722, 1851,
1853
bottom, definition, 412
Bounded , 143, 251
burrito, 1141, 1518
ByteString , 1239, 1253, 1255,
1415, 1469, 1670, 1762,
1767, 1874
ByteString
String conversion, 1769
lazy, 1476, 1479, 1768
lazy vs strict, 1476
strict, 1769
versusText, 1775
bytestring (library), 1241,
1769
Cabal, 752, 753, 1716
.cabal file, 754, 757, 759, 764,
786, 822, 826, 847, 1237,1712
cabal install, 139
Caesar cipher, 519
CAF (constant applicative
form), 1733, 1734, 1736,
1737
call by name, 1657, 1664
call by need, 1657, 1658
call by value, 1657
cardinality, 615, 616, 618, 619,
627, 628, 631–634, 637,
1474
Carnap, Rudolf, 959, 1048
Cartesian product, 1072
case expression, 360–363,
375, 537, 710, 806, 1639,
1644, 1652, 1654
cassava , 1425
cast, seeTypeable
catamorphism, 531, 908,
1260, 1268, 1270
catamorphism, definition,
585
catch, 1824, 1829–1831, 1833
catMaybes , 1287, 1294
Char, 99, 100, 150

INDEX 1893
Char8, 1769
character, 99, 100
checkers (library), 1115, 1117,
1191, 1308
Chomsky hierarchy, 1461
Church, Alonzo, 2
Clinton, George, 1050
closure, 1761
CoArbitrary , 873, 1012
combinator, 21, 22, 1399
command line argument,
1226
comment syntax, 65, 66
commutative monoid, 917
commutativity, 203, 916, 917,
1617
compare , 286
comparison functions, 147,
152, 284
compile a binary, 1826, 1832
compile time, 181, 605, 620,
622
composability, 985
Compose (type), 1308, 1509,
1511, 1513, 1514, 1516,
1548composition, 387–392, 394,
395, 397, 400, 425–428,
511, 1199, 1295, 1307,
1320, 1335, 1505, 1518
composition
Traversable , 1308
definition, 415
law, 980, 984, 1108
concat , 107, 110, 1147
concatenation, 105, 107, 110,
113, 115, 891, 894, 916,
1748
concatenation, definition,
129
concrete type, 187, 196, 209,
211, 212, 226, 279, 313,
314, 595, 598, 599, 603,
721–724, 1526, 1543, 1677,
1679, 1681
concurrency, 1708
conditional, 155
conduit (library), 1567
conjunction, 133, 154, 161,
526
conjunction ( Monoid ), 909,
910

INDEX 1894
cons (:), 119, 459, 460, 464,
487, 503, 504, 533, 554,
555
cons cell, 464, 465, 485–487,
489, 494, 546
cons cell, definition, 528
cons, definition, 526
Const(type), 1024, 1303
const, 538, 547, 556, 559, 569,
1024
Constant (type), 1024, 1025,
1080
Constant (type)Functor , 1025
constant, 595, 596, 600, 604,
618
constant applicative form,
see CAF
constrained polymorphism,
174, 183, 187, 190, 209,
211, 215, 239, 249, 279,
310, 314, 981, 1466, 1468
constructor, 594, 595, 603,
604, 606, 642
constructor
data, see data constructor
nullary, see nullaryconstructor
smart, see smart
constructor
type, see type constructor
constructor class, 979
containers (library), 1738
Control.Exception , 1857
Control.Monad , 1685
ContT, 1567
criterion , 1709, 1712, 1790
CSV parsing, 1228, 1425
curry, 201
Curry, Haskell, 15
currying, 15, 43, 192,
194–196, 199, 330, 367
currying, definition, 406
daemon, 1861
Damas-Hindley-Milner, 180,
217
data constructor, 133–135,
137, 152, 153, 162, 179,
260, 262, 344, 345, 349,
350, 352, 353, 360, 361,
363, 491, 591, 595, 596,
599, 600, 602, 607, 608,

INDEX 1895
611, 612, 619, 632, 633,
638, 707, 710, 729, 730,
732, 1229, 1266, 1644,
1661, 1676, 1720, 1724,
1819
data constructor
currying, 729
definition, 173
infix, 460, 679, 937
data declaration, 134, 135,
179, 258, 592–594, 597,
606, 608, 705
data declaration
definition, 174
how to read, 134
data structure, 1708, 1736,
1738
Data.Bool , 510
Data.Char , 517, 1772, 1773
Data.Foldable , 1262
Data.Map , 1073, 1219
Data.Maybe , 1356
Data.Monoid , 898, 1262
Data.Tuple , 163
database, 1872, 1875, 1877,
1882, 1885database
FromRow , 1876
ToRow, 1876
datatype, 128, 132, 135, 152,
179, 353, 360, 592
datatype
algebraic, see algebraic
datatype
definition, 129, 703
recursive, 460, 485
Debug.Trace , 1665, 1796
declaration, 41, 45, 46, 61, 85,
108, 152
declaration
class, see typeclass
declaration
data, see data declaration
instance , see typeclass
instance
type, see type alias
fixity, 1712, 1776
import, 765
local, 108
module, 758
newtype, 620
top level, 117, 223, 226

INDEX 1896
type signature, 99, 196
deepseq , 1715
dependency, 753, 758, 787,
823, 824, 826, 827, 833
DeriveGeneric , 874
deriving, 136, 257, 259, 289,
290, 609, 624, 710, 1844
deriving Show, 265, 301, 303
deriving, definition, 323
desugar, 199, 465, 505, 1155,
1161
diﬀerence list, see DList
disjunction, 133, 134, 152, 154,
527, 627, 652, 1231
disjunction ( Monoid ), 909, 910
distributive property, 636,
637, 639, 640
division, 73, 442, 443, 445
division
fractional, 145, 146, 187
integral, 68
DList, 1388, 1775
dosyntax, 104, 774, 779,
781–784, 797, 806, 828,
1033, 1145, 1154, 1155,
1161, 1169, 1234, 1350,1554
documentation, 181, 827
Double , 139, 145, 146, 1693
drop, 120, 469, 471
dropWhile , 469, 472, 473
dynamic typechecking, 1823
eﬀects, 103, 299–301, 774,
783, 1031, 1149, 1159,
1161, 1221, 1227, 1761,
1782, 1788, 1798, 1799,
1836, 1841
eﬀects, definition, 323
Either , 640, 709, 712–714, 716,
717, 722, 844, 1006, 1021,
1132, 1255, 1276, 1298,
1306, 1608, 1819, 1821,
1832
Either
Applicative , 1131
Functor , 1021
Monad, 1182
EitherT , 1235, 1555, 1600,
1610, 1614
elem, 203, 482, 801, 1276
Elliott, Conal, 1115, 1567

INDEX 1897
empty list, 503
Enum, 251, 261, 275, 294, 467
Enumfunctions, 294
enumFromTo , 296
Eq, 148, 249, 251, 252, 256,
258–260, 270, 272, 292,
293, 710
Eqfunctions, 254
equality, 147, 148, 249, 252,
253, 259, 264, 293
Erlang, 1853
error, 433, 434, 1376
error
ambiguous type variable,
1272
could not deduce, 277, 284,
312, 341
expected vs actual type,
189, 255, 284, 368, 435,
566, 622, 648, 653, 666,
707, 729, 777, 782, 900,
963, 1030, 1201
expecting one more
argument, 725–727
no instance for, 114, 151,
158, 216, 225, 259, 265,271, 272, 292, 302, 308,
310, 434, 731, 895, 899
no instance for Show, 162,
288
not in scope, 118, 336, 732,
767
too many arguments, 900
error message, how to read,
114, 119, 152, 288
eta reduction, 1673, 1736
Eval(typeclass), 1640
evaluate, 3
evaluation, 20, 41, 47, 49, 196,
485, 487–491, 493–498,
504, 508, 539, 550, 1630,
1633, 1640, 1788
evaluation
foldl, 549, 553–555,
559–561, 569
foldr, 538, 539, 541–543,
545, 551, 569
call by need, 491
folds, 541
inside out, 1635
outside in, 1635
recursive function, 538,

INDEX 1898
541, 543
strategies, 1657
evaluation order, 1782, 1786,
1790
Exception (typeclass), 1815,
1844
Exception ,throw, 1840
exception, 122, 347, 361, 431,
432, 1421, 1813, 1825,
1833, 1878
exception
mask_, 1857
asynchronous, see
asynchronous
exception
empty structure, 1278
handling , see exception
handling
loop, 431
missing field, 660
no match, 665
no parse, 304
non-exhaustive patterns,
266, 347, 433, 462
thread blocked, 1794
throw, 1838, 1840, 1842,1843, 1845
undefined, 508, 546, 547,
559, 569
exception handling, 1823,
1824, 1827, 1829, 1831,
1833, 1840, 1841
exception handling
catch, 1824, 1829, 1833, see
alsocatch
Either , 1832
Maybe, 1831
try, 1832, 1833, 1848
bottom, 1851
ExceptT , 1569, 1570, 1584, 1610
executable, 757, 759, 1835,
1880
executable, with arguments,
1836
ExistentialQuantification ,
1819
existential quantification,
1762, 1817, 1821, 1848
exitSuccess , 792
expected type, 190, 256, 284
exponentiation, 55
export, 849

INDEX 1899
expression, 3, 7, 41, 43, 47, 48,
60, 85, 133
expression problem, 249
expression, definition, 95
factorial, 421, 425
fail, 1422
fibonacci, 436, 441, 576–578
file, 58
filter , 511, 578
finger , 1860
finger tree, 1744
finger, MIT, 1862
First(newtype), 910, 912
FlexibleInstances , 1044
flip, 365, 553
flip bind ( Monad), 1288
Float, 138, 145
floating point numbers, 139
fmap, 500, 502, 508, 714, 732,
962, 1018, 1057, 1059,
1087, 1145, 1157, 1201,
1221, 1224, 1274, 1288,
1295, 1302, 1318, 1330,
1539, 1574
fmap,IO, 1801fmap, infix, 1087
fold, 531, 532, 903, 1038,
1542, 1543, 1635
fold, 1263
fold left, see foldl
fold right, see foldr
fold, definition, 585
Foldable , 112, 482, 532, 790,
903, 1261, 1262, 1302,
1303, 1390
foldl, 548, 549, 559, 570, 572,
588, 1269
foldl' , 561
foldMap , 1261, 1263, 1265,
1270
foldr, 532, 533, 536, 545, 569,
571, 1261, 1267, 1269,
1717, 1724
forall , 1818
foreign function interface
(FFI), 1475
forever , 809, 1685, 1840
Fractional , 138, 146, 215, 276,
277
fractional, 145
FromJSON , 1484, 1486

INDEX 1900
fromMaybe , 1293, 1359
fst, 162, 508
function, 3, 4, 7, 41, 43–45,
192, 298, 330, 332, 333,
984
function
anonymous, 8
application, 10, 41, 46–48,
80, 185, 195, 330, 334,
335, 389, 427, 507, 598,
707, 973, 984, 1059, 1071,
1110, 1199, 1219
body, 46
composition, see function
composition
datatype, 184
first-class, 3, 330
head, 46
higher-order, see
higher-order function
infix, 51
mathematical, 48
parameter, 192
prefix, 51
structure, 7, 8
unsafe, 122function composition, 985,
995, 1203, 1209, 1325,
1328, 1331, 1334, 1507,
1509, 1573
function type, 192, 194, 195,
252, 289, 1319
function type
Applicative , 1337
Functor , 1328, 1330, 1336
Monad, 1348
Monoid , 1218, 1220
asReader , 1334
function type constructor,
185, 195, 197, 367, 723,
729, 748, 875
function, definition, 96
functional dependencies,
596
Functor , 500, 502, 714, 958,
960–962, 965, 972, 973,
984, 1005, 1018, 1039,
1054, 1057, 1143, 1197,
1221, 1276, 1319, 1328,
1330, 1334, 1511, 1524
Functor laws, 979, 983, 1010
Functor , definition, 1047

INDEX 1901
functor, 959, 960, 1142, 1200,
1318, 1325
functor, applicative, 1145
fusion, 1753, 1754
GADTs, 1818, 1819
garbage collection, 1628,
1631, 1736
Gen, 835, 837, 839, 861, 864,
866, 869, 873
GeneralizedNewtypeDeriving ,
624, 626
generalized algebraic
datatype, see GADTs
generator, 477, 478
generator
multiple, 479, 480
Generic , 874
getArgs , 1835
getChar , 1782
getLine , 775, 777, 1157
GHC 8.0, 1693
GHC Core, 1647, 1652, 1654,
1680, 1689
GHC extension, see
language extensionGHC flag, 1729
GHC flag
-ddump , 1435, 1648
-fprof-auto , 1729
-I, 921
-O2, 1710, 1730
-O, 1710
-prof, 1729
-rtsopts , 1730
-Wall, 267, 348, 385
GHC optimization, 1661,
1716, 1730, 1754, 1787
GHC optimization,
strictness, 1669, 1676
GHC Rules, 1754
GHC.Prim , 1785
GHCi, 36, 41, 45, 339, 929,
1031, 1032, 1825
GHCi block syntax, 345, 348,
378
GHCi command
:browse , 358, 765, 827
:info, 53, 78, 135, 144, 155,
250, 252, 843
:kind, 598, 676, 721
:load, 40

INDEX 1902
:main, 1837
:module , 40, 770
:reload , 47
:set, 348, 385, 770, 1436,
1648
:sprint , 488, 493, 1660
:type, 99, 111, 152, 182, 618,
964
GHCi options, 766
Gibbard, Cale, 540
git, 753, 754
gopattern, 445
Gofer, 979
guard, 377, 379, 380,
382–385, 711
guarded recursion, 1724
gzip, 1768
Hackage, 258
HashMap , 1741
Haskell ninjas, 307
Haskell Report, 279, 596,
600, 720, 1716
head, 120
heap profiling, 1731
hGetChar , 1809hgrev(library), 1226
higher-kinded, 720, 722, 726,
729
higher-kinded
polymorphism
definition, 1047
higher-kinded type, 674, 677,
680, 681, 720, 965, 974,
979, 1005, 1007, 1034,
1047, 1262
higher-kinded type, Functor ,
972
higher-kinded type,
definition, 748
higher-order function, 200,
365, 366, 369, 375, 387,
421, 425, 472, 500, 512,
1399
higher-order function
definition, 413
Hindley-Milner, see
Damas-Hindley-Milner
homomorphism, 1109, 1130
Hoogle, 252
hspec(testing), 822, 825, 828,
833, 845, 1456

INDEX 1903
http-client (library), 1842,
1843
Hutton’s Razor, 702
I/O, 103, 301, 775, 1825
id, 210, 1507, 1508
idempotent, 881
idempotent, definition, 886
Identity (type), 270, 865,
1078, 1269, 1270, 1302,
1307, 1507, 1508, 1511,
1524, 1568, 1569
identity, 905
identity
function, 11, 12, 21, see id
law, 979, 983, 1106, 1189
property, 929, 932
identity value, 425, 535, 536,
561, 890–892, 917, 933,
936, 939, 941, 1270
IdentityT , 1508, 1523, 1524,
1526, 1530, 1534, 1538,
1541, 1544, 1589, 1600
idiom, 1139
ifexpression, 155, 156, 360,
361, 363, 377–380, 509,510, 783, 805
immutability, 504, 511, 643,
683
imperative programming,
783, 1149
import, 109, 155, 163, 246,
765, 767, 787, 825, 850,
855
import
hiding, 1469, 1866
qualified, 768, 1244
qualified as, 769, 850, 1244
import syntax, 1583, 1587
indentation, 59, 60
indexing, 121, 577, 796, 1748
infinite list, 1733
infix operator, 51, 53, 68, 79,
185, 194, 202, 678, 1712
infix operator
associativity, 53–55, 194,
195
precedence, 53, 54, 195
prefix, 52, 82, 110, 115
sectioning, see sectioning
infix, definition, 96
infixl , 54

INDEX 1904
infixr , 55, 1776
:info, 135
INI, 1444
INLINABLE , 1717
INLINE , 1776
inlining, 1670, 1672, 1787
input, 3
input/output, see I/O
instance, 140, 146, 257
instance , 260
instance, orphan, see orphan
instance
InstanceSigs , 1343, 1515, 1516,
1531
Int, 138, 141, 616, 1693
IntversusInteger , 1719
Int32, 1368
Int8, 142, 616, 629
Integer , 128, 138, 140, 143,
895, 1488
Integer ,Monoid , 895, 896
integer, 68, 140
Integral , 274, 275
Integral functions, 274
interface, 249
intersperse , 791IntMap , 1741
IO (), 103, 300, 776, 779,
1031, 1158
IO, 103, 299, 301, 774, 781,
782, 837, 1031, 1032,
1149, 1154, 1224, 1233,
1235, 1244, 1505, 1522,
1573, 1597, 1759, 1782,
1784, 1825, 1841
IO
Applicative , 1075, 1801,
1802
Functor , 1031, 1157, 1800
Monad, 1149, 1804
asState, 1785, 1786
associativity, 1806
exceptions, 1833
sharing, 1664
unsafe functions, 1809
IOaction, 301, 1161, 1255,
1787, 1789
IO, definition, 323
IO Monad , the, 1784
IOException , 1828, 1831
IRC, 540, 591
irrefutable pattern,

INDEX 1905
1686–1688, 1694
isomorphism, 885
IsString , 1239, 1241
JavaScript, 1488
join, seeMonad, 1148, 1154,
1160, 1209, 1296, 1529,
1536, 1539, 1543, 1801
join,IO, 1804
JSON, 676, 677, 1397, 1425,
1476, 1481, 1488
JSON parsing, 1227
key-value pair, see Map(type)
keyword
~, 1688, 1694
!, 1691, 1739
*, 598, 966
--, 65
->, 192, 966
::, 99, 106, 196, 720
<-, 781
=>, 190
=, 45
@, 693
#, 625, 1786
_, 137, 345as, 769, 1244
case, see case expression,
see case expression
class, 305, 960
data, 135, 174, 593
deriving , 257, 609
do, 104, 775, 779
forall , 1818
hiding , 1469, 1866
if-then-else , 156
if, 156
import , 765, 767
infixl , 54, 1712
infixr , 55, 1776
instance , 251, 260, 261
let, 45, 85
let,in, 61
module , 751
newtype , 620
qualified , 768, 769, 1244
type, 620, 633
where, 85, 88, 261, 961
|, 134, 593
kind, 597–599, 603, 674, 681,
720, 722, 723, 725, 726,
966, 971, 978, 989

INDEX 1906
kind inference, 970
Kleisli composition, 1202,
1203, 1209
lambda, 3, 31, 195
lambda calculus, 2, 32, 43,
48, 180, 192, 298, 427,
1161, 1798
lambda expression, 1761
lambda term, 7
language extension
BangPatterns , 1689
DeriveGeneric , 874
ExistentialQuantification ,
1819
GADTs, 1819
GeneralizedNewtypeDeriving ,
624, 626
InstanceSigs , 1343, 1515,
1516, 1531
NegativeLiterals , 631
NoImplicitPrelude , 766
NoMonomorphismRestriction ,
226
OverloadedStrings , 1214,
1239, 1416, 1469, 1485,1598, 1605, 1767
QuasiQuotes , 1433, 1485,
1873
RankNTypes , 1035
RecordWildCards , 1873
StrictData , 1693
Strict , 1693
TypeApplications , 964
Last(newtype), 910, 912
laws, 904
laws
Applicative , 1106
Functor , 979, 982
Monad, 1188
Monoid , 904
Traversable , 1307
mathematical, 955
laziness, 1630, 1632, see also
nonstrictness
leaf, 682
length , 167, 186, 215, 490,
494–496, 535, 544, 1275
let, 45, 59, 85, 86, 108, 224,
335, 338, 1685
letexpression, 86, 338
letversuswhere, 85

INDEX 1907
lexing, 1460, 1461
library, 759, 761, 764
library
aeson, 1476
attoparsec , 1425
bytestring , 1241, 1476, 1769
checkers , 1115, 1191
containers , 849, 1738
criterion , 1709
hspec, 822
http-client , 1842
network , 1866
parsec , 1468
parsers , 1412, 1426
QuickCheck , 871, 1012, 1115
random , 792, 1367
scientific , 139, 145
scotty , 1247, 1256, 1576
snap, 1224
sqlite-simple , 1875
text, 1223, 1241, 1763, 1774
time, 1789
transformers , 1378, 1564,
1585, 1598
trifecta , 1415, 1425, 1468
uuid, 1223vector , 1290, 1748
wreq, 1300
lift, seeMonadTrans , 1589,
1608
liftA2 , 1229, 1231
lifting, 977, 981, 1032, 1054,
1055, 1059, 1150, 1221,
1229, 1331, 1511, 1574,
1575, 1580, 1582, 1588
lifting
definition, 1048
liftIO , 1253, 1597, 1608
liftM, 1151
lines, 793
List, 592, 597, 678–680, 726,
727
list
Applicative , 1061, 1068,
1069
Monad, 1163
Monoid , 1265
datatype, 459, 527, 528
empty, 536
infinite, 542, 543, 561, 576
structure, 486, 487,
494–496, 507, 529

INDEX 1908
type constructor, 165
list comprehension, 477–479,
482, 512
list comprehension, with
condition, 478, 479, 482
list functions, 119
list monoid, 1120
list syntax, 165, 460
lists, 98–100, 105, 112, 113,
119, 165, 464, 500, 503,
891, 893, 1260, 1567,
1695, 1738, 1745, 1747,
1748, 1752
ListT, 1567
logging, 1566
lookup , 1072, 1356
loop fusion, see fusion
LTS Haskell, 754, 755
Main, 89, 1832, 1880
:main, 1837
main, 102–104, 1243, 1256,
1788, 1835, 1880
main, with arguments, 1836
many, seeAlternative
Map(type), 850, 853, 1073,1219, 1738, 1739, 1741,
1748
map, 500, 502, 506–508, 533,
534, 962, 1725
mapM_, 1836
mapM, 1289
mappend , 891, 893, 896, 899,
1216
mappend
infix, 901, 1215, 1219
Marlow, Simon, 1792, 1814
marshalling, 1475, 1476, 1486,
see also serialization
marshalling, definition, 1501
max, 286
maxBound , 143
maximum , 1277
Maybe, 128, 433–435, 462, 463,
705, 706, 708, 722, 724,
725, 727, 802, 805, 844,
871, 910, 1062, 1224,
1255, 1270, 1404, 1604,
1608, 1831
Maybe
Applicative , 1066, 1075,
1083, 1097, 1172

INDEX 1909
Functor , 1015
Monad, 1166, 1172, 1174
Monoid , 1066
MaybeT , 1506, 1548, 1552, 1589,
1591, 1601, 1606
mconcat , 901, 1214
memoization, 1632
memory, 143, 1731, 1733, 1736
memory leak, 1566, 1693
memory leak, definition,
1627
mempty , 891, 893, 902, 1268,
1272, 1419
min, 286
minBound , 143
minimal complete instance,
258, 1285, 1427
minimum , 1277
mod, 69, 71
mod, diﬀerence from rem, 75
module, 58, 107, 109, 117,
246, 751, 753
module
definition, 239
export, 762
import, 765modules, 175
Monad, 775, 779, 1142–1144,
1197, 1224, 1233, 1235,
1252, 1327, 1328, 1341,
1345, 1353, 1366, 1516,
1526, 1543, 1563, 1784,
1788, 1798
Monad
(>>), 1402
fail, 1422
IO, 1804
Reader , 1348
composition, 1200
laws, 1188
monad, 828, 839, 866, 1617
monad transformer, 1216,
1235, 1256, 1355, 1378,
1505, 1506, 1508, 1518,
1520, 1523, 1541–1544,
1547, 1571, 1576, 1591,
1595, 1604
monad transformer,
definition, 1362
monad, definition, 1209
MonadFail , 1422
MonadIO , 1597, 1598, 1600

INDEX 1910
MonadTrans , 1574, 1575, 1582,
1589, 1591
Monoid , 891, 892, 902, 942,
1197, 1215, 1218, 1226,
1262, 1263, 1265
Monoid
Bool, 909, 930, 932
Integer , 895
Maybe, 910–912
Monoid , of functions, 1218
monoid, 888, 890, 892, 893,
897, 902, 903, 909,
1064, 1071, 1110, 1133,
1152, 1213, 1221, 1233,
1260, 1262, 1263
monoid
commutative, 902
definition, 955
monoidal functor, 1053,
1059, 1066, 1110
monomorphism restriction,
226, 1317
Morse code, 1292
mtl(library), 1383
mutable state, 1760
mutable vector, 1757, 1758mutation, 1366, 1757, 1759,
1761, 1762
MVar, 1792, 1806, 1808
named entities, 175
natural transformation,
1034, 1038, 1132
negate , 77
negation, 84
NegativeLiterals , 630, 631
negative number, 76
nesting, 15, 42, 1161, 1787,
1790, 1798, 1804
network-uri (library), 1252
network (library), 1234, 1866,
1868
network interface, 1475
newtype, 306, 349, 350, 591,
620–622, 624, 810, 897,
898, 908, 911, 919, 1334,
1338, 1507, 1508, 1578,
1595, 1749, 1775
nf, 1712
NICTA, 1563
nil, 434
NoImplicitPrelude , 766

INDEX 1911
NoMonomorphismRestriction ,
226
non-exhaustive patterns,
267–269, 385
NonEmpty , 463, 937, 939
nonstrict evaluation, 460,
485–488, 500, 507
nonstrictness, 47, 196, 508,
542, 546, 561, 1150, 1630,
1632, 1633, 1635,
1657–1659, 1672, 1694,
1851, 1853
nonstrictness, sharing, 1664
normal form, 20, 21, 42, 49,
490–492, 499, 637, 639,
640, 1712, 1720, 1727,
1747
normal order, 29, 31, 33
not, 135
null, 1274
nullary, 593, 596, 611, 618
nullary constructor, 729
nullary type, 720
Num, 139, 146, 188, 249, 252,
273, 276, 1240
Numfunctions, 273number, 47
numeric literal, 41, 183, 188,
215, 219, 249, 345, 623,
1240
numeric type, 137
O’Sullivan, Bryan, 1709
Only, 1879
operator, 51, 890
operator
infix, see infix operator
operator, definition, 96
optimization, 181, 1753
Ord, 148, 150, 251, 261, 272,
284, 286, 289, 290, 292,
293, 312, 369, 371, 682,
851, 1741
Ordfunctions, 284
Ordering , 286
orphan instance, 919,
921–923, 928
otherwise, 381, 382, 385
overflow, 141, 143
OverloadedStrings , 1214, 1239,
1241, 1243, 1416, 1469,
1485, 1598, 1605, 1767

INDEX 1912
package, 753
parallelism, 1708
param, 1249
parameter, 7, 45, 46, 195, 209,
330–334, 438, 721
parameter, definition, 95
parametric polymorphism,
174, 209–211, 213, 239,
310
parametricity, 211, 213, 239,
314, 1039
parentheses, 54, 56, 79, 82,
84, 196, 367, 390, 392,
551, 707
parse error, 62, 64, 66, 826
parsec (library), 1425, 1465,
1468, 1470
Parser (type), 1403, 1422
parser, 1226, 1227, 1250, 1399,
1500, 1565
parser
Hutton-Meijer, 1405
parser combinator, 1399
parser combinator
definition, 1501
parsers (library), 1426, 1429Parsing (typeclass), 1427
parsing, 1396, 1398, 1401,
1460, 1461, 1465, 1468,
1474, 1486
parsing, backtracking, see
backtracking
partial application, 83, 196,
197, 202, 204, 1007, 1319
partial function, 122, 265,
266, 268, 292, 304, 347,
361, 432, 433
pattern match
non-exhaustive, 347, 348
pattern matching, 137, 164,
232, 344–347, 349, 350,
352–357, 361, 363, 375,
460, 463, 494, 496, 503,
545, 621, 647, 710, 713,
719, 1031, 1224, 1527,
1644, 1654, 1686, 1850
pattern matching
definition, 406
lazy, 1688
penguins, 355
Peyton-Jones, Simon, 1149
phantom type, 597, 601, 912,

INDEX 1913
1025
pipe, 134, 381, 477, 527, 593,
594
pipes(library), 1567, 1766
pointer, 898, 1749
pointfree, 392–394, 399,
400, 1673, 1684, 1736
pointfree
definition, 416
polymorphic literal, 1239
polymorphism, 113, 142, 152,
183, 208, 211, 216, 217,
283, 284, 310, 334, 488,
1047, 1317, 1521, 1662,
1681
polymorphism
ad hoc, see constrained
polymorphism
constrained, see
constrained
polymorphism
definition, 174, 239
higher-kinded, 749
parametric, see
parametric
polymorphismpragma, 624, 625
pragma
INLINABLE , 1717
LANGUAGE , 624
MINIMAL , 1261, 1285
UNPACK , 1739
precedence, 53, 55, 76, 79,
389, 1712
prefix, 51
Prelude , 155, 495, 766, 767,
1262, 1283, 1295
primary key, 1875, 1877
primitive type, 1785, 1786
principal type, 239
print it , 288
print, 101, 288, 289, 298, 299,
301, 396, 397, 784
Product (newtype), 896, 898,
901
Product (type), 644, 651, 1272
product, 459, 460, 612, 613
product , 1278
product type, 161, 354, 594,
615, 631–635, 649, 650,
798, 867
product type, definition, 526

INDEX 1914
profiling, 1727, 1729, 1731,
1733, 1737
prompt, 41
property test, 927
property testing, definition,
885
pseudorandom, 837, 1367,
1369
puppies, 384
pure, 1054, 1144, 1197
pure,IO, 1803
purity, 3, 298, 1798, 1799,
1825
putStr , 101
putStrLn , 101, 784, 1157
quantification
existential, see existential
quantification
universal, 1819
QuasiQuotes , 1433, 1485, 1873
queue, 1778
QuickCheck , 821, 833, 835, 871,
927, 928, 930, 1010,
1012, 1115, 1199, 1308
random (function), 787, 1370random (library), 792, 1367
random number generation,
1223, 1244, 1367, 1382
random values, 793, 837
randomRIO , 795, 797
range syntax, 203, 465, 467,
470, 493, 494
RankNTypes , 1034, 1035
Rational , 139, 145
Read, 251, 303, 304, 1250
Read, is not good, 303, 304
read, 1031
Reader , 989, 1224, 1325, 1327,
1330, 1334, 1335, 1337,
1340, 1341, 1522, 1564,
1566, 1569, 1619
Reader
Functor , 1336
Monad, 1348
ReaderT , 1354, 1355, 1557, 1561,
1564, 1566, 1569, 1585,
1619, 1676, 1808
readFile , 1766
Real, 275
RealWorld , 1785, 1786
record

INDEX 1915
accessor, 635, 665, 1544
syntax, 591, 635, 901
record type, 591
RecordWildCards , 1873
recursion, 420, 421, 428, 439,
494, 500, 504, 534, 535
recursion
definition, 455
guarded, 1724
tail, 587
recursive function, 436, 438,
441, 443–445, 576
recursive function
evaluation, 425, 440, 446
recursive type, 682
Redis, 1246, 1255
reduce, 3
reducible expression, 42, 48,
49
reduction, 41, 47, 80
referential transparency, 3,
1757, 1760, 1799, 1841
referential transparency, IO,
1799
refutable pattern, 1686
regular expression, 1461:reload , 47
remainder, 68
REPL, 36, 41, 47, 54
replicate , 1854
replicateM , 1245
return , 781, 782, 839, 864,
1032, 1144, 1189, 1591
runtime, 605, 620, 1824
RWST, 1565
scan, 549, 573, 574, 576
Schönfinkel, Moses, 15
Scientific , 139, 145, 1488
scope, 40, 87, 99, 108, 117, 119,
155, 223, 335, 338, 339,
608, 751, 765
scope
definition, 129
lexical, 337, 339
scotty (web framework),
1214, 1215, 1244, 1247,
1256, 1569, 1576, 1584,
1587, 1595, 1598, 1604,
1609, 1622
sectioning, 81, 83, 202, 204
semantics, 77

INDEX 1916
semantics
IO, 1806
Haskell, 25, 1798
program, 1799
Semigroup , 936, 937, 939, 942
semigroup, 888, 936
semigroup, definition, 955
seq, 1639, 1640, 1643, 1655,
1689, 1693
Sequence (type), 1738, 1744,
1745, 1747, 1748
sequence , 1290, 1294, 1296,
1298
sequenceA , 1285, 1289–1291,
1308, see also
Traversable
sequencing, 1144, 1149, 1154,
1161, 1227, 1787, see
Monad, 1790
serialization, 296, 303, 1397,
1475, 1486, 1491, 1501
server, 1861, 1867
Set(type), 1738, 1741
set, 133
set theory, 133, 615
Setup.hs , 846shadowing, 336–339
sharing, 1664, 1668–1670,
1672, 1677, 1681–1685,
1733, 1734, 1783, 1786,
1788, 1790, 1791
sharing, IO, 1789, 1791, 1796
Show, 136, 251, 261, 265, 288,
289, 296, 297, 299,
301–303, 396, 731, 830,
1815
Showfunctions, 297
show, 798
side eﬀect, see eﬀects
Simons, 651
smart constructor, 1234
snap(web framework), 1224
snd, 162
snoc, 1776
Snoyman, Michael, 1842
socket, 1234, 1863, 1867, 1869
some, seeAlternative
SomeException , 1817–1819,
1823, 1824, 1838
source code, 45
spine, 464, 465, 485–487, 494,
531, 545

INDEX 1917
spine
definition, 529
recursion, 541, 542, 544,
547, 557, 559–561
spine strict, 490, 494
splitAt , 469, 471
:sprint , 1660, 1676, 1679
SQLite, 1872, 1877, 1881
sqlite-simple (library), 1875,
1879
ST, 1366, 1758–1760, 1762,
1782, 1785, 1788
Stack, 103, 752, 753, 755, 1239,
1712, 1880
stack.yaml , 755
Stack commands, 754, 784,
846, 1826
Stack commands
build, 755, 760, 824, 827,
833, 1579, 1712, 1887
clean, 1716
exec, 756, 760, 776, 859,
1869
ghci, 36, 755, 824, 862,
1579, 1881
ghciwith options, 766ghc, 1710, 1712, 1729, 1836
init, 824
install , 139
new, 784, 1863
setup, 755
compile a binary, 1836
Stackage, 754
StackOverflow (exception),
1838
State# , 1786
State, 1354, 1366, 1367, 1371,
1378, 1404, 1406, 1564,
1566, 1759, 1761, 1784,
1785
state, 1365, 1371
StateT , 1406, 1561, 1563, 1564,
1566, 1587
static typing, 181
StdGen , 1368
stdin, 1809
stdout , 1809
streaming, 1566, 1567
Strict , 1693
StrictData , 1693
strictness, 488, 494, 497, 508,
538, 545, 561, 1217, 1631,

INDEX 1918
1637–1639, 1654, 1655,
1657, 1661, 1691, 1695,
1701
String , 99, 100, 106, 113, 119,
126, 128, 259, 289, 296,
297, 301, 303, 304, 482,
1239, 1669, 1670, 1762,
1766
String , definition, 128
strings, 98, 100, 105
subclass, 212
Sum(newtype), 896, 898, 1219
Sum(type), 644, 652, 1272
sum, 497, 534, 1278
sum type, 134, 152, 350, 353,
459, 503, 594, 607, 615,
616, 627, 628, 638, 640,
649, 652, 665, 709, 716,
869, 871
sum type, definition, 526
superclass, 146, 212, 293, 323,
1143
syntactic sugar, 77, 100, 180,
464, 774, 1154
syntactic sugar, definition,
96syntax, 59
System.Environment , 1835
System F, 180
tail, 120, 727
tail call, definition, 587
take, 120, 469, 470, 509, 544,
578
takeWhile , 469, 472
TCP, 1863
Template Haskell, 1435
term level, 133, 152, 161, 175
terminate, 41, 431
testing, 181
testing
property, 820, 833, 861,
862, 873
spec, 820, 822, 828
unit, 819
Text, 1223, 1239, 1247, 1253,
1256, 1670, 1762–1764,
1766, 1775
text(library), 1223, 1241,
1763, 1774
thread, 1855, 1857
threadDelay , 1840

INDEX 1919
throw, 1840, 1841
throwIO , 1838, 1840–1842,
1845
thunk, 1628, 1631, 1660, 1662,
1670, 1685, 1693
tie fighter, 1054
tilde, 1688, 1694
time(library), 1789
ToJSON , 1486
token (parsing), 1444, 1460
tokenize, 1460, 1461, 1465
tokenizer, definition, 1501
toList , 1273
top level, 106–108, 117
total function, 433
trace, 1665, 1677
transformer stack, 1363
transformers (library), 1378,
1383, 1564, 1566, 1569,
1585, 1598, 1610, 1619
Traversable , 1283, 1284, 1306
Traversable laws, 1307
Traversable naturality law,
1307
traverse , 857, 1285, 1287,
1289, 1291, 1292, 1295,1296, 1299, 1307, 1836
tree, binary, see binary tree
trifecta (library), 1400, 1415,
1425, 1465, 1468, 1469
Trivial (type), 258, 259, 596,
597
try(exceptions), 1832, 1833
try(parsing), 1444, 1473
tuple, 161, 198, 256, 356–358,
445, 471, 480, 613, 632,
633, 640, 721, 989, 1006,
1064, 1229, 1306
tuple
Applicative , 1064
Functor , 1064
constructor, 164
definition, 172
single element, 1879
syntax, 161, 164
typeclass instances, 1306
tuple functions, 162, 163
Turing completeness, 421
twitter-conduit (library), 1831
two’s complement, 143
type, 98, 100, 101, 132, 133,
249, 591

INDEX 1920
type
concrete, see concrete
type
definition, 129
higher-kinded, see
higher-kinded
lifted, 723
static, 605
unlifted, 723
type alias, 100, 165, 442, 622,
624, 633, 639, 651, 654,
707, 715, 717, 1224, see
also type synonym
type alias, definition, 174
type argument, 459,
594–597, 606, 608, 609,
613, 619, 633, 650, 674,
680, 681, 721, 722, 725,
729, 1509, 1521
type assignment, 142, 196,
279
type constant, 720, 973
type constructor, 112,
133–135, 152, 179, 184,
260, 361, 593, 595, 596,
599, 638, 720–724, 728,748, 974, 979, 1039, 1050,
1507, 1509, 1511, 1521
type constructor
definition, 173
infix, 679
type declaration, 106
type defaulting, 142, 146,
279–281, 929, 1481, 1681,
1719
type error, 114
type families, 596
type inference, 109, 180, 217,
218, 224, 279, 680, 1719
type inference, definition,
239
type level, 175
type parameter, 161
type signature, 39, 99, 100,
106, 112, 133, 181, 196,
222, 436, 442, 445, 598,
603, 720, 1718
type signature
how to read, 111, 148, 185
type synonym, 106, 442, 443,
1578, see also type alias
type theory, 615

INDEX 1921
type variable, 112, 176, 210,
211, 215, 239, 334, 594,
597, 600, 601
Typeable , 1815, 1823, 1824
TypeApplications , 964, 1068
typechecking, 181
typeclass, 114, 136, 140, 146,
187, 209, 249, 258, 306,
532, 624, 890, 892, 922,
979, 1466
typeclass
definition, 172, 239
dispatched by type, 304,
307, 309
unique pairing, 1039
typeclass constraint, 146, 148,
183, 187, 190, 211, 215,
219, 223, 256, 271, 272,
275, 279, 281, 292, 293,
310, 312–314, 731, 912,
1029, 1054, 1143, 1662,
1673, 1675, 1676, 1679,
1681
typeclass declaration, 305,
307, 960
typeclass deriving, 257, seealso deriving
typeclass hierarchy, 252
typeclass inheritance, 212,
275, 276
typeclass inheritance,
definition, 323
typeclass instance, 249–251,
253, 257–261, 263, 264,
270–272, 289–292, 302,
304–306, 308, 609, 622,
624, 625, 836
typeclass instance
Show, 799
how to read, 262
unique, 923
typeclass instance,
definition, 323
types vs terms, 209, 211, 249,
310, 595, 596, 605, 651
unary, 596, 612, 613, 619, 620
unconditional case, 269
uncurry, 198, 199, 201
uncurry , 1358
undefined, 236, 468, 487,
495, 508, 543, 545, 546,

INDEX 1922
1531, 1658
underscore, 137, 269,
345–347, 354, 375, 496,
503, 801
unfold, 742
Unicode, 100, 1253, 1769, 1771
unit, 299, 301, 782, 839
unit testing, definition, 885
unmarshalling, 1484, see also
serialization
unmarshalling, definition,
1501
UNPACK , 1739
unsafePerformIO , 1807, 1808
URL shortener, 1237
UTC time, 1789
UTF-16, 1764, 1767, 1769
UTF-8, 1253, 1256, 1415, 1764,
1769, 1773–1775, 1874
utf8-string (library), 1774
uuid(library), 1223
Validation , 1132, 1133, 1186
value, 3, 41, 45, 47, 99, 133,
135, 330–333, 595, 599,
600, 604, 618, 1676value, definition, 96
variable, 3, 7, 9, 44, 46, 99,
176, 330, 334
variable
bound, 7, 11, 13
free, 13, 15, 21
naming conventions, 176
single letter, 177
type, see type variable
Vector , 1290, 1741, 1748, 1750,
1752, 1756
Vector , mutable, 1757, 1758
vector, 1767
vector (library), 1290, 1748,
1754
vector
batch updates, 1756
boxed, 1749
slicing, 1750
unboxed, 1749
Vigenère cipher, 692, 1809
Wadler, Philip, 209, 249
Wall, 268
-Wall, 267
warning, 268, 269, 348

INDEX 1923
warning
non-exhaustive patterns,
267, 348
out of range, 142
pattern match overlap,
346
shadowing, 348
weak head normal form, 49,
490, 491, 493, 494, 499,
1217, 1640, 1643, 1654,
1661, 1663, 1712, 1713,
1720, 1721, 1725
web application, 1214, 1215,
1225, 1249
web framework, see scotty
web server, 1256
where, 85, 86, 88, 108, 117, 224,
261, 362, 384whitespace, 59
whnf, see weak head normal
form
whnf, 1712
Windows, 1861
Word8, 1767
words, 794
wreq(library), 1300
writeFile , 1826, 1829
Writer , 1564, 1566
WriterT , 1564, 1566
XML, 1397
xmonad , 1216, 1220
Y combinator, 421, 428
zip, 514, 1121
zipList , 1120, 1123
zipWith , 515, 806, 1152

