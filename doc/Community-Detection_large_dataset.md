Social Network Analysis - 2019W
================
Group 18
14/01/2020

# Task 1

## Setting up enviroment, read data.

``` r
#igraph library check and load
if("igraph" %in% rownames(installed.packages()) == FALSE) {
  install.packages("igraph")
}
library("igraph")

#dplyr library check and load
if("dplyr" %in% rownames(installed.packages()) == FALSE) {
  install.packages("dplyr")
}
library(dplyr)
```

## Function for reading posting dataset and selection of colums

``` r
# --------------- read_in_postings_csv Function ----------------------

read_in_postings_csv <- function (filename) {
  
  print(filename)
  postings <- read.table(filename, header = TRUE, sep = ";", fill=TRUE)
  
  #Rename the columns for easier usage later
  names(postings)[1] <- "ID_Posting"
  names(postings)[3] <- "ID_Author"
  names(postings)[13] <- "Gender_Author"
  
  #Extract a subset from the data that only consists of interesting columns
  postings <- subset(postings, select=c("ID_Author", 
                                       "ID_Posting", 
                                       "ArticleChannel",
                                       "Gender_Author",
                                       "PostingCreatedAt"))
  
  #postings$ID_Posting = as.integer(postings$ID_Posting)
  postings$Gender_Author = as.character(postings$Gender_Author)
  
  return(postings)
}
# ------------------------ end Function--------------------------------
```

## Actually reading postings files

``` r
all_data = TRUE

if (!all_data){
  
  
  setwd("./data/Data files 01.05.2019-02.05.2019 (40,8 MB)-20191126")
  
  print("Not alles data..")
  all_postings_original <- read_in_postings_csv("Postings_01052019_02052019.csv")
  #write.csv(all_postings_original, file = "all_postings_original.csv")
  all_postings_original$PostingCreatedAt = as.Date(all_postings_original$PostingCreatedAt)
  all_postings1 <- subset(all_postings_original, PostingCreatedAt == as.Date("2019-05-01"))
  all_postings2 <- subset(all_postings_original, PostingCreatedAt == as.Date("2019-05-02")) 
  
  #Number of postings in total
  
  nrow(all_postings1) 
  nrow(all_postings2) 
  all_postings <- all_postings1
  
}else{
  
  setwd("./data/Data files 01.05.2019-31.05.2019 (660,6 MB)-20191126")
  
  print("Alles data..")
  
  all_postings1 <- read_in_postings_csv("Postings_01052019_15052019.csv")
  all_postings2 <- read_in_postings_csv("Postings_16052019_31052019.csv")
  all_postings <- rbind (all_postings1,all_postings2)
  all_postings$PostingCreatedAt = as.Date(all_postings$PostingCreatedAt)
  
  #all_votes 
  
}
```

    ## [1] "Alles data.."
    ## [1] "Postings_01052019_15052019.csv"
    ## [1] "Postings_16052019_31052019.csv"

``` r
nrow(all_postings)
```

    ## [1] 255402

## Function to read votes files

``` r
# --------------- read_in_votes_csv Function ----------------------

read_in_votes_csv <- function (filename) {
  print (filename)
  
  votes <- read.table(filename, header = TRUE, sep = ";", fill=TRUE)
  
  #Rename the ID_CommunityIdentity column because it is not read in correctly
  names(votes)[1] <- "ID_Voter"
  names(votes)[7] <- "Gender_Voter"
  
  #Extract a subset from the data that only consists of interesting columns
  votes <- subset(votes, select=c("ID_Voter", 
                                 "ID_Posting", 
                                 "VoteNegative", 
                                 "VotePositive",
                                 "Gender_Voter",
                                 "VoteCreatedAt"))
  
  votes$Gender_Voter = as.character(votes$Gender_Voter)
  
  return(votes)
  
}
# ------------------------ end Function--------------------------------
```

## Actually reading votes datasets

