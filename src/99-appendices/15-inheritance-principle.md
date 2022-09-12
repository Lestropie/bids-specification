# Appendix XV: The Inheritance Principle

The [Inheritance Principle](#inheritance-principle) specifies how it is
not strictly necessary to have data and metadata files with identical names
apart from the [file extension](#definitions).
It is possible to have a single metadata file with its contents applicable
to multiple data files;
it is additionally possible to have one data file where there are multiple
metadata files from which metadata are applicable.
This Appendix provides the precise definition of how this principle operates
and the consequences of such,
as reference for both software implementations and advanced BIDS users.

## Rules

1.  Any metadata file (such as `.json`, `.bvec` or `.tsv`) MAY be defined at any directory level.

1.  For a given data file, any metadata file is applicable to that data file if:
    1.  It is stored at the same directory level or higher;
    1.  The metadata and the data filenames possess the same suffix;
    1.  The metadata filename does not include any entity absent from the data filename.

1.  A metadata file MUST NOT have a filename that would be otherwise applicable
    to some data file based on rules 2.b and 2.c but is made inapplicable based on its
    location in the directory structure as per rule 2.a.

1.  If, for a given data file, multiple metadata files satisfy criteria 2.a-c above:

    1.  The set of applicable metadata files is ordered as follows.
        Within each level of the filesystem hierarchy independently,
        applicable files are ordered from fewest to most entities.
        These lists are concatenated in order of highest to lowest level in the
        filesystem hierarchy.

    1.  For [tabular files](#tabular-files) and other simple metadata files
        (for instance, [`bvec` / `bval` files for diffusion MRI](#bvec-bval):

        1.  Accessing metadata associated with a data file MUST consider only the
            last file in the order established by rule 4.a.

        1.  There MUST NOT be any ambiguity in determining this file via the
            sorting described in rule 4.a.

    1.  For [JSON files](#json-files):

        1.  Key-values MUST be loaded from applicable files sequentially in the
            order established by rule 4.a,
            overwriting any existing key-values when doing so.

        1.  Where multiple metadata files appear in the same location in the list
            sorted as per rule 4.a, a metadata key MUST NOT appear in more than
            one such metadata file.

### Corollaries

1.  As per rule 3, metadata files applicable only to a specific participant / session
    MUST be defined in or below the directory corresponding to that participant / session;
    similarly, a metadata file that is applicable to multiple participants / sessions
    MUST NOT be placed within a directory corresponding to only one such participant / session.

1.  It is permissible for a single metadata file to be applicable to multiple data files.
    Where such metadata content is consistent across multiple data files,
    it is RECOMMENDED to store metadata in this way,
    rather than duplicating that metadata content across multiple metadata files.

1.  Where multiple applicable [JSON files](#json-files) are loaded
    for one data file as per rules 4.a and 4.c:

    1.  Where the same key is present in multiple applicable metadata files,
        the final value associated with that key will be that of the file latest in
        the order in which that key is defined;
        any values associated with that key earlier in the ordering are overridden
        (though it is RECOMMENDED to minimize the extent of such overrides).

    1.  A key-value being present in a metadata file earlier in the ordering but absent in
        any file later in the ordering does not imply the "unsetting" of that key-value.

    1.  Removal of key-values present in files earlier in the ordering based on the content
        of files later in the ordering is not possible.

## Complex inheritance scenario

<!-- This block generates a file tree.
A guide for using macros can be found at
 https://github.com/bids-standard/bids-specification/blob/master/macros_doc.md
-->

{{ MACROS___make_filetree_example(
    {
    "bold.json": "",
    "sub-01": {
        "ses-01": {
            "func": {
                "sub-01_ses-01_bold.json": "",
                "sub-01_ses-01_task-ovg_bold.json": "",
                "sub-01_ses-01_task-ovg_run-1_bold.nii.gz": "",
                "sub-01_ses-01_task-ovg_run-2_bold.nii.gz": "",
                "sub-01_ses-01_task-ovg_run-2_bold.json": "",
                "sub-01_ses-01_task-rest_bold.nii.gz": "",
                "sub-01_ses-01_task-rest_bold.json": "",
                }
            },
        "ses-02": {
            "func": {
                "sub-01_ses-02_task-ovg_bold.nii.gz": "",
                "sub-01_ses-02_task-rest_bold.nii.gz": "",
                }
            },
        "sub-01_bold.json": "",
        },
    "sub-02": {
        "ses-01": {
            "func": {
                "sub-02_ses-01_res-high_task-olr_bold.nii.gz": "",
                "sub-02_ses-01_res-high_task-rest_bold.nii.gz": "",
                "sub-02_ses-01_res-high_bold.json": "",
                "sub-02_ses-01_res-low_task-olr_bold.nii.gz": "",
                "sub-02_ses-01_res-low_task-rest_bold.nii.gz": "",
                "sub-02_ses-01_res-low_bold.json": "",
                "sub-02_ses-01_task-olr_bold.json": "",
                "sub-02_ses-01_task-rest_bold.json": "",
                }
            }
        },
    "task-olr_bold.json": "",
    "task-ovg_bold.json": "",
    "task-rest_bold.json": "",
    }
) }}

### Applicable data files per metadata file

For each metadata file, the set of data files to which its contents are
applicable is as follows:

-   `bold.json`:
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-1_bold.nii.gz`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-2_bold.nii.gz`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-rest_bold.nii.gz`
    -   `sub-01/ses-02/func/sub-01_ses-02_task-ovg_bold.nii.gz`
    -   `sub-01/ses-02/func/sub-01_ses-02_task-rest_bold.nii.gz`
    -   `sub-02/ses-01/func/sub-02_ses-01_task-rest_bold.nii.gz`

-   `task-olr_bold.json`:
    -   `sub-02/ses-01/func/sub-02_ses-01_res-high_task-olr_bold.nii.gz`
    -   `sub-02/ses-01/func/sub-02_ses-01_res-low_task-olr_bold.nii.gz`

-   `task-ovg_bold.json`:
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-1_bold.nii.gz`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-2_bold.nii.gz`
    -   `sub-01/ses-02/func/sub-01_ses-02_task-ovg_bold.nii.gz`

-   `task-rest_bold.json`:
    -   `sub-01/ses-01/func/sub-01_ses-01_task-rest_bold.nii.gz`
    -   `sub-01/ses-02/func/sub-01_ses-02_task-rest_bold.nii.gz`
    -   `sub-02/ses-01/func/sub-02_ses-01_res-high_task-rest_bold.nii.gz`
    -   `sub-02/ses-01/func/sub-02_ses-01_res-low_task-rest_bold.nii.gz`

-   `sub-01/sub-01_bold.json`:
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-1_bold.nii.gz`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-2_bold.nii.gz`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-rest_bold.nii.gz`
    -   `sub-01/ses-02/func/sub-01_ses-02_task-ovg_bold.nii.gz`
    -   `sub-01/ses-02/func/sub-01_ses-02_task-rest_bold.nii.gz`

-   `sub-01/ses-01/sub-01_ses-01_bold.json`:
    -   `sub-01/ses-01/sub-01_ses-01_task-ovg_run-1_bold.nii.gz`
    -   `sub-01/ses-01/sub-01_ses-01_task-ovg_run-2_bold.nii.gz`
    -   `sub-01/ses-01/sub-01_ses-01_task-rest_bold.nii.gz`

-   `sub-01/ses-01/sub-01_ses-01_task-ovg_bold.json`:
    -   `sub-01/ses-01/sub-01_ses-01_task-ovg_run-1_bold.nii.gz`
    -   `sub-01/ses-01/sub-01_ses-01_task-ovg_run-2_bold.nii.gz`

-   `sub-01/ses-01/sub-01_ses-01_task-ovg_run-2_bold.json`:
    -   `sub-01/ses-01/sub-01_ses-01_task-ovg_run-2_bold.nii.gz`

-   `sub-01/ses-01/sub-01_ses-01_task-rest_bold.json`:
    -   `sub-01/ses-01/sub-01_ses-01_task-rest_bold.nii.gz`

-   `sub-02/ses-01/sub-02_ses-01_res-high_bold.json`:
    -   `sub-02/ses-01/sub-02_ses-01_res-high_task-olr_bold.nii.gz`
    -   `sub-02/ses-01/sub-02_ses-01_res-high_task-rest_bold.nii.gz`

-   `sub-02/ses-01/sub-02_ses-01_res-low_bold.json`:
    -   `sub-02/ses-01/sub-02_ses-01_res-low_task-olr_bold.nii.gz`
    -   `sub-02/ses-01/sub-02_ses-01_res-low_task-rest_bold.nii.gz`

-   `sub-02/ses-01/sub-02_ses-01_task-olr_bold.json`:
    -   `sub-02/ses-01/sub-02_ses-01_res-high_task-olr_bold.nii.gz`
    -   `sub-02/ses-01/sub-02_ses-01_res-low_task-olr_bold.nii.gz`

-   `sub-02/ses-01/sub-02_ses-01_task-rest_bold.json`:
    -   `sub-02/ses-01/sub-02_ses-01_res-high_task-rest_bold.nii.gz`
    -   `sub-02/ses-01/sub-02_ses-01_res-low_task-rest_bold.nii.gz`

### Applicable metadata files per data file

For each data file, the order in which the set of applicable metadata
files would be loaded is as follows:

-   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-1_bold.nii.gz`:
    -   `bold.json`
    -   `task-ovg_bold.json`
    -   `sub-01/sub-01_bold.json`
    -   `sub-01/ses-01/func/sub-01_ses-01_bold.json`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_bold.json`

-   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-2_bold.nii.gz`:
    -   `bold.json`
    -   `task-ovg_bold.json`
    -   `sub-01/sub-01_bold.json`
    -   `sub-01/ses-01/func/sub-01_ses-01_bold.json`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_bold.json`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-ovg_run-2_bold.json`

-   `sub-01/ses-01/func/sub-01_ses-01_task-rest_bold.nii.gz`:
    -   `bold.json`
    -   `task-rest_bold.json`
    -   `sub-01/sub-01_bold.json`
    -   `sub-01/ses-01/func/sub-01_ses-01_bold.json`
    -   `sub-01/ses-01/func/sub-01_ses-01_task-rest_bold.json`

-   `sub-01/ses-02/func/sub-01_ses-02_task-ovg_bold.nii.gz`:
    -   `bold.json`
    -   `task-ovg_bold.json`
    -   `sub-01/sub-01_bold.json`

-   `sub-01/ses-02/func/sub-01_ses-02_task-rest_bold.nii.gz`:
    -   `bold.json`
    -   `task-rest_bold.json`
    -   `sub-01/sub-01_bold.json`

-   `sub-02_ses-01_res-high_task-olr_bold.nii.gz`:
    -   `bold.json`
    -   `task-olr_bold.json`
    -   `sub-02/ses-01/func/sub-02_ses-01_res-high_bold.json`
        and `sub-02/ses-01/func/sub-02_ses-01_task-olr_bold.json` (ambiguous order)

-   `sub-02_ses-01_res-high_task-rest_bold.nii.gz`:
    -   `bold.json`
    -   `task-rest_bold.json`
    -   `sub-02/ses-01/func/sub-02_ses-01_res-high_bold.json`
        and `sub-02/ses-01/func/sub-02_ses-01_task-rest_bold.json` (ambiguous order)

-   `sub-02_ses-01_res-low_task-olr_bold.nii.gz`:
    -   `bold.json`
    -   `task-olr_bold.json`
    -   `sub-02/ses-01/func/sub-02_ses-01_res-low_bold.json`
        and `sub-02/ses-01/func/sub-02_ses-01_task-olr_bold.json` (ambiguous order)

-   `sub-02_ses-01_res-low_task-rest_bold.nii.gz`:
    -   `bold.json`
    -   `task-rest_bold.json`
    -   `sub-02/ses-01/func/sub-02_ses-01_res-low_bold.json`
        and `sub-02/ses-01/func/sub-02_ses-01_task-rest_bold.json` (ambiguous order)

### Principle violation due to key precedence ambiguity

Consider the inheritance of file `sub-02_ses-01_res-high_task-olr_bold.nii.gz`,
where the contents of the applicable metadata files are as follows:

Contents of file `bold.json`:
```
{
    "Key": "One"
}
```

Contents of file `task-rest_bold.json`:
```
{
    "Key": "Two"
}
```

Contents of file `sub-02/ses-01/func/sub-02_ses-01_res-high_bold.json`:
```
{
    "Key": "Three"
}
```

Contents of file `sub-02/ses-01/func/sub-02_ses-01_task-olr_bold.json`:
```
{
    "Key": "Four"
}
```

In loading inherited metadata for this file as per rule 4.c.i.,
the value associated with key "`Key`" would be initialised as "`One`" as
loaded from file `bold.json`,
then overridden to "`Two`" as loaded from file `task-rest_bold.json`.
However it is not possible as per rule 4.a. to unambiguously sort applicable metadata
files `sub-02/ses-01/func/sub-02_ses-01_res-high_bold.json` and
`sub-02/ses-01/func/sub-02_ses-01_task-olr_bold.json`,
as they reside at the same level of the filesystem hierarchy
and possess the same number of entities.
As these two files additionally possess values for the same metadata key "`Key`",
it is therefore impossible to determine whether the final value of key "`Key`" ascribed
to file `sub-02_ses-01_res-high_task-olr_bold.nii.gz` should be "`Three`" or "`Four`".
It is specifically this conflict that results in Inheritance Principle violation
as per rule 4.c.ii.

<!-- Link Definitions -->

[bvec-bval]: 04-modality-specific-files/01-magnetic-resonance-imaging#required-gradient-orientation-information

[definitions]: 02-common-principles.md#definitions

[inheritance-principle]: 02-common-principles.md#the-inheritance-principle

[json-files]: 02-common-principles.md#key-value-files-dictionaries

[tabular-files]: 02-common-principles.md#tabular-files
