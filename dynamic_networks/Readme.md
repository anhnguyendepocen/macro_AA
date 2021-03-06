Readme of the `dynamic_networks` directory
================
Aurélien Goutsmedt and Alexandre Truc
/ Last compiled on 2021-03-11

## What will you find in this directory?

Here are all the scripts for creating “dynamic” networks. By “dynamic”,
we mean to create several networks for small time windows and move these
windows year by year. In other words, we are creating networks for
1969-1973, 1970-1974, 1971-1975, until 2011-2015. We then identify
communities that are persisting over time, and give them names.

This directory contains:

### Script\_paths\_and\_basic\_objects

As in other directories, there is a
[Script\_paths\_and\_basic\_objects.R](/dynamic_networks/Script_paths_and_basic_objects.R)
with all the packages needed in the directory, and essential path and
data. This script is loaded in other scripts;

### 1\_building\_dynamic\_networks

The script
[1\_building\_dynamic\_networks](/dynamic_networks/1_building_dynamic_networks.md)
creates all the networks for each window, then find communities and
calculates coordinates. All the networks are fill in a list, and the
list is saved. The script also saves separately all the nodes and edges
in a long format;

### 2\_building\_dynamic\_communities

The script
[2\_building\_dynamic\_communities](/dynamic_networks/2_building_dynamic_communities.md)
takes as an input the list of networks saved in the previous script. The
script implements a procedure to find communities that are persisting
over time. Basically, for each community in two close networks (for
instance 1973-1977 and 1974-1978), it looks how many nodes existing in
both networks (that is articles published between 1974 and 1977) are in
each community. If a community A from 1973-1977 has a high percentage of
nodes going in community B from 1974-1978, and if a large proportion of
nodes in community B comes from A, thus we consider A and B as the same
community. The output is a data frame with all the nodes of the corpus,
and their respective communities for each time window.

### 3\_introducing\_communities\_names

The script
[3\_introducing\_communities\_names](/dynamic_networks/3_introducing_communities_names.md)
takes as an input the list of nodes and their communities saved in the
former script, and introduces the names of the communities. These names
have been created by a qualitative assessment of the characteristics of
each community.

### naming\_communities

The Rmarkdown file
[“naming\_communities”](/dynamic_networks/naming_communities.Rmd)
produces an html document with the main characteristics of all the
communities identified in
[2\_building\_dynamic\_communities](/dynamic_networks/2_building_dynamic_communities.md).
This document will help to name communities qualitatively.

### testing\_threshold\_for\_building\_networks

The Rmarkdown file
[“testing\_threshold\_for\_building\_networks”](/dynamic_networks/testing_threshold_for_building_networks.Rmd)
runs some tests on the effects of the different thresholds implemented
in
[1\_building\_dynamic\_networks](/dynamic_networks/1_building_dynamic_networks.md).
