---
title: 'BioHack24 report: Using discovered RDF schemes: a compilation of potential use cases for shapes reusage'
title_short: 'BioHack24 report: Using discovered RDF schemes'
tags:
  - RDF
  - ShEx
  - SHACL
  - Schema
  - Automatic extraction
  - Large inputs
authors:
  - name: Daniel Fernández-Álvarez
    affiliation: 1
    orcid: 0000-0000-0000-0000
  - name: Jerven Bolleman
    affiliation: 2
    orcid: 0000-0000-0000-0000
  - name: Jose Emilio Labra Gayo
    affiliation: 1
    orcid: 0000-0000-0000-0000
  - name: Yasunori Yamamoto
    affiliation: 2
    orcid: 0000-0000-0000-0000
  - name: Andra Waagmaaster
    affiliation: 2
    orcid: 0000-0000-0000-0000
affiliations:
  - name: Weso Research Group, University of Oviedo, Spain
    index: 1
  - name: Second Affiliation
    index: 2
date: 04 September 2024
cito-bibliography: paper.bib
event: BH24
biohackathon_name: "DBCLS BioHackathon 2024"
biohackathon_url:   "http://2024.biohackathon.org/"
biohackathon_location: "Fukushima, Japan, 2024"
group: Project 26
# URL to project git repo --- should contain the actual paper.md:
git_url: https://github.com/biohackrxiv/publication-template
# This is the short authors description that is used at the
# bottom of the generated paper (typically the first two authors):
authors_short: First Author \emph{et al.}
---


# Introduction



Shape schemes (SHACL, ShEx) have proben to be effective tools to validate and describe RDF content. However, writing and maintaining RDF schemas can be a time-consuming task, especially when many different shapes are involved. To address this issue, several approaches for automatic schema discovery have been proposed (CITE sheXer, Jerven’s [tool name! how to cite? @Jerven], Consolidation, Astrea, QSE...). 

Those approaches are specially useful when handling large RDF datasources, as in such scenarios writing, maintaining or even knowing the schemes becomes a more challeging task. However, most of the existing tools to perform automatic schema discoverage run into different issues related to eprformance or hardware requirements when used in such scenasrios.

Current "mature" approaches to extract shapes form large inputs ar largely on sampling or interpretation of VOID data. The approach based on VOID is the best performant one both in terms on the quality of the obtained schemes and the speed of the process itself. However, the VOID data should exist in order to be able to get the data schemes. On the other hand, sampling-based approaches can extract approximate shapes from any input, but the sample size may affect the quality of the results .


In a previous BioHackathon, we were able to propose and imeplement a prototype for extracting shapes using a different strategy:

1. We split the input source in different slices.
2. We run an extration process in the different slices.
3. We merge the schemes obtained in a single one.

The shape extractor used in the second step is sheXer. The approach, results, and experiments performed has been published [CITA AQUÍ AL SWAT4HCLS]


In this project, we aim to improve this approach as well as find new usages for our prototype.

In a previos BioHackathon, we were able to propose and produce a prototype code for extracting shapes in a different way.


# Schemes extracted

During this hackathon, we have been able to obtain schemes from a large protion of Uniprot's graph. We have used as input 308 different files from Uniprot's dump, which we estimate to contain 15.9G triples. We have run an individual extraction process over each file and then we have been able to merge the result in a single schema described all shapes observed among such data. Each partial extraction was executed sequentially, althoguh this approache could be trivially parallelized. In this case, the execution of every extraction was running for more than 2 days in a server ... [should we describe the machine or detail the execution time?] @Yasunori.

The results obtained are publicly available [once I fix the consolidator w.r.t. annotations, maybe we should place the shapes somewhere and share them. Could be in this repository.]@Yasunori

## Process improvements

During this BioHackathon, we have been able to add new features in sheXer, the library used for the schema extraction. Such improvements cause an improvement on the obtained schemas or the integration of different pieces in the workflow that allows us to consolidate schemes. The new features are the following ones:



* Modification of log output channels, so those messages are not going trough the standar output along (potentially) with the extracted schemes. This allows for a better integration of the process with other tools.

* Change w.r.t. handling BNodes when minign information from SPARQL endpoints. We stop considering them as potential seed nodes when sampling instance data, as this could leed to errors or invalid SPARQL queries when tryin to retrieve information about such nodes in other SPARQL queries.

* Use of RDF annotations at constraint level within the produced shapes. With this, sheXer is able to provide a machine readable and ShEx compliant example of each extracted feature, which allows to feed further products for better documentation.

* Full support to schema extraction with BNodes with certain types of input. The way in which sheXer processes the graph locally does not require to load the graph in memory nor any endpoint. However, as a trade-off, the graph should be walked twice. This need introduces an issue regarding BNodes, as they lack an indentifier and one cannot ensure with most parsers to be able to reference a certain BNode in a consistat manner in different walsk of a graph. Nevertheless, during this hackathon, we were able to implement a sollution to this problem able to naturally handle BNodes for turtle and NTriples sintaxes. 



## Conclusions and future work

The approach proposed allows us to extract shapes using the whole information in large sources such as UniProt. We have several lines of future work for this approach:

* Propose scalable and adequate startegies to slice the input source, so the subgraphs of each slice are as well-connected and complete as possible.

* Complement the schema consolidation not only using the partial schemes extracted, but also ontology information related to domain, range or expected cardinality for certain types of nodes.

* Provide a parallelized implementation of this approach to reduce execution times

## Acknowledgements

[...]

## References

[...]
