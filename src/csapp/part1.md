

Computer	Systems
A	Programmer's	Perspective

Computer	Systems
A	Programmer's	Perspective
Third	Edition
Randal	E.	Bryant
Carnegie	Mellon	University
David	R.	O'Hallaron
Carnegie	Mellon	University
Pearson
Boston
 
Columbus
 
Hoboken
 
Indianapolis
 
New	York	San
Francisco
 
Amsterdam
 
Cape	Town
 
Dubai
 
London
 
Madrid
 
Milan
 
Munich
 
Paris
 
Montreal
 
Toronto
 
Delhi
 
Mexico
 
City
 
Sao
Paulo
 
Sydney
 
Hong	Kong
 
Seoul
 
Singapore
 
Taipei
 
Tokyo

Vice	President	and	Editorial	Director:	Marcia	J.	Horton
Executive	Editor:	Matt	Goldstein
Editorial	Assistant:	Kelsey	Loanes
VP	of	Marketing:	Christy	Lesko
Director	of	Field	Marketing:	Tim	Galligan
Product	Marketing	Manager:	Bram	van	Kempen
Field	Marketing	Manager:	Demetrius	Hall
Marketing	Assistant:	Jon	Bryant
Director	of	Product	Management:	Erin	Gregg
Team	Lead	Product	Management:	Scott	Disanno
Program	Manager:	Joanne	Manning
Procurement	Manager:	Mary	Fischer
Senior	Specialist,	Program	Planning	and	Support:	Maura	Zaldivar-Garcia
over	Designer:	Joyce	Wells
Manager,	Rights	Management:	Rachel	Youdelman

Associate	Project	Manager,	Rights	Management:	William	J.	Opaluch
Full-Service	Project	Management:	Paul	Anagnostopoulos,	Windfall
Software
Composition:	Windfall	Software
Printer/Binder:	Courier	Westford
Cover	Printer:	Courier	Westford
Typeface:	10/12	Times	10,	ITC	Stone	Sans
The	graph	on	the	front	cover	is	a	"memory	mountain"	that	shows	the
measured	read	throughput	of	an	Intel	Core	i7	processor	as	a	function	of
spatial	and	temporal	locality.
Copyright	©	2016,	2011,	and	2003	by	Randal	E.	Bryant	and	David	R.
O'Hallaron.
	All	Rights	Reserved.	Printed	in	the	United	States	of	America.
This	publication	is	protected	by	copyright,	and	permission	should	be
obtained	from	the	publisher	prior	to	any	prohibited	reproduction,	storage
in	a	retrieval	system,	or	transmission	in	any	form	or	by	any	means,
electronic,	mechanical,	photocopying,	recording,	or	otherwise.	For
information	regarding	permissions,	request	forms	and	the	appropriate
contacts	within	the	Pearson	Education	Global	Rights	&	Permissions
department,	please	visit	
www.pearsoned.com/
permissions/
.
Many	of	the	designations	by	manufacturers	and	seller	to	distinguish	their
products	are	claimed	as	trademarks.	Where	those	designations	appear	in

this	book,	and	the	publisher	was	aware	of	a	trademark	claim,	the
designations	have	been	printed	in	initial	caps	or	all	caps.
The	author	and	publisher	of	this	book	have	used	their	best	efforts	in
preparing	this	book.	These	efforts	include	the	development,	research,
and	testing	of	theories	and	programs	to	determine	their	effectiveness.
The	author	and	publisher	make	no	warranty	of	any	kind,	expressed	or
implied,	with	regard	to	these	programs	or	the	documentation	contained	in
this	book.	The	author	and	publisher	shall	not	be	liable	in	any	event	for
incidental	or	consequential	damages	with,	or	arising	out	of,	the
furnishing,	performance,	or	use	of	these	programs.
Pearson	Education	Ltd.,	
London
Pearson	Education	Singapore,	Pte.	Ltd
Pearson	Education	Canada,	Inc.
Pearson	Education—Japan
Pearson	Education	Australia	PTY,	Limited
Pearson	Education	North	Asia,	Ltd.,	
Hong	Kong
Pearson	Educaciń	de	Mexico,	S.A.	de	C.V.
Pearson	Education	Malaysia,	Pte.	Ltd.
Pearson	Education,	Inc.,	
Upper	Saddle	River,	New	Jersey

Library	of	Congress	Cataloging-in-Publication	Data
Bryant,	Randal	E.
 
Computer	systems	:	a	programmer's	perspective	/	Randal	E.	Bryant,
Carnegie	Mellon	University,	David	R.	O'Hallaron,	Carnegie	Mellon.
University.—Third	edition.
   
pages	cm
 
Includes	bibliographical	references	and	index.
 
ISBN	978-0-13-409266-9—ISBN	0-13-409266-X
 
1.	Computer	systems.	2.	Computers.	3.	Telecommunication.	4.	User
interfaces	(Computer	systems)	I.	O'Hallaron,	David	R.	(David	Richard)	II.
Title.
 
QA76.5.B795	2016
 
005.3—
dc23
                                                                                   
2015000930
10	9	8	7	6	5	4	3	2	1
www.pearsonhighered.com

 
ISBN	10:	0-13-409266-X
ISBN	13:	978-0-13-409266-9

To	the	students	and	instructors	of	the	15−213	course	at
Carnegie	Mellon	University,	for	inspiring	us	to	develop	and
refine	the	material	for	this	book.

MasteringEngineering
For	
Computer	Systems:	A	Programmer's	Perspective
,	Third	Edition
Mastering	is	Pearson's	proven	online	Tutorial	Homework	program,	newly
available	with	the	third	edition	of	
Computer	Systems:	A	Programmer's
Perspective
.	The	Mastering	platform	allows	you	to	integrate	dynamic
homework—with	many	problems	taken	directly	from	the
Bryant/O'Hallaron	textbook—with	automatic	grading.	Mastering	allows
you	to	easily	track	the	performance	of	your	entire	class	on	an
assignment-by-assignment	basis,	or	view	the	detailed	work	of	an
individual	student.
For	more	information	or	a	demonstration	of	the	course,	visit
www.MasteringEngineering.com
	or	contact	your	local	Pearson
representative.
®

Contents
Preface	
xix
About	the	Authors	
xxxv
1	
A	Tour	of	Computer	Systems	
1
1.1	
Information	Is	Bits	+	Context	
3
1.2	
Programs	Are	Translated	by	Other	Programs	into	Different
Forms	
4
1.3	
It	Pays	to	Understand	How	Compilation	Systems	Work	
6
1.4	
Processors	Read	and	Interpret	Instructions	Stored	in	Memory
7
1.4.1	
Hardware	Organization	of	a	System	
8
1.4.2	
Running	the	
	Program	
10
1.5	
Caches	Matter	
11
1.6	
Storage	Devices	Form	a	Hierarchy	
14
1.7	
The	Operating	System	Manages	the	Hardware	
14
1.7.1	
Processes	
15
1.7.2	
Threads	
17
1.7.3	
Virtual	Memory	
18
1.7.4	
Files	
19

1.8	
Systems	Communicate	with	Other	Systems	Using	Networks
19
1.9	
Important	Themes	
22
1.9.1	
Amdahl's	Law	
22
1.9.2	
Concurrency	and	Parallelism	
24
1.9.3	
The	Importance	of	Abstractions	in	Computer	Systems	
26
1.10	
Summary
	
27
Bibliographic	Notes	
28
Solutions	to	Practice	Problems	
28
Part	
I	
Program	Structure	and	Execution
2	
Representing	and	Manipulating	Information	
31
2.1	
Information	Storage	
34
2.1.1	
Hexadecimal	Notation	
36
2.1.2	
Data	Sizes	
39
2.1.3	
Addressing	and	Byte	Ordering	
42
2.1.4	
Representing	Strings	
49
2.1.5	
Representing	Code	
49
2.1.6	
Introduction	to	Boolean	Algebra	
50
2.1.7	
Bit-Level	Operations	in	C	
54
2.1.8	
Logical	Operations	in	C	
56
2.1.9	
Shift	Operations	in	C	
57

2.2	
Integer	Representations	
59
2.2.1	
Integral	Data	Types	
60
2.2.2	
Unsigned	Encodings	
62
2.2.3	
Two's-Complement	Encodings	
64
2.2.4	
Conversions	between	Signed	and	Unsigned	
70
2.2.5	
Signed	versus	Unsigned	in	C	
74
2.2.6	
Expanding	the	Bit	Representation	of	a	Number	
76
2.2.7	
Truncating	Numbers	
81
2.2.8	
Advice	on	Signed	versus	Unsigned	
83
2.3	
Integer	Arithmetic	
84
2.3.1	
Unsigned	Addition	
84
2.3.2	
Two's-Complement	Addition	
90
2.3.3	
Two's-Complement	Negation	
95
2.3.4	
Unsigned	Multiplication	
96
2.3.5	
Two's-Complement	Multiplication	
97
2.3.6	
Multiplying	by	Constants	
101
2.3.7	
Dividing	by	Powers	of	2	
103
2.3.8	
Final	Thoughts	on	Integer	Arithmetic	
107
2.4	
Floating	Point	
108
2.4.1	
Fractional	Binary	Numbers	
109

2.4.2	
IEEE	Floating-Point	Representation	
112
2.4.3	
Example	Numbers	
115
2.4.4	
Rounding	
120
2.4.5	
Floating-Point	Operations	
122
2.4.6	
Floating	Point	in	C	
124
2.5	
Summary
	
126
Bibliographic	Notes	
127
Homework	Problems	
128
Solutions	to	Practice	Problems	
143
3	
Machine-Level	Representation	of	Programs	
163
3.1	
A	Historical	Perspective	
166
3.2	
Program	Encodings	
169
3.2.1	
Machine-Level	Code	
170
3.2.2	
Code	Examples	
172
3.2.3	
Notes	on	Formatting	
175
3.3	
Data	Formats	
177
3.4	
Accessing	Information	
179
3.4.1	
Operand	Specifiers	
180
3.4.2	
Data	Movement	Instructions	
182
3.4.3	
Data	Movement	Example	
186

3.4.4	
Pushing	and	Popping	Stack	Data	
189
3.5	
Arithmetic	and	Logical	Operations	
191
3.5.1	
Load	Effective	Address	
191
3.5.2	
Unary	and	Binary	Operations	
194
3.5.3	
Shift	Operations	
194
3.5.4	
Discussion	
196
3.5.5	
Special	Arithmetic	Operations	
197
3.6	
Control	
200
3.6.1	
Condition	Codes	
201
3.6.2	
Accessing	the	Condition	Codes	
202
3.6.3	
Jump	Instructions	
205
3.6.4	
Jump	Instruction	Encodings	
207
3.6.5	
Implementing	Conditional	Branches	with	Conditional
Control	
209
3.6.6	
Implementing	Conditional	Branches	with	Conditional
Moves	
214
3.6.7	
Loops	
220
3.6.8	
Switch	Statements	
232
3.7	
Procedures	
238
3.7.1	
The	Run-Time	Stack	
239
3.7.2	
Control	Transfer	
241

3.7.3	
Data	Transfer	
245
3.7.4	
Local	Storage	on	the	Stack	
248
3.7.5	
Local	Storage	in	Registers	
251
3.7.6	
Recursive	Procedures	
253
3.8	
Array	Allocation	and	Access	
255
3.8.1	
Basic	Principles	
255
3.8.2	
Pointer	Arithmetic	
257
3.8.3	
Nested	Arrays	
258
3.8.4	
Fixed-Size	Arrays	
260
3.8.5	
Variable-Size	Arrays	
262
3.9	
Heterogeneous	Data	Structures	
265
3.9.1	
Structures	
265
3.9.2	
Unions	
269
3.9.3	
Data	Alignment	
273
3.10	
Combining	Control	and	Data	in	Machine-Level	Programs
276
3.10.1	
Understanding	Pointers	
277
3.10.2	
Life	in	the	Real	World:	Using	the	
GDB
	Debugger	
279
3.10.3	
Out-of-Bounds	Memory	References	and	Buffer
Overflow	
279

3.10.4	
Thwarting	Buffer	Overflow	Attacks	
284
3.10.5	
Supporting	Variable-Size	Stack	Frames	
290
3.11	
Floating-Point	Code	
293
3.11.1	
Floating-Point	Movement	and	Conversion
Operations	
296
3.11.2	
Floating-Point	Code	in	Procedures	
301
3.11.3	
Floating-Point	Arithmetic	Operations	
302
3.11.4	
Defining	and	Using	Floating-Point	Constants	
304
3.11.5	
Using	Bitwise	Operations	in	Floating-Point	Code	
305
3.11.6	
Floating-Point	Comparison	Operations	
306
3.11.7	
Observations	about	Floating-Point	Code	
309
3.12	
Summary
	
309
Bibliographic	Notes	
310
Homework	Problems	
311
Solutions	to	Practice	Problems	
325
4	
Processor	Architecture	
351
4.1	
The	Y86-64	Instruction	Set	Architecture	
355
4.1.1	
Programmer-Visible	State	
355
4.1.2	
Y86-64	Instructions	
356
4.1.3	
Instruction	Encoding	
358

4.1.4	
Y86-64	Exceptions	
363
4.1.5	
Y86-64	Programs	
364
4.1.6	
Some	Y86-64	Instruction	Details	
370
4.2	
Logic	Design	and	the	Hardware	Control	Language	HCL
372
4.2.1	
Logic	Gates	
373
4.2.2	
Combinational	Circuits	and	HCL	Boolean
Expressions	
374
4.2.3	
Word-Level	Combinational	Circuits	and	HCL	Integer
Expressions	
376
4.2.4	
Set	Membership	
380
4.2.5	
Memory	and	Clocking	
381
4.3	
Sequential	Y86-64	Implementations	
384
4.3.1	
Organizing	Processing	into	Stages	
384
4.3.2	
SEQ	Hardware	Structure	
396
4.3.3	
SEQ	Timing	
400
4.3.4	
SEQ	Stage	Implementations	
404
4.4	
General	Principles	of	Pipelining	
412
4.4.1	
Computational	Pipelines	
412
4.4.2	
A	Detailed	Look	at	Pipeline	Operation	
414
4.4.3	
Limitations	of	Pipelining	
416

4.4.4	
Pipelining	a	System	with	Feedback	
419
4.5	
Pipelined	Y86-64	Implementations	
421
4.5.1	
SEQ+:	Rearranging	the	Computation	Stages	
421
4.5.2	
Inserting	Pipeline	Registers	
422
4.5.3	
Rearranging	and	Relabeling	Signals	
426
4.5.4	
Next	PC	Prediction	
427
4.5.5	
Pipeline	Hazards	
429
4.5.6	
Exception	Handling	
444
4.5.7	
PIPE	Stage	Implementations	
447
4.5.8	
Pipeline	Control	Logic	
455
4.5.9	
Performance	Analysis	
464
4.5.10	
Unfinished	Business	
468
4.6	
Summary
	
470
4.6.1	
Y86-64	Simulators	
472
Bibliographic	Notes	
473
Homework	Problems	
473
Solutions	to	Practice	Problems	
480
5	
Optimizing	Program	Performance	
495
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
5.7.1	
Overall	Operation	
518
5.7.2	
Functional	Unit	Performance	
523
5.7.3	
An	Abstract	Model	of	Processor	Operation	
525
5.8	
Loop	Unrolling
	
531
5.9	
Enhancing	Parallelism
	
536
5.9.1	
Multiple	Accumulators	
536
5.9.2	
Reassociation	Transformation	
541
5.10	
Summary	of	Results	for	Optimizing	Combining	Code	
547
5.11	
Some	Limiting	Factors	
548
5.11.1	
Register	Spilling	
548
5.11.2	
Branch	Prediction	and	Misprediction	Penalties	
549
5.12	
Understanding	Memory	Performance	
553
5.12.1	
Load	Performance	
554
5.12.2	
Store	Performance	
555
5.13	
Life	in	the	Real	World:	Performance	Improvement

Techniques	
561
5.14	
Identifying	and	Eliminating	Performance	Bottlenecks	
562
5.14.1	
Program	Profiling	
562
5.14.2	
Using	a	Profiler	to	Guide	Optimization	
565
5.15	
Summary
	
568
Bibliographic	Notes	
569
Homework	Problems	
570
Solutions	to	Practice	Problems	
573
6	
The	Memory	Hierarchy	
579
6.1	
Storage	Technologies	
581
6.1.1	
Random	Access	Memory	
581
6.1.2	
Disk	Storage	
589
6.1.3	
Solid	State	Disks	
600
6.1.4	
Storage	Technology	Trends	
602
6.2	
Locality	
604
6.2.1	
Locality	of	References	to	Program	Data	
606
6.2.2	
Locality	of	Instruction	Fetches	
607
6.2.3	
Summary	of	Locality	
608
6.3	
The	Memory	Hierarchy	
609
6.3.1	
Caching	in	the	Memory	Hierarchy	
610

6.3.2	
Summary	of	Memory	Hierarchy	Concepts	
614
6.4	
Cache	Memories	
614
6.4.1	
Generic	Cache	Memory	Organization	
615
6.4.2	
Direct-Mapped	Caches	
617
6.4.3	
Set	Associative	Caches	
624
6.4.4	
Fully	Associative	Caches	
626
6.4.5	
Issues	with	Writes	
630
6.4.6	
Anatomy	of	a	Real	Cache	Hierarchy	
631

6.4.7	
Performance	Impact	of	Cache	Parameters	
631
6.5	
Writing	Cache-Friendly	Code	
633
6.6	
Putting	It	Together:	The	Impact	of	Caches	on	Program
Performance	
639
6.6.1	
The	Memory	Mountain	
639
6.6.2	
Rearranging	Loops	to	Increase	Spatial	Locality	
643
6.6.3	
Exploiting	Locality	in	Your	Programs	
647
6.7	
Summary
	
648
Bibliographic	Notes	
648
Homework	Problems	
649
Solutions	to	Practice	Problems	
660
Part	
II	
Running	Programs	on	a	System
7	
Linking	
669
7.1	
Compiler	Drivers	
671
7.2	
Static	Linking	
672
7.3	
Object	Files	
673
7.4	
Relocatable	Object	Files	
674
7.5	
Symbols	and	Symbol	Tables	
675
7.6	
Symbol	Resolution	
679
7.6.1	
How	Linkers	Resolve	Duplicate	Symbol	Names	
680
7.6.2	
Linking	with	Static	Libraries	
684

7.6.3	
How	Linkers	Use	Static	Libraries	to	Resolve
References	
688
7.7	
Relocation	
689
7.7.1	
Relocation	Entries	
690
7.7.2	
Relocating	Symbol	References	
691
7.8	
Executable	Object	Files	
695
7.9	
Loading	Executable	Object	Files	
697
7.10	
Dynamic	Linking	with	Shared	Libraries	
698
7.11	
Loading	and	Linking	Shared	Libraries	from	Applications
701
7.12	
Position-Independent	Code	(PIC)	
704
7.13	
Library	Interpositioning	
707
7.13.1	
Compile-Time	Interpositioning	
708
7.13.2	
Link-Time	Interpositioning	
708
7.13.3	
Run-Time	Interpositioning	
710
7.14	
Tools	for	Manipulating	Object	Files	
713
7.15	
Summary
	
713
Bibliographic	Notes	
714
Homework	Problems	
714
Solutions	to	Practice	Problems	
717

8	
Exceptional	Control	Flow	
721
8.1	
Exceptions	
723
8.1.1	
Exception	Handling	
724
8.1.2	
Classes	of	Exceptions	
726
8.1.3	
Exceptions	in	Linux/x86-64	Systems	
729
8.2	
Processes	
732
8.2.1	
Logical	Control	Flow	
732
8.2.2	
Concurrent	Flows	
733
8.2.3	
Private	Address	Space	
734
8.2.4	
User	and	Kernel	Modes	
734
8.2.5	
Context	Switches	
736
8.3	
System	Call	Error	Handling	
737
8.4	
Process	Control	
738
8.4.1	
Obtaining	Process	IDs	
739
8.4.2	
Creating	and	Terminating	Processes	
739
8.4.3	
Reaping	Child	Processes	
743
8.4.4	
Putting	Processes	to	Sleep	
749
8.4.5	
Loading	and	Running	Programs	
750
8.4.6	
Using	
	to	Run	Programs	
753

8.5	
Signals	
756
8.5.1	
Signal	Terminology	
758
8.5.2	
Sending	Signals	
759
8.5.3	
Receiving	Signals	
762
8.5.4	
Blocking	and	Unblocking	Signals	
764
8.5.5	
Writing	Signal	Handlers	
766
8.5.6	
Synchronizing	Flows	to	Avoid	Nasty	Concurrency
Bugs	
776
8.5.7	
Explicitly	Waiting	for	Signals	
778
8.6	
Nonlocal	Jumps	
781
8.7	
Tools	for	Manipulating	Processes	
786
8.8	
Summary
	
787
Bibliographic	Notes	
787
Homework	Problems	
788
Solutions	to	Practice	Problems	
795
9	
Virtual	Memory	
801
9.1	
Physical	and	Virtual	Addressing	
803
9.2	
Address	Spaces	
804
9.3	
VM	as	a	Tool	for	Caching	
805
9.3.1	
DRAM	Cache	Organization	
806

9.3.2	
Page	Tables	
806
9.3.3	
Page	Hits	
808
9.3.4	
Page	Faults	
808
9.3.5	
Allocating	Pages	
810
9.3.6	
Locality	to	the	Rescue	Again	
810
9.4	
VM	as	a	Tool	for	Memory	Management	
811
9.5	
VM	as	a	Tool	for	Memory	Protection	
812
9.6	
Address	Translation	
813
9.6.1	
Integrating	Caches	and	VM	
817
9.6.2	
Speeding	Up	Address	Translation	with	a	TLB	
817
9.6.3	
Multi-Level	Page	Tables	
819
9.6.4	
Putting	It	Together:	End-to-End	Address	Translation
821
9.7	
Case	Study:	The	Intel	Core	i7/Linux	Memory	System	
825
9.7.1	
Core	i7	Address	Translation	
826
9.7.2	
Linux	Virtual	Memory	System	
828
9.8	
Memory	Mapping	
833
9.8.1	
Shared	Objects	Revisited	
833
9.8.2	
The	
	Function	Revisited	
836
9.8.3	
The	
	Function	Revisited	
836

9.8.4	
User-Level	Memory	Mapping	with	the	
	Function
837
9.9	
Dynamic	Memory	Allocation	
839
9.9.1	
The	
	and	
	Functions	
840
9.9.2	
Why	Dynamic	Memory	Allocation?	
843
9.9.3	
Allocator	Requirements	and	Goals	
844
9.9.4	
Fragmentation	
846
9.9.5	
Implementation	Issues	
846
9.9.6	
Implicit	Free	Lists	
847
9.9.7	
Placing	Allocated	Blocks	
849
9.9.8	
Splitting	Free	Blocks	
849
9.9.9	
Getting	Additional	Heap	Memory	
850
9.9.10	
Coalescing	Free	Blocks	
850
9.9.11	
Coalescing	with	Boundary	Tags	
851
9.9.12	
Putting	It	Together:	Implementing	a	Simple	Allocator
854
9.9.13	
Explicit	Free	Lists	
862
9.9.14	
Segregated	Free	Lists	
863
9.10	
Garbage	Collection	
865
9.10.1	
Garbage	Collector	Basics	
866

9.10.2	
Mark&Sweep	Garbage	Collectors	
867
9.10.3	
Conservative	Mark&Sweep	for	C	Programs	
869
9.11	
Common	Memory-Related	Bugs	in	C	Programs	
870
9.11.1	
Dereferencing	Bad	Pointers	
870
9.11.2	
Reading	Uninitialized	Memory	
871
9.11.3	
Allowing	Stack	Buffer	Overflows	
871
9.11.4	
Assuming	That	Pointers	and	the	Objects	They	Point
to	Are	the	Same	Size	
872
9.11.5	
Making	Off-by-One	Errors	
872
9.11.6	
Referencing	a	Pointer	Instead	of	the	Object	It	Points
To	
873
9.11.7	
Misunderstanding	Pointer	Arithmetic	
873
9.11.8	
Referencing	Nonexistent	Variables	
874
9.11.9	
Referencing	Data	in	Free	Heap	Blocks	
874
9.11.10	
Introducing	Memory	Leaks	
875
9.12	
Summary
	
875
Bibliographic	Notes	
876
Homework	Problems	
876
Solutions	to	Practice	Problems	
880
Part	
III	
Interaction	and	Communication	between	Programs

10	
System-Level	I/O	
889
10.1	
Unix	I/O	
890
10.2	
Files	
891
10.3	
Opening	and	Closing	Files	
893
10.4	
Reading	and	Writing	Files	
895
10.5	
Robust	Reading	and	Writing	with	the	R
IO
	Package	
897
10.5.1	
R
IO
	Unbuffered	Input	and	Output	Functions	
897
10.5.2	
R
IO
	Buffered	Input	Functions	
898
10.6	
Reading	File	Metadata	
903
10.7	
Reading	Directory	Contents	
905
10.8	
Sharing	Files	
906
10.9	
I/O	Redirection	
909
10.10	
Standard	I/O	
911
10.11	
Putting	It	Together:	Which	I/O	Functions	Should	I	Use?
911
10.12	
Summary
	
913
Bibliographic	Notes	
914
Homework	Problems	
914
Solutions	to	Practice	Problems	
915
11	
Network	Programming	
917

11.1	
The	Client-Server	Programming	Model	
918
11.2	
Networks	
919
11.3	
The	Global	IP	Internet	
924
11.3.1	
IP	Addresses	
925
11.3.2	
Internet	Domain	Names	
927
11.3.3	
Internet	Connections	
929
11.4	
The	Sockets	Interface	
932
11.4.1	
Socket	Address	Structures	
933
11.4.2	
The	
	Function	
934
11.4.3	
The	
	Function	
934
11.4.4	
The	
	Function	
935
11.4.5	
The	
	Function	
935
11.4.6	
The	
	Function	
936
11.4.7	
Host	and	Service	Conversion	
937
11.4.8	
Helper	Functions	for	the	Sockets	Interface	
942
11.4.9	
Example	Echo	Client	and	Server	
944
11.5	
Web	Servers	
948
11.5.1	
Web	Basics	
948
11.5.2	
Web	Content	
949
11.5.3	
HTTP	Transactions	
950

11.5.4	
Serving	Dynamic	Content	
953
11.6	
Putting	It	Together:	The	
TINY
	Web	Server	
956
11.7	
Summary
	
964
Bibliographic	Notes	
965
Homework	Problems	
965
Solutions	to	Practice	Problems	
966
12	
Concurrent	Programming	
971
12.1	
Concurrent	Programming	with	Processes	
973
12.1.1	
A	Concurrent	Server	Based	on	Processes	
974
12.1.2	
Pros	and	Cons	of	Processes	
975
12.2	
Concurrent	Programming	with	I/O	Multiplexing	
977
12.2.1	
A	Concurrent	Event-Driven	Server	Based	on	I/O
Multiplexing	
980
12.2.2	
Pros	and	Cons	of	I/O	Multiplexing	
985
12.3	
Concurrent	Programming	with	Threads	
985
12.3.1	
Thread	Execution	Model	
986
12.3.2	
Posix	Threads	
987
12.3.3	
Creating	Threads	
988
12.3.4	
Terminating	Threads	
988
12.3.5	
Reaping	Terminated	Threads	
989

12.3.6	
Detaching	Threads	
989
12.3.7	
Initializing	Threads	
990
12.3.8	
A	Concurrent	Server	Based	on	Threads	
991
12.4	
Shared	Variables	in	Threaded	Programs	
992
12.4.1	
Threads	Memory	Model	
993
12.4.2	
Mapping	Variables	to	Memory	
994
12.4.3	
Shared	Variables	
995
12.5	
Synchronizing	Threads	with	Semaphores	
995
12.5.1	
Progress	Graphs	
999
12.5.2	
Semaphores	
1001
12.5.3	
Using	Semaphores	for	Mutual	Exclusion	
1002
12.5.4	
Using	Semaphores	to	Schedule	Shared	Resources
1004
12.5.5	
Putting	It	Together:	A	Concurrent	Server	Based	on
Prethreading	
1008
12.6	
Using	Threads	for	Parallelism	
1013
12.7	
Other	Concurrency	Issues	
1020
12.7.1	
Thread	Safety	
1020
12.7.2	
Reentrancy	
1023
12.7.3	
Using	Existing	Library	Functions	in	Threaded
Programs	
1024

12.7.4	
Races	
1025
12.7.5	
Deadlocks	
1027
12.8	
Summary
	
1030
Bibliographic	Notes	
1030
Homework	Problems	
1031
Solutions	to	Practice	Problems	
1036
A	
Error	Handling	
1041
A.1	
Error	Handling	in	Unix	Systems	
1042
A.2	
Error-Handling	Wrappers	
1043
References	
1047
Index	
1053

Preface
This	book	(known	as	CS:APP)	is	for	computer	scientists,	computer
engineers,	and	others	who	want	to	be	able	to	write	better	programs	by
learning	what	is	going	on	"under	the	hood"	of	a	computer	system.
Our	aim	is	to	explain	the	enduring	concepts	underlying	all	computer
systems,	and	to	show	you	the	concrete	ways	that	these	ideas	affect	the
correctness,	performance,	and	utility	of	your	application	programs.	Many
systems	books	are	written	from	a	
builder's	perspective
,	describing	how	to
implement	the	hardware	or	the	systems	software,	including	the	operating
system,	compiler,	and	network	interface.	This	book	is	written	from	a
programmer's	perspective
,	describing	how	application	programmers	can
use	their	knowledge	of	a	system	to	write	better	programs.	Of	course,
learning	what	a	system	is	supposed	to	do	provides	a	good	first	step	in
learning	how	to	build	one,	so	this	book	also	serves	as	a	valuable
introduction	to	those	who	go	on	to	implement	systems	hardware	and
software.	Most	systems	books	also	tend	to	focus	on	just	one	aspect	of
the	system,	for	example,	the	hardware	architecture,	the	operating
system,	the	compiler,	or	the	network.	This	book	spans	all	of	these
aspects,	with	the	unifying	theme	of	a	programmer's	perspective.
If	you	study	and	learn	the	concepts	in	this	book,	you	will	be	on	your	way
to	becoming	the	rare	
power	programmer
	who	knows	how	things	work	and
how	to	fix	them	when	they	break.	You	will	be	able	to	write	programs	that
make	better	use	of	the	capabilities	provided	by	the	operating	system	and
systems	software,	that	operate	correctly	across	a	wide	range	of	operating

conditions	and	run-time	parameters,	that	run	faster,	and	that	avoid	the
flaws	that	make	programs	vulnerable	to	cyberattack.	You	will	be	prepared
to	delve	deeper	into	advanced	topics	such	as	compilers,	computer
architecture,	operating	systems,	embedded	systems,	networking,	and
cybersecurity.
Assumptions	about	the	Reader's
Background
This	book	focuses	on	systems	that	execute	x86-64	machine	code.	x86-
64	is	the	latest	in	an	evolutionary	path	followed	by	Intel	and	its
competitors	that	started	with	the	8086	microprocessor	in	1978.	Due	to	the
naming	conventions	used	by	Intel	for	its	microprocessor	line,	this	class	of
microprocessors	is	referred	to	colloquially	as	"x86."	As	semiconductor
technology	has	evolved	to	allow	more	transistors	to	be	integrated	onto	a
single	chip,	these	processors	have	progressed	greatly	in	their	computing
power	and	their	memory	capacity.	As	part	of	this	progression,	they	have
gone	from	operating	on	16-bit	words,	to	32-bit	words	with	the	introduction
of	IA32	processors,	and	most	recently	to	64-bit	words	with	x86-64.
We	consider	how	these	machines	execute	C	programs	on	Linux.	Linux	is
one	of	a	number	of	operating	systems	having	their	heritage	in	the	Unix
operating	system	developed	originally	by	Bell	Laboratories.	Other
members	of	this	class
New	to	C?	
Advice	on	the	C

programming	language
To	help	readers	whose	background	in	C	programming	is	weak	(or
nonexistent),	we	have	also	included	these	special	notes	to
highlight	features	that	are	especially	important	in	C.	We	assume
you	are	familiar	with	C++	or	Java.
of	operating	systems	include	Solaris,	FreeBSD,	and	MacOS	X.	In	recent
years,	these	operating	systems	have	maintained	a	high	level	of
compatibility	through	the	efforts	of	the	Posix	and	Standard	Unix
Specification	standardization	efforts.	Thus,	the	material	in	this	book
applies	almost	directly	to	these	"Unix-like"	operating	systems.
The	text	contains	numerous	programming	examples	that	have	been
compiled	and	run	on	Linux	systems.	We	assume	that	you	have	access	to
such	a	machine,	and	are	able	to	log	in	and	do	simple	things	such	as
listing	files	and	changing	directories.	If	your	computer	runs	Microsoft
Windows,	we	recommend	that	you	install	one	of	the	many	different	virtual
machine	environments	(such	as	VirtualBox	or	VMWare)	that	allow
programs	written	for	one	operating	system	(the	guest	OS)	to	run	under
another	(the	host	OS).
We	also	assume	that	you	have	some	familiarity	with	C	or	C++.	If	your
only	prior	experience	is	with	Java,	the	transition	will	require	more	effort
on	your	part,	but	we	will	help	you.	Java	and	C	share	similar	syntax	and
control	statements.	However,	there	are	aspects	of	C	(particularly
pointers,	explicit	dynamic	memory	allocation,	and	formatted	I/O)	that	do
not	exist	in	Java.	Fortunately,	C	is	a	small	language,	and	it	is	clearly	and
beautifully	described	in	the	classic	"K&R"	text	by	Brian	Kernighan	and
Dennis	Ritchie	[
61
].	Regardless	of	your	programming	background,

