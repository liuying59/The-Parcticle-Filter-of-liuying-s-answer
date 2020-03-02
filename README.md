
一. 对particle filtering的理解
particle filtering算法在robot定位中的应用，其主要步骤包括：生成—预测—更新—重采样，其中修改粒子权值的update步骤最为重要，经过不断迭代，淘汰权值较低的粒子，最终可求得所剩粒子的平均值，即为待求robot的位置。

二.原码的三个改造及结果
1.计算robot的唯一位置：
    计算思路:利用particles中的大量高权值粒子，求平均值，并将其作为robot的坐标（直接调用estimate函数得到平均坐标，并将其打印）
    添加代码:
	mean,var = estimate(particles, weights)#可置于mouceCallback函数中                                
	#通过调用estimate函数，计算出robot的坐标，即particles的平均坐标     
	cv2.circle(img,tuple((int(mean[0]),int(mean[1]))),5,(255,0,255),-1)   
	#将计算出的robot坐标mean以酒红色打印出来                                                                        



2.修改weights的分布为帕累托分布：
	修改思路:修改weights服从的高斯分布为帕累托分布，利用pareto函数直接修改
	添加代码：
#weights *= scipy.stats.norm(distance, R).pdf(z[i])                                   
weights *= scipy.stats.pareto(distance, R).pdf(z[i])                
#利用pareto函数替换norm函数，即可将weights的分布修改为pareto分布              
	运行结果图：












        图中显示，将weights改为帕累托分布后，因帕累托分布与实际weights分布不相符，导致拟合效果较差

3为landmark和robot之间的距离增加随机误差，观察定位结果
思路：利用numpy.random.randn函数产生服从高斯分布的随机误差，并将其加入landmark与robot之间距离中
代码:        
zs = (np.linalg.norm((landmarks - center), axis=1) +                            (np.random.randn(NL) * sensor_std_err))#sensor_std_err改变随机误差       
运行结果图：
当sensor_std_err为5时，结果为左图，预测点与实际鼠标位置重叠
当sensor_std_err为50时，结果为右图，预测点与实际鼠标位置差别较大











4.修改particle filtering过程的思路：
1.增加生成的微粒数量，在增加计算量的同时，减小误差对定位的影响
2.在预测微粒状态时，引入与robot同分布的随机误差




