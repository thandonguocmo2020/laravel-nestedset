[![Build Status](https://travis-ci.org/lazychaser/laravel-nestedset.svg?branch=master)](https://travis-ci.org/lazychaser/laravel-nestedset)
[![Total Downloads](https://poser.pugx.org/kalnoy/nestedset/downloads.svg)](https://packagist.org/packages/kalnoy/nestedset)
[![Latest Stable Version](https://poser.pugx.org/kalnoy/nestedset/v/stable.svg)](https://packagist.org/packages/kalnoy/nestedset)
[![Latest Unstable Version](https://poser.pugx.org/kalnoy/nestedset/v/unstable.svg)](https://packagist.org/packages/kalnoy/nestedset)
[![License](https://poser.pugx.org/kalnoy/nestedset/license.svg)](https://packagist.org/packages/kalnoy/nestedset)


Đây là một gói thư viện làm việc với Laravel 4-5 để làm việc với cây trong cơ sở dữ liệu quan hệ

*   **Laravel 5.2** sử dụng  v4
*   **Laravel 5.1** sử dụng v3
*   **Laravel 4** sử dụng v2



- [Lý thuyết](#lý-thuyết)
- [Yêu cầu](#yÊu-cẦu)
- [Cài đặt](#cÀi-ĐẶt)
- [Tài liệu ](#tài-liệu)
    -   [TẠO NÚT](#tẠo-nÚt) 
    -   [THÊM NÚT ](#thÊm-nÚt)
    -   [LẤY NÚT ](#lẤy-ra-nÚt-giao-ĐiỂm)
    -   [XÓA NÚT](#xÓa-nÚt-giao-ĐiỂm-trong-mÔ-hÌnh)
    -   [TÌNH NHẤT QUÁN KIỂM TRA   & SỬA CHỮA](#kiểm-tra--tính-nhất-quán-của-cấu-trúc)
    -   [PHẠM VI](#phẠm-vi)

### Lý thuyết
### Nested Sets MÔ HÌNH LỒNG NHAU HAY CẤU TRÚC PHÂN CẤP DẠNG CÂY LÀ GÌ?
---------------------

Nested sets hoặc  [MÔ HÌNH PHÂN CẤP](http://blog.siliconstraits.vn/mo-hinh-tap-hop-long-nhau-trong-co-so-du-lieu-phan-cap/) 
một cách để lưu trữ hiệu quả phân cấp dữ liệu trong một bảng quan hệ. 

![alt tag](https://scontent-hkg3-1.xx.fbcdn.net/v/t1.0-9/13686553_644095012411687_8366930733854424679_n.jpg?oh=dda5e2675817798aba2afc6ce43e615c&oe=585BA1FA)


### Ứng dụng

NSM cho thấy hiệu suất khi cây được update ít. Nó được tạo ra làm nhanh chóng có được các giao điểm nút liên quan. Nó phù hợp xây dựng cho menu  " building multi-depth menu " hoặc categories của shop. 


Tài liệu
-------------

Giả sử chúng ta có một mô hình model `Category`; và một `$node` biến là một  một thể hiện của class mô hình model
và các giao điểm mà chúng ta tương tác . Nó có thể là một mô hình mới hoặc một bảng từ database.  

### Mối quan hê trong cấu trúc cây 

Node giao điểm có mối quan hệ đẩy đủ chức năng với các function và và cấu trúc tự động nạp.

-   Node belongs to `parent` => nút giao điểm thuộc về một giao điểm cha
-   Node has many `children` => nút giao điểm có nhiều giao điểm con
-   Node has many `descendants` => giao điểm có nhiều giao điểm cháu


####  TẠO NÚT 

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


#### THÊM NÚT

Giao dịch tự động khi các nút được save() thêm mới.

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

#### NỐI VÀO CUỐI HOẶC THÊM VÀO TRƯỚC NHÁNH HIỆN CÓ TRONG CẤU TRÚC PHÂN CẤP

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

####  THÊM MỚI PHÍA TRƯỚC HOẶC SAU MỘT NÚT CHỈ ĐỊNH

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

#### XÂY DỰNG CẤU TRÚC PHÂN CẤP LỒNG NHAU TỪ MẢNG

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

#### XÂY DỰNG LẠI MỘT CÂY PHÂN CẤP TỪ MẢNG

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

### LẤY RA NÚT GIAO ĐIỂM

*Trong một số trường hợp, chúng tôi sẽ sử dụng một biến `$id` mà là id của nút mục tiêu.*

#### LẤY RA NÚT TỔ TIÊN ROOT

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

#### LẤY RA CÁC NHÁNH CON TRONG PHÂN CẤP

Con cháu là tất cả các  nút giao điểm trong một nhánh của cây có _lft và _rgt là một khoảng trống vị trí.  Nghĩa là lấy ra các con của nút hoặc cháu hoặc tất cả ... v.v tùy vào khoảng trống cụ thể giữa column của  nút giao điểm _lft và _rgt trong table

```php
// #1 Using mối liên hệ
$result = $node->descendants;

// #2 Using một query
$result = $node->descendants()->get();

// #3 lấy ra con cháu  descendants sử dụng khóa chính
$result = Category::descendantsOf($id);
```

Lấy ra con cháu hay hậu duệ của các giao điểm có id là một trong các số trong mảng id list nếu các id đó có cột _lft và _rgt có khoảng trống là hậu duệ sẽ được lấy ra :

```php
$nodes = Category::with('descendants')->whereIn('id', $idList)->get();
```

#### LẤY RA CÁC NÚT CÙNG CẤP TRONG 1 NHÁNH

Anh chị em là các giao điểm có cùng Parent.

ví dụ parent có _lft là 13 và _rgt là 18 vậy các khoảng trong 13 và 18 có 2 nút cùng cấp mang giá trị 2 column là 14-15 và 16-17 sẽ ;à ạ e cùng parent giao điểm cha nút.

```php
$result = $node->getSiblings();

$result = $node->siblings()->get();
```

Để có được anh em phía sau cùng cấp của giao điểm hiện tại :

ví dụ _lft = 15 vậy sau 15 là 16 ...... với điều kiện cùng thuộc parent giao điểm. 

```php
$node = Model::find(15);
// lấy được một anh em được thêm vào sau node hiện có return 16
$result = $node->getNextSibling();


// lấy tất cả ae được thêm vào sau 16-> ... v.v.. <= nhỏ hơn parent _rgt
$result = $node->getNextSiblings();

// lấy tất cả thêm vào sau sử dụng query
$result = $node->nextSiblings()->get();
```

Để có được anh chị em giao điểm nút thêm vào trước :

```php
// Lấy một người anh em của nút được thêm và trước giao điểm hiện tại

$result = $node->getPrevSibling();

// lấy tất cả các nút được thêm vào trước nút hiện tại 

$result = $node->getPrevSiblings();

// Lấy tất cả  nút cuối cùng được thêm vào trước object hiện tại  bằng query xem right để biết giao điểm trước cùng cấp
$result = $node->prevSiblings()->get();
```

#### BẮT DỮ LIỆU MODEL TỪ BẢNG KHÁC

Hãy tưởng tượng một category `has many` có nhiều goods. I.e. `HasMany` mối quan hệ được thiết lập.
Làm thế nào bạn có thể nhận được tất cả  get all goods của `$category` và mỗi hậu duệ của nó ?

```php
// Nhận id của con cháu 
$categories = $category->descendants()->lists('id');

// Bao gồm các id của category chính no
$categories[] = $category->getKey();

// Nhận hàng goods
$goods = Goods::whereIn('category_id', $categories)->get();
```

#### LẤY LEVEL CHIỀU SÂU  CỦA NÚT HIỆN TẠI TRONG MÔ HÌNH PHÂN CẤP

Nếu bạn cần biết level các giao điểm nút với nút id là :

```php
$result = Category::withDepth()->find($id);

$depth = $result->depth;
```

Root  nút sẽ ở  level 0. Children của nút root sẽ có level 1, 

Để có được các name giao điểm nút trong cấp level cụ thể, bạn có thể áp dụng  `having` hạn chế :

```php
$result = Category::withDepth()->having('depth', '=', 1)->get();
```

#### SẮP XẾP MẶC ĐỊNH CỦA MÔ HÌNH PHÂN CẤP

Mỗi nút giao điểm có giá trị cột `_lft` là duy nhất để xác định vị trí của nó trong cây. Nếu bạn muốn nút giao điểm  được sắp xếp theo các giá trị này, bạn cần sử dụng `defaultOrder` phương pháp trong câu lệnh truy vấn query builder :

```php
// Muốn lấy tất cả các nút giao điểm theo giá trị sắp xếp cột lft 
$result = Category::defaultOrder()->get();
```

Bạn có thể nhận được các nút theo thứ tự đảo ngược lại :

```php
$result = Category::reversed()->get();
```

##### DI CHUYỂN VỊ TRÍ CỦA NÚT TRONG MÔ HÌNH PHÂN CẤP THAY ĐỔI MẶC ĐỊNH

Để thay đổi nút giao điểm lên hoặc xuống  phía trong nhánh  parent làm thay đổi thứ tự mặc định sử dụng :

```php
$bool = $node->down();
$bool = $node->up();

// Di chuyển giao điểm xuống 3 đơn vị sắp xếp trong các giao điểm cùng cấp anh em.
$bool = $node->down(3);
```

Kết quả của hoạt động này là giá trị boolean của khi được thay đổi vị trí

#### TRUY VẤN QUERY ĐI KÈM HẠN CHẾ LẤY RA.

những hạn chế khác nhau có thể được áp dụng cho truy vấn query builder:

-   __whereIsRoot()__ chỉ lấy duy nhất nút giao điểm gốc Root;
-   __whereIsAfter($id)__ Để có được tất cả các nút phía sau (Không chỉ là các nút đồng cấp anh em)  với 1 id giao điểm chỉ định
-   __whereIsBefore($id)__ Để có được tất cả các nút phía trước của nút giao điểm với id chỉ định.

Hạn chế  đối với con cháu :

```php
// Lấy ra tất cả con cháu phía sau của nút
$result = Category::whereDescendantOf($node)->get();
// lấy ra tất cả không phải con cháu của nút 
$result = Category::whereNotDescendantOf($node)->get();
// Hoặc sử dụng 2 phương pháp tương ứng
$result = Category::orWhereDescendantOf($node)->get();
$result = Category::orWhereNotDescendantOf($node)->get();
```

Tổ tiên cây cấu trúc hạn chế constraints:

```php
// lấy ra tất cả các nút tổ tiên của nút 
$result = Category::whereAncestorOf($node)->get();
```

`$node` có thể là một a primary key khóa chính sử dụng trong bảng làm việc thông qua Model

#### XÂY DỰNG MÔ HÌNH CÂY PHÂN CẤP TỪ DỮ LIỆU LẤY RA

Sau khi nhận được một bộ sưu tập các giao điểm bạn có thể chuyển nó sang  cấu trúc dạng cây  ví dụ 

```php
$tree = Category::get()->toTree();
```
Điều này sẽ điền các giao điểm cha `parent` và con  `children` có mối quan hệ trong các giao điểm được thiết lập và bạn cần render hiển thị cây bằng thuật toán đệ qui phương pháp :

```php
$nodes = Category::get()->toTree();

$traverse = function ($categories, $prefix = '-') use (&$traverse) {
    foreach ($categories as $category) {
        echo PHP_EOL.$prefix.' '.$category->name;

        $traverse($category->children, $prefix.'-');
    }
};

$traverse($nodes);


hoặc sắp xếp  ul

        $categories =  Category::get()->toTree();
        
        $traverse = function ($categories, $prefix = '<li>', $suffix = '</li>') use (&$traverse) {
        foreach ($categories as $category) {
            echo $prefix.$category->name.$suffix;

            $hasChildren = (count($category->children) > 0);

            if($hasChildren) {
                echo('<ul>');
            }

            $traverse($category->children);

            if($hasChildren) {
                echo('</ul>');
               }
           }
       };

        $traverse($categories);

 





```

Kết quả khi output :

```
- Root
-- Child 1
--- Sub child 1
-- Child 2
- Another root
```

##### XÂY DỰNG MỘT CÂY DỮ LIỆU PHẲNG THAY VÌ PHÂN CẤP

Ngoài ra bạn có thể xây dựng một cấu trúc cây phẳng thay vì nặp các nút theo đệ qui.
Điều này thực sự có ích khi bạn muốn sắp xếp cây cấu trúc theo thứ tự abc....mà được một danh sách các nút mà nút con là ngay lập tức
sau khi nút cha.

```php
$nodes = Category::get()->toFlatTree();
```

##### LẤY MỘT MÔ HÌNH PHÂN CẤP CỤ THỂ 

Trong bảng dữ liệu của bạn có thể có nhiều cấu trúc cây, Đôi khi bạn không cần tải toàn bộ chúng bạn chỉ 
cần một cây của một giao điểm cụ thể

Đây là một ví dụ để hiển thị.

```php
$root = Category::find($rootId);
$tree = $root->descendants->toTree($root);
```

Điều này `$tree` sẽ trả về cấu trúc cây là các giao điểm con cháu của giao điểm `$root`.

Nếu bạn không cần cấu trúc cây của nút  `$root` đó  Bạn có thể phủ định nó để lấy ra các cây cấu trúc khác mà không phải là nó:

```php
$tree = Category::descendantsOf($rootId)->toTree($rootId);
```

### XÓA NÚT GIAO ĐIỂM TRONG MÔ HÌNH

Để xóa một giao điểm nút

```php
$node->delete();
```

**Lưu Ý!** bất kỳ các giao điểm con cháu cùng bị xóa

**IMPORTANT!** Các nút yêu cầu xóa phải được xóa như các model orm , **don't** không thực hiện xóa các giao điểm bằng các truy vấn tương tự như sau :

```php
Category::where('id', '=', $id)->delete();
```


Điều này sẽ phá vỡ các cấu trúc của cây!

`SoftDeletes` đặc điểm được hỗ trợ

### PHƯƠNG PHÁP HỖ TRỢ

Để kiểm tra nếu giao điểm hiện tại là con cháu của một giao điểm khác

```php
$bool = $node->isDescendantOf($parent);
```

Để kiểm tra xem các giao điểm hiện tại có phải là nút Root :

```php
$bool = $node->isRoot();
```

Kiểm tra khác :


// nếu đúng là con của nút khác

*   `$node->isChildOf($other);`
*   // nếu đúng là tổ tiên của nút khác
*   `$node->isAncestorOf($other);`
*   // nếu đúng là anh em của nút khác
*   `$node->isSiblingOf($other);`

### Kiểm Tra  Tính nhất quán của cấu trúc

Bạn có thể kiểm tra xem một cây bị phá vỡ cấu trúc của nó mà sinh lỗi cấu trúc

```php
$bool = Category::isBroken();
```

Nó có thể cho phép hiển thị các lỗi 

```php
$data = Category::countErrors();
```

nó sẽ return ra một mảng các key sau kèm theo thông báo lỗi :

-   `oddness` -- số giao điểm đã bị sai bộ thứ tự sắp xếp `lft` và `rgt` giá trị
-   `duplicates` -- các số của giao điểm này đã có `lft` hoặc `rgt` giá trị trong cột
-   `wrong_parent` -- các số của  thứ tự của giao điểm không hợp lệ trong `parent_id` giá trị 
   không tương ứng với  `lft` và `rgt` giá trị cột
-   `missing_parent` -- number đã có  `parent_id` chỉ đến một giao điểm không tồn tại
    
#### Sửa lỗi cấu trúc cây 

Kể từ khi cây v3.1 có thể được cố định. Sử dụng thông tin kế thừa từ `parent_id` cột, 
với đặc tính riêng  `_lft` và  `_rgt` giá trị được thiết lập là một cặp của giao điểm .

```php
Node::fixTree();
```

### PHẠM VI

Hãy thử nghĩ bạn có một bảng  `Menu` với model và một  `MenuItems` bảng với model. tức là mỗi quan hệ một nhiều one-to-many 
giữa 2 mô hình này. `MenuItem` có `menu_id` thuộc tình  sử dụng để nối 2 bảng qua model. `MenuItem` sử dụng bộ cấu trúc cây lồng nhau. Rõ ràng rằng bạn sẽ muốn 
xử lý từng cấu trúc dựa trên  `menu_id` thuộc tính. Để làm như vậy bạn cần xác định thuộc tính này như một phạm vi truy vấn sử dụng
ở mọi nơi.

```php
protected function getScopeAttributes()
{
    return [ 'menu_id' ];
}
```

Ngay bây giờ để thực hiện một tùy chỉnh query  bạn cần cung cấp thuộc tính attributes được sử dụng của menu_id scoping:

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

###YÊU CẦU
------------

- PHP >= 5.4
- Laravel >= 4.1

It is highly suggested to use database that supports transactions (like MySql's InnoDb) 
to secure a tree from possible corruption.

### CÀI ĐẶT 

Installation
------------

To install the package, in terminal:

```
composer require kalnoy/nestedset
```

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
