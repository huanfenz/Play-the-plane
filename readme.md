# 打飞机游戏

## 介绍

使用威纶通触摸屏开发的打飞机小游戏，颇有一些乐趣。

触摸屏型号：TK6071IP

使用 EasyBuilder Pro 创建。

## 效果演示

![plane1](https://github.com/huanfenz/Play-the-plane/assets/49386166/dd51ae00-c0ca-4a8d-b5a2-9cb4cc22fc6d)


![plane2](https://github.com/huanfenz/Play-the-plane/assets/49386166/5c9f3718-601e-410f-8130-8b5eadabbcad)




## 代码分析

1、初始化

> HMI启动时执行一次

```vba
macro_command main()

short zero[5]={0,0,0,0,0}
short five[5]={5,5,5,5,5}

short tmp

//初始化飞机血量
tmp = 20
SetData(tmp, "Local HMI", LW, 30, 1)

//初始化飞机位置
tmp = 360
SetData(tmp, "Local HMI", LW, 1, 1)
tmp = 200
SetData(tmp, "Local HMI", LW, 2, 1)

//初始化分数和歼敌数
tmp = 0
SetData(tmp, "Local HMI", LW, 20, 1)
SetData(tmp, "Local HMI", LW, 21, 1)

//初始化敌军
SetData(zero[0], "Local HMI", LB, 0, 5)
SetData(five[0], "Local HMI", LW, 10, 5)

//第一个敌军
SYNC_TRIG_MACRO(0)

end macro_command 
```

2、敌军随机出现

> 执行条件：LB100 为 on 时执行
> 
> 周期执行：7000ms

```vba
macro_command main()

//5个敌机的显示状态
short state[5]
GetData(state[0], "Local HMI", LB, 0, 5)

short sjs,tmp//随机数，中间变量
bool tp//布尔型中间变量

//获取一个0-4的随机数
RAND(sjs)
sjs = sjs%5

short count = 0//计次变量
//跳过已经显示的敌机
while state[sjs]==1
    sjs = sjs+1
    if sjs == 5 then
        sjs = 0
    end if
    //计次
    count = count+1
    if count == 5 then
        //全部出现了,不刷，返回
        return
    end if
wend

//敌机初始化血量
tmp = 10
SetData(tmp, "Local HMI", LW, 10+sjs, 1)

//显示敌机
tp = 1
SetData(tp, "Local HMI", LB, sjs, 1)

end macro_command
```

3、敌军消失

> 执行条件：LB100 为 on 时执行
> 
> 周期执行：0ms

```vba
macro_command main()

short state[5]
short blood[5]
short kill_enemies,score
bool tp

GetData(state[0], "Local HMI", LB, 0, 5)
GetData(blood[0], "Local HMI", LW, 10, 5)
GetData(kill_enemies, "Local HMI", LW, 20, 1)
GetData(score, "Local HMI", LW, 21, 1)

int i
for i=0 to 4
    if state[i] == 1 then
        if blood[i] == 0 then
            //敌机消失
            tp = 0
            SetData(tp, "Local HMI", LB, i, 1)
            //敌机的激光消失
            SetData(tp, "Local HMI", LB, 50+i, 1)
            //歼敌数++
            kill_enemies = kill_enemies + 1
            SetData(kill_enemies, "Local HMI", LW, 20, 1)
            //分数+=10
            score = score + 10
            SetData(score, "Local HMI", LW, 21, 1)
        end if
    end if
next

end macro_command
```

4、敌军激光

> 执行条件：LB100 为 on 时执行
> 
> 周期执行：4000ms

```vba
macro_command main()
//获取飞机起始位置和结束位置
short place_st,place_ed
GetData(place_st, "Local HMI", LW, 1, 1)
place_ed = place_st + 80

//敌军显示状态
short state[5]
GetData(state[0], "Local HMI", LB, 0, 5)
short i,j
bool tp

//激光出现
for i=0 to 4
    if state[i] == 1 then
        tp = 1
        SetData(tp, "Local HMI", LB, 50+i, 1)
    end if
next

for j=0 to 4
    for i=0 to 4
        if state[i] == 1 then
            //获取激光位置
            short curPlace
            curPlace = 120 + i * 120 + 40
            //激光位置在飞机范围内，那么扣血
            if curPlace>=place_st and curPlace<=place_ed then
                TRACE("kou xue")
                short plane_blood
                GetData(plane_blood, "Local HMI", LW, 30, 1)
                plane_blood = plane_blood - 1
                SetData(plane_blood, "Local HMI", LW, 30, 1)
                if plane_blood == 0 then
                    tp = 1
                    SetData(tp, "Local HMI", LB, 100, 1)
                end if
            end if
        end if
    next
    DELAY(100)
next

//激光消失
for i=0 to 4
    if state[i] == 1 then
        tp = 0
        SetData(tp, "Local HMI", LB, 50+i, 1)
    end if
next

end macro_command
```

5、我方射击

> 执行条件：LB100 为 on 时执行
> 
> 周期执行：500ms

```vba
macro_command main()

//获取激光位置
short light_place
GetData(light_place, "Local HMI", LW, 1, 1)
//矫正位置
light_place = light_place + 40

//获取敌机状态
short enemy_state[5]
GetData(enemy_state[0], "Local HMI", LB, 0, 5)

//遍历敌机状态
short i
for i=0 to 4
    //敌机是出现的
    if enemy_state[i] == 1 then
        //获取位置
        short curPlace
        curPlace = 120 + i * 120
        //如果飞机的激光 在 敌机位置范围中，那么敌机扣血
        if light_place >= curPlace and light_place <= curPlace+80 then
            short blood
            GetData(blood, "Local HMI", LW, 10+i, 1)
            if blood > 0 then
                blood = blood - 1
                SetData(blood, "Local HMI", LW, 10+i, 1)
            end if
        end if
    end if
next

end macro_command
```

6、重新开始

```vba
macro_command main()

SYNC_TRIG_MACRO(5)
short tp
tp = 0
SetData(tp, "Local HMI", LB, 100, 1)

end macro_command
```
