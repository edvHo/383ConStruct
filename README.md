# 383ConStruct


To download the conStruct package, run the following command in the console
```
install.packages("conStruct")
```
Running this command will also download dependencies for conStruct

Load the package after installation:
```
library(conStruct)
```

The following instructions are for using data to recreate Figure 7 of Black Bears and Poplars study

conStruct expects three inputs: a matrix of allele frequencies, a matrix of geographical coordinates, and a distance matrix.
The data from the paper used in the original analysis (https://datadryad.org/dataset/doi:10.5061/dryad.5qj7h09) contains a file called 
"bear.dataset.Robj" which contains the allele frequencies and the coordinates.
```
load("bear.dataset.Robj")
sample.freqs <- bear.dataset$sample.freqs
sample.coords <- bear.dataset$sample.coords
```
Now, a distance matrix needs to be calculated. The geosphere R package can be used to do this.
```
install.packages("geosphere")
library("geosphere")
geo.dist.matrix <- distm(sample.coords, fun=distGeo)
```
Now that all inputs are ready, a conStruct analysis can be run.
```
mr.run <- conStruct(spatial=TRUE, K = 3, freqs = sample.freqs, 
                    geoDist = geo.dist.matrix, 
                    coords = sample.coords, 
                    prefix = "bear_k3", n.chains=1, 
                    n.iter=1000, make.figs=TRUE, save.files=TRUE)
```
Setting spatial to true outputs a spatial model, false for a non-spatial model.

K represents the number of layers.

freqs includes the allele frequency data.

geoDist represents the geographic distance matrix.

coords includes the sampling coordinates data.

prefix sets the string prepended to the output files.

n.iter represents the number of MCMC iterations

make.figs = TRUE and save.files = TRUE are already defaulted to "TRUE" but can be useful when running many independent analyses. 

Although the make.figs function creates the figures; it is important to be able to create them separately. First, load the output data
```
load("bear_k3_data.block.Robj")
load("bear_k3_conStruct.results.Robj")
```
Now, different plots can be made. First, the structure plot
```
admix.props <- conStruct.results$chain_1$MAP$admix.proportions
make.structure.plot(admix.proportions = admix.props)
```
<img width="645" height="321" alt="Screenshot 2026-04-10 at 2 12 12 PM" src="https://github.com/user-attachments/assets/742a323f-d756-4705-81a1-1d65970232f4" />

You can also order these structure plots by membership and change how the layers are stacked
```
# ordering plotted samples by membership
make.structure.plot(admix.proportions = admix.props,
                    sort.by = 1)
# re-order the stacking of the layers
make.structure.plot(admix.proportions = admix.props,
                    layer.order = c(2,1,3),
                    sort.by = 2)
```
<img width="548" height="303" alt="Screenshot 2026-04-10 at 2 12 43 PM" src="https://github.com/user-attachments/assets/2d74a331-0c39-44ad-9c4f-0c24435c56ca" />
<img width="641" height="316" alt="Screenshot 2026-04-10 at 2 12 26 PM" src="https://github.com/user-attachments/assets/940c9c35-8e34-43ad-aa10-b421ef560da8" />

Admixture pie plots can also be constructed with parameters to change pie chart size as well
```
#pie admixture pie plot
make.admix.pie.plot(admix.proportions = admix.props,
                    coords = sample.coords)
make.admix.pie.plot(admix.proportions = admix.props,
                    coords = sample.coords,
                    radii = 4)
```
<img width="545" height="323" alt="Screenshot 2026-04-10 at 2 12 55 PM" src="https://github.com/user-attachments/assets/833b13aa-491a-437e-954f-6c9b1fe905ad" />

You can then layer these pie charts over a map. This step requires the "maps" R package 
```
install.packages("maps")
library(maps)
# Construct the map
maps::map(xlim = range(sample.coords[,1]) + c(-5,5), 
          ylim = range(sample.coords[,2]) + c(-2,2), 
          col="gray")
# Place the pie plot over the map
make.admix.pie.plot(admix.proportions = admix.props,
                    coords = sample.coords,
                    add = TRUE)
```
<img width="537" height="304" alt="Screenshot 2026-04-10 at 2 13 03 PM" src="https://github.com/user-attachments/assets/c1d92f9d-fb0a-44c6-ade6-a46c9523cebe" />

One issue with running a conStruct analysis is getting a number of error messages, such as:
<img width="719" height="309" alt="Screenshot 2026-04-10 at 3 14 46 PM" src="https://github.com/user-attachments/assets/3db0283a-39b4-4364-aaeb-995f078104fe" />
However, these did not seem to impact the output data, but one way to check is by looking at the trace plots that are output from the make.figs function. If the run is successful, the trace plot would look like a "fuzzy caterpillar" as shown:
<img width="666" height="265" alt="Screenshot 2026-04-10 at 3 13 16 PM" src="https://github.com/user-attachments/assets/4c3d4193-2234-4ff6-911e-2e36f273b4f1" />



## Working with new Dataset
NOTE: The following section is project specific, filepaths may differ

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

