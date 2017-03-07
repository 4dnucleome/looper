# looper
Free energy based high-resolution modeling of CTCF-mediated chromatin loops for human genome

# current packages: chreval

### What does this package do? ###
This package contains an object oriented programs for calculating 
meta-structures based on the observed frequency of contacts 
obtained from experimental data and a model for polymer entropy
known as cross-linking entropy (CLE). Cross link is used in a 
generic sense to mean any contact caused by van der Waals 
interactions or chemical bonds. It is based on integration of
the probability density function for end-to-end distance; 
typically introduced as the Gaussian polymer chain model in 
polymer textbooks.

The primary target experimental data for this package is 
ChIA-PET data; however, the package also has an option for 
analysing Hi-C data. Because the there are more issues with 
self-ligation products in a general all-in-all HiC experiment, 
it is strongly advised to stick to ChIA-PET data in general,
because this method targets immunopreciptation of specific 
protein-protein and protein-chromatin products.

The calculation scheme uses the dynamic programming algorithm
to compute the free energy based on the observed frequency of 
contacts, or a weighted heat map constructed from epigenetic 
data -- be one proposed or a scheme developed. Code for building
heatmaps is provided. 

+ We urge users to be sensible about what they try to calculate. 

This is _not_ meant to calculate whole genomes on a single input, 
though one program in this contributed package can calculate a 
collection of loops from various parts of the genome. this is a 
grid based program, and operates at O(N^3) time, with similar 
demands on memory. Therefore, it _can_ blow up.

+ This is _not_ a black box

Science is about learning to think and learning how to ask the 
right questions. Please learn to use your mind and think about 
what you are doing.


### What does this package contain? ###

This repository contains the following executable programs.


* chreval.py (CHRomatin EVALuation program for heatmaps)

_Chreval_ is the main driver for calculating the free energy of
observed heatmaps.  This is the actually dynamic programming 
algorithm part of the package. It calculates the optimal and 
suboptmal structures of chromatin the Boltzmann distribution of 
the principal structures. The program analyzes the energy of 
various structures and defines them in terms of structural motifs. 
These basic motifs can then be used to build up a whole complex 
chromatin structure.

* anal_loops.py

_Anal_loops_ is for handling a large collection of heatmap files 
in an orderly fashion. The format of the input file is according
to Przemek's bed format. This is a driver for carrying out analysis 
of the bed files with corresponding heat maps. See the included 
directory in this distribution

tests/test_anal_loops 

for an example. The program calls objects in _Chreval_. You should make 
sure that the files listed in the bed file also exist in your directory.

* make_heatmap.py

_Make_heatmap_ is a tool to generate a visualization of the 2D heatmaps 
either from "*.heat" files, extended heatmap files "*.eheat", or the output 
files "*.clust" generated by _Chreval_.

* my_generation.py

_My_generation_ is a program to actually generate heatmaps that can be 
calculated using _Chreval_. Entries must be given in the Fontana dot 
bracket format. This can use extended notation like "ABC..abc" etc. 
For the file input, the user can specify both the specific contacts
and the desired weights. For more inforation, please see an example 
provided by typing the command line with the flag -hExFile. 


* SimRNA_make_polyA.py and SimRNA2mds.py

_SimRNA_make_polyA_ is used to generate a polyA sequence of a specified 
length. This program generates a poly(A) sequence of a specified length 
and is intended as an aid in generating a 3D ensemble of the 2D meta-structures 
generated by _Chreval_ using the a given output restraint file *.simres.  

_SimRNA2mds_ converts the SimRNA3.21 generated pdb output files into a single
bead model (for building 3D representations of chromatin .... presently, the 
coordinates are _not_ rescaled).
 

### How to run Chreval? ###

To run, the minimum information you need is a heat map data file.
This file contains experimental data from a source (particularly
ChIA-PET but possibly Hi-C). This data (particularly ChIA-PET) is
highly correlated with positions of the CTCF binding proteins and
cohesin. The CTCF sites largely correspond to regions that form loops
and typically regulate the chromatin.

The heat maps consists of a symmetric matrix where the indices (i,j),
corresponding to row and column positions, indicate points on the
chromatin chain where segment i and j interact with each other. For
simplicity, we assume i < j, and look only at the upper triangle. A
reflection of matrix is found across the diagonal. The intensity of a
given point (i,j) is proportional to the frequency that this
particular interaction was encountered in the experimental data, so
small numbers mean only weak interactions, and large numbers mean
strong interactions.

Once you obtain a heat make, or have created one using
my_generation.py, you can run this program with the following command
line example:

    > chreval.py -f myexample.heat

Specifically, in the directory "tests", the following command can be
used to calculate the example heatmap:

    > cd tests
    > chreval.py -f chr10_64313472_64921344_res5kb.heat

