# Overview
This tutorial provides a walkthrough of the framework used to generate your venmo transaction analysis.
<br>
<br>
# Data Preparation
## Load Packages
<br>
<br>
<details>
<summary>Package loading code</summary>
<br>
  
```r
#install.packages("ggplot2")
#install.packages("dplyr")
#install.packages("tidyr")
library(ggplot2)
library(dplyr)
library(tidyr)
```
</details>
## Environment Set-Up
<br>
<br>
<details>
<summary>User Inputs</summary>
<br>

```r
################################# USER INPUTS  #################################
startDate = "2025-01-15"
endDate = Sys.Date()
user = "Adam Aslam"
folderPath="/Users/adamaslam/Desktop/MyCode/Venmo/"
################################################################################  

setwd(folderPath)

# Move all .csv files to folderPath
rawFiles = list.files(path = folderPath, pattern = paste0("VenmoStatement.+.csv"))

# Format the start and end dates of the analysis to go from midnight on the first morning to midnight at the end of the endDate in POSIXct
startDate = as.POSIXct(as.Date(paste0(startDate, "T00:00:00")), format="%Y-%m-%d", tz="UTC")
endDate = as.POSIXct(endDate + 1, format="%Y-%m-%dT%H:%M:%S", tz="UTC")
```
</details>



