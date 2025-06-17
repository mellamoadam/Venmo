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

# Data Visualization

## Net Transactions
To get a general overview of the incoming and outgoing transactions, we make a pie chart that shows overall dollar value of payments made versus payments received. 
<br>
<br>
<details>
<summary>Net Tx Code</summary>
<br>
  
```r
pieData = allData %>%
  group_by(PaymentType) %>%
  summarise(TotalAmount = sum(AmountTotal))


# Pie chart
ggplot(pieData, aes(x = "", y = abs(TotalAmount), fill = PaymentType)) +
  geom_col(width = 1, color = "white") +
  coord_polar(theta = "y") +
  geom_text(aes(label = paste0("$", round(abs(TotalAmount), 2))), 
            position = position_stack(vjust = 0.5), 
            size = 4) +
  scale_fill_manual(values = c("Received" = "#6BAF75", "Paid" = "#D46A6A")) +
  theme_void() +
  theme(legend.title = element_blank())
```
</details>

![NetTxPie](https://github.com/mellamoadam/Venmo/blob/main/Images/NetTxPie.png)


## Payment Initiation
Each transaction was previously categorized based on whether the `user ` initiated the payment or request, or whether it was initiated by someone else, using both the direction of the transaction and the sign of the amount (positive or negative). By breaking down the information in the previous graph, we see how the incoming and outgoing payments are split by who initiated the payment.
<br>
<br>
<details>
<summary>Payment Initiation Code</summary>
<br>
  
```r
barData = allData %>%
  group_by(PaymentType, PaymentInitiator) %>%
  summarise(TotalAmount = sum(AmountTotal), .groups = "drop")


ggplot(barData, aes(x = PaymentType, y = abs(TotalAmount), fill = PaymentInitiator)) +
  geom_col(position = "stack") +
  scale_fill_manual(
    values = c("Initiated" = "#6baed6", "Other" = "#9ecae1")
  ) +
  labs(y = "Total Amount ($)", x = "Payment Type", fill = "Payment Initiator") +
  theme_minimal()
```
</details>

![PaymentInitiation](https://github.com/mellamoadam/Venmo/blob/main/Images/PaymentInitiation.png)



## Venmo Balance
The Venmo balance chart provide a visual summary of your net activity and bank transfers, using color to distinguish between positive (green) and negative (red) flow relative to the Venmo account. This offers a quick overview of your overall Venmo usage and account flow.
<br>
<br>
<details>
<summary>Venmo Balance Code</summary>
<br>
  
```r

# Create df with net income and bank transfers. Numbers are from the venmo balance perspective
venmoBalaceDF = data.frame(net = sum(allData$AmountTotal), bank = sum(bankTransferDF$AmountTotal))

venmoBalaceDFLong = pivot_longer(venmoBalaceDF, cols = everything(), names_to = "Tx", values_to = "balance") %>%
  mutate(fillColor = ifelse(balance >= 0, "Positive", "Negative"))

ggplot(venmoBalaceDFLong, aes(x = Tx, y = balance, fill = fillColor)) +
  geom_col() +
  scale_fill_manual(values = c(Positive = "#6BAF75", Negative = "#D46A6A")) +
  scale_x_discrete(labels = c(net = "Net Payments", bank = "Bank Transfers")) +
  labs(x="", y = "Into Venmo Account ($)", title = "Venmo Balance") +
  theme_minimal() +
  theme(legend.position = "none",
    plot.title = element_text(hjust = 0.5))
```
</details>

![VenmoBalance](https://github.com/mellamoadam/Venmo/blob/main/Images/VenmoBalance.png)






