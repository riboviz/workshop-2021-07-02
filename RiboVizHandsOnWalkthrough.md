# Hands-on session to run RiboViz on a small dataset

Experts session: Hands-on Ribo-Seq workflows - RiboViz

Ribosome Profiling Workshop

2nd July 2021.

Mike Jackson

---

## Run on "vignette"

I'll start this hands-on session by leading you through running RiboViz on the "vignette", the downsampled *Saccharomyces cerevisiae* (yeast) dataset that is provided with RiboViz.

As I do this I'll give an introduction to how RiboViz is configured, how it is run, and the outputs it produces. This also provides an opportunity to resolve any last minute issues you may have with your local installation of RiboViz.

We'll be using the current version of RiboViz in our Git repository, [riboviz/riboviz](https://github.com/riboviz/riboviz) on GitHub. This is release 2.1 of RiboViz in all but name (we have some final tests to do before the official release).

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

**Question: Does anyone have any questions about the configuration?**

### RiboViz directory

Now, let's look at the the `riboviz` directory itself.

```console
$ ls
bash  docs  LICENSE          py-docs    riboviz    rscripts  work
data  jobs  prep_riboviz.nf  README.md  rmarkdown  vignette
```

Of note, we have our Python scripts, R and Rmarkdown scripts which implement specific analyses steps in the RiboViz workflow. These are in `riboviz`, `rscripts` and `rmarkdown` respectively. Within the workflow, these scripts are complemented by the use of third-party tools such as `hisat2-build`, `cutadapt`, `hisat2`, `samtools`, `bedtools` and `umi_tools`.

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

`nextflow run` is the command to run a Nextflow workflow, in this case our RiboViz workflow, `prep_riboviz.nf`. `-params-file` is the location of our RiboViz configuration file. `--validate_only` says that the workflow should only validate the configuration and then exit. Take care that you provide an underscore (`_`) not a dash (`-`) within the RiboViz parameter name. Unlike Nextflow's own paramters (which include dashes) custom parameters, such as those supported by RiboViz, need to underscores if provided via the command-line.

All going well, you should see:

```
N E X T F L O W  ~  version 20.04.1
Launching `prep_riboviz.nf` [nauseous_boyd] - revision: 6c6670470d
Validating configuration only
samples_dir: .
organisms_dir: .
data_dir: .
No such sample file (NotHere): example_missing_file.fastq.gz
Validated configuration
```

**Question: Does everyone see that? Does anyone have any problems?**

This shows the Nextflow version and a unique name and identifier for this run of RiboViz. `samples_dir`, `organisms_dir`, `data_dir` can be ignored. These are only used if using environment variables to specify the locations of input directories.

The `No such sample file (NotHere)` error can be ignored since we deliberately include a missing sample file in the vignette configuration.

`Validated configuration` was printed even though one sample file could not be found. This is because the configuration specifies 3 sample files and could still successfully run if one of these is missing. If all 3 were missing then the validation would fail.

If the configuration was problematic, then we'd sort that and revalidate the configuration. As our configuration is fine, we can now run the workflow.

### Run workflow

We run the workflow as follows:

```console
$ nextflow run -ansi-log false prep_riboviz.nf -params-file vignette/vignette_config.yaml
```

This is the same invocation as previously, minus `--validate_only` and with an  `-ansi-log false` parameter which I'll explain shortly.

All going well, you should see:

