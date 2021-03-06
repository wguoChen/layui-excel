# 扩展 layui 的导出插件 layui.excel

之前在工作过程中还有社区交流过程中，发现对导出 Excel 文件有需求，所以就萌发了封装插件的想法。导出excel功能基于 XLSX.js，下载功能基于 FileSaver，读取文件基于 H5的 FileReader。

> 环境提示：预览环境需要部署在服务器下，不然无法异步获取需要导出的数据

> 版本更新提示：如果使用 v1.2 以前的版本，filterExportData 的映射关系有所调整，请注意

> 浏览器兼容性：支持IE10+、Firefox、chrome


## 功能演示：

##### 在线演示：

[http://excel.wj2015.com/](http://excel.wj2015.com/)

![功能演示](https://raw.githubusercontent.com/wangerzi/layui-excel/master/screenGIF.gif)

## 期望收集


- [x] 支持导出到IE、Firefox(社区：[TeAmo](https://fly.layui.com/u/2297904/))
- [x] 梳理数据函数支持列合并(社区：[SoloAsural](https://fly.layui.com/u/10405920/))
- [ ] 支持Excel内列合并(社区：[SoloAsural](https://fly.layui.com/u/10405920/))
- [x] 优化大量数据导出，比如~~100W~~45W(社区：[Th_omas](https://fly.layui.com/u/28037520/))
- [ ] 支持Excel样式设置(社区：[锁哥](https://fly.layui.com/u/17116008/))
- [x] 可以读取Excel内容(个人)
- [x] 支持一个Excel导出多个sheet（个人、社区：[玛琳菲森 ](https://fly.layui.com/u/29272992/)）


## BUG收集

空

## 接口设计和后台程序参考

完善中....

## 函数列表

> 仅做函数用途介绍，具体使用方法请见 『重要函数参数配置』

| 函数名                                     | 描述                                             |
| ------------------------------------------ | ------------------------------------------------ |
| **exportExcel(data, filename, type, opt)** | 导出数据，并弹出指定文件名的下载框               |
| **filterExportData(data, fields)**         | 梳理导出的数据，包括字段排序和多余数据过滤       |
| **importExcel(files, opt, callback)**      | 读取Excel，支持多文件多表格读取                  |
| filterImportData(data, fields)             | 梳理导入的数据，字段含义与 filterExportData 类似 |
| numToTitle(num)                            | 将0/1/2...转换为A/B/C/D.../AA/AB/.../ZZ/AAA形式  |

## 重要函数参数配置

#### exportExcel参数配置

> 核心方法，用于将 data 数据依次导出，如果需要调整导出后的文件字段顺序或者过滤多余数据，请查看 filterExportData 方法

| 参数名称 | 描述                                             | 默认值 |
| -------- | ------------------------------------------------ | ------ |
| data     | 数据列表（需要指定表名）                         | 必填   |
| filename | 文件名称（带后缀）                               | 必填   |
| type     | 导出类型，支持 xlsx、csv、ods、xlsb、fods、biff2 | xlsx   |
| opt      | 其他可选配置                                     | null   |

##### data样例：

```javascript
{
    "sheet1": [
        {name: '111', sex: 'male'},
        {name: '222', sex: 'female'},
    ]
}
```

##### opt支持的配置项

| 参数名称  | 描述                                                         | 默认值 |
| --------- | ------------------------------------------------------------ | ------ |
| opt.Props | 配置文档基础属性，支持Title、Subject、Author、Manager、Company、Category、Keywords、Comments、LastAuthor、CreatedData |        |

#### filterExportData参数配置

> 辅助方法，梳理导出的数据，包括字段排序和多余数据过滤

| 参数名称 | 描述                                             | 默认值 |
| -------- | ------------------------------------------------ | ------ |
| data     | 需要梳理的数据                                   | 必填   |
| fields   | 支持数组、对象和回调函数，用于映射关系和字段排序 | 必填   |

> fields参数设计

在实际使用的过程中，后端给的参数多了，或者字段数据不符合导出要求，这都是很常见的情况。为了导出数据的顺序正确和数据映射正确，于是新增了这个方法。

fields 用于表示对象中的属性顺序和映射关系，支持『数组』和『对象』两种方式

假如后台给出了这样的数据：

```json
{
    "code":0,
    "msg":"",
    "count":3,
    "data":[
        {
            "id":10000,
            "username":"user-0",
            "sex":"女",
            "city":"城市-0",
            "sign":"签名-0",
            "experience":255,
            "logins":24,
            "wealth":82830700,
            "classify":"作家",
            "score":57,
            "start": '2018-12-29',
            "end": "2018-12-30"
        }
    ]
}
```

**数组方式：**

仅用于排序、字段过滤，比如我希望的导出顺序和字段是：

`id`、`sex`、`username`、`city`

那么，我可以这样写：

```javascript
var data = [];// 假设的后台的数据
data = excel.filterExportData(data, ['id', 'sex', 'username', 'city']);
excel.exportExcel(data, '导出测试.xlsx', 'xlsx');
```

**对象方式：**

可以用于排序、重命名字段、字段过滤，比如我希望 `username` 字段重命名为 `name`，保留 `sex` 和 `city` 字段

那么，我可以这样写：

```javascript
var data = [];// 假设的后台的数据
data = excel.filterExportData(data, {
    name: 'username',
    sex:'sex',
    city: 'city'
});
excel.exportExcel(data, '导出测试.xlsx', 'xlsx');
```

##### 回调方式：

可以用于排序、重命名字段、字段过滤、自定义列，比如我希望 `range` 由 `start` `end` 聚合并以 `~` 分割；修改 `score` 为原有值的 10倍，并且 `username` 字段重命名为 `name`，保留 `sex` 和 `city`  字段。

那么，我可以这样写：

```javascript
var data = [];// 假设的后台的数据
data = excel.filterExportData(data, {
    name: 'username',
    sex:'sex',
    city: 'city',
    range: function(value, line, data) {
        return line['start'] + '~' + line['end'];
    },
    score: function(value, line, data) {
        return value * 10;
    }
});
excel.exportExcel(data, '导出测试.xlsx', 'xlsx');
```

##### 调用样例

请见下方『使用方法』

#### importExcel参数配置

> 核心方法，用于读取用户选择的Excel信息，文件读取基于 FileReader，所以对浏览器版本要求较高

| 参数名称 | 描述                                                         | 默认值    |
| -------- | ------------------------------------------------------------ | --------- |
| files    | 上传文件DOM对象的 files 属性                                 | undefined |
| opt      | 导出参数配置，详见下方描述                                   | undefined |
| callback | 完全读取完毕的回调函数，传入一个参数「data」表示所有数据的集合 | undefined |

##### opt参数配置

| 参数名称 | 描述                                                         | 默认值 |
| -------- | ------------------------------------------------------------ | ------ |
| header   | 导入参数的headers，支持"A"、1等，[详见XLSX官方文档](https://github.com/SheetJS/js-xlsx#json) | A      |
| range    | 读取的范围，支持数字、字符等，[详见XLSX官方文档](https://github.com/SheetJS/js-xlsx#json) | null   |
| fields   | 可以在读取的过程中进行数据梳理，参数意义请参见「filterExportData参数配置」 | null   |

> 由于处理过程中会抛出一些异常，所以请使用 try{}catch(e){}接收并提示用户！
>
> 如果对导出数据格式的键不满意，可以有两种方式梳理：
>
>  	1. 调用 filterImportData(data, fields)
>  	2. 直接在 importExcel() 的 opt 配置中进行数据梳理

##### 调用样例

```javascript
$(function(){
    // 监听上传文件的事件
    $('#LAY-excel-import-excel').change(function(e) {
        var files = e.target.files;
        try {
            // 方式一：先读取数据，后梳理数据
            excel.importExcel(files, {}, function(data) {
                console.log(data);
                data = excel.filterImportData(data, {
                    'id': 'A'
                    ,'username': 'B'
                    ,'experience': 'C'
                    ,'sex': 'D'
                    ,'score': 'E'
                    ,'city': 'F'
                    ,'classify': 'G'
                    ,'wealth': 'H'
                    ,'sign': 'I'
                })
                console.log(data);
            });
            // 方式二：可以在读取过程中梳理数据
            excel.importExcel(files, {
                fields: {
                    'id': 'A'
                    ,'username': 'B'
                    ,'experience': 'C'
                    ,'sex': 'D'
                    ,'score': 'E'
                    ,'city': 'F'
                    ,'classify': 'G'
                    ,'wealth': 'H'
                    ,'sign': 'I'
                }
            }, function(data) {
                console.log(data);
            });
        } catch (e) {
            layer.alert(e.message);
        }
    });
});
```



## 提效建议：

> 数据规模：前端导出**纯数据 9列10w** 的数据量需要 **7秒左右**的时间，**30W数据占用1.8G，耗时24秒**，普通电脑**最多能导出50w数据，耗时45秒**，文件大小173M，提示内存超限

- 如果数据量比较大，最好直接转换为纯数组的导出，可以省去 filterExportData 和 转换为AOA数组的时间和内存（PS：效率提升不算太大，30W数据能提速2s左右，资源主要消耗在调用 XLSX.js之后）
- 一般 exportExcel 会放在 $.ajax() 等异步调用中，如果有需要在点击后纯前端生成Excel，可以使用 async、setTimeout等方式实现异步导出，否则会阻塞主进程。

## 功能概览：

- 支持梳理导出的数据并导出多种格式数据
- 支持IE、火狐、chrome等主流浏览器
- 普通工作电脑最多支持9列45W行数据规模的导出
- 支持 xlx、xlsx、csv格式的前端数据读取以及数据梳理

## 使用方法：

> 注意：此扩展需先引入layui.js方可正常使用。demo详见index.html

##### js使用样例：

```javascript
// 注：lay_exts/ 为扩展中所有文件的存放路径
layui.config({
	base: 'lay_exts/',
}).extend({
	excel: 'excel',
});
layui.use(['jquery', 'excel', 'layer'], function() {
		var $ = layui.jquery;
		var layer = layui.layer;
		var excel = layui.excel;

		// 模拟从后端接口读取需要导出的数据
		$.ajax({
			url: 'list.json'
			,dataType: 'json'
			,success(res) {
				var data = res.data;
				// 重点！！！如果后端给的数据顺序和映射关系不对，请执行梳理函数后导出
				data = excel.filterExportData(data, [
					'id'
					,'username'
					,'experience'
					,'sex'
					,'score'
					,'city'
					,'classify'
					,'wealth'
					,'sign'
				]);
				// 重点2！！！一般都需要加一个表头，表头的键名顺序需要与最终导出的数据一致
				data.unshift({ id: "ID", username: "用户名", experience: '积分', sex: '性别', score: '评分', city: '城市', classify: '职业', wealth: '财富', sign: '签名' });

				var timestart = Date.now();
				excel.exportExcel(data, '导出接口数据', 'xlsx');
				var timeend = Date.now();

				var spent = (timeend - timestart) / 1000;
				layer.alert('单纯导出耗时 '+spent+' s');
			}
			,error() {
				layer.alert('获取数据失败，请检查是否部署在本地服务器环境下');
			}
		});
	});
```

##### 导出数据返回样例：

> 此数据来自 layui 官方的表格样例

```json
{
    "code":0,
    "msg":"",
    "count":3,
    "data":[
        {
            "id":10000,
            "username":"user-0",
            "sex":"女",
            "city":"城市-0",
            "sign":"签名-0",
            "experience":255,
            "logins":24,
            "wealth":82830700,
            "classify":"作家",
            "score":57
        },
        {
            "id":10001,
            "username":"user-1",
            "sex":"男",
            "city":"城市-1",
            "sign":"签名-1",
            "experience":884,
            "logins":58,
            "wealth":64928690,
            "classify":"词人",
            "score":27
        },
        {
            "id":10002,
            "username":"user-2",
            "sex":"女",
            "city":"城市-2",
            "sign":"签名-2",
            "experience":650,
            "logins":77,
            "wealth":6298078,
            "classify":"酱油",
            "score":31
        }
    ]
}
```

##### Demo说明：

index.html			页面文件+JS处理文件

list.json				模拟导出的数据

extends/excel.js		权限树扩展

layui/				官网下载的layui

## 更新预告：

支持导出多个sheet，合并导出的列，设置单元格样式

## 更新记录：

2019-01-04 v1.2 支持前端多文件多Sheet读取 Excel 数据并梳理数据格式，大量数据导出效率优化

2018-12-29 v1.1 重写内部下载逻辑，支持IE、Firefox、chrome等主流浏览器，梳理数据函数支持回调

2018-12-14 v1.0 最初版本

## 特别感谢

暂无
