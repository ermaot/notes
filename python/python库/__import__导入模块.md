import语句是用来导入模块的，但本质上还是通过\_\_import\_\_来工作的

```
import glob, os 
 
modules = [] 
 
for module_file in glob.glob("*-plugin.py"): 
    try: 
        module_name, ext = os.path.splitext(os.path.basename(module_file)) 
        module = __import__(module_name) 
        modules.append(module) 
    except ImportError: 
        pass # ignore broken modules 
```

在一些import不能工作的情况下，比如模块中含有-，可以使用\_\_import\_\_

