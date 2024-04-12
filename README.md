/*-- Query 8:

From the following 3 tables (event_category, physician_speciality, patient_treatment),
write a SQL query to get the histogram of specialities of the unique physicians
who have done the procedures but never did prescribe anything.

--Table Structure:*/

```sql
drop table event_category;
create table event_category
(
  event_name varchar(50),
  category varchar(100)
);

drop table physician_speciality;
create table physician_speciality
(
  physician_id int,
  speciality varchar(50)
);

drop table patient_treatment;
create table patient_treatment
(
  patient_id int,
  event_name varchar(50),
  physician_id int
);


insert into event_category values ('Chemotherapy','Procedure');
insert into event_category values ('Radiation','Procedure');
insert into event_category values ('Immunosuppressants','Prescription');
insert into event_category values ('BTKI','Prescription');
insert into event_category values ('Biopsy','Test');


insert into physician_speciality values (1000,'Radiologist');
insert into physician_speciality values (2000,'Oncologist');
insert into physician_speciality values (3000,'Hermatologist');
insert into physician_speciality values (4000,'Oncologist');
insert into physician_speciality values (5000,'Pathologist');
insert into physician_speciality values (6000,'Oncologist');


insert into patient_treatment values (1,'Radiation', 1000);
insert into patient_treatment values (2,'Chemotherapy', 2000);
insert into patient_treatment values (1,'Biopsy', 1000);
insert into patient_treatment values (3,'Immunosuppressants', 2000);
insert into patient_treatment values (4,'BTKI', 3000);
insert into patient_treatment values (5,'Radiation', 4000);
insert into patient_treatment values (4,'Chemotherapy', 2000);
insert into patient_treatment values (1,'Biopsy', 5000);
insert into patient_treatment values (6,'Chemotherapy', 6000);


select * from patient_treatment;
select * from event_category;
select * from physician_speciality;


SELECT T3.SPECIALITY, count(1) as speciality_count
FROM PATIENT_TREATMENT T1
JOIN EVENT_CATEGORY T2 ON T1.EVENT_NAME = T2.EVENT_NAME
JOIN PHYSICIAN_SPECIALITY T3 ON T1.PHYSICIAN_ID = T3.PHYSICIAN_ID
WHERE T2.CATEGORY = 'PROCEDURE'
AND T1.PHYSICIAN_ID NOT IN ( SELECT TP.PHYSICIAN_ID
							FROM PATIENT_TREATMENT TP
							JOIN EVENT_CATEGORY TE ON TP.EVENT_NAME = TE.EVENT_NAME
							WHERE TE.CATEGORY IN ('PRESCRIPTION'))
GROUP BY 1;
```
This SQL query retrieves the count of each physician's specialty for procedures performed, excluding physicians who only perform prescriptions. Let's break it down:

### SELECT Statement:

SELECT T3.SPECIALITY: This selects the specialty of physicians from the PHYSICIAN_SPECIALITY table.
count(1) as speciality_count: This counts the number of occurrences of each specialty and aliases the result column as "speciality_count".

### FROM Clause:
FROM PATIENT_TREATMENT T1: This specifies the PATIENT_TREATMENT table and aliases it as "T1".
JOIN EVENT_CATEGORY T2 ON T1.EVENT_NAME = T2.EVENT_NAME: This joins the PATIENT_TREATMENT table with the EVENT_CATEGORY table based on the EVENT_NAME column to link treatments with event categories. It aliases the EVENT_CATEGORY table as "T2".
JOIN PHYSICIAN_SPECIALITY T3 ON T1.PHYSICIAN_ID = T3.PHYSICIAN_ID: This joins the PATIENT_TREATMENT table with the PHYSICIAN_SPECIALITY table based on the PHYSICIAN_ID column to link treatments with physician specialties. It aliases the PHYSICIAN_SPECIALITY table as "T3".

### WHERE Clause:
WHERE T2.CATEGORY = 'PROCEDURE': This condition filters the results to only include procedures by selecting rows where the event category is 'PROCEDURE'.
AND T1.PHYSICIAN_ID NOT IN (...): This subquery excludes physicians who only perform prescriptions. It selects physicians from the PATIENT_TREATMENT table who are associated with events categorized as 'PRESCRIPTION' in the EVENT_CATEGORY table.

### GROUP BY Clause:
GROUP BY 1: This groups the results by the first column in the SELECT statement, which is T3.SPECIALITY. It means that the count of each specialty will be calculated separately.

### Summary :
This query retrieves the count of each physician's specialty for procedures performed, excluding physicians who only perform prescriptions. It groups the results by specialty.

### Output
![image](https://github.com/vijaykumarmech1997/SQL/assets/128212120/dbd83d2a-cc30-40a9-9503-833ddb988e0d)
