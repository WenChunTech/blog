# Torch.fx模块的使用


<!--more-->

# torch.fx模块的使用

## torch.fx主要有3个组件：
-	**符号追踪器（symbolic tracer)**
-	**中间表示（intermediate representation)**
-	**Python代码生成（Python code generation)**

```python
import torch
# Simple module for demonstration
class MyModule(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.param = torch.nn.Parameter(torch.rand(3, 4))
        self.linear = torch.nn.Linear(4, 5)

    def forward(self, x):
        return self.linear(x + self.param).clamp(min=0.0, max=1.0)

module = MyModule()

from torch.fx import symbolic_trace
# 符号追踪这个模块
# Symbolic tracing frontend - captures the semantics of the module
symbolic_traced : torch.fx.GraphModule = symbolic_trace(module)

# 中间表示
# High-level intermediate representation (IR) - Graph representation
print(symbolic_traced.graph)
"""
graph():
    %x : [#users=1] = placeholder[target=x]
    %param : [#users=1] = get_attr[target=param]
    %add : [#users=1] = call_function[target=operator.add](args = (%x, %param), kwargs = {})
    %linear : [#users=1] = call_module[target=linear](args = (%add,), kwargs = {})
    %clamp : [#users=1] = call_method[target=clamp](args = (%linear,), kwargs = {min: 0.0, max: 1.0})
    return clamp
"""

# 生成代码
# Code generation - valid Python code
print(symbolic_traced.code)
"""
def forward(self, x):
    param = self.param
    add = x + param;  x = param = None
    linear = self.linear(add);  add = None
    clamp = linear.clamp(min = 0.0, max = 1.0);  linear = None
    return clamp
"""
```



## 使用场景

- [Replace one op](https://link.zhihu.com/?target=https%3A//github.com/pytorch/examples/blob/master/fx/replace_op.py)
- [Conv/Batch Norm fusion](https://link.zhihu.com/?target=https%3A//github.com/pytorch/pytorch/blob/40cbf342d3c000712da92cfafeaca651b3e0bd3e/torch/fx/experimental/optimization.py%23L50)
- [replace_pattern: Basic usage](https://link.zhihu.com/?target=https%3A//github.com/pytorch/examples/blob/master/fx/subgraph_rewriter_basic_use.py)
- [Quantization](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/master/quantization.html%23prototype-fx-graph-mode-quantization)
- [Invert Transformation](https://link.zhihu.com/?target=https%3A//github.com/pytorch/examples/blob/master/fx/invert.py)

融合例子:

```python
# Works for length 2 patterns with 2 modules
def matches_module_pattern(pattern: Iterable[Type], node: fx.Node, modules: Dict[str, Any]):
    if len(node.args) == 0:
        return False
    nodes: Tuple[Any, fx.Node] = (node.args[0], node)
    for expected_type, current_node in zip(pattern, nodes):
        if not isinstance(current_node, fx.Node):
            return False
        if current_node.op != 'call_module':
            return False
        if not isinstance(current_node.target, str):
            return False
        if current_node.target not in modules:
            return False
        if type(modules[current_node.target]) is not expected_type:
            return False
    return True


def replace_node_module(node: fx.Node, modules: Dict[str, Any], new_module: torch.nn.Module):
    assert(isinstance(node.target, str))
    parent_name, name = _parent_name(node.target)
    modules[node.target] = new_module
    setattr(modules[parent_name], name, new_module)

def fuse(model: torch.nn.Module, inplace=False) -> torch.nn.Module:
    """
    Fuses convolution/BN layers for inference purposes. Will deepcopy your
    model by default, but can modify the model inplace as well.
    """
    patterns = [(nn.Conv1d, nn.BatchNorm1d),
                (nn.Conv2d, nn.BatchNorm2d),
                (nn.Conv3d, nn.BatchNorm3d)]
    if not inplace:
        model = copy.deepcopy(model)
    fx_model = fx.symbolic_trace(model)
    modules = dict(fx_model.named_modules())
    new_graph = copy.deepcopy(fx_model.graph)

    for pattern in patterns:
        for node in new_graph.nodes:
            # 找到目标Node：args是Conv，target是BN
            if matches_module_pattern(pattern, node, modules):
                if len(node.args[0].users) > 1:  # Output of conv is used by other nodes
                    continue
                conv = modules[node.args[0].target]
                bn = modules[node.target]
                # 融合BN和Conv
                fused_conv = fuse_conv_bn_eval(conv, bn)
                # 替换Node的module，其实就是将融合后的module替换Conv Node的target，背后是模块替换
                replace_node_module(node.args[0], modules, fused_conv)
                # 将所有用到BN Node的替换为Conv Node（已经融合后的Conv）
                node.replace_all_uses_with(node.args[0])
                # 删除BN Node
                new_graph.erase_node(node)
    return fx.GraphModule(fx_model, new_graph)

from torch.fx.experimental.optimization import fuse
from torchvision.models import resnet18

model = resnet18()
model.eval() # 必须在eval模型下fuse

'''
  (layer4): Sequential(
    (0): BasicBlock(
      (conv1): Conv2d(256, 512, kernel_size=(3, 3), stride=(2, 2), padding=(1, 1), bias=False)
      (bn1): BatchNorm2d(512, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      (relu): ReLU(inplace=True)
      (conv2): Conv2d(512, 512, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
      (bn2): BatchNorm2d(512, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      (downsample): Sequential(
        (0): Conv2d(256, 512, kernel_size=(1, 1), stride=(2, 2), bias=False)
        (1): BatchNorm2d(512, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      )
    )
    (1): BasicBlock(
      (conv1): Conv2d(512, 512, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
      (bn1): BatchNorm2d(512, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
      (relu): ReLU(inplace=True)
      (conv2): Conv2d(512, 512, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1), bias=False)
      (bn2): BatchNorm2d(512, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
    )
  )
  (avgpool): AdaptiveAvgPool2d(output_size=(1, 1))
  (fc): Linear(in_features=512, out_features=1000, bias=True)
)
'''

fused_model = fuse(model)

'''
  (layer4): Module(
    (0): Module(
      (conv1): Conv2d(256, 512, kernel_size=(3, 3), stride=(2, 2), padding=(1, 1))
      (relu): ReLU(inplace=True)
      (conv2): Conv2d(512, 512, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (downsample): Module(
        (0): Conv2d(256, 512, kernel_size=(1, 1), stride=(2, 2))
      )
    )
    (1): Module(
      (conv1): Conv2d(512, 512, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (relu): ReLU(inplace=True)
      (conv2): Conv2d(512, 512, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
    )
  )
  (avgpool): AdaptiveAvgPool2d(output_size=(1, 1))
  (fc): Linear(in_features=512, out_features=1000, bias=True)
)
'''
```




## 参考链接:
- [PyTorch新技能解锁：torch.fx - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/428735136)

---

> 作者: map[avatar:<nil> email:<nil> link:<nil> name:<nil>]  
> URL: https://blog-12x.pages.dev/torch.fx%E6%A8%A1%E5%9D%97%E7%9A%84%E4%BD%BF%E7%94%A8/  

