[The-TechNova-Murder-Mystery.pdf](https://github.com/user-attachments/files/24232254/The-TechNova-Murder-Mystery.pdf)
# Tech-Nova-Murder-Mystery
SQL-based forensic data analysis project that uses key card logs, call records, alibis, and evidence to solve a simulated corporate murder mystery.
SQL Capstone Project: TechNova Murder Mystery
Project Overview

This capstone project demonstrates the application of SQL for forensic data analysis through an investigative case study. The scenario revolves around the mysterious murder of the CEO of TechNova Inc., discovered on 15 October 2025 in the CEO’s office.

Using structured query language (SQL), we analyze multiple datasets to reconstruct events, verify alibis, trace movements, and ultimately identify the prime suspect. This project highlights real-world SQL skills such as joins, filtering, CTEs, and data correlation.

Objectives:

Identify the perpetrator using data-driven investigation

Pinpoint the exact time and location of the crime

Detect false alibis through cross-verification

Analyze keycard logs, call records, and physical evidence

Showcase advanced SQL querying and logical reasoning

Database Schema:

The investigation relies on five core tables:

Create  database  MurderMystery;
-- DROP TABLES if exist
DROP TABLE IF EXISTS employees, keycard_logs, calls, alibis, evidence;
use MurderMystery;

-- Employees Table
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    name VARCHAR(50),
    department VARCHAR(50),
    role VARCHAR(50)
);

INSERT INTO employees VALUES
(1, 'Alice Johnson', 'Engineering', 'Software Engineer'),
(2, 'Bob Smith', 'HR', 'HR Manager'),
(3, 'Clara Lee', 'Finance', 'Accountant'),
(4, 'David Kumar', 'Engineering', 'DevOps Engineer'),
(5, 'Eva Brown', 'Marketing', 'Marketing Lead'),
(6, 'Frank Li', 'Engineering', 'QA Engineer'),
(7, 'Grace Tan', 'Finance', 'CFO'),
(8, 'Henry Wu', 'Engineering', 'CTO'),
(9, 'Isla Patel', 'Support', 'Customer Support'),
(10, 'Jack Chen', 'HR', 'Recruiter');

-- Keycard Logs Table
CREATE TABLE keycard_logs (
    log_id INT PRIMARY KEY,
    employee_id INT,
    room VARCHAR(50),
    entry_time TIMESTAMP,
    exit_time TIMESTAMP
);

INSERT INTO keycard_logs VALUES
(1, 1, 'Office', '2025-10-15 08:00', '2025-10-15 12:00'),
(2, 2, 'HR Office', '2025-10-15 08:30', '2025-10-15 17:00'),
(3, 3, 'Finance Office', '2025-10-15 08:45', '2025-10-15 12:30'),
(4, 4, 'Server Room', '2025-10-15 08:50', '2025-10-15 09:10'),
(5, 5, 'Marketing Office', '2025-10-15 09:00', '2025-10-15 17:30'),
(6, 6, 'Office', '2025-10-15 08:30', '2025-10-15 12:30'),
(7, 7, 'Finance Office', '2025-10-15 08:00', '2025-10-15 18:00'),
(8, 8, 'Server Room', '2025-10-15 08:40', '2025-10-15 09:05'),
(9, 9, 'Support Office', '2025-10-15 08:30', '2025-10-15 16:30'),
(10, 10, 'HR Office', '2025-10-15 09:00', '2025-10-15 17:00'),
(11, 4, 'CEO Office', '2025-10-15 20:50', '2025-10-15 21:00'); -- killer

-- Calls Table
CREATE TABLE calls (
    call_id INT PRIMARY KEY,
    caller_id INT,
    receiver_id INT,
    call_time TIMESTAMP,
    duration_sec INT
);

INSERT INTO calls VALUES
(1, 4, 1, '2025-10-15 20:55', 45),
(2, 5, 1, '2025-10-15 19:30', 120),
(3, 3, 7, '2025-10-15 14:00', 60),
(4, 2, 10, '2025-10-15 16:30', 30),
(5, 4, 7, '2025-10-15 20:40', 90);

-- Alibis Table
CREATE TABLE alibis (
    alibi_id INT PRIMARY KEY,
    employee_id INT,
    claimed_location VARCHAR(50),
    claim_time TIMESTAMP
);

INSERT INTO alibis VALUES
(1, 1, 'Office', '2025-10-15 20:50'),
(2, 4, 'Server Room', '2025-10-15 20:50'), -- false alibi
(3, 5, 'Marketing Office', '2025-10-15 20:50'),
(4, 6, 'Office', '2025-10-15 20:50');

