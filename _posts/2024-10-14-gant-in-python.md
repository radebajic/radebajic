---
layout: post
title: gant chart in python
date: 20241014 18:01:00
description: making customizable gant chart in python using matplotlib
tags: schedule code python
categories: visualization
featured: true
---

Visualizations are a crucial part of delivering the story of data. The time needed to produce one, and to do it in a consistent, structured manner throughout the whole claim/delay report, is often underestimated. Nothing leaves a worse impression than a chart that's squinty, poorly designed, and full of riddles.

I was searching for a way to make a Gantt chart without using planning software. Why? Because most planning software's graphical outputs aren't meant to be used in a size that can be screenshotted and pasted into an A4/letter-sized page width — anyone who's tried that knows the struggle.

Another reason is that I tried several "specialized" solutions for creating Gantt charts aimed at delivering presentation-type outputs i.e. slick, uncluttered charts ready to be used in PowerPoint or Word. However, these cost money, are clunky to use, and provide rigid, inconsistent outputs. Whoever came up with the idea to make a Gantt charting solution in PowerPoint is a true champion of awkward decisions.

If only there was a tool that's free, highly customizable, and—with some knowledge—could be utilized for visualization, data wrangling, and a thousand other tasks... 

*[Python enters the scene.]*

There are a few Python libraries that could do this, but they were either too complex or not customizable enough. So I decided to make my own approach using the famous Matplotlib. Here’s a simple way to make a Gantt chart in Python using Matplotlib (and a few other libraries).

First, we import the necessary libraries:

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.patches import Patch
from pandas import Timestamp
```
Next, we define the data we want to visualize. We have a dictionary containing the task name, department, start date, end date, and completion percentage. The data is stored in a dictionary format for presentation purposes, but it could also be in CSV, Excel, or any other format.
```python
#Project Data
data = {
    'Task': {
        0: 'Site \n Preparation',
        1: 'Foundation \n Work',
        2: 'Structural \n Framing',
        3: 'Roofing \n Installation',
        4: 'Exterior \n Finishing',
        5: 'Plumbing \n Installation',
        6: 'Electrical \n Installation',
        7: 'HVAC \n Installation',
        8: 'Interior \n Finishing',
        9: 'Landscaping \n and Cleanup',
        10: 'Final \n Inspection'
    },
    
    'Department': {
        0: 'Contractor D',
        1: 'Contractor A',
        2: 'Contractor A',
        3: 'Contractor B',
        4: 'Contractor C',
        5: 'Contractor B',
        6: 'Contractor B',
        7: 'Contractor B',
        8: 'Contractor C',
        9: 'Contractor D',
        10: 'All'
    },
    
    'Start': {
        0: Timestamp('2024-08-23'),
        1: Timestamp('2024-09-02'),
        2: Timestamp('2024-09-20'),
        3: Timestamp('2024-09-20'),
        4: Timestamp('2024-09-20'),
        5: Timestamp('2024-09-28'),
        6: Timestamp('2024-10-11'),
        7: Timestamp('2024-10-18'),
        8: Timestamp('2024-10-18'),
        9: Timestamp('2024-10-25'),
        10: Timestamp('2024-11-01')
    },
    
    'End': {
        0: Timestamp('2024-09-27'),
        1: Timestamp('2024-10-04'),
        2: Timestamp('2024-11-08'),
        3: Timestamp('2024-11-08'),
        4: Timestamp('2024-11-08'),
        5: Timestamp('2024-10-11'),
        6: Timestamp('2024-11-08'),
        7: Timestamp('2024-11-01'),
        8: Timestamp('2024-11-01'),
        9: Timestamp('2024-11-15'),
        10: Timestamp('2024-11-22')
    },
    
    'Completion': {
        0: 1.0,
        1: 1.0,
        2: 0.42,
        3: 0.42,
        4: 0.42,
        5: 1.0,
        6: 0.0,
        7: 0.0,
        8: 0.0,
        9: 0.0,
        10: 0.0
    }
}
```
Then, we prepare the data for Matplotlib. We calculate the number of days from the project start to the task start, the number of days from the project start to the task end, the days between the start and end of each task, and the days from the start to the current progression of each task. Additionally, we create a column with the corresponding color for each department.
```python
##### DATA PREP ##### 
df = pd.DataFrame(data)

# project start date
proj_start = df.Start.min()

# number of days from project start to task start
df['start_num'] = (df.Start - proj_start).dt.days

# number of days from project start to end of tasks
df['end_num'] = (df.End - proj_start).dt.days

# days between start and end of each task
df['days_start_to_end'] = df.end_num - df.start_num

# days between start and current progression of each task
df['current_num'] = df.days_start_to_end * df.Completion

# create a column with the color for each department
def color(row):
    c_dict = {'Contractor A': '#E64646', 
              'Contractor B': '#E69646', 
              'Contractor C': '#34D05C', 
              'Contractor D': '#34D0C3', 
              'All': '#3475D0'}
    return c_dict[row['Department']]

df['color'] = df.apply(color, axis=1)

