```python
import os
import sys
import time

def remove_files(breday, path,keyword):
    bretime = time.time() - 3600 * 24 * breday
    
    for file in os.listdir(path):
        filename = path + os.sep + file
        if os.path.getmtime(filename) < bretime:
            try:
                if os.path.isfile(filename) and keyword in filename:
                    os.remove(filename)
            except Exception as error:
                print error
                print "%s remove faild." % filename
          

if __name__ == "__main__":
    try:
        breday = int(sys.argv[1])
        remove_files(breday,'/root/test', 'log')
    except Exception as e:
        print e
        sys.exit(-1)
```