consider	K&R	an	essential	part	of	your	personal	systems	library.	If	your
prior	experience	is	with	an	interpreted	language,	such	as	Python,	Ruby,
or	Perl,	you	will	definitely	want	to	devote	some	time	to	learning	C	before
you	attempt	to	use	this	book.
Several	of	the	early	chapters	in	the	book	explore	the	interactions
between	C	programs	and	their	machine-language	counterparts.	The
machine-language	examples	were	all	generated	by	the	GNU	
GCC
compiler	running	on	x86-64	processors.	We	do	not	assume	any	prior
experience	with	hardware,	machine	language,	or	assembly-language
programming.
How	to	Read	the	Book
Learning	how	computer	systems	work	from	a	programmer's	perspective
is	great	fun,	mainly	because	you	can	do	it	actively.	Whenever	you	learn
something	new,	you	can	try	it	out	right	away	and	see	the	result	firsthand.
In	fact,	we	believe	that	the	only	way	to	learn	systems	is	to	
do
	systems,
either	working	concrete	problems	or	writing	and	running	programs	on	real
systems.
This	theme	pervades	the	entire	book.	When	a	new	concept	is	introduced,
it	is	followed	in	the	text	by	one	or	more	
practice	problems
	that	you	should
work
--------------------------------------------------
code/intro/hello.c

--------------------------------------------------
code/intro/hello.c
Figure	
1	
A	typical	code	example.
immediately	to	test	your	understanding.	Solutions	to	the	practice
problems	are	at	the	end	of	each	chapter.	As	you	read,	try	to	solve	each
problem	on	your	own	and	then	check	the	solution	to	make	sure	you	are
on	the	right	track.	Each	chapter	is	followed	by	a	set	of	
homework
problems
	of	varying	difficulty.	Your	instructor	has	the	solutions	to	the
homework	problems	in	an	instructor's	manual.	For	each	homework
problem,	we	show	a	rating	of	the	amount	of	effort	we	feel	it	will	require:
♦	Should	require	just	a	few	minutes.	Little	or	no	programming
required.
♦♦	Might	require	up	to	20	minutes.	Often	involves	writing	and	testing
some	code.	(Many	of	these	are	derived	from	problems	we	have	given
on	exams.)

♦♦♦	Requires	a	significant	effort,	perhaps	1−2	hours.	Generally
involves	writing	and	testing	a	significant	amount	of	code.
♦♦♦♦	A	lab	assignment,	requiring	up	to	10	hours	of	effort.
Each	code	example	in	the	text	was	formatted	directly,	without	any	manual
intervention,	from	a	C	program	compiled	with	
GCC
	and	tested	on	a	Linux
