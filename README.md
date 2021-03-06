# Kirara-Fantasia-Script #
## *@author:EnderLop* ##
## 脚本仅供玩家活动末期赶成就或刷作品珠、专武之用，请勿恶意或依赖使用！ ##
## 脚本仅供玩家活动末期赶成就或刷作品珠、专武之用，请勿恶意或依赖使用！ ##
## 脚本仅供玩家活动末期赶成就或刷作品珠、专武之用，请勿恶意或依赖使用！ ##
---
---
### 写个*Python*脚本拯救模拟器玩家的肝 ###
+ 使用*Pillow*库扫色获取游戏进程信息并给出回应
+ 实现战斗结束自动点击再来一关
+ 珍藏珠就绪自动进行芳文跳，跳完开启*AUTO*模式
+ 支持全屏幕和窗口化游戏
#### 目前功能较单一，后期可能会添加其他功能 ####
1. 尝试自动续表、设置最大游戏次数
2. 尝试兼容安卓系统
3. 尝试句柄操作
---
### 更新记录 ###
+ 2019.9.27:
  1. 更新了窗口模式，这下可以边刷*Kirara Fantasia*边补番力！
+ 2020.1.1:
	1. 使用四分色块法精确查找游戏窗口的左上角及右下角的绝对坐标，避免窗口模式下手工打点误差
+ 2020.1.18：
	1. 补充循环次数限制，并去除了持续检测的等待时间，增加程序运行的效率
+ 2020.1.23：
  1. 更新了代码框架，现在程序可以适配不同的显示尺寸。
+ 2020.1.25：
	1. 脚本读入模拟器窗口的句柄，自动获取其尺寸及位置
	2. 只要模拟器窗口不太小，无论如何移动、放缩，脚本都能捕获到并执行操作
#### 更新笔记 ####
+ 2021.1月初更新补充：
  1. 理论上只要模拟器能将游戏画面开到全屏就可以使用全屏模式
  2. 如果模拟器四面都有边框且颜色一致，理论上可以使用窗口模式（*推荐用NOX模拟器，贴吧有使用教程*）
+ 2021.1月末更新补充：
  1. 25号的更新中提供了读取模拟器句柄进行分析的版本，目前还不是太稳定
  2. 已经可以从理论上实现后台操纵，但是这方面仍在考虑，挂后台刷游戏似乎让游戏失去了乐趣
---
## 使用方法 ##
1. 启动：
  + 使用*NOX*模拟器开启*Kirara Fantasia*，进入将要刷次数的对战，开启脚本，选择模式后两秒脚本启动（*成功跳忙音*）
  + **注意：刷次数过程中*NOX*模拟器必须时刻处于最上层**
2. 停止：
  + 将光标移至显示器左上角，一段时间后程序自动停止运行（*成功跳忙音*）
  + **注意：若出现错误，请重启脚本（如遇死循环，请打开任务管理器关闭脚本）**
