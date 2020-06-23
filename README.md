## 0. 问题
一个floor层高的楼里，有cup个杯子。当杯子从某一层扔下时，由于楼层的高度不同，可能会碎掉也可能不会碎。

问题：

1. 最差的情况下，扔几次可以检测出杯子破碎的楼层高度？
2. 应该如何来扔？

## 1. 简略分析
### 1.1 初步分析
这个问题可以先从两个极端考虑：
* 如果只有1个杯子的话，那没啥说的了，从floor=1开始，一层一层往上扔，直到某一层碎掉为止。
* 另一个极端，如果有无数个杯子，那妥了，第一次可以扔floor/2，接下来就按照二分查找进行下去就可以了。
* 仔细考虑一下第二个极端，其实这个极端是有一个边界的：如果cup>=log2(floor)。超出这个编辑，就是二分查找。
* 所以，我们需要讨论的区间，就是2<=cup<log2(floor)

### 1.2 再一次分析
回过头来，我们从一个一般化的角度捋一下这个问题。

比如说，cup=3，floor=100层，如果我们第一次扔50层（假设二分查找）。如果杯子碎了，那这个问题就变成两个杯子扔49层；如果杯子没碎，就变成3个杯子扔50层。

这样看的话，第一次扔50层就不合理了，至少应该扔在3：2的位置，也就是40层。

但是，再往下扔呢？第一次的上半部分又变成了3：2，而下半部分却是2：1。这对第一次扔杯子的影响变成了5：3。也就是说第一次应该扔100/8*3=37.5层。

如此反复下去，分析下去，除非把最终的可能性都考虑清楚，否则我们连第一步都走不出去。

### 1.3 进一步分析
看来这个问题无法简单地一层层剥开，那就从一个整体来考虑——二叉树（为了描述方便，本文中出现的"二叉树"均指二叉查找树）。

像前面初步分析的那样，如果杯子不会碎，我们的二分查找就相当于检索一棵完全二叉树（注意不是满二叉树）。

n阶的满二叉树指的是：每层（第m层）有2^(m-1)个节点的二叉树；
n阶完全二叉树指的是除了第n层，每层（第m层）有2^(m-1)个节点，第n层节点从左到右排列。
在完全二叉树里没有子节点的节点只能存在于第n阶或者第n-1阶，而且不会出现只有右子节点的节点。
所以说，满二叉树是第n阶正好有2^(n-1)个节点的特殊地完全二叉树。

但是，毕竟杯子还是会碎的，所以这个二叉树将不再是一个完全二叉树，而是一个镂空的二叉树，镂空的点就是那些杯子碎掉的位置。
以下，我们来详细分析这个镂空的二叉树的形态。

## 2. 详细分析
为了描述问题方便，我们假设二叉树的左子树表示杯子碎掉后向下查找，右子树表示杯子没碎向上查找。

### 2.1 当cup>=log2(floor)时
为了能够更加形象的分析问题，我们先假设floor=10，于是我们可以构成这样的一个完全二叉树（T1）进行二分查找：

          7
       ┌──┴──┐
       4     9
     ┌─┴─┐ ┌─┴─┐
     2   6 8   10
    ┌┴┐ ┌┘
    1 3 5
            
这棵二叉树表示的含义为：第一次将杯子扔到7层，如果没碎，就扔9层，否则就扔4层，以此类推，最终会在4步之内解决。

有人会问了，不是说好的二分查找吗？怎么从7开始了呢？

其实不论从7开始还是从5开始，结果都是差不多的。下面就是一个从5开始的二叉树（T2），我们来看看它跟T1有啥区别：

           5
       ┌───┴───┐
       3       8
     ┌─┴─┐   ┌─┴─┐
     2   4   7   10
    ┌┘      ┌┘  ┌┘
    1       6   9

我们可以看到T1和T2都是4阶平衡二叉树（任意节点的左子树和右子树的高度差<=1），也就是说不论我们用T1的方式还是T2的方式进行二分查找，其结果都一样——最多4次。

我们甚至可以构成一个非平衡二叉树T3，也可能达到类似的查找效果：

          5
       ┌──┴──┐
       4     8
     ┌─┘   ┌─┴─┐
     2     7   10
    ┌┴┐   ┌┘  ┌┘
    1 3   6   9

