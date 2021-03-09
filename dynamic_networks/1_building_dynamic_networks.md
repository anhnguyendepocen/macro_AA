Script for building the networks for moving time window
================
Aurélien Goutsmedt and Alexandre Truc
2021-03-08

## Introduction

This script aims at creating the networks for different time windows. We
want one-year moving time windows on the whole period (1969-2016) and we
need functions automating the creation of the 44 windows. This script
creates the networks, finds communities, integrates the name of
communities, calculates coordinates and saves the nodes and edges data
in a long format, used for producing the platform.

## PART I: LOADING PACKAGES, PATH AND DATA

We first load all the functions created in the functions script. We also
have a separated script with all the packages needed for the scripts in
the `dynamic_networks` directory, as well as all the paths (for saving
pictures for instance). This separated script also loads our data and
color palettes.

``` r
source("~/macro_AA/functions/functions_for_network_analysis.R")
source("~/macro_AA/dynamic_networks/Script_paths_and_basic_objects.R")
```

## PART II: BUILDING THE NETWORKS

### Creation the networks

We prepare our list

``` r
tbl_coup_list <- list()
Limit_edges <- 400000
decile <- FALSE
quantile_threshold <- 0.85

for (Year in all_years) {
  message(paste0("Creation of the network for the ", Year, "-", Year + time_window - 1, " window."))
  # fixing the initial value of the threshold
  edges_threshold <- 1

  # creating nodes and edges
  nodes_of_the_year <- nodes_JEL[Annee_Bibliographique >= Year & Annee_Bibliographique < Year + time_window, ]
  edges_of_the_year <- edges_JEL[ID_Art %in% nodes_of_the_year$ID_Art]

  # size of nodes
  nb_cit <- edges_of_the_year[, .N, ItemID_Ref]
  colnames(nb_cit)[colnames(nb_cit) == "N"] <- "size"
  nodes_of_the_year <- merge(nodes_of_the_year, nb_cit, by.x = "ItemID_Ref", by.y = "ItemID_Ref", all.x = TRUE)
  nodes_of_the_year[is.na(size), size := 0]

  # coupling
  edges_of_the_year <- coupling_strength(edges_of_the_year, "ID_Art", "New_id2",
    weight_threshold = edges_threshold,
    output_in_character = TRUE
  )


  if (decile == TRUE) {
    edges_prob_threshold <- quantile(edges_of_the_year$weight, prob = quantile_threshold + 0.0025)
    edges_of_the_year <- edges_of_the_year[weight >= edges_prob_threshold]
  }

  # Loop to avoid to large networks - Step 2: reducing edges
  if (length(edges_of_the_year$from) > Limit_edges) {
    for (k in 1:100) {
      edges_threshold <- edges_threshold + 1

      nodes_of_the_year <- nodes_JEL[Annee_Bibliographique >= Year & Annee_Bibliographique < Year + time_window]
      edges_of_the_year <- edges_JEL[ID_Art %in% nodes_of_the_year$ID_Art]

      # size of nodes
      nb_cit <- edges_of_the_year[, .N, ItemID_Ref]
      colnames(nb_cit)[colnames(nb_cit) == "N"] <- "size"
      nodes_of_the_year <- merge(nodes_of_the_year, nb_cit, by.x = "ItemID_Ref", by.y = "ItemID_Ref", all.x = TRUE)
      nodes_of_the_year[is.na(size), size := 0]

      # coupling
      edges_of_the_year <- coupling_strength(edges_of_the_year, "ID_Art", "New_id2",
        weight_threshold = edges_threshold, output_in_character = TRUE
      )
      if (length(edges_of_the_year$from) < Limit_edges) {
        break
      }
    }
  }

  message(paste0("The final threshold for edges is:", edges_threshold))
  # remove nodes with no edges
  nodes_of_the_year <- nodes_of_the_year[ID_Art %in% edges_of_the_year$from | ID_Art %in% edges_of_the_year$to]
  nodes_of_the_year$ID_Art <- as.character(nodes_of_the_year$ID_Art)
  edges_of_the_year$threshold <- edges_threshold

  # make tbl
  tbl_coup_list[[as.character(Year)]] <- networkflow::tbl_main_component(nodes = nodes_of_the_year, edges = edges_of_the_year, directed = FALSE, node_key = "ID_Art", nb_components = 2)

  gc() # cleaning what is not useful anymore
}

# cleaning now useless objects
rm(list = c("edges_JEL", "nb_cit", "edges_of_the_year", "nodes_JEL", "nodes_of_the_year"))
```

