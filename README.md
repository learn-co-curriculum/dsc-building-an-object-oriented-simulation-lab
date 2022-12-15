# Building An Object-Oriented Simulation - Lab

## Introduction

In this lab, you'll build a stochastic simulation to model herd immunity in a population and examine how a virus moves through a population, depending on what percentage of the population is vaccinated against the disease. 

## Objectives

You will be able to:

* Use inheritance to write nonredundant code
* Create methods that calculate statistics of the attributes of an object
* Create object-oriented data models that describe the real world with multiple classes and subclasses and interaction between classes

<img src='images/herd_immunity.gif'>

In the previous lesson, you saw some of the various steps you'll now take to create this simulation.

Since you'll be building a stochastic simulation that makes use of randomness, start by importing `numpy` and setting a random seed for reproducibility.

Run the cell below to do this now. 


```python
import numpy as np
import pandas as pd 
from tqdm import tqdm
np.random.seed(0)
```

## The Assumptions of Our Model

In order to build this stochastic simulation, you'll have to make some assumptions. In order to simplify the complexity of the model, assume:

* Vaccines are 100% effective 
* Infected individuals that recover from the disease are now immune to catching the disease a second time (think chickenpox)
* Dead individuals are not contagious  
* All infections happen from person-to-person interaction
* All individuals interact with the same amount of people each day
* The `r0` value (pronounced _"R-nought"_) is a statistic from the Centers for Disease Control that estimates the average number of people an infected person will infect before they are no longer contagious.  For this value, we assume:
    * that this number is out of 100 people
    * that this statistic is accurate
    
Building simulations is always a trade-off since the real world is very, very complex.  As you build the simulation, try to think about ways in which you could make our model more realistic by writing it in such a way that it eliminates one of the assumptions above (e.g. generating a float value for vaccine efficacy on a person-by-person level to eliminate our first assumption). 


## Building our `Person()` class

Start by building out our `Person()` class, which will represent the individuals in the population. 

The `Person()` class should have the following attributes:

* `is_alive = True` 
* `is_vaccinated`, a boolean value which will be determined by generating a random value between 0 and 1.  Then compare this to `(1 - pct_vaccinated)`, a variable that should be passed in at instantiation time.  If the random number is greater, then this attribute should be `True` -- otherwise, `False`
* `is_infected = False`
* `has_been_infected = False`
* `newly_infected = False`

In the cell below, complete the `Person()` class.

**_NOTE:_** To generate a random number between 0 and 1, use `np.random.random()`. 


```python
class Person(object):
    
    def __init__(self):
        self.is_alive = True
        self.is_infected = False
        self.has_been_infected = False
        self.newly_infected = False
        self.is_vaccinated = False
        
    def get_vaccinated(self, pct_vaccinated):
        if (1 - pct_vaccinated) < np.random.random():
            self.is_vaccinated = True
```

Great! Since you're using OOP to build this simulation, it makes sense to have each individual `Person` instance take care of certain details, such as determining if they are vaccinated or not.  The `pct_vaccinated` argument is a value you'll pass into the `Simulation()` class, which you can then pass along to each `Person()` once our population is created to vaccinate the right amount of people.  

## Creating our `Simulation()` class

Creating our `Simulation()` class will be a bit more involved because this class does all the heavy lifting.  You'll handle this piece by piece, and test that everything is working along the way. 

### Writing our `__init__` method

The init method should take in the following arguments at instantiation time:

* `self` 
* `population_size`
* `disease_name`
* `r0`
* `mortality_rate`
* `total_time_steps`
* `pct_vaccinated`
* `num_initial_infected`

**_Attributes_**

The attributes `self.disease_name`, `self.mortality_rate`, and `self.total_time_steps` should be set to the corresponding arguments passed in at instantiation time. 

The attribute `self.r0` should be set to set to `r0 / 100` to convert the number to a decimal between 0 and 1.

Also create attributes for keeping track of what time step the simulation is on, as well as both the current number of infected during this time step and the total number of people that have been infected at any time during the simulation.  For these, set `self.current_time_step`, `self._total_infected_counter`, and `self.current_infected_counter` to  0.

You'll also need to create an array to hold all of the `Person` objects in our simulation.  Set `self.population` equal to an empty list.  

