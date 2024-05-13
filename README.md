# BRIC_LED_AWG
BRIC_LED_AWG


**This R markdown file was auto-generated by the iDEP shiny application.**

knitr::opts_chunk$set(echo = TRUE)
knitr::opts_chunk$set(fig.width=6, fig.height=5, fig.align = 'center') 


**1. Read data**
First we set up the working directory to where the files are saved.

setwd('C:/Users/DRB/Downloads')   # Needs to be changed to your drive 

R packages and iDEP core Functions. 
Install all the function in the iDEP_core_functions.R file. 

 if(file.exists('iDEP_core_functions.R'))
	source('iDEP_core_functions.R') else 
    source('https://raw.githubusercontent.com/iDEP-SDSU/idep/master/shinyapps/idep/iDEP_core_functions.R') 

We are using the downloaded gene expression file where gene IDs has been converted to Ensembl gene IDs. 

 inputFile <- 'Downloaded_Converted_Data.csv'  # Expression matrix
 sampleInfoFile <- 'Downloaded_sampleInfoFile.csv' # Experiment design file 
 geneInfoFile <- 'Arabidopsis_thaliana__athaliana_eg_gene_GeneInfo.csv' #Gene symbols, location etc. 
 geneSetFile <- 'Arabidopsis thaliana__athaliana_eg_gene.db'  # pathway database in SQL; can be GMT format 
 STRING10_speciesFile <- 'https://raw.githubusercontent.com/iDEP-SDSU/idep/master/shinyapps/idep/STRING10_species.csv' 

# Parameters for reading data

 input_missingValue <- 'geneMedian'	#Missing values imputation method
 input_dataFileFormat <- 1	#1- read counts, 2 FKPM/RPKM or DNA microarray
 input_minCounts <- 0.5	#Min counts
 input_NminSamples <- 1	#Minimum number of samples 
 input_countsLogStart <- 4	#Pseudo count for log CPM
 input_CountsTransform <- 1	#Methods for data transformation of counts. 1-EdgeR's logCPM 2-VST, 3-rlog 

 readData.out <- readData(inputFile) 
 library(knitr)   #  install if needed. for showing tables with kable 
 kable( head(readData.out$data) )    # show the first few rows of data 
 readSampleInfo.out <- readSampleInfo(sampleInfoFile) 
 kable( readSampleInfo.out )  
 input_selectOrg ="NEW" 
 input_selectGO <- 'KEGG'	#Gene set category 
 input_noIDConversion = TRUE  
 allGeneInfo.out <- geneInfo(geneInfoFile) 
 converted.out = NULL 
 convertedData.out <- convertedData()	 
 nGenesFilter()  
 convertedCounts.out <- convertedCounts()  # converted counts, just for compatibility 
 
**2. Pre-process**
# Read counts per library 
 parDefault = par() 
 par(mar=c(12,4,2,2)) 
 # barplot of total read counts
 x <- readData.out$rawCounts
 groups = as.factor( detectGroups(colnames(x ) ) )
 if(nlevels(groups)<=1 | nlevels(groups) >20 )  
  col1 = 'green'  else
  col1 = rainbow(nlevels(groups))[ groups ]				
		 
 barplot( colSums(x)/1e6, 
		col=col1,las=3, main="Total read counts (millions)")  
 readCountsBias()  # detecting bias in sequencing depth 
![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/37abe84a-c194-4ad7-bd77-cda27751750f)


 # Box plot 
 x = readData.out$data 
 boxplot(x, las = 2, col=col1,
    ylab='Transformed expression levels',
    main='Distribution of transformed data') 
![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/0a75b313-ebb3-4f4c-bd31-78aeb8be3bbd)


 # Density plot 
 par(parDefault) 
 densityPlot()   
 ![density plot](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/sd_density_plot.png "density plot")

 # Scatter plot of the first two samples 
 plot(x[,1:2],xlab=colnames(x)[1],ylab=colnames(x)[2], 
    main='Scatter plot of first two samples') 
    ![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/ecf4e0d8-5adb-4da2-b7ce-025c8c994352)

# Dispersion plot 
![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/197938c9-c276-42e8-8ca6-f911bad1bf6f)


# QC PLots 
![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/bb5a3516-6ded-41ff-a6a5-5738114e0b28)

![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/d16cdb3a-ed81-4089-b27d-5e6f6ea0cb2b)


 #### plot gene or gene family
 input_selectOrg ="BestMatch" 
 input_geneSearch <- 'SNCA;Robo3;GAPDH'	#Gene ID for searching 
 genePlot()  
 input_useSD <- 'FALSE'	#Use standard deviation instead of standard error in error bar? 
 geneBarPlotError()       

