---
title: "AB Testing Project"
knit: (function(input_file, encoding) {
  out_dir <- 'docs';
  rmarkdown::render(input_file,
 encoding=encoding,
 output_file=file.path(dirname(input_file), out_dir, 'index.html'))})
subtitle: "WNW streaming company"
author: "Mohammad R. Hosseinzadeh"
date: "Last updated: 04 July, 2021"
output:
  html_document:
    toc: true
    toc_depth: 5
    toc_float: true
    theme: default 
    highlight: tango
    keep_md: yes
    code_folding: show
    mathjax: default
fontsize: 12pt
spacing: double
line-height: 1.5
---

## Introduction

- WNW streaming company is looking to improve viewer engagement to remain in the forefront of the competitive streaming market.      
- Increased user engagement means an increase in average hours watched per day per user - a key metric used to price advertisements. 
- Increase in hours watched per user will result in increased advertisement revenue and profit margins for our company.
- We have recently refined our recommendation engine algorithm with the aim of better recommendations to users.

***Problem Statement*** 

  - **Will rolling out the new recommendation engine algorithm to all WNW subscribers be beneficial?**

  - **Is there any bias in the data collection?**

  - **What improvements could be made to future A/B tests?**

  - **Descriptive and statistical analysis to determine key trends**

***Statistical techniques used***

- Statistical analysis:
  - A/B testing to measure the significance of average hours watched between demographic sub-groups in group B and group A.
    - Group B was used as the treatment group - users unknowingly treated with the new recommendation engine algorithm.
    - Group A was use as the control group - were not exposed to changed algorithm.
    - Outcome variable -> hours watched.
    - Two-sided two-sample t-test was used to compare the means and determine the significance of results.
  - Linear and multi regression analysis was conducted to determine key predictors in increasing user engagement and viewing hours.
    - Analysis of variance (ANOVA) was used to determine the best independent variables to be used in regression models.
    - Model accuracy was checked by using F-test.
  
- Statistical tests were performed on the complete dataset, the control group and the treatment group to determine if changes to the algorithm resulted in an increase in hours watched per user.
- Simple linear regression model revealed a statistically significant negative linear relationship between age and hours watched.  
- Multiple linear regression model found that **age** and **social metric** scores where the most accurate predictors of the outcome variable (hours watched). 
- Our investigation found statistically significant evidence that changes made to recommendation engine increased viewing hours in some demographic groups (2 & 4).
- In some demographic groups (1 & 3) the test results were either invalid or not statistically significant.
- It is recommended to conduct another campaign with a larger treatment sample size as well as a longer period of time to achieve more accuracy in testing the viewing habits of the subgroups.
- Overall, the recommendation engine can be recommended to be rolled out to all current users.

## Setup

Load the required packages to reproduce this project.


```r
# Load required packages:
library(knitr) # Dynamic report generation 
library(readr) # Read rectangular data, i.e., csv files
library(dplyr) # Data manipulation tasks
library(tidyr) # Tidy data
library(lubridate) # For dates and date-times
library(magrittr) # For pipes and double assignment
library(ggplot2) # For visualisations
library(gridExtra) # For visualisation
library(ggfortify) # for visualisation
library(car) # For statistical testing
library(kableExtra) # For HTML and PDF tables 

# set chunk options: do not messages or warnings set figure dims
knitr::opts_chunk$set(
  echo = TRUE,
  eval = TRUE,
  fig.width = 6, fig.height = 4,
  fig.align = "center",
  message = FALSE,
  warning = FALSE,
  comment = "") 
```

## Data

#### Retreive data

