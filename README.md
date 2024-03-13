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

### My analytical workflow


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
In this section, we will manipulate and analyze the data to gain a comprehensive understanding of the trends and patterns in the Melbourne housing market. Our objective is to explore the relationships between various factors and their impact on house prices. In this analysis, my primary emphasis will be on evaluating total sales performance. While I will occasionally reference the total number of properties sold where relevant, it will not be as prominent as the discussion on sales performance.

I will provide the code used for these calculations. While I will include results and visualizations where appropriate, the majority of findings will be presented through graphs or charts in the visualization stage of this project.

### Max, Min, Avg, Median and Sd
Taking a deep dive into Melbourne's housing market data, I want to get a clear picture of what's happening overall. To do this, we will crunch the numbers and summarize the key insights  such as max, min, average, median and standard deviation values into a handy table. This snapshot of the data gives a quick overview of the highs, lows, and averages, helping to make sense of the broader trends for each types of properties in the market. 

```
sum_table <- melbourne_housing_clean %>% 
  group_by(type) %>% 
  summarise(max = comma(max(price)),
            min = comma(min(price)),
            avg = comma(mean(price)),
            median = comma(median(price)),
            sd = comma(sd(price)))
```
![Screen Shot 2024-03-12 at 1 00 13 PM](https://github.com/ksadangrit/melbourne-housing/assets/156267785/f3bf9e2d-e93e-422c-8f05-4f635e2bea09)

From the table, we can see that the most expensive house sold in Melbourne is 2.49 times more than the most expensive apartment and 2.5 times more expensive than the most expensive townhouse. Surprisingly, the chapest townhouse is more expensive the the cheapest house and the cheapest apartment is only under $100,000 which is less than the cheapest townhouse and the cheapest house

When we look at the average numbers and the median numbers, it is clear that all the average prices are higher than the median prices, this means that the data is generally **skewed right** and a few values are latger than the rest. The sd values also confirms this interpretation as they are all larger than $200,000.

### Most common types of properties 
We'll find out the most common type of properties based on sales and total number using the below code.
```
# The most common type of houses sold in Melbourne
house_types <- melbourne_housing_clean %>% 
  group_by(type) %>% 
  summarise(number = n(),
            total_sales = sum(price))
```

We'll now separate the results by years to see if that makes any difference.
```
house_types_year <- melbourne_housing_clean %>% 
  group_by(type, year) %>% 
  summarise(number = n(),
            total_sales = sum(price))
```

### Yearly
We will employ various time measurements to analyze the sales data. To begin, we'll determine the total number of property sales and the number of properties sold in each year using the code provided below.

**Total sales and number of properties sold**
```
sales_year <- melbourne_housing_clean %>% 
  group_by(year) %>% 
  summarise(number_sold = n(),
            total_sales = sum(price))
```

**The median price**

We'll calculate the **median** price for each type of property in each year. I've opted for the median over the mean, particularly for time-based trends, as outliers with large values could distort the portrayal of the property market. This differs from our approach for suburb averages, which are area-based.
```
median_per_year <- melbourne_housing_clean %>% 
  group_by(type, year) %>% 
  summarise(median_price = median(price)) 
```

### Monthly
We'll check out the total number of property sales and the number of properties sold in a monthly basis.
```
# Total number of properties sold and total sales per month 
sales_month <- melbourne_housing_clean %>% 
  group_by(month) %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) 
```

Instead of calculating the median house price for each month, we'll compare the **total sales** for each month. Since months represent a shorter time scale compared to years, variations in median house prices for each month may be influenced by factors such as housing market availability. Therefore, focusing on total sales can offer valuable insights into fluctuations in housing demand throughout the year.

**Total sales and number of properties sold**
```
sales_month <- melbourne_housing_clean %>% 
  group_by(month) %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) 
```

**Total sales separated by types**
```
sales_month_type <- melbourne_housing_clean %>% 
  group_by(type, month) %>% 
  summarise(total_sales = sum(price))
```

### Suburbs
I opted to calculate the **average house price** for each suburb rather than median because despite potential outliers, it offers a more comprehensive reflection of suburb **desirability,** capturing the overall pricing trend across the area rather than focusing solely on a single central value. In the subsequent analysis, we'll compile a list of the top 10 suburbs with the highest average prices for each type of property.

#### For Houses
```
# Top 10 suburbs with the highest house prices
avg_house <- melbourne_housing_clean %>%
  group_by(suburb) %>%
  filter(type == "House") %>% 
  summarise(avg_price = mean(price)) %>%
  arrange(desc(avg_price)) %>% 
  head(10)
```
#### For Apartment/ unit
```
# Top 10 suburbs with the highest apartment prices
avg_apartment <- melbourne_housing_clean %>%
  group_by(suburb) %>%
  filter(type == "Apartment/Unit") %>% 
  summarise(avg_price = mean(price)) %>%
  arrange(desc(avg_price)) %>% 
  head(10)
```

#### For Townhouse
```
# Top 10 suburbs with the highest townhouse prices
avg_townhouse <- melbourne_housing_clean %>%
  group_by(suburb) %>%
  filter(type == "Townhouse") %>% 
  summarise(avg_price = mean(price)) %>%
  arrange(desc(avg_price)) %>% 
  head(10)
```

### Council area
Similar to the suburbs, we'll rank top 10 council area with the highest average price for different types of properties. We'll also rank all the council areas based on the total sales.
### Total sales
```
council_sales <- melbourne_housing_clean %>% 
  group_by(council_area) %>% 
  filter(!is.na(council_area)) %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) %>%
  arrange(desc(total_sales))
```

### Top 10 with the highest average house price
```
avg_house_council <- melbourne_housing_clean %>%
  group_by(council_area) %>%
  filter(type == "House", !is.na(council_area)) %>% 
  summarise(avg_price = mean(price)) %>% 
  arrange(desc(avg_price)) %>% 
  head(10)
```

### Top 10 with the highest average apartment/unit price
```
avg_apartment_council <- melbourne_housing_clean %>%
  group_by(council_area) %>%
  filter(type == "Apartment/Unit", !is.na(council_area)) %>% 
  summarise(avg_price = mean(price)) %>%
  arrange(desc(avg_price)) %>% 
  head(10)
```

### Top 10 with the highest average townhouse price
```
avg_townhouse_council <- melbourne_housing_clean %>%
  group_by(council_area) %>%
  filter(type == "Townhouse", !is.na(council_area)) %>% 
  summarise(avg_price = mean(price)) %>%
  arrange(desc(avg_price)) %>% 
  head(10)
```

### Regions
For regions, we'll calculate the total sales and the average price for different properties in each region.
#### Total sales
```
region_sales <- melbourne_housing_clean %>% 
  group_by(regionname) %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) %>%
  arrange(desc(total_sales))
```

#### Average price
We'll consolidate the average prices for different property types into a single dataframe. This will serve as the basis for creating graphs in the next section.

```
avg_region <- melbourne_housing_clean %>% 
  group_by(regionname, type) %>% 
  summarise(avg_price = mean(price)) 
```

### Sellers
We'll initially rank all real estate agents or companies based on their overall sales performance and find out who are in the top 5. Then, we'll further rank them based on the total sales they made for each type of property.

#### Top 5 for overall sales
```
sales_seller <- melbourne_housing_clean %>% 
  group_by(seller) %>% 
  summarise(total_sales = sum(price)) %>% 
  arrange(desc(total_sales)) %>% 
  head(5)
```

We'll also rank all sellers based on the total number of properties they sold to investigate potential variations in ranking.
#### Top 5 based on number of properties sold
```
num_seller <- melbourne_housing_clean %>% 
  group_by(seller) %>% 
  summarise(number_sold = n()) %>% 
  arrange(desc(number_sold)) %>% 
  head(5)
```

The below rankings will be based on the sales for different types of properties only.
#### Top 5 sellers for house
```
top_sellers_house <- melbourne_housing_clean %>% 
  group_by(seller) %>% 
  filter(type == "House") %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) %>% 
  arrange(desc(total_sales)) %>% 
  head(5)  
```

#### Top 5 sellers for apartment/unit
```
top_sellers_apartment <- melbourne_housing_clean %>% 
  group_by(seller) %>% 
  filter(type == "Apartment/Unit") %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) %>% 
  arrange(desc(total_sales)) %>% 
  head(5)
```

#### Top 5 sellers for townhouse
```
top_sellers_townhouse <- melbourne_housing_clean %>% 
  group_by(seller) %>% 
  filter(type == "Townhouse") %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) %>% 
  arrange(desc(total_sales)) %>% 
  head(5) 
```

### Method
#### Most common selling method based on the total sales
```
method_sales <- melbourne_housing_clean %>% 
  group_by(method) %>% 
  summarise(total_sales = sum(price))
```

#### Total sales based on types 
```
method_sales2 <- melbourne_housing_clean %>% 
  group_by(method, type) %>% 
  summarise(number = n(),
            total_sales = sum(price))
```

### Property Count
We'll create a dataframe that includes all suburbs along with the counts of properties and houses sold within each suburb. This dataframe will serve as the basis for creating visualizations in the next section.
```
count_suburb <- melbourne_housing_clean %>% 
  group_by(suburb, propertycount) %>% 
  count() %>% 
  rename(total = n) %>% 
  arrange(desc(total))
```

### Rooms
We'll calculate the percentage of houses sold based on the number of rooms they possess. This analysis will shed light on the most common number of rooms among the houses sold in the market.
```
rooms <- melbourne_housing_clean %>%
  mutate(condition = case_when(
    rooms <= 3 ~ "<= 3",
    rooms > 3 & rooms <= 6 ~ "3 < and <= 6",
    rooms > 6 & rooms <= 9 ~ "6 < and <= 9",
    rooms > 9 ~ "> 9",
    TRUE ~ "Other"
  )) %>%
  group_by(condition) %>%
  summarise(
    count = n(),
    percentage = round((n() / nrow(melbourne_housing_clean)) * 100,2)
  )
```

![Screen Shot 2024-03-13 at 2 32 01 PM](https://github.com/ksadangrit/melbourne-housing/assets/156267785/af4bfc60-7933-4c14-b396-c7c6e0d7d2ac)

Based on the results, houses with 1 to 3 rooms were the most common, making up over 75.31% of the total houses sold between 2016-2017. Conversely, the number of houses sold decreased as the number of rooms increased. This pattern might be because the market availability tends to be opposite to the number of rooms in the houses.


## Visualising the findings
For this part of the project, I will not include the codes used for creating visualisations but the full codes can be accessed in the melbourne_housing_complete.R file under the same repository. 

### Most common type of houses based on sales 
![sales](https://github.com/ksadangrit/melbourne-housing/assets/156267785/cddc0ce3-6135-4150-8d71-10af4999a223)

### Based on total number
![total01](https://github.com/ksadangrit/melbourne-housing/assets/156267785/57cf89a9-4d73-4208-b5c0-827351570e33)

In Melbourne during 2016-2017, houses were the most commonly sold property type, making up about 77% of total sales and total number of properties sold. Apartments/units followed at 14%, and townhouses at 10%. The number of houses sold was nearly double that of apartments/units and townhouses combined.

### Based on sales separated by year
Now, we'll break down the total sales by year to observe any changes in the trend.

![type_sales_year](https://github.com/ksadangrit/melbourne-housing/assets/156267785/f727991d-7141-4675-99d1-4627bb01343b)

The trend persists even after separating the total sales by year, with houses remaining the most purchased property type, followed by apartments and townhouses. Additionally, sales across all property types are generally higher in 2017 compared to 2016.

### Total number sold each year
![total_year](https://github.com/ksadangrit/melbourne-housing/assets/156267785/d299cd3e-9c9f-440d-a44b-f9f5bff5a3a2)

When examining the total number of houses sold each year, this bar chart provides further clarity on why 2017 outperformed 2016 in terms of sales. With over 2000 more properties sold in 2017 compared to 2016, it's logical that the sales figures would also be higher.






