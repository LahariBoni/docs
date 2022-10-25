<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Draft%20Kings/Draft_Kings_Get_NBA_Moneylines.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Draft+Kings+-+Get+NBA+Moneylines:+Error+short+description">🚨 Bug report</a>

**Tags:** #draftkings #nba #betting #python #analytics #automation #sports #sports_betting #opendata #notification #email

**Author:** [JA Williams](https://www.linkedin.com/in/ja-williams-529517187/)

This notebooks gathers the betting lines for NBA games each day. It scrapes the Moneylines for each team from the Draft Kings website.

Data source: https://sportsbook.draftkings.com/leagues/basketball/88670846?wpsrc=Organic%20Search&wpaffn=Google&wpkw=https%3A%2F%2Fsportsbook.draftkings.com%2Fleagues%2Fbasketball%2F88670846&wpcn=leagues&wpscn=basketball%2F88670846

## Input

### Import libraries


```python
import naas
import requests
import pandas as pd
from bs4 import BeautifulSoup
from naas_drivers import emailbuilder
from datetime import datetime
import pytz
```

### Setup Draft Kings


```python
# URL to scrap data
URL = 'https://sportsbook.draftkings.com/leagues/basketball/88670846'
```

### Setup Naas


```python
# Get all timezone
pytz.all_timezones
```


```python
# Set Time zone
TIME_ZONE = "America/New_York"
naas.set_remote_timezone(TIME_ZONE)
```


```python
# Schedule the Notebook to run at 11 AM each day of the NBA season
naas.scheduler.add(cron='0 11 * 10,11,12,1,2,3,4,5 *')

# To delete your scheduler, uncomment the line below and execute the cell
# naas.scheduler.delete()
```


```python
# Email
EMAIL_TO = "<YOUR_EMAIL>"
TODAY = datetime.now(pytz.timezone(TIME_ZONE)).strftime("%Y-%m-%d")
EMAIL_SUBJECT = f"🏀 Draft Kings : Your NBA game of the day {TODAY}"
```

## Model

### Scrape Data from Draft Kings


```python
page = requests.get(URL)
soup = BeautifulSoup(page.content, "html.parser")
```


```python
results = soup.find(id='root')
```


```python
div = results.find_all('div', class_='parlay-card-10-a')
```

### Separate the Elements


```python
teams = []
lines = []

for element in div:
    team = element.find_all("div", class_="event-cell__name-text")
    teams.append(f'{team}')
    line = element.find_all("span", class_="sportsbook-odds american no-margin default-color")
    lines.append(f'{line}')
```

### Clean Team Element


```python
team_a = teams[0]
team_b = teams[1]
team_list_a = team_a.split(", ")
team_list_b = team_b.split(",")
```


```python
team_list2_a = []
team_list2_b = []

for t in team_list_a:
    new = t[:-6]
    team_list2_a.append(new)

for t in team_list_b:
    new = t[:-6]
    team_list2_b.append(new)
```


```python
team_list3_a = []
team_list3_b = []

for x in team_list2_a:
    new = x[35:]
    team_list3_a.append(new)

for x in team_list2_b:
    new = x[35:]
    team_list3_b.append(new)
```


```python
team_list4_a = [q.replace('>', '') for q in team_list3_a]
dk_team_names_a = [q.replace('<', '') for q in team_list4_a]

team_list4_b = [q.replace('>', '') for q in team_list3_b]
dk_team_names_b = [q.replace('<', '') for q in team_list4_b]
```


```python
dk_team_names_a.extend(dk_team_names_b)
```


```python
# Change Team Names to Match with 3-letter Code
new1 = [team.replace('CLE Cavaliers', 'CLE') for team in dk_team_names_a]
new2 = [team.replace('CHI Bulls', 'CHI') for team in new1]
new3 = [team.replace('MIN Timberwolves', 'MIN') for team in new2]
new4 = [team.replace('MIA Heat', 'MIA') for team in new3]
new5 = [team.replace('IND Pacers', 'IND') for team in new4]
new6 = [team.replace('SA Spurs', 'SAS') for team in new5]
new7 = [team.replace('MIL Bucks', 'MIL') for team in new6]
new8 = [team.replace('GS Warriors', 'GSW') for team in new7]
new9 = [team.replace('SAC Kings', 'SAC') for team in new8]
new10 = [team.replace('UTA Jazz', 'UTA') for team in new9]
new11 = [team.replace('TOR Raptors', 'TOR') for team in new10]
new12 = [team.replace('DEN Nuggets', 'DEN') for team in new11]
new13 = [team.replace('WAS Wizards', 'WAS') for team in new12]
new14 = [team.replace('POR Trail Blazers', 'POR') for team in new13]
new15 = [team.replace('NY Knicks', 'NYK') for team in new14]
new16 = [team.replace('BKN Nets', 'BRK') for team in new15]
new17 = [team.replace('LA Clippers', 'LAC') for team in new16]
new18 = [team.replace('DET Pistons', 'DET') for team in new17]
new19 = [team.replace('DAL Mavericks', 'DAL') for team in new18]
new20 = [team.replace('BOS Celtics', 'BOS') for team in new19]
new21 = [team.replace('PHI 76ers', 'PHI') for team in new20]
new22 = [team.replace('ORL Magic', 'ORL') for team in new21]
new23 = [team.replace('MEM Grizzlies', 'MEM') for team in new22]
new24 = [team.replace('OKC Thunder', 'OKC') for team in new23]
new25 = [team.replace('HOU Rockets', 'HOU') for team in new24]
new26 = [team.replace('NO Pelicans', 'NOP') for team in new25]
new27 = [team.replace('LA Lakers', 'LAL') for team in new26]
new28 = [team.replace('ATL Hawks', 'ATL') for team in new27]
new29 = [team.replace('PHO Suns', 'PHO') for team in new28]
new30 = [team.replace('CHA Hornets', 'CHA') for team in new29]
```


```python
teams_today = new30
```

### Clean Lines Element


```python
line_a = lines[0]
line_b = lines[1]
line_list_a = line_a.split(", ")
line_list_b = line_b.split(", ")
```


```python
line_list2_a = []
line_list2_b = []

for t in line_list_a:
    new = t[:-6]
    line_list2_a.append(new)

for t in line_list_b:
    new = t[:-6]
    line_list2_b.append(new)
```


```python
line_list3_a = []
line_list3_b = []

for x in line_list2_a:
    new = x[63:]
    line_list3_a.append(new)

for x in line_list2_b:
    new = x[63:]
    line_list3_b.append(new)
```


```python
line_list4_a = [q.replace('>', '') for q in line_list3_a]
line_list5_a = [q.replace('/', '') for q in line_list4_a]
dk_lines_a = [q.replace('<', '') for q in line_list5_a]

line_list4_b = [q.replace('>', '') for q in line_list3_b]
line_list5_b = [q.replace('/', '') for q in line_list4_b]
dk_lines_b = [q.replace('<', '') for q in line_list5_b]
```


```python
dk_lines_a.extend(dk_lines_b)
```


```python
lines_today = dk_lines_a
```

### Check that there is the same number of lines as their is games


```python
correct_num_games = len(lines_today)
```


```python
correct_teams_today = []

if len(teams_today) > len(lines_today):
    correct_teams_today = teams_today[:correct_num_games]
else:
    correct_teams_today = teams_today
```

### Combine the Two Elements


```python
team1 = correct_teams_today[1::2]
team2 = correct_teams_today[::2]
```


```python
team1_line = lines_today[1::2]
team2_line = lines_today[::2]
```


```python
combined_list = pd.DataFrame(
    {'Team 1' : team1, 
     'Team 2' : team2, 
     'Team 1 - Moneyline' : team1_line, 
     'Team 2 - Moneyline' : team2_line
})
```

### Today's NBA Games with Betting Lines from Draft Kings:


```python
combined_list
```

### Create email with data


```python
content = {
    "header": emailbuilder.image(src="https://s3-symbol-logo.tradingview.com/draftkings--600.png",
                                 link="https://sportsbook.draftkings.com",
                                 align="center",
                                 width="20%"),
    "txt_0": emailbuilder.text("Hi Ballers 🏀,<br><br>"
                               f"Here below the Moneylines* for NBA games as of {TODAY} :<br>"),
    "table": emailbuilder.table(combined_list,
                                border=True,
                                header=True,
                                col_size={0: "15%", 1: "15%", 2: "35%", 3: "35%"},
                                col_align={0: "center", 1: "center", 2: "center", 3: "center"}),
    "txt_1": emailbuilder.text("<i>*A moneyline bet in sports refers to a wager on the winning team.</i>"),
    "button_1": emailbuilder.button(link="https://sportsbook.draftkings.com",
                                    text="Bet on Draft Kings",
                                    color="white",
                                    background_color="#53d337"),
    "txt_4": ("Interested to improve this template, please send contact <a href='https://www.linkedin.com/in/ja-williams-529517187'>JA Williams<a/> or send a message to Naas Core Team at hello@naas.ai.<br><br>"),
    "heading_5": emailbuilder.text("Happy betting 💸!"),
    "footer": emailbuilder.footer_company(naas=True)
}
email_content = emailbuilder.generate(display='iframe', **content)
```

## Output

### Send email


```python
naas.notification.send(EMAIL_TO,
                       EMAIL_SUBJECT,
                       email_content)
```
