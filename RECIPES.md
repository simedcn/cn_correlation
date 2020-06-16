#->> key: # = comment; $ = unix; | = what you should see; >> = matlab

### clone the broadinstitute cn_correlation github repository
# My local github repositories are under the directory ~/git
# Yours may vary...
```
$ cd ~/git/broadinstitute
$ git clone https://github.com/broadinstitute/cn_correlation.git
$ cd cn_correlation/
```
### matlab setup
The user matlab startup script ~/matlab/startup.m runs when you start
up a matlab session. It is useful for setting up the paths matlab
searches for functions (defined in text files with the .m extension).
The recipe below assumes you are creating the file for the first
time: if you already have a startup file you need to edit it and add
the contents of the repo startup. Note that you should edit the file
anyway in order to reflect your own local git repo ppath.
```
$ cat startup.m
|if ~isdeployed
|    addpath ~/git/broadinstitute/cn_correlation
|    addpath ~/git/broadinstitute/cn_correlation/snputil
|end
$ mkdir ~/matlab
$ cp startup.m ~/matlab/
```
### edit the matlab function definition file corrperm_lsf_submission.m
the string cmd_template defined starting on line 42 will need to be modified
|cmd_template = ['bsub ',...
|          '-R "rusage[mem=4]" ',... 
|          '-mig 5 ',...
|          '-R "select[cpuf>100]" ',... 
|          '-Q "EXCLUDE(127)" ',...
|          '-q hour ',...
|          '-W 240 ',...
|          '-P cancerfolk ',...
|          '-o ',perm_dir,'%s.out.txt -e ',perm_dir,'%s.err.txt ',...
|          '-r ',wrapscript,' ',executable,' ',ref_dir,' ',perm_dir ' %s %d\n']; 
# In particular, the '-P cancerfolk' is a Broad-specific option naming
# the group that is funding the compute that will need to change. The
# '...'s at the end of each line are to continue the matlab statements
# to the next line. 

### copy the example directory files to a working directory with large storage quota
I left my example run from this recipe there for comparison (run with GridEngine instead of LSF)
```
$ ls example/
$ mkdir /xchip/beroukhimlab/gistic/corrperm/example
$ cp example/* /xchip/beroukhimlab/gistic/corrperm/example
$ cd /xchip/beroukhimlab/gistic/corrperm/example
```
### do whatever you need to do on your cluster to run matlab 2014a
(Broad uses the dotkit 'use .matlab-2014a' command)

### then run matlab, and launch the example script
This reruns the 5000-permutation test on low-level disruption from
Travis's 2013 paper. It's worth looking at the first dozen and last
30 lines of ng_pancan_corrperm_ll_recap.m to understand what's going on.
The outputs will be placed in 3 directories
.../example/ll_work - input directory for the permutations
.../example/ll_work/ll_permout - output directory for the permutations
.../example/ll_work/ll_results - output directory for significance
                                 analysis of the permutations

```
$ matlab
>> ng_pancan_corrperm_ll_recap
```
If all goes well, this will run the data preparation and tuning phases for a few
minutes and then pause with a 'K>>' prompt from the 'keyboard'
matlab command. This allows you to enter matlab commands before
starting the permutations. If you type the command 'return' you will
run the edited corrperm_lsf_submission() function and attempt to
submit 100 permutation chunk tasks of 50 permutations each to LSF

K>> return

If by some miracle this works the first time, you will get another
keyboard 'K>>' prompt. Typing 'return' again will proceed to running 
the analysis corrperm_analyze_pairs2 once the completed permutations.
More likely you will keep editing corrperm_lsf_submission.m and
resubmitting until it works. To resubmit, type

>> corrperm_lsf_resub(ref_dir)

This matlab function picks up the parameters from the last LSF
submission stored in the file permute_options.mat in the reference 
directory and attempts a rerun. 

### analyze data

The last 10-ish lines of ng_pancan_corrperm_ll_recap do a "pairs2"
analysis of the data. I snipped them off and put them in
the file example/analyze_ll_pairs2.m to define a matlab command script you can
run by typing

>> analyze_ll_pairs2

                                                                                
