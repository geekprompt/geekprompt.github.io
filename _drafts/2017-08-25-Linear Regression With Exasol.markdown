---
layout: post
title:  "Linear Regression with EXASolution"
date:   2017-08-25 00:34:23 +0530
categories: machine learning, database
comments: true
---

EXASolution is an in-memory relational database management system. It's a
product of [EXASOL](http://www.exasol.com/){:target="_blank"}. If you are asking yourself the question "Why EXASOL?" then may be [this link](http://www.exasol.com/site/assets/files/1883/5_top_reasons.pdf){:target="_blank"}
might be able to satisfy your curiosity. We will move on to the steps to execute
 Linear Regression operation on EXASolution.

EXASolution allows us to write UDF (user defined function) in any of the
following languages:

- R
- Python
- Java
- Lua

Using this feature we can leverage machine learning capabilities of R.

**Prerequisites**:

- _EXASolution_

  You can download EXASolution free version VM from [here](https://www.exasol.com/portal/display/DOWNLOAD/Free+Trial) and start the VM to bring on the EXASolution server. Once the VM is started, you can
  see all necessary information to connect to the EXASolution server on the VM
  screen.

- _EXAPlus_

  EXAplus provides user interface to manage your EXASolution database.
  EXAPlus can be downloaded from [this link](https://www.exasol.com/portal/display/DOWNLOAD/6.0).

- _R_

  You can download R for windows from [here](https://cran.r-project.org/).
  R is a statistical language which bring machine learning capabilities.
- _RStudio_

  RStudio is an IDE for R. Free version of RStudio can be downloaded from [here](https://www.rstudio.com/products/rstudio/download/).

  Note that this is an optional step. You can run R scripts directly from R
  console too.

- _r-exasol package_

  For windows following steps needs to be followed for this:

  - Install RTools. Can be downloaded from [here](http://mirror.fcaglp.unlp.edu.ar/CRAN/bin/windows/Rtools/).
    RTools is R's developer extension which allows you to build R package from
    source. This is need to build `r-exasol` plugin from it's source.

  - Install `devtools` package for R which contains `install_github()` method
    which allow us to build `r-exasol` package from github sources.

    You need to fire following commands in your R console:

    {% highlight R %}
    package.install("devtools")
    devtools::install_github("EXASOL/r-exasol")
    {% endhighlight %}


  After successful execution of above mentioned commands, we can use `r-exasol`
  package in R.   

- _EXASOL ODBC Driver_

  This can be downloaded from [here](https://www.exasol.com/portal/display/DOWNLOAD/6.0){:target="_blank"}. We need ODBC Driver
  in order to fetch data to R from EXASolution.

**Dataset:**

For this tutorial we will be using healthcare dataset available [here](http://college.cengage.com/mathematics/brase/understandable_statistics/7e/students/datasets/mlr/frames/frame.html){:target="_blank"}.

The dataset contains following columns:

- X1 = death rate per 1000 residents
- X2 = doctor availability per 100,000 residents
- X3 = hospital availability per 100,000 residents
- X4 = annual per capita income in thousands of dollars
- X5 = population density people per square mile

We will use X2, X3, X4 and X5 as features to predict X1.

I have converted the dataset in CSV format and added an id column. It can be
downloaded from [here](https://drive.google.com/file/d/0B2AwCXFgWWjNWVNkTDY5UmtuVTg/view?usp=sharing){:target="_blank"}.

- **Importing the data into EXASolution:**

Save the downloaded data file as CSV file as EXASolution has supports data import
from CSV file.

Open EXAPlus client and connect to the EXASolution server. Again the details
required to establish the connection should be available on EXASolution VM screen.

Here is the script to import data in EXASolution:

{% highlight SQL %}
--create a new schema
CREATE SCHEMA ml;

--create table to store the input data
CREATE TABLE ml.input (
                          id decimal(5, 0) primary key,
                          X1 decimal(5, 2),
                          X2 decimal(5, 2),
                          X3 decimal(5, 2),
                          X4 decimal(5, 2),
                          X5 decimal(5, 2)
                      );

--load data into the table

IMPORT INTO ml.input FROM LOCAL CSV
  FILE '<local path of input file>'
  ENCODING = 'UTF-8'
  ROW SEPARATOR = 'CRLF'
  COLUMN SEPARATOR = ','
  SKIP = 1
  REJECT LIMIT 0;

--split input table in two tables for training and testing data
CREATE TABLE ml.training AS SELECT * FROM ml.input WHERE id <= 30;
CREATE TABLE ml.testing AS SELECT * FROM ml.input WHERE id > 30;

{% endhighlight %}

- **Configure EXABucket**

EXABucket is a distributed file-system which EXASolution can access. We need to
upload the generated regression model to EXABucket so that using it we can
perform predictions on EXASolution.

In order to upload the files to EXABucket, I had to manually set the bucket
service port and write user password from EXASolution admin panel.


- **Generate regression model and upload to EXABucket:**

Open RStudio or R terminal and execute following script (explanation inline):

{% highlight R %}
library(exasol)

#establish the connection using user name password
con <- dbConnect("exa", exahost = "<exasol url with port>", uid = "<user-name>", pwd = "<password>")
#or using dsn
#con <- dbConnect("exa", dsn="exa")

#load training data in R dataframe
train <- dbGetQuery(con, "SELECT X1, X2, X3, X4, X5 from ml.training")

#close the db connection
dbDisconnect(con)

#train model
lr_model <- lm(X1 ~ X2 + X3 + X4 + X5, data= train)

# upload the model to an EXABucket via httr
library(httr)
PUT(
	# EXABucket URL
	url = "<exabucket url>/lr_model",
	body = serialize(lr_model, ascii = FALSE, connection = NULL),
	# EXABucket: authenticate with write_user_name / write_user_password
	config = authenticate("w", "exasol")
)

{% endhighlight %}

*Note:* More details regarding EXABucket can be found [here](https://www.exasol.com/support/browse/SOL-503){:target="_blank"}.

- **Predict result on EXASolution:**

Now since we have regression model available on EXABucket, we can perform predictions by using it. For predictions we will write an UDF in R.

{% highlight SQL %}

-- create UDF
create or replace r set script ml.predict_lr(...) emits (predicted numeric, actual numeric, residual numeric) as
run <- function(ctx) {
  ctx$next_row(NA)

  ## read the model from bucket which was uploaded by the client side R script
  load("<exabucket root>/lr_model")

  ## create a dataframe from all input columns
  df <- data.frame(ctx[[1]]())
  for (i in 1:exa$meta$input_column_count) {
    df <- cbind(df,ctx[[i]]())
  }

  ## add columns names to dataframe
  colnames(df) <- c("id", "X1", "X2", "X3", "X4", "X5")

  ## predict and return prediction. Excluding id column and label column (X1) for prediction
  prediction <- predict(reg, df[,3:6])

  ctx$emit(prediction, df$X1, prediction-df$X1)
}
/

--predict the data
select ml.predict_lr(
  X2,
  X3,
  X4,
  X5
) from ml.testing;

{% endhighlight %}

_**References:**_

[r-exasol github](https://github.com/EXASOL/r-exasol){:target="_blank"}

[How to create an EXABucketFS service and bucket](https://www.exasol.com/support/browse/SOL-503){:target="_blank"}

[Deploying an R scoring model in EXASOL via UDF](https://www.exasol.com/support/browse/SOL-542){:target="_blank"}