Now comes the fun part -- creating the population, and determining if they are healthy, vaccinated, or infected.  

Follow the instructions inside the `__init__` method to write the logic that will set up our `Simulation` correctly. 



```python
class Simulation(object):
    
    def __init__(self, population_size, disease_name, r0, mortality_rate,  total_time_steps, pct_vaccinated, num_initial_infected):
        self.r0 = r0 / 100.
        self.disease_name = disease_name
        self.mortality_rate = mortality_rate
        self.total_time_steps = total_time_steps
        self.current_time_step = 0
        self.total_infected_counter = 0
        self.current_infected_counter = 0
        self.dead_counter = 0
        self.population = []
        # This attribute is used in a function that is provided for you in order to log statistics from each time_step.  
        # Don't touch it!
        self.time_step_statistics_df = pd.DataFrame()
        
        # Create a for loop the size of the population we want in this simulation
        for i in range(population_size):
            # Create new person
            new_person = Person()
            # We'll add infected persons to our simulation first.  Check if the current number of infected are equal to the 
            # num_initial_infected parameter.  If not, set new_person to be infected
            if self.current_infected_counter != num_initial_infected:
                new_person.is_infected = True
                # don't forget to increment both infected counters!
                self.total_infected_counter += 1
                self.current_infected_counter += 1
            # if new_person is not infected, determine if they are vaccinated or not by using their `get_vaccinated` method
            # Then, append new_person to self.population
            else:
                new_person.get_vaccinated(pct_vaccinated)
            self.population.append(new_person)
       
        print("-" * 50)
        print("Simulation Initiated!")
        print("-" * 50)
        self._get_sim_statistics()
        
        
    
    def _get_sim_statistics(self):
    # In the interest of time, this method has been provided for you.  No extra code needed.
        num_infected = 0
        num_dead = 0
        num_vaccinated = 0
        num_immune = 0
        for i in self.population:
            if i.is_infected:
                num_infected += 1
            if not i.is_alive:
                num_dead += 1
            if i.is_vaccinated:
                num_vaccinated += 1
                num_immune += 1
            if i.has_been_infected:
                num_immune += 1
        assert num_infected == self.current_infected_counter
        assert num_dead == self.dead_counter
        
        
        print("")
        print("Summary Statistics for Time Step {}".format(self.current_time_step))
        print("")
        print("-" * 50)
        print("Disease Name: {}".format(self.disease_name))
        print("R0: {}".format(self.r0 * 100))
        print("Mortality Rate: {}%".format(self.mortality_rate * 100))
        print("Total Population Size: {}".format(len(self.population)))
        print("Total Number of Vaccinated People: {}".format(num_vaccinated))
        print("Total Number of Immune: {}".format(num_immune))
        print("Current Infected: {}".format(num_infected))
        print("Deaths So Far: {}".format(num_dead))     
```

Great! You've now created a basic `Simulation()` class that is capable of instantiating itself according to your specifications. However, your simulation doesn't currently do anything.  Now, add the appropriate behaviors to the simulation. 

### Building Our Simulation's Behavior

For any given time step, your simulation should complete the following steps in order:

* Loop through each living person in the population

    * If the person is currently infected:
    
        * Select another random person from the population  
        
        * If this person is alive, not infected, unvaccinated, and hasn't been infected before: 
        
            * Generate a random number between 0 and 1.  If this random number is greater than `(1 - self.r0)`, then mark this new person as newly infected
            
        * If the person is vaccinated, currently infected, or has been infected in a previous round of the simulation, do nothing  
        
    * Repeat the step above until the infected person has interacted with 100 random living people from the population  
    
* Once every infected person has interacted with 100 random living people, resolve all current illnesses and new infections

    * For each person that started this round as infected, generate a random number between 0 and 1.  If that number is greater than `(1 - mortality rate)`, then that person has been killed by the disease. They should be marked as dead. Otherwise, they stay alive, and can longer catch the disease 
    
    * All people that were infected in this round move from `newly_infected` to `is_infected` 
    
Begin by breaking up most of this logic into helper functions, so that your main functions will be simple.  
    
#### `infected_interaction()` function

Write a function called `infected_interaction()` that will be called for every infected person in the population in a given time step. This function handles all the possible cases that can happen with the following logic:

