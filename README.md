详细参考：[https://blog.csdn.net/NoamaNelson/article/details/103188081](https://blog.csdn.net/NoamaNelson/article/details/103188081)
## 1、安装python的PIL图像处理库 
[https://blog.csdn.net/NoamaNelson/article/details/103179475](https://blog.csdn.net/NoamaNelson/article/details/103179475 "安装方法，点击此处：Win7 64位下Python安装PIL图像处理库") 
## 2、需要安装摄像机驱动 


1. 进入大恒官网  
官网地址：[http://www.daheng-imaging.com/index.aspx](http://www.daheng-imaging.com/index.aspx) 
![](https://img-blog.csdnimg.cn/20191121180309847.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05vYW1hTmVsc29u,size_16,color_FFFFFF,t_70) 
2. 点击注册，填写信息注册成功后，点击下载中心，找到自己使用的摄像头，以及对应的系统，进行驱动下载安装即可 
![](https://img-blog.csdnimg.cn/20191121180722176.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05vYW1hTmVsc29u,size_16,color_FFFFFF,t_70) 
3. 直接在驱动安装路径下，找到Python SDK，然后直接在对应的目录下写脚本即可。 
![](https://img-blog.csdnimg.cn/20191121180852169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05vYW1hTmVsc29u,size_16,color_FFFFFF,t_70) 
4. 对部分常用参数进行封装 
![](https://img-blog.csdnimg.cn/20191121181027227.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05vYW1hTmVsc29u,size_16,color_FFFFFF,t_70) 
## 3、实现的脚本如下 
    `import gxipy as gx
	from PIL import Image
	import datetime

	"""
	Author:NoamaNelson
	Date:2019-11-21
	Discription:Secondary development of pythonsdk of Daheng camera.
	"""

	def main():
    
	    Width_set = 640 # 设置分辨率宽
	    Height_set = 480 # 设置分辨率高
	    framerate_set = 80 # 设置帧率
	    num = 500 # 采集帧率次数（为调试用，可把后边的图像采集设置成while循环，进行无限制循环采集）
	    
	    #打印
	    print("")
	    print("###############################################################")
	    print("               连续获取彩色图像并显示获取的图像.")
	    print("###############################################################")
	    print("")
	    print("摄像机初始化......")
	    print("")
	 
	    #创建设备
	    device_manager = gx.DeviceManager() # 创建设备对象
	    dev_num, dev_info_list = device_manager.update_device_list() #枚举设备，即枚举所有可用的设备
	    if dev_num is 0:
	        print("Number of enumerated devices is 0")
	        return
	    else:
	        print("")
	        print("**********************************************************")
	        print("创建设备成功，设备号为:%d" % dev_num)
	
	    #通过设备序列号打开一个设备
	    cam = device_manager.open_device_by_sn(dev_info_list[0].get("sn"))
	
	    #如果是黑白相机
	    if cam.PixelColorFilter.is_implemented() is False: # is_implemented判断枚举型属性参数是否已实现
	        print("该示例不支持黑白相机.")
	        cam.close_device()
	        return
	    else:
	        print("")
	        print("**********************************************************")
	        print("打开彩色摄像机成功，SN号为：%s" % dev_info_list[0].get("sn"))
	
	
	    #设置宽和高
	    cam.Width.set(Width_set)
	    cam.Height.set(Height_set)
	    
	    #设置连续采集
	    #cam.TriggerMode.set(gx.GxSwitchEntry.OFF) # 设置触发模式
	    cam.AcquisitionFrameRateMode.set(gx.GxSwitchEntry.ON)
	
	    #设置帧率
	    cam.AcquisitionFrameRate.set(framerate_set)
	    print("")
	    print("**********************************************************")
	    print("用户设置的帧率为:%d fps"%framerate_set)
	    framerate_get = cam.CurrentAcquisitionFrameRate.get() #获取当前采集的帧率
	    print("当前采集的帧率为:%d fps"%framerate_get)
	
	
	     #开始数据采集
	    print("")
	    print("**********************************************************")
	    print("开始数据采集......")
	    print("")
	    cam.stream_on()
	
	    #采集图像
	    for i in range(num):
	        raw_image = cam.data_stream[0].get_image() # 打开第0通道数据流
	        if raw_image is None:
	            print("获取彩色原始图像失败.")
	            continue
	
	        rgb_image = raw_image.convert("RGB") # 从彩色原始图像获取RGB图像
	        if rgb_image is None:
	            continue
	
	        #rgb_image.image_improvement(color_correction_param, contrast_lut, gamma_lut)  # 实现图像增强
	
	        numpy_image = rgb_image.get_numpy_array() # 从RGB图像数据创建numpy数组
	        if numpy_image is None:
	            continue
	
	        img = Image.fromarray(numpy_image, 'RGB') # 展示获取的图像
	        #img.show()
	        mtime = datetime.datetime.now().strftime('%Y-%m-%d_%H_%M_%S')
	    
	        img.save(r"D:\image\\" + str(i) + str("-") + mtime + ".jpg") # 保存图片到本地
	
	        print("Frame ID: %d   Height: %d   Width: %d   framerate_set:%dfps   framerate_get:%dfps"
	              % (raw_image.get_frame_id(), raw_image.get_height(), raw_image.get_width(), framerate_set, framerate_get)) # 打印采集的图像的高度、宽度、帧ID、用户设置的帧率、当前采集到的帧率
	         
	    #停止采集
	    print("")
	    print("**********************************************************")
	    print("摄像机已经停止采集")
	    cam.stream_off()
	
	    #关闭设备
	    print("")
	    print("**********************************************************")
	    print("系统提示您：设备已经关闭！")
	    cam.close_device()

	if __name__ == "__main__":
    	main()

` 
## 4、运行结果 
![](https://img-blog.csdnimg.cn/20191121185239727.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05vYW1hTmVsc29u,size_16,color_FFFFFF,t_70) 
![](https://img-blog.csdnimg.cn/20191121185303588.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L05vYW1hTmVsc29u,size_16,color_FFFFFF,t_70)