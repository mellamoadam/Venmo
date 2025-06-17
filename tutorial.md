# Overview
This tutorial provides a walkthrough of the framework used to generate your venmo transaction analysis.
<br>
<br>
# Data Preparation
## Load Packages
<br>
<br>
<details>
<summary>Package Loading Code</summary>
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
<summary>Set-Up Code</summary>
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

## Data Cleaning
<br>
<br>
<details>
<summary>Cleaning Code</summary>
<br>

```r

# Combine and clean all transaction reports

# Combine
allData = do.call(rbind, lapply(rawFiles, function(f) read.csv(paste0(folderPath, f))))

# Remove rows with <= 4 non-empty values
allData = allData[rowSums(!is.na(allData) & allData != "") > 4, ]

# Set the second row as column names
colnames(allData) = as.character(unlist(allData[1, ]))

# Drop the first row and col
allData = allData[-c(1), -c(1)]

# Drop repeated header cols
allData = allData[allData$ID != "ID", ]

# Sort by dateTime
allData = allData[order(allData$Datetime), ]

# Grab transactions from dates of interest
allData = allData[allData$Datetime >= startDate & allData$Datetime <= endDate, ]

# Reset row names
rownames(allData) = NULL

# Restructure data classes
allData$dateTime = as.POSIXct(allData$Datetime, format="%Y-%m-%dT%H:%M:%S", tz="UTC")
allData = allData[ , !(names(allData) == "Datetime")]

allData$AmountTotal = as.numeric(gsub("[$ ,]", "", allData[["Amount (total)"]]))
allData = allData[ , !(names(allData) == "Amount (total)")]


# Make separate df for bank transfers and drop from allData
bankTransferDF = allData[allData$Type == "Standard Transfer",]
allData = allData[!(allData$Type == "Standard Transfer"),]


# Add Payment Type column ("Received" or "Paid")
allData = allData %>%
  mutate(PaymentType = ifelse(AmountTotal >= 0, "Received", "Paid")) 

# Add Payment Initiator column (Initiated or Requested)
allData = allData %>%
  mutate(PaymentInitiator = case_when(
    From == user & AmountTotal > 0 ~ "Initiated", # You sent request and received payment
    From == user & AmountTotal < 0 ~ "Initiated", # You sent money without being requested
    To == user & AmountTotal > 0 ~ "Other", # Someone else sent money without being requested
    To == user & AmountTotal < 0 ~ "Other", # You sent money after being requested
    TRUE ~ "Unknown"
  ))
```
</details>

## Environment Set-Up