```
N E X T F L O W  ~  version 20.04.1
Launching `prep_riboviz.nf` [pedantic_woese] - revision: 6c6670470d
samples_dir: .
organisms_dir: .
data_dir: .
No such sample file (NotHere): example_missing_file.fastq.gz
[33/5699e6] Submitted process > buildIndicesORF (YAL_CDS_w_250)
[80/b8d009] Submitted process > cutAdapters (WTnone)
[c0/8d2098] Submitted process > buildIndicesrRNA (yeast_rRNA)
[43/780ea7] Submitted process > cutAdapters (WT3AT)
[3f/00dfe9] Submitted process > createVizParamsConfigFile
[65/1316d1] Submitted process > hisat2rRNA (WTnone)
[f1/e44315] Submitted process > hisat2rRNA (WT3AT)
[23/5707fa] Submitted process > hisat2ORF (WTnone)
[c6/756098] Submitted process > trim5pMismatches (WTnone)
[99/28641d] Submitted process > samViewSort (WTnone)
[28/8963de] Submitted process > outputBams (WTnone)
[95/797add] Submitted process > makeBedgraphs (WTnone)
[ee/7c540d] Submitted process > bamToH5 (WTnone)
[92/e5f8c2] Submitted process > hisat2ORF (WT3AT)
[d1/806db1] Submitted process > trim5pMismatches (WT3AT)
[c6/b2b184] Submitted process > samViewSort (WT3AT)
[03/2398b5] Submitted process > outputBams (WT3AT)
[44/f3b268] Submitted process > makeBedgraphs (WT3AT)
[bd/2163d2] Submitted process > bamToH5 (WT3AT)
[0e/d6d03c] Submitted process > generateStatsFigs (WTnone)
[1a/8bcebe] Submitted process > generateStatsFigs (WT3AT)
Finished processing sample: WTnone
[7b/e1ff20] Submitted process > renameTpms (WTnone)
[05/2c0276] Submitted process > staticHTML (WTnone)
Finished processing sample: WT3AT
[d1/712205] Submitted process > renameTpms (WT3AT)
[96/3b622b] Submitted process > staticHTML (WT3AT)
[6c/8aa8a0] Submitted process > collateTpms (WTnone, WT3AT)
[ac/7f7333] Submitted process > countReads
Finished visualising sample: WTnone
Finished visualising sample: WT3AT
Workflow finished! (OK)
```

The workflow first builds HISAT2 indices for the ORF and rRNA FASTA files. It then, for each sample, cuts out sequencing library adapters, removes rRNA or other contaminating reads by alignment to the rRNA index files, aligns the remaining reads to the ORF index files, trims 5' mismatches from the resulting reads and removes reads with more than 2 mismatches, exports bedgraph files for plus and minus strands, makes length-sensitive alignments, and calculates summary statistics, and analyses and quality control plots for both RPF and mRNA datasets including estimated read counts, reads per base, and transcripts per million for each ORF in each sample. These transcripts per million are then collated across all samples and the number of reads (sequences) processed by specific stages is calculated.

Your process identifiers, for example, `33/5699e6`, will differ from mine.

Each process corresponds to a single step in the workflow. Some of these are sample-independent, for example `buildIndicesORF` and `buildIndicesrRNA`. Many of these are sample-specific and are labelled with the sample identifier, for example, `cutAdapters (WTnone)` and `cutAdapters (WT3AT)`. `collateTpms` which aggregates data from all the samples is labelled with all the sample identifiers.

If we had not provided the `-ansi-log false` parameter then sample-specific steps would be combined within the display for a more succinct visualisation. For example we'd only see one `cutAdapters` step shown and this would show the status of running `cutadapt` on `WTnone` and then `WT3AT`, or vice versa. I prefer seeing each sample-specific step explicitly.

**In your own time, feel free to rerun the workflow but without `-ansi-log false` to see the difference in the output.**

Each process has a unique identifier specific to the current run of the workflow. This is the value delimited by the square brackets on the left. We'll revisit this shortly.

**Question: Does everyone see that? Does anyone have any problems?**

**Question: Does anyone have any questions about running riboviz at this time?**

If a workflow fails then Nextflow supports a `-resume` feature which allows the workflow to restart from where it failed. This means that the workflow doesn't redo steps that it successfully completed. Though our workflow successfully completed, we can see how this works by rerunning the workflow and adding Nextflow's `-resume` flag:

```console
$ nextflow run -ansi-log false prep_riboviz.nf -params-file vignette/vignette_config.yaml -resume
```

Each process has the same identifier as the previous run and also that the output shows `Cached process`. This means that the workflow is reusing the outputs computed for each step. Similarly, if we were to add an additional sample file to the configuration and rerun the workflow in this way then only the steps relating to the additional sample would be run. We'll see how this is possible shortly.

---

## Interpreting RiboViz outputs

Now that we've looked at how to configure and run RiboViz, let's look at its outputs. 

### Analysis outputs

The outputs from running the RiboViz workflow, the analysis outputs, are put in an output directory. This output directory is defined in the `dir_out` configuration parameter. For the "vignette", this is `vignette/output/`, so let's take a look:

```console
$ ls vignette/output
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

There is one output files subdirectory for each sample plus data files holding data from the processing of all samples. These latter are `TPMs_collated.tsv` which holds the transcripts per million from each sample and `read_counts.tsv` which summarises the number of reads processed at each step in the workflow for each sample.

Let's take a look at the outputs for a single sample:

```console
$ ls vignette/output/WTnone
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

