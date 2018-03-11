 # Tensorflow note

## 基本用法
* 起始节点
```python
c = tf.Constant(tf.int32, shape=[None, 1], name="Constant")
v = tf.Variable(...)    # 运行中被更新
p = tf.placeholder(...)
```

* shape
```python
[None, 784]   # None代表第一纬度不确定，视输入而定, 用于不定batch输入
tf.reshape(x, [-1, 28, 28, 1])  # 第一个纬度根据计算得到。
```
* operation
```python
tf.matmul() #矩阵乘法
tf.square()
tf.log()
tf.reduce_sum()
tf.arg_max()    
tf.equal()
tf.assign()
```
* 初始化
1、 简单初始化
```python
# tf.initialize_all_variables() 已经被废弃
init = tf.global_variables_initializer()    # 在session中,可以直接init.run()
sess.run(init)
```
2、 复杂初始化
```
# 截断正态分布
tf.Variable(tf.truncated_normal(shape, stddev=0.1))
tf.randint(-1, 1)
tf.uniform_random()
```
* 损失
```python
# 交叉损失熵
# softmax is used to normalize all dimesion.
y = tf.nn.softmax(tf.matmul(x, W) + b)
cross_entropy = -tf.reduce_sum(y_ * tf.log(y))  # y_hat(label) multiply log(y)
```
* 优化
```python
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)
_, loss = sess.run([train, loss], feed_dict={x: xx})
print(loss)     # 运行中查看损失    
```

* 评估模型
```python
tf.argmax(y, 1) # 在第二维度找最大值变为1， 如softmax后，每个纬度都有取值，找到最大值变为1， 第一维度用于batch

# 训练结果与标签值每个判断是否相等，得到的correct的形式类似于
# [True, True, False, True]
correct = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
# 转换为准确率
# cast用于把True变为1， False变为0
# reduce_mean取平均值
accuracy = tf.reduce_mean(tf.cast(correct, "float"))
# 运行accuracy 的Graph
a = sess.run(accuracy, feed_dict={test_dict})
```

* Session 和 Graph
```
sess = tf.InteractiveSession() 
直接成为默认的Session, 可以直接用op.run() 或 tensor.eval()
结束时必须使用 sess.close()
```
1、 正常情况下
```python
with tf.Session() as sess:
    sess.run(...)
```
2、 默认session
```python
sess = tf.Session()
with sess.as_default():
    c.eval()
# 尽管调用完毕，但sess仍是默认Session()，必须显示调用sess.close()
```
## 保存和加载变量
```python
saver = tf.train.Saver()
saver.save(self.sess, "./model.ckpt", global_step=1)    # 目录./是很必要的，或者用os.path.join(...)

# 加载
self.saver.restore(self.sess, "./model.ckpt-1")     # -1 是保存中指定的global_step
```
* 注：变量的name不能改变，在保存和加载的过程中。

## 模型
* CNN
```python
# conv2d代表二维卷积网络(如图片)，需要输入四个纬度的元素, 第一纬度为batch_size（无意义）, 第二、三分别维宽和高，第四个纬度为每个元素的通道数，如黑白为1， 彩色为3.
# stride为步长， 重点关注第二第三纬度。
# padding为SAME： 输出与输入的形状(shape)相同
# padding为VALID： 输出与输入通过公式计算: width= (28-5+2*0/1)+1=24, 
# 卷积层
tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding="SAME")

# ksize代表卷积核的大小，本例为2*2,步长分别为2, 2
# 输出的大小为width/=2, height/=2
# 池化层
tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding="SAME")

h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)
h_pool1 = max_pool(h_conv1)

W_conv2 = ...
```

## tensorboard
```python
# 设置图标中记录的名称和变量值。
tf.summary.scalar('loss', cross_entropy)
# merge所有的操作符
merged_summary_op = tf.summary.merge_all()
# 定义写入目录。
summary_writer = tf.summary.FileWriter("/tmp/mnist_logs", sess.graph)
# 运行merge操作符并写入。
summary_str = sess.run(merged_summary_op, feed_dict={x: batch_xs, y_: batch_ys})
summary_writer.add_summary(summary_str, i)
```
启动
```
tensorboard --logdir /tmp/mnist_logs
```

## scope(variable/name)
- variable_scope
It is used to share variable by `get_variable`. 
```python
with tf.variable_scope('v_scope') as scope1:
    Weights1 = tf.get_variable('Weights', shape=[2,3])

# 下面来共享上面已经定义好的变量
with tf.variable_scope('v_scope', reuse=True) as scope2:
    Weights2 = tf.get_variable('Weights')
```
- name_scope
It is mainly used to create diffent variable with same name. `It will create a new name_scope, if the name has been existed.`e.g. `conv1_1`.
Usually every layer is a name_scope.
```python
with tf.name_scope('conv1') as scope:
    weights1 = tf.Variable([1.0, 2.0], name='weights')
    bias1 = tf.Variable([0.3], name='bias')

with tf.name_scope('conv2') as scope:
    weights2 = tf.Variable([4.0, 2.0], name='weights')
    bias2 = tf.Variable([0.33], name='bias')
```