The dataset is available for download via [My GitHub Repository](http://github.com/carimo198/data_science_uni_projects).   


```r
# Load dataset
df <- read_csv("streaming_data.csv")

# Convert to correct data types
df$date <- as.Date(df$date, format = "%d/%m")
df$gender <- as.factor(df$gender)
df$group <- as.factor(df$group)
df$demographic <- as.factor(df$demographic)
df$social_metric <- as.factor(df$social_metric)

# table title
title_tbl1 <- "<strong>Preview of Why Not Watch streaming dataset</strong>"

# Display first 5 rows of df
head(df) %>% 
  kbl(caption = title_tbl1, align = "l", booktabs = TRUE) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>% 
  column_spec(8, color = "white", bold = TRUE,
              background = spec_color(df$hours_watched[1:6], end = 0.7))
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;'>
<caption style="font-size: initial !important;"><strong>Preview of Why Not Watch streaming dataset</strong></caption>
 <thead>
  <tr>
   <th style="text-align:left;"> date </th>
   <th style="text-align:left;"> gender </th>
   <th style="text-align:left;"> age </th>
   <th style="text-align:left;"> social_metric </th>
   <th style="text-align:left;"> time_since_signup </th>
   <th style="text-align:left;"> demographic </th>
   <th style="text-align:left;"> group </th>
   <th style="text-align:left;"> hours_watched </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 2021-07-01 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 28 </td>
   <td style="text-align:left;"> 5 </td>
   <td style="text-align:left;"> 19.3 </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> A </td>
   <td style="text-align:left;font-weight: bold;color: white !important;background-color: rgba(55, 90, 140, 1) !important;"> 4.08 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-07-01 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 32 </td>
   <td style="text-align:left;"> 7 </td>
   <td style="text-align:left;"> 11.5 </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> A </td>
   <td style="text-align:left;font-weight: bold;color: white !important;background-color: rgba(68, 1, 84, 1) !important;"> 2.99 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-07-01 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 39 </td>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> 4.3 </td>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:left;"> A </td>
   <td style="text-align:left;font-weight: bold;color: white !important;background-color: rgba(67, 191, 113, 1) !important;"> 5.74 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-07-01 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 52 </td>
   <td style="text-align:left;"> 10 </td>
   <td style="text-align:left;"> 9.5 </td>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> A </td>
   <td style="text-align:left;font-weight: bold;color: white !important;background-color: rgba(54, 93, 141, 1) !important;"> 4.13 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-07-01 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 25 </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 19.5 </td>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:left;"> A </td>
   <td style="text-align:left;font-weight: bold;color: white !important;background-color: rgba(39, 128, 142, 1) !important;"> 4.68 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2021-07-01 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 51 </td>
   <td style="text-align:left;"> 0 </td>
   <td style="text-align:left;"> 22.6 </td>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> A </td>
   <td style="text-align:left;font-weight: bold;color: white !important;background-color: rgba(72, 37, 119, 1) !important;"> 3.40 </td>
  </tr>
</tbody>
</table>
<br>

#### Data description

- Dataset contains 1000 observations and the following 8 variables:

  - *date*: 2021/07/01 to 2021/07/31 in the format yyyy/mm/dd 
  - *gender*: male or female
  - *age*: from 18 to 55 years
  - *social_metric*: combined metric based on previous viewing habits
  - *time_since_signup*: number of months since subscribing
  - *demographic*: number 1 to 4 assigned to customer based on age and gender
  - *group*: A is the control and B is the treated group - treatment applied from 2021/07/18 to group B only
  - *hours_watched*: number of hours watched per user per day    

- **Outcome variable** - *hours_watched*: 

  - key metric used to price ads
  - better recommendations improve user engagement & increase the average hours watched per user per day
  - measured in hours   

- **Important independent variables**:  

  - *age*
  - *social metric*   

- Data Preprocessing:   

  - converted variables to correct data type:
    - date to **DateTime**
    - categorical variables to **factor**   

- Variables converted to factors representing categorical data:

  - gender: 2 levels - **"F"** and **"M"**
  - social metric: 11 levels - **"0"** to **"10"**
  - demographic: 4 levels - **"1"** to **"4"**
  - group: 2 levels - **"A"** and **"B"**   

<br>

#### Manipulate data

Divide the dataset into three new dataframes containing data for only the following groups:

  - Group A from 07/01 to 07/31 - **before & after trial**
  - Group A from 07/18 to 07/31 - **control group**
  - Group B from 07/18 to 07/31 - **treatment group**  
  

```r
# There are two groups A/B - A is the control, B is the treated group

# create df for each group for later analysis
A_df <- df %>% filter(group == "A")
B_df <- df %>% filter(group == "B")

# Group A df for control group - after trial
A2_df <- A_df %>% 
  filter(date >= "2021-07-18")

# create new column for group A df - label dates from 18/7 as 'after trial' since data from this date corresponds to trial commencement and aligns with group B, otherwise 'before trial'
A_df <- A_df %>% 
  mutate(date_group = ifelse(date >= as.Date('2021-07-18'),
                             'after trial', 'before trial'))

# convert date group to factor
A_df$date_group <- as.factor(A_df$date_group)


# add to column categorising customers into 2 age groups: younger & older
A_df <- A_df %>% 
  mutate(age_group = ifelse(age > 35, "older", "younger"))

# convert age group to factor
A_df$age_group <- as.factor(A_df$age_group)

# add to column categorising customers into 2 age groups: younger & older
B_df$age_group <- if_else(B_df$age <= 35, "younger", "older")

# convert age group to factor
B_df$age_group <- as.factor(B_df$age_group)
```

<br>

## Group A - Analysis

We begin our analysis by testing the means of hours watched before and after trial to determine if group A data is suitable for examining the relationship between factors and outcome as well as ruling out any external factors influencing viewing hours. To compare the means of two population groups, the two-sample t-test (independent samples t-test) can be used.

<br>

#### Descriptive statistics - boxplot

Box plots introduced by John Tukey (1977), are based on percentiles and provide a quick way to visualise the distribution of data. We will use box plots to compare hours watched, before and after trial in group A.


```r
# visually compare data for group A before and after trial date - boxplot:
# plot title text
p2_title <- 
  "Box plot of hours watched for group A, before and after trial"

# set colour for plot
fillcol <- "#4271AE"
linecol <- "#1F3552"

# boxplot
A_df %>% 
  ggplot(aes(x = date_group, y = hours_watched)) + 
  geom_boxplot(colour = linecol, fill = fillcol) + 
  labs(x = "Date group", y = "Hours watched") + 
  ggtitle(p2_title) + 
  theme_bw() + 
  theme(plot.title = element_text(face = "bold"))
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/bp-A-1.png" style="display: block; margin: auto;" />
<br>

The box plot shows that there seems to be no obvious difference in hours watched for group A before and after trial. However, simply looking at the visualisation is not a precise way to measure statistical significance. We will use additional statistical methods to gather more evidence and then combine our findings to draw conclusions from. 

<br>

#### Summary statistics

Compute the mean and standard deviation for hours watched, before and after trial:


```r
# compute the mean and sd and n() for hours watched before and after trial
A_df %>% 
  group_by(date_group) %>% 
  summarise(
    mean = mean(hours_watched) %>% round(3),
    sd = sd(hours_watched) %>% round(3),
    n = n()
  ) %>% 
  kbl(align = "l", booktabs = TRUE,
      format = "html", table.attr = "style='width:40%;'") %>% 
  kable_classic("hover", font_size = 16) %>% 
  row_spec(0, bold = TRUE) %>% 
  footnote("Group A, entire dataset.")
```

<table style='width:40%; font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; margin-left: auto; margin-right: auto;border-bottom: 0;' class=" lightable-classic lightable-hover">
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;"> date_group </th>
   <th style="text-align:left;font-weight: bold;"> mean </th>
   <th style="text-align:left;font-weight: bold;"> sd </th>
   <th style="text-align:left;font-weight: bold;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> after trial </td>
   <td style="text-align:left;"> 4.402 </td>
   <td style="text-align:left;"> 1.378 </td>
   <td style="text-align:left;"> 332 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> before trial </td>
   <td style="text-align:left;"> 4.296 </td>
   <td style="text-align:left;"> 1.290 </td>
   <td style="text-align:left;"> 548 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
</tfoot>
</table>
<br>

#### Check assumptions before t-test 

To ensure accuracy of results from parametric statistical techniques (i.e. t.test, regression, means testing) certain assumptions must be met:

- Assumption of normality: since sample size is large ($n>30$), the central limit theorem can be applied and normal distribution can be assumed.
- Homogeneity of variance - Levene's test is a robust method to test for equal variance and has the following hypotheses (Bruce et al. 2020):
  - $H_0: \sigma_1^2 = \sigma_1^2$
  - $H_A: \sigma_1^2 \ne \sigma_1^2$


```r
# Homogeneity of variance - examine variation
# Table title
title_tbl3 <- "<strong>Assumption of equal variance - Levene's test</strong>"
# Levene's test (from car library)
leveneTest(hours_watched ~ date_group, data = A_df, center = mean) %>% 
  kbl(caption = title_tbl3, align = "l", booktabs = TRUE, 
      digits = 3, format = "html", table.attr = "style='width:40%;'") %>% 
  kable_classic("hover", font_size = 16) %>%
  row_spec(0, bold = TRUE) %>%
  footnote("Group A, entire dataset.")
```

<table style='width:40%; font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; margin-left: auto; margin-right: auto;border-bottom: 0;' class=" lightable-classic lightable-hover">
<caption style="font-size: initial !important;"><strong>Assumption of equal variance - Levene's test</strong></caption>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;">   </th>
   <th style="text-align:left;font-weight: bold;"> Df </th>
   <th style="text-align:left;font-weight: bold;"> F value </th>
   <th style="text-align:left;font-weight: bold;"> Pr(&gt;F) </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> group </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 0.644 </td>
   <td style="text-align:left;"> 0.422 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 878 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
</tfoot>
</table>
<br>

The p-value for Levene's test of equal variance for hours watched before trial and hours watched after trial was $p=0.42$. We find $p>0.05$, therefore, we fail to reject $H_0$ and are safe to assume equal variance.

<br>

#### T-test - before & after trial

We conducted a two-sided, two-sample t-test to compare the means of hours watched in group A, before and after trial with the following hypotheses: 

- $H_0: \mu_1 = \mu_2$
- $H_A: \mu_1 \ne \mu_2$


```r
# the null hypothesis: the mean between the two groups are the same
# two-sample test statistic
t.test(hours_watched ~ date_group, data = A_df, 
       var.equal = TRUE, alternative = "two.sided") 
```

```

	Two Sample t-test

data:  hours_watched by date_group
t = 1.1476, df = 878, p-value = 0.2515
alternative hypothesis: true difference in means between group after trial and group before trial is not equal to 0
95 percent confidence interval:
 -0.07505363  0.28639081
sample estimates:
 mean in group after trial mean in group before trial 
                  4.401928                   4.296259 
```

Result of the two-sample t-test found no statistical significance in the difference between the mean hours watched before and after trial in group A, $t = 1.148$, $df = 878$, $p>0.05$, $95\%CI[-0.075, 0.286]$. Therefore, there is not enough evidence to reject the null hypothesis. 

This makes group A (entire dataset) ideal for examining the relationships between the independent variables and the outcome variable via descriptive and regression analysis.

## Relationship between age, gender, demographic & hours watched 

#### Descriptive analysis - scatter plot


```r
# Scatter plot of age & gender vs hours watched with line of best fit
sp1 <- A_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = gender)) +
  geom_smooth(method = "lm", se = FALSE) +
  ggtitle("Age & Gender vs Hours Watched") +
  theme_bw()

# scatter plot of relationship between age, demographic & hours_watched
sp2 <- A_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = demographic)) +
  geom_smooth(method = "lm", se = FALSE) +
  ggtitle("Age & Demographic vs Hours Watched") +
  theme_bw()

grid.arrange(sp1, sp2, ncol = 2)
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/sp-dem-A-1.png" style="display: block; margin: auto;" />
<br>

The scatter plots illustrate: 

- negative correlation between age and hours watched
- gender does not appear to be correlated to hours watched
- demographic is based on customer age group and gender

<br>

#### Summary statistics - *demographic*


```r
# summary stats for demographic - Group A
A_df %>% group_by(demographic, gender) %>% 
  summarise(
    n = n(),
    min = min(age, na.rm = TRUE),
    max = max(age, na.rm = TRUE),
    mean = mean(hours_watched, na.rm = TRUE),
    sd = sd(hours_watched, na.rm = TRUE)
  ) %>% 
  kbl(align = "l", booktabs = TRUE, digits = 3) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>%
  row_spec(0, bold = TRUE) %>% 
  footnote("Group A, entire dataset.",
           number = 
             c("Footnote 1; mean = mean of hours watched",
               "Footnote 2; sd = standard deviation of hours watched"))
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;'>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;"> demographic </th>
   <th style="text-align:left;font-weight: bold;"> gender </th>
   <th style="text-align:left;font-weight: bold;"> n </th>
   <th style="text-align:left;font-weight: bold;"> min </th>
   <th style="text-align:left;font-weight: bold;"> max </th>
   <th style="text-align:left;font-weight: bold;"> mean </th>
   <th style="text-align:left;font-weight: bold;"> sd </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 203 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 5.024 </td>
   <td style="text-align:left;"> 1.064 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 236 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 5.014 </td>
   <td style="text-align:left;"> 1.149 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 197 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 3.578 </td>
   <td style="text-align:left;"> 1.216 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 244 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 3.719 </td>
   <td style="text-align:left;"> 1.116 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup>1</sup> Footnote 1; mean = mean of hours watched</td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup>2</sup> Footnote 2; sd = standard deviation of hours watched</td></tr>
</tfoot>
</table>
<br>

This demonstrates that demographic is directly related to gender and age:

- D1 - females aged 18-35 
- D2  - males aged 18-35 
- D3 - females aged 36-55 
- D4 - males aged 36-55

Therefore, we can remove demographic and gender from future multi regression analysis due to collinearity.

<br>

#### Summary statistics - *age* 


```r
# summary stats on age in group A
A_df %>% summarise(
  mean = mean(age, na.rm=TRUE),
  sd = sd(age, na.rm=TRUE),
  min = min(age, na.rm=TRUE),
  Q1 = quantile(age, probs=0.25, na.rm=TRUE),
  median = median(age, na.rm=TRUE),
  Q3 = quantile(age, probs=0.75, na.rm=TRUE),
  max = max(age, na.rm=TRUE),
  IQR = IQR(age, na.rm=TRUE),
  n = n()
) %>%
  round(3) %>% 
  kbl(align = "l", booktabs = TRUE, digits = 3) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>%
  row_spec(0, bold = TRUE) %>% 
  footnote("Group A, entire dataset.") 
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;'>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;"> mean </th>
   <th style="text-align:left;font-weight: bold;"> sd </th>
   <th style="text-align:left;font-weight: bold;"> min </th>
   <th style="text-align:left;font-weight: bold;"> Q1 </th>
   <th style="text-align:left;font-weight: bold;"> median </th>
   <th style="text-align:left;font-weight: bold;"> Q3 </th>
   <th style="text-align:left;font-weight: bold;"> max </th>
   <th style="text-align:left;font-weight: bold;"> IQR </th>
   <th style="text-align:left;font-weight: bold;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 36.151 </td>
   <td style="text-align:left;"> 10.728 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 27 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 45 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 880 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
</tfoot>
</table>
<br>

#### Simple linear regression model - *age*

Simple linear regression provides a model of the relationship between the magnitude of one variable, $X$, and a second variable, $Y$. Correlation is also another method to measure how two variables are related. The difference is that correlation measures the *strength* of the relationship between two variables, whereas regression quantifies the *nature* of the relationship (Brice et al. 2020).


```r
# simple linear regression model
age_model <- lm(hours_watched ~ age, data = A_df)
age_model %>% summary()
```

```

Call:
lm(formula = hours_watched ~ age, data = A_df)

Residuals:
    Min      1Q  Median      3Q     Max 
-3.5142 -0.7242 -0.0030  0.7474  3.0046 

Coefficients:
             Estimate Std. Error t value Pr(>|t|)    
(Intercept)  6.980111   0.126542   55.16   <2e-16 ***
age         -0.073137   0.003356  -21.79   <2e-16 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 1.067 on 878 degrees of freedom
Multiple R-squared:  0.3511,	Adjusted R-squared:  0.3503 
F-statistic:   475 on 1 and 878 DF,  p-value: < 2.2e-16
```

We can also compute confidence intervals for intercept $a$, and slope, $b$:


```r
# table title
title_tbl4 <- "<strong>Confidence interval for intercept and slope</strong>"
# compute CI for intercept and slope
age_model %>% 
  confint() %>% 
  kbl(caption = title_tbl4, align = "l", booktabs = TRUE, digits = 3) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>% 
  row_spec(2, bold = TRUE) %>% 
  footnote("Simple linear regression model - hours watched ~ age",
           number = c("Footnote 1; 'age' row represents CI interval for slope"))
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;'>
<caption style="font-size: initial !important;"><strong>Confidence interval for intercept and slope</strong></caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;"> 2.5 % </th>
   <th style="text-align:left;"> 97.5 % </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> (Intercept) </td>
   <td style="text-align:left;"> 6.732 </td>
   <td style="text-align:left;"> 7.228 </td>
  </tr>
  <tr>
   <td style="text-align:left;font-weight: bold;"> age </td>
   <td style="text-align:left;font-weight: bold;"> -0.080 </td>
   <td style="text-align:left;font-weight: bold;"> -0.067 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Simple linear regression model - hours watched ~ age</td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup>1</sup> Footnote 1; 'age' row represents CI interval for slope</td></tr>
</tfoot>
</table>

As the significance of $RSE$ can be subjective as it provides an absolute measure of lack of fit of our model to the data. However, since it is measured in the units of $Y$, it is not always clear what constitutes a good $RSE$. Therefore, We can compare it to the average hours watched for the entire group A which will provide us with the percentage error:


```r
# compare RSE with avg hours watched to compute percentage error
round( sigma(age_model)/mean(A_df$hours_watched) * 100, 2)
```

```
[1] 24.62
```
<br>

#### Results of regression model - *age*

- $R$ squared ($R^2$) - $R^2 = 0.351$, reflects the proportion of variability in our dependent *hours_watched* variable that can be explained by a linear relationship with the predictor variable, *age*. 

- $F$-statistic - $F(1, 878)=475$, $p<0.001$, tests the overall regression model with the following statistical hypotheses:

  - $H_0$: The data does not fit the linear regression model
  - $H_A$: The data fit the linear regression model

- Slope - $b = -0.073$, represents the average decrease in *hours_watched* per one year increase in *age*. 

- Residual standard error ($RSE$) - $RSE = 1.067$, an estimate of the standard deviation of $\epsilon$, it is the average amount that the response will deviate from the true regression line. 

<br>

#### Validating the assumptions for regression model 

An important component of assessing regression models is validating the assumptions for linear regression:

- Independence
- Linearity
- Normality of residuals
- Homoscedasticity 

We can validate normality of residuals and homoscedasticity by visualising residuals. Here we are looking to see if the median is approximately 0 and whether the first and third quantiles are approximately the same. If unusual patterns are detected, it might suggest that the residuals are either not normally distributed, or heteroscedastic. In both situations, our interpretation of the slopes could be incorrect as well as having a negative impact on the performance of our model (Rhys 2020). 


```r
# Age linear regression model diagnostics with ggfortify and ggplot2
autoplot(age_model, label.size = 3, 
         colour = "dodgerblue3", alpha = 0.7,
         smooth.colour = "red") + 
  theme_bw()
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/age-lm-diagnostics-1.png" style="display: block; margin: auto;" />
<br>

#### Interpretation of descriptive analysis and regression model 

The bivariate analysis via a scatter plot demonstrated evidence of a negative linear relationship between age and the outcome variable hours watched. A regression model was then fitted to predict hours watched using age of users. The overall model was statistically significant, $F(1, 878)=475$, $p<0.001$ and explained 35.1% of the variability in hours watched, $R^2 = 0.351$. The negative slope for age was statistically significant, $b=-0.073, t(878) = -21.79, p<0.001, 95\%CI[-0.080, -0.067]$. Model accuracy analysis results show the model has an $RSE$ of $1.067$ and when compared to the average hours watched per viewer, has a percentage error of $24.62\%$. The combined analysis of our $RSE$, $R^2$ and $F$ statistic results suggest that our model has an ok fit. Final inspection of the residuals supported normality and homoscedasticity.  

Therefore, we conclude that there was statistically significant evidence that *age* was negatively related to *hours watched* with the data fitting the regression model. 

## Relationship between hours watched & gender

#### Homogeneity of variance

Check assumption of equal variance:

  - $H_0: \sigma_1^2 = \sigma_2^2$
  - $H_A: \sigma_1^2 \ne \sigma_2^2$
  

```r
# Levene's test gender hours watched
leveneTest(hours_watched ~ gender, data = A_df, center = mean) %>% 
  kbl(caption = title_tbl3, align = "l", booktabs = TRUE, digits = 3, 
      format = "html", table.attr = "style='width:40%;'") %>% 
  kable_classic("hover", font_size = 16) %>% 
  row_spec(0, bold = TRUE) %>% 
  footnote("Group A, entire dataset.")
```

<table style='width:40%; font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; margin-left: auto; margin-right: auto;border-bottom: 0;' class=" lightable-classic lightable-hover">
<caption style="font-size: initial !important;"><strong>Assumption of equal variance - Levene's test</strong></caption>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;">   </th>
   <th style="text-align:left;font-weight: bold;"> Df </th>
   <th style="text-align:left;font-weight: bold;"> F value </th>
   <th style="text-align:left;font-weight: bold;"> Pr(&gt;F) </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> group </td>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> 0.796 </td>
   <td style="text-align:left;"> 0.373 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 878 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
</tfoot>
</table>

The p-value for the Levene's test of equal variance between gender and hours watched was $p=0.373$, $p>0.05$. Therefore, we do not have enough evidence to reject the null hypothesis and we can assume equal variance. 

We can compare the means of hours watched based on gender by using a two-sample t-test with the following hypotheses:

  - $H_0: \mu_1 = \mu_2$
  - $H_A: \mu_1 \ne \mu_2$
  

```r
# t-test for gender and hours watched
t.test(hours_watched ~ gender, data = A_df, 
       var.equal = TRUE, alternative = "two.sided")
```

```

	Two Sample t-test

data:  hours_watched by gender
t = -0.48752, df = 878, p-value = 0.626
alternative hypothesis: true difference in means between group F and group M is not equal to 0
95 percent confidence interval:
 -0.2197551  0.1323051
sample estimates:
mean in group F mean in group M 
       4.312275        4.356000 
```

The result of the two-sample t-test assuming equal variance and a two-sided hypothesis test found there was no statistically significant difference between the mean hours watched between females (4.31 hours) and males (4.35 hours), $t(df = 878) = -0.487, 95\%CI[-0.220, 0.132]$, $p-value = 0.626, p>0.05$. 

<br>

#### ANOVA using linear model - *gender*

An analysis of variance (ANOVA) model can also be used to examine how the variance in data is contributed to by the different groups. A one-way ANOVA model where there is one factor (variable) with at least two levels is a parametric test designed to compare the means of two or more groups. The null hypothesis states that the means of all groups to be tested are equal. 


```r
# linear model for gender
gender_model <- lm(hours_watched ~ gender, data = A_df)

# ANOVA - gender
Anova(gender_model) # from car package
```

```
Anova Table (Type II tests)

Response: hours_watched
           Sum Sq  Df F value Pr(>F)
gender       0.42   1  0.2377  0.626
Residuals 1540.96 878               
```

The result of the ANOVA test found no statistically significant difference between hours watched and gender, $F(1, 878) = 0.238$, $p$-value = $0.626$, $p>0.05$.

From our analysis, we can conclude that gender is **not** a good predictor of viewing hours for our users.

## Relationship between hours watched & social_metric

#### Descriptive analysis

We can compare the distribution of the different levels of *social metric* with box plots: 


```r
# boxplot hours watched vs social metric
A_df %>% 
  ggplot(aes(x= social_metric, y = hours_watched)) +
  geom_boxplot(fill = "dodgerblue3") +
  ggtitle("Boxplot of social metric vs hours watched") +
  xlab("Social Metric") +
  ylab("Hours Watched") +
  theme_bw()
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/unnamed-chunk-10-1.png" style="display: block; margin: auto;" />

#### Homogeneity of variance

Check assumption of equal variance before conducting ANOVA test:

  - $H_0: \sigma_1^2 = \sigma_2^2$
  - $H_A: \sigma_1^2 \ne \sigma_2^2$
  

```r
# homogeneity of variance for hours_watched vs social_metric
leveneTest(hours_watched ~ social_metric, data = A_df, center = "mean") %>% 
  kbl(caption = title_tbl3, align = "l", booktabs = TRUE, digits = 3,
      format = "html", table.attr = "style='width:40%;'") %>% 
  kable_classic("hover", font_size = 16) %>% 
  footnote("Group A, entire dataset.")
```

<table style='width:40%; font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; margin-left: auto; margin-right: auto;border-bottom: 0;' class=" lightable-classic lightable-hover">
<caption style="font-size: initial !important;"><strong>Assumption of equal variance - Levene's test</strong></caption>
 <thead>
  <tr>
   <th style="text-align:left;">   </th>
   <th style="text-align:left;"> Df </th>
   <th style="text-align:left;"> F value </th>
   <th style="text-align:left;"> Pr(&gt;F) </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> group </td>
   <td style="text-align:left;"> 10 </td>
   <td style="text-align:left;"> 1.041 </td>
   <td style="text-align:left;"> 0.406 </td>
  </tr>
  <tr>
   <td style="text-align:left;">  </td>
   <td style="text-align:left;"> 869 </td>
   <td style="text-align:left;"> NA </td>
   <td style="text-align:left;"> NA </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
</tfoot>
</table>
<br>

The p-value for the Levene's test of equal variance between social metric and hours watched was $p=0.406$, $p>0.05$. Therefore, we do not have enough evidence to reject the null hypothesis and we can assume equal variance. 

<br>

#### One-way ANOVA - *social metric*

Since social metric has more than two factor levels, a one-way ANOVA would be appropriate to compare the means between the groups. The hypotheses are as follows (Bruce et al. 2020):

  - $H_0: \mu_1 = ... = \mu_10$
  - $H_A: \mu_1 \ne ... \ne \mu_10$


```r
# one-way ANOVA testing the means of different social_metric scores
social_anova <- aov(hours_watched ~ social_metric, data = A_df)

# Display results
social_anova %>% 
  summary() 
```

```
               Df Sum Sq Mean Sq F value   Pr(>F)    
social_metric  10   82.7   8.271   4.928 5.83e-07 ***
Residuals     869 1458.7   1.679                     
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

The result of the ANOVA test shows there is a statistically significant difference between hours watched and social metric, F(10, 869) = 4.928, $p<0.001$. To get a deeper understanding on where the exact difference is we will follow up with a *'post hoc'* test using regression analysis.

<br>

#### Linear regression - *social metric*


```r
# linear regression analysis for categorical social_metric
social_model <- lm(hours_watched ~ social_metric, data = A_df)
summary(social_model)
```

```

Call:
lm(formula = hours_watched ~ social_metric, data = A_df)

Residuals:
    Min      1Q  Median      3Q     Max 
-3.7900 -0.8182  0.0137  0.8511  3.6063 

Coefficients:
                Estimate Std. Error t value Pr(>|t|)    
(Intercept)       3.7382     0.1731  21.592  < 2e-16 ***
social_metric1    0.1040     0.2170   0.479 0.631821    
social_metric2    0.4828     0.2205   2.189 0.028833 *  
social_metric3    0.5437     0.2230   2.438 0.014962 *  
social_metric4    0.5518     0.2179   2.533 0.011489 *  
social_metric5    0.5402     0.2282   2.368 0.018123 *  
social_metric6    0.6699     0.2301   2.911 0.003700 ** 
social_metric7    0.9555     0.2282   4.188 3.11e-05 ***
social_metric8    0.8007     0.2170   3.689 0.000239 ***
social_metric9    1.0704     0.2220   4.822 1.67e-06 ***
social_metric10   0.8705     0.2594   3.356 0.000825 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 1.296 on 869 degrees of freedom
Multiple R-squared:  0.05366,	Adjusted R-squared:  0.04277 
F-statistic: 4.928 on 10 and 869 DF,  p-value: 5.83e-07
```

The result of the simple linear regression model for *hours_watched* and *social_metric* demonstrate that there is considerable significant evidence for social metric scores of 7, 8, 9 and 10 to have higher average hours watched compared to the reference social metric score of 0 with $p < 0.001$. There also is significant evidence that social metric scores of 2, 3, 4, 5 and 6 have higher average viewing hours than the reference score of 0 with $p < 0.05$. However, the difference between the average viewing hours of social metric 0 and 1 is not statistically significant $p=0.63$, $p>0.05$. The $F$-statistic $4.928, p < 0.001$ illustrates that there was a statistically significant evidence that the data does fit the linear regression model. The $RSE$ of $1.296$ and when compared to the average hours watched per viewer, has a percentage error of $29.88\%$. The combined analysis of our $RSE$, $R^2$ and $F$-statistic results suggest that our model has an ok fit. 

Therefore, social metric is a good predictor variable for our outcome variable hours watched. 


## Relationship between hours watched & time since sign-up

<br>

#### Summary statistics 


```r
# summary stats on time since sign-up - group A
A_df %>% summarise(
  mean = mean(time_since_signup, na.rm=TRUE),
  sd = sd(time_since_signup, na.rm=TRUE),
  min = min(time_since_signup, na.rm=TRUE),
  Q1 = quantile(time_since_signup, probs=0.25, na.rm=TRUE),
  median = median(time_since_signup, na.rm=TRUE),
  Q3 = quantile(time_since_signup, probs=0.75, na.rm=TRUE),
  max = max(time_since_signup, na.rm=TRUE),
  IQR = IQR(time_since_signup, na.rm=TRUE),
  n = n()
) %>%
  round(3) %>% 
  kbl(align = "l", booktabs = TRUE, digits = 3) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>%
  row_spec(0, bold = TRUE) %>% 
  footnote("Group A, entire dataset.",
           number = c("Footnote 1; measured in months"))
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;'>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;"> mean </th>
   <th style="text-align:left;font-weight: bold;"> sd </th>
   <th style="text-align:left;font-weight: bold;"> min </th>
   <th style="text-align:left;font-weight: bold;"> Q1 </th>
   <th style="text-align:left;font-weight: bold;"> median </th>
   <th style="text-align:left;font-weight: bold;"> Q3 </th>
   <th style="text-align:left;font-weight: bold;"> max </th>
   <th style="text-align:left;font-weight: bold;"> IQR </th>
   <th style="text-align:left;font-weight: bold;"> n </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 12.034 </td>
   <td style="text-align:left;"> 7.224 </td>
   <td style="text-align:left;"> 0 </td>
   <td style="text-align:left;"> 5.875 </td>
   <td style="text-align:left;"> 11.9 </td>
   <td style="text-align:left;"> 18.7 </td>
   <td style="text-align:left;"> 24 </td>
   <td style="text-align:left;"> 12.825 </td>
   <td style="text-align:left;"> 880 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup>1</sup> Footnote 1; measured in months</td></tr>
</tfoot>
</table>
<br>

#### Descriptive analysis - scatter plot


```r
# Scatter plot of relationship between time_since_signup and hours_watched
A_df %>% 
  ggplot(aes(x = time_since_signup, y = hours_watched)) +
  geom_point(colour = "dodgerblue3") +
  ggtitle("Relationship between time since sign-up and hours watched") +
  xlab("Time since sign-up") +
  ylab("Hours watched") + 
  theme_bw()
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/unnamed-chunk-15-1.png" style="display: block; margin: auto;" />
<br>

The scatter plot does not show a relationship between time since sign up and hours watched.

<br>

#### Correlation analysis

We can compute a correlation analysis to measure the strength of the linear relationship between time since sign-up and hours watched with the following statistical hypothesis for the Pearson correlation coefficient, $r$:

  - $H_0: r=0$
  - $H_A: r \ne 0$

```r
cor.test(A_df$hours_watched, A_df$time_since_signup,
         method = "pearson")
```

```

	Pearson's product-moment correlation

data:  A_df$hours_watched and A_df$time_since_signup
t = -0.19602, df = 878, p-value = 0.8446
alternative hypothesis: true correlation is not equal to 0
95 percent confidence interval:
 -0.07267021  0.05949767
sample estimates:
        cor 
-0.00661516 
```

A Pearson's correlation was calculated to measure the relationship between *time since signup* and *hours watched*. The correlation was not statistically significant with $r = -0.006, p=0.845, 95\%CI[-0.073, -0.059]$.

The scatter plot along with correlation analysis demonstrates that the there is no relationship between time since sign up and streaming habits of viewers. 

## Multiple Regression Analysis

Through our analysis we have concluded that ***age*** and ***social metric*** are the most important independent variables for predicting our outcome variable and are best suited for use in multiple regression model.


```r
# Multi regression model - age + social metric
mr_age_social <- lm(
  hours_watched ~ age + social_metric,
  data = A_df
)

# display results
summary(mr_age_social)
```

```

Call:
lm(formula = hours_watched ~ age + social_metric, data = A_df)

Residuals:
    Min      1Q  Median      3Q     Max 
-3.6051 -0.6516 -0.0184  0.6919  2.8343 

Coefficients:
                 Estimate Std. Error t value Pr(>|t|)    
(Intercept)      6.493529   0.186983  34.728  < 2e-16 ***
age             -0.072237   0.003284 -21.994  < 2e-16 ***
social_metric1  -0.019804   0.174104  -0.114 0.909465    
social_metric2   0.356887   0.176896   2.018 0.043951 *  
social_metric3   0.393124   0.178918   2.197 0.028267 *  
social_metric4   0.393983   0.174818   2.254 0.024466 *  
social_metric5   0.478561   0.182964   2.616 0.009062 ** 
social_metric6   0.516064   0.184663   2.795 0.005311 ** 
social_metric7   0.616283   0.183592   3.357 0.000823 ***
social_metric8   0.632602   0.174181   3.632 0.000298 ***
social_metric9   0.875759   0.178188   4.915 1.06e-06 ***
social_metric10  0.868159   0.207965   4.175 3.29e-05 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Residual standard error: 1.039 on 868 degrees of freedom
Multiple R-squared:  0.3923,	Adjusted R-squared:  0.3846 
F-statistic: 50.95 on 11 and 868 DF,  p-value: < 2.2e-16
```


```r
# Age + social metric multi regression model diagnostics with ggfortify and ggplot2
autoplot(mr_age_social, label.size = 3, 
         colour = "dodgerblue3", alpha = 0.7,
         smooth.colour = "red") + 
  theme_bw()
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/unnamed-chunk-18-1.png" style="display: block; margin: auto;" />
<br>

#### Interpretation

The results from our multi regression analysis demonstrates that: 

- Age, $b=-0.072$ with $p<0.001$
- for every 1 year increase in age, an average reduction of **-0.072 hrs (4.2 mins)** in hours watched per user
- higher social metric scores result in increase hours watched compared to score 0; i.e. score of 10 results in an average increase of **+0.869 hours (52 mins)** compared to score of 0 with $p<0.001$
- hours watched between social metric 0 and 1 are not statistically significance, $p > 0.05$
- $F(11, 868) = 50.95$, $p < 0.001$
- $R^2 = 0.392$
- $RSE = 1.039$
- error percentage = $23.96\%$.

From the visualisation of residual behaviour in our model, we can validate the assumptions of linear regression. Therefore, our model can be used to predict outcomes in user viewing habits based on two variables, age and social metric.

<br>

## A/B test - check bias

#### Scatter plot - *demographic* across groups

Visualise *demographic* across the groups via scatter plots to see if we detect any problematic patterns:


```r
# hours_watched, age, demographic - Group A, complete (population)
mvp4 <- A_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = demographic)) +
  ggtitle("Hours watched vs age and demographic - Group A, complete") + 
  xlab("Age") + ylab("Hours Watched") +
  theme_bw()

# hours_watched, age, demographic - Group A, control
mvp5 <- A2_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = demographic)) +
  ggtitle("Hours watched vs age and demographic - Group A, control") + 
  xlab("Age") + ylab("Hours Watched") +
  theme_bw()

# hours_watched, age, demographic - Group B, target
mvp6 <- B_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = demographic)) +
  ggtitle("Hours watched vs age and demographic - Group B, treatment") + 
  xlab("Age") + ylab("Hours Watched") +
  theme_bw()

grid.arrange(mvp4, mvp5, mvp6, nrow = 2)
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/unnamed-chunk-19-1.png" style="display: block; margin: auto;" />

The scatter plots demonstrate an imbalance of demographic and age in target group B which has a much higher proportion of demographic group 4.

<br>

#### Scatter plot - *gender* across groups

Visualise *gender* across the groups via scatter plots to see if we detect any problematic patterns:


```r
# hours_watched, age, gender - Group A, complete (population)
mvp1 <- A_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = gender)) +
  ggtitle("Hours watched vs age and gender - Group A, complete") + 
  xlab("Age") + ylab("Hours Watched") +
  theme_bw()