需要注意的是，T3有4个节点在第四阶，说明其查找效率要低于T1和T2，这正是非平衡二叉树的弱点。

另一方面，用程序来构成T2或T3这样的非完全二叉树，其实是比较困难的，因为这样的二叉树没有一定之规。

相比之下，恰恰是T1这种完全二叉树更容易构成，所以接下来的讨论，我们会专注于完全二叉树。

### 2.2 当 cup<log2(floor) 时
上述#1.3中，我们得出结论说要构成一个镂空的二叉树。那么，这个镂空是什么样子呢？

我们依然假设floor=10，当cup=3时，T1会发生什么样的变化（T4）？

         7
      ┌──┴──┐
      3     9
    ┌─┴─┐ ┌─┴─┐
    1   5 8   10
    └┐ ┌┴┐
     2 4 6

先观察一下T4的结构，注意到T4右下角的那个空缺了吗？T4中节点1占据了T1的节点2的位置，而且T4的节点1没有左子树。

如果你没有想清楚为什么的话，请重温上述#2下第一行的描述。是的，在这里3个杯子都碎掉了，不能再有左子树了。

这就是我们基于完全二叉树（T1）构成的镂空二叉树（T4）。

这里再多说一句，如果cup=4，那T4就会变回T1，因为4>log2(10)。

如果cap=2，又会有怎样的变化呢（T5）？

       4
    ┌──┴──┐
    1     7
    └┐  ┌─┴─┐
     2  5   9
     └┐ └┐ ┌┴┐
      3  6 8 10

T5恰好是一个基于满二叉树的镂空树，这只是一个巧合罢了。

到此为止，分析结束。如果各位有任何疑问，请直接联系作者：imddl@outlook.com

## 3. 解题过程
很多问题都是想着容易做起来难。这个问题也是这样，只不过它是想着难，做起来更难 ;-)

### 3.1 二叉树的结构
上述分析的过程中出现的T4和T5都是针对floor=10的楼，但是形态相差非常大，貌似floor与我们的二叉树的形态关系不大。

那cup数呢？会不会有关呢？

我们先假定cup=3，由于3=log2(8)，所以我们先关注floor>8情况。

floor=9：

         6
      ┌──┴──┐
      3     8
    ┌─┴─┐ ┌─┴─┐
    1   5 7   9
    └┐ ┌┘
     2 4

floor=10即为上述T4：

         7
      ┌──┴──┐
      3     9
    ┌─┴─┐ ┌─┴─┐
    1   5 8   10
    └┐ ┌┴┐
     2 4 6

floor=11：

          7
      ┌───┴───┐
      3       10
    ┌─┴─┐   ┌─┴─┐
    1   5   9   11
    └┐ ┌┴┐ ┌┘
     2 4 6 8

floor=12：

          7
      ┌───┴───┐
      3       11
    ┌─┴─┐   ┌─┴─┐
    1   5   9   12
    └┐ ┌┴┐ ┌┴┐
     2 4 6 8 10

先到这里，规律已经很明显了：镂空树的结构，基本就是基于给定cup值的镂空树的广度优先的生长过程。

而且在生长的过程中，虽然结构只发生简单的变化，但是节点的数值会有波动。接下来我们先专注于结构本身，而忽略每个节点的数值。

当cup=3的镂空树生长到floor=41时，是这个样子（T7）：

                  0                         ----  1
        ┌─────────┴─────────┐
        0                   0               ----  2
    ┌───┴───┐        ┌──────┴──────┐
    0       0        0             0        ----  4
    └┐   ┌──┴──┐   ┌─┴──┐      ┌───┴───┐
     0   0     0   0    0      0       0    ----  7
     └┐  └┐  ┌─┴─┐ └┐ ┌─┴─┐  ┌─┴─┐   ┌─┴─┐
      0   0  0   0  0 0   0  0   0   0   0  ---- 11
      └┐  └┐ └┐ ┌┴┐ └┐└┐ ┌┴┐ └┐ ┌┴┐ ┌┴┐ ┌┴┐
       0   0  0 0 0  0 0 0 0  0 0 0 0 0 0 0 ---- 16

这既是一个cup=3的6阶镂空树。右侧的数字表示的是当前层的节点总数。这个数字有什么规律吗？

### 3.2 节点数的规律
通过对T7结构的观察，我们可以发现
* 对于一个cup=n的m阶镂空树，其root节点的左子树是一棵cup=n-1的m-1阶镂空树
* 其root节点的右子树是一棵cup=n的m-1阶镂空树