or, another example

    > cd tests/test_anal_loops/eheat_files
    > chreval.py -f chr1_1890973_2316695.eheat


The file `chr10_64313472_64921344_res5kb.heat` has the labeling
information `chrN_x_y_res5kb.heat`, where N is the chromosome number,
`x` is the starting position and `y` is the ending position. Further, 
the `res5kb` indicates the resolution of the grid that was generated. 
If the grid size is smaller or larger, an option can be used to change
this grid size from the default (presently 5 kb). The file must end
with the extension heat or the newer form "eheat" (extended heatmap
file).

The output directory from chreval in the above example will be
`chr10_64313472_64921344_res5kb`; i.e., the directory name
`chrN_x_y_res5kb`. This directory contains (in separate files) the top
layer of suboptimal structures within some specifable energy range
from the mimumum free energy (default is 10 kcal/mol) or a fractional
percentage of the free energy. These files have the extension `*.DBN`
and can be read by the 3rd party software package [VARNA](http://varna.lri.fr/).
Additionally, _Chevral_ is set up to provide heatmap, pairing info, 
and restraint files for `SimRNA 3D` calculations (see instructions 
below). Two additional  files are `chrN_x_y_res5kb_BDwt.clust` that 
contains a matrix with the Boltzmann probabilities for different 
interactions and `chrN_x_y_res5kb_summary.txt` that contains a shorthand 
list of the secondary structures.

There are a variety of additional options. Please run

    > chreval.py -h 

to obtain additional information on additional command line options.


### How to run Anal_Loops? ###

An example for `anal_loops.py` is provided in the directory `tests/test_anal_loops`

    > cd tests/test_anal_loops
    > anal_loops.py -ff test_loops.CTCF.withAandB.annotated.bed

The program `anal_loops.py` will look up files in the same directory as
the `*.bed` file and try to compute the free energy using the object
_Chreval_.

For more information on how to run the program, please run

    > anal_loops.py -h


### How to run Make_HeatMap? ###

To generate visuals of the 2D heatmaps from the standard input heatmap files, 

    > cd tests/test_anal_loops/eheat_files
    > make_heatmap.py chr10_64313472_64921344_res5kb.heat

More recent file design includes extended heatmaps that contain more information 
than just the 2D contact weights. Extended heatmaps are valuable for inputing 
specific information about about CTCF orientations, where the older heatmap file 
version only offers a generic intensity as a means to distinguish CTCF sites

    > cd tests/test_anal_loops/eheat_files
    > make_heatmap.py chr1_1890973_2316695.eheat 

where the extension `*.eheat` indicates an extended heatmap file.

The output at the end of the program indicates the distribution of contacts
as a function of genomic distance.

The program can also make heat maps based on the `*.clust` files from a 
_Chreval_ calculation.

    > chreval.py -f chr1_1890973_2316695.eheat
    ... (wait for various output to finish)
    > cd chr1_1890973_2316695
    > make_heatmap.py chr1_1890973_2316695_BDwt.clust

For more information on how to run the program, please run

    > make_heatmap.py -h


### How to run My_Generation? ###

...Good luck with running my generation, you might have an easier time 
"hearding cats"! ;-) <grin>.  

Back to the subject at hand, here is an example on how to run **the program**:

    > my_generation.py -seq ".ABCDE.((..)).abcde" "{.................}"

Note that it doesn't matter that one of the structures overlaps the
other one.

For information on the input format, run

    > my_generation.py -hExFile
  
when using an input file, or 

    > my_generation.py -hExSeq

when using the command line. The input file has the advantage that one 
can specify the actual weight of the particular structure on each line
and build up a complex structure as a result of specific information 
entered. Whereas the program comes with a few very simple error checks,
it should be remembered that the program **doesn't replace the job of 
thinking**. Therefore, the user is responsible for inputting information 
that makes sense.

For more information on how to run the program, please run

    > my_generation.py -h


### How to run the SimRNA package to obtain 3D structures from Chreval outputs? ###

