+++ 
draft = false
date = 2022-12-09T19:30:48-06:00
title = "Calculating Pi Using Random Numbers"
description = "Using the coprime probabilities of random number pairs to calculate the value of Pi"
slug = "calculating-pi-with-random-numbers"
authors = ["Matthew Sunner"]
tags = ["Python", "Maths", "Data Visualization"]
categories = []
externalLink = ""
series = []
+++

Calculating Pi is possible by using the probability of co-prime values in a large set of random numbers. This was inspired by Matt Parker’s YouTube video, [Generating Pi From 1,000 Random Numbers](https://youtu.be/RZBhSi_PwHU), posted in 2017. This video is excellent at explaining the maths logic behind the formula, so I won't cover it here. The repository discussed here uses the same methodology as Matt Parker used, but on a much larger scale. Matt Parker used 120 sided dice to calculate 1,000 random co-prime values. My approach uses the equivalent of 100,000,000,000 sided die and calculates 100,000 co-prime pair values. In addition to this, I ran this full simulation 50,000 times, as opposed to once in Matt’s situation (to be fair, he did physically roll the dice 1,000 times, I just pressed ‘Enter’ on my laptop). Below is a walk through of the script and resulting data visualization. The repo for this project can be found [here](https://github.com/mattsunner/Ridiculous_Data_Viz/tree/main/PiCalc_by_Est_Random).

The main function of the script to generate calculations of Pi is called prime_list_gen 

```python
def prime_list_gen(max_pair_value: int, num_of_trials: int) -> int:
    """prime_list_gen - Method to calculate the number of prime number GCD pairs. The 
    function is customized to a user input maximum pair value and set number of trials to run.
    Args:
        max_pair_value (int): Maximum value for coprime computation
        num_of_trials (int): Number of times the computation should run to sum up the number
        of coprime values
    Returns:
        int: Sum of coprime values found out of the number of trials done
    """
    x = 0
    prime_counter = 0
    for x in range(0, num_of_trials):
        # Pair Assignment
        pair_1 = random.randint(0, max_pair_value)
        pair_2 = random.randint(0, max_pair_value)

        if math.gcd(pair_1, pair_2) == 1:
            prime_counter += 1
        
        x += 1
    
    return prime_counter
```

The function takes in two arguments, a max value for the random number generator and a max trial integer, for the number of iterations used to calculate the co-prime probability in a set of data pairs. The function keeps a running sum of the number of co-prime pairs and returns this as an integer. 

The above function was the original implementation, which ran horribly slow, given the inefficient looping and calculations. A second function, prime_array_gen, was created to leverage Numpy array operations. The function below performs the exact same steps in a much more efficient manner. 

```python
def prime_array_gen(max_pair_value: int, num_of_trials: int) -> int:
    """prime_array_gen - Rebuild of prime_list_gen function to calculate the number of experimental prime
    number GCD pairs. Uses custom values for random number maximums and the number of trials used to calcualte.
    Args:
        max_pair_value (int): Maximum value for coprime computation
        num_of_trials (int): Number of times the computation should run to sum up the number
        of coprime values
    Returns:
        int: Sum of coprime values found out of the number of trials done
    """
    primes_range_array = np.array([np.gcd(np.random.randint(max_pair_value), np.random.randint(max_pair_value)) for i in range(num_of_trials)])
    filtered_primes_array = np.where(primes_range_array == 1, 1, 0) # Catches where GCD is 1
    sum_primes = np.sum(filtered_primes_array)

    return sum_primes
```

Next, there is a boilerplate function for connecting to a local SQLite database using the built in driver in Python. 

```python
def connect_to_db(conn_file: str) -> object:
    """connect_to_db - SQLite3 connection function based on whether or not a db exists.
    Args:
        conn_file (str): Path the database file to be used
    Returns:
        object: SQLite3 connection object
    """
    if os.path.exists(conn_file):
        conn = sqlite3.connect(conn_file)

        return conn
    else:
        conn = sqlite3.connect(conn_file)

        cur.execute('''
                CREATE TABLE results (
                    id integer PRIMARY KEY,
                    result REAL
                ); ''')
        
        return conn
```

This function attempts to connect to a local SQLite file and if it doesn’t exist, it creates it. This ensures the script will operate properly even if a .db file hasn’t been created yet. The SQLite table, called “results”, has a very simple schema to collect the one data point being calculated, the experimental value of Pi. 

Lastly, there is a procedure to run the script and gather a command line input from the user for the number of times the application should calculate Pi. 

```python
if __name__ == '__main__':
    # SQLite Config
    conn = connect_to_db('primesPi.db')

    cur = conn.cursor()

    # User Inputs
    max = 100_000_000_000
    trials = 100_000
    iterations = int(input('Enter the Number of times the Script Should Calculate Pi: '))

    # Actual Logic Run
    # Looping through the iterations of full trials
    i = 0
    for i in range(0, iterations):
        print('Calculating...')
        primes = prime_array_gen(max, trials)
        prob = primes / trials
        pi_est = math.sqrt(6 / prob)
        cur.execute('INSERT INTO results (result) VALUES (?)', (pi_est,))

        print(f'Trial {i} Completed...Pi = {pi_est}')

        i += 1
    
    # Save all content to the database and close out the connection
    conn.commit()    
    conn.close()
```

For this script, the maximum number for the random number generation is 100,000,000,000 and the number of trials for each calculation of Pi is 100,000. These numbers were significantly large enough to yield a decent probability pool for the Pi calculation, but also manageable enough to create 50,000 total iterations of calculations. Moving the trials number up to 1,000,000 took the script from a few hours to calculate Pi 50,000 times up to over a day on my local machine. This snipet also contains the formula being used to calculate Pi, the square root of 6 divided by the probability of a co-prime value out of the total pool of values. 

After all of the calculations are done, it is time to put together a quick visualization to see how we did. This was done in a Jupyter Notebook to allow for easier iteration and styling, but could be done in a normal .py file as well. 

```python
import pandas as pd
import plotly.express as plt

import sqlite3
import math

------

# ONLY RUN THIS CELL ONCE TO LOAD IN THE DATA AND CLOSE OUT THE CONNECTION; DONE SO DATA CAN BE GENERATED INTO THE DB WHILE THE ANALYSIS IS HAPPENING
# Load in primesPi.db
file_name = 'primesPi.db'
conn = sqlite3.connect(file_name)
sql = 'SELECT * FROM results;'

df = pd.read_sql(sql, conn)

conn.close()

------

df.info()
df.head(15)

------

# Mathmatical Analysis on the Result Table

df_median = df['result'].median()
print(df_median)

df_average = df['result'].mean()
print(df_average)

------

# Figure #1

fig = plt.scatter(df, x=df['id'], y=df['result'],
                  title=f'Pi Estimate Compared to Fixed Value of Pi: Mean Estimate - {df_median}',
                  labels={'id': 'Id of Test Run', 'result': 'Estimated Value of Pi'},
                  )

fig.add_shape(type='line',
                x0=0,
                y0=math.pi,
                x1=50000,
                y1=math.pi,
                line=dict(color='Red',), # Hard Coded Value Line
                xref='x',
                yref='y'
)

fig.add_shape(type='line',
                x0=0,
                y0=df['result'].mean(),
                x1=50000,
                y1=df['result'].mean(),
                line=dict(color='Green',), # Mean Pi Estimate Line
                xref='x',
                yref='y'
)

fig.show()
```

The notebook is fairly straight forward in what it does. The notebook loads in the SQLite database contents into a two feature dataframe, the id and results, then performs a couple of math analyses on the dataframe, before finally producing a data visualization. 

![Figure 1](/images/post_images/pi-calc-fig1.png)

The outputed data visualization shows the results on a scatter plot with two lines superimposed over the data points. The red line on the chart is the value of Pi, as it is built into Python. The green line represents the mean value of all 50,000 Pi calculations. In the above chart, these delineation is difficult to make out…

![Figure 2](/images/post_images/pi-calc-fig2.png)

Figure 2 provides a clearer view of the difference between the two values, and illustrates how closely it is possible to estimate Pi using random numbers.

This project was put together over the course of a couple of days. In approaching it again, there are a few different strategies that I would apply. For one, even with Numpy, looping over the data is slower than with a compiled language. If I were to do this again, I would use a concurrent database, like MySQL, and Go in order to more quickly and efficiently generate data by taking advantage of goroutines. In this case, I would also increase the maximum value for random number generation, the number of integers used to calculate Pi, as well as the full amount of trials run. Using more data and a larger pool of random numbers should statistically get the estimate of Pi closer to the real value.
