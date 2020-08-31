##### From Codecademy skill path

# Analyze Data With R

### Data Cleaning in R

A huge part of data science involves acquiring raw data and getting it into a form ready for analysis. Some have estimated that data scientists spend 80% of their time cleaning and manipulating data, and only 20% of their time actually analyzing it or building models from it.

When we receive raw data, we have to do a number of things before we’re ready to analyze it, possibly including:

- diagnosing the “tidiness” of the data — how much data cleaning we will have to do;
- reshaping the data — getting the right rows and columns for effective analysis;
- combining multiple files;
- changing the types of values — how we fix a column where numerical values are stored as strings;
- dropping or filling missing values - how we deal with data that is incomplete or missing; and
- manipulating strings to represent the data better.

#### Diagnose the Data

We often describe data that is easy to analyze and visualize as “tidy data”. What does it mean to have tidy data?

For data to be tidy, it must have:
- each variable as a separate column, and
- each row as a separate observation.

#### Dealing with Multiple Files

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
