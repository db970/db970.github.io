---
title: "Final Project"
subtitle: " Data 333 "
author: |
  | Dov Bechhofer
date: "December 22nd, 2023"
output: 
  html_document:
    code_folding: hide
    toc: true
    toc_depth: 1
    toc_float: true
    theme: united
---




<style type="text/css">

h4.author { 
  font-size: 30px;
  font-family: Cursive;
  color: #424c52;
  text-align: center;
  font-weight: bold;
}


h4.date { 
  font-size: 24px;
  font-family: Times New Roman;
  color: #928971;
  text-align: center;
  font-weight: bold;


</style>

<style>

.list-group-item.active, .list-group-item.active:focus, .list-group-item.active:hover {
    background-color: #0096a7;
     text-align: center;
     font-weight: bold;
}

</style>

<style type="text/css">

  body{
  font-size: 14pt;
  font-family: Times New Roman;
  border: 3px solid black;
  border-radius: 25px;
  margin: 0 110px;
  max-width: 100%;
  
}

</style>
```{r setup, include = FALSE}

# R PACKAGES -------------------------------------------------------------------

if (!require('tidyverse')) 
  install.packages('tidyverse', 
                   repos = "http://cran.us.r-project.org"); 
library('tidyverse')

if (!require('tidycensus')) 
  install.packages('tidycensus', 
                   repos = "http://cran.us.r-project.org"); 
library('tidycensus')

if (!require('ggExtra')) 
  install.packages('ggExtra', 
                   repos = "http://cran.us.r-project.org"); 
library('ggExtra')

if (!require('kableExtra')) 
  install.packages('kableExtra', 
                   repos = "http://cran.us.r-project.org"); 
library('kableExtra')
# This is an 'extension' for ggplot2, or like an 'add-on'. I like to think of packages like this as 'expansion packs'. It will give us some new functionality for our plots!

# API KEY  --------------------------------------------------------------

census_api_key("f8db1f50b1f860424a98e31a90d124dcc8f908dd")

# CHUNK SETTINGS  --------------------------------------------------------------

knitr::opts_chunk$set(fig.width = 12, fig.height = 10)
options(scipen=999)
set.seed(123) 


acs_pums_variables = c("JWMNP",
                        "FINCP", 
                        "ESR",
                        "WKHP", 
                        "JWTRNS",
                        "RAC1P") 

acs_pums_data_nd <- get_pums(variables = acs_pums_variables, 
                     state = "ND",
                     year = "2019",
                     survey = "acs1")

acs_pums_data_sd <- get_pums(variables = acs_pums_variables, 
                     state = "SD",
                     year = "2019",
                     survey = "acs1")

acs_pums_data_project <- rbind(acs_pums_data_nd, 
                       acs_pums_data_sd)

rm(acs_pums_data_nd, acs_pums_data_sd, acs_pums_variables)


head(acs_pums_data_project)

tail(acs_pums_data_project)

summary(acs_pums_data_project)

colnames(acs_pums_data_project)

acs_pums_data_project_clean <- acs_pums_data_project%>%
  select("SERIALNO", "ST", "JWMNP", "FINCP", "ESR", "WKHP", "JWTRNS", "RAC1P")%>% 
  rename("ID" = SERIALNO, 
         "State" = ST, 
         "Travel_time_work" = JWMNP, 
         "Family_income" = FINCP, 
         "Employment_status" = ESR,
         "Hours_worked_week" = WKHP, 
         "Transportation_method" = JWTRNS, 
         "Race" = RAC1P)%>% 
  mutate(State = case_when(
      State == '38' ~ 'ND', 
      State == '46' ~ 'SD',
      TRUE ~ "Other"),
#---------------------------------- PROFESSOR ---------------------------------#
# These categories are probably going to be small and we can collapse them. Think
# 'private car', 'public transportation', 'walking/biking', etc.
#---------------------------------- PROFESSOR ---------------------------------#

     Transportation_categorical = case_when(
       Transportation_method %in% c("01", "07", "08") ~ "Private Car",
    Transportation_method %in% c("02", "03", "04", "05", "06", "12") ~ "Public Transportation",
    Transportation_method %in% c("09", "10") ~ "Walking/Biking",
    Transportation_method == "11" ~ "Worked from Home",
     Transportation_method == "12" ~ "Other method",
      Transportation_method == "bb" ~ as.character(NA),
      TRUE ~ "Other" 
    ),
    Employment_categorical = case_when(
      Employment_status == "b" ~ as.character(NA),
      Employment_status %in% c("1", "2") ~ "Civilian employed",
      Employment_status == "3" ~ "Unemployed",
      Employment_status %in% c("4", "5") ~ "Armed forces",
      Employment_status == "6" ~ "Not in labor force",
      TRUE ~ "Other"
    ),
    Race_categorical = case_when(
      Race == "1" ~ "White",
      Race == "2" ~ "Black",
      Race %in% c("3", "4") ~ "Indigenous", 
      Race %in% c("5", "7", "8") ~ "Other Pacific Islander or Some Other Race",  
      Race == "6" ~ "Asian",
      Race == "9" ~ "Multiracial",  
      TRUE ~ "Other"  
    )) %>%
  filter( Travel_time_work <= 40 &
          Travel_time_work != 'bbb' &
           Family_income <= quantile(Family_income, .95) & 
           Family_income >= quantile(Family_income, .05) & 
           Family_income != "bbbbbbb" &
           Hours_worked_week != 'bb')
#---------------------------------- PROFESSOR ---------------------------------#
# I think you may not want to exclude anyone who has 0 hours travel time, as that
# is what removes an overwhelming number of your data. Above 40 makes sense, 
# but you might want to keep those at 0 as well. 

    summary(acs_pums_data_project_clean$Family_income)
       boxplot(acs_pums_data_project_clean$Family_income)
     hist(acs_pums_data_project_clean$Family_income)
      summary(acs_pums_data_project_clean$Hours_worked_week)
      boxplot(acs_pums_data_project_clean$Hours_worked_week)
      summary(acs_pums_data_project_clean$Travel_time_work)
      boxplot(acs_pums_data_project_clean$Travel_time_work)
      hist(acs_pums_data_project_clean$Travel_time_work)
      
      acs_pums_data_project_clean <- acs_pums_data_project_clean %>%
        select(-Race, -Transportation_method, -Employment_status) %>%
      drop_na()
      
      100 - round((nrow(acs_pums_data_project_clean)/nrow(acs_pums_data_project)*100))

```

<center> <h1> ***Introduction***  </h1> </center>
  The topic of this study is the impact of travel time to work on family income and employment status in the Upper Midwest, specifically in North and South Dakota. This topic is intriguing as it explores how commuting affects economic and employment aspects of life, potentially highlighting regional disparities and infrastructure issues.
  The paper aims to analyze correlations between commute duration and socioeconomic factors, using the 2019 American Community Survey Public Use Microdata Sample (ACS PUMS). By examining continuous variables like travel time and family income, alongside categorical variables like employment status and race, the study will provide insights into the interplay between commuting patterns and socio-economic outcomes across different demographic groups. This comprehensive approach will help in understanding broader regional trends and individual experiences in the labor market.

<center> <h1> ***Hypotheses***  </h1> </center>
- There is a significant difference in average family income between civilian employed individuals and individuals in the armed forces..
  - This hypothesis is chosen because by using a t-test it directly compares the impact of employment status on family income.
- There is an association between employment status and the means of transportation used to get to work.  
  - This hypothesis is selected to explore potential association between transportation methods and employment status.
  
<center> <h1> ***Methods***  </h1> </center>
## ***Data and Sample:***
 The American Community Survey Public Use Microdata Sample (ACS PUMS) 2019. The Data collected by the U.S. Census Bureau, encompassing a range of socio-economic indicators. The sample focus on North Dakota and South Dakota.
 
## ***Measures:***
- Travel Time to Work (JWMNP)
- Family Income (FINCP)
- Employment Status (ESR)
- Usual Hours Worked Per Week (WKHP)
- Means of Transportation to Work (JWTRNS)
- Race (RAC1P)

## ***Analysis:***
- *Data Preparation* 
  - Filtering: Using `filter()` include relevant subsets, like specific states or removing extreme values.
  - Recoding: Using `mutate()` and` case_when()` to categorize variables like employment status into more meaningful groups for analysis.
  - Variable Selection: Using `select()` to Choose variables pertinent to the hypotheses.

- *Statistical Tests*
  - t-Test: To assess differences in mean travel time to work between employed and unemployed groups. This test will help understand if employment status significantly affects commuting time.
  - Chi-Square Test: Used to examine the relationship between transportation and employment status. This test will help determine if there is a significant association between these categorical variables.

- *Data Visualization* 
  - Create visual representations such as bar charts and histograms. These will be used to illustrate the distribution of variables like travel time, family income, and to compare these variables across different categories like race or employment status or compare one state to another.

<center> <h1> ***Overview***  </h1> </center>
Utilizing the tidycensus package, the 2019 ACS (American Community Survey) PUMS (Public Use Micro Sample) was obtained for both North Dakota and South Dakota. The variables selected from the data were the following: Family income (past 12 months, use ADJINC to adjust FINCP to constant dollars), Travel time to work, Means of transportation to work, Usual hours worked per week past 12 months, Employment status recode, Recoded detailed race code. With these variables and the data pulled from the ACS PUMS dataset the sample before cleaning was 8 variables, and a total of 17088 observations/respondents. North Dakota has 7,960 respondents and South Dakota has 9,128 respondents.
After re-coding was preformed for our categorical variables and continuous variable 17088, was reduced to 7,282 after removing the missing values and extreme outliers.

```{r ggplot styling, include=FALSE}
categorical_graphs_styling = list(
  # THEME OF CHOICE ----------------
  theme_classic(),
  # ADDITIONAL THEME ITEMS ----------------
  theme(
    title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
    ),
  # CLEAN LEGEND  ----------------
  guides(
    fill = guide_legend(
      override.aes = aes(label = "")
      )
    ),
  # COLOR SCALE  ----------------
  scale_fill_brewer(palette = "GnBu"), 
  # CAPTION  ----------------
  labs(caption = "Data from ACS PUMS")
)

ggplot(acs_pums_data_project_clean, aes(x = State, fill = State)) +
  geom_bar() +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Number of Respondents by State", caption = "Data from ACS PUMS", x = "State", y = "Count") +
  scale_fill_brewer(palette = "GnBu") 
```

```{r analysis, include=FALSE} 
acs_pums_data_project_clean %>%
  group_by(State) %>%
  summarize(
    Count = n(),
    Percentage = n() / nrow(acs_pums_data_project_clean) * 100)
    
    acs_pums_data_project_clean %>%
  group_by(Transportation_categorical) %>%
  summarize(
    Count = n(),
    Percentage = n()/ nrow(acs_pums_data_project_clean) * 100)
    
    
  acs_pums_data_project_clean %>%
  group_by(Employment_categorical) %>%
  summarize(
    Count = n(),
    Percentage = n()/ nrow(acs_pums_data_project_clean) * 100)
  
  	acs_pums_data_project_clean %>%
  group_by(Race_categorical) %>%
  summarize(
    Count = n(),
    Percentage = n()/ nrow(acs_pums_data_project_clean) * 100)
  	
  	acs_pums_data_project_clean %>%
  summarise(mean = mean(Travel_time_work), 
            sd = sd(Travel_time_work), 
            median = median(Travel_time_work), 
            IQR = IQR(Travel_time_work),
            min = min(Travel_time_work), 
            max = max(Travel_time_work)) 
  	
  	acs_pums_data_project_clean %>%
  summarise(mean = mean(Family_income), 
            sd = sd(Family_income), 
            median = median(Family_income), 
            IQR = IQR(Family_income),
            min = min(Family_income), 
            max = max(Family_income)) 
  	
  	acs_pums_data_project_clean %>%
  summarise(mean = mean(Hours_worked_week), 
            sd = sd(Hours_worked_week), 
            median = median(Hours_worked_week), 
            IQR = IQR(Hours_worked_week),
            min = min(Hours_worked_week), 
            max = max(Hours_worked_week)) 
  	
  	prop_table_race_employment <- table(acs_pums_data_project_clean$Race_categorical, acs_pums_data_project_clean$Employment_categorical)
prop_table_race_employment_prop <- prop.table(prop_table_race_employment, margin = 2)
prop_table_race_employment_percent <- prop_table_race_employment_prop * 100
rounded_percent_table <- round(prop_table_race_employment_percent, 2)
print(rounded_percent_table)
print(prop_table_race_employment_prop)


t_test_result <- t.test(acs_pums_data_project_clean$Travel_time_work, acs_pums_data_project_clean$Family_income)
print(t_test_result)
  

chi_square_result <- chisq.test(acs_pums_data_project_clean$Race_categorical, acs_pums_data_project_clean$Employment_categorical)
print(chi_square_result)

#---------------------------------- PROFESSOR ---------------------------------#
# Good job with the analysis. Just be sure to round off the %, as I see you're 
# thinking about using kable extra and if you want to show your prop.tables 
# in the final you'll want to have them rounded off. Also you analysis numbers
# may change if you collapse/re code slightly as suggested above. 
#---------------------------------- PROFESSOR ---------------------------------#

```
<center> <h1> ***Data Analysis***  </h1> </center>
In this data, the state of ND makes up 46.3% of the data or 3,369 respondents. The state of SD makes up 53.7% of the data or 3,913 respondents.

# ***Univariate Analysis*** {.tabset .tabset-fade .tabset-pills}


## *State*
```{r}
ggplot(acs_pums_data_project_clean, aes(x = State, fill = State)) +
  geom_bar() +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Number of Respondents by State", caption = "Data from ACS PUMS", x = "State", y = "Count") +
  scale_fill_brewer(palette = "GnBu") 
```

``` {r}

acs_pums_data_project_clean %>%
  group_by(State) %>%
  summarize(
    Count = n(),
    Percentage = round((n() / nrow(acs_pums_data_project_clean) * 100), 2)
  ) %>%
  kable("html", caption = "State-wise Summary") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```

## *Transportation method*

```{r}
ggplot(acs_pums_data_project_clean, aes(x = Transportation_categorical, fill = Transportation_categorical)) +
  geom_bar() +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Distribution of Transportation Methods", caption = "Data from ACS PUMS", y = "Transportation Method", x = "Count") +
  scale_fill_brewer(palette = "GnBu")

```

```{r}
acs_pums_data_project_clean %>%
  group_by(Transportation_categorical) %>%
  summarize(
    Count = n(),
    Percentage = round((n() / nrow(acs_pums_data_project_clean) * 100), 2)
  ) %>%
  kable("html", caption = "Transportation method Summary") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```



## *Employment status*
```{r}
ggplot(acs_pums_data_project_clean, aes(x = Employment_categorical, fill = Employment_categorical)) +
  geom_bar() +
  theme_classic() +
  coord_flip() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Employment Distribution", caption = "Data from ACS PUMS", y = "Count", x = "Employment Status") +
  scale_fill_brewer(palette = "GnBu")
```

``` {r}
acs_pums_data_project_clean %>%
  group_by(Employment_categorical) %>%
  summarize(
    Count = n(),
    Percentage = round((n() / nrow(acs_pums_data_project_clean) * 100), 2)
  ) %>%
  kable("html", caption = "Transportation Status Summary") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```



## *Race*
```{r}
ggplot(acs_pums_data_project_clean, aes(x = Race_categorical, fill = Race_categorical)) +
  geom_bar() +
  theme_classic() +
  coord_flip() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Race Distribution", caption = "Data from ACS PUMS", y = "Count", x = "Race") +
  scale_fill_brewer(palette = "GnBu")
```

``` {r}
acs_pums_data_project_clean %>%
  group_by(Race_categorical) %>%
  summarize(
    Count = n(),
    Percentage = round((n() / nrow(acs_pums_data_project_clean) * 100), 2)
  ) %>%
  kable("html", caption = "Race Summary") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```
   

## *Travel Time to Work*

```{r}
ggplot(acs_pums_data_project_clean, aes(x = Travel_time_work, fill = after_stat(count))) +
  geom_histogram(binwidth = 5) +
  scale_fill_gradientn(colors = RColorBrewer::brewer.pal(9, "GnBu")) +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Distribution of Travel Time to Work", caption = "Data from ACS PUMS", x = "Travel Time to Work (minutes)", y = "Count")

#Source CHATGPT[Error: Continuous value supplied to discrete scale]
```

```{r}
acs_pums_data_project_clean %>%
  summarise(
    mean = mean(Travel_time_work, na.rm = TRUE), 
    sd = sd(Travel_time_work, na.rm = TRUE), 
    median = median(Travel_time_work, na.rm = TRUE), 
    IQR = IQR(Travel_time_work, na.rm = TRUE),
    min = min(Travel_time_work, na.rm = TRUE), 
    max = max(Travel_time_work, na.rm = TRUE)
  ) %>%
  kable("html", caption = "Summary Statistics for Travel Time to Work") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))

```
The basic summary statistics for Travel time to work are the following: A min of 0. A max of 40. A mean of 13.3 A standard deviation of +/- 10.2. A median of 10 And an IQR of 15.

## *Family Income*
```{r}
ggplot(acs_pums_data_project_clean, aes(x = State, y = Family_income, fill = State)) +
  geom_boxplot() +
  scale_fill_brewer(palette = "GnBu") +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Comparison of Family Income between SD and ND", caption = "Data from ACS PUMS", x = "State", y = "Family Income")

```

```{r}
acs_pums_data_project_clean %>%
  summarise(
    mean = mean(Family_income, na.rm = TRUE), 
    sd = sd(Family_income, na.rm = TRUE), 
    median = median(Family_income, na.rm = TRUE), 
    IQR = IQR(Family_income, na.rm = TRUE),
    min = min(Family_income, na.rm = TRUE), 
    max = max(Family_income, na.rm = TRUE)
  ) %>%
  kable("html", caption = "Summary Statistics for Family Income") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```
The basic summary statistics for Family income  are the following: A min of -60,000. A max of 215,000. A mean of 57,577.  A standard deviation of +/- 76,112. A median of 72,000. And an IQR of 95,500

## *Hours Worked per Week*
```{r}
ggplot(acs_pums_data_project_clean, aes(x = Hours_worked_week, fill = after_stat(count))) +
  geom_histogram(binwidth = 30) +  # Adjust the binwidth as needed
  scale_fill_gradientn(colors = RColorBrewer::brewer.pal(9, "GnBu")) +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Distribution of Hours Worked Per Week", caption = "Data from ACS PUMS", x = "Hours Worked Per Week", y = "Count")


```

```{r}
acs_pums_data_project_clean %>%
  summarise(
    mean = mean(Hours_worked_week, na.rm = TRUE), 
    sd = sd(Hours_worked_week, na.rm = TRUE), 
    median = median(Hours_worked_week, na.rm = TRUE), 
    IQR = IQR(Hours_worked_week, na.rm = TRUE),
    min = min(Hours_worked_week, na.rm = TRUE), 
    max = max(Hours_worked_week, na.rm = TRUE)
  ) %>%
  kable("html", caption = "Summary Statistics for Hours Worked Per Week") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```

The basic summary statistics for Usual hours worked per week past 12 months are the following: A min of 1. A max of 99. A mean of 40.3. A standard deviation of +/- 14.3. A median of 40, and an IQR of 9	

## {-}






# ***Civilian vs Armed Forces*** {.tabset .tabset-fade .tabset-pills}





## *Table*
```{r}
income_summary <- acs_pums_data_project_clean %>%
  group_by(Employment_categorical) %>%
  summarize(Average_Income = mean(Family_income, na.rm = TRUE)) %>%
  filter(Employment_categorical %in% c("Civilian employed", "Armed forces"))

kable(income_summary, format = "html", col.names = c("Employment Category", "Average Income"), caption = "Average Income by Employment Category")
```


***Armed Forces***: The average income for individuals in the armed forces is $3,076.21. This figure can encompass a range of ranks and roles within the military.

***Civilian Employed***: For those employed in civilian roles, the average income is significantly higher, at $58,014.63. This category likely includes a wide array of professions and industries, contributing to the higher average income.

It's important to note that these figures can vary widely based on factors such as experience, education, and specific job roles within each category. Additionally, after removing NA's armed forces only accounts for 0.8% of our data

## *Visualization*
```{r vis}

ggplot(acs_pums_data_project_clean, aes(x = Employment_categorical, y = Family_income, fill = Employment_categorical)) +
  geom_boxplot() +
  scale_fill_brewer(palette = "GnBu") +
  theme_classic() +
  theme(plot.title = element_text(face = "bold"), 
        axis.title = element_text(face = "bold"), 
        axis.text = element_text(face = "bold")) +
  labs(title = "Income Distribution: Civilian Employed vs Armed Forces", caption = "Data from ACS PUMS",
       x = "Employment Status",
       y = "Income")

#"[uploaded an image]provide interpretation" prompt. ChatGPT, OpenAI, 18 Dec. 2023. https://chat.openai.com.
```
For "Armed forces," the boxplot shows no variation or range, implying that all data points might have the same value or very little variation around the average income provided. This results in a box without any visible whiskers or variability, which is not typical for a boxplot (probably due to small sample of armed forces).

For "Civilian employed," there's a noticeable range shown in the blue boxplot. This suggests there's variability in the income data for this group. The box represents the interquartile range (IQR), where the middle 50% of data points lie, with the line in the box likely representing the median income. The "whiskers" extending from the box suggest the range of the rest of the data, except potential outliers which are not shown here.

```{r}
ggplot(acs_pums_data_project_clean, aes(x = Employment_categorical, y = Hours_worked_week, fill = Employment_categorical)) +
  geom_boxplot() +
  scale_fill_brewer(palette = "GnBu") +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Comparison of Hours Worked Per Week by Employment Category", caption = "Data from ACS PUMS", x = "Employment Category", y = "Hours Worked Per Week")


#"[uploaded an image]provide interpretation" prompt. ChatGPT, OpenAI, 18 Dec. 2023. https://chat.openai.com.
```
The civilian employed category has a greater variance in the number of hours worked per week than the armed forces, with many more outliers, especially on the higher end. The armed forces appear to have a more consistent workweek, with fewer outliers.

***Armed Forces***: The boxplot for the armed forces shows a more compact interquartile range (IQR), which is the range between the first quartile (25th percentile) and the third quartile (75th percentile), as indicated by the box's height. The median, represented by the line within the box, appears to be around the 50-hour mark. There are a few outliers indicated by the individual dots above the upper whisker, which stretches just above the 75-hour mark. This suggests that while most armed forces personnel work around a median of 50 hours, there are some who work significantly more.

***Civilian Employed***: The civilian employed boxplot shows a wider IQR, suggesting more variation in the number of hours worked per week within this group. The median is lower compared to the armed forces, situated just above the 40-hour mark, which is typically considered a standard workweek. The upper whisker extends to around the 60-hour mark, indicating that some civilians work up to 60 hours. Notably, there are numerous outliers, depicted as dots, both below and above the main box. The outliers above show that a significant number of civilian employees work more than what the upper whisker indicates (over 60 hours), with some approaching 100 hours per week.

## *T-test*
```{r}
civilian_employed_income <- acs_pums_data_project_clean %>%
  filter(Employment_categorical == "Civilian employed") %>%
  pull(Family_income)

armed_forces_income <- acs_pums_data_project_clean %>%
  filter(Employment_categorical == "Armed forces") %>%
  pull(Family_income)

t_test_result <- t.test(civilian_employed_income, armed_forces_income)
print(t_test_result)





#The Welch Two Sample t-test results indicate a statistically significant difference in average family income between #civilian employed individuals and those in the armed forces:

#t-Value (5.6953): Indicates a substantial difference between the means of the two groups.
#Degrees of Freedom (57.992): Suggests the sample size and variance of the two groups.
#-Value (0.000000431): Extremely small, indicating the difference in average incomes is statistically significant.
#95% Confidence Interval (35629.36 to 74247.48): The difference in mean family income lies within this range, not including 0, reinforcing a significant difference.
#Mean Estimates: The average family income for civilian employed (58014.628) is significantly higher than for armed forces (3076.207).
#In summary, civilian employed individuals have a significantly higher average family income compared to those in the armed forces.
```
The Welch Two Sample t-test results indicate a statistically significant difference in average family income between civilian employed individuals and those in the armed forces:

t-Value (5.6953): Indicates a substantial difference between the means of the two groups.
Degrees of Freedom (57.992): Suggests the sample size and variance of the two groups.
p-Value (0.000000431): Extremely small, indicating the difference in average incomes is statistically significant.
95% Confidence Interval (35629.36 to 74247.48): The difference in mean family income lies within this range, not including 0, reinforcing a significant difference.
Mean Estimates: The average family income for civilian employed (58014.628) is significantly higher than for armed forces (3076.207).
In summary, civilian employed individuals have a significantly higher average family income compared to those in the armed forces.

## {-}



# ***Transportation vs Employment Status*** {.tabset .tabset-fade .tabset-pills}

## *Table*
```{r race-employment-table, echo=FALSE}
cross_tab <- table(acs_pums_data_project_clean$Employment_categorical, acs_pums_data_project_clean$Transportation_categorical)


kable(cross_tab, caption = "Cross-tabulation of Employment Status and Means of Transportation") %>%
  kable_styling(bootstrap_options = c("striped", "hover"))
```
***Private Car***:

- A large majority of both Armed Forces and Civilian Employed individuals use private cars to get to work. Notably, 56 Armed Forces individuals use private cars, which is a 100% utilization rate if we consider only the modes listed here.
- For Civilian Employed, 6260 use private cars, which is the dominant mode of transportation for this group as well.

***Public Transportation***:

- None of the Armed Forces individuals in this sample use public transportation.
- A small number (72) of Civilian Employed individuals use public transportation, suggesting that it's a less preferred mode among those who are employed in civilian roles, at least within this dataset.

***Walking/Biking***:

- Walking or biking is slightly more common among Armed Forces individuals (2) compared to public transportation but still not a significant number.
- For Civilian Employed, 374 individuals walk or bike to work, indicating that a higher, yet still small, proportion of civilians use these modes of transport compared to Armed Forces personnel.

***Worked from Home***:

- No Armed Forces individuals reported working from home, which may reflect the nature of Armed Forces jobs, requiring a physical presence.
- In contrast, 518 Civilian Employed individuals work from home, reflecting a growing trend or possibility within civilian employment to have remote work options.



It should be noted the under representation of armed forces individuals in the data sample.


## *Visualization* 
```{r}
ggplot(acs_pums_data_project_clean, aes(x = Employment_categorical, fill = Transportation_categorical)) +
  geom_bar(position = "fill") +
  scale_fill_brewer(palette = "GnBu") +
  theme_classic() +
  theme(
    plot.title = element_text(face = "bold"), 
    axis.title = element_text(face = "bold"), 
    axis.text = element_text(face = "bold")
  ) +
  labs(title = "Proportion of Means of Transportation by Employment Status",
       x = "Employment Status",
       y = "Proportion")

```


## *Chi-square*
```{r chi-square, message = FALSE, warning = FALSE}
chi_square_result1 <- chisq.test(acs_pums_data_project_clean$Employment_categorical, acs_pums_data_project_clean$Transportation_categorical)


print(chi_square_result1)



#"what the interpretation of this and to what extent does small sample of armed forces play" prompt. ChatGPT, OpenAI, 20 Dec. #2023. https://chat.openai.com.


```

Given the p-value is greater than 0.05, we would typically conclude that there is no statistically significant association between employment status and means of transportation to work at the conventional 5% level of significance. In other words, any observed association in the sample data could likely be due to random chance.

*Limitation*

Statistical Power: A small sample size can lead to a lack of statistical power, which means the test might not be sensitive enough to detect an actual association even if one exists.

Expected Cell Counts: Chi-squared tests assume that the expected frequency counts for each cell in the contingency table are sufficiently large (usually at least 5). If many cells have counts less than 5, which might be the case with a small Armed Forces sample, the test may not be valid.

Effect Size: A small sample could mask a potentially large effect size. Even if the proportion of Armed Forces using different transportation means is very different from that of Civilian Employed, the small number of Armed Forces could result in a non-significant p-value.

Overestimation of the P-value: With small expected counts, the chi-squared test can overestimate the p-value, leading to a Type II error (failing to reject the null hypothesis when it is false).


## {-}


# ***Conclusion***

This research aimed to explore the impact of employment status on family income and the choice of transportation method to work. The study was designed with two primary hypotheses: one positing a significant difference in average family income between civilian employed individuals and those in the armed forces, and another suggesting an association between employment status and the chosen means of transportation to work.

***Findings***:

- The income analysis revealed a stark difference between the two employment statuses. Civilian employees had a substantially higher average income than their armed forces counterparts. This finding was supported by a Welch Two Sample t-test, which indicated a statistically significant difference between the groups.

- In terms of transportation, the majority of individuals in both employment categories predominantly used private cars to commute. Public transportation was less favored, and working from home was notably more common among civilian employees, likely reflecting the flexibility of civilian roles and the need for physical presence in military jobs. However, the chi-square test did not find a statistically significant association between employment status and transportation method.

***Limitations***:

- The study's statistical power was potentially compromised by the small sample size of armed forces personnel, representing only 0.8% of the data. This small representation could hinder the detection of a true association in the data.

- Chi-square test validity might be questioned due to low expected cell counts, specifically for the armed forces category, leading to possible Type II errors.

- The effect size of the differences in transportation methods could be significant, but the small sample size for armed forces personnel might have diluted the apparent association.

***Implications and Future Research***:


- Future studies could benefit from a larger and more balanced sample size, especially increasing the representation of armed forces personnel to enhance the reliability of statistical tests.

- Subsequent research might also delve into qualitative aspects, such as job satisfaction, quality of life, and the social implications of the observed income disparities.

- Investigating the intersectionality of employment status with other socio-demographic factors (like age, gender, and urban vs. rural residency) could provide a more nuanced understanding of the dynamics at play.



##### References
[kableExtra](https://cran.r-project.org/web/packages/kableExtra/vignettes/awesome_table_in_html.html#Overview)

[R Markdown Theme](https://rpubs.com/ranydc/rmarkdown_themes#:~:text=To%20apply%20a%20theme%20to,your%20document%2C%20including%20the%20theme)

[HTML Fonts](https://www.hostinger.com/tutorials/best-html-web-fonts)

[SD Census](https://www.census.gov/quickfacts/fact/table/SD/LFE305222#LFE305222)

[ND Census](https://www.census.gov/quickfacts/fact/table/ND/PST045222)

[GGPLOT](https://ggplot2.tidyverse.org/reference/element.html)
