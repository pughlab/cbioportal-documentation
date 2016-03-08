---
layout: default
---

### [Import Guide](import-guide.html)

# Annotated example import file

This shows an example configuration file for the [import wrapper](import-wrapper.html).

First, we start with a YAML document file header.

    ---

Next, we usually set up a set of sources. Sources are the places where we gather data from, but they do a little more than this. Some sources might add a sample-specific clinical attribute (a directory of files might have come from a particular sample source, for example). We might also want to do some filtering for particular files. And in almost every case, we need a way to derive a patient identifier from a sample identifier, and to distinguish between tumour and normal samples.

Here's an example source for some exome data:

    sources:
      exome:
        directory: '/tmp/study/MutectVCF/output/PASS'
        pattern: '(?i)\.vcf$'

The `directory` is fairly obvious, and the `pattern` just provides a filter on files within that directory. In this case, we only process files that end with `.vcf` (case insensitively, using the Perl `(?i)` prefix regular expression flag) from the given directory.

        origin: 'mutect'

Specifies that these VCF files come from Mutect.

        attributes:
          SEQUENCING_TYPE: 'EXOME'

This adds a sample attribute with the name `SEQUENCING_TYPE` and the value `EXOME`.

        sample_matcher: '(?i)([^_]+)_(Tumor|Normal)'

Next, each VCF file is parsed for the sample names. We're expecting the sample names to match (this is specified by the `sample_matcher` setting) the Perl regular expression `(?i)([^_]+)_(Tumor|Normal)`, and if they don't, we'll throw an error. Note that this also captures some bracketed groups. We'll use these in a moment.

        patient_generator: '$1'

For these, the patient identifier is constructed using the setting `patient_generator`, and in this case consists only of the value from the first bracketed group. But you add prefixes and do more complex substitutions here if you need to.

        tumour_sample: '(?i)_Tumor$'
        normal_sample: '(?i)_Normal$'

These two settings are used to detect tumour and normal sample identifiers, and decide which is which. A sample identifier which matches the `tumour_sample` will be classed as a tumour sample, and one which matches `normal_sample` will be a normal sample.

      targeted:
        directory: '/tmp/study/Target/MutectVCF/output/PASS'
        pattern: '(?i)\.vcf$'
        origin: 'mutect'
        attributes:
          SEQUENCING_TYPE: 'TARGETED'
        sample_matcher: '(?i)Target_([^_]+)_(Tumor|Normal)'
        patient_generator: '$1'
        tumour_sample: '(?i)_Tumor$'
        normal_sample: '(?i)_Normal$'

This is a second source, which uses slightly different sample naming conventions and sets a different clinical attribute.

    additional_clinical_attributes:
      - name: 'SEQUENCING_TYPE'
        description: 'Sequencing Type'
        type: 'STRING'
        label: 'SAMPLE'
        header: 'SEQUENCING_TYPE'
        count: 1

We need to declare some information about any additional clinical attributes we use, such as the `SEQUENCING_TYPE` one we used above. Each attribute needs a:

 * a `name`
 * a `description`
 * a `type` (typically `STRING` or `NUMBER`)
 * a `label` (which *must* be either `PATIENT` or `SAMPLE` and is used to decide whether it's a patient-specific or a sample-specific value)
 * a `header` (for the import column name)
 * a `count` (almost always one)

<!-- end the list -->

    clinical_file: ''

A pointer to a clinical data file, or the empty string if there isn't one.

    cancer_study:
      identifier: 'study'
      name: 'STUDY'
      description: 'STUDY'
      type_of_cancer: 'mixed'
      groups: ''
      dedicated_color: 'Green'
      short_name: 'STUDY'

General values for the whole cancer study. This is used to make all the stable identifiers and the metadata for all different data files.

Next, we add some case lists. These allow sets of samples to be explored through the web front end.

    case_lists:
      all:
        name: 'All'
        description: 'All exome and targeted'
        data:
          union:
            - 'exome'
            - 'targeted'

Our first case set consists of all the samples. The settings are as follows:

 * the `name` of the case set
 * the `description` for the case set
 * the `data` -- this can either be a source name (the key from the `sources` property above) or, as in this case, an object with a `union` name, which then merges samples from several sources.

 <!-- end the list -->

      exome:
        name: 'Exome'
        description: 'Exome'
        data: 'exome'

This is a simpler case set, consisting only of the exome samples.

      targeted:
        name: 'Targeted'
        description: 'Targeted'
        data: 'targeted'

And these are only the targeted samples.

If you use this to analyze all the data, you'll get a complete data package that is easy to load into cBioPortal at UHN.
