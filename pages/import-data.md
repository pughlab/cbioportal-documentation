---
layout: default
---

### [Import Guide](import-guide.html)

# Import data raw format

cBioPortal expects a very specific set of files for importing. Its directory structure looks something like this:

```
+ case_lists
  - cases_all.txt
  - cases_cnaseq.txt
  - cases_complete.txt
  - cases_log2CNA.txt
  - cases_mRNA.txt
- data_CNA
- data_clinical.txt
- data_expression_median.txt
- data_log2CNA.txt
- data_methylation_hm27.txt
- meta_CNA.txt
- meta_expression_median.txt
- meta_log2CNA.txt
- meta_methylation_hm27.txt
- meta_study.txt
```

We'll come back to the formats of the data files later. All the meta files, and the case lists, have a vaguely "MIME-like" key/value format.

However, we don't want you to worry about any of this nonsense, so at UHN we use a [wrapper tool](import-wrapper.html) which translates pipeline data into the right format.
