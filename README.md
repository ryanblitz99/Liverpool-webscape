# Table of contents

- [Introduction](#Introduction)

- [Technologies](#Technologies)

- [Data](#Data)

- [webscraping logic](#webscraping logic)

- [Resulting dataframe](#Resulting dataframe)

#### Introduction

The goal of the project is to:

1. Webscrape data of each liverpool match in the 2021/22 season, since my father is a Liverpool fan
2. Join this dataframe together with 'scores & fixtures' table to create a more comprehensive dataframe

#### Technologies

```
#import all the necessary modules
import numpy as np # linear algebra
import time
import os
import pandas as pd # data processing, CSV file I/O (e.g. pd.read_csv)
import requests
from bs4 import BeautifulSoup
import json
```

#### Data
We will be webscraping from the website https://fbref.com/en/. We will be looking to scrape 2 categories of tables:

1. The 'Scores & Fixtures' table
   ![image](https://github.com/ryanblitz99/Liverpool-webscape/assets/131364279/d821fb95-8ae9-4571-bb9c-1ba61d018a30)

2. The 'Liverpool Player stats' table for each match. In particular, we want the last row which contains the teams overall stats
   ![image](https://github.com/ryanblitz99/Liverpool-webscape/assets/131364279/7e50ff11-b8c2-460c-b7bc-50223a1c1414)

We will be joining these 2 tables on the 'Venue" which indicates if Liverpool were the Home or Away team, as well as the match date

#### webscraping logic
The first step would be to parse the main table containing all the premier league teams' final standings
```
base_url = 'https://fbref.com/en/comps/9/2021-2022/2021-2022-Premier-League-Stats'

response = requests.get(base_url)

soup = BeautifulSoup(response.content, "html")

print(response)
```

All of Liverpool's data can be found by clicking on their name in the table. For this we need to find the 'table' tag and class of this table
```
The table that we want is under the <table> tab with class = 'stats table'
#lets select the table

standings = soup.select('table', class_ = 'stats_table')


#now lets parse the contents of the table
standings_table = soup.select('table.stats_table')[0]
print(standings_table)

```

Next, we find the links for each team, let's append them into a list called "links_list"
```
#next we need to find the links for each team, they are under the 'a' class
links = standings_table.find_all('a')
links

#we are only interested in the href links
links_list = []
for l in links:
  links_list.append(l.get('href'))

links_list

```

Liverpool's link index in the list is 3
```
#we only want to look at liverpool
liverpool = 'https://fbref.com' + links_list[3]

liverpool

```
We now get the final link for all Liverpool data as 'https://fbref.com/en/squads/822bd0ba/2021-2022/Liverpool-Stats'

Now lets parse this web link and use the pandas function "pd.read_html()" to load the "Scores & Fixtures" table
```
#lets parse the liverpool data
data = requests.get(liverpool)
data.text

#get all scores and fixtures data
#use pandas.read_html function to get the table we want
scores_fixtures = pd.read_html(data.text, match = 'Scores & Fixtures')
scores_fixtures_df = scores_fixtures[0]

scores_fixtures_df = scores_fixtures_df.drop('Match Report', axis=1)
scores_fixtures_df

```

#parsing this table, we can find the links for data of all liverpool matches

[<a href="/en/matches/c52500ad/Norwich-City-Liverpool-August-14-2021-Premier-League">2021-08-14</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Premier League</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Matchweek 1</a>,
 <a href="/en/squads/1c781004/2021-2022/Norwich-City-Stats">Norwich City</a>,
 <a href="/en/players/2f90f6b8/James-Milner">James Milner</a>,
 <a href="/en/matches/c52500ad/Norwich-City-Liverpool-August-14-2021-Premier-League">Match Report</a>,
 <a href="/en/matches/94d9dac0/Liverpool-Burnley-August-21-2021-Premier-League">2021-08-21</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Premier League</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Matchweek 2</a>,
 <a href="/en/squads/943e8050/2021-2022/Burnley-Stats">Burnley</a>,
 <a href="/en/players/935e6b8f/Jordan-Henderson">Jordan Henderson</a>,
 <a href="/en/matches/94d9dac0/Liverpool-Burnley-August-21-2021-Premier-League">Match Report</a>,
 <a href="/en/matches/78aa75e6/Liverpool-Chelsea-August-28-2021-Premier-League">2021-08-28</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Premier League</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Matchweek 3</a>,
 <a href="/en/squads/cff3d9bb/2021-2022/Chelsea-Stats">Chelsea</a>,
 <a href="/en/players/935e6b8f/Jordan-Henderson">Jordan Henderson</a>,
 <a href="/en/matches/78aa75e6/Liverpool-Chelsea-August-28-2021-Premier-League">Match Report</a>,
 <a href="/en/matches/e6a245be/Leeds-United-Liverpool-September-12-2021-Premier-League">2021-09-12</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Premier League</a>,
 <a href="/en/comps/9/2021-2022/schedule/2021-2022-Premier-League-Scores-and-Fixtures">Matchweek 4</a>,
 <a href="/en/squads/5bfb9659/2021-2022/Leeds-United-Stats">Leeds United</a>,
 <a href="/en/players/e06683ca/Virgil-van-Dijk">Virgil van Dijk</a>,
 <a href="/en/matches/e6a245be/Leeds-United-Liverpool-September-12-2021-Premier-League">Match Report</a>,
 <a href="/en/matches/ff3e4ae2/Liverpool-Milan-September-15-2021-Champions-League">2021-09-15</a>,
...
 <a href="/en/comps/8/2021-2022/schedule/2021-2022-Champions-League-Scores-and-Fixtures">Champions Lg</a>,
 <a href="/en/comps/8/2021-2022/schedule/2021-2022-Champions-League-Scores-and-Fixtures">Final</a>,
 <a href="/en/squads/53a2f082/2021-2022/Real-Madrid-Stats">Real Madrid</a>,
 <a href="/en/players/935e6b8f/Jordan-Henderson">Jordan Henderson</a>,
 <a href="/en/matches/e81453d3/Liverpool-Real-Madrid-May-28-2022-Champions-League">Match Report</a>]

 However, as we can see, not all links contain match data, we hence have to filter out for the match links that we want

```
#get the hrefs of the match reports
reports_list = []
for r in reports:
  reports_list.append(r.get('href'))



#we only want the links with 'matches' in them
match_stats = []
for x in reports_list:
  if '/matches/' in x:
     y =  'https://fbref.com'
     link_string = y + x
     match_stats.append(link_string)



match_stat_list_final = []
for item in match_stats:
  if item not in match_stat_list_final:
    match_stat_list_final.append(item)


match_stat_list_final
```
We should get the following output:
['https://fbref.com/en/matches/c52500ad/Norwich-City-Liverpool-August-14-2021-Premier-League',
 'https://fbref.com/en/matches/94d9dac0/Liverpool-Burnley-August-21-2021-Premier-League',
 'https://fbref.com/en/matches/78aa75e6/Liverpool-Chelsea-August-28-2021-Premier-League',
 'https://fbref.com/en/matches/e6a245be/Leeds-United-Liverpool-September-12-2021-Premier-League',
 'https://fbref.com/en/matches/ff3e4ae2/Liverpool-Milan-September-15-2021-Champions-League',
 'https://fbref.com/en/matches/59ef8c18/Liverpool-Crystal-Palace-September-18-2021-Premier-League',
 'https://fbref.com/en/matches/1d32b710/Norwich-City-Liverpool-September-21-2021-EFL-Cup',
 'https://fbref.com/en/matches/87140543/Brentford-Liverpool-September-25-2021-Premier-League',
 'https://fbref.com/en/matches/1c876d93/Porto-Liverpool-September-28-2021-Champions-League',
 'https://fbref.com/en/matches/2598b046/Liverpool-Manchester-City-October-3-2021-Premier-League',
 'https://fbref.com/en/matches/2ce6d340/Watford-Liverpool-October-16-2021-Premier-League',
 'https://fbref.com/en/matches/0f05c98d/Atletico-Madrid-Liverpool-October-19-2021-Champions-League',
 'https://fbref.com/en/matches/b886bec4/North-West-Derby-Manchester-United-Liverpool-October-24-2021-Premier-League',
 'https://fbref.com/en/matches/54dca3c9/Preston-North-End-Liverpool-October-27-2021-EFL-Cup',
 'https://fbref.com/en/matches/c68998b5/Liverpool-Brighton-and-Hove-Albion-October-30-2021-Premier-League',
 'https://fbref.com/en/matches/d1319048/Liverpool-Atletico-Madrid-November-3-2021-Champions-League',
 'https://fbref.com/en/matches/63538dc7/West-Ham-United-Liverpool-November-7-2021-Premier-League',
 'https://fbref.com/en/matches/e9ea66e1/Liverpool-Arsenal-November-20-2021-Premier-League',
 'https://fbref.com/en/matches/a8898813/Liverpool-Porto-November-24-2021-Champions-League',
 'https://fbref.com/en/matches/aa9b882d/Liverpool-Southampton-November-27-2021-Premier-League',
 'https://fbref.com/en/matches/b46554fb/Merseyside-Derby-Everton-Liverpool-December-1-2021-Premier-League',
 'https://fbref.com/en/matches/37d61ff3/Wolverhampton-Wanderers-Liverpool-December-4-2021-Premier-League',
 'https://fbref.com/en/matches/dd8d2903/Milan-Liverpool-December-7-2021-Champions-League',
 'https://fbref.com/en/matches/678d7dca/Liverpool-Aston-Villa-December-11-2021-Premier-League',
 'https://fbref.com/en/matches/5aec4772/Liverpool-Newcastle-United-December-16-2021-Premier-League',
...
 'https://fbref.com/en/matches/1016efad/Aston-Villa-Liverpool-May-10-2022-Premier-League',
 'https://fbref.com/en/matches/42e0c267/Chelsea-Liverpool-May-14-2022-FA-Cup',
 'https://fbref.com/en/matches/733409b2/Southampton-Liverpool-May-17-2022-Premier-League',
 'https://fbref.com/en/matches/8b0f3e28/Liverpool-Wolverhampton-Wanderers-May-22-2022-Premier-League',
 'https://fbref.com/en/matches/e81453d3/Liverpool-Real-Madrid-May-28-2022-Champions-League']

As the match date for each match is not shown in the table itself, we have to extract the date from the html address itself. We get all the match dates and append them to a list
```
#get the dates of the matches in the match reports
dates_list = []
for r in reports:
  reports_list.append(r.get('href'))
  r_string = str(r)

  if ('/en/matches/' in r_string) and ('Match Report' not in r_string):
    first_index = r_string.find('>') + 1
    last_index = r_string.rfind('<')
    date_string = r_string[first_index:last_index]
    dates_list.append(date_string)

dates_list
len(dates_list)


```

Now with the match data urls and match date information, we are ready to parse the match summary statistics for each match and combine them all into 1 dataframe

I found that the "Liverpool player stats" table has multiple tabs
#EXTRACT the last row of the df, we only want the team's stats for that match
            #match[0] : gives match summary
            #match[1] : passing summary
            #match[2] : pass types
            #match[3] : defensive actions
            #match[4] : possession
            #match[5] : misc stats

We are only going to be looking at the match summary table. The table itself also has a multi-hierachical index:
![image](https://github.com/ryanblitz99/Liverpool-webscape/assets/131364279/605c4c35-61fb-43cd-b16c-cb6304881872)

Because of the large amount of data we are parsing, I often got timed out due to the large number of requests that im sending to the fbref server. This why we implement a time delay of 10 seconds using "time.sleep(10)" and we send a request to the server by session instead of doing it in 1 shot.

```
#now lets define a function to do this for all matches
#arguments: match_stat_list_final = list of all match stat urls
def get_match_stats(match_stat_list_final):
    
    #define the list to store all the values

    lst = []
    
    with requests.Session() as session:
        #lets try getting the first batch
        for i in range(0,len(match_stat_list_final+1)):
            match_url = match_stat_list_final[i]
            request = session.get(match_url)
            if request.status_code == 429:
                # Wait for a certain amount of time (e.g., 5 seconds) and then retry
                time.sleep(10)
                request = requests.get(match_url)
            #find the index of the last instance of the character '/'
            #we want to check if liverpool was the home team etc. '/Liverpool' or away
            last = match_stat_list_final[i].rfind('/')

            if match_stat_list_final[i][last+1:last+10] == 'Liverpool':
                Venue = 'Home'

            else:
                Venue = 'Away'

            #get the date of the match
            date = dates_list[i]


            match_stat_table = pd.read_html(request.text, match = 'Liverpool Player Stats')
            #EXTRACT the last row of the df, we only want the team's stats for that match
            #we are also just looking at the summary table, particularly the last row that contains the combined team's stats

            #EXTRACT the last row of the df, we only want the team's stats for that match
            #match[0] : gives match summary
            #match[1] : passing summary
            #match[2] : pass types
            #match[3] : defensive actions
            #match[4] : possession
            #match[5] : misc stats
            final = match_stat_table[0].iloc[[-1]] 
        
            try:
                performance = final[['Performance'][0]]
                expected = final[['Expected'][0]]
                sca = final[['SCA'][0]]
                passes = final[['Passes'][0]]
                #rename variable 'Att' so no key clash
                passes = passes.rename({'Att':'Pass_att'}, axis=1)
                carries = final[['Carries'][0]]
                take_ons = final[['Take-Ons'][0]]
                #rename column 'Att' so the meaning is clearer
                take_ons = take_ons.rename({'Att':'Take_on_att'}, axis=1)
            
            except KeyError or ValueError:
                continue


            #concat all the df
            try:
                final_values1 = pd.concat([performance,expected], axis=1)
                final_values2 = pd.concat([final_values1,sca,passes,carries,take_ons], axis=1)
                #get the row values only
                #get the row values only
                values = final_values2.iloc[0].tolist()
                #append the addtional value 'Venue'
                values.append(Venue)

                #append the addtional value 'Date'
                values.append(date)
                
                lst.append(values)


            except ValueError:
                continue

          
    #lastly, create the dataframe
    df_final = pd.DataFrame(data = lst, columns = ['Goals','Assists','PK','PKatt','Sh','SoT','CrdY','CrdR','Touches','Tkl','Int','Blocks','xG','npxG','xAG'
            'SCA','GCA','Cmp','Cmp_att','Cmp%','PrgP','Carries','PrgC','Tackle_att','Tackle_succ','Venue','Venue_type','Match_date'])
    

    return df_final

```

running the function, we get the output:
![image](https://github.com/ryanblitz99/Liverpool-webscape/assets/131364279/55e122d5-be45-40c9-9b85-55aa686025c6)

I noticed that there was an error with how the dataframes were being concatenated. For example, the 'xAG' column title was being merged together with the 'SCA' title to get the final column title of 'xAGSCA'. While at first I was unsure about what was causing this, I eventually realised this could be due to a key mismatch error:
![image](https://github.com/ryanblitz99/Liverpool-webscape/assets/131364279/7399c36a-8261-4778-a02a-295272bf5a37)

The subheader 'SCA' was the same as the column name 'SCA'. As such, I had to manualy rename the columns so the column title and data matched:

```
#rename columns so no clash
df_match_stats = df_match_stats.rename({'xAGSCA':'xAG', 'GCA':'SCA', 'Cmp':'GCA', 'Cmp_att':'Cmp', 'Cmp%':'Cmp_att', 'PrgP':'Cmp%', 'Carries':'PrgP', 'PrgC':'Carries', 'Tackle_att':'PrgC', 'Tackle_succ':'Tackle_att', 'Venue':'Tackle_succ','Venue_type':'Venue'}, axis=1)

df_match_stats

```

This eventually gave us the right format that we were loking for. Now we can join this table with the 'Scores & Fixtures' dataframe
```
#join the dataframe with the scores & fixtures dataframe
df_match_stats = df_match_stats.merge(scores_fixtures_df, left_on = ['Venue','Match_date'], right_on = ['Venue','Date'], how = 'left')

df_match_stats

```

Lastly, save this dataframe to a path
```
#save the dataframe into the path as a csv
outname = 'liverpool.csv'

outdir = 'C:/Users/Ryan Goh/Desktop/liverpool_data/data/raw'
if not os.path.exists(outdir):
    os.mkdir(outdir)
    
df_match_stats.to_csv(f"{outdir}/{outname}")

```

#### Resulting dataframe
Below are the following columns and their defintions:

Goals: Total goals scored
Assists: Total assists recorded
PK: Penalty kicks awarded	
PKatt: Penalty kick attempts
Sh: Total shots made	
SoT: Total shots on target
CrdY: Total number of yellow cards	
CrdR: Total number of red cards
Touches: Total number of touches on the ball	
Tkl: Total number of tackles made
Int: Total numbe rof interceptions recorded
Blocks: Total number of shots blocked
xG_x: Total number of expected goals	
npxG: Total number of expected non penalty goals
xAG: Total number of expected assists
SCA: Total number of shot creating actions
GCA: Total number of goal creating actions
Cmp: Total number of passes completed
Cmp_att: Total number of pass attempts
Cmp%: Pass completion percentage
PrgP: Total number of progressive passes	
Carries: Total number of ball carries
PrgC: Total number of progressive carries
Tackle_att: Total number of tackle attempts
Tackle_succ: Total number of successful tackles
Venue: Indicator if Liverpool were the home or away side
Match_date: match date	
Time_x: Kick-off time	
Comp_x: Competiton type(eg. premier league)
Round_x: Round of tournament
Day_x: Day of match
Result_x: W/L/D
GF_x: goals scored	
GA_x: goals conceeded
Opponent_x: opponent name	
xG_y	
xGA_x	
Poss_x: Possession %
Attendance_x: Total spectator attendance	
Captain_x: Which player captained liverpool in that match
Formation_x: formation used at the start of the match
Referee_x: Referee name
Notes_x	
Date_y	
Time_y	
Comp_y	
Round_y	
Day_y	
Result_y	
Opponent_y	
Poss_y 
Attendance_y	
Captain_y	
Formation_y	
Referee_y	
Notes_y
![image](https://github.com/ryanblitz99/Liverpool-webscape/assets/131364279/c21de831-3c36-4d7a-8280-669f4e26343d)

