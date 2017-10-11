Title: 规模化爬取NCBI
Date: 2016-03-20
Category: bioinformatics
Tags: bioinpormatics, python, spider

在生物信息领域, [NCBI(*National Center for Biotechnology Information, 美国国家生物信息中心*)](http://www.ncbi.nlm.nih.gov)绝对是必不可少的网站, 该网站提供了很多文章的原始测序数据(SRA)、基因组数据(genome)、核酸库(NT)、蛋白库(NR)、基因信息(gene)、文章数据(PubMed/PMC)、物种分类数据(Taxnomy)、变异数据(SNP)、临床数据(Clinvar)等等大量的数据信息来源, 以及提供blast在线比对服务;

大多数NCBI的数据库都可以在其[FTP站点](ftp.ncbi.nlm.nih.gov)上找到, 但是仍然有一些数据没有被固化成文件, 为了要使用这些数据, 避免每次都手动去网站上查询, 就必须动用一些网络爬虫的手段;

## 目标

1. 尽量不要动用网络爬虫, 如果可以, 尽量尝试使用bioperl或者biopython等提供的Entrez数据库的API, 这样可以减少NCBI网站的访问压力, 毕竟NCBI是非营利性站点, 给大家提供许多便利, 也需要大家的爱护;
2. 当必须借助网络爬虫时, 尽量控制访问频率;
3. 在以上两条前提之下, 实现规模化的信息爬取;
4. 由于国内网络环境, 需要处理各种网络不稳定造成的连接问题;
5. 由于国内网络环境, 应该实现实时输出, 断点续传的功能;

## 获取Clinvar转录本信息

在NCBI的FTP站点上下载的Clinvar数据库(vcf)中, 并没有包含转录本信息, 如果需要对变异注释的结果进行转录本的验证, 就需要获取到Clinvar的转录本信息;

Clinvar通过rs号查询转录本的链接格式为: 

`http://www.ncbi.nlm.nih.gov/clinvar/?term=rs80359578`

当查询到的转录本条目数量只有一条时, Clinvar会自动跳转到该记录的详细页面, 如上述例子中, 会被自动转到`http://www.ncbi.nlm.nih.gov/clinvar/variation/52075/`, 网页形式为:

![Clinvar单条](http://ww2.sinaimg.cn/large/95202659gw1f2euem0zp4j21kw0x47g0.jpg)

当查询到的转录本条目数量大于一条时, Clinvar会显示查询到的列表内容, 网页形式为:

![Clinvar多条](http://ww3.sinaimg.cn/large/95202659gw1f2euemh9jqj21kw0sw46x.jpg)

所以不同的页面需要使用不同的解析方式, 具体就不再展开, 大家根据不同的网页自己解析, 示例代码:

```python
from bs4 import BeautifulSoup


def parse_html(data):
    soup = BeautifulSoup(data, 'lxml')
    result_list = []
    if soup.find('div', id='faceted_search') is not None:
        location_list = soup.find_all('span', class_='ui-button-text')
        for location in location_list:
            if location.string == 'x':
                continue
            result_list.append(location.string)
    else:
        maincontent = soup.find('div', id='maincontent')
        location = maincontent.find('h2')
        result_list.append(location.string)
    return result_list
```

## 基本爬虫实现

完成了网页解析, 下一步就要设计爬虫开始爬取了, 首先不考虑其他问题, 先将网页成功爬取并解析;

由于之后的爬取量会非常大(如Clinvar中大概有11万条rs记录), 需要一些必要的头文件伪装:

```python
from urllib import request


def get_request(url):
    req = request.Request(url=url, method='GET')
    req.add_header(
        'Content-Type',
        'application/x-www-form-urlencoded;'
        'charset=utf-8'
    )
    req.add_header(
        'User-Agent',
        'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_0) '
        'AppleWebKit/601.1.56 (KHTML, like Gecko) '
        'Version/9.0 Safari/601.1.56'
    )
    req.add_header(
        'Referer', 'http://www.ncbi.nlm.nih.gov'
    )
    req.add_header('Host', 'www.ncbi.nlm.nih.gov')
    return req
```

以上使用了Python3的`urllib`, 如果使用Python2, 则需要换成对应的request模块;

通过简单伪装之后, 就可以通过rs号进行爬取了:

```python
from urllib import request


def get_clinvar_transcript(rs_string):
    url = 'http://www.ncbi.nlm.nih.gov/clinvar/?term=%s' % rs_string
    req = get_request(url)
    with request.urlopen(req) as fp:
        data = fp.read()
    transcript_list = parse_html(data)
    return transcript_list
```

## 网络不稳定问题

到这里为止, 简单的爬虫功能已经实现, 但是经验告诉我们, 在国内NCBI的访问并不是很稳定, 在爬取过程中也有可能出现很多的网络问题, 常见的异常有: `urllib.error.URLError`、`http.client.IncompleteRead`、`ConnectionResetError`等, 为了防止响应速度过慢, 我们还需要在访问时加上超时设置, 所以还需要解决`socket.timeout`异常;

```python
from urllib.error import URLError
from http.client import IncompleteRead
from socket import timeout


def get_data(req):
    try_count = 0
    while 1:
        if try_count >= 10:
            return None
        try:
            with request.urlopen(req, timeout=10) as fp:
                data = fp.read()
            break
        except (URLError,
                IncompleteRead,
                ConnectionResetError,
                timeout):
            try_count += 1
            print('error occurred, reloading %s time: %s... ' % (try_count, url))
            continue
    return data
```

以上只是我暂时发现的一些连接问题, 如果还有其他问题也类似解决;

## 并行化爬虫

由于爬取内容非常多, 单进程的爬虫效率太慢, 为了提升爬虫效率, 应该采用多进程的爬虫结构并行爬取; 这将连带涉及到以下问题:

1. 多进程爬虫涉及到并行模型的设计, 由于爬取量非常大, 除了避免进程资源的重复申请, 应该设计在某个时间节点对爬取结果进行一次同步输出; 
2. 由于网络爬虫的问题特性, IO速度往往是影响爬虫效率最重要的因素, **异步**结构显得非常重要;

Python有非常多的异步并行设计方案, 这里介绍`multiprocessing.Pool`进程池的异步并行结构: `Pool.map_async`和`Pool.apply_async`; 

在并行结构同步时, 每个进程返回的内容必须具有连续内存结构, 否则可能导致`MaybeEncodingError`异常, 当需要同步一些容器结构(如:字典)时, 则需要使用json格式进行中转, 所以需要对之前的函数进行简单的修改:

```python
import json


def get_clinvar_transcript(rs_string):
    url = 'http://www.ncbi.nlm.nih.gov/clinvar/?term=%s' % rs_string
    req = get_request(url)
    data = get_data(req)
    if data is None:
        return None
    transcript_list = parse_html(data)
    result = {rs_string: transcript_list}

    return json.dumps(result)
```

以下展示一种异步并行实现方案:

```python
import json
from multiprocessing import Pool


def get_multiprocessing(rs_list, out_fp, threading=20):
    pool = Pool(threading)
    result_list = []
    for rs_string in rs_list:
        result = pool.apply_async(get_clinvar_transcript,
                                  args=(rs_string,))
        result_list.append(result)
        # 同步条件, 这里设置成了当获取的条目数等于进程池大小时同步
        if len(result_list) >= threading:

            # 同步输出函数
            output_result(result_list, out_fp)
            result_list = []
    output_result(result_list, out_fp)
    pool.close()
    pool.join()


def output_result(result_list, out_fp):
    result_list = (
        json.loads(result.get())
        for result in result_list
        if result.get() is not None
    )
    for result in result_list:
        for rs_id, variant_list in result.items():
            for variant_id in variant_list:
                out_fp.write('%s\t%s\n' % (rs_id, variant_id))
	 
```

在以上实现中, rs_list中的rs号被推进固定大小的进程池中, 并执行爬取, 当爬取的结果达到一定的条件时, 执行同步输出, 此时`result.get()`会形成阻塞, 达到同步的目的; 该条件不应该小于进程池的大小, 否则会由于阻塞而不能达到预定的并行量, 同时也不应该设置太大, 避免出现异常而没有对结果及时输出;

## 设置断点

由于爬取内容很多, 耗时较长, 在实际爬取过程中, 并不能一次性成功爬取完毕, 很难避免中途需要中断暂停等, 设置断点记录爬取状态显得尤为重要; 而实现这一功能却非常简单: 将每一个已经爬取到的rs号记录下来即可, 而当获取欲求的rs号时, 直接排除已经爬取过的rs号;

```python
def read_already(file_name):
    with open(file_name) as fp:
        for line in fp:
            rs_string = line.rstrip()
            if rs_string:
                yield rs_string


def read_rs(file_name, already_list=None):
    with open(file_name) as fp:
        for line in fp:
            rs_string = line.rstrip()
            if rs_string:
                if (already_list is not None and
                        rs_string not in already_list):
                    yield rs_string
```

然后在`get_multiprocessing()`方法中, 将其中的`rs_list`改成`read_rs`, 并在同步输出时, 将rs号也输出到某一文件中;

## 设置代理

当爬取太过频繁时, 可能会面临被站点封锁的风险, 当然最好是不要太过频繁地爬取, 如果非要这么做, 设置代理访问会变得比较安全; 这里不建议使用国内某些网站提供的免费代理, 这些代理地址稳定性太差, 远不如不设置代理时的稳定性; 可能付费的代理会好一些, 这里我没有进行更加深入的尝试;

## 代码清单

最终代码清单为

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*- \#
"""
@author = 'liangzb'
@date = '2016-03-30'

"""

import os
import sys
import json
import argparse
from multiprocessing import Pool
from socket import timeout
from urllib import request
from urllib.error import URLError
from http.client import IncompleteRead
from bs4 import BeautifulSoup


class SafeSub(dict):
    def __missing__(self, key):
        return '{' + key + '}'


def f(text, mapping=None):
    """
    it is a imitation of py3.6
    """
    if mapping is None:
        text = text.format_map(SafeSub(sys._getframe(1).f_locals))
        return text.format_map(SafeSub(sys._getframe(1).f_globals))
    elif isinstance(mapping, dict):
        return text.format_map(SafeSub(mapping))
    else:
        return text.format_map(SafeSub(vars(mapping)))


def read_params():
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument('-r', '--rs_file', dest='rs_file',
                        metavar='FILE', type=str, required=True,
                        help="set the rs file, "
                             "with one rs_num per line")
    parser.add_argument('-o', '--out_file', dest='out_file',
                        metavar='FILE', type=str, default='./output.txt',
                        help="set the output file")
    parser.add_argument('-m', '--already_file', dest='already_file',
                        metavar='FILE', type=str, default='./already.txt',
                        help="set the already file, "
                             "with one rs_string per line "
                             "shows which rs has been crawled")
    parser.add_argument('-t', '--threading', dest='threading',
                        metavar='INT', type=int, default=20,
                        help="how many threads will you use")
    args = parser.parse_args()
    return args


def read_already(file_name):
    with open(file_name) as fp:
        for line in fp:
            rs_string = line.rstrip()
            if rs_string:
                yield rs_string


def read_rs(file_name, already_list=None):
    with open(file_name) as fp:
        for line in fp:
            rs_string = line.rstrip()
            if (rs_string and
                    already_list is not None and
                    rs_string not in already_list):
                yield rs_string


def get_request(url):
    req = request.Request(url=url, method='GET')
    req.add_header(
        'Content-Type',
        'application/x-www-form-urlencoded;'
        'charset=utf-8'
    )
    req.add_header(
        'User-Agent',
        'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_0) '
        'AppleWebKit/601.1.56 (KHTML, like Gecko) '
        'Version/9.0 Safari/601.1.56'
    )
    req.add_header(
        'Referer', 'http://www.ncbi.nlm.nih.gov'
    )
    req.add_header('Host', 'www.ncbi.nlm.nih.gov')
    return req


def parse_html(data):
    soup = BeautifulSoup(data, 'lxml')
    result_list = []
    if soup.find('div', id='faceted_search') is not None:
        location_list = soup.find_all('span', class_='ui-button-text')
        for location in location_list:
            if location.string == 'x':
                continue
            result_list.append(location.string)
    else:
        maincontent = soup.find('div', id='maincontent')
        location = maincontent.find('h2')
        result_list.append(location.string)
    return result_list


def get_data(req):
    try_count = 0
    url = req.get_full_url()
    while 1:
        if try_count >= 10:
            return None
        try:
            with request.urlopen(req) as fp:
                data = fp.read()
                return data
        except (URLError,
                IncompleteRead,
                ConnectionResetError,
                timeout):
            try_count += 1
            print(f('error occurred, reloading {try_count} time: {url}... '))
            continue


def get_clinvar_transcript(rs_string):
    url = f('http://www.ncbi.nlm.nih.gov/clinvar/?term={rs_string}')
    req = get_request(url)
    data = get_data(req)
    if data is None:
        return None
    transcript_list = parse_html(data)
    result = {rs_string: transcript_list}
    return json.dumps(result)


def get_multiprocessing(rs_file, already_list,
                        out_fp, log_fp, threading=20):
    pool = Pool(threading)
    result_list = []
    for rs_string in read_rs(rs_file, already_list):
        result = pool.apply_async(get_clinvar_transcript,
                                  args=(rs_string,))
        result_list.append(result)
        # 同步条件, 这里设置成了当获取的条目数等于进程池大小时同步
        if len(result_list) >= threading:
            # 同步输出函数
            output_result(result_list, out_fp, log_fp)
            result_list = []
    output_result(result_list, out_fp, log_fp)
    pool.close()
    pool.join()


def output_result(result_list, out_fp, log_fp):
    result_list = (
        json.loads(result.get())
        for result in result_list
        if result.get() is not None
    )
    for result in result_list:
        print(result)
        for rs_id, variant_list in result.items():
            log_fp.write(f('{rs_id}\n'))
            for variant_id in variant_list:
                out_fp.write(f('{rs_id}\t{variant_id}\n'))


def create_file(file_name):
    if not os.path.isfile(file_name):
        open(file_name, 'w').close()


if __name__ == '__main__':
    params = read_params()
    create_file(params.already_file)
    create_file(params.out_file)
    already_list = list(read_already(params.already_file))
    with open(str(params.out_file), 'a') as out, \
            open(str(params.already_file), 'a') as already:
        get_multiprocessing(params.rs_file, already_list,
                            out, already, params.threading)

```

## 文明上网, 和谐爬取

再强的服务器都会有访问上限, 太频繁的爬取会影响网站性能, 希望大家注意和谐。