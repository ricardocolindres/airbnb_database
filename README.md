#Airbnb Rental Database

#Installation Instructions 

This database was design to support most of the features provided by a service such as Airbnb. A full description of the database’s structure is included alongside an entity relationship model to allow for better navigation and usage of the database. 

#Installation Options. 

1.Restore from backup.
The database can be restored from the following files included in the “DatabaseBackups” folder: 
rentals_db_ricardo_colindres_15-09-22_fast
rentals_db_ricardo_colindres_15-09-22_slow
The file ending with the word “slow” includes all CREATE AND INSERT statements and thus is slower. Please use the one that suits better your needs. 
Option 1: To restore the database backup, the pg_restore command can be used in the following way: After setting up a new database server and a new database, execute the following command:
pg_restore -U username -d dbname -1 rentals_db_ricardo_colindres_15-09-22_fast.dump
Option 2: To restore the database backup using the PgAdmin interface, follow the following steps:
1.	Open PgAdmin 6.11 or later
2.	Set up your server
3.	Set up your database
4.	Right click on the database
5.	Select “Restore…” and the restore backup dialog will open
6.	Load backup file and select custom for file type
7.	Click “Restore” button 
2. Execute SQL statements
If a clean copy of the database is needed, the SQL commands can be executed in the same order as provided.
3. Pull Docker Image from Docker Hub
docker image pull ricardocolindres/airbnb

Replica of an Airbnb Database

Rentals websites are big now. Companies such as Expedia and Bookings have taken over the hotel booking market while other such as Airbnb have a large market cap in private property rentals. This database was designed to support most of the features provided by a service such as Airbnb. The entity-relationship model can be easily scalable to allow for new features. Analogously, these features should easily integrate to the physical model. The database backups, from which the original database can be easily restored, include not just the database but also, thousands of entries of consistent data across all the tables. This allows developers and database managers to construct different queries to explore and test the database. Besides some third-party integrations, this database is ready for deployment in an operational environment. Before diving into the operational aspects of the database, I will briefly review the conceptual model behind this database.

The database is composed of 33 tables of which 5 are reference tables, i.e., they contain data that shouldn’t change frequently and is shared across all schemas. Tables such as the language, countries, and cities tables are among these reference tables. Data contained in these tables will be used across the database to provide consistent data. As a result, the database will have less redundancy and better performance. This database handles a large volume of data, so the hardware should be of great importance. Queries can get very complex for certain operations since data is normalized across different tables. Thus, developers will certainly end up using lots of JOIN queries and/or WINDOW functions to retrieve data from the database. As consequence, the server running the database should be capable of handling forecasted peak loads and still have some room for unexpected loads. Before moving on, it is important to mention these reference tables contain information that could be used to provide extra information that doesn’t necessarily have to be allocated in the database. For example, the cities table contains a wikiID for each city. Therefore, a simple python script could pull updated information from Wikipedia and/or other sources to provide relevant information about the location where the rentals are located. The database could be expanded to store this kind of data if performance were to be an issue when retrieving this data in real time for every incoming query. 

At a higher level, the database model makes no difference between a host and a user. This means that a user, which is the traveler who rents properties, can become a host at any time or vice versa, a host can also be a traveler. This makes sense from a business perspective since it reduces the overall number of accounts in the database and thus, the load on the database system. Consequently, it modifies the hardware specifications as well. Furthermore, it promotes current users to engage in both traveling and hosting activities which are at the core of a rental business. Two main tables handle users' and hosts' data. First, the user table. This table is where all the data that defines a user is contained; therefore, to engage in any activity in the system, every customer must be a user first. Moreover, a user can rent any property, message hosts, and review properties; however, users cannot “own” rentals. If a user wants to become a host (i.e., be able to post properties for rent), it must undergo a data expansion process where a boolean field is modified in the user table which identifies this user as a host. Likewise, a new host ID is created in the hosts table to further identify this new user as a host. This table (hosts table) references the user table, so for any given user that becomes a host, there must be only one row in the host table. Additionally, data only relevant to hosts is collected in the host table. This structure helps to separate travelers from hosts in other schemas such as the inbox and rentals schema but allows hosts to remain active as travelers and still be one single entity with users.

Furthermore, this database contains several unique functions and triggers that help automate certain operations within the database, removing the load from physical or cloud servers executing the actual back-end logic of the rental service. This will result in more load on the database server; however, this is usually a better idea since the server is optimized to perform these tasks. For example, host response times and read rates are automatically calculated every time a chat session is updated. Also, the rentals' ratings are automatically calculated every time someone submits a review. Consequently, this allows any service communicating with the database to easily pull the specific value rather than calculating it every time a front-end user request it. As a result, better response times will be observed on the front end of the rental service, improving the customers’ experience.  

On the side of security, I have chosen to go with salted-hashed passwords generated by a blowfish algorithm which delivers great security standards. Moreover, all payment data, as best practices call for, have been delegated to well-established payment processing systems. In this case, I have connected the database with the Stripe’s API for handling and processing payments. Therefore, the rental database will only store a stripe customer ID as instructed by Stripe’s API documentation. In the case of a breach that results in the exposure of data, critical payment information such as credit cards will remain secure. 

Next, the database can also support messaging. This feature is under the inbox schema, and it allows any user to communicate with a host or vice versa. Moreover, there is the rentals schema. This handles everything related to the rental process. First, all rentals are contained in the rentals table. Every rental supports custom pricing. This means that the host can establish special prices for certain dates of the year. Moreover, the host can establish other properties for the rentals such as the preparation time which establishes the number of nights before and after each reservation that the rental is blocked to allow for the user to prepare the property. The calendar availability table centralizes all data relevant to scheduling the property in one table for quick and easy reference. Other tables surround the rentals table forming a snowflake-like schema and providing additional data such as the spaces, property type, and rules for any given rental. There is one more important table that handles all bookings, that is the reservations table. The reservation tables keep track of data related to a reservation. That includes the dates, check-in and out status, reservation confirmation, daily price for a rental, the total price for a rental, and fees and discounts applied. Fees and discounts that were applied to the reservations are stored in an array containing the strings that represent them. This is helpful for the accounting process or any issue or dispute that may arise in the future. 

Finally, some very basic level of historization has been added to the database model as well. The rentals and rules tables, for example, manage the possible rules that any given rental may be subject to. The former gets information from the rules tables and the rentals table. However, a host may change the rules of its property at any given time; thus, rendering it impossible to check to which rules past reservations were subjected. This may be a very big problem if any dispute arises between the traveler and the host. Therefore, I have added a very basic historization table that keeps track of any changes made to the rentals and rules table. In this way, if travelers or hosts ever want to recall this info, it is possible to retrieve it by checking the “reservation_made_on” column on the reservation table and comparing it to the rental and rules table and its corresponding historization table on that same time frame. If someone were to implement this database for a real rental service, I would highly suggest further developing historization for tables such as the rentals table which may frequently change. 

To conclude, this rental database is based on a robust model and is easily deployable. Some adjustments may be needed to fully take it into an operational environment. For deployment, it is highly recommended not to use the admin user to connect to the database since this user is a superuser and can jeopardize the database’s security.  A user named  user_01 has been created which has limited privileges. This user inherits privileges from a role named user_group. This role can be given to other users if several connections are needed. I have included some python scripts that facilitate the insertion of a great amount of consistent data into the database. It may need some modifications for the actual names of the tables since in the process of creating the database some tables’ names where modified. 
