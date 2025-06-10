
https://www.textfromtospeech.com/en/voice-to-text/



### splunk
In general, machine data is difficult to understand and has an unstructured format (not arranged as per pre-defined data model), making it unsuitable for analysis and visualization of data. Splunk is the perfect tool for tackling such problems. Splunk is used to analyze machine data for several reasons: 

- **Provides business insights:** The Splunk platform analyzes machine data for patterns and trends, providing operational insights that assist businesses in making smarter decisions for the profitability of the organization.
- **Enhances operational visibility:** Splunk obtains a comprehensive view of overall operations based on machine data and then aggregates it across the entire infrastructure.
- **Ensures proactive monitoring:** Splunk employs a real-time analysis of machine data to discover system errors and vulnerabilities (external/internal breaches and intrusions).
- **Search and Investigation:** Splunk uses machine data to pinpoint and fix problems by correlating events across numerous data sources and detecting patterns in large datasets.

Works - In order to use Splunk in your infrastructure, you must understand how Splunk performs on the internal level. In general, Splunk processes data in three stages:

- **Data Input Stage:** This stage involves Splunk consuming raw data not from a single, but from many sources, breaking it up into 64K blocks, and annotating each block with metadata keys. A metadata key includes the hostname, source, and source type of the data.
- **Data Storage Stage:** In this stage, two different phases are performed, Parsing and Indexing.
In the Parsing phase, Splunk analyzes the data, transforms it, and extracts only the relevant information. This is also called "event processing," since it breaks down the data sets into different events.
During the indexing phase, Splunk software writes the parsed events into the index queue. One of the main benefits of using this is to make sure the data is easily accessible for anyone during the search.
- **Data Searching Stage:** This stage usually controls how the index data is accessed, viewed, and used by the user. Reports, event types, dashboards, visualization, alerts, and other knowledge objects can be created based on the user's reporting requirements.

#### components & Architecture
Details
- **Forwarder:** These are components that you use to collect machine data/logs. This is responsible for gathering and forwarding real-time data with less processing power to Indexer.  Splunk forwarder performs cleansing of data depending on the type of forwarder used (Universal or Heavy forwarder).
- **Indexer:** The indexer allows you to index i.e., transform raw data into events and then store the results data coming from the forwarder. Incoming data is processed by the indexer in real-time. Forwarder transforms data into events and stores them in indexes to enable search operations to be performed efficiently.
- **Search Head:** This component is used to interact with Splunk. It lets users perform various operations like performing queries, analysis, etc., on stored data through a graphical user interface. Users can perform searches, analyze data, and report results.

#### Different forwarder.
A forwarder is a Splunk instance or agent you deploy on IT systems, which collects machine logs and sends them to the indexer. You can choose between two types of forwarders:  
- **Universal Forwarder:** A universal forwarder is ideal for sending raw data collected at the source to an indexer without any prior processing. Basically, it's a component that performs minimal processing before forwarding incoming data streams to an indexer. Although it is faster, it also results in a lot of unnecessary information being forwarded to the indexer, which will result in higher performance overhead for the indexer.
- **Heavy Forwarder:** You can eliminate half of your problems using a heavy forwarder since one level of data processing happens at the source before forwarding the data to the indexer. Parsing and indexing take place on the source machine and only data events that are parsed are sent to the indexer.

#### advantages of getting data into a Splunk instance through Forwarders
Data entering into Splunk instances via forwarders has many advantages including bandwidth throttling, a TCP connection, and an encrypted SSL connection between the forwarder and indexer. By default, data forwarded to the indexers are also load-balanced, and if one indexer goes down for any reason, that data can always be routed to another indexer instance in a very short amount of time. Furthermore, the forwarder stores the data events locally before forwarding them, creating a temporary backup of the data.

#### Dashboards and write its type
In a dashboard, tables, charts, event lists, etc., are used to represent data visualizations, and they do so by using panels. Dashboard panels present or display chart data, table data, or summarized data visually in a pleasing manner. On the same dashboard, we can add multiple panels, and therefore multiple reports and charts. Splunk is a popular data platform with lots of customization options and dashboard options. 

