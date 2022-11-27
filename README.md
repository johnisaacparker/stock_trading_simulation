# stock_trading_simulation
#program runs mean reversion, simple moving average, and
#bollinger band trading strategies
#on 10 different stocks.
#prints out all buys and sells on all stocks, as well as
#stores final profit and final returns in a json file.
import json
import requests
import time
import os


def create_data(ticker):

    url = 'http://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol='+ticker+'&outputsize=full&apikey=NG9C9EPVYBMQT0C8'
    
    req = requests.get(url) #loads alphavantage into program
    
    time.sleep(12) #delays the program's running so alphavantage
                    #doesn't crash
    
    req_dict = json.loads(req.text) #turns alphavantage data 
                        #into a json string
    
    print(req_dict.keys())
    
    key1 = "Time Series (Daily)" # dictionary with all prices by date
    
    key2 = '4. close'
    
    csv_file = open("/home/ubuntu/environment/final_project/data/" + ticker + ".csv", "w")
    
    write_lines = [] #puts data in json string into a list
    
    for date in req_dict[key1]:
        
        print(date + "," + req_dict[key1][date][key2]) #print key, value
        
        write_lines.append(date + "," + req_dict[key1][date][key2]+"\n")
        
    write_lines = write_lines[::-1]
    
    csv_file.writelines(write_lines) #puts the list into the csv file
    
    csv_file.close()
    
def append_data(ticker):
    
    url = 'http://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol='+ ticker +'&outputsize=full&apikey=NG9C9EPVYBMQT0C8'
    
    req = requests.get(url)
    
    time.sleep(12)
    
    req_dict = json.loads(req.text)
    
    print(req_dict.keys())
    
    key1 = "Time Series (Daily)" # dictionary with all prices by date
    
    key2 = '4. close'
    
    # checking to see if file exists, or if its empty
    # size = os.path.size(ticker + ".csv")
    # if size == 0: # file exists, but is empty
    #     print("file exists, but is empty")
    # if not os.path.exists(ticker + ".csv"): # file does not exist
    #     print("file does not exist")
        
    # opening file and getting the last date
    # this will be used to know what dates, to add data to the file
    csv_file = open("/home/ubuntu/environment/final_project/data/" + ticker + ".csv", "r")
    #reads csv file 
    lines = csv_file.readlines()
    
    last_date = lines[-1].split(",")[0]
    
    new_lines = []
    
    for date in req_dict[key1]:
       
        if date == last_date:
            
            break    #if today is the last date, nothing happen
                    #otherwise, it adds the new data from the alphavantage
                    #json api
        
        print(date + "," + req_dict[key1][date][key2]) #print key, value
        
        new_lines.append(date + "," + req_dict[key1][date][key2]+"\n")
        
    new_lines = new_lines[::-1]
    
    csv_file = open("/home/ubuntu/environment/final_project/data/" + ticker + ".csv", "a") # opening the file to append data
    
    csv_file.writelines(new_lines) # appending new data
    
    csv_file.close()
        
def meanReversionStrategy(prices):
    #makes mean reversion strategy a reusable function
    buy = 0
    
    first_buy = 0
    
    total_profit = 0
    
    to_do = 0
    
    for i in range(len(prices)): #for every price in price list
    
        if i > 5: #if we have enough prices to calculate 5-day moving average
            
            current_price = prices[i]#the next price in the list
    
            avg_price = (prices[i-1] + prices[i-2] + prices[i-3] + prices[i-4] + prices[i-5]) / 5#average price for the 5 previous days
            
            
            if current_price < avg_price * .98 and buy == 0: #buy
                
                buy = current_price #update buy variable
                
                if first_buy == 0: #identifies first buy for final profit
                                    #percentage
                    
                    first_buy = buy
                    
                to_do = 'buy'
            
                print("buy at: ", current_price)
                
            elif current_price > avg_price * 1.02 and buy != 0: #sell
            
                print("sell at: ", current_price)
            
                print("trade profit: ", round((current_price - buy), 2))#calculate profit of this individual trade
                
                total_profit += current_price - buy #keep a running total of all profit
                
                buy = 0 #reset buy variable so we can buy again
                
                to_do = 'sell'
            
            else: #do nothing
            
                pass
                
                to_do = 'pass on'
    
    final_profit_percentage = (total_profit  / first_buy) * 100
    
    print("total profit: ", total_profit)
      
    print('you should', to_do, 'this today.')
    
    return total_profit, final_profit_percentage
    