system.	Of	course,	your	system	may	have	a	different	version	of	
GCC
,	or	a
different	compiler	altogether,	so	your	compiler	might	generate	different
machine	code;	but	the	overall	behavior	should	be	the	same.	All	of	the
source	code	is	available	from	the	CS:APP	Web	page	("CS:APP"	being
our	shorthand	for	the	book's	title)	at	csapp.cs.cmu.edu.	In	the	text,	the
filenames	of	the	source	programs	are	documented	in	horizontal	bars	that
surround	the	formatted	code.	For	example,	the	program	in	
Figure	
1
can	be	found	in	the	file	
	in	directory	
.	We	encourage
you	to	try	running	the	example	programs	on	your	system	as	you
encounter	them.
To	avoid	having	a	book	that	is	overwhelming,	both	in	bulk	and	in	content,
we	have	created	a	number	of	
Web	asides
	containing	material	that
supplements	the	main	presentation	of	the	book.	These	asides	are
referenced	within	the	book	with	a	notation	of	the	form	
CHAP
:
TOP
,	where
CHAP
	is	a	short	encoding	of	the	chapter	subject,	and	
TOP
	is	a	short	code
for	the	topic	that	is	covered.	For	example,	Web	Aside	
DATA
:
BOOL
	contains
supplementary	material	on	Boolean	algebra	for	the	presentation	on	data
representations	in	
Chapter	
2
,	while	Web	Aside	
ARCH
:
VLOG
	contains
material	describing	processor	designs	using	the	Verilog	hardware
description	language,	supplementing	the	presentation	of	processor
design	in	
Chapter	
4
.	All	of	these	Web	asides	are	available	from	the
CS:APP	Web	page.

Book	Overview
The	CS:APP	book	consists	of	12	chapters	designed	to	capture	the	core
ideas	in	computer	systems.	Here	is	an	overview.
Chapter	
1
:	A	Tour	of	Computer	Systems.	
This	chapter
introduces	the	major	ideas	and	themes	in	computer	systems	by
tracing	the	life	cycle	of	a	simple	"hello,	world"	program.
Chapter	
2
:	Representing	and	Manipulating	Information.	
We
cover	computer	arithmetic,	emphasizing	the	properties	of	unsigned
and	two's-complement	number	representations	that	affect
programmers.	We	consider	how	numbers	are	represented	and
therefore	what	range	of	values	can	be	encoded	for	a	given	word	size.
We	consider	the	effect	of	casting	between	signed	and	unsigned
numbers.	We	cover	the	mathematical	properties	of	arithmetic
operations.	Novice	programmers	are	often	surprised	to	learn	that	the
(two's-complement)	sum	or	product	of	two	positive	numbers	can	be
negative.	On	the	other	hand,	two's-complement	arithmetic	satisfies
many	of	the	algebraic	properties	of	integer	arithmetic,	and	hence	a
compiler	can	safely	transform	multiplication	by	a	constant	into	a
sequence	of	shifts	and	adds.	We	use	the	bit-level	operations	of	C	to
demonstrate	the	principles	and	applications	of	Boolean	algebra.	We
cover	the	IEEE	floating-point	format	in	terms	of	how	it	represents
values	and	the	mathematical	properties	of	floating-point	operations.
Having	a	solid	understanding	of	computer	arithmetic	is	critical	to
writing	reliable	programs.	For	example,	programmers	and	compilers

cannot	replace	the	expression	
	with	
,	due	to	the
possibility	of	overflow.	They	cannot	even	replace	it	with	the
expression	
,	due	to	the	asymmetric	range	of	negative	and
positive	numbers	in	the	two's-complement	representation.	Arithmetic
overflow	is	a	common	source	of	programming	errors	and	security
vulnerabilities,	yet	few	other	books	cover	the	properties	of	computer
arithmetic	from	a	programmer's	perspective.
Chapter	
3
:	Machine-Level	Representation	of	Programs.	
We
teach	you	how	to	read	the	x86-64	machine	code	generated	by	a	C
compiler.	We	cover	the	basic	instruction	patterns	generated	for
different	control	constructs,	such	as	conditionals,	loops,	and	switch
statements.	We	cover	the	implementation	of	procedures,	including
stack	allocation,	register	usage	conventions,	and	parameter	passing.
We	cover	the	way	different	data	structures	such	as	structures,	unions,
and	arrays	are	allocated	and	accessed.	We	cover	the	instructions	that
implement	both	integer	and	floating-point	arithmetic.	We	also	use	the
machine-level	view	of	programs	as	a	way	to	understand	common
code	security	vulnerabilities,	such	as	buffer	overflow,	and	steps	that
the	programmer,
Aside	
What	is	an	aside?
You	will	encounter	asides	of	this	form	throughout	the	text.
Asides	are	parenthetical	remarks	that	give	you	some	additional
insight	into	the	current	topic.	Asides	serve	a	number	of
purposes.	Some	are	little	history	lessons.	For	example,	where
did	C,	Linux,	and	the	Internet	come	from?	Other	asides	are
meant	to	clarify	ideas	that	students	often	find	confusing.	For
example,	what	is	the	difference	between	a	cache	line,	set,	and

block?	Other	asides	give	real-world	examples,	such	as	how	a
floating-point	error	crashed	a	French	rocket	or	the	geometric
and	operational	parameters	of	a	commercial	disk	drive.	Finally,
some	asides	are	just	fun	stuff.	For	example,	what	is	a
"hoinky"?
grammer,	the	compiler,	and	the	operating	system	can	take	to	reduce
these	threats.	Learning	the	concepts	in	this	chapter	helps	you
become	a	better	programmer,	because	you	will	understand	how
programs	are	represented	on	a	machine.	One	certain	benefit	is	that
you	will	develop	a	thorough	and	concrete	understanding	of	pointers.
Chapter	
4
:	Processor	Architecture.	
This	chapter	covers	basic
combinational	and	sequential	logic	elements,	and	then	shows	how
these	elements	can	be	combined	in	a	datapath	that	executes	a
simplified	subset	of	the	x86-64	instruction	set	called	"Y86-64."	We
begin	with	the	design	of	a	single-cycle	datapath.	This	design	is
conceptually	very	simple,	but	it	would	not	be	very	fast.	We	then
introduce	
pipelining
,	where	the	different	steps	required	to	process	an
instruction	are	implemented	as	separate	stages.	At	any	given	time,
each	stage	can	work	on	a	different	instruction.	Our	five-stage
processor	pipeline	is	much	more	realistic.	The	control	logic	for	the
processor	designs	is	described	using	a	simple	hardware	description
language	called	HCL.	Hardware	designs	written	in	HCL	can	be
compiled	and	linked	into	simulators	provided	with	the	textbook,	and
they	can	be	used	to	generate	Verilog	descriptions	suitable	for
synthesis	into	working	hardware.
Chapter	
5
:	Optimizing	Program	Performance.	
This	chapter
introduces	a	number	of	techniques	for	improving	code	performance,
with	the	idea	being	that	programmers	learn	to	write	their	C	code	in

such	a	way	that	a	compiler	can	then	generate	efficient	machine	code.
We	start	with	transformations	that	reduce	the	work	to	be	done	by	a
program	and	hence	should	be	standard	practice	when	writing	any
program	for	any	machine.	We	then	progress	to	transformations	that
enhance	the	degree	of	instruction-level	parallelism	in	the	generated
machine	code,	thereby	improving	their	performance	on	modern
"superscalar"	processors.	To	motivate	these	transformations,	we
introduce	a	simple	operational	model	of	how	modern	out-of-order
processors	work,	and	show	how	to	measure	the	potential
performance	of	a	program	in	terms	of	the	critical	paths	through	a
graphical	representation	of	a	program.	You	will	be	surprised	how
much	you	can	speed	up	a	program	by	simple	transformations	of	the	C
code.
Chapter	
6
:	The	Memory	Hierarchy.	
The	memory	system	is	one	of
the	most	visible	parts	of	a	computer	system	to	application
programmers.	To	this	point,	you	have	relied	on	a	conceptual	model	of
the	memory	system	as	a	linear	array	with	uniform	access	times.	In
practice,	a	memory	system	is	a	hierarchy	of	storage	devices	with
different	capacities,	costs,	and	access	times.	We	cover	the	different
types	of	RAM	and	ROM	memories	and	the	geometry	and	organization
of	magnetic-disk	and	solid	state	drives.	We	describe	how	these
storage	devices	are	arranged	in	a	hierarchy.	We	show	how	this
hierarchy	is	made	possible	by	locality	of	reference.	We	make	these
ideas	concrete	by	introducing	a	unique	view	of	a	memory	system	as	a
"memory	mountain"	with	ridges	of	temporal	locality	and	slopes	of
spatial	locality.	Finally,	we	show	you	how	to	improve	the	performance
of	application	programs	by	improving	their	temporal	and	spatial
locality.
Chapter	
7
:	Linking.	
This	chapter	covers	both	static	and	dynamic

linking,	including	the	ideas	of	relocatable	and	executable	object	files,
symbol	resolution,	relocation,	static	libraries,	shared	object	libraries,
position-independent	code,	and	library	interpositioning.	Linking	is	not
covered	in	most	systems	texts,	but	we	cover	it	for	two	reasons.	First,
some	of	the	most	confusing	errors	that	programmers	can	encounter
are	related	to	glitches	during	linking,	especially	for	large	software
packages.	Second,	the	object	files	produced	by	linkers	are	tied	to
concepts	such	as	loading,	virtual	memory,	and	memory	mapping.
Chapter	
8
:	Exceptional	Control	Flow.	
In	this	part	of	the
presentation,	we	step	beyond	the	single-program	model	by
introducing	the	general	concept	of	exceptional	control	flow	(i.e.,
changes	in	control	flow	that	are	outside	the	normal	branches	and
procedure	calls).	We	cover	examples	of	exceptional	control	flow	that
exist	at	all	levels	of	the	system,	from	low-level	hardware	exceptions
and	interrupts,	to	context	switches	between	concurrent	processes,	to
abrupt	changes	in	control	flow	caused	by	the	receipt	of	Linux	signals,
to	the	nonlocal	jumps	in	C	that	break	the	stack	discipline.
This	is	the	part	of	the	book	where	we	introduce	the	fundamental	idea
of	a	
process
,	an	abstraction	of	an	executing	program.	You	will	learn
how	processes	work	and	how	they	can	be	created	and	manipulated
from	application	programs.	We	show	how	application	programmers
can	make	use	of	multiple	processes	via	Linux	system	calls.	When	you
finish	this	chapter,	you	will	be	able	to	write	a	simple	Linux	shell	with
job	control.	It	is	also	your	first	introduction	to	the	nondeterministic
behavior	that	arises	with	concurrent	program	execution.
Chapter	
9
:	Virtual	Memory.	
Our	presentation	of	the	virtual
memory	system	seeks	to	give	some	understanding	of	how	it	works
and	its	characteristics.	We	want	you	to	know	how	it	is	that	the

different	simultaneous	processes	can	each	use	an	identical	range	of
addresses,	sharing	some	pages	but	having	individual	copies	of
others.	We	also	cover	issues	involved	in	managing	and	manipulating
virtual	memory.	In	particular,	we	cover	the	operation	of	storage
allocators	such	as	the	standard-library	
	and	
	operations.
Covering	
this	material	serves	several	purposes.	It	reinforces	the
concept	that	the	virtual	memory	space	is	just	an	array	of	bytes	that
the	program	can	subdivide	into	different	storage	units.	It	helps	you
understand	the	effects	of	programs	containing	memory	referencing
errors	such	as	storage	leaks	and	invalid	pointer	references.	Finally,
many	application	programmers	write	their	own	storage	allocators
optimized	toward	the	needs	and	characteristics	of	the	application.
This	chapter,	more	than	any	other,	demonstrates	the	benefit	of
covering	both	the	hardware	and	the	software	aspects	of	computer
systems	in	a	unified	way.	Traditional	computer	architecture	and
operating	systems	texts	present	only	part	of	the	virtual	memory	story.
Chapter	
10
:	System-Level	I/O.	
We	cover	the	basic	concepts	of
Unix	I/O	such	as	files	and	descriptors.	We	describe	how	files	are
shared,	how	I/O	redirection	works,	and	how	to	access	file	metadata.
We	also	develop	a	robust	buffered	I/O	package	that	deals	correctly
with	a	curious	behavior	known	as	
short	counts
,	where	the	library
function	reads	only	part	of	the	input	data.	We	cover	the	C	standard
I/O	library	and	its	relationship	to	Linux	I/O,	focusing	on	limitations	of
standard	I/O	that	make	it	unsuitable	for	network	programming.	In
general,	the	topics	covered	in	this	chapter	are	building	blocks	for	the
next	two	chapters	on	network	and	concurrent	programming.
Chapter	
11
:	Network	Programming.	
Networks	are	interesting	I/O
devices	to	program,	tying	together	many	of	the	ideas	that	we	study

earlier	in	the	text,	such	as	processes,	signals,	byte	ordering,	memory
mapping,	and	dynamic	storage	allocation.	Network	programs	also
provide	a	compelling	context	for	concurrency,	which	is	the	topic	of	the
next	chapter.	This	chapter	is	a	thin	slice	through	network
programming	that	gets	you	to	the	point	where	you	can	write	a	simple
Web	server.	We	cover	the	client-server	model	that	underlies	all
network	applications.	We	present	a	programmer's	view	of	the	Internet
and	show	how	to	write	Internet	clients	and	servers	using	the	sockets
interface.	Finally,	we	introduce	HTTP	and	develop	a	simple	iterative
Web	server.
Chapter	
12
:	Concurrent	Programming.	
This	chapter	introduces
concurrent	programming	using	Internet	server	design	as	the	running
motivational	example.	We	compare	and	contrast	the	three	basic
mechanisms	for	writing	concurrent	programs—processes,	I/O
multiplexing,	and	threads—and	show	how	to	use	them	to	build
concurrent	Internet	servers.	We	cover	basic	principles	of
synchronization	using	
P
	and	
V
	semaphore	operations,	thread	safety
and	reentrancy,	race	conditions,	and	deadlocks.	Writing	concurrent
code	is	essential	for	most	server	applications.	We	also	describe	the
use	of	thread-level	programming	to	express	parallelism	in	an
application	program,	enabling	faster	execution	on	multi-core
processors.	Getting	all	of	the	cores	working	on	a	single	computational
problem	requires	a	careful	coordination	of	the	concurrent	threads,
both	for	correctness	and	to	achieve	high	performance.
New	to	This	Edition

The	first	edition	of	this	book	was	published	with	a	copyright	of	2003,	while
the	second	had	a	copyright	of	2011.	Considering	the	rapid	evolution	of
computer	technology,	the	book	content	has	held	up	surprisingly	well.	Intel
x86	machines	running	C	programs	under	Linux	(and	related	operating
systems)	has	proved	to	be	a	combination	that	continues	to	encompass
many	systems	today.	However,	changes	in	hardware	technology,
compilers,	program	library	interfaces,	and	the	experience	of	many
instructors	teaching	the	material	have	prompted	a	substantial	revision.
The	biggest	overall	change	from	the	second	edition	is	that	we	have
switched	our	presentation	from	one	based	on	a	mix	of	IA32	and	x86-64	to
one	based	exclusively	on	x86-64.	This	shift	in	focus	affected	the	contents
of	many	of	the	chapters.	Here	is	a	summary	of	the	significant	changes.
Chapter	
1
:	A	Tour	of	Computer	Systems	
We	have	moved	the
discussion	of	Amdahl's	Law	from	
Chapter	
5
	into	this	chapter.
Chapter	
2
:	Representing	and	Manipulating	Information.	
A
consistent	bit	of	feedback	from	readers	and	reviewers	is	that	some	of
the	material	in	this	chapter	can	be	a	bit	overwhelming.	So	we	have
tried	to	make	the	material	more	accessible	by	clarifying	the	points	at
which	we	delve	into	a	more	mathematical	style	of	presentation.	This
enables	readers	to	first	skim	over	mathematical	details	to	get	a	high-
level	overview	and	then	return	for	a	more	thorough	reading.
Chapter	
3
:	Machine-Level	Representation	of	Programs.	
We
have	converted	from	the	earlier	presentation	based	on	a	mix	of	IA32
and	x86-64	to	one	based	entirely	on	x86-64.	We	have	also	updated
for	the	style	of	code	generated	by	more	recent	versions	of	
GCC
.	The
result	is	a	substantial	rewriting,	including	changing	the	order	in	which
some	of	the	concepts	are	presented.	We	also	have	included,	for	the

first	time,	a	presentation	of	the	machine-level	support	for	programs
operating	on	floating-point	data.	We	have	created	a	Web	aside
describing	IA32	machine	code	for	legacy	reasons.
Chapter	
4
:	Processor	Architecture.	
We	have	revised	the	earlier
processor	design,	based	on	a	32-bit	architecture,	to	one	that	supports
64-bit	words	and	operations.
Chapter	
5
:	Optimizing	Program	Performance.	
We	have	updated
the	material	to	reflect	the	performance	capabilities	of	recent
generations	of	x86-64	processors.	With	the	introduction	of	more
functional	units	and	more	sophisticated	control	logic,	the	model	of
program	performance	we	developed	based	on	a	data-flow
representation	of	programs	has	become	a	more	reliable	predictor	of
performance	than	it	was	before.
Chapter	
6
:	The	Memory	Hierarchy.	
We	have	updated	the	material
to	reflect	more	recent	technology.
Chapter	
7
:	Linking.	
We	have	rewritten	this	chapter	for	x86-64,
expanded	the	discussion	of	using	the	GOT	and	PLT	to	create
position-independent	code,	and	added	a	new	section	on	a	powerful
linking	technique	known	as	
library	interpositioning.
Chapter	
8
:	Exceptional	Control	Flow.	
We	have	added	a	more
rigorous	treatment	of	signal	handlers,	including	async-signal-safe
functions,	specific	guidelines	for	writing	signal	handlers,	and	using
sigsuspend	to	wait	for	handlers.
Chapter	
9
:	Virtual	Memory.	
This	chapter	has	changed	only
slightly.

Chapter	
10
:	System-Level	I/O.	
We	have	added	a	new	section	on
files	and	the	file	hierarchy,	but	otherwise,	this	chapter	has	changed
only	slightly.
Chapter	
11
:	Network	Programming.	
We	have	introduced
techniques	for	protocol-independent	and	thread-safe	network
programming	using	the	modern	getaddrinfo	and	getnameinfo
functions,	which	replace	the	obsolete	and	non-reentrant
gethostbyname	and	gethostbyaddr	functions.
Chapter	
12
:	Concurrent	Programming.	
We	have	increased	our
coverage	of	using	thread-level	parallelism	to	make	programs	run
faster	on	multi-core	machines.
In	addition,	we	have	added	and	revised	a	number	of	practice	and
homework	problems	throughout	the	text.
Origins	of	the	Book
This	book	stems	from	an	introductory	course	that	we	developed	at
Carnegie	Mellon	University	in	the	fall	of	1998,	called	15−213:	Introduction
to	Computer	Systems	(ICS)	[
14
].	The	ICS	course	has	been	taught	every
semester	since	then.	Over	400	students	take	the	course	each	semester.
The	students	range	from	sophomores	to	graduate	students	in	a	wide
variety	of	majors.	It	is	a	required	core	course	for	all	undergraduates	in	the
CS	and	ECE	departments	at	Carnegie	Mellon,	and	it	has	become	a
prerequisite	for	most	upper-level	systems	courses	in	CS	and	ECE.

The	idea	with	ICS	was	to	introduce	students	to	computers	in	a	different
way.	Few	of	our	students	would	have	the	opportunity	to	build	a	computer
system.	On	the	other	hand,	most	students,	including	all	computer
scientists	and	computer	engineers,	would	be	required	to	use	and
program	computers	on	a	daily	basis.	So	we	decided	to	teach	about
systems	from	the	point	of	view	of	the	programmer,	using	the	following
filter:	we	would	cover	a	topic	only	if	it	affected	the	performance,
correctness,	or	utility	of	user-level	C	programs.
For	example,	topics	such	as	hardware	adder	and	bus	designs	were	out.
Topics	such	as	machine	language	were	in;	but	instead	of	focusing	on
how	to	write	assembly	language	by	hand,	we	would	look	at	how	a	C
compiler	translates	C	constructs	into	machine	code,	including	pointers,
loops,	procedure	calls,	and	switch	statements.	Further,	we	would	take	a
broader	and	more	holistic	view	of	the	system	as	both	hardware	and
systems	software,	covering	such	topics	as	linking,	loading,	
processes,
signals,	performance	optimization,	virtual	memory,	I/O,	and	network	and
concurrent	programming.
This	approach	allowed	us	to	teach	the	ICS	course	in	a	way	that	is
practical,	concrete,	hands-on,	and	exciting	for	the	students.	The
response	from	our	students	and	faculty	colleagues	was	immediate	and
overwhelmingly	positive,	and	we	realized	that	others	outside	of	CMU
might	benefit	from	using	our	approach.	Hence	this	book,	which	we
developed	from	the	ICS	lecture	notes,	and	which	we	have	now	revised	to
reflect	changes	in	technology	and	in	how	computer	systems	are
implemented.
Via	the	multiple	editions	and	multiple	translations	of	this	book,	ICS	and
many	variants	have	become	part	of	the	computer	science	and	computer

engineering	curricula	at	hundreds	of	colleges	and	universities	worldwide.
For	Instructors:	Courses	Based	on
the	Book
Instructors	can	use	the	CS:APP	book	to	teach	a	number	of	different	types
of	systems	courses.	Five	categories	of	these	courses	are	illustrated	in
Figure	
2
.	The	particular	course	depends	on	curriculum	requirements,
personal	taste,	and	the	backgrounds	and	abilities	of	the	students.	From
left	to	right	in	the	figure,	the	courses	are	characterized	by	an	increasing
emphasis	on	the	programmer's	perspective	of	a	system.	Here	is	a	brief
description.
ORG.	
A	computer	organization	course	with	traditional	topics	covered
in	an	un-traditional	style.	Traditional	topics	such	as	logic	design,
processor	architecture,	assembly	language,	and	memory	systems	are
covered.	However,	there	is	more	emphasis	on	the	impact	for	the
programmer.	For	example,	data	representations	are	related	back	to
the	data	types	and	operations	of	C	programs,	and	the	presentation	on
assembly	code	is	based	on	machine	code	generated	by	a	C	compiler
rather	than	handwritten	assembly	code.
ORG+.	
The	ORG	course	with	additional	emphasis	on	the	impact	of
hardware	on	the	performance	of	application	programs.	Compared	to
ORG,	students	learn	more	about	code	optimization	and	about
improving	the	memory	performance	of	their	C	programs.

ICS.	
The	baseline	ICS	course,	designed	to	produce	enlightened
programmers	who	understand	the	impact	of	the	hardware,	operating
system,	and	compilation	system	on	the	performance	and	correctness
of	their	application	programs.	A	significant	difference	from	ORG+	is
that	low-level	processor	architecture	is	not	covered.	Instead,
programmers	work	with	a	higher-level	model	of	a	modern	out-of-order
processor.	The	ICS	course	fits	nicely	into	a	10-week	quarter,	and	can
also	be	stretched	to	a	15-week	semester	if	covered	at	a	more
leisurely	pace.
ICS+.	
The	baseline	ICS	course	with	additional	coverage	of	systems
programming	topics	such	as	system-level	I/O,	network	programming,
and	concurrent	programming.	This	is	the	semester-long	Carnegie
Mellon	course,	which	covers	every	chapter	in	CS:APP	except	low-
level	processor	architecture.
Course
Chapter
Topic
ORG
ORG+
ICS
ICS+
SP
1
Tour	of	systems
•
•
•
•
•
2
Data	representation
•
•
•
•
⊙
3
Machine	language
•
•
•
•
•
4
Processor	architecture
•
•
5
Code	optimization
•
•
•
6
Memory	hierarchy
⊙
•
•
•
⊙
7
Linking
⊙
⊙
•
(d)
(a)
(a)
(c)
(d)

8
Exceptional	control	flow
•
•
•
9
Virtual	memory
⊙
•
•
•
•
10
System-level	I/O
•
•
11
Network	programming
•
•
12
Concurrent	programming
•
•
Figure	
2	
Five	systems	courses	based	on	the	CS:APP	book.
ICS+	is	the	15−213	course	from	Carnegie	Mellon.	Notes:	The	
(c)
symbol	denotes	partial	coverage	of	a	chapter,	as	follows:	(a)
hardware	only;	(b)	no	dynamic	storage	allocation;	(c)	no	dynamic
linking;	(d)	no	floating	point.
SP.	
A	systems	programming	course.	This	course	is	similar	to	ICS+,
but	it	drops	floating	point	and	performance	optimization,	and	it	places
more	emphasis	on	systems	programming,	including	process	control,
dynamic	linking,	system-level	I/O,	network	programming,	and
concurrent	programming.	Instructors	might	want	to	supplement	from
other	sources	for	advanced	topics	such	as	daemons,	terminal	control,
and	Unix	IPC.
The	main	message	of	
Figure	
2
	is	that	the	CS:APP	book	gives	a	lot	of
options	to	students	and	instructors.	If	you	want	your	students	to	be
exposed	to	lower-level	processor	architecture,	then	that	option	is
available	via	the	ORG	and	ORG+	courses.	On	the	other	hand,	if	you
want	to	switch	from	your	current	computer	organization	course	to	an	ICS
or	ICS+	course,	but	are	wary	of	making	such	a	drastic	change	all	at	once,
then	you	can	move	toward	ICS	incrementally.	You	can	start	with	ORG,
(b)

which	teaches	the	traditional	topics	in	a	nontraditional	way.	Once	you	are
comfortable	with	that	material,	then	you	can	move	to	ORG+,	and
eventually	to	ICS.	If	students	have	no	experience	in	C	(e.g.,	they	have
only	programmed	in	Java),	you	could	spend	several	weeks	on	C	and
then	cover	the	material	of	ORG	or	ICS.
Finally,	we	note	that	the	ORG+	and	SP	courses	would	make	a	nice	two-
term	sequence	(either	quarters	or	semesters).	Or	you	might	consider
offering	ICS+	as	one	term	of	ICS	and	one	term	of	SP.
For	Instructors:	Classroom-Tested
Laboratory	Exercises
The	ICS+	course	at	Carnegie	Mellon	receives	very	high	evaluations	from
students.	Median	scores	of	5.0/5.0	and	means	of	4.6/5.0	are	typical	for
the	student	course	evaluations.	Students	cite	the	fun,	exciting,	and
relevant	laboratory	exercises	as	the	primary	reason.	The	labs	are
available	from	the	CS:APP	Web	page.	Here	are	examples	of	the	labs	that
are	provided	with	the	book.
Data	Lab.	
This	lab	requires	students	to	implement	simple	logical	and
arithmetic	functions,	but	using	a	highly	restricted	subset	of	C.	For
example,	they	must	compute	the	absolute	value	of	a	number	using
only	bit-level	operations.	This	lab	helps	students	understand	the	bit-
level	representations	of	C	data	types	and	the	bit-level	behavior	of	the
operations	on	data.

Binary	Bomb	Lab.	
A	
binary	bomb
	is	a	program	provided	to	students
as	an	object-code	file.	When	run,	it	prompts	the	user	to	type	in	six
different	strings.	If	any	of	these	are	incorrect,	the	bomb	"explodes,"
printing	an	error	message	and	logging	the	event	on	a	grading	server.
Students	must	"defuse"	their	own	unique	bombs	by	disassembling
and	reverse	engineering	the	programs	to	determine	what	the	six
strings	should	be.	The	lab	teaches	students	to	understand	assembly
language	and	also	forces	them	to	learn	how	to	use	a	debugger.
Buffer	Overflow	Lab.	
Students	are	required	to	modify	the	run-time
behavior	of	a	binary	executable	by	exploiting	a	buffer	overflow
vulnerability.	This	lab	teaches	the	students	about	the	stack	discipline
and	about	the	danger	of	writing	code	that	is	vulnerable	to	buffer
overflow	attacks.
Architecture	Lab.	
Several	of	the	homework	problems	of	
Chapter
4
	can	be	combined	into	a	lab	assignment,	where	students	modify
the	HCL	description	of	a	processor	to	add	new	instructions,	change
the	branch	prediction	policy,	or	add	or	remove	bypassing	paths	and
register	ports.	The	resulting	processors	can	be	simulated	and	run
through	automated	tests	that	will	detect	most	of	the	possible	bugs.
This	lab	lets	students	experience	the	exciting	parts	of	processor
design	without	requiring	a	complete	background	in	logic	design	and
hardware	description	languages.
Performance	Lab.	
Students	must	optimize	the	performance	of	an
application	kernel	function	such	as	convolution	or	matrix	transposition.
This	lab	provides	a	very	clear	demonstration	of	the	properties	of
cache	memories	and	gives	students	experience	with	low-level
program	optimization.
Cache	Lab.	
In	this	alternative	to	the	performance	lab,	students	write	a

general-purpose	cache	simulator,	and	then	optimize	a	small	matrix
transpose	kernel	to	minimize	the	number	of	misses	on	a	simulated
cache.	We	use	the	Valgrind	tool	to	generate	real	address	traces	for
the	matrix	transpose	kernel.
Shell	Lab.	
Students	implement	their	own	Unix	shell	program	with	job
control,	including	the	Ctrl+C	and	Ctrl+Z	keystrokes	and	the	
and	
	commands.	
This	is	the	student's	first	introduction	to
concurrency,	and	it	gives	them	a	clear	idea	of	Unix	process	control,
signals,	and	signal	handling.
Malloc	Lab.	
Students	implement	their	own	versions	of	
and	(optionally)	
	This	lab	gives	students	a	clear
understanding	of	data	layout	and	organization,	and	requires	them	to
evaluate	different	trade-offs	between	space	and	time	efficiency.
Proxy	Lab.	
Students	implement	a	concurrent	Web	proxy	that	sits
between	their	browsers	and	the	rest	of	the	World	Wide	Web.	This	lab
exposes	the	students	to	such	topics	as	Web	clients	and	servers,	and
ties	together	many	of	the	concepts	from	the	course,	such	as	byte
ordering,	file	I/O,	process	control,	signals,	signal	handling,	memory
mapping,	sockets,	and	concurrency.	Students	like	being	able	to	see
their	programs	in	action	with	real	Web	browsers	and	Web	servers.
The	CS:APP	instructor's	manual	has	a	detailed	discussion	of	the	labs,	as
well	as	directions	for	downloading	the	support	software.
Acknowledgments	for	the	Third

Edition
It	is	a	pleasure	to	acknowledge	and	thank	those	who	have	helped	us
produce	this	third	edition	of	the	CS:APP	text.
We	would	like	to	thank	our	Carnegie	Mellon	colleagues	who	have	taught
the	ICS	course	over	the	years	and	who	have	provided	so	much	insightful
feedback	and	encouragement:	Guy	Blelloch,	Roger	Dannenberg,	David
Eckhardt,	Franz	Franchetti,	Greg	Ganger,	Seth	Goldstein,	Khaled	Harras,
Greg	Kesden,	Bruce	Maggs,	Todd	Mowry,	Andreas	Nowatzyk,	Frank
Pfenning,	Markus	Pueschel,	and	Anthony	Rowe.	David	Winters	was	very
helpful	in	installing	and	configuring	the	reference	Linux	box.
Jason	Fritts	(St.	Louis	University)	and	Cindy	Norris	(Appalachian	State)
provided	us	with	detailed	and	thoughtful	reviews	of	the	second	edition.
Yili	Gong	(Wuhan	University)	wrote	the	Chinese	translation,	maintained
the	errata	page	for	the	Chinese	version,	and	contributed	many	bug
reports.	Godmar	Back	(Virginia	Tech)	helped	us	improve	the	text
significantly	by	introducing	us	to	the	notions	of	async-signal	safety	and
protocol-independent	network	programming.
Many	thanks	to	our	eagle-eyed	readers	who	reported	bugs	in	the	second
edition:	Rami	Ammari,	Paul	Anagnostopoulos,	Lucas	Bärenfänger,
Godmar	Back,	Ji	Bin,	Sharbel	Bousemaan,	Richard	Callahan,	Seth
Chaiken,	Cheng	Chen,	Libo	Chen,	Tao	Du,	Pascal	Garcia,	Yili	Gong,
Ronald	Greenberg,	Dorukhan	Gülöz,	Dong	Han,	Dominik	Helm,	Ronald
Jones,	Mustafa	Kazdagli,	Gordon	Kindlmann,	Sankar	Krishnan,	Kanak
Kshetri,	Junlin	Lu,	Qiangqiang	Luo,	Sebastian	Luy,	Lei	Ma,	Ashwin

Nanjappa,	Gregoire	Paradis,	Jonas	Pfenninger,	Karl	Pichotta,	David
Ramsey,	Kaustabh	Roy,	David	Selvaraj,	Sankar	Shanmugam,	Dominique
Smulkowska,	Dag	Sørbø,	Michael	Spear,	Yu	Tanaka,	Steven
Tricanowicz,	Scott	Wright,	Waiki	Wright,	Han	Xu,	Zhengshan	Yan,	Firo
Yang,	Shuang	Yang,	John	Ye,	Taketo	Yoshida,	Yan	Zhu,	and	Michael
Zink.
Thanks	also	to	our	readers	who	have	contributed	to	the	labs,	including
God-mar	Back	(Virginia	Tech),	Taymon	Beal	(Worcester	Polytechnic
Institute),	Aran	Clauson	(Western	Washington	University),	Cary	Gray
(Wheaton	College),	Paul	Haiduk	(West	Texas	A&M	University),	Len
Hamey	(Macquarie	University),	Eddie	Kohler	(Harvard),	Hugh	Lauer
(Worcester	Polytechnic	Institute),	Robert	Marmorstein	(Longwood
University),	and	James	Riely	(DePaul	University).
Once	again,	Paul	Anagnostopoulos	of	Windfall	Software	did	a	masterful
job	of	typesetting	the	book	and	leading	the	production	process.	Many
thanks	to	Paul	and	his	stellar	team:	Richard	Camp	(copyediting),	Jennifer
McClain	(proofreading),	Laurel	Muller	(art	production),	and	Ted	Laux
(indexing).	Paul	even	spotted	a	bug	in	our	description	of	the	origins	of	the
acronym	BSS	that	had	persisted	undetected	since	the	first	edition!
Finally,	we	would	like	to	thank	our	friends	at	Prentice	Hall.	Marcia	Horton
and	our	editor,	Matt	Goldstein,	have	been	unflagging	in	their	support	and
encouragement,	and	we	are	deeply	grateful	to	them.
Acknowledgments	from	the	Second

Edition
We	are	deeply	grateful	to	the	many	people	who	have	helped	us	produce
this	second	edition	of	the	CS:APP	text.
First	and	foremost,	we	would	like	to	recognize	our	colleagues	who	have
taught	the	ICS	course	at	Carnegie	Mellon	for	their	insightful	feedback	and
encouragement:	Guy	Blelloch,	Roger	Dannenberg,	David	Eckhardt,	Greg
Ganger,	Seth	Goldstein,	Greg	Kesden,	Bruce	Maggs,	Todd	Mowry,
Andreas	Nowatzyk,	Frank	Pfenning,	and	Markus	Pueschel.
Thanks	also	to	our	sharp-eyed	readers	who	contributed	reports	to	the
errata	page	for	the	first	edition:	Daniel	Amelang,	Rui	Baptista,	Quarup
Barreirinhas,	Michael	Bombyk,	Jörg	Brauer,	Jordan	Brough,	Yixin	Cao,
James	Caroll,	Rui	Carvalho,	Hyoung-Kee	Choi,	Al	Davis,	Grant	Davis,
Christian	Dufour,	Mao	Fan,	Tim	Freeman,	Inge	Frick,	Max	Gebhardt,	Jeff
Goldblat,	Thomas	Gross,	Anita	Gupta,	John	Hampton,	Hiep	Hong,	Greg
Israelsen,	Ronald	Jones,	Haudy	Kazemi,	Brian	Kell,	Constantine
Kousoulis,	Sacha	Krakowiak,	Arun	Krishnaswamy,	Martin	Kulas,	Michael
Li,	Zeyang	Li,	Ricky	Liu,	Mario	Lo	Conte,	Dirk	Maas,	Devon	Macey,	Carl
Marcinik,	Will	Marrero,	Simone	Martins,	Tao	Men,	Mark	Morrissey,
Venkata	Naidu,	Bhas	Nalabothula,	Thomas	Niemann,	Eric	Peskin,	David
Po,	Anne	Rogers,	John	Ross,	Michael	Scott,	Seiki,	Ray	Shih,	Darren
Shultz,	Erik	Silkensen,	Suryanto,	Emil	Tarazi,	Nawanan	Theera-
Ampornpunt,	Joe	Trdinich,	Michael	Trigoboff,	James	Troup,	Martin
Vopatek,	Alan	West,	Betsy	Wolff,	Tim	Wong,	James	Woodruff,	Scott
Wright,	Jackie	Xiao,	Guanpeng	Xu,	Qing	Xu,	Caren	Yang,	Yin
Yongsheng,	Wang	Yuanxuan,	Steven	Zhang,	and	Day	Zhong.	Special

thanks	to	Inge	Frick,	who	identified	a	subtle	deep	copy	bug	in	our	lock-
and-copy	example,	and	to	Ricky	Liu	for	his	amazing	proofreading	skills.
Our	Intel	Labs	colleagues	Andrew	Chien	and	Limor	Fix	were
exceptionally	supportive	throughout	the	writing	of	the	text.	Steve
Schlosser	graciously	provided	some	disk	drive	characterizations.	Casey
Helfrich	and	Michael	Ryan	installed	
and	maintained	our	new	Core	i7	box.
Michael	Kozuch,	Babu	Pillai,	and	Jason	Campbell	provided	valuable
insight	on	memory	system	performance,	multi-core	systems,	and	the
power	wall.	Phil	Gibbons	and	Shimin	Chen	shared	their	considerable
expertise	on	solid	state	disk	designs.
We	have	been	able	to	call	on	the	talents	of	many,	including	Wen-Mei
Hwu,	Markus	Pueschel,	and	Jiri	Simsa,	to	provide	both	detailed
comments	and	high-level	advice.	James	Hoe	helped	us	create	a	Verilog
version	of	the	Y86	processor	and	did	all	of	the	work	needed	to	synthesize
working	hardware.
Many	thanks	to	our	colleagues	who	provided	reviews	of	the	draft
manuscript:	James	Archibald	(Brigham	Young	University),	Richard
Carver	(George	Mason	University),	Mirela	Damian	(Villanova	University),
Peter	Dinda	(Northwestern	University),	John	Fiore	(Temple	University),
Jason	Fritts	(St.	Louis	University),	John	Greiner	(Rice	University),	Brian
Harvey	(University	of	California,	Berkeley),	Don	Heller	(Penn	State
University),	Wei	Chung	Hsu	(University	of	Minnesota),	Michelle	Hugue
(University	of	Maryland),	Jeremy	Johnson	(Drexel	University),	Geoff
Kuenning	(Harvey	Mudd	College),	Ricky	Liu,	Sam	Madden	(MIT),	Fred
Martin	(University	of	Massachusetts,	Lowell),	Abraham	Matta	(Boston
University),	Markus	Pueschel	(Carnegie	Mellon	University),	Norman

Ramsey	(Tufts	University),	Glenn	Reinmann	(UCLA),	Michela	Taufer
(University	of	Delaware),	and	Craig	Zilles	(UIUC).
Paul	Anagnostopoulos	of	Windfall	Software	did	an	outstanding	job	of
typesetting	the	book	and	leading	the	production	team.	Many	thanks	to
Paul	and	his	superb	team:	Rick	Camp	(copyeditor),	Joe	Snowden
(compositor),	MaryEllen	N.	Oliver	(proofreader),	Laurel	Muller	(artist),	and
Ted	Laux	(indexer).
Finally,	we	would	like	to	thank	our	friends	at	Prentice	Hall.	Marcia	Horton
has	always	been	there	for	us.	Our	editor,	Matt	Goldstein,	provided	stellar
leadership	from	beginning	to	end.	We	are	profoundly	grateful	for	their
help,	encouragement,	and	insights.
Acknowledgments	from	the	First
Edition
We	are	deeply	indebted	to	many	friends	and	colleagues	for	their
thoughtful	criticisms	and	encouragement.	A	special	thanks	to	our	15−213
students,	whose	infectious	energy	and	enthusiasm	spurred	us	on.	Nick
Carter	and	Vinny	Furia	generously	provided	their	malloc	package.
Guy	Blelloch,	Greg	Kesden,	Bruce	Maggs,	and	Todd	Mowry	taught	the
course	over	multiple	semesters,	gave	us	encouragement,	and	helped
improve	the	course	material.	Herb	Derby	provided	early	spiritual
guidance	and	encouragement.	Allan	Fisher,	Garth	Gibson,	Thomas
Gross,	Satya,	Peter	Steenkiste,	and	Hui	Zhang	encouraged	us	to

develop	the	course	from	the	start.	A	suggestion	from	Garth	early	on	got
the	whole	ball	rolling,	and	this	was	picked	up	and	refined	with	the	help	of
a	group	led	by	Allan	Fisher.	Mark	Stehlik	and	Peter	Lee	have	been	very
supportive	about	building	this	material	into	the	undergraduate	curriculum.
Greg	Kesden	provided	helpful	feedback	on	the	impact	of	ICS	on	the	OS
course.	Greg	Ganger	and	Jiri	Schindler	graciously	provided	some	disk
drive	characterizations	
and	answered	our	questions	on	modern	disks.
Tom	Stricker	showed	us	the	memory	mountain.	James	Hoe	provided
useful	ideas	and	feedback	on	how	to	present	processor	architecture.
A	special	group	of	students—Khalil	Amiri,	Angela	Demke	Brown,	Chris
Colohan,	Jason	Crawford,	Peter	Dinda,	Julio	Lopez,	Bruce	Lowekamp,
Jeff	Pierce,	Sanjay	Rao,	Balaji	Sarpeshkar,	Blake	Scholl,	Sanjit	Seshia,
Greg	Steffan,	Tiankai	Tu,	Kip	Walker,	and	Yinglian	Xie—were
instrumental	in	helping	us	develop	the	content	of	the	course.	In	particular,
Chris	Colohan	established	a	fun	(and	funny)	tone	that	persists	to	this	day,
and	invented	the	legendary	"binary	bomb"	that	has	proven	to	be	a	great
tool	for	teaching	machine	code	and	debugging	concepts.
Chris	Bauer,	Alan	Cox,	Peter	Dinda,	Sandhya	Dwarkadas,	John	Greiner,
Don	Heller,	Bruce	Jacob,	Barry	Johnson,	Bruce	Lowekamp,	Greg
Morrisett,	Brian	Noble,	Bobbie	Othmer,	Bill	Pugh,	Michael	Scott,	Mark
Smotherman,	Greg	Steffan,	and	Bob	Wier	took	time	that	they	did	not
have	to	read	and	advise	us	on	early	drafts	of	the	book.	A	very	special
thanks	to	Al	Davis	(University	of	Utah),	Peter	Dinda	(Northwestern
University),	John	Greiner	(Rice	University),	Wei	Hsu	(University	of
Minnesota),	Bruce	Lowekamp	(College	of	William	&	Mary),	Bobbie
Othmer	(University	of	Minnesota),	Michael	Scott	(University	of
Rochester),	and	Bob	Wier	(Rocky	Mountain	College)	for	class	testing	the
beta	version.	A	special	thanks	to	their	students	as	well!

We	would	also	like	to	thank	our	colleagues	at	Prentice	Hall.	Marcia
Horton,	Eric	Frank,	and	Harold	Stone	have	been	unflagging	in	their
support	and	vision.	Harold	also	helped	us	present	an	accurate	historical
perspective	on	RISC	and	CISC	processor	architectures.	Jerry	Ralya
provided	sharp	insights	and	taught	us	a	lot	about	good	writing.
Finally,	we	would	like	to	acknowledge	the	great	technical	writers	Brian
Kernighan	and	the	late	W.	Richard	Stevens,	for	showing	us	that	technical
books	can	be	beautiful.
Thank	you	all.
Randy	Bryant
Dave	O'Hallaron
Pittsburgh,	Pennsylvania

About	the	Authors
Randal	E.	Bryant
	received	his	bachelor's	degree	from	the	University	of
Michigan	in	1973	and	then	attended	graduate	school	at	the
Massachusetts	Institute	of	Technology,	receiving	his	PhD	degree	in
computer	science	in	1981.	He	spent	three	years	as	an	assistant
professor	at	the	California	Institute	of	Technology,	and	has	been	on	the
faculty	at	Carnegie	Mellon	since	1984.	For	five	of	those	years	he	served
as	head	of	the	Computer	Science	Department,	and	for	ten	of	them	he
served	as	Dean	of	the	School	of	Computer	Science.	He	is	currently	a
university	professor	of	computer	science.	He	also	holds	a	courtesy
appointment	with	the	Department	of	Electrical	and	Computer
Engineering.
Professor	Bryant	has	taught	courses	in	computer	systems	at	both	the
undergraduate	and	graduate	level	for	around	40	years.	Over	many	years
of	teaching	computer	architecture	courses,	he	began	shifting	the	focus
from	how	computers	are	designed	to	how	programmers	can	write	more
efficient	and	reliable	programs	if	they	understand	the	system	better.
Together	with	Professor	O'Hallaron,	he	developed	the	course	15−213,

Introduction	to	Computer	Systems,	at	Carnegie	Mellon	that	is	the	basis
for	this	book.	He	has	also	taught	courses	in	algorithms,	programming,
computer	networking,	distributed	systems,	and	VLSI	design.
Most	of	Professor	Bryant's	research	concerns	the	design	of	software
tools	to	help	software	and	hardware	designers	verify	the	correctness	of
their	systems.	These	include	several	types	of	simulators,	as	well	as
formal	verification	tools	that	prove	the	correctness	of	a	design	using
mathematical	methods.	He	has	published	over	150	technical	papers.	His
research	results	are	used	by	major	computer	manufacturers,	including
Intel,	IBM,	Fujitsu,	and	Microsoft.	He	has	won	several	major	awards	for
his	research.	These	include	two	inventor	recognition	awards	and	a
technical	achievement	award	from	the	Semiconductor	Research
Corporation,	the	Kanellakis	Theory	and	Practice	Award	from	the
Association	for	Computer	Machinery	(ACM),	and	the	W.	R.	G.	Baker
Award,	the	Emmanuel	Piore	Award,	the	Phil	Kaufman	Award,	and	the	A.
Richard	Newton	Award	from	the	Institute	of	Electrical	and	Electronics
Engineers	(IEEE).	He	is	a	fellow	of	both	the	ACM	and	the	IEEE	and	a
member	of	both	the	US	National	Academy	of	Engineering	and	the
American	Academy	of	Arts	and	Sciences.


David	R.	O'Hallaron
	is	a	professor	of	computer	science	and	electrical
and	computer	engineering	at	Carnegie	Mellon	University.	He	received	his
PhD	from	the	University	of	Virginia.	He	served	as	the	director	of	Intel
Labs,	Pittsburgh,	from	2007	to	2010.
He	has	taught	computer	systems	courses	at	the	undergraduate	and
graduate	levels	for	20	years	on	such	topics	as	computer	architecture,
introductory	computer	systems,	parallel	processor	design,	and	Internet
services.	Together	with	Professor	Bryant,	he	developed	the	course	at
Carnegie	Mellon	that	led	to	this	book.	In	2004,	he	was	awarded	the
Herbert	Simon	Award	for	Teaching	Excellence	by	the	CMU	School	of
Computer	Science,	an	award	for	which	the	winner	is	chosen	based	on	a
poll	of	the	students.
Professor	O'Hallaron	works	in	the	area	of	computer	systems,	with
specific	interests	in	software	systems	for	scientific	computing,	data-
intensive	computing,	and	virtualization.	The	best-known	example	of	his
work	is	the	Quake	project,	an	endeavor	involving	a	group	of	computer
scientists,	civil	engineers,	and	seismologists	who	have	developed	the
ability	to	predict	the	motion	of	the	ground	during	strong	earthquakes.	In
2003,	Professor	O'Hallaron	and	the	other	members	of	the	Quake	team
won	the	Gordon	Bell	Prize,	the	top	international	prize	in	high-
performance	computing.	His	current	work	focuses	on	the	notion	of
autograding,	that	is,	programs	that	evaluate	the	quality	of	other
programs.

Chapter	
1	
A	Tour	of	Computer
Systems
1.1	
Information	Is	Bits	+	Context	
3
1.2	
Programs	Are	Translated	by	Other	Programs	into	Different
Forms	
4
1.3	
It	Pays	to	Understand	How	Compilation	Systems	Work	
6
1.4	
Processors	Read	and	Interpret	Instructions	Stored	in	Memory	
7
1.5	
Caches	Matter	
11
1.6	
Storage	Devices	Form	a	Hierarchy	
14
1.7	
The	Operating	System	Manages	the	Hardware	
14
1.8	
Systems	Communicate	with	Other	Systems	Using	Networks	
19
1.9	
Important	Themes	
22
1.10	
Summary
	
27
Bibliographic	Notes	
28
Solutions	to	Practice	Problems	
28

A	
computer	system
	consists	of	hardware	and
systems	software	that	work	together	to	run
application	programs.	Specific	implementations	of
systems	change	over	time,	but	the	underlying
concepts	do	not.	All	computer	systems	have	similar
hardware	and	software	components	that	perform
similar	functions.	This	book	is	written	for
programmers	who	want	to	get	better	at	their	craft	by
understanding	how	these	components	work	and
how	they	affect	the	correctness	and	performance	of
their	programs.
You	are	poised	for	an	exciting	journey.	If	you
dedicate	yourself	to	learning	the	concepts	in	this
book,	then	you	will	be	on	your	way	to	be	coming	a
rare	"power	programmer,"	enlightened	by	an
understanding	of	the	underlying	computer	system
and	its	impact	on	your	application	programs.
You	are	going	to	learn	practical	skills	such	as	how	to
avoid	strange	numerical	errors	caused	by	the	way
that	computers	represent	numbers.	You	will	learn
how	to	optimize	your	C	code	by	using	clever	tricks
that	exploit	the	designs	of	modern	processors	and
memory	systems.	You	will	learn	how	the	compiler
implements	procedure	calls	and	how	to	use	this
knowledge	to	avoid	the	security	holes	from	buffer
overflow	vulnerabilities	that	plague	network	and
Internet	software.	You	will	learn	how	to	recognize

and	avoid	the	nasty	errors	during	linking	that
confound	the	average	programmer.	You	will	learn
how	to	write	your	own	Unix	shell,	your	own	dynamic
storage	allocation	package,	and	even	your	own	Web
server.	You	will	learn	the	promises	and	pitfalls	of
concurrency,	a	topic	of	increasing	importance	as
multiple	processor	cores	are	integrated	onto	single
chips.
In	their	classic	text	on	the	C	programming	language
[
61
],	Kernighan	and	Ritchie	introduce	readers	to	C
using	the	
	program	shown	in	
Figure	
1.1
.
Although	
	is	a	very	simple	program,	every
major	part	of	the	system	must	work	in	concert	in
order	for	it	to	run	to	completion.	In	a	sense,	the	goal
of	this	book	is	to	help	you	understand	what	happens
and	why	when	you	run	
	on	your	system.
We	begin	our	study	of	systems	by	tracing	the
lifetime	of	the	
	program,	from	the	time	it	is
created	by	a	programmer,	until	it	runs	on	a	system,
prints	its	simple	message,	and	terminates.	As	we
follow	the	lifetime	of	the	program,	we	will	briefly
introduce	the	key	concepts,	terminology,	and
components	that	come	into	play.	Later	chapters	will
expand	on	these	ideas.
-------------------------------------------
code/intro/hello.c

-------------------------------------------
code/intro/hello.c
Figure	
1.1	
The	
	program.
(
Source:
	[
60
])

Figure	
1.2	
The	ASCII	text	representation	of

1.1	
Information	Is	Bits	+	Context
Our	
	program	begins	life	as	a	
source	program
	(or	
source	file
)	that
the	programmer	creates	with	an	editor	and	saves	in	a	text	file	called
	The	source	program	is	a	sequence	of	bits,	each	with	a	value	of
0	or	1,	organized	in	8-bit	chunks	called	
bytes
.	Each	byte	represents	some
text	character	in	the	program.
Most	computer	systems	represent	text	characters	using	the	ASCII
standard	that	represents	each	character	with	a	unique	byte-size	integer
value.
	For	example,	
Figure	
1.2
	shows	the	ASCII	representation	of	the
	program.
1.	
Other	encoding	methods	are	used	to	represent	text	in	non-English	languages.	See	the	aside	on
page	50	for	a	discussion	on	this.
The	
	program	is	stored	in	a	file	as	a	sequence	of	bytes.	Each
byte	has	an	integer	value	that	corresponds	to	some	character.	For
example,	the	first	byte	has	the	integer	value	35,	which	corresponds	to	the
character	`
'.	The	second	byte	has	the	integer	value	105,	which
corresponds	to	the	character	
,	and	so	on.	Notice	that	each	text	line	is
terminated	by	the	invisible	
newline
	character	
,	which	is	represented
by	the	integer	value	10.	Files	such	as	
	that	consist	exclusively	of
ASCII	characters	are	known	as	
text	files
.	All	other	files	are	known	as
binary	files
.
1

The	representation	of	
	illustrates	a	fundamental	idea:	All
information	in	a	system—including	disk	files,	programs	stored	in	memory,
user	data	stored	in	memory,	and	data	transferred	across	a	network—is
represented	as	a	bunch	of	bits.	The	only	thing	that	distinguishes	different
data	objects	is	the	context	in	which	we	view	them.	For	example,	in
different	contexts,	the	same	sequence	of	bytes	might	represent	an
integer,	floating-point	number,	character	string,	or	machine	instruction.
As	programmers,	we	need	to	understand	machine	representations	of
numbers	because	they	are	not	the	same	as	integers	and	real	numbers.
They	are	finite
Aside	
Origins	of	the	C	programming
language
C	was	developed	from	1969	to	1973	by	Dennis	Ritchie	of	Bell
Laboratories.	The	American	National	Standards	Institute	(ANSI)
ratified	the	ANSI	C	standard	in	1989,	and	this	standardization	later
became	the	responsibility	of	the	International	Standards
Organization	(ISO).	The	standards	define	the	C	language	and	a
set	of	library	functions	known	as	the	
C	standard	library
.	Kernighan
and	Ritchie	describe	ANSI	C	in	their	classic	book,	which	is	known
affectionately	as	"K&R"	[
61
].	In	Ritchie's	words	[
92
],	C	is	"quirky,
flawed,	and	an	enormous	success."	So	why	the	success?
C	was	closely	tied	with	the	Unix	operating	system.	
C	was
developed	from	the	beginning	as	the	system	programming
language	for	Unix.	Most	of	the	Unix	kernel	(the	core	part	of	the
operating	system),	and	all	of	its	supporting	tools	and	libraries,

were	written	in	C.	As	Unix	became	popular	in	universities	in
the	late	1970s	and	early	1980s,	many	people	were	exposed	to
C	and	found	that	they	liked	it.	Since	Unix	was	written	almost
entirely	in	C,	it	could	be	easily	ported	to	new	machines,	which
created	an	even	wider	audience	for	both	C	and	Unix.
C	is	a	small,	simple	language.	
The	design	was	controlled	by
a	single	person,	rather	than	a	committee,	and	the	result	was	a
clean,	consistent	design	with	little	baggage.	The	K&R	book
describes	the	complete	language	and	standard	library,	with
numerous	examples	and	exercises,	in	only	261	pages.	The
simplicity	of	C	made	it	relatively	easy	to	learn	and	to	port	to
different	computers.
C	was	designed	for	a	practical	purpose.	
C	was	designed	to
implement	the	Unix	operating	system.	Later,	other	people
found	that	they	could	write	the	programs	they	wanted,	without
the	language	getting	in	the	way.
C	is	the	language	of	choice	for	system-level	programming,	and
there	is	a	huge	installed	base	of	application-level	programs	as
well.	However,	it	is	not	perfect	for	all	programmers	and	all
situations.	C	pointers	are	a	common	source	of	confusion	and
programming	errors.	C	also	lacks	explicit	support	for	useful
abstractions	such	as	classes,	objects,	and	exceptions.	Newer
languages	such	as	C++	and	Java	address	these	issues	for
application-level	programs.
approximations	that	can	behave	in	unexpected	ways.	This	fundamental
idea	is	explored	in	detail	in	
Chapter	
2
.

1.2	
Programs	Are	Translated	by
Other	Programs	into	Different	Forms
The	
	program	begins	life	as	a	high-level	C	program	because	it	can
be	read	and	understood	by	human	beings	in	that	form.	However,	in	order
to	run	
	on	the	system,	the	individual	C	statements	must	be
translated	by	other	programs	into	a	sequence	of	low-level	
machine-
language
	instructions.	These	instructions	are	then	packaged	in	a	form
called	an	
executable	object	program
	and	stored	as	a	binary	disk	file.
Object	programs	are	also	referred	to	as	
executable	object	files
.
On	a	Unix	system,	the	translation	from	source	file	to	object	file	is
performed	by	a	
compiler	driver:
Figure	
1.3	
The	compilation	system.

Here,	the	
GCC
	
compiler	driver	reads	the	source	file	
	and	translates
it	into	an	executable	object	file	
.	The	translation	is	performed	in	the
sequence	of	four	phases	shown	in	
Figure	
1.3
.	The	programs	that
perform	the	four	phases	(
preprocessor
,	
compiler
,	
assembler
,	and	
linker
)
are	known	collectively	as	the	
compilation	system
.
Preprocessing	phase.	
The	preprocessor	(cpp)	modifies	the	original
C	program	according	to	directives	that	begin	with	the	`
'	character.
For	example,	the	
	command	in	line	1	of	
tells	the	preprocessor	to	read	the	contents	of	the	system	header	file
	and	insert	it	directly	into	the	program	text.	The	result	is
another	C	program,	typically	with	the	
	suffix.
Compilation	phase.	
The	compiler	(
)	translates	the	text	file
	into	the	text	file	
,	which	contains	an	
assembly-
language	program
.	This	program	includes	the	following	definition	of
:
Each	of	lines	2-7	in	this	definition	describes	one	low-level	machine-
language	instruction	in	a	textual	form.	Assembly	language	is	useful

because	it	provides	a	common	output	language	for	different	compilers
for	different	high-level	languages.	For	example,	C	compilers	and
Fortran	compilers	both	generate	output	files	in	the	same	assembly
language.
Assembly	phase.	
Next,	the	assembler	(
)	translates	
	into
machine-language	instructions,	packages	them	in	a	form	known	as	a
relocatable	object	program
,	and	stores	the	result	in	the	object	file
	This	file	is	a	binary	file	containing	17	bytes	to	encode	the
instructions	for	function	main.	If	we	were	to	view	
	with	a	text
editor,	it	would	appear	to	be	gibberish.
Aside	
The	GNU	project
G
CC
	
is	one	of	many	useful	tools	developed	by	the	GNU	(short
for	GNU's	Not	Unix)	project.	The	GNU	project	is	a	tax-exempt
charity	started	by	Richard	Stallman	in	1984,	with	the	ambitious
goal	of	developing	a	complete	Unix-like	system	whose	source
code	is	unencumbered	by	restrictions	on	how	it	can	be
modified	or	distributed.	The	GNU	project	has	developed	an
environment	with	all	the	major	components	of	a	Unix	operating
system,	except	for	the	kernel,	which	was	developed	separately
by	the	Linux	project.	The	GNU	environment	includes	the	
EMACS
editor,	
GCC
	
compiler,	
GDB
	
debugger,	assembler,	linker,	utilities
for	manipulating	binaries,	and	other	components.	The	
GCC
compiler	has	grown	to	support	many	different	languages,	with
the	ability	to	generate	code	for	many	different	machines.
Supported	languages	include	C,	C++,	Fortran,	Java,	Pascal,
Objective-C,	and	Ada.
The	GNU	project	is	a	remarkable	achievement,	and	yet	it	is
often	overlooked.	The	modern	open-source	movement

(commonly	associated	with	Linux)	owes	its	intellectual	origins
to	the	GNU	project's	notion	of	
free	software
	("free"	as	in	"free
speech,"	not	"free	beer").	Further,	Linux	owes	much	of	its
popularity	to	the	GNU	tools,	which	provide	the	environment	for
the	Linux	kernel.
Linking	phase.	
Notice	Notice	that	our	
	program	calls	the	
function,	which	is	part	of	the	
standard	C	library
	provided	by	every	C
compiler.	The	
	function	resides	in	a	separate	precompiled
object	file	called	
,	which	must	somehow	be	merged	with	our
	program.	The	linker	(
)	handles	this	merging.	The	result	is
the	
	file,	which	is	an	executable	object	file	(or	simply	
executable
)
that	is	ready	to	be	loaded	into	memory	and	executed	by	the	system.

1.3	
It	Pays	to	Understand	How
Compilation	Systems	Work
For	simple	programs	such	as	
,	we	can	rely	on	the	compilation
system	to	produce	correct	and	efficient	machine	code.	However,	there
are	some	important	reasons	why	programmers	need	to	understand	how
compilation	systems	work:
Optimizing	program	performance.	
Modern	compilers	are
sophisticated	tools	that	usually	produce	good	code.	As	programmers,
we	do	not	need	to	know	the	inner	workings	of	the	compiler	in	order	to
write	efficient	code.	However,	in	order	to	make	good	coding	decisions
in	our	C	programs,	we	do	need	a	basic	understanding	of	machine-
level	code	and	how	the	compiler	translates	different	C	statements	into
machine	code.	For	example,	is	a	
	statement	always	more
efficient	than	a	sequence	of	
	statements?	How	much
overhead	is	incurred	by	a	function	call?	Is	a	
	loop	more	efficient
than	a	
	loop?	Are	pointer	references	more	efficient	than	array
indexes?	Why	does	our	loop	run	so	much	faster	if	we	sum	into	a	local
variable	instead	of	an	argument	that	is	passed	by	reference?	How	can
a	function	run	faster	when	we	simply	rearrange	the	parentheses	in	an
arithmetic	expression?
In	
Chapter	
3
,	we	introduce	x86-64,	the	machine	language	of	recent
generations	of	Linux,	Macintosh,	and	Windows	computers.	We
describe	how	compilers	translate	different	C	constructs	into	this

language.	In	
Chapter	
5
,	you	will	learn	how	to	tune	the	performance
of	your	C	programs	by	making	simple	transformations	to	the	C	code
that	help	the	compiler	do	its	job	better.	In	
Chapter	
6
,	you	will	learn
about	the	hierarchical	nature	of	the	memory	system,	how	C	compilers
store	data	arrays	in	memory,	and	how	your	C	programs	can	exploit
this	knowledge	to	run	more	efficiently.
Understanding	link-time	errors.	
In	our	experience,	some	of	the
most	perplexing	programming	errors	are	related	to	the	operation	of
the	linker,	especially	when	you	are	trying	to	build	large	software
systems.	For	example,	what	does	it	mean	when	the	linker	reports	that
it	cannot	resolve	a	reference?	What	is	the	difference	between	a	static
variable	and	a	global	variable?	What	happens	if	you	define	two	global
variables	in	different	C	files	with	the	same	name?	What	is	the
difference	between	a	static	library	and	a	dynamic	library?	Why	does	it
matter	what	order	we	list	libraries	on	the	command	line?	And	scariest
of	all,	why	do	some	linker-related	errors	not	appear	until	run	time?
You	will	learn	the	answers	to	these	kinds	of	questions	in	
Chapter	
7
.
Avoiding	security	holes.	
For	many	years,	
buffer	overflow
vulnerabilities
	have	accounted	for	many	of	the	security	holes	in
network	and	Internet	servers.	These	vulnerabilities	exist	because	too
few	programmers	understand	the	need	to	carefully	restrict	the
quantity	and	forms	of	data	they	accept	from	untrusted	sources.	A	first
step	in	learning	secure	programming	is	to	understand	the
consequences	of	the	way	data	and	control	information	are	stored	on
the	program	stack.	We	cover	the	stack	discipline	and	buffer	overflow
vulnerabilities	in	
Chapter	
3
	as	part	of	our	study	of	assembly
language.	We	will	also	learn	about	methods	that	can	be	used	by	the
programmer,	compiler,	and	operating	system	to	reduce	the	threat	of
attack.

1.4	
Processors	Read	and	Interpret
Instructions	Stored	in	Memory
At	this	point,	our	
	source	program	has	been	translated	by	the
compilation	system	into	an	executable	object	file	called	
	that	is
stored	on	disk.	To	run	the	executable	file	on	a	Unix	system,	we	type	its
name	to	an	application	program	known	as	a	
shell:
The	shell	is	a	command-line	interpreter	that	prints	a	prompt,	waits	for	you
to	type	a	command	line,	and	then	performs	the	command.	If	the	first	word
of	the	command	line	does	not	correspond	to	a	built-in	shell	command,
then	the	shell

Figure	
1.4	
Hardware	organization	of	a	typical	system.
CPU:	central	processing	unit,	ALU:	arithmetic/logic	unit,	PC:	program
counter,	USB:	Universal	Serial	Bus.
assumes	that	it	is	the	name	of	an	executable	file	that	it	should	load	and
run.	So	in	this	case,	the	shell	loads	and	runs	the	
	program	and	then
waits	for	it	to	terminate.	The	
	program	prints	its	message	to	the
screen	and	then	terminates.	The	shell	then	prints	a	prompt	and	waits	for
the	next	input	command	line.
1.4.1	
Hardware	Organization	of	a
System
To	understand	what	happens	to	our	
	program	when	we	run	it,	we
need	to	understand	the	hardware	organization	of	a	typical	system,	which
is	shown	in	
Figure	
1.4
.	This	particular	picture	is	modeled	after	the

family	of	recent	Intel	systems,	but	all	systems	have	a	similar	look	and
feel.	Don't	worry	about	the	complexity	of	this	figure	just	now.	We	will	get
to	its	various	details	in	stages	throughout	the	course	of	the	book.
Buses
Running	throughout	the	system	is	a	collection	of	electrical	conduits	called
buses
	that	carry	bytes	of	information	back	and	forth	between	the
components.	Buses	are	typically	designed	to	transfer	fixed-size	chunks	of
bytes	known	as	
words
.	The	number	of	bytes	in	a	word	(the	
word	size
)	is
a	fundamental	system	parameter	that	varies	across	systems.	Most
machines	today	have	word	sizes	of	either	4	bytes	(32	bits)	or	8	bytes	(64
bits).	In	this	book,	we	do	not	assume	any	fixed	definition	of	word	size.
Instead,	we	will	specify	what	we	mean	by	a	"word"	in	any	context	that
requires	this	to	be	defined.
I/O	Devices
Input/output	(I/O)	devices	are	the	system's	connection	to	the	external
world.	Our	example	system	has	four	I/O	devices:	a	keyboard	and	mouse
for	user	input,	a	display	for	user	output,	and	a	disk	drive	(or	simply	disk)
for	long-term	storage	of	data	and	programs.	Initially,	the	executable	
program	resides	on	the	disk.
Each	I/O	device	is	connected	to	the	I/O	bus	by	either	a	
controller
	or	an
adapter
.	The	distinction	between	the	two	is	mainly	one	of	packaging.
Controllers	are	chip	sets	in	the	device	itself	or	on	the	system's	main
printed	circuit	board	(often	called	the	
motherboard
).	An	adapter	is	a	card
that	plugs	into	a	slot	on	the	motherboard.	Regardless,	the	purpose	of

each	is	to	transfer	information	back	and	forth	between	the	I/O	bus	and	an
I/O	device.
Chapter	
6
	has	more	to	say	about	how	I/O	devices	such	as	disks	work.
In	
Chapter	
10
,	you	will	learn	how	to	use	the	Unix	I/O	interface	to
access	devices	from	your	application	programs.	We	focus	on	the
especially	interesting	class	of	devices	known	as	networks,	but	the
techniques	generalize	to	other	kinds	of	devices	as	well.
Main	Memory
The	
main	memory
	is	a	temporary	storage	device	that	holds	both	a
program	and	the	data	it	manipulates	while	the	processor	is	executing	the
program.	Physically,	main	memory	consists	of	a	collection	of	
dynamic
random	access	memory
(DRAM)	chips.	Logically,	memory	is	organized	as
a	linear	array	of	bytes,	each	with	its	own	unique	address	(array	index)
starting	at	zero.	In	general,	each	of	the	machine	instructions	that
constitute	a	program	can	consist	of	a	variable	number	of	bytes.	The	sizes
of	data	items	that	correspond	to	C	program	variables	vary	according	to
type.	For	example,	on	an	x86-64	machine	running	Linux,	data	of	type
	require	2	bytes,	types	
	and	
	4	bytes,	and	types	
	and
	8	bytes.
Chapter	
6
	has	more	to	say	about	how	memory	technologies	such	as
DRAM	chips	work,	and	how	they	are	combined	to	form	main	memory.
Processor

The	
central	processing	unit
	(CPU),	or	simply	
processor
,	is	the	engine	that
interprets	(or	
executes
)	instructions	stored	in	main	memory.	At	its	core	is
a	word-size	storage	device	(or	
register
)	called	the	
program	counter
	(PC).
At	any	point	in	time,	the	PC	points	at	(contains	the	address	of)	some
machine-language	instruction	in	main	memory.
2.	
PC	is	also	a	commonly	used	acronym	for	"personal	computer."	However,	the	distinction
between	the	two	should	be	clear	from	the	context.
From	the	time	that	power	is	applied	to	the	system	until	the	time	that	the
power	is	shut	off,	a	processor	repeatedly	executes	the	instruction	pointed
at	by	the	program	counter	and	updates	the	program	counter	to	point	to
the	next	instruction.	A	processor	
appears
	to	operate	according	to	a	very
simple	instruction	execution	model,	defined	by	its	
instruction	set
architecture
.	In	this	model,	instructions	execute	
in	strict	sequence,	and
executing	a	single	instruction	involves	performing	a	series	of	steps.	The
processor	reads	the	instruction	from	memory	pointed	at	by	the	program
counter	(PC),	interprets	the	bits	in	the	instruction,	performs	some	simple
operation	dictated	by	the	instruction,	and	then	updates	the	PC	to	point	to
the	next	instruction,	which	may	or	may	not	be	contiguous	in	memory	to
the	instruction	that	was	just	executed.
There	are	only	a	few	of	these	simple	operations,	and	they	revolve	around
main	memory,	the	
register	file
,	and	the	
arithmetic/logic	unit
	(ALU).	The
register	file	is	a	small	storage	device	that	consists	of	a	collection	of	word-
size	registers,	each	with	its	own	unique	name.	The	ALU	computes	new
data	and	address	values.	Here	are	some	examples	of	the	simple
operations	that	the	CPU	might	carry	out	at	the	request	of	an	instruction:
2

Load:	
Copy	a	byte	or	a	word	from	main	memory	into	a	register,
overwriting	the	previous	contents	of	the	register.
Store:	
Copy	a	byte	or	a	word	from	a	register	to	a	location	in	main
memory,	overwriting	the	previous	contents	of	that	location.
Operate:	
Copy	the	contents	of	two	registers	to	the	ALU,	perform	an
arithmetic	operation	on	the	two	words,	and	store	the	result	in	a
register,	overwriting	the	previous	contents	of	that	register.
Jump:	
Extract	a	word	from	the	instruction	itself	and	copy	that	word
into	the	program	counter	(PC),	overwriting	the	previous	value	of	the
PC.
We	say	that	a	processor	appears	to	be	a	simple	implementation	of	its
instruction	set	architecture,	but	in	fact	modern	processors	use	far	more
complex	mechanisms	to	speed	up	program	execution.	Thus,	we	can
distinguish	the	processor's	instruction	set	architecture,	describing	the
effect	of	each	machine-code	instruction,	from	its	
microarchitecture
,
describing	how	the	processor	is	actually	implemented.	When	we	study
machine	code	in	
Chapter	
3
,	we	will	consider	the	abstraction	provided
by	the	machine's	instruction	set	architecture.	
Chapter	
4
	has	more	to
say	about	how	processors	are	actually	implemented.	
Chapter	
5
describes	a	model	of	how	modern	processors	work	that	enables
predicting	and	optimizing	the	performance	of	machine-language
programs.
1.4.2	
Running	the	
	Program
Given	this	simple	view	of	a	system's	hardware	organization	and
operation,	we	can	begin	to	understand	what	happens	when	we	run	our

example	program.	We	must	omit	a	lot	of	details	here	that	will	be	filled	in
later,	but	for	now	we	will	be	content	with	the	big	picture.
Initially,	the	shell	program	is	executing	its	instructions,	waiting	for	us	to
type	a	command.	As	we	type	the	characters	
	at	the	keyboard,	the
shell	program	reads	each	one	into	a	register	and	then	stores	it	in
memory,	as	shown	in	
Figure	
1.5
.
When	we	hit	the	enter	key	on	the	keyboard,	the	shell	knows	that	we	have
finished	typing	the	command.	The	shell	then	loads	the	executable	
file	by	executing	a	sequence	of	instructions	that	copies	the	code	and	data
in	the	
Figure	
1.5	
Reading	the	
	command	from	the	keyboard.
object	file	from	disk	to	main	memory.	The	data	includes	the	string	of
characters	
	that	will	eventually	be	printed	out.

Using	a	technique	known	as	
direct	memory	access
	(DMA,	discussed	in
Chapter	
6
),	the	data	travel	directly	from	disk	to	main	memory,	without
passing	through	the	processor.	This	step	is	shown	in	
Figure	
1.6
.
Once	the	code	and	data	in	the	
	object	file	are	loaded	into	memory,
the	processor	begins	executing	the	machine-language	instructions	in	the
	program's	
	routine.	These	instructions	copy	the	bytes	in	the
	string	from	memory	to	the	register	file,	and	from	there	to
the	display	device,	where	they	are	displayed	on	the	screen.	This	step	is
shown	in	
Figure	
1.7
.

1.5	
Caches	Matter
An	important	lesson	from	this	simple	example	is	that	a	system	spends	a
lot	of	time	moving	information	from	one	place	to	another.	The	machine
instructions	in	the	
	program	are	originally	stored	on	disk.	When	the
program	is	loaded,	they	are	copied	to	main	memory.	As	the	processor
runs	the	program,	instructions	are	copied	from	main	memory	into	the
processor.	Similarly,	the	data	string	
,	originally	on	disk,	is
copied	to	main	memory	and	then	copied	from	main	memory	to	the
display	device.	From	a	programmer's	perspective,	much	of	this	copying	is
overhead	that	slows	down	the	"real	work"	of	the	program.	Thus,	a	major
goal	for	system	designers	is	to	make	these	copy	operations	run	as	fast
as	possible.
Because	of	physical	laws,	larger	storage	devices	are	slower	than	smaller
storage	devices.	And	faster	devices	are	more	expensive	to	build	than
their	slower

Figure	
1.6	
Loading	the	executable	from	disk	into	main	memory.
Figure	
1.7	
Writing	the	output	string	from	memory	to	the	display.

Figure	
1.8	
Cache	memories.
counterparts.	For	example,	the	disk	drive	on	a	typical	system	might	be
1,000	times	larger	than	the	main	memory,	but	it	might	take	the	processor
10,000,000	times	longer	to	read	a	word	from	disk	than	from	memory.
Similarly,	a	typical	register	file	stores	only	a	few	hundred	bytes	of
information,	as	opposed	to	billions	of	bytes	in	the	main	memory.
However,	the	processor	can	read	data	from	the	register	file	almost	100
times	faster	than	from	memory.	Even	more	troublesome,	as
semiconductor	technology	progresses	over	the	years,	this	
processor-
memory	gap
	continues	to	increase.	It	is	easier	and	cheaper	to	make
processors	run	faster	than	it	is	to	make	main	memory	run	faster.
To	deal	with	the	processor-memory	gap,	system	designers	include
smaller,	faster	storage	devices	called	
cache	memories
	(or	simply	caches)
that	serve	as	temporary	staging	areas	for	information	that	the	processor
is	likely	to	need	in	the	near	future.	
Figure	
1.8
	shows	the	cache
memories	in	a	typical	system.	An	
L1	cache
	on	the	processor	chip	holds
tens	of	thousands	of	bytes	and	can	be	accessed	nearly	as	fast	as	the
register	file.	A	larger	
L2	cache
	with	hundreds	of	thousands	to	millions	of
bytes	is	connected	to	the	processor	by	a	special	bus.	It	might	take	5
times	longer	for	the	processor	to	access	the	L2	cache	than	the	L1	cache,

but	this	is	still	5	to	10	times	faster	than	accessing	the	main	memory.	The
L1	and	L2	caches	are	implemented	with	a	hardware	technology	known
as	
static	random	access	memory
	(SRAM).	Newer	and	more	powerful
systems	even	have	three	levels	of	cache:	L1,	L2,	and	L3.	The	idea
behind	caching	is	that	a	system	can	get	the	effect	of	both	a	very	large
memory	and	a	very	fast	one	by	exploiting	
locality
,	the	tendency	for
programs	to	access	data	and	code	in	localized	regions.	By	setting	up
caches	to	hold	data	that	are	likely	to	be	accessed	often,	we	can	perform
most	memory	operations	using	the	fast	caches.
One	of	the	most	important	lessons	in	this	book	is	that	application
programmers	who	are	aware	of	cache	memories	can	exploit	them	to
improve	the	performance	of	their	programs	by	an	order	of	magnitude.
You	will	learn	more	about	these	important	devices	and	how	to	exploit
them	in	
Chapter	
6
.
Figure	
1.9	
An	example	of	a	memory	hierarchy.

1.6	
Storage	Devices	Form	a
Hierarchy
This	notion	of	inserting	a	smaller,	faster	storage	device	(e.g.,	cache
memory)	between	the	processor	and	a	larger,	slower	device	(e.g.,	main
memory)	turns	out	to	be	a	general	idea.	In	fact,	the	storage	devices	in
every	computer	system	are	organized	as	a	
memory	hierarchy
	similar	to
Figure	
1.9
.	As	we	move	from	the	top	of	the	hierarchy	to	the	bottom,
the	devices	become	slower,	larger,	and	less	costly	per	byte.	The	register
file	occupies	the	top	level	in	the	hierarchy,	which	is	known	as	level	0	or
L0.	We	show	three	levels	of	caching	L1	to	L3,	occupying	memory
hierarchy	levels	1	to	3.	Main	memory	occupies	level	4,	and	so	on.
The	main	idea	of	a	memory	hierarchy	is	that	storage	at	one	level	serves
as	a	cache	for	storage	at	the	next	lower	level.	Thus,	the	register	file	is	a
cache	for	the	L1	cache.	Caches	L1	and	L2	are	caches	for	L2	and	L3,
respectively.	The	L3	cache	is	a	cache	for	the	main	memory,	which	is	a
cache	for	the	disk.	On	some	networked	systems	with	distributed	file
systems,	the	local	disk	serves	as	a	cache	for	data	stored	on	the	disks	of
other	systems.
Just	as	programmers	can	exploit	knowledge	of	the	different	caches	to
improve	performance,	programmers	can	exploit	their	understanding	of
the	entire	memory	hierarchy.	
Chapter	
6
	will	have	much	more	to	say
about	this.

1.7	
The	Operating	System	Manages
the	Hardware
Back	to	our	
	example.	When	the	shell	loaded	and	ran	the	
program,	and	when	the	
	program	printed	its	message,	neither
program	accessed	the
Figure	
1.10	
Layered	view	of	a	computer	system.
Figure	
1.11	
Abstractions	provided	by	an	operating	system.
keyboard,	display,	disk,	or	main	memory	directly.	Rather,	they	relied	on
the	services	provided	by	the	
operating	system
.	We	can	think	of	the
operating	system	as	a	layer	of	software	interposed	between	the
application	program	and	the	hardware,	as	shown	in	
Figure	
1.10
.	All
attempts	by	an	application	program	to	manipulate	the	hardware	must	go
through	the	operating	system.

The	operating	system	has	two	primary	purposes:	(1)	to	protect	the
hardware	from	misuse	by	runaway	applications	and	(2)	to	provide
applications	with	simple	and	uniform	mechanisms	for	manipulating
complicated	and	often	wildly	different	low-level	hardware	devices.	The
operating	system	achieves	both	goals	via	the	fundamental	abstractions
shown	in	
Figure	
1.11
:	
processes
,	
virtual	memory
,	and	
files
.	As	this
figure	suggests,	files	are	abstractions	for	I/O	devices,	virtual	memory	is
an	abstraction	for	both	the	main	memory	and	disk	I/O	devices,	and
processes	are	abstractions	for	the	processor,	main	memory,	and	I/O
devices.	We	will	discuss	each	in	turn.
1.7.1	
Processes
When	a	program	such	as	
	runs	on	a	modern	system,	the	operating
system	provides	the	illusion	that	the	program	is	the	only	one	running	on
the	system.	The	program	appears	to	have	exclusive	use	of	both	the
processor,	main	memory,	and	I/O	devices.	The	processor	appears	to
execute	the	instructions	in	the	program,	one	after	the	other,	without
interruption.	And	the	code	and	data	of	the	program	appear	to	be	the	only
objects	in	the	system's	memory.	These	illusions	are	provided	by	the
notion	of	a	process,	one	of	the	most	important	and	successful	ideas	in
computer	science.
A	
process
	is	the	operating	system's	abstraction	for	a	running	program.
Multiple	processes	can	run	concurrently	on	the	same	system,	and	each
process	appears	to	have	exclusive	use	of	the	hardware.	By	
concurrently
,
we	mean	that	the	instructions	of	one	process	are	interleaved	with	the

instructions	of	another	process.	In	most	systems,	there	are	more
processes	to	run	than	there	are	CPUs	to	run	them.
Aside	
Unix,	Posix,	and	the	Standard
Unix	Specification
The	1960s	was	an	era	of	huge,	complex	operating	systems,	such
as	IBM's	OS/360	and	Honeywell's	Multics	systems.	While	OS/360
was	one	of	the	most	successful	software	projects	in	history,
Multics	dragged	on	for	years	and	never	achieved	wide-scale	use.
Bell	Laboratories	was	an	original	partner	in	the	Multics	project	but
dropped	out	in	1969	because	of	concern	over	the	complexity	of
the	project	and	the	lack	of	progress.	In	reaction	to	their	unpleasant
Multics	experience,	a	group	of	Bell	Labs	researchers—Ken
Thompson,	Dennis	Ritchie,	Doug	McIlroy,	and	Joe	Ossanna—
began	work	in	1969	on	a	simpler	operating	system	for	a	Digital
Equipment	Corporation	PDP-7	computer,	written	entirely	in
machine	language.	Many	of	the	ideas	in	the	new	system,	such	as
the	hierarchical	file	system	and	the	notion	of	a	shell	as	a	user-
level	process,	were	borrowed	from	Multics	but	implemented	in	a
smaller,	simpler	package.	In	1970,	Brian	Kernighan	dubbed	the
new	system	"Unix"	as	a	pun	on	the	complexity	of	"Multics."	The
kernel	was	rewritten	in	C	in	1973,	and	Unix	was	announced	to	the
outside	world	in	1974	[
93
].
Because	Bell	Labs	made	the	source	code	available	to	schools
with	generous	terms,	Unix	developed	a	large	following	at
universities.	The	most	influential	work	was	done	at	the	University
of	California	at	Berkeley	in	the	late	1970s	and	early	1980s,	with

Berkeley	researchers	adding	virtual	memory	and	the	Internet
protocols	in	a	series	of	releases	called	Unix	4.xBSD	(Berkeley
Software	Distributimn).	Concurrently,	Bell	Labs	was	releasing	their
own	versions,	which	became	known	as	System	V	Unix.	Versions
from	other	vendors,	such	as	the	Sun	Microsystems	Solaris
system,	were	derived	from	these	original	BSD	and	System	V
versions.
Trouble	arose	in	the	mid	1980s	as	Unix	vendors	tried	to
differentiate	themselves	by	adding	new	and	often	incompatible
features.	To	combat	this	trend,	IEEE	(Institute	for	Electrical	and
Electronics	Engineers)	sponsored	an	effort	to	standardize	Unix,
later	dubbed	"Posix"	by	Richard	Stallman.	The	result	was	a	family
of	standards,	known	as	the	Posix	standards,	that	cover	such
issues	as	the	C	language	interface	for	Unix	system	calls,	shell
programs	and	utilities,	threads,	and	network	programming.	More
recently,	a	separate	standardization	effort,	known	as	the
"Standard	Unix	Specification,"	has	joined	forces	with	Posix	to
create	a	single,	unified	standard	for	Unix	systems.	As	a	result	of
these	standardization	efforts,	the	differences	between	Unix
versions	have	largely	disappeared.
Traditional	systems	could	only	execute	one	program	at	a	time,	while
newer	
multi-core
	processors	can	execute	several	programs
simultaneously.	In	either	case,	a	single	CPU	can	appear	to	execute
multiple	processes	concurrently	by	having	the	processor	switch	among
them.	The	operating	system	performs	this	interleaving	with	a	mechanism
known	as	
context	switching
.	To	simplify	the	rest	of	this	discussion,	we
consider	only	a	
uniprocessor	system
	containing	a	single	CPU.	We	will
return	to	the	discussion	of	
multiprocessor
	systems	in	
Section	
1.9.2
.

The	operating	system	keeps	track	of	all	the	state	information	that	the
process	needs	in	order	to	run.	This	state,	which	is	known	as	the	
context
,
includes	information	such	as	the	current	values	of	the	PC,	the	register
file,	and	the	contents	of	main	memory.	At	any	point	in	time,	a
uniprocessor	system	can	only	execute	the	code	for	a	single	process.
When	the	operating	system	decides	to	transfer	control	from	the	current
process	to	some	new	process,	it	performs	a	
context	switch
	by	saving	the
context	of	the	current	process,	restoring	the	context	of	the	new	process,
and
Figure	
1.12	
Process	context	switching.
then	passing	control	to	the	new	process.	The	new	process	picks	up
exactly	where	it	left	off.	
Figure	
1.12
	shows	the	basic	idea	for	our
example	
	scenario.
There	are	two	concurrent	processes	in	our	example	scenario:	the	shell
process	and	the	
	process.	Initially,	the	shell	process	is	running
alone,	waiting	for	input	on	the	command	line.	When	we	ask	it	to	run	the
	program,	the	shell	carries	out	our	request	by	invoking	a	special
function	known	as	a	
system	call
	that	passes	control	to	the	operating
system.	The	operating	system	saves	the	shell's	context,	creates	a	new
	process	and	its	context,	and	then	passes	control	to	the	new	

process.	After	
	terminates,	the	operating	system	restores	the
context	of	the	shell	process	and	passes	control	back	to	it,	where	it	waits
for	the	next	command-line	input.
As	
Figure	
1.12
	indicates,	the	transition	from	one	process	to	another	is
managed	by	the	operating	system	
kernel
.	The	kernel	is	the	portion	of	the
operating	system	code	that	is	always	resident	in	memory.	When	an
application	program	requires	some	action	by	the	operating	system,	such
as	to	read	or	write	a	file,	it	executes	a	special	
system	call
	instruction,
transferring	control	to	the	kernel.	The	kernel	then	performs	the	requested
operation	and	returns	back	to	the	application	program.	Note	that	the
kernel	is	not	a	separate	process.	Instead,	it	is	a	collection	of	code	and
data	structures	that	the	system	uses	to	manage	all	the	processes.
Implementing	the	process	abstraction	requires	close	cooperation
between	both	the	low-level	hardware	and	the	operating	system	software.
We	will	explore	how	this	works,	and	how	applications	can	create	and
control	their	own	processes,	in	
Chapter	
8
.
1.7.2	
Threads
Although	we	normally	think	of	a	process	as	having	a	single	control	flow,
in	modern	systems	a	process	can	actually	consist	of	multiple	execution
units,	called	
threads
,	each	running	in	the	context	of	the	process	and
sharing	the	same	code	and	global	data.	Threads	are	an	increasingly
important	programming	model	because	of	the	requirement	for
concurrency	in	network	servers,	because	it	is	easier	to	share	data
between	multiple	threads	than	between	multiple	processes,	and	because

threads	are	typically	more	efficient	than	processes.	Multi-threading	is	also
one	way	to	make	programs	run	faster	when	multiple	processors	are
available,	as	we	will	discuss	in
Figure	
1.13	
Process	virtual	address	space.
(The	regions	are	not	drawn	to	scale.)
Section	
1.9.2
.	You	will	learn	the	basic	concepts	of	concurrency,
including	how	to	write	threaded	programs,	in	
Chapter	
12
.
1.7.3	
Virtual	Memory
Virtual	memory
	is	an	abstraction	that	provides	each	process	with	the
illusion	that	it	has	exclusive	use	of	the	main	memory.	Each	process	has
the	same	uniform	view	of	memory,	which	is	known	as	its	
virtual	address
space
.	The	virtual	address	space	for	Linux	processes	is	shown	in	
Figure

1.13
.	(Other	Unix	systems	use	a	similar	layout.)	In	Linux,	the	topmost
region	of	the	address	space	is	reserved	for	code	and	data	in	the
operating	system	that	is	common	to	all	processes.	The	lower	region	of
the	address	space	holds	the	code	and	data	defined	by	the	user's
process.	Note	that	addresses	in	the	figure	increase	from	the	bottom	to
the	top.
The	virtual	address	space	seen	by	each	process	consists	of	a	number	of
well-defined	areas,	each	with	a	specific	purpose.	You	will	learn	more
about	these	areas	later	in	the	book,	but	it	will	be	helpful	to	look	briefly	at
each,	starting	with	the	lowest	addresses	and	working	our	way	up:
Program	code	and	data.	
Code	begins	at	the	same	fixed	address	for
all	processes,	followed	by	data	locations	that	correspond	to	global	C
variables.	The	code	and	data	areas	are	initialized	directly	from	the
contents	of	an	executable	object	file—in	our	case,	the	
executable.	You	will	learn	more	about	this	part	of	the	address	space
when	we	study	linking	and	loading	in	
Chapter	
7
.
Heap.	
The	code	and	data	areas	are	followed	immediately	by	the	run-
time	
heap
.	Unlike	the	code	and	data	areas,	which	are	fixed	in	size
once	the	process	begins	
running,	the	heap	expands	and	contracts
dynamically	at	run	time	as	a	result	of	calls	to	C	standard	library
routines	such	as	
	and	
.	We	will	study	heaps	in	detail	when
we	learn	about	managing	virtual	memory	in	
Chapter	
9
.
Shared	libraries.	
Near	the	middle	of	the	address	space	is	an	area
that	holds	the	code	and	data	for	
shared	libraries
	such	as	the	C
standard	library	and	the	math	library.	The	notion	of	a	shared	library	is
a	powerful	but	somewhat	difficult	concept.	You	will	learn	how	they
work	when	we	study	dynamic	linking	in	
Chapter	
7
.

Stack.	
At	the	top	of	the	user's	virtual	address	space	is	the	
user	stack
that	the	compiler	uses	to	implement	function	calls.	Like	the	heap,	the
user	stack	expands	and	contracts	dynamically	during	the	execution	of
the	program.	In	particular,	each	time	we	call	a	function,	the	stack
grows.	Each	time	we	return	from	a	function,	it	contracts.	You	will	learn
how	the	compiler	uses	the	stack	in	
Chapter	
3
.
Kernel	virtual	memory.	
The	top	region	of	the	address	space	is
reserved	for	the	kernel.	Application	programs	are	not	allowed	to	read
or	write	the	contents	of	this	area	or	to	directly	call	functions	defined	in
the	kernel	code.	Instead,	they	must	invoke	the	kernel	to	perform	these
operations.
For	virtual	memory	to	work,	a	sophisticated	interaction	is	required
between	the	hardware	and	the	operating	system	software,	including	a
hardware	translation	of	every	address	generated	by	the	processor.	The
basic	idea	is	to	store	the	contents	of	a	process's	virtual	memory	on	disk
and	then	use	the	main	memory	as	a	cache	for	the	disk.	
Chapter	
9
explains	how	this	works	and	why	it	is	so	important	to	the	operation	of
modern	systems.
1.7.4	
Files
A	
file
	is	a	sequence	of	bytes,	nothing	more	and	nothing	less.	Every	I/O
device,	including	disks,	keyboards,	displays,	and	even	networks,	is
modeled	as	a	file.	All	input	and	output	in	the	system	is	performed	by
reading	and	writing	files,	using	a	small	set	of	system	calls	known	as	
Unix
I/O
.

This	simple	and	elegant	notion	of	a	file	is	nonetheless	very	powerful
because	it	provides	applications	with	a	uniform	view	of	all	the	varied	I/O
devices	that	might	be	contained	in	the	system.	For	example,	application
programmers	who	manipulate	the	contents	of	a	disk	file	are	blissfully
unaware	of	the	specific	disk	technology.	Further,	the	same	program	will
run	on	different	systems	that	use	different	disk	technologies.	You	will
learn	about	Unix	I/O	in	
Chapter	
10
.

1.8	
Systems	Communicate	with
Other	Systems	Using	Networks
Up	to	this	point	in	our	tour	of	systems,	we	have	treated	a	system	as	an
isolated	collection	of	hardware	and	software.	In	practice,	modern	systems
are	often	linked	to	other	systems	by	networks.	From	the	point	of	view	of
an	individual	system,	the
Aside	
The	Linux	project
In	August	1991,	a	Finnish	graduate	student	named	Linus	Torvalds
modestly	announced	a	new	Unix-like	operating	system	kernel:

As	Torvalds	indicates,	his	starting	point	for	creating	Linux	was
Minix,	an	operating	system	developed	by	Andrew	S.	Tanenbaum
for	educational	purposes	[
113
].
The	rest,	as	they	say,	is	history.	Linux	has	evolved	into	a	technical
and	cultural	phenomenon.	By	combining	forces	with	the	GNU
project,	the	Linux	project	has	developed	a	complete,	Posix-
compliant	version	of	the	Unix	operating	system,	including	the
kernel	and	all	of	the	supporting	infrastructure.	Linux	is	available	on
a	wide	array	of	computers,	from	handheld	devices	to	mainframe
computers.	A	group	at	IBM	has	even	ported	Linux	to	a	wristwatch!
network	can	be	viewed	as	just	another	I/O	device,	as	shown	in	
Figure
1.14
.	When	the	system	copies	a	sequence	of	bytes	from	main	memory
to	the	network	adapter,	the	data	flow	across	the	network	to	another
machine,	instead	of,	say,	to	a	local	disk	drive.	Similarly,	the	system	can
read	data	sent	from	other	machines	and	copy	these	data	to	its	main
memory.
With	the	advent	of	global	networks	such	as	the	Internet,	copying
information	from	one	machine	to	another	has	become	one	of	the	most

important	uses	of	computer	systems.	For	example,	applications	such	as
email,	instant	messaging,	the	World	Wide	Web,	FTP,	and	telnet	are	all
based	on	the	ability	to	copy	information	over	a	network.
Figure	
1.14	
A	network	is	another	I/O	device.
Figure	
1.15	
Using	telnet	to	run	
	remotely	over	a	network.
Returning	to	our	
	example,	we	could	use	the	familiar	telnet
application	to	run	
	on	a	remote	machine.	Suppose	we	use	a	telnet
client
	running	on	our	local	machine	to	connect	to	a	telnet	
server
	on	a
remote	machine.	After	we	log	in	to	the	remote	machine	and	run	a	shell,
the	remote	shell	is	waiting	to	receive	an	input	command.	From	this	point,

running	the	
	program	remotely	involves	the	five	basic	steps	shown
in	
Figure	
1.15
.
After	we	type	in	the	
	string	to	the	telnet	client	and	hit	the	enter	key,
the	client	sends	the	string	to	the	telnet	server.	After	the	telnet	server
receives	the	string	from	the	network,	it	passes	it	along	to	the	remote	shell
program.	Next,	the	remote	shell	runs	the	
	program	and	passes	the
output	line	back	to	the	telnet	server.	Finally,	the	telnet	server	forwards	the
output	string	across	the	network	to	the	telnet	client,	which	prints	the
output	string	on	our	local	terminal.
This	type	of	exchange	between	clients	and	servers	is	typical	of	all
network	applications.	In	
Chapter	
11
	you	will	learn	how	to	build	network
applications	and	apply	this	knowledge	to	build	a	simple	Web	server.

1.9	
Important	Themes
This	concludes	our	initial	whirlwind	tour	of	systems.	An	important	idea	to
take	away	from	this	discussion	is	that	a	system	is	more	than	just
hardware.	It	is	a	collection	of	intertwined	hardware	and	systems	software
that	must	cooperate	in	order	to	achieve	the	ultimate	goal	of	running
application	programs.	The	rest	of	this	book	will	fill	in	some	details	about
the	hardware	and	the	software,	and	it	will	show	how,	by	knowing	these
details,	you	can	write	programs	that	are	faster,	more	reliable,	and	more
secure.
To	close	out	this	chapter,	we	highlight	several	important	concepts	that	cut
across	all	aspects	of	computer	systems.	We	will	discuss	the	importance
of	these	concepts	at	multiple	places	within	the	book.
1.9.1	
Amdahl's	Law
Gene	Amdahl,	one	of	the	early	pioneers	in	computing,	made	a	simple	but
insightful	observation	about	the	effectiveness	of	improving	the
performance	of	one	part	of	a	system.	This	observation	has	come	to	be
known	as	
Amdahl's	law.
	The	main	idea	is	that	when	we	speed	up	one
part	of	a	system,	the	effect	on	the	overall	system	performance	depends
on	both	how	significant	this	part	was	and	how	much	it	sped	up.	Consider
a	system	in	which	executing	some	application	requires	time	
T
.	Suppose
some	part	of	the	system	requires	a	fraction	α	of	this	time,	and	that	we
old

improve	its	performance	by	a	factor	of	
k.
	That	is,	the	component	originally
required	time	α
T
,	and	it	now	requires	time	(α
T
)/
k
.	The	overall
execution	time	would	thus	be
From	this,	we	can	compute	the	speedup	
S
	=	
T
/
T
	as
As	an	example,	consider	the	case	where	a	part	of	the	system	that	initially
consumed	60%	of	the	time	(α	=	0.6)	is	sped	up	by	a	factor	of	3	(
k
	=	3).
Then	we	get	a	speedup	of	1/[0.4	+	0.6/3]	=	1.67×.	Even	though	we	made
a	substantial	improvement	to	a	major	part	of	the	system,	our	net	speedup
was	significantly	less	than	the	speedup	for	the	one	part.	This	is	the	major
insight	of	Amdahl's	law—to	significantly	speed	up	the	entire	system,	we
must	improve	the	speed	of	a	very	large	fraction	of	the	overall	system.
Practice	Problem	
1.1	
(solution	page	
28
)
Suppose	you	work	as	a	truck	driver,	and	you	have	been	hired	to
carry	a	load	of	potatoes	from	Boise,	Idaho,	to	Minneapolis,
Minnesota,	a	total	distance	of	2,500	kilometers.	You	estimate	you
can	average	100	km/hr	driving	within	the	speed	limits,	requiring	a
total	of	25	hours	for	the	trip.
Aside	
Expressing	relative	performance
old
old
T
new
=
(
1
−
α
)
T
old
+
(
α
T
old
)
/
k
=
T
o
l
d
[
(
1
−
α
)
+
α
/
k
]
old
new
S
=
1
(
1
−
α
)
+
α
/
k
(1.1)

The	best	way	to	express	a	performance	improvement	is	as	a	ratio
of	the	form	
T
/
T
,	where	
T
	is	the	time	required	for	the	original
version	and	
T
	is	the	time	required	by	the	modified	version.	This
will	be	a	number	greater	than	1.0	if	any	real	improvement
occurred.	We	use	the	suffix	`×'	to	indicate	such	a	ratio,	where	the
factor	"2.2×"	is	expressed	verbally	as	"2.2	times."
The	more	traditional	way	of	expressing	relative	change	as	a
percentage	works	well	when	the	change	is	small,	but	its	definition
is	ambiguous.	Should	it	be	100	·	(
T
	−	
T
)/
T
,	or	possibly	100	·
(
T
	−	
T
)/
T
,	or	something	else?	In	addition,	it	is	less	instructive
for	large	changes.	Saying	that	"performance	improved	by	120%"
is	more	difficult	to	comprehend	than	simply	saying	that	the
performance	improved	by	2.2×.
A
.	
You	hear	on	the	news	that	Montana	has	just	abolished	its	speed
limit,	which	constitutes	1,500	km	of	the	trip.	Your	truck	can	travel
at	150	km/hr.	What	will	be	your	speedup	for	the	trip?
B
.	
You	can	buy	a	new	turbocharger	for	your	truck	at
www.fasttrucks.com
.	They	stock	a	variety	of	models,	but	the
faster	you	want	to	go,	the	more	it	will	cost.	How	fast	must	you
travel	through	Montana	to	get	an	overall	speedup	for	your	trip	of
1.67×?
Practice	Problem	
1.2	
(solution	page	
28
)
The	marketing	department	at	your	company	has	promised	your
customers	that	the	next	software	release	will	show	a	2×
performance	improvement.	You	have	been	assigned	the	task	of
delivering	on	that	promise.	You	have	determined	that	only	80%	of
old
new
old
new
old
new
new
old
new
old

the	system	can	be	improved.	How	much	(i.e.,	what	value	of	
k
)
would	you	need	to	improve	this	part	to	meet	the	overall
performance	target?
One	interesting	special	case	of	Amdahl's	law	is	to	consider	the	effect	of
setting	
k
	to	∞.	That	is,	we	are	able	to	take	some	part	of	the	system	and
speed	it	up	to	the	point	at	which	it	takes	a	negligible	amount	of	time.	We
then	get
So,	for	example,	if	we	can	speed	up	60%	of	the	system	to	the	point
where	it	requires	close	to	no	time,	our	net	speedup	will	still	only	be	1/0.4
=	2.5×.
Amdahl's	law	describes	a	general	principle	for	improving	any	process.	In
addition	to	its	application	to	speeding	up	computer	systems,	it	can	guide
a	company	trying	to	reduce	the	cost	of	manufacturing	razor	blades,	or	a
student	trying	to	improve	his	or	her	grade	point	average.	Perhaps	it	is
most	meaningful	in	the	world	
of	computers,	where	we	routinely	improve
performance	by	factors	of	2	or	more.	Such	high	factors	can	only	be
achieved	by	optimizing	large	parts	of	a	system.
1.9.2	
Concurrency	and	Parallelism
Throughout	the	history	of	digital	computers,	two	demands	have	been
constant	forces	in	driving	improvements:	we	want	them	to	do	more,	and
we	want	them	to	run	faster.	Both	of	these	factors	improve	when	the
S
∞
=
1
(
1
-
α
)
(1.2)

processor	does	more	things	at	once.	We	use	the	term	
concurrency
	to
refer	to	the	general	concept	of	a	system	with	multiple,	simultaneous
activities,	and	the	term	
parallelism
	to	refer	to	the	use	of	concurrency	to
make	a	system	run	faster.	Parallelism	can	be	exploited	at	multiple	levels
of	abstraction	in	a	computer	system.	We	highlight	three	levels	here,
working	from	the	highest	to	the	lowest	level	in	the	system	hierarchy.
Thread-Level	Concurrency
Building	on	the	process	abstraction,	we	are	able	to	devise	systems	where
multiple	programs	execute	at	the	same	time,	leading	to	
concurrency
.
With	threads,	we	can	even	have	multiple	control	flows	executing	within	a
single	process.	Support	for	concurrent	execution	has	been	found	in
computer	systems	since	the	advent	of	time-sharing	in	the	early	1960s.
Traditionally,	this	concurrent	execution	was	only	
simulated
,	by	having	a
single	computer	rapidly	switch	among	its	executing	processes,	much	as	a
juggler	keeps	multiple	balls	flying	through	the	air.	This	form	of
concurrency	allows	multiple	users	to	interact	with	a	system	at	the	same
time,	such	as	when	many	people	want	to	get	pages	from	a	single	Web
server.	It	also	allows	a	single	user	to	engage	in	multiple	tasks
concurrently,	such	as	having	a	Web	browser	in	one	window,	a	word
processor	in	another,	and	streaming	music	playing	at	the	same	time.	Until
recently,	most	actual	computing	was	done	by	a	single	processor,	even	if
that	processor	had	to	switch	among	multiple	tasks.	This	configuration	is
known	as	a	
uniprocessor	system.
When	we	construct	a	system	consisting	of	multiple	processors	all	under
the	control	of	a	single	operating	system	kernel,	we	have	a	
multiprocessor
system
.	Such	systems	have	been	available	for	large-scale	computing

since	the	1980s,	but	they	have	more	recently	become	commonplace	with
the	advent	of	
multi-core
	processors	and	
hyperthreading
.	
Figure	
1.16
shows	a	taxonomy	of	these	different	processor	types.
Multi-core	processors	have	several	CPUs	(referred	to	as	"cores")
integrated	onto	a	single	integrated-circuit	chip.	
Figure	
1.17
	illustrates
the	organization	of	a
Figure	
1.16	
Categorizing	different	processor	configurations.
Multiprocessors	are	becoming	prevalent	with	the	advent	of	multi-core
processors	and	hyperthreading.
Figure	
1.17	
Multi-core	processor	organization.

Four	processor	cores	are	integrated	onto	a	single	chip.
typical	multi-core	processor,	where	the	chip	has	four	CPU	cores,	each
with	its	own	L1	and	L2	caches,	and	with	each	L1	cache	split	into	two
parts—one	to	hold	recently	fetched	instructions	and	one	to	hold	data.	The
cores	share	higher	levels	of	cache	as	well	as	the	interface	to	main
memory.	Industry	experts	predict	that	they	will	be	able	to	have	dozens,
and	ultimately	hundreds,	of	cores	on	a	single	chip.
Hyperthreading,	sometimes	called	
simultaneous	multi-threading
,	is	a
technique	that	allows	a	single	CPU	to	execute	multiple	flows	of	control.	It
involves	having	multiple	copies	of	some	of	the	CPU	hardware,	such	as
program	counters	and	register	files,	while	having	only	single	copies	of
other	parts	of	the	hardware,	such	as	the	units	that	perform	floating-point
arithmetic.	Whereas	a	conventional	processor	requires	around	20,000
clock	cycles	to	shift	between	different	threads,	a	hyper	threaded
processor	decides	which	of	its	threads	to	execute	on	a	cycle-by-cycle
basis.	It	enables	the	CPU	to	take	better	advantage	of	its	processing
resources.	For	example,	if	one	thread	must	wait	for	some	data	to	be
loaded	into	a	cache,	the	CPU	can	proceed	with	the	execution	of	a
different	thread.	As	an	example,	the	Intel	Core	i7	processor	can	have
each	core	executing	two	threads,	and	so	a	four-core	system	can	actually
execute	eight	threads	in	parallel.
The	use	of	multiprocessing	can	improve	system	performance	in	two
ways.	First,	it	reduces	the	need	to	simulate	concurrency	when	performing
multiple	tasks.	As	mentioned,	even	a	personal	computer	being	used	by	a
single	person	is	expected	to	perform	many	activities	concurrently.
Second,	it	can	run	a	single	application	program	faster,	but	only	if	that
program	is	expressed	in	terms	of	multiple	threads	that	can	effectively

execute	in	parallel.	Thus,	although	the	principles	of	concurrency	have
been	formulated	and	studied	for	over	50	years,	the	advent	of	multi-core
and	hyperthreaded	systems	has	greatly	increased	the	desire	to	find	ways
to	write	application	programs	that	can	exploit	the	thread-level	parallelism
available	with	
the	hardware.	
Chapter	
12
	will	look	much	more	deeply
into	concurrency	and	its	use	to	provide	a	sharing	of	processing	resources
and	to	enable	more	parallelism	in	program	execution.
Instruction-Level	Parallelism
At	a	much	lower	level	of	abstraction,	modern	processors	can	execute
multiple	instructions	at	one	time,	a	property	known	as	
instruction-level
parallelism
.	For	example,	early	microprocessors,	such	as	the	1978-
vintage	Intel	8086,	required	multiple	(typically	3-10)	clock	cycles	to
execute	a	single	instruction.	More	recent	processors	can	sustain
execution	rates	of	2-4	instructions	per	clock	cycle.	Any	given	instruction
requires	much	longer	from	start	to	finish,	perhaps	20	cycles	or	more,	but
the	processor	uses	a	number	of	clever	tricks	to	process	as	many	as	100
instructions	at	a	time.	In	
Chapter	
4
,	we	will	explore	the	use	of
pipelining
,	where	the	actions	required	to	execute	an	instruction	are
partitioned	into	different	steps	and	the	processor	hardware	is	organized
as	a	series	of	stages,	each	performing	one	of	these	steps.	The	stages
can	operate	in	parallel,	working	on	different	parts	of	different	instructions.
We	will	see	that	a	fairly	simple	hardware	design	can	sustain	an	execution
rate	close	to	1	instruction	per	clock	cycle.
Processors	that	can	sustain	execution	rates	faster	than	1	instruction	per
cycle	are	known	as	
superscalar
	processors.	Most	modern	processors
support	superscalar	operation.	In	
Chapter	
5
,	we	will	describe	a	high-

level	model	of	such	processors.	We	will	see	that	application
programmers	can	use	this	model	to	understand	the	performance	of	their
programs.	They	can	then	write	programs	such	that	the	generated	code
achieves	higher	degrees	of	instruction-level	parallelism	and	therefore
runs	faster.
Single-Instruction,	Multiple-Data	(SIMD)
Parallelism
At	the	lowest	level,	many	modern	processors	have	special	hardware	that
allows	a	single	instruction	to	cause	multiple	operations	to	be	performed	in
parallel,	a	mode	known	as	
single-instruction,	multiple-data
(SIMD)
parallelism.	For	example,	recent	generations	of	Intel	and	AMD
processors	have	instructions	that	can	add	8	pairs	of	single-precision
floating-point	numbers	(C	data	type	
)	in	parallel.
These	SIMD	instructions	are	provided	mostly	to	speed	up	applications
that	process	image,	sound,	and	video	data.	Although	some	compilers
attempt	to	automatically	extract	SIMD	parallelism	from	C	programs,	a
more	reliable	method	is	to	write	programs	using	special	
vector
	data	types
supported	in	compilers	such	as	
GCC
.	We	describe	this	style	of
programming	in	Web	Aside	
OPT
:
SIMD
,	as	a	supplement	to	the	more
general	presentation	on	program	optimization	found	in	
Chapter	
5
.
1.9.3	
The	Importance	of
Abstractions	in	Computer	Systems

The	use	of	
abstractions
	is	one	of	the	most	important	concepts	in
computer	science.	For	example,	one	aspect	of	good	programming
practice	is	to	formulate	a	simple	application	program	interface	(API)	for	a
set	of	functions	that	allow	programmers	to	use	the	code	without	having	to
delve	into	its	inner	workings.	Different	programming
Figure	
1.18	
Some	abstractions	provided	by	a	computer	system.
A	major	theme	in	computer	systems	is	to	provide	abstract
representations	at	different	levels	to	hide	the	complexity	of	the	actual
implementations.
languages	provide	different	forms	and	levels	of	support	for	abstraction,
such	as	Java	class	declarations	and	C	function	prototypes.
We	have	already	been	introduced	to	several	of	the	abstractions	seen	in
computer	systems,	as	indicated	in	
Figure	
1.18
.	On	the	processor	side,
the	
instruction	set	architecture
	provides	an	abstraction	of	the	actual
processor	hardware.	With	this	abstraction,	a	machine-code	program
behaves	as	if	it	were	executed	on	a	processor	that	performs	just	one
instruction	at	a	time.	The	underlying	hardware	is	far	more	elaborate,
executing	multiple	instructions	in	parallel,	but	always	in	a	way	that	is
consistent	with	the	simple,	sequential	model.	By	keeping	the	same
execution	model,	different	processor	implementations	can	execute	the
same	machine	code	while	offering	a	range	of	cost	and	performance.

On	the	operating	system	side,	we	have	introduced	three	abstractions:
files
	as	an	abstraction	of	I/O	devices,	
virtual	memory
	as	an	abstraction	of
program	memory,	and	
processes
	as	an	abstraction	of	a	running	program.
To	these	abstractions	we	add	a	new	one:	the	
virtual	machine
,	providing
an	abstraction	of	the	entire	computer,	including	the	operating	system,	the
processor,	and	the	programs.	The	idea	of	a	virtual	machine	was
introduced	by	IBM	in	the	1960s,	but	it	has	become	more	prominent
recently	as	a	way	to	manage	computers	that	must	be	able	to	run
programs	designed	for	multiple	operating	systems	(such	as	Microsoft
Windows,	Mac	OS	X,	and	Linux)	or	different	versions	of	the	same
operating	system.
We	will	return	to	these	abstractions	in	subsequent	sections	of	the	book.

1.10	
Summary
A	computer	system	consists	of	hardware	and	systems	software	that
cooperate	to	run	application	programs.	Information	inside	the	computer	is
represented	as	groups	of	bits	that	are	interpreted	in	different	ways,
depending	on	the	context.	Programs	are	translated	by	other	programs
into	different	forms,	beginning	as	ASCII	text	and	then	translated	by
compilers	and	linkers	into	binary	executable	files.
Processors	read	and	interpret	binary	instructions	that	are	stored	in	main
memory.	Since	computers	spend	most	of	their	time	copying	data	between
memory,	I/O	devices,	and	the	CPU	registers,	the	storage	devices	in	a
system	are	arranged	in	a	hierarchy,	with	the	CPU	registers	at	the	top,
followed	by	multiple	levels	of	hardware	cache	memories,	DRAM	main
memory,	and	disk	storage.	Storage	devices	that	are	higher	in	the
hierarchy	are	faster	and	more	costly	per	bit	than	those	lower	in	the
hierarchy.	Storage	devices	that	are	higher	in	the	hierarchy	serve	as
caches	for	devices	that	are	lower	in	the	hierarchy.	Programmers	can
optimize	the	performance	of	their	C	programs	by	understanding	and
exploiting	the	memory	hierarchy.
The	operating	system	kernel	serves	as	an	intermediary	between	the
application	and	the	hardware.	It	provides	three	fundamental	abstractions:
(1)	Files	are	abstractions	for	I/O	devices.	(2)	Virtual	memory	is	an
abstraction	for	both	main	memory	and	disks.	(3)	Processes	are
abstractions	for	the	processor,	main	memory,	and	I/O	devices.

Finally,	networks	provide	ways	for	computer	systems	to	communicate
with	one	another.	From	the	viewpoint	of	a	particular	system,	the	network
is	just	another	I/O	device.

Bibliographic	Notes
Ritchie	has	written	interesting	firsthand	accounts	of	the	early	days	of	C
and	
	[
91
,	
92
].	Ritchie	and	Thompson	presented	the	first	published
account	of	
	[
93
].	Silberschatz,	Galvin,	and	Gagne	[
102
]	provide	a
comprehensive	history	of	the	different	flavors	of	Unix.	The	GNU
(
www.gnu.org
)	and	Linux	(
www.linux.org
)	Web	pages	have	loads	of
current	and	historical	information.	The	Posix	standards	are	available
online	at	(
www.unix.org
).

Solutions	to	Practice	Problems
Solution	to	Problem	
1.1	
(page
22
)
This	problem	illustrates	that	Amdahl's	law	applies	to	more	than	just
computer	systems.
A
.	
In	terms	of	
Equation
	
1.1
,	we	have	α	=	0.6	and	
k
	=	1.5.	More
directly,	traveling	the	1,500	kilometers	through	Montana	will
require	10	hours,	and	the	rest	of	the	trip	also	requires	10	hours.
This	will	give	a	speedup	of	25/(10	+	10)	=	1.25×.
B
.	
In	terms	of	
Equation
	
1.1
,	we	have	α	=	0.6,	and	we	require	
S
	=
1.67,	from	which	we	can	solve	for	
k
.	More	directly,	to	speed	up	the
trip	by	1.67×,	we	must	decrease	the	overall	time	to	15	hours.	The
parts	outside	of	Montana	will	still	require	10	hours,	so	we	must
drive	through	Montana	in	5	hours.	This	requires	traveling	at	300
km/hr,	which	is	pretty	fast	for	a	truck!
Solution	to	Problem	
1.2	
(page
23
)

Amdahl's	law	is	best	understood	by	working	through	some	examples.
This	one	requires	you	to	look	at	
Equation
	
1.1
	from	an	unusual
perspective.
This	problem	is	a	simple	application	of	the	equation.	You	are	given	
S
	=	2
and	α	=	0.8,	and	you	must	then	solve	for	
k
:
2
=
1
(
1
-
0.8
)
+
0.8
/
k
0.4
+
1.6
/
k
=
1.0
k
=
2.67

Part	
I	
Program	Structure	and
Execution
Our	exploration	of	computer	systems	starts	by	studying	the	computer
itself,	comprising	a	processor	and	a	memory	subsystem.	At	the	core,	we
require	ways	to	represent	basic	data	types,	such	as	approximations	to
integer	and	real	arithmetic.	From	there,	we	can	consider	how	machine-
level	instructions	manipulate	data	and	how	a	compiler	translates	C
programs	into	these	instructions.	Next,	we	study	several	methods	of
implementing	a	processor	to	gain	a	better	understanding	of	how
hardware	resources	are	used	to	execute	instructions.	Once	we
understand	compilers	and	machine-level	code,	we	can	examine	how	to
maximize	program	performance	by	writing	C	programs	that,	when
compiled,	achieve	the	maximum	possible	performance.	We	conclude	with
the	design	of	the	memory	subsystem,	one	of	the	most	complex
components	of	a	modern	computer	system.
This	part	of	the	book	will	give	you	a	deep	understanding	of	how
application	programs	are	represented	and	executed.	You	will	gain	skills
that	help	you	write	programs	that	are	secure,	reliable,	and	make	the	best
use	of	the	computing	resources.

Chapter	
2	
Representing	and
Manipulating	Information
2.1	
Information	Storage	
34
2.2	
Integer	Representations	
59
2.3	
Integer	Arithmetic	
84
2.4	
Floating	Point	
108
2.5	
Summary
	
126
Bibliographic	Notes	
127
Homework	Problems	
128
Solutions	to	Practice	Problems	
143
Modern	computers	store	and	process	information
represented	as	two-valued	signals.	These	lowly
binary	digits,	or	
bits
,	form	the	basis	of	the	digital
revolution.	The	familiar	decimal,	or	base-10,
representation	has	been	in	use	for	over	1,000	years,
having	been	developed	in	India,	improved	by	Arab
mathematicians	in	the	12th	century,	and	brought	to

the	West	in	the	13th	century	by	the	Italian
mathematician	Leonardo	Pisano	(ca.	1170	to	ca.
1250),	better	known	as	Fibonacci.	Using	decimal
notation	is	natural	for	10-fingered	humans,	but
binary	values	work	better	when	building	machines
that	store	and	process	information.	Two-valued
signals	can	readily	be	represented,	stored,	and
transmitted—for	example,	as	the	presence	or
absence	of	a	hole	in	a	punched	card,	as	a	high	or
low	voltage	on	a	wire,	or	as	a	magnetic	domain
oriented	clockwise	or	counterclockwise.	The
electronic	circuitry	for	storing	and	performing
computations	on	two-valued	signals	is	very	simple
and	reliable,	enabling	manufacturers	to	integrate
millions,	or	even	billions,	of	such	circuits	on	a	single
silicon	chip.
In	isolation,	a	single	bit	is	not	very	useful.	When	we
group	bits	together	and	apply	some	
interpretation
that	gives	meaning	to	the	different	possible	bit
patterns,	however,	we	can	represent	the	elements
of	any	finite	set.	For	example,	using	a	binary
number	system,	we	can	use	groups	of	bits	to
encode	nonnegative	numbers.	By	using	a	standard
character	code,	we	can	encode	the	letters	and
symbols	in	a	document.	We	cover	both	of	these
encodings	in	this	chapter,	as	well	as	encodings	to
represent	negative	numbers	and	to	approximate
real	numbers.

We	consider	the	three	most	important
representations	of	numbers.	
Unsigned
	encodings
are	based	on	traditional	binary	notation,
representing	numbers	greater	than	or	equal	to	0.
Two's-complement
	encodings	are	the	most	common
way	to	represent	
signed
	integers,	that	is,	numbers
that	may	be	either	positive	or	negative.	
Floating-
point
	encodings	are	a	base-2	version	of	scientific
notation	for	representing	real	numbers.	Computers
implement	arithmetic	operations,	such	as	addition
and	multiplication,	with	these	different
representations,	similar	to	the	corresponding
operations	on	integers	and	real	numbers.
Computer	representations	use	a	limited	number	of
bits	to	encode	a	number,	and	hence	some
operations	can	
overflow
	when	the	results	are	too
large	to	be	represented.	This	can	lead	to	some
surprising	results.	For	example,	on	most	of	today's
computers	(those	using	a	32-bit	representation	for
data	type	int),	computing	the	expression
yields	–884,901,888.	This	runs	counter	to	the
properties	of	integer	arithmetic—computing	the

product	of	a	set	of	positive	numbers	has	yielded	a
negative	result.
On	the	other	hand,	integer	computer	arithmetic
satisfies	many	of	the	familiar	properties	of	true
integer	arithmetic.	For	example,	multiplication	is
associative	and	commutative,	so	that	computing	any
of	the	following	C	expressions	yields	–884,901,888:
The	computer	might	not	generate	the	expected
result,	but	at	least	it	is	consistent!
Floating-point	arithmetic	has	altogether	different
mathematical	properties.	The	product	of	a	set	of
positive	numbers	will	always	be	positive,	although
overflow	will	yield	the	special	value	+∞.	Floating-
point	arithmetic	is	not	associative	due	to	the	finite
precision	of	the	representation.	For	example,	the	C
expression	
	will	evaluate	to	0.0	on
most	machines,	while	
	will	evaluate

to	3.14.	The	different	mathematical	properties	of
integer	versus.	floating-point	arithmetic	stem	from
the	difference	in	how	they	handle	the	finiteness	of
their	representations—integer	representations	can
encode	a	comparatively	small	range	of	values,	but
do	so	precisely,	while	floating-point	representations
can	encode	a	wide	range	of	values,	but	only
approximately.
By	studying	the	actual	number	representations,	we
can	understand	the	ranges	of	values	that	can	be
represented	and	the	properties	of	the	different
arithmetic	operations.	This	understanding	is	critical
to	writing	programs	that	work	correctly	over	the	full
range	of	numeric	values	and	that	are	portable
across	different	combinations	of	machine,	operating
system,	and	compiler.	As	we	will	describe,	a	number
of	computer	security	vulnerabilities	have	arisen	due
to	some	of	the	subtleties	of	computer	arithmetic.
Whereas	in	an	earlier	era	program	bugs	would	only
inconvenience	people	when	they	happened	to	be
triggered,	there	are	now	legions	of	hackers	who	try
to	exploit	any	bug	they	can	find	to	obtain
unauthorized	access	to	other	people's	systems.	This
puts	a	higher	level	of	obligation	on	programmers	to
understand	how	their	programs	work	and	how	they
can	be	made	to	behave	in	undesirable	ways.
Computers	use	several	different	binary

Computers	use	several	different	binary
representations	to	encode	numeric	values.	You	will
need	to	be	familiar	with	these	representations	as
you	progress	into	machine-level	programming	in
Chapter	
3
.	We	describe	these	encodings	in	this
chapter	and	show	you	how	to	reason	about	number
representations.
We	derive	several	ways	to	perform	arithmetic
operations	by	directly	manipulating	the	bit-level
representations	of	numbers.	Understanding	these
techniques	will	be	important	for	understanding	the
machine-level	code	generated	by	compilers	in	their
attempt	to	optimize	the	performance	of	arithmetic
expression	evaluation.
Our	treatment	of	this	material	is	based	on	a	core	set
of	mathematical	principles.	We	start	with	the	basic
definitions	of	the	encodings	and	then	derive	such
properties	as	the	range	of	representable	numbers,
their	bit-level	representations,	and	the	properties	of
the	arithmetic	operations.	We	believe	it	is	important
for	you	to	examine	the	material	from	this	abstract
viewpoint,	because	programmers	need	to	have	a
clear	understanding	of	how	computer	arithmetic
relates	to	the	more	familiar	integer	and	real
arithmetic.

The	C++	programming	language	is	built	upon	C,
using	the	exact	same	numeric	representations	and
operations.	Everything	said	in	this	chapter	about	C
also	holds	for	C++.	The	Java	language	definition,	on
the	other	hand,	created	a	new	set	of	standards	for
numeric	representations	and	operations.	Whereas
the	C	standards	are	designed	to	allow	a	wide	range
of	implementations,	the	Java	standard	is	quite
specific	on	the	formats	and	encodings	of	data.	We
highlight	the	representations	and	operations
supported	by	Java	at	several	places	in	the	chapter.
Aside	
How	to	read	this	chapter
In	this	chapter,	we	examine	the	fundamental
properties	of	how	numbers	and	other	forms
of	data	are	represented	on	a	computer	and
the	properties	of	the	operations	that
computers	perform	on	these	data.	This
requires	us	to	delve	into	the	language	of
mathematics,	writing	formulas	and	equations
and	showing	derivations	of	important
properties.
To	help	you	navigate	this	exposition,	we	have
structured	the	presentation	to	first	state	a
property	as	a	
principle
	in	mathematical
notation.	We	then	illustrate	this	principle	with
examples	and	an	informal	discussion.	We

recommend	that	you	go	back	and	forth
between	the	statement	of	the	principle	and
the	examples	and	discussion	until	you	have
a	solid	intuition	for	what	is	being	said	and
what	is	important	about	the	property.	For
more	complex	properties,	we	also	provide	a
derivation
,	structured	much	like	a
mathematical	proof.	You	should	try	to
understand	these	derivations	eventually,	but
you	could	skip	over	them	on	first	reading.
We	also	encourage	you	to	work	on	the
practice	problems	as	you	proceed	through
the	presentation.	The	practice	problems
engage	you	in	
active	learning
,	helping	you
put	thoughts	into	action.	With	these	as
background,	you	will	find	it	much	easier	to	go
back	and	follow	the	derivations.	Be	assured,
as	well,	that	the	mathematical	skills	required
to	understand	this	material	are	within	reach
of	someone	with	a	good	grasp	of	high	school
algebra.

2.1	
Information	Storage
Rather	than	accessing	individual	bits	in	memory,	most	computers	use
blocks	of	8	bits,	or	
bytes
,	as	the	smallest	addressable	unit	of	memory.	A
machine-level	program	views	memory	as	a	very	large	array	of	bytes,
referred	to	as	
virtual	memory
.	Every	byte	of	memory	is	identified	by	a
unique	number,	known	as	its	
address
,	and	the	set	of	all	possible
addresses	is	known	as	the	
virtual	address	space
.	As	indicated	by	its
name,	this	virtual	address	space	is	just	a	conceptual	image	presented	to
the	machine-level	program.	The	actual	implementation	(presented	in
Chapter	
9
)	uses	a	combination	of	dynamic	random	access	memory
(DRAM),	flash	memory,	disk	storage,	special	hardware,	and	operating
system	software	to	provide	the	program	with	what	appears	to	be	a
monolithic	byte	array.
In	subsequent	chapters,	we	will	cover	how	the	compiler	and	run-time
system	partitions	this	memory	space	into	more	manageable	units	to	store
the	different	
program	objects
,	that	is,	program	data,	instructions,	and
control	information.	Various	mechanisms	are	used	to	allocate	and
manage	the	storage	for	different	parts	of	the	program.	This	management
is	all	performed	within	the	virtual	address	space.	For	example,	the	value
of	a	pointer	in	C—whether	it	points	to	an	integer,	a	structure,	or	some
other	program	object—is	the	virtual	address	of	the	first	byte	of	some
block	of	storage.	The	C	compiler	also	associates	
type
	information	with
each	pointer,	so	that	it	can	generate	different	machine-level	code	to
access	the	value	stored	at	the	location	designated	by	the	pointer
depending	on	the	type	of	that	value.	Although	the	C	compiler	maintains

this	type	information,	the	actual	machine-level	program	it	generates	has
no	information	about	data	types.	It	simply	treats	each	program	object	as
a	block	of	bytes	and	the	program	itself	as	a	sequence	of	bytes.
Aside	
The	evolution	of	the	C
programming	language
As	was	described	in	an	aside	on	page	4,	the	C	programming
language	was	first	developed	by	Dennis	Ritchie	of	Bell
Laboratories	for	use	with	the	Unix	operating	system	(also
developed	at	Bell	Labs).	At	the	time,	most	system	programs,	such
as	operating	systems,	had	to	be	written	largely	in	assembly	code
in	order	to	have	access	to	the	low-level	representations	of
different	data	types.	For	example,	it	was	not	feasible	to	write	a
memory	allocator,	such	as	is	provided	by	the	
	library
function,	in	other	high-level	languages	of	that	era.
The	original	Bell	Labs	version	of	C	was	documented	in	the	first
edition	of	the	book	by	Brian	Kernighan	and	Dennis	Ritchie	[
60
].
Over	time,	C	has	evolved	through	the	efforts	of	several
standardization	groups.	The	first	major	revision	of	the	original	Bell
Labs	C	led	to	the	ANSI	C	standard	in	1989,	by	a	group	working
under	the	auspices	of	the	American	National	Standards	Institute.
ANSI	C	was	a	major	departure	from	Bell	Labs	C,	especially	in	the
way	functions	are	declared.	ANSI	C	is	described	in	the	second
edition	of	Kernighan	and	Ritchie's	book	[
61
],	which	is	still
considered	one	of	the	best	references	on	C.

The	International	Standards	Organization	took	over	responsibility
for	standardizing	the	C	language,	adopting	a	version	that	was
substantially	the	same	as	ANSI	C	in	1990	and	hence	is	referred	to
as	“ISO	C90.”
This	same	organization	sponsored	an	updating	of	the	language	in
1999,	yielding	“ISO	C99.”	Among	other	things,	this	version
introduced	some	new	data	types	and	provided	support	for	text
strings	requiring	characters	not	found	in	the	English	language.	A
more	recent	standard	was	approved	in	2011,	and	hence	is	named
“ISO	C11,”	again	adding	more	data	types	and	features.	Most	of
these	recent	additions	have	been	
backward	compatible
,	meaning
that	programs	written	according	to	the	earlier	standard	(at	least	as
far	back	as	ISO	C90)	will	have	the	same	behavior	when	compiled
according	to	the	newer	standards.
The	GNU	Compiler	Collection	(
)	can	compile	programs
according	to	the	conventions	of	several	different	versions	of	the	C
language,	based	on	different	command-line	options,	as	shown	in
Figure	
2.1
.	For	example,	to	compile	program	
	according
to	ISO	C11,	we	could	give	the	command	line
The	options	
	and	
	have	identical	effect—the	code	is
compiled	according	to	the	ANSI	or	ISO	C90	standard.	(C90	is
sometimes	referred	to	as	“C89,”	since	its	standardization	effort

began	in	1989.)	The	option	
	causes	the	compiler	to	follow
the	ISO	C99	convention.
As	of	the	writing	of	this	book,	when	no	option	is	specified,	the
program	will	be	compiled	according	to	a	version	of	C	based	on
ISO	C90,	but	including	some	features	of	C99,	some	of	C11,	some
of	C++,	and	others	specific	to	
GCC
.	The	GNU	project	is	developing
a	version	that	combines	ISO	C11,	plus	other	features,	that	can	be
specified	with	command-line	option	
.	(Currently,	this
implementation	is	incomplete.)	This	will	become	the	default
version.
C	version
	command-line	option
GNU	89
none
,	
ANSI,	ISO	C90
ISO	C99
ISO	C11
Figure	
2.1	
Specifying	different	versions	of	C	to	
.
New	to	C?	
The	role	of	pointers	in	C
Pointers	are	a	central	feature	of	C.	They	provide	the	mechanism
for	referencing	elements	of	data	structures,	including	arrays.	Just
like	a	variable,	a	pointer	has	two	aspects:	its	
value
	and	its	
type
.
The	value	indicates	the	location	of	some	object,	while	its	type
indicates	what	kind	of	object	(e.g.,	integer	or	floating-point
number)	is	stored	at	that	location.

Truly	understanding	pointers	requires	examining	their
representation	and	implementation	at	the	machine	level.	This	will
be	a	major	focus	in	
Chapter	
3
,	culminating	in	an	in-depth
presentation	in	
Section	
3.10.1
.
2.1.1	
Hexadecimal	Notation
A	single	byte	consists	of	8	bits.	In	binary	notation,	its	value	ranges	from
00000000
	to	11111111
.	When	viewed	as	a	decimal	integer,	its	value
ranges	from	0
	to	255
.	Neither	notation	is	very	convenient	for
describing	bit	patterns.	Binary	notation	is	too	verbose,	while	with	decimal
notation	it	is	tedious	to	convert	to	and	from	bit	patterns.	Instead,	we	write
bit	patterns	as	base-16,	or	
hexadecimal
	numbers.	Hexadecimal	(or
simply	“hex”)	uses	digits	‘0’	through	‘9’	along	with	characters	‘A’	through
‘F’	to	represent	16	possible	values.	
Figure	
2.2
	shows	the	decimal	and
binary	values	associated	with	the	16	hexadecimal	digits.	Written	in
hexadecimal,	the	value	of	a	single	byte	can	range	from	00
	to	FF
.
In	C,	numeric	constants	starting	with	
	or	
	are	interpreted	as	being	in
hexadecimal.	The	characters	‘A’	through	‘F’	may	be	written	in	either
upper-	or	lowercase.	For	example,	we	could	write	the	number	FA1D37B
as	
,	as	
,	or	even	mixing	upper-	and	lower	case	(e.g.,
).	We	will	use	the	C	notation	for	representing	hexadecimal
values	in	this	book.
A	common	task	in	working	with	machine-level	programs	is	to	manually
convert	between	decimal,	binary,	and	hexadecimal	representations	of	bit
2
2
10
10
16
16
16

patterns.	Converting	between	binary	and	hexadecimal	is	straightforward,
since	it	can	be	performed	one	hexadecimal	digit	at	a	time.	Digits	can	be
converted	by	referring	to	a	chart	such	as	that	shown	in	
Figure	
2.2
.
One	simple	trick	for	doing	the	conversion	in	your	head	is	to	memorize	the
decimal	equivalents	of	hex	digits	
,	and	
.
Hex	digit
0
1
2
3
4
5
6
7
Decimal	value
0
1
2
3
4
5
6
7
Binary	value
0000
0001
0010
0011
0100
0101
0110
0111
Hex	digit
8
9
A
B
C
D
E
F
Decimal	value
8
9
10
11
12
13
14
15
Binary	value
1000
1001
1010
1011
1100
1101
1110
1111
Figure	
2.2	
Hexadecimal	notation.
Each	hex	digit	encodes	one	of	16	values.
The	hex	values	
,	and	
	can	be	translated	to	decimal	by	computing
their	values	relative	to	the	first	three.
For	example,	suppose	you	are	given	the	number	
.	You	can
convert	this	to	binary	format	by	expanding	each	hexadecimal	digit,	as
follows:
Hexadecimal
Binary

This	gives	the	binary	representation	000101110011101001001100.
Conversely,	given	a	binary	number	1111001010110110110011,	you
convert	it	to	hexadecimal	by	first	splitting	it	into	groups	of	4	bits	each.
Note,	however,	that	if	the	total	number	of	bits	is	not	a	multiple	of	4,	you
should	make	the	
leftmost
	group	be	the	one	with	fewer	than	4	bits,
effectively	padding	the	number	with	leading	zeros.	Then	you	translate
each	group	of	bits	into	the	corresponding	hexadecimal	digit:
Binary
Hexadecimal
Practice	Problem	
2.1
	(solution	page	
143
)
Perform	the	following	number	conversions:
A
.	
	to	binary
B
.	
binary	1100100101111011	to	hexadecimal
C
.	
	to	binary
D
.	
binary	1001101110011110110101	to	hexadecimal
When	a	value	
x
	is	a	power	of	2,	that	is,	
x
	=	2
	for	some	nonnegative
integer	
n
,	we	can	readily	write	
x
	in	hexadecimal	form	by	remembering
that	the	binary	representation	of	
x
	is	simply	1	followed	by	
n
	zeros.	The
hexadecimal	digit	0	represents	4	binary	zeros.	So,	for	
n
	written	in	the
form	
i
	+	4
j
,	where	0	≤	
i
	≤	3,	we	can	write	
x
	with	a	leading	hex	digit	of	1	(
i
	=
0),	2	(
i
	=	1),	4	(
i
	=	2),	or	8	(
i
	=	3),	followed	by	
j
	hexadecimal	
.	As	an
n

example,	for	
x
	=	2,048	=	211,	we	have	
n
	=	11	=	3	+	4·2,	giving
hexadecimal	representation	
.
Practice	Problem	
2.2
	(solution	page	
143
)
Fill	in	the	blank	entries	in	the	following	table,	giving	the	decimal
and	hexadecimal	representations	of	different	powers	of	2:
n
2
	(decimal)
2
	(hexadecimal)
9
512
19
__________
__________
16,384
__________
__________
17
__________
__________
__________
32
__________
__________
__________
Converting	between	decimal	and	hexadecimal	representations	requires
using	multiplication	or	division	to	handle	the	general	case.	To	convert	a
decimal	number	
x
	to	hexadecimal,	we	can	repeatedly	divide	
x
	by	16,
giving	a	quotient	
q
	and	a	remainder
r
,	such	that	
x
	=	
q
	·	16	+	
r
.We	then	use
the	hexadecimal	digit	representing	
r
	as	the	least	significant	digit	and
generate	the	remaining	digits	by	repeating	the	process	on	
q
.	As	an
example,	consider	the	conversion	of	decimal	314,156:314,156
n
n

From	this	we	can	read	off	the	hexadecimal	representation	as	
.
Conversely,	to	convert	a	hexadecimal	number	to	decimal,	we	can	multiply
each	of	the	hexadecimal	digits	by	the	appropriate	power	of	16.	For
example,	given	the	number	
,	we	compute	its	decimal	equivalent	as
7	·	16
	+	10	·	16	+	15	=	7	·	256	+	10	·	16	+	15	=	1,792	+	160	+	15	=
1,967.
Practice	Problem	
2.3
	(solution	page	
144
)
A	single	byte	can	be	represented	by	2	hexadecimal	digits.	Fill	in
the	missing	entries	in	the	following	table,	giving	the	decimal,
binary,	and	hexadecimal	values	of	different	byte	patterns:
Decimal
Binary
Hexadecimal
0
0000	0000
167
__________
__________
62
__________
__________
188
__________
__________
__________
0011	0111
__________
__________
1000	1000
__________
__________
1111	0011
__________
314,156:314,156
=
19,634
⋅
16
+
12
(
C
)
19,634
=
1,227
⋅
16
+
2
(
2
)
1,227
=
76
⋅
16
+
11
2

Aside	
Converting	between	decimal
and	hexadecimal
For	converting	larger	values	between	decimal	and
hexadecimal,	it	is	best	to	let	a	computer	or	calculator	do	the
work.	There	are	numerous	tools	that	can	do	this.	One
simple	way	is	to	use	any	of	the	standard	search	engines,
with	queries	such	as
Convert	
	to	decimal
or
123	in	hex
Decimal
Binary
Hexadecimal
__________
__________
__________
__________
__________
__________
Practice	Problem	
2.4
	(solution	page	
144
)
Without	converting	the	numbers	to	decimal	or	binary,	try	to	solve
the	following	arithmetic	problems,	giving	the	answers	in
hexadecimal.	
Hint:
	Just	modify	the	methods	you	use	for
performing	decimal	addition	and	subtraction	to	use	base	16.
A
.	
	__________
B
.	
	__________

C
.	
	__________
D
.	
	__________
2.1.2	
Data	Sizes
Every	computer	has	a	
word	size
,	indicating	the	nominal	size	of	pointer
data.	Since	a	virtual	address	is	encoded	by	such	a	word,	the	most
important	system	parameter	determined	by	the	word	size	is	the	maximum
size	of	the	virtual	address	space.	That	is,	for	a	machine	with	a	
w
-bit	word
size,	the	virtual	addresses	can	range	from	0	to	2
	—	1,	giving	the
program	access	to	at	most	2
	bytes.
In	recent	years,	there	has	been	a	widespread	shift	from	machines	with
32-bit	word	sizes	to	those	with	word	sizes	of	64	bits.	This	occurred	first
for	high-end	machines	designed	for	large-scale	scientific	and	database
applications,	followed	by	desktop	and	laptop	machines,	and	most
recently	for	the	processors	found	in	smartphones.	A	32-bit	word	size
limits	the	virtual	address	space	to	4	gigabytes	(written	4	GB),	that	is,	just
over	4	×	10
	bytes.	Scaling	up	to	a	64-bit	word	size	leads	to	a	virtual
address	space	of	16	
exabytes
,	or	around	1.84	×	10
	bytes.
Most	64-bit	machines	can	also	run	programs	compiled	for	use	on	32-bit
machines,	a	form	of	backward	compatibility.	So,	for	example,	when	a
program	
	is	compiled	with	the	directive
w
w
9
19

then	this	program	will	run	correctly	on	either	a	32-bit	or	a	64-bit	machine.
On	the	other	hand,	a	program	compiled	with	the	directive
will	only	run	on	a	64-bit	machine.	We	will	therefore	refer	to	programs	as
being	either	“32-bit	programs”	or	“64-bit	programs,”	since	the	distinction
lies	in	how	a	program	is	compiled,	rather	than	the	type	of	machine	on
which	it	runs.
Computers	and	compilers	support	multiple	data	formats	using	different
ways	to	encode	data,	such	as	integers	and	floating	point,	as	well	as
different	lengths.	For	example,	many	machines	have	instructions	for
manipulating	single	bytes,	as	well	as	integers	represented	as	2-,	4-,	and
8-byte	quantities.	They	also	support	floating-point	numbers	represented
as	4-	and	8-byte	quantities.
The	C	language	supports	multiple	data	formats	for	both	integer	and
floating-point	data.	
Figure	
2.3
	shows	the	number	of	bytes	typically
allocated	for	different	C	data	types.	(We	discuss	the	relation	between
what	is	guaranteed	by	the	C	standard	versus.	what	is	typical	in	
Section
2.2
.)	The	exact	numbers	of	bytes	for	some	data	types	depends	on	how
the	program	is	compiled.	We	show	sizes	for	typical	32-bit	and	64-bit
programs.	Integer	data	can	be	either	
signed
,	able	to	represent	negative,
zero,	and	positive	values,	or	
unsigned
,	only	allowing	nonnegative	values.

Data	type	char	represents	a	single	byte.	Although	the	name	char	derives
from	the	fact	that	it	is	used	to	store	a	single	character	in	a	text	string,	it
can	also	be	used	to	store	integer	values.	Data	types	
,	and
	are	intended	to	provide	a	range	of
C	declaration
Bytes
Signed
Unsigned
32-bit
64-bit
1
1
2
2
4
4
4
8
4
4
8
8
4
8
4
4
8
8
Figure	
2.3	
Typical	sizes	(in	bytes)	of	basic	C	data	types.
The	number	of	bytes	allocated	varies	with	how	the	program	is	compiled.
This	chart	shows	the	values	typical	of	32-bit	and	64-bit	programs.
New	to	C?	
Declaring	pointers

For	any	data	type	
T
,	the	declaration
indicates	that	p	is	a	pointer	variable,	pointing	to	an	object	of	type
T
.	For	example,
is	the	declaration	of	a	pointer	to	an	object	of	type	
.
sizes.	Even	when	compiled	for	64-bit	systems,	data	type	
	is	usually
just	4	bytes.	Data	type	
	commonly	has	4	bytes	in	32-bit	programs
and	8	bytes	in	64-bit	programs.
To	avoid	the	vagaries	of	relying	on	“typical”	sizes	and	different	compiler
settings,	ISO	C99	introduced	a	class	of	data	types	where	the	data	sizes
are	fixed	regardless	of	compiler	and	machine	settings.	Among	these	are
data	types	
	and	
,	having	exactly	4	and	8	bytes,
respectively.	Using	fixed-size	integer	types	is	the	best	way	for
programmers	to	have	close	control	over	data	representations.
Most	of	the	data	types	encode	signed	values,	unless	prefixed	by	the
keyword	unsigned	or	using	the	specific	unsigned	declaration	for	fixed-
size	data	types.	The	exception	to	this	is	data	type	
.	Although	most
compilers	and	machines	treat	these	as	signed	data,	the	C	standard	does
not	guarantee	this.	Instead,	as	indicated	by	the	square	brackets,	the

programmer	should	use	the	declaration	
	to	guarantee	a	1-
byte	signed	value.	In	many	contexts,	however,	the	program's	behavior	is
insensitive	to	whether	data	type	
	is	signed	or	unsigned.
The	C	language	allows	a	variety	of	ways	to	order	the	keywords	and	to
include	or	omit	optional	keywords.	As	examples,	all	of	the	following
declarations	have	identical	meaning:
We	will	consistently	use	the	forms	found	in	
Figure	
2.3
.
Figure	
2.3
	also	shows	that	a	pointer	(e.g.,	a	variable	declared	as
being	of	type	
)	uses	the	full	word	size	of	the	program.	Most
machines	also	support	two	different	floating-point	formats:	single
precision,	declared	in	C	as	
,	and	double	precision,	declared	in	C	as
.	These	formats	use	4	and	8	bytes,	respectively.
Programmers	should	strive	to	make	their	programs	portable	across
different	machines	and	compilers.	One	aspect	of	portability	is	to	make	the
program	insensitive	to	the	exact	sizes	of	the	different	data	types.	The	C
standards	set	lower	bounds	
on	the	numeric	ranges	of	the	different	data
types,	as	will	be	covered	later,	but	there	are	no	upper	bounds	(except
with	the	fixed-size	types).	With	32-bit	machines	and	32-bit	programs

being	the	dominant	combination	from	around	1980	until	around	2010,
many	programs	have	been	written	assuming	the	allocations	listed	for	32-
bit	programs	in	
Figure	
2.3
.	With	the	transition	to	64-bit	machines,
many	hidden	word	size	dependencies	have	arisen	as	bugs	in	migrating
these	programs	to	new	machines.	For	example,	many	programmers
historically	assumed	that	an	object	declared	as	type	
	could	be	used	to
store	a	pointer.	This	works	fine	for	most	32-bit	programs,	but	it	leads	to
problems	for	64-bit	programs.
2.1.3	
Addressing	and	Byte	Ordering
For	program	objects	that	span	multiple	bytes,	we	must	establish	two
conventions:	what	the	address	of	the	object	will	be,	and	how	we	will	order
the	bytes	in	memory.	In	virtually	all	machines,	a	multi-byte	object	is
stored	as	a	contiguous	sequence	of	bytes,	with	the	address	of	the	object
given	by	the	smallest	address	of	the	bytes	used.	For	example,	suppose	a
variable	
	of	type	
	has	address	
;	that	is,	the	value	of	the
address	expression	
	is	
.	Then	(assuming	data	type	
	has	a	32-
bit	representation)	the	4	bytes	of	
	would	be	stored	in	memory	locations
,	and	
For	ordering	the	bytes	representing	an	object,	there	are	two	common
conventions.	Consider	a	
w
-bit	integer	having	a	bit	representation	
,	where	
x
	is	the	most	significant	bit	and	
x
	is	the
least.	Assuming	
w
	is	a	multiple	of	8,	these	bits	can	be	grouped	as	bytes,
with	the	most	significant	byte	having	bits	
,	the	least
significant	byte	having	bits	
,	and	the	other	bytes	having	bits
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
⋯
,
x
1
,
x
0
]
w
–1
0
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
⋯
,
x
w
−
8
]
[
x
7
,
x
6
,
…
,
x
0
]

from	the	middle.	Some	machines	choose	to	store	the	object	in	memory
ordered	from	least	significant	byte	to	most,	while	other	machines	store
them	from	most	to	least.	The	former	convention—where	the	least
significant	byte	comes	first—is	referred	to	as	
little	endian
.	The	latter
convention—where	the	most	significant	byte	comes	first—is	referred	to
as	
big	endian
.
Suppose	the	variable	
	of	type	
	and	at	address	
	has	a
hexadecimal	value	of	
.	The	ordering	of	the	bytes	within	the
address	range	
	through	
	depends	on	the	type	of	machine:
Note	that	in	the	word	
	the	high-order	byte	has	hexadecimal
value	
,	while	the	low-order	byte	has	value	
Most	Intel-compatible	machines	operate	exclusively	in	little-endian	mode.
On	the	other	hand,	most	machines	from	IBM	and	Oracle	(arising	from
their	acquisition
Aside	
Origin	of	“endian”

Here	is	how	Jonathan	Swift,	writing	in	1726,	described	the	history
of	the	controversy	between	big	and	little	endians:
.	.	.	Lilliput	and	Blefuscu	.	.	.	have,	as	I	was	going	to	tell	you,	been	engaged	in	a	most
obstinate	war	for	six-and-thirty	moons	past.	It	began	upon	the	following	occasion.	It	is
allowed	on	all	hands,	that	the	primitive	way	of	breaking	eggs,	before	we	eat	them,
was	upon	the	larger	end;	but	his	present	majesty's	grandfather,	while	he	was	a	boy,
going	to	eat	an	egg,	and	breaking	it	according	to	the	ancient	practice,	happened	to
cut	one	of	his	fingers.	Whereupon	the	emperor	his	father	published	an	edict,
commanding	all	his	subjects,	upon	great	penalties,	to	break	the	smaller	end	of	their
eggs.	The	people	so	highly	resented	this	law,	that	our	histories	tell	us,	there	have
been	six	rebellions	raised	on	that	account;	wherein	one	emperor	lost	his	life,	and
another	his	crown.	These	civil	commotions	were	constantly	fomented	by	the
monarchs	of	Blefuscu;	and	when	they	were	quelled,	the	exiles	always	fled	for	refuge
to	that	empire.	It	is	computed	that	eleven	thousand	persons	have	at	several	times
suffered	death,	rather	than	submit	to	break	their	eggs	at	the	smaller	end.	Many
hundred	large	volumes	have	been	published	upon	this	controversy:	but	the	books	of
the	Big-endians	have	been	long	forbidden,	and	the	whole	party	rendered	incapable
by	law	of	holding	employments.
(Jonathan	Swift.	Gulliver's	Travels,	Benjamin	Motte,	1726.)
In	his	day,	Swift	was	satirizing	the	continued	conflicts	between
England	(Lilliput)	and	France	(Blefuscu).	Danny	Cohen,	an	early
pioneer	in	networking	protocols,	first	applied	these	terms	to	refer
to	byte	ordering	[
24
],	and	the	terminology	has	been	widely
adopted.
of	Sun	Microsystems	in	2010)	operate	in	big-endian	mode.	Note	that	we
said	“most.”	The	conventions	do	not	split	precisely	along	corporate
boundaries.	For	example,	both	IBM	and	Oracle	manufacture	machines
that	use	Intel-compatible	processors	and	hence	are	little	endian.	Many
recent	microprocessor	chips	are	
bi-endian
,	meaning	that	they	can	be

configured	to	operate	as	either	little-	or	big-endian	machines.	In	practice,
however,	byte	ordering	becomes	fixed	once	a	particular	operating	system
is	chosen.	For	example,	ARM	microprocessors,	used	in	many	cell
phones,	have	hardware	that	can	operate	in	either	little-	or	big-endian
mode,	but	the	two	most	common	operating	systems	for	these	chips—
Android	(from	Google)	and	IOS	(from	Apple)	—operate	only	in	little-
endian	mode.
People	get	surprisingly	emotional	about	which	byte	ordering	is	the	proper
one.	In	fact,	the	terms	“little	endian”	and	“big	endian”	come	from	the	book
Gulliver's	Travels
	by	Jonathan	Swift,	where	two	warring	factions	could	not
agree	as	to	how	a	soft-boiled	egg	should	be	opened—by	the	little	end	or
by	the	big.	Just	like	the	egg	issue,	there	is	no	technological	reason	to
choose	one	byte	ordering	convention	over	the	other,	and	hence	the
arguments	degenerate	into	bickering	about	sociopolitical	issues.	As	long
as	one	of	the	conventions	is	selected	and	adhered	to	consistently,	the
choice	is	arbitrary.
For	most	application	programmers,	the	byte	orderings	used	by	their
machines	are	totally	invisible;	programs	compiled	for	either	class	of
machine	give	identical	results.	At	times,	however,	byte	ordering	becomes
an	issue.	The	first	is	when	
binary	data	are	communicated	over	a	network
between	different	machines.	A	common	problem	is	for	data	produced	by
a	little-endian	machine	to	be	sent	to	a	big-endian	machine,	or	vice	versa,
leading	to	the	bytes	within	the	words	being	in	reverse	order	for	the
receiving	program.	To	avoid	such	problems,	code	written	for	networking
applications	must	follow	established	conventions	for	byte	ordering	to
make	sure	the	sending	machine	converts	its	internal	representation	to	the
network	standard,	while	the	receiving	machine	converts	the	network

standard	to	its	internal	representation.	We	will	see	examples	of	these
conversions	in	
Chapter	
11
.
A	second	case	where	byte	ordering	becomes	important	is	when	looking
at	the	byte	sequences	representing	integer	data.	This	occurs	often	when
inspecting	machine-level	programs.	As	an	example,	the	following	line
occurs	in	a	file	that	gives	a	text	representation	of	the	machine-level	code
for	an	Intel	x86–64	processor:
This	line	was	generated	by	a	
disassembler
,	a	tool	that	determines	the
instruction	sequence	represented	by	an	executable	program	file.	We	will
learn	more	about	disassemblers	and	how	to	interpret	lines	such	as	this	in
Chapter	
3
.	For	now,	we	simply	note	that	this	line	states	that	the
hexadecimal	byte	sequence	
	is	the	byte-level
representation	of	an	instruction	that	adds	a	word	of	data	to	the	value
stored	at	an	address	computed	by	adding	
	to	the	current	value
of	the	
program	counter
,	the	address	of	the	next	instruction	to	be
executed.	If	we	take	the	final	4	bytes	of	the	sequence	
	and
write	them	in	reverse	order,	we	have	
.	Dropping	the	leading
0,	we	have	the	value	
,	the	numeric	value	written	on	the	right.
Having	bytes	appear	in	reverse	order	is	a	common	occurrence	when
reading	machine-level	program	representations	generated	for	little-
endian	machines	such	as	this	one.	The	natural	way	to	write	a	byte
sequence	is	to	have	the	lowest-numbered	byte	on	the	left	and	the	highest

on	the	right,	but	this	is	contrary	to	the	normal	way	of	writing	numbers	with
the	most	significant	digit	on	the	left	and	the	least	on	the	right.
A	third	case	where	byte	ordering	becomes	visible	is	when	programs	are
written	that	circumvent	the	normal	type	system.	In	the	C	language,	this
can	be	done	using	a	
cast
	or	a	
union
	to	allow	an	object	to	be	referenced
according	to	a	different	data	type	from	which	it	was	created.	Such	coding
tricks	are	strongly	discouraged	for	most	application	programming,	but
they	can	be	quite	useful	and	even	necessary	for	system-level
programming.
Figure	
2.4
	shows	C	code	that	uses	casting	to	access	and	print	the
byte	representations	of	different	program	objects.	We	use	
	to
define	data	type	
	as	a	pointer	to	an	object	of	type	
	Such	a	byte	pointer	references	a	sequence	of	bytes	where	each
byte	is	considered	to	be	a	nonnegative	integer.	The	first	routine
	is	given	the	address	of	a	sequence	of	bytes,	indicated	by	a
byte	pointer,	and	a	byte	count.	The	byte	count	is	specified	as	having	data
type	
,	the	preferred	data	type	for	expressing	the	sizes	of	data
structures.	It	prints	the	individual	bytes	in	hexadecimal.	The	C	formatting
directive	
	indicates	that	an	integer	should	be	printed	in	hexadecimal
with	at	least	2	digits.

Figure	
2.4	
Code	to	print	the	byte	representation	of	program	objects.
This	code	uses	casting	to	circumvent	the	type	system.	Similar	functions
are	easily	defined	for	other	data	types.
Procedures	
,	and	
	demonstrate	how	to
use	procedure	
	to	print	the	byte	representations	of	C	program
objects	of	type	
,	and	
,	respectively.	Observe	that	they
simply	pass	
	a	pointer	
	to	their	argument	
,	casting	the
pointer	to	be	of	type	
.	This	cast	indicates	to	the	compiler
that	the	program	should	consider	the	pointer	to	be	to	a	sequence	of	bytes

rather	than	to	an	object	of	the	original	data	type.	This	pointer	will	then	be
to	the	lowest	byte	address	occupied	by	the	object.
These	procedures	use	the	C	
	to	determine	the	number
of	bytes	used	by	the	object.	In	general,	the	expression	
	returns
the	number	of	bytes	required	to	store	an	object	of	type	
T
.	Using	
rather	than	a	fixed	value	is	one	step	toward	writing	code	that	is	portable
across	different	machine	types.
We	ran	the	code	shown	in	
Figure	
2.5
	on	several	different	machines,
giving	the	results	shown	in	
Figure	
2.6
.	The	following	machines	were
used:
Linux	32
Intel	IA32	processor	running	Linux.
Windows
Intel	IA32	processor	running	Windows.
Sun
Sun	Microsystems	SPARC	processor	running	Solaris.	(These	machines	are	now
produced	by	Oracle.)
Linux	64
Intel	x86–64	processor	running	Linux.

Figure	
2.5	
Byte	representation	examples.
This	code	prints	the	byte	representations	of	sample	data	objects.
Machine
Value
Type
Bytes	(hex)
Linux	32
12,345
Windows
12,345
Sun
12,345
Linux	64
12,345
Linux	32
12,345.0
Windows
12,345.0
Sun
12,345.0
Linux	64
12,345.0
Linux	32
Windows
Sun
Linux	64
Figure	
2.6	
Byte	representations	of	different	data	values.
Results	for	
	and	
	are	identical,	except	for	byte	ordering.	Pointer
values	are	machine	dependent.

Our	argument	12,345	has	hexadecimal	representation	
.	For
the	
	data,	we	get	identical	results	for	all	machines,	except	for	the	byte
ordering.	In	particular,	we	can	see	that	the	least	significant	byte	value	of
	is	printed	first	for	Linux	32,	Windows,	and	Linux	64,	indicating	little-
endian	machines,	and	last	for	Sun,	indicating	a	big-endian	machine.
Similarly,	the	bytes	of	the	
	data	are	identical,	except	for	the	byte
ordering.	On	the	other	hand,	the	pointer	values	are	completely	different.
The	different	machine/operating	system	configurations	use	different
conventions	for	storage	allocation.	One	feature	to	note	is	that	the	Linux
32,	Windows,	and	Sun	machines	use	4-byte	addresses,	while	the	Linux
64	machine	uses	8-byte	addresses.
New	to	C?	
Naming	data	types	with
The	
	declaration	in	C	provides	a	way	of	giving	a	name	to	a
data	type.	This	can	be	a	great	help	in	improving	code	readability,
since	deeply	nested	type	declarations	can	be	difficult	to	decipher.
The	syntax	for	
	is	exactly	like	that	of	declaring	a	variable,
except	that	it	uses	a	type	name	rather	than	a	variable	name.	Thus,
the	declaration	of	
	in	
Figure	
2.4
	has	the	same
form	as	the	declaration	of	a	variable	of	type	
For	example,	the	declaration

defines	type	
	be	a	pointer	to	an	int,	and	declares	a
variable	
	of	this	type.	Alternatively,	we	could	declare	this
variable	directly	as
New	to	C?	
Formatted	printing	with
The	
	function	(along	with	its	cousins	
	and	
)
provides	a	way	to	print	information	with	considerable	control	over
the	formatting	details.	The	first	argument	is	a	
format	string
,	while
any	remaining	arguments	are	values	to	be	printed.	Within	the
format	string,	each	character	sequence	starting	with	
	indicates
how	to	format	the	next	argument.	Typical	examples	include	
to	print	a	decimal	integer,	
	to	print	a	floating-point	number,
and	
	to	print	a	character	having	the	character	code	given	by
the	argument.
Specifying	the	formatting	of	fixed-size	data	types,	such	as
,	is	a	bit	more	involved,	as	is	described	in	the	aside	on
page	
67
.

Observe	that	although	the	floating-point	and	the	integer	data	both	encode
the	numeric	value	12,345,	they	have	very	different	byte	patterns:
	for	the	integer	and	
	for	floating	point.	In	general,
these	two	formats	use	different	encoding	schemes.	If	we	expand	these
hexadecimal	patterns	into	binary	form	and	shift	them	appropriately,	we
find	a	sequence	of	13	matching	bits,	indicated	by	a	sequence	of
asterisks,	as	follows:
This	is	not	coincidental.	We	will	return	to	this	example	when	we	study
floating-point	formats.
New	to	C?	
Pointers	and	arrays
In	function	
	(
Figure	
2.4
),	we	see	the	close
connection	between	pointers	and	arrays,	as	will	be	discussed	in
detail	in	
Section	
3.8
.	We	see	that	this	function	has	an	argument
	of	type	
	(which	has	been	defined	to	be	a
pointer	to	
),	but	we	see	the	array	reference	
on	line	8.	In	C,	we	can	dereference	a	pointer	with	array	notation,
and	we	can	reference	array	elements	with	pointer	notation.	In	this
example,	the	reference	
	indicates	that	we	want	to	read
the	byte	that	is	
	positions	beyond	the	location	pointed	to	by
.

New	to	C?	
Pointer	creation	and
dereferencing
In	lines	13,	17,	and	21	of	
Figure	
2.4
	we	see	uses	of	two
operations	that	give	C	(and	therefore	C++)	its	distinctive	character.
The	C	“address	of”	operator	
	creates	a	pointer.	On	all	three	lines,
the	expression	
	creates	a	pointer	to	the	location	holding	the
object	indicated	by	variable	
.	The	type	of	this	pointer	depends	on
the	type	of	
,	and	hence	these	three	pointers	are	of	type	
,	and	
,	respectively.	(Data	type	
	
	is	a	special
kind	of	pointer	with	no	associated	type	information.)
The	cast	operator	converts	from	one	data	type	to	another.	Thus,
the	cast	(
)	
	indicates	that	whatever	type	the	pointer
	had	before,	the	program	will	now	reference	a	pointer	to	data	of
type	
.	The	casts	shown	here	do	not	change	the
actual	pointer;	they	simply	direct	the	compiler	to	refer	to	the	data
being	pointed	to	according	to	the	new	data	type.

Aside	
Generating	an	ASCII	table
You	can	display	a	table	showing	the	ASCII	character	code	by
executing	the	command	
Practice	Problem	
2.5
	(solution	page	
144
)
Consider	the	following	three	calls	to	
:
Indicate	the	values	that	will	be	printed	by	each	call	on	a	little-
endian	machine	and	on	a	big-endian	machine:
A
.	
Little	endian:
														
Big	endian:
													
B
.	
Little	endian:
														
Big	endian:
													
C
.	
Little	endian:
														
Big	endian:
													
Practice	Problem	
2.6
	(solution	page	
145
)
Using	
	and	
,	we	determine	that	the	integer
3510593	has	hexadecimal	representation	
,	while	the
floating-point	number	3510593.0	has	hexadecimal	representation

A
.	
Write	the	binary	representations	of	these	two	hexadecimal
values.
B
.	
Shift	these	two	strings	relative	to	one	another	to	maximize
the	number	of	matching	bits.	How	many	bits	match?
C
.	
What	parts	of	the	strings	do	not	match?
2.1.4	
Representing	Strings
A	string	in	C	is	encoded	by	an	array	of	characters	terminated	by	the	null
(having	value	0)	character.	Each	character	is	represented	by	some
standard	encoding,	with	the	most	common	being	the	ASCII	character
code.	Thus,	if	we	run	our	routine	
	with	arguments	
	and
	(to	include	the	terminating	character),	we	get	the	result	
.	Observe	that	the	ASCII	code	for	decimal	digit	
x
	happens	to	be	
,
and	that	the	terminating	byte	has	the	hex	representation	
.	This	same
result	would	be	obtained	on	any	system	using	ASCII	as	its	character
code,	independent	of	the	byte	ordering	and	word	size	conventions.	As	a
consequence,	text	data	are	more	platform	independent	than	binary	data.
Practice	Problem	
2.7
	(solution	page	
145
)
What	would	be	printed	as	a	result	of	the	following	call	to

Note	that	letters	
	through	
	have	ASCII	codes	
	through
.
2.1.5	
Representing	Code
Consider	the	following	C	function:
When	compiled	on	our	sample	machines,	we	generate	machine	code
having	the	following	byte	representations:
Linux	32
Windows
Sun
Linux	64
Aside	
The	Unicode	standard	for	text
encoding

The	ASCII	character	set	is	suitable	for	encoding	English-language
documents,	but	it	does	not	have	much	in	the	way	of	special
characters,	such	as	the	French	‘ç'.	It	is	wholly	unsuited	for
encoding	documents	in	languages	such	as	Greek,	Russian,	and
Chinese.	Over	the	years,	a	variety	of	methods	have	been
developed	to	encode	text	for	different	languages.	The	Unicode
Consortium	has	devised	the	most	comprehensive	and	widely
accepted	standard	for	encoding	text.	The	current	Unicode
standard	(version	7.0)	has	a	repertoire	of	over	100,000	characters
supporting	a	wide	range	of	languages,	including	the	ancient
languages	of	Egypt	and	Babylon.	To	their	credit,	the	Unicode
Technical	Committee	rejected	a	proposal	to	include	a	standard
writing	for	Klingon,	a	fictional	civilization	from	the	television	series
Star	Trek.
The	base	encoding,	known	as	the	“Universal	Character	Set”	of
Unicode,	uses	a	32-bit	representation	of	characters.	This	would
seem	to	require	every	string	of	text	to	consist	of	4	bytes	per
character.	However,	alternative	codings	are	possible	where
common	characters	require	just	1	or	2	bytes,	while	less	common
ones	require	more.	In	particular,	the	UTF-8	representation
encodes	each	character	as	a	sequence	of	bytes,	such	that	the
standard	ASCII	characters	use	the	same	single-byte	encodings	as
they	have	in	ASCII,	implying	that	all	ASCII	byte	sequences	have
the	same	meaning	in	UTF-8	as	they	do	in	ASCII.
The	Java	programming	language	uses	Unicode	in	its
representations	of	strings.	Program	libraries	are	also	available	for
C	to	support	Unicode.
Here	we	find	that	the	instruction	codings	are	different.	Different	machine
types	use	different	and	incompatible	instructions	and	encodings.	Even

identical	processors	running	different	operating	systems	have	differences
in	their	coding	conventions	and	hence	are	not	binary	compatible.	Binary
code	is	seldom	portable	across	different	combinations	of	machine	and
operating	system.
A	fundamental	concept	of	computer	systems	is	that	a	program,	from	the
perspective	of	the	machine,	is	simply	a	sequence	of	bytes.	The	machine
has	no	information	about	the	original	source	program,	except	perhaps
some	auxiliary	tables	maintained	to	aid	in	debugging.	We	will	see	this
more	clearly	when	we	study	machine-level	programming	in	
Chapter	
3
.
2.1.6	
Introduction	to	Boolean
Algebra
Since	binary	values	are	at	the	core	of	how	computers	encode,	store,	and
manipulate	information,	a	rich	body	of	mathematical	knowledge	has
evolved	around	the	study	of	the	values	0	and	1.	This	started	with	the
work	of	George	Boole	(1815–1864)	around	1850	and	thus	is	known	as
Boolean	algebra
.	Boole	observed	that	by	encoding	logic	values	
TRUE
	
and
FALSE
	
as	binary	values	1	and	0,	he	could	formulate	an	algebra	that
captures	the	basic	principles	of	logical	reasoning.
The	simplest	Boolean	algebra	is	defined	over	the	two-element	set	{0,	1}.
Figure	
2.7
	defines	several	operations	in	this	algebra.	Our	symbols	for
representing	these	operations	are	chosen	to	match	those	used	by	the	C
bit-level	operations,

Figure	
2.7	
Operations	of	Boolean	algebra.
Binary	values	1	and	0	encode	logic	values	
TRUE
	
and	
FALSE
,	while
operations	~,	&,	|,	and	^	encode	logical	operations	
NOT
,	
AND
,	
OR
,	and
EXCLUSIVE
-
OR
,	respectively.
as	will	be	discussed	later.	The	Boolean	operation	~	corresponds	to	the
logical	operation	
NOT
,	denoted	by	the	symbol	¬.	That	is,	we	say	that	¬
P
	is
true	when	
P
	is	not	true,	and	vice	versa.	Correspondingly,	~
p
	equals	1
when	
p
	equals	0,	and	vice	versa.	Boolean	operation	
	corresponds	to
the	logical	operation	
AND
,	denoted	by	the	symbol	
∧
.	We	say	that	
P
	
∧
	
Q
holds	when	both	
P
	is	true	and	
Q
	is	true.	Correspondingly,	
p
	
	
q
	equals	1
only	when	
p
	=	1	and	
q
	=	1.	Boolean	operation	|	corresponds	to	the	logical
operation	
OR
,	denoted	by	the	symbol	
∨
.	We	say	that	
P
	
∨
	
Q
	holds	when
either	
P
	is	true	or	
Q
	is	true.	Correspondingly,	
p
	|	
q
	equals	1	when	either	
p
=	1	or	
q
	=	1.	Boolean	operation	^	corresponds	to	the	logical	operation
EXCLUSIVE
-
OR
,	denoted	by	the	symbol	
⊕
.	We	say	that	
P
	
⊕
	
Q
	holds	when
either	
P
	is	true	or	
Q
	is	true,	but	not	both.	Correspondingly,	
p
	^	
q
	equals	1
when	either	
p
	=	1	and	
q
	=	0,	or	
p
	=	0	and	
q
	=	1.
Claude	Shannon	(1916–2001),	who	later	founded	the	field	of	information
theory,	first	made	the	connection	between	Boolean	algebra	and	digital
logic.	In	his	1937	master's	thesis,	he	showed	that	Boolean	algebra	could
be	applied	to	the	design	and	analysis	of	networks	of	electromechanical
relays.	Although	computer	technology	has	advanced	considerably	since,
Boolean	algebra	still	plays	a	central	role	in	the	design	and	analysis	of
digital	systems.

We	can	extend	the	four	Boolean	operations	to	also	operate	on	
bit
vectors
,	strings	of	zeros	and	ones	of	some	fixed	length	
w
.	We	define	the
operations	over	bit	vectors	according	to	their	applications	to	the	matching
elements	of	the	arguments.	Let	
a
	and	
b
	denote	the	bit	vectors	
and	
,	respectively.	We	define	
a
	&	
b
	to	also
be	a	bit	vector	of	length	
w
,	where	the	
i
th	element	equals	
a
	&	
b
,	for	0	≤	
i
	<
w.
	The	operations	|,	^,	and	~	are	extended	to	bit	vectors	in	a	similar
fashion.
As	examples,	consider	the	case	where	
w
	=	4,	and	with	arguments	
a
	=
[0110]	and	
b
	=	[1100].	Then	the	four	operations	
a	&	b,	a
	|	
b,	a
	^	
b
,	and	~
b
yield
Practice	Problem	
2.8
	(solution	page	
145
)
Fill	in	the	following	table	showing	the	results	of	evaluating	Boolean
operations	on	bit	vectors.
Web	Aside	DATA:BOOL	
More	on
Boolean	algebra	and	Boolean	rings
The	Boolean	operations	
	operating	on	bit
vectors	of	length	
w
	form	a	
Boolean	algebra
,	for	any	integer
w
	>	0.	The	simplest	is	the	case	where	
w
	=	1	and	there	are
just	two	elements,	but	for	the	more	general	case	there	are
2
	bit	vectors	of	length	
w.
	Boolean	algebra	has	many	of	the
same	properties	as	arithmetic	over	integers.	For	example,
[
a
w
−
1
,
 
a
w
−
2
,
…
,
a
0
]
	
[
b
w
−
1
,
 
b
w
−
2
,
…
,
b
0
]
i
i
0110
&
1100
0100
¯
0110
|
1100
1110
¯
0110
^
1100
1010
¯
~
1100
0011
¯
w

just	as	multiplication	distributes	over	addition,	written	
a
	·	(
b
+	
c
)	=	(
a
	·	
b
)	+	(
a
	·	
c
),	Boolean	operation	&	distributes	over
|,	written	
a
	&	(
b
	|	
c
)	=	(
a
	&	
b
)	|	(
a
	&	
c
).	In	addition,	however.
Boolean	operation	|	distributes	over	&,	and	so	we	can	write
a
	|	(
b
	&	
c
)	=	(
a
	|	
b
)	&	(
a
	|	
c
),	whereas	we	cannot	say	that	
a
+	(
b
	·	
c
)	=	(
a
	+	
b
)	·	(
a
	+	
c
)	holds	for	all	integers.
When	we	consider	operations	^,	&,	and	~	operating	on	bit
vectors	of	length	
w
,	we	get	a	different	mathematical	form,
known	as	a	
Boolean	ring.
	Boolean	rings	have	many
properties	in	common	with	integer	arithmetic.	For	example,
one	property	of	integer	arithmetic	is	that	every	value	
x
	has
an	
additive	inverse	–x
,	such	that	
x
	+	–
x
	=	0.	A	similar
property	holds	for	Boolean	rings,	where	^	is	the	“addition”
operation,	but	in	this	case	each	element	is	its	own	additive
inverse.	That	is,	
a
	^	
a
	=	0	for	any	value	
a
,	where	we	use	0
here	to	represent	a	bit	vector	of	all	zeros.	We	can	see	this
holds	for	single	bits,	since	0	^	0	=	1	^	1	=	0,	and	it	extends
to	bit	vectors	as	well.	This	property	holds	even	when	we
rearrange	terms	and	combine	them	in	a	different	order,	and
so	(
a
	^	
b
)	^	
a
	=	
b.
	This	property	leads	to	some	interesting
results	and	clever	tricks,	as	we	will	explore	in	
Problem
2.10
.
Operation
Result
a
[01101001]
b
[01010101]
~
a
__________
~
b
__________

a
	&	
b
__________
a
	|	
b
__________
a
	^	
b
__________
One	useful	application	of	bit	vectors	is	to	represent	finite	sets.	We	can
encode	any	subset	
with	a	bit	vector	
,
where	
a
	=	1	if	and	only	if	
i
	
∊
	
A.
	For	example,	recalling	that	we	write	
a
on	the	left	and	
a
	on	the	right,	bit	vector	
a
	=	[01101001]	encodes	the	set
A
	=	{0,	3,	5,	6},	while	bit	vector	
b
	=	[01010101]	encodes	the	set	
B
	=	{0,	2,
4,	6}.	With	this	way	of	encoding	sets,	Boolean	operations	|	and	&
correspond	to	set	union	and	intersection,	respectively,	and	~	corresponds
to	set	complement.	Continuing	our	earlier	example,	the	operation	
a
	&	
b
yields	bit	vector	[01000001],	while	
A
	∩	
B
	=	{0,	6}.
We	will	see	the	encoding	of	sets	by	bit	vectors	in	a	number	of	practical
applications.	For	example,	in	
Chapter	
8
,	we	will	see	that	there	are	a
number	of	different	
signals
	that	can	interrupt	the	execution	of	a	program.
We	can	selectively	enable	or	disable	different	signals	by	specifying	a	bit-
vector	mask,	where	a	1	in	bit	position	
i
	indicates	that	signal	
i
	is	enabled
and	a	0	indicates	that	it	is	disabled.	Thus,	the	mask	represents	the	set	of
enabled	signals.
Practice	Problem	
2.9
	(solution	page	
146
)
Computers	generate	color	pictures	on	a	video	screen	or	liquid
crystal	display	by	mixing	three	different	colors	of	light:	red,	green,
and	blue.	Imagine	a	simple	scheme,	with	three	different	lights,
A
⊆
{
0
,
1
,
…
,
w
−
1
}
	
[
a
w
−
1
,
…
,
a
1
,
a
0
]
i
w
–1
0

each	of	which	can	be	turned	on	or	off,	projecting	onto	a	glass
screen:
We	can	then	create	eight	different	colors	based	on	the	absence
(0)	or	presence	(1)	of	light	sources	
R
,	
G
,	and	
B
:
R
G
B
Color
0
0
0
Black
0
0
1
Blue
0
1
0
Green
0
1
1
Cyan
1
0
0
Red
1
0
1
Magenta
1
1
0
Yellow
1
1
1
White

Each	of	these	colors	can	be	represented	as	a	bit	vector	of	length
3,	and	we	can	apply	Boolean	operations	to	them.
A
.	
The	complement	of	a	color	is	formed	by	turning	off	the
lights	that	are	on	and	turning	on	the	lights	that	are	off.	What
would	be	the	complement	of	each	of	the	eight	colors	listed
above?
B
.	
Describe	the	effect	of	applying	Boolean	operations	on	the
following	colors:
Blue	|	Green	=__________
Yellow	&	Cyan	=__________
Red	^	Magenta	=__________
2.1.7	
Bit-Level	Operations	in	C
One	useful	feature	of	C	is	that	it	supports	bitwise	Boolean	operations.	In
fact,	the	symbols	we	have	used	for	the	Boolean	operations	are	exactly
those	used	by	C:	|	for	
OR
,	&	for	
AND
,	~	for	
NOT
,	and	^	for	
EXCLUSIVE
-
OR
.
These	can	be	applied	to	any	“integral”	data	type,	including	all	of	those
listed	in	
Figure	
2.3
.	Here	are	some	examples	of	expression	evaluation
for	data	type	char:
C	expression
Binary	expression
Binary	result
Hexadecimal	result
~[0100	0001]
[1011	1110]
~[0000	0000]
[1111	1111]
[0110	1001]	&	[0101	0101]
[0100	0001]

[0110	1001]	|	[01010101]
[0111	1101]
As	our	examples	show,	the	best	way	to	determine	the	effect	of	a	bit-level
expression	is	to	expand	the	hexadecimal	arguments	to	their	binary
representations,	perform	the	operations	in	binary,	and	then	convert	back
to	hexadecimal.
Practice	Problem	
2.10
	(solution	page	
146
)
As	an	application	of	the	property	that	
a
	^	
a
	=	0	for	any	bit	vector	
a
,
consider	the	following	program:
As	the	name	implies,	we	claim	that	the	effect	of	this	procedure	is
to	swap	the	values	stored	at	the	locations	denoted	by	pointer
variables	
	and	
.	Note	that	unlike	the	usual	technique	for
swapping	two	values,	we	do	not	need	a	third	location	to
temporarily	store	one	value	while	we	are	moving	the	other.	There
is	no	performance	advantage	to	this	way	of	swapping;	it	is	merely
an	intellectual	amusement.
Starting	with	values	
a
	and	
b
	in	the	locations	pointed	to	by	
	and
,	respectively,	fill	in	the	table	that	follows,	giving	the	values

stored	at	the	two	locations	after	each	step	of	the	procedure.	Use
the	properties	of	^	to	show	that	the	desired	effect	is	achieved.
Recall	that	every	element	is	its	own	additive	inverse	(that	is,	
a
	^	
a
=	0).
Step
*x
*y
Initially
a
b
Step	1
__________
__________
Step	2
__________
__________
Step	3
__________
__________
Practice	Problem	
2.11
	(solution	page	
146
)
Armed	with	the	function	
	from	
Problem	
2.10
,	you
decide	to	write	code	that	will	reverse	the	elements	of	an	array	by
swapping	elements	from	opposite	ends	of	the	array,	working
toward	the	middle.
You	arrive	at	the	following	function:

When	you	apply	your	function	to	an	array	containing	elements	1,
2,	3,	and	4,	you	find	the	array	now	has,	as	expected,	elements	4,
3,	2,	and	1.	When	you	try	it	on	an	array	with	elements	1,	2,	3,	4,
and	5,	however,	you	are	surprised	to	see	that	the	array	now	has
elements	5,	4,	0,	2,	and	1.	In	fact,	you	discover	that	the	code
always	works	correctly	on	arrays	of	even	length,	but	it	sets	the
middle	element	to	0	whenever	the	array	has	odd	length.
A
.	
For	an	array	of	odd	length	
,	what	are	the
values	of	variables	first	and	last	in	the	final	iteration	of
function	
B
.	
Why	does	this	call	to	function	
	set	the	array
element	to	0?
C
.	
What	simple	modification	to	the	code	for	
would	eliminate	this	problem?
One	common	use	of	bit-level	operations	is	to	implement	
masking
operations,	where	a	mask	is	a	bit	pattern	that	indicates	a	selected	set	of
bits	within	a	word.	As	an	example,	the	mask	
	(having	ones	for	the
least	significant	8	bits)	indicates	the	low-order	byte	of	a	word.	The	bit-
level	operation	
	yields	a	value	consisting	of	the	least	significant
byte	of	
,	but	with	all	other	bytes	set	to	0.	For	example,	with	
,	the	expression	would	yield	
.	The	expression	
will	yield	a	mask	of	all	ones,	regardless	of	the	size	of	the	data
representation.	The	same	mask	can	be	written	
	when	data
type	
	is	32	bits,	but	it	would	not	be	as	portable.
Practice	Problem	
2.12
	(solution	page	
146
)

Write	C	expressions,	in	terms	of	variable	
,	for	the	following
values.	Your	code	should	work	for	any	word	size	
w
	≥	8.	For
reference,	we	show	the	result	of	evaluating	the	expressions	for	
,	with	
w
	=	32.
A
.	
The	least	significant	byte	of	
,	with	all	other	bits	set	to	0.
[
]
B
.	
All	but	the	least	significant	byte	of	
	complemented,	with
the	least	significant	byte	left	unchanged.	
C
.	
The	least	significant	byte	set	to	all	ones,	and	all	other	bytes
of	
	left	unchanged.	
Practice	Problem	
2.13
	(solution	page	
147
)
The	Digital	Equipment	VAX	computer	was	a	very	popular	machine
from	the	late	1970s	until	the	late	1980s.	Rather	than	instructions
for	Boolean	operations	
AND
	
and	
OR
,	it	had	instructions	
	(bit	set)
and	
	(bit	clear).	Both	instructions	take	a	data	word	
	and	a
mask	word	
.	They	generate	a	result	
	consisting	of	the	bits	of	
modified	according	to	the	bits	of	
	With	
,	the	modification
involves	setting	
	to	1	at	each	bit	position	where	
	is	1.	With	
,
the	modification	involves	setting	
	to	0	at	each	bit	position	where
	is	1.
To	see	how	these	operations	relate	to	the	C	bit-level	operations,
assume	we	have	functions	
	and	
	implementing	the	bit	set
and	bit	clear	operations,	and	that	we	want	to	use	these	to
implement	functions	computing	bitwise	operations	|	and	^,	without
using	any	other	C	operations.	Fill	in	the	missing	code	below.	
Hint:
Write	C	expressions	for	the	operations	
	and	
.

2.1.8	
Logical	Operations	in	C
C	also	provides	a	set	of	
logical
	operators	|	|,	&&,	and	!,	which	correspond
to	the	
OR
,	
AND
,	and	
NOT
	
operations	of	logic.	These	can	easily	be	confused
with	the	bit-level	operations,	but	their	behavior	is	quite	different.	The
logical	operations	treat	any	nonzero	argument	as	representing	
TRUE
	
and
argument	0	as	representing	
FALSE
.	They	return	either	1	or	0,	indicating	a

result	of	either	
TRUE
	
OR
	
FALSE
,	respectively.	Here	are	some	examples	of
expression	evaluation:
Expression
Result
Observe	that	a	bitwise	operation	will	have	behavior	matching	that	of	its
logical	counterpart	only	in	the	special	case	in	which	the	arguments	are
restricted	to	0	or	1.
A	second	important	distinction	between	the	logical	operators	‘
’	and	‘
’	versus	their	bit-level	counterparts	‘
’	and	‘
’	is	that	the	logical
operators	do	not	evaluate	their	second	argument	if	the	result	of	the
expression	can	be	determined	by	evaluating	the	first	argument.	Thus,	for
example,	the	expression	a	
	will	never	cause	a	division	by	zero,
and	the	expression	
	will	never	cause	the	dereferencing	of	a	null
pointer.
Practice	Problem	
2.14
	(solution	page	
147
)

Suppose	that	
	and	
	have	byte	values	
	and	
,
respectively.	Fill	in	the	following	table	indicating	the	byte	values	of
the	different	C	expressions:
Expression
Value
Expression
Value
__________
__________
__________
__________
__________
__________
__________
__________
Practice	Problem	
2.15
	(solution	page	
148
)
Using	only	bit-level	and	logical	operations,	write	a	C	expression
that	is	equivalent	to	
.	In	other	words,	it	will	return	1	when	
and	
	are	equal	and	0	otherwise.
2.1.9	
Shift	Operations	in	C
C	also	provides	a	set	of	
shift
	operations	for	shifting	bit	patterns	to	the	left
and	to	the	right.	For	an	operand	
	having	bit	representation	
,	the	C	expression	
	yields	a	value	with	bit	representation	
.	That	is,	
	is	shifted	
k
	bits	to	the	left,	dropping
off	the	
k
	most	significant	bits	and	filling	the	right	end	with	
k
	zeros.	The
shift	amount	should	be	a	value	between	0	and	
w
	–	1.	Shift	operations
associate	from	left	to	right,	so	
	is	equivalent	to	
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

There	is	a	corresponding	right	shift	operation,	written	in	C	as	
,	but
it	has	a	slightly	subtle	behavior.	Generally,	machines	support	two	forms	of
right	shift:
Logical	
.	A	logical	right	shift	fills	the	left	end	with	
k
	zeros,	giving	a
result	
.
Arithmetic
.	An	arithmetic	right	shift	fills	the	left	end	with	
k
	repetitions
of	the	most	significant	bit,	giving	a	result	
.	This	convention	might	seem	peculiar,	but	as	we	will	see,	it	is
useful	for	operating	on	signed	integer	data.
As	examples,	the	following	table	shows	the	effect	of	applying	the	different
shift	operations	to	two	different	values	of	an	8-bit	argument	
x
:
Operation
Value	1
Value	2
Argument	
[01100011]
[10010101]
[0011
0000
]
[0101
0000
]
[
0000
0110]
[
0000
1001]
	(arithmetic)
[
0000
0110]
[
1111
1001]
The	italicized	digits	indicate	the	values	that	fill	the	right	(left	shift)	or	left
(right	shift)	ends.	Observe	that	all	but	one	entry	involves	filling	with	zeros.
The	exception	is	the	case	of	shifting	[10010101]	right	arithmetically.	Since
its	most	significant	bit	is	1,	this	will	be	used	as	the	fill	value.
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
x
k
]
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
x
k
]

The	C	standards	do	not	precisely	define	which	type	of	right	shift	should
be	used	with	signed	numbers—either	arithmetic	or	logical	shifts	may	be
used.	This	unfortunately	means	that	any	code	assuming	one	form	or	the
other	will	potentially	encounter	portability	problems.	In	practice,	however,
almost	all	compiler/machine	combinations	use	arithmetic	right	shifts	for
signed	data,	and	many	programmers	assume	this	to	be	the	case.	For
unsigned	data,	on	the	other	hand,	right	shifts	must	be	logical.
In	contrast	to	C,	Java	has	a	precise	definition	of	how	right	shifts	should
be	performed.	The	expression	
	shifts	
	arithmetically	by	
positions,	while	
	shifts	it	logically.
Practice	Problem	
2.16
	(solution	page	
148
)
Fill	in	the	table	below	showing	the	effects	of	the	different	shift
operations	on	single-byte	quantities.	The	best	way	to	think	about
shift	operations	is	to	work	with	binary	representations.	Convert	the
initial	values	to	binary,	perform	the	shifts,	and	then	convert	back	to
hexadecimal.	Each	of	the	answers	should	be	8	binary	digits	or	2
hexadecimal	digits.
Logical	
Hex
Binary
Binary
Hex
Binary
Hex
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

Aside	
Shifting	by	
k
,	for	large	values	of	
k
For	a	data	type	consisting	of	
w
	bits,	what	should	be	the	effect	of
shifting	by	some	value	
k
	≥	
w
?	For	example,	what	should	be	the
effect	of	computing	the	following	expressions,	assuming	data	type
	has	
w
	=	32:
The	C	standards	carefully	avoid	stating	what	should	be	done	in
such	a	case.	On	many	machines,	the	shift	instructions	consider
only	the	lower	log
	
w
	bits	of	the	shift	amount	when	shifting	a	
w
-bit
value,	and	so	the	shift	amount	is	computed	as	
k
	mod	
w
.	For
example,	with	
w
	=	32,	the	above	three	shifts	would	be	computed
as	if	they	were	by	amounts	0,	4,	and	8,	respectively,	giving	results
This	behavior	is	not	guaranteed	for	C	programs,	however,	and	so
shift	amounts	should	be	kept	less	than	the	word	size.
2

Java,	on	the	other	hand,	specifically	requires	that	shift	amounts
should	be	computed	in	the	modular	fashion	we	have	shown.
Aside	
Operator	precedence	issues	with
shift	operations
It	might	be	tempting	to	write	the	expression	
,	intending
it	to	mean	
.	However,	in	C	the	former	expression
is	equivalent	to	
,	since	addition	(and	subtraction)
have	higher	precedence	than	shifts.	The	left-to-right	associativity
rule	then	causes	this	to	be	parenthesized	as	
,
giving	value	512,	rather	than	the	intended	52.
Getting	the	precedence	wrong	in	C	expressions	is	a	common
source	of	program	errors,	and	often	these	are	difficult	to	spot	by
inspection.	When	in	doubt,	put	in	parentheses!

2.2	
Integer	Representations
In	this	section,	we	describe	two	different	ways	bits	can	be	used	to	encode
integers—one	that	can	only	represent	nonnegative	numbers,	and	one
that	can	represent	negative,	zero,	and	positive	numbers.	We	will	see	later
that	they	are	strongly	related	both	in	their	mathematical	properties	and
their	machine-level	implementations.	We	also	investigate	the	effect	of
expanding	or	shrinking	an	encoded	integer	to	fit	a	representation	with	a
different	length.
Figure	
2.8
	lists	the	mathematical	terminology	we	introduce	to	precisely
define	and	characterize	how	computers	encode	and	operate	on	integer
data.	This
Symbol
Type
Meaning
Page
B2T
Function
Binary	to	two's	complement
64
B2U
Function
Binary	to	unsigned
62
U2B
Function
Unsigned	to	binary
64
U2T
Function
Unsigned	to	two's	complement
71
T2B
Function
Two's	complement	to	binary
65
T2U
Function
Two's	complement	to	unsigned
71
TMin
Constant
Minimum	two's-complement	value
65
w
w
w
w
w
w
w

TMax
Constant
Maximum	two's-complement	value
65
UMax
Constant
Maximum	unsigned	value
63
Operation
Two's-complement	addition
90
Operation
Unsigned	addition
85
Operation
Two's-complement	multiplication
97
Operation
Unsigned	multiplication
96
Operation
Two's-complement	negation
95
Operation
Unsigned	negation
89
Figure	
2.8	
Terminology	for	integer	data	and	arithmetic	operations.
The	subscript	
w
	denotes	the	number	of	bits	in	the	data	representation.
The	“Page”	column	indicates	the	page	on	which	the	term	is	defined.
terminology	will	be	introduced	over	the	course	of	the	presentation.	The
figure	is	included	here	as	a	reference.
2.2.1	
Integral	Data	Types
C	supports	a	variety	of	
integral
	data	types—ones	that	represent	finite
ranges	of	integers.	These	are	shown	in	
Figures	
2.9
	and	
2.10
,	along
with	the	ranges	of	values	they	can	have	for	“typical”	32-	and	64-bit
programs.	Each	type	can	specify	a	size	with	keyword	char,	short,	long,	as
well	as	an	indication	of	whether	the	represented	numbers	are	all
nonnegative	(declared	as	
),	or	possibly	negative	(the	default.)	As
w
w
+
w
t
+
w
u
*
w
t
*
w
u
−
w
t
−
w
u

we	saw	in	
Figure	
2.3
,	the	number	of	bytes	allocated	for	the	different
sizes	varies	according	to	whether	the	program	is	compiled	for	32	or	64
bits.	Based	on	the	byte	allocations,	the	different	sizes	allow	different
ranges	of	values	to	be	represented.	The	only	machine-dependent	range
indicated	is	for	size	designator	
	Most	64-bit	programs	use	an	8-byte
representation,	giving	a	much	wider	range	of	values	than	the	4-byte
representation	used	with	32-bit	programs.
One	important	feature	to	note	in	
Figures	
2.9
	and	
2.10
	is	that	the
ranges	are	not	symmetric—the	range	of	negative	numbers	extends	one
further	than	the	range	of	positive	numbers.	We	will	see	why	this	happens
when	we	consider	how	negative	numbers	are	represented.
C	data	type
Minimum
Maximum
–128
127
0
255
–32,768
32,767
0
65,535
–2,147,483,648
2,147,483,647
0
4,294,967,295
–2,147,483,648
2,147,483,647
0
4,294,967,295
–2,147,483,648
2,147,483,647

0
4,294,967,295
–9,223,372,036,854,775,808
9,223,372,036,854,775,807
0
18,446,744,073,709,551,615
Figure	
2.9	
Typical	ranges	for	C	integral	data	types	for	32-bit
programs.
C	data	type
Minimum
Maximum
−128
127
0
255
–32,768
32,767
0
65,535
–2,147,483,648
2,147,483,647
0
4,294,967,295
–9,223,372,036,854,775,808
9,223,372,036,854,775,807
0
18,446,744,073,709,551,615
–2,147,483,648
2,147,483,647
0
4,294,967,295
–9,223,372,036,854,775,808
9,223,372,036,854,775,807
0
18,446,744,073,709,551,615

Figure	
2.10	
Typical	ranges	for	C	integral	data	types	for	64-bit
programs.
The	C	standards	define	minimum	ranges	of	values	that	each	data	type
must	be	able	to	represent.	As	shown	in	
Figure	
2.11
,	their	ranges	are
the	same	or	smaller	than	the	typical	implementations	shown	in	
Figures
2.9
	and	
2.10
.	In	particular,	with	the	exception	of	the	fixed-size	data
types,	we	see	that	they	require	only	a
New	to	C?	
Signed	and	unsigned
numbers	in	C,	C++,	and	Java
Both	C	and	C++	support	signed	(the	default)	and	unsigned
numbers.	Java	supports	only	signed	numbers.
C	data	type
Minimum
Maximum
–127
127
0
255
–32,767
32,767
0
65,535
–32,767
32,767
0
65,535
–2,147,483,647
2,147,483,647
0
4,294,967,295

–2,147,483,648
2,147,483,647
0
4,294,967,295
–9,223,372,036,854,775,808
9,223,372,036,854,775,807
0
18,446,744,073,709,551,615
Figure	
2.11	
Guaranteed	ranges	for	C	integral	data	types.
The	C	standards	require	that	the	data	types	have	at	least	these	ranges	of
values.
symmetric	range	of	positive	and	negative	numbers.	We	also	see	that
data	type	
	could	be	implemented	with	2-byte	numbers,	although	this
is	mostly	a	throwback	to	the	days	of	16-bit	machines.	We	also	see	that
size	long	can	be	implemented	with	4-byte	numbers,	and	it	typically	is	for
32-bit	programs.	The	fixed-size	data	types	guarantee	that	the	ranges	of
values	will	be	exactly	those	given	by	the	typical	numbers	of	
Figure
2.9
,	including	the	asymmetry	between	negative	and	positive.
2.2.2	
Unsigned	Encodings
Let	us	consider	an	integer	data	type	of	
w
	bits.	We	write	a	bit	vector	as
either	
,	to	denote	the	entire	vector,	or	as	
to	denote
the	individual	bits	within	the	vector.	Treating	
as	a	number	written	in
binary	notation,	we	obtain	the	
unsigned
	interpretation	of	
.	In	this
encoding,	each	bit	
x
	has	value	0	or	1,	with	the	latter	case	indicating	that
value	2
	should	be	included	as	part	of	the	numeric	value.	We	can	express
this	interpretation	as	a	function	
B2U
	(for	“binary	to	unsigned,”	length	
w
):
x
→
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
x
0
]
	
x
→
	
x
→
i
i
w

Figure	
2.12	
Unsigned	number	examples	for
w
	=	4.	When	bit	
i
	in	the	binary	representation	has	value	1,	it	contributes	2
to	the	value.
Principle:
Definition	of	unsigned	encoding
For	vector	
In	this	equation,	the	notation	
≐
	means	that	the	left-hand	side	is	defined	to
be	equal	to	the	right-hand	side.	The	function	
B2U
	maps	strings	of	zeros
and	ones	of	length	
w
	to	nonnegative	integers.	As	examples,	
Figure
i
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
 
:
B
2
U
w
(
x
→
)
=
˙
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
(2.1)
w

2.12
	shows	the	mapping,	given	by	
B2U
,	from	bit	vectors	to	integers	for
the	following	cases:
In	the	figure,	we	represent	each	bit	position	
i
	by	a	rightward-pointing	blue
bar	of	length	2
.
	The	numeric	value	associated	with	a	bit	vector	then
equals	the	sum	of	the	lengths	of	the	bars	for	which	the	corresponding	bit
values	are	1.
Let	us	consider	the	range	of	values	that	can	be	represented	using	
w
	bits.
The	least	value	is	given	by	bit	vector	[00	...	0]	having	integer	value	0,	and
the	greatest	value	is	given	by	bit	vector	[11	...	1]	having	integer	value
.	Using	the	4-bit	case	as	an	example,	we	have
.	Thus,	the	function	
B2U
	can	be	defined
as	a	mapping	
.
The	unsigned	binary	representation	has	the	important	property	that	every
number	between	0	and	2
	—	1	has	a	unique	encoding	as	a	
w
-bit	value.
For	example,	
there	is	only	one	representation	of	decimal	value	11	as	an
unsigned	4–bit	number—namely,	[1011].	We	highlight	this	as	a
mathematical	principle,	which	we	first	state	and	then	explain.
Principle:
Uniqueness	of	unsigned	encoding
B
2
U
4
(
[
0001
]
)
=
0
⋅
2
3
+
0
⋅
2
2
+
0
⋅
2
1
+
1
⋅
2
0
=
0
+
0
+
0
+
1
=
1
B
2
U
4
(
[
0101
]
)
=
0
⋅
2
3
+
1
⋅
2
2
(2.2)
i
U
M
a
x
w
≐
Σ
i
=
0
w
−
1
2
i
=
2
w
−
1
U
M
a
x
4
=
B
2
U
4
(
[
1111
]
)
=
2
4
−
1
=
15
w
B
2
U
w
:
{
0
,
1
}
w
→
{
0
,
…
,
U
M
a
x
w
}
w

Function	
B2U
	is	a	bijection.
The	mathematical	term	
bijection
	refers	to	a	function	
f
	that	goes	two	ways:
it	maps	a	value	
x
	to	a	value	
y
	where	
y
	=	
f(x)
,	but	it	can	also	operate	in
reverse,	since	for	every	
y
,	there	is	a	unique	value	
x
	such	that	
f(x)
	=	
y.
This	is	given	by	the	
inverse
	function	
f
,	where,	for	our	example,	
x
	=	
f
(
y
).
The	function	
B2U
	maps	each	bit	vector	of	length	
w
	to	a	unique	number
between	0	and	2
	–	1,	and	it	has	an	inverse,	which	we	call	
U2B
	(for
“unsigned	to	binary”),	that	maps	each	number	in	the	range	0	to	2
	–	1	to
a	unique	pattern	of	
w
	bits.
2.2.3	
Two's-Complement	Encodings
For	many	applications,	we	wish	to	represent	negative	values	as	well.	The
most	common	computer	representation	of	signed	numbers	is	known	as
two's-complement
	form.	This	is	defined	by	interpreting	the	most
significant	bit	of	the	word	to	have	negative	weight.	We	express	this
interpretation	as	a	function	
B2T
	(for	“binary	to	two's	complement”	length
w
):
Principle:
Definition	of	two's-complement	encoding
w
−1
−1
w
w
w
w
w

For	vector	
:
The	most	significant	bit	
x
	is	also	called	the	
sign	bit.
	Its	“weight”	is	–2
,
the	negation	of	its	weight	in	an	unsigned	representation.	When	the	sign
bit	is	set	to	1,	the	represented	value	is	negative,	and	when	set	to	0,	the
value	is	nonnegative.	As	examples,	
Figure	
2.13
	shows	the	mapping,
given	by	
B2T
,	from	bit	vectors	to	integers	for	the	following	cases:
In	the	figure,	we	indicate	that	the	sign	bit	has	negative	weight	by	showing
it	as	a	leftward-pointing	gray	bar.	The	numeric	value	associated	with	a	bit
vector	is	then	given	by	the	combination	of	the	possible	leftward-pointing
gray	bar	and	the	rightward-pointing	blue	bars.
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
B
2
T
w
(
x
→
)
=
˙
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
(2.3)
w
–1
w
–1
B
2
T
4
(
[
0001
]
)
=
−
0
⋅
2
3
+
0
⋅
2
2
+
0
⋅
2
1
+
1
⋅
2
0
=
0
+
0
+
0
+
1
=
1
B
2
T
4
(
[
0101
]
)
=
−
0
⋅
2
3
+
1
⋅
2
2
+
0
⋅
2
1
+
1
⋅
2
0
=
0
+
4
+
0
+
1
=
5
B
2
T
4
(
[
1011
]
)
=
−
1
⋅
2
3
+
0
⋅
2
2
+
1
⋅
2
1
+
1
⋅
2
0
=
−
8
+
0
+
2
+
1
=
−
5
B
2
T
4
(
[
1111
]
)
=
−
1
⋅
2
3
+
1
⋅
2
2
+
1
⋅
2
1
+
1
⋅
2
0
=
−
8
+
4
+
2
+
1
=
−
1
(2.4)

Figure	
2.13	
Two's-complement	number	examples	for
w
	=	4.	Bit	3	serves	as	a	sign	bit;	when	set	to	1,	it	contributes	–2
	=	–8	to
the	value.	This	weighting	is	shown	as	a	leftward-pointing	gray	bar.
We	see	that	the	bit	patterns	are	identical	for	
Figures	
2.12
	and	
2.13
(as	well	as	for	
Equations	
2.2
	and	
2.4
),	but	the	values	differ	when
the	most	significant	bit	is	1,	since	in	one	case	it	has	weight	+8,	and	in	the
other	case	it	has	weight	–8.
Let	us	consider	the	range	of	values	that	can	be	represented	as	a	
w
-bit
two's-complement	number.	The	least	representable	value	is	given	by	bit
vector	[10	...	0]	(set	the	bit	with	negative	weight	but	clear	all	others),
having	integer	value	
.	The	greatest	value	is	given	by	bit
vector	[01	...	1]	(clear	the	bit	with	negative	weight	but	set	all	others),
having	integer	value	
.	Using	the	4-bit	case
as	an	example,	we	have	
and
.
We	can	see	that	
B2T
	is	a	mapping	of	bit	patterns	of	length	
w
	to	numbers
3
T
M
i
n
w
=
˙
−
2
w
−
1
T
M
a
x
w
=
˙
∑
i
=
0
w
−
2
2
i
=
2
w
−
1
−
1
T
M
i
n
4
=
B
2
T
4
(
[
1000
]
)
=
−
2
3
=
−
8
	
T
M
a
x
4
=
B
2
T
4
(
[
0111
]
)
=
2
2
+
2
1
+
2
0
=
4
+
2
+
1
=
7
w

between	
TMin
	and	
TMax
,	written	as	
.
As	we	saw	with	the	unsigned	representation,	every	number	within	the
representable	range	has	a	unique	encoding	as	a	
w
-bit	two's-complement
number.	This	leads	to	a	principle	for	two's-complement	numbers	similar
to	that	for	unsigned	numbers:
Principle:
Uniqueness	of	two's-complement	encoding
Function	
B2T
	is	a	bijection.
We	define	function	
T2B
	(for	“two's	complement	to	binary”)	to	be	the
inverse	of	
B2T
.
	That	is,	for	a	number	
x
,	such	that
is	the	(unique)	
w
-bit	pattern	that	encodes	
x
.
Practice	Problem	
2.17
	(solution	page	
148
)
Assuming	
w
	=	4,	we	can	assign	a	numeric	value	to	each	possible
hexadecimal	digit,	assuming	either	an	unsigned	or	a	two's-
complement	interpretation.	Fill	in	the	following	table	according	to
these	interpretations	by	writing	out	the	nonzero	powers	of	2	in	the
summations	shown	in	
Equations	
2.1
	and	
2.3
:
w
w
B
2
T
w
:
{
0
,
1
}
w
→
{
T
M
i
n
w
,
…
,
T
M
a
x
w
}
w
w
w
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
,
 
T
2
B
w
(
x
)
	
x
→

Hexadecimal
Binary
B2U
B2T
[1110]
2
	+	2
	+	2
	=	14
–2
	+	2
	+	2
	=	–2
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
Figure	
2.14
	shows	the	bit	patterns	and	numeric	values	for	several
important	numbers	for	different	word	sizes.	The	first	three	give	the	ranges
of	representable	integers	in	terms	of	the	values	of	
UMax
,	
TMin
,	and
TMax
.	We	will	refer	to	these	three	special	values	often	in	the	ensuing
discussion.	We	will	drop	the	subscript	
w
	and	refer	to	the	values
UMax
,
TMin
,	and	
TMax
	when	
w
	can	be	inferred	from	context	or	is	not	central	to
the	discussion.
A	few	points	are	worth	highlighting	about	these	numbers.	First,	as
observed	in	
Figures	
2.9
	and	
2.10
,	the	two's-complement	range	is
asymmetric:	|
TMin
|	=	|
TMax
|	+	1;	that	is,	there	is	no	positive	counterpart
to	
TMin
.	As	we	shall	see,	this	leads	to	some	peculiar	properties	of	two's-
complement	arithmetic	and	can	be	the	source	of	subtle	program	bugs.
This	asymmetry	arises	because	half	the	bit	patterns	(those	with	the	sign
bit	set	to	1)	represent	negative	numbers,	while	half	(those	with	the	sign
bit	set	to	0)	represent	nonnegative	numbers.	Since	0	is	nonnegative,	this
means	that	it	can	represent	one	less	positive	number	than	negative.
4
(
x
→
)
4
(
x
→
)
3
2
1
3
2
1
w
w
w

Second,	the	maximum	unsigned	value	is	just	over	twice	the	maximum
two's-complement	value:	
UMax
	=	2
TMax
	+	1.	All	of	the	bit	patterns	that
denote	negative	numbers	in	two's-complement	notation	become	positive
values	in	an	unsigned	representation.
Word	size	
w
Value
8
16
32
64
UMax
255
65,535
4,294,967,295
18,446,744,073,709,551,615
TMin
–128
–32,768
–2,147,483,648
–9,223,372,036,854,775,808
TMax
127
32,767
2,147,483,647
9,223,372,036,854,775,807
–1
0
Figure	
2.14	
Important	numbers.
Both	numeric	values	and	hexadecimal	representations	are	shown.
Aside	
More	on	fixed-size	integer	types
For	some	programs,	it	is	essential	that	data	types	be	encoded
using	representations	with	specific	sizes.	For	example,	when
writing	programs	to	enable	a	machine	to	communicate	over	the
w
w
w

Internet	according	to	a	standard	protocol,	it	is	important	to	have
data	types	compatible	with	those	specified	by	the	protocol.	We
have	seen	that	some	C	data	types,	especially	
,	have	different
ranges	on	different	machines,	and	in	fact	the	C	standards	only
specify	the	minimum	ranges	for	any	data	type,	not	the	exact
ranges.	Although	we	can	choose	data	types	that	will	be
compatible	with	standard	representations	on	most	machines,
there	is	no	guarantee	of	portability.
We	have	already	encountered	the	32-	and	64-bit	versions	of	fixed-
size	integer	types	(
Figure	
2.3
);	they	are	part	of	a	larger	class	of
data	types.	The	ISO	C99	standard	introduces	this	class	of	integer
types	in	the	file	
.	This	file	defines	a	set	of	data	types	with
declarations	of	the	form	
	and	
,	specifying	
N
-bit
signed	and	unsigned	integers,	for	different	values	of	
N
.	The	exact
values	of	
N
	are	implementation	dependent,	but	most	compilers
allow	values	of	8,	16,	32,	and	64.	Thus,	we	can	unambiguously
declare	an	unsigned	16–bit	variable	by	giving	it	type	
,	and
a	signed	variable	of	32	bits	as	
.
Along	with	these	data	types	are	a	set	of	macros	defining	the
minimum	and	maximum	values	for	each	value	of	
N
.	These	have
names	of	the	form	
,	and	
.
Formatted	printing	with	fixed-width	types	requires	use	of	macros
that	expand	into	format	strings	in	a	system-dependent	manner.
So,	for	example,	the	values	of	variables	
x
	and	
y
	of	type	
and	
	can	be	printed	by	the	following	call	to	
:

When	compiled	as	a	64–bit	program,	macro	
	expands	to
the	string	
,	while	
	expands	to	the	pair	of	strings	
.
When	the	C	preprocessor	encounters	a	sequence	of	string
constants	separated	only	by	spaces	(or	other	whitespace
characters),	it	concatenates	them	together.	Thus,	the	above	call	to
	becomes
Using	the	macros	ensures	that	a	correct	format	string	will	be
generated	regardless	of	how	the	code	is	compiled.
Figure	
2.14
	also	shows	the	representations	of	constants	–1	and	0.
Note	that	–1	has	the	same	bit	representation	as	
UMax
—a	string	of	all
ones.	Numeric	value	0	is	represented	as	a	string	of	all	zeros	in	both
representations.
The	C	standards	do	not	require	signed	integers	to	be	represented	in
two's-complement	form,	but	nearly	all	machines	do	so.	Programmers
who	are	concerned	with	maximizing	portability	across	all	possible
machines	should	not	assume	any	particular	range	of	representable
values,	beyond	the	ranges	indicated	in	
Figure	
2.11
,	nor	should	they
assume	any	particular	representation	of	signed	numbers.	On	the	other
hand,	many	programs	are	written	assuming	a	two's-complement
representation	of	signed	numbers,	and	the	“typical”	ranges	shown	in
Figures	
2.9
	and	
2.10
,	and	these	programs	are	portable	across	a

broad	range	of	machines	and	compilers.	The	file	
	in	the	C
library	defines	a	set	of	constants
Aside	
Alternative	representations	of
signed	numbers
There	are	two	other	standard	representations	for	signed	numbers:
Ones’	complement.
	This	is	the	same	as	two's	complement,
except	that	the	most	significant	bit	has	weight	–(2
	–	1)	rather
than	–2
:
Sign-magnitude.
	The	most	significant	bit	is	a	sign	bit	that
determines	whether	the	remaining	bits	should	be	given	negative
or	positive	weight:
Both	of	these	representations	have	the	curious	property	that	there
are	two	different	encodings	of	the	number	0.	For	both
representations,	[00	...	0]	is	interpreted	as	+0.	The	value	–0	can
be	represented	in	sign-magnitude	form	as	[10	...	0]	and	in	ones’
complement	as	[11	...	1].	Although	machines	based	on	ones'-
complement	representations	were	built	in	the	past,	almost	all
modern	machines	use	two's	complement.	We	will	see	that	sign-
magnitude	encoding	is	used	with	floating-point	numbers.
w–1
w
–1
B
2
O
w
(
x
→
)
=
˙
−
x
w
−
1
(
2
w
−
1
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
B
2
S
w
(
x
→
)
=
˙
(
−
1
)
x
w
−
1.
(
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
1
)

Note	the	different	position	of	apostrophes:	
two's
	complement
versus	
ones'
	complement.	The	term	“two's	complement”	arises
from	the	fact	that	for	nonnegative	
x
	we	compute	a	
w
-bit
representation	of	–
x
	as	2
	–	
x
	(a	single	two.)	The	term	“ones’
complement”	comes	from	the	property	that	we	can	compute	–
x
	in
this	notation	as	[111	...	1]	–	
x
	(multiple	ones).
delimiting	the	ranges	of	the	different	integer	data	types	for	the	particular
machine	on	which	the	compiler	is	running.	For	example,	it	defines
constants	
,	and	
	describing	the	ranges	of
signed	and	unsigned	integers.	For	a	two's-complement	machine	in	which
data	type	
	has	
w
	bits,	these	constants	correspond	to	the	values	of
TMax
,	TMin
,	and	
UMax
.
The	Java	standard	is	quite	specific	about	integer	data	type	ranges	and
representations.	It	requires	a	two's-complement	representation	with	the
exact	ranges	shown	for	the	64-bit	case	(
Figure	
2.10
).	In	Java,	the
single-byte	data	type	is	called	
	instead	of	
.	These	detailed
requirements	are	intended	to	enable	Java	programs	to	behave	identically
regardless	of	the	machines	or	operating	systems	running	them.
To	get	a	better	understanding	of	the	two's-complement	representation,
consider	the	following	code	example:
w
w
w
w

12,345
–12,345
53,191
Weight
Bit
Value
Bit
Value
Bit
Value
1
1
1
1
2
0
2
2
4
0
4
4
8
8
0
0
16
16
0
0
32
32
0
0
64
0
64
64
128
0
128
128
256
0
256
256
512
0
512
512
1,024
0
1,024
1,024
2,048
0
2,048
2,048
4,096
4,096
0
0
8,192
8,192
0
0
16,384
0
16,384
16,384

±32,768
0
–32,768
32,768
Total
12,345
–12,345
53,191
Figure	
2.15	
Two's-complement	representations	of	12,345	and	–
12,345,	and	unsigned	representation	of	53,191.
Note	that	the	latter	two	have	identical	bit	representations.
When	run	on	a	big-endian	machine,	this	code	prints	
	and	
,
indicating	that	
	has	hexadecimal	representation	
,	while	
	has
hexadecimal	representation	
.	Expanding	these	into	binary,	we	get
bit	patterns	[0011000000111001]	for	
	and	[1100111111000111]	for	
.
As	
Figure	
2.15
	shows,	
Equation	
2.3
	yields	values	12,345	and	–
12,345	for	these	two	bit	patterns.
Practice	Problem	
2.18
	(solution	page	
149
)
In	
Chapter	
3
,	we	will	look	at	listings	generated	by	a
disassembler
,	a	program	that	converts	an	executable	program	file
back	to	a	more	readable	ASCII	form.	These	files	contain	many
hexadecimal	numbers,	typically	representing	values	in	two's-
complement	form.	Being	able	to	recognize	these	numbers	and
understand	their	significance	(for	example,	whether	they	are
negative	or	positive)	is	an	important	skill.
For	the	lines	labeled	A–I	(on	the	right)	in	the	following	listing,
convert	the	hexadecimal	values	(in	32-bit	two's-complement	form)
shown	to	the	right	of	the	instruction	names	(
	and	
)
into	their	decimal	equivalents:

2.2.4	
Conversions	between	Signed
and	Unsigned

C	allows	casting	between	different	numeric	data	types.	For	example,
suppose	variable	
	is	declared	as	
	and	
	as	unsigned.	The
expression	
	converts	the	value	of	
	to	an	unsigned	value,
and	
	converts	the	value	of	
	to	a	signed	integer.	What	should	be
the	effect	of	casting	signed	value	to	unsigned,	or	vice	versa?	From	a
mathematical	perspective,	one	can	imagine	several	different	conventions.
Clearly,	we	want	to	preserve	any	value	that	can	be	represented	in	both
forms.	On	the	other	hand,	converting	a	negative	value	to	unsigned	might
yield	zero.	Converting	an	unsigned	value	that	is	too	large	to	be
represented	in	two's-complement	form	might	yield	
TMax.
	For	most
implementations	of	C,	however,	the	answer	to	this	question	is	based	on	a
bit-level	perspective,	rather	than	on	a	numeric	one.
For	example,	consider	the	following	code:
When	run	on	a	two's-complement	machine,	it	generates	the	following
output:

What	we	see	here	is	that	the	effect	of	casting	is	to	keep	the	bit	values
identical	but	change	how	these	bits	are	interpreted.	We	saw	in	
Figure
2.15
	that	the	16-bit	two's-complement	representation	of	–12,345	is
identical	to	the	16-bit	unsigned	representation	of	53,191.	Casting	from
	to	
	changed	the	numeric	value,	but	not	the	bit
representation.
Similarly,	consider	the	following	code:
When	run	on	a	two's-complement	machine,	it	generates	the	following
output:
We	can	see	from	
Figure	
2.14
	that,	for	a	32-bit	word	size,	the	bit
patterns	representing	4,294,967,295	(
UMax
)	in	unsigned	form	and	–1	in
two's-complement	form	are	identical.	In	casting	from	
	to	
,	the
underlying	bit	representation	stays	the	same.
This	is	a	general	rule	for	how	most	C	implementations	handle
conversions	between	signed	and	unsigned	numbers	with	the	same	word
32

size—the	numeric	values	might	change,	but	the	bit	patterns	do	not.	Let
us	capture	this	idea	in	a	more	mathematical	form.	We	defined	functions
U2B
	and	
T2B
	that	map	numbers	to	their	bit	representations	in	either
unsigned	or	two's-complement	form.	That	is,	given	an	integer	
x
	in	the
range	
,	the	function	
U2B
(x)
	gives	the	unique	
w
-bit	unsigned
representation	of	
x
.	Similarly,	when	
x
	is	in	the	range	
,
the	function	
T2B
(x)
	gives	the	unique	
w
-bit	two's-complement
representation	of	
x
.
Now	define	the	function	
.	This	function
takes	a	number	between	
TMin
	and	
TMax
	and	yields	a	number	between
0	and	
UMax
,	where	the	two	numbers	have	identical	bit	representations,
except	that	the	argument	has	a	two's-complement	representation	while
the	result	is	unsigned.	Similarly,	for	
x
	between	0	and	
UMax
,	the	function
U2T
,	defined	as	
,	yields	the	number	having	the
same	two's-complement	representation	as	the	unsigned	representation
of	
x
.
Pursuing	our	earlier	examples,	we	see	from	
Figure	
2.15
	that	
T2U
(–
12,345)	=	53,191,	and	that	
U2T
(53,191)	=	–12,345.	That	is,	the	16-bit
pattern	written	in	hexadecimal	as	
	is	both	the	two's-complement
representation	of	–12,345	and	the	unsigned	representation	of	53,191.
Note	also	that	12,345	+	53,191	=	65,536	=	2
.	This	property	generalizes
to	a	relationship	between	the	two	numeric	values	(two's	complement	and
unsigned)	represented	by	a	given	bit	pattern.	Similarly,	from	
Figure
2.14
,	we	see	that	
T2U32
(–1)	=	4,294,967,295,	and
U2T
(4,294,967,295)	=	–1.	That	is,	
UMax
	has	the	same	bit
representation	in	unsigned	form	as	does	–1	in	two's-complement	form.
w
w
0
≤
x
<
U
M
a
x
w
w
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
w
T
2
U
w
 
as
 
T
2
U
w
(
x
)
≐
B
2
U
w
(
T
2
B
w
(
x
)
)
w
w
w
w
w
U
2
T
w
=
˙
B
2
T
w
(
U
2
B
w
(
x
)
)
16
16
16
32

We	can	also	see	the	relationship	between	these	two	numbers:	1	+	
UMax
=	2
.
We	see,	then,	that	function	
T2U
	describes	the	conversion	of	a
two'scomplement	number	to	its	unsigned	counterpart,	while	
U2T
	converts
in	the	opposite	direction.	These	describe	the	effect	of	casting	between
these	data	types	in	most	C	implementations.
Practice	Problem	
2.19
	(solution	page	
149
)
Using	the	table	you	filled	in	when	solving	
Problem	
2.17
,	fill	in
the	following	table	describing	the	function	
T2U
:
x
T2U
(x)
–8
__________
–3
__________
–2
__________
–1
__________
0
__________
5
__________
The	relationship	we	have	seen,	via	several	examples,	between	the	two's-
complement	and	unsigned	values	for	a	given	bit	pattern	can	be
expressed	as	a	property	of	the	function	
T2U
:
w
w
4
4

Principle:
Conversion	from	two's	complement	to	unsigned
For	
x
	such	that	
:
For	example,	we	saw	that	
,	and
also	that	
.
This	property	can	be	derived	by	comparing	
Equations	
2.1
	and	
2.3
.
Derivation:
Conversion	from	two's	complement	to	unsigned
Comparing	
Equations	
2.1
	and	
2.3
,	we	can	see	that
for	bit	pattern	
,	if	we	compute	the	difference	
,	the	weighted	sums	for	bits	from	0	to	
w
	–2	will
cancel	each	other,	leaving	a	value	
.	This	gives	a
relationship	
.	We
therefore	have
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
T
2
U
w
(
x
)
=
{
x
+
2
w
,
x
<
0
x
,
x
≥
0
(2.5)
T
2
U
16
(
−
12
,
345
)
=
−
12
,
345
+
2
16
=
53
,
191
T
2
U
w
(
−
1
)
=
−
1
+
2
w
=
U
M
a
x
w
x
→
B
2
U
w
(
x
→
)
−
B
2
T
w
(
x
→
)
B
2
U
w
	
(
	
x
→
	
)
−
B
2
T
w
	
(
x
→
	
)
=
x
	
w
−
1
	
(
	
2
	
w
−
1
	
−
−
2
	
w
−
1
	
)
=
x
	
w
−
1
	
2
w
	
B
2
U
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
	
(
	
x
→
	
)
+
x
	
w
−
1
	
2
w
	

In	a	two's-complement	representation	of	
x
,	bit	
x
determines	whether	or	not	
x
	is	negative,	giving	the	two
cases	of	
Equation	
2.5
.
As	examples,	
Figure	
2.16
	compares	how	functions	
B2U
	and	
B2T
assign	values	to	bit	patterns	for	
w
	=	4.	For	the	two's-complement	case,
the	most	significant	bit	serves	as	the	sign	bit,	which	we	diagram	as	a
leftward-pointing	gray	bar.	For	the	unsigned	case,	this	bit	has	positive
weight,	which	we	show	as	a	rightward-pointing	black	bar.	In	going	from
two's	complement	to	unsigned,	the	most	significant	bit	changes	its	weight
from	–8	to	+8.	As	a	consequence,	the	values	that	are	negative	in	a	two's-
complement	representation	increase	by	2
	=	16	with	an	unsigned
representation.	Thus,	–5	becomes	+11,	and	–1	becomes	+15.
B
2
U
w
(
T
2
B
w
(
x
)
)
=
T
2
U
w
(
x
)
=
x
+
x
w
−
1
2
w
(2.6)
w
–1
4

Figure	
2.16	
Comparing	unsigned	and	two's-complement
representations	for
w
	=	4.	The	weight	of	the	most	significant	bit	is	–8	for	two's	complement
and	+8	for	unsigned,	yielding	a	net	difference	of	16.
Figure	
2.17	
Conversion	from	two's	complement	to	unsigned.
Function	
T2U
	converts	negative	numbers	to	large	positive	numbers.
Figure	
2.17
	illustrates	the	general	behavior	of	function	
T2U
.	As	it
shows,	when	mapping	a	signed	number	to	its	unsigned	counterpart,
negative	numbers	are	converted	to	large	positive	numbers,	while
nonnegative	numbers	remain	unchanged.
Practice	Problem	
2.20
	(solution	page	
149
)
Explain	how	
Equation	
2.5
	applies	to	the	entries	in	the	table	you
generated	when	solving	
Problem	
2.19
.
Going	in	the	other	direction,	we	can	state	the	relationship	between	an
unsigned	number	
u
	and	its	signed	counterpart	
U2T
(u)
:
w

Principle:
Unsigned	to	two's-complement	conversion
For	
u
	such	that	0	≤	
u
	≤	
UMax
:
Figure	
2.18	
Conversion	from	unsigned	to	two's	complement.
Function	
U2T
	converts	numbers	greater	than	
to
negative	values.
This	principle	can	be	justified	as	follows:
Derivation:
w
U
2
T
w
(
u
)
=
{
u
,
u
≥
T
M
a
x
w
u
−
2
w
,
u
>
T
M
a
x
w
(2.7)
T
M
a
x
w
=
2
w
−
1
−
1
	

Unsigned	to	two's-complement	conversion
Let	
.	This	bit	vector	will	also	be	the	two's-
complement	representation	of	
U2T
(u)
.	
Equations	
2.1
and	
2.3
	can	be	combined	to	give
In	the	unsigned	representation	of	
u
,	bit	
u
	determines
whether	or	not	
u
	is	greater	than	
TMax
	=	2
	–	1,	giving	the
two	cases	of	
Equation	
2.7
.
The	behavior	of	function	
U2T
	is	illustrated	in	
Figure	
2.18
.	For	small	(≤
TMax
)	numbers,	the	conversion	from	unsigned	to	signed	preserves	the
nu-meric	value.	Large	(>	
TMax
)	numbers	are	converted	to	negative
values.
To	summarize,	we	considered	the	effects	of	converting	in	both	directions
between	unsigned	and	two's-complement	representations.	For	values	
x
in	the	range	
,	we	have	
and	
.	That	is,
numbers	in	this	range	have	identical	unsigned	and	two's-complement
representations.	For	values	outside	of	this	range,	the	conversions	either
add	or	subtract	2
.	For	example,	we	have	
—
the	negative	number	closest	to	zero	maps	to	the	largest	unsigned
number.	At	the	other	extreme,	one	can	see	that	
—the	most	negative	number	maps	to	an
unsigned	number	just	outside	the	range	of	positive	two's-complement
u
→
=
U
2
B
w
(
u
)
w
U
2
T
w
(
u
)
=
−
u
w
−
1
2
w
+
u
(2.8)
w
–1
w
w
–1
w
w
0
≤
x
≤
T
M
a
x
w
T
2
U
w
(
x
)
=
x
	
U
2
T
w
(
x
)
=
x
w
T
2
U
w
(
−
1
)
=
−
1
+
2
w
=
U
M
a
x
w
T
2
U
w
(
T
M
i
n
w
)
=
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
=
T
M
a
x
w
+
1

numbers.	Using	the	example	of	
Figure	
2.15
,	we	can	see	that
.
2.2.5	
Signed	versus	Unsigned	in	C
As	indicated	in	
Figures	
2.9
	and	
2.10
,	C	supports	both	signed	and
unsigned	arithmetic	for	all	of	its	integer	data	types.	Although	the	C
standard	does	not	specify	a	particular	representation	of	signed	numbers,
almost	all	machines	use	two's	complement.	Generally,	most	numbers	are
signed	by	default.	For	example,	when	declaring	a	constant	such	as	
or	
,	the	value	is	considered	signed.	Adding	character	
	or	
as	a	suffix	creates	an	unsigned	constant;	for	example,	
	or	
.
C	allows	conversion	between	unsigned	and	signed.	Although	the	C
standard	does	not	specify	precisely	how	this	conversion	should	be	made,
most	systems	follow	the	rule	that	the	underlying	bit	representation	does
not	change.	This	rule	has	the	effect	of	applying	the	function	
U2T
	when
converting	from	unsigned	to	signed,	and	
T2U
	when	converting	from
signed	to	unsigned,	where	
w
	is	the	number	of	bits	for	the	data	type.
Conversions	can	happen	due	to	explicit	casting,	such	as	in	the	following
code:
T
2
U
16
(
−
12
,
345
)
=
65
,
536
+
−
12
,
345
=
53
,
191
w
w

Alternatively,	they	can	happen	implicitly	when	an	expression	of	one	type
is	assigned	to	a	variable	of	another,	as	in	the	following	code:
When	printing	numeric	values	with	printf,	the	directives	
	and	
are	used	to	print	a	number	as	a	signed	decimal,	an	unsigned	decimal,
and	in	hexadecimal	format,	respectively.	Note	that	
	does	not	make
use	of	any	type	information,	and	so	it	is	possible	to	print	a	value	of	type
	with	directive	
	and	a	value	of	type	
	with	directive	
.	For
example,	consider	the	following	code:

When	compiled	as	a	32-bit	program,	it	prints	the	following:
In	both	cases,	
	prints	the	word	first	as	if	it	represented	an	unsigned
number	and	second	as	if	it	represented	a	signed	number.	We	can	see	the
conversion	routines	in	action:	
and
.
Some	possibly	nonintuitive	behavior	arises	due	to	C's	handling	of
expressions	containing	combinations	of	signed	and	unsigned	quantities.
When	an	operation	is	performed	where	one	operand	is	signed	and	the
other	is	unsigned,	C	implicitly	casts	the	signed	argument	to	unsigned	and
performs	the	operations
Expression
Type
Evaluation
T
2
U
32
(
−
1
)
=
U
M
a
x
32
=
2
32
−
1
	
U
2
T
32
(
2
31
)
=
2
31
−
2
32
=
−
2
31
=
T
M
i
n
32

Figure	
2.19	
Effects	of	C	promotion	rules.
Nonintuitive	cases	are	marked	by	‘
’.	When	either	operand	of	a
comparison	is	unsigned,	the	other	operand	is	implicitly	cast	to	unsigned.
See	Web	Aside	
DATA
:
TMIN
	
for	why	we	write	
TMin
	as	
.
assuming	the	numbers	are	nonnegative.	As	we	will	see,	this	convention
makes	little	difference	for	standard	arithmetic	operations,	but	it	leads	to
nonintuitive	results	for	relational	operators	such	as	<	and	>.	
Figure
2.19
	shows	some	sample	relational	expressions	and	their	resulting
evaluations,	when	data	type	
	has	a	32-bit,	two's-complement
representation.	Consider	the	comparison	
.	Since	the	second
operand	is	unsigned,	the	first	one	is	implicitly	cast	to	unsigned,	and
hence	the	expression	is	equivalent	to	the	comparison	
(recall	that	
),	which	of	course	is	false.	The	other	cases
can	be	understood	by	similar	analyses.
Practice	Problem	
2.21
	(solution	page	
149
)
Assuming	the	expressions	are	evaluated	when	executing	a	32-bit
program	on	a	machine	that	uses	two's-complement	arithmetic,	fill
in	the	following	table	describing	the	effect	of	casting	and	relational
operations,	in	the	style	of	
Figure	
2.19
:
Expression
Type
Evaluation
–2147483647–1	==	2147483648U
__________
__________
32
T
2
U
w
(
−
1
)
=
U
M
a
x
w

–2147483647–1	<	2147483647
__________
__________
–2147483647–1U	<	2147483647
__________
__________
–2147483647–1	<	–2147483647
__________
__________
–2147483647–1U	<	–2147483647
_________
__________
2.2.6	
Expanding	the	Bit
Representation	of	a	Number
One	common	operation	is	to	convert	between	integers	having	different
word	sizes	while	retaining	the	same	numeric	value.	Of	course,	this	may
not	be	possible	when	the	destination	data	type	is	too	small	to	represent
the	desired	value.	Converting	from	a	smaller	to	a	larger	data	type,
however,	should	always	be	possible.
Web	Aside	DATA:TMIN	
Writing	
TMin
	in
C
In	
Figure	
2.19
	and	in	
Problem	
2.21
,	we	carefully	wrote	the
value	of	
TMin
	as	
.	Why	not	simply	write	it	as
either	
	or	
	Looking	at	the	C	header	file
,	we	see	that	they	use	a	similar	method	as	we	have	to
write	
TMin
	and	
TMax
:
32
32
32

Unfortunately,	a	curious	interaction	between	the	asymmetry	of	the
two's-complement	representation	and	the	conversion	rules	of	C
forces	us	to	write	
TMin
	in	this	unusual	way.	Although
understanding	this	issue	requires	us	to	delve	into	one	of	the
murkier	corners	of	the	C	language	standards,	it	will	help	us
appreciate	some	of	the	subtleties	of	integer	data	types	and