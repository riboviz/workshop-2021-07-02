# Hands-on session to run RiboViz on a small dataset

Experts session: Hands-on Ribo-Seq workflows - RiboViz

Ribosome Profiling Workshop

2nd July 2021.

Mike Jackson

---

## Run RiboViz on "vignette" dataset

In this hands-on session, I will lead you through running RiboViz on a dataset, explaining how RiboViz is configured and run and the various outputs that it produces.

We'll be using the current version of RiboViz from our Git repository, [riboviz/riboviz](https://github.com/riboviz/riboviz) on GitHub. This is release 2.1 of RiboViz in all but name (we have some final tests to do before the official release).

The dataset we'll use is the "vignette", the downsampled *Saccharomyces cerevisiae* (yeast) dataset that is provided with RiboViz. This is a dataset that we prepared for testing and demonstration purposes. Our documentation on the [vignette](https://github.com/riboviz/riboviz/blob/main/docs/user/run-vignette.md), which you can look at later, describes the provenance of all the data files used in the "vignette". To complement RiboViz, we provide a [riboviz/example-datasets](https://github.com/riboviz/example-datasets) repository on GitHub which is a library of configuration files produced by the RiboViz project that have been run on full-size datasets for various organisms.

### Configuration

I'll start with the RiboViz configuration file for the vignette.

```console
$ cd riboviz
$ more vignette/vignette_config.yaml
```

All of RiboViz's configuration is contained within a single configuration file. This is in YAML format, consisting of configuration parameter names and their values. So, for example, we can see that we have defined:

* `build_indices` to be TRUE, which states that we want to create indices for our FASTA files.
* `count_threshold` to be 64, so genes with less than 64 reads will be removed from our results.

The configuration file also specifies the location of all the data files processed by RiboViz. These can be divided into three classes:

We have organism data. This includes:
 
* A FASTA file with transcript sequences with both coding regions and flanking regions, `orf_fasta_file`.
* A GFF file consistent with this FASTA file which specifies the location of coding sequences within the FASTA file, `orf_gff_file`.
* And, a FASTA file with ribosomal rRNA and other contaminant sequences that we want to avoid aligning to, `rrna_fasta_file`.

We have ribosome profiling data, `fq_files`. This is one or more FASTQ files, each with an associated identifier. These are termed samples. These FASTQ files are assumed to be relative to the directory specified in the `dir_in` parameter. In this configuration these FASTQ files hold downsampled ribosome profiling data for *Saccharomyces cerevisiae* (yeast).

And we have additional organism-specific data, for creating statistics and figures. This includes:

* Summary of fixed A-site positions for each read length, `asite_disp_length_file`.
* Positions of codons within each gene, `codon_positions_file`.
* Features to correlate with ORFs, `features_file`
* tRNA estimates, `t_rna_file`.

Finally, the configuration file also specifies the location to where output files are to be written. These include:

* Index files, `dir_index`.
* Temporary files, `dir_tmp`.
* And, most important of all, output files, `dir_out`.

