<p style="text-align:center">
    <a href="https://skills.network/?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01" target="_blank">
    <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/assets/logos/SN_web_lightmode.png" width="200" alt="Skills Network Logo"  />
    </a>
</p>

<h1 align=center><font size = 5>Assignment: SQL Notebook for Peer Assignment</font></h1>

Estimated time needed: **60** minutes.

## Introduction

Using this Python notebook you will:

1.  Understand the Spacex DataSet
2.  Load the dataset  into the corresponding table in a Db2 database
3.  Execute SQL queries to answer assignment questions


## Overview of the DataSet

SpaceX has gained worldwide attention for a series of historic milestones.

It is the only private company ever to return a spacecraft from low-earth orbit, which it first accomplished in December 2010.
SpaceX advertises Falcon 9 rocket launches on its website with a cost of 62 million dollars wheras other providers cost upward of 165 million dollars each, much of the savings is because Space X can reuse the first stage.

Therefore if we can determine if the first stage will land, we can determine the cost of a launch.

This information can be used if an alternate company wants to bid against SpaceX for a rocket launch.

This dataset includes a record for each payload carried during a SpaceX mission into outer space.


### Download the datasets

This assignment requires you to load the spacex dataset.

In many cases the dataset to be analyzed is available as a .CSV (comma separated values) file, perhaps on the internet. Click on the link below to download and save the dataset (.CSV file):

<a href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/data/Spacex.csv?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01" target="_blank">Spacex DataSet</a>


### Store the dataset in database table

**it is highly recommended to manually load the table using the database console LOAD tool in DB2**.

<img src = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/images/spacexload.png">

Now open the Db2 console, open the LOAD tool, Select / Drag the .CSV file for the  dataset, Next create a New Table, and then follow the steps on-screen instructions to load the data. Name the new table as follows:

**SPACEXDATASET**

**Follow these steps while using old DB2 UI which is having Open Console Screen**

**Note:While loading Spacex dataset, ensure that detect datatypes is disabled. Later click on the pencil icon(edit option).**

1.  Change the Date Format by manually typing DD-MM-YYYY and timestamp format as DD-MM-YYYY HH\:MM:SS

2.  Change the PAYLOAD_MASS\_\_KG\_  datatype  to INTEGER.

<img src = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/images/spacexload2.png">


**Changes to be considered when having DB2 instance with the new UI having Go to UI screen**

*   Refer to this insruction in this <a href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Labs_Coursera_V5/labs/Lab%20-%20Sign%20up%20for%20IBM%20Cloud%20-%20Create%20Db2%20service%20instance%20-%20Get%20started%20with%20the%20Db2%20console/instructional-labs.md.html?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01">link</a> for viewing  the new  Go to UI screen.

*   Later click on **Data link(below SQL)**  in the Go to UI screen  and click on **Load Data** tab.

*   Later browse for the downloaded spacex file.

<img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/images/browsefile.png" width="800"/>

*   Once done select the schema andload the file.

 <img src="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/images/spacexload3.png" width="800"/>



