[<img alt="LOGO" src="http://www.gqylpy.com/static/img/favicon.ico" height="21" width="21"/>](http://www.gqylpy.com)
[![Version](https://img.shields.io/pypi/v/gqylpy_exception)](https://pypi.org/project/gqylpy_exception/)
[![Python Versions](https://img.shields.io/pypi/pyversions/gqylpy_exception)](https://pypi.org/project/gqylpy_exception)
[![License](https://img.shields.io/pypi/l/gqylpy_exception)](https://github.com/gqylpy/gqylpy-exception/blob/master/LICENSE)
[![Downloads](https://pepy.tech/badge/gqylpy_exception/month)](https://pepy.tech/project/gqylpy_exception)

# gqylpy-exception


> 在执行 `raise` 语句的同时创建异常类，无需事先定义异常类，方便快捷。例如，你想抛出一个名为 `NotUnderstandError` 的异常，
> 导入 `import gqylpy_exception as ge` 后直接执行 `raise ge.NotUnderstandError` 即可。
> 
> `gqylpy-exception` 还提供了两个处理异常的装饰器：
> - `TryExcept`: 截获被装饰的函数中触发的异常。并将异常信息输出到终端，不是抛出。
> - `Retry`: 同上，并会尝试重新执行，通过参数控制次数，在达到最大次数后抛出异常。

<kbd>pip3 install gqylpy_exception</kbd>


### 使用 `gqylpy_exception` 创建异常类
```python
import gqylpy_exception as ge

raise ge.AnError(...)
```
`gqylpy_exception` 可以创建任何异常类。`AnError` 不是 `gqylpy_exception` 中内置的，它是在你的代码执行到 `ge.` 
时创建的，魔化方法 `__getattr__` 的特性。你还可以使用魔法方法 `__getitem__` 获得它：
```python
e: ge.GqylpyError = ge['AnError'](...)
```
是的，使用 `gqylpy_exception` 创建的异常类都继承 `GqylpyError`，`GqylpyError` 继承内置的 `Exception`。

另外，`gqylpy_exception` 不会重复创建异常类，创建过的异常类将存入 `ge.__history__` 字典，当你再次创建时从这个字典中取值。

### 使用装饰器 `TryExcept` 处理函数中引发的异常
```python
from gqylpy_exception import TryExcept

@TryExcept(ValueError)
def func():
    int('a')
```
默认的处理流程是将异常简要信息输出到终端。当然，也可以输出到文件或做其它处理，通过参数控制：
```python
def TryExcept(
        etype:          Union[type, tuple],
        *,
        ignore:         bool               = False,
        output_raw_exc: bool               = False,
        logger:         logging.Logger     = None,
        ereturn:        Any                = None,
        ecallback:      Callable           = None,
        eexit:          bool               = False
):
    ...
```
- 参数 `ingore=True` 将静默处理异常，没有任何输出。
- 参数 `output_raw_exc=True` 将输出完整的异常信息，注意其优先级低于 `ignore`。
- 参数 `logger` 接收一个日志记录器对象，`TryExcept` 希望使用日志记录器记录异常。缺省情况下使用 `sys.stderr` 输出异常。
- 参数 `ereturn` 用于指定被装饰函数在引发异常后的返回值，默认为 `None`。
- 参数 `ecallback` 接收一个可调用对象，这个可调用对象还需接收多个参数：引发的异常对象，被装饰的函数和它的所有参数。
- 参数 `eexit=True` 将在引发异常后抛出 `SystemExit(4)`。

### 使用装饰器 `Retry` 重试函数中引发的异常
```python
import gqylpy_exception as ge

@ge.Retry(count=3, cycle=1)
def func():
    int('a')
```
