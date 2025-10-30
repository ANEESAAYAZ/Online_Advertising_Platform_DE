# Online_Advertising_Platform_DE

Task is to build an online advertising platform. The following points were discussed in the video above:

The platform will have an interface for campaign managers to run the Ad campaign and another interface for the client to present the Ads and send the user action back to the advertising platform.
Through the campaign manager, the Ad instructions (New Ad Campaign, Stopping the existing Ad campaign) will be published to a Kafka Queue. The ‘Ad Manager’ will read the message from that Kafka queue and update the MySQL store accordingly.
An Ad Server will hold the auction of Ads for displaying the Ad to a user device. The auction winner will pay the amount bid by the second Ad. A user simulator will hit the Ad Server API for displaying Ads and send the user interaction feedback back to the feedback handler through API.
Upon receiving the user interaction feedback, the feedback handler will publish that event to another Kafka queue.
The User feedback handler will collect the feedback through the feedback API and it will be responsible for updating the leftover budget for the Ad campaign into MySQL. It will also publish the feedback to the internal Kafka queue, which will be later published to HIVE through user feedback writer for billing and archiving.
The whole system needs to be run for an hour and after that, the report generator will generate the bill report for all Ads displayed so far.

for the given project, you need to use the following three data sets:


Amazon Advertisements: This data set contains data pertaining to the advertisement from Amazon (stand by the end of 2019)

ADS 16 data set: This data set was used to ascertain the user preference for the advertisement data. 

Advertising: This data set contains the demographics and the internet usage patterns of the users.

For our project, the above data sets were used as a baseline to create the Ad campaigns as well as the simulated users which will be used to produce the Ad info data and the user data. 

 

The details of the Kafka broker for the Ad campaign are as follows:

IP:Port - 18.211.252.152:9092 
topic -  de-capstone1
 

You can download the user simulator program here:

User Simulator
Download
Download the user dataset from the link below:

 

https://de-capstone-project1.s3.amazonaws.com/users_500k.csv
 

Now, having understood the various data sets and APIs involved in the project, you learnt about the architecture which has to be implemented for this case study. Following is a graphic representation of the same. 

 you need to code the following components of the architecture diagram:

Ad Manager

Ad Server

Feedback Handler

Slot Budget Manager

User Feedback Writer

Data Archiver

Report Generator

the schema of the **Ad campaign** Kafka queue is as follows:

 

Column	Description
Text	Text present in the advertisement
Category	Category of the product being advertised
Keywords	Textual keywords for the advertisement
Campaign ID	The identifier for the advertising campaign
Action	Type of campaign instruction
Target Gender	This can be ‘M’, ‘F’ or ‘All’
Target Age Range	Target age range; here, ‘0–0’ means all age groups
Target City	Target city; here, ‘All’ means no specific city is targeted
Target State	Target state; here, ‘All’ means no specific state is targeted
Target Country	Target country; here, ‘All’ means no specific country is targeted
Target Income Bucket	Target income bucket: ‘H’, ‘M’, ‘L’ or ‘All’
Target Device	Target device type
CPC	The amount spent for every click on the Ad
CPA	The amount spent for every call-to-action placed on an Ad
Budget	Total campaign budget
Date Range	The date range when the campaign will run
Time Range	The time range during which the campaign will run

 

This Ad campaign Kafka queue would be used for different kinds of Ad campaign operations. ‘Action’ can take any of the following values:

New Campaign

Update Campaign

Stop Campaign

The **Ad storage** schema is as follows:

 

Column	Datatype	Description
text	NVARCHAR	Text present in the advertisement
category	NVARCHAR	Category of the product being advertised
keywords	NVARCHAR	Textual keywords for the advertisement
campaign_id	NVARCHAR	An identifier for the advertising campaign
status(Derived)	NVARCHAR	Either ‘ACTIVE’ or ‘INACTIVE’
target_gender	NVARCHAR	This can be either ‘M’, ‘F’ or ‘All’
target_age_start	INT	Target age lower limit
target_age_end	INT	Target age upper limit
target_city	NVARCHAR	Target city; here ‘All’ means no specific city is targeted
target_state	NVARCHAR	Target state; here ‘All’ means no specific state is targeted
target_country	NVARCHAR	Target country; here ‘All’ means no specific country is targeted
target_income_bucket	NVARCHAR	Target income bucket: ‘H’, ‘M’, ‘L’ or ‘All’
target_device	NVARCHAR	Target device type
cpc	DOUBLE	The amount spent for every click on the Ad
cpa	DOUBLE	The amount spent on every call-to-action placed on the Ad
cpm(Derived)	DOUBLE	The amount spent on every impression/view of the Ad
budget	DOUBLE	Total campaign budget
current_slot_budget(Derived)	DOUBLE	The budget allocated for the current Ad campaign slot
date_range_start	NVARCHAR	The starting date for the campaign
date_range_end	NVARCHAR	The ending date for the campaign
time_range_start	NVARCHAR	The starting time for the ad campaign
time_range_end	NVARCHAR	The ending time for the ad campaign
 


