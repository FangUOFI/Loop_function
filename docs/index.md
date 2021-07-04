---
title: "Advanced Data Science for planners"
subtitle: Lecture 3 Loops and functions
author: "Fang Fang"
knit: (function(input_file, encoding) {
  out_dir <- 'docs';
  rmarkdown::render(input_file,
 encoding=encoding,
 output_file=file.path(dirname(input_file), out_dir, 'index.html'))})
output: 
  html_document: 
    keep_md: yes

---

Objectives

- Make decisions with if, else, while, repeat statements;
- Understand conditional execution in R;  
- Write, define and apply new functions in R;
- Use loop function: lapply, sapply, mapply, tapply, map, etc;

I would recommend you to create a new script called lecture3.r under the folder (week3) for this lecture to try out the scripts we introduced below. 

# Preps


![ ](images/procedure.jpg)


When we first load the dataset in R, if the data is messy and full of errors, it is hard for us to proceed to analysis. However, most of the datasets are not clean at the beginning.  Below is the preview of features for clean datasets:


![ ](images/data.png)

- Each variable must have its own column.
- Each observation must have its own row.
- Each value must have its own cell.


From R for Data Science. Click [here](https://r4ds.had.co.nz/tidy-data.html) for more information. 

Here are a few things you need to do when get started with a dataset. 

- Identifying Data Types (i.e. convert chr into num if necessary)
- Data Entry Errors (Upper/lower case, abbreviations...etc)
- Missing Values 


Let's take a look at a dataset to demonstrate the data examination process. This landings_data data frame is from a fishery-dependent landing site survey. The included species is Caesio cuning. Click [here](https://raw.githubusercontent.com/SFG-UCSB/fishery-manageR/master/_data/sample_landings_data_raw.csv) to download the dataset. Click [here](https://sfg-ucsb.github.io/fishery-manageR/loading-data-and-data-cleaning.html) for more information about this data clean up process. 


Let's load the dataset in R first. 

```r
library(tidyverse)
landings_data <- read_csv("data/sample_landings_data_raw.csv")
glimpse(landings_data)
```


We have 7214 rows and 8 columns. You will realize that 1) the column names are not straightforwad enough; 2) the data type for the second column is wrong. 3) convert `gear` as factor.  


```r
 library(lubridate)

 # Rename the columns
landings_data <-   rename(landings_data,Year = yy,
         Date = dat,
         Trip_ID = trip,
         Effort_Hours = effort,
         Gear = gr,
         Species = sp,
         Length_cm = l_cm,
         Weight_g = w_cm)

# Convert chr into date format
landings_data$Date <- mdy(landings_data$Date)

unique(landings_data$Gear)
```

You might also noticed that there are typos in the `Gear` column. Let's try to convert these characters into lower case. Then we convert them into factors. 


```r
landings_data$Gear <- tolower(landings_data$Gear)

landings_data$Gear <- as.factor(landings_data$Gear)
```


How about the Species column? Any typos?


```r
unique(landings_data$Species)
```

So we need to fix these typos. We can just replace the errors by "Caesio cuning". 

```r
landings_data$Species <- replace(landings_data$Species,
      landings_data$Species == "Caesoi cunning", "Caesio cuning")
```


How about the length_cm column? Let's use five number summary to quickly identify if there are any extreme values. 

```r
fivenum(landings_data$Length_cm)
```

So the maximum value is abnormal. Based on the domain knowledge, the size of our species is around 100cm. So we can preserve the records which is less and equal to 100 cm for Length. 

```r
landings_data<- landings_data[landings_data$Length_cm<101,]
```

You should have 7211 rows instead of 7214. So 3 extreme values are removed. 

Next, let's check out missing values by rows.  This `complete.cases` function will return a logical vector indicating which record is complete. So we take the opposite to subset the dataset with missing values and take a look at these incomplete records. 


```r
landings_data[!complete.cases(landings_data),]
```

You should get 3 rows with missing values. We can omit these rows by running:

```r
landings_data_clean <- na.omit(landings_data)
```



# 1. Loops
If you google what are loops, it will tell you that "loops are used in programming to repeat a specific block of code". Let's start this concept with an example. Below is the weekly scheduel for Tom. Tom has to take Physics every Mon/Wed/Fri at 10 am. In other words, Tom has to repeat the action (learn Physics) only if it is Mon/Wed/Fri (conditions) of the week. 

![ ](images/class.jpg)

Conceptually, we can code Tom's schedul below:
Note DON'T execute the script below. 

```r
if (Mon/Wed/Fri)

    Take Math 101/ Biology / Physics 101

if (Tue/Thur)

    Take English 101

if (Sun/Sat)

    Rest
```

This is a typical loop in R, as far as the conditions are met. In this lecture, I will introduce your a few commands to control flows in R. 

In general, there are two types of commands to efficiently control the flow of execution of a series of R expressions in R.

1) The number of the iteration is given. You need to repeat the action which is prescribed by certain number of times, or so called limit the action by a counter or an index. This index will be incremented at each iteration (e.g. print "Tom" repeatly for ten times, ten is predefined). For example, `for` function in R.

