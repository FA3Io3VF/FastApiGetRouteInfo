# FastApiGetRouteInfo
Get Route Information

## First Idea:

```python
import inspect
def get_function_name():
    return inspect.stack()[1][3]
```

## Second way:

```python
from fastapi.concurrency import run_in_threadpool
import inspect

async def get_function_name() -> str:
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

Please note the use of run_in_threadpool in the function dependency get_db to perform asynchronous operations
in a thread pool. This allows you to avoid blocking of the main thread, improving application performance.

```python
from fastapi.concurrency import run_in_threadpool
import inspect

async def get_function_name() -> str:
    frame = inspect.currentframe().f_back
    func_name = frame.f_code.co_name
    return func_name

@app.put("/users/{username}/password", summary = "", description="", response_description="", response_model=None)
async def change_password(username: str, 
                          new_password: str, 
                          db: AsyncSession = Depends(get_db),  
                          current_user = Depends(get_current_user)): -> Dict[str, str]: 
    try:
        db_user = db.query(User).filter_by(username=username).first()
    except:
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, 
                            detail=f"Internal Server Error in {await get_function_name()} at \
                            {app.url_path_for(await get_function_name())} - user: {current_user.username}"
                            )
        
    if not db_user:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                            detail=f"User not found at: {await get_function_name()} at \
                            {app.url_path_for(await get_function_name())} - user: {current_user.username}"
                            )
    try:
        db_user.password = new_password
        await db.commit()
    except Exception as e:
        await db.rollback()
        raise HTTPException(status_code=status.HTTP_500_INTERNAL_SERVER_ERROR, 
                            detail=f"Internal Server Error in {await get_function_name()} at \
                            {app.url_path_for(await get_function_name())} - user: {current_user.username} - {str(e)}"
                            )
    return {"message": "Password changed successfully"}
 ```
 
