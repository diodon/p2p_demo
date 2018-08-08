Extract Taxa list from SARCE country data
================

Introduction
------------

This script will read SARCE data and extract the list of taxa. The extracted list will be matched against WoRMS and the corrections made. The SARCE dataset is already splitted by country.

Read the data
-------------

The SARCE dataset in full is a table in wide format, with the taxon name in the columns along with many other variables identifiying the site. We will produce a table in long format with the taxon name in the column "scientificName" (standard DwC name). As this table is only presence/absence we will recode that in the "occurence" variable.

``` r
library(tidyr)
library(readr)  ## this one is better for reading the wide-table

## I'll use Venezuela as case study here
## read the data
VEN <- read_csv("data/SARCE/Venezuela.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   Abiota = col_integer(),
    ##   Acanthophora_spicifera = col_integer(),
    ##   Acanthopleura_granulata = col_integer(),
    ##   Acmaea_antillarum = col_integer(),
    ##   Acmaea_sp1 = col_integer(),
    ##   Acmaea_sp2 = col_integer(),
    ##   Acmaea_sp4 = col_integer(),
    ##   Actiniaria_spp1 = col_integer(),
    ##   Agaricia_agaricites = col_integer(),
    ##   Amphiroa_fragilissima = col_integer(),
    ##   Amphiroa_sp1 = col_integer(),
    ##   Aplysia_dactylomela = col_integer(),
    ##   Ascidiidae_spp = col_integer(),
    ##   Asparagopsis_sp1 = col_integer(),
    ##   Asparagopsis_sp2 = col_integer(),
    ##   Asparagopsis_sp3 = col_integer(),
    ##   Astraea_sp1 = col_integer(),
    ##   Astraea_tecta = col_integer(),
    ##   Balanus_sp2 = col_integer(),
    ##   Bivalvia_spp = col_integer()
    ##   # ... with 203 more columns
    ## )

    ## See spec(...) for full column specifications.

``` r
## see the tablel dimensions
dim(VEN)
```

    ## [1]  620 1136

Clean the data
--------------

You see that the Venezuela SARCE table has 620 rows and 1136 colums.

**TIP**: in RStudio, if you have a very wide table (i.e. many columns) don't try to view the table in the viewer as it will take very long time to accomodate all the columns in the memory

Now, let extract the names of the taxa in the table. In the SARCE table, the first two colums are identification values, then the taxon name are in columns 3:1113. The rest of the colums are variables associated with the site (lat, lon, depth, zone, etc). More on those columns later...

So lets use `tidyr::gather` to convert the table from wide to long format

``` r
VEN.long = gather(VEN, key=scientificName, value=occurrence, 3:1113)