You must download the executable version of `SimRNA` from the 
[Bujnicki lab website](http://genesilico.pl/software/stand-alone/simrna).

Copy the `data` directory and `config.dat` file to a separate directory 
where you want to build the 3D structure.

Copy the relevant simres file to that same directory. 

in data, change all the values in `histograms3D_3.list` to `0.0`, except for 
the last four that have `0.1` already in the file:

    > cat histograms3D_3.list
    ./data/AA3.hist    0.0
    ./data/AC3.hist    0.0
    
    ...
    
    ./data/A_3_exvol.hist 0.1
    ...

in the `config.dat` file, change the parameter `ETA_THETA_WEIGHT` from `0.4` to `0.0`

    > cat config.dat
    ...
    
    ETA_THETA_WEIGHT 0.00


Build a sequence of the proper length (`N = integer`) for poly(A)

    > SimRNA_make_polyA.py N  > myseq.seq


Now run a replica exchange Monte Carlo simulation using SimRNA

    > SimRNA3 -s myseq.seq -r myseq.simres -c config.dat -E 10 -o myseq_x >& myseq_x.log &

To generate sequences, use the following

    > cat myseq_x_??.trafl >> myseq_x.trafl
    > clustering myseq_x.trafl 0.1 15.0

this generates the files

    > myseq_x_thrs15.00A_clust01.trafl

To convert the first trajectory in this file to a pdb representation:

    > SimRNA_trafl2pdbs myseq_x_01-000001.pdb myseq_x.trafl 1

this generates the file `myseq_x_thrs15.00A_clust01-000001.pdb`.

Now we convert that to a single bead representation

    > SimRNA2mds.py myseq_x_thrs15.00A_clust01-000001.pdb

which generates a file `myseq_x_thrs15.00A_clust01-000001_v.pdb`, where 
this pdb file can be viewed in a recognizable for using `chimera`, `vmd`,
or `pymol`


----
Example of command line calls for the contributed software

    > SimRNA_make_polyA.py 10
    aaaaaaaaaa

    > SimRNA2mds.py SimRNA_pdboutput.pdb

generates a PDB file formatted in a way so that (currently) the
phosphate atom is treated as the main binding interaction. The
particular atom can be changed in the program, and even more than one
atom displayed, if actually desired.

----




Version 0.5


### How do I get set up? ###

* Summary of set up

The package can run as is, but some applications may require the
installation of the following python packages, if they are not 
already installed:

`matplotlib`  
`numpy`  
`random`  
`argparse`


* Configuration

Standard python 2.7

* Dependencies

`matplot`, `numpy`, `random` and `argparse`; otherwise, none

* Database configuration

Presently, this package is essentially complete. However, the user
will have to build a databased of ChIA-PET data (or Hi-C) data to
do any real analysis. More extensive data is (or will be) made 
available at some point at the contributing laboratory; 
[Laboratory of Functional and Structural Genomics](http://nucleus3d.cent.uw.edu.pl/).

Please consult Dariusz Plewczynski for access to such data that can 
be used in this venue: 

email: dariuszplewczynski@cent.uw.edu.pl


* How to run tests

To see how these program works, please follow the examples above and
refer to examples in the contributed directory titled "tests" in this
distribution.


### Contribution guidelines ###

We welcome any comments, advice and/or suggestion about the 
programs in this package. 

* Writing tests
* Code review
* Other comments or information

### Who do I talk to? ###

To consult about bugs and other issues with the code, please contact 

Wayne Dawson  
Laboratory of Functional and Structural Genomeics  
Center of New Technologies  
University of Warsaw  
Banacha 2C, 02-098 Warsaw  
  
email: w.dawson@cent.uw.edu.pl  

* Repo owner/developer: Wayne Dawson


### Legal notice ###

With the use of _Chreval_, the Licensee who obtains access to this 
software package _Chreval_ agrees to the following terms with respect 
to use of the software package, _Chreval_ (version 0.5) (hereinafter 
called _Chreval_) furnished by the other party (the “Licensor”).

The Licensor grants to the Licensee a non-exclusive, non-transferable, 
permanent license to install and use the _Chreval_ on computer systems 
located at the site of the Licensee’s organization. However, a violation 
of any of the clauses of this license agreement by the Licensee shall 
terminate automatically and with immediate effect the Licensee’s right 
to install, use or store the _Chreval_ in any form. Use of the _Chreval_ 
is restricted to the Licensee and to direct collaborators who are members 
of the organization or company of the Licensee at the site of the Licensee 
and who accept the terms of this license.

1. The Licensee agrees that the _Chreval_ has been developed in 
connection with academic research projects and is provided **as is**. 
The Licensor disclaims all warranties with regard to the _Chreval_ 
or any of its results, including any implied warranties of 
merchantability or fitness for a particular purpose. In no 
event shall the Licensor be liable for any damages, however 
caused, including, without limitation, any damages arising out 
of the use of the _Chreval_, loss of use of the _Chreval_, or damage 
of any sort to the Licensee.

2. The Licensee understands that the _Cherval_ is a pre-released 
version and does not represent a final product from the Licensor. 
In addition, the Licensee understands that the _Chreval_ contain 
errors, or bugs, and other problems that could potentially 
result in system failure or failure in the use of the _Chreval_ 
Licensee holds the entire risk as to the quality and performance 
of the Chreval and the sole responsibility for protecting 1

3. _Cherval_ shall not be used for activities or purposes that are 
unethical or immoral by the standards of the Geneva convention.
