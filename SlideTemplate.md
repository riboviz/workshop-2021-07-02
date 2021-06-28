---
title: "Experts session: Hands-on Ribo-Seq workflows - riboviz"
author: "riboviz Team"
date: "02/07/2021"
output:
  ioslides_presentation
---

<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.2/jquery.min.js"></script>

<script>
    $(document).ready(function() {
      $('slide:not(.title-slide, .backdrop, .segue)').append('<footer label=\"bit.ly/..."></footer>');
    })
</script>

<style>
  footer:after {
    content: attr(label);
    font-size: 12pt;
    position: absolute;
    bottom: 20px;
    left: 100px;
    line-height: 1.9;
  }
</style>

<style type="text/css">
slides > slide:not(.nobackground):after {
  content: '';
}
</style>

<div class="notes">

Bio:

Title: Experts session: Hands-on Ribo-Seq workflows - riboviz

Abstract:

Overview of riboviz workflow and features
Hands-on session to run riboviz on a small dataset
 - Run on vignette
 - Run on downsampled dataset
 - Finding and interpreting riboviz output
Overview of how to adapt to your own dataset
Question-and-answer session
Evaluation

</div>

# Hands-on Ribo-Seq workflows - riboviz



<div class="notes">

This is a template for individual slide sections.

</div>


## Goals of this session

We aim for you to:

* Understand what riboviz does and if it would work for your needs
* Succeed in running riboviz on small datasets
* Know where to start on getting riboviz to work on new data




# Overview of riboviz workflow and features


# Hands-on session to run riboviz on a small dataset

## Run on vignette  

<div class="notes">
This should show what it "looks like" for riboviz to run and take about 3 minutes.

We can flush out bugs or installation problems.

We can talk through what the nextflow log means, where the output goes, and what the html output means, very briefly.
</div>

## Run on downsampled dataset  

<div class="notes">
Hands-on approach to:
 - download the data + config file
 - ensure that prep_riboviz.nf can find these inputs, using --validate-only
 - run on that data, ideally takes under 5 minutes on a laptop.
 - as the data are running, Q&A on dataset setup and nextflow
 - refer to documentation, especially troubleshooting
</div>

## Finding and interpreting riboviz output  

<div class="notes">
 - talk through where the files are
 - show and talk through .html output from the new downsampled dataset.
 - have one example from another full-size dataset to compare?
</div>



# Overview of how to adapt to your own dataset

<div class="notes">
focus on key considerations: 
 - annotation files (inc. contaminants) - checkfastagff tool
 - the annotation file follows the biological question and must be chosen by the user
 - read structure (adapters, UMI/regex)
 - everything goes in config.yaml
refer to documentation for config.yaml
explain example-datasets repository
</div>



# Questions & Answer Session

<div class="notes">
... slide notes ...
</div>



# Workshop Evaluation

<div class="notes">
... slide notes ...
</div>