There are three kinds of the dashboard you can create with Splunk: 
- **Dynamic form-based dashboards:** They allow Splunk users to change the dashboard data based on values entered in input fields without leaving the page. A dashboard can be customized by adding input fields (such as time, radio buttons, text boxes, checkboxes, dropdowns, and so on) that change the data, depending on the selection made. Dashboards of this type are useful for troubleshooting issues and analyzing data.
- **Static Real-time Dashboards:** They are often displayed on a large screen for constant viewing. It also provides alerts and indicators to prompt quick responses from relevant personnel.
- **Scheduled Dashboards:** These dashboards can be downloaded as PDF files and shared with team members at predetermined intervals. There are times when active live dashboards can only be viewed by certain viewers/users only.
- Some of the Splunk dashboard examples include security analytics dashboard, patient treatment flow dashboard, eCommerce website monitoring dashboard, exercise tracking dashboard, runner data dashboard, etc

#### Query.
Splunk queries allow specific operations to be run on machine-generated data. Splunk queries communicate with a database or source of data by using SPL (Search Processing Language). This language contains many functions, arguments, commands, etc., that can be used to extract desired information from machine-generated data. This makes it possible for users to analyze their data by running queries. Similar to SQL, it allows users to update, query, and change data in databases.

It is primarily used to analyze log files and extract reference information from machine-generated data. In particular, it is beneficial to companies that have a variety of data sources and need to process and analyze them simultaneously in order to produce real-time results. 

#### License
A license is required for each Splunk instance. With Splunk, you receive a license that specifies which features you can use and how much data can be indexed. Various Splunk License types include:
- The Splunk Enterprise license: Among all Splunk license types, Enterprise licenses are the most popular. These licenses give users access to all the features of Splunk Enterprise within a specified limit of indexed data or vCPU usage per day. These licenses include enterprise features such as authentication and distributed search. Several types of Splunk Enterprise licenses are available, including the Splunk for Industrial IoT license and Splunk Enterprise Trial license.
- The Free license: Under the Free license, Splunk Enterprise is completely free to use with limited functionality. Some features are not available under this license, such as authentication. Only a limited amount of data can be indexed.
- The Forwarder license: A Forwarder license enables unlimited forwarding of data, as well as a subset of the Splunk Enterprise features that are required for authentication, configuration management, and sending data.
- The Beta license: Each Splunk beta release requires a separate beta license, which cannot be used with other Splunk releases. With a beta license, Splunk Enterprise features are enabled for a specific beta release period

-  License Master in Splunk? If the License Master is not reachable, what will happen
     - It is the responsibility of the license master in Splunk to ensure that the limited amount of data is indexed. Since each Splunk license is based on the amount of data that is coming into the platform in 24 hours, it is essential to keep the environment within the limits of its purchased volume.  

- Whenever the license master becomes unavailable, it is simply impossible to search the data. Therefore, only searching remains halted while the indexing of data continues. Data entering the Indexer won't be impacted. Your Splunk deployment will continue to receive data, and the Indexers will continue to index the data as usual. However, upon exceeding the indexing volume, you will receive a warning message on top of your Search head or web interface so that you can either reduce your data intake or purchase a larger capacity license.


- Explain License violation. How will you handle or troubleshoot a license violation warning
     - License violations occur after a series of license warnings, and license warnings occur when your daily indexing volume exceeds the license's limit. Getting multiple license warnings and exceeding the maximum warning limit for your license will result in a license violation. With a Splunk commercial license, users can receive five warnings within a 30-day period before Indexer stops triggering search results and reports. Users of the free version, however, will only receive three warnings

**Avoid License Warning:**
- Monitor your license usage over time and ensure that you have enough license volume to meet your daily needs.
- Viewing the license usage report in the license master can help troubleshoot index volume.
- In the monitoring console, set up an alert to track daily license usage.

