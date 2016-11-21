# Teched2016 - DMM162 
  
## Content/Purpose
DMM162 is a simple 3NF based DWH data model, based on the SalesOrder data of EPM and Shine content. It uses only Flatfile CSV-Tables as sources and can easily deployed and used for demos and education. Data is loaded via virtual table and flowgraphs into staging tables. From there, the data is transformed and loaded via flowgraphs into DWH-layer tables. The DWH-layer tables are accessed by a Virtual Data Mart using Calc Views (NameSpace DM).

DMM162 also includes Virtual Data Mart Calc Views directly accessing the Virtual Tables of the Flatfile Remote Source (Namespace VT_DM), including astacked Calc View on top of them (VT_DM::SO_ITEM_PD).

The PD-Models can be found in this repo within the folder "PD"

## How to build and deploy the model (for development and tests)
1. Create a remote source *LOCAL_FLATFILE_1* pointing to the SDI Flatfile Agent/Adapter and the data + config files
      
  1.1 Install DB-Agend (### additional Info from Peter needed###)
  
  1.2 Create a root-directory for data files, e.g. Unix Command `mkdir /hana/shared/DDR/dp_LOCALHOST_FLATFILE_1/data`
  
  1.3 Copy data from the given folder
  
  1.4 Create a config-directory for config files, e.g. Unix Command `mkdir /hana/shared/DDR/dp_LOCALHOST_FLATFILE_1/config`
  
  1.5 Copy data from https://github.wdf.sap.corp/HANAEDW/EPM-SQL/tree/master/CSV_EPM/config to config-directory
  
  1.6. Create Agend, e.g. SQL Statement `CREATE AGENT "LOCAL_FLATFILE_1" protocol 'TCP' host '<hostname>' port 5050;`
  
  1.7. Creae Adapter, e.g. SQL Statement `CREATE ADAPTER "FileAdapter" at location agent "LOCAL_FLATFILE_1";`
  
  1.8. Create Remote Source, e.g. SQL Statement
  
	    CREATE REMOTE SOURCE "LOCAL_FLATFILE_1" ADAPTER "FileAdapter" at location agent "LOCAL_FLATFILE_1"
	      configuration '<?xml version="1.0" encoding="UTF-8"?>
	        <ConnectionProperties name="ConnectionInfo">
	          <PropertyEntry name="rootdir">/hana/shared/DDR/dp_LOCALHOST_FLATFILE_1/data</PropertyEntry>
	          <PropertyEntry name="fileformatdir">/hana/shared/DDR/dp_LOCALHOST_FLATFILE_1/config</PropertyEntry>
	          <PropertyEntry name="target_options"></PropertyEntry>
	        </ConnectionProperties>'
	      WITH CREDENTIAL TYPE 'PASSWORD' USING
	        '<CredentialEntry name="AccessTokenEntry">
	          <password>1234</password>
	        </CredentialEntry>';

2. Prepare your user so that you can work in DevX 

3. Create a Role in H-Studio: CREATE ROLE "hdi::remote_sources::deploy_virtual"; 

4. Clone this repository in the DevX WebIDE

5. Build the DB module.

6. Logon to BusinessObject Cloud an establish a connection to the HANA Backend System and select the SalesDetails Calculation View.


##How to load data
Important: Please make sure that your container schema is the default schema, e.g. by `SET SCHEMA ...`
0. If you want to empty the target tables first, call procedure *STG2SHN::SHINE_TRUNCATE_ALL* and/or *SRC2STG::STAGING_TRUNCATE_ALL* in your container schema

1. Fill Staging tables by calling procedure *SRC2STG::SRC_LOAD_ALL* in your container schema (or *SRC2STG::FILE_LOAD_ALL*, since we use only one Remote Source in this Repo there is no difference)

2. Fill DWH-tables by calling procedure *STG2SHN::E2S_LOAD_ALL* in your container schema

3. When all the data is loaded, the simple ui should show some SalesOrders

##How to test
For a simple smoke test, e.g. after deploying to a new system, execute the following statements in your container schema:

    call "TEST_DB::SIMPLE_LOAD_TEST";
    select * from "TEST_DB::RESULTS";