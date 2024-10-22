# 【困难】Python 茶话会

我看 wp ，好多 GPT ，好吧我承认这个题目 GPT 确实可以秒杀，其实我就是想和大家介绍一个在 CTF 中很常见的算法—— TEA。

本题是 TEA 家族中最简单的，原滋原味，如果你想要知道更多，可以去了解一下 XTEA 和 XXTEA 。

题目给了一个 pyd 文件，和一个出题人的友善的 md 提示文件（为了减低难度）。

第一步，pyd 反编译，可以用 [pycdc](https://github.com/zrax/pycdc)，当然，用在线的网站也可以 [在线Python pyc文件编译与反编译](https://www.lddgo.net/string/pyc-compile-decompile) 。

```python
# Visit https://www.lddgo.net/string/pyc-compile-decompile for more information
# Version : Python 3.9

from ctypes import c_uint32
encrypted_flag = [
    1887071573,
    1987092183,
    528573895,
    0xAAD682E7L,
    1514065471,
    1557533937,
    2022731508,
    0xC695EC0EL,
    0xFF36F6EAL,
    0xE7B3EC45L,
    2120747857,
    0xE5D2379DL]

def str_to_ints(s):
    return (lambda .0 = None: [ int.from_bytes(s[i:i + 4].encode(), 'little', **('byteorder',)) for i in .0 ])(range(0, len(s), 4))


def ints_to_str(ints):
    return ''.join((lambda .0: [ int.to_bytes(i, 4, 'little', **('length', 'byteorder')).decode() for i in .0 ])(ints))


def encrypt(v, k):
    v0 = c_uint32(v[0])
    v1 = c_uint32(v[1])
    sum_val = c_uint32(0)
    delta = c_uint32(289739793)
    (k0, k1, k2, k3) = (c_uint32(k[0]), c_uint32(k[1]), c_uint32(k[2]), c_uint32(k[3]))
    for _ in range(32):
        sum_val.value += delta.value
        v0.value += (v1.value << 4) + k0.value ^ v1.value + sum_val.value ^ (v1.value >> 5) + k1.value
        v1.value += (v0.value << 4) + k2.value ^ v0.value + sum_val.value ^ (v0.value >> 5) + k3.value
    return [
        v0.value,
        v1.value]


def check_flag(text):
    if len(text) % 8 != 0:
        return False
    text_ints = None(text)
    encrypted_ints = []
    for i in range(0, len(text_ints), 2):
        pair = encrypt([
            text_ints[i],
            text_ints[i + 1]], key)
        encrypted_ints.extend(pair)
    return encrypted_ints == encrypted_flag

if __name__ == '__main__':
    print('👋 Welcome to the Tea Party! ☕️')
    print("💡 Hint: Remember to bring your own 'tea' to the party! 🫖 👩‍🍳")
    print('👉 Please enter your secret tea recipe:')
    user_input = input()
    print('🔍 Let me check your secret tea recipe...')
    key = [
        305419896,
        0x9ABCDEF0L,
        0xFEDCBA98L,
        1985229328]
    if check_flag(user_input):
        print("🎉 Your tea recipe is correct! You're the Tea Master now! 🏆")
    else:
        print("😔 Oops! It seems like you've brought the wrong blend. Try again? ☕️")
```

反编译有问题的地方，用出题人给的提示，加一点点 python 的经验（如果你想看python的字节码也不是不行），可以恢复为如下的代码：

```python
from ctypes import c_uint32

encrypted_flag = [
    1887071573, 1987092183, 
    528573895, 2866184935, 
    1514065471, 1557533937, 
    2022731508, 3331714062, 
    4281792234, 3887328325, 
    2120747857, 3855759261
]

# 将字符串转换成整数
def str_to_ints(s):
    return [int.from_bytes(s[i:i+4].encode(), byteorder='little') for i in range(0, len(s), 4)]

# 将整数转换回字符串
def ints_to_str(ints):
    return ''.join([int.to_bytes(i, length=4, byteorder='little').decode() for i in ints])

# 加密函数
def encrypt(v, k):
    v0, v1 = c_uint32(v[0]), c_uint32(v[1])
    sum_val = c_uint32(0)
    delta = c_uint32(0x11451411)
    k0, k1, k2, k3 = c_uint32(k[0]), c_uint32(k[1]), c_uint32(k[2]), c_uint32(k[3])
    for _ in range(32):
        sum_val.value += delta.value
        v0.value += ((v1.value << 4) + k0.value) ^ (v1.value + sum_val.value) ^ ((v1.value >> 5) + k1.value)
        v1.value += ((v0.value << 4) + k2.value) ^ (v0.value + sum_val.value) ^ ((v0.value >> 5) + k3.value)
    return [v0.value, v1.value]

# 检查函数
def check_flag(text):
    if len(text) % 8 != 0: return False
    # 将text转换成整数
    text_ints = str_to_ints(text)
    # 对每个整数进行加密
    encrypted_ints = []
    for i in range(0, len(text_ints), 2):
        pair = encrypt([text_ints[i], text_ints[i+1]], key)
        encrypted_ints.extend(pair)
    return encrypted_ints == encrypted_flag

if __name__ == "__main__":
    print("👋 Welcome to the Tea Party! ☕️")
    print("💡 Hint: Remember to bring your own 'tea' to the party! 🫖 👩‍🍳")
    print("👉 Please enter your secret tea recipe:")
    user_input = input()
    print("🔍 Let me check your secret tea recipe...")
    
    # 定义密钥
    key = [0x12345678, 0x9ABCDEF0, 0xFEDCBA98, 0x76543210]

    if check_flag(user_input):
        print("🎉 Your tea recipe is correct! You're the Tea Master now! 🏆")
    else:
        print("😔 Oops! It seems like you've brought the wrong blend. Try again? ☕️")
```

接下来，简单的写一个 TEA 解密脚本就可以了：

```python
from ctypes import c_uint32

encrypted_flag = [1887071573, 1987092183, 528573895, 2866184935, 1514065471, 1557533937, 2022731508, 3331714062, 4281792234, 3887328325, 2120747857, 3855759261]

# 将字符串转换成整数
def str_to_ints(s):
    return [int.from_bytes(s[i:i+4].encode(), byteorder='little') for i in range(0, len(s), 4)]

# 将整数转换回字符串
def ints_to_str(ints):
    return ''.join([int.to_bytes(i, length=4, byteorder='little').decode() for i in ints])

def decrypt(v, k):
    v0, v1 = c_uint32(v[0]), c_uint32(v[1])
    sum_val = c_uint32(0x11451411*32)
    delta = c_uint32(0x11451411)
    k0, k1, k2, k3 = c_uint32(k[0]), c_uint32(k[1]), c_uint32(k[2]), c_uint32(k[3])
    for _ in range(32):
        v1.value -= ((v0.value << 4) + k2.value) ^ (v0.value + sum_val.value) ^ ((v0.value >> 5) + k3.value)
        v0.value -= ((v1.value << 4) + k0.value) ^ (v1.value + sum_val.value) ^ ((v1.value >> 5) + k1.value)
        sum_val.value -= delta.value
    return [v0.value, v1.value]

if __name__ == "__main__":
    key = [0x12345678, 0x9ABCDEF0, 0xFEDCBA98, 0x76543210]
    decrypted_ints = []
    for i in range(0, len(encrypted_flag), 2): 
        decrypted_ints.extend(decrypt([encrypted_flag[i], encrypted_flag[i+1]], key))
    decrypted_flag = ints_to_str(decrypted_ints)
    print("Decrypted flag:", decrypted_flag)
```

运行结果：nex{7ime_F0R_@_Tea_PARTY_W1TH_pyth0n_Bytec0de!!}
