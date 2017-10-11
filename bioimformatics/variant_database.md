Title: 基于MongoDB和Flask的变异信息数据库
Date: 2016-09-21
Category: bioinformatics
Tags: bioinformatics, MongoDB, Flask, variant database

## 临床变异数据库

当积累的一定的样本量之后, 就应该积累自己的数据库, 建立变异数据库的意义在于:

1. 样品管理;
2. 变异数据搜集, 以及应用该数据库注释新的变异;

变异数据库的建立不应该脱离表型, 结合LIMS系统中提供的样品信息, 可以结合表型和基因型;

## 读取VCF文件中的变异信息

在构建变异数据库时, 不应该建立在已有注释的前提之下, 所以数据库应该以calling的VCF结果为起点;

为了保证VCF文件中的变异条目具有唯一表示, 在录入之前必须先进行[标准化处理](http://www.lyon0804.com/fan-yi-variant-normalization.html);

以下是VCF文件的读取实现:

```python
import vcf
# 自己实现的VariantRecord类, 其中包含变异标准化方法
from lib.variant import VariantRecord

def vcf_reader(file_name):
    result_list = []
    with open(file_name, encoding='utf-8') as fp:
        reader = vcf.Reader(fp, 'r')
        for record in reader:
            for alt in record.ALT:
                variant = VariantRecord(str(record.CHROM), record.POS, str(record.REF), str(alt))
                variant.normalization()
                result_list.append({
                    'CHROM': variant.chrom,
                    'POS': variant.pos,
                    'ID': record.ID,
                    'REF': variant.ref,
                    'ALT': variant.alt,
                    'FILTER': record.FILTER,
                    'QUAL': record.QUAL,
                    'INFO': record.INFO,
                    'GT': record.samples[0].data.GT,
                    'AD': record.samples[0].data.AD,
                    'DP': record.samples[0].data.DP,
                    'GQ': record.samples[0].data.GQ,
                    'PL': record.samples[0].data.PL,
                })
    return result_list
```

## 录入变异信息到MongoDB

首先在mongo下建立数据库以及新增用户;

```mongo
> use variant_info
> db.addUser('user', '123', true)
> db.addUser('admin', '123')
```

只有VCF文件难以对应到具体的样品, 所以在VCF信息录入时, 添加相应的样品信息, 并方便通过样品条码号等调用lims中的表型信息;

以下提供一个插入的示例, *.seq*文件是记录一些样品测序信息的tsv文件.

```python
import csv
from collections import namedtuple
from datetime import datetime
from pymongo import MongoClient

client = MongoClient('xxxx', xxx)

def admin_acquire(fn):
    def wrapper(self, *args, **kwargs):
        self.db.logout()
        self.db.authenticate(name='admin', password='123')
        fn(self, *args, **kwargs)
        self.db.logout()
        self.db.authenticate(name='user', password='123')

    return wrapper


class Germline(object):
    def __init__(self):
        self.db = client.variant_info
        self.db.authenticate(name='user', password='123')
        self.collection = self.db.germline

    def create_index(self):
        ## 按照具体查询需求建立索引

    @admin_acquire
    def db_insert(self, seq_date, update=False):
        ## 一个插入示例,     
        self.create_index()
        file_name = "%s.seq" % seq_date.strftime('%Y-%m-%d')
        with open(file_name, encoding='utf-8') as fp:
            reader = csv.reader(fp, delimiter='\t')
            header = next(reader)
            Row = namedtuple('Row', header)
            for row in reader:
                row = Row(*row)._asdict()
                row['seq_date'] = seq_date
                seq_reg_date = seq_date.strftime('%Y-%m-%d')
                seq_name = row['seq_name']
                vcf_file = ("/bio01/database/VariantCalling/germline/vcf_filtered/"
                            "{seq_reg_date}/{seq_name}.filtered.vcf"
                            .format_map(vars()))
                calling_result = self.vcf_reader(vcf_file)
                document = row
                document.update({'calling_result': calling_result})
                try_find = self.collection.find_one({'seq_date': seq_date, 'seq_name': seq_name})
                if try_find is None:
                    self.collection.insert(document)
                elif update:
                    self.collection.update({'_id': try_find['_id']}, document)
                else:
                    continue
```

## 通用查询接口

变异数据库查询时, 若所需的结果格式相同, 则可以通过一个简单的方法, 以便处理来自所有不同字段的查询;

当某种类型的查询比较频繁时, 可以针对性建立index来加速查询;

```python
class Germline(object):
    ## 续上类
    
    @staticmethod
    def format_query(query_result):
        search_results = []
        show_item = ['seq_date', 'seq_name', 'sample_name', 'sample_id', 'seq_kit']

        def sort_key(x):
            return x['seq_date'], x['sample_name']

        if not query_result:
            r = {k: 'NA' for k in show_item}
            search_results.append(r)
        for q in query_result:
            q['seq_date'] = q['seq_date'].strftime('%Y-%m-%d')
            r = {k: q[k] for k in show_item}
            search_results.append(r)
        search_results.sort(key=sort_key)
        return search_results

    def auto_query(self, **kwargs):
        binding_items = {
            'sample_name': 'sample_name',
            'sample_id': 'sample_id',
            'seq_kit': 'seq_kit',
            'seq_date': 'seq_date',
            'CHROM': 'calling_result.CHROM',
            'POS': 'calling_result.POS',
            'REF': 'calling_result.REF',
            'ALT': 'calling_result.ALT',
            'ID': 'calling_result.ID',
        }
        search_key = {}
        for k, v in binding_items.items():
            if k in kwargs:
                search_key[v] = kwargs[k]
        cursor = self.collection.find(search_key)
        return self.format_query(list(cursor))
```

当需要多种结果格式时, 可以在此基础上再做封装;

## 使用Flask提供API

后台选择使用Flask来提供web api, 下面是一个实现示例:

```python
from datetime import datetime
from flask import jsonify

@app.route('/seq_info', methods=['POST', 'GET'])
def seq_info():
    if request.method == 'POST':
        item = request.form.to_dict()
        if 'POS' in item:
            item['POS'] = int(item['POS'])
        if 'seq_date' in item:
            item['seq_date'] = datetime.strptime(item['seq_date'], '%Y-%m-%d')
        print(item, file=sys.stderr)
        from database.variant_info import Germline
        db = Germline()
        search_result = db.auto_query(**item)
        return jsonify(seq_info=search_result)
    return render_template('seq_info.html')
```

## 使用Ajax查询结果并展示

整个页面的设计比较简单, 主要就通过一个Ajax来调用api, 然后返回结果展示;

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>查询变异信息</title>
    <script src="https://code.jquery.com/jquery-1.12.4.min.js"
            integrity="sha256-ZosEbRLbNQzLpnKIkEdrPv7lOy9C27hHQ+Xp8a4MxAQ=" crossorigin="anonymous">
    </script>
</head>
<body>
<script>
    $(function () {
        var result_function = function (req) {
            return function () {
                if (req.readyState == 4) {
                    if (req.status != 200) {
                        //error handling code here
                    }
                    else {
                        var response = JSON.parse(req.responseText);
                        for (var i = 0, l = response.seq_info.length; i < l; i++) {
                            $("#result_table").append('<tr/>');
                            $("<td>" + response.seq_info[i].seq_date + "</td>").appendTo("#result_table tr:last");
                            $("<td>" + response.seq_info[i].seq_name + "</td>").appendTo("#result_table tr:last");
                            $("<td>" + response.seq_info[i].sample_name + "</td>").appendTo("#result_table tr:last");
                            $("<td>" + response.seq_info[i].sample_id + "</td>").appendTo("#result_table tr:last");
                            $("<td>" + response.seq_info[i].seq_kit + "</td>").appendTo("#result_table tr:last");
                        }
                    }
                }
            }
        };
        var get_item = function (key_name) {
            var search_key = '#' + key_name;
            return $(search_key).val().trim();
        };
        $(document).keydown(function (event) {
            if (event.keyCode == 13) {
                $("#submit").click();
                return false
            }
        });
        $("#submit").click(function () {
            $("#result_content").empty();
            var req = new XMLHttpRequest();
            req.onreadystatechange = result_function(req);
            req.open('POST', '{{ url_for("seq_info") }}');
            req.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
            var search_keys = [
                'sample_name', 'sample_id', 'seq_kit', 'seq_date',
                'CHROM', 'POS', 'REF', 'ALT', 'ID'
            ];
            var postVars = 'secret=' + get_item('secret');
            search_keys.forEach(function (key) {
                var value = get_item(key);
                if (value.length > 0) {
                    postVars += '&' + key + '=' + value;
                }
            });
            req.send(postVars);
            return false
        });
    })
</script>
<center>

    <form action="" method="POST">
        <p>
            快速查询方式1：
            <br>
        </p>
        <span>样品名称：</span><input type="text" name="sample_name" id="sample_name" title="样品名称">
        <p>
            快速查询方式2:
            <br>
        </p>
        <span>染色体编号：</span><input type="text" name="CHROM" id="CHROM" title="染色体编号">
        <br>
        <span>染色体位置：</span><input type="number" name="POS" id="POS" title="染色体位置">
        <br>
        <span>REF: </span><input type="text" name="REF" id="REF" title="REF">
        <br>
        <span>ALT: </span><input type="text" name="ALT" id="ALT" title="ALT">

        <p>
            普通查询方式（若包含以下查询项，则查询速度会较慢）：
        </p>
        <span>变异ID（早期样品无法通过该项查找）：</span><input type="text" name="ID" id="ID" title="变异ID">
        <br>
        <span>条码号：</span><input type="text" name="sample_id" id="sample_id" title="条码号">
        <br>
        <span>测序试剂盒：</span><input type="text" name="seq_kit" id="seq_kit" title="测序试剂盒">
        <br>
        <span>测序日期（%Y-%m-%d格式）：</span><input type="datetime" name="seq_date" id="seq_date" title="测序日期">
        <br>

        <input type="hidden" name="secret" id="secret" value="shhh">
        <input type="button" value="查询" id="submit">
    </form>
    <div id="myDiv">
        <table id="result_table" border="1">
            <thead>
            <tr>
                <th width="200px">测序时间</th>
                <th width="200px">测序样品名</th>
                <th width="200px">样品名</th>
                <th width="200px">条码号</th>
                <th width="200px">试剂盒</th>
            </tr>
            </thead>
            <tbody id="result_content">
            </tbody>
        </table>
    </div>
</center>
</body>
</html>
```

## 最终实现

![实现结果](http://ww3.sinaimg.cn/large/95202659jw1f81c8kamq5j21vy0tkn1x.jpg)

该网页对外实现的功能有:

1. 通过样品信息查询测序情况;
2. 通过变异位点, 查询曾经出现过相同变异的样品;