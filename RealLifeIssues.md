# 1. Pass argument to Python from command line
Sometimes user input is needed during programme running process. Though we can achieve it simply from Python terminal, I need to do it from command line somehow. The advantage is that multiple types of programmes (including Python, R, VBS) can be run through one double-click (.bat file), rather than run each programme one by one.
### Step 1: Prepare Python file to get argument
```py
import sys

a = sys.argv[1]
b = sys.argv[2]
print('I got a:' + str(a))
print('I got b:' + str(b))
```

### Step 2: Prepare a .bat file to pass arguments and run Python file
```
@echo off
set /p a=First argument:
set /p b=Second argument:

python "python path.py" %a% %b%
pause
```

# 2. CSV reader
The CSV file is opened as a text file with Python's built-in open() function, which returns a file object. This is then passed to the reader, which does the heavy lifting.
```python
import csv

with open('employee_birthday.csv') as csv_file:
    csv_reader = csv.reader(csv_file, delimiter=',')

# csv_reader is a reader object
csv_reader
```
csv_reader would return a reader object, which looks like this:
```
<_csv.reader at 0x2011be5ef20>
```

Must do the operation inside with open, or error would occur:
```py
for row in csv_reader:
    print(row)
```
![Alt text](image.png)

The right codes are:
```py
with open('employee_birthday.csv') as csv_file:
    csv_reader = csv.reader(csv_file, delimiter=',')
    for row in csv_reader:
        print(row)
```
Each row would be printed out inside a list.
```
['name', 'department', 'birthday month']
['John Smith', 'Accounting', 'November']
['Erica Meyers', 'IT', 'March']
```

Then how can we count the number of rows, print out column names and print fields of a row without list bracket?
```py
fields = []
rows = []

# Open CSV file in READ mode. The file object is names as csv_file, then the file object is converted to csv.reader object, which is saved in csv_reader.
with open('employee_birthday.csv', 'r') as csv_file:
    csv_reader = csv.reader(csv_file, delimiter=',')
    # csv_reader is an iterable object, next() returns the current row and advances the iterator to the next row. The first row of the csv file contains the headers, we save them in a list called fields
    fields = next(csv_reader)

    # Now we iterate through the remaining rows using a for loop. Each row is saved to a list.
    for row in csv_reader:
        rows.append(row)
    
    # line_num is a counter which returns the number of rows that have benn iterated
    # The number of rows includes header
    print("Total number of rows: %d" %(csv_reader.line_num))

print("Field names are: " + ", ".join(field for field in fields))

print('\nFirst 2 rows are:\n')
for row in rows[:2]:
    print(row)
    for col in row:
        print(col, end="    ")
    print('\n')
```

# 3. Web scraping with selenium
Sometimes we need to scrape webpages and store some information into dataframes. One way to achieve this goal is using selenium, which is a built-in package of Python.
Let's use [Hacker News](https://news.ycombinator.com/) as an example. I want to scrape articles' titles and source links, then save them into a dataframe. Here is how I did it.
### Step 1: Import necessary libraries and access webdriver.
```py
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
import pandas as pd

# Download webdriver according to your browser
service = webdriver.EdgeService(executable_path=r"C:\\Users\\16C7656\\Documents\\webdriver\\msedgedriver.exe")
driver = webdriver.Edge(service=service)
```

### Step 2: Access target website
```py
driver.get('https://news.ycombinator.com/')
```

### Step 3: Locate title elements and save them into a list
```py
titles = driver.find_elements(By.XPATH,'//span[@class="titleline"]/a[1]')

title_list = []

for p in range(len(titles)):
    title_list.append(titles[p].text)

```

### Step 4: Locate link elements according to titles
```py

link_list = []

for i in range(0,len(title_list)):
    title_name = title_list[i]

    if '"' in title_name:
        xpath = "//*[text()='{}']/..//span[@class='sitestr']".format(title_name)
    elif "'" in title_name:
        xpath = "//*[text()=\"{}\"]/..//span[@class='sitestr']".format(title_name)
    else:
        xpath = "//*[text()='{}']/..//span[@class='sitestr']".format(title_name)

    link = driver.find_elements(By.XPATH,xpath)

    if link:
        link_list.append(link[0].text)
    else:
        link_list.append("Null")
```

### Step 5: Convert lists into dataframe
```py
data_tuples = list(zip(title_list[0:],link_list[0:])) 
df = pd.DataFrame(data_tuples, columns=['Title','Link'])
```

