## 基于websocket和opencv的水下机器人控制系统

### 项目简介

本次项目的目的是实现水下机器人的基本控制和特殊颜色物体识别。本次实验使用的水下机器人的ROV的一种。ROV，即遥控无人潜水器（Remote Operated Vehicle ），无人水下航行器（Unmanned Underwater Vehicle，UUV）的一种，系统组成一般包括：动力推进器、遥控电子通讯装置、 黑白或彩色摄像头、摄像俯仰云台、用户外围传感器接口、实时在线显示单元、导航定位装置、自动舵手导航单元、辅助照明灯和凯夫拉零浮力拖缆等单元部件。

### 需求分析

本次实验需要实现的功能如下：

- 完成与水下机器人的连接，能够获取水下机器人状态数据。
  - 这里需要使用利用websocket建立主机和机器人的通信，随时获取机器人的状态。
  - 这里需要的状态数据为水下机器人的三维坐标以及航向角，向水下机器人传输的数据为四个电机的转速参数以及内部树莓派的启动和关闭。
- 完成获取水下机器人摄像机图像数据。
  - 这里需要使用python中的opencv函数库获取摄像头的数据，主要实现将实时的识别图像显示在pc上，同时获取已识别物体的中心坐标，传入pc。
- 实现基本的控制水下机器人推进器和摄像机云台的功能。
  - 考虑到机器人的四个推进器在同一个参数情况下推进力不同，我们经过多次测试实现了机器人的平稳漂浮、旋转、前进、后退等基本单步控制，实现了推进器的需求。
- 整体业务需求：水池内部前方放置红、黄、绿三个颜色的信标（悬挂，顺序可能会变化），要求水下机器人能够在程序的控制下，自主的运动至指定颜色的信标处，并接触信标。
- 在本次实验中，需要满足水下机器人在控制下稳定、自主地到达信标。考虑到绳子的牵引作用，在行进过程中需要对机器人的行进方向进行调整。
- 接触信标之后需要让水下机器人立刻停止，防止对信标和机器人造成影响。
- 出发点要求：水下机器人位于水池信标的对侧，摄像机必须背对或侧对信标。
- 初始化运动要求：水下机器人能够在电子罗盘的引导下，调整方向，旋转至正确的头部朝向信标的方向。
- 水下机器人在运动过程中，逐渐寻找目标；找到指定颜色后，前进并接触信标。
- 测试环节必须包含随机角度出发的测试用例

程序流程图：

<div align=center><img src="https://amao996.github.io/blogs/project/robot/fig1.png" width="  "></div><br>

### 详细设计

#### 视频流获取与物体检测

首先创建一个Window窗口，传入摄像头的信息，创建VideoCapture。确认摄像头已开启的情况下，显示缓存、调节摄像头分辨率、设置FPS。

然后逐帧捕获摄像头的信息，根据读取到的图片，进行高斯模糊、转换演的空间、定义特殊颜色的无图HSV阈值、对图片进行二值化处理、腐蚀、膨胀消除噪声、寻找图中轮廓等一系列操作获取特殊颜色物体的分布范围。

如果存在至少一个轮廓则进行如下操作，即找到面积最大的轮廓、使用最小外接圆圈出面积最大的轮廓、计算轮廓的矩和重心，处理半径大于5的轮廓（因此特殊颜色物体识别只关心近距离的识别）。之后，在屏幕上划出最小外接圆，保存此时物体中心点的坐标。

