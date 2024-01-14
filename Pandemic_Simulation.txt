import numpy as np
import pandas as pd


### Functions for running Simulation
def create_students(Imm_50_50):
    '''Create a dictionary consisting of strings as keys, 
    and list with 3 elements as values, representing the attributes of each student.
    Imm_50_50 = False if no you want no students to start out with immunity, 
    True if you want each student to have a 50/50 chance of being immune.'''
    
    stud_dict = {}
    for i in range(1, 32):
        immune = ['Immune','Not Immune']
        if i == 31:
            stud_dict[f'Stud_{i}'] = ['Sick', 0, 'Not Immune']
            continue
        else:
            if Imm_50_50:
                immunity = np.random.choice(immune, p=[0.5, 0.5])
                stud_dict[f'Stud_{i}'] = ['Healthy', 0, immunity]
            else:
                stud_dict[f'Stud_{i}'] = ['Healthy', 0, 'Not Immune']
    return stud_dict

def anyone_still_sick(stud_dict):
    '''Return True if student dictionary if one or more students is sick,
    return False if all students are healthy.'''
    
    check = list(set([stud_dict[student][0] for student in stud_dict]))
    if len(check) == 1 and  check[0] == 'Healthy':
        return False
    return True

def no_of_sick_students(stud_dict):
    '''Return the number of sick students in stud_dict.'''
    
    n = 0
    for student in stud_dict:
        if stud_dict[student][0] == 'Sick':
            n += 1
    return n

def expose_students(stud_dict):
    '''Expose each healthy student in stud_dict to the flu one time.'''
    
    lst = ['Sick', 'Healthy']
    for stud in stud_dict:
        if stud_dict[stud][2] == 'Immune':
            continue
        elif stud_dict[stud][0] == 'Sick':
            continue
        else:
            status = np.random.choice(lst, p=[0.02, 0.98])
            stud_dict[stud][0] = status

def update_days_sick(stud_dict):
    '''Update number of days sick for each student in stud_dict.
    If student has been sick for 3 days, status set to 'Healthy' and 'Immune'
    Else if the student is sick, add 1 to number of days sick.'''
    
    for student in stud_dict:
        if stud_dict[student][1] == 3:
            stud_dict[student] = ['Healthy', 0, 'Immune']
        else:
            if stud_dict[student][0] == 'Sick':
                stud_dict[student][1] += 1

def update_stats(stud_dict, stats):
    '''Update dictionary of statistical data that tracks the number
    of healthy and sick students each day of the simulation'''
    
    healthy = 0
    sick = 0
    for student in stud_dict:
        if stud_dict[student][0] == 'Healthy':
            healthy += 1
        else:
            sick += 1
    stats[day] = [healthy, sick]
    
    
  
    
### Simulate the Flu Pandemic
# set how many times you want to run the simulation
trials = 10000

# Set to False for no students to start out with immunity
# Set to True for students to have a 50% chance to start out with immunity
Imm_50_50 = True

# list for storing dictionaries of data compiled from each trial
all_stats = []

# list for storing number of days each trial lasted
p_len = []

for _ in range(trials):
    # Variable for tracking number of days 
    day = 0

    # dict for storing stats 
    # {day: [number that are healthy, number that are sick]}
    stats = {}

    # Create Students Dictionary, which will be used to store various attributes 
    stud_dict = create_students(Imm_50_50)
    
    while anyone_still_sick(stud_dict):

        day += 1
        update_days_sick(stud_dict)
        
        # ensure that once everyone is healthy, the simulation terminates and starts a new trial
        if anyone_still_sick(stud_dict) == False:
            update_stats(stud_dict, stats)
            break
        
        # simulate exposing all healthy classmates to each student that is sick
        for _ in range(no_of_sick_students(stud_dict)):
            expose_students(stud_dict)
        update_stats(stud_dict, stats)

        # check stud_dict to ensure no unexpected attributes (or combination of attributes) 
        for student in stud_dict:
            assert stud_dict[student][0] == 'Healthy' or stud_dict[student][0] == 'Sick'
            assert stud_dict[student][2] == 'Immune' or stud_dict[student][2] == 'Not Immune'
            if stud_dict[student][0] == 'Healthy':
                assert stud_dict[student][1] == 0
            if stud_dict[student][0] == 'Sick':
                assert 0 <= stud_dict[student][1] <= 3
                assert stud_dict[student][2] == 'Not Immune'
    
    all_stats.append(stats) 
    p_len.append(day-1)

# get max days from all trials
max_days = 0
for d in all_stats:
    if max_days < max(d.keys()):
        max_days = max(d.keys())

# for each trial less then the max_days, add a day where all students are healthy
# this is so we can create a pandas dataframe of our statistics
for d in all_stats:        
    for i in range(1, max_days+1):
        if i not in d:
            d[i] = [31, 0]
            


# Create df dataframe containing statistics for all trial runs
df = pd.DataFrame({'Infected':(max_days+1)*[0], 'Healthy':(max_days+1)*[0]})

for d in all_stats:
    for i in range(1, max_days+1):
        df.loc[i, 'Healthy'] += d[i][0]
        df.loc[i, 'Infected'] += d[i][1]

df.loc[0, 'Healthy'] = df.loc[len(df)-1, 'Healthy']

df['Percent Infected'] = df['Infected'] / df.loc[0, 'Healthy']
df['Percent Healthy'] = df['Healthy'] / df.loc[0, 'Healthy']
df['Expected Infected'] = round(df['Percent Infected'] * len(stud_dict))
df['Expected Healthy'] = round(df['Percent Healthy'] * len(stud_dict))
print(df)

# create df_len dataframe consisting of how long each trial run lasted
df_len = pd.DataFrame({'Pandemic Length':p_len})
print(df_len.groupby(['Pandemic Length']).size())




### Export results to excel for further analysis
if Imm_50_50:
    df.to_excel(r"C:\Users\keith\Desktop\Jupyter Files\Pandemic_50_50_rerun.xlsx")
    df_len.to_excel(r"C:\Users\keith\Desktop\Jupyter Files\Pandemic_50_50_length_rerun.xlsx")
else:
    df.to_excel(r"C:\Users\keith\Desktop\Jupyter Files\Pandemic_rerun.xlsx")
    df_len.to_excel(r"C:\Users\keith\Desktop\Jupyter Files\Pandemic_length_rerun.xlsx")