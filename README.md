# TwinCons: analysis toolkit for two sequence groups within an MSA

Conservation score that highlights conserved, variable and diverging (signature) positions between two sequence groups within an alignment. The method mathematically determines a ‘cost’ of transforming one alignment group to the other. Includes automated parsing protocol for the detection of continuous stretches (segments) of high TwinCons scoring columns within protein alignments to query deep ancestry of short peptides.

## Dependencies
Programs required to be present in the PATH:
- MAFFT https://mafft.cbrc.jp/alignment/software/
- DSSP https://swift.cmbi.umcn.nl/gv/dssp/ or in Linux just execute
	> apt-get install dssp

## [TwinCons.py](./bin/TwinCons.py)

Generates data for subsequent scripts or can be used independently. Calculates TwinCons conservation for a given alignment.

### Usage

1. Input files
- One fasta alignment file with two defined groups **(Required)**. If no groups are defined a phylogenetic tree can be built from the alignment, the groups are defined by the deepest branching point in the tree. Alternatively two alignment files can be provided, each defining a single group - mafft-merge will be used to merge them in a single alignment.
- One or two structure files for each group to map data. The name of the file must include the sequence group name as defined in the alignment. (Optional)

Example sequence group definitions in fasta format:
```
>GROUP1_TAXID1_SEQNAME1
MTKF-EVPKEISDKVLQTLELAKNTG
>GROUP1_TAXID2_SEQNAME2
MTKF-EVPKEISDKVLQTLELAKNTG
>GROUP2_TAXID3_SEQNAME3
MTKF-EVPKEISDKVLQTLELAKNTG
>GROUP2_TAXID4_SEQNAME4
MTKF-EVPKEISDKVLQTLELAKNTG
```

Example structure file naming:
```
SEQNAME1_GROUP1.pdb
SEQNAME3_GROUP2.pdb
```

Typical usage:
```
TwinCons.py -a ./data/ALNS/test_aln.fa -mx blosum62 -csv -o ./test_aln
TwinCons.py -a ./data/ALNS/casp9-mcasp_struct.fa -pml unix -s ./data/PDB/HUMAN_CASP9.pdb ./data/PDB/YEAST_MCASP.pdb -ssbe -sy ./data/PDB/HUMAN_CASP9.pdb ./data/PDB/YEAST_MCASP.pdb -o ./twc_ssbe_HS-CASP9_SC-MCASP
```

