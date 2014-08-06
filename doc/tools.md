# Tools

## Compilation

For sambamba ldc (the D llvm compiler) is used for development and for
distribution. See

  https://github.com/ldc-developers/ldc

## Command line build with rdmd

D comes with some powerful build tools. For example to build main.d with all its
dependencies

  rdmd --build-only -IBioD/ -ofbuild/sambamba main.d

To view a BAM file

   rdmd -IBioD/ -ofbuild/sambamba main.d view ./BioD/test/data/bins.bam

## Using tags in emacs/vim

It may be useful to generate a tag file - for navigation in vim and emacs:

  rdmd -c -Xftags.json --build-only -IBioD/ -ofbuild/sambamba main.d

and run d2tags.d - a recent version may be in the
[sambamba-dev-docs](https://github.com/lomereiter/sambamba-dev-docs)
repository:

  cd sambamba
  rdmd ../sambamba-dev-docs/scripts/d2tags . > tags