``` r
if (!all_data){
  setwd("./data/Data files 01.05.2019-02.05.2019 (40,8 MB)-20191126")
  
  print("Not alles data..")
  
  all_votes <- read_in_votes_csv("Votes_01052019_02052019.csv")
  #write.csv(all_votes, file="all_votes.csv")
}else{
  #read everything
  
  setwd("./data/Data files 01.05.2019-31.05.2019 (660,6 MB)-20191126")
  
  print("Alles data..")
  
  all_votes1 <- read_in_votes_csv("Votes_01052019_15052019.csv")
  all_votes2 <- read_in_votes_csv("Votes_16052019_31052019.csv")
  all_votes <- rbind (all_votes1,all_votes2)
  
}
```

    ## [1] "Alles data.."
    ## [1] "Votes_01052019_15052019.csv"
    ## [1] "Votes_16052019_31052019.csv"

``` r
all_votes$VoteCreatedAt = as.Date(all_votes$VoteCreatedAt)


#all_votes <- subset(all_votes, VoteCreatedAt == as.Date("2019-05-01"))


#Number of votes
nrow(all_votes)
```

    ## [1] 1920997

## Select by Gender, Transform and merge datasets

``` r
postings_M <<- subset(all_postings, Gender_Author == "m")
postings_W <<- subset(all_postings, Gender_Author == "w")
all_postings <- rbind(postings_M, postings_W)

votes_M <<- subset(all_votes, Gender_Voter == "m")
votes_W <<- subset(all_votes, Gender_Voter == "w")
all_votes <- rbind(votes_M, votes_W)


#Join votes and postings on ID_Posting, then remove the ID_Posting column 
joined_votes_postings <-merge(all_votes, all_postings, by = "ID_Posting")
write.csv(joined_votes_postings, file = "joined_votes_postings.csv")

gender_authors <- select(joined_votes_postings, ID_Author, Gender_Author)
colnames(gender_authors)[1] <- "ID"
colnames(gender_authors)[2] <- "Gender"
gender_voters <- select(joined_votes_postings, ID_Voter, Gender_Voter)
colnames(gender_voters)[1] <- "ID"
colnames(gender_voters)[2] <- "Gender"
#Here we merge gender_authors with gender_voters
gender_id <- rbind(gender_authors, gender_voters)
#Create a subset of the data set taking the gender
gender_id_male <- subset(gender_id, Gender =="m")
```

# Task 2

## Creating the graph function based on the data

``` r
csv_to_xml <- function (article_Channel, w = 1) {
  graph <<- graph[0,]
  filtered_joined <<- subset(joined_votes_postings, ArticleChannel == paste(article_Channel))
  unique_voters <<- unique(joined_votes_postings[["ID_Voter"]])
  
  helper <- select(filtered_joined, ID_Author, ID_Voter)
  helper <- helper[!duplicated(helper), ]
  
  for (voter_ID in unique_voters) {
    
    #get all votes that a user made
    votes_by_user <- subset(filtered_joined, ID_Voter == voter_ID)
    #get all unique IDs of authors of the posts
    unique_authors <- unique(votes_by_user[["ID_Author"]])
    
    #Iterate over all authors that a user voted for
    for (author_ID in unique_authors) {
      votes_by_user_and_author <- subset(votes_by_user, 
                                         ID_Author == author_ID)
      sum_neg <- sum(votes_by_user_and_author[["VoteNegative"]])
      sum_pos <- sum(votes_by_user_and_author[["VotePositive"]])
      
      # Here we can identify three different approaches
      
      
      weight = switch(
        w,
        sum_pos + sum_neg, #1
        abs(sum_pos - sum_neg), #2
        sum_pos - sum_neg #3
      )
      
      
      graph[nrow(graph) + 1,] <<- list(voter_ID, author_ID, weight)
      
      if (nrow(graph) %% 5000 == 0) {
        cat("...", nrow(graph), "edges added to graph.\n")
      }
    }
  }
  
  cat("Sucessfully created graph", article_Channel, "\n")
  #Import edges in igraph
  g <<- graph.data.frame(graph)
}
```

## Testing the function

``` r
#Generate dataframe for edgelist (will be reused for every ArticelChannel)
graph <- data.frame(matrix(ncol = 3, nrow = 0))
colnames(graph) <- c("voter", "author", "weight")
#Generate igraph (will be reused for every ArticleChannel)
g <- graph.data.frame(graph)


categories <- unique(joined_votes_postings$ArticleChannel)
#categories

c <- categories[8]

csv_to_xml(c)
```

    ## ... 5000 edges added to graph.
    ## ... 10000 edges added to graph.
    ## ... 15000 edges added to graph.
    ## ... 20000 edges added to graph.
    ## ... 25000 edges added to graph.
    ## ... 30000 edges added to graph.
    ## Sucessfully created graph Web

