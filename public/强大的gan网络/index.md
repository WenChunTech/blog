# 强大的GAN网络


<!--more-->

# 强大的GAN网络

## 1. 概要

​	GAN网络全称**generative adversarial network**,翻译为生成式对抗网络,是一种非监督式学习机器学习方法。由`Ian J,Goodfello` 等人于2014年在[Generative Adversarial Nets](https://arxiv.org/pdf/1406.2661.pdf) 论文中提出。其中在GAN网络中,有两个模型——生成模型(generative model G),判别模型(discriminative model D).

## 2. 原理

​	GAN网络主要运用了博弈论的思想,模型中的2为博弈方分别由生成模型和判别模型担当.生成模型用随机取样作为输入,它的输出结果要尽可能和训练样本尽可能相似,最好的情况就是分辨不出是真实样本还是生成出来的样本.而判别模型就是尽可能判别生成模型生成的结果和真实样本.这样2个网络相互对抗,不断调整参数,最终达到纳什均衡.

这个过程可以表示为:
$$
min_G max_DV(D,G) = \Epsilon_{x\sim P_{data}(x)}[logD(x)] + \Epsilon_{z\sim p_{z}(z)}[log(1-D(G(z)))]
$$
公式解释:

	1. 当训练D时,希望这个式子的值越大越好.真实数据希望被D分成1,生成数据希望被分成0
	2. 当训练G时,希望这个式子的值越小越好.希望D分不开真实数据还是生成数据

>  零和博弈（zero-sum game），又称零和游戏，与非零和博弈相对，是博弈论的一个概念，属非合作博弈。指参与博弈的各方，在严格竞争下，一方的收益必然意味着另一方的损失，博弈各方的收益和损失相加总和永远为“零”，双方不存在合作的可能。就像下棋的游戏一样，你走的每一步和对方走的每一步都是向着对自己有利的方向走，然后你和对手轮流走步
> 每一步都向着自己最大可能能赢的地方走。这就是零和博弈。



## 3. 简单代码实现

### 3.1 导包

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.utils.data import DataLoader
from torch import optim
import torchvision
from torchvision import transforms
from torch.utils.tensorboard import SummaryWriter
import matplotlib.pyplot as plt
```

### 3.2 加载数据集

```python
# 初始化tensorboard数据保存路径
writer = SummaryWriter('./logs')
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
batch_size = 32

transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize(mean=(0.5, ), std=(0.5, ))])

train_dataset = torchvision.datasets.MNIST(root='./data', train=True, download=False, transform=transform)
train_loader = DataLoader(dataset=train_dataset, batch_size=batch_size, shuffle=True, drop_last=True)
```



### 3.3 定义生成模型

```python
class Generator(nn.Module):
    def __init__(self, g_input_dim, g_output_dim):
        super(Generator, self).__init__()       
        self.fc1 = nn.Linear(g_input_dim, 256)
        self.fc2 = nn.Linear(self.fc1.out_features, self.fc1.out_features*2)
        self.fc3 = nn.Linear(self.fc2.out_features, self.fc2.out_features*2)
        self.fc4 = nn.Linear(self.fc3.out_features, g_output_dim)
    
    # forward method
    def forward(self, x): 
        x = F.leaky_relu(self.fc1(x), 0.2)
        x = F.leaky_relu(self.fc2(x), 0.2)
        x = F.leaky_relu(self.fc3(x), 0.2)
        return torch.tanh(self.fc4(x))
