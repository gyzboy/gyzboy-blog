---
title: "openGL"
date: 2021-04-13T16:12:16+08:00
---
{{< toc >}}

## 基本概念
### 着色器
着色器（Shader）是在GPU上运行的小程序
### 顶点着色器
把每个顶点在虚拟空间中的三维坐标变换为可以在屏幕上显示的二维坐标，并带有用于z-buffer的深度信息。顶点着色器可以操作的属性有：位置、颜色、纹理坐标，但是不能创建新的顶点
### 片元着色器
片元着色器计算每个像素的颜色和其它属性。它通过应用光照值、凹凸贴图，阴影，镜面高光，半透明等处理来计算像素的颜色并输出。它也可改变像素的深度(z-buffering)或在多个渲染目标被激活的状态下输出多种颜色。一个片元着色器不能产生复杂的效果，因为它只在一个像素上进行操作，而不知道场景的几何形状
### 坐标系
OpenGL ES采用的是右手坐标，选取屏幕中心为原点，从原点到屏幕边缘默认长度为1，也就是说默认情况下，从原点到（1,0,0）的距离和到（0,1,0）的距离在屏幕上展示的并不相同。即向右为X正轴方向，向左为X负轴方向，向上为Y轴正轴方向，向下为Y轴负轴方向，屏幕面垂直向上为Z轴正轴方向，垂直向下为Z轴负轴方向
### 图形的绘制
OpenGL ES2.0的世界里面只有点、线、三角形，其它更为复杂的几何形状都是由三角形构成的
### 投影
OpenGL ES中有两种投影方式：正交投影和透视投影。正交投影，物体不会随距离观测点的位置而大小发生变化。而透视投影，距离观测点越远，物体越小，距离观测点越近，物体越大
### 光照
光照由三种元素组成（也可以说是三种通道组成），分别为环境光、镜面光及散射光
* 环境光是指从四面八方照射到物体上，其具体公式为：
    > 环境光照射结果=材质反射系数∗环境光强度
* 散射光是指现实世界中组草的物体表面被光照射时，反射光在各个方向基本均匀的情况，其具体公式为：
    > 散射光照射结果=材质的反射系数∗散射光强度∗max(cos(入射角)，0)
* 镜面光是指现实世界中光滑的表面被照射时会有方向很集中的反射光，与散射光最终强度依赖于入射光与被照射点的法向量夹角不同，镜面光的强度还依赖于观察者的位置，具体公式如下：
    > 镜面光照射结果=材质的反射系数∗镜面光强度∗max(0,cos(半向量与法向量的夹角)粗糙度)
### 纹理映射
纹理映射是将2D的纹理映射到3D场景中的立体物体上
### 渲染流程
读取顶点数据——执行顶点着色器——组装图元——光栅化图元——执行片元着色器——写入帧缓冲区——显示到屏幕上
## 绘制一个图形的步骤
### 1.绑定GLSurfaceView
```java
setEGLContextClientVersion(2);
setRenderer(renderer);//绑定Render
setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
```
### 2.创建shape并加载
#### 2.1 准备顶点着色器、片元着色器
```java
private final String vertexShaderCode =
            "attribute vec4 vPosition;" +
                    "void main() {" +
                    "  gl_Position = vPosition;" +
                    "}";

private final String fragmentShaderCode =
            "precision mediump float;" +
                    "uniform vec4 vColor;" +
                    "void main() {" +
                    "  gl_FragColor = vColor;" +
                    "}";
```
#### 2.2 创建OpenGL程序并加载着色器
```java
    private int mProgram;
    int vertexShader = loadShader(GLES20.GL_VERTEX_SHADER,
                vertexShaderCode);
    int fragmentShader = loadShader(GLES20.GL_FRAGMENT_SHADER,
                fragmentShaderCode);

    //创建一个空的OpenGLES程序
    mProgram = GLES20.glCreateProgram();
    //将顶点着色器加入到程序
    GLES20.glAttachShader(mProgram, vertexShader);
    //将片元着色器加入到程序中
    GLES20.glAttachShader(mProgram, fragmentShader);
    //连接到着色器程序
    GLES20.glLinkProgram(mProgram);

    public int loadShader(int type, String shaderCode){
        //根据type创建顶点着色器或者片元着色器
        int shader = GLES20.glCreateShader(type);
        //将资源加入到着色器中，并编译
        GLES20.glShaderSource(shader, shaderCode);
        GLES20.glCompileShader(shader);
        return shader;
    }
```
#### 2.3 确定需要绘制图形的坐标和颜色数据
```java
    private FloatBuffer vertexBuffer;
    static float triangleCoords[] = {
            0.5f,  0.5f, 0.0f, // top
            -0.5f, -0.5f, 0.0f, // bottom left
            0.5f, -0.5f, 0.0f  // bottom right
    };
    ByteBuffer bb = ByteBuffer.allocateDirect(
                triangleCoords.length * 4);
    bb.order(ByteOrder.nativeOrder());
    vertexBuffer = bb.asFloatBuffer();
    vertexBuffer.put(triangleCoords);
    vertexBuffer.position(0);
```
#### 2.4 设置视图窗口(viewport)。
```java
    @Override
    public void onSurfaceChanged(GL10 gl, int width, int height) {
        GLES20.glViewport(0,0,width,height);
    }
```
#### 2.5 将坐标数据颜色数据传入OpenGL ES程序中,并绘制缓冲区内容
```java
    @Override
    public void onDrawFrame(GL10 gl) {
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT|GLES20.GL_DEPTH_BUFFER_BIT);
        //将程序加入到OpenGLES2.0环境
        GLES20.glUseProgram(mProgram);

        //获取顶点着色器的vPosition成员句柄
        mPositionHandle = GLES20.glGetAttribLocation(mProgram, "vPosition");
        //启用三角形顶点的句柄
        GLES20.glEnableVertexAttribArray(mPositionHandle);
        //准备三角形的坐标数据
        GLES20.glVertexAttribPointer(mPositionHandle, COORDS_PER_VERTEX,
                GLES20.GL_FLOAT, false,
                vertexStride, vertexBuffer);
        //获取片元着色器的vColor成员的句柄
        mColorHandle = GLES20.glGetUniformLocation(mProgram, "vColor");
        //设置绘制三角形的颜色
        GLES20.glUniform4fv(mColorHandle, 1, color, 0);
        //绘制三角形
        GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);
        //禁止顶点数组的句柄
        GLES20.glDisableVertexAttribArray(mPositionHandle);
    }   
```