**Troubleshoot License Violation Warning:**
- Determine which index/source type recently received more data than usual.
- Splunk Master license pool-wise quotas can be checked to identify the pool for which the violation occurred.
- Once we know which pool is receiving more data, then we need to determine which source type is likely to be receiving more than normal data.
- Having identified the source type, the next step is to find out which machine is sending so many logs and the reason behind it.
- We can then troubleshoot the problem accordingly

#### Splunk ports
- Details
     - Web Port: 8000 
     - Management Port: 8089 
     - Network port: 514 or 9997
     - Index Replication Port: 8080 
     - Indexing Port: 9997 or 9887
     - KV store: 8191

#### Splunk Database (DB) Connect.
Splunk Database (DB) Connect is a general-purpose SQL (Structured Query Language) database extension/plugin for Splunk that permits easy integration between database information and Splunk queries/reports. Splunk DB Connect is effectively used to combine structured data from databases with unstructured machine data, and Splunk Enterprise can then be used to uncover insights from the combined data. 

Some of the benefits of using Splunk Database Connect connect are as follows:   
- By using Splunk DB Connect, you are adding new data inputs for Splunk Enterprise, i.e., adding additional sources of data to Splunk Enterprise. Splunk DB Connect lets you import your database tables, rows, and columns directly into Splunk Enterprise, which then indexes them. Once that relational data is within Splunk Enterprise, you can analyze and visualize it the same way you would any other Splunk Enterprise data.
- In addition, Splunk DB Connect enables you to write your Splunk Enterprise data back to your relational databases.
- With DB Connect, you can reference fields from an external database that match fields in your event data, using the Database Lookup feature. This way, you can enrich your event data with more meaningful information.

#### versions of the Splunk product
Splunk products come in three different versions as follows:  
     - **Splunk Enterprise:** A number of IT companies use Splunk Enterprise. This software analyzes data from diverse websites, applications, devices, sensors, etc. Data from your IT or business infrastructure can be searched, analyzed, and visualized using this program.
     - **Splunk Cloud:** It is basically a SaaS (Software as a Service) offering many of the same features as enterprise versions, including APIs, SDKs, etc. User logins, lost passwords, failed login attempts, and server restarts can all be tracked and sorted.
     - **Splunk Light:** This is a free version of Splunk which allows you to view, search, and edit your log data. This version has fewer capabilities and features than other versions

#### free version - features not available.
Details 
     - Distributed searching
     - Forwarding of data through HTTP or TCP (to non-Splunk)
     - Agile reporting and statistics based on a real-time architecture
     - Scheduled searches/alerts and authentication
     - Managing deployments

#### alerts and write about different options available while setting up alerts.
Splunk alerts are actions that get triggered when a specific criterion is met; these conditions are defined by the user. You can use Splunk Alerts to be notified whenever anything goes awry with your system. For instance, the user can set up Alerts so that an email notification will be sent to the admin when three unsuccessful login attempts are made within 24 hours. 
     - A webhook can be created to send messages to Hipchat or Github. With this email, you can send a message to a group of machines along with a subject, priority, and message body.
     - Results can be attached as .csv files, pdf files, or inline with the message body to ensure the recipient understands what alerts have been fired, at what conditions, and what actions have been taken.
     - You can also create tickets and control alerts based on conditions such as an IP address or machine name. As an example, if a virus outbreak occurs, you do not want every alert to be triggered as it will create a lot of tickets in your system, which will be overwhelming. Such alerts can be controlled from the alert window.

#### Summary Index
Summary indexes store analyses, reports, and summaries computed by Splunk. This is an inexpensive and fast way to run a query for a long period of time. Essentially, it is the default index that Splunk Enterprise uses if there isn't another one specified by the user. Among the key features of the Summary Index is that you can retain the analytics and reports even after the data has gotten older.

