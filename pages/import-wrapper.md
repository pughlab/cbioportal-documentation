---
layout: default
---

### [Import Guide](import-guide.html)

# The import wrapper

The import wrapper is a command-line tool that helps package a set of pipeline data for easy import into cBioPortal. it creates all the various meta files, and makes sure that everything is packaged correctly. It also takes care of merging a large block of VCF files into a single MAF file with the correct columns for the portal.

It generates a complete directory suitable for safe and consistent use with the [import runner](import-runner.html).

## Installing the import wrapper

The import wrapper needs the following to be installed:

 * Perl -- we recommend 5.18 or newer
 * The Ensembl variant effect predictor with:
   - Cache files (e.g., `homo_sapiens_vep_83_GRCh37.tar.gz`)
   - Reference FASTA files (e.g., `Homo_sapiens.GRCh37.75.dna.primary_assembly.fa.gz`)
   - The ExAC plugin
   - Data required for the ExAC plugin (e.g., `ExAC.r0.3.sites.vep.vcf.gz`)
 * Recent versions of the following non-core CPAN modules:
   - `Parallel::ForkManager`
   - `Log::Log4perl`
   - `Config::Any`
   - `Hash::Merge::Simple`
   - `Text::CSV`
   - `VCF`

Note that you are not required to use the given reference FASTA file or VEP cache data. Because these can vary from deployment to deployment, many of these settings can be overridden. However, it is best to use a consistent reference genome and set of Perl modules if you can.

If you can't get these modules installed, we've used `perlbrew` to make a standalone Perl. This works well, although some of the Ensembl dependencies can require a little C compilation and dynamic library niftiness.

## Using the import wrapper

Using the script is very simple, because almost all the interesting information is set in a configuration file. To run the script:

```perl
perl import.pl --config <config.yml> --output <directory>
```

Note that the import wrapper doesn't overwrite anything that is already there, so if you have new input data, it's best to remove the output directory to ensure it actually gets processed.

## Configuration

All the configuration is done with YAML, because it's readable, commentable, and not XML.

There are several parts to the configuration:

 * [Sources](#sources)
 * [Cancer study](#cancer-study)
 * [Settings](#settings)

### <a name="sources"></a> Sources

Most of the genomic data comes in a variety of different sources. Many of these require some significant processing before they are used. A typical example are the VCF files from Mutect and Varscan. These are all re-annotated using the Ensembl VEP tool, and bundled into a monolithic MAF file for import. The import wrapper takes care of all of these steps. However, there are a few subtle points relating to sample and patient identifiers, which we will come to shortly.

A typical source definition looks like this:

```yaml
sources:
  exome:
    directory: '/mnt/work1/users/pughlab/projects/PJC003/Mutect_VCF/output/PASS'
    pattern: '(?i)\.vcf$'
    origin: 'mutect'
    sample_matcher: '(?i)([^_]+)_([A-Z]+)_(Tumor|Normal)'
    patient_generator: '$1'
    tumour_sample: '(?i)_Tumor$'
    normal_sample: '(?i)_Normal$'
```

This actually defines one source, using VCFs and analyzed by Mutect.

The fields here are interpreted as follows:

 * `directory` -- the directory where the VCFs will be found
 * `pattern` -- a regular expression which filters the files in that directory
 * `origin` -- which program generated the files, typically `mutect` or `varscan`
 * `sample_matcher` -- a regular expression which matches a sample identifier. Bracketed groups can be used to find parts of the identifier, and these can be substituted into the `patient_generator` field
 * `patient_generator` -- a string, using dollar strings, which builds a patient identifier from the values matched in the `sample_matcher`
 * `tumour_sample` -- a regular expression which tests whether a sample identifier is for a tumour sample or not
 * `normal_sample` -- a regular expression which tests whether a sample identifier is for a normal sample or not

### <a name="cancer-study"></a> Cancer study

A typical configuration for this looks like:

```yaml
cancer_study:
  identifier: 'impact_compact'
  name: 'IMPACT/COMPACT'
  description: 'test'
  type_of_cancer: 'mixed'
  groups: ''
  dedicated_color: 'Black'
  short_name: 'IMPACT/COMPACT'
```

The settings here are used in all the secondary meta files needed by the whole study, as well as in the main meta file. In particular, `cancer_study.identifier` is used as a root stable identifier, so that you don't usually need to worry about any other stable identifiers in the whole study.

### <a name="settings"></a> Settings

There are a moderate number of other settings which can be adjusted, and which might need to be set depending on your environment. Many of these are used to make sure the Ensembl variant effect predictor can be run correctly, as its annotation is essential to proper import.

Normally, we'd make these settings in a `defaults.yml` file which is used to fill in any default settings. So site-wide settings are best set here, rather than in the configuration for an individual study, where they'll just create an additional maintenance burden.

These settings include:

 * `vep_path` -- where is the Ensembl VEP installed
 * `vep_data` -- where is the Ensembl VEP reading its data from
 * `vep_dir_plugins ` -- what is the plugins directory for VEP
 * `ref_fasta` -- where is the reference genome FASTA file for VEP
 * `vep_forks` -- how much should we let VEP fork? (default is 4)
 * `max_processes` -- how many files should we process in parallel? (default is 4)
 * `no_vep_check_ref` -- if true, turns off reference cehcking in VEP (false by default)
