<a href="https://app.naas.ai/user-redirect/naas/downloader?url=https://raw.githubusercontent.com/jupyter-naas/awesome-notebooks/master/Telegram/Telegram_Create_crypto_sentiment_bot.ipynb" target="_parent"><img src="https://naasai-public.s3.eu-west-3.amazonaws.com/open_in_naas.svg"/></a><br><br><a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=&template=template-request.md&title=Tool+-+Action+of+the+notebook+">💡 Template request</a> | <a href="https://github.com/jupyter-naas/awesome-notebooks/issues/new?assignees=&labels=bug&template=bug_report.md&title=Telegram+-+Create+crypto+sentiment+bot:+Error+short+description">🚨 Bug report</a>

**Tags:** #telegram #sentiment #bot #naas_drivers #ai #investors

**Author:** [Yaswanthkumar GOTHIREDDY](https://www.linkedin.com/in/yaswanthkumargothireddy/)

This notebook is a demo of how you can use naas drivers and Telegram to create a Crypto sentiment bot. <br>
Please note that if you stop the notebook, the bot will not work anymore.

## Input

### Install packages


```python
!pip install python-telegram-bot --user
```

### Import libraries


```python
import logging
from telegram.ext import *
import numpy as np
from naas_drivers import newsapi
from naas_drivers import sentiment
from datetime import datetime, timedelta
```

### Variables

[Create a Telegram bot and get your API KEY](https://core.telegram.org/bots#6-botfather)


```python
TELEGRAM_API_KEY = "***********"
```

[Get Newsapi API KEY](https://newsapi.org/)


```python
NEWSAPI_API_KEY = "***********"
```


```python
# Set up the logging
logging.basicConfig(format='%(asctime)s - %(name)s - %(levelname)s - %(message)s', level=logging.INFO)
logging.info('Starting Bot...')
```

### Get last 24 hours articles from NewsAPI on a CoinMarketCap top coins


```python
#parameters for newsapi
FROM_DATE = (datetime.now()-timedelta(days=1)).replace(microsecond=0).isoformat()
TO_DATE = datetime.now().replace(microsecond=0).isoformat()

#top marketcap coins according to coinmarketcap
coins= ["bitcoin","ethereum", "ripple","dogecoin","cardano", 
        "polygon","binance","polkadot","uniswap","litecoin",
        "chainlink","solana","stellar"]

#connect to Newsapi Naas drivers
newsapi.connect(NEWSAPI_API_KEY)
newsapi.connect()
```

## Model

### User need to ask for coins ticker and get responses


```python
#user_response
def get_response(user_message):
    
    usr_msg_text = user_message.lower()
    
    if usr_msg_text in coins: #check if returned message has the top coins 
        data = newsapi.get(q=usr_msg_text,
                                       from_param=FROM_DATE,
                                       to = TO_DATE,
                                       language ="en")
        if len(data)==0:#extra check to returned data
            sentiment = "No news for this ticker in last 24 hours, try another"
            return sentiment
        else:
            sentiment_df =  sentiment.get(data, column_name="title")#sentiment calculation
            sentiment = sentiment_df["Score"].mean()
    else:
        sentiment = 'There is no data for this ticker\n use \help command'
    
    return sentiment
```

### Here are the commands to communicate with Telegram


```python
#commands
def start_command(update, context):
    update.message.reply_text('''Hello there! I\'m a crypto sentiment bot.
    Send me the coin name, I\'ll send you sentiment!''')


def help_command(update, context):
    update.message.reply_text('''check the coin names from the list below!
    bitcoin\n etherium\n ripple\n dogecoin\n cardano\n polygon
    binance\n polkadot\n uniswap\n litecoin\n chainlink\n solana\n stellar''')

#message handler
def handle_message(update, context):
    text = str(update.message.text).lower()
    logging.info(f'User ({update.message.chat.id}) says: {text}')

    # Bot response
    sentiment_naas = get_response(text)
    update.message.reply_text(sentiment_naas)

#error logs
def error(update, context):
    # Logs errors
    logging.error(f'Update {update} caused error {context.error}')
```

## Output

### Start the bot


```python
# Run the programme

# create an object "bot"
updater = Updater(TELEGRAM_API_KEY, use_context=True)
dp = updater.dispatcher

#  bot's command handlers
dp.add_handler(CommandHandler('start', start_command))
dp.add_handler(CommandHandler('help', help_command))

# bot's text handlers 
dp.add_handler(MessageHandler(Filters.text, handle_message))

# bot's error handler
dp.add_error_handler(error)

    
# Run the bot
updater.start_polling(1.0)
updater.idle()
```

Info: the cell above will stay alive until you press stop, the bot will not respond anymore once the cell is stoped.