* Initialize a counter called `num_interactions` to 0 
* Select a random person from `self.population` 
* Check if the person is alive. If the person is dead, we will not count this as an interaction 
* If the random person is alive and not vaccinated, generate a random number between 0 and 1.  If the random number is greater than `(1 - self.r0)`, change the random person's `newly_infected` attribute to `True` 
* Increment `num_interactions` by 1. Do not increment any of the infected counters in the simulation class -- you'll have another method deal with those 

Complete the `infected_interaction()` method in the cell below.  Comments have been provided to help you write it.

**_HINT_**: To randomly select an item from a list, use `np.random.choice()`. 


```python
def infected_interaction(self, infected_person):
    num_interactions = 0
    while num_interactions < 100:
        # Randomly select a person from self.population
        random_person = np.random.choice(self.population)
        # This only counts as an interaction if the random person selected is alive.  If the person is dead, we do nothing, 
        # and the counter doesn't increment, repeating the loop and selecting a new person at random.
        # check if the person is alive.
        if random_person.is_alive:
            # CASE: Random person is not vaccinated, and has not been infected before, making them vulnerable to infection
            if random_person.is_vaccinated == False and random_person.has_been_infected == False:
                # Generate a random number between 0 and 1
                random_number = np.random.random()
                # If random_number is greater than or equal to (1 - self.r0), set random person as newly_infected
                if random_number >= (1 - self.r0):
                    random_person.newly_infected = True
            # Don't forget to increment num_interactions, and make sure it's at this level of indentation
            num_interactions += 1

# Adds this function to our Simulation class
Simulation.infected_interaction = infected_interaction
```

#### `_resolve_states()` function

The second helper function you'll use during each time step is one that resolves any temporary states. Recall that people do not stay newly infected or infected for more than a turn. That means you need a function to figure out what happens to these people at the end of each turn, so that everything is ready to go for the next time step. 

This function will:

