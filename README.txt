The SDREM software is described in:
Linking the signaling cascades and dynamic regulatory networks controlling stress responses. 
Anthony Gitter, Miri Carmi, Naama Barkai, Ziv Bar-Joseph. 
Genome Research. doi:10.1101/gr.138628.112.

Contact agitter@cs.cmu.edu with any questions.

Use of this code implies the user has accepted the terms of license.txt.
The code requires Java 5.0 or above.

Properties files are used to specify the paramaters and input data.
See the DREM manual (http://sb.cs.cmu.edu/drem/DREMmanual.pdf) for details regarding the gene expression and
protein-DNA binding file formats.  The TF-gene priors have the same
format as the DREM protein-DNA binding data file, but have the value 0.5
instead of 1 for all non-zero entries (assuming uniform initial priors).

Examples of how to run SDREM on a cluster are included.  The maximum Java heap size must be
increased when working with large networks.

Many intermediate output files are generated during each iteration of SDREM.
The SDREM executables, code, input files, output produced, and example data are described below:


***********************************************
Precomputing and storing paths (optional)
***********************************************
This is an optional step that can be performed before running SDREM.

StorePaths.jar - The executable for preprocessing the network data.  It searches for paths from the sources to each TF and writes them all to disk, optionally filtering them to keep only the highest confidence paths.  Without doing this preprocessing, SDREM has to search for the paths many times as it iterates, which is wasteful and makes it very, very, very slow.

allPathsEgf.props - An example properties file for StorePaths.jar.  store/filter means the user wants to enumerate paths and remove low-confidence paths (recommended for large PPI networks).  The next lines define the sources, targets, and networks.  Node priors are used to give weights to vertices in the network.  If the user stores and filters paths, then he/she can delete the intermediate output in stored.paths.dir once the filtering step is done (just give the final directory with the filtered paths to SDREM as input).

allPathsEgf.qub - An example showing how to call StorePaths.jar.  This is a submission script for a PBS cluster and the last line shows the actual command.


***********************************************
SDREM
***********************************************
The SDREM algorithm executable.

sdrem.jar - The SDREM executable.

sdrem011712egf.props - A sample SDREM properties file.  model.dir is where the output will be written and where some of the input files must be.  stored.paths.dir is the location of the filtered paths from the preprocessing jar if it was used.  It's important to use the same settings (sources, targets, node priors, path length, etc.) for the StorePaths and SDREM properties.  The rest of the parameters can usually be left at the default values and are described in the Genome Research supplement.

DREM_defaults.txt - A separate DREM properties file has to be set.  It's the same format that the original DREM software uses (as described in http://sb.cs.cmu.edu/drem/DREMmanual.pdf) except the TF-gene_Interactions_File will be generated dynamically and Active_TF_influence is a new SDREM parameter.

sdrem011712egf.qub - An example showing how to call sdrem.jar on a PBS cluster.


***********************************************
Modified DREM
***********************************************
The DREM software (http://sb.cs.cmu.edu/drem/) that SDREM was built upon allows visualization of the
active TFs and gene expression profiles after SDREM is run.  There are several differences between the version distributed here and DREM 2.0:
None of the new features descrbied in the DREM 2.0 manuscript are present yet (e.g. support for motif finding).
To view the final output (assuming 10 iterations), should load 10.model as the saved model and tfActivityPriors_round9.txt as the TF-gene interactions.  Use these priors instead of tfActivityPriors_round10.txt because tfActivityPriors_round10.txt are the updated priors after running the 10th round of network orientation as opposed to the file that was given to DREM as input at the start of the 10th SDREM iteration.
The split table will show the activity score for each TF at that split and the max activity score across all splits.
Key TF Labels includes options to display TFs at each split based on activity score.  If the user uses activity scores to choose which TFs to show, the slider will be used to calculate 10^X (instead of 10^-X) and all TFs with activity scores greater than 10^X at a split are shown.  
The output file 10.targetsStd can be used to help choose the activity score threshold.  The last column in this file gives the max activity score for each active TF.  Therefore, the user can find the minimum of these values and use it (or a value slightly less than it) as the threshold for display purposes.
There is also an option to show the TF only at the earliest split at which it exceeds this threshold on each path.
When using activity score to display TFs, there will be [1] or [2] after the TF id if binary splits were used.  [1] means the TF primarily controls the lower path out of the split and [2] is for the higher path.  For higher order splits (3+), [1] is the lowest path out of the split and [3] is the highest.

drem.jar - The DREM executable, which can be run without command line parameters.  Use the GUI to load the model file from SDREM and the data.  Optionally provide the DREM_defaults.txt file at the command line to load the same settings that SDREM used.  See http://sb.cs.cmu.edu/drem/DREMmanual.pdf for details.


***********************************************
Source code
***********************************************
The executable jar files correspond to following classes:
scripts.StorePaths
alg.SDREM
edu.cmu.cs.sb.drem.DREM_IO


***********************************************
Output files
***********************************************
Intermediate output files are generated at every iteration of SDREM, which can be used for debugging and analysis.  At each iteration 'N', SDREM writes the following files (some filenames may vary slightly based on the parameters used):

/N - A subdirectory that contains output from the DREM runs that use randomized TF binding data.  These can be ignored or deleted.

N.model - The DREM model file for iteration N, which can be loaded by drem.jar.
N.model.activities - Activity information for the TFs used to determine which TFs should be targets.
N.model.activitiesDynamic - Activity information for the TFs used to determine which TFs should be targets.
N.model.activitiesStd - Activity information for the TFs used to determine which TFs should be targets.

N.targets -The TFs that were selected as targets at iteration N and their target weights.  These are the proteins that were used for the network orientation.
N.targetsStd - Like N.targets but contains information about the distribution of random TF activity scores.

itrN.out - A log file.

conflictOrientations_itrN.txt - A code that represents how each PPI was oriented.  See pathEdges_iterN.txt for a human-readable version.

nodeScores_iterN_Pathweight_10_1000.txt - A summary of the oriented network at iteration N.  It gives the sources, target TFs (same ones as N.targets) and various measures of how many oriented paths use a particular protein.  The '% top 1000 paths through node' column was used to select nodes in the Genome Research paper.  From this file, the user can extract the sources, targets, and all other proteins that have a value >= 0.01 in this column (called the 'internal' or 'signaling' proteins).  That score is saying that at least 1% of the highest confidence oriented paths go through that particular protein.

pathEdges_iterN.txt - The predicted PPI orientation for all edges that were used on at least one source-target path (i.e. some edges are excluded because they are not "between" the sources and targets).

scores_itrN_r1.0_PathWeight_10_1000.txt - Connectivity of the true target TFs in the network orientation versus randomly selected targets, which is used to determine which TFs' priors should be increased or decreased at the next iteration of SDREM.

tfActivityPriors_roundN.txt - The new TF-gene binding file that will be used as input at iteration N+1.

statisfiedPaths_itrN.txt.gz - Lists every oriented, satisfied path that connects a source and target and has less than 5 edges.  It doesn't contain any information that couldn't be reconstructed from pathEdges_itrN.txt but is more convenient for downstream analyses that examine individual paths.

After the final iteration SDREM writes:
droppedTargets.txt - Targets that were present at iteration N-1 but not iteration N.
targetsByIteration.txt - All targets at each iteration.
newTargets.txt - Targets that were not present at iteration N-1 but are at iteration N.


***********************************************
Sample data
***********************************************
The example property files refer to these data files, which demonstrate the expected file formats.  These are human datasets, and all ids are NCBI Gene ids.  DREM always requires that all proteins and genes use the same type of identifier, and it is especially important that the same types of ids are used in the PPI network and the TF columns of the TF-gene interaction files.  For example, do not use gene symbols in one and ids in the other.

Yarden_MCF10A_expr.txt - Sample expression data in DREM format (see http://sb.cs.cmu.edu/drem/DREMmanual.pdf).

ppi_ptm_pd_edges.txt - The human interaction network, which uses PPI from BioGRID and HPRD (pp), predicted protein-DNA binding edges (pd), and post-translational modifications from HPRD (ptm).

tfList.txt - Gene ids for the TFs that have protein-DNA interactions.  This is used for enumerating paths and these proteins are also the set of possible random targets for when SDREM builds a background distribution of TF network connectivity scores.

tfActivityPriors_round0.txt  - A DREM-style protein-DNA binding grid (see http://sb.cs.cmu.edu/drem/DREMmanual.pdf) but with 0.5 as a prior for all interactions.  This will be updated at each iteration as the network is used to refine the priors.

sources.txt - The ids of the proteins that are used as the sources for the network orientation in all of the iterations.