2. Output files
	- **pml file** for all structures with residue colors defined by the score
	- **svg** with score trace for alignment position
	- **csv** output for [**RiboVision**](http://apollo.chemistry.gatech.edu/RiboVision2/) when provided with structure file
	- **csv** output with scores per alignment position
	- **optional data** if ran as a module within other python scripts **for multiple alignment analysis**

Usage:
```
TwinCons.py [-h] [-o OUTPUT_PATH] (-a ALIGNMENT_PATHS [ALIGNMENT_PATHS ...] | -as ALIGNMENT_STRING) [-bn {uniform,bgfreq}] [-cg] [-gg] [-gt GAP_THRESHOLD] [-s STRUCTURE_PATHS [STRUCTURE_PATHS ...]] [-sy STRUCTURE_PYMOL [STRUCTURE_PYMOL ...]]
                   [-phy] [-nc] [-w {pairwise,voronoi}] [-ca] (-p | -pml {unix,windows} | -r | -csv | -rv | -jv)
                   [-mx {benner6,benner22,benner74,blosum100,blosum30,blosum35,blosum40,blosum45,blosum50,blosum55,blosum60,blosum62,blosum65,blosum70,blosum75,blosum80,blosum85,blosum90,blosum95,genetic,gonnet,ident,johnson,levin,miyata,nwsgappep,pam120,pam180,pam250,pam30,pam300,pam60,pam90,risler,structure,blastn,identity,trans} | -cm CUSTOM_MATRIX | -lg | -e | -rs]
                   [-ss | -be | -ssbe]

Calculate and visualize conservation between two groups of sequences from one alignment

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT_PATH, --output_path OUTPUT_PATH
                        Output path
  -a ALIGNMENT_PATHS [ALIGNMENT_PATHS ...], --alignment_paths ALIGNMENT_PATHS [ALIGNMENT_PATHS ...]
                        Path to alignment files. If given two files it will use mafft --merge to merge them in single alignment.
  -as ALIGNMENT_STRING, --alignment_string ALIGNMENT_STRING
                        Alignment string
  -bn {uniform,bgfreq}, --baseline {uniform,bgfreq}
                        Whether to baseline the used matrix with the uniform vector or with the matrix background frequency.
                                (Default: bgfreq)
  -cg, --cut_gaps       Remove alignment positions with % gaps greater than the specified value with gap_threshold.
  -gg, --calculate_group_gaps
                        Calculate alignment position gaps in 3 groups using 2*gap threshold value:
                                Ungapped - Aligned positions;
                                GroupGap - Only one group has sequences;
                                AllGap - Both groups are gapped.
  -gt GAP_THRESHOLD, --gap_threshold GAP_THRESHOLD
                        Specify % gaps per alignment position. (Default = the smaller between ((sequences of group1)/(all sequences) and (sequences of group2)/(all sequences)) minus 0.05)
  -s STRUCTURE_PATHS [STRUCTURE_PATHS ...], --structure_paths STRUCTURE_PATHS [STRUCTURE_PATHS ...]
                        Paths to structure files, for score calculation. Does not work with --nucleotide!
  -sy STRUCTURE_PYMOL [STRUCTURE_PYMOL ...], --structure_pymol STRUCTURE_PYMOL [STRUCTURE_PYMOL ...]
                        Paths to structure files, for plotting a pml.
  -phy, --phylo_split   Split the alignment in two groups by constructing a tree instead of looking for _ separated strings.
  -nc, --nucleotide     Input is nucleotide sequence. Specify nucleotide matrix for score calculation with -mx or entropy calculations with -e or -rs
  -w {pairwise,voronoi}, --weigh_sequences {pairwise,voronoi}
                        Weigh sequences within each alignment group.
  -ca, --compositional_adjustment
                        Adjust the substitution matrix with residue frequencies computed from the two alignment groups.
                         Available only for BLOSUM matrices, using the methods decribed in doi.org/10.1073/pnas.2533904100 and doi.org/10.1093/bioinformatics/bti070.
  -p, --plotit          Plots the calculated score as a bar graph for each alignment position.
  -pml {unix,windows}, --write_pml_script {unix,windows}
                        Writes out a PyMOL coloring script for any structure files that have been defined. Choose between unix or windows style paths for the pymol script.
  -r, --return_within   To be used from within other python programs. Returns dictionary of alnpos->score.
  -csv, --return_csv    Saves a csv with alignment position -> score.
  -rv, --ribovision     Saves a csv formatted for RiboVision. Requires at least one structure defined with the -sy argument.
  -jv, --jalview_output
                        Saves an annotation file for Jalview.
  -mx {benner6,benner22,benner74,blosum100,blosum30,blosum35,blosum40,blosum45,blosum50,blosum55,blosum60,blosum62,blosum65,blosum70,blosum75,blosum80,blosum85,blosum90,blosum95,genetic,gonnet,ident,johnson,levin,miyata,nwsgappep,pam120,pam180,pam250,pam30,pam300,pam60,pam90,risler,structure,blastn,identity,trans}, --substitution_matrix {benner6,benner22,benner74,blosum100,blosum30,blosum35,blosum40,blosum45,blosum50,blosum55,blosum60,blosum62,blosum65,blosum70,blosum75,blosum80,blosum85,blosum90,blosum95,genetic,gonnet,ident,johnson,levin,miyata,nwsgappep,pam120,pam180,pam250,pam30,pam300,pam60,pam90,risler,structure,blastn,identity,trans}
                        Choose a substitution matrix for score calculation.
  -cm CUSTOM_MATRIX, --custom_matrix CUSTOM_MATRIX
                        Provide path to a custom PAML format matrix. For example format see the matrices folder.
  -lg, --leegascuel     Use LG matrix for score calculation
  -e, --shannon_entropy
                        Use shannon entropy for conservation calculation.
  -rs, --reflected_shannon
                        Use shannon entropy for conservation calculation and reflect the result so that a fully random sequence will be scored as 0.
  -ss, --secondary_structure
                        Use substitution matrices derived from data dependent on the secondary structure assignment.
  -be, --burried_exposed
                        Use substitution matrices derived from data dependent on the solvent accessability of a residue.
  -ssbe, --both         Use substitution matrices derived from data dependent on both the secondary structure and the solvent accessability of a residue.
```

# Multiple alignment analysis

[CalculateSegments.py](./bin/CalculateSegments.py) executes the [TwinCons.py](./bin/TwinCons.py) script for a given folder with sequence alignments. Calculates length, weight, normalized lengths and positions of high scoring segments from the results of TwinCons.

Tries to guess the type of comparison and color code the included datasets. For lower number of alignments (up to 20) applies different color for each alignment. For greater number of alignments tagged in different groups (e.g. A_alignment-nameX.fas, B_alignment-nameY.fas and so on), uses the viridis colormap to color each group of alignments together. For exactly 10 alignments in a folder assumes they are ordered by similarity and colors them with a Purple Green gradient.

Can pass all options for calculation already present in TwinCons with the option -co. <span style="color:red">However, as of now it does not support structure mapping of scores or using structure defined matrices</span>. 

It does support the options: **-gt**, **-cg**, **-phy**, [**-lg**, **-bl**, **-e**, **-c**]. Should be passed as separate arguments after the flag -co without the dashes and underscore for flags with parameters. -co should be the last argument passed to [CalculateSegments.py](./bin/CalculateSegments.py) since any argument following -co will be passed to [TwinCons.py](./bin/TwinCons.py). 

Typical usage:
```
twcCalculateSegments.py ./folder_with_alignments/ ./output_file -c -cms 9 -co cg gt_0.9 phy bl
```
Usage:
```
twcCalculateSegments.py [-h] (-a ALIGNMENT_PATH | -twc TWINCONS_PATH) [-t LENGTH_THRESHOLD] [-it INTENSITY_THRESHOLD] [-cms CUMULATIVE_SEGMENTS] [-avew] [-np] [-c] [-p] [-l] [-co CALCULATION_OPTIONS [CALCULATION_OPTIONS ...]] output_path

Calculates segments for multiple or single alignments

positional arguments:
  output_path           Path to image for output.

optional arguments:
  -h, --help            show this help message and exit
  -a ALIGNMENT_PATH, --alignment_path ALIGNMENT_PATH
                        Path to folder with alignment files.
  -twc TWINCONS_PATH, --twincons_path TWINCONS_PATH
                        Path to folder with csv output files from TwinCons.py
  -t LENGTH_THRESHOLD, --length_threshold LENGTH_THRESHOLD
                        Threshold for consecutive low scores that split positive segments.                                                
                        Default: 3
  -it INTENSITY_THRESHOLD, --intensity_threshold INTENSITY_THRESHOLD
                        Threshold for intensity over which a score is considered truly positive.                                                
                        Default: 1
  -cms CUMULATIVE_SEGMENTS, --cumulative_segments CUMULATIVE_SEGMENTS
                        Delineate segments based on cumulative score and local minima/maxima, instead of specific thresholds.                                                
                        Argument should provide window size for smoothing of the cumulative score.
  -avew, --average_weight
                        Use average weight for segments, instead of using their total weight.
  -np, --treat_highly_negative_as_conserved
                        Treat low scoring positions as conserved for segment calculation.                                                 
                        Considers the absolute for negative positions when comparing with intensity threshold.
  -c, --csv             Output length and weight distributions in a csv file.                                                 
                        Uses the output file name specified by appending .csv
  -p, --plot            Plot a scatter of the segments.
  -l, --legend          Draw a legend.
  -co CALCULATION_OPTIONS [CALCULATION_OPTIONS ...], --calculation_options CALCULATION_OPTIONS [CALCULATION_OPTIONS ...]
                        Options for TwinCons calculation. See README for details.
```

Sample output:

<img src="./data/CSV/BBS_cg09_it1_lt3.png">

# Analyzing TWC results

## Training a classifier

Must include parameters used in TwinCons and CalculateSegments. Use the same format as -co from Calculate segments. For example:

	twcSVMtrain.py output.csv output.pkl -pd output.png -ts 1 -tp 1 -twca mx_blosum62 gt_0.9 cg -csa lt_3 it_2

Usage:
```
twcSVMtrain.py [-h] [-twca TWINCONS_ARGS [TWINCONS_ARGS ...]] [-csa CALCSEGM_ARGS [CALCSEGM_ARGS ...]] [-pd PLOT_DF] [-tp PENALTY] [-k {linear,poly,rbf,sigmoid,precomputed}] [-g {auto,scale}]
                      [-l {absolute,normalized,cms}] [-ts TOP_SEGMENTS]
                      csv_path output_path

Generate SVM from alignment segments.
Computes a decision function from csv generated with twcCalculateSegments

positional arguments:
  csv_path              Path to csv file storing alignment segment data
  output_path           Output path

optional arguments:
  -h, --help            show this help message and exit
  -twca TWINCONS_ARGS [TWINCONS_ARGS ...], --twincons_args TWINCONS_ARGS [TWINCONS_ARGS ...]
                        Arguments used with TwinCons.
  -csa CALCSEGM_ARGS [CALCSEGM_ARGS ...], --calcsegm_args CALCSEGM_ARGS [CALCSEGM_ARGS ...]
                        Arguments used with twcCalculateSegments.
  -pd PLOT_DF, --plot_df PLOT_DF
                        Path to output plot for the decision function.
  -tp PENALTY, --penalty PENALTY
                        Penalty for training algorithm. (Default = 1)
  -k {linear,poly,rbf,sigmoid,precomputed}, --kernel {linear,poly,rbf,sigmoid,precomputed}
                        Kernel for the training algorithm
  -g {auto,scale}, --gamma {auto,scale}
                        Gamma function for training algorithm
  -l {absolute,normalized,cms}, --length_type_calculation {absolute,normalized,cms}
                        Choose what type of segment calculation should be used.        
                                 absolute:   absolute length of the segments.        
                                 normalized: length of segments is normalized with the total alignment length.        
                                 cms:        average position (center of mass) from all segments per alignment.
  -ts TOP_SEGMENTS, --top_segments TOP_SEGMENTS
                        Limit input for each alignment to the top segments that cover        
                        this percentage of the total normalized length and weight. (Default = 0.5)
```

Example output of BaliBASE decision boundary:
<img src="./data/outputs/SVM/BBS.png">

## Testing a classifier
Usage:
```
twcSVMtest.py [-h] [-pd PLOT_DF] [-ts TOP_SEGMENTS] [-l {absolute,normalized,cms}] (-tcp | -tqa) [-dt Start End Step] csv_path output_path pickle

Evaluates alignment entries in csv generated from twcCalculateSegments.
Requires a decision function and json features generated from SVM_train.
Train and test only with the same parameters!
Such parameters can be % cutting gaps, center mass segments, top segments.

positional arguments:
  csv_path              Path to csv file storing alignment segment data
  output_path           Path to output significant segment results
  pickle                Provide path to classifier pickle binary file. The script will also search for an identically                                    
                        named file with extension ".json" containing parameters used for training the classifier, for example:                                    
                        pickle file:    random_test.pkl                                    
                        maximal values: random_test.pkl.maxvals

optional arguments:
  -h, --help            show this help message and exit
  -pd PLOT_DF, --plot_df PLOT_DF
                        Path to output plot for the decision function.
  -ts TOP_SEGMENTS, --top_segments TOP_SEGMENTS
                        Limit input for each alignment to the top segments that cover                                    
                         this percentage of the total normalized length and weight. (Default = 0.5)
  -l {absolute,normalized,cms}, --length_type_calculation {absolute,normalized,cms}
                        Choose what type of segment calculation should be used.        
                                 absolute:   absolute length of the segments.        
                                 normalized: length of segments is normalized with the total alignment length.        
                                 cms:        average position (center of mass) from all segments per alignment.
  -tcp, --test_classifier_precision
                        Provided csv is annotated for testing the classifier.
  -tqa, --test_query_alignments
                        Provided csv is a query and is not annotated for testing the classifier.
  -dt Start End Step, --range_distance_thresholds Start End Step
                        Range of distances from the decision boundary to evaluate.                                    
                        Default for non evalue (-20, 20, 0.05).
```


### Average distance
In the case of large segments there will be few segments and they will be far away from the boundary => cost nearing 0. In the case of many small segments their distance to the boundary will be accumulated resulting in big negative number (larger than any segment can attain on it's own) => cost nearing infinity.

### Identifying significant segments

<img src="./data/outputs/SVM/BBS_vs_BBS.png">
