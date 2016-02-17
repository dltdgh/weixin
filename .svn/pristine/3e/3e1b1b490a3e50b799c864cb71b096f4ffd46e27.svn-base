# -*- coding: utf-8 -*-
from django.http import HttpResponse
from django.utils.encoding import smart_str, smart_unicode

import xml.etree.ElementTree as ET
import hashlib, urllib, urllib2, time
import json

TOKEN = "dltdwxpt"
YOUDAO_KEY = "1206169545"
YOUDAO_KEY_FROM = "hulala"
YOUDAO_DOC_TYPE = "xml"
TULING_KEY = "a05a8acca0c2321d4cb434f52bc76417"

def checkSignature(request):
    global TOKEN
    signature = request.GET.get('signature', None)
    timestamp = request.GET.get('timestamp', None)
    nonce = request.GET.get('nonce', None)
    echoStr = request.GET.get('echostr', None)

    token = TOKEN
    tempList = [token, timestamp, nonce]
    tempList.sort()
    tempStr = "%s%s%s" % tuple(tempList)
    tempStr = hashlib.sha1(tempStr).hexdigest()
    if tempStr == signature:
        return echoStr
    else:
        return None
    pass

def index(request):
    if request.method == 'GET':
        response = HttpResponse(checkSignature(request),content_type="text/plain")
        return response
    elif request.method == 'POST':
        response = HttpResponse(responseMsg(request),content_type="application/xml")
        return response
    else:
        return HttpResponse('hello world!!!')
    pass

def responseMsg(request):
    rawStr = smart_str(request.raw_post_data)
    msg = parseMsgXml(ET.fromstring(rawStr))

    queryStr = msg.get('Content', "You have input nothing!")
    replyContent = ""
    if queryStr[0:1] == "#":
        replyContent = queryInYoudao(msg, queryStr[1:])
        pass
    elif queryStr == "help":
        replyContent = u'主人不在家，拒绝调戏哦～\n目前支持：\n翻译：\n示例\n#how are you\n#魑魅魍魉\n聊天：\n示例\n你好，你是美女么？\n挖掘机技术哪家强？\n笑话：\n示例\n讲个笑话\n冷笑话\n图片：\n示例\n刘亦菲的图片\n天气：\n示例\n北京今天的天气\n北京今天的空气质量\n问答：\n示例\n地球到月球的距离\n感冒怎么办\n虎皮鹦鹉吃什么\n百科：\n示例\n百科周杰伦\n李连杰的介绍\n新闻：\n示例\n我要看新闻\n科技新闻\n周杰伦的新闻\n星座：\n示例\n天蝎座明天的运势\n现在是什么星座\n吉凶：\n示例\n解梦：梦到桃花怎么回事\n周杰伦这个名字好不好\n10086凶吉\n成语接龙：\n示例\n开始成语接龙\n快递：\n顺丰快递\n飞机：\n示例\n明天从北京到上海的航班\n列车：\n示例\n明天从北京到石家庄的火车\n计算：\n示例\n3乘以5等于多少\n25*25等多少\n以上示例不唯一，用户可自行摸索~\n'
        pass
    else:
        replyContent = queryInTuling(msg, queryStr)
        replyContent = replyContent.replace('1<br>', '\n')
        replyContent = replyContent.replace('<br>', '\n')
        pass
    return getReplyXml(msg, replyContent)

def queryInTuling(msg, queryStr):
    raw_tulingURL = "http://www.tuling123.com/openapi/api?key=%s&info=" % (TULING_KEY)
    tulingURL = "%s%s&userid=%s" % (raw_tulingURL, urllib2.quote(queryStr), msg.get('FromUserName'))
    print tulingURL
    req = urllib2.Request(url=tulingURL)
    result = urllib2.urlopen(req).read()
    replyContent = parseTulingJson(json.loads(result))
    return replyContent

