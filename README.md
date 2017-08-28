# 利用python进行数据分析之数据聚合和分组运算
对数据集进行分组并对各分组应用函数是数据分析中的重要环节。

group by技术

pandas对象中的数据会根据你所提供的一个或多个键被拆分为多组，拆分操作是在对象的特定轴上执行的，然后将一个函数应用到各个分组并产生一个新值，最后所有这些函数的执行结果会被合并到最终的结果对象中。

复制代码
>>> from pandas import *
>>> df=DataFrame({'key1':['a','a','b','b','a'],'key2':['one','two','one','two','one'],'data1':np.random.randn(5),'data2':np.random.randn(5)})
>>> df
      data1     data2 key1 key2
0 -1.413818 -0.865514    a  one
1 -1.001804  0.309597    a  two
2  0.357458 -0.387695    b  one
3  0.674294 -0.977009    b  two
4 -0.090150  2.444888    a  one
>>> grouped=df['data1'].groupby(df['key1'])
>>> grouped
<pandas.core.groupby.SeriesGroupBy object at 0x04005770>
#生成一个groupby对象，实际上还未进行任何计算，可对其调用方法进行计算
>>> grouped.mean()
key1
a   -0.835257
b    0.515876
Name: data1, dtype: float64
#此外，可将列名直接当作分组对象，分组中，数值列会被聚合，非数值列会从结果中排除
>>> df.groupby('key1').mean()
         data1     data2
key1                    
a    -0.835257  0.629657
b     0.515876 -0.682352
>>> df.groupby(['key1','key2']).mean()
              data1     data2
key1 key2                    
a    one  -0.751984  0.789687
     two  -1.001804  0.309597
b    one   0.357458 -0.387695
     two   0.674294 -0.977009
复制代码
 无论你准准备拿groupby做什么，都可能会使用groupby的size方法，可以返回一个含有分组大小的series；

复制代码
>>> df.groupby(['key1','key2']).size()
key1  key2
a     one     2
      two     1
b     one     1
      two     1
dtype: int64
复制代码
 1、对分组进行迭代

groupby对象支持迭代，可以产生一组二元数组（由分组名称和数据块构成）

复制代码
>>> for name,group in df.groupby('key1'):
    print name
    print group

    
a
      data1     data2 key1 key2
0 -1.413818 -0.865514    a  one
1 -1.001804  0.309597    a  two
4 -0.090150  2.444888    a  one
b
      data1     data2 key1 key2
2  0.357458 -0.387695    b  one
3  0.674294 -0.977009    b  two
复制代码
 对于多重键的情况，元祖的第一个元素将会是由键值组成的元组

复制代码
>>> for (k1,k2),group in df.groupby(['key1','key2']):
    print k1,k2
    print group
    
a one
      data1     data2 key1 key2
0 -1.413818 -0.865514    a  one
4 -0.090150  2.444888    a  one
a two
      data1     data2 key1 key2
1 -1.001804  0.309597    a  two
b one
      data1     data2 key1 key2
2  0.357458 -0.387695    b  one
b two
      data1     data2 key1 key2
3  0.674294 -0.977009    b  two
复制代码
 groupby分组默认是在axis=0上进行分组的，通过设置也可以在其他轴上进行分组

复制代码
>>> df.dtypes
data1    float64
data2    float64
key1      object
key2      object
dtype: object
>>> grouped=df.groupby(df.dtypes,axis=1)
>>> dict(list(grouped))
{dtype('O'):   key1 key2
0    a  one
1    a  two
2    b  one
3    b  two
4    a  one, dtype('float64'):       data1     data2
0 -1.413818 -0.865514
1 -1.001804  0.309597
2  0.357458 -0.387695
3  0.674294 -0.977009
4 -0.090150  2.444888}
复制代码
 2、选取一个或一组列

对于DataFrame产生的groupby对象，如果用一个或一组列名对其进行索引，就能实现选取部分列进行聚合的目的

复制代码
>>> df
      data1     data2 key1 key2