也就是说对于原镂空树的第i层节点数，其实就是另外两个m-1阶镂空树的i-1层节点数之和。
这个规律是我们计算节点数的基本规则（虽然还可以优化）。

为了能够进行计算，我们还需要两个非常简单的初始条件：
* 每棵镂空树的根节点的数目是1
* 当cup=1时，每层的节点数为:1,1,1,1,1 ... 1

不要笑，这两句貌似废话的条件缺一不可，他们构成了这样一个矩阵的初始状态（Table1）：

|     | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 |
|-----|---|---|---|---|---|---|---|---|---|
|cup=1| 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
|cup=2| 1 | a | b
|cup=3| 1
|cup=4| 1
|cup=5| 1
|cup=6| 1
|cup=7| 1
|cup=8| 1
    
为了讨论问题方便，我们假设上表中cup=1时镂空树的第1层节点数为1的那个点为原点，向左为x轴正方向，向下为y轴正方向。
则a的位置是(1,1)，b的位置是(2，1)。

a的值v(a) = v(1,1) = v(0,1) + v(0,0) = 1 + 1 = 2

b的值v(b) = v(2,1) = v(1,1) + v(1,0) = v(a) + 1 = 2 + 1 = 3

通项公式（M1）为：

    v(x,y) = v(x-1,y) + v(x-1,y-1)

通过计算，我们可以将上表填充完整（Table2）：

|     |  1  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |  9  |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|cup=1|  1  |  1  |  1  |  1  |  1  |  1  |  1  |  1  |  1  |
|cup=2|  1  |  2  |  3  |  4  |  5  |  6  |  7  |  8  |  9  |
|cup=3|  1  |  2  |  4  |  7  | 11  | 16  | 22  | 29  | 37  |
|cup=4|  1  |  2  |  4  |  8  | 15  | 26  | 42  | 64  | 93  |
|cup=5|  1  |  2  |  4  |  8  | 16  | 31  | 57  | 99  | 163 |
|cup=6|  1  |  2  |  4  |  8  | 16  | 32  | 63  | 120 | 219 |
|cup=7|  1  |  2  |  4  |  8  | 16  | 32  | 64  | 127 | 247 |
|cup=8|  1  |  2  |  4  |  8  | 16  | 32  | 64  | 128 | 256 |

这张表的含义为：对于指定的cup数（比如说5），如果想检测floor层楼（比如说100），需要查看表中cup（5）这行，看看floor（100）落在哪个区间（163）。
这个区间对应的阶数，就是最差情况下，需要扔杯子的次数。到目前为止，我们已经解决了问题1。

### 3.3 构成并填充镂空树
回答了问题1，问题2就容易了。我们只需要将这个m阶的镂空树构成出来并将数字填充进去就好了

构成cup=c且包含floor=f个节点的镂空树：
* 从根节点出发，以广度优先的方式创建子节点
* 设定当前破碎杯子数b=0
* 每次创建左节点前，判断如果b<c则创建左节点，否则只能创建右子节点
* 每次创建完左节点后，b++

填充镂空树前，我们定义如下参数：
* v - 节点的数值
* cl - 节点的左子孙数
* cr - 节点的右子孙数
* pv - 父节点数值
* pcl - 父节点的左子孙数
* pcr - 父节点的右子孙数

为了填充cup=c且深度为n阶的镂空树，我们需要遍历两次镂空树，第一次是从第n-1阶向root节点进行反向的广度优先遍历
* 如果当前节点是其父节点的左子节点，则将此节点的左子孙数pcl=cl+cr+1
* 如果当前节点是其父节点的右子节点，则其父节点的右子孙数pcr=cl+cr+1
* 如果当前节点没有父节点，则结束遍历

第二次是从root节点遍历镂空树，此次遍历不限定广度优先或者深度优先。
* 如果当前节点没有父节点，则此节点的数值v=cl+1
* 如果当前结点是其父节点的左子节点的话，则v=pv-cr-1
* 如果当前结点是其父节点的右子节点的话，则v=pv+cl+1

到这里，问题2也得到了解答。
而且，有意思的一点是：问题1和问题2可以分别使用独立地方法进行解决，也就是说#3.2和#3.3是分别独立的算法。