# Reverse the DataFrame to reverse task order
df = df.iloc[::-1].reset_index(drop=True)
```
Next, we plot the data by creating a horizontal bar chart that includes the task name, the number of days from the project start to the current progression of each task, the days between the start and end of each task, and the color for each department. We also place the task labels inside the task bars, display the completion percentage outside the bars, and set the ticks accordingly.
```python
##### PLOT #####
fig, ax = plt.subplots(figsize=(10, 6))
# bars
ax.barh(df.Task, df.current_num, left=df.start_num, color=df.color)
ax.barh(df.Task, df.days_start_to_end, left=df.start_num, color=df.color, alpha=0.5)

# Place task labels inside the task bars and fix font size
for idx, row in df.iterrows():
    ax.text(
        row.start_num + (row.days_start_to_end / 2),  # Middle of the task bar
        float(idx) if isinstance(idx, (int, float)) else 0.0,  # Ensure idx is convertible to float
        row.Task, 
        va='center', 
        ha='center', 
        color='black', 
        fontsize=8,  # Font for task label
        weight='bold',
        wrap=True,
    )
    
    # Display completion percentage outside the bar
    ax.text(
        row.end_num + 0.5, 
        float(idx) if isinstance(idx, (int, float)) else 0.0,  # Ensure idx is convertible to float
        f"{int(row.Completion * 100)}%", 
        va='center', 
        ha='left', 
        alpha=0.8,
        fontsize=8,  # Font for completion percentage
    )

# grid lines
ax.set_axisbelow(True)
ax.xaxis.grid(color='gray', linestyle='dashed', alpha=0.2, which='major')

# ticks
xticks = np.arange(0, df.end_num.max() + 1, 7)  # Set ticks every 7 days (one week)
xticks_labels = pd.date_range(proj_start, end=df.End.max(), freq='7D').strftime("%m/%d").tolist() 
ax.set_xticks(xticks)
ax.set_xticklabels([str(label) for label in xticks_labels])  # Convert Timestamps to strings
ax.set_yticks([])

# ticks top
ax_top = ax.twiny()

# align x axis
ax.set_xlim(0, df.end_num.max()+2)
ax_top.set_xlim(0, df.end_num.max()+2)

# top ticks (markings)
xticks_top_minor = np.arange(0, df.end_num.max() + 1, 7)  # Minor ticks every 7 days
ax_top.set_xticks(xticks_top_minor, minor=True)

# top ticks (label)
xticks_top_major = np.arange(3.5, df.end_num.max() + 1, 7)  # Major ticks to align with weeks
ax_top.set_xticks(xticks_top_major, minor=False)

# week labels
xticks_top_labels = [f"W{i}" for i in np.arange(1, len(xticks_top_major) + 1)]
ax_top.set_xticklabels(xticks_top_labels, ha='center', minor=False, fontsize=8, rotation=90)

# hide major tick (we only want the label)
ax_top.tick_params(which='major', color='w')

# increase minor ticks (to mark the weeks start and end)
ax_top.tick_params(which='minor', length=8, color='k')

# remove spines
ax.spines['right'].set_visible(False)
ax.spines['left'].set_visible(False)
ax.spines['left'].set_position(('outward', 10))
ax.spines['top'].set_visible(False)

ax_top.spines['right'].set_visible(False)
ax_top.spines['left'].set_visible(False)
ax_top.spines['top'].set_visible(False)

plt.suptitle('Summary Schedule - Building A', 
fontsize=12, fontweight='bold', color='black', ha='center', va='top')
plt.subplots_adjust(top=0.85)  # Adjust the top margin to add space under the title
```
Finally, we add legends, hide the bottom ticks and labels, draw a vertical line at the status date, and add a label for the status date.
```python
##### LEGENDS #####
legend_elements = [Patch(facecolor='#E64646', label='Contractor A'),
                   Patch(facecolor='#E69646', label='Contractor B'),
                   Patch(facecolor='#34D05C', label='Contractor C'),
                   Patch(facecolor='#34D0C3', label='Contractor D'),
                   Patch(facecolor='#3475D0', label='All')]

ax.legend(handles=legend_elements, loc="upper right", frameon=True, fontsize=8, bbox_to_anchor=(0.25, 0.4))

# Hide bottom ticks and labels
ax.tick_params(axis='x', which='both', bottom=False, labelbottom=False)

# Add a status line for today's date
status = Timestamp("2024-10-11")  

status_num = (status - proj_start).days  # Calculate number of days since project start

# Draw a vertical line at the status position
ax.axvline(status_num, color='red', linestyle='--', linewidth=1.5, label='Today')

# Optionally, add a label to indicate the status line
ax.text(status_num + 0.5, len(df) - 0.5, '2024-10-11', color='red', va='center', fontsize=8)

plt.show()
```
Voila. 
<div class="row mt-3">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/gant_chart.png" class="img-fluid rounded z-depth-1" zoomable=true %}
    </div>
</div>
Gantt chart in Python using Matplotlib. You can further customize the chart by changing the colors, adding more data, or modifying the labels. The possibilities are endless.