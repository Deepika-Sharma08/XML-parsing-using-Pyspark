# XML-parsing-using-Pyspark

## Parsing XML file using Pyspark : Part 1



`Author : Deepika Sharma`    `| | `  `         Time : September 2020`       `                `

## Analysis

***

Here are the steps for parsing xml file using Pyspark functionalities

`Method 1`
1. Reading the file as string as a contineous stream, *`only if file size is small`*.
2. Create a parser based on patterns observed, parse the file (file which is cached in memory)

`Method 2`
1. Reading the file as list of strings split by \n.
2. Partition of data to run parallel jobs on various nodes.
3. Distribute the partitioned RDDs.
4. Parse the strings using key values (Since it is converted to string format, functionalities of python re, and pattern match are used for parsing the xml).



### Method 1

  Step 1: Read XML files into RDD
  file_rdd = spark.read.text("./xml_data/sample_order.xml", wholetext=True).rdd


  Step 2: Make use of the python library for XML parsing (in case RDD returns one long string of the entire data)
  import xml.etree.ElementTree as ET


  Step 3: file_rdd.take(1)
  Lets look at a few entries of the xml file..
  
  
  

Lets get the root for parsing.

I have explored two ways,

a). Reading the whole data as one string and parsing via navigating through root.

b) Reading the whole data as sequential stream by controlling batch size (in case of large files) and parsing string using pattern match via passing spark udf to each node along with the data.
