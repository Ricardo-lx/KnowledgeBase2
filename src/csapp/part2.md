representations.
To	convert	an	unsigned	number	to	a	larger	data	type,	we	can	simply	add
leading	zeros	to	the	representation;	this	operation	is	known	as	
zero
extension
,	expressed	by	the	following	principle:
Principle:
Expansion	of	an	unsigned	number	by	zero	extension
Define	bit	vectors	
of	width	
w
	and
of	width	
w
′,	where	
w
′	>	
w
.
Then	
.
32
u
→
=
[
u
w
−
1
,
u
w
−
1
,
…
,
u
0
]
	
u
→
′
=
[
0
,
…
,
0
,
u
w
−
1
,
u
w
−
2
,
…
u
0
]
	
B
2
U
w
(
u
→
)
=
B
2
U
w
′
(
u
→
′
)

This	principle	can	be	seen	to	follow	directly	from	the	definition	of	the
unsigned	encoding,	given	by	
Equation	
2.1
.
For	converting	a	two's-complement	number	to	a	larger	data	type,	the	rule
is	to	perform	a	
sign	extension
,	adding	copies	of	the	most	significant	bit	to
the	representation,	expressed	by	the	following	principle.	We	show	the
sign	bit	
x
	in	blue	to	highlight	its	role	in	sign	extension.
Principle:
Expansion	of	a	two's-complement	number	by	sign
extension
Define	bit	vectors	
of	width	
w
	and
of	width	
w
′,	where	
w
′
>	
w
.	Then	
.
As	an	example,	consider	the	following	code:
w
–1
x
→
=
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
	