## look at the structure
str(VEN.long)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    688820 obs. of  27 variables:
    ##  $ Id            : chr  "Ve-Mi-Chi-Cor-L1" "Ve-Mi-Chi-Cor-L2" "Ve-Mi-Chi-Cor-L3" "Ve-Mi-Chi-Cor-L4" ...
    ##  $ pathString    : chr  "Ve/Mi/Chi/Cor/L1" "Ve/Mi/Chi/Cor/L2" "Ve/Mi/Chi/Cor/L3" "Ve/Mi/Chi/Cor/L4" ...
    ##  $ Year          : int  2010 2010 2010 2010 2010 2010 2010 2010 2010 2010 ...
    ##  $ Months        : int  7 7 7 7 7 7 7 7 7 7 ...
    ##  $ Country       : chr  "Venezuela" "Venezuela" "Venezuela" "Venezuela" ...
    ##  $ Country_code  : chr  "Ve" "Ve" "Ve" "Ve" ...
    ##  $ State         : chr  "Miranda" "Miranda" "Miranda" "Miranda" ...
    ##  $ State_code    : chr  "Mi" "Mi" "Mi" "Mi" ...
    ##  $ Locality      : chr  "Chirimena" "Chirimena" "Chirimena" "Chirimena" ...
    ##  $ Locality_code : chr  "Chi" "Chi" "Chi" "Chi" ...
    ##  $ Site          : chr  "Corrales" "Corrales" "Corrales" "Corrales" ...
    ##  $ Site_code     : chr  "Cor" "Cor" "Cor" "Cor" ...
    ##  $ Strata        : chr  "Lowtide" "Lowtide" "Lowtide" "Lowtide" ...
    ##  $ Strata_code   : chr  "L" "L" "L" "L" ...
    ##  $ Sampling Date : chr  "07/30/14" "07/30/14" "07/30/14" "07/30/14" ...
    ##  $ Observers     : chr  "JJ_Cruz/C_Herrera/A_Hernandez/N_Fernandez" "JJ_Cruz/C_Herrera/A_Hernandez/N_Fernandez" "JJ_Cruz/C_Herrera/A_Hernandez/N_Fernandez" "JJ_Cruz/C_Herrera/A_Hernandez/N_Fernandez" ...
    ##  $ Picture number: chr  "IMG_3308" "IMG_3309" "IMG_3310" "IMG_3311" ...
    ##  $ Replicate     : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ Labelscopy    : chr  "Ve-Mi-Chi-Cor-L1" "Ve-Mi-Chi-Cor-L2" "Ve-Mi-Chi-Cor-L3" "Ve-Mi-Chi-Cor-L4" ...
    ##  $ Si-Strata     : chr  "Cor-Lowtide" "Cor-Lowtide" "Cor-Lowtide" "Cor-Lowtide" ...
    ##  $ BioR          : int  66 66 66 66 66 66 66 66 66 66 ...
    ##  $ Latitude      : num  10.6 10.6 10.6 10.6 10.6 ...
    ##  $ Bioregion     : chr  "South_caribbean" "South_caribbean" "South_caribbean" "South_caribbean" ...
    ##  $ Ocean         : chr  "Atlantic" "Atlantic" "Atlantic" "Atlantic" ...
    ##  $ Zone          : chr  "Tropics" "Tropics" "Tropics" "Tropics" ...
    ##  $ scientificName: chr  "Abiota" "Abiota" "Abiota" "Abiota" ...
    ##  $ occurrence    : chr  NA NA NA NA ...

We need to do some cleaning. The taxon name is separated by an underscore. we want a space (for WoRMS to process the match). We don't need lines with `occurrence` equal to NA (that means that this particular taxon was not observed) nor the taxon "Abiotic"

``` r
## remote the lines with occurrence == NA
VEN.long = VEN.long[!is.na(VEN.long$occurrence),]

## remote the lines with taxa "abiotic"
VEN.long = VEN.long[VEN.long$scientificName!="Abiota",]

## replace the underscore in the taxon name by a space
VEN.long$scientificName = gsub("_", " ", VEN.long$scientificName)
```

Extract the taxa list
---------------------

Now with the date clean, let extract the taxon list, and save it in a text file for matching with WoRMS