0 -1.413818 -0.865514    a  one
1 -1.001804  0.309597    a  two
2  0.357458 -0.387695    b  one
3  0.674294 -0.977009    b  two
4 -0.090150  2.444888    a  one
>>> df.groupby('key1')['data1']
<pandas.core.groupby.SeriesGroupBy object at 0x04005FB0>
　　>>> df.groupby('key1')['data1'].mean()
　　key1
　　a -0.835257
　　b 0.515876

复制代码
尤其对于大数据集，可能只需要对部分列进行聚合

复制代码
>>> df.groupby(['key1','key2'])[['data2']].mean()
#注意data2的形式，如果传入的是标量名称则不同
              data2
key1 key2          
a    one   0.789687
     two   0.309597
b    one  -0.387695
     two  -0.977009

>>> df.groupby(['key1','key2'])['data2'].mean()
key1  key2
a     one     0.789687
      two     0.309597
b     one    -0.387695
      two    -0.977009
Name: data2, dtype: float64
复制代码
 3、通过字典或Series进行分组

除数组以外，分组信息还可以以其他形式存在

复制代码
>>> people=DataFrame(np.random.randn(5,5),columns=['a','b','c','d','e'],index=['joe','steve','wes','jim','travis'])
>>> people
               a         b         c         d         e
joe    -1.136829 -0.549897  1.382399 -1.457968 -1.975316
steve   0.633057  0.905028  0.615449 -1.307026 -0.150066
wes     0.715308 -1.546033  1.090450 -0.699447  0.308514
jim     0.127834  0.134140  0.218690  0.298301  0.722678
travis  1.561881  0.283804  0.017650  1.231204 -1.732033
>>> people.ix[2:3,['b','c']]=np.nan
>>> people
               a         b         c         d         e
joe    -1.136829 -0.549897  1.382399 -1.457968 -1.975316
steve   0.633057  0.905028  0.615449 -1.307026 -0.150066
wes     0.715308       NaN       NaN -0.699447  0.308514
jim     0.127834  0.134140  0.218690  0.298301  0.722678
travis  1.561881  0.283804  0.017650  1.231204 -1.732033
>>> mapping={'a':'red','b':'red','c':'blue','d':'blue','e':'red'}
>>> by_column=people.groupby(mapping,axis=1)
>>> by_column.sum()
            blue       red
joe    -0.075569 -3.662042
steve  -0.691577  1.388018
wes    -0.699447  1.023822
jim     0.516991  0.984652
travis  1.248854  0.113652
复制代码
Series也有这样的功能，它可以被看作一个固定大小的映射

复制代码
>>> map_series=Series(mapping)
>>> map_series
a     red
b     red
c    blue
d    blue
e     red
dtype: object
>>> people.groupby(map_series,axis=1).sum()
            blue       red
joe    -0.075569 -3.662042
steve  -0.691577  1.388018
wes    -0.699447  1.023822
jim     0.516991  0.984652
travis  1.248854  0.113652
复制代码
4、通过函数进行分组

相较于字典或者Series，python函数在定义分组映射关系时可以更具创意和抽象，任何被当作分组键的函数都会在索引值上被调用一次，其返回值被当作分组名称

复制代码
#根据人名长度进行分组
>>> people.groupby(len).sum()
          a         b         c         d         e
3 -0.293687 -0.415757  1.601089 -1.859114 -0.944124
5  0.633057  0.905028  0.615449 -1.307026 -0.150066
6  1.561881  0.283804  0.017650  1.231204 -1.732033
复制代码
将函数，列表，字典混用也没问题，因为任何东西最终会被转换为数组

复制代码
>>> keyliat=['one','one','one','two','two']
>>> people.groupby([len,keyliat]).min()
              a         b         c         d         e
3 one -1.136829 -0.549897  1.382399 -1.457968 -1.975316
  two  0.127834  0.134140  0.218690  0.298301  0.722678
5 one  0.633057  0.905028  0.615449 -1.307026 -0.150066
6 two  1.561881  0.283804  0.017650  1.231204 -1.732033
复制代码
5、根据索引级别分组