```python
!pip install sqlalchemy==1.3.9
#!pip install sqlalchemy==1.3.9

```

    Collecting sqlalchemy==1.3.9
      Downloading SQLAlchemy-1.3.9.tar.gz (6.0 MB)
    [2K     [90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━[0m [32m6.0/6.0 MB[0m [31m53.1 MB/s[0m eta [36m0:00:00[0m00:01[0m00:01[0m
    [?25h  Preparing metadata (setup.py) ... [?25ldone
    [?25hBuilding wheels for collected packages: sqlalchemy
      Building wheel for sqlalchemy (setup.py) ... [?25ldone
    [?25h  Created wheel for sqlalchemy: filename=SQLAlchemy-1.3.9-cp37-cp37m-linux_x86_64.whl size=1159122 sha256=72b996f536dd0e615a7b43dc4fe021291b56d131104efaf042ee8dd0d1d85ec7
      Stored in directory: /home/jupyterlab/.cache/pip/wheels/ef/95/ac/c232f83b415900c26553c64266e1a2b2863bc63e7a5d606c7e
    Successfully built sqlalchemy
    Installing collected packages: sqlalchemy
      Attempting uninstall: sqlalchemy
        Found existing installation: SQLAlchemy 1.3.24
        Uninstalling SQLAlchemy-1.3.24:
          Successfully uninstalled SQLAlchemy-1.3.24
    Successfully installed sqlalchemy-1.3.9


### Connect to the database

Let us first load the SQL extension and establish a connection with the database



```python
%load_ext sql
```


```python
import csv, sqlite3

con = sqlite3.connect("my_data1.db")
cur = con.cursor()
```


```python
!pip install -q pandas==1.1.5
```


```python
%sql sqlite:///my_data1.db
```




    'Connected: @my_data1.db'




```python
import pandas as pd
df = pd.read_csv("https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/data/Spacex.csv")
df.to_sql("SPACEXTBL", con, if_exists='replace', index=False, method ="multi")
```

    /home/jupyterlab/conda/envs/python/lib/python3.7/site-packages/pandas/core/generic.py:2882: UserWarning: The spaces in these column names will not be changed. In pandas versions < 0.14, spaces were converted to underscores.
      both result in 0.1234 being formatted as 0.12.


## Tasks

Now write and execute SQL queries to solve the assignment tasks.

**Note: If the column names are in mixed case enclose it in double quotes
For Example "Landing_Outcome"**

### Task 1

##### Display the names of the unique launch sites  in the space mission



```python
%sql SELECT * FROM SPACEXTBL
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>Date</th>
            <th>Time (UTC)</th>
            <th>Booster_Version</th>
            <th>Launch_Site</th>
            <th>Payload</th>
            <th>PAYLOAD_MASS__KG_</th>
            <th>Orbit</th>
            <th>Customer</th>
            <th>Mission_Outcome</th>
            <th>Landing _Outcome</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>04-06-2010</td>
            <td>18:45:00</td>
            <td>F9 v1.0  B0003</td>
            <td>CCAFS LC-40</td>
            <td>Dragon Spacecraft Qualification Unit</td>
            <td>0</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Failure (parachute)</td>
        </tr>
        <tr>
            <td>08-12-2010</td>
            <td>15:43:00</td>
            <td>F9 v1.0  B0004</td>
            <td>CCAFS LC-40</td>
            <td>Dragon demo flight C1, two CubeSats, barrel of Brouere cheese</td>
            <td>0</td>
            <td>LEO (ISS)</td>
            <td>NASA (COTS) NRO</td>
            <td>Success</td>
            <td>Failure (parachute)</td>
        </tr>
        <tr>
            <td>22-05-2012</td>
            <td>07:44:00</td>
            <td>F9 v1.0  B0005</td>
            <td>CCAFS LC-40</td>
            <td>Dragon demo flight C2</td>
            <td>525</td>
            <td>LEO (ISS)</td>
            <td>NASA (COTS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>08-10-2012</td>
            <td>00:35:00</td>
            <td>F9 v1.0  B0006</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-1</td>
            <td>500</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>01-03-2013</td>
            <td>15:10:00</td>
            <td>F9 v1.0  B0007</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-2</td>
            <td>677</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>29-09-2013</td>
            <td>16:00:00</td>
            <td>F9 v1.1  B1003</td>
            <td>VAFB SLC-4E</td>
            <td>CASSIOPE</td>
            <td>500</td>
            <td>Polar LEO</td>
            <td>MDA</td>
            <td>Success</td>
            <td>Uncontrolled (ocean)</td>
        </tr>
        <tr>
            <td>03-12-2013</td>
            <td>22:41:00</td>
            <td>F9 v1.1</td>
            <td>CCAFS LC-40</td>
            <td>SES-8</td>
            <td>3170</td>
            <td>GTO</td>
            <td>SES</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>06-01-2014</td>
            <td>22:06:00</td>
            <td>F9 v1.1</td>
            <td>CCAFS LC-40</td>
            <td>Thaicom 6</td>
            <td>3325</td>
            <td>GTO</td>
            <td>Thaicom</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>18-04-2014</td>
            <td>19:25:00</td>
            <td>F9 v1.1</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-3</td>
            <td>2296</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Controlled (ocean)</td>
        </tr>
        <tr>
            <td>14-07-2014</td>
            <td>15:15:00</td>
            <td>F9 v1.1</td>
            <td>CCAFS LC-40</td>
            <td>OG2 Mission 1  6 Orbcomm-OG2 satellites</td>
            <td>1316</td>
            <td>LEO</td>
            <td>Orbcomm</td>
            <td>Success</td>
            <td>Controlled (ocean)</td>
        </tr>
        <tr>
            <td>05-08-2014</td>
            <td>08:00:00</td>
            <td>F9 v1.1</td>
            <td>CCAFS LC-40</td>
            <td>AsiaSat 8</td>
            <td>4535</td>
            <td>GTO</td>
            <td>AsiaSat</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>07-09-2014</td>
            <td>05:00:00</td>
            <td>F9 v1.1 B1011</td>
            <td>CCAFS LC-40</td>
            <td>AsiaSat 6</td>
            <td>4428</td>
            <td>GTO</td>
            <td>AsiaSat</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>21-09-2014</td>
            <td>05:52:00</td>
            <td>F9 v1.1 B1010</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-4</td>
            <td>2216</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Uncontrolled (ocean)</td>
        </tr>
        <tr>
            <td>10-01-2015</td>
            <td>09:47:00</td>
            <td>F9 v1.1 B1012</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-5</td>
            <td>2395</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Failure (drone ship)</td>
        </tr>
        <tr>
            <td>11-02-2015</td>
            <td>23:03:00</td>
            <td>F9 v1.1 B1013</td>
            <td>CCAFS LC-40</td>
            <td>DSCOVR</td>
            <td>570</td>
            <td>HEO</td>
            <td>U.S. Air Force NASA NOAA</td>
            <td>Success</td>
            <td>Controlled (ocean)</td>
        </tr>
        <tr>
            <td>02-03-2015</td>
            <td>03:50:00</td>
            <td>F9 v1.1 B1014</td>
            <td>CCAFS LC-40</td>
            <td>ABS-3A Eutelsat 115 West B</td>
            <td>4159</td>
            <td>GTO</td>
            <td>ABS Eutelsat</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>14-04-2015</td>
            <td>20:10:00</td>
            <td>F9 v1.1 B1015</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-6</td>
            <td>1898</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Failure (drone ship)</td>
        </tr>
        <tr>
            <td>27-04-2015</td>
            <td>23:03:00</td>
            <td>F9 v1.1 B1016</td>
            <td>CCAFS LC-40</td>
            <td>Turkmen 52 / MonacoSAT</td>
            <td>4707</td>
            <td>GTO</td>
            <td>Turkmenistan National Space Agency</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>28-06-2015</td>
            <td>14:21:00</td>
            <td>F9 v1.1 B1018</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-7</td>
            <td>1952</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Failure (in flight)</td>
            <td>Precluded (drone ship)</td>
        </tr>
        <tr>
            <td>22-12-2015</td>
            <td>01:29:00</td>
            <td>F9 FT B1019</td>
            <td>CCAFS LC-40</td>
            <td>OG2 Mission 2  11 Orbcomm-OG2 satellites</td>
            <td>2034</td>
            <td>LEO</td>
            <td>Orbcomm</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>17-01-2016</td>
            <td>18:42:00</td>
            <td>F9 v1.1 B1017</td>
            <td>VAFB SLC-4E</td>
            <td>Jason-3</td>
            <td>553</td>
            <td>LEO</td>
            <td>NASA (LSP) NOAA CNES</td>
            <td>Success</td>
            <td>Failure (drone ship)</td>
        </tr>
        <tr>
            <td>04-03-2016</td>
            <td>23:35:00</td>
            <td>F9 FT B1020</td>
            <td>CCAFS LC-40</td>
            <td>SES-9</td>
            <td>5271</td>
            <td>GTO</td>
            <td>SES</td>
            <td>Success</td>
            <td>Failure (drone ship)</td>
        </tr>
        <tr>
            <td>08-04-2016</td>
            <td>20:43:00</td>
            <td>F9 FT B1021.1</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-8</td>
            <td>3136</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>06-05-2016</td>
            <td>05:21:00</td>
            <td>F9 FT B1022</td>
            <td>CCAFS LC-40</td>
            <td>JCSAT-14</td>
            <td>4696</td>
            <td>GTO</td>
            <td>SKY Perfect JSAT Group</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>27-05-2016</td>
            <td>21:39:00</td>
            <td>F9 FT B1023.1</td>
            <td>CCAFS LC-40</td>
            <td>Thaicom 8</td>
            <td>3100</td>
            <td>GTO</td>
            <td>Thaicom</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>15-06-2016</td>
            <td>14:29:00</td>
            <td>F9 FT B1024</td>
            <td>CCAFS LC-40</td>
            <td>ABS-2A Eutelsat 117 West B</td>
            <td>3600</td>
            <td>GTO</td>
            <td>ABS Eutelsat</td>
            <td>Success</td>
            <td>Failure (drone ship)</td>
        </tr>
        <tr>
            <td>18-07-2016</td>
            <td>04:45:00</td>
            <td>F9 FT B1025.1</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-9</td>
            <td>2257</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>14-08-2016</td>
            <td>05:26:00</td>
            <td>F9 FT B1026</td>
            <td>CCAFS LC-40</td>
            <td>JCSAT-16</td>
            <td>4600</td>
            <td>GTO</td>
            <td>SKY Perfect JSAT Group</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>14-01-2017</td>
            <td>17:54:00</td>
            <td>F9 FT B1029.1</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT 1</td>
            <td>9600</td>
            <td>Polar LEO</td>
            <td>Iridium Communications</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>19-02-2017</td>
            <td>14:39:00</td>
            <td>F9 FT B1031.1</td>
            <td>KSC LC-39A</td>
            <td>SpaceX CRS-10</td>
            <td>2490</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>16-03-2017</td>
            <td>06:00:00</td>
            <td>F9 FT B1030</td>
            <td>KSC LC-39A</td>
            <td>EchoStar 23</td>
            <td>5600</td>
            <td>GTO</td>
            <td>EchoStar</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>30-03-2017</td>
            <td>22:27:00</td>
            <td>F9 FT  B1021.2</td>
            <td>KSC LC-39A</td>
            <td>SES-10</td>
            <td>5300</td>
            <td>GTO</td>
            <td>SES</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>01-05-2017</td>
            <td>11:15:00</td>
            <td>F9 FT B1032.1</td>
            <td>KSC LC-39A</td>
            <td>NROL-76</td>
            <td>5300</td>
            <td>LEO</td>
            <td>NRO</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>15-05-2017</td>
            <td>23:21:00</td>
            <td>F9 FT B1034</td>
            <td>KSC LC-39A</td>
            <td>Inmarsat-5 F4</td>
            <td>6070</td>
            <td>GTO</td>
            <td>Inmarsat</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>03-06-2017</td>
            <td>21:07:00</td>
            <td>F9 FT B1035.1</td>
            <td>KSC LC-39A</td>
            <td>SpaceX CRS-11</td>
            <td>2708</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>23-06-2017</td>
            <td>19:10:00</td>
            <td>F9 FT  B1029.2</td>
            <td>KSC LC-39A</td>
            <td>BulgariaSat-1</td>
            <td>3669</td>
            <td>GTO</td>
            <td>Bulsatcom</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>25-06-2017</td>
            <td>20:25:00</td>
            <td>F9 FT B1036.1</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT 2</td>
            <td>9600</td>
            <td>LEO</td>
            <td>Iridium Communications</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>05-07-2017</td>
            <td>23:38:00</td>
            <td>F9 FT B1037</td>
            <td>KSC LC-39A</td>
            <td>Intelsat 35e</td>
            <td>6761</td>
            <td>GTO</td>
            <td>Intelsat</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>14-08-2017</td>
            <td>16:31:00</td>
            <td>F9 B4 B1039.1</td>
            <td>KSC LC-39A</td>
            <td>SpaceX CRS-12</td>
            <td>3310</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>24-08-2017</td>
            <td>18:51:00</td>
            <td>F9 FT B1038.1</td>
            <td>VAFB SLC-4E</td>
            <td>Formosat-5</td>
            <td>475</td>
            <td>SSO</td>
            <td>NSPO</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>07-09-2017</td>
            <td>14:00:00</td>
            <td>F9 B4 B1040.1</td>
            <td>KSC LC-39A</td>
            <td>Boeing X-37B OTV-5</td>
            <td>4990</td>
            <td>LEO</td>
            <td>U.S. Air Force</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>09-10-2017</td>
            <td>12:37:00</td>
            <td>F9 B4 B1041.1</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT 3</td>
            <td>9600</td>
            <td>Polar LEO</td>
            <td>Iridium Communications</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>11-10-2017</td>
            <td>22:53:00</td>
            <td>F9 FT  B1031.2</td>
            <td>KSC LC-39A</td>
            <td>SES-11 / EchoStar 105</td>
            <td>5200</td>
            <td>GTO</td>
            <td>SES EchoStar</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>30-10-2017</td>
            <td>19:34:00</td>
            <td>F9 B4 B1042.1</td>
            <td>KSC LC-39A</td>
            <td>Koreasat 5A</td>
            <td>3500</td>
            <td>GTO</td>
            <td>KT Corporation</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>15-12-2017</td>
            <td>15:36:00</td>
            <td>F9 FT  B1035.2</td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-13</td>
            <td>2205</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>23-12-2017</td>
            <td>01:27:00</td>
            <td>F9 FT  B1036.2</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT 4</td>
            <td>9600</td>
            <td>Polar LEO</td>
            <td>Iridium Communications</td>
            <td>Success</td>
            <td>Controlled (ocean)</td>
        </tr>
        <tr>
            <td>08-01-2018</td>
            <td>01:00:00</td>
            <td>F9 B4 B1043.1</td>
            <td>CCAFS SLC-40</td>
            <td>Zuma</td>
            <td>5000</td>
            <td>LEO</td>
            <td>Northrop Grumman</td>
            <td>Success (payload status unclear)</td>
            <td>Success (ground pad)</td>
        </tr>
        <tr>
            <td>31-01-2018</td>
            <td>21:25:00</td>
            <td>F9 FT  B1032.2</td>
            <td>CCAFS SLC-40</td>
            <td>GovSat-1 / SES-16</td>
            <td>4230</td>
            <td>GTO</td>
            <td>SES</td>
            <td>Success</td>
            <td>Controlled (ocean)</td>
        </tr>
        <tr>
            <td>22-02-2018</td>
            <td>14:17:00</td>
            <td>F9 FT  B1038.2</td>
            <td>VAFB SLC-4E</td>
            <td>Paz  Tintin A &amp; B</td>
            <td>2150</td>
            <td>SSO</td>
            <td>Hisdesat exactEarth SpaceX</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>06-03-2018</td>
            <td>05:33:00</td>
            <td>F9 B4 B1044</td>
            <td>CCAFS SLC-40</td>
            <td>Hispasat 30W-6  PODSat</td>
            <td>6092</td>
            <td>GTO</td>
            <td>Hispasat  NovaWurks</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>30-03-2018</td>
            <td>14:14:00</td>
            <td>F9 B4  B1041.2</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT 5</td>
            <td>9600</td>
            <td>Polar LEO</td>
            <td>Iridium Communications</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>02-04-2018</td>
            <td>20:30:00</td>
            <td>F9 B4  B1039.2</td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-14</td>
            <td>2647</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>18-04-2018</td>
            <td>22:51:00</td>
            <td>F9 B4 B1045.1</td>
            <td>CCAFS SLC-40</td>
            <td>Transiting Exoplanet Survey Satellite (TESS)</td>
            <td>362</td>
            <td>HEO</td>
            <td>NASA (LSP)</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>11-05-2018</td>
            <td>20:14:00</td>
            <td>F9 B5  B1046.1</td>
            <td>KSC LC-39A</td>
            <td>Bangabandhu-1</td>
            <td>3600</td>
            <td>GTO</td>
            <td>Thales-Alenia/BTRC</td>
            <td>Success</td>
            <td>Success (drone ship)</td>
        </tr>
        <tr>
            <td>22-05-2018</td>
            <td>19:47:58</td>
            <td>F9 B4  B1043.2</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT 6   GRACE-FO 1, 2</td>
            <td>6460</td>
            <td>Polar LEO</td>
            <td>Iridium Communications GFZ ‚ NASA</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>04-06-2018</td>
            <td>04:45:00</td>
            <td>F9 B4  B1040.2</td>
            <td>CCAFS SLC-40</td>
            <td>SES-12</td>
            <td>5384</td>
            <td>GTO</td>
            <td>SES</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>29-06-2018</td>
            <td>09:42:00</td>
            <td>F9 B4 B1045.2</td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-15</td>
            <td>2697</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>22-07-2018</td>
            <td>05:50:00</td>
            <td>F9 B5B1047.1</td>
            <td>CCAFS SLC-40</td>
            <td>Telstar 19V</td>
            <td>7075</td>
            <td>GTO</td>
            <td>Telesat</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>25-07-2018</td>
            <td>11:39:00</td>
            <td>F9 B5B1048.1</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT-7</td>
            <td>9600</td>
            <td>Polar LEO</td>
            <td>Iridium Communications</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>07-08-2018</td>
            <td>05:18:00</td>
            <td>F9 B5 B1046.2</td>
            <td>CCAFS SLC-40</td>
            <td>Merah Putih </td>
            <td>5800</td>
            <td>GTO</td>
            <td>Telkom Indonesia</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>10-09-2018</td>
            <td>04:45:00</td>
            <td>F9 B5B1049.1</td>
            <td>CCAFS SLC-40</td>
            <td>Telstar 18V / Apstar-5C</td>
            <td>7060</td>
            <td>GTO</td>
            <td>Telesat</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>08-10-2018</td>
            <td>02:22:00</td>
            <td>F9 B5 B1048.2</td>
            <td>VAFB SLC-4E</td>
            <td>SAOCOM 1A</td>
            <td>3000</td>
            <td>SSO</td>
            <td>CONAE</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>15-11-2018</td>
            <td>20:46:00</td>
            <td>F9 B5 B1047.2</td>
            <td>KSC LC-39A</td>
            <td>Es hail 2</td>
            <td>5300</td>
            <td>GTO</td>
            <td>Es hailSat</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>03-12-2018</td>
            <td>18:34:05</td>
            <td>F9 B5 B1046.3</td>
            <td>VAFB SLC-4E</td>
            <td>SSO-A </td>
            <td>4000</td>
            <td>SSO</td>
            <td>Spaceflight Industries</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>05-12-2018</td>
            <td>18:16:00</td>
            <td>F9 B5B1050</td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-16</td>
            <td>2500</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Failure</td>
        </tr>
        <tr>
            <td>23-12-2018</td>
            <td>13:51:00</td>
            <td>F9 B5B1054</td>
            <td>CCAFS SLC-40</td>
            <td>GPS III-01 </td>
            <td>4400</td>
            <td>MEO</td>
            <td>USAF</td>
            <td>Success </td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>11-01-2019</td>
            <td>15:31:00</td>
            <td>F9 B5 B1049.2</td>
            <td>VAFB SLC-4E</td>
            <td>Iridium NEXT-8</td>
            <td>9600</td>
            <td>Polar LEO</td>
            <td>Iridium Communications</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>22-02-2019</td>
            <td>01:45:00</td>
            <td>F9 B5 B1048.3</td>
            <td>CCAFS SLC-40</td>
            <td>Nusantara Satu, Beresheet Moon lander, S5</td>
            <td>4850</td>
            <td>GTO</td>
            <td>PSN, SpaceIL / IAI</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>02-03-2019</td>
            <td>07:49:00</td>
            <td>F9 B5B1051.1</td>
            <td>KSC LC-39A</td>
            <td>Crew Dragon Demo-1, SpaceX CRS-17 </td>
            <td>12055</td>
            <td>LEO (ISS)</td>
            <td>NASA (CCD) </td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>04-05-2019</td>
            <td>06:48:00</td>
            <td>F9 B5B1056.1 </td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-17, Starlink v0.9</td>
            <td>2495</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>24-05-2019</td>
            <td>02:30:00</td>
            <td>F9 B5 B1049.3</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink v0.9, RADARSAT Constellation</td>
            <td>13620</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>12-06-2019</td>
            <td>14:17:00</td>
            <td>F9 B5 B1051.2 </td>
            <td>VAFB SLC-4E</td>
            <td>RADARSAT Constellation, SpaceX CRS-18 </td>
            <td>4200</td>
            <td>SSO</td>
            <td>Canadian Space Agency (CSA)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>25-07-2019</td>
            <td>22:01:00</td>
            <td>F9 B5 B1056.2 </td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-18, AMOS-17 </td>
            <td>2268</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>06-08-2019</td>
            <td>23:23:00</td>
            <td>F9 B5 B1047.3 </td>
            <td>CCAFS SLC-40</td>
            <td>AMOS-17, Starlink 1 v1.0 </td>
            <td>6500</td>
            <td>GTO</td>
            <td>Spacecom</td>
            <td>Success</td>
            <td>No attempt </td>
        </tr>
        <tr>
            <td>11-11-2019</td>
            <td>14:56:00</td>
            <td>F9 B5 B1048.4</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 1 v1.0, SpaceX CRS-19 </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>05-12-2019</td>
            <td>17:29:00</td>
            <td>F9 B5B1059.1</td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-19, JCSat-18 / Kacific 1 </td>
            <td>2617</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS), Kacific 1</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>17-12-2019</td>
            <td>00:10:00</td>
            <td>F9 B5 B1056.3 </td>
            <td>CCAFS SLC-40</td>
            <td>JCSat-18 / Kacific 1, Starlink 2 v1.0 </td>
            <td>6956</td>
            <td>GTO</td>
            <td>Sky Perfect JSAT, Kacific 1</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>07-01-2020</td>
            <td>02:33:00</td>
            <td>F9 B5 B1049.4</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 2 v1.0, Crew Dragon in-flight abort test </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>19-01-2020</td>
            <td>15:30:00</td>
            <td>F9 B5 B1046.4</td>
            <td>KSC LC-39A</td>
            <td>Crew Dragon in-flight abort test, Starlink 3 v1.0 </td>
            <td>12050</td>
            <td>Sub-orbital</td>
            <td>NASA (CTS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>29-01-2020</td>
            <td>14:07:00</td>
            <td>F9 B5 B1051.3</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 3 v1.0, Starlink 4 v1.0 </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>17-02-2020</td>
            <td>15:05:00</td>
            <td>F9 B5 B1056.4</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 4 v1.0, SpaceX CRS-20</td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Failure</td>
        </tr>
        <tr>
            <td>07-03-2020</td>
            <td>04:50:00</td>
            <td>F9 B5 B1059.2</td>
            <td>CCAFS SLC-40</td>
            <td>SpaceX CRS-20, Starlink 5 v1.0 </td>
            <td>1977</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>18-03-2020</td>
            <td>12:16:00</td>
            <td>F9 B5 B1048.5</td>
            <td>KSC LC-39A</td>
            <td>Starlink 5 v1.0, Starlink 6 v1.0 </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Failure</td>
        </tr>
        <tr>
            <td>22-04-2020</td>
            <td>19:30:00</td>
            <td>F9 B5 B1051.4</td>
            <td>KSC LC-39A</td>
            <td>Starlink 6 v1.0, Crew Dragon Demo-2 </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>30-05-2020</td>
            <td>19:22:00</td>
            <td>F9 B5B1058.1 </td>
            <td>KSC LC-39A</td>
            <td>Crew Dragon Demo-2, Starlink 7 v1.0 </td>
            <td>12530</td>
            <td>LEO (ISS)</td>
            <td>NASA (CCDev)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>04-06-2020</td>
            <td>01:25:00</td>
            <td>F9 B5 B1049.5</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 7 v1.0, Starlink 8 v1.0</td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX, Planet Labs</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>13-06-2020</td>
            <td>09:21:00</td>
            <td>F9 B5 B1059.3</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 8 v1.0, SkySats-16, -17, -18, GPS III-03 </td>
            <td>15410</td>
            <td>LEO</td>
            <td>SpaceX, Planet Labs</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>30-06-2020</td>
            <td>20:10:46</td>
            <td>F9 B5B1060.1</td>
            <td>CCAFS SLC-40</td>
            <td>GPS III-03, ANASIS-II</td>
            <td>4311</td>
            <td>MEO</td>
            <td>U.S. Space Force</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>20-07-2020</td>
            <td>21:30:00</td>
            <td>F9 B5 B1058.2 </td>
            <td>CCAFS SLC-40</td>
            <td>ANASIS-II, Starlink 9 v1.0</td>
            <td>5500</td>
            <td>GTO</td>
            <td>Republic of Korea Army, Spaceflight Industries (BlackSky)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>07-08-2020</td>
            <td>05:12:00</td>
            <td>F9 B5 B1051.5</td>
            <td>KSC LC-39A</td>
            <td>Starlink 9 v1.0, SXRS-1, Starlink 10 v1.0 </td>
            <td>14932</td>
            <td>LEO</td>
            <td>SpaceX, Spaceflight Industries (BlackSky), Planet Labs</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>18-08-2020</td>
            <td>14:31:00</td>
            <td>F9 B5 B1049.6</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 10 v1.0, SkySat-19, -20, -21, SAOCOM 1B </td>
            <td>15440</td>
            <td>LEO</td>
            <td>SpaceX, Planet Labs, PlanetIQ</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>30-08-2020</td>
            <td>23:18:00</td>
            <td>F9 B5 B1059.4</td>
            <td>CCAFS SLC-40</td>
            <td>SAOCOM 1B, GNOMES 1, Tyvak-0172</td>
            <td>3130</td>
            <td>SSO</td>
            <td>CONAE, PlanetIQ, SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>03-09-2020</td>
            <td>12:46:14</td>
            <td>F9 B5 B1060.2 </td>
            <td>KSC LC-39A</td>
            <td>Starlink 11 v1.0, Starlink 12 v1.0 </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>06-10-2020</td>
            <td>11:29:34</td>
            <td>F9 B5 B1058.3 </td>
            <td>KSC LC-39A</td>
            <td>Starlink 12 v1.0, Starlink 13 v1.0 </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>18-10-2020</td>
            <td>12:25:57</td>
            <td>F9 B5 B1051.6</td>
            <td>KSC LC-39A</td>
            <td>Starlink 13 v1.0, Starlink 14 v1.0 </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>24-10-2020</td>
            <td>15:31:34</td>
            <td>F9 B5 B1060.3</td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 14 v1.0, GPS III-04  </td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>05-11-2020</td>
            <td>23:24:23</td>
            <td>F9 B5B1062.1</td>
            <td>CCAFS SLC-40</td>
            <td>GPS III-04 , Crew-1</td>
            <td>4311</td>
            <td>MEO</td>
            <td>USSF</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>16-11-2020</td>
            <td>00:27:00</td>
            <td>F9 B5B1061.1 </td>
            <td>KSC LC-39A</td>
            <td>Crew-1, Sentinel-6 Michael Freilich </td>
            <td>12500</td>
            <td>LEO (ISS)</td>
            <td>NASA (CCP)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>21-11-2020</td>
            <td>17:17:08</td>
            <td>F9 B5B1063.1</td>
            <td>VAFB SLC-4E</td>
            <td>Sentinel-6 Michael Freilich, Starlink 15 v1.0 </td>
            <td>1192</td>
            <td>LEO</td>
            <td>NASA / NOAA / ESA / EUMETSAT</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>25-11-2020</td>
            <td>02:13:00</td>
            <td>F9 B5 B1049.7 </td>
            <td>CCAFS SLC-40</td>
            <td>Starlink 15 v1.0, SpaceX CRS-21</td>
            <td>15600</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
        <tr>
            <td>06-12-2020</td>
            <td>16:17:08</td>
            <td>F9 B5 B1058.4 </td>
            <td>KSC LC-39A</td>
            <td>SpaceX CRS-21</td>
            <td>2972</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>Success</td>
        </tr>
    </tbody>
</table>




```python
%sql SELECT DISTINCT(LAUNCH_SITE) FROM SPACEXTBL
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>Launch_Site</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>CCAFS LC-40</td>
        </tr>
        <tr>
            <td>VAFB SLC-4E</td>
        </tr>
        <tr>
            <td>KSC LC-39A</td>
        </tr>
        <tr>
            <td>CCAFS SLC-40</td>
        </tr>
    </tbody>
</table>



### Task 2

##### Display 5 records where launch sites begin with the string 'CCA'



```python
%sql SELECT * FROM SPACEXTBL WHERE LAUNCH_SITE LIKE "CCA%" LIMIT 5

```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>Date</th>
            <th>Time (UTC)</th>
            <th>Booster_Version</th>
            <th>Launch_Site</th>
            <th>Payload</th>
            <th>PAYLOAD_MASS__KG_</th>
            <th>Orbit</th>
            <th>Customer</th>
            <th>Mission_Outcome</th>
            <th>Landing _Outcome</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>04-06-2010</td>
            <td>18:45:00</td>
            <td>F9 v1.0  B0003</td>
            <td>CCAFS LC-40</td>
            <td>Dragon Spacecraft Qualification Unit</td>
            <td>0</td>
            <td>LEO</td>
            <td>SpaceX</td>
            <td>Success</td>
            <td>Failure (parachute)</td>
        </tr>
        <tr>
            <td>08-12-2010</td>
            <td>15:43:00</td>
            <td>F9 v1.0  B0004</td>
            <td>CCAFS LC-40</td>
            <td>Dragon demo flight C1, two CubeSats, barrel of Brouere cheese</td>
            <td>0</td>
            <td>LEO (ISS)</td>
            <td>NASA (COTS) NRO</td>
            <td>Success</td>
            <td>Failure (parachute)</td>
        </tr>
        <tr>
            <td>22-05-2012</td>
            <td>07:44:00</td>
            <td>F9 v1.0  B0005</td>
            <td>CCAFS LC-40</td>
            <td>Dragon demo flight C2</td>
            <td>525</td>
            <td>LEO (ISS)</td>
            <td>NASA (COTS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>08-10-2012</td>
            <td>00:35:00</td>
            <td>F9 v1.0  B0006</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-1</td>
            <td>500</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
        <tr>
            <td>01-03-2013</td>
            <td>15:10:00</td>
            <td>F9 v1.0  B0007</td>
            <td>CCAFS LC-40</td>
            <td>SpaceX CRS-2</td>
            <td>677</td>
            <td>LEO (ISS)</td>
            <td>NASA (CRS)</td>
            <td>Success</td>
            <td>No attempt</td>
        </tr>
    </tbody>
</table>



### Task 3

##### Display the total payload mass carried by boosters launched by NASA (CRS)



```python
%sql SELECT SUM(PAYLOAD_MASS__KG_) FROM SPACEXTBL WHERE CUSTOMER = "NASA (CRS)"
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>SUM(PAYLOAD_MASS__KG_)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>45596</td>
        </tr>
    </tbody>
</table>



### Task 4

##### Display average payload mass carried by booster version F9 v1.1



```python
%sql SELECT AVG(PAYLOAD_MASS__KG_) FROM SPACEXTBL WHERE Booster_Version	LIKE "F9 v1.1%"
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>AVG(PAYLOAD_MASS__KG_)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>2534.6666666666665</td>
        </tr>
    </tbody>
</table>



### Task 5

##### List the date when the first succesful landing outcome in ground pad was acheived.

*Hint:Use min function*



```sql
%%sql 
SELECT MIN("Date")
FROM SPACEXTBL
WHERE
    "Landing _Outcome" = 'Success'
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>MIN(&quot;Date&quot;)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>02-03-2019</td>
        </tr>
    </tbody>
</table>



## Double check for missing Successful Mission Outcomes, Success payload status unclear


```python
%sql SELECT DISTINCT(COUNT(Mission_Outcome)) FROM SPACEXTBL WHERE Mission_Outcome = "Success (payload status unclear)"
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>(COUNT(Mission_Outcome))</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
        </tr>
    </tbody>
</table>



## Double Check for successful misssion outcomes


```python
%sql SELECT COUNT(Mission_Outcome) FROM SPACEXTBL WHERE Mission_Outcome = "Success"
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>COUNT(Mission_Outcome)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>98</td>
        </tr>
    </tbody>
</table>



## Double Check for failure mission outcomes


```python
%sql SELECT COUNT(Mission_Outcome) FROM SPACEXTBL WHERE Mission_Outcome Like "Fail%"
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>COUNT(Mission_Outcome)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>1</td>
        </tr>
    </tbody>
</table>



### Task 6

##### List the names of the boosters which have success in drone ship and have payload mass greater than 4000 but less than 6000



```python
%sql SELECT Booster_Version,Customer FROM SPACEXTBL WHERE PAYLOAD_MASS__KG_ BETWEEN 4000 AND 6000
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>Booster_Version</th>
            <th>Customer</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>F9 v1.1</td>
            <td>AsiaSat</td>
        </tr>
        <tr>
            <td>F9 v1.1 B1011</td>
            <td>AsiaSat</td>
        </tr>
        <tr>
            <td>F9 v1.1 B1014</td>
            <td>ABS Eutelsat</td>
        </tr>
        <tr>
            <td>F9 v1.1 B1016</td>
            <td>Turkmenistan National Space Agency</td>
        </tr>
        <tr>
            <td>F9 FT B1020</td>
            <td>SES</td>
        </tr>
        <tr>
            <td>F9 FT B1022</td>
            <td>SKY Perfect JSAT Group</td>
        </tr>
        <tr>
            <td>F9 FT B1026</td>
            <td>SKY Perfect JSAT Group</td>
        </tr>
        <tr>
            <td>F9 FT B1030</td>
            <td>EchoStar</td>
        </tr>
        <tr>
            <td>F9 FT  B1021.2</td>
            <td>SES</td>
        </tr>
        <tr>
            <td>F9 FT B1032.1</td>
            <td>NRO</td>
        </tr>
        <tr>
            <td>F9 B4 B1040.1</td>
            <td>U.S. Air Force</td>
        </tr>
        <tr>
            <td>F9 FT  B1031.2</td>
            <td>SES EchoStar</td>
        </tr>
        <tr>
            <td>F9 B4 B1043.1</td>
            <td>Northrop Grumman</td>
        </tr>
        <tr>
            <td>F9 FT  B1032.2</td>
            <td>SES</td>
        </tr>
        <tr>
            <td>F9 B4  B1040.2</td>
            <td>SES</td>
        </tr>
        <tr>
            <td>F9 B5 B1046.2</td>
            <td>Telkom Indonesia</td>
        </tr>
        <tr>
            <td>F9 B5 B1047.2</td>
            <td>Es hailSat</td>
        </tr>
        <tr>
            <td>F9 B5 B1046.3</td>
            <td>Spaceflight Industries</td>
        </tr>
        <tr>
            <td>F9 B5B1054</td>
            <td>USAF</td>
        </tr>
        <tr>
            <td>F9 B5 B1048.3</td>
            <td>PSN, SpaceIL / IAI</td>
        </tr>
        <tr>
            <td>F9 B5 B1051.2 </td>
            <td>Canadian Space Agency (CSA)</td>
        </tr>
        <tr>
            <td>F9 B5B1060.1</td>
            <td>U.S. Space Force</td>
        </tr>
        <tr>
            <td>F9 B5 B1058.2 </td>
            <td>Republic of Korea Army, Spaceflight Industries (BlackSky)</td>
        </tr>
        <tr>
            <td>F9 B5B1062.1</td>
            <td>USSF</td>
        </tr>
    </tbody>
</table>



### Task 7

##### List the total number of successful and failure mission outcomes



```python
%sql SELECT COUNT(Mission_Outcome) FROM SPACEXTBL
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>COUNT(Mission_Outcome)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>101</td>
        </tr>
    </tbody>
</table>



### Task 8

##### List the   names of the booster_versions which have carried the maximum payload mass. Use a subquery



```python
%sql SELECT Booster_Version FROM SPACEXTBL WHERE PAYLOAD_MASS__KG_ = (SELECT MAX(PAYLOAD_MASS__KG_) from spacextbl)
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>Booster_Version</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>F9 B5 B1048.4</td>
        </tr>
        <tr>
            <td>F9 B5 B1049.4</td>
        </tr>
        <tr>
            <td>F9 B5 B1051.3</td>
        </tr>
        <tr>
            <td>F9 B5 B1056.4</td>
        </tr>
        <tr>
            <td>F9 B5 B1048.5</td>
        </tr>
        <tr>
            <td>F9 B5 B1051.4</td>
        </tr>
        <tr>
            <td>F9 B5 B1049.5</td>
        </tr>
        <tr>
            <td>F9 B5 B1060.2 </td>
        </tr>
        <tr>
            <td>F9 B5 B1058.3 </td>
        </tr>
        <tr>
            <td>F9 B5 B1051.6</td>
        </tr>
        <tr>
            <td>F9 B5 B1060.3</td>
        </tr>
        <tr>
            <td>F9 B5 B1049.7 </td>
        </tr>
    </tbody>
</table>



### Task 9

##### List the records which will display the month names, failure landing_outcomes in drone ship ,booster versions, launch_site for the months in year 2015.

**Note: SQLLite does not support monthnames. So you need to use  substr(Date, 4, 2) as month to get the months and substr(Date,7,4)='2015' for year.**



```sql
%%sql 
SELECT
    "Landing _Outcome",
    Booster_Version,
    Launch_Site,
    substr(Date,4,2)
FROM SPACEXTBL
WHERE
    substr(Date,7,4) = '2015'
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>Landing _Outcome</th>
            <th>Booster_Version</th>
            <th>Launch_Site</th>
            <th>substr(Date,4,2)</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Failure (drone ship)</td>
            <td>F9 v1.1 B1012</td>
            <td>CCAFS LC-40</td>
            <td>01</td>
        </tr>
        <tr>
            <td>Controlled (ocean)</td>
            <td>F9 v1.1 B1013</td>
            <td>CCAFS LC-40</td>
            <td>02</td>
        </tr>
        <tr>
            <td>No attempt</td>
            <td>F9 v1.1 B1014</td>
            <td>CCAFS LC-40</td>
            <td>03</td>
        </tr>
        <tr>
            <td>Failure (drone ship)</td>
            <td>F9 v1.1 B1015</td>
            <td>CCAFS LC-40</td>
            <td>04</td>
        </tr>
        <tr>
            <td>No attempt</td>
            <td>F9 v1.1 B1016</td>
            <td>CCAFS LC-40</td>
            <td>04</td>
        </tr>
        <tr>
            <td>Precluded (drone ship)</td>
            <td>F9 v1.1 B1018</td>
            <td>CCAFS LC-40</td>
            <td>06</td>
        </tr>
        <tr>
            <td>Success (ground pad)</td>
            <td>F9 FT B1019</td>
            <td>CCAFS LC-40</td>
            <td>12</td>
        </tr>
    </tbody>
</table>



### Task 10

##### Rank the  count of  successful landing_outcomes between the date 04-06-2010 and 20-03-2017 in descending order.



```sql
%%sql 
SELECT
    "Landing _Outcome",
    COUNT("Landing _Outcome"),
    RANK() OVER(ORDER BY COUNT("Landing _Outcome") DESC) as ranking
FROM SPACEXTBL
WHERE
    Date BETWEEN '04-06-2010' AND '20-03-2017'
GROUP BY
    1
ORDER BY
    2 DESC
```

     * sqlite:///my_data1.db
    Done.





<table>
    <thead>
        <tr>
            <th>Landing _Outcome</th>
            <th>COUNT(&quot;Landing _Outcome&quot;)</th>
            <th>ranking</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Success</td>
            <td>20</td>
            <td>1</td>
        </tr>
        <tr>
            <td>No attempt</td>
            <td>10</td>
            <td>2</td>
        </tr>
        <tr>
            <td>Success (drone ship)</td>
            <td>8</td>
            <td>3</td>
        </tr>
        <tr>
            <td>Success (ground pad)</td>
            <td>6</td>
            <td>4</td>
        </tr>
        <tr>
            <td>Failure (drone ship)</td>
            <td>4</td>
            <td>5</td>
        </tr>
        <tr>
            <td>Failure</td>
            <td>3</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Controlled (ocean)</td>
            <td>3</td>
            <td>6</td>
        </tr>
        <tr>
            <td>Failure (parachute)</td>
            <td>2</td>
            <td>8</td>
        </tr>
        <tr>
            <td>No attempt </td>
            <td>1</td>
            <td>9</td>
        </tr>
    </tbody>
</table>



### Reference Links

*   <a href ="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Labs_Coursera_V5/labs/Lab%20-%20String%20Patterns%20-%20Sorting%20-%20Grouping/instructional-labs.md.html?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01&origin=www.coursera.org">Hands-on Lab : String Patterns, Sorting and Grouping</a>

*   <a  href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Labs_Coursera_V5/labs/Lab%20-%20Built-in%20functions%20/Hands-on_Lab__Built-in_Functions.md.html?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01&origin=www.coursera.org">Hands-on Lab: Built-in functions</a>

*   <a  href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Labs_Coursera_V5/labs/Lab%20-%20Sub-queries%20and%20Nested%20SELECTs%20/instructional-labs.md.html?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01&origin=www.coursera.org">Hands-on Lab : Sub-queries and Nested SELECT Statements</a>

*   <a href="https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Module%205/DB0201EN-Week3-1-3-SQLmagic.ipynb?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01">Hands-on Tutorial: Accessing Databases with SQL magic</a>

*   <a href= "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBMDeveloperSkillsNetwork-DB0201EN-SkillsNetwork/labs/Module%205/DB0201EN-Week3-1-4-Analyzing.ipynb?utm_medium=Exinfluencer&utm_source=Exinfluencer&utm_content=000026UJ&utm_term=10006555&utm_id=NA-SkillsNetwork-Channel-SkillsNetworkCoursesIBMDS0321ENSkillsNetwork26802033-2022-01-01">Hands-on Lab: Analyzing a real World Data Set</a>


## Author(s)

<h4> Lakshmi Holla </h4>


## Other Contributors

<h4> Rav Ahuja </h4>


## Change log

| Date       | Version | Changed by    | Change Description        |
| ---------- | ------- | ------------- | ------------------------- |
| 2021-07-09 | 0.2     | Lakshmi Holla | Changes made in magic sql |
| 2021-05-20 | 0.1     | Lakshmi Holla | Created Initial Version   |


## <h3 align="center"> © IBM Corporation 2021. All rights reserved. <h3/>

