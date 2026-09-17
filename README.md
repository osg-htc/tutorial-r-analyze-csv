---
ospool:
  path: software_examples/r/tutorial-spills-R/README.md
path:
  path: 
---

# Analyzing Multiple .csv Files with R
### <i>An OSPool Tutorial</i>

Spills of hazardous materials, like petroleum, mercury, and battery acid, that can impact water and land quality are required to be reported to the United State's government by law. In this tutorial, we will analyze records provided by the state of New York on occurances of spills of hazardous materials that occured from 1950 to 2019.

The data used in this tutorial was collected from https://catalog.data.gov/dataset/spill-incidents/resource/a8f9d3c8-c3fa-4ca1-a97a-55e55ca6f8c0 and modified for teaching purposes. 

## What we're trying to achieve

You are a researcher analyzing a dataset of hazardous material spills. On your own computer, your R script loops through a directory of data files to analyze each one, one at a time. 

```
# R code

list_of_datasets <- Sys.glob("data/*.csv")

for (datafile in list_of_datasets) {
    data <- read.csv(datafile)
    # do analysis
    }
```

Suppose the analysis for each datafile took 1 hour. Looping through all of them would take multiple hours. 

Our goal is to split apart this loop and run each analysis as its own job. First, we change our script to just analyze one dataset at a time, and we create a list of our data, just like the glob in the code above: 

```
# R code

datafile <- args[1]
# do analysis
```


```bash
ls data/ > list_of_datasets.csv
```


```bash
cat list_of_datasets.csv
```

With our changed code, and our list of files, we can use HTCondor to submit a job for each file in the list: 

```
## HTCondor submit file 

shell = Rscript spill_calculation.ospool.R $(dataset)

### other options

queue dataset from list_of_datasets.csv
```

Based on the `list_of_datasets.csv` file, how many jobs should be submitted? Let's submit them and see! 


```bash
condor_submit many-spills.submit
```


```bash
condor_q
```

Great! In a few minutes, all of your jobs should be done. Look at the outputs in the `output/` folder


```bash
cat output/*
```


If each analysis would take an hour (or multiple hours), submitting the list of jobs will have saved a lot of time. 

> Tip: While in this example we are using R as our programming language, and our data files are in a `.csv` format, the principles in this example can be applied to ANY research that involves looping through a list of files. 
> * If your research problem can be expressed as looping through a list of files -- what kind of files? What program are you using for analysis? 
> * If your research doesn't involving looping through a list of files, try to come up with 2-3 examples of people who might use this kind of workflow. 

Let's now go through the step by step process to go from running a local script on your computer, to using the OSPool. Delete the files created in our test to start fresh: 


```bash
rm log/* error/* output/* 
```

## Step by step to get on the OSPool

### Step 1: Changes to our R code

Look at the original R script and then compare with the one used to run jobs on the OSPool


```bash
cat spill_calculation.original.R
```


```bash
cat spill_calculation.ospool.R
```

<details>
    <summary>Differences</summary>
    The first script uses a "glob" to create a list of data files and then loops through the files one by one. The second script reads in a single file name from the command line, and only analyzes that file. 
</details>

One important feature we are using here is capturing arguments from the command line. On the OSPool, the R script will need to be run with a command like this: 

```
Rscript spill_calculation.ospool.R
```

In order to provide the names of different input files to the script, we are using R's `commandArgs()` function. This captures trailing arguments and allows us to use them inside the script. So if we run: 

```
Rscript spill_calculation.ospool.R spills_1950_1959.csv 
```

The `commandArgs()` function can capture the name of the file `spills_1950_1959.csv`, and we can then use it in our script. 

> Tip: `commandArgs()` is specific to R, but most scripting languages have a similar functionality! Python, for example, has an option called `sys.argv` that is very similar. If you work with scripting languages, it is worth figuring out how to add arguments to your scripts. 

### Step 2: Recreate our software environment with containers

Some common software, like R, are provided by OSG using containers. Because of this, you do not need to install R yourself, you will just tell HTCondor what container to use for your jobs. Additionally, this tutorial just uses base-R and no special libraries, but if you need libraries (e.g., tidyverse, ggplot2) you can always install them in your R container. 

A list of containers and other software provided by OSG staff can be found on our website [https://portal.osg-htc.org/documentation/](https://portal.osg-htc.org/documentation/), along with resources for learning how to add libraries to your container. 

We will be using the R container for R 3.5.0, which is accessable under `/cvmfs/singularity.opensciencegrid.org/opensciencegrid/osgvo-r:3.5.0`, so we must make sure to tell HTCondor to fetch this container when starting each of our jobs. To learn how to tell HTCondor to do this, see below. 

### Step 3: Upload data

In this case, our data was included when we cloned the repository. However, if you were working with your own data, it's more likely that you would need to upload it to an OSPool Access Point like ap40 or ap41. 

There are two different places to upload your data. You can learn more about them in this documentation page: [Data Staging and Transfer to Jobs](https://portal.osg-htc.org/documentation/htc_workloads/managing_data/overview/)

### Step 4: Create a submit file

The HTCondor submit file tells the HTCondor how you would like your job to be run on your behalf.

For example, you should specify what executable you want run, if you want a container/the name of that container, the resources you would like available to your job, and any special requirements. When starting out, it's a good idea to just run ONE job at once!! This submit file does just that: 


```bash
cat one-spill.submit
```

Note that this submit file doesn't loop at all. It just sets one dataset name, and submits one job. 


```bash
condor_submit one-spill.submit
```


```bash
condor_q
```

### Step 5: Review and scale up

Once our job is done running, we can check the results by looking in our `output` folder: 


```bash
cat output/*
```

We should see that from 1950-1959, New York recorded five spills that totalled less than 0 recorded gallons. 

We can also look at the end of the log file to see how many resources the job used: 


```bash
tail log/*.log
```

If the job ran successfully, and our resource requests are accurate, you can proceed with submitting the whole batch of jobs. Generate a list of inputs if you haven't already: 
```
ls data/*.csv > list_of_datasets.csv
```

Then, modify the submit file to use a "looping" queue syntax. 


```bash
tail -n 2 many-spills.submit
```

We've reached the completed example from the beginning: 


```bash
condor_submit many-spills.submit
```


```bash
condor_q
```

Once our jobs are done, we can also review our output files:


```bash
cat output/*.csv.out
```

In a few minutes, we were able to take our R script and run several jobs to analyze all of our real-world data. <i>Congratulations!</i>
