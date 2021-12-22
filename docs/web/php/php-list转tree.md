### 引用链接法

```php
    /**
     * 数据列表转换成树
     *
     * @param  array   $dataArr   数据列表
     * @param  integer $rootId    根节点ID
     * @param  string  $pkName    主键
     * @param  string  $pIdName   父节点名称
     * @param  string  $childName 子节点名称
     *
     * @return array  转换后的树
     */
    function ListToTree($dataArr, $rootId = 0, $pkName = 'cc_id', $pIdName = 'parent_id', $childName = 'children')
    {
        $tree = [];
        if (is_array($dataArr))
        {
            //1.0 创建基于主键的数组引用
            $referList  = [];
            foreach ($dataArr as $key => & $sorData)
            {
                $referList[$sorData[$pkName]] =& $dataArr[$key];
            }

            //2.0 list 转换为 tree
            foreach ($dataArr as $key => $data)
            {
                $pId = $data[$pIdName];
                if ($rootId == $pId) //一级
                {
                    $tree[] =& $dataArr[$key];
                }
                else //多级
                {
                    if (isset($referList[$pId]))
                    {
                        $pNode               =& $referList[$pId];
                        $pNode[$childName][] =& $dataArr[$key];
                    }
                }
            }
        }

        return $tree;
    }
```

> 主要通过引用指向同一地址的技术手段解决问题。`$referList` 的每个元素指向 `$dataArr` 数组的元素地址。遍历时，通过将 `$rootId` 下的元素指向 `$tree[]` ，非 `$rootId` 下的元素指向父级的 `$referList[$pId]` 的 `$childName` 元素，遍历完毕后，链接形成整个树结构。



转换前：

```php
        {
            "cc_id": 1,
            "cc_name": "语文",
            "cc_path": "0/1/",
            "parent_id": 0,
            "children": []
        },
        {
            "cc_id": 2,
            "cc_name": "数学",
            "cc_path": "0/1/2/",
            "parent_id": 1,
            "children": []
        },
        {
            "cc_id": 3,
            "cc_name": "英语",
            "cc_path": "0/3/",
            "parent_id": 0,
            "children": []
        },
        {
            "cc_id": 9,
            "cc_name": "一年级",
            "cc_path": "0/1/9/",
            "parent_id": 1,
            "children": []
        },
        {
            "cc_id": 10,
            "cc_name": "二年级",
            "cc_path": "0/1/10/",
            "parent_id": 1,
            "children": []
        },
        {
            "cc_id": 11,
            "cc_name": "三年级",
            "cc_path": "0/1/11/",
            "parent_id": 1,
            "children": []
        }
```

转换后：

```php
        {
            "cc_id": 1,
            "cc_name": "语文",
            "cc_path": "0/1/",
            "parent_id": 0,
            "children": [
                {
                    "cc_id": 2,
                    "cc_name": "数学",
                    "cc_path": "0/1/2/",
                    "parent_id": 1,
                    "children": []
                },
                {
                    "cc_id": 9,
                    "cc_name": "一年级",
                    "cc_path": "0/1/9/",
                    "parent_id": 1,
                    "children": []
                },
                {
                    "cc_id": 10,
                    "cc_name": "二年级",
                    "cc_path": "0/1/10/",
                    "parent_id": 1,
                    "children": []
                },
                {
                    "cc_id": 11,
                    "cc_name": "三年级",
                    "cc_path": "0/1/11/",
                    "parent_id": 1,
                    "children": []
                }
            ]
        },
        {
            "cc_id": 3,
            "cc_name": "英语",
            "cc_path": "0/3/",
            "parent_id": 0,
            "children": []
        }
```



### 递归法

```php
    /**
     * 采用递归将数据列表转换成树
     *
     * @param  array   $dataArr   数据列表
     * @param  integer $rootId    根节点ID
     * @param  string  $pkName    主键
     * @param  string  $pIdName   父节点名称
     * @param  string  $childName 子节点名称
     *
     * @return array  转换后的树
     */
    function ListToTreeRecursive($dataArr, $rootId = 0, $pkName = 'cc_id', $pIdName = 'parent_id', $childName = 'children')
    {
        $arr = [];
        foreach ($dataArr as $sorData)
        {
            if ($sorData[$pIdName] == $rootId)
            {
                $sorData[$childName] = ListToTreeRecursive($dataArr, $sorData[$pkName]);
                $arr[]               = $sorData;
            }
        }

        return $arr;
    }

```