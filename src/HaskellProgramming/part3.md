go n (x:xs)=(go (n*x) xs)
niceProduct ::Numa=>[a]->a
niceProduct =foldl' ( *)1

CHAPTER 12. SIGNALING ADVERSITY 744
Remember the redundant structure when we looked at
folds?
mehConcat ::[[a]]->[a]
mehConcat xs=go[]xs
wherego::[a]->[[a]]->[a]
go xs'[]=xs'
go xs' (x :xs)=(go (xs' ++x) xs)
niceConcat ::[[a]]->[a]
niceConcat =foldr (++)[]
This may have given you a mild headache, but you may
also see that this same principle of abstracting out common
patterns and giving them names applies as well to unfolds as
it does to folds.
Write your own iterate and unfoldr
1.Write the function myIterate using direct recursion. Com-
pare the behavior with the built-in iterate to gauge cor-
rectness. Do not look at the source or any examples of
iterate so that you are forced to do this yourself.
myIterate ::(a->a)->a->[a]
myIterate =undefined

CHAPTER 12. SIGNALING ADVERSITY 745
2.Write the function myUnfoldr using direct recursion. Com-
pare with the built-in unfoldr to check your implementa-
tion. Again, don’t look at implementations of unfoldr so
that you figure it out yourself.
myUnfoldr ::(b->Maybe(a, b))
->b
->[a]
myUnfoldr =undefined
3.Rewrite myIterate intobetterIterate usingmyUnfoldr . A
hint — we used unfoldr to produce the same results as
iterate earlier. Do this with diﬀerent functions and see if
you can abstract the structure out.
-- It helps to have the
-- types in front of you
-- myUnfoldr :: (b -> Maybe (a, b))
-- -> b
-- -> [a]
betterIterate ::(a->a)->a->[a]
betterIterate f x=myUnfoldr ...?
Remember, your betterIterate should have the same re-
sults as iterate .

CHAPTER 12. SIGNALING ADVERSITY 746
Prelude> take 10 $ iterate (+1) 0
[0,1,2,3,4,5,6,7,8,9]
Prelude> take 10 $ betterIterate (+1) 0
[0,1,2,3,4,5,6,7,8,9]
Finally something other than a list!
Given the BinaryTree from last chapter, complete the following
exercises. Here’s that datatype again:
dataBinaryTree a=
Leaf
|Node(BinaryTree a) a (BinaryTree a)
deriving (Eq,Ord,Show)
1.Writeunfold forBinaryTree .
unfold::(a->Maybe(a,b,a))
->a
->BinaryTree b
unfold=undefined
2.Make a tree builder.
Usingthe unfold functionyou’vemadefor BinaryTree , write
the following function:

CHAPTER 12. SIGNALING ADVERSITY 747
treeBuild ::Integer ->BinaryTree Integer
treeBuild n=undefined
You should be producing results that look like the following:
Prelude> treeBuild 0
Leaf
Prelude> treeBuild 1
Node Leaf 0 Leaf
Prelude> treeBuild 2
Node (Node Leaf 1 Leaf)
0
(Node Leaf 1 Leaf)
Prelude> treeBuild 3
Node (Node (Node Leaf 2 Leaf)
1
(Node Leaf 2 Leaf))
0
(Node (Node Leaf 2 Leaf)
1
(Node Leaf 2 Leaf))
Or in a slightly diﬀerent representation:
0
0

CHAPTER 12. SIGNALING ADVERSITY 748
/ \
1 1
0
/ \
1 1
/\ /\
2 2 2 2
Good work.
12.6 Definitions
1.Ahigher-kinded type type is any type whose kind has a
function arrow in it and which can be described as a type
constructor rather than a type constant. The following
types are of a higher kind than *:
Maybe:: * -> *
[]:: * -> *
Either:: * -> * -> *
(->):: * -> * -> *
The following are not:

CHAPTER 12. SIGNALING ADVERSITY 749
Int:: *
Char:: *
String:: *
[Char]:: *
This is not to be confused with higher kinded polymor-
phism, which we’ll discuss later.

Chapter 13
Building projects
Wherever there is
modularity there is the
potential for
misunderstanding:
Hiding information
implies a need to check
communication
Alan Perlis
750

CHAPTER 13. BUILDING PROJECTS 751
13.1 Modules
Haskell programs are organized into modules. Modules con-
tain the datatypes, type synonyms, typeclasses, typeclass in-
stances, and values you’ve defined at the top level. They oﬀer
a means to import other modules into the scope of your pro-
gram, and they also contain values that can be exported to
other modules. If you’ve ever used a language with names-
paces, it’s the same thing.
In this chapter, we will be building a small, interactive
hangman-style game. Students of Haskell often ask what kind
of project they should work on as a way to learn Haskell, and
they want to jump right into the kind of program they’re
used to building in the languages they already know. What
most often happens is the student realizes how much they still
don’t understand about Haskell, shakes their fist at the sky, and
curses Haskell’s very name and all the elitist jerks who write
Haskell and flees to relative safety. Nobody wants that. Haskell
is sufficiently diﬀerent from other languages that we think it’s
best to spend time getting comfortable with how Haskell itself
works before trying to build substantial projects.
This chapter’s primary focus is not so much on code but on
how to set up a project in Haskell, use the package manager
known as Cabal, build the project with Stack, and work with
Haskell modules as they are. There are a few times we ask
you to implement part of the hangman game yourself, but

CHAPTER 13. BUILDING PROJECTS 752
much of the code is already written for you, and we’ve tried to
explain the structure as well as we can at this point in the book.
Some of it you won’t properly understand until we’ve covered
at least monads and IO. But if you finish the chapter feeling
like you now know how to set up a project environment and
get things running, then this chapter will have accomplished
its goal and we’ll all go oﬀ and take a much needed mid-book
nap.
Try to relax and have fun with this. You’ve earned it after
those binary tree exercises.
In this chapter, we’ll cover:
•writing Haskell programs with modules;
•using the Cabal package manager;
•building our project with Stack;
•conventions around project organization;
•building a small interactive game.
Note that you’ll need to have Stack1and Git2to follow along
with the instructions in this chapter. We’ll be using gitto
download an example project. Depending on your level of
prior experience, some of this may not be new information
1http://haskellstack.org
2https://git-scm.com/

CHAPTER 13. BUILDING PROJECTS 753
for you. Feel free to move as quickly through this material as
feels comfortable.
13.2 Making packages with Stack
The Haskell Cabal, or Common Architecture for Building Ap-
plications and Libraries, is a package manager. A package is
a program you’re building, including all of its modules and
dependencies, whether you’ve written it or you’re building
someone else’s program. A package has dependencies which are
the interlinked elements of that program, the other packages
and libraries it may depend on and any tests and documenta-
tion associated with the project. Cabal exists to help organize
all this and make sure all dependencies are properly in scope.
Stack is a cross-platform program for developing Haskell
projects. It is aimed at Haskellers both new and experienced,
and it helps you manage both projects made up of multiple
packages as well as individual packages, whereas Cabal exists
primarily to describe a single package with a Cabal file that
has the .cabal file extension.
Stack is built on top of Cabal in some important senses,
so we will still be working with .cabal files. However, Stack
simplifies the process somewhat, especially in large projects
with multiple dependencies, by allowing you to build those
large libraries only once and use them across projects. Stack

CHAPTER 13. BUILDING PROJECTS 754
also relies on an LTS (long term support) snapshot of Haskell
packages from Stackage3that are guaranteed to work together,
unlike packages from Hackage which may have conflicting
dependencies.
While the Haskell community does not have a prescribed
project layout, we recommend the basic structure embodied
in the Stack templates.
13.3 Working with a basic project
We’re going to start learning Cabal and Stack by building a
sample project called hello. To make this less tedious, we’re
going to use gitto checkout the sample project. In an appro-
priate directory for storing your projects, you’ll want to git
clonethe repository https://github.com/haskellbook/hello .
Building the project
Change into the project directory that the git clone invocation
created.
$ cd hello
You could edit the hello.cabal file. There you can replace
“Your Name Here” with…your name. We’ll next build our
project:
3https://www.stackage.org/

CHAPTER 13. BUILDING PROJECTS 755
$ stack build
If it complains about needing GHC to be installed, don’t
panic! Part of the benefit of Stack is that it can manage your
GHC installs for you. Before re-attempting stack build , do the
following:
$ stack setup
Thesetupcommand for Stack determines what version of
GHC you need based on the LTS snapshot specified in the
stack.yaml file of your project. The stack.yaml file is used to
determine the versions of your packages and what version
of GHC they’ll work best with. If you didn’t need to do this,
it’s possible you had a compatible version of GHC already
installed or that you’d run setup for an LTS snapshot that
needed the same version of GHC in the past. To learn more
about this, check out the Stackage website.
Loading and running code from the REPL
Having done that, next we’ll fire up the REPL.
$ stack ghci
[... some other noise...]
Ok, modules loaded: Main.
Prelude> :l Main

CHAPTER 13. BUILDING PROJECTS 756
[1 of 1] Compiling Main
Ok, modules loaded: Main.
Prelude> main
hello world
Above, we successfully started a GHCi REPL that is aware
of our project, loaded our Mainmodule, and then ran the main
function. Using Stack’s GHCi integration to fire up a REPL
doesn’t just let us load and run code in our project, but also
enables us to make use of our project’s dependencies. We’ll
demonstrate this later. indexmain@ main
stack exec
When you ran buildearlier, you may have seen something
like:
Linking .stack-work/dist/{...noise...}/hello
This noise is Stack compiling an executable binary and
linking to it. You can type the full path that Stack mentioned
in order to run the binary, but there’s an easier way — exec!
From our project directory, consider the following:
$ hello
zsh: command not found: hello
$ stack exec -- hello
hello world

CHAPTER 13. BUILDING PROJECTS 757
Stack knows what paths any executables might be located in,
so using Stack’s execcommand saves you the hassle of typing
out a potentially verbose path.
Executable stanzas in Cabal files
Stack created an executable earlier because of the following
stanza in the hello.cabal file:
executable hello
-- [1]
hs-source-dirs: src
-- [2]
main-is: Main.hs
-- [3]
default-language: Haskell2010
-- [4]
build-depends: base >= 4.7 && < 5
-- [5]
1.Thisnamefollowingthedeclarationofan executable stanza
tells Stack or Cabal what to name the binary or executable
it creates.
2.Tells this stanza where to look for source code — in this
case, the srcsubdirectory.

CHAPTER 13. BUILDING PROJECTS 758
3.Execution of this binary should begin by looking for a main
function inside a file named Mainwith the module name
Main. Note that module names have to match filenames.
Your compiler (not just Stack) will reject using a file that
isn’t aMainmodule as the entry point to executing the
program. Also note that it’ll look for the Main.hs file under
all directories you specified in hs-source-dirs . Since we
specified only one, it’ll find this in src/Main.hs , which is
our only source file right now anyway.
4.Defines the version of the Haskell standard to expect. Not
very interesting and doesn’t do much — mostly boiler-
plate, but necessary.
5.This is usually a meatier part of any Cabal stanza, whether
it’s an executable, library, or test suite. This example ( base)
is really the bare minimum or baseline dependency in
almost any Haskell project as you can’t really get anything
done without the baselibrary. We’ll show you how to add
and install dependencies later.
A sidebar about executables and libraries Our project here
only has an executable stanza, which is appropriate for mak-
ing a command-line application which will be run and used.
When we’re writing code we want people to be able to reuse
in other projects, we need a library stanza in the .cabal file
and to choose which modules we want to expose. Executables

CHAPTER 13. BUILDING PROJECTS 759
are applications that the operating system will run directly,
while software libraries are code arranged in a manner so that
they can be reused by the compiler in the building of other
libraries and programs.
13.4 Making our project a library
First we’re going to add a library stanza to hello.cabal :
library
hs-source-dirs: src
exposed-modules: Hello
build-depends: base >= 4.7 && < 5
default-language: Haskell2010
Then we’re going to create a file located at src/Hello.hs :
moduleHellowhere
sayHello ::IO()
sayHello = do
putStrLn "hello world"
Then we’re going to change our Mainmodule to use this
library function:

CHAPTER 13. BUILDING PROJECTS 760
moduleMainwhere
importHello
main::IO()
main= do
sayHello
If we try to build and run this now, it’ll work.
$ stack build
$ stack exec hello
hello world
But what if we had made a separate exedirectory?
$ mkdir exe
$ mv src/Main.hs exe/Main.hs
Then we need to edit the .cabal file to let it know our hello
executable uses the exedirectory:
executable hello
hs-source-dirs: exe
main-is: Main.hs
default-language: Haskell2010
build-depends: base >= 4.7 && < 5

CHAPTER 13. BUILDING PROJECTS 761
If you then attempt to build this, it will fail.
hello/exe/Main.hs:3:8:
Could not find module ‘Hello’
It is a member of the hidden package
‘hello-0.1.0.0@hello_IJIUuynUbgsHAquBKsAsb5’.
Perhaps you need to add ‘hello’ to the
build-depends in your .cabal file.
Use -v to see a list of the files searched for.
We have two paths for fixing this, one better than the other.
One way is to simply add srcto the source directories the
executable is permitted to search. But it turns out that Cabal’s
suggestion here is precisely right. The better way to fix this is
to respect the boundaries of the library and executable and
instead to add your own library as a dependency:
executable hello
hs-source-dirs: exe
main-is: Main.hs
default-language: Haskell2010
build-depends: base >= 4.7 && < 5
, hello
The build will now succeed. This also makes it easier to
know when you need to change what is exposed or exported
in your library, because you’re using your own interface.

CHAPTER 13. BUILDING PROJECTS 762
13.5 Module exports
By default, when you don’t specify any exports in a module,
every top-level binding is exported and can be imported by
another module. This is the case in our Hellomodule:
moduleHellowhere
sayHello ::IO()
sayHello = do
putStrLn "hello world"
But what happens if we specify an empty export list?
moduleHello
()
where
sayHello ::IO()
sayHello = do
putStrLn "hello world"
We’ll get the following error if we attempt to build it:
Not in scope: ‘sayHello’
To fix that explicitly, we add the top-level binding to the
export list:

CHAPTER 13. BUILDING PROJECTS 763
moduleHello
(sayHello )
where
sayHello ::IO()
sayHello = do
putStrLn "hello world"
Now the sayHello function will be exported. It seems point-
less in a module like this, but in bigger projects, it sometimes
makes sense to specify your exports in this way.
Exposing modules
First we’ll add a new module with a new IO action for our main
action to run: indexmain@ main
-- src/DogsRule.hs
moduleDogsRule
(dogs)
where
dogs::IO()
dogs= do
putStrLn "Who's a good puppy?!"
putStrLn "YOU ARE!!!!!"

CHAPTER 13. BUILDING PROJECTS 764
Then we’ll change our Mainmodule to make use of this:
moduleMainwhere
importDogsRule
importHello
main::IO()
main= do
sayHello
dogs
But if we attempt to build this, we’ll get the following error:
Could not find module ‘DogsRule’
As we did earlier with our library stanza, we need to also
expose the DogsRule module:
library
hs-source-dirs: src
exposed-modules: DogsRule
, Hello
build-depends: base >= 4.7 && < 5
default-language: Haskell2010
Now it should be able to find our very important dog prais-
ing.

CHAPTER 13. BUILDING PROJECTS 765
13.6 More on importing modules
Importing modules brings more functions into scope beyond
those available in the standard Prelude . Imported modules
are top-level declarations. The entities imported as part of
thosedeclarations, likeothertop-leveldeclarations, havescope
throughout the module, although they can be shadowed by
local bindings. The eﬀect of multiple import declarations is cu-
mulative, but the ordering of import declarations is irrelevant.
An entity is in scope for the entire module if it is imported by
any of the import declarations.
In previous chapters, we’ve brought functions like booland
toUpper into scope for exercises by importing the modules they
are part of, Data.Bool andData.Char , respectively.
Let’s refresh our memory of how to do this in GHCi. The
:browse command allows us to see what functions are included
in the named module, while importing the module allows
us to use those functions. You can browse modules that you
haven’t imported yet, which can be useful if you’re not sure
which module the function you’re looking for is in:
Prelude> :browse Data.Bool
bool :: a -> a -> Bool -> a
(&&) :: Bool -> Bool -> Bool
data Bool = False | True
not :: Bool -> Bool

CHAPTER 13. BUILDING PROJECTS 766
otherwise :: Bool
(||) :: Bool -> Bool -> Bool
Prelude> import Data.Bool
Prelude> :t bool
bool :: a -> a -> Bool -> a
In the example above, we used an unqualified import of
everything in Data.Bool . What if we only wanted boolfrom
Data.Bool ?
First, we’re going to turn oﬀ Prelude so that we don’t have
any of the default imports. We will use another extension
when we start GHCi to turn Prelude oﬀ. You’ve previously seen
how to use language extensions in source files, but now we’ll
enter-XNoImplicitPrelude right when we enter our REPL:
-- Do this outside of any projects
$ stack ghci --ghci-options -XNoImplicitPrelude
Prelude>
We can check that boolandnotare not in scope yet:
Prelude> :t bool
<interactive>:1:1: Not in scope: ‘bool’
Prelude> :t not

CHAPTER 13. BUILDING PROJECTS 767
<interactive>:1:1: Not in scope: ‘not’
Next we’ll do a selective import from Data.Bool , specifying
that we only want to import bool:
Prelude> import Data.Bool (bool)
Prelude> :t bool
bool :: a -> a -> GHC.Types.Bool -> a
Prelude> :t not
<interactive>:1:1: Not in scope: ‘not’
Now, normally in the Prelude ,notis in scope already but
boolis not. So you can see that by turning oﬀ Prelude , taking
its standard functions out of scope, and then importing only
bool, we no longer have the standard notfunction in scope.
You can import one or more functions from a module or
library. The syntax is just as we demonstrated with GHCi,
but your import declarations have to be at the beginning of
a module. Putting import Data.Char (toUpper) in the import
declarations of a module will ensure that toUpper , but not any
of the other entities contained in Data.Char , is in scope for that
module.
For the examples in the next section, you’ll want Prelude
back on, so please restart GHCi before proceeding.

CHAPTER 13. BUILDING PROJECTS 768
Qualified imports
What if you wanted to know where something you imported
came from in the code that uses it? We can use qualified
imports to make the names more explicit.
We use the qualified keyword in our imports to do this.
Sometimes you’ll have stuﬀ with the same name imported
from two diﬀerent modules; qualifying your imports is a com-
mon way of dealing with this. We’ll go through an example of
how you might use a qualified import.
Prelude> import qualified Data.Bool
Prelude> :t bool
<interactive>:1:1:
Not in scope: ‘bool’
Perhaps you meant ‘Data.Bool.bool’
Prelude> :t Data.Bool.bool
Data.Bool.bool :: a -> a -> Bool -> a
Prelude> :t Data.Bool.not
Data.Bool.not :: Bool -> Bool
In the case of import qualified Data.Bool , everything from
Data.Bool is in scope, but only when accessed with the full

CHAPTER 13. BUILDING PROJECTS 769
Data.Bool namespace. Now we are marking where the func-
tions that we’re using came from, which can be useful.
We can also provide aliases or alternate names for our mod-
ules when we qualify them so we don’t have to type out the
full namespace:
Prelude> import qualified Data.Bool as B
Prelude> :t bool
<interactive>:1:1:
Not in scope: ‘bool’
Perhaps you meant ‘B.bool’
Prelude> :t B.bool
B.bool :: a -> a -> Bool -> a
Prelude> :t B.not
B.not :: Bool -> Bool
You can do qualified imports in the import declarations at
the beginning of your module in the same way.
Setting the Prelude prompt When you imported Data.Bool
asBabove, you may have seen your prompt change:
Prelude> import qualified Data.Bool as B
Prelude B>

CHAPTER 13. BUILDING PROJECTS 770
And if you don’t want to unload the imported modules
(because you want them all to stay in scope), your prompt
could keep growing:
Prelude B> import Data.Char
Prelude B Data.Char>
(Reminder: you can use :mto unload the modules, which
does, of course, prevent the prompt from growing ever larger,
but also, well, unloads the modules so they’re not in scope
anymore!)
If you want to prevent the ever-growing prompt, you can
use the :setcommand to set the prompt to whatever you
prefer:
Prelude> :set prompt "Lambda> "
Lambda> import Data.Char
Lambda> :t B.bool
B.bool :: a -> a -> Bool -> a
As you can see, Data.Bool is still in scope as B, but it doesn’t
show up in our prompt. You can set your Prelude prompt
permanently, if you wish, by changing it in your GHCi config-
uration file, but instructions for doing that are somewhat out
of the scope of the current chapter.

CHAPTER 13. BUILDING PROJECTS 771
Intermission: Check your understanding
Here is the import list from one of the modules in Chris’s
library called blacktip :
import qualified Control.Concurrent
asCC
import qualified Control.Concurrent.MVar
asMV
import qualified Data.ByteString.Char8
asB
import qualified Data.Locator
asDL
import qualified Data.Time.Clock.POSIX
asPSX
import qualified Filesystem
asFS
import qualified Filesystem.Path.CurrentOS
asFPC
import qualified Network.Info
asNI

CHAPTER 13. BUILDING PROJECTS 772
import qualified Safe
importControl.Exception (mask,try)
importControl.Monad (forever,when)
importData.Bits
importData.Bits.Bitwise (fromListBE )
importData.List.Split (chunksOf )
importDatabase.Blacktip.Types
importSystem.IO.Unsafe (unsafePerformIO )
For our purposes right now, it does not matter whether you
are familiar with the modules referenced in the import list.
Look at the declarations and answer the questions below:
1.What functions are being imported from Control.Monad ?
2.Which imports are both unqualified and imported in their
entirety?
3.From the name, what do you suppose importing blacktip ’s
Typesmodule brings in?
4.Now let’s compare a small part of blacktip ’s code to the
above import list:

CHAPTER 13. BUILDING PROJECTS 773
writeTimestamp ::MV.MVarServerState
->FPC.FilePath
->IOCC.ThreadId
writeTimestamp s path= do
CC.forkIO go
wherego=forever $ do
ss<-MV.readMVar s
mask$\_ -> do
FS.writeFile path
(B.pack (show (ssTime ss)))
-- sleep for 1 second
CC.threadDelay 1000000
a)The type signature refers to three aliased imports.
What modules are named in those aliases?
b)Which import does FS.writeFile refer to?
c)Which import did forever come from?
13.7 Making our program interactive
Now we’re going to make our program ask for your name, then
greet you by name. First, we’ll rewrite our sayHello function
to take an argument:

CHAPTER 13. BUILDING PROJECTS 774
sayHello ::String->IO()
sayHello name=
putStrLn ( "Hi "++name++"!")
Note we parenthesized the appending (++)function of the
String argument to putStrLn .
Next we’ll change mainto get the user’s name:
-- src/Main.hs
main::IO()
main= do
name<-getLine
sayHello name
dogs
There are a couple of new things here. We’re using some-
thing called dosyntax, which is syntactic sugar. We use do
inside functions that return IOin order to sequence side eﬀects
in a convenient syntax. Let’s decompose what’s going on here:

CHAPTER 13. BUILDING PROJECTS 775
main::IO()
main= do
-- [1]
name<-getLine
-- [4] [3] [2]
sayHello name
-- [5]
dogs
-- [6]
1.Thedohere begins the block.
2.getLine has type IO String , because it must perform I/O
(input/output, side eﬀects) in order to obtain the String.
getLine is what will allow you to enter your name to be
used in the mainfunction.
3.<-in adoblock is pronounced bind. We’ll explain what
this is and how it works in the chapters on MonadandIO.
4.The result of binding ( <-) over the IO String isString . We
bound it to the variable name. Remember, getLine has type
IO String ,namehas type String .
5.sayHello expects an argument String , which is the type of
namebutnotgetLine .

CHAPTER 13. BUILDING PROJECTS 776
6.dogs4expects nothing and is an IOaction of type IO (),
which fits the overall type of main.
indexmain@ main
Now we’ll fire oﬀ a build:
$ stack build
And run the program:
$ stack exec hello
After you hit enter, the program is going to wait for your
input. You’ll just see the cursor blinking on the line, waiting
for you to enter your name. As soon as you do, and hit enter,
it should greet you and then rave about the wonderfulness of
a dog.
What if we tried to pass getLine to sayHello? If we tried to
writemainwithout the use of dosyntax, particularly without
using<-such as in the following example:
main::IO()
main=sayHello getLine
We’d get the following type error:
4Much like actual dogs.

CHAPTER 13. BUILDING PROJECTS 777
$ stack build
[2 of 2] Compiling Main
src/Main.hs:8:17:
Couldn't match type ‘IO String’ with ‘[Char]’
Expected type: String
Actual type: IO String
In the first argument of ‘sayHello’, namely ‘getLine’
In the expression: sayHello getLine
This is because getLine is anIOaction with type IO String ,
whereas sayHello expects a value of type String. We have to
use<-to bind over the IOto get the string that we want to
pass tosayHello . This will be explained in more detail — a bit
more detail later in the chapter, and a lot more detail in a later
chapter.
Adding a prompt
Let’s make our program a bit easier to use by adding a prompt
that tells us our program is expecting input! We need to change
main:

CHAPTER 13. BUILDING PROJECTS 778
moduleMainwhere
importDogsRule
importHello
importSystem.IO
main::IO()
main= do
hSetBuffering stdout NoBuffering
putStr"Please input your name: "
name<-getLine
sayHello name
dogs
We did several things here. One is that we used putStr in-
stead of putStrLn so that our input could be on the same line as
our prompt. We also imported from System.IO so that we could
usehSetBuffering ,stdout, andNoBuffering . That line of code is
so thatputStr isn’t buﬀered (deferred) and prints immediately.
Rebuild and rerun your program, and it should now work like
this:
$ stack exec hello
Please input your name: julie
Hi julie!
Who's a good puppy?!

CHAPTER 13. BUILDING PROJECTS 779
YOU ARE!!!!!
You can try removing the NoBuffering line (that whole first
line) from mainand rebuilding and running your program
to see how it changes. We will be using this as part of our
hangman game in a bit, but it isn’t necessary at this point to
understand how the buﬀering functions work in any detail.
13.8 do syntax and IO
We touched on donotation a bit above, but we want to explain
a few more things about it. doblocks are convenient syntactic
sugar that allow for sequencing actions, but because they are
only syntactic sugar, they are not, strictly speaking, necessary.
They can make blocks of code more readable and also hide
the underlying nesting, and that can help you write eﬀectful
code before you understand monads and IO. So you’ll see it
a lot in this chapter (and, indeed, you’ll see it quite a bit in
idiomatic Haskell code).
Themainexecutable in a Haskell program must always have
the type IO (). Thedosyntax specifically allows us to sequence
monadic actions .Monadis a typeclass we’ll explain in great detail
in a later chapter; here, the instance of Monad we care about is
IO. That is why mainfunctions are often (not always) doblocks.
indexmain@ main

CHAPTER 13. BUILDING PROJECTS 780
This syntax also provides a way of naming values returned
by monadic IOactions so that they can be used as inputs to
actions that happen later in the program. Let’s look at a very
simpledoblock and try to get a feel for what’s happening here:
main= do
-- [1]
x1<-getLine
-- [2] [3] [4]
x2<-getLine
-- [5]
return (x1 ++x2)
-- [6] [7]
1.dointroduces the block of IO actions.
2.𝑥1is a variable representing the value obtained from the
IO action getLine .
3.<-binds the variable on the left to the result of the IO
action on the right.
4.getLine has the type IO String and takes user input of a
string value. In this case, the string the user inputs will be
the value bound to the 𝑥1name.

CHAPTER 13. BUILDING PROJECTS 781
5.𝑥2is a variable representing the value obtained from our
second getLine . As above it is bound to that value by the
<-.
6.return will be discussed in more detail shortly, but here it
is the concluding action of our doblock.
7.This is the value return , well, returns — the conjunction of
the two strings we obtained from our two getLine actions.
While<-is used to bind a variable, it is diﬀerent from other
methods we’ve seen in earlier chapters for naming and binding
variables. This arrow is part of the special dosugar and specif-
ically binds a name to the 𝑎of anm avalue, where 𝑚is some
monadic structure, in this case IO. The<-allows us to extract
that𝑎and name it within the limited scope of the doblock
and use that named value as an input to another expression
within that same scope. Each assignment using <-creates a
new variable rather than mutating an existing variable because
data is immutable.
return
This function really doesn’t do a lot, but the purpose it serves
is important, given the way monads and IOwork. It does noth-
ing but return a value, but it returns a value inside monadic
structure:

CHAPTER 13. BUILDING PROJECTS 782
Prelude> :t return
return :: Monad m => a -> m a
For our purposes in this chapter, return returns a value in
IO. Because the obligatory type of mainisIO (), the final value
must also have an IO ()type, and return gives us a way to add
no extra function except putting the final value in IO. If the
final action of a doblock is return () , that means there is no
real value to return at the end of performing the I/O actions,
but since Haskell programs can’t return literally nothing, they
return this empty tuple called unit simply to have something
to return. That empty tuple will not print to the screen in the
REPL, but it’s there in the underlying representation.
Let’s take a look at return in action. Let’s say you want to get
user input of two characters and test them for equality. You
can’t do this:
twoo::IOBool
twoo= doc<-getChar
c'<-getChar
c==c'
Try it and see what your type error looks like. It should
tell you that it can’t match the expected type IO Bool with the
actual type of c == c' , which is Bool. So, our final line needs to
return that Boolvalue in IO:

CHAPTER 13. BUILDING PROJECTS 783
twoo::IOBool
twoo= doc<-getChar
c'<-getChar
return (c ==c')
We put the Boolvalue into IOby using return. Cool. How
about if we have cases where we want to return nothing? We’ll
reuse the same basic code from above but make an if-then-else
within our doblock:
main::IO()
main= doc<-getChar
c'<-getChar
ifc==c'
thenputStrLn "True"
elsereturn()
What happens when the two input characters are equal?
What happens when they aren’t?
Some people have noted that dosyntax makes it feel like
you’re doing imperative programming in Haskell. It’s impor-
tant to note that this eﬀectful imperative style requires having
IOin our result type. We cannot perform eﬀects without evi-
dence of having done so in the type. dois only syntactic sugar,
but the monadic syntax we’ll cover in a later chapter works in
a similar way for monads other than IO.

CHAPTER 13. BUILDING PROJECTS 784
Do notation considered harmful! Just kidding. But some-
times enthusiastic programmers overuse doblocks. It is not
necessary, and considered bad style, to use doin single-line
expressions. You will eventually learn to use >>=in single-
line expressions instead of do(there’s an example of that in
this chapter). Similarly, it is unnecessary to use dowith func-
tions like putStrLn andprintthat already have the eﬀects baked
in. In the function above, we could have put doin front of
bothputStrLn andreturn and it would have worked the same,
but things get messy and the Haskell ninjas will come and be
severely disappointed in you.
13.9 Hangman game
Now we’re ready to build a game. We’ll use Stack’s newcom-
mand to create this project:
$ stack new hangman simple
That will generate a directory named hangman for you and
some put some default files into the directory.
You need a wordsfile for getting words from. Most Unix-
based operating systems will have a words list located at a
directory like the following:
$ ls /usr/share/dict/
american-english british-english

CHAPTER 13. BUILDING PROJECTS 785
cracklib-small README.select-wordlist
words words.pre-dictionaries-common
In this case, we’ll use the wordsword list which should be
your operating system’s default. You may have one that is
diﬀerently located, or you may need to download one. We
put it in the working directory at data/dict.txt :
$ tree .
.
├── LICENSE
├── Setup.hs
├── data
│ └── dict.txt
├── hangman.cabal
├── src
│ └── Main.hs
└── stack.yaml
The file was newline separated and so looked like:
$ head data/dict.txt
A
a
aa
aal
aalii

CHAPTER 13. BUILDING PROJECTS 786
aam
Aani
aardvark
aardwolf
Aaron
Now edit the .cabal file as follows:
name: hangman
version: 0.1.0.0
synopsis: Playing Hangman
homepage: Chris N Julie
license: BSD3
license-file: LICENSE
author: Chris Allen and Julie Moronuki
maintainer: haskellbook.com
category: Game
build-type: Simple
extra-source-files: data/dict.txt
cabal-version: >=1.10
executable hangman
main-is: Main.hs
hs-source-dirs: src
build-depends: base >=4.7 && <5
, random
, split
default-language: Haskell2010

CHAPTER 13. BUILDING PROJECTS 787
The important bit here is that we used two libraries: random
andsplit. Normally you’d do version ranges for your depen-
dencies like you see with base, but we left the versions of random
andsplitunassigned because they do not change much. The
primary and only source file was in src/Main.hs .
13.10 Step One: Importing modules
-- src/Main.hs
moduleMainwhere
importControl.Monad (forever)-- [1]
importData.Char (toLower)-- [2]
importData.Maybe (isJust)-- [3]
importData.List (intersperse )-- [4]
importSystem.Exit (exitSuccess )-- [5]
importSystem.Random (randomRIO )-- [6]
Here the imports are enumerated in the source code. For
your version of this project, you don’t need to add the enumer-
ating comments. All modules listed below are part of the main
baselibrary that comes with your GHC install unless otherwise
noted.

CHAPTER 13. BUILDING PROJECTS 788
1.We’re using forever fromControl.Monad to make an infinite
loop. A couple points to note:
a)You don’t haveto useforever to do this, but we’re going
to.
b)You are not expected to understand what it does or
how it works exactly. Basically it allows us to execute
a function over and over again, infinitely, or until we
cause the program to exit or fail, instead of evaluating
once and then stopping.
2.We will use toLower fromData.Char to convert all characters
of our string to lowercase:
Prelude> import Data.Char (toLower)
Prelude> toLower 'A'
'a'
Be aware that if you pass a character that doesn’t have a
sensible lowercase, toLower will kick the same character
back out:
Prelude> toLower ':'
':'

CHAPTER 13. BUILDING PROJECTS 789
3.We will use isJust fromData.Maybe to determine if every
character in our puzzle has been discovered already or
not:
Prelude> import Data.Maybe (isJust)
Prelude> isJust Nothing
False
Prelude> isJust (Just 10)
True
We will combine this with all, a standard function in the
Prelude . Hereallis a function which answers the question,
“given a function that will return True or False for each
element, does it return True for allof them?”
Prelude> all even [2, 4, 6]
True
Prelude> all even [2, 4, 7]
False
Prelude> all isJust [Just 'd', Nothing, Just 'g']
False
Prelude> all isJust [Just 'd', Just 'o', Just 'g']
True
The function allhas the type:

CHAPTER 13. BUILDING PROJECTS 790
Foldable t=>(a->Bool)->t a->Bool
We haven’t explained the Foldable typeclass. For your
purposes you can assume it’s a set of operations for types
that can be folded in a manner conceptually similar to
the list type but which don’t necessarily contain more than
one value (or any values at all) the way a list or similar
datatype does. We can make the type more specific by
asserting a type signature like so:
Prelude> :t all :: (a -> Bool) -> [a] -> Bool
all :: (a -> Bool) -> [a] -> Bool
This will work for any type which has a Foldable instance:
Prelude> :t all :: (a -> Bool) -> Maybe a -> Bool
all :: (a -> Bool) -> Maybe a -> Bool
-- note the type variables used and
-- experiment independently
Prelude> :t all :: (a -> Bool) -> Either b a -> Bool
all :: (a -> Bool) -> Either b a -> Bool
But it will not work if the datatype doesn’t have an instance
ofFoldable :

CHAPTER 13. BUILDING PROJECTS 791
Prelude> :t all :: (a -> Bool) -> (b -> a) -> Bool
No instance for (Foldable ((->) b1)) arising
from a use of ‘all’
In the expression:
all :: (a -> Bool) -> (b -> a) -> Bool
4.Weuseintersperse fromData.List to…intersperseelements
in a list. In this case, we’re putting spaces between the
characters guessed so far by the player. You may remem-
ber we used intersperse back in the Recursion chapter to
put hyphens in our Numbers Into Words exercise:
Prelude> import Data.List (intersperse)
Prelude> intersperse ' ' "Blah"
"B l a h"
Conveniently, the type of intersperse says nothing about
characters or strings, so we can use it with lists containing
elements of any type:
Prelude> :t intersperse
intersperse :: a -> [a] -> [a]
Prelude> intersperse 0 [1, 1, 1]

CHAPTER 13. BUILDING PROJECTS 792
[1,0,1,0,1]
5.We use exitSuccess fromSystem.Exit to exit successfully —
no errors, we’re simply done. We indicate whether it was
a success or not so our operating system knows whether
an error occurred. Note that if you evaluate exitSuccess
in the REPL, it’ll report that an exception occurred. In a
normal running program that doesn’t catch the exception,
it’ll end your whole program.
6.We use randomRIO fromSystem.Random to select a word from
our dictionary at random. System.Random is in the library
random . Once again, you’ll need to have the library in scope
for your REPL to be able to load it. Once it’s in scope,
we can use randomRIO to get a random number. You can
see from the type signature that it takes a tuple as an
argument, but it uses the tuple as a range from which to
select a random item:
Prelude> import System.Random
Prelude System.Random> :t randomRIO
randomRIO :: Random a => (a, a) -> IO a
Prelude System.Random> randomRIO (0, 5)
4
Prelude System.Random> randomRIO (1, 100)
71

CHAPTER 13. BUILDING PROJECTS 793
Prelude System.Random> randomRIO (1, 100)
12
We will later use this random number generation to pro-
duce a random index of a word list to provide a means of
selecting random words for our puzzle.
13.11 Step Two: Generating a word list
For clarity’s sake, we’re using a type synonym to declare what
we mean by [String] in our types. Later we’ll show you a
version that’s even more explicit using newtype . We also use do
syntax to read the contents of our dictionary into a variable
named dict. We use the linesfunction to split our big blob
string we read from the file into a list of string values each
representing a single line. Each line is a single word, so our
result is the WordList :
typeWordList =[String]
allWords ::IOWordList
allWords = do
dict<-readFile "data/dict.txt"
return (lines dict)
Let’s take a moment to look at lines, which splits strings at
the newline marks and returns a list of strings:

CHAPTER 13. BUILDING PROJECTS 794
Prelude> lines "aardvark\naaron"
["aardvark","aaron"]
Prelude> length $ lines "aardvark\naaron"
2
Prelude> length $ lines "aardvark\naaron\nwoot"
3
Prelude> lines "aardvark aaron"
["aardvark aaron"]
Prelude> length $ lines "aardvark aaron"
1
Note that this does something similar but diﬀerent from
wordswhich splits by spaces (ostensibly between words) and
newlines:
Prelude> words "aardvark aaron"
["aardvark","aaron"]
Prelude> words "aardvark\naaron"
["aardvark","aaron"]
The next part of building our word list for our puzzle is to
set upper and lower bounds for the size of words we’ll use in
the puzzles. Feel free to change them if you want:

CHAPTER 13. BUILDING PROJECTS 795
minWordLength ::Int
minWordLength =5
maxWordLength ::Int
maxWordLength =9
The next thing we’re going to do is take the output of
allWords and filter it to fit the length criteria we defined above.
That will give us a shorter list of words to use in the puzzles:
gameWords ::IOWordList
gameWords = do
aw<-allWords
return (filter gameLength aw)
wheregameLength w =
letl=length (w ::String)
inl>=minWordLength
&&l<maxWordLength
We next need to write a pair of functions that will pull a
random word out of our word list for us, so that the puzzle
player doesn’t know what the word will be. We’re going to use
therandomRIO function we mentioned above to facilitate that.
We’ll pass randomRIO a tuple of zero (the first indexed position
in our word list) and the number that is the length of our word
list minus one. Why minus one?

CHAPTER 13. BUILDING PROJECTS 796
We have to subtract one from the length of the word list
in order to index it because length starts counting from 1 but
an index of the list starts from 0. A list of length 5 does not
have a member indexed at position 5 — it has inhabitants at
positions 0-4 instead:
Prelude> [1..5] !! 4
5
Prelude> [1..5] !! 5
*** Exception: Prelude.(!!): index too large
In order to get the last value in the list, then, we must ask
for the member in the position of the length of the list minus
one:
Prelude> let myList = [1..5]
Prelude> length myList
5
Prelude> myList !! length myList
*** Exception: Prelude.!!: index too large
Prelude> myList !! (length myList - 1)
5
The next two functions work together to pull a random
word out of the gameWords list we had created above. Roughly
speaking, randomWord generates a random index number based

CHAPTER 13. BUILDING PROJECTS 797
on the length of a word list, wl, and then selects the member
of that list that is at that indexed position and returns an IO
String. Given what you know about randomRIO and indexing,
you should be able to supply the tuple argument to randomRIO
yourself:
randomWord ::WordList ->IOString
randomWord wl= do
randomIndex <-randomRIO ( , )
-- fill this part in ^^^
return$wl!!randomIndex
The second function, randomWord' binds the gameWords list to
therandomWord function so that the random word we’re getting
is from that list. We’re going to delay a full discussion of the
>>=operator known as “bind” until we get to the Monadchapter.
For now, we can say that, as we said about dosyntax, bindallows
us to sequentially compose actions such that a value generated
by the first becomes an argument to the second:
randomWord' ::IOString
randomWord' =gameWords >>=randomWord
Now that we have a word list, we turn our attention to the
building of an interactive game using it.

CHAPTER 13. BUILDING PROJECTS 798
13.12 Step Three: Making a puzzle
Our next step is to formulate the core game play. We need a
way to hide the word from the player (while giving them an
indication of how many letters it has) and create a means of
asking for letter guesses, determining if the guessed letter is
in the word, putting it in the word if it is and putting it into
an “already guessed” list if it’s not, and determining when the
game ends.
We start with a datatype for our puzzle. The puzzle is a
product of aString , a list of Maybe Char , and a list of Char:
dataPuzzle=
PuzzleString[MaybeChar] [Char]
-- [1] [2] [3]
1.the word we’re trying to guess
2.the characters we’ve filled in so far
3.the letters we’ve guessed so far
Next we’re going to write an instance of the typeclass Show
for our datatype Puzzle. You may recall that showallows us to
print human-readable stringy things to the screen, which is
obviously something we have to do to interact with our game.
But we want it to print our puzzle a certain way, so we define
this instance.

CHAPTER 13. BUILDING PROJECTS 799
Notice how the argument to showlines up with our datatype
definition above. Now discovered refers to our list of Maybe Char
andguessed is what we’ve named our list of Char, but we’ve
done nothing with the String itself:
instance ShowPuzzlewhere
show (Puzzle_discovered guessed) =
(intersperse ' '$
fmap renderPuzzleChar discovered)
++" Guessed so far: " ++guessed
This is going to show us two things as part of our puzzle:
the list of Maybe Char which is the string of characters we have
correctly guessed and the rest of the characters of the puzzle
word represented by underscores, interspersed with spaces;
and a list of Charthat reminds us of which characters we’ve
already guessed. We’ll talk about renderPuzzleChar below.
First we’re going to write a function that will take our puzzle
word and turn it into a list of Nothing . This is the first step in
hiding the word from the player. We’re going to ask you to
write this one yourself, using the following information:
•We’ve given you a type signature. Your first argument is a
String , which will be the word that is in play. It will return
a value of type Puzzle . Remember that the Puzzle type is a
product of three things.

CHAPTER 13. BUILDING PROJECTS 800
•Your first value in the output will be the same string as
the argument to the function.
•The second value will be the result of mapping a function
over that String argument. Consider using constin the
mapped function, as it will always return its first argument,
no matter what its second argument is.
•For purposes of this function, the final argument of Puzzle
is an empty list.
Go for it:
freshPuzzle ::String->Puzzle
freshPuzzle =undefined
Now we need a function that looks at the Puzzle String and
determines whether the character you guessed is an element
of that string. Here are some hints:
•This is going to need two arguments, and one of those
is of type Puzzle which is a product of 3 types. But for
the purpose of this function, we only care about the first
argument to Puzzle .
•We can use underscores to signal that there are values
we don’t care about and tell the function to ignore them.
Whether you use underscores to represent the arguments

CHAPTER 13. BUILDING PROJECTS 801
you don’t care about or go ahead and put the names of
those in won’t aﬀect the result of the function. It does,
however, keep your code a bit cleaner and easier to read
by explicitly signaling which arguments you care about
in a given function.
•The standard function elemworks like this:
Prelude> :t elem
elem :: Eq a => a -> [a] -> Bool
Prelude> elem 'a' "julie"
False
Prelude> elem 3 [1..5]
True
So, here you go:
charInWord ::Puzzle->Char->Bool
charInWord =undefined
The next function is very similar to the one you just wrote,
but this time we don’t care if the Charis part of the String
argument — this time we want to check and see if it is an
element of the guessed list.
You’ve totally got this:
alreadyGuessed ::Puzzle->Char->Bool
alreadyGuessed =undefined

CHAPTER 13. BUILDING PROJECTS 802
OK, so far we have ways to choose a word that we’re trying
to guess and determine if a guessed character is part of that
word or not. But we need a way to hide the rest of the word
from the player while they’re guessing. Computers are a bit
dumb, after all, and can’t figure out how to keep secrets on
their own. Back when we defined our Showinstance for this
puzzle, we fmapped a function called renderPuzzleChar over
our second Puzzle argument. Let’s work on that function next.
The goal here is to use Maybeto permit two diﬀerent out-
comes. It will be mapped over a string in the typeclass instance,
so this function works on only one character at a time. If that
character has not been correctly guessed yet, it’s a Nothing value
and should appear on the screen as an underscore. If the char-
acter has been guessed, we want to display that character so
the player can see which positions they’ve correctly filled:
Prelude> renderPuzzleChar Nothing
'_'
Prelude> renderPuzzleChar (Just 'c')
'c'
Prelude> let n = Nothing
Prelude> let daturr = [n, Just 'h', n, Just 'e', n]
Prelude> fmap renderPuzzleChar daturr
"_h_e_"
Your turn. Remember, you don’t need to do the mapping
part of it here:

CHAPTER 13. BUILDING PROJECTS 803
renderPuzzleChar ::MaybeChar->Char
renderPuzzleChar =undefined
The next bit is a touch tricky. The point is to insert a cor-
rectly guessed character into the string. Although none of the
components here are new to you, they’re put together in a
somewhat dense manner, so we’re going to unpack it (obvi-
ously, when you type this into your own file, you do not need
to add the enumerations):

CHAPTER 13. BUILDING PROJECTS 804
fillInCharacter ::Puzzle->Char->Puzzle
fillInCharacter (Puzzleword
-- [1]
filledInSoFar s) c =
-- [2]
Puzzleword newFilledInSoFar (c :s)
-- [ 3 ]
wherezipper guessed wordChar guessChar =
-- [4] [5] [6] [7]
ifwordChar ==guessed
thenJustwordChar
elseguessChar
-- [ 8 ]
newFilledInSoFar =
-- [9]
zipWith (zipper c)
word filledInSoFar
-- [ 10 ]
1.The first argument is our Puzzle with its three arguments,
with𝑠representing the list of characters already guessed.
2.The𝑐is ourCharargument and is the character the player
guessed on this turn.
3.Our result is the Puzzle with the filledInSoFar replaced by

CHAPTER 13. BUILDING PROJECTS 805
newFilledInSoFar the𝑐consed onto the front of the 𝑠list.
4.zipper is a combining function for deciding how to handle
the character in the word, what’s been guessed already,
and the character that was just guessed. If the current
character in the word is equal to what the player guessed,
then we go ahead and return Just wordChar to fill in that
spot in the puzzle. Otherwise, we kick the guessChar back
out. We kick guessChar back out because it might either be
a previously correctly guessed character oraNothing that
has not been guessed correctly this time nor in the past.
5.guessed is the character they guessed.
6.wordChar is the characters in the puzzle word — not the
ones they’ve guessed or not guessed, but the characters
in the word that they’re supposed to be guessing.
7.guessChar is the list that keeps track of the characters the
player has guessed so far.
8.Thisif-then-else expression checks to see if the guessed
character is one of the word characters. If it is, it wraps it
in aJustbecause our puzzle word is a list of Maybevalues.
9.newFilledInSoFar is the new state of the puzzle which uses
zipWith and the zipper combining function to fill in char-
acters in the puzzle. The zipper function is first applied to

CHAPTER 13. BUILDING PROJECTS 806
the character the player just guessed because that doesn’t
change. Then it’s zipped across two lists. One list is word
which is the word the user is trying to guess. The second
list,filledInSoFar is the puzzle state we’re starting with of
type[Maybe Char] . That’s telling us which characters in
wordhave been guessed.
10.Now we’re going to make our newFilledInSoFar by using
zipWith . You may remember this from the Lists chapter.
It’s going to zip the wordwith the filledInSoFar values while
applying the zipper function from just above it to the
values as it does.
Next we have this big doblock with a case expression and
each case also has a doblock inside it. Why not, right?
First, it tells the player what you guessed. The case ex-
pression is to give diﬀerent responses based on whether the
guessed character:
•had already been guessed previously;
•is in the word and needs to be filled in;
•or, was not previously guessed but also isn’t in the puzzle
word.
Despite the initial appearance of complexity, most of this
is syntax you’ve seen before, and you can look through it step-
by-step and see what’s going on:

CHAPTER 13. BUILDING PROJECTS 807
handleGuess ::Puzzle->Char->IOPuzzle
handleGuess puzzle guess = do
putStrLn $"Your guess was: " ++[guess]
case(charInWord puzzle guess
, alreadyGuessed puzzle guess) of
(_,True)-> do
putStrLn "You already guessed that \
\character, pick \
\something else!"
return puzzle
(True,_)-> do
putStrLn "This character was in the \
\word, filling in the word \
\accordingly"
return (fillInCharacter puzzle guess)
(False,_)-> do
putStrLn "This character wasn't in \
\the word, try again."
return (fillInCharacter puzzle guess)
All right, next we need to devise a way to stop the game
after a certain number of guesses. Hangman games normally
stop only after a certain number of incorrect guesses, but for
the sake of simplicity here, we’re stopping after a set number
of guesses, whether they’re correct or not. Again, the syntax

CHAPTER 13. BUILDING PROJECTS 808
here should be comprehensible to you from what we’ve done
so far:
gameOver ::Puzzle->IO()
gameOver (PuzzlewordToGuess _guessed) =
if(length guessed) >7then
doputStrLn "You lose!"
putStrLn $
"The word was: " ++wordToGuess
exitSuccess
elsereturn()
Notice the way it’s written says you lose and exits the game
once you’ve guessed seven characters, even if the final (seventh)
guess is the final letter to fill into the word. There are, of course,
ways to modify that to make it more the way you’d expect a
hangman game to go, and we encourage you to play with that.
Next we need to provide a way to exit after winning the
game. We showed you how the combination of isJust andall
works earlier in the chapter, and you can see that in action
here. Recall that our puzzle word is a list of Maybevalues, so
when each character is represented by a Just Char rather than
aNothing , you win the game and we exit:

CHAPTER 13. BUILDING PROJECTS 809
gameWin ::Puzzle->IO()
gameWin (Puzzle_filledInSoFar _)=
ifall isJust filledInSoFar then
doputStrLn "You win!"
exitSuccess
elsereturn()
Next is the instruction for running a game. Here we use
forever so that this will execute this series of actions indef-
initely:
runGame ::Puzzle->IO()
runGame puzzle=forever $ do
gameOver puzzle
gameWin puzzle
putStrLn $
"Current puzzle is: " ++show puzzle
putStr"Guess a letter: "
guess<-getLine
caseguessof
[c]->handleGuess puzzle c >>=runGame
_ ->
putStrLn "Your guess must \
\be a single character"
And, finally, here is mainbringing everything together: it

CHAPTER 13. BUILDING PROJECTS 810
gets a word from the word list we generated, generates a fresh
puzzle, and then executes the runGame actions we saw above,
until such time as you guess all the characters in the word
correctly or have made seven guesses, whichever comes first:
indexmain@ main
main::IO()
main= do
word<-randomWord'
letpuzzle=
freshPuzzle (fmap toLower word)
runGame puzzle
13.13 Adding a newtype
Another way you could modify your code in the above and
gain, perhaps, more clarity in places is with the use of newtype:
-- replace this type synonym
-- type WordList = [String]
newtype WordList =
WordList [String]
deriving (Eq,Show)

CHAPTER 13. BUILDING PROJECTS 811
allWords ::IOWordList
allWords = do
dict<-readFile "data/dict.txt"
return$WordList (lines dict)
gameWords ::IOWordList
gameWords = do
(WordList aw)<-allWords
return$WordList (filter gameLength aw)
wheregameLength w =
letl=length (w ::String)
inl>minWordLength
&&l<maxWordLength
randomWord ::WordList ->IOString
randomWord (WordList wl)= do
randomIndex <-
randomRIO ( 0, (length wl) -1)
return$wl!!randomIndex
13.14 Chapter exercises
Hangman game logic
You may have noticed when you were playing with the hang-
man game, that there are some weird things about its game

CHAPTER 13. BUILDING PROJECTS 812
logic:
•although it can play with words up to 9 characters long,
you only get to guess 7 characters;
•it ends the game after 7 guesses, whether they were correct
or incorrect;
•if your 7th guess supplies the last letter in the word, it may
still tell you you lost;
•it picks some very strange words that you didn’t suspect
were even in the dictionary.
These make it unlike hangman as you might have played it
in the past. Ordinarily, only incorrect guesses count against
you, so you can make as many correct guesses as you need
to fill in the word. Modifying the game so that it either gives
you more guesses before the game ends or only uses shorter
words (or both) involves only a couple of uncomplicated steps.
A bit more complicated but worth attempting as an exercise
is changing the game so that, as with normal hangman, only
incorrect guesses count towards the guess limit.
Modifying code
1.Ciphers: Open your Ciphers module and modify it so that
the Caesar and Vigenère ciphers work with user input.

CHAPTER 13. BUILDING PROJECTS 813
2.Here is a very simple, short block of code. Notice it has
aforever that will make it keep running, over and over
again. Load it into your REPL and test it out. Then refer
back to the chapter and modify it to exit successfully after
a False result.
importControl.Monad
palindrome ::IO()
palindrome =forever $ do
line1<-getLine
case(line1==reverse line1) of
True->putStrLn "It's a palindrome!"
False->putStrLn "Nope!"
3.If you tried using palindrome on a sentence such as “Madam
I’mAdam,”youmayhavenoticedthatpalindromechecker
doesn’t work on that. Modifying the above so that it works
on sentences, too, involves several steps. You may need
to refer back to previous examples in the chapter to get
ideas for proper ordering and nesting. You may wish to
import Data.Char to use the function toLower . Have fun.

CHAPTER 13. BUILDING PROJECTS 814
4.typeName=String
typeAge=Integer
dataPerson=PersonNameAgederiving Show
dataPersonInvalid =
NameEmpty
|AgeTooLow
|PersonInvalidUnknown String
deriving (Eq,Show)
mkPerson ::Name
->Age
->EitherPersonInvalid Person
mkPerson name age
|name/=""&&age>0=
Right$Personname age
|name==""=LeftNameEmpty
|not (age >0)=LeftAgeTooLow
|otherwise =
Left$PersonInvalidUnknown $
"Name was: " ++show name ++
" Age was: " ++show age
Your job is to write the following function without modi-

CHAPTER 13. BUILDING PROJECTS 815
fying the code above.
gimmePerson ::IO()
gimmePerson =undefined
SinceIO ()is about the least informative type imaginable,
we’ll tell what it should do.
a)It should prompt the user for a name and age input.
b)It should attempt to construct a Person value using
the name and age the user entered. You’ll need the
readfunction for Age because it’s an Integer rather
than a String.
c)If it constructed a successful person, it should print
”Yay! Successfully got a person:” followed by the Per-
son value.
d)If it got an error value, report that an error occurred
and print the error.
13.15 Follow-up resources
1.Stack
https://github.com/commercialhaskell/stack
2.How I Start: Haskell
http://bitemyapp.com/posts/2014-11-18-how-i-start-haskell.
html

CHAPTER 13. BUILDING PROJECTS 816
3.Cabal FAQ
https://www.haskell.org/cabal/FAQ.html
4.Cabal user’s guide
https://www.haskell.org/cabal/users-guide/
5.A Gentle Introduction to Haskell, Modules chapter.
https://www.haskell.org/tutorial/modules.html

Chapter 14
Testing
We’ve tended to forget
that no computer will
ever ask a new question.
Grace Murray Hopper
817

CHAPTER 14. TESTING 818
14.1 Testing
This chapter, likethe one before it, is more focused on practical
matters rather than writing Haskell code per se. We will be
covering two testing libraries (there are others) and how and
when to use them. You will not be writing much of the code
in the chapter on your own; instead, please follow along by
entering it into files as directed (you will learn more if you
type rather than copy and paste). At the end of the chapter,
there are a number of exercises that ask you to write your own
tests for practice.
Testing is a core part of the working programmer’s toolkit,
and Haskell is no exception. Well-specified types can enable
programmers to avoid many obvious and tedious tests that
mightotherwisebenecessarytomaintaininuntypedprogram-
ming languages, but there’s still a lot of value to be obtained
in executable specifications. This chapter will introduce you
to testing methods for Haskell.
This chapter will cover:
•the whats and whys of testing;
•using the testing libraries HspecandQuickCheck ;
•a bit of fun with Morse code.

CHAPTER 14. TESTING 819
14.2 A quick tour of testing for the
uninitiated
When we write Haskell, we rely on the compiler to judge for
us whether our code is well formed. That prevents a great
number of errors, but it does not prevent them all. It is still
possible to write well-typed code that doesn’t perform as ex-
pected, and runtime errors can still occur. That’s where testing
comes in.
In general, tests allow you to state an expectation and then
verify that the result of an operation meets that expectation.
They allow you to verify that your code will do what you want
when executed.
For the sake of simplicity, we’ll say there are two broad cate-
gories of testing: unit testing and property testing. Unit testing
tests the smallest atomic units of software independently of
one another. Unit testing allows the programmer to check that
each function is performing the task it is meant to do. You
assert that when the code runs with a specified input, the result
is equal to the result you want.
Spec testing is a somewhat newer version of unit testing.
Like unit testing, it tests specific functions independently and
asksyoutoassertthat, whengiventhedeclaredinput, theresult
of the operation will be equal to the desired result. When you
run the test, the computer checks that the expected result is

CHAPTER 14. TESTING 820
equal to the actual result and everyone moves on with their day.
Some people prefer spec testing to unit testing because spec
testing is more often written in terms of assertions that are in
human-readable language. This can be especially valuable if
nonprogrammers need to be able to read and interpret the
results of the tests — they can read the English-language results
of the tests and, in some cases, write tests themselves.
Haskell provides libraries for both unit and spec testing.
We’ll focus on specification testing with the hspeclibrary in
this chapter, but HUnitis also available. One limitation to unit
and spec testing is that they test atomic units of code indepen-
dently, so they do not verify that all the pieces work together
properly.
Property testing is a diﬀerent beast. This kind of testing
was pioneered in Haskell because the type system and straight-
forward logic of the language lend themselves to property
tests, but it has since been adopted by other languages as well.
Property tests test the formal properties of programs without
requiring formal proofs by allowing you to express a truth-
valued, universally quantified (that is, will apply to all cases)
function — usually equality — which will then be checked
against randomly generated inputs.
The inputs are generated randomly by the standard func-
tions inside the QuickCheck library we use for property testing.
This relies on the type system to know what kinds of data to
generate. The default setting is for 100 inputs to be generated,

CHAPTER 14. TESTING 821
giving you 100 results. If it fails any one of these, then you
know your program doesn’t have the specified property. If
it passes, you can’t be positive it will never fail because the
data are randomly generated — there could be a weird edge
case out there that will cause your software to fail. QuickCheck is
cleverly written to be as thorough as possible and will usually
check the most common edge cases (for example, empty lists
and the maxBound andminBound s of the types in question, where
appropriate). You can also change the setting so that it runs
more tests.
Property testing is fantastic for ensuring that you’ve met
the minimum requirements to satisfy laws, such as the laws
of monads or basic associativity. It is not appropriate for all
programs, though, as it is not useful for times when there are
no assertable, truth-valued properties of the software.
14.3 Conventional testing
We are going to use the library hspec1to demonstrate a test
case, but we’re not going to explain hspecdeeply. The current
chapter will equip you with a means of writing tests for your
code later, but it’s not necessary to understand the details of
how the library works to do that. Some of the concepts hspec
leans on, such as functor, applicative, and monad, are covered

CHAPTER 14. TESTING 822
later as independent concepts.
First, let’s come up with a test case for addition. Generally
we want to make a Cabal project, even for small experiments.
Having a permanent project for experiments can eliminate
some of this overhead, but we’ll assume you haven’t done this
yet and start a small project:
-- addition.cabal
name: addition
version: 0.1.0.0
license-file: LICENSE
author: Chicken Little
maintainer: sky@isfalling.org
category: Text
build-type: Simple
cabal-version: >=1.10
library
exposed-modules: Addition
ghc-options: -Wall -fwarn-tabs
build-depends: base >=4.7 && <5
, hspec
hs-source-dirs: .
default-language: Haskell2010
1http://hackage.haskell.org/package/hspec

CHAPTER 14. TESTING 823
Note we’ve specified the hspecdependency, but not a version
range for it. You’ll probably want whatever the newest version
of it is but can probably get away with not specifying it for
now.
Next we’ll make the Addition module (exposed-modules) in
thesamedirectoryasourCabalfile. Thisiswhythe hs-source-dirs
option in the library stanza was set to .— this is the convention
for referring to the current directory.
For now, we’ll write a simple placeholder function to make
sure everything’s working:
-- Addition.hs
moduleAddition where
sayHello ::IO()
sayHello =putStrLn "hello!"
Then you can create an empty LICENSE file so the build
doesn’t complain:
$ touch LICENSE
Your local project directory should look like this now, before
having run any Stack commands:
$ tree
.

CHAPTER 14. TESTING 824
├── Addition.hs
└── addition.cabal
└── LICENSE
The next steps are to initialize the Stack file for describing
what snapshot of Stackage we’ll use:
$ stack init
Then we’ll want to build our project which will also install
the dependencies we need:
$ stack build
If that succeeded, let’s fire up a REEEEEEEPL and see if we
can call sayHello :
$ stack ghci
[some noise about configuring, loading packages, etc.]
Ok, modules loaded: Addition.
Prelude> sayHello
hello!
If you got here, you’ve got a working test bed for making a
simple test case in hspec!

CHAPTER 14. TESTING 825
Truth according to Hspec
Next we’ll add the import of hspec’s primary module:
moduleAddition where
importTest.Hspec
sayHello ::IO()
sayHello =putStrLn "hello!"
Note that allof your imports must occur after the module
has been declared and before any expressions have been de-
fined in your module. You may have encountered an error or
a mistake might’ve been made. Here are a couple of examples.
moduleAddition where
sayHello ::IO()
sayHello =putStrLn "hello!"
importTest.Hspec
Here we put an import after at least one declaration. The
compiler parser doesn’t have a means of recognizing this spe-
cific mistake, so it can’t tell you properly what the error is:

CHAPTER 14. TESTING 826
Prelude> :r
[1 of 1] Compiling Addition
Addition.hs:7:1: parse error on input ‘import’
Failed, modules loaded: none.
What else may have gone wrong? Well, we might have the
package hspecinstalled, but not included in our build-depends
for our project. Note you’ll need to quit and reopen the REPL
if you’ve made any changes to your .cabal file to reproduce
this error or fixed a mistake:
$ stack build
{... noise ...}
Could not find module ‘Test.Hspec’
It is a member of the hidden package
‘hspec-2.2.3@hspec_JWyjr3DNMsw1kiPzf88M5w’.
Perhaps you need to add ‘hspec’ to the
build-depends in your .cabal file.
Use -v to see a list of the files searched for.
{... other noise ...}
Process exited with code: ExitFailure 1

CHAPTER 14. TESTING 827
If you changed anything in order to test these error modes,
you’ll need to add hspecback to your build-depends and reinstall
it. Ifhspecis listed in your dependencies, stack build will set
you right.
Assuming everything is in order and Test.Hspec is being
imported, we can do a little exploration. We can use the :browse
command to get a listing of types from a module and get a
thousand-foot-view of what it oﬀers:
Prelude> :browse Test.Hspec
context :: String -> SpecWith a -> SpecWith a
example :: Expectation -> Expectation
specify :: Example a => String -> a -> SpecWith (Arg a)
(... list goes on for awhile ..)
Prelude>
:browse is more useful when you already have some famil-
iarity with the library and how it works. When you’re using
an unfamiliar library, documentation is easier to digest. Good
documentation explains how important pieces of the library
work and gives examples of their use. This is especially valu-
able when encountering new concepts. As it happens, hspec
has some pretty good documentation at their website.2
2http://hspec.github.io/

CHAPTER 14. TESTING 828
Our first Hspec test
Let’s add a test assertion to our module now. If you glance
at the documentation, you’ll see that our example isn’t very
interesting, but we’ll make it somewhat more interesting soon:
moduleAddition where
importTest.Hspec
main::IO()
main=hspec$ do
describe "Addition" $ do
it"1 + 1 is greater than 1" $ do
(1+1)>1`shouldBe` True
We’ve asserted in both English and code that (1 + 1) should
be greater than 1, and that is what hspecwill test for us. You
may recognize the donotation from the previous chapter. As
we said then, this syntax allows us to sequence monadic actions.
In the previous chapter, the monad in question was IO.
Here, we’re nesting multiple doblocks. The types of the do
blocks passed to hspec,describe , anditaren’tIO ()but some-
thing more specific to hspec. They result in IO ()in the end, but
there are other monads involved. We haven’t covered monads
yet, and this works fine without understanding precisely how
it works, so let’s just roll with it for now.

CHAPTER 14. TESTING 829
Note that you’ll get warnings about the Num a => a literals
getting defaulted to Integer . You can ignore this or add explicit
type signatures, it is up to you. With the above code in place,
we can load or reload our module and run mainto see the test
results:
Prelude> main
Addition
1 + 1 is greater than 1
Finished in 0.0041 seconds
1 example, 0 failures
OK, so what happened here? Basically, hspecruns your code
and verifies that the arguments you passed to shouldBe are
equal. Let’s look at the types:
shouldBe ::(Eqa,Showa)
=>a->a->Expectation
-- contrast with
(==)::Eqa=>a->a->Bool
In a sense, it’s an augmented ==embedded in hspec’s model
of the universe. It needs the Showinstance in order to render a

CHAPTER 14. TESTING 830
value. That is, the Showinstance allows hspecto show you the
result of the tests, not just return a Boolvalue.
Let’s add another test, one that reads a little diﬀerently:
main::IO()
main=hspec$ do
describe "Addition" $ do
it"1 + 1 is greater than 1" $ do
(1+1)>1`shouldBe` True
it"2 + 2 is equal to 4" $ do
2+2`shouldBe` 4
Modify your describe block about Addition so that it looks
like the above and run it in the REPL:
Prelude> main
Addition
1 + 1 is greater than 1
2 + 2 is equal to 4
Finished in 0.0004 seconds
2 examples, 0 failures
For fun, we’ll look back to something you wrote early in the
book and write a short hspectest for it. Back in the Recursion

CHAPTER 14. TESTING 831
chapter, we wrote our own division function that looked like
this:
dividedBy ::Integral a=>a->a->(a, a)
dividedBy num denom =go num denom 0
wherego n d count
|n<d=(count, n)
|otherwise =
go (n-d) d (count +1)
We want to test that to see that it works as it should. To keep
things simple, we added dividedBy to ourAddition.hs file and
then rewrote the hspectests that were already there. We want
to test that the function is both subtracting the correct number
of times and keeping an accurate count of that subtraction
and also that it’s telling us the correct remainder, so we’ll give
hspectwo things to test for:
main::IO()
main=hspec$ do
describe "Addition" $ do
it"15 divided by 3 is 5" $ do
dividedBy 153`shouldBe` ( 5,0)
it"22 divided by 5 is \
\4 remainder 2" $ do
dividedBy 225`shouldBe` ( 4,2)

CHAPTER 14. TESTING 832
That’s it. When we reload Addition.hs in our REPL, we can
test our division function:
*Addition> main
Addition
15 divided by 3 is 5
22 divided by 5 is 4 remainder 2
Finished in 0.0012 seconds
2 examples, 0 failures
Hurrah! We can do arithmetic!
Intermission: Short Exercise
In the Chapter Exercises at the end of Recursion, you were
given this exercise:
Write a function that multiplies two numbers using recur-
sive summation. The type should be (Eq a, Num a) => a -> a
-> aalthough, depending on how you do it, you might also
consider adding an Ordconstraint.
If you still have your answer, great! If not, rewrite it and
then write hspectests for it.
The above examples demonstrate the basics of writing in-
dividual tests to test particular values. If you’d like to see a

CHAPTER 14. TESTING 833
more developed example, you could refer to Chris’s library,
Bloodhound.3
14.4 Enter QuickCheck
hspecdoes a nice job with spec testing, but we’re Haskell users
— we’re never satisfied!! hspeccan only prove something about
particular values. Can we get assurances that are stronger,
something closer to proofs? As it happens, we can.
QuickCheck was the first library to oﬀer what is today called
property testing. hspectesting is more like what is known
as unit testing — the testing of individual units of code —
whereas property testing is done with the assertion of laws or
properties.
First, we’ll need to add QuickCheck to ourbuild-depends . Open
your.cabal file and add it. Be sure to capitalize QuickCheck (un-
likehspec, which begins with a lowercase ℎ). It should already
be installed, as hspechasQuickCheck as a dependency, but you
may need to reinstall it ( stack build ). Then open a new stack
ghcisession.
hspechasQuickCheck integration out of the box, so once that
is done, add the following to your module:
3https://github.com/bitemyapp/bloodhound

CHAPTER 14. TESTING 834
-- with your imports
importTest.QuickCheck
-- to the same describe block as the others
it"x + 1 is always \
\greater than x" $ do
property $\x->x+1>(x::Int)
If we had not asserted the type of 𝑥in the property test, the
compiler would not have known what concrete type to use,
and we’d see a message like this:
No instance for (Show a0) arising from a use of ‘property’
The type variable ‘a0’ is ambiguous
...
No instance for (Num a0) arising from a use of ‘+’
The type variable ‘a0’ is ambiguous
...
No instance for (Ord a0) arising from a use of ‘>’
The type variable ‘a0’ is ambiguous
Avoid this by asserting a concrete type, for example, (x ::
Int), in the property.
Assuming all is well, when we run it, we’ll see something
like the following:
Prelude> main

CHAPTER 14. TESTING 835
Addition
1 + 1 is greater than 1
2 + 2 is equal to 4
x + 1 is always greater than x
Finished in 0.0067 seconds
3 examples, 0 failures
What’s being hidden a bit by hspecis that QuickCheck tests
manyvalues to see if your assertions hold for all of them. It
does this by randomly generating values of the type you said
you expected. So, it’ll keep feeding our function random Int
values to see if the property is ever false. The number of tests
QuickCheck runs defaults to 100.
Arbitrary instances
QuickCheck relies on a typeclass called Arbitrary and anewtype
calledGenfor generating its random data.
arbitrary is a value of type Gen:
Prelude> :t arbitrary
arbitrary :: Arbitrary a => Gen a
This is a way to set a default generator for a type. When
you use the arbitrary value, you have to specify the type to

CHAPTER 14. TESTING 836
dispatch the right typeclass instance, as types and typeclass
instances form unique pairings. But this is just a value. How
do we see a list of values of the correct type?
We can use sample andsample' from the Test.QuickCheck mod-
ule in order to see some random data:
-- this prints each value on a new line
Prelude> :t sample
sample :: Show a => Gen a -> IO ()
-- this one returns a list
Prelude> :t sample'
sample' :: Gen a -> IO [a]
TheIOis necessary because it’s using a global resource of
random values to generate the data. A common way to gener-
ate pseudorandom data is to have a function that, given some
input “seed” value, returns a value and another seed value for
generating a diﬀerent value. You can bind the two actions
together, as we explained in the last chapter, to pass a new seed
value each time and keep generating seemingly random data.
In this case, however, we’re not doing that. Here we’re using
IOso that our function that generates our data can return a
diﬀerent result each time (not something pure functions are
allowed to do) by pulling from a global resource of random
values. If this doesn’t make a great deal of sense at this point,

CHAPTER 14. TESTING 837
it will be more clear once we’ve covered monads, and even
more so once we cover IO.
We use the Arbitrary typeclass in order to provide a genera-
tor forsample. It isn’t a terribly principled typeclass, but it is
popular and useful for this. We say it is unprincipled because
it has no laws and nothing specific it’s supposed to do. It’s a
convenient way of plucking a canonical generator for Gen a
out of thin air without having to know where it comes from.
If it feels a bit like *MAGICK* at this point, that’s fine. It is, a
bit, and the inner workings of Arbitrary are not worth fussing
over right now.
As you’ll see later, this isn’t necessary if you have a Genvalue
ready to go already. Genis a newtype with a single type argu-
ment. It exists for wrapping up a function to generate pseudo-
random values. The function takes an argument that is usually
provided by some kind of random value generator to give you
a pseudorandom value of that type, assuming it’s a type that
has an instance of the Arbitrary typeclass.
And this is what we get when we use the sample functions.
We use the arbitrary value but specify the type, so that it gives
us a list of random values of that type:
Prelude> sample (arbitrary :: Gen Int)
0
-2
-1

CHAPTER 14. TESTING 838
4
-3
4
2
4
-3
2
-4
Prelude> sample (arbitrary :: Gen Double)
0.0
0.13712502861905426
2.9801894108743605
-8.960645064542609
4.494161946149201
7.903662448338119
-5.221729489254451
31.64874305324701
77.43118278366954
-539.7148886375935
26.87468214215407
If you run sample arbitrary directly in GHCi without speci-
fying a type, it will default the type to ()and give you a very
nice list of empty tuples. If you try loading an unspecified
sample arbitrary from a source file, though, you will get an af-
fectionate message from GHC about having an ambiguous

CHAPTER 14. TESTING 839
type. Try it if you like. GHCi has somewhat diﬀerent rules for
default types than GHC does.
We can specify our own data for generating Genvalues. In
this example, we’ll specify a trivial function that always returns
a1of type Int:
-- trivial generator of values
trivialInt ::GenInt
trivialInt =return1
You may remember return from the previous chapter as
well. Here, it providesan expedientwayto construct a function.
Inthelastchapter, wenotedthatitdoesn’tdoawholelotexcept
return a value inside of a monad. Before we were using it to
put a value into IObut it’s not limited to use with that monad:
return::Monadm=>a->m a
-- when `m` is Gen:
return::a->Gena
Putting 1into the Genmonad constructs a generator that
always returns the same value, 1.
So, what happens when we sample data from this?

CHAPTER 14. TESTING 840
Prelude> sample' trivialInt
[1,1,1,1,1,1,1,1,1,1,1]
Notice now our value isn’t arbitrary for some type, but the
trivialInt value we defined above. That generator always re-
turns1, so allsample' can return for us is a list of 1.
Let’s explore diﬀerent means of generating values:
oneThroughThree ::GenInt
oneThroughThree =elements [ 1,2,3]
Try loading that via your Addition module and asking for a
sample set of random oneThroughThree values:
*Addition> sample' oneThroughThree
[2,3,3,2,2,1,2,1,1,3,3]
Yep, it gave us random values from only that limited set.
At this time, each number in that set has the same chance of
showing up in our random data set. We could tinker with
those odds by having a list with repeated elements to give
those elements a higher probability of showing up in each
generation:
oneThroughThree ::GenInt
oneThroughThree =
elements [ 1,2,2,2,2,3]

CHAPTER 14. TESTING 841
Try running sample' again with this set and see if you no-
tice the diﬀerence. You may not, of course, because due to
the nature of probability, there is at least some chance that
2wouldn’t show up any more than it did with the previous
sample.
Next we’ll use choose andelements from the QuickCheck library
as generators of values:
-- choose :: System.Random.Random a
-- => (a, a) -> Gen a
-- elements :: [a] -> Gen a
genBool ::GenBool
genBool =choose ( False,True)
genBool' ::GenBool
genBool' =elements [ False,True]
genOrdering ::GenOrdering
genOrdering =elements [ LT,EQ,GT]
genChar ::GenChar
genChar =elements [ 'a'..'z']
You should enter all these into your Addition module, load
them into your REPL, and play with getting lists of sample

CHAPTER 14. TESTING 842
data for each.
Our next examples are a bit more complex:
genTuple ::(Arbitrary a,Arbitrary b)
=>Gen(a, b)
genTuple = do
a<-arbitrary
b<-arbitrary
return (a, b)
genThreeple ::(Arbitrary a,Arbitrary b,
Arbitrary c)
=>Gen(a, b, c)
genThreeple = do
a<-arbitrary
b<-arbitrary
c<-arbitrary
return (a, b, c)
Here’s how to use generators when they have polymor-
phic type arguments. Remember that if you leave the types
unspecified, the extended defaulting behavior of GHCi will
(helpfully?) pick ())for you. Outside of GHCi, you’ll get an
error about an ambiguous type — we covered some of this
when we explained typeclasses earlier:

CHAPTER 14. TESTING 843
Prelude> sample genTuple
((),())
((),())
((),())
Here it’s defaulting the 𝑎and𝑏to(). We can get more
interesting output if we tell it what we expect 𝑎and𝑏to be.
Note it’ll always pick 0 and 0.0 for the first numeric values:
Prelude> sample (genTuple :: Gen (Int, Float))
(0,0.0)
(-1,0.2516606)
(3,0.7800742)
(5,-61.62875)
We can ask for lists and characters, or anything with an
instance of the Arbitrary typeclass:
Prelude> sample (genTuple :: Gen ([()], Char))
([],'\STX')
([()],'X')
([],'?')
([],'\137')
([(),()],'\DC1')
([(),()],'z')
You can use :info Arbitrary in your GHCi to see what in-
stances are available.

CHAPTER 14. TESTING 844
We can also generate arbitrary MaybeandEither values:
genEither ::(Arbitrary a,Arbitrary b)
=>Gen(Eithera b)
genEither = do
a<-arbitrary
b<-arbitrary
elements [ Lefta,Rightb]
-- equal probability
genMaybe ::Arbitrary a=>Gen(Maybea)
genMaybe = do
a<-arbitrary
elements [ Nothing,Justa]
-- What QuickCheck does so
-- you get more Just values
genMaybe' ::Arbitrary a=>Gen(Maybea)
genMaybe' = do
a<-arbitrary
frequency [ ( 1, return Nothing)
, (3, return ( Justa))]
-- frequency :: [(Int, Gen a)] -> Gen a
For now, you should play with this in the REPL; it will

CHAPTER 14. TESTING 845
become useful to know later on.
Using QuickCheck without Hspec
We can also use QuickCheck without hspec. In that case, we no
longer need to specify 𝑥in our expression, because the type
ofprop_additionGreater provides for it. Thus, we rewrite our
previous example as follows:
prop_additionGreater ::Int->Bool
prop_additionGreater x=x+1>x
runQc::IO()
runQc=quickCheck prop_additionGreater
For now, we don’t need to worry about how runQcdoes its
work. It’s a generic function, like main, that signals that it’s time
to do stuﬀ. Specifically, in this case, it’s time to perform the
QuickCheck tests.
Now, when we run it in the REPL, instead of the mainwe were
calling with hspec, we’ll call runQc, which will call on QuickCheck
to test the property we defined. When we run QuickCheck di-
rectly, it reports how many tests it ran:
Prelude> runQc
+++ OK, passed 100 tests.
What happens if we assert something untrue?

CHAPTER 14. TESTING 846
prop_additionGreater x=x+0>x
Prelude> :r
[1 of 1] Compiling Addition
Ok, modules loaded: Addition.
Prelude> runQc
*** Failed! Falsifiable (after 1 test):
0
Conveniently, QuickCheck doesn’t only tell us that our test
failed, but it tells us the first input it encountered that it failed
on. If you try to keep running it, you may notice that the
value that it fails on is always 0. A while ago, we said that
QuickCheck has some built-in cleverness and tries to ensure that
common error boundaries will always get tested. The input 0
is a frequent point of failure, so QuickCheck tries to ensure that
it is always tested (when appropriate, given the types, etc etc).
14.5 Morse code
In the interest of playing with testing, we’ll work through an
example project where we translate text to and from Morse
code. We’re going to start a new project for this. When you
do usestack new project-name to start a new project instead of
stack init for an existing project, it automatically generates a
file called Setup.hs that looks like this:

CHAPTER 14. TESTING 847
importDistribution.Simple
main=defaultMain
This isn’t terribly important. You rarely need to modify
or do anything at all with the Setup.hs file, and usually you
shouldn’t touch it at all. Occasionally, you may need to edit it
for certain tasks, so it is good to recognize that it’s there.
Next, as always, let’s get our .cabal file configured properly.
Some of this will be automatically generated by your stack new
project-name , but you’ll have to add to what it generates, being
careful about things like capitalization and indentation:
name: morse
version: 0.1.0.0
license-file: LICENSE
author: Chris Allen
maintainer: cma@bitemyapp.com
category: Text
build-type: Simple
cabal-version: >=1.10
library
exposed-modules: Morse
ghc-options: -Wall -fwarn-tabs
build-depends: base >=4.7 && <5
, containers

CHAPTER 14. TESTING 848
, QuickCheck
hs-source-dirs: src
default-language: Haskell2010
executable morse
main-is: Main.hs
ghc-options: -Wall -fwarn-tabs
hs-source-dirs: src
build-depends: base >=4.7 && <5
, containers
, morse
, QuickCheck
default-language: Haskell2010
test-suite tests
ghc-options: -Wall -fno-warn-orphans
type: exitcode-stdio-1.0
main-is: tests.hs
hs-source-dirs: tests
build-depends: base
, containers
, morse
, QuickCheck
default-language: Haskell2010
Don’t forget to capitalize the QuickCheck dependency prop-

CHAPTER 14. TESTING 849
erly! Now that is set up and ready for us, so the next step is
to make our srcdirectory and the file called Morse.hs as our
“exposed module:”
-- src/Morse.hs
moduleMorse
(Morse
,charToMorse
,morseToChar
,stringToMorse
,letterToMorse
,morseToLetter
)where
import qualified Data.Map asM
typeMorse=String
Whoa, there — what’s all that stuﬀ after the module name?
That is a list of everything this module will export. We talked
a bit about this in the previous chapter, but didn’t make use of
it. In the hangman game, we had all our functions in one file,
so nothing needed to be exported.

CHAPTER 14. TESTING 850
Nota bene You don’t have to specify exports in this manner.
By default, the entire module is exposed and can be imported
by any other module. If you want to export everything in a
module, then specifying exports is unnecessary. However, it
can help, when managing large projects, to specify what will
get used by another module (and, by exclusion, what will not)
as a way of documenting your intent. In this case, we have
exported here more than we imported into Main, as we realized
that we only needed the two specified functions for Main. We
could go back and remove the things we didn’t specifically
import from the above export list, but we haven’t now, to give
you an idea of the process we’re going through putting our
project together.
Turning words into code
We are also using a qualified import of Data.Map . We covered
this type of import somewhat in the previous chapter. We
qualify the import and name it 𝑀so that we can use that 𝑀
as a prefix for the functions we’re using from that package.
That will help us keep track of where the functions came from
and also avoid same-name clashes with Prelude functions, but
without requiring us to tediously type Data.Map as a prefix to
each function name.
We’ll talk more about Mapas a data structure later in the book.
For now, we can understand it as being a balanced binary tree,

CHAPTER 14. TESTING 851
where each node is a pairing of a key and a value. The key is
an index for the value — a marker of how to find the value
in the tree. The key must be orderable (that is, must have an
Ordinstance), much like our binary tree functions earlier, such
asinsert , needed an Ordinstance. Maps can be more efficient
than lists because you do not have to search linearly through
a bunch of data. Because the keys are ordered and the tree is
balanced, searching through the binary tree divides the search
space in half each time you go “left” or “right.” You compare
the key to the index of the current node to determine if you
need to go left (less), right (greater), or if you’ve arrived at the
node for your value (equal).
You can see below why we used a Mapinstead of a simple list.
We want to make a list of pairs, where each pair includes both
the English-language character and its Morse code represen-
tation. We define our transliteration table thus:
letterToMorse ::(M.MapCharMorse)
letterToMorse =M.fromList [
('a',".-")
, ('b',"-...")
, ('c',"-.-.")
, ('d',"-..")
, ('e',".")

CHAPTER 14. TESTING 852
, ('f',"..-.")
, ('g',"--.")
, ('h',"....")
, ('i',"..")
, ('j',".---")
, ('k',"-.-")
, ('l',".-..")
, ('m',"--")
, ('n',"-.")
, ('o',"---")
, ('p',".--.")
, ('q',"--.-")
, ('r',".-.")
, ('s',"...")
, ('t',"-")
, ('u',"..-")
, ('v',"...-")
, ('w',".--")
, ('x',"-..-")
, ('y',"-.--")
, ('z',"--..")
, ('1',".----")
, ('2',"..---")

CHAPTER 14. TESTING 853
, ('3',"...--")
, ('4',"....-")
, ('5',".....")
, ('6',"-....")
, ('7',"--...")
, ('8',"---..")
, ('9',"----.")
, ('0',"-----")
]
Note that we used M.fromList — the𝑀prefix tells us this
comes from Data.Map . We’re using a Mapto associate characters
with their Morse code representations. letterToMorse is the def-
inition of the Mapwe’ll use to look up the codes for individual
characters.
Next we write a few functions that allow us to convert a
Morse character to an English character and vice versa, and
also functions to do the same for strings:

CHAPTER 14. TESTING 854
morseToLetter ::M.MapMorseChar
morseToLetter =
M.foldWithKey (flip M.insert) M.empty
letterToMorse
charToMorse ::Char->MaybeMorse
charToMorse c=
M.lookup c letterToMorse
stringToMorse ::String->Maybe[Morse]
stringToMorse s=
sequence $fmap charToMorse s
morseToChar ::Morse->MaybeChar
morseToChar m=
M.lookup m morseToLetter
Notice we used Maybein three of those: not every Charthat
could potentially occur in a String has a Morse representation.
The Main event
Next we want to set up a Mainmodule that will handle our
Morse code conversions. Note that it’s going to import a bunch
of things, some of which we covered in the last chapter and
some we have not. Since we will not be going into the specifics

CHAPTER 14. TESTING 855
of how this code works, we won’t discuss those imports here.
It is, however, important to note that one of our imports is our
Morse.hs module from above:
-- src/Main.hs
moduleMainwhere
importControl.Monad (forever,when)
importData.List (intercalate )
importData.Traversable (traverse )
importMorse(stringToMorse ,morseToChar )
importSystem.Environment (getArgs)
importSystem.Exit (exitFailure ,
exitSuccess )
importSystem.IO (hGetLine ,hIsEOF,stdin)
As we said, we’re not going to explain this part in detail.
We encourage you to do your best reading and interpreting
it, but it’s quite dense, and this chapter isn’t about this code
— it’s about the tests. We’re cargo-culting a bit here, which
we don’t like to do, but we’re doing it so that we can focus on
the testing. Type this all into your Mainmodule — first the
function to convert to Morse:

CHAPTER 14. TESTING 856
convertToMorse ::IO()
convertToMorse =forever $ do
weAreDone <-hIsEOF stdin
when weAreDone exitSuccess
-- otherwise, proceed.
line<-hGetLine stdin
convertLine line
where
convertLine line = do
letmorse=stringToMorse line
casemorseof
(Juststr)
->putStrLn
(intercalate " "str)
Nothing
-> do
putStrLn $"ERROR: " ++line
exitFailure
Now add the function to convert from Morse:

CHAPTER 14. TESTING 857
convertFromMorse ::IO()
convertFromMorse =forever $ do
weAreDone <-hIsEOF stdin
when weAreDone exitSuccess
-- otherwise, proceed.
line<-hGetLine stdin
convertLine line
where
convertLine line = do
letdecoded ::MaybeString
decoded =
traverse morseToChar
(words line)
casedecoded of
(Justs)->putStrLn s
Nothing -> do
putStrLn $"ERROR: " ++line
exitFailure
And now our obligatory main:

CHAPTER 14. TESTING 858
main::IO()
main= do
mode<-getArgs
casemodeof
[arg]->
caseargof
"from"->convertFromMorse
"to"->convertToMorse
_ -> argError
_ ->argError
whereargError = do
putStrLn "Please specify the \
\first argument \
\as being 'from' or \
\'to' morse, \
\such as: morse to"
exitFailure
Make sure it’s all working
One way we can make sure everything is working for us from
the command line is by using echo. If this is familiar to you
and you feel comfortable with this, go ahead and try this:
$ echo "hi" | stack exec morse to

CHAPTER 14. TESTING 859
.... ..
$ echo ".... .." | stack exec morse from
hi
If you’d like to find out where Stack put the executable, you
can use stack exec which morse on Mac and Linux. You can also
usestack install to ask Stack to build (if needed) and copy the
binaries from your project into a common directory. On Mac
and Linux that will be .local/bin in your home directory. The
location was chosen partly to respect XDG4guidelines.
Otherwise, load this module into your GHCi REPL and
give it a try to ensure everything compiles and seems to be in
working order. It’ll be helpful to fix any type or syntax errors
now, before we start trying to run the tests.
Time to test!
Now we need to write our test suite. We have those in their
own directory and file. We will again call the module Main
but note the file name (the name per se isn’t important, but
it must agree with the test file you have named in your Cabal
configuration for this project):
4https://wiki.archlinux.org/index.php/Xdg_user_directories

CHAPTER 14. TESTING 860
-- tests/tests.hs
moduleMainwhere
import qualified Data.Map asM
importMorse
importTest.QuickCheck
We have many fewer imports for this, which should all
already be familiar to you.
Now we set up our generators for ensuring that the random
valuesQuickCheck uses to test our program are sensible for our
Morse code program:
allowedChars ::[Char]
allowedChars =M.keys letterToMorse
allowedMorse ::[Morse]
allowedMorse =M.elems letterToMorse
charGen ::GenChar
charGen =elements allowedChars
morseGen ::GenMorse
morseGen =elements allowedMorse

CHAPTER 14. TESTING 861
We saw elements briefly above. It takes a list of some type
— in these cases, our lists of allowed characters and Morse
characters — and chooses a Genvalue from the values in that
list. Because Charincludes thousands of characters that have
no legitimate equivalent in Morse code, we need to write our
own custom generators.
Now we write up the property we want to check. We want
to check that when we convert something to Morse code and
then back again, it comes out as the same string we started out
with:
prop_thereAndBackAgain ::Property
prop_thereAndBackAgain =
forAll charGen
(\c->((charToMorse c)
>>=morseToChar) ==Justc)
main::IO()
main=quickCheck prop_thereAndBackAgain
This is how your setup should look when you have all this
done:
$ tree
.
├── LICENSE

CHAPTER 14. TESTING 862
├── Setup.hs
├── morse.cabal
├── src
│ ├── Main.hs
│ └── Morse.hs
├── stack.yaml
└── tests
└── tests.hs
Testing the Morse code
Now that our conversions seem to be working, let’s run our
tests to make sure. The property we’re testing is that we get the
same string after we convert it to Morse and back again. Let’s
load up our tests by opening a REPL from our main project
directory:
$ stack ghci morse:tests
{... noise noise noise ...}
Ok, modules loaded: Main.
Prelude>
Sweet. Stack loaded everything for us and even built our
dependencies if needs be. Let’s see what happens:

CHAPTER 14. TESTING 863
Prelude> main
+++ OK, passed 100 tests.
The test generates 100 random Morse code conversions
(a bunch of random strings) and makes sure they are always
equal once you have converted to and then from Morse code.
This gives you a pretty strong assurance that your program is
correct and will perform as expected for any input value.
14.6 Arbitrary instances
One of the more important parts of QuickCheck is learning to
write instances of the Arbitrary typeclass for your datatypes.
It’s a somewhat unfortunate but still necessary convenience
for your code to integrate cleanly with QuickCheck code. It’s
initially a bit confusing for beginners because it compacts a
few diﬀerent concepts and solutions to problems into a single
typeclass.
Babby’s First Arbitrary
First, we’ll begin with a maximally simple Arbitrary instance
for theTrivial datatype:

CHAPTER 14. TESTING 864
moduleMainwhere
importTest.QuickCheck
dataTrivial =
Trivial
deriving (Eq,Show)
trivialGen ::GenTrivial
trivialGen =
returnTrivial
instance Arbitrary Trivial where
arbitrary =trivialGen
Thereturn is necessary to return Trivial in theGenmonad:
main::IO()
main= do
sample trivialGen
Let’s take a sample:
Prelude> sample trivialGen
Trivial
Trivial
Trivial

CHAPTER 14. TESTING 865
Trivial
Trivial
Trivial
Trivial
Trivial
Trivial
Trivial
Trivial
Although it’s impossible to see the point with Trivial by it-
self,Genvalues are generators of random values that QuickCheck
uses to get test values from.
Identity Crisis
This one is a little diﬀerent. It will produce random values
even if the Identity structure itself doesn’t and cannot vary.
dataIdentity a=
Identity a
deriving (Eq,Show)
identityGen ::Arbitrary a=>
Gen(Identity a)
identityGen = do
a<-arbitrary
return ( Identity a)

CHAPTER 14. TESTING 866
We’re using the Genmonad to pluck a single value of type
𝑎out of the air, embed it in Identity , then return as part of
theGenmonad. We know this is weird, but if you do it ten or
twenty times you might start to like it.
We’ll reuse the original identityGen we wrote. We can make
it the default generator for the Identity type by making it the
arbitrary value in the Arbitrary instance:
instance Arbitrary a=>
Arbitrary (Identity a)where
arbitrary =identityGen
identityGenInt ::Gen(Identity Int)
identityGenInt =identityGen
We’re making a generator suitable for sampling by making
the type argument of Identity unambiguous for testing with
thesample function. Your output in the terminal could look
something like:
Prelude> sample identityGenInt
Identity 0
Identity (-1)
Identity 2
Identity 4
Identity (-3)

CHAPTER 14. TESTING 867
Identity 5
Identity 3
Identity (-1)
Identity 12
Identity 16
Identity 0
You should be able to change the concrete type of Identity ’s
type argument and generate diﬀerent types of sample values.
Arbitrary Products
Arbitrary instances for product types get a teensy bit more
interesting, but they’re really an extension of what we did for
Identity :
dataPaira b=
Paira b
deriving (Eq,Show)
pairGen ::(Arbitrary a,
Arbitrary b)=>
Gen(Paira b)
pairGen = do
a<-arbitrary
b<-arbitrary
return ( Paira b)

CHAPTER 14. TESTING 868
We will reuse our pairGen function as the arbitrary value in
the instance:
instance (Arbitrary a,
Arbitrary b)=>
Arbitrary (Paira b)where
arbitrary =pairGen
pairGenIntString ::Gen(PairIntString)
pairGenIntString =pairGen
And now we can generate some sample values:
Pair 0 ""
Pair (-2) ""
Pair (-3) "26"
Pair (-5) "B\NUL\143:\254\SO"
Pair (-6) "\184*\239\DC4"
Pair 5 "\238\213=J\NAK!"
Pair 6 "Pv$y"
Pair (-10) "G|J^"
Pair 16 "R"
Pair (-7) "("
Pair 19 "i\ETX]\182\ENQ"
Ah, the beauty of random String values.

CHAPTER 14. TESTING 869
Greater than the sum of its parts
Writing Arbitrary instances for sum types is a bit more inter-
esting still. First, make sure the following is included in your
imports:
importTest.QuickCheck.Gen (oneof)
Sum types represent disjunction, so with a sum type like
Sum, we need to represent the exclusive possibilities in our Gen.
One way to do that is to pull out as many arbitrary values
as you require for the cases of your sum type. We have two
data constructors in this sum type, so we’ll want two arbitrary
values. Then we’ll repack them into Genvalues, resulting in a
value of type [Gen a] that can be passed to oneof:

CHAPTER 14. TESTING 870
dataSuma b=
Firsta
|Secondb
deriving (Eq,Show)
-- equal odds for each
sumGenEqual ::(Arbitrary a,
Arbitrary b)=>
Gen(Suma b)
sumGenEqual = do
a<-arbitrary
b<-arbitrary
oneof [return $Firsta,
return$Secondb]
Theoneoffunction will create a Gen afrom a list of Gen aby
giving each value an equal probability. From there, you’re
delegating to the Arbitrary instances of the types 𝑎and𝑏.
sumGenCharInt ::Gen(SumCharInt)
sumGenCharInt =sumGenEqual
We specify which Arbitrary instances to use for 𝑎and𝑏and
do a test run:
Prelude> sample sumGenCharInt

CHAPTER 14. TESTING 871
First 'P'
First '\227'
First '\238'
First '.'
Second (-3)
First '\132'
Second (-12)
Second (-12)
First '\186'
Second (-11)
First '\v'
Where sum types get even more interesting is that you can
choose a diﬀerent weighting of probabilities than an equal dis-
tribution. Consider this snippet of the Maybe Arbitrary instance
from the QuickCheck library:
instance Arbitrary a=>
Arbitrary (Maybea)where
arbitrary =
frequency [( 1, return Nothing),
(3, liftM Justarbitrary)]
It’s making an arbitrary Justvalue three times more likely
than aNothing value because the former is more likely to be
interesting and useful, but you still want to try shaking things
out with a Nothing from time to time.

CHAPTER 14. TESTING 872
Accordingly, we can assign a 10 times higher probability to
ourFirstdata constructor in a diﬀerent GenforSum:
sumGenFirstPls ::(Arbitrary a,
Arbitrary b)=>
Gen(Suma b)
sumGenFirstPls = do
a<-arbitrary
b<-arbitrary
frequency [( 10, return $Firsta),
(1, return $Secondb)]
sumGenCharIntFirst ::Gen(SumCharInt)
sumGenCharIntFirst =sumGenFirstPls
With that modified version, you’ll find Second values are
much less common:
First '\208'
First '\242'
First '\159'
First 'v'
First '\159'
First '\232'
First '3'
First 'l'

CHAPTER 14. TESTING 873
Second (-16)
First 'x'
First 'Y'
One of the key insights here is that the Arbitrary instance
for a datatype doesn’t have to be the only way to generate or
provide random values of your datatype for QuickCheck tests.
You can oﬀer alternative Gens for your type with interesting or
useful behavior as well.
CoArbitrary
CoArbitrary is a counterpart to Arbitrary that enables the gener-
ation of functions fitting a particular type. Rather than talking
about random values you can get via Gen, it lets you provide
functions with a value of type 𝑎as an argument in order to
varyaGen:
arbitrary ::Arbitrary a=>
Gena
coarbitrary ::CoArbitrary a=>
a->Genb->Genb
-- [1] [ 2 ] [ 3 ]
Here[1]is used to return a modification or variant of [2]
which is the result [3]at the end.

