# 383ConStruct


To download the conStruct package, run the following command in the console
```
install.packages("conStruct")
```
Running this command will also download dependencies for conStruct

ConStruct includes multiple vignettes, which act as the user manual and can be accessed by:
```
# formatting data
vignette(topic="format-data",package="conStruct")

# how to run a conStruct analysis
vignette(topic="run-conStruct",package="conStruct")

# how to visualize the output of a conStruct model
vignette(topic="visualize-results",package="conStruct")

# how to compare and select between different conStruct models
vignette(topic="model-comparison",package="conStruct")
```

Load the package after installation:
```
library(conStruct)
```
ConStruct comes with a sample dataset included with the package. To load, simply enter:
```
data(conStruct.data)
```

This includes the three data types conStruct expects: allele frequency data, a geographic distance matrix, and geographic sampling coordinates.

## Running a conStruct analysis and constructing figures

Now that all inputs are ready, a conStruct analysis can be run.
```
my.run <- conStruct(spatial = TRUE, 
                    K = 3, 
                    freqs = conStruct.data$allele.frequencies,
                    geoDist = conStruct.data$geoDist, 
                    coords = conStruct.data$coords,
                    prefix = "spK3")
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
load("spk3_data.block.Robj")
load("spk3_conStruct.results.Robj")
```
Now, different plots can be made. A quick note: the figures created using the sample data will not look the same as the figures shown, which were made using the full dataset.
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

## Comparing Models
After running a conStruct analysis, it is best to find which K value, or the number of layers, best fits your data. This can be done by using conStruct's integrated cross-validation command:
```
my.xvals <- x.validation(train.prop = 0.9,
                         n.reps = 8,
                         K = 1:3,
                         freqs = conStruct.data$allele.frequencies,
                         data.partitions = NULL,
                         geoDist = conStruct.data$geoDist,
                         coords = conStruct.data$coords,
                         prefix = "example",
                         n.iter = 1000,
                         make.figs = FALSE,
                         save.files = FALSE,
                         parallel = FALSE,
                         n.nodes = NULL)
```
This tool runs 8 cross-validation analyses, from K=1 to K=3, by randomly samples 90% of the loci in the dataset at each of these values. This results in 24 individual conStruct analyses being run, which can lead to a long runtime. It is best to leave make.figs=FALSE as leaving it on can accumulate a bunch of figures. The command to plot this data is:
```
sp.results <- as.matrix(
  read.table("example_sp_xval_results.txt",
             header = TRUE,
             stringsAsFactors = FALSE)
)
nsp.results <- as.matrix(
  read.table("example_nsp_xval_results.txt",
             header = TRUE,
             stringsAsFactors = FALSE)
)

sp.CIs <- apply(sp.results,1,function(x){mean(x) + c(-1.96,1.96) * sd(x)/length(x)})
nsp.CIs <- apply(nsp.results,1,function(x){mean(x) + c(-1.96,1.96) * sd(x)/length(x)})

par(mfrow=c(1,2))
plot(rowMeans(sp.results),
     pch=19,col="blue",
     ylab="predictive accuracy",xlab="values of K",
     ylim=range(sp.results,nsp.results),
     main="cross-validation results")
points(rowMeans(nsp.results),col="green",pch=19)

plot(rowMeans(sp.results),
     pch=19,col="blue",
     ylab="predictive accuracy",xlab="values of K",
     ylim=range(sp.CIs),
     main="spatial cross-validation results")
segments(x0 = 1:nrow(sp.results),
         y0 = sp.CIs[1,],
         x1 = 1:nrow(sp.results),
         y1 = sp.CIs[2,],
         col = "blue",lwd=2)
```
Running these commands here results in a figure that helps you determine which K value is best, by showing you which layer has the best predictive accuracy. This looks like:

<img width="696" height="288" alt="Screenshot 2026-04-22 at 4 02 32 PM" src="https://github.com/user-attachments/assets/e48d72b8-9ada-4fdb-8c1e-745e949af921" />


## Working with new Dataset
NOTE: The following section is project-specific, filepaths may differ

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
2. Process Data and Convert Structure file to conStruct format
```
install.packages("dplyr")
library(dplyr)

#find sample id of NA values and remove the loci
plant.na <- plant %>% filter(if_any(everything(), is.na))
na.sampleid <- plant.na %>% pull(sample_id)
na.sampleid

#note that missing.datum was changed to -1
#the sample id's that showed to have missing coordinates in the plantmetadata.txt file were manually deleted
population.data <- structure2conStruct(
  infile = "/home/data/Project5/populations.structure",
  onerowperind = FALSE,
  start.loci = 2,
  start.samples = 3,
  missing.datum = -1,
  outfile = "~/conStruct_project/structurefileConStructData.txt"
)

head(population.data)

plant.no.na <- sampling.coords[complete.cases(sampling.coords),]
m.plant <- as.matrix(a)

```
3. Create Sampling Coordinates 
```
sampling.coords <- plant %>% 
  select(lat, lon)

plant.no.na <- sampling.coords[complete.cases(sampling.coords),]
m.plant <- as.matrix(a)
head(m.plant)

```
4. Create Distance Matrix
```
install.packages("geosphere")
library(geosphere)

geosphere::distGeo
?distGeo
geoDist <- distm(b)
head(geoDist)

```
5. Combine data to a list
```
grillodata <- list(
  population = population.data,
  coords = m.plant,
  distance = geoDist
)

#view each part here
grillodata$population
grillodata$coords 
grillodata$distance

```
5. Run conStruct
```
#run conStruct
my.run <- conStruct(spatial = TRUE, 
                    K = 2, 
                    freqs = grillodata$population, 
                    geoDist = grillodata$distance,  
                    coords = grillodata$coords, 
                    prefix = "spK2", 
                    n.chains = 1, 
                    n.iter = 500, 
                    make.figs = TRUE, 
                    save.files = TRUE)
```



