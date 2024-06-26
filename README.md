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
1. Importing data
2. Cleaning the data
3. Analysis
4. Visualisation
5. Key findings and recommendations

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


## Cleaning data
We'll start the cleaning step by checking how many rows there are in this dataset using the following code.
```
nrow(melbourne_housing)
```
There are 18396 rows in total.
```
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
![Screen Shot 2024-03-27 at 11 46 43 AM](https://github.com/ksadangrit/melbourne-housing/assets/156267785/c8221215-4c7a-4850-ae9b-d4663fd8d0b3)

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

Next, we'll extract the month and year from the date column and deselect any unnecessary columns, as well as those with over 18% of missing data. Since the missing data for each house is unique and cannot be inferred, analysis involving columns with missing data may lead to biased and inaccurate findings. Additionally, we'll rename the `seller_g` column to `seller` for improved readability. Finally, we'll name the cleaned dataset as `melbourne_housing_clean`.

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
    -postcode,
    -car,
    -landsize,
    -building_area,
    -council_area,
    -year_built,
    -bedroom2,
    -bathroom,
    -lattitude,
    -longtitude
      ) %>% 
  rename(seller = seller_g) 
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

#### Total sales and number of properties sold
```
sales_year <- melbourne_housing_clean %>% 
  group_by(year) %>% 
  summarise(number_sold = n(),
            total_sales = sum(price))
```

In the previous part, we found the average and middle prices for each property type. Now, we'll split the data by year and compare how the average and middle prices differ. This comparison will help us understand how the data is spread out. We'll use this information for our next step in visualizing the data.

#### Average and Median price
```
sum_per_year <- melbourne_housing_clean %>% 
  group_by(type, year) %>% 
  summarise(median_price = median(price),
            avg_price = mean(price)) 
```

### Monthly
We'll check out the total number of property sales and the number of properties sold in a monthly basis.

Instead of calculating the median house price for each month, we'll compare the **total sales** for each month. Since months represent a shorter time scale compared to years, variations in median house prices for each month may be influenced by factors such as housing market availability. Therefore, focusing on total sales can offer valuable insights into fluctuations in housing demand throughout the year.

#### Total sales and number of properties sold
```
sales_month <- melbourne_housing_clean %>% 
  group_by(month) %>% 
  summarise(number_sold = n(),
            total_sales = sum(price)) 