层次化索引的数据集最方便的地方在于它能够根据索引级别进行聚合，实现该目的，通过level关键字传入级别编号或名称即可。

复制代码
>>> import numpy as np
>>> hief_df=DataFrame(np.random.randn(4,5),columns=columns)
>>> hief_df
cty           us                            jp          
tennor         1         3         5         1         3
0      -0.185892 -0.517436 -0.040285  1.274849  0.015439
1      -1.757972 -0.650451  0.863938  0.467745 -0.288524
2       1.512232 -0.494746 -0.119517  1.047349 -0.627444
3      -0.656453  0.858041  1.218276  1.138983  0.997657
>>> hief_df.groupby(level='cty',axis=1).count()
cty  jp  us
0     2   3
1     2   3
2     2   3
3     2   3
复制代码
数据聚合

 对于聚合,一般指的是能够从数组产生的标量值的数据转换过程,常见的聚合运算都有相关的统计函数快速实现,当然也可以自定义聚合运算

要使用自己的定义的聚合函数,需将其传入aggregate或agg方法即可

复制代码
>>> df=DataFrame({'key1':['a','a','b','b','a'],'key2':['one','two','one','two','one'],'data1':np.random.randn(5),'data2':np.random.randn(5)})
>>> df
      data1     data2 key1 key2
0 -1.299938 -1.269616    a  one
1 -0.279184 -0.037004    a  two
2 -0.851559 -0.527337    b  one
3  1.140124  0.882907    b  two
4  0.406030 -0.365484    a  one
>>> grouped=df.groupby('key1')
>>> def peak_to_peak(arr):
    return arr.max()-arr.min()

>>> grouped.agg(peak_to_peak)
         data1     data2
key1                    
a     1.705968  1.232612
b     1.991683  1.410243
复制代码
 describe方法也可使用,但严格来说这些并非聚合运算

复制代码
>>> grouped.describe()
               data1     data2
key1                          
a    count  3.000000  3.000000
     mean  -0.391031 -0.557368
     std    0.858466  0.638316
     min   -1.299938 -1.269616
     25%   -0.789561 -0.817550
     50%   -0.279184 -0.365484
     75%    0.063423 -0.201244
     max    0.406030 -0.037004
b    count  2.000000  2.000000
     mean   0.144282  0.177785
     std    1.408332  0.997193
     min   -0.851559 -0.527337
     25%   -0.353638 -0.174776
     50%    0.144282  0.177785
     75%    0.642203  0.530346
     max    1.140124  0.882907
复制代码
 1、面向列的多函数应用

前面已经看到对Series或DataFrame列的聚合运算其实就是使用aggregate调用自定义函数或者直接调用诸如mean，std之类的方法；

但是当你希望对不同列使用不同的聚合函数时看如下事例：

复制代码
>>> tips['tip_pct']=tips['tip']/tips['total_bill']
>>> tips[:6]
   total_bill   tip     sex smoker  day    time  size   tip_pct
0       16.99  1.01  Female     No  Sun  Dinner     2  0.059447
1       10.34  1.66    Male     No  Sun  Dinner     3  0.160542
2       21.01  3.50    Male     No  Sun  Dinner     3  0.166587
3       23.68  3.31    Male     No  Sun  Dinner     2  0.139780
4       24.59  3.61  Female     No  Sun  Dinner     4  0.146808
5       25.29  4.71    Male     No  Sun  Dinner     4  0.186240
>>> grouped=tips.groupby(['sex','smoker'])
>>> grouped_pct=grouped['tip_pct']
#可以将函数名以字符串的形式传入
>>> grouped_pct.agg('mean')
sex     smoker
Female  No        0.156921
        Yes       0.182150
Male    No        0.160669
        Yes       0.152771
Name: tip_pct, dtype: float64
复制代码
 如果传入一组函数或者函数名，则得到的DataFrame列就会以相应的函数命名，实际操作中并不一定需要接受默认的函数名，可以传入一个由（name,function）元组组成的列表当作一个有序映射。

