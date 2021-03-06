[TOC]
# pandas使用记录#

## 小知识

### pandas对于read_csv比read_excel要快。

### xlsx格式可以保存DataFrame的时间格式，csv无法保存。时间格式导出到csv再回去时会变成str格式。

### 批量处理指定文件夹的所有文件及其子目录下的文件

```python
import pandas as pd
import os
#地图数据转换
for root,dirs,files in os.walk('/home/weblogic/DATA/public/cc热点大数据/工单机构分布/工单机构分布（综合）/'):
    for filespath in files:
        #root为路径，files为文件列表
        print(os.path.join(root,filespath))
        #没有找到.csv会返回-1，否则返回位置
        if filespath.find('.csv')>-1:
            df=pd.read_csv(os.path.join(root,filespath),encoding="gb18030")
            #用os.path.join合成路径
            df1.to_csv(os.path.join('/home/weblogic/DATA/private/shangguanxf/cc_bigdata/create/工单机构分布/标准化',filespath),encoding='gb18030')
```

### 批量处理指定文件夹的文件

```python
import pandas as pd
import os
for entry in os.scandir(path):
   if  entry.is_file():
       print(entry.name)
```

### 使用sample对数据进行重采样

```python
DataFrame.sample(n=None, frac=None, replace=False, weights=None, random_state=None, axis=None)
#n 采样数据个数
#frac 采样比例
#replace 采样是否替换原有数据
#weights 采样权重，需与原df列同样长。缺失值被视为0。

#通过设置采样比例为1，对数据进行重新排序。
df=df.sample(frac=1)
df.reset_index(inplace=True, drop=True)

>>> s.sample(n=3)
27   -0.994689
55   -1.049016
67   -0.224565
dtype: float64
```

### iloc和loc

pandas中iloc是用来对位置索引进行操作，loc是用行索引进行操作。

### df.series.str.split(expand=True)

在`df.serise.str`中的`split`比`python`的`str`类型的`split`多了`expand`参数，可以通过将该参数设置为True，来使分割后的字符扩展成多列而不是一个列表。

```python
import pandas as pd
df = pd.DataFrame({'Country':['China','US','Japan','EU','UK/Australia', 'UK/Netherland'],
               'Number':[100, 150, 120, 90, 30, 2],
               'Value': [1, 2, 3, 4, 5, 6],
               'label': list('abcdef')})

df.Country.str.split('/',expand=True)
```

### 空表可以进行df的方法使用

### apply可以对groupby后的数据进行操作

```python
#取分组后top5
report2.groupby('二级机构').apply(lambda x:x.sort_values('渠道出险次数',ascending=False).head())
```

### 划分训练、测试集

```python
#数据本身分布为随机分布，且数量足够大，可以直接按照索引切分
dfl=len(df)
train=df.iloc[range(int(dfl*0.7)),:]
test=df.iloc[range(int(dfl*0.7),dfl),:]

#数据分布为有序，可以进行抽样
train=df.sample(frac=0.7)
test_index=set(df.index)-set(train.index)
test=df.loc[test_index,:]

#数据分布符合时间规律，直接按照时间进行划分
df.date=pd.to_datetime(df.date,format='%Y/%m/%d')
train=df[df.date<'2017/05/01']
test=df[df.date>'2017/04/30']
```

### 在对df或series进行赋值时，注意添加`.copy()`

如果赋值时不加.copy(),会指向原始对象的内存块，更改新的对象时也会使源对象改变。

```python
import pandas as pd
df1=pd.DataFrame({'a':[1,1,1,1,1],'b':[2,2,2,2,2]})
df2=df1
df2.a=2
df1
->
a	b
0	2	2
1	2	2
2	2	2
3	2	2
4	2	2
```



## DataFrame常用操作

### 数据探查

```python
#得到数值型，object以及混合类型变量的统计信息
df.describe(percentiles=None, include=None, exclude=None)
#percentiles 输出中包含的百分位数。 全部应该在0和1之间。默认值为[.25，.5，.75]，返回第25，第50和第75百分位数。
#include 结果包含数据类型的白名单。有三种设置：
'all' 输入的所有列都将包含在输出中。
类型列表 将结果限制为提供的数据类型。 将结果限制为数字类型，提交numpy.number。 要将其限制为分类对象，请提交numpy.object数据类型。 字符串也可以以select_dtypes的形式使用（例如，df.describe（include = ['0']））
'None' 默认，结果包含所有数值类型
#exclude 从结果中忽略的黑名单数据类型。 忽略了系列。 以下是选项：
类型列表 从结果中排除提供的数据类型。 选择数字类型submit numpy.number。 要选择分类对象，请提交数据类型numpy.object。 字符串也可以以select_dtypes的形式使用（例如，df.describe（include = ['0']））
'None' 默认，结果将不排除任何类型


#得到表的空值，类型及内存大小
DataFrame.info(verbose=None, buf=None, max_cols=None, memory_usage=None, null_counts=None)
#verbose{None,True,False} 是否打印完整的摘要。 None跟随display.max_info_columns设置。 True或False覆盖了display.max_info_columns设置。
#buf 可写缓冲区，默认为sys.stdout
#max_cols 数值类型或None 确定是否打印完整的摘要或简短摘要。 None则跟随display.max_info_columns设置。
#memory_usage 指定是否显示DataFrame元素（包括索引）的总内存使用情况。 None则跟随display.memory_usage设置。 True或False覆盖了display.memory_usage设置。 深刻的“deep”值相当于True， 内存使用情况以人类可读的单位（基数2表示）显示。
#null_counts 是否显示非空计数
如果None，则只显示df中小于max_info_rows和max_info_columns。
如果为True，总是显示计数。
如果False，永不显示。
```

### 建立Series

```python
pandas.Series(data=None, index=None, dtype=None, name=None, copy=False, fastpath=False)
#data 为数据
#index为索引。用在apply中的函数中时，index即为合并后产生的列名
#dtype 设置数据的类型
#copy 是否覆盖输入数据

pd.Series({'log_id':None, 'timelong':None, 'device':None, 'log_from':None, 'ip':None, 'city':None, 'result':None,'timestamp':None, 'type':None, 'is_scan':None, 'is_sec':None, 'time':None})

col=['test'+str(i) for i in range(5)]
pd.Series(np.zeros(5),index=col,dtype='int8')
```

###建表

**注意：**建表时可通过index参数设置索引，`df=pd.DataFrame(columns=col,index=col)`可建立行列对称矩阵

**注意：**建表时用dict赋值时（`pd.DataFrame({'文件名':file,'客服':cc,'客户':customer})`），列名会倒排，df中的列名为（`客户  客服  文件`)。可以使用`columns=`参数来强制指定排序。

```python
#先建立表，再逐步赋值
df1=pd.DataFrame(columns=('机构名称','minmax变换乘100','工单/客户数','该机构工单量','有效客户数'))
#用loc为表赋值
df1.loc[n]=[df.机构名称[i],m3,df.工单机构分布[i],df.该机构工单量[i],df.有效客户数[i]]

#通过直接赋值的方法建立表
df2 = pd.DataFrame({ 'A' : 1,
                     'B' : pd.Timestamp('20130102'),
                     'C' : pd.Series(1,index=list(range(4)),dtype='float32'),
                     'D' : np.array([3] * 4,dtype='int32'),
                     'E' : pd.Categorical(["test","train","test","train"]),
                     'F' : 'foo' })
#生成离散值列
df3 = pd.DataFrame({ 'A' : 1,
                     'G' : pd.np.random.choice(['lol','dota','cf'],10)})

#通过列直接建表
#此法只可建一列，第二列为index，当有多列值时和上面一样，需要用字典去赋值。
a=[1,2,3]
b=['n','n','m']
df4=pd.DataFrame(a,b)
#pandas.DataFrame(data=None, index=None, columns=None, dtype=None, copy=False),故df4中a为列，b为索引。如果想a,b均为列，需要加{}，如df2那般
```

### 数据赋值

建议使用loc和iloc进行赋值

```python
#通过索引进行数据赋值
df.loc[0,['A','B']=[2,3]
#at和loc等价
df.at[0,['A','B']=[2,3]
      
#通过位置进行数据赋值
df.iloc[0:3,2]=3
#iat和iloc类似，但行列只能单选
df.iat[0:3,2]=3
```

### 数据复制

**注意：**浅拷贝会和原始数据共享数据

```python
df.copy(deep=True)
#复制这个对象的索引和数据。
#deep 当deep = True（默认值）时，将使用调用对象的数据和索引副本创建一个新对象。 数据或副本索引的修改不会反映在原始对象中（请参阅下面的注释）。当deep = False时，将创建一个新对象而不复制调用对象的数据或索引（只复制对数据和索引的引用）。 对原稿数据的任何更改都会反映在浅拷贝中（反之亦然）。
>>> s = pd.Series([1, 2], index=["a", "b"])
>>> deep = s.copy()
>>> shallow = s.copy(deep=False)

>>> s[0] = 3
>>> shallow[1] = 4
>>> s
a    3
b    4
dtype: int64
>>> shallow
a    3
b    4
dtype: int64
>>> deep
a    1
b    2
dtype: int64
```

### 按照条件更改列的值

```python
#使用.loc来按照条件更改列的值
df.loc[df['A'] > 2, 'B'] = new_val

df=pd.DataFrame()
a=np.array([0.3,0.4,0.5,0.6,0.7])
df['a']=a

df['b']=a>0.5
df['b']=df['b'].astype('int8')

df['c']=0
df.loc[df['a']<0.5,'c']=1
```

### 数据选择

通过标签（df.loc）选择时行[1:3]表示[1,2,3],通过位置(df.iloc)进行选择时行[1:3]表示[1,2]

当对指定列选取最后一行时，`df.col[-1]`会报错，无法这样取数，需要使用iloc来取数`df.col.iloc[-1]`

