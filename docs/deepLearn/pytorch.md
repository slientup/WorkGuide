#### 神经网络
![](https://files.mdnice.com/user/4251/e093d40a-6e66-43db-90b5-0bf14ab7d5d6.png)

x是输入，y是输出，当输入x经过神经网络后会得到y，我们要做的就是通过大量数据训练出这个神经网络模型。

当输入数据x经过第一层forward(激活函数)会得到一个x1，x1会作为第二层激活函数的`input`，依次类推直到输出y1，当我们得到`y1`的值时，需要与我们期望值一比较，肯定会有差异，当有差异后，我们就需要修改调整参数，那这个参数应该调整到多少啦，这又涉及到一个算法，然后再根据调整后的参数重新跑神经网络。

1. 激活函数，正向传播的过程需要激活函数计算对应的输出值

2. 误差函数，期望值和实际值是有差异的，所以我们需要一个**误差函数**，如果无差函数认为没问题了，就不再重新修正参数。

3. **优化器和梯度**，参数需要修正，那这个值应该修正到多少？这需要一个算法，算法里面需要梯度，梯度通过反向传播计算出来。对反向传播的步骤而言，就是对正向传播的一系列的反向迭代，通过反向计算梯度，来优化我们需要训练的$W$和$b$。

**涉及概念：**
1. 数据集
2. 神经网络模型
3. 损失函数
4. 优化器

**激活函数列表**
-  sigmoid 函数
-  tanh 函数
-  ReLU 函数
-  Leaky Relu 函数

**优化器列表**
- SGD、Nesterov-SGD、Adam、RMSPROP等，PyTorch中构建了一个包torch.optim实现了所有的这些规则

#### 实现步骤

1. 提供大量的数据，包括训练集和测试集
2. 定义神经网络
3. 定义损失函数
4. 定义优化器(参数修正规则)
5. 在训练集上训练网络
6. 在测试集上测试网络

1. 定义神经网络
```
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.pool = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x
net = Net()
```
2. 定义损失函数和优化器
```
import torch.optim as optim

criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)  // 使用SGD的优化器
```

3. 训练网络 
我们只需在数据迭代器上循环，将数据输入给网络，并优化
```
for epoch in range(2):  # 多批次循环

    running_loss = 0.0
    for i, data in enumerate(trainloader, 0):
        # 获取输入
        inputs, labels = data
        # 梯度置0 在调用前需要清除已存在的梯度，否则梯度将被累加到已存在的梯度  
        optimizer.zero_grad()
        # 正向传播，反向传播，优化
        outputs = net(inputs)
        loss = criterion(outputs, labels) (误差值)
        loss.backward() # 反向传播获得对应的梯度
        optimizer.step() # Does the update  更新参数
        # 打印状态信息
        running_loss += loss.item() 
        if i % 2000 == 1999:    # 每2000批次打印一次
            print('[%d, %5d] loss: %.3f' %
                  (epoch + 1, i + 1, running_loss / 2000))
            running_loss = 0.0
print('Finished Training')
```
#### 参考连接

- [训练一个分类器](https://github.com/zergtant/pytorch-handbook/blob/master/chapter1/4_cifar10_tutorial.ipynb)
- [神经网络](https://github.com/zergtant/pytorch-handbook/blob/master/chapter1/3_neural_networks_tutorial.ipynb)
- [logistic回归实战](https://github.com/zergtant/pytorch-handbook/blob/master/chapter3/3.1-logistic-regression.ipynb)
