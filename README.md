# algorithmic_trading_bot.py

import yfinance as yf
from datetime import datetime
import time

# Step 1: Install yfinance
# !pip install yfinance

"""
Questions & Answers:

What is yfinance?
- yfinance is a Python library that allows users to download historical market data from Yahoo Finance.

Why do we need to install yfinance?
- We need to install yfinance to fetch live market data for our trading bot.

How do we install yfinance?
- By using the command !pip install yfinance in a Jupyter notebook or pip install yfinance in a command line.
"""

# Step 2: Base Class for Trading Strategies
class TradingStrategy:
    """
    Base class for all trading strategies.
    
    Questions & Answers:
    
    What is a class?
    - A class is a blueprint for creating objects. It defines a set of attributes and methods.
    
    What is the purpose of TradingStrategy?
    - TradingStrategy is a base class for defining different trading strategies.
    
    Why do we use a constructor (__init__ method)?
    - The constructor initializes the class with a name attribute when an instance is created.
    
    What does generate_signal do?
    - generate_signal is a placeholder method intended to be overridden by subclasses.
    
    Why make __name private?
    - To enforce encapsulation and ensure the attribute cannot be modified directly.
    
    What is the @property decorator?
    - The @property decorator allows a method to be accessed like an attribute.
    """
    
    def __init__(self, name):
        self.__name = name

    def generate_signal(self, price_data):
        print("This method should be overridden by subclasses")
        return "hold"
    
    @property
    def name(self):
        return self.__name

# Step 3: Simple Moving Average Strategy
class SMAStrategy(TradingStrategy):
    """
    Simple Moving Average trading strategy.
    
    Questions & Answers:
    
    What is inheritance in OOP?
    - Inheritance allows a new class to inherit attributes and methods from an existing class.
    
    How does SMAStrategy use inheritance?
    - SMAStrategy inherits from TradingStrategy, getting all its attributes and methods.
    
    What does super().__init__() do?
    - It calls the constructor of the base class (TradingStrategy).
    
    What is method overriding?
    - Method overriding allows a subclass to provide a specific implementation of a method.
    """
    
    def __init__(self, short_window, long_window):
        super().__init__("Simple Moving Average Strategy")
        self.__short_window = short_window
        self.__long_window = long_window

    def generate_signal(self, price_data):
        if len(price_data) < self.__long_window:
            return "hold"
        
        short_avg = sum(price_data[-self.__short_window:]) / self.__short_window
        long_avg = sum(price_data[-self.__long_window:]) / self.__long_window
        
        if short_avg > long_avg:
            return "buy"
        elif short_avg < long_avg:
            return "sell"
        else:
            return "hold"

    @property
    def short_window(self):
        return self.__short_window

    @property
    def long_window(self):
        return self.__long_window

# Step 4: Trade Management Class
class Trade:
    """
    Manages individual trades.
    
    Questions & Answers:
    
    What is the purpose of the Trade class?
    - The Trade class manages individual trades with strategy name, signal, amount, and timestamp.
    
    Why do we need a timestamp?
    - The timestamp records the exact time when the trade is executed for tracking and analysis.
    
    What does the execute method do?
    - The execute method prints the details of the trade being executed.
    """
    
    def __init__(self, strategy_name, signal, amount):
        self.__strategy_name = strategy_name
        self.__signal = signal
        self.__amount = amount
        self.__timestamp = datetime.now()

    def execute(self):
        print(f"Executing {self.__signal} for {self.__amount} units using {self.__strategy_name} at {self.__timestamp}")

    @property
    def strategy_name(self):
        return self.__strategy_name

    @property
    def signal(self):
        return self.__signal

    @property
    def amount(self):
        return self.__amount

    @property
    def timestamp(self):
        return self.__timestamp

# Step 5: Mock Trading API
class MockTradingAPI:
    """
    Simulates a trading API for testing.
    
    Questions & Answers:
    
    What is the purpose of MockTradingAPI?
    - MockTradingAPI simulates a trading API to place orders and manage balance without real money.
    
    Why do we need a mock API?
    - It allows testing trading bot logic without risking real money or needing live trading access.
    
    What does place_order method do?
    - It updates the balance based on trade signal and prints order details.
    """
    
    def __init__(self, balance):
        self.__balance = balance

    def place_order(self, trade, price):
        if trade.signal == "buy" and self.__balance >= trade.amount * price:
            self.__balance -= trade.amount * price
            print(f"Placed buy order for {trade.amount} units at {price}. Remaining balance: {self.__balance}")
        elif trade.signal == "sell":
            self.__balance += trade.amount * price
            print(f"Placed sell order for {trade.amount} units at {price}. Remaining balance: {self.__balance}")
        else:
            print("Insufficient balance or invalid signal.")

    def get_balance(self):
        return self.__balance