# hours_watched, age, gender - Group A, control
mvp2 <- A2_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = gender)) +
  ggtitle("Hours watched vs age and gender - Group A, control") + 
  xlab("Age") + ylab("Hours Watched") +
  theme_bw()

# hours_watched, age, gender - Group B, target
mvp3 <- B_df %>% 
  ggplot(aes(x = age, y = hours_watched)) +
  geom_point(aes(colour = gender)) +
  ggtitle("Hours watched vs age and gender - Group B, target") + 
  xlab("Age") + ylab("Hours Watched") +
  theme_bw()

grid.arrange(mvp1, mvp2, mvp3, nrow = 2)
```

<img src="/Users/mohammadhosseinzadeh/Desktop/GitHub/data_science_uni_projects/docs/index_files/figure-html/unnamed-chunk-20-1.png" style="display: block; margin: auto;" />

We can see a clear gender and age imbalance in target group B compared to both control group A and population group A. Target group B has a much higher proportion of older males. 

<br>

#### Summary statistics, *demographic* - group A, entire data 


```r
# summary of group A demographic
A_df %>% group_by(demographic, gender) %>% 
  summarise(
    n = n(),
    Min = min(age, na.rm = TRUE),
    Max = max(age, na.rm = TRUE),
    Mean = mean(hours_watched, na.rm = TRUE),
    Median = median(hours_watched, na.rm = TRUE),
    SD = sd(hours_watched, na.rm = TRUE)) %>% 
  ungroup() %>%  
  mutate(Prop = n / sum(n)) %>%
  kbl(align = "l", booktabs = TRUE, digits = 3) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>%
  row_spec(0, bold = TRUE) %>% 
  footnote("Group A, entire dataset.") %>% 
  column_spec(9, bold = TRUE, background = "#6897BB")
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;'>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;"> demographic </th>
   <th style="text-align:left;font-weight: bold;"> gender </th>
   <th style="text-align:left;font-weight: bold;"> n </th>
   <th style="text-align:left;font-weight: bold;"> Min </th>
   <th style="text-align:left;font-weight: bold;"> Max </th>
   <th style="text-align:left;font-weight: bold;"> Mean </th>
   <th style="text-align:left;font-weight: bold;"> Median </th>
   <th style="text-align:left;font-weight: bold;"> SD </th>
   <th style="text-align:left;font-weight: bold;"> Prop </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 203 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 5.024 </td>
   <td style="text-align:left;"> 5.12 </td>
   <td style="text-align:left;"> 1.064 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.231 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 236 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 5.014 </td>
   <td style="text-align:left;"> 5.04 </td>
   <td style="text-align:left;"> 1.149 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.268 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 197 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 3.578 </td>
   <td style="text-align:left;"> 3.67 </td>
   <td style="text-align:left;"> 1.216 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.224 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 244 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 3.719 </td>
   <td style="text-align:left;"> 3.75 </td>
   <td style="text-align:left;"> 1.116 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.277 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, entire dataset.</td></tr>
