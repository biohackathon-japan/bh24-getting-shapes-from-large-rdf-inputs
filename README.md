# Getting shapes from large rdf inputs

## Background
In this project we aim to improve the current mechanism to get shapes from large RDF inputs. We focus on practical uses cases related to LifeSciences and data handleded in some way by DBCLS. 

Current "mature" approaches for doing that are largely based on sampling or interpretation of VOID data. The approach based on void seems to be performant, both in terms on the quality of the obtained schemes and the speed of the process itself. However, the VOID data should exist in order to be able to get the data schemes. 
On the other hand, sampling approaches may suffer fomr some data lost. 

In a previos BioHackathon, we were able to propose and produce a prototype code for extracting shapes in a different way.

1) We split the input source in different slices.
2) We run an extration process in the different slices.
3) We merge the schemes obtained in a single one.

## Sources from which we were able to extract schemes

During this hackathon, we have been able to obtain schemes using a large portion of Uniprot's grpah, using 86 files of its rdf dumps. Also, we have fully implemented or started the implementation of several features in sheXer, the extraction tool used to perform the shape extraction itself. These features are the following ones:

## New features in the tools used for the extraction process.

- IMPLEMENTED: Modification of log output channels, so those messages are not going trough the standar output along (potentially) with the extracted schemes. This allows for a better integration of the process with other tools.
- IMPLEMENTED: Change w.r.t. handling BNodes when minign information from SPARQL endpoints. We stop considering them as potential seed nodes when sampling instance data, as this could leed to errors or invalid SPARQL queries when tryin to retrieve information about such nodes in other SPARQL queries.
- IMPLEMENTED: Use of RDF annotations at constraint level within the produced shapes. With this, sheXer is able to provide a machine readable and ShEx compliant example of each extracted feature, which allows to feed further products for better documentation.

- WORK ON PROGRESS: Full support to schema extraction with BNodes with certain types of input. The way in which sheXer processes the graph locally does not require to load the graph in memory nor any endpoint, but, as a trade-off, the graph should be walked twice. This introduce a hard to deal with issue regarding BNodes, as they lack an indentifier and you can'e ensure with most parsers to be able to reference in a consistat monner such nodes on different walks. The code is not completed yet, but during the biohackathon we figure out a way to handle this issue as long as the input meets certain conditions.

## Future work
We want to finish the work initiated regarding implementation during this hackathon as well as working in a scalable and efficient strategy to perform input slicing. 