``` r
# preliminary info

is.bipartite(g)
```

    ## [1] FALSE

``` r
vcount(g)
```

    ## [1] 7986

``` r
ecount(g)
```

    ## [1] 30170

``` r
V(g)$type <- V(g)$name %in% gender_id_male$ID
V(g)$color <- ifelse(V(g)$type, "lightblue", "salmon")
V(g)$shape <- ifelse(V(g)$type, "circle", "square")
plot(g, vertex.size=5, layout=layout_nicely, vertex.label=NA)
```

<img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-8-1.png" style="display: block; margin: auto;" />

``` r
# summary(g)
# str(g)
# cat(V(g)$type)
# is.directed(g)
```

# Task 3

## Comparation function betweeen all the algorithms

``` r
######## COMPARE ###########

compare_alles_con_alles <- function (clusters_list, comp_cluster) {
  
  for (cmm in clusters_list){
    for (m in c("vi", "nmi", "split.join", "rand", "adjusted.rand"))
    {
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cim$algorithm,
                        m,
                        compare(cim, cmm, method=m)
                        )
      
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cml$algorithm,
                        m,
                        compare(cml, cmm, method=m)
                        )
      
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cwt$algorithm,
                        m,
                        compare(cwt, cmm, method=m)
                        )
      
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cfg$algorithm,
                        m,
                        compare(cfg, cmm, method=m)
                        )

      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        clp$algorithm,
                        m,
                        compare(clp, cmm, method=m)
                        )

      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cle$algorithm,
                        m,
                        compare(cle, cmm, method=m)
                        )
    }
  }
  return(comp_cluster)
}
```

## Computing the algorithms for community detection