```python
import pandas as pd
import numpy as np
df=pd.DataFrame({ 'A' : 1,
                     'B' : pd.Timestamp('20130102'),
                     'C' : pd.Series(1,index=list(range(4)),dtype='float32'),
                     'D' : np.array([3] * 4,dtype='int32'),
                     'E' : pd.Categorical(["test","train","test","train"]),
                     'F' : 'foo' })
#按列选择,选择多列时需用括号
df[['A','B']]

#按行选择,0:3表示[0,1,2]
df[0:3]

#通过布尔值选择
df[df.E=='test']
#用isin进行筛选
df[df.E.isin(['test','train'])]

#通过标签获得行列,0:3为行标签即index，['A','B']为列名
注意：此时0:3得到的是[0,1,2,3]
df.loc[0:3,['A','B']]
df.loc[0]<==>df.loc[0,:]#两者结果相同
#loc行可以为bool值
df.loc[df.E=='test',['A','B']]


#通过位置获得行列,0:4为行位置，0:2为列位置
注意：行列都是从0开始计数
注意：此时0:4=[0,1,2,3],0:2=[0,1]
df.iloc[0:4,0:2]
#单独取一列时可直接使用该列位置.直接取C列的值
df.iloc[0:4,2]

```

### 获得指定数据类型的变量

```python
DataFrame.select_dtypes(include=None, exclude=None)
#选择其中一个，以列表的形式传入所需或不需要的变量类型
#include 包含的类型
#exclude 不包含的类型

>>> df = pd.DataFrame({'a': np.random.randn(6).astype('f4'),
...                    'b': [True, False] * 3,
...                    'c': [1.0, 2.0] * 3})
>>> df
        a      b  c
0  0.3962   True  1
1  0.1459  False  2
2  0.2623   True  1
3  0.0764  False  2
4 -0.9703   True  1
5 -1.2094  False  2
>>> df.select_dtypes(include=['float64'])
   c
0  1
1  2
2  1
3  2
4  1
5  2
>>> df.select_dtypes(exclude=['floating'])
       b
0   True
1  False
2   True
3  False
4   True
5  False
```

### 行索引及多重行索引

```python
import pandas as pd
#单列索引
df=pd.DataFrame(columns=['age','gender','name'])
#利用索引新添数据，1,2,3为当前行索引
df.loc[1]=[18,'female','July']
df.loc[2]=[19,'male','Jeo']
df.loc[3]=[12,'female','Lily']
df.loc[4]=[18,'male','Jack']
#设置索引
df.set_index('age',inplace=True)
#选择数据
df.loc[19]
>>
gender    male
name       Jeo
Name: 19.0, dtype: object

#多列索引
df.set_index(['age','gender'],inplace=True)
#选择数据
df.loc[(18,'female')]
>>
name    July
Name: (18.0, female), dtype: object
#利用查询来筛选数据
df.query('age==18')
>>
		        name
age	    gender	
18.0	female	July
        male	Jack
```

### 得到行索引

通过.name得到行索引属性

```python
df=pd.DataFrame(columns=['age','gender','name'])
#利用索引新添数据，1,2,3为当前行索引
df.loc[1]=[18,'female','July']
df.loc[2]=[19,'male','Jeo']
df.loc[3]=[12,'female','Lily']
df.loc[4]=[18,'male','Jack']
#多列索引
df.set_index(['age','gender'],inplace=True)

df.iloc[0,:].name
```

### 得到最后一个有效的索引

```python
df=pd.DataFrame({'a':[1,2,3,4,5]})
df.last_valid_index()
```

### 重置索引

在使用pd.concat之后需要重置索引，否则df.iloc[0,1]返回合并前每个表的[0,1]

```python
#设置索引，将特定列设置为索引
df.set_index(keys, drop=True, append=False, inplace=False, verify_integrity=False)
#keys 列名或列名列表
#drop 是否删除用来做索引的列，默认删除
#append 是否保留已有的索引
#inplace 是否替换原df
#verfy_intergrity 是否立即检查新索引的重复性。False则推迟检查，直到必要时。设置为False将提高此方法的性能


#重置索引
df.reset_index(level=None, drop=False, inplace=False, col_level=0, col_fill='')
#level {int, str, tuple}只移除指定的索引级别，默认移除全部
#inplece 是否替代替原df
#drop 是否删除原索引
#col_level	{int,str}当col有多级列索引时，将重置后的索引放置的级别
#col_fill	当有多级列索引时，重置后索引未放置的级别填充的列名


df.reset_index(inplace=True, drop=True)


#重新排列索引
df.reindex(new_index)
#将原来的索引按照新索引进行排列，如果新索引在原索引中不存在，返回NaN。如果想要生成新的索引，使用reset_index()
```

### 索引排序

```python
df.sort_index(axis=0, level=None, ascending=True, inplace=False, kind='quicksort', na_position='last', sort_remaining=True, by=None)
#axis=0 按列排序
#level 排序等级，可以为索引等级（int）或索引名称（level name）或者为list格式
#ascending 是否正序排列
#inplace 是否替代原数据
#kind 排列方式{‘quicksort’, ‘mergesort’, ‘heapsort’}
#na_position {'first', 'last'}Nan的数据摆放位置，first为放在数据前面，last为放在末尾
#sort_remaining 如果为真，并按级别和索引排序为多级，则按照指定级别排序后，按其他级别排序（按顺序排列）
```

### 索引操作

```python
#取交集
df.index.intersection(df1.index)
#取并集
df.index.union(df1.index)
#取差集
df.index.diff(df1.index)
```

### 得到列名

```python
df.column[0]
#得到第0列的列名
```

### 更改列名

   ```python
   import pandas as pd
   df=pd.DataFrame({'A':[1,2,3], 'B':[4,5,6], 'C':[7,8,9]})

   #方法一，需要一次更改全部列名
   df.columns=['a','b','c']
   #方法二,更改指定列名
   df.rename(columns={'a':'A'},inplace=True)
   ```

### 插入列

```python
#直接在最后添加列
df=pd.DataFrame()
df['列']=a

#在指定位置插入列
DataFrame.insert(loc, column, value, allow_duplicates=False)
#loc插入的位置，在此列及之后的列向后顺延
#column插入列的列名
#value插入的值
#allow_duplicates=False即不允许插入列与已有列名重复
df.insert(9,'班次',d)
```

### 列扩行

将多列的数据转换为一列，列名变为二级行索引名。

可和`df.col.str.split()`搭配使用

```python
DataFrame.stack(level=-1, dropna=True)
#level {int, string, list}将列转换为行后的列名的行索引级别，默认为最后一级
#dropna 是否去掉空值

df=pd.DataFrame({'a':[1,2],'b':[3,4]})
df.stack()

#一般用于将split(expand=True)转为列
new_df=df.drop('Country', axis=1).join(df['Country'].str.split(',', expand=True).stack().reset_index(level=1, drop=True).rename('Country'))
```

### 行扩列

```python
unstack(level=-1, fill_value=None)
#level 拆分成行的索引名称
#fill_value	拆分后的nan填充值


>>> index = pd.MultiIndex.from_tuples([('one', 'a'), ('one', 'b'),
...                                    ('two', 'a'), ('two', 'b')])
>>> s = pd.Series(np.arange(1.0, 5.0), index=index)

s.unstack(level=-1)
     a   b
one  1.0  2.0
two  3.0  4.0

s.unstack(level=0)
   one  two
a  1.0   3.0
b  2.0   4.0
```

### 删除行列

```python
#返回一个删除行/列的新的DataFrame
DataFrame.drop(labels, axis=0, level=None, inplace=False, errors='raise')
#labels标签名或标签名的列表
#axis轴，当label为列名时，axis=1
#implace是否代替原Dataframe，默认为False

#删除列

del df['列名']

df.drop([df.columns[0]], axis=1,inplace=True)
#df.column[0]得到第0列的列名

#删除行
df.drop(0)
#删除标签为0的行

df.drop(0,inplace=True)
#删除行标签为0的行，并代替原表
```

### 排序列

```python
pandas.DataFrame.sort_values(by, axis=0, ascending=True, inplace=False, kind='quicksort', na_position='last')
#by 排序所依赖的轴的名称或名称列表
#axis=0 按什么轴进行排序，0为行，1为列。默认为0
#ascending 是否升序，应为bool或bool列表。默认为True升序
#inplace 是否替换原表。默认为否
#kind 排序方法，有{‘quicksort’, ‘mergesort’, ‘heapsort’}，默认为'quicksort'
#na_position NaN的位置,有{‘first’, ‘last’}，放在开始还是结尾，默认放结尾

import numpy as np
import pandas as pd
df = pd.DataFrame({'A' : ['foo', 'bar', 'foo', 'bar', 'foo', 'bar', 'foo', 'foo'],
'B' : ['one', 'one', 'two', 'three','two', 'two', 'one', 'three'],
'C' : np.random.randn(8),'D' : np.random.randn(8)})
#按照C列排序，递增为关
df.sort_values('C',ascending=False)
#按照b，c列排序
df.sort_values(['B','C'])
```

### 更改列类型

更改类型需要使用astype()函数，有三种使用方法。astype()里可以用numpy的数据类型，也可以直接用'int8'类型的字符串形式。

**注意：**用第二种方法更改部分列时，即使只有一列也需要用`dft[['a']]`，因为`dft.a`和`dft['a']`都得到的是array形式，`dft[['a']]`得到的是df格式。只有df格式才能用astype。

**注意：**当对数据进行格式更改时，注意index需要唯一，即在pd.concat()后需要做df.reset_index然后再做astype的格式转换。

```python

dft = pd.DataFrame({'a': [1,2,3], 'b': [4,5,6], 'c': [7, 8, 9]})
#全表更改类型
#不会直接更改原表，需要赋值回去
dft=dft.astype(np.int16)
dft=dft.astype('int16')

#更改部分列
#不会直接更改原表，需要赋值回去
dft[['a','b']] = dft[['a','b']].astype(np.uint8)

#用loc更改部分列，容易出现异常，导致列格式无法传递回去,要避免使用第三种方法
dft.loc[:, ['a', 'b']].astype(np.int16).dtypes
>
a    int16
b    int16
dtype: object
    
dft.loc[:, ['a', 'b']] = dft.loc[:, ['a', 'b']].astype(np.int16)
dft.dtypes
>    
a    int64
b    int64
c    int64
dtype: object
```

