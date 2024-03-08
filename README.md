# Melbourne Housing Sales
![pexels-julia-foroni-4664660](https://github.com/ksadangrit/melbourne-housing/assets/156267785/dc8c956c-4efc-48db-95cd-8dd9bc039572)


_Photo by Julia Foroni from [Pexels](https://www.pexels.com/photo/photo-of-assorted-colored-houses-4664660)_
## Introduction
The Melbourne housing market presents a fascinating landscape of trends and patterns that can provide valuable insights for homeowners, investors, and policymakers alike. In this personal data analysis project, I aim to explore the dynamics of the Melbourne housing market using a dataset of property sales. 

This property dataset had been extracted from Wikipage by Amal A Biju. Click [here](https://opendatacommons.org/licenses/dbcl/1-0/) to view the license. The dataset includes property sales data from Melbourne, including sale prices, property types, locations, and other relevant attributes.

**Main objectives**

* Identify trends in housing prices over time and space.
* Explore the relationship between housing prices and key factors such as distance from the CBD, property type, and suburb.
* Uncover any significant patterns or outliers in the data.
* Provide insights and recommendations for homebuyers, sellers, investors and policy makers.

Descriptive statistics and data visualization techniques will be used to explore the relationshop between housing prices and other factors. **R** is the tool that will be used for this analysis.

## My analytical workflow


## Importing data
We will download the dataset from [kaggle](https://www.kaggle.com/datasets/amalab182/property-salesmelbourne-city) ans save it into my computer file.

Firstly, we will download all the packages and libraries that will be used in this project.
```
# Install packages
install.packages("tidyverse")
installed.packages("janitor")
install.packages("scales")

# Download libraries
library(tidyverse)
library(janitor)
library(lubridate)
library(dplyr)
library(scales)
```
Then we'll import the CSV file for the housing data, clean column names and trim whitespace from all columns. We'll name this dataset `melbourne_housing`.
```
melbourne_housing <- read_csv("/Users/yanhua1/Downloads/melbourne_property_sales.csv") %>% 
  clean_names() %>% 
  mutate_if(is.character, trimws)
```

### Data Dictionary
For full dictionary click [here](https://www.kaggle.com/datasets/amalab182/property-salesmelbourne-city).

* `suburb`: Name of the suburb where the property is located
* `address`: Street address of the property
* `rooms`: Number of rooms in the property (excluding bathrooms and other non-living spaces)
* `price`: Sale price of the property in Australian dollars (AUD)
* `method`: Method of sale (e.g., property sold, property sold prior, property passed in, vendor bid, sold after auction)
* `type`: Type of property (e.g., house, townhouse, apartment/unit)
* `seller`: Real estate agency or agent handling the sale
* `date`: Date of the sale
* `distance`: Distance from the property to Melbourne central business district (CBD) in kilometers
* `regionname`: Name of the region where the property is located (e.g., Eastern Metropolitan, Northern Metropolitan, Southern Metropolitan, Western Metropolitan)
* `propertycount`: Number of properties that exist in the suburb
* `bedroom`: Number of bedrooms in the property (including any non-living spaces that could be used as bedrooms)
* `bathroom`: Number of bathrooms in the property
* `car`: Number of car spaces associated with the property
* `landsize`: Land size of the property in square meters
* `building_area`: Total building area of the property in square meters
* `council_area`: Name of the local government area where the property is located


## Cleaning data
We'll start the cleaning step by checking how many rows there are in this dataset using the following code.
```
nrow(melbourne_housing)
```

```
# Check how many rows there are in the melbourne_housing dataset
nrow(melbourne_housing)

# Check the column names and data structure
colnames(melbourne_housing)
str(melbourne_housing)
```

We'll now check whether all the rows are unique and there are any duplicates.
```
# Check if there are any duplicate rows in the dataframe using if-else statement and duplicated() function
if (any(duplicated(melbourne_housing))) {
  print("There are duplicate rows in the dataframe.")
} else {
  print("All rows in the dataframe are unique.")
}
```

As we're confirmed that there is no duplicates, we'll run the below code to see how many NA values there are for each column.
```
colSums(is.na(melbourne_housing))
```
![Screen Shot 2024-03-08 at 1 49 47 PM](https://github.com/ksadangrit/melbourne-housing/assets/156267785/ffadf58f-6dbb-42d5-8248-ba33879ef17b)

After running the code, the result shows that the following columns do not have any missing values: suburb, address, rooms, type, price, method, seller_g. 

I won't delete all the rows with any NA values as those rows still have values for most columns. I will try to replace the NA values with missing data that can be obtained from the corresponding values in a different column. For the rows with missing data that cannot be obtained, I will exclude them when it comes to specific calculation or analysis.

There are 4 columns with only 1 missing value and they are `distance`, `postcode`, `regionname` and `propertycount`. The missing values for those columns, except for `distance` can be found based on the suburb name. Therefore, we’ll find the row that has a missing value in the `postcode` column using the below code. 

```
# Find out the name of the suburb that has the a missing value in the distance column
na_row <- melbourne_housing %>% 
  filter(is.na(postcode))
view(na_row) # To view the row in a separate window.
```

It appears that the row with the footscray_lot suburb is the only row with missing values for the  `postcode`, `regionname` and `propertycount` column. 
```
# Find the missing values based on the suburb name by pulling other rows with the same suburb
footscray_lot <- melbourne_housing %>% 
  filter(suburb == "Footscray Lot")
view(footscray_lot)
```

There is only one row with `footscray_lot` suburb. It appears that this row was created with the incorrect suburb name as Footscray lot suburb doesn't exist and this house address is in the Footscray suburb. We'll then find the missing values from the rows with `footscray` suburb

```
# Find the missing values for our row by filtering in all the rows that have Footscray suburb
footscray <- melbourne_housing %>% 
  filter(suburb == "Footscray")
view(footscray)
```

Now that we has obtained the missing values for the mentioned columns, we'll update the row using the below code.
```
melbourne_housing <- melbourne_housing %>%
  mutate(
    suburb = ifelse(row_number() == 14441, "Footscray", suburb),
    postcode = ifelse(row_number() == 14441, 3011, postcode),
    regionname = ifelse(row_number() == 14441, "Western Metropolitan", regionname),
    propertycount = ifelse(row_number() == 14441, 7570, propertycount)
  )
```
The other columns contain specific missing values which we cannot obtain based on the suburb names. 

Next, we'll extract month and year from the date column deselect the columns that are not needed such as longtitude, latitude and the id. Well also rename some variables and columns to make them more readable. We'll name the clean dataset as `melbourne_housing_clean`.
```
melbourne_housing_clean <- melbourne_housing %>%
  mutate(month = month(dmy(date)),
         year = year(dmy(date))) %>%
  mutate(type = recode(type, 
                       "u" = "Apartment/Unit",
                       "h" = "House",
                       "t" = "Townhouse"),
         method = recode(method,
                         S = "property sold",
                         SP = "property sold prior",
                         PI = "property passed in",
                         VB = "vendor bid",
                         SA = "sold after auction")) %>% 
  select(       # De-select columns that are not needed
    -x1,
    -lattitude,
    -longtitude
      ) %>% 
  rename(bedroom = bedroom2,
         seller = seller_g) 
```

A look into the clean dataset.
```
glimpse(melbourne_housing_clean)
```

## Analysis





