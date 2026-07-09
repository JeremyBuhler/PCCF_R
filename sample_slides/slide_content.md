<!-- .slide: data-transition="none-in none-out" -->
## PCCF+ R Program 
<br/><br/>
  
**Author/Institution**  
email@address.ca <!-- .element style="color:lightblue" --> 


---

<!-- .slide: data-transition="none-in slide-out" -->
### Steps 
1. Download PCCF+ files
3. Modify PCCF+ R script
4. Run PCCF+ R script


--





---

<!-- .slide: data-background="green" data-transition="slide-in none-out" -->
## 1. Download PCCF+ files 


---

<!-- .slide: data-transition="none-in slide-out -->

Find desired PCCF+ at <https://odesi.ca>


---

... 


---

<!-- .slide: data-background="green" data-transition="none-in slide-out" -->
## 2. Modify PCCF+ R script





---

Open your copy of the PCCF+ R script in RStudio, then update the script as shown in the next slides

---

This is the script as downloaded

```r[]
# PCCF+ Version 8D
# 
# R routine for automated geographic coding from postal codes using the Postal Code Conversion
# File (PCCF) and Weighted Conversion File (WCF).
# 
# Reads incoming postal codes from a user specified input file and creates a geocoded output file
# as well as a "problem" file that includes records that may potentially be a problem. Diagnostics
# are also included as output Word documents with a summary of geocoding results and several additional
# diagnostic variables to aid in assessing geocoding accuracy.
# 
# Version 8 of PCCF+ includes options for geocoding residential or institutional postal codes.
# Residential postal codes are geocoded using a weighted conversion file, which allocates duplicate
# postal codes according to the underlying population distribution. Institutional geocoding is for
# coding office and institutional (hospital, businesses, etc...) and links rural postal codes to the
# the location of the rural post office rather than using population-weighted allocation.
#
# PCCF+ was developed and written by the Health Analysis Division,
# Statistics Canada, 100 Tunney's Pasture Driveway, Ottawa ON  K1A 0T6
# Support can be obtained at statcan.pccfplus-fccpplus.statcan@statcan.gc.ca  

# List of required packages 

required_packages <- c("dplyr", "readr", "haven", "tidyverse", "stringr", "gtsummary", "gt", "data.table")

# The packages must be successfully installed and loaded. Missing packages may lead to errors and prevent the 
# program from functioning as intended.

# Function to check and install missing packages
check_and_install <- function(packages) {
  for (pkg in packages) {
    if (!requireNamespace(pkg, quietly = TRUE)) {
      cat(paste("Package", pkg, "is not installed. Installing now...\n"))
      install.packages(pkg, method = "libcurl", extra = "-k")
    }
    library(pkg, character.only = TRUE)
    cat(paste("Package", pkg, "loaded successfully.\n"))
  }
} 

# Run the function
check_and_install(required_packages)

# Step 1: Set the installation folder path
installDir <- "//stpcflr-sasfs50/Saslibe/PCCFplus_FCCPplus/Processing/Programs/PCCF8D"
              
# Step 2: Specify the input data library, the input file, and the file type
# The file must contain the following fields:
# - PCODE: Postal code (no spaces), character variable
# - ID: Unique identifier, character variable

# Set the input data library (folder path where dataset is located)

inData <- "//stpcflr-sasfs50/Saslibe/PCCFplus_FCCPplus/Processing/Programs/PCCF8D/sample" # Folder path

inFile <- "sampledat" # File name (without extension)

file_type <- "CSV"  # Valid options: "R", "CSV", or "SAS"

# Function to read the input data based on file type
input_data <- switch(
  file_type,
  
  # Read SAS file
  "SAS" = read_sas(file.path(inData, paste0(inFile, ".sas7bdat"))),
  
  # Read R file (.rds)
  "R" = readRDS(file.path(inData, paste0(inFile, ".rds"))),
  
  # Read CSV file
  "CSV" = fread(file.path(inData, paste0(inFile, ".csv")), stringsAsFactors = FALSE),
  
  # Default case for unsupported file types
  stop("Invalid file type. Please specify 'SAS', 'R', or 'CSV'.")
)

# Step 3: Specify the output data library and file name
outData <- "//stpcflr-sasfs50/Saslibe/PCCFplus_FCCPplus/Processing/Programs/PCCF8D/sample"
outName <- "sampledat_R2"

# Problem records and diagnostics are provided in the problem file, along with reference information
# for possible solutions. Reviewing it is strongly recommended to assess the quality of the geocoding.

# Problem output file (optional): 1 = include the problem file, 0 = omit the problem file

problem_file <- 1 # The problem file is included by default

# A summary of the geocoding process for geocoded tables and problem tables, including the number of records in 
# each link type, is printed in the Word documents below. The summary also shows the distribution of records 
# by the number of geographic codes which were assigned.
  
# Step 4: Specify the output file path for the Word documents
file_path <- file.path(outData, "geocoded_tables.docx")
file_pathx <- file.path(outData, "problem_tables.docx")

# Step 5: Select the version to run (Residential or Institutional)
codeVersion <- 0  # 0 for Residential, 1 for Institutional

# Optional: Specify additional settings
seedVal <- 0     # Random Seed Value
Version <- "8D"  # PCCF+ Release Version
set.seed(seedVal)

# Step 6: Specify variables and output options
# Allow users to select the groups of variables to keep

# IDs are always kept
keepIDs <- c("CODER", "Version", "ID", "PCODE")

# Choice=1: keep standard Census geography
keepSCG <- c("PR", "DAUID", "DB", "DB_ir2021", "CSDuid", "CSDname", "CSDtype", "CMA", "CMAtype", "CMAname",
             "CTname", "Tracted", "SACcode", "SACtype", "CCSuid", "FEDuid", "FEDname", "LAT", "LONG", "DPLuid",
             "DPLtype", "DPLname", "ERuid", "ERname", "CARUID", "CARname", "PopCtrRAPuid", "PopCtrRAname",
             "PopCtrRAtype", "PopCtrRAclass", "CSIZE", "CSIZEMIZ")

# Choice=2: keep accuracy and precision-related variables
keepQIs <- c("SLI", "Rep_Pt_Type", "RPF", "PCtype", "DMT", "H_DMT", "DMTDIFF", "PO", "QI", "Source", 
             "Link_Source", "LINK", "nCD", "nCSD", "PREC")

# Choice=3: keep income category related variables
keepINCOME <- c("BTIPPE", "ATIPPE", "QABTIPPE", "QNBTIPPE", "DABTIPPE", "DNBTIPPE", "QAATIPPE", "QNATIPPE",
                "DAATIPPE", "DNATIPPE", "IMPFLG")

# Choice=4: keep health region-related variables
keepHRs <- c("HRuid", "HRename", "HRfname", "AHRuid", "AHRename", "AHRfname")

# Choice=5: keep Canada Post-related and supplementary variables
keepSupps <- c("Comm_Name", "AIRLIFT", "InstFlag", "ResFlag", "InuitLands")

# Choice=6: keep historical census geography from the 1981 Census onward
keepHistories <- c("DA11uid", "DB11uid", "DA06uid", "DA01uid", "EA96uid", "EA91uid", "EA86uid", "EA81uid",
                   "DA16uid", "DB16uid")

# Define user's choice(s) - select one or multiple choices. NB: Exclude 0 to make a choice

choice <- c(0, 1, 2, 3, 4, 5, 6)  # Keep ALL categories

# choice <- c(4)  # For example: Keep only the health region variables

# Function to build selected variables
buildKeepStatement <- function() {
  selectedVars <<- keepIDs
  if (0 %in% choice) {
    selectedVars <<- unique(c(selectedVars, keepSCG, keepQIs, keepINCOME, keepHRs, keepSupps, keepHistories))
    return()
  }
  
  category_map <- list(
    `1` = keepSCG,
    `2` = keepQIs,
    `3` = keepINCOME,
    `4` = keepHRs,
    `5` = keepSupps,
    `6` = keepHistories
  )
  
  for (i in choice) {
    if (as.character(i) %in% names(category_map)) {
      selectedVars <<- unique(c(selectedVars, category_map[[as.character(i)]]))
    }
  }
}

# Build the selectedVars
buildKeepStatement()

# Run the line below to execute the rest of the program, which does not require any modification
source(paste0(installDir, "/DATA_R/PCCFplus_8D_do_not_edit.R"))


```

---

<!-- .slide: data-auto-animate="true" -->

Update this line...

```r [42: 2-3]

# Step 1: Set the installation folder path
installDir <- "//stpcflr-sasfs50/Saslibe/PCCFplus_FCCPplus/Processing/Programs/PCCF8D"


```

---

<!-- .slide: data-auto-animate="true" -->

...and these...


```r [50: 4,6]

# Set the input data library (folder path where dataset is located)

inData <- "//stpcflr-sasfs50/Saslibe/PCCFplus_FCCPplus/Processing/Programs/PCCF8D/sample" # Folder path

inFile <- "sampledat" # File name (without extension)

```

---

...and so on!
