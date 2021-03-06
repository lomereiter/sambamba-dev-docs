# Sambamba programmer manual 

Sambamba is for programmers. While sambamba allows rapid access to SAM and BAM
data (and soon CRAM), leveraging the cores on a computer, sambamba has more to
offer than that.  Sambamba comes in the form of accessible source code that
allows programmers to add more functionality. This manual is meant to help
software developers to achieve their goals.

Sambamba is (mostly) written in the D progamming language. D is a modern
compiled, strictly typed, hybrid OOP-functional language, that means it
supports both the OOP and FP paradigms to a large degree. In addition D is
fast, almost as fast as C. Hopefully Sambamba will also let you appreciate the
power of D.

One thing to appreciate is that Sambamba is built around multiple worker
threads, using (lazy) functions, when a file is read, it is delivered in chunks
to worker threads. D helps to abstract away most of the plumbing. Some typical
approach may look like:

```D
        void progress(lazy float p) {
            static uint n;
            if (++n % 63 == 0) writeln(p); // prints progress after every 63 records
        }
        ...
        foreach (read; bam.readsWithProgress(toDelegate(&progress))) {
            ...
        }
```

where the foreach loop is executed in parallel and progress() is passed in as a
lazy function. The progress function is made lazy so the user can control the
amount of times progress is calculated, which may be expensive. Note that the
static declaration makes n shared between threads. More information on D thread
pooling can be found in the D module
[std.parallelism](http://dlang.org/phobos/std_parallelism.html).

# main.d

Sambamba has modules for indexing, sorting, markdup, coverage etc. The command
line interface (CLI) starts from main.d and multiplexes into, for example,
markdup.d. Here we visit markdup because it goes through the BAM/SAM data read
by read. Next we go through the data by visiting the sambamba view
functionality.

# markup.d

The first task of the function markdup_main is to parse the command
line. Next a TaskPool is set up for multi-core processing. The main program
sizes the Hash table and sets up the BAM reader, which uses the thread
taskpool. BamReader is defined in BioD/bio/bam/reader.d, which is part of the
BioD project repository (git pulls in that repo when you clone sambamba with
submodules). SamReader and BamReader share a common interface for access,
defined by IBamSamReader. BamReader handles parallel decompression of BGZF
blocks, see below for more.

In the next step all (bgzf) offsets for the reads are gathered and stored in a
list/array of VirtualOffset type using the function getDuplicateOffsets().

Note that this function uses malloc/free directly to allocate memory on the
heap.  D allows you to do that in addition to using the (standard) way of
implicit allocation (JAVA style) using the garbage collector. Another point of
interest it the compression of data in tight data structures, such as with
SingleEndBasicInfo which counts 8 bytes.

The function readPairsAndFragments() walks all reads using the thread pool(!)
and returns results, one at a time, in the variable pf and look at
the read positions. The function computeFivePrimeCoord() displays some of the
D functional programming patterns used in sambamba, e.g.,

```D
  auto clipped = reduce!q{ a + b }(0, ops.map!q{ a.length });
```

First 'auto' can autodetect the type - which is an integer here (ultimately
derived from read.length).  Map and reduce are template functons which take
anonymous functions containing 'a+b' and 'a.length' respectively. This
one-liner walks the reads, returns the length of each and calculates the total
length. The final line
  
```D
        return read.position + read.basesCovered() + clipped;
```

returns the 5 prime position calculated from the read position.

All read positions for the full set are stored in RAM. Next they are sorted
using the fast multi-threaded (in-place) 'unstableSort' a mixed algorithm of
quick sort and heap sort algorithms. From the sorted data we can pull the
matching read positions and mark/remove the duplicate reads, which are used
to generate the output file.

# BioD/bio/bam/reader.d

Interesting methods are header() which returns the SAM header in the BAM file;
getBgzfBlockAt(), which get a BGZF block at a given file offset;
reference_sequences() and reference() get information on reference sequences;
reads() and readsWithProgress() fetch reads; getReadAt(); getReadsBetween();
unmappedReads(); all read related.

In a way the BamReader class is one of the more complex ones as it supports
multi-threading, multiple policies and multiple ways of accessing data
(seekable with index, sequential, compressed, uncompressed etc.). Much of this
complexity is handled by the underlying classes. The SamReader is shorter (in
lines of code) and perhaps easier to study first.

# BioD/bio/sam/reader.d

To parse files sambamba uses the ragel lexical parser which generates
remarkably efficient parser code in many languages, including D. Not
only does ragel give us speed, it also throws in syntactical checking and
error messages. Quite a large step from ad hoc hand-written textual 
parsers so common in bioinformatics. Have a look at Ragel also when you 
write parsers for other programming languages! For the SAM parser
look in BioD/src_ragel/sam_alignment.rl, which is 400 lines of Ragel
mixed with D statements that are used to generate the parser and unit 
tests.

With sambamba Ragel is also used for parsing command line arguments and the
filtering language which includes awesome support for regular expressions(!)
In all, the combination of D and Ragel makes sambamba a powerful and
maintainable software product which can be built up on.

# view.d

The next one to study is the view routine which parses SAM/BAM and 
outputs the contents in a human readable form. Again the BamReader/SamReader
is used. 

The first step is to read the unittests in BioD/src_ragel/sam_alignment.rl.
These show how a SAM record is unpacked.

The actual work happens in processAlignments() which takes an output
method as an argument (SAM, BAM/MsgPack or JSON). Check out SamSerializer
next:

# sambamba/utils/view/alignmentrangeprocessor.d

The sequence writer will write sequences using a thread pool (again).

# BioD/bio/sam/writer.d

The SamSerializer.process() function processes reads in parallel, gathers
output in chunks in parallel and outputs a set of chunks. To write to output it
locks the file. Impressively in all the sambamba and BioD fine grained parallel
code, locking is called only 5 times!

# slice.d

Slicing is one of the simplest manipulations, even if deciding what belongs in
a slice may be a bit hairy. First fetchUnmapped() rewrites all unmapped reads
to a BAM file. Likewise fetchRegion() a slice will write out all reads in a
region.

# depth.d

Under construction

# mpileup.d


