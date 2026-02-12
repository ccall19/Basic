# Pandas
Pandas [官方文档](https://pandas.pydata.org/docs/getting_started/index.html)

官方文档写的还是挺不错的，Intro to pandas 提出了很多关于Pandas的问题，挨个看下来就是对于Pandas 的很好学习了。

## 1、What kind of data does pandas handle?
Pandas 处理的 数据类型 被称为 DataFrame（实际不止这一个，不过都是基于DataFrame延伸的一些概念，后续会解释），DataFrame 是一个二维的表格，column 由 label 指引，row 由 index 指引。

## 2、How do I read and write tabular data?
Pandas 支持把多种外部数据类型转为 DataFrame 格式，API也非常简单直观：The ability to import data from each of these data sources is provided by functions with the prefix, `read_*`. Similarly, the `to_*` methods are used to store data.

## 3、How do I select a subset of a table?
这应该是一个重头戏吧。教程里分了三步：      

### 3.1、How do I select specific columns from a DataFrame?     
---
列 的话 可以通过 label 名 来选取：`ages = titanic["Age"]`，像这样，DataFrame 中选出来的每一列 都被称为一个 Series，Series也是一个类。我们也可以选择多个列：`titanic[["Age", "Sex"]]`，就像是 Python 的列表。Series 是一维的，所以选择多个列的话，返回的数据类型就又是 DataFrame了，一列就返回 Series。

### 3.2、How do I filter specific rows from a DataFrame?      
--------
Filter(与 select 的区别：select 更加明确，更具有针对性) `行` 的话，比较有意思，可以依靠 类似 列表表达式 的东西：     
````py
above_35 = titanic[titanic["Age"] > 35]
````
> 单纯的 表达式 在 Pandas 中返回的是一个 由布尔值组成的 Pandas Series，`titanic["Age"] > 35`

除了利用简单的大小比较运算，Pandas 还提供了一个 isin 的API，用来利用数值范围取值，比如：
````py
class_23 = titanic[titanic["Pclass"].isin([2, 3])]
````
以及  或（or）运算符 `|` ：
````py
class_23 = titanic[(titanic["Pclass"] == 2) | (titanic["Pclass"] == 3)]
````

### 3.3、How do I select specific rows and columns from a DataFrame?      
----
主要是俩 方法： `loc()`、 `iloc()`：
- **loc()**
````py
adult_names = titanic.loc[titanic["Age"] > 35, "Name"]
````
俩参数，逗号前选择行，逗号后选择列。iloc() 也是这样（插播了，丢个炸弹💣）

- **iloc()**        

i 的意思是 integer 。因此 明确自己要选择的行列位置时使用：`iloc()` 
````py
titanic.iloc[9:25, 2:5]
````

## 4、How to create plots in pandas?
Pandas 借助 matplotlib 之神力 实现开箱即用的绘图能力，只需选择与数据对应的图表类型（散点图、条形图、箱线图等）。比如：
````py
# 简单的绘制
dataframe.plot()
# 散点图
dataframe.plot.scatter(x="station_london", y="station_paris", alpha=0.5)
#箱线图
dataframe.plot.box()
````

## 5、How to create new columns derived from existing columns?
基于现有数据派生出一个新的列，并不需要循环每一行。pandas 中列数据操作是逐元素进行的。

比如，可以直接以表达式赋值的形式，四则运算与逻辑运算具可矣：
````py
air_quality["london_mg_per_cubic"] = air_quality["station_london"] * 1.882
````
还可以对列重命名，此处不细说了。

## 6、How to calculate summary statistics?
均值，中位数啥的，使用比较简单：
````py
titanic["Age"].mean()
titanic[["Age", "Fare"]].median()
titanic[["Age", "Fare"]].describe() # 这个就是可以把指定列的几乎所有的统计数据都列出来，すごい
````
除了这些预定义好的统计数据，Pandas 还可以指定 统计数据的 聚合，使用 `.agg()` 方法：
````py
titanic.agg(
    {
        "Age": ["min", "max", "median", "skew"],
        "Fare": ["min", "max", "median", "mean"],
    }
)
````

### 再详细一些：Aggregating statistics grouped by category     
使用 `.groupby()` 方法 对列的数据进行分组归类后 再计算统计数据： 
````py
titanic[["Sex", "Age"]].groupby("Sex").mean()
````
````sh
Out[8]: 
              Age
Sex              
female  27.915709
male    30.726645
````
也可以对多个 label 归类：
````py
titanic.groupby(["Sex", "Pclass"])["Fare"].mean()
````

## 7、How to reshape the layout of tables?
### Sort table rows
````py
titanic.sort_values(by="Age").head()
titanic.sort_values(by=['Pclass', 'Age'], ascending=False).head()
````
可以看出，默认升序。

## 8、How to combine data from multiple tables?
````py
air_quality = pd.concat([air_quality_pm25, air_quality_no2], axis=0)
````