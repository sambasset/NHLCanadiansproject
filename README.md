# NHLCanadiansproject
SQL Repository for Tableau Desktop Project entitled "Canadians in the NHL from 2017/18 to 2021/22"

-- Primary Question: 
    -- How many players from each province played in the NHL from the years 2017/18 to 2021/22?

--  To answer this question I used datasets from..
  
  --  NHL Game Data by Martin Ellis : [Link](https://www.kaggle.com/datasets/martinellis/nhl-game-data)
        This dataset has player information dating years back so it wouldn’t be a clear representation of players who only played a minimum of 1 game in the      
        seasons I want
    NHL Player and Goalie Stats by Season from Moneypuck : [Link](https://moneypuck.com/data.htm)
        This data is organized season by season so I can isolate the data for skaters and goalies for just the seasons I want.

-- The Moneypuck dataset had every single player that played a minimum of 1 game in the NHL during the 2021-2022 season and the NHL Game Data set had player stat info as well as what nationality and what State/Province the player was from.
 
-- In order to get the data I desired I had to consolidate the goalie and skater datasets (they were separate) and then join them with the NHL Game Data set so each player could have an associated birth province.
 
-- The datasets shared a playerID column.

## Cleaning ##

-- Checking for NULLS

SELECT player_id, birthStateProvince, nationality, lastName
  FROM `clean-result-365422.NHL.player_info`
    WHERE player_id IS NULL OR 
      birthStateProvince IS NULL OR 
      nationality IS NULL OR 
      lastName IS NULL
      
      -- Returned no data

-- To check the Moneypuck Data Sets we grabbed the years we wanted and first wanted to union the Skaters and Goalies

SELECT playerId, name, position, team, situation, season
  FROM `clean-result-365422.NHL.goalies_2021_moneypuck`
    WHERE situation = "all"

  UNION ALL
    
    SELECT playerId, name, position, team, situation, season
      FROM `clean-result-365422.NHL.skaters_2021_moneypuck`
        WHERE situation = "all"
        
        -- We use "all" in WHERE because the dataset would return duplicate names because it also has "4on4", "5on5", etc for each situation a player may find themselved in on the ice.
        
-- Check nulls on new dataset that merged the goalies and skaters (NHL_2021_skater_and_goalie)

SELECT playerId, name, team, situation, season, position
  FROM `clean-result-365422.NHL.NHL_2021_skater_and_goalie`
    WHERE playerId IS NULL OR
      name IS NULL OR
      team IS NULL OR
      situation IS NULL OR
      season IS NULL OR
      position IS NULL 

-- This happened later in my work on this project however there were inproper codes for the names. Ex. "LAK" (Los Angeles Kings) in a different year was labelled as "L.A"

-- First I wanted to merge all the seasons I needed (after using the union I did on Line 34 for each season)

SELECT *
  FROM `clean-result-365422.NHL.NHL_2017_2018_ALL`
    UNION ALL
  SELECT *
    FROM `clean-result-365422.NHL.NHL_2018_2019_ALL`
      UNION ALL
    SELECT *
      FROM `clean-result-365422.NHL.NHL_2019_2020_ALL`
        UNION ALL
      SELECT *
          FROM `clean-result-365422.NHL.NHL_2020_2021_ALL`
            UNION ALL
          SELECT * 
            FROM `clean-result-365422.NHL.NHL_2021_skater_and_goalie`
           
-- This was saved as a table named NHL_2017_2021_all.

-- Verifying that the player id's are consistent across both datasets. Using a mix of current, retired and low usage players
 
SELECT playerId
  FROM `clean-result-365422.NHL.skaters_2021_moneypuck`
    WHERE name = "Sidney Crosby"
 
SELECT playerId
  FROM `clean-result-365422.NHL.skaters_2021_moneypuck`
    WHERE name = "Jonathan Toews"
 
SELECT playerId
  FROM `clean-result-365422.NHL.skaters_2021_moneypuck`
    WHERE name = "Henrik Sedin"
 
SELECT player_id
  FROM `clean-result-365422.NHL.player_info`
    WHERE lastName = "Crosby"
 
SELECT player_id, 
  FROM `clean-result-365422.NHL.player_info`
    WHERE lastName = "Toews"

-- Toews returned two ID's, added "firstName" to clarify. Could also use a concat to add first name and last name together then use WHERE to filter to Toews.
 
SELECT player_id, firstName
  FROM `clean-result-365422.NHL.player_info`
    WHERE lastName = "Toews"
 
-- Checked some retired players to verify they were in the system.
 
SELECT player_id, firstName
  FROM `clean-result-365422.NHL.player_info`
    WHERE lastName = "Sedin"
 
SELECT player_id, firstName
  FROM `clean-result-365422.NHL.player_info`
    WHERE lastName = "Sundin"
 
-- Check to see if Adam Brooks (Played 14 games for WPG) is in system
 
SELECT player_id, firstName
  FROM `clean-result-365422.NHL.player_info`
    WHERE lastName = "Brooks"
    
-- Next I am checking to see what team code names are incorrect and then fixing to create consistency. (should be 32 teams) *This was actually found later in my initial work but adding it here for clarity*

SELECT DISTINCT team
  FROM `clean-result-365422.NHL.NHL_2017_2021_all`
    ORDER BY team DESC

-- returned 36 teams. Incorrect teams were .. L.A to LAK. N.J to NJD. T.B to TBL. S.J to SJS. Need to update the dataset.

UPDATE `clean-result-365422.NHL.NHL_2017_2021_all`
  SET team = "SJS"
    WHERE team = "S.J"

UPDATE `clean-result-365422.NHL.NHL_2017_2021_all`
  SET team = "LAK"
    WHERE team = "L.A"

UPDATE `clean-result-365422.NHL.NHL_2017_2021_all`
  SET team = "TBL"
    WHERE team = "T.B"

UPDATE `clean-result-365422.NHL.NHL_2017_2021_all`
  SET team = "NJD"
    WHERE team = "N.J"

-- Now returns 32 Teams

-- Want to make sure provinces are also consistent in player info dataset. *This was actually found later in my initial work but adding it here for clarity*

SELECT DISTINCT birthStateProvince
  FROM `clean-result-365422.NHL.player_info`
    WHERE nationality = "CAN"
    
    -- Returned some state codes, should be 10 distinct returns, I got 19. Going to update to fix, will give one example then just did with the rest.

UPDATE `clean-result-365422.NHL.player_info`
  SET nationality = "USA"
    WHERE birthStateProvince = "AZ"
        
        -- Now returns 10 (10 provinces)
        
-- Going to join the tables now to create the table to export to tableau

SELECT s.name, s.position, s.team, s.situation, s.season, p.nationality, p.birthStateProvince as province
  FROM `clean-result-365422.NHL.NHL_2017_2021_all` AS s
    JOIN `clean-result-365422.NHL.player_info` AS p
      ON s.playerId = p.player_id
        WHERE nationality = "CAN"
        
        -- Exported to CSV as FINAL_NHL_2017_2021_all