-- Evidence Table
CREATE TABLE evidence (
    evidence_id INT PRIMARY KEY,
    room VARCHAR(50),
    description VARCHAR(255),
    found_time TIMESTAMP
);

INSERT INTO evidence VALUES
(1, 'CEO Office', 'Fingerprint on desk', '2025-10-15 21:05'),
(2, 'CEO Office', 'Keycard swipe logs mismatch', '2025-10-15 21:10'),
(3, 'Server Room', 'Unusual access pattern', '2025-10-15 21:15');



Investigation Steps & SQL Analysis

-- 1. Who entered the **CEO’s Office** close to the time of the murder?

select e.name, kl.room,kl.entry_time,kl.exit_time
from employees e
join keycard_logs kl
on e.employee_id = kl.employee_id
where room = "CEO Office";

Insights:
Name         room         entry_time            exit_time
David Kumar	 CEO Office	 2025-10-15 20:50:00	  2025-10-15 21:00:00

-- 2. Who "claimed" to be somewhere else but was not?

select e.employee_id,e.name,kl.room,kl.entry_time,kl.exit_time,al.claimed_location,al.claim_time
from employees e 
Join keycard_logs kl
on kl.employee_id=e.employee_id
join alibis al 
on al.employee_id = e.employee_id  
where kl.room = "CEO Office";

Insights:
employee_id    name        room        entry_time              exit_time            claim_location    claim_time
4	           David Kumar	 CEO Office	 2025-10-15 20:50:00	  2025-10-15 21:00:00 	Server Room	      2025-10-15 20:50:00

-- 3. Who made or received calls around 20:50–21:00?

 select c.call_id,c.caller_id,caller.name,c.receiver_id,receiver.name,c.call_time,c.duration_sec
 from calls c 
join employees caller on c.caller_id=caller.employee_id
join employees receiver on c.receiver_id=receiver.employee_id
where call_time between "2025-10-15 20:50" and "2025-10-15 21:00"

Insights:
call_id   caller_id       name         receiver_id           name         call_time            duration(sec)
  1	          4	       David Kumar	      1	            Alice Johnson	  2025-10-15 20:55:00	        45

-- 4. What evidence was found at the "crime scene"?

select evidence_id,room,description,found_time
from evidence
where room="CEO Office"
order by found_time;

Insights:
evidence_id    room            description                   found_time
1	            CEO Office	  Fingerprint on desk	            2025-10-15 21:05:00
2	            CEO Office	  Keycard swipe logs mismatch	    2025-10-15 21:10:00
			

-- 5.. Which suspect’s movements, alibi, and call activity "don’t add up"?

with crime_location as
(select kl.employee_id,e.name,kl.room,kl.entry_time,kl.exit_time
from keycard_logs kl
join employees e on kl.employee_id=e.employee_id
where room= "CEO Office"
and entry_time between "2025-10-15 20:50"  and "2025-10-15 21:00") ,
false_alibis as
(select a.employee_id,a.claimed_location,a.claim_time,kl.room as actual_room
from employees e  
join alibis a on a.employee_id = e.employee_id
join keycard_logs kl on kl.employee_id=a.employee_id
and a.claim_time between kl.entry_time and exit_time
where a.claimed_location <> kl.room),
call_activity as
(select c.caller_id,e.name as callerName,c.call_time,c.duration_sec
from calls c 
join employees e 
on e.employee_id = c.caller_id
where call_time between "2025-10-15 20:45" and "2025-10-15 21:05")

select cl.employee_id,cl.name as killer,cl.room  as murder_location,fa.claimed_location as false_claimroom,
fa.claim_time as false_claimtime, ca.call_time as suspicious_call,ca.duration_sec as duration
from crime_location cl
Left join false_alibis fa on cl.employee_id = fa.employee_id
Left join call_activity ca on cl.employee_id= ca.caller_id;

Final Verdict:
employee_id   killer        murder_location  false_claimroom  false_claimtime        suspicious_call        duration
4	            David Kumar	  CEO Office	      Server Room	    2025-10-15 20:50:00	    2025-10-15 20:55:00	    45

Conclusion: All data points converge to identify David Kumar as the perpetrator.

Skills Demonstrated

Advanced SQL querying

INNER & LEFT JOINs

Time-based filtering

Common Table Expressions (CTEs)

Logical reasoning & data storytelling


