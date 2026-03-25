# 383ConStruct

\[Instructions for installing in RStudio]

To download the conStruct package, run the following command in the console
```
install.packages("conStruct")
```
Running this command will also download dependencies for conStruct


\[Instructions for using data to recreate Figure 7 of Black Bears and Poplars study]

The following example uses the example Construct.data file included in the R package:
```
my.run <- conStruct(spatial = TRUE, 
                    K = 3, 
                    freqs = conStruct.data$allele.frequencies,
                    geoDist = conStruct.data$geoDist, 
                    coords = conStruct.data$coords,
                    prefix = "spK3")
```
Setting spatial to true outputs a spatial model, false for a non-spatial model.
K represents the number of layers
freqs includes the allele frequency data
geoDist represents the geographic distance matrix
coords includes the sampling coordinates data
