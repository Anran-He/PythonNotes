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