</tfoot>
</table>
<br>

#### Summary statistics, *demographic* - group A, control 


```r
A2_df %>% group_by(demographic, gender) %>% 
  summarise(
    n = n(),
    Min = min(age, na.rm = TRUE),
    Max = max(age, na.rm = TRUE),
    Mean = mean(hours_watched, na.rm = TRUE),
    Median = median(hours_watched, na.rm = TRUE),
    SD = sd(hours_watched, na.rm = TRUE)
  ) %>% 
  ungroup() %>%  
  mutate(Prop = n / sum(n)) %>%
  kbl(align = "l", booktabs = TRUE, digits = 3) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>% 
  row_spec(0, bold = TRUE) %>% 
  footnote("Group A, control.") %>% 
  column_spec(9, bold = TRUE, background = "#6897BB")
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;'>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;"> demographic </th>
   <th style="text-align:left;font-weight: bold;"> gender </th>
   <th style="text-align:left;font-weight: bold;"> n </th>
   <th style="text-align:left;font-weight: bold;"> Min </th>
   <th style="text-align:left;font-weight: bold;"> Max </th>
   <th style="text-align:left;font-weight: bold;"> Mean </th>
   <th style="text-align:left;font-weight: bold;"> Median </th>
   <th style="text-align:left;font-weight: bold;"> SD </th>
   <th style="text-align:left;font-weight: bold;"> Prop </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 74 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 5.199 </td>
   <td style="text-align:left;"> 5.210 </td>
   <td style="text-align:left;"> 1.020 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.223 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 96 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 5.111 </td>
   <td style="text-align:left;"> 5.005 </td>
   <td style="text-align:left;"> 1.222 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.289 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 89 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 3.560 </td>
   <td style="text-align:left;"> 3.730 </td>
   <td style="text-align:left;"> 1.255 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.268 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 73 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 3.688 </td>
   <td style="text-align:left;"> 3.790 </td>
   <td style="text-align:left;"> 1.030 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.220 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group A, control.</td></tr>
</tfoot>
</table>
<br>

#### Summary statistics, *demographic* - group B, treatment


```r
B_df %>% group_by(demographic, gender) %>% 
  summarise(
    n = n(),
    Min = min(age, na.rm = TRUE),
    Max = max(age, na.rm = TRUE),
    Mean = mean(hours_watched, na.rm = TRUE),
    Median = median(hours_watched, na.rm = TRUE),
    SD = sd(hours_watched, na.rm = TRUE)
  ) %>% 
  ungroup() %>%  
  mutate(Prop = n / sum(n)) %>%
  kbl(align = "l", booktabs = TRUE, digits = 3) %>% 
  kable_classic("hover", font_size = 16, full_width = FALSE) %>%
  row_spec(0, bold = TRUE) %>% 
  footnote("Group B, treatment.") %>% 
  column_spec(9, bold = TRUE, background = "#6897BB")
