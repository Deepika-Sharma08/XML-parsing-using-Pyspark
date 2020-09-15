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
  
  ![alt text](https://github.com/Deepika-Sharma08/XML-parsing-using-Pyspark/blob/master/images/im4.png?raw=true)
  


  Step 4:Lets get the root for parsing.
  root = ET.fromstring(file_rdd.take(batch)[0][0])
  
  
  
  From here, it is just a matter of parsing (Note, this method is suitable for small size files)

  
    for i,child in enumerate(root[0]):
         for value in child:

             if len(value) > 0:

                 for val in value:

                     print(val.tag," :",value.find(val.tag).text)

             else:

                 print(value.tag, " :",child.find(value.tag).text)
                 
                 

  ![alt text](https://github.com/Deepika-Sharma08/XML-parsing-using-Pyspark/blob/master/images/im2.png?raw=true)
  


--------------------------------------------------------------------------------------------------------------
### Method 2

    1. Read RDD as sequential data, this will give you data as list, where you can control batch size

    file_rdd = spark.read.text("./xml_data/sample_order.xml", wholetext=False)


    2. Write your parser as def get_values, and register it with Pyspark using *sql.udf*
       pyspark.sql.udf.UDFRegistration.register(name="get_values", f = get_values, returnType=StringType())
       
    3. Parallelize the RDD
    myRDD = sc.parallelize(file_rdd.take(batch),num_of_partitions)
    

Note, batch size will depend on your system configuration, and so will num_of_partitions. These numbers can be reconfigured. based on performance measures like (system capacity, num of cores, latency allowed in parsed values return)


    4. Run the spark job on partitioned RDD
    parsed_records = sc.runJob(myRDD, lambda part: [get_values(x) for x in part])
    
    5. Lets' convert results to pandas DataFrame
  
  ![alt text](https://github.com/Deepika-Sharma08/XML-parsing-using-Pyspark/blob/master/images/im1.png?raw=true)
  
  ![alt text](https://github.com/Deepika-Sharma08/XML-parsing-using-Pyspark/blob/master/images/im3.png?raw=true)
  
  

Next step is to pattern match and identify wrong NANs :) and correction via feedback loop to spark streaming. 


I have explored two ways,

a). Reading the whole data as one string and parsing via navigating through root.

b) Reading the whole data as sequential stream by controlling batch size (in case of large files) and parsing string using pattern match via passing spark udf to each node along with the data.