Majorly it has the same attributes as that of the Kafka queue with a few additional attributes as follows:

Status: It can either be ‘Active’ or ‘Inactive’. In the case of the ‘New Campaign’ and ‘Update Campaign’ instructions, the status will be ‘ACTIVE’. In the case of ‘Stop Campaign’ messages, the value should be set to ‘INACTIVE’. 

Cost Per Mille (CPM): This is the amount spent for every impression/view of the Ad. This attribute is derived from the following equation:

C
P
M
=
(
W
e
i
g
h
t
C
P
C
∗
A
m
o
u
n
t
C
P
C
)
+
(
W
e
i
g
h
t
C
P
A
∗
A
m
o
u
n
t
C
P
A
)

 

Here, the 
W
e
i
g
h
t
C
P
C
 will be 0.0075 and 
W
e
i
g
h
t
C
P
A
will be 0.0005.

 

Current Slot Budget: This refers to the budget allocated for the current Ad campaign slot. To distribute the advertisement uniformly in the timespan of the campaign, the complete duration is broken into multiple slots of uniform length (e.g., a day or an hour) and budget is allocated uniformly among these slots. For this exercise, as discussed by Mayukh in the video, the slot duration will be 10 minutes. The budget for each slot will be allocated as follows:

B
u
d
g
e
t
N
u
m
b
e
r
o
f
S
l
o
t
s

**Ad manager** reads data from the central Kafka queue for the Ad campaign, and will process the following and then take corresponding actions to MySQL events:


New Campaign: Insert Operation

Update Campaign: Update Operation

Stop Campaign: Update Operation

 