复制代码
>>> grouped_pct.agg(['mean','std'])
                   mean       std
sex    smoker                    
Female No      0.156921  0.036421
       Yes     0.182150  0.071595
Male   No      0.160669  0.041849
       Yes     0.152771  0.090588
复制代码
复制代码
>>> grouped_pct.agg([('foo','mean'),('bar',np.std)])
                    foo       bar
sex    smoker                    
Female No      0.156921  0.036421
       Yes     0.182150  0.071595
Male   No      0.160669  0.041849
       Yes     0.152771  0.090588
复制代码
对于DataFrame，还可以定义一组应用于全部列的函数，或不同的列应用不同的函数，这样会产生层次化索引的DataFrame

复制代码
>>> functions=['count','mean','max']
>>> result=grouped['tip_pct','total_bill'].agg(functions)
>>> result
              tip_pct                     total_bill                  
                count      mean       max      count       mean    max
sex    smoker                                                         
Female No          54  0.156921  0.252672         54  18.105185  35.83
       Yes         33  0.182150  0.416667         33  17.977879  44.30
Male   No          97  0.160669  0.291990         97  19.791237  48.33
       Yes         60  0.152771  0.710345         60  22.284500  50.81
复制代码
 现在假设想要对不同的列应用不同的函数，具体的办法就是向agg传入一个从列名映射到函数的字典

复制代码
>>> grouped.agg({'tip':np.max,'size':'sum'})
                tip  size
sex    smoker            
Female No       5.2   140
       Yes      6.5    74
Male   No       9.0   263
       Yes     10.0   150
>>> grouped.agg({'tip_pct':['min','max','mean'],'size':'sum'})
                tip_pct                     size
                    min       max      mean  sum
sex    smoker                                   
Female No      0.056797  0.252672  0.156921  140
       Yes     0.056433  0.416667  0.182150   74
Male   No      0.071804  0.291990  0.160669  263
       Yes     0.035638  0.710345  0.152771  150
复制代码
 2、以无索引的形式返回聚合数据

一般情况下，聚合数据都需要唯一的分组键组成的索引，但也可以通过向groupby传入as_index=False以禁用该功能

复制代码
>>> tips.groupby(['sex','smoker'],as_index=False).mean()
      sex smoker  total_bill       tip      size   tip_pct
0  Female     No   18.105185  2.773519  2.592593  0.156921
1  Female    Yes   17.977879  2.931515  2.242424  0.182150
2    Male     No   19.791237  3.113402  2.711340  0.160669
3    Male    Yes   22.284500  3.051167  2.500000  0.152771
复制代码
分组运算和转换

聚合仅是分组运算的一种，它是数据转换的一个特例，本节介绍transform和apply方法，他们能够执行更多其他的分组运算

以下是为一个DataFrame添加一个用于存放各索引组平均值的列，利用了先聚合再合并

 

复制代码
>>> df
      data1     data2 key1 key2
0 -1.359405 -0.567306    a  one
1 -0.298647 -1.078614    a  two
2  0.355256  0.693866    b  one
3 -1.452335 -0.666225    b  two
4  1.036177  1.811104    a  one
>>> k1_means=df.groupby('key1').mean()
>>> k2_means=df.groupby('key1').mean().add_prefix('mean_')
>>> k1_means
         data1     data2
key1                    
a    -0.207292  0.055061
b    -0.548539  0.013821
>>> k2_means
      mean_data1  mean_data2
key1                        
a      -0.207292    0.055061
b      -0.548539    0.013821
>>> merge(df,k2_means,left_on='key1',right_index=True)
      data1     data2 key1 key2  mean_data1  mean_data2
0 -1.359405 -0.567306    a  one   -0.207292    0.055061
1 -0.298647 -1.078614    a  two   -0.207292    0.055061
4  1.036177  1.811104    a  one   -0.207292    0.055061
2  0.355256  0.693866    b  one   -0.548539    0.013821
3 -1.452335 -0.666225    b  two   -0.548539    0.013821
复制代码
 

