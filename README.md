#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#define MAXSIZE 50
#define _CRT_SECURE_NO_WARNINGS 1
#define OK 1;
#define ERROR 0;
using namespace std; 

typedef int SElemType;

typedef struct //定义一个顺序存储栈
{
    SElemType data[MAXSIZE];
    int top;
}SqStack;

int To_PostFix(char *infix, char *postfix);
int  Calculator(char *postfix, int *result);

int InitStack(SqStack *s)//构造函数 
{
    s->top = -1;
    return OK;
}

int ClearStack(SqStack *s)//置空函数 
{
    s->top = -1;
    return OK;
}

int StackEmpty(SqStack s)//检验栈是否为空 
{
    if (s.top == -1)
       { return OK;} 
    else
        return ERROR;
}

int StackLength(SqStack *s)//输出栈的长度 
{
    return s->top + 1;
}

int Push(SqStack *s, SElemType e)//压栈操作 
{
    if (s->top == MAXSIZE - 1)
        return 0;
    s->top++;//栈长度增加 
    s->data[s->top] = e;//利用e返回栈顶元素 
    return OK;
}

int Pop(SqStack *s, SElemType *e)//出栈操作 
{
    if (s->top == -1)
        return ERROR;
    *e = s->data[s->top];//利用e返回栈顶元素 
    s->top--;//栈的长度减少 
    return OK;
}

/*******************中序表达式转换为后序表达式********************************/
int To_PostFix(char *infix, char *postfix)
{
    SqStack s;
    int e = 0;
    int i = 0, j = 0;
    int flag = 0;

    if (InitStack(&s) != 1)   //判断栈是否为空，
        return 0;

    while (infix[i] != '\0')   //说明栈中有元素。
    {

        while (infix[i] >= '0' && infix[i] <= '9')    //如果是数字则输出
        {
            if (flag) //考虑负数的情况
            {
                flag = 0;
                postfix[j++] = '-';
            }
            postfix[j++] = infix[i];//让存放后缀表达式的数组存放字符
            i++;
            if (infix[i]<'0' || infix[i]>'9') //判断是否为操作符，如果是，让后缀表达式中存放一个' '
                postfix[j++] = ' ';
        }
        if (infix[i] == ')' )       //如果是关于括号的符号，则进行栈操作
        {
            Pop(&s, &e);
            while (e != '(' )
            {
                postfix[j++] = e;
                postfix[j++] = ' ';
                Pop(&s, &e);
            }
        }
        else if (infix[i] == '+' || infix[i] == '-') //对于同运算级的+和-操作
        {
            if (infix[i] == '-' && (i == 0 || (i != 0 && (infix[i - 1]<'0' || infix[i - 1]>'9'))))  //当'-'号处于第一位，或前面是符号时，为负号标志
                flag = 1;
            else if (StackEmpty(s))
                Push(&s, infix[i]);
            else
            {
                do
                {
                    Pop(&s, &e);
                    if (e == '(' )
                        Push(&s, e);
                    else
                    {
                        postfix[j++] = e;
                        postfix[j++] = ' ';
                    }
                } while (!StackEmpty(s) && e != '(' );
                Push(&s, infix[i]);
            }
        }
        else if (infix[i] == '*' || infix[i] == '/' || infix[i] == '(' )//对于乘除以及括号的开始进行压栈
            Push(&s, infix[i]);
        else if (infix[i] == '\0')
            break;
        else
            return 0;
        i++;
    }

    while (!StackEmpty(s))
    {
        Pop(&s, &e);
        postfix[j++] = e;
        postfix[j++] = ' ';
    }

    ClearStack(&s);
    return 1;
}

int  Calculator(char *postfix, int *result)
{
    SqStack s;
    char *op; //存放后缀表达式中的每个因数或运算符  
    char *buf = postfix; //声明bufhe saveptr两个变量，是strtok_r函数的需要。  
    char *saveptr = NULL;
    int d, e, f;

    if (InitStack(&s) != 1)
        return 0;

    while ((op = strtok(buf, " ")) != NULL)//利用字符串分割函数
    {
        buf = NULL;                     //把指针置空
        switch (op[0])
        {
        case '+':
            Pop(&s, &d);
            Pop(&s, &e);
            f = d + e;
            Push(&s, f);
            break;
        case '-':
            if (op[1] >= '0' && op[1] <= '9')    //是负号而不是减号
            {
                d = atoi(op);
                Push(&s, d);
                break;
            }
            Pop(&s, &d);
            Pop(&s, &e);
            f = e - d;
            Push(&s, f);
            break;
        case '*':
            Pop(&s, &d);
            Pop(&s, &e);
            f = e*d;
            Push(&s, f);
            break;
        case '/':
            Pop(&s, &d);
            Pop(&s, &e);
            f = e / d;
            Push(&s, f);
            break;
        default:                //考虑数字的情况，进行atoi函数进行转化
            d = atoi(op);
            Push(&s, d);        //进行压栈
            break;
        }
    }
    Pop(&s, result);
    ClearStack(&s);
    return 0;
}

int main()
{
    char infix[MAXSIZE] = { 0 };
    char postfix[MAXSIZE] = { 0 };
    int result = 0;
    printf("请输入一个中缀表达式：\n");
    scanf("%s", infix);
    fflush(stdin);
    To_PostFix(infix, postfix);
    printf("转换得到的后缀表达式是：\n");
    printf("%s\n", postfix);
    Calculator(postfix, &result);
    printf("最后得到的结果：\n");
    printf("%d\n", result);
    system("pause");
    return 0;
}