CHAPTER 14. TESTING 874
It turns out, as long as your datatype has a Generic instance
derived, you can get these instances for free. The following
should work fine:
{-# LANGUAGE DeriveGeneric #-}
moduleCoArbitrary where
importGHC.Generics
importTest.QuickCheck
dataBool'=
True'
|False'
deriving (Generic)
instance CoArbitrary Bool'
This’ll then let you do things like the following:

CHAPTER 14. TESTING 875
importTest.QuickCheck
-- plus the above
trueGen ::GenInt
trueGen =coarbitrary True'arbitrary
falseGen ::GenInt
falseGen =coarbitrary False'arbitrary
Essentially this lets you randomly generate a function. It
might be a little hard to see why you’d care for now, but if
you ever find yourself wanting to randomly generate anything
with the (->)type inside it somewhere, it becomes salient in a
hurry.
14.7 Chapter Exercises
Now it’s time to write some tests of your own. You could write
tests for most of the exercises you’ve done in the book, but
whether you’d want to use hspecorQuickCheck depends on what
you’re trying to test. We’ve tried to simplify things a bit by
telling you which to use for these exercises, but, as always, we
encourage you to experiment on your own.

CHAPTER 14. TESTING 876
Validating numbers into words
Remember the “numbers into words” exercise in Recursion?
You’ll be writing tests to validate the functions you wrote.

CHAPTER 14. TESTING 877
moduleWordNumberTest where
importTest.Hspec
importWordNumber
(digitToWord ,digits,wordNumber )
main::IO()
main=hspec$ do
describe "digitToWord" $ do
it"returns zero for 0" $ do
digitToWord 0`shouldBe` "zero"
it"returns one for 1" $ do
print"???"
describe "digits" $ do
it"returns [1] for 1" $ do
digits1`shouldBe` [ 1]
it"returns [1, 0, 0] for 100" $ do
print"???"
describe "wordNumber" $ do
it"one-zero-zero given 100" $ do
wordNumber 100
`shouldBe` "one-zero-zero"
it"nine-zero-zero-one for 9001" $ do
print"???"

CHAPTER 14. TESTING 878
Fill in the test cases that print question marks. If you think
of additional tests you could perform, add them.
Using QuickCheck
Test some simple arithmetic properties using QuickCheck .
1.-- for a function
halfx=x/2
-- this property should hold
halfIdentity =(*2).half
2.importData.List (sort)
-- for any list you apply sort to
-- this property should hold
listOrdered ::(Orda)=>[a]->Bool
listOrdered xs=
snd$foldr go ( Nothing,True) xs
wherego_status@(_,False)=status
go y (Nothing, t)=(Justy, t)
go y (Justx, t)=(Justy, x>=y)
3.Now we’ll test the associative and commutative properties
of addition:

CHAPTER 14. TESTING 879
plusAssociative x y z=
x+(y+z)==(x+y)+z
plusCommutative x y=
x+y==y+x
Keep in mind these properties won’t hold for types based
on IEEE-754 floating point numbers, such as Floator
Double .
4.Now do the same for multiplication.
5.We mentioned in one of the first chapters that there are
some laws involving the relationship of quotandremand
divandmod. Write QuickCheck tests to prove them.
-- quot rem
(quot x y) *y+(rem x y) ==x
(div x y) *y+(mod x y) ==x
6.Is (^) associative? Is it commutative? Use QuickCheck to see
if the computer can contradict such an assertion.
7.Test that reversing a list twice is the same as the identity
of the list:
reverse .reverse ==id
8.Write a property for the definition of ($).

CHAPTER 14. TESTING 880
f$a=f a
f.g=\x->f (g x)
9.See if these two functions are equal:
foldr(:)==(++)
foldr(++)[]==concat
10.Hm. Is that so?
fn xs=length (take n xs) ==n
11.Finally, this is a fun one. You may remember we had you
compose readandshowone time to complete a “round
trip.” Well, now you can test that it works:
fx=(read (show x)) ==x
Failure
Find out why this property fails.

CHAPTER 14. TESTING 881
-- for a function
squarex=x*x
-- why does this property not hold?
-- Examine the type of sqrt.
squareIdentity =square.sqrt
Hint: Read about floating point arithmetic and precision if
you’re unfamiliar with it.
Idempotence
Idempotence refers to a property of some functions in which
the result value does not change beyond the initial application.
If you apply the function once, it returns a result, and applying
the same function to that value won’t ever change it. You might
think of a list that you sort: once you sort it, the sorted list will
remain the same after applying the same sorting function to
it. It’s already sorted, so new applications of the sort function
won’t change it.
UseQuickCheck and the following helper functions to demon-
strate idempotence for the following:
twicef=f.f
fourTimes =twice.twice

CHAPTER 14. TESTING 882
1.fx=
(capitalizeWord x
==twice capitalizeWord x)
&&
(capitalizeWord x
==fourTimes capitalizeWord x)
2.f'x=
(sort x
==twice sort x)
&&
(sort x
==fourTimes sort x)
Make a Gen random generator for the datatype
We demonstrated in the chapter how to make Gengenerators
for diﬀerent datatypes. We are so certain you enjoyed that, we
are going to ask you to do it for some new datatypes:
1.Equal probabilities for each.
dataFool=
Fulse
|Frue
deriving (Eq,Show)

CHAPTER 14. TESTING 883
2.2/3s chance of Fulse, 1/3 chance of Frue.
dataFool=
Fulse
|Frue
deriving (Eq,Show)
Hangman testing
Next, you should go back to the hangman project from the
previous chapter and write tests. The kinds of tests you can
write at this point will be limited due to the interactive nature
of the game. However, you can test the functions. Focus your
attention on testing the following:
fillInCharacter ::Puzzle->Char->Puzzle
fillInCharacter (Puzzleword
filledInSoFar s) c =
Puzzleword newFilledInSoFar (c :s)
wherezipper guessed wordChar guessChar =
ifwordChar ==guessed
thenJustwordChar
elseguessChar
newFilledInSoFar =
letzd=(zipper c)
inzipWith zd word filledInSoFar

CHAPTER 14. TESTING 884
and:
handleGuess ::Puzzle->Char->IOPuzzle
handleGuess puzzle guess = do
putStrLn $"Your guess was: " ++[guess]
case(charInWord puzzle guess
, alreadyGuessed puzzle guess) of
(_,True)-> do
putStrLn "You already guessed that \
\character, pick \
\something else!"
return puzzle
(True,_)-> do
putStrLn "This character was in the \
\word, filling in the \
\word accordingly"
return (fillInCharacter puzzle guess)
(False,_)-> do
putStrLn "This character wasn't in \
\the word, try again."
return (fillInCharacter puzzle guess)
Refresh your memory on what those are supposed to do
and then test to make sure they do.

CHAPTER 14. TESTING 885
Validating ciphers
As a final exercise, create QuickCheck properties that verify your
Caesar and Vigenère ciphers return the same data after encod-
ing and decoding a string.
14.8 Definitions
1.Unit testing is a method in which you test the smallest
parts of an application possible. These units are individu-
ally and independently scrutinized for desired behaviors.
Unit testing is better automated but it can also be done
manually via a human entering inputs and verifying out-
puts.
2.Property testing is a testing method where a subset of a
large input space is validated, usually against a property
or law some code should abide by. In Haskell, this is
usually done with QuickCheck which facilitates the random
generation of input and definition of properties to be veri-
fied. Common properties that are checked using property
testing are things like identity, associativity, isomorphism,
and idempotence.
3.When we say an operation or function is idempotent or
satisfies idempotence , we mean that applying it multiple
times doesn’t produce a diﬀerent result from the first time.

CHAPTER 14. TESTING 886
One example is multiplying by one or zero. You always
get the same result as the first time you multipled by one
or zero.
14.9 Follow-up resources
1.Pedro Vasconcelos; An introduction to QuickCheck
testing;
https://www.fpcomplete.com/user/pbv/
an-introduction-to-quickcheck-testing
2.Koen Claessen and John Hughes; (2000)
QuickCheck: A Lightweight Tool for Random Testing of
Haskell Programs
3.Pedro Vasconcelos; Verifying a Simple Compiler Using
Property-based Random Testing;
http://www.dcc.fc.up.pt/dcc/Pubs/TReports/TR13/
dcc-2013-06.pdf

Chapter 15
Monoid, Semigroup
Simplicity does not
precede complexity, but
follows it.
Alan Perlis
887

CHAPTER 15. MONOID, SEMIGROUP 888
15.1 Monoids and semigroups
One of the finer points of the Haskell community has been
its propensity for recognizing abstract patterns in code which
have well-defined, lawful representations in mathematics. A
word frequently used to describe these abstractions is algebra ,
by which we mean one or more operations and the setthey
operate over. Over the next few chapters, we’re going to be
looking at some of these. Some you may have heard of, such
as functor and monad. Some, such as monoid and the humble
semigroup, may seem new to you. One of the things that
Haskell is really good at is these algebras, and it’s important to
master them before we can do some of the exciting stuﬀ that’s
coming.
This chapter will include:
•Algebras!
•Laws!
•Monoids!
•Semigroups!

CHAPTER 15. MONOID, SEMIGROUP 889
15.2 What we talk about when we talk
about algebras
For some of us, talking about “an algebra” may sound some-
what foreign. So let’s take a second and talk about what we’re
talking about when we use this phrase, at least when we’re
talking about Haskell.
Algebra generally refers to one of the most important fields
of mathematics. In this usage, it means the study of mathe-
matical symbols and the rules governing their manipulation.
It is diﬀerentiated from arithmetic by its use of abstractions
such as variables. By the use of variables, we’re saying we don’t
care much what value will be put into that slot. We care about
the rules of how to manipulate this thing without reference to
its particular value.
And so, as we said above, an algebra refers to some opera-
tions and the set they operate over. Here again, we care less
about the particulars of the values or data we’re working with
and more about the general rules of their use.
In Haskell, these algebras can be implemented with type-
classes; the typeclasses define the set of operations. When we
talk about operations over a set, the set is the typethe opera-
tions are for. The instance defines how each operation will
perform for a given type or set. One of those algebras we use
ismonoid . If you’re a working programmer, you’ve probably

CHAPTER 15. MONOID, SEMIGROUP 890
had monoidal patterns in your code already, perhaps without
realizing it.
15.3 Monoid
A monoid is a binary associative operation with an identity.
This definition tells you a lot — if you’re accustomed to picking
apart mathematical definitions. Let us dissect this frog!
A monoid is a binary associative operation with an identity.
[1] [2] [3] [4] [5]
1.The thing we’re talking about — monoids. That’ll end up
being the name of our typeclass.
2.Binary, i.e., two. So, there will be two of something.
3.Associative — this is a property or law that must be satis-
fied. You’ve seen associativity with addition and multipli-
cation. We’ll explain it more in a moment.
4.Operation — so called because in mathematics, it’s usually
used as an infix operator. You can read this interchange-
ably as “function.” Note that given the mention of “binary”
earlier, we know that this is a function of two arguments.
5.Identity is one of those words in mathematics that pops
up a lot. In this context, we can take this to mean there’ll

CHAPTER 15. MONOID, SEMIGROUP 891
be some value which, when combined with any other
value, will always return that other value. This can be
seen most immediately with examples.
For lists, we have a binary operator, (++), that joins two
lists together. We can also use a function, mappend , from
theMonoid typeclass to do the same thing:
Prelude> mappend [1, 2, 3] [4, 5, 6]
[1,2,3,4,5,6]
For lists, the empty list, [], is the identity value:
mappend [1..5][]=[1..5]
mappend [][1..5]=[1..5]
We can rewrite this as a more general rule, using mempty
from the Monoid typeclass as a generic identity value (more
on this later):
mappend x mempty =x
mappend mempty x =x
In plain English, a monoid is a function that takes two argu-
ments and follows two laws: associativity and identity. Asso-
ciativity means the arguments can be regrouped (or reparen-
thesized, or reassociated) in diﬀerent orders and give the same

CHAPTER 15. MONOID, SEMIGROUP 892
result, as in addition. Identity means there exists some value
such that when we pass it as input to our function, the opera-
tion is rendered moot and the other value is returned, such as
when we add zero or multiply by one. Monoid is the typeclass
that generalizes these laws across types.
15.4 How Monoid is defined in Haskell
Typeclasses give us a way to recognize, organize, and use com-
mon functionalities and patterns across types that diﬀer in
some ways but also have things in common. So, we recognize
that, although there are many types of numbers, all of them
can be arguments in addition, and then we make an addition
function as part of the Numclass that all numbers implement.
TheMonoid typeclass recognizes and orders a diﬀerent pat-
tern than Numbut the goal is similar. The pattern of Monoid is
outlined above: types that have binary functions that let you
join things together in accordance with the laws of associa-
tivity, along with an identity value that will return the other
argument unmodified. This is the pattern of summation, mul-
tiplication, and list concatenation, among other things. The
typeclass abstracts and generalizes the pattern so that you write
code in terms of anytype that can be monoidally combined.
The typeclass Monoid is defined:

CHAPTER 15. MONOID, SEMIGROUP 893
classMonoidmwhere
mempty ::m
mappend ::m->m->m
mconcat ::[m]->m
mconcat =foldr mappend mempty
mappend is howanytwo values that inhabit your type can be
joined together. mempty is the identity value for that mappend
operation. There are some laws that all Monoid instances must
abide, and we’ll get to those soon. Next, let’s look at some
examples of monoids in action!
15.5 Examples of using Monoid
The nice thing about monoids is that they are familiar; they’re
all over the place. The best way to understand them initially
is to look at examples of some common monoidal operations
and remember that this typeclass abstracts the pattern out,
giving you the ability to use the operations over a larger range
of types.
List
One common type with an instance of Monoid isList. Check
out how monoidal operations work with lists:
Prelude> mappend [1, 2, 3] [4, 5, 6]

CHAPTER 15. MONOID, SEMIGROUP 894
[1,2,3,4,5,6]
Prelude> mconcat [[1..3], [4..6]]
[1,2,3,4,5,6]
Prelude> mappend "Trout" " goes well with garlic"
"Trout goes well with garlic"
This should look familiar, because we’ve certainly seen this
before:
Prelude> (++) [1, 2, 3] [4, 5, 6]
[1,2,3,4,5,6]
Prelude> (++) "Trout" " goes well with garlic"
"Trout goes well with garlic"
Prelude> foldr (++) [] [[1..3], [4..6]]
[1,2,3,4,5,6]
Prelude> foldr mappend mempty [[1..3], [4..6]]
[1,2,3,4,5,6]
Our old friend (++)! And if we look at the definition of
Monoid for lists, we can see how this all lines up:
instance Monoid[a]where
mempty =[]
mappend =(++)
For other types, the instances would be diﬀerent, but the
ideas behind them remain the same.

CHAPTER 15. MONOID, SEMIGROUP 895
15.6 Why Integer doesn’t have a
Monoid
The type Integer does not have a Monoid instance. None of the
numeric types do. Yet it’s clear that numbers have monoidal
operations, so what’s up with that, Haskell?
While in mathematics the monoid of numbers is summa-
tion, there’s not a clear reason why it can’t be multiplication.
Both operations are monoidal (binary, associative, having an
identity value), but each type should only have one unique
instance for a given typeclass, not two (one instance for a sum,
one for a product).
This won’t work:
Prelude> let x = 1 :: Integer
Prelude> let y = 3 :: Integer
Prelude> mappend x y
<interactive>:6:1: error:
• No instance for (Monoid Integer)
arising from a use of ‘mappend’
• In the expression: mappend x y
In an equation for ‘it’:
it = mappend x y

CHAPTER 15. MONOID, SEMIGROUP 896
It isn’t clear if those should be added or multiplied as a
mappend operation. It says there’s no Monoid for those Integer s
for that reason. You get the idea.
To resolve the conflict, we have the SumandProduct newtypes
to wrap numeric values and signal which Monoid instance we
want. These newtypes are built into the Data.Monoid module.
While there are two possible instances of Monoid for numeric
values, we avoid using scoping tricks and abide by the rule that
typeclass instances are unique to the types they are for:
Prelude> mappend (Sum 1) (Sum 5)
Sum {getSum = 6}
Prelude> mappend (Product 5) (Product 5)
Product {getProduct = 25}
Prelude> mappend (Sum 4.5) (Sum 3.4)
Sum {getSum = 7.9}
Note that we could use it with values that aren’t integral.
We can use these Monoid newtypes for all the types that have
instances of Num.
Integersformamonoidundersummationandmultiplication . We
can similarly say that lists form a monoid under concatenation.
It’s worth pointing out here that numbers aren’t the only
sets that have more than one possible monoid. Lists have
more than one possible monoid, although for now we’re only
working with concatenation (we’ll look at the other list monoid

CHAPTER 15. MONOID, SEMIGROUP 897
in another chapter). Several other types do as well. We usually
enforce the unique instance rule by using newtype to separate
the diﬀerent monoidal behaviors.
Why newtype?
Use of a newtype can be hard to justify or explain to people that
don’t yet have good intuitions for how Haskell code gets com-
piled and the representations of data used by your computer
in the course of executing your programs. With that in mind,
we’ll do our best and oﬀer two explanations intended for two
diﬀerent audiences. We will return to the topic of newtype in
more detail later in the book.
First, there’s not much semantic diﬀerence (except for cir-
cumstances involving bottom , explained later) between the fol-
lowing datatypes:
dataServer=ServerString
newtype Server' =Server' String
The main diﬀerences are that using newtype constrains the
datatype to having a single unary data constructor and newtype
guarantees no additional runtime overhead in “wrapping” the
original type. That is, the runtime representation of newtype
and what it wraps are always identical — no additional “boxing
up” of the data as is necessary for typical products and sums.

CHAPTER 15. MONOID, SEMIGROUP 898
Forveteranprogrammerswhounderstandpointers newtype
is like a single-member C union that avoids creating an extra
pointer, but still gives you a new type constructor and data
constructor so you don’t mix up the many many many things
that share a single representation.
In summary, why you might use newtype
1.To signal intent: using newtype makes it clear that you only
intend for it to be a wrapper for the underlying type. The
newtype cannot eventually grow into a more complicated
sum or product type, while a normal datatype can.
2.To improve type safety: avoid mixing up many values of
the same representation, such as TextorInteger .
3.To add diﬀerent typeclass instances to a type that is other-
wise unchanged representationally, such as with Sumand
Product .
More on Sum and Product
There’s more than one valid Monoid instance one can write for
numbers, so we use newtype wrappers to distinguish which we
want. If you import Data.Monoid you’ll see the SumandProduct
newtypes:

CHAPTER 15. MONOID, SEMIGROUP 899
Prelude> import Data.Monoid
Prelude> :info Sum
newtype Sum a = Sum {getSum :: a}
...some instances elided...
instance Num a => Monoid (Sum a)
Prelude> :info Product
newtype Product a =
Product {getProduct :: a}
...some instances elided...
instance Num a => Monoid (Product a)
The instances say that we can use SumorProduct values as a
Monoid as long as they contain numeric values. We can prove
this is the case for ourselves. We’re going to be using the infix
operator for mappend in these examples. It has the same type
and does the same thing but saves some characters and will
make these examples a bit cleaner:
Prelude Data.Monoid> :t (<>)
(<>) :: Monoid m => m -> m -> m
Prelude> Sum "Frank" <> Sum "Herbert"
No instance for (Num [Char]) ...

CHAPTER 15. MONOID, SEMIGROUP 900
The example didn’t work because the 𝑎inSum awasString
which is not an instance of Num.
SumandProduct do what you’d expect with a bit of syntactic
surprise:
Prelude Data.Monoid> (Sum 8) <> (Sum 9)
Sum {getSum = 17}
Prelude Data.Monoid> mappend mempty Sum 9
Sum {getSum = 9}
Butmappend joins two things, so you can’t do this:
Prelude> mappend (Sum 8) (Sum 9) (Sum 10)
You’ll get a big error message including this line:
Possible cause: ‘Sum’ is applied to too many arguments
In the first argument of ‘mappend’, namely ‘(Sum 8)’
So, that’s easy enough to fix by nesting:
Prelude> mappend (Sum 1) (mappend (Sum 2) (Sum 3))
Sum {getSum = 6}
Or somewhat less tedious by infixing the mappend function:
Prelude> Sum 1 <> Sum 1 <> Sum 1
Sum {getSum = 3}

CHAPTER 15. MONOID, SEMIGROUP 901
Or you could also put your Sums in a list and use mconcat :
Prelude> mconcat [Sum 8, Sum 9, Sum 10]
Sum {getSum = 27}
Due to the special syntax of SumandProduct , we also have
built-in record field accessors we can use to unwrap the value:
Prelude> getSum $ mappend (Sum 1) (Sum 1)
2
Prelude> getProduct $ mappend (Product 5) (Product 5)
25
Prelude> getSum $ mconcat [(Sum 5), (Sum 6), (Sum 7)]
18
Product is similar to Sumbut for multiplication.
15.7 Why bother?
Because monoids are common and they’re a nice abstraction
to work with when you have multiple monoidal things run-
ning around in a project. Knowing what a monoid is can help
you to recognize when you’ve encountered the pattern. Fur-
ther, having principled laws for it means you know you can
combine monoidal operations safely. When we say something

CHAPTER 15. MONOID, SEMIGROUP 902
is a monoid or can be described as monoidal , we mean you can
define at least one law-abiding Monoid instance for it.
A common use of monoids is to structure and describe com-
mon modes of processing data. Sometimes this is to describe
an API for incrementally processing a large dataset, sometimes
to describe guarantees needed to roll up aggregations (think
summation) in a parallel, concurrent, or distributed processing
framework.
One example of where things like the identity can be useful
is if you want to write a generic library for doing work in
parallel. You could choose to describe your work as being like
a tree, with each unit of work being a leaf. From there you
can partition the tree into as many chunks as are necessary to
saturate the number of processor cores or entire computers
you want to devote to the work. The problem is, if we have a
pair-wise operation and we need to combine an odd number
of leaves, how do we even out the count?
One straightforward way could be to simply provide mempty
(the identity value) to the odd leaves out so we get the same
result and pass it up to the next layer of aggregation!
A variant of monoid that provides more guarantees is the
Abelian or commutative monoid. Commutativity can be par-
ticularly helpful when doing concurrent or distributed pro-
cessing of data because it means the intermediate results being
computed in a diﬀerent order won’t change the eventual an-
swer.

CHAPTER 15. MONOID, SEMIGROUP 903
Monoids are even strongly associated with the concept of
folding or catamorphism — something we do all the time in
Haskell. You’ll see this more explicitly in the Foldable chapter,
but here’s a taste:
Prelude> foldr mappend mempty ([2, 4, 6] :: [Product Int])
Product {getProduct = 48}
Prelude> foldr mappend mempty ([2, 4, 6] :: [Sum Int])
Sum {getSum = 12}
Prelude> foldr mappend mempty ["blah", "woot"]
"blahwoot"
You’ll see monoidal structure come up when we explain
Applicative andMonadas well.
15.8 Laws
We’ll get to those laws in a moment. First, heed our little cri de
coeurabout why you should care about mathematical laws:
Laws circumscribe what constitutes a valid instance or con-
crete instance of the algebra or set of operations we’re working
with. We care about the laws a Monoid instance must adhere to
because we want our programs to be correct wherever possible.
Proofs are programs, and programs are proofs. We care about

CHAPTER 15. MONOID, SEMIGROUP 904
programs that compose well, that are easy to understand, and
which have predictable behavior. To that end, we should steal
prolifically from mathematics.
Algebras are defined by their laws and are useful principally
fortheir laws. Laws make up what algebras are.
Among other things, laws provide us guarantees that let
us build on solid foundations. Those guarantees give us pre-
dictable composition (or combination) of programs. Without
the ability to safely combine programs, everything must be
written from scratch, nothing could be reused. The physical
world has enjoyed the useful properties of stone stacked up
on top of stone since the Great Pyramid of Giza was built in
the pharaoh Sneferu’s reign in 2,600 BC. Similarly, if we want
to be able to stack up functions scalably, they need to obey
laws. Stones don’t evaporate into thin air or explode violently.
It’d be nice if our programs were similarly trustworthy.
There are more possible laws we can require for an algebra
than associativity or an identity, but these are simple examples
we are starting with for now, partly because Monoid is a good
place to start with algebras-as-typeclasses. We’ll see examples
of more later.
Monoid instances must abide by the following laws:

CHAPTER 15. MONOID, SEMIGROUP 905
-- left identity
mappend mempty x =x
-- right identity
mappend x mempty =x
-- associativity
mappend x (mappend y z) =
mappend (mappend x y) z
mconcat =foldr mappend mempty
Here is how the identity law looks in practice:
Prelude> import Data.Monoid
-- left identity
Prelude> mappend mempty (Sum 1)
Sum {getSum = 1}
-- right identity
Prelude> mappend (Sum 1) mempty
Sum {getSum = 1}
We can demonstrate associativity more easily if we first
introduce the infix operator for mappend ,<>. Note the parenthe-
sization on the two examples:

CHAPTER 15. MONOID, SEMIGROUP 906
Prelude> :t (<>)
(<>) :: Monoid m => m -> m -> m
-- associativity
Prelude> (Sum 1) <> (Sum 2 <> Sum 3)
Sum {getSum = 6}
Prelude> (Sum 1 <> Sum 2) <> (Sum 3)
Sum {getSum = 6}
Andmconcat should have the same result as foldr mappend
mempty :
Prelude> mconcat [Sum 1, Sum 2, Sum 3]
Sum {getSum = 6}
Prelude> foldr mappend mempty [Sum 1, Sum 2, Sum 3]
Sum {getSum = 6}
Now let’s see all of that again, but using the Monoid of lists:
-- mempty is []
-- mappend is (++)
-- left identity
Prelude> mappend mempty [1, 2, 3]
[1,2,3]

CHAPTER 15. MONOID, SEMIGROUP 907
-- right identity
Prelude> mappend [1, 2, 3] mempty
[1,2,3]
-- associativity
Prelude> [1] <> ([2] <> [3])
[1,2,3]
Prelude> ([1] <> [2]) <> [3]
[1,2,3]
-- mconcat ~ foldr mappend mempty
Prelude> mconcat [[1], [2], [3]]
[1,2,3]
Prelude> foldr mappend mempty [[1], [2], [3]]
[1,2,3]
Prelude> concat [[1], [2], [3]]
[1,2,3]
The important part here is that you have these guarantees
even when you don’t know whatMonoid you’ll be working with.

CHAPTER 15. MONOID, SEMIGROUP 908
15.9 Diﬀerent instance, same
representation
Monoid is somewhat diﬀerent from other typeclasses in Haskell,
in that many datatypes have more than one valid monoid. We
saw that for numbers, both addition and multiplication are sen-
sible monoids with diﬀerent behaviors. When we have more
than one potential implementation for Monoid for a datatype,
it’s most convenient to use newtypes to tell them apart, as we
did with SumandProduct .
Addition is a classic appending operation, as is list concate-
nation. Referring to multiplication as an appending operation
may also seem intuitive enough, as it still follows the basic
pattern of combining two values of one type into one value.
But for other datatypes the meaning of append is less clear.
In these cases, the monoidal operation is less about combining
the values and more about finding a summary value for the set.
We mentioned above that monoids are important to folding
and catamorphisms more generally. Mappending is perhaps
bestthoughtofnotasawayofcombiningvaluesinthewaythat
addition or list concatenation does, but as a way to condense
any set of values to a summary value. We’ll start by looking at
theMonoid instances for Boolto see what we mean.
Boolean values have two possible monoids — a monoid of
conjunction and one of disjunction. As we do with numbers,

CHAPTER 15. MONOID, SEMIGROUP 909
we use newtypes to distinguish the two instances. AllandAny
are the newtypes for Bool’s monoids:
Prelude> import Data.Monoid
Prelude> All True <> All True
All {getAll = True}
Prelude> All True <> All False
All {getAll = False}
Prelude> Any True <> Any False
Any {getAny = True}
Prelude> Any False <> Any False
Any {getAny = False}
Allrepresents boolean conjunction: it returns a Trueif and
only if all values it is “appending” are True.Anyis the monoid
of boolean disjunction: it returns a Trueif any value is True.
There is some sense in which it might feel strange to think of
this as a combining or mappending operation, unless we recall
that mappending is less about combining and more about
condensing or reducing.
TheMaybetype has more than two possible Monoids. We’ll
look at each in turn, but the two that have an obvious relation-
ship are FirstandLast. They are like boolean disjunction, but
with explicit preference for the leftmost or rightmost success

CHAPTER 15. MONOID, SEMIGROUP 910
in a series of Maybevalues. We have to choose because with
Bool, all you know is TrueorFalse— it doesn’t matter where
yourTrueorFalsevalues occurred. With Maybe, however, you
need to make a decision as to which Justvalue you’ll return
if there are multiple successes. FirstandLastencode these
diﬀerent possibilities.
Firstreturns the first or leftmost non- Nothing value:
Prelude> First (Just 1) `mappend` First (Just 2)
First {getFirst = Just 1}
Lastreturns the last or rightmost non- Nothing value:
Prelude> Last (Just 1) `mappend` Last (Just 2)
Last {getLast = Just 2}
Both will succeed in returning something in spite of Nothing
values as long as there’s at least one Just:
Prelude> Last Nothing `mappend` Last (Just 2)
Last {getLast = Just 2}
Prelude> First Nothing `mappend` First (Just 2)
First {getFirst = Just 2}
Neither can, for obvious reasons, return anything if all val-
ues are Nothing :

CHAPTER 15. MONOID, SEMIGROUP 911
Prelude> First Nothing `mappend` First Nothing
First {getFirst = Nothing}
Prelude> Last Nothing `mappend` Last Nothing
Last {getLast = Nothing}
To maintain the unique pairing of type and typeclass in-
stance, newtypes are used for all of those, the same as we saw
withSumandProduct .
Let’s look next at the third variety of Maybe Monoid .
15.10 Reusing algebras by asking for
algebras
We alluded to there being more possible Monoid s forMaybethan
justFirstandLast. Let’s write that other Monoid instance. We
will now be concerned not with choosing one value out of a
set of values but of combining the 𝑎values contained within
theMaybe a type.
First, try to notice a pattern:

CHAPTER 15. MONOID, SEMIGROUP 912
instance Monoidb=>Monoid(a->b)
instance (Monoida,Monoidb)
=>Monoid(a, b)
instance (Monoida,Monoidb,Monoidc)
=>Monoid(a, b, c)
What these Monoids have in common is that they are giv-
ing you a new Monoid for a larger type by reusing the Monoid
instances of types that represent components of the larger
type.
This obligation to ask for a Monoid for an encapsulated type
(such as the 𝑎inMaybe a ) exists even when not all possible val-
ues of the larger type contain the value of the type argument.
For example, Nothing does not contain the 𝑎we’re trying to
get aMonoid for, but Just a does, so not all possible Maybevalues
contain the 𝑎type argument. For a Maybe Monoid that will have
amappend operation for the 𝑎values, we need a Monoid for what-
ever type 𝑎is.Monoids likeFirstandLastwrap the Maybe a but
do not require a Monoid for the𝑎value itself because they don’t
mappend the𝑎values or provide a mempty of them.
If you do have a datatype that has a type argument that
does not appear anywhere in the terms (a phantom type), the
typechecker does not demand that you have a Monoid instance
for that argument. For example:

CHAPTER 15. MONOID, SEMIGROUP 913
dataBoolya=
False'
|True'
deriving (Eq,Show)
-- conjunction
instance Monoid(Boolya)where
mappend False'_ =False'
mappend _False'=False'
mappend True'True'=True'
We didn’t need a Monoid constraint for 𝑎because we’re never
mappending 𝑎values (we can’t; none exist) and we’re never
asking for a mempty of type 𝑎. This is the fundamental reason
we don’t need the constraint, but it can happen that we don’t
do this even when the type doesoccur in the datatype.
Exercise: Optional Monoid
Writethe Monoid instanceforour Maybetyperenamedto Optional .

CHAPTER 15. MONOID, SEMIGROUP 914
dataOptional a=
Nada
|Onlya
deriving (Eq,Show)
instance Monoida
=>Monoid(Optional a)where
mempty=undefined
mappend =undefined
Expected output:
Prelude> Only (Sum 1) `mappend` Only (Sum 1)
Only (Sum {getSum = 2})
Prelude> Only (Product 4) `mappend` Only (Product 2)
Only (Product {getProduct = 8})
Prelude> Only (Sum 1) `mappend` Nada
Only (Sum {getSum = 1})
Prelude> Only [1] `mappend` Nada
Only [1]
Prelude> Nada `mappend` Only (Sum 1)
Only (Sum {getSum = 1})

CHAPTER 15. MONOID, SEMIGROUP 915
Associativity
This will be mostly review, but we want to be specific about
associativity. Associativitysaysthatyoucanassociate, orgroup,
the arguments of your operation diﬀerently and the result will
be the same.
Let’s review examples of some operations that can be reas-
sociated :
Prelude> (1 + 9001) + 9001
18003
Prelude> 1 + (9001 + 9001)
18003
Prelude> (7 * 8) * 3
168
Prelude> 7 * (8 * 3)
168
And some that cannot have the parentheses reassociated
without changing the result:
Prelude> (1 - 10) - 100
-109
Prelude> 1 - (10 - 100)
91
This isnotas strong a property as an operation that com-
mutes or is commutative . Commutative means you can reorder

CHAPTER 15. MONOID, SEMIGROUP 916
the arguments and still get the same result. Addition and mul-
tiplication are commutative, but (++)for the list type is only
associative.
Let’s demonstrate this by writing a mildly evil version of
addition that flips the order of its arguments:
Prelude> let evilPlus = flip (+)
Prelude> 76 + 67
143
Prelude> 76 `evilPlus` 67
143
We have some evidence, but not proof, that(+)commutes.
However, we can’t do the same with (++):
Prelude> let evilPlusPlus = flip (++)
Prelude> let oneList = [1..3]
Prelude> let otherList = [4..6]
Prelude> oneList ++ otherList
[1,2,3,4,5,6]
Prelude> oneList `evilPlusPlus` otherList
[4,5,6,1,2,3]
In this case, this serves as a proof by counterexample that
(++)doesnotcommute. It doesn’t matter if it commutes for all

CHAPTER 15. MONOID, SEMIGROUP 917
other inputs; that it doesn’t commute for one of them means
thelaw of commutativity does not hold.
Commutativity is a useful property and can be helpful in
circumstances when you might need to be able to reorder
evaluation of your data for efficiency purposes without need-
ing to worry about the result changing. Distributed systems
use commutative monoids in designing and thinking about
constraints, which are monoids that guarantee their operation
commutes.
But, for our purposes, Monoid abides by the law of associa-
tivity but not the law of commutativity, even though some
monoidal operations (addition and multiplication) are com-
mutative.
Identity
An identity is a value with a special relationship with an oper-
ation: it turns the operation into the identity function. There
are no identities without operations. The concept is defined in
terms of its relationship with a given operation. If you’ve done
grade school arithmetic, you’ve already seen some identities:
Prelude> 1 + 0
1
Prelude> 521 + 0
521
Prelude> 1 * 1

CHAPTER 15. MONOID, SEMIGROUP 918
1
Prelude> 521 * 1
521
Zero is the identity value for addition, while 1is the identity
value for multiplication. As we said, it doesn’t make sense to
talk about zero and one as identity values outside the context
of those operations. That is, zero is definitely not the identity
value for other operations. We can check this property with a
simple equality test as well:
Prelude> let myList = [1..424242]
-- 0 serves as identity for addition
Prelude> map (+0) myList == myList
True
-- but not for multiplication
Prelude> map (*0) myList == myList
False
-- 1 serves as identity for multiplication
Prelude> map (*1) myList == myList
True
-- but not for addition
Prelude> map (+1) myList == myList
False

CHAPTER 15. MONOID, SEMIGROUP 919
This is the other law for Monoid : the binary operation must
be associative andit must have a sensible identity value.
The problem of orphan instances
We’ve said both in this chapter and in the earlier chapter de-
voted to Typeclasses that typeclasses have unique pairings of
the class and the instance for a particular type.
We do sometimes end up with multiple instances for a
single type when orphan instances are written. But writing
orphan instances should be avoided at all costs . If you get an
orphan instance warning from GHC, fix it.
An orphan instance is when an instance is defined for a
datatype and typeclass, but not in the same module as either
the declaration of the typeclass or the datatype. If you don’t
own the typeclass or the datatype, newtype it!
If you want an orphan instance so that you can have multi-
ple instances for the same type, you still want to use newtype .
We saw this earlier with SumandProduct which let us have two
diﬀerent Monoid instances for numbers without resorting to
orphans or messing up typeclass instance uniqueness.
Let’s see an example of an orphan instance and how to fix it.
First, make a project directory and change into that directory:
$ mkdir orphan-instance && cd orphan-instance

CHAPTER 15. MONOID, SEMIGROUP 920
Then we’re going to make a couple of files, one module in
each:
moduleListywhere
newtype Listya=
Listy[a]
deriving (Eq,Show)
moduleListyInstances where
importData.Monoid
importListy
instance Monoid(Listya)where
mempty=Listy[]
mappend ( Listyl) (Listyl')=
Listy$mappend l l'
So our directory will look like:
$ tree
.
├── Listy.hs
└── ListyInstances.hs

CHAPTER 15. MONOID, SEMIGROUP 921
Then to build ListyInstances such that it can see Listy, we
must use the -Iflag to include the current directory and make
modules within it discoverable. The .after the Iis how we
say “this directory” in Unix-alikes. If you succeed, you should
see something like the following:
$ ghc -I. --make ListyInstances.hs
[2 of 2] Compiling ListyInstances
Note that the only output will be an object file, the result of
compiling a module that can be reused as a library by Haskell
code, because we didn’t define a mainsuitable for producing an
executable. We’re only using this approach to build this so that
we can avoid the hassle of initializing (via stack new or similar)
a project. For anything more complicated or long-lived than
this, use a dependency and build management tool like Cabal
(if you’re using Stack, you’re also using Cabal).
Now to provide one example of why orphan instances are
problematic. Ifwecopyour Monoid instancefrom ListyInstances
intoListy, then rebuild ListyInstances , we’ll get the following
error.
$ ghc -I. --make ListyInstances.hs
[1 of 2] Compiling Listy
[2 of 2] Compiling ListyInstances
Listy.hs:7:10:

CHAPTER 15. MONOID, SEMIGROUP 922
Duplicate instance declarations:
instance Monoid (Listy a)
-- Defined at Listy.hs:7:10
instance Monoid (Listy a)
-- Defined at ListyInstances.hs:5:10
These conflicting instance declarations could happen to
anybody who uses the previous version of our code. And
that’s a problem.
Orphan instances are stilla problem even if duplicate in-
stances aren’t both imported into a module because it means
your typeclass methods will start behaving diﬀerently depend-
ing on what modules are imported, which breaks the funda-
mental assumptions and niceties of typeclasses.
There are a few solutions for addressing orphan instances:
1.You defined the type but not the typeclass? Put the in-
stance in the same module as the type so that the type
cannot be imported without its instances.
2.You defined the typeclass but not the type? Put the in-
stance in the same module as the typeclass definition
so that the typeclass cannot be imported without its in-
stances.
3.Neither the type nor the typeclass are yours? Define your
own newtype wrapping the original type and now you’ve

CHAPTER 15. MONOID, SEMIGROUP 923
got a type that “belongs” to you for which you can rightly
define typeclass instances. There are means of making
this less annoying which we’ll discuss later.
These restrictions must be maintained in order for us to
reap the full benefit of typeclasses along with the reasoning
properties that are associated with them. A type musthave
a unique (singular) implementation of a typeclass in scope,
and avoiding orphan instances is how we prevent conflict-
ing instances. Be aware, however, that avoidance of orphan
instances is more strictly adhered to among library authors
rather than application developers, although it’s no less im-
portant in applications.
15.11 Madness
You may have seen mad libs before. The idea is to take a tem-
plate of phrases, fill them in with blindly selected categories
of words, and see if saying the final version is amusing.
Using a lightly edited example from the Wikipedia article
on Mad Libs:
"___________! he said ______ as he
exclamation adverb
jumped into his car ____ and drove
noun

CHAPTER 15. MONOID, SEMIGROUP 924
off with his _________ wife."
adjective
We can make this into a function, like the following:
importData.Monoid
typeVerb=String
typeAdjective =String
typeAdverb=String
typeNoun=String
typeExclamation =String
madlibbin' ::Exclamation
->Adverb
->Noun
->Adjective
->String
madlibbin' e adv noun adj =
e<>"! he said " <>
adv<>" as he jumped into his car " <>
noun<>" and drove off with his " <>
adj<>" wife."
Now you’re going to refactor this code a bit! Rewrite it using
mconcat .

CHAPTER 15. MONOID, SEMIGROUP 925
madlibbinBetter' ::Exclamation
->Adverb
->Noun
->Adjective
->String
madlibbinBetter' e adv noun adj =undefined
15.12 Better living through QuickCheck
Proving laws can be tedious, especially if the code we’re check-
ing is in the middle of changing frequently. Accordingly, hav-
ing a cheap way to get a sense of whether or not the laws are
likelyto be obeyed by an instance is pretty useful. QuickCheck
happens to be an excellent way to accomplish this.
Validating associativity with QuickCheck
You can check the associativity of some simple arithemetic
expressions by asserting equality between two versions with
diﬀerent parenthesization and checking them in the REPL:

CHAPTER 15. MONOID, SEMIGROUP 926
-- we're saying these are the same because
-- (+) and (*) are associative
1+(2+3)==(1+2)+3
4*(5*6)==(4*5)*6
This doesn’t tell us that associativity holds for anyinputs to
(+)and(*), though, and that is what we want to test. Our old
friend from the lambda calculus — abstraction! — suffices for
this:
\a b c->a+(b+c)==(a+b)+c
\a b c->a*(b*c)==(a*b)*c
But our arguments aren’t the only thing we can abstract.
What if we want to talk about the abstract property of associa-
tivity for some given function 𝑓?
\f a b c ->
f a (f b c) ==f (f a b) c
-- or infix
\(<>) a b c ->
a<>(b<>c)==(a<>b)<>c

CHAPTER 15. MONOID, SEMIGROUP 927
Surprise! You can bind infix names for function arguments.
asc::Eqa
=>(a->a->a)
->a->a->a
->Bool
asc(<>) a b c =
a<>(b<>c)==(a<>b)<>c
Now how do we turn this function into something we can
property test with QuickCheck ? The quickest and easiest way
would probably look something like the following:
importData.Monoid
importTest.QuickCheck
monoidAssoc ::(Eqm,Monoidm)
=>m->m->m->Bool
monoidAssoc a b c=
(a<>(b<>c))==((a<>b)<>c)
We have to declare the types for the function in order to
run the tests, so that QuickCheck knows what types of data to
generate.
We can now use this to check associativity of functions:

CHAPTER 15. MONOID, SEMIGROUP 928
-- for brevity
Prelude> type S = String
Prelude> type B = Bool
Prelude> quickCheck (monoidAssoc :: S -> S -> S -> B)
+++ OK, passed 100 tests.
ThequickCheck function uses the Arbitrary typeclass to pro-
vide the randomly generated inputs for testing the function.
Although it’s common to do so, we may not want to rely on an
Arbitrary instance existing for the type of our inputs, for one
of a few reasons. It may be that we need a generator for a type
that doesn’t belong to us, so we’d rather not make an orphan
instance. Or it could be a type that already has an Arbitrary
instance, but we want to run tests with a diﬀerent random
distribution of values, or to make sure we check certain special
edge cases in addition to the random values.
You want to be careful to assert types so that QuickCheck
knows which Arbitrary instance to get random values for test-
ing from. You can use verboseCheck to see what values were
tested. If you try running the check verbosely and without
asserting a type for the arguments:
Prelude> verboseCheck monoidAssoc
Passed:
()
()

CHAPTER 15. MONOID, SEMIGROUP 929
()
(repeated 100 times)
This is GHCi’s type-defaulting biting you, as we saw back in
the Testing chapter. GHCi has slightly more aggressive type-
defaulting which can be handy in an interactive session when
you want to fire oﬀ some code and have your REPL pick a
winner for the typeclasses it doesn’t know how to dispatch.
Compiled in a source file, GHC would’ve complained about
an ambiguous type.
Testing left and right identity
Following on from what we did with associativity, we can also
useQuickCheck to test left and right identity:
monoidLeftIdentity ::(Eqm,Monoidm)
=>m
->Bool
monoidLeftIdentity a=(mempty <>a)==a
monoidRightIdentity ::(Eqm,Monoidm)
=>m
->Bool
monoidRightIdentity a=(a<>mempty) ==a

CHAPTER 15. MONOID, SEMIGROUP 930
Then running these properties against a Monoid :
Prelude> quickCheck (monoidLeftIdentity :: String -> Bool)
+++ OK, passed 100 tests.
Prelude> quickCheck (monoidRightIdentity :: String -> Bool)
+++ OK, passed 100 tests.
Testing QuickCheck’s patience
Let us see an example of QuickCheck catching us out for having
an invalid Monoid . Here we’re going to demonstrate why a Bool
Monoid can’t have Falseas the identity, always returning the
valueFalse, and still be a valid Monoid :
-- associative, left identity, and right
-- identity properties have been elided.
-- Add them to your copy of this.
importControl.Monad
importData.Monoid
importTest.QuickCheck
dataBull=
Fools
|Twoo
deriving (Eq,Show)

CHAPTER 15. MONOID, SEMIGROUP 931
instance Arbitrary Bullwhere
arbitrary =
frequency [ ( 1, return Fools)
, (1, return Twoo) ]
instance MonoidBullwhere
mempty=Fools
mappend _ _ =Fools
typeBullMappend =
Bull->Bull->Bull->Bool
main::IO()
main= do
letma=monoidAssoc
mli=monoidLeftIdentity
mlr=monoidRightIdentity
quickCheck (ma ::BullMappend )
quickCheck (mli ::Bull->Bool)
quickCheck (mlr ::Bull->Bool)
If you load this up in GHCi and run main, you’ll get the
following output:
Prelude> main
+++ OK, passed 100 tests.

CHAPTER 15. MONOID, SEMIGROUP 932
*** Failed! Falsifiable (after 1 test):
Twoo
*** Failed! Falsifiable (after 1 test):
Twoo
So this not-actually-a- Monoid forBoolturns out to pass asso-
ciativity, but fail on the right and left identity checks. To see
why, let’s line up the laws against what our mempty andmappend
are:
-- how the instance is defined
mempty=Fools
mappend _ _ =Fools
-- identity laws
mappend mempty x =x
mappend x mempty =x
-- Does it obey the laws?
-- because of how mappend is defined
mappend mempty x =Fools
mappend x mempty =Fools
-- Fools is not x, so it
-- fails the identity laws.

CHAPTER 15. MONOID, SEMIGROUP 933
It’s fine if your identity value is Fools, but if your mappend
always returns the identity, then it’s not an identity. It’s not
behaving like a zero as you’re not even checking if either argu-
ment is Foolsbefore returning Fools. It’s a black hole that spits
out one value, which is senseless. For an example of what is
meant by zero, consider multiplication which has an identity
anda zero:
-- Thus why the mempty for Sum is 0
0+x==x
x+0==x
-- Thus why the mempty for Product is 1
1*x==x
x*1==x
-- Thus why the mempty for
-- Product is *not* 0
0*x==0
x*0==0
UsingQuickCheck can be a great way to cheaply and easily
sanity check the validity of your instances against their laws.
You’ll see more of this.

CHAPTER 15. MONOID, SEMIGROUP 934
Exercise: Maybe Another Monoid
Write a Monoid instance for a Maybetype which doesn’t require
aMonoid for the contents. Reuse the Monoid lawQuickCheck prop-
erties and use them to validate the instance.
Don’t forget to write an Arbitrary instance for First'. We
won’t always stub that out explicitly for you. We suggest
learning how to use the frequency function from QuickCheck
forFirst' ’s instance.
newtype First'a=
First'{ getFirst' ::Optional a }
deriving (Eq,Show)
instance Monoid(First'a)where
mempty=undefined
mappend =undefined
firstMappend ::First'a
->First'a
->First'a
firstMappend =mappend

CHAPTER 15. MONOID, SEMIGROUP 935
typeFirstMappend =
First'String
->First'String
->First'String
->Bool
typeFstId=
First'String->Bool
main::IO()
main= do
quickCheck (monoidAssoc ::FirstMappend )
quickCheck (monoidLeftIdentity ::FstId)
quickCheck (monoidRightIdentity ::FstId)
Our expected output demonstrates a diﬀerent Monoid for
Optional /Maybewhich is getting the first success and holding
onto it, where any exist. This could be seen, with a bit of
hand-waving, as being a disjunctive (“or”) Monoid instance.
Prelude> First' (Only 1) `mappend` First' Nada
First' {getFirst' = Only 1}
Prelude> First' Nada `mappend` First' Nada
First' {getFirst' = Nada}
Prelude> First' Nada `mappend` First' (Only 2)
First' {getFirst' = Only 2}

CHAPTER 15. MONOID, SEMIGROUP 936
Prelude> First' (Only 1) `mappend` First' (Only 2)
First' {getFirst' = Only 1}
15.13 Semigroup
Mathematicians play with algebras like that creepy kid you
knew in grade school who would pull legs oﬀ of insects. Some-
times, they glue legs onto insects too, but in the case where
we’re going from Monoid toSemigroup , we’re pulling a leg oﬀ.
In this case, the leg is our identity. To get from a monoid
to a semigroup, we simply no longer furnish nor require an
identity. The core operation remains binary and associative.
With this, our definition of Semigroup is:
classSemigroup awhere
(<>)::a->a->a
And we’re left with one law:
(a<>b)<>c=a<>(b<>c)
Semigroup still provides a binary associative operation, one
that typically joins two things together (as in concatenation or
summation), but doesn’t have an identity value. In that sense,
it’s a weaker algebra.

CHAPTER 15. MONOID, SEMIGROUP 937
Not yet part of base As of GHC 8, the Semigroup typeclass
is part of basebut not part of Prelude . You need to import
Data.Semigroup to use its operations. Keep in mind that it de-
fines its own more general version of (<>)which only requires
aSemigroup constraint rather than a Monoid constraint.
You can import the NonEmpty datatype we are about to discuss
into your REPL by importing Data.List.NonEmpty .
NonEmpty, a useful datatype
One useful datatype that can’t have a Monoid instance but does
have a Semigroup instance is the NonEmpty list type. It is a list
datatype that can never be an empty list:
dataNonEmpty a=a:|[a]
deriving (Eq,Ord,Show)
-- some instances from the
-- real module elided
Here:|is an infix data constructor that takes two (type)
arguments. It’s a product of aand[a]. It guarantees that we
always have at least one value of type 𝑎, which [a]does not
guarantee as any list might be empty.
Note that although :|is not alphanumeric, as most of the
other data constructors you’re used to seeing are, it is a name
for an infix data constructor. Data constructors with only

CHAPTER 15. MONOID, SEMIGROUP 938
nonalphanumeric symbols and that begin with a colon are
infix by default; those with alphanumeric names are prefix by
default:
-- Prefix, works.
dataP=
PrefixIntString
-- Infix, works.
dataQ=
Int:!!:String
Since that data constructor is symbolic rather than alphanu-
meric, it can’t be used as a prefix:
dataR=
:!!:IntString
Using it as a prefix will cause a syntax error:
parse error on input ‘:!!:’
Failed, modules loaded: none.
On the other hand, an alphanumeric data constructor can’t
be used as an infix:
dataS=
IntPrefixString

CHAPTER 15. MONOID, SEMIGROUP 939
It will cause another error:
Not in scope: type constructor or class ‘Prefix’
A data constructor of that name is in scope;
did you mean DataKinds?
Failed, modules loaded: none.
Let’s return to the main point, which is NonEmpty . Because
NonEmpty isa productof twoarguments, wecould’vealso written
it as:
newtype NonEmpty a=
NonEmpty (a, [a])
deriving (Eq,Ord,Show)
We can’t write a Monoid forNonEmpty because it has no identity
value by design! There is no empty list to serve as an identity
for any operation over a NonEmpty list, yet there is still a binary
associative operation: two NonEmpty lists can still be concate-
nated. A type with a canonical binary associative operation but
no identity value is a natural fit for Semigroup . Here is a brief
example of using NonEmpty from the semigroups library with the
semigroup mappend (as of GHC 8.0.1, Semigroup andNonEmpty are
both in basebut not in Prelude ):
-- you may need to install `semigroups`
Prelude> import Data.List.NonEmpty as N

CHAPTER 15. MONOID, SEMIGROUP 940
Prelude N> import Data.Semigroup as S
Prelude N S> 1 :| [2, 3]
1 :| [2,3]
Prelude N S> :t 1 :| [2, 3]
1 :| [2, 3] :: Num a => NonEmpty a
Prelude N S> :t (<>)
(<>) :: Semigroup a => a -> a -> a
Prelude N S> let xs = 1 :| [2, 3]
Prelude N S> let ys = 4 :| [5, 6]
Prelude N S> xs <> ys
1 :| [2,3,4,5,6]
Prelude N S> N.head xs
1
Prelude N S> N.length (xs <> ys)
6
Beyond this, you use NonEmpty as you would a list, but what
you’ve gained is being explicit that having zero values is not
valid for your use-case. The datatype helps you enforce this
constraint by not letting you construct a NonEmpty unless you
have at least one value.

CHAPTER 15. MONOID, SEMIGROUP 941
15.14 Strength can be weakness
When Haskellers talk about the strength of an algebra, they
usually mean the number of operations it provides which in
turn expands what you can do with any given instance of that
algebra without needing to know specifically what type you
are working with.
The reason we cannot and do not want to make all of our
algebras as big as possible is that there are datatypes which
are very useful representationally, but which do not have the
ability to satisfy everything in a larger algebra that could work
fine if you removed an operation or law. This becomes a seri-
ous problem if NonEmpty is the right datatype for something in
the domain you’re representing. If you’re an experienced pro-
grammer, think carefully. How many times have you meant
for a list to never be empty? To guarantee this and make the
types more informative, we use types like NonEmpty .
The problem is that NonEmpty has no identity value for the
combining operation ( mappend ) inMonoid. So, we keep the as-
sociativity but drop the identity value and its laws of left and
right identity. This is what introduces the need for and idea
ofSemigroup from a datatype.
The most obvious way to see that a monoid is stronger than
a semigroup is to observe that it has a strict superset of the op-
erations and laws that Semigroup provides. Anything which is a
monoid is by definition alsoa semigroup. It is to be hoped that

CHAPTER 15. MONOID, SEMIGROUP 942
Semigroup will be made a superclass of Monoid in an upcoming
version of GHC.
classSemigroup a=>Monoidawhere
...
Earlier we reasoned about the inverse relationship between
operations permitted over a type and the number of types that
can satisfy. We can see this relationship between the number
of operations and laws an algebra demands and the number
of datatypes that can provide a law abiding instance of that
algebra.
In the following example, 𝑎can be anything in the universe,
but there are no operations over it — we can only return the
same value.
id::a->a
•Number of types: Infinite — universally quantified so
it can be any type the expression applying the function
wants.
•Number of operations: one, if you can call it an operation,
referencing the value you were passed.
Withinc𝑎now has all the operations from Num, which lets
us do more. But that also means it’s now a finite set of types

