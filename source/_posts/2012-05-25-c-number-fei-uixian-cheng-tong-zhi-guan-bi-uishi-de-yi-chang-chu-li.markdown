---
layout: post
title: "C# 非UI线程通知关闭UI时的异常处理"
date: 2012-05-25 19:59
comments: true
categories: [Code]
---
C#中非UI线程正常情况下无法操作UI控件，一般使用委托的方式访问这些控件。  
示例代码：

    private void DelegateHandler()
    {
        if (this.InvokeRequired)
        {
            this.Invoke(new MethodInvoker(
                delegate()
                {
                    this.DoSomething();
                }));
        }
        else
        {
            this.DoSomething();
        }
    }
<!--more-->
按照此模式，外部通知关闭UI的委托处理逻辑就应该如下：

    private void CloseFormDelegateHandler()
    {
        if (this.InvokeRequired)
        {
            this.Invoke(new MethodInvoker(
                delegate()
                {
                    this.Close();
                }));
        }
        else
        {
            this.Close();
        }
    }

但此段代码有可能会造成程序崩溃，这是一个MS的bug，详情请查看此[链接](https://connect.microsoft.com/VisualStudio/feedback/details/560637/exception-invoke-or-begininvoke-cannot-be-called-on-a-control-until-the-window-handle-has-been-created-is-thrown)，MS的建议是 `catch and swallow the ObjectDisposedException when you calls Control.Invoke() to dispose a control from another thread.` 这个建议其实也是有问题的，事实上抛出来的异常是`InvalidOperationException`，所以应该捕获并吞掉此异常。完整的代码如下：

        private void CloseFormDelegateHandler()
        {
            if (this.InvokeRequired)
            {
                try
                {
                    this.Invoke(new MethodInvoker(
                        delegate()
                        {
                            this.Close();
                        }));
                }
                catch (ObjectDisposedException ex)
                {
                    Trace.TraceInformation("微软的建议:{0}",ex.Message);
                }
                catch (InvalidOperationException ex)
                {
                    Trace.TraceInformation("莫惊慌，微软的bug:{0}",ex.Message);
                }
            }
            else
            {
                this.Close();
            }
        }