#### exclude certain events from being indexed by Splunk
Details
     - In the case where you do not wish to index all of your events in Splunk, what can you do to prevent the entry of those events into Splunk? Debug messages are a good example of this in your application development cycle.  
     - Such debug messages can be excluded by putting them in the null queue. This is achieved by specifying a regex that matches the necessary events and sending the rest to the NULL queue. Null queues are defined at the forwarder level in transforms.conf. Below is an example that drops all events except those containing the debug message. 

#### time zone property in Splunk
A time zone is a crucial factor to consider when searching for events from a fraud or security perspective. This is because Splunk uses the time zone defined by your browser. Your browser then picks up the time zone associated with the machine/computer system you're working on. So, you will not be able to find your desired event if you search for it in the wrong time zone. The timezone is picked up by Splunk when data is entered, and it is particularly important when you are searching and comparing data from different sources. You can, for instance, look for events coming in at 4:00 PM IST, for your London data centre, or for your Singapore data centre, etc. The timezone property is therefore vital when correlating such events

#### app and add-on.
Generally, Splunk applications and add-ons are separate entities, but both have the same extension, i.e., SPL files. 
     - **Splunk Apps:** A Splunk app extends Splunk functionality with its own inbuilt user interface. Each of these apps are separate and serves a specific purpose. Each Splunk app consists of a collection of Splunk knowledge objects (lookups, tags, saved searches, event types, etc). They can also make use of other Splunk apps or add-ons. Multiple apps can be run simultaneously in Splunk. Several apps offer the option of restricting or limiting the amount of information a user can access. By controlling access levels, the user has access to only the information that is necessary for him and not the rest. You can open apps from the Splunk Enterprise homepage or through the App menu or in the Apps section of the Settings page. 
Example: Splunk Enterprise Security App, etc.
     - **Splunk Add-on:** These are types of applications that are built on top of the Splunk platform that add features and functionality to other apps, such as allowing users to import data, map data, save searches, macros. Add-ons typically do not run as standalone apps, rather they are reusable components that support other apps in different scenarios. Most of the time, it is used as a framework, where a team leverages its functionality to some extent and creates something new on top of it. As a rule, they do not have navigable user interfaces. You cannot open an Add-on from the Splunk Enterprise homepage or app menu. 
Examples: Splunk Add-on for Checkpoint OPSEC LEA, Splunk Add-on for EMC VNX or the Splunk Common Information Model Add-on.

#### important configuration files
Details
- **Props.conf:** It configures indexing properties, such as timezone offset, pattern collision priority, custom source type rules, etc.
- **Indexes.conf:** It configures and manages index settings.
- **Inputs.conf:** It is used to set up data inputs.
- **Transforms.conf:** It can be used to configure regex transformations to be performed on data inputs.
- **Server.conf:** There are a variety of settings available for configuring the overall state of the Splunk Enterprise instance.

#### Load Balancer (LB)
A Load Balancer (LB) is used to distribute incoming network traffic across multiple servers to ensure high availability, reliability, and performance. In Splunk, load balancers help distribute search or indexing requests across multiple search heads or indexers.

#### Heavy Forwarder (HF)
A Heavy Forwarder (HF) is a full-instance Splunk component that processes and filters data before forwarding it to indexers. Unlike a Universal Forwarder, it can apply parsing, filtering, and transformation logic to the data before forwarding

#### Splunk Enhanced Solutions
- Splunk Enhanced Solutions refer to premium solutions that extend Splunk's capabilities, such as:
     - Splunk ITSI (IT Service Intelligence) for IT monitoring
     - Splunk ES (Enterprise Security) for security analytics
     - Splunk UBA (User Behavior Analytics) for threat detection

#### Splunk ITSI
Splunk IT Service Intelligence (ITSI) is a premium application that provides machine learning-driven monitoring, predictive analytics, and real-time insights into IT operations. It helps in event correlation, service dependency mapping, and business impact analysis.

#### hosts, source, and sourcetype tab

- Host: Identifies the machine from which the data originated.
- Source: The file, directory, or network port where the data is collected from.
- Sourcetype: Defines the format and structure of the ingested data for proper parsing.

