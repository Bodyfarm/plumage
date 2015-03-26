## What is Plumage ##

Plumage is a bridge between Python and Tcl, meaning it embedds Tcl into Python by using both the Tcl API and the CPython API.

It is an attempt to fix problems found in the current tkinter, like Tcl <=> Python type conversions, bgerror handling, multithreaded applications calling into tk (and other extensions -- not fully done yet), and others.

## How to use Plumage ##

While you could use it in "raw mode", you probably won't want to.
Since there are some differences between this wrapper and the current one in Python's stdlib, a patch was needed to take the differences into account. This patch lives in the [download section](http://code.google.com/p/plumage/downloads/list), and alternatively there is an improved Tkinter which lives in its own branch (http://plumage.googlecode.com/svn/branches/Tkinter) that you could checkout. Warning: The patch provided is deprecated, expect a new one soon.

Note that extensions and other programs using the previous Tkinter.py may need to do some minor adaptations in its code in order to work with Plumage.
Plumage is able to run IDLE without touching it, but it is needed to replace the bgerror Tcl command to something else that does nothing.

## Playing without Tkinter.py ##

It is possible, albeit discouraged, to use Plumage without the Tkinter interface to the C module. But here goes something to get you going:


**Creating a Tcl interpreter**

```
import plumage

interp = plumage.Interp()
```

Now you have a Tcl interp with Tk loaded too. If you don't want to load tk right now, pass a use\_tk=False parameter to the Interp class when instantiating it, later if you want you can call interp.load\_tk() then.


**Calling into Tcl**

To call into Tcl you will almost always be using the Interp.call method, or sometimes the Interp.eval method. Interp.call takes as many arguments as you want to give it, while Interp.eval takes a single string.

For instance, doing:

```
interp.call("array", "get", "tcl_platform")
```

Should return a Python dict (if you are using tcl 8.5+) to you, something like this:

```
{'threaded': '1', 'pointerSize': 4, 'machine': 'i686', 'platform': 'unix', 'wordSize': 4,
 'user': 'gpolo', 'osVersion': '2.6.27-7-generic', 'os': 'Linux', 
 'byteOrder':  'littleEndian'}
```

And doing:

```
interp.eval("array get tcl_platform")
```

Should return a dict with the same contents for you. It tends to be more natural to use the call method because you can pass it Python objects, which will then be converted to Tcl objects, while when using eval you must pass a single string already formatted as Tcl code.


**Getting/Setting/Unsetting tcl variables**

There are some methods for accessing tcl variables. These are: get\_var, set\_var, unset\_var, get\_arrayvar, set\_arrayvar, unset\_arrayvar.

So, if you wanted the value of "threaded" in the tcl\_platform array, you could do:

```
interp.get_arrayvar("tcl_platform", "threaded")
```

Which can result in

```
'1'
```

or

```
plumage.TclError: can't read "tcl_platform(threaded)": no such element in array
```

if your Tcl was not configured with support for threads.


**Creating functions**

It is rare to not need to create new functions (or commands, in Tcl) while using Tkinter. If you want to a function to be called when you press a Tk Button, there must be a function associated with it. Tkinter does this for you naturally, but since we are using the raw interface we must do it ourselves.

How you would do it (if it weren't for Tkinter):

```
def test():
    print "Look at me!"

interp.createcommand("somename", test)
```

Then suppose you have a button:

```
interp.call("button", ".b", "-command", "somename")
interp.call("pack", ".b")
interp.mainloop()
```

When you click the button, the command test (which is named "somename" in Tcl) is invoked.
You could have passed any amount of arguments when calling or creating the command, which then would be passed to the associated command when it gets called.