``` r
set.seed(9103)

setwd("./output/plots_large_dataset")

i = 1

v_clusters <- list()
r_clusters <- list()

clusters_list <- list()
comp_cluster <- list()

categories <- c("Wissenschaft","Inland","Meinung","Etat","Web","Kultur")

#Wissenschaft [84], Inland [936], Meinung [223], International, Etat [63], Wirtschaft, Gesundheit, Web [530], Panorama, Kultur [270], Sport, Zukunft, Reisen

print (categories)


for (c in categories){
  
  #Time performance
  
  start_time <- Sys.time()
  csv_to_xml(c, w = 1)
  
  ud_g = as.undirected(g, mode = c("collapse"), edge.attr.comb=list(weight="sum", "ignore"))
  g <- ud_g
  
  V(g)$type <- V(g)$name %in% gender_id_male$ID
  V(g)$color <- ifelse(V(g)$type, "lightblue", "salmon")
  V(g)$shape <- ifelse(V(g)$type, "circle", "square")
  
  time <- Sys.time() - start_time
  
  png(paste(i,"_",c,"",'_graph_plot.png',sep = ""))
  
  plot(g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
       arrow.size = 0, arrow.width = 0, 
       main=(paste(i,". Channel ",c," - No clustering",sep="")),
       sub=paste("Vertices: ",vcount(g),
                 " - Edges: ",ecount(g),
                 " - Avg. degree: ", round(mean(degree(g, v= V(g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 
                 " - Time: ", round(time,3),sep=""))
  
  dev.off()
  
  plot(g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
       arrow.size = 0, arrow.width = 0, 
       main=(paste(i,". Channel ",c," - No clustering",sep="")),
       sub=paste("Vertices: ",vcount(g),
                 " - Edges: ",ecount(g),
                 " - Avg. degree: ", round(mean(degree(g, v= V(g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 
                 " - Time: ", round(time,3),sep=""))
  
  #### METRICS ####
  
  #print (paste(c," computed in: ",round(time,3)," sec",sep=""))
  
  v_clusters <- c(v_clusters,
                  c,
                  vcount(g), #Nodes
                  ecount(g), #Edges
                  mean(degree(g, v= V(g), mode = "in", normalized = FALSE)), # Average degree of the Nodes
                  "no clustering", # Algorithm
                  time, # Time
                  FALSE, # Hierarchi
                  0, # Communities
                  0, # Average of Communities Size
                  #cl$codelength,
                  0, # Total Modularity
                  0 # Modularity with Membership
                  )
  
  r_clusters <- c(r_clusters,i)
  i <- i+1
  
  
  ######## CLUSTERING #######
  ###########################
  
  #### INFOMAP ####
  
  #print(c)
  start_time <- Sys.time()
  
  
  cim = cluster_infomap(ud_g, e.weights = E(ud_g)$weight, v.weights = NULL,
                     nb.trials = 10, modularity = TRUE)
  
  cl <- cim
  
  print(paste(c," - ",cl$algorithm))
  
  time <- Sys.time()-start_time
  
  png(paste(i,"_",c,"_",cl$algorithm,'_graph_plot.png',sep = ""))
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  dev.off()
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  
  #### METRICS ####
  
  v_clusters <- c(v_clusters,
                  c,
                  vcount(ud_g), #Nodes
                  ecount(ud_g), #Edges
                  mean(degree(ud_g, v= V(ud_g), mode = "in", normalized = FALSE)), # Average degree of the Nodes
                  cl$algorithm,
                  time,
                  is_hierarchical(cl),
                  length(communities(cl)), # Communities
                  mean(sizes(cl)), # Average of Communities Size
                  #cl$codelength,
                  modularity(cl), # Total Modularity
                  modularity(ud_g, membership(cl)) # Modularity with Membership
                  )
  
  r_clusters <- c(r_clusters,i)
  i <- i+1
  
  
  #### MULTILEVEL ####
  
  
  start_time <- Sys.time()
  
  cml <- multilevel.community (ud_g, weights = E(ud_g)$weight)
  
  cl <- cml
  
  print(paste(c," - ",cl$algorithm))
  
  time <- Sys.time()-start_time
  
  png(paste(i,"_",c,"_",cl$algorithm,'_graph_plot.png',sep = ""))
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  dev.off()
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  
  #### METRICS ####
  
  v_clusters <- c(v_clusters,
                  c,
                  vcount(ud_g), #Nodes
                  ecount(ud_g), #Edges
                  mean(degree(ud_g, v= V(ud_g), mode = "in", normalized = FALSE)), # Average degree of the Nodes
                  cl$algorithm,
                  time,
                  is_hierarchical(cl),
                  length(communities(cl)), # Communities
                  mean(sizes(cl)), # Average of Communities Size
                  #cl$codelength,
                  modularity(cl), # Total Modularity
                  modularity(ud_g, membership(cl)) # Modularity with Membership
                  )
  
  r_clusters <- c(r_clusters,i)
  i <- i+1
  
  #### END OF METRICS ####
  

  #### WALKTRAP ####
  
  start_time <- Sys.time()
  
  
  cwt = cluster_walktrap(ud_g, weights = E(ud_g)$weight, steps = 4,
                        merges = TRUE, modularity = TRUE, membership = TRUE)
  
  cl <- cwt
  
  print(paste(c," - ",cl$algorithm))
  
  time <- Sys.time()-start_time
  
  png(paste(i,"_",c,"_",cl$algorithm,'_graph_plot.png',sep = ""))
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  dev.off()
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  
  #### METRICS ####
  
  v_clusters <- c(v_clusters,
                  c,
                  vcount(ud_g), #Nodes
                  ecount(ud_g), #Edges
                  mean(degree(ud_g, v= V(ud_g), mode = "in", normalized = FALSE)), # Average degree of the Nodes
                  cl$algorithm,
                  time,
                  is_hierarchical(cl),
                  length(communities(cl)), # Communities
                  mean(sizes(cl)), # Average of Communities Size
                  #cl$codelength,
                  modularity(cl), # Total Modularity
                  modularity(ud_g, membership(cl)) # Modularity with Membership
                  )
  
  r_clusters <- c(r_clusters,i)
  i <- i+1
  
  #### END OF METRICS ####
  
  
  #### FAST_GREEDY ####
  
  start_time <- Sys.time()
  
  cfg = cluster_fast_greedy(ud_g, merges = TRUE, modularity = TRUE,
                          membership = TRUE, weights = E(ud_g)$weight)
  
  cl <- cfg
  
  print(paste(c," - ",cl$algorithm))
  
  time <- Sys.time()-start_time
  
  png(paste(i,"_",c,"_",cl$algorithm,'_graph_plot.png',sep = ""))
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  dev.off()
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  #### METRICS ####
  
  v_clusters <- c(v_clusters,
                  c,
                  vcount(ud_g), #Nodes
                  ecount(ud_g), #Edges
                  mean(degree(ud_g, v= V(ud_g), mode = "in", normalized = FALSE)), # Average degree of the Nodes
                  cl$algorithm,
                  time,
                  is_hierarchical(cl),
                  length(communities(cl)), # Communities
                  mean(sizes(cl)), # Average of Communities Size
                  #cl$codelength,
                  modularity(cl), # Total Modularity
                  modularity(ud_g, membership(cl)) # Modularity with Membership
                  )
  
  r_clusters <- c(r_clusters,i)
  i <- i+1
  
  #### END OF METRICS ####
  
  #### LABEL PROPAGATION ####
  
  start_time <- Sys.time()
  
  clp = cluster_label_prop(ud_g, weights = E(ud_g)$weight , initial = NULL,
                           fixed = NULL)
  
  cl <- clp
  
  print(paste(c," - ",cl$algorithm))
  
  time <- Sys.time()-start_time
  
  png(paste(i,"_",c,"_",cl$algorithm,'_graph_plot.png',sep = ""))
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, arrow.mode = 0,
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  dev.off()
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),                 
                 " - Time: ", round(time,3),sep=""))
  
  #### METRICS ####
  
  v_clusters <- c(v_clusters,
                  c,
                  vcount(ud_g), #Nodes
                  ecount(ud_g), #Edges
                  mean(degree(ud_g, v= V(ud_g), mode = "in", normalized = FALSE)), # Average degree of the Nodes
                  cl$algorithm,
                  time,
                  is_hierarchical(cl),
                  length(communities(cl)), # Communities
                  mean(sizes(cl)), # Average of Communities Size
                  #cl$codelength,
                  modularity(cl), # Total Modularity
                  modularity(ud_g, membership(cl)) # Modularity with Membership
                  )
  
  r_clusters <- c(r_clusters,i)
  i <- i+1
  
  #### END OF METRICS ####
  
  #### LEADING EIGEN ####
  
  start_time <- Sys.time()
  
  #cle = cluster_leading_eigen(ud_g, steps = -1, weights = E(ud_g)$weight,
  #                            start = NULL, options = arpack_defaults, 
  #                            callback = NULL, extra = NULL, env = parent.frame())
  
  cle = cluster_leading_eigen(ud_g, steps = -1, weights = E(ud_g)$weight,
                              start = NULL, options = list(maxiter=1000000), #ADDED CODE TO AVOID EXCEPTION (OVERLOAD)
                              callback = NULL, extra = NULL, env = parent.frame())
  
  cl <- cle
  
  print(paste(c," - ",cl$algorithm))
  
  time <- Sys.time()-start_time
  
  png(paste(i,"_",c,"_",cl$algorithm,'_graph_plot.png',sep = ""))
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),
                 " - Time: ", round(time,3),sep=""))
  
  dev.off()
  
  plot (cl, ud_g, vertex.size=5, layout=layout_nicely, vertex.label=NA, color='darkgrey', 
        arrow.size = 0, arrow.width = 0, 
        main=(paste(i,". Channel ",c," - Algorithm ",cl$algorithm, sep="")),
        sub=paste("Vertices: ",vcount(ud_g),
                 " - Edges: ",ecount(ud_g),
                 " - Avg. degree: ", round(mean(degree(ud_g, v= V(ud_g), 
                                                    mode = "in", 
                                                    normalized = FALSE)),3),
                 " - Modularity: ", round(modularity(cl),3),
                 " - Time: ", round(time,3),sep=""))
  
  
  #### METRICS ####
  
  v_clusters <- c(v_clusters,
                  c,
                  vcount(ud_g), #Nodes
                  ecount(ud_g), #Edges
                  mean(degree(ud_g, v= V(ud_g), mode = "in", normalized = FALSE)), # Average degree of the Nodes
                  cl$algorithm,
                  time,
                  is_hierarchical(cl),
                  length(communities(cl)), # Communities
                  mean(sizes(cl)), # Average of Communities Size
                  #cl$codelength,
                  modularity(cl), # Total Modularity
                  modularity(ud_g, membership(cl)) # Modularity with Membership
                  )
  
  r_clusters <- c(r_clusters,i)
  i <- i+1
  
  clusters_list <- list(cim,cml,cwt,cfg,clp,cle)
  
  comp_cluster <- compare_alles_con_alles(clusters_list,comp_cluster)
  
}
```

