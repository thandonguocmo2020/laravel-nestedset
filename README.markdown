[![Build Status](https://travis-ci.org/lazychaser/laravel-nestedset.svg?branch=master)](https://travis-ci.org/lazychaser/laravel-nestedset)
[![Total Downloads](https://poser.pugx.org/kalnoy/nestedset/downloads.svg)](https://packagist.org/packages/kalnoy/nestedset)
[![Latest Stable Version](https://poser.pugx.org/kalnoy/nestedset/v/stable.svg)](https://packagist.org/packages/kalnoy/nestedset)
[![Latest Unstable Version](https://poser.pugx.org/kalnoy/nestedset/v/unstable.svg)](https://packagist.org/packages/kalnoy/nestedset)
[![License](https://poser.pugx.org/kalnoy/nestedset/license.svg)](https://packagist.org/packages/kalnoy/nestedset)

This is a Laravel 4-5 package for working with trees in relational databases.

*   **Laravel 5.2** is supported since v4
*   **Laravel 5.1** is supported in v3
*   **Laravel 4** is supported in v2



- [Theory](#what-are-nested-sets)
- [Tài liệu ](#documentation)
    -   [Thêm mới giao điểm ](#inserting-nodes)
    -   [lấy nút giao điểm ](#retrieving-nodes)
    -   [xóa nút giao điểm](#deleting-nodes)
    -   [Tính nhất quán kiểm tra  & sửa chữa](#checking-consistency)
    -   [Phạm vi ](#scoping)
- [Yêu cầu](#requirements)
- [Cài đặt](#installation)

Cấu trúc cây lồng nhau là gì nested sets?
---------------------

Nested sets hoặc  [Nested Set Model](http://en.wikipedia.org/wiki/Nested_set_model) 
một cách để lưu trữ hiệu quả phân cấp dữ liệu trong một bảng quan hệ. 

> The nested set model is to number the nodes according to a tree traversal,
> which visits each node twice, assigning numbers in the order of visiting, and
> at both visits. This leaves two numbers for each node, which are stored as two
> attributes. Querying becomes inexpensive: hierarchy membership can be tested by
> comparing these numbers. Updating requires renumbering and is therefore expensive.

### Applications

NSM cho thấy hiệu suất khi cây được update ít. Nó được tạo ra làm nhanh chóng có được các giao điểm nút liên quan. Nó phù hợp xây dựng cho menu  " building multi-depth menu " hoặc categories của shop. 


Tài liệu 
-------------

Giả sử chúng ta có một mô hình model `Category`; và một `$node` biến là một  một thể hiện của class mô hình model
và các giao điểm mà chúng ta tương tác . Nó có thể là một mô hình mới hoặc một from database.  

### Mối quan hê trong cấu trúc cây 

Node giao điểm có mối quan hệ đẩy đủ chức năng với các function và và cấu trúc tự động nạp.

-   Node belongs to `parent` => nút giao điểm thuộc về một giao điểm cha
-   Node has many `children` => nút giao điểm có nhiều giao điểm con
-   Node has many `descendants` => giao điểm có nhiều giao điểm cháu

### Thêm một nút giao điểm

Di chuyển hay thêm mới một nút bao gồm truy vấn tới cơ sở dữ liệu. Thực thi được tự đông khi nào các nút giao điểm được lưu.  Nó là an toàn sử dụng thực thi toàn cầu. 

Nếu bạn làm việc với một số Models

Lưu ý quan trọng :  __Điều khiển cấu trúc cây bị trì hoãn__ cho đến khi bạn 
 `save` trong model (một số phương pháp ngầm `save` and trả về  boolean result).

Nếu mô hình được saved nó không có nghĩa các nút giao điểm được di chuyển. Nếu ứng dụng của bạn
phụ thuộc vào việc các nút thực sự đã được thay đổi vị trí sắp xếp của nó , sử dụng `hasMoved` method:

```php
if ($node->save()) {
    $moved = $node->hasMoved();
}
```

#### Tạo ra một nút giao điểm "nodes"

Khi bạn chỉ đơn giản là tạo ra một nút giao điểm mới, nó sẽ được thêm vào cuối cây của cấu trúc:

```php
Category::create($attributes); // Saved as root
```

```php
$node = new Category($attributes);
$node->save(); // Saved as root
```

Trong các trường hợp này các nút là một nút giao điểm  _root_  cấp cao nhât không có giao điểm cha.

#### Thay đổi một nút root giao điểm gốc từ nút hiện có 

```php
// #1 Implicit save
$node->saveAsRoot();

// #2 Explicit save
$node->makeRoot()->save();
```

Các nút sẽ tạo ra cấu trúc cây mới được thêm vào cuối cây hiện tại cùng cấp các nút root đã có.

#### Nối thêm vào  and thêm vào trước đến các giao điểm đã xác định  parent

Nếu bạn muốn làm cho nút hiện tại là giao điểm nút con của một nút khác. Bạn có thể cho nó là cuối cùng hoặc đầu tiên. "last or first child".

*Áp dụng trong ví dụ này, `$parent` là một nút hiện có.*

`$parent = find($id) return object $nodes parent`

Có vài cách để nối thêm một giao điểm vào cuối :

```php
// #1 Sử dụng một định nghĩa insert
$node->appendToNode($parent)->save();

ví dụ :
$parent = Category::find(10);
$attributes = [
    'name' => 'Màu Sắc',

    'children' => [
        [
            'name' => 'Đỏ',

            'children' => [
                [ 'name' => 'Đỏ Đậm' ],
            ],
         ],
      ],
   ];

$node = Category::create($attributes);
$node->appendToNode($parent)->save();

// #2  Sử dụng giao điểm là object parent thêm mới một giao điểm con
$parent->appendNode($node);

ví dụ :
thay thế function $node->appendToNode($parent)->save() bằng $parent->appendNode($node);


// #3 Sử dụng một mối quan hệ giữa nút parent và các con sau đó tạo mới một giao điểm vào cuối cùng
$parent->children()->create($attributes);

// #5 Sử dụng một giao điểm parent mối quan hệ 
$node->parent()->associate($parent)->save();

// #6 Using the parent attribute
$node->parent_id = $parent->id;
$node->save();

// #7 Using static method
Category::create($attributes, $parent);
```

Chỉ có một vài cách để thêm vào trước đầu tiên của các nút cùng cấp thay vì nối thêm vào ở cuối cùng:

```php
// #1
$node->prependToNode($parent)->save();

// #2
$parent->prependNode($node);
```

#### Thêm mới  phía trước  or phía sau một giao điểm nút chỉ định

Bạn có thể làm cho các giao điểm nút `$node` là một giao điểm cùng cấp hàng xóm `$neighbor` của nút giao điểm hiện có bằng các phương pháp:

*`$neighbor` phải tồn tại, nút giao điểm ``$node`` phải là mới. nếu nút ``$node`` tồn tại , 
nó sẽ được di chuyển đến vị trí mới  và parent của nó sẽ được thay đổi nếu nó yêu cầu.*

```php

$neighbor = Category::find(13);
$attributes = [
	    'name' => 'Thit heo Tây',

	    'children' => [
	        [
	            'name' => 'Thị Heo Tây Nuôi Nhà',

	         ],
	      ],
	   ];
	   // tạo nút mới 
$node = Category::create($attributes);
// gọi function để di chuyển nút là hàng xóm hoặc thay đổi vị trí parent nếu được yêu cầu

# rõ ràng  save
$node->afterNode($neighbor)->save();
$node->beforeNode($neighbor)->save();

# ngầm định  save
$node->insertAfterNode($neighbor);
$node->insertBeforeNode($neighbor);
```

#### Xây dựng một cây cấu trúc lồng nhau từ mảng

khi nào sử dụng các method static `create` trong các giao điểm, nó sẽ kiểm tra xem các attributes chứa
`children` key. Nếu nó có nó tạo ra nhiều hơn các nút đệ quy.

```php
$node = Category::create(
			['name' => 'Food',

			    'children' => [
			        [
			            'name' => 'Fruid',

			            'children' => [
			                [ 
			                	'name' => 'Red',
			                  	'children' => [
			                  		['name'=>'Cherry'],
			                  		['name'=>'Torry'],
			                  	]
			                 ],
			              
			            ],
			        ],
			        [
			            'name' => 'Meat',

			            'children' => 
			           		 [
				                [ 'name' => 'Beef',
				                		'children' =>
				                		[
						                	 ['name'=>'Cherry'],
						                	 ['name'=>'Torry'],
					                	],
				                ],
				                [ 'name' => 'Pork',
				                		'children' =>
				                			[
							                	 ['name'=>'Niten'],
							                	 ['name'=>'Forry']
				                			 ]

				                 ],
			           		 ],
			        ],

			    ],
		 
			]
		);


```

`$node->children` bây giờ chứa một danh sách các nút giao điểm con child đươc tạo ra.

#### Xây dựng lại cây cấu trúc  từ 1 mảng

Bạn có thể dễ dàng xây dựng lại một cây. Điều này rất hữu ích cho hàng loạt thay đổi cơ cấu 

```php
Category::rebuildTree($data, $delete);
```

`$data` là một mảng của các nút :

```php
$data = [
    [ 'id' => 1, 'name' => 'foo', 'children' => [ ... ] ],
    [ 'name' => 'bar' ],
];
```

Có một id được chỉ định cho nút với tên của `foo` có nghĩa là hiện tại
nút sẽ được  filled and saved. nếu nút không tồn tại một lỗi được trả về `ModelNotFoundException`. Cũng như thế,
Nếu nút có  `children` quy định đó cũng là một mảng của các nút giao điểm;

họ sẽ được xử lý trong theo cách tương tự và  Lưu giống như children của nút giao điểm  `foo`.

Nút giao điểm `bar` không có khóa chính primary key , 
do đó, nó sẽ được tạo ra.

`$delete` cho thấy cho dù để xóa các nút mà đã tồn tại nhưng không có mặt trong `$data`.
Theo mặc định, nút không bị xóa.

### Tìm lấy ra được  nút giao điểm "nodes"

*Trong một số trường hợp, chúng tôi sẽ sử dụng một biến `$id` mà là id của nút mục tiêu.*

#### Lấy ra nút tổ tiên

Một chuỗi từ tổ tiến đến parents đến nút hiện tại sẽ được trả về phù hợp để thực hiện hiển thị 
 breadcrumbs category hiện tại.

```php
// #1 Sử dụng truy cập
$result = $node->getAncestors();

// #2 Sử dụng một query
$result = $node->ancestors()->get();

// #3 Sử dụng truy cập từ 1 khóa chính
$result = Category::ancestorsOf($id);
```

#### Lấy ra nút giao điểm là con cháu

Con cháu là tất cả các  nút giao điểm trong một nhánh của cây có _lft và _rgt là một khoảng trống vị trí.  Nghĩa là lấy ra các con của nút hoặc cháu hoặc tất cả ... v.v tùy vào khoảng trống cụ thể giữa column của  nút giao điểm _lft và _rgt trong table

```php
// #1 Using mối liên hệ
$result = $node->descendants;

// #2 Using một query
$result = $node->descendants()->get();

// #3 lấy ra con cháu  descendants sử dụng khóa chính
$result = Category::descendantsOf($id);
```

Lấy ra con cháu hay hậu duệ của các giao điểm có id là một trong các số trong ảng id list nếu các id đó có cột _lft và _rgt có khoảng trống là hậu duệ sẽ được lấy ra :

```php
$nodes = Category::with('descendants')->whereIn('id', $idList)->get();
```

#### Siblings

Siblings are nodes that have same parent.

```php
$result = $node->getSiblings();

$result = $node->siblings()->get();
```

To get only next siblings:

```php
// Get a sibling that is immediately after the node
$result = $node->getNextSibling();

// Get all siblings that are after the node 
$result = $node->getNextSiblings();

// Get all siblings using a query
$result = $node->nextSiblings()->get();
```

To get previous siblings:

```php
// Get a sibling that is immediately before the node
$result = $node->getPrevSibling();

// Get all siblings that are before the node 
$result = $node->getPrevSiblings();

// Get all siblings using a query
$result = $node->prevSiblings()->get();
```

#### Getting related models from other table

Imagine that each category `has many` goods. I.e. `HasMany` relationship is established.
How can you get all goods of `$category` and every its descendant? Easy!

```php
// Get ids of descendants
$categories = $category->descendants()->lists('id');

// Include the id of category itself
$categories[] = $category->getKey();

// Get goods
$goods = Goods::whereIn('category_id', $categories)->get();
```

#### Including node depth

If you need to know at which level the node is:

```php
$result = Category::withDepth()->find($id);

$depth = $result->depth;
```

Root node will be at level 0. Children of root nodes will have a level of 1, etc.

To get nodes of specified level, you can apply `having` constraint:

```php
$result = Category::withDepth()->having('depth', '=', 1)->get();
```

#### Default order

Each node has it's own unique `_lft` value that determines its position in the tree. If
you want node to be ordered by this value, you can use `defaultOrder` method on
the query builder:

```php
// All nodes will now be ordered by lft value
$result = Category::defaultOrder()->get();
```

You can get nodes in reversed order:

```php
$result = Category::reversed()->get();
```

##### Shifting a node

To shift node up or down inside parent to affect default order:

```php
$bool = $node->down();
$bool = $node->up();

// Shift node by 3 siblings
$bool = $node->down(3);
```

The result of the operation is boolean value of whether the node has changed its
position.

#### Constraints

Various constraints that can be applied to the query builder:

-   __whereIsRoot()__ to get only root nodes;
-   __whereIsAfter($id)__ to get every node (not just siblings) that are after a node
    with specified id;
-   __whereIsBefore($id)__ to get every node that is before a node with specified id.

Descendants constraints:

```php
$result = Category::whereDescendantOf($node)->get();
$result = Category::whereNotDescendantOf($node)->get();
$result = Category::orWhereDescendantOf($node)->get();
$result = Category::orWhereNotDescendantOf($node)->get();
```

Ancestor constraints:

```php
$result = Category::whereAncestorOf($node)->get();
```

`$node` can be either a primary key of the model or model instance.

#### Building a tree

After getting a set of nodes, you can convert it to tree. For example:

```php
$tree = Category::get()->toTree();
```

This will fill `parent` and `children` relationships on every node in the set and
you can render a tree using recursive algorithm:

```php
$nodes = Category::get()->toTree();

$traverse = function ($categories, $prefix = '-') use (&$traverse) {
    foreach ($categories as $category) {
        echo PHP_EOL.$prefix.' '.$category->name;

        $traverse($category->children, $prefix.'-');
    }
};

$traverse($nodes);
```

This will output something like this:

```
- Root
-- Child 1
--- Sub child 1
-- Child 2
- Another root
```

##### Building flat tree

Also, you can build a flat tree: a list of nodes where child nodes are immediately
after parent node. This is helpful when you get nodes with custom order
(i.e. alphabetically) and don't want to use recursion to iterate over your nodes.

```php
$nodes = Category::get()->toFlatTree();
```

##### Getting a subtree

Sometimes you don't need whole tree to be loaded and just some subtree of specific node.
It is show in following example:

```php
$root = Category::find($rootId);
$tree = $root->descendants->toTree($root);
```

Now `$tree` contains children of `$root` node.

If you don't need `$root` node itself, do following instead:

```php
$tree = Category::descendantsOf($rootId)->toTree($rootId);
```

### Deleting nodes

To delete a node:

```php
$node->delete();
```

**IMPORTANT!** Any descendant that node has will also be deleted!

**IMPORTANT!** Nodes are required to be deleted as models, **don't** try do delete them using a query like so:

```php
Category::where('id', '=', $id)->delete();
```

This will break the tree!

`SoftDeletes` trait is supported, also on model level.

### Helper methods

To check if node is a descendant of other node:

```php
$bool = $node->isDescendantOf($parent);
```

To check whether the node is a root:

```php
$bool = $node->isRoot();
```

Other checks:

*   `$node->isChildOf($other);`
*   `$node->isAncestorOf($other);`
*   `$node->isSiblingOf($other);`

### Checking consistency

You can check whether a tree is broken (i.e. has some structural errors):

```php
$bool = Category::isBroken();
```

It is possible to get error statistics:

```php
$data = Category::countErrors();
```

It will return an array with following keys:

-   `oddness` -- the number of nodes that have wrong set of `lft` and `rgt` values
-   `duplicates` -- the number of nodes that have same `lft` or `rgt` values
-   `wrong_parent` -- the number of nodes that have invalid `parent_id` value that
    doesn't correspond to `lft` and `rgt` values
-   `missing_parent` -- the number of nodes that have `parent_id` pointing to
    node that doesn't exists
    
#### Fixing tree

Since v3.1 tree can now be fixed. Using inheritance info from `parent_id` column, 
proper `_lft` and `_rgt` values are set for every node.

```php
Node::fixTree();
```

### Scoping

Imagine you have `Menu` model and `MenuItems`. There is a one-to-many relationship
set up between these models. `MenuItem` has `menu_id` attribute for joining models
together. `MenuItem` incorporates nested sets. It is obvious that you would want to 
process each tree separately based on `menu_id` attribute. In order to do so, you
need to specify this attribute as scope attribute:

```php
protected function getScopeAttributes()
{
    return [ 'menu_id' ];
}
```

But now in order to execute some custom query, you need to provide attributes
that are used for scoping:

```php
MenuItem::scoped([ 'menu_id' => 5 ])->withDepth()->get(); // OK
MenuItem::descendantsOf($id)->get(); // WRONG: returns nodes from other scope
MenuItem::scoped([ 'menu_id' => 5 ])->fixTree();
```

When requesting nodes using model instance, scopes applied automatically based 
on the attributes of that model. See examples:

```php
$node = MenuItem::findOrFail($id);

$node->siblings()->withDepth()->get(); // OK
```

To get scoped query builder using instance:

```php
$node->newScopedQuery();
```

Note, that scoping is not required when retrieving model by primary key 
(since the key is unique):

```php
$node = MenuItem::findOrFail($id); // OK
$node = MenuItem::scoped([ 'menu_id' => 5 ])->findOrFail(); // OK, but redundant
```

Requirements
------------

- PHP >= 5.4
- Laravel >= 4.1

It is highly suggested to use database that supports transactions (like MySql's InnoDb) 
to secure a tree from possible corruption.

Installation
------------

To install the package, in terminal:

```
composer require kalnoy/nestedset
```

### Setting up from scratch

#### The schema

You can use a method to add needed columns with default names:

```php
Schema::create('table', function (Blueprint $table) {
    ...
    NestedSet::columns($table);
});
```

To drop columns:

```php
Schema::table('table', function (Blueprint $table) {
    NestedSet::dropColumns($table);
});
```

#### The model

Your model should use `Kalnoy\Nestedset\NodeTrait` trait to enable nested sets:

```php
use Kalnoy\Nestedset\NodeTrait;

class Foo extends Model {
    use NodeTrait;
}
```

### Migrating existing data

#### Migrating from other nested set extension

If your previous extension used different set of columns, you just need to override
following methods on your model class:

```php
public function getLftName()
{
    return 'left';
}

public function getRgtName()
{
    return 'right';
}

public function getParentIdName()
{
    return 'parent';
}

// Specify parent id attribute mutator
public function setParentAttribute($value)
{
    $this->setParentIdAttribute($value);
}
```

#### Migrating from basic parentage info

If your tree contains `parent_id` info, you need to add two columns to your schema:

```php
$table->unsignedInteger('_lft');
$table->unsignedInteger('_rgt');
```

After [setting up your model](#the-model) you only need to fix the tree to fill 
`_lft` and `_rgt` columns:

```php
MyModel::fixTree();
```

License
=======

Copyright (c) 2016 Alexander Kalnoy

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