## 4. Golang算法
为了验证我们上述算法的正确性，我们使用Go语言进行编程，完成上述#3.2和#3.3中的算法。

### 4.1 计算镂空树的degree
先放代码：
```go
func InnerCalculateA(floor, cup int) (degree int) {
    list := make([]int, cup)
    for c := 0; c < cup; c++ {
        list[c] = 1
    }

    sum := 1
    for degree = 1; sum < floor; degree++ {
        calList := make([]int, cup)
        calList[0] = 1
        for c := 1; c < cup; c++ {
            calList[c] = list[c] + list[c-1]
        }
        list = calList
        sum += calList[cup-1]
    }
    return
}
```
参照公式M1我们需要构建两个长度为cup的数组（其实是slice切片，为了描述方便，本文中全部使用数组这个词）

第一个数组list，其中的元素全为1，代表Table2表中的第一列；

第二个数组calList，它的第0个元素为1，其余各个元素通过公式calList[i]=list[i]+list[i-1]计算得出。

calList中全部元素计算完成后，将其赋给list，并将calList[cup-1]的值进行累加（sum的初始值为1，这个1其实就是初始的list的最后那个元素1），然后继续计算右侧的下一列。

这里需要两个循环，内循环计算calList的各个元素；外循环进行累加和list<=>calList切换，退出的条件是sum>=floor。外循环执行的次数就是我们要计算的degree值。

#### 优化
细心地读者可能会想到，如果cup>=log2(floor)的话，degree直接返回log2(floor)就可以了，完全不需要循环嵌套的计算。

是的，所以我们要在上述foo外面增加一个预处理：
```go
func Calculate(floor, cup int, innerCalculate func(int, int) int) (degree int) {
    // 1.0 If eggs are enough then the binary tree is a non-hollow tree
    log2Floor := math.Log2(float64(floor))
    if float64(cup) >= log2Floor {
        degree = int(math.Ceil(log2Floor))
        return
    }

    return innerCalculate(floor, cup)
}
```
注意，这里的foo函数是作为参数传进来并赋给innerCalculate()的，所以调用时要写成：

    d := bar(n, m, foo) // n为floor的值，m为cup的值，d为计算degree的结果

#### 另一种算法
上述#4.1的求解过程，使用的是公式M1，这个公式很简单（而且实现起来效率也不错），但是有一点不尽如人意：需要将前面cup数1~n-1的值都计算出来。
有没有办法，只看cup=n这一行，就可以进行计算呢？

办法倒是有，只不过分析起来很绕。其中涉及到Pascal三角的通项公式，及其每一行的求和。这里只给出公式（M2）不做详细讨论。

    v(x,y) = ∑n=1~y(∑m=0~x(C(n,m)))

这是一个嵌套的求和过程，后面的C(n,m)是组合公式。感兴趣的读者可以参照degree.go文件里的InnerCalculateB()方法。

这个公式不仅复杂，其中还涉及到大量的乘法和除法运算，所以效率反而低。degree_test.go中有对比这两种方法的benchmark，下面是benchmark结果比较：

    goos: windows
    goarch: amd64
    pkg: int-floor-cup/degree
    BenchmarkInnerCalculateA-8        352951              3176 ns/op
    BenchmarkInnerCalculateB-8         69752             17591 ns/op
    PASS
    ok      int-floor-cup/degree    2.902s

差的还不是一星半点呢，可惜作者挖空心思的想到这么漂亮的公式，居然这么中看不中用！

### 4.2 计算扔杯子的楼层
为了计算扔杯子的楼层，我们需要先构建如下的struct：
```go
type node struct {
	Value      int   `json:"V"`
	Left       *node `json:"L"`
	Right      *node `json:"R"`
	Parent     *node `json:"-"`
	LeftCount  int   `json:"-"`
	RightCount int   `json:"-"`
	Remain     int   `json:"-"`
	IsLeft     bool  `json:"-"`
}
```
* Value 表示当前节点的数值
* Left 是指向左节点的指针
* Right 是指向右节点的指针
* Parent 是指向父节点的指针
* LeftCount 表示左子树的全部节点数
* RightCount 表示右子树的全部节点数
* Remain 用来在镂空树生长过程中记录杯子破碎的次数
* IsLeft 标记当前节点是其父节点的左/右子树

