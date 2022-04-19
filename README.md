# Using Github as an IIIF Image Repository

[IIIF](https://iiif.io) is a set of open standards for delivering high-quality, attributed digital objects online at scale. It’s also an international community developing and implementing the IIIF APIs. IIIF is backed by a consortium of leading cultural institutions.

This document describes an approach for hosting IIIF images in a Github repository.  In this approach images and IIIF properties files are stored in a Github repository and served by an external service that dynamically generates IIIF presentation manifests and serves IIIF image tiles from the Github content.  The metadata in this repository are used to generate manifest files that are compliant with the [IIIF Presentation API 3.0](https://iiif.io/api/presentation/3.0/).

Where possible, purpose-built image sharing sites should be used for hosting and sharing image files.  [Wikimedia Commons](https://commons.wikimedia.org/wiki/Main_Page) is one such (free-to-use) site for hosting and sharing media files.  While Wikimedia Commons (and some other file sharing sites) do not yet support IIIF, the same service that is used to generate IIIF manifests and serve images from Github repositories also works with many of these sites.  The advantage of using images stored on a site like Wikimedia Commons is that IIIF manifests can be automatically generated using metadata obtained from Application Programming Interfaces (APIs) provided by the sites.

As an example, an IIIF manifest for the [https://commons.wikimedia.org/wiki/File:White-cheeked_Honeyeater_-_Maddens_Plains.jpg](https://commons.wikimedia.org/wiki/File:White-cheeked_Honeyeater_-_Maddens_Plains.jpg) image hosted by Wikimedia Commons is automatically generated using the URL [https://iiif.visual-essays.net/wc:White-cheeked_Honeyeater_-_Maddens_Plains.jpg/manifest.json](https://iiif.visual-essays.net/wc:White-cheeked_Honeyeater_-_Maddens_Plains.jpg/manifest.json).  Using the auto-generated IIIF manifest the image can easily be viewed in an IIIF compatible viewer or used by IIIF tools.  The White-cheeked Honeyeater image can be seen in the iiif.visual-essays.net image viewer at [https://iiif.visual-essays.net/wc:White-cheeked_Honeyeater_-_Maddens_Plains.jpg](https://iiif.visual-essays.net/wc:White-cheeked_Honeyeater_-_Maddens_Plains.jpg)

In many cases a minimal set of IIIF required metadata for a single image can be generated from site-wide default values and image-specific metadata that can be inferred from the image file name (such as the image title and reuse rights code).

In situations where more descriptive metadata is needed a simple key-value properties file can be associated with a single image or a group of images located in a folder.

# IIIF Manifest Generation

IIIF manifests are automatically generated for Github files by the [iiif.visual-essays.net](https://iiif.visual-essays.net) service.  Manifest URLs for Github hosted content take the form of `https://iiif.visual-essays.net/gh:<GITHUB_USERNAME>/<GITHUB_REPOSITORY>/<GITHUB_FILE_PATH>/manifest.json`

For example, the manifest URL for the image file [images/Italy_2022/Positano-CC0.jpg](images/Italy_2022/Positano-CC0.jpg) in this repository would be [https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Italy_2022/Positano-CC0.jpg/manifest.json](https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Italy_2022/Positano-CC0.jpg/manifest.json)

A similar URL can be used to view the image in a simple IIIF viewer.  The viewer URL is the same as the manifest URL after the removal of the trailing `/manifest.json` path segment.  For instance, using the example from above the image can be viewed at the URL [https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Italy_2022/Positano-CC0.jpg](https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Italy_2022/Positano-CC0.jpg)

Note that the image viewer includes a small info icon near the top right of the image.  Clicking this icon will display a small popup with a condensed version of key manifest information, in particular a definition of the image reuse rights and required attribution statements.

# Options for image and/or metadata hosting

## Image only

The easiest method for hosting a IIIF image file in a Github repository is to use a simple file naming convention that is used to define the image label and an optional reuse rights code.  This represents the minimal user-defined metadata to create a valid IIIF manifest.  When a rights code is omitted from the file name the value `InC` (In copyright) is used as a default and the owner of the Github repository hosting the image is assumed to be the image owner.

The convention splits the file name into 2 segments that are separated by the first dash (`-`) character found in the name.  The characters in the first  segment are used for the image label in the generated IIIF manifest after replacing all underscore (`_`) characters with spaces.  The text in the second segment represent the reuse rights code to be associated with the image.  The reuse rights code will be transformed into a full URL as required by the IIIF manifest.

In cases where multiple images in the same folder need to use the same label a double underscore (`__`) can be used to signify the end of a label and the start of some arbitrary text that results in a unique file name.  The text after the double underscore but before the first dash is not included in the generated label.  For instance, file names like `Some_label__1-CC0.jpg` and `Some_label__2-CC0.jpg` could be used to create a unique file name for images that share a label and rights code.

The reuse rights for an image is defined by appending a Creative Commons or Rights Statements code to the end of the file name (but before the file extension).  The reuse rights code must be one of the [Creative Commons](https://creativecommons.org/licenses/) or [RightsStatements.org](https://rightsstatements.org/page/1.0/) codes defined in [Reuse Rights](#reuse-rights) section below.  If a rights code is not provided in the file name a default value is used.  The default value is obtained from properties files found in the folder hierarchy in which the image file is located.  If rights metadata is available in multiple properties files the value in the file that is closest to the image takes precedence.  For instance, if a properties file at the root of the repository defined the reuse rights as `CC BY` and then another properties file in a parent/ancestor folder of the image defined the reuse rights as `CC BY-SA`, the `CC BY SA` value would take precedence and would be used in the generated IIIF manifest.  If not properties files are found in the image folder hierarchy a default value of `InC` is used.

## Image and properties file

In situations where it is desired to associate richer metadata with an image a supplemental properties file may be used.  In this example the summary and depicts properties are defined. The `.yaml` properties files can be used to define standard IIIF properties like `label`, `summary`, `rights` and others.  Properties included in the `.yaml` file that are not associated with standard IIIF properties are added to the `metadata` manifest section which supports the definition of user-defined properties, like `depicts` in this example.  The `depicts` property in this example includes the [Wikidata](https://www.wikidata.org) QID for [Furore](https://www.wikidata.org/wiki/Q80948). 

```yaml
summary: The Fiordo di Furore, considered one of the most interesting geological features on the Amalfi Coast.
rights: http://creativecommons.org/licenses/by/4.0/
requiredStatement:
  label: attribution
  value: Image provided by Ron Snyder
depicts: Q80948
```

In this example the [generated IIIF manifest](https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Italy_2022/Fiordo_di_Furore.jpg/manifest.json) includes a label extracted from the image file name and the properties defined in both the 
[images/Italy_2022/iiif-props.yaml](images/Italy_2022/iiif-props.yaml) and [images/Italy_2022/iiif-props.yaml](images/Italy_2022/Fiordo_di_Furore.yaml) files.

The image can be viewed at [https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Italy_2022/Fiordo_di_Furore.jpg](https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Italy_2022/Fiordo_di_Furore.jpg)

## Properties file only

In situations where an image is hosted on another web site that does not provide a IIIF manifest a properties file is used to define the IIIF metadata in a generated IIIF manifest.  When using this method the URL to the externally hosted image is included in the properties file in addition to the usual IIIF metadata properties.

An example of Github hosted IIIF properties file with an externally hosted image can be seen at [images/Positano.yaml](images/Positano.yaml).

-  The generated manifest: [https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Positano.yaml/manifest.json](https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Positano.yaml/manifest.json)
-  Viewed in a IIIF image viewer: [https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Positano.yaml](https://iiif.visual-essays.net/gh:visual-essays/iiif-github/images/Positano.yaml)

# Defining Reuse Rights

The IIIF Presentation Manifests provide a flexible approach for explicitly defining the reuse rights for an image and any attribution (or other) statements that must be displayed when the image is used.

`Rights` refers string that identifies a license or rights statement that applies to the content of the resource, such as the JSON of a Manifest or the pixels of an image. The value must be drawn from the set of [Creative Commons](https://creativecommons.org/licenses/) license URIs or [RightsStatements.org](https://rightsstatements.org/page/1.0/) rights statement URIs.  When defining the image rights in an IIIF properties file the full URL is included.  When the rights are defined as part of the image filename the rights code (e.g, `CC-BY`) is used.  When a rights code is not defined in the image file name and a value is not found in a properties file a CC-BY license is used with a attribution to the Github repository owner.

## Creative Commons Licenses

- `CC0`: [Public Domain Dedication](http://creativecommons.org/publicdomain/zero/1.0/)
- `CC-BY`: [Attribution](http://creativecommons.org/licenses/by/4.0/)
- `CC-BY-SA`: [Attribution-ShareAlike](http://creativecommons.org/licenses/by-sa/4.0/)
- `CC-BY-ND`: [Attribution-NoDerivs](http://creativecommons.org/licenses/by-nd/4.0/)
- `CC-BY-NC`: [Attribution-NonCommercial](http://creativecommons.org/licenses/by-nc/4.0/)
- `CC-BY-NC-SA`: [Attribution-NonCommercial](http://creativecommons.org/licenses/by-nc-sa/4.0/)
- `CC-BY-NC-ND`: [Attribution-NonCommercial-NoDerivs](http://creativecommons.org/licenses/by-nc-nd/4.0/)

## Rights Statements

- `InC`: [In Copyright](http://rightsstatements.org/vocab/InC/1.0/)
- `InC-OW-EU`: [In Copyright - EU Orphan Work](http://rightsstatements.org/vocab/InC-OW-EU/1.0/)
- `InC-EDU`: [In Copyright - Educational Use Permitted](http://rightsstatements.org/vocab/InC-EDU/1.0/)
- `InC-NC`: [In Copyright - Non-Commercial Use Permitted](http://rightsstatements.org/vocab/InC-NC/1.0/)
- `InC-RUU`: [In Copyright - Rights-Holder(s) Unlocatable or Unidentifiable](http://rightsstatements.org/vocab/InC-RUU/1.0/)
- `NoC-CR`: [No Copyright - Contractual Restrictions](http://rightsstatements.org/vocab/NoC-CR/1.0/)
- `NoC-NC`: [No Copyright - Non-Commercial Use Only](http://rightsstatements.org/vocab/NoC-NC/1.0/)
- `NoC-OKLR`: [No Copyright - Other Known Legal Restrictions](http://rightsstatements.org/vocab/NoC-OKLR/1.0/)
- `NoC-US`: [No Copyright - United States](http://rightsstatements.org/vocab/NoC-US/1.0/)
- `CNE`: [Copyright Not Evaluated](http://rightsstatements.org/vocab/CNE/1.0/)
- `UND`: [Copyright Undertermined](http://rightsstatements.org/vocab/UND/1.0/)
- `NKC`: [No Known Copyright](http://rightsstatements.org/vocab/NKC/1.0/)
