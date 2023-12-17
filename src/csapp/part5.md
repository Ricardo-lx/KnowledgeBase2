requires	a	jump.	Similarly,	the	second	and	third	
	combinations	cannot
occur	at	the	same	time	as	a	load/use	hazard	or	a	mispredicted	branch.
Only	the	two	combinations	indicated	by	arrows	can	arise	simultaneously.
Combination	A	involves	a	not-taken	jump	instruction	in	the	execute	stage
and	a	
	instruction	in	the	decode	stage.	Setting	up	this	combination

requires	the	
	to	be	at	the	target	of	a	not-taken	branch.	The	pipeline
control	logic	should	detect	that	the	branch	was	mispredicted	and
therefore	cancel	the	
	instruction.
Practice	Problem	
4.37	
(solution	page	
492
)
Write	a	Y86-64	assembly-language	program	that	causes	combination	A
to	arise	and	determines	whether	the	control	logic	handles	it	correctly.
Combining	the	control	actions	for	the	combination	A	conditions	(
Figure
4.66
),	we	get	the	following	pipeline	control	actions	(assuming	that
either	a	bubble	or	a	stall	overrides	the	normal	case):
Pipeline	resister
Condition
F
D
E
M
W
Processing	
stall
bubble
normal
normal
normal
Mispredicted	branch
normal
bubble
bubble
normal
normal
Combination
stall
bubble
bubble
normal
normal
That	is,	it	would	be	handled	like	a	mispredicted	branch,	but	with	a	stall	in
the	fetch	stage.	Fortunately,	on	the	next	cycle,	the	PC	selection	logic	will
choose	the	address	of	the	instruction	following	the	jump,	rather	than	the
predicted	program	counter,	and	so	it	does	not	matter	what	happens	with
the	pipeline	register	F.	We	conclude	that	the	pipeline	will	correctly	handle
this	combination.

Combination	B	involves	a	load/use	hazard,	where	the	loading	instruction
sets	register	
	and	the	
	instruction	then	uses	this	register	as	a
source	operand,	since	it	must	pop	the	return	address	from	the	stack.	The
pipeline	control	logic	should	hold	back	the	
	instruction	in	the	decode
stage.
Practice	Problem	
4.38	
(solution	page	
492
)
Write	a	Y86-64	assembly-language	program	that	causes	combination	B
to	arise	and	completes	with	a	halt	instruction	if	the	pipeline	operates
correctly.
Combining	the	control	actions	for	the	combination	B	conditions	(
Figure
4.66
),	we	get	the	following	pipeline	control	actions:
Pipeline	resister
Condition
F
D
E
M
W
Processing	
stall
bubble
normal
normal
normal
Load/use	hazard
stall
stall
bubble
normal
normal
Combination
stall
bubble+stall
bubble
normal
normal
Desired
stall
stall
bubble
normal
normal
If	both	sets	of	actions	were	triggered,	the	control	logic	would	try	to	stall
the	
	instruction	to	avoid	the	load/use	hazard	but	also	inject	a	bubble
into	the	decode	stage	due	to	the	
	instruction.	Clearly,	we	do	not	want
the	pipeline	to	perform	both	sets	of	actions.	Instead,	we	want	it	to	just

take	the	actions	for	the	load/use	hazard.	The	actions	for	processing	the
	instruction	should	be	delayed	for	one	cycle.
This	analysis	shows	that	combination	B	requires	special	handling.	In	fact,
our	original	implementation	of	the	PIPE	control	logic	did	not	handle	this
combination	correctly.	Even	though	the	design	had	passed	many
simulation	tests,	it	had	a	subtle	bug	that	was	uncovered	only	by	the
analysis	we	have	just	shown.	When	a	program	having	combination	B	was
executed,	the	control	logic	would	set	both	the	bubble	and	the	stall	signals
for	pipeline	register	D	to	1.	This	example	shows	the	importance	of
systematic	analysis.	It	would	be	unlikely	to	uncover	this	bug	by	just
running	normal	programs.	If	left	undetected,	the	pipeline	would	not
faithfully	implement	the	ISA	behavior.
Control	Logic	Implementation
Figure	
4.68
	shows	the	overall	structure	of	the	pipeline	control	logic.
Based	on	signals	from	the	pipeline	registers	and	pipeline	stages,	the
control	logic	generates

Figure	
4.68	
PIPE	pipeline	control	logic.
This	logic	overrides	the	normal	flow	of	instructions	through	the	pipeline	to
handle	special	conditions	such	as	procedure	returns,	mispredicted
branches,	load/use	hazards,	and	program	exceptions.
stall	and	bubble	control	signals	for	the	pipeline	registers	and	also
determines	whether	the	condition	code	registers	should	be	updated.	We
can	combine	the	detection	conditions	of	
Figure	
4.64
	with	the	actions
of	
Figure	
4.66
	to	create	HCL	descriptions	for	the	different	pipeline
control	signals.
Pipeline	register	F	must	be	stalled	for	either	a	load/use	hazard	or	a	
instruction:

Practice	Problem	
4.39	
(solution	page	
493
)
Write	HCL	code	for	the	signal	D_stall	in	the	PIPE	implementation.
Pipeline	register	D	must	be	set	to	bubble	for	a	mispredicted	branch	or	a
	instruction.	As	the	analysis	in	the	preceding	section	shows,	however,
it	should	
not	inject	a	bubble	when	there	is	a	load/use	hazard	in
combination	with	a	
	instruction:
Practice	Problem	
4.40	
(solution	page	
493
)
Write	HCL	code	for	the	signal	E_bubble	in	the	PIPE	implementation.

Practice	Problem	
4.41	
(solution	page	
493
)
Write	HCL	code	for	the	signal	set_cc	in	the	PIPE	implementation.	This
should	only	occur	for	
	instructions,	and	should	consider	the	effects	of
program	exceptions.
Practice	Problem	
4.42	
(solution	page	
493
)
Write	HCL	code	for	the	signals	M_bubble	and	W_stall	in	the	PIPE
implementation.	The	latter	signal	requires	modifying	the	exception
condition	listed	in	
Figure	
4.64
.
This	covers	all	of	the	special	pipeline	control	signal	values.	In	the
complete	HCL	code	for	PIPE,	all	other	pipeline	control	signals	are	set	to
zero.
4.5.9	
Performance	Analysis
We	can	see	that	the	conditions	requiring	special	action	by	the	pipeline
control	logic	all	cause	our	pipeline	to	fall	short	of	the	goal	of	issuing	a
new	instruction	on	every	clock	cycle.	We	can	measure	this	inefficiency	by
determining	how	often	a	bubble	gets	injected	into	the	pipeline,	since
these	cause	unused	pipeline	cycles.	A	return	instruction	generates	three
bubbles,	a	load/use	hazard	generates	one,	and	a	mispredicted	branch
generates	two.	We	can	quantify	the	effect	these	penalties	have	on	the
overall	performance	by	computing	an	estimate	of	the	average	number	of
clock	cycles	PIPE	would	require	per	instruction	it	executes,	a	measure
known	as	the	CPI	(for	"cycles	per	instruction").	This	measure	is	the

reciprocal	of	the	average	throughput	of	the	pipeline,	but	with	time
measured	in	clock	cycles	rather	than	picoseconds.	It	is	a	useful	measure
of	the	architectural	efficiency	of	a	design.
If	we	ignore	the	performance	implications	of	exceptions	(which,	by
definition,	will	only	occur	rarely),	another	way	to	think	about	CPI	is	to
imagine	we	run	the
Aside	
Testing	the	design
As	we	have	seen,	there	are	many	ways	to	introduce	bugs	into	a
design,	even	for	a	simple	microprocessor.	With	pipelining,	there
are	many	subtle	interactions	between	the	instructions	at	different
pipeline	stages.	We	have	seen	that	many	of	the	design	challenges
involve	unusual	instructions	(such	as	popping	to	the	stack	pointer)
or	unusual	instruction	combinations	(such	as	a	not-taken	jump
followed	by	a	
).	We	also	see	that	exception	handling	adds	an
entirely	new	dimension	to	the	possible	pipeline	behaviors.	How,
then,	can	we	be	sure	that	our	design	is	correct?	For	hardware
manufacturers,	this	is	a	dominant	concern,	since	they	cannot
simply	report	an	error	and	have	users	download	code	patches
over	the	Internet.	Even	a	simple	logic	design	error	can	have
serious	consequences,	especially	as	microprocessors	are
increasingly	used	to	operate	systems	that	are	critical	to	our	lives
and	health,	such	as	automotive	antilock	braking	systems,	heart
pacemakers,	and	aircraft	control	systems.
Simply	simulating	a	design	while	running	a	number	of	"typical"
programs	is	not	a	sufficient	means	of	testing	a	system.	Instead,
thorough	testing	requires	devising	ways	of	systematically

generating	many	tests	that	will	exercise	as	many	different
instructions	and	instruction	combinations	as	possible.	In	creating
our	Y86-64	processor	designs,	we	also	devised	a	number	of
testing	scripts,	each	of	which	generates	many	different	tests,	runs
simulations	of	the	processor,	and	compares	the	resulting	register
and	memory	values	to	those	produced	by	our	
YIS
	
instruction	set
simulator.	Here	is	a	brief	description	of	the	scripts:
optest.	
Runs	49	tests	of	different	Y86-64	instructions	with
different	source	and	destination	registers
jtest.	
Runs	64	tests	of	the	different	jump	and	call	instructions,
with	different	combinations	of	whether	or	not	the	branches	are
taken
erntest.	
Runs	28	tests	of	the	different	conditional	move
instructions,	with	different	control	combinations
htest.	
Runs	600	tests	of	different	data	hazard	possibilities,	with
different	combinations	of	source	and	destination	instructions,
and	with	different	numbers	of	
	instructions	between	the
instruction	pairs
ctest.	
Tests	22	different	control	combinations,	based	on	an
analysis	similar	to	what	we	did	in	
Section	
4.5.8
etest.	
Tests	12	different	combinations	where	an	instruction
causes	an	exception	and	the	instructions	following	it	could	alter
the	programmer-visible	state
The	key	idea	of	this	testing	method	is	that	we	want	to	be	as
systematic	as	possible,	generating	tests	that	create	the	different

conditions	that	are	likely	to	cause	pipeline	errors.
processor	on	some	benchmark	program	and	observe	the	operation	of	the
execute	stage.	On	each	cycle,	the	execute	stage	either	(1)	processes	an
instruction	and	this	instruction	continues	through	the	remaining	stages	to
completion,	or	(2)	processes	a	bubble	injected	due	to	one	of	the	three
special	cases.	If	the	stage	processes	a	total	of	
C
	instructions	and	
C
bubbles,	then	the	processor	has	required	around	
C
	+	
C
	total	clock
cycles	to	execute	
C
	instructions.	We	say	"around"	because	we	ignore
Aside	
Formally	verifying	our	design
Even	when	a	design	passes	an	extensive	set	of	tests,	we	cannot
be	certain	that	it	will	operate	correctly	for	all	possible	programs.
The	number	of	possible	programs	we	could	test	is	unimaginably
large,	even	if	we	only	consider	tests	consisting	of	short	code
segments.	Newer	methods	of	
formal	verification
,	however,	hold
the	promise	that	we	can	have	tools	that	rigorously	consider	all
possible	behaviors	of	a	system	and	determine	whether	or	not
there	are	any	design	errors.
We	were	able	to	apply	formal	verification	to	an	earlier	version	of
our	Y86-64	processors	
[13]
.	We	set	up	a	framework	to	compare
the	behavior	of	the	pipelined	design	PIPE	to	the	unpipelined
version	SEQ.	That	is,	it	was	able	to	prove	that	for	an	arbitrary
machine-language	program,	the	two	processors	would	have
identical	effects	on	the	programmer-visible	state.	Of	course,	our
verifier	cannot	actually	run	all	possible	programs,	since	there	are
an	infinite	number	of	them.	Instead,	it	uses	a	form	of	proof	by
induction,	showing	a	consistency	between	the	two	processors	on
a	cycle-by-cycle	basis.	Carrying	out	this	analysis	requires
i
b
i
b
i

reasoning	about	the	hardware	using	
symbolic	methods
	in	which
we	consider	all	program	values	to	be	arbitrary	integers,	and	we
abstract	the	ALU	as	a	sort	of	"black	box,"	computing	some
unspecified	function	over	its	arguments.	We	assume	only	that	the
ALUs	for	SEQ	and	PIPE	compute	identical	functions.
We	used	the	HCL	descriptions	of	the	control	logic	to	generate	the
control	logic	for	our	symbolic	processor	models,	and	so	we	could
catch	any	bugs	in	the	HCL	code.	Being	able	to	show	that	SEQ	and
PIPE	are	identical	does	not	guarantee	that	either	of	them	faithfully
implements	the	instruction	set	architecture.	However,	it	would
uncover	any	bug	due	to	an	incorrect	pipeline	design,	and	this	is
the	major	source	of	design	errors.
In	our	experiments,	we	verified	not	only	a	version	of	PIPE	similar
to	the	one	we	have	presented	in	this	chapter	but	also	several
variants	that	we	give	as	homework	problems,	in	which	we	add
more	instructions,	modify	the	hardware	capabilities,	or	use
different	branch	prediction	strategies.	Interestingly,	we	found	only
one	bug	in	all	of	our	designs,	involving	control	combination	B
(described	in	
Section	
4.5.8
)	for	our	solution	to	the	variant
described	in	
Problem	
4.58
.	This	exposed	a	weakness	in	our
testing	regime	that	caused	us	to	add	additional	cases	to	the	ctest
testing	script.
Formal	verification	is	still	in	an	early	stage	of	development.	The
tools	are	often	difficult	to	use,	and	they	do	not	have	the	capacity	to
verify	large-scale	designs.	We	were	able	to	verify	our	processors
in	part	because	of	their	relative	simplicity.	Even	then,	it	required
several	weeks	of	effort	and	multiple	runs	of	the	tools,	each
requiring	up	to	8	hours	of	computer	time.	This	is	an	active	area	of

research,	with	some	tools	becoming	commercially	available	and
some	in	use	at	companies	such	as	Intel,	AMD,	and	IBM.
the	cycles	required	to	start	the	instructions	flowing	through	the	pipeline.
We	can	then	compute	the	CPI	for	this	benchmark	as	follows:
That	is,	the	CPI	equals	1.0	plus	a	penalty	term	
C
/
C
	indicating	the
average	number	of	bubbles	injected	per	instruction	executed.	Since	only
three	different	instruction	types	can	cause	a	bubble	to	be	injected,	we
can	break	this	penalty	term	into	three	components:
Web	Aside	ARCH:VLOG	
Verilog
implementation	of	a	pipelined	Y86-64
processor
As	we	have	mentioned,	modern	logic	design	involves	writing
textual	representations	of	hardware	designs	in	a	
hardware
description	language.
	The	design	can	then	be	tested	by	both
simulation	and	a	variety	of	formal	verification	tools.	Once	we	have
confidence	in	the	design,	we	can	use	
logic	synthesis
	tools	to
translate	the	design	into	actual	logic	circuits.
We	have	developed	models	of	our	Y86-64	processor	designs	in
the	Verilog	hardware	description	language.	These	designs
combine	modules	implementing	the	basic	building	blocks	of	the
processor,	along	with	control	logic	generated	directly	from	the	HCL
descriptions.	We	have	been	able	to	synthesize	some	of	these
CPI=
C
i
+
C
b
C
i
=
1.0
+
C
b
C
i
b
i

designs,	download	the	logic	circuit	descriptions	onto	field-
programmable	gate	array	(FPGA)	hardware,	and	run	the
processors	on	actual	programs.
where	
lp
	(for	"load	penalty")	is	the	average	frequency	with	which	bubbles
are	injected	while	stalling	for	load/use	hazards,	
mp
	(for	"mispredicted
branch	penalty")	is	the	average	frequency	with	which	bubbles	are
injected	when	canceling	instructions	due	to	mispredicted	branches,	and
rp
	(for	"return	penalty")	is	the	average	frequency	with	which	bubbles	are
injected	while	stalling	for	
	instructions.	Each	of	these	penalties
indicates	the	total	number	of	bubbles	injected	for	the	stated	reason
(some	portion	of	
C
)	divided	by	the	total	number	of	instructions	that	were
executed	(
C
.)
To	estimate	each	of	these	penalties,	we	need	to	know	how	frequently	the
relevant	instructions	(load,	conditional	branch,	and	return)	occur,	and	for
each	of	these	how	frequently	the	particular	condition	arises.	Let	us	pick
the	following	set	of	frequencies	for	our	CPI	computation	(these	are
comparable	to	measurements	reported	in	
[44]
	and	
[46]
):
Load	instructions	(
	and	
)	account	for	25%	of	all
instructions	executed.	Of	these,	20%	cause	load/use	hazards.
Conditional	branches	account	for	20%	of	all	instructions	executed.	Of
these,	60%	are	taken	and	40%	are	not	taken.
Return	instructions	account	for	2%	of	all	instructions	executed.
We	can	therefore	estimate	each	of	our	penalties	as	the	product	of	the
frequency	of	the	instruction	type,	the	frequency	the	condition	arises,	and
CPI=1
.0+
l
p
+
m
p
+
r
p
b
i

the	number	of	bubbles	that	get	injected	when	the	condition	occurs:
Cause
Name
Instruction
frequency
Condition
frequency
Bubbles
Product
Load/use
lp
0.25
0.20
1
0.05
Mispredict
mp
0.20
0.40
2
0.16
Return
rp
0.02
1.00
3
		0.06		
Total
penalty
0.27
The	sum	of	the	three	penalties	is	0.27,	giving	a	CPI	of	1.27.
Our	goal	was	to	design	a	pipeline	that	can	issue	one	instruction	per
cycle,	giving	a	CPI	of	1.0.	We	did	not	quite	meet	this	goal,	but	the	overall
performance	is	still	quite	good.	We	can	also	see	that	any	effort	to	reduce
the	CPI	further	should	focus	on	mispredicted	branches.	They	account	for
0.16	of	our	total	penalty	of	0.27,	because	conditional	branches	are
common,	our	prediction	strategy	often	fails,	and	we	cancel	two
instructions	for	every	misprediction.
Practice	Problem	
4.43	
(solution	page	
494
)
Suppose	we	use	a	branch	prediction	strategy	that	achieves	a	success
rate	of	65%,	such	as	backward	taken,	forward	not	taken	(BTFNT),	as
described	in	
Section	
4.5.4
.	What	would	be	the	impact	on	CPI,
assuming	all	of	the	other	frequencies	are	not	affected?

Practice	Problem	
4.44	
(solution	page	
494
)
Let	us	analyze	the	relative	performance	of	using	conditional	data
transfers	versus	conditional	control	transfers	for	the	programs	you	wrote
for	Problems	4.5	and	4.6.	Assume	that	we	are	using	these	programs	to
compute	the	sum	of	the	absolute	values	of	a	very	long	array,	and	so	the
overall	performance	is	determined	largely	by	the	number	of	cycles
required	by	the	inner	loop.	Assume	that	our	jump	instructions	are
predicted	as	being	taken,	and	that	around	50%	of	the	array	values	are
positive.
A
.	
On	average,	how	many	instructions	are	executed	in	the	inner
loops	of	the	two	programs?
B
.	
On	average,	how	many	bubbles	would	be	injected	into	the	inner
loops	of	the	two	programs?
C
.	
What	is	the	average	number	of	clock	cycles	required	per	array
element	for	the	two	programs?
4.5.10	
Unfinished	Business
We	have	created	a	structure	for	the	PIPE	pipelined	microprocessor,
designed	the	control	logic	blocks,	and	implemented	pipeline	control	logic
to	handle	special	cases	where	normal	pipeline	flow	does	not	suffice.	Still,
PIPE	lacks	several	key	features	that	would	be	required	in	an	actual
microprocessor	design.	We	highlight	a	few	of	these	and	discuss	what
would	be	required	to	add	them.

Multicycle	Instructions
All	of	the	instructions	in	the	Y86-64	instruction	set	involve	simple
operations	such	as	adding	numbers.	These	can	be	processed	in	a	single
clock	cycle	within	the	execute	stage.	In	a	more	complete	instruction	set,
we	would	also	need	to	implement	instructions	requiring	more	complex
operations	such	as	integer	multiplication	and	
division	and	floating-point
operations.	In	a	medium-performance	processor	such	as	PIPE,	typical
execution	times	for	these	operations	range	from	3	or	4	cycles	for	floating-
point	addition	up	to	64	cycles	for	integer	division.	To	implement	these
instructions,	we	require	both	additional	hardware	to	perform	the
computations	and	a	mechanism	to	coordinate	the	processing	of	these
instructions	with	the	rest	of	the	pipeline.
One	simple	approach	to	implementing	multicycle	instructions	is	to	simply
expand	the	capabilities	of	the	execute	stage	logic	with	integer	and
floating-point	arithmetic	units.	An	instruction	remains	in	the	execute	stage
for	as	many	clock	cycles	as	it	requires,	causing	the	fetch	and	decode
stages	to	stall.	This	approach	is	simple	to	implement,	but	the	resulting
performance	is	not	very	good.
Better	performance	can	be	achieved	by	handling	the	more	complex
operations	with	special	hardware	functional	units	that	operate
independently	of	the	main	pipeline.	Typically,	there	is	one	functional	unit
for	performing	integer	multiplication	and	division,	and	another	for
performing	floating-point	operations.	As	an	instruction	enters	the	decode
stage,	it	can	be	
issued
	to	the	special	unit.	While	the	unit	performs	the
operation,	the	pipeline	continues	processing	other	instructions.	Typically,

the	floating-point	unit	is	itself	pipelined,	and	thus	multiple	operations	can
execute	concurrently	in	the	main	pipeline	and	in	the	different	units.
The	operations	of	the	different	units	must	be	synchronized	to	avoid
incorrect	behavior.	For	example,	if	there	are	data	dependencies	between
the	different	operations	being	handled	by	different	units,	the	control	logic
may	need	to	stall	one	part	of	the	system	until	the	results	from	an
operation	handled	by	some	other	part	of	the	system	have	been
completed.	Often,	different	forms	of	forwarding	are	used	to	convey
results	from	one	part	of	the	system	to	other	parts,	just	as	we	saw
between	the	different	stages	of	PIPE.	The	overall	design	becomes	more
complex	than	we	have	seen	with	PIPE,	but	the	same	techniques	of
stalling,	forwarding,	and	pipeline	control	can	be	used	to	make	the	overall
behavior	match	the	sequential	ISA	model.
Interfacing	with	the	Memory	System
In	our	presentation	of	PIPE,	we	assumed	that	both	the	instruction	fetch
unit	and	the	data	memory	could	read	or	write	any	memory	location	in	one
clock	cycle.	We	also	ignored	the	possible	hazards	caused	by	self-
modifying	code	where	one	instruction	writes	to	the	region	of	memory	from
which	later	instructions	are	fetched.	Furthermore,	we	reference	memory
locations	according	to	their	virtual	addresses,	and	these	require	a
translation	into	physical	addresses	before	the	actual	read	or	write
operation	can	be	performed.	Clearly,	it	is	unrealistic	to	do	all	of	this
processing	in	a	single	clock	cycle.	Even	worse,	the	memory	values	being
accessed	may	reside	on	disk,	requiring	millions	of	clock	cycles	to	read
into	the	processor	memory.

As	will	be	discussed	in	
Chapters	
6
	and	
9
,	the	memory	system	of	a
processor	uses	a	combination	of	multiple	hardware	memories	and
operating	system	software	to	manage	the	virtual	memory	system.	The
memory	system	is	organized	as	a	hierarchy,	with	faster	but	smaller
memories	holding	a	subset	of	the	memory	being	
backed	up	by	slower
and	larger	memories.	At	the	level	closest	to	the	processor,	the	
cache
memories	provide	fast	access	to	the	most	heavily	referenced	memory
locations.	A	typical	processor	has	two	first-level	caches—one	for	reading
instructions	and	one	for	reading	and	writing	data.	Another	type	of	cache
memory,	known	as	a	
translation	look-aside	buffer
,	or	TLB,	provides	a	fast
translation	from	virtual	to	physical	addresses.	Using	a	combination	of
TLBs	and	caches,	it	is	indeed	possible	to	read	instructions	and	read	or
write	data	in	a	single	clock	cycle	most	of	the	time.	Thus,	our	simplified
view	of	memory	referencing	by	our	processors	is	actually	quite
reasonable.
Although	the	caches	hold	the	most	heavily	referenced	memory	locations,
there	will	be	times	when	a	cache	
miss
	occurs,	where	some	reference	is
made	to	a	location	that	is	not	held	in	the	cache.	In	the	best	case,	the
missing	data	can	be	retrieved	from	a	higher-level	cache	or	from	the	main
memory	of	the	processor,	requiring	3	to	20	clock	cycles.	Meanwhile,	the
pipeline	simply	stalls,	holding	the	instruction	in	the	fetch	or	memory	stage
until	the	cache	can	perform	the	read	or	write	operation.	In	terms	of	our
pipeline	design,	this	can	be	implemented	by	adding	more	stall	conditions
to	the	pipeline	control	logic.	A	cache	miss	and	the	consequent
synchronization	with	the	pipeline	is	handled	completely	by	hardware,
keeping	the	time	required	down	to	a	small	number	of	clock	cycles.
In	some	cases,	the	memory	location	being	referenced	is	actually	stored
in	the	disk	or	nonvolatile	memory.	When	this	occurs,	the	hardware

signals	a	
page	fault
	exception.	Like	other	exceptions,	this	will	cause	the
processor	to	invoke	the	operating	system's	exception	handler	code.	This
code	will	then	set	up	a	transfer	from	the	disk	to	the	main	memory.	Once
this	completes,	the	operating	system	will	return	to	the	original	program,
where	the	instruction	causing	the	page	fault	will	be	re-executed.	This
time,	the	memory	reference	will	succeed,	although	it	might	cause	a	cache
miss.	Having	the	hardware	invoke	an	operating	system	routine,	which
then	returns	control	back	to	the	hardware,	allows	the	hardware	and
system	software	to	cooperate	in	the	handling	of	page	faults.	Since
accessing	a	disk	can	require	millions	of	clock	cycles,	the	several
thousand	cycles	of	processing	performed	by	the	OS	page	fault	handler
has	little	impact	on	performance.
From	the	perspective	of	the	processor,	the	combination	of	stalling	to
handle	short-duration	cache	misses	and	exception	handling	to	handle
long-duration	page	faults	takes	care	of	any	unpredictability	in	memory
access	times	due	to	the	structure	of	the	memory	hierarchy.

4.6	
Summary
We	have	seen	that	the	instruction	set	architecture,	or	ISA,	provides	a
layer	of	abstraction	between	the	behavior	of	a	processor—in	terms	of	the
set	of	instructions	and	their	encodings—and	how	the	processor	is
implemented.	The	ISA	provides	a	very	sequential	view	of	program
execution,	with	one	instruction	executed	to	completion	before	the	next
one	begins.
Aside	
State-of-the-art	microprocessor
design
A	five-stage	pipeline,	such	as	we	have	shown	with	the	PIPE
processor,	represented	the	state	of	the	art	in	processor	design	in
the	mid-1980s.	The	prototype	RISC	processor	developed	by
Patterson's	research	group	at	Berkeley	formed	the	basis	for	the
first	SPARC	processor,	developed	by	Sun	Microsystems	in	1987.
The	processor	developed	by	Hennessy's	research	group	at
Stanford	was	commercialized	by	MIPS	Technologies	(a	company
founded	by	Hennessy)	in	1986.	Both	of	these	used	five-stage
pipelines.	The	Intel	i486	processor	also	uses	a	five-stage	pipeline,
although	with	a	different	partitioning	of	responsibilities	among	the
stages,	with	two	decode	stages	and	a	combined	execute/memory
stage	
[27]
.

These	pipelined	designs	are	limited	to	a	throughput	of	at	most	one
instruction	per	clock	cycle.	The	CPI	(for	"cycles	per	instruction")
measure	described	in	
Section	
4.5.9
	can	never	be	less	than	1.0.
The	different	stages	can	only	process	one	instruction	at	a	time.
More	recent	processors	support	
superscalar
	operation,	meaning
that	they	can	achieve	a	CPI	less	than	1.0	by	fetching,	decoding,
and	executing	multiple	instructions	in	parallel.	As	superscalar
processors	have	become	widespread,	the	accepted	performance
measure	has	shifted	from	CPI	to	its	reciprocal—the	average
number	of	instructions	executed	per	cycle,	or	IPC.	It	can	exceed
1.0	for	superscalar	processors.	The	most	advanced	designs	use	a
technique	known	as	
out-of-order
	execution	to	execute	multiple
instructions	in	parallel,	possibly	in	a	totally	different	order	than
they	occur	in	the	program,	while	preserving	the	overall	behavior
implied	by	the	sequential	ISA	model.	This	form	of	execution	is
described	in	
Chapter	
5
	as	part	of	our	discussion	of	program
optimization.
Pipelined	processors	are	not	just	historical	artifacts,	however.	The
majority	of	processors	sold	are	used	in	embedded	systems,
controlling	automotive	functions,	consumer	products,	and	other
devices	where	the	processor	is	not	directly	visible	to	the	system
user.	In	these	applications,	the	simplicity	of	a	pipelined	processor,
such	as	the	one	we	have	explored	in	this	chapter,	reduces	its	cost
and	power	requirements	compared	to	higher-performance	models.
More	recently,	as	multicore	processors	have	gained	a	following,
some	have	argued	that	we	could	get	more	overall	computing
power	by	integrating	many	simple	processors	on	a	single	chip
rather	than	a	smaller	number	of	more	complex	ones.	This	strategy
is	sometimes	referred	to	as	"many-core"	processors	
[10]
.

We	defined	the	Y86-64	instruction	set	by	starting	with	the	x86-64
instructions	and	simplifying	the	data	types,	address	modes,	and
instruction	encoding	considerably.	The	resulting	ISA	has	attributes	of	both
RISC	and	CISC	instruction	sets.	We	then	organized	the	processing
required	for	the	different	instructions	into	a	series	of	five	stages,	where
the	operations	at	each	stage	vary	according	to	the	instruction	being
executed.	From	this,	we	constructed	the	SEQ	processor,	in	which	an
entire	instruction	is	executed	every	clock	cycle	by	having	it	flow	through
all	five	stages.
Pipelining	improves	the	throughput	performance	of	a	system	by	letting
the	different	stages	operate	concurrently.	At	any	given	time,	multiple
operations	are	being	processed	by	the	different	stages.	In	introducing	this
concurrency,	we	must	be	careful	to	provide	the	same	program-level
behavior	as	would	a	sequential	execution	of	the	program.	We	introduced
pipelining	by	reordering	parts	of	SEQ	to	get	SEQ+	and	then	adding
pipeline	registers	to	create	the	PIPE—	pipeline.
Web	Aside	ARCH:HCL	
HCL
descriptions	of	Y86-64	processors
In	this	chapter,	we	have	looked	at	portions	of	the	HCL	code	for
several	simple	logic	designs	and	for	the	control	logic	for	Y86-64
processors	SEQ	and	PIPE.	For	reference,	we	provide
documentation	of	the	HCL	language	and	complete	HCL
descriptions	for	the	control	logic	of	the	two	processors.	Each	of
these	descriptions	requires	only	five	to	seven	pages	of	HCL	code,
and	it	is	worthwhile	to	study	them	in	their	entirety.

We	enhanced	the	pipeline	performance	by	adding	forwarding	logic	to
speed	the	sending	of	a	result	from	one	instruction	to	another.	Several
special	cases	require	additional	pipeline	control	logic	to	stall	or	cancel
some	of	the	pipeline	stages.
Our	design	included	rudimentary	mechanisms	to	handle	exceptions,
where	we	make	sure	that	only	instructions	up	to	the	excepting	instruction
affect	the	programmer-visible	state.	Implementing	a	complete	handling	of
exceptions	would	be	significantly	more	challenging.	Properly	handling
exceptions	gets	even	more	complex	in	systems	that	employ	greater
degrees	of	pipelining	and	parallelism.
In	this	chapter,	we	have	learned	several	important	lessons	about
processor	design:
Managing	complexity	is	a	top	priority.	
We	want	to	make	optimum
use	of	the	hardware	resources	to	get	maximum	performance	at
minimum	cost.	We	did	this	by	creating	a	very	simple	and	uniform
framework	for	processing	all	of	the	different	instruction	types.	With
this	framework,	we	could	share	the	hardware	units	among	the	logic
for	processing	the	different	instruction	types.
We	do	not	need	to	implement	the	ISA	directly.	
A	direct
implementation	of	the	ISA	would	imply	a	very	sequential	design.	To
achieve	higher	performance,	we	want	to	exploit	the	ability	in	hardware
to	perform	many	operations	simultaneously.	This	led	to	the	use	of	a
pipelined	design.	By	careful	design	and	analysis,	we	can	handle	the
various	pipeline	hazards,	so	that	the	overall	effect	of	running	a
program	exactly	matches	what	would	be	obtained	with	the	ISA	model.
Hardware	designers	must	be	meticulous.	
Once	a	chip	has	been
fabricated,	it	is	nearly	impossible	to	correct	any	errors.	It	is	very

important	to	get	the	design	right	on	the	first	try.	This	means	carefully
analyzing	different	instruction	types	and	combinations,	even	ones	that
do	not	seem	to	make	sense,	such	as	popping	to	the	stack	pointer.
Designs	must	be	thoroughly	tested	with	systematic	simulation	test
programs.	In	developing	the	control	logic	for	PIPE,	our	design	had	a
subtle	bug	that	was	uncovered	only	after	a	careful	and	systematic
analysis	of	control	combinations.
4.6.1	
Y86-64	Simulators
The	lab	materials	for	this	chapter	include	simulators	for	the	SEQ	and
PIPE	processors.	Each	simulator	has	two	versions:
The	GUI	(graphic	user	interface)	version	displays	the	memory,
program	code,	and	processor	state	in	graphic	windows.	This	provides
a	way	to	readily	see	how	the	instructions	flow	through	the	processors.
The	control	panel	also	allows	you	to	reset,	single-step,	or	run	the
simulator	interactively.
The	text	version	runs	the	same	simulator,	but	it	only	displays
information	by	printing	to	the	terminal.	This	version	is	not	as	useful	for
debugging,	but	it	allows	automated	testing	of	the	processor.
The	control	logic	for	the	simulators	is	generated	by	translating	the	HCL
declarations	of	the	logic	blocks	into	C	code.	This	code	is	then	compiled
and	linked	with	the	rest	of	the	simulation	code.	This	combination	makes	it
possible	for	you	to	test	out	variants	of	the	original	designs	using	the
simulators.	Testing	scripts	are	also	available	that	thoroughly	exercise	the
different	instructions	and	the	different	hazard	possibilities.

Bibliographic	Notes
For	those	interested	in	learning	more	about	logic	design,	the	Katz	and
Borriello	logic	design	textbook	
[58]
	is	a	standard	introductory	text,
emphasizing	the	use	of	hardware	description	languages.	Hennessy	and
Patterson's	computer	architecture	textbook	
[46]
	provides	extensive
coverage	of	processor	design,	including	both	simple	pipelines,	such	as
the	one	we	have	presented	here,	and	advanced	processors	that	execute
more	instructions	in	parallel.	Shriver	and	Smith	
[101]
	give	a	very
thorough	presentation	of	an	Intel-compatible	x86-64	processor
manufactured	by	AMD.

Homework	Problems
4.45
In	
Section	
3.4.2
,	the	x86-64	
	instruction	was	described	as
decrementing	the	stack	pointer	and	then	storing	the	register	at	the	stack
pointer	location.	So,	if	we	had	an	instruction	of	the	form	
	
REG
,	for
some	register	
REG
,	it	would	be	equivalent	to	the	code	sequence
A
.	
In	light	of	analysis	done	in	
Practice	Problem	
4.7
,	does	this
code	sequence	correctly	describe	the	behavior	of	the	instruction
	
?	Explain.
B
.	
How	could	you	rewrite	the	code	sequence	so	that	it	correctly
describes	both	the	cases	where	
REG
	is	
	as	well	as	any	other
register?
4.46

In	
Section	
3.4.2
,	the	x86-64	
	instruction	was	described	as
copying	the	result	from	the	top	of	the	stack	to	the	destination	register	and
then	incrementing	the	stack	pointer.	So,	if	we	had	an	instruction	of	the
form	
	
REG
,	it	would	be	equivalent	to	the	code	sequence
A
.	
In	light	of	analysis	done	in	
Practice	Problem	
4.8
,	does	this
code	sequence	correctly	describe	the	behavior	of	the	instruction
	
?	Explain.
B
.	
How	could	you	rewrite	the	code	sequence	so	that	it	correctly
describes	both	the	cases	where	
REG
	is	
	as	well	as	any	other
register?
4.47
Your	assignment	will	be	to	write	a	Y86-64	program	to	perform	bubblesort.
For	reference,	the	following	C	function	implements	bubblesort	using	array
referencing:

A
.	
Write	and	test	a	C	version	that	references	the	array	elements	with
pointers,	rather	than	using	array	indexing.
B
.	
Write	and	test	a	Y86-64	program	consisting	of	the	function	and
test	code.	You	may	find	it	useful	to	pattern	your	implementation
after	x86-64	code	generated	by	compiling	your	C	code.	Although
pointer	comparisons	are	normally	done	using	unsigned	arithmetic,
you	can	use	signed	arithmetic	for	this	exercise.
4.48
Modify	the	code	you	wrote	for	
Problem	
4.47
	to	implement	the	test	and
swap	in	the	bubblesort	function	(lines	6-11)	using	no	jumps	and	at	most
three	conditional	moves.
4.49

Modify	the	code	you	wrote	for	
Problem	
4.47
	to	implement	the	test	and
swap	in	the	bubblesort	function	(lines	6-11)	using	no	jumps	and	just	one
conditional	move.
4.50
In	
Section	
3.6.8
,	we	saw	that	a	common	way	to	implement	
statements	is	to	create	a	set	of	code	blocks	and	then	index	those	blocks
using	a	jump	table.	Consider

Figure	
4.69	
Switch	statements	can	be	translated	into	Y86-64	code.
This	requires	implementation	of	a	jump	table.
the	C	code	shown	in	
Figure	
4.69
	for	a	function	
,	along	with
associated	test	code.
Implement	
	in	Y86-64	using	a	jump	table.	Although	the	Y86-64
instruction	set	does	not	include	an	indirect	jump	instruction,	you	can	get
the	same	effect	by	pushing	a	computed	address	onto	the	stack	and	then

executing	the	
	
instruction.	Implement	test	code	similar	to	what	is
shown	in	C	to	demonstrate	that	your	implementation	of	
	will
handle	both	the	cases	handled	explicitly	as	well	as	those	that	trigger	the
	case.
4.51
Practice	Problem	
4.3
	introduced	the	
	instruction	to	add
immediate	data	to	a	register.	Describe	the	computations	performed	to
implement	this	instruction.	Use	the	computations	for	
	and	
(
Figure	
4.18
)	as	a	guide.
4.52
The	file	
	contains	the	HCL	description	for	SEQ,	along	with
the	declaration	of	a	constant	
	having	hexadecimal	value	C,	the
instruction	code	for	
.	Modify	the	HCL	descriptions	of	the	control
logic	blocks	to	implement	the	
	instruction,	as	described	in	
Practice
Problem	
4.3
	and	
Problem	
4.51
.	See	the	lab	material	for	directions
on	how	to	generate	a	simulator	for	your	solution	and	how	to	test	it.
4.53

Suppose	we	wanted	to	create	a	lower-cost	pipelined	processor	based	on
the	structure	we	devised	for	PIPE—	(
Figure	
4.41
),	without	any
bypassing.	This	design	would	handle	all	data	dependencies	by	stalling
until	the	instruction	generating	a	needed	value	has	passed	through	the
write-back	stage.
The	file	
	contains	a	modified	version	of	the	HCL	code	for
PIPE	in	which	the	bypassing	logic	has	been	disabled.	That	is,	the	signals
	and	
	are	simply	declared	as	follows:
Modify	the	pipeline	control	logic	at	the	end	of	this	file	so	that	it	correctly
handles	all	possible	control	and	data	hazards.	As	part	of	your	design
effort,	you	should	analyze	the	different	combinations	of	control	cases,	as
we	did	in	the	design	of	the	pipeline	control	logic	for	PIPE.	You	will	find
that	many	different	combinations	can	occur,	since	many	more	conditions
require	the	pipeline	to	stall.	Make	sure	your	control	logic	handles	each
combination	correctly.	See	the	lab	material	for	directions	on	how	to
generate	a	simulator	for	your	solution	and	how	to	test	it.

4.54
The	
	contains	a	copy	of	the	PIPE	HCL	description,
along	with	a	declaration	of	the	constant	value	
.	Modify	this	file	to
implement	the	
	instruction,	as	described	in	
Practice	Problem	
4.3
and	
Problem	
4.51
.	See	the	lab	
material	for	directions	on	how	to
generate	a	simulator	for	your	solution	and	how	to	test	it.
4.55
The	file	
	contains	a	copy	of	the	HCL	code	for	PIPE,	plus	a
declaration	of	the	constant	
	with	value	0,	the	function	code	for	an
unconditional	jump	instruction.	Modify	the	branch	prediction	logic	so	that
it	predicts	conditional	jumps	as	being	not	taken	while	continuing	to	predict
unconditional	jumps	and	
	as	being	taken.	You	will	need	to	devise	a
way	to	get	valC,	the	jump	target	address,	to	pipeline	register	M	to	recover
from	mispredicted	branches.	See	the	lab	material	for	directions	on	how	to
generate	a	simulator	for	your	solution	and	how	to	test	it.
4.56
The	file	
	contains	a	copy	of	the	HCL	code	for	PIPE,	plus	a
declaration	of	the	constant	
	with	value	0,	the	function	code	for	an
unconditional	jump	instruction.	Modify	the	branch	prediction	logic	so	that

it	predicts	conditional	jumps	as	being	taken	when	
	(backward
branch)	and	as	being	not	taken	when	
	(forward	branch).
(Since	Y86-64	does	not	support	unsigned	arithmetic,	you	should
implement	this	test	using	a	signed	comparison.)	Continue	to	predict
unconditional	jumps	and	
	as	being	taken.	You	will	need	to	devise	a
way	to	get	both	
	and	
	to	pipeline	register	M	to	recover	from
mispredicted	branches.	See	the	lab	material	for	directions	on	how	to
generate	a	simulator	for	your	solution	and	how	to	test	it.
4.57
In	our	design	of	PIPE,	we	generate	a	stall	whenever	one	instruction
performs	a	
load
,	reading	a	value	from	memory	into	a	register,	and	the
next	instruction	has	this	register	as	a	source	operand.	When	the	source
gets	used	in	the	execute	stage,	this	stalling	is	the	only	way	to	avoid	a
hazard.	For	cases	where	the	second	instruction	stores	the	source
operand	to	memory,	such	as	with	an	
	or	
	instruction,	this
stalling	is	not	necessary.	Consider	the	following	code	examples:

In	lines	1	and	2,	the	
	instruction	reads	a	value	from	memory	into
,	and	the	
	instruction	then	pushes	this	value	onto	the	stack.
Our	design	for	PIPE	would	stall	the	
	instruction	to	avoid	a	load/use
hazard.	Observe,	however,	that	the	value	of	
	is	not	required	by	the
	instruction	until	it	reaches	the	memory	stage.	We	can	add	an
additional	bypass	path,	as	diagrammed	in	
Figure	
4.70
,	to	forward	the
memory	output	(signal	m_valM)	to	the	valA	field	in	pipeline	register	M.	On
the	next	clock	cycle,	this	forwarded	value	can	then	be	written	to	memory.
This	technique	is	known	as	
load	forwarding.
Note	that	the	second	example	(lines	4	and	5)	in	the	code	sequence
above	cannot	make	use	of	load	forwarding.	The	value	loaded	by	the	
instruction	is


Figure	
4.70	
Execute	and	memory	stages	capable	of	load	forwarding.
By	adding	a	bypass	path	from	the	memory	output	to	the	source	of	valA	in
pipeline	register	M,	we	can	use	forwarding	rather	than	stalling	for	one
form	of	load/use	hazard.	This	is	the	subject	of	
Problem	
4.57
.
used	as	part	of	the	address	computation	by	the	next	instruction,	and	this
value	is	required	in	the	execute	stage	rather	than	the	memory	stage.
A
.	
Write	a	logic	formula	describing	the	detection	condition	for	a
load/use	hazard,	similar	to	the	one	given	in	
Figure	
4.64
,	except
that	it	will	not	cause	a	stall	in	cases	where	load	forwarding	can	be
used.
B
.	
The	file	
	contains	a	modified	version	of	the	control
logic	for	PIPE.	It	contains	the	definition	of	a	signal	
	to
implement	the	block	labeled	"Fwd	A"	in	
Figure	
4.70
.	It	also	has
the	conditions	for	a	load/use	hazard	in	the	pipeline	control	logic	set
to	zero,	and	so	the	pipeline	control	logic	will	not	detect	any	forms
of	load/use	hazards.	Modify	this	HCL	description	to	implement
load	forwarding.	See	the	lab	material	for	directions	on	how	to
generate	a	simulator	for	your	solution	and	how	to	test	it.
4.58
Our	pipelined	design	is	a	bit	unrealistic	in	that	we	have	two	write	ports	for
the	register	file,	but	only	the	
	instruction	requires	two	simultaneous
writes	to	the	register	file.	The	other	instructions	could	therefore	use	a

single	write	port,	sharing	this	for	writing	
	and	
.	The	following
figure	shows	a	modified	version	of	the	write-back	logic,	in	which	we
merge	the	write-back	register	IDs	(
	and	
)	into	a	single	signal
	and	the	write-back	values	(
	and	
)	into	a	single	signal
:
The	logic	for	performing	the	merges	is	written	in	HCL	as	follows:

The	control	for	these	multiplexors	is	determined	by	
—when	it
indicates	there	is	some	register,	then	it	selects	the	value	for	port	E,	and
otherwise	it	selects	the	value	for	port	M.
In	the	simulation	model,	we	can	then	disable	register	port	M,	as	shown	by
the	following	HCL	code:
The	challenge	then	becomes	to	devise	a	way	to	handle	
.	One
method	is	to	use	the	control	logic	to	dynamically	process	the	instruction
	rA	so	that	it	has	the	same	effect	as	the	two-instruction	sequence
(See	
Practice	Problem	
4.3
	for	a	description	of	the	
	instruction.)
Note	the	ordering	of	the	two	instructions	to	make	sure	
	
	works
properly.	You	can	do	this	by	having	the	logic	in	the	decode	stage	treat
	the	same	as	it	would	the	
	listed	above,	except	that	it	predicts
the	next	PC	to	be	equal	to	the	current	PC.	On	the	next	cycle,	the	

instruction	is	refetched,	but	the	instruction	code	is	converted	to	a	special
value	
.	This	is	treated	as	a	special	instruction	that	has	the	same
behavior	as	the	
	instruction	listed	above.
The	file	
	contains	the	modified	write	port	logic	described
above.	It	contains	a	declaration	of	the	constant	
	having
hexadecimal	value	E.	It	also	contains	the	definition	of	a	signal	f_icode
that	generates	the	icode	field	for	pipeline	register	D.	This	definition	can
be	modified	to	insert	the	instruction	code	
	the	second	time	the	
instruction	is	fetched.	The	HCL	file	also	contains	a	declaration	of	the
signal	f_pc,	the	value	of	the	program	counter	generated	in	the	fetch	stage
by	the	block	labeled	"Select	PC"	(
Figure	
4.57
).
Modify	the	control	logic	in	this	file	to	process	
	instructions	in	the
manner	we	have	described.	See	the	lab	material	for	directions	on	how	to
generate	a	simulator	for	your	solution	and	how	to	test	it.
4.59
Compare	the	performance	of	the	three	versions	of	bubblesort	(
Problems
4.47
,	
4.48
,	and	
4.49
).	Explain	why	one	version	performs	better
than	the	other.

Solutions	to	Practice	Problems
Solution	to	Problem	
4.1	
(page
360
)
Encoding	instructions	by	hand	is	rather	tedious,	but	it	will	solidify	your
understanding	of	the	idea	that	assembly	code	gets	turned	into	byte
sequences	by	the	assembler.	In	the	following	output	from	our	Y86-64
assembler,	each	line	shows	an	address	and	a	byte	sequence	that	starts
at	that	address:
Several	features	of	this	encoding	are	worth	noting:

Decimal	15	(line	2)	has	hex	representation	
.
Writing	the	bytes	in	reverse	order	gives	
.
Decimal	-3	(line	5)	has	hex	representation	
.	Writing
the	bytes	in	reverse	order	gives	
.
The	code	starts	at	address	
.	The	first	instruction	requires	10
bytes,	while	the	second	requires	2.	Thus,	the	loop	target	will	be
.	Writing	these	bytes	in	reverse	order	gives	
.
Solution	to	Problem	
4.2	
(page
360
)
Decoding	a	byte	sequence	by	hand	helps	you	understand	the	task	faced
by	a	processor.	It	must	read	byte	sequences	and	determine	what
instructions	are	to	be	executed.	In	the	following,	we	show	the	assembly
code	used	to	generate	each	of	the	byte	sequences.	To	the	left	of	the
assembly	code,	you	can	see	the	address	and	byte	sequence	for	each
instruction.
A
.	
Some	operations	with	immediate	data	and	address	displacements:

B
.	
Code	including	a	function	call:
C
.	
Code	containing	illegal	instruction	specifier	byte	
:
D
.	
Code	containing	a	jump	operation:
E
.	
Code	containing	an	invalid	second	byte	in	a	
	instruction:

Solution	to	Problem	
4.3	
(page
369
)
Using	the	
	instruction,	we	can	rewrite	the	
	function	as

Solution	to	Problem	
4.4	
(page
370
)
G
CC
,	running	on	an	x86-64	machine,	produces	the	following	code	for
:

This	can	easily	be	adapted	to	produce	Y86-64	code:
Solution	to	Problem	
4.5	
(page
370
)

This	problem	gives	you	a	chance	to	try	your	hand	at	writing	assembly
code.

Solution	to	Problem	
4.6	
(page
370
)
This	problem	gives	you	a	chance	to	try	your	hand	at	writing	assembly
code	with	conditional	moves.	We	show	only	the	code	for	the	loop.	The
rest	is	the	same	as	for	
Problem	
4.5
:
Solution	to	Problem	
4.7	
(page
370
)

Although	it	is	hard	to	imagine	any	practical	use	for	this	particular
instruction,	it	is	important	when	designing	a	system	to	avoid	any
ambiguities	in	the	specification.	We	want	to	determine	a	reasonable
convention	for	the	instruction's	behavior	and	to	make	sure	each	of	our
implementations	adheres	to	this	convention.
The	
	instruction	in	this	test	compares	the	starting	value	of	
	to
the	value	pushed	onto	the	stack.	The	fact	that	the	result	of	this
subtraction	is	zero	implies	that	the	old	value	of	
	gets	pushed.
Solution	to	Problem	
4.8	
(page
371
)
It	is	even	more	difficult	to	imagine	why	anyone	would	want	to	pop	to	the
stack	pointer.	Still,	we	should	decide	on	a	convention	and	stick	with	it.
This	code	sequence	pushes	
	onto	the	stack,	pops	to	
,	and
returns	the	popped	value.	Since	the	result	equals	
,	we	can	deduce
that	
	sets	the	stack	pointer	to	the	value	read	from	memory.	It	is
therefore	equivalent	to	the	instruction	
	(
),
.
Solution	to	Problem	
4.9	
(page
374
)
The	
EXCLUSIVE
-
OR
	
function	requires	that	the	2	bits	have	opposite	values:

In	general,	the	signals	
	and	
	will	be	complements	of	each	other.
That	is,	one	will	equal	1	whenever	the	other	is	0.
Solution	to	Problem	
4.10	
(page
377
)
The	outputs	of	the	
EXCLUSIVE
-
OR
	
circuits	will	be	the	complements	of	the	bit
equality	values.	Using	DeMorgan's	laws	(Web	Aside	
DATA
:
BOOL
	
on	page
52),	we	can	implement	
AND
	
using	
OR
	
and	
NOT
,	yielding	the	circuit	shown	in
Figure	
4.71
.
Solution	to	Problem	
4.11	
(page
379
)
We	can	see	that	the	second	part	of	the	case	expression	can	be	written	as
Since	the	first	line	will	detect	the	case	where	A	is	the	minimum	element,
the	second	line	need	only	determine	whether	B	or	C	is	minimum.

Solution	to	Problem	
4.12	
(page
380
)
This	design	is	a	variant	of	the	one	to	find	the	minimum	of	the	three	inputs:
Figure	
4.71	
Solution	for	
Problem	
4.10
.

Solution	to	Problem	
4.13	
(page
387
)
These	exercises	help	make	the	stage	computations	more	concrete.	We
can	see	from	the	object	code	that	this	instruction	is	located	at	address
.	It	consists	of	10	bytes,	with	the	first	two	being	
	and	
.	The
last	8	bytes	are	a	byte-reversed	version	of	
	(decimal
128).
Stage
Generic	
	V,	rB
Specific	
	$128,	
Fetch
icode:ifun	←	M
[PC]
icode:ifun	←	M
rA:rB	←	MfiTC	+	1]
rA:rB	←	M
[
valC	←	M
[PC	+	2]
valC	←	M
valP	←	PC	+	10
valP	←	
Decode
Execute
valE	←	0	+	valC
valE	←	
Memory
Write	back
R[rB]	←	valE
R[
]	←	valE=128
PC	update
PC	←	valP
PC	←	valP	=	
This	instruction	sets	register	
	to	128	and	increments	the	PC	by	10.
1
1
1
8
8

Solution	to	Problem	
4.14	
(page
390
)
We	can	see	that	the	instruction	is	located	at	address	
	and	consists
of	2	bytes	with	values	
	and	
.	Register	
	was	set	to	120	by
the	
	instruction	(line	6),	which	also	stored	9	at	this	memory	location.
Stage
Generic	
	rA
Specific	
Fetch
icode:ifun	←	M
[PC]
rA:rB	←	M
[PC	+	1]
icode:ifun	←	M
rA:rB	←	M
valP	←	PC	+	2
valP	←	
Decode
valA	←	R[
]
valB	←	R[
]
valA	←	R[
]	=	
	
valB	←	R[
]	=	
Execute
valE	←	valB	+	8
valE	←	
Memory
valM	←	M
[valA]
valM	←	M
Write	back
R[
]	←	valE	
R[rA]	←	valM
R[
]	←	128	
R[
]	←	
PC	update
PC	←	valP
PC	←	
The	instruction	sets	
	to	9,	sets	
	to	128,	and	increments	the	PC
by	2.
1
1
1
1
8
8

Solution	to	Problem	
4.15	
(page
391
)
Tracing	the	steps	listed	in	
Figure	
4.20
	with	rA	equal	to	
,	we	can
see	that	in	the	memory	stage	the	instruction	will	store	valA,	the	original
value	of	the	stack	pointer,	to	memory,	just	as	we	found	for	x86-64.
Solution	to	Problem	
4.16	
(page
392
)
Tracing	the	steps	listed	in	
Figure	
4.20
	with	rA	equal	to	
,	we	can
see	that	both	of	the	write-back	operations	will	update	
.	Since	the	one
writing	valM	would	occur	last,	the	net	effect	of	the	instruction	will	be	to
write	the	value	read	from	memory	to	
,	just	as	we	saw	for	x86-64.
Solution	to	Problem	
4.17	
(page
393
)
Implementing	conditional	moves	requires	only	minor	changes	from
register-to-register	moves.	We	simply	condition	the	write-back	step	on
the	outcome	of	the	conditional	test:

Stage
	rA,	rB
Fetch
icode:ifun	←	M
[PC]
rA:rB	←	M
[PC	+	1]
valP	←	PC	+	2
Decode
valA	←	R[rA]
Execute
valE	←	0	+	valA
Cnd	←	Cond(CC,	ifun)
Memory
Write	back
	(Cnd)	R[rB]	←	valE
PC	update
PC	←	valP
Solution	to	Problem	
4.18	
(page
394
)
We	can	see	that	this	instruction	is	located	at	address	
	and	is	9
bytes	long.	The	first	byte	has	value	
,	while	the	last	8	bytes	are	a
byte-reversed	version	of	
,	the	call	target.	The	stack
pointer	was	set	to	128	by	the	
	instruction	(line	7).
Stage
Generic	
	Dest
Specific	
1
1

Fetch
icode:ifun	←	M
[PC]
icode:ifun	←	M
valC	←	M
[PC	+	1]
valP	←	PC	+	9
valC	←	M
valP	←	
Decode
valB	←	R[
]
valB	←	R[
]	=	
Execute
valE	←	valB	+	-8
valE	←	
Memory
M
[valE]	←	valP
M
[
]	←	
Write	back
R[
]	←	valE
R[
]	←	
PC	update
PC	←	valC
PC	←	
The	effect	of	this	instruction	is	to	set	
	to	120,	to	store	
	(the
return	address)	at	this	memory	address,	and	to	set	the	PC	to	
	(the
call	target).
Solution	to	Problem	
4.19	
(page
406
)
All	of	the	HCL	code	in	this	and	other	practice	problems	is	straightforward,
but	trying	to	generate	it	yourself	will	help	you	think	about	the	different
instructions	and	how	they	are	processed.	For	this	problem,	we	can	simply
look	at	the	set	of	Y86-64	instructions	(
Figure	
4.2
)	and	determine	which
have	a	constant	field.
1
1
8
8
8
8

Solution	to	Problem	
4.20	
(page
407
)
This	code	is	similar	to	the	code	for	srcA.
Solution	to	Problem	
4.21	
(page
408
)
This	code	is	similar	to	the	code	for	dstE.

Solution	to	Problem	
4.22	
(page
408
)
As	we	found	in	
Practice	Problem	
4.16
,	we	want	the	write	via	the	M
port	to	take	priority	over	the	write	via	the	E	port	in	order	to	store	the	value
read	from	memory	into	
.
Solution	to	Problem	
4.23	
(page
409
)
This	code	is	similar	to	the	code	for	aluA.

Solution	to	Problem	
4.24	
(page
409
)
Implementing	conditional	moves	is	surprisingly	simple:	we	disable	writing
to	the	register	file	by	setting	the	destination	register	to	
	when	the
condition	does	not	hold.
Solution	to	Problem	
4.25	
(page
410
)

This	code	is	similar	to	the	code	for	mem_addr.
Solution	to	Problem	
4.26	
(page
410
)
This	code	is	similar	to	the	code	for	mem_read.
Solution	to	Problem	
4.27	
(page
411
)

Computing	the	Stat	field	requires	collecting	status	information	from
several	stages:
Solution	to	Problem	
4.28	
(page
417
)
This	problem	is	an	interesting	exercise	in	trying	to	find	the	optimal
balance	among	a	set	of	partitions.	It	provides	a	number	of	opportunities
to	compute	throughputs	and	latencies	in	pipelines.
A
.	
For	a	two-stage	pipeline,	the	best	partition	would	be	to	have
blocks	A,	B,	and	C	in	the	first	stage	and	D,	E,	and	F	in	the	second.
The	first	stage	has	a	delay	of	170	ps,	giving	a	total	cycle	time	of
170	+	20	=	190	ps.	We	therefore	have	a	throughput	of	5.26	GIPS
and	a	latency	of	380	ps.

B
.	
For	a	three-stage	pipeline,	we	should	have	blocks	A	and	B	in	the
first	stage,	blocks	C	and	D	in	the	second,	and	blocks	E	and	F	in
the	third.	The	first	two	stages	have	a	delay	of	110	ps,	giving	a	total
cycle	time	of	130	ps	and	a	throughput	of	7.69	GIPS.	The	latency	is
390	ps.
C
.	
For	a	four-stage	pipeline,	we	should	have	block	A	in	the	first	stage,
blocks	B	and	C	in	the	second,	block	D	in	the	third,	and	blocks	E
and	F	in	the	fourth.	The	second	stage	requires	90	ps,	giving	a	total
cycle	time	of	110	ps	and	a	throughput	of	9.09	GIPS.	The	latency	is
440	ps.
D
.	
The	optimal	design	would	be	a	five-stage	pipeline,	with	each	block
in	its	own	stage,	except	that	the	fifth	stage	has	blocks	E	and	F	The
cycle	time	is	80	+	20	=	100	ps,	for	a	throughput	of	around	10.00
GIPS	and	a	latency	of	
500	ps.	Adding	more	stages	would	not	help,
since	we	cannot	run	the	pipeline	any	faster	than	one	cycle	every
100	ps.
Solution	to	Problem	
4.29	
(page
418
)
Each	stage	would	have	combinational	logic	requiring	300/
k
	ps	and	a
pipeline	register	requiring	20	ps.
A
.	
The	total	latency	would	be	300	+	20
k
	ps,	while	the	throughput	(in
GIPS)	would	be
1
,
000
300
k
+
20
=
1
,
000
k
300
+
20
k

B
.	
As	we	let	
k
	go	to	infinity,	the	throughput	becomes	1,000/20	=	50
GIPS.	Of	course,	the	latency	would	approach	infinity	as	well.
This	exercise	quantifies	the	diminishing	returns	of	deep	pipelining.	As	we
try	to	subdivide	the	logic	into	many	stages,	the	latency	of	the	pipeline
registers	becomes	a	limiting	factor.
Solution	to	Problem	
4.30	
(page
449
)
This	code	is	very	similar	to	the	corresponding	code	for	SEQ,	except	that
we	cannot	yet	determine	whether	the	data	memory	will	generate	an	error
signal	for	this	instruction.
Solution	to	Problem	
4.31	
(page

449
)
This	code	simply	involves	prefixing	the	signal	names	in	the	code	for	SEQ
with	
	and	
.
Solution	to	Problem	
4.32	
(page
452
)
The	
	instruction	(line	5)	would	stall	for	one	cycle	due	to	a	load/use
hazard	caused	by	the	
	instruction	(line	4).	As	it	enters	the	decode
stage,	the	
	instruction	would	be	in	the	memory	stage,	giving	both
M_dstE	and	M_dstM	equal	to	
.	If	the	two	cases	were	reversed,	then
the	write	back	from	M_valE	would	take	priority,	causing	the	incremented
stack	pointer	to	be	passed	as	the	argument	
to	the	
	instruction.	This
would	not	be	consistent	with	the	convention	for	handling	
determined	in	
Practice	Problem	
4.8
.

Solution	to	Problem	
4.33	
(page
452
)
This	problem	lets	you	experience	one	of	the	important	tasks	in	processor
design—devising	test	programs	for	a	new	processor.	In	general,	we
should	have	test	programs	that	will	exercise	all	of	the	different	hazard
possibilities	and	will	generate	incorrect	results	if	some	dependency	is	not
handled	properly.
For	this	example,	we	can	use	a	slightly	modified	version	of	the	program
shown	in	
Practice	Problem	
4.32
:
The	two	
	instructions	will	cause	the	
	instruction	to	be	in	the	write-
back	stage	when	the	
	instruction	is	in	the	decode	stage.	If	the	two
forwarding	sources	in	the	write-back	stage	are	given	the	wrong	priority,
then	register	
	will	be	set	to	the	incremented	program	counter	rather
than	the	value	read	from	memory.

Solution	to	Problem	
4.34	
(page
453
)
This	logic	only	needs	to	check	the	five	forwarding	sources:
Solution	to	Problem	
4.35	
(page
454
)
This	change	would	not	handle	the	case	where	a	conditional	move	fails	to
satisfy	the	condition,	and	therefore	sets	the	dstE	value	to	
.	The

resulting	value	could	get	forwarded	to	the	next	instruction,	even	though
the	conditional	transfer	does	not	occur.
This	code	initializes	register	
	to	
.	The	conditional	data	transfer
does	not	take	place,	and	so	the	final	
	instruction	should	double	the
value	in	
	to	
.	With	the	altered	design,	however,	the	conditional
move	source	value	
	gets	forwarded	into	ALU	input	valA,	while	input
valB	correctly	gets	operand	value	
.	These	inputs	get	added	to
produce	result	
.
Solution	to	Problem	
4.36	
(page
455
)
This	code	completes	the	computation	of	the	status	code	for	this
instruction.

Solution	to	Problem	
4.37	
(page
461
)
The	following	test	program	is	designed	to	set	up	control	combination	A
(
Figure	
4.67
)	and	detect	whether	something	goes	wrong:

This	program	is	designed	so	that	if	something	goes	wrong	(for	example,	if
the	
	instruction	is	actually	executed),	then	the	program	will	execute
one	of	the	extra	
	instructions	and	then	halt.	Thus,	an	error	in	the
pipeline	would	cause	some	register	to	be	updated	incorrectly.	This	code
illustrates	the	care	required	to	implement	a	test	program.	It	must	set	up	a
potential	error	condition	and	then	detect	whether	or	not	an	error	occurs.
Solution	to	Problem	
4.38	
(page
462
)
The	following	test	program	is	designed	to	set	up	control	combination	B
(
Figure	
4.67
).	The	simulator	will	detect	a	case	where	the	bubble	and
stall	control	signals	for	a	pipeline	register	are	both	set	to	zero,	and	so	our
test	program	need	only	set	up	the	combination	for	it	to	be	detected.	The
biggest	challenge	is	to	make	the	program	do	something	sensible	when
handled	correctly.

This	program	uses	two	initialized	words	in	memory.	The	first	word	(M
)
holds	the	address	of	the	second	(
--the	desired	stack	pointer).	The
second	word	holds	the	address	of	the	desired	return	point	for	the	
instruction.	The	program	loads	the	stack	pointer	into	
	and	executes
the	
	instruction.
Solution	to	Problem	
4.39	
(page
463
)
From	
Figure	
4.66
,	we	can	see	that	pipeline	register	D	must	be	stalled
for	a	load/use	hazard:

Solution	to	Problem	
4.40	
(page
464
)
From	
Figure	
4.66
,	we	can	see	that	pipeline	register	E	must	be	set	to
bubble	for	a	load/use	hazard	or	for	a	mispredicted	branch:
Solution	to	Problem	
4.41	
(page
464
)

This	control	requires	examining	the	code	of	the	executing	instruction	and
checking	for	exceptions	further	down	the	pipeline.
Solution	to	Problem	
4.42	
(page
464
)
Injecting	a	bubble	into	the	memory	stage	on	the	next	cycle	involves
checking	for	an	exception	in	either	the	memory	or	the	write-back	stage
during	the	current	cycle.
For	stalling	the	write-back	stage,	we	check	only	the	status	of	the
instruction	in	this	stage.	If	we	also	stalled	when	an	excepting	instruction

was	in	the	memory	stage,	then	this	instruction	would	not	be	able	to	enter
the	write-back	stage.
Solution	to	Problem	
4.43	
(page
468
)
We	would	then	have	a	misprediction	frequency	of	0.35,	giving	
mp
	=	0.20
×	0.35	×	2	=	0.14,	giving	an	overall	CPI	of	1.25.	This	seems	like	a	fairly
marginal	gain,	but	it	would	be	worthwhile	if	the	cost	of	implementing	the
new	branch	prediction	strategy	were	not	too	high.
Solution	to	Problem	
4.44	
(page
468
)
This	simplified	analysis,	where	we	focus	on	the	inner	loop,	is	a	useful
way	to	estimate	program	performance.	As	long	as	the	array	is	sufficiently
large,	the	time	spent	in	other	parts	of	the	code	will	be	negligible.
A
.	
The	inner	loop	of	the	code	using	the	conditional	jump	has	11
instructions,	all	of	which	are	executed	when	the	array	element	is
zero	or	negative,	and	10	of	which	are	executed	when	the	array

element	is	positive.	The	average	is	10.5.	The	inner	loop	of	the
code	using	the	conditional	move	has	10	instructions,	all	of	which
are	executed	every	time.
B
.	
The	loop-closing	jump	will	be	predicted	correctly,	except	when	the
loop	terminates.	For	a	very	long	array,	this	one	misprediction	will
have	a	negligible	effect	on	the	performance.	The	only	other	source
of	bubbles	for	the	jump-based	code	is	the	conditional	jump,
depending	on	whether	or	not	the	array	element	is	positive.	This
will	cause	two	bubbles,	but	it	only	occurs	50%	of	the	time,	so	the
average	is	1.0.	There	are	no	bubbles	in	the	conditional	move
code.
C
.	
Our	conditional	jump	code	requires	an	average	of	10.5	+	1.0	=
11.5	cycles	per	array	element	(11	cycles	in	the	best	case	and	12
cycles	in	the	worst),	while	our	conditional	move	code	requires	10.0
cycles	in	all	cases.
Our	pipeline	has	a	branch	misprediction	penalty	of	only	two	cycles—far
better	than	those	for	the	deep	pipelines	of	higher-performance
processors.	As	a	result,	using	conditional	moves	does	not	affect	program
performance	very	much.

Chapter	
5	
Optimizing	Program
Performance
5.1	
Capabilities	and	Limitations	of	Optimizing	Compilers	
498
5.2	
Expressing	Program	Performance	
502
5.3	
Program	Example	
504
5.4	
Eliminating	Loop	Inefficiencies	
508
5.5	
Reducing	Procedure	Calls	
512
5.6	
Eliminating	Unneeded	Memory	References	
514
5.7	
Understanding	Modern	Processors	
517
5.8	
Loop	Unrolling	
531
5.9	
Enhancing	Parallelism	
536
5.10	
Summary	of	Results	for	Optimizing	Combining	Code	
547
5.11	
Some	Limiting	Factors	
548
5.12	
Understanding	Memory	Performance	
553
5.13	
Life	in	the	Real	World:	Performance	Improvement	Techniques
561

5.14	
Identifying	and	Eliminating	Performance	Bottlenecks	
562
5.15	
Summary
	
568
Bibliographic	Notes	
569
Homework	Problems	
570
Solutions	to	Practice	Problems	
573
The	primary	objective	in	writing	a	program	must	be
to	make	it	work	correctly	under	all	possible
conditions.	A	program	that	runs	fast	but	gives
incorrect	results	serves	no	useful	purpose.
Programmers	must	write	clear	and	concise	code,
not	only	so	that	they	can	make	sense	of	it,	but	also
so	that	others	can	read	and	understand	the	code
during	code	reviews	and	when	modifications	are
required	later.
On	the	other	hand,	there	are	many	occasions	when
making	a	program	run	fast	is	also	an	important
consideration.	If	a	program	must	process	video
frames	or	network	packets	in	real	time,	then	a	slow-
running	program	will	not	provide	the	needed
functionality.	When	a	computational	task	is	so
demanding	that	it	requires	days	or	weeks	to
execute,	then	making	it	run	just	20%	faster	can
have	significant	impact.	In	this	chapter,	we	will
explore	how	to	make	programs	run	faster	via
several	different	types	of	program	optimization.

Writing	an	efficient	program	requires	several	types
of	activities.	First,	we	must	select	an	appropriate	set
of	algorithms	and	data	structures.	Second,	we	must
write	source	code	that	the	compiler	can	effectively
optimize	to	turn	into	efficient	executable	code.	For
this	second	part,	it	is	important	to	understand	the
capabilities	and	limitations	of	optimizing	compilers.
Seemingly	minor	changes	in	how	a	program	is
written	can	make	large	differences	in	how	well	a
compiler	can	optimize	it.	Some	programming
languages	are	more	easily	optimized	than	others.
Some	features	of	C,	such	as	the	ability	to	perform
pointer	arithmetic	and	casting,	make	it	challenging
for	a	compiler	to	optimize.	Programmers	can	often
write	their	programs	in	ways	that	make	it	easier	for
compilers	to	generate	efficient	code.	A	third
technique	for	dealing	with	especially	demanding
computations	is	to	divide	a	task	into	portions	that
can	be	computed	in	parallel,	on	some	combination
of	multiple	cores	and	multiple	processors.	We	will
defer	this	aspect	of	performance	enhancement	to
Chapter	
12
.	Even	when	exploiting	parallelism,	it	is
important	that	each	parallel	thread	execute	with
maximum	performance,	and	so	the	material	of	this
chapter	remains	relevant	in	any	case.
In	approaching	program	development	and
optimization,	we	must	consider	how	the	code	will	be
used	and	what	critical	factors	affect	it.	In	general,

programmers	must	make	a	trade-off	between	how
easy	a	program	is	to	implement	and	maintain,	and
how	fast	it	runs.	At	an	algorithmic	level,	a	simple
insertion	sort	can	be	programmed	in	a	matter	of
minutes,	whereas	a	highly	efficient	sort	routine	may
take	a	day	or	more	to	implement	and	optimize.	At
the	coding	level,	many	low-level	optimizations	tend
to	reduce	code	readability	and	modularity,	making
the	programs	more	susceptible	to	bugs	and	more
difficult	to	modify	or	extend.	For	code	that	will	be
executed	repeatedly	in	a	performance-critical
environment,	extensive	optimization	may	be
appropriate.	One	challenge	is	to	maintain	some
degree	of	elegance	and	readability	in	the	code
despite	extensive	transformations.
We	describe	a	number	of	techniques	for	improving
code	performance.	Ideally,	a	compiler	would	be	able
to	take	whatever	code	we	write	and	generate	the
most	efficient	possible	machine-level	program
having	the	specified	behavior.	Modern	compilers
employ	sophisticated	forms	of	analysis	and
optimization,	and	they	keep	getting	better.	Even	the
best	compilers,	however,	can	be	thwarted	by
optimization	blockers
—aspects	of	the	program's
behavior	that	depend	strongly	on	the	execution
environment.	Programmers	must	assist	the	compiler
by	writing	code	that	can	be	optimized	readily.

The	first	step	in	optimizing	a	program	is	to	eliminate
unnecessary	work,	making	the	code	perform	its
intended	task	as	efficiently	as	possible.	This
includes	eliminating	unnecessary	function	calls,
conditional	tests,	and	memory	references.	These
optimizations	do	not	depend	on	any	specific
properties	of	the	target	machine.
To	maximize	the	performance	of	a	program,	both
the	programmer	and	the	compiler	require	a	model	of
the	target	machine,	specifying	how	instructions	are
processed	and	the	timing	characteristics	of	the
different	operations.	For	example,	the	compiler	must
know	timing	information	to	be	able	to	decide
whether	it	should	use	a	multiply	instruction	or	some
combination	of	shifts	and	adds.	Modern	computers
use	sophisticated	techniques	to	process	a	machine-
level	program,	executing	many	instructions	in
parallel	and	possibly	in	a	different	order	than	they
appear	in	the	program.	Programmers	must
understand	how	these	processors	work	to	be	able	to
tune	their	programs	for	maximum	speed.	We
present	a	high-level	model	of	such	a	machine	based
on	recent	designs	of	Intel	and	AMD	processors.	We
also	devise	a	graphical	
data-flow
	notation	to
visualize	the	execution	of	instructions	by	the
processor,	with	which	we	can	predict	program
performance.

With	this	understanding	of	processor	operation,	we
can	take	a	second	step	in	program	optimization,
exploiting	the	capability	of	processors	to	provide
instruction-level	parallelism
,	executing	multiple
instructions	simultaneously.	We	cover	several
program	transformations	that	reduce	the	data
dependencies	between	different	parts	of	a
computation,	increasing	the	degree	of	parallelism
with	which	they	can	be	executed.
We	conclude	the	chapter	by	discussing	issues
related	to	optimizing	large	programs.	We	describe
the	use	of	code	
profilers
—tools	that	measure	the
performance	of	different	parts	of	a	program.	This
analysis	can	help	find	inefficiencies	in	the	code	and
identify	the	parts	of	the	program	on	which	we	should
focus	our	optimization	efforts.
In	this	presentation,	we	make	code	optimization	look
like	a	simple	linear	process	of	applying	a	series	of
transformations	to	the	code	in	a	particular	order.	In
fact,	the	task	is	not	nearly	so	straightforward.	A	fair
amount	of	trial-and-error	experimentation	is
required.	This	is	especially	true	as	we	approach	the
later	optimization	stages,	where	seemingly	small
changes	can	cause	major	changes	in	performance
and	some	very	promising	techniques	prove
ineffective.	As	we	will	see	in	the	examples	that
follow,	it	can	be	difficult	to	explain	exactly	why	a

particular	code	sequence	has	a	particular	execution
time.	Performance	can	depend	on	many	detailed
features	of	the	processor	design	for	which	we	have
relatively	little	documentation	or	understanding.	This
is	another	reason	to	try	a	number	of	different
variations	and	combinations	of	techniques.
Studying	the	assembly-code	representation	of	a
program	is	one	of	the	most	effective	means	for
gaining	an	understanding	of	the	compiler	and	how
the	generated	code	will	run.	A	good	strategy	is	to
start	by	looking	carefully	at	the	code	for	the	inner
loops,	identifying	performance-reducing	attributes
such	as	excessive	memory	references	and	poor	use
of	registers.	Starting	with	the	assembly	code,	we
can	also	predict	what	operations	will	be	performed
in	parallel	and	how	well	they	will	use	the	processor
resources.	As	we	will	see,	we	can	often	determine
the	time	(or	at	least	a	lower	bound	on	the	time)
required	to	execute	a	loop	by	identifying	
critical
paths
,	chains	of	data	dependencies	that	form	during
repeated	executions	of	a	loop.	We	can	then	go	back
and	modify	the	source	code	to	try	to	steer	the
compiler	toward	more	efficient	implementations.
Most	major	compilers,	including	
GCC
,	are	continually
being	updated	and	improved,	especially	in	terms	of
their	optimization	abilities.	One	useful	strategy	is	to
do	only	as	much	rewriting	of	a	program	as	is

required	to	get	it	to	the	point	where	the	compiler	can
then	generate	efficient	code.	By	this	means,	we
avoid	compromising	the	readability,	modularity,	and
portability	of	the	code	as	much	as	if	we	had	to	work
with	a	compiler	of	only	minimal	capabilities.	Again,	it
helps	to	iteratively	modify	the	code	and	analyze	its
performance	both	through	measurements	and	by
examining	the	generated	assembly	code.
To	novice	programmers,	it	might	seem	strange	to
keep	modifying	the	source	code	in	an	attempt	to
coax	the	compiler	into	generating	efficient	code,	but
this	is	indeed	how	many	high-performance
programs	are	written.	Compared	to	the	alternative	of
writing	code	in	assembly	language,	this	indirect
approach	has	the	advantage	that	the	resulting	code
will	still	run	on	other	machines,	although	perhaps
not	with	peak	performance.

5.1	
Capabilities	and	Limitations	of
Optimizing	Compilers
Modern	compilers	employ	sophisticated	algorithms	to	determine	what
values	are	computed	in	a	program	and	how	they	are	used.	They	can	then
exploit	opportunities	to	simplify	expressions,	to	use	a	single	computation
in	several	different	places,	and	to	reduce	the	number	of	times	a	given
computation	must	be	performed.	Most	compilers,	including	
GCC
,	provide
users	with	some	control	over	which	optimizations	they	apply.	As
discussed	in	
Chapter	
3
,	the	simplest	control	is	to	specify	the
optimization	level.
	For	example,	invoking	
GCC
	
with	the	command-line
option	
	specifies	that	it	should	apply	a	basic	set	of	optimizations.
Invoking	
GCC
	
with	option	
	or	higher	(e.g.,	
	or	
)	will	cause	it	to
apply	more	extensive	optimizations.	These	can	further	improve	program
performance,	but	they	may	expand	the	program	size	and	they	may	make
the	program	more	difficult	to	debug	using	standard	debugging	tools.	For
our	presentation,	we	will	mostly	consider	code	compiled	with	optimization
level	
,	even	though	level	
	has	become	the	accepted	standard	for
most	software	projects	that	use	
GCC
.	We	purposely	limit	the	level	of
optimization	to	demonstrate	how	different	ways	of	writing	a	function	in	C
can	affect	the	efficiency	of	the	code	generated	by	a	compiler.	We	will	find
that	we	can	write	C	code	that,	when	compiled	just	with	option	
,	vastly
outperforms	a	more	naive	version	compiled	with	the	highest	possible
optimization	levels.

Compilers	must	be	careful	to	apply	only	
safe
	optimizations	to	a	program,
meaning	that	the	resulting	program	will	have	the	exact	same	behavior	as
would	an	unoptimized	version	for	all	possible	cases	the	program	may
encounter,	up	to	the	limits	of	the	guarantees	provided	by	the	C	language
standards.	Constraining	
the	compiler	to	perform	only	safe	optimizations
eliminates	possible	sources	of	undesired	run-time	behavior,	but	it	also
means	that	the	programmer	must	make	more	of	an	effort	to	write
programs	in	a	way	that	the	compiler	can	then	transform	into	efficient
machine-level	code.	To	appreciate	the	challenges	of	deciding	which
program	transformations	are	safe	or	not,	consider	the	following	two
procedures:
At	first	glance,	both	procedures	seem	to	have	identical	behavior.	They
both	add	twice	the	value	stored	at	the	location	designated	by	pointer	
to	that	designated	by	pointer	
.	On	the	other	hand,	function	
	is
more	efficient.	It	requires	only	three	memory	references	(read	
,	read

,	write	
),	whereas	
	requires	six	(two	reads	of	
,	two
reads	of	
,	and	two	writes	of	
).	Hence,	if	a	compiler	is	given
procedure	
	to	compile,	one	might	think	it	could	generate	more
efficient	code	based	on	the	computations	performed	by	
.
Consider,	however,	the	case	in	which	
	and	
	are	equal.	Then	function
	will	perform	the	following	computations:
The	result	will	be	that	the	value	at	
	will	be	increased	by	a	factor	of	4.
On	the	other	hand,	function	
	will	perform	the	following
computation:
The	result	will	be	that	the	value	at	
	will	be	increased	by	a	factor	of	3.
The	compiler	knows	nothing	about	how	
	will	be	called,	and	so	it
must	assume	that	arguments	
	and	
	can	be	equal.	It	therefore	cannot
generate	code	in	the	style	of	
	as	an	optimized	version	of
.

The	case	where	two	pointers	may	designate	the	same	memory	location
is	known	as	
memory	aliasing.
	In	performing	only	safe	optimizations,	the
compiler	must	assume	that	different	pointers	may	be	aliased.	As	another
example,	for	a	program	with	pointer	variables	
	and	
,	consider	the
following	code	sequence:
The	value	computed	for	
	depends	on	whether	or	not	pointers	
	and	
are	aliased—if	not,	it	will	equal	3,000,	but	if	so	it	will	equal	1,000.	This
leads	to	one	of	the	major	
optimization	blockers
,	aspects	of	programs	that
can	severely	limit	the	opportunities	for	a	compiler	to	generate	optimized
code.	If	a	compiler	cannot	determine	whether	or	not	two	pointers	may	be
aliased,	it	must	assume	that	either	case	is	possible,	limiting	the	set	of
possible	optimizations.
Practice	Problem	
5.1	
(solution	page
573
)
The	following	problem	illustrates	the	way	memory	aliasing	can
cause	unexpected	program	behavior.	Consider	the	following

procedure	to	swap	two	values:
If	this	procedure	is	called	with	
	equal	to	
,	what	effect	will	it
have?
A	second	optimization	blocker	is	due	to	function	calls.	As	an	example,
consider	the	following	two	procedures:

It	might	seem	at	first	that	both	compute	the	same	result,	but	with	
calling	
	only	once,	whereas	
	calls	it	four	times.	It	is	tempting	to
generate	code	in	the	style	of	
	when	given	
	as	the	source.
Consider,	however,	the	following	code	for	f:
This	function	has	a	
side	effect
—it	modifies	some	part	of	the	global
program	state.	Changing	the	number	of	times	it	gets	called	changes	the
program	behavior.	In
Aside	
Optimizing	function	calls	by	inline
substitution
Code	involving	function	calls	can	be	optimized	by	a	process
known	as	
inline	substitution
	(or	simply	"inlining"),	where	the
function	call	is	replaced	by	the	code	for	the	body	of	the	function.
For	example,	we	can	expand	the	code	for	
	by	substituting
four	instantiations	of	function	
:

This	transformation	both	reduces	the	overhead	of	the	function
calls	and	allows	further	optimization	of	the	expanded	code.	For
example,	the	compiler	can	consolidate	the	updates	of	global
variable	
	in	
	in	to	generate	an	optimized	version	of
the	function:
This	code	faithfully	reproduces	the	behavior	of	
	for	this
particular	definition	of	function	
.

Recent	versions	of	
GCC
	
attempt	this	form	of	optimization,	either
when	directed	to	with	the	command-line	option	
	or	for
optimization	level	
	and	higher.	Unfortunately,	
GCC
	
only	attempts
inlining	for	functions	defined	within	a	single	file.	That	means	it	will
not	be	applied	in	the	common	case	where	a	set	of	library	functions
is	defined	in	one	file	but	invoked	by	functions	in	other	files.
There	are	times	when	it	is	best	to	prevent	a	compiler	from
performing	inline	substitution.	One	is	when	the	code	will	be
evaluated	using	a	symbolic	debugger,	such	as	
GDB
,	as	described
in	
Section	
3.10.2
.	If	a	function	call	has	been	optimized	away
via	inline	substitution,	then	any	attempt	to	trace	or	set	a	breakpoint
for	that	call	will	fail.	The	second	is	when	evaluating	the
performance	of	a	program	by	profiling,	as	is	discussed	in	
Section
5.14.1
.	Calls	to	functions	that	have	been	eliminated	by	inline
substitution	will	not	be	profiled	correctly.
particular,	a	call	to	
	would	return	0	+	1	+	2	+	3	=	6,	whereas	a	call	to
	would	return	4	·	0	=	0,	assuming	both	started	with	global	variable
counter	set	to	zero.
Most	compilers	do	not	try	to	determine	whether	a	function	is	free	of	side
effects	and	hence	is	a	candidate	for	optimizations	such	as	those
attempted	in	
.	Instead,	the	compiler	assumes	the	worst	case	and
leaves	function	calls	intact.
Among	compilers,	
GCC
	
is	considered	adequate,	but	not	exceptional,	in
terms	of	its	optimization	capabilities.	It	performs	basic	optimizations,	but	it
does	not	perform	the	radical	transformations	on	programs	that	more
"aggressive"	compilers	do.	As	a	consequence,	programmers	using	
GCC

must	put	more	effort	into	writing	programs	in	a	way	that	simplifies	the
compiler's	task	of	generating	efficient	code.

5.2	
Expressing	Program
Performance
We	introduce	the	metric	
cycles	per	element
,	abbreviated	CPE,	to	express
program	performance	in	a	way	that	can	guide	us	in	improving	the	code.
CPE	measurements	help	us	understand	the	loop	performance	of	an
iterative	program	at	a	detailed	level.	It	is	appropriate	for	programs	that
perform	a	repetitive	computation,	such	as	processing	the	pixels	in	an
image	or	computing	the	elements	in	a	matrix	product.
The	sequencing	of	activities	by	a	processor	is	controlled	by	a	clock
providing	a	regular	signal	of	some	frequency,	usually	expressed	in
gigahertz
	(GHz),	billions	of	cycles	per	second.	For	example,	when
product	literature	characterizes	a	system	as	a	"4	GHz"	processor,	it
means	that	the	processor	clock	runs	at	4.0	×	10
	cycles	per	second.	The
time	required	for	each	clock	cycle	is	given	by	the	reciprocal	of	the	clock
frequency.	These	typically	are	expressed	in	
nanoseconds
	(1	nanosecond
is	10
	seconds)	or	
picoseconds
	(1	picosecond	is	10
	seconds).	For
example,	the	period	of	a	4	GHz	clock	can	be	expressed	as	either	0.25
nanoseconds	or	250	picoseconds.	From	a	programmer's	perspective,	it	is
more	instructive	to	express	measurements	in	clock	cycles	rather	than
nanoseconds	or	picoseconds.	That	way,	the	measurements	express	how
many	instructions	are	being	executed	rather	than	how	fast	the	clock	runs.
Many	procedures	contain	a	loop	that	iterates	over	a	set	of	elements.	For
example,	functions	
	and	
	in	
Figure	
5.1
	both	compute	the
−9
−9
−12

prefix	sum
	of	a	vector	of	length	
n
.	For	a	vector	
,	the
prefix	sum	
is	defined	as
Function	
	computes	one	element	of	the	result	vector	per	iteration.
Function	
	uses	a	technique	known	as	
loop	unrolling
	to	compute	two
elements	per	iteration.	We	will	explore	the	benefits	of	loop	unrolling	later
in	this	chapter.	(See	Problems	5.11,5.12,	and	5.19	for	more	about
analyzing	and	optimizing	the	prefix-sum	computation.)
The	time	required	by	such	a	procedure	can	be	characterized	as	a
constant	plus	a	factor	proportional	to	the	number	of	elements	processed.
For	example,	
Figure	
5.2
	shows	a	plot	of	the	number	of	clock	cycles
required	by	the	two	functions	for	a	range	of	values	of	
n
.	Using	a	
least
squares	fit
,	we	find	that	the	run	times	(in	clock	cycles)	for	
	and
	can	be	approximated	by	the	equations	368	+	9.0
n
	and	368	+	6.0
n
,
respectively.	These	equations	indicate	an	overhead	of	368	cycles	due	to
the	timing	code	and	to	initiate	the	procedure,	set	up	the	loop,	and
complete	the
a
→
=
〈
a
0
,
a
1
,
…
 
,
a
n
−
1
〉
p
→
=
〈
p
0
,
p
1
,
…
 
,
p
n
−
1
〉
	
p
0
=
a
0
p
i
=
p
i
-
1
+
a
i
,
 
1
<
i
<
n
(5.1)

Figure	
5.1	
Prefix-sum	functions.
These	functions	provide	examples	for	how	we	express	program
performance.

Figure	
5.2	
Performance	of	prefix-sum	functions.
The	slope	of	the	lines	indicates	the	number	of	clock	cycles	per	element
(CPE).
Aside	
What	is	a	least	squares	fit?
For	a	set	of	data	points	(
x
,	
y
),.	.	.	(
x
,	
y
),	we	often	try	to	draw	a
line	that	best	approximates	the	X-Y	trend	represented	by	these
data.	With	a	least	squares	fit,	we	look	for	a	line	of	the	form	
y	=	mx
+	
b
	that	minimizes	the	following	error	measure:
An	algorithm	for	computing	
m
	and	
b
	can	be	derived	by	finding	the
derivatives	of	
E(m,	b)
	with	respect	to	
m
	and	
b
	and	setting	them	to
0.
procedure,	plus	a	linear	factor	of	6.0	or	9.0	cycles	per	element.	For	large
values	of	
n
	(say,	greater	than	200),	the	run	times	will	be	dominated	by	the
linear	factors.	We	refer	to	the	coefficients	in	these	terms	as	the	effective
number	of	cycles	per	element.	We	prefer	measuring	the	number	of	cycles
1
1
n
n
E
(
m
,
b
)
=
∑
i
=
1
,
n
(
m
x
i
+
b
-
y
i
)
2

per	
element
	rather	than	the	number	of	cycles	per	
iteration
,	because
techniques	such	as	loop	unrolling	allow	us	to	use	fewer	iterations	to
complete	the	computation,	but	our	ultimate	concern	is	how	fast	the
procedure	will	run	for	a	given	vector	length.	We	focus	our	efforts	on
minimizing	the	CPE	for	our	computations.	By	this	measure,	
,	with	a
CPE	of	6.0,	is	superior	to	
,	with	a	CPE	of	9.0.
Practice	Problem	
5.2	
(solution	page
573
)
Later	in	this	chapter	we	will	start	with	a	single	function	and
generate	many	different	variants	that	preserve	the	function's
behavior,	but	with	different	performance	characteristics.	For	three
of	these	variants,	we	found	that	the	run	times	(in	clock	cycles)	can
be	approximated	by	the	following	functions:
Version	1:	60	+	35
n
Version	2:	136	+	4
n
Version	3:	157	+	1.25
n
For	what	values	of	
n
	would	each	version	be	the	fastest	of	the
three?	Remember	that	
n
	will	always	be	an	integer.

5.3	
Program	Example
To	demonstrate	how	an	abstract	program	can	be	systematically
transformed	into	more	efficient	code,	we	will	use	a	running	example
based	on	the	vector	data	structure	shown	in	
Figure	
5.3
.	A	vector	is
represented	with	two	blocks	of	memory:	the	header	and	the	data	array.
The	header	is	a	structure	declared	as	follows:
Figure	
5.3	
Vector	abstract	data	type.
A	vector	is	represented	by	header	information	plus	an	array	of	designated
length.

The	declaration	uses	
	to	designate	the	data	type	of	the	underlying
elements.	In	our	evaluation,	we	measured	the	performance	of	our	code
for	integer	(C	
	and	
),	and	floating-point	(C	
	and	
)
data.	We	do	this	by	compiling	and	running	the	program	separately	for
different	type	declarations,	such	as	the	following	for	data	type	
:
We	allocate	the	data	array	block	to	store	the	vector	elements	as	an	array
of	
	objects	of	type	
.
Figure	
5.4
	shows	some	basic	procedures	for	generating	vectors,
accessing	vector	elements,	and	determining	the	length	of	a	vector.	An
important	feature	to	note	is	that	
,	the	vector	access
routine,	performs	bounds	checking	for	every	vector	reference.	This	code
is	similar	to	the	array	representations	used	in	many	other	languages,
including	Java.	Bounds	checking	reduces	the	chances	of	program	error,
but	it	can	also	slow	down	program	execution.
As	an	optimization	example,	consider	the	code	shown	in	
Figure	
5.5
,
which	combines	all	of	the	elements	in	a	vector	into	a	single	value
according	to	some	operation.	By	using	different	definitions	of	compile-
time	constants	
	and	OP,	the	code	can	be	recompiled	to	perform
different	operations	on	the	data.	In	particular,	using	the	declarations

it	sums	the	elements	of	the	vector.	Using	the	declarations
it	computes	the	product	of	the	vector	elements.
In	our	presentation,	we	will	proceed	through	a	series	of	transformations
of	the	code,	writing	different	versions	of	the	combining	function.	To	gauge
progress,



Figure	
5.4	
Implementation	of	vector	abstract	data	type.
In	the	actual	program,	data	type	
	is	declared	to	be	
Figure	
5.5	
Initial	implementation	of	combining	operation.
Using	different	declarations	of	identity	element	
	and	combining
operation	
,	we	can	measure	the	routine	for	different	operations.
we	measured	the	CPE	performance	of	the	functions	on	a	machine	with
an	Intel	Core	i7	Haswell	processor,	which	we	refer	to	as	our	
reference
machine.
	Some	characteristics	of	this	processor	were	given	in	
Section
3.1
.	These	measurements	characterize	performance	in	terms	of	how

the	programs	run	on	just	one	particular	machine,	and	so	there	is	no
guarantee	of	comparable	performance	on	other	combinations	of	machine
and	compiler.	However,	we	have	compared	the	results	with	those	for	a
number	of	different	compiler/processor	combinations,	and	we	have	found
them	generally	consistent	with	those	presented	here.
As	we	proceed	through	a	set	of	transformations,	we	will	find	that	many
lead	to	only	minimal	performance	gains,	while	others	have	more	dramatic
effects.	Determining	which	combinations	of	transformations	to	apply	is
indeed	part	of	the	"black	art"	of	writing	fast	code.	Some	combinations	that
do	not	provide	measurable	benefits	are	indeed	ineffective,	while	others
are	important	as	ways	to	enable	further	optimizations	by	the	compiler.	In
our	experience,	the	best	approach	involves	a	combination	of
experimentation	and	analysis:	repeatedly	attempting	different
approaches,	performing	measurements,	and	examining	the	assembly-
code	representations	to	identify	underlying	performance	bottlenecks.
As	a	starting	point,	the	following	table	shows	CPE	measurements	for
	running	on	our	reference	machine,	with	different	combinations
of	operation	(addition	or	multiplication)	and	data	type	(long	integer	and
double-precision	floating-point).	Our	experiments	with	many	different
programs	showed	that	operations	on	32-bit	and	64-bit	integers	have
identical	performance,	with	the	exception	of	code	involving	division
operations.	Similarly,	we	found	identical	performance	for	programs
operating	on	single-	or	double-precision	floating-point	data.	In	our	tables,
we	will	therefore	show	only	separate	results	for	integer	data	and	for
floating-point	data.
Integer
Floating	point

Function
Page
Method
+
*
+
*
507
Abstract	unoptimized
22.68
20.02
19.98
20.18
507
Abstract	
10.12
10.12
10.17
11.14
We	can	see	that	our	measurements	are	somewhat	imprecise.	The	more
likely	CPE	number	for	integer	sum	is	23.00,	rather	than	22.68,	while	the
number	for	integer	product	is	likely	20.0	instead	of	20.02.	Rather	than
"fudging"	our	numbers	to	make	them	look	good,	we	will	present	the
measurements	we	actually	obtained.	There	are	many	factors	that
complicate	the	task	of	reliably	measuring	the	precise	number	of	clock
cycles	required	by	some	code	sequence.	It	helps	when	examining	these
numbers	to	mentally	round	the	results	up	or	down	by	a	few	hundredths	of
a	clock	cycle.
The	unoptimized	code	provides	a	direct	translation	of	the	C	code	into
machine	code,	often	with	obvious	inefficiencies.	By	simply	giving	the
command-line	option	
,	we	enable	a	basic	set	of	optimizations.	As	can
be	seen,	this	significantly	improves	the	program	performance—more	than
a	factor	of	2—with	no	effort	on	behalf	of	the	programmer.	In	general,	it	is
good	to	get	into	the	habit	of	enabling	some	level	of	optimization.	(Similar
performance	results	were	obtained	with	optimization	level	
.)	For	the
remainder	of	our	measurements,	we	use	optimization	levels	
	and	
when	generating	and	measuring	our	programs.

5.4	
Eliminating	Loop	Inefficiencies
Observe	that	procedure	
,	as	shown	in	
Figure	
5.5
,	calls
function	
	as	the	test	condition	of	the	
	loop.	Recall	from	our
discussion	of	how	to	translate	code	containing	loops	into	machine-level
programs	(
Section	
3.6.7
)	that	the	test	condition	must	be	evaluated	on
every	iteration	of	the	loop.	On	the	other	hand,	the	length	of	the	vector
does	not	change	as	the	loop	proceeds.	We	could	therefore	compute	the
vector	length	only	once	and	use	this	value	in	our	test	condition.
Figure	
5.6
	shows	a	modified	version	called	
.	It	calls
	at	the	beginning	and	assigns	the	result	to	a	local	variable
length.	This	transformation	has	noticeable	effect	on	the	overall
performance	for	some	data	types	and	operations,	and	minimal	or	even
none	for	others.	In	any	case,	this	transformation	is	required	to	eliminate
inefficiencies	that	would	become	bottlenecks	as	we	attempt	further
optimizations.
Integer
Floating
point
Function
Page
Method
+
*
+
*
507	Abstract	
10.12
10.12
10.17
11.14
509
Move	
7.02
9.03
9.02
11.03

This	optimization	is	an	instance	of	a	general	class	of	optimizations	known
as	
code	motion.
	They	involve	identifying	a	computation	that	is	performed
multiple
Figure	
5.6	
Improving	the	efficiency	of	the	loop	test.
By	moving	the	call	to	
	out	of	the	loop	test,	we	eliminate	the
need	to	execute	it	on	every	iteration.
times,	(e.g.,	within	a	loop),	but	such	that	the	result	of	the	computation	will
not	change.	We	can	therefore	move	the	computation	to	an	earlier	section
of	the	code	that	does	not	get	evaluated	as	often.	In	this	case,	we	moved
the	call	to	
	from	within	the	loop	to	just	before	the	loop.

Optimizing	compilers	attempt	to	perform	code	motion.	Unfortunately,	as
discussed	previously,	they	are	typically	very	cautious	about	making
transformations	that	change	where	or	how	many	times	a	procedure	is
called.	They	cannot	reliably	detect	whether	or	not	a	function	will	have
side	effects,	and	so	they	assume	that	it	might.	For	example,	if	
had	some	side	effect,	then	
	and	
	could	have	different
behaviors.	To	improve	the	code,	the	programmer	must	often	help	the
compiler	by	explicitly	performing	code	motion.
As	an	extreme	example	of	the	loop	inefficiency	seen	in	
,
consider	the	procedure	
	shown	in	
Figure	
5.7
.	This	procedure	is
styled	after	routines	submitted	by	several	students	as	part	of	a	network
programming	project.	Its	purpose	is	to	convert	all	of	the	uppercase	letters
in	a	string	to	lowercase.	The	procedure	steps	through	the	string,
converting	each	uppercase	character	to	lowercase.	The	case	conversion
involves	shifting	characters	in	the	range	`A'	to	`Z'	to	the	range	`a'	to	`z'.
The	library	function	
	is	called	as	part	of	the	loop	test	of	
.
Although	
	is	typically	implemented	with	special	x86	string-
processing	instructions,	its	overall	execution	is	similar	to	the	simple
version	that	is	also	shown	in	
Figure	
5.7
.	Since	strings	in	C	are	null-
terminated	character	sequences,	
	can	only	determine	the	length	of
a	string	by	stepping	through	the	sequence	until	it	hits	a	null	character.	For
a	string	of	length	
n
,	
	takes	time	proportional	to	
n.
	Since	
	is
called	in	each	of	the	
n
	iterations	of	
,	the	overall	run	time	of	
is	quadratic	in	the	string	length,	proportional	to	
n
.
2



Figure	
5.7	
Lowercase	conversion	routines.
The	two	procedures	have	radically	different	performance.
This	analysis	is	confirmed	by	actual	measurements	of	the	functions	for
different	length	strings,	as	shown	in	
Figure	
5.8
	(and	using	the	library
version	of	
).	The	graph	of	the	run	time	for	
	rises	steeply	as
the	string	length	increases	(
Figure	
5.8(a)
).	
Figure	
5.8(b)
	shows	the
run	times	for	seven	different	lengths	(not	the	same	as	shown	in	the
graph),	each	of	which	is	a	power	of	2.	Observe	that	for	
	each
doubling	of	the	string	length	causes	a	quadrupling	of	the	run	time.	This	is
a	clear	indicator	of	a	quadratic	run	time.	For	a	string	of	length	1,048,576,
	requires	over	17	minutes	of	CPU	time.
String	length
Function
16,384
32,768
65,536
131,072
262,144
524,288
1,048,576
0.26
1.03
4.10
16.41
65.62
262.48
1,049.89

0.0000
0.0001
0.0001
0.0003
0.0005
0.0010
0.0020
(b)
Figure	
5.8	
Comparative	performance	of	lowercase	conversion
routines.
The	original	code	
	has	a	quadratic	run	time	due	to	an	inefficient
loop	structure.	The	modified	code	
	has	a	linear	run	time.
Function	
	shown	in	
Figure	
5.7
	is	identical	to	that	of	
,
except	that	we	have	moved	the	call	to	
	out	of	the	loop.	The
performance	improves	dramatically.	For	a	string	length	of	1,048,576,	the
function	requires	just	2.0	milliseconds—over	500,000	times	faster	than
.	Each	doubling	of	the	string	length	causes	a	doubling	of	the	run
time—a	clear	indicator	of	linear	run	time.	For	longer	strings,	the	run-time
improvement	will	be	even	greater.
In	an	ideal	world,	a	compiler	would	recognize	that	each	call	to	
	in
the	loop	test	will	return	the	same	result,	and	thus	the	call	could	be	moved
out	of	the	loop.	This	would	require	a	very	sophisticated	analysis,	since
	checks	the	elements	of	the	string	and	these	values	are	changing
as	
	proceeds.	The	compiler	would	need	to	detect	that	even	though
the	characters	within	the	string	are	changing,	none	are	being	set	from
nonzero	to	zero,	or	vice	versa.	Such	an	analysis	is	well	beyond	the	ability
of	even	the	most	sophisticated	compilers,	even	if	they	employ	inlining,
and	so	programmers	must	do	such	transformations	themselves.
This	example	illustrates	a	common	problem	in	writing	programs,	in	which
a	seemingly	trivial	piece	of	code	has	a	hidden	asymptotic	inefficiency.

One	would	not	expect	a	lowercase	conversion	routine	to	be	a	limiting
factor	in	a	program's	performance.	Typically,	programs	are	tested	and
analyzed	on	small	data	sets,	for	which	the	performance	of	
	is
adequate.	When	the	program	is	ultimately	
deployed,	however,	it	is
entirely	possible	that	the	procedure	could	be	applied	to	strings	of	over
one	million	characters.	All	of	a	sudden	this	benign	piece	of	code	has
become	a	major	performance	bottleneck.	By	contrast,	the	performance	of
	will	be	adequate	for	strings	of	arbitrary	length.	Stories	abound	of
major	programming	projects	in	which	problems	of	this	sort	occur.	Part	of
the	job	of	a	competent	programmer	is	to	avoid	ever	introducing	such
asymptotic	inefficiency.
Practice	Problem	
5.3	
(solution	page
573
)
Consider	the	following	functions:
The	following	three	code	fragments	call	these	functions:
A
.	

	
B
.	
	
C
.	
	
Assume	
	equals	10	and	
	equals	100.	Fill	in	the	following	table
indicating	the	number	of	times	each	of	the	four	functions	is	called
in	code	fragments	A–C:
Code
A.
_____
_____
_____
_____
B.
_____
_____
_____
_____
C.
_____
_____
_____
_____

5.5	
Reducing	Procedure	Calls
As	we	have	seen,	procedure	calls	can	incur	overhead	and	also	block
most	forms	of	program	optimization.	We	can	see	in	the	code	for	
(
Figure	
5.6
)	that	
	is	called	on	every	loop	iteration	to
retrieve	the	next	vector	element.	This	function	checks	the	vector	index	
against	the	loop	bounds	with	every	vector	reference,	a	clear	source	of
inefficiency.	Bounds	checking	might	be	a	useful	feature	when	dealing
with	arbitrary	array	accesses,	but	a	simple	analysis	of	the	code	for
	shows	that	all	references	will	be	valid.
---------------------------------------------------------------------------
code/opt/vec.c
---------------------------------------------------------------------------
code/opt/vec.c

Figure	
5.9	
Eliminating	function	calls	within	the	loop.
The	resulting	code	does	not	show	a	performance	gain,	but	it	enables
additional	optimizations.
Suppose	instead	that	we	add	a	function	
	to	our	abstract
data	type.	This	function	returns	the	starting	address	of	the	data	array,	as
shown	in	
Figure	
5.9
.	We	could	then	write	the	procedure	shown	as
	in	this	figure,	having	no	function	calls	in	the	inner	loop.	Rather
than	making	a	function	call	to	retrieve	each	vector	element,	it	accesses
the	array	directly.	A	purist	might	say	that	this	transformation	seriously
impairs	the	program	modularity.	In	principle,	the	user	of	the	vector
abstract	data	type	should	not	even	need	to	know	that	the	vector	contents
are	stored	as	an	array,	rather	than	as	some	other	data	structure	such	as
a	linked	list.	A	more	pragmatic	programmer	would	argue	that	this
transformation	is	a	necessary	step	toward	achieving	high-performance
results.
Integer
Floating	point

Function
Page
Method
+
*
+
*
509
Move	
7.02
9.03
9.02
11.03
513
Direct	data	access
7.17
9.02
9.02
11.03
Surprisingly,	there	is	no	apparent	performance	improvement.	Indeed,	the
performance	for	integer	sum	has	gotten	slightly	worse.	Evidently,	other
operations	in	the	inner	loop	are	forming	a	bottleneck	that	limits	the
performance	more	than	the	call	to	
.	We	will	return	to	this
function	later	(
Section	
5.11.2
)	and	see	why	the	repeated	bounds
checking	by	
	does	not	incur	a	performance	penalty.	For	now,	we
can	view	this	transformation	as	one	of	a	series	of	steps	that	will	ultimately
lead	to	greatly	improved	performance.

5.6	
Eliminating	Unneeded	Memory
References
The	code	for	
	accumulates	the	value	being	computed	by	the
combining	operation	at	the	location	designated	by	the	pointer	
.	This
attribute	can	be	seen	by	examining	the	assembly	code	generated	for	the
inner	loop	of	the	compiled	code.	We	show	here	the	x86-64	code
generated	for	data	type	
	and	with	multiplication	as	the	combining
operation:
We	see	in	this	loop	code	that	the	address	corresponding	to	pointer	
is	held	in	register	
.	It	has	also	transformed	the	code	to	maintain	a

pointer	to	the	
i
th	data	element	in	register	
,	shown	in	the	annotations
as	
.	This	pointer	is	incremented	by	8	on	every	iteration.	The	loop
termination	is	detected	by	comparing	this	pointer	to	one	stored	in	register
.	We	can	see	that	the	accumulated	value	is	read	from	and	written	to
memory	on	each	iteration.	This	reading	and	writing	is	wasteful,	since	the
value	read	from	
	at	the	beginning	of	each	iteration	should	simply	be
the	value	written	at	the	end	of	the	previous	iteration.
We	can	eliminate	this	needless	reading	and	writing	of	memory	by
rewriting	the	code	in	the	style	of	
	in	
Figure	
5.10
.	We	introduce
a	temporary	variable	
	that	is	used	in	the	loop	to	accumulate	the
computed	value.	The	result	is	stored	at	
	only	after	the	loop	has	been
completed.	As	the	assembly	code	that	follows	shows,	the	compiler	can
now	use	register	
	to	hold	the	accumulated	value.	Compared	to	the
loop	in	
,	we	have	reduced	the	memory	operations	per	iteration
from	two	reads	and	one	write	to	just	a	single	read.

We	see	a	significant	improvement	in	program	performance,	as	shown	in
the	following	table:
Figure	
5.10	
Accumulating	result	in	temporary.
Holding	the	accumulated	value	in	local	variable	
	(short	for
"accumulator")	eliminates	the	need	to	retrieve	it	from	memory	and	write
back	the	updated	value	on	every	loop	iteration.
Integer
Floating	point
Function
Page
Method
+
*
+
*
513
Direct	data	access
7.17
9.02
9.02
11.03

515
Accumulate	in	temporary
1.27
3.01
3.01
5.01
All	of	our	times	improve	by	factors	ranging	from	2.2×	to	5.7×,	with	the
integer	addition	case	dropping	to	just	1.27	clock	cycles	per	element.
Again,	one	might	think	that	a	compiler	should	be	able	to	automatically
transform	the	
	code	shown	in	
Figure	
5.9
	to	accumulate	the
value	in	a	register,	as	it	does	with	the	code	for	
	shown	in	
Figure
5.10
.	In	fact,	however,	the	two	functions	can	have	different	behaviors
due	to	memory	aliasing.	Consider,	for	example,	the	case	of	integer	data
with	multiplication	as	the	operation	and	1	as	the	identity	element.	Let	
	=
[2,	3,	5]	be	a	vector	of	three	elements	and	consider	the	following	two
function	calls:
That	is,	we	create	an	alias	between	the	last	element	of	the	vector	and	the
destination	for	storing	the	result.	The	two	functions	would	then	execute
as	follows:
Function
Initial
Before	loop
	=0
	=1
	=2
Final
[2,	3,	5]
[2,	3,	1]
[2,	3,	2]
[2,	3,	6]
[2,	3,	36]
[2,	3,	36]
[2,	3,	5]
[2,	3,	5]
[2,	3,	5]
[2,	3,	5]
[2,	3,	5]
[2,	3,	30]
As	shown	previously,	
	accumulates	its	result	at	the	destination,
which	in	this	case	is	the	final	vector	element.	This	value	is	therefore	set

first	to	1,	then	to	2	·	1	=	2,	and	then	to	3	·	2	=	6.	On	the	last	iteration,	this
value	is	then	multiplied	by	itself	to	yield	a	final	value	of	36.	For	the	case
of	
,	the	vector	remains	unchanged	until	the	end,	when	the	final
element	is	set	to	the	computed	result	1	·	2	·	3	·	5	=	30.
Of	course,	our	example	showing	the	distinction	between	
	and
	is	highly	contrived.	One	could	argue	that	the	behavior	of
	more	closely	matches	the	intention	of	the	function	description.
Unfortunately,	a	compiler	cannot	make	a	judgment	about	the	conditions
under	which	a	function	might	be	used	and	what	the	programmer's
intentions	might	be.	Instead,	when	given	
	to	compile,	the
conservative	approach	is	to	keep	reading	and	writing	memory,	even
though	this	is	less	efficient.
Practice	Problem	
5.4	
(solution	page
574
)
When	we	use	
GCC
	
to	compile	
	with	command-line	option
,	we	get	code	with	substantially	better	CPE	performance	than
with	
:
Integer
Floating	point
Function
Page
Method
+
*
+
*
513
Compiled	
7.17
9.02
9.02
11.03
513
Compiled	
1.60
3.01
3.01
5.01

515
Accumulate	in	temporary
1.27
3.01
3.01
5.01
We	achieve	performance	comparable	to	that	for	
,	except
for	the	case	of	integer	sum,	but	even	it	improves	significantly.	On
examining	the	assembly	code	generated	by	the	compiler,	we	find
an	interesting	variant	for	the	inner	loop:
We	can	compare	this	to	the	version	created	with	optimization	level
1:

We	see	that,	besides	some	reordering	of	instructions,	the	only
difference	is	that	the	more	optimized	version	does	not	contain	the
	implementing	the	read	from	the	location	designated	by
	(line	2).
A
.	
How	does	the	role	of	register	
	differ	in	these	two
loops?
B
.	
Will	the	more	optimized	version	faithfully	implement	the	C
code	of	
,	including	when	there	is	memory	aliasing
between	
	and	the	vector	data?
C
.	
Either	explain	why	this	optimization	preserves	the	desired
behavior,	or	give	an	example	where	it	would	produce
different	results	than	the	less	optimized	code.
With	this	final	transformation,	we	reached	a	point	where	we	require	just
1.25-5	clock	cycles	for	each	element	to	be	computed.	This	is	a
considerable	improvement	over	the	original	9-11	cycles	when	we	first
enabled	optimization.	We	would	now	like	to	see	just	what	factors	are
constraining	the	performance	of	our	code	and	how	we	can	improve	things
even	further.

5.7	
Understanding	Modern
Processors
Up	to	this	point,	we	have	applied	optimizations	that	did	not	rely	on	any
features	of	the	target	machine.	They	simply	reduced	the	overhead	of
procedure	calls	and	eliminated	some	of	the	critical	"optimization	blockers"
that	cause	difficulties	for	optimizing	compilers.	As	we	seek	to	push	the
performance	further,	we	must	consider	optimizations	that	exploit	the
microarchitecture
	of	the	processor—that	is,	the	underlying	system	design
by	which	a	processor	executes	instructions.	Getting	every	last	bit	of
performance	requires	a	detailed	analysis	of	the	program	as	well	as	code
generation	tuned	for	the	target	processor.	Nonetheless,	we	can	apply
some	basic	optimizations	that	will	yield	an	overall	performance
improvement	on	a	large	class	of	processors.	The	detailed	performance
results	we	report	here	may	not	hold	for	other	machines,	but	the	general
principles	of	operation	and	optimization	apply	to	a	wide	variety	of
machines.
To	understand	ways	to	improve	performance,	we	require	a	basic
understanding	of	the	microarchitectures	of	modern	processors.	Due	to
the	large	number	of	transistors	that	can	be	integrated	onto	a	single	chip,
modern	microprocessors	employ	complex	hardware	that	attempts	to
maximize	program	performance.	One	result	is	that	their	actual	operation
is	far	different	from	the	view	that	is	perceived	by	looking	at	machine-level
programs.	At	the	code	level,	it	appears	as	if	instructions	are	executed
one	at	a	time,	where	each	instruction	involves	fetching	values	from

registers	or	memory,	performing	an	operation,	and	storing	results	back	to
a	register	or	memory	location.	In	the	actual	processor,	a	number	of
instructions	
are	evaluated	simultaneously,	a	phenomenon	referred	to	as
instruction-level	parallelism
.	In	some	designs,	there	can	be	100	or	more
instructions	"in	flight."	Elaborate	mechanisms	are	employed	to	make	sure
the	behavior	of	this	parallel	execution	exactly	captures	the	sequential
semantic	model	required	by	the	machine-level	program.	This	is	one	of
the	remarkable	feats	of	modern	microprocessors:	they	employ	complex
and	exotic	microarchitectures,	in	which	multiple	instructions	can	be
executed	in	parallel,	while	presenting	an	operational	view	of	simple
sequential	instruction	execution.
Although	the	detailed	design	of	a	modern	microprocessor	is	well	beyond
the	scope	of	this	book,	having	a	general	idea	of	the	principles	by	which
they	operate	suffices	to	understand	how	they	achieve	instruction-level
parallelism.	We	will	find	that	two	different	lower	bounds	characterize	the
maximum	performance	of	a	program.	The	
latency	bound
	is	encountered
when	a	series	of	operations	must	be	performed	in	strict	sequence,
because	the	result	of	one	operation	is	required	before	the	next	one	can
begin.	This	bound	can	limit	program	performance	when	the	data
dependencies	in	the	code	limit	the	ability	of	the	processor	to	exploit
instruction-level	parallelism.	The	
throughput	bound
	characterizes	the	raw
computing	capacity	of	the	processor's	functional	units.	This	bound
becomes	the	ultimate	limit	on	program	performance.
5.7.1	
Overall	Operation

Figure	
5.11
	shows	a	very	simplified	view	of	a	modern	microprocessor.
Our	hypothetical	processor	design	is	based	loosely	on	the	structure	of
recent	Intel	processors.	These	processors	are	described	in	the	industry
as	being	
superscalar
,	which	means	they	can	perform	multiple	operations
on	every	clock	cycle	and	
out	of	order
,	meaning	that	the	order	in	which
instructions	execute	need	not	correspond	to	their	ordering	in	the
machine-level	program.	The	overall	design	has	two	main	parts:	the
instruction	control	unit
	(ICU),	which	is	responsible	for	reading	a	sequence
of	instructions	from	memory	and	generating	from	these	a	set	of	primitive
operations	to	perform	on	program	data,	and	the	
execution	unit
	(EU),
which	then	executes	these	operations.	Compared	to	the	simple	
in-order
pipeline	we	studied	in	
Chapter	
4
,	out-of-order	processors	require	far
greater	and	more	complex	hardware,	but	they	are	better	at	achieving
higher	degrees	of	instruction-level	parallelism.
The	ICU	reads	the	instructions	from	an	
instruction	cache
—a	special	high-
speed	memory	containing	the	most	recently	accessed	instructions.	In
general,	the	ICU	fetches	well	ahead	of	the	currently	executing
instructions,	so	that	it	has	enough	time	to	decode	these	and	send
operations	down	to	the	EU.	One	problem,	however,	is	that	when	a
program	hits	a	branch,
	there	are	two	possible	directions	the	program
might	go.	The	branch	can	be	
taken
,	with	control	passing	to	the	branch
target.	Alternatively,	the	branch	can	be	
not	taken
,	with	control	passing	to
the	next
1.	
We	use	the	term	"branch"	specifically	to	refer	to	conditional	jump	instructions.	Other	instructions
that	can	transfer	control	to	multiple	destinations,	such	as	procedure	return	and	indirect	jumps,
provide	similar	challenges	for	the	processor.
1

Figure	
5.11	
Block	diagram	of	an	out-of-order	processor.
The	instruction	control	unit	is	responsible	for	reading	instructions	from
memory	and	generating	a	sequence	of	primitive	operations.	The
execution	unit	then	performs	the	operations	and	indicates	whether	the
branches	were	correctly	predicted.
instruction	in	the	instruction	sequence.	Modern	processors	employ	a
technique	known	as	
branch	prediction
,	in	which	they	guess	whether	or
not	a	branch	will	be	taken	and	also	predict	the	target	address	for	the
branch.	Using	a	technique	known	as	
speculative	execution
,	the
processor	begins	fetching	and	decoding	instructions	at	where	it	predicts
the	branch	will	go,	and	even	begins	executing	these	operations	before	it
has	been	determined	whether	or	not	the	branch	prediction	was	correct.	If
it	later	determines	that	the	branch	was	predicted	incorrectly,	it	resets	the
state	to	that	at	the	branch	point	and	begins	fetching	and	executing

instructions	in	the	other	direction.	The	block	labeled	"Fetch	control"
incorporates	branch	prediction	to	perform	the	task	of	determining	which
instructions	to	fetch.
The	
instruction	decoding
	logic	takes	the	actual	program	instructions	and
converts	them	into	a	set	of	primitive	
operations
	(sometimes	referred	to	as
micro-operations
).	Each	of	these	operations	performs	some	simple
computational	task	such	as	adding	two	numbers,	reading	data	from
memory,	or	writing	data	to	memory.	For	machines	with	complex
instructions,	such	as	x86	processors,	an	instruction	
can	be	decoded	into
multiple	operations.	The	details	of	how	instructions	are	decoded	into
sequences	of	operations	varies	between	machines,	and	this	information
is	considered	highly	proprietary.	Fortunately,	we	can	optimize	our
programs	without	knowing	the	low-level	details	of	a	particular	machine
implementation.
In	a	typical	x86	implementation,	an	instruction	that	only	operates	on
registers,	such	as
is	converted	into	a	single	operation.	On	the	other	hand,	an	instruction
involving	one	or	more	memory	references,	such	as

yields	multiple	operations,	separating	the	memory	references	from	the
arithmetic	operations.	This	particular	instruction	would	be	decoded	as
three	operations:	one	to	
load
	a	value	from	memory	into	the	processor,
one	to	add	the	loaded	value	to	the	value	in	register	
,	and	one	to
store
	the	result	back	to	memory.	The	decoding	splits	instructions	to	allow
a	division	of	labor	among	a	set	of	dedicated	hardware	units.	These	units
can	then	execute	the	different	parts	of	multiple	instructions	in	parallel.
The	EU	receives	operations	from	the	instruction	fetch	unit.	Typically,	it
can	receive	a	number	of	them	on	each	clock	cycle.	These	operations	are
dispatched	to	a	set	of	
functional	units
	that	perform	the	actual	operations.
These	functional	units	are	specialized	to	handle	different	types	of
operations.
Reading	and	writing	memory	is	implemented	by	the	load	and	store	units.
The	load	unit	handles	operations	that	read	data	from	the	memory	into	the
processor.	This	unit	has	an	adder	to	perform	address	computations.
Similarly,	the	store	unit	handles	operations	that	write	data	from	the
processor	to	the	memory.	It	also	has	an	adder	to	perform	address
computations.	As	shown	in	the	figure,	the	load	and	store	units	access
memory	via	a	
data	cache
,	a	high-speed	memory	containing	the	most
recently	accessed	data	values.
With	speculative	execution,	the	operations	are	evaluated,	but	the	final
results	are	not	stored	in	the	program	registers	or	data	memory	until	the
processor	can	be	certain	that	these	instructions	should	actually	have
been	executed.	Branch	operations	are	sent	to	the	EU,	not	to	determine
where	the	branch	should	go,	but	rather	to	determine	whether	or	not	they
were	predicted	correctly.	If	the	prediction	was	incorrect,	the	EU	will

discard	the	results	that	have	been	computed	beyond	the	branch	point.	It
will	also	signal	the	branch	unit	that	the	prediction	was	incorrect	and
indicate	the	correct	branch	destination.	In	this	case,	the	branch	unit
begins	fetching	at	the	new	location.	As	we	saw	in	
Section	
3.6.6
,	such
a	
misprediction
	incurs	a	significant	cost	in	performance.	It	takes	a	while
before	the	new	instructions	can	be	fetched,	decoded,	and	sent	to	the
functional	units.
Figure	
5.11
	indicates	that	the	different	functional	units	are	designed	to
perform	different	operations.	Those	labeled	as	performing	"arithmetic
operations"	are	typically	specialized	to	perform	different	combinations	of
integer	and	floating-point	operations.	As	the	number	of	transistors	that
can	be	integrated	onto	a	single	
microprocessor	chip	has	grown	over	time,
successive	models	of	microprocessors	have	increased	the	total	number
of	functional	units,	the	combinations	of	operations	each	unit	can	perform,
and	the	performance	of	each	of	these	units.	The	arithmetic	units	are
intentionally	designed	to	be	able	to	perform	a	variety	of	different
operations,	since	the	required	operations	vary	widely	across	different
programs.	For	example,	some	programs	might	involve	many	integer
operations,	while	others	require	many	floating-point	operations.	If	one
functional	unit	were	specialized	to	perform	integer	operations	while
another	could	only	perform	floating-point	operations,	then	none	of	these
programs	would	get	the	full	benefit	of	having	multiple	functional	units.
For	example,	our	Intel	Core	i7	Has	well	reference	machine	has	eight
functional	units,	numbered	0−7.	Here	is	a	partial	list	of	each	one's
capabilities:
0
.	
Integer	arithmetic,	floating-point	multiplication,	integer	and	floating-
point	division,	branches

1
.	
Integer	arithmetic,	floating-point	addition,	integer	multiplication,
floating-point	multiplication
2
.	
Load,	address	computation
3
.	
Load,	address	computation
4
.	
Store
5
.	
Integer	arithmetic
6
.	
Integer	arithmetic,	branches
7
.	
Store	address	computation
In	the	above	list,	"integer	arithmetic"	refers	to	basic	operations,	such	as
addition,	bitwise	operations,	and	shifting.	Multiplication	and	division
require	more	specialized	resources.	We	see	that	a	store	operation
requires	two	functional	units—one	to	compute	the	store	address	and	one
to	actually	store	the	data.	We	will	discuss	the	mechanics	of	store	(and
load)	operations	in	
Section	
5.12
.
We	can	see	that	this	combination	of	functional	units	has	the	potential	to
perform	multiple	operations	of	the	same	type	simultaneously.	It	has	four
units	capable	of	performing	integer	operations,	two	that	can	perform	load
operations,	and	two	that	can	perform	floating-point	multiplication.	We	will
later	see	the	impact	these	resources	have	on	the	maximum	performance
our	programs	can	achieve.
Within	the	ICU,	the	
retirement	unit
	keeps	track	of	the	ongoing	processing
and	makes	sure	that	it	obeys	the	sequential	semantics	of	the	machine-
level	program.	Our	figure	shows	a	
register	file
	containing	the	integer,
floating-point,	and,	more	recently,	SSE	and	AVX	registers	as	part	of	the
retirement	unit,	because	this	unit	controls	the	updating	of	these	registers.
As	an	instruction	is	decoded,	information	about	it	is	placed	into	a	first-in,
first-out	queue.	This	information	remains	in	the	queue	until	one	of	two

outcomes	occurs.	First,	once	the	operations	for	the	instruction	have
completed	and	any	branch	points	leading	to	this	instruction	are	confirmed
as	having	been	correctly	predicted,	the	instruction	can	be	
retired
,	with
any	updates	to	the	program	registers	being	made.	If	some	branch	point
leading	to	this	instruction	was	mispredicted,	on	the	other	hand,	the
instruction	will	be
Aside	
The	history	of	out-of-order
processing
Out-of-order	processing	was	first	implemented	in	the	Control	Data
Corporation	6600	processor	in	1964.	Instructions	were	processed
by	10	different	functional	units,	each	of	which	could	be	operated
independently.	In	its	day,	this	machine,	with	a	clock	rate	of	10
MHz,	was	considered	the	premium	machine	for	scientific
computing.
IBM	first	implemented	out-of-order	processing	with	the	IBM	360/91
processor	in	1966,	but	just	to	execute	the	floating-point
instructions.	For	around	25	years,	out-of-order	processing	was
considered	an	exotic	technology,	found	only	in	machines	striving
for	the	highest	possible	performance,	until	IBM	reintroduced	it	in
the	RS/6000	line	of	workstations	in	1990.	This	design	became	the
basis	for	the	IBM/Motorola	PowerPC	line,	with	the	model	601,
introduced	in	1993,	becoming	the	first	single-chip	microprocessor
to	use	out-of-order	processing.	Intel	introduced	out-of-order
processing	with	its	PentiumPro	model	in	1995,	with	an	underlying
microarchitecture	similar	to	that	of	our	reference	machine.

flushed
,	discarding	any	results	that	may	have	been	computed.	By	this
means,	mispredictions	will	not	alter	the	program	state.
As	we	have	described,	any	updates	to	the	program	registers	occur	only
as	instructions	are	being	retired,	and	this	takes	place	only	after	the
processor	can	be	certain	that	any	branches	leading	to	this	instruction
have	been	correctly	predicted.	To	expedite	the	communication	of	results
from	one	instruction	to	another,	much	of	this	information	is	exchanged
among	the	execution	units,	shown	in	the	figure	as	"Operation	results."	As
the	arrows	in	the	figure	show,	the	execution	units	can	send	results
directly	to	each	other.	This	is	a	more	elaborate	form	of	the	data-
forwarding	techniques	we	incorporated	into	our	simple	processor	design
in	
Section	
4.5.5
.
The	most	common	mechanism	for	controlling	the	communication	of
operands	among	the	execution	units	is	called	
register	renaming
.	When
an	instruction	that	updates	register	
r
	is	decoded,	a	
tag	t
	is	generated
giving	a	unique	identifier	to	the	result	of	the	operation.	An	entry	
(r,	t)
	is
added	to	a	table	maintaining	the	association	between	program	register	
r
and	tag	
t
	for	an	operation	that	will	update	this	register.	When	a
subsequent	instruction	using	register	
r
	as	an	operand	is	decoded,	the
operation	sent	to	the	execution	unit	will	contain	
t
	as	the	source	for	the
operand	value.	When	some	execution	unit	completes	the	first	operation,
it	generates	a	result	
(v,	t)
,	indicating	that	the	operation	with	tag	
t
produced	value	
v
.	Any	operation	waiting	for	
t
	as	a	source	will	then	use	
v
as	the	source	value,	a	form	of	data	forwarding.	By	this	mechanism,
values	can	be	forwarded	directly	from	one	operation	to	another,	rather
than	being	written	to	and	read	from	the	register	file,	enabling	the	second
operation	to	begin	as	soon	as	the	first	has	completed.	The	renaming
table	only	contains	entries	for	registers	having	pending	write	operations.

When	a	decoded	instruction	requires	a	register	
r
,	and	there	is	no	tag
associated	with	this	register,	the	operand	is	retrieved	directly	from	the
register	file.	With	register	renaming,	an	entire	sequence	of	operations	can
be	performed	speculatively,	even	though	the	registers	are	updated	only
after	the	processor	is	certain	of	the	branch	outcomes.
Integer
Floating	point
Operation
Latency
Issue
Capacity
Latency
Issue
Capacity
Addition
1
1
4
3
1
1
Multiplication
3
1
1
5
1
2
Division
3−30
3−30
1
3−15
3−15
1
Figure	
5.12	
Latency,	issue	time,	and	capacity	characteristics	of
reference	machine	operations.
Latency	indicates	the	total	number	of	clock	cycles	required	to	perform	the
actual	operations,	while	issue	time	indicates	the	minimum	number	of
cycles	between	two	independent	operations.	The	capacity	indicates	how
many	of	these	operations	can	be	issued	simultaneously.	The	times	for
division	depend	on	the	data	values.
5.7.2	
Functional	Unit	Performance
Figure	
5.12
	documents	the	performance	of	some	of	the	arithmetic
operations	for	our	Intel	Core	i7	Haswell	reference	machine,	determined
by	both	measurements	and	by	reference	to	Intel	literature	[
49
].	These

timings	are	typical	for	other	processors	as	well.	Each	operation	is
characterized	by	its	
latency
,	meaning	the	total	time	required	to	perform
the	operation,	the	
issue	time
,	meaning	the	minimum	number	of	clock
cycles	between	two	independent	operations	of	the	same	type,	and	the
capacity
,	indicating	the	number	of	functional	units	capable	of	performing
that	operation.
We	see	that	the	latencies	increase	in	going	from	integer	to	floating-point
operations.	We	see	also	that	the	addition	and	multiplication	operations	all
have	issue	times	of	1,	meaning	that	on	each	clock	cycle,	the	processor
can	start	a	new	one	of	these	operations.	This	short	issue	time	is	achieved
through	the	use	of	
pipelining
.	A	pipelined	function	unit	is	implemented	as
a	series	of	
stages
,	each	of	which	performs	part	of	the	operation.	For
example,	a	typical	floating-point	adder	contains	three	stages	(and	hence
the	three-cycle	latency):	one	to	process	the	exponent	values,	one	to	add
the	fractions,	and	one	to	round	the	result.	The	arithmetic	operations	can
proceed	through	the	stages	in	close	succession	rather	than	waiting	for
one	operation	to	complete	before	the	next	begins.	This	capability	can	be
exploited	only	if	there	are	successive,	logically	independent	operations	to
be	performed.	Functional	units	with	issue	times	of	1	cycle	are	said	to	be
fully	pipelined:
	they	can	start	a	new	operation	every	clock	cycle.
Operations	with	capacity	greater	than	1	arise	due	to	the	capabilities	of
the	multiple	functional	units,	as	was	described	earlier	for	the	reference
machine.
We	see	also	that	the	divider	(used	for	integer	and	floating-point	division,
as	well	as	floating-point	square	root)	is	not	pipelined—its	issue	time
equals	its	latency.	What	this	means	is	that	the	divider	must	perform	a
complete	division	before	it	can	begin	anew	one.	We	also	see	that	the
latencies	and	issue	times	for	division	are	given	as	ranges,	because	some

combinations	of	dividend	and	divisor	require	more	steps	than	others.	The
long	latency	and	issue	times	of	division	make	it	a	comparatively	costly
operation.
A	more	common	way	of	expressing	issue	time	is	to	specify	the	maximum
throughput
	of	the	unit,	defined	as	the	reciprocal	of	the	issue	time.	A	fully
pipelined	functional	unit	has	a	maximum	throughput	of	1	operation	per
clock	cycle,	while	units	with	higher	issue	times	have	lower	maximum
throughput.	Having	multiple	functional	units	can	increase	throughput
even	further.	For	an	operation	with	capacity	
C
	and	issue	time	
I
,	the
processor	can	potentially	achieve	a	throughput	of	
C/I
	operations	per
clock	cycle.	For	example,	our	reference	machine	is	capable	of	performing
floating-point	multiplication	operations	at	a	rate	of	2	per	clock	cycle.	We
will	see	how	this	capability	can	be	exploited	to	increase	program
performance.
Circuit	designers	can	create	functional	units	with	wide	ranges	of
performance	characteristics.	Creating	a	unit	with	short	latency	or	with
pipelining	requires	more	hardware,	especially	for	more	complex	functions
such	as	multiplication	and	floating-point	operations.	Since	there	is	only	a
limited	amount	of	space	for	these	units	on	the	microprocessor	chip,	CPU
designers	must	carefully	balance	the	number	of	functional	units	and	their
individual	performance	to	achieve	optimal	overall	performance.	They
evaluate	many	different	benchmark	programs	and	dedicate	the	most
resources	to	the	most	critical	operations.	As	
Figure	
5.12
	indicates,
integer	multiplication	and	floating-point	multiplication	and	addition	were
considered	important	operations	in	the	design	of	the	Core	i7	Haswell
processor,	even	though	a	significant	amount	of	hardware	is	required	to
achieve	the	low	latencies	and	high	degree	of	pipelining	shown.	On	the

other	hand,	division	is	relatively	infrequent	and	difficult	to	implement	with
either	short	latency	or	full	pipelining.
The	latencies,	issue	times,	and	capacities	of	these	arithmetic	operations
can	affect	the	performance	of	our	combining	functions.	We	can	express
these	effects	in	terms	of	two	fundamental	bounds	on	the	CPE	values:
Integer
Floating	point
Bound
+
*
+
*
Latency
1.00
3.00
3.00
5.00
Throughput
0.50
1.00
1.00
0.50
The	
latency	bound
	gives	a	minimum	value	for	the	CPE	for	any	function
that	must	perform	the	combining	operation	in	a	strict	sequence.	The
throughput	bound
	gives	a	minimum	bound	for	the	CPE	based	on	the
maximum	rate	at	which	the	functional	units	can	produce	results.	For
example,	since	there	is	only	one	integer	multiplier,	and	it	has	an	issue
time	of	1	clock	cycle,	the	processor	cannot	possibly	sustain	a	rate	of
more	than	1	multiplication	per	clock	cycle.	On	the	other	hand,	with	four
functional	units	capable	of	performing	integer	addition,	the	processor	can
potentially	sustain	a	rate	of	4	operations	per	cycle.	Unfortunately,	the
need	to	read	elements	from	memory	creates	an	additional	throughput
bound.	The	two	load	units	limit	the	processor	to	reading	at	most	2	data
values	per	clock	cycle,	yielding	a	throughput	bound	of	0.50.	We	will
demonstrate	the	effect	of	both	the	latency	and	throughput	bounds	with
different	versions	of	the	combining	functions.

5.7.3	
An	Abstract	Model	of
Processor	Operation
As	a	tool	for	analyzing	the	performance	of	a	machine-level	program
executing	on	a	modern	processor,	we	will	use	a	
data-flow
	representation
of	programs,	a	graphical	notation	showing	how	the	data	dependencies
between	the	different	operations	constrain	the	order	in	which	they	are
executed.	These	constraints	then	lead	to	
critical	paths
	in	the	graph,
putting	a	lower	bound	on	the	number	of	clock	cycles	required	to	execute
a	set	of	machine	instructions.
Before	proceeding	with	the	technical	details,	it	is	instructive	to	examine
the	CPE	measurements	obtained	for	function	
,	our	fastest	code
up	to	this	point:
Integer
Floating	point
Function
Page
Method
+
*
+
*
515
Accumulate	in	temporary
1.27
3.01
3.01
5.01
Latency	bound
1.00
3.00
3.00
5.00
Throughput	bound
0.50
1.00
1.00
0.50
We	can	see	that	these	measurements	match	the	latency	bound	for	the
processor,	except	for	the	case	of	integer	addition.	This	is	not	a
coincidence—it	indicates	that	the	performance	of	these	functions	is
dictated	by	the	latency	of	the	sum	or	product	computation	being

performed.	Computing	the	product	or	sum	of	
n
	elements	requires	around
L	·	n
	+	
K
	clock	cycles,	where	
L
	is	the	latency	of	the	combining	operation
and	
K
	represents	the	overhead	of	calling	the	function	and	initiating	and
terminating	the	loop.	The	CPE	is	therefore	equal	to	the	latency	bound	
L
.
From	Machine-Level	Code	to	Data-Flow
Graphs
Our	data-flow	representation	of	programs	is	informal.	We	use	it	as	a	way
to	visualize	how	the	data	dependencies	in	a	program	dictate	its
performance.	We	present	the	data-flow	notation	by	working	with	
(
Figure	
5.10
)	as	an	example.	We	focus	just	on	the	computation
performed	by	the	loop,	since	this	is	the	dominating	factor	in	performance
for	large	vectors.	We	consider	the	case	of	data	type	
	with
multiplication	as	the	combining	operation.	Other	combinations	of	data
type	and	operation	yield	similar	code.	The	compiled	code	for	this	loop
consists	of	four	instructions,	with	registers	
	holding	a	pointer	to	the
i
th	element	of	array	data,	
	holding	a	pointer	to	the	end	of	the	array,
and	
	holding	the	accumulated	value	
.

Figure	
5.13	
Graphical	representation	of	inner-loop	code	for	
Instructions	are	dynamically	translated	into	one	or	two	operations,	each
of	which	receives	values	from	other	operations	or	from	registers	and
produces	values	for	other	operations	and	for	registers.	We	show	the
target	of	the	final	instruction	as	the	label	loop.	It	jumps	to	the	first
instruction	shown.
As	
Figure	
5.13
	indicates,	with	our	hypothetical	processor	design,	the
four	instructions	are	expanded	by	the	instruction	decoder	into	a	series	of
five	
operations
,	with	the	initial	multiplication	instruction	being	expanded
into	a	load	operation	to	read	the	source	operand	from	memory,	and	a	mul
operation	to	perform	the	multiplication.
As	a	step	toward	generating	a	data-flow	graph	representation	of	the
program,	the	boxes	and	lines	along	the	left-hand	side	of	
Figure	
5.13
show	how	the	registers	are	used	and	updated	by	the	different	operations,
with	the	boxes	along	the	top	representing	the	register	values	at	the
beginning	of	the	loop,	and	those	along	the	bottom	representing	the
values	at	the	end.	For	example,	register	
	is	only	used	as	a	source

value	by	the	
	operation,	and	so	the	register	has	the	same	value	at	the
end	of	the	loop	as	at	the	beginning.	Register	
,	on	the	other	hand,	is
both	used	and	updated	within	the	loop.	Its	initial	value	is	used	by	the	load
and	add	operations;	its	new	value	is	generated	by	the	add	operation,
which	is	then	used	by	the	
	operation.	Register	
	is	also	updated
within	the	loop	by	the	mul	operation,	which	first	uses	the	initial	value	as	a
source	value.
Some	of	the	operations	in	
Figure	
5.13
	produce	values	that	do	not
correspond	to	registers.	We	show	these	as	arcs	between	operations	on
the	right-hand	side.	The	load	operation	reads	a	value	from	memory	and
passes	it	directly	to	the	
	operation.	Since	these	two	operations	arise
from	decoding	a	single	
	instruction,	there	is	no	register	associated
with	the	intermediate	value	passing	between	them.	The	
	operation
updates	the	condition	codes,	and	these	are	then	tested	by	the	
operation.
For	a	code	segment	forming	a	loop,	we	can	classify	the	registers	that	are
accessed	into	four	categories:


Figure	
5.14	
Abstracting	
	operations	as	a	data-flow	graph.
We	rearrange	the	operators	of	
Figure	
5.13
	to	more	clearly	show	the
data	dependencies	(a),	and	then	further	show	only	those	operations	that
use	values	from	one	iteration	to	produce	new	values	for	the	next	(b).
Read-only.	
These	are	used	as	source	values,	either	as	data	or	to
compute	memory	addresses,	but	they	are	not	modified	within	the
loop.	The	only	read	only	register	for	the	loop	in	
	is	
.
Write-only.	
These	are	used	as	the	destinations	of	data-movement
operations.	There	are	no	such	registers	in	this	loop.
Local.	
These	are	updated	and	used	within	the	loop,	but	there	is	no
dependency	from	one	iteration	to	another.	The	condition	code
registers	are	examples	for	this	loop:	they	are	updated	by	the	
operation	and	used	by	the	
	operation,	but	this	dependency	is
contained	within	individual	iterations.
Loop.	
These	are	used	both	as	source	values	and	as	destinations	for
the	loop,	with	the	value	generated	in	one	iteration	being	used	in
another.	We	can	see	that	
	and	
	are	loop	registers	for
,	corresponding	to	program	values	
	and	
.
As	we	will	see,	the	chains	of	operations	between	loop	registers	determine
the	performance-limiting	data	dependencies.
Figure	
5.14
	shows	further	refinements	of	the	graphical	representation
of	
Figure	
5.13
,	with	a	goal	of	showing	only	those	operations	and	data
dependencies	that	affect	the	program	execution	time.	We	see	in	
Figure
5.14(a)
	that	we	rearranged	the	operators	to	show	more	clearly	the	flow
of	data	from	the	source	registers	at	the	top	(both	read-only	and	loop

registers)	and	to	the	destination	registers	at	the	bottom	(both	write-only
and	loop	registers).
In	
Figure	
5.14(a)
,	we	also	color	operators	white	if	they	are	not	part	of
some	chain	of	dependencies	between	loop	registers.	For	this	example,
the	comparison	(cmp)	and	branch	(jne)	operations	do	not	directly	affect
the	flow	of	data	in	the	program.	We	assume	that	the	instruction	control
unit	predicts	that	branch	will	be	taken,	and	hence	the	program	will
continue	looping.	The	purpose	of	the	compare	and	branch	operations	is
to	test	the	branch	condition	and	notify	the	ICU	if	it	is	
not	taken.	We
assume	this	checking	can	be	done	quickly	enough	that	it	does	not	slow
down	the	processor.
In	
Figure	
5.14(b)
,	we	have	eliminated	the	operators	that	were	colored
white	on	the	left,	and	we	have	retained	only	the	loop	registers.	What	we
have	left	is	an	abstract	template	showing	the	data	dependencies	that
form	among	loop	registers	due	to	one	iteration	of	the	loop.	We	can	see	in
this	diagram	that	there	are	two	data	dependencies	from	one	iteration	to
the	next.	Along	one	side,	we	see	the	dependencies	between	successive
values	of	program	value	
,	stored	in	register	
.	The	loop	computes
a	new	value	for	
	by	multiplying	the	old	value	by	a	data	element,
generated	by	the	load	operation.	Along	the	other	side,	we	see	the
dependencies	between	successive	values	of	the	pointer	to	the	
i
th	data
element.	On	each	iteration,	the	old	value	is	used	as	the	address	for	the
load	operation,	and	it	is	also	incremented	by	the	add	operation	to
compute	its	new	value.
Figure	
5.15
	shows	the	data-flow	representation	of	
n
	iterations	by	the
inner	loop	of	function	
.	This	graph	was	obtained	by	simply

replicating	the	template	shown	in	
Figure	
5.14(b)
n
times.Wecan	see
that	the	program	has	two	chains	of	data
Figure	
5.15	
Data-flow	representation	of	computation	by	
n
	iterations
of	the	inner	loop	of	
.
The	sequence	of	multiplication	operations	forms	a	critical	path	that	limits
program	performance.
dependencies,	corresponding	to	the	updating	of	program	values	
	and
	with	operations	mul	and	add,	respectively.	Given	that	floating-
point	multiplication	has	a	latency	of	5	cycles,	while	integer	addition	has	a
latency	of	1	cycle,	we	can	see	that	the	chain	on	the	left	will	form	a	
critical
path
,	requiring	5
n
	cycles	to	execute.	The	chain	on	the	right	would	require

only	
n
	cycles	to	execute,	and	so	it	does	not	limit	the	program
performance.
Figure	
5.15
	demonstrates	why	we	achieved	a	CPE	equal	to	the
latency	bound	of	5	cycles	for	
,	when	performing	floating-point
multiplication.	When	executing	the	function,	the	floating-point	multiplier
becomes	the	limiting	resource.	The	other	operations	required	during	the
loop—manipulating	and	testing	pointer	value	
	and	reading	data
from	memory—proceed	in	parallel	with	the	multiplication.	As	each
successive	value	of	
	is	computed,	it	is	fed	back	around	to	compute
the	next	value,	but	this	will	not	occur	until	5	cycles	later.
The	flow	for	other	combinations	of	data	type	and	operation	are	identical
to	those	shown	in	
Figure	
5.15
,	but	with	a	different	data	operation
forming	the	chain	of	data	dependencies	shown	on	the	left.	For	all	of	the
cases	where	the	operation	has	a	latency	
L
	greater	than	1,	we	see	that
the	measured	CPE	is	simply	
L
,	indicating	that	this	chain	forms	the
performance-limiting	critical	path.
Other	Performance	Factors
For	the	case	of	integer	addition,	on	the	other	hand,	our	measurements	of
	show	a	CPE	of	1.27,	slower	than	the	CPE	of	1.00	we	would
predict	based	on	the	chains	of	dependencies	formed	along	either	the	left-
or	the	right-hand	side	of	the	graph	of	
Figure	
5.15
.	This	illustrates	the
principle	that	the	critical	paths	in	a	data-flow	representation	provide	only
a	
lower
	bound	on	how	many	cycles	a	program	will	require.	Other	factors
can	also	limit	performance,	including	the	total	number	of	functional	units
available	and	the	number	of	data	values	that	can	be	passed	among	the

functional	units	on	any	given	step.	For	the	case	of	integer	addition	as	the
combining	operation,	the	data	operation	is	sufficiently	fast	that	the	rest	of
the	operations	cannot	supply	data	fast	enough.	Determining	exactly	why
the	program	requires	1.27	cycles	per	element	would	require	a	much	more
detailed	knowledge	of	the	hardware	design	than	is	publicly	available.
To	summarize	our	performance	analysis	of	
:	our	abstract	data-
flow	representation	of	program	operation	showed	that	
	has	a
critical	path	of	length	
L	·	n
	caused	by	the	successive	updating	of	program
value	
,	and	this	path	limits	the	CPE	to	at	least	
L
.	This	is	indeed	the
CPE	we	measure	for	all	cases	except	integer	addition,	which	has	a
measured	CPE	of	1.27	rather	than	the	CPE	of	1.00	we	would	expect	from
the	critical	path	length.
It	may	seem	that	the	latency	bound	forms	a	fundamental	limit	on	how	fast
our	combining	operations	can	be	performed.	Our	next	task	will	be	to
restructure	the	operations	to	enhance	instruction-level	parallelism.	We
want	to	transform	the	program	in	such	a	way	that	our	only	limitation
becomes	the	throughput	bound,	yielding	CPEs	below	or	close	to	1.00.
Practice	Problem	
5.5	
(solution	page	
575
)
Supposewewishtowriteafunctiontoevaluateapolynomial,	where	a
polynomial	of	degree	
n
	is	defined	to	have	a	set	of	coefficients	
a
,
a
,	
a
,	.	.	.,	
a
.	For	a	value	
x
,	we	evaluate	the	polynomial	by
computing
0
1
2
n
a
0
+
a
1
x
+
a
2
x
2
+
…
+
a
n
x
n
(5.2)

This	evaluation	can	be	implemented	by	the	following	function,
having	as	arguments	an	array	of	coefficients	a,	a	value	
,	and	the
polynomial	degree	
	(the	value	
n
	in	Equation	5.2).	In	this
function,	we	compute	both	the	successive	terms	of	the	equation
and	the	successive	powers	of	
x
	within	a	single	loop:
⁁
A
.	
For	degree	
n
,	how	many	additions	and	how	many
multiplications	does	this	code	perform?
B
.	
On	our	reference	machine,	with	arithmetic	operations
having	the	latencies	shown	in	
Figure	
5.12
,	we	measure
the	CPE	for	this	function	to	be	5.00.	Explain	how	this	CPE
arises	based	on	the	data	dependencies	formed	between
iterations	due	to	the	operations	implementing	lines	7-8	of
the	function.

Practice	Problem	
5.6	
(solution	page	
575
)
Let	us	continue	exploring	ways	to	evaluate	polynomials,	as
described	in	
Practice	Problem	
5.5
.	We	can	reduce	the	number
of	multiplications	in	evaluating	a	polynomial	by	applying	
Horner's
method
,	named	after	British	mathematician	William	G.	Horner
(1786-1837).	The	idea	is	to	repeatedly	factor	out	the	powers	of	
x
to	get	the	following	evaluation:
Using	Horner's	method,	we	can	implement	polynomial	evaluation
using	the	following	code:
A
.	
For	degree	
n
,	how	many	additions	and	how	many
multiplications	does	this	code	perform?
B
.	
On	our	reference	machine,	with	the	arithmetic	operations
having	the	latencies	shown	in	
Figure	
5.12
,	we	measure
the	CPE	for	this	function	to	be	8.00.	Explain	how	this	CPE
a
0
+
x
(
a
1
+
x
(
a
2
+
…
+
x
(
a
n
-
1
+
x
a
n
)
…
)
)
(5.3)

arises	based	on	the	data	dependencies	formed	between
iterations	due	to	the	operations	implementing	line	7	of	the
function.
C
.	
Explain	how	the	function	shown	in	
Practice	Problem	
5.5
can	run	faster,	even	though	it	requires	more	operations.

5.8	
Loop	Unrolling
Loop	unrolling	is	a	program	transformation	that	reduces	the	number	of
iterations	for	a	loop	by	increasing	the	number	of	elements	computed	on
each	iteration.	We	saw	an	example	of	this	with	the	function	
	(
Figure
5.1
),	where	each	iteration	computes	two	elements	of	the	prefix	sum,
thereby	halving	the	total	number	of	iterations	required.	Loop	unrolling	can
improve	performance	in	two	ways.	First,	it	reduces	the	number	of
operations	that	do	not	contribute	directly	to	the	program	result,	such	as
loop	indexing	and	conditional	branching.	Second,	it	exposes	ways	in
which	we	can	further	transform	the	code	to	reduce	the	number	of
operations	in	the	critical	paths	of	the	overall	computation.	In	this	section,
we	will	examine	simple	loop	unrolling,	without	any	further
transformations.
Figure	
5.16
	shows	a	version	of	our	combining	code	using	what	we	will
refer	to	as	"2	×	1	loop	unrolling."	The	first	loop	steps	through	the	array
two	elements	at	a	time.	That	is,	the	loop	index	
	is	incremented	by	2	on
each	iteration,	and	the	combining	operation	is	applied	to	array	elements	
i
and	
i
	+	1	in	a	single	iteration.
In	general,	the	vector	length	will	not	be	a	multiple	of	2.	We	want	our	code
to	work	correctly	for	arbitrary	vector	lengths.	We	account	for	this
requirement	in	two	ways.	First,	we	make	sure	the	first	loop	does	not
overrun	the	array	bounds.	For	a	vector	of	length	
n
,	we	set	the	loop	limit	to
be	
n
	−	1.	We	are	then	assured	that	the	loop	will	only	be	executed	when

the	loop	index	
i
	satisfies	
i
	<	
n
	−	1,	and	hence	the	maximum	array	index	
i
+	1	will	satisfy	
i
	+	1	<	(
n
	−	1)	+	1	=	
n
.
We	can	generalize	this	idea	to	unroll	a	loop	by	any	factor	
k
,	yielding	
k
	×	1
loop	unrolling.
	To	do	so,	we	set	the	upper	limit	to	be	
n
	−	
k
	+	1	and	within
the	loop	apply	the	combining	operation	to	elements	
i
	through	
i
	+	
k
	−	1.
Loop	index	
	is	incremented	by	
k
	in	each	iteration.	The	maximum	array
index	
i
	+	
k
	−	1	will	then	be	less	than	
n
.	We	include	the	second	loop	to
step	through	the	final	few	elements	of	the	vector	one	at	a	time.	The	body
of	this	loop	will	be	executed	between	0	and	
k
	−	1	times.	For	
k
	=	2,	we
could	use	a	simple	conditional	statement

Figure	
5.16	
Applying	2	×	1	loop	unrolling.
This	transformation	can	reduce	the	effect	of	loop	overhead.
to	optionally	add	a	final	iteration,	as	we	did	with	the	function	
(
Figure	
5.1
).	For	
k
	>	2,	the	finishing	cases	are	better	expressed	with	a
loop,	and	so	we	adopt	this	programming	convention	for	
k
	=	2	as	well.	We
refer	to	this	transformation	as	"
k
	×	1	loop	unrolling,"	since	we	unroll	by	a
factor	of	
k
	but	accumulate	values	in	a	single	variable	
.
Practice	Problem	
5.7	
(solution	page
575
)
Modify	the	code	for	
	to	unroll	the	loop	by	a	factor	
k
	=	5.
When	we	measure	the	performance	of	unrolled	code	for	unrolling	factors
k
	=	2	(
)	and	
k
	=	3,	we	get	the	following	results:
Integer
Floating	point
Function
Page
Method
+
*
+
*
515
No	unrolling
1.27
3.01
3.01
5.01

532
2	×	1	unrolling
1.01
3.01
3.01
5.01
3	×	1	unrolling
1.01
3.01
3.01
5.01
Latency	bound
1.00
3.00
3.00
5.00
Throughput	bound
0.50
1.00
1.00
0.50
Figure	
5.17	
CPE	performance	for	different	degrees	of	
k
	×	1	loop
unrolling.
Only	integer	addition	improves	with	this	transformation.
We	see	that	the	CPE	for	integer	addition	improves,	achieving	the	latency
bound	of	1.00.	This	result	can	be	attributed	to	the	benefits	of	reducing
loop	overhead	operations.	By	reducing	the	number	of	overhead
operations	relative	to	the	number	of	additions	required	to	compute	the
vector	sum,	we	can	reach	the	point	where	the	1-cycle	latency	of	integer
addition	becomes	the	performance-limiting	factor.	On	the	other	hand,
none	of	the	other	cases	improve—they	are	already	at	their	latency
bounds.	
Figure	
5.17
	shows	CPE	measurements	when	unrolling	the
loop	by	up	to	a	factor	of	10.	We	see	that	the	trends	we	observed	for
unrolling	by	2	and	3	continue—none	go	below	their	latency	bounds.
To	understand	why	
k
	×	1	unrolling	cannot	improve	performance	beyond
the	latency	bound,	let	us	examine	the	machine-level	code	for	the	inner

loop	of	
,	having	
k
	=	2.	The	following	code	gets	generated	when
type	
	is	double,	and	the	operation	is	multiplication:
We	can	see	that	
GCC
	
uses	a	more	direct	translation	of	the	array
referencing	seen	in	the	C	code,	compared	to	the	pointer-based	code
generated	for	
.
	Loop	index	
	is	held	in	register	
,	and	the
address	of	data	is	held	in	register	
.	As	before,	the	accumulated	value
	is	held	in	vector	register	
.	The	loop	unrolling	leads	to	two
	instructions—one	to	add	
	to	
,	and
2.	
The	
GCC
	
optimizer	operates	by	generating	multiple	variants	of	a	function	and	then	choosing	one
that	it	predicts	will	yield	the	best	performance	and	smallest	code	size.	As	a	consequence,	small
changes	in	the	source	code	can	yield	widely	varying	forms	of	machine	code.	We	have	found	that
the	choice	of	pointer-based	or	array-based	code	has	no	impact	on	the	performance	of	programs
running	on	our	reference	machine.
2

Figure	
5.18	
Graphical	representation	of	inner-loop	code	for	
.
Each	iteration	has	two	
	instructions,	each	of	which	is	translated
into	a	load	and	a	mul	operation.
Figure	
5.19	
Abstracting	
	operations	as	a	data-flow	graph.
We	rearrange,	simplify,	and	abstract	the	representation	of	
Figure	
5.18
to	show	the	data	dependencies	between	successive	iterations	(a).	We
see	that	each	iteration	must	perform	two	multiplications	in	sequence	(b).

the	second	to	add	
	to	
.	
Figure	
5.18
	shows	a	graphical
representation	of	this	code.	The	
	instructions	each	get	translated
into	two	operations:	one	to	load	an	array	element	from	memory	and	one
to	multiply	this	value	by	the	accumulated	value.	We	see	here	that	register
	gets	read	and	written	twice	in	each	execution	of	the	loop.	We	can
rearrange,	simplify,	and	abstract	this	graph,	following	the	process	shown
in	
Figure	
5.19(a)
,	to	obtain	the	template	shown	in	
Figure	
5.19(b)
.
We	then	replicate	this	template	
n
/2	times	to	show	the	computation	for	a
vector	of	length	
n
,	obtaining	the	data-flow	representation

Figure	
5.20	
Data-flow	representation	of	
	operating	on	a
vector	of	length	
n
.
Even	though	the	loop	has	been	unrolled	by	a	factor	of	2,	there	are	still	
n
mul	operations	along	the	critical	path.
shown	in	
Figure	
5.20
.	We	see	here	that	there	is	still	a	critical	path	of	
n
mul	operations	in	this	graph—there	are	half	as	many	iterations,	but	each
iteration	has	two	multiplication	operations	in	sequence.	Since	the	critical

path	was	the	limiting	factor	for	the	performance	of	the	code	without	loop
unrolling,	it	remains	so	with	
k
	×	1	loop	unrolling.
Aside	
Getting	the	compiler	to	unroll
loops
Loop	unrolling	can	easily	be	performed	by	a	compiler.	Many
compilers	do	this	as	part	of	their	collection	of	optimizations.	
GCC
will	perform	some	forms	of	loop	unrolling	when	invoked	with
optimization	level	3	or	higher.

5.9	
Enhancing	Parallelism
At	this	point,	our	functions	have	hit	the	bounds	imposed	by	the	latencies
of	the	arithmetic	units.	As	we	have	noted,	however,	the	functional	units
performing	addition	and	multiplication	are	all	fully	pipelined,	meaning	that
they	can	start	new	operations	every	clock	cycle,	and	some	of	the
operations	can	be	performed	by	multiple	functional	units.	The	hardware
has	the	potential	to	perform	multiplications	and	additions	at	a	much
higher	rate,	but	our	code	cannot	take	advantage	of	this	capability,	even
with	loop	unrolling,	since	we	are	accumulating	the	value	as	a	single
variable	
.	We	cannot	compute	a	new	value	for	
	until	the	preceding
computation	has	completed.	Even	though	the	functional	unit	computing	a
new	value	for	
	can	start	a	new	operation	every	clock	cycle,	it	will	only
start	one	every	
L
	cycles,	where	
L
	is	the	latency	of	the	combining
operation.	We	will	now	investigate	ways	to	break	this	sequential
dependency	and	get	performance	better	than	the	latency	bound.
5.9.1	
Multiple	Accumulators
For	a	combining	operation	that	is	associative	and	commutative,	such	as
integer	addition	or	multiplication,	we	can	improve	performance	by	splitting
the	set	of	combining	operations	into	two	or	more	parts	and	combining	the
results	at	the	end.	For	example,	let	
P
	denote	the	product	of	elements	
a
,
a
,	.	.	.,	
a
:
n
0
1
n
−1

Assuming	
n
	is	even,	we	can	also	write	this	as	
P
	=	
PE
	×	
PO
,	where	
PE
is	the	product	of	the	elements	with	even	indices,	and	
PO
	is	the	product
of	the	elements	with	odd	indices:
Figure	
5.21
	shows	code	that	uses	this	method.	It	uses	both	two-way
loop	unrolling,	to	combine	more	elements	per	iteration,	and	two-way
parallelism,	accumulating	elements	with	even	indices	in	variable	
and	elements	with	odd	indices	in	variable	
.	We	therefore	refer	to	this
as	"2	×	2	loop	unrolling."	As	before,	we	include	a	second	loop	to
accumulate	any	remaining	array	elements	for	the	case	where	the	vector
length	is	not	a	multiple	of	2.	We	then	apply	the	combining	operation	to
	and	
	to	compute	the	final	result.
Comparing	loop	unrolling	alone	to	loop	unrolling	with	two-way	parallelism,
we	obtain	the	following	performance:
p
n
=
∏
i
=
0
n
-
1
a
i
n
n
n
n
n
P
E
n
=
∏
i
=
0
n
/
2
−
1
a
2
i
P
O
n
=
∏
i
=
0
n
/
2
−
1
a
2
i
+
1

Figure	
5.21	
Applying	2	×	2	loop	unrolling.
By	maintaining	multiple	accumulators,	this	approach	can	make	better	use
of	the	multiple	functional	units	and	their	pipelining	capabilities.
Integer
Floating	point
Function
Page
Method
+
*
+
*
515
Accumulate	in	temporary
1.27
3.01
3.01
5.01
532
2	×	1	unrolling
1.01
3.01
3.01
5.01
537
2	×	2	unrolling
0.81
1.51
1.51
2.51
Latency	bound
1.00
3.00
3.00
5.00

Throughput	bound
0.50
1.00
1.00
0.50
We	see	that	we	have	improved	the	performance	for	all	cases,	with
integer	product,	floating-point	addition,	and	floating-point	multiplication
improving	by	a	factor	of	around	2,	and	integer	addition	improving
somewhat	as	well.	Most	significantly,	we	have	broken	through	the	barrier
imposed	by	the	latency	bound.	The	processor	no	longer	needs	to	delay
the	start	of	one	sum	or	product	operation	until	the	previous	one	has
completed.
To	understand	the	performance	of	
,	we	start	with	the	code	and
operation	sequence	shown	in	
Figure	
5.22
.	We	can	derive	a	template
showing	the
Figure	
5.22	
Graphical	representation	of	inner-loop	code	for	
.
Each	iteration	has	two	
	instructions,	each	of	which	is	translated
into	a	load	and	a	mul	operation.

Figure	
5.23	
Abstracting	
	operations	as	a	data-flow	graph.
We	rearrange,	simplify,	and	abstract	the	representation	of	
Figure	
5.22
to	show	the	data	dependencies	between	successive	iterations	(a).	We
see	that	there	is	no	dependency	between	the	two	mul	operations	(b).
data	dependencies	between	iterations	through	the	process	shown	in
Figure	
5.23
.	As	with	
,	the	inner	loop	contains	two	
operations,	but	these	instructions	translate	into	mul	operations	that	read
and	write	separate	registers,	with	no	data	dependency	between	them
(
Figure	
5.23(b)
).	We	then	replicate	this	template	
n
/2	times	(
Figure
5.24
),	modeling	the	execution	of	the	function	on	a	vector	of	length	
n
.
We	see	that	we	now	have	two	critical	paths,	one	corresponding	to
computing	the	product	of	even-numbered	elements	(program	value	
)
and

Figure	
5.24	
Data-flow	representation	of	
	operating	on	a
vector	of	length	
n
.
We	now	have	two	critical	paths,	each	containing	
n
/2	operations.
one	for	the	odd-numbered	elements	(program	value	
).	Each	of	these
critical	paths	contains	only	
n
/2	operations,	thus	leading	to	a	CPE	of
around	5.00/2	=	2.50.	A	similar	analysis	explains	our	observed	CPE	of
around	
L
/2	for	operations	with	latency	
L
	for	the	different	combinations	of
data	type	and	combining	operation.	Operationally,	the	programs	are
exploiting	the	capabilities	of	the	functional	units	to	increase	their

utilization	by	a	factor	of	2.	The	only	exception	is	for	integer	addition.	We
have	reduced	the	CPE	to	below	1.0,	but	there	is	still	too	much	loop
overhead	to	achieve	the	theoretical	limit	of	0.50.
We	can	generalize	the	multiple	accumulator	transformation	to	unroll	the
loop	by	a	factor	of	
k
	and	accumulate	
k
	values	in	parallel,	yielding	
k
	×	
k
loop	unrolling.
	
Figure	
5.25
	demonstrates	the	effect	of	applying	this
transformation	for	values	up	to	
k
	=	10.	We	can	see	that,	for	sufficiently
large	values	of	
k
,	the	program	can
Figure	
5.25	
CPE	performance	of	
k
	×	
k
	loop	unrolling.
All	of	the	CPEs	improve	with	this	transformation,	achieving	near	or	at
their	throughput	bounds.
achieve	nearly	the	throughput	bounds	for	all	cases.	Integer	addition
achieves	a	CPE	of	0.54	with	
k
	=	7,	close	to	the	throughput	bound	of	0.50
caused	by	the	two	load	units.	Integer	multiplication	and	floating-point
addition	achieve	CPEs	of	1.01	when	
k
	≥	3,	approaching	the	throughput
bound	of	1.00	set	by	their	functional	units.	Floating-point	multiplication
achieves	a	CPE	of	0.51	for	
k
	≥	10,	approaching	the	throughput	bound	of
0.50	set	by	the	two	floating-point	multipliers	and	the	two	load	units.	It	is
worth	noting	that	our	code	is	able	to	achieve	nearly	twice	the	throughput

with	floating-point	multiplication	as	it	can	with	floating-point	addition,	even
though	multiplication	is	a	more	complex	operation.
In	general,	a	program	can	achieve	the	throughput	bound	for	an	operation
only	when	it	can	keep	the	pipelines	filled	for	all	of	the	functional	units
capable	of	performing	that	operation.	For	an	operation	with	latency	
L
	and
capacity	
C
,	this	requires	an	unrolling	factor	
k
	≥	
C	·	L.
	For	example,
floating-point	multiplication	has	
C
	=	2	and	
L
	=	5,	necessitating	an
unrolling	factor	of	
k
	≥	10.	Floating-point	addition	has	
C
	=	1	and	
L
	=	3,
achieving	maximum	throughput	with	
k
	≥	3.
In	performing	the	
k
	×	
k
	unrolling	transformation,	we	must	consider
whether	it	preserves	the	functionality	of	the	original	function.	We	have
seen	in	
Chapter	
2
	that	two's-complement	arithmetic	is	commutative
and	associative,	even	when	overflow	occurs.	Hence,	for	an	integer	data
type,	the	result	computed	by	
	will	be	identical	to	that	computed
by	
	under	all	possible	conditions.	Thus,	an	optimizing	compiler
could	potentially	convert	the	code	shown	in	
	first	to	a	two-way
unrolled	variant	of	
	by	loop	unrolling,	and	then	to	that	of	
by	introducing	parallelism.	Some	compilers	do	either	this	or	similar
transformations	to	improve	performance	for	integer	data.
On	the	other	hand,	floating-point	multiplication	and	addition	are	not
associative.	Thus,	
	and	
	could	produce	different	results
due	to	rounding	or	overflow.	Imagine,	for	example,	a	product	computation
in	which	all	of	the	elements	with	even	indices	are	numbers	with	very	large
absolute	values,	while	those	with	odd	indices	are	very	close	to	0.0.	In
such	a	case,	product	
PE
	might	overflow,	or	
PO
	might	underflow,	even
though	computing	product	
P
	proceeds	
normally.	In	most	real-life
n
n
n

applications,	however,	such	patterns	are	unlikely.	Since	most	physical
phenomena	are	continuous,	numerical	data	tend	to	be	reasonably
smooth	and	well	behaved.	Even	when	there	are	discontinuities,	they	do
not	generally	cause	periodic	patterns	that	lead	to	a	condition	such	as	that
sketched	earlier.	It	is	unlikely	that	multiplying	the	elements	in	strict	order
gives	fundamentally	better	accuracy	than	does	multiplying	two	groups
independently	and	then	multiplying	those	products	together.	For	most
applications,	achieving	a	performance	gain	of	2×	outweighs	the	risk	of
generating	different	results	for	strange	data	patterns.	Nevertheless,	a
program	developer	should	check	with	potential	users	to	see	if	there	are
particular	conditions	that	may	cause	the	revised	algorithm	to	be
unacceptable.	Most	compilers	do	not	attempt	such	transformations	with
floating-point	code,	since	they	have	no	way	to	judge	the	risks	of
introducing	transformations	that	can	change	the	program	behavior,	no
matter	how	small.
5.9.2	
Reassociation	Transformation
We	now	explore	another	way	to	break	the	sequential	dependencies	and
thereby	improve	performance	beyond	the	latency	bound.	We	saw	that	the
k
	×	1	loop	unrolling	of	
	did	not	change	the	set	of	operations
performed	in	combining	the	vector	elements	to	form	their	sum	or	product.
By	a	very	small	change	in	the	code,	however,	we	can	fundamentally
change	the	way	the	combining	is	performed,	and	also	greatly	increase
the	program	performance.
Figure	
5.26
	shows	a	function	
	that	differs	from	the	unrolled
code	of	
	(
Figure	
5.16
)	only	in	the	way	the	elements	are

combined	in	the	inner	loop.	In	
,	the	combining	is	performed	by
the	statement
while	in	
	it	is	performed	by	the	statement
differing	only	in	how	two	parentheses	are	placed.	We	call	this	a
reassociation	transformation
,	because	the	parentheses	shift	the	order	in
which	the	vector	elements	are	combined	with	the	accumulated	value	acc,
yielding	a	form	of	loop	unrolling	we	refer	to	as	"2	×	1
a
."
To	an	untrained	eye,	the	two	statements	may	seem	essentially	the	same,
but	when	we	measure	the	CPE,	we	get	a	surprising	result:
Integer
Floating	point
Function
Page
Method
+
*
+
*
515
Accumulate	in	temporary
1.27
3.01
3.01
5.01
532
2	×	1	unrolling
1.01
3.01
3.01
5.01
537
2	×	2	unrolling
0.81
1.51
1.51
2.51

542
2	×	1
a
	unrolling
1.01
1.51
1.51
2.51
Latency	bound
1.00
3.00
3.00
5.00
Throughput	bound
0.50
1.00
1.00
0.50
Figure	
5.26	
Applying	2	×	1
a
	unrolling.
By	reassociating	the	arithmetic,	this	approach	increases	the	number	of
operations	that	can	be	performed	in	parallel.

The	integer	addition	case	matches	the	performance	of	
k
	×	1	unrolling
(
),	while	the	other	three	cases	match	the	performance	of	the
versions	with	parallel	accumulators	(
),	doubling	the	performance
relative	to	
k
	×	1	unrolling.	These	cases	have	broken	through	the	barrier
imposed	by	the	latency	bound.
Figure	
5.27
	illustrates	how	the	code	for	the	inner	loop	of	
	(for
the	case	of	multiplication	as	the	combining	operation	and	double	as	data
type)	gets	decoded	into	operations	and	the	resulting	data	dependencies.
We	see	that	the	load	operations	resulting	from	the	
	and	the	first
	instructions	load	vector	elements	
i
	and	
i
	+	1	from	memory,	and	the
first	mul	operation	multiplies	them	together.	The	second	mul	operation
then	multiples	this	result	by	the	accumulated	value	
.	
Figure	
5.28(a)
shows	how	we	rearrange,	refine,	and	abstract	the	operations	of	
Figure
5.27
	to	get	a	template	representing	the	data	dependencies	for	one
iteration	(
Figure	
5.28(b)
).	As	with	the	templates	for	
	and
,	we	have	two	load	and	two	mul	operations,	but	only	one	of	the
mul	operations	forms	a	data-dependency	chain	between	loop	registers.
When	we	then	replicate	this	template	
n
/2	times	to	show	the	computations
performed	in	multiplying	
n
	vector	elements	(
Figure	
5.29
),	we	see	that
we	only	have	
n
/2	operations	along	the	critical	path.	The	first	multiplication
within	each	iteration	can	be	performed	without	waiting	for	the
accumulated	value	from	the	previous	iteration.	Thus,	we	reduce	the
minimum	possible	CPE	by	a	factor	of	around	2.

Figure	
5.27	
Graphical	representation	of	inner-loop	code	for	
.
Each	iteration	gets	decoded	into	similar	operations	as	for	
	or
,	but	with	different	data	dependencies.
Figure	
5.28	
Abstracting	
	operations	as	a	data-flow	graph.
We	rearrange,	simplify,	and	abstract	the	representation	of	
Figure	
5.27
to	show	the	data	dependencies	between	successive	iterations.	The	upper
mul	operation	multiplies	two	2-vector	elements	with	each	other,	while	the
lower	one	multiplies	the	result	by	loop	variable	
.

Figure	
5.29	
Data-flow	representation	of	
	operating	on	a
vector	of	length	
n
.
We	have	a	single	critical	path,	but	it	contains	only	
n
/2	operations.
Figure	
5.30
	demonstrates	the	effect	of	applying	the	reassociation
transformation	to	achieve	what	we	refer	to	as	
k
	×	1
a	loop	unrolling
	for
values	up	to	
k
	=	10.	We	can	see	that	this	transformation	yields
performance	results	similar	to	what	is	achieved	by	maintaining	
k
	separate

accumulators	with	
k
	×	
k
	unrolling.	In	all	cases,	we	come	close	to	the
throughput	bounds	imposed	by	the	functional	units.
In	performing	the	reassociation	transformation,	we	once	again	change
the	order	in	which	the	vector	elements	will	be	combined	together.	For
integer	addition	and	multiplication,	the	fact	that	these	operations	are
associative	implies	that	this	reordering	will	have	no	effect	on	the	result.
For	the	floating-point	cases,	we	must	once	again	assess	whether	this
reassociation	is	likely	to	significantly	affect
Figure	
5.30	
CPE	performance	for	
k
	×	1
a
	loop	unrolling.
All	of	the	CPEs	improve	with	this	transformation,	nearly	approaching	their
throughput	bounds.
the	outcome.	We	would	argue	that	the	difference	would	be	immaterial	for
most	applications.
In	summary,	a	reassociation	transformation	can	reduce	the	number	of
operations	along	the	critical	path	in	a	computation,	resulting	in	better
performance	by	better	utilizing	the	multiple	functional	units	and	their
pipelining	capabilities.	Most	compilers	will	not	attempt	any	reassociations
of	floating-point	operations,	since	these	operations	are	not	guaranteed	to

be	associative.	Current	versions	of	
GCC
	
do	perform	reassociations	of
integer	operations,	but	not	always	with	good	effects.	In	general,	we	have
found	that	unrolling	a	loop	and	accumulating	multiple	values	in	parallel	is
a	more	reliable	way	to	achieve	improved	program	performance.
Practice	Problem	
5.8	
(solution	page	
576
)
Consider	the	following	function	for	computing	the	product	of	an
array	of	
n
	double-precision	numbers.	We	have	unrolled	the	loop	by
a	factor	of	3.
For	the	line	labeled	"Product	computation,"	we	can	use
parentheses	to	create	five	different	associations	of	the
computation,	as	follows:

Assume	we	run	these	functions	on	a	machine	where	floating-point
multiplication	has	a	latency	of	5	clock	cycles.	Determine	the	lower
bound	on	the	CPE	set	by	the	data	dependencies	of	the
multiplication.	(
Hint:
	It	helps	to	draw	a	data-flow	representation	of
how	
	is	computed	on	every	iteration.)
Web	Aside	OPT:SIMD	
Achieving
greater	parallelism	with	vector
instructions
As	described	in	
Section	
3.1
,	Intel	introduced	the	SSE
instructions	in	1999,	where	SSE	is	the	acronym	for	"streaming
SIMD	extensions"	and,	in	turn,	SIMD	(pronounced	"sim-dee")	is
the	acronym	for	"single	instruction,	multiple	data."	The	SSE
capability	has	gone	through	multiple	generations,	with	more	recent
versions	being	named	
advanced	vector	extensions
,	or	AVX.	The
SIMD	execution	model	involves	operating	on	entire	vectors	of	data
within	single	instructions.	These	vectors	are	held	in	a	special	set
of	
vector	registers
,	named	
.	Current	AVX	vector

registers	are	32	bytes	long,	and	therefore	each	can	hold	eight	32-
bit	numbers	or	four	64-bit	numbers,	where	the	numbers	can	be
either	integer	or	floating-point	values.	AVX	instructions	can	then
perform	vector	operations	on	these	registers,	such	as	adding	or
multiplying	eight	or	four	sets	of	values	in	parallel.	For	example,	if
YMM	register	
	contains	eight	single-precision	floating-point
numbers,	which	we	denote	
a
,	.	.	.,	
a
,	and	
	contains	the
memory	address	of	a	sequence	of	eight	single-precision	floating-
point	numbers,	which	we	denote	
b
,	.	.	.,	
b
,	then	the	instruction
will	read	the	eight	values	from	memory	and	perform	eight
multiplications	in	parallel,	computing	
a
	←	
a
	·	b
,	for	0	<	
i
	≤	7	and
storing	the	resulting	eight	products	in	vector	register	
.	We
see	that	a	single	instruction	is	able	to	generate	a	computation	over
multiple	data	values,	hence	the	term	"SIMD."
GCC
	
supports	extensions	to	the	C	language	that	let	programmers
express	a	program	in	terms	of	vector	operations	that	can	be
compiled	into	the	vector	instructions	of	AVX	(as	well	as	code
based	on	the	earlier	SSE	instructions).	This	coding	style	is
preferable	to	writing	code	directly	in	assembly	language,	since	
GCC
can	also	generate	code	for	the	vector	instructions	found	on	other
processors.
Using	a	combination	of	
GCC
	
instructions,	loop	unrolling,	and
multiple	accumulators,	we	are	able	to	achieve	the	following
0
7
0
7
i
i
i

performance	for	our	combining	functions:
Integer
Floating	point
int
long
int
long
Method
+
*
+
*
+
*
+
*
Scalar	10	×	10
0.54
1.01
0.55
1.00
1.01
0.51
1.01
0.52
Scalar	throughput
bound
0.50
0.50
1.00
1.00
1.00
1.00
0.50
0.50
Vector	8	×	8
0.05
0.24
0.13
1.51
0.12
0.08
0.25
0.16
Vector	throughput
bound
0.06
0.12
0.12
—
0.12
0.06
0.25
0.12
In	this	chart,	the	first	set	of	numbers	is	for	conventional,	
scalar
code	written	in	the	style	of	
,	unrolling	by	a	factor	of	10
and	maintaining	10	accumulators.	The	second	set	of	numbers	is
for	code	written	in	a	form	that	
GCC
	
can	compile	into	AVX	vector
code.	In	addition	to	using	vector	operations,	this	version	unrolls
the	main	loop	by	a	factor	of	8	and	maintains	eight	separate	vector
accumulators.	We	show	results	for	both	32-bit	and	64-bit
numbers,	since	the	vector	instructions	achieve	8-way	parallelism
in	the	first	case,	but	only	4-way	parallelism	in	the	second.
We	can	see	that	the	vector	code	achieves	almost	an	eightfold
improvement	on	the	four	32-bit	cases,	and	a	fourfold	improvement
on	three	of	the	four	64-bit	cases.	Only	the	long	integer
multiplication	code	does	not	perform	well	when	we	attempt	to
express	it	in	vector	code.	The	AVX	instruction	set	does	not	include

one	to	do	parallel	multiplication	of	64-bit	integers,	and	so	
GCC
cannot	generate	vector	code	for	this	case.	Using	vector
instructions	creates	a	new	throughput	bound	for	the	combining
operations.	These	are	eight	times	lower	for	32-bit	operations	and
four	times	lower	for	64-bit	operations	than	the	scalar	limits.	Our
code	comes	close	to	achieving	these	bounds	for	several
combinations	of	data	type	and	operation.

5.10	
Summary	of	Results	for
Optimizing	Combining	Code
Our	efforts	at	maximizing	the	performance	of	a	routine	that	adds	or
multiplies	the	elements	of	a	vector	have	clearly	paid	off.	The	following
summarizes	the	results	we	obtain	with	
scalar
	code,	not	making	use	of	the
vector	parallelism	provided	by	AVX	vector	instructions:
Integer
Floating	point
Function
Page
Method
+
*
+
*
507
Abstract	
10.12
10.12
10.17
11.14
537
2	×	2	unrolling
0.81
1.51
1.51
2.51
10	×	10	unrolling
0.55
1.00
1.01
0.52
Latency	bound
1.00
3.00
3.00
5.00
Throughput	bound
0.50
1.00
1.00
0.50
By	using	multiple	optimizations,	we	have	been	able	to	achieve	CPEs
close	to	the	throughput	bounds	of	0.50	and	1.00,	limited	only	by	the
capacities	of	the	functional	units.	These	represent	10−20×	improvements
on	the	original	code.	This	has	all	been	done	using	ordinary	C	code	and	a
standard	compiler.	Rewriting	the	code	to	take	advantage	of	the	newer
SIMD	instructions	yields	additional	performance	gains	of	nearly	4×	or	8×.
For	example,	for	single-precision	multiplication,	the	CPE	drops	from	the

original	value	of	11.14	down	to	0.06,	an	overall	performance	gain	of	over
180×.	This	example	demonstrates	that	modern	processors	have
considerable	amounts	of	computing	power,	but	we	may	need	to	coax	this
power	out	of	them	by	writing	our	programs	in	very	stylized	ways.

5.11	
Some	Limiting	Factors
We	have	seen	that	the	critical	path	in	a	data-flow	graph	representation	of
a	program	indicates	a	fundamental	lower	bound	on	the	time	required	to
execute	a	program.	That	is,	if	there	is	some	chain	of	data	dependencies
in	a	program	where	the	sum	of	all	of	the	latencies	along	that	chain	equals
T
,	then	the	program	will	require	at	least	
T
	cycles	to	execute.
We	have	also	seen	that	the	throughput	bounds	of	the	functional	units
also	impose	a	lower	bound	on	the	execution	time	for	a	program.	That	is,
assume	that	a	program	requires	a	total	of	
N
	computations	of	some
operation,	that	the	microprocessor	has	
C
	functional	units	capable	of
performing	that	operation,	and	that	these	units	have	an	issue	time	of	
I
.
Then	the	program	will	require	at	least	
N	·	I/C
	cycles	to	execute.
In	this	section,	we	will	consider	some	other	factors	that	limit	the
performance	of	programs	on	actual	machines.
5.11.1	
Register	Spilling
The	benefits	of	loop	parallelism	are	limited	by	the	ability	to	express	the
computation	in	assembly	code.	If	a	program	has	a	degree	of	parallelism
P
	that	exceeds	the	number	of	available	registers,	then	the	compiler	will
resort	to	
spilling
,	storing	some	of	the	temporary	values	in	memory,
typically	by	allocating	space	on	the	run-time	stack.	As	an	example,	the

following	measurements	compare	the	result	of	extending	the	multiple
accumulator	scheme	of	
	to	the	cases	of	
k
	=	10	and	
k
	=	20:
Integer
Floating	point
Function
Page
Method
+
*
+
*
537
10	×	10	unrolling
0.55
1.00
1.01
0.52
20	×	20	unrolling
0.83
1.03
1.02
0.68
Throughput	bound
0.50
1.00
1.00
0.50
We	can	see	that	none	of	the	CPEs	improve	with	this	increased	unrolling,
and	some	even	get	worse.	Modern	x86-64	processors	have	16	integer
registers	and	can	make	use	of	the	16	YMM	registers	to	store	floating-
point	data.	Once	the	number	of	loop	variables	exceeds	the	number	of
available	registers,	the	program	must	allocate	some	on	the	stack.
As	an	example,	the	following	snippet	of	code	shows	how	accumulator
	is	updated	in	the	inner	loop	of	the	code	with	10	×	10	unrolling:
We	can	see	that	the	accumulator	is	kept	in	register	
,	and	so	the
program	can	simply	read	
	from	memory	and	multiply	it	by	this

register.
The	comparable	part	of	the	code	for	20	×	20	unrolling	has	a	much
different	form:
The	accumulator	is	kept	as	a	local	variable	on	the	stack,	at	offset	40	from
the	stack	pointer.	The	program	must	read	both	its	value	and	the	value	of
	from	memory,	multiply	them,	and	store	the	result	back	to
memory.
Once	a	compiler	must	resort	to	register	spilling,	any	advantage	of
maintaining	multiple	accumulators	will	most	likely	be	lost.	Fortunately,
x86-64	has	enough	registers	that	most	loops	will	become	throughput
limited	before	this	occurs.
5.11.2	
Branch	Prediction	and
Misprediction	Penalties
We	demonstrated	via	experiments	in	
Section	
3.6.6
	that	a	conditional
branch	can	incur	a	significant	
misprediction	penalty
	when	the	branch

prediction	logic	does	not	correctly	anticipate	whether	or	not	a	branch	will
be	taken.	Now	that	we	have	learned	something	about	how	processors
operate,	we	can	understand	where	this	penalty	arises.
Modern	processors	work	well	ahead	of	the	currently	executing
instructions,	reading	new	instructions	from	memory	and	decoding	them	to
determine	what	operations	to	perform	on	what	operands.	This	
instruction
pipelining
	works	well	as	long	as	the	instructions	follow	in	a	simple
sequence.	When	a	branch	is	encountered,	the	processor	must	guess
which	way	the	branch	will	go.	For	the	case	of	a	conditional	jump,	this
means	predicting	whether	or	not	the	branch	will	be	taken.	For	an
instruction	such	as	an	indirect	jump	(as	we	saw	in	the	code	to	jump	to	an
address	specified	by	a	jump	table	entry)	or	a	procedure	return,	this
means	predicting	the	target	address.	In	this	discussion,	we	focus	on
conditional	branches.
In	a	processor	that	employs	
speculative	execution
,	the	processor	begins
executing	the	instructions	at	the	predicted	branch	target.	It	does	this	in	a
way	that	avoids	modifying	any	actual	register	or	memory	locations	until
the	actual	outcome	has	been	determined.	If	the	prediction	is	correct,	the
processor	can	then	
"commit"	the	results	of	the	speculatively	executed
instructions	by	storing	them	in	registers	or	memory.	If	the	prediction	is
incorrect,	the	processor	must	discard	all	of	the	speculatively	executed
results	and	restart	the	instruction	fetch	process	at	the	correct	location.
The	misprediction	penalty	is	incurred	in	doing	this,	because	the
instruction	pipeline	must	be	refilled	before	useful	results	are	generated.
We	saw	in	
Section	
3.6.6
	that	recent	versions	of	x86	processors,
including	all	processors	capable	of	executing	x86-64	programs,	have
conditional	move
	instructions.	
GCC
	
can	generate	code	that	uses	these

instructions	when	compiling	conditional	statements	and	expressions,
rather	than	the	more	traditional	realizations	based	on	conditional
transfers	of	control.	The	basic	idea	for	translating	into	conditional	moves
is	to	compute	the	values	along	both	branches	of	a	conditional	expression
or	statement	and	then	use	conditional	moves	to	select	the	desired	value.
We	saw	in	
Section	
4.5.7
	that	conditional	move	instructions	can	be
implemented	as	part	of	the	pipelined	processing	of	ordinary	instructions.
There	is	no	need	to	guess	whether	or	not	the	condition	will	hold,	and
hence	no	penalty	for	guessing	incorrectly.
How,	then,	can	a	C	programmer	make	sure	that	branch	misprediction
penalties	do	not	hamper	a	program's	efficiency?	Given	the	19-cycle
misprediction	penalty	we	measured	for	the	reference	machine,	the	stakes
are	very	high.	There	is	no	simple	answer	to	this	question,	but	the
following	general	principles	apply.
Do	Not	Be	Overly	Concerned	about
Predictable	Branches
We	have	seen	that	the	effect	of	a	mispredicted	branch	can	be	very	high,
but	that	does	not	mean	that	all	program	branches	will	slow	a	program
down.	In	fact,	the	branch	prediction	logic	found	in	modern	processors	is
very	good	at	discerning	regular	patterns	and	long-term	trends	for	the
different	branch	instructions.	For	example,	the	loop-closing	branches	in
our	combining	routines	would	typically	be	predicted	as	being	taken,	and
hence	would	only	incur	a	misprediction	penalty	on	the	last	time	around.
As	another	example,	consider	the	results	we	observed	when	shifting	from
	to	
,	when	we	took	the	function	
	out	of

the	inner	loop	of	the	function,	as	is	reproduced	below:
Integer
Floating	point
Function
Page
Method
+
*
+
*
509
Move	
7.02
9.03
9.02
11.03
513
Direct	data	access
7.17
9.02
9.02
11.03
The	CPE	did	not	improve,	even	though	the	transformation	eliminated	two
conditionals	on	each	iteration	that	check	whether	the	vector	index	is
within	bounds.	For	this	function,	the	checks	always	succeed,	and	hence
they	are	highly	predictable.
As	a	way	to	measure	the	performance	impact	of	bounds	checking,
consider	the	following	combining	code,	where	we	have	modified	the	inner
loop	of	
	by	replacing	the	access	to	the	data	element	with	the
result	of	performing	an	inline	substitution	of	the	code	for	
.
We	will	call	this	new	version	
.	This	code	performs	bounds
checking	and	also	references	the	vector	elements	through	the	vector
data	structure.

We	can	then	directly	compare	the	CPE	for	the	functions	with	and	without
bounds	checking:
Integer
Floating	point
Function
Page
Method
+
*
+
*
515
No	bounds	checking
1.27
3.01
3.01
5.01
515
Bounds	checking
2.02
3.01
3.01
5.01
The	version	with	bounds	checking	is	slightly	slower	for	the	case	of	integer
addition,	but	it	achieves	the	same	performance	for	the	other	three	cases.
The	performance	of	these	cases	is	limited	by	the	latencies	of	their
respective	combining	operations.	The	additional	computation	required	to
perform	bounds	checking	can	take	place	in	parallel	with	the	combining
operations.	The	processor	is	able	to	predict	the	outcomes	of	these
branches,	and	so	none	of	this	evaluation	has	much	effect	on	the	fetching
and	processing	of	the	instructions	that	form	the	critical	path	in	the
program	execution.

Write	Code	Suitable	for	Implementation	with
Conditional	Moves
Branch	prediction	is	only	reliable	for	regular	patterns.	Many	tests	in	a
program	are	completely	unpredictable,	dependent	on	arbitrary	features	of
the	data,	such	as	whether	a	number	is	negative	or	positive.	For	these,
the	branch	prediction	logic	will	do	very	poorly.	For	inherently
unpredictable	cases,	program	performance	can	be	greatly	enhanced	if
the	compiler	is	able	to	generate	code	using	conditional	data	transfers
rather	than	conditional	control	transfers.	This	cannot	be	controlled	directly
by	the	C	programmer,	but	some	ways	of	expressing	conditional	behavior
can	be	more	directly	translated	into	conditional	moves	than	others.
We	have	found	that	
GCC
	
is	able	to	generate	conditional	moves	for	code
written	in	a	more	"functional"	style,	where	we	use	conditional	operations
to	compute	
values	and	then	update	the	program	state	with	these	values,
as	opposed	to	a	more	"imperative"	style,	where	we	use	conditionals	to
selectively	update	program	state.
There	are	no	strict	rules	for	these	two	styles,	and	so	we	illustrate	with	an
example.	Suppose	we	are	given	two	arrays	of	integers	
a
	and	
b
,	and	at
each	position	
i
,	we	want	to	set	
	to	the	minimum	of	
	and	
,
and	
	to	the	maximum.
An	imperative	style	of	implementing	this	function	is	to	check	at	each
position	
i
	and	swap	the	two	elements	if	they	are	out	of	order:

Our	measurements	for	this	function	show	a	CPE	of	around	13.5	for
random	data	and	2.5-3.5	for	predictable	data,	an	indication	of	a
misprediction	penalty	of	around	20	cycles.
A	functional	style	of	implementing	this	function	is	to	compute	the
minimum	and	maximum	values	at	each	position	
i
	and	then	assign	these
values	to	
	and	
,	respectively:

Our	measurements	for	this	function	show	a	CPE	of	around	4.0	regardless
of	whether	the	data	are	arbitrary	or	predictable.	(We	also	examined	the
generated	assembly	code	to	make	sure	that	it	indeed	uses	conditional
moves.)
As	discussed	in	
Section	
3.6.6
,	not	all	conditional	behavior	can	be
implemented	with	conditional	data	transfers,	and	so	there	are	inevitably
cases	where	programmers	cannot	avoid	writing	code	that	will	lead	to
conditional	branches	for	which	the	processor	will	do	poorly	with	its	branch
prediction.	But,	as	we	have	shown,	a	little	cleverness	on	the	part	of	the
programmer	can	sometimes	make	code	more	amenable	to	translation
into	conditional	data	transfers.	This	requires	some	amount	
of
experimentation,	writing	different	versions	of	the	function	and	then
examining	the	generated	assembly	code	and	measuring	performance.
Practice	Problem	
5.9	
(solution	page	
576
)
The	traditional	implementation	of	the	merge	step	of	mergesort
requires	three	loops	[
98
]:

The	branches	caused	by	comparing	variables	
	and	
	to	n	have
good	prediction	performance—the	only	mispredictions	occur	when
they	first	become	false.	The	comparison	between	values	
and	
	(line	6),	on	the	other	hand,	is	highly	unpredictable	for
typical	data.	This	comparison	controls	a	conditional	branch,
yielding	a	CPE	(where	the	number	of	elements	is	2
n
)	of	around
15.0	when	run	on	random	data.
Rewrite	the	code	so	that	the	effect	of	the	conditional	statement	in
the	first	loop	(lines	6-9)	can	be	implemented	with	a	conditional
move.

5.12	
Understanding	Memory
Performance
All	of	the	code	we	have	written	thus	far,	and	all	the	tests	we	have	run,
access	relatively	small	amounts	of	memory.	For	example,	the	combining
routines	were	measured	over	vectors	of	length	less	than	1,000	elements,
requiring	no	more	than	8,000	bytes	of	data.	All	modern	processors
contain	one	or	more	
cache
	memories	to	provide	fast	access	to	such
small	amounts	of	memory.	In	this	section,	we	will	further	investigate	the
performance	of	programs	that	involve	load	(reading	from	memory	into
registers)	and	store	(writing	from	registers	to	memory)	operations,
considering	only	the	cases	where	all	data	are	held	in	cache.	In	
Chapter
6
,	we	go	into	much	more	detail	about	how	caches	work,	their
performance	characteristics,	and	how	to	write	code	that	makes	best	use
of	caches.
As	
Figure	
5.11
	shows,	modern	processors	have	dedicated	functional
units	to	perform	load	and	store	operations,	and	these	units	have	internal
buffers	to	hold	sets	of	outstanding	requests	for	memory	operations.	For
example,	our	reference	machine	has	two	load	units,	each	of	which	can
holdup	to	72	pending	read	requests.	It	has	a	single	store	unit	with	a	store
buffer	containing	up	to	42	write	requests.	Each	of	these	units	can	initiate
1	operation	every	clock	cycle.
5.12.1	
Load	Performance

The	performance	of	a	program	containing	load	operations	depends	on
both	the	pipelining	capability	and	the	latency	of	the	load	unit.	In	our
experiments	with	combining	operations	using	our	reference	machine,	we
saw	that	the	CPE	never	got	below	0.50	for	any	combination	of	data	type
and	combining	operation,	except	when	using	SIMD	operations.	One
factor	limiting	the	CPE	for	our	examples	is	that	they	all	require	reading
one	value	from	memory	for	each	element	computed.	With	two	load	units,
each	able	to	initiate	at	most	1	load	operation	every	clock	cycle,	the	CPE
cannot	be	less	than	0.50.	For	applications	where	we	must	load	
k
	values
for	every	element	computed,	we	can	never	achieve	a	CPE	lower	than	
k
/2
(see,	for	example,	
Problem	
5.15
).
In	our	examples	so	far,	we	have	not	seen	any	performance	effects	due	to
the	latency	of	load	operations.	The	addresses	for	our	load	operations
depended	only	on	the	loop	index	
i
,	and	so	the	load	operations	did	not
form	part	of	a	performance-limiting	critical	path.
To	determine	the	latency	of	the	load	operation	on	a	machine,	we	can	set
up	a	computation	with	a	sequence	of	load	operations,	where	the	outcome
of	one	determines	the	address	for	the	next.	As	an	example,	consider	the
function	
	in	
Figure	
5.31
,	which	computes	the	length	of	a
linked	list.	In	the	loop	of	this	function,	each	successive	value	of	variable
	depends	on	the	value	read	by	the	pointer	reference	
.	Our
measurements	show	that	function	
	has

Figure	
5.31	
Linked	list	function.
Its	performance	is	limited	by	the	latency	of	the	load	operation.
a	CPE	of	4.00,	which	we	claim	is	a	direct	indication	of	the	latency	of	the
load	operation.	To	see	this,	consider	the	assembly	code	for	the	loop:
The	
	instruction	on	line	3	forms	the	critical	bottleneck	in	this	loop.
Each	successive	value	of	register	
	depends	on	the	result	of	a	load

operation	having	the	value	in	
	as	its	address.	Thus,	the	load
operation	for	one	iteration	cannot	begin	until	the	one	for	the	previous
iteration	has	completed.	The	CPE	of	4.00	for	this	function	is	determined
by	the	latency	of	the	load	operation.	Indeed,	this	measurement	matches
the	documented	access	time	of	4	cycles	for	the	reference	machine's	L1
cache,	as	is	discussed	in	
Section	
6.4
.
5.12.2	
Store	Performance
In	all	of	our	examples	thus	far,	we	analyzed	only	functions	that	reference
memory	mostly	with	load	operations,	reading	from	a	memory	location	into
a	register.	Its	counterpart,	the	
store
	operation,	writes	a	register	value	to
memory.	The	performance	of	this	operation,	particularly	in	relation	to	its
interactions	with	load	operations,	involves	several	subtle	issues.
As	with	the	load	operation,	in	most	cases,	the	store	operation	can
operate	in	a	fully	pipelined	mode,	beginning	a	new	store	on	every	cycle.
For	example,	consider	the	function	shown	in	
Figure	
5.32
	that	sets	the
elements	of	an	array	
	of	length	
	to	zero.	Our	measurements	show
a	CPE	of	1.0.	This	is	the	best	we	can	achieve	on	a	machine	with	a	single
store	functional	unit.
Unlike	the	other	operations	we	have	considered	so	far,	the	store
operation	does	not	affect	any	register	values.	Thus,	by	their	very	nature,
a	series	of	store	operations	cannot	create	a	data	dependency.	Only	a
load	operation	is	affected	by	the	result	of	a	store	operation,	since	only	a
load	can	read	back	the	memory	value	that	has	been	written	by	the	store.
The	function	
	shown	in	
Figure	
5.33

Figure	
5.32	
Function	to	set	array	elements	to	0.
This	code	achieves	a	CPE	of	1.0.

Figure	
5.33	
Code	to	write	and	read	memory	locations,	along	with
illustrative	executions.
This	function	highlights	the	interactions	between	stores	and	loads	when
arguments	
	and	
	are	equal.
illustrates	the	potential	interactions	between	loads	and	stores.	This	figure
also	shows	two	example	executions	of	this	function,	when	it	is	called	for	a
two-element	array	a,	with	initial	contents	−10	and	17,	and	with	argument
	equal	to	3.	These	executions	illustrate	some	subtleties	of	the	load
and	store	operations.
In	Example	A	of	
Figure	
5.33
,	argument	
	is	a	pointer	to	array
element	
,	while	
	is	a	pointer	to	array	element	
.	In	this	case,
each	load	by	the	pointer	reference	
	will	yield	the	value	−10.	Hence,
after	two	iterations,	the	array	elements	will	remain	fixed	at	−10	and	−9,
respectively.	The	result	of	the	read	from	
	is	not	affected	by	the	write	to
.	Measuring	this	example	over	a	larger	number	of	iterations	gives	a
CPE	of	1.3.

In	Example	B	of	
Figure	
5.33
,	both	arguments	
	and	
	are
pointers	to	array	element	
.	In	this	case,	each	load	by	the	pointer
reference	
	will	yield	the	value	stored	by	the	previous	execution	of	the
pointer	reference	
.
Figure	
5.34	
Detail	of	load	and	store	units.
The	store	unit	maintains	a	buffer	of	pending	writes.	The	load	unit	must
check	its	address	with	those	in	the	store	unit	to	detect	a	write/read
dependency.
As	a	consequence,	a	series	of	ascending	values	will	be	stored	in	this
location.	In	general,	if	function	
	is	called	with	arguments	
and	
	pointing	to	the	same	memory	location,	and	with	argument	
having	some	value	
n
	>	0,	the	net	effect	is	to	set	the	location	to	
n
	−	1.	This
example	illustrates	a	phenomenon	we	will	call	a	
write/read	dependency
—
the	outcome	of	a	memory	read	depends	on	a	recent	memory	write.	Our
performance	measurements	show	that	Example	B	has	a	CPE	of	7.3.	The
write/read	dependency	causes	a	slowdown	in	the	processing	of	around	6
clock	cycles.

To	see	how	the	processor	can	distinguish	between	these	two	cases	and
why	one	runs	slower	than	the	other,	we	must	take	a	more	detailed	look	at
the	load	and	store	execution	units,	as	shown	in	
Figure	
5.34
.	The	store
unit	includes	a	
store	buffer
	containing	the	addresses	and	data	of	the
store	operations	that	have	been	issued	to	the	store	unit,	but	have	not	yet
been	completed,	where	completion	involves	updating	the	data	cache.
This	buffer	is	provided	so	that	a	series	of	store	operations	can	be
executed	without	having	to	wait	for	each	one	to	update	the	cache.	When
a	load	operation	occurs,	it	must	check	the	entries	in	the	store	buffer	for
matching	addresses.	If	it	finds	a	match	(meaning	that	any	of	the	bytes
being	written	have	the	same	address	as	any	of	the	bytes	being	read),	it
retrieves	the	corresponding	data	entry	as	the	result	of	the	load	operation.
GCC
	
generates	the	following	code	for	the	inner	loop	of	
:

Figure	
5.35	
Graphical	representation	of	inner-loop	code	for
.
The	first	
	instruction	is	decoded	into	separate	operations	to	compute
the	store	address	and	to	store	the	data	to	memory.
Figure	
5.35
	shows	a	data-flow	representation	of	this	loop	code.	The
instruction	
	is	translated	into	two	operations:	The	s_addr
instruction	computes	the	address	for	the	store	operation,	creates	an	entry
in	the	store	buffer,	and	sets	the	address	field	for	that	entry.	The	s_data
operation	sets	the	data	field	for	the	entry.	As	we	will	see,	the	fact	that
these	two	computations	are	performed	independently	can	be	important	to
program	performance.	This	motivates	the	separate	functional	units	for
these	operations	in	the	reference	machine.
In	addition	to	the	data	dependencies	between	the	operations	caused	by
the	writing	and	reading	of	registers,	the	arcs	on	the	right	of	the	operators
denote	a	set	of	implicit	dependencies	for	these	operations.	In	particular,
the	address	computation	of	the	s_addr	operation	must	clearly	precede
the	s_data	operation.	In	addition,	the	load	operation	generated	by
decoding	the	instruction	
	
	must	check	the	addresses	of
any	pending	store	operations,	creating	a	data	dependency	between	it

and	the	s_addr	operation.	The	figure	shows	a	dashed	arc	between	the
s_data	and	load	operations.	This	dependency	is	conditional:	if	the	two
addresses	match,	the	load	operation	must	wait	until	the	s_data	has
deposited	its	result	into	the	store	buffer,	but	if	the	two	addresses	differ,
the	two	operations	can	proceed	independently.
Figure	
5.36
	illustrates	the	data	dependencies	between	the	operations
for	the	inner	loop	of	
.	In	
Figure	
5.36(a)
,	we	have	rearranged
the	operations	to	allow	the	dependencies	to	be	seen	more	clearly.	We
have	labeled	the	three	dependencies	involving	the	load	and	store
operations	for	special	attention.	The	arc	labeled	"1"	represents	the
requirement	that	the	store	address	must	be	computed	before	the	data
can	be	stored.	The	arc	labeled	"2"	represents	the	need	for	the	load
operation	to	compare	its	address	with	that	for	any	pending	store
operations.	Finally,	the	dashed	arc	labeled	"3"	represents	the	conditional
data	dependency	that	arises	when	the	load	and	store	addresses	match.
Figure	
5.36(b)
	illustrates	what	happens	when	we	take	away	those
operations	that	do	not	directly	affect	the	flow	of	data	from	one	iteration	to
the	next.	The	data-flow	graph	shows	just	two	chains	of	dependencies:	the
one	on	the	left,	with	data	values	being	stored,	loaded,	and	incremented
(only	for	the	case	of	matching	addresses);	and	the	one	on	the	right,
decrementing	variable	
.

Figure	
5.36	
Abstracting	the	operations	for	
.
We	first	rearrange	the	operators	of	
Figure	
5.35(a)
	and	then	show	only
those	operations	that	use	values	from	one	iteration	to	produce	new
values	for	the	next	(b).
We	can	now	understand	the	performance	characteristics	of	function
.	
Figure	
5.37
	illustrates	the	data	dependencies	formed	by
multiple	iterations	of	its	inner	loop.	For	the	case	of	Example	A	in	
Figure
5.33
,	with	differing	source	and	destination	addresses,	the	load	and
store	operations	can	proceed	independently,	and	hence	the	only	critical
path	is	formed	by	the	decrementing	of	variable	
,	resulting	in	a	CPE
bound	of	1.0.	For	the	case	of	Example	B	with	matching	source	and
destination	addresses,	the	data	dependency	between	the	s_data	and
load	instructions	causes	a	critical	path	to	form	involving	data	being
stored,	loaded,	and	incremented.	We	found	that	these	three	operations	in
sequence	require	a	total	of	around	7	clock	cycles.
As	these	two	examples	show,	the	implementation	of	memory	operations
involves	many	subtleties.	With	operations	on	registers,	the	processor	can
determine	which	instructions	will	affect	which	others	as	they	are	being
decoded	into	operations.	With	memory	operations,	on	the	other	hand,	the

processor	cannot	predict	which	will	affect	which	others	until	the	load	and
store	addresses	have	been	computed.	Efficient	handling	of	memory
operations	is	critical	to	the	performance	of	many	programs.	The	memory
subsystem	makes	use	of	many	optimizations,	such	as	the	potential
parallelism	when	operations	can	proceed	independently.
Practice	Problem	
5.10	
(solution	page	
577
)
As	another	example	of	code	with	potential	load-store	interactions,
consider	the	following	function	to	copy	the	contents	of	one	array	to
another:

Figure	
5.37	
Data-flow	representation	of	function	
.
When	the	two	addresses	do	not	match,	the	only	critical	path	is
formed	by	the	decrementing	of	
cnt
	(Example	A).	When	they	do
match,	the	chain	of	data	being	stored,	loaded,	and	incremented
forms	the	critical	path	(Example	B).
Suppose	
	is	an	array	of	length	1,000	initialized	so	that	each
element	
	equals	
i.
A
.	
What	would	be	the	effect	of	the	call	
B
.	
What	would	be	the	effect	of	the	call	

C
.	
Our	performance	measurements	indicate	that	the	call	of
part	A	has	a	CPE	of	1.2	(which	drops	to	1.0	when	the	loop
is	unrolled	by	a	factor	of	4),	while	the	call	of	part	B	has	a
CPE	of	5.0.	To	what	factor	do	you	attribute	this	performance
difference?
D
.	
What	performance	would	you	expect	for	the	call	
Practice	Problem	
5.11	
(solution	page	
577
)
We	saw	that	our	measurements	of	the	prefix-sum	function	
(
Figure	
5.1
)	yield	a	CPE	of	9.00	on	a	machine	where	the	basic
operation	to	be	performed,	floating-point	addition,	has	a	latency	of
just	3	clock	cycles.	Let	us	try	to	understand	why	our	function
performs	so	poorly.
The	following	is	the	assembly	code	for	the	inner	loop	of	the
function:

Perform	an	analysis	similar	to	those	shown	for	
	(
Figure
5.14
)	and	for	
	(
Figure	
5.36
)	to	diagram	the	data
dependencies	created	by	this	loop,	and	hence	the	critical	path	that
forms	as	the	computation	proceeds.	Explain	why	the	CPE	is	so
high.
Practice	Problem	
5.12	
(solution	page	
577
)
Rewrite	the	code	for	
	(
Figure	
5.1
)	so	that	it	does	not	need
to	repeatedly	retrieve	the	value	of	
	from	memory.	You	do	not
need	to	use	loop	unrolling.	We	measured	the	resulting	code	to
have	a	CPE	of	3.00,	limited	by	the	latency	of	floating-point
addition.

5.13	
Life	in	the	Real	World:
Performance	Improvement
Techniques
Although	we	have	only	considered	a	limited	set	of	applications,	we	can
draw	important	lessons	on	how	to	write	efficient	code.	We	have
described	a	number	of	basic	strategies	for	optimizing	program
performance:
High-level	design.	
Choose	appropriate	algorithms	and	data
structures	for	the	problem	at	hand.	Be	especially	vigilant	to	avoid
algorithms	or	coding	techniques	that	yield	asymptotically	poor
performance.
Basic	coding	principles.	
Avoid	optimization	blockers	so	that	a
compiler	can	generate	efficient	code.
Eliminate	excessive	function	calls.	Move	computations	out	of	loops
when	possible.	Consider	selective	compromises	of	program
modularity	to	gain	greater	efficiency.
Eliminate	unnecessary	memory	references.	Introduce	temporary
variables	to	hold	intermediate	results.	Store	a	result	in	an	array	or
global	variable	only	when	the	final	value	has	been	computed.
Low-level	optimizations.	
Structure	code	to	take	advantage	of	the
hardware	capabilities.

Unroll	loops	to	reduce	overhead	and	to	enable	further
optimizations.
Find	ways	to	increase	instruction-level	parallelism	by	techniques
such	as	multiple	accumulators	and	reassociation.
Rewrite	conditional	operations	in	a	functional	style	to	enable
compilation	via	conditional	data	transfers.
A	final	word	of	advice	to	the	reader	is	to	be	vigilant	to	avoid	introducing
errors	as	you	rewrite	programs	in	the	interest	of	efficiency.	It	is	very	easy
to	make	mistakes	when	introducing	new	variables,	changing	loop
bounds,	and	making	the	code	more	complex	overall.	One	useful
technique	is	to	use	checking	code	to	test	each	version	of	a	function	as	it
is	being	optimized,	to	ensure	no	bugs	are	introduced	during	this	process.
Checking	code	applies	a	series	of	tests	to	the	new	versions	of	a	function
and	makes	sure	they	yield	the	same	results	as	the	original.	The	set	of
test	cases	must	become	more	extensive	with	highly	optimized	code,
since	there	are	more	cases	to	consider.	For	example,	checking	code	that
uses	loop	unrolling	requires	testing	for	many	different	loop	bounds	to
make	sure	it	handles	all	of	the	different	possible	numbers	of	single-step
iterations	required	at	the	end.

5.14	
Identifying	and	Eliminating
Performance	Bottlenecks
Up	to	this	point,	we	have	only	considered	optimizing	small	programs,
where	there	is	some	clear	place	in	the	program	that	limits	its	performance
and	therefore	should	be	the	focus	of	our	optimization	efforts.	When
working	with	large	programs,	even	knowing	where	to	focus	our
optimization	efforts	can	be	difficult.	In	this	section,	we	describe	how	to
use	
code	profilers
,	analysis	tools	that	collect	performance	data	about	a
program	as	it	executes.	We	also	discuss	some	general	principles	of	code
optimization,	including	the	implications	of	Amdahl's	law,	introduced	in
Section	
1.9.1
.
5.14.1	
Program	Profiling
Program	
profiling
	involves	running	a	version	of	a	program	in	which
instrumentation	code	has	been	incorporated	to	determine	how	much	time
the	different	parts	of	the	program	require.	It	can	be	very	useful	for
identifying	the	parts	of	a	program	we	should	focus	on	in	our	optimization
efforts.	One	strength	of	profiling	is	that	it	can	be	performed	while	running
the	actual	program	on	realistic	benchmark	data.
Unix	systems	provide	the	profiling	program	
GPROF
.	This	program
generates	two	forms	of	information.	First,	it	determines	how	much	CPU

time	was	spent	for	each	of	the	functions	in	the	program.	Second,	it
computes	a	count	of	how	many	times	each	function	gets	called,
categorized	by	which	function	performs	the	call.	Both	forms	of	information
can	be	quite	useful.	The	timings	give	a	sense	of	
the	relative	importance
of	the	different	functions	in	determining	the	overall	run	time.	The	calling
information	allows	us	to	understand	the	dynamic	behavior	of	the
program.
Profiling	with	
GPROF
	
requires	three	steps,	as	shown	for	a	C	program
,	which	runs	with	command-line	argument	
:
1
.	
The	program	must	be	compiled	and	linked	for	profiling.	With	
GCC
(and	other	C	compilers),	this	involves	simply	including	the	run-time
flag	
	on	the	command	line.	It	is	important	to	ensure	that	the
compiler	does	not	attempt	to	perform	any	optimizations	via	inline
substitution,	or	else	the	calls	to	functions	may	not	be	tabulated
accurately.	We	use	optimization	flag	
,	guaranteeing	that
function	calls	will	be	tracked	properly.
2
.	
The	program	is	then	executed	as	usual:
It	runs	slightly	(around	a	factor	of	2)	slower	than	normal,	but
otherwise	the	only	difference	is	that	it	generates	a	file	
.
3
.	
GPROF
	
is	invoked	to	analyze	the	data	in	
:

The	first	part	of	the	profile	report	lists	the	times	spent	executing	the
different	functions,	sorted	in	descending	order.	As	an	example,	the
following	listing	shows	this	part	of	the	report	for	the	three	most	time-
consuming	functions	in	a	program:
Each	row	represents	the	time	spent	for	all	calls	to	some	function.	The	first
column	indicates	the	percentage	of	the	overall	time	spent	on	the	function.
The	second	shows	the	cumulative	time	spent	by	the	functions	up	to	and
including	the	one	on	this	row.	The	third	shows	the	time	spent	on	this
particular	function,	and	the	fourth	shows	how	many	times	it	was	called
(not	counting	recursive	calls).	In	our	example,	the	function	
was	called	only	once,	but	this	single	call	required	203.66	seconds,	while
the	function	
	was	called	965,027	times	(not	including
recursive	calls),	requiring	a	total	of	4.85	seconds.	Function	
computes	the	length	of	a	string	by	calling	the	library	function	
.
Library	function	calls	are	normally	not	shown	in	the	results	by	
GPROF
.
Their	times	are	usually	reported	as	part	of	the	function	calling	them.	By

creating	the	"wrapper	function"	
,	we	can	reliably	track	the	calls	to
,	showing	that	it	was	called	12,511,031	times	but	only	requiring	a
total	of	0.30	seconds.
The	second	part	of	the	profile	report	shows	the	calling	history	of	the
functions.	The	following	is	the	history	for	a	recursive	function
:
This	history	shows	both	the	functions	that	called	
,	as	well	as
the	functions	that	it	called.	The	first	two	lines	show	the	calls	to	the
function:	158,655,725	calls	by	itself	recursively,	and	965,027	calls	by
function	
	(which	is	itself	called	965,027	times).	Function
,	in	turn,	called	two	other	functions,	
	and
,	each	a	total	of	363,039	times.
From	these	call	data,	we	can	often	infer	useful	information	about	the
program	behavior.	For	example,	the	function	
	is	a	recursive
procedure	that	scans	the	linked	list	for	a	hash	bucket	looking	for	a

particular	string.	For	this	function,	comparing	the	number	of	recursive
calls	with	the	number	of	top-level	calls	provides	statistical	information
about	the	lengths	of	the	traversals	through	these	lists.	Given	that	their
ratio	is	164.4:1,	we	can	infer	that	the	program	scanned	an	average	of
around	164	elements	each	time.
Some	properties	of	
GPROF
	
are	worth	noting:
The	timing	is	not	very	precise.	It	is	based	on	a	simple	
interval
counting
	scheme	in	which	the	compiled	program	maintains	a	counter
for	each	function	recording	the	time	spent	executing	that	function.	The
operating	system	causes	the	program	to	be	interrupted	at	some
regular	time	interval	
δ
.	Typical	values	of	
δ
	range	between	1.0	and
10.0	milliseconds.	It	then	determines	what	function	the	program	was
executing	when	the	interrupt	occurred	and	increments	the	counter	for
that	function	by	
δ
.	Of	course,	it	may	happen	that	this	function	just
started	executing	and	will	shortly	be	completed,	but	it	is	assigned	the
full	cost	of	the	execution	since	the	previous	interrupt.	Some	other
function	may	run	between	two	interrupts	and	therefore	not	be	charged
any	time	at	all.
Over	a	long	duration,	this	scheme	works	reasonably	well.	Statistically,
every	function	should	be	charged	according	to	the	relative	time	spent
executing	it.	For	programs	that	run	for	less	than	around	1	second,
however,	the	numbers	should	be	viewed	as	only	rough	estimates.
The	calling	information	is	quite	reliable,	assuming	no	inline
substitutions	have	been	performed.	The	compiled	program	maintains
a	counter	for	each	combination	of	caller	and	callee.	The	appropriate
counter	is	incremented	every	time	a	procedure	is	called.

By	default,	the	timings	for	library	functions	are	not	shown.	Instead,
these	times	are	incorporated	into	the	times	for	the	calling	functions.
5.14.2	
Using	a	Profiler	to	Guide
Optimization
As	an	example	of	using	a	profiler	to	guide	program	optimization,	we
created	an	application	that	involves	several	different	tasks	and	data
structures.	This	application	analyzes	the	
n-gram
	statistics	of	a	text
document,	where	an	
n
-gram	is	a	sequence	of	
n
	words	occurring	in	a
document.	For	
n
	=	1,	we	collect	statistics	on	individual	words,	for	
n
	=	2	on
pairs	of	words,	and	so	on.	For	a	given	value	of	
n
,	our	program	reads	a
text	file,	creates	a	table	of	unique	
n
-grams	and	how	many	times	each	one
occurs,	then	sorts	the	
n
-grams	in	descending	order	of	occurrence.
As	a	benchmark,	we	ran	it	on	a	file	consisting	of	the	complete	works	of
William	Shakespeare,	totaling	965,028	words,	of	which	23,706	are
unique.	We	found	that	for	
n
	=	1,	even	a	poorly	written	analysis	program
can	readily	process	the	entire	file	in	under	1	second,	and	so	we	set	
n
	=	2
to	make	things	more	challenging.	For	the	case	of	
n
	=	2,	
n
-grams	are
referred	to	as	
bigrams
	(pronounced	"bye-grams").	We	determined	that
Shakespeare's	works	contain	363,039	unique	bigrams.	The	most
common	is	"I	am,"	occurring	1,892	times.	Perhaps	his	most	famous
bigram,	"to	be,"	occurs	1,020	times.	Fully	266,018	of	the	bigrams	occur
only	once.

Our	program	consists	of	the	following	parts.	We	created	multiple
versions,	starting	with	simple	algorithms	for	the	different	parts	and	then
replacing	them	with	more	sophisticated	ones:
1
.	
Each	word	is	read	from	the	file	and	converted	to	lowercase.	Our
initial	version	used	the	function	
	(
Figure	
5.7
),	which	we
know	to	have	quadratic	run	time	due	to	repeated	calls	to	
.
2
.	
A	hash	function	is	applied	to	the	string	to	create	a	number
between	0	and	
s
	−	1,	for	a	hash	table	with	
s
	buckets.	Our	initial
function	simply	summed	the	ASCII	codes	for	the	characters
modulo	
s.
3
.	
Each	hash	bucket	is	organized	as	a	linked	list.	The	program	scans
down	this	list	looking	for	a	matching	entry.	If	one	is	found,	the
frequency	for	this	
n
-gram	is	incremented.	Otherwise,	a	new	list
element	is	created.	Our	initial	version	performed	this	operation
recursively,	inserting	new	elements	at	the	end	of	the	list.
4
.	
Once	the	table	has	been	generated,	we	sort	all	of	the	elements
according	to	the	frequencies.	Our	initial	version	used	insertion
sort.
Figure	
5.38
	shows	the	profile	results	for	six	different	versions	of	our	
n
-
gram-frequency	analysis	program.	For	each	version,	we	divide	the	time
into	the	following	categories:
Sort.	
Sorting	
n
-grams	by	frequency
List.	
Scanning	the	linked	list	for	a	matching	
n
-gram,	inserting	a	new
element	if	necessary
Lower.	
Converting	strings	to	lowercase

Strlen.	
Computing	string	lengths
Figure	
5.38	
Profile	results	for	different	versions	of	bigram-
frequency	counting	program.
Time	is	divided	according	to	the	different	major	operations	in	the
program.
Hash.	
Computing	the	hash	function
Rest.	
The	sum	of	all	other	functions
As	part	(a)	of	the	figure	shows,	our	initial	version	required	3.5	minutes,
with	most	of	the	time	spent	sorting.	This	is	not	surprising,	since	insertion
sort	has	quadratic	run	time	and	the	program	sorted	363,039	values.

In	our	next	version,	we	performed	sorting	using	the	library	function	
,
which	is	based	on	the	quicksort	algorithm	[
98
].	It	has	an	expected	run
time	of	
O
(
n
	log	
n
).	This	version	is	labeled	"Quicksort"	in	the	figure.	The
more	efficient	sorting	algorithm	reduces	the	time	spent	sorting	to	become
negligible,	and	the	overall	run	time	to	around	5.4	seconds.	Part	(b)	of	the
figure	shows	the	times	for	the	remaining	version	on	a	scale	where	we	can
see	them	more	clearly.
With	improved	sorting,	we	now	find	that	list	scanning	becomes	the
bottleneck.	Thinking	that	the	inefficiency	is	due	to	the	recursive	structure
of	the	function,	we	replaced	it	by	an	iterative	one,	shown	as	"Iter	first."
Surprisingly,	the	run	time	increases	to	around	7.5	seconds.	On	closer
study,	we	find	a	subtle	difference	between	the	two	list	functions.	The
recursive	version	inserted	new	elements	at	the	end	of	the	list,	while	the
iterative	one	inserted	them	at	the	front.	To	maximize	performance,	we
want	the	most	frequent	
n
-grams	to	occur	near	the	beginning	of	the	lists.
That	way,	the	function	will	quickly	locate	the	common	cases.	Assuming
that	
n
-grams	are	spread	uniformly	throughout	the	document,	we	would
expect	the	first	occurrence	of	a	frequent	one	to	come	before	that	of	a	less
frequent	one.	By	inserting	new	
n
-grams	at	the	end,	the	first	function
tended	to	order	
n
-grams	in	descending	order	of	frequency,	while	the
second	function	tended	to	do	just	the	opposite.	We	therefore	created	a
third	list-scanning	function	that	uses	iteration	but	inserts	new	elements	at
the	end	of	this	list.	With	this	version,	shown	as	"Iter	last,"	the	time
dropped	to	around	5.3	seconds,	slightly	better	than	with	the	recursive
version.	These	measurements	demonstrate	the	importance	of	running
experiments	on	a	program	as	part	of	an	optimization	effort.	We	initially
assumed	that	converting	recursive	code	to	iterative	code	would	improve

its	performance	and	did	not	consider	the	distinction	between	adding	to
the	end	or	to	the	beginning	of	a	list.
Next,	we	consider	the	hash	table	structure.	The	initial	version	had	only
1,021	buckets	(typically,	the	number	of	buckets	is	chosen	to	be	a	prime
number	to	enhance	the	ability	of	the	hash	function	to	distribute	keys
uniformly	among	the	buckets).	For	a	table	with	363,039	entries,	this
would	imply	an	average	
load
	of	363,039/1,021	=	355.6.	That	explains
why	so	much	of	the	time	is	spent	performing	list	operations—the
searches	involve	testing	a	significant	number	of	candidate	
n
-grams.	It
also	explains	why	the	performance	is	so	sensitive	to	the	list	ordering.	We
then	increased	the	number	of	buckets	to	199,999,	reducing	the	average
load	to	1.8.	Oddly	enough,	however,	our	overall	run	time	only	drops	to	5.1
seconds,	a	difference	of	only	0.2	seconds.
On	further	inspection,	we	can	see	that	the	minimal	performance	gain	with
a	larger	table	was	due	to	a	poor	choice	of	hash	function.	Simply	summing
the	character	codes	for	a	string	does	not	produce	a	very	wide	range	of
values.	In	particular,	the	maximum	code	value	for	a	letter	is	122,	and	so	a
string	of	
n
	characters	will	generate	a	sum	of	at	most	122
n
.	The	longest
bigram	in	our	document,	"honorificabilitudinitatibus***	thou"	sums	to	just
3,371,	and	so	most	of	the	buckets	in	our	hash	table	will	go	unused.	In
addition,	a	commutative	hash	function,	such	as	addition,	does	not
differentiate	among	the	different	possible	orderings	of	characters	with	a
string.	For	example,	the	words	"rat"	and	"tar"	will	generate	the	same
sums.
We	switched	to	a	hash	function	that	uses	shift	and	
EXCLUSIVE
-
OR
operations.	With	this	version,	shown	as	"Better	hash,"	the	time	drops	to
0.6	seconds.	A	more	systematic	approach	would	be	to	study	the

distribution	of	keys	among	the	buckets	more	carefully,	making	sure	that	it
comes	close	to	what	one	would	expect	if	the	hash	function	had	a	uniform
output	distribution.
Finally,	we	have	reduced	the	run	time	to	the	point	where	most	of	the	time
is	spent	in	
,	and	most	of	the	calls	to	
	occur	as	part	of	the
lowercase	conversion.	We	have	already	seen	that	function	
	has
quadratic	performance,	especially	for	long	strings.	The	words	in	this
document	are	short	enough	to	avoid	the	disastrous	consequences	of
quadratic	performance;	the	longest	bigram	is	just	32	characters.	Still,
switching	to	
,	shown	as	"Linear	lower,"	yields	a	significant
improvement,	with	the	overall	time	dropping	to	around	0.2	seconds.
With	this	exercise,	we	have	shown	that	code	profiling	can	help	drop	the
time	required	for	a	simple	application	from	3.5	minutes	down	to	0.2
seconds,	yielding	a	performance	gain	of	around	1,000×.	The	profiler
helps	us	focus	our	attention	on	the	most	time-consuming	parts	of	the
program	and	also	provides	useful	information	about	the	procedure	call
structure.	Some	of	the	bottlenecks	in	our	code,	such	as	using	a	quadratic
sort	routine,	are	easy	to	anticipate,	while	others,	such	as	whether	to
append	to	the	beginning	or	end	of	a	list,	emerge	only	through	a	careful
analysis.
We	can	see	that	profiling	is	a	useful	tool	to	have	in	the	toolbox,	but	it
should	not	be	the	only	one.	The	timing	measurements	are	imperfect,
especially	for	shorter	(less	than	1	second)	run	times.	More	significantly,
the	results	apply	only	to	the	particular	data	tested.	For	example,	if	we	had
run	the	original	function	on	data	consisting	of	a	smaller	number	of	longer
strings,	we	would	have	found	that	the	lowercase	conversion	routine	was

the	major	performance	bottleneck.	Even	worse,	if	it	only	profiled
documents	with	short	words,	we	might	never	detect	hidden	bottlenecks
such	as	the	quadratic	performance	of	
.	In	general,	profiling	can
help	us	optimize	for	
typical
	cases,	assuming	we	run	the	program	on
representative	data,	but	we	should	also	make	sure	the	program	will	have
respectable	performance	for	all	possible	cases.	This	mainly	involves
avoiding	algorithms	(such	as	insertion	sort)	and	bad	programming
practices	(such	as	
)	that	yield	poor	asymptotic	performance.
Amdahl's	law,	described	in	
Section	
1.9.1
,	provides	some	additional
insights	into	the	performance	gains	that	can	be	obtained	by	targeted
optimizations.	For	our	
n
-gram	code,	we	saw	the	total	execution	time	drop
from	209.0	to	5.4	seconds	when	we	replaced	insertion	sort	by	quicksort.
The	initial	version	spent	203.7	of	its	209.0	seconds	performing	insertion
sort,	giving	
α
	=	0.974,	the	fraction	of	time	subject	to	speedup.	With
quicksort,	the	time	spent	sorting	becomes	negligible,	giving	a	predicted
speedup	of	209/
α
	=	39.0,	close	to	the	measured	speedup	of	38.5.	We
were	able	to	gain	a	large	speedup	because	sorting	constituted	a	very
large	fraction	of	the	overall	execution	time.	However,	when	one
bottleneck	is	eliminated,	a	new	one	arises,	and	so	gaining	additional
speedup	required	focusing	on	other	parts	of	the	program.

5.15	
Summary
Although	most	presentations	on	code	optimization	describe	how
compilers	can	generate	efficient	code,	much	can	be	done	by	an
application	programmer	to	assist	the	compiler	in	this	task.	No	compiler
can	replace	an	inefficient	algorithm	or	data	
structure	by	a	good	one,	and
so	these	aspects	of	program	design	should	remain	a	primary	concern	for
programmers.	We	also	have	seen	that	optimization	blockers,	such	as
memory	aliasing	and	procedure	calls,	seriously	restrict	the	ability	of
compilers	to	perform	extensive	optimizations.	Again,	the	programmer
must	take	primary	responsibility	for	eliminating	these.	These	should
simply	be	considered	parts	of	good	programming	practice,	since	they
serve	to	eliminate	unneeded	work.
Tuning	performance	beyond	a	basic	level	requires	some	understanding
of	the	processor's	microarchitecture,	describing	the	underlying
mechanisms	by	which	the	processor	implements	its	instruction	set
architecture.	For	the	case	of	out-of-order	processors,	just	knowing
something	about	the	operations,	capabilities,	latencies,	and	issue	times
of	the	functional	units	establishes	a	baseline	for	predicting	program
performance.
We	have	studied	a	series	of	techniques—including	loop	unrolling,
creating	multiple	accumulators,	and	reassociation—that	can	exploit	the
instruction-level	parallelism	provided	by	modern	processors.	As	we	get
deeper	into	the	optimization,	it	becomes	important	to	study	the	generated
assembly	code	and	to	try	to	understand	how	the	computation	is	being

performed	by	the	machine.	Much	can	be	gained	by	identifying	the	critical
paths	determined	by	the	data	dependencies	in	the	program,	especially
between	the	different	iterations	of	a	loop.	We	can	also	compute	a
throughput	bound	for	a	computation,	based	on	the	number	of	operations
that	must	be	computed	and	the	number	and	issue	times	of	the	units	that
perform	those	operations.
Programs	that	involve	conditional	branches	or	complex	interactions	with
the	memory	system	are	more	difficult	to	analyze	and	optimize	than	the
simple	loop	programs	we	first	considered.	The	basic	strategy	is	to	try	to
make	branches	more	predictable	or	make	them	amenable	to
implementation	using	conditional	data	transfers.	We	must	also	watch	out
for	the	interactions	between	store	and	load	operations.	Keeping	values	in
local	variables,	allowing	them	to	be	stored	in	registers,	can	often	be
helpful.
When	working	with	large	programs,	it	becomes	important	to	focus	our
optimization	efforts	on	the	parts	that	consume	the	most	time.	Code
profilers	and	related	tools	can	help	us	systematically	evaluate	and
improve	program	performance.	We	described	
GPROF
,	a	standard	Unix
profiling	tool.	More	sophisticated	profilers	are	available,	such	as	the	
VTUNE
program	development	system	from	Intel,	and	
VALGRIND
,	commonly
available	on	Linux	systems.	These	tools	can	break	down	the	execution
time	below	the	procedure	level	to	estimate	the	performance	of	each	
basic
block
	of	the	program.	(A	basic	block	is	a	sequence	of	instructions	that
has	no	transfers	of	control	out	of	its	middle,	and	so	the	block	is	always
executed	in	its	entirety.)

Bibliographic	Notes
Our	focus	has	been	to	describe	code	optimization	from	the	programmer's
perspective,	demonstrating	how	to	write	code	that	will	make	it	easier	for
compilers	to	generate	efficient	code.	An	extended	paper	by	Chellappa,
Franchetti,	and	P$uUschel	[
19
]	
takes	a	similar	approach	but	goes	into
more	detail	with	respect	to	the	processor's	characteristics.
Many	publications	describe	code	optimization	from	a	compiler's
perspective,	formulating	ways	that	compilers	can	generate	more	efficient
code.	Muchnick's	book	is	considered	the	most	comprehensive	[
80
].
Wadleigh	and	Crawford's	book	on	software	optimization	[
115
]	covers
some	of	the	material	we	have	presented,	but	it	also	describes	the
process	of	getting	high	performance	on	parallel	machines.	An	early	paper
by	Mahlke	et	al.	[
75
]	describes	how	several	techniques	developed	for
compilers	that	map	programs	onto	parallel	machines	can	be	adapted	to
exploit	the	instruction-level	parallelism	of	modern	processors.	This	paper
covers	the	code	transformations	we	presented,	including	loop	unrolling,
multiple	accumulators	(which	they	refer	to	as	
accumulator	variable
expansion
),	and	reassociation	(which	they	refer	to	as	
tree	height
reduction
).
Our	presentation	of	the	operation	of	an	out-of-order	processor	is	fairly
brief	and	abstract.	More	complete	descriptions	of	the	general	principles
can	be	found	in	advanced	computer	architecture	textbooks,	such	as	the
one	by	Hennessy	and	Patterson	[
46
,	Ch.	2−3].	Shen	and	Lipasti's	book
[
100
]	provides	an	in-depth	treatment	of	modern	processor	design.

Homework	Problems
5.13	
♦♦
Suppose	we	wish	to	write	a	procedure	that	computes	the	inner	product	of
two	vectors	u	and	v.	An	abstract	version	of	the	function	has	a	CPE	of
14−18	with	x86-64	for	different	types	of	integer	and	floating-point	data.	By
doing	the	same	sort	of	transformations	we	did	to	transform	the	abstract
program	
	into	the	more	efficient	
,	we	get	the	following
code:

Our	measurements	show	that	this	function	has	CPEs	of	1.50	for	integer
data	and	3.00	for	floating-point	data.	For	data	type	double,	the	x86-64
assembly	code	for	the	inner	loop	is	as	follows:
Assume	that	the	functional	units	have	the	characteristics	listed	in	
Figure
5.12
.
A
.	
Diagram	how	this	instruction	sequence	would	be	decoded	into
operations	and	show	how	the	data	dependencies	between	them
would	create	a	critical	path	of	operations,	in	the	style	of	
Figures
5.13
	and	
5.14
.

B
.	
For	data	type	double,	what	lower	bound	on	the	CPE	is	determined
by	the	critical	path?
C
.	
Assuming	similar	instruction	sequences	for	the	integer	code	as
well,	what	lower	bound	on	the	CPE	is	determined	by	the	critical
path	for	integer	data?
D
.	
Explain	how	the	floating-point	versions	can	have	CPEs	of	3.00,
even	though	the	multiplication	operation	requires	5	clock	cycles.
5.14	
♦
Write	a	version	of	the	inner	product	procedure	described	in	
Problem
5.13
	that	uses	6	×	1	loop	unrolling.	For	x86-64,	our	measurements	of
the	unrolled	version	give	a	CPE	of	1.07	for	integer	data	but	still	3.01	for
both	floating-point	data.
A
.	
Explain	why	any	(scalar)	version	of	an	inner	product	procedure
running	on	an	Intel	Core	i7	Haswell	processor	cannot	achieve	a
CPE	less	than	1.00.
B
.	
Explain	why	the	performance	for	floating-point	data	did	not
improve	with	loop	unrolling.
5.15	
♦
Write	a	version	of	the	inner	product	procedure	described	in	
Problem
5.13
	that	uses	6	×	6	loop	unrolling.	Our	measurements	for	this	function

with	x86-64	give	a	CPE	of	1.06	for	integer	data	and	1.01	for	floating-point
data.
What	factor	limits	the	performance	to	a	CPE	of	1.00?
5.16	
♦
Write	a	version	of	the	inner	product	procedure	described	in	
Problem
5.13
	that	uses	6	×	1
a
	loop	unrolling	to	enable	greater	parallelism.	Our
measurements	for	this	function	give	a	CPE	of	1.10	for	integer	data	and
1.05	for	floating-point	data.
5.17	
♦♦
The	library	function	
	has	the	following	prototype:
This	function	fills	
	bytes	of	the	memory	area	starting	at	
	with	copies	of
the	low-order	byte	of	
.	For	example,	it	can	be	used	to	zero	out	a	region
of	memory	by	giving	argument	0	for	
,	but	other	values	are	possible.
The	following	is	a	straightforward	implementation	of	
:

Implement	a	more	efficient	version	of	the	function	by	using	a	word	of	data
type	
	to	pack	eight	copies	of	
,	and	then	step	through	the
region	using	word-level	writes.	You	might	find	it	helpful	to	do	additional
loop	unrolling	as	well.	On	our	reference	machine,	we	were	able	to	reduce
the	CPE	from	1.00	for	the	straightforward	implementation	to	0.127.	That
is,	the	program	is	able	to	write	8	bytes	every	clock	cycle.
Here	are	some	additional	guidelines.	To	ensure	portability,	let	
K
	denote
the	value	of	
	for	the	machine	on	which	you	run
your	program.
You	may	not	call	any	library	functions.
Your	code	should	work	for	arbitrary	values	of	
,	including	when	it	is
not	a	multiple	of	
K.
	You	can	do	this	in	a	manner	similar	to	the	way	we
finish	the	last	few	iterations	with	loop	unrolling.

You	should	write	your	code	so	that	it	will	compile	and	run	correctly	on
any	machine	regardless	of	the	value	of	
K.
	Make	use	of	the	operation
	to	do	this.
On	some	machines,	unaligned	writes	can	be	much	slower	than
aligned	ones.	(On	some	non-x86	machines,	they	can	even	cause
segmentation	faults.)	Write	your	code	so	that	it	starts	with	byte-level
writes	until	the	destination	address	is	a	multiple	of	
K
,	then	do	word-
level	writes,	and	then	(if	necessary)	finish	with	byte-level	writes.
Beware	of	the	case	where	
	is	small	enough	that	the	upper	bounds
on	some	of	the	loops	become	negative.	With	expressions	involving
the	
	operator,	the	testing	may	be	performed	with	unsigned
arithmetic.	(See	
Section	
2.2.8
	and	
Problem	
2.72
.)
5.18	
♦♦♦
We	considered	the	task	of	polynomial	evaluation	in	Practice	Problems
5.5	and	5.6,	with	both	a	direct	evaluation	and	an	evaluation	by	Horner's
method.	Try	to	write	
faster	versions	of	the	function	using	the	optimization
techniques	we	have	explored,	including	loop	unrolling,	parallel
accumulation,	and	reassociation.	You	will	find	many	different	ways	of
mixing	together	Horner's	scheme	and	direct	evaluation	with	these
optimization	techniques.
Ideally,	you	should	be	able	to	reach	a	CPE	close	to	the	throughput	limit	of
your	machine.	Our	best	version	achieves	a	CPE	of	1.07	on	our	reference
machine.

5.19	
♦♦♦
In	
Problem	
5.12
,	we	were	able	to	reduce	the	CPE	for	the	prefix-sum
computation	to	3.00,	limited	by	the	latency	of	floating-point	addition	on
this	machine.	Simple	loop	unrolling	does	not	improve	things.
Using	a	combination	of	loop	unrolling	and	reassociation,	write	code	for	a
prefix	sum	that	achieves	a	CPE	less	than	the	latency	of	floating-point
addition	on	your	machine.	Doing	this	requires	actually	increasing	the
number	of	additions	performed.	For	example,	our	version	with	two-way
unrolling	requires	three	additions	per	iteration,	while	our	version	with	four-
way	unrolling	requires	five.	Our	best	implementation	achieves	a	CPE	of
1.67	on	our	reference	machine.
Determine	how	the	throughput	and	latency	limits	of	your	machine	limit	the
minimum	CPE	you	can	achieve	for	the	prefix-sum	operation.

Solutions	to	Practice	Problems
Solution	to	Problem	
5.1	
(page
500
)
This	problem	illustrates	some	of	the	subtle	effects	of	memory	aliasing.
As	the	following	commented	code	shows,	the	effect	will	be	to	set	the
value	at	
	to	zero:
This	example	illustrates	that	our	intuition	about	program	behavior	can
often	be	wrong.	We	naturally	think	of	the	case	where	
	and	
	are
distinct	but	overlook	the	possibility	that	they	might	be	equal.	Bugs	often
arise	due	to	conditions	the	programmer	does	not	anticipate.
Solution	to	Problem	
5.2	
(page

504
)
This	problem	illustrates	the	relationship	between	CPE	and	absolute
performance.	It	can	be	solved	using	elementary	algebra.	We	find	that	for
n
	≤	2,	version	1	is	the	fastest.	Version	2	is	fastest	for	3	≤	
n
	≤	7,	and
version	3	is	fastest	for	
n
	≥	8.
Solution	to	Problem	
5.3	
(page
512
)
This	is	a	simple	exercise,	but	it	is	important	to	recognize	that	the	four
statements	of	a	
	loop—initial,	test,	update,	and	body—get	executed
different	numbers	of	times.
Code
A.
1
91
90
90
B.
91
1
90
90
C.
1
1
90
90
Solution	to	Problem	
5.4	
(page
516
)

This	assembly	code	demonstrates	a	clever	optimization	opportunity
detected	by	
GCC
.	It	is	worth	studying	this	code	carefully	to	better
understand	the	subtleties	of	code	optimization.
A
.	
In	the	less	optimized	code,	register	
	is	simply	used	as	a
temporary	value,	both	set	and	used	on	each	loop	iteration.	In	the
more	optimized	code,	it	is	used	more	in	the	manner	of	variable
	in	
,	accumulating	the	product	of	the	vector	elements.
The	difference	with	
,	however,	is	that	location	
	is
updated	on	each	iteration	by	the	second	
	instruction.
We	can	see	that	this	optimized	version	operates	much	like	the
following	C	code:

B
.	
The	two	versions	of	
	will	have	identical	functionality,	even
with	memory	aliasing.
C
.	
This	transformation	can	be	made	without	changing	the	program
behavior,	because,	with	the	exception	of	the	first	iteration,	the
value	read	from	
	at	the	beginning	of	each	iteration	will	be	the
same	value	written	to	this	register	
at	the	end	of	the	previous
iteration.	Therefore,	the	combining	instruction	can	simply	use	the
value	already	in	
	at	the	beginning	of	the	loop.
Solution	to	Problem	
5.5	
(page
530
)
Polynomial	evaluation	is	a	core	technique	for	solving	many	problems.	For
example,	polynomial	functions	are	commonly	used	to	approximate
trigonometric	functions	in	math	libraries.
A
.	
The	function	performs	2
n
	multiplications	and	
n
	additions.
B
.	
We	can	see	that	the	performance-limiting	computation	here	is	the
repeated	computation	of	the	expression	
.	This
requires	a	floating-point	multiplication	(5	clock	cycles),	and	the
computation	for	one	iteration	cannot	begin	until	the	one	for	the
previous	iteration	has	completed.	The	updating	of	
	only
requires	a	floating-point	addition	(3	clock	cycles)	between
successive	iterations.

Solution	to	Problem	
5.6	
(page
530
)
This	problem	demonstrates	that	minimizing	the	number	of	operations	in	a
computation	may	not	improve	its	performance.
A
.	
The	function	performs	
n
	multiplications	and	
n
	additions,	half	the
number	of	multiplications	as	the	original	function	
.
B
.	
We	can	see	that	the	performance-limiting	computation	here	is	the
repeated	computation	of	the	expression	
Starting	from	the	value	of	
	from	the	previous	iteration,	we
must	first	multiply	it	by	
	(5	clock	cycles)	and	then	add	it	to	
(3	cycles)	before	we	have	the	value	for	this	iteration.	Thus,	each
iteration	imposes	a	minimum	latency	of	8	cycles,	exactly	our
measured	CPE.
C
.	
Although	each	iteration	in	function	
	requires	two
multiplications	rather	than	one,	only	a	single	multiplication	occurs
along	the	critical	path	per	iteration.
Solution	to	Problem	
5.7	
(page
532
)
The	following	code	directly	follows	the	rules	we	have	stated	for	unrolling
a	loop	by	some	factor	
k:

Solution	to	Problem	
5.8	
(page
545
)

This	problem	demonstrates	how	small	changes	in	a	program	can	yield
dramatic	performance	differences,	especially	on	a	machine	with	out-of-
order	execution.	
Figure	
5.39
	diagrams	the	three	multiplication
operations	for	a	single	iteration	of	the	function.	In	this	figure,	the
operations	shown	as	blue	boxes	are	along	the	critical	path—they	need	to
be	computed	in	sequence	to	compute	a	new	value	for	loop	variable	
.
The	operations	shown	as	light	boxes	can	be	computed	in	parallel	with	the
critical	path	operations.	For	a	loop	with	
P
	operations	along	the	critical
path,	each	iteration	will	require	a	minimum	of	5
P
	clock	cycles	and	will
compute	the	product	for	three	elements,	giving	a	lower	bound	on	the
CPE	of	5
P
/3.	This	implies	lower	bounds	of	5.00	for	Al,	3.33	for	A2	and
A5,	and	1.67	for	A3	and	A4.	We	ran	these	functions	on	an	Intel	Core	i7
Haswell	processor	and	found	that	it	could	achieve	these	CPE	values.
Solution	to	Problem	
5.9	
(page
553
)
This	is	another	demonstration	that	a	slight	change	in	coding	style	can
make	it	much	easier	for	the	compiler	to	detect	opportunities	to	use
conditional	moves:

Figure	
5.39	
Data	dependencies	among	multiplication	operations	for
cases	in	
Problem	
5.8
.
The	operations	shown	as	blue	boxes	form	the	critical	paths	for	the
iterations.
We	measured	a	CPE	of	around	12.0	for	this	version	of	the	code,	a
modest	improvement	over	the	original	CPE	of	15.0.
Solution	to	Problem	
5.10	
(page
559
)

This	problem	requires	you	to	analyze	the	potential	load-store	interactions
in	a	program.
A
.	
It	will	set	each	element	
	to	
i
	+	1,	for	0	≤	
i
	≤	998.
B
.	
It	will	set	each	element	
	to	0,	for	1	≤	
i
	≤	999.
C
.	
In	the	second	case,	the	load	of	one	iteration	depends	on	the	result
of	the	store	from	the	previous	iteration.	Thus,	there	is	a	write/read
dependency	between	successive	iterations.
D
.	
It	will	give	a	CPE	of	1.2,	the	same	as	for	Example	A,	since	there
are	no	dependencies	between	stores	and	subsequent	loads.
Solution	to	Problem	
5.11	
(page
561
)
We	can	see	that	this	function	has	a	write/read	dependency	between
successive	iterations—the	destination	value	
	on	one	iteration
matches	the	source	value	
	on	the	next.	A	critical	path	is	therefore
formed	for	each	iteration	consisting	of	a	store	(from	the	previous
iteration),	a	load,	and	a	floating-point	addition.	The	CPE	measurement	of
9.0	is	consistent	with	our	measurement	of	7.3	for	the	CPE	of	
when	there	is	a	data	dependency,	since	
	involves	an	integer
addition	(1	clock-cycle	latency),	while	
	involves	a	floating-point
addition	(3	clock-cycle	latency).
Solution	to	Problem	
5.12	
(page

561
)
Here	is	a	revised	version	of	the	function:
We	introduce	a	local	variable	
.	At	the	start	of	iteration	
,	it	holds
the	value	of	
.	We	then	compute	
	to	be	the	value	of	
	and	to
be	the	new	value	for	
.
This	version	compiles	to	the	following	assembly	code:

This	code	holds	
	in	
,	avoiding	the	need	to	read	
	from
memory	and	thus	eliminating	the	write/read	dependency	seen	in	
.

Chapter	
6	
The	Memory	Hierarchy
6.1	
Storage	Technologies	
581
6.2	
Locality	
604
6.3	
The	Memory	Hierarchy	
609
6.4	
Cache	Memories	
614
6.5	
Writing	Cache-Friendly	Code	
633
6.6	
Putting	It	Together:	The	Impact	of	Caches	on	Program
Performance	
639
6.7	
Summary
	
648
Bibliographic	Notes	
648
Homework	Problems	
649
Solutions	to	Practice	Problems	
660
To	this	point	in	our	study	of	systems,	we	have	relied
on	a	simple	model	of	a	computer	system	as	a	CPU
that	executes	instructions	and	a	memory	system
that	holds	instructions	and	data	for	the	CPU.	In	our

simple	model,	the	memory	system	is	a	linear	array
of	bytes,	and	the	CPU	can	access	each	memory
location	in	a	constant	amount	of	time.	While	this	is
an	effective	model	up	to	a	point,	it	does	not	reflect
the	way	that	modern	systems	really	work.
In	practice,	a	
memory	system
	is	a	hierarchy	of
storage	devices	with	different	capacities,	costs,	and
access	times.	CPU	registers	hold	the	most
frequently	used	data.	Small,	fast	
cache	memories
nearby	the	CPU	act	as	staging	areas	for	a	subset	of
the	data	and	instructions	stored	in	the	relatively	slow
main	memory.	The	main	memory	stages	data	stored
on	large,	slow	disks,	which	in	turn	often	serve	as
staging	areas	for	data	stored	on	the	disks	or	tapes
of	other	machines	connected	by	networks.
Memory	hierarchies	work	because	well-written
programs	tend	to	access	the	storage	at	any
particular	level	more	frequently	than	they	access	the
storage	at	the	next	lower	level.	So	the	storage	at	the
next	level	can	be	slower,	and	thus	larger	and
cheaper	per	bit.	The	overall	effect	is	a	large	pool	of
memory	that	costs	as	much	as	the	cheap	storage
near	the	bottom	of	the	hierarchy	but	that	serves
data	to	programs	at	the	rate	of	the	fast	storage	near
the	top	of	the	hierarchy.

As	a	programmer,	you	need	to	understand	the
memory	hierarchy	because	it	has	a	big	impact	on
the	performance	of	your	applications.	If	the	data
your	program	needs	are	stored	in	a	CPU	register,
then	they	can	be	accessed	in	0	cycles	during	the
execution	of	the	instruction.	If	stored	in	a	cache,	4	to
75	cycles.	If	stored	in	main	memory,	hundreds	of
cycles.	And	if	stored	in	disk,	tens	of	millions	of
cycles!
Here,	then,	is	a	fundamental	and	enduring	idea	in
computer	systems:	if	you	understand	how	the
system	moves	data	up	and	down	the	memory
hierarchy,	then	you	can	write	your	application
programs	so	that	their	data	items	are	stored	higher
in	the	hierarchy,	where	the	CPU	can	access	them
more	quickly.
This	idea	centers	around	a	fundamental	property	of
computer	programs	known	as	
locality.
	Programs
with	good	locality	tend	to	access	the	same	set	of
data	items	over	and	over	again,	or	they	tend	to
access	sets	of	nearby	data	items.	Programs	with
good	locality	tend	to	access	more	data	items	from
the	upper	levels	of	the	memory	hierarchy	than
programs	with	poor	locality,	and	thus	run	faster.	For
example,	on	our	Core	i7	system,	the	running	times
of	different	matrix	multiplication	kernels	that	perform
the	same	number	of	arithmetic	operations,	but	have

different	degrees	of	locality,	can	vary	by	a	factor	of
almost	40!
In	this	chapter,	we	will	look	at	the	basic	storage
technologies—SRAM	memory,	DRAM	memory,
ROM	memory,	and	rotating	and	solid	state	disks—
and	describe	how	they	are	organized	into
hierarchies.	In	particular,	we	focus	on	the	cache
memories	that	act	as	staging	areas	between	the
CPU	and	main	memory,	because	they	have	the
most	impact	on	application	program	performance.
We	show	you	how	to	analyze	your	C	programs	for
locality,	and	we	introduce	techniques	for	improving
the	locality	in	your	programs.	You	will	also	learn	an
interesting	way	to	characterize	the	performance	of
the	memory	hierarchy	on	a	particular	machine	as	a
"memory	mountain"	that	shows	read	access	times
as	a	function	of	locality.

6.1	
Storage	Technologies
Much	of	the	success	of	computer	technology	stems	from	the	tremendous
progress	in	storage	technology.	Early	computers	had	a	few	kilobytes	of
random	access	memory.	The	earliest	IBM	PCs	didn't	even	have	a	hard
disk.	That	changed	with	the	introduction	of	the	IBM	PC-XT	in	1982,	with
its	10-megabyte	disk.	By	the	year	2015,	typical	machines	had	300,000
times	as	much	disk	storage,	and	the	amount	of	storage	was	increasing
by	a	factor	of	2	every	couple	of	years.
6.1.1	
Random	Access	Memory
Random	access	memory	(RAM)
	comes	in	two	varieties—static	and
dynamic.	
Static	RAM	(SRAM)
	is	faster	and	significantly	more	expensive
than	
dynamic	RAM	(DRAM).
	SRAM	is	used	for	cache	memories,	both	on
and	off	the	CPU	chip.	DRAM	is	used	for	the	main	memory	plus	the	frame
buffer	of	a	graphics	system.	Typically,	a	desktop	system	will	have	no
more	than	a	few	tens	of	megabytes	of	SRAM,	but	hundreds	or	thousands
of	megabytes	of	DRAM.
Static	RAM
SRAM	stores	each	bit	in	a	
bistable
	memory	cell.	Each	cell	is
implemented	with	a	six-transistor	circuit.	This	circuit	has	the	property	that
it	can	stay	indefinitely	in	either	of	two	different	voltage	configurations,	or

states.
	Any	other	state	will	be	unstable—starting	from	there,	the	circuit
will	quickly	move	toward	one	of	the	stable	states.	Such	a	memory	cell	is
analogous	to	the	inverted	pendulum	illustrated	in	
Figure	
6.1
.
The	pendulum	is	stable	when	it	is	tilted	either	all	the	way	to	the	left	or	all
the	way	to	the	right.	From	any	other	position,	the	pendulum	will	fall	to	one
side	or	the	other.	In	principle,	the	pendulum	could	also	remain	balanced
in	a	vertical	position	indefinitely,	but	this	state	is	
metastable
—the	smallest
disturbance	would	make	it	start	to	fall,	and	once	it	fell	it	would	never
return	to	the	vertical	position.
Due	to	its	bistable	nature,	an	SRAM	memory	cell	will	retain	its	value
indefinitely,	as	long	as	it	is	kept	powered.	Even	when	a	disturbance,	such
as	electrical	noise,	perturbs	the	voltages,	the	circuit	will	return	to	the
stable	value	when	the	disturbance	is	removed.
Figure	
6.1	
Inverted	pendulum.
Like	an	SRAM	cell,	the	pendulum	has	only	two	stable	configurations,	or
states.
Transistors
per	bit
Relative
access
time
Persistent?
Sensitive?
Relative
cost
Applications
SRAM
6
1×
Yes
No
1,000×
Cache
memory

DRAM
1
10×
No
Yes
1×
Main
memory,
frame	buffers
Figure	
6.2	
Characteristics	of	DRAM	and	SRAM	memory.
Dynamic	RAM
DRAM	stores	each	bit	as	charge	on	a	capacitor.	This	capacitor	is	very
small—	typically	around	30	femtofarads—that	is,	30	×	10
	farads.
Recall,	however,	that	a	farad	is	a	very	large	unit	of	measure.	DRAM
storage	can	be	made	very	dense—each	cell	consists	of	a	capacitor	and	a
single	access	transistor.	Unlike	SRAM,	however,	a	DRAM	memory	cell	is
very	sensitive	to	any	disturbance.	When	the	capacitor	voltage	is
disturbed,	it	will	never	recover.	Exposure	to	light	rays	will	cause	the
capacitor	voltages	to	change.	In	fact,	the	sensors	in	digital	cameras	and
camcorders	are	essentially	arrays	of	DRAM	cells.
Various	sources	of	leakage	current	cause	a	DRAM	cell	to	lose	its	charge
within	a	time	period	of	around	10	to	100	milliseconds.	Fortunately,	for
computers	operating	with	clock	cycle	times	measured	in	nanoseconds,
this	retention	time	is	quite	long.	The	memory	system	must	periodically
refresh	every	bit	of	memory	by	reading	it	out	and	then	rewriting	it.	Some
systems	also	use	error-correcting	codes,	where	the	computer	words	are
encoded	using	a	few	more	bits	(e.g.,	a	64-bit	word	might	be	encoded
using	72	bits),	such	that	circuitry	can	detect	and	correct	any	single
erroneous	bit	within	a	word.
−15

Figure	
6.2
	summarizes	the	characteristics	of	SRAM	and	DRAM
memory.	SRAM	is	persistent	as	long	as	power	is	applied.	Unlike	DRAM,
no	refresh	is	necessary.	SRAM	can	be	accessed	faster	than	DRAM.
SRAM	is	not	sensitive	to	disturbances	such	as	light	and	electrical	noise.
The	trade-off	is	that	SRAM	cells	use	more	transistors	than	DRAM	cells
and	thus	have	lower	densities,	are	more	expensive,	and	consume	more
power.
Conventional	DRAMs
The	cells	(bits)	in	a	DRAM	chip	are	partitioned	into	
d	supercells
,	each
consisting	of	
w
	DRAM	cells.	
Ad
	×	
w
	DRAM	stores	a	total	of	
dw
	bits	of
information.	The	supercells	are	organized	as	a	rectangular	array	with	
r
rows	and	
c
	columns,	where	
rc
	=	
d
.	Each	supercell	has	an	address	of	the
form	
(i,	j)
,	where	
i
	denotes	the	row	and	
j
	denotes	the	column.
For	example,	
Figure	
6.3
	shows	the	organization	of	a	16	×	8	DRAM
chip	with	
d
	=	16	supercells,	
w
	=	8	bits	per	supercell,	
r
	=	4	rows,	and	
c
	=	4
columns.	The	shaded	box	denotes	the	supercell	at	address	(2,1).
Information	flows	in	and	out	of	the	chip	via	external	connectors	called
pins.
	Each	pin	carries	a	1-bit	signal.	
Figure	
6.3
	shows	two	of	these
sets	of	pins:	eight	data	pins	that	can	transfer	1	byte
Aside	
A	note	on	terminology
The	storage	community	has	never	settled	on	a	standard	name	for
a	DRAM	array	element.	Computer	architects	tend	to	refer	to	it	as	a
"cell,"	overloading	the	term	with	the	DRAM	storage	cell.	Circuit
designers	tend	to	refer	to	it	as	a	"word,"	overloading	the	term	with

a	word	of	main	memory.	To	avoid	confusion,	we	have	adopted	the
unambiguous	term	"supercell."
Figure	
6.3	
High-level	view	of	a	128-bit	16	×	8	DRAM	chip.
in	or	out	of	the	chip,	and	two	
	pins	that	carry	two-bit	row	and	column
supercell	addresses.	Other	pins	that	carry	control	information	are	not
shown.
Each	DRAM	chip	is	connected	to	some	circuitry,	known	as	the	
memory
controller
,	that	can	transfer	
w
	bits	at	a	time	to	and	from	each	DRAM	chip.
To	read	the	contents	of	supercell	
(i,	j)
,	the	memory	controller	sends	the
row	address	
i
	to	the	DRAM,	followed	by	the	column	address	
j.
	The	DRAM
responds	by	sending	the	contents	of	supercell	
(i,	j)
	back	to	the	controller.
The	row	address	
i
	is	called	a	
RAS	(row	access	strobe)	request.
	The
column	address	
j
	is	called	a	
CAS	(column	access	strobe)	request.
	Notice
that	the	RAS	and	CAS	requests	share	the	same	DRAM	address	pins.
For	example,	to	read	supercell	(2,1)	from	the	16	×	8	DRAM	in	
Figure
6.3
,	the	memory	controller	sends	row	address	2,	as	shown	in	
Figure
6.4(a)
.	The	DRAM	responds	by	copying	the	entire	contents	of	row	2
into	an	internal	row	buffer.	Next,	the	memory	controller	sends	column
address	1,	as	shown	in	
Figure	
6.4(b)
.	The	DRAM	responds	by

copying	the	8	bits	in	supercell	(2,1)	from	the	row	buffer	and	sending	them
to	the	memory	controller.
One	reason	circuit	designers	organize	DRAMs	as	two-dimensional	arrays
instead	of	linear	arrays	is	to	reduce	the	number	of	address	pins	on	the
chip.	For	example,	if	our	example	128-bit	DRAM	were	organized	as	a
linear	array	of	16	supercells	with	addresses	0	to	15,	then	the	chip	would
need	four	address	pins	instead	of	two.	The	disadvantage	of	the	two-
dimensional	array	organization	is	that	addresses	must	be	sent	in	two
distinct	steps,	which	increases	the	access	time.
Figure	
6.4	
Reading	the	contents	of	a	DRAM	supercell.
Memory	Modules
DRAM	chips	are	packaged	in	
memory	modules
	that	plug	into	expansion
slots	on	the	main	system	board	(motherboard).	Core	i7	systems	use	the
240-pin	
dual	inline	memory	module	(DIMM)
,	which	transfers	data	to	and
from	the	memory	controller	in	64-bit	chunks.

Figure	
6.5
	shows	the	basic	idea	of	a	memory	module.	The	example
module	stores	a	total	of	64	MB	(megabytes)	using	eight	64-Mbit	8M	×	8
DRAM	chips,	numbered	0	to	7.	Each	supercell	stores	1	byte	of	
main
memory
,	and	each	64-bit	word	at	byte	address	A	in	main	memory	is
represented	by	the	eight	supercells	whose	corresponding	supercell
address	is	
(i,	j).
	In	the	example	in	
Figure	
6.5
,	DRAM	0	stores	the	first
(lower-order)	byte,	DRAM	1	stores	the	next	byte,	and	so	on.
To	retrieve	the	word	at	memory	address	
A
,	the	memory	controller
converts	A	to	a	supercell	address	
(i,	j)
	and	sends	it	to	the	memory
module,	which	then	broadcasts	
i
	and	
j
	to	each	DRAM.	In	response,	each
DRAM	outputs	the	8-bit	contents	of	its	
(i,	j)
	supercell.	Circuitry	in	the
module	collects	these	outputs	and	forms	them	into	a	64-bit	word,	which	it
returns	to	the	memory	controller.
Main	memory	can	be	aggregated	by	connecting	multiple	memory
modules	to	the	memory	controller.	In	this	case,	when	the	controller
receives	an	address	
A
,	the	controller	selects	the	module	
k
	that	contains
A
,	converts	
A
	to	its	
(i,	j)
	form,	and	sends	
(i,	j)
	to	module	
k.
Practice	Problem	
6.1	
(solution	page	
660
)
In	the	following,	let	
r
	be	the	number	of	rows	in	a	DRAM	array,	
c
	the
number	of	columns,	
b
	the	number	of	bits	needed	to	address	the
rows,	and	
b
	the	number	of	bits	needed	to	address	the	columns.
For	each	of	the	following	DRAMs,	determine	the	power-of-2	array
dimensions	that	minimize	max(
b
,	b
),	the	maximum	number	of	bits
needed	to	address	the	rows	or	columns	of	the	array.
r
c
r
c

Figure	
6.5	
Reading	the	contents	of	a	memory	module.
Organization
r
c
b
b
max
(
b
,	b
)
16	×	1
_____
_____
_____
_____
_____
16	×	4
_____
_____
_____
_____
_____
128	×	8
_____
_____
_____
_____
_____
512	×	4
_____
_____
_____
_____
_____
1,024	×	4
_____
_____
_____
_____
_____
Enhanced	DRAMs
There	are	many	kinds	of	DRAM	memories,	and	new	kinds	appear	on	the
market	with	regularity	as	manufacturers	attempt	to	keep	up	with	rapidly
increasing	processor	speeds.	Each	is	based	on	the	conventional	DRAM
r
c
r
c

cell,	with	optimizations	that	improve	the	speed	with	which	the	basic
DRAM	cells	can	be	accessed.
Fast	page	mode	DRAM	(FPM	DRAM).	
A	conventional	DRAM	copies
an	entire	row	of	supercells	into	its	internal	row	buffer,	uses	one,	and
then	discards	the	rest.	FPM	DRAM	improves	on	this	by	allowing
consecutive	accesses	to	the	same	row	to	be	served	directly	from	the
row	buffer.	For	example,	to	read	four	supercells	from	row	
i
	of	a
conventional	DRAM,	the	memory	controller	must	send	four	RAS/CAS
requests,	even	though	the	row	address	
i
	is	identical	in	each	case.	To
read	supercells	from	the	same	row	of	an	FPM	DRAM,	the	memory
controller	sends	an	initial	RAS/CAS	request,	followed	by	three	CAS
requests.	The	initial	RAS/CAS	request	copies	row	
i
	into	the	row	buffer
and	returns	the	supercell	addressed	by	the	
CAS.	The	next	three
supercells	are	served	directly	from	the	row	buffer,	and	thus	are
returned	more	quickly	than	the	initial	supercell.
Extended	data	out	DRAM	(EDO	DRAM).	
An	enhanced	form	of	FPM
DRAM	that	allows	the	individual	CAS	signals	to	be	spaced	closer
together	in	time.
Synchronous	DRAM	(SDRAM).	
Conventional,	FPM,	and	EDO
DRAMs	are	asynchronous	in	the	sense	that	they	communicate	with
the	memory	controller	using	a	set	of	explicit	control	signals.	SDRAM
replaces	many	of	these	control	signals	with	the	rising	edges	of	the
same	external	clock	signal	that	drives	the	memory	controller.	Without
going	into	detail,	the	net	effect	is	that	an	SDRAM	can	output	the
contents	of	its	supercells	at	a	faster	rate	than	its	asynchronous
counterparts.

Double	Data-Rate	Synchronous	DRAM	(DDR	SDRAM).	
DDR
SDRAM	is	an	enhancement	of	SDRAM	that	doubles	the	speed	of	the
DRAM	by	using	both	clock	edges	as	control	signals.	Different	types	of
DDR	SDRAMs	are	characterized	by	the	size	of	a	small	prefetch	buffer
that	increases	the	effective	bandwidth:	DDR	(2	bits),	DDR2	(4	bits),
and	DDR3	(8	bits).
Video	RAM	(VRAM).	
Used	in	the	frame	buffers	of	graphics	systems.
VRAM	is	similar	in	spirit	to	FPM	DRAM.	Two	major	differences	are
that	(1)	VRAM	output	is	produced	by	shifting	the	entire	contents	of	the
internal	buffer	in	sequence	and	(2)	VRAM	allows	concurrent	reads
and	writes	to	the	memory.	Thus,	the	system	can	be	painting	the
screen	with	the	pixels	in	the	frame	buffer	(reads)	while	concurrently
writing	new	values	for	the	next	update	(writes).
Nonvolatile	Memory
DRAMs	and	SRAMs	are	
volatile
	in	the	sense	that	they	lose	their
information	if	the	supply	voltage	is	turned	off.	
Nonvolatile	memories
,	on
the	other	hand,	retain	their	information	even	when	they	are	powered	off.
There	are	a	variety	of	nonvolatile	memories.	For	historical	reasons,	they
are	referred	to	collectively	as	
read-only	memories
	(ROMs),	even	though
some	types	of	ROMs	can	be	written	to	as	well	as	read.	ROMs	are
distinguished	by	the	number	of	times	they	can	be	reprogrammed	(written
to)	and	by	the	mechanism	for	reprogramming	them.
Aside	
Historical	popularity	of	DRAM
technologies

Until	1995,	most	PCs	were	built	with	FPM	DRAMs.	From	1996	to
1999,	EDO	DRAMs	dominated	the	market,	while	FPM	DRAMs	all
but	disappeared.	SDRAMs	first	appeared	in	1995	in	high-end
systems,	and	by	2002	most	PCs	were	built	with	SDRAMs	and
DDR	SDRAMs.	By	2010,	most	server	and	desktop	systems	were
built	with	DDR3	SDRAMs.	In	fact,	the	Intel	Core	i7	supports	only
DDR3	SDRAM.
A	
programmable	ROM	(PROM)
	can	be	programmed	exactly	once.
PROMs	include	a	sort	of	fuse	with	each	memory	cell	that	can	be	blown
once	by	zapping	it	with	a	high	current.
An	
erasable	programmable	ROM	(EPROM)
	has	a	transparent	quartz
window	that	permits	light	to	reach	the	storage	cells.	The	EPROM	cells
are	cleared	to	zeros	by	shining	ultraviolet	light	through	the	window.
Programming	an	EPROM	is	done	by	using	a	special	device	to	write	ones
into	the	EPROM.	An	EPROM	can	be	erased	and	reprogrammed	on	the
order	of	1,000	times.	An	
electrically	erasable	PROM	(EEPROM)
	is	akin	to
an	EPROM,	but	it	does	not	require	a	physically	separate	programming
device,	and	thus	can	be	reprogrammed	in-place	on	printed	circuit	cards.
An	EEPROM	can	be	reprogrammed	on	the	order	of	10
	times	before	it
wears	out.
Flash	memory
	is	a	type	of	nonvolatile	memory,	based	on	EEPROMs,	that
has	become	an	important	storage	technology.	Flash	memories	are
everywhere,	providing	fast	and	durable	nonvolatile	storage	for	a	slew	of
electronic	devices,	including	digital	cameras,	cell	phones,	and	music
players,	as	well	as	laptop,	desktop,	and	server	computer	systems.	In
Section	
6.1.3
,	we	will	look	in	detail	at	a	new	form	of	flash-based	disk
5

drive,	known	as	a	
solid	state	disk	(SSD)
,	that	provides	a	faster,	sturdier,
and	less	power-hungry	alternative	to	conventional	rotating	disks.
Programs	stored	in	ROM	devices	are	often	referred	to	as	
firmware.
	When
a	computer	system	is	powered	up,	it	runs	firmware	stored	in	a	ROM.
Some	systems	provide	a	small	set	of	primitive	input	and	output	functions
in	firmware—for	example,	a	PC's	BIOS	(basic	input/output	system)
routines.	Complicated	devices	such	as	graphics	cards	and	disk	drive
controllers	also	rely	on	firmware	to	translate	I/O	(input/output)	requests
from	the	CPU.
Accessing	Main	Memory
Data	flows	back	and	forth	between	the	processor	and	the	DRAM	main
memory	over	shared	electrical	conduits	called	
buses.
	Each	transfer	of
data	between	the	CPU	and	memory	is	accomplished	with	a	series	of
steps	called	a	
bus	transaction.
	A	
read	transaction
	transfers	data	from	the
main	memory	to	the	CPU.	A	
write	transaction
	transfers	data	from	the
CPU	to	the	main	memory.
A	
bus
	is	a	collection	of	parallel	wires	that	carry	address,	data,	and	control
signals.	Depending	on	the	particular	bus	design,	data	and	address
signals	can	share	the	same	set	of	wires	or	can	use	different	sets.	Also,
more	than	two	devices	can	share	the	same	bus.	The	control	wires	carry
signals	that	synchronize	the	transaction	and	identify	what	kind	of
transaction	is	currently	being	performed.	For	example,	is	this	transaction
of	interest	to	the	main	memory,	or	to	some	other	I/O	device	such	as	a
disk	controller?	Is	the	transaction	a	read	or	a	write?	Is	the	information	on
the	bus	an	address	or	a	data	item?

Figure	
6.6
	shows	the	configuration	of	an	example	computer	system.
The	main	components	are	the	CPU	chip,	a	chipset	that	we	will	call	an	
I/O
bridge
	(which	includes	the	memory	controller),	and	the	DRAM	memory
modules	that	make	up	main	memory.	These	components	are	connected
by	a	pair	of	buses:	a	
system	bus
	that	connects	the	CPU	to	the	I/O	bridge,
and	a	
memory	bus
	that	connects	the	I/O
Aside	
A	note	on	bus	designs
Bus	design	is	a	complex	and	rapidly	changing	aspect	of	computer
systems.	Different	vendors	develop	different	bus	architectures	as
a	way	to	differentiate	their	products.	For	example,	some	Intel
systems	use	chipsets	known	as	the	
northbridge
	and	the
southbridge
	to	connect	the	CPU	to	memory	and	I/O	devices,
respectively.	In	older	Pentium	and	Core	2	systems,	a	
front	side
bus
	(FSB)	connects	the	CPU	to	the	northbridge.	Systems	from
AMD	replace	the	FSB	with	the	
HyperTransport
	interconnect,	while
newer	Intel	Core	i7	systems	use	the	
QuickPath
	interconnect.	The
details	of	these	different	bus	architectures	are	beyond	the	scope
of	this	text.	Instead,	we	will	use	the	high-level	bus	architecture
from	
Figure	
6.6
	as	a	running	example	throughout.	It	is	a	simple
but	useful	abstraction	that	allows	us	to	be	concrete.	It	captures	the
main	ideas	without	being	tied	too	closely	to	the	detail	of	any
proprietary	designs.


Figure	
6.6	
Example	bus	structure	that	connects	the	CPU	and	main
memory.
bridge	to	the	main	memory.	The	I/O	bridge	translates	the	electrical
signals	of	the	system	bus	into	the	electrical	signals	of	the	memory	bus.
As	we	will	see,	the	I/O	bridge	also	connects	the	system	bus	and	memory
bus	to	an	
I/O	bus
	that	is	shared	by	I/O	devices	such	as	disks	and
graphics	cards.	For	now,	though,	we	will	focus	on	the	memory	bus.
Consider	what	happens	when	the	CPU	performs	a	load	operation	such
as
where	the	contents	of	address	
A
	are	loaded	into	register	
.	Circuitry
on	the	CPU	chip	called	the	
bus	interface
	initiates	a	read	transaction	on
the	bus.	The	read	transaction	consists	of	three	steps.	First,	the	CPU
places	the	address	
A
	on	the	system	bus.	The	I/O	bridge	passes	the
signal	along	to	the	memory	bus	(
Figure	
6.7(a)
).	Next,	the	main
memory	senses	the	address	signal	on	the	memory	bus,	reads	the
address	from	the	memory	bus,	fetches	the	data	from	the	DRAM,	and
writes	the	data	to	the	memory	bus.	The	I/O	bridge	translates	the	memory
bus	signal	into	a	system	bus	signal	and	passes	it	along	to	the	system	bus
(
Figure	
6.7(b)
).	Finally,	the	CPU	senses	the	data	on	the	system	bus,
reads	the	data	from	the	bus,	and	copies	the	data	to	register	
	(
Figure
6.7(c)
).

Conversely,	when	the	CPU	performs	a	store	operation	such	as
Figure	
6.7	
Memory	read	transaction	for	a	load	operation:	
where	the	contents	of	register	
	are	written	to	address	
A
,	the	CPU
initiates	a	write	transaction.	Again,	there	are	three	basic	steps.	First,	the
CPU	places	the	address	on	the	system	bus.	The	memory	reads	the

address	from	the	memory	bus	and	waits	for	the	data	to	arrive	(
Figure
6.8(a)
).	Next,	the	CPU	copies	the	data	in	
	to	the	system	bus
(
Figure	
6.8(b)
).	Finally,	the	main	memory	reads	the	data	from	the
memory	bus	and	stores	the	bits	in	the	DRAM	(
Figure	
6.8(c)
).
6.1.2	
Disk	Storage
Disks
	are	workhorse	storage	devices	that	hold	enormous	amounts	of
data,	on	the	order	of	hundreds	to	thousands	of	gigabytes,	as	opposed	to
the	hundreds	or	thousands	of	megabytes	in	a	RAM-based	memory.
However,	it	takes	on	the	order	of	milliseconds	to	read	information	from	a
disk,	a	hundred	thousand	times	longer	than	from	DRAM	and	a	million
times	longer	than	from	SRAM.

Figure	
6.8	
Memory	write	transaction	for	a	store	operation:	
Disk	Geometry
Disks	are	constructed	from	
platters.
	Each	platter	consists	of	two	sides,	or
surfaces
,	that	are	coated	with	magnetic	recording	material.	A	rotating
spindle
	in	the	center	of	the	platter	spins	the	platter	at	a	fixed	
rotational
rate
,	typically	between	5,400	and	15,000	
revolutions	per	minute	(RPM).
	A
disk	will	typically	contain	one	or	more	of	these	platters	encased	in	a
sealed	container.

Figure	
6.9(a)
	shows	the	geometry	of	a	typical	disk	surface.	Each
surface	consists	of	a	collection	of	concentric	rings	called	
tracks.
	Each
track	is	partitioned	into	a	collection	of	
sectors.
	Each	sector	contains	an
equal	number	of	data	bits	(typically	512	bytes)	encoded	in	the	magnetic
material	on	the	sector.	Sectors	are	separated	by	
gaps
	where	no	data	bits
are	stored.	Gaps	store	formatting	bits	that	identify	sectors.
Figure	
6.9	
Disk	geometry.
A	disk	consists	of	one	or	more	platters	stacked	on	top	of	each	other	and
encased	in	a	sealed	package,	as	shown	in	
Figure	
6.9(b)
.	The	entire
assembly	is	often	referred	to	as	a	
disk	drive
,	although	we	will	usually
refer	to	it	as	simply	a	
disk.
	We	will	sometimes	refer	to	disks	as	
rotating
disks
	to	distinguish	them	from	flash-based	solid	state	disks	(SSDs),
which	have	no	moving	parts.
Disk	manufacturers	describe	the	geometry	of	multiple-platter	drives	in
terms	of	
cylinders
,	where	a	cylinder	is	the	collection	of	tracks	on	all	the
surfaces	that	are	equidistant	from	the	center	of	the	spindle.	For	example,
if	a	drive	has	three	platters	and	six	surfaces,	and	the	tracks	on	each
surface	are	numbered	consistently,	then	cylinder	
k
	is	the	collection	of	the
six	instances	of	track	
k.

Disk	Capacity
The	maximum	number	of	bits	that	can	be	recorded	by	a	disk	is	known	as
its	
maximum	capacity
,	or	simply	
capacity.
	Disk	capacity	is	determined	by
the	following	technology	factors:
Recording	density
	(bits/in).	The	number	of	bits	that	can	be	squeezed
into	a	1-inch	segment	of	a	track.
Track	density
	(tracks/in).	The	number	of	tracks	that	can	be	squeezed
into	a	l-inch	segment	of	the	radius	extending	from	the	center	of	the
platter.
Areal	density
	(bits/in
).	The	product	of	the	recording	density	and	the
track	density.
Disk	manufacturers	work	tirelessly	to	increase	areal	density	(and	thus
capacity),	and	this	is	doubling	every	couple	of	years.	The	original	disks,
designed	in	an	age	of	low	areal	density,	partitioned	every	track	into	the
same	number	of	sectors,	which	was	determined	by	the	number	of	sectors
that	could	be	recorded	on	the	innermost	track.	To	maintain	a	fixed
number	of	sectors	per	track,	the	sectors	were	spaced	farther	apart	on	the
outer	tracks.	This	was	a	reasonable	approach
Aside	
How	much	is	a	gigabyte?
Unfortunately,	the	meanings	of	prefixes	such	as	kilo	(K),	mega
(M),	giga	(G),	and	tera	(T)	depend	on	the	context.	For	measures
that	relate	to	the	capacity	of	DRAMs	and	SRAMs,	typically	K	=	2
,
M	=	2
,	G	=	2
,	and	T	=	2
.	For	measures	related	to	the	capacity
2
10
20
30
40
3

of	I/O	devices	such	as	disks	and	networks,	typically	K	=	10
,	M	=
10
,	G	=	10
,	and	T	=	10
.	Rates	and	throughputs	usually	use
these	prefix	values	as	well.
Fortunately,	for	the	back-of-the-envelope	estimates	that	we
typically	rely	on,	either	assumption	works	fine	in	practice.	For
example,	the	relative	difference	between	2
	and	10
	is	not	that
large:	(2
	−	10
)/10
	≈	7%.	Similarly,	(2
	−	10
)/10
	≈	10%.
when	areal	densities	were	relatively	low.	However,	as	areal	densities
increased,	the	gaps	between	sectors	(where	no	data	bits	were	stored)
became	unacceptably	large.	Thus,	modern	high-capacity	disks	use	a
technique	known	as	
multiple	zone	recording
,	where	the	set	of	cylinders	is
partitioned	into	disjoint	subsets	known	as	
recording	zones.
	Each	zone
consists	of	a	contiguous	collection	of	cylinders.	Each	track	in	each
cylinder	in	a	zone	has	the	same	number	of	sectors,	which	is	determined
by	the	number	of	sectors	that	can	be	packed	into	the	innermost	track	of
the	zone.
The	capacity	of	a	disk	is	given	by	the	following	formula:
For	example,	suppose	we	have	a	disk	with	five	platters,	512	bytes	per
sector,	20,000	tracks	per	surface,	and	an	average	of	300	sectors	per
track.	Then	the	capacity	of	the	disk	is
Notice	that	manufacturers	express	disk	capacity	in	units	of	gigabytes
(GB)	or	terabytes	(TB),	where	1	GB	=	10
	bytes	and	1	TB	=	10
	bytes.
3
6
9
12
30
9
30
9
9
40
12
12
Capacity
=
#
 
bytes
sector
×
verage
 
#
 
sectors
track
×
#
 
tracks
surface
×
#
 
surfaces
Capacity
=
512
 
bytes
sector
×
300
 
sectors
track
×
20
,
000
 
tracks
surface
×
2
 
surfaces
9
12

Practice	Problem	
6.2	
(solution	page	
661
)
What	is	the	capacity	of	a	disk	with	2	platters,	10,000	cylinders,	an
average	of	400	sectors	per	track,	and	512	bytes	per	sector?
Disk	Operation
Disks	read	and	write	bits	stored	on	the	magnetic	surface	using	a
read/write	head
	connected	to	the	end	of	an	
actuator	arm
,	as	shown	in
Figure	
6.10(a)
.	By	moving
Figure	
6.10	
Disk	dynamics.
the	arm	back	and	forth	along	its	radial	axis,	the	drive	can	position	the
head	over	any	track	on	the	surface.	This	mechanical	motion	is	known	as
a	
seek.
	Once	the	head	is	positioned	over	the	desired	track,	then,	as	each
bit	on	the	track	passes	underneath,	the	head	can	either	sense	the	value
of	the	bit	(read	the	bit)	or	alter	the	value	of	the	bit	(write	the	bit).	Disks
with	multiple	platters	have	a	separate	read/write	head	for	each	surface,
as	shown	in	
Figure	
6.10(b)
.	The	heads	are	lined	up	vertically	and

move	in	unison.	At	any	point	in	time,	all	heads	are	positioned	on	the
same	cylinder.
The	read/write	head	at	the	end	of	the	arm	flies	(literally)	on	a	thin	cushion
of	air	over	the	disk	surface	at	a	height	of	about	0.1	microns	and	a	speed
of	about	80	km/h.	This	is	analogous	to	placing	a	skyscraper	on	its	side
and	flying	it	around	the	world	at	a	height	of	2.5	cm	(1	inch)	above	the
ground,	with	each	orbit	of	the	earth	taking	only	8	seconds!	At	these
tolerances,	a	tiny	piece	of	dust	on	the	surface	is	like	a	huge	boulder.	If
the	head	were	to	strike	one	of	these	boulders,	the	head	would	cease
flying	and	crash	into	the	surface	(a	so-called	head	crash).	For	this
reason,	disks	are	always	sealed	in	airtight	packages.
Disks	read	and	write	data	in	sector-size	blocks.	The	
access	time
	for	a
sector	has	three	main	components:	
seek	time,	rotational	latency
,	and
transfer	time:
Seek	time.	
To	read	the	contents	of	some	target	sector,	the	arm	first
positions	the	head	over	the	track	that	contains	the	target	sector.	The
time	required	to	move	the	arm	is	called	the	
seek	time.
	The	seek	time,
T
,	depends	on	the	previous	position	of	the	head	and	the	speed	that
the	arm	moves	across	the	surface.	The	average	seek	time	in	modern
drives,	
T
,	measured	by	taking	the	mean	of	several	thousand
seeks	to	random	sectors,	is	typically	on	the	order	of	3	to	9	ms.	The
maximum	time	for	a	single	seek,	
T
,	can	be	as	high	as	20	ms.
Rotational	latency.	
Once	the	head	is	in	position	over	the	track,	the
drive	waits	for	the	first	bit	of	the	target	sector	to	pass	under	the	head.
The	performance	of	this	step	depends	on	both	the	position	of	the
surface	when	the	head	arrives	at	the	target	track	and	the	rotational
seek
avg	seek
max	seek

speed	of	the	disk.	In	the	worst	case,	the	head	just	misses	the	target
sector	and	waits	for	the	disk	to	make	a	full	rotation.	Thus,	the
maximum	rotational	latency,	in	seconds,	is	given	by
The	average	rotational	latency,	
T
,	is	simply	half	of	
T
.
Transfer	time.	
When	the	first	bit	of	the	target	sector	is	under	the
head,	the	drive	can	begin	to	read	or	write	the	contents	of	the	sector.
The	transfer	time	for	one	sector	depends	on	the	rotational	speed	and
the	number	of	sectors	per	track.	Thus,	we	can	roughly	estimate	the
average	transfer	time	for	one	sector	in	seconds	as
We	can	estimate	the	average	time	to	access	the	contents	of	a	disk	sector
as	the	sum	of	the	average	seek	time,	the	average	rotational	latency,	and
the	average	transfer	time.	For	example,	consider	a	disk	with	the	following
parameters:
Parameter
Value
Rotational	rate
7,200	RPM
T
9	ms
Average	number	of	sectors/track
400
For	this	disk,	the	average	rotational	latency	(in	ms)	is
T
max
 
rotation
=
1
RPM
×
60
 
secs
1
 
min
avg	rotation
max	rotation
T
avg
 
transfer
=
1
RPM
×
1
(
average
 
#
 
sectors/track
)
×
60
 
secs
1
 
min
avg	seek
T
avg
 
rotation
=
1
/
2
×
T
max
 
rotation
=
1
/
2
×
(
60
 
sec
s
/
7
,
200
 
RPM
)
×
1
,
000
 
ms/sec

The	average	transfer	time	is
Putting	it	all	together,	the	total	estimated	access	time	is
This	example	illustrates	some	important	points:
The	time	to	access	the	512	bytes	in	a	disk	sector	is	dominated	by	the
seek	time	and	the	rotational	latency.	Accessing	the	first	byte	in	the
sector	takes	a	long	time,	but	the	remaining	bytes	are	essentially	free.
Since	the	seek	time	and	rotational	latency	are	roughly	the	same,	twice
the	seek	time	is	a	simple	and	reasonable	rule	for	estimating	disk
access	time.
The	access	time	for	a	64-bit	word	stored	in	SRAM	is	roughly	4	ns,	and
60	ns	for	DRAM.	Thus,	the	time	to	read	a	512-byte	sector-size	block
from	memory	is	roughly	256	ns	for	SRAM	and	4,000	ns	for	DRAM.
The	disk	access	time,	roughly	10	ms,	is	about	40,000	times	greater
than	SRAM,	and	about	2,500	times	greater	than	DRAM.
Practice	Problem	
6.3	
(solution	page	
661
)
Estimate	the	average	time	(in	ms)	to	access	a	sector	on	the
following	disk:
Parameter
Value
Rotational	rate
15,000	RPM
T
avg
 
rotation
=
60
/
7
,
200
 
RPM
×
1
/
400
 
sectors/track
×
1
,
000
 
ms/sec
≈
0.02
 
ms
T
access
=
T
avg
 
seek
+
T
avg
 
rotation
+
T
avg
 
transfer
=
9
 
ms
+
4
 
ms
+
0.02
 
ms
=
13.02

T
8	ms
Average	number	of	sectors/track
500
Logical	Disk	Blocks
As	we	have	seen,	modern	disks	have	complex	geometries,	with	multiple
surfaces	and	different	recording	zones	on	those	surfaces.	To	hide	this
complexity	from	the	operating	system,	modern	disks	present	a	simpler
view	of	their	geometry	as	a	sequence	of	
B
	sector-size	
logical	blocks
,
numbered	0,	1,	...,	
B
	−	1.	A	small	hardware/firmware	device	in	the	disk
package,	called	the	
disk	controller
,	maintains	the	mapping	between
logical	block	numbers	and	actual	(physical)	disk	sectors.
When	the	operating	system	wants	to	perform	an	I/O	operation	such	as
reading	a	disk	sector	into	main	memory,	it	sends	a	command	to	the	disk
controller	asking	it	to	read	a	particular	logical	block	number.	Firmware	on
the	controller	performs	a	fast	table	lookup	that	translates	the	logical	block
number	into	a	
(surface,	track,	sector)
	triple	that	uniquely	identifies	the
corresponding	physical	sector.	Hardware	on	the	controller	interprets	this
triple	to	move	the	heads	to	the	appropriate	cylinder,	waits	for	the	sector	to
pass	under	the	head,	gathers	up	the	bits	sensed	by	the	head	into	a	small
memory	buffer	on	the	controller,	and	copies	them	into	main	memory.
Practice	Problem	
6.4	
(solution	page	
661
)
Suppose	that	a	1	MB	file	consisting	of	512-byte	logical	blocks	is
stored	on	a	disk	drive	with	the	following	characteristics:
abg	seek

Aside	
Formatted	disk	capacity
Before	a	disk	can	be	used	to	store	data,	it	must	be
formatted
	by	the	disk	controller.	This	involves	filling	in	the
gaps	between	sectors	with	information	that	identifies	the
sectors,	identifying	any	cylinders	with	surface	defects	and
taking	them	out	of	action,	and	setting	aside	a	set	of
cylinders	in	each	zone	as	spares	that	can	be	called	into
action	if	one	or	more	cylinders	in	the	zone	goes	bad	during
the	lifetime	of	the	disk.	The	
formatted	capacity
	quoted	by
disk	manufacturers	is	less	than	the	maximum	capacity
because	of	the	existence	of	these	spare	cylinders.
Parameter
Value
Rotational	rate
10,000	RPM
T
5	ms
Average	number	of	sectors/track
1,000
Surfaces
4
Sector	size
512	bytes
For	each	case	below,	suppose	that	a	program	reads	the	logical
blocks	of	the	file	sequentially,	one	after	the	other,	and	that	the	time
to	position	the	head	over	the	first	block	is	
T
	+	
T
.
A
.	
Best	case:	
Estimate	the	optimal	time	(in	ms)	required	to
read	the	file	given	the	best	possible	mapping	of	logical
blocks	to	disk	sectors	(i.e.,	sequential).
B
.	
Random	case:	
Estimate	the	time	(in	ms)	required	to	read
the	file	if	blocks	are	mapped	randomly	to	disk	sectors.
avg	seek
avg	seek
avg	rotation

Connecting	I/O	Devices
Input/output	(I/O)	devices	such	as	graphics	cards,	monitors,	mice,
keyboards,	and	disks	are	connected	to	the	CPU	and	main	memory	using
an	
I/O	bus.
	Unlike	the	system	bus	and	memory	buses,	which	are	CPU-
specific,	I/O	buses	are	designed	to	be	independent	of	the	underlying
CPU.	
Figure	
6.11
	shows	a	representative	I/O	bus	structure	that
connects	the	CPU,	main	memory,	and	I/O	devices.
Although	the	I/O	bus	is	slower	than	the	system	and	memory	buses,	it	can
accommodate	a	wide	variety	of	third-party	I/O	devices.	For	example,	the
bus	in	
Figure	
6.11
	has	three	different	types	of	devices	attached	to	it.
A	
Universal	Serial	Bus	(USB)
	controller	is	a	conduit	for	devices
attached	to	a	USB	bus,	which	is	a	wildly	popular	standard	for
connecting	a	variety	of	peripheral	I/O	devices,	including	keyboards,
mice,	modems,	digital	cameras,	game	controllers,	printers,	external
disk	drives,	and	solid	state	disks.	USB	3.0	buses	have	a	maximum
bandwidth	of	625	MB/s.	USB	3.1	buses	have	a	maximum	bandwidth
of	1,250	MB/s.

Figure	
6.11	
Example	bus	structure	that	connects	the	CPU,	main
memory,	and	I/O	devices.
A	
graphics	card
	(or	
adapter
)	contains	hardware	and	software	logic
that	is	responsible	for	painting	the	pixels	on	the	display	monitor	on
behalf	of	the	CPU.
A	
host	bus	adapter
	that	connects	one	or	more	disks	to	the	I/O	bus
using	a	communication	protocol	defined	by	a	particular	
host	bus
interface.
	The	two	most	popular	such	interfaces	for	disks	are	
SCSI
(pronounced	"scuzzy")	and	
SATA
	(pronounced	"sat-uh").	SCSI	disks
are	typically	faster	and	more	expensive	than	SATA	drives.	A	SCSI
host	bus	adapter	(often	called	a	
SCSI	controller)
	can	support	multiple
disk	drives,	as	opposed	to	SATA	adapters,	which	can	only	support
one	drive.

Additional	devices	such	as	
network	adapters
	can	be	attached	to	the	I/O
bus	by	plugging	the	adapter	into	empty	
expansion	slots
	on	the
motherboard	that	provide	a	direct	electrical	connection	to	the	bus.
Accessing	Disks
While	a	detailed	description	of	how	I/O	devices	work	and	how	they	are
programmed	is	outside	our	scope	here,	we	can	give	you	a	general	idea.
For	example,	
Figure	
6.12
	summarizes	the	steps	that	take	place	when
a	CPU	reads	data	from	a	disk.
Aside	
Advances	in	I/O	bus	designs
The	I/O	bus	in	
Figure	
6.11
	is	a	simple	abstraction	that	allows
us	to	be	concrete,	without	being	tied	too	closely	to	the	details	of
any	specific	system.	It	is	based	on	the	
peripheral	component
interconnect	(PCI)
	bus,	which	was	popular	until	around	2010.	In
the	PCI	model,	each	device	in	the	system	shares	the	bus,	and
only	one	device	at	a	time	can	access	these	wires.	In	modern
systems,	the	shared	PCI	bus	has	been	replaced	by	a	
PCI	express
(PCIe)	bus,	which	is	a	set	of	high-speed	serial,	point-to-point	links
connected	by	switches,	akin	to	the	switched	Ethernets	that	you
will	learn	about	in	
Chapter	
11
.	A	PCIe	bus,	with	a	maximum
throughput	of	16	GB/s,	is	an	order	of	magnitude	faster	than	a	PCI
bus,	which	has	a	maximum	throughput	of	533	MB/s.	Except	for
measured	I/O	performance,	the	differences	between	the	different
bus	designs	are	not	visible	to	application	programs,	so	we	will	use
the	simple	shared	bus	abstraction	throughout	the	text.

The	CPU	issues	commands	to	I/O	devices	using	a	technique	called
memory-mapped	I/O
	(
Figure	
6.12(a)
).	In	a	system	with	memory-
mapped	I/O,	a	block	of	addresses	in	the	address	space	is	reserved	for
communicating	with	I/O	devices.	Each	of	these	addresses	is	known	as	an
I/O	port.
	Each	device	is	associated	with	(or	mapped	to)	one	or	more	ports
when	it	is	attached	to	the	bus.
As	a	simple	example,	suppose	that	the	disk	controller	is	mapped	to	port
.	Then	the	CPU	might	initiate	a	disk	read	by	executing	three	store
instructions	to	address	
:	The	first	of	these	instructions	sends	a
command	word	that	tells	the	disk	to	initiate	a	read,	along	with	other
parameters	such	as	whether	to	interrupt	the	CPU	when	the	read	is
finished.	(We	will	discuss	interrupts	in	
Section	
8.1
.)	The	second
instruction	indicates	the	logical	block	number	that	should	be	read.	The
third	instruction	indicates	the	main	memory	address	where	the	contents
of	the	disk	sector	should	be	stored.
After	it	issues	the	request,	the	CPU	will	typically	do	other	work	while	the
disk	is	performing	the	read.	Recall	that	a	1	GHz	processor	with	a	1	ns
clock	cycle	can	potentially	execute	16	million	instructions	in	the	16	ms	it
takes	to	read	the	disk.	Simply	waiting	and	doing	nothing	while	the
transfer	is	taking	place	would	be	enormously	wasteful.
After	the	disk	controller	receives	the	read	command	from	the	CPU,	it
translates	the	logical	block	number	to	a	sector	address,	reads	the
contents	of	the	sector,	and	transfers	the	contents	directly	to	main
memory,	without	any	intervention	from	the	CPU	(
Figure	
6.12(b)
).	This
process,	whereby	a	device	performs	a	read	or	write	bus	transaction	on	its

own,	without	any	involvement	of	the	CPU,	is	known	as	
direct	memory
access
	(DMA).	The	transfer	of	data	is	known	as	a	
DMA	transfer.
After	the	DMA	transfer	is	complete	and	the	contents	of	the	disk	sector	are
safely	stored	in	main	memory,	the	disk	controller	notifies	the	CPU	by
sending	an	interrupt	signal	to	the	CPU	(
Figure	
6.12(c)
).	The	basic
idea	is	that	an	interrupt	signals	an	external	pin	on	the	CPU	chip.	This
causes	the	CPU	to	stop	what	it	is	currently	working	on	and	jump	to	an
operating	system	routine.	The	routine	records	the	fact	that	the	I/O	has
finished	and	then	returns	control	to	the	point	where	the	CPU	was
interrupted.



Figure	
6.12	
Reading	a	disk	sector.
Aside	
Characteristics	of	a	commercial
disk	drive
Disk	manufacturers	publish	a	lot	of	useful	high-level	technical
information	on	their	Web	sites.	For	example,	the	Seagate	Web	site
contains	the	following	information	(and	much	more!)	about	one	of
their	popular	drives,	the	Barracuda	7400.	(
Seagate.com
)
Geometry	characteristic
Value
Surface	diameter
3.5	in
Formatted	capacity
3	TB
Platters
3
Surfaces
6
Logical	blocks
5,860,533,168
Logical	block	size
512	bytes
Rotational	rate
7,200	RPM
Average	rotational	latency
4.16	ms
Average	seek	time
8.5	ms
Track-to-track	seek	time
1.0	ms
Average	transfer	rate
156	MB/s

Maximum	sustained	transfer	rate
210	MB/s
Figure	
6.13	
Solid	state	disk	(SSD).
6.1.3	
Solid	State	Disks
A	solid	state	disk	(SSD)	is	a	storage	technology,	based	on	flash	memory
(
Section	
6.1.1
),	that	in	some	situations	is	an	attractive	alternative	to
the	conventional	rotating	disk.	
Figure	
6.13
	shows	the	basic	idea.	An
SSD	package	plugs	into	a	standard	disk	slot	on	the	I/O	bus	(typically
USB	or	SATA)	and	behaves	like	any	other	disk,	processing	requests	from
the	CPU	to	read	and	write	logical	disk	blocks.	An	SSD	package	consists
of	one	or	more	flash	memory	chips,	which	replace	the	mechanical	drive
in	a	conventional	rotating	disk,	and	a	
flash	translation	layer
,	which	is	a
hardware/firmware	device	that	plays	the	same	role	as	a	disk	controller,
translating	requests	for	logical	blocks	into	accesses	of	the	underlying
physical	device.
Figure	
6.14
	shows	the	performance	characteristics	of	a	typical	SSD.
Notice	that	reading	from	SSDs	is	faster	than	writing.	The	difference
between	random	reading	and	writing	performance	is	caused	by	a

fundamental	property	of	the	underlying	flash	memory.	As	shown	in
Figure	
6.13
,	a	flash	memory	consists	of	a	sequence	of	
B	blocks
,
where	each	block	consists	of	
P
	pages.	Typically,	pages	are	512	bytes	to
4	KB	in	size,	and	a	block	consists	of	32−128	pages,	with	total	block	sizes
ranging	from	16
Reads
Writes
Sequential	read	throughput
550	MB/s
Sequential	write	throughput
470	MB/s
Random	read	throughput
(IOPS)
89,000
IOPS
Random	write	throughput
(IOPS)
74,000
IOPS
Random	read	throughput
(MB/s)
365	MB/s
Random	write	throughput
(MB/s)
303	MB/s
Avg.	sequential	read	access
time
50	
μ
s
Avg.	sequential	write	access
time
60	
μ
s
Figure	
6.14	
Performance	characteristics	of	a	commercial	solid	state
disk.
Source:	
Intel	SSD	730	product	specification	[
53
].	
IOPS
	is	I/O	operations	per	second.	Throughput	numbers	are	based	on
reads	and	writes	of	4	KB	blocks.	(Intel	SSD	730	product	specification.	Intel	Corporation.	52.)
KB	to	512	KB.	Data	are	read	and	written	in	units	of	pages.	A	page	can	be
written	only	after	the	entire	block	to	which	it	belongs	has	been	
erased
(typically,	this	means	that	all	bits	in	the	block	are	set	to	1).	However,	once
a	block	is	erased,	each	page	in	the	block	can	be	written	once	with	no
further	erasing.	A	block	wears	out	after	roughly	100,000	repeated	writes.
Once	a	block	wears	out,	it	can	no	longer	be	used.

Random	writes	are	slower	for	two	reasons.	First,	erasing	a	block	takes	a
relatively	long	time,	on	the	order	of	1	ms,	which	is	more	than	an	order	of
magnitude	longer	than	it	takes	to	access	a	page.	Second,	if	a	write
operation	attempts	to	modify	a	page	
p
	that	contains	existing	data	(i.e.,	not
all	ones),	then	any	pages	in	the	same	block	with	useful	data	must	be
copied	to	a	new	(erased)	block	before	the	write	to	page	
p
	can	occur.
Manufacturers	have	developed	sophisticated	logic	in	the	flash	translation
layer	that	attempts	to	amortize	the	high	cost	of	erasing	blocks	and	to
minimize	the	number	of	internal	copies	on	writes,	but	it	is	unlikely	that
random	writing	will	ever	perform	as	well	as	reading.
SSDs	have	a	number	of	advantages	over	rotating	disks.	They	are	built	of
semiconductor	memory,	with	no	moving	parts,	and	thus	have	much	faster
random	access	times	than	rotating	disks,	use	less	power,	and	are	more
rugged.	However,	there	are	some	disadvantages.	First,	because	flash
blocks	wear	out	after	repeated	writes,	SSDs	have	the	potential	to	wear
out	as	well.	
Wear-leveling
	logic	in	the	flash	translation	layer	attempts	to
maximize	the	lifetime	of	each	block	by	spreading	erasures	evenly	across
all	blocks.	In	practice,	the	wear-leveling	logic	is	so	good	that	it	takes
many	years	for	SSDs	to	wear	out	(see	
Practice	Problem	
6.5
).
Second,	SSDs	are	about	30	times	more	expensive	per	byte	than	rotating
disks,	and	thus	the	typical	storage	capacities	are	significantly	less	than
rotating	disks.	However,	SSD	prices	are	decreasing	rapidly	as	they
become	more	popular,	and	the	gap	between	the	two	is	decreasing.
SSDs	have	completely	replaced	rotating	disks	in	portable	music	devices,
are	popular	as	disk	replacements	in	laptops,	and	have	even	begun	to
appear	in	desktops	and	servers.	While	rotating	disks	are	here	to	stay,	it	is
clear	that	SSDs	are	an	important	alternative.

Practice	Problem	
6.5	
(solution	page	
662
)
As	we	have	seen,	a	potential	drawback	of	SSDs	is	that	the
underlying	flash	memory	can	wear	out.	For	example,	for	the	SSD
in	
Figure	
6.14
,	Intel	guarantees	about	
128	petabytes	(128	×
10
	bytes)	of	writes	before	the	drive	wears	out.	Given	this
assumption,	estimate	the	lifetime	(in	years)	of	this	SSD	for	the
following	workloads:
A
.	
Worst	case	for	sequential	writes:
	The	SSD	is	written	to
continuously	at	a	rate	of	470	MB/s	(the	average	sequential
write	throughput	of	the	device).
B
.	
Worst	case	for	random	writes:
	The	SSD	is	written	to
continuously	at	a	rate	of	303	MB/s	(the	average	random
write	throughput	of	the	device).
C
.	
Average	case:
	The	SSD	is	written	to	at	a	rate	of	20	GB/day
(the	average	daily	write	rate	assumed	by	some	computer
manufacturers	in	their	mobile	computer	workload
simulations).
6.1.4	
Storage	Technology	Trends
There	are	several	important	concepts	to	take	away	from	our	discussion	of
storage	technologies.
Different	storage	technologies	have	different	price	and	performance
trade-offs.
	SRAM	is	somewhat	faster	than	DRAM,	and	DRAM	is	much
faster	than	disk.	On	the	other	hand,	fast	storage	is	always	more
expensive	than	slower	storage.	SRAM	costs	more	per	byte	than	DRAM.
15

DRAM	costs	much	more	than	disk.	SSDs	split	the	difference	between
DRAM	and	rotating	disk.
The	price	and	performance	properties	of	different	storage	technologies
are	changing	at	dramatically	different	rates.
	
Figure	
6.15
	summarizes
the	price	and	performance	properties	of	storage	technologies	since	1985,
shortly	after	the	first	PCs	were	introduced.	The	numbers	were	culled	from
back	issues	of	trade	magazines	and	the	Web.	Although	they	were
collected	in	an	informal	survey,	the	numbers	reveal	some	interesting
trends.
Since	1985,	both	the	cost	and	performance	of	SRAM	technology	have
improved	at	roughly	the	same	rate.	Access	times	and	cost	per	megabyte
have	decreased	by	a	factor	of	about	100	(
Figure	
6.15(a)
).	However,
the	trends	for	DRAM	and	disk	are	much	more	dramatic	and	divergent.
While	the	cost	per	megabyte	of	DRAM	has	decreased	by	a	factor	of
44,000	(more	than	four	orders	of	magnitude!),	DRAM	access	times	have
decreased	by	only	a	factor	of	10	(
Figure	
6.15(b)
).	Disk	technology	has
followed	the	same	trend	as	DRAM	and	in	even	more	dramatic	fashion.
While	the	cost	of	a	megabyte	of	disk	storage	has	plummeted	by	a	factor
of	more	than	3,000,000	(more	than	six	orders	of	magnitude!)	since	1980,
access	times	have	improved	much	more	slowly,	by	only	a	factor	of	25
(
Figure	
6.15(c)
).	These	startling	long-term	trends	highlight	a	basic
truth	of	memory	and	disk	technology:	it	is	much	easier	to	increase	density
(and	thereby	reduce	cost)	than	to	decrease	access	time.
DRAM	and	disk	performance	are	lagging	behind	CPU	performance.
	As
we	see	in	
Figure	
6.15(d)
,	CPU	cycle	times	improved	by	a	factor	of	500
between	1985	and	2010.	If	we	look	at	the	
effective	cycle	time
—which	we
define	to	be	the	cycle	time	of	an	individual	CPU	(processor)	divided	by

the	number	of	its	processor	cores—then	the	improvement	between	1985
and	2010	is	even	greater,	a	factor	of	2,000.
Metric
1985
1990
1995
2000
2005
2010
2015
2015:1985
$/MB
2,900
320
256
100
75
60
25
116
Access	(ns)
150
35
15
3
2
1.5
1.3
115
(a)	SRAM	trends
Metric
1985
1990
1995
2000
2005
2010
2015
2015:1985
$/MB
880
100
30
1
0.1
0.06
0.02
44,000
Access	(ns)
200
100
70
60
50
40
20
10
Typical	size
(MB)
0.256
4
16
64
2,000
8,000
16,000
62,500
(b)	DRAM	trends
Metric
1985
1990
1995
2000
2005
2010
2015
2015:1985
$/GB
100,000
8,000
300
10
5
0.3
0.03
3,333,333
Min.	seek	time
(ms)
75
28
10
8
5
3
3
25
Typical	size
(GB)
0.01
0.16
1
20
160
1,500
3,000
300,000
(c)	Rotating	disk	trends
Metric
1985
1990
1995
2000
2003
2005
2010
2015
2015:1985

Intel
CPU
80286
80386
Pent.
P-III
Pent.
4
Core
2
Core
i7	(n)
Core
i7	(h)
—
Clock
rate
(MHz)
6
20
150
600
3,300
2,000
2,500
3,000
500
Cycle
time
(ns)
166
50
6
1.6
0.3
0.5
0.4
0.33
500
Cores
1
1
1
1
1
2
4
4
4
Effective
cycle
time
(ns)
166
50
6
1.6
0.30
0.25
0.10
0.08
2,075
(d)	CPU	trends
Figure	
6.15	
Storage	and	processing	technology	trends.
The	Core	i7	circa	201	0	uses	the	Nehalem	processor	core.	The	Core	i7
circa	201	5	uses	the	Haswell	core.
The	split	in	the	CPU	performance	curve	around	2003	reflects	the
introduction	of	multi-core	processors	(see	aside	on	page	605).	After	this
split,	cycle	times	of	individual	cores	actually	increased	a	bit	before
starting	to	decrease	again,	albeit	at	a	slower	rate	than	before.
Note	that	while	SRAM	performance	lags,	it	is	roughly	keeping	up.
However,	the	gap	between	DRAM	and	disk	performance	and	CPU
performance	is	actually	widening.	Until	the	advent	of	multi-core
processors	around	2003,	this	performance	gap	was	a	function	of	latency,

with	DRAM	and	disk	access	times	decreasing	more	slowly	than	the	cycle
time	of	an	individual	processor.	However,	with	the	introduction	of	multiple
cores,	this	performance	gap	is	increasingly	a	function	of
Figure	
6.16	
The	gap	between	disk,	DRAM,	and	CPU	speeds.
throughput,	with	multiple	processor	cores	issuing	requests	to	the	DRAM
and	disk	in	parallel.
The	various	trends	are	shown	quite	clearly	in	
Figure	
6.16
,	which	plots
the	access	and	cycle	times	from	
Figure	
6.15
	on	a	semi-log	scale.
As	we	will	see	in	
Section	
6.4
,	modern	computers	make	heavy	use	of
SRAM-based	caches	to	try	to	bridge	the	processor-memory	gap.	This
approach	works	because	of	a	fundamental	property	of	application
programs	known	as	
locality
,	which	we	discuss	next.
Practice	Problem	
6.6	
(solution	page	
662
)

Using	the	data	from	the	years	2005	to	2015	in	
Figure	
6.15(c)
,
estimate	the	year	when	you	will	be	able	to	buy	a	petabyte	(10
bytes)	of	rotating	disk	storage	for	$500.	Assume	actual	dollars	(no
inflation).
15

6.2	
Locality
Well-written	computer	programs	tend	to	exhibit	good	
locality.
	That	is,	they
tend	to	reference	data	items	that	are	near	other	recently	referenced	data
items	or	that	were	recently	referenced	themselves.	This	tendency,	known
as	the	
principle	of	locality
,	is	an	enduring	concept	that	has	enormous
impact	on	the	design	and	performance	of	hardware	and	software
systems.
Locality	is	typically	described	as	having	two	distinct	forms:	
temporal
locality
	and	
spatial	locality.
	In	a	program	with	good	temporal	locality,	a
memory	location	that	is	referenced	once	is	likely	to	be	referenced	again
multiple	times	in	the	near	future.	In	a	program	with	good	spatial	locality,	if
a	memory	location	is	referenced
Aside	
When	cycle	time	stood	still:	The
advent	of	multi-core	processors
The	history	of	computers	is	marked	by	some	singular	events	that
caused	profound	changes	in	the	industry	and	the	world.
Interestingly,	these	inflection	points	tend	to	occur	about	once	per
decade:	the	development	of	Fortran	in	the	1950s,	the	introduction
of	the	IBM	360	in	the	early	1960s,	the	dawn	of	the	Internet	(then
called	ARPANET)	in	the	early	1970s,	the	introduction	of	the	IBM
PC	in	the	early	1980s,	and	the	creation	of	the	World	Wide	Web	in
the	early	1990s.

The	most	recent	such	event	occurred	early	in	the	21st	century,
when	computer	manufacturers	ran	headlong	into	the	so-called
power	wall,	discovering	that	they	could	no	longer	increase	CPU
clock	frequencies	as	quickly	because	the	chips	would	then
consume	too	much	power.	The	solution	was	to	improve
performance	by	replacing	a	single	large	processor	with	multiple
smaller	processor	
cores
,	each	a	complete	processor	capable	of
executing	programs	independently	and	in	parallel	with	the	other
cores.	This	
multi-core
	approach	works	in	part	because	the	power
consumed	by	a	processor	is	proportional	to	
P
	=	
fCV
,	where	
f
	is
the	clock	frequency,	
C
	is	the	capacitance,	and	
V
	is	the	voltage.
The	capacitance	
C
	is	roughly	proportional	to	the	area,	so	the
power	drawn	by	multiple	cores	can	be	held	constant	as	long	as
the	total	area	of	the	cores	is	constant.	As	long	as	feature	sizes
continue	to	shrink	at	the	exponential	Moore's	Law	rate,	the
number	of	cores	in	each	processor,	and	thus	its	effective
performance,	will	continue	to	increase.
From	this	point	forward,	computers	will	get	faster	not	because	the
clock	frequency	increases	but	because	the	number	of	cores	in
each	processor	increases,	and	because	architectural	innovations
increase	the	efficiency	of	programs	running	on	those	cores.	We
can	see	this	trend	clearly	in	
Figure	
6.16
.	CPU	cycle	time
reached	its	lowest	point	in	2003	and	then	actually	started	to	rise
before	leveling	off	and	starting	to	decline	again	at	a	slower	rate
than	before.	However,	because	of	the	advent	of	multi-core
processors	(dual-core	in	2004	and	quad-core	in	2007),	the
effective	cycle	time	continues	to	decrease	at	close	to	its	previous
rate.
2

once,	then	the	program	is	likely	to	reference	a	nearby	memory	location	in
the	near	future.
Programmers	should	understand	the	principle	of	locality	because,	in
general,	
programs	with	good	locality	run	faster	than	programs	with	poor
locality.
	All	levels	of	modern	computer	systems,	from	the	hardware,	to	the
operating	system,	to	application	programs,	are	designed	to	exploit
locality.	At	the	hardware	level,	the	principle	of	locality	allows	computer
designers	to	speed	up	main	memory	accesses	by	introducing	small	fast
memories	known	as	
cache	memories
	that	hold	blocks	of	the	most
recently	referenced	instructions	and	data	items.	At	the	operating	system
level,	the	principle	of	locality	allows	the	system	to	use	the	main	memory
as	a	cache	of	the	most	recently	referenced	chunks	of	the	virtual	address
space.	Similarly,	the	operating	system	uses	main	memory	to	cache	the
most	recently	used	disk	blocks	in	the	disk	file	system.	The	principle	of
locality	also	plays	a	crucial	role	in	the	design	of	application	programs.	For
example,	Web	browsers	exploit	temporal	locality	by	caching	recently
referenced	documents	on	a	local	disk.	High-volume	Web	servers	hold
recently	requested	documents	in	front-end	disk	caches	that	satisfy
requests	for	these	documents	without	requiring	any	intervention	from	the
server.

(a)
Address
0
4
8
12
16
20
24
28
Contents
v
v
v
v
v
v
v
v
Access	order
1
2
3
4
5
6
7
8
(b)
Figure	
6.17	
(a)	A	function	with	good	locality,	(b)	Reference	pattern
for	vector	
	(
N
	=	8).
Notice	how	the	vector	elements	are	accessed	in	the	same	order	that	they
are	stored	in	memory.
6.2.1	
Locality	of	References	to
Program	Data
Consider	the	simple	function	in	
Figure	
6.17(a)
	that	sums	the	elements
of	a	vector.	Does	this	function	have	good	locality?	To	answer	this
question,	we	look	at	the	reference	pattern	for	each	variable.	In	this
example,	the	
	variable	is	referenced	once	in	each	loop	iteration,	and
thus	there	is	good	temporal	locality	with	respect	to	
.	On	the	other
hand,	since	
	is	a	scalar,	there	is	no	spatial	locality	with	respect	to	
.
0
1
2
3
4
5
6
7

As	we	see	in	
Figure	
6.17(b)
,	the	elements	of	vector	
	are	read
sequentially,	one	after	the	other,	in	the	order	they	are	stored	in	memory
(we	assume	for	convenience	that	the	array	starts	at	address	0).	Thus,
with	respect	to	variable	v,	the	function	has	good	spatial	locality	but	poor
temporal	locality	since	each	vector	element	is	accessed	exactly	once.
Since	the	function	has	either	good	spatial	or	temporal	locality	with
respect	to	each	variable	in	the	loop	body,	we	can	conclude	that	the
	function	enjoys	good	locality.
A	function	such	as	
	that	visits	each	element	of	a	vector
sequentially	is	said	to	have	a	
stride-1	reference	pattern
	(with	respect	to
the	element	size).	We	will	sometimes	refer	to	stride-1	reference	patterns
as	
sequential	reference	patterns.
	Visiting	every	
k
th	element	of	a
contiguous	vector	is	called	a	
stride-k	reference	pattern.
	Stride-1
reference	patterns	are	a	common	and	important	source	of	spatial	locality
in	programs.	In	general,	as	the	stride	increases,	the	spatial	locality
decreases.
Stride	is	also	an	important	issue	for	programs	that	reference
multidimensional	arrays.	For	example,	consider	the	
	function
in	
Figure	
6.18(a)
	that	sums	the	elements	of	a	two-dimensional	array.
The	doubly	nested	loop	reads	the	elements	of	the	array	in	
row-major
order.
	That	is,	the	inner	loop	reads	the	elements	of	the	first	row,	then	the
second	row,	and	so	on.	The	
	function	enjoys	good	spatial
locality	because	it	references	the	array	in	the	same	row-major	order	that
the	array	is	stored	(
Figure	
6.18(b)
).	The	result	is	a	nice	stride-1
reference	pattern	with	excellent	spatial	locality.

(a)
Address
0
4
8
12
16
20
Contents
a
a
a
a
a
a
Access	order
1
2
3
4
5
6
(b)
Figure	
6.18	
(a)	Another	function	with	good	locality,	(b)	Reference
pattern	for	array	
a
	(
M
	=	2,	
N
	=	3).
There	is	good	spatial	locality	because	the	array	is	accessed	in	the	same
row-major	order	in	which	it	is	stored	in	memory.
00
01
02
10
11
12

(a)
Address
0
4
8
12
16
20
Contents
a
a
a
a
a
a
Access	order
1
3
5
2
4
6
(b)
Figure	
6.19	
(a)	A	function	with	poor	spatial	locality,	(b)	Reference
pattern	for	array	a	(
M
	=	2,	
N
	=	3).
The	function	has	poor	spatial	locality	because	it	scans	memory	with	a
stride-N	reference	pattern.
Seemingly	trivial	changes	to	a	program	can	have	a	big	impact	on	its
locality.	For	example,	the	
	function	in	
Figure	
6.19(a)
computes	the	same	result	as	the	
	function	in	
Figure
6.18(a)
.	The	only	difference	is	that	we	have	interchanged	the	
i
	and	
j
loops.	What	impact	does	interchanging	the	loops	have	on	its	locality?
00
01
02
10
11
12

The	
	function	suffers	from	poor	spatial	locality	because	it
scans	the	array	column-wise	instead	of	row-wise.	Since	C	arrays	are	laid
out	in	memory	row-wise,	the	result	is	a	stride-
N
	reference	pattern,	as
shown	in	
Figure	
6.19(b)
.
6.2.2	
Locality	of	Instruction	Fetches
Since	program	instructions	are	stored	in	memory	and	must	be	fetched
(read)	by	the	CPU,	we	can	also	evaluate	the	locality	of	a	program	with
respect	to	its	instruction	fetches.	For	example,	in	
Figure	
6.17
	the
instructions	in	the	body	of	the	
	loop	are	executed	in	sequential
memory	order,	and	thus	the	loop	enjoys	good	spatial	locality.	Since	the
loop	body	is	executed	multiple	times,	it	also	enjoys	good	temporal
locality.
An	important	property	of	code	that	distinguishes	it	from	program	data	is
that	it	is	rarely	modified	at	run	time.	While	a	program	is	executing,	the
CPU	reads	its	instructions	from	memory.	The	CPU	rarely	overwrites	or
modifies	these	instructions.
6.2.3	
Summary	of	Locality
In	this	section,	we	have	introduced	the	fundamental	idea	of	locality	and
have	identified	some	simple	rules	for	qualitatively	evaluating	the	locality
in	a	program:

Programs	that	repeatedly	reference	the	same	variables	enjoy	good
temporal	locality.
For	programs	with	stride-
k
	reference	patterns,	the	smaller	the	stride,
the	better	the	spatial	locality.	Programs	with	stride-1	reference
patterns	have	good	spatial	locality.	Programs	that	hop	around
memory	with	large	strides	have	poor	spatial	locality.
Loops	have	good	temporal	and	spatial	locality	with	respect	to
instruction	fetches.	The	smaller	the	loop	body	and	the	greater	the
number	of	loop	iterations,	the	better	the	locality.
Later	in	this	chapter,	after	we	have	learned	about	cache	memories	and
how	they	work,	we	will	show	you	how	to	quantify	the	idea	of	locality	in
terms	of	cache	hits	and	misses.	It	will	also	become	clear	to	you	why
programs	with	good	locality	typically	run	faster	than	programs	with	poor
locality.	Nonetheless,	knowing	how	to	glance	at	a	source	code	and
getting	a	high-level	feel	for	the	locality	in	the	program	is	a	useful	and
important	skill	for	a	programmer	to	master.
Practice	Problem	
6.7	
(solution	page	
662
)
Permute	the	loops	in	the	following	function	so	that	it	scans	the
three-dimensional	array	
a
	with	a	stride-1	reference	pattern.

(a)	An	array	of	
(b)	The	
	function

(c)	The	
	function
(d)	The	
	function

Figure	
6.20	
Code	examples	for	
Practice	Problem	
6.8
.
Practice	Problem	
6.8	
(solution	page	
663
)
The	three	functions	in	
Figure	
6.20
	perform	the	same	operation
with	varying	degrees	of	spatial	locality.	Rank-order	the	functions
with	respect	to	the	spatial	locality	enjoyed	by	each.	Explain	how
you	arrived	at	your	ranking.

6.3	
The	Memory	Hierarchy
Section	
6.1
	and	
6.2
	described	some	fundamental	and	enduring
properties	of	storage	technology	and	computer	software:
Storage	technology.	
Different	storage	technologies	have	widely
different	access	times.	Faster	technologies	cost	more	per	byte	than
slower	ones	and	have	less	capacity.	The	gap	between	CPU	and	main
memory	speed	is	widening.
Computer	software.	
Well-written	programs	tend	to	exhibit	good
locality.
Figure	
6.21	
The	memory	hierarchy.
In	one	of	the	happier	coincidences	of	computing,	these	fundamental
properties	of	hardware	and	software	complement	each	other	beautifully.

Their	complementary	nature	suggests	an	approach	for	organizing
memory	systems,	known	as	the	
memory	hierarchy
,	that	is	used	in	all
modern	computer	systems.	
Figure	
6.21
	shows	a	typical	memory
hierarchy.
In	general,	the	storage	devices	get	slower,	cheaper,	and	larger	as	we
move	from	higher	to	lower	
levels.
	At	the	highest	level	(L0)	are	a	small
number	of	fast	CPU	registers	that	the	CPU	can	access	in	a	single	clock
cycle.	Next	are	one	or	more	small	to	moderate-size	SRAM-based	cache
memories	that	can	be	accessed	in	a	few	CPU	clock	cycles.	These	are
followed	by	a	large	DRAM-based	main	memory	that	can	be	accessed	in
tens	to	hundreds	of	clock	cycles.	Next	are	slow	but	enormous	local	disks.
Finally,	some	systems	even	include	an	additional	level	of	disks	on	remote
servers	that	can	be	accessed	over	a	network.	For	example,	distributed
file	systems	such	as	the	Andrew	File	System	(AFS)	or	the	Network	File
System	(NFS)	allow	a	program	to	access	files	that	are	stored	on	remote
network-connected	servers.	Similarly,	the	World	Wide	Web	allows
programs	to	access	remote	files	stored	on	Web	servers	anywhere	in	the
world.