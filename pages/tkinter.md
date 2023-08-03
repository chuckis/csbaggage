- ## tkinter inside vscode
- Most likely the problem is that you aren't starting the event loop. Without the event loop the program will exit immediately. Try altering your program to look like this:
  
  ``` python
  import tkinter as tk
  root = tk.Tk()
  root.mainloop()
  ```
- The reason you don't need to call mainloop in IDLE is because IDLE does that for you. In all other cases you must call mainloop.