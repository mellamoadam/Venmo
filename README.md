This repository contains R code to analyze Venmo transactions from CSV files. The script processes your transaction data and generates summaries and visualizations of your Venmo account activity.

## Instructions
1. Export your Venmo transaction history as CSV files from the Venmo website.
2. Put all your downloaded CSV files into the folder specified by the variable `folderPath` in the script.
3. The script will read the CSV files from the folder, clean and transform the data, and produce analysis outputs.

## How to Use
1. Open the R script.
2. Set the `folderPath` variable to the directory containing your Venmo CSV files.
3. Set the `startDate` and `endDate` variables to include transactions from the dates of interest.
4. Set the variable `user` to your name that matches the exact spelling used in the `allData` To or From columns to correctly determine who initiated or received payments.
5. Run the script. This will generate a pdf file in the path specified in `folderPath`.
6. Outputs will include transaction summaries and visualizations, such as bar charts and pie charts showing payment flows and balances.

## Requirements
R (version 4.0 or higher recommended)
Packages: dplyr, ggplot2, tidyr

You can install missing packages with:
install.packages(c("dplyr", "ggplot2", "tidyr", "readr"))

## Notes
The script assumes that the CSV files contain columns such as Amount (total), From, To, and DateTime.
Negative amounts represent payments sent, positive amounts represent payments received.

