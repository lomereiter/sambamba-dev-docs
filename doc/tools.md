# Tools

For sambamba ldc is used for development and for distribution. See

  https://github.com/ldc-developers/ldc

D comes with some powerful build tools. For example to build main.d with all its
dependencies

  rdmd --build-only -IBioD/ -ofbuild/sambamba main.d

To view a BAM file

   rdmd -IBioD/ -ofbuild/sambamba main.d view ./BioD/test/data/bins.bam

It may be useful to generate a tag file - for navigation in vim and emacs:

  rdmd -c -Xftags.json --build-only -IBioD/ -ofbuild/sambamba main.d

and run d2tags.d - a recent version in the qtlHD repository:

  rdmd ../qtlHD/scripts/d2tags . > tags