```

<table class=" lightable-classic lightable-hover" style='font-size: 16px; font-family: "Arial Narrow", "Source Sans Pro", sans-serif; width: auto !important; margin-left: auto; margin-right: auto;border-bottom: 0;'>
 <thead>
  <tr>
   <th style="text-align:left;font-weight: bold;"> demographic </th>
   <th style="text-align:left;font-weight: bold;"> gender </th>
   <th style="text-align:left;font-weight: bold;"> n </th>
   <th style="text-align:left;font-weight: bold;"> Min </th>
   <th style="text-align:left;font-weight: bold;"> Max </th>
   <th style="text-align:left;font-weight: bold;"> Mean </th>
   <th style="text-align:left;font-weight: bold;"> Median </th>
   <th style="text-align:left;font-weight: bold;"> SD </th>
   <th style="text-align:left;font-weight: bold;"> Prop </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> 1 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 13 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 34 </td>
   <td style="text-align:left;"> 5.745 </td>
   <td style="text-align:left;"> 5.540 </td>
   <td style="text-align:left;"> 0.684 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.108 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 2 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 32 </td>
   <td style="text-align:left;"> 18 </td>
   <td style="text-align:left;"> 35 </td>
   <td style="text-align:left;"> 5.706 </td>
   <td style="text-align:left;"> 5.725 </td>
   <td style="text-align:left;"> 1.103 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.267 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 3 </td>
   <td style="text-align:left;"> F </td>
   <td style="text-align:left;"> 16 </td>
   <td style="text-align:left;"> 38 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 4.221 </td>
   <td style="text-align:left;"> 4.480 </td>
   <td style="text-align:left;"> 1.356 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.133 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> 4 </td>
   <td style="text-align:left;"> M </td>
   <td style="text-align:left;"> 59 </td>
   <td style="text-align:left;"> 36 </td>
   <td style="text-align:left;"> 55 </td>
   <td style="text-align:left;"> 4.279 </td>
   <td style="text-align:left;"> 4.255 </td>
   <td style="text-align:left;"> 1.175 </td>
   <td style="text-align:left;font-weight: bold;background-color: #6897BB !important;"> 0.492 </td>
  </tr>
</tbody>
<tfoot>
<tr><td style="padding: 0; " colspan="100%"><span style="font-style: italic;">Note: </span></td></tr>
<tr><td style="padding: 0; " colspan="100%">
<sup></sup> Group B, treatment.</td></tr>
</tfoot>
</table>
<br>

We can see a clear bias in sampling with group B compared to population has: 

  - higher proportion of older males (<35 years)
  - lower proportion of females

**Since there are clear biases in our samples, we can not directly compare the means of hours watched between the A/B group. However, we can analyse the difference between each demographic group.**

## Comparison of demogrphic 1 in A/B testing groups

Before we can compare the means of the outcome variable between groups A and B, we will need to assign two dataframes to represent the groups. 

*Note: group A is the control group and group B represents the treated group.*


```r
# assign hours_watched for demographic group 1 to vector - group A, control
d1_A2 <- A2_df %>% 
  filter(demographic == "1") %>% 
  pull(hours_watched)

