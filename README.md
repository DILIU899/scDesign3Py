目录

- [1. 新版本(基于 rpy2)](#1-新版本基于-rpy2)
- [2. 开发过程所学](#2-开发过程所学)
  - [2.1. 背景知识 and 统计知识](#21-背景知识-and-统计知识)
  - [2.2. rpy2 相关](#22-rpy2-相关)
    - [2.2.1. init 的方法](#221-init-的方法)
    - [2.2.2. 查看当前的 R 环境](#222-查看当前的-r-环境)
    - [2.2.3. `rpy2.robjects.r`函数](#223-rpy2robjectsr函数)
    - [2.2.4. `rpy2.robjects.packages`函数](#224-rpy2robjectspackages函数)
    - [2.2.5. `rpy2.robjects.converter`相关函数](#225-rpy2robjectsconverter相关函数)
  - [2.3. R 相关](#23-r-相关)
    - [2.3.1. 经验](#231-经验)
    - [2.3.2. API](#232-api)
  - [2.4. python 基础类型](#24-python-基础类型)
  - [2.5. pandas and numpy 相关](#25-pandas-and-numpy-相关)
    - [2.5.1. 经验](#251-经验)
    - [2.5.2. API](#252-api)
  - [2.6. matplotlib 相关](#26-matplotlib-相关)
    - [2.6.1. 子图作图](#261-子图作图)
    - [2.6.2. 添加文字](#262-添加文字)
    - [2.6.3. scatter 函数](#263-scatter-函数)
    - [2.6.4. 样例](#264-样例)
  - [2.7. sklearn 相关](#27-sklearn-相关)
  - [2.8. scipy 相关](#28-scipy-相关)
  - [2.9. anndata 相关](#29-anndata-相关)
  - [2.10. pyvinecopulib 相关](#210-pyvinecopulib-相关)
  - [2.11. Statsmodels 相关](#211-statsmodels-相关)
    - [2.11.1. 经验](#2111-经验)
    - [2.11.2. 补充](#2112-补充)
  - [2.12. pygam 相关](#212-pygam-相关)
  - [2.13. python 相关](#213-python-相关)
    - [2.13.1. python 底层基础](#2131-python-底层基础)
      - [2.13.1.1. 线程与进程](#21311-线程与进程)
      - [2.13.1.2. 其他](#21312-其他)
      - [2.13.1.3. Anaconda 相关和库管理相关](#21313-anaconda-相关和库管理相关)
    - [2.13.2. python 机制](#2132-python-机制)
      - [2.13.2.1. 命名空间相关](#21321-命名空间相关)
      - [2.13.2.2. class 相关](#21322-class-相关)
      - [2.13.2.3. python warning 相关](#21323-python-warning-相关)
      - [2.13.2.4. python 计时](#21324-python-计时)
      - [2.13.2.5. python 类型判断和函数传参](#21325-python-类型判断和函数传参)
      - [2.13.2.6. python os](#21326-python-os)
      - [2.13.2.7. 其他](#21327-其他)
  - [2.14. GitHub](#214-github)
  - [2.15. 其他](#215-其他)
  - [2.16. 简单 linux](#216-简单-linux)
- [3. 老版本(不用 rpy2 版本)的相关记录](#3-老版本不用-rpy2-版本的相关记录)
  - [3.1. 未来的下一步功能:](#31-未来的下一步功能)
  - [3.2. Questions](#32-questions)
  - [3.3. scDesign3 相关](#33-scdesign3-相关)
    - [3.3.1. construct data](#331-construct-data)
    - [3.3.2. fit marginal](#332-fit-marginal)

---

## 1. 新版本(基于 rpy2)

- [x] 测试工作 spatial 模拟，过程中有一些 2 维度的 smoother，不一定用高斯过程，看结果怎么样

  - 补充

    - s：tp（各向同性的光滑度，spatial 的数据是会有分层的，感觉光滑度还是不是各向同性） ts（可能会压缩一些系数到 0），gp，cr
    - te，两个 spline 的乘积，指定 1 维的 spline，cr tp gp（？不一定好）
    - k 指模型复杂度，一维的 k 和 te 的 k 是不一样的，te 的 k 是一维 k 的平方

  - 测试：在不同的 k（如 100,200,400），bs 情况下，比较

    - time
    - 效果（图形可视化，比如一个 spatial 的颜色的可视化，spatial 会有一个比较 sharp 的 edge 看看能不能捕捉到）
    - 输出结果算 RMSE

  - 测试结果
    - "ds" spline 速度太慢了，不用考虑了
    - 建议`fit_marginal`时间不要转换成 numeric，转换之后就没有办法看出真实的时间了，特别是运行时间超过 1 min 的时候
    - 挑选了全实验中 RMSE 最大的 20 个基因作为研究对象，因为感觉这些基因总体的预测难度会比较大

- [ ] 完成一个从 R list 转换成 python dict 的函数作为 interface，则只要留一个 return 的 API，如果有需要就转换成 python dict 的形式
- [ ] scDesign 最后是一个类，各个数据以类属性的方式继承，默认这些类属性都是不可修改的，并且就保留着 Rdata 的形式

---

## 2. 开发过程所学

### 2.1. 背景知识 and 统计知识

[Tutorial](https://songdongyuan1994.github.io/scDesign3/docs/articles/scDesign3-introduction-vignette.html)

[GAM 建模、平滑系数选择、R 函数](https://www.bilibili.com/video/BV1h8411s7a1/?p=5&spm_id_from=333.880.my_history.page.click)

[mgcv_R_documentation](https://rdrr.io/cran/mgcv/api/)

[GAM 平滑参数估计方法 GCV 和 REML 的比较](https://github.com/DistanceDevelopment/dsm/wiki/Why-is-the-default-smoothing-method-%22REML%22-rather-than-%22GCV.Cp%22%3F)，REML 和 GCV 都试图达到同样的目标：使模型中的平滑部分恰到好处，不过分平滑。已经证明，当样本量趋向无穷大时，GCV 将选择最佳的平滑参数（即预测误差较低）。在实际应用中，可能存在一个样本量，使得这个渐近结果成立，但我们中很少有人拥有那么大规模的数据。在较小（有限）的样本量下，GCV 可能会出现多个最小值，导致优化困难，因此估计的平滑参数更加变化。此外，GCV 往往会低估平滑（即平滑程度比应该的要高），因为它对过度拟合的惩罚较弱（这在基于预测误差的准则中是有意义的，您希望模型能够很好地拟合数据）。REML（以及 ML、边际似然）对过度拟合的惩罚更重，因此具有更明显的极值，导致更少的优化问题和估计的平滑参数更加稳定。

[Statsmodels GAM documentation](https://www.statsmodels.org/stable/gam.html),[Statsmodels GLM documentation](https://www.statsmodels.org/stable/glm.html)

[pygam documentation](https://pygam.readthedocs.io/en/latest/index.html)

在广义可加模型（Generalized Additive Models，GAM）中，Edof（Effective Degrees of Freedom，有效自由度）是一种度量模型复杂度的指标。它表示模型在拟合数据时使用的有效自由度数量。在 GAM 中，每个平滑项（spline term）的自由度决定了该项对模型的贡献和灵活性。较高的自由度可以使平滑项更加灵活，能够更好地适应数据的变化，但也容易导致过拟合。Edof 考虑了平滑项的自由度以及其他相关因素，提供了一个更准确的模型复杂度度量。

[model selcetion](https://arxiv.org/pdf/1306.2427.pdf)

### 2.2. rpy2 相关

#### 2.2.1. init 的方法

1. 告诉 python 解释器电脑上 R 语言的位置，运行 R 中`R.home()`的字符串复制到`"R_home_location"`即可；如果电脑的环境变量中有 R_HOME 则不需要这一步的配置，`os.environ.get()`就能直接拿到这个 R 的地址

```python
import os
os.environ['R_HOME'] = "R_home_location"
```

2. 相当于在 python 进程中开了一个 R 进程，只要是在同一个 python 进程中，所有的 R 进程都是共享的，即在另一个 module 中对 R 做了什么操作，在主 module 中调用 R 的时候也能看到

3. 对所有的`rpy2.robjects`下属的 R 对象 call `print`的输出结果会是完全 R 风格的输出，适合直接查看；感觉上 rpy2 就是拿到了 R 运行结果的 object

#### 2.2.2. 查看当前的 R 环境

1. `rpy2.robjects.r.search()`可以输出当前已经 attach 的 namesapce 的名字，对应 R 中的`search()`函数
2. `rpy2.robjects.r.ls()`可以输出当前 global 环境中的变量名称，对应 R 中的`ls()`函数
3. `rpy2.robjects.globalenv`是一个类似字典的对象，可以用`items()`方法拿到 global 环境下的所有变量和对应的值

```python
from rpy2 import robjects as ro
{key:value for key,value in ro.globalenv.items()}
```

#### 2.2.3. `rpy2.robjects.r`函数

1. 只要使用`r("""R_script""")`就能把整个 R_script 传到 R 终端作为运行并且拿到最后的运行结果输出
2. 已经 attach 后的 namespace 里的函数就可以使用`r['func_name']`的方式拿到了，在没有 attach 的时候在 R 环境中找不到对应的函数名称所以无法找到。实际上对于在 R 环境中能找到名字的函数可以直接用`r.func_name()`调用该函数。**非嵌套调用只是用一下的情况下可以考虑用 method2，要多重函数嵌套使用的情况下用 method1 感觉会更好**，这两种方法都适合只要拿到结果，不适合把结果还要存一个 R 中的变量（这意味着所有的操作都要借着 python 的 interface 来操作不能在 R 中操作，因为所有的结果都是以 python object 的形式保存的）

```python
from rpy2.robjects import pandas2ri
from rpy2.robjects import r
pandas2ri.activate()
data = pd.DataFrame([[1,4],[2,5],[3,6]],columns=['x','y'])
# method 1 to call lm
lm = r['lm']
print(lm(ro.Formula('y~x'),data=data))

# method 2 to call lm
print(r.lm(ro.Formula('y~x'),data=data))
```

3. 从 R 拿到的函数一般要用 R 对象（即经过 rpy2 处理过后，从 python 对象变为 r 对象）作为输入才能进行调用，拿到的结果也是 R 对象，如果有必要再手动转换为 python 对象。
   - `str`对象是直接兼容的，但是返回的`str`对象也是封装在`np.array`中的；`bool`对象也是直接兼容的，返回的对象封装在 BoolVector 中；普通的数字对象也是兼容的
   - python 中的`list`对应到 R 中也是`list`，而不是直观上的 vector，要得到 vector 类型的输入必须先用`rpy2.robjects.vectors`中的对应函数对`list`进行转换
   - `dict`是不兼容的
   - `None`是不兼容的，专门有`rpy2.robjects.NULL`对象对应的是`NULL`，但是这个`NULL`的布尔类型是`False`
   - 使用`pandas2ri.activate()`和`numpy2ri.activate()`后不需要显式地将 pandas 和 numpy 对象转换为 R 对象，rpy2 会自动将这两种对象转换为 R 对象，返回值如果是可以转换为 numpy(如 vector,matrix) 的类型也可以直接从 R 对象转换为 Python 对象，但是返回值是`data.frame`或者`list`就会直接存成 rpy2 中对应的 R object，不会自动转成`pd.DataFrame`或者`dict`
   - **拿到的 matrix 对象是没有行名和列名的！**直接转换成 np 格式了
   - python 命名的`_`和 R 中的`.`是可以被 rpy2 自动转换的

#### 2.2.4. `rpy2.robjects.packages`函数

1. `packages.isinstalled()`函数可以检查函数包是否已经被下载，返回 python bool 对象，并且不会像 r 的`require()`函数一样自动就把包 attach 了
2. `packages.importr()`函数一方面把 R 对应包的命名空间拿到 python 中供 python 调用，一方面在对应的 R 进程中会把对应的包 attach 上，因此运行`packages.importr()`以后就能用`r['func_name']`拿到对应的包里的函数了

#### 2.2.5. `rpy2.robjects.converter`相关函数

主要功能是实现 python object 和 R object 的相互转换，默认是`default_converter`并且设置全局，其他的内置 converter 就包括`pandas2ri`和`numpy2ri`，`pandas2ri`和`numpy2ri`只包含对应的类型的转换，但对于简单的`str`的转换又是只有`default_converter`才含有的，因此要转换含有字符串的`pd.DataFrame`时要把两种 converter 连用

- 如果需要在一个 local 的地方实现特殊的 convert 需求，可以使用

```python
with default_converter.context():
  pass
```

- 如果有多个 converter 一起使用，可以把对应的 converter 相加

```python
with (default_converter + pandas2ri.converter).context():
    r('function(x){print(x)}')(example_sce.obs)
```

- 可以通过配置`converter`的方式，再结合`ro.conversion.get_conversion().py2rpy()`，可以实现局部类型的单向转换

```python
with (default_converter+pandas2ri.converter).context():
    test = ro.conversion.get_conversion().py2rpy(test)
```

### 2.3. R 相关

#### 2.3.1. 经验

- R 中的[环境概念](https://zhuanlan.zhihu.com/p/585188479)，
  - `!require('package_name',quietly = TRUE)`如果该包没有被下载就会返回 True，如果该包已经被下载，就会返回 False，同时被 attach 到 Namespace 中。在没有 attach 的情况下，只要该包被下载了，就可以用`package_name::func_name`的方式调用包中的函数
- SingleCellExperiment 相关
  - 如何从 singlecellexperiment object 转换成 h5ad 存储后，用 scanpy 读取 [参考](https://www.bilibili.com/read/cv8675277/)使用`sceasy`包
  - SummerizedExperiment 集成从 singlecellexperiment object 中提取 coldata/assay 等的方法，对应`anndata.obs`, `anndata.layer`
- 并行计算用函数 `parallel::mcmapply`, `BiocParallel::bpmapply`([document](https://www.bioconductor.org/packages/devel/bioc/manuals/BiocParallel/man/BiocParallel.pdf)), `pbmcapply::pbmcmapply`都有两类参数，一类参数是直接传进去的，一类是封装在 MoreArgs 中传进去的，前者在每一个进程中是可迭代式地一个个取出来的，因此在每个进程中是不同的，后者是每一个进程中都需要用到的部分。需要注意的是`mcmapply`在传入第一类参数的时候不会自动实现对参数的排列组合，而是**按照参数所在位置进行对应**。如果要生成所有的排列组合，调用`expand.grid()`，可以生成有所有排列组合 dataframe。

```r
parafunc(fun_name,arg1=...,arg2=...,...,MoreArgs=list())
```

- `tryCatch`和 python 中的`try...except...`一致，`withCallingHandlers`，可以在特定的代码块中捕获和处理异常、警告和消息
- `list`和`data.frame`相关
  - `list`和`data.frame`数据类型，`list`可以认为是一个字典（如果`list`创建的时候不写明键名字`list(0,1,2)`，会自动用 1 开始的数字编号作为键名字），访问某一个键的方式可以是`$name`也可以是`list[["name"]]`。`data.frame`可以看成一个特殊的`list`，列名是键名，每一个值都是一个`vector`对象，因此在访问列名时也可以使用`$name`方法，也因此调用`length()`函数时，`data.frame`输出的是列数，即键数目，和对`list`计算得到的结果是一样的（也和 python 对 dict 调用 len 输出键数是一样的）。当然`data.frame`还可以使用很多切片索引等方法是`list`不行的
  - `is.factor`等函数比起`typeof`更适合判断`data.frame`一列的数据类型
  - `lapply`和`sapply`的区别：都是对 list 的每个键下的元素用相同的 function 进行操作，区别在于`lapply`返回一个对应的 list，而`sapply`会尝试返回一个更简单的结果，如 vector 或者 matrix，若返回矩阵，其中**每列**对应于输入 list 中的一个元素的结果。（感觉操作的实际上是任意可迭代对象）
  - `list`对象非常的恶心，很难转换成别的类型的对象，如果要变成`vector`，使用`unlist()`函数。
  - 使用`cbind`将带有 name 的`vector`或`data.frame`并在一起时会把每一列变成一个`list`，很抽象
- `formula`对象可以用`update.formula`进行修改，在`formula`用`-`代表删去一些变量
- `invisible()`可以对单行运行的 R 代码不输出返回值

#### 2.3.2. API

- `mclust:Mclust`使用高斯混合模型进行聚类 [参考](https://mclust-org.github.io/mclust/reference/Mclust.html)
- `interaction`函数计算 factor 变量的交互结果（乘积）
- `dplyr`操作 dataframe 的包 `dplyr::mutate(df,name = func(column))`: 在原来的 dataframe 中加一列名为`name`，内容为`func(column)`的列
- `table`: 计算每个元素的出现次数
- `names`: 返回给的 vector dataframe 等的元素名称
- singlecellexperiment 对象的存储和加载分别使用`saveRDS(sce,file ='xxx.rds')`和`loadRDS(sce,file ='xxx.rds')`实现
- `grepl`正则表达式
- `mgcv::gam(s(varible_name,k=10,bs='cr'),method=REML)`bs 代表回归样条的[基函数类型](https://www.rdocumentation.org/packages/mgcv/versions/1.9-0/topics/smooth.terms)（cr 为三次样条函数，三次样条函数指的就是具有连续性、且一阶和二阶导连续的三阶分段多项式），[k](https://www.rdocumentation.org/packages/mgcv/versions/1.9-0/topics/choose.k)，method 指定了平滑参数的估计方式
- `mgcv`中 s,ti,te 的区别和指定方式，简单而言，ti 仅代表交互效应，te 代表又有交互效应又有非交互效应，`te(x1,x2)`与`s(x1)+s(x2)+ti(x1,x2)`大致类似，但是估计的平滑惩罚数目不同(2 vs 4)。而 s(x1,x2)和 te(x1,x2)是类似的，区别在于 s(x1,x2)假设 x1,x2 有相同的 scale，即 isotropic 各向同性，te(x1,x2)认为是各变量的 scale 不同，各向异性(tensor product ti and te 处理的就是各向异性的情况)。[参考](https://stats.stackexchange.com/questions/519433/gam-and-multiple-continuous-continuous-interactions-tensor-smooths)
- `all.vars(formula)`返回一个 vector，第一位置是 y 的 var，第二位置是 x 的 var
- `formals(func)` 返回函数所有参数的标准输入，返回值是`list`

### 2.4. python 基础类型

- list \* k，即可把 list 中的元素复制 k 遍，和 list1+list2 异曲同工
- `set(list)`得到 list 中所有的 unique 内容
- `map(func,iterable)`得到一个经过 func 操作的 iterable
- `set().issubset()`检查是不是一个类列表对象的子集，注意**空列表也会被认为是子集**
- 利用字典构造式进行字典合并

```python
def func(x):
    return {x: 'test'}

# 生成一个包含字典的列表
result_list = [func(i) for i in range(5)]

# 使用字典推导式将字典拼接成一个大字典
merged_dict = {k: v for d in result_list for k, v in d.items()}
```

- `dict`对象如果使用`dict[key]`拿到对应的 key 的值，且没有这个 key，那么就会报错 keyerror；如果使用`dict.get()`方法，则没有这个 key 时会返回一个`None`
- 将多个`list`对象中的内容合并在一个`list`中可以使用`itertools`包，返回一个可迭代对象，用类型转换为 list。代码为：`list(itertools.chain(*filter(None, [lists])))`（同时忽略其中的所有`None`对象）
- 当有一个字典存储了一个 python 函数的参数名和参数值时，可以通过`func(**dict)`向那个函数传参
- 当使用一个`list`存储不定数量的返回值时，如果只有一个返回值，需要在赋值时使用 `(a,) = res_list` 的格式，只有这样才能正确拿到里面的值，而不是拿到嵌套在`list`中的值

### 2.5. pandas and numpy 相关

#### 2.5.1. 经验

- 从其他 dataframe 赋值的新 dataframe，要用.copy()得到一个备份，否则会出问题
- 如果用下述代码进行 dataframe 赋值，需要`series`和`dataframe`有相同的`index`，否则匹配不上的情况下`dataframe`中`index`匹配不上的部分直接赋值为`nan`，`series`的内容就丢了

```python
# not recommend, in this case, pd.concat may be preferred
df['name'] = pd.Series()
# recommend
df['name'] = np.array()
# or
df['name'] = list
# but not generator
```

- pandas 复制一行最快的办法：用这一行的索引重复调用该行即可

```python
# 得到index行重复repeat_times次的索引
repeat_index = np.repeat(np.array(index),repeat_times)

# 用该索引提取dataframe中的这一行
new_df = df.iloc[repeat_index]
```

- 遍历每一列的数据类型的方法(对于要把所有数值变量都认为是数值的方法，关键在于`np.issubdtype(dtype, np.number)`的应用)

```python
import pandas as pd
import numpy as np

# 创建示例DataFrame
data = {
    'col1': [1, 2, 3, 4],
    'col2': ['a', 'b', 'c', 'd'],
    'col3': [1.1, 2.2, 3.3, 4.4],
    'col4': [5, 6, 7, 8],
}
df = pd.DataFrame(data)

# 获取每列的数据类型
dtypes = df.dtypes

# 创建分类列表
category_cols = []
numeric_cols = []

# 遍历每列的数据类型
for col, dtype in dtypes.items():
    if dtype == 'category':
        category_cols.append(col)
    elif np.issubdtype(dtype, np.number):
        numeric_cols.append(col)

# 输出结果
print("Category columns:", category_cols)
print("Numeric columns:", numeric_cols)
```

- pandas 处理 NAN 数据(简单版)

```python
# 检查数据中是否有NAN最快的办法
test.isnull().values.any()
# inplace = True，则直接在原dataframe上进行修改
## 所有NA替换为100
test.fillna(100,inplace=True)
## 指定列的NA替换为某个给定的值
test.fillna({'lineage1':10,'lineage2':100},inplace=True)
## 删除指定列为NA的行
test.dropna(subset=['lineage2'],inplace=True)
## 删除所有feature中有NA的observation
test.dropna(inplace=True)
```

- Pandas 中的 isin() 方法来判断一个 Series 中的值是否在一个可迭代对象中。isin() 方法返回一个布尔型的 Series，表示每个元素是否在给定的可迭代对象中。

```python
# 示例数据
data = pd.Series([1, 2, 3, 4, 5])
values = [3, 5, 7]

# 判断每个元素是否在 values 中
result = data.isin(values)

# 0    False
# 1    False
# 2     True
# 3    False
# 4     True
# dtype: bool

# e.g 用来反选index
dat_use = dat_use.loc[~dat_use.index.isin(removed_cell)]
```

#### 2.5.2. API

- pandas 支持`dtype('category')`对应 R 中的`factor`类型 [参考](https://zhuanlan.zhihu.com/p/384727931)

```python
pd.dataframe['col'] = pd.dataframe['col'].astype('category')
```

- [`pd.groupby`操作](https://zhuanlan.zhihu.com/p/388789630)

```python
for name,group in df.groupby('name'):
    name    # category
    group   # corresponding dataframe
```

- `pd.value_counts`函数统计各个元素的出现次数
- `df.values`返回 dataframe 中的数值，为 np.array
- `df.describe()`返回所有的数值变量的描述性统计量，类似`summary` in R
- `np.random.seed(0)`设置`numpy`中的随机数种子
- `np.ndarray`中 dtype 为`U1`， 表示 Unicode 字符串，长度为 1。
- `pd.DataFrame.style`可以对数据框的一些外观（比如字体颜色和背景色颜色做修改），在存储时可以使用`.to_excel(path.xlsx", engine="openpyxl")`的方式进行保存，这样可以存储所有的颜色注释。**所有的 style 对应的对象都很难再像正常的数据框对象一样进行数值操作了，因此在做颜色标注之前一定要先做完数据处理。**

```python
## 自定义染色的一个实例
pos_dict = {"best": [], "second": [], "third": []}
for name in compare.columns:
    if name in ["time(min)", "RMSE", "MAE", "aic", "bic"]:
        sorted = compare.sort_values(
            name,
        )
    elif name in ["Person_corr", "R_adj", "exp_dev"]:
        sorted = compare.sort_values(
            name,
            ascending=False,
        )

    ## 找到对应结果最好的前三个，将坐标记录
    pos_dict["best"].append([sorted.head(3).index[0], name])
    pos_dict["second"].append([sorted.head(3).index[1], name])
    pos_dict["third"].append([sorted.head(3).index[2], name])

value_dict = {"best": [], "second": [], "third": []}
for key, values in pos_dict.items():
    for value in values:
        value_dict[key].append(compare.loc[value[0], value[1]])

## 根据数据结果进行颜色调整
def set_cell_color(value):
    if value in value_dict["best"]:
        return "background-color: orange"
    elif value in value_dict["second"]:
        return "background-color: yellow"
    elif value in value_dict["third"]:
        return "background-color: blue"

## 直接用applymap对每一个单元格都进行一遍检索和改变样式过程
res = compare.style.applymap(set_cell_color)
```

### 2.6. matplotlib 相关

#### 2.6.1. 子图作图

- `fig, axes = plt.subplots(a, b, figsize=(b * 5, a * 5))`得到一张有 n 张子图的一个大图，通过前两个参数将整个大图分割成 $a \times b$个子图，通过`figsize`相当于得到每个子图的大小，但是要注意参数是反过来的
- 进一步用`row, col = np.unravel_index(i, axes.shape)`和`ax = axes[row, col]`来得到一张子图的位置，其中`i`是第`i`张子图
- 使用`ax.scatter`对于该子图进行绘图，理论上任何可以画的子图函数都可以调用，调用就像普通的`plt`调用方法一样
- `fig.suptitle()`指明全图的标题，`ax.set_title()`得到小标题

#### 2.6.2. 添加文字

- `ax.text()`自定义添加文字，文字位置如果都是 0-1 之间的数字就是按比例放置，超过范围的就会转换成在对应的坐标处放文字
- `ax.legend()`添加图例

#### 2.6.3. scatter 函数

- `c`参数代表颜色，是一种数值上的对应，和`cmap`对应使用，前者给一个数值，后者拿到这个数值后在对应的 map 上找对应的颜色
- `s`参数给出点的大小，可以是一个可迭代对象给每个点一个特定的大小
- `plt.colorbar(scatter_plt_object)`可以把对应的颜色柱对应的图例显示在边上

#### 2.6.4. 样例

```python
## 作图样例
fig, axes = plt.subplots(8, 5, figsize=(5 * 5, 9 * 5))
fig.suptitle("Gene name: " + gene, fontsize=16, y=0.95)

for i, (method, v2) in enumerate(v1.items()):
    rmse = v2["RMSE"]
    r = v2["r"]
    df = v2["plot"]

    row, col = np.unravel_index(i, axes.shape)
    ax = axes[row, col]
    sc = ax.scatter(df["X"], df["Y"], c=df["Expression"], cmap="viridis")
    ax.set_title(f"Setting: {method}\nRMSE: {rmse:.2f}\nr: {r:.2f}")
    ax.set_xlabel("X")
    ax.set_ylabel("Y")
    plt.colorbar(sc)

    plt.subplots_adjust(hspace=0.4, wspace=0.3)

    # ax.text(0.8, 0.02, f"RMSE: {rmse:.2f}\nr: {r:.2f}", transform=ax.transAxes, ha="center")

plt.savefig(f"./Res/{datasetname}/plot/{gene}.pdf")
```

### 2.7. sklearn 相关

- `sklearn.mixture.GussianMixture`为高斯混合模型
- [KDE 核密度估计，非参方法估计分布函数](https://scikit-learn.org/stable/modules/density.html)
- sklearn 实现数据记忆的方式就是将一个 model 封装成一个类，类内不需要设置私有变量，可参考
- sklearn 训练数据如果是`np.array()`则维度必须为 n\*k，不能是只有单维度，也因此如果传 pandas 数据必须是`dataframe`不能是`series`

### 2.8. scipy 相关

- [稀疏矩阵信息](https://zhuanlan.zhihu.com/p/306320459)，可以使用`toarray()`转换成一般的 ndarray 的样式

### 2.9. anndata 相关

- anndata 数据结构存储都是 scipy 稀疏矩阵形式
- anndata.layer 存储不同类型的 assay（即对 anndata.X 的各种 transform）

```python
adata.layers['log_transformed'] = np.log1p(adata.X)
```

### 2.10. pyvinecopulib 相关

pyvinecopulib 和 rvinecopulib 是 C++库 Vinecopulib 在 python 和 R 上的分别实现

```python
import pyvinecopulib as pv
d = length(x.columns)
# calculate the empirical distribution of the raw RV
## u for a uniform distribution RV
u = pv.to_pseudo_obs(x)
# construct vine copula
cop = pv.Vinecop(data = u)
# simu new unifrom distribution RV
u_simu = cop.simulate(n=n_cell)
x_simu = np.asarray([np.quantile(x[:, i], u_sim[:, i]) for i in range(0, d)])
## x_simu shape n_variate * n_observation
x_simu = x_simu.T
```

输入数据 data 可以是 np.array 也可以是 pd.Dataframe，返回值为 np.array，其特性与 rvinecopulib 相似，只有一个变量时也能拟合与生成分布和采样数据。

注意其输入数据必须是 0-1 之间的数值，因为本质上 copula 函数是对一组 0-1 间的均匀分布数据进行计算，随机变量的 CDF 即为 0-1 均匀分布。但是 pyvinecopulib 不会像 rvinecopulib 一样给 raw data 自动做好转换为 CDF 的对应取值再做拟合，需要添加一步得到，并且要将拟合结果返回到原本的数据。

### 2.11. Statsmodels 相关

#### 2.11.1. 经验

- 使用`GLMGam.from_formula`构建模型会更好，线性的部分用 formula 的形式给出，其会将 dataframe 中 category 的部分直接转换为 dummy 编码并且自动添加截距项。如果直接用 y 和 x 对应的 dataframe 输入（用 endog 和 exog）则两者都需要手动操作，很不方便。`from_formula`由 package`patsy`提供支持([documentation](https://patsy.readthedocs.io/en/latest/))，`patsy`本身支持根据 mgcv 格式产生 spline 项，但是不支持惩罚项等的计算，因此无法投入应用。
- 超参数的选择可以指定，[`select_penweight`](https://www.statsmodels.org/stable/generated/statsmodels.gam.generalized_additive_model.GLMGam.select_penweight.html#statsmodels.gam.generalized_additive_model.GLMGam.select_penweight)，方法默认 aic，也可以选 gcv 等方法

```python
# 先大致指定建模方法
gam_bs = GLMGam.from_formula('YES ~ GDP + AGE + test',data = data.data, smoother=bs)
gam_bs.fit()
# 挑选超参数(平滑参数的选择)
alpha = gam_bs.select_penweight()[0]
# 指定完整的模型
model = GLMGam.from_formula('YES ~ GDP + AGE + test',data = data.data, smoother=bs,alpha=alpha)
# 模型参数拟合
result = model.fit()
```

- GLMGam 类使用 fit 方法后会返回 GLMGamResults 类，该类有两种获取 predict value 的方法，一种是`predict()`，直接返回预测值，为`np.array()`，另一种是`get_prediction().summary_frame()`前者返回一个 predcit result 类，后者用 dataframe 形式呈现结果，除了包含预测值外，还有对应的标准误差估计和置信区间

#### 2.11.2. 补充

[GAM 示例](https://www.statsmodels.org/stable/gam.html) [支持的 distribution](https://www.statsmodels.org/stable/glm.html) [model API](https://www.statsmodels.org/stable/generated/statsmodels.gam.generalized_additive_model.GLMGam.select_penweight.html#statsmodels.gam.generalized_additive_model.GLMGam.select_penweight)，`smoother`是样条函数选择，需要调用别的类来指定，在其中进一步规定样条函数自由度，`alpha`是平滑参数选择，可用`select_penweight()`得到。`k`代表了将一个拟合区间细分成 k+1 个小区间，每个区间使用一个 spline 去拟合，`k`与 spline 类型和要估计的参数`df`有换算关系。具体关系见[此](https://stats.stackexchange.com/questions/517375/splines-relationship-of-knots-degree-and-degrees-of-freedom#:~:text=In%20essence%2C%20splines%20are%20piecewise,degree%203%20and%20so%20on.)。

### 2.12. pygam 相关

- `model.gridsearch(X,y)`自动找到最好的超参数，[progress 是否显示进度条;用 lam 参数可以指定 lam 的搜索空间;objective 为搜索的优化目标](https://pygam.readthedocs.io/en/latest/api/gam.html)
- `model.sample`和`model.predict`的区别，sample 方法用于从模型的后验分布中生成样本。它可以用于生成模型的不确定性估计，以及对未观测数据的采样预测。sample 方法可以接受一个输入特征矩阵，并为每个样本生成一组预测。它返回一个 numpy 数组，其中每一行代表一个样本的预测。predict 方法用于对给定的输入进行预测。它接受一个输入特征矩阵，并为每个样本生成预测值。与 sample 方法不同，predict 方法不考虑模型的不确定性，它仅返回点估计的预测结果。它也返回一个 numpy 数组，其中每一行代表一个样本的预测值。
- AICc = AIC + 2p(p + 1)/(n − p − 1)

### 2.13. python 相关

#### 2.13.1. python 底层基础

##### 2.13.1.1. 线程与进程

- [进程与线程的区别](https://www.bilibili.com/video/BV19b411L7y1/?spm_id_from=333.337.search-card.all.click&vd_source=44ff757ed2fdadeadf410d10bde17c2c)进程是分配资源的操作，是一个资源的综合，一个进程中是线程在进行操作，一个进程中的多线程，用一块资源来实现，实现多任务；而多进程则是多划了几块资源，每个进程只有一个主线程，每个线程都用自己的进程资源实现多任务
- [python 的进程和线程的编程](https://www.bilibili.com/video/BV1jW411Y7pv?p=2&vd_source=44ff757ed2fdadeadf410d10bde17c2c)
- [`multiprocessing`包大致功能介绍及编程](https://blog.csdn.net/sqzhao/article/details/120732881)
- [如何得到子进程的返回值](https://blog.csdn.net/u013510614/article/details/109852590)

```python
# multiprocess demo
import multiprocessing as mp

# The function for the subprocess to call
def test(i):
    i += 1
    print(f'The {i}th process has been done')
    return i

if __name__ == '__main__':
    # Pool default for create the same number of subprocess as the cpu core
    pool = mp.Pool()
    storage = []
    # call the function name and the args are loaded as a tuple
    for i in range(15):
        storage.append(pool.apply_async(test, (i,)))
    # stop adding subprocess
    pool.close()
    # wait for every subprocess to finish
    pool.join()
    # get the result from the subprocess object
    result = [i.get() for i in storage]

    print(result)
```

##### 2.13.1.2. 其他

- [如何组织 python 项目](https://zhuanlan.zhihu.com/p/335347908#:~:text=1%20%E7%BB%84%E7%BB%87%E7%BB%93%E6%9E%84%201%20Python%E9%A1%B9%E7%9B%AE%E7%9A%84%E7%BB%84%E7%BB%87%E7%BB%93%E6%9E%84%E4%B8%BB%E8%A6%81%E7%94%B1%20%E5%8C%85--%E6%A8%A1%E5%9D%97--%E7%B1%BB%20%E8%BF%99%E4%BA%9B%E9%83%A8%E5%88%86%E6%9E%84%E6%88%90%E3%80%82%202%20%E5%8C%85%EF%BC%9A,%E4%B8%80%E4%B8%AA%E6%A8%A1%E5%9D%97%E4%B8%8B%E9%9D%A2%E4%B9%9F%E5%8F%AF%E4%BB%A5%E5%8C%85%E5%90%AB%E5%A4%9A%E4%B8%AA%E7%B1%BB%E3%80%82%20%E6%A8%A1%E5%9D%97%E4%B8%8B%E9%9D%A2%E4%B9%9F%E5%8F%AF%E4%BB%A5%E7%9B%B4%E6%8E%A5%E5%86%99%E5%87%BD%E6%95%B0%E3%80%82%204%20%E7%B1%BB%EF%BC%9A%20%E7%B1%BB%E4%B8%AD%E5%AE%9A%E4%B9%89%E4%BA%86%E5%8F%98%E9%87%8F%E5%92%8C%E5%87%BD%E6%95%B0%E3%80%82%205%20%E5%87%BD%E6%95%B0%EF%BC%9A%20%E7%94%A8%E4%BA%8E%E5%AE%9E%E7%8E%B0%E7%89%B9%E5%AE%9A%E5%8A%9F%E8%83%BD%EF%BC%8C)

- 通过 C++库的 Python 接口，可以在 Python 环境中直接调用 C++库的功能，而无需安装额外的 C++解释器。Python 本身已经提供了 CPython 解释器，它是使用 C 语言实现的，并且是 Python 的默认解释器。可以将 C++代码编译为动态链接库（如.so 文件或.dll 文件），然后在 Python 中导入这些库，并使用其中的函数和类。Python 解释器会与这些动态链接库进行交互，以实现对 C++代码的调用和执行。

##### 2.13.1.3. Anaconda 相关和库管理相关

- `conda init powershell`使得 anaconda 在 windows powershell 中会显示虚拟环境名称，如果遇到遇到无法加载 profile.ps1，[解决方案](https://zhuanlan.zhihu.com/p/452273123)
- `pip -V`查看现在使用的 pip 对应的下载位置和 python 解释器版本
- 新 python 环境安装 jupyter kernel 的方式

```bash
conda activate newenv
conda install ipykernel
python -m ipykernel install --user --name newenv --display-name 'any name is OK, just kernel name'
```

- 依赖管理问题

  - `conda env export > environment.yaml`命令导出虚拟环境中的所有包依赖。用 conda 下载的包用`pip freeze > requirements.txt`时不会正确显示在 pypi 上的路径，只会显示一个本地路径，该命令主要是为了导出 pip 的相关依赖。
    - 个人感觉如果是想要包最后能直接发布到 Pypi 上，那就都用 Pypi 来装实现基础功能的包，这样方便直接得到直接可以用 Pypi 安装的 requirements，运行`pip install -r requirements.txt`即可实现迁移。
    - 如果为了真个 conda 环境的可迁移性，则直接用 ymal 文件即可，在创建新的虚拟环境时使用`conda env create -f environment.yaml`即可重新配置相同的环境。
  - 在 conda 环境中得到纯 pip 的 requirements 感觉也没什么特别好的办法，要么从 conda 的 yaml 里将 pip 相关的复制出来统一格式，要么从 pip freeze 中去掉本地安装路径的部分(目前在`modify_requirements.py`中实现，并验证过应该可以完整复现 pip 的所有安装)。<b>conda 和 pip 产生的文件都是 utf-16 编码的，处理解码时需要注意</b>
  - 如果全都是用 pip 装包，额外装了依赖，要卸除这些依赖，可以考虑`pip install python3-pip-autoremove`后运行`pip3-autoremove package_name`进行包及其依赖的卸载工作

#### 2.13.2. python 机制

##### 2.13.2.1. 命名空间相关

- [import 函数详解](https://www.bilibili.com/video/BV1K24y1k7XA/?spm_id_from=333.788&vd_source=44ff757ed2fdadeadf410d10bde17c2c) 在 import 一个 module 的时候 python 会进行以下步骤
  - 创建模块命名空间：Python 会为模块创建一个独立的命名空间，该命名空间用于存储模块中定义的变量、函数和类。这样可以避免不同模块之间的命名冲突。
  - 执行模块代码：Python 会<b>按顺序执行</b>模块中的代码，将定义的变量、函数和类加载到模块的命名空间中。这样，其他代码可以通过模块名来访问这些定义。
  - 返回模块对象：在导入过程完成后，import module 语句会返回一个表示模块的对象，该对象可以在后续的代码中使用。可以使用该对象访问模块中定义的内容，例如变量、函数和类。
- [`if __name__ == '__main__'` 详解](https://blog.csdn.net/heqiang525/article/details/89879056),**name**中保存的是[module 的名字](https://zhuanlan.zhihu.com/p/57309137)，用来判断这个脚本是不是在被主调用还是被 import

##### 2.13.2.2. class 相关

1. 类的属性分为两种，一种是类属性，一种是实例属性，类属性在定义类的时候直接指名即可，实例属性是只有创建了类的实例，并且运行了创建实例属性的语句之后才会产生。
2. `self`字段就意味着带`self`字段的对象（var，fun 等）会被绑定到一个给定的实例上，可以用`__dict__`方法查看类/实例有哪些属性。**实例的`__dict__`看不到定义在类层面上的东西，但是定义在类上的东西依然可以被实例调用；类的`__dict__`看不到实例上面的定义，也调用不到实例中定义的东西。即一个实例可以访问已经创建的实例属性和类属性，一个类只能访问类属性。此外在实例层面的对于类属性的修改并不会对类本身该属性造成任何影响**

```python
class test_class():
  # 类属性
  class_var = 'This is a class var'
  def create_ob_var (self):
    # 带self，被绑定在一个实例上，是实例属性
    self.ob_var = 'This is a object var'

# 未运行create_ob_var，没有实例属性产生，也看不到类级别的属性和函数定义
test_ob = test_class()
test_ob.__dict__
# {}

# 看不到实例级别的属性，但是可以看到类级别的属性和函数
test_class.__dict__
# mappingproxy({'__module__': '__main__',
#               'class_var': 'This is a class var',
#               'create_ob_var': <function __main__.test_class.create_ob_var(self)>,
#               '__dict__': <attribute '__dict__' of 'test_class' objects>,
#               '__weakref__': <attribute '__weakref__' of 'test_class' objects>,
#               '__doc__': None})

# 可以看到实例属性在产生后就可以被查看到
test_ob.create_ob_var()
test_ob.__dict__
# {'ob_var': 'This is a object var'}

# 实例可以访问实例属性和类定义的属性
test_ob.class_var
# 'This is a class var'
test_ob.ob_var
# 'This is a object var'

# 类只能访问类属性
test_class.class_var
# 'This is a class var'
test_class.ob_var
# AttributeError: type object 'test_class' has no attribute 'ob_var'

# 实例对象对于类属性的改变不会影响类属性的值，而是会创建一个随着该实例对象的同名属性
test_ob.class_var = 1
test_ob.class_var
# 1
test_class.class_var
# 'This is a class var'
```

3. Python 类中所有可以被调用到的属性都是可以被直接赋值修改的，因此需要[Python 类的私有公有变量设置](https://zhuanlan.zhihu.com/p/30223570)，可以控制一些不想被修改的变量不被修改（但其实也是假的，python 没办法严格控制类的私有变量，也因此一般就用`_var_name`代替`__var_name`表示不想被用户随意拿到和修改的类属性）。这种设置**在类属性和实例属性、函数上都生效**

```python
class test():
  def __init__(self):
    self.__hidden = 'Hidden'
  def give_hidden(self):
    return self.__hidden

# 无法直接访问hidden
ob = test()
ob.__hidden
# AttributeError: 'test' object has no attribute '__hidden'

# 原因其实就是__开都的属性名字之前加了_class_name导致找不到了
ob.__dict__
# {'_test__hidden': 'Hidden'}

# 但是实际上还是可以对该变量进行赋值修改
ob.__hidden = 1
ob.__hidden
# 1

# 但赋值修改的其实是另一个就叫__hidden的属性
ob.__dict__
# {'_test__hidden': 'Hidden', '__hidden': 1}

# 实际在调用的时候用的还是_test__hidden这个属性
ob.give_hidden()
# 'Hidden'
```

4. 类的两种 built in 的装饰器函数，`@classmethod`和`@staticmethod`，都是为了在不实例化类的情况下就能调用类内的一些方法。因此这两种方法函数的第一个 argument 都不用是`self`，`@classmethod`的第一个 argument 必须是`cls`。`@classmethod`可以被用来不需要实例化的情况下访问一些类变量并且进行操作，比如修改一个一般不被外部允许改变的类属性；`@staticmethod`相当于为了功能完整性封装在一个类内的方法。`@classmethod`和`@staticmethod`装饰的函数在**实例对象也可以使用**。

```python
class test:
  __hidden = 1

  @classmethod
  def change_hidden(cls,value):
    cls.__hidden = value
    return cls.__hidden

  @staticmethod
  def f(a,b):
    return a+b

test.f(1,2)
# 3

test.change_hidden(5)
# 5
```

5. `hasattr`函数判断一个对象是否有对应的属性

##### 2.13.2.3. python warning 相关

```python
import warnings

# simplefilter 和 filterwarnings 可以认为前者是后者的简化版本
warnings.simplefilter('ignore')

def divide_numbers(a, b):
    if b == 1:
        warnings.warn(f"The {a}th divide is nonsense", RuntimeWarning)
        warnings.warn(f'Another warning for {a}',UserWarning)

## 两种warning都不会出现
divide_numbers(10, 1)

# 控制某个局部产生warning，并且抓住warning的信息(此时不会有warning的显示)
with warnings.catch_warnings(record =True) as warninglist:
    warnings.filterwarnings('default',category=RuntimeWarning)
    ## 只会出现runtime warning
    divide_numbers(9, 1)

# 重设warning filter
warnings.resetwarnings()
## 两种warning都会出现
divide_numbers(8, 1)

# 展示捕获的warning信息
for warning in warninglist:
    print(warning.message)
```

##### 2.13.2.4. python 计时

```python
import time
start_time = time.time()
end_time = time.time()
running_time = start_time - end_time
```

##### 2.13.2.5. python 类型判断和函数传参

- `isinstance` or `type`，[建议使用前者](https://www.jianshu.com/p/7ef549503c93)
- [函数指定传参的数据类型](https://blog.csdn.net/qq_42327755/article/details/87196150)
- python 在声明函数时如果可以接受不同的输入类型的话可以使用如下方式，[参考 1](https://www.bilibili.com/video/BV11Z4y1h79y/?spm_id_from=333.788.recommend_more_video.-1&vd_source=44ff757ed2fdadeadf410d10bde17c2c)，[参考 2](https://www.bilibili.com/video/BV16Y411K73d/?p=23&spm_id_from=pageDriver)，编译时通过标注类型实现编译时报错而非运行时报错

```python
from typing import Union
def toy(x:Union[list,str,int]) -> None:
    pass
```

- python 中可以通过`*args`和`**kwargs`进行**不定数量的参数**的传参，前者是以`tuple`的形式传入，后者以`dict`的形式传入

##### 2.13.2.6. python os

`os`库作用是可以和操作系统交互，常用功能如下：

- `os.environ`是一个类似字典的对象，存储所有 python 运行的环境变量的名称和对应的地址，可以用字典的方式，如`get(key)`或`[key]`方式得到某一个环境变量的具体地址
- `os.path.exists()`判断文件是否存在
- `os.listdir()`输出对应路径下所有文件(夹)的对应名字
- 将脚本工作路径设置为当前脚本所在位置的方法

```python
import os
os.chdir(os.path.dirname(__file__))
```

##### 2.13.2.7. 其他

- 判断 None Type 时，必须要用`variable is None`进行判断

- [python 装饰器](https://www.bilibili.com/video/BV1Gu411Q7JV/?spm_id_from=333.337.search-card.all.click&vd_source=44ff757ed2fdadeadf410d10bde17c2c)，本质上就是一个可以以函数为输入以函数为输出的函数，以`@decorator`形式写在装饰的函数的上方

### 2.14. GitHub

[git 标准工作流程](https://www.bilibili.com/video/BV19e4y1q7JJ/?spm_id_from=333.337.search-card.all.click&vd_source=44ff757ed2fdadeadf410d10bde17c2c)

[获得 github token](https://blog.csdn.net/edsoki/article/details/126913354)

[git 操作介绍(菜鸟)](https://www.runoob.com/git/git-install-setup.html)

[git 版本管理与回退](https://www.bilibili.com/video/BV1ne4y1S7S9/?spm_id_from=333.337.search-card.all.click&vd_source=44ff757ed2fdadeadf410d10bde17c2c)，可以直接用 tag 名称（也可以用 harsh 值作为索引）作为回退，hard 会改变本地，soft 会只改变 commit，mixed 改变 commit 和 add。建议在版本回退之前先`git checkout -b`创建新分支在新分支上回退版本，避免丢失重要的东西。

[.gitignore 设置](https://cloud.tencent.com/developer/article/1640679)

```bash
# ignore all __pychche__
**/__pycache__
```

- 如果想要原本被跟踪的内容不再被跟踪，可以`git rm -r --cached .`把这些文件修改为不被跟踪的状态
- `.gitignore`和直接修改`.git/info/exclude`都能实现对文件的忽略，前者是对所有项目成员都有效，当所有项目成员都不需要版本控制一些文件的时候就用`.gitignore`，一般`.gitignore`也会上传到开源 repo 中，但是如果本地工作路径中有一些文件不需要做版本管理，那么直接按照`.gitignore`的格式将这些文件添加到`.git/info/exclude`中就好了，[参考 1](https://blog.csdn.net/qq_28505809/article/details/124946867)，[参考 2](https://www.jianshu.com/p/f42254bc3ffb)

[本地 github 传不上去的问题解决](https://blog.csdn.net/Xminyang/article/details/124837086)貌似是一个由于代理设置导致的问题

### 2.15. 其他

[VSCode 图标含义](https://www.zhihu.com/question/370258254)

[简单代码在线测试工具](https://app.datacamp.com/)

[VSCode 中使用 Black Formatter 和 isort](https://medium.com/mlearning-ai/python-auto-formatter-autopep8-vs-black-and-some-practical-tips-e71adb24aee1)，如果需要调整参数设置，可以在 arg 中设置，如`-l 100`设置每行最多的字符数；另外如果需要强制换行的，在最后多加一个`,`即可，formatter 会自动形成换行

### 2.16. 简单 linux

- `which`命令输出当前命令的对应路径
- 后台任务相关
  - `nohup python -u my.py > log.txt 2>&1 &`后台挂起程序运行，关闭 terminal 也生效，其中`-u`参数是为了 python 能实时写入，`log.txt`是最后输出日志文件的地方
  - 挂起 R 脚本的时候用的是`Rscript`，即`nohup Rscript rscript.R > log.txt 2>&1 &`，不需要像 python 一样指定参数，会实时写入
  - `kill`+进程 PID 可以终止进程，一般`kill`就直接终止了，如果用`kill --STOP`则会暂停任务
  - 查看后台运行情况（静态）的命令`ps aux`，[具体参数](https://www.cnblogs.com/5201351/p/4206461.html)
  - 可以用`top`命令实时观察后台进程的运行情况
  - `ps -ef|grep python|grep -v grep|grep -v jupyter|cut -c 9-15|xargs kill -9`尝试关闭所有多进程的任务（9-15 是 PID），多进程中杀死父进程子进程还是会继续运行并且占用资源，所以要一起杀光
- 貌似使用`apt`在线下载软件和使用`dpkg -i`安装下载好的`.deb`文件，最后都是用 apt 管理的
- ***

## 3. 老版本(不用 rpy2 版本)的相关记录

### 3.1. 未来的下一步功能:

- [ ] 无法实现`fit marginal`中的`use bam`功能
- [ ] 边际分布函数族只包括 binomial，gaussian，poison，negative binomial，两种 zeroinflated 分布没做
- [ ] `fit marginal`使用的`statsmodels`的所有变量只能选择一种样条基函数(BSplines/CyclicCubicSplines)进行拟合
- [ ] 平滑超参数选择只能使用 gcv 方式选择
- [ ] 目前不支持`gamlss`模式
- [ ] 目前不支持`bam`模式
- [ ] 目前不支持 fit smooth surface，即无法考虑空间转录组数据(受限于`statsmodels`提供的功能有限)

### 3.2. Questions

- [x] vine copula 需要我手动去进行数据标准化吗(在 pyvinecopulib 中需要手动进行调整)
- [x] copulas package vine 在高于 3.8 的版本的应用
- [x] 用哪一种 criteria 进行超参数选择，默认是 AIC，需要换成 GCV 吗还有就是 CV 和 BIC (选 gcv 方法)

### 3.3. scDesign3 相关

#### 3.3.1. construct data

- [x] 一个 numeric 的 varible 就 fit kernel density estimation
- [x] pesudotime NA 是正常的，但是要 check 一下会不会报错(<b>会报错，需要解决，方案为直接退出程序让用户检查</b>)
- [x] warning 信息变为可选择性弹出
- [x] 尝试用 pyvinecopula 来 fit vine copula(<b>优先考虑使用 pyvinecopulib，该包与 R 中的 rvinecopulib 是相同的一个 C++库 vinecopulib 支持</b>)

#### 3.3.2. fit marginal

- mu sigma 都是只有~之后的 formula
- use bam bam 就是 gam，只是 for big data
- sigma 部分可以先留空，因为现在不用
- log 是记录 warning
- 183-210 去掉一部分不要的 cell，某一个 category 中如果有 gene 没有一个 cell 表达，会把这些 cell 去掉（而不是把 gene 去掉）
- mgcv 是 gam 的 package name
- `statsmodels`和`pygam`共性问题是支持的样条函数都比较少

|            `statsmodels`            |                 `pygam`                  |
| :---------------------------------: | :--------------------------------------: |
|           支持所有函数族            |                不支持 nb                 |
|       所有变量只能用一个样条        |        不同变量可以使用不同的样条        |
|              bs 和 cc               |     ps 和 cp(不知道是不是真的实现了)     |
| 不支持 te，不支持一切复杂一点的操作 | 支持 te，无法实现 mgcv 中 s(x1,x2)的效果 |
|             不支持 bam              |                不支持 bam                |

- 输出数据结构

```python
{
    'gene_name':{
        'fit': marginal_dis_model,
        'warning': {'warning_num': {
            'func_name':
            'type':
            'message':}}
        'time': fit_running_time,
        'removed_cell':
    }
}
```

- [x] 需要给用户调整样条函数的基函数和阶数的接口
- [x] 所有的 warning 返回给用户有一个 parameter
- [x] 记录时间，设置一个 para，如果用户需要就返回
- [x] patsy 无法 pickle，因此需要先更改数据阵的存储形式(用 patsy 生成数据阵后转换为 pandas 即可)再输入模型
- [ ] select penalty 运行速度奇慢无比，目前直接不做这个 select 了(调整一下优化方式，以后再选一个合适的算法，可能要大改包内的结构)
- [ ] statsmodels 需要广泛测试一下几个函数族能不能正确拟合出来，可能有一些会有问题，比如 binomial
- [ ] 加一个接口让用户可以调整一下各个 family 的参数设置

测试：

- [ ] alpha 和 gamma 的差距和区别（如何调出 R 中的 gamma 的数值？）
- [x] 比较 aic 的差距(自己和自己不带搜索，和 mgcv)
- [x] mgcv 中 negative binomial 的计算方法(在 statsmodels 中 $var=\mu+\alpha\mu^2$，在 mgcv 中 $var=\mu+\mu^2/\theta$，但是在 mgcv 中会优化这个 theta 参数使得尽量 dispersion 为 1，在 statsmodels 中是一个超参数)
- [x] mgcv 中的 select penal 的优化算法是什么（牛顿法）
- [x] 测试的 R 尽量与 python 对齐
- [x] 看看 statsmodel 超参数搜索为毛这么慢
