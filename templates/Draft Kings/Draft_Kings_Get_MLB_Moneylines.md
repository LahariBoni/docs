<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Draft%20Kings/Draft_Kings_Get_MLB_Moneylines.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Draft+Kings+-+Get+MLB+Moneylines:+Error+short+description">🚨 Bug report</a>

**Tags:** #draftkings #mlb #betting #python #analytics #automation #sports #sports_betting #opendata #notification #email

**Author:** [JA Williams](https://www.linkedin.com/in/ja-williams-529517187/)

This notebooks gathers the betting lines for MLB games each day. It scrapes the Moneylines for each team from the Draft Kings website and emails you the lines.

Data source: https://sportsbook.draftkings.com/leagues/baseball/88670847

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
URL = 'https://sportsbook.draftkings.com/leagues/baseball/88670847'
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
# Schedule the Notebook to run at 10 AM each day of the MLB season
naas.scheduler.add(cron='0 10 * 10,11,12,1,2,3,4,5 *')

# To delete your scheduler, uncomment the line below and execute the cell
# naas.scheduler.delete()
```


```python
# Email
EMAIL_TO = "<YOUR EMAIL>"
TODAY = datetime.now(pytz.timezone(TIME_ZONE)).strftime("%Y-%m-%d")
EMAIL_SUBJECT = f"⚾ Draft Kings : Your MLB game of the day {TODAY}"
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
game_list_1 = []

for t in teams:
    new = t[36:]
    game_list_1.append(new)

```


```python
game_list_2 = []

for g in game_list_1:
    new = g.split(",")
    game_list_2.extend(new)
```


```python
game_list_3 = [x.replace(' <div class="event-cell__name-text">', '') for x in game_list_2]
```


```python
game_list_4 = []

for t in game_list_3:
    new = t[:-6]
    game_list_4.append(new)
```


```python
game_list_5 = [x.replace('<', '') for x in game_list_4]
```


```python
outcomes_today = game_list_5
```


```python
# Change Team Names to Match with 3-letter Code

new1 = [team.replace('ARI Diamondbacks', 'ARI') for team in game_list_5]
new2 = [team.replace('ATL Braves', 'ATL') for team in new1]
new3 = [team.replace('BAL Orioles', 'BAL') for team in new2]
new4 = [team.replace('BOS Red Sox', 'BOS') for team in new3]
new5 = [team.replace('CHI Cubs', 'CHC') for team in new4]
new6 = [team.replace('CHI White Sox', 'CHW') for team in new5]
new7 = [team.replace('CIN Reds', 'CIN') for team in new6]
new8 = [team.replace('CLE Guardians', 'CLE') for team in new7]
new9 = [team.replace('COL Rockies', 'COL') for team in new8]
new10 = [team.replace('DET Tigers', 'DET') for team in new9]
new11 = [team.replace('MIA Marlins', 'FLA') for team in new10]
new12 = [team.replace('HOU Astros', 'HOU') for team in new11]
new13 = [team.replace('KC Royals', 'KCR') for team in new12]
new14 = [team.replace('LA Angels', 'ANA') for team in new13]
new15 = [team.replace('LA Dodgers', 'LAD') for team in new14]
new16 = [team.replace('MIL Brewers', 'MIL') for team in new15]
new17 = [team.replace('MIN Twins', 'MIN') for team in new16]
new18 = [team.replace('NY Mets', 'NYM') for team in new17]
new19 = [team.replace('NY Yankees', 'NYY') for team in new18]
new20 = [team.replace('OAK Athletics', 'OAK') for team in new19]
new21 = [team.replace('PHI Phillies', 'PHI') for team in new20]
new22 = [team.replace('PIT Pirates', 'PIT') for team in new21]
new23 = [team.replace('SD Padres', 'SDP') for team in new22]
new24 = [team.replace('SF Giants', 'SFG') for team in new23]
new25 = [team.replace('SEA Mariners', 'SEA') for team in new24]
new26 = [team.replace('STL Cardinals', 'STL') for team in new25]
new27 = [team.replace('TB Rays', 'TBD') for team in new26]
new28 = [team.replace('TEX Rangers', 'TEX') for team in new27]
new29 = [team.replace('TOR Blue Jays', 'TOR') for team in new28]
new30 = [team.replace('WAS Nationals', 'WAN') for team in new29]
```


```python
teams_today = new30
```

### Clean Lines Element


```python
lines_list_1 = []

for l in lines:
    new = l.split(",")
    lines_list_1.extend(new)
```


```python
lines_list_2 = []

for l in lines_list_1:
    new = l[64:]
    lines_list_2.append(new)
```


```python
lines_list_3 = []

for l in lines_list_2:
    new = l[:-7]
    lines_list_3.append(new)
```


```python
lines_list_4 = [x.replace('<', '') for x in lines_list_3]
```


```python
lines_today = lines_list_4
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

### Combine the Elements


```python
team1 = teams_today[::2]
team2 = teams_today[1::2]

team1_line = lines_today[::2]
team2_line = lines_today[1::2]

combined_list = pd.DataFrame(
    {'team1' : team2, 
    'team2' : team1, 
    'team1_line' : team2_line, 
    'team2_line' : team1_line,
})

combined_list.drop_duplicates(subset = ['team1', 'team2'], inplace = True)
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
    "txt_0": emailbuilder.text("Hi Slugger ⚾,<br><br>"
                               f"Here below the Moneylines* for MLB games as of {TODAY} :<br>"),
    "table": emailbuilder.table(combined_list,
                                border=True,
                                header=True,
                                col_size={0: "15%", 1: "15%", 2: "35%", 3: "35%"},
                                col_align={0: "center", 1: "center", 2: "center", 3: "center"}),
    "txt_1": emailbuilder.text("<i>*A moneyline bet in sports refers to a wager on the winning team. Plus odds (+) mean that amount of money would be made on a $100 bet. (eg. +150 means if you bet $100, you would win $150 of profit, for a total return of $250). Minus odds (-) mean you would have to bet that amount of money to make $100 of profit. (eg. -150 means you would need to bet $150 to win $100 of profit, for a total return of $250).</i>"),
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