# assign hours_watched for demographic group 1 to vector - group B, treated
d1_B <- B_df %>% 
  filter(demographic == "1") %>% 
  pull(hours_watched)

# sd for population hours watched
SD <- sd(A_df$hours_watched)

# mean for demographic 1 in group A, control
mu_d1a <- mean(d1_A2)

# mean for demographic 1 in group B, treated
mu_d1b <- mean(d1_B)

# sd for demographic 1 in group B, treated 
sd_d1b <- sd(d1_B)
```


To compare the means of group A and B, we can use a two-sided, two-sample t-test:


```r
# minimum sample size - demographic group 1
nss_d1 <-  ceiling( ( (1.96 * SD) / (mu_d1b - mu_d1a) )^2 )
paste('Min sample size needed for statistically valid results:', nss_d1)
```

```
[1] "Min sample size needed for statistically valid results: 23"
```

```r
paste("Group B sample size: 13")
```

```
[1] "Group B sample size: 13"
```

```r
paste("Sample size is NOT sufficient for statistically valid results")
```

```
[1] "Sample size is NOT sufficient for statistically valid results"
```

```r
# two-sample t-test for difference in means
t.test(d1_B, d1_A2, alternative = "two.sided", var.equal = TRUE)
```

```

	Two Sample t-test

