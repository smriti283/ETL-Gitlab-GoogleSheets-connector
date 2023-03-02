# ETL-Gitlab-GoogleSheets-connector
An ETL script that extracts Gitlab data, pre-processes the dataset and write to Google Sheets


## PYTHON LIBRARIES:

	The following libraries are used in the script, some of which may need to be installed locally on the system running the script. 
  
- pandas: The pandas library is used for manipulating data and data analysis.
- numpy: The NumPy library is used for working with arrays.
datetime - The datetime module supplies classes for manipulating dates and times.
requests - The requests library is used for making HTTP requests in Python
gspread - We use gspread to authenticate the Google Sheets and Google Drive API credentials so that a connection can be made between Python and the Google Sheet for CRUD operations. 
df2gspread - df2gspread allows you to easily interact with Google Spreadsheets.
gspread_dataframe - This package allows easy data flow between a worksheet in a Google spreadsheet and a Pandas DataFrame.
oauth2client.service_account - This is a client library for accessing resources protected by OAuth 2.0. We particularly use the Service Account credential for OAuth 2.0. 
curl - PycURL is a Python interface to libcurl, the multiprotocol file transfer library.

B. SCRIPT DESCRIPTION:
The script is divided into 5 functions, the detailed descriptions of which are given below:
extract_data()- 
This function is used to make the API call to load the dataset and return it as a dataframe. 
API private token:
The API private token must be replaced by the user’s token as given by Gitlab. The user can generate one by accessing Gitlab > User icon (on top right) > Edit Profile > Access Tokens > Create Personal Access Token.
https://code.venom360.com/-/profile/personal_access_tokens
Data Transfer:
The API returns a maximum of 100 datarows per call (100 tickets = 1 page). We iterate through the pages to extract data and store the result locally in a file named ‘data.json’. We then read the file and append it to a new dataframe. 
The range of iterations is currently set at 19. This means that it will capture 19 pages of data (i.e a max of 1900 tickets). This range must be updated in the script as the number of tickets for the project increases. (As of Feb 6th, 2023, we’re at ~1510 tickets). 


transform_data():
This function pre-processes the data frame obtained from extract_data() and so that it becomes usable for creating PowerBI charts and analyses. It returns the transformed dataframe. 
The different steps followed here are as below:
Filtering out columns that are not required
Flattening the nested columns of the dataframe 
Combining the flattened columns into a single dataframe
Datetime conversions for ease of future analysis- converting time from seconds to hours
Pre-calculating ticket closure time
Data type conversions and string replacement of some columns. 
Extracting the status and duplicate labels and mapping them to populate empty cells. 
Renaming milestone titles
Resetting the index
Note - The above preprocessing steps are specific to Gitlab data for VASOO project. Other projects have different settings (such as labels, milestones etc.) and the function must be modified for each use case. 
c.  load_data():
This function writes the transformed data frame obtained from transform_data() to a Google Sheet named ‘Vasoo_Data’, which is our database for all analysis related to the current week of data. This function uses Google Sheets API and the libraries ‘gspread’, ‘df2gspread’ and ‘oauth2client’ to interact with a predefined google sheet and write the passed dataframe. 
In order to read/write to Google Sheets from Python, the following steps need to be followed:
Create a Google Sheet to which the data frame is supposed to be loaded.
Head over to the Google API console.  
Create a new project: Project Field > New Project > Add details > Create
Head over to ‘Enabled APIs and services’ (sidebar) > Search for 'Google Drive API' and ‘Google Sheets API’ in the list. Enable both.
Head over to 'Credentials' (sidebar), click 'Create Credentials' > 'Service Account'
Fill in the appropriate name and description, add the role as ‘Owner’ (or any appropriate role) in Step 2, add account details in Step 3 and click Done. 
On the ‘Credentials’ page under ‘Service Accounts’, click ‘Manage Service Accounts’. Click the 3 dot bar under ‘Actions’ of the service account you just created, and click ‘Manage Keys’. On the ‘Keys’ page > Add Key > Create New Key > JSON. Save the downloaded key file in the same folder as the python script, as we will be accessing it in the script.
Capture the Email address listed under the ‘Service Accounts’ section on the ‘Credentials’ page.
Head over to the Google Sheet created in Step 1. Share the document with the Email address captured from Step 8 and assign the role of editor. 
Capture the spreadsheet key of this Google Sheet. The spreadsheet key is the alphanumeric digits displayed on the Google Sheet URL after d/ and before /edit. Assign this key to the variable ‘spreadsheet_key’ in the script. 
https://docs.google.com/spreadsheets/d/spreadsheetKey/edit#gid=0
Capture the name of the spreadsheet tab that you want to write the data on. Assign this to the variable ‘wks_name’ in the script. 
Once the above credentials are set up, you can use the API to link with your spreadsheet and start with Create, Read, Update, and Delete (CRUD) operation on it. 
We first define the scope and set our credentials with the JSON file downloaded previously. We then authenticate these credentials and upload the dataframe to the spreadsheet. 
For our script for VASOO, the following must be updated - 
For every new project/ account or user, the name of the downloaded JSON file. 
For every new Google Sheet created, the spreadsheet_key and wks_name. 
d.  load_data_to_historical():
This function is used to build the historical data in order to perform week-on-week analysis. The data extracted this week is appended to the previously captured data in a separate Google Sheet named ‘Vasoo_Historical_Data’. 
As on Feb 6, 2023, this weekly database consists of 47 weeks of VASOO Gitlab data. 
This function follows the same code as load_data(), along with some additional steps such as:
Extracting only the columns that are needed for the week-on-week analysis.
Opening the Google Sheet and adding required rows towards the end for appending the dataframe. 
Setting the sheet index to begin incrementing from the last row index.
Adding data date and week number as a separate column and incrementing with every script run. 
e.  main():
This function is the entry point of the script and calls all other functions for their respective return values. 