Also, you need to calculate the derived attributes namely (Cost Per Mille (CPM) and status for every advertising campaign and insert/update the entry in the database.

 

As explained, for the Ad Manager, you will be using a PyKafka consumer for reading the data from Kafka and Python MySQL connector for connecting to MySQL.

Note: Additional resources are provided in subsequent segments for reading data from Kafka and writing to MySQL. You can go through that in order to get a better understanding of the same.

The purpose of the **slot budget manager** is to distribute the leftover budget uniformly on the Ad slots and utilise the budget fully. As explained in the video earlier, it will be a cron job running every 10 minutes and the Python MySQL connector would be used to write the code. 


Note: For the cron job, you can refer to the additional resources provided in the subsequent segments wherein you can learn more about how to run a cron job.

you need to create a MySQL table from the user’s data set provided in the previous segment which should have the following schema:

 

Column	Datatype	Description
id	NVARCHAR	User Identifier
age	INT	Age of the user
gender	NVARCHAR	Gender of the user
internet_usage	NVARCHAR	Daily average internet usage
income_bucket	NVARCHAR	Estimated income bucket; possible values: ‘H’, ‘M’, ‘L’
user_agent_string	NVARCHAR	User-agent string used by the user
device_type	NVARCHAR	Type of the device used by the user
websites	NVARCHAR	Types of websites liked by the user
movies	NVARCHAR	Types of movies liked by the user
music	NVARCHAR	Types of music liked by the user
program 	NVARCHAR	Types of programs liked by the user
books	NVARCHAR	Types of books liked by the user
negatives	NVARCHAR	Keywords representing the user’s dislikes
positives	NVARCHAR	Keywords representing the user's likes

The **Ad Server** is responsible for serving the ads to the users. When a user is active online, the client application will send a request to the Ad server along with the user details. The request from a typical mobile app client will contain a Google Play Store ID (GPID) or an Apple ID along with locality details. Upon receiving the request, the Ad server will execute the following events:

Find the list of available Ads

Hold an auction among candidate Ads and the winning Ad will be served to the user.
For the auction of the Ads, you will be using the ‘Second-Price Auction’ strategy. The auction bid for every Ad will be its Cost Per Mille (CPM). The highest bidder will win the auction, but the winner has to pay the price that is bid by the second-highest bidder. If there is a single eligible Ad, the campaigner has to pay the price same as the original bid.

 

As explained, the responsibility of the Ad Server is to serve Ads to the user. This will be done through an API call. You have to use the Flask library to create a web server and serve APIs in Python. The API can be described as follows:

 

URL Format: 

<host>/ad/user/<user_id>/serve?device_type= <device_type>&city=<city>&state=<state>
 

HTTP Method: GET

Sample URL: 

http://localhost:5000/ad/user/6abc435e-0f72-11eb-8a4e-acde48001122/serve?device_type=android-mobile&city=mumbai&state=maharastra
 

 

Sample Response:

{
         "text": "Jack Wolfskin Men's Rock Hunter Low Water
         Resistant Hiking Shoe",
         "request_id":"17001d26-0f72-11eb-8a4e-acde48001122"
}
 

 

The schema for the served ad table in MySQL is as follows:

 

Column	Datatype	Description
request_id	NVARCHAR	Identifier for the Ad serve request
campaign_id	NVARCHAR	Identifier for the advertising campaign
user_id	NVARCHAR	Identifier for the user to whom this Ad was displayed
auction_cpm	DOUBLE	CPM decided during the auction
auction_cpc	DOUBLE	CPC decided during the auction
auction_cpa	DOUBLE	CPA decided during the auction
target_age_range	NVARCHAR	Target age range of the Ad – combined into a single attribute
target_location	NVARCHAR	Target city, state and country of the Ad combined
target_gender	NVARCHAR	Target gender for the Ad campaign
target_income_bucket	NVARCHAR	Target income bucket for the Ad campaign
target_device_type	NVARCHAR	Target device type for the Ad campaign
campaign_start_ime	NVARCHAR	Campaign start date and time combined into a single attribute
campaign_end_time	NVARCHAR	Campaign end date and time combined into a single attribute
timestamp	TIMESTAMP	Timestamp of when the user feedback was received

Note: In order to learn more about the Flask framework and APIs, you can refer to the Additional Resources II segment.


 

Now, in order to get user feedback, we need a user simulator that simulates user behaviour.
you will be provided with a simulator script, which will simulate the user behaviour in the client application. This script will hit the APIs hosted in the Ad Server in a manner similar to the user. First, it will hit the API for getting Ads and then, it will hit the feedback API with the request identifier received in the earlier API call. It will query the ‘users’ table in MySQL to get the list of available users. Therefore, the naming of the tables is crucial since the same script will be used during evaluation as well.


 

The script needs to be run with the parameters as follows:

python user_simulator.py <database_host> <database_username>
<database_password> <database_name> <protocol> <ad_server_host>  
<ad_server_port> <feedback_handler_host> <feedback_handler_port>
For example, the command used by us was as follows:

python3 user_simulator.py <public DNS of Master Node> root 123 upgrad http 0.0.0.0 5000 0.0.0.0 8000

The **feedback handler** is responsible for submitting user feedback. It will enrich the data before publishing it to the Kafka queue. You have to create a new Kafka topic for this purpose. The topic should have one partition and a replication factor of 1.

 


Now, the client application shares only the user interaction data in the feedback. The same needs to be combined with other attributes of the auction and the Ad campaign to make it efficient for consumption in billing and reports. When the client application sends the user feedback to the feedback handler through the feedback API, it will retrieve the extra attributes from MySQL to enrich the feedback data. The user feedback API will have the original Ad request identifier as an argument and that will be used for fetching the additional attributes.

Here, you will need to use Flask for the APIs and MySQL connector library to connect to MySQL.

 

Once an Ad has been displayed, information on whether the user has clicked on the Ad, downloaded the advertised App or only viewed the Ad needs to be sent back to the Ad server through the user feedback API. The feedback handler will enrich the feedback data and publish it to the internal Kafka topic.

 

URL Format:

 <host>/ad/<ad_request_id>/feedback
 

HTTP Method: POST

Sample URL: 

 http://localhost:8080/ad/17001d26-0f72-11eb-8a4e-acde48001122/feedback
 

Sample Request Body:

{
     “View”:1,
     “Click”:1,
     “Acquisition”:0
 }
 

Sample Response:

{
    “status”:” SUCCESS”
 }
 

 

once the Ad mapping attributes are retrieved from the MySQL table ‘served_ads’, the feedback handler needs to derive two more additional attributes –  expenditure and user_action – before proceeding further. 

Expenditure: If the ‘acquisition’ attribute is set to 1 in the user feedback, then the expenditure amount will be the same as ‘auction_cpa’. If the ‘click’ attribute is set to 1, then the expenditure amount will be the same as ‘auction_cpc’. Otherwise, the expenditure will be 0.

User_action: If the ‘acquisition’ attribute is set to 1 in the user feedback, then the ‘user_action’ will be ‘acquisition’. If the ‘click’ attribute is set to 1, then the ‘user_action’ will be ‘click’. Otherwise, ‘user_action’ will be ‘view’.

Once the expenditure is calculated, it has to be reflected in the Ad campaign budget in MySQL. Subtract the expenditure amount from the campaign's budget and slot budget. If the budget becomes 0 or negative, then the Ad status is set to INACTIVE. 

 

Finally, the enriched user feedback data needs to be sent to the internal Kafka topic named ‘user-feedback’ for archiving and billing purposes.

The user feedback queue schema is as follows:

 

Columns	Description
Campaign ID	The identifier for the advertising campaign
User ID	The identifier for the user to whom this Ad was displayed
Request ID	The identifier for the Ad serve request
Click	Flag indicating whether or not the Ad was displayed to the user
View	Flag indicating whether or not the Ad was clicked by the user
Acquisiton	Flag indicating whether or not the user has done the call-to-action
Auction CPM	CPM decided during the auction
Auction CPC	CPC decided during the auction
Auction CPA	CPAdecided during the auction
Target Age Range	Target age range of the Ad combined into a single attribute
Target Location	Target city, state and country of the Ad combined
Target Gender	Target gender for the Ad campaign
Target Income Bucket	Target income bucket for the Ad campaign
Target Device type	Target device type for the Ad campaign
Campaign Start Time	Campaign start date and time combined into a single attribute
Campaign End Time	Campaign end date and time combined into a single attribute
Action	Action performed by the user. Possible values: 'view', 'click', 'acquisition'
Expenditure	The amount payable by the Ad campaign runner
Timestamp	The timestamp of when the user feedback was received

 the user **feedback writer** will read the user feedback messages from the Kafka queue and write the feedback data to HDFS(Hadoop Distributed File System) for archiving and billing purposes. This will be a PySpark consumer job.

 


Note: In order to learn more about HDFS sink in PySpark, you can refer to the additional resources provided in the subsequent segments.

data archiver is responsible for exporting Ads table data from MySQL to Hive. You do not need all the attributes of the table. The required attributes are as follows:


Campaign ID

Category

Budget

CPM

CPC

CPA

Target Device

 

Note: You should run the Sqoop command once you have run the pipeline for over an hour. After that, you will need to stop the pipeline and proceed towards report generation in Hue. You can use the same name for creating a table in Hive. 

  you will need to calculate a few metrics from the Ad feedback data collected in Hive. The whole report generation needs to be done in Apache Hue.


Top 10 under-utilised Ad campaigns: Top 10 Ad campaigns with the highest leftover budget

Top 10 spending Ad campaigns: Top 10 Ad campaigns with the highest expenditure

Total expenditure and click-through rates (CTR) of Ad campaigns [CTR = (number of clicks/ of page views)

Top five interactive (highest CTRs) age groups: Show this in a bar chart in Hue (CTR should be on the y-axis)

Top five interactive locations: Show this in a bar chart in Hue (CTR should be on the y-axis)

Top interactive gender: Show this in a bar chart in Hue (CTR should be on the y-axis)

Top interactive income buckets: Show this in a bar chart in Hue (CTR should be on the y-axis)

Top five interactive device types: Show this in a bar chart in Hue (CTR should be on the y-axis)

Top 10 spending Ad categories: Top 10 Ad categories with the highest expenditure. Show this in a bar chart in Hue (Expenditure should be on the y-axis)

Highest price differences in CPM during auctions

**Guidelines**
Before we discuss the guidelines and tasks that you need to perform, make sure that you are configuring the EMR cluster properly.

 

You will need to make sure that you have Hadoop, Sqoop, Hive and Spark installed on your cluster with Hue as an optional service. Also as an added step, make sure that in the Hardware configuration step for the EMR cluster generation, scroll down to the EBS Root Volume configuration and type the Root device EBS volume size as 20 GB. 

 

As part of the project, broadly you are required to perform the following tasks:

 

Task 1: Setup MySQL Environment. You need to create "users", "ads" and "served_ads" table in MySQL using the given schema.

 

Task 2: Writing Ad Manager using PyKafka. Here you need to read the data from Kafka then add the additional attributes followed by MySQL queries and finally printing the Campaign Id, Action & Status on Console.

 

Task 3: Writing Ad Slot Budget manager using MySQL Connector. Here you need to write a python script for slot budget calculation and updation. Here you also need to set up a proper cron job. In order to learn more about cron job, you can refer to the subsequent session of Additional Resources. 

 

Task 4: Writing Ad Server using Flask & MySQL Connector. Here you need to perform the following task:

Write the API code using Flask.
Performing the ad auction.
 

Task 5: Writing Feedback Handler using Flask & MySQL Connector. Here you need to perform the following major tasks:

Write the API code using Flask.
Retrieve the entry from "served_ads" table.
Add the additional attributes with the user feedback data.
 

Task 6: Writing User Feedback Writer using PySpark.

 

Task 7: Backing up Ad data from MySQL to Hive using Sqoop.

 

 Task 8: Report Generator wherein you have to use Hive create table query for user feedback. Also, you need to generate reports based on the following parameters:

Top 10 under-utilised Ad Campaign
Top 10 spending Ad Campaign
Total expenditure and CTR of the Ad Campaigns
Top 5 Interactive(based on the CTR)
Top 10 spending Ad Category
Top Auction price differences.
In the next segment, you will go through a Code Run walkthrough of the entire project.
