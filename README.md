# Data Engineer Coding Exercise

Hi there!

If you're reading this, it means you're now at the coding exercise step of the data-engineering hiring process. 
We're really happy that you made it here and super appreciative of your time!

This repo is intentionally left empty except for the file you are currently reading. Please add your work with as much 
detail as you see fit.

## Expectations

* It should be executable, production ready code
* Take whatever time you need - we won’t look at start/end dates, you have a life besides this, and we respect that! 
Moreover, if there is something you had to leave incomplete or there is a better solution you would implement but 
couldn't due to personal time constraints, please try to walk us through your thought process or any missing parts, 
using the “Implementation Details” section below.

## About the Challenge

The goal of this exercise is for you to implement a fully functional end-to-end ETL pipeline via the open-source, 
full-stack data integration platform [Meltano](https://meltano.com/).

You will be working with the data provided by the [SpaceX API](https://github.com/r-spacex/SpaceX-API/tree/master), and
as it should be with any good data challenge, your work will be guided by one central question that we are aiming to
help find an answer for:

> When will there be 42,000 Starlink satellites in orbit, and how many launches will it take to get there?

You can assume that you will have a team of Data Analysts working in collaboration with you so you do not have to
provide the full solution. However, in your role as a Data Engineer, we would expect you to:

* create a Meltano custom extractor for [SpaceX-API](https://github.com/r-spacex/SpaceX-API/blob/master/docs/README.md) 
(following [this official tutorial](https://docs.meltano.com/tutorials/custom-extractor))
* configure Meltano`s [target-postgres](https://hub.meltano.com/loaders/target-postgres/) loader to send the data
extracted from the source by your custom extractor in the previous step into [Postgres](https://www.postgresql.org/)
* add a data transformation step via [Meltano`s implementation of dbt](https://docs.meltano.com/guide/transformation). 
Specifically:
  * Add a data transformation step via Meltano’s implementation of dbt. It must include a `transformation_updated_at` timestamp. This should result in a model (or models) that can be handed off to a team of data analysts.
* configure a [job](https://docs.meltano.com/reference/command-line-interface#job) in your Meltano project that runs the 
previous 3 steps in sequence

### When you are done

* Complete the "Implementation Details" section at the bottom of this README.
* Open a Pull Request in this repo and send the link to the recruiter with whom you have been in touch.
* You can also send some feedback about this exercise. Was it too short/big? Boring? Let us know!

## Useful Resources

* [Meltano Official Website](https://meltano.com/)
* [Meltano Docs](https://docs.meltano.com/)
* [Meltano Tutorial: Create a Custom Extractor](https://docs.meltano.com/tutorials/custom-extractor)
* [Meltano target-postgres Docs](https://hub.meltano.com/loaders/target-postgres/)
* [SpaceX-API Docs](https://github.com/r-spacex/SpaceX-API/blob/master/docs/README.md)
* [dbt Docs](https://docs.getdbt.com/)

## Implementation Details


# Meltano Installation
Following the steps specified in the meltano documentation .

I installed Meltano locally using pipx , using the guide in the meltano documentation below :
Pipx install : https://docs.meltano.com/guide/installation-guide/

> pipx install "meltano"
Created a meltano project in the git repository using the command below :
> meltano init meltano_project
Navigate to the newly created project directory:
> cd meltano-project 

# Create a Custom Extractor (tap-jsonplaceholder)
 Added the extractor to pull from the API source using the steps here :
* Installed the dependencies (cookiecutter, poetry,etc) except meltano which was already installed in the previous step .
* Created a Project Using the Cookiecutter Template :
> cookiecutter https://github.com/meltano/sdk --directory="cookiecutter/tap-template"

Followed the prompt to configure the tap information which creates the folder tap-jsonplaceholder that contains boilerplate code for developing the tap .

* Installed the Custom Extractor Python dependencies Using Poetry in the tap-jsonplaceholder folder.
Used the command below :
> cd tap-jsonplaceholder
> # [Optional] but useful if you need to debug your custom extractor
> poetry config virtualenvs.in-project true
> poetry install

* Configure the Custom Extractor to Consume Data from the Source

This step involved modifying the content of the boiler plate streams.py ,tap.py and the client.py template file

tap-jsonplaceholder/tap_jsonplaceholder/streams.py - In the streams.py file import the necessary libaries ,create the stream classes  
* path - contains the api end point gotten from the SpaceX repo (e.g /v5/launches )
* name - specified the schema name (e.g launches)
* schema - specified the schema properties and the columns to extracted , created a class for this.

*tap-jsonplaceholder/tap_jsonplaceholder/tap.py - Modified the tap.py file to specify the various stream types to be imported 

* tap-jsonplaceholder/tap_jsonplaceholder/client.py - Modified the client.py file to capture the spacex API base url  

> class jsonplaceholderStream(RESTStream):
 >   """jsonplaceholder stream class."""

   > @property
   > def url_base(self) -> str:
   >     """Return the API URL root, configurable via tap settings."""
   >    # TODO: hardcode a value here, or retrieve it from self.config
   >     return "https://api.spacexdata.com"
  

 Further information on the steps followed are contained in the guide with link :
https://docs.meltano.com/tutorials/custom-extractor/


## Storing and Loading the Data into the Target

This step involved configuring the postgres database where the data will be stored after extraction.
Loaded the data into a dockerized PostgreSQL database running on my laptop. 
To launch a local PostgreSQL container, run the command below :

> docker run --name meltano_postgres -p 5432:5432 -e POSTGRES_USER=meltano -e POSTGRES_PASSWORD=password -d postgres

Once the postgres database is up and running I configured postgres loader using the steps below :
Add the postgres loader using the command :
 > meltano add loader target-postgres --variant=meltanolabs 

To configure the plugin, using the command :
>  meltano config target-postgres 

Filled in the details for the attributes ,This added the non-sensitive configuration to the meltano.yml project file.

Ran the data integration (EL) pipeline :
> meltano run tap-jsonplaceholder target-postgres ---Execute the command below to run an ELT pipeline that replicates data from the REST API to postgres

## Process the  data using DBT

Install and configure the Postgres-specific dbt utility
> meltano add utility dbt-postgres

Initialize dbt
> meltano invoke dbt-postgres:initialize

Configure the dbt-postgres utility to use the same configuration as our target-postgres loader using meltano config

> meltano config dbt-postgres set host localhost
> meltano config dbt-postgres set port 5432 
> meltano config dbt-postgres set user meltano
> meltano config dbt-postgres set password password
> meltano config dbt-postgres set dbname postgres
> meltano config dbt-postgres set schema analytics

Add our source data to dbt
> mkdir transform/models/tap_jsonplaceholder

Configure the  source.yml file with the schema, source name, etc : meltano_project\transform\models\spacex\source.yml

Add the transformed model (i.e. launches.sql) 

Run the transformation process :
> meltano invoke dbt-postgres:run

To test the final pipeline alltogether :
> meltano run --full-refresh tap-jsonplaceholder target-postgres dbt-postgres:run

More guides on the dbt setup documented here :
https://docs.meltano.com/getting-started/part3#initialize-dbt


## Orchestrate and create a job 

# Define a job
> meltano --environment=prod job add spacex_extraction --tasks "tap-jsonplaceholder target-postgres dbt-postgres:run"

# Schedule the job

> meltano --environment=prod schedule add daily-spacex-load --job spacex_extraction --interval '@daily'
> meltano job list ---to list all jobs


## Assumptions
After going through the available datasets based on the central question. I came to the conclusion that the necessary schemas for addressing the question are the Launches and Starlink tables.

* The launches table contains information on all launches made successful or not (this includes starlink launches)
* The starlink table contains information on starlink satelites with a corresponding launch id as foreign key.

To get the number of starlink satelites, the DA can query the analytics.launches table where name like '%Starlink%'

# Notes
N:B : I considered loading the schema using a json path but due to time constraints in trying to format the json file , I moved forward with specifying the schema the Properties list in the class stream.