### Finding Communities

We use the leiden\_workflow function of the networkflow package (it uses
the leidenAlg package).

``` r
tbl_coup_list <- lapply(tbl_coup_list, leiden_workflow)
# list_graph <- lapply(list_graph, FUN = community_colors, palette = mypalette)

# intermediary saving
saveRDS(tbl_coup_list, paste0(graph_data_path, "list_graph_", first_year, "-", last_year + time_window - 1, ".rds"))
```

### Running force atlas

``` r
tbl_coup_list <- readRDS(paste0(graph_data_path, "list_graph_", first_year, "-", last_year + time_window - 1, ".rds"))

list_graph_position <- list()

for (Year in all_years) {
  message(paste0("Running Force Atlas for the ", Year, "-", Year + time_window - 1, " window."))
  if (is.null(tbl_coup_list[[paste0(Year - 1)]])) {
    list_graph_position[[paste0(Year)]] <- force_atlas(tbl_coup_list[[paste0(Year)]], kgrav = 4, change_size = FALSE)
  }
  if (!is.null(tbl_coup_list[[paste0(Year - 1)]])) {
    past_position <- list_graph_position[[paste0(Year - 1)]] %>%
      activate(nodes) %>%
      as.data.table()
    past_position <- past_position[, .(ID_Art, x, y)]
    tbl <- tbl_coup_list[[paste0(Year)]] %>%
      activate(nodes) %>%
      left_join(past_position)
    list_graph_position[[paste0(Year)]] <- force_atlas(tbl, kgrav = 4, change_size = FALSE)
  }
  # saveRDS(tbl_coup_list, paste0(graph_data_path,"coupling_graph_",Year,"-",Year+time_window-1,".rds"))
  saveRDS(list_graph_position[[paste0(Year)]], paste0(graph_data_path, "coupling_graph_", Year, "-", Year + time_window - 1, ".rds"))
  gc()
}

saveRDS(list_graph_position, paste0(graph_data_path, "list_graph_", first_year, "-", last_year + time_window - 1, ".rds"))
```

### Integrating Community names (temporary)

``` r
# Listing all the graph computed in `static_network_analysis.R`
all_nodes <- data.table("Id" = c(), "Annee_Bibliographique" = c(), "Titre" = c(), "Label" = c(), "color" = c(), "Community_name" = c())
for (i in 1:length(start_date)) {
  graph <- readRDS(paste0(graph_data_path, "graph_coupling_", start_date[i], "-", end_date[i], ".rds"))
  graph <- graph %>%
    activate(nodes) %>%
    select(Id, Annee_Bibliographique, Titre, Label, color, Community_name) %>%
    as.data.table()
  all_nodes <- rbind(all_nodes, graph)
}

for (Year in all_years) {
  nodes <- list_graph_position[[paste0(Year)]] %>%
    activate(nodes) %>%
    select(ID_Art, Annee_Bibliographique, Titre, Label, Com_ID) %>%
    as.data.table()
  communities <- all_nodes[between(Annee_Bibliographique, Year, Year + 4)]

  communities <- merge(nodes, communities[, c("Id", "color", "Community_name")], by.x = "ID_Art", by.y = "Id")
  communities <- communities[, size_com := .N, by = "Com_ID"][, .N, by = c("Com_ID", "size_com", "Community_name", "color")]

  communities <- communities %>%
    group_by(Com_ID) %>%
    arrange(-N) %>%
    mutate(share = N / size_com) %>%
    select(Com_ID, Community_name, color, share) %>%
    slice(1)

  list_graph_position[[paste0(Year)]] <- list_graph_position[[paste0(Year)]] %>%
    activate(nodes) %>%
    left_join(communities)

  # Mix color for edges of different color
  list_graph_position[[paste0(Year)]] <- list_graph_position[[paste0(Year)]] %>%
    activate(edges) %>%
    mutate(color_com_ID_to = .N()$color[to], color_com_ID_from = .N()$color[from]) %>%
    mutate(color_edges = DescTools::MixColor(color_com_ID_to, color_com_ID_from, amount1 = 0.5))

  # Cleaning progressively
  gc()
}

saveRDS(list_graph_position, paste0(graph_data_path, "list_graph_", first_year, "-", last_year + time_window - 1, ".rds"))
```

