# -
面试代码
开发工程师测试题
一、编程题
https://iftp.chinamoney.com.cn/english/bdInfo/
用程序实现：
1. 从链接页面获取公开数据
2. 需要获取数据的条件: Bond Type=Treasury Bond, Issue Year=2023
3. 解析返回表格数据，列名包括ISIN, Bond Code, Issuer, Bond Type, Issue Date, Latest Rating
4. 保存成有效csv文件

代码：


import requests
import csv
f=open('第一题.csv',mode='a',encoding='utf-8',newline='')
csv_writer=csv.DictWriter(f,fieldnames=['ISIN', 'Bond_Code','Issuer','Bond_Type',' Issue Date','Latest Rating'])
csv_writer.writeheader()
cookies = {
    'AlteonP10': 'CN2NZiw/F6ySRs5eVXISBA$$',
    'apache': '4a63b086221745dd13be58c2f7de0338',
    'ags': 'b168c5dd63e5c0bebdd4fb78b2b4704a',
    '_ulta_id.ECM-Prod.ccc4': '98d17bfdfc8400d4',
    '_ulta_ses.ECM-Prod.ccc4': 'eae47994ee1dac26',
}

headers = {
    'Accept': 'application/json, text/javascript, */*; q=0.01',
    'Accept-Language': 'zh-CN,zh;q=0.9',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'Content-Type': 'application/x-www-form-urlencoded; charset=UTF-8',
    # 'Cookie': 'AlteonP10=CN2NZiw/F6ySRs5eVXISBA$$; apache=4a63b086221745dd13be58c2f7de0338; ags=b168c5dd63e5c0bebdd4fb78b2b4704a; _ulta_id.ECM-Prod.ccc4=98d17bfdfc8400d4; _ulta_ses.ECM-Prod.ccc4=eae47994ee1dac26',
    'Origin': 'https://iftp.chinamoney.com.cn',
    'Pragma': 'no-cache',
    'Referer': 'https://iftp.chinamoney.com.cn/english/bdInfo/',
    'Sec-Fetch-Dest': 'empty',
    'Sec-Fetch-Mode': 'cors',
    'Sec-Fetch-Site': 'same-origin',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36 SLBrowser/9.0.3.5211 SLBChan/11',
    'X-Requested-With': 'XMLHttpRequest',
    'sec-ch-ua': '"Chromium";v="9", "Not?A_Brand";v="8"',
    'sec-ch-ua-mobile': '?0',
    'sec-ch-ua-platform': '"Windows"',
}

data = {
    'pageNo': '1',
    'pageSize': '15',
    'isin': '',
    'bondCode': '',
    'issueEnty': '',
    'bondType': '100001',
    'couponType': '',
    'issueYear': '2023',
    'rtngShrt': '',
    'bondSpclPrjctVrty': '',
}

response = requests.post(
    'https://iftp.chinamoney.com.cn/ags/ms/cm-u-bond-md/BondMarketInfoListEN',
    cookies=cookies,
    headers=headers,
    data=data,
).json()
for i in range(0,15):
    ISIN = response['data']['resultList'][i]['isin']
    BondCode = response['data']['resultList'][i]['bondCode']
    Issuer = response['data']['resultList'][i]['entyFullName']
    Bond_Type = response['data']['resultList'][i]['bondType']
    Issue_Date = response['data']['resultList'][i]['issueEndDate']
    Latest_Rating = response['data']['resultList'][i]['debtRtng']
    # print(ISIN, '\n', BondCode, '\n', Issuer, Bond_Type, Issue_Date, Latest_Rating)
    data = {
        'ISIN': ISIN,
        'BondCode': BondCode,
        'Issuer': Issuer,
        'Bond_Type': Bond_Type,
        'Issue_Date': Issue_Date,
        'Latest_Rating': Latest_Rating,
    }
    csv_writer.writerow(data)

二、编程题
实现自定义正则匹配函数 reg_search，要求：
输入参数：
1. text (需要正则匹配的文本内容)
2. regex_list (正则表达式列表)
输出参数：
匹配到的结果列表

例如：
text = ’‘’
标的证券：本期发行的证券为可交换为发行人所持中国长江电力股份
有限公司股票（股票代码：600900.SH，股票简称：长江电力）的可交换公司债券。
换股期限：本期可交换公司债券换股期限自可交换公司债券发行结束
之日满 12 个月后的第一个交易日起至可交换债券到期日止，即 2023 年 6 月 2
日至 2027 年 6 月 1 日止。
‘’‘
regex_list=[{
	'标的证券': '*自定义*',
	'换股期限': '*自定义*'
}]

返回结果：
[{
	'标的证券': '600900.SH',
	'换股期限': ['2023-06-02', '2027-06-01']
}]
代码：



import re

def reg_search(text, regex_list):
    results = []

    for regex_dict in regex_list:
        results=[]
        for key, pattern in regex_dict.items():
                if key == '标的证券':
                    # 匹配股票代码的正则表达式
                    match = re.search(pattern,text)
                    if match:
                        results.append({key: match.group(1)})
                if key == '换股期限':
                    # 匹配日期的正则表达式
                    match = re.findall(pattern,text)
                    if match:
                        list=[]
                        for mat in match:
                            mat=mat.replace(' 年','-').replace(' 月','-').replace(' 日','')
                            mat=mat.replace(' ','0')
                            list.append(mat)
                            results[0]['换股期限'] =list

    return results
text = '''
标的证券：本期发行的证券为可交换为发行人所持中国长江电力股份有限公司股票（股票代码：600900.SH，股票简称：长江电力）的可交换公司债券。
换股期限：本期可交换公司债券换股期限自可交换债券发行结束之日满 12 个月后的第一个交易日起至可交换债券到期日止，即 2023 年 6 月 2 日至 2027 年 6 月 1 日止。
'''
regex_list = [
    {
        '标的证券': r'股票代码：(\w+\.\w+)',
        '换股期限': r' (\d{4} 年 \d{1,2} 月 \d{1,2} 日)'
    }
]

print(reg_search(text, regex_list))
