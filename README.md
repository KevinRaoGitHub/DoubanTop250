# DoubanTop250
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# __author__ = "Kevin Rao"
# Date: 2018/10/03


import requests
from bs4 import BeautifulSoup
import re
import xlwt
import pymysql




# 根据地址、开始页码和参数信息获取网页文本内容
def getDouban(url, startRow):  # 自定义函数名称；豆瓣网页地址分析start：(页码数-1)*25，即开始行；字符串形式返回网页内容
    if startRow == 0:
        param = {}  # 设置param参数，判断开始行是否为0，为0则返回空
    else:
        param = {'start': startRow, 'filter': ''}  # 头部筛选链接参数；注意：startRow不能加引号，否则重复前25行信息至250行
    r = requests.get(url, params=param, headers={'User-Agent': 'Mozilla/4.0'})  # 伪装成浏览器访问
    return r.text  # 返回内容

# 获取字符串中有效数据
dataList = []  # 定义全局变量


def getdata(html):
    soup = BeautifulSoup(html, 'html.parser')
    movieList = soup.find('ol', attrs={'class': 'grid_view'})
    for movieLi in movieList.find_all('li'):
        data = []
        # 获取电影名称
        movieHd = movieLi.find('div', attrs={'class': 'hd'})
        movieName = movieHd.find('span', attrs={'class': 'title'}).getText()
        data.append(movieName)

        # 获取电影地区
        movieArea = movieLi.find('div', attrs={'class': 'bd'})
        movieCountry = re.findall('&nbsp;/&nbsp;(/...<br>(.+?)<div class="star"> )&nbsp;/&nbsp;', str(movieArea))  # 引用正则
        data.append(movieCountry)

        # 获取电影年份
        movieTime = movieLi.find('div', attrs={'class': 'bd'})
        movieYear = re.findall('([0-9]{2,4})&nbsp;/&nbsp;', str(movieTime))  # 引用正则
        data.append(movieYear)
        < p

        class ="" >.* ?(\d+)

        # 获取电影分数
        movieScore = movieLi.find('span', attrs={'class': 'rating_num'}).getText()
        data.append(movieScore)

        # 获取影评人数
        movieEval = movieLi.find('div', attrs={'class': 'star'})
        movieEvalNum = re.findall(r'\d+', str(movieEval))[-1]  # 引用正则、[-1]取最后的数字
        data.append(movieEvalNum)

        # 获取影评
        movieQuote = movieLi.find('span', attrs={'class': 'inq'})
        if movieQuote:
            data.append(movieQuote.getText())
        else:
            data.append('无影评')

        dataList.append(data)  # 存放进全局变量
    return

# 数据保存进Excel


def saveDate(savePath):
    book = xlwt.Workbook(encoding='utf-8')
    sheet = book.add_sheet('豆瓣Top250电影信息')
    col = (u'电影名称', u'电影地区', u'电影年份', u'评分', u'影评人数', u'影评')  # 定义Excel各列，元组固定
    for i in range(0, 6):
        sheet.write(0, i, col[i])
    for i in range(0, 250):
        data = dataList[i]
        for j in range(0, 6):
            sheet.write(i+1, j, data[j])
    book.save(savePath)
    return

# 定义主函数调用


def mainFunc():
    url = 'https://movie.douban.com/top250'
    startRow = 0
    while startRow < 250:
        html = getDouban(url, startRow)
        getdata(html)
        startRow += 25  # 25叠加
    saveDate('DoubanMovieDataTOP250.xls')
    return
mainFunc()

# 数据保存进MySQL
