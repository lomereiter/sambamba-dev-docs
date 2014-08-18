# Tools

## Compilation

For sambamba ldc (the D llvm compiler) is used for development and for
distribution. See

  https://github.com/ldc-developers/ldc

ldc comes with all major Linux distributions. It may be an idea, however, to use one
of the latest releases for development.

## Makefile

Build all

    make 

Build one tool

    make sambamba-depth

See the Makefile for details.

## Command line build with rdmd

D comes with some powerful build tools. For example to build main.d with all its
dependencies

    rdmd --build-only -IBioD/ -ofbuild/sambamba main.d

To view a BAM file

    rdmd -IBioD/ -ofbuild/sambamba main.d view ./BioD/test/data/bins.bam

## Debugging

Compile using the -g switch, e.g.,

    rdmd -g --build-only -IBioD/ -ofbuild/sambamba main.d

Now you should be able to use ./build/sambamba with gdb or lldb

    gdb build/sambamba pileup

A quick tour would be to set a breakpoint at main

    (gdb) b main
    Breakpoint 1 at 0xd38684

run up to the breakpoint and list the lines

    (gdb) r
    Breakpoint 1, 0x0000000000d38684 in main ()
    (gdb) l 48
    48      int main(string[] args) {
    49          if (args.length == 1) {
    50              printUsage();
    51              return 1;
    52          }
    (gdb) b 49
    (gdb) r

and print some values

    (gdb) p args
    $1 = {"/export/local/users/wrk/izip/git/opensource/D/sambamba/build/sambamba"}
    (gdb) p args.length
    $2 = 1

## Using tags in emacs/vim

It may be useful to generate a tag file - for navigation in vim and emacs:

  rdmd -c -Xftags.json --build-only -IBioD/ -ofbuild/sambamba main.d

and run d2tags.d - a recent version may be in the
[sambamba-dev-docs](https://github.com/lomereiter/sambamba-dev-docs)
repository:

  cd sambamba
  rdmd ../sambamba-dev-docs/scripts/d2tags . > tags


