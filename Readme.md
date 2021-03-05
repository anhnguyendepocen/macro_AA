Presentation of the Mapping Macroeconomics Project
================
Aurélien Goutsmedt and Alexandre Truc

## What is the Mapping Macroeconomics Project

This project has received a grant from the *History of Economics Sociey*
[New Initiatives
Fund](https://historyofeconomics.org/about-the-society/new-initiatives/).
It is conducted by [Aurélien Goutsmedt](aurelien-goutsmedt.com) and
[Alexandre
Truc](https://sites.google.com/view/alexandre-truc/home-and-contact).
The full project demand can be find
[here](aurelien-goutsmedt.com/project/mapping-macroeconomics/proposal_hes.pdf).
Several collaborators are likely to join us progressively.

It takes as a point of departure that economics has been characterized
since the 1970s by an exponential increase in the number of articles
published in academic journals. This phenomenon makes it harder for
historians of economics to properly assess the trends in the
transformation of economics, the main topics researched, the most
influential authors and ideas, *etc*. As a result, historians of this
period run the risk of relying, at least as first pass, on a presentist
view of the history of ideas: what today’s economists consider as
important contributions from the past and the most influential authors.
When historians shield themselves from the retrospective views of the
practising economists, they have to delve into the literature of the
past that has been forgotten for the most part and to scrutinize the
network of authors that are little known. This effort allows historians
to correct the potted histories of the practising economists and to
propose alternative narratives of the evolution of the discipline.
However, the bigger the literature to be studied is, the harder the
historians’ capacity to grasp the overall picture of the time.

We agree with those who consider that developing collective quantitative
tools could help historians to confront this challenge. The issues and
uses of quantitative tools in history of economics have been the topic
of many discussions in our field—see for instance @backhouse1998,
@claveau2016 and the *Journal of Economic Methodology* special issue in
2018, in particular @edwards2018a and @cherrier2018a—, but the technical
barrier posed by the methods remains an obstacle for many historians.
The challenges and opportunities that a quantitative history brings are
particularly useful to the recent history of macroeconomics. Practising
macroeconomists are eager to tell narratives of the evolution of their
field that serve the purpose of intervening on current debates, by
giving credit to particular authors and weight to specific ideas.
Historians who go into this area find plenty of accounts by
macroeconomists and have to handle the vast increase in the
macroeconomic literature since the last quarter of the past century.

Our goal is to build an online interactive platform displaying
bibliometric data on a large set of macroeconomic articles. We aim at
settling the basis for a broad and long-run project on the history of
macroeconomics, as well as to bring to historians tools to run
quantitative inquiries to support their own research work.

## Structure of the Github repository

### Corpus

You will find here all the scripts and documentation relative to the
extractiong of the JEL codes from Econlit and the matching with Web of
Science database. It deals with generating the list of articles,
references, authors, affiliations, *etc.* that will be used after.

### Static\_Network\_Analysis

This file gathers all the scripts and documentation for the first step
of the project:

  - testing different tools and methods
  - first round of analysis of our data by segmenting them in several
    sub-periods. We build different networks (coupling, cocitation,
    co-authorship, *etc*.) and analyse them for each sub-period.

### dynamic\_networks

This file gathers all the scripts and documentation for the production
of data for the online platform. It deals with:

  - creating networks with a moving window;
  - finding communities that remain over the windows.

### Interactive\_networks

This file gathers some tests with interactive networks. Not really
useful for our project and not at all for the platform, but it allows us
to test different packages to produce interactives networks, that could
be useful in communicating on our project or to explore our
data/networks.

### EER\_Paper

Here are all the scripts and docs used by Aurélien and Alexandre for
their paper on the *European Economic Review*.
