# ExtracTable
Freeware - ExtracTable - download tables from SAP ABAP & HANA based systems

The downloaded table can be directly published into the MS-Excel application (if installed) and/or as a CSV-file in the csv folder where the ExtracTable program resides.
It applies (customizable) resume logic when connections get lost during downloads.

-> ALWAYS WHEN DOWNLOADING TABLES KEEP IN MIND THAT DURING YOUR DOWNLOAD TABLE UPDATES IN SAP CAN TAKE PLACE, unless you explicitly wish to capture volatile data it is wise to download during your SAP system 'most quiet hours'
-> WHEN DOWNLOADING TABLES WITH FOR EXAMPLE MULTI MILLION RECORDS IT IS WISER TO ASK YOUR DATABASE ADMIN FOR TABLE REPLICAS

Upon initial execution of ExtracTable on your PC it needs to register its plugin rfcconnector.dll. For that it will execute command regsvr32.
Please make sure that your MS-Windows user id has sufficient (elevated) privileges to do such. Typically selecting [Run as administrator] to execute it suffices.

1. Input data

On the left side of the screen you must enter all relevant data that ExtracTable needs to build its RFC connection. This then is to be saved as a system for easier retrieval.
You will be informed when any of this input data is missing once you push the [Download] or [Record Count] button.
Typically you will find the data for input from your 'SAP Logon Pad' settings or else your SAP system administrator can inform you.

Assure that the SAP user ID that you use to connect with has at least:

1.1. Execution authorization for standard SAP RFC module: RFC_READ_TABLE and its generic prerequisites (via object S_RFC)

AND:

1.2. Table READ ONLY authorizations on the SAP system (via objects S_TABU_CLI and/or S_TABU_DIS or S_TABU_NAM) so that it can:
1.2.1. Read all record data from table DD02L and DD03L in order to determine meta data (field names / field types / field lengths etc) for downloading
1.2.2. Read all record data from the table you entered to be downloaded (e.g.: MARA, T001W, LFA1 etc etc)

Depending on your SAP system settings an active role/profile assigned with following authorizations usually suffices:

OBJECT		FIELD		VALUE
------		-----		-----
S_RFC		ACTVT		16
S_RFC		RFC_NAME	SYST
S_RFC		RFC_NAME	RFCH
S_RFC		RFC_NAME	RFCPING
S_RFC		RFC_NAME	SDTX
S_RFC		RFC_NAME	RFC1
S_RFC		RFC_NAME	RFC_GET_FUNCTION_INTERFACE
S_RFC		RFC_NAME	RFC_GET_UNICODE_STRUCTURE
S_RFC		RFC_NAME	RFC_READ_TABLE
S_RFC		RFC_TYPE	FUNC
S_RFC		RFC_TYPE	FUGR
S_TABU_DIS	ACTVT		03
S_TABU_DIS	DICBERCLS	* (or specified table authorization groups from table TDDAT)

If your organisation does not want to grant * or table authorization groups for S_TABU_DIS then you would need to specify the relevant table names, via object S_TABU_NAM.
Example:
S_TABU_NAM	ACTVT		03
S_TABU_NAM	TABLE		DD02L
S_TABU_NAM	TABLE		DD03L
S_TABU_NAM	TABLE		MARA, LFA1, BSEG (<- etc etc)

2.0 Output data

SAP's Function Module RFC_READ_TABLE does not necessarily show the same output as e.g. SAP transactions SE11, SE80, SE16, SE16N etc., yet what it produces however is integer compared with the values it retrieves from the database.
This is due to the fact that the source code logic in the (SE16 etc) transactions -under conditions- can for example display additional related data from other tables or even surpress data from a table.
RFC_READ_TABLE directly pulls the values from the database table without internally enriching/surpressing it.

2.1 RFC_READ_TABLE Limitations

2.1.1. SAP's Function Module RFC_READ_TABLE has the limitation that whenever a table that has fields of type RAW (uninterpreted bytes) or FLOAT (floating point) that those will not represent correctly. ExtracTable will give you warnings for each relevant field and still downloads the table / counts records.
2.1.2. When an SAP table has a field of type LCHR (Character string of arbitrary length) in it, RFC_READ_TABLE can not absorb that field. ExtracTable will then not download nor do a Record Count on that SAP table. ExtracTable will inform you for each relevant field and then informs you of the error.
The number of tables in SAP using above types from 2.1.1. and 2.1.2. is quite limited.
2.1.3. When the total technical length of fields of an SAP record exceed 512 characters then RFC_READ_TABLE cannot transfer that table. ExtracTable will then apply 'Split & Join' logic and download the full table column by column.

3. Connecting to the SAP system and accessing the tables

ExtracTable supports connection resumes when connection get lost. This only applies when records are in the process of being downloaded. Meaning that a connection loss during record count (which automatically happens before any download) is not auto resumed.

3.1. Note that ExtracTable has 2 modes of processing:
A. Standard Mode (messages you receive are simple and generic)
B. Enhanced Mode (messages your receive are more extensive and specific)

Using Enhanced Mode is strongly recommended. In order to use Enhanced Mode you need to make sure to put the following (copyrighted) file in ExtracTable's [plugins] folder
(which is in the same location as this ExtracTable executable program):

sapnwrfc.dll

Your PC may have this file already available, especially when you have SAPGUI 7.50 or higher installed. 
It can also be obtained as part of a standalone download from the SAP marketplace as the SAP NW RFC SDK.

3.2. Once you push the [Download] or [Record Count] button ExtracTable attempts to connect to your defined SAP system. The tab [Event Log] informs you of steps done and statuses.
If you enter an SAP table that does not exist you will be informed via a messagebox and/or it will be logged in tab the Event Log once ExtracTable detects such from the chosen SAP system.


4.0 Settings

These settings are build to balance between choices to make when starting downloads:

[Number of resume attempts] -> When TCP/IP and/or RFC connections get lost ExtracTable will try a number of attempts to resume the connection, you specify that number here. Standard setting is 999 but certain networks are set to block you when you retry an unsuccesful connection too often, so you may want to reduce then.

[Second(s) between new resume attempt] -> Between each resume attempt you can set a delay before ExtracTable tries again to establish the connection, you specify the number of seconds here. The higher the delay, the more time your download will take if your connection is instable. The lower the delay, the higher the chance that your connection still is unavailable. Standard setting is 90.

[Thread 1 Packet size for record fetch from SAP system] -> Any SAP Table that which has less than 512 characters when all columns their maximum field length is totalled, will be downloaded via a logic called "Thread 1" in Extractable. This is at maximum speed that SAP's module RFC_READ_TABLE offers. Each time when ExtracTable fetches/pulls data from SAP it will do so in packet size/chunks as specified in number of records here. Standard setting is 10000.

[Thread 2 Packet size for record fetch from SAP system] -> Any SAP Table that which has more than 512 characters when all columns their maximum field length is totalled, will be downloaded via a logic called "Thread 2" in Extractable. SAP's RFC_READ_TABLE module has the limitation of reading a maximum of 512 characters for a record. ExtracTable internally 'splits & joins' the SAP table whilst instructing module RFC_READ_TABLE to download sequential segments. This comes at a performance cost. Each time when ExtracTable fetches/pulls data from SAP it will do so in packet size/chunks as specified in number of records here. The maximum is 1000 in order to avoid data deviations as much as possible when table updates take place during download. Standard setting is 500. 

[Maximum rows in your MS-Excel version] -> Depending on MS-Excel versions there can be different maximum rows for a sheet. Set the maximum  of your Excel version here. Standard setting is 1048576.