``` r
taxa = unique(VEN.long$scientificName)
print(taxa)
```

    ##   [1] "Acanthophora spicifera"    "Acanthopleura granulata"  
    ##   [3] "Acmaea antillarum"         "Acmaea sp1"               
    ##   [5] "Acmaea sp2"                "Acmaea sp4"               
    ##   [7] "Actiniaria spp1"           "Agaricia agaricites"      
    ##   [9] "Amphiroa fragilissima"     "Amphiroa sp1"             
    ##  [11] "Aplysia dactylomela"       "Ascidiidae spp"           
    ##  [13] "Asparagopsis sp1"          "Asparagopsis sp2"         
    ##  [15] "Asparagopsis sp3"          "Astraea sp1"              
    ##  [17] "Astraea tecta"             "Balanus sp2"              
    ##  [19] "Bivalvia spp"              "Brachidontes dominguensis"
    ##  [21] "Brachidontes sp1"          "Brachidontes sp2"         
    ##  [23] "Brachidontes sp3"          "Brachidontes sp4"         
    ##  [25] "Brachidontes sp5"          "Brachidontes sp6"         
    ##  [27] "Brachidontes sp7"          "Bryopsis pennata"         
    ##  [29] "Bryopsis plumosa"          "Bryopsis sp1"             
    ##  [31] "Caulerpa mexicana"         "Caulerpa racemosa"        
    ##  [33] "Caulerpa sertularioides"   "Caulerpa sp1"             
    ##  [35] "Caulerpa sp2"              "Caulerpa sp3"             
    ##  [37] "CCA"                       "Centroceras sp1"          
    ##  [39] "Ceratozona squalida"       "Cerithium atratum"        
    ##  [41] "Cerithium sp1"             "Chaetomorpha antennina"   
    ##  [43] "Chaetomorpha crassa"       "Chaetomorpha gracilis"    
    ##  [45] "Chaetomorpha sp1"          "Chama sp1"                
    ##  [47] "Chiton sp4"                "Chiton sp5"               
    ##  [49] "Chiton sp6"                "Chiton squamosus"         
    ##  [51] "Chlorophyta spp"           "Cirripedia spp"           
    ##  [53] "Cittarium pica"            "Cittarium sp1"            
    ##  [55] "Cittarium sp2"             "Cliona sp1"               
    ##  [57] "Cliona sp4"                "Colpomenia sinuosa"       
    ##  [59] "Columbella sp1"            "Corallina racemosa"       
    ##  [61] "Corallina sp2"             "Dasya sp1"                
    ##  [63] "Dasya sp2"                 "Decapoda spp"             
    ##  [65] "Dichocoenia sp1"           "Dictyosphaeria sp1"       
    ##  [67] "Dictyota cervicornis"      "Dictyota dichotoma"       
    ##  [69] "Dictyota hamifera"         "Dictyota humifusa"        
    ##  [71] "Dictyota menstrualis"      "Dictyota pfaffi"          
    ##  [73] "Dictyota sp1"              "Dictyota sp2"             
    ##  [75] "Dictyota sp3"              "Dictyota sp4"             
    ##  [77] "Dictyota sp5"              "Dictyota sp6"             
    ##  [79] "Diodora cayenensis"        "Diploria clivosa"         
    ##  [81] "Diploria strigosa"         "Distaplia sp1"            
    ##  [83] "Dysidea etheria"           "Echininus nodulosus"      
    ##  [85] "Echinometra lucunter"      "Echinometra viridis"      
    ##  [87] "Erythropodium caribaeorum" "Eualetes sp1"             
    ##  [89] "Film"                      "Fissurella angusta"       
    ##  [91] "Fissurella barbadensis"    "Fissurella nimbosa"       
    ##  [93] "Fissurella nodosa"         "Fissurella sp1"           
    ##  [95] "Fissurella sp11"           "Fissurella sp2"           
    ##  [97] "Fissurella sp3"            "Fissurella sp4"           
    ##  [99] "Fissurella sp5"            "Galaxaura sp1"            
    ## [101] "Gastropoda spp"            "Gelidiella acerosa"       
    ## [103] "Gelidiella sp1"            "Gelidiella sp2"           
    ## [105] "Gelidiella sp3"            "Gelidium americanum"      
    ## [107] "Gelidium sp1"              "Gelidium sp4"             
    ## [109] "Gracilaria domingensis"    "Gracilaria sp1"           
    ## [111] "Gracilaria sp2"            "Grapsidae spp"            
    ## [113] "Grapsus sp1"               "Halimeda opuntia"         
    ## [115] "Hemitoma octoradiata"      "Hildenbrandia sp2"        
    ## [117] "Holothuroidea spp"         "Hydrozoa spp"             
    ## [119] "Hypnea sp1"                "Hypnea sp2"               
    ## [121] "Hypnea sp3"                "Hypnea spinella"          
    ## [123] "Ircinia strobilina"        "Isognomon alatus"         
    ## [125] "Isognomon radiatus"        "Isognomon sp1"            
    ## [127] "Isognomon sp3"             "Isognomon sp4"            
    ## [129] "Isognomon sp5"             "Laurencia filiformis"     
    ## [131] "Laurencia mamilosa"        "Laurencia obtusa"         
    ## [133] "Laurencia papillosa"       "Laurencia poteaui"        
    ## [135] "Laurencia sp1"             "Laurencia sp2"            
    ## [137] "Laurencia sp3"             "Laurencia sp4"            
    ## [139] "Laurencia sp5"             "Laurencia sp6"            
    ## [141] "Lebrunia coralligens"      "Lebrunia sp1"             
    ## [143] "Lebrunia sp2"              "Leucozonia ocellata"      
    ## [145] "Littorina angustior"       "Littorina interrupta"     
    ## [147] "Littorina meleagris"       "Littorina sp1"            
    ## [149] "Littorina sp2"             "Littorina sp3"            
    ## [151] "Littorina sp5"             "Littorina ziczac"         
    ## [153] "Lyngbya sp1"               "Lyngbya sp2"              
    ## [155] "Lyngbya sp3"               "Lyngbya sp4"              
    ## [157] "Lyngbya sp5"               "Millepora alcicornis"     
    ## [159] "Millepora complanata"      "Millepora sp1"            
    ## [161] "Mitrella ocellata"         "Mitrella sp1"             
    ## [163] "Mitrella sp2"              "Mitrella sp3"             
    ## [165] "Mitrella sp4"              "Mitrella sp5"             
    ## [167] "Mitrella sp6"              "Muricidae spp"            
    ## [169] "Nerita peloronta"          "Nerita tessellata"        
    ## [171] "Nerita versicolor"         "Niphates erecta"          
    ## [173] "Nitidella laevigata"       "Ophioderma sp1"           
    ## [175] "Ophiurida spp"             "Padina gymnospora"        
    ## [177] "Padina sp1"                "Palythoa caribaeorum"     
    ## [179] "Palythoa sp1"              "Petaloconchus sp1"        
    ## [181] "Petrolisthes sp1"          "Petrolisthes sp2"         
    ## [183] "Phaeophyceae spp"          "Phallusia nigra"          
    ## [185] "Planaxis sp1"              "Planaxis sp2"             
    ## [187] "Planaxis sp3"              "Planaxis sp4"             
    ## [189] "Plantae spp"               "Polysiphonia atlantica"   
    ## [191] "Porites astreoides"        "Porites sp2"              
    ## [193] "Pseudolithoderma extensum" "Pteria colimbus"          
    ## [195] "Purpura patula"            "Sacoglossa spp"           
    ## [197] "Sargassum sp1"             "Siderastrea radians"      
    ## [199] "Siderastrea sp1"           "Siphonaria sp1"           
    ## [201] "Siphonaria sp2"            "Tectarius muricatus"      
    ## [203] "Tetraclita sp1"            "Thais deltoidea"          
    ## [205] "Thais rustica"             "Thais sp1"                
    ## [207] "Thais sp2"                 "Thais sp3"                
    ## [209] "Thais sp4"                 "Thalassia testudinum"     
    ## [211] "Tubastrea aurea"           "Ulva lactuca"             
    ## [213] "Ulva sp1"                  "Ulva sp20"                
    ## [215] "Ulva sp21"                 "Vermetidae spp"           
    ## [217] "Zoanthus pulchellus"

``` r
## save it in a text file
writeLines(unique(VEN.long$scientificName), con="data/SARCE/VEN_taxa.csv")
```

Now got to [WoRMS](marinespecies.org) and do the match