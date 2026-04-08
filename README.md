# 383ConStruct

\[Instructions for installing in RStudio]

To download the conStruct package, run the following command in the console
```
install.packages("conStruct")
```
Running this command will also download dependencies for conStruct

Load the package after installation:
```
library(conStruct)
```

\[Instructions for using data to recreate Figure 7 of Black Bears and Poplars study]

The following example uses the example Construct.data file included in the R package:
```
my.run <- conStruct(spatial = TRUE, 
                    K = 3, 
                    freqs = conStruct.data$allele.frequencies,
                    geoDist = conStruct.data$geoDist, 
                    coords = conStruct.data$coords,
                    prefix = "spK3",
                    n.chains = 1,
                    n.iter = 1000, 
                    make.figs = TRUE, 
                    save.files = TRUE)
```
Setting spatial to true outputs a spatial model, false for a non-spatial model.

K represents the number of layers.

freqs includes the allele frequency data.

geoDist represents the geographic distance matrix.

coords includes the sampling coordinates data.

prefix sets the string prepended to the output files.

n.iter represents the number of MCMC iterations

make.figs = TRUE and save.files = TRUE are already defaulted to "TRUE" but can be usefull when running many independant analyses. 

## Working with new Dataset
1. Working Directory setup and Loading Metadata
```
setwd("/home/data/Project5")

plant <- read.table(
  "plantmetadata.txt", 
  header = TRUE,       # first row is column names
  sep = "\t",          # tab-separated file
  stringsAsFactors = FALSE
)

head(plant)
```
2. Convert Structure file to conStruct format
```
population.data <- structure2conStruct(
  infile = "/home/data/Project5/populations.structure",
  onerowperind = FALSE,
  start.loci = 2,
  start.samples = 3,
  missing.datum = 0,
  outfile = "~/conStruct_project/structurefileConStructData.txt"
)

head(population.data)
```
3. Combine data to a list
```
grillodata <- list(
  population = population.data,
  plantmetadata = plant
)
```
4. Update for new Dataset
```
construct.data <- structure2conStruct(
  infile = "/home/data/Project5/populations.structure",
  onerowperind = FALSE,
  start.loci = 2,
  start.samples = 3,
  missing.datum = 0,
  outfile = "~/conStruct_project/structurefileConStructData2.txt"
)
```


5. Extract coordinates
```
install.packages("dplyr")
library(dplyr)

sampling.coords <- plant %>% 
  select(lat, lon)

head(sampling.coords)
```

