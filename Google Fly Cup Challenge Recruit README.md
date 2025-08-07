
# 🏁 Google Fly Cup Challenge: Recruit — Arcade Genius Edition

🔗 YouTube: [Arcade Genius YouTube Channel](https://www.youtube.com/@ArcadeGenius-z1)

---

## 🚀 Setup: Load the Dataset in Cloud Shell

```bash
for file in `gsutil ls gs://spls/gsp394/tables/*.csv`; do TABLE_NAME=`echo $file | cut -d '/' -f6 | cut -d '.' -f1`; bq load --autodetect --source_format=CSV --replace=true drl.$TABLE_NAME $file; done


---

🧩 Task 1: Query Events by City (Replace ENTER_YOUR_CITY_NAME)

SELECT name 
FROM `drl.events` 
WHERE city = 'ENTER_YOUR_CITY_NAME'


---

🧩 Task 2: Join Pilots with Event Pilots

SELECT 
  `drl.pilots`.name, 
  `drl.event_pilots`.id 
FROM `drl.event_pilots` 
LEFT JOIN `drl.pilots` 
ON `drl.pilots`.id = `drl.event_pilots`.pilot_id


---

🧩 Task 3: Get Pilots by Event (Replace ENTER_YOUR_EVENT_NAME)

SELECT 
  `drl.pilots`.name, 
  `drl.events`.name AS event_name 
FROM `drl.event_pilots` 
LEFT OUTER JOIN `drl.pilots` 
ON `drl.pilots`.id = `drl.event_pilots`.pilot_id 
LEFT OUTER JOIN `drl.events` 
ON `drl.events`.id = `drl.event_pilots`.event_id 
WHERE `drl.events`.name = 'ENTER_YOUR_EVENT_NAME'


---

🧩 Task 4: Average of 1st Rank Round Minimum Time

WITH cte AS (
  SELECT `drl.round_standings`.minimum_time 
  FROM `drl.round_standings` 
  WHERE `rank` = 1
)

SELECT TIME(
  TIMESTAMP_SECONDS(
    CAST(
      AVG(
        UNIX_SECONDS(
          PARSE_TIMESTAMP('%H:%M.%S', minimum_time)
        )
      ) AS INT64
    )
  )
) AS avg 
FROM cte


---

🧩 Task 5: Create Table drl.time_trial_cleaned

CREATE TABLE drl.time_trial_cleaned AS (
  SELECT
    `drl.time_trial_group_pilot_times`.id AS time_trial_group_pilot_times_id,
    `drl.time_trial_group_pilots`.id AS time_trial_group_pilot_id,
    `drl.time_trial_groups`.id AS time_trial_group_id,
    round_id,
    CASE
      WHEN `drl.time_trial_group_pilot_times`.time_adjusted IS NOT NULL THEN `drl.time_trial_group_pilot_times`.time_adjusted
      WHEN `drl.time_trial_groups`.racestack_scoring = 0 THEN `drl.time_trial_group_pilot_times`.time
      ELSE `drl.time_trial_group_pilot_times`.racestack_time
    END AS time
  FROM `drl.time_trial_group_pilot_times` 
  LEFT OUTER JOIN `drl.time_trial_group_pilots` 
    ON `drl.time_trial_group_pilot_times`.time_trial_group_pilot_id = `drl.time_trial_group_pilots`.id 
  LEFT OUTER JOIN `drl.time_trial_groups` 
    ON `drl.time_trial_group_pilots`.time_trial_group_id = `drl.time_trial_groups`.id
)


---

🧩 Task 6: Fastest Time for a Specific Event (Replace ENTER_YOUR_EVENT_NAME)

WITH cte AS (
  SELECT
    `drl.rounds`.event_id,
    `drl.rounds`.name,
    `drl.events`.name AS event_name,
    time
  FROM `drl.time_trial_cleaned`
  LEFT OUTER JOIN `drl.rounds` 
    ON `drl.time_trial_cleaned`.round_id = `drl.rounds`.id
  LEFT OUTER JOIN `drl.events` 
    ON `drl.events`.id = `drl.rounds`.event_id
)

SELECT MIN(time) AS fastest_time 
FROM cte 
WHERE event_name = 'ENTER_YOUR_EVENT_NAME' AND name = 'Time Trials'


---

🧩 Task 7: Pilot Stats in Heat (Replace ENTER_NAME)

SELECT
  `drl.pilots`.name AS pilot_name,
  `drl.heat_standings`.heat_id AS heat_id,
  `drl.heat_standings`.minimum_time,
  `drl.heat_standings`.points
FROM `drl.heat_standings`
LEFT JOIN `drl.event_pilots` 
  ON `drl.event_pilots`.id = event_pilot_id
LEFT JOIN `drl.pilots` 
  ON `drl.pilots`.id = `drl.event_pilots`.pilot_id
WHERE name = 'ENTER_NAME'
  AND minimum_time != 'NURK'
  AND minimum_time != ''


---

🧩 Task 8: Running Average per Heat (Replace ENTER_NAME)

WITH cte AS (
  SELECT 
    `drl.pilots`.name, 
    `drl.heat_standings`.heat_id, 
    `drl.heat_standings`.points, 
    `drl.heat_standings`.minimum_time
  FROM `drl.heat_standings`
  LEFT JOIN `drl.event_pilots` 
    ON `drl.event_pilots`.id = event_pilot_id
  LEFT JOIN `drl.pilots` 
    ON `drl.pilots`.id = `drl.event_pilots`.pilot_id
  WHERE name = 'ENTER_NAME' 
    AND minimum_time != 'DNF' 
    AND minimum_time != ''
)

SELECT
  name AS pilot_name,
  heat_id,
  minimum_time,
  points,
  TIME(
    TIMESTAMP_SECONDS(
      CAST(
        AVG(
          UNIX_SECONDS(
            PARSE_TIMESTAMP('%H:%M.%S', minimum_time)
          )
        ) OVER (ORDER BY heat_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
      AS INT64)
    )
  ) AS running_avg
FROM cte


---

🧩 Task 9: Compare Time with Running Average (Replace ENTER_NAME)

WITH cte AS (
  SELECT 
    `drl.pilots`.name, 
    `drl.heat_standings`.heat_id, 
    `drl.heat_standings`.points, 
    `drl.heat_standings`.minimum_time
  FROM `drl.heat_standings`
  LEFT JOIN `drl.event_pilots` 
    ON `drl.event_pilots`.id = event_pilot_id
  LEFT JOIN `drl.pilots` 
    ON `drl.pilots`.id = `drl.event_pilots`.pilot_id
  WHERE name = 'ENTER_NAME' 
    AND minimum_time != 'DNF' 
    AND minimum_time != ''
),

cte2 AS (
  SELECT 
    name AS pilot_name, 
    heat_id, 
    minimum_time, 
    points, 
    TIME(
      TIMESTAMP_SECONDS(
        CAST(
          AVG(
            UNIX_SECONDS(
              PARSE_TIMESTAMP('%H:%M.%S', minimum_time)
            )
          ) OVER (ORDER BY heat_id ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
        AS INT64)
      )
    ) AS running_avg 
  FROM cte
)

SELECT *,
  TIME_DIFF(
    PARSE_TIME('%H:%M.%S', minimum_time), 
    running_avg, 
    SECOND
  ) AS time_diff_from_avg 
FROM cte2


---

🎉 Congrats on Finishing the Lab!

If this helped, consider subscribing to the official Arcade Genius YouTube Channel 🚀
Happy Learning!

---

