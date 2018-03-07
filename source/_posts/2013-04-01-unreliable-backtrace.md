---
layout: post
title: "不靠谱的backtrace"
description: "不靠谱的backtrace，增加日志后调试发现确实如此，backtrace报告的代码不是真正出问题的代码。"
category: 
tags: []
---

#起因
今天遇到一个问题，程序收到abort信号后退出，打开core文件后很快定位到目标代码，但是很奇怪的是目标代码似乎没有问题，反而是目标代码附近的代码看起来比较可疑。
增加日志后调试发现确实如此，backtrace报告的代码不是真正出问题的代码。

#重现

    #include "base.h"
    #include "stdio.h"
    boost::shared_ptr<std::string> err_str;
    boost::shared_ptr<std::string> normal_str2;
    int main(int argc, char **argv)
    {
        err_str.reset();
        normal_str2 = boost::shared_ptr<std::string>(new std::string);
        *normal_str2 = "test";
        printf("before\n");
        std::string l_test_str = *(err_str); // !!!
        printf("after\n");
        printf("%c\n", normal_str2->at(0));
        return 0;
    }

上面代码的错误很明显，使用＊获取一个空指针的值，会导致BOOST_ASSERT失败，程序会因abort信号而退出。在我的机器上，gdb告诉我，是reset的那行代码出了问题，真让人摸不着头脑。当然before还是会打印出来的。

    #include "base.h"
    #include "stdio.h"
    boost::shared_ptr<std::string> err_str;
    boost::shared_ptr<std::string> normal_str2;
    int main(int argc, char **argv)
    {
        err_str.reset();
        normal_str2 = boost::shared_ptr<std::string>(new std::string);
        *normal_str2 = "test";
        printf("before\n");
        assert(err_str.get()); // !!!
        printf("after\n");
        printf("%c\n", normal_str2->at(0));
        return 0;
    }

上面的代码会在错误行abort，并且backtrace给出的行号是正确的。

# 结论
相信日志吧！backtrace仅供参考。
本人能力有限，目前只能怀疑是＊操作破坏了栈，导致backtrace报告了错误的代码位置。有心的朋友可以用objdump看看汇编代码，若找到真正原因，希望也能和我分享。谢谢！