2) It is hard to know how many times you need to repeat the action. But you can start and end a loop via the onset and verification of a given logical condition (Recall what are logical variables we mentioned last week). The logical condition is tested at the start or the end of the loop.You can decide to stop/continue/exit the loop based on the conditoin of the logical condition (e.g. print "excellent" as far as the student get 90+ for the exam). For example, `while` command is one of the family of loop function in this category.  

In this handout, we will introduce these commands for flow control with examples. Please run these examples in R yourself. 

1) `if`, `else`: test a condition; if true, execute action 1. if not true, execute action 2  under `else.` 
2) `for`: execute a loop (execute action) within a given number of times
3) `while`: execute a loop as far as while condition is true
4) `repeat`: execute an *infinite* loop
5) `break`: exit the entire loop, when the condition is met
6) `next`: jump to the next iteration, when the condition is met.

Let's go over some examples for each of the functions. 

1) `if`, `else`:

As we the example we mentioned above, let's print out Tom's schedule for 2020-09-07 below. Note you need to put the condition with `()` after if, and the action beneath it with a pair of `{}`. We use the function called `wday()` from `lubridate` package to extract the weekday of 2020-09-07.  


```r
library(lubridate)
the_date <- ymd(c("2020-09-07"))
weekday <- wday(the_date)
# please check out the value of weekday before
# you execute the if statement below. 

if (weekday %in% c(2,4,6)){  # if it is Mon/Wed/Fri. 
    # `%in%` identifies if the weekday belongs to a vector (2,4,6).
        print("Math, Biology, Physics")
}  else   if ( weekday %in% c(3,5)) {  # if it is Tue/Thur
        print("English")
}   else  { # anything else, should be weekend
        print("Weekend")
}
```

2) `for` loops

If you want to repeat an action over a vector, you need for loops. For example, if we want to count the number of odd numbers for a given vector, you can use the for loops. Note here we also bring in the if else statement.


```r
# Create a new vector
x <- c(2,15,3,98,28,55,6,100)
# Initiate number of odd numbers as 0
count <- 0
# for each element (we called them val temporally) within x
for (val in x) {
if(val %% 2 == 1)  #take the remainder, if it is 1, it will be an odd number. 
    count = count+1 
}
print(count)
```




3) `While` function

In R programming, the `while` action will be executed as far as a specific condition is met.It will stop only if the condition is false.
You need to use while loop with caution to avoid infinite loops. Note you need to put the condition with `()` after while, and the action beneath it with a pair of `{}`.

For example, if we want to print the day of the week, we can use while to control this process. As far as the day is less than 8, it will be printed. 


```r
day <- 1 
while (day <8 ) {
    print (day)
    day <- day +1
}
```

4) repeat and break statment

We can also use repeat and break to print the day of the week. We will repeat the action (print the day of the week from number 1), after each action day will be incremented by 1. The action will stop when the value is 8. Break statement will end the loop immediately. 

```r
day <- 1
repeat{
    print(day)
    day <- day+1
    if (day == 8){
        break
    }
}
```


5) next

Another commonly used statement with if statement is next. The next statement will skip the current iteration inside the loop if the condition is met, and move to the next item directly. The action will be executed when the condition is not met. 
For example, the script below only print out numbers which is greater than 5 among 1-20. 

```r
for (i in 1:20){
    if (i<5){
        next   
# if i<5, go to next element 
    }
    print (i)
}
```

Try out to use the break function instead of next. What will happen this time? Try to understand the results.


```r
for (i in 1:20){
    if (i<5){
        break   
# if i<5, stop the loop.  
    }
    print (i)
}
```


# 2. The map functions 