CHAPTER 15. MONOID, SEMIGROUP 943
that can satisfy the Numconstraint rather than being strictly any
type in the universe:
inc::Numa=>a->a
•Number of types: anything that implements Num. Zero to
many.
•Number of operations: 7 methods in Num
In the next example we know it’s an Integer , which gives us
many more operations than just a Numinstance:
somethingInt ::Int->Int
•Number of types: one — Int.
•Number of operations: considerably more than 7. In ad-
dition to Num,Inthas instances of Bounded ,Enum,Eq,Integral ,
Ord,Read,Real, andShow. On top of that, you can write ar-
bitrary functions that pattern match on concrete types
and return arbitrary values in that same type as the re-
sult. Polymorphism isn’t only useful for reusing code;
it’s also useful for expressing intent through parametricity
so that people reading the code know what we meant to
accomplish.
WhenMonoid is too strong or more than we need, we can use
Semigroup . If you’re wondering what’s weaker than Semigroup ,

CHAPTER 15. MONOID, SEMIGROUP 944
the usual next step is removing the associativity requirement,
giving you a magma. It’s not likely to come up in day to day
Haskell, but you can sound cool at programming conferences
for knowing what’s weaker than a semigroup so pocket that
one for the pub.
15.15 Chapter exercises
Semigroup exercises
Given a datatype, implement the Semigroup instance. Add
Semigroup constraints to type variables where needed. Use the
Semigroup class from the semigroups library (or from baseif you
are on GHC 8) or write your own. When we use (<>), we mean
the infix mappend from the Semigroup typeclass.
Note We’re not always going to derive every instance you
may want or need in the datatypes we provide for exercises.
We expect you to know what you need and to take care of it
yourself by this point.
1.Validate allof your instances with QuickCheck. Since
Semigroup ’s only law is associativity, that’s the only prop-
erty you need to reuse. Keep in mind that you’ll poten-
tially need to import the modules for Monoid andSemigroup
and to avoid naming conflicts for the (<>)depending on
your version of GHC.

CHAPTER 15. MONOID, SEMIGROUP 945
dataTrivial =Trivial deriving (Eq,Show)
instance Semigroup Trivial where
_ <> _ = undefined
instance Arbitrary Trivial where
arbitrary =returnTrivial
semigroupAssoc ::(Eqm,Semigroup m)
=>m->m->m->Bool
semigroupAssoc a b c=
(a<>(b<>c))==((a<>b)<>c)
typeTrivAssoc =
Trivial ->Trivial ->Trivial ->Bool
main::IO()
main=
quickCheck (semigroupAssoc ::TrivAssoc )
2.newtype Identity a=Identity a
3.dataTwoa b=Twoa b
Hint: Ask for another Semigroup instance.

CHAPTER 15. MONOID, SEMIGROUP 946
4.dataThreea b c=Threea b c
5.dataFoura b c d =Foura b c d
6.newtype BoolConj =
BoolConj Bool
What it should do:
Prelude> (BoolConj True) <> (BoolConj True)
BoolConj True
Prelude> (BoolConj True) <> (BoolConj False)
BoolConj False
7.newtype BoolDisj =
BoolDisj Bool
What it should do:
Prelude> (BoolDisj True) <> (BoolDisj True)
BoolDisj True
Prelude> (BoolDisj True) <> (BoolDisj False)
BoolDisj True
8.dataOra b=
Fsta
|Sndb

CHAPTER 15. MONOID, SEMIGROUP 947
TheSemigroup forOrshould have the following behavior.
We can think of this as having a “sticky” Sndvalue where
it’ll hold onto the first Sndvalue when and if one is passed
as an argument. This is similar to the First' Monoid you
wrote earlier.
Prelude> Fst 1 <> Snd 2
Snd 2
Prelude> Fst 1 <> Fst 2
Fst 2
Prelude> Snd 1 <> Fst 2
Snd 1
Prelude> Snd 1 <> Snd 2
Snd 1
9.newtype Combine a b=
Combine { unCombine ::(a->b) }
What it should do:
Prelude> let f = Combine $ \n -> Sum (n + 1)
Prelude> let g = Combine $ \n -> Sum (n - 1)
Prelude> unCombine (f <> g) $ 0
Sum {getSum = 0}
Prelude> unCombine (f <> g) $ 1
Sum {getSum = 2}

CHAPTER 15. MONOID, SEMIGROUP 948
Prelude> unCombine (f <> f) $ 1
Sum {getSum = 4}
Prelude> unCombine (g <> f) $ 1
Sum {getSum = 2}
Hint: This function will eventually be applied to a single
value of type 𝑎. But you’ll have multiple functions that can
produce a value of type 𝑏. How do we combine multiple
values so we have a single 𝑏? This one will probably be
tricky! Remember that the type of the value inside of
Combine is that of a function . The type of functions should
already have an Arbitrary instance that you can reuse for
testing this instance.
10.newtype Compa=
Comp{ unComp ::(a->a) }
Hint: We can do something that seems a little more spe-
cific and natural to functions now that the input and out-
put types are the same.

CHAPTER 15. MONOID, SEMIGROUP 949
11.-- Look familiar?
dataValidation a b=
Failure a|Success b
deriving (Eq,Show)
instance Semigroup a=>
Semigroup (Validation a b)where
(<>)=undefined
Given this code:
main= do
letfailure ::String
->Validation StringInt
failure =Failure
success ::Int
->Validation StringInt
success =Success
print$success 1<>failure "blah"
print$failure "woot"<>failure "blah"
print$success 1<>success 2
print$failure "woot"<>success 2
You should get this output:

CHAPTER 15. MONOID, SEMIGROUP 950
Prelude> main
Success 1
Failure "wootblah"
Success 1
Success 2
Monoid exercises
Given a datatype, implement the Monoid instance. Add Monoid
constraints to type variables where needed. For the datatypes
you’ve already implemented Semigroup instances for, you need
to figure out what the identity value is.
1.Again, validate allof your instances with QuickCheck.
Example scaﬀold is provided for the Trivial type.

CHAPTER 15. MONOID, SEMIGROUP 951
dataTrivial =Trivial deriving (Eq,Show)
instance Semigroup Trivial where
(<>)=undefined
instance MonoidTrivial where
mempty=undefined
mappend =(<>)
typeTrivAssoc =
Trivial ->Trivial ->Trivial ->Bool
main::IO()
main= do
letsa=semigroupAssoc
mli=monoidLeftIdentity
mlr=monoidRightIdentity
quickCheck (sa ::TrivAssoc )
quickCheck (mli ::Trivial ->Bool)
quickCheck (mlr ::Trivial ->Bool)
2.newtype Identity a=
Identity aderiving Show
3.dataTwoa b=Twoa bderiving Show

CHAPTER 15. MONOID, SEMIGROUP 952
4.newtype BoolConj =
BoolConj Bool
What it should do:
Prelude> (BoolConj True) `mappend` mempty
BoolConj True
Prelude> mempty `mappend` (BoolConj False)
BoolConj False
5.newtype BoolDisj =
BoolDisj Bool
What it should do:
Prelude> (BoolDisj True) `mappend` mempty
BoolDisj True
Prelude> mempty `mappend` (BoolDisj False)
BoolDisj False
6.newtype Combine a b=
Combine { unCombine ::(a->b) }
What it should do:
Prelude> let f = Combine $ \n -> Sum (n + 1)
Prelude> unCombine (mappend f mempty) $ 1
Sum {getSum = 2}

CHAPTER 15. MONOID, SEMIGROUP 953
7.Hint: We can do something that seems a little more spe-
cific and natural to functions now that the input and out-
put types are the same.
newtype Compa=
Comp(a->a)
8.This next exercise will involve doing something that will
feel a bit unnatural still and you may find it difficult. If you
get it and you haven’t done much FP or Haskell before,
get yourself a nice beverage. We’re going to toss you
the instance declaration so you don’t churn on a missing
Monoid constraint you didn’t know you needed.
newtype Mems a=
Mem{
runMem::s->(a,s)
}
instance Monoida=>Monoid(Mems a)where
mempty=undefined
mappend =undefined
Given the following code:

CHAPTER 15. MONOID, SEMIGROUP 954
f'=Mem$\s->("hi", s+1)
main= do
letrmzero=runMem mempty 0
rmleft=runMem (f' <>mempty) 0
rmright =runMem (mempty <>f')0
print$rmleft
print$rmright
print$(rmzero ::(String,Int))
print$rmleft==runMem f' 0
print$rmright ==runMem f' 0
A correct Monoid forMemshould, given the above code, get
the following output:
Prelude> main
("hi",1)
("hi",1)
("",0)
True
True
Make certain your instance has output like the above, this
is sanity-checking the Monoid identity laws for you! It’s not
a proof and it’s not even as good as property testing, but
it’ll catch the most common mistakes people make.

CHAPTER 15. MONOID, SEMIGROUP 955
It’s not a trick and you don’t need a Monoid for𝑠. Yes, such
aMonoid can and does exist. Hint: chain the 𝑠values from
one function to the other. You’ll want to check the identity
laws as a common first attempt will break them.
15.16 Definitions
1.Amonoid is a set that is closed under an associative binary
operation and has an identity element. Closed is the posh
mathematical way of saying its type is:
mappend ::m->m->m
Such that your arguments and output will always inhabit
the same type (set).
2.Asemigroup is a set that is closed under an associative
binary operation — and nothing else.
3.Laws are rules about how an algebra or structure should
behave. These are needed in part to make abstraction over
the commonalities of diﬀerent instantiations of the same
sort of algebra possible and practical . This is critical to
having abstractions which aren’t unpleasantly surprising.
4.Analgebra is variously:

CHAPTER 15. MONOID, SEMIGROUP 956
a)School algebra, such as that taught in primary and
secondary school. This usually entails the balancing
of polynomial equations and learning how functions
and graphs work.
b)The study of number systems and operations within
them. This will typically entail a particular area such
as groups or rings. This is what mathematicians com-
monly mean by “algebra.” This is sometimes disam-
biguated by being referred to as abstract algebra.
c)A third and final way algebra is used is to refer to a
vector space over a field with a multiplication.
When Haskellers refer to algebras, they’re usually talking
about a somewhat informal notion of operations over
a type and its laws, such as with semigroups, monoids,
groups, semirings, and rings.
15.17 Follow-up resources
1.Algebraic structure; Simple English Wikipedia
2.Haskell Monoids and Their Uses; Dan Piponi

Chapter 16
Functor
Lifting is the ”cheat
mode” of type tetris.
Michael Neale
957

CHAPTER 16. FUNCTOR 958
16.1 Functor
In the last chapter on Monoid , we saw what it means to talk about
an algebra and turn that into a typeclass. This chapter and
the two that follow, on Applicative andMonad, will be similar.
Each of these algebras is more powerful than the last, but the
general concept here will remain the same: we abstract out
a common pattern, make certain it follows some laws, give it
an awesome name, and wonder how we ever lived without it.
Monadsort of steals the Haskell spotlight, but you can do more
withFunctor andApplicative than many people realize. Also,
understanding Functor andApplicative is important to a deep
understanding of Monad.
This chapter is all about Functor , andFunctor is all about a
pattern of mapping over structure. We saw fmapway back in
the chapter on lists and noted that it worked just the same as
map, but we alsosaid back then that the diﬀerence is that you
can use fmapwith structures that aren’t lists . Now we will begin
to see what that means.
The great logician Rudolf Carnap appears to have been the
first person to use the word functor in the 1930s. He invented
the word to describe grammatical function words and logical
operations over sentences or phrases. Functors are combina-
tors: they take a sentence or phrase as input and produce a
sentence or phrase as an output, with some logical operation
applied to the whole. For example, negation is a functor in

CHAPTER 16. FUNCTOR 959
this sense because when negation is applied to a sentence, 𝐴,
it produces the negated version, ¬𝐴, as an output. It lifts the
concept of negation over the entire sentence or phrase struc-
ture without changing the internal structure. (Yes, in English
the negation word often appears inside the sentence, not on
the outside, but he was a logician and unconcerned with how
normal humans produced such pedestrian things as spoken
sentences. In logic, the negation operator is typically written
as a prefix, as above.)
This chapter will include:
•the return of the higher-kinded types;
•fmaps galore, and not only on lists;
•no more digressions about dusty logicians;
•words about typeclasses and constructor classes;
•puns based on George Clinton music, probably.
16.2 What’s a functor?
A functor is a way to apply a function over or around some
structure that we don’t want to alter. That is, we want to apply
the function to the value that is “inside” some structure and
leave the structure alone. That’s why it is most common to
introduce functor by way of fmapping over lists, as we did

CHAPTER 16. FUNCTOR 960
back in the lists chapter. The function gets applied to each
value inside the list, and the list structure remains. A good way
to relate “not altering the structure” to lists is that the length
of the list after mapping a function over it will always be the
same. No elements are removed or added, only transformed.
The typeclass Functor generalizes this pattern so that we can
use that basic idea with many types of structure, not just lists.
Functor is implemented in Haskell with a typeclass, just like
Monoid . Other means of implementing it are possible, but this
is the most convenient way to do so. The definition of the
Functor typeclass looks like this:
classFunctor fwhere
fmap::(a->b)->f a->f b
Now let’s dissect this a bit:
classFunctor fwhere
[1] [2] [3] [4]
fmap::(a->b)->f a->f b
[5] [ 6] [ 7] [8]
1.classis the keyword to begin the definition of a typeclass.
2.Functor is the name of the typeclass we are defining.

CHAPTER 16. FUNCTOR 961
3.Typeclasses in Haskell usually refer to a type. The letters
themselves, as with type variables in type signatures, do
not mean anything special. 𝑓is a conventional letter to
choose when referring to types that have functorial struc-
ture. The 𝑓must be the same 𝑓throughout the typeclass
definition.
4.Thewherekeyword ends the declaration of the typeclass
name and associated types. After the wherethe operations
provided by the typeclass are listed.
5.We begin the declaration of an operation named fmap.
6.The argument a -> b is any Haskell function of that type
(remembering that it could be an (a -> a) function for
this purpose).
7.The argument f ais aFunctor𝑓that takes a type argument
𝑎. That is, the 𝑓is a type that has an instance of the Functor
typeclass.
8.The return value is f b. It is the same𝑓fromf a, while
the type argument 𝑏possibly but not necessarily refers to a
diﬀerent type.
Before we delve into the details of how this typeclass works,
let’s see fmapin action so you get a feel for what’s going on first.

CHAPTER 16. FUNCTOR 962
16.3 There’s a whole lot of fmapgoin’
round
We have seen fmapbefore but we haven’t used it much except
for with lists. With lists, it seems to do the same thing as map:
Prelude> map (\x -> x > 3) [1..6]
[False,False,False,True,True,True]
Prelude> fmap (\x -> x > 3) [1..6]
[False,False,False,True,True,True]
Listis, of course, one type that implements the typeclass
Functor , but it seems unremarkable when it just does the same
thing as map. However, Listisn’t the only type that implements
Functor , andfmapcan apply a function over or around any of
those functorial structures, while mapcannot:
Prelude> map (+1) (Just 1)
Couldn't match expected type ‘[b]’
with actual type ‘Maybe a0’
Relevant bindings include
it :: [b] (bound at 16:1)
In the second argument of ‘map’,
namely ‘(Just 1)’

CHAPTER 16. FUNCTOR 963
In the expression: map (+ 1) (Just 1)
Prelude> fmap (+1) (Just 1)
Just 2
Intriguing! What else?
--with a tuple!
Prelude> fmap (10/) (4, 5)
(4,2.0)
--with Either!
Prelude> let rca = Right "Chris Allen"
Prelude> fmap (++ ", Esq.") rca
Right "Chris Allen, Esq."
We can see how the type of fmapspecializes to diﬀerent
types here:

CHAPTER 16. FUNCTOR 964
typeEe=Eithere
typeCe=Constant e
typeI=Identity
-- Functor f =>
fmap::(a->b)->f a->f b
::(a->b)->[ ] a->[ ] b
::(a->b)->Maybea->Maybeb
::(a->b)->Ee a->Ee b
::(a->b)->(e,) a->(e,) b
::(a->b)->Ia->Ib
::(a->b)->Ce a->Ce b
If you are using GHC 8 or newer, you can also see this for
yourself in your REPL by doing this:
Prelude> :set -XTypeApplications
Prelude> :type fmap @Maybe
fmap @Maybe ::
(a -> b) -> Maybe a -> Maybe b
Prelude> :type fmap @(Either _)
fmap @(Either _) ::
(a -> b) -> Either t a -> Either t b
You may have noticed in the tuple and Either examples that
the first arguments (labeled 𝑒in the above chart) are ignored

CHAPTER 16. FUNCTOR 965
byfmap. We’ll talk about why that is in just a bit. Let’s first turn
our attention to what makes a functor. Later we’ll come back
to longer examples and expand on this considerably.
16.4 Let’s talk about 𝑓, baby
As we said above, the 𝑓in the typeclass definition for Functor
must be the same 𝑓throughout the entire definition, and it
must refer to a type that implements the typeclass. This sec-
tion details the practical ramifications of those facts.
The first thing we know is that our 𝑓here must have the kind
* -> * . We talked about higher-kinded types in previous chap-
ters, and we recall that a type constant or a fully applied type
has the kind *. A type with kind * -> * is awaiting application
to a type constant of kind *.
We know that the 𝑓in ourFunctor definition must be kind *
-> *for a couple of reasons, which we will first describe and
then demonstrate:
1.Each argument (and result) in the type signature for a
function must be a fully applied (and inhabitable, modulo
Void, etc.) type. Each argument must have the kind *.
2.The type 𝑓was applied to a single argument in two dif-
ferent places: f aandf b. Since f aandf bmust each
have the kind *,𝑓by itself must be kind * -> * .

CHAPTER 16. FUNCTOR 966
It’s easier to see what these mean in practice by demonstrat-
ing with lots of code, so let’s tear the roof oﬀ this sucker.
Shining star come into view
Every argument to the type constructor of ->must be of kind
*. We can verify this simply by querying kind of the function
type constructor for ourselves:
Prelude> :k (->)
(->) :: * -> * -> *
Each argument and result of every function must be a type
constant, not a type constructor. Given that knowledge, we
can know something about Functor from the type of fmap:
classFunctor fwhere
fmap::(a->b)->f a->f b
--has kind: * -> * -> *
The type signature of fmaptells us that the 𝑓introduced
by the class definition for Functor mustaccept a single type
argument and thus be of kind * -> *. We can determine this
even without knowing anything about the typeclass, which
we’ll demonstrate with some meaningless typeclasses:

CHAPTER 16. FUNCTOR 967
classSumthin awhere
s::a->a
classElsewhere
e::b->f (g a b c)
classBiffywhere
slayer::e a b
->(a->c)
->(b->d)
->e c d
Let’s deconstruct the previous couple of examples:
classSumthin awhere
s::a->a
-- [1] [1]
1.The argument and result type are both 𝑎. There’s nothing
else, so 𝑎has kind *.
classElsewhere
e::b->f (g a b c)
-- [1] [2] [3]
1.This𝑏, like𝑎in the previous example, stands alone as the
first argument to (->), so it is kind *.

CHAPTER 16. FUNCTOR 968
2.Here𝑓is the outermost type constructor for the second
argument (the result type) of (->). It takes a single argu-
ment, the type g a b c wrapped in parentheses. Thus, 𝑓
has kind * -> * .
3.And𝑔is applied to three arguments 𝑎,𝑏, and𝑐. That means
it is kind * -> * -> * -> * , where:
-- using :: to denote kind signature
g:: * -> * -> * -> *
-- a, b, and c are each kind *
g:: * -> * -> * -> *
ga b c (g a b c)
classBiffywhere
slayer::e a b
-- [1]
->(a->c)
-- [2] [3]
->(b->d)
->e c d
1.First,𝑒is an argument to (->)so the application of its
arguments must result in kind *. Given that, and knowing

CHAPTER 16. FUNCTOR 969
there are two arguments, 𝑎and𝑏, we can determine 𝑒is
kind* -> * -> * .
2.This𝑎is an argument to a function that takes no argu-
ments itself, so it’s kind *
3.The story for 𝑐is identical to 𝑎, just in another spot of the
same function.
The kind checker is going to fail on the next couple of
examples:
classImpishvwhere
impossibleKind ::v->v a
classAlsoImp vwhere
nope::v a->v
Remember that the name of the variable before the where
in a typeclass definition binds the occurrences of that name
throughout the definition. GHC will notice that our 𝑣some-
times has a type argument and sometimes not, and it will call
our bluﬀ if we attempt to feed it this nonsense:
‘v’ is applied to too many type arguments
In the type ‘v -> v a’
In the class declaration for ‘Impish’

CHAPTER 16. FUNCTOR 970
Expecting one more argument to ‘v’
Expected a type, but ‘v’ has kind ‘k0 -> *’
In the type ‘v a -> v’
In the class declaration for ‘AlsoImp’
Just as GHC has type inference, it also has kind inference.
And just as it does with types, it can not only infer the kinds
but also validate that they’re consistent and make sense.
Exercises: Be Kind
Given a type signature, determine the kinds of each type vari-
able:
1.What’s the kind of 𝑎?
a->a
2.What are the kinds of 𝑏and𝑇? (The𝑇is capitalized on
purpose!)
a->b a->T(b a)
3.What’s the kind of 𝑐?
ca b->c b a

CHAPTER 16. FUNCTOR 971
A shining star for you to see
So, what if our type isn’t higher kinded? Let’s try it with a type
constant and see what happens:
-- functors1.hs
dataFixMePls =
FixMe
|Pls
deriving (Eq,Show)
instance Functor FixMePls where
fmap=
error
"it doesn't matter, it won't compile"
Notice there are no type arguments anywhere — everything
is one shining (kind) star! And if we load this file from GHCi,
we’ll get the following error:
Prelude> :l functors1.hs
[1 of 1] Compiling Main
( functors1.hs, interpreted )
functors1.hs:8:18:
The first argument of ‘Functor’

CHAPTER 16. FUNCTOR 972
should have kind ‘* -> *’,
but ‘FixMePls’ has kind ‘*’
In the instance declaration for
‘Functor FixMePls’
Failed, modules loaded: none.
In fact, asking for a Functor forFixMePls doesn’t really make
sense. To see why this doesn’t make sense, consider the types
involved:
-- Functor is:
fmap::Functor f=>(a->b)->f a->f b
-- If we replace f with FixMePls
(a->b)->FixMePls a->FixMePls b
-- But FixMePls doesn't take
-- type arguments, so this is
-- really more like:
(FixMePls ->FixMePls )
->FixMePls
->FixMePls
There’s no type constructor 𝑓in there! The maximally
polymorphic version of this is:
(a->b)->a->b

CHAPTER 16. FUNCTOR 973
So in fact, not having a type argument means this is:
($)::(a->b)->a->b
Without a type argument, this is mere function application.
Functor is function application
We just saw how trying to make a Functor instance for a type
constant means you have function application. But, in fact,
fmapis a specific sort of function application. Let’s look at the
types:
fmap::Functor f=>(a->b)->f a->f b
There is also an infix operator for fmap. If you’re using an
older version of GHC, you may need to import Data.Functor
in order to use it in the REPL. Of course, it has the same type
as the prefix fmap:
-- <$> is the infix alias for fmap:
(<$>)::Functor f
=>(a->b)
->f a
->f b
Notice something?

CHAPTER 16. FUNCTOR 974
(<$>)::Functor f
=>(a->b)->f a->f b
($)::(a->b)->a->b
Functor is a typeclass for function application “over”, or
“through”, some structure fthat we want to ignore and leave
untouched. We’ll explain “leave untouched” in more detail
later when we talk about the Functor laws.
A shining star for you to see what your 𝑓can
truly be
Let’s resume our exploration of why we need a higher-kinded
𝑓.
If we add a type argument to the datatype from above, we
makeFixMePls into a type constructor, and this will work:

CHAPTER 16. FUNCTOR 975
-- functors2.hs
dataFixMePls a=
FixMe
|Plsa
deriving (Eq,Show)
instance Functor FixMePls where
fmap=
error
"it doesn't matter, it won't compile"
Now it’ll compile!
Prelude> :l code/functors2.hs
[1 of 1] Compiling Main
Ok, modules loaded: Main.
But wait, we don’t need the error anymore! Let’s fix that
Functor instance:

CHAPTER 16. FUNCTOR 976
-- functors3.hs
dataFixMePls a=
FixMe
|Plsa
deriving (Eq,Show)
instance Functor FixMePls where
fmap_FixMe=FixMe
fmap f ( Plsa)=Pls(f a)
Let’s see how our instance lines up with the type of fmap:
fmap::Functor f
=>(a->b)->f a->f b
fmap f ( Plsa)=Pls(f a)
-- (a -> b) f a f b
While𝑓is used in the type of fmapto represent the Functor ,
by convention, it is also conventionally used in function def-
initions to name an argument that is itself a function. Don’t let
the names fool you into thinking the 𝑓in ourFixMePls instance
is the same 𝑓as in the Functor typeclass definition.
Now our code is happy-making!
Prelude> :l code/functors3.hs

CHAPTER 16. FUNCTOR 977
[1 of 1] Compiling Main
Ok, modules loaded: Main.
Prelude> fmap (+1) (Pls 1)
Pls 2
Notice the function gets applied over and inside of the
structure. This is how Haskell coders lift big heavy functions
over abstract structure!
Okay, let’s make another mistake for the sake of being ex-
plicit. What if we change the type of our Functor instance from
FixMePls toFixMePls a ?
-- functors4.hs
dataFixMePls a=
FixMe
|Plsa
deriving (Eq,Show)
instance Functor (FixMePls a)where
fmap_FixMe=FixMe
fmap f ( Plsa)=Pls(f a)
Notice we didn’t change the type; it still only takes one
argument. But now that argument is part of the 𝑓structure. If
we load this ill-conceived code:

CHAPTER 16. FUNCTOR 978
Prelude> :l functors4.hs
[1 of 1] Compiling Main
functors4.hs:8:19:
The first argument of ‘Functor’
should have kind ‘* -> *’,
but ‘FixMePls a’ has kind ‘*’
In the instance declaration for
‘Functor (FixMePls a)’
Failed, modules loaded: none.
We get the same error as earlier, because applying the type
constructor gave us something of kind *from the original kind
of* -> * .
Typeclasses and constructor classes
You may have initially paused on the type constructor 𝑓in
the definition of Functor having kind * -> * — this is quite
natural! In fact, earlier versions of Haskell didn’t have a facility
for expressing typeclasses in terms of higher-kinded types
at all. This was developed by Mark P. Jones1while he was
working on an implementation of Haskell called Gofer. This
work generalized typeclasses from being usable only with
types of kind *(also called type constants ) to being usable with

CHAPTER 16. FUNCTOR 979
higher-kinded types, called type constructors , as well.
In Haskell, the two use cases have been merged such that
we don’t call out constructor classes as being separate from
typeclasses, but we think it’s useful to highlight that something
significant has happened here. Now we have a means of talking
about the contents of types independently from the type that
structures those contents. That’s why we can have something
likefmapthat allows us to alter the contents of a value without
altering the structure (a list, or a Just) around the value.
16.5 Functor Laws
Instances of the Functor typeclass should abide by two basic
laws. Understanding these laws is critical for understanding
Functor and writing typeclass instances that are composable
and easy to reason about.
Identity
The first law is the law of identity:
fmapid==id
If wefmapthe identity function, it should have the same
result as passing our value to identity. We shouldn’t be chang-
1A system of constructor classes: overloading and implicit higher-order polymor-
phism
http://www.cs.tufts.edu/~nr/cs257/archive/mark-jones/fpca93.pdf

CHAPTER 16. FUNCTOR 980
ing any of the outer structure 𝑓that we’re mapping over by
mapping id. That’s why it’s the same as id. If we didn’t return
a new value in the a -> b function mapped over the structure,
then nothing should’ve changed:
Prelude> fmap id "Hi Julie"
"Hi Julie"
Prelude> id "Hi Julie"
"Hi Julie"
Try it out on a few diﬀerent structures and check for your-
self.
Composition
The second law for Functor is the law of composition:
fmap(f.g)==fmap f.fmap g
This concerns the composability of fmap. If we compose
two functions, 𝑓and𝑔, andfmapthat over some structure, we
should get the same result as if we fmapped them and then
composed them:
Prelude> fmap ((+1) . (*2)) [1..5]
[3,5,7,9,11]
Prelude> fmap (+1) . fmap (*2) $ [1..5]
[3,5,7,9,11]

CHAPTER 16. FUNCTOR 981
If an implementation of fmapdoesn’t do that, it’s a broken
functor.
Structure preservation
Both of these laws touch on the essential rule that functors
must be structure preserving.
All we’re allowed to know in the type about our instance of
Functor implemented by 𝑓is that it implements Functor :
fmap::Functor f=>(a->b)->f a->f b
The𝑓is constrained by the typeclass Functor , but that is all
we know about its type from this definition. As we’ve seen with
typeclass-constrained polymorphism, this still allows it to be
any type that has an instance of Functor . The core operation
that this typeclass provides for these types is fmap. Because the
𝑓persists through the type of fmap, whatever the type is, we
know it must be a type that can take an argument, as in f aand
f band that it will be the “structure” we’re lifting the function
over when we apply it to the value inside.
16.6 The Good, the Bad, and the Ugly
We’ll get a better picture of what it means for Functor instances
to be law-abiding or law-breaking by walking through some

CHAPTER 16. FUNCTOR 982
examples. We start by definining a type constructor with one
argument:
dataWhoCares a=
ItDoesnt
|Mattera
|WhatThisIsCalled
deriving (Eq,Show)
This datatype only has one data constructor containing a
value we could fmapover, and that is Matter. The others are
nullary so there is no value to work with inside the structure;
there is only structure.
Here we see a law-abiding instance:
instance Functor WhoCares where
fmap_ItDoesnt =ItDoesnt
fmap_WhatThisIsCalled =
WhatThisIsCalled
fmap f ( Mattera)=Matter(f a)
Our instance must follow the identity law or else it’s not a
valid functor. That law dictates that fmap id (Matter _) must
nottouchMatter — that is, it must be identical to id (Matter _) .
Functor is a way of lifting over structure (mapping) in such a
manner that you don’t have to care about the structure because
you’re not allowed to touch the structure anyway.

CHAPTER 16. FUNCTOR 983
Let us next consider a law-breaking instance:
instance Functor WhoCares where
fmap_ItDoesnt =WhatThisIsCalled
fmap fWhatThisIsCalled =ItDoesnt
fmap f ( Mattera)=Matter(f a)
Nowwecontemplatewhatitmeanstoleavethestructureun-
touched. In this instance, we’ve made our structure — not the
values wrapped or contained within the structure — change
by making ItDoesnt andWhatThisIsCalled do a little dosey-do.
It becomes rapidly apparent why this isn’t kosher at all.
Prelude> fmap id ItDoesnt
WhatThisIsCalled
Prelude> fmap id WhatThisIsCalled
ItDoesnt
Prelude> fmap id ItDoesnt == id ItDoesnt
False
Prelude> :{
*Main| fmap id WhatThisIsCalled ==
*Main| id WhatThisIsCalled
*Main| :}
False
This certainly does not abide by the identity law. It is not a
validFunctor instance.

CHAPTER 16. FUNCTOR 984
The law won But what if you dowant a function that can
change the value andthe structure?
We’ve got wonderful news for you: that exists! It’s a plain
old function. Write one. Write many! The point of Functor is
to reify and be able to talk about cases where we want to reuse
functions in the presence of more structure and be transpar-
entlyoblivious to that additional structure. We already saw that
Functor is in some sense a special sort of function application,
but since it is special , we want to preserve the things about
it that make it diﬀerent and more powerful than ordinary
function application. So, we stick to the laws.
Later in this chapter, we will talk about a sort of opposite,
where you can transform the structure but leave the type ar-
gument alone. This has a special name too, but there isn’t a
widely agreed upon typeclass.
Composition should just work
All right, now that we’ve seen how we can make a Functor in-
stance violate the identity law, let’s take a look at how we abide
by — and break! — the composition law. You may recall from
above that the law looks like this:
fmap (f . g) == fmap f . fmap g
Technically this follows from fmap id == id , but it’s worth
calling out so that we can talk about composition. This law

CHAPTER 16. FUNCTOR 985
says composing two functions lifted separately should pro-
duce the same result as if we composed the functions ahead
of time and then lifted the composed function all together.
Maintaining this property is about preserving composability
of our code and preventing our software from doing unpleas-
antly surprising things. We will now consider another invalid
Functor instance to see why this is bad news:
dataCountingBad a=
Heisenberg Inta
deriving (Eq,Show)
-- super NOT okay
instance Functor CountingBad where
fmap f ( Heisenberg n a)=
-- (a -> b) f a =
Heisenberg (n+1) (f a)
-- f b
Well, what did we do here? CountingBad has one type argu-
ment, but Heisenberg has two arguments. If you look at how
that lines up with the type of fmap, you get a hint of why this
isn’t going to work out well. What part of our fmaptype does
the𝑛representing the Intargument to Heisenberg belong to?
We can load this horribleness up in the REPL and see that
composing two fmaps here does not produce the same results,

CHAPTER 16. FUNCTOR 986
so the composition law doesn’t hold:
Prelude> let u = "Uncle"
Prelude> let oneWhoKnocks = Heisenberg 0 u
Prelude> fmap (++" Jesse") oneWhoKnocks
Heisenberg 1 "Uncle Jesse"
Prelude> let f = ((++" Jesse").(++" lol"))
Prelude> fmap f oneWhoKnocks
Heisenberg 1 "Uncle lol Jesse"
So far it seems OK, but what if we compose the two con-
catenation functions separately?
Prelude> let j = (++ " Jesse")
Prelude> let l = (++ " lol")
Prelude> fmap j . fmap l $ oneWhoKnocks
Heisenberg 2 "Uncle lol Jesse"
Or to make it look more like the law:
Prelude> let f = (++" Jesse")
Prelude> let g = (++" lol")
Prelude> fmap (f . g) oneWhoKnocks
Heisenberg 1 "Uncle lol Jesse"
Prelude> fmap f . fmap g $ oneWhoKnocks
Heisenberg 2 "Uncle lol Jesse"
We can clearly see that

CHAPTER 16. FUNCTOR 987
fmap (f . g) == fmap f . fmap g
does not hold. So how do we fix it?
dataCountingGood a=
Heisenberg Inta
deriving (Eq,Show)
-- Totes cool.
instance Functor CountingGood where
fmap f ( Heisenberg n a)=
Heisenberg (n) (f a)
Stop messing with the IntinHeisenberg . Think of anything
that isn’t the final type argument of our 𝑓inFunctor as being
part of the structure that the functions being lifted should be
oblivious to.
16.7 Commonly used functors
Now that we have a sense of what Functor does for us and
how it’s meant to work, it’s time to start working through
some longer examples. This section is nearly all code and
examples with minimal prose explanation. Interacting with
these examples will help you develop an intuition for what’s
going on with a minimum of fuss.
We begin with a utility function:

CHAPTER 16. FUNCTOR 988
Prelude> :t const
const :: a -> b -> a
Prelude> let replaceWithP = const 'p'
Prelude> replaceWithP 10000
'p'
Prelude> replaceWithP "woohoo"
'p'
Prelude> replaceWithP (Just 10)
'p'
We’ll use it with fmapnow for various datatypes that have
instances:
-- data Maybe a = Nothing | Just a
Prelude> fmap replaceWithP (Just 10)
Just 'p'
Prelude> fmap replaceWithP Nothing
Nothing
-- data [] a = [] | a : [a]
Prelude> fmap replaceWithP [1, 2, 3, 4, 5]
"ppppp"
Prelude> fmap replaceWithP "Ave"
"ppp"

CHAPTER 16. FUNCTOR 989
Prelude> fmap (+1) []
[]
Prelude> fmap replaceWithP []
""
-- data (,) a b = (,) a b
Prelude> fmap replaceWithP (10, 20)
(10,'p')
Prelude> fmap replaceWithP (10, "woo")
(10,'p')
Again, we’ll talk about why it skips the first value in the
tuple in a bit. It has to do with the kindedness of tuples and
the kindedness of the 𝑓inFunctor .
Now the instance for functions:
Prelude> negate 10
-10
Prelude> let tossEmOne = fmap (+1) negate
Prelude> tossEmOne 10
-9
Prelude> tossEmOne (-10)
11
The functor of functions won’t be discussed in great detail
until we get to the chapter on Reader, but it should look sort
of familiar:

CHAPTER 16. FUNCTOR 990
Prelude> let tossEmOne' = (+1) . negate
Prelude> tossEmOne' 10
-9
Prelude> tossEmOne' (-10)
11
Now you’re starting to get into the groove; let’s see what
else we can do with our fancy new moves.
The functors are stacked and that’s a fact
We can combine datatypes, as we’ve seen, usually by nesting
them. We’ll be using the tilde character as a shorthand for “is
roughly equivalent to” throughout these examples:
-- lms ~ List (Maybe (String))
Prelude> let n = Nothing
Prelude> let w = Just "woohoo"
Prelude> let ave = Just "Ave"
Prelude> let lms = [ave, n, w]
Prelude> let replaceWithP = const 'p'
Prelude> replaceWithP lms
'p'
Prelude> fmap replaceWithP lms
"ppp"

CHAPTER 16. FUNCTOR 991
Nothing unexpected there, but we notice that lmshas more
than one Functor type.Maybeand List (which includes String)
both have Functor instances. So, are we obligated to fmaponly
to the outermost datatype? No way, mate:
Prelude> (fmap . fmap) replaceWithP lms
[Just 'p',Nothing,Just 'p']
Prelude> let tripFmap = fmap . fmap . fmap
Prelude> tripFmap replaceWithP lms
[Just "ppp",Nothing,Just "pppppp"]
Let’s review in detail:
-- lms ~ List (Maybe String)
Prelude> let ave = Just "Ave"
Prelude> let n = Nothing
Prelude> let w = Just "woohoo"
Prelude> let lms = [ave, n, w]
Prelude> replaceWithP lms
'p'
Prelude> :t replaceWithP lms
replaceWithP lms :: Char
-- In:

