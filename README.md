# HPC Common Crawl Analytics

This project contains analytics designed to run on AWS clusters. 

## Running Jobs

### On a Cluster

* To build the jar, download sbt-1.0.2 and run `sbt assembly`
* Copy the jar to an Amazon S3 bucket
* Go to EMR and create cluster
* Use Spark software and applicable hardware configuration
* Run spark application (not custom .jar file) with spark submit option: --class edu.usma.cc.SimpleApp

### Current Problems:

* Running out of physical memory
* Running out of Java heap space memory
* Estimating cost of a job

* Attempt m4.xlarge for large job (3 nodes)
* Attempted I2.8xlarge for large job (3 nodes)
* Want to try on r5d.2xlarge with configurations:
Spark.executor.instances: 6
spark.yarn.executor.memoryOverhead: 3072
Spark.executor.memory: 18
spark.yarn.driver.memoryOverhead: 7168
Spark.driver.memory: 56
Spark.executor.cores: 1
Spark.driver.cores: 3
Spark.default.parallelism: 12


### Check out the folowing Apache Spark webpage links below for useful information

* A quick-start guide which will help you to gain a greater understanding of how to exercies the basic utilities of Apache Spark
`https://spark.apache.org/docs/latest/quick-start.html`
* The RDD Programming Guide, useful in understanding how Apache Spark parallelizes data for use in a SparkContext
`https://spark.apache.org/docs/latest/rdd-programming-guide.html#resilient-distributed-datasets-rdds`
* The Spark Streaming webpage, useful in understanding how the file is opened as a file stream.
`https://spark.apache.org/docs/latest/streaming-programming-guide.html`
  * Used the Hadoop filestream format described under the _Basic Sources_ -> _FileStreams_ paragraph with the format `StreamingContext.fileStream[KeyClass, ValueClass, InputFormatClass](filePath)`
  * KeyClass: `LongWritable` (imported with `org.apache.hadoop.io.LongWritable`)
  * ValueClass: `WARCWritable` (imported with `com.martinkle.warc.warc-hadoop.WARCWritable`)
  * InputFormatClass: `WARCInputFormat` (import with `com.martinkle.warc.warc-hadoop.WARCInputFormat`)

## The WARC Parser

Unless you are a prodigal programmer, or just happen to have your own WARC parser lying around, you'll want to use an open source project.  The one I used in this project (and have referenced a few other places so far) is available on GitHub at the following URL.  The package I imported to use it belongs to is com.martinkl.warc.\_
`https://github.com/ept/warc-hadoop`

## Compiling

This project was built using sbt.  You'll notice it uses a number of specific packages and they are not always included if you compile with `sbt package`, therefore I always compiled from the common-crawl-analytics directory with `sbt assembly`.

## Spark 2.-.- idiosyncracies

* Using map & flatMap methods on Datasets
You can no longer map directly over the rows of DataFrames, as they are now simply a type alias for Dataset[Row]. In Spark 2, to map over a Dataset[T], an encoder for type T must be available in the current context. Encoders are available for standard types and for case classes, but not for Row since it is a generic container type. If you wish to map over the rows of a DataFrame df, you should now convert it to an RDD first:

* Where before you would write:

`df.map { row: Row  => ... }`

* Now you should write:

`df.rdd.map { row: Row => ... }`

## Pulling Down a Common Crawl WET File

You generally have two options to get an individual WET file onto your computer. You can either put the URL into your browser _(I'm using Safari with the safe download feature off)_ or you can _curl_ the same URL from your command line. Simple instructions for both are included below. I should note that you have to have downloaded the wet.paths file from [Common Crawl](https://www.commoncrawl.org/).

### Using the URL

* Place the following URL into your browser `https://commoncrawl.s3.amazonaws.com/`
* Now, check the `wet.paths` file and pick the file you want to pull down. It does not matter which you choose. I'll paste `crawl-data/CC-MAIN-2018-05/segments/1516084891105.83/wet/CC-MAIN-20180122054202-20180122074202-00450.warc.wet.gz` for this one since I am looking at the 58051th file in the crawl from January of 2018.
* Hit return and your browser will start the download presently
* The files are not that large (~130MB) so they likely will not cause memory issues for you.
* Once downloaded, move the file to the directory you are working in and gunzip it before starting your work!

### Using 'curl' in bash

* Navigate to the directory in which you want to work with the WET file.
* Type the following, `curl https://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-05/segments/1516084891105.83/wet/CC-MAIN-20180122054202-20180122074202-00450.warc.wet.gz`, substituting the part after the initial domain name for the file you want to download.
* For this one I wrote a short script so that I could pull multiple files while working on something else. I've included it below, however it is specific for my use and you should only use it as an idea for one you could write yourself.

```
for fileNum in {'451','452','453','454'}; do curl "https://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-05/segments/1516084891105.83/wet/CC-MAIN-20180122054202-20180122074202-00$fileNum.warc.wet.gz" > "CC-MAIN-20180122054202-20180122074202-00$fileNum.warc.wet.gz"; done
```

OR _if you were to place the file paths in a file, or you just read them from the `wet.paths` file, then you could write the following_

```
while read filePath; do curl https://commoncrawl.s3.amazonaws.com/$filePath > $filePath; done < crawl-files.txt
```

### Ingesting Data With a Bash Script

The file, `ingestData.sh` is available for your use to ingest the entire corpus.  Update the shell in the following ways for your use.

* Update the `runDir` variable at the top of the file with the full path to the directory you want to store the crawl-data directory in.
* Provide the script with the wet.paths file you specifically are looking for.

## Useful Bash Scripts

### Unzipping many files in a single folder

The first one-liner will show you what files are contained within the directory. I recommend running it before and then again after to double check your work. The second one-liner will unzip each of the files.
```
for file in 1516084891105.83/wet/*; do echo $file; done
for file in 1516084891105.83/wet/*; do gunzip $file; done
```

### Storing the output into a single file

The two one liners below store the emails in a single file and place a line of text to indicate where the emails below came from.
```
for num in {00..09}; do echo part-0000$num >> 0003-emails.txt; cat emails.txt/part-0000$num >> 0003-emails.txt; done
for num in {10..27}; do echo part-000$num >> 0003-emails.txt; cat emails.txt/part-000$num >> 0003-emails.txt; done
```

