2. Initialize a Git repository. Commit to the repo throughout the assignment as you see fit, but at least once for each “Part” of this assignment. Use and commit a .gitignore file as appropriate. (Tip: Many of you would do well to recall the feedback you got on your Git repo from the previous assignment!)

```bash
git init
touch .gitignore
```

3. Inside dir GA4, create a README.md file and open it in the VS Code editor. Use this file throughout the assigment to add your answers where appropriate.

```bash
touch README.md
```

4. Inside dir GA4, also create dirs scripts and results. We’ll make your dir self-contained again (like a typical situation for a research project) by copying some data into it:

```bash
mkdir scripts/ results/
```

5. As a starting script, store the code below in a shell script scripts/trimgalore.sh. This is a script to run TrimGalore, which should be very similar to your final script from last week’s exercises.

```bash
touch scripts/trimgalore.sh
```
Git realted commands:

```bash
git add README.md 
git commit -m "Answered Part -A"
echo data/ > .gitignore 
echo results/ >> .gitignore 
git add .gitignore 
git commit -m "Added a .gitignore file for the project"
git add scripts/
git commit -m "Added script dir for the project"
```
---

## PART-B

6. Add sbatch options:

#SBATCH --account=PAS2880
#SBATCH --cpus-per-task=8
#SBATCH --time=30
#SBATCH --output=slurm-trim-%j.out
#SBATCH --mail-type=FAIL


7. By printing and scanning through the TrimGalore help info once again (see last week’s exercises), find the TrimGalore option that specifies how many cores it can use – add the relevant line(s) from the TrimGalore help info to your README.md. In the script, change the trim_galore command accordingly to use the available number of cores.

```bash
apptainer exec oras://community.wave.seqera.io/library/trim-galore:0.6.10--bc38c9238980c80e trim_galore --help
```
The option that sepcifies number of cores usage is `-j` 

For paired end reads, `--cores 4` is the sweet spot and alloting more could cause diminishing returns

```bash
git add scripts/trimgalore.sh
git commit -m "Changed the number of cores"
```

8. To test the script and batch job submission, submit the script as a batch job only for sample ERR10802863.

```bash
R1=data/ERR10802863_R1.fastq.gz
R2=data/ERR10802863_R2.fastq.gz
sbatch scripts/trimgalore.sh "$R1" "$R2" results/trimgalore
```

9. Monitor the job, and when it’s done, check that everything went well (if it didn’t, redo until you get it right). In your README.md, explain your monitoring and checking process. Then, remove all outputs (Slurm log files and TrimGalore output files) produced by this test-run.

```bash
squeue -u sskavya123 -l
```
The output was:

```
Sun Oct 19 15:05:15 2025
             JOBID PARTITION     NAME     USER    STATE       TIME TIME_LIMI  NODES NODELIST(REASON)
          37867440       cpu trimgalo sskavya1  RUNNING       0:06     30:00      1 p0010
          37867351       cpu ondemand sskavya1  RUNNING    1:10:54   2:00:00      1 p0224
```

```bash
ls -lh results/trimgalore/
```
The output was:

```
total 43M
-rw-rw----+ 1 sskavya123 PAS1838 2.4K Oct 19 15:05 ERR10802863_R1.fastq.gz_trimming_report.txt
-rw-rw----+ 1 sskavya123 PAS1838 675K Oct 19 15:05 ERR10802863_R1_val_1_fastqc.html
-rw-rw----+ 1 sskavya123 PAS1838 348K Oct 19 15:05 ERR10802863_R1_val_1_fastqc.zip
-rw-rw----+ 1 sskavya123 PAS1838  20M Oct 19 15:05 ERR10802863_R1_val_1.fq.gz
-rw-rw----+ 1 sskavya123 PAS1838 2.3K Oct 19 15:05 ERR10802863_R2.fastq.gz_trimming_report.txt
-rw-rw----+ 1 sskavya123 PAS1838 676K Oct 19 15:05 ERR10802863_R2_val_2_fastqc.html
-rw-rw----+ 1 sskavya123 PAS1838 341K Oct 19 15:05 ERR10802863_R2_val_2_fastqc.zip
-rw-rw----+ 1 sskavya123 PAS1838  21M Oct 19 15:05 ERR10802863_R2_val_2.fq.gz
```

```bash
rm -r results/trimgalore/
 rm -r slurm-trim-37867440.out 
```
10. In the FastQC outputs for the R2 file that you just produced with TrimGalore (recall that it runs FastQC after trimmming!), do you see any evidence for this problem? Explain.

Yes, there is evidence for over-represented G nucleotides in R2 fastqc report.

Sequence	                                          Count	  Percentage	     Possible Source
GGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGGG	1140	0.24461157184392066	No Hit

11. We’ll assume that the data was indeed produced with the newer Illumina color chemistry. In the TrimGalore help info, find the relevant TrimGalore option to deal with the poly-G probelm, and again add the relevant line(s) from the help info to your README.md. Then, use the TrimGalore option you found, but don’t change the quality score threshold from the default.

Since the problem is particularly with read 2 of paired end sequencing, 

`-a2` or `--adapter2` will be the relevant option and since the over represented sequence has 50 G's, `-a2 G{50}` could be used.

12. Rerun TrimGalore with the added color-chemistry option. Check all outputs and confirm that usage of this option made a difference. Then, remove all outputs produced by this test-run again.

```bash
R1=data/ERR10802863_R1.fastq.gz
R2=data/ERR10802863_R2.fastq.gz
sbatch scripts/trimgalore.sh "$R1" "$R2" results/trimgalore
```
Yes, adding the option to the script made a difference. There are no over-represented sequence in the fastqc report for read 2.

```bash
rm -r slurm-trim-37867458.out 
rm -r results/trimgalore/
```