<img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-1.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-2.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-3.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-4.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-5.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-6.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-7.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-8.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-9.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-10.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-11.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-12.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-13.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-14.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-15.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-16.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-17.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-18.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-19.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-20.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-21.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-22.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-23.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-24.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-25.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-26.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-27.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-28.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-29.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-30.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-31.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-32.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-33.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-34.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-35.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-36.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-37.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-38.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-39.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-40.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-41.png" style="display: block; margin: auto;" /><img src="Community-Detection_large_dataset_files/figure-gfm/unnamed-chunk-10-42.png" style="display: block; margin: auto;" />

## Exporting the metrics into a csv file

``` r
#### METRICS OF GRAPHS ####

  c_metrics <- matrix(v_clusters,ncol=11,byrow=TRUE)
  colnames(c_metrics) <- c("Channel","Nodes","Edges","Avg. degree","Algorithm","Time","Hierarchical","Communities","Avg. Size Communities","Modularity","Modularity with Membership")
  rownames(c_metrics) <- r_clusters
  c_metrics <- as.table(c_metrics)
  
  setwd("./output/target_large_dataset")
  
  write.csv(c_metrics, file=paste('clustering',"_ALL_metrics_3",'.csv', sep =''))
  colnames(c_metrics)
```

    ##  [1] "Channel"                    "Nodes"                     
    ##  [3] "Edges"                      "Avg. degree"               
    ##  [5] "Algorithm"                  "Time"                      
    ##  [7] "Hierarchical"               "Communities"               
    ##  [9] "Avg. Size Communities"      "Modularity"                
    ## [11] "Modularity with Membership"