def parseTulingJson(dic):
    stateCode = dic['code']
    print stateCode
    if stateCode == 100000:
        replyContent = dic['text']
        pass
    elif stateCode == 200000:
        replyContent = u"%s\n地址:\n%s" % (dic['text'], dic['url'])
        pass
    elif stateCode == 302000:
        replyContent = dic['text']
        for item in dic['list']:
            replyContent = u"%s\n标题:\n%s\n来源:\n%s\n地址:\n%s" % (replyContent, item['article'], item['source'], item['detailurl'])
        pass
    elif stateCode == 305000:
        replyContent = dic['text']
        for item in dic['list']:
            replyContent = u"%s\n车次:\n%s\n起始站:\n%s\n到达站:\n%s\n开车时间:\n%s\n到达时间:\n%s\n地址:\n%s" % (replyContent, item['trainnum'], item['start'], item['terminal'], item['starttime'], item['endtime'], item['detailurl'])
        pass
    elif stateCode == 306000:
        replyContent = dic['text']
        for item in dic['list']:
            replyContent = u"%s\n航班:\n%s\n航班路线:\n%s\n起飞时间:\n%s\n到达时间:\n%s\n航班状态:\n%s\n地址:\n%s" % (replyContent, item['flight'], item['route'], item['starttime'], item['endtime'], item['state'], item['detailurl'])
        pass
    elif stateCode == 308000:
        replyContent = dic['text']
        for item in dic['list']:
            replyContent = u"%s\n名称:\n%s\n详情:\n%s\n地址:\n%s" % (replyContent, item['name'], item['info'],  item['detailurl'])
        pass
    print replyContent
    return replyContent

def queryInYoudao(msg, queryStr):
    raw_youdaoURL = "http://fanyi.youdao.com/openapi.do?keyfrom=%s&key=%s&type=data&doctype=%s&version=1.1&q=" % (YOUDAO_KEY_FROM, YOUDAO_KEY, YOUDAO_DOC_TYPE)
    youdaoURL = "%s%s" % (raw_youdaoURL, urllib2.quote(queryStr))
    req = urllib2.Request(url=youdaoURL)
    result = urllib2.urlopen(req).read()
    replyContent = parseYouDaoXml(ET.fromstring(result))
    return replyContent

def parseMsgXml(rootElem):
    msg = {}
    if rootElem.tag == 'xml':
        for child in rootElem:
            msg[child.tag] = smart_str(child.text)
            pass
        pass
    return msg

def parseYouDaoXml(rootElem):
    replyContent = ""
    if rootElem.tag == 'youdao-fanyi':
        for child in rootElem:
            if child.tag == 'errorCode':
                if child.text == '20':
                    return 'too long to translate\n'
                elif child.text == '30':
                    return "can't be able to translate with effect\n"
                elif child.text == '40':
                    return "can't be able to support this language\n"
                elif child.text == '50':
                    return 'invavid key\n'
                pass
            elif child.tag == 'query':
                replyContent = "%s%s\n" % (replyContent, child.text)
                pass
            elif child.tag == 'translation':
                replyContent = '%s%s\n%s\n' % (replyContent, '-' * 3 + u'有道翻译' + '-' * 3, child[0].text)
                pass
            elif child.tag == 'basic':
                replyContent = "%s%s\n" % (replyContent, '-' * 3 + u'基本翻译' + '-' * 3)
                for c in child:
                    if c.tag == 'phonetic':
                        replyContent = '%s%s\n' % (replyContent, c.text)
                    elif c.tag == 'explains':
                        for ex in c.findall('ex'):
                            replyContent = '%s%s\n' % (replyContent, ex.text)
                            pass
                        pass
                    pass
                pass
            elif child.tag == 'web':
                replyContent = "%s%s\n" % (replyContent, '-' * 3 + u'网络释义' + '-' * 3)
                for explain in child.findall('explain'):
                    for key in explain.findall('key'):
                        replyContent = '%s%s\n' % (replyContent, key.text)
                    for value in explain.findall('value'):
                        for ex in value.findall('ex'):
                            replyContent = '%s%s\n' % (replyContent, ex.text)
                            pass
                    replyContent = '%s%s\n' % (replyContent,'--')
                    pass
    return replyContent

def getReplyXml(msg, replyContent):
     extp = "<xml><ToUserName><![CDATA[%s]]></ToUserName><FromUserName><![CDATA[%s]]></FromUserName><CreateTime>%s</CreateTime><MsgType><![CDATA[%s]]></MsgType><Content><![CDATA[%s]]></Content><FuncFlag>0</FuncFlag></xml>";
     extp = extp % (msg['FromUserName'], msg['ToUserName'], str(int(time.time())), 'text', replyContent)
     return extp

