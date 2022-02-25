# Robinaut Controller v0.1 Beta
The Freqtrade Telegram Helper Bot

![alt-text](https://i.imgur.com/s88cypn.png)

![alt-text-1](https://i.imgur.com/iEmtc83.png) ![alt-text-2](https://i.imgur.com/8Rs99NX.png)

# Features
The goal of this bot is to add a lot more features to control and manage multiple Freqtrade instances as well as view live data.  (Stuff that Freqtrade's Telegram bot currently cannot do and may never do.)  I like to use this bot personally to extend my ability to control multiple bots running on a server to get a general picture of all the pairs without having to go into FreqUI or to pull up FreqUI's dynamic ngrok HTTPS web urls.  Use NGROK at your own risk, however I personally think its fine for me.  I believe that wrapping the url in NGROK's HTTPS tunnels is safer when viewing data from an external server, but I was told before by the FreqUI developer that this method of tunneling FreqUI is not recommended (but neither is accessing FreqUI externally in general).  It is useful for me personally, and I will configure it later so that you can refresh NGROK's random tunnels URL or even set it to automatically reset each day. 

Besides the commands below, this bot can also plot live when orders are placed on your Freqtrade bots.
```
/help_control - default help page
/plot - plot pairs for specified bot
/indicators - obtain indicator values for desired pairs (still beta, needs work)
/public_url - obtain local NGROK tunnels for bots
/sync_configs - (something i'm playing with to sync data.  may not be useful for you right now.)
```
# How to use
1.  Download and install 'memcached'.  Then use the command `memcached -d` to launch the memcached daemon.
1.  Place the files in the directory above the folder containing `user_data`.  Name the folder according to the bot name.
```
/ft_bots/ft_bot_1/user_data
/ft_bots/ft_bot_2/user_data
...
/ft_bots/rbn_controller.py
/ft_bots/rbn_controller.json
/ft_bots/rbn_controller_launcher.zsh
/ft_bots/ngrok_launcher.zsh
```
2. Add in your configurations into the .json.
3. When working with Telegram, be sure to create a group for your Freqtrade bot.  This group will contain you, your Freqtrade bot, and the controller bot.  Use the same group chat id for both bots.  Make sure that the bots are set to "admin" privileges, that you are set to "anonymous", and that history is set to be logged.
4.  Use the `rbn_controller_launcher.zsh` to run the controller on a screen named`rbn_controller`.
5.  You will need to also modify your strategy file.
```
# add this to import
from pymemcache.client import base

# Memcache settings, they go at the top of your strategy class.
use_memcache = True
num_cached_candles = 50

# add this to end of 'populate_indicators'
try:
    pair = pair.replace('/', '_')
    if self.use_memcache:
        client = base.Client(('localhost', 11211))
        datetime_entries = [str(entry) for entry in dataframe[f"datetime"].iloc[-self.num_cached_candles:]]
        client.set(ft_bot+'_'+pair+'_date', str(datetime_entries))
        client.set(ft_bot+'_'+pair+'_open', str(list(dataframe[f"open"].iloc[-self.num_cached_candles:])))
        client.set(ft_bot+'_'+pair+'_high', str(list(dataframe[f"high"].iloc[-self.num_cached_candles:])))
        client.set(ft_bot+'_'+pair+'_low',  str(list(dataframe[f"low"].iloc[-self.num_cached_candles:])))
        client.set(ft_bot+'_'+pair+'_close', str(list(dataframe[f"close"].iloc[-self.num_cached_candles:])))
        client.set(ft_bot+'_'+pair+'_volume', str(list(dataframe[f"volume"].iloc[-self.num_cached_candles:])))
        client.set(ft_bot+'_'+pair+'_indicator_1', str(list(dataframe[f"indicator_1"].iloc[-self.num_cached_candles:])))
        client.set(ft_bot+'_'+pair+'_indicator_2', str(list(dataframe[f"indicator_2"].iloc[-self.num_cached_candles:])))
except Exception as e:
    print(e)
    print('Memcache has failed.')
```
NOTE: If you are running hyperopt, be sure that use_memcache = False.

6.  To use the NGROK features, add these lines to your `~/.ngrok2/ngrok.yml` file after applying your authtoken.
```
authtoken: XXXXXXXXXXXXXXXX
tunnels:
  ft_bot_1:
    addr: <FREQ_UI_PORT>
    proto: http
    bind_tls: true
  ft_bot_2:
    addr: <FREQ_UI_PORT>
    proto: http
    bind_tls: true
  ...
```
After, you can use `ngrok_launcher.zsh` to run ngrok on a screen named `ngrok`.
