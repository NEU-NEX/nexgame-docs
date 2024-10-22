# 【中等】高速迷宫

与[上一道](./chall_71.md)相比，增加了一秒内必须走完的限制。如果会算法的话，随便用bfs/dfs搜一下就行，如果不会的话，用右手原则应该也能出去？没试过。

这两道pwn预热题，主要是为了引入方便的pwntools，它可以sendlineafter，recvuntil什么的。但我看wp里有直接用socket干的，orz。

下面是用bfs的解法。

```python
from collections import deque

# 定义四个方向：上、下、左、右
DIRS = {"w": (-1, 0), "s": (1, 0), "a": (0, -1), "d": (0, 1)}  # 上  # 下  # 左  # 右


# 将迷宫地图解析为二维列表，并找到起点和终点
def parse_maze(input_maze):
    maze = []
    start = None
    end = None
    for y, line in enumerate(input_maze.split("\n")):
        row = list(line)
        maze.append(row)
        if "p" in row:
            start = (y, row.index("p"))
        if "e" in row:
            end = (y, row.index("e"))
    return maze, start, end


# 判断坐标是否在迷宫范围内
def is_valid(y, x, maze):
    #print("checking", y,x , 0 <= y < len(maze) , 0 <= x < len(maze[0]))
    return 0 <= y < len(maze) and 0 <= x < len(maze[0]) and maze[y][x] != "*"


# 使用广度优先搜索（BFS）找到迷宫的最短路径
def find_path(maze, start, end):
    queue = deque([(start, "")])  # (当前坐标, 移动方向字符串)
    visited = set()
    visited.add(start)

    while queue:
        (cur_y, cur_x), path = queue.popleft()

        if (cur_y, cur_x) == end:
            return path

        # 尝试四个方向移动
        for direction, (dy, dx) in DIRS.items():
            new_y, new_x = cur_y + dy, cur_x + dx
            if is_valid(new_y, new_x, maze) and (new_y, new_x) not in visited:
                visited.add((new_y, new_x))
                queue.append(((new_y, new_x), path + direction))
                print(queue)

    return None  # 如果没有找到路径

from pwn import *
def main():
    
    p = remote("202.199.6.66","35273")
    input_maze = p.recvlines(9)
    for i in range(len(input_maze)):
        input_maze[i] = input_maze[i].decode()
        input_maze[i] += " " * (9-len(input_maze[i]))
    input_maze = "\n".join(input_maze)
    # 解析迷宫并获取起点和终点
    maze, start, end = parse_maze(input_maze)
    print(maze)

    if not start or not end:
        print("迷宫没有找到起点或终点！")
        return

    # 计算路径
    path = find_path(maze, start, end)

    if path:
        print(f"找到路径: {path}")
    else:
        print("没有找到从起点到终点的路径！")
        
    for i in path:
        p.sendlineafter("：", i)
        
    p.interactive()


if __name__ == "__main__":
    main()
```