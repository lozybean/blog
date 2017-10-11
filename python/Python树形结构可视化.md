Title: 树形结构数据可视化展示
Date: 2016-01-12
Category: bioinformatics
Tags: Python, ete3, newick

## 树形数据结构

树状结构是一种经典的数据类型, 一个树形结构的一层可以用一个父节点以及若干子节点表示, 而若干层这种结构则组成一颗树;

如一个完全二叉树的例子:
![完全二叉树](https://upload.wikimedia.org/wikipedia/commons/5/57/Binary-tree-structure.png)

节点是树的基本组成, 节点除了层次特征之外, 还可以具有枝长等节点本身的值, 定义一个节点, 如:

```python
class Node(object):

    def __init__(self, name, branch=None):
        self.name = name
        self.up = None
        self.children = []
        self.__branch = branch

    @property
    def branch(self):
        return self.__branch

    @branch.setter
    def branch(self, value):
        try:
            self.__branch = float(value)
        except ValueError:
            pass
```

上述节点具有一个父节点: `up`, 以及一个子节点列表: `children`, 并且还有自身属性: `branch`等, 描述了一个节点基本的元素;

为了操作节点数据, 需要添加一些常用的方法, 如增删改查等, 如:

```python
class Node(object):

	def add_child(self, child=None, name=None, branch=None):
	    """
	    Adds a new child to this node.
	    """
	    if child is None:
	        child = self.__class__()
	    if name is not None:
	        child.name = name
	    if branch is not None:
	        child.branch = branch
	    if child not in self.children:
	        self.children.append(child)
	    child.up = self
	    return child

    def remove_child(self, child):
        """
        Removes a child from this node (parent and child
        nodes still exit but are no longer connected).
        """
        try:
            self.children.remove(child)
        except ValueError as e:
            raise TreeError("child not found")
        else:
            child.up = None
            return child

	def add_sister(self, sister=None, name=None, branch=None):
	    """
	    Adds a sister to this node.
	    """
	    if self.up is None:
	        raise TreeError("A parent node is required to add a sister")
	    else:
	        return self.up.add_child(child=sister, name=name, branch=branch)

	def get_sisters(self, include=False):
	    """
	    Returns an indepent list of sister nodes.
	    """
	    if self.up is not None:
	        if include:
	            return [ch for ch in self.up.children]
	        else:
	            return [ch for ch in self.up.children if ch != self]
	    else:
	        return []
```

上述方法实现两个最基本操作: 添加节点, 删除节点, 获取节点等;

树结构具有三种遍历方式: 先序遍历, 中序遍历, 后序遍历; 具体可以参考[维基百科](https://zh.wikipedia.org/wiki/树的遍历);

在Python中可以有如下实现:

```python
from collections import deque

class TreeError(Exception):
    """
    A problem occurred during a TreeNode operation
    """
    def __init__(self, value=''):
        self.value = value
    def __str__(self):
        return repr(self.value)


class Node(object):

    def is_leaf(self):
        """
        Return True if current node is a leaf.
        """
        return len(self.children) == 0

    def _iter_descendants_postorder(self, is_leaf_fn=None):
        to_visit = [self]
        if is_leaf_fn is not None:
            _leaf = is_leaf_fn
        else:
            _leaf = self.__class__.is_leaf

        while to_visit:
            node = to_visit.pop(-1)
            try:
                node = node[1]
            except TypeError:
                # PREORDER ACTIONS
                if not _leaf(node):
                    # ADD CHILDREN
                    to_visit.extend(reversed(node.children + [[1, node]]))
                else:
                    yield node
            else:
                # POSTORDER ACTIONS
                yield node

    def _iter_descendants_levelorder(self, is_leaf_fn=None):
        """
        Iterate over all desdecendant nodes.
        """
        to_visit = deque([self])
        while len(to_visit) > 0:
            node = to_visit.popleft()
            yield node
            if not is_leaf_fn or not is_leaf_fn(node):
                to_visit.extend(node.children)

    def _iter_descendants_preorder(self, is_leaf_fn=None):
        """
        Iterator over all descendant nodes.
        """
        to_visit = deque()
        node = self
        while node is not None:
            yield node
            if not is_leaf_fn or not is_leaf_fn(node):
                to_visit.extendleft(reversed(node.children))
            try:
                node = to_visit.popleft()
            except IndexError:
                node = None

	def traverse(self, strategy="levelorder", is_leaf_fn=None):
        """
        Returns an iterator to traverse the tree structure under this
        node.
        """
        if strategy == "preorder":
            return self._iter_descendants_preorder(is_leaf_fn=is_leaf_fn)
        elif strategy == "levelorder":
            return self._iter_descendants_levelorder(is_leaf_fn=is_leaf_fn)
        elif strategy == "postorder":
            return self._iter_descendants_postorder(is_leaf_fn=is_leaf_fn)

```

上述方法描述了一个对树形结构的三种遍历方式, 在树形结构的遍历, 查找, 删除等等操作基础上, 可以实现更加复杂的树形结构操作;

## newick格式

[newick](http://evolution.genetics.washington.edu/phylip/newicktree.html)则是为了计算机友好的树形结构展示, 在1857年由英国数学家[Arthur Cayley](http://www-history.mcs.st-andrews.ac.uk/history/Biographies/Cayley.html)提出;

这种结构使用一个括号来表示一个节点:

如:

```newick
()demo1
```

表示一个名称为demo1的节点, 前面的括号表示具有若干子节点, 如:

```newick
(child1, child2)demo1
```

表示demo1节点以及他的两个子节点, 子节点之间使用逗号`,`分隔, 节点除了名字之外, 也可以拥有枝长的属性, 使用冒号可以给节点赋予该属性, 如:

```newick
(child1: 5, child2: 2)demo1: 6
```

多个上述节点可以组成树, 树必须由单个根节点出发, 并向下延伸, 在末尾添加分号`;`表示树, 如:

```newick
((child1: 5, child2: 2)demo1: 6, (child3: 3, child4: 3, child5: 2)demo2: 1);
```

上述描述了一颗匿名树, 树的根节点名称被隐藏, 并直接在节点后面加上分号表示结尾;

以上就是newick格式的结构, 其表示非常简单明了, 并且易于被计算机所接受, 如在前面的节点结构上, 只需加入简单的方法, 即可完成newick格式的输出:

```python
class Tree(object):
    def __init__(self, root=None):
        if root is None:
            self.__root_node = TreeNode('root', level='root')
        else:
            self.__root_node = root
        self.__root_node.tree = self
        self.nodes = defaultdict(TreeNode)
        self.nodes['root'] = self.root

    def __str__(self):
        return str(self.root) + ';'

    @property
    def root(self):
        return self.__root_node

class Node(object):
    def __str__(self):
        if self.children:
            result = '('
            result += ','.join(map(str, self.children))
            result += ')%s' % self.name
        else:
            result = self.name

        return result
```

上述过程描述了一个节点的递归生成newick格式过程, 以及树形结构的newick格式即根节点之后添加一个分号;

## 利用ete3做树形结构可视化

newick格式是计算机友好的格式, 但是却不利于我们人类观察树形结构的特点, 将newick格式进行可视化操作, 绘制出漂亮的图表显得尤为重要;

[ETE](http://etetoolkit.org)工具旨在解决newick格式的可视化展示问题, 该工具提供了很多可视化特性, 除了基本的树结构, 枝长, 节点大小等之外, ETE包支持在节点上添加各种Face, 更好描述一个节点的特性, 如:

* `TextFace`: 文字描述, 可以在节点的枝上各个位置添加介绍性文字或者数据;
* `CirCleFace`: 圆形图标, 可以在节点的某些位置添加圆形图标, 并利用半径, 面积等图形特性描述一个节点的数据特征;
* `PieCharFace`: 饼图, 可以在节点某些位置添加饼图, 配合相应的说明, 描述一个节点的数据组成特征;
* `ImgFace`: 图形, 可以在节点的某些位置添加图形, 使用直观的图形来描述一个节点的性状特征;
* `TreeFace`: 子树, 在叶节点上添加绘制子树, 某些情况下可以更清晰描述一个树的层次结构;

等等;

在[官方文档](http://etetoolkit.org/docs/latest/)中对其说明有详细描述;

以及[官方示例](http://etetoolkit.org/gallery/)中也展示了一些可以做出的结果示例;

在[这个项目](https://github.com/lozybean/newick_plot)里面, 示范了从创建newick到可视化展示的全部过程, 详细代码在里面可以找到;

示例结果如:

![示例结果1](http://ww3.sinaimg.cn/large/95202659gw1ezws3gpwxgj21ig0oxtfk.jpg)

![示例结果2](http://ww1.sinaimg.cn/large/95202659gw1ezws3hfl7bj21kd1av46o.jpg)