#### Events tab
The Events tab in Splunk provides a view of raw event data. It allows users to inspect, filter, and analyze log events with detailed timestamps, metadata, and indexed fields.

#### Raw Data
Raw data refers to the unprocessed, unstructured log data collected by Splunk before being indexed. It is stored in Splunk's index after parsing, transforming, and enriching.

#### Cluster Master
The Cluster Master is a Splunk component that manages index replication in clustered environments. It ensures data consistency and failover protection in indexer clusters.

#### Search Peers
Search Peers are indexers in a distributed Splunk environment that perform data indexing and respond to search requests initiated by a Search Head.

#### Traditional Index Clusters and Non-replicating Index Clusters

- Traditional Index Clusters: Use index replication to ensure data redundancy and availability.
- Non-replicating Index Clusters: Do not replicate data and rely on external backup strategies.

####  Input Phase, Parsing Phase, and Indexing Phase

- Input Phase: Collects raw data from sources (logs, files, network streams).
- Parsing Phase: Breaks data into events, extracts timestamps, and applies transformations.
- Indexing Phase: Stores the processed data into Splunk indexes for searching.

#### License Meter
The Splunk License Meter tracks the amount of data indexed per day to ensure compliance with Splunk's licensing model.

#### Management Port
Port 8089 is the default management port used for Splunk instance communication and administration.

#### Web Port
Port 8000 is the default port for accessing the Splunk Web UI.

#### Network Port
Port 514 is commonly used for receiving syslog data in Splunk.

#### Index Replication Port
Port 8080 (or a custom-configured port) is used for indexer replication in a clustered Splunk environment.

#### Indexing Port
Port 9997 is the default port used by Universal Forwarders to send data to indexers.

#### KV Store
KV Store (Key-Value Store) in Splunk is a NoSQL-like database that allows for storage and retrieval of structured data in key-value pairs, commonly used in Splunk apps and lookups.

#### Agent
A Splunk Agent is a lightweight software (typically a Universal Forwarder) installed on endpoints to collect and send data to Splunk indexers.

#### Search Head Pooling
Search Head Pooling is an old clustering method where multiple search heads shared a common filesystem for coordination, but it has been deprecated in favor of Search Head Clustering.

#### Search Head Clustering
Search Head Clustering is a high-availability solution where multiple search heads work together to distribute search loads and provide fault tolerance.

#### Universal Forwarder
A Universal Forwarder is a lightweight, low-resource Splunk component that forwards raw data to indexers without performing transformations.

#### Lifecycle (Hot, Warm, Cold, Frozen)

- Hot: Actively written and searchable data.
- Warm: Recently used but not actively written data.
- Cold: Less frequently accessed data.
- Frozen: Archived data, deleted unless moved to a backup.

#### Real-time Dashboards
those display live, continuously updating data streams in Splunk, used for monitoring and alerting.

#### Dynamic Form-based Dashboards
Dynamic dashboards allow users to interact with form inputs (dropdowns, text fields) to modify visualized data in real-time.

#### Fast Mode
Fast Mode optimizes search speed by returning minimal field data, making it suitable for quick analysis.

#### Smart Mode
Smart Mode dynamically switches between Fast Mode and Verbose Mode based on the search query's requirements.

#### Verbose Mode
Verbose Mode retrieves all available fields in search results, providing comprehensive data but at the cost of performance.

#### Abstract command
The abstract command summarizes long text fields to extract key phrases or insights.

#### Erex command
The erex (example regex) command helps users generate regular expressions automatically from sample data.

#### Addtotals command?
The addtotals command calculates and appends row-wise total values across specified numeric fields.

#### Accum command?
The accum command creates a cumulative sum of a numeric field over time.

#### Filldown command?
The filldown command fills empty field values with the last known non-null value in search results

#### Typer command
The typer command is used to predict the field type of indexed data based on existing values. It helps in automatic data classification.