x
→
′
=
[
x
w
−
1
,
…
,
x
w
−
1
,
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
	
B
2
T
w
(
x
→
)
=
B
2
T
w
′
(
x
→
′
)

When	run	as	a	32–bit	program	on	a	big-endian	machine	that	uses	a
two's-complement	representation,	this	code	prints	the	output
We	see	that,	although	the	two's-complement	representation	of	–12,345
and	the	unsigned	representation	of	53,191	are	identical	for	a	16–bit	word
size,	they	differ	for	a	32–bit	word	size.	In	particular,	-12,345	has
hexadecimal	representation	
,	while	53,191	has	hexadecimal
representation	
.	The	former	has	been	sign	extended—16
copies	of	the	most	significant	bit	1,	having	hexadecimal	representation
0xFFFF,	have	been	added	as	leading	bits.	The	latter	has	been	extended
with	16	leading	zeros,	having	hexadecimal	representation	
.

As	an	illustration,	
Figure	
2.20
	shows	the	result	of	expanding	from
word	size	
w
	=	3	to	
w
	=	4	by	sign	extension.	Bit	vector	[
101
]represents	the
value	–4	+	1	=	–3.	Applying	sign	extension	gives	bit	vector	[1101]
representing	the	value	–8	+	4	+	1	=	–3.	We	can	see	that,	for	
w
	=	4,	the
combined	value	of	the	two	most	significant	bits,	–8	+	4	=	–4,	matches	the
value	of	the	sign	bit	for	
w
	=	3.	Similarly,	bit	vectors	[
111
]	and	[1111]	both
represent	the	value	–1.
With	this	as	intuition,	we	can	now	show	that	sign	extension	preserves	the
value	of	a	two's-complement	number.
Figure	
2.20	
Examples	of	sign	extension	from	
w
	=	3	to	
w
	=	4.
For	
w
	=	4,	the	combined	weight	of	the	upper	2	bits	is	–8	+	4	=	–4,
matching	that	of	the	sign	bit	for	
w
	=	3.

Derivation:
Expansion	of	a	two's-complement	number	by	sign
extension	Let	
w′	=	w	+	k.
	What	we	want	to	prove	is	that
The	proof	follows	by	induction	on	
k.
	That	is,	if	we	can	prove
that	sign	extending	by	1	bit	preserves	the	numeric	value,
then	this	property	will	hold	when	sign	extending	by	an
arbitrary	number	of	bits.	Thus,	the	task	reduces	to	proving
that
Expanding	the	left-hand	expression	with	
Equation	
2.3
gives	the	following:
The	key	property	we	exploit	is	that	
.	Thus,
the	combined	effect	of	adding	a	bit	of	weight	–2
	and	of
converting	the	bit	having	weight	–2
	to	be	one	with	weight
2
	is	to	preserve	the	original	numeric	value.
B
2
T
w
+
k
(
[
x
w
−
1
,
…
,
x
w
−
1
︸
k
 
times
,
 
x
w
−
2
,
…
,
 
x
0
]
)
=
B
2
T
(
[
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
B
2
T
w
+
1
(
[
x
w
−
1
,
 
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
=
B
2
T
w
(
[
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
B
2
T
w
+
1
(
[
x
w
−
1
,
 
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
=
−
x
w
−
1
2
w
+
∑
i
=
0
w
−
1
x
i
2
i
 
=
−
x
w
−
1
2
w
+
x
w
−
1
2
w
−
1
+
∑
i
=
0
w
−
2
x
i
2
i
 
=
−
x
w
−
1
(
2
w
−
2
w
−
1
)
+
∑
i
=
0
w
−
2
x
i
2
i
 
=
−
x
w
−
1
2
w
−
1
+
∑
i
=
0
w
−
2
x
i
2
i
 
=
B
2
T
w
(
[
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
2
w
−
2
w
−
1
=
2
w
−
1
w
w
–1
w
–1

Practice	Problem	
2.22
	(solution	page	
150
)
Show	that	each	of	the	following	bit	vectors	is	a	two's-complement
representation	of	–5	by	applying	
Equation	
2.3
:
A
.	
[1011]
B
.	
[11011]
C
.	
[111011]
Observe	that	the	second	and	third	bit	vectors	can	be	derived	from
the	first	by	sign	extension.
One	point	worth	making	is	that	the	relative	order	of	conversion	from	one
data	size	to	another	and	between	unsigned	and	signed	can	affect	the
behavior	of	a	program.	Consider	the	following	code:
When	run	on	a	big-endian	machine,	this	code	causes	the	following	output
to	be	printed:

This	shows	that,	when	converting	from	
	to	
,	the	program
first	changes	the	size	and	then	the	type.	That	is,	
	is
equivalent	to	
,	evaluating	to	4,294,954,951,	not
,	which	evaluates	to	53,191.	Indeed,	this
convention	is	required	by	the	C	standards.
Practice	Problem	
2.23
	(solution	page	
150
)
Consider	the	following	C	functions:
Assume	these	are	executed	as	a	32–bit	program	on	a	machine
that	uses	two's-complement	arithmetic.	Assume	also	that	right
shifts	of	signed	values	are	performed	arithmetically,	while	right
shifts	of	unsigned	values	are	performed	logically.
A
.	
Fill	in	the	following	table	showing	the	effect	of	these
functions	for	several	example	arguments.	You	will	find	it

more	convenient	to	work	with	a	hexadecimal
representation.	Just	remember	that	hex	digits	8	through	F
have	their	most	significant	bits	equal	to	1.
_________
_________
_________
_________
_________
_________
_________
_________
B
.	
Describe	in	words	the	useful	computation	each	of	these
functions	performs.
2.2.7	
Truncating	Numbers
Suppose	that,	rather	than	extending	a	value	with	extra	bits,	we	reduce
the	number	of	bits	representing	a	number.	This	occurs,	for	example,	in
the	following	code:

Casting	
	to	be	
	will	truncate	a	32-bit	
	to	a	16-bit	
.	As	we
saw	before,	this	16–bit	pattern	is	the	two's-complement	representation	of
–12,345.	When	casting	this	back	to	
,	sign	extension	will	set	the	high-
order	16	bits	to	ones,	yielding	the	32–bit	two's-complement
representation	of	–12,345.
When	truncating	a	
w
-bit	number	
to	a	
k
-bit
number,	we	drop	the	high-order	
w	–	k
	bits,	giving	a	bit	vector	
.	Truncating	a	number	can	alter	its	value—a	form	of
overflow.	For	an	unsigned	number,	we	can	readily	characterize	the
numeric	value	that	will	result.
Principle:
Truncation	of	an	unsigned	number
Let	
be	the	bit	vector	
,	and	let	
be
the	result	of	truncating	it	to	
k
	bits:	
and	
.	Then	
x
′	=	
x
mod	2
.
The	intuition	behind	this	principle	is	simply	that	all	of	the	bits	that	were
truncated	have	weights	of	the	form	2
i
,	where	
i
	≥	
k
,	and	therefore	each	of
these	weights	reduces	to	zero	under	the	modulus	operation.	This	is
formalized	by	the	following	derivation:
x
→
=
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
	
x
→
′
=
[
x
k
−
1
,
x
k
−
2
,
…
,
x
0
]
x
→
	
[
x
w
−
1
.
x
w
−
2
,
…
,
x
0
]
x
→
′
	
x
→
′
=
[
x
k
−
1
,
x
k
−
2
,
…
,
x
0
]
.
 
L
e
t
 
x
=
B
2
U
w
(
x
→
)
	
x
′
=
B
2
U
k
(
x
→
′
)
k

Derivation:
Truncation	of	an	unsigned	number
Applying	the	modulus	operation	to	
Equation	
2.1
	yields
In	this	derivation,	we	make	use	of	the	property	that	2
	mod
2
	=	0	for	any	
i
	≥	
k
.
A	similar	property	holds	for	truncating	a	two's-complement	number,
except	that	it	then	converts	the	most	significant	bit	into	a	sign	bit:
Principle:
Truncation	of	a	two's-complement	number
Let	
be	the	bit	vector	
,	and	let	
be
the	result	of	truncating	it	to	
k
	bits:	
.
B
2
U
w
(
[
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
 
mod
 
2
k
=
[
∑
i
=
0
w
−
1
x
i
2
i
]
 
mod
 
2
k
 
=
[
∑
i
=
0
k
−
1
x
i
2
i
]
 
mod
 
2
k
 
=
∑
i
=
0
k
−
1
x
i
2
i
 
=
B
2
U
k
(
[
x
k
−
1
,
 
x
k
−
2
,
…
 
x
0
]
)
i
k
x
→
	
[
x
w
−
1
.
x
w
−
2
,
…
,
x
0
]
′
x
→
	
x
→
′
=
[
x
k
−
1
,
x
k
−
2
,
…
,
x
0
]

Let	
and	
.	Then	
x
′	=	
U2T
(
x
	mod
2
).
In	this	formulation,	
x
	mod	2
	will	be	a	number	between	0	and	2
	–	1.
Applying	function	
U2T
	to	it	will	have	the	effect	of	converting	the	most
significant	bit	
x
	from	having	weight	2
	to	having	weight	–2
.	We	can
see	this	with	the	example	of	converting	value	
x
	=	53,191	from	
int
	to	
short
.
Since	2
	=	65,536	≥	
x
,	we	have	
x
	mod	2
	=	
x
.	But	when	we	convert	this
number	to	a	16–bit	two's-complement	number,	we	get	
.
Derivation:
Truncation	of	a	two's-complement	number
Using	a	similar	argument	to	the	one	we	used	for	truncation
of	an	unsigned	number	shows	that
That	is,	
x
	mod	2
	can	be	represented	by	an	unsigned
number	having	bit-level	representation	
.
Converting	this	to	a	two's-complement	number	gives	
).
x
=
B
2
T
w
(
x
→
)
	
x
′
=
B
2
T
k
(
x
→
′
)
k
k
k
k
k
k
–1
k
–1
k
–1
16
16
x
′
=
53
,
191
−
65
,
536
=
−
12
,
345
B
2
T
w
(
[
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
 
mod
 
2
k
=
B
2
U
k
(
[
x
k
−
1
,
 
x
k
−
2
,
…
,
 
x
0
]
)
k
[
x
k
−
1
,
x
k
−
2
,
…
,
x
0
]
x
′
=
U
2
T
k
(
x
 
mod
 
2
k
)

Summarizing,	the	effect	of	truncation	for	unsigned	numbers	is
while	the	effect	for	two's-complement	numbers	is
Practice	Problem	
2.24
	(solution	page	
150
)
Suppose	we	truncate	a	4–bit	value	(represented	by	hex	digits	
through	
)	to	a	3–bit	value	(represented	as	hex	digits	
	through
.)	Fill	in	the	table	below	showing	the	effect	of	this	truncation	for
some	cases,	in	terms	of	the	unsigned	and	two's-complement
interpretations	of	those	bit	patterns.
Hex
Unsigned
Two's	complement
Original
Truncated
Original
Truncated
Original
Truncated
0
___________
0
___________
2
___________
2
___________
9
___________
–7
___________
11
___________
–5
___________
15
___________
–1
___________
B
2
U
k
(
[
x
k
−
1
,
 
x
k
−
2
,
…
,
 
x
0
]
)
=
B
2
U
w
(
[
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
 
mod
 
2
k
(2.9)
B
2
T
k
(
[
x
k
−
1
,
 
x
k
−
2
,
…
,
 
x
0
]
)
=
B
2
U
w
(
B
2
U
w
(
[
x
w
−
1
,
 
x
w
−
2
,
…
,
 
x
0
]
)
 
mod
 
2
k
)
(2.10)

Explain	how	
Equations	
2.9
	and	
2.10
	apply	to	these	cases.
2.2.8	
Advice	on	Signed	versus
Unsigned
As	we	have	seen,	the	implicit	casting	of	signed	to	unsigned	leads	to
some	nonintuitive	behavior.	Nonintuitive	features	often	lead	to	program
bugs,	and	ones	involving	the	nuances	of	implicit	casting	can	be
especially	difficult	to	see.	Since	the	casting	takes	place	without	any	clear
indication	in	the	code,	programmers	often	overlook	its	effects.
The	following	two	practice	problems	illustrate	some	of	the	subtle	errors
that	can	arise	due	to	implicit	casting	and	the	unsigned	data	type.
Practice	Problem	
2.25
	(solution	page	
151
)
Consider	the	following	code	that	attempts	to	sum	the	elements	of
an	array	a,	where	the	number	of	elements	is	given	by	parameter
:

When	run	with	argument	
	equal	to	0,	this	code	should
return	0.0.	Instead,	it	encounters	a	memory	error.	Explain	why	this
happens.	Show	how	this	code	can	be	corrected.
Practice	Problem	
2.26
	(solution	page	
151
)
You	are	given	the	assignment	of	writing	a	function	that	determines
whether	one	string	is	longer	than	another.	You	decide	to	make	use
of	the	string	library	function	
	having	the	following
declaration:
Here	is	your	first	attempt	at	the	function:

When	you	test	this	on	some	sample	data,	things	do	not	seem	to
work	quite	right.	You	investigate	further	and	determine	that,	when
compiled	as	a	32-bit	
program,	data	type	
	is	defined	(via
)	in	header	file	
	to	be	
.
A
.	
For	what	cases	will	this	function	produce	an	incorrect
result?
B
.	
Explain	how	this	incorrect	result	comes	about.
C
.	
Show	how	to	fix	the	code	so	that	it	will	work	reliably.
We	have	seen	multiple	ways	in	which	the	subtle	features	of	unsigned
arithmetic,	and	especially	the	implicit	conversion	of	signed	to	unsigned,
can	lead	to	errors	or	vulnerabilities.	One	way	to	avoid	such	bugs	is	to
never	use	unsigned	numbers.	In	fact,	few	languages	other	than	C
support	unsigned	integers.	Apparently,	these	other	language	designers
viewed	them	as	more	trouble	than	they	are	worth.	For	example,	Java
supports	only	signed	integers,	and	it	requires	that	they	be	implemented
with	two's-complement	arithmetic.	The	normal	right	shift	operator	>>	is
guaranteed	to	perform	an	arithmetic	shift.	The	special	operator	>>>	is
defined	to	perform	a	logical	right	shift.
Unsigned	values	are	very	useful	when	we	want	to	think	of	words	as	just
collections	of	bits	with	no	numeric	interpretation.	This	occurs,	for
example,	when	packing	a	word	with	
flags
	describing	various	Boolean
conditions.	Addresses	are	naturally	unsigned,	so	systems	programmers
find	unsigned	types	to	be	helpful.	Unsigned	values	are	also	useful	when
implementing	mathematical	packages	for	modular	arithmetic	and	for
multiprecision	arithmetic,	in	which	numbers	are	represented	by	arrays	of
words.

2.3	
Integer	Arithmetic
Many	beginning	programmers	are	surprised	to	find	that	adding	two
positive	numbers	can	yield	a	negative	result,	and	that	the	comparison	
	can	yield	a	different	result	than	the	comparison	
.	These
properties	are	artifacts	of	the	finite	nature	of	computer	arithmetic.
Understanding	the	nuances	of	computer	arithmetic	can	help
programmers	write	more	reliable	code.
2.3.1	
Unsigned	Addition
Consider	two	nonnegative	integers	
x
	and	
y
,	such	that	0	≤	
x,	y
	<	2
.	Each
of	these	values	can	be	represented	by	a	
w
-bit	unsigned	number.	If	we
compute	their	sum,	however,	we	have	a	possible	range	
.
Representing	this	sum	could	require	
w
	+	1	bits.	For	example,	
Figure
2.21
	shows	a	plot	of	the	function	
x
	+	
y
	when	
x
	and	
y
	have	4-bit
representations.	The	arguments	(shown	on	the	horizontal	axes)	range
from	0	to	15,	but	the	sum	ranges	from	0	to	30.	The	shape	of	the	function
is	a	sloping	plane	(the	function	is	linear	in	both	dimensions).	If	we	were	to
maintain	the	sum	as	a	(
w
	+	1)-bit	number	and	add	it	to	another	value,	we
may	require	
w
	+	2	bits,	and	so	on.	This	continued	“word	size
w
0
≤
x
+
y
≤
2
w
+
1
−
2

Figure	
2.21	
Integer	addition.
With	a	4–bit	word	size,	the	sum	could	require	5	bits.
inflation”	means	we	cannot	place	any	bound	on	the	word	size	required	to
fully	represent	the	results	of	arithmetic	operations.	Some	programming
languages,	such	as	Lisp,	actually	support	
arbitrary	size
	arithmetic	to
allow	integers	of	any	size	(within	the	memory	limits	of	the	computer,	of
course.)	More	commonly,	programming	languages	support	fixed-size

arithmetic,	and	hence	operations	such	as	“addition”	and	“multiplication”
differ	from	their	counterpart	operations	over	integers.
Let	us	define	the	operation	
for	arguments	
x
	and	
y
,	where	0	≤	
x,	y
	<
2
,	as	the	result	of	truncating	the	integer	sum	
x
	+	
y
	to	be	
w
	bits	long	and
then	viewing	the	result	as	an	unsigned	number.	This	can	be
characterized	as	a	form	of	modular	arithmetic,	computing	the	sum
modulo	2
	by	simply	discarding	any	bits	with	weight	greater	than	2
	in
the	bit-level	representation	of	
x
	+	
y
.	For	example,	consider	a	4–bit
number	representation	with	
x
	=	9	and	
y
	=	12,	having	bit	representations
[1001]	and	[1100],	respectively.	Their	sum	is	21,	having	a	5–bit
representation	[10101].	But	if	we	discard	the	high-order	bit,	we	get
[0101],	that	is,	decimal	value	5.	This	matches	the	value	21	mod	16	=	5.
Aside	
Security	vulnerability	in
In	2002,	programmers	involved	in	the	FreeBSD	open-source
operating	systems	project	realized	that	their	implementation	of	the
	library	function	had	a	security	vulnerability.	A
simplified	version	of	their	code	went	something	like	this:
+
w
u
	
w
w
w
–1

In	this	code,	we	show	the	prototype	for	library	function	memcpy	on
line	7,	which	is	designed	to	copy	a	specified	number	of	bytes	
from	one	region	of	memory	to	another.
The	function	
,	starting	at	line	14,	is	designed	to
copy	some	of	the	data	maintained	by	the	operating	system	kernel
to	a	designated	region	of	memory	accessible	to	the	user.	Most	of
the	data	structures	maintained	by	the	kernel	should	not	be
readable	by	a	user,	since	they	may	contain	sensitive	information
about	other	users	and	about	other	jobs	running	on	the	system,	but

the	region	shown	as	
	was	intended	to	be	one	that	the	user
could	read.	The	parameter	
	is	intended	to	be	the	length	of
the	buffer	allocated	by	the	user	and	indicated	by	argument
.	The	computation	at	line	16	then	makes	sure	that	no
more	bytes	are	copied	than	are	available	in	either	the	source	or
the	destination	buffer.
Suppose,	however,	that	some	malicious	programmer	writes	code
that	calls	
	with	a	negative	value	of	
.	Then
the	minimum	computation	on	line	16	will	compute	this	value	for
,	which	will	then	be	passed	as	the	parameter	
	to	memcpy.
Note,	however,	that	parameter	n	is	declared	as	having	data	type
.	This	data	type	is	declared	(via	
)	in	the	library	file
.	Typically,	it	is	defined	to	be	
	for	32–bit	programs
and	
	for	64–bit	programs.	Since	argument	
	is
unsigned,	
	will	treat	it	as	a	very	large	positive	number	and
attempt	to	copy	that	many	bytes	from	the	kernel	region	to	the
user's	buffer.	Copying	that	many	bytes	(at	least	2
)	will	not
actually	work,	because	the	program	will	encounter	invalid
addresses	in	the	process,	but	the	program	could	read	regions	of
the	kernel	memory	for	which	it	is	not	authorized.
We	can	see	that	this	problem	arises	due	to	the	mismatch	between
data	types:	in	one	place	the	length	parameter	is	signed;	in	another
place	it	is	unsigned.	Such	mismatches	can	be	a	source	of	bugs
and,	as	this	example	shows,	can	even	lead	to	security
vulnerabilities.	Fortunately,	there	were	no	reported	cases	where	a
programmer	had	exploited	the	vulnerability	in	FreeBSD.	They
issued	a	security	advisory	“FreeBSD-SA-02:38.signed-error”
advising	system	administrators	on	how	to	apply	a	patch	that	would
31

remove	the	vulnerability.	The	bug	can	be	fixed	by	declaring
parameter	
	to	
	to	be	of	type	
,	to	be
consistent	with	parameter	
	of	
.	We	should	also	declare
local	variable	
	and	the	return	value	to	be	of	type	
.
We	can	characterize	operation	
as	follows:
Principle:
Unsigned	addition
For	
x
	and	
y
	such	that	0	≤	
x,	y
	<	2
:
The	two	cases	of	
Equation	
2.11
	are	illustrated	in	
Figure	
2.22
,
showing	the	sum	
x
	+	
y
	on	the	left	mapping	to	the	unsigned	
w
-bit	sum
	on	the	right.	The	normal	case	preserves	the	value	of	
x
	+	
y
,	while
the	overflow	case	has	the	effect	of	decrementing	this	sum	by	2
.
Derivation:
Unsigned	addition
+
w
u
	
w
x
+
w
u
y
=
{
x
+
y
,
x
+
y
<
2
w
 
Normal
x
+
y
−
2
w
,
2
w
≤
x
+
y
<
2
w
+
1
 
Overflow
(2.11)
x
+
w
u
y
w

In	general,	we	can	see	that	if	
,	the	leading	bit	in	the
(
w
	+	1)-bit	representation	of	the	sum	will	equal	0,	and
hence	discarding	it	will	not	change	the	numeric	value.	On
the	other	hand,	if	
,	the	leading	bit	in	the	(
w
	+
1)-bit	representation	of	the	sum	will	equal	1,	and	hence
discarding	it	is	equivalent	to	subtracting	2
	from	the	sum.
An	arithmetic	operation	is	said	to	
overflow
	when	the	full	integer	result
cannot	fit	within	the	word	size	limits	of	the	data	type.	As	
Equation	
2.11
indicates,	overflow
Figure	
2.22	
Relation	between	integer	addition	and	unsigned
addition.
When	
x
	+
y
	is	greater	than	2
	–	1,	the	sum	overflows.
x
+
y
<
2
w
2
w
≤
x
+
y
<
2
w
+
1
w
w

Figure	
2.23	
Unsigned	addition.
With	a	4-bit	word	size,	addition	is	performed	modulo	16.
occurs	when	the	two	operands	sum	to	2
	or	more.	
Figure	
2.23
	shows
a	plot	of	the	unsigned	addition	function	for	word	size	
w
	=	4.	The	sum	is
computed	modulo	2
	=	16.	When	
x
	+	
y
	<	16,	there	is	no	overflow,	and
is	simply	
x
	+	
y
.	This	is	shown	as	the	region	forming	a	sloping	plane
w
4
x
+
4
u
y
	

labeled	“Normal.”	When	
x
	+	
y
	≥	16,	the	addition	overflows,	having	the
effect	of	decrementing	the	sum	by	16.	This	is	shown	as	the	region
forming	a	sloping	plane	labeled	“Overflow.”
When	executing	C	programs,	overflows	are	not	signaled	as	errors.	At
times,	however,	we	might	wish	to	determine	whether	or	not	overflow	has
occurred.
Principle:
Detecting	overflow	of	unsigned	addition
For	
x
	and	
y
	in	the	range	
,	let	
.	Then
the	computation	of	
s
	overflowed	if	and	only	if	
s
	<	
x
	(or
equivalently,	
s
	<	
y
).
As	an	illustration,	in	our	earlier	example,	we	saw	that	
.	We	can
see	that	overflow	occurred,	since	5	<	9.
Derivation:
Detecting	overflow	of	unsigned	addition
0
≤
x
,
y
≤
U
M
a
x
w
s
 
≐
x
+
w
u
y
9
+
4
u
12
=
5

Observe	that	
,	and	hence	if	
s
	did	not	overflow,	we	will
surely	have	
s
	≥	
x.
	On	the	other	hand,	if	
s
	did	overflow,	we
have	
.	Given	that	
y
	<	2
,	we	have	
,	and
hence	
.
Practice	Problem	
2.27
	(solution	page	
152
)
Write	a	function	with	the	following	prototype:
This	function	should	return	1	if	arguments	
	and	
	can	be	added
without	causing	overflow.
Modular	addition	forms	a	mathematical	structure	known	as	an	
abelian
group
,	named	after	the	Norwegian	mathematician	Niels	Henrik	Abel
(1802–1829).	That	is,	it	is	commutative	(that's	where	the	“abelian”	part
comes	in)	and	associative;	it	has	an	identity	element	0,	and	every
element	has	an	additive	inverse.	Let	us	consider	the	set	of	
w
-bit	unsigned
numbers	with	addition	operation	
.	For	every	value	
x
,	there	must	be
some	value	
such	that	
.	This	additive	inverse
operation	can	be	characterized	as	follows:
x
+
y
≥
x
s
=
x
+
y
−
2
w
w
y
−
2
w
<
0
s
=
x
+
(
y
−
2
w
)
<
x
+
w
u
−
w
u
x
	
−
w
u
x
 
+
w
u
 
x
=
0

Principle:
Unsigned	negation
For	any	number	
x
	such	that	0	≤	
x
	<	2
,	its	
w
-bit	unsigned
negation	
is	given	by	the	following:
This	result	can	readily	be	derived	by	case	analysis:
Derivation:
Unsigned	negation
When	
x
	=	0,	the	additive	inverse	is	clearly	0.	For	
x
	>	0,
consider	the	value	2
	–	
x
.	Observe	that	this	number	is	in
the	range	
.	We	can	also	see	that	
.	Hence	it	is	the	inverse	of	
x
under	
.
w
−
w
u
 
x
	
−
w
u
x
=
{
x
,
x
=
0
2
w
−
x
,
x
>
0
(2.12)
w
0
<
2
w
−
x
<
2
w
(
x
+
2
w
−
x
)
 
mod
 
2
w
=
2
w
 
mod
 
2
w
=
0
+
w
u

Practice	Problem	
2.28
	(solution	page	
152
)
We	can	represent	a	bit	pattern	of	length	
w
	=	4	with	a	single	hex
digit.	For	an	unsigned	interpretation	of	these	digits,	use	
Equation
2.12
	to	fill	in	the	following	table	giving	the	values	and	the	bit
representations	(in	hex)	of	the	unsigned	additive	inverses	of	the
digits	shown.
x
Hex
Decimal
Decimal
Hex
0
___________
___________
___________
5
___________
___________
___________
8
___________
___________
___________
D
___________
___________
___________
F
___________
___________
___________
2.3.2	
Two's-Complement	Addition
With	two's-complement	addition,	we	must	decide	what	to	do	when	the
result	is	either	too	large	(positive)	or	too	small	(negative)	to	represent.
Given	integer	values	
x
	and	
y
	in	the	range	
,	their	sum
is	in	the	range	
,	potentially	requiring	
w
	+	1	bits	to
represent	exactly.	As	before,	we	avoid	ever-expanding	data	sizes	by
truncating	the	representation	to	
w
	bits.	The	result	is	not	as	familiar
–
4
u
 
x
−
2
w
−
1
≤
x
,
y
≤
2
w
−
1
−
1
−
2
w
≤
x
+
y
≤
2
w
−
1
−
2

mathematically	as	modular	addition,	however.	Let	us	define	
to	be
the	result	of	truncating	the	integer	sum	
x
	+	
y
	to	be	
w
	bits	long	and	then
viewing	the	result	as	a	two's-complement	number.
Principle:
Two's-complement	addition
For	integer	values	
x
	and	
y
	in	the	range	
This	principle	is	illustrated	in	
Figure	
2.24
,	where	the	sum	
x
	+	
y
	is
shown	on	the	left,	having	a	value	in	the	range	
,	and	the
result	of	truncating	the	sum	to	a	
w
-bit,	two's-complement	number	is
shown	on	the	right.	(The	labels	“Case	1”	to	“Case	4”	in	this	figure	are	for
the	case	analysis	of	the	formal	derivation	of	the	principle.)	When	the	sum
x
	+	
y
	exceeds	
TMax
	(
Case	
4
),	we	say	that	
positive	overflow
	has
occurred.	In	this	case,	the	effect	of	truncation	is	to	subtract	2
	from	the
sum.	When	the	sum	
x
	+	
y
	is	less	than	
TMin
	(Case	1),	we	say	that
negative	overflow
	has	occurred.	In	this	case,	the	effect	of	truncation	is	to
add	2
	to	the	sum.
x
+
w
t
 
y
	
−
2
w
−
1
≤
x
,
 
y
 
≤
 
2
w
−
1
−
1
:
x
+
w
t
y
=
{
x
+
y
−
2
w
,
2
w
−
1
≤
x
+
y
 
	
	
	
	
	
	
	
	
	
Positive
 
overflow
x
+
y
,
−
2
w
−
1
≤
x
+
y
<
2
w
−
1
 
	
	
Normal
x
+
y
+
2
w
,
x
+
y
<
−
2
w
−
1
 
	
	
	
	
Negative
 
overflow
(2.13)
−
2
w
≤
x
+
y
≤
2
w
−
2
w
w
w
w

The	
w
-bit	two's-complement	sum	of	two	numbers	has	the	exact	same	bit-
level	representation	as	the	unsigned	sum.	In	fact,	most	computers	use
the	same	machine	instruction	to	perform	either	unsigned	or	signed
addition.
Derivation:
Two's-complement	addition
Since	two's-complement	addition	has	the	exact	same	bit-
level	representation	as	unsigned	addition,	we	can
characterize	the	operation	
as	one	of	converting	its
arguments	to	unsigned,	performing	unsigned	addition,	and
then	converting	back	to	two's	complement:
Figure	
2.24	
Relation	between	integer
+
w
t
	

and	two's-complement	addition.
When	
x
	+	
y
	is	less	than	–2
,	there	is	a	negative	overflow.
When	it	is	greater	than	or	equal	to	2
,	there	is	a	positive
overflow.
By	
Equation	
2.6
,	we	can	write	
and	
as	
.	Using	the	property	that	
is
simply	addition	modulo	2
,	along	with	the	properties	of
modular	addition,	we	then	have
The	terms	
and	
drop	out	since	they	equal
0	modulo	2
.
To	better	understand	this	quantity,	let	us	define	
z
	as	the
integer	sum	
as	
mod	2
,	and	
z
″	as	
.	The	value	
z
″	is	equal	to	
.	We	can	divide	the
analysis	into	four	cases	as	illustrated	in	
Figure	
2.24
:
1
.	
.	Then	we	will	have	
.	This
gives	
.	Examining	
Equation
2.7
,	we	see	that	
z
′	is	in	the	range	such	that	
z
″	=	
z
′.
This	is	the	case	of	negative	overflow.	We	have
added	two	negative	numbers	
x
	and	
y
	(that's	the	only
w
–1
w
–1
x
+
w
t
y
=
U
2
T
w
(
T
2
U
w
(
x
)
+
w
u
T
2
U
w
(
y
)
)
(2.14)
T
2
U
w
(
x
)
 
as
 
x
w
−
1
2
w
+
x
T
2
U
w
(
y
)
	
y
w
−
1
2
w
+
y
+
w
u
	
w
x
+
w
t
y
=
U
2
T
w
(
T
2
U
w
(
x
)
+
w
u
T
2
U
w
(
y
)
)
 
=
U
2
T
w
[
(
x
w
−
1
2
w
+
x
+
y
w
−
1
2
w
+
y
)
 
mod
 
2
w
]
 
=
U
2
T
w
[
(
x
+
y
)
 
mod
 
2
w
]
x
w
−
1
2
w
	
y
w
−
1
2
w
	
w
z
≐
x
+
y
,
 
z
'
	
z
'
≐
z
	
w
z
″
≐
U
2
T
w
(
z
′
)
x
 
+
w
t
 
y
−
2
w
≤
z
<
−
2
w
−
1
z
′
=
z
+
2
w
0
≤
z
′
<
−
2
w
−
1
+
2
w
=
2
w
−
1
w
–1

way	we	can	have	
z
	<	–2
)	and	obtained	a
nonnegative	result	
.
2
.	
.	Then	we	will	again	have	
,
giving	
.	Examining
Equation	
2.7
,	we	see	that	
z
′	is	in	such	a	range
that	
,	and	therefore	
.
That	is,	our	two's-complement	sum	
z
″	equals	the
integer	sum	
x
	+	
y
.
3
.	
.	Then	we	will	have	
z
′	=	
z
,	giving	
,	and	hence	
z
″	=	
z
′	=	
z.
	Again,	the	two's-
complement	sum	
z
″	equals	the	integer	sum	
x
	+	
y.
4
.	
.	We	will	again	have	
z
′	=	
z
,	giving	
.	But	in	this	range	we	have	
,
giving	
.	This	is	the	case	of	positive
overflow.	We	have	added	two	positive	numbers	
x
and	
y
	(that's	the	only	way	we	can	have	
)	and
obtained	a	negative	result	
.
x
y
x
	+	
y
Case
–8
–5
–13
3
1
[1000]
[1011]
[10011]
[0011]
–8
–8
–16
0
1
[1000]
[1000]
[10000]
[0000]
w
–1
z
′
′
=
x
+
y
+
2
w
−
2
w
−
1
≤
z
<
0
z
′
=
z
+
2
w
−
2
w
−
1
+
2
w
=
2
w
−
1
≤
z
′
<
2
w
z
′
′
=
z
′
−
2
w
z
′
′
=
z
′
−
2
w
=
z
+
2
w
−
2
w
=
z
0
≤
z
<
2
w
−
1
0
≤
z
′
<
2
w
−
1
2
w
−
1
≤
z
<
2
w
2
w
−
1
≤
z
′
<
2
w
z
′
′
=
z
′
−
2
w
z
′
′
=
x
+
y
−
2
w
z
≥
2
w
−
1
z
′
′
=
x
+
y
−
2
w
x
+
4
t
y

–8
5
–3
–3
2
[1000]
[0101]
[11101]
[1101]
2
5
7
7
3
[0010]
[0101]
[00111]
[0111]
5
5
10
–6
4
[0101]
[0101]
[01010]
[1010]
Figure	
2.25	
Two's-complement	addition	examples.
The	bit-level	representation	of	the	4-bit	two's-complement	sum	can	be
obtained	by	performing	binary	addition	of	the	operands	and	truncating	the
result	to	4	bits.
As	illustrations	of	two's-complement	addition,	
Figure	
2.25
	shows	some
examples	when	
w
	=	4.	Each	example	is	labeled	by	the	case	to	which	it
corresponds	in	the	derivation	of	
Equation	
2.13
.	Note	that	2
	=	16,	and
hence	negative	overflow	yields	a	result	16	more	than	the	integer	sum,
and	positive	overflow	yields	a	result	16	less.	We	include	bit-level
representations	of	the	operands	and	the	result.	Observe	that	the	result
can	be	obtained	by	performing	binary	addition	of	the	operands	and
truncating	the	result	to	4	bits.
Figure	
2.26
	illustrates	two's-complement	addition	for	word	size	
w
	=	4.
The	operands	range	between	–8	and	7.	When	
x
	+	
y
	<	–8,	two's-
complement	addition	has	a	negative	overflow,	causing	the	sum	to	be
incremented	by	16.	When	–8	≤	
x
	+	
y
	<	8,	the	addition	yields	
x
	+	
y
.	When
x
	+	
y
	≥	8,	the	addition	has	a	positive	overflow,	causing	the	sum	to	be
4

decremented	by	16.	Each	of	these	three	ranges	forms	a	sloping	plane	in
the	figure.
Equation	
2.13
	also	lets	us	identify	the	cases	where	overflow	has
occurred:
Principle:
Detecting	overflow	in	two's-complement	addition
For	
x
	and	
y
	in	the	range	
,	let	
.
Then	the	computation	of	
s
	has	had	positive	overflow	if	and
only	if	
x
	>	0	and	
y
	>	0	but	
s
	≤	0.	The	computation	has	had
negative	overflow	if	and	only	if	
x
	<	0	and	
y
	<	0	but	
s
	≥	0.
Figure	
2.25
	shows	several	illustrations	of	this	principle	for	
w
	=	4.	The
first	entry	shows	a	case	of	negative	overflow,	where	two	negative
numbers	sum	to	a	positive	one.	The	final	entry	shows	a	case	of	positive
overflow,	where	two	positive	numbers	sum	to	a	negative	one.
T
M
i
n
w
<
x
,
 
y
≤
T
M
a
x
w
s
≐
x
+
w
t
 
y

Figure	
2.26	
Two's-complement	addition.
With	a	4-bit	word	size,	addition	can	have	a	negative	overflow	when	
x
	+	
y
<	–8	and	a	positive	overflow	when	
x
	+	
y
	≥	8.
Derivation:
Detecting	overflow	of	two's-complement	addition

Let	us	first	do	the	analysis	for	positive	overflow.	If	both	
x
	>	0
and	
y
	>	0	but	
s
	≤	0,	then	clearly	positive	overflow	has
occurred.	Conversely,	positive	overflow	requires	(1)	that	
x
	>
0	and	
y
	>	0	(otherwise,	
),	and	(2)	
s
	≤	0	(from
Equation	
2.13
.)	A	similar	set	of	arguments	holds	for
negative	overflow.
Practice	Problem	
2.29
	(solution	page	
152
)
Fill	in	the	following	table	in	the	style	of	
Figure	
2.25
.	Give	the
integer	values	of	the	5-bit	arguments,	the	values	of	both	their
integer	and	two's-complement	sums,	the	bit-level	representation	of
the	two's-complement	sum,	and	the	case	from	the	derivation	of
Equation	
2.13
.
x
y
x
	+	
y
Case
_____________
_____________
_____________
_____________
_____________
[10100]
[10001]
_____________
_____________
_____________
_____________
_____________
_____________
_____________
_____________
[11000]
[11000]
_____________
_____________
_____________
_____________
_____________
_____________
_____________
_____________
[10111]
[01000]
_____________
_____________
_____________
_____________
_____________
_____________
_____________
_____________
x
+
y
<
T
M
a
x
w
x
 
+
5
t
y

[00010]
[00101]
_____________
_____________
_____________
_____________
_____________
_____________
_____________
_____________
[01100]
[00100]
_____________
_____________
_____________
_____________
_____________
_____________
_____________
_____________
Practice	Problem	
2.30
	(solution	page	
153
)
Write	a	function	with	the	following	prototype:
This	function	should	return	1	if	arguments	
	and	
	can	be	added
without	causing	overflow.
Practice	Problem	
2.31
	(solution	page	
153
)
Your	coworker	gets	impatient	with	your	analysis	of	the	overflow
conditions	for	two's-complement	addition	and	presents	you	with
the	following	implementation	of	

You	look	at	the	code	and	laugh.	Explain	why.
Practice	Problem	
2.32
	(solution	page	
153
)
You	are	assigned	the	task	of	writing	code	for	a	function	
,
with	arguments	
	and	
,	that	will	return	1	if	computing	
	does
not	cause	overflow.	Having	just	written	the	code	for	
Problem
2.30
,	you	write	the	following:
For	what	values	of	
	and	
	will	this	function	give	incorrect	results?
Writing	a	correct	version	of	this	function	is	left	as	an	exercise
(
Problem	
2.74
).
2.3.3	
Two's-Complement	Negation

We	can	see	that	every	number	
x
	in	the	range	
has	an
additive	inverse	under	
,	which	we	denote	
as	follows:
Principle:
Two's-complement	negation
For	
x
	in	the	range	
,	its	two's-complement
negation	
is	given	by	the	formula
That	is,	for	
w
-bit,	two's-complement	addition,	
TMin
	is	its
own	additive	in-verse,	while	any	other	value	
x
	has	–
x
	as	its
additive	inverse.
Derivation:
Two's-complement	negation
Observe	that	
.	This
would	cause	negative	overflow,	and	hence
.	For	values	of	
x
	such	that	
x
T
M
i
n
w
≤
x
≤
T
M
a
x
w
	
−
w
t
 
−
w
t
 
x
	
T
M
i
n
w
≤
x
≤
T
M
a
x
w
−
w
t
 
x
	
−
w
t
x
=
{
T
M
i
n
w
,
x
=
T
M
i
n
w
−
x
x
>
T
M
i
n
w
(2.15)
w
T
M
i
n
w
+
T
M
i
n
w
=
−
2
ω
−
1
+
−
2
w
−
1
=
−
2
w
T
M
i
n
w
 
+
w
t
 
T
M
i
n
w
 
=
 
−
2
w
+
2
w
=
0

>	
TMin
,	the	value	–
x
	can	also	be	represented	as	a	
w
-bit,
two's-complement	number,	and	their	sum	will	be	–
x
	+	
x
	=	0.
Practice	Problem	
2.33
	(solution	page	
153
)
We	can	represent	a	bit	pattern	of	length	
w
	=	4	with	a	single	hex
digit.	For	a	two's-complement	interpretation	of	these	digits,	fill	in
the	following	table	to	determine	the	additive	inverses	of	the	digits
shown:
x
Hex
Decimal
Decimal
Hex
0
_________________
_________________
_________________
5
_________________
_________________
_________________
8
_________________
_________________
_________________
D
_________________
_________________
_________________
F
_________________
_________________
_________________
What	do	you	observe	about	the	bit	patterns	generated	by	two's-
complement	and	unsigned	(
Problem	
2.28
)	negation?
Web	Aside	DATA:TNEG	
it-level
w
−
4
t
 
x

representation	of	two's-complement
negation
There	are	several	clever	ways	to	determine	the	two's-complement
negation	of	a	value	represented	at	the	bit	level.	The	following	two
techniques	are	both	useful,	such	as	when	one	encounters	the
value	
	when	debugging	a	program,	and	they	lend
insight	into	the	nature	of	the	two's-complement	representation.
One	technique	for	performing	two's-complement	negation	at	the
bit	level	is	to	complement	the	bits	and	then	increment	the	result.	In
C,	we	can	state	that	for	any	integer	value	x,	computing	the
expressions	
	and	
	will	give	identical	results.
Here	are	some	examples	with	a	4-bit	word	size:
[0101]
5
[1010]
–6
[1011]
–5
[0111]
7
[1000]
–8
[1001]
–7
[1100]
–4
[0011]
3
[0100]
4
[0000]
0
[1111]
–1
[0000]
0
[1000]
–8
[0111]
7
[1000]
–8
For	our	earlier	example,	we	know	that	the	complement	of	
	is
	and	the	complement	of	
	is	
,	and	so	
	is	the
two's-complement	representation	of	–6.
x
→
~
x
→
i
n
c
r
(
~
x
→
)

A	second	way	to	perform	two's-complement	negation	of	a	number
x
	is	based	on	splitting	the	bit	vector	into	two	parts.	Let	
k
	be	the
position	of	the	rightmost	1,	so	the	bit-level	representation	of	
x
	has
the	form	
.	(This	is	possible	as	long	as
x
	≠	0.)	The	negation	is	then	written	in	binary	form	as	
.	That	is,	we	complement	each	bit	to	the	left
of	bit	position	
k
.
We	illustrate	this	idea	with	some	4-bit	numbers,	where	we
highlight	the	rightmost	pattern	1,	0,	...,	0	in	italics:
x
–
x
[1
100
]
–4
[0
100
]
4
[
1000
]
–8
[
1000
]
–8
[010
1
]
5
[101
1
]
–5
[011
1
]
7
[100
1
]
–7
2.3.4	
Unsigned	Multiplication
Integers	
x
	and	
y
	in	the	range	
can	be	represented	as	
w
-bit
unsigned	numbers,	but	their	product	
x
	·	
y
	can	range	between	0	and	
.	This	could	require	as	many	as	2
w
	bits	to	represent.
Instead,	unsigned	multiplication	in	C	is	defined	to	yield	the	
w
-bit	value
given	by	the	low-order	
w
	bits	of	the	2
w
-bit	integer	product.	Let	us	denote
this	value	as	
.
[
x
w
−
1
,
x
w
−
2
,
…
,
x
k
+
1
,
1
,
0
,
…
0
]
[
∼
x
w
−
1
,
∼
x
w
−
2
,
…
∼
x
k
+
1
,
1
,
0
,
…
,
0
]
0
≤
x
,
y
≤
2
w
−
1
	
(
2
w
−
1
)
2
=
2
2
w
−
2
w
+
1
+
1
x
 
 
*
w
u
 
y

Truncating	an	unsigned	number	to	
w
	bits	is	equivalent	to	computing	its
value	modulo	2
,	giving	the	following:
Principle:
Unsigned	multiplication
For	
x
	and	
y
	such	that	
:
w
0
≤
x
,
y
≤
U
M
a
x
w
x
∗
w
u
y
=
(
x
⋅
y
)
 
mod
 
2
w
(2.16)

2.3.5	
Two's-Complement
Multiplication
Integers	
x
	and	
y
	in	the	range	
can	be	represented	as
w
-bit	two's-complement	numbers,	but	their	product	
x
	·	
y
	can	range
between	
and	
.
This	could	require	as	many	as	2
w
	bits	to	represent	in	two's-complement
form.	Instead,	signed	multiplication	in	C	generally	is	performed	by
truncating	the	2
w
-bit	product	to	
w
	bits.	We	denote	this	value	as	
.
Truncating	a	two's-complement	number	to	
w
	bits	is	equivalent	to	first
computing	its	value	modulo	2
	and	then	converting	from	unsigned	to
two's	complement,	giving	the	following:
Principle:
Two's-complement	multiplication
For	
x
	and	
y
	such	that	
TMin
	≤	
x,	y
	≤	
TMax
:
−
2
w
−
1
≤
x
,
y
≤
2
w
−
1
−
1
	
−
2
w
−
1
⋅
(
2
w
−
1
−
1
)
=
−
2
2
w
−
2
+
2
w
−
1
	
−
2
w
−
1
⋅
−
2
w
−
1
=
2
2
w
−
2
x
 
 
*
w
t
 
y
w
w
w
x
∗
w
t
y
=
U
2
T
w
(
(
x
⋅
y
)
 
mod
 
2
w
)
(2.17)

We	claim	that	the	bit-level	representation	of	the	product	operation	is
identical	for	both	unsigned	and	two's-complement	multiplication,	as
stated	by	the	following	principle:
Principle:
Bit-level	equivalence	of	unsigned	and	two's-complement
multiplication
Let	
and	
be	bit	vectors	of	length	
w.
	Define	integers	
x
and	
y
	as	the	values	represented	by	these	bits	in	two's-
complement	form:	
and	
.	Define
nonnegative	integers	
x
′	and	
y
′	as	the	values	represented	by
these	bits	in	unsigned	form:	
and	
.	Then
As	illustrations,	
Figure	
2.27
	shows	the	results	of	multiplying	different
3-bit	numbers.	For	each	pair	of	bit-level	operands,	we	perform	both
unsigned	and	two's-complement	multiplication,	yielding	6-bit	products,
and	then	truncate	these	to	3	bits.	The	unsigned	truncated	product	always
equals	
x
	·	
y
	mod	8.	The	bit-level	representations	of	both	truncated
products	are	identical	for	both	unsigned	and	two's-complement
multiplication,	even	though	the	full	6-bit	representations	differ.
x
→
	
y
→
	
x
=
B
2
T
w
(
x
→
)
	
y
=
B
2
T
w
(
y
→
)
x
′
=
B
2
U
w
(
x
→
)
	
y
′
=
B
2
U
w
(
y
→
)
T
2
B
w
(
x
∗
w
t
y
)
=
U
2
B
w
(
x
′
∗
w
u
y
′
)

Mode
x
y
x
	·	
y
Truncated	
x
	·	
y
Unsigned
5
[101]
3
[011]
15
[001111]
7
[111]
Two's	complement
–3
[101]
3
[011]
–9
[110111]
–1
[111]
Unsigned	complement
4
[100]
7
[111]
28
[011100]
4
[100]
Two's	complement
–4
[100]
–1
[111]
4
[000100]
–4
[100]
Unsigned
3
[011]
3
[011]
9
[001001]
1
[001]
Two's	comp.
3
[011]
3
[011]
9
[001001]
1
[001]
Figure	
2.27	
Three-bit	unsigned	and	two's-complement	multiplication
examples.
Although	the	bit-level	representations	of	the	full	products	may	differ,
those	of	the	truncated	products	are	identical.
Derivation:
Bit-level	equivalence	of	unsigned	and	two's-complement
multiplication
From	
Equation	
2.6
,	we	have	
and	
.	Computing	the	product	of	these	values	modulo	2
gives	the	following:
x
′
=
x
+
x
w
−
1
2
w
	
y
′
=
y
+
y
w
−
1
2
w
w
(
x
′
⋅
y
′
)
 
mod
 
2
w
=
[
(
x
+
x
w
−
1
2
w
)
⋅
(
y
+
y
w
−
1
2
w
)
]
 
mod
 
2
w
 
=
[
x
⋅
y
+
(
x
w
−
1
y
+
y
w
−
1
x
)
2
w
+
x
w
−
1
2
2
w
]
 
mod
 
2
w
 
=
(
x
⋅
y
)
 
mod
 
2
w
(2.18)
w
2w

The	terms	with	weight	2
	and	2
	drop	out	due	to	the
modulus	operator.	By	
Equation	
2.17
,	we	have
.	We	can	apply	the	operation
T2U
	to	both	sides	to	get
Combining	this	result	with	
Equations	
2.16
	and	
2.18
shows	that	
.	We	can
then	apply	
U2B
	to	both	sides	to	get
Practice	Problem	
2.34
	(solution	page	
153
)
Fill	in	the	following	table	showing	the	results	of	multiplying	different
3-bit	numbers,	in	the	style	of	
Figure	
2.27
:
Mode
x
y
x
	·	
y
Unsigned
___________
[100]
___________
[101]
___________
___________
Two's
complement
___________
[100]
___________
[101]
___________
___________
Unsigned
___________
[010]
___________
[111]
___________
___________
Two's
complement
___________
[010]
___________
[111]
___________
___________
w
2w
x
 
*
w
t
 
y
=
U
2
T
w
(
(
x
⋅
y
)
 
mod
 
2
w
)
w
T
2
U
w
(
x
∗
w
t
y
)
=
T
2
U
w
(
U
2
T
w
(
(
x
⋅
y
)
 
mod
 
2
w
)
)
=
(
x
⋅
y
)
 
mod
 
2
w
T
2
U
w
(
x
 
*
w
t
y
)
=
(
x
′
⋅
y
′
)
 
mod
 
2
w
=
x
′
 
*
w
u
 
y
′
w
U
2
B
w
(
T
2
U
w
(
x
∗
w
t
y
)
)
=
T
2
B
w
(
x
∗
w
t
y
)
=
U
2
B
w
(
x
′
∗
w
t
y
′
)

Unsigned
___________
[110]
___________
[110]
___________
___________
Two's
complement
___________
[110]
___________
[110]
___________
___________
Practice	Problem	
2.35
	(solution	page	
154
)
You	are	given	the	assignment	to	develop	code	for	a	function
	that	will	determine	whether	two	arguments	can	be
multiplied	without	causing	overflow.	Here	is	your	solution:
You	test	this	code	for	a	number	of	values	of	
	and	
,	and	it	seems
to	work	properly.	Your	coworker	challenges	you,	saying,	“If	I	can't
use	subtraction	to	test	whether	addition	has	overflowed	(see
Problem	
2.31
),	then	how	can	you	use	division	to	test	whether
multiplication	has	overflowed?”
Devise	a	mathematical	justification	of	your	approach,	along	the
following	lines.	First,	argue	that	the	case	
x
	=	0	is	handled	correctly.
Otherwise,	consider	
w
-bit	numbers	
x
	(
x
	≠	0),	
y,	p
,	and	
q
,	where	
p
is	the	result	of	performing	two's-complement	multiplication	on	
x
and	
y
,	and	
q
	is	the	result	of	dividing	
p
	by	
x.

1
.	
Show	that	
x
	·	
y
,	the	integer	product	of	
x
	and	
y
,	can	be
written	in	the	form	
,	where	
t
	≠	0	if	and	only	if	the
computation	of	
p
	overflows.
2
.	
Show	that	
p
	can	be	written	in	the	form	
,	where	|
r
|	<
|
x
|.
3
.	
Show	that	
q
	=	
y
	if	and	only	if	
r
	=	
t
	=	0.
Practice	Problem	
2.36
	(solution	page	
154
)
For	the	case	where	data	type	
	has	32	bits,	devise	a	version	of
	(
Problem	
2.35
)	that	uses	the	64-bit	precision	of	data
type	
,	without	using	division.
Practice	Problem	
2.37
	(solution	page	
155
)
You	are	given	the	task	of	patching	the	vulnerability	in	the	XDR
code	shown	in	the	aside	on	page	100	for	the	case	where	both	data
types	
	and	
	are	32	bits.	You	decide	to	eliminate	the
possibility	of	the	multiplication	overflowing	by	computing	the
number	of	bytes	to	allocate	using	data	type	
.	You	replace
the	original	call	to	
	(line	9)	as	follows:
Aside	
Security	vulnerability	in	the
XDR	library
In	2002,	it	was	discovered	that	code	supplied	by	Sun
Microsystems	to	implement	the	XDR	library,	a	widely	used
facility	for	sharing	data	structures	between	programs,	had	a
x
⋅
y
=
p
+
t
2
w
p
=
x
⋅
q
+
r

security	vulnerability	arising	from	the	fact	that	multiplication
can	overflow	without	any	notice	being	given	to	the	program.
Code	similar	to	that	containing	the	vulnerability	is	shown
below:

The	function	
	is	designed	to	copy	
data	structures,	each	consisting	of	
	bytes	into	a
buffer	allocated	by	the	function	on	line	9.	The	number	of
bytes	required	is	computed	as	
Imagine,	however,	that	a	malicious	programmer	calls	this
function	with	
	being	1,048,577	(2
	+	1)	and
	being	4,096	(2
)	with	the	program	compiled	for
32	bits.	Then	the	multiplication	on	line	9	will	overflow,
causing	only	4096	bytes	to	be	allocated,	rather	than	the
4,294,971,392	bytes	required	to	hold	that	much	data.	The
loop	starting	at	line	15	will	attempt	to	copy	all	of	those
bytes,	overrunning	the	end	of	the	allocated	buffer,	and
therefore	corrupting	other	data	structures.	This	could	cause
the	program	to	crash	or	otherwise	misbehave.
The	Sun	code	was	used	by	almost	every	operating	system,
and	in	such	widely	used	programs	as	Internet	Explorer	and
the	Kerberos	authentication	system.	The	Computer
Emergency	Response	Team	(CERT),	an	organization	run
by	the	Carnegie	Mellon	Software	Engineering	Institute	to
track	security	vulnerabilities	and	breaches,	issued	advisory
“CA-2002-25,”	and	many	companies	rushed	to	patch	their
code.	Fortunately,	there	were	no	reported	security
breaches	caused	by	this	vulnerability.
A	similar	vulnerability	existed	in	many	implementations	of
the	library	function	
	These	have	since	been
patched.	Unfortunately,	many	programmers	call	allocation
20
12

functions,	such	as	
,	using	arithmetic	expressions	as
arguments,	without	checking	these	expressions	for
overflow.	Writing	a	reliable	version	of	
	is	left	as	an
exercise	(
Problem	
2.76
.)
Recall	that	the	argument	to	
	has	type	
.
A
.	
Does	your	code	provide	any	improvement	over	the	original?
B
.	
How	would	you	change	the	code	to	eliminate	the
vulnerability?
2.3.6	
Multiplying	by	Constants
Historically,	the	integer	multiply	instruction	on	many	machines	was	fairly
slow,	requiring	10	or	more	clock	cycles,	whereas	other	integer	operations
—such	as	addition,	subtraction,	bit-level	operations,	and	shifting—
required	only	1	clock	cycle.	Even	on	the	Intel	Core	i7	Haswell	we	use	as
our	reference	machine,	integer	multiply	requires	3	clock	cycles.	As	a
consequence,	one	important	optimization	used	by	compilers	is	to	attempt
to	replace	multiplications	by	constant	factors	with	combinations	of	shift
and	addition	operations.	We	will	first	consider	the	case	of	multiplying	by	a
power	of	2,	and	then	we	will	generalize	this	to	arbitrary	constants.

Principle:
Multiplication	by	a	power	of	2
Let	
x
	be	the	unsigned	integer	represented	by	bit	pattern	
.	Then	for	any	
k
	≥	0,	the	
w
	+	
k
-bit	unsigned
representation	of	
x
2
	is	given	by	
,
where	
k
	zeros	have	been	added	to	the	right.
So,	for	example,	11	can	be	represented	for	
w
	=	4	as	[1011].	Shifting	this
left	by	
k
	=	2	yields	the	6-bit	vector	[101100],	which	encodes	the	unsigned
number	11	·	4	=	44.
Derivation:
Multiplication	by	a	power	of	2
This	property	can	be	derived	using	
Equation	
2.1
:
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
k
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
,
0
,
…
,
0
]
B
2
U
w
+
k
(
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
,
0
,
…
,
0
]
)
=
∑
i
=
0
w
−
1
x
i
2
i
+
k
 
=
[
∑
i
=
0
w
−
1
x
i
2
i
]
⋅
2
k
 
=
x
2
k

When	shifting	left	by	
k
	for	a	fixed	word	size,	the	high-order	
k
	bits	are
discarded,	yielding
but	this	is	also	the	case	when	performing	multiplication	on	fixed-size
words.	We	can	therefore	see	that	shifting	a	value	left	is	equivalent	to
performing	unsigned	multiplication	by	a	power	of	2:
Principle:
Unsigned	multiplication	by	a	power	of	2
For	C	variables	
	and	
	with	unsigned	values	
x
	and	
k
,
such	that	0	≤	
k
	<	
w
,	the	C	expression	
	<<	
	yields	the
value	
.
Since	the	bit-level	operation	of	fixed-size	two's-complement	arithmetic	is
equivalent	to	that	for	unsigned	arithmetic,	we	can	make	a	similar
statement	about	the	relationship	between	left	shifts	and	multiplication	by
a	power	of	2	for	two's-complement	arithmetic:
Principle:
[
x
w
−
k
−
1
,
x
w
−
k
−
2
,
…
,
x
0
,
0
,
…
,
0
]
x
 
*
w
u
 
2
k

Two's-complement	multiplication	by	a	power	of	2
For	C	variables	
	and	
	with	two's-complement	value	
x
and	unsigned	value	
k
,	such	that	0	≤	
k
	<	
w
,	the	C	expression
	<<	
	yields	the	value	
.
Note	that	multiplying	by	a	power	of	2	can	cause	overflow	with	either
unsigned	or	two's-complement	arithmetic.	Our	result	shows	that	even
then	we	will	get	the	same	effect	by	shifting.	Returning	to	our	earlier
example,	we	shifted	the	4-bit	pattern	[1011]	(numeric	value	11)	left	by	two
positions	to	get	[101100]	(numeric	value	44).	Truncating	this	to	4	bits
gives	[1100]	(numeric	value	12	=	44	mod	16).
Given	that	integer	multiplication	is	more	costly	than	shifting	and	adding,
many	C	compilers	try	to	remove	many	cases	where	an	integer	is	being
multiplied	by	a	constant	with	combinations	of	shifting,	adding,	and
subtracting.	For	example,	suppose	a	program	contains	the	expression
.	Recognizing	that	14	=	2
	+	2
	+	2
,	the	compiler	can	rewrite	the
multiplication	as	(
)	+	(
)	+	(
),	replacing	one	multiplication
with	three	shifts	and	two	additions.	The	two	computations	will	yield	the
same	result,	regardless	of	whether	
	is	unsigned	or	two's	complement,
and	even	if	the	multiplication	would	cause	an	overflow.	Even	better,	the
compiler	can	also	use	the	property	14	=	2
	–	2
	to	rewrite	the
multiplication	as	(
)	–	(
),	requiring	only	two	shifts	and	a
subtraction.
x
 
*
w
t
 
2
k
3
2
1
4
1

Practice	Problem	
2.38
	(solution	page	
155
)
As	we	will	see	in	
Chapter	
3
,	the	
LEA
	
instruction	can	perform
computations	of	the	form	
,	where	
	is	either	0,	1,	2,	or
3,	and	
	is	either	0	or	some	program	value.	The	compiler	often
uses	this	instruction	to	perform	multiplications	by	constant	factors.
For	example,	we	can	compute	
.
Considering	cases	where	
	is	either	0	or	equal	to	
,	and	all
possible	values	of	
,	what	multiples	of	
	can	be	computed	with	a
single	
LEA
	
instruction?
Generalizing	from	our	example,	consider	the	task	of	generating	code	for
the	expression	
	*	
K
,	for	some	constant	
K.
	The	compiler	can	express	the
binary	representation	of	
K
	as	an	alternating	sequence	of	zeros	and	ones:
For	example,	14	can	be	written	as	[(0	...	0)(111)(0)].	Consider	a	run	of
ones	from	bit	position	
n
	down	to	bit	position	
m
	(
n
	≥	
m
).	(For	the	case	of
14,	we	have	
n
	=	3	and	
m
	=	1.)	We	can	compute	the	effect	of	these	bits	on
the	product	using	either	of	two	different	forms:
Form	A:	
Form	B:	
By	adding	together	the	results	for	each	run,	we	are	able	to	compute	
	*	
K
without	any	multiplications.	Of	course,	the	trade-off	between	using
combinations	of	shifting,	adding,	and	subtracting	versus	a	single
[
(
0
…
0
)
(
1
…
1
)
(
0
…
0
)
⋯
(
1
…
1
)
]

multiplication	instruction	depends	on	the	relative	speeds	of	these
instructions,	and	these	can	be	highly	machine	dependent.	Most	compilers
only	perform	this	optimization	when	a	small	number	of	shifts,	adds,	and
subtractions	suffice.
Practice	Problem	
2.39
	(solution	page	
156
)
How	could	we	modify	the	expression	for	form	B	for	the	case	where
bit	position	
n
	is	the	most	significant	bit?
Practice	Problem	
2.40
	(solution	page	
156
)
For	each	of	the	following	values	of	
K
,	find	ways	to	express	
	*	
K
using	only	the	specified	number	of	operations,	where	we	consider
both	additions	and	subtractions	to	have	comparable	cost.	You	may
need	to	use	some	tricks	beyond	the	simple	form	A	and	B	rules	we
have	considered	so	far.
K
Shifts
Add/Subs
Expression
6
2
1
__________
31
1
1
__________
–6
2
1
__________
55
2
2
__________
Practice	Problem	
2.41
	(solution	page	
156
)

For	a	run	of	ones	starting	at	bit	position	
n
	down	to	bit	position	
m
	(
n
≥	
m
),	we	saw	that	we	can	generate	two	forms	of	code,	A	and	B.
How	should	the	compiler	decide	which	form	to	use?
2.3.7	
Dividing	by	Powers	of	2
Integer	division	on	most	machines	is	even	slower	than	integer
multiplication—requiring	30	or	more	clock	cycles.	Dividing	by	a	power	of
2	can	also	be	performed
	(binary)
decimal
12,340/2
0
0011000000110100
12,340
12,340.0
1
0001100000011010
6,170
6,170.0
4
0000001100000011
771
771.25
8
0000000000110000
48
48.203125
Figure	
2.28	
Dividing	unsigned	numbers	by	powers	of	2.
The	examples	illustrate	how	performing	a	logical	right	shift	by	
	has	the
same	effect	as	dividing	by	
	and	then	rounding	toward	zero.
using	shift	operations,	but	we	use	a	right	shift	rather	than	a	left	shift.	The
two	different	right	shifts—logical	and	arithmetic—serve	this	purpose	for
unsigned	and	two's-complement	numbers,	respectively.

Integer	division	always	rounds	toward	zero.	To	define	this	precisely,	let	us
introduce	some	notation.	For	any	real	number	
a
,	define	
⌊
a
⌋
	to	be	the
unique	integer	
a
′	such	that	
.	As	examples,	
.	Similarly,	define	
⌈
a
⌉
	to	be	the	unique	integer	
a
′	such	that	
.	As	examples,	
,	and	
⌈
3
⌉
	=	3.	For	
x
	≥	0	and	
y
>	0,	integer	division	should	yield	
⌊
x
/
y
⌋
,	while	for	
x
	<	0	and	
y
	>	0,	it	should
yield	
⌈
x
/
y
⌉
.	That	is,	it	should	round	down	a	positive	result	but	round	up	a
negative	one.
The	case	for	using	shifts	with	unsigned	arithmetic	is	straightforward,	in
part	because	right	shifting	is	guaranteed	to	be	performed	logically	for
unsigned	values.
Principle:
Unsigned	division	by	a	power	of	2
For	C	variables	
	and	
	with	unsigned	values	
x
	and	
k
,
such	that	0	≤	
k
	<	
w
,	the	C	expression	
	yields	the
value	
⌊
x
/2
⌋
.
As	examples,	
Figure	
2.28
	shows	the	effects	of	performing	logical	right
shifts	on	a	16-bit	representation	of	12,340	to	perform	division	by	1,	2,	16,
and	256.	The	zeros	shifted	in	from	the	left	are	shown	in	italics.	We	also
show	the	result	we	would	obtain	if	we	did	these	divisions	with	real
a
′
≤
a
<
a
′
+
1
⌊
3.14
⌋
=
3
,
 
⌊
−
3.14
⌋
=
−
4
,
 
and
 
⌊
3
⌋
=
3
a
′
−
1
<
a
≤
a
′
⌈
3.14
⌉
=
4
,
⌈
−
3.14
⌉
=
−
3
k

arithmetic.	These	examples	show	that	the	result	of	shifting	consistently
rounds	toward	zero,	as	is	the	convention	for	integer	division.
Derivation:
Unsigned	division	by	a	power	of	2
Let	
x
	be	the	unsigned	integer	represented	by	bit	pattern	
,	and	let	
k
	be	in	the	range	0	≤	
k
	<	
w.
	Let	
x
′
be	the	unsigned	number	with	
w
	–	
k
-bit	representation	
,	and	let	
x
″	be	the	unsigned	number	with	
k
-
bit	representation	
.	We	can	therefore	see	that
,	and	that	
.	It	therefore	follows	that	
⌊
x
/2
⌋
=	
x
′.
Performing	a	logical	right	shift	of	bit	vector	
by	
k
	yields	the	bit	vector
	(binary)
decimal
–12340/2
0
–12,340
–12,340.0
1
–6,170
–6,170.0
4
–772
–771.25
8
–49
–48.203125
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
[
x
w
−
1
,
x
w
−
2
,
…
,
x
k
]
[
x
k
−
1
,
…
,
x
0
]
x
=
2
k
x
′
+
x
′
′
0
≤
x
′
′
<
2
k
k
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
	
[
0
,
…
,
0
,
x
w
−
1
,
x
w
−
2
,
…
,
x
k
]
k

Figure	
2.29	
Applying	arithmetic	right
shift.
The	examples	illustrate	that	arithmetic	right	shift	is	similar	to
division	by	a	power	of	2,	except	that	it	rounds	down	rather
than	toward	zero.
This	bit	vector	has	numeric	value	
x
′,	which	we	have	seen	is
the	value	that	would	result	by	computing	the	expression	
.
The	case	for	dividing	by	a	power	of	2	with	two's-complement	arithmetic	is
slightly	more	complex.	First,	the	shifting	should	be	performed	using	an
arithmetic
	right	shift,	to	ensure	that	negative	values	remain	negative.	Let
us	investigate	what	value	such	a	right	shift	would	produce.
Principle:
Two's-complement	division	by	a	power	of	2,	rounding	down
Let	C	variables	
	and	
	have	two's-complement	value	
x
and	unsigned	value	
k
,	respectively,	such	that	0	≤	
k
	<	
w.
	The
C	expression	
,	when	the	shift	is	performed
arithmetically,	yields	the	value	
⌊
x
/2
⌋
.
k

For	
x
	≥	0,	variable	
	has	0	as	the	most	significant	bit,	and	so	the	effect	of
an	arithmetic	shift	is	the	same	as	for	a	logical	right	shift.	Thus,	an
arithmetic	right	shift	by	
k
	is	the	same	as	division	by	2
	for	a	nonnegative
number.	As	an	example	of	a	negative	number,	
Figure	
2.29
	shows	the
effect	of	applying	arithmetic	right	shift	to	a	16-bit	representation	of	–
12,340	for	different	shift	amounts.	For	the	case	when	no	rounding	is
required	(
k
	=	1),	the	result	will	be	
x
/2
.	When	rounding	is	required,	shifting
causes	the	result	to	be	rounded	downward.	For	example,	the	shifting
right	by	four	has	the	effect	of	rounding	–771.25	down	to	–772.	We	will
need	to	adjust	our	strategy	to	handle	division	for	negative	values	of	
x.
Derivation:
Two's-complement	division	by	a	power	of	2,	rounding	down
Let	
x
	be	the	two's-complement	integer	represented	by	bit
pattern	
,	and	let	
k
	be	in	the	range	0	≤	
k
	<
w.
	Let	
x
′	be	the	two's-complement	number	represented	by
the	
w
	–	
k
	bits	
,	and	let	
x
″	be	the	
unsigned
number	represented	by	the	low-order	
k
	bits	
.	By
a	similar	analysis	as	the	unsigned	case,	we	have	
and	
,	giving	
x
′	=	
⌊
x
/2
⌋
.	Furthermore,	observe	that
shifting	bit	vector	
right	
arithmetically
	by	
k
yields	the	bit	vector
k
k
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
[
x
w
−
1
,
x
w
−
2
,
…
,
x
k
]
[
x
k
−
1
,
…
,
x
0
]
x
=
2
k
x
′
+
x
′
′
0
≤
x
′
′
<
2
k
k
[
x
w
−
1
,
x
w
−
2
,
…
,
x
0
]
	

which	is	the	sign	extension	from	
w
	–	
k
	bits	to	
w
	bits	of	
.	Thus,	this	shifted	bit	vector	is	the	two's-
complement	representation	of	
⌊
x
/2
⌋
.
Bias
–12,340	+	bias	(binary)
	(binary)
Decimal
–12,340/2
0
0
–12,340
–12,340.0
1
1
–6,170
–6,170.0
4
15
–771
–771.25
8
255
–48
–48.203125
Figure	
2.30	
Dividing	two's-complement	numbers	by	powers	of	2.
By	adding	a	bias	before	the	right	shift,	the	result	is	rounded	toward	zero.
We	can	correct	for	the	improper	rounding	that	occurs	when	a	negative
number	is	shifted	right	by	“biasing”	the	value	before	shifting.
Principle:
Two's-complement	division	by	a	power	of	2,	rounding	up
[
x
w
−
1
,
…
,
x
w
−
1
,
x
w
−
1
,
x
w
−
2
,
…
,
x
k
]
[
x
w
−
1
,
x
w
−
2
,
…
,
x
k
]
k
k

Let	C	variables	
	and	
	have	two's-complement	value	
x
and	unsigned	value	
k
,	respectively,	such	that	0	≤	
k
	<	
w.
	The
C	expression	
,	when	the	shift	is
performed	arithmetically,	yields	the	value	
⌈
x
/2
⌉
.
Figure	
2.30
	demonstrates	how	adding	the	appropriate	bias	before
performing	the	arithmetic	right	shift	causes	the	result	to	be	correctly
rounded.	In	the	third	column,	we	show	the	result	of	adding	the	bias	value
to	–12,340,	with	the	lower	
k
	bits	(those	that	will	be	shifted	off	to	the	right)
shown	in	italics.	We	can	see	that	the	bits	to	the	left	of	these	may	or	may
not	be	incremented.	For	the	case	where	no	rounding	is	required	(
k
	=	1),
adding	the	bias	only	affects	bits	that	are	shifted	off.	For	the	cases	where
rounding	is	required,	adding	the	bias	causes	the	upper	bits	to	be
incremented,	so	that	the	result	will	be	rounded	toward	zero.
The	biasing	technique	exploits	the	property	that	
⌈
x/y
⌉
	=	
⌊
(
x
	+	
y
	–1)/
y
⌋
	for
integers	
x
	and	
y
	such	that	
y
	>	0.	As	examples,	when	
x
	=	–30	and	
y
	=	4,
we	have	
x
	+	
y
	–	1	=	–27	and	
⌈
–30/4
⌉
	=	–7	=	
⌊
–27/4
⌋
.	When	
x
	=	–32	and	
y
=	4,	we	have	
x
	+	
y
	–	1	=	–29	and	
⌈
–32/4
⌉
	=	–8	=	
⌊
–29/4
⌋
.
Derivation:
Two's-complement	division	by	a	power	of	2,	rounding	up
k

To	see	that	
⌈
x/y
⌉
	=	
⌊
(
x
	+	
y
	–	1)/
y
⌋
,	suppose	that	
x
	=	
qy
	+	
r
,
where	0	≤	
r
	<	
y
,	giving	(
x
	+	
y
	–	1)/
y
	=	
q
	+	(
r
	+	
y
	–	1)/
y
,	and
so	
⌊
(
x
	+	
y
	–	1)/
y
⌋
	=	
q
	+	[(
r
	+	
y
	–	1)/
y
⌋
.	The	latter	term	will
equal	0	when	
r
	=	0	and	1	when	
r
	>	0.	That	is,	by	adding	a
bias	of	
y
	–	1	to	
x
	and	then	rounding	the	division	downward,
we	will	get	
q
	when	
y
	divides	
x
	and	
q
	+	1	otherwise.
Returning	to	the	case	where	
y
	=	2
,	the	C	expression	
	—	1	yields	the	value	
x
	+	2
	–	1.	Shifting	this	right
arithmetically	by	
k
	therefore	yields	
⌈
x
/2
⌉
.
These	analyses	show	that	for	a	two's-complement	machine	using
arithmetic	right	shifts,	the	C	expression
will	compute	the	value	
x
/2
.
Practice	Problem	
2.42
	(solution	page	
156
)
Write	a	function	
	that	returns	the	value	
	for	integer
argument	
.	Your	function	should	not	use	division,	modulus,
multiplication,	any	conditionals	(
),	any	comparison
operators	(e.g.,	<,	>,	or	==),	or	any	loops.	You	may	assume	that
k
k
k
k

data	type	
	is	32	bits	long	and	uses	a	two's-complement
representation,	and	that	right	shifts	are	performed	arithmetically.
We	now	see	that	division	by	a	power	of	2	can	be	implemented	using
logical	or	arithmetic	right	shifts.	This	is	precisely	the	reason	the	two	types
of	right	shifts	are	available	on	most	machines.	Unfortunately,	this
approach	does	not	generalize	to	division	by	arbitrary	constants.	Unlike
multiplication,	we	cannot	express	division	by	arbitrary	constants	
K
	in
terms	of	division	by	powers	of	2.
Practice	Problem	
2.43
	(solution	page	
157
)
In	the	following	code,	we	have	omitted	the	definitions	of	constants
	and	
We	compiled	this	code	for	particular	values	of	
	and	
.	The
compiler	optimized	the	multiplication	and	division	using	the
methods	we	have	discussed.	The	following	is	a	translation	of	the
generated	machine	code	back	into	C:

What	are	the	values	of	
	and	
?
2.3.8	
Final	Thoughts	on	Integer
Arithmetic
As	we	have	seen,	the	“integer”	arithmetic	performed	by	computers	is
really	a	form	of	modular	arithmetic.	The	finite	word	size	used	to	represent
numbers	
limits	the	range	of	possible	values,	and	the	resulting	operations
can	overflow.	We	have	also	seen	that	the	two's-complement
representation	provides	a	clever	way	to	represent	both	negative	and
positive	values,	while	using	the	same	bit-level	implementations	as	are
used	to	perform	unsigned	arithmetic—operations	such	as	addition,
subtraction,	multiplication,	and	even	division	have	either	identical	or	very
similar	bit-level	behaviors,	whether	the	operands	are	in	unsigned	or
two's-complement	form.

We	have	seen	that	some	of	the	conventions	in	the	C	language	can	yield
some	surprising	results,	and	these	can	be	sources	of	bugs	that	are	hard
to	recognize	or	understand.	We	have	especially	seen	that	the	unsigned
data	type,	while	conceptually	straightforward,	can	lead	to	behaviors	that
even	experienced	programmers	do	not	expect.	We	have	also	seen	that
this	data	type	can	arise	in	unexpected	ways—for	example,	when	writing
integer	constants	and	when	invoking	library	routines.
Practice	Problem	
2.44
	(solution	page	
157
)
Assume	data	type	
	is	32	bits	long	and	uses	a	two's-
complement	representation	for	signed	values.	Right	shifts	are
performed	arithmetically	for	signed	values	and	logically	for
unsigned	values.	The	variables	are	declared	and	initialized	as
follows:
For	each	of	the	following	C	expressions,	either	(1)	argue	that	it	is
true	(evaluates	to	1)	for	all	values	of	
	and	
,	or	(2)	give	values	of
	and	
	for	which	it	is	false	(evaluates	to	0):
A
.	
B
.	

C
.	
D
.	
E
.	
F
.	
G
.	

2.4	
Floating	Point
A	floating-point	representation	encodes	rational	numbers	of	the	form	
V
	=
x
	×	2
.
	It	is	useful	for	performing	computations	involving	very	large
numbers	(|
V
|	
≫
	0),
Aside	
The	IEEE
The	Institute	of	Electrical	and	Electronics	Engineers	(IEEE—
pronounced	“eye-triple-ee”)	is	a	professional	society	that
encompasses	all	of	electronic	and	computer	technology.	It
publishes	journals,	sponsors	conferences,	and	sets	up
committees	to	define	standards	on	topics	ranging	from	power
transmission	to	software	engineering.	Another	example	of	an
IEEE	standard	is	the	802.11	standard	for	wireless	networking.
numbers	very	close	to	0	(|
V
|	
≪
	1),	and	more	generally	as	an
approximation	to	real	arithmetic.
Up	until	the	1980s,	every	computer	manufacturer	devised	its	own
conventions	for	how	floating-point	numbers	were	represented	and	the
details	of	the	operations	performed	on	them.	In	addition,	they	often	did
not	worry	too	much	about	the	accuracy	of	the	operations,	viewing	speed
and	ease	of	implementation	as	being	more	critical	than	numerical
precision.
All	of	this	changed	around	1985	with	the	advent	of	IEEE	Standard	754,	a
carefully	crafted	standard	for	representing	floating-point	numbers	and	the
y

operations	performed	on	them.	This	effort	started	in	1976	under	Intel's
sponsorship	with	the	design	of	the	8087,	a	chip	that	provided	floating-
point	support	for	the	8086	processor.	Intel	hired	William	Kahan,	a
professor	at	the	University	of	California,	Berkeley,	as	a	consultant	to	help
design	a	floating-point	standard	for	its	future	processors.	They	allowed
Kahan	to	join	forces	with	a	committee	generating	an	industry-wide
standard	under	the	auspices	of	the	Institute	of	Electrical	and	Electronics
Engineers	(IEEE).	The	committee	ultimately	adopted	a	standard	close	to
the	one	Kahan	had	devised	for	Intel.	Nowadays,	virtually	all	computers
support	what	has	become	known	as	
IEEE	floating	point.
	This	has	greatly
improved	the	portability	of	scientific	application	programs	across	different
machines.
In	this	section,	we	will	see	how	numbers	are	represented	in	the	IEEE
floating-point	format.	We	will	also	explore	issues	of	
rounding
,	when	a
number	cannot	be	represented	exactly	in	the	format	and	hence	must	be
adjusted	upward	or	downward.	We	will	then	explore	the	mathematical
properties	of	addition,	multiplication,	and	relational	operators.	Many
programmers	consider	floating	point	to	be	at	best	uninteresting	and	at
worst	arcane	and	incomprehensible.	We	will	see	that	since	the	IEEE
format	is	based	on	a	small	and	consistent	set	of	principles,	it	is	really
quite	elegant	and	understandable.
2.4.1	
Fractional	Binary	Numbers
A	first	step	in	understanding	floating-point	numbers	is	to	consider	binary
numbers	having	fractional	values.	Let	us	first	examine	the	more	familiar
decimal	notation.	Decimal	notation	uses	a	representation	of	the	form

Figure	
2.31	
Fractional	binary	representation.
Digits	to	the	left	of	the	binary	point	have	weights	of	the	form	2
,	while
those	to	the	right	have	weights	of	the	form	1/2
.
where	each	decimal	digit	
d
	ranges	between	0	and	9.	This	notation
represents	a	value	
d
	defined	as
The	weighting	of	the	digits	is	defined	relative	to	the	decimal	point	symbol
(‘.'),	meaning	that	digits	to	the	left	are	weighted	by	nonnegative	powers	of
10,	giving	integral	values,	while	digits	to	the	right	are	weighted	by
negative	powers	of	10,	giving	fractional	values.	For	example,	12.34
represents	the	number	
.
d
m
d
m
−
1
…
d
1
d
0
.
 
d
−
1
 
d
−
2
…
d
−
n
i
i
i
d
=
∑
i
=
−
n
m
10
i
×
d
i
10
1
×
10
1
+
2
×
10
0
+
3
×
10
−
1
+
4
×
10
−
2
=
12
34
100

By	analogy,	consider	a	notation	of	the	form
where	each	binary	digit,	or	bit,	
b
	ranges	between	0	and	1,	as	is	illustrated
in	
Figure	
2.31
.	This	notation	represents	a	number	
b
	defined	as
The	symbol	‘.’	now	becomes	a	
binary	point
,	with	bits	on	the	left	being
weighted	by	nonnegative	powers	of	2,	and	those	on	the	right	being
weighted	by	negative	powers	of	2.	For	example,	101.11
	represents	the
number	
.
One	can	readily	see	from	
Equation	
2.19
	that	shifting	the	binary	point
one	position	to	the	left	has	the	effect	of	dividing	the	number	by	2.	For
example,	while	101.11
	represents	the	number	
,	10.111
	represents
the	number	
.	
Similarly,	shifting	the	binary	point	one
position	to	the	right	has	the	effect	of	multiplying	the	number	by	2.	For
example,	1011.1
	represents	the	number	
.
Note	that	numbers	of	the	form	0.11	·	·	·	1
	represent	numbers	just	below
1.	For	example,	0.111111
	represents	
.	We	will	use	the	shorthand
notation	1.0	—	
∊
	
to	represent	such	values.
Assuming	we	consider	only	finite-length	encodings,	decimal	notation
cannot	represent	numbers	such	as	
	and	
	exactly.	Similarly,	fractional
binary	notation	can	only	represent	numbers	that	can	be	written	
x
	×	2
.
Other	values	can	only	be	approximated.	For	example,	the	number	
	can
b
m
 
b
m
−
1
…
 
b
1
 
b
0
 
.
 
b
−
1
 
b
−
2
…
 
b
−
n
+
1
 
b
−
n
i
b
=
∑
i
=
−
n
m
2
i
×
b
i
(2.19)
2
1
×
2
2
+
0
×
2
1
+
1
×
2
0
+
1
×
2
−
1
+
1
×
2
−
2
=
4
+
0
+
1
+
1
2
+
1
4
=
5
3
4
2
5
3
4
2
2
+
0
+
1
2
+
1
4
+
1
8
=
2
7
8
2
8
+
0
+
2
+
1
+
1
2
=
11
1
2
2
2
63
64
1
3
5
7
y
1
5

be	represented	exactly	as	the	fractional	decimal	number	0.20.	As	a
fractional	binary	number,	however,	we	cannot	represent	it	exactly	and
instead	must	approximate	it	with	increasing	accuracy	by	lengthening	the
binary	representation:
Representation
Value
Decimal
0.0
0.0
0.01
0.25
0.010
0.25
0.0011
0.1875
0.00110
0.1875
0.001101
0.203125
0.0011010
0.203125
0.00110011
0.19921875
Practice	Problem	
2.45
	(solution	page	
157
)
Fill	in	the	missing	information	in	the	following	table:
Fractional	value
Binary	representation
Decimal	representation
0.001
0.125
__________
__________
2
0
2
10
2
1
4
10
2
2
8
10
2
3
16
10
2
6
32
10
2
13
64
10
2
26
128
10
2
51
256
10
1
8
3
4

__________
__________
__________
10.1011
__________
__________
1.001
__________
__________
__________
5.875
__________
__________
3.1875
Practice	Problem	
2.46
	(solution	page	
158
)
The	imprecision	of	floating-point	arithmetic	can	have	disastrous
effects.	On	February	25,	1991,	during	the	first	Gulf	War,	an
American	Patriot	Missile	battery	in	Dharan,	Saudi	Arabia,	failed	to
intercept	an	incoming	Iraqi	Scud	missile.	The	Scud	struck	an
American	Army	barracks	and	killed	28	soldiers.	The	US	General
Accounting	Office	(GAO)	conducted	a	detailed	analysis	of	the
failure	[
76
]	and	determined	that	the	underlying	cause	was	an
imprecision	in	a	numeric	calculation.	In	this	exercise,	you	will
reproduce	part	of	the	GAO's	analysis.
The	Patriot	system	contains	an	internal	clock,	implemented	as	a
counter	that	is	incremented	every	0.1	seconds.	To	determine	the
time	in	seconds,	the	program	would	multiply	the	value	of	this
counter	by	a	24-bit	quantity	that	was	a	fractional	binary
approximation	to	
.	In	particular,	the	binary	representation	of
is	the	nonterminating	sequence	0.000110011[0011]...
,	where
the	portion	in	brackets	is	repeated	indefinitely.	The	program
approximated	0.1,	as	a	value	
x
,	by	considering	just	the	first	23	bits
of	the	sequence	to	the	right	of	the	binary	point:	
x
	=
0.00011001100110011001100.	(See	
Problem	
2.51
	for	a
5
16
1
10
1
10
	
2

discussion	of	how	they	could	have	approximated	0.1	more
precisely.)
A
.	
What	is	the	binary	representation	of	0.1	–	
x
?
B
.	
What	is	the	approximate	decimal	value	of	0.1	–	
x
?
C
.	
The	clock	starts	at	0	when	the	system	is	first	powered	up
and	keeps	counting	up	from	there.	In	this	case,	the	system
had	been	running	for	around	100	hours.	What	was	the
difference	between	the	actual	time	and	the	time	computed
by	the	software?
D
.	
The	system	predicts	where	an	incoming	missile	will	appear
based	on	its	velocity	and	the	time	of	the	last	radar
detection.	Given	that	a	Scud	travels	at	around	2,000	meters
per	second,	how	far	off	was	its	prediction?
Normally,	a	slight	error	in	the	absolute	time	reported	by	a	clock
reading	would	not	affect	a	tracking	computation.	Instead,	it	should
depend	on	the	relative	time	between	two	successive	readings.	The
problem	was	that	the	Patriot	software	had	been	upgraded	to	use	a
more	accurate	function	for	reading	time,	but	not	all	of	the	function
calls	had	been	replaced	by	the	new	code.	As	a	result,	the	tracking
software	used	the	accurate	time	for	one	reading	and	the
inaccurate	time	for	the	other	[
103
].
2.4.2	
IEEE	Floating-Point
Representation

Positional	notation	such	as	considered	in	the	previous	section	would	not
be	efficient	for	representing	very	large	numbers.	For	example,	the
representation	of	5	×	2
	would	consist	of	the	bit	pattern	101	followed	by
100	zeros.	Instead,	we	would	like	to	represent	numbers	in	a	form	
x
	×	2
by	giving	the	values	of	
x
	and	
y.
The	IEEE	floating-point	standard	represents	a	number	in	a	form	
V
	=	(–1)
×	
M
	×	2
:
The	
sign	s
	determines	whether	the	number	is	negative	(
s
	=	1)	or
positive	(
s
	=	0),	where	the	interpretation	of	the	sign	bit	for	numeric
value	0	is	handled	as	a	special	case.
The	
significand	M
	is	a	fractional	binary	number	that	ranges	either
between	1	and	2	–	
∊
	or	between	0	and	1	–	
∊
.
The	
exponent	E
	weights	the	value	by	a	(possibly	negative)	power	of
2.
Figure	
2.32	
Standard	floating-point	formats.
Floating-point	numbers	are	represented	by	three	fields.	For	the	two	most
common	formats,	these	are	packed	in	32-bit	(single-precision)	or	64-bit
100
y
s
E

(double-precision)	words.
The	bit	representation	of	a	floating-point	number	is	divided	into	three
fields	to	encode	these	values:
The	single	sign	bit	
	directly	encodes	the	sign	
s.
The	
k
-bit	exponent	field	
	=	
e
	·	·	·	
e
e
	encodes	the	exponent	
E.
The	
n
-bit	fraction	field	
	=	
f
	·	·	·	
f
f
	encodes	the	significand	
M
,
but	the	value	encoded	also	depends	on	whether	or	not	the	exponent
field	equals	0.
Figure	
2.32
	shows	the	packing	of	these	three	fields	into	words	for	the
two	most	common	formats.	In	the	single-precision	floating-point	format	(a
	in	C),	fields	
,	and	
	are	1,	
k
	=	8,	and	
n
	=	23	bits	each,
yielding	a	32-bit	representation.	In	the	double-precision	floating-point
format	(a	
	in	C),	fields	
,	and	
	are	1,	
k
	=	11,	and	
n
	=	52
bits	each,	yielding	a	64-bit	representation.
The	value	encoded	by	a	given	bit	representation	can	be	divided	into	three
different	cases	(the	latter	having	two	variants),	depending	on	the	value	of
exp.	These	are	illustrated	in	
Figure	
2.33
	for	the	single-precision
format.
Case	
1
:	Normalized	Values
This	is	the	most	common	case.	It	occurs	when	the	bit	pattern	of	
	is
neither	all	zeros	(numeric	value	0)	nor	all	ones	(numeric	value	255	for
single	precision,	2047	for	double).	In	this	case,	the	exponent	field	is
interpreted	as	representing	a	signed	integer	in	
biased
	form.	That	is,	the
k
–1
1
0
n
–1
1
0

exponent	value	is	
E
	=	
e
	–	
Bias
,	where	
e
	is	the	unsigned	number	having
bit	representation	
e
	·	·	·	
e
e
	and	
Bias
	is	a	bias	value	equal	to	2
	–	1
(127	for	single	precision	and	1023	for	double).	This	yields	exponent
ranges	from	–126	to	+127	for	single	precision	and	–1022	to	+1023	for
double	precision.
The	fraction	field	
	is	interpreted	as	representing	the	fractional	value	
f
,
where	0	≤	
f
	<	1,	having	binary	representation	0.	
f
	·	·	·	
f
f
,	that	is,	with
the
Aside	
Why	set	the	bias	this	way	for
denormalized	values?
Having	the	exponent	value	be	1	–	
Bias
	rather	than	simply	–
Bias
might	seem	counterintuitive.	We	will	see	shortly	that	it	provides	for
smooth	transition	from	denormalized	to	normalized	values.
Figure	
2.33	
Categories	of	single-precision	floating-point	values.
k
–1
1
0
k
-1
n
–1
1
0

The	value	of	the	exponent	determines	whether	the	number	is	(1)
normalized,	(2)	denormalized,	or	(3)	a	special	value.
binary	point	to	the	left	of	the	most	significant	bit.	The	significand	is
defined	to	be	
M
	=	1	+	
f.
	This	is	sometimes	called	an	
implied	leading	1
representation,	because	we	can	view	
M
	to	be	the	number	with	binary
representation	1.	
.	This	representation	is	a	trick	for	getting
an	additional	bit	of	precision	for	free,	since	we	can	always	adjust	the
exponent	
E
	so	that	significand	
M
	is	in	the	range	1	≤	
M
	<	2	(assuming
there	is	no	overflow).	We	therefore	do	not	need	to	explicitly	represent	the
leading	bit,	since	it	always	equals	1.
Case	
2
:	Denormalized	Values
When	the	exponent	field	is	all	zeros,	the	represented	number	is	in
denormalized
	form.	In	this	case,	the	exponent	value	is	
E
	=	1	–	
Bias
,	and
the	significand	value	is	
M
	=	
f
,	that	is,	the	value	of	the	fraction	field	without
an	implied	leading	1.
Denormalized	numbers	serve	two	purposes.	First,	they	provide	a	way	to
represent	numeric	value	0,	since	with	a	normalized	number	we	must
always	have	
M
	≥	1,	and	hence	we	cannot	represent	0.	In	fact,	the
floating-point	representation	of	+0.0	has	a	bit	pattern	of	all	zeros:	the	sign
bit	is	0,	the	exponent	field	is	all	zeros	(indicating	a	denormalized	value),
and	the	fraction	field	is	all	zeros,	giving	
M
	=	
f
	=	0.	Curiously,	when	the
sign	bit	is	1,	but	the	other	fields	are	all	zeros,	we	get	the	value	–0.0.	With
IEEE	floating-point	format,	the	values	–0.0	and	+0.0	are	considered
different	in	some	ways	and	the	same	in	others.
f
n
−
1
f
n
−
2
⋯
f
0

A	second	function	of	denormalized	numbers	is	to	represent	numbers	that
are	very	close	to	0.0.	They	provide	a	property	known	as	
gradual
underflow
	in	which	possible	numeric	values	are	spaced	evenly	near	0.0.
Case	
3
:	Special	Values
A	final	category	of	values	occurs	when	the	exponent	field	is	all	ones.
When	the	fraction	field	is	all	zeros,	the	resulting	values	represent	infinity,
either	+∞	when	
s
	=	0	or	-∞	when	
s
	=	1.	Infinity	can	represent	results	that
overflow
,	as	when	we	multiply	two	very	large	numbers,	or	when	we	divide
by	zero.	When	the	fraction	field	is	nonzero,	the	resulting	value	is	called	a
“NaN
,”	short	for	“not	a	number.”	Such	values	are	returned	as	the	result	of
an	operation	where	the	result	cannot	be	given	as	a	real	number	or	as
infinity,	as	when	computing	
or	∞	–	∞.	They	can	also	be	useful	in	some
applications	for	representing	uninitialized	data.
2.4.3	
Example	Numbers
Figure	
2.34
	shows	the	set	of	values	that	can	be	represented	in	a
hypothetical	6-bit	format	having	
k
	=	3	exponent	bits	and	
n
	=	2	fraction
bits.	The	bias	is	2
	–	1	=	3.	Part	(a)	of	the	figure	shows	all	representable
values	(other	than	
NaN
).	The	two	infinities	are	at	the	extreme	ends.	The
normalized	numbers	with	maximum	magnitude	are	±14.	The
denormalized	numbers	are	clustered	around	0.	These	can	be	seen	more
clearly	in	part	(b)	of	the	figure,	where	we	show	just	the	numbers	between
–1.0	and	+1.0.	The	two	zeros	are	special	cases	of	denormalized
−
1
	
3–1

numbers.	Observe	that	the	representable	numbers	are	not	uniformly
distributed—they	are	denser	nearer	the	origin.
Figure	
2.35
	shows	some	examples	for	a	hypothetical	8-bit	floating-
point	format	having	
k
	=	4	exponent	bits	and	
n
	=	3	fraction	bits.	The	bias	is
2
	–	1	=	7.	The	figure	is	divided	into	three	regions	representing	the	three
classes	of	numbers.	The	different	columns	show	how	the	exponent	field
encodes	the	exponent	
E
,	while	the	fraction	field	encodes	the	significand
M
,	and	together	they	form	the
Figure	
2.34	
Representable	values	for	6-bit	floating-point	format.
There	are	
k
	=	3	exponent	bits	and	
n
	=	2	fraction	bits.	The	bias	is	3.
Exponent
Fraction
Value
Description
Bit
representation
e
E
2
f
M
2
	×
M
V
Decimal
Zero
0
–
6
0
0.0
Smallest
0
–
0.001953
4–1
E
E
1
64
0
8
0
8
0
512
1
64
1
8
1
8
1
512
1
512

positive
6
0
–
6
0.003906
0
–
6
0.005859
⋮
Largest
denormalized
0
–
6
0.013672
Smallest
normalized
1
–
6
0.015625
1
–
6
0.017578
⋮
6
–
1
0.875
6
–
1
0.9375
One
7
0
1
1
1.0
7
0
1
1.125
7
0
1
1.25
⋮
14
7
128
224
224.0
1
64
2
8
2
8
2
512
2
256
1
64
3
8
3
8
3
512
3
512
1
64
7
8
7
8
7
512
7
512
1
64
0
8
8
8
8
512
1
64
1
64
1
8
9
8
9
512
9
512
1
2
6
8
14
8
14
16
7
8
1
2
7
8
15
8
15
16
15
16
0
8
8
8
8
8
1
8
9
8
9
8
9
8
2
8
10
8
10
8
5
4
6
8
14
8
1792
8

Largest
normalized
14
7
128
240
240.0
Infinity
—
—
—
—
—
—
∞
—
Figure	
2.35	
Example	nonnegative	values	for	8-bit	floating-point
format.
There	are	
k
	=	4	exponent	bits	and	
n
	=	3	fraction	bits.	The	bias	is	7.
represented	value	
V
	=	2
	×	
M
.	Closest	to	0	are	the	denormalized
numbers,	starting	with	0	itself.	Denormalized	numbers	in	this	format	have
E
	=	1	–	7	=	–6,	giving	a	weight	
.	The	fractions	
f
	and	significands
M
	range	over	the	values	0,	
,	giving	numbers	
V
	in	the	range	0	to
.
The	smallest	normalized	numbers	in	this	format	also	have	
E
	=	1	–	7	=	–6,
and	the	fractions	also	range	over	the	values	0,	
.	However,	the
significands	then	range	from	1	+	0	=	1	to	
,	giving	numbers	
V
	in
the	range	
	to	
.
Observe	the	smooth	transition	between	the	largest	denormalized	number
	and	the	smallest	normalized	number	
.	This	smoothness	is	due
to	our	definition	of	
E
	for	denormalized	values.	By	making	it	1	–	
Bias
rather	than	–
Bias
,	we	compensate	for	the	fact	that	the	significand	of	a
denormalized	number	does	not	have	an	implied	leading	1.
As	we	increase	the	exponent,	we	get	successively	larger	normalized
values,	passing	through	1.0	and	then	to	the	largest	normalized	number.
This	number	has	exponent	
E
	=7,	giving	a	weight	2
	=	128.	The	fraction
6
8
15
8
1920
8
E
2
E
=
1
64
1
8
,
…
,
 
7
8
1
64
 
×
 
7
8
=
7
512
1
8
,
 
…
7
8
1
+
7
8
=
15
8
8
512
=
1
64
15
512
7
512
8
512
E

equals	
giving	a	significand	
.	Thus,	the	numeric	value	is	
V
	=
240.	Going	beyond	this	overflows	to	+∞.
One	interesting	property	of	this	representation	is	that	if	we	interpret	the
bit	representations	of	the	values	in	
Figure	
2.35
	as	unsigned	integers,
they	occur	in	ascending	order,	as	do	the	values	they	represent	as
floating-point	numbers.	This	is	no	accident—the	IEEE	format	was
designed	so	that	floating-point	numbers	could	be	sorted	using	an	integer
sorting	routine.	A	minor	difficulty	occurs	when	dealing	with	negative
numbers,	since	they	have	a	leading	1	and	occur	in	descending	order,	but
this	can	be	overcome	without	requiring	floating-point	operations	to
perform	comparisons	(see	
Problem	
2.84
).
Practice	Problem	
2.47
	(solution	page	
158
)
Consider	a	5-bit	floating-point	representation	based	on	the	IEEE
floating-point	format,	with	one	sign	bit,	two	exponent	bits	(
k
	=	2),
and	two	fraction	bits	(
n
	=	2).	The	exponent	bias	is	2
	–	1	=	1.
The	table	that	follows	enumerates	the	entire	nonnegative	range	for
this	5-bit	floating-point	representation.	Fill	in	the	blank	table	entries
using	the	following	directions:
e:
	The	value	represented	by	considering	the	exponent	field	to
be	an	unsigned	integer
E:
	The	value	of	the	exponent	after	biasing
2
:
	The	numeric	weight	of	the	exponent
f
:	The	value	of	the	fraction
M:
	The	value	of	the	significand
7
8
	
M
=
15
8
2–1
E
E

2
	×	
M:
	The	(unreduced)	fractional	value	of	the	number
V:
	The	reduced	fractional	value	of	the	number
Decimal:	The	decimal	representation	of	the	number
Express	the	values	of	2
,	f,	M
,	2
	×	
M
,	and	
V
	either	as	integers
(when	possible)	or	as	fractions	of	the	form	
,	where	
y
	is	a	power
of	2.	You	need	not	fill	in	entries	marked	—.
Bits
e
E
2
f
M
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
1
0
1
E
E
E
x
y
E
1
4
5
4

__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
—
—
—
—
—
—
—
—
—
—
—
—
—
—
—

—
—
—
—
—
Figure	
2.36
	shows	the	representations	and	numeric	values	of	some
important	single-	and	double-precision	floating-point	numbers.	As	with
the	8-bit	format	shown	in	
Figure	
2.35
,	we	can	see	some	general
properties	for	a	floating-point	representation	with	a	
k
-bit	exponent	and	an
n
-bit	fraction:
The	value	+0.0	always	has	a	bit	representation	of	all	zeros.
The	smallest	positive	denormalized	value	has	a	bit	representation
consisting	of	a	1	in	the	least	significant	bit	position	and	otherwise	all
zeros.	It	has	a	fraction	(and	significand)	value	
M
	=	
f
	=	2
	and	an
exponent	value	
.	The	numeric	value	is	therefore	
.
The	largest	denormalized	value	has	a	bit	representation	consisting	of
an	exponent	field	of	all	zeros	and	a	fraction	field	of	all	ones.	It	has	a
fraction	(and	significand)	value	
M
	=	
f
	=	1	–	2
	(which	we	have	written
1	—	
∊
)	and	an	exponent	value	
E
	=	–2
	+	2.	The	numeric	value	is
therefore	
,	which	is	just	slightly	smaller	than	the
smallest	normalized	value.
Single	precision
Double	precision
Description
Value
Decimal
Value
Decimal
Zero
00	·	·	·
00
0	·	·	·
00
0
0.0
0
0.0
Smallest
denormalized
00	·	·	·
00
0	·	·	·
01
2
	×
1.4	×
10
2
	×
4.9	×
10
–
n
E
=
−
2
k
−
1
+
2
V
=
2
−
n
−
2
k
−
1
+
2 
−
n
k
–1
V
=
(
1
−
2
−
n
)
×
2
−
2
k
−
1
+
2
−23
−126
−45
−52
−1022
−324

2
2
Largest
denormalized
00	···
00
1	···
11
(1	–	
∊
)	×
2
1.2	×
10
(1	–	
∊
)	×
2
2.2	×
10
Smallest
normalized
00	···
01
0	···
00
1	×	2
1.2	×
10
1	×	2
2.2	×
10
One
01	···
11
0	···
00
1	×	2
1.0
1	×	2
1.0
Largest
normalized
11	···
10
1	···
11
(2	–	
∊
)	×
2
3.4	×
10
(2	–	
∊
)	×
2
1.8	×
10
Figure	
2.36	
Examples	of	nonnegative	floating-point	numbers.
The	smallest	positive	normalized	value	has	a	bit	representation	with	a
1	in	the	least	significant	bit	of	the	exponent	field	and	otherwise	all
zeros.	It	has	a	significand	value	
M
	=	1	and	an	exponent	value	
E
	=	–2
	+	2.	The	numeric	value	is	therefore	
.
The	value	1.0	has	a	bit	representation	with	all	but	the	most	significant
bit	of	the	exponent	field	equal	to	1	and	all	other	bits	equal	to	0.	Its
significand	value	is	
M
	=	1	and	its	exponent	value	is	
E
	=	0.
The	largest	normalized	value	has	a	bit	representation	with	a	sign	bit	of
0,	the	least	significant	bit	of	the	exponent	equal	to	0,	and	all	other	bits
equal	to	1.	It	has	a	fraction	value	of	
f
	=	1	–	2
,	giving	a	significand	
M
=	2	–	2
	(which	we	have	written	2	–	
∊
.
)	It	has	an	exponent	value	
E
	=
2
	–	1,	giving	a	numeric	value	
.
−126
−1022
−126
−38
−1022
−308
−126
−38
−1022
−308
0
0
127
38
1023
308
k
–
1
V
=
2
−
2
k
−
1
+
2
–
n
–
n
k
–1
V
=
(
2
−
2
−
n
)
×
2
2
k
−
1
−
1
=
(
1
−
2
−
n
−
1
)
×
2
2
k
−
1

One	useful	exercise	for	understanding	floating-point	representations	is	to
convert	sample	integer	values	into	floating-point	form.	For	example,	we
saw	in	
Figure	
2.15
	that	12,345	has	binary	representation
[11000000111001].	We	create	a	normalized	representation	of	this	by
shifting	13	positions	to	the	right	of	a	binary	point,	giving	12345	=
1.1000000111001
	×	2
.	To	encode	this	in	IEEE	single-precision	format,
we	construct	the	fraction	field	by	dropping	the	leading	1	and	adding	10
zeros	to	the	end,	giving	binary	representation
[10000001110010000000000].	To	construct	the	exponent	field,	we	add
bias	127	to	13,	giving	140,	which	has	binary	representation	[10001100].
We	combine	this	with	a	sign	bit	of	0	to	get	the	floating-point
representation	in	binary	of	[01000110010000001110010000000000].
Recall	from	
Section	
2.1.3
	that	we	observed	the	following	correlation	in
the	bit-level	representations	of	the	integer	value	
	and	the
single-precision	floating-point	value	
We	can	now	see	that	the	region	of	correlation	corresponds	to	the	low-
order	bits	of	the	integer,	stopping	just	before	the	most	significant	bit	equal
to	1	(this	bit	forms	the	implied	leading	1),	matching	the	high-order	bits	in
the	fraction	part	of	the	floating-point	representation.
Practice	Problem	
2.48
	(solution	page	
159
)
2
13

As	mentioned	in	
Problem	
2.6
,	the	integer	3,510,593	has
hexadecimal	representation	
,	while	the	single-precision
floating-point	number	3,510,593.0	has	hexadecimal	representation
.	Derive	this	floating-point	representation	and	explain
the	correlation	between	the	bits	of	the	integer	and	floating-point
representations.
Practice	Problem	
2.49
	(solution	page	
159
)
A
.	
For	a	floating-point	format	with	an	
n
-bit	fraction,	give	a
formula	for	the	smallest	positive	integer	that	cannot	be
represented	exactly	(because	it	would	require	an	(
n
	+	1)-bit
fraction	to	be	exact).	Assume	the	exponent	field	size	
k
	is
large	enough	that	the	range	of	representable	exponents
does	not	provide	a	limitation	for	this	problem.
B
.	
What	is	the	numeric	value	of	this	integer	for	single-precision
format	(
n
	=	23)?
2.4.4	
Rounding
Floating-point	arithmetic	can	only	approximate	real	arithmetic,	since	the
representation	has	limited	range	and	precision.	Thus,	for	a	value	
x
,	we
generally	want	a	systematic	method	of	finding	the	“closest”	matching
value	
x
′	that	can	be	represented	in	the	desired	floating-point	format.	This
is	the	task	of	the	
rounding
	operation.	One	key	problem	is	to	define	the
direction	to	round	a	value	that	is	halfway	between	two	possibilities.	For
example,	if	I	have	$1.50	and	want	to	round	it	to	the	nearest	dollar,	should

the	result	be	$1	or	$2?	An	alternative	approach	is	to	maintain	a	lower	and
an	upper	bound	on	the	actual	number.	For	example,	we	could	determine
representable	values	
x
	and	
x
	such	that	the	value	
x
	is	guaranteed	to	lie
between	them:	
x
	≤	
x
	≤	
x
.	The	IEEE	floating-point	format	defines	four
different	
rounding	modes.
	The	default	method	finds	a	closest	match,
while	the	other	three	can	be	used	for	computing	upper	and	lower	bounds.
Figure	
2.37
	illustrates	the	four	rounding	modes	applied	to	the	problem
of	rounding	a	monetary	amount	to	the	nearest	whole	dollar.	Round-to-
even	(also	called	round-to-nearest)	is	the	default	mode.	It	attempts	to	find
a	closest	match.	Thus,	it	rounds	$1.40	to	$1	and	$1.60	to	$2,	since	these
are	the	closest	whole	dollar	values.	The	only	design	decision	is	to
determine	the	effect	of	rounding	values	that	are	halfway	between	two
possible	results.	Round-to-even	mode	adopts	the	convention	that	it
rounds	the	number	either	upward	or	downward	such	that	the	least
significant	digit	of	the	result	is	even.	Thus,	it	rounds	both	$1.50	and	$2.50
to	$2.
The	other	three	modes	produce	guaranteed	bounds	on	the	actual	value.
These	can	be	useful	in	some	numerical	applications.	Round-toward-zero
mode	rounds	positive	numbers	downward	and	negative	numbers	upward,
giving	a	value	
	such
Mode
$1.40
$1.60
$1.50
$2.50
$–1.50
Round-to-even
$1
$2
$2
$2
$–2
Round-toward-zero
$1
$1
$1
$2
$–1
Round-down
$1
$1
$1
$2
$–2
−
+
−
+
x
^

Round-up
$2
$2
$2
$3
$–1
Figure	
2.37	
Illustration	of	rounding	modes	for	dollar	rounding.
The	first	rounds	to	a	nearest	value,	while	the	other	three	bound	the	result
above	or	below.
that	
.	Round-down	mode	rounds	both	positive	and	negative
numbers	downward,	giving	a	value	
x
	such	that	
x
	≤	
x.
	Round-up	mode
rounds	both	positive	and	negative	numbers	upward,	giving	a	value	
x
such	that	
x
	≤	
x
.
Round-to-even	at	first	seems	like	it	has	a	rather	arbitrary	goal—why	is
there	any	reason	to	prefer	even	numbers?	Why	not	consistently	round
values	halfway	between	two	representable	values	upward?	The	problem
with	such	a	convention	is	that	one	can	easily	imagine	scenarios	in	which
rounding	a	set	of	data	values	would	then	introduce	a	statistical	bias	into
the	computation	of	an	average	of	the	values.	The	average	of	a	set	of
numbers	that	we	rounded	by	this	means	would	be	slightly	higher	than	the
average	of	the	numbers	themselves.	Conversely,	if	we	always	rounded
numbers	halfway	between	downward,	the	average	of	a	set	of	rounded
numbers	would	be	slightly	lower	than	the	average	of	the	numbers
themselves.	Rounding	toward	even	numbers	avoids	this	statistical	bias	in
most	real-life	situations.	It	will	round	upward	about	50%	of	the	time	and
round	downward	about	50%	of	the	time.
Round-to-even	rounding	can	be	applied	even	when	we	are	not	rounding
to	a	whole	number.	We	simply	consider	whether	the	least	significant	digit
is	even	or	odd.	For	example,	suppose	we	want	to	round	decimal	numbers
to	the	nearest	hundredth.	We	would	round	1.2349999	to	1.23	and
|
x
^
|
 
≤
 
|
x
|
−
−
+
+

1.2350001	to	1.24,	regardless	of	rounding	mode,	since	they	are	not
halfway	between	1.23	and	1.24.	On	the	other	hand,	we	would	round	both
1.2350000	and	1.2450000	to	1.24,	since	4	is	even.
Similarly,	round-to-even	rounding	can	be	applied	to	binary	fractional
numbers.	We	consider	least	significant	bit	value	0	to	be	even	and	1	to	be
odd.	In	general,	the	rounding	mode	is	only	significant	when	we	have	a	bit
pattern	of	the	form	
XX
	·	·	·	
X.YY
	·	·	·	
Y
100	·	·	·,	where	
X
	and	
Y
	denote
arbitrary	bit	values	with	the	rightmost	
Y
	being	the	position	to	which	we
wish	to	round.	Only	bit	patterns	of	this	form	denote	values	that	are
halfway	between	two	possible	results.	As	examples,	consider	the
problem	of	rounding	values	to	the	nearest	quarter	(i.e.,	2	bits	to	the	right
of	the	binary	point.)	We	would	round	
	down	to	10.00
(2),	and	
	up	to	
,	because	these	values	are
not	halfway	between	two	possible	values.	We	would	round
	up	to	11.00
	(3)	and	
	down	to
,	since	these	values	are	halfway	between	two	possible
results,	and	we	prefer	to	have	the	least	significant	bit	equal	to	zero.
Practice	Problem	
2.50
	(solution	page	
159
)
Show	how	the	following	binary	fractional	values	would	be	rounded
to	the	nearest	half	(1	bit	to	the	right	of	the	binary	point),	according
to	the	round-to-even	rule.	In	each	case,	show	the	numeric	values,
both	before	and	after	rounding.
A
.	
10.010
B
.	
10.011
C
.	
10.110
D
.	
11.001
10.00011
2
(
2
3
32
)
2
10.00110
2
(
2
3
16
)
10.01
2
(
2
1
4
)
10.11100
2
(
2
7
8
)
2
10.10100
2
(
2
5
8
)
10.10
2
(
2
1
2
)
2
2
2
2

Practice	Problem	
2.51
	(solution	page	
159
)
We	saw	in	
Problem	
2.46
	that	the	Patriot	missile	software
approximated	0.1	as	
x
	=	0.	00011001100110011001100
.	Suppose
instead	that	they	had	used	IEEE	round-to-even	mode	to	determine
an	approximation	
x
′	to	0.1	with	23	bits	to	the	right	of	the	binary
point.
A
.	
What	is	the	binary	representation	of	
x
′?
B
.	
What	is	the	approximate	decimal	value	of	
x
′	–	0.1?
C
.	
How	far	off	would	the	computed	clock	have	been	after	100
hours	of	operation?
D
.	
How	far	off	would	the	program's	prediction	of	the	position	of
the	Scud	missile	have	been?
Practice	Problem	
2.52
	(solution	page	
160
)
Consider	the	following	two	7-bit	floating-point	representations
based	on	the	IEEE	floating-point	format.	Neither	has	a	sign	bit—
they	can	only	represent	nonnegative	numbers.
1
.	
Format	A
There	are	
k	=	3
	exponent	bits.	The	exponent	bias	is	3.
There	are	
n
	=	4	fraction	bits.
2
.	
Format	B
There	are	
k
	=	4	exponent	bits.	The	exponent	bias	is	7.
There	are	
n
	=	3	fraction	bits.
Below,	you	are	given	some	bit	patterns	in	format	A,	and	your	task
is	to	convert	them	to	the	closest	value	in	format	B.	If	necessary,
2

you	should	apply	the	round-to-even	rounding	rule.	In	addition,	give
the	values	of	numbers	given	by	the	format	A	and	format	B	bit
patterns.	Give	these	as	whole	numbers	(e.g.,	17)	or	as	fractions
(e.g.,	17/64).
Format	A
Format	B
Bits
Value
Bits
Value
1
1
__________
__________
__________
__________
__________
__________
__________
__________
__________
2.4.5	
Floating-Point	Operations
The	IEEE	standard	specifies	a	simple	rule	for	determining	the	result	of	an
arithmetic	operation	such	as	addition	or	multiplication.	Viewing	floating-
point	values	
x
	
and	
y
	as	real	numbers,	and	some	operation	
⊙
	defined	over
real	numbers,	the	computation	should	yield	
Round
(
x
	
⊙
	
y
),	the	result	of
applying	rounding	to	the	exact	result	of	the	real	operation.	In	practice,
there	are	clever	tricks	floating-point	unit	designers	use	to	avoid
performing	this	exact	computation,	since	the	computation	need	only	be
sufficiently	precise	to	guarantee	a	correctly	rounded	result.	When	one	of
the	arguments	is	a	special	value,	such	as	–0,	∞,	or	
NaN
,	the	standard
specifies	conventions	that	attempt	to	be	reasonable.	For	example,	1/–0	is
defined	to	yield	-∞,	while	1/+0	is	defined	to	yield	+∞.

One	strength	of	the	IEEE	standard's	method	of	specifying	the	behavior	of
floating-point	operations	is	that	it	is	independent	of	any	particular
hardware	or	software	realization.	Thus,	we	can	examine	its	abstract
mathematical	properties	without	considering	how	it	is	actually
implemented.
We	saw	earlier	that	integer	addition,	both	unsigned	and	two's
complement,	forms	an	abelian	group.	Addition	over	real	numbers	also
forms	an	abelian	group,	but	we	must	consider	what	effect	rounding	has
on	these	properties.	Let	us	define	
x
	+
	
y
	to	be	
Round
(
x
	+	
y
).	This
operation	is	defined	for	all	values	of	
x
	and	
y
,	although	it	may	yield	infinity
even	when	both	
x
	and	
y
	are	real	numbers	due	to	overflow.	The	operation
is	commutative,	with	
x
	+
	
y
	=	
y
	+
	
x
	for	all	values	of	
x
	and	
y.
	On	the	other
hand,	the	operation	is	not	associative.	For	example,	with	single-precision
floating	point	the	expression	
	evaluates	to	
—the
value	3.14	is	lost	due	to	rounding.	On	the	other	hand,	the	expression
	evaluates	to	3.14.	As	with	an	abelian	group,	most
values	have	inverses	under	floating-point	addition,	that	is,	
x
	+
	–	
x
	=	0.
The	exceptions	are	infinities	(since	+∞	–∞	=	
NaN
),	and	
NaN
s,	since	
NaN
+
	
x
	=	
NaN
	for	any	
x.
The	lack	of	associativity	in	floating-point	addition	is	the	most	important
group	property	that	is	lacking.	It	has	important	implications	for	scientific
programmers	and	compiler	writers.	For	example,	suppose	a	compiler	is
given	the	following	code	fragment:
f
f
f
f
f

The	compiler	might	be	tempted	to	save	one	floating-point	addition	by
generating	the	following	code:
However,	this	computation	might	yield	a	different	value	for	
	than	would
the	original,	since	it	uses	a	different	association	of	the	addition
operations.	In	most	applications,	the	difference	would	be	so	small	as	to
be	inconsequential.	Unfortunately,	compilers	have	no	way	of	knowing
what	trade-offs	the	user	is	willing	to	make	between	efficiency	and
faithfulness	to	the	exact	behavior	of	the	original	program.	As	a	result,
they	tend	to	be	very	conservative,	avoiding	any	optimizations	that	could
have	even	the	slightest	effect	on	functionality.
On	the	other	hand,	floating-point	addition	satisfies	the	following
monotonicity	property:	if	
a
	≥	
b
,	then	
for	any	values	of	
a,	b,
	and
x
	other	than	
NaN
.	This	property	of	real	(and	integer)	addition	is	not
obeyed	by	unsigned	or	two's-complement	addition.
Floating-point	multiplication	also	obeys	many	of	the	properties	one
normally	associates	with	multiplication.	Let	us	define	
x
	*
	
y
	to	be	
Round(x
×	y).
	This	operation	is	closed	under	multiplication	(although	possibly
yielding	infinity	or	
NaN
),	it	is	commutative,	and	it	has	1.0	as	a
x
+
f
a
≥
x
+
f
b
	
f

multiplicative	identity.	On	the	other	hand,	it	is	not	associative,	due	to	the
possibility	of	overflow	or	the	loss	of	precision	due	to	rounding.	For
example,	with	single-precision	floating	point,	the	expression
,	while	
	evaluates	to
.	In	addition,	floating-point	multiplication	does	not	distribute	over
addition.	For	example,	with	single-precision	floating	point,	the	expression
	evaluates	to	
,	while	
	evaluates
to	
.
On	the	other	hand,	floating-point	multiplication	satisfies	the	following
monotonicity	properties	for	any	values	
a,	b
,	and	
c
	other	than	
NaN
:
In	addition,	we	are	also	guaranteed	that	
a
	*
	
a
	≥	0,	as	long	as	
a
	≠	
NaN.
	As
we	saw	earlier,	none	of	these	monotonicity	properties	hold	for	unsigned
or	two's-complement	multiplication.
This	lack	of	associativity	and	distributivity	is	of	serious	concern	to
scientific	programmers	and	to	compiler	writers.	Even	such	a	seemingly
simple	task	as	writing	code	to	determine	whether	two	lines	intersect	in
three-dimensional	space	can	be	a	major	challenge.
2.4.6	
Floating	Point	in	C
All	versions	of	C	provide	two	different	floating-point	data	types:	
	and
.	On	machines	that	support	IEEE	floating	point,	these	data	types
a
≥
b
and
c
≥
0
⇒
a
∗
f
c
≥
b
∗
f
c
a
≥
b
and
c
≤
0
⇒
a
∗
f
c
≤
b
∗
f
c
f

correspond	to	single-	and	double-precision	floating	point.	In	addition,	the
machines	use	the	round-to-even	rounding	mode.	Unfortunately,	since	the
C	standards	do	not	require	the	machine	to	use	IEEE	floating	point,	there
are	no	standard	methods	to	change	the	rounding	mode	or	to	get	special
values	such	as	–0,	+∞,	–∞,	or	
NaN.
	Most	systems	provide	a	combination
of	include	
	files	and	procedure	libraries	to	provide	access	to	these
features,	but	the	details	vary	from	one	system	to	another.	For	example,
the	GNU	compiler	
GCC
	
defines	program	constants	
	(for	+∞)	and
	(for	
NaN
)	when	the	following	sequence	occurs	in	the	program	file:
Practice	Problem	
2.53
	(solution	page	
160
)
Fill	in	the	following	macro	definitions	to	generate	the	double-
precision	values	+∞,	–∞,	and	–0:
You	cannot	use	any	include	files	(such	as	
),	but	you	can
make	use	of	the	fact	that	the	largest	finite	number	that	can	be
represented	with	double	precision	is	around	1.8	×	10
.
308

When	casting	values	between	
,	and	
	formats,	the
program	changes	the	numeric	values	and	the	bit	representations	as
follows	(assuming	data	type	
	is	32	bits):
From	
	to	
,	the	number	cannot	overflow,	but	it	may	be
rounded.
From	
	or	
	to	
,	the	exact	numeric	value	can	be
preserved	because	
	has	both	greater	range	(i.e.,	the	range	of
representable	values),	as	well	as	greater	precision	(i.e.,	the	number	of
significant	bits).
From	
	to	
,	the	value	can	overflow	to	+∞	or	–∞,	since	the
range	is	smaller.	Otherwise,	it	may	be	rounded,	because	the	precision
is	smaller.
From	
	or	
	to	
,	the	value	will	be	rounded	toward	zero.
For	example,	1.999	will	be	converted	to	1,	while	–1.999	will	be
converted	to	–1.	Furthermore,	the	value	may	overflow.	The	C
standards	do	not	specify	a	fixed	result	for	this	case.	Intel-compatible
microprocessors	designate	the	bit	pattern	[10	...	00]	(
TMin
	for	word
size	
w
)	as	an	
integer	indefinite
	value.	Any	conversion	from	floating
point	to	integer	that	cannot	assign	a	reasonable	integer	approximation
yields	this	value.	Thus,	the	expression	
,
generating	a	negative	value	from	a	positive	one.
Practice	Problem	
2.54
	(solution	page	
160
)
Assume	variables	
	and	
	are	of	type	
	and
,	respectively.	Their	values	are	arbitrary,	except	that	neither
	nor	
	equals	+∞,	–∞,	or	
NaN.
	For	each	of	the	following	C
w

expressions,	either	argue	that	it	will	always	be	true	(i.e.,	evaluate
to	1)	or	give	a	value	for	the	variables	such	that	it	is	not	true	(i.e.,
evaluates	to	0).
A
.	
B
.	
C
.	
D
.	
E
.	
F
.	
G
.	
H
.	

2.5	
Summary
Computers	encode	information	as	bits,	generally	organized	as
sequences	of	bytes.	Different	encodings	are	used	for	representing
integers,	real	numbers,	and	character	strings.	Different	models	of
computers	use	different	conventions	for	encoding	numbers	and	for
ordering	the	bytes	within	multi-byte	data.
The	C	language	is	designed	to	accommodate	a	wide	range	of	different
implementations	in	terms	of	word	sizes	and	numeric	encodings.
Machines	with	64-bit	word	sizes	have	become	increasingly	common,
replacing	the	32-bit	machines	that	dominated	the	market	for	around	30
years.	Because	64-bit	machines	can	also	run	programs	compiled	for	32-
bit	machines,	we	have	focused	on	the	distinction	between	32-and	64-bit
programs,	rather	than	machines.	The	advantage	of	64-bit	programs	is
that	they	can	go	beyond	the	4	GB	address	limitation	of	32-bit	programs.
Most	machines	encode	signed	numbers	using	a	two's-complement
representation	and	encode	floating-point	numbers	using	IEEE	Standard
754.	Understanding	these	encodings	at	the	bit	level,	as	well	as
understanding	the	mathematical	characteristics	of	the	arithmetic
operations,	is	important	for	writing	programs	that	operate	correctly	over
the	full	range	of	numeric	values.
When	casting	between	signed	and	unsigned	integers	of	the	same	size,
most	C	implementations	follow	the	convention	that	the	underlying	bit
pattern	does	not	change.	On	a	two's-complement	machine,	this	behavior

is	characterized	by	functions	
T2U
	and	
U2T
,	for	a	
w
-bit	value.	The
implicit	casting	of	C	gives	results	that	many	programmers	do	not
anticipate,	often	leading	to	program	bugs.
Due	to	the	finite	lengths	of	the	encodings,	computer	arithmetic	has
properties	quite	different	from	conventional	integer	and	real	arithmetic.
The	finite	length	can	cause	numbers	to	overflow,	when	they	exceed	the
range	of	the	representation.	Floating-point	values	can	also	underflow,
when	they	are	so	close	to	0.0	that	they	are	changed	to	zero.
The	finite	integer	arithmetic	implemented	by	C,	as	well	as	most	other
programming	languages,	has	some	peculiar	properties	compared	to	true
integer	arithmetic.	For	example,	the	expression	
	can	evaluate	to	a
negative	number	due	to	overflow.	Nonetheless,	both	unsigned	and	two's-
complement	arithmetic	satisfy	many	of	the	other	properties	of	integer
arithmetic,	including	associativity,	commutativity,	and	distributivity.	This
allows	compilers	to	do	many	optimizations.	For	example,	in	replacing	the
expression	
	by	
,	we	make	use	of	the	associative,
commutative,	and	distributive	properties,	along	with	the	relationship
between	shifting	and	multiplying	by	powers	of	2.
We	have	seen	several	clever	ways	to	exploit	combinations	of	bit-level
operations	and	arithmetic	operations.	For	example,	we	saw	that	with
two's-complement	arithmetic,	
	is	equivalent	to	
.	As	another
example,	suppose	we	want	a	bit
Aside	
Ariane	5:	The	high	cost	of
w
w

floating-point	overflow
Converting	large	floating-point	numbers	to	integers	is	a	common
source	of	programming	errors.	Such	an	error	had	disastrous
consequences	for	the	maiden	voyage	of	the	Ariane	5	rocket,	on
June	4,	1996.	Just	37	seconds	after	liftoff,	the	rocket	veered	off	its
flight	path,	broke	up,	and	exploded.	Communication	satellites
valued	at	$500	million	were	on	board	the	rocket.
A	later	investigation	[
73
,	
33
]	showed	that	the	computer	controlling
the	inertial	navigation	system	had	sent	invalid	data	to	the
computer	controlling	the	engine	nozzles.	Instead	of	sending	flight
control	information,	it	had	sent	a	diagnostic	bit	pattern	indicating
that	an	overflow	had	occurred	during	the	conversion	of	a	64-bit
floating-point	number	to	a	16-bit	signed	integer.
The	value	that	overflowed	measured	the	horizontal	velocity	of	the
rocket,	which	could	be	more	than	five	times	higher	than	that
achieved	by	the	earlier	Ariane	4	rocket.	In	the	design	of	the	Ariane
4	software,	they	had	carefully	analyzed	the	numeric	values	and
determined	that	the	horizontal	velocity	would	never	overflow	a	16-
bit	number.	Unfortunately,	they	simply	reused	this	part	of	the
software	in	the	Ariane	5	without	checking	the	assumptions	on
which	it	had	been	based.
pattern	of	the	form	[0,	...	,	0,	1,	...,	1],	consisting	of	
w
	–	
k
	zeros	followed
by	
k
	ones.	Such	bit	patterns	are	useful	for	masking	operations.	This
pattern	can	be	generated	by	the	C	expression	
,	exploiting	the
property	that	the	desired	bit	pattern	has	numeric	value	2
	–	1.	For
example,	the	expression	
	will	generate	the	bit	pattern	
.
k

Floating-point	representations	approximate	real	numbers	by	encoding
numbers	of	the	form	
x
	×	2
.	IEEE	Standard	754	provides	for	several
different	precisions,	with	the	most	common	being	single	(32	bits)	and
double	(64	bits).	IEEE	floating	point	also	has	representations	for	special
values	representing	plus	and	minus	infinity,	as	well	as	not-a-number.
Floating-point	arithmetic	must	be	used	very	carefully,	because	it	has	only
limited	range	and	precision,	and	because	it	does	not	obey	common
mathematical	properties	such	as	associativity.
y

Bibliographic	Notes
Reference	books	on	C	[
45
,	
61
]	discuss	properties	of	the	different	data
types	and	operations.	Of	these	two,	only	Steele	and	Harbison	[
45
]	cover
the	newer	features	found	in	ISO	C99.	There	do	not	yet	seem	to	be	any
books	that	cover	the	features	found	in	ISO	C11.	The	C	standards	do	not
specify	details	such	as	precise	word	sizes	or	numeric	encodings.	Such
details	are	intentionally	omitted	to	make	it	possible	to	implement	C	on	a
wide	range	of	different	machines.	Several	books	have	been	written	giving
advice	to	C	programmers	[
59
,	
74
]	that	warn	about	problems	with
overflow,	implicit	casting	to	unsigned,	and	some	of	the	other	pitfalls	we
have	covered	in	this	chapter.	These	books	also	provide	helpful	advice	on
variable	naming,	coding	styles,	and	code	testing.	Seacord's	book	on
security	issues	in	C	and	C++	programs	[
97
]	combines	information	about
C	programs,	how	they	are	compiled	and	executed,	and	how
vulnerabilities	may	arise.	Books	on	Java	(we	
recommend	the	one
coauthored	by	James	Gosling,	the	creator	of	the	language	[
5
])	describe
the	data	formats	and	arithmetic	operations	supported	by	Java.
Most	books	on	logic	design	[
58
,	
116
]	have	a	section	on	encodings	and
arithmetic	operations.	Such	books	describe	different	ways	of
implementing	arithmetic	circuits.	Overton's	book	on	IEEE	floating	point
[
82
]	provides	a	detailed	description	of	the	format	as	well	as	the	properties
from	the	perspective	of	a	numerical	applications	programmer.

Homework	Problems
2.55	
♦
Compile	and	run	the	sample	code	that	uses	
	(file	
)	on	different	machines	to	which	you	have	access.
Determine	the	byte	orderings	used	by	these	machines.
2.56	
♦
Try	running	the	code	for	
	for	different	sample	values.
2.57	
♦
Write	procedures	
,	and	
	that
print	the	byte	representations	of	C	objects	of	types	
and	
	respectively.	Try	these	out	on	several	machines.
2.58	
♦♦

Write	a	procedure	
	that	will	return	1	when
compiled	and	run	on	a	little-endian	machine,	and	will	return	0
when	compiled	and	run	on	a	big-endian	machine.	This	program
should	run	on	any	machine,	regardless	of	its	word	size.
2.59	
♦♦
Write	a	C	expression	that	will	yield	a	word	consisting	of	the	least
significant	byte	of	
	and	the	remaining	bytes	of	
.	For	operands	
	and	
	this	would	give	
.
2.60	♦♦
Suppose	we	number	the	bytes	in	a	
w
-bit	word	from	0	(least
significant)	to	
w
/8	–	1	(most	significant).	Write	code	for	the
following	C	function,	which	will	return	an	unsigned	value	in	which
byte	
	of	argument	
	has	been	replaced	by	byte	
:
Here	are	some	examples	showing	how	the	function	should	work:

Bit-Level	Integer	Coding	Rules
In	several	of	the	following	problems,	we	will	artificially	restrict	what
programming	constructs	you	can	use	to	help	you	gain	a	better
understanding	of	the	bit-level,	
logic,	and	arithmetic	operations	of	C.	In
answering	these	problems,	your	code	must	follow	these	rules:
Assumptions
Integers	are	represented	in	two's-complement	form.
Right	shifts	of	signed	data	are	performed	arithmetically.
Data	type	
	is	
w
	bits	long.	For	some	of	the	problems,	you	will	be
given	a	specific	value	for	
w
,	but	otherwise	your	code	should	work
as	long	as	
w
	is	a	multiple	of	8.	You	can	use	the	expression
	to	compute	
w
.
Forbidden
Conditionals	
	loops,	switch	statements,	function	calls,
and	macro	invocations.
Division,	modulus,	and	multiplication.
Relative	comparison	operators	
.
Allowed	operations
All	bit-level	and	logic	operations.
Left	and	right	shifts,	but	only	with	shift	amounts	between	0	and	
w
	–
1.
Addition	and	subtraction.
Equality	
	and	inequality	
	tests.	(Some	of	the	problems	do
not	allow	these.)

Integer	constants	
	and	
Casting	between	data	types	
	and	
,	either	explicitly	or
implicitly.
Even	with	these	rules,	you	should	try	to	make	your	code	readable	by
choosing	descriptive	variable	names	and	using	comments	to	describe	the
logic	behind	your	solutions.	As	an	example,	the	following	code	extracts
the	most	significant	byte	from	integer	argument	
:
2.61	♦♦
Write	C	expressions	that	evaluate	to	1	when	the	following
conditions	are	true	and	to	0	when	they	are	false.	Assume	
	is	of
type	
.
A
.	
Any	bit	of	
	equals	1.

B
.	
Any	bit	of	
	equals	0.
C
.	
Any	bit	in	the	least	significant	byte	of	
	equals	1.
D
.	
Any	bit	in	the	most	significant	byte	of	
	equals	0.
Your	code	should	follow	the	bit-level	integer	coding	rules	(page
128
),	with	the	additional	restriction	that	you	may	not	use	equality
	or	inequality	
	tests.
2.62	♦♦♦
Write	a	function	
	that	yields	1	when
run	on	a	machine	that	uses	arithmetic	right	shifts	for	data	type	
and	yields	
	otherwise.	Your	code	should	work	on	a	machine	with
any	word	size.	Test	your	code	on	several	machines.
2.63	♦♦♦
Fill	in	code	for	the	following	C	functions.	Function	
	performs	a
logical	right	shift	using	an	arithmetic	right	shift	(given	by	value
),	followed	by	other	operations	not	including	right	shifts	or
division.	Function	
	performs	an	arithmetic	right	shift	using	a
logical	right	shift	(given	by	value	
),	followed	by	other
operations	not	including	right	shifts	or	division.	You	may	use	the
computation	
	to	determine	
w
,	the	number	of	bits	in
data	type	
.	The	shift	amount	
	can	range	from	
	to	
w
	–	1.

2.64	♦
Write	code	to	implement	the	following	function:

Your	function	should	follow	the	bit-level	integer	coding	rules	(page
128
),	except	that	you	may	assume	that	data	type	
	has	
w
	=	32
bits.
2.65	♦♦♦♦
Write	code	to	implement	the	following	function:
Your	function	should	follow	the	bit-level	integer	coding	rules	(page
128
),	except	that	you	may	assume	that	data	type	
	has	
w
	=	32
bits.
Your	code	should	contain	a	total	of	at	most	12	arithmetic,	bitwise,
and	logical	operations.
2.66	♦♦♦♦

Write	code	to	implement	the	following	function:
Your	function	should	follow	the	bit-level	integer	coding	rules	(page
128
),	except	that	you	may	assume	that	data	type	
	has	
w
	=	32
bits.
Your	code	should	contain	a	total	of	at	most	15	arithmetic,	bitwise,
and	logical	operations.
Hint:
	First	transform	
	into	a	bit	vector	of	the	form	[0	...	011	...	1].
2.67	♦♦
You	are	given	the	task	of	writing	a	procedure	
	that
yields	1	when	run	on	a	machine	for	which	an	
	is	32	bits,	and
yields	0	otherwise.	You	are	not	allowed	to	use	the	
	operator.
Here	is	a	first	attempt:

When	compiled	and	run	on	a	32-bit	SUN	SPARC,	however,	this
procedure	returns	0.	The	following	compiler	message	gives	us	an
indication	of	the	problem:
A
.	
In	what	way	does	our	code	fail	to	comply	with	the	C
standard?
B
.	
Modify	the	code	to	run	properly	on	any	machine	for	which
data	type	
	is	at	least	32	bits.
C
.	
Modify	the	code	to	run	properly	on	any	machine	for	which
data	type	
	is	at	least	16	bits.
2.68	♦♦

Write	code	for	a	function	with	the	following	prototype:
Your	function	should	follow	the	bit-level	integer	coding	rules	(page
128
).	Be	careful	of	the	case	
	=	
w
.
2.69	♦♦♦
Write	code	for	a	function	with	the	following	prototype:
Your	function	should	follow	the	bit-level	integer	coding	rules	(page
128
).	Be	careful	of	the	case	
	=	0.

2.70	♦♦
Write	code	for	the	function	with	the	following	prototype:
Your	function	should	follow	the	bit-level	integer	coding	rules	(page
128
).
2.71
You	just	started	working	for	a	company	that	is	implementing	a	set
of	procedures	to	operate	on	a	data	structure	where	4	signed	bytes
are	packed	into	a	32-bit	
.	Bytes	within	the	word	are
numbered	from	0	(least	significant)	to	3	
(most	significant).	You
have	been	assigned	the	task	of	implementing	a	function	for	a
machine	using	two's-complement	arithmetic	and	arithmetic	right
shifts	with	the	following	prototype:

That	is,	the	function	will	extract	the	designated	byte	and	sign
extend	it	to	be	a	32-bit	
.
Your	predecessor	(who	was	fired	for	incompetence)	wrote	the
following	code:
A
.	
What	is	wrong	with	this	code?
B
.	
Give	a	correct	implementation	of	the	function	that	uses	only
left	and	right	shifts,	along	with	one	subtraction.
2.72
You	are	given	the	task	of	writing	a	function	that	will	copy	an	integer
	into	a	buffer	
,	but	it	should	do	so	only	if	enough	space	is

available	in	the	buffer.
Here	is	the	code	you	write:
This	code	makes	use	of	the	library	function	
.	Although	its
use	is	a	bit	artificial	here,	where	we	simply	want	to	copy	an	
,	it
illustrates	an	approach	commonly	used	to	copy	larger	data
structures.
You	carefully	test	the	code	and	discover	that	it	
always
	copies	the
value	to	the	buffer,	even	when	
	is	too	small.
A
.	
Explain	why	the	conditional	test	in	the	code	always
succeeds.	
Hint:
	The	
	operator	returns	a	value	of	type
.
B
.	
Show	how	you	can	rewrite	the	conditional	test	to	make	it
work	properly.
2.73
Write	code	for	a	function	with	the	following	prototype:

Instead	of	overflowing	the	way	normal	two's-complement	addition
does,	saturating	addition	returns	
TMax
	when	there	would	be
positive	overflow,	and	
TMin
	when	there	would	be	negative
overflow.	Saturating	arithmetic	is	commonly	used	in	programs	that
perform	digital	signal	processing.
Your	function	should	follow	the	bit-level	integer	coding	rules	(page
128
).
2.74
Write	a	function	with	the	following	prototype:
This	function	should	return	1	if	the	computation	
	does	not
overflow.
2.75

Suppose	we	want	to	compute	the	complete	2
w
-bit	representation
of	
x	·	y
,	where	both	
x
	and	
y
	are	unsigned,	on	a	machine	for	which
data	type	
	is	
w
	bits.	The	low-order	
w
	bits	of	the	product
can	be	computed	with	the	expression	
,	so	we	only	require	a
procedure	with	prototype
that	computes	the	high-order	
w
	bits	of	
x	·	y
	for	unsigned	variables.
We	have	access	to	a	library	function	with	prototype
that	computes	the	high-order	
w
	bits	of	
x	·	y
	for	the	case	where	
x
and	
y
	are	in	two's-complement	form.	Write	code	calling	this
procedure	to	implement	the	function	for	unsigned	arguments.
Justify	the	correctness	of	your	solution.
Hint:
	Look	at	the	relationship	between	the	signed	product	
x	·	y
	and
the	unsigned	product	
x′	·	y′
	in	the	derivation	of	
Equation	
2.18
.
2.76
The	library	function	
	has	the	following	declaration:

According	to	the	library	documentation,	“The	
	function
allocates	memory	for	an	array	of	
	elements	of	
	bytes
each.	The	memory	is	set	to	zero.	If	
	or	
	is	zero,	then
	returns	
.”
Write	an	implementation	of	
	that	performs	the	allocation	by
a	call	to	
	and	sets	the	memory	to	zero	via	
.	Your
code	should	not	have	any	vulnerabilities	due	to	arithmetic
overflow,	and	it	should	work	correctly	regardless	of	the	number	of
bits	used	to	represent	data	of	type	
.
As	a	reference,	functions	
	and	
	have	the	following
declarations:
2.77
Suppose	we	are	given	the	task	of	generating	code	to	multiply
integer	variable	
	by	various	different	constant	factors	
K
.	To	be
efficient,	we	want	to	use	only	the	operations	+,	–,	and	
≪
.	For	the
following	values	of	
K
,	write	C	expressions	to	perform	the
multiplication	using	at	most	three	operations	per	expression.
A
.	
K	=	17

B
.	
K	=	–7
C
.	
K
	=	60
D
.	
K
	=	–112
2.78
Write	code	for	a	function	with	the	following	prototype:
The	function	should	compute	
x
/2
	with	correct	rounding,	and	it
should	follow	the	bit-level	integer	coding	rules	(page	
128
).
2.79
Write	code	for	a	function	
	that,	for	integer	argument	
,
computes	
	but	follows	the	bit-level	integer	coding	rules	(page
128
).	Your	code	should	replicate	the	fact	that	the	computation	
can	cause	overflow.
2.80
k

Write	code	for	a	function	
	that,	for	integer	argument
,	computes	the	value	of	
,	rounded	toward	zero.	It	should	not
overflow.	Your	function	should	follow	the	bit-level	integer	coding
rules	(page	
128
).
2.81
Write	C	expressions	to	generate	the	bit	patterns	that	follow,	where
a
	represents	
k
	repetitions	of	symbol	
a
.	Assume	a	
w
-bit	data	type.
Your	code	may	contain	references	to	parameters	
	and	
,
representing	the	values	of	
j
	and	
k
,	but	not	a	parameter
representing	
w
.
A
.	
1
0
B
.	
0
1
0
2.82
We	are	running	programs	where	values	of	type	
	are	32	bits.
They	are	represented	in	two's	complement,	and	they	are	right
shifted	arithmetically.	Values	of	type	
	are	also	32	bits.
We	generate	arbitrary	values	
	and	
,	and	convert	them	to
unsigned	values	as	follows:
3
4
x
k
w-k
k
w-k-j
k
j

For	each	of	the	following	C	expressions,	you	are	to	indicate
whether	or	not	the	expression	
always
	yields	1.	If	it	always	yields	1,
describe	the	underlying	mathematical	principles.	Otherwise,	give
an	example	of	arguments	that	make	it	yield	0.
A
.	
B
.	
C
.	
D
.	
E
.	
2.83
Consider	numbers	having	a	binary	representation	consisting	of	an
infinite	string	of	the	form	0.
y	y	y	y	y	y
	...,	where	
y
	is	a	
k
-bit
sequence.	For	example,	the	binary	representation	of	
	is
0.01010101	...	(
y
	=	01),	while	the	representation	of	
	is
0.001100110011	...	(
y
	=	0011).
A
.	
Let	
Y
	=	
B2U
(y)
,	that	is,	the	number	having	binary
representation	
y
.	Give	a	formula	in	terms	of	
Y
	and	
k
	for	the
value	represented	by	the	infinite	string.	
Hint:
	Consider	the
effect	of	shifting	the	binary	point	
k
	positions	to	the	right.
1
3
1
5
k

B
.	
What	is	the	numeric	value	of	the	string	for	the	following
values	of	
y
?
a
.	
101
b
.	
0110
c
.	
010011
2.84
Fill	in	the	return	value	for	the	following	procedure,	which	tests
whether	its	first	argument	is	less	than	or	equal	to	its	second.
Assume	the	function	
	returns	an	unsigned	32-bit	number
having	the	same	bit	representation	as	its	floating-point	argument.
You	can	assume	that	neither	argument	is	
NaN
.	The	two	flavors	of
zero,	+0	and	–0,	are	considered	equal.

2.85
Given	a	floating-point	format	with	a	
k
-bit	exponent	and	an	
n
-bit
fraction,	write	formulas	for	the	exponent	
E
,	the	significand	
M
,	the
fraction	
f
,	and	the	value	
V
	for	the	quantities	that	follow.	In	addition,
describe	the	bit	representation.
A
.	
The	number	7.0
B
.	
The	largest	odd	integer	that	can	be	represented	exactly
C
.	
The	reciprocal	of	the	smallest	positive	normalized	value
2.86
Intel-compatible	processors	also	support	an	“extended-precision”
floating-point	format	with	an	80-bit	word	divided	into	a	sign	bit,	
k
	=
15	exponent	bits,	a	single	
integer
	bit,	and	
n
	=	63	fraction	bits.	The
integer	bit	is	an	explicit	copy	of	the	implied	bit	in	the	IEEE	floating-
point	representation.	That	is,	it	equals	1	for	normalized	values	and
0	for	denormalized	values.	Fill	in	the	following	table	giving	the
approximate	values	of	some	“interesting”	numbers	in	this	format:
Extended	precision
Description
Value
Decimal
Smallest	positive	denormalized
__________
__________
Smallest	positive	normalized
__________
__________
Largest	normalized
__________
__________

This	format	can	be	used	in	C	programs	compiled	for	Intel-
compatible	machines	by	declaring	the	data	to	be	of	type	
.	However,	it	forces	the	compiler	to	generate	code	based	on
the	legacy	8087	floating-point	instructions.	The	resulting	program
will	most	likely	run	much	slower	than	would	be	the	case	for	data
type	
	or	
.
2.87
The	2008	version	of	the	IEEE	floating-point	standard,	named	IEEE
754-2008,	includes	a	16-bit	“half-precision”	floating-point	format.	It
was	originally	devised	by	computer	graphics	companies	for	storing
data	in	which	a	higher	dynamic	range	is	required	than	can	be
achieved	with	16-bit	integers.	This	format	has	1	sign	bit,	5
exponent	bits	(
k
	=	5),	and	10	fraction	bits	(
n
	=	10).	The	exponent
bias	is	2
	–	1	=	15.
Fill	in	the	table	that	follows	for	each	of	the	numbers	given,	with	the
following	instructions	for	each	column:
Hex:	The	four	hexadecimal	digits	describing	the	encoded	form.
M
:	The	value	of	the	significand.	This	should	be	a	number	of	the
form	
x
	or	
,	where	
x
	is	an	integer	and	
y
	is	an	integral	power	of
2.	Examples	include	0,	
,	and	
.
E
:	The	integer	value	of	the	exponent.
V
:	The	numeric	value	represented.	Use	the	notation	
x
	or	
x
	×	2
,
where	
x
	and	
z
	are	integers.
5–1
x
y
67
64
1
256
z

D
:	The	(possibly	approximate)	numerical	value,	as	is	printed
using	the	
	formatting	specification	of	
.
As	an	example,	to	represent	the	number	
,	we	would	have	
s
	=	0,
	and	
E
	=	–1.	Our	number	would	therefore	have	an	exponent
field	of	01110
	(decimal	value	15	–	1	=	14)	and	a	significand	field
of	1100000000
,	giving	a	hex	representation	
.	The	numerical
value	is	0.875.
You	need	not	fill	in	entries	marked	—.
Description
Hex
M
E
V
D
–0
__________
__________
__________
–0
–0.0
Smallest
value	>	2
__________
__________
__________
__________
__________
512
__________
__________
__________
512
512.0
Largest
denormalized
__________
__________
__________
__________
__________
–∞
__________
—
—
-∞
–∞
Number	with
hex
representation
__________
__________
__________
__________
2.88
7
8
M
=
7
4
2
2

Consider	the	following	two	9-bit	floating-point	representations
based	on	the	IEEE	floating-point	format.
1
.	
Format	A
There	is	1	sign	bit.
There	are	
k
	=	5	exponent	bits.	The	exponent	bias	is	15.
There	are	
n
	=	3	fraction	bits.
2
.	
Format	B
There	is	1	sign	bit.
There	are	
k
	=	4	exponent	bits.	The	exponent	bias	is	7.
There	are	
n
	=	4	fraction	bits.
In	the	following	table,	you	are	given	some	bit	patterns	in	format	A,
and	your	task	is	to	convert	them	to	the	closest	value	in	format	B.	If
rounding	is	necessary	you	should	
round	toward
	+∞.	In	addition,
give	the	values	of	numbers	given	by	the	format	A	and	format	B	bit
patterns.	Give	these	as	whole	numbers	(e.g.,	17)	or	as	fractions
(e.g.,	17/64	or	17/2
).
Format	A
Format	B
Bits
Value
Bits
Value
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
__________
6
−
9
8
−
9
8

__________
__________
__________
2.89
We	are	running	programs	on	a	machine	where	values	of	type	
have	a	32-bit	two's-complement	representation.	Values	of	type
	use	the	32-bit	IEEE	format,	and	values	of	type	
	use
the	64-bit	IEEE	format.
We	generate	arbitrary	integer	values	
,	
,	and	
,	and	convert
them	to	values	of	type	
	follows:
For	each	of	the	following	C	expressions,	you	are	to	indicate
whether	or	not	the	expression	
always
	yields	1.	If	it	always	yields	1,
describe	the	underlying	mathematical	principles.	Otherwise,	give
an	example	of	arguments	that	make	it	yield	0.	Note	that	you
cannot	use	an	IA32	machine	running	
GCC
	
to	test	your	answers,

since	it	would	use	the	80-bit	extended-precision	representation	for
both	
	and	
.
A
.	
B
.	
C
.	
D
.	
E
.	
2.90
You	have	been	assigned	the	task	of	writing	a	C	function	to
compute	a	floating-point	representation	of	2
.	You	decide	that	the
best	way	to	do	this	is	to	directly	construct	the	IEEE	single-
precision	representation	of	the	result.	When	
x
	is	too	small,	your
routine	will	return	0.0.	When	
x
	is	too	large,	it	will	return	+∞.	Fill	in
the	blank	portions	of	the	code	that	follows	to	compute	the	correct
result.	Assume	the	
function	
	returns	a	floating-point	value
having	an	identical	bit	representation	as	its	unsigned	argument.
x

2.91
Around	250	B.C.,	the	Greek	mathematician	Archimedes	proved
that	
.	Had	he	had	access	to	a	computer	and	the
standard	library	
,	he	would	have	been	able	to	determine
that	the	single-precision	floating-point	approximation	of	
π
	has	the
223
71
<
π
<
22
7

hexadecimal	representation	
.	Of	course,	all	of	these	are
just	approximations,	since	
π
	is	not	rational.
A
.	
What	is	the	fractional	binary	number	denoted	by	this
floating-point	value?
B
.	
What	is	the	fractional	binary	representation	of	
?	
Hint:
See	
Problem	
2.83
.
C
.	
At	what	bit	position	(relative	to	the	binary	point)	do	these
two	approximations	to	
π
	diverge?
Bit-Level	Floating-Point	Coding
Rules
In	the	following	problems,	you	will	write	code	to	implement	floating-point
functions,	operating	directly	on	bit-level	representations	of	floating-point
numbers.	Your	code	should	exactly	replicate	the	conventions	for	IEEE
floating-point	operations,	including	using	round-to-even	mode	when
rounding	is	required.
To	this	end,	we	define	data	type	
	to	be	equivalent	to	
22
7

Rather	than	using	data	type	
	in	your	code,	you	will	use	
.
You	may	use	both	
	and	
	data	types,	including	unsigned	and
integer	constants	and	operations.	You	may	not	use	any	unions,	structs,
or	arrays.	Most	significantly,	you	may	not	use	any	floating-point	data
types,	operations,	or	constants.	Instead,	your	code	should	perform	the	bit
manipulations	that	implement	the	specified	floating-point	operations.
The	following	function	illustrates	the	use	of	these	coding	rules.	For
argument	
f
,	it	returns	±0	if	
f
	is	denormalized	(preserving	the	sign	of	
f
),	and
returns	
f
	otherwise.

2.92	♦♦
Following	the	bit-level	floating-point	coding	rules,	implement	the
function	with	the	following	prototype:
For	floating-point	number	
f
,	this	function	computes	–
f
.	If	
f
	is	
NaN
,
your	function	should	simply	return	
f
.
Test	your	function	by	evaluating	it	for	all	2
	values	of	argument	
and	comparing	the	result	to	what	would	be	obtained	using	your
machine's	floating-point	operations.
2.93	
Following	the	bit-level	floating-point	coding	rules,	implement
the	function	with	the	following	prototype:
For	floating-point	number	
f
,	this	function	computes	|
f
|.	If	
f
	is	
NaN
,
your	function	should	simply	return	
f
.
Test	your	function	by	evaluating	it	for	all	2
	values	of	argument	
and	comparing	the	result	to	what	would	be	obtained	using	your
machine's	floating-point	operations.
32
32

2.94
Following	the	bit-level	floating-point	coding	rules,	implement	the
function	with	the	following	prototype:
For	floating-point	number	
f
,	this	function	computes	2.0	·	
f
.	If	
f
	is
NaN
,	your	function	should	simply	return	
f
.
Test	your	function	by	evaluating	it	for	all	2
	values	of	argument	
and	comparing	the	result	to	what	would	be	obtained	using	your
machine's	floating-point	operations.
2.95
Following	the	bit-level	floating-point	coding	rules,	implement	the
function	with	the	following	prototype:
For	floating-point	number	
f
,	this	function	computes	0.5	·	
f
.	If	
f
	is
NaN
,	your	function	should	simply	return	
f
.
32
32

Test	your	function	by	evaluating	it	for	all	2
	values	of	argument	
and	comparing	the	result	to	what	would	be	obtained	using	your
machine's	floating-point	operations.
2.96
Following	the	bit-level	floating-point	coding	rules,	implement	the
function	with	the	following	prototype:
For	floating-point	number	
f
,	this	function	computes	(
)	
f
.	Your
function	should	round	toward	zero.	If	
f
	cannot	be	represented	as
an	integer	(e.g.,	it	is	out	of	range,	or	it	is	
NaN
),	then	the	function
should	return	
.
Test	your	function	by	evaluating	it	for	all	2
	values	of	argument	
and	comparing	the	result	to	what	would	be	obtained	using	your
machine's	floating-point	operations.
2.97
32
32

Following	the	bit-level	floating-point	coding	rules,	implement	the
function	with	the	following	prototype:
For	argument	
,	this	function	computes	the	bit-level
representation	of	
.
Test	your	function	by	evaluating	it	for	all	2
	values	of	argument	
and	comparing	the	result	to	what	would	be	obtained	using	your
machine's	floating-point	operations.
32

Solutions	to	Practice	Problems
Solution	to	Problem	
2.1	
(page
37
)
Understanding	the	relation	between	hexadecimal	and	binary	formats	will
be	important	once	we	start	looking	at	machine-level	programs.	The
method	for	doing	these	conversions	is	in	the	text,	but	it	takes	a	little
practice	to	become	familiar.
A
.	
	to	binary:
Hexadecimal
Binary
B
.	
Binary	
	to	hexadecimal:
Binary
Hexadecimal
C
.	
	to	binary:
Hexadecimal
Binary

D
.	
Binary	
	to	hexadecimal:
Binary
Hexadecimal
Solution	to	Problem	
2.2	
(page
37
)
This	problem	gives	you	a	chance	to	think	about	powers	of	2	and	their
hexadecimal	representations.
n
2
	(decimal)
2
	(hexadecimal)
9
512
19
524,288
14
16,384
16
65,536
17
131,072
5
32
7
128
n
n

Solution	to	Problem	
2.3	
(page
38
)
This	problem	gives	you	a	chance	to	try	out	conversions	between
hexadecimal	and	decimal	representations	for	some	smaller	numbers.	For
larger	ones,	it	becomes	much	more	convenient	and	reliable	to	use	a
calculator	or	conversion	program.
Decimal
Binary
Hexadecimal
0
0000	0000
167	=	10	·	16	+	7
1010	0111
62	=	3	·	16	+	14
0011	1110
188	=	11	·	16	+	12
1011	1100
3	·	16	+	7	=	55
0011	0111
8	·	16	+	8	=	136
1000	1000
15	·	16	+	3	=	243
1111	0011
5	·	16	+	2	=	82
0101	0010
10	·	16	+	12	=	172
1010	1100
14	·	16	+	7	=	231
1110	0111

Solution	to	Problem	
2.4	
(page
39
)
When	you	begin	debugging	machine-level	programs,	you	will	find	many
cases	where	some	simple	hexadecimal	arithmetic	would	be	useful.	You
can	always	convert	numbers	to	decimal,	perform	the	arithmetic,	and
convert	them	back,	but	being	able	to	work	directly	in	hexadecimal	is	more
efficient	and	informative.
A
.	
	Adding	
	to	hex	
	gives	
	with	a	carry	of
.
B
.	
	Subtracting	
	from	
	in	the	second	digit
position	requires	a	borrow	from	the	third.	Since	this	digit	is	
,	we
must	also	borrow	from	the	fourth	position.
C
.	
	Decimal	64	(2
)	equals	hexadecimal	
D
.	
	To	subtract	hex	
	(decimal	12)	from	hex
	(decimal	10),	we	borrow	16	from	the	second	digit,	giving	hex	
(decimal	14).	In	the	second	digit,	we	now	subtract	3	from	hex	
(decimal	13),	giving	hex	
	(decimal	10).
Solution	to	Problem	
2.5	
(page
48
)
6

This	problem	tests	your	understanding	of	the	byte	representation	of	data
and	the	two	different	byte	orderings.
Recall	that	
	enumerates	a	series	of	bytes	starting	from	the	one
with	lowest	address	and	working	toward	the	one	with	highest	address.
On	a	little-endian	machine,	it	will	list	the	bytes	from	least	significant	to
most.	On	a	big-endian	machine,	it	will	list	bytes	from	the	most	significant
byte	to	the	least.
Solution	to	Problem	
2.6	
(page
49
)
This	problem	is	another	chance	to	practice	hexadecimal	to	binary
conversion.	It	also	gets	you	thinking	about	integer	and	floating-point
representations.	We	will	explore	these	representations	in	more	detail
later	in	this	chapter.
A
.	
Using	the	notation	of	the	example	in	the	text,	we	write	the	two
strings	as	follows:

B
.	
With	the	second	word	shifted	two	positions	to	the	right	relative	to
the	first,	we	find	a	sequence	with	21	matching	bits.
C
.	
We	find	all	bits	of	the	integer	embedded	in	the	floating-point
number,	except	for	the	most	significant	bit	having	value	1.	Such	is
the	case	for	the	example	in	the	text	as	well.	In	addition,	the
floating-point	number	has	some	nonzero	high-order	bits	that	do
not	match	those	of	the	integer.
Solution	to	Problem	
2.7	
(page
49
)
It	prints	
.	Recall	also	that	the	library	routine	
does	not	count	the	terminating	null	character,	and	so	
	printed
only	through	the	character	
.
Solution	to	Problem	
2.8	
(page
51
)

This	problem	is	a	drill	to	help	you	become	more	familiar	with	Boolean
operations.
Operation
Result
a
[01101001]
b
[01010101]
~
a
[10010110]
~
b
[10101010]
a
	&	
b
[01000001]
a
	|	
b
[01111101]
a
	^	
b
[00111100]
Solution	to	Problem	
2.9	
(page
53
)
This	problem	illustrates	how	Boolean	algebra	can	be	used	to	describe
and	reason	about	real-world	systems.	We	can	see	that	this	color	algebra
is	identical	to	the	Boolean	algebra	over	bit	vectors	of	length	3.
A
.	
Colors	are	complemented	by	complementing	the	values	of	
R
,	
G
,
and	
B
.	From	this,	we	can	see	that	white	is	the	complement	of

black,	yellow	is	the	complement	of	blue,	magenta	is	the
complement	of	green,	and	cyan	is	the	complement	of	red.
B
.	
We	perform	Boolean	operations	based	on	a	bit-vector
representation	of	the	colors.	From	this	we	get	the	following:
Solution	to	Problem	
2.10	
(page
54
)
This	procedure	relies	on	the	fact	that	