实际上可以对DataFrame进行transform方法，对比一下下面两种的区别，transform会将一个函数应用到各个分组

复制代码
>>> df.groupby('key2').transform(np.mean)
      data1     data2
0  0.010676  0.645888
1 -0.875491 -0.872420
2  0.010676  0.645888
3 -0.875491 -0.872420
4  0.010676  0.645888
>>> df.groupby('key2').mean()
         data1     data2
key2                    
one   0.010676  0.645888
two  -0.875491 -0.872420
复制代码
1、apply，一般性的拆分-应用-合并

 最一般的groupby方法是apply，apply会将待处理的对象拆分为多个片段，然后对各个片段调用传入的函数，最后尝试将各片段组合在一起，

在groupby中，当你调用诸如describe之类的方法时，实际上是应用了快捷方式：f=lambda x:x.describe();grouped.apply(f)

 2、分位数和桶分析

pandas有一些能根据指定面元或样本分位数将数据拆分为多块的工具（比如cut和qcut），将这些数据跟groupby结合起来，就能轻松的对数据集的桶或分位数分析

复制代码
>>>frame=DataFrame({'data1':np.random.randn(1000),'data2':np.random.randn(1000)})
>>> factor=cut(frame.data1,4)
>>> factor[:10]
0     (-1.35, 0.107]
1     (0.107, 1.563]
2     (-1.35, 0.107]
3    (-2.812, -1.35]
4     (0.107, 1.563]
5     (0.107, 1.563]
6     (-1.35, 0.107]
7     (-1.35, 0.107]
8     (-1.35, 0.107]
9      (1.563, 3.02]
Name: data1, dtype: category
Categories (4, object): [(-2.812, -1.35] < (-1.35, 0.107] < (0.107, 1.563] < (1.563, 3.02]]
复制代码
cut返回的factor对象可直接用于groupby，分为长度相等的桶；

复制代码
>>> def get_stats(group):
    return {'min':group.min(),'max':group.max(),'count':group.count(),'mean':group.mean()}

>>> grouped=frame.data2.groupby(factor)
>>> grouped.apply(get_stats).unstack()
                 count       max      mean       min
data1                                               
(-2.812, -1.35]     79  2.791474  0.023155 -2.577103
(-1.35, 0.107]     433  2.942033  0.066771 -2.812077
(0.107, 1.563]     437  2.391669  0.022582 -2.654376
(1.563, 3.02]       51  2.652038  0.406708 -2.387372
复制代码
 若要得到大小相等的桶，使用qcut即可

复制代码
>>> grouping=qcut(frame.data1,10,labels=False)
>>> grouped=frame.data2.groupby(grouping)
>>> grouped.apply(get_stats).unstack()
   count       max      mean       min
0    100  2.791474  0.025400 -2.577103
1    100  2.536797 -0.094773 -2.046163
2    100  2.942033  0.243372 -1.671060
3    100  2.566991  0.059096 -2.252417
4    100  2.589560  0.053143 -2.812077
5    100  1.743871 -0.041336 -2.448941
6    100  2.295631  0.157645 -2.264740
7    100  2.391669 -0.012642 -2.076873
8    100  2.164782  0.026390 -2.654376
9    100  2.652038  0.197221 -2.387372
复制代码
 3、用特定分组的值填充缺失值

对于缺失数据的清理工作，有时你会用dropna将其删除，有时可能会希望用一个固定值或由数据集本事衍生出来的值去填充na值，这时应该使用fillna工具

复制代码
>>> from pandas import *
>>> s=Series(np.random.randn(6))
>>> s[::2]=np.nan
>>> s
0         NaN
1    0.730366
2         NaN
3    1.072793
4         NaN
5   -0.720886
dtype: float64
>>> s.fillna(s.mean())
0    0.360758
1    0.730366
2    0.360758
3    1.072793
4    0.360758
5   -0.720886
dtype: float64
复制代码
假设需要对不同的分组填充不同的值，只需将数据分组，并使用apply和一个能够对各数据块调用的fillna的函数即可

