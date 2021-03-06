*Not officially affiliated with ClicData*
# clicdata-python
A package to work with your ClicData account natively in Python.

Currently only supports the Client Credentials or Basic authentication methods. 
* See how to generate your Client ID and Client Secret for Client Credentials here: 
 https://app.clicdata.com/help/apidocumentation/api-auth-oauth-client
* See how to generate your Client ID for Basic here: 
 https://app.clicdata.com/help/apidocumentation/api-auth-basic

Right now this is an early alpha version with limited type checking and error handling. As development progresses more handling will be built in and more API functionality will be incorporated.

**Example Use:**
Returns list of all dashboards on your account with additional meta data
```py
from clicdata_api_wraper.session import SessionManager
from clicdata_api_wraper.data import Data

if __name__ == '__main__':
    client_id = "yourclientidhere"
    client_secret = "yourclientsecrethere"

    # Initializes SessionManager which keeps a connection open for all moduels to use
    SessionManager(client_id=client_id, client_secret=client_secret)
    list_of_tables = Data().get_data()

    # Ignores the SessionManager initiation and uses an isolated session
    # Useful for pushing to or modifying on a second account
    list_of_tables_2 = Data(client_id=client_id, client_secret=client_secret).get_data()
```

## Currently working:

## Classes

* **[Session](#session)**
* **[SessionManager](#sessionmanager)**
* **[Data](#data)**
* **[Account](#account)**
* **[Dashboard](#dashboard)**
* **[Schedule](#schedule)**
* **[To Do](#planned-method-endpoints)**


### Session
* **Parameters**:
  * **auth_method** : Session - accepts session class as input to authenticate
  * **client_id** : str - client_id generated by ClicData, required for all authentication methods
  * *(Optional)* **client_secret** : str - client_secret generated by ClicData, required for **Client Credentials**
  * *(Optional)* **username** : str - username as a string, required only for **Basic**
  * *(Optional)* **password** : str - password as a string, required only for **Basic**

This class is used by SessionManager to open a single session for the entire runtime or by each individual class directly to open one-off sessions.

### SessionManager
* **Parameters**:
  * **\*\*connection_params**: \*\*Kwargs to pass-through required session parameters to Session object
  
This class intializes Session as a class parameter and uses @classmethod to persist the session token through all modules used in this library. It's passed automatically to other classes as a parameter unless otherwise specified.
  
**Client Credentials Example:**
```py
SessionManager(
  auth_method='client_credentials', 
  client_id='youridhere', 
  client_secret='yoursecrethere'
)
list_data_sets = Data().get_data()
```
*returns dataframe containing list of data on your account*

**Basic Auth Example:**
```py
SessionManager(
  auth_method='basic', 
  client_id='youridhere', 
  username='yourplaintextusername', 
  password='yourplaintextpassword'
)
list_data_sets = Data().get_data()
```
*returns dataframe containing list of data on your account*

## Modules (CLasses & Methods)

### Data

#### get_data()
* **Parameters**:
  * *(Optional)* **rec_id** : int - RecId of the data you want to retrieve
  * **output** : str - Output format, either df or dict
* **Endpoints**:
  * List Data: GET /data
  * Retrieve Data: GET /data/{id}
* **Usage**:
  * If no rec_id is provided lists all data on account, if rec_id is provided, retrieves data from specified data set.

#### get_data_history()
* **Parameters**:
  * **rec_id** : int - RecId of the data you want to retrieve
  * *(Optional)* **ver_id** : int - Version ID of the data you want to retrieve from your data set
  * **output** : str - Output format, either df or dict
* **Endpoints**:
  * List Data History: GET /data/{id}/versions
  * Retrieve Historical Data: GET /data/{id}/v/{ver}
* **Usage**:
  * If no ver_id is provided lists all stored data versions, if ver_id is provided, retrieves data from specified data set version. Must have Data History enabled on data set to use.

#### create_data()
* **Parameters**:
  * **name** : str - Name of data table created in ClicData. Must be unique to account.
  * *(Optional)* **desc** : str - Long form description attached to table in ClicData.
  * **cols** : dict - Column name as key, data type as value.
* **Endpoints**:
  * Create New Custom Table: POST /data
* **Usage**:
  * Creates and empty custom data table on your ClicData account, which can then be appended to

#### append_data()
* **Parameters**:
  * **rec_id** : int - id of your data in ClicData
  * **data** : pandas.Dataframe - df containing the data you want to append.
* **Endpoints**:
  * Append Data: POST /data/{id}/row
* **Usage**:
  * Append your data to an existing dataset
  
#### static_send_data()
* **Parameters**:
  * **name** : str - Name of data set to create, must be unique to your account
  * *(Optional)* **desc** : str - Optional, long-form details about your data
  * **data** : pandas.Dataframe - Data to upload
* **Usage**:
  * Uses create_data() to create a static data set, then uses append_data() to add the input data to it. Note: Input must be a dataframe.
  
#### rebuild_data()
* **Parameters**:
  * **rec_id** : int - id of your data in ClicData
  * **method** : str - reload method to refresh the data with [
            "reload",
            "recreate",
            "update",
            "updateappend",
            "append"
        ]
* **Endpoints**:
  * Recreate Data: POST /data/{id}/recreate
  * Reload Data: POST /data/{id}/rebuild
  * Append Data: POST /data/{id}/append
  * Update Data: POST /data/{id}/update
  * Update/Append Data: POST /data/{id}/updateappend
* **Usage**:
  * Refreshes specified data set using the method specified by the passed parameter
  
#### delete_data()
* **Parameters**:
  * **session** : Session - accepts session class as input to authenticate
  * **rec_id** :  int - id of your data in ClicData
  * **filters** : dict - dictionary with column names as keys and values as values specifying which rows to delete
  * *(Optional)* **multiple_rows** : str - accepts 'all' or 'first', defaults to 'all'
* **Endpoints**:
  * Delete Data: DELETE /data/{id}/row
* **Usage**:
  * Deletes rows from a dataset on your account based on the filters passed.

### Account

#### get_account()
* **Parameters**:
  * **output** : str - Output format, either df or dict
* **Endpoints**:
  * Account Metrics: GET /account
* **Usage**:
  * Get details on account usage and limits
 
#### get_account_activity()
* **Parameters**:
  * **entity** : str - Pull activity data for 'dashboards' vs 'users'
  * **output** : string - Output format, either df or dict
* **Endpoints**:
  * User Activity: GET /account/activity/users
  * Dashboard Activity: GET /account/activity/dashboards
* **Usage**:
  * Retrieve either dashboard or user activity

### Dashboard

#### get_dashboard()
* **Parameters**:
  * **thumbnail** : bool - Whether to include base64 copies of dashboard thumbnails
  * *(Optional)* **name** : str - Filter dashboards by name
  * **output** : string - Output format, either df or dict
* **Endpoints**:
  * Dashboard Details: GET /dashboard
* **Usage**:
  * Get details of all dashboards on an account

#### get_dashboard_thumbnail()
* **Parameters**:
  * **name** : str : Filter dashboards by name
  * **output** : string - Whether the function output a string or an image ['base64', 'image']
* **Endpoints**:
  * Dashboard Thumbnail: GET /dashboard/{id}/thumbnail
* **Usage**:
  * Returns thumbnail either ase base64 encoded string or image
 
#### get_dashboard_snapshot()
* **Parameters**:
  * **name** : str : Filter dashboards by name
  * **output** : string - Whether the function output a string or an image ['base64', 'image']
* **Endpoints**:
  * Dashboard Snapshot: GET /dashboard/{id}/snapshot
* **Usage**:
  * Returns snapshot either ase base64 encoded string or image

### Schedule

#### get_schedule()
* **Parameters**:
  * *(Optional)* **rec_id** : int - Id of your schedule in ClicData
  * **output** : string - Output format, either df or dict
* **Endpoints**:
  * List Schedules: GET /schedule
  * Schedule Details: GET /schedule/{id}
* **Usage**:
  * Get details of all schedules on an account or single schedule if rec_id is passed

#### trigger_schedule()
* **Parameters**:
  * **rec_id** : int - Id of your schedule in ClicData
* **Endpoints**:
  * Trigger Schedule: POST /schedule/{id}/trigger
* **Usage**:
  * Trigger a specified schedule by id
 

## To Do:

### Planned Method Endpoints:

* **Data:**
  * Update Data: PUT /data/{id}/row
* **Team:**
  * List Teams: GET /team
  * Create Team: POST /team
  * Update Team: PUT /team/{id}
  * Delete Team: DELETE /team/{id}
  * Add User To Team: POST /team/{id}/user
  * Remove User From Team: DELETE /team/{id}/user
  * Add/Update Team Paramter: POST /team/{id}/parameter
  * Remove Team Paramter: DELETE /team/{id}/parameter
* **User:**
  * List Users: GET /user
  * Create new User: POST /user
  * Get User Details: GET /user/{id}
  * Update User: PUT /user/{id}
  * Delete User: DELETE /user/{id}
  * Set User Password: PUT /user/{id}/password
  
  
  # Resources:
  
  * **ClicData API Documentation:** https://app.clicdata.com/help/apidocumentation/api-docs
  