```



### 3.4 定义判别模型

```python
class Discriminator(nn.Module):
    def __init__(self, d_input_dim):
        super(Discriminator, self).__init__()
        self.fc1 = nn.Linear(d_input_dim, 1024)
        self.fc2 = nn.Linear(self.fc1.out_features, self.fc1.out_features//2)
        self.fc3 = nn.Linear(self.fc2.out_features, self.fc2.out_features//2)
        self.fc4 = nn.Linear(self.fc3.out_features, 1)
    
    # forward method
    def forward(self, x):
        x = F.leaky_relu(self.fc1(x), 0.2)
        x = F.dropout(x, 0.3)
        x = F.leaky_relu(self.fc2(x), 0.2)
        x = F.dropout(x, 0.3)
        x = F.leaky_relu(self.fc3(x), 0.2)
        x = F.dropout(x, 0.3)
        return torch.sigmoid(self.fc4(x))
```



### 3.5 构造模型,定义损失和优化器

```python
z_dim = 100
mnist_dim = train_dataset.train_data.size(1) * train_dataset.train_data.size(2)

# build network
G = Generator(g_input_dim = z_dim, g_output_dim = mnist_dim).to(device)
D = Discriminator(mnist_dim).to(device)

writer.add_graph(G, input_to_model=torch.randn(batch_size, z_dim))
writer.add_graph(D, input_to_model=torch.randn(batch_size, mnist_dim))

# optimizer
lr = 0.0002
g_optimizer = optim.Adam(G.parameters(), lr = lr)
d_optimizer = optim.Adam(D.parameters(), lr = lr)

# loss
criterion = nn.BCELoss() 
```

### 3.6 训练判别器和生成器

```python
def d_train(x):
    D.zero_grad()

    x_real, y_real = x.view(-1, mnist_dim).to(device), torch.ones(batch_size, 1).to(device)

    print(x_real.shape, y_real.shape)
    d_output = D(x_real)

    print(d_output.shape, y_real.shape)
    d_real_loss = criterion(d_output, y_real)
    d_real_score = d_output

    z = torch.randn(batch_size, z_dim).to(device)
    x_fake, y_fake = G(z), torch.zeros(batch_size, 1).to(device)

    d_output = D(x_fake)
    d_fake_loss = criterion(d_output, y_fake)
    d_fake_score = d_output

    d_loss = d_real_loss + d_fake_loss
    d_loss.backward()
    d_optimizer.step()

    return d_loss.item()


def g_train(x):
    G.zero_grad()
    z = torch.randn(batch_size, z_dim).to(device)
    y = torch.ones(batch_size, 1).to(device)

    g_output = G(z)
    d_output =  D(g_output)
    g_loss = criterion(d_output, y)

    g_loss.backward()
    g_optimizer.step()

    return g_loss.item()
```

### 3.7 训练网络

```python
epochs = 10
step = 0
for epoch in range(epochs):
    d_losses, g_losses = [], []
    for batch_idx, (x, _) in enumerate(train_loader):
        step += 1
        d_losses.append(d_train(x))
        g_losses.append(g_train(x))
        print('[%d/%d]: [%d/%d]: loss_d: %.3f, loss_g: %.3f' % (
        epoch, epochs,batch_idx, len(train_loader), torch.mean(torch.FloatTensor(d_losses)), torch.mean(torch.FloatTensor(g_losses))))
        writer.add_scalar('g_loss', torch.mean(torch.FloatTensor(g_losses)), step)
        writer.add_scalar('d_loss', torch.mean(torch.FloatTensor(d_losses)), step)
        if batch_idx % 10 == 0:
            with torch.no_grad():
                test_z = torch.randn(batch_size, z_dim).to(device)
                generated = G(test_z)
                img = img = torchvision.utils.make_grid(generated.view(generated.size(0), 1, 28, 28))
                writer.add_image(f'mnist_{epoch}_{batch_idx}', img, global_step=step)

writer.close()                
```

### 3.8 保存模型

```python
torch.save(D, './model/discriminator.pt')
torch.save(G, './model/generator.pt')
```

### 3.9 汇总代码

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from torch.utils.data import DataLoader
from torch import optim
import torchvision
from torchvision import transforms
from torchinfo import summary
from torch.utils.tensorboard import SummaryWriter
import matplotlib.pyplot as plt


writer = SummaryWriter('./logs')
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
batch_size = 32

transform = transforms.Compose([transforms.ToTensor(), transforms.Normalize(mean=(0.5, ), std=(0.5, ))])

train_dataset = torchvision.datasets.MNIST(root='./data', train=True, download=False, transform=transform)
# 设置drop_last丢弃最后不满一个batch_size的数据
train_loader = DataLoader(dataset=train_dataset, batch_size=batch_size, shuffle=True, drop_last=True)

class Generator(nn.Module):
    def __init__(self, g_input_dim, g_output_dim):
        super(Generator, self).__init__()       
        self.fc1 = nn.Linear(g_input_dim, 256)
        self.fc2 = nn.Linear(self.fc1.out_features, self.fc1.out_features*2)
        self.fc3 = nn.Linear(self.fc2.out_features, self.fc2.out_features*2)
        self.fc4 = nn.Linear(self.fc3.out_features, g_output_dim)
    
    # forward method
    def forward(self, x): 
        x = F.leaky_relu(self.fc1(x), 0.2)
        x = F.leaky_relu(self.fc2(x), 0.2)
        x = F.leaky_relu(self.fc3(x), 0.2)
        return torch.tanh(self.fc4(x))
    
class Discriminator(nn.Module):
    def __init__(self, d_input_dim):
        super(Discriminator, self).__init__()
        self.fc1 = nn.Linear(d_input_dim, 1024)
        self.fc2 = nn.Linear(self.fc1.out_features, self.fc1.out_features//2)
        self.fc3 = nn.Linear(self.fc2.out_features, self.fc2.out_features//2)
        self.fc4 = nn.Linear(self.fc3.out_features, 1)
    
    # forward method
    def forward(self, x):
        x = F.leaky_relu(self.fc1(x), 0.2)
        x = F.dropout(x, 0.3)
        x = F.leaky_relu(self.fc2(x), 0.2)
        x = F.dropout(x, 0.3)
        x = F.leaky_relu(self.fc3(x), 0.2)
        x = F.dropout(x, 0.3)
        return torch.sigmoid(self.fc4(x))
    
   
z_dim = 100
mnist_dim = train_dataset.train_data.size(1) * train_dataset.train_data.size(2)

# build network
G = Generator(g_input_dim = z_dim, g_output_dim = mnist_dim).to(device)
D = Discriminator(mnist_dim).to(device)

# 添加网络图到tensorboard
writer.add_graph(G, input_to_model=torch.randn(batch_size, z_dim))
writer.add_graph(D, input_to_model=torch.randn(batch_size, mnist_dim))

# optimizer
lr = 0.0002
g_optimizer = optim.Adam(G.parameters(), lr = lr)
d_optimizer = optim.Adam(D.parameters(), lr = lr)

# loss
criterion = nn.BCELoss() 


def d_train(x):
    D.zero_grad()

    x_real, y_real = x.view(-1, mnist_dim).to(device), torch.ones(batch_size, 1).to(device)

    print(x_real.shape, y_real.shape)
    d_output = D(x_real)

    print(d_output.shape, y_real.shape)
    d_real_loss = criterion(d_output, y_real)
    d_real_score = d_output

    z = torch.randn(batch_size, z_dim).to(device)
    x_fake, y_fake = G(z), torch.zeros(batch_size, 1).to(device)

    d_output = D(x_fake)
    d_fake_loss = criterion(d_output, y_fake)
    d_fake_score = d_output

    d_loss = d_real_loss + d_fake_loss
    d_loss.backward()
    d_optimizer.step()

    return d_loss.item()


def g_train(x):
    G.zero_grad()
    z = torch.randn(batch_size, z_dim).to(device)
    y = torch.ones(batch_size, 1).to(device)

    g_output = G(z)
    d_output =  D(g_output)
    g_loss = criterion(d_output, y)

    g_loss.backward()
    g_optimizer.step()

    return g_loss.item()


epochs = 10
step = 0
for epoch in range(epochs):
    d_losses, g_losses = [], []
    for batch_idx, (x, _) in enumerate(train_loader):
        step += 1
        d_losses.append(d_train(x))
        g_losses.append(g_train(x))
        print('[%d/%d]: [%d/%d]: loss_d: %.3f, loss_g: %.3f' % (
        epoch, epochs,batch_idx, len(train_loader), torch.mean(torch.FloatTensor(d_losses)), torch.mean(torch.FloatTensor(g_losses))))
        writer.add_scalar('g_loss', torch.mean(torch.FloatTensor(g_losses)), step)
        writer.add_scalar('d_loss', torch.mean(torch.FloatTensor(d_losses)), step)
        if batch_idx % 10 == 0:
            with torch.no_grad():
                test_z = torch.randn(batch_size, z_dim).to(device)
                generated = G(test_z)
                img = img = torchvision.utils.make_grid(generated.view(generated.size(0), 1, 28, 28))
                writer.add_image(f'mnist_{epoch}_{batch_idx}', img, global_step=step)
    if epoch % 10 == 0:
        D.eval()
        G.eval()
        torch.save({
        'epoch': epoch,
        'd_model_state_dict': D.state_dict(),
        'g_model_state_dict': G.state_dict(),
        'd_optimizer_state_dict': d_optimizer.state_dict(),
        'd_loss': d_losses,
        'g_optimizer_state_dict': g_optimizer.state_dict(),
        'g_loss': g_losses,
        }, f'./checkpoint/epoch{epoch}_weight.pth')
        D.train()
        G.train()
                
                
writer.close()
torch.save(D, './model/discriminator.pt')
torch.save(G, './model/generator.pt')
```



### 参考资料:

1. [GAN入门理解及公式推导 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/28853704)
1. [lyeoni/pytorch-mnist-GAN (github.com)](https://github.com/lyeoni/pytorch-mnist-GAN)


---

> 作者:   
> URL: https://blog-12x.pages.dev/%E5%BC%BA%E5%A4%A7%E7%9A%84gan%E7%BD%91%E7%BB%9C/  

