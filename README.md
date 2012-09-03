csit_hadoop
===========

Deliverables from my internship at CSITDate: 17/07/12
Author: Ng Zhi An
Organization: CSIT

The directory structure should be as such:
	--Intern Journal.xls					(01)
	--Internship Scope.doc					(02)
	--documentation							(03)
		|--Hadoop and HBase Set-up guide
		|--Hadoop Cluster Setup
		|--Hadoop Single-node setup
		|--MapReduce Data Flow
		|--HBase
		|--Zookeeper Replicated Setup
		|--HBase Fully Distributed
		|--Hadoop
		|--Pig
		|--Column-oriented Database
		|--Relational Database
	--guide									(04)
		|--guide.html
		|--guide.css
	--presentation							(05)
		|--CSIT_Hadoop_Presentation.ppt
		|--present.html
		|--Script.doc
		|--js
			|--impress.js
		|--img
			|--19 images
		|--css
			|--present.css
			|--new_athena_unicode.ttf
			|--ColabThi.otf
	--code									(06)
		|--RowCounter.txt
		|--getStationInfo.txt
		|--HBaseTemperatureImporter.txt
		|--getStationObservations.txt
	--uml									(07)
		|--csit_hadoop.uml
		|--sd_hbaseoperation_retrieve_scan.jpg
		|--sd_hbaseoperation_retrieve_get.jpg
		|--sd_hbaseoperation_delete_delete.jpg
		|--sd_hbaseoperation_create_update_put.jpg
		|--sd_hbasemain_overview.jpg
		|--sd_copy_from_hdfs_to_hbase.jpg
		|--ClassDiagram.jpg
	--app									(08)
		|--csit-hadoop.jar

(01) This journal is a log of what I have been doing at CSIT,
it includes obstacles I have faced and how I overcome them.

(02) The scope was defined initially by my mentor. It was amended
and updated throughout the internship to faciliate my learning.

(03) I've done extensive documentation, beginning with research
on databases (both relational and column). "Hadoop", "HBase" and "Pig" are
detailed documentation on how these technology works, including
architecture and data structure. The rest are more of set-up guides
and tutorials. The most important documentation here is the
"Hadoop and HBase Set-up guide", which is a step-by-step guide to setting
up a full system. This is in the form of a long document and may overwhelm
some people, an alternative representation of this guide is
available in "guide" as guide.html.

(04) "guide.html" is the same as "Hadoop and HBase Set-up guide", just presented
in a multi-page view with links for easy digestion.

(05) "presentation" contains firstly "present.html", which is a html
presentation running on "impress.js". "present.html" is aesthetically
sweetened and thus is more suitable for presenting. A more clean look
at the presentation is available at "CSIT_Hadoop_Presentation.ppt".

(06) The "code" folder contains helper code from reference books
on HBase and Hadoop. You will see that in the documentation I will
refer to these code. I added the code here for convenience.

(07) My mentor asked me to explore the Unified Process and
Object-Oriented Design, and these are the products of my learning.
The diagrams illustrates how my app will run and how each of the query
by the user will result in different flow of the program.
"csit_hadoop.uml" is the project file itself, and I used StarUML.

(08) "app" folder contains my application archive and source.
My application consists of two parts. Part 1 deals with uploading files
from HDFS to HBase. Part 2 deals with querying HBase table.

--Part 1--
HtmlToHBaseTable.class takes care of uploading files from HDFS to HBase
via a Map task.
The prerequisites are that the table must already exist, with
at least 1 column family. 
It begins with the HtmlInputFormat, which treats each file as a single
split. It spawns a HtmlRecordReader for each split, and HtmlRecordReader
will treat the HTML file's name as the key, and the entire HTML code
as value. It reads in the HTML code via the HtmlFileReader,
which takes care of reading in the raw text of the HTML file.
Then this key-value pair is passed to the Map inner-class in HtmlToHBaseTable
and parsed via method calls to the HtmlParser.
HtmlParser does many things, including removal of HTML tags,
replacing HTML encoded values with appropriate symbols, extracting
date of article, language and categories.
From HTML files, date and language attributes
are extracted based on DATE_IDENTIFIER and LANG_IDENTIFIER.
This two fields determine where in the HTML file to find the date
and language. Another important thing to note is the DATE_PATTERN, this
determines the original representation of date in the HTML file.
We will then use this pattern to convert the representation of date into
a more index-able format, from MMMM DD, yyyy to dd/MM/yy.
After parsing, all the attributes are placed together in a Put object,
and TableOutputFormat will ensure that this record is written
to our table. I chose to let the user specify the table name, and
an exception will be thrown if table does not exist.
The connection to the table and configuration is handled by
TableMapReduceUtil.initTableReducerJob.

--Part 2--
HBaseOperation takes care of querying a HBase table, it is simply
an abstraction layer on top of HBase's own query classes on Java.
HBaseOperation provides simple get(), scan(), put(), delete() methods for
operations on the table. These methods can be extended because
many settings can be tweaked for each command. The get() and scan() methods
return byte arrays, hence a helper method addRecord() will retrieve all 
fields of the record, using a helper method getAllAttributes(), and store
it in a ArrayList<Record>. A Record is a simple compound object made up of
row, family, qualifier, value, timestamp, and uniquely identifies
a single cell in the table. Many Record objects can be converted into a Row
object via Row.formRow. A Row is a true representation of a row,
which can have multiple column-families, and each column-family can have
multiple columns, each column has multiple values with different timestamps.
Row.formRow makes use of the Row.addRecordToRow method, which adds a single
Record object to a Row object, which in turn depends on methods like
retrieveRowValue() and retrieveColumnFamilyValue(). Such methods will retrieve
an instance of a single HashTable representation of the Row or ColumnFamily.
This is to allow the methods addColumnFamily and addQualifier to add to
the correct field.
HBaseMain is a interative command-line prompt. It asks the user for the query
he wishes to enter (get, scan, put or delete), and will pass the input to
the CommandParser. CommandParser will interpret the input and proceed to prompt
for the required information, such as table name, row key etc. It then calls
the appropriate HBaseOperation methods, passing in the information the user
has entered. In the case where results are to be shown to user (get and scan),
it will provide a choice to the user, either to print the results on screen,
or to save the results to HDFS. If printed on screen, the results will be
converted to a Row object (as specified above), and then printed on console.
Else CommandParser.getFileName() will be called to prompt the user to enter
a name for the file they wish to save the results in, and writeToFile()
will be called. The creation of file on HDFS and stream to write to the file
is handled by Utils.HadoopUtils. The Utils class aims to separate all utility
methods, such as connection and stream, from the main logic.