#### Rename command
The rename command is used to rename fields in search results, making them more readable or aligning them with standard naming conventions.
Example:
     - index=web_logs | rename clientip AS IP_Address

#### Anomalies command
The anomalies command is used to detect outliers or unusual patterns in search results based on statistical models.

#### License Violation
A license violation occurs when the indexed data exceeds the daily limit set by the Splunk license. If violations exceed the allowed limit, search capabilities may be restricted.

#### Time Zone property
The Time Zone property ensures that logs from different sources are correctly aligned based on their timestamps for accurate analysis.

#### sourcetype in Splunk
A sourcetype defines the format and structure of incoming data, enabling Splunk to parse it correctly. Examples include access_combined, json, and csv.

#### Splunk Alerts

Splunk Alerts notify users when specific conditions are met.
- Types of alerts:
     - Scheduled Alerts (Triggered at set intervals)
     - Real-time Alerts (Triggered instantly on matching conditions)
     - Rolling Window Alerts (Triggered based on events in a time window)
     - Alert actions include:
          - Email notifications
          - Script execution
          - Webhook calls
          - Triggering another search

#### configuration files precedence - directory

Splunk follows a hierarchical order for .conf file processing:
     - System Local Directory (/opt/splunk/etc/system/local/) – Highest priority
     - App Local Directory (/opt/splunk/etc/apps/<app>/local/)
     - App Default Directory (/opt/splunk/etc/apps/<app>/default/)
     - System Default Directory (/opt/splunk/etc/system/default/) – Lowest priority

#### Replication Factor
The Replication Factor determines how many copies of indexed data exist in an indexer cluster to ensure redundancy and failover protection.

#### components of Splunk
- Details
     - Indexer (Stores and processes data)
     - Search Head (User interface for searching and reporting)
     - Forwarder (Collects and sends logs)
     - Deployment Server (Manages Splunk configurations)
     
#### Splunk Indexer and stages
- A Splunk Indexer processes, stores, and retrieves indexed data.
- Indexing Stages:
     - Input Phase (Collects raw data)
     - Parsing Phase (Extracts timestamps and applies transformations)
     - Indexing Phase (Stores processed data in indexes)
       
#### Forwarder
- A Splunk Forwarder collects and forwards logs to indexers.
- Types:
     - Universal Forwarder (UF): Lightweight, no parsing
     - Heavy Forwarder (HF): Full Splunk instance, can filter & route data

#### Splunk Licenses
- list
     - Enterprise License (Full-featured)
     - Free License (Limited functionality)
     - Trial License (Enterprise features for a limited period)
     - Dev/Test License (For development environments)
     - Forwarder License (Only forwarding, no searching)
     
#### Splunk App
A Splunk App is a packaged set of dashboards, reports, alerts, and configurations that extend Splunk's functionality.

#### features are not available in Splunk Free
- list
     - No alerting
     - No distributed search
     - No authentication & role-based access control
     
#### License Master is unreachable
If the License Master is down for more than 72 hours, indexing stops, but searching continues.

#### Summary Index
A Summary Index stores precomputed results from scheduled searches to speed up reporting.

#### DB Connect
Splunk DB Connect integrates Splunk with relational databases for querying and indexing structured data.

#### dashboards
- list 
     - Real-time Dashboards (Live updating)
     - Dynamic Dashboards (User-driven input)
     - Static Dashboards (Pre-defined reports)

#### Buckets
- Buckets store indexed data and follow this lifecycle:
     - Hot: Active data being written
     - Warm: Recently used data
     - Cold: Older data
     - Frozen: Archived or deleted data

#### stats 
stats produces final aggregated results.

#### eventstats
eventstats adds aggregated values to each event without changing the original data.

#### Licenses specify
They define indexing limits, feature availability, and expiration.

#### restart Splunk Web Server
     - splunk restart splunkweb

#### restart Splunk Daemon
     - splunk restart

#### enable to start on boot
     - splunk enable boot-start

#### Source Type
A source type defines how Splunk interprets incoming data, helping in parsing and indexing.