![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/19de2300-fb7b-42fe-92ba-29b4405d2f15)

Random selectino of mitochochonrial genes
![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/45e4a787-d89a-4d10-8657-fe5f8a24f717)


**3. Heatmap**

 # hierarchical clustering tree
 x <- readData.out$data
 maxGene <- apply(x,1,max)
 # remove bottom 25% lowly expressed genes, which inflate the PPC
 x <- x[which(maxGene > quantile(maxGene)[1] ) ,] 
 plot(as.dendrogram(hclust2( dist2(t(x)))), ylab="1 - Pearson C.C.", type = "rectangle") 
 **#Correlation matrix**
 input_labelPCC <- TRUE	#Show correlation coefficient? 
 correlationMatrix() 

  # Parameters for heatmap
 input_nGenes <- 300	#Top genes for heatmap
 input_geneCentering <- TRUE	#centering genes ?
 input_sampleCentering <- FALSE	#Center by sample?
 input_geneNormalize <- TRUE	#Normalize by gene?
 input_sampleNormalize <- FALSE	#Normalize by sample?
 input_noSampleClustering <- FALSE	#Use original sample order
 input_heatmapCutoff <- 4	#Remove outliers beyond number of SDs 
 input_distFunctions <- 1	#which distant function to use
 input_hclustFunctions <- 1	#Linkage type
 input_heatColors1 <- 2	#Colors
 input_selectFactorsHeatmap <- 'Treatment'	#Sample coloring factors 
 png('heatmap.png', width = 10, height = 15, units = 'in', res = 300) 
 staticHeatmap() 
 dev.off()  
![heatmap] (heatmap.png)

 ![heatmap1](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/heatmap_main.png "heatmap1")


 ![heatmap zoom]( https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/heatmap_zoom.png "heatmap2")


 heatmapPlotly() # interactive heatmap using Plotly 
 
**4. K-means clustering**
 input_nGenesKNN <- 2000	#Number of genes fro k-Means
 input_nClusters <- 4	#Number of clusters 
 maxGeneClustering = 12000
 input_kmeansNormalization <- 'geneMean'	#Normalization
 input_KmeansReRun <- 0	#Random seed 
 distributionSD()  #Distribution of standard deviations 
 KmeansNclusters()  #Number of clusters 
 Kmeans.out = Kmeans()   #Running K-means 
 KmeansHeatmap()   #Heatmap for k-Means 

 ![heatmap ER cluster](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/heatmap_zoom%20(ER_cluster).png "heatmap ER cluster") 

**#Read gene sets for enrichment analysis**
 sqlite  <- dbDriver('SQLite')
 input_selectGO3 <- 'GOBP'	#Gene set category
 input_minSetSize <- 5	#Min gene set size
 input_maxSetSize <- 2000	#Max gene set size 
 GeneSets.out <-readGeneSets( geneSetFile,
    convertedData.out, input_selectGO3,input_selectOrg,
    c(input_minSetSize, input_maxSetSize)  )  
 
 #GeneSets.out <- readGMTRobust('somefile.GMT')  
 results <- KmeansGO()  #Enrichment analysis for k-Means clusters	
 results$adj.Pval <- format( results$adj.Pval,digits=3 )
 kable( results, row.names=FALSE) 
 input_seedTSNE <- 0	#Random seed for t-SNE
 input_colorGenes <- TRUE	#Color genes in t-SNE plot? 
 tSNEgenePlot()  #Plot genes using t-SNE 

**5. PCA and beyond**

 input_selectFactors <- 'Treatment'	#Factor coded by color
 input_selectFactors2 <- 'Treatment'	#Factor coded by shape
 input_tsneSeed2 <- 0	#Random seed for t-SNE 
 #PCA, MDS and t-SNE plots
 PCAplot()  
 ![BRIC-LED_shoots_RNAseq_PCA](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/PCA_with_loadings.png "PCA")

![BRIC-LED_shoots_RNAseq_PCA scree](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/scree.png "PCA scree")
 
 MDSplot() 
 
 tSNEplot()  

**#Read gene sets for pathway analysis using PGSEA on principal components**
 input_selectGO6 <- 'All'	#Gene set category 
 GeneSets.out <-readGeneSets( geneSetFile,
    convertedData.out, input_selectGO6,input_selectOrg,
    c(input_minSetSize, input_maxSetSize)  )  
 PCApathway() # Run PGSEA analysis 
 cat( PCA2factor() )   #The correlation between PCs with factors 
 