While the number of configuration parameters may seem daunting, please be reassured that you would never write a configuration file from scratch. Rather, you would take one that you know already works for an existing dataset, such as this one for the "vignette" or one of those from the [riboviz/example-datasets](https://github.com/riboviz/example-datasets) repository and customise it for your own dataset. We also have documentation on [Configuring the RiboViz workflow](https://github.com/riboviz/riboviz/blob/main/docs/user/prep-riboviz-config.md) which describes every configuration parameter. As many have remarked, in various ways, "a few hours of trial and error can save you several minutes of looking at the documentation".

**Question: Does anyone have any questions about the configuration?**

### RiboViz directory contents

Now, let's look at the the `riboviz` directory itself.

```console
$ ls
bash  docs  LICENSE          py-docs    riboviz    rscripts  work
data  jobs  prep_riboviz.nf  README.md  rmarkdown  vignette
```

Of note, we have our Python, R and Rmarkdown scripts which implement specific analysis steps in the workflow. These are in `riboviz`, `rscripts` and `rmarkdown` respectively. These scripts are complemented by the use of third-party tools, including `hisat2-build`, `cutadapt`, `hisat2`, `samtools`, `bedtools` and `umi_tools`.

`prep_riboviz.nf` itself contains the RiboViz workflow, implemented as a Nextflow workflow.

### Set environment

Now, if you haven't already done so, set up your environment by sourcing the `set-riboviz-env.sh` script that you created when you installed RiboViz:

```console
$ source ../set-riboviz-env.sh
```

You may have created yours in a different directory from mine.

Depending on your operating system, the prompt may now show `(riboviz)` showing that the Miniconda Python `riboviz` environment is now active.

**Question: Does anyone have any problems?**

### Validate configuration

We will now run RiboViz. It is good practice to validate the configuration before running the workflow. This checks that the all mandatory parameters have been defined, that parameters have valid values, and that all the input files exist. We can do this as follows:

```console
$ nextflow run prep_riboviz.nf -params-file vignette/vignette_config.yaml --validate_only
```

`nextflow run` is the command to run a Nextflow workflow, in this case our RiboViz workflow, `prep_riboviz.nf`. `-params-file` is the location of our RiboViz configuration file. `--validate_only` says that the workflow should only validate the configuration and then exit. Take care that you provide an underscore (`_`) not a dash (`-`) within the RiboViz parameter name. Unlike Nextflow's own parameters (which include dashes), custom parameters, such as those supported by RiboViz, need underscores if provided via the command-line.

All going well, you should see:

```
N E X T F L O W  ~  version 20.04.1
Launching `prep_riboviz.nf` [fervent_volta] - revision: 6c6670470d
Validating configuration only
samples_dir: .
organisms_dir: .
data_dir: .
No such sample file (NotHere): example_missing_file.fastq.gz
Validated configuration
```

**Question: Does everyone see that? Does anyone have any problems?**

This shows the Nextflow version and a unique auto-generated name and identifier for this run of RiboViz. Yours will differ from mine. `samples_dir`, `organisms_dir`, `data_dir` can be ignored. These are only used if using environment variables to specify the locations of input directories.

The `No such sample file (NotHere)` error can be ignored since we deliberately include a missing sample file in the "vignette" configuration.

`Validated configuration` was printed even though one sample file could not be found. This is because the configuration specifies 3 sample files and could still successfully run if one of these is missing. If all 3 were missing then the validation would fail.

If the configuration was problematic, then we'd sort that and revalidate the configuration. As our configuration is fine, we can now run the workflow.

### Run workflow

We run the workflow as follows:

```console
$ nextflow run -ansi-log false prep_riboviz.nf -params-file vignette/vignette_config.yaml
```

This is the same invocation as previously, minus `--validate_only` and with an  `-ansi-log false` parameter which ensures that each invocation of a workflow step is displayed on a separate line.

All going well, you should see:

```
$ nextflow run -ansi-log false prep_riboviz.nf -params-file vignette/vignette_config.yaml
N E X T F L O W  ~  version 20.04.1
Launching `prep_riboviz.nf` [modest_shaw] - revision: 6c6670470d
samples_dir: .
organisms_dir: .
data_dir: .
No such sample file (NotHere): example_missing_file.fastq.gz
[7e/e093a6] Submitted process > cutAdapters (WT3AT)
[42/b1c9a2] Submitted process > buildIndicesrRNA (yeast_rRNA)
[3d/5bcc67] Submitted process > cutAdapters (WTnone)
[0f/0757fc] Submitted process > buildIndicesORF (YAL_CDS_w_250)
[e4/850912] Submitted process > createVizParamsConfigFile
[e9/5f318c] Submitted process > hisat2rRNA (WTnone)
[8d/1ecb5d] Submitted process > hisat2rRNA (WT3AT)
[f5/a34897] Submitted process > hisat2ORF (WTnone)
[11/a2c59f] Submitted process > trim5pMismatches (WTnone)
[7e/2579c7] Submitted process > samViewSort (WTnone)
[d2/68d6aa] Submitted process > outputBams (WTnone)
[53/5bee02] Submitted process > makeBedgraphs (WTnone)
[16/26d72f] Submitted process > bamToH5 (WTnone)
[2d/0330b5] Submitted process > hisat2ORF (WT3AT)
[02/5594ad] Submitted process > trim5pMismatches (WT3AT)
[91/3ad140] Submitted process > samViewSort (WT3AT)
[60/796bd2] Submitted process > outputBams (WT3AT)
[e2/254948] Submitted process > makeBedgraphs (WT3AT)
[14/154ad6] Submitted process > bamToH5 (WT3AT)
[e1/bebe80] Submitted process > generateStatsFigs (WTnone)
[b0/013c22] Submitted process > generateStatsFigs (WT3AT)
Finished processing sample: WTnone
[69/1f6794] Submitted process > renameTpms (WTnone)
[a1/b61f46] Submitted process > staticHTML (WTnone)
Finished processing sample: WT3AT
[a1/6eb17a] Submitted process > renameTpms (WT3AT)
[b0/5a6872] Submitted process > staticHTML (WT3AT)
[93/46c33b] Submitted process > collateTpms (WTnone, WT3AT)
[b8/11a56c] Submitted process > countReads
Finished visualising sample: WTnone
Finished visualising sample: WT3AT
Workflow finished! (OK)
```

The workflow first builds HISAT2 indices for the ORF and rRNA FASTA files. It then, for each sample, cuts out sequencing library adapters, removes rRNA or other contaminating reads by alignment to the rRNA index files, aligns the remaining reads to the ORF index files, trims 5' mismatches from the resulting reads and removes reads with more than 2 mismatches, exports bedgraph files for plus and minus strands, makes length-sensitive alignments, and calculates summary statistics, and analyses and quality control plots for both RPF and mRNA datasets including estimated read counts, reads per base, and transcripts per million for each ORF in each sample. These transcripts per million are then collated across all samples and the number of reads (sequences) processed by specific stages is calculated.

Each step in the workflow is implemented as a Nextflow process. Sample-independent processes are invoked once, for example `buildIndicesORF` and `buildIndicesrRNA`. Sample-specific processes, for example, `cutAdapters`, are invoked once per sample. Some invocations of processes are labelled with indices and sample names to make it clearer what that specific process is doing. For example, `collateTpms`, which aggregates data from all the samples, is labelled with all the sample identifiers.

Again, the workflow run has a unique name, for my run above this was `modest_shaw`. Yours will differ. Take a note of the run name as we'll be using this again shortly.

If we had not provided the `-ansi-log false` parameter then sample-specific steps would be combined within the display for a more succinct visualisation. For example we'd only see one `cutAdapters` step shown and this would show the status of running `cutadapt` on `WTnone` and then `WT3AT`, or vice versa. I prefer seeing each sample-specific step explicitly.

Each invocation of a process has a unique identifier specific to the current run of the workflow. This is the value delimited by the square brackets on the left. For example, `3d/5bcc67` for `cutAdapters (WTnone)`. Yours will differ from mine.

**Question: Does everyone see that? Does anyone have any problems?**

**Question: Does anyone have any questions about running riboviz at this time?**

**In your own time, feel free to rerun the workflow but without `-ansi-log false` to see the difference in the output.**

---

## Interpreting RiboViz outputs

Now that we've looked at how to configure and run RiboViz, let's look at its outputs. 

### Analysis outputs

The outputs from running the RiboViz workflow, the analysis outputs, are put in an output directory. This output directory is defined in the `dir_out` configuration parameter. For the "vignette", this is `vignette/output/`, so let's take a look:

```console
$ ls vignette/output/
read_counts.tsv  TPMs_collated.tsv  WT3AT  WTnone
$ ls vignette/output/WTnone/
3ntframe_bygene_filtered.tsv            pos_sp_rpf_norm_reads.pdf
3ntframe_bygene.tsv                     pos_sp_rpf_norm_reads.tsv
3ntframe_propbygene.pdf                 read_lengths.pdf
3nt_periodicity.pdf                     read_lengths.tsv
3nt_periodicity.tsv                     sequence_features.tsv
codon_ribodens_gathered.tsv             startcodon_ribogridbar.pdf
codon_ribodens.pdf                      startcodon_ribogrid.pdf
codon_ribodens.tsv                      tpms.tsv
features.pdf                            WTnone.bam
gene_position_length_counts_5start.tsv  WTnone.bam.bai
minus.bedgraph                          WTnone.h5
plus.bedgraph                           WTnone_output_report.html
pos_sp_nt_freq.tsv
```

There is one output files subdirectory for each sample, in addition to data files holding data about all the samples. These latter are `TPMs_collated.tsv` which holds the transcripts per million from each sample and `read_counts.tsv` which summarises the number of reads processed at each step in the workflow for each sample.

Let's take a look at the outputs for a single sample:

```console
$ ls vignette/output/WTnone/
3ntframe_bygene_filtered.tsv            pos_sp_rpf_norm_reads.pdf
3ntframe_bygene.tsv                     pos_sp_rpf_norm_reads.tsv
3ntframe_propbygene.pdf                 read_lengths.pdf
3nt_periodicity.pdf                     read_lengths.tsv
3nt_periodicity.tsv                     sequence_features.tsv
codon_ribodens_gathered.tsv             startcodon_ribogridbar.pdf
codon_ribodens.pdf                      startcodon_ribogrid.pdf
codon_ribodens.tsv                      tpms.tsv
features.pdf                            WTnone.bam
gene_position_length_counts_5start.tsv  WTnone.bam.bai
minus.bedgraph                          WTnone.h5
plus.bedgraph                           WTnone_output_report.html
pos_sp_nt_freq.tsv
```

There are myriad files with data from the processing of the sample. These include a BAM file of reads mapped to transcripts, which can be directly used in genome browsers, alignments within an HDF5 file, tab-separated values files that can be used in your own custom analyses and complemented by PDF files which contain graphs of this data.

However, we'll look at the HTML file, `WTnone_output_report.html`. This is a new feature for RiboViz 2.1 in which the outputs from the analysis of a sample are summarised as a browsable HTML document.

Let's open that document within a web browser and look at its contents.

**Question: Does anyone have any questions about the outputs?**

### Index and temporary files

The RiboViz configuration also specifies directories for index and temporary files, in `dir_index` and `dir_tmp`. These contain files produced during intermediate steps within the workflow and which may or may not be of interest. Let's take a look:

```console
$ ls vignette/index/
YAL_CDS_w_250.1.ht2  YAL_CDS_w_250.5.ht2  yeast_rRNA.1.ht2  yeast_rRNA.5.ht2
YAL_CDS_w_250.2.ht2  YAL_CDS_w_250.6.ht2  yeast_rRNA.2.ht2  yeast_rRNA.6.ht2
YAL_CDS_w_250.3.ht2  YAL_CDS_w_250.7.ht2  yeast_rRNA.3.ht2  yeast_rRNA.7.ht2
YAL_CDS_w_250.4.ht2  YAL_CDS_w_250.8.ht2  yeast_rRNA.4.ht2  yeast_rRNA.8.ht2
$ ls vignette/tmp/
WT3AT  WTnone
```

There is one temporary files subdirectory for each sample.

```
$ ls vignette/tmp/WTnone/
nonrRNA.fq             orf_map_clean.sam  trim_5p_mismatch.tsv
orf_map_clean.bam      orf_map.sam        trim.fq
orf_map_clean.bam.bai  rRNA_map.sam       unaligned.fq
```

These directories does not contain the files themselves, but, instead so-called symbolic links to these files which are actually located within a Nextflow-specific directory which I'll describe shortly. These symbolic links makes it appear that these files themselves are in these directories, and they can be used as if they are. This avoids the need to have multiple copies of these, sometimes very large, files within your file system.

If you want copies of, rather than symbolic links to, index and temporary files then there is a `publish_index_tmp` configuration parameter that allows you to request this.

The files in the workflow output directory, specified in the `dir_out` configuration parameter, are also located within a Nextflow-specific directory. However, these are not symbolic links but these are copies of those files, as you may want to keep these output files for later use.

**Question: Does anyone have any questions about these index and temporary files?**

---

## Troubleshooting a workflow

Our workflow successfully completed. If it had failed, then we'd want to troubleshoot as to why it failed. Nextflow logs data about each workflow that it runs and we can explore these logs, for both successful and failed workflows, using the `nextflow log` command.

For example, to see the instances of all the processes we run, we can use `nextflow log` with our workflow run name, that we noted down earlier, to show both process names and the names of their instances:

```console
$ nextflow log modest_shaw -f process,name
cutAdapters	cutAdapters (WT3AT)
buildIndicesrRNA	buildIndicesrRNA (yeast_rRNA)
cutAdapters	cutAdapters (WTnone)
buildIndicesORF	buildIndicesORF (YAL_CDS_w_250)
createVizParamsConfigFile	createVizParamsConfigFile
hisat2rRNA	hisat2rRNA (WTnone)
hisat2rRNA	hisat2rRNA (WT3AT)
hisat2ORF	hisat2ORF (WTnone)
trim5pMismatches	trim5pMismatches (WTnone)
samViewSort	samViewSort (WTnone)
outputBams	outputBams (WTnone)
makeBedgraphs	makeBedgraphs (WTnone)
bamToH5	bamToH5 (WTnone)
hisat2ORF	hisat2ORF (WT3AT)
trim5pMismatches	trim5pMismatches (WT3AT)
samViewSort	samViewSort (WT3AT)
outputBams	outputBams (WT3AT)
makeBedgraphs	makeBedgraphs (WT3AT)
bamToH5	bamToH5 (WT3AT)
generateStatsFigs	generateStatsFigs (WTnone)
generateStatsFigs	generateStatsFigs (WT3AT)
renameTpms	renameTpms (WTnone)
staticHTML	staticHTML (WTnone)
renameTpms	renameTpms (WT3AT)
staticHTML	staticHTML (WT3AT)
collateTpms	collateTpms (WTnone, WT3AT)
countReads	countReads
```

If we were only interested in the names of the invocations of the `cutAdapters` process, we'd run:

```console
$ nextflow log modest_shaw -f name -filter "process == 'cutAdapters'"
cutAdapters (WT3AT)
cutAdapters (WTnone)
```

We can find out information about the invocation of a specific process including the command that was run by Nextflow, and its exit code, as follows:

```console
$ nextflow log modest_shaw -f script,exit -filter "name == 'cutAdapters (WTnone)'"

        cutadapt --trim-n -O 1 -m 5 -a CTGTAGGCACC             -o trim.fq SRR1042855_s1mi.fastq.gz -j 0
        	0
```

Here we have specified that we want `script` and `exit` fields from the log relating to the invocation of a process that had the name `cutAdapters (WTnone)`. We are shown the command used to invoke the `cutadapt` tool with the parameters that were passed to it, and its exit code, which is 0 as `cutadapt` completed successfully.

We can also browse any output and error messages printed by `cutadapt` when it ran, by asking for the `stdout` and `stderr` fields from the log:

```console
$ nextflow log modest_shaw -f stdout,stderr -filter "name == 'cutAdapters (WTnone)'"
This is cutadapt 1.18 with Python 3.7.6Command line parameters: --trim-n -O 1 -m 5 -a CTGTAGGCACC -o trim.fq SRR1042855_s1mi.fastq.gz -j 0Processing reads on 4 cores in single-end mode ...Finished in 7.35 s (8 us/read; 7.87 M reads/minute).-
```

Only the first few lines of the output are shown. There were no error messages so `-` is displayed.

If our workflow failed and we wanted to see the names of the failed steps, we can filter on the `status` log field for the workflow:

```console
$ nextflow log modest_shaw -f name -filter "status == 'FAILED'"
```

As all our steps succeeded, no steps are returned. 

**In your own time, explore `nextflow log`. For more information see Nextflow [Tracing & visualisation](https://www.nextflow.io/docs/latest/tracing.html), `nextflow log -h` and to see the log fields available run `nextflow log -l`.**

### Resuming a failed workflow

If a workflow fails then we can use Nextflow's "resume" feature which allows the workflow to restart from where it failed. This means that the workflow doesn't redo steps that it successfully completed. Though our workflow successfully completed, we can see how this works by rerunning the workflow and adding Nextflow's `-resume` flag:

```console
$ nextflow run -ansi-log false prep_riboviz.nf -params-file vignette/vignette_config.yaml -resume
N E X T F L O W  ~  version 20.04.1
Launching `prep_riboviz.nf` [berserk_banach] - revision: 6c6670470d
samples_dir: .
organisms_dir: .
data_dir: .
No such sample file (NotHere): example_missing_file.fastq.gz
[3d/5bcc67] Cached process > cutAdapters (WTnone)
[7e/e093a6] Cached process > cutAdapters (WT3AT)
[42/b1c9a2] Cached process > buildIndicesrRNA (yeast_rRNA)
[0f/0757fc] Cached process > buildIndicesORF (YAL_CDS_w_250)
[e9/5f318c] Cached process > hisat2rRNA (WTnone)
[8d/1ecb5d] Cached process > hisat2rRNA (WT3AT)
[2d/0330b5] Cached process > hisat2ORF (WT3AT)
[f5/a34897] Cached process > hisat2ORF (WTnone)
[e4/850912] Cached process > createVizParamsConfigFile
[11/a2c59f] Cached process > trim5pMismatches (WTnone)
[02/5594ad] Cached process > trim5pMismatches (WT3AT)
[91/3ad140] Cached process > samViewSort (WT3AT)
[7e/2579c7] Cached process > samViewSort (WTnone)
[60/796bd2] Cached process > outputBams (WT3AT)
[d2/68d6aa] Cached process > outputBams (WTnone)
[e2/254948] Cached process > makeBedgraphs (WT3AT)
[53/5bee02] Cached process > makeBedgraphs (WTnone)
[16/26d72f] Cached process > bamToH5 (WTnone)
[14/154ad6] Cached process > bamToH5 (WT3AT)
[e1/bebe80] Cached process > generateStatsFigs (WTnone)
[b0/013c22] Cached process > generateStatsFigs (WT3AT)
Finished processing sample: WTnone
Finished processing sample: WT3AT
[69/1f6794] Cached process > renameTpms (WTnone)
[a1/6eb17a] Cached process > renameTpms (WT3AT)
[a1/b61f46] Cached process > staticHTML (WTnone)
Finished visualising sample: WTnone
[b0/5a6872] Cached process > staticHTML (WT3AT)
Finished visualising sample: WT3AT
[93/46c33b] Cached process > collateTpms (WTnone, WT3AT)
[b8/11a56c] Cached process > countReads
Workflow finished! (OK)
```

The invocation of each process has the same identifier as the previous run and the output now shows `Cached process`. This means that the workflow is reusing the outputs computed in the previous run. Similarly, if we were to add an additional sample file to the configuration and rerun the workflow in this way then only the steps relating to the additional sample would be run.

### Nextflow's files 

Now, what is this Nextflow-specific directory I've mentioned and where does `nextflow log` get its information from? When Nextflow runs, it creates a unique directory for every step in the workflow. This directory contains the input files to the step, the command to be invoked at that step and the output files from that step. These are created within a `work/` directory. Let's take a look:

```console
$ ls work
02  11  16  3d  53  69  8d  93  b0  d2  e2  e9
0f  14  2d  42  60  7e  91  a1  b8  e1  e4  f5
```

This may seem very cryptic at first. Now, recall, that the invocation of each process has a unique identifier, the value delimited by the square brackets and printed to the left of the process when the workflow is run. This identifier specifies a unique subdirectory within `work/`. However, these identifiers are not very usable, so we can use `nextflow log` to get the `work/` subdirectory:

```console
$ nextflow log modest_shaw -f hash,workdir -filter "name == 'cutAdapters (WTnone)'"
3d/5bcc67	/home/ubuntu/riboviz/work/3d/5bcc6780442d00c216c5c1c116ea90
```

This shows both the unique identifier, termed a "hash", of this step that was shown when we ran the workflow, and also the corresponding subdirectory within `work/` for that step. Note that the identifier is a prefix of the subdirectory under `work/`. We can now list its contents, including any hidden files:

```console
$ ls -1a /home/ubuntu/riboviz/work/3d/5bcc6780442d00c216c5c1c116ea90
.
..
.command.begin
.command.err
.command.lo
.command.out
.command.run
.command.sh
.exitcode
SRR1042855_s1mi.fastq.gz
trim.fq
```

This is the actual directory within which Nextflow ran `cutadapt` on sample `WTnone`'s FASTQ file. The directory does not contain the FASTQ file itself, but, instead a symbolic link to the FASTQ file, which is in the directory specified by the `dir_in` parameter. However, this symbolic link makes it appear that the file itself is in this `work/` subdirectory. Again, the use of symbolic links avoids the need to have multiple copies of files.

The output from applying `cutadapt` in the sample's FASTQ file, `trim.fq` is also in this directory.

`.command.sh` is a bash script with the invocation of `cutadapt` that Nextflow ran and `.exitcode` contains the exit code from the command. `.command.out` and `.command.err` contain any output or error messages printed when the above command was run. The contents of these files were what we saw when running `nextflow log` to access the `script`, `exit`, `stdout` and `stderr` log fields earlier. I mentioned earlier that only the first few lines of the output and error messages captured were displayed by `nextflow log`. If we need to see their entire contents we can view these, for example:

```console
$ cat /home/ubuntu/riboviz/work/3d/5bcc6780442d00c216c5c1c116ea90/.command.out 
This is cutadapt 1.18 with Python 3.7.6
Command line parameters: --trim-n -O 1 -m 5 -a CTGTAGGCACC -o trim.fq SRR1042855_s1mi.fastq.gz -j 0
Processing reads on 4 cores in single-end mode ...
Finished in 7.35 s (8 us/read; 7.87 M reads/minute).

=== Summary ===

Total reads processed:                 963,571
Reads with adapters:                   958,926 (99.5%)
Reads that were too short:              11,228 (1.2%
...
```

As the step-specific work directory contains everything that Nextflow used to run the command and as the command is in `.command.sh`, when troubleshooting, we can go into this directory and rerun the `.command.sh` bash script directly.

This `work/` directory is used by Nextflow to support its "resume" feature. If you delete the `work/` folder then the `-resume` flag has no effect and the whole workflow will be rerun.

**Question: Does anyone have any questions about the Nextflow `work/` directory?**

---

## Questions

**Question: Does anyone have any questions about what has been covered in this session?**

Full information on configuring and running RiboViz is available within our Git repository, [riboviz/riboviz](https://github.com/riboviz/riboviz), on GitHub. As I mentioned previously, to complement RiboViz, we provide a [riboviz/example-datasets](https://github.com/riboviz/example-datasets) repository on GitHub which is a library of configuration files produced by the RiboViz project that have been run on full-size datasets for various organisms. You are encouraged to explore these and try these out and contributions of configurations for new datasets are most welcome.

Thank you!