``` r
#### METRICS OF COMPARATIONS #####

  comp_metrics <- matrix(comp_cluster,ncol=5,byrow=TRUE)
  colnames(comp_metrics) <- c("Channel","cmm1_algo","cmm2_algo","method","score")
  rownames(comp_metrics) <- c(1:dim(comp_metrics)[1])
  comp_metrics <- as.table(comp_metrics)
  
  
  #setwd("./target")
  
  write.csv(comp_metrics, file=paste('comparing',"_ALL_metrics_3",'.csv', sep =''))
  colnames(comp_metrics)
```

    ## [1] "Channel"   "cmm1_algo" "cmm2_algo" "method"    "score"

# Task 4

## Comparing metrics with different algorithms

``` r
######## COMPARE ###########

compare_alles_con_alles <- function (clusters_list, comp_cluster) {
  
  for (cmm in clusters_list){
    for (m in c("vi", "nmi", "split.join", "rand", "adjusted.rand"))
    {
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cim$algorithm,
                        m,
                        compare(cim, cmm, method=m)
                        )
      
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cml$algorithm,
                        m,
                        compare(cml, cmm, method=m)
                        )
      
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cwt$algorithm,
                        m,
                        compare(cwt, cmm, method=m)
                        )
      
      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cfg$algorithm,
                        m,
                        compare(cfg, cmm, method=m)
                        )

      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        clp$algorithm,
                        m,
                        compare(clp, cmm, method=m)
                        )

      comp_cluster <- c(comp_cluster,
                        c,
                        cmm$algorithm,
                        cle$algorithm,
                        m,
                        compare(cle, cmm, method=m)
                        )
    }
  }
  return(comp_cluster)
}
```

``` r
#5. Behaviour of the methods regarding the detected community sizes with different graph sizes.
# how the detected communities zises change when we change the graph size.
# run the algorithms on different graph sizes and use the results in tables.
```

# Task 5

## Conclusions

``` r
 print ('The conclusions are written in the report attached')
```

    ## [1] "The conclusions are written in the report attached"