CHAPTER 16. FUNCTOR 992
replaceWithP lms
-- replaceWithP's input type is:
List (Maybe String)
-- The output type is Char
-- So applying
replaceWithP
-- to
lms
-- accomplishes
List (Maybe String) -> Char
The output type of replaceWithP is always the same.
If we do this:
Prelude> fmap replaceWithP lms
"ppp"
-- fmap is going to leave the list
-- structure intact around our result:
Prelude> :t fmap replaceWithP lms
fmap replaceWithP lms :: [Char]

CHAPTER 16. FUNCTOR 993
Here’s the X-ray view:
-- In:
fmap replaceWithP lms
-- replaceWithP's input type is:
Maybe String
-- The output type is Char
-- So applying
fmap replaceWithP
-- to
lms
-- accomplishes:
List (Maybe String) -> List Char
-- List Char ~ String
What if we lift twice?
Keep on stacking them up:
Prelude> (fmap . fmap) replaceWithP lms
[Just 'p',Nothing,Just 'p']

CHAPTER 16. FUNCTOR 994
Prelude> :t (fmap . fmap) replaceWithP lms
(fmap . fmap) replaceWithP lms
:: [Maybe Char]
And the X-ray view:
-- In:
(fmap . fmap) replaceWithP lms
-- replaceWithP's input type is:
-- String aka List Char or [Char]
-- The output type is Char
-- So applying
(fmap . fmap) replaceWithP
-- to
lms
-- accomplishes
List (Maybe String) -> List (Maybe Char)
Wait, how does that even typecheck? It may not seem obvi-
ous at first how (fmap . fmap) could typecheck. We’re going to

CHAPTER 16. FUNCTOR 995
ask you to work through the types. You might prefer to write
it out with pen and paper, as Julie does, or type it all out in a
text editor, as Chris does. We’ll help you out by providing the
type signatures. Since the two fmapfunctions being composed
could have diﬀerent types, we’ll make the type variables for
each function unique. Start by substituting the type of each
fmapfor each of the function types in the (.)signature:
(.)::(b->c)->(a->b)->a->c
-- fmap fmap
fmap::Functor f=>(m->n)->f m->f n
fmap::Functor g=>(x->y)->g x->g y
It might also be helpful to query the type of (fmap . fmap)
to get an idea of what your end type should look like (modulo
diﬀerent type variables).
Lift me baby one more time
We have another layer we can lift over if we wish:
Prelude> let tripFmap = fmap . fmap . fmap
Prelude> tripFmap replaceWithP lms
[Just "ppp",Nothing,Just "pppppp"]
Prelude> :t tripFmap replaceWithP lms
(fmap . fmap . fmap) replaceWithP lms

CHAPTER 16. FUNCTOR 996
:: [Maybe [Char]]
And the X-ray view:
-- In
(fmap . fmap . fmap) replaceWithP lms
-- replaceWithP's input type is:
-- Char
-- because we lifted over
-- the [] of [Char]
-- The output type is Char
-- So applying
(fmap . fmap . fmap) replaceWithP
-- to
lms
-- accomplishes
List (Maybe String) -> List (Maybe String)
So, we see there’s a pattern.

CHAPTER 16. FUNCTOR 997
The real type of thing going down
We saw the pattern above, but for clarity we’ll summarize here
before moving on:
Prelude> fmap replaceWithP lms
"ppp"
Prelude> (fmap . fmap) replaceWithP lms
[Just 'p',Nothing,Just 'p']
Prelude> let tripFmap = fmap . fmap . fmap
Prelude> tripFmap replaceWithP lms
[Just "ppp",Nothing,Just "pppppp"]
Let’s summarize the types, too, to validate our understand-
ing:

CHAPTER 16. FUNCTOR 998
-- replacing the type synonym String
-- with the underlying type [Char]
-- intentionally
replaceWithP' ::[Maybe[Char]]->Char
replaceWithP' =replaceWithP
[Maybe[Char]]->[Char]
[Maybe[Char]]->[MaybeChar]
[Maybe[Char]]->[Maybe[Char]]
Pause for a second and make sure you’re understanding
everything we’ve done so far. If not, play with it until it starts
to feel comfortable.
Get on up and get down
We’ll work through the same idea, but with more funky struc-
ture to lift over:
-- lmls ~ List (Maybe (List String))
Prelude> let ha = Just ["Ha", "Ha"]
Prelude> let lmls = [ha, Nothing, Just []]
Prelude> (fmap . fmap) replaceWithP lmls
[Just 'p',Nothing,Just 'p']

CHAPTER 16. FUNCTOR 999
Prelude> let tripFmap = fmap . fmap . fmap
Prelude> tripFmap replaceWithP lmls
[Just "pp",Nothing,Just ""]
Prelude> (tripFmap.fmap) replaceWithP lmls
[Just ["pp","pp"],Nothing,Just []]
See if you can trace the changing result types as we did
above.
One more round for the P-Funkshun
For those who like their funk uncut, here’s another look at the
changing types that result from lifting over multiple layers of
functorial structure, with a slightly higher resolution. We start
this time from a source file:

CHAPTER 16. FUNCTOR 1000
moduleReplaceExperiment where
replaceWithP ::b->Char
replaceWithP =const'p'
lms::[Maybe[Char]]
lms=[Just"Ave",Nothing,Just"woohoo" ]
-- Just making the argument more specific
replaceWithP' ::[Maybe[Char]]->Char
replaceWithP' =replaceWithP
What happens if we lift it?
-- Prelude> :t fmap replaceWithP
-- fmap replaceWithP :: Functor f
-- => f a -> f Char
liftedReplace ::Functor f=>f a->fChar
liftedReplace =fmap replaceWithP
But we can assert a more specific type for liftedReplace !
liftedReplace' ::[Maybe[Char]]->[Char]
liftedReplace' =liftedReplace

CHAPTER 16. FUNCTOR 1001
The[]around Char is the 𝑓off Char, or the structure we
lifted over. The 𝑓off ais the outermost []in [Maybe [Char]].
So,𝑓is instantiated to []when we make the type more specific,
whether by applying it to a value of type [Maybe [Char]] or by
means of explicitly writing liftedReplace' .
Stay on the scene like an fmapmachine
What if we lift it twice?
-- Prelude> :t (fmap . fmap) replaceWithP
-- (fmap . fmap) replaceWithP
-- :: (Functor f1, Functor f)
-- => f (f1 a) -> f (f1 Char)
twiceLifted ::(Functor f1,Functor f)=>
f (f1 a) ->f (f1Char)
twiceLifted =(fmap.fmap) replaceWithP
-- Making it more specific
twiceLifted' ::[Maybe[Char]]
->[MaybeChar]
twiceLifted' =twiceLifted
-- f ~ []
-- f1 ~ Maybe
Thrice?

CHAPTER 16. FUNCTOR 1002
-- Prelude> let rWP = replaceWithP
-- Prelude> :t (fmap . fmap . fmap) rWP
-- (fmap . fmap . fmap) replaceWithP
-- :: (Functor f2, Functor f1, Functor f)
-- => f (f1 (f2 a)) -> f (f1 (f2 Char))
thriceLifted ::
(Functor f2,Functor f1,Functor f)
=>f (f1 (f2 a)) ->f (f1 (f2 Char))
thriceLifted =
(fmap.fmap.fmap) replaceWithP
-- More specific or "concrete"
thriceLifted' ::[Maybe[Char]]
->[Maybe[Char]]
thriceLifted' =thriceLifted
-- f ~ []
-- f1 ~ Maybe
-- f2 ~ []
Now we can print the results from our expressions and
compare them:

CHAPTER 16. FUNCTOR 1003
main::IO()
main= do
putStr"replaceWithP' lms: "
print (replaceWithP' lms)
putStr"liftedReplace lms: "
print (liftedReplace lms)
putStr"liftedReplace' lms: "
print (liftedReplace' lms)
putStr"twiceLifted lms: "
print (twiceLifted lms)
putStr"twiceLifted' lms: "
print (twiceLifted' lms)
putStr"thriceLifted lms: "
print (thriceLifted lms)
putStr"thriceLifted' lms: "
print (thriceLifted' lms)
Be sure to type all this into a file, load it in GHCi, run main
to see what output results. Then, modify the types and code-

CHAPTER 16. FUNCTOR 1004
based ideas and guesses of what should and shouldn’t work.
Forming hypotheses, creating experiments based on them
or modifying existing experiments, and validating them is a
critical part of becoming comfortable with abstractions like
Functor !
Exercises: Heavy Lifting
Addfmap, parentheses, and function composition to the expres-
sion as needed for the expression to typecheck and produce
the expected result. It may not always need to go in the same
place, so don’t get complacent.
1.a=(+1)$read"[1]"::[Int]
Expected result
Prelude> a
[2]
2.b=(++"lol") (Just["Hi,","Hello"])
Prelude> b
Just ["Hi,lol","Hellolol"]
3.c=(*2) (\x->x-2)

CHAPTER 16. FUNCTOR 1005
Prelude> c 1
-2
4.d=
((return '1'++).show)
(\x->[x,1..3])
Prelude> d 0
"1[0,1,2,3]"
5.e::IOInteger
e= letioi=readIO"1"::IOInteger
changed =read ("123"++) show ioi
in(*3) changed
Prelude> e
3693
16.8 Transforming the unapplied type
argument
We’veseen that 𝑓mustbeahigher-kindedtypeandthat Functor
instances must abide by two laws, and we’ve played around
with some basic fmapping. We know that the goal of fmapping
is to leave the outer structure untouched while transforming
the type arguments inside.

CHAPTER 16. FUNCTOR 1006
Way back in the beginning, we noticed that when we fmap
over a tuple, it only transforms the second argument (the 𝑏).
We saw a similar thing when we fmapped over an Either value,
and we said we’d come back to this topic. Then we saw another
hint of it above in the Heisenberg example. Now the time has
come to talk about what happens to the other type arguments
(if any) when we can only tranform the innermost.
We’ll start with a couple of canonical types:
dataTwoa b=
Twoa b
deriving (Eq,Show)
dataOra b=
Firsta
|Secondb
deriving (Eq,Show)
You may recognize these as (,)andEither recapitulated, the
generic product and sum types, from which any combination
ofandandormay be made. But these are both kind * -> *
-> *, which isn’t compatible with Functor , so how do we write
Functor instances for them?
These wouldn’t work because TwoandOrhave the wrong
kind:

CHAPTER 16. FUNCTOR 1007
instance Functor Twowhere
fmap=undefined
instance Functor Orwhere
fmap=undefined
We know that we can partially apply functions, and we’ve
seen previously that we can do this:
Prelude> :k Either
Either :: * -> * -> *
Prelude> :k Either Integer
Either Integer :: * -> *
Prelude> :k Either Integer String
Either Integer String :: *
That has the eﬀect of applying out some of the arguments,
reducing the kindedness of the type. Previously, we’ve demon-
strated this by applying the type constructor to concrete types;
however, you can also apply it to a type variable that represents
a type constant to produce the same eﬀect.
So to fix the kind incompatibility for our TwoandOrtypes,
we apply one of the arguments of each type constructor, giving
us kind * -> * :

CHAPTER 16. FUNCTOR 1008
-- we use 'a' for clarity, so you
-- can see more readily which type
-- was applied out but the letter
-- doesn't matter.
instance Functor (Twoa)where
fmap=undefined
instance Functor (Ora)where
fmap=undefined
These will pass the typechecker already, but we still need
to write the implementations of fmapfor both, so let’s proceed.
First we’ll turn our attention to Two:
instance Functor (Twoa)where
fmap f ( Twoa b)=Two$(f a) (f b)
This won’t fly, because the 𝑎is part of the functorial struc-
ture (the 𝑓). We’re not supposed to touch anything in the 𝑓
referenced in the type of fmap, so we can’t apply the function
(named 𝑓in ourfmapdefinition) to the 𝑎because the 𝑎is now
untouchable.

CHAPTER 16. FUNCTOR 1009
fmap::Functor f=>(a->b)->f a->f b
-- here, f is (Two a) because
classFunctor fwhere
fmap::(a->b)->f a->f b
instance Functor (Twoa)where
-- remember, names don't mean
-- anything beyond their relationships
-- to each other.
::(a->b)->(Twoz) a->(Twoz) b
So to fix our Functor instance, we have to leave the left value
(it’s part of the structure of 𝑓) inTwoalone, and have our func-
tion only apply to the innermost value, in this case named
𝑏:
instance Functor (Twoa)where
fmap f ( Twoa b)=Twoa (f b)
Then with Or, we’re dealing with the independent possibility
of two diﬀerent values and types, but the same basic constraint
applies:

CHAPTER 16. FUNCTOR 1010
instance Functor (Ora)where
fmap_(Firsta)=Firsta
fmap f ( Secondb)=Second(f b)
We’ve applied out the first argument, so now it’s part of the
𝑓. The function we’re mapping around that structure can only
transform the innermost argument.
16.9 QuickChecking Functor instances
We know the Functor laws are the following:
fmapid =id
fmap(p.q)=(fmap p) .(fmap q)
We can turn those into the following QuickCheck properties:

CHAPTER 16. FUNCTOR 1011
functorIdentity ::(Functor f,Eq(f a))=>
f a
->Bool
functorIdentity f=
fmap id f ==f
functorCompose ::(Eq(f c),Functor f)=>
(a->b)
->(b->c)
->f a
->Bool
functorCompose f g x=
(fmap g (fmap f x)) ==(fmap (g .f) x)
As long as we provided concrete instances, we can now run
these to test them.
Prelude> :{
*Main| let f :: [Int] -> Bool
*Main| f x = functorIdentity x
*Main| :}
Prelude> quickCheck f
+++ OK, passed 100 tests.
Prelude> let c = functorCompose (+1) (*2)
Prelude> let li x = c (x :: [Int])

CHAPTER 16. FUNCTOR 1012
Prelude> quickCheck li
+++ OK, passed 100 tests.
Groovy.
Making QuickCheck generate functions too
QuickCheck happens to oﬀer the ability to generate functions.
There’s a typeclass called CoArbitrary that covers the function
argument type, whereas the (related) Arbitrary typeclass is used
for the function result type. If you’re curious about this, take
a look at the Function module in the QuickCheck library to see
how functions are generated from a datatype that represents
patterns in function construction.

CHAPTER 16. FUNCTOR 1013
{-# LANGUAGE ViewPatterns #-}
importTest.QuickCheck
importTest.QuickCheck.Function
functorCompose' ::(Eq(f c),Functor f)=>
f a
->Funa b
->Funb c
->Bool
functorCompose' x (Fun_f) (Fun_g)=
(fmap (g .f) x)==(fmap g .fmap f$x)
There are a couple things going on here. One is that we
needed to import a new module from QuickCheck . Another
is that we’re pattern matching on the Funvalue that we’re
askingQuickCheck to generate. The underlying Funtype is es-
sentially a product of the weird function type and an ordi-
nary Haskell function generated from the weirdo. The weirdo
QuickCheck -specific concrete function is a function represented
by a datatype which can be inspected and recursed. We only
want the second part, the ordinary Haskell function, so we’re
pattern-matching that one out.
Prelude> type IntToInt = Fun Int Int
Prelude> :{

CHAPTER 16. FUNCTOR 1014
*Main| type IntFC =
*Main| [Int]
*Main| -> IntToInt
*Main| -> IntToInt
*Main| -> Bool
*Main| :}
Prelude> let fc' = functorCompose'
Prelude> quickCheck (fc' :: IntFC)
+++ OK, passed 100 tests.
Noteofwarning, youcan’tprintthose Funvalues, so verboseCheck
will curse Socrates and spin in a circle if you try it.
16.10 Exercises: Instances of Func
Implement Functor instances for the following datatypes. Use
theQuickCheck properties we showed you to validate them.
1.newtype Identity a=Identity a
2.dataPaira=Paira a
3.dataTwoa b=Twoa b
4.dataThreea b c=Threea b c
5.dataThree'a b=Three'a b b

CHAPTER 16. FUNCTOR 1015
6.dataFoura b c d =Foura b c d
7.dataFour'a b=Four'a a a b
8.Can you implement one for this type? Why? Why not?
dataTrivial =Trivial
Doingtheseexercisesis critical tounderstandinghow Functor
works, do not skip past them!
16.11 Ignoring possibilities
We’ve already touched on the MaybeandEither functors. Now
we’ll examine in a bit more detail what those do for us. As
the title of this section suggests, the Functor instances for these
datatypes are handy for times you intend to ignore the left
cases, which are typically your error or failure cases. Because
fmapdoesn’t touch those cases, you can map your function
right to the values that you intend to work with and ignore
those failure cases.
Maybe
Let’s start with some ordinary pattern matching on Maybe:

CHAPTER 16. FUNCTOR 1016
incIfJust ::Numa=>Maybea->Maybea
incIfJust (Justn)=Just$n+1
incIfJust Nothing =Nothing
showIfJust ::Showa
=>Maybea
->MaybeString
showIfJust (Justs)=Just$show s
showIfJust Nothing =Nothing
Well, that’s boring, and there’s some redundant structure.
For one thing, they have the Nothing case in common:
someFunc Nothing =Nothing
Then they’re applying some function to the value if it’s a
Just:
someFunc (Justx)=Just$someOtherFunc x
What happens if we use fmap?

CHAPTER 16. FUNCTOR 1017
incMaybe ::Numa=>Maybea->Maybea
incMaybe m=fmap (+1) m
showMaybe ::Showa
=>Maybea
->MaybeString
showMaybe s=fmap show s
That appears to have cleaned things up a bit. Does it still
work?
Prelude> incMaybe (Just 1)
Just 2
Prelude> incMaybe Nothing
Nothing
Prelude> showMaybe (Just 9001)
Just "9001"
Prelude> showMaybe Nothing
Nothing
Yeah,fmaphas no reason to concern itself with the Nothing
— there’s no value there for it to operate on, so this all seems
to be working properly.
But we can abstract this a bit more. For one thing, we can
eta-reduce these functions. That is, we can rewrite them with-
out naming the arguments:

CHAPTER 16. FUNCTOR 1018
incMaybe'' ::Numa=>Maybea->Maybea
incMaybe'' =fmap (+1)
showMaybe'' ::Showa
=>Maybea
->MaybeString
showMaybe'' =fmap show
And they don’t even really have to be specific to Maybe!fmap
works for all datatypes with a Functor instance! We can query
the type of the expressions in GHCi and see for ourselves the
more generic type:
Prelude> :t fmap (+1)
fmap (+1)
:: (Functor f, Num b) => f b -> f b
Prelude> :t fmap show
fmap show
:: (Functor f, Show a) => f a -> f String
With that, we can rewrite them as much more generic func-
tions:

CHAPTER 16. FUNCTOR 1019
-- ``lifted'' because they've been
-- lifted over some structure f
liftedInc ::(Functor f,Numb)
=>f b->f b
liftedInc =fmap (+1)
liftedShow ::(Functor f,Showa)
=>f a->fString
liftedShow =fmap show
And they have the same behavior as always:
Prelude> liftedInc (Just 1)
Just 2
Prelude> liftedInc Nothing
Nothing
Prelude> liftedShow (Just 1)
Just "1"
Prelude> liftedShow Nothing
Nothing
Making them more polymorphic in the type of the functo-
rial structure means they’re more reusable now:
Prelude> liftedInc [1..5]

CHAPTER 16. FUNCTOR 1020
[2,3,4,5,6]
Prelude> liftedShow [1..5]
["1","2","3","4","5"]
Exercise: Possibly
Write a Functor instance for a datatype identical to Maybe. We’ll
use our own datatype because Maybealready has a Functor in-
stance and we cannot make a duplicate one.
dataPossibly a=
LolNope
|Yeppers a
deriving (Eq,Show)
instance Functor Possibly where
fmap=undefined
If it helps, you’re basically writing the following function:
applyIfJust ::(a->b)
->Maybea
->Maybeb

CHAPTER 16. FUNCTOR 1021
Either
TheMaybetype solves some problems for Haskellers, but it
doesn’t solve all of them. As we saw in a previous chapter,
sometimes we want to preserve the reason whya computation
failed rather than only the information thatit failed. And for
that, we use Either .
By this point, you know that Either has aFunctor instance
inbasefor grateful programmers to use. So let’s put it to use.
We’ll stick to the same pattern we used for demonstrating
Maybe, for the sake of clarity:
incIfRight ::Numa
=>Eithere a
->Eithere a
incIfRight (Rightn)=Right$n+1
incIfRight (Lefte)=Lefte
showIfRight ::Showa
=>Eithere a
->EithereString
showIfRight (Rights)=Right$show s
showIfRight (Lefte)=Lefte
Once again we can simplify these using fmapso we don’t
have to address the case of leaving the error value alone:

CHAPTER 16. FUNCTOR 1022
incEither ::Numa
=>Eithere a
->Eithere a
incEither m=fmap (+1) m
showEither ::Showa
=>Eithere a
->EithereString
showEither s=fmap show s
And again we can eta-contract to drop the obvious argu-
ment:
incEither' ::Numa
=>Eithere a
->Eithere a
incEither' =fmap (+1)
showEither' ::Showa
=>Eithere a
->EithereString
showEither' =fmap show
And once againwe are confronted with functions that really
didn’t need to be specific to Either at all:

CHAPTER 16. FUNCTOR 1023
-- f ~ Either e
liftedInc ::(Functor f,Numb)
=>f b->f b
liftedInc =fmap (+1)
liftedShow ::(Functor f,Showa)
=>f a->fString
liftedShow =fmap show
Take a few moments to play around with this and note how
it works.
Short Exercise
1.Write a Functor instance for a datatype identical to Either .
We’ll use our own datatype because Either has aFunctor
instance.
dataSuma b=
Firsta
|Secondb
deriving (Eq,Show)
instance Functor (Suma)where
fmap=undefined

CHAPTER 16. FUNCTOR 1024
Your hint for this one is that you’re writing the following
function.
applyIfSecond ::(a->b)
->(Sume) a
->(Sume) b
2.Why is a Functor instance that applies the function only to
First,Either ’sLeft, impossible? We covered this earlier.
16.12 A somewhat surprising functor
There’s a datatype named ConstorConstant — you’ll see both
names depending on which library you use. Constant has a
validFunctor , but the behavior of the Functor instance may
surprise you a bit. First, let’s look at the constfunction , and
then we’ll look at the datatype:
Prelude> :t const
const :: a -> b -> a
Prelude> let a = const 1
Prelude> a 1
1
Prelude> a 2
1
Prelude> a 3
1

CHAPTER 16. FUNCTOR 1025
Prelude> a "blah"
1
Prelude> a id
1
Withasimilarconceptinmind, thereisthe Constant datatype.
Constant looks like this:
newtype Constant a b=
Constant { getConstant ::a }
deriving (Eq,Show)
One thing we notice about this type is that the type param-
eter𝑏is aphantom type. It has no corresponding witness at
the value/term level. This is a concept and tactic we’ll explore
more later, but for now we can see how it echoes the function
const:
Prelude> Constant 2
Constant {getConstant = 2}
Despite 𝑏being a phantom type, though, Constant is kind*
-> * -> * , and that is not a valid Functor . So how do we get one?
Well, there’s only one thing we can do with a type constructor,
just as with functions: apply it. So we dohave aFunctor for
Constant a , but not Constant alone. It has to be Constant a and
notConstant a b because Constant a b would be kind *.
Let’s look at the implementation of Functor forConstant :

CHAPTER 16. FUNCTOR 1026
instance Functor (Constant m)where
fmap_(Constant v)=Constant v
Looks like identity right? Let’s use this in the REPL and run
it through the Functor laws:
Prelude> const 2 (getConstant (Constant 3))
2
Prelude> fmap (const 2) (Constant 3)
Constant {getConstant = 3}
Prelude> let gc = getConstant
Prelude> let c = Constant 3
Prelude> gc $ fmap (const 2) c
3
Prelude> gc $ fmap (const "blah") c
3
When you fmaptheconstfunction over the Constant type,
the first argument to constis never used because the partially
applied constis itself never used. The first type argument to
Constant ’s type constructor is in the part of the structure that
Functor skips over. The second argument to the Constant type
constructor is the phantom type variable 𝑏which has no value
or term-level witness in the datatype. Since there are no values
of the type the Functor is supposed to be mapping, we have

CHAPTER 16. FUNCTOR 1027
nothing we’re allowed to apply the function to, so we never
use the constexpressions.
But does this adhere to the Functor laws?
-- Testing identity
Prelude> getConstant (id (Constant 3))
3
Prelude> getConstant (fmap id (Constant 3))
3
-- Composition of the const function
Prelude> ((const 3) . (const 5)) 10
3
Prelude> ((const 5) . (const 3)) 10
5
-- Composition
Prelude> let fc = fmap (const 3)
Prelude> let fc' = fmap (const 5)
Prelude> let separate = fc . fc'
Prelude> let c = const 3
Prelude> let c' = const 5
Prelude> let fused = fmap (c . c')
Prelude> let cw = Constant "WOOHOO"
Prelude> getConstant $ separate $ cw
"WOOHOO"

CHAPTER 16. FUNCTOR 1028
Prelude> let cdr = Constant "Dogs rule"
Prelude> getConstant $ fused $ cdr
"Dogs rule"
(Constant a) is* -> * which you need for the Functor , but
now you’re mapping over that 𝑏, and not the 𝑎.
This is a mere cursory check, not a proof that this is a valid
Functor . Most assurances of correctness that programmers use
exist on a gradient and aren’t proper proofs. Despite seeming
a bit pointless, Constant is a lawful Functor .
16.13 More structure, more functors
At times the structure of our types may require that we also
have aFunctor instance for an intermediate type layer. We’ll
demonstrate this using this datatype:
dataWrapf a=
Wrap(f a)
deriving (Eq,Show)
Notice that our 𝑎here is an argument to the 𝑓. So how are
we going to write a Functor instance for this?
instance Functor (Wrapf)where
fmap f ( Wrapfa)=Wrap(f fa)

CHAPTER 16. FUNCTOR 1029
This won’t work because there’s this 𝑓that we’re not hop-
ping over, and 𝑎(the value fmapshould be applying the function
to) is an argument to that 𝑓— the function can’t apply to that
𝑓that is wrapping 𝑎.
instance Functor (Wrapf)where
fmap f ( Wrapfa)=Wrap(fmap f fa)
Here we don’t know what type 𝑓is and it could be anything,
but it needs to be a type that has a Functor instance so that we
canfmapover it. So we add a constraint:
instance Functor f
=>Functor (Wrapf)where
fmap f ( Wrapfa)=Wrap(fmap f fa)
And if we load up the final instance, we can use this wrapper
type:
Prelude> fmap (+1) (Wrap (Just 1))
Wrap (Just 2)
Prelude> fmap (+1) (Wrap [1, 2, 3])
Wrap [2,3,4]
It should work for any Functor . If we pass it something that
isn’t?

CHAPTER 16. FUNCTOR 1030
Prelude> let n = 1 :: Integer
Prelude> fmap (+1) (Wrap n)
Couldn't match expected type ‘f b’
with actual type ‘Integer’
Relevant bindings include
it :: Wrap f b (bound at <interactive>:8:1)
In the first argument of ‘Wrap’, namely ‘n’
In the second argument of ‘fmap’,
namely ‘(Wrap n)’
The number by itself doesn’t oﬀer the additional structure
needs for Wrapto work as a Functor . It’s expecting to be able to
fmapover some 𝑓independent of an 𝑎and this isn’t the case
with any type constant such as Integer .
16.14 IO Functor
We’ve seen the IOtype in the modules and testing chapters
already, but we weren’t doing much with it save to print text
or ask for string input from the user. The IOtype will get
a full chapter of its own later in the book. It is an abstract
datatype; there are no data constructors that you’re permitted
to pattern match on, so the typeclasses IOprovides are the

CHAPTER 16. FUNCTOR 1031
only way you can work with values of type IO a. One of the
simplest provided is Functor .
-- getLine :: IO String
-- read :: Read a => String -> a
getInt::IOInt
getInt=fmap read getLine
Inthas aReadinstance, and fmapliftsreadover the IOtype. A
way you can read getLine here is that it’s not a String , but rather
away to obtain a string .IOdoesn’t guarantee that eﬀects will
be performed, but it does mean that they couldbe performed.
Here the side eﬀect is needing to block and wait for user input
via the standard input stream the OS provides:
Prelude> getInt
10
10
We type 10 and hit enter. GHCi prints IOvalues unless the
type isIO (), in which case it hides the Unitvalue because it’s
meaningless:
Prelude> fmap (const ()) getInt
10

CHAPTER 16. FUNCTOR 1032
The “10” in the GHCi session above is from typing 10 and
hitting enter. GHCi isn’t printing any result after that because
we’re replacing the Intvalue we read from a String. That
information is getting dropped on the floor because we applied
const () to the contents of the IO Int . If we ignore the presence
of IO, it’s as if we did this:
Prelude> let getInt = 10 :: Int
Prelude> const () getInt
()
GHCi as a matter of convenience and design, will not print
any value of type IO ()on the assumption that the IO action
you evaluated was evaluated for eﬀects and because the unit
value cannot communicate anything. We can use the return
function (seen earlier, explained later) to lift a unit value in IO
and reproduce this behavior of GHCi’s:
Prelude> return 1 :: IO Int
1
Prelude> ()
()
Prelude> return () :: IO ()
Prelude>
What if we want to do something more useful? We can fmap
any function we want over IO:

CHAPTER 16. FUNCTOR 1033
Prelude> fmap (+1) getInt
10
11
Prelude> fmap (++ " and me too!") getLine
hello
"hello and me too!"
Wecanalso use dosyntaxtodo whatwe’redoingwith Functor
here:
meTooIsm ::IOString
meTooIsm = do
input<-getLine
return (input ++"and me too!" )
bumpIt::IOInt
bumpIt= do
intVal<-getInt
return (intVal +1)
But iffmap f suffices for what you’re doing, that’s usually
shorter and clearer. It’s perfectly all right and quite common
to start with a more verbose form of some expression and
then clean it up after you’ve got something that works.

CHAPTER 16. FUNCTOR 1034
16.15 What if we want to do something
diﬀerent?
We talked about Functor as a means of lifting functions over
structure so that we may transform only the contents, leaving
the structure alone. What if we wanted to transform only the
structure and leave the type argument to that structure or type
constructor alone? With this, we’ve arrived at natural transfor-
mations . We can attempt to put together a type to express what
we want:
nat::(f->g)->f a->g a
This type is impossible because we can’t have higher-kinded
types as argument types to the function type. What’s the
problem, though? It looks like the type signature for fmap,
doesn’t it? Yet 𝑓and𝑔inf -> g are higher-kinded types. They
must be, because they are the same 𝑓and𝑔that, later in the
type signature, are taking arguments. But in those places they
are applied to their arguments and so have kind *.
So we make a modest change to fix it.
{-# LANGUAGE RankNTypes #-}
typeNatf g=forall a .f a->g a

CHAPTER 16. FUNCTOR 1035
So in a sense, we’re doing the opposite of what a Functor does.
We’re transforming the structure, preserving the values as they
were. We won’t explain it fully here, but the quantification of
𝑎in the right-hand side of the declaration allows us to obligate
all functions of this type to be oblivious to the contents of
the structures 𝑓and𝑔in much the same way that the identity
function cannot do anything but return the argument it was
given.
Syntactically, it lets us avoid talking about 𝑎in the type of
Nat— which is what we want, we shouldn’t haveany specific
information about the contents of 𝑓and𝑔because we’re sup-
posed to be only performing a structural transformation, not
a fold.
If you try to elide the 𝑎from the type arguments without
quantifying it separately, you’ll get an error:
Prelude> type Nat f g = f a -> g a
Not in scope: type variable ‘a’
Wecanaddthequantifier, butifweforgettoturnon RankNTypes
(orRank2Types ), it won’t work:
Prelude> :{
*Main| type Nat f g =
*Main| forall a . f a -> g a
*Main| :}

CHAPTER 16. FUNCTOR 1036
Illegal symbol '.' in type
Perhaps you intended to use RankNTypes or a
similar language extension to enable
explicit-forall syntax:
forall <tvs>. <type>
If we turn on RankNTypes , it works fine:
Prelude> :set -XRank2Types
Prelude> :{
*Main| type Nat f g =
*Main| forall a . f a -> g a
*Main| :}
Prelude>
To see an example of what the quantification prevents, con-
sider the following:

CHAPTER 16. FUNCTOR 1037
typeNatf g=forall a .f a->g a
-- This'll work
maybeToList ::NatMaybe[]
maybeToList Nothing =[]
maybeToList (Justa)=[a]
-- This will not work, not allowed.
degenerateMtl ::NatMaybe[]
degenerateMtl Nothing =[]
degenerateMtl (Justa)=[a+1]
What if we use a version of Natthat mentions 𝑎in the type?

CHAPTER 16. FUNCTOR 1038
moduleBadNatwhere
typeNatf g a=f a->g a
-- This'll work
maybeToList ::NatMaybe[]a
maybeToList Nothing =[]
maybeToList (Justa)=[a]
-- But this will too if we tell it
-- 'a' is Num a => a
degenerateMtl ::Numa=>NatMaybe[]a
degenerateMtl Nothing =[]
degenerateMtl (Justa)=[a+1]
That last example should notwork and is not a good way to
think about natural transformation. Part of software is being
precise and when we talk about natural transformations we’re
saying as much about what we don’twant as we are about what
wedowant. In this case, the invariant we want to preserve is
that the function cannot do anything mischievous with the
values. If you want to transform the values, write a plain old
fold!
We’re going to return to the topic of natural transformations

CHAPTER 16. FUNCTOR 1039
in the next chapter, so cool your jets for now.
16.16 Functors are unique to a datatype
In Haskell, Functor instances will be unique for a given datatype.
We saw that this isn’t true for Monoid; however, we use new-
types to preserve the unique pairing of an instance to a type.
ButFunctor instances will be unique for a datatype, in part
because of parametricity, in part because arguments to type
constructors are applied in order of definition. In a hypothet-
ical not-Haskell language, the following might be possible:
dataTuplea b=
Tuplea b
deriving (Eq,Show)
-- this is impossible in Haskell
instance Functor (Tuple?b)where
fmap f ( Tuplea b)=Tuple(f a) b
There are essentially two ways to address this. One is to flip
the arguments to the type constructor; the other is to make a
new datatype using a Flipnewtype:

CHAPTER 16. FUNCTOR 1040
{-# LANGUAGE FlexibleInstances #-}
moduleFlipFunctor where
dataTuplea b=
Tuplea b
deriving (Eq,Show)
newtype Flipf a b=
Flip(f b a)
deriving (Eq,Show)
-- this works, goofy as it looks.
instance Functor (FlipTuplea)where
fmap f ( Flip(Tuplea b))=
Flip$Tuple(f a) b
Prelude> fmap (+1) (Flip (Tuple 1 "blah"))
Flip (Tuple 2 "blah")
However, Flip Tuple a b is a distinct type from Tuple a b
even if it’s only there to provide for diﬀerent Functor instance
behavior.

CHAPTER 16. FUNCTOR 1041
16.17 Chapter exercises
Determine if a valid Functor can be written for the datatype
provided.
1.dataBool=
False|True
2.dataBoolAndSomethingElse a=
False'a|True'a
3.dataBoolAndMaybeSomethingElse a=
Falsish |Truisha
4.Use the kinds to guide you on this one, don’t get too hung
up on the details.
newtype Muf=InF{ outF::f (Muf) }
5.Again, follow the kinds and ignore the unfamiliar parts
importGHC.Arr
dataD=
D(ArrayWordWord)IntInt
Rearrange the arguments to the type constructor of the
datatype so the Functor instance works.

CHAPTER 16. FUNCTOR 1042
1.dataSuma b=
Firsta
|Secondb
instance Functor (Sume)where
fmap f ( Firsta)=First(f a)
fmap f ( Secondb)=Secondb
2.dataCompany a b c=
DeepBlue a c
|Something b
instance Functor (Company e e')where
fmap f ( Something b)=Something (f b)
fmap_(DeepBlue a c)=DeepBlue a c
3.dataMorea b=
La b a
|Rb a b
deriving (Eq,Show)
instance Functor (Morex)where
fmap f ( La b a') =L(f a) b (f a')
fmap f ( Rb a b') =Rb (f a) b'

CHAPTER 16. FUNCTOR 1043
Keeping in mind that it should result in a Functor that does
the following:
Prelude> fmap (+1) (L 1 2 3)
L 2 2 4
Prelude> fmap (+1) (R 1 2 3)
R 1 3 3
WriteFunctor instances for the following datatypes.
1.dataQuanta b=
Finance
|Deska
|Bloorb
2.No, it’s not interesting by itself.
dataKa b=
Ka

CHAPTER 16. FUNCTOR 1044
3.{-# LANGUAGE FlexibleInstances #-}
newtype Flipf a b=
Flip(f b a)
deriving (Eq,Show)
newtype Ka b=
Ka
-- should remind you of an
-- instance you've written before
instance Functor (FlipKa)where
fmap=undefined
4.dataEvilGoateeConst a b=
GoatyConst b
-- You thought you'd escaped the goats
-- by now didn't you? Nope.
No, it doesn’t do anything interesting. No magic here or
in the previous exercise. If it works, you succeeded.
5.Do you need something extra to make the instance work?

CHAPTER 16. FUNCTOR 1045
dataLiftItOut f a=
LiftItOut (f a)
6.dataParappa f g a=
DaWrappa (f a) (g a)
7.Don’t ask for more typeclass instances than you need. You
can let GHC tell you what to do.
dataIgnoreOne f g a b =
IgnoringSomething (f a) (g b)
8.dataNotorious g o a t =
Notorious (g o) (g a) (g t)
9.You’ll need to use recursion.
dataLista=
Nil
|Consa (Lista)
10.A tree of goats forms a Goat-Lord, fearsome poly-creature.

CHAPTER 16. FUNCTOR 1046
dataGoatLord a=
NoGoat
|OneGoat a
|MoreGoats (GoatLord a)
(GoatLord a)
(GoatLord a)
-- A VERITABLE HYDRA OF GOATS
11.You’ll use an extra functor for this one, although your so-
lution might do it monomorphically without using fmap.
Keep in mind that you will probably not be able to vali-
date this one in the usual manner. Do your best to make
it work.2
dataTalkToMe a=
Halt
|PrintStringa
|Read(String->a)
16.18 Definitions
1.Higher-kinded polymorphism is polymorphism which has
a type variable abstracting over types of a higher kind.
Functor is an example of higher-kinded polymorphism
because the kind of the 𝑓parameter to Functor is* -> *.
2Thanks to Andraz Bajt for inspiring this exercise.

CHAPTER 16. FUNCTOR 1047
Another example of higher-kinded polymorphism would
be a datatype having a parameter to the type constructor
which is of a higher kind, such as the following:
dataWeirdf a=Weird(f a)
Where the kinds of the types involved are:
a:: *
f:: * -> *
Weird::(* -> *)-> * -> *
Here both Weirdand𝑓are higher kinded, with Weirdbeing
an example of higher-kinded polymorphism.
2.Functor is a mapping between categories. In Haskell, this
manifests as a typeclass that generalizes the concept of map:
it takes a function (a -> b) and lifts it into a diﬀerent type.
This conventionally implies some notion of a function
which can be applied to a value with more structure than
the unlifted function was originally designed for. The
additional structure is represented by the use of a higher-
kinded type 𝑓, introduced by the definition of the Functor
typeclass.

CHAPTER 16. FUNCTOR 1048
f ::a->b
-- ``more structure''
fmapf::f a->f b
-- f is applied to a single argument,
-- and so is kind * -> *
One should be careful not to confuse this intuition for
it necessarily being exclusively about containers or data
structures. There’s a Functor of functions and many exotic
types will have a lawful Functor instance.
3.Let’s talk about lifting. Because most of the rest of the
book deals with applicatives and monads of various fla-
vors, we’re going to be lifting a lot, but what do we mean?
When Carnap first described functors in the context of
linguistics, he didn’t really talk about it as lifting anything,
and mathematicians have followed in his footsteps, fo-
cusing on mapping and the production of outputs from
certain types of inputs. Very mathematical of them, and
yet Haskellers use the lifting metaphor often (as we do, in
this book).
There are a couple of ways people commonly think about
it. One is that we can lift a function into a context. Another

CHAPTER 16. FUNCTOR 1049
is that we lift a function over some layer of structure to
apply it. The eﬀect is the same:
Prelude> fmap (+1) $ Just 1
Just 2
Prelude> fmap (+1) [1, 2, 3]
[2,3,4]
In the first case, we lift that function into a Maybecontext
in order to apply it; in the second case, into a list con-
text. It can be helpful to think of it in terms of lifting the
function into the context, because it’s the context we’ve
lifted the function into that determines how the function
will get applied (to one value or, recursively, to many, for
example). The context is the datatype, the definition of
the datatype, and the Functor instance we have for that
datatype. It’s also the contexts that determine what hap-
pens when we try to apply a function to an 𝑎that isn’t
there:
Prelude> fmap (+1) []
[]
Prelude> fmap (+1) Nothing
Nothing
But we often speak more casually about lifting over, as in
fmaplifts a function overa data constructor. This works,

CHAPTER 16. FUNCTOR 1050
too, if you think of the data constructor as a layer of struc-
ture. The function hops over that layer and applies to
what’s inside, if anything.
More precisely, lifting means applying a type constructor
to a type, as in taking an 𝑎type variable and applying an
𝑓type constructor to it to get an f a. Keeping this def-
inition in mind will be helpful. Remember to follow the
typesrather than getting too caught up in the web of a
metaphor.
4.George Clinton is one of the most important innovators of
funk music. Clinton headed up the bands Parliament and
Funkadelic, whose collective style of music is known as
P-Funk; the two bands have fused into a single apotheosis
of booty-shaking rhythm. The Parliament album Mother-
ship Connection is one of the most famous and influential
albums in rock history. Not a Functor , but you can pretend
the album is mapping your consciousness from the real
world into the category of funkiness if that helps.
16.19 Follow-up resources
1.Haskell Wikibook; The Functor class.
https://en.wikibooks.org/wiki/Haskell/The_Functor_class

CHAPTER 16. FUNCTOR 1051
2.Mark P. Jones; A system of constructor classes: overload-
ing and implicit higher-order polymorphism.
3.Gabriel Gonzalez; The functor design pattern.

Chapter 17
Applicative
…the images I most
connect to, historically
speaking, are in black and
white. I see more in black
and white – I like the
abstraction of it.
Mary Ellen Mark
1052

CHAPTER 17. APPLICATIVE 1053
17.1 Applicative
In the previous chapters, we’ve seen two common algebras
that are used as typeclasses. Monoid gives us a means of mashing
two values of the same type together. Functor , on the other
hand, is for function application oversome structure we don’t
want to have to think about. Monoid’s core operation, mappend ,
smashes the structures together — when you mappend two lists,
they become one list, so the structures themselves have been
joined. However, the core operation of Functor ,fmap, applies a
function to a value that is within some structure while leaving
that structure unaltered.
We come now to Applicative . Applicatives are monoidal
functors. No, no, stay with us. The Applicative typeclass allows
for function application lifted over structure (like Functor ). But
withApplicative the function we’re applying is also embed-
ded in some structure. Because the function andthe value
it’s being applied to both have structure, we have to smash
those structures together. So, Applicative involves monoids
and functors. And that’s a pretty powerful thing.
In this chapter, we will:
•define and explore the Applicative typeclass and its core
operations;
•demonstrate why applicatives are monoidal functors;

CHAPTER 17. APPLICATIVE 1054
•make the usual chitchat about laws and instances;
•do a lot of lifting;
•give you some Validation .
17.2 Defining Applicative
The first thing you’re going to notice about this typeclass dec-
laration is that the 𝑓that represents the structure, similar to
Functor , is itself constrained by the Functor typeclass:
classFunctor f=>Applicative fwhere
pure::a->f a
(<*>)::f (a->b)->f a->f b
So, every type that can have an Applicative instance must
also have a Functor instance.
Thepurefunction does a simple and very boring thing:
it lifts something into functorial (applicative) structure. You
can think of this as being a bare minimum bit of structure or
structural identity . Identity for what, you’ll see later when we go
over the laws. The more interesting operation of this typeclass
is<*>. This is an infix function called ‘apply’ or sometimes
‘ap,’ or sometimes ‘tie fighter’ when we’re feeling particularly
zippy.
If we compare the types of <*>andfmap, we see the similar-
ity:

CHAPTER 17. APPLICATIVE 1055
-- fmap
(<$>)::Functor f
=>(a->b)->f a->f b
(<*>)::Applicative f
=>f (a->b)->f a->f b
The diﬀerence is the 𝑓representing functorial structure
that is on the outside of our function in the second definition.
We’ll see good examples of what that means in practice in a
moment.
Along with these core functions, the Control.Applicative li-
brary provides some other convenient functions: liftA,liftA2 ,
andliftA3 :

CHAPTER 17. APPLICATIVE 1056
liftA::Applicative f=>
(a->b)
->f a
->f b
liftA2::Applicative f=>
(a->b->c)
->f a
->f b
->f c
liftA3::Applicative f=>
(a->b->c->d)
->f a
->f b
->f c
->f d
If you’re looking at the type of liftAand thinking, but that’s
fmap, you are correct. It is basically the same as fmaponly with
anApplicative typeclass constraint instead of a Functor one.
Since all applicatives are also functors, though, this is a distinc-
tion without much significance.
Similarly you can see that liftA2 andliftA3 arefmapbut with
functions involving more arguments. It can be a little difficult

CHAPTER 17. APPLICATIVE 1057
to wrap one’s head around how those will work in practice, so
we’ll want to look next at some examples to start developing a
sense of what applicatives can do for us.
17.3 Functor vs. Applicative
We’ve already said that applicatives are monoidal functors,
so what we’ve already learned about Monoid andFunctor is rele-
vant to our understanding of Applicative . We’ve already seen
some examples of what this means in practice, but we want to
develop a stronger intuition for the relationship.
Let’s review the diﬀerence between fmapand<*>:
fmap::(a->b)->f a->f b
(<*>)::f (a->b)->f a->f b
The diﬀerence is we now have an 𝑓in front of our function
(a -> b) . The increase in power it introduces is profound. For
one thing, any Applicative also has a Functor and not merely
by definition — you can define a Functor in terms of a pro-
videdApplicative instance. Proving it is outside the scope of
the current book, but this follows from the laws of Functor
andApplicative (we’ll get to the applicative laws later in this
chapter):
fmapf x=pure f<*>x

CHAPTER 17. APPLICATIVE 1058
How might we demonstrate this? You’ll need to import
Control.Applicative if you’re using GHC 7.8 or older to test this
example:
Prelude> fmap (+1) [1, 2, 3]
[2,3,4]
Prelude> pure (+1) <*> [1..3]
[2,3,4]
Keeping in mind that pure has type Applicative f => a -> f
a, we can think of it as a means of embedding a value of any
type in the structure we’re working with:
Prelude> pure 1 :: [Int]
[1]
Prelude> pure 1 :: Maybe Int
Just 1
Prelude> pure 1 :: Either a Int
Right 1
Prelude> pure 1 :: ([a], Int)
([],1)
The left type is handled diﬀerently from the right in the
final two examples for the same reason as here:
Prelude> fmap (+1) (4, 5)
(4,6)

CHAPTER 17. APPLICATIVE 1059
The left type is part of the structure, and the structure is
not transformed by the function application.
17.4 Applicative functors are monoidal
functors
First let us notice something:
($)::(a->b)->a->b
(<$>)::(a->b)->f a->f b
(<*>)::f (a->b)->f a->f b
We already know $to be something of a do-nothing infix
function which exists to give the right-hand side more prece-
dence and thus avoid parentheses. For our present purposes
it acts as a nice proxy for ordinary function application in its
type.
When we get to <$>, the alias for fmap, we notice the first
change is that we’re now lifting our (a -> b) over the 𝑓wrapped
around our value and applying the function to that value.
Then as we arrive at apor<*>, theApplicative apply method,
our function is now also embedded in the functorial structure.
Now we get to the monoidal in “monoidal functor”:

CHAPTER 17. APPLICATIVE 1060
::f (a->b)->f a->f b
-- The two arguments to our function are:
f(a->b)
-- and
fa
If we imagine that we can apply (a -> b) to𝑎and get 𝑏,
ignoring the functorial structure, we still have a problem as we
need to return f b. When we were dealing with fmap, we had
only one bit of structure, so it was left unchanged. Now we
have two bits of structure of type 𝑓that we need to deal with
somehow before returning a value of type f b. We can’t simply
leave them unchanged; we must unite them somehow. Now,
they will be definitely the same type, because the 𝑓must be
the same type throughout. In fact, if we separate the structure
parts from the function parts, maybe we’ll see what we need:
::f (a->b)->f a->f b
f f f
(a->b) a b
Didn’t we have something earlier that can take two values
of one type and return one value of the same type? Provided

CHAPTER 17. APPLICATIVE 1061
the𝑓is a type with a Monoid instance, then we have a good way
to make them play nice together:
mappend ::Monoida=>a->a->a
So, with Applicative , we have a Monoid for our structure and
function application for our values!
mappend ::f f f
$ :: (a->b) a b
(<*>)::f (a->b)->f a->f b
-- plus Functor fmap to be able to map
-- over the f to begin with.
So in a sense, we’re bolting a Monoid onto aFunctor to be able
to deal with functions embedded in additional structure. In
another sense, we’re enriching function application with the
very structure we were previously mapping over with Functor .
Let’s consider a few familiar examples to examine what this
means:

CHAPTER 17. APPLICATIVE 1062
-- List
[(*2), (*3)]<*>[4,5]
=
[2*4,2*5,3*4,3*5]
-- reduced
[8,10,12,15]
So what was (a -> b) enriched with in f (a -> b) -> f a ->
f b? In this case, “list-ness”. Although the actual application
of each (a -> b) to a value of type 𝑎is quite ordinary, we now
have a list of functions rather than a single function as would
be the case if it was the list Functor .
But lists aren’t the only structure we can enrich our func-
tions with — not even close! The structure bit can also be Maybe:

CHAPTER 17. APPLICATIVE 1063
Just(*2)<*>Just2
=
Just4
Just(*2)<*>Nothing
=
Nothing
Nothing <*>Just2
=
Nothing
Nothing <*>Nothing
=
Nothing
WithMaybe, the ordinary functor is mapping over the pos-
sibility of a value’s nonexistence. With the Applicative , now
the function also might not be provided. We’ll see a couple
of nice, long examples of how this might happen — how you
could end up not even providing a function to apply — in a
bit, not just with Maybe, but with Either and a new type called
Validation as well.

CHAPTER 17. APPLICATIVE 1064
Show me the monoids
Recall that the Functor instance for the two-tuple ignores the
first value inside the tuple:
Prelude> fmap (+1) ("blah", 0)
("blah",1)
Butthe Applicative forthetwo-tupledemonstratesthemonoid
inApplicative nicely for us. In fact, if you call :infoon(,)in
your REPL you’ll notice something:
Prelude> :info (,)
data (,) a b = (,) a b
-- Defined in ‘GHC.Tuple’
...
instance Monoid a
=> Applicative ((,) a)
-- Defined in ‘GHC.Base’
...
instance (Monoid a, Monoid b)
=> Monoid (a, b)
For the Applicative instance of two-tuple, we don’t need a
Monoid for the 𝑏because we’re using function application to
produce the 𝑏. However, for the first value in the tuple, we
still need the Monoid because we have two values and need to
somehow turn that into one value of the same type:

CHAPTER 17. APPLICATIVE 1065
Prelude> ("Woo", (+1)) <*> (" Hoo!", 0)
("Woo Hoo!", 1)
Notice that for the 𝑎value, we didn’t apply any function,
but they have combined themselves as if by magic; that’s due
to theMonoid instance for the 𝑎values. The function in the 𝑏
position of the left tuple has been applied to the value in the 𝑏
position of the right tuple to produce a result. That function
application is why we don’t need a Monoid instance on the 𝑏.
Let’s look at more such examples. Pay careful attention to
how the 𝑎values in the tuples are combined:
Prelude> import Data.Monoid
Prelude> (Sum 2, (+1)) <*> (Sum 0, 0)
(Sum {getSum = 2},1)
Prelude> (Product 3, (+9))<*>(Product 2, 8)
(Product {getProduct = 6},17)
Prelude> (All True, (+1))<*>(All False, 0)
(All {getAll = False},1)
It doesn’t really matter whatMonoid , but we need some way
of combining or choosing our values.
Tuple Monoid and Applicative side by side
Squint if you can’t see it.

CHAPTER 17. APPLICATIVE 1066
instance (Monoida,Monoidb)
=>Monoid(a,b)where
mempty=(mempty, mempty)
(a, b) `mappend` (a',b') =
(a `mappend` a', b `mappend` b')
instance Monoida
=>Applicative ((,) a) where
pure x=(mempty, x)
(u, f)<*>(v, x)=
(u `mappend` v, f x)
Maybe Monoid and Applicative
While applicatives are monoidal functors, be careful about
making assumptions based on this. For one thing, Monoid and
Applicative instances aren’t required or guaranteed to have the
same monoid of structure, and the functorial part may change
the way it behaves. Nevertheless, you might be able to see the
implicit monoid in how the Applicative pattern matches on
theJustandNothing cases and compare that with this Monoid :

CHAPTER 17. APPLICATIVE 1067
instance Monoida=>Monoid(Maybea)where
mempty=Nothing
mappend m Nothing =m
mappend Nothing m=m
mappend ( Justa) (Justa')=
Just(mappend a a')
instance Applicative Maybewhere
pure=Just
Nothing <*> _ = Nothing
_ <*>Nothing =Nothing
Justf<*>Justa=Just(f a)
Later we’ll see some examples of how diﬀerent Monoid in-
stances can give diﬀerent results for applicatives. For now,
recognize that the monoidal bit may not be what you recog-
nize as the canonical mappend of that type, because some types
can have multiple monoids.
17.5 Applicative in use
Bynowitshouldcomeasnosurprisethatmanyofthedatatypes
we’ve been working with in the past two chapters also have
Applicative instances. Since we are already so familiar with list
andMaybe, those examples will be a good place to start. Later

CHAPTER 17. APPLICATIVE 1068
in the chapter, we will be introducing some new types, so just
hang onto your hats.
List Applicative
We’ll start with the list Applicative because it’s a clear way to
get a sense of the pattern. Let’s start by specializing the types:
-- f ~ []
(<*>)::f (a->b)->f a->f b
(<*>)::[ ] (a->b)->[ ] a->[ ] b
-- more syntactically typical
(<*>)::[(a->b)]->[a]->[b]
pure::a->f a
pure::a->[ ] a
Or, again, if you have GHC 8 or newer, you can do this:
Prelude> :set -XTypeApplications
Prelude> :type (<*>) @[]
(<*>) @[] :: [a -> b] -> [a] -> [b]
Prelude> :type pure @[]
pure @[] :: a -> [a]

CHAPTER 17. APPLICATIVE 1069
What’s the List applicative do?
Previously with list Functor , we were mapping a single function
over a plurality of values:
Prelude> fmap (2^) [1, 2, 3]
[2,4,8]
Prelude> fmap (^2) [1, 2, 3]
[1,4,9]
With the list Applicative , we are mapping a plurality of func-
tions over a plurality of values:
Prelude> [(+1), (*2)] <*> [2, 4]
[3,5,4,8]
We can see how this makes sense given that:
(<*>)::Applicative f
=>f (a->b)->f a->f b
f~[]
listApply ::[(a->b)]->[a]->[b]
listFmap ::(a->b)->[a]->[b]

CHAPTER 17. APPLICATIVE 1070
The𝑓structure that is wrapped around our function in the
listApply function is itself a list. Therefore, our a -> b from
Functor has become a listofa -> b .
Now what happened with that expression we tested? Some-
thing like this:
[(+1), (*2)]<*>[2,4]==[3,5,4,8]
[3,5,4,8]
-- [1] [2] [3] [4]
1.The first item in the list, 3, is the result of (+1) being applied
to 2.
2.5 is the result of applying (+1) to 4.
3.4 is the result of applying (*2) to 2.
4.8 is the result of applying (*2) to 4.
More visually:
[(+1), (*2)]<*>[2,4]
[ (+1)2, (+1)4, (*2)2, (*2)4]
It maps each function value from the first list over the sec-
ond list, applies the operations, and returns one list. The fact

CHAPTER 17. APPLICATIVE 1071
that it doesn’t return two lists or a nested list or some other
configuration in which both structures are preserved is the
monoidal part; the reason we don’t have a list of functions
concatenated with a list of values is the function application
part.
We can see this relationship more readily if we use the
tuple constructor with the list Applicative . We’ll use the infix
operator for fmapto map the tuple constructor over the first list.
This embeds an unapplied function (the tuple data constructor
in this case) into some structure (a list, in this case), and returns
a list of partially applied functions. The (infix) applicative will
then apply one list of operations to the second list, monoidally
appending the two lists:
Prelude> (,) <$> [1, 2] <*> [3, 4]
[(1,3),(1,4),(2,3),(2,4)]
You might think of it this way:
Prelude> (,) <$> [1, 2] <*> [3, 4]
-- fmap the (,) over the first list
[(1, ), (2, )] <*> [3, 4]
-- then we apply the first list
-- to the second
[(1,3),(1,4),(2,3),(2,4)]

CHAPTER 17. APPLICATIVE 1072
TheliftA2 function gives us another way to write this, too:
Prelude> liftA2 (,) [1, 2] [3, 4]
[(1,3),(1,4),(2,3),(2,4)]
Let’s look at a few more examples of the same pattern:
Prelude> (+) <$> [1, 2] <*> [3, 5]
[4,6,5,7]
Prelude> liftA2 (+) [1, 2] [3, 5]
[4,6,5,7]
Prelude> max <$> [1, 2] <*> [1, 4]
[1,4,2,4]
Prelude> liftA2 max [1, 2] [1, 4]
[1,4,2,4]
If you’re familiar with Cartesian products1, this probably
looks a lot like one, but with functions.
We’re going to run through some more examples, to give
you a little more context for when these functions can become
useful. Thefollowingexampleswilluseafunctioncalled lookup
that we’ll briefly demonstrate:
Prelude> :t lookup
lookup :: Eq a => a -> [(a, b)] -> Maybe b
1The Cartesian product is the product of two sets that results in all the ordered pairs
(tuples) of the elements of those sets.

CHAPTER 17. APPLICATIVE 1073
Prelude> let l = lookup 3 [(3, "hello")]
Prelude> l
Just "hello"
Prelude> fmap length $ l
Just 5
Prelude> let c (x:xs) = toUpper x:xs
Prelude> fmap c $ l
Just "Hello"
So,lookup searches inside a list of tuples for a value that
matches the input and returns the paired value wrapped inside
aMaybecontext.
It’s worth pointing out here that if you’re working with
Mapdata structures instead of lists of tuples, you can import
Data.Map and use a Mapversion of lookup along with fromList to
accomplish the same thing with that data structure:
Prelude> let m = fromList [(3, "hello")]
Prelude> fmap c $ Data.Map.lookup 3 m
Just "Hello"
That may seem trivial at the moment, but Mapis a frequently
used data structure, so it’s worth mentioning.
Nowthatwehavevalueswrappedina Maybecontext, perhaps
we’d like to apply some functions to them. This is where we
want applicative operations. Although it’s more likely that
we’d have functions fetching data from somewhere else rather

CHAPTER 17. APPLICATIVE 1074
than having it all listed in our code file, we’ll go ahead and
define some values in a source file for convenience:
importControl.Applicative
fx=
lookup x [ ( 3,"hello")
, (4,"julie")
, (5,"kbai")]
gy=
lookup y [ ( 7,"sup?")
, (8,"chris")
, (9,"aloha")]
hz=
lookup z [( 2,3), (5,6), (7,8)]
mx=
lookup x [( 4,10), (8,13), (1,9001)]
Now we want to look things up and add them together. We’ll
start with some operations over these data:
Prelude> f 3
Just "hello"
Prelude> g 8
Just "chris"

CHAPTER 17. APPLICATIVE 1075
Prelude> (++) <$> f 3 <*> g 7
Just "hellosup?"
Prelude> (+) <$> h 5 <*> m 1
Just 9007
Prelude> (+) <$> h 5 <*> m 6
Nothing
So we first fmapthose functions over the value inside the first
Maybecontext, if it’s a Justvalue, making it a partially applied
function wrapped in a Maybecontext. Then we use the tie-
fighter to apply that to the second value, again wrapped in a
Maybe. If either value is a Nothing , we get Nothing .
We can again do the same thing with liftA2 :
Prelude> liftA2 (++) (g 9) (f 4)
Just "alohajulie"
Prelude> liftA2 (^) (h 5) (m 4)
Just 60466176
Prelude> liftA2 (*) (h 5) (m 4)
Just 60
Prelude> liftA2 (*) (h 1) (m 1)
Nothing
Your applicative context can also sometimes be IO:
(++)<$>getLine <*>getLine
(,)<$>getLine <*>getLine

CHAPTER 17. APPLICATIVE 1076
Try it. Now try using fmapto get the length of the resulting
string of the first example.
Exercises: Lookups
In the following exercises you will need to use the following
terms to make the expressions typecheck:
1.pure
2.(<$>)
-- or fmap
3.(<*>)
Make the following expressions typecheck.
1.added::MaybeInteger
added=
(+3) (lookup 3$zip [1,2,3] [4,5,6])
2.y::MaybeInteger
y=lookup3$zip [1,2,3] [4,5,6]
z::MaybeInteger
z=lookup2$zip [1,2,3] [4,5,6]
tupled::Maybe(Integer,Integer)
tupled=(,) y z

CHAPTER 17. APPLICATIVE 1077
3.importData.List (elemIndex )
x::MaybeInt
x=elemIndex 3[1,2,3,4,5]
y::MaybeInt
y=elemIndex 4[1,2,3,4,5]
max'::Int->Int->Int
max'=max
maxed::MaybeInt
maxed=max' x y
4.xs=[1,2,3]
ys=[4,5,6]
x::MaybeInteger
x=lookup3$zip xs ys
y::MaybeInteger
y=lookup2$zip xs ys
summed::MaybeInteger
summed=sum$(,) x y

CHAPTER 17. APPLICATIVE 1078
Identity
TheIdentity type here is a way to introduce structure without
changing the semantics of what you’re doing. We’ll see it used
with these typeclasses that involve function application around
and over structure, but this type itself isn’t very interesting, as
it has no semantic flavor.
Specializing the types
Here is what the type will look like when our structure is
Identity :
-- f ~ Identity
-- Applicative f =>
typeId=Identity
(<*>)::f (a->b)->f a->f b
(<*>)::Id(a->b)->Ida->Idb
pure::a->f a
pure::a->Ida
Why would we use Identity just to introduce some struc-
ture? What is the meaning of all this?
Prelude> let xs = [1, 2, 3]

CHAPTER 17. APPLICATIVE 1079
Prelude> let xs' = [9, 9, 9]
Prelude> const <$> xs <*> xs'
[1,1,1,2,2,2,3,3,3]
Prelude> let mkId = Identity
Prelude> const <$> mkId xs <*> mkId xs'
Identity [1,2,3]
Having this extra bit of structure around our values lifts the
constfunction, from mapping over the lists to mapping over
theIdentity . We have to go over an 𝑓structure to apply the
function to the values inside. If our 𝑓is the list, constapplies to
the values inside the list, as we saw above. If the 𝑓isIdentity ,
thenconsttreats the lists inside the Identity structure as single
values, not structure containing values.
Exercise: Identity Instance
Write an Applicative instance for Identity .

CHAPTER 17. APPLICATIVE 1080
newtype Identity a=Identity a
deriving (Eq,Ord,Show)
instance Functor Identity where
fmap=undefined
instance Applicative Identity where
pure=undefined
(<*>)=undefined
Constant
This is not so diﬀerent from the Identity type, except this
not only provides structure it also acts like the constfunction.
It sort of throws away a function application. If this seems
confusing, it’s because it is. However, it is also something that,
likeIdentity has real use cases, and you will see it in other
people’s code. It can be difficult to get used to using it yourself,
but we keep trying.
This datatype is like the constfunction in that it takes two
arguments but one of them gets discarded. In the case of the
datatype, we have to map our function over the argument
that gets discarded. So there is no value to map over, and the
function application doesn’t happen.

CHAPTER 17. APPLICATIVE 1081
Specializing the types
All right, so here’s what the types will look like:
-- f ~ Constant e
typeC=Constant
(<*>)::f (a->b)->f a->f b
(<*>)::Ce (a->b)->Ce a->Ce b
pure::a->f a
pure::a->Ce a
And here are some examples of how it works. These are,
yes, a bit contrived, but showing you real code with this in it
would probably make it much harder for you to see what’s
going on:
Prelude> let f = Constant (Sum 1)
Prelude> let g = Constant (Sum 2)
Prelude> f <*> g
Constant {getConstant = Sum {getSum = 3}
Prelude> Constant undefined <*> g
Constant (Sum {getSum =
*** Exception: Prelude.undefined
Prelude> pure 1
1

CHAPTER 17. APPLICATIVE 1082
Prelude> pure 1 :: Constant String Int
Constant {getConstant = ""}
It can’t do anything because it can only hold onto the one
value. The function doesn’t exist, and the 𝑏is a ghost. So you
use this when whatever you want to do involves throwing away
a function application. We know it seems somewhat crazy, but
we promise there are really times real coders do this in real
code. Pinky swear.
Exercise: Constant Instance
Write an Applicative instance for Constant .
newtype Constant a b=
Constant { getConstant ::a }
deriving (Eq,Ord,Show)
instance Functor (Constant a)where
fmap=undefined
instance Monoida
=>Applicative (Constant a)where
pure=undefined
(<*>)=undefined

CHAPTER 17. APPLICATIVE 1083
Maybe Applicative
WithMaybe, we’re doing something a bit diﬀerent from above.
We saw previously how to use fmapwithMaybe, but here our
function is also embedded in a Maybestructure. Therefore,
when𝑓isMaybe, we’re saying the function itself might not exist,
because we’re allowing the possibility of the function to be
applied being a Nothing case.
Specializing the types
Here’s what the type looks like when we’re using Maybeas our
𝑓structure:
-- f ~ Maybe
typeM=Maybe
(<*>)::f (a->b)->f a->f b
(<*>)::M(a->b)->Ma->Mb
pure::a->f a
pure::a->Ma
Are you ready to validate some persons? Yes. Yes, you are.

CHAPTER 17. APPLICATIVE 1084
Using the Maybe Applicative
Consider the following example where we validate our inputs
to create a value of type Maybe Person , where the Maybeis because
our inputs might be invalid:
validateLength ::Int
->String
->MaybeString
validateLength maxLen s =
if(length s) >maxLen
thenNothing
elseJusts
newtype Name=
NameStringderiving (Eq,Show)
newtype Address =
Address Stringderiving (Eq,Show)
mkName::String->MaybeName
mkNames=
fmapName$validateLength 25s
mkAddress ::String->MaybeAddress
mkAddress a=
fmapAddress $validateLength 100a

CHAPTER 17. APPLICATIVE 1085
Now we’ll make a smart constructor for a Person :
dataPerson=
PersonNameAddress
deriving (Eq,Show)
mkPerson ::String
->String
->MaybePerson
mkPerson n a=
casemkName n of
Nothing ->Nothing
Justn'->
casemkAddress a of
Nothing ->Nothing
Justa'->
Just$Personn' a'
The problem here is while we’ve successfully leveraged fmap
fromFunctor in the simpler cases of mkName andmkAddress , we
can’t really make that work here with mkPerson . Let’s investigate
why:
Prelude> :t fmap Person (mkName "Babe")
fmap Person (mkName "Babe")
:: Maybe (Address -> Person)

CHAPTER 17. APPLICATIVE 1086
This has worked so far for the first argument to the Person
constructor that we’re validating, but we’ve hit a roadblock.
Can you see the problem?
Prelude> :{
*Main| fmap (fmap Person (mkName "Babe"))
*Main| (mkAddress "old macdonald's")
*Main| :}
Couldn't match expected type ‘Address -> b’
with actual type
‘Maybe (Address -> Person)’
Possible cause: ‘fmap’ is applied to too
many arguments
In the first argument of ‘fmap’, namely
‘(fmap Person (mkName "Babe"))’
In the expression:
fmap (fmap Person (mkName "Babe")) v
The problem is that our (a -> b) is now hiding inside Maybe.
Let’s look at the type of fmapagain:
fmap :: Functor f => (a -> b) -> f a -> f b
Maybeis definitely a Functor , but that’s not really going to
help us here. We need to be able to map a function embedded
in our𝑓.Applicative gives us what we need here!

CHAPTER 17. APPLICATIVE 1087
(<*>)::Applicative f
=>f (a->b)->f a->f b
Now let’s see if we can wield this new toy:
Prelude> let s = "old macdonald's"
Prelude> let addy = mkAddress s
Prelude> let b = mkName "Babe"
Prelude> let person = fmap Person b
Prelude> person <*> addy
Just (Person (Name "Babe")
(Address "old macdonald's"))
Nice, right? A little ugly though. Using the infix alias for
fmapcalled<$>cleans it up a bit, at least to Haskellers’ eyes:
Prelude> Person <$> mkName "Babe" <*> addy
Just (Person (Name "Babe")
(Address "old macdonald's"))
We still use fmap(via<$>) here for the first lifting over Maybe;
after that our (a -> b) is hiding in the 𝑓where𝑓=Maybe, so we
have to start using Applicative to keep mapping over that.
We can now use a much shorter definition of mkPerson !

CHAPTER 17. APPLICATIVE 1088
mkPerson ::String
->String
->MaybePerson
mkPerson n a=
Person<$>mkName n <*>mkAddress a
As an additional bonus, this is now far less annoying to
extend if we added new fields as well.
Breaking down that example
We’re going to give the Functor andApplicative instances for
Maybethe same treatment we gave folds. This will be a bit long.
It is possible that some of this will seem like too much detail;
read it to whatever depth you feel you need to. It will sit here,
patiently waiting to see if you ever need to come back and
read it more closely.
Maybe Functor and the Name constructor

CHAPTER 17. APPLICATIVE 1089
instance Functor Maybewhere
fmap_Nothing =Nothing
fmap f ( Justa) =Just(f a)
instance Applicative Maybewhere
pure=Just
Nothing <*> _ = Nothing
_ <*>Nothing =Nothing
Justf<*>Justa=Just(f a)
TheApplicative instance is not exactly the same as the in-
stance in base, but that’s for simplification. For your purposes,
it produces the same results.
First the function and datatype definitions for our functor
write-up for how we’re using the validateLength function with
NameandAddress :

CHAPTER 17. APPLICATIVE 1090
validateLength ::Int
->String
->MaybeString
validateLength maxLen s =
if(length s) >maxLen
thenNothing
elseJusts
newtype Name=
NameStringderiving (Eq,Show)
newtype Address =
Address Stringderiving (Eq,Show)
mkName::String->MaybeName
mkNames=fmapName$validateLength 25s
mkAddress ::String->MaybeAddress
mkAddress a=
fmapAddress $validateLength 100a
Now we’re going to start filling in the definitions and ex-
panding them equationally like we did in the chapter on folds.
First we apply mkName to the value "babe" so that𝑠is bound
to that string:

CHAPTER 17. APPLICATIVE 1091
mkNames=
fmapName$validateLength 25s
mkName"babe"=
fmapName$validateLength 25"babe"
Now we need to figure out what validateLength is about since
that has to be evaluated before we know what fmapis mapping
over. Here we’re applying it to 25 and ”babe”, evaluating the
length of the string ”babe”, and then determining which branch
in the if-then-else wins:
validateLength ::Int
->String
->MaybeString
validateLength 25"babe"=
if(length "babe")>25
thenNothing
elseJust"babe"
if4>25
thenNothing
elseJust"babe"
-- 4 isn't greater than 25, so:
validateLength 25"babe"=
Just"babe"

CHAPTER 17. APPLICATIVE 1092
Now we’re going to replace validateLength applied to 25 and
”babe” with what it evaluated to, then figure out what the fmap
NameoverJust "babe" business is about:
mkName"babe"=
fmapName$Just"babe"
fmapName$Just"babe"
Keeping in mind the type of fmapfromFunctor , we see the
data constructor Nameis the function (a -> b) we’re mapping
over some functorial 𝑓. In this case, 𝑓isMaybe. The𝑎in𝑓 𝑎is
String :
(a->b)->f a->f b
:tName ::(String->Name)
:tJust"babe"::MaybeString
typeM=Maybe
(a->b)->f a ->f b
(String->Name)->MString->MName
Since we know we’re dealing with the Functor instance for
Maybe, we can inline thatfunction’s definition too!

CHAPTER 17. APPLICATIVE 1093
fmap_Nothing =Nothing
fmapf (Justa) =Just(f a)
-- We have (Just "babe") so
-- skipping Nothing case
-- fmap _ Nothing = Nothing
fmapf (Justa) =
Just(f a)
fmapName(Just"babe")=
Just(Name"babe")
mkName"babe"=fmapName$Just"babe"
mkName"babe"=Just(Name"babe")
-- f b
Maybe Applicative and Person
dataPerson=
PersonNameAddress
deriving (Eq,Show)
First we’ll be using the Functor to map the Person data con-
structor over the Maybe Name value. Unlike NameandAddress ,
Person takes two arguments rather than one.

CHAPTER 17. APPLICATIVE 1094
Person
<$>Just(Name"babe")
<*>Just(Address "farm")
fmapPerson(Just(Name"babe"))
:tPerson::Name->Address ->Person
:tJust(Name"babe")::MaybeName
(a->b)->f a->f b
(Name->Address ->Person)
a->b
->MaybeName->Maybe(Address ->Person)
f a f b

CHAPTER 17. APPLICATIVE 1095
fmap_Nothing =Nothing
fmapf (Justa) =Just(f a)
fmapPerson(Just(Name"babe"))
f::Person
a::Name"babe"
-- We skip this pattern match
-- because we have Just
-- fmap _ Nothing = Nothing
fmapf ( Justa) =
Just(f a)
fmapPerson(Just(Name"babe"))=
Just(Person(Name"babe"))
The problem is Person (Name "babe") is awaiting another ar-
gument, the address, so it’s a partially applied function. That’s
our(a -> b) in the type of Applicative ’s(<*>). The𝑓wrapping
our(a -> b) is theMaybewhich results from us possibly not hav-
ing had an 𝑎to map over to begin with, resulting in a Nothing
value:

CHAPTER 17. APPLICATIVE 1096
-- Person is awaiting another argument
:tJust(Person(Name"babe"))
::Maybe(Address ->Person)
:tJust(Address "farm")::MaybeAddress
-- We want to apply the partially
-- applied (Person "babe") inside the
-- 'Just' to the "farm" inside the Just.
Just(Person(Name"babe"))
<*>Just(Address "farm")
So, since the function we want to map is inside the same
structure as the value we want to apply it to, we need the
Applicative (<*>) . In the following, we remind you of what the
type looks like and how the type specializes to this application:
f(a->b)->f a->f b
typeM=Maybe
typeAddy=Address
M(Addy->Person)->MAddy->MPerson
f( a->b )->f a->f b

CHAPTER 17. APPLICATIVE 1097
We know we’re using the Maybe Applicative , so we can go
ahead and inline the definition. Reminder that this version of
theApplicative instance is simplified from the one in GHC, so
please don’t email us to tell us our instance is wrong:
instance Applicative Maybewhere
pure=Just
Nothing <*> _ = Nothing
_ <*>Nothing =Nothing
Justf<*>Justa=Just(f a)
We know we can ignore the Nothing cases because our func-
tion isJust, our value is Just...and our cause is just! Just…kid-
ding.
If we fill in our partially applied Person constructor for 𝑓,
and our Address value for 𝑎, it’s not too hard to see how the
final result fits.

CHAPTER 17. APPLICATIVE 1098
-- Neither function nor value are Nothing,
-- so we skip these two cases
-- Nothing <*> _ = Nothing
-- _ <*> Nothing = Nothing
Justf<*>Justa=Just(f a)
Just(Person(Name"babe"))
<*>Just(Address "farm")=
Just(Person(Name"babe")
(Address "farm"))
Before we moooove on
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

CHAPTER 17. APPLICATIVE 1099
-- Validating to get rid of empty
-- strings, negative numbers
cowFromString ::String
->Int
->Int
->MaybeCow
cowFromString name' age' weight' =
casenoEmpty name' of
Nothing ->Nothing
Justnammy->
casenoNegative age' of
Nothing ->Nothing
Justagey->
casenoNegative weight' of
Nothing ->Nothing
Justweighty ->
Just(Cownammy agey weighty)
cowFromString is…bad. You can probably tell. But by the use
of Applicative, it can be improved!

CHAPTER 17. APPLICATIVE 1100
-- you'll need to import this if
-- you have GHC <7.10
importControl.Applicative
cowFromString' ::String
->Int
->Int
->MaybeCow
cowFromString' name' age' weight' =
Cow<$>noEmpty name'
<*>noNegative age'
<*>noNegative weight'
Or if we want other Haskellers to think we’re really cool
and hip:
cowFromString'' ::String
->Int
->Int
->MaybeCow
cowFromString'' name' age' weight' =
liftA3Cow(noEmpty name')
(noNegative age')
(noNegative weight')

CHAPTER 17. APPLICATIVE 1101
So, we’re taking advantage of the Maybe Applicative here.
What does that look like? First we’ll use the infix syntax for
fmap,<$>, and apply <*>:
Prelude> let cow1 = Cow <$> noEmpty "Bess"
Prelude> :t cow1
cow1 :: Maybe (Int -> Int -> Cow)
Prelude> let cow2 = cow1 <*> noNegative 1
Prelude> :t cow2
cow2 :: Maybe (Int -> Cow)
Prelude> let cow3 = cow2 <*> noNegative 2
Prelude> :t cow3
cow3 :: Maybe Cow
Then with liftA3 :
Prelude> let cow1 = liftA3 Cow
Prelude> :t cow1
cow1 :: Applicative f
=> f String -> f Int -> f Int -> f Cow

CHAPTER 17. APPLICATIVE 1102
Prelude> let cow2 = cow1 (noEmpty "blah")
Prelude> :t cow2
cow2 :: Maybe Int -> Maybe Int -> Maybe Cow
Prelude> let cow3 = cow2 (noNegative 1)
Prelude> :t cow3
cow3 :: Maybe Int -> Maybe Cow
Prelude> let cow4 = cow3 (noNegative 2)
Prelude> :t cow4
cow4 :: Maybe Cow
So, from a simplified point of view, Applicative is really just
a way of saying:

CHAPTER 17. APPLICATIVE 1103
-- we fmap'd my function over some
-- functorial ``f'' or it already
-- was in ``f'' somehow
-- f ~ Maybe
cow1::Maybe(Int->Int->Cow)
cow1=fmapCow(noEmpty "Bess")
-- and we hit a situation where want to map
-- f (a -> b)
-- not just (a -> b)
(<*>)::Applicative f
=>f (a->b)->f a->f b
-- over some f a
-- to get an f b
cow2::Maybe(Int->Cow)
cow2=cow1<*>noNegative 1
As a result, you may be able to imagine yourself saying, “I
want to do something kinda like an fmap, but my function is
embedded in the functorial structure too, not only the value I
want to apply my function to.” This is a basic motivation for
Applicative .
With the Applicative instance for Maybe, what we’re doing is

CHAPTER 17. APPLICATIVE 1104
enriching functorial application with the additional proviso
that, “I may not have a function at all.”
We can see this in the following specialization of the apply
function (<*>):
(<*>)::Applicative f
=>f (a->b)->f a->f b
f~Maybe
typeM=Maybe
maybeApply ::M(a->b)->Ma->Mb
maybeFmap ::(a->b)->Ma->Mb
-- maybeFmap is just fmap's type
-- specialized to Maybe
You can test these specializations (more concrete versions)
of the types:

CHAPTER 17. APPLICATIVE 1105
maybeApply ::Maybe(a->b)
->Maybea
->Maybeb
maybeApply =(<*>)
maybeMap ::(a->b)
->Maybea
->Maybeb
maybeMap =fmap
If you make any mistakes, the compiler will let you know:
maybeMapBad ::(a->b)
->Maybea
->f b
maybeMapBad =fmap
Couldn't match type ‘f1’ with ‘Maybe’
‘f1’ is a rigid type variable bound by
an expression type signature:
(a1 -> b1) -> Maybe a1 -> f1 b1
Exercise: Fixer Upper
Given the function and values provided, use (<$>)fromFunctor ,
(<*>)andpurefrom the Applicative typeclass to fill in missing
bits of the broken code to make it work.

CHAPTER 17. APPLICATIVE 1106
1.const<$>Just"Hello" <*>"World"
2.(,,,)Just90
<*>Just10Just"Tierness" [1,2,3]
17.6 Applicative laws
After examining the law, test each of the expressions in the
REPL.
1.Identity
Here is the definition of the identity law:
pureid<*>v=v
To see examples of this law, evaluate these expressions.

CHAPTER 17. APPLICATIVE 1107
pureid<*>[1..5]
pureid<*>Just"Hello Applicative"
pureid<*>Nothing
pureid<*>Left"Error'ish"
pureid<*>Right8001
-- ((->) a) has an instance
pureid<*>(+1)$2
As you may recall, Functor has a similar identity law, and
comparing them directly might help you see what’s hap-
pening:
id[1..5]
fmapid [1..5]
pureid<*>[1..5]
The identity law states that all three of those should be
equal. You can test them for equality in your REPL or you
could write a simple test to get the answer. So, what’s pure

CHAPTER 17. APPLICATIVE 1108
doing for us? It’s embedding our idfunction into some
structure so that we can use applyinstead of fmap.
2.Composition
Here is the definition of the composition law for applica-
tives:
pure(.)<*>u<*>v<*>w=
u<*>(v<*>w)
You may find the syntax a bit unusual and difficult to
read here. This is similar to the law of composition for
Functor . It is the law stating that the result of composing
our functions first and then applying them and the re-
sult of applying the functions first then composing them
should be the same. We’re using the composition opera-
tor as a prefix instead of the more usual infix, and using
purein order to embed that operator into the appropriate
structure so that it can work with apply.

CHAPTER 17. APPLICATIVE 1109
pure (.)
<*>[(+1)]
<*>[(*2)]
<*>[1,2,3]
[(+1)]<*>([(*2)]<*>[1,2,3])
pure (.)
<*>Just(+1)
<*>Just(*2)
<*>Just1
Just(+1)
<*>(Just(*2)<*>Just1)
This law is meant to ensure that there are no surprises
resulting from composing your function applications.
3.Homomorphism
Ahomomorphism is a structure-preserving map between
two algebraic structures. The eﬀect of applying a func-
tion that is embedded in some structure to a value that is
embedded in some structure should be the same as ap-
plying a function to a value without aﬀecting any outside
structure:

CHAPTER 17. APPLICATIVE 1110
puref<*>pure x=pure (f x)
That’s the statement of the law. Here’s how it looks in
practice:
pure(+1)<*>pure1
pure((+1)1)
Those two lines of code should give you the same result.
In fact, the result you see for those should be indistin-
guishable from the result of:
(+1)1
Because the structure that pureis providing there isn’t
meaningful. So you can think of this law as having to do
with the monoidal part of the applicative deal: the result
should be the result of the function application without
doing anything other than combining the structure bits.
Just as we saw how fmapis really just a special type of
function application that ignores a context or surround-
ing structure, applicative is also function application that
preserves structure. However, with applicative, since the
function being applied alsohas structure, the structures
have to be monoidal and come together in some fashion.

CHAPTER 17. APPLICATIVE 1111
pure(+1)<*>pure1::MaybeInt
pure((+1)1)::MaybeInt
Those two results should again be the same, but this time
the structure is being provided by Maybe, so will the result
of:
(+1)1
be equal this time around?
Here are a couple more examples to try out:
pure(+1)<*>pure1::[Int]
pure(+1)<*>pure1::EitheraInt
The general idea of the homomorphism law is that apply-
ing the function doesn’t change the structure around the
values.
4.Interchange
We begin again by looking at the definition of the inter-
change law:
u<*>pure y=pure ($y)<*>u

CHAPTER 17. APPLICATIVE 1112
It might help to break that down a bit. To the left of <*>
must always be a function embedded in some structure.
In the above definition, 𝑢represents a function embedded
in some structure:
Just(+2)<*>pure2
-- u <*> pure y
-- equals
Just4
The right side of the definition might be a bit less obvious.
By sectioning the $function application operator with
the𝑦, we create an environment in which the 𝑦is there,
awaiting a function to apply to it. Let’s try lining up the
types again and see if that clears this up:
pure($2)<*>Just(+2)
-- Remember, ($ 2) can become more concrete
($2)::Numa=>(a->b)->b
Just(+2)::Numa=>Maybe(a->a)
If you’re a bit confused by ($ 2), keep in mind that this
is sectioning the dollar-sign operator and applying the
second argument only, not the first. As a result, the type
changes in the following manner:

CHAPTER 17. APPLICATIVE 1113
-- These are the same
($2)
\f->f$2
($)::(a->b)->a->b
($2)::(a->b) ->b
Then concreting the types of Applicative ’s methods:
mPure::a->Maybea
mPure=pure
embed::Numa=>Maybe((a->b)->b)
embed=mPure ($2)
mApply::Maybe((a->b)->b)
->Maybe(a->b)
->Maybe b
mApply=(<*>)
myResult =pure ($2) `mApply` Just(+2)
-- myResult == Just 4
Then translating the types side by side, with diﬀerent
letters for some of the type variables to avoid confusion

CHAPTER 17. APPLICATIVE 1114
when comparing the original type with the more concrete
form:
(<*>)::Applicative f
=>f (x->y)
->f x
->f y
mApply::Maybe((a->b)->b)
->Maybe(a->b)
->Maybe b
f ~Maybe
x ~(a->b)
y ~ b
(x->y)~(a->b)->b
According to the interchange law, this should be true:
(Just(+2)<*>pure2)
==(pure ($2)<*>Just(+2))
And you can see why that should be true, because despite
the weird syntax, the two functions are doing the same
job. Here are some more examples for you to try out:

CHAPTER 17. APPLICATIVE 1115
[(+1), (*2)]<*>pure1
pure($1)<*>[(+1), (*2)]
Just(+3)<*>pure1
pure($1)<*>Just(+3)
EveryApplicative instance you write should obey those four
laws. This keeps your code composable and free of unpleasant
surprises.
17.7 You knew this was coming
Property testing the Applicative laws! You should have got the
gist of how to write properties based on laws, so we’re going to
use a library this time. Conal Elliott has a nice library called
checkers on Hackage and Github which provides some nice
properties and utilities for QuickCheck .
After installing checkers , we can reuse the existing proper-
ties for validating Monoids andFunctor s to revisit what we did
previously.

CHAPTER 17. APPLICATIVE 1116
moduleBadMonoid where
importData.Monoid
importTest.QuickCheck
importTest.QuickCheck.Checkers
importTest.QuickCheck.Classes
dataBull=
Fools
|Twoo
deriving (Eq,Show)
instance Arbitrary Bullwhere
arbitrary =
frequency [ ( 1, return Fools)
, (1, return Twoo) ]
instance MonoidBullwhere
mempty=Fools
mappend _ _ =Fools
instance EqPropBullwhere(=-=)=eq
main::IO()
main=quickBatch (monoid Twoo)

CHAPTER 17. APPLICATIVE 1117
There are some diﬀerences here worth noting. One is that
we don’t have to define the Monoid laws asQuickCheck properties
ourselves; they are already bundled into a TestBatch called
monoid . Another is that we need to define EqProp for our custom
datatype. This is straightforward because checkers exports a
function called eqwhich reuses the pre-existing Eqinstance
for the datatype. Finally, we’re passing a value of our type
tomonoid so it knows which Arbitrary instance to use to get
random values — note it doesn’t usethis value for anything.
Then we can run mainto kick it oﬀ and see how it goes:
Prelude> main
monoid:
left identity:
*** Failed! Falsifiable (after 1 test):
Twoo
right identity:
*** Failed! Falsifiable (after 2 tests):
Twoo
associativity: +++ OK, passed 500 tests.
As we expect, it was able to falsify left and right identity for
Bull. Now let’s test a pre-existing Applicative instance, such
as list or Maybe. The type for the TestBatch which validates
Applicative instances is a bit gnarly, so please bear with us:

CHAPTER 17. APPLICATIVE 1118
applicative
::(Showa,Show(m a),Show(m (a->b))
,Show(m (b->c)),Applicative m
,CoArbitrary a,EqProp(m a)
,EqProp(m b),EqProp(m c)
,Arbitrary a,Arbitrary b
,Arbitrary (m a)
,Arbitrary (m (a->b))
,Arbitrary (m (b->c)))
=>m (a, b, c) ->TestBatch
First, a trick for managing functions like this. We know it’s
going to want Arbitrary instances for the Applicative structure,
functions (from 𝑎to𝑏,𝑏to𝑐) embedded in that structure, and
that it wants EqProp instances. That’s all well and good, but we
can ignore that.
m (a, b, c) ->TestBatch
We just care about m (a, b, c) -> TestBatch . We could pass
an actual value giving us our Applicative structure and three
values which could be of diﬀerent type, but don’t have to
be. We could also pass a bottom with a type assigned to let it
know what to randomly generate for validating the Applicative
instance.
Prelude> let xs = [("b", "w", 1)]

CHAPTER 17. APPLICATIVE 1119
Prelude> quickBatch $ applicative xs
applicative:
identity: +++ OK, passed 500 tests.
composition: +++ OK, passed 500 tests.
homomorphism: +++ OK, passed 500 tests.
interchange: +++ OK, passed 500 tests.
functor: +++ OK, passed 500 tests.
Note that it defaulted the 1 :: Num a => a in order to not
have an ambiguous type. We would’ve had to specify that
outside of GHCi. In the following example we’ll use a bottom
to fire the typeclass dispatch:
Prelude> type SSI = (String, String, Int)
Prelude> :{
*Main| let trigger :: [SSI]
*Main| trigger = undefined
*Main| :}
Prelude> quickBatch (applicative trigger)
applicative:
identity: +++ OK, passed 500 tests.
composition: +++ OK, passed 500 tests.
homomorphism: +++ OK, passed 500 tests.
interchange: +++ OK, passed 500 tests.
functor: +++ OK, passed 500 tests.

CHAPTER 17. APPLICATIVE 1120
Again, it’s not evaluating the value you pass it. That value
is just to let it know what types to use.
17.8 ZipList Monoid
The default monoid of lists in the GHC Prelude is concatena-
tion, but there is another way to monoidally combine lists.
Whereas the default list mappend ends up doing the following:
[1,2,3]<>[4,5,6]
-- changes to
[1,2,3]++[4,5,6]
[1,2,3,4,5,6]
TheZipList monoid combines the values of the two lists
as parallel sequences using a monoid provided by the values
themselves to get the job done:

CHAPTER 17. APPLICATIVE 1121
[1,2,3]<>[4,5,6]
-- changes to
[
1<>4
,2<>5
,3<>6
]
This should remind you of functions like zipandzipWith .
To make the above example work, you can assert a type like
Sum Integer for theNumvalues to get a Monoid .
Prelude> import Data.Monoid
Prelude> 1 <> 2
No instance for (Num a0) arising
from a use of ‘it’
The type variable ‘a0’ is ambiguous
Note: there are several potential
instances:
... some blather that mentions Num ...
Prelude> 1 <> (2 :: Sum Integer)
Sum {getSum = 3}

CHAPTER 17. APPLICATIVE 1122
Prelude doesn’t provide this Monoid for us, so we must define
it ourselves.
moduleApl1where
importControl.Applicative
importData.Monoid
importTest.QuickCheck
importTest.QuickCheck.Checkers
importTest.QuickCheck.Classes
Some unfortunate orphan instances follow. Try to avoid
these in code you’re going to keep or release.

CHAPTER 17. APPLICATIVE 1123
-- this isn't going to work properly
instance Monoida
=>Monoid(ZipList a)where
mempty =ZipList []
mappend =liftA2 mappend
instance Arbitrary a
=>Arbitrary (ZipList a)where
arbitrary =ZipList <$>arbitrary
instance Arbitrary a
=>Arbitrary (Suma)where
arbitrary =Sum<$>arbitrary
instance Eqa
=>EqProp(ZipList a)where
(=-=)=eq
If we fire this up in the REPL, and test for its validity as a
Monoid , it’ll fail.
Prelude> let zl = ZipList [1 :: Sum Int]
Prelude> quickBatch $ monoid zl
monoid:
left identity:

CHAPTER 17. APPLICATIVE 1124
*** Failed! Falsifiable (after 3 tests):
ZipList [ Sum {getSum = -1} ]
right identity:
*** Failed! Falsifiable (after 4 tests):
ZipList [ Sum {getSum = -1}
, Sum {getSum = 3}
, Sum {getSum = 2} ]
associativity: +++ OK, passed 500 tests.
The problem is that the empty ZipList is thezeroand not
theidentity !
Zero vs. Identity
-- Zero
n*0==0
-- Identity
n*1==n
So how do we get an identity for ZipList ?
Sum1`mappend` ??? ->Sum1
instance Monoida
=>Monoid(ZipList a)where
mempty =pure mempty
mappend =liftA2 mappend

CHAPTER 17. APPLICATIVE 1125
You’ll find out what the puredoes here when you write the
Applicative forZipList yourself.
List Applicative Exercise
Implement the list Applicative . Writing a minimally complete
Applicative instance calls for writing the definitions of both
pureand<*>. We’re going to provide a hint as well. Use the
checkers library to validate your Applicative instance.
dataLista=
Nil
|Consa (Lista)
deriving (Eq,Show)
Remember what you wrote for the list Functor :
instance Functor Listwhere
fmap=undefined
Writing the list Applicative is similar.
instance Applicative Listwhere
pure=undefined
(<*>)=undefined
Expected result:

CHAPTER 17. APPLICATIVE 1126
Prelude> let f = Cons (+1) (Cons (*2) Nil)
Prelude> let v = Cons 1 (Cons 2 Nil)
Prelude> f <*> v
Cons 2 (Cons 3 (Cons 2 (Cons 4 Nil)))
In case you get stuck, use the following functions and hints.
append::Lista->Lista->Lista
appendNilys=ys
append(Consx xs) ys =
Consx$xs `append` ys
fold::(a->b->b)->b->Lista->b
fold_bNil =b
foldf b (Consh t)=f h (fold f b t)
concat' ::List(Lista)->Lista
concat' =fold append Nil
-- write this one in terms
-- of concat' and fmap
flatMap ::(a->Listb)
->Lista
->Listb
flatMap f as=undefined

CHAPTER 17. APPLICATIVE 1127
Use the above and try using flatMap andfmapwithout explic-
itly pattern matching on cons cells. You’ll still need to handle
theNilcases.
flatMap is less strange than it would initially seem. It’s basi-
cally “fmap, then smush.”
Prelude> fmap (\x -> [x, 9]) [1, 2, 3]
[[1,9],[2,9],[3,9]]
Prelude> let toMyList = foldr Cons Nil
Prelude> let xs = toMyList [1, 2, 3]
Prelude> let c = Cons
Prelude> let f x = x `c` (9 `c` Nil)
Prelude> flatMap f xs
Cons 1 (Cons 9 (Cons 2
(Cons 9 (Cons 3 (Cons 9 Nil)))))
Applicative instances, unlike Functor s, are not guaranteed to
have a unique implementation for a given datatype.
ZipList Applicative Exercise
Implement the ZipList Applicative . Use the checkers library to
validate your Applicative instance. We’re going to provide the
EqProp instance and explain the weirdness in a moment.

CHAPTER 17. APPLICATIVE 1128
dataLista=
Nil
|Consa (Lista)
deriving (Eq,Show)
take'::Int->Lista->Lista
take'=undefined
instance Functor Listwhere
fmap=undefined
instance Applicative Listwhere
pure=undefined
(<*>)=undefined
newtype ZipList' a=
ZipList' (Lista)
deriving (Eq,Show)
instance Eqa=>EqProp(ZipList' a)where
xs=-=ys=xs' `eq` ys'
wherexs'= let(ZipList' l)=xs
intake'3000l
ys'= let(ZipList' l)=ys
intake'3000l

CHAPTER 17. APPLICATIVE 1129
instance Functor ZipList' where
fmap f ( ZipList' xs)=
ZipList' $fmap f xs
instance Applicative ZipList' where
pure=undefined
(<*>)=undefined
The idea is to align a list of functions with a list of values
and apply the first function to the first value and so on. The
instance should work with infinite lists. Some examples:
Prelude> let zl' = ZipList'
Prelude> let z = zl' [(+9), (*2), (+8)]
Prelude> let z' = zl' [1..3]
Prelude> z <*> z'
ZipList' [10,4,11]
Prelude> let z' = zl' (repeat 1)
Prelude> z <*> z'
ZipList' [10,2,9]
Note that the second z'was an infinite list. Check Prelude
for functions that can give you what you need. One starts
with the letter z, the other with the letter r. You’re looking
for inspiration from these functions, not to be able to directly
reusethemasyou’reusingacustom Listtype, nottheprovided
Prelude list type.

CHAPTER 17. APPLICATIVE 1130
Explaining and justifying the weird EqProp The good news is
it’sEqProp that has the weird “check only the first 3,000 values”
semantics instead of making the Eqinstance weird. The bad
news is this is a byproduct of testing for equality between infi-
nite lists…that is, you can’t. If you use a typical EqProp instance,
the test for homomorphism in your Applicative instance will
chase the infinite lists forever. Since QuickCheck is already an
exercise in “good enough” validity checking, we could choose
to feel justified in this. If you don’t believe us try running the
following in your REPL:
repeat 1 == repeat 1
Either and Validation Applicative
Yep, here we go again with the types:
Specializing the types
-- f ~ Either e
typeE=Either
(<*>)::f (a->b)->f a->f b
(<*>)::Ee (a->b)->Ee a->Ee b
pure::a->f a
pure::a->Ee a

CHAPTER 17. APPLICATIVE 1131
Either versus Validation
Often the interesting part of an Applicative is the monoid. One
byproduct of this is that just as you can have more than one
validMonoid for a given datatype, unlike Functor ,Applicative
can have more than one valid and lawful instance for a given
datatype.
The following is a brief demonstration of Either :
Prelude> pure 1 :: Either e Int
Right 1
Prelude> Right (+1) <*> Right 1
Right 2
Prelude> Right (+1) <*> Left ":("
Left ":("
Prelude> Left ":(" <*> Right 1
Left ":("
Prelude> Left ":(" <*> Left "sadface.png"
Left ":("
We’vecoveredthebenefitsof Either alreadyandwe’veshown
you what the Maybe Applicative can clean up, so we won’t be-
labor those points. There’s an alternative to Either, called
Validation , that diﬀers only in the Applicative instance:

CHAPTER 17. APPLICATIVE 1132
dataValidation err a=
Failure err
|Success a
deriving (Eq,Show)
One thing to realize is that this is identical to theEither
datatype and there is even a pair of total functions which can
go between Validation andEither values interchangeably. Re-
member when we mentioned natural transformations? Both
of these functions are natural transformations:
validToEither ::Validation e a
->Eithere a
validToEither (Failure err)=Lefterr
validToEither (Success a)=Righta
eitherToValid ::Eithere a
->Validation e a
eitherToValid (Lefterr)=Failure err
eitherToValid (Righta)=Success a
eitherToValid .validToEither ==id
validToEither .eitherToValid ==id
Howdoes Validation diﬀer? Principallyinwhatthe Applicative
instance does with errors. Rather than just short-circuiting

CHAPTER 17. APPLICATIVE 1133
when it has two error values, it’ll use the Monoid typeclass to
combine them. Often this’ll just be a list or set of errors but
you can do whatever you want.
dataErrors=
DividedByZero
|StackOverflow
|MooglesChewedWires
deriving (Eq,Show)
success =Success (+1)
<*>Success 1
success ==Success 2
failure =Success (+1)
<*>Failure [StackOverflow ]
failure ==Failure [StackOverflow ]
failure' =Failure [StackOverflow ]
<*>Success (+1)
failure' ==Failure [StackOverflow ]

CHAPTER 17. APPLICATIVE 1134
failures =
Failure [MooglesChewedWires ]
<*>Failure [StackOverflow ]
failures ==
Failure [MooglesChewedWires
,StackOverflow ]
With the value failures , we see what distinguishes Either
andValidation : we can now preserve allfailures that occurred,
not just the first one.
Exercise: Variations on Either
Validation has the same representation as Either , but it can be
diﬀerent. The Functor will behave the same, but the Applicative
willbediﬀerent. Seeaboveforanideaofhow Validation should
behave. Use the checkers library.
dataValidation e a=
Failure e
|Success a
deriving (Eq,Show)

CHAPTER 17. APPLICATIVE 1135
-- same as Either
instance Functor (Validation e)where
fmap=undefined
-- This is different
instance Monoide=>
Applicative (Validation e)where
pure=undefined
(<*>)=undefined
17.9 Chapter Exercises
Given a type that has an instance of Applicative , specialize the
types of the methods. Test your specialization in the REPL.
One way to do this is to bind aliases of the typeclass methods
to more concrete types that have the type we told you to fill
in.
1.-- Type
[]
-- Methods
pure::a-> ?a
(<*>):: ?(a->b)-> ?a-> ?b

CHAPTER 17. APPLICATIVE 1136
2.-- Type
IO
-- Methods
pure::a-> ?a
(<*>):: ?(a->b)-> ?a-> ?b
3.-- Type
(,) a
-- Methods
pure::a-> ?a
(<*>):: ?(a->b)-> ?a-> ?b
4.-- Type
(->) e
-- Methods
pure::a-> ?a
(<*>):: ?(a->b)-> ?a-> ?b
Write instances for the following datatypes. Confused?
Write out what the type should be. Use the checkers library
to validate the instances.
1.dataPaira=Paira aderiving Show

CHAPTER 17. APPLICATIVE 1137
2.This should look familiar.
dataTwoa b=Twoa b
3.dataThreea b c=Threea b c
4.dataThree'a b=Three'a b b
5.dataFoura b c d =Foura b c d
6.dataFour'a b=Four'a a a b
Combinations
Remember the vowels and stops exercise in the folds chapter?
Write the function to generate the possible combinations of
three input lists using liftA3 fromControl.Applicative .
importControl.Applicative (liftA3)
stops::String
stops="pbtdkg"
vowels::String
vowels="aeiou"
combos::[a]->[b]->[c]->[(a, b, c)]
combos=undefined

CHAPTER 17. APPLICATIVE 1138
17.10 Definitions
1.Applicative can be thought of characterizing monoidal
functors in Haskell. For a Haskeller’s purposes, it’s a way
to functorially apply a function which is embedded in
structure 𝑓of the same type as the value you’re mapping
it over.
fmap::(a->b)->f a->f b
(<*>)::f (a->b)->f a->f b
17.11 Follow-up resources
1.Tony Morris; Nick Partridge; Validation library
http://hackage.haskell.org/package/validation
2.Conor McBride; Ross Paterson; Applicative Programming
with Eﬀects
http://staff.city.ac.uk/~ross/papers/Applicative.html
3.Jeremy Gibbons; Bruno C. d. S. Oliveira; Essence of the
Iterator Pattern
4.Ross Paterson; Constructing Applicative Functors
http://staff.city.ac.uk/~ross/papers/Constructors.html

CHAPTER 17. APPLICATIVE 1139
5.Sam Lindley; Philip Wadler; Jeremy Yallop; Idioms are
oblivious, arrows are meticulous, monads are promiscu-
ous.
Note: Idiom means applicative functor and is a useful
search term for published work on applicative functors.

Chapter 18
Monad
There is nothing so
practical as a good theory
Phil Wadler, quoting Kurt
Lewin
1140

CHAPTER 18. MONAD 1141
18.1 Monad
Finally we come to one of the most talked about structures in
Haskell: the monad. Monads are not, strictly speaking, neces-
sary to Haskell. Although the current Haskell standard does
use monad for constructing and transforming IOactions, older
implementations of Haskell did not. Monads are powerful
and fun, but they do not define Haskell. Rather, monads are
defined in terms of Haskell.
Monads are applicative functors, but they have something
special about them that makes them diﬀerent from and more
powerful than either <*>orfmapalone. In this chapter, we
•defineMonad, its operations and laws;
•look at several examples of monads in practice;
•write the Monadinstances for various types;
•address some misinformation about monads.
18.2 Sorry — a monad is not a burrito
Well, then what the heck is a monad?1
1Section title with all due respect and gratitude to Mark Jason Dominus, whose
blog post, “Monads are like burritos” is a classic of its genre. http://blog.plover.com/prog/
burritos.html

CHAPTER 18. MONAD 1142
As we said above, a monad is an applicative functor with
some unique features that make it a bit more powerful than
either alone. A functor maps a function over some structure;
an applicative maps a function that is contained in some struc-
ture over some other structure and then combines the two
layers of structure like mappend . So you can think of monads
as another way of applying functions over structure, with a
couple of additional features. We’ll get to those features in a
moment. For now, let’s check out the typeclass definition and
core operations.
If you are using GHC 7.10 or newer, you’ll see an Applicative
constraint in the definition of Monad, as it should be:
classApplicative m=>Monadmwhere
(>>=)::m a->(a->m b)->m b
(>>)::m a->m b->m b
return::a->m a
We’re going to explore this in some detail. Let’s start with
the typeclass constraint on 𝑚.
Applicative m
Older versions of GHC did not have Applicative as a superclass
ofMonad. Given that Monadis stronger than Applicative , and
Applicative is stronger than Functor , you can derive Applicative
andFunctor in terms of Monad, just as you can derive Functor in

CHAPTER 18. MONAD 1143
terms of Applicative . What does this mean? It means you can
writefmapusing monadic operations and it works:
fmapf xs=xs>>=return.f
Try it for yourself:
Prelude> fmap (+1) [1..3]
[2,3,4]
Prelude> [1..3] >>= return . (+1)
[2,3,4]
Thishappenstobealaw, notaconvenience. Functor ,Applicative ,
andMonadinstances over a given type should have the same
core behavior.
We’ll explore the relationship between these classes more
completely in a bit, but as part of understanding the typeclass
definition above, it’s important to understand this chain of
dependency:
Functor ->Applicative ->Monad
Whenever you’ve implemented an instance of Monadfor a
type you necessarily have an Applicative and aFunctor as well.

CHAPTER 18. MONAD 1144
Core operations
TheMonadtypeclass defines three core operations, although
you only need to define >>=for a minimally complete Monad
instance. Let’s look at all three:
(>>=)::m a->(a->m b)->m b
(>>)::m a->m b->m b
return::a->m a
We can dispense with the last of those, return: it’s just the
same as pure. All it does is take a value and return it inside
your structure, whether that structure is a list or JustorIO. We
talked about it a bit, and used it, back in the Modules chapter,
and we covered purein theApplicative chapter, so there isn’t
much else to say about it.
Thenextoperator, >>, doesn’thaveanofficialEnglish-language
name, but we like to call it Mr. Pointy. Some people do re-
fer to it as the sequencing operator, which we must admit is
more informative than Mr. Pointy. Mr. Pointy sequences
two actions while discarding any resulting value of the first
action. Applicative has a similar operator as well, although we
didn’t talk about it in that chapter. We will see examples of
this operator in the upcoming section on dosyntax.
Finally, the big bind! The>>=operator is called bindand is
— or, at least, comprises — what makes Monadspecial.

CHAPTER 18. MONAD 1145
The novel part of Monad
Conventionallywhenweusemonads, weusethebindfunction,
>>=. Sometimes we use it directly, sometimes indirectly via do
syntax. The question we should ask ourselves is, what’s unique
toMonad— at least from the point of view of types?
We already saw that it’s not return ; that’s another name for
purefromApplicative .
We also noted (and will see more clearly soon) that it also
isn’t>>which has a counterpart in Applicative .
And it also isn’t >>=, at least not in its entirety. The type of
>>=is visibly similar to that of fmapand<*>, which makes sense
since monads are applicative functors. For the sake of making
this maximally similar, we’re going to change the 𝑚ofMonad
to𝑓:
fmap::Functor f
=>(a->b)->f a->f b
<*> :: Applicative f
=>f (a->b)->f a->f b
>>= :: Monadf
=>f a->(a->f b)->f b
OK, so bind is quite similar to <*>andfmapbut with the first
two arguments flipped. Still, the idea of mapping a function
over a value while bypassing its surrounding structure is not
unique to Monad.

CHAPTER 18. MONAD 1146
We can demonstrate this by fmapping a function of type (a
-> m b) to make it more like >>=, and it will work. Nothing will
stop us. We will continue using the tilde to represent rough
equivalence between two things:
-- If b == f b
fmap::Functor f
=>(a->f b)->f a->f (f b)
Let’s demonstrate this idea with list as our structure:
Prelude> let andOne x = [x, 1]
Prelude> andOne 10
[10,1]
Prelude> :t fmap andOne [4, 5, 6]
fmap andOne [4, 5, 6] :: Num t => [[t]]
Prelude> fmap andOne [4, 5, 6]
[[4,1],[5,1],[6,1]]
But, lo! We knew from our types that we’d end up with
anf (f b) — that is, an extra layer of structure, and now we
have a result of nested lists. What if we wanted Num a => [a]
instead of nested lists? We want a single layer of 𝑓structure,
but our mapped function has itself generated more structure !

CHAPTER 18. MONAD 1147
After mapping a function that generates additional monadic
structure in its return type, we want a way to discard one layer
of that structure.
So how do we accomplish that? Well, we saw how to do
what we want with lists very early on in this book:
Prelude> concat $ fmap andOne [4, 5, 6]
[4,1,5,1,6,1]
The type of concat , fully generalized:
concat::Foldable t=>t [a]->[a]
-- we can assert a less general type
-- for our purposes here
concat::[[a]]->[a]
Monad, in a sense, is a generalization of concat! The unique
part ofMonadis the following function:
importControl.Monad (join)
join::Monadm=>m (m a) ->m a
-- compare
concat:: [[a]] ->[a]

CHAPTER 18. MONAD 1148
It’s somewhat novel that we can inject more structure via
our function application, where applicatives and fmaps have to
leave the structure untouched. Allowing the function itself to
alter the structure is something we’ve not seen in Functor and
Applicative , and we’ll explore the ramifications of that ability
more, especially when we start talking about the Maybemonad.
But we caninject more structure with a standard fmapif we
wish, as we saw above. However, the ability to flatten those two
layers of structure into one is what makes Monadspecial. And
it’s by putting that joinfunction together with the mapping
function that we get bind, also known as >>=.
So how do we get bind?
The answer is the exercise Writebindin terms of fmapand
join.
Fear is the mind-killer, friend. You can do it.
-- keep in mind this is (>>=) flipped
bind::Monadm=>(a->m b)->m a->m b
bind=undefined
WhatMonadis not
SinceMonadis somewhat abstract and a little slippery, many
people talk about it from one or two perspectives that they feel
most comfortable with. Quite often, they address what Monad

CHAPTER 18. MONAD 1149
is from the perspective of the IO Monad .IOdoes have a Monad
instance, and it is a very common use of monads. However,
understanding monads only through that instance leads to
limited intuitions for what monads are and can do, and to a
lesser extent, a wrong notion of what IOis all about.
A monad is not:
1.Impure. Monadic functions are pure functions. IOis an ab-
stract datatype that allows for impure, or eﬀectful, actions,
and it has a Monadinstance. But there’s nothing impure
about monads.
2.An embedded language for imperative programming. Si-
mon Peyton-Jones, one of the lead developers and re-
searchers of Haskell and its implementation in GHC, has
famously said, “Haskell is the world’s finest imperative
programming language,” and he was talking about the
way monads handle eﬀectful programming. While mon-
ads are often used for sequencing actions in a way that
looks like imperative programming, there are commuta-
tive monads that do not order actions. We’ll see one a few
chapters down the line when we talk about Reader .
3.A value. The typeclass describes a specific relationship be-
tween elements in a domain and defines some operations
over them. When we refer to something as “a monad,”

CHAPTER 18. MONAD 1150
we’re using that the same way we talk about “a monoid,”
or “a functor.” None of those are values.
4.About strictness. The monadic operations of bindand
return are nonstrict. Some operations can be made strict
within a specific instance. We’ll talk more about this later
in the book.
Using monads also doesn’t require knowing math. Or cate-
gory theory. It does not require mystical trips to the tops of
mountains or starving oneself in a desert somewhere.
TheMonadtypeclass is generalized structure manipulation
withsomelawstomakeitsensible. Justlike Functor andApplicative .
We sort of hate to diminish the mystique, but that’s all there is
to it.
Monadalso lifts!
TheMonadclass also includes a set of liftfunctions that are the
same as the ones we already saw in Applicative . They don’t
do anything diﬀerent, but they are still around because some
libraries used them before applicatives were discovered, so
theliftMset of functions still exists to maintain compatibil-
ity. So, you may still see them sometimes. We’ll take a short
tour of them, comparing them directly to their applicative
counterparts:

CHAPTER 18. MONAD 1151
liftA::Applicative f
=>(a->b)->f a->f b
liftM::Monadm
=>(a1->r)->m a1->m r
As you may recall, that is fmapwith a diﬀerent typeclass
constraint. If you’d like to see examples of how it works, we
encourage you to write fmapfunctions in your REPL and take
turns replacing the fmapwithliftAorliftM.
But that’s not all we have:
liftA2::Applicative f
=>(a->b->c)
->f a
->f b
->f c
liftM2::Monadm
=>(a1->a2->r)
->m a1
->m a2
->m r
Aside from the numbering these appear the same. Let’s try
them out:
Prelude> liftA2 (,) (Just 3) (Just 5)

CHAPTER 18. MONAD 1152
Just (3,5)
Prelude> liftM2 (,) (Just 3) (Just 5)
Just (3,5)
You may remember way back in Lists, we talked about a
function called zipWith .zipWith isliftA2 orliftM2 specialized
to lists:
Prelude> :t zipWith
zipWith :: (a -> b -> c)
-> [a] -> [b] -> [c]
Prelude> zipWith (+) [3, 4] [5, 6]
[8,10]
Prelude> liftA2 (+) [3, 4] [5, 6]
[8,9,9,10]
Well, the types are the same, but the behavior diﬀers. The
diﬀering behavior has to do with which list monoid is being
used.
All right. Then we have the threes:

CHAPTER 18. MONAD 1153
liftA3::Applicative f
=>(a->b->c->d)
->f a->f b
->f c->f d
liftM3::Monadm
=>(a1->a2->a3->r)
->m a1->m a2
->m a3->m r
And, coincidentally, there is also a zipWith3 function. Let’s
see what happens:
Prelude> :t zipWith3
zipWith3 :: (a -> b -> c -> d) ->
[a] -> [b] -> [c] -> [d]
Prelude> liftM3 (,,) [1, 2] [3] [5, 6]
[(1,3,5),(1,3,6),(2,3,5),(2,3,6)]
Prelude> zipWith3 (,,) [1, 2] [3] [5, 6]
[(1,3,5)]
Again, using a diﬀerent monoid gives us a diﬀerent set of
results.
We wanted to introduce these functions here because they
will come up in some later examples in the chapter, but they
aren’t especially pertinent to Monad, and we saw the gist of them

CHAPTER 18. MONAD 1154
in the previous chapter. So, let’s turn our attention back to
monads, shall we?
18.3 Do syntax and monads
We introduced dosyntax in the Modules chapter. We were
using it within the context of IOas syntactic sugar that allowed
us to easily sequence actions by feeding the result of one action
