sql_query_1
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



sql_query_2
update belgium_data_1
set orgc_value = replace(orgc_value, '{', '')

sql_query_3
update belgium_data_1
set orgc_value = replace(orgc_value, '}', '')

sql_query_4
update belgium_data_1
set orgc_method = replace(orgc_method, '{1:', '')

sql_query_5
update belgium_data_1
set orgc_method = replace(orgc_method, '}', '')

sql_query_6
update belgium_data_1
set orgc_date = replace(orgc_date, '{1:', '')

sql_query_7
update belgium_data_1
set orgc_date = substring(orgc_date, '{2:', '')

sql_query_8
update belgium_data_1
set orgc_date = replace(orgc_date, '}', '')

sql_query_9
update belgium_data_1
set orgc_date = REPLACE(orgc_date, (SUBSTRING(orgc_date, POSITION(',2:' IN orgc_date))), '') where orgc_date like '%,2:%'

sql_query_10
alter table belgium_data_1
alter column orgc_date TYPE date using (orgc_date::date)

sql_query_11
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
from belgium_data_1)
