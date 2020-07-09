&nbsp;
# Specification of the GSuite file format
&nbsp;

- **GSuite version:** 0.9
- **Document version:** 1.0a
- **Date:** May 27, 2015
- **Authors:** Sveinung Gundersen, Boris Simovski, Abdulrahman Azab, Diana
  Domanska, Eivind Hovig, and Geir Kjetil Sandve


## Contents
&nbsp;

- [A. Introduction and background](#a-introduction-and-background)
  - [A.1 Overview](#a1-overview)
  - [A.2 Suites of tracks](#a2-suites-of-tracks)
  - [A.3 Location of tracks](#a3-location-of-tracks)
  - [A.4 Preprocessing of tracks into a binary format (BTrack)](#a4-preprocessing-of-tracks-into-a-binary-format-btrack)
  - [A.5 Track types](#a5-track-types)
- [B. Example GSuite files](#b-example-gsuite-files)
  - [B.1 Example 1](#b1-example-1)
  - [B.2 Example 2](#b2-example-2)
  - [B.3 Example 3](#b3-example-3)
- [C. Syntax of the GSuite format](#c-syntax-of-the-gsuite-format)
  - [C.i Empty lines](#ci-empty-lines)
  - [C.ii Comment lines](#cii-comment-lines)
  - [C.1 Header lines](#c1-header-lines)
  - [C.2 Column specification line](#c2-column-specification-line)
  - [C.3 Track lines](#c3-track-lines)
- [D. References](#d-references)
- [E. Change log](#e-change-log)


&nbsp;
## A. Introduction and background
&nbsp;
_(Return to: [Contents](#contents))_


&nbsp;
### A.1 Overview
&nbsp;

_(Return to: [Contents](#contents))_

GSuite is a simple tabular text format for use in specifying a suite (ie. a set
or collection) of genomic annotation tracks (simply called tracks in this
document). A GSuite file does not contain any genomic data as such, but
provides metadata necessary to locate the track contents, info on whether the
track has been preprocessed in a manner suitable for analysis (see the BTrack
file format), some basic information on how to analyze the data (see the track
type concept), as well as the reference genome build that the track coordinates
are based upon. In addition to this, the user may add as many custom metadata
columns he/she needs.


&nbsp;
### A.2 Suites of tracks
&nbsp;

_(Return to: [Contents](#contents))_

Central to the concept of track suites is the idea that tracks which take part
in a GSuite file should be somewhat related in contents and format. Although
the GSuite format allows heterogeneous tracks to be banded together in a single
file, such files will typically not be useful for analysis purposes, as one
would almost always need to restrict the contents and/or format as required by
the analysis tools. For instance, a tool that finds the intersection of base
pairs covered by all tracks in a suite would require all tracks to be of type
"points" or "segments", not "function", as tracks of that type cover all base
pairs (see section Track Types below for more info). For this reason, the
GSuite file format specifies a set of four header variables that could (and
should) be stated in the beginning of the file. These header variables function
as a summary over the tracks in the file, providing a specific value if all the
tracks are in accordance with each other. If the different tracks varies on
this particular aspect, the header variable is set to the reserved keyword
"multiple". This typically indicates that the collection of tracks is not yet
focused enough to be usable as a suite of tracks for analysis purposes.


&nbsp;
### A.3 Location of tracks
&nbsp;

_(Return to: [Contents](#contents))_

In order to analyze multiple tracks, one obviously needs to acquire such
tracks. Some tracks one might have acquired directly from sequencing endeavors
(e.g. ChIP-seq peaks), but often one needs to fetch such tracks from public
repositories and databases such as those provided by the ENCODE
[1](#d-references) and Roadmap Epigenomics [2](#d-references) projects. GSuite
supports the specification of suites of tracks before the actual track files
has been retrieved from a server. In such cases, the location of the tracks are
termed "remote" and the GSuite file would typically contain an HTTP or FTP
address to the remote location. Tracks that have been retrieved and is stored
at the same place as the GSuite file are termed as "local".


&nbsp;
### A.4 Preprocessing of tracks into a binary format (BTrack)
&nbsp;

_(Return to: [Contents](#contents))_

As part of the implementation of an analysis tool, one would typically need a
track to be translated, or preprocessed, into a binary format before analyses
takes place, as this greatly improves analysis speed. Often this is done
behind-the-scenes inside the analysis tool. In the GSuite format, however, the
concept of preprocessed binary versions of a track has been included explicitly
as part of the format. The reason for this is that preprocessing typically
takes some time per track, and when one works with multiple tracks (often
hundreds) this step will thus consume a significant amount of time. Carrying
out the preprocessing step as a one-time process, instead of every time one
runs an analysis tool, will thus save much time for the user. Analysis tools
therefore typically require the tracks in a GSuite to be preprocessed in
advance.

Preprocessing of a GSuite file results in the tracks being stored in the BTrack
format. BTrack is a binary format for genomic tracks that allows for fast
retrieval and efficient analyses by the storage of data columns as numeric
arrays. An analogue to the BTrack format in the domain of sequence alignment is
the BAM format, which is a binary version of the textual SAM format. BTrack is
thus the binary version of the previously published GTrack format
[3](#d-references).

BTrack is the new name for the previously unnamed internal track storage format
used in the Genomic HyperBrowser [3,4]. The BTrack format has seen several
major updates as part of the HyperBrowser code base, and will now soon be
released as a separate binary format that allows multiple tracks to be stored
in a single binary file (currently unpublished). The GSuite format is
intimately linked to the BTrack format, as a BTrack file would be able to store
both a GSuite file together with the actual track contents.


&nbsp;
### A.5 Track types
&nbsp;

_(Return to: [Contents](#contents))_

The concept of track types has been examined in detail in a previous
publication [2](#d-references). Briefly, a track type is a characterization of
a the geometrical/mathematical properties of a track. A track is typically
envisioned as data somehow located along the DNA sequence of a particular
reference genome. The simplest track type is "points", which refer to single
base pairs scattered along the genome, e.g. SNPs. "Segments" are the more
common ones, which represents regions of the DNA, e.g. genes. With the addition
of values and/or cross-genomic links, a total of 15 track types was delineated
in [3](#d-references). The main usage scenario of track types is to limit which
tracks it makes sense to use as input to a particular analysis tool. For
example, an analysis of the base pair overlap of two tracks would typically
require the tracks to be of type "segments". When it comes to the analysis of
multiple tracks, one would typically require the tracks to be analyzed to be of
the same track type. The GSuite format thus supports "track type" as one of the
main header variables (see below). The following is a list of all the supported
15 track types, as delineated in [3](#d-references):

- Points (P)
- Valued Points (VP)
- Segments (S)
- Valued Segments (VS)
- Genome Partition (GP)
- Step Function (SF)
- Function (F)
- Linked Points (LP)
- Linked Valued Points (LVP)
- Linked Segments (LS)
- Linked Valued Segments (LVS)
- Linked Genome Partition (LGP)
- Linked Step Function (LSF)
- Linked Function (LF)
- Linked Base Pairs (LBP)


&nbsp;
## B. Example GSuite files
&nbsp;

_(Return to: [Contents](#contents))_

Before going into the details of the GSuite format, one should be able to get a
quick overview of the format by looking at these example files:


&nbsp;
### B.1 Example 1
&nbsp;

_(Return to: [Contents](#contents))_

- List of URLs

```
http://www.server.com/path/to/file.bed
http://www.server.com/path/to/file2.bed
http://www.server2.com/path/to/other_file.bed
ftp://www.server3.com/path/to/new_file.wig
```


&nbsp;
### B.2 Example 2
&nbsp;

_(Return to: [Contents](#contents))_

- Header lines
- List of URLs

```
##location: remote
##file format: primary
##track type: segments
##genome: hg38
http://www.server.com/path/to/file.bed
http://www.server.com/path/to/file2.bed
http://www.server2.com/path/to/other_file.bed
ftp://www.server3.com/path/to/new_file.gff
```


&nbsp;
### B.3 Example 3
&nbsp;

_(Return to: [Contents](#contents))_

- Header lines
- List of URLs
  - including GSuite-specific URL schemes
- Comment lines
- Extra columns

```
##location: multiple
##file format: multiple
##track type: segments
##genome: hg38
###uri                                           title      p-values
http://www.server.com/path/to/file.bed           track_1    0.002
http://www.server.com/path/to/file2.bed          track_2    0.1
http://www.server2.com/path/to/other_file.bed    track_3    1.0
ftp://www.server3.com/path/to/new_file.gff       track_4    0.8
galaxy:/abcd1234abcd;bed                         track_5    0.012
hb:/my/track/name                                track_6    .
```

- _Note:_  
  This example text has been formatted in columns using spaces, not tabs, as
  required by the GSuite format.


&nbsp;
## C. Syntax of the GSuite format
&nbsp;

_(Return to: [Contents](#contents))_

GSuite is a tabular text file format. All GSuite filenames should end with
".gsuite". The GSuite format consists of 5 different line types, distinguished
by the leading characters and numbered here by order of appearance in the file:

- [A.1 Overview](#a1-overview)
- [A.2 Suites of tracks](#a2-suites-of-tracks)
- [A.3 Location of tracks](#a3-location-of-tracks)
- [A.4 Preprocessing of tracks into a binary format (BTrack)](#a4-preprocessing-of-tracks-into-a-binary-format-btrack)
- [A.5 Track types](#a5-track-types)
- [B.1 Example 1](#b1-example-1)
- [B.2 Example 2](#b2-example-2)
- [B.3 Example 3](#b3-example-3)
- [C.i Empty lines](#ci-empty-lines)
  - [_C.i.1 Leading characters_](#_ci1-leading-characters_)
  - [_C.i.2 Syntax_](#_ci2-syntax_)
  - [_C.i.3 Usage_](#_ci3-usage_)
  - [_C.i.4 Description_](#_ci4-description_)
- [C.ii Comment lines](#cii-comment-lines)
  - [_C.ii.1 Leading characters_](#_cii1-leading-characters_)
  - [_C.ii.2 Example_](#_cii2-example_)
  - [_C.ii.3 Usage_](#_cii3-usage_)
  - [_C.ii.4 Description_](#_cii4-description_)
- [C.1 Header lines](#c1-header-lines)
  - [_C.1.1 Leading characters_](#_c11-leading-characters_)
  - [_C.1.2 Syntax_](#_c12-syntax_)
  - [_C.1.3 Example_](#_c13-example_)
  - [_C.1.4 Usage_](#_c14-usage_)
  - [_C.1.5 Description_](#_c15-description_)
  - [_C.1.6 Reserved header variable names_](#_c16-reserved-header-variable-names_)
    - [_C.1.6.1 `location`_](#_c161-location_)
    - [_C.1.6.2 `file format`_](#_c162-file-format_)
    - [_C.1.6.3 `track type`_](#_c163-track-type_)
    - [_C.1.6.4 `genome`_](#_c164-genome_)
  - [_C.1.7 Parser notes_](#_c17-parser-notes_)
- [C.2 Column specification line](#c2-column-specification-line)
  - [_C.2.1 Leading characters_](#_c21-leading-characters_)
  - [_C.2.1 Syntax_](#_c21-syntax_)
  - [_C.2.2 Example_](#_c22-example_)
  - [_C.2.3 Default value_](#_c23-default-value_)
  - [_C.2.4 Usage_](#_c24-usage_)
  - [_C.2.5 Description_](#_c25-description_)
  - [_C.2.6 Reserved column names_](#_c26-reserved-column-names_)
    - [_C.2.6.1 `uri`_](#_c261-uri_)
    - [_C.2.6.1.1 Remote URI schemes_](#_c2611-remote-uri-schemes_)
    - [_C.2.6.1.2 Local URI schemes_](#_c2612-local-uri-schemes_)
    - [_C.2.6.1.2.1 `file`_](#_c26121-file_)
    - [_C.2.6.1.2.2 `galaxy`_](#_c26122-galaxy_)
    - [_C.2.6.1.2.3 `hb`_](#_c26123-hb_)
    - [_C.2.6.2 `title`_](#_c262-title_)
    - [_C.2.6.3 `file_format`_](#_c263-file_format_)
    - [_C.2.6.4 `track_type`_](#_c264-track_type_)
    - [_C.2.6.5 `genome`_](#_c265-genome_)
    - [_C.2.6.5 Custom columns_](#_c265-custom-columns_)
  - [_C.2.7 Optional columns_](#_c27-optional-columns_)
  - [_C.2.8 Parser notes_](#_c28-parser-notes_)
- [C.3 Track lines](#c3-track-lines)
  - [_C.3.2 Leading characters_](#_c32-leading-characters_)
  - [_C.3.3 Syntax_](#_c33-syntax_)
  - [_C.3.4 Example_](#_c34-example_)
  - [_C.3.5 Usage_](#_c35-usage_)
  - [_C.3.6 Description_](#_c36-description_)

_Note_:  
The arabic number preceding each line type defines the order in which the lines
must be present: i.e., the column specification line (C.2) must follow the
header lines (C.1). Roman numbers indicate comments and emtpy lines, which may
be present anywhere.


&nbsp;
### C.i Empty lines
&nbsp;  

_(Return to: [Contents](#contents)) /
[C. Syntax of the GSuite format](#c-syntax-of-the-gsuite-format))_


&nbsp;
#### _C.i.1 Leading characters_
&nbsp;
```
(none)
```


&nbsp;
#### _C.i.2 Syntax_
&nbsp;
```
only whitespace characters (space, tab, newline, return)
```


&nbsp;
#### _C.i.3 Usage_
&nbsp;

_optional_


&nbsp;
#### _C.i.4 Description_
&nbsp;

Empty lines are allowed anywhere in the GSuite file.

These will be ignored by the parsers


&nbsp;
### C.ii Comment lines
&nbsp;

_(Return to: [Contents](#contents) /
[C. Syntax of the GSuite format](#c-syntax-of-the-gsuite-format))_


&nbsp;
#### _C.ii.1 Leading characters_
&nbsp;

```
#
```

(a single hash character)


&nbsp;
#### _C.ii.2 Example_
&nbsp;

```
# this is a comment
```


&nbsp;
#### _C.ii.3 Usage_
&nbsp;

optional


&nbsp;
#### _C.ii.4 Description_
&nbsp;

Comments are allowed anywhere and will be ignored by parsers. Note that
a comment line following a track line is considered to be a comment for that
track and can for instance be used by tools that creates GSuite files to
present track-specific error messages to the user.


&nbsp;
### C.1 Header lines
&nbsp;

_(Return to: [Contents](#contents) /
[C. Syntax of the GSuite format](#c-syntax-of-the-gsuite-format))_


&nbsp;
#### _C.1.1 Leading characters_
&nbsp;

```
##
```

&nbsp;
#### _C.1.2 Syntax_
&nbsp;

```
##variable:[ ]*value
```

- where
  - `variable` = Header variable name
  - `[ ]*` = Optional space characters
  - `value` = Header variable value


&nbsp;
#### _C.1.3 Example_
&nbsp;

```
##location: local
##file format: preprocessed
##track type:  segments
##genome: hg38
```

&nbsp;
#### _C.1.4 Usage_
&nbsp;

optional in an input GSuite file, but auto-generated when a GSuite is created
as output from a tool


&nbsp;
#### _C.1.5 Description_
&nbsp;

A header variable contains information that relates to the whole of the GSuite
file, and is thus a summary over all the tracks in the file. The header
variables names are limited to a set of reserved keywords, each with a
restricted set of values. The header variables are related to reserved columns
of the track lines (see the section "Column specification line" below).


&nbsp;
#### _C.1.6 Reserved header variable names_
&nbsp;

The following header variable names have reserved meaning, and are the only
headers allowed by the GSuite format:

    `location`  
    `file format`  
    `track type`  
    `genome`  

In the following, the reserved header variables are described in detail:


&nbsp;
##### _C.1.6.1 `location`_
&nbsp;

  Specifies whether the data contents of all tracks in the GSuite are found at
  remote locations on the Internet, or if they have been downloaded locally to
  the service parsing the GSuite file (see section "Location of tracks" above).
  Note that the service parsing the GSuite may itself be located on e.g. a web
  server, but the tracks of the GSuite is still considered as local if they are
  on the same server as the service.

  The location header is a summary of the different types of URI schemes
  present in the `uri` column in the track lines (see the section "Column
  specification line" below). All supported types of URIs are thus defined as
  either remote or local.

  _Allowed values:_  
  `unknown`  
  `remote`  
  `local`  
  `multiple`


&nbsp;
##### _C.1.6.2 `file format`_
&nbsp;

  Specifies whether all tracks have been preprocessed into the binary format
  BTrack, which is a prerequisite for most analysis tools. The `file format`
  header variable is a summary of the contents of the `file_format` column in
  the track lines (see the section "Column specification line" below).

  _Allowed values:_  
  `unknown`  
  `primary`  
  `preprocessed`  
  `multiple`


&nbsp;
##### _C.1.6.3 `track type`_
&nbsp;

  Specifies the track type common for all the tracks in the GSuite file, if
  any. See the section "Track types" above for more information. The `track
  type` header variable is a summary of the contents of the `track_type` column
  in the track lines (see the section "Column specification line" below).

  Note that if the track types of the tracks are different, but based upon the
  same basic type, the common track type of the GSuite file is set to the
  simplest track type that can used to describe all tracks, if any. E.g. if two
  tracks have the types `valued segments` and `linked segments`, respectively,
  the track type of the GSuite file is `segments`. If there is no such simple
  track type, the keyword `multiple` is used.

  _Allowed values:_  
  `unknown`  
  `points`  
  `valued points`  
  `segments`  
  `valued segments`  
  `genome partition`  
  `step function`  
  `function`  
  `linked points`  
  `linked valued points`  
  `linked segments`  
  `linked valued segments`  
  `linked genome partition`  
  `linked step function`  
  `linked function`  
  `linked base pairs`  
  `multiple`


&nbsp;
##### _C.1.6.4 `genome`_
&nbsp;

  Specifies the reference genome for all the tracks in the GSuite file. The
  `genome` header variable is a summary of the contents of the `genome` column
  in the track lines (see the section "Column specification line" below. The
  actual keyword for the genome build is dependent on the implementation of the
  analysis tools that will make use of the information. The GSuite format
  accepts any string as the genome.

  _Allowed values:_  
  `unknown`  
  `multiple`  
  \+ any other string specifying a reference genome assembly


&nbsp;
#### _C.1.7 Parser notes_
&nbsp;

If a header variable is missing, it will be auto-generated from the track
lines. If a header variable is present, but with a value that is inconsistent
with the track lines, the parser will return an error. Note that all header
variable lines except for the "genome" variable allow a mix of lower- and
uppercase characters.

The following logic for the values `unknown` and `multiple` will hold for all
header variables:

- `unknown`:  
  if at least one track has `unknown` as it value, the value of the GSuite
  header variable will also be `unknown`, regardless of the values for the
  other tracks.

- `multiple`:  
  if at least one track has a different value than the others, the value of the
  GSuite header variable will be `multiple` (unless the value for one of the
  tracks is `unknown`, in which case that keyword takes precedence).


&nbsp;
### C.2 Column specification line
&nbsp;

_(Return to: [Contents](#contents) /
[C. Syntax of the GSuite format](#c-syntax-of-the-gsuite-format))_


&nbsp;
#### _C.2.1 Leading characters_
&nbsp;

```
###
```


&nbsp;
#### _C.2.1 Syntax_
&nbsp;

```
###col1    col2    col3...
```

- where
  - `col1`, `col2`, `col3` = Column names
  - `" "` = tab character


&nbsp;
#### _C.2.2 Example_
&nbsp;

```
###uri   title   file_format   track_type   genome   description   p-value
```

(here: spaces instead of tabs)


&nbsp;
#### _C.2.3 Default value_
&nbsp;

```
###uri
```


&nbsp;
#### _C.2.4 Usage_
&nbsp;

Optional, but if not defined the column specification line retains the default
value. This means that a list of URI's is a valid GSuite file.


&nbsp;
#### _C.2.5 Description_
&nbsp;

The column specification line is a tab-separated list of column names,
containing metadata information about the tracks in the track suite defined i a
GSuite file.


&nbsp;
#### _C.2.6 Reserved column names_
&nbsp;

The GSuite specification defines a set of five reserved column names:

    `uri`  
    `title`  
    `file_format`  
    `track_type`  
    `genome`  

In addition, any number of custom column names can be specified, each
representing varied types of metadata available for the tracks.

In the following, the reserved column names are described in detail:


&nbsp;
##### _C.2.6.1 `uri`_
&nbsp;

A unique identifier of a track file, following the Universal Resource
Identifier (URI) format [5](#d-references). GSuite supports a diverse set of URI schemes:


&nbsp;
##### _C.2.6.1.1 Remote URI schemes_
&nbsp;

- URI schemes for data residing at a remote location (i.e., when `location` is
  `remote`):

  `ftp`  
  `http`  
  `https`  
  `rsync`

  _Examples:_

  ```
  ftp://ftp.server.com/path/to/file.bed
  http://www.server.com:8080/index?filename=track.wig
  rsync://server.com/path/to/file
  ```


&nbsp;
##### _C.2.6.1.2 Local URI schemes_
&nbsp;

- URI schemes for data residing locally (i.e., when `location` is `local`):

  `file`  
  `galaxy`  
  `hb`

  defined as follows:


&nbsp;
##### _C.2.6.1.2.1 `file`_
&nbsp;

  - `file`  
    Standard URI schema for local files.

    **Example:**

    ```
      file:///path/to/file/bed
    ```

    **Note:**  
    The `file` scheme does not support files residing other places than
    "localhost". The host part of the URI is thus unneeded, hence the triple
    `/` characters.


&nbsp;
##### _C.2.6.1.2.2 `galaxy`_
&nbsp;

-   `galaxy`  
    The `galaxy` scheme uniquely identifies a Galaxy dataset, but currently
    only works for the local installation of the Galaxy analysis framework that
    is set up with GSuite support, i.e. one cannot (yet) provide an URI to a
    remote Galaxy installation [6](#d-references). The

    **Syntax:**

    ```
    galaxy:/dataset_key[/directory/structure/to/file]
    ```

    **Example**

    Multiple files can be stored within one Galaxy history element using the
    directory structure syntax.


&nbsp;
##### _C.2.6.1.2.3 `hb`_
&nbsp;

  - `hb` The "HB" scheme identifies a track stored as the BTrack format within
    the local installation of GSuite HyperBrowser. The syntax is as follows:

    `hb:/track/name/hierarchy`

  Note that for all the URI schemes except the "HB" one, GSuite supports the
  additional specification of file suffix after a semicolon, as in this
  example:

  `ftp://ftp.server.com/path/to/file;bed`

  This usable if the file path itself does not contain the suffix, and hence
  does not contain any information on the actual file format of the track.

  - Parser notes:

  Note that services available from e.g. the web should disable the "file"
  scheme, as this is inherently insecure.


&nbsp;
##### _C.2.6.2 `title`_
&nbsp;

    The title of the track, as specified by the user. Each track title must be
    unique within a specific GSuite, so that one may use the title as a key to
    uniquely reference specific tracks in a GSuite.

    Allowed values: *any*


&nbsp;
##### _C.2.6.3 `file_format`_
&nbsp;

  - `File_format`:

    Specifies whether the track has been preprocessed into the binary format
    BTrack or not, as described in the section "Header lines" above.

    If the GSuite parser understands the file suffix to be an un-preprocessed
    format, file format is automatically set to "primary". Similarly, tracks in
    the BTrack format (including those with "HB" as URI) automatically gets
    "preprocessed" as "file_format".

    Allowed values: `unknown`, `primary`, `preprocessed`

    Default value: `unknown`


&nbsp;
##### _C.2.6.4 `track_type`_
&nbsp;
  - `Track_type`:

    Specifies the track type of the track, as described in the section "Header
    lines" above. If the track is preprocessed into a BTrack file, the value of
    the "track_type" is automatically collected from the BTrack file(s)
    themselves.

    Allowed values: `unknown`, `points`, `valued points`, `segments`, `valued
    segments`, `genome partition`, `step function`, `function`, `linked
    points`, `linked valued` `points`, `linked segments`, `linked valued
    segments`, `linked genome partition`, `linked step function`, `linked
    function`, `linked base pairs`

    Default value: `unknown`


&nbsp;
##### _C.2.6.5 `genome`_
&nbsp;

  - `Genome`:

    Specifies the reference genome build used as basis of the track, as
    described in the section "Header lines" above.

    Allowed: `unknown`, any other string specifying a reference genome

    Default value: `unknown`


&nbsp;
##### _C.2.6.5 Custom columns_
&nbsp;

  - Custom columns

    Any number of custom columns can be added. Any string can be used as value
    for each track, so there are little or no rules on the content defined
    within the GSuite format. Missing values for custom columns are denoted
    with the period character: '.'


&nbsp;
#### _C.2.7 Optional columns_
&nbsp;

If the value in the "file_format" column is the same for all tracks in a
GSuite, the column can be removed, leaving only the value of the "file format"
header variable to speak for all individual tracks. The same logic holds also
for the columns "track_type" and "genome".


&nbsp;
#### _C.2.8 Parser notes_
&nbsp;

Column names are treated as case insensitive. All column names must also
be unique. The columns can be ordered in any way, but it is recommended for
readability to use "uri" and "title" as the first two rows, if defined.


&nbsp;
### C.3 Track lines
&nbsp;

_(Return to: [Contents](#contents) /
[C. Syntax of the GSuite format](#c-syntax-of-the-gsuite-format))_



&nbsp;
#### _C.3.2 Leading characters_
&nbsp;
```
(none)
```


&nbsp;
#### _C.3.3 Syntax_
&nbsp;
```
val1 val2 val3...
```
where
- `val1, val2, val3` = column values
- `" "` = tab character


&nbsp;
#### _C.3.4 Example_
&nbsp;

```
###uri                                  title          p-value
http://www.server.com/path/to/file.bed  My cool track  0.00013
```

(here: spaces instead of tabs)


&nbsp;
#### _C.3.5 Usage_
&nbsp;

Track lines are optional. If no track lines are specified, the GSuite file
represents an empty collection of tracks.



&nbsp;
#### _C.3.6 Description_
&nbsp;

Each track is specified as a tab-separated list of metadata values, as defined
by the column specification line. See the section
["C.2.6 Reserved column names"](#_c26-reserved-column-names_)
for a more detailed discussion on the allowed values.


&nbsp;
## D. References
&nbsp;

_(Return to: [Contents](#contents)_

1. ENCODE Project Consortium. "An integrated encyclopedia of DNA elements in
   the human genome." _Nature_ 489.7414 (2012): 57-74.
2. Kundaje, Anshul, et al. "Integrative analysis of 111 reference human
   epigenomes." _Nature_ 518.7539 (2015): 317-330.
3. Gundersen, Sveinung, et al. "Identifying elemental genomic track types and
   representing them uniformly." _BMC Bioinformatics_ 12.1 (2011): 1.
4. Sandve, Geir K., et al. "The Genomic HyperBrowser: inferential genomics at
   the sequence level." _Genome Biology_ 11.12 (2010): 1-12.
5. Uniform Resource Identifier (URI): Generic Syntax
   (https://tools.ietf.org/html/rfc3986)
6. Goecks, Jeremy, Anton Nekrutenko, and James Taylor. "Galaxy: a comprehensive
   approach for supporting accessible, reproducible, and transparent
   computational research in the life sciences." _Genome Biology_ 11.8 (2010):
   R86.


&nbsp;
## E. Change log
&nbsp;

_(Return to: [Contents](#contents)_

v0.1 - 2015.07.06:

* Initial version of the GSuite specification document.

v0.2 - 2016.07.06:

* Fixed typos and cleaned up text several places. Ready for initial submission
  of the GSuite HyperBrowser manuscript.

v0.3 - 2018.05.23:

* Converted specification to Markdown
* Smaller changes in formatting and wording

v1.0a - 2020.07.13:

* Heavy cleanup and reformatting of Markdown
* Still quite a bit of cleanup to do in the syntax section
