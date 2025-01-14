---
layout: post
title: "提高初学者的PID"
date: 2022-08-02
tag: PID
---


> 本文为翻译 [Brett Beauregard](http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-introduction/)的博文


# 提高初学者的 PID - 介绍

结合新的[Arduino PID 库](http://www.arduino.cc/playground/Code/PIDLibrary)的发布，我决定发布这一系列的帖子。最后一个库虽然可靠，但并没有真正提供任何代码解释。这一次的计划是非常详细地解释为什么代码是这样的。我希望这对两类人有用：

- 对 Arduino PID 库内部发生的事情直接感兴趣的人将得到详细的解释。
- 任何编写自己的 PID 算法的人都可以看看我是如何做事的，并借用他们喜欢的任何东西。 

这将是一个艰难的过程，但我想我找到了一种不太痛苦的方式来解释我的代码。我将从我所谓的==初学者的 PID==开始。然后我将逐步改进它，直到我们得到一个高效、健壮的 pid 算法。

### 初学者的PID

这是每个人第一次学习的 PID 方程：

![img](https://s2.loli.net/2022/08/02/IsnFcgoXjhS9KiZ.png)

这导致几乎每个人都编写了以下 PID 控制器：

```c
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double errSum, lastErr;
double kp, ki, kd;
void Compute()
{
   /*How long since we last calculated*/
   unsigned long now = millis();
   double timeChange = (double)(now - lastTime);
  
   /*Compute all the working error variables*/
   double error = Setpoint - Input;
   errSum += (error * timeChange);
   double dErr = (error - lastErr) / timeChange;
  
   /*Compute PID Output*/
   Output = kp * error + ki * errSum + kd * dErr;
  
   /*Remember some variables for next time*/
   lastErr = error;
   lastTime = now;
}
  
void SetTunings(double Kp, double Ki, double Kd)
{
   kp = Kp;
   ki = Ki;
   kd = Kd;
}
```

 Compute() 被定期或不定期地调用，它工作得很好。不过，这个系列并不是关于“效果很好”。如果我们要将这段代码变成与工业 PID 控制器相当的代码，我们必须解决一些问题：

1. **采样时间**：如果以固定间隔进行评估，则**PID**算法功能最佳。如果算法知道这个区间，我们还可以简化一些内部数学。
2. **衍生踢**：不是最大的问题，但很容易摆脱，所以我们将这样做。
3. **即时调整更改**：一个好的**PID**算法是一种可以在不影响内部工作的情况下更改调整参数的算法。
4. **重置终止缓解**：我们将探讨重置终止是什么，并实施具有附带好处的解决方案
5. **开**/**关（自动**/**手动）**：在大多数应用中，有时需要关闭**PID**控制器并手动调整输出，而不会受到控制器的干扰。
6. **初始化**：当控制器第一次打开时，我们需要==“无扰动传输”==。也就是说，我们不希望输出突然猛增到某个新值**
7. **控制器方向**：最后一个并不是健壮性名称的改变。它旨在确保用户输入带有正确符号的调谐参数。
8. **新：按比例测量**：添加此功能可以更轻松地控制某些类型的过程。

一旦我们解决了所有这些问题，我们就会有一个可靠的 PID 算法。并非巧合的是，我们还将获得最新版本的 Arduino PID 库中使用的代码。因此，无论您是在尝试编写自己的算法，还是在尝试了解 PID 库中发生的事情，我希望这对您有所帮助。让我们开始吧。

更新：在所有代码示例中，我都使用double。在 Arduino 上，double 与 float 相同（单精度）。真正的双精度对于 PID 来说是一种矫枉过正。如果您使用的语言是真正的双精度，我建议将所有双精度数更改为浮点数。



# 提高初学者的 PID - 1. 采样时间

来自: [http://brettbeauregard.com/blog/2011/04/improving-the-beginner%e2%80%99s-pid-sample-time/](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-sample-time/)

（这是关于编写可靠 PID 算法的[更大系列](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-sample-time/improving-the-beginners-pid-introduction)中的修改＃1 ）

### 问题

初学者的 PID 被设计为不规则调用。这会导致2个问题：

- 您不会从 PID 获得一致的行为，因为有时它会被频繁调用，有时则不会。
- 你需要做额外的数学计算来计算导数和积分，因为它们都依赖于时间的变化。

### 解决方案

确保定期调用 PID。我决定这样做的方法是指定计算函数在每个周期都被调用。根据预先确定的采样时间，PID 决定是否应该立即计算或返回。

一旦我们知道 PID 是以恒定间隔计算的，也可以简化微分和积分计算。奖金！

### 编码

```c
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double errSum, lastErr;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
void Compute()
{
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
      /*Compute all the working error variables*/
      double error = Setpoint - Input;
      errSum += error;
      double dErr = (error - lastErr);
 
      /*Compute PID Output*/
      Output = kp * error + ki * errSum + kd * dErr;
 
      /*Remember some variables for next time*/
      lastErr = error;
      lastTime = now;
   }
}
 
void SetTunings(double Kp, double Ki, double Kd)
{
  double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
}
 
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
```


在第 10 行和第 11 行，算法现在自行决定是否需要计算。此外，因为我们现在知道样本之间的时间将相同，所以我们不需要不断乘以时间变化。我们只需适当调整 Ki 和 Kd（第 31 和 32 行），结果在数学上是等效的，但效率更高。

尽管如此，这样做还是有点麻烦。如果用户决定在操作期间更改采样时间，则需要重新调整 Ki 和 Kd 以反映这一新变化。这就是第 39-42 行的全部内容。

另请注意，我在第 29 行将采样时间转换为秒。严格来说这不是必需的，但允许用户以 1/sec 和 s 为单位输入 Ki 和 Kd，而不是 1/mS 和 mS。

### 结果

上面的变化为我们做了三件事

1. 无论调用 Compute() 的频率如何，都将定期评估 PID 算法 [第 11 行]
2. 由于时间减法[第 10 行]，当millis() 返回到0 时不会有任何问题。这种情况每55 天才会发生一次，但我们是为了防弹，还记得吗？
3. 我们不再需要乘以和除以时间变化。由于它是一个常量，我们可以将它从计算代码中移出 [第 15+16 行]，并将其与调整常量 [第 31+32 行] 混在一起。从数学上讲，它的结果是一样的，但是每次计算 PID 时它都会保存乘法和除法

### 关于中断的旁注

如果此 PID 进入微控制器，则可以为使用中断做出很好的论证。SetSampleTime 设置中断频率，然后在时间到时调用 Compute。在这种情况下，第 9-12、23 和 24 行就没有必要了。如果您打算使用 PID 实现来执行此操作，那就去做吧！继续阅读这个系列。希望您仍能从随后的修改中获得一些好处。
 我不使用中断的三个原因

1. 就本系列而言，并不是每个人都能使用中断。
2. 如果您希望它同时实现多个 PID 控制器，事情会变得棘手。
3. 老实说，我没有想到。 [吉米罗杰斯](http://jimmieprodgers.com/)在为我校对系列时提出了建议。我可能会决定在 PID 库的未来版本中使用中断。



# 提高初学者的 PID - 2. 衍生踢

来自: [http://brettbeauregard.com/blog/2011/04/improving-the-beginner%e2%80%99s-pid-derivative-kick/](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-derivative-kick/)

（这是关于编写可靠 PID 算法的[更大系列](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-derivative-kick/improving-the-beginners-pid-introduction)中的修改＃2 ）

### 问题

此修改将稍微调整导数项。目标是消除一种称为“衍生踢”的现象。

![](https://s2.loli.net/2022/08/02/p1IHJgtKS5bD8UT.png)

上图说明了这个问题。由于 error = Setpoint - Input，Setpoint 的任何变化都会导致误差的瞬时变化。这种变化的导数是无穷大（实际上，因为 dt 不是 0，所以它只是一个非常大的数字。）这个数字被输入到 pid 方程中，这会导致输出中出现不希望有的尖峰。幸运的是，有一种简单的方法可以摆脱这种情况。

### 解决方案

![](https://s2.loli.net/2022/08/02/qZ7Dz2PEvWwb93B.png)


事实证明，误差的导数等于输入的负导数，除非设置点发生变化。这最终是一个完美的解决方案。我们不加（Kd * 误差的导数），而是减去（Kd * 输入的导数）。这被称为使用“测量导数”

### 编码

```c++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double errSum, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
void Compute()
{
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
      /*Compute all the working error variables*/
      double error = Setpoint - Input;
      errSum += error;
      double dInput = (Input - lastInput);
 
      /*Compute PID Output*/
      Output = kp * error + ki * errSum - kd * dInput;
 
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
 
void SetTunings(double Kp, double Ki, double Kd)
{
  double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
}
 
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
```

 这里的修改非常简单。我们用 -dInput 替换 +dError。我们现在记住 lastInput 而不是记住 lastError

### 结果

![](https://s2.loli.net/2022/08/02/knJu3yoATEwjti9.png)

这就是这些修改带给我们的东西。请注意，输入看起来仍然大致相同。因此，我们获得了相同的性能，但不会在每次设置点更改时发出巨大的输出尖峰。

这可能是也可能不是什么大问题。这完全取决于您的应用程序对输出尖峰的敏感程度。不过，在我看来，不踢就不需要做更多的工作，所以为什么不把事情做对呢？




# 改进初学者的 PID - 3. 调整更改

来自: [http://brettbeauregard.com/blog/2011/04/improving-the-beginner%e2%80%99s-pid-tuning-changes/](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-tuning-changes/)

（这是关于编写可靠 PID 算法的[更大系列](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-tuning-changes/improving-the-beginners-pid-introduction)中的修改 #3 ）

### 问题

在系统运行时更改调整参数的能力对于任何可敬的 PID 算法都是必须的。

![q pun ](https://s2.loli.net/2022/08/02/YJEhiVAQCdD6wRS.png)

如果你试图在它运行时改变调谐，则初学者的 PID 表现得有点疯狂。让我们看看为什么。下面是初学者PID在上述参数变化前后的状态：

![](https://s2.loli.net/2022/08/02/gh2V5j1kcJrDn3f.png)

因此，我们可以立即将这种颠簸归咎于积分项（或“I 项”）。当参数改变时，它是唯一会发生剧烈变化的东西。为什么会这样？它与初学者对积分的解释有关：

![](https://s2.loli.net/2022/08/02/YyKgBDdLHPJocXR.png)

这种解释工作正常，直到 Ki 改变。然后，突然之间，您将这个新 Ki 乘以您累积的整个误差总和。那不是我们想要的！我们只想影响事情的进展！

### 解决方案

我知道有几种方法可以解决这个问题。我在上一个库中使用的方法是重新调整 errSum。Ki翻了一番？将 errSum 减半。这可以防止 I 术语发生碰撞，并且它有效。不过它有点笨重，我想出了一些更优雅的东西。（我不可能是第一个想到这一点的人，但我确实自己想到了。那该死的！）

该解决方案需要一些基本的代数（或者是微积分？）

![](https://s2.loli.net/2022/08/02/QRdusFSvI6HG518.png)

我们没有让 Ki 存在于积分之外，而是将其带入内部。看起来我们什么都没做，但我们会看到在实践中这会产生很大的不同。

现在，我们将误差乘以当时的 Ki 值。然后我们存储它的总和。当 Ki 改变时，不会有任何颠簸，因为所有旧 Ki 都已经“在银行”可以这么说。我们无需额外的数学运算即可得到了一个平滑的传输。这可能让我成为一个极客，但我认为这很性感。

### 编码

 因此，我们将 errSum 变量替换为复合 ITerm 变量 [第 4 行]。它总结了 Ki*error，而不仅仅是错误 [第 15 行]。此外，由于 Ki 现在已隐藏在 ITerm 中，因此它已从主 PID 计算中删除 [第 19 行]。

### 结果

![](https://s2.loli.net/2022/08/02/pcLo894Yu3jIkHq.png)

![](https://s2.loli.net/2022/08/02/8bKFplhcrPQXLC3.png)

那么这是如何解决问题的。在改变 ki 之前，它重新调整了整个误差总和；我们看到的每个错误值。使用此代码，之前的错误保持不变，而新的 ki 只会影响前进的事物，这正是我们想要的。

# 改善初学者的 PID - 4. 重置结束

来自: [http://brettbeauregard.com/blog/2011/04/improving-the-beginner%e2%80%99s-pid-reset-windup/](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-reset-windup/)

（这是关于编写可靠 PID 算法的[更大系列](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-reset-windup/improving-the-beginners-pid-introduction)中的修改＃4 ）

### 问题

![ ](https://s2.loli.net/2022/08/02/iN7vfD3AaSuPBx2.png)

 重置结束是一个陷阱，它可能比其他任何陷阱都更能吸引初学者。当 PID 认为它可以做一些它不能做的事情时，就会发生这种情况。例如，Arduino 上的 PWM 输出接受 0-255 之间的值。默认情况下，PID 不知道这一点。如果它认为 300-400-500 会起作用，它就会尝试那些期望得到它需要的值。由于实际上该值被限制在 255，因此它只会继续尝试越来越高的数字而没有任何进展。

这个问题以奇怪的滞后形式显示出来。在上面我们可以看到，输出被“上升”，远远超过了外部极限。当设定值下降时，输出必须在低于255线之前就开始下降。

### 解决方案 - 第 1 步

![](https://s2.loli.net/2022/08/02/7e3Z5cTOaibsEHx.png)

有几种方法可以减轻饱和，但我选择的一种方法如下：告诉 PID 输出限制是什么。在下面的代码中，您将看到现在有一 个 SetOuputLimits 函数。一旦达到任一限制，pid 就停止求和（积分）。它知道没有什么可做的；由于输出不会结束，当设定点下降到我们可以做某事的范围时，我们会立即得到响应。

### 解决方案 - 第 2 步

但是请注意，在上图中，虽然我们摆脱了终止滞后，但我们并没有完全做到这一点。PID认为它的输出（Output）未能使输入的数据（Input）达到设定值（Setpoint）。为什么？比例期限和（在较小程度上）衍生期限。 

即使积分项已被安全钳位，P 和 D 仍将其二等份相加，从而产生高于输出限制的结果。在我看来，这是不可接受的。如果用户调用名为“SetOutputLimits”的函数，他们必须假设这意味着“输出将保持在这些值内”。因此，对于第 2 步，我们将其作为有效假设。除了限制 I 项，我们还限制输出值，使其保持在我们期望的位置。

（注意：您可能会问为什么我们需要对两者进行钳位。如果我们无论如何都要进行输出，为什么要单独钳制积分？如果我们所做的只是钳制输出，那么积分项将继续增长和增长。尽管在升压过程中输出看起来不错，但我们会在降压时看到明显的滞后。）

### 编码

```c
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double ITerm, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
void Compute()
{
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
      /*Compute all the working error variables*/
      double error = Setpoint - Input;
      ITerm+= (ki * error);
      if(ITerm> outMax) ITerm= outMax;
      else if(ITerm< outMin) ITerm= outMin;
      double dInput = (Input - lastInput);
 
      /*Compute PID Output*/
      Output = kp * error + ITerm- kd * dInput;
      if(Output > outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
 
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
 
void SetTunings(double Kp, double Ki, double Kd)
{
  double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
}
 
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
 
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
    
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
 
   if(ITerm> outMax) ITerm= outMax;
   else if(ITerm< outMin) ITerm= outMin;
}
```

添加了一个新函数以允许用户指定输出限制 [第 52-63 行]。这些限制用于钳制 I 项 [17-18] 和输出 [23-24]

### 结果

![Still no Lag  What pid thinks it'ssending](https://s2.loli.net/2022/08/02/wWmojUzZTLqBENv.png)

 如我们所见，消除了结束。此外，输出保持在我们想要的位置。这意味着不需要对输出进行外部钳位。如果您希望它的范围从 23 到 167，您可以将它们设置为输出限制。



# 提高初学者的PID - 5. 开/关

来自: [http://brettbeauregard.com/blog/2011/04/improving-the-beginner%e2%80%99s-pid-onoff/](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-onoff/)

（这是关于编写可靠 PID 算法的[更大系列](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-onoff/improving-the-beginners-pid-introduction)中的修改 #5 ）

### 问题

拥有 PID 控制器固然很好，但有时您并不关心它要说什么。

![img](https://s2.loli.net/2022/08/02/PS9uGMAbV6TdBDs.png)

假设在您的程序中的某个时刻，您希望将输出强制为某个值（例如 0），您当然可以在调用例程中执行此操作：

```c++
void loop()
{
    Compute();
    Output = 0;
}
```

这样，无论 PID 说什么，您只需覆盖其值。然而，这在实践中是一个可怕的想法。PID 将变得非常困惑：“我一直在移动输出，但什么也没发生！是什么赋予了？！让我再挪动一下。” 因此，当您停止覆盖输出并切换回 PID 时，输出值可能会立即发生巨大变化。

### 解决方案

这个问题的解决方案是有一种方法来关闭和打开 PID。这些状态的常用术语是“手动”（我将手动调整值）和“自动”（PID 将自动调整输出）。让我们看看这是如何在代码中完成的：

### 编码

```c++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double ITerm, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
bool inAuto = false;
 
#define MANUAL 0
#define AUTOMATIC 1
 
void Compute()
{
   if(!inAuto) return;
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
      /*Compute all the working error variables*/
      double error = Setpoint - Input;
      ITerm+= (ki * error);
      if(ITerm> outMax) ITerm= outMax;
      else if(ITerm< outMin) ITerm= outMin;
      double dInput = (Input - lastInput);
 
      /*Compute PID Output*/
      Output = kp * error + ITerm- kd * dInput;
      if(Output > outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
 
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
 
void SetTunings(double Kp, double Ki, double Kd)
{
  double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
}
 
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
 
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
    
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
 
   if(ITerm> outMax) ITerm= outMax;
   else if(ITerm< outMin) ITerm= outMin;
}
 
void SetMode(int Mode)
{
  inAuto = (Mode == AUTOMATIC);
}

```

一个相当简单的解决方案。如果您未处于自动模式，请立即离开计算功能而不调整输出或任何内部变量。

### 结果

![img](https://s2.loli.net/2022/08/02/eVj1A4ikRQLqzhE.png)

确实，您可以通过不从调用例程中调用 Compute 来实现类似的效果，但是此解决方案保留了包含的 PID 的工作原理，这正是我们所需要的。通过将事情保持在内部，我们可以跟踪处于哪种模式，更重要的是它让我们知道何时更改模式。这就引出了下一个问题……



# 提高初学者的 PID - 6. 初始化

来自: [http://brettbeauregard.com/blog/2011/04/improving-the-beginner%e2%80%99s-pid-initialization/](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-initialization/)

（这是关于编写可靠 PID 算法的[更大系列](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-initialization/improving-the-beginners-pid-introduction)中的修改 #6 ）

### 问题

在上一节中，我们实现了关闭和打开 PID 的功能。我们关闭了它，但现在让我们看看当我们重新打开它时会发生什么：

![img](https://s2.loli.net/2022/08/02/fUpwME5vPad9YL1.png)

哎呀！PID 跳回到它发送的最后一个Output值，然后从那里开始调整。这会导致我们不想拥有的 Input 颠簸。

### 解决方案

这个很容易修复。由于我们现在知道何时开启（从手动到自动），我们只需要初始化一些东西即可平滑过渡。这意味着按摩 2 个存储的工作变量（ITerm 和 lastInput）以防止输出跳跃。

### 编码

```C++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double ITerm, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
bool inAuto = false;
 
#define MANUAL 0
#define AUTOMATIC 1
 
void Compute()
{
   if(!inAuto) return;
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
      /*Compute all the working error variables*/
      double error = Setpoint - Input;
      ITerm+= (ki * error);
      if(ITerm> outMax) ITerm= outMax;
      else if(ITerm< outMin) ITerm= outMin;
      double dInput = (Input - lastInput);
 
      /*Compute PID Output*/
      Output = kp * error + ITerm- kd * dInput;
      if(Output> outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
 
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
 
void SetTunings(double Kp, double Ki, double Kd)
{
  double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
}
 
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
 
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
    
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
 
   if(ITerm> outMax) ITerm= outMax;
   else if(ITerm< outMin) ITerm= outMin;
}
 
void SetMode(int Mode)
{
    bool newAuto = (Mode == AUTOMATIC);
    if(newAuto && !inAuto)
    {  /*we just went from manual to auto 我们刚刚从手动变为自动 */
        Initialize();
    }
    inAuto = newAuto;
}
 
void Initialize()
{
   lastInput = Input;
   ITerm = Output;
   if(ITerm> outMax) ITerm= outMax;
   else if(ITerm< outMin) ITerm= outMin;
}

```

我们修改了 SetMode(...) 来检测从手动到自动的转换，并添加了我们的初始化函数。它设置 ITerm=Output 来处理积分项，设置 lastInput = Input 以防止导数出现尖峰。比例项不依赖于过去的任何信息，因此不需要任何初始化。

### 结果

![img](https://s2.loli.net/2022/08/02/R3QtOghd9xTA1Br.png)

我们从上图中看到，正确的初始化会导致从手动到自动的无扰转换：正是我们所追求的。
 [下一页 >>](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-initialization/improving-the-beginners-pid-direction)

### 更新：为什么不是 ITerm=0？

我最近收到很多问题，问为什么我不在初始化时设置 ITerm=0。作为答案，我会要求您考虑以下情况：pid 是手动的，并且用户已将输出设置为 50。一段时间后，该过程稳定到输入 75.2。用户将设定值设为 75.2 并打开 pid。应该发生什么？

我认为在切换到自动后，输出值应该保持在 50。由于 P 和 D 项将为零，发生这种情况的唯一方法是将 ITerm 初始化为输出值。 

如果您需要将输出初始化为零，则无需更改上面的代码。在将 PID 从手动变为自动之前，只需在调用例程中设置 Output=0。



# 提高初学者的 PID - 7. 方向

来自: http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-direction/

（这是关于编写可靠的 PID 算法的[更大系列中](http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-direction/improving-the-beginners-pid-introduction)的最后一个修改）

### 问题

PID 将连接的过程分为两类：直接作用和反向作用。到目前为止，我展示的所有示例都是直接作用的。也就是说，输出的增加导致输入的增加。对于反向作用过程，情况正好相反。例如，在冰箱中，增加冷却会导致温度下降。为了使初学者PID以相反的过程工作，kp、ki和kd的符号都必须为负。

这本身不是问题，但用户必须选择正确的符号，并确保所有参数具有相同的符号。

### 解决方案

为了使过程更简单一些，我要求 kp、ki 和 kd 都 >=0。如果用户连接到反向进程，则他们使用 SetControllerDirection 函数单独指定。这确保参数都具有相同的符号，并希望使事情更直观。

### 编码

```c++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double ITerm, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
bool inAuto = false;
 
#define MANUAL 0
#define AUTOMATIC 1
 
#define DIRECT 0
#define REVERSE 1
int controllerDirection = DIRECT;
 
void Compute()
{
   if(!inAuto) return;
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
      /*Compute all the working error variables*/
      double error = Setpoint - Input;
      ITerm+= (ki * error);
      if(ITerm > outMax) ITerm= outMax;
      else if(ITerm < outMin) ITerm= outMin;
      double dInput = (Input - lastInput);
 
      /*Compute PID Output*/
      Output = kp * error + ITerm- kd * dInput;
      if(Output > outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
 
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
 
void SetTunings(double Kp, double Ki, double Kd)
{
   if (Kp<0 || Ki<0|| Kd<0) return;
 
  double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
 
  if(controllerDirection ==REVERSE)
   {
      kp = (0 - kp);
      ki = (0 - ki);
      kd = (0 - kd);
   }
}
 
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
 
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
 
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
 
   if(ITerm > outMax) ITerm= outMax;
   else if(ITerm < outMin) ITerm= outMin;
}
 
void SetMode(int Mode)
{
    bool newAuto = (Mode == AUTOMATIC);
    if(newAuto == !inAuto)
    {  /*we just went from manual to auto*/
        Initialize();
    }
    inAuto = newAuto;
}
 
void Initialize()
{
   lastInput = Input;
   ITerm = Output;
   if(ITerm > outMax) ITerm= outMax;
   else if(ITerm < outMin) ITerm= outMin;
}
 
void SetControllerDirection(int Direction)
{
   controllerDirection = Direction;
}
```

## PID 完成

就这样结束了。我们已经将“初学者的 PID”变成了我目前知道如何制作的最强大的控制器。对于那些正在寻找 PID 库的详细解释的读者，我希望你能得到你想要的。对于那些编写自己的 PID 的人，我希望您能够收集到一些想法，从而为您节省一些周期。

两个最终说明：

1. 如果本系列中的某些内容看起来不对，请告诉我。我可能错过了一些东西，或者可能只是需要在我的解释中更清楚。无论哪种方式，我都想知道。

2. 这只是一个基本的PID。为了简单起见，我有意忽略了许多其他问题。在我的头顶上：前馈、重置回扣、整数数学、不同的 pid 形式、使用速度而不是位置。如果有兴趣让我探索这些主题，请告诉我。



# 介绍比例测量

来自: http://brettbeauregard.com/blog/2017/06/introducing-proportional-on-measurement/

已经有一段时间了，但我终于更新了[Arduino PID 库](https://github.com/br3ttb/Arduino-PID-Library)。我添加的是一个几乎不为人知的功能，但我认为这将是业余爱好者社区的福音。它被称为“比例测量”（对测量结果（Input）进行比例运）（简称 PonM）。

### 为什么你应该关心

![process-types](https://s2.loli.net/2022/08/02/s4U5LZ1V7MqwuFh.png)

有一些过程被称为 "整合过程"。这些过程是由pid的输出来控制输入的变化率。在工业中，这些仅占所有流程的一小部分，但在业余爱好者的世界中，这些人**无处不在**：真空低温烹调法、线性滑动和 3D 打印机挤出机温度都是此类流程的示例。

这些过程令人沮丧的是，使用传统的 PI 或 PID 控制，它们会超出设定值。不是有时，而是总是：

![PonE-9](https://s2.loli.net/2022/08/02/IHuk1Qx5EJczVoB.png)

如果你不了解这一点，这可能会让人抓狂。您可以永远调整调谐参数，但超调仍然存在；基础数学使之如此。测量的比例改变了基础数学。因此，可以找到不会发生 过冲的调整参数集： 

![PonM-9](https://s2.loli.net/2022/08/02/16KWUrjQu9o4wDT.png)

过冲仍然可以确定，但并非不可避免。使用 PonM 和正确的调整参数，真空或线性滑动可以直接进入设定点而不会过冲。


### 那么什么是测量的比例？

类似于[Derivative on Measurement](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-derivative-kick/)（对测量结果进行微分），PonM 改变了比例项所关注的内容。P-Term 不是误差，而是 PID 输入的当前值。

对Error进行比例运算：

![PonE-eqn](https://s2.loli.net/2022/08/02/mAyhasFTv9Jn6bj.png)

对测量结果（Input）进行比例运（比例测量）：

![](https://s2.loli.net/2022/08/02/mAyhasFTv9Jn6bj.png)

与测量导数（对测量结果进行微分）不同，它对性能的影响是巨大的。使用 DonM，导数项仍然具有相同的作用：抵抗急剧变化，从而抑制由 P 和 I 驱动的振荡。另一方面，对测量结果进行比例运算从根本上改变了比例项的作用。它不是像 I 这样的驱动力，而是像 D 一样的阻力。这意味着对于 PonM，更大的 Kp 会使您的控制器更加保守。

### 伟大的。但这如何消除过冲？

为了理解这个问题，并加以解决，我们可以看看不同的术语以及它们对整个PID输出的贡献。这是对使用传统 PID 的积分过程（真空低温烹调法）的设定点变化的响应：

![PonE-Components](https://s2.loli.net/2022/08/02/ZK1uXELaVcyrT8e.png)

1. 输出必须返回到7（平衡点）以保持稳定状态；

2. P和D项返回到0，所以l项需要自己将输出恢复到7；

3. 为了使I项回到7，需要有负的误差（过冲）。

   

需要注意的两件大事是： 

- 当我们处于设定点时，I 项是整体输出的唯一贡献者。
- 即使开始和结束时的设定值不同，输出也会返回相同的值。该值通常称为“平衡点”：导致 0 输入斜率的输出。对于真空低温烹调法，这相当于刚好足以补偿周围环境的热量损失。

在这里我们可以看到为什么会发生超调，并且会一直发生。当设定值第一次改变时，存在的误差会导致 I 项增长。为了在新的设定点上保持过程稳定，输出将需要返回到平衡点。发生这种情况的唯一方法是缩小 I-Term。发生这种情况的唯一方法是产生负误差，只有在高于设定点时才会发生这种情况。

### PonM 改变了游戏规则

这是使用比例测量（和相同的调谐参数）控制的相同真空低温烹调法：

![PonM-Components](https://s2.loli.net/2022/08/02/fc8ltwzjYi5QeaW.png)

1. 输出必须返回到7（平衡点）以保持稳定状态；
2. P-Term随着输入的增加而增加阻力，在稳定状态下达到新的值；
3. 因为p-Term现在有贡献，I-erm不需要下降，所以不需要过冲。
4. (D端仍然返回到0)



在这里你应该注意到： 

- P 项现在提供了阻力。输入越高，它变得越负。
- 在 P 项在新设定点变为零之前，它现在继续有一个值。

P-Term 不返回到 0 的事实是关键。这意味着 I-Term 不必自行返回平衡点。P 和 I 一起可以将输出返回到平衡点，而无需收缩 I-Term。因为它不需要收缩，所以不需要超调。

### 如何在新的PID库中使用它

如果您准备好在测量中尝试 Proportional，并且您已经安装了最新版本的 PID 库，那么设置它非常简单。使用 PonM 的主要方法是在重载的构造函数中指定它：

![Constructor](https://s2.loli.net/2022/08/02/n5Z1lrWqPL4Eem2.png)

如果你想在运行时在 PonM 和 PonE 之间切换，SetTunings 函数也被重载：

![SetTunings](https://s2.loli.net/2022/08/02/SLQRvzHqNdUP7Ks.png)

只需要在想要切换的时候调用重载的方法即可。否则，您可以使用常规的 SetTunings 功能，它会记住您的选择。



# 8. 测量的比例 - 代码

来自: http://brettbeauregard.com/blog/2017/06/proportional-on-measurement-the-code/

在上[一篇文章中，](http://brettbeauregard.com/blog/2017/06/introducing-proportional-on-measurement/)我花了我所有的时间来解释比例测量的好处。在这篇文章中，我将解释代码。人们似乎很欣赏我[上次](http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-introduction/)解释事情的循序渐进的方式[，](http://brettbeauregard.com/blog/2011/04/improving-the-beginners-pid-introduction/)所以这就是我在这里要做的。下面的 3 次详细介绍了我如何将 PonM 添加到 PID 库中。

### First Pass – 初始输入和比例模式选择

```C++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double ITerm, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
bool inAuto = false;
  
#define MANUAL 0
#define AUTOMATIC 1
  
#define DIRECT 0
#define REVERSE 1
int controllerDirection = DIRECT;
  
#define P_ON_M 0
#define P_ON_E 1
bool pOnE = true;
double initInput;
 
void Compute()
{
   if(!inAuto) return;
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
      /*Compute all the working error variables*/
      double error = Setpoint - Input;
      ITerm+= (ki * error);
      if(ITerm > outMax) ITerm= outMax;
      else if(ITerm < outMin) ITerm= outMin;
      double dInput = (Input - lastInput);
  
      /*Compute P-Term*/
      if(pOnE) Output = kp * error;
      else Output = -kp * (Input-initInput); 
 
      /*Compute Rest of PID Output*/
      Output += ITerm - kd * dInput;
      if(Output > outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
  
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
  
void SetTunings(double Kp, double Ki, double Kd, int pOn) 
{
   if (Kp<0 || Ki<0|| Kd<0) return;
  
   pOnE = pOn == P_ON_E;
   
   double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
  
  if(controllerDirection ==REVERSE)
   {
      kp = (0 - kp);
      ki = (0 - ki);
      kd = (0 - kd);
   }
}
  
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
  
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
  
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
  
   if(ITerm > outMax) ITerm= outMax;
   else if(ITerm < outMin) ITerm= outMin;
}
  
void SetMode(int Mode)
{
    bool newAuto = (Mode == AUTOMATIC);
    if(newAuto == !inAuto)
    {  /*we just went from manual to auto*/
        Initialize();
    }
    inAuto = newAuto;
}
  
void Initialize()
{
   lastInput = Input;
   initInput = Input;
   ITerm = Output;
   if(ITerm > outMax) ITerm= outMax;
   else if(ITerm < outMin) ITerm= outMin;
}
  
void SetControllerDirection(int Direction)
{
   controllerDirection = Direction;
}
```

随着输入的变化，测量成比例提供增加的阻力，但如果没有参考框架，我们的表现会有点不稳定。如果我们第一次打开控制器的时候PID Input是10000，我们真的要从Kp*10000开始抵抗吗？不。我们想使用我们的初始输入作为参考点（第 108 行），随着输入的变化（第 38 行）增加或减少电阻。

我们需要做的另一件事是允许用户选择他们是要进行误差比例还是测量。在[上](http://brettbeauregard.com/blog/2017/06/introducing-proportional-on-measurement/)一篇[文章](http://brettbeauregard.com/blog/2017/06/introducing-proportional-on-measurement/)之后，PonE 似乎没用了，但重要的是要记住，对于许多循环，它运行良好。因此，我们需要让用户选择他们想要的模式（第 51 和 55 行），然后在计算中采取相应的行动（第 37 和 38 行）。

### Second Pass - 即时调整更改

虽然上面的代码确实有效，但它有一个我们以前见过的问题。当在运行时更改调整参数时，我们会得到一个不希望的信号。

![PonM-blip](https://s2.loli.net/2022/08/02/jmMAIsZKEdSiy4p.png)

为什么会这样？

![PonM-blip-math](https://s2.loli.net/2022/08/02/LVrgFSMof8du2IO.png)

[我们最后一次看到这个，](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-tuning-changes/)是积分被一个新的 Ki 重新调整。这一次，是 (Input – initInput) 被 Kp 重新缩放。我选择的解决方案与我为 Ki 所做的类似：我没有将 Input – initInput 视为乘以当前 Kp 的整体单元，而是将其分解为当时乘以 Kp 的单个步骤：

![PonM expansion](https://s2.loli.net/2022/08/02/yIBAxhL8FmGcR47.png)

```C++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double ITerm, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
bool inAuto = false;
  
#define MANUAL 0
#define AUTOMATIC 1
  
#define DIRECT 0
#define REVERSE 1
int controllerDirection = DIRECT;
  
#define P_ON_M 0
#define P_ON_E 1
bool pOnE = true;
double PTerm;
 
void Compute()
{
   if(!inAuto) return;
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
    
      /*Compute all the working error variables*/      
      double error = Setpoint - Input;   
      ITerm+= (ki * error);  
      if(ITerm > outMax) ITerm= outMax;      
      else if(ITerm < outMin) ITerm= outMin;  
      double dInput = (Input - lastInput);
 
      /*Compute P-Term*/
      if(pOnE) Output = kp * error; 
      else 
      { 
         PTerm -= kp * dInput; 
         Output = PTerm; 
      } 
       
      /*Compute Rest of PID Output*/
      Output += ITerm - kd * dInput; 
    
      if(Output > outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
  
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
  
void SetTunings(double Kp, double Ki, double Kd, int pOn)
{
   if (Kp<0 || Ki<0|| Kd<0) return;
  
   pOnE = pOn == P_ON_E;
   
   double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
  
  if(controllerDirection ==REVERSE)
   {
      kp = (0 - kp);
      ki = (0 - ki);
      kd = (0 - kd);
   }
}
  
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
  
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
  
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
  
   if(ITerm > outMax) ITerm= outMax;
   else if(ITerm < outMin) ITerm= outMin;
}
  
void SetMode(int Mode)
{
    bool newAuto = (Mode == AUTOMATIC);
    if(newAuto == !inAuto)
    {  /*we just went from manual to auto*/
        Initialize();
    }
    inAuto = newAuto;
}
  
void Initialize()
{
   lastInput = Input;
   PTerm = 0;
   ITerm = Output;
   if(ITerm > outMax) ITerm= outMax;
   else if(ITerm < outMin) ITerm= outMin;
}
  
void SetControllerDirection(int Direction)
{
   controllerDirection = Direction;
}
```

我们现在没有将 Input-initInput 的整体乘以 Kp，而是保留一个工作总和 PTerm。在每一步，我们只将当前输入变化乘以 Kp，然后从 PTerm 中减去它（第 41 行）。在这里我们可以看到变化的影响：

![PonM-no-blip](https://s2.loli.net/2022/08/02/B3pStDojzXFTmPI.png)

![PonM-no-blip-math](https://s2.loli.net/2022/08/02/4d2uiqztJh9fQZV.png)

因为旧的 Kps 是“in the bank”，调整参数的变化只会影响我们前进

### Final Pass – 求和问题。

我不会详细说明上面的代码有什么问题（花哨的趋势等）。这是相当不错的，但它仍然存在重大问题。例如：

1. **结束，有点**：虽然最终输出限制在 outMin 和     outMax 之间，但 PTerm 有可能在不应该增长的情况下增长。它不会像[积分结束](http://brettbeauregard.com/blog/2011/04/improving-the-beginner’s-pid-reset-windup/)那么糟糕，但它仍然不能被接受
2. **即时更改**：如果用户在运行时从 P_ON_M 更改为 P_ON_E，那么一段时间后返回，PTerm 不会被属性初始化，并且会导致输出颠簸

还有更多，但仅这些就足以了解真正的问题是什么。我们之前已经处理过所有这些，回到我们创建 ITerm 的时候。我没有为 PTerm 遍历并重新实现相同的解决方案，而是决定采用更美观的解决方案。

通过将 PTerm 和 ITerm 合并为一个名为“outputSum”的变量，P_ON_M 代码就可以从所有已经到位的 ITerm 修复中受益，并且因为代码中没有两个和，所以没有不必要的冗余。

```C++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double outputSum, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
bool inAuto = false;
  
#define MANUAL 0
#define AUTOMATIC 1
  
#define DIRECT 0
#define REVERSE 1
int controllerDirection = DIRECT;
  
#define P_ON_M 0
#define P_ON_E 1
bool pOnE = true;
 
 
void Compute()
{
   if(!inAuto) return;
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
    
      /*Compute all the working error variables*/      
      double error = Setpoint - Input;   
      double dInput = (Input - lastInput);
      outputSum+= (ki * error);  
       
      /*Add Proportional on Measurement, if P_ON_M is specified*/
      if(!pOnE) outputSum-= kp * dInput
       
      if(outputSum > outMax) outputSum= outMax;      
      else if(outputSum < outMin) outputSum= outMin;  
     
      /*Add Proportional on Error, if P_ON_E is specified*/
      if(pOnE) Output = kp * error; 
      else Output = 0;
       
      /*Compute Rest of PID Output*/
      Output += outputSum - kd * dInput; 
    
      if(Output > outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
  
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
  
void SetTunings(double Kp, double Ki, double Kd, int pOn)
{
   if (Kp<0 || Ki<0|| Kd<0) return;
  
   pOnE = pOn == P_ON_E;
   
   double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
  
  if(controllerDirection ==REVERSE)
   {
      kp = (0 - kp);
      ki = (0 - ki);
      kd = (0 - kd);
   }
}
  
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
  
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
  
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
  
   if(outputSum > outMax) outputSum= outMax;
   else if(outputSum < outMin) outputSum= outMin;
}
  
void SetMode(int Mode)
{
    bool newAuto = (Mode == AUTOMATIC);
    if(newAuto == !inAuto)
    {  /*we just went from manual to auto*/
        Initialize();
    }
    inAuto = newAuto;
}
  
void Initialize()
{
   lastInput = Input;
    
   outputSum = Output;
   if(outputSum > outMax) outputSum= outMax;
   else if(outputSum < outMin) outputSum= outMin;
}
  
void SetControllerDirection(int Direction)
{
   controllerDirection = Direction;
}
```

上述功能是现在在 Arduino PID 的 v1.2.0 中存在的功能。

### 设定点加权。

我没有将以下内容添加到 Arduino 库代码中，但如果您想推出自己的功能，可能会对这个功能感兴趣。设定点加权的核心是一种同时拥有 PonE 和 PonM 的方法。通过指定 0 和 1 之间的比率，您可以拥有 100% PonM、100% PonE（分别）或介于两者之间的某个比率。如果您有一个不完美集成的过程（如回流炉）并且想要解决这个问题，这可能会有所帮助。

最终我决定此时不将它添加到库中，因为它最终成为另一个需要调整/解释的参数，我认为由此产生的好处不值得。无论如何，如果您想修改代码以使用设置点加权而不是纯粹的 PonM/PonE 选择，这里是代码：

```C++
/*working variables*/
unsigned long lastTime;
double Input, Output, Setpoint;
double outputSum, lastInput;
double kp, ki, kd;
int SampleTime = 1000; //1 sec
double outMin, outMax;
bool inAuto = false;
  
#define MANUAL 0
#define AUTOMATIC 1
  
#define DIRECT 0
#define REVERSE 1
int controllerDirection = DIRECT;
  
#define P_ON_M 0
#define P_ON_E 1
bool pOnE = true, pOnM = false;
double pOnEKp, pOnMKp;
 
 
void Compute()
{
   if(!inAuto) return;
   unsigned long now = millis();
   int timeChange = (now - lastTime);
   if(timeChange>=SampleTime)
   {
    
      /*Compute all the working error variables*/      
      double error = Setpoint - Input;   
      double dInput = (Input - lastInput);
      outputSum+= (ki * error);  
       
      /*Add Proportional on Measurement, if P_ON_M is specified*/
      if(pOnM) outputSum-= pOnMKp * dInput
       
      if(outputSum > outMax) outputSum= outMax;      
      else if(outputSum < outMin) outputSum= outMin;  
     
      /*Add Proportional on Error, if P_ON_E is specified*/
      if(pOnE) Output = pOnEKp * error; 
      else Output = 0;
       
      /*Compute Rest of PID Output*/
      Output += outputSum - kd * dInput; 
    
      if(Output > outMax) Output = outMax;
      else if(Output < outMin) Output = outMin;
  
      /*Remember some variables for next time*/
      lastInput = Input;
      lastTime = now;
   }
}
  
void SetTunings(double Kp, double Ki, double Kd, double pOn)
{
   if (Kp<0 || Ki<0|| Kd<0 || pOn<0 || pOn>1) return;
  
   pOnE = pOn>0; //some p on error is desired;
   pOnM = pOn<1; //some p on measurement is desired;  
   
   double SampleTimeInSec = ((double)SampleTime)/1000;
   kp = Kp;
   ki = Ki * SampleTimeInSec;
   kd = Kd / SampleTimeInSec;
  
  if(controllerDirection ==REVERSE)
   {
      kp = (0 - kp);
      ki = (0 - ki);
      kd = (0 - kd);
   }
    
   pOnEKp = pOn * kp; 
   pOnMKp = (1 - pOn) * kp;
}
  
void SetSampleTime(int NewSampleTime)
{
   if (NewSampleTime > 0)
   {
      double ratio  = (double)NewSampleTime
                      / (double)SampleTime;
      ki *= ratio;
      kd /= ratio;
      SampleTime = (unsigned long)NewSampleTime;
   }
}
  
void SetOutputLimits(double Min, double Max)
{
   if(Min > Max) return;
   outMin = Min;
   outMax = Max;
  
   if(Output > outMax) Output = outMax;
   else if(Output < outMin) Output = outMin;
  
   if(outputSum > outMax) outputSum= outMax;
   else if(outputSum < outMin) outputSum= outMin;
}
  
void SetMode(int Mode)
{
    bool newAuto = (Mode == AUTOMATIC);
    if(newAuto == !inAuto)
    {  /*we just went from manual to auto*/
        Initialize();
    }
    inAuto = newAuto;
}
  
void Initialize()
{
   lastInput = Input;
   outputSum = Output;
   if(outputSum > outMax) outputSum= outMax;
   else if(outputSum < outMin) outputSum= outMin;
}
  
void SetControllerDirection(int Direction)
{
   controllerDirection = Direction;
}
```

代替将 pOn 设置为整数，它现在以允许比率的双精度形式出现（第 58 行）。除了一些标志（第 62 和 63 行）之外，在第 77-78 行计算加权 Kp 项。然后在第 37 和 43 行，加权的 PonM 和 PonE 贡献被添加到整个 PID 输出中。