```

#### Total sales separated by types
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

### Yearly
#### Total sales each year and total number sold
| Total sales                           | Number of properties sold                       |
| ------------------------------------- | ----------------------------------------------- |
| ![sales_year](https://github.com/ksadangrit/melbourne-housing/assets/156267785/3f2fffb9-50dc-4773-9277-b45cb38af794)| ![total_year](https://github.com/ksadangrit/melbourne-housing/assets/156267785/d299cd3e-9c9f-440d-a44b-f9f5bff5a3a2) |

In 2017, sales outperformed 2016 by over 2 billion dollars. When examining the total number of houses sold each year, this bar chart provides further clarity on why 2017 outperformed 2016 in terms of sales. With over 2000 more properties sold in 2017 compared to 2016, it's logical that the sales figures would also be higher.

### Yearly and type
#### Most common type of houses based on sales and total number
| Total sales                           | Number of properties sold                       |
| ------------------------------------- | ----------------------------------------------- |
| ![sales](https://github.com/ksadangrit/melbourne-housing/assets/156267785/ae3b4ddb-0c19-4939-bf2f-01c4d4016674)| ![total](https://github.com/ksadangrit/melbourne-housing/assets/156267785/04ea5402-8fcf-4639-8135-fa2d8ff1bc65) |


In Melbourne during 2016-2017, houses were the most commonly sold property type, making up about 77% of total sales and total number of properties sold. Apartments/units followed at 14%, and townhouses at 10%. The number of houses sold was nearly double that of apartments/units and townhouses combined.

#### Based on sales separated by year
Now, we'll break down the total sales by year to observe any changes in the trend.

![type_sales_year](https://github.com/ksadangrit/melbourne-housing/assets/156267785/f727991d-7141-4675-99d1-4627bb01343b)

The trend persists even after separating the total sales by year, with houses remaining the most purchased property type, followed by apartments and townhouses. Additionally, sales across all property types are generally higher in 2017 compared to 2016. This can be attributed to the notable increase in the number of properties sold in 2017 compared to 2016.

### Median vs Average house price 
| Median Price by Year                           | Average Price by Year                         |
| --------------------------------------------- | --------------------------------------------- |
| ![Median Price](https://github.com/ksadangrit/melbourne-housing/assets/156267785/b562ce9a-a575-4d80-a342-cd746063ad9d) | ![Average Price](https://github.com/ksadangrit/melbourne-housing/assets/156267785/d5cede3b-704a-442e-8202-19626b7aa816) |

* Average prices for each property type exceeded the respective median prices in both years.
* Houses consistently exhibited the highest average and median prices, followed by townhouses and apartment/units.
* Average prices were generally higher in 2016 compared to 2017 across all property types.
* The difference between average and median values was largest for houses, followed by townhouses and apartment/units.

### Monthly
#### Sales by month
![sales_month](https://github.com/ksadangrit/melbourne-housing/assets/156267785/58c07d3e-73d8-440d-9ab3-7d5b69a3da25)

The sales of houses showed a consistent pattern of being generally high from April to September. Total sales across all properties reached their lowest point in January. From January onwards, sales gradually increased, with the highest sales volume occurring in September, totaling around $3.4 billion. However, there was a significant drop in sales between September and October, with a difference of almost $2 billion.

#### Sales and Total number seperated by type and month

Now, we'll compare the total number of houses sold and the total sales for each month of the year, categorized by property type.
![sales_month_type](https://github.com/ksadangrit/melbourne-housing/assets/156267785/4caacc1f-c6b7-496b-8f5f-b09a35b2b0ac)

![total_month_type](https://github.com/ksadangrit/melbourne-housing/assets/156267785/6b998b9e-3407-416a-931d-ca2adade56ad)

* The graphs illustrate a consistent trend across all three property types, with houses leading in both sales numbers and total properties sold, followed by apartments and townhouses.
* This trend reflects the pattern observed in the total sales from all property types.
* September stands out as the month with the highest sales and total number of properties sold across all property types.
* However, houses show more pronounced fluctuations in sales, with distinct peaks and troughs, while apartments and townhouses exhibit similar but less dramatic changes over time.

### Suburbs
In this section, we'll identify and rank the top 10 different types of properties based on their average prices.

#### Top 10 suburbs for House
![top_suburb_house](https://github.com/ksadangrit/melbourne-housing/assets/156267785/e6ed559b-5226-4b73-92e5-ec8ab192f522)

#### Top 10 suburbs for Apartment/ Unit
![top_council_apartment](https://github.com/ksadangrit/melbourne-housing/assets/156267785/328a97a0-eb63-41b2-9f3d-6e21399520d8)

#### Top 10 suburbs for Townhouse
![top_council_townhouse](https://github.com/ksadangrit/melbourne-housing/assets/156267785/7e069112-d219-431c-997e-659fde467afc)

* None of the suburbs listed for houses appear on the top 10 list for apartments or townhouses. Notably, even the lowest average house price among these suburbs exceeds the highest average price for other property types.
* Bayside ranks first for both apartments and townhouses. Furthermore, all suburbs listed for apartments also appear on the townhouse list.
* Manningham, Melbourne, Whitehorse, and Monash rank higher for apartments but lower for townhouses. Conversely, suburbs ranking higher for townhouses include Stonington, Boroondara, Port Phillip, Yarra, and Glen Eira. Additionally, the average apartment prices across all suburbs are generally lower than the average prices for townhouses in the same suburbs.

### Regions 
#### Sales by region
![sales_region](https://github.com/ksadangrit/melbourne-housing/assets/156267785/8e0e1391-c4be-4d6e-944f-5a84cfecf1c9)

* Metropolitan regions generally have large total sales, with Southern Metropolitan topping the list at over $8.6 billion, followed by Northern Metropolitan and Western Metropolitan.
* The Victoria region had the lowest sales overall, with Western Victoria recording the lowest sales of around $16.5 million.
* Among the Metropolitan regions, South-Eastern Metropolitan was the only one with less than $1 billion in sales.

#### Average house price by region
Now, let's examine the average house prices for different regions based on property types.
![avg_region](https://github.com/ksadangrit/melbourne-housing/assets/156267785/b2f962f1-1191-4101-a9f0-4ee9c1e01824)

* No sales records are available in our data for townhouses and apartments in Northern Victoria and Western Victoria.
* Average house prices exceed those of townhouses and apartments in every region.
* Southern Metropolitan continues to have the highest sales across all property types.
* Interestingly, Eastern outperforms Northern Metropolitan to become the region with the second-highest average property prices.
* There is a more significant disparity in average house prices across different regions compared to apartments and townhouses, as evidenced by Southern Metropolitan having an average house price more than three times higher than that of Western Victoria.

### Sellers
We'll begin by ranking the top 5 sellers across all types of properties based on sales and the number of properties sold to explore their relationships and differences.
| Total sales                           | Number of properties sold                       |
| --------------------------------------------- | --------------------------------------------- |
| ![sales_seller](https://github.com/ksadangrit/melbourne-housing/assets/156267785/f42357c9-c751-4e01-b3af-cca7e3489f64)| ![total_seller](https://github.com/ksadangrit/melbourne-housing/assets/156267785/c6a3f797-f7ca-4057-931d-c5a13d0fbc1e)|

* Jellis, Nelson, Hockingstuart, and Barry appear in the top 5 list for both criteria.
* Jellis ranks highest in sales but is 2nd in total number of properties sold, whereas Nelson has sold more houses but ranks 2nd in sales.
* Marshall ranks 3rd for sales but is not in the top 5 for total number of properties sold.
* Hockingstuart and Barry rank 1 position lower in sales compared to their ranks in total number of properties sold.
* Ray is not in the top 5 list for sales but ranks 5th for total number of properties sold.
* The differences between each rank in the top 5 are consistent, with no drastic variations observed.

Next, we'll examine the top 5 sellers with the highest sales for different types of properties to determine if any sellers dominate the Melbourne housing market or specialize in certain types of properties.

#### Top 5 sellers for House
![seller_house](https://github.com/ksadangrit/melbourne-housing/assets/156267785/406b73e5-0440-487c-a367-3f65001b8b43)

#### Top 5 Sellers for Apartment/ Unit
![seller_apartment](https://github.com/ksadangrit/melbourne-housing/assets/156267785/9e4252ff-24de-40ad-b2d3-435a56035207)

#### Top 5 sellers for Townhouse
![seller_townhouse](https://github.com/ksadangrit/melbourne-housing/assets/156267785/fdca9aea-87ed-45e5-a72a-bacadd74f516)

* The top 5 sellers for houses closely resemble the list of top 5 sellers for total sales we previously discussed, indicating that house sales significantly contribute to the overall ranking.
* Surprisingly, Hockingstuart surpassed Jellis and Nelson to rank 1 for apartments, with Buxton also making it to the top 5 sellers for apartments.
* Jellis ranks 1st for townhouses, followed by Buxton and Nelson.
* Barry, Buxton, and Marshall are among the top 5 sellers for 2 out of 3 types of properties, while Jellis, Nelson, and Hockingstuart are the top sellers for all types.
* The total sales from the top-ranked seller for apartments and townhouses are still lower than the total sales made by the rank 5 seller for houses.

### Method
| Total sales                           | Number of properties sold                       |
| --------------------------------------------- | --------------------------------------------- |
| ![method_sales](https://github.com/ksadangrit/melbourne-housing/assets/156267785/71be72dc-fe3f-44ea-9c83-6c2da4a755ee) | ![method_total](https://github.com/ksadangrit/melbourne-housing/assets/156267785/0ed4aa86-6f5e-4dff-b7b5-c787e07ffde7)|

* Over 65% of Melbourne houses were sold without using the auction method.
* The number of properties that were passed in and sold afterward was slightly lower than the properties that were sold prior to the auction. However, this is the opposite for the total sales.
* Less than 9% of all properties were sold at auction.
* Less than 1% were sold after the auction took place and contributed to the total sales.

We'll segregate the total sales by property type to determine the most popular method among different types of properties.

#### Total sales by type and selling method
![method_sales_type](https://github.com/ksadangrit/melbourne-housing/assets/156267785/90f34d68-8998-4b54-b52d-d22877a013ee)

* Even when separated by property type, common methods' rankings still mirror the overall sales trends.
* The most popular method across all property types is non-auction sales, with significantly higher sales compared to other methods.
* Selling after auctions is the least common method across all types.

Next, we'll compare house prices with other factors to identify any trends or relationships between them

### Price vs Distance
![price vs distance](https://github.com/ksadangrit/melbourne-housing/assets/156267785/c23938d5-d290-4802-8145-d8414b11f874)

* The graph shows a clustering of data where properties within 1-15km of the CBD are priced under $2 million.
* As the distance from the CBD increases, the data becomes less clustered, suggesting a preference for properties closer to the city.
* Properties sold for over $5 million are mostly within 15km of the CBD, except for one priced over $8 million located further away.
* The graph suggests that proximity to the CBD influences buyer decisions, with more sales observed for houses closer to the city. This may be due to buyers' willingness to pay higher prices, resulting in increased property values for homes near the CBD.

### Price vs Number of Rooms
![price vs room](https://github.com/ksadangrit/melbourne-housing/assets/156267785/7d7d4a8d-d2c9-45fd-9725-68b65380235d)

* Most houses have 1-5 bedrooms.
* The cheapest house has 1 bedroom, while the most expensive has 3 bedrooms.
* A few houses with over 7 bedrooms are priced under 4 million.
* There isn't a clear relationship between the number of bedrooms and property prices. Despite houses worth over $5 million typically having 3-6 bedrooms, there are still houses with over 5 bedrooms priced below $5 million.

### Property count vs Total number 
Now, we'll explore whether there is a positive relationship between the number of properties existing in suburbs and the number of properties sold. Each dot on the graph represents a single suburb.
![count vs total](https://github.com/ksadangrit/melbourne-housing/assets/156267785/a08ff764-8b09-4d2e-917b-8e2343aed6ad)

* Most suburbs have fewer than 7000 properties and less than 200 properties sold.
* Suburbs with over 10,000 existing properties show varying numbers of properties sold, ranging from 1 to over 400.
* One suburb stands out with over 20,000 properties and over 500 properties sold, suggesting a potential positive correlation.
* However, this correlation is not universally applicable across all suburbs upon closer examination of the overall data.

## Key Findings and Insights
* There are a few properties with large prices affecting the overall median of house prices.
* The average in 2016 is around $1.1 million, 7% higher than 2017, but 16% lower than the median house price of the same year.
* Average apartment prices for both years are under $870k, with higher prices in 2017.
* For Townhouses, it's cheaper to buy in 2016, with an average price of $546k, 7% cheaper than in 2017.
* Houses rank first for total sales and total number sold, followed by apartments and townhouses, consistently across all years.
* Despite higher average and median prices for townhouses, more apartments were sold, possibly due to the availability of different property types on the market.
* Sales and properties sold were higher in 2017 compared to 2016.
* Most houses (75%) consist of 1-3 bedrooms, with fewer houses having higher room numbers.
* May to September sees higher sales volume, with September having the highest sales across all types, while January has the lowest.
* Toorak ranks as the top suburb for houses. None of the top suburbs on the houses list appear in the top 10 for apartments or townhouses. Bayside ranks first for both apartments and townhouses.
* Southern Metropolis consistently has the highest average prices for all property types, while Eastern Victoria is the most affordable across the board.
* Jellis and Nelson consistently rank as the top two sellers across all property types, except for townhouses, where Buxton surpasses Nelson to rank second.
* Most houses were sold without involving auctions.
* Buyers prefer houses closer to the city, as evidenced by higher sales volumes and prices.
* There appears to be no clear relationship between house prices and the number of rooms, or between the number of properties sold and the existing properties in each suburb.

## Recommendations
* Investors should prioritize location-based strategies. Southern Metropolis offers higher prices for potential higher returns, while Eastern Victoria provides affordability. Proximity to city centers should also be considered for potential demand and appreciation, benefiting buyers and investors.
* Buyers should strategically time purchases, particularly for townhouses, aiming for lower prices during downturns like in 2016 compared to 2017.
* To gain a comprehensive understanding and better assess market dynamics, it'd be helpful to know the number of houses listed for sale. This data provides insights into demand trends, transaction flow, and overall market activity in Melbourne, aiding in informed decision-making for buyers, sellers, and investors alike.