# 4. Convert rows to columns and columns to rows
From time to time, we need to showcase information through different table formats. Sometimes, I want to convert values into headers, and sometimes, I feel like converting headers into values. This conversion involves pivot and melt function, we'd better understand and apply them flexibly to help us present data in a better way.

Before we start, I wanna give a brief summary of what pivot and melt function do. To simply put it, melt function is used to change dataframe from wide to long, while pivot function is the opposite, and it helps to unmelt.

We all know what pivot table in Excel does, so it is easy to understand these two functions.

Let's use a dataframe as example:

### Step 1: Create a dataframe
```py
import pandas as pd

df=pd.DataFrame([['John','20231001',5,6,7],['Mary','20231001',1,2,3],['Anna','20231002',0,1,5]],columns=['name','Date','00:00','01:00', '02:00'])
```

Here is the dataframe. What I want to achieve is add an extra time column, and convert names to headers.

![Alt text](image-2.png)

### Step 2: Melt dataframe to convert time header to values
```py
melted_df = df.melt(id_vars=['name', 'Date'], var_name='Time', value_name='Value')
```

Now time headers become a seperate column, with them being values.

![Alt text](image-3.png)

### Step 3: Convert names into headers using pivot() method
```py
transposed_df = melted_df.pivot(index=['Time', 'Date'], columns='name', values='Value')
```
Now "name" column is converted to headers. It is unmelted.  

![Alt text](image-4.png)


### Step 4: Reset index
```py
transposed_df = transposed_df.reset_index()
```

![Alt text](image-5.png)

### Step 5: Reorder columns
It would be better to put "Date" column before "Time" column. So let's reorder.

```py
transposed_df = transposed_df[['Date', 'Time'] + list(transposed_df.columns[2:])]
```

![Alt text](image-6.png)

### Step 6: Rename index
No need to name index column as "name", so we need to rename it.
```py
transposed_df = transposed_df.rename_axis(None, axis=1)
```
![Alt text](image-7.png)

### Step 7: Sort by "Date" and "Time"
```py
transposed_df = transposed_df.sort_values(['Date','Time'])
```
![Alt text](image-8.png)

# 5. Pandas: convert numerical column (with NA) to integer data type
This is a common step of data manipulation, which seems easy. However, I ran into an error once, saying that "cannot convert NA to integer". This situation would occur if there exist NA values in the conlumn which requires data type conversion.

Here is an example.

### Step 1: Create a simple data frame
This data frame has no meaning at all :) Column "Number" should be integer data type, but somehow the values are decimal numbers. Please note that 
there is a missing value.

```py
import pandas as pd

df=pd.DataFrame([['John',7.0],['Mary',3.0],['Anna',5.0],['Tom',]],columns=['Name','Number'])
```

![Alt text](image-9.png)

### Step 2: Convert column data type from float to integer
**Wrong:**
```py
df['Number'] = df['Number'].astype("int")
df['Number'] = df['Number'].astype("int64")
```

Error would occur after this step.

![Alt text](image-10.png)

**Correct:**
```py
df['Number'] = df['Number'].astype("Int64")
```

Now the data frame is successfully converted.  

![Alt text](image-11.png)

Reason: **Pandas primarily uses NaN to represent missing data, and it is a float.** Therefore, this forces an array of integers with even one missing value to become floating point.
Pandas can represent integer data with possibly missing values using arrays.IntegerArray.

# 6. Delete files older than N days
This action is always required when it comes to house keeping. Nothing to say here, just share codes below.

### Scenario 1: delete files based on file creation time
```py
import os
from datetime import datetime
import glob

source_folder = r"your folder path" # remember to add "\\" in the end

# delete file by ctime
today = datetime.now()

all_files = glob.glob(source_folder + "*.csv")

for f in all_files:
    file_path = os.path.join(source_folder,f)
    file_time = os.path.getctime(file_path)
    file_date = datetime.fromtimestamp(file_time)
    day_gap = (today-file_date).days
    
    if day_gap > 30:
        os.remove(file_path)
```

### Scenario 2: delete files based on the date string in file name
```py

import os
from datetime import datetime
import glob

source_folder = r"your folder path" # remember to add "\\" in the end

today = today.date()

for f in all_files:
    file_path = os.path.join(source_folder,f)

    basename = os.path.basename(f).split('/')[-1]

    # slicing is based on your filename format
    file_date = basename[-18:-10]
   
    file_date = datetime.strptime(file_date,'%Y%m%d').date()
    day_gap = today - file_date
    days = day_gap.days

    if days < 2:
         os.remove(file_path)
```

