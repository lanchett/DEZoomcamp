### Ingesting data into postgres

Running the pgcli for checking the data

```
pgcli -h localhost -p 5432  -u root -d ny_taxi 
```

```bash
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi_trips \
  --url=${URL}
```

PS: lpep_pickup_datetime and lpep_dropoff_datetime are named tpep_pickup_datetime and tpep_dropoff_datetime in the CSV file (class walkthrough), 
but the script is expecting lpep_pickup_datetime and lpep_dropoff_datetime for homework.

### Homework 
#### Docker

## Question 1. Understanding docker first run 

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

What's the version of `pip` in the image?

Commands run:
```bash
# Run the container
docker run -it --entrypoint=bash python:3.12.8

# Check pip version inside the container
pip --version
```

Answer: 23.2.1

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

There are actually two valid options to connect from pgAdmin to PostgreSQL:

db:5432 
- This works because:
Docker Compose creates an internal network where containers can refer to each other using their service names
db is the service name in the compose file
5432 is the internal port PostgreSQL is listening on inside its container

postgres:5432 
- This also works because:
The container_name is set to "postgres"
Docker Compose's internal DNS will resolve this container name as well

#### Homework SQL
```
SELECT 
  CASE 
    WHEN trip_distance <= 1 THEN '0-1 miles'
    WHEN trip_distance > 1 AND trip_distance <= 3 THEN '1-3 miles'
    WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3-7 miles'
    WHEN trip_distance > 7 AND trip_distance <= 10 THEN '7-10 miles'
    ELSE 'Over 10 miles'
  END AS distance_category,
  COUNT(*) as trip_count
FROM green_tripdata
--WHERE lpep_pickup_datetime >= '2019-10-01' 
 -- AND lpep_pickup_datetime < '2019-11-01'
GROUP BY 
  CASE 
    WHEN trip_distance <= 1 THEN '0-1 miles'
    WHEN trip_distance > 1 AND trip_distance <= 3 THEN '1-3 miles'
    WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3-7 miles'
    WHEN trip_distance > 7 AND trip_distance <= 10 THEN '7-10 miles'
    ELSE 'Over 10 miles'
  END
;


SELECT 
  DATE(lpep_pickup_datetime) as pickup_day,
  MAX(trip_distance) as max_distance
FROM green_tripdata
WHERE lpep_pickup_datetime >= '2019-10-01' 
  AND lpep_pickup_datetime < '2019-11-01'
GROUP BY DATE(lpep_pickup_datetime)
ORDER BY max_distance DESC
LIMIT 1;


SELECT 
  zpu."Zone"  as pickup_zone,
  SUM(g.total_amount) as total_amount
FROM green_tripdata g
JOIN taxi_zone_lookup zpu ON g."PULocationID"  = zpu."LocationID" 
WHERE DATE(lpep_pickup_datetime) = '2019-10-18'
GROUP BY zpu."Zone"
HAVING SUM(g.total_amount) > 13000
ORDER BY total_amount DESC;


SELECT 
  zdo."Zone" as dropoff_zone,
  MAX(g.tip_amount) as max_tip
FROM green_tripdata g
JOIN taxi_zone_lookup zpu ON g."PULocationID" = zpu."LocationID"
JOIN taxi_zone_lookup zdo ON g."DOLocationID" = zdo."LocationID"
WHERE 
  zpu."Zone"= 'East Harlem North'
  AND DATE_TRUNC('month', lpep_pickup_datetime) = '2019-10-01'
GROUP BY zdo."Zone"
ORDER BY max_tip DESC
LIMIT 1;
```
# Question 7. Terraform Workflow
*Downloading provider plugins and setting up backend:*
- terraform init is the correct command. This initializes a Terraform working directory by downloading required providers and setting up the backend configuration

*Generating proposed changes and auto-executing the plan:*
- terraform apply -auto-approve is the correct command.This combines generating the execution plan and applying it without requiring manual confirmation


*Remove all resources managed by terraform:*
- terraform destroy is the correct command

1