# [The SQL Murder Mystery](http://mystery.knightlab.com/)

A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a __murder__ that occurred sometime on __Jan.15, 2018__ and that it took place in __SQL City__. Start by retrieving the corresponding crime scene report from the police department’s database.

### Searching for murder on __January 15th, 2018__

~~~sql
SQL
SELECT * FROM crime_scene_report
WHERE date=20180115 AND city='SQL City';
~~~
**Output:**
>Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave".

## **Seeking the witnesses**

#### The first witness, the one who lives in the last house of "Northwestern Dr".

~~~sql

SELECT * FROM person 
WHERE address_street_name='Northwestern Dr' 
Order by address_number desc
LIMIT 1
~~~

**Results:**

| id | name | lincense_id | address_number | address_stress_name| ssn |
| --- | --- | ---| --- | --- | --- | 
| 14887 | Morty Schapiro | 118009 | 4919 | Northwestern Dr | 111564949


### The second witness, named Annabel, lives somewhere on "Franklin Ave".
~~~sql
SELECT * FROM person 
WHERE name like "%Annabel%" AND address_street_name like '%Franklin Ave%'
~~~

**Results:**
| id | name | lincense_id | address_number | address_stress_name| ssn |
| --- | --- | ---| --- | --- | --- | 
| 16371 | Annabel Miller | 490173 | 103 | Franklin Ave | 318771143 |

## **Witnesses Interviews**
~~~sql
SELECT * FROM interview
WHERE person_id IN (14887, 16371)
~~~

**First Witness Transcripts:**
>I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W".

**Second Witness Transcripts:**
>I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th.

## **Suspects**
Cross-referencing the gym memberships, gym check-ins, and drivers license -

Gym membership info:
~~~sql
SELECT person_id, name FROM get_fit_now_check_in as t1 INNER JOIN get_fit_now_member as t2                   
ON t1.membership_id = t2.id
WHERE t1.membership_id LIKE '%48Z%' AND check_in_date = '20180109' AND t2.membership_status = 'gold'
~~~
**Results:**
| person_id | name |
| --- | --- |
| 28819 | Joe Germuska
| 67318 | Jeremy Bowers

License Plate Lookup:

~~~sql
Select name, gender, plate_number FROM drivers_license INNER JOIN person
ON drivers_license.id = person.license_id
WHERE person.id IN (28819, 67318)
~~~
**Results:**
|name | gender | plate_number
| --- | --- | ---|
| Jeremy Bowers | Male | 0H42W2

***A match! This is our man.***

Checking Solution:
Did you find the killer?
~~~sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
        
        SELECT value FROM solution;
~~~

**Output:**
>Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer.

## **Further Investigation:**
~~~sql
Select *
From interview
Where person_id = '67318';
~~~

**Suspect Transcript:**
>I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017.

**Finding the woman in question:**
~~~sql
SELECT p.name, dl.gender, dl.height, dl.hair_color, dl.car_make, dl.car_model, fb.event_name, count(fb.event_id) AS Event_count FROM facebook_event_checkin AS fb INNER JOIN person AS p ON fb.person_id = p.id 
INNER JOIN drivers_license AS dl ON p.license_id = dl.id
WHERE car_make = "Tesla" AND car_model = "Model S" AND hair_color = "red" 
AND gender = "female" AND event_name = 'SQL Symphony Concert' AND height BETWEEN 65 AND 67
GROUP BY p.id
~~~
**Results:**
|Name | Gender | Height | Hair color | Car make | Car model | Event name | Event Count|
| ---| ---| ---| ---| ---| ---| ---| ---|
| Miranda Priestly | Female | 66 | Red | Tesla | Model S | SQL Symphony Concert | 3

**Checking solution:**

Did you find the killer?
~~~sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
        
        SELECT value FROM solution;
~~~

**Output:** 
>Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!

