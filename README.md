# FastApiGetRouteInfo
Get Route Information

## First Idea:

```python
import inspect
def get_function_name():
    return inspect.stack()[1][3]
```

## Second way (Full BackTrack):

```python

import traceback

def get_calling_functions():
    stack_trace = traceback.extract_stack()
    calling_functions = []
    for frame in stack_trace[:-1]:
        filename = frame.filename
        function_name = frame.name
        calling_functions.append(f"{filename} -> {function_name}")
    return " -> ".join(calling_functions)

```

## Another way

```python
import inspect

async def current_frame_get_function_name() -> str:
    """Ottiene il nome della funzione dal frame corrente"""
    frame = inspect.currentframe().f_back
    func_name = frame.f_code.co_name
    return func_name
```

## Use the first way:

```python
import inspect
def get_function_name():
    return inspect.stack()[1][3]
    
@app.put("/users/{username}/password", summary = "", description="", response_description="", response_model=None)
async def change_password(username: str, 
                          new_password: str, 
                          db: Session = Depends(get_db),  
                          current_user = Depends(get_current_user)): -> Dict[str, str]:
    try:
        db_user = db.query(User).filter_by(username=username).first()
    except:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, 
                            detail=f"Internal Server Error in {get_function_name()} at \
                            {app.url_path_for(get_function_name())} - user: {current_user.username}"
                            )
    try:
        db_user.password = new_password
        db.commit()
    except Exception as e:
        db.rollback()
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, 
                            detail=f"Internal Server Error in {get_function_name()} at \
                            {app.url_path_for(get_function_name())} - user: {current_user.username} - {str(e)}"
                            )
    return {"message": "Password changed successfully"}
```


## Use the second way:

```python
import random
import time

def my_function():
    calling_functions = get_calling_functions()
    print(f"Calling functions: {calling_functions}")
    time.sleep(random.uniform(1, 3))
    another_function()

def another_function():
    calling_functions = get_calling_functions()
    print(f"Calling functions: {calling_functions}")
    time.sleep(random.uniform(1, 3))
    third_function()

def third_function():
    calling_functions = get_calling_functions()
    print(f"Calling functions: {calling_functions}")

my_function()

```
 
