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
  - name: Andra Waagmaaster
    affiliation: 3
    orcid: 0000-0001-9773-4008
affiliations:
  - name: Weso Research Group, University of Oviedo, Spain.
    index: 1
  - name: Database Center for Life Science (DBCLS), University of Tokyo Kashiwa-no-ha Campus Station Satellite 6F. 178-4-4 Wakashiba, Kashiwa-shi, Chiba, Japan.
    index: 2
  - name: Department of Medical Informatics, University of Amsterdam, The Netherlands.
    index: 3
date: 04 September 2024
cito-bibliography: paper.bib
event: BH24
biohackathon_name: "DBCLS BioHackathon 2024"
biohackathon_url:   "http://2024.biohackathon.org/"
biohackathon_location: "Fukushima, Japan, 2024"
group: Using (discovered) schemes
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackathon-japan/bh24-getting-shapes-from-large-rdf-inputs
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: Daniel Fernández-Álvarez, Yasunori Yamamoto \emph{et al.}
---


# Introduction



Shape schemas (SHACL [@citesAsAuthority:SHACLSpec], ShEx [@citesAsAuthority:prud2014shape]) have proven to be effective tools for validating and describing RDF content. However, writing and maintaining RDF schemas can be a time-consuming task, particularly when multiple shapes are involved. To address this challenge, several approaches for automatic schema discovery have been proposed [@citesAsAuthority:fernandez2022shexer; @citesAsAuthority:Bolleman2023, @citesAsAuthority:fernandez2024extracting; @citesAsAuthority:cimmino2020astrea; @citesAsAuthority:rabbani2023extraction; @citesAsAuthority:keely2023shaclgen; @citesAsAuthority:boneva2019shape; @citesAsAuthority:mihindukulasooriya2018rdf; @citesAsAuthority:groz2022inference; @citesAsAuthority:omran2020towards; @citesAsAuthority:spahiu2018towards; @citesAsAuthority:rabbani2023shactor]. 

These approaches are especially useful when working with large RDF datasets, where manually writing, maintaining, or even understanding schemas becomes significantly more difficult. However, most existing tools for automatic schema discovery encounter performance issues or high hardware requirements when applied to large-scale datasets.

Current established methods for extracting shapes from large datasets mainly rely on sampling [@citesAsAuthority:fernandez2022shexer; @citesAsAuthority:rabbani2023extraction] or utilizing [VoID](https://www.w3.org/TR/void/) data [@citesAsAuthority:Bolleman2023]. VoID-based approaches generally offer the best performance in terms of execution time and can produce high-quality shapes, provided the VoID data itself is of sufficient quality. However, this method is limited by the need for pre-existing VoID data. On the other hand, sampling-based approaches can extract approximate shapes from any input, but the sample size can impact the quality of the results.

During a previous BioHackathon, we proposed and implemented a prototype for extracting shapes using a different strategy:

1. We split the input source into distinct slices.
2. We ran the extraction process on each slice.
3. We merged the resulting schemas into a single, unified schema.

The shape extractor used in the second step was [sheXer](https://github.com/DaniFdezAlvarez/shexer). The approach, results, and experiments conducted have been published [@citesAsAuthority:fernandez2024extracting].


In this project, we aim to refine this approach and explore new use cases for our prototype. [Our current implementation of this approach](https://github.com/shex-consolidator/shex-consolidator) is publicly available. 


# Extracted Schemas

During this hackathon, we successfully extracted schemas from a significant portion of UniProt’s graph. We used 358 different files from UniProt's dump, which we estimate to contain approximately 15.9 G triples. Speciphically, we used every file from the Uniprot release (dated tentyfifth of August, 2024) whose name starts by 'uniprotkb_reviewed_' or 'uniprotkb_unreviewed_'at  from a [Uniprot's dump](https://rdfportal.org/download/uniprot/20240826/).


We performed individual extraction processes for each file and subsequently merged the results into a single schema that describes all shapes observed within the dataset. The [shapes obtained](https://github.com/biohackathon-japan/bh24-getting-shapes-from-large-rdf-inputs/blob/main/data/uniprotkb_consolidated_358.shex) are publicly available.

We would like to highlight several facts about this experience extracting shapes form such a huge source that can help us to guide future efforts improving this approach.

* The extraction process lasted for several days. Even if the partial extractions over each data slice could be performed independently, our current prototype uses a sequential approach. This should be tackled in order to reduce execution time.

* We noticed that the resulting shapes obtained when only 262 out of 358 files were processes were identical to the one sobtained after processing the whole input data. Although this behaviour cannot be generalized to every input source, it could be useful also to integrate convergence detection mechanisms in our process to avoid exploring likely redudant data. With that we could also reduce executime time. 

* Our current prototype runs on top of sheXer extractor, which has a  influence on the proces in several ways:
  
  * sheXer is able to produce statistics of the constraints extracted, as well as examples of nodes confirming with a shape of objecs conforming with the object position in a triple constraint. However, our current prototype is not always able to keep that information while merging different versions of a shape. We have detected that the integration between sheXer and our prototype could be improved in order to keep such potentially usefil information.
  
  * Due to sheXer's internal RDF parsers when handling local data, the format of the input data can have a determinant impact on the process' performance. In this case, the data used was serialized in RDF/XML, which causes sheXer to use an [rdflib](https://rdflib.readthedocs.io/en/stable/) parser more uneficent and demanding with memory compared to the parsers used for turtle of n-triples formats. We aim to work with different parsers in order to improve the performance of this task with such types of inputs.

  * We have discussed the effect on the performance of having sheXer fully implemented in an interpreted programming language such as Python. We aim to produce an equivalent implementation in a compiled language such as Rust to experiment with that.


# Process improvements

During this BioHackathon, we introduced several enhancements to sheXer, the library used for schema extraction. These improvements enhanced the quality of the extracted schemas and streamlined the integration of different components in the workflow for schema consolidation. The key improvements include:


* Modification of log output channels. Log messages are no longer sent to the standard output, potentially alongside the extracted schemas. This change facilitates better integration with other tools and workflows.

* Handling of blank nodes (BNodes) when mining SPARQL endpoint data. We stopped considering BNodes as potential seed nodes when sampling instance data from an endpoint. This change prevents errors or invalid SPARQL queries when attempting to retrieve information about such nodes in subsequent queries.

* Use of RDF annotations at constraint-level. sheXer now provides machine-readable and ShEx-compliant examples of each extracted feature, enabling integration with other tools for improved documentation.

* Support for schema extraction with BNodes from certain inputs. sheXer processes graphs locally without requiring them to be loaded into memory or queried via an endpoint. This approach needs traversing the graph twice, posing a challenge when handling BNodes due to their lack of unique identifiers. However, during this hackathon, we implemented a solution to this issue, allowing natural handling of BNodes for Turtle and N-Triples formats.



# Conclusions and future work

The proposed approach enables us to extract shapes from large datasets, such as UniProt, by leveraging the full range of available data. Moving forward, we aim to explore several areas for improvement:

* Develop scalable strategies for slicing input sources so that the subgraphs in each slice are as well-connected and complete as possible.

* Enhance schema consolidation by incorporating not only the extracted partial schemas but also relevant ontology information, such as domain, range, or expected cardinality for specific node types.

* Implement a parallelized version of this approach to reduce execution times.

# Acknowledgements

Many thanks are due to the organisers of BioHackathon 2024 and to the DBCLS for organising an extremely useful event to test out new ideas, build new collaborations, and reinforce existing ones.


# References

