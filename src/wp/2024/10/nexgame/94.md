# 【困难】愤怒喵 NaN~

本想模仿某些题的思路出个只能用angr解的题，但现在发现由于单个函数只判断一个字符，并且返回逻辑非常简单，导致出现了一些爆破的解法，特意打乱的函数调用和数组下标的顺序也被反汇编读取偏移给还原了🤡

还是放一下angr的解法吧。校验的是`flag.png`，打不开会崩溃，之后从每个so导入一个函数，对某一个字节进行判断。因此需要做一个文件模拟，且不能关掉auto_load_libs。跑的时候如果开了veritesting=True的话，需要几十分钟，不开的话一分钟内就能出来。

```python
import angr
import sys
import claripy
def Go():
    path_to_binary = "./problem" 
    project = angr.Project(path_to_binary)
    start_address =  0x757a

    filename = 'flag.png'
    symbolic_file_size_bytes = 322
    passwd0 = claripy.BVS('password', symbolic_file_size_bytes * 8)
    passwd_file = angr.storage.SimFile(filename, content=passwd0, size=symbolic_file_size_bytes)


    initial_state = project.factory.entry_state(fs={filename: passwd_file})
    simulation = project.factory.simgr(initial_state)
    
    def is_successful(state):
        stdout_output = state.posix.dumps(1)
        if b'G0' in stdout_output:
            return True
        else: 
            return False

    def should_abort(state):
        stdout_output = state.posix.dumps(1)
        if b'N0' in  stdout_output:
            return True
        else: 
            return False

    simulation.explore(find=is_successful, avoid=should_abort)
  
    if simulation.found:
        for i in simulation.found:
            solution_state = i
            solution0 = solution_state.solver.eval(passwd0, cast_to=bytes)
            open("flag2.png", "wb").write(solution0)
    else:
        raise Exception('Could not find the solution')
    
if __name__ == "__main__":
    Go()
```