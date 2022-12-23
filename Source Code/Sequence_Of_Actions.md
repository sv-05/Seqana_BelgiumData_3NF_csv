SEQUENCE OF ACTIONS - 

IMPORTANT NOTE - SQL queries performed in the process are present in "Queries.md" file in folder "Source Code". Queries named as 'sql_query_1', 'sql_query_2'.... and so on.

1. Import csv file to POSTGRE
   
    a. Run sql_query_1 and create table for the data -
    
    CREATE TABLE public.belgium_data
    (
    "X" numeric,
    "Y" numeric,
    profile_id integer,
    profile_layer_id integer,
    country_name character(20),
    upper_depth integer,
    lower_depth integer,
    layer_name character(20),
    litter character(20),
    orgc_value character(20),
    orgc_value_avg numeric,
    orgc_method character varying,
    orgc_date character(40),
    orgc_dataset_id character(20)
    orgc_profile_code character(20)
    );
    
    b. Import the data into the table "public.belgium_data".
    
2. Observe the data. Few observations are - 
   
   -> Contains over 4k records, that means easily managed by POSTGRE no Big data technology needed.
   -> Column 'profile_layer_id' is unique for every row and is made the PRIMARY KEY.
   -> Batch patterns can be observed for columns - 'X', 'Y', 'profile_id', 'upper_depth', 'lower_depth', 'orgc_date' in parellel that all records changes as upperdepth       and lower depth's range ends.
   -> orgc_date datatype needs to be changed to 'date' type so analytics can be performed smoothly.
   -> Column orgc_method needs to be further bifurgated into - 7 more columns - calculation,	detection,	reaction,	sample_pretreatment,	spectral,	temperature	             and treatment to achieve 1NF.
   
3. Cleaning Data.

   a. Unwanted string from data like - "{", "{1:", "}", "2{" etc. needs to be removed. And Data type of 'orgc_date' changed to date type, so run below queries - 
   
    sql_query_2: 
    update belgium_data
    set orgc_value = replace(orgc_value, '{', '')

    sql_query_3: 
    update belgium_data
    set orgc_value = replace(orgc_value, '}', '')

    sql_query_4: 
    update belgium_data
    set orgc_method = replace(orgc_method, '{1:', '')

    sql_query_5: 
    update belgium_data
    set orgc_method = replace(orgc_method, '}', '')

    sql_query_6: 
    update belgium_data
    set orgc_date = replace(orgc_date, '{1:', '')

    sql_query_7: 
    update belgium_data
    set orgc_date = substring(orgc_date, '{2:', '')

    sql_query_8: 
    update belgium_data
    set orgc_date = replace(orgc_date, '}', '')
    
    sql_query_9: 
    update belgium_data
    set orgc_date = REPLACE(orgc_date, (SUBSTRING(orgc_date, POSITION(',2:' IN orgc_date))), '') where orgc_date like '%,2:%'

    sql_query_10: 
    alter table belgium_data
    alter column orgc_date TYPE date using (orgc_date::date)
    
    
4. Now to convert the table into 1NF, column'orgc_method needs to be bifurgated by using below query - 

    sql_query_11: 
    CREATE TABLE belgium_data_1NF AS
    (SELECT "X", "Y", profile_id, profile_layer_id, country_name, upper_depth, lower_depth, layer_name, litter, orgc_value,
    orgc_value_avg, 
    trim(split_part(split_part(orgc_method, ',', 1), '=', 2)) as calculation,
    trim(split_part(split_part(orgc_method, ',', 2), '=', 2)) as detection,
    trim(split_part(split_part(orgc_method, ',', 3), '=', 2)) as reaction,
    trim(split_part(split_part(orgc_method, ',', 4), '=', 2)) as sample_pretreatment,
    trim(split_part(split_part(orgc_method, ',', 5), '=', 2)) as spectral,
    trim(split_part(split_part(orgc_method, ',', 6), '=', 2)) as temperature,
    trim(split_part(split_part(orgc_method, ',', 7), '=', 2)) as treatment,
    orgc_date, orgc_dataset_id, orgc_profile_code
    from belgium_data)
    
    sql_query_12: 
    ALTER TABLE belgium_data_1NF ADD PRIMARY KEY (profile_layer_id);
    
    
5. Test our new Data.

IMPORTANT NOTE - Since the below queries are performed randomly as the data needs so no query naming like sql_query_1 is done.

    
    Perform a combination of the following queries-
    
    select count(profile_layer_id) from belgium_data_1NF
    select count(*) from belgium_data_1NF

    select count(profile_layer_id) from belgium_data
    select count(*) from belgium_data

    select count(distinct(profile_layer_id)) from belgium_data_1NF
    select count(*) from belgium_data_1NF

    select count(distinct(profile_layer_id)) from belgium_data
    select count(*) from belgium_data
    
    Also, there were tests performed to make sure that the columns - orgc_date and orgc_method were handled without any data being lost.
    
    
6. Converting Data into 2NF.

    Precisley the data is doesn't violates the condition of 2NF since there was not much proof for the partial dependency of non-primary columns.
    So, the table "belgium_data_1NF" was declared in second normal form.
    Also, every column could be retrieved using the primary key - 'profile_layer_id'.
    
7. Converting Data to 3NF.


    Observations - 
    -> Columns x, y, country, orgc_method, orgc_date, orgc_dataset_id, orgc_profile_code contains same records for each 'profile_id'. When checked these columns with a        GROUP BY then all records were same for each profile_id.
    -> On the second hand, Columns upper_depth, lower_depth, layer_name, litter, orgc_value, orgc_avg contains different records for each 'profile_layer_id'. And for          each 'profile_layer_id' we have a unique record for 'profile_id' in different table.
    
    Therefore, two tables have been made - 
    
    a. belgium_data_FACT - Includes columns - "X", "Y", country_name, profile_id, MAX(profile_layer_id) AS profile_layer_id, 
       calculation, detection, reaction, sample_pretreatment, spectral, temperature, treatment, orgc_date, orgc_dataset_id, orgc_profile_code as a Fact Table.
    b. belgium_data_FACT - Includes columns - profile_layer_id, upper_depth, lower_depth, layer_name, litter, orgc_value, orgc_value_avg as a Dimention Table.
    
    Queries used - 
    
      sql_query_13: 
      CREATE TABLE belgium_data_FACT AS
      (select "X", "Y", country_name, profile_id, MAX(profile_layer_id) profile_layer_id, 
      calculation, detection,	reaction, sample_pretreatment,	spectral,
      temperature, treatment,	orgc_date, orgc_dataset_id,	orgc_profile_code
      from belgium_data_1NF
      group by "X", "Y", country_name, profile_id, 
      calculation, detection,	reaction, sample_pretreatment,	spectral,
      temperature, treatment,	orgc_date, orgc_dataset_id,	orgc_profile_code)

      sql_query_14: 
      CREATE TABLE belgium_data_DIM AS
      (select profile_layer_id, upper_depth, lower_depth, layer_name, litter, orgc_value, orgc_value_avg
      from belgium_data_1NF)

      sql_query_15: 
      ALTER TABLE belgium_data_FACT ADD PRIMARY KEY (profile_layer_id);

      sql_query_16: 
      ALTER TABLE belgium_data_DIM ADD PRIMARY KEY (profile_layer_id);
    