**6. DEG1**
 input_CountsDEGMethod <- 3	#DESeq2= 3
 input_limmaPval <- 0.1	#FDR cutoff
 input_limmaFC <- 2	#Fold-change cutoff
 input_selectModelComprions <- 'Treatment: Flight vs. Ground'	#Selected comparisons
 input_selectFactorsModel <- 'Treatment'	#Selected comparisons
 input_selectInteractions <- NULL 	#Selected comparisons
 input_selectBlockFactorsModel <- NULL 	#Selected comparisons
 factorReferenceLevels.out <- c('Treatment:Flight') 

 limma.out <- limma()
 DEG.data.out <- DEG.data()
 limma.out$comparisons 
 input_selectComparisonsVenn = limma.out$comparisons[1:3] # use first three comparisons
 input_UpDownRegulated <- FALSE	#Split up and down regulated genes 
 vennPlot() # Venn diagram 
  sigGeneStats() # number of DEGs as figure 
  sigGeneStatsTable() # number of DEGs as table 

![image](https://github.com/dr-richard-barker/BRIC_LED_AWG/assets/8679982/56e7f5d2-3d02-4dbf-9415-2c1f53a8fb4f)


**7. DEG2**
 input_selectContrast <- 'Flight-Ground'	#Selected comparisons 
 selectedHeatmap.data.out <- selectedHeatmap.data()
 selectedHeatmap()   # heatmap for DEGs in selected comparison


 # Save gene lists and data into files
 write.csv( selectedHeatmap.data()$genes, 'heatmap.data.csv') 
 write.csv(DEG.data(),'DEG.data.csv' )
 write(AllGeneListsGMT() ,'AllGeneListsGMT.gmt')
 
 input_selectGO2 <- 'GOBP'	#Gene set category 
 geneListData.out <- geneListData()  
 volcanoPlot()  
  ![volcano_plot](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/volcano_plot.png "volcano_plot")
  
  scatterPlot()  
   ![scatter_plot](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/scatter_plot.png "scatter_plot")
   
  MAplot()  
   ![ma_plot](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/ma_plot.png "ma_plot")
  geneListGOTable.out <- geneListGOTable()  


 
 # Read pathway data again 
 GeneSets.out <-readGeneSets( geneSetFile,
    convertedData.out, input_selectGO2,input_selectOrg,
    c(input_minSetSize, input_maxSetSize)  ) 
 input_removeRedudantSets <- TRUE	#Remove highly redundant gene sets? 
 results <- geneListGO()  #Enrichment analysis
 results$adj.Pval <- format( results$adj.Pval,digits=3 )
 kable( results, row.names=FALSE) 

 ![defence cluster](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/enrichemnt_botton_cluser_defensecemetabolism.png "defence cluster")
  
 
**Enrichment analysis using STRING**

  STRINGdb_geneList.out <- STRINGdb_geneList() #convert gene lists
 input_STRINGdbGO <- 'Process'	#'Process', 'Component', 'Function', 'KEGG', 'Pfam', 'InterPro' 
 results <- stringDB_GO_enrichmentData()  # enrichment using STRING	
 results$adj.Pval <- format( results$adj.Pval,digits=3 )
 kable( results, row.names=FALSE) 
PPI network retrieval and analysis

 input_nGenesPPI <- 100	#Number of top genes for PPI retrieval and analysis 
 stringDB_network1(1) #Show PPI network 
Generating interactive PPI

 write(stringDB_network_link(), 'PPI_results.html') # write results to html file 
 browseURL('PPI_results.html') # open in browser 

![BRIC-LED_shoots_RNAseq_FLvsGC_metabolism](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/BRIC-LED_shoots_RNAseq_FLvsGC_metabolism.png "BRIC-LED_shoots_RNAseq_FLvsGC_metabolism")


**8. Pathway analysis**
 input_selectContrast1 <- 'Flight-Ground'	#select Comparison 
 #input_selectContrast1 = limma.out$comparisons[3] # manually set
 input_selectGO <- 'KEGG'	#Gene set category 
 #input_selectGO='custom' # if custom gmt file
 input_minSetSize <- 5	#Min size for gene set
 input_maxSetSize <- 2000	#Max size for gene set 


 
 # Read pathway data again 
 GeneSets.out <-readGeneSets( geneSetFile,
    convertedData.out, input_selectGO,input_selectOrg,
    c(input_minSetSize, input_maxSetSize)  ) 
 input_pathwayPvalCutoff <- 0.2	#FDR cutoff
 input_nPathwayShow <- 30	#Top pathways to show
 input_absoluteFold <- FALSE	#Use absolute values of fold-change?
 input_GenePvalCutoff <- 1	#FDR to remove genes 

 input_pathwayMethod = 1  # 1  GAGE
 gagePathwayData.out <- gagePathwayData()  # pathway analysis using GAGE  
   
 results <- gagePathwayData.out  #Enrichment analysis for k-Means clusters	
 results$adj.Pval <- format( results$adj.Pval,digits=3 )
 kable( results, row.names=FALSE) 
 pathwayListData.out = pathwayListData() 
 enrichmentPlot(pathwayListData.out, 25  ) 
  enrichmentNetwork(pathwayListData.out )  
  enrichmentNetworkPlotly(pathwayListData.out) 

 input_pathwayMethod = 3  # 1  fgsea 
 fgseaPathwayData.out <- fgseaPathwayData() #Pathway analysis using fgsea 
 results <- fgseaPathwayData.out  #Enrichment analysis for k-Means clusters	
 results$adj.Pval <- format( results$adj.Pval,digits=3 )
 kable( results, row.names=FALSE) 
  pathwayListData.out = pathwayListData() 
 enrichmentPlot(pathwayListData.out, 25  ) 
  enrichmentNetwork(pathwayListData.out )  
  enrichmentNetworkPlotly(pathwayListData.out) 
   PGSEAplot() # pathway analysis using PGSEA 

# Down_regulated
![enrichment_barplot_down_regulated](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/enrichment_barplot_down_regulated.png "enrichment_barplot_down_regulated")

# Up_regulated
![enrichment_barplot_up_regulated](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/enrichment_barplo_(Up_regulated)t.png "enrichment_barplot_up_regulated")

**9. Chromosome**
 input_selectContrast2 <- 'Flight-Ground'	#select Comparison 
 #input_selectContrast2 = limma.out$comparisons[3] # manually set
 input_limmaPvalViz <- 0.1	#FDR to filter genes
 input_limmaFCViz <- 2	#FDR to filter genes 
 genomePlotly() # shows fold-changes on the genome 

![enrichment_BRIC_LED](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/gnome_FDR_enrichment_BRIC_LED.png "enrichment_BRIC_LED")


**10. Biclustering**
 input_nGenesBiclust <- 1000	#Top genes for biclustering
 input_biclustMethod <- 'BCCC()'	#Method: 'BCCC', 'QUBIC', 'runibic' ... 
 biclustering.out = biclustering()  # run analysis
 input_selectBicluster <- 1	#select a cluster 
 biclustHeatmap()   # heatmap for selected cluster 
 input_selectGO4 <- 'GOBP'	#Gene set category 

**ER cluster plotted using KEGG path view**
![BRIC-LED_shoots_RNAseq_FLvsGC_ER](https://github.com/dr-richard-barker/BRIC_LED_AWG/blob/main/ER_protein_processing.png "BRIC-LED_shoots_RNAseq_FLvsGC_ER")
 
 # Read pathway data again 
 GeneSets.out <-readGeneSets( geneSetFile,
    convertedData.out, input_selectGO4,input_selectOrg,
    c(input_minSetSize, input_maxSetSize)  )  
 results <- geneListBclustGO()  #Enrichment analysis for k-Means clusters	
 results$adj.Pval <- format( results$adj.Pval,digits=3 )
 kable( results, row.names=FALSE) 

**11. Co-expression network**
 input_mySoftPower <- 8	#SoftPower to cutoff
 input_nGenesNetwork <- 1000	#Number of top genes
 input_minModuleSize <- 20	#Module size minimum 
 wgcna.out = wgcna()   # run WGCNA  
 softPower()  # soft power curve 
  modulePlot()  # plot modules  
  listWGCNA.Modules.out = listWGCNA.Modules() #modules
 
 input_selectGO5 <- 'GOBP'	#Gene set category 

 # Read pathway data again 
 GeneSets.out <-readGeneSets( geneSetFile,
    convertedData.out, input_selectGO5,input_selectOrg,
    c(input_minSetSize, input_maxSetSize)  ) 
 input_selectWGCNA.Module <- '1. turquoise (255 genes)'	#Select a module
 input_topGenesNetwork <- 10	#SoftPower to cutoff
 input_edgeThreshold <- 0.4	#Number of top genes 
 moduleNetwork()	# show network of top genes in selected module
 
 input_removeRedudantSets <- TRUE	#Remove redundant gene sets 
 results <- networkModuleGO()  #Enrichment analysis of selected module
 results$adj.Pval <- format( results$adj.Pval,digits=3 )
 kable( results, row.names=FALSE) 