* Iterate through every person in the population 
* Check if the person is alive (since we don't need to bother checking anything for the dead ones)
* If the person is infected, we need to resolve whether they survive the infection or die from it:   
    * Generate a random number between 0 and 1   
    * If this number is greater than `(1 - self.mortality_rate)`, the person has died   
        * Set the person's `.is_alive` and `.is_infected` attributes both to `False`   
        * Increment the simulation's `self.dead_counter` attribute by 1 
        * Decrement the simulation's `self.current_infected_counter` attribute by 1 
    * Else, the person has survived the infection and is now immune to future infections 
        * Set the person's `is_infected` attribute to `False`
        * Set the person's `has_been_infected` attribute to `True`
        * Decrement the simulation's `self.current_infected_counter` by 1 
* If the person is newly infected:
    * Set the person's `newly_infected` attribute to `False`
    * Set the person's `is_infected` attribute to `True`
    * Increment `total_infected_counter` and `current_infected_counter` by 1 
    

Complete the function `_resolve_states()` in the cell below.  Comments have been provided to help you write it. 


```python
def _resolve_states(self):
    """
    Every person in the simulation falls into 1 of 4 states at any given time:
    1. Dead 
    2. Alive and not infected
    3. Currently infected
    4. Newly Infected
    
    States 1 and 2 need no resolving, but State 3 will resolve by either dying or surviving the disease, and State 4 will resolve
    by turning from newly infected to currently infected.
    
    This method will be called at the end of each time step.  All states must be resolved before the next time step can begin.
    """
    # Iterate through each person in the population
    for person in self.population:
        # We only need to worry about the people that are still alive
        if person.is_alive: 
            # CASE: Person was infected this round.  We need to stochastically determine if they die or recover from the disease
            # Check if person is_infected
            if person.is_infected:
                # Generate a random number
                random_number = np.random.random()
                # If random_number is >= (1 - self.mortality_rate), set the person to dead and increment the simulation's death
                # counter
                if random_number >= (1 - self.mortality_rate):
                    # Set is_alive and in_infected both to False
                    person.is_alive = False
                    person.is_infected = False
                    # Don't forget to increment self.dead_counter, and decrement self.current_infected_counter
                    self.dead_counter += 1
                    self.current_infected_counter -= 1
                else:
                    # CASE: They survive the disease and recover.  Set is_infected to False and has_been_infected to True
                    person.is_infected = False
                    person.has_been_infected = True
                    # Don't forget to decrement self.current_infected_counter!
                    self.current_infected_counter -= 1
            # CASE: Person was newly infected during this round, and needs to be set to infected before the start of next round
            elif person.newly_infected:
                # Set is_infected to True, newly_infected to False, and increment both self.current_infected_counter and 
                # self.total_infected_counter
                person.is_infected = True
                person.newly_infected = False
                self.current_infected_counter += 1
                self.total_infected_counter += 1    
                
Simulation._resolve_states = _resolve_states
```

#### `_time_step()` function

Now that you have two helper functions to do most of the heavy lifting, the `_time_step()` function will be pretty simple.  

This function should:

* Iterate through each person in the population
* If the person is alive and infected, call `self.infected_interaction()` and pass in this infected person
* Once we have looped through every person, call `self._resolve_states()` to resolve all outstanding states and prepare for the next round   
* Log the statistics from this round by calling `self._log_time_step_statistics()`.  This function has been provided for you further down the notebook  
* Increment `self.current_time_step` 


```python
def _time_step(self):
    """
    Compute 1 time step of the simulation. This function will make use of the helper methods we've created above. 
    
    The steps for a given time step are:
    1.  Iterate through each person in self.population.
        - For each infected person, call infected_interaction() and pass in that person.
    2.  Use _resolve_states() to resolve all states for the newly infected and the currently infected.
    3. Increment self.current_time_step by 1. 
    """
    # Iterate through each person in the population
    for person in self.population:
        # Check only for people that are alive and infected
        if person.is_alive and person.is_infected:
            # Call self.infecteed_interaction() and pass in this infected person
            self.infected_interaction(person)
    
    # Once we've made it through the entire population, call self._resolve_states()
    self._resolve_states()
    
    # Now, we're almost done with this time step.  Log summary statistics, and then increment self.current_time_step by 1.
    self._log_time_step_statistics()
    self.current_time_step += 1

# Adds this function to our Simulation class
Simulation._time_step = _time_step
```

Finally, you just need to write a function that logs the results of each time step by storing it in a DataFrame, and then writes the end result of the simulation to a csv file.

In the interest of time, this function has been provided for you.  You do not need to write any code in the cell below -- just run the cell.


```python
def _log_time_step_statistics(self, write_to_file=False):
    # This function has been provided for you, you do not need to write any code for it.
    # Gets the current number of dead,
    # CASE: Round 0 of simulation, need to create and Structure DataFrame
#     if self.time_step_statistics_df == None:
#         import pandas as pd
#         self.time_step_statistics_df = pd.DataFrame()
# #         col_names = ['Time Step', 'Currently Infected', "Total Infected So Far" "Alive", "Dead"]
# #         self.time_step_statistics_df.columns = col_names
#     # CASE: Any other round
#     else:
        # Compute summary statistics for currently infected, alive, and dead, and append them to time_step_snapshots_df
    row = {
        "Time Step": self.current_time_step,
        "Currently Infected": self.current_infected_counter,
        "Total Infected So Far": self.total_infected_counter,
        "Alive": len(self.population) - self.dead_counter,
        "Dead": self.dead_counter
    }
    self.time_step_statistics_df = self.time_step_statistics_df.append(row, ignore_index=True)
    
    if write_to_file:
        self.time_step_statistics_df.to_csv("simulation.csv", mode='w+')
        
Simulation._log_time_step_statistics = _log_time_step_statistics
```

#### Wrapping It All Up With `run()`

This is the function that you (or your hypothetical user) will actually interact with. It will act as the "main" function. Since you've done a great job of writing very clean, modular helper functions, you should find that this function is the simplest one in the entire program. 

This function should:

* Start a for loop that runs `self.total_time_steps` number of times.  In order to demonstrate how to easily add a progress bar to iterables with the `tqdm` library, this line has been added for you  
* Display a message telling the user the time step that it is currently working on  
* Call `self._time_step()`
* Once the simulation has finished, write the DataFrame containing the summary statistics from each step to a csv file. This line of code has also been provided for you  

In the cell below, complete the `run()` function.  Comments have been added to help you write it. 


```python
def run(self):
    """
    The main function of the simulation.  This will run the simulation starting at time step 0, calculating
    and logging the results of each time step until the final time_step is reached. 
    """
    
    for _ in tqdm(range(self.total_time_steps)):
        # Print out the current time step 
        print("Beginning Time Step {}".format(self.current_time_step))
        # Call our `_time_step()` function
        self._time_step()
    
    # Simulation is over--log results to a file by calling _log_time_step_statistics(write_to_file=True)
    self._log_time_step_statistics(write_to_file=True)

# Adds the run() function to our Simulation class.
Simulation.run = run
```

### Running Our Simulation

Now comes the fun part -- actually running your simulation!

In the cell below, create a simulation with the following parameters:

* Population size of 2000
* Disease name is `'Ebola'`
* `r0` value of 2
* Mortality rate of 0.5
* 20 time steps
* Vaccination rate of 0.85
* 50 initial infected


```python
sim = Simulation(2000, "Ebola", 2, 0.5, 20, .85, 50)
```

    --------------------------------------------------
    Simulation Initiated!
    --------------------------------------------------
    
    Summary Statistics for Time Step 0
    
    --------------------------------------------------
    Disease Name: Ebola
    R0: 2.0
    Mortality Rate: 50.0%
    Total Population Size: 2000
    Total Number of Vaccinated People: 1643
    Total Number of Immune: 1643
    Current Infected: 50
    Deaths So Far: 0



```python
sim.run()
```

      0%|          | 0/20 [00:00<?, ?it/s]

    Beginning Time Step 0


     15%|█▌        | 3/20 [00:12<01:28,  5.20s/it]

    Beginning Time Step 1
    Beginning Time Step 2


    100%|██████████| 20/20 [00:12<00:00,  1.61it/s]

    Beginning Time Step 3
    Beginning Time Step 4
    Beginning Time Step 5
    Beginning Time Step 6
    Beginning Time Step 7
    Beginning Time Step 8
    Beginning Time Step 9
    Beginning Time Step 10
    Beginning Time Step 11
    Beginning Time Step 12
    Beginning Time Step 13
    Beginning Time Step 14
    Beginning Time Step 15
    Beginning Time Step 16
    Beginning Time Step 17
    Beginning Time Step 18
    Beginning Time Step 19


    



```python
results = pd.read_csv('simulation.csv')
results
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Unnamed: 0</th>
      <th>Alive</th>
      <th>Currently Infected</th>
      <th>Dead</th>
      <th>Time Step</th>
      <th>Total Infected So Far</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>1971.0</td>
      <td>10.0</td>
      <td>29.0</td>
      <td>0.0</td>
      <td>60.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1</td>
      <td>1967.0</td>
      <td>5.0</td>
      <td>33.0</td>
      <td>1.0</td>
      <td>65.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2</td>
      <td>1964.0</td>
      <td>2.0</td>
      <td>36.0</td>
      <td>2.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>3.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>4.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>5.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>6.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>7.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>8</th>
      <td>8</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>8.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>9</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>9.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>10</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>10.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>11</th>
      <td>11</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>11.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>12</th>
      <td>12</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>12.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>13</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>13.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>14</th>
      <td>14</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>14.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>15</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>15.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>16</th>
      <td>16</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>16.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>17</th>
      <td>17</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>17.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>18</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>18.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>19</th>
      <td>19</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>19.0</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>20</th>
      <td>20</td>
      <td>1964.0</td>
      <td>0.0</td>
      <td>36.0</td>
      <td>20.0</td>
      <td>67.0</td>
    </tr>
  </tbody>
</table>
</div>



If you didn't change the random seed, your results should look like this:

<img src='images/example_sim_results.png'>

As you can see from the table above, even though the average person with Ebola will infect two other people, herd immunity protects the majority of the unvaccinated people in the population.  Although there were approximately 400 people in the population that were susceptible to Ebola, only 67 were actually infected before the virus burned itself out!

## Extra Practice

Try different values for the `pct_vaccinated()` argument, and see how it changes the results of the model. These would look great as a visualization -- consider comparing them all on the same line graph!

## Summary

Great job! You've just written a simulation to demonstrate the effects of herd immunity in action. Specifically, in this lab, you demonstrated mastery of using inheritance to write nonredundant code, creating methods that calculate statistics of the attributes of an object, and creating object-oriented data models that describe the real world with multiple classes and subclasses and interaction between classes!
