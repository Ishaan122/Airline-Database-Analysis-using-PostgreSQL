# Airline-Database-Analysis-using-PostgreSQL

Extracted and transformed essential flights detail data from multiple tables using multiple joins , CTE(common table expressions) , 
SQL commands , windows , aggregate and rank functions leading to fast decision making process.Also, identified most important metrics to track,
then created a series of SQL queries to extract the necessary data from transactional database.






AIRLINE DATABASE


1.Questions: Find list of airport codes in Europe/Moscow timezone
 Expected Output:  Airport_code 


Answer:select airport_code
from airports 
where timezone = 'Europe/Moscow'



2.Write a query to get the count of seats in various fare condition for every aircraft code?
 Expected Outputs: Aircraft_code, fare_conditions ,seat count

Answer:select aircraft_code,fare_conditions,
count(seat_no) seat_count
from seats
group by 1,2


3.How many aircrafts codes have at least one Business class seats?
 Expected Output : Count of aircraft codes

Answer:select count(aircraft_code)

from seats 
where fare_conditions = 'Business'


4.Find out the name of the airport having maximum number of departure flight
 Expected Output : Airport_name 

Answer:with cte as (select airport_name , count(departure_airport)
from airports t 
full outer join flights f 
on t.airport_code = f.departure_airport 
group by 1
order by 2 desc 
limit 1)

select airport_name from cte 



5.Find out the name of the airport having least number of scheduled departure flights
 Expected Output : Airport_name 


Answer:with cte as (select airport_name , count(departure_airport)
from airports t 
full outer join flights f 
on t.airport_code = f.departure_airport 
where status = 'Scheduled'
group by 1
order by 2 asc 
limit 1)

select airport_name from cte 



6.How many flights from ‘DME’ airport don’t have actual departure?
 Expected Output : Flight Count 


Answer:select count(*) flight_count
from flights 
where departure_airport = 'DME' and actual_departure is null



7.Identify flight ids having range between 3000 to 6000
 Expected Output : Flight_Number , aircraft_code, ranges 


Answer:select f.flight_no , a.aircraft_code ,a.range

from flights f 
left join aircrafts a 
on f.aircraft_code = a.aircraft_code
where range between 5000 and 6000



8.Write a query to get the count of flights flying between URS and KUF?
 Expected Output : Flight_count



Answer:select count(*) flights_count
from flights
where (DEPARTURE_AIRPORT = 'KUF' AND arrival_airport = 'URS') OR (ARRIVAL_AIRPORT = 'KUF' AND DEPARTURE_AIRPORT = 'URS')




9.Write a query to get the count of flights flying from either from NOZ or KRR?
 Expected Output : Flight count 


Answer:select count(*) flights_count
from flights
where (DEPARTURE_AIRPORT = 'NOZ'  OR  DEPARTURE_AIRPORT = 'KRR')




10.Write a query to get the count of flights flying from KZN,DME,NBC,NJC,GDX,SGC,VKO,ROV
Expected Output : Departure airport ,count of flights flying from these   airports.


Answer:select DEPARTURE_AIRPORT, count(DEPARTURE_AIRPORT) flights_count
from flights
where DEPARTURE_AIRPORT IN ( 'KZN' ,'DME' ,'NBC' ,'NJC' ,'GDX' ,'SGC' ,'VKO' ,'ROV' )
GROUP BY 1




11.Write a query to extract flight details having range between 3000 and 6000 and flying from DME
Expected Output :Flight_no,aircraft_code,range,departure_airport


Answer:select f.Flight_no,f.aircraft_code,a.range,f.departure_airport

from flights f 
left join aircrafts a 
on f.aircraft_code = a.aircraft_code
where (range between 3000 and 6000) and departure_airport = 'DME'



12.Find the list of flight ids which are using aircrafts from “Airbus” company and got cancelled or delayed
 Expected Output : Flight_id,aircraft_model


Answer:select f.flight_id,a.model aircraft_model

from flights f 
left join aircrafts a 
on f.aircraft_code = a.aircraft_code
where model ilike '%Airbus%' and (status = 'Cancelled' or status = 'Delayed' )



13.Find the list of flight ids which are using aircrafts from “Boeing” company and got cancelled or delayed
Expected Output : Flight_id,aircraft_model


Answer:select f.flight_id,a.model aircraft_model

from flights f 
left join aircrafts a 
on f.aircraft_code = a.aircraft_code
where model ilike '%Boeing%' and (f.status = 'Cancelled' OR f.status = 'Delayed')



14.Which airport(name) has most cancelled flights (arriving)?
Expected Output : Airport_name 



Answer:with max_cancel as (select a.airport_name , count(status)

from flights f 
left join airports a 
on f.arrival_airport = a.airport_code
where status = 'Cancelled'
group by 1)
select airport_name
from max_cancel
where count = 2
limit 1



15.Identify flight ids which are using “Airbus aircrafts”
Expected Output : Flight_id,aircraft_model



Answer:select f.flight_id , model as aircraft_model

from flights f 
left join aircrafts a 
on f.aircraft_code = a.aircraft_code
where model ilike '%Airbus%'



16.Identify date-wise last flight id flying from every airport?
Expected Output: Flight_id,flight_number,schedule_departure,departure_airport


Answer:with cte as (
select flight_id, flight_no , scheduled_departure, rank() over (partition by departure_airport order by (to_char(scheduled_departure, 'yyyy-mm-dd')) desc) ,departure_airport
from flights)

select flight_id, flight_no, scheduled_departure,departure_airport from cte 
where rank = 1



17.Identify list of customers who will get the refund due to cancellation of the flights and how much amount they will get?
Expected Output : Passenger_name,total_refund


Answer:select t.passenger_name, i.amount 

from tickets t 
full outer join ticket_flights i
on t.ticket_no = i.ticket_no
full outer join flights f 
on i.flight_id = f.flight_id
where status = 'Cancelled'



18.Identify date wise first cancelled flight id flying for every airport?
Expected Output : Flight_id,flight_number,schedule_departure,departure_airport


Answer:with cte as (
select flight_id, flight_no , scheduled_departure,status,departure_airport,
       row_number() over (partition by status,departure_airport order by (to_char(scheduled_departure, 'yyyy-mm-dd')) asc) arank
from flights
where status = 'Cancelled')

select Flight_id,flight_no,scheduled_departure,departure_airport
from cte 
where arank = 1



19.Identify list of Airbus flight ids which got cancelled.
Expected Output : Flight_id

Answer:select flight_id 

from flights f 
left join aircrafts a 
on f.aircraft_code = a.aircraft_code
where status = 'Cancelled' and model ilike '%Airbus%'


     
 20.Identify list of flight ids having highest range.
 Expected Output : Flight_no, range


Answer:select flight_no , max(a.range) as range

from aircrafts a 
left join flights f 
on a.aircraft_code = f.aircraft_code
group by 1
order by 2 desc