### Projecting graphs

This step is not necessary for producting the data for the online
platform. If you want to run this part, set `run` to “TRUE”.

``` r
run = FALSE

if(run == TRUE){
list_graph_position <- readRDS(paste0(graph_data_path, "list_graph_", first_year, "-", last_year + time_window - 1, ".rds"))

com_label <- list()

for (Year in all_years) {
  com_label[[paste0(Year)]] <- label_com(list_graph_position[[paste0(Year)]], biggest_community = TRUE, community_threshold = 0.01)
}

list_ggplot <- list()
for (Year in all_years) {
  list_ggplot[[as.character(Year)]] <- ggraph(list_graph_position[[paste0(Year)]], "manual", x = x, y = y) +
    geom_edge_link0(aes(color = color_edges, width = weight), alpha = 0.5) +
    geom_node_point(aes(fill = color, size = size), pch = 21) +
    scale_edge_width_continuous(range = c(0.1, 0.5)) +
    scale_size_continuous(range = c(0.1, 3)) +
    theme_void() +
    # new_scale("size") +
    geom_label_repel(data = com_label[[paste0(Year)]], aes(x = x, y = y, label = Community_name, fill = color), size = 0.5, fontface = "bold", alpha = 0.9, point.padding = NA, show.legend = FALSE) +
    theme(legend.position = "none") +
    scale_fill_identity() +
    scale_edge_colour_identity() +
    labs(title = paste0(as.character(Year), "-", as.character(Year + time_window - 1)))
  # ggsave("Networks/coup_2000.png", width=30, height=20, units = "cm")
}


benchmark <- do.call(grid.arrange, list_ggplot[c(1, 5, 9, 13, 17, 22)])
ggsave(paste0(picture_path, "benchmark_edge_threshold_3.png"), benchmark, width = 30, height = 30, unit = "cm")

ggsave(plot = g, paste0("Graphs/", author, "_networks.png"), width = 30, height = 40, units = "cm")

library(ggpubr)
for (i in c(1, 9, 18, 27, 35)) {
  g <- ggarrange(plotlist = list_ggplot[i:(i + 7)], common.legend = TRUE, legend = "none")
  ggsave(plot = g, paste0(picture_path, "Graph_", i), width = 30, height = 40, units = "cm")
  gc()
}
}
```

## PART II: EXTRACTING NETWORKS DATA FOR THE PLATFORM

### Transforming in long format

``` r
# loading the data
list_graph_position <- readRDS(paste0(graph_data_path, "list_graph_", first_year, "-", last_year + time_window - 1, ".rds"))

# creating a table with the data for nodes and edges for each window

nodes_lf <- lapply(list_graph_position, function(tbl) (tbl %>% activate(nodes) %>% as.data.table()))
nodes_lf <- lapply(nodes_lf, function(dt) (dt[, .(ID_Art, x, y, size, Com_ID, color)]))
nodes_lf <- rbindlist(nodes_lf, idcol = "window")
nodes_lf <- nodes_lf[, window := paste0(window, "-", as.integer(window) + 4)][order(ID_Art, window)]

edges_lf <- lapply(list_graph_position, function(tbl) (tbl %>% activate(edges) %>% as.data.table()))
edges_lf <- lapply(edges_lf, function(dt) (dt[, .(Source, Target, weight, Com_ID, color_edges)]))
edges_lf <- rbindlist(edges_lf, idcol = "window")
edges_lf <- edges_lf[, window := paste0(window, "-", as.integer(window) + 4)]

nodes_info <- nodes_JEL[ID_Art %in% unique(nodes_lf$ID_Art)][, c("ID_Art", "Titre", "Annee_Bibliographique", "Nom", "Label", "Revue", "ESpecialite")]

write_csv(nodes_lf, paste0(platform_data, "nodes_lf.csv"))
write_csv(edges_lf, paste0(platform_data, "edges_lf.csv"))
write_csv(nodes_info, paste0(platform_data, "nodes_info.csv"))
```