#### btool
     - splunk btool inputs list --debug

#### Add-on
Provides data inputs and configurations but lacks a UI

#### Fishbucket
Fishbucket is an internal index that tracks file read positions to prevent duplicate data ingestion.

#### Dispatch Directory
The Dispatch Directory stores search artifacts like job results and logs.

#### MapReduce Algorithm?
- The MapReduce algorithm is a programming model used for processing large datasets across distributed systems. It consists of two main steps:
     - Map: Splits data into key-value pairs and processes them in parallel.
     - Reduce: Aggregates the mapped data to generate the final result.
     - Splunk uses a similar approach internally for distributed searches.

#### SDK
Provides APIs (Python, Java, JavaScript) to integrate Splunk with other applications programmatically.

#### Framework
A development environment for building Splunk Apps using web technologies like Django or JavaScript.

#### configuration files
- Some key configuration files in Splunk include:
     - indexes.conf - Manages indexes and storage settings
     - inputs.conf - Defines data inputs
     - props.conf - Configures source types and field extractions
     - transforms.conf - Defines field transformations and filtering
     - server.conf - Configures Splunk server settings

#### Splunk Buckets
Buckets are directories where Splunk stores indexed data. - - The bucket lifecycle stages:
     - Hot – Actively written
     - Warm – Recently used, read-only
     - Cold – Older, stored on cheaper storage
     - Frozen – Archived or deleted

#### architecture.
- Splunk has the following components:
     - Forwarders – Collect data from sources
     - Indexers – Store and process data
     - Search Heads – Provide UI and run searches
     - Deployment Server – Manages configurations