3. 具体细节可参考“示例”文件夹中示范视频
---
---
## 源代码 ##
```
#Kirara Fantasia代肝脚本（新）
from os import getcwd
from PIL import ImageGrab
import numpy as np
import time,win32gui,win32api,win32con
totalRound = 0
windowNum = 0
x0,y0,x1,y1 = 0,0,1919,1079
lenX,lenY = 1920,1080
R0,G0,B0 = 0,0,0


"""游戏预先处理单元"""
'''窗口变换适应算法'''
def stateChange():
    global x0,y0,x1,y1,windowNum
    x0,y0,x1,y1 = win32gui.GetWindowRect(windowNum)
    cornerConfig(detectFlash())

'''1080P→窗口化绝对位置调整'''
def stateTransform(xOri,yOri):
    global x0,y0,x1,y1
    xNew = x0 + xOri * (x1 - x0) / 1919#X轴绝对位置变换
    xNew = int (xNew)
    yNew = y0 + yOri * (y1 - y0) / 1079#Y轴绝对位置变换
    yNew = int (yNew)
    return xNew,yNew

'''配置文件读入及应用'''
def configReader():
    global windowNum,lenX,lenY,R0,G0,B0
    fileRoad = getcwd() + "\\Config.txt"
    with open(fileRoad,"r",encoding = "utf-8") as configFile:
        windowName = configFile.readlines()[0].split(":")[-1][:-1]
        windowNum = win32gui.FindWindow(0,windowName)
        configFile.seek(0)
        windowSize = configFile.readlines()[1].split(":")[-1][:-1]
        lenX = eval(windowSize.split("*")[0])
        lenY = eval(windowSize.split("*")[1])
        configFile.seek(0)
        R0,G0,B0 = configFile.readlines()[2].split(":")[1:]
        R0,G0,B0 = eval(R0),eval(G0),eval(B0)

'''模糊绝对位置精确化（四分色块法）'''
def cornerConfig(info):
    global x0,y0,x1,y1,lenX,lenY
    x0Min,y0Min = max(1,x0-5),max(1,y0-5)
    x1Max,y1Max = min(x1+5,lenX-2),min(y1+5,lenY-2)
    for i in range(x0Min,x0+5):
        for j in range(y0Min,y0+100):
            if not colorCompareOrigin(info,i,j) and colorCompareOrigin(info,(i-1),j) and colorCompareOrigin(info,i,(j-1)) and colorCompareOrigin(info,(i-1),(j-1)):
                x0,y0 = i,j
    for i in range(x1-100,x1Max):
        for j in range(y1-5,y1Max):
            if not colorCompareOrigin(info,i,j) and colorCompareOrigin(info,(i+1),j) and colorCompareOrigin(info,i,(j+1)) and colorCompareOrigin(info,(i+1),(j+1)):
                x1,y1 = i,j


"""游戏信息获取单元"""
'''截取当前屏幕色块信息'''
def detectFlash():
    pic = ImageGrab.grab()#截屏
    info = np.asarray(pic,'int')#图像数字化
    info = info.swapaxes(0,1)#Shape转置
    return info

'''判断当前位置颜色是否含有特殊信息'''
def colorCompare(info,stateX,stateY,R,G,B):
    if info[stateTransform(stateX,stateY)].tolist() == [R,G,B]:
        return True
    else:
        return False
    
'''判断当前位置（绝对坐标）是否为边框'''
def colorCompareOrigin(info,stateX,stateY):
    global R0,G0,B0
    if info[stateX,stateY].tolist() == [R0,G0,B0]:
        return True
    else:
        return False

'''鼠标操作'''
def mouse_click(state_x,state_y,n):
    win32api.SetCursorPos((state_x,state_y))
    for i  in range(n):
        win32api.mouse_event(win32con.MOUSEEVENTF_LEFTDOWN, 0, 0, 0, 0)
        win32api.mouse_event(win32con.MOUSEEVENTF_LEFTUP, 0, 0, 0, 0)

'''判断珍藏技是否准备就绪'''
def starShine(info):
    if colorCompare(info,655,965,7,227,209) and colorCompare(info,655,920,250,250,235):#检查珍藏第一颗星星为亮蓝 且 周遭界面亮
        return True
    elif colorCompare(info,655,965,119,92,92) and colorCompare(info,655,920,125,125,117):#检查珍藏第一颗星星为深粉 且 周遭界面暗
        return False

'''判断当前是否为AUTO模式'''
def autoShine(info):
    if colorCompare(info,1825,30,255,255,255) and colorCompare(info,1820,55,255,255,255):#检查AUTO箭头头部为纯白
        return True
    elif colorCompare(info,1825,30,110,65,53) and colorCompare(info,1850,40,255,253,228):#检查AUTO箭头头部为深棕 且 周遭界面白
        return False

'''判断当前是否已完成对战'''
def finishShine(info):
    if colorCompare(info,1080,225,255,154,185) and colorCompare(info,1450,655,255,255,255):#检查表头为亮粉 且 两按钮间为纯白
        return True
    else:
        return False


"""战斗阶段响应单元"""
'''在珍藏技准备就绪的情况下释放芳文跳'''
def fangWenJump(info):
    if starShine(info):
        if autoShine(info):#关闭AUTO
            for round in range(5):
                if autoShine(detectFlash()):
                    mouse_click(stateTransform(1825,30)[0],stateTransform(1825,30)[1],1)
                    time.sleep(0.25)
                elif not autoShine(detectFlash()) and not (autoShine(detectFlash()) is None):
                        break
            else:
                return None
        mouse_click(stateTransform(655,965)[0],stateTransform(655,965)[1],1)
        time.sleep(0.5)
    elif colorCompare(detectFlash(),1350,920,3,113,104):
        mouse_click(stateTransform(1690,190)[0],stateTransform(1690,190)[1],1)
        time.sleep(0.5)
        mouse_click(stateTransform(1565,965)[0],stateTransform(1565,965)[1],1)
        time.sleep(0.5)
        for i in range(7):
            mouse_click(stateTransform(960,540)[0],stateTransform(960,540)[1],1)
            time.sleep(1)
        print("{}：完成一次芳文跳释放".format(time.strftime("%Y/%m/%d %H:%M:%S")))

'''在珍藏技未就绪的情况下开启AUTO模式'''
def feelFish(info):
    if not starShine(info) and not autoShine(info) and not (starShine(info) is None) and not (autoShine(info) is None):
        for round in range(5):
            if not autoShine(detectFlash()) and not (autoShine(detectFlash()) is None):
                mouse_click(stateTransform(1825,30)[0],stateTransform(1825,30)[1],1)
                time.sleep(0.5)
            elif autoShine(info):
                print("{}：完成AUTO模式的开启".format(time.strftime("%Y/%m/%d %H:%M:%S")))
                break
        else:
            return None

'''在对战结束后点击跳过结算界面'''
def finallyFinsih(info):
    if finishShine(info):
        time.sleep(1)
        for round in range(3):
            mouse_click(stateTransform(960,800)[0],stateTransform(960,800)[1],1)
            time.sleep(1)
            if colorCompare(detectFlash(),630,280,255,154,185) and colorCompare(detectFlash(),630,130,127,77,92):
                time.sleep(1)
                mouse_click(stateTransform(960,680)[0],stateTransform(960,680)[1],1)
        print("{}：完成一场大对战刷轮".format(time.strftime("%Y/%m/%d %H:%M:%S")))

'''在对战结束且仍有剩余体力的情况下自动再开一把对战'''
def restartGame(info):
    global totalRound
    if colorCompare(info,580,85,255,154,185) and colorCompare(info,695,885,200,152,110):
        mouse_click(stateTransform(630,990)[0],stateTransform(630,990)[1],1)
        time.sleep(3)
        totalRound += 1
        print("{0}：已完成{1}场战斗,正在开始第{2}场战斗".format(time.strftime("%Y/%m/%d %H:%M:%S"),totalRound,totalRound+1))

'''主函数'''
def main():
    fangWenJump(detectFlash())
    feelFish(detectFlash())
    finallyFinsih(detectFlash())
    restartGame(detectFlash())


'''运行函数'''
configReader()
mouse_position = (1,1)
print("脚本启动中...")
time.sleep(5)
print("\a{}：代肝工作开始".format(time.strftime("%Y/%m/%d %H:%M:%S")))
timeStart = time.perf_counter()
while mouse_position != (0,0):
    mouse_position = win32api.GetCursorPos()
    stateChange()
    main()
dur = time.perf_counter() - timeStart
print("\a{}：代肝工作结束".format(time.strftime("%Y/%m/%d %H:%M:%S")))
print("共完成{3}场对战的刷轮，总耗时为:{2:.0f}h {1:.0f}min {0:.2f}s".format(dur % 60,(dur / 60) % 60,dur / 3600,totalRound))
input("\a输入任何字符以结束:\n")
```
