##### From Codecademy skill path: Analyze Data With R

# Data Cleaning in R

A huge part of data science involves acquiring raw data and getting it into a form ready for analysis. Some have estimated that data scientists spend 80% of their time cleaning and manipulating data, and only 20% of their time actually analyzing it or building models from it.

When we receive raw data, we have to do a number of things before we’re ready to analyze it, possibly including:

- diagnosing the “tidiness” of the data — how much data cleaning we will have to do;
- reshaping the data — getting the right rows and columns for effective analysis;
- combining multiple files;
- changing the types of values — how we fix a column where numerical values are stored as strings;
- dropping or filling missing values - how we deal with data that is incomplete or missing; and
- manipulating strings to represent the data better.

### Diagnose the Data

We often describe data that is easy to analyze and visualize as “tidy data”. What does it mean to have tidy data?

For data to be tidy, it must have:
- each variable as a separate column, and
- each row as a separate observation.

### Dealing with Multiple Files

Often, you have the same data separated out into multiple files.

Let’s say that you have a ton of files following the filename structure: `'file_1.csv'`, `'file_2.csv'`, `'file_3.csv'`, and so on. The power of dplyr and tidyr is mainly in being able to manipulate large amounts of structured data, so you want to be able to get all of the relevant information into one table so that you can analyze the aggregate data.

You can combine the base R functions `list.files()` and `lapply()` with readr and dplyr to organize this data better, as shown below:

```r
files <- list.files(pattern = "file_.*csv")
df_list <- lapply(files,read_csv)
df <- bind_rows(df_list)
```

- The first line uses `list.files()` and a regular expression, a sequence of characters describing a pattern of text that should be matched, to find any file in the current directory that starts with `'file_'` and has an extension of `csv`, storing the name of each file in a vector `files`.
- The second line uses `lapply()` to read each file in `files` into a data frame with `read_csv()`, storing the data frames in `df_list`.
- The third line then concatenates all of those data frames together with dplyr’s `bind_rows()` function.

### Reshaping your Data

Since we want
- each variable as a separate column, and
- each row as a separate observation

We would want to reshape a table like:

Account |	Checking |Savings
--- | --- | ---
“12456543” | 8500 | 8900
“12283942” | 6410 | 8020
“12839485” | 78000 | 92000

into a table that looks more like:

Account | Account Type | Amount
--- | --- | ---
“12456543” | “Checking”	8500
“12456543” | “Savings” | 8900
“12283942” | “Checking” | 6410
“12283942” | “Savings” | 8020
“12839485” | “Checking” | 78000
“12839485” | “Savings” | 920000

We can use tidyr’s `gather()` function to do this transformation. `gather()` takes a data frame and the columns to unpack:

```r
df %>%
  gather('Checking','Savings',key='Account Type',value='Amount')
```

The arguments you provide are:
- `df`: the data frame you want to gather, which can be piped into `gather()`
- `Checking` and `Savings`: the columns of the old data frame that you want to turn into variables
- `key`: what to call the column of the new data frame that stores the variables
- `value`: what to call the column of the new data frame that stores the values

### Dealing with Duplicates

Often we see duplicated rows of data in the data frames we are working with. This could happen due to errors in data collection or in saving and loading the data.

To check for duplicates, we can use the base R function `duplicated()`, which will return a logical vector telling us which rows are duplicate rows.

We can use the dplyr `distinct()` function to remove all rows of a data frame that are duplicates of another row.

If we wanted to remove every row with a duplicate value in the item column, we could specify a `subset`:

```r
fruits %>%
  distinct(item,.keep_all=TRUE)
```

By default, this keeps the first occurrence of the duplicate.

### Splitting by Index

In trying to get clean data, we want to make sure each column represents one type of measurement. Often, multiple measurements are recorded in the same column, and we want to separate these out so that we can do individual analysis on each variable.

Let’s say we have a column “birthday” with data formatted in MMDDYYYY format. In other words, “11011993” represents a birthday of November 1, 1993. We want to split this data into day, month, and year so that we can use these columns as separate features.

In this case, we know the exact structure of these strings. The first two characters will always correspond to the month, the second two to the day, and the rest of the string will always correspond to year. We can easily break the data into three separate columns by splitting the strings into substrings using `str_sub()`, a helpful function from the stringr package:

```r
# Create the 'month' column
df %>%
  mutate(month = str_sub(birthday,1,2))

# Create the 'day' column
df %>%
  mutate(day = str_sub(birthday,3,4))

# Create the 'year' column
df %>%
  mutate(year = str_sub(birthday,5))
```

- The first command takes the characters starting at index `1` and ending at index `2` of each value in the `birthday` column and puts it into a `month` column.
- The second command takes the characters starting at index `3` and ending at index `4` of each value in the `birthday` column and puts it into a `day` column.
- The third command takes the characters starting at index `5` and ending at the end of the value in the `birthday` column and puts it into a `year` column.

### Splitting by Character

Let’s say we have a column called "type" with data entries in the format "admin_US" or "user_Kenya", as shown in the table below.

id	| type
--- | ---
1011	| “user_Kenya”
1112	| “admin_US”
1113	| “moderator_UK”

Just like we saw before, this column actually contains two types of data. One seems to be the user type (with values like “admin” or “user”) and one seems to be the country this user is in (with values like “US” or “Kenya”).

We can no longer just split along the first 4 characters because admin and user are of different lengths. Instead, we know that we want to split along the `"\_"`. We can thus use the tidyr function `separate()` to split this column into two, separate columns:

```r
# Create the 'user_type' and 'country' columns
df %>%
  separate(type,c('user_type','country'),'_')
```

- `type` is the column to split
- `c('user_type','country')` is a vector with the names of the two new columns
- `'\_'` is the character to split on

### Looking at Data Types

Each column of a data frame can hold items of the same data type. The data types that R uses are: character, numeric (real or decimal), integer, logical, or complex. Often, we want to convert between types so that we can do better analysis. If a numerical category like `"num_users"` is stored as a vector of `character`s instead of `numeric`s, for example, it makes it more difficult to do something like make a line graph of users over time.

To see the types of each column of a data frame, we can use `str()`, which displays the internal structure of an R object. Calling `str()` with a data frame as an argument will return a variety of information, including the data types.

### String Parsing

Sometimes we need to modify strings in our data frames to help us transform them into more meaningful metrics. For example, consider the following table:

item | price	| calories
--- | --- | ---
“banana”	| “$1”	| 105
“apple”	| “$0.75”	| 95
“peach”	| “$3”	| 55
“peach”	| “$4”	| 55
“clementine” | “$2.5” |	35

We can see that the `'price'` column is actually composed of character strings representing dollar amounts. This column could be much better represented as numeric, so that we could take the mean, calculate other aggregate statistics, or compare different fruits to one another in terms of price.

First, we can use a regular expression, a sequence of characters that describe a pattern of text to be matched, to remove all of the dollar signs. The base R function `gsub()` will remove the `$` from the `price` column, replacing the symbol with an empty string `''`:

```r
fruit %>%
  mutate(price=gsub('\\$','',price))
Then, we can use the base R function as.numeric() to convert character strings containing numerical values to numeric:

fruit %>%
  mutate(price = as.numeric(price))
```