### 得到重复值

```python
df.duplicated(subset=None, keep='first')
#subsets 为考虑是否重复的列，None则考虑全表
#keep为重复值标签方式{'first','last',False}
#first:重复值中第一个为false，即不视为重复值
#last：重复值中最后一个为false，即不视为重复值
#False：所有重复值均为True。
```

### 去除重复值

```python
infor2.drop_duplicates(subset,keep,inplace)
#subset 为所需的列标签或行标签，可以一个，也可以是标签的列表
#keep的值有三种{‘first’, ‘last’, False},默认为‘first’
#first : 保留重复值中的第一个
#last : 保留重复值中的最后一个
#False : 删除所有重复值
#inplace=True代替原df，默认为False
infor2.drop_duplicates(['工作所在地','性别'],False,inplace=True)
#删除所有性别，工作所在地重复行
```

### 得到inf

pandas 无法直接进行inf的查询，需要使用numpy.isinf来进行查找，得到inf的值

```python
#判断是否为inf
np.isinf(df)
#得到包含inf的行
df[np.isinf(df).values==True]
```

### 得到nan

pandas 提供isnull和notnull两种方法。isnull是为nan返回True，notnull是nan返回False。

```python
df1=pd.DataFrame({'A':range(5),'B':[np.nan,np.nan,np.nan,3,1],'C':np.random.rand(5)})
#使用pd函数
pd.isnull(df1.B)
pd.notnull(df1.B)

#使用df的函数
df1.B.isnull()
df1.B.notnull()
#找到值为空的行
df[df.isnull().values==True]
df[df.notnull().values==False]
```

### 去除nan

```python
dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)
#axis 按行还是按列去除，axis=0，按行删，axis=1按列删
#how{'any','all'} 删除条件，'any'删除所有存在空值的行/列，'all'当该列或行全为空时删除
#thresh 不被去除需要的最少非空值数
#subset 只根据subset参数里的列进行操作
#inplece 是否代替原数据

df1=pd.DataFrame({'A':range(5),'B':[np.nan,np.nan,np.nan,3,1],'C':np.random.rand(5),'D':[1,2,3,4,np.nan]})
df1
	A	B	       C	D
0	0	NaN	0.494810	1.0
1	1	NaN	0.001969	2.0
2	2	NaN	0.774776	3.0
3	3	3.0	0.073526	4.0
4	4	1.0	0.532342	NaN
df1.dropna(axis=0,subset=['D'])#只根据列表中的列的空值来决定是否保留，保留方式按how的参数执行
    A	 B	       C	D
0	0	NaN	0.494810	1.0
1	1	NaN	0.001969	2.0
2	2	NaN	0.774776	3.0
3	3	3.0	0.073526	4.0
df1.dropna(axis=0,thresh=4)#只保留非空值大于等于4的行
    A	B	       C	D
3	3	3.0	0.073526	4.0
```

### 填充nan

在使用fillna时，注意value和method为冲突的参数，设置了value则mothod只能用None，使用设置的value去填充所有的空值。设置了模式，就按模式去做填充，而不会使用value。

fillna内部的limit参数应该是针对每个value所能使用的次数，当指定模式时，有多个值（每个空值前/后的有效值），而无指定模式时，实际上是指定模式填充的特例，只用一个值value去填充。

```python
DataFrame.fillna(value=None, method=None, axis=None, inplace=False, limit=None, downcast=None, **kwargs)
#value 用于填充空值的值
#填充时的模式{'backfill', 'bfill', 'pad', 'ffill', None} backfill/bfill:使用后一个有效值对空值进行填充	pad/ffill:使用前一个有效值进行填充
#axis 按行还是列进行填充{0 or 'index', 1 or 'columns'}
#inplace 是否代替原有数据
#limit	当为指定填充模式时，limit为使用前一个/后一个有效数字可以填充的空值最大数目。当没有指定模式时，此值为可以填充的空值的最大数目，当空值多于此值时，只填充从前面开始的limit个空值。limit如果存在，则应为大于0的整数。
#downcast 用于转换列的格式，说明中表示可以用字典和'infer'两种用法，但在0.20中仍然无法使用字典，只能使用'infer'（推断）。好像pandas中自动后调整输入的数据格式，不太需要用到这个。

df = pd.DataFrame([[np.nan, 2, np.nan, 0],[3, 4, np.nan, 1],[np.nan, np.nan, np.nan, 5],[np.nan, 3, np.nan, 4]],columns=list('ABCD'))
#可以通过字典来对针对特定的列的空值赋值
values = {'C': 1,'B':0}
df.fillna(value=values)
```

### 插值nan

