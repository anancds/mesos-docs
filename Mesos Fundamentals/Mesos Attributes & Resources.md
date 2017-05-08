### Mesos Attributes & Resources

Mesos有两种方式来解释集群里的agents。一种是agents是由master管理的，另外一种通过集群沟通Frameworks。

###

Mesos中的属性和资源支持的数据类型有：scalar(标量)、ranges(范围)、sets(集合)、text(文本)。
下面就是这些类型的定义。

    scalar : floatValue

    floatValue : ( intValue ( "." intValue )? ) | ...

    intValue : [0-9]+

    range : "[" rangeValue ( "," rangeValue )* "]"

    rangeValue : scalar "-" scalar

    set : "{" text ( "," text )* "}"

    text : [a-zA-Z0-9_/.-]

### Attributes

属性是一些键值对(值是可选的)，这些键值对是Mesos向Frameworks发出资源邀约的时候传递的。一个属性值支持三种不同的类型：scalar、range、text。

    attributes : attribute ( ";" attribute )*

    attribute : text ":" ( scalar | range | text )

### Resources

Mesos可以管理三种不同的resources类型：scalars、ranges、sets。当mesos agent上报资源时，这些类型就用来表示不同类型的资源。比如标量资源可以用来表示一个agent上的内存总量。标量资源是用一些浮点数表示的，并且允许指定小数部分(比如1.5CPUs)。Mesos仅支持三位小数的精度(比如预留“1.5123 CPUs”相当于就是预留“1.512 CPUs”)。对于GPU，Mesos仅支持整数。

Resources可以用json来表示，也可以用冒号分割的键值对表示。如果看了下面的例子，对json格式还有问题的话，那么就查看**Resource**的protobuf定义，在include/mesos/mesos.proto目录。

json格式如下：

    [
      {
        "name": "<resource_name>",
        "type": "SCALAR",
        "scalar": {
          "value": <resource_value>
        }
      },
      {
        "name": "<resource_name>",
        "type": "RANGES",
        "ranges": {
          "range": [
            {
              "begin": <range_beginning>,
              "end": <range_ending>
            },
            ...
          ]
        }
      },
      {
        "name": "<resource_name>",
        "type": "SET",
        "set": {
          "item": [
            "<first_item>",
            ...
          ]
        },
        "role": "<role_name>"
      },
      ...
    ]

键值对格式如下：

    resources : resource ( ";" resource )*

    resource : key ":" ( scalar | range | set )

    key : text ( "(" resourceRole ")" )?

    resourceRole : text | "* "

注意：resourceRole必须是一个合法的role名字。

### Predefined Uses & Conventions

有一些预定义的资源，比如：

* cpus
* gpus
* disk
* mem
* prots

注意**disk**和**mem**资源是用兆字节(megabytes)指定的。master的用户接口会把资源值转换成更容易阅读的形式，比如值**15000**会被显示成**14.65GB**。

一个没有**cpus**和**mem**资源的agent不会被推荐给任何的Frameworks。

### Examples

默认的，当mesos-agent服务起来后，Mesos会自动检测本地机器上的可用资源。或者也可以显示的配置一个agent上必须可用的资源。下面是一个配置agent资源的例子：

    --resources='cpus:24;gpus:2;mem:24576;disk:409600;ports:[21000-24000,30000-34000];bugs(debug_role):{a,b,c}'

    --resources='[{"name":"cpus","type":"SCALAR","scalar":{"value":24}},{"name":"gpus","type":"SCALAR","scalar":{"value":2}},{"name":"mem","type":"SCALAR","scalar":{"value":24576}},{"name":"disk","type":"SCALAR","scalar":{"value":409600}},{"name":"ports","type":"RANGES","ranges":{"range":[{"begin":21000,"end":24000},{"begin":30000,"end":34000}]}},{"name":"bugs","type":"SET","set":{"item":["a","b","c"]},"role":"debug_role"}]'

或者是一个**resource.txt**这么一个文件，包含如下：

    [
      {
        "name": "cpus",
        "type": "SCALAR",
        "scalar": {
          "value": 24
        }
      },
      {
        "name": "gpus",
        "type": "SCALAR",
        "scalar": {
          "value": 2
        }
      },
      {
        "name": "mem",
        "type": "SCALAR",
        "scalar": {
          "value": 24576
        }
      },
      {
        "name": "disk",
        "type": "SCALAR",
        "scalar": {
          "value": 409600
        }
      },
      {
        "name": "ports",
        "type": "RANGES",
        "ranges": {
          "range": [
            {
              "begin": 21000,
              "end": 24000
            },
            {
              "begin": 30000,
              "end": 34000
            }
          ]
        }
      },
      {
        "name": "bugs",
        "type": "SET",
        "set": {
          "item": [
            "a",
            "b",
            "c"
          ]
        },
        "role": "debug_role"
      }
    ]

那么就可以这么起agent服务：

    $ path/to/mesos-agent --resources=file:///path/to/resources.txt ...

在这里例子中，我们有三种类型的五个资源：scalar,1个range和一个set。scalar有：cpus，gpus，mem和disk，range有prots，set有bugs，bugs被分配给了debug_role。当其他的资源没有指定role时，那么就会被分配给一个默认的role。

注意：默认的role可以通过**--default_role**标签来指定。
* cups是scalar，值是24
* gpus是scalar，值是2
* mem是scalar，值是24576
* disk是scalr，值是409600
* ports是range，值是21000到24000和30000-34000
* bugs是set，值是a,b,c并且指定到debug_role上

配置Mesos agent的attribute，可以通过mesos-agent的命令行参数：**--attribute**。

    --attributes='rack:abc;zone:west;os:centos5;level:10;keys:[1000-1500]'

上面配置了五种属性：

* rack，值是abc
* zone，值是west
* os，值是centos
* level， 值是10
* keys，值是1000-1500