data:  d1_B and d1_A2
t = 1.8527, df = 85, p-value = 0.06739
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -0.03993872  1.13178903
sample estimates:
mean of x mean of y 
 5.745385  5.199459 
```
  
The result of the two-sided, two-sample t-test in demographic 1 between group B and A demonstrate:

- $t(df = 85) = 1.85$, $p>0.05$
- Difference in mean = **0.546 hours (32 minutes)** increase in group B
- 95% CI for difference in means $95\%CI[-0.040, 1.318]$
- 95% CI captures difference in means
- Weak statistical evidence of difference in average hours watched between the groups
- Sample size of 13 in group B indicates that the results of this test is **not** statistically valid (minimum sample size = 23)
- More samples are required to understand and make valid conclusions for demographic group 1 (young females <35yrs)

## Comparison of demogrphic 2 in A/B testing groups


```r
# assign hours_watched for demographic group 1 to vector - group A, control
d2_A2 <- A2_df %>% 
  filter(demographic == "2") %>% 
  pull(hours_watched)

# assign hours_watched for demographic group 1 to vector - group B, treated
d2_B <- B_df %>% 
  filter(demographic == "2") %>% 
  pull(hours_watched)

# mean for demographic 2 in group A, control
mu_d2a <- mean(d2_A2)

# mean for demographic 2 in group B, treated
mu_d2b <- mean(d2_B)

# sd for demographic 2 in group B, treated 
sd_d2b <- sd(d2_B)

#minimum sample size
nss_d2 <-  ceiling( ( (1.96 * SD) / (mu_d2b - mu_d2a) )^2 )
paste('Min sample size needed for statistically valid results:', nss_d2)
```

```
[1] "Min sample size needed for statistically valid results: 20"
```

```r
paste("Group B sample size: 32")
```

```
[1] "Group B sample size: 32"
```

```r
paste("Sample size is sufficient for statistically valid results")
```

```
[1] "Sample size is sufficient for statistically valid results"
```

```r
# t-test for difference in means
t.test(d2_B, d2_A2, alternative = "two.sided", var.equal = TRUE)
```

```

	Two Sample t-test

data:  d2_B and d2_A2
t = 2.442, df = 126, p-value = 0.01599
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 0.1128256 1.0771744
sample estimates:
mean of x mean of y 
  5.70625   5.11125 
```

The result of the two-sided, two-sample t-test in demographic 2 between group B and A demonstrate:

- $t(df = 126) = 2.442$, $p<0.05$
- Difference in mean = **0.595 hours (35 minutes)** increase in group B  
- 95% CI for difference in means $95\%CI[0.113, 1.077]$
- 95% CI captures difference in means
- Sample size of 32 is sufficient for valid results
- Strong evidence recommendation engine changes has had a significant effect in increasing hours watched in younger male users (age <35) 

## Comparison of demogrphic 3 in A/B testing groups  


```r
# assign hours_watched for demographic group 1 to vector - group A, control
d3_A2 <- A2_df %>% 
  filter(demographic == "3") %>% 
  pull(hours_watched)

# assign hours_watched for demographic group 1 to vector - group B, treated
d3_B <- B_df %>% 
  filter(demographic == "3") %>% 
  pull(hours_watched)

# mean for demographic 3 in group A, control
mu_d3a <- mean(d3_A2)

# mean for demographic 3 in group B, treated
mu_d3b <- mean(d3_B)

# sd for demographic 3 in group B, treated 
sd_d3b <- sd(d3_B)

