---
title: 'BioHack24 report: Toward improving mechanisms for extracting RDF shapes from large inputs'
title_short: 'BioHack24 report: RDF shapes from large inputs'
tags:
  - RDF
  - ShEx
  - SHACL
  - Schema
  - Automatic extraction
  - Large data
  - sheXer
authors:
  - name: Daniel Fernández-Álvarez
    affiliation: 1
    orcid: 0000-0002-8666-7660
  - name: Yasunori Yamamoto
    affiliation: 2
    orcid: 0000-0002-6943-6887
  - name: Jose Emilio Labra Gayo
    affiliation: 1
    orcid: 0000-0001-8907-5348
  - name: Andra Waagmeester
    affiliation: 3
    orcid: 0000-0001-9773-4008
affiliations:
  - name: Weso Research Group, University of Oviedo, Spain.
    index: 1
  - name: Database Center for Life Science (DBCLS), ROIS-DS, Kashiwa, Chiba, Japan
    index: 2
  - name: Department of Medical Informatics, University of Amsterdam, The Netherlands.
    index: 3
date: 04 September 2024
cito-bibliography: paper.bib
event: BH24
biohackathon_name: "DBCLS BioHackathon 2024"
biohackathon_url:   "http://2024.biohackathon.org/"
biohackathon_location: "Fukushima, Japan, 2024"
group: Getting shapes from large RDF inputs
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-japan/bh24-getting-shapes-from-large-rdf-inputs
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Daniel Fernández-Álvarez, Yasunori Yamamoto \emph{et al.}
---
# Abstract

RDF shapes have proven to be effective mechanisms for describing and validating RDF content. Typically, shapes are written by domain experts. However, writing and maintaining these shapes can be challenging when dealing with large and complex schemas. To address this issue, automatic shape extractors have been proposed. These tools are designed to analyze existing RDF content and generate shapes that conform with the underlying schemas. Nevertheless, extracting shapes from large datasets presents significant scalability challenges.

In this document, we describe our work during the 2024 BioHackathon held in Fukushima, Japan, to tackle this problem. Our approach is based on slicing the input data, performing parallelized shape extraction processes, and merging the resulting partial outputs. By refining our software and methods, we successfully extracted shapes from a subset of UniProt, containing an estimated 15.9 billion triples.

# Introduction


Shape schemas (SHACL [@citesAsAuthority:SHACLSpec], ShEx [@citesAsAuthority:prud2014shape]) have proven to be effective tools for validating and describing RDF content. However, writing and maintaining RDF schemas can be a time-consuming task, particularly when multiple shapes are involved. To address this challenge, several approaches for automatic schema discovery have been proposed [@citesAsAuthority:fernandez2022shexer; @citesAsAuthority:Bolleman2023; @citesAsAuthority:fernandez2024extracting; @citesAsAuthority:cimmino2020astrea; @citesAsAuthority:rabbani2023extraction; @citesAsAuthority:keely2023shaclgen; @citesAsAuthority:boneva2019shape; @citesAsAuthority:mihindukulasooriya2018rdf; @citesAsAuthority:groz2022inference; @citesAsAuthority:omran2020towards; @citesAsAuthority:spahiu2018towards; @citesAsAuthority:rabbani2023shactor]. 

These approaches are especially useful when working with large RDF datasets, where manually writing, maintaining, or even understanding schemas becomes significantly more difficult. However, most existing tools for automatic schema discovery encounter performance issues or high hardware requirements when applied to large-scale datasets.