```c++
def cv_config_video(self):
    global target_found
    cv2.namedWindow('Window', flags=cv2.WINDOW_NORMAL | cv2.WINDOW_KEEPRATIO | cv2.WINDOW_GUI_EXPANDED)
    # 摄像头的IP地址,http://用户名：密码@IP地址：端口/
    ip_camera_url = 'http://192.168.137.100:8080/?action=stream'  # 摄像头的地址
    # 创建一个VideoCapture
    cap = cv2.VideoCapture(ip_camera_url)
    print('IP摄像头是否开启： {}'.format(cap.isOpened()))

    print(cap.get(cv2.CAP_PROP_BUFFERSIZE))
    cap.set(cv2.CAP_PROP_BUFFERSIZE, 1)
    # 调节摄像头分辨率
    cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1920)
    cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 1080)

    # 设置FPS
    cap.set(cv2.CAP_PROP_FPS, 25)
    print(cap.get(cv2.CAP_PROP_FPS))

    while True:
        # 第一个参数返回一个布尔值（True/False），代表有没有读取到图片；第二个参数表示截取到一帧的图片
        ret, frame = cap.read()
        cv2.imshow('Frame', frame)

        if cv2.waitKey(1) & keyboard.is_pressed('q'):  # 按q退出
            break
        frame = imutils.resize(frame, width=500)
        # 进行高斯模糊并转换颜色空间到HSV
        blurred = cv2.GaussianBlur(frame, (11, 11), 0)
        hsv = cv2.cvtColor(blurred, cv2.COLOR_BGR2HSV)
        # 定义红色无图的HSV阈值
        lower_red = np.array([20, 100, 100])  # 低阈值
        higher_red = np.array([30, 255, 255])  # 高阈值
        mask = cv2.inRange(hsv, lower_red, higher_red)

        mask = cv2.erode(mask, None, iterations=7)
        mask = cv2.dilate(mask, None, iterations=2)
        contours, hierarchy = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        print(len(contours))
        # 如果存在轮廓
        if len(contours) > 0:
            print("a")
            largest_contour = max(contours, key=cv2.contourArea)
            moment = cv2.moments(largest_contour)
            cx = int(moment["m10"] / moment["m00"])
            cy = int(moment["m01"] / moment["m00"])
            error = cx - frame.shape[1] / 2

            ((x, y), radius) = cv2.minEnclosingCircle(largest_contour)
            M = cv2.moments(largest_contour)
            center = (int(M["m10"] / M["m00"]), int(M["m01"] / M["m00"]))
            if radius > 5:
                cv2.circle(frame, (int(x), int(y)), int(radius), (0, 255, 255), 2)
                cv2.circle(frame, center, 5, (0, 0, 255), -1)
                self.x = int(x)
                self.y = int(y)
            max_contour = sorted(contours, key=cv2.contourArea)[-1]
            if self.get_sign_angle(np.mean(max_contour[:, :, 0]), np.sum(mask)) == 0:
                return 0
```

### 检测识别物体并返回坐标

查看是否检测到物体，当获取的mask超过10时，认为检测到目标，同时保存位置信息，以便pid算法的计算。

```c++
def get_sign_angle(self, position, cre):
	global target_found
	if cre > 10:
    	print("检测到目标")
        self.stop()
        target_found = False
        return 0
	else:
		print("未检测到目标")
    	return 1
```

### 机器人控制模块

包括机器人的初始化、转向、前进与停止。

```c++
async def begin(self):
    async with websockets.connect(self.uri) as ws:
        await ws.send("{}:{}:{}:{}:{}".format(1462, 1462, 1462, 1462, 0))
        data = await ws.recv()
        self.r = data.split(':', -1)[-1]
        print("------Begin------")
    time.sleep(0.02)

async def turn(self):
    i = 0
    target = 275.0
    while i == 0:
        r = self.r
        if float(r) < target:
            control_command = f"{1550}:{1550}:{1462}:{1462}:{0}"
            await self.ws.send(control_command)
            # print(control_command)
            data = await self.ws.recv()
            self.r = data.split(':', -1)[-1]
            print(r)
        elif float(r) > target:
            control_command = f"{1400}:{1400}:{1462}:{1462}:{0}"
            await self.ws.send(control_command)
            data = await self.ws.recv()
            self.r = data.split(':', -1)[-1]
            print(r)
        elif float(r) == target:
            i = 1
    print("------Finish Turning------")
    time.sleep(0.02)

async def stop(self):
    control_command = f"{1462}:{1462}:{1462}:{1462}:{0}"
    await self.ws.send(control_command)
    data = await self.ws.recv()
    print(data)
    time.sleep(0.02)

async def forward(self):
    while 1:
        control_command = f"{1562}:{1362}:{1462}:{1462}:{0}"
        await self.ws.send(control_command)
        data = await self.ws.recv()
        if keyboard.is_pressed('q'):
            break
    time.sleep(0.02)
```

### 自动行进模块

```c++
async def cv_control(self):
    global prev_error, integral, target_found
    # 摄像头视频流地址
    camera_url = "http://192.168.137.100:8080/?action=stream"
    cap = cv2.VideoCapture(camera_url)
    prev_error = 0
    integral = 0

    target_found = True
    right_direction = False
    control_command = f"{1462}:{1462}:{1462}:{1462}:{0}"
    await self.ws.send(control_command)
    data = await self.ws.recv()
    self.r = float(data.split(':', -1)[-1])
    while target_found:
        target_yaw = 275.0
        if not right_direction:
            if abs(float(self.r) - float(target_yaw)) < 3:  # 调整的角度容忍范围
                # 开始前进
                print("------right------")
                control_command = f"{1600}:{1300}:{1462}:{1462}:{0}"
                self.cv_config_video()
                await self.ws.send(control_command)
                data = await self.ws.recv()
                self.r = data.split(':', -1)[-1]
            else:
                # 转向
                await self.turn()
                self.cv_config_video()
```

整体来看，本项目相对完善但还有许多不足的地方，比如机器人的自动行进模块不太智能，作为一个选修课的课设，本项目达到了基本要求。但是并未具备应用到实战的能力，有待继续开发。