# Step 6: Main Trading System
class TradingSystem:
    """
    Main system integrating all components.
    
    Questions & Answers:
    
    What is the purpose of TradingSystem?
    - TradingSystem integrates the trading API, strategy, and symbol to manage the overall trading process.
    
    What does fetch_price_data do?
    - It fetches latest market data using yfinance and updates the price data list.
    
    What does the run method do?
    - It fetches price data, generates signals, and executes trades.
    
    How does this demonstrate composition?
    - TradingSystem is composed of instances of other classes (MockTradingAPI, SMAStrategy).
    """
    
    def __init__(self, api, strategy, symbol):
        self.__api = api
        self.__strategy = strategy
        self.__symbol = symbol
        self.__price_data = []

    def fetch_price_data(self):
        data = yf.download(tickers=self.__symbol, period='1d', interval='1m')
        if not data.empty:
            price = data['Close'].iloc[-1]
            self.__price_data.append(price)
            if len(self.__price_data) > self.__strategy.long_window:
                self.__price_data.pop(0)
            print(f"Fetched new price data: {price}")
        else:
            print("No data fetched")

    def run(self):
        self.fetch_price_data()
        signal = self.__strategy.generate_signal(self.__price_data)
        print(f"Generated signal: {signal}")
        if signal in ["buy", "sell"]:
            trade = Trade(self.__strategy.name, signal, 1)
            trade.execute()
            self.__api.place_order(trade, self.__price_data[-1])

    @property
    def api(self):
        return self.__api

    @property
    def strategy(self):
        return self.__strategy

    @property
    def symbol(self):
        return self.__symbol

    @property
    def price_data(self):
        return self.__price_data

# Step 7: Main Execution
def main():
    """
    Main execution function.
    
    Questions & Answers:
    
    What does if __name__ == "__main__": do?
    - Ensures the code runs only if the script is executed directly, not when imported.
    
    Why use time.sleep(60)?
    - To wait 60 seconds before fetching new data, simulating real-time trading.
    """
    
    symbol = 'AAPL'

    # Create instances
    api = MockTradingAPI(balance=10000)
    strategy = SMAStrategy(short_window=3, long_window=5)
    system = TradingSystem(api, strategy, symbol)

    # Run trading bot
    print("Starting Algorithmic Trading Bot...")
    print("=" * 50)
    
    for i in range(10):
        print(f"\n--- Iteration {i+1} ---")
        system.run()
        print(f"Current balance: ${api.get_balance():.2f}")
        
        if i < 9:  # Don't sleep after the last iteration
            print("Waiting 60 seconds for next update...")
            time.sleep(60)
    
    print("\n" + "=" * 50)
    print("Trading session completed!")

# Property Decorator Examples
class PropertyExamples:
    """
    Examples demonstrating @property decorator usage.
    
    Without @property:
    class Circle:
        def __init__(self, radius):
            self._radius = radius
    
        def get_radius(self):
            return self._radius
    
        def set_radius(self, radius):
            if radius >= 0:
                self._radius = radius
            else:
                raise ValueError("Radius cannot be negative")
    
    With @property:
    class Circle:
        def __init__(self, radius):
            self._radius = radius
    
        @property
        def radius(self):
            return self._radius
    
        @radius.setter
        def radius(self, radius):
            if radius >= 0:
                self._radius = radius
            else:
                raise ValueError("Radius cannot be negative")
    """
    
    def __init__(self, value):
        self._value = value
    
    @property
    def value(self):
        return self._value
    
    @value.setter
    def value(self, new_value):
        if new_value >= 0:
            self._value = new_value
        else:
            raise ValueError("Value cannot be negative")

# OOP Concepts Summary
"""
SUMMARY OF PYTHON AND OOP CONCEPTS:

1. Classes and Objects: Define reusable blueprints (classes) and create instances (objects)
2. Inheritance: SMAStrategy inherits from TradingStrategy
3. Encapsulation: Data hiding using private attributes and methods
4. Method Overriding: Subclass provides specific implementation of base class method
5. Polymorphism: Same method name works differently for different classes
6. Composition: TradingSystem uses MockTradingAPI and SMAStrategy objects
7. Private Attributes: Using double underscores for data hiding
8. Properties: Using @property decorator for controlled attribute access
"""

# Trading Bot Flow Explanation
"""
TRADING BOT FLOW:

1. INITIALIZATION:
   - Set up stock symbol, mock API, trading strategy, and trading system

2. DATA FETCHING:
   - Fetch latest price data from Yahoo Finance
   - Update price history list

3. SIGNAL GENERATION:
   - Strategy analyzes price data
   - Returns "buy", "sell", or "hold" signal

4. TRADE EXECUTION:
   - If signal is buy/sell, create Trade object
   - Execute trade through mock API
   - Update account balance

5. LOOP CONTINUES:
   - Wait specified interval
   - Repeat process for next trading cycle
"""

if __name__ == "__main__":
    main()