复制代码
>>> state=['ohio','new york','vermont','florida','oregen','nevada','california','idaho']
>>> group_key=['east']*4+['west']*4
>>> group_key
['east', 'east', 'east', 'east', 'west', 'west', 'west', 'west']
>>> data=Series(np.random.randn(8),index=state)
>>> data[['vermont','nevada','idaho']]=np.nan
>>> data
ohio         -1.032728
new york     -1.162002
vermont            NaN
florida      -0.571487
oregen       -0.997641
nevada             NaN
california    1.149481
idaho              NaN
dtype: float64
>>> data.groupby(group_key).mean()
east   -0.922072
west    0.075920
dtype: float64
#利用分组平均去填充na值
>>> fill_mean=lambda g:g.fillna(g.mean())
>>> data.groupby(group_key).apply(fill_mean)
ohio         -1.032728
new york     -1.162002
vermont      -0.922072
florida      -0.571487
oregen       -0.997641
nevada        0.075920
california    1.149481
idaho         0.075920
dtype: float64
复制代码
4、分组加权平均数和相关系数

根据 拆分-应用-合并 范式，DataFrame的列与列之间或两个Series之间的运算成为一种标准运算

复制代码
>>> df=DataFrame({'category':['a','a','a','a','b','b','b','b'],'data':np.random.randn(8),'weights':np.random.rand(8)})
>>> df
  category      data   weights
0        a -1.196080  0.247188
1        a -1.695342  0.914525
2        a  1.521977  0.483654
3        a  0.814892  0.267910
4        b -0.507479  0.204920
5        b -0.696985  0.097827
6        b -0.748492  0.105464
7        b  0.837663  0.404254
>>> grouped=df.groupby('category')
>>> get_wavg=lambda g:np.average(g['data'],weights=g['weights'])
>>> grouped.apply(get_wavg)
category
a   -0.466038
b    0.107713
dtype: float64
复制代码
5、面向分组的线性回归

你可以用groupby执行分组更为复杂的分组统计分析，只要函数返回的是pandas对象或者标量值即可。

透视表和交叉表

在pandas中，可以通过groupby功能以及重塑运算制作透视表，DataFrame还有一个pivot_table方法，此外还有一个顶级的pandas.pivot_table函数。

复制代码
>>> tips.pivot_table(index=['sex','smoker'])
                   size       tip  total_bill
sex    smoker                                
Female No      2.592593  2.773519   18.105185
       Yes     2.242424  2.931515   17.977879
Male   No      2.711340  3.113402   19.791237
       Yes     2.500000  3.051167   22.284500
>>> tips.pivot_table(['tip_pct','size'],index=['sex','day'],columns='smoker')
                 size          
smoker             No       Yes
sex    day                     
Female Fri   2.500000  2.000000
       Sat   2.307692  2.200000
       Sun   3.071429  2.500000
       Thur  2.480000  2.428571
Male   Fri   2.000000  2.125000
       Sat   2.656250  2.629630
       Sun   2.883721  2.600000
       Thur  2.500000  2.300000
复制代码
要使用其他的聚合函数，可将函数传入aggfunc参数即可

复制代码
>>> tips.pivot_table('size',index=['sex','smoker'],columns='day',aggfunc=len)
day            Fri  Sat  Sun  Thur
sex    smoker                     
Female No        2   13   14    25
       Yes       7   15    4     7
Male   No        2   32   43    20
       Yes       8   27   15    10
复制代码
 交叉表是一种用于计算分组频率的特殊透视表

复制代码
>>> pd.crosstab([tips.time,tips.day],tips.smoker,margins=True)
#指定行与列交叉统计，margins参数用于是否进行分项小计
smoker        No  Yes  All
time   day                
Dinner Fri     3    9   12
       Sat    45   42   87
       Sun    57   19   76
       Thur    1    0    1
Lunch  Fri     1    6    7
       Thur   44   17   61
All          151   93  244
复制代码