#minimum sample size
nss_d3 <-  ceiling( ( (1.96 * SD) / (mu_d3b - mu_d3a) )^2 )
paste('Min sample size needed for statistically valid results:', nss_d3)
```

```
[1] "Min sample size needed for statistically valid results: 16"
```

```r
paste("Group B sample size: 16")
```

```
[1] "Group B sample size: 16"
```

```r
paste("Sample size 16 is sufficient for statistically valid results")
```

```
[1] "Sample size 16 is sufficient for statistically valid results"
```

```r
# two-sample t-test for difference in means
t.test(d3_B, d3_A2, alternative = "two.sided", var.equal = TRUE)
```

```

	Two Sample t-test

data:  d3_B and d3_A2
t = 1.916, df = 103, p-value = 0.05814
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 -0.02321566  1.34536453
sample estimates:
mean of x mean of y 
 4.220625  3.559551 
```

The result of the two-sided, two-sample t-test in demographic 3 between group B and A demonstrate:
  
- $t(df = 103) = 1.916$, $p>0.05$ 
- difference in means = **0.661 (39 minutes)** increase in group B
- 95%CI for difference in means $95\%CI[-0.023, 1.345]$
- 95%CI captures difference in means
- Sample size of 16 is sufficient for valid results
- Weak statistical evidence recommendation engine changes has had a significant effect in increasing hours watched in older female users (age >35)
- a longer campaign will useful to confirm the outcome of effects on this sub-group

## Comparison of demogrphic 4 in A/B testing groups  


```r
# assign hours_watched for demographic group 1 to vector - group A, control
d4_A2 <- A2_df %>% 
  filter(demographic == "4") %>% 
  pull(hours_watched)

# assign hours_watched for demographic group 1 to vector - group B, treated
d4_B <- B_df %>% 
  filter(demographic == "4") %>% 
  pull(hours_watched)

# mean for demographic 3 in group A, control
mu_d4a <- mean(d4_A2)

# mean for demographic 3 in group B, treated
mu_d4b <- mean(d4_B)

# sd for demographic 3 in group B, treated 
sd_d4b <- sd(d4_B)

#minimum sample size
nss_d4 <-  ceiling( ( (1.96 * SD) / (mu_d4b - mu_d4a) )^2 )
paste('Min sample size needed for statistically valid results:', nss_d4)
```

```
[1] "Min sample size needed for statistically valid results: 20"
```

```r
paste("Group B sample size: 16")
```

```
[1] "Group B sample size: 16"
```

```r
paste("Sample size 59 is sufficient for statistically valid results")
```

```
[1] "Sample size 59 is sufficient for statistically valid results"
```

```r
# two-sample t-test for difference in means
t.test(d4_B, d4_A2, alternative = "two.sided", var.equal = TRUE)
```

```

	Two Sample t-test

data:  d4_B and d4_A2
t = 3.0813, df = 130, p-value = 0.002516
alternative hypothesis: true difference in means is not equal to 0
95 percent confidence interval:
 0.2118074 0.9716637
sample estimates:
mean of x mean of y 
 4.279407  3.687671 
```

The result of the t-test in demographic 4 between group B and A demonstrate:
  
- $t(df = 130) = 3.08$, $p<0.05$
- Difference in mean = **0.592 hours (35 minutes)** increase in group B
- 95%CI for the difference in mean $95\%CI[0.212, 0.972]$
- Sample size of 59 is sufficient for valid results
- Strong evidence recommendation engine changes has had a significant effect in increasing hours watched in younger male users (age >35) 


## Conclusion

There was a clear sampling bias with gender and age imbalance across the treatment group. This limits the generalisability of our findings. As a result, we were not able to directly compare the average hours watched between treatment and control group. However, we were able to compare 3 out of the 4 demographic sub-groups which met the required minimum sample size to draw statistically valid conclusions. We did not have enough samples to make valid conclusions for demographic 1 in treatment group B.

A/B testing investigation of the effects of refined recommendation engine on user sub-groups shows:

- demographic group 2 - **young males** (age <35) had an average **increase of 35 minutes** in hours watched per user 
- demographic group 3 - **older females** (age>35) had weak evidence for an average **increase of 40 minutes** in hours watched per user 
- demographic group 4 - **older males** (age >35) had an average **increase of 35 minutes** in hours watched per user with the new recommendation algorithm
  
Simple linear and multi regression analysis found:

- best predictors for hours watched were **age** and **social metric** scores 
- Age was shown to have a negative relationship with hours watched 
- social metric had a positive relationship

**Overall, we do recommend to go ahead with the roll-out of the new recommendation engine algorithm to our subscribers.**

<br>

#### Limitations:
  
- sample biases
- Small sample size for demographic group 1
- Limited time frame of the campaign (further investigations needed to assess seasonal effects on viewing hours)

<br>

#### Future direction:

- Collect more samples 
- Longer campaign period for more balanced results 
- Target users based on age and social metric

## References

- Wild, CJ, Pfannkuch, M, Regan, M, Horton, NJ 2011, 'Towards more accessible conceptions of statistical inference', *Journal of the Royal Statistical Society*, vol. 174, pp. 247-295. [URL]<http://doi.org/10.1111/j.1467-985X.2010.00678.x>

- R Core Team 2020, *R: A Language and Environment for Statistical Computing*, R Foundation for Statistical Computing, Vienna, Austria, viewed 2 May 2021, [URL]<http://www.R-project.org/>.

- Wickham, H & Hester, J 2020, *readr: Read Rectangular Text Data*, R package version 1.4.0, viewed 2 May 2021, [URL]<http://CRAN.R-project.org/package=readr>.

- Wickham, H et al. 2020, *dplyr: A Grammar of Data Manipulation*, R package version 1.0.2, viewed 2 May 2021, [URL]<http://CRAN.R-project.org/package=dplyr>.

- Xie, Y 2020, *knitr: A General-Purpose Package for Dynamic Report Generation in R*, R package version 1.30, viewed 2 May 2021, [URL] <http://cran.r-project.org/web/packages/knitr/index.html>.

- Wickham, H 2016, *ggplot2: Elegant Graphics for Data Analysis*, Springer-Verlag New York, viewed 2 May 2021, [URL]<http://ggplot2.tidyverse.org>.

- Spinu, V et.al 2020, *lubridate: Make Dealing with Dates a Little Easier*, R package version 1.7.9.2, viewed 2 May 2021, [URL]<http://cloud.r-project.org/web/packages/lubridate/index.html>.

- Wickham, 2020, *tidyr: Tidy Messy Data*, R package version 1.1.2, viewed 2 May 2021, [URL]<http://CRAN.R-project.org/package=tidyr>.

- Zhu, H 2021, *kableExtra: Construct Complex Table with 'kable' and Pipe Syntax*, R package version 1.3.4, viewed 2 May 2021, [URL]<http://CRAN.R-project.org/package=kableExtra>

- Auguie, B 2017, *gridExtra: Miscellaneous Functions for "Grid" Graphics*, R package version 2.3, [URL]<http://CRAN.R-project.org/package=gridExtra>

- Tang, Y, Horikoshi, M and Li, W, 2016, *ggfortify: Unified Interface to Visualize Statistical Result of Popular R Packages*, The R Journal 8.2: 478-489.

- Fox, J and Weisberg, S 2019, *An R Companion to Applied Regression*, 3rd edn, Thousand Oaks CA, Sage, viewed 2 May 2021, [URL]<http://socialsciences.mcmaster.ca/jfox/Books/Companion/>

- Tukey, JW 1977, *Exploratory Data Analysis*, Pearson, London, UK.

- Bruce, P, Bruce A, Gedeck P, *Practical Statistics for Data Scientists, 2nd edn, O'Reilly Media, Sebastopol, CA, USA.

- Rhys, H 2020, *Machine Learning with R, the tidyverse, and mlr*, Manning Publications, Shelter Island, NY, USA.