#### 构建镂空树
数据结构定义好之后，我们来看看addNode()方法
```go
func addNode(parent *node, nodeList *[]*node, single bool) (count int) {
	if parent.Remain > 0 {
		left := node{}
		left.Parent = parent
		left.Remain = parent.Remain - 1
		left.IsLeft = true
		parent.Left = &left
		*nodeList = append(*nodeList, &left)

		if single {
			return 1
		} else {
			count = 2
		}
	} else {
		count = 1
	}

	right := node{}
	right.Parent = parent
	right.Remain = parent.Remain
	parent.Right = &right
	*nodeList = append(*nodeList, &right)
	return
}
```
逻辑很简单：
* 如果还有未破随的杯子（Remain>0)，就创建左节点，并且在创建之后将子节点的Remain值减1。
* 不论Remain为何值，都可以添加右节点。
* 此处还有一个额外的逻辑是：加节点到最后一刻，如果只剩1个节点，那只能指定single=true之后再调用addNode()，这时就不能添加右节点了。
* 节点添加完成之后，返回添加节点的个数。

注意，addNode()方法并不是直接被调用的，真正被调用的是addBothNode()和addSingleNode()。
```go
	sum := 1
	for i := 1; i < degree-1; i++ {
		nodeList[i] = make([]*node, 0, int(math.Pow(2, float64(i))))
		for _, eachNode := range nodeList[i-1] {
			sum += addBothNode(eachNode, &nodeList[i])
		}
	}
	restNodeNumber := floor - sum
	for _, eachNode := range nodeList[degree-2] {
		if restNodeNumber == 0 {
			break
		}
		if restNodeNumber == 1 {
			addSingleNode(eachNode, &nodeList[degree-1])
			break
		}
		restNodeNumber -= addBothNode(eachNode, &nodeList[degree-1])
	}
```
上述第一个循环体内是对镂空树内除了bottom阶的各个节点的添加，nodeList是广度优先遍历需要用到的额外存储空间，其中按顺序存储每一阶的各个节点。
上述第二个循环体内是对镂空树的bottom阶节点的添加。restNodeNumber为1则调用addSingleNode()否则调用addBothNode()。循环的退出条件是restNodeNumber为0。

#### 计算左右子孙数——逆向广度优先遍历
从镂空树的倒数第二阶（n-1）向root遍历，对每个节点计算其左右子孙数。
具体算法参照#3.3
```go
func adjustCount(nodeList *[]*node) {
	for _, eachNode := range *nodeList {
		if eachNode.Left != nil {
			eachNode.LeftCount = eachNode.Left.LeftCount + eachNode.Left.RightCount + 1
		}
		if eachNode.Right != nil {
			eachNode.RightCount = eachNode.Right.LeftCount + eachNode.Right.RightCount + 1
		}
	}
}
```

#### 计算节点值——正向广度优先遍历
从镂空树的root节点向下遍历，计算每个节点的值。
具体算法参照#3.3
```go
func fillValue(nodeList *[]*node) {
	for _, eachNode := range *nodeList {
		if eachNode.Parent == nil { // root
			eachNode.Value = eachNode.LeftCount + 1
		} else {
			if eachNode.IsLeft { // Left node
				eachNode.Value = eachNode.Parent.Value - eachNode.RightCount - 1
			} else { // Right node
				eachNode.Value = eachNode.Parent.Value + eachNode.LeftCount + 1
			}
		}
	}
}
```

#### 输出
到此为止，整个扔杯子问题解决完毕。为了把结构直观地呈现出来，我们使用OutputJson()方法将镂空树输出为JSON文件。

## 5. 后记
作者是在几周前，跟从朋友的闲聊过程中，听说这个问题的。当时朋友建议使用动态规划来解决此问题，但是作者连动态规划的基本原理都不了解，所以只能从自己的理解出发，尝试解决问题。

理论上说，这种复杂的问题会有很多种解法，solution_c.go文件中就是作者的一位好朋友提供的方法。

这些方法从思路上来说都是条条大路通罗马，完全没有优劣之分。只不过有的方法赶巧效率高，有的方法效率低些，有的方法会更早地遇到整数越界而导致结果错误而已。

希望能有更多的朋友对这个问题提供解法，也希望能够得到大家对上述解题过程中的不充分或者不完整之处进行指点。

最后，送上四个字<big>**进无止境**</big>与各位共勉之。