- What are Splunk buckets Explain the bucket lifecycle
- What are the common port numbers used by Splunk
- What are the components of Splunk Explain Splunk architecture
- What are the features that are not available in Splunk free
- What Are Types Of Splunk Licenses
- What command is used to enable and disable Splunk to boot start
- What do you understand by Splunk Administration What is the latest version of the tool Splunk
- What is a syslog server
- What is bucket  How data ages in Splunk 
- What is CIM and what is it used for
- What is command for restarting just the splunk daemon
- What is crontab
- What is indexes.conf
- What is inputs.conf
- What is props.conf
- What is server.conf
- What is Splunk Administration
- What is Splunk cloud administration
- What is Splunk
- What is the difference between Search time and Index time field extractions
- What is the eval command
- What is the null queue
- What is the source type in Splunk
- What is the use of DB Connect in Splunk
- What is the use of License Master in Splunk
- What is the use of sort command
- What is the use of the deployment server in Splunk administration
- What is transforms.conf
- What is use of Time Zone property in Splunk and when it is required
- What will you do in case License Master is unreachable
- What would you use to edit contents of the file in Linux Describe some of the important commands mode in vi editor
- What would you use to view contents of a large file How to copy/remove file How to look for help on a Linux
- What are types of field extraction. How to mask a data in either of case
- What do you mean by roles based access control
- What is null queue
- What are the types of search modes supported in splunk
- What is difference between source & source type
- What is join command and what are various flavours of join command.
- What is Splunk Why Splunk is used for analysing machine data
- What are the benefits of getting data using forwarders
- What happens if License master is unreachable
- What is the command to get list of configuration files in Splunk
- What is the command to stop and start Splunk service
- What is index bucket What are all stages of buckets
- What are important configuration files in Splunk
- What is global file precedence in Splunk
- What is difference between stats and timechart command
- What is lookup command
- What is the role of Deployment server
- What are the default fields in Splunk
- What is Search Factor (SF)
- What is the difference between Splunk apps and add-ons
- What restrict to find data
- What happens when to increase the data limitation
- What is used for building a ranking
- What is used to process huge data sets
- What is used to track internally
- what is the use of DB connect
- What is single-instance storage
- What is used for collecting the logs
- What is known as a central resource for searching
- What is used to conduct the group of field details
- What is Occasion management
- What is Transaction
- What is splunk latest version
- What is splunk Success Framework
- What is pivot
- Name the common port numbers used by Splunk.
- Name the types of search modes supported in Splunk.
- Name the items for migration
- Name the disadvantages of Splunk
- Name the features of a knowledge object
- Name the uses of Knowledge object
- Explain about Splunk Enterprise Security (ES)
- Explain about Splunk User Behavior Analytics (UBA)
- Explain Splunk Index Time Process
- Explain Stats vs Transaction commands.
- Explain search head pool & search head clusters
- Explain Splunk app How it differs from Add-on
- Explain the use of License Master in Splunk
- Why is Splunk used for analyzing machine data
- Why do companies adopt Splunk
- Why is Splunk administration used for the analysis of machine data
- Why is Splunk used for the analysis of machine data
- Why people prefer Splunk as compared to other open-source options
- Why use only Splunk
- Why DB connect is important
- Which is the latest Splunk version in use
- Which role can create a data model
- Which Splunk Roles can share the same machine
- Which command is used to the filtering results category- explain
- Which role can create data model
- Which app ships with splunk enterprise
- How to trouble shooting Splunk errors in splunk
- How Does Splunk Work
- How Is Splunk Deployed
- How many type data input supported by Splunk
- How can you troubleshoot Splunk performance issues
- How to know when Splunk has completed indexing a log file
- How can you add folder access logs from a Windows machine to Splunk
- How does Splunk avoid duplicate indexing of logs
- How to troubleshoot Splunk performance issues
- How does Splunk determine 1 day, from a licensing perspective
- How are Forwarder Licenses purchased
- How to disable Splunk boot-start
- How to reset Splunk Admin password
- How to disable Splunk Launch Message
- How to clear Splunk Search History
- How do I exclude some events from being indexed by Splunk
- How to set the default search time in Splunk 6
- How would you handle/troubleshoot Splunk License Violation Warning
- How can we extract fields
- How do you reset Splunk Admin Password
- How does Splunk help in the Organization
- How is a career path in Splunk Administration
- How many types of Splunk forwarders are there
- How Splunk helps the enterprise
- How to reset splunk password
- How do you log in to a remote Unix box using ssh
- How you will uncompressed the file How to install Splunk/app using the Splunk Enterprise .tgz file
- How to use btool for splunk conf file approach
- How to turn down a peer without affecting any other peer of cluster
- How to show which deployment server in configured to pull data from
- How to see all the license pool active in our Splunk environment
- How do we convert unix time into string and string back to unix time format
- How do we find total number of host or source type reporting splunk instance. Report should consider host across the cluster
- How can you exclude some events from being indexed in Splunk
- How do we sync and deploy configurational files and updates across multiple deployment servers in a large multi-layered clustered
- How to map the keys and values
- How the company manages the data
- How to ignore the incoming data
- How to remove all the events
- How to connect two BLE devices
- How to recover a non-functioning group
- When to use auto_high_volume in splunk
- Where is Splunk Default Configuration stored
- Where does Splunk default configuration file located
- Where to keep the listed data in directories
- Who are the top direct competitors to Splunk
- Who are the competitors of Splunk in the market Why is Splunk efficient
- Who is responsible for the right quantity of data
- Who analyzes data in a backup system
- Can you name a few most important configuration files in Splunk
- Can you tell some use cases of Knowledge Objects
- CLI to validate bundles
- Command to change splunkweb port to 9000 via CLI
- Create new app from templet
- Discuss about the sequence in which splunk upgrade can be done in a clustered environment
- Does Splunk administration support user authentication systems
- Draw a difference between Splunk app and Splunk add-on
- Effect of a non-functioning cluster
- Give a few use cases of Knowledge Objects.
- If I want to add folder access logs from a windows machine to Splunk, how do I do it
- If you wish to use a free version of Splunk, which features are lacking
- List some of the features that is lagging in Splunkfree
- Mention the use of the Dedup command of Splunk
- Rollback your aplunk web configuration bundle to previous version
- Types Of Splunk Forwarder