def simpleMovingAverageStrategy(prices):
    #puts simple moving average trading strategy into 
    #a reusable function
    buy = 0
    
    first_buy = 0
    
    total_profit = 0
    
    for i in range(len(prices)): #for every price in price list
    
        if i > 5: #if we have enough prices to calculate 5-day moving average
            
            current_price = prices[i]#the next price in the list
    
            avg_price = (prices[i-1] + prices[i-2] + prices[i-3] + prices[i-4]) / 4#average price for the 5 previous days
            
            
            if current_price > avg_price and buy == 0: #buy
                
                buy = current_price #update buy variable
                
                if first_buy == 0: #identifies first buy for final profit
                                    #percentage
                    first_buy = buy
            
                print("buy at: ", current_price)
            
            elif current_price < avg_price and buy != 0: #sell
            
                print("sell at: ", current_price)
            
                print("trade profit: ", current_price - buy)#calculate profit of this individual trade
                
                total_profit += current_price - buy #keep a running total of all profit
                
                buy = 0 #reset buy variable so we can buy again
                
            else: #do nothing
            
                pass
    
    final_profit_percentage = (total_profit  / first_buy) * 100
    
    print("total profit: ", total_profit)
    
    return total_profit, final_profit_percentage
    
def BollingerBandStrategy(prices):
    #puts bollinger band trading strategy into 
    #a reusable function
    buy = 0
    
    first_buy = 0
    
    total_profit = 0
    
    for i in range(len(prices)): #for every price in price list
    
        if i > 5: #if we have enough prices to calculate 5-day moving average
            
            current_price = prices[i]#the next price in the list
    
            avg_price = (prices[i-1] + prices[i-2] + prices[i-3] + prices[i-4]) / 4#average price for the 5 previous days
            
            
            if current_price > avg_price * 1.05 and buy == 0: #buy
                
                buy = current_price #update buy variable
                
                if first_buy == 0:  #identifies first buy for final profit
                                    #percentage
                    first_buy = buy
            
                print("buy at: ", current_price)
            
            elif current_price < avg_price * .95 and buy != 0: #sell
            
                print("sell at: ", current_price)
            
                print("trade profit: ", round((current_price - buy), 2))#calculate profit of this individual trade
                
                total_profit += current_price - buy #keep a running total of all profit
                
                buy = 0 #reset buy variable so we can buy again
            
            else:
                
                pass
    
    final_profit_percentage = (total_profit  / first_buy) * 100 
    
    print("total profit: ", total_profit)
    
    return total_profit, final_profit_percentage

def save_results(results):   #saves results in json file
        
    json.dump(results, open('results.json', 'w'), indent=4)
    
#main line code, runs all strategies on csv files

results = {} #creates dictionary that stores all results

tickers = ['aapl', 'adbe']#, 'goog', 'tsla', 'amzn', 'dis', 'msft', 'nflx', 'wmt', 'gm']
#list of all stocks that will be used in this program
max_total_profit = 0

most_profitable_ticker: ''

most_profitable_strategy: ''

for ticker in tickers: #run simple moving average strategy
    
    #create_data(ticker)
    
    append_data(ticker) #runs the append_data function to make
                        #sure we have the latest prices
    
    lines = open('/home/ubuntu/environment/final_project/data/' + ticker + '.csv').readlines()
    
    prices = []
        
    for line in lines:
        
        line = float(line.split(',')[1])
            
        prices.append(line) #populates prices
                                    #list with rounded prices
                                    
    
    #runs simple moving average strategy
    profit, returns = simpleMovingAverageStrategy(prices)
    
    if profit > max_total_profit:
        
        max_total_profit = profit
        
        most_profitable_ticker = ticker
        
        most_profitable_strategy = 'simple moving average'
        
        
    print('-----', ticker, "simple moving average strategy", "-----")
    
    print('profit: ', profit)
    
    print('returns: ', returns)
    
    #store results into our results dictionary
    
    results[ticker + '_mas_profit'] = profit
   
    results[ticker + '_mas_returns'] = returns
  
    
    #run mean reversion strategy
    profit, returns = meanReversionStrategy(prices)
    
    if profit > max_total_profit:
        
        max_total_profit = profit
        
        most_profitable_ticker = ticker
        
        most_profitable_strategy = 'mean reversion'
        
    print('-----', ticker, "mean reversion strategy", "-----")
    
    print('profit: ', profit)
    
    print('returns: ', returns)
    
    #store results into our results dictionary
    
    results[ticker + '_mr_profit'] = profit 

    results[ticker + '_mr_returns'] = returns
    
    #run bollinger band strategy
    profit, returns = BollingerBandStrategy(prices)
    
    if profit > max_total_profit:
        
        max_total_profit = profit
        
        most_profitable_ticker = ticker
        
        most_profitable_strategy = 'bollinger band'
    
    print('-----', ticker, "bollinger band strategy", "-----")
    
    print('profit: ', profit)
    
    print('returns: ', returns)
    
    #store results into our results dictionary
    
    results[ticker + '_bbs_profit'] = profit 

    results[ticker + '_bbs_returns'] = returns

results['most profitable ticker'] = most_profitable_ticker #puts all the info on the most profitable

results['most profitable strategy'] = most_profitable_strategy #ticker/strategy into the results dictionary

results['most profit'] = max_total_profit                   #which will go into json file

save_results(results)

print(results)
