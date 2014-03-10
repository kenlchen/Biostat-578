Bioinformatics for Big Omics Data: Introduction to Bioconductor
========================================================
#width: 1440
#height: 900
autosize: true
transition: none
font-family: 'Helvetica'
css: my_style.css
author: Raphael Gottardo, PhD
date: February 27, 2014

<a rel="license" href="http://creativecommons.org/licenses/by-sa/3.0/deed.en_US"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by-sa/3.0/88x31.png" /></a><br /><tiny>This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/3.0/deed.en_US">Creative Commons Attribution-ShareAlike 3.0 Unported License</tiny></a>.

The Bioconductor project
========================

- [Bioconductor](http://www.bioconductor.org) is an open source, open development software project to provide tools for the analysis and comprehension of high-throughput genomic data. It is based primarily on the R programming language.

- Most Bioconductor components are distributed as R packages. The functional scope of Bioconductor packages includes the analysis of DNA microarray, sequence, flow, SNP, and other data.

Project Goals
==============
The broad goals of the Bioconductor project are:
- To provide widespread access to a broad range of powerful statistical and graphical methods for the analysis of genomic data.
- To facilitate the inclusion of biological metadata in the analysis of genomic data, e.g. literature data from PubMed, annotation data from Entrez genes.
- To provide a common software platform that enables the rapid development and deployment of extensible, scalable, and interoperable software.
- To further scientific understanding by producing high-quality documentation and reproducible research.
- To train researchers on computational and statistical methods for the analysis of genomic data.

Getting started
================
Before running this presentation, you will need to 
 - set the current directory in the console (go to session -> Set Working Directory -> To Source File Location)
 - run the following commands which take up time
    - source("http://bioconductor.org/biocLite.R")
    - biocLite()
    - getSQLiteFile() Note: this will take some time
    - getGEOSuppFiles("GSE29617", makeDirectory = TRUE, baseDir = "./")
    - untar("./GSE29617/GSE29617_RAW.tar", exdir="./GSE29617", tar = Sys.getenv("TAR"))

Although you will have already connected to the bioconductor website, I have found that I need to keep the code on the next page running (i.e., don't use eval=FALSE)

Getting started
================


```r
source("http://bioconductor.org/biocLite.R")
# Install all core packages and update all installed packages
biocLite()
```


You can also install specific packages


```r
biocLite(c("GEOmetadb", "GEOquery"))
```



The Gene Expression Omnibus (GEO)
==========================

The [Gene Expression Omnibus](http://www.ncbi.nlm.nih.gov/geo/) is an international public repository that archives and freely distributes microarray, next-generation sequencing, and other forms of high-throughput functional genomics data submitted by the research community.

The three main goals of GEO are to:

- Provide a robust, versatile database in which to efficiently store high-throughput functional genomic data
- Offer simple submission procedures and formats that support complete and well-annotated data deposits from the research community
- Provide user-friendly mechanisms that allow users to query, locate, review and download studies and gene expression profiles of interest

Getting data from GEO
=====================

Before getting data from GEO, we need to see what data we want. For that we can use the `GEOmetadb` package. 


```r
library(GEOmetadb)
```


Remember that packages in Bioconductor are well documented with a vignette that can be access as follows:


```r
vignette("GEOmetadb")
```

or if the package contains multiple vignettes or a vignette with a non-standard name

```r
browseVignettes(package = "GEOmetadb")
```


Finding the right data in GEO
==========================

<small>Zhu, Y., Davis, S., Stephens, R., Meltzer, P. S., & Chen, Y. (2008). GEOmetadb: powerful alternative search engine for the Gene Expression Omnibus. Bioinformatics (Oxford, England), 24(23), 2798–2800. doi:10.1093/bioinformatics/btn520</small>

GEOmetadb uses a SQLite database to store all metadata associate with GEO.

```r
## This will download the entire database, so can be slow
getSQLiteFile()
```



```r
geo_con <- dbConnect(SQLite(),'GEOmetadb.sqlite')
dbListTables(geo_con)
```

```
 [1] "gds"               "gds_subset"        "geoConvert"       
 [4] "geodb_column_desc" "gpl"               "gse"              
 [7] "gse_gpl"           "gse_gsm"           "gsm"              
[10] "metaInfo"          "sMatrix"          
```



```r
dbListFields(geo_con, 'gse')
```

```
 [1] "ID"                   "title"                "gse"                 
 [4] "status"               "submission_date"      "last_update_date"    
 [7] "pubmed_id"            "summary"              "type"                
[10] "contributor"          "web_link"             "overall_design"      
[13] "repeats"              "repeats_sample_list"  "variable"            
[16] "variable_description" "contact"              "supplementary_file"  
```



Finding a study
===============

The basic record types in GEO include Platforms (GPL), Samples (GSM), Series (GSE) and DataSets (GDS)


```r
dbGetQuery(geo_con, "SELECT gse.ID, gse.title, gse.gse FROM gse WHERE gse.pubmed_id='21743478';")
```

```
     ID
1 26409
2 26410
3 26412
4 26413
5 26414
                                                                                                         title
1                  Time Course of Young Adults Vaccinated with Influenza TIV Vaccine during 2007/08 Flu Season
2                 Time Course of Young Adults Vaccinated with Influenza LAIV Vaccine during 2008/09 Flu Season
3                  Time Course of Young Adults Vaccinated with Influenza TIV Vaccine during 2008/09 Flu Season
4 FACS-sorted cells from Young Adults Vaccinated with Influenza TIV or LAIV Vaccines during 2008/09 Flu Season
5                                              Systems biology of vaccination for seasonal influenza in humans
       gse
1 GSE29614
2 GSE29615
3 GSE29617
4 GSE29618
5 GSE29619
```


Finding a study (suite)
===============

What samples were used?


```r
dbGetQuery(geo_con, "SELECT gse.gse, gsm.gsm, gsm.title FROM (gse JOIN gse_gsm ON gse.gse=gse_gsm.gse) j JOIN gsm ON j.gsm=gsm.gsm WHERE gse.pubmed_id='21743478' LIMIT 5;")
```

```
   gse.gse   gsm.gsm                                     gsm.title
1 GSE29614 GSM733816 2007 TIV subject ID 12 at D0 post-vaccination
2 GSE29614 GSM733817 2007 TIV subject ID 12 at D3 post-vaccination
3 GSE29614 GSM733818 2007 TIV subject ID 12 at D7 post-vaccination
4 GSE29614 GSM733819 2007 TIV subject ID 16 at D0 post-vaccination
5 GSE29614 GSM733820 2007 TIV subject ID 16 at D3 post-vaccination
```

gse_gsm contains the gse number that is associated with the gsm number. 
j is the name of a table that is created by joining gse and ges_gsm. Then j is joined with table gsm. 

Finding a study (suite)
===============

What about raw data?


```r
res <- dbGetQuery(geo_con, "SELECT gsm.gsm, gsm.supplementary_file FROM (gse JOIN gse_gsm ON gse.gse=gse_gsm.gse) j JOIN gsm ON j.gsm=gsm.gsm WHERE gse.pubmed_id='21743478' LIMIT 5;")
head(res)
```

```
    gsm.gsm
1 GSM733816
2 GSM733817
3 GSM733818
4 GSM733819
5 GSM733820
                                                                                                                                                gsm.supplementary_file
1 ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733816/suppl/GSM733816.CEL.gz;\tftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733816/suppl/GSM733816.chp.gz
2 ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733817/suppl/GSM733817.CEL.gz;\tftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733817/suppl/GSM733817.chp.gz
3 ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733818/suppl/GSM733818.CEL.gz;\tftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733818/suppl/GSM733818.chp.gz
4 ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733819/suppl/GSM733819.CEL.gz;\tftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733819/suppl/GSM733819.chp.gz
5 ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733820/suppl/GSM733820.CEL.gz;\tftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM733nnn/GSM733820/suppl/GSM733820.chp.gz
```

raw data is contained in the supplementary files, which are listed in the gsm file. 


Finding specific data
=====================
To get list of manufacturers:

```r
library(data.table)
manu <- data.table(dbGetQuery(geo_con, "SELECT manufacturer FROM gpl"))
manu[,list(length(manufacturer)), by=manufacturer]
```

```
                                                                                                                                                                                                                                   manufacturer
   1:                                                                                                                                                                                                                                        NA
   2:                                                                                                                                                                                                                                Affymetrix
   3:                                                                                                                                                                                                                   BD Biosciences Clontech
   4:                                                                                                                                                                                                                             Sigma Genosys
   5:                                                                                                                                                                          Advanced Technology Center Microarray Facility (NIH/NCI/CCR/ATC)
  ---                                                                                                                                                                                                                                          
1989:                                                                                                                                                                                         Institute for Environmental Genomics (Norman, OK)
1990:                                                                                                                                                                                                     Memorec Biotec GmbH, Cologne, Germany
1991:                                                                                                                                                                                         Johns Hopkins University microarray core facility
1992: 70mer oligodeoxyribonucleotides, covering all 2857 open reading frames of the L. monocytogenes EGDe, were ordered from Operon Biotechnologies (Cologne, Germany). Oligos were spotted on the slides by BioCat GmbH (Heidelberg, Germany).
1993:                                                                                                                                                                                                                      SABiosciences Qiagen
      V1
   1:  1
   2:  1
   3:  1
   4:  1
   5:  1
  ---   
1989:  1
1990:  1
1991:  1
1992:  1
1993:  1
```


Finding specific data
=====================
To get supplementary file names ending with cel.gz from only manufacturer Affymetrix

```r
res <- dbGetQuery(geo_con, "SELECT gpl.bioc_package, gsm.title, gsm.series_id, gsm.gpl, gsm.supplementary_file FROM gsm JOIN gpl ON gsm.gpl=gpl.gpl WHERE gpl.manufacturer='Affymetrix' AND gsm.supplementary_file like '%CEL.gz';")
head(res)
```

```
  bioc_package               title series_id   gpl
1       hu6800          BM_CD34-1a    GSE500 GPL80
2       hu6800          BM_CD34-1b    GSE500 GPL80
3       hu6800           BM_CD34-2    GSE500 GPL80
4       hu6800        GPBMC_CD34-1    GSE500 GPL80
5       hu6800        GPBMC_CD34-2    GSE500 GPL80
6       rgu34a CNS-SC_Inj24h-3A-s2    GSE464 GPL85
                                                           supplementary_file
1    ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSMnnn/GSM575/suppl/GSM575.cel.gz
2    ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSMnnn/GSM576/suppl/GSM576.cel.gz
3    ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSMnnn/GSM577/suppl/GSM577.cel.gz
4    ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSMnnn/GSM578/suppl/GSM578.cel.gz
5    ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSMnnn/GSM579/suppl/GSM579.cel.gz
6 ftp://ftp.ncbi.nlm.nih.gov/geo/samples/GSM1nnn/GSM1136/suppl/GSM1136.CEL.gz
```


Finding specific data
=====================
From previous table:
bioc_package = bioconductor package
hu6800 = Affymetrix HuGeneFL Genome Array annotation data (chip hu6800) 
rgu34a = Affymetrix Rat Genome U34 Set annotation data (chip rgu34a)

title = data set title (?)
For example BM_CD34-1a = bone marrow flow-sorted CD34+ cells (>95% purity) and has GSM sample number GSM575. 

Getting the data we want
========================
Now that we have our GSE ID we can use the GEOquery to download the corresponding data
as follows,

```r
# Download the mapping information and processed data
# This returns a list of eSets
GSE29617_set <- getGEO("GSE29617")[[1]]
```

which returns (a list of) an ExpressionSet (eSet).


The eSet class
==================

What is an eSet? An S4 class that tries to:
- Coordinate high through-put (e.g., gene expression) and phenotype data.
- Provide common data container for diverse Bioconductor packages.


```r
str(GSE29617_set,max.level=2)
```

```
Formal class 'ExpressionSet' [package "Biobase"] with 7 slots
  ..@ experimentData   :Formal class 'MIAME' [package "Biobase"] with 13 slots
  ..@ assayData        :<environment: 0x000000000ef1a118> 
  ..@ phenoData        :Formal class 'AnnotatedDataFrame' [package "Biobase"] with 4 slots
  ..@ featureData      :Formal class 'AnnotatedDataFrame' [package "Biobase"] with 4 slots
  ..@ annotation       : chr "GPL13158"
  ..@ protocolData     :Formal class 'AnnotatedDataFrame' [package "Biobase"] with 4 slots
  ..@ .__classVersion__:Formal class 'Versions' [package "Biobase"] with 1 slots
```

str() is the command to get the internal structure of an R object. 
An eSet contains the necessary "parts" to summarize an experiment.

Classes and methods
====================

**Everything in R is an OBJECT.**


- A class is the definition of an object.
- A method is a function that performs specific calculations on objects of a
specific class. Generic functions are used to determine the class of its
arguments and select the appropriate method. A generic function is a
function with a collection of methods.
- See ?Classes and ?Methods for more information.


```r
data(iris)
class(iris)
```

```
[1] "data.frame"
```

```r
summary(iris)
```

```
  Sepal.Length   Sepal.Width    Petal.Length   Petal.Width 
 Min.   :4.30   Min.   :2.00   Min.   :1.00   Min.   :0.1  
 1st Qu.:5.10   1st Qu.:2.80   1st Qu.:1.60   1st Qu.:0.3  
 Median :5.80   Median :3.00   Median :4.35   Median :1.3  
 Mean   :5.84   Mean   :3.06   Mean   :3.76   Mean   :1.2  
 3rd Qu.:6.40   3rd Qu.:3.30   3rd Qu.:5.10   3rd Qu.:1.8  
 Max.   :7.90   Max.   :4.40   Max.   :6.90   Max.   :2.5  
       Species  
 setosa    :50  
 versicolor:50  
 virginica :50  
                
                
                
```


Classes and methods (suite)
====================

There are two types of classes in R: S3 Classes (old style, informal) and S4 Classes - (new style, more rigorous and formal)


```r
# S3 class
head(methods(class="data.frame"))
```

```
[1] "$<-.data.frame"       "[.data.frame"         "[[.data.frame"       
[4] "[[<-.data.frame"      "[<-.data.frame"       "aggregate.data.frame"
```

```r
# S4 class
showMethods(class="eSet")
```

```
Function: $ (package base)
x="eSet"

Function: $<- (package base)
x="eSet"

Function: [ (package base)
x="eSet"

Function: [[ (package base)
x="eSet"

Function: [[<- (package base)
x="eSet"

Function: abstract (package Biobase)
object="eSet"


Function "addAttributes":
 <not an S4 generic function>
Function: annotation (package BiocGenerics)
object="eSet"

Function: annotation<- (package BiocGenerics)
object="eSet", value="character"

Function: assayData (package Biobase)
object="eSet"

Function: assayData<- (package Biobase)
object="eSet", value="AssayData"


Function "clearMemoryManagement":
 <not an S4 generic function>

Function "clone":
 <not an S4 generic function>

Function "close":
 <not an S4 generic function>
Function: coerce (package methods)
from="eSet", to="ExpressionSet"
from="eSet", to="MultiSet"

Function: combine (package BiocGenerics)
x="eSet", y="eSet"


Function "comment.SAX":
 <not an S4 generic function>

Function "complete":
 <not an S4 generic function>
Function: description (package Biobase)
object="eSet"

Function: description<- (package Biobase)
object="eSet", value="MIAME"

Function: dim (package base)
x="eSet"

Function: dimnames (package base)
x="eSet"

Function: dimnames<- (package base)
x="eSet"

Function: dims (package Biobase)
object="eSet"


Function "docName":
 <not an S4 generic function>

Function "docName<-":
 <not an S4 generic function>

Function "endElement.SAX":
 <not an S4 generic function>

Function "entityDeclaration.SAX":
 <not an S4 generic function>
Function: experimentData (package Biobase)
object="eSet"

Function: experimentData<- (package Biobase)
object="eSet", value="MIAME"

Function: fData (package Biobase)
object="eSet"

Function: fData<- (package Biobase)
object="eSet", value="data.frame"

Function: featureData (package Biobase)
object="eSet"

Function: featureData<- (package Biobase)
object="eSet", value="AnnotatedDataFrame"

Function: featureNames (package Biobase)
object="eSet"

Function: featureNames<- (package Biobase)
object="eSet"


Function "findXIncludeStartNodes":
 <not an S4 generic function>

Function "free":
 <not an S4 generic function>
Function: fvarLabels (package Biobase)
object="eSet"

Function: fvarLabels<- (package Biobase)
object="eSet"

Function: fvarMetadata (package Biobase)
object="eSet"

Function: fvarMetadata<- (package Biobase)
object="eSet", value="data.frame"


Function "getEffectiveNamespaces":
 <not an S4 generic function>

Function "getEncoding":
 <not an S4 generic function>

Function "getEncodingREnum":
 <not an S4 generic function>

Function "getTableElementType":
 <not an S4 generic function>

Function "GPL":
 <not an S4 generic function>

Function "GSM":
 <not an S4 generic function>
Function: initialize (package methods)
.Object="eSet"

Function: notes (package Biobase)
object="eSet"

Function: notes<- (package Biobase)
object="eSet", value="ANY"

Function: pData (package Biobase)
object="eSet"

Function: pData<- (package Biobase)
object="eSet", value="data.frame"

Function: phenoData (package Biobase)
object="eSet"

Function: phenoData<- (package Biobase)
object="eSet", value="AnnotatedDataFrame"


Function "pop":
 <not an S4 generic function>
Function: preproc (package Biobase)
object="eSet"

Function: preproc<- (package Biobase)
object="eSet"


Function "processingInstruction.SAX":
 <not an S4 generic function>
Function: protocolData (package Biobase)
object="eSet"

Function: protocolData<- (package Biobase)
object="eSet", value="AnnotatedDataFrame"

Function: pubMedIds (package Biobase)
object="eSet"

Function: pubMedIds<- (package Biobase)
object="eSet", value="character"


Function "push":
 <not an S4 generic function>

Function "readHTMLList":
 <not an S4 generic function>

Function "readHTMLTable":
 <not an S4 generic function>

Function "readKeyValueDB":
 <not an S4 generic function>

Function "readSolrDoc":
 <not an S4 generic function>

Function "removeAttributes":
 <not an S4 generic function>

Function "removeXMLNamespaces":
 <not an S4 generic function>

Function "reset":
 <not an S4 generic function>
Function: sampleNames (package Biobase)
object="eSet"

Function: sampleNames<- (package Biobase)
object="eSet", value="ANY"


Function "saveXML":
 <not an S4 generic function>

Function "selectSomeIndex":
 <not an S4 generic function>
Function: show (package methods)
object="eSet"


Function "simplifyNamespaces":
 <not an S4 generic function>

Function "source":
 <not an S4 generic function>

Function "startElement.SAX":
 <not an S4 generic function>
Function: storageMode (package Biobase)
object="eSet"

Function: storageMode<- (package Biobase)
object="eSet", value="character"


Function "text.SAX":
 <not an S4 generic function>

Function "toHTML":
 <not an S4 generic function>
Function: updateObject (package BiocGenerics)
object="eSet"

Function: updateObjectTo (package Biobase)
object="eSet", template="eSet"

Function: varLabels (package Biobase)
object="eSet"

Function: varLabels<- (package Biobase)
object="eSet"

Function: varMetadata (package Biobase)
object="eSet"

Function: varMetadata<- (package Biobase)
object="eSet", value="data.frame"


Function "xmlAttrs<-":
 <not an S4 generic function>

Function "xmlAttrsToDataFrame":
 <not an S4 generic function>

Function "xmlChildren<-":
 <not an S4 generic function>

Function "xmlClone":
 <not an S4 generic function>

Function "xmlNamespace<-":
 <not an S4 generic function>

Function "xmlNamespaces<-":
 <not an S4 generic function>

Function "xmlParent":
 <not an S4 generic function>

Function "xmlRoot<-":
 <not an S4 generic function>

Function "xmlSource":
 <not an S4 generic function>

Function "xmlSourceFunctions":
 <not an S4 generic function>

Function "xmlSourceSection":
 <not an S4 generic function>

Function "xmlSourceTask":
 <not an S4 generic function>

Function "xmlSourceThread":
 <not an S4 generic function>

Function "xmlToDataFrame":
 <not an S4 generic function>

Function "xmlToS4":
 <not an S4 generic function>

Function "xmlValue<-":
 <not an S4 generic function>
```



The eSet (suite)
==========================
  
You can get a sense of the defined methods for an `eSet` as follows:

```r
library(Biobase)
showMethods(class="eSet")
```

```
Function: $ (package base)
x="eSet"

Function: $<- (package base)
x="eSet"

Function: [ (package base)
x="eSet"

Function: [[ (package base)
x="eSet"

Function: [[<- (package base)
x="eSet"

Function: abstract (package Biobase)
object="eSet"


Function "addAttributes":
 <not an S4 generic function>
Function: annotation (package BiocGenerics)
object="eSet"

Function: annotation<- (package BiocGenerics)
object="eSet", value="character"

Function: assayData (package Biobase)
object="eSet"

Function: assayData<- (package Biobase)
object="eSet", value="AssayData"


Function "clearMemoryManagement":
 <not an S4 generic function>

Function "clone":
 <not an S4 generic function>

Function "close":
 <not an S4 generic function>
Function: coerce (package methods)
from="eSet", to="ExpressionSet"
from="eSet", to="MultiSet"

Function: combine (package BiocGenerics)
x="eSet", y="eSet"


Function "comment.SAX":
 <not an S4 generic function>

Function "complete":
 <not an S4 generic function>
Function: description (package Biobase)
object="eSet"

Function: description<- (package Biobase)
object="eSet", value="MIAME"

Function: dim (package base)
x="eSet"

Function: dimnames (package base)
x="eSet"

Function: dimnames<- (package base)
x="eSet"

Function: dims (package Biobase)
object="eSet"


Function "docName":
 <not an S4 generic function>

Function "docName<-":
 <not an S4 generic function>

Function "endElement.SAX":
 <not an S4 generic function>

Function "entityDeclaration.SAX":
 <not an S4 generic function>
Function: experimentData (package Biobase)
object="eSet"

Function: experimentData<- (package Biobase)
object="eSet", value="MIAME"

Function: fData (package Biobase)
object="eSet"

Function: fData<- (package Biobase)
object="eSet", value="data.frame"

Function: featureData (package Biobase)
object="eSet"

Function: featureData<- (package Biobase)
object="eSet", value="AnnotatedDataFrame"

Function: featureNames (package Biobase)
object="eSet"

Function: featureNames<- (package Biobase)
object="eSet"


Function "findXIncludeStartNodes":
 <not an S4 generic function>

Function "free":
 <not an S4 generic function>
Function: fvarLabels (package Biobase)
object="eSet"

Function: fvarLabels<- (package Biobase)
object="eSet"

Function: fvarMetadata (package Biobase)
object="eSet"

Function: fvarMetadata<- (package Biobase)
object="eSet", value="data.frame"


Function "getEffectiveNamespaces":
 <not an S4 generic function>

Function "getEncoding":
 <not an S4 generic function>

Function "getEncodingREnum":
 <not an S4 generic function>

Function "getTableElementType":
 <not an S4 generic function>

Function "GPL":
 <not an S4 generic function>

Function "GSM":
 <not an S4 generic function>
Function: initialize (package methods)
.Object="eSet"

Function: notes (package Biobase)
object="eSet"

Function: notes<- (package Biobase)
object="eSet", value="ANY"

Function: pData (package Biobase)
object="eSet"

Function: pData<- (package Biobase)
object="eSet", value="data.frame"

Function: phenoData (package Biobase)
object="eSet"

Function: phenoData<- (package Biobase)
object="eSet", value="AnnotatedDataFrame"


Function "pop":
 <not an S4 generic function>
Function: preproc (package Biobase)
object="eSet"

Function: preproc<- (package Biobase)
object="eSet"


Function "processingInstruction.SAX":
 <not an S4 generic function>
Function: protocolData (package Biobase)
object="eSet"

Function: protocolData<- (package Biobase)
object="eSet", value="AnnotatedDataFrame"

Function: pubMedIds (package Biobase)
object="eSet"

Function: pubMedIds<- (package Biobase)
object="eSet", value="character"


Function "push":
 <not an S4 generic function>

Function "readHTMLList":
 <not an S4 generic function>

Function "readHTMLTable":
 <not an S4 generic function>

Function "readKeyValueDB":
 <not an S4 generic function>

Function "readSolrDoc":
 <not an S4 generic function>

Function "removeAttributes":
 <not an S4 generic function>

Function "removeXMLNamespaces":
 <not an S4 generic function>

Function "reset":
 <not an S4 generic function>
Function: sampleNames (package Biobase)
object="eSet"

Function: sampleNames<- (package Biobase)
object="eSet", value="ANY"


Function "saveXML":
 <not an S4 generic function>

Function "selectSomeIndex":
 <not an S4 generic function>
Function: show (package methods)
object="eSet"


Function "simplifyNamespaces":
 <not an S4 generic function>

Function "source":
 <not an S4 generic function>

Function "startElement.SAX":
 <not an S4 generic function>
Function: storageMode (package Biobase)
object="eSet"

Function: storageMode<- (package Biobase)
object="eSet", value="character"


Function "text.SAX":
 <not an S4 generic function>

Function "toHTML":
 <not an S4 generic function>
Function: updateObject (package BiocGenerics)
object="eSet"

Function: updateObjectTo (package Biobase)
object="eSet", template="eSet"

Function: varLabels (package Biobase)
object="eSet"

Function: varLabels<- (package Biobase)
object="eSet"

Function: varMetadata (package Biobase)
object="eSet"

Function: varMetadata<- (package Biobase)
object="eSet", value="data.frame"


Function "xmlAttrs<-":
 <not an S4 generic function>

Function "xmlAttrsToDataFrame":
 <not an S4 generic function>

Function "xmlChildren<-":
 <not an S4 generic function>

Function "xmlClone":
 <not an S4 generic function>

Function "xmlNamespace<-":
 <not an S4 generic function>

Function "xmlNamespaces<-":
 <not an S4 generic function>

Function "xmlParent":
 <not an S4 generic function>

Function "xmlRoot<-":
 <not an S4 generic function>

Function "xmlSource":
 <not an S4 generic function>

Function "xmlSourceFunctions":
 <not an S4 generic function>

Function "xmlSourceSection":
 <not an S4 generic function>

Function "xmlSourceTask":
 <not an S4 generic function>

Function "xmlSourceThread":
 <not an S4 generic function>

Function "xmlToDataFrame":
 <not an S4 generic function>

Function "xmlToS4":
 <not an S4 generic function>

Function "xmlValue<-":
 <not an S4 generic function>
```

in particular, the following methods are rather convenient:
  - assayData(obj); assayData(obj) `<-` value: access or assign assayData
- phenoData(obj); phenoData(obj) `<-` value: access or assign phenoData
- experimentData(obj); experimentData(obj) `<-` value: access or assign experimentData
- annotation(obj); annotation(obj) `<-` value: access or assign annotation


The ExpressionSet subclass
==========================
  
  Similar to the `eSet` class but tailored to gene expression, with an expression matrix that can be accessed with the `exprs` method.


```r
class(GSE29617_set)
```

```
[1] "ExpressionSet"
attr(,"package")
[1] "Biobase"
```

```r
exprs(GSE29617_set)[1:2,1:3]
```

```
             GSM733942 GSM733943 GSM733944
1007_PM_s_at     4.630     4.461     4.547
1053_PM_at       4.414     4.724     4.483
```


also provides additional methods such as `fData`.

The ExpressionSet subclass (suite)
==========================
  `ExpressionSet` objects are meant to facilitate the adoption of MIAME standard. MIAME = "Minimum Information about a Microarray experiment". Alvis Brazma et. al. (2001) Nature Genetics

Unfortrunately, not all contributors will upload all the information.

```r
# Information about preprocessing
# Nothing in here!
preproc(GSE29617_set)
```

```
list()
```

```r
# A data.frame with number of rows equal to the number of samples
pData(GSE29617_set)[1:2,1:2]
```

```
                                                 title geo_accession
GSM733942 2008 TIV subject ID 2 at D0 post-vaccination     GSM733942
GSM733943 2008 TIV subject ID 2 at D7 post-vaccination     GSM733943
```

```r
# A data.frame with number of rows equal to the number of features/probes
fData(GSE29617_set)[1:2,1:2]
```

```
                       ID GB_ACC
1007_PM_s_at 1007_PM_s_at U48705
1053_PM_at     1053_PM_at M87338
```



The ExpressionSet subclass (suite)
==========================
  So the `ExpressionSet` objects facilitate the encapsulation of everything that's needed to summarize and analyze an experiment. Specific elements can be access with the `@` operator but many classes have convenient accessor methods.


```r
fData(GSE29617_set)[1:2,1:2]
```

```
                       ID GB_ACC
1007_PM_s_at 1007_PM_s_at U48705
1053_PM_at     1053_PM_at M87338
```

```r
# Note that S4 classes can be nested!
GSE29617_set@featureData@data[1:2,1:2]
```

```
                       ID GB_ACC
1007_PM_s_at 1007_PM_s_at U48705
1053_PM_at     1053_PM_at M87338
```


What if you want the raw data?
==============================

GEO also provides access to raw data that can be downloaded with `GEOquery`.



```r
# Download all raw data. This should only be evaluated once
# Then the data would be stored locally in the data directory
getGEOSuppFiles("GSE29617", makeDirectory = FALSE, baseDir = "./Data/GEO/")
# untar downloaded data
untar("Data/GEO/GSE29617_RAW.tar", exdir="Data/GEO/GSE29617/", tar="gtar")
# If the above commands do not work, try:
getGEOSuppFiles("GSE29617", makeDirectory = TRUE, baseDir = "./")
untar("./GSE29617/GSE29617_RAW.tar", exdir="./GSE29617", tar = Sys.getenv("TAR"))
```


Starting from the raw data
==========================

Now that we have the Affymetrix raw data (CEL) files, we can apply some of the concepts we've discussed related to normalization and probe summary. We first need to load the appropriate package



```r
biocLite("affy")
```

```
package 'affy' successfully unpacked and MD5 sums checked

The downloaded binary packages are in
	C:\Users\Username\AppData\Local\Temp\RtmpMD5X0g\downloaded_packages
```


```r
library(affy)
```


then we use the following commands

```r
# Read the CEL file and creates and AffyBatch
GSE29617_affyBatch <- ReadAffy(celfile.path="Data/GEO/GSE29617/")
# Normalize and summarize the data
GSE29617_set2 <- rma(GSE29617_affyBatch)
```

```
Background correcting
Normalizing
Calculating Expression
```


Starting from the raw data
==========================

Let's check the results and compare to the expression matrix that was submitted to GEO

```r
exprs(GSE29617_set2)[1:2,1:2]
```

```
             GSM733942.CEL.gz GSM733943.CEL.gz
1007_PM_s_at            4.630            4.461
1053_PM_at              4.414            4.724
```


The rows are the features (i.e., probes). Columns are the samples.

What are those probes?
======================


```r
# We first need to install our annotation package
library(BiocInstaller)
# Note that you don't have to use source anymore!
biocLite("hthgu133a.db")
```

```

The downloaded binary packages are in
	C:\Users\Username\AppData\Local\Temp\RtmpMD5X0g\downloaded_packages
```





```r
library(hthgu133a.db)
probe_ids <- rownames(GSE29617_set2)
probe_data <- select(hthgu133a.db, keys = probe_ids, columns = "SYMBOL", keytype = "PROBEID")
probe_data[1,]
```

```
       PROBEID SYMBOL
1 1007_PM_s_at   <NA>
```

This didn't work very well, did it?
The problem is that the probe IDs in hthgu133a.db have a different naming scheme than those in GSE29617_set2. This is fixed on the next slide.

What are those probes? (Suite)
======================

Let's fix this: Replace _PM with <empty> for the probe id names in GSE29617_set2

```r
probe_ids <- gsub("_PM","",rownames(GSE29617_set2))
probe_data <- select(hthgu133a.db, keys = probe_ids, columns = "SYMBOL", keytype = "PROBEID")
```

```
Warning: 'select' resulted in 1:many mapping between keys and return rows
```

```r
probe_data[1, ]
```

```
    PROBEID  SYMBOL
1 1007_s_at MIR4640
```

What's the warning? Some probes match up with multiple genes, therefore those probe IDs will have more than one record.

What are those probes? (Suite)
======================


This gives us too many rows, what do we do? Concatenate the gene names so that there will be one row per probe ID.


```r
library(data.table)
probe_data_dt <- data.table(probe_data)
probe_data_dt_unique <- probe_data_dt[,list(SYMBOL=paste(SYMBOL,collapse=";")),by="PROBEID"]
probe_data_dt_unique[SYMBOL%like%";"]
```

```
          PROBEID                          SYMBOL
   1:   1007_s_at                    MIR4640;DDR1
   2:     1773_at                FNTB;CHURC1-FNTB
   3: 200012_x_at RPL21;RPL21P28;SNORD102;SNORA27
   4:   200018_at     RPS13;LOC100508408;SNORD14B
   5: 200038_s_at            RPL17;RPL17-C18orf32
  ---                                            
1090:  64474_g_at                   DGCR8;MIR1306
1091:  65133_i_at              INO80B;INO80B-WBP1
1092:    66053_at         HNRNPUL2;HNRNPUL2-BSCL2
1093:    78495_at                LOC155060;ZNF783
1094:    91617_at                   DGCR8;MIR1306
```


Completing our ExpressionSet
============================


```r
annotaded_probes <- data.frame(probe_data_dt_unique)
rownames(annotaded_probes) <- rownames(GSE29617_set2)
fData(GSE29617_set2) <- annotaded_probes
head(fData(GSE29617_set2))
```

```
               PROBEID       SYMBOL
1007_PM_s_at 1007_s_at MIR4640;DDR1
1053_PM_at     1053_at         RFC2
117_PM_at       117_at        HSPA6
121_PM_at       121_at         PAX8
1255_PM_g_at 1255_g_at       GUCA1A
1294_PM_at     1294_at         UBA7
```


Now you're ready to get the data that you will have to analyze for your second homework. 
