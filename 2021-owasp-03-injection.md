# Injection

Another fairly straightforward one to understand.

Imagine a web-based Python runner

```python
print("Enter your python code, we will run it and show the output!")
user_input = input()
user_output = eval(user_input)
print(user_output)
```

Imagine we send in `import os; os.system("rm -rf --no-preserve-root")` -> this would be run on the server.


_Do you think the web server would respond to the web request?_