其中有封装scipy程序，可参考[scipy documentation](http://docs.scipy.org/doc/scipy/reference/interpolate.html#univariate-interpolation) and [tutorial documentation](http://docs.scipy.org/doc/scipy/reference/tutorial/interpolate.html)

```python
DataFrame.interpolate(method='linear', axis=0, limit=None, inplace=False, limit_direction='forward', downcast=None, **kwargs)
#method 插值的方式{'linear', 'time', 'index', 'values', 'nearest', 'zero','slinear', 'quadratic', 'cubic', 'barycentric', 'krogh', 'polynomial', 'spline', 'piecewise_polynomial', 'from_derivatives', 'pchip', 'akima'}
'linear'：忽视索引并将值视为等距间隔。这是支持多重索引的唯一方法。是method默认的参数
'time'：需要索引为时间索引，按照时间进行插值，具体插值函数会根据情况自行选择。通常为linear，按照时间的先后进行等距填充
'index','value':使用索引的实际数值
'nearest','nearest', 'zero', 'slinear', 'quadratic', 'cubic', 'barycentric', 'polynomial'传递给scipy.interpolate.interp1d。 'polynormal'(多项式)和'spline'(样条)都要求您也指定一个顺序（int），例如。 df.interpolate（method ='polynomial'，order = 4）。 这些使用索引的实际数值。
'krogh', 'piecewise_polynomial', 'spline', 'pchip' and 'akima'都是相似名称的scipy插值方法的包装。 这些使用索引的实际数值。 有关其行为的更多信息，请参阅scipy文档和教程文档 
'from_derivatives'是指用于替代scify中'piecewise_polynomial'插值法的'BPoly.from_derivatives'
增加了对'akima'方法的支持添加了插入方法'from_derivatives'，它替代了scipy中的“piecewise_polynomial”0.18; 向后兼容scipy <0.18
#axis 填充方式 0为逐列填充，1为逐行填充
#limit 去填充连续NaN的最大数量，必须大于0
#inpalce 是否替代原有df
#limit_direction  {'forward', 'backward', 'both'}指定连续NaN填充的方向
```

### 列去重

```python
Series.unique()

df.f1.unique()
->[0, 1, 2]
```

### 统计各轴变量种数

```python
DataFrame.nunique(axis=0, dropna=True)
#axis 进行统计的轴的方向
#是否去除空值

df = pd.DataFrame({'A': [1, 2, 3], 'B': [1, 1, 1]})
>>> df.nunique()
A    3
B    1
```

### 统计各列的值的数目

```python
#利用series.value_counts()计算列中每个值的数目
Series.value_counts(normalize=False, sort=True, ascending=False, bins=None, dropna=True)
#normalize 是否转换成频率，normalize=True则频数除以总频数作归一化处理
#sort 是否排序
#ascending 是否升序 默认ascending=False降序排列
#bins=int 将连续变量分成离散变量
#dropna=True删除空值

#对test列统计值的词数
a=['一','二','一']
df=pd.DataFrame({'test':a})
df.test.value_counts()
#用bins将其分组
a=[1,2,3,4,5,6,1,2,3]
df=pd.DataFrame({'test':a})
df.test.value_counts(bins=2)
#结果
0.995    6
3.500    3
#即将列a分成两个离散变量，(0.995,3.5] 频数为6, (3.5,6]频数为3
```

### 数值分组

cut函数可以用于从连续变量到分类变量。 例如，剪切可以将年龄转换为年龄范围组。

任何NA在结果中仍然是NA。 在所得到的分类对象中，越界值（即使用序列bins时不再分箱中的值）将是NA

```python
pandas.cut(x, bins, right=True, labels=None, retbins=False, precision=3, include_lowest=False)
#x 需要分箱的数组，必须是一维数组
#bins 如果bin是一个int，它定义了x范围内的等宽bin数。 但是，在这种情况下，x的范围在每一边扩展了0.1％，以包含x的最小值或最大值。 如果箱是一个序列，它定义了箱边，允许箱宽度不均匀。 在这种情况下，x的范围没有扩展。
#right 分箱（分组）是否包含右边， 如果right == True（默认），分组表示为（1,2]，（2,3]，（3,4）。
#labels 用作分箱结果的标签。 个数必须与分箱个数相同。 如果为False，则只返回数组在分箱中的位置索引。
#retbins 是否返回分箱节点数组。 如果分箱以标量形式提供，可能会有用。
#precision 分箱的展示精度
#include_lowest 第一个时间间隔是否应该包含在内。

pd.cut(np.array([.2, 1.4, 2.5, 6.2, 10.2, 2.1]), [0,5,10], retbins=True,precision=3,include_lowest=False)
->
([(0, 5], (0, 5], (0, 5], (5, 10], NaN, (0, 5]]
 Categories (2, interval[int64]): [(0, 5] < (5, 10]], array([ 0,  5, 10]))
```

### 值累加

将一列/行的值逐行累加。

```python
DataFrame.cumsum(axis=None, skipna=True, *args, **kwargs)
#axis 按行还是按列，默认按行。0为按行，1为按列
#skipna 是否跳过空值

df=pd.DataFrame({'a':list(range(5)),'b':list(range(5))})

df.cumsum(0)
-->
	a	b
0	0	0
1	1	1
2	3	3
3	6	6
4	10	10
df.cumsum(1)
-->
	a	b
0	0	0
1	1	2
2	2	4
3	3	6
4	4	8
```

### 数据合并

数据合并有两种，一种是行数不变，合并列。一种是列不变，合并行

#### concat

```python
#列不变，增加行（合并行）
df2=pd.concat([df,df1])
#df和df1的列名及排列要一致
```

#### merge

```python
#行不变，增加列（合并列）
df4=pd.merge(df3,df1,how='inner',on=None,left_on=['投保人姓名','保单号'],right_on=['投保人姓名','保单号']
#根据多列去做关联，需要两列都相同才关联
#how有'left','right','inner'三种，用法和sql中一致
#on为关联时匹配的列，当只有一列时用on，多列时on=None，用left_on和right_on来关联
             
main=pd.merge(infor4,infor2,how='left',on='姓名')
#用名字去关联，左边保持不变，右边和左边相同时匹配过去
             
#根据索引去匹配
#将left_index和right_index设为True
df1=pd.merge(df1,df2,how='left',on=None,left_index=True,right_index=True)   

#合并时如果有重名的列名，使用suffixes参数设置,默认为suffixes=('_x', '_y')
df=pd.merge(df,df1,how='left',on=None,left_index=True,right_index=True,suffixes=('_1', '_2'))
```

#### join

```python
#行不变，增加列，比较适合操作同一个df中的不同列
DataFrame.join(other, on=None, how='left', lsuffix='', rsuffix='', sort=False)
#other 有相似索引的DataFrame, Series with name field set, or list of DataFrame
#on 默认使用index做关联，也可以使用列名或者列名列表
#how 结合方式。{'left','right','outer','inner'}
#lsuffix 重叠列中左边所用的后缀
#rsuffix 重叠列中右边所用的后缀
#sort 是否按照join key排序，如果为False，依赖于join key的类型

caller = pd.DataFrame({'key': ['K0', 'K1', 'K2', 'K3', 'K4', 'K5'],
...                        'A': ['A0', 'A1', 'A2', 'A3', 'A4', 'A5']})
other = pd.DataFrame({'key': ['K0', 'K1', 'K2'],
...                       'B': ['B0', 'B1', 'B2']})
caller.join(other, lsuffix='_caller', rsuffix='_other')
```

### dict转DataFrame

dict转DataFrame的三种方法。转成列表再导入速度最快。

```python
#用for循环主键，依次导入。用时：45.9755654335022   
df1=pd.DataFrame(columns=('word','value'))
n=0
for i in one_hot.keys():
   df1.loc[n]=[i,one_hot[i]]
   n=n+1
    
#将dict转成列表，导入list。用时：0.010346174240112305
df1=pd.DataFrame(list(one_hot.items()), columns=['Date', 'DateValue'])

#将dict直接导入，再重命名列名。用时：0.024332761764526367
df1=pd.DataFrame({'value':one_hot})
df1.index.name='key'
df1=df1.reset_index()
```

### df中str函数使用

python str本身自带了很多函数。在pandas中使用需要在str类型的series后加str。**不加.str将无法有效进行替换。**

 ```python
   infor['日期']=infor['日期'].str.replace('-','')#字符替换
    
   df1.通话文本.str.count('。')#字符计数
   
   df1.通话文本.str.len()#得到每一行的字符长度

   df1.通话文本.str.find('中')#得到第一个‘中’（查询的字符串）出现的位置
 ```

### str类型的series转int类型

将str的series转换为int的方法。

```python
#使用pandas自带函数
df1.录音编号=pd.to_numeric(df1.录音编号)
#使用numpy类型转换
df1.录音编号=np.int64(df1.录音编号)
```

## 文件读存

### csv

```python
#csv读取
pd.read_csv(filepath_or_buffer, sep=', ', delimiter=None, header='infer', names=None, index_col=None, usecols=None, squeeze=False, prefix=None, mangle_dupe_cols=True, dtype=None, engine=None, converters=None, true_values=None, false_values=None, skipinitialspace=False, skiprows=None, nrows=None, na_values=None, keep_default_na=True, na_filter=True, verbose=False, skip_blank_lines=True, parse_dates=False, infer_datetime_format=False, keep_date_col=False, date_parser=None, dayfirst=False, iterator=False, chunksize=None, compression='infer', thousands=None, decimal=b'.', lineterminator=None, quotechar='”', quoting=0, escapechar=None, comment=None, encoding=None, dialect=None, tupleize_cols=False, error_bad_lines=True, warn_bad_lines=True, skipfooter=0, skip_footer=0, doublequote=True, delim_whitespace=False, as_recarray=False, compact_ints=False, use_unsigned=False, low_memory=True, buffer_lines=None, memory_map=False, float_precision=None)
只列出常用参数，其余用时查询。
#filepath_or_buffer 文件地址
#sep设置分隔符
#engine : {‘c’, ‘python’}, 选用引擎，c速度快，python功能更丰富。 
#encoding 编码格式，默认为utf-8


#常用写法
df=pd.read_csv('/home/weblogic/DATA/public/cc热点大数据/太寿工单/太寿工单.csv',encoding='gb18030')

#csv写出
df.to_csv(path_or_buf=None, sep=', ', na_rep='', float_format=None, columns=None, header=True, index=True, index_label=None, mode='w', encoding=None, compression=None, quoting=None, quotechar='”', line_terminator='\n', chunksize=None, tupleize_cols=False, date_format=None, doublequote=True, escapechar=None, decimal='.')
只列出常用参数，其余用时查询
#path_or_buf 保存文件位置
#sep 设置分割符
#header 是否保留列名，默认header=True保留列名
#index 是否保留索引，默认index=True保留索引
#encoding 编码格式

#常用写法.因为read_csv读取时会自动生成索引，所以生成时关闭索引
df.to_csv('/home/weblogic/DATA/private/shangguanxf/cc_copy/create/10-3/寿险投诉机构工单.csv',index=False)
```

#### 读取的csv没有列名

```python
df=pd.read_csv(os.path.join(root,filespath),encoding="gb18030",header=None,names=['机构名称','工单客户比'])
#注意，设置列名时需header=None
```

#### 读取csv更改列名

跳过第一行的列名，再设置新的列名。在0.22.0中header有了`'infer'`推断,,无需设置header为None。

```python
 training_set = pd.read_csv("boston_train.csv", skipinitialspace=True,skiprows=1, names=columns)
 #columns 为列名的list
```

#### 读取csv有不规范数据，有的行中列和其他行不统一

通过设置 error_bad_lines=True, warn_bad_lines=True来决定是否遇到不合规范的行数进行报错和警告。

**注意：**如果error_bad_lines=False,虽然不会报错，但会跳过不合规范的数据

```python
CParserError: Error tokenizing data. C error: Expected 10 fields in line 45, saw 11
报错：数据期待为10列，但是在45行见到了11列

df=pd.read_csv('/home/weblogic/DATA/private/shangguanxf/cc_txt1/warning.txt',encoding='gb18030',error_bad_lines=False)
#遇到数据格式错误会不进行报错，但仍会跳出警告
#报错的数据会被跳过
```

### excel

**注意：**不要把sheetname写成sheet_name，并没有下划线

```python
#读取excel
pd.read_excel(io, sheetname=0, header=0, skiprows=None, skip_footer=0, index_col=None, names=None, parse_cols=None, parse_dates=False, date_parser=None, na_values=None, thousands=None, convert_float=True, has_index_names=None, converters=None, dtype=None, true_values=None, false_values=None, engine=None, squeeze=False, **kwds)
只列出常用参数，余下用时查询
#io 文件位置
#sheetname 选择所用的sheet，可用string或int。string则应为所取sheet名。int则是sheet位置，0为第一个sheet。当取多个sheet时，使用列表[]。如果为None，则取所有的sheet
#header 列名位置，默认为0
#names 要使用的列名列表。 如果文件不包含标题行，应设置header = None

#常用写法
df1=pd.read_excel('/home/weblogic/DATA/private/shangguanxf/cc_copy/1-3/寿险预约与咨询工单1_3.xls','Sheet1')

#保存excel
df.to_excel(excel_writer, sheet_name='Sheet1', na_rep='', float_format=None, columns=None, header=True, index=True, index_label=None, startrow=0, startcol=0, engine=None, merge_cells=True, encoding=None, inf_rep='inf', verbose=True, freeze_panes=None)
#excel_writer 保存文件位置
#sheet_name 表格名称
#encoding 编码格式

#常用写法
df.to_excel(os.path.join('/home/weblogic/DATA/private/shangguanxf/cc_copy/create/',res_path,filespath1),sheet_name='sheet1')

#将多个sheet写入同一个excel文件中
writer = pd.ExcelWriter('output.xlsx')
df1.to_excel(writer,'Sheet1')
df2.to_excel(writer,'Sheet2')
writer.save()
```

### sql

#### 连接数据库

使用python连接数据库，可以使用sqlalchemy包，通过这个包连接数据库，对里面的数据进行操作。也可以通过pandas内部的函数直接读取数据库中的表。针对oracle，可以使用cx_Oracle包来读取oracle数据。读取oracle数据库出现中文乱码，是由于编码不匹配，需在前面设置编码格式。`'NLS_LANG'`为oracle数据库编码环境。

**注意：**表名需要为小写，大写无法识别。可以使用'T_CONTRACT_MASTER'.lower()来将表名变成小写。

```python
#设置oracle编码格式
import os
os.environ['NLS_LANG'] = 'SIMPLIFIED CHINESE_CHINA.UTF8'
#用sqlalchemy创立连接
from sqlalchemy import create_engine
engine = create_engine('oracle+cx_oracle://shangguan:shangguan_test123@sandbox')

#设置编码格式
import os
os.environ['NLS_LANG'] = 'SIMPLIFIED CHINESE_CHINA.UTF8'
#直接读取数据库中的表
df=pd.read_sql_table('id_infor','oracle+cx_oracle://shangguan:shangguan_test123@sandbox')
#通过schema参数设置表所在空间（即name.table的name）
#df=pd.read_sql_table('tb_cc_record_analysis','oracle+cx_oracle://shangguan:shangguan_test123@sandbox',schema='sandbox_fixed')
#如果要连接其他地址，在@后加ip地址和端口
#cc=pd.read_sql_table('tb_cc_record_analysis','oracle+cx_oracle://shangguan:shangguan_test123@10.1.33.2:1521/sandbox',schema='sandbox_fixed')

#设置编码格式
import os
os.environ['NLS_LANG'] = 'SIMPLIFIED CHINESE_CHINA.UTF8'
#用cx_Oracle读取数据库中的表
import cx_Oracle as ora
import pandas as pd
con = ora.connect('shangguan/shangguan_test123@sandbox')
cur = con.cursor()
cur.execute('select * from cc_text_question')
res = cur.fetchall()[][].read()
data = eval(res)
df = pd.DataFrame(res, columns = ['保单号', '录音编号', '问答文本'])
print(df)
```

#### 写入数据库

通过pandas将数据写入数据库。

```python
DataFrame.to_sql(name, con, flavor=None, schema=None, if_exists='fail', index=True, index_label=None, chunksize=None, dtype=None)

#name 为写入的表名
#con 连接方式
#schema 表所在的空间
#chunksize 如果为空，则一次写入全部。如果不为空，则每次以这样的大小批量写入。
```

## 时间类型处理

参考链接:

[pandas 时间序列操作](http://zqdevres.qiniucdn.com/data/20150813205120/index.html)

导入的包

```python
from datetime import time
from pandas.tseries.offsets import *
from datetime import datetime,timedelta
```

### 时间格式字符串

`%a `星期的简写。如 星期三为Web
`%A `星期的全写。如 星期三为Wednesday
`%b `月份的简写。如4月份为Apr
`%B 月份的全写。如4月份为April `
`%c:  `日期时间的字符串表示。（如： 04/07/10 10:43:39）
`%d:  `日在这个月中的天数（是这个月的第几天）
`%f:  `微秒（范围[0,999999]）
`%H:  `小时（24小时制，[0, 23]）
`%I:  `小时（12小时制，[0, 11]）
`%j:  `日在年中的天数 [001,366]（是当年的第几天）
`%m:  `月份（[01,12]）
`%M:  `分钟（[00,59]）
`%p:  `AM或者PM
`%S:  `秒（范围为[00,61]，为什么不是[00, 59]，参考python手册~_~）
`%U:  `周在当年的周数当年的第几周），星期天作为周的第一天
`%w:  `今天在这周的天数，范围为[0, 6]，6表示星期天
`%W:  `周在当年的周数（是当年的第几周），星期一作为周的第一天
`%x:  `日期字符串（如：04/07/10）
`%X:  `时间字符串（如：10:43:39）
`%y:  `2个数字表示的年份
`%Y:  `4个数字表示的年份
`%z:  `与utc时间的间隔 （如果是本地时间，返回空字符串）
`%Z:  `时区名称（如果是本地时间，返回空字符串）
`%%:  `%% => %

### dt常用函数

#### **Datetime Properties**

| [`Series.dt.date`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.date.html#pandas.Series.dt.date) | 返回日期                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`Series.dt.time`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.time.html#pandas.Series.dt.time) | 返回时间                                                     |
| [`Series.dt.year`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.year.html#pandas.Series.dt.year) | 返回日期的年                                                 |
| [`Series.dt.month`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.month.html#pandas.Series.dt.month) | 返回日期的月，January（一月）=1, December（十二月）=12       |
| [`Series.dt.day`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.day.html#pandas.Series.dt.day) | 返回日期的日                                                 |
| [`Series.dt.hour`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.hour.html#pandas.Series.dt.hour) | 返回时间的时                                                 |
| [`Series.dt.minute`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.minute.html#pandas.Series.dt.minute) | 返回时间的分                                                 |
| [`Series.dt.second`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.second.html#pandas.Series.dt.second) | 返回时间的秒                                                 |
| [`Series.dt.microsecond`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.microsecond.html#pandas.Series.dt.microsecond) | 返回时间的 microseconds （一百万分之1秒）                    |
| [`Series.dt.nanosecond`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.nanosecond.html#pandas.Series.dt.nanosecond) | 返回时间的 nanoseconds （10的负9次方秒）                     |
| [`Series.dt.week`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.week.html#pandas.Series.dt.week) | 一年中的第n个星期                                            |
| [`Series.dt.weekofyear`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.weekofyear.html#pandas.Series.dt.weekofyear) | 一年中的第n个星期                                            |
| [`Series.dt.dayofweek`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.dayofweek.html#pandas.Series.dt.dayofweek) | 一周中的第n天，注意，周一返回为0，周日返回为6.               |
| [`Series.dt.weekday`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.weekday.html#pandas.Series.dt.weekday) | 一周中的第n天，注意，周一返回为0，周日返回为6.               |
| [`Series.dt.weekday_name`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.weekday_name.html#pandas.Series.dt.weekday_name) | 一周中的第n天的英文 (ex: Friday)                             |
| [`Series.dt.dayofyear`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.dayofyear.html#pandas.Series.dt.dayofyear) | 一年中的第n天                                                |
| [`Series.dt.quarter`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.quarter.html#pandas.Series.dt.quarter) | 一年中的第n季度                                              |
| [`Series.dt.is_month_start`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.is_month_start.html#pandas.Series.dt.is_month_start) | Logical indicating if first day of month (defined by frequency) |
| [`Series.dt.is_month_end`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.is_month_end.html#pandas.Series.dt.is_month_end) | Logical indicating if last day of month (defined by frequency) |
| [`Series.dt.is_quarter_start`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.is_quarter_start.html#pandas.Series.dt.is_quarter_start) | Logical indicating if first day of quarter (defined by frequency) |
| [`Series.dt.is_quarter_end`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.is_quarter_end.html#pandas.Series.dt.is_quarter_end) | Logical indicating if last day of quarter (defined by frequency) |
| [`Series.dt.is_year_start`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.is_year_start.html#pandas.Series.dt.is_year_start) | Logical indicating if first day of year (defined by frequency) |
| [`Series.dt.is_year_end`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.is_year_end.html#pandas.Series.dt.is_year_end) | Logical indicating if last day of year (defined by frequency) |
| [`Series.dt.is_leap_year`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.is_leap_year.html#pandas.Series.dt.is_leap_year) | Logical indicating if the date belongs to a leap year        |
| [`Series.dt.daysinmonth`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.daysinmonth.html#pandas.Series.dt.daysinmonth) | 该月的天数                                                   |
| [`Series.dt.days_in_month`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.days_in_month.html#pandas.Series.dt.days_in_month) | 该月的天数                                                   |
| [`Series.dt.tz`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.tz.html#pandas.Series.dt.tz) |                                                              |
| [`Series.dt.freq`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.freq.html#pandas.Series.dt.freq) |                                                              |

#### **Datetime Methods**

| [`Series.dt.to_period`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.to_period.html#pandas.Series.dt.to_period)(*args, **kwargs) | Cast to PeriodIndex at a particular frequency                |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`Series.dt.to_pydatetime`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.to_pydatetime.html#pandas.Series.dt.to_pydatetime)() |                                                              |
| [`Series.dt.tz_localize`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.tz_localize.html#pandas.Series.dt.tz_localize)(*args, **kwargs) | Localize tz-naive DatetimeIndex to given time zone (using    |
| [`Series.dt.tz_convert`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.tz_convert.html#pandas.Series.dt.tz_convert)(*args, **kwargs) | Convert tz-aware DatetimeIndex from one time zone to another (using |
| [`Series.dt.normalize`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.normalize.html#pandas.Series.dt.normalize)(*args, **kwargs) | Return DatetimeIndex with times to midnight.                 |
| [`Series.dt.strftime`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.strftime.html#pandas.Series.dt.strftime)(*args, **kwargs) | Return an array of formatted strings specified by date_format, which supports the same string format as the python standard library. |
| [`Series.dt.round`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.round.html#pandas.Series.dt.round)(*args, **kwargs) | round the index to the specified freq                        |
| [`Series.dt.floor`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.floor.html#pandas.Series.dt.floor)(*args, **kwargs) | floor the index to the specified freq                        |
| [`Series.dt.ceil`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.ceil.html#pandas.Series.dt.ceil)(*args, **kwargs) | ceil the index to the specified freq                         |

#### **Timedelta Properties**

| [`Series.dt.days`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.days.html#pandas.Series.dt.days) | Number of days for each element.                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`Series.dt.seconds`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.seconds.html#pandas.Series.dt.seconds) | Number of seconds (>= 0 and less than 1 day) for each element. |
| [`Series.dt.microseconds`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.microseconds.html#pandas.Series.dt.microseconds) | Number of microseconds (>= 0 and less than 1 second) for each element. |
| [`Series.dt.nanoseconds`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.nanoseconds.html#pandas.Series.dt.nanoseconds) | Number of nanoseconds (>= 0 and less than 1 microsecond) for each element. |
| [`Series.dt.components`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.components.html#pandas.Series.dt.components) | Return a dataframe of the components (days, hours, minutes, seconds, milliseconds, microseconds, nanoseconds) of the Timedeltas. |

#### **Timedelta Methods**

| [`Series.dt.to_pytimedelta`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.to_pytimedelta.html#pandas.Series.dt.to_pytimedelta)() |                                                      |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| [`Series.dt.total_seconds`](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.Series.dt.total_seconds.html#pandas.Series.dt.total_seconds)(*args, **kwargs) | Total duration of each element expressed in seconds. |

### 转换为时间格式

#### 字符串转换为时间

```python
pd.to_datetime(df.工单生成时间,format='%Y/%m/%d')
#df.工单生成时间是格式为'2016/08/12'类型的str的列
#format识别的字符串格式
```

#### unix时间戳转换为时间

```python
#unit 可选参数 D,s,ms,us,ns
#origin : 定义参考日期scalar, default is ‘unix’
'''
If ‘unix’ (or POSIX) time; origin is set to 1970-01-01.
If ‘julian’, unit must be ‘D’, and origin is set to beginning of Julian Calendar. Julian day number 0 is assigned to the day starting at noon on January 1, 4713 BC.
If Timestamp convertible, origin is set to Timestamp identified by origin.
'''
>>> pd.to_datetime(1490195805, unit='s')
Timestamp('2017-03-22 15:16:45')
>>> pd.to_datetime(1490195805433502912, unit='ns')
Timestamp('2017-03-22 15:16:45.433502912')

#设置新的原点时间戳
pd.to_datetime([1, 2, 3], unit='D',origin=pd.Timestamp('1960-01-01'))
```

### 标准时间格式转换其他时间格式

```python
df.工单生成时间.dt.strftime('%Y-%m-%d')
#将时间转换为年月日的格式
```

### 转换到周几

**注意：**得到的结果为星期几-1.即返回0是星期一而不是星期日，返回6是星期日不是星期六。

```python
time_list = ["2017-05-10 17:19:19", "2017-05-11 17:19:20", "2017-05-12 17:19:20", "2017-05-13 17:19:20"]  
time_ser = pd.Series(time_list)  
time_ser = pd.to_datetime(time_ser)  
#方法一 利用apply
time_weekday=time_ser.apply(lambda x:x.weekday())
#利用dt的函数
time_weekday=time_ser.dt.dayofweek
```

### 进行时间计算

```python
DateOffset(months=3,days=10)
#时间格式的加减要通过DateOffset转换才能运算。

#得到当前时间
datetime.now()==>datetime.datetime(2017, 4, 15, 16, 2, 24, 663820)
#加3月10天
datetime.now()+DateOffset(months=3,days=10)==>Timestamp('2017-07-25 16:03:17.801300')
```

### 时间转换

将时间差值转换为特定计量单位的差值

时间格式有`"as", "fs", "ps", "ns", "us", "ms", "s", "m","h", "D", "W", "M", "Y"`

```python
td = pd.Series(pd.date_range('20130101 11:00:00', periods=4)) - pd.Series(pd.date_range('20121201 10:23:00', periods=4))

#将时间差值转换为以秒为单位
#方法一
td / np.timedelta64(1, 's')
#方法二
td.astype('timedelta64[s]')

#将日期中的时间（时分秒）转换为秒
td.dt.seconds
```

### 日期时间戳转换为日期

```python
import datetime
from datetime import timedelta
#日期时间戳转换
def datetamp2date(date,datestamp,datestamp1):
    #date 已知的日期,'yyyymmdd'的字符串格式
    #datestamp 已知日期的时间戳
    #datestamp1 未知日期的时间戳
    d1 = datetime.datetime.strptime(date, '%Y%m%d')
    d2 =d1+timedelta(days=(datestamp1-datestamp))
    return d2.strftime('%Y%m%d')  
datetamp2date('20180401',43191,43192)

#可以使用pandas.to_datetime快速实现,td可为list或series
td=43191-43192
pd.to_datetime(td,unit='D',box=True,origin=pd.Timestamp('20180401'))
```

### 转换到月初

```python
MonthBegin()
#将时间转换到月初

#减MonthBegin()转换到本月月初
datetime.now()+DateOffset(months=3,days=10)-MonthBegin()==>Timestamp('2017-07-01 16:05:47.921094')
#加MonthBegin()转换到下月月初
datetime.now()+DateOffset(months=3,days=10)+MonthBegin()==>Timestamp('2017-08-01 16:09:18.443141')
```

### 生成时间范围的列表

可以通过freq参数来得到不同频率的日期。freq的参数可以参照[时间频率列表](#时间频率列表)

```python
pandas.date_range(start=None, end=None, periods=None, freq='D', tz=None, normalize=False, name=None, closed=None, **kwargs)
#start：string或datetime-like，默认值是None，表示日期的起点。
#end：string或datetime-like，默认值是None，表示日期的终点。
#periods：integer或None，默认值是None，表示你要从这个函数产生多少个日期索引值；如果是None的话，那么start和end必须不能为None。
#freq：string或DateOffset，默认值是’D’，表示以自然日为单位，这个参数用来指定计时单位，比如’5H’表示每隔5个小时计算一次。
#tz：string或None，表示时区，例如：’Asia/Hong_Kong’。
#normalize：bool，默认值为False，如果为True的话，那么在产生时间索引值之前会先把start和end都转化为当日的午夜0点。
#name：str，默认值为None，给返回的时间索引指定一个名字。
#closed：string或者None，默认值为None，表示start和end这个区间端点是否包含在区间内，可以有三个值，’left’表示左闭右开区间，’right’表示左开右闭区间，None表示两边都是闭区间。

pd.date_range('2011-01-02','2016-11-30',freq='W')
#按周生成时间。起始时间为'2011-01-02'，截止时间为'2016-11-30'
```

### 时间频率列表

| Alias    | Description                                      |
| -------- | ------------------------------------------------ |
| B        | business day frequency                           |
| C        | custom business day frequency                    |
| D        | calendar day frequency                           |
| W        | weekly frequency                                 |
| M        | month end frequency                              |
| SM       | semi-month end frequency (15th and end of month) |
| BM       | business month end frequency                     |
| CBM      | custom business month end frequency              |
| MS       | month start frequency                            |
| SMS      | semi-month start frequency (1st and 15th)        |
| BMS      | business month start frequency                   |
| CBMS     | custom business month start frequency            |
| Q        | quarter end frequency                            |
| BQ       | business quarter end frequency                   |
| QS       | quarter start frequency                          |
| BQS      | business quarter start frequency                 |
| A, Y     | year end frequency                               |
| BA, BY   | business year end frequency                      |
| AS, YS   | year start frequency                             |
| BAS, BYS | business year start frequency                    |
| BH       | business hour frequency                          |
| H        | hourly frequency                                 |
| T, min   | minutely frequency                               |
| S        | secondly frequency                               |
| L, ms    | milliseconds                                     |
| U, us    | microseconds                                     |
| N        | nanoseconds                                      |

## 画图

### 基础画图

在pandas中可以直接画图，使用plt来展示图片。具体参数见[链接](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.plot.html)。

通过plot中的参数kind，可以调整画图的类型。除了默认的'line'还有以下类型：

- [‘bar’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-barplot) or [‘barh’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-barplot) for bar plots
- [‘hist’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-hist) for histogram
- [‘box’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-box) for boxplot
- [‘kde’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-kde) or `'density'` for density plots
- [‘area’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-area-plot) for area plots
- [‘scatter’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-scatter) for scatter plots
- [‘hexbin’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-hexbin) for hexagonal bin plots
- [‘pie’](http://pandas.pydata.org/pandas-docs/stable/visualization.html#visualization-pie) for pie plots

```python
DataFrame.plot(x=None, y=None, kind='line', ax=None, subplots=False, sharex=None, sharey=False, layout=None, figsize=None, use_index=True, title=None, grid=None, legend=True, style=None, logx=False, logy=False, loglog=False, xticks=None, yticks=None, xlim=None, ylim=None, rot=None, fontsize=None, colormap=None, table=False, yerr=None, xerr=None, secondary_y=False, sort_columns=False, **kwds)

import matplotlib.pyplot as plt 
import pandas as pd

df = pd.DataFrame({'a': np.random.randn(6).astype('f4'),
...                    'b': [True, False] * 3,
...                    'c': [1.0, 2.0] * 3})
df.plot(y='a')
plt.show()
```

### 分组柱状图

```python
#当使用多列数据时，画出的图自然就是分组柱状图
df2 = pd.DataFrame(np.random.rand(10, 4), columns=['a', 'b', 'c', 'd'])
#下面为画柱状图的两种写法
#用法一
df2.plot.bar()
#用法二
df2.plot(kind='bar')
```

### 堆叠柱状图

```python
#使用stacked参数，设置其为True则为堆叠状态

df2.plot.bar(stacked=True)
```

### 堆叠条状图

```python
df2.plot.barh(stacked=True);
```

### 画分布图(hist)

```python
DataFrame.hist(data, column=None, by=None, grid=True, xlabelsize=None, xrot=None, ylabelsize=None, yrot=None, ax=None, sharex=False, sharey=False, figsize=None, layout=None, bins=10, **kwds)


import seaborn as sb
import matplotlib
import matplotlib.pyplot as plt

#将时间转换成小时得到时间的分布
login.time=login.time.astype('datetime64')
login_hour=login.time.dt.strftime('%H')
login_hour.astype('int32').hist(bins=24)
plt.show()


#分布的柱状图为堆叠状态
df4 = pd.DataFrame({'a': np.random.randn(1000) + 1, 'b': np.random.randn(1000),'c': np.random.randn(1000) - 1}, columns=['a', 'b', 'c'])

df4.plot.hist(stacked=True, bins=20)
plt.show()
```

### 箱线图

```python
DataFrame.boxplot(column=None, by=None, ax=None, fontsize=None, rot=0, grid=True, figsize=None, layout=None, return_type=None, **kwds)
#by 将数据进行分组
#grid 是否显示网格
#rot 标签旋转角度

df = pd.DataFrame(np.random.rand(10, 5), columns=['A', 'B', 'C', 'D', 'E'])
df.plot.box()

#设置颜色
color = dict(boxes='DarkGreen', whiskers='DarkOrange',medians='DarkBlue', caps='Gray')
df.plot.box(color=color, sym='r+')

#对字段按照分组画图
login.boxplot(['id'],by='type')
plt.show()

#设置位置
login.boxplot(['id'],by='type',positions=[1, 4, 8])
plt.show()

#设置是否为竖直
login.boxplot(['id'],by='type',vert=False)

```

### 画图时对缺失数据的处理

| Plot Type      | NaN Handling            |
| -------------- | ----------------------- |
| Line           | Leave gaps at NaNs      |
| Line (stacked) | Fill 0’s                |
| Bar            | Fill 0’s                |
| Scatter        | Drop NaNs               |
| Histogram      | Drop NaNs (column-wise) |
| Box            | Drop NaNs (column-wise) |
| Area           | Fill 0’s                |
| KDE            | Drop NaNs (column-wise) |
| Hexbin         | Drop NaNs               |
| Pie            | Fill 0’s                |

## 分组计算

### groupby分组

```python
import numpy as np
import pandas as pd
df = pd.DataFrame({'A' : ['foo', 'bar', 'foo', 'bar', 'foo', 'bar', 'foo', 'foo'],
'B' : ['one', 'one', 'two', 'three','two', 'two', 'one', 'three'],
'C' : np.random.randn(8),'D' : np.random.randn(8)})
#按A列进行分组
df1=df.groupby('A')
#按A、B列进行分组
df2=df.groupby(['A','B'])
#按索引进行分组
df3=df.groupby(level=0)
```

### apply使用

apply对整行或整列的进行操作，类似matlab中cellfun对单个cell进行操作。

**注意：**apply最后生成的结果为Series或者DataFrame，因此apply中函数返回的值需为value或者series或者list。
建议使用series最后形成df，或者value（即单个的值）最后形成series。使用list当输出维度大于输入维度时可能会遇到错误。

```python
DataFrame.apply(func, axis=0, broadcast=False, raw=False, reduce=None, args=(), **kwds)
#func 对行/列进行操作的函数，可以自定义函数，多使用匿名函数
#axis=0 1为行及操作一行的所有列，0为列即操作一列的所有行
#broadcast=False 是否和输入dataframe的尺寸保持一致
#raw=False 是否转换为Series格式，False为转换为Series格式，True则使用ndarray代替。如果只使用apply numpy函数，使用ndarray将得到更高的性能。
#reduce 当为None时自动推断输出为Series还是DataFrame，当为True则默认输出为Series,False则默认输出为DataFrame
#args 传递给函数的位置参数

df=pd.DataFrame({'a':[1,2,3],'b':[4,5,6],'c':[7,8,9]})
df.apply(lambda x:print(x),axis=1)
```

### agg

agg可以用作多列都进行某种函数计算，即对队列同时apply操作。也可以用作对单列（或多列）同时进行多个函数操作。

```python
DataFrameGroupBy.agg(func)
#func 可以输入函数字符串，或者函数列表，或者字典{列名:函数名}

df.groupby('A').agg({'B': ['min', 'max'], 'C': 'sum'})
    B             C
  min max       sum
A
1   1   2  0.590716
2   3   4  0.704907
```

## 问题解决记录

### pandas读取csv，前面有\ufeff

用pandas读取csv时，有时前面会有\ufeff，这是因为读取为'utf-8'+bom文件。可以通过'utf-8-sig'读取方式解决。

注意首行可能会有`'\ufeff'`，注意剔除。

```python
#用utf-8-sig格式读取
df1=pd.read_csv('/home/weblogic/DATA/private/shangguanxf/cc_txt2/疑似误导录音/疑似诱导录音文本.csv',encoding='utf-8-sig',header=None,names=['录音编号','通话文本'])
```
### pandas 读取csv遇到分割错误，预期数据比实际少

pandas 读取数据时遇到`C error: Expected 24 fields in line 12599, saw 26`，但是数据按照分割符去分确实只有24个，可以考虑换python引擎，c引擎有时会遇到奇怪的bug。

```python
#将引擎设置为python,会导致速度下降，c引擎遇到bug是可以考虑尝试换python引擎解决
df=pd.read_csv('/tpdata/DATA/private/shangguanxf/2017白皮书/data/车险报案数据.txt',encoding='gb18030',sep='|',engine='python')

```

### 根据特定列去除包含空值的行

df.dropna(axis=0)为去除包含空值的行，无法指定只根据特定行删除。如果不需要和其他值关联，可以使用`df.坐席原始意见..dropna(axis=0)`来获取特定列去除空值后的结果。

```python
df4=df4[df4.坐席原始意见.isnull()==False]
#1.df4.坐席原始意见.isnull() 得到这一列是否为空的真值列（空为真，非空为假）
#2.通过==False将非空列转化为真
#3.df4[df4.坐席原始意见.isnull()==False]获取df4中为空的行

df.isnull()
#将返回一个真值矩阵，空为真，非空为假
```

### 生成训练集和测试集

```python
#取70%为训练集，得到再采样的70%的index
train_list=list(all_.sample(frac=0.7).index)
#通过set得到全量的index和训练集的index的差集作为测试集
test_list=list(set(list(range(len(all_))))-set(train_list))
#通过index取得训练集
train=all_.iloc[train_list,:]
train.reset_index(inplace=True, drop=True)
#训练集中目标分类数量（分类为1）
print(sum(train.标签))
#通过index取得测试集
test=all_.iloc[test_list,:]
test.reset_index(inplace=True, drop=True)
#测试集中目标分类数量（分类为1）
print(sum(test.标签))
```

### 文本特征中错误分行

数据中的文本在转换过程中空格被转换成换行符。

```python
#将语音文本数据中的错误分开的文本合并
pathname='/home/weblogic/DATA/private/shangguanxf/cc_txt2/大数据语音文本信息.txt'
#二进制读取文件
f = open(pathname, 'rb')  
line=f.readlines()
for i in range(2,len(line)):
    #解码文本
    s=line[i].decode()
    #只保留数字行前面的换行符。正常一行应有6列，由于空格被转换成换行符，导致最后一列文本被错分多行。删除非6列的行的换行符，合并文本数据。
    if s.count(',')<6:
        line[i-1]=line[i-1][:-2]
with open ('/home/weblogic/DATA/private/shangguanxf/cc_txt2/大数据语音文本信息.csv','wb') as file2:
    file2.writelines(line)    
print('end')
```

### DataFrame相加值全为nan

**注意：**dataframe相加需要列名且索引（index）一致，否则会导致结果全为nan。解决方法：

1. 将一个dataframe的列名与另一个dataframe改一致。
2. 将df转换为np.array()，即数组形式，但会丢失列名和索引信息。

```python
df1=pd.DataFrame({'A':[1,2,3], 'B':[4,5,6], 'C':[7,8,9]})
df2=pd.DataFrame({'a':[1,2,3], 'b':[4,5,6], 'c':[7,8,9]})
df1+df2
-->A	B	C	a	b	c
0	NaN	NaN	NaN	NaN	NaN	NaN
1	NaN	NaN	NaN	NaN	NaN	NaN
2	NaN	NaN	NaN	NaN	NaN	NaN
#更改列名
df1.columns=df2.columns
df1+df2
-->a	b	c
0	2	8	14
1	4	10	16
2	6	12	18

#转换为array相加
np.array(df1)+np.array(df2)
-->
array([[ 2,  8, 14],
       [ 4, 10, 16],
       [ 6, 12, 18]], dtype=int64)
```

### 查看df中索引是否存在

```python
import pandas as pd
df1=pd.DataFrame({'A':[1,2,3], 'B':[4,5,6], 'C':[7,8,9]})
#isin(list),list为需要确认是否存在的索引
df.index.isin([3])
->array([False, False,  True, False], dtype=bool)
#使用逻辑词直接判断
3 in df.index
->True
#可以结合if进行使用
if 3 in df.index:print('ok')
```

### np数组无法用df真值赋值

将df真值转为np真值即可成功使用

```python
a=np.array([1,2,3,4,5,6,7,8])
df=pd.DataFrame({'A':[1,2,3,4,5,6,7,8]})
#直接用df的真值去操作
a[df.A>3]
-->array([1, 1, 1, 2, 2, 2, 2, 2])
a[df.A>3]=1
a
-->array([1, 1, 3, 4, 5, 6, 7, 8])
#将df的真值转为np数组
a[np.array(df.A>3)]
-->array([4, 5, 6, 7, 8])
a[np.array(df.A>3)]=1
a
-->array([1, 2, 3, 1, 1, 1, 1, 1])
```

### df无法用真值赋值

在使用真值为df赋值时，出现以下错误提示：

`IndexingError: Unalignable boolean Series provided as indexer (index of the boolean Series and of the indexed object do not match`

这时由于df匹配真值时是按照索引进行匹配，而df如果是抽样或者合并取得，会导致索引变化，而真值仍为从0开始，所以索引会无法匹配。解决办法为对df进行索引重排序。

```python
from sklearn.ensemble import RandomForestClassifier

train=df.sample(n=30000)
test_index=set(df.index)-set(train.index)
test=df.loc[test_index,:]

clf = RandomForestClassifier(n_estimators=500,max_depth=2, random_state=0)
clf.fit(train_vec, train_label)
p_y=clf.predict(test_vec)
#会报错
err_df=df[p_y!=test_label]

#需进行重排序
train=train.reset_index(drop=True)
test=test.reset_index(drop=True)
err_df=df[p_y!=test_label]
```

### 判断df是否存在

```python
df is None
#为空返回真
df is not None
#不为空返回真
```

### df读取数据过慢

在对数据重采样时，使用df进行数据重采样。发现速度过慢，在使用numpy.array测试时发现速度差距，怀疑df不是索引获取数据。为提高重采样速度，使用numpy.array()代替df。

```python
test=np.ones((10000,1))
prob=numpy.ones(10000)/10000
t0=time.time()
for i in range(100):
    for j in range(10000):
        #使用numpy.array数值进行取数
        p=prob[j]
print(time.time()-t0)
t1=time.time()
df=pd.DataFrame({'b':prob})
for i in range(10):
    for j in range(10000):
        #使用df的函数进行取数
        p=df.loc[j,'b']
print(time.time()-t1)
->0.3686678409576416
->36.97714591026306
```

### 使用df.astype时报错

**问题：**在对数据进行合并（pd.concat()）后直接进行df.astype()的格式转换时报错。

**解决方法：**在对合并后的数据进行格式转换前使用df.reset_index()来进行重置索引，之后再进行df.astype()的格式转换。

### df中的列有文本数字混在一起的列

**问题：**在对id号进行匹配时，一部分id号前面有标签，格式为'寿险1234567'，需将'寿险'两字去掉。

**解决方法：**通过df.apply函数结合re.findall将id号中的数字匹配出来

```python
import re
df.录音编号=df.录音编号.apply(lambda x:re.findall('[1-9]\d*',str(x))[0])
#'[1-9]\d*'匹配连续数字
#re.findall()[0]将匹配到的数字从列表中取出，即列表第一位
```

### 处理英文bytes数据

**问题：**在使用open的`'rb'`得到bytes类型数据后常规编码无法解码('utf-8','gbk','gb2312','gb18030')，导致无法进行str的相关操作。源文件疑似使用unicode编码。

**解决方法：**使用`'unicode_escape'`方式解码，因为python3中使用unicode作为中间编码，而源文件也为unicode编码，故可使用python独有编码`'unicode_escape'`进行处理。

```python
text.decode('unicode_escape')
```

### 获取文件夹中最新的n个文件

```python
#定义一个队列，保留该文件内创建时间最新的n个文件
def newest_file(path,n=2):
    newest_list=[]
    for root, dirs, files in os.walk(path):
        for file in files:
            if len(newest_list)<n+1:
                newest_list.append((file,os.path.getctime(os.path.join(root,file))))
            else:
                newest_list=sorted(newest_list, key=lambda x:x[1],  reverse = True)
                nt=os.path.getctime(os.path.join(root,file))
                if nt>newest_list[n-1][1]:
                    newest_list[n-1]=(file,nt)
    return newest_list
```

### 用df中的值给df赋值

在生成df时想用已有的df中的值进行赋值。

```python
df2=pd.DataFrame(columns=['录音编号','录音文本','坐席录音文本','客户录音文本'])

#使用下面命令会报错。ValueError: Incompatible indexer with DataFrame（不相容的索引与数据帧）
df2.loc[0]=pd.DataFrame(df1.loc[df1.inum==i,['录音编号','录音文本','坐席录音文本','客户录音文本']])
#个人理解这种情况无法赋值是因为会有标题和索引存在的缘故
df1.loc[df1.inum==i,:].loc[:,['录音编号','录音文本','坐席录音文本','客户录音文本']]
#输出如下面表格
```

|      | 录音编号         | 录音文本                                     | 坐席录音文本                                   | 客户录音文本                                   |
| ---- | ------------ | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| 2568 | 7.000000e+14 | ￡OP不按。?￡CU嗯。?￡OP不按。?￡OP我您好我是中国太平客服专员柳工号113450沈... | OP不按OP不按OP我您好我是中国太平客服专员柳工号113450沈好天洪这什么噢您好感谢您近... | CU嗯CU嗯CU谢谢吧CU好CU好谢谢CU嗯CU法院去掉了吧CU好CU应该清楚了吧呃它是所有... |

可以使用下面的方法进行赋值

```python
#方法一，使用iloc进行取值。不过缺点是需要数清对应列的index
df2.loc[0]=df1.loc[df1.inum==i,:].iloc[0,[2,16,6,10]]

#方法二，单独取每列的值合并到一起
df1.loc[n]=[df.机构名称[i],m3,df.工单机构分布[i],df.该机构工单量[i],df.有效客户数[i]]


```

**注意：**df.loc[i,:]和df.iloc[i,:]取出来的值都有列名信息，需要转换成list。否则会导致值无法插入，新的列表中值为Nan

```python
#需要列名相匹配，否则值为Nan
tmp_time=trade.iloc[i,1]
tmp_login0=login[(login['id']==tmp_id)]
tmp_login1=tmp_login0[tmp_login0.time<tmp_time]
result.loc[i,3:]=tmp_login1.iloc[-1,:]

#或者使用list插入
result.loc[i,3:]=list(tmp_login1.iloc[-1,:])
```

### 使用计算结果来为df赋值

在利用时间进行计算得到差值来给df的新列赋值后，后续计算遇到错误。考虑预先用None为列赋值，在将结果赋值到列。

```python
tmp_login['interval']=x[1]-tmp_login.time
#赋值后进行下列计算遇到格式错误，因为第一次赋值无法改变类型，tmp_login['interval']是int类型
tmp_login['interval']=tmp_login['interval']/np.timedelta64(1, 'D')
```

### 使用apply来建表

使用apply来建表，通过表中的字段关联另一表产生新的表。初用for循环速度太慢，改用apply。但一直无法顺利赋值。后来通过观察发现，当apply中结果为series时需要函数返回单个的值，当apply结果为DataFrame时，需要函数返回Series。

```python
def concat(x,login):
    r=pd.Serise([None for i in range(12)],index=login.columns)
    try:
        tmp_login1=login.loc[x[2]][login.loc[x[2]].time<x[1]]
        try:
            result=tmp_login1.iloc[-1,:]
        #遇到只有一次登录记录的，如果不是在交易前登录则赋值为None
        except AttributeError as a:
            if login.loc[x[2]].time<x[1]:
                result=login.loc[x[2]]
            else:
                result=r
        #遇到没有在购买之前登录的赋值为None
        except IndexError as e:
            result=r
        finally:
            pass
    #遇到login里面没有登录过的id，赋值为None
    except KeyError as k:
        result=r
    finally:
        pass
    return result
trade1=trade.iloc[0:1000,:]
r=trade1.apply(lambda x:concat(x,login),axis=1,reduce=False)
```

### 计算DBI

　$\Large DBI=\frac{1}{k}\sum_{i=1}^{k}\max\limits_{j\neq i}(\frac{avg(C_i)+avg(C_j)}{d_{cen(u_i,u_j)}})$

DB用于计算 任意两类别的类内距离平均距离(CP)之和除以两聚类中心距离 求最大值

DB越小意味着类内距离越小 同时类间距离越大 

缺点：因使用欧式距离 所以对于环状分布  聚类评测很差

```python
#计算DBI代码
def dis(data,position,cluster):
    tmp=pd.DataFrame()
    for i in position:
        tmp[i]=np.array(data[i])-np.array(cluster[i])
    dis=tmp.apply(lambda x:np.sqrt(sum(np.power(np.array(x),2))),axis=1)
    return dis
def dbi(data,label,position,cluster,label_c,position_c):
    #data 数据
    #label 数据的聚类类列名
    #position 数据位置列名
    #cluster 聚类点数据
    #label_c 聚类类别列名
    #position_c 聚类中心点位置列名
    S={}
    col=data[label].unique()
    for i in col:
        tmp_data=data[data[label]==i]
        tmp_cluster=cluster[cluster[label]==i]
        tmp_dis=dis(tmp_data,position,tmp_cluster)
        S[i]=np.sqrt(sum(np.power(tmp_dis,2))/len(tmp_data))
    M=pd.DataFrame(columns=data[label].unique(),index=data[label].unique())
    R=pd.DataFrame(columns=data[label].unique(),index=data[label].unique())
    for i in col:
        for j in col:
            if i!=j:
                Mi=cluster[cluster[label]==i]
                Mj=cluster[cluster[label]==j]
                M.loc[i,j]=np.sqrt(sum(np.power((np.array(Mi[position])[0]-np.array(Mj[position]))[0],2)))
                R.loc[i,j]=(S[i]+S[j])/M.loc[i,j]
    dbi=R.max(axis=1).mean()
    return dbi
```

### 探查缺失值情况

对表中为Nan的值画图

```python
#导入包
import matplotlib.pyplot as plt
import matplotlib
import pandas as pd
import seaborn as sb
#获取数据
df=pd.read_csv('/tpdata/DATA/private/shangguanxf/2017白皮书/data/车险报案数据.txt',encoding='gb18030',sep='|',engine='python')
#将数据缺失情况的bool类型转换为int类型
nullar=df.isnull().values.astype(np.int8)
#画图
sb.heatmap(df2[0:100],vmin=0,vmax=1，cmap=sb.dark_palette("white",as_cmap=True),yticklabels=False,cbar=False)
plt.show()
```

### 一列中的文本拆分为多行

在做数据处理时遇到有一列数据为按照`','`分割的多个值。为了分析，需要把它拆分成多行。

主要是用split的expand参数将其扩展成多列，在利用stack函数进行列转行，最后在将其与原数据合并。

```python
import pandas as pd
df = pd.DataFrame({'Country':['China','US','Japan','EU','UK/Australia', 'UK/Netherland'],
               'Number':[100, 150, 120, 90, 30, 2],
               'Value': [1, 2, 3, 4, 5, 6],
               'label': list('abcdef')})


new_df=df.drop('Country', axis=1).join(df['Country'].str.split(',', expand=True).stack().reset_index(level=1, drop=True).rename('Country'))
```

### 用df列中的值是否存在

在使用df中的列进行判断某些值是否存在时发现，`value in df['columns']`是看value是否在`df['columns']`的索引中，而不是是否存在于`df['columns']`的值中。想要看是否存在于`df['columns']`中的值，可以用`df['columns'].values`或者`set(df['columns'])`

```python
import pandas as pd

df=pd.DataFrame({'a':[3,6,9]})
6 in df['a']
->False

df=pd.DataFrame({'a':[3,6,9]})
6 in df['a'].values
->True

df=pd.DataFrame({'a':[3,6,9]})
6 in set(df['a'])
->True
```

### df中赋值后的表更改会导致源表的变化

在进行表的运算时发现新的表运算会导致原表值的变化，了解发现，直接赋值是映射内存块，不会产生新的副本。只有使用`.copy()`函数后才会产生新的副本或者新建一个df，再一列一列的赋值。

```python
import pandas as pd
df1=pd.DataFrame({'a':[1,1,1,1,1],'b':[2,2,2,2,2]})
df2=df1
df2.a=2
df1
->
a	b
0	2	2
1	2	2
2	2	2
3	2	2
4	2	2

#正确用法一
df1=pd.DataFrame({'a':[1,1,1,1,1],'b':[2,2,2,2,2]})
df2=df1.copy()
df2.a=2
df1
->
	a	b
0	1	2
1	1	2
2	1	2
3	1	2
4	1	2

#正确用法二
df1=pd.DataFrame({'a':[1,1,1,1,1],'b':[2,2,2,2,2]})
df2=pd.DataFrame()
df2['a']=df1['a']
df2.a=2
df1
->
	a	b
0	1	2
1	1	2
2	1	2
3	1	2
4	1	2
```