There are myriad files with data from the processing of the sample. These include a BAM file of reads mapped to transcripts, which can be directly used in genome browsers, alignments within an HDF5 file, tab-separated values files that can be used in your own custom analyses and complementary PDF files which contain graphs of this data.

However, we'll look at the HTML file. `WTnone_output_report.html`. This is a new feature for RiboViz 2.1 by which the outputs from the analysis are summarised as a browsable HTML document.

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

These directories does not contain the files themselves, but, instead so-called symbolic links to these files which are actually located within a Nextflow-specific directory which I'll describe in a moment. However, these symbolic links makes it appear that these files themselves are in these directories, and they can be used as if they are, and avoids the need to have multiple copies of the files within your file system.

**Question: Does anyone have any questions about these index and temporary files?**

### Nextflow files 

Now, what is this Nextflow-specific directory I just mentioned? When Nextflow runs it creates a unique directory for every step in the workflow. This directory contains the input files to the step, the command to be invoked at that step and the output files from that step. These are created within a `work/` directory. Let's take a look:

```console
$ ls work
```

This may seem cryptic at first. Now, recall, that each process has a unique identifier, the value delimited by the square brackets and printed to the left of the process when the workflow is run. For example `[80/b8d009]` for `Submitted process > cutAdapters (WTnone)`. This identifier specifies a unique subdirectory within `work/`. For example:

```console
$ ls -a1 work/80/b8d009c7f3d27ac0b56c2a5a327d72/
.
..
.command.begin
.command.err
.command.log
.command.out
.command.run
.command.sh
.exitcode
SRR1042855_s1mi.fastq.gz
trim.fq
```

This is the directory within which Nextflow runs `cutadapt` on sample `WTnone`'s FASTQ file. The directory does not contain the FASTQ file itself, but, instead a symbolic link to the FASTQ file, which is in the directory specified by the `dir_in` parameter. However, this symbolic link makes it appear that the file itself is in this `work/` subdirectory. Again, the use of symbolic links avoids the need to have multiple copies of the files.

The output from applying `cutadapt` in the sample's FASTQ file, `trim.fq` is also in this directory.

`.command.sh` contains the actual command that Nextflow ran:

```console
$ cat work/80/b8d009c7f3d27ac0b56c2a5a327d72/.command.sh 
#!/bin/bash -ue
cutadapt --trim-n -O 1 -m 5 -a CTGTAGGCACC             -o trim.fq SRR1042855_s1mi.fastq.gz -j 0
```

`.exitcode` contains the exit code from the command:

```console
$ cat work/80/b8d009c7f3d27ac0b56c2a5a327d72/.exitcode 
0
```

`.command.log` contains any output or error messages printed when the above command was run:

```console
$ cat work/80/b8d009c7f3d27ac0b56c2a5a327d72/.command.log
```

This information can be very useful for debugging. You can also go into that directory and run the `.command.sh` script directly.

These files are also used when restarting a workflow using the `-resume` option. If you delete the `work/` folder then `-resume` has no effect and the whole workflow will be rerun.

**Question: Does anyone have any questions about the Nextflow `work/` directory?**

---

## Run on downsampled dataset

The "vignette" dataset is provided with RiboViz. We'll now look at preparing another dataset to be analysed by RiboViz. This time we'll download another downsampled dataset and a configuration file, validate our configuration and run RiboViz.

TODO:

* Download data and config file.
* Set up directories.
* Ensure that `prep_riboviz.nf` can find these inputs, using `--validate-only`:
* Run, ideally takes under 5 minutes on a laptop.
* Refer to documentation, especially troubleshooting

```console
$ nextflow run prep_riboviz.nf -params-file <TODO>.yaml --validate_only
$ nextflow run -ansi-log false prep_riboviz.nf -params-file <TODO>.yaml
```

**Question: Does anyone have any questions about preparing the data, configuring RiboViz or running the workflow?**


---

## Interpreting RiboViz outputs revisited

TODO:

* Show and talk through `.html` output from the downsampled dataset.
* Have one example from another full size dataset to compare?

**Question: Does anyone have any questions about the outputs?**

---

## Questions on running RiboViz

**Question: Does anyone have any questions on running RiboViz?**

Full information on configuring and running RiboViz is available at [riboviz/riboviz](https://github.com/riboviz/riboviz) on GitHub. We have a [riboviz/example-datasets](https://github.com/riboviz/example-datasets) repository which is library of pre-existing configuration files produced by the RiboViz project.

Thank you!