Click [here](https://r4ds.had.co.nz/iteration.html) to get more information from the textbook: R for data science chapter 21 interation.

Let's try to execute the commands below first. 


```r
# Generate a vector includes the value from 1 to 10. 
vector_example <- 1:10
# Try to add 1 to the vector you just created
vector_example+1
```

You might already noticed that we add 1 to each of the element in the vector. This is the vectorized process, which is every efficient in R to avoid establish loops. 

Ideally, a dataset should be well organized as the screenshot below. 

![ ](images/data.png)

R also provides a set of *apply functions to execute a certain function over a vector/list. 

- `lapply`: loop over a list and execute a function on each of the items (a list will be returned)
- `sapply`: loop over a list and execute a function with simplified results (a vector will be returned)
- `tapply`: Apply a function over subsets of the input vector. 


```r
# Create a new list
x <- list(a = 1:4, b = c(2,4,6,7,20), c = 1:20) # Calculate average for each item in the list. 
# The result should be a list too. 
lapply(x,mean)
sapply(x, mean)
```

You can use `tapply` as a short cut when you are working with dataframes. 

tapply will apply a function to each item of a ragged array, that is to each group of the values. The group is determined by a certain column. The basic template is:

```r
# Please DO NOT execute the script below. 
# This is a template only
tapply(column to be calculated,column which specifying the groups, function/action...)
```
Load the dataset called `weather` from nycflights13 package. It includes hourly measured meterological data for LGA, JFK and EWR airports in NYC, 2013.  

If we want to calculate the average temperature (based on the temp column) for each month (column called month), we can use the `tapply` function. 



```r
library(nycflights13)
data(weather)
tapply(weather$temp,weather$month,mean,na.rm=T)
```


There is a package called `purrr` in R which provides a family of functions to repeat certain actions over each item of a list/vector. Feel free to use these funcitons instead of writing for loops if needed. Make sure the package is installed before you move forward. 

Click [here](https://github.com/rstudio/cheatsheets/blob/master/purrr.pdf) to view the cheat sheet of functions in `purrr` package.


The `map` function applies the same action/function to each element of the input (e.g. each item of a list, or each column in a data frame).

```r
library(purrr)
# Identify the data type for each column in the weather dataset. 
map(weather,class)
```

Alternatively, you can change your output in a more elegant way. Let's store the results in a dataframe using the map_df function. At the same time, we use `.id` to preserve the column names for the output. Try out the commands below. 



```r
map_df(weather, ~(data.frame(class = class(.x))),.id = "variable")
```

For more information of map functions in `purrr` package, click [here](https://r4ds.had.co.nz/iteration.html#the-map-functions) in our textbook R for data science. 

# 3. Functions
Click [here](https://r4ds.had.co.nz/functions.html) to get more information from the textbook: R for data science chapter 19 functions.

We already learned lots of different R functions till now. For example, `mean(x)` function will return you the average of a given object x, where we call `x` as the argument. 

Sometimes you have to finish a particular task, but cannot a dedicated function or an existing library. You can create your own functions in order to automate these certain commands and tasks in a more powerful and elegant way instead of repeating the same scripts. We called them user defined functions. The basic template is: please DO NOT execute this script below. It is a template only. When naming your function, try to avoid an existing function name for a user defined function. 


```r
# Name your function properly. Put the arguments after function. 
function.name <- function(arguments) 
{
  computations on the arguments
  some other code
}
```

After you have defined the new function in a function definition above, you can use it later somewhere else in your script.

Let's take a look at this example below. We will generate a new function, which converts temperatures from Fahrenheit (F) to Celsius (C). We will name our function as "F_to_C", and the input argument is temperature in Fahrenheit, and we put them in `()`. The body of the function should be placed within the `{}`. 

Execute the script below in your R window. 

```r
F_to_C <- function(temp_F) {
  temp_C <- (temp_F - 32) * 5 / 9
  return(temp_C) # return: send the results back after executing the function
}
```

Note it is not required (but highly recommended) to include the return statement. R will automatically returns the variable on the last line of the script body in the function. 


Now you can use this F_to_C function to convert temperature in Fahrenheit into Celsius. For example, let's turn 90 F into Celsius. You can also convert temperature(F) in the `weather` datasets into Celsius. 


```r
F_to_C(90)
F_to_C(weather$temp)
```

In fact you can create an anonymous function if no names are provided. But the structure is a little different from the user defined function we introduced above. The anonymous function is recommended for simple calculation, or it will not be needed anywhere else in your script. 


```r
(function(temp_F) (temp_F - 32) * 5 / 9)(90)
```


Let's take a look at another application of user generated functions. Please download the dataset from compass called "CPI_U.csv" and load it into R. This table is originally derived from [US census Bureau](https://www.census.gov/topics/income-poverty/income/guidance/current-vs-constant-dollars.html). 

When we need to compare median household income over different years, we need to adjust the median household income value for changes in cost of living (prices). Here we will use the Bureau of Labor Statistics' (BLS) Consumer Price Index Research Series (CPI-U-RS) to adjust for changes in the cost of living.This table includes the Annual Average Consumer Price Index Research Series (CPI-U-RS) from 1947 to 2018. 

Please check out the example for adjusting income estimates below. You can also click [here](https://www.census.gov/topics/income-poverty/income/guidance/current-vs-constant-dollars.html) for more info. 
![ ](images/census.jpg)

Let's establish a new function called "Inflation_fct", which takes two years as the input argument, and return the inflation factor as the output. So in the future we can directly use this function to adjust income from multiple years. Here we set year_1 as the earlier time period , and year_2 as the latter time period. If year_1 or year_2 is out of the range of 1947 to 2018, we will return an error message. 


```r
library(tidyverse)
index <- read_csv("data/CPI_U.csv")

Inflation_fct <- function(year_1,year_2) {
  if (year_1<year_2 & year_1>1946 & year_2 < 2019)
  {
    index_1 <- index[index$Year==year_1,]$"CPI-U-RS"
    index_2 <- index[index$Year==year_2,]$"CPI-U-RS"
    index_2/index_1
  }
  else 
  {
    print("Error, please double check the time period for two years")
  }
}


Inflation_fct(1999,2015)
```

Try to execute the commands below. What message you get this time?

```r
Inflation_fct(1997,2020)
```











