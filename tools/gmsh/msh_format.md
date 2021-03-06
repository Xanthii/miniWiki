# MSH 文件说明
MSH 文件 (即扩展名为 `.msh` 的文件) 用于存储网格信息 (结点位置和连接关系) 以及与之关联的属性数据 (位移, 速度, 应力等).
MSH 文件有两种模式:
- ASCII 文本模式: 所有信息均以 ASCII 字符表示.
- 二进制模式: 一些信息由 ASCII 字符表示, 浮点型坐标值和整型编号以二进制表示.

一个 `MSH` 文件由以下一个或多个段落组成.
每个段落都是以一对`起始符`和`结束符`为界的 ASCII 字符或二进制数据块.
起始符有以下几种:

| 起始符 | 内容 |
| -------- | ---- |
| `$MeshFormat` | 网格格式 |
| `$PhysicalName` (optional) | 物理名 |
| `$Entities` | 实体 |
| `$PartitionedEntities` (optional) | 分块实体 |
| `$Nodes` | 结点 |
| `$Elements` | 单元 |
| `$Periodic` (optional) | 周期关系 |
| `$GhostElements` (optional) | 幽灵单元 |
| `$NodeData` (optional) | 结点数据 |
| `$ElementData` (optional) | 单元数据 |
| `$ElementNodeData` (optional) | 单元结点数据 |

起始符的 `$` 后加上 `End` 就是对应的结束符, 例如: 起始符 `$Nodes` 所对应的结束符为 `$EndNodes`.
相同类型的段落可以重复多次, 属性数据可以分属不同文件, 但要求 `Nodes` 位于 `Elements` 之前.

凡是未识别的起始符都视作开启一个注释段, 直到对应的结束符, 例如:
```cpp
$Comments
...
$EndComments
```

结点和单元的标签不要求连续.

下面以脚本文件 [`RECTANGLE.geo`](./RECTANGLE.geo) 生成的网格文件 [`RECTANGLE.msh`](./RECTANGLE.msh) 为例来说明 MSH 文件格式.

## `MeshFormat`
```cpp
$MeshFormat
4 0 8
$EndMeshFormat
```
三个数 (即使在二进制模式下, 也用字符表示) 的含义依次为:
- MSH 格式`版本`;
- 文件`模式`, `0` 为文本模式, `1` 为二进制模式;
- `字长`, 即 `sizeof(double)` 的值, 一般为 `8`.

## `PhysicalNames` (optional)
这一段用于描述物理对象信息, 以便后面按物理对象查找结点和单元.
```cpp
$PhysicalNames
3
1 1 "LeftBoundary"
1 2 "RightBoundary"
2 3 "Domain"
$EndPhysicalNames
```
各行的含义依次为:
- 第一行的 `3` 表示本文件中有 `3` 个物理实体;
- 随后三行分别为各物理实体的信息:
  - 前两个整数 (即使在二进制模式下, 也用字符表示) 分别表示`维度`和`标签`;
  - 最后的字符串 (最长可含 `127` 个字符) 为`名称`.

## `Entities`
这一段用于描述初等实体信息, 以便后面按初等实体查找结点和单元.

### 总数
第一行表示各维度初等实体的个数:
```cpp
4 4 1 0
```
- `4` 个零维初等实体, 即初等点;
- `4` 个一维初等实体, 即初等曲线;
- `1` 个二维初等实体, 即初等曲面;
- `0` 个三维初等实体, 即初等空间区域.

随后各行依次表示各初等实体的信息.

### `Point`s
前 `4` 行表示初等 `Point`s:
```cpp
1 0 0 0 0 0 0 0 
2 2 0 0 2 0 0 0 
3 2 1 0 2 1 0 0 
4 0 1 0 0 1 0 0 
```
各行含义如下:
- 第一个 `int` 表示该 `Point` 的`标签`.
- 随后三个 `double`s 表示`包围盒顶点坐标的最小值`.
- 随后三个 `double`s 表示`包围盒顶点坐标的最大值`.
- 随后一个 `unsigned long` 表示`物理标签的个数`:
  - 如果为 `0`, 则该行到此结束, 例如: 这里的所有各行.
  - 如果不为 `0`, 则给出相应个数 `int`, 表示该点所属`物理实体的标签`.

### `Curve`s
随后 `4` 行表示初等 `Curve`s:
```cpp
1 0 0 0 0 0 0 0 0 
2 2 0 0 2 1 0 1 2 2 2 -3 
3 0 1 0 2 1 0 0 2 3 -4 
4 0 0 0 0 1 0 1 1 2 4 -1 
```
各行含义如下:
- 第一个 `int` 表示该 `Curve` 的`标签`.
- 随后三个 `double`s 表示`包围盒顶点坐标的最小值`.
- 随后三个 `double`s 表示`包围盒顶点坐标的最大值`.
- 随后一个 `unsigned long` 表示`物理标签的个数`:
  - 如果为 `0`, 则进入下一项, 例如: 这里的第 `1`, `3` 行.
  - 如果不为 `0`, 则给出相应个数 `int`, 表示该 `Curve` 所属`物理实体的标签`, 例如:
    - `Curve{2}` 属于 `Physical Curve{2}`.
    - `Curve{4}` 属于 `Physical Curve{1}`.
