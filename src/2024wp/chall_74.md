# 【中等】2024 爱护你的蟒蛇

> 转眼间，2024 年已过大半，秋天悄然来临。在这个收获的季节里，除了要照顾好自己，别忘了也要继续好好对待你的「蟒蛇」——Python 哦！

> Python 不仅是你的好伙伴，还是你开发项目时的最佳助手。它温柔又强大，无论你是初学者还是经验丰富的开发者，都能在它的陪伴下感受到编程的乐趣。

> 记得每天喂养它一点新鲜的代码，保持它的活力；定期给它梳理一下臃肿的代码库，让它保持轻盈；还要记得经常带它去见见外面的世界，让它在实战中成长。

> 2024 年的下半场，让我们携手 Python，一起创造更多精彩的应用吧！

非常简单的代码逻辑，你把这个交给GPT，他都可以秒杀，吐槽一下，本来应该是简单题，但是考虑到有同学不会 python ，就调整为了中等。

题目很简单，check_flag 函数会将输入 xor 0xCC 再加上 3，最后和 enc 进行比较，如果相等，返回 True，否则返回 False。

``` python

enc = [159, 166, 177, 180, 133, 249, 159, 181, 144, 135, 249, 187, 168, 252, 181, 144, 149, 161, 252, 144, 153, 178, 181, 161, 249, 159, 144, 245, 159, 165, 144, 125, 249, 183, 166, 144, 146, 249, 182, 187, 144, 156, 159, 245, 164, 166, 188, 144, 250, 159, 144, 251, 249, 251, 245, 174]

def check_flag(text):
    text = list(text)
    for i in range(len(text)):
        text[i] = (ord(text[i]) ^ 0xCC) - 3
    return text == enc

def get_funny_response(is_correct):
    if is_correct:
        responses = [
            "Wow! You've got the magic Python spell right!",
            "Absolutely correct! Keep up the good work.",
            "Bingo! That's the flag we were looking for.",
            "Great job, coder! You cracked it.",
            "Python approves of your skills!"
        ]
    else:
        responses = [
            "Oopsie! That's not quite it. Try again!",
            "Nope, that's not the one. Remember, Python loves readable code!",
            "Sorry, wrong flag. Maybe check your syntax?",
            "Almost there, but not quite. Keep trying!",
            "That's not it, but don't worry, you'll get it!"
        ]
    return responses[random.randint(0, len(responses)-1)]

if __name__ == "__main__":
    import random
    
    user_input = input("Enter your flag: ")
    print(get_funny_response(check_flag(user_input)))
```

那么，flag 将 check_flag 这个操作反过来，将 enc 加 3 再 xor 0xCC，即可。

``` python
enc = [159, 166, 177, 180, 133, 249, 159, 181, 144, 135, 249, 187, 168, 252, 181, 144, 149, 161, 252, 144, 153, 178, 181, 161, 249, 159, 144, 245, 159, 165, 144, 125, 249, 183, 166, 144, 146, 249, 182, 187, 144, 156, 159, 245, 164, 166, 188, 144, 250, 159, 144, 251, 249, 251, 245, 174]

def decrypt_flag(text):
    for i in range(len(text)):
        text[i] = (text[i] + 3) ^ 0xCC
    return bytes(text)

if __name__ == "__main__":
    print(decrypt_flag(enc))
```
运行结果： b'nex{D0nt_F0rg3t_Th3_Pyth0n_4nd_L0ve_Y0ur_Sn4kes_1n_2024}'