Current established methods for extracting shapes from large datasets mainly rely on sampling [@citesAsAuthority:fernandez2022shexer; @citesAsAuthority:rabbani2023extraction] or utilizing [VoID](https://www.w3.org/TR/void/) data [@citesAsAuthority:Bolleman2023]. VoID-based approaches generally offer the best performance in terms of execution time and can produce high-quality shapes, provided the VoID data itself is of sufficient quality. However, this method is limited by the need for pre-existing VoID data. On the other hand, sampling-based approaches can extract approximate shapes from any input, but the sample size can impact the quality of the results.

During a previous BioHackathon, we proposed and implemented a prototype for extracting shapes using a different strategy:

1. We split the input source into distinct slices.
2. We ran the extraction process on each slice.
3. We merged the resulting schemas into a single, unified schema.

The shape extractor used in the second step was [sheXer](https://github.com/DaniFdezAlvarez/shexer). The approach, results, and experiments conducted have been published [@citesAsAuthority:fernandez2024extracting].


In this project, we aim to refine this approach and explore new use cases for our prototype. [Our current implementation of this approach](https://github.com/shex-consolidator/shex-consolidator) is publicly available. 


# Extracted Schemas

During this hackathon, we successfully extracted schemas from a substantial portion of UniProt’s graph. We used 358 different files from UniProt's dump, which we estimate to contain approximately 15.9 billion triples. Specifically, we used all files from the UniProt release (dated August 25th, 2024) whose names begin with 'uniprotkb_reviewed_' or 'uniprotkb_unreviewed_' from a [UniProt's dump](https://rdfportal.org/download/uniprot/20240826/).


We performed individual extraction processes for each file and then merged the results into a single schema that describes all shapes observed within the dataset. The [extracted shapes](https://github.com/biohackathon-japan/bh24-getting-shapes-from-large-rdf-inputs/blob/main/data/uniprotkb_consolidated_358.shex) are publicly available.

We would like to highlight several insights from this experience of extracting shapes from such a large source, which can guide future efforts to improve this approach:

* The extraction process took several days. While partial extractions for each data slice could be performed independently, our current prototype uses a sequential approach. Addressing this limitation is essential for reducing execution time.

* We observed that the shapes extracted from only 262 out of the 358 files were identical to those obtained from processing the entire dataset. Although this behavior cannot be generalized to all input sources, it suggests that incorporating convergence detection mechanisms into our process could help avoid exploring redundant data, potentially reducing execution time.

* Our current prototype is based on the sheXer extractor, which influences the process in several ways:
  
  * sheXer produces statistics on the extracted constraints, as well as examples of nodes conforming to a shape and objects conforming to the object position in a triple constraint. However, our prototype does not always retain this information when merging different versions of a shape. Improving the integration between sheXer and our prototype could help preserve this valuable data.
  
  * Due to sheXer's internal RDF parsers, the format of the input data significantly impacts performance. In this case, the input data was serialized in RDF/XML, causing sheXer to use an [rdflib](https://rdflib.readthedocs.io/en/stable/) parser that is less efficient and more memory-intensive compared to the parsers used for Turtle or N-Triples formats. We plan to experiment with different parsers to improve performance for these types of inputs.

  * We discussed the performance implications of sheXer being fully implemented in an interpreted language like Python. We plan to develop an equivalent implementation in a compiled language, such as Rust, to explore potential performance improvements.


# Process improvements

During this BioHackathon, we introduced several enhancements to sheXer, the library used for schema extraction. These improvements increased the quality of the extracted schemas and streamlined the workflow for schema consolidation. The key improvements include:


* Modified log output channels. Log messages are no longer sent to standard output alongside the extracted schemas, improving integration with other tools and workflows.

* Enhanced handling of blank nodes (BNodes) when mining SPARQL endpoint data. We stopped considering BNodes as potential seed nodes when sampling instance data from an endpoint. This change prevents errors or invalid SPARQL queries when attempting to retrieve information about such nodes in subsequent queries.

* Added RDF annotations at constraint level. sheXer now provides machine-readable, ShEx-compliant examples of each extracted feature, allowing integration with other tools for better documentation.

* Supported schema extraction involving BNodes from certain inputs. sheXer processes graphs locally without loading them entirely into memory or querying them via an endpoint. This approach requires traversing the graph twice, which is challenging when handling BNodes due to their lack of unique identifiers. During this hackathon, we implemented a solution that allows natural handling of BNodes in Turtle and N-Triples formats.


# Conclusions and future work

The proposed approach enables us to extract shapes from large datasets, such as UniProt, by leveraging the full range of available data. For future work, we aim to explore several areas for improvement:

* Develop scalable strategies for slicing input sources so that the subgraphs in each slice are as well-connected and complete as possible.

* Enhance schema consolidation by incorporating not only the extracted partial schemas but also relevant ontology information, such as domain, range, or expected cardinality for specific node types.

* Implement a parallelized version of this approach to reduce execution times.

* Work on identified areas to improve the performance of the individual shape extraction processes.

# Acknowledgements

We extend our sincere thanks to the organizers of BioHackathon 2024 and to the DBCLS for hosting such a valuable event, which provided a great platform to test new ideas, build collaborations, and reinforce existing ones.


# References