- 随后一个 `unsigned long` 表示`边界点的个数`:
  - 如果为 `0`, 则该行到此结束, 例如这里的第 `1` 行.
  - 如果不为 `0`, 则给出相应个数 `int`, 表示该 `Curve` 所拥有的`边界点的标签`, 负标签值表示终点, 例如:
    - `Curve{2}` 有两个边界 `Point`s: `Point{2, -3}`.
    - `Curve{3}` 有两个边界 `Point`s: `Point{3, -4}`.
    - `Curve{4}` 有两个边界 `Point`s: `Point{4, -1}`.

### `Surface`s
`Surface` 段的格式与 `Curve` 段类似, 只不过把最后的`边界点`替换为`边界曲线`.

这里有 `1` 行表示初等 `Surface`:
```cpp
1 -1.8e+308 -1.8e+308 -1.8e+308 1.8e+308 1.8e+308 1.8e+308 1 3 4 1 2 3 4 
```
各行含义如下:
- 第一个 `int` 表示该 `Surface` 的`标签`.
- 随后三个 `double`s 表示`包围盒顶点坐标的最小值`, 这里是负无穷.
- 随后三个 `double`s 表示`包围盒顶点坐标的最大值`, 这里是正无穷.
- 随后一个 `unsigned long` 表示`物理标签的个数`:
  - 如果为 `0`, 则进入下一项.
  - 如果不为 `0`, 则给出相应个数 `int`, 表示该 `Surface` 所属`物理实体的标签`. 例如这里的 `Surface{1}` 属于 `Physical Surface{3}`.
- 随后一个 `unsigned long` 表示`边界曲面的个数`:
  - 如果为 `0`, 则该行到此结束, 例如这里的第 `1` 行.
  - 如果不为 `0`, 则给出相应个数 `int`, 表示该 `Surface` 所拥有的`边界曲线的标签`, 负标签值表示取向相反. 例如:
    - `Surface{1}` 拥有 `4` 条边界 `Curve`s: `Curve{1, 2, 3, 4}`.

### `Volume`s
`Volume` 段的格式与 `Surface` 段类似, 只不过把最后的`边界曲线`替换为`边界曲面`.

## `PartitionedEntities` (optional)
暂时不用.

## `Nodes`
这一段用于描述`结点位置`以及`结点与实体的归属关系`.

第一行表示各维度实体的个数:
```cpp
15 6
```
- 第一个 `unsigned long` 表示`实体总数`. 这里有 `15` 个实体:
    - `6` 个 `Point`s,
    - `7` 条 `Curve`s,
    - `2` 片 `Surface`s,
    - `0` 块 `Volume`.
- 第二个 `unsigned long` 表示`结点总数`. 这里有 `6` 个 `Node`s.

然后以每个实体为一组, 依次列出各组所拥有的结点.
例如第一行:
```cpp
5 0 0 1
```
- 第一个 `int` 表示`实体标签`.
- 第二个 `int` 表示`实体维度`.
- 第三个 `int` 表示`参数个数`.
- 最后一个 `unsigned long` 表示该实体上的`结点个数`. 这里为 `1`, 因此紧随其后的 `1` 行表示该实体上的 `1` 个结点的信息:
```cpp
1 0 0 0
```
- 第一个 `int` 表示`结点标签`.
- 紧随其后的三个 `double`s 表示`结点坐标`.

其他实体上的结点信息可以按同样的方法读出.

## `Elements`
这一段用于描述`单元类型`, `结点与单元的归属关系`以及`单元与实体的归属关系`.

第一行表示各维度实体的个数:
```cpp
5 5
```
- 第一个 `unsigned long` 表示`实体总数`. 这里有 `5` 个实体:
    - `0` 个 `Point`s,
    - `3` 条 `Curve`s,
    - `2` 片 `Surface`s,
    - `0` 块 `Volume`.
- 第二个 `unsigned long` 表示`单元总数`. 这里有 `5` 个 `Element`s:
    - `0` 个零维单元,
    - `3` 个一维单元,
    - `2` 个二维单元,
    - `0` 个三维单元.

然后以每个实体为一组, 依次列出各组所拥有的单元.
例如第一行:
```cpp
7 1 1 1
```
- 第一个 `int` 表示`实体标签`.
- 第二个 `int` 表示`实体维度`.
- 第三个 `int` 表示`单元类型`. 这里为 `1`, 表示`二结点直线单元`. 
- 最后一个 `unsigned long` 表示该实体上的`单元个数`. 这里为 `1`, 因此紧随其后的 `1` 行表示该实体上的 `1` 个单元的信息:
```cpp
7 2 3 
```
- 第一个 `int` 表示`单元标签`.
- 紧随其后的 `int`s 表示`结点标签`.
- 这一行的完整语义是: `7` 号单元有 `2` 个结点 (从单元类型推断), 结点标签分别为 `2` 和 `3`.

接下来四行表示另外两个一维单元:
```cpp
10 1 1 1
10 4 1 
11 1 1 1
13 6 5 
```
最后四行表示两个二维 (四结点四边形) 单元:
```cpp
$Elements
2 2 3 1
11 1 5 6 4 
3 2 3 1
12 5 2 3 6 
$EndElements
```

### 单元结点编号

完整列表参见 [Gmsh Reference Manual](http://gmsh.info/doc/texinfo/gmsh.html) 的 [9.2 Node ordering](http://gmsh.info/doc/texinfo/gmsh.html#Node-ordering).

## `Periodic` (optional)
暂时不用.

## `GhostElements` (optional)
暂时不用.

## `NodeData` (optional)
暂时不用.

## `ElementData` (optional)
暂时不用.

## `ElementNodeData` (optional)
暂时不用.

## `InterpolationScheme` (optional)
暂时不用.
