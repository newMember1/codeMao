## 从shadow mapping说起  
最基本的shadow map工作流可以看成两步：  
1.将model的坐标变换到light坐标系下，将该坐标系下的model坐标作为depth，记录到深度贴图shadowMap中，需要注意的是在opengl中，深度gl_FragDepth = gl_FragCoord.z会默认进行配置和更新，保证其中的深度值都是最小的。  
2.进行第二遍渲染，将视角变换为camera视角，同时对每个要进行渲染的三角形(fragment)进行判断，如果该fragment在light坐标系下的深度大于shadowMap中对应的深度，说明该片段是不可见的，记可见强度为0。否则可见并记可见强度为1。  
渲染结果如下：  
![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_shadows.png)  

存在明显的问题，阴影失真(shadow acene),即在距离光源较远的情况下，shadowmap的分辨率不够造成的
![img](https://pic4.zhimg.com/80/88ffd93a4fa8cc66d31ae25bf2d00366_hd.jpg)  
如上图所示，由于分辨率不够，四个像素共用一个深度值，那么在计算这四个像素点的深度时会出现a，d被点亮，b，c阴影的现象。  
解决方法可以通过添加一个小的bias来使得深度图中保存的深度值加上一个偏移值来进行比较。但是如果在坡度较大的时候该方法的偏移值难以确定，可以通过法向量来设置bias。结果如下图：  
![img](https://learnopengl-cn.github.io/img/05/03/01/shadow_mapping_with_bias.png)
还可能存在悬浮的现象，可以使用正面剔除来解决。  

第二个明显的问题，由于深度图有一个固定的分辨率，容易产生锯齿边。

## PCF  
PCF(percent-closer filtering)，是一种多个不同过滤方式的组合,最原始的PCF算法是通过随机选取微小四边形中的某个点的深度来模糊边缘，目前更一般的PCF是通过一个卷积核来实现shadow map的平滑。  

## variance shadow mapping  
方差阴影贴图是为了改进PCF无法提前进行滤波处理的一种方法，用两通道shadow map保存实际的阴影深度和深度的平方值。从而可以高效的找出任意区域的方差。得到方差以后，可以得到一个被遮挡阴影片段的上界。这个上界通常对实际的遮挡效果有着很好的逼近。  
**方差可以很好的描述一个分布(这是基于中心